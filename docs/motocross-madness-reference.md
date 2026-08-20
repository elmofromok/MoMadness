# Motocross Madness (1998) — Design Reference

Source material for the remake project. Most detailed mechanical specifics below come from Rainbow Studios' *Motocross Madness 2* (2000) manual, since no MM1 manual has survived online in full text — but MM2 is a direct, mechanically similar sequel by the same team, built on the same core (separate bike/rider physics), so it's the closest primary-source documentation we have. Differences between the two are noted where known.

## Credits & background

- Developer: Rainbow Studios (Phoenix, AZ) — later became known for the *MX vs. ATV* series
- Publisher: Microsoft
- Designer: Robb Rinard
- Programmers: Mark DeSimone, Glenn O'Bannon
- Artists: Brian Gillies, Kevin Riley
- Released: August 1998 (NA), Windows only
- Minimum spec: Pentium II 233MHz, 32MB RAM, Windows 98
- Sequel *Motocross Madness 2* released May 2000, same studio, expanded scope

## Core physics model — the part that matters most

- Bike and rider are modeled as **separate physics bodies**, not one rigid unit — this is the mechanical heart of the "feel"
- On the ground: throttle/brake/steer control the bike; the rider's weight subtly affects balance
- In the air: the player can independently move/rotate the rider (lean forward/back, cross over left/right) to correct the bike's landing angle — a badly-timed jump can be *saved* in the air, which is the core skill expression
- Terrain uses adaptive tire grip rather than full soft-body deformation — the ground reacts to contact but isn't simulated as deformable
- The physics engine deliberately exaggerates realism for "spectacular crashes" and huge air — contemporary reviews describe it as arcade-leaning, not simulation-accurate
- MM2 added: independent rider/bike motion was *upgraded* further, plus variable surface friction (ice, gravel, mud) — implying MM1's surface interaction was simpler/uniform by comparison

## Controls (from MM2 manual — MM1 was very likely near-identical given shared engine)

**On the ground:**

- Throttle: Up
- Brake: Down
- Steer left/right: Left/Right
- Clutch: hold a modifier key
- Gyro effect toggles for gas/brake (assist options)

**In the air:**

- Cross-over (bike rotation left/right): Left/Right
- Lean forward: one key
- Lean back: another key
- Stunts: held modifier + directional input, mapped to a stunt wheel

Force feedback was supported via DirectInput for compatible joysticks — worth considering rumble/haptic support in a remake for controller players.

## Game modes / events

**MM1 (1998):**

- Baja — open-terrain, waypoint-based racing with freedom to choose your own route
- Supercross — indoor stadium tracks
- National — outdoor motocross tracks
- Enduro
- Stunt Quarry — freeform trick arenas (5 different quarries)
- Career mode — earn money, unlock further races
- Over 30 indoor/outdoor tracks total
- 16 stunts, performed via button + directional combos, viewable from a dedicated stunt camera
- LAN / modem / serial-cable multiplayer

**MM2 (2000) added:**

- Pro-Circuit career mode (rookie → pro progression, sponsorships, repair costs)
- Tag mode (multiplayer — keep-away with a ball)
- Larger, more detailed environments (trailer park, jungle, open-pit mine, farm) populated with ambient vehicles, objects, and "Easter eggs"
- Over 40 tracks/environments
- Named stunts: Heel Clicker, Superman, Barney, Air Walk, Nac-Nac, Big Kahuna, Split X-Up, Cliff Hanger, Double Can-Can, Tail Grab, Cordoba, Superman Seat Grab, and others

## Customization & tools

- In-garage bike tuning: engine, suspension, tires
- Included track editor — build and share custom tracks, choose riders/bikes
- Multiple selectable riders and bikes

## Known signature quirks (worth deciding whether to keep)

- The "invisible slingshot": if a player climbed out of the intended play area (e.g. up an out-of-bounds cliff in Stunt mode), an invisible force would launch them back toward the map with a comedic sound effect, rather than using an invisible wall. Rainbow Studios reused this trick later in *ATV Offroad Fury*.
- Reviewers specifically called out the "bone-chilling" crash reactions as a highlight — crash feedback (audio + rider ragdoll behavior) was treated as a feature, not just a fail-state.

## Reception (for context on what worked)

- GameRankings aggregate: 87% (MM1)
- PC Gamer US named it Racing Game of the Year 1998; Computer Games Strategy Plus gave the same award
- Common praise across reviews: graphics-for-its-time, physics/air-time feel, replayability of open Baja terrain
- Common criticism: repetitive soundtrack, Supercross mode difficulty spikes

## Open questions to resolve before/during prototyping

- Exact numeric feel (turn rate, gravity scale, air-control sensitivity) isn't published anywhere — this will need to be tuned by feel, not sourced
- Whether to replicate MM1's simpler surface model or start closer to MM2's multi-surface friction
- Whether "16 stunts" from MM1 or the larger MM2 stunt list is the better scope target for a first playable
