# Physics stays deterministic without multiplayer

Multiplayer is out of scope for the base game, but it is wanted eventually.
Physics therefore runs in `FixedUpdate` with deterministic-friendly structure:
no frame-rate-coupled forces, no dependence on `Update` ordering, no simulation
state mutated outside the fixed step. No netcode is written, and none is planned
for the base game.

The reason is asymmetric cost. Writing physics this way in a single-player game
costs close to nothing, since it is good practice regardless. Retrofitting it
onto a rig built with frame-coupled forces means rewriting the core, and the
core here is the bike and rider simulation, which is the hardest and most tuned
part of the project.

## Consequences

The discipline will look like unnecessary ceremony to anyone reading the code
without this context, because nothing in the shipped game requires it. Expect
the temptation to "simplify" by moving force application into `Update` or
scaling by `Time.deltaTime` in the wrong place. Do not.

This constrains the ground handling model. Anything chosen for bike-to-ground
contact has to behave predictably under a fixed step, which is a live criterion
in that decision rather than an afterthought.

It does not make the game multiplayer-ready. It only avoids foreclosing it.
Networked physics needs far more than a fixed timestep, and all of that is a
separate effort.
