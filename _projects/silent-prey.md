---
layout: project
title: Silent Prey
category: Unity, C#
image: /assets/images/Silent-Prey-MainMenu.png
order: 0
---
[**Play Silent Prey on itch.io!**](https://lucasvd-hva.itch.io/silent-prey)

<img src="/assets/images/Silent-Prey-Gameplay.png" alt="Silent Prey gameplay demo" class="project-demo-gif project-demo-gif-wide">
    
## Overview

Silent Prey is a 3D stealth-horror game made in Unity. The player explores dark corridors, avoids a boss that reacts to sight and sound, and looks for safe moments to strike from behind. Sound-based distraction is a core loop: by throwing cans, the player can redirect the boss, open a route, and create a short attack window.

## Core Features

- Enemy AI boss that tracks you by reacting to sound
- Enemy that builds traps to catch the player
- Sound-based distraction mechanics, like throwing cans to create safe paths
- Atmospheric horror environment designed for stealth and exploration


**My contributions focused on:**
- Can throw gameplay mechanic
- UI feedback for player actions
- Atmosphere and visuals
## Throw Mechanic

The part I am most proud of is the can throw mechanic. Holding right mouse button charges power over time between 0 and 1 seconds. Then releasing launches the can forward from the player camera direction. It shows a trajectory line where the can is going to land so the player knows has more control over it.


<img src="/assets/images/Can Throwing.gif" alt="Can throwing mechanic demo" class="project-demo-gif project-demo-gif-wide">

**Code:**

This logic handles distance- and raycast-based visibility for distractions, ensuring the boss only reacts if the sound is within range and not blocked by walls. The state machine follows a strict priority: the player always takes precedence over sound distractions.

**Key Parameters:**
- **`_canDetectionRange`**: Maximum radius for detecting sound distractions.
- **`_detectionMask`**: Defines which layers (e.g., Environment) block the boss’s line of sight.
- **`_eyeOffset`**: Offsets the raycast origin from the pivot to the boss’s eye level.

If the player is detected, the boss immediately enters `ChaseState`. Otherwise, reaching the distraction point within 3 units triggers a return to `PatrolState`.

```csharp
// CanThrower.cs
public class CanLandingDetector : MonoBehaviour
{
    [SerializeField] private float _canDetectionRange = 20f;
    [SerializeField] private LayerMask _detectionMask;
    [SerializeField] private Vector3 _eyeOffset = new Vector3(0f, 1.7f, 0f);

    // Returns true only if the can is close enough AND not blocked by geometry
    public bool IsCanInSight(Vector3 canPosition, Transform canTransform)
    {
        float distance = Vector3.Distance(transform.position, canPosition);
        if (distance > _canDetectionRange) return false;

        Vector3 eyePos = transform.position + _eyeOffset;
        Vector3 dir    = (canPosition - eyePos).normalized;

        // Raycast from the boss' eyes to the can
        if (Physics.Raycast(eyePos, dir, out RaycastHit hit, distance, _detectionMask))
        {
            // If we hit something else first, the can is hidden behind it
            return false;
        }
        return true;
    }
}
private void CheckStateTransitions()
{
    // Check for player in sight (higher priority)
    if (_playerInSightDetector.IsPlayerInSight())
    {
        p_stateMachine.GoToState<ChaseState>();
        return;
    }
    // Check if reached the can
    if (Vector3.Distance(transform.position, _canLandingPos) <= 3f)
    {
        p_stateMachine.GoToState<PatrolState>();
        return;
    }
}
```
