# JaruMoves

A tribute to **Army Moves** (Dinamic Software, 1986), following the style of
its Amiga conversion: four chained stages, a status panel along the bottom and
multi-layer backgrounds. Written entirely in JARU, with characters and vehicles
as sprites and the scenery as a tilemap.

**The game boots into demo mode and plays itself.** On Windows, `D` hands you
the controls and `D` again gives them back to the autopilot. On boards with no
buttons wired up there is no way out of the demo — which is exactly what you
want from a machine sitting in a shop window.

## The four stages

| # | Stage | How it plays |
|---|---|---|
| 1 | **The bridge** | The jeep rolls with the scroll, jumps the gaps in the deck and shoots down the helicopters dropping bombs and the jeeps coming head-on. Falling into the river is fatal. Long stretches carry a raised walkway with medkits. |
| 2 | **Helicopter** | Low flight at dusk. Fighters, helicopters and anti-aircraft batteries anchored to the ground. Clipping the treetops brings you down. |
| 3 | **The river** | The diver swims across between moored mines and human torpedoes. The harpoon takes them out at range; touching a mine kills instantly. |
| 4 | **The jungle** | On foot, camera following the soldier. Pits, suspended logs, soldiers and mortar nests, all the way to the enemy barracks where the documents are. |

## Controls

| Key | |
|---|---|
| Arrows | move / climb and descend |
| `Z` | fire |
| `X` | jump |
| `D` | take control / return to demo |
| `N` | skip to the next stage |
| `R` | back to the title screen |

On a generic ESP32: GPIO 39 up, 38 down, 37 fire, 36 jump, 35 advance. Boards
with no buttons wired up map nothing and stay in demo mode.

## Music

Six original pieces — one per stage on a loop, plus two fanfares that play
once. Nothing is transcribed from the original Army Moves theme; these are new
compositions written to suit each stage.

| Stage | Track | Length | Character |
|---|---|---|---|
| 1 | `Bridge` | 26.6 s | Military march in D minor, alternating kick and snare |
| 2 | `Air` | 21.5 s | E minor, faster, driving eighth-note bass and constant hi-hat |
| 3 | `River` | 24.6 s | Slow and airy, **no percussion**: long attack, deep bass |
| 4 | `Jungle` | 27.6 s | C minor, brass up front and a countermelody closing each phrase |
| — | `Clear` | 2.5 s | Stage-cleared fanfare: rising brass triad and a drum roll |
| — | `Victory` | 9.0 s | End of game: five bars in C major, brass, pad and arpeggio |

## Requirements

JARU IDE. Open `JaruMoves.jpr`, then Build and Run.
