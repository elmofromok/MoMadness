# The engine model

The Bike has a real engine model driving the rear wheel: a torque curve, five
gears, a clutch with slip, a rev limiter, engine braking, and a crank whose
spin counts in the Gyro effect. Its numbers start from a real 450cc four-stroke
and live in an engine asset.

[ADR-0004](0004-the-gyro-effect-turns-the-bike-in-the-air.md) asked for this
model because the engine caps throttle's nose-up authority in the air. This
records what was settled by grilling in
[issue #18](https://github.com/elmofromok/MoMadness/issues/18). Nothing here has
been ridden, and it is expected to change once it has. The source figures are in
[engine-model-research.md](../engine-model-research.md).

## What the first pass builds

- **Every engine input enters the model as a value from 0 to 1.** Throttle,
  both brakes and the clutch. A trigger or a joystick throttle lever sends the
  value directly, and a key or button is ramped to it. The model never knows
  which device sent it. Which device sends what is for
  [#8](https://github.com/elmofromok/MoMadness/issues/8).
- **The torque curve is a point table** of about 6 to 10 RPM/torque points with
  linear interpolation, plus a single torque multiplier for arcade
  exaggeration. It starts from real 450 proportions: roughly 47 Nm peak at the
  rear wheel near 7,000 RPM, peak power near 9,300 RPM, a broad flat plateau.
  A flat peak also suits keyboard players, whose throttle mostly sits at 0 or 1.
- **The numbers live in an engine asset**, a ScriptableObject holding the torque
  table, the engine-braking curve, the gear ratios, idle and limiter RPM, the
  limiter taper width, crank inertia and shift time. A second engine would be a
  second asset. The one-Bike decision stands; this only keeps that door open.
- **Five gears, on the CRF450R's real ratios** as a starting point. See the open
  question on speeds below.
- **The clutch has a slip range.** Engagement runs from 0 to 1 and caps the
  torque it can pass. The player's clutch is a digital button that ramps
  engagement over a short, tunable time, so #8 does not need a third analog
  axis. In the air, holding it takes engagement to 0 and the rear wheel coasts.
- **The clutch runs on lock/slip states.** While slipping, crank and rear wheel
  are two bodies with friction torque between them. Once their speeds match and
  the torque fits within clutch capacity, they lock and are integrated as one
  body with combined inertia. A stiff friction link between two bodies would
  oscillate at a 50 Hz physics step; locking avoids it at any step rate.
- **The engine cannot stall.** An idle governor holds about 2,000 RPM and the
  clutch slips instead. A stall would need a restart input that no device has
  room for, and the sandbox
  ([ADR-0001](0001-racing-is-cut-from-the-base-game.md)) has nothing for a
  stall to penalize.
- **The rev limiter is a soft taper.** Torque falls linearly to zero over the
  last 200 to 300 RPM. The limiter stutter is played in the audio only. A hard
  ignition cut would shake an airborne Bike through the gyro at a frequency set
  by the physics timestep.
- **Closing the throttle brakes the engine.** A closed-throttle drag curve by
  RPM, blended by throttle position, with a strength multiplier. With the clutch
  engaged in the air, chopping the throttle slows the rear wheel and drops the
  nose a little.
- **An upshift cuts torque for about 0.1 s**, a value in the engine asset. Not
  grilled; a default to revisit.
- **There is no added throttle lag.** Any smoothing for keyboard belongs to #8.

## The crank counts in the Gyro effect

The crank, flywheel and generator rotor turn the same way as the wheels on
every production 450 the research found. Their angular momentum is added to the
wheels' in the same vector sum ADR-0004 uses, at real estimated inertia, with a
strength float of its own. Its roll and yaw terms use the existing precession
float.

At 9,000 RPM the crank carries very roughly a third of the rear wheel's angular
momentum at 15 m/s, and it changes far faster. So blipping the throttle with
the clutch pulled lifts the nose slightly even while the wheel coasts. The
engine model needs crank inertia anyway, to make RPM rise and fall believably.

## Shifting

The engine model always has real gears. An **Auto-shifter** sits on top by
default and calls the same shift function a manual shift button would. Manual
shifting is available. **The Auto-shifter does not shift while airborne**, so
the gear the Bike leaves the lip in is the gear it has in the air, and throttle
authority there is predictable. The auto-clutch for pulling away and stopping
belongs with it.

The Auto-shifter is a **Control preference**, not an Assist. It changes who
presses the button, not how the Bike behaves, so Hard mode can be played with
it on. This keeps Hard mode playable on a gamepad layout that may have no room
for shift buttons.

## Determinism

Engine state is plain values: RPM, gear, clutch engagement, lock state, and the
crank's angular velocity. It is updated only in `FixedUpdate`, under
[ADR-0002](0002-physics-stays-deterministic-without-multiplayer.md), and is
small enough to snapshot.

## Still open

- **Gear ratios against riding speed.** On the CRF's real ratios, 1st gear
  reaches about 77 km/h at the limiter and runs out near 60 km/h at peak power.
  [#6](https://github.com/elmofromok/MoMadness/issues/6) put riding speeds at 40
  to 60 km/h, so the Bike would live in 1st and gear choice in the air would
  barely vary. Either the ratios get compressed so that range spans about 1st to
  3rd, or #6's speeds turn out to be low. Riding decides. The 400 m gap
  threshold from [#11](https://github.com/elmofromok/MoMadness/issues/11) hangs
  on the same number.
- **Inertia figures.** No one publishes crank or wheel inertia for 450 MX bikes.
  The 0.01 to 0.02 kg·m² crank and 0.8 to 1.0 kg·m² rear wheel figures are
  estimates.
- **The torque table's points.** Dyno tests publish peaks and charts, not
  tables. The plateau shape is read off charts and estimated.
- **Every strength, multiplier and ramp time**, until the Bike is ridden.
- **Whether a Hard mode with no stall feels wrong.** If it does, stalling could
  return as an option.

## Current thinking

- [#8](https://github.com/elmofromok/MoMadness/issues/8) needs homes for
  throttle, front brake, rear brake, clutch, steering and two lean axes, plus two
  shift inputs only when manual shifting is on. It is marked blocked on #18.
- [#9](https://github.com/elmofromok/MoMadness/issues/9): a hard landing with
  the clutch out drags the engine down to idle rather than stalling it.
- The audio direction on the map's fog list now owes a limiter stutter, since
  the physics no longer has one.
