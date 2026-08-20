# Ground handling model — research findings

Research for [issue #3](https://github.com/elmofromok/MoMadness/issues/3).

## Recommendation: custom raycast spring-damper, not `WheelCollider`

The decisive argument is not deprecation or performance. It is one structural
fact that hits this game's core mechanic directly:

> "No, it's by design. The WheelCollider always point downward with respect to
> the Rigidbody it's associated to."
> — Edy (Edy's Vehicle Physics / Vehicle Physics Pro), Unity Discussions,
> 18 March 2025, replying to a developer whose motorcycle wheels sank into the
> ground during wheelies and stoppies. https://discussions.unity.com/t/1617294

The suspension ray is cast along the parent Rigidbody's local −Y, not world down
and not the wheel's own orientation. A game built on wheelies, stoppies, whips
and backflips rotates that Rigidbody through large pitch and roll constantly.
A car never pitches 80 degrees. This bike will.

## Corrections to the ticket text

Two claims in issue #3 did not survive checking. Both were written into the
ticket from received wisdom rather than sources.

1. **`WheelCollider` is not deprecated.** It has a full five-page manual section
   for 6000.5, no deprecation notice, and no `[Obsolete]` on any API member.
   https://docs.unity3d.com/6000.5/Documentation/Manual/wheel-colliders.html
   It is *frozen*, not deprecated: backed by PhysX 4.1, and a Unity staff promise
   (26 July 2022) to replace the single ray with a suspension sweep remains
   unshipped roughly 3.5 years later.
2. **The `sidewaysFriction`-on-steep-slopes claim is unsubstantiated.** No primary
   source documents it. The real disqualifiers are the local-Y cast direction,
   the documented smooth-geometry requirement, and opaque PhysX state.

## Evidence against `WheelCollider` for this project

**Unity's own docs argue against it on this terrain.** From
https://docs.unity3d.com/6000.5/Documentation/Manual/wheel-colliders-introduction.html

- "PhysX casts the ray down the local Y-axis along the direction of suspension
  through the center of the wheel."
- Wheels "don't always smoothly roll up or step down variations in road level";
  on a step the wheel "is likely to clip and then 'pop' up".
- "The ground collision geometry must be as smooth as possible to ensure a
  smooth and accurate simulation."

That last line directly contradicts steep, uneven, procedurally generated terrain.
Critically, this buys nothing to give up: `WheelCollider` ground detection *is* a
single downward raycast, the same primitive you would write yourself.

**Friction is opaque and car-shaped.** Sideways friction's documented job is
"keeping the car oriented", which is the opposite of what a bike needs. Wheel
friction is "calculated separately from the rest of the physics engine, and
ignores standard Physics Material settings", so MM2-style per-surface grip
(ice, gravel, mud) is not available as a free feature.

**Opaque state works against ADR-0002.** "PhysX maintains internal state that is
not accessible by Unity." `WheelHit` exposes point, normal, slip, force and
direction, and nothing else. Wheel state cannot be snapshotted, serialised or
restored, which is what deterministic replay and rollback netcode require. There
is also an open, unanswered bug (Unity 6.x, Oct 2025) where destroying a vehicle
Rigidbody corrupts `WheelCollider` state across all remaining vehicles.
https://discussions.unity.com/t/wheel-collider-bug-in-unity-6-x/1688571

**Defaults are calibrated for a 1500 kg car.** Spring 35000 N/m, damper 4500,
wheel mass 20 kg, radius 0.5 m. A bike plus rider is roughly 190 kg.

## Honest counter-evidence

Naive raycast suspension goes unstable below roughly 50 Hz, where `WheelCollider`
stays stable down to 16 Hz, because PhysX solves all wheels together with caching
rather than each spring independently. Edy, May 2020:
https://discussions.unity.com/t/how-to-make-a-raycast-suspension-stable-like-built-in-spring/788747

Mitigations: keep fixed timestep at 0.02 or lower; compute damper velocity from
`Rigidbody.GetPointVelocity` projected onto the suspension axis rather than a
finite difference; clamp compression to `[0, travel]`; never let the spring pull
the chassis down; clamp damper force so it cannot reverse velocity in one step.

A 9 Feb 2025 thread shows a developer who, after anti-roll bars, non-linear curves
and multiple raycasts, still could not get the glide they wanted. Custom raycast
is not automatically smooth. Budget the tuning time.

## Implementation cost

| Scope | Reference | Size |
|---|---|---|
| Minimal raycast suspension | `hayden-donnelly/vehicle-physics` `Suspension.cs` (MIT) | ~35 lines |
| Plus drive, steer, lateral grip | same repo, `VehicleController.cs` | ~105 lines |
| Full production arcade vehicle | `sergeymakeev/arcadecarphysics` (MIT, 455 stars) | ~1,100 lines |

Estimate for a two-wheeler ground-contact layer: **250 to 400 lines**, plus one to
two weeks of tuning that `WheelCollider` would also demand, but more blindly.
Migration cost is zero: `Assets/` currently holds two tutorial scripts and no
gameplay code.

Core loop: raycast down `travel + radius`; `compression = travel − (hit.distance −
radius)`; `F = k·compression − c·springVelocity`; `AddForceAtPosition(up * F,
wheelPos)`.

Structural reference: Toyful Games, *Very Very Valet* car physics deep-dive,
8 July 2022. https://www.toyfulgames.com/blog/deep-dive-car-physics

**This serves the "assists as tunable floats" constraint directly.** With
`WheelCollider` the unassisted model lives inside PhysX; you can only observe slip
after the fact and bolt corrections on top. With custom raycast the unassisted
model *is* your code, and each assist is one term with a gain you can set to zero.

## What real projects use

Custom raycast dominates actively-maintained arcade vehicle frameworks (7 confirmed
by reading source), including `JustInvoke/Randomation-Vehicle-Physics` (932 stars),
`adrenak/tork` (447), and `sergeymakeev/arcadecarphysics` (455). Vehicle Physics Pro
shipped a component "completely replacing Unity's WheelCollider" in January 2024.
Unity's own experimental replacement package uses a convex mesh cast plus an
explicit spring-damper, though it is ECS/DOTS-only and therefore unavailable here.

`WheelCollider` still dominates the motorcycle-specific repos (5 found), and they
all complain. Read the split honestly: nobody has shipped a good public two-wheeler
on `WheelCollider`. The actively-maintained new repos are raycast; the stale ones
are `WheelCollider`.

Shipped commercial bike games on Unity: *Lonely Mountains: Downhill* used "a custom
bike physics system to achieve very tight and fun controls" (developer, 21 May
2017); *Descenders* confirms "custom tweaks to Unity's physics". Neither states
`WheelCollider` versus raycast specifically.

## World scale: 1 unit = 1 metre

Unity ties this to physics, not art: "Unity's physics and lighting systems expect
1 meter in the game world to be 1 unit in the imported Model file."
https://docs.unity3d.com/6000.0/Documentation/Manual/models-preparing.html

`DynamicsManager.asset` already has gravity −9.81, so metre scale is baked in, and
every scale-dependent default (contact offset 0.01, bounce 2, sleep 0.005) is
calibrated for it. Get exaggeration from launch velocity and input response, not
from bending scale or gravity.

### Bike dimensions (450cc motocross, 2026 model year)

| Quantity | Value | Note |
|---|---|---|
| Wheelbase | 1.48 m | YZ450F 1.476, CRF450R 1.481, KX450 1.481 |
| Seat height | 0.96 m | YZ450F 0.965, KTM 0.958 |
| Ground clearance | 0.34 m | 0.333 to 0.345 across four bikes |
| Overall L x W x H | 2.17 x 0.83 x 1.28 m | Yamaha spec sheet |
| Bike mass (wet) | 110 kg | YZ450F 110.2, CRF450R 112.9 |
| Rider mass (geared) | 80 kg | *estimate*: 70 kg body plus 10 kg gear |
| Front suspension travel | 0.31 m | Yamaha, Honda, KTM agree on 310 mm |
| Rear wheel travel | 0.30 m | 295 to 307 mm |
| Front wheel radius | 0.35 m | 80/100-21, *computed from tire geometry* |
| Rear wheel radius | 0.34 m | 120/80-19, *computed* |
| Wheel mass each | 12 to 15 kg | *estimate* |

Sources: https://www.yamahamotorsports.com/models/26-yz450f/specs and
https://www.ktm.com/en-us/models/motocross/4-stroke/2026-ktm-450-sx-f/technical-specifications.html

**Two things worth internalising:**

1. Suspension travel is roughly 21% of the wheelbase. A model that does not show
   the full 0.30 m of travel will not read as motocross.
2. **Bike to rider mass ratio is 1.38:1** (110:80). The rider is about 42% of system
   mass, so rider weight-shift genuinely moves the combined centre of mass. The
   Motocross Madness mechanic falls out of correct physics for free. Do not give
   wheels token 1 kg masses; that would push the ratio to 110:1 and past the legacy
   stability limit.

**Calibrating "exaggerated":** AMA Supercross defines a triple as a leap of 70 ft or
more (21.3 m), which is roughly 15.5 m/s launch, 1.6 s airtime, 3.1 m above the lip.
For heroic arcade air, target 2.5 to 4 s airtime and 8 to 15 m apex. Reachable with
real gravity at real scale purely via launch speed and takeoff angle.

## Recommended project-settings changes

Current values verified directly in the repo.

| Setting | Current | Recommended | Why |
|---|---|---|---|
| `m_EnableEnhancedDeterminism` | 0 | **1** | Required for same-machine determinism. Serves ADR-0002 at near-zero cost |
| `Fixed Timestep` | 0.02 | **0.0166 or 0.01** | 50 Hz is the documented edge for naive raycast suspension stability |
| `m_SolverType` | 0 (PGS) | evaluate **1 (TGS)** | Handles stiff joints better; relevant to the bike-rider joint |
| `m_DefaultMaxAngularSpeed` | 50 rad/s | raise if flips clamp | A backflip or whip may exceed 50 rad/s and be silently clamped |

Gravity, contact offset, sleep threshold and bounce threshold are correct for metre
scale. Leave them alone. Note that the terrain research recommends *lowering*
contact offset to 0.002–0.005 to reduce edge catching; reconcile the two before
changing it.

## Open questions

1. **One Rigidbody or three?** The reference implementations use a single chassis
   Rigidbody with visual-only wheels. This project already has two bodies (bike and
   rider). Whether wheels also get Rigidbodies interacts with issue #7.
2. **One raycast per wheel, or a fan?** A single ray reproduces the documented
   pop-up-on-steps artefact. A 3 to 5 ray fan or a spherecast smooths it cheaply,
   and Unity's own new package chose a shapecast. Worth prototyping early. The
   terrain research independently reaches the same conclusion.
3. **How does rider weight-shift couple to grip?** Physically the rider's centre of
   mass shift changes front and rear load split and therefore available grip.
   Modelling that emergently (correct, unpredictable) versus as an explicit assist
   float (tunable, on-brand) is a design call.

## Coverage gaps

Reddit was crawler-blocked, so the community read is Unity Discussions only. Asset
Store description bodies do not render, so *Arcade Bike Physics* and *Simple
Motorcycle Physics Pro* internals are unconfirmed. Top speed of a 450 MX bike is
unsourced (60 to 70 mph estimate).
