---
layout: project
title: NEMO Slide & Shine
category: Unity, C#
image: /assets/images/schuif-en-schijn.png
image_contain: true
order: 0
---
[**Play Slide & Shine on itch.io!**](https://swzwij.itch.io/schuif-en-schijn)

[📄 Read the full process report (PDF)](/assets/Proces-Schuif-en-Schijn.pdf)



<img src="/assets/images/NEMO%20Gameplay.gif" alt="NEMO gameplay demo" class="project-demo-gif project-demo-gif-wide">

<img src="/assets/images/schuif-en-schijn-gameplay.png" alt="Slide & Shine gameplay demo" class="project-demo-gif project-demo-gif-wide">

## Overview

Slide & Shine is a 2D puzzle game developed in Unity for the NEMO Science Museum. In this game, players learn through play about the principles of light reflection and subtractive colour mixing. By rotating mirrors and sliding filters into the right positions, players guide laser beams towards end targets and activate them with the required colour.

The project was created by four game development students from the Amsterdam University of Applied Sciences as part of an assignment for an exhibition about game development at the NEMO Science Museum. The team went through the full development process, giving visitors an interactive look behind the scenes of game design and software engineering.

## Core Features

- **Dynamic Laser System**: Laser beams that reflect off mirrors in real time and are blocked by objects.
- **CMY Colour Filters**: Colour filters that apply subtractive colour mixing (Cyan, Magenta, Yellow) to passing laser light. White light contains all colours — a filter absorbs part of the spectrum and lets the rest through. A magenta filter removes green, a yellow filter removes blue; stack those two together and you're left with red.

  If we had used primary RGB filters instead, this puzzle mechanic would disappear: an RGB filter only lets one colour through and immediately blocks the other two. Stacking two different RGB filters would therefore block the entire spectrum at once. The CMY model provides exactly the structure needed — every combination of two filters still yields a usable colour, making meaningful puzzles possible. This contrasts with additive colour mixing (RGB), where colours are added together: red + green = yellow, all three together = white.
- **Chargeable Targets**: End targets that only charge when hit by the correct colour of laser light, and slowly discharge when no light is present.
- **Level Management**: A centralised system that manages player progress and dynamically loads new levels. Players must complete 2 levels at a given difficulty before the difficulty increases.

**My contributions focused on:**
- Designing and implementing the charging system for the **end targets** (`LevelTarget.cs` and `LevelTargetColor.cs`).
- Implementing the **subtractive colour filters** (`LaserColorFilter.cs`) to change laser beam colours based on RGB/CMY logic.
- Co-developing the **Level Manager** (`LevelManager.cs`) for registering targets and checking win conditions.
- Analysing how children learn and how this can best be applied in game design. By studying levels and progression, the structure was made suitable for a broad audience with varying levels of understanding, ensuring everyone can learn something from the game in their own way.

---

## Level Progression

The level structure follows **Kishōtenketsu**: an East Asian narrative form with four phases, also applied in level design. Introduction (*Ki*) — mirrors are introduced without distraction. Development (*Shō*) — filters are explored separately. Twist (*Ten*) — the two mechanics are combined, creating a new problem without introducing a new rule. Resolution (*Ketsu*) — the slide puzzles bring everything together in increasing complexity.

| Level | Description |
|-------|-------------|
| 1 | Mirrors only |
| 2 | Filters only |
| 3 | No overlapping beams |
| 4 | Crossing light beams |
| 5 | Filters and mirrors together |
| 6–8 | Slide puzzles (easy → hard) |

The decision to introduce filters **separately** from mirrors was a lesson learned from playtests: players who received both at the same time did not understand the colour rules and started guessing randomly. Players were more focused on solving puzzles and resorted to trial and error to figure out which combination would produce a given beam colour. By reinforcing the basic principles through repetition, players could learn them properly before we later raised the complexity by combining both mechanics.

---

## Code Showcase

<img src="/assets/images/schuif-en-schijn-demo.gif" alt="Slide & Shine colour filter demo" class="project-demo-gif">

### 1. Subtractive Colour Filters

When a laser beam passes through a filter, the RGB value is multiplied by the filter's subtractive CMY mask. A cyan filter lets green and blue through while blocking red (`new Color(0, 1, 1)`). If all colour channels drop to zero, the beam is fully absorbed and the laser stops.

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

### 2. Chargeable Target with Colour Check

`ReceiveLight` is called every frame by the laser. If the colour does not match, the method returns immediately. Once the timer reaches `requiredSeconds`, the target reports itself to the LevelManager. The `Update` loop gradually drains the timer when no light is hitting it, and updates the visual progress ring.

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

    _progressImage.fillAmount = FillProgress; // visual feedback
    _hasLight = false;                        // reset every frame
}

public void ReceiveLight(float deltaTime, Color laserColor)
{
    if (laserColor != LevelTargetColorUtility.ToUnityColor(targetColor))
        return;

    _hasLight = true;
    if (_isActivated) return;

    _timer += deltaTime;
    OnAnyTargetCharging?.Invoke(this); // event for UI feedback

    if (_timer >= requiredSeconds)
    {
        _timer       = requiredSeconds;
        _isActivated = true;
        LevelManager.Instance?.OnTargetActivated(this);
    }
}
```

### 3. Level Manager — Registering Targets and Win Condition

Each target registers itself with the LevelManager via `OnEnable` and removes itself via `OnDisable`. As soon as a target charges up and calls `OnTargetActivated`, the LevelManager checks whether **all** tracked targets are active — only then is the level considered complete.

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


