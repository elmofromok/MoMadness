# Authoring and placing playground features and vignettes

Recorded for [#11, Decide how playground features and vignettes get authored and placed](https://github.com/elmofromok/MoMadness/issues/11).
Settled by grilling in November 2026. None of this tooling exists yet and no
zone has been built, so treat it as the plan for the mine, to be revised once
the mine shows what it actually costs.

## What this records

Features and vignettes are prefabs placed by hand in the zone scene. One
editor helper does the terrain chores for each feature. Mass-produced objects
are reused and landmarks are one-offs. A jump is checked three ways: an arc
preview while placing it, riding it, and later an automated physics test. A
zone counts as finished when it meets a count, a gap check and a ride check.

The base game is scoped as an MVP. Where something was put off on purpose,
this doc says so.

## Placement: by hand, with one helper

Every playground feature and vignette is a prefab instance placed by hand in
the zone scene. There is no data asset listing placements and no tool that
spawns them. At 40 to 50 placements per zone
([zones](zones.md#density)), a data layer would be a second source of truth
that can drift from the scene, and spawning by rule works against an authored
sandbox.

The terrain chores are where tooling pays. Under the generate-then-edit
workflow from [#5](terrain-generation-research.md#the-generate-then-edit-workflow),
each mesh feature needs three steps: a Mesh Stamp with Behaviour Min to
conform the heightmap, a Slope Flatten on the run-up and landing, and a
terrain hole painted under the footprint. That comes to about 150 steps per
zone. A missed hole leaves the double-contact seam, and that only shows when
someone rides over it. So an editor helper does all three for the selected
feature, and it can redo them after a terrain regenerate.

### Features that are shaped land

Some features are carved into the ground and have no prefab. In the mine
these are the benched pit walls and the spiral haul road. In the forest they
are the ridgeline rollers, the creek bed and the cliff drop. Each gets a
**marker**: an empty GameObject at the spot that matters, holding the lip
position and the design speed range. Their heightmap edits stay in the delta
script from #5.

A long feature can have several markers, one at each jump or landing that
matters. It still counts as one placement, following the definition in
[zones](zones.md#density). The markers are there so that the arc preview, the
count and the gap map see shaped land the same way they see a prefab.

## Reuse: split by what the thing is

- **Reused prefabs** for things that are mass-produced in the real world: haul
  trucks, spoil piles, roof sections, log decks. A real mine has a dozen
  identical trucks, and the player reads them as a fleet.
- **One-offs** for landmarks and shaped land: the spiral haul road, the benched
  pit walls, the conveyor gantry, the cliff drop. These are what a player
  remembers a zone by, so they are where repetition would show.

**Rideable geometry gets uniform scale, rotation and prefab variants only.**
Non-uniform scale changes the takeoff and landing angles, so a stretched
spoil pile can stop landing even though it looks fine. A different shape gets
a new mesh. With that rule, a jump that has been validated stays valid
wherever it is placed.

## Validation: arc preview, riding, then a harness

Each rideable prefab and each marker carries a **design speed** range: the
speeds the feature is built to be hit at. All three checks read from it.

1. **Arc preview, while placing.** The helper draws the ballistic arc off the
   lip at the low and high ends of the design speed range, so you can see
   where the Bike comes down against the landing slope. It needs no Bike and
   runs no physics, so it is available from day one.
2. **Riding, as the final judge.** Only riding says whether a jump is fun.
3. **A PlayMode harness, once the Bike exists.** It launches the real Bike
   with no input, in Hard mode, at each feature's design speeds, and asserts
   that it lands upright. The physics is deterministic-friendly
   ([ADR-0002](adr/0002-physics-stays-deterministic-without-multiplayer.md)),
   so the result is repeatable. Its job is catching regressions: when the gyro
   or engine tuning from [#17](https://github.com/elmofromok/MoMadness/issues/17)
   and [#18](https://github.com/elmofromok/MoMadness/issues/18) changes, some
   of the mine's roughly 38 jumps may stop landing, and nothing else would
   show it.

A jump that lands only with an Air-save is allowed. The harness can't pass it
with no input, so it gets flagged as one, not failed.

## Vignettes: plain prefabs for now

For the MVP, a vignette is a prefab built only from Unity's own components.
The idling haul truck is a looping engine AudioSource and an exhaust particle
system. The break-shack radio is a looping clip. The cabin smoke and the
campfire are particle systems. The dragline, the boarded-up tunnel and the
find at the bottom of the pit are meshes. None of them needs a custom script.

The expected next step is a shared `Vignette` component on every vignette.
It isn't built now because it would have no job yet. Once telegraphing is
decided (still unspecified on the
[map](https://github.com/elmofromok/MoMadness/issues/1)), that component is
the likely place for it. Adding it to about 50 prefabs later is cheap.

**Nothing a vignette does records that the player found it.** Discovery
tracking is out of scope
([ADR-0001](adr/0001-racing-is-cut-from-the-base-game.md)), and a "seen" flag
tends to grow into a counter.

## Dynamic vignettes: phase 2

Vignettes that move or react to the player wait until the core ride is
working. That covers the forest's shape that disappears as you approach, the
deer that bolts, a bear in a dumpster, and the helicopter on a timed path from
the ticket. The mine has none, since [#6](zones.md#open-pit-mine) made it fully
static, so the mine can be built and finished without them.

One constraint is carried forward provisionally from #6's cut of the ore
train. **Whatever moves should never collide with the Bike or Rider.** Its
moving parts either have no colliders or sit on a layer that ignores both.
That keeps a dynamic vignette purely visual and out of crash handling
([#9](https://github.com/elmofromok/MoMadness/issues/9)) and determinism.
Phase 2 decides whether these share a system or stay one-offs.

## A finished zone

A zone is finished when all four hold:

1. **Count.** 40 to 50 placements, near the zone's ratio (about 85/15 for the
   mine, 70/30 for the forest).
2. **Gap map.** An editor overlay in the helper shades every spot on the
   rideable ground more than about 400 m from the nearest placement or
   marker. Nothing on the rideable ground is shaded. The 400 m comes from 30
   seconds of riding at 40 to 60 km/h, which is roughly 330 to 500 m.
3. **Validation.** Every playground feature has passed the arc preview and
   been ridden, and has passed the harness once the harness exists.
4. **Ride check.** On several crossings in different directions, you never
   ride 30 seconds without something to steer toward.

The gap map finds the empty areas, and only riding shows whether something is
visible on the horizon. If the gap map costs too much for the MVP, it is the
first part to cut. Checks 1, 3 and 4 still stand without it.

## Still open

- The design speed ranges themselves, per feature.
- The gap-map threshold, which assumes the 40 to 60 km/h from #6 and should
  move if riding speeds turn out different.
- What the harness counts as landing upright.
- Whether dynamic vignettes get a shared system in phase 2, or stay one-offs.
- When the shared `Vignette` component gets built, which waits on
  telegraphing.
