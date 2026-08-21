# Prototype: does Rider lean rotate the Bike?

Throwaway prototype for
[issue #2](https://github.com/elmofromok/MoMadness/issues/2). Not production
code, not shipped, not covered by tests. It lives outside `Assets/`, so the
Unity asset pipeline never sees it.

## Run it

Double-click `air-control-prototype.html`. No server, no build, no install.

Then:

1. Click the picture once, so the keyboard reaches it.
2. The Bike rides and jumps by itself. You never steer it. The only thing you
   control is the Rider, the orange figure.
3. While it is in the air, hold `W` to throw the Rider forward or `S` to throw
   them back. Does the Bike rotate underneath them? That is the question.
   Note that **`S`, throwing the Rider back, is what brings the nose down**.
   See "Which key brings the nose down" below.
4. It lands. Within 25&deg; of the ground is clean, otherwise it crashes and
   respawns. The bar top-left shows how wrong the landing would be right now,
   and a line across the picture names the key you want.
5. Work down the seven guided runs in order. Each sets the sliders for you and
   drops you into the same jump, so only the physics changes between runs.

## What you are trying to decide

You are not trying to win. Landing clean is feedback, not a score, and there is
nothing to complete.

Runs 1 to 4 are a taste test. Ride each twice and leave the sliders alone. They
are the same jump with different physics behind it: coupled, a control case
where nothing you do works, independent, then both. By the end of run 4 you want
an opinion on one question. **Does moving the Rider feel like it is doing the
work, or does it feel indirect next to run 3?** That opinion is the answer to
issue #2. Runs 5 to 7 then poke at the edges: exaggeration, the nose-low case,
and the roll axis.

The sliders come after. Two of them matter. Set Direct Bike authority to 0,
leave the guided runs, ride free play, and walk Lean coupling up from about 0.5
to 2.5, looking for the point where the air-save can rescue a bad jump without
feeling twitchy. That number is the second half of the answer. The other three
sliders are there to explain the model, not to be tuned.

## The question

When the Bike is airborne, does moving the Rider apply reaction torque to the
Bike, or are the two rotations independent? The reference doc reads both ways,
and roughly half the effort map hangs off the answer.

## What it builds

Model 3 from the issue, so all three candidates are reachable from one build.
A single stick input drives two channels at once:

- **Lean coupling** (`c`) scales the reaction torque from throwing the Rider's
  weight. Angular momentum is conserved across Bike and Rider:
  `wBike = (L - c*Ir*wRiderLean) / (Ib + c*Ir)`. `c = 1` is honest physics,
  `c > 1` exaggerates it.
- **Direct Bike authority** applies torque straight to the Bike, which is
  control the Rider never paid for.

So `c = 0` with authority up is **model 2**, `c = 1` with authority at 0 is
**model 1**, and both together is **model 3**.

Pitch and roll are two independent one-degree-of-freedom axes. Real 3D flight
adds gyroscopic cross-talk this does not simulate, so a coupling number found
here is a starting point for the Unity tune. The choice of model is what should
transfer intact.

## The part worth keeping

`AirControl` in the `<script>` block is a pure, deterministic, fixed-step module
with no DOM and no engine calls. If a coupled model wins, that is what lifts
into `FixedUpdate`. Everything else on the page is scaffolding.

## What the guided runs already show

Every run launches from an identical flight state, so only the tuning differs.
Doing nothing lands 34&deg; nose-high and crashes in all of them.

| Run | Hold the throw | Stab and release |
| --- | --- | --- |
| Model 1, coupled | saves to 16&deg; | no effect |
| Model 2, independent | overshoots to -35&deg; | saves to 5&deg; |
| Model 3, both | saves to -15&deg; | saves to 19&deg; |

That split is the real finding so far, and it is a feel question, not a physics
one. Under coupling, a throw buys a **fixed budget of degrees** set by the
Rider's inertia share and reach (about 18&deg; at the defaults). It moves where
the Bike is pointing, never how fast it is spinning, and letting go before the
landing hands the angle straight back, so the player has to hold the throw
through touchdown. Direct authority instead gives rate control: a stab of the
stick, then release.

## Which key brings the nose down

Throwing the Rider **back** brings the nose down. Throwing them **forward**
brings it up. That is backwards from what most people guess, and it is what
conservation of angular momentum gives you: with no external torque, the Rider
rotating one way forces the Bike to rotate the other. Pike forward and your legs
come up; the Bike is the legs.

The everyday motocross coaching, weight forward to bring the front down, is
about a different thing. On the ground it is weight transfer through the
suspension. In the air, riders bring the nose down with the rear brake, and when
they use their body they push and pull between the bars and the pegs, which is a
force couple acting on the Bike directly rather than a body rotation. This
prototype models the body rotation only, which is the mechanism issue #2 is
actually asking about.

So the direction of the mapping is a design decision in its own right, and the
**Invert the lean keys** box in the Tuning panel exists to let you feel both. Off
is what the physics gives you, nose-down on `S`. On is what a player probably
expects, nose-down on `W`. The Rider still visibly moves the way the physics says
in both cases, so only the key mapping changes. Every guided run follows whichever
way you set it.

If a coupled model wins but only with the keys inverted, that is worth recording
on the issue too: it means the animation and the control will read as
contradictory to anyone who looks closely, and someone has to decide that is
acceptable.

## Recording the answer

The landing log stamps every landing with the coupling and authority in force at
the time. The **Your verdict** box at the bottom of the page turns your pick,
plus whatever the sliders are set to, into a note to paste onto
[issue #2](https://github.com/elmofromok/MoMadness/issues/2). What goes on the
issue is the model, and if coupled, roughly what coupling value.

Then the validated `AirControl` module lifts into the real code, and this branch
stays put as the primary source behind the decision. It is not meant to merge.
