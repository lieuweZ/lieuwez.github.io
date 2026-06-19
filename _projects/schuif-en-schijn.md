---
layout: project
title: NEMO schuif & schijn
category: Unity, C#
image: /assets/images/schuif-en-schijn.png
image_contain: true
order: 0
---
[**Speel Schuif en Schijn op itch.io!**](https://swzwij.itch.io/schuif-en-schijn)

[📄 Lees het volledige procesverslag (PDF)](/assets/Proces-Schuif-en-Schijn.pdf)


<img src="/assets/images/NEMO%20Gameplay.gif" alt="NEMO gameplay demo" class="project-demo-gif project-demo-gif-wide">

<img src="/assets/images/schuif-en-schijn-gameplay.png" alt="Schuif en Schijn gameplay demo" class="project-demo-gif project-demo-gif-wide">

## Overview

Schuif en Schijn is een 2D puzzelgame ontwikkeld in Unity voor het NEMO Science Museum. In deze game leren spelers spelenderwijs over de principes van lichtreflectie en subtractieve kleurmenging. Door spiegels te roteren en filters op de juiste posities te schuiven, sturen spelers de laserstralen naar de einddoelen om ze te activeren met de gevraagde kleur.

Het project is gemaakt door vier game development studenten van de Hogeschool van Amsterdam als onderdeel van een opdracht voor een tentoonstelling over game development in het NEMO Science Museum. Het team heeft het volledige ontwikkelproces doorlopen en geeft bezoekers zo een interactieve blik achter de schermen van gamedesign en software engineering.

## Core Features

- **Dynamisch Lasersysteem**: Laserstralen die in real-time reflecteren op spiegels en door objecten worden geblokkeerd.
- **CMY Kleurfilters**: Kleurfilters die subtractieve kleurmenging (Cyan, Magenta, Yellow) toepassen op passerend laserlicht. Wit licht bevat alle kleuren — een filter absorbeert een deel van het spectrum en laat de rest door. Een magentafilter haalt groen weg, een geelfilter haalt blauw weg; zet je die twee achter elkaar, dan blijft rood over. 

Al zouden we primaire RGB-filters gebruiken, dan verdwijnt deze puzzelmogelijkheid: een RGB-filter laat slechts één kleur door en blokkeert de andere twee direct. Het stapelen van twee verschillende RGB-filters blokkeert daardoor gelijk het volledige spectrum. Het CMY-model biedt precies de structuur die nodig is — elke combinatie van twee filters levert nog een bruikbare kleur op, waardoor zinvolle puzzels mogelijk zijn.Dit staat tegenover additieve kleurmenging (RGB), waarbij kleuren juist worden opgeteld: rood + groen = geel, alle drie samen = wit.
- **Oplaadbare Targets**: Einddoelen die alleen opladen wanneer ze door de juiste kleur laserlicht worden geraakt, en langzaam ontladen bij gebrek aan licht.
- **Level Management**: Een gecentraliseerd systeem dat de voortgang van de speler beheert en dynamisch nieuwe levels inlaadt. Spelers moeten in totaal 2 levels van een moeilijkheidsniveau halen voordat de moeilijkheidsgraad omhoog gaat.

**Mijn bijdragen waren gericht op:**
- Ontwerp en implementatie van het oplaadsysteem voor de **einddoelen** (`LevelTarget.cs` en `LevelTargetColor.cs`).
- Realisatie van de **subtractieve kleurfilters** (`LaserColorFilter.cs`) om laserstralen van kleur te veranderen op basis van RGB/CMY-logica.
- Mede-ontwikkeling van de **Level Manager** (`LevelManager.cs`) voor het registreren van doelen en het controleren van de win-condities.
- Analyse maken hoe kinderen leren en hoe dit het beste kan worden toegepast in game design. Door levels en progressie te bestuderen zorgt het ervoor dat de opbouw geschikt is voor een brede doelgroep met verschillende niveaus van begrip. Dat zorgt ervoor dat iedereen op hun eigen manier iets van het spel leert.

---

## Level Progressie

De levelopbouw volgt de structuur van **Kishōtenketsu** : een Oost-Aziatisch vertelvorm met vier fases die ook in level design wordt toegepast. Introductie (*Ki*) spiegels worden geïntroduceerd zonder afleiding. Ontwikkeling (*Shō*) filters worden apart uitgediept. Wending (*Ten*) de twee mechanics worden gecombineerd, wat een nieuw probleem creëert zonder dat er een nieuwe regel nodig is. Oplossing (*Ketsu*) de schuifpuzzels brengen alles samen in toenemende complexiteit.

| Niveau 
|--------|---------|
| 1 | Alleen spiegels |
| 2 | Alleen filters |
| 3 | Niet over elkaar heen |
| 4 | Kruisende lichtstralen |
| 5 | Filters én spiegels samen |
| 6–8 | Schuifpuzzels (makkelijk → moeilijk) |

De keuze om filters **los** van spiegels te introduceren was een leerresultaat van playtests: spelers die filters en spiegels tegelijk kregen de kleurregels niet begrepen en willekeurig begonnen te gokken. Spelers waren meer bezig met de puzzels en gingen dus gokken welke combinatie een bepaalde lichtstraal zou maken. Door herhaling van de basisprincipes konden spelers leren en konden we later de complexiteit verhogen door beide mechanics tegelijk aan te bieden.

---

## Code Showcase

<img src="/assets/images/schuif-en-schijn-demo.gif" alt="Schuif en Schijn kleurfilter demo" class="project-demo-gif">

### 1. Subtractieve kleurfilters

Wanneer een laserstraal door een filter gaat, wordt de RGB-waarde vermenigvuldigd met het subtractieve CMY-masker van het filter. Een cyaanfilter laat groen en blauw door en blokkeert rood (`new Color(0, 1, 1)`). Als alle kleurkanalen naar nul dalen, is de straal volledig geabsorbeerd en stopt de laser.

```csharp
// LaserColorFilter.cs
private static Color SubtractiveMask(CMYColor c) => c switch
{
    CMYColor.Cyan    => new Color(0f, 1f, 1f),
    CMYColor.Magenta => new Color(1f, 0f, 1f),
    CMYColor.Yellow  => new Color(1f, 1f, 0f),
    _                => Color.white
};

public Color ApplyFilter(Color beamColor)
{
    Color mask = SubtractiveMask(filterColor);
    return new Color(
        beamColor.r * mask.r,
        beamColor.g * mask.g,
        beamColor.b * mask.b,
        beamColor.a);
}

public static bool IsAbsorbed(Color color)
    => color.r < 0.01f && color.g < 0.01f && color.b < 0.01f;
```

### 2. Oplaadbaar target met kleurcheck

`ReceiveLight` wordt elke frame aangeroepen door de laser. Als de kleur niet klopt, keert de methode direct terug. Zodra de timer `requiredSeconds` bereikt, meldt het target zichzelf bij de LevelManager. De `Update`-loop laat de timer langzaam leeglopen als er geen licht op valt, en werkt de visuele progress-ring bij.

```csharp
// LevelTarget.cs
private void Update()
{
    if (!_hasLight)
    {
        _timer -= decaySpeed * Time.deltaTime;
        _timer  = Mathf.Max(_timer, 0f);
    }

    if (_isActivated && _timer < requiredSeconds)
        _isActivated = false;

    _progressImage.fillAmount = FillProgress; // visuele feedback
    _hasLight = false;                        // reset elke frame
}

public void ReceiveLight(float deltaTime, Color laserColor)
{
    if (laserColor != LevelTargetColorUtility.ToUnityColor(targetColor))
        return;

    _hasLight = true;
    if (_isActivated) return;

    _timer += deltaTime;
    OnAnyTargetCharging?.Invoke(this); // event voor UI-feedback

    if (_timer >= requiredSeconds)
    {
        _timer       = requiredSeconds;
        _isActivated = true;
        LevelManager.Instance?.OnTargetActivated(this);
    }
}
```

### 3. Level Manager — targets registreren en win-conditie

Elk target registreert zichzelf bij de LevelManager via `OnEnable` en verwijdert zichzelf via `OnDisable`. Zodra een target oplaat en `OnTargetActivated` aanroept, controleert de LevelManager of **alle** bijgehouden targets actief zijn — pas dan wordt het level als voltooid beschouwd.

```csharp
// LevelManager.cs
public void RegisterTarget(LevelTarget target)
{
    if (target == null || _targets.Contains(target))
        return;
    _targets.Add(target);
}

public void UnregisterTarget(LevelTarget target)
{
    if (target == null) return;
    _targets.Remove(target);
}

public void OnTargetActivated(LevelTarget activatedTarget)
{
    if (_isLevelComplete || _targets.Count == 0)
        return;

    _lastActivatedTarget = activatedTarget;

    if (_targets.All(t => t != null && t.IsActivated))
    {
        _isLevelComplete = true;
        CompleteLevel();
    }
}
```

---


