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

**Air-save**:
Recovering a badly-launched jump by leaning in flight so the Bike lands at a
survivable angle. The game's central skill.
_Avoid_: correction, recovery, save

**Assist**:
An automatic correction layered on top of the unassisted physics, always
expressible as a value that can be reduced to nothing.
_Avoid_: help, aid, auto-correct, arcade mode

**Crash**:
The Rider parting from the Bike after a bad landing or impact. A spectacle to
watch rather than a failure, since there is no progress to lose.
_Avoid_: wipeout, fail, death

## Unresolved

**Cross-over**:
Contested. The source material uses it for a Rider movement in one place and a
Bike rotation in another. Which one it names depends on a design decision that
is still open. Do not use the term in a spec until it is settled.
See [issue #2](https://github.com/elmofromok/MoMadness/issues/2).
