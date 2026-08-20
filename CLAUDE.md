# MoMadness

An arcade motocross game in Unity 6 (URP), built as a spiritual successor to
Motocross Madness (1998). The defining mechanic is separate bike and rider
physics: the rider can be leaned and rotated independently of the bike while
airborne, to control landing angle. Terrain is open and rolling rather than
tight linear tracks.

The Unity project lives at the repo root (`Assets/`, `Packages/`,
`ProjectSettings/`). Anything outside `Assets/` is invisible to the Unity
asset pipeline and needs no `.meta` files.

## Agent skills

### Issue tracker

Issues live as GitHub issues in `elmofromok/MoMadness`, managed via the `gh` CLI.
See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles, each label string equal to its name.
See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` and `docs/adr/` at the repo root.
See `docs/agents/domain.md`.
