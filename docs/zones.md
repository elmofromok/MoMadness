# Zones and their content palettes

Recorded for [#6, Define the base game's zones and their content palettes](https://github.com/elmofromok/MoMadness/issues/6).
Settled by grilling in November 2026. No zone has been built or ridden yet, so
every number here is a starting target. Expect the first zone to move them.

## What this records

The base game ships two zones: a mountain forest and an open-pit mine. The mine
gets built first. Each zone holds about 40 to 50 placements, roughly one find
every 30 seconds of riding. The forest leans toward vignettes and the mine
toward playground features.

## What a zone is

A zone is a separate, self-contained place. It is not a region of one connected
world. This matches the existing [`CONTEXT.md`](../CONTEXT.md) entry ("entered
on its own"). A connected world would add terrain, streaming and transition
work, and discovery doesn't need it.

How the player gets into a zone (a menu, a map screen, something else) is still
open.

## How many: two

Two zones, picked over three or four. Each zone is roughly 3 to 5 km across.
That size comes from the 4 to 5 minute crossing time at 40 to 60 km/h. Each
zone also needs its own playground features, vignettes and asset-store art set,
and the work is solo to a production-quality bar. Two zones that are full beat
four that are thin.

The two themes were picked to differ as much as possible. They differ in where
the playground features come from (shaped land in the forest, built objects in
the mine), in how they look (green against ochre, dense against open), and in
the playground-to-vignette ratio. Between them they exercise both halves of the
pipeline from [#5](https://github.com/elmofromok/MoMadness/issues/5): sculpted
terrain and placed mesh.

## Mountain forest

The zone for looking around. Target ratio is about 70/30 features to
vignettes, roughly 32 features and 13 vignettes.

**Playground features**

- Ridgeline rollers, a run of humps along a spine
- A creek bed with cut banks to hop in and out of
- Logging road switchbacks with natural berms
- Fallen logs and log decks used as ramps
- A clear-cut slope full of stumps, steep and rough
- A cliff drop into a lake or clearing

**Vignettes**

- An abandoned cabin with smoke still rising from the chimney
- A small plane wreck hung up in the treetops
- A hidden waterfall reached only through dense trees
- A fire lookout tower on the highest peak, with the zone's best view. It is
  also a climb, so it counts as both.
- An empty campsite with the fire still burning and the tent zipped shut
- A large unexplained shape seen only at a distance, gone when you get there

The ratio needs about 13 vignettes and the list has six. The other seven or so
are small one-off moments, like a bear in a dumpster or a deer that bolts. They
aren't listed yet.

**Cut:** a timber trestle bridge and a dam with a rideable spillway. Both are
large bespoke structures, the most expensive content there is, and the mine
already proves the built-structure path. The forest relies on terrain and small
props.

**Early checks for when the forest gets built:** vegetation performance at zone
scale, and the cost of authoring its larger vignette count.

## Open-pit mine

The zone for riding. Target ratio is about 85/15, roughly 38 features and 7
vignettes. The six vignettes listed nearly cover it.

**Playground features**

- Benched pit walls, stepped terraces with drops and gaps between benches
- A spiral haul road down the pit with steep banked turns
- Spoil and tailings piles, loose dirt mounds that work as jumps
- A conveyor gantry, a long inclined ramp to ride up and launch off
- Parked haul trucks, with their dump beds used as ramps
- A loading hopper or chute to drop through
- Crusher and processing building roofs, linked by gaps

**Vignettes**

- A parked dragline excavator with its bucket on the ground, big enough to ride
  under
- A haul truck with its door open and the engine idling, no driver
- The break shack, with a radio still playing
- An old timbered tunnel entrance in a pit wall, boarded up, from workings older
  than the pit
- Something odd half dug out at the bottom of the pit
- A blast zone with warning signs and a siren pole

**Static for now.** Nothing in the mine moves. A moving ore train was cut,
because kinematic bodies hitting the Bike and Rider complicate crashes
([#9](https://github.com/elmofromok/MoMadness/issues/9)) and determinism
([ADR-0002](adr/0002-physics-stays-deterministic-without-multiplayer.md)). The
haul trucks are props, not vehicles.

## Density

About one find every 30 seconds of riding, which works out to roughly 40 to 50
placements per zone. The aim is for something to usually be visible on the
horizon to ride toward, without any UI.

One placement means one instance from a palette: one spoil-pile jump, one
cabin. Scattered repeats like loose fallen logs don't count one by one.

If a zone costs too much at this density, the current thinking is to cut
density before shrinking the zone.

## Build order: mine first

The first zone has two jobs. It puts the ride mechanic (air-saves, the gyro
effect, landings) onto real terrain, and it shows what a finished zone costs.
The mine suits both:

- Its features are clean, readable shapes, good for judging jumps and landings.
- It has open sightlines and little foliage, so no tree collisions and no
  vegetation performance cost yet.
- It needs about 7 vignettes against the forest's 13, so it reaches finished
  quality sooner.

The forest's risks are real. They are early checks for the forest, not reasons
to build it first.

## Still open

- How the player enters a zone.
- The forest's extra small vignettes, about seven of them.
- Zone edges and out-of-bounds handling (the reference doc's "invisible
  slingshot").
- Whether two zones holds once the mine's real cost is known. The mine might
  show that one zone is the honest count, or that a third is affordable.
- Every ratio and count above, until a zone has actually been ridden.
