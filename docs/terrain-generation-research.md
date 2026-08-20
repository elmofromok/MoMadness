# Research: terrain generation tool

Research for [#5 — Choose the terrain generation tool](https://github.com/elmofromok/MoMadness/issues/5).
Target: Unity **6000.5.9f1**, URP, solo dev, discrete themed zones, ~4–5 min to cross,
no streaming, fast bike physics with suspension.

Every claim is marked **[cited]** (traced to a primary source) or **[inferred]** (my reasoning
on top of cited facts). Prices that could not be verified from the vendor's own page are
marked **[unverified]**.

Researched August 2026. Sources are Unity 6000.5 docs, package docs, vendor store pages fetched
directly, NVIDIA PhysX docs, and developer write-ups.

**Short answer: Unity's own Terrain Tools package (free), generating in-editor — and the rideable
shapes are authored as mesh, not sculpted into the heightmap.** Reasoning in
[Recommendation](#recommendation).

---

## Recommendation

### The tool: `com.unity.terrain-tools` 5.3.3, in-editor. Cost: **$0.**

Buy nothing yet. The procedural base is generated in Unity, on the TerrainData asset you are already
editing, by the free first-party package.

**Why, in order of weight:**

1. **No tool on the market generates what this project actually needs.** Gaea, World Creator and
   Instant Terra all generate *naturalistic landform* — mountains, erosion, rivers. None of them
   generates a ramp, a gap, or a landable slope. The ticket's own framing is correct and it cuts
   further than it looks: since the rideable shape has to be hand-authored regardless, the generator
   is only responsible for **believable ground**, and Terrain Tools' Noise plus hydraulic/thermal/
   wind erosion produces believable ground at zone scale. The expensive tools' advantage —
   photoreal geological realism at 8k+ — is not this project's bottleneck.

2. **Generating in-editor deletes the round-trip problem rather than managing it.** Every external
   tool's Unity ingest is a heightmap file, and Unity's importer "replaces the existing heightmap"
   (§1.1). Terrain Tools has no import boundary because there is no separate generate step to
   re-run — brushes mutate the live asset. The criterion the ticket says to judge on is won by
   default.

3. **Scope discipline is this project's stated dominant constraint** (issue #1). Adding a second
   application, its learning curve, a file pipeline, tile-naming conventions and a 16-bit
   requantisation on every loop — to get erosion realism on a trailer park and a farm — is exactly
   the kind of cost this project has already decided to refuse.

4. **Two of the four planned zones barely want a generator at all.** Trailer park and farm are flat.
   Open-pit mine is terraced, and Terrain Tools ships a **Terrace** brush. Only jungle really wants
   erosion. The case for a paid generator is thinner here than the tool category suggests.

5. **Mesh Stamp closes the gap that would otherwise justify a purchase.** Author a shape as a mesh
   anywhere — ProBuilder, Blender — and stamp it into the heightmap with Behaviour Min/Max. That is
   the "put an authored shape into the terrain" capability, first-party and free.

### The representation: Terrain for the ground, **mesh for everything meant to be ridden**

This is the more consequential half of the recommendation, and the evidence converges hard:

- A jump lip needs ≤ 0.15 m sampling. Unity Terrain's *maximum* over a 2 km zone is 0.5 m (§5.6).
  **Structural, not tunable.**
- Unity's own WheelCollider manual tells racing developers the collision mesh "is made separately
  from the visible mesh, making the collision mesh as smooth as possible" — which a TerrainCollider
  cannot be, because it *is* the visible surface (§5.4).
- The closest shipped precedent — Lonely Mountains: Downhill, a fast Unity off-road bike game —
  started on Unity Terrain, abandoned it over "restrictions on the level design", and decoupled
  collision from rendering entirely (§6.1).
- An adaptive authored collision mesh (~15 MB) is both **cheaper and sharper** than a max-resolution
  terrain (~101 MB) for a 2 km zone (§6.3).
- And it makes the round-trip question moot: playground features that are GameObjects live in the
  *scene*, which no terrain regenerate can touch (§4.2).

Keep Terrain as the base anyway, because for *uniform* ground coverage a heightfield beats a mesh
15–25× on memory (§6.3), it gives free smoothed normals via `GetInterpolatedNormal` (§5.4), and it
is the only representation that keeps MX vs ATV-style **deformable ruts** cheaply available (§6.2).

### The generate-then-edit workflow

1. **Size the zone at 2048 m × 2048 m, one Terrain tile, no tiling.** `heightmapResolution` **2049**
   (1.0 m spacing, ~25 MB heights + collider). Tiling buys no memory and hands you collider seams
   Unity's Auto Connect does not fix (§5.5, §6.4).
2. **Set Terrain Height to ~200–250 m. Never leave it at 600.** Vertical quantisation is
   `size.y / 32766`; 250 m gives ~7.6 mm steps, 600 m gives ~18 mm (§5.5).
3. **Generate the base** in Terrain Toolbox / Terrain Tools: Create New Terrain → **Noise** for
   large-scale relief → **Hydraulic / Thermal** erosion for believability → **Terrace** for the mine
   → **Slope Flatten** to open rideable corridors → **Brush Mask Filters (Slope)** to audit and
   knock down anything over ~30°.
4. **Snapshot `GetHeights()` as `base`** with the delta-layer editor script (§4.3). Do this before
   any hand pass, every time.
5. **Hand pass — this is where the game gets made:**
   - Playground features (ramps, kickers, tabletops, gaps, berms) authored as **meshes**, placed as
     GameObjects, each with a **separate smooth low-poly convex collision mesh** and a feathered
     skirt past the visual edge.
   - **Mesh Stamp with Behaviour: Min** to conform the heightmap under each feature before placing
     it, then **Slope Flatten** the run-up and landing.
   - **Paint a terrain hole under every mesh footprint** — this removes the double-contact seam at
     the root rather than papering over it (§6.4).
   - Vignettes as plain GameObjects.
   - Any height edits that genuinely must be in the heightmap (rut lines, blending a feature into a
     hill) are captured by the delta script.
6. **Regenerating is now safe.** The delta script replays hand height edits onto the new base;
   splat, trees, details, holes and every authored GameObject were never in the overwritten channel.
7. **Physics setup implied by this choice** (feeds the bike-physics ticket, not this one):
   custom multi-raycast suspension rather than WheelCollider; `TerrainData.GetInterpolatedNormal`
   for the orientation/friction basis; `Physics.defaultContactOffset` **lowered** to ~0.002–0.005,
   not raised; `ContinuousDynamic` CCD, which requires the bike's colliders be Sphere/Capsule/Box
   primitives, not convex meshes; reduced fixed timestep.
8. **URP build gate — do this in week one.** Enable **Terrain Holes** in *every* URP asset in the
   project, and add a built-player smoke test. The failure mode is an invisible pit that only
   appears in a shipped build (§6).

### If Terrain Tools proves insufficient after zone one

In priority order, and only on evidence:

- **World Creator Indie — $149 perpetual (or $59/yr).** Cheapest, the only maintained first-party
  Unity Editor bridge, and Path layers with per-control-point height and Flatten are the closest
  thing on the market to "draw the line the bike should ride." **Verify URP + Unity 6 support with
  the vendor before paying** — the bridge docs name only Built-In RP.
- **Heightmap Studio — free for personal use, worth trying now.** It is the only tool surveyed built
  for motocross specifically (jump placement, rut and bump carving, track splines, non-destructive
  layers, 16-bit 1025/2049 export matching Unity exactly). Low cost to evaluate; get a commercial
  licence quote before depending on it.
- **MapMagic 2 — free.** If a re-runnable node graph is wanted inside Unity, its **Lock** feature is
  the only in-Unity mechanism found that preserves hand edits across regeneration.
- **Gaea Indie — $99.** Better procedural power and a proper Unity output node at Unity's exact
  resolutions, but **Gaea 2 has no spline/path/road tools** and you write the Unity importer.
  Defensible only as a bet on Gaea 3.0's promised vector road tools and Unity plugin.
- **Houdini Indie — $299/yr.** The only *true* round-trip (Unity Terrain in, real TerrainData out)
  and the only tool that can express "carve a rideable line along this spline at a graded slope" as
  a repeatable rule. Reject it **for now**: no free way to evaluate the Unity path, months of
  ramp-up, the one solo Unity dev on record abandoned it over Engine-for-Unity integration, and the
  motocross-specific parts (banking, grade caps, ramps) have essentially no prior art and would be
  yours to write in VEX. Revisit only if the zone count grows past ~6 and hand-authoring becomes the
  measured bottleneck.
- **Instant Terra — eliminated.** No release since March 2022, no Unity plugin, dead community domain.

---

## 1. Unity 6 built-in Terrain and the Terrain Tools package

### 1.1 What ships in the box

**[cited].** The complete built-in sculpt set in 6000.5 is **Raise or Lower Terrain, Set Height,
Smooth Height, Stamp Terrain, Paint Holes** — plus Paint Texture, Paint Trees, Paint Details.
There is **no built-in noise, no erosion, no procedural generator of any kind**. Unity ships the
data-write API and brushes, nothing that authors shape for you.
[Create and edit Terrains](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-UsingTerrains.html)

**[cited].** Heightmap exchange is `Import Raw` / `Export Raw` on the Terrain Settings tab.
RAW only, "a 16-bit grayscale format that's compatible with most image and landscape editors",
with 8- or 16-bit depth and a Windows/Mac byte-order toggle.
[Working with Heightmaps](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-Heightmaps.html)

**[cited].** Importing "applies it to the selected terrain's height data, effectively replacing the
existing heightmap with the imported one." This one sentence is the whole round-trip problem for
every file-based generator. (same page)

**[cited].** The scripting surface is channel-separated, which turns out to matter enormously
(see §4): heights (`GetHeights`/`SetHeights`/`SetHeightsDelayLOD`), splat (`SetAlphamaps`,
`terrainLayers`), detail (`SetDetailLayer`), trees (`SetTreeInstances`), holes (`SetHoles`).
[TerrainData](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData.html)

**[cited].** `SetHeights(int xBase, int yBase, float[,] heights)` — heights are normalized 0–1,
array indexed `[y,x]`, and **the write can target a sub-region**: "The area affected is defined by
the array dimensions and starts at xBase and yBase."
[TerrainData.SetHeights](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData.SetHeights.html)

### 1.2 Terrain Tools package — alive, current, and the actual answer

**[cited].** `com.unity.terrain-tools` is **not deprecated**. Current version **5.3.3**, changelog
dated **2026-08-19**, with entries explicitly targeting this editor line:
5.3.2 "Migrate Mono APIs to CoreCLR-compatible APIs available in Unity 6000.5 and above";
5.3.3 fixes a compile error "on Unity 6000.4 and above".
[Changelog](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/changelog/CHANGELOG.html)
· [package.json](https://raw.githubusercontent.com/needle-mirror/com.unity.terrain-tools/master/package.json)

**[inferred] (high confidence).** 5.3.3 is the version to use on 6000.5.9f1; anything below 5.3.2
risks CoreCLR/Mono breakage.

**[cited].** It adds the procedural and shaping toolset Unity lacks, all as brushes:
**Noise** ("uses different noise types and fractal types to modify Terrain height"),
**Hydraulic / Thermal / Wind erosion**, **Bridge** ("creates a Brush stroke between two selected
points to build a land bridge"), **Clone**, **Terrace**, **Contrast**, **Sharpen Peaks**,
**Slope Flatten** ("flattens the Terrain while maintaining the average slope"), **Pinch**,
**Smudge**, **Twist**.
[Paint Terrain tools](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/paint-terrain.html)

**[cited] — the load-bearing feature for this project.** Terrain Tools overrides Stamp Terrain and
adds **Mesh** stamp mode. You supply an arbitrary mesh; it is projected into the heightmap.
Behaviour is **Min** ("only where it lowers the Terrain") / **Set** / **Max** ("only where it
raises the Terrain"), with a Blend Amount where 0 "applies the stamp while overriding existing
Terrain features" and 1 preserves them. Documented limits: "only the widest section of a convex
mesh projects into the heightmap"; fidelity "is based on the Terrain's resolution, and not the
vertex density of the assigned mesh"; "Shapes with holes or a lot of concave details don't work
well as mesh stamps."
[Stamp Terrain](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/stamp-terrain.html)

**[cited].** **Brush Mask Filters** let a sculpt or paint operation be restricted by Aspect,
Concavity, **Height**, **Slope**, or Layer, composed with math nodes (Add, Clamp, Remap, Min,
Max…). "Unity sequentially applies Brush Mask Filters starting from the top of the list."
Caveat in the docs: terrain-based filters "give the best results when you use them to paint
Terrain Layers" and may give "unusual results and visual artifacts" on height-modifying tools.
[Brush Mask Filters](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/brush-mask-filters.html)

**[cited].** **Terrain Toolbox** (`Window > Terrain > Terrain Toolbox`) provides Create New Terrain,
batch Terrain Settings, Terrain Utilities (including **Split**, which "lets you divide Terrain into
smaller tiles while properly preserving Terrain height, Terrain Layers, and other details"), and
Terrain Visualization (Altitude Heatmap).
[Terrain Toolbox](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/terrain-toolbox.html)

**[cited].** Toolbox heightmap **import** supports Global (one map over all tiles), **Batch**
(filenames "must end with two index digits — the index along the X axis, and the index along the Z
axis", e.g. `Zone_0_0`, `Zone_1_1` for a 2×2 grid), and Tiles (manual assignment). RAW only,
power-of-two sizes, with Height Remap.
[Import Heightmap](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/toolbox-import-heightmap.html)

**[cited].** **Export** supports PNG, RAW and TGA at 16- or 8-bit, with Levels Correction.
So a Unity terrain can be round-tripped *out* to any external tool.
[Export Heightmaps](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/toolbox-export-heightmaps.html)

### 1.3 No successor is coming

**[cited].** Unity's announced next-gen world-building/terrain work was cancelled. Staff in the
"Terrain system improvements" thread: Unity "made the difficult decision to deprioritize some
previously announced features", the World Building team was laid off, and "We are currently
reassessing how we can improve the existing Terrain system moving forward."
[Terrain system improvements](https://discussions.unity.com/t/terrain-system-improvements/1637270)
The Unite 2025 roadmap thread confirms: "We have paused some other ambitious work like our new
animation and world building workflows."
[Unite 2025 roadmap](https://discussions.unity.com/t/the-unity-engine-roadmap-unite-2025/1696495)
The Unity 7 announcement (Unite Seoul, July 2026) names no terrain system.
[Unity 7 roadmap](https://unity.com/news/unity-7-roadmap-revealed-at-unite-seoul)

**[inferred] (high confidence).** Terrain Tools 5.3.3 *is* the Unity terrain authoring story for the
6.x lifetime. Do not architect around a future first-party replacement.

---

## 2. Dedicated generators

All prices below were read from the vendor's own store page in August 2026.

### 2.1 Gaea (QuadSpinner) — current 2.3.0.1, with 3.0 in development

**[cited].** Perpetual licences, revenue-gated:

| Edition | Price | Eligibility | Notable limits / features |
|---|---|---|---|
| Community | **FREE forever** | everyone | max **1024×1024**, **no tiled builds**, **no commercial use**, no regions/mutations/variables/automation/bridges |
| Indie | **$99** | revenue < $100K | adaptive meshes, split builds to tiles, **Unreal bridge**, commercial use, 2 PCs |
| Professional | **$199** | revenue < $1M | tiled builds to 262,144², regions, mutations, variables, **Houdini bridge**, custom bridges |
| Enterprise | **$299** | revenue > $1M | all features |

[quadspinner.com/order](https://quadspinner.com/order) · [quadspinner.com](https://quadspinner.com/)

**[cited].** The free Community edition **forbids commercial use** and caps at 1024². It is a learning
tier, not a production tier. Indie at $99 is the real entry price.

**[cited] — current version is 2.3.0.1 (13 May 2026).** Gaea 3.0 Early Access was targeted for
July 2026 with stable around September 2026; a 2.x purchase includes 3.0.
[Changelog](https://quadspinner.com/Download/Changelog)
· [Gaea 3.0 development update](https://blog.quadspinner.com/gaea-3-0-development-update/)
· [Gaea3](https://quadspinner.com/Gaea3)

**[cited] — no Unity bridge, but a dedicated Unity *output node*.** Gaea ships plugins for Unreal and
Houdini only. The **`Unity` output node** exports power-of-two-plus-one heightmaps at
**513/1025/2049/4097/8193** — exactly Unity's allowed values — as `.raw`/`.r16` 16-bit ushort, with
up to 10 inputs (heightfield plus masks), and a `Splat` node packs 4 heightfields to RGBA. Tiled
builds use `_y%Y%_x%X%` naming. **You write the Unity-side importer yourself**; note the contrast
that the Unreal output node emits a JSON handshake for its bridge and the Unity node does not.
[Gaea → Unity docs](https://docs.gaea.app/guides/use-in/engines/unity.html)

**[cited] — it can ingest a heightmap back into the graph.** The **`File` node** loads
`.raw/.r32/.exr/.tif/.png/.psd/.hdr/.pfm/.bmp` at 8/16/32-bit as a terrain and everything downstream
treats it as native, with Raw / Normalized / **Mapped** scale modes (Mapped is the one to use for
deterministic Unity round-trips). Gaea works internally in 32-bit float, so a Unity→Gaea→Unity loop
quantises at the Unity end.

**[cited] — good shape primitives, one decisive gap.** `Shape` (circle/rectangle with a **Sides**
control for angular polygons), `Cone`, `Hemisphere`, `Constant`, `LinearGradient`/`RadialGradient`
(ramps), **`Draw`** (freehand painting), **`Object`** (imports FBX/OBJ and projects the mesh into
the heightfield — the route for a Blender-authored ramp), **`Bomber`** (stamping with
Max/Min/Subtract blend), `Transpose` (transfers surface detail onto a silhouette you authored
elsewhere). **But Gaea 2 has no spline / path / road node at all** — the exact feature class a
motocross track needs. Vector spline road-and-river tools are the headline of Gaea 3.0.

**[unverified].** I could not reach quadspinner.com's Compare page directly (403 to automated fetch); the
tier table above came through the Order page. Whether Gaea 3.0 Early Access actually shipped is
**UNVERIFIED** — no blog post since June 2026.

### 2.2 World Creator (BiteTheBytes) — current version 2026.4

**[cited].** Three purchase models × four revenue tiers. Indie (revenue below $100,000):
**$59/yr subscription**, **$149 perpetual** (12 months of updates included, optional $45/yr renewal),
or **$379 lifetime** (free updates forever). All tiers are floating: "Use on 2 PCs (1 User)."
Professional (revenue to $200K) is $99/yr, $289 perpetual, $729 lifetime.
[world-creator.com/en/buy.phtml](https://www.world-creator.com/en/buy.phtml)

**[cited] — the free tier is a demo, not a licence.** Community Edition gives "Full Feature Access"
but "Fixed Sandbox Mode: Projects allow resolutions up to 4096 x 4096 pixels" and
**"No Export: Exporting and Sync-Exporting are disabled (Non-commercial use and learning only)."**

**[cited] — it has a Unity bridge.** Features list "Bridge to Unity & Godot", "Bridge to Unreal &
Houdini", "Bridge to Blender & Cinema 4D". Exports heightmaps in "RAW 8-bit, 16-bit and 32-bit,
EXR, ASC, XYZ, DEM formats and even as meshes", plus "Height, color, splat, normal, AO, biome,
simulation, spline and DEM data" and slope/elevation/flow/cavity/angle/rock masks.
[Features](https://www.world-creator.com/en/features.phtml)

**[cited] — workflow shape.** Explicitly **"Layer Based, No Nodes"** — deliberately not a node graph.
It does have **2D Terrain Stamping** ("Import any heightmap along with its corresponding color map
and stamp it directly onto your terrain") and **"Vector Tools for Paths, Rivers and Mountains"**.

**[cited] — the Unity bridge is a real Editor package, and it is maintained.** Installed as a
`.unitypackage`, used via **Window > World Creator Bridge**: hit Sync in World Creator, Synchronize
in Unity, and the terrain loads into the scene. The bridge writes `bridge.xml` (resolution, size,
min/max height), **`heightmap_x_y.raw` (uint16)**, `colormap_x_y.png`, `texturemap_x_y.png`, and
`splatmap_x_y.tga`. Parameters include **Split Threshold** (controls how many Unity terrain tiles
are produced), World Scale, **Material Type**, and **Import Layers** (individual TerrainLayers vs a
baked colour map). Objects marked "Is Entity" spawn as GameObjects. The bridge was updated
**16 June 2026**.
[Unity Bridge docs](https://docs.world-creator.com/reference/export/bridge-tools/unity-bridge.md)
· [News](https://www.world-creator.com/en/news.phtml)

**⚠️ [unverified] — the biggest open risk in this document.** The bridge docs name only
**Built-In RP → `Standard`** as a Material Type. **No documentation confirms URP or HDRP support,
or Unity 6 compatibility.** A custom-material option exists as a workaround. **This project is URP
on 6000.5.9f1 — verify with World Creator support before paying.**

**[cited] — it can ingest a heightmap back.** Add a **Stamp Layer** and drop a file into the Heightmap
slot: RAW 16/32, TIFF 8/16/32, PNG 8/16, EXR 16/32. Gotcha: "When importing RAW for 2D Stamp Layers,
World Creator estimates the proper dimensions assuming 16-bit, little-endian." Unity's 16-bit RAW
export with Windows byte order matches — but check the toggle.

**[cited] — the best shape toolkit of the three, and the reason it is a contender.**
**Primitive layers**: Square (with Round Corners), Polygon, Circle, Gradient, Donut, Helix, Star,
each with Falloff, Curve, Base Offset and Overwrite/Add/Min/Max operations.
**Path layers**: control points with a **Flatten** option ("flattens the heights of all control
points") plus Height Offset and per-point height dragging. Also **Advanced 3D Model Stamping**
(import a model as a shape layer), 2D Terrain Stamping, real-time brush sculpting, Custom Paint
drawable masks, River Generator, and **Terrain Conformity** (auto-flattens ground under placed
objects).

**[inferred].** Path layers with per-control-point height and Flatten are the closest thing any tool
here offers to "draw the line the bike should ride, and make the ground follow it." Combined with
the only maintained first-party Unity bridge and the lowest entry price, World Creator is the
strongest of the three external generators — conditional on the URP question.

**[cited] — corroborating adoption.** [Roland09/Terrain-Examples](https://github.com/Roland09/Terrain-Examples),
a Unity heightmap resource, recommends the World Creator sync tool as its import route.

### 2.3 Instant Terra (Wysilab)

**[cited].** Perpetual licences, prices exclude VAT, 14-day refund, 12 months of updates:

| Edition | Price | Terrain cap | Eligibility |
|---|---|---|---|
| Instant Terra | **$149** | up to **8k × 8k** | revenue under $500K |
| Instant Terra Unlimited | **$199** | unlimited | revenue under $500K |
| Instant Terra PRO | on request | unlimited | no condition |

All editions "Export as heightmaps or meshes" and "Export masks, simple or tiled export."
[wysilab.com/Editions_Pricing.html](https://wysilab.com/Editions_Pricing.html)

**⚠️ [cited] — apparently abandoned. Rule it out.** Version 2.5 shipped **March 2022 — no release in
~4.5 years**. Live docs are still titled "Instant Terra V2.5"; the product blog's newest post is
September 2019; the `instantterrapro.wysilab.com` domain (blog + release-note archive) no longer
resolves although the homepage still links to it; `wysilab.com/Download.html` 404s; the Unity export
forum thread's newest post is 2017. Windows-only. No free tier — one-month trial only.

**[cited] — no Unity plugin at all.** Plugins exist for Unreal (4.26/4.27/UE5) and Houdini only.
Masks export as **separate greyscale images with no packed RGBA splatmap export**, so you assemble
Unity splatmaps yourself. Tiled export nodes do offer an **overlap parameter** (overlap=1 makes
tiles share a row/column), which is a clean answer to Unity multi-terrain seams.

**[cited] — the shape tools are genuinely good, which makes the abandonment sad.**
`Constant elevation` (plateaus and landing zones), **`Slope`** (editable inclination and orientation
— ramps), `Cone`, `Half-sphere`, `Profile curve`, **`Painted mask`** (in-viewport 3D painting with
tablet pressure), and the 2.5 headline **`Road` node**: "the relief is deformed to adapt to the
position of the roads and ensure a horizontal section along the entire length", accepting any input
mask including a painted one. Stamping supports Add / **Subtract** (gaps) / Max / Min.

**[inferred].** Technically capable and well-suited to track building, but paying $149–199 for an
unmaintained Windows-only tool with no Unity plugin and a near-nonexistent Unity user base is poor
value against World Creator at the same price. **Eliminated.**

### 2.4 The category the ticket omitted: generators that run *inside* Unity

**[inferred] — this is the important omission.** Every tool in §2.1–B.3 is an external application
whose only Unity ingest is a heightmap file, and Unity's importer "replaces the existing heightmap"
(§1.1). Generators that run inside the editor and write `TerrainData` directly do not have that
problem, because they can choose what to overwrite.

**[cited] — MapMagic 2 (Denis Pahunov) is free, node-based, and solves the round-trip explicitly.**
The core asset is **free** on the Unity Asset Store, version 2.1.20 (Jul 2026), listed compatible
with Built-in / URP / HDRP. It is "a node based procedural and infinite game map generator";
"Each node on a graph represents a terrain or object generator: noise, voronoi, blend, curve,
erosion, etc."
[MapMagic 2 on the Asset Store](https://assetstore.unity.com/packages/tools/terrain/mapmagic-2-165180)

**[cited] — the Lock feature is the closest thing found to a real answer to criterion 4.** The lock
"works by reading terrain maps (including the grass/detail int masks) in locked areas before
generate starts; and writes these maps back to generated results after, before applying the new
generation." Documented caveat: "try not to modify a height greatly, otherwise you can get
noticeable borders between your unchanged, locked terrains and their neighbors."
[MapMagic wiki / Unity Discussions thread](https://discussions.unity.com/t/mapmagic-2-infinite-procedural-land-generator/787350)

**[unverified].** I could not reach the MapMagic wiki pages directly (404s on the URLs I tried) and the paid
add-on modules (Objects, Splines, Biomes, Brush) are on Fab, which blocks automated fetch.
**Module pricing is UNVERIFIED.** The free core's node graph and Lock are verified.

**[unverified].** The Asset Store listing shows compatibility with 2022.3.62f1 (a minimum, not a maximum),
and community threads report Unity 6 use, but **I did not find a first-party statement of
6000.5 support.** Verify before committing.

**[cited] — Gaia Pro (Procedural Worlds)** is the other in-Unity option, wizard-and-stamp driven
rather than node-graph, with a Unity 6 SKU. Not evaluated in depth because its model is
stamp-and-spawn rather than a re-runnable graph, which is the property this ticket cares about.
[Gaia for Unity 6](https://assetstore.unity.com/packages/tools/terrain/gaia-for-unity-6-264629)

### 2.5 The one tool built for this exact problem

**[cited] — Heightmap Studio** is "an intuitive terrain and track creation tool designed to generate
seamless, high-fidelity 16-bit heightmaps for any modern game engine", "engineered to handle the
brutal, micro-precise physics tracking of simulators like MX Bikes", and "universally compatible
with any engine that utilizes grayscale heightfields." Features: procedural terrain generation with
seeded presets, **spline track path drawing, jump placement and customization, rut and bump
carving**, sculpt/smooth brushes, a **non-destructive layer system**, and 16-bit grayscale PNG
export at 1025 or 2049. Browser-based or Windows download.
Licence: "Personal use is free. Commercial track releases, commissions, paid packs, studio work,
and monetized projects require a license."
[heightmapstudio.com](https://heightmapstudio.com/)

**[unverified] — the commercial licence price is UNVERIFIED.** The site states a licence is required for
commercial work but the `/license` and `/pricing` paths 404 to automated fetch; the price is behind
an in-app "License" button. **Ask the vendor before planning around this.**

**[inferred].** This is the only tool surveyed whose feature list is literally "jumps, ruts, bumps,
track splines" rather than "mountains, erosion, rivers". It is aimed at MX Bikes track modders, so
its output conventions (16-bit, 2ⁿ+1, 1025/2049) line up with Unity Terrain exactly. Its ceiling is
also its niche: it produces one heightmap, not a zone with biomes and scatter.

---

## 3. Houdini / Houdini Engine

### 3.1 Cost

**[cited].** Houdini 22 shipped 15 July 2026. **Houdini Indie is a subscription: $299 USD/yr or
$449 USD/2yr.** Eligibility: annual gross revenue **under $100K USD** *and* under $1M USD funding
received in the last 24 months. Max 3 Indie + 3 Engine Indie licences per entity.
[Compare products](https://www.sidefx.com/products/compare/)
· [Houdini Indie](https://www.sidefx.com/products/houdini-indie/)
· [Indie FAQ](https://www.sidefx.com/faq/indie-new/)

**[unverified].** `sidefx.com/buy/` does not render prices without login; the $299/$449 figures come from two
other sidefx.com pages that agree with each other. Treat as high-confidence but not from the buy page.

**[cited].** **Houdini Engine Indie is free and included with Indie** (3 licences). The paid Houdini
Engine ($525 Workstation / $795 Floating) is only needed for Maya/Max/batch/farm, not Unity —
Houdini Engine for Unity/Unreal is a free commercial licence, up to 10 per studio.
[Engine FAQ](https://www.sidefx.com/faq/houdini-engine-faq/)
· [Engine for Unreal and Unity](https://www.sidefx.com/community/houdini-engine-for-unreal-and-unity/)

**[cited] — there is no free way to evaluate this.** "The Houdini Engine plug-ins do not work with the
FREE Houdini Apprentice license", and digital assets created in Apprentice cannot be used with
Houdini Engine at all.
[Unity plug-in](https://www.sidefx.com/products/houdini-engine/plug-ins/unity-plug-in/)
· [Apprentice restrictions](https://www.sidefx.com/faq/question/apprentice-restrictions/)

**[cited] — licensing trap.** Indie saves limited-commercial assets that the *free commercial*
Engine-for-Unity licence will not open; you must install **Engine Indie** specifically, and having
the free commercial Engine licence installed breaks loading.
[Why can't I use Indie assets with Engine for Unity/Unreal licences?](https://www.sidefx.com/faq/question/why-cant-i-use-assets-created-in-houdini-indie-with-houdini-engine-for-unityunreal-licenses/)

### 3.2 Houdini Engine for Unity — it produces real TerrainData, and it round-trips

**[cited].** "This plug-in supports Unity 2018.4 LTS and newer versions, up to Unity 6.5" — 6000.5.9f1
is in range. (The [GitHub README](https://github.com/sideeffects/HoudiniEngineForUnity) is stale and
still says 2018.1+; the docs are authoritative.) URP terrain requires the material set to
`Universal Render Pipeline/Terrain/Lit`.
[About the Unity plug-in](https://www.sidefx.com/docs/houdini/unity/about.html)

**[cited] — this is the differentiator.** Houdini heightfields generate **actual Unity Terrain
components with real `TerrainData.asset` files**, not meshes. The `height` layer becomes the
heightmap; every other layer becomes a TerrainLayer plus splatmap. Terrain tiling is supported
(`unity_hf_tile`). Scattering emits real `TreePrototype`/`TreeInstance` data, Unity detail layers,
or prefab instances.
[Terrain basics](https://www.sidefx.com/docs/houdini/unity/terrain/basics.html)
· [Terrain scattering](https://www.sidefx.com/docs/houdini/unity/terrain/scattering.html)

**[cited] — the plug-in leaves no runtime dependency.** The `HEU_HoudiniAsset` data is tagged
EditorOnly and stripped at build; output GameObjects carry plain MeshFilter/MeshRenderer.
[Assets](https://www.sidefx.com/docs/houdini/unity/assets.html)

**[cited] — constraints.** Terrain must be **square**; heightmap resolution must be power-of-2 + 1 or
"Unity will automatically change to the next lowest power of 2 size, which means the height values
will not be properly applied". (same page)

**[cited] — round-trip into Houdini exists.** Supported HDA inputs include "HDA (as node connection),
Unity Mesh, Curve, Spline …, **Terrain** …, Bounding Box, Tilemap". A Unity Terrain assigned to an
HDA input becomes a Houdini heightfield: `height` plus one layer per TerrainLayer splatmap, with
`unity_hf_terraindata_file` / `unity_hf_terrainlayer_file` preserved as string attributes so the
settings survive the round trip. Limitation: "Only square sized terrain are currently supported."
**Trees and detail layers are not documented as transferring back in.**
[Inputs](https://www.sidefx.com/docs/houdini/unity/inputs.html)
· [Terrain inputs](https://www.sidefx.com/docs/houdini/unity/terrain/inputs.html)

**[cited] — re-cook semantics.** "On Recook, the TerrainData, TerrainLayers, and splatmaps are reused
(and settings untouched) **unless overridden by the generated height field**." So settings you did
not drive from Houdini survive, but **any sculpting on a layer the HDA outputs is overwritten**.
There is no merge of hand edits into HDA output. The escape hatches are **Bake GameObject**,
**Bake Prefab**, and **Bake Update** (re-push HDA changes onto previously-baked targets by matching
output GameObject names).
[Terrain basics](https://www.sidefx.com/docs/houdini/unity/terrain/basics.html)
· [Assets](https://www.sidefx.com/docs/houdini/unity/assets.html)

**[cited] — Engine-free fallback.** `heightfield_output` writes 16-bit PNG or 32-bit float EXR
heightmaps for manual Unity import, no Engine licence involved.
[heightfield_output](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_output.html)

### 3.3 Can Houdini make *rideable* shape?

**[cited].** `heightfield_project` "sends rays from the height field to the surface and (if it hits)
uses the distance between the points to modify the height field value", with Combine Method
**Maximum** (only add — ramps and berms) or **Minimum** (only carve — ruts and trenches). Its
companion `heightfield_maskbyobject` projects the 2D outline as a mask, with Gaussian/Box/Expand/
Shrink blur for shoulder feathering.
[heightfield_project](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_project.html)
· [heightfield_maskbyobject](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_maskbyobject.html)

**[cited].** Rideability can be *audited* with `heightfield_maskbyfeature` (Min/Max Slope Angle in
degrees) and partly *enforced* with `heightfield_slump`'s **Repose Angle** — "the maximum slope,
measured in degrees from the horizontal, at which loose solid material will remain in place without
sliding."
[maskbyfeature](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_maskbyfeature.html)
· [slump](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_slump.html)

**[cited] — the closest real case study.** *Offroad Heat* (Luminet, **Unity**, 4-person core team):
3 biomes across 19 levels in **6 weeks**, ~80% procedural. Roads "generated from a single hand-drawn
curve by a sweep node", then "projected down against the raw input terrain giving it the intended
elevations." They converted premade Unity terrains into Houdini heightfields and could "convert any
level to any biome in minutes."
[Using Houdini for Large Terrains (80.lv)](https://80.lv/articles/using-houdini-for-large-terrains-006sdf)

**[cited] — the counter-evidence.** The one solo Unity developer on record got a low-poly cliff with
collision mesh in about a week from zero experience, then **abandoned the approach**, citing
Houdini Engine *for Unity* integration problems and unreliable curve management.
[Houdini for a solo game developer](https://www.sidefx.com/forum/topic/48018/)

**[cited] — free, directly relevant training.** *PDG for Indie Gamedev* (Kenny Lammers) — 6h18m,
Unity-based, using Houdini Engine for Unity, with a "Create Terrains" section and a
"Paths & Roads" section covering a path system that modifies terrain.
[PDG for Indie Gamedev](https://www.sidefx.com/learn/collections/pdg-for-indie-gamedev/)

**[cited] — the gaps.** There is **no dedicated racetrack tool** in SideFX Labs; Labs Road Generator
"builds mesh only, does not deform terrain, and has no banking/slope/grade controls". Banking has no
heightfield node — every documented implementation is VEX on a swept ribbon. Jump ramps have
essentially no published prior art.
[SideFX Labs](https://www.sidefx.com/docs/houdini/labs/)
· [Labs Road Generator](https://www.sidefx.com/docs/houdini/nodes/sop/labs--road_generator.html)

**[cited] — and heightfields are 2.5D too.** One height per voxel, so undercut berms, overhangs,
tunnels and bridges cannot be represented in the heightfield and must be separate mesh — the same
constraint Unity Terrain has.

**[inferred].** Houdini is the only tool evaluated that offers a *true* round-trip (Unity Terrain in,
Unity TerrainData out) and the only one that can express "carve a rideable line along this spline
with a graded slope" as a repeatable rule. It is also the only one costing $299/yr with no free
trial of the Unity path, requiring months of ramp-up, and requiring you to build the
motocross-specific parts (banking, grade caps, ramps) yourself in VEX.

---

## 4. Round-trip — the deciding criterion

### 4.1 The answer, per tool

| Tool | Regenerate vs hand edits | Can a Unity-side edit go *back* in? |
|---|---|---|
| Unity Terrain + Terrain Tools | N/A — there is no "generate" step to re-run; every edit is the authoritative state | N/A |
| Gaea / World Creator / Instant Terra | **Destroys all height hand-edits.** Re-import replaces the heightmap wholesale | **Yes, but only as pixels.** Each has a heightmap-ingest node — Gaea `File` (Primitive), World Creator **Stamp Layer**, Instant Terra `Import terrain from file` — so a Unity-exported heightmap can re-enter the graph as a *source image*. It does not re-enter as editable structure, and every loop re-quantises to 16-bit |
| MapMagic 2 | **Preserves hand edits inside locked areas**, with seam artefacts at lock borders | Partially — locked regions are read back and re-applied |
| Houdini Engine | Re-cook **overwrites any layer the HDA outputs**; untouched settings survive. Bake detaches | **Yes** — a Unity Terrain can be an HDA input, height + splatmaps, with file-path attributes preserved |
| Heightmap Studio | Internal layer system is non-destructive **inside the tool**; the Unity handoff is still a file | **No** |

### 4.2 The insight that reframes the criterion

**[inferred] (high confidence).** "Does a regenerate destroy hand edits" is the wrong granularity.
A Unity terrain is not one blob — it is five independent channels plus the scene:

| Channel | Lives in | Survives a heightmap re-import? |
|---|---|---|
| Heights | `TerrainData` heightmap | **No** |
| Splat / terrain layers | `TerrainData` alphamaps | Yes |
| Trees | `TerrainData.treeInstances` | Yes (but float/sink if the surface moved) |
| Detail / grass | `TerrainData` detail layers | Yes (same caveat) |
| Holes | `TerrainData` holes texture | Yes |
| **Ramps, set pieces, vignettes, triggers, spawns** | **the scene, as GameObjects** | **Yes, entirely untouched** |

**This is the finding that decides the ticket.** If the authored playground features — the things
that exist *to be ridden* — are **GameObjects with their own colliders rather than heightmap
sculpting**, then regenerating the terrain base cannot destroy them. The round-trip problem
evaporates, not because a tool solved it, but because the valuable work was never in the channel
that gets overwritten.

### 4.3 The delta-layer script (buildable, tool-agnostic, ~100 lines)

**[inferred], grounded in the cited API.** For the height edits that genuinely must live in the
heightmap (blending a ramp's run-up into the hill, carving a rut line), `SetHeights` accepts a
sub-region and heights are plain normalized floats (§1.1). So an editor script can:

1. after each generate, snapshot `GetHeights()` as `base`;
2. later, compute `delta = current − base` and save it as an asset;
3. on the next generate, write `newBase + delta`.

Hand height edits then survive regeneration regardless of which tool produced `base`. This is worth
building early and is the cheapest insurance in this whole document.

### 4.4 The file-based baseline

**[cited].** For every file-based generator (Gaea, World Creator, Instant Terra, and any Houdini
heightfield exported as an image), the Unity ingest is `Import Raw` or the Toolbox importer, and
Unity's own docs say the import replaces the terrain's height data wholesale.
[Working with Heightmaps](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-Heightmaps.html)

**[inferred] (high confidence) — the round-trip question decomposes by channel.** A heightmap
re-import only destroys the *height* channel. Everything else in a Unity terrain lives in separate
arrays (`alphamaps`, `detailLayers`, `treeInstances`, `holes`) and everything placed as a
GameObject — ramps, set pieces, vignettes, spawn points, triggers — lives in the *scene*, not in
`TerrainData`. Those all survive a regenerate untouched, though tree/detail instances that were
snapped to the old surface will float or sink.

**[inferred] — this makes a delta-layer workflow buildable in-house.** Because `SetHeights` accepts
a sub-region and heights are plain normalized floats, a ~100-line editor script can:
1. snapshot `GetHeights()` immediately after each generate (`base`),
2. at any later time compute `delta = current - base` and save it as an asset,
3. on the next regenerate, write `newBase + delta`.
Hand height edits then survive regeneration. This is the single highest-leverage piece of tooling
identified by this research, and it is tool-agnostic — it works regardless of which generator
produces `base`.

---

## 5. Collision fidelity

### 5.1 Terrain colliders are exact, and full-resolution regardless of visual LOD

**[cited].** "PhysX generates Terrain colliders from the heightmap data of the corresponding Terrain,
rather than from a pre-defined shape or a Mesh." They "match the shape of a Terrain exactly, for
extremely accurate Terrain collision simulation" and are "the correct collider choice for a Terrain
in almost all cases."
[Terrain colliders](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-colliders-introduction.html)

**[cited].** `heightmapPixelError` is a *rendering* LOD parameter — "an approximation of how many
pixels the terrain will pop in the worst case when switching lod". The documented way to reduce
*collider* cost is entirely separate: duplicate the TerrainData, lower its resolution, and assign
that to the Terrain Collider's `terrainData` field.
[heightmapPixelError](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Terrain-heightmapPixelError.html)
· [Terrain colliders](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-colliders-introduction.html)

**[inferred] (high confidence).** The collider therefore always uses full heightmap resolution
regardless of Pixel Error, and visual/physical divergence grows with Pixel Error at distance.
The `TerrainCollider.terrainData` field is the sanctioned decoupling point if you ever want
physics and rendering at different resolutions.

### 5.2 Tunnelling — the real hazard for a fast bike

**[cited].** Since 2018.3, terrain contacts go through the general mesh path:
"A new algorithm is now used to compute contacts with terrains. It used to be special-cased, but
now it's the same mesh-primitive code that is used with MeshColliders. **However, it doesn't
support `TerrainData.thickness`, which means the tunneling effect with terrains is now possible.**
To avoid this, you need to enable continuous collision detection on the fast moving objects."
[Upgrading to 2018.3](https://docs.unity3d.com/2018.3/Documentation/Manual/UpgradeGuide20183.html)

**[cited].** `TerrainData.thickness` is now obsolete — "Terrain thickness is no longer required by
the physics engine", replacement guidance: "Set appropriate continuous collision detection modes to
fast moving bodies."
[TerrainData.thickness](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData-thickness.html)

**[cited] — corroborated upstream.** PhysX's own docs: "the heightfield surface has no thickness so
fast-moving objects may tunnel if CCD is not enabled."
[PhysX Geometry — Height Fields](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/Geometry.html)

**[cited] — and here is the constraint that shapes the bike's collider design.**
"Continuous Collision Detection is only supported for Rigidbodies with Sphere-, Capsule- or
BoxColliders."
[Rigidbody.collisionDetectionMode](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html)

**[inferred] (high confidence).** A bike chassis built from a convex MeshCollider **cannot** use
sweep-based CCD against terrain. Either model the chassis and wheels from primitives
(sphere wheels, capsule/box chassis) and set `ContinuousDynamic`, or accept
`ContinuousSpeculative`, which is weaker against fast thin-surface tunnelling. For a game whose
core loop is *launching a bike into the air and landing it hard*, this is not a detail to defer.

### 5.3 Edge catching (ghost collisions)

**[cited] (adjacent product — see caveat).** Unity defines the failure mode precisely: "Ghost
collisions … occur when dynamic objects moving across a surface bump into unintended collisions at
the boundaries or connections between adjacent physics colliders." Listed causes include adjacent
colliders being processed independently, separating-plane misalignment at edge transitions, and
"If an object moves rapidly between frames, it might erroneously interact with shapes it shouldn't
have encountered."
[Ghost Collisions](https://docs.unity3d.com/Packages/com.unity.physics@6.5/manual/ghost-collisions.html)

**Caveat (CITED).** That page documents the **DOTS `com.unity.physics`** package. Its named
mitigations — ghost vertices, narrowphase contact modification — are **DOTS features and are not
exposed for built-in PhysX TerrainCollider**. The mechanism transfers; the fixes do not.

**[cited] — Unity's example is literally this game.** The same page: "a car driving over a terrain mesh
might bounce unexpectedly at triangle edges", most likely "when rigid bodies move at high speeds
relative to the size of the colliders they interact with."

**[cited] — contact offset must go DOWN, not up.** Default `Physics.defaultContactOffset` is 0.01, and
"Colliders whose distance is less than the **sum** of their contactOffset values will generate
contacts" — so two default colliders trigger at 0.02 m apart. Raising it makes a buried vertical
seam face generate contacts from further away, i.e. worse. Practitioners report fixing seam-launch
by lowering it to 0.001–0.005.
[Physics.defaultContactOffset](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Physics-defaultContactOffset.html)
· [Ghost collisions on adjacent box colliders](https://discussions.unity.com/t/ghost-collisions-on-adjacent-box-colliders/781680)

**[cited] — the mechanism, stated cleanly** (user arkano22): when a ball sinks slightly into the floor
it "ends up closer to the side wall of the collider than the top side (floor), so the collision
system … deems it more appropriate to get the ball out … pushing it to the side instead of pushing
it upwards." Suggested fixes include sweep CCD, lower contact offset, contact modification, and
"using raycasting to artificially move the ball upwards before actual collision detection takes place."
[How to handle ghost collisions](https://discussions.unity.com/t/how-to-handle-ghost-collisions/948686)

**[cited] — `Physics.ContactModifyEvent` is the sanctioned fix and exists in Unity 6.** Requires
`Collider.hasModifiableContacts`; lets you change impulses and normals and drop individual contact
points with `pair.IgnoreContact(i)`. Fires from any thread, possibly multiple times per frame;
CCD pairs arrive on a separate `Physics.ContactModifyEventCCD`.
[Physics.ContactModifyEvent](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Physics.ContactModifyEvent.html)

### 5.4 WheelCollider — and Unity's own advice for racing games

**[cited].** WheelCollider is a single raycast: "PhysX casts the ray down the local Y-axis along the
direction of suspension through the center of the wheel." And: "The ground collision geometry must
be as smooth as possible to ensure a smooth and accurate simulation."
[Wheel colliders introduction](https://docs.unity3d.com/6000.3/Documentation/Manual/wheel-colliders-introduction.html)

**[cited] — Unity's manual explicitly tells racing developers to build a separate collision mesh:**
> "Because cars can achieve large velocities, getting race track collision geometry right is very
> important. Specifically, the collision mesh should not have small bumps or dents that make up the
> visible models (e.g. fence poles). **Usually a collision mesh for the race track is made separately
> from the visible mesh, making the collision mesh as smooth as possible.** It also should not have
> thin objects … You might want to decrease physics timestep length in the Time window to get more
> stable car physics, especially if it's a racing car that can achieve high velocities."

[WheelCollider manual](https://docs.unity3d.com/2020.2/Documentation/Manual/class-WheelCollider.html)

**[inferred] — this is a direct argument against Terrain for the ride surface.** A TerrainCollider
structurally *cannot* be a separate smooth collision mesh: it **is** the render surface, at full
resolution, by construction (§5.1).

**[cited] — WheelCollider's terrain weakness.** On slopes the wheel circumference contacts ground the
central ray misses, so "the wheel colliders will sink into the ground up to the point where the
central raycast position is in contact with the ground." Community remedy: cast several rays
radially from the wheel centre and combine — i.e. write your own suspension.
[Using WheelCollider over bumpy terrain](https://discussions.unity.com/t/using-wheelcollider-to-move-a-car-over-extremely-bumpy-terrain/843)

**[cited] — a shipped Unity arcade racer abandoned WheelCollider**, building custom suspension from
"raycasts down … in the shape of the tires" to get arcade handling and smooth terrain transitions.
[NEODRIVE devlog: How to exit collider hell](https://sevencrane.itch.io/neodrive/devlog/1070030/how-to-exit-collider-hell)

**[cited] — free smoothing that only Terrain gives you.** `TerrainData.GetInterpolatedNormal(x, y)`
computes normals at surrounding grid points with a Sobel filter then bilinearly interpolates them —
a smooth normal explicitly distinct from the faceted collision normal.
[GetInterpolatedNormal](https://docs.unity3d.com/ScriptReference/TerrainData.GetInterpolatedNormal.html)

**[inferred].** Raycast against the collider for the *contact point*, and read
`GetInterpolatedNormal` for the *orientation and friction basis*. There is no equivalent free
smoothed normal for an arbitrary MeshCollider. This is a genuine, under-appreciated argument
*for* keeping the base surface as Terrain.

### 5.5 Vertical quantisation — the number that constrains zone height

**[cited].** Unity's terrain heightmap is 16-bit (`R16_UNorm` on DX11; packed into `R8G8_UNorm` on
some APIs, hence `UnpackHeightmap()`), but the shader documentation reveals only *half* the range
is used: heights are clamped "between 0.0f and 0.5f because the Heightmap itself is signed but is
treated as an unsigned texture when rendering the Terrain", with
**`kMaxHeight = 32766.0f / 65535.0f`**.
[Custom Terrain Tool shaders](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/create-use-custom-shaders.html)
· [heightmapTexture](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData-heightmapTexture.html)

**[inferred] (arithmetic on the cited constant).** Effective precision is ~15 bits: **32766 levels**
across the full height range. `step = TerrainData.size.y / 32766`.

| Terrain Height | Vertical step |
|---|---|
| 150 m | 4.6 mm |
| 200 m | 6.1 mm |
| 300 m | 9.2 mm |
| 400 m | 12.2 mm |
| **600 m** | **18.3 mm** |
| 1000 m | 30.5 mm |

**[inferred].** Below ~1 cm the step is irrelevant next to tyre radius and suspension travel.
Above ~2 cm it becomes a visible micro-terrace on gentle slopes and a plausible source of wheel
chatter. **Set Terrain Height to the smallest value the zone's relief actually needs** — this costs
nothing and is free precision. A motocross zone almost certainly needs ≤ 300 m of relief.

**[cited], corroborating.** PhysX heightfield samples are "a 16 bit integer height value, two
materials … and a tessellation flag", scaled by `heightScale` — so the collision surface carries
the same quantisation as the visual one, not a finer one.
[PhysX Geometry](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/Geometry.html)

### 5.6 Horizontal resolution and memory — the hard scale limit

**[cited].** `heightmapResolution` is clamped to one of **33, 65, 129, 257, 513, 1025, 2049, 4097**.
**4097 is the ceiling.** Terrain Width/Length range 1–100,000; Terrain Height range 1–10,000.
[heightmapResolution](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData-heightmapResolution.html)
· [Terrain Settings](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-OtherSettings.html)

**[inferred] (arithmetic; heightmap at 2 B/sample, PhysX heightfield at 4 B/sample per §5.4):**

| Tile size | Resolution | Sample spacing | Heightmap | Collider |
|---|---|---|---|---|
| 1024 m | 2049 | **0.50 m** | 8 MB | 16 MB |
| 1024 m | 4097 | **0.25 m** | 32 MB | 64 MB |
| 2048 m | 2049 | 1.00 m | 8 MB | 16 MB |
| 2048 m | 4097 | **0.50 m** | 32 MB | 64 MB |
| 4096 m | 4097 | 1.00 m | 32 MB | 64 MB |

**[cited].** Multi-tile is well supported: Auto Connect + Grouping ID stitches neighbours, "Fill
Heightmap Using Neighbors … ensures that the height of the new tile's edges match up with adjacent
tiles", and "At run time, the Terrain system automatically blends the tessellation and normal map
of connected tiles."
[Create Neighbor Terrains](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-CreateNeighborTerrains.html)

**[inferred] — the finding that reframes the ticket.** A 4 km zone at 0.5 m sampling needs 2×2 tiles
of 2048 m @ 4097 ≈ **128 MB heightmap + 256 MB collider**. That does not fit the "no streaming"
constraint comfortably. A 2 km zone at 0.5 m is ~1/4 of that and does. **The zone's physical extent
and the terrain sampling density trade directly against each other, and 0.5 m sampling still cannot
express a crisp jump lip.** Fine ramp geometry has to come from mesh, or the zone has to shrink.

---

## 6. Is heightmap terrain the right representation?

**[cited] — a dedicated motocross simulator answers this directly.** MX Bikes (PiBoSo) builds tracks
from a heightfield: "Heightmap must be 16bpp IBM RAW. The heightmap dimensions must be a power of
2 +1", recommended **2049×2049**, with collision sampling the same grid (`samples_x = 2049`,
`samples_y = 2049`). On a ~480 m track that is roughly **0.23 m per sample**. Scenery is separate
mesh (`scenery.edf` from FBX) with its own collision types.
[MXB Track Creation Guide](https://docs.piboso.com/wiki/index.php/MXB_Track_Creation_Guide)

**[inferred] (high confidence).** A 16-bit heightfield is demonstrably adequate for dirt-bike
physics — *at ~0.25 m sampling*. That is the fidelity bar the genre sets, and per §5.5 Unity can
hit it only over a 1 km tile. This is strong evidence that **jumps and ruts belong in the
heightmap** (that is where MX Bikes puts them) and **props/scenery belong in mesh**.

**[cited].** Unity's own documented answer for overhangs, caves and tunnels is terrain holes:
"use the Paint Holes tool to hide a part of the terrain. Then use other Editor tools, such as
ProBuilder, to create the cave or overhang geometry." Critically for physics: "A hole in the
terrain is automatically excluded from the Terrain Collider, so that characters and objects can
fall through it."
[Paint Holes](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-PaintHoles.html)

**[cited] — URP gotcha worth writing down now.** "enable **Terrain Holes** in the URP asset.
Otherwise, terrain holes disappear when you build your application." Hole mask resolution must
equal heightmap resolution minus 1. Hole edges are texture-based and alias; hide them with geometry.
(same page)

**[inferred].** So heightmap terrain does not actually foreclose overhangs, caves or tunnels —
it just means they are authored as mesh through a hole rather than sculpted. Given the project
already needs ProBuilder-class mesh authoring for ramps and set pieces, this costs no new tool.

**[cited].** ProBuilder 6.0.9 supports "in-scene level design, prototyping, collision Meshes, and
play-testing" with parametric shapes and export to external DCC.
[ProBuilder manual](https://docs.unity3d.com/Packages/com.unity.probuilder@6.0/manual/index.html)

### 6.1 The closest shipped precedent left Unity Terrain

**[cited] — Lonely Mountains: Downhill** is a fast off-road bike game made in Unity, and it started on
Unity Terrain and deliberately abandoned it. "When we started the development, we build the first
levels with the standard Unity terrain which was then automatically converted to a low-poly mesh" —
and they dropped it because "this workflow was really impractical and had a lot of restrictions on
the level design."

What they replaced it with: a custom terrain system, one mountain per scene, "flat terrain divided
into small chunks (a few hundred of them)", "each chunk has its own LOD mesh". Authoring is
spline-first — they "normally start by using a spline tool, which automatically sets the height of
the underlying terrain", then refine with custom sculpting tools. Collision is decoupled from
rendering entirely: "a combination of raycasts and some rigid bodies moving along with the bike just
for collision detection", chosen over standard physics-engine behaviour for stability.
[80.lv: Level & Game Production](https://80.lv/articles/level-game-production-lonely-mountains-downhill)
· [Road to the IGF: Megagon Industries](https://www.gamedeveloper.com/disciplines/road-to-the-igf-megagon-industries-i-lonely-mountains-downhill-i-)

**[inferred].** They kept the heightfield *authoring metaphor* (a spline drives the underlying
height) while discarding the heightfield *representation*. That is a live migration path if Terrain
proves limiting later — and it costs nothing to keep open.

### 6.2 The one strong argument for heightmap: deformable ruts

**[cited].** Persistent real-time terrain deformation is the MX vs ATV series signature — vehicles dig
"deep, persistent ruts that physically alter every track"; it "goes beyond visuals — it has an
actual physical effect"; ruts don't fade, so the track changes lap to lap. Branded "Real-Time
Deformation 2.0" / "Rhythm Racing 2.0". Notably it was retrofitted **per-track**, not universally.
[Vital MX](https://www.vitalmx.com/videos/features/MX-vs-ATV-Reflex-Terrain-Deformation,822/GuyB,64)
· [Dirt Wheels](https://dirtwheelsmag.com/mx-vs-atv-reflex-redefines-racing-with-revolutionary-rhythm-racing-2-0-physics/)
· [Steam community thread on the per-track retrofit cost](https://steamcommunity.com/app/520940/discussions/0/1743346190280181977/)

**[inferred] (high confidence).** Per-XZ displacement written by wheel contacts, read back by physics,
accumulating persistently and bounded in memory *is* the canonical heightfield use case. A sculpted
mesh cannot be deformed this way cheaply. **This is the one genuine reason to keep the ride surface
as a heightfield** — and it comes from the studio that made the game this project is a successor to.
Rainbow Studios made both.

**GAP.** No primary Rainbow Studios engineering source found. Next lead: Google Patents,
assignee THQ Inc. / Rainbow Studios, terms "terrain deformation" / "height field" / "rut".

### 6.3 Uniform mesh terrain is not affordable; adaptive mesh is

**[inferred] (arithmetic; PhysX publishes no bytes-per-triangle figure, so ~35–50 B/tri is an
estimate from the documented BVH34 structures).**

- A **uniform** mesh at 0.5 m over 2048 m = 2 × 4096² = **33.6 M triangles ≈ 1.2–1.5 GB.** Dead.
  For *uniform* coverage a heightfield beats a mesh by roughly 15–25×. This is the legitimate,
  decisive argument for heightfield as the base.
- An **adaptive authored** collision mesh — 90% of the area at 4–8 m triangles plus ~150 authored
  features at 1.5–3 k tris each — lands at **300–600 k triangles ≈ 12–25 MB**, with exact
  centimetre-accurate ramp lips and native overhangs.

**[inferred] — the punchline.** Uniform density is what makes mesh terrain expensive, and you do not
need uniform density where nothing happens. A 1025 terrain base (~6 MB) plus ~400 k-tri authored
collision mesh (~15 MB) is about **21 MB of collision for a 2 km zone** — cheaper *and* sharper than
a 4097 terrain at ~101 MB.

### 6.4 The seam where a mesh ramp meets terrain

**[cited] — and it refutes the obvious fix.** A developer who sank a ramp mesh into the ground
diagnosed it themselves: "My ramp with the mesh collider was inset by a few units into the ground
surface. This caused the ball to take both the ground surface and the ramp colliders into account
where these two colliders were very close to each other causing a 'double' collision."
[Ball collides instead of rolls onto ramp](https://discussions.unity.com/t/ball-collides-instead-of-rolls-onto-ramp-with-mesh-collider-answered/915533)

**[inferred] — the seam problem has two independent halves:**
1. **Double contacts** where the mesh footprint overlaps live terrain collider. → fix with a
   **terrain hole** under the footprint.
2. **A buried near-vertical seam face** whose contact normal points sideways, so the solver ejects
   the wheel laterally instead of upward. → fix with **lower contact offset** plus a feathered
   skirt so no near-vertical face sits within wheel contact range.

**[cited] — terrain-to-mesh blending assets are VISUAL ONLY.** MicroSplat · Terrain Blending ($20):
"just add a component to your object and it will blend with the terrain beneath it", implemented as
a depth snapshot from an orthographic camera used to reconstruct world position in the fragment
shader.
[MicroSplat Terrain Blending](https://assetstore.unity.com/packages/tools/terrain/microsplat-terrain-blending-97364)

**[inferred], stated plainly.** MicroSplat will make the ramp look like it grew out of the dirt while
the wheel still slams into an invisible lip. Buy it as the *second* half of the fix, after collision
is smooth. **No asset on the Asset Store advertises solving the physics seam** — that is geometry
discipline plus physics tuning.

**[cited] — Mesh Stamp is the sanctioned "flatten terrain under the mesh" step.** Stamp your actual
ramp mesh with **Behaviour: Min** so it only carves down and never pushes terrain up through the
ramp, then place the real mesh on top; **Slope Flatten** conditions the run-up and landing.
[Stamp Terrain](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/stamp-terrain.html)

**[cited] — a free terrain→mesh escape hatch exists** if Terrain is ever outgrown:
**FastTerrainToMesh** (unitycoder, free, commercial use permitted) converts Terrain to optimised
mesh chunks with splatmaps, LOD generation, a URP-lit variant, tree/detail export, and
"Mesh collider and static flag options."
[FastTerrainToMesh](https://github.com/unitycoder/FastTerrainToMesh)

---

## Open questions

1. **Zone extent is still guessed, not measured.** "4–5 minutes to cross" resolves to anywhere
   between 1.4 km and 5.4 km depending on assumed average speed, and the memory maths changes
   completely across that range. **Grey-box a 2048 m zone and time an actual crossing** before
   committing to terrain resolution. This is the cheapest experiment in the whole plan.
2. **World Creator URP / Unity 6 support** — undocumented, and it is the deciding fact for the
   leading paid option. Ask the vendor.
3. **Heightmap Studio commercial licence price** — stated as required, never published. Ask.
4. **MapMagic 2 on Unity 6000.5** — Asset Store lists 2022.3 as minimum; no first-party Unity 6
   statement found. Paid module pricing on Fab unverified (Fab blocks automated fetch).
5. **Are deformable ruts in scope?** If yes, that locks the ride surface to a heightfield and
   changes this recommendation's balance considerably. It is also the single most distinctive
   thing Rainbow Studios did after MM2.
6. **Did Gaea 3.0 Early Access actually ship?** Targeted July 2026; no vendor post since June 2026.
   Its spline road tools plus promised Unity plugin would materially improve Gaea's standing.
7. **Vertical quantisation: 32766 or 65535 levels?** The `kMaxHeight = 32766/65535` constant in
   Unity's Terrain Tools shader docs implies ~15 effective bits, but this was not stated outright
   anywhere. **Verify empirically** — write a ramp of known slope via `SetHeights`, raycast it, and
   measure the step. The answer changes the recommended Terrain Height by 2×.
8. **Rainbow Studios' terrain deformation patent** — not retrieved. Google Patents, assignee
   THQ Inc. / Rainbow Studios.
9. Unverified: exact PhysX bytes-per-triangle for cooked BVH34 meshes (no published figure), and
   whether Unity keeps a CPU-side float copy of the heightmap in players (would add R²×4 B to the
   memory tables).

---

## Sources

89 unique URLs are cited inline above. The ones that carried the most weight:

**Unity primary docs (6000.5)**
- [Create and edit Terrains](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-UsingTerrains.html)
- [Working with Heightmaps](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-Heightmaps.html) — "importing … effectively replacing the existing heightmap"
- [Terrain Settings reference](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-OtherSettings.html)
- [Paint Holes](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-PaintHoles.html) — holes excluded from the collider; URP toggle required
- [Create Neighbor Terrains](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-CreateNeighborTerrains.html)
- [Introduction to Terrain colliders](https://docs.unity3d.com/6000.5/Documentation/Manual/terrain-colliders-introduction.html)
- [Terrain collider component](https://docs.unity3d.com/6000.5/Documentation/Manual/class-TerrainCollider.html)
- [TerrainData](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData.html) · [SetHeights](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData.SetHeights.html) · [heightmapResolution](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData-heightmapResolution.html) · [thickness (obsolete)](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/TerrainData-thickness.html) · [GetInterpolatedNormal](https://docs.unity3d.com/ScriptReference/TerrainData.GetInterpolatedNormal.html)
- [Rigidbody.collisionDetectionMode](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-collisionDetectionMode.html) — CCD is Sphere/Capsule/Box only
- [Physics.defaultContactOffset](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Physics-defaultContactOffset.html) · [ContactModifyEvent](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Physics.ContactModifyEvent.html)
- [Wheel colliders introduction](https://docs.unity3d.com/6000.3/Documentation/Manual/wheel-colliders-introduction.html) · [WheelCollider manual (race track collision geometry advice)](https://docs.unity3d.com/2020.2/Documentation/Manual/class-WheelCollider.html)
- [Upgrading to Unity 2018.3](https://docs.unity3d.com/2018.3/Documentation/Manual/UpgradeGuide20183.html) — terrain tunnelling is now possible
- [Ghost Collisions (DOTS physics — mechanism transfers, fixes do not)](https://docs.unity3d.com/Packages/com.unity.physics@6.5/manual/ghost-collisions.html)

**Terrain Tools package**
- [Changelog (5.3.3, 2026-08-19)](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/changelog/CHANGELOG.html) · [package.json](https://raw.githubusercontent.com/needle-mirror/com.unity.terrain-tools/master/package.json)
- [Paint Terrain tools](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/paint-terrain.html) · [Stamp Terrain / Mesh Stamp](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/stamp-terrain.html) · [Brush Mask Filters](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/brush-mask-filters.html) · [Terrain Toolbox](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/terrain-toolbox.html)
- [Custom Terrain Tool shaders — `kMaxHeight = 32766/65535`](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.3/manual/create-use-custom-shaders.html)

**Unity's terrain roadmap**
- [Terrain system improvements (staff: world-building work deprioritized)](https://discussions.unity.com/t/terrain-system-improvements/1637270)
- [The Unity Engine roadmap — Unite 2025](https://discussions.unity.com/t/the-unity-engine-roadmap-unite-2025/1696495)
- [Unity 7 roadmap, Unite Seoul 2026](https://unity.com/news/unity-7-roadmap-revealed-at-unite-seoul)

**Vendor pricing (fetched Aug 2026)**
- [Gaea order page](https://quadspinner.com/Order) · [Gaea changelog](https://quadspinner.com/Download/Changelog) · [Gaea → Unity output node](https://docs.gaea.app/guides/use-in/engines/unity.html) · [Gaea 3.0](https://quadspinner.com/Gaea3)
- [World Creator buy page](https://www.world-creator.com/en/buy.phtml) · [Unity Bridge docs](https://docs.world-creator.com/reference/export/bridge-tools/unity-bridge.md) · [Features](https://www.world-creator.com/en/features.phtml)
- [Instant Terra editions and pricing](https://wysilab.com/Editions_Pricing.html)
- [Houdini compare products](https://www.sidefx.com/products/compare/) · [Houdini Indie](https://www.sidefx.com/products/houdini-indie/) · [Indie FAQ](https://www.sidefx.com/faq/indie-new/)
- [Heightmap Studio](https://heightmapstudio.com/)
- [MapMagic 2 (free)](https://assetstore.unity.com/packages/tools/terrain/mapmagic-2-165180)

**Houdini Engine for Unity**
- [About the Unity plug-in (supports up to Unity 6.5)](https://www.sidefx.com/docs/houdini/unity/about.html) · [Terrain basics](https://www.sidefx.com/docs/houdini/unity/terrain/basics.html) · [Terrain inputs](https://www.sidefx.com/docs/houdini/unity/terrain/inputs.html) · [Assets / baking](https://www.sidefx.com/docs/houdini/unity/assets.html)
- [heightfield_project](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_project.html) · [heightfield_slump](https://www.sidefx.com/docs/houdini/nodes/sop/heightfield_slump.html)
- [PDG for Indie Gamedev (free, Unity-based)](https://www.sidefx.com/learn/collections/pdg-for-indie-gamedev/)

**Physics and real projects**
- [NVIDIA PhysX — Geometry / Height Fields](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/Geometry.html)
- [MX Bikes track creation guide (16bpp RAW, 2049², ~0.23 m/sample)](https://docs.piboso.com/wiki/index.php/MXB_Track_Creation_Guide)
- [Lonely Mountains: Downhill — level & game production](https://80.lv/articles/level-game-production-lonely-mountains-downhill) · [Road to the IGF](https://www.gamedeveloper.com/disciplines/road-to-the-igf-megagon-industries-i-lonely-mountains-downhill-i-)
- [Offroad Heat — using Houdini for large terrains (Unity)](https://80.lv/articles/using-houdini-for-large-terrains-006sdf)
- [MX vs ATV Reflex terrain deformation](https://www.vitalmx.com/videos/features/MX-vs-ATV-Reflex-Terrain-Deformation,822/GuyB,64)
- [NEODRIVE devlog — abandoning WheelCollider](https://sevencrane.itch.io/neodrive/devlog/1070030/how-to-exit-collider-hell)
- [Ball collides instead of rolls onto ramp (the double-contact seam)](https://discussions.unity.com/t/ball-collides-instead-of-rolls-onto-ramp-with-mesh-collider-answered/915533)
