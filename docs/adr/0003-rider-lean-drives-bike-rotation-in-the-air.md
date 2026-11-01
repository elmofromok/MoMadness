# Rider lean drives Bike rotation in the air

When the Bike is airborne, the player has no direct control over its rotation.
Moving the Rider is the only thing that turns it, through reaction torque
between the two bodies. The unassisted model is fully coupled.

This was settled by riding, not by argument. The prototype for
[issue #2](https://github.com/elmofromok/MoMadness/issues/2) offered the same
jump under two models: Version A, where the Bike turns because the Rider's body
moved, and Version B, where the Bike turns on its own and the Rider animates
along. Both saved the jump when the input was held all the way, and both crashed
when it was ignored, so neither version was the obviously broken one. They
differed only on release. In A the correction is worth whatever the Rider's lean
is worth at that instant, so letting go gives it back and the lean has to be
carried through the landing. In B what was turned stays turned.

A felt like MoMadness. The reason is that property on release: the Air-save
becomes something held rather than something spent. The Rider's position is the
Bike's angle at every instant, which makes the input continuous and keeps the
Rider from degrading into animation over the physics.

## Consequences

**Cross-over names a Rider movement.** The source material used the term both
ways and `CONTEXT.md` listed it as contested pending this decision. The Bike
rotation reading is dead. The term is now defined and usable in specs.

**Reaction torque has to come from the bodies, not from the input.** A script
that reads the lean axis and applies torque to the Bike directly would
reproduce A's feel on a flat jump and still be B underneath, and it would come
apart the moment the Rider's body is obstructed, mid-crash, or at a joint limit.
The torque must fall out of the Rider and Bike actually pushing on each other.

**The Rider rig is now load-bearing.** [#4](https://github.com/elmofromok/MoMadness/issues/4)
settled on a hand-authored `ConfigurableJoint` hierarchy, chosen as the way to
pose and ragdoll the Rider. It is now the air-control mechanism. Joint drive
strengths stop being cosmetic and become feel parameters, and the rig's
non-reversibility costs more than it did when that ticket closed. PuppetMaster
remains unauthorised.

**Coupling strength is not decided.** The prototype could not judge it and was
cut back until it stopped pretending to. That number wants a real Bike in a
Unity scene. Per the project's assist rule it is a tunable float over an
unassisted model that is already fully coupled, so any direct Bike air-authority
can only exist as an Assist that reduces to nothing, never as a second input
channel in the base physics.

**Reaction torque is computed in `FixedUpdate`**, under
[ADR-0002](0002-physics-stays-deterministic-without-multiplayer.md). It is core
simulation, not a cosmetic layer, so it gets no exemption.

The risk the ticket named, that a coupled model feels indirect, did not show up
at prototype fidelity, where the Rider is a two-dimensional figure on a scripted
arc. It can still show up on a jointed rig with drive lag between the input and
the body. If the air-save feels mushy once the real rig exists, this is the
decision to reopen, and the fix is rig responsiveness before it is a coupling
float.

Downstream, [#7](https://github.com/elmofromok/MoMadness/issues/7) inherits a
constraint: whatever keeps the Bike upright cannot fight the Rider for the same
rotation while airborne. [#8](https://github.com/elmofromok/MoMadness/issues/8)
now has its most demanding axis, since lean is the air-control input and needs
analog resolution, which keyboard cannot give. [#10](https://github.com/elmofromok/MoMadness/issues/10)
gains a requirement: the camera has to make the Rider's position legible in
flight, because that position is the thing the player is reading.
