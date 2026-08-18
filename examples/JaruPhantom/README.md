# JaruPhantom

A single-screen platformer written in JARU, inspired by *Phantomas* (Dinamic,
1986). It is not a remake: all the artwork is original, drawn from scratch
under Spectrum rules — 8x8 blocks, two colours per cell, a 15-colour palette.

A thief robot lands on the **rooftop** and works its way **down** the tower,
throwing levers as it goes. Once all thirty are thrown the vault door at the
bottom gives way, and behind it is the loot. Falling down a floor is free;
climbing back up costs you the whole staircase, so picking the wrong floor
hurts.

You get one life. The battery only drains when a drone or a spike touches you
— there is no timer, the game is about not getting hit. Health pickups
scattered around the tower give some of it back.

## Two jumps, and that is the whole game

You choose a jump as you take off, and you have no control in the air.

| Jump | Height | Reach |
|---|---|---|
| High | 47.5 px | 29.25 px |
| Long | 16.5 px | 55.20 px |

Every obstacle in the tower is measured against those two numbers, and each
one has exactly **one** answer:

| Obstacle | |
|---|---|
| 5-cell pit (40 px) | long jump clears it, high jump falls short |
| 4-cell step (32 px) | high jump climbs it, long jump falls short |

Reading the obstacle and picking the right jump *before* you leave the ground
is the entire mechanic.

## Controls

| Key | |
|---|---|
| `O` / `P` or arrows | left / right |
| `Z` | **high** jump — gains height |
| `SHIFT` or `SPACE` | **long** jump — gains distance |
| `D` | hand control back to the autopilot |
| `C` | jump calibration bench |
| `R` | restart |

The game boots into autopilot, like an arcade attract mode, and **any game key
takes over**, so you do not need to know a single key before sitting down.
Press `D` to hand it back.

## What is in the tower

- Eight floors, each built around one idea: The Rooftop, The Pillars, The
  Chimney, The Corridor, The Floor Slabs, The Overhang, The Towers and
  The Crypt.
- Three biomes — mansion, caverns and crypts. Same drawing, different ink,
  the way a Spectrum attribute byte works.
- 30 levers that **stay thrown**, so coming back to a room shows you what you
  have already done. They are mounted on walls and consoles, not lying on the
  floor, as in the original.
- One drone per floor, spike pits, health pickups and the vault door.
- An autopilot that **clears the whole tower on its own** in about a minute.

## Requirements

JARU IDE. Open `JaruPhantom.jpr`, then Build and Run.
