# MoMadness

An arcade motocross game in which the bike and the rider are separate physics
bodies, set in an open world built for riding rather than racing. This glossary
fixes the vocabulary the design and the code both use.

## Language

### The world

**Authored sandbox**:
A world with no goals, no score, and no completion tracking, whose content is
hand-placed moments rather than generated activities. Discovery is the reward.
_Avoid_: open world, sandbox, free roam

**Zone**:
A self-contained themed area, entered on its own, roughly four to five minutes
to cross by bike.
_Avoid_: level, track, map, stage, environment

**Playground feature**:
Authored terrain or an authored object placed so it can be ridden.
_Avoid_: obstacle, prop, jump

**Vignette**:
An authored moment placed so it can be noticed. It need not be rideable.
_Avoid_: set dressing, scenery, secret

**Design speed**:
The range of speeds a Playground feature is built to be hit at. Below it the
Bike comes up short, above it the Bike overshoots the landing.
_Avoid_: target speed, entry speed, approach speed

**Finished zone**:
A Zone with enough Playground features and Vignettes, spread evenly enough,
that a rider rarely goes half a minute without something to steer toward, and
whose every Playground feature has been ridden.
_Avoid_: complete zone, done zone, shippable zone

**Easter egg**:
The informal umbrella for both a Playground feature and a Vignette. Imprecise
by nature, since the two cost and behave differently. Use the precise term
when it matters.

### Riding

**Player**:
The person holding the controller.

**Rider**:
The human figure in the world. A body in its own right, never a part of the
Bike.
_Avoid_: character, driver, racer

**Bike**:
The motorcycle in the world. A body in its own right, never carrying the Rider
as a part.
_Avoid_: vehicle, machine

**Lean**:
Player-driven rotation of the Rider relative to the Bike.
_Avoid_: tilt, shift, weight transfer

**Cross-over**:
A Lean that throws the Rider across the Bike to one side. In the air it turns
the Bike by reaction torque.
_Avoid_: whip, body english, bike rotation

**Air-save**:
Recovering a badly-launched jump by leaning in flight so the Bike lands at a
survivable angle. The game's central skill.
_Avoid_: correction, recovery, save

**Assist**:
An automatic correction layered on top of the unassisted physics, always
expressible as a value that can be reduced to nothing.
_Avoid_: help, aid, auto-correct, arcade mode

**Gyro effect**:
Rotation of the Bike caused by changing a wheel's spin. Braking drops the nose
and throttle lifts it. Unlike a Lean, it changes how fast the Bike spins, and
the change stays after the input is released.
_Avoid_: gyroscope, wheel torque, brake tap

**Hard mode**:
The game with every Assist at zero: the unassisted physics.
_Avoid_: sim mode, realistic mode, pro mode

**Easy mode**:
The game with Assists raised so that releasing an input settles the Bike
instead of leaving it spinning.
_Avoid_: arcade mode, casual mode, beginner mode

**Crash**:
The Rider parting from the Bike after a bad landing or impact. A spectacle to
watch rather than a failure, since there is no progress to lose.
_Avoid_: wipeout, fail, death
