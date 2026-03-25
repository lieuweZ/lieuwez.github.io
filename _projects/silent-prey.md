---
layout: project
title: Silent Prey
category: Unity, C#
image: /assets/images/Silent-Prey.png
order: 0
---
[**Play Silent Prey on itch.io!**](https://lucasvd-hva.itch.io/silent-prey)

<img src="/assets/images/Silent-Prey-Gameplay.png" alt="Silent Prey gameplay demo" class="project-demo-gif project-demo-gif-wide">
    
## Overview

Silent Prey is a 3D stealth-horror game made in Unity. The player explores dark corridors, avoids a boss that reacts to sight and sound, and looks for safe moments to strike from behind. Sound-based distraction is a core loop: by throwing cans, the player can redirect the boss, open a route, and create a short attack window.

## Core Features

- 3D stealth, horror and exploration game
- Enemy AI boss that reacts to sound
- Enemy that builds traps to catch the player
- Sound based distraction mechanics like throwing cans

**My contributions focused on:**
- Map atmosphere and visual setup
- Can throw gameplay mechanic
- UI feedback for player actions

## Throw Mechanic

The part I am most proud of is the can throw mechanic. Holding right mouse button charges power over time, and releasing launches the can forward from the player camera direction. This gives intentional control: a short hold for close distractions, and a long hold for deeper baits.


<img src="/assets/images/Can Throwing.gif" alt="Can throwing mechanic demo" class="project-demo-gif project-demo-gif-wide">

```csharp
// CanThrower.cs
private void UpdateCharge()
{
    float chargeTime = Time.time - _chargeStartTime;
    _currentCharge = Mathf.Clamp01(chargeTime / _maxChargeTime);
}

private Vector3 CalculateThrowVelocity()
{
    float throwForce = Mathf.Lerp(_minThrowForce, _maxThrowForce, _currentCharge);
    Vector3 direction = _playerCamera.transform.forward;
    return direction * throwForce;
}

private void ThrowCan()
{
    Vector3 velocity = CalculateThrowVelocity();
    CanProjectile projectile = Instantiate(_canPrefab, _throwPoint.position, Quaternion.identity)
        .GetComponent<CanProjectile>();
    projectile.Launch(velocity);
}
```

What this code does:
- `UpdateCharge()` converts hold-time into a normalized value between `0` and `1`.
- `CalculateThrowVelocity()` maps that value to a force range and creates a velocity vector.
- `ThrowCan()` spawns the projectile at the throw point and applies launch velocity.

It is predictable for players, because force scales linearly with charge time. It keeps responsibilities separated charge, velocity calculation and spawn/launch, which makes it easier to debug and extend. This mechanic directly supports the stealth loop: the player controls where sound happens, pulls the boss away from key paths, and creates a safe window to move or attack.
