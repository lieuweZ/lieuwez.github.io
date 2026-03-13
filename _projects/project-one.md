---
layout: project
title: HexFront
category: MonoGame, JavaScript
image: /assets/images/HexFront.png
order: 1
---

<img src="/assets/images/GameplayHexfront.gif" alt="HexFront gameplay demo" class="project-demo-gif">

## Overview

This is a tactical strategy game played on a hexagonal grid. Place unique building pieces to expand your control, defend your territory, and launch attacks. Each piece has a different role—choose the right ones to gain the upper hand. Your ultimate goal: destroy the enemy’s central building before they destroy yours.

## Core Features

- Territory conquest with buildings and unit
- turn based
- Resource Management
- PVP

## My Contributions


**Unit spawns on the hex grid, and moves.**

![Unit](/assets/images/Unit.png)

**Achievement system**

The feature makes it possible to get achievements. many games have achievements, usually to ensure that a player gets more motivation. When you achieve an achievement, a popup with a sound will appear at the bottom right of the screen. 

The achievements are based on amount of units placed. The player is incentivized to try other strategies.

![Achievement](/assets/images/Achievement.png)

**Code**

• **Real-time achievement tracking** - Monitors player actions during gameplay and unlocks achievements when milestones are reached (e.g., placing 10 units, building 5 resource collectors)

• **Database integration** - Loads achievement definitions from database, tracks player progress with SQL queries, prevents duplicate unlocks using ON DUPLICATE KEY UPDATE

• **Socket.IO notification**s - Instantly broadcasts achievement unlocks to player.

• **Event-driven architecture** - Integrates with game's piece placement system, automatically checks achievements whenever players perform actions

```csharp
    async notifyAchievementUnlocked(gameId, playerName, achievement) {
        // Find the room this game belongs to
        const gameResult = await this._databaseConnector.executePreparedQuery(
            "SELECT room_id FROM games WHERE game_id = ?",
            [gameId]
        );
        
        if (gameResult.rows.length === 0) return;
        
        const roomId = gameResult.rows[0].room_id;
        
        // Emit achievement notification to the specific player
        this._io.to(roomId).emit("achievement unlocked", {
            playerName: playerName,
            achievement: {
                id: achievement.achievement_id,
                key: achievement.achievement_key,
                name: achievement.name,
                description: achievement.description,
                points: achievement.points,
                category: achievement.category
            }
        });
    }
```
