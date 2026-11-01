# The gyro effect turns the Bike in the air

When the Bike is airborne, changing a wheel's spin turns it. Braking moves a
wheel's spin into the frame and drops the nose. Throttle spins the rear wheel up
and lifts it. This is a second air-control channel alongside Rider lean
([ADR-0003](0003-rider-lean-drives-bike-rotation-in-the-air.md)), and it does a
different job.

This was settled by grilling in
[issue #17](https://github.com/elmofromok/MoMadness/issues/17), not by riding.
Nothing here has been felt yet, and it is expected to change once it has.

## Why lean was not enough

Lean changes the Bike's angle and gives it back when the Rider comes back. It
cannot change how fast the Bike is spinning. On a 3 s jump a 13°/s launch spin
builds up to nearly 40° of error, against roughly 20° of lean authority in the
issue #2 prototype. The arcade airtime target is 2.5 to 4 s.

The gyro changes the spin rate, and the change stays after the input is
released. So the two channels split the work: lean trims the angle and is
reversible, the gyro sets the spin and has to be countered with the opposite
input.

## What the first pass builds

The first pass targets **Hard mode**, the unassisted physics. **Easy mode**
comes later, as Assists layered on top.

- **The gyro is a real tool at roughly real strength.** A rough, unmeasured
  estimate puts a full rear-brake lock on a 15 m/s jump at 40 to 50°/s of
  nose-down spin.
- **It acts on every axis.** The wheels' angular momentum is computed as a full
  vector, so a spinning wheel resists roll and turns roll into yaw
  (precession). The roll and yaw terms sit on their own float, at real strength
  in Hard mode.
- **A real engine model drives the rear wheel**, with RPM, a torque curve,
  gears, a rev limiter and a clutch. Throttle's nose-up authority is capped by
  the current gear and the rev limiter, and pulling the clutch lets the wheel
  coast. Detail is for
  [#18](https://github.com/elmofromok/MoMadness/issues/18).
- **Front and rear brakes are separate inputs.** Both drop the nose in the air.
- **Wheel speed is reconciled on landing by tire friction.** A slow or locked
  wheel skids and pitches the Bike forward, an overspinning one drives it ahead,
  and a locked front wheel can send the Rider over the bars. An Assist float
  softens this, at 0 in Hard mode.

#3 uses raycast wheels, so the wheels are not physics bodies. Wheel spin is
state the Bike tracks, and the gyro torque on the frame is computed from it.
That is honest physics and runs in `FixedUpdate` under
[ADR-0002](0002-physics-stays-deterministic-without-multiplayer.md).

## Easy mode, later

Easy mode drains the gyro's spin out of the Bike once the input is released, so
the Bike stops spinning when the player lets go, as Version A did in the issue
#2 prototype. Other Assists can soften the landing mismatch and precession.
Each is a float, so Hard mode is all of them at 0.

## Still open

- **Strength.** Real strength is the starting point and riding decides.
- **Whether the engine's own spinning parts add to the gyro.** Belongs to #18.
- **Whether the two channels feel right together.** Nobody has ridden lean and
  gyro on the same Bike yet.

## Current thinking

- [#8](https://github.com/elmofromok/MoMadness/issues/8) needs room for
  throttle, front brake, rear brake, clutch and two lean axes on a gamepad.
- [#9](https://github.com/elmofromok/MoMadness/issues/9) inherits landing
  outcomes from wheel-speed mismatch, including over-the-bars on a locked front
  wheel.
- #3's tire friction model has to handle large slip at touchdown, not only
  steady riding.
- [#7](https://github.com/elmofromok/MoMadness/issues/7): precession is part of
  what keeps a moving Bike upright, so that ticket starts with it in the model.
