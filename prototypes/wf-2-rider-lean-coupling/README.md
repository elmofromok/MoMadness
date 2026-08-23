# Prototype: does the Rider turn the Bike?

Throwaway prototype for
[issue #2](https://github.com/elmofromok/MoMadness/issues/2). Lives outside
`Assets/`, so the Unity asset pipeline never sees it.

## Run it

Double-click `first-question.html`. Click Version A, hold the orange button
until you land. Then do the same with Version B.

## The question

When the player presses a button in the air, does the Bike turn because the
Rider's body moved, or does the Bike turn on its own with the Rider animating
along?

That is the whole thing. Both versions save the same jump, so neither is the
obviously broken one. They are two different ideas about what the player is
doing in the air.

## What separates them

Doing nothing crashes in both. Holding the button all the way lands in both.
The difference only shows up when you let go early:

- **Version A**, the Rider turns the Bike. The correction is worth whatever the
  Rider's lean is worth right now. Let go and it goes away with them, so you
  have to hold it through the landing.
- **Version B**, the Bike turns itself. What you turned stays turned. Let go
  and it holds.

## Answer to record

Which one felt more like MoMadness. That goes on issue #2, and it decides
whether the Rider is the instrument or the animation.

Everything else the issue raises (how strong the effect is, whether the two mix,
which key does what) only matters after that, and probably cannot be judged
without a real bike in a Unity scene.

## History

An earlier version of this prototype tried to answer all of issue #2 at once:
seven scenarios, five tuning sliders, live physics readouts. It was
unreadable. It is gone from the working tree and kept in this branch's history
if the numbers are ever wanted. The lesson is in the commit log: answer one
question, add more only when the answer demands it.
