---
layout: project
title: Drink Mixer Match-3
category: MonoGame, JavaScript
image: /assets/images/Game_tweede__semester.png
order: 2
---
[**Download Drink Mixer Match-3 on itch.io!**](https://fuzesuga.itch.io/match-3-mixer)

<img src="/assets/images/SwipeDrink.gif" alt="Drink Mixer gameplay demo" class="project-demo-gif">

**A score-based puzzle game with grid mechanics where players mix falling ingredients to create drinks for customers.**

I implemented a custom system for falling tiles and ingredient movement, inspired by classic match-3 games but with a unique bartending twist.

## Gameplay

Players match ingredients on a grid to create drink recipes for waiting customers. Each customer has specific drink orders that need to be fulfilled within time limits or move constraints.

**Core Features:**
- Grid-based matching with falling ingredients
- Recipe system combining specific ingredients into drinks
- Customer orders with time-based challenges
- Multiple difficulty levels for different skill levels

## My Contributions

- Grid-based logic for placing and matching ingredients
- Falling animations and smooth movement between tiles

## Tile Gravity System

One of the most important systems in a match-3 game is the grid gravity and fall animation. When pieces are cleared, the board needs to update fluidly as new pieces fall into place.

**What this code does:**
- `dropTile()` is a recursive function that handles the game's gravity and cascading effect when ingredients are cleared.
- `tile.animateTo()` visually slides the tile down one space before seamlessly triggering the callback logic.
- Inside the callback, `this.swapTiles()` updates the internal grid state, and then recursively pulls down the tile above it (`this.dropTile(above)`).
- `this.summonNewTile()` calls that the column is refilled from the top.


```javascript
dropTile(pos, previousDrops = 0) {
    // Generate new tile at top if dropped above grid
    if (pos.y < 0) {
        this.summonNewTile(pos.x);
        return true;
    }

    const tile = this.getTile(pos);
    if (!tile || tile.isMoving()) {
        return false;
    }

    const below = pos.copy().add(new p5.Vector(0, 1));
    if (this.getTile(below) || below.y >= this.size.y) {
        return false;  // can't drop further
    }

    // Animate tile falling with duration based on game state
    tile.animateTo(
        this.gridPosToWorldPos(below),
        this.gameHolder.playerMoves > 0 ? this.tileMoveAnimationDuration : 0,
        tile => {
            this.swapTiles(pos, below);

            // Recursively drop more tiles (cascade effect)
            const droppedMore = this.dropTile(below, previousDrops + 1);

            if (!droppedMore) {
                this.gameHolder.tilesUpdated([below]);
            }

            // Refill from top and drop tiles above
            if (pos.y == 0) {
                this.summonNewTile(pos.x);
            } else {
                const above = pos.copy().sub(new p5.Vector(0, 1));
                this.dropTile(above);
            }
        }
    );

    return true;
}
```
