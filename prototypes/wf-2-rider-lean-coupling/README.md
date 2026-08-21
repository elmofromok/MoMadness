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
4. It lands. Within 25&deg; of the ground is clean, otherwise it crashes and
   respawns. The bar top-left shows how wrong the landing would be right now,
   and a line across the picture names the key you want.
5. Work down the seven guided runs in order. Each sets the sliders for you and
   drops you into the same jump, so only the physics changes between runs.

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

## Recording the answer

The landing log stamps every landing with the coupling and authority in force at
the time. Ride, sweep, and the answer to write back to issue #2 is the model,
and if coupled, roughly what coupling value.
