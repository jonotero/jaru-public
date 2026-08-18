# JaruQuest

A NES-style top-down adventure written in JARU. A 4x4-screen overworld, a 4x3
dungeon with a key, a locked door and a boss, four kinds of enemy, a sword,
hearts and pickups. All the artwork is original.

It is inspired by NES action-RPGs rather than copied from any of them: the
hero wears blue, the enemies are a *spitter*, a *hopper*, an *ogre* and a
*bat*, and the prize at the end is a gem.

**The game plays itself when nobody touches the controls.** That is what you
see on a board with no gamepad wired up, and on Windows it takes over after 12
seconds of inactivity, like an arcade demo mode.

## Controls

| Key | |
|---|---|
| Arrows | move |
| `Z` | attack |
| `ENTER` | start |

## What is in it

- Two worlds — overworld and dungeon — with camera-sliding screen transitions
  and passages between them.
- Top-down movement resolved axis by axis, with corner sliding so you do not
  snag on doorways.
- A sword that cuts bushes and deflects projectiles, contact damage, knockback,
  invulnerability frames and death.
- Four enemy behaviours, a boss, projectiles, pickups, and a key that opens the
  locked door.
- Coins dropped by enemies and hidden on the screens that sit off the main
  route.
- Four music tracks — title, gameplay, game over and victory — arranged across
  five channels, following the NES 2A03 voice split plus the extra tone channel
  JARU provides.
- An autopilot that navigates the map, dodges projectiles, fights what gets in
  the way, collects the key, opens the locked door, kills the boss and takes
  the prize.

## Requirements

JARU IDE. Open `JaruQuest.jpr`, then Build and Run.

Build the project from the IDE at least once before running it: the music is
compiled from the tracker sources as part of the build. Without that step the
game runs, but silent.
