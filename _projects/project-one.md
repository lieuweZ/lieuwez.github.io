---
layout: project
title: HexFront
category: MonoGame, JavaScript
image: /assets/images/HexFront.png
order: 1
---

[**Download HexFront on itch.io!**](https://nickdisp.itch.io/hexfront?secret=migH0h72DMa66YptyipiH7geRg)

<img src="/assets/images/GameplayHexfront.gif" alt="HexFront gameplay demo" class="project-demo-gif">

## Overview

HexFront is a tactical turn-based strategy game played on a hexagonal grid. You place building pieces to expand your territory, defend your base, and attack the opponent. Every piece has its own unique role. The main goal is to destroy the enemy's central building before yours goes down.

## Core Features

- Territory conquest with buildings and units
- Turn-based gameplay
- Resource management
- PvP multiplayer

**My contributions focused on:**
- Unit spawning and grid movement logic
- Real-time achievement system
- Database integration and Socket.IO networking

## Achievement System

![Achievement](/assets/images/Achievement.png)

The feature I am most proud of is the achievement system. To give players extra motivation, I built a system that encourages them to try out different strategies, like placing a specific number of units. Whenever an achievement is unlocked, a popup notification with a sound effect dynamically appears on the screen.

**What this code does:**
- `executePreparedQuery()` looks up the specific multiplayer room ID that the game is running in.
- The `if` check acts as a safeguard so we don't try to send notifications to non-existent rooms.
- `this._io.to(roomId).emit()` uses Socket.IO to instantly broadcast a live popup to the exact room the player is in.

It works smoothly in the background without interrupting gameplay. By hooking the achievement checks directly into the piece placement logic, we track player actions, use SQL to prevent duplicate unlocks, and deliver real-time feedback that makes the game feel more rewarding.


```javascript
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

