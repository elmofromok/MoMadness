# Rider lean drives Bike rotation in the air

When the Bike is airborne, moving the Rider turns it, through reaction torque
between the two bodies. The Rider is the instrument of air control, not an
animation over it.

That is the whole of what this records. It came from one prototype, and it is
expected to be refined as the real Bike and Rider get built and ridden.

## How it was learned

The prototype for [issue #2](https://github.com/elmofromok/MoMadness/issues/2)
put the same jump under two models. In Version A the Bike turns because the
Rider's body moved. In Version B the Bike turns on its own and the Rider
animates along. Both saved the jump when the input was held all the way, and
both crashed when it was ignored, so neither was the obviously broken one. They
differed on release. In A the correction is worth whatever the Rider's lean is
worth at that instant, so letting go gives it back and the lean is carried
through the landing. In B what was turned stays turned.

A felt like MoMadness. The release behaviour is why: the Air-save is something
held rather than something spent.

## Still open

The prototype was a 2D figure on a scripted arc, and these questions were out of
its reach:

- **Whether lean is the only thing that turns the Bike in the air.** Lean
  corrects an angle but cannot stop a spin, so on long jumps it may not be
  enough alone. Picked up in
  [ADR-0004](0004-the-gyro-effect-turns-the-bike-in-the-air.md): the gyro
  effect from gas and brake is a second channel.
- **How strong the coupling is.** That needs the real rig in a Unity scene.
- **Whether any direct Bike air control exists**, as an Assist or otherwise.
- **Whether the coupled feel survives a jointed rig.** Drive lag between input
  and body could make it feel indirect, which was the risk the ticket named.

## Current thinking

These follow from the decision as it stands and should be revisited with it.

- **`cross-over` names a Rider movement**, not a Bike rotation. `CONTEXT.md`
  had it as contested pending this ticket.
- **The reaction torque probably wants to come from the bodies.** A script that
  reads the lean axis and torques the Bike directly could pass for A on a simple
  jump while behaving like B underneath.
- **The #4 rig carries more weight.** If the torque comes from the bodies, the
  `ConfigurableJoint` hierarchy becomes part of air control and its drive
  strengths become feel parameters.
- **Reaction torque would run in `FixedUpdate`**, like the rest of the core
  simulation, under
  [ADR-0002](0002-physics-stays-deterministic-without-multiplayer.md).
- **Downstream tickets get inputs.**
  [#7](https://github.com/elmofromok/MoMadness/issues/7): keeping the Bike
  upright should not fight the Rider for the same rotation in the air.
  [#8](https://github.com/elmofromok/MoMadness/issues/8): lean is an air-control
  axis and probably wants analog input.
  [#10](https://github.com/elmofromok/MoMadness/issues/10): the camera should
  let the player read the Rider's position in flight.
