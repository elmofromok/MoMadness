# MoMadness

An arcade motocross game in Unity 6 (URP), taking its cue from Motocross
Madness (1998). The defining mechanic is separate bike and rider physics: the
rider is its own body, leaned and rotated independently of the bike while
airborne, to control landing angle.

It is **not a racing game**. The base game is an authored sandbox of open
themed zones, with no goals, no score, and no completion tracking. Discovery is
the reward. See [ADR-0001](docs/adr/0001-racing-is-cut-from-the-base-game.md).

The Unity project lives at the repo root (`Assets/`, `Packages/`,
`ProjectSettings/`). Anything outside `Assets/` is invisible to the Unity
asset pipeline and needs no `.meta` files.

## Working agreements

### Branches and PRs

Changes go through a pull request. Do not commit directly to `main`.

Branches are named after the wayfinder ticket the work resolves:
`wf-<issue>/<slug>`, for example `wf-2/rider-lean-coupling`.

### Physics

Physics runs in `FixedUpdate` with deterministic-friendly structure: no
frame-rate-coupled forces, no dependence on `Update` ordering. Multiplayer is
out of scope but must not be foreclosed. This is deliberate and should not be
"simplified" away. See
[ADR-0002](docs/adr/0002-physics-stays-deterministic-without-multiplayer.md).

Assists are tunable floats layered over an unassisted model, never baked into
the physics.

## Agent skills

### Effort map

The effort is charted as a wayfinder map at
[issue #1](https://github.com/elmofromok/MoMadness/issues/1). Tickets are its
sub-issues; the frontier is the open, unblocked, unassigned ones.

### Issue tracker

Issues live as GitHub issues in `elmofromok/MoMadness`, managed via the `gh` CLI.
See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles, each label string equal to its name.
See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` and `docs/adr/` at the repo root.
See `docs/agents/domain.md`.
