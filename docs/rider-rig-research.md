# Rider Rig Research — a Rider that physics can drive

Research for [issue #4](https://github.com/elmofromok/MoMadness/issues/4). Target: Unity
6000.5.9f1, URP, solo developer, commercial release possible.

The question is not "which character model". It is: **what rig can be a Rigidbody
hierarchy that code poses, that the Player can Lean while airborne, and that goes limp on
a Crash.** The model is the cheap part. The joint setup is the expensive part, and the
Bike↔Rider joint is the expensive part of *that*.

Claims are marked **[cited]** with a source, **[inferred]** where it is reasoning rather
than something a source states, or **[unverified]** where it could not be confirmed.

---

## Recommendation

**Mecanim-humanoid mesh + hand-authored `ConfigurableJoint` hierarchy, seeded by Unity 6's
rewritten Ragdoll Wizard. `CharacterJoint` is disqualified — it has no drive of any kind.
`ArticulationBody` is disqualified — it forbids kinematic loops, and hands-on-bars plus
pelvis-to-frame *is* a loop.**

| Decision | Recommendation | Licence | Reversible? |
|---|---|---|---|
| Physics-phase mesh | **Starter Assets `Armature.fbx`** | **Unity Companion License** | Yes |
| Production mesh | Purchased humanoid, retargeted | Asset Store EULA | Yes |
| Joint / physics rig | Wizard Auto-Fill → convert every `CharacterJoint` to `ConfigurableJoint` | your own work | **No — this is the real asset** |
| Driving solver | **PuppetMaster ($90)**, *contingent on one check* — see below | Extension Asset | Mostly |
| Coordinate-space helper | `mstevenson/ConfigurableJointExtensions` | **MIT, © 2012 Michael Stevenson** | n/a |

**Why Starter Assets over Mixamo for the physics phase:** verified Mecanim Humanoid at
file level, free, maintained (v1.1.7, March 2026), licence ships *in the package* as a
text file rather than being inferred from a five-year-stale FAQ, and it does not depend on
a web service that has been intermittently broken since mid-2025. **[cited — see §1, §3]**

**Why the mesh choice is low-stakes:** Unity's humanoid avatar system retargets between
humanoid rigs, so swapping the mesh later does not invalidate the joint work — provided
the replacement is *also* Humanoid. Enforce that when buying. **[inferred]**

### The one check to run before spending $90

**Does PuppetMaster support pinning a puppet to a moving, dynamic vehicle?** That is this
project's load-bearing requirement and it could **not** be verified — `root-motion.com`,
which hosts the documentation, was DNS-unreachable throughout this research.
**[unverified]** Check the RootMotion forum and the `PuppetMaster/_DEMOS` folder before
buying. If the answer is no, the fallback is to write it: the technique is ~200 lines and
is fully documented in §5.

### What needs hand-authoring no matter what is bought

1. **The riding pose** — hands on bars, feet on pegs, knees on the tank. No stock asset has
   motocross poses.
2. **The Bike↔Rider joint** and its break behaviour — the thing that makes a Crash happen.
3. **Lean authority** — mapping Player stick input to a target-pose offset.
4. **Per-bone drive tuning** — stiff torso, soft neck and forearms. One global spring will
   not do. **[inferred, supported by ashleve's per-bone torque profile]**
5. **Feet and hands entirely.** The Wizard gives them nothing (§1).
6. **The whole animation↔ragdoll blend.** Unity ships no such thing (§6).

---

## 1. Unity's own humanoid rigs, and the Ragdoll Wizard

### Nothing Unity ships free contains a configured ragdoll

A firm negative, verified at file level.

**Starter Assets — the pick.** **[cited]**
<https://assetstore.unity.com/packages/essentials/starter-assets-thirdperson-urp-196526>

- Unity Technologies, **free**, v1.1.7 (2026-03-16), supports 6000.0.0f1. A newer listing
  exists as [Starter Assets: Character Controllers | URP](https://assetstore.unity.com/packages/essentials/starter-assets-character-controllers-urp-267961),
  v2.0.2 (2025-01-04), 6000.0.11f1.
- Model: `Assets/StarterAssets/ThirdPersonController/Character/Models/Armature.fbx`.
- **Verified Mecanim Humanoid** — `.meta` has `animationType: 3`, `avatarSetup: 1`, and 53
  human bone mappings including full finger chains. **[cited — file inspection]**
- **Verified no ragdoll**: `PlayerArmature.prefab` has **0 `CharacterJoint`, 0 `Rigidbody`,
  0 colliders**; the string "ragdoll" appears zero times. Movement is
  `CharacterController`-based. **[cited — file inspection]**

**Licence — resolved.** The package ships `Assets/StarterAssets/license.txt` reading
verbatim: *"This package is licensed under the Unity Companion License."* **[cited]** The
UCL grants *"a worldwide, non-exclusive, no-charge, and royalty-free copyright license to
reproduce, prepare derivative works of, publicly display, publicly perform, and distribute
the work"*, conditioned on **Unity Companion Use** — in connection with distributing
applications built under a valid Unity licence. **[cited]**
<https://unity.com/legal/licenses/unity-companion-license>

A shipped Unity game sits squarely inside that. Ship the character inside the game; do not
redistribute the FBX standalone.

**Recorded contradiction:** the two Asset Store listings carry **conflicting licence
metadata** — 196526 says "Non standard EULA", 267961 says "Extension Asset / Standard
Unity Asset Store EULA". No Unity statement reconciles them. **Trust the shipped
`license.txt`.** Commercial release is permitted under either reading. **[cited —
observed contradiction]**

There is **no UPM package** — `com.unity.starter-assets` returns *"Package not found"*.
**[cited — registry probe]** And the name "Nathan" that circulates for this character
appears **nowhere in the package** — folklore. **[cited — absence]**

**The alternatives:**

- **[Robot Kyle | URP](https://assetstore.unity.com/packages/3d/characters/robots/robot-kyle-urp-4696)** —
  Unity Technologies, free, **v5.0 (2026-01-19), supports 6000.3.0f1**. Best-maintained
  free Unity character. Humanoid per listing metadata, not file-verified. Ragdoll
  **[unverified]**. Wrong look for a Rider; fine as a test dummy.
- **Unity-Chan** — publisher is Unity Technologies **Japan**, not Unity Technologies ApS.
  Stale (v1.2.2, June 2020, Unity 2018.4), **not maintained for Unity 6**. Licence is
  Unity-chan License Terms v3.0; commercial use permitted and the advance-approval
  requirement was dropped in June 2024, but obligations remain: display the UCL logo, no
  adult content, and **the character may not be used as AI training data**. **[cited]**
  <https://support.unity.com/hc/en-us/articles/29102655807252-Guidelines-regarding-the-use-of-Unity-chan-in-your-app>
  The v3.0 text could not be quoted verbatim — the site is now a JS SPA. **[unverified]**
- **`Unity-Technologies/Standard-Assets-Characters`** — UCL, one humanoid
  (`DefaultMale.fbx`, `animationType: 3`), but a Unity 2019.3 project with **zero** files
  matching `ragdoll` or `joint`. **[cited]**
- **ml-agents** — **Apache 2.0**, and the only Unity-official ragdoll found anywhere.
  `WalkerRagdoll.prefab` has **15 `ConfigurableJoint`**, 16 Rigidbody, 11 Capsule / 2 Box /
  3 Sphere colliders, and **zero `CharacterJoint`, zero SkinnedMeshRenderer, zero
  Animator**. Primitives, not a skinned humanoid. **Useful as a permissively-licensed
  joint-limit and mass reference.** **[cited]**
- **Animation Rigging** `com.unity.animation.rigging` **1.4.1**, UCL, `package.json`
  declares `"unity": "6000.0"`. Its one sample was **extracted and verified: zero `.fbx`,
  zero skinned meshes, zero Avatars** — twelve scenes of primitives. **[cited]** It is
  kinematic constraint rigging, not a physics solver. Not in this project's
  `Packages/manifest.json`; would need adding. **[cited — the repo]**
- **No "Third Person" Hub template exists** — `com.unity.template.thirdperson` and
  `.third-person` both return not-found. **[cited — registry probe]**

### The Ragdoll Wizard: present, and rewritten in Unity 6

**Confirmed present in 6000.5, no deprecation notice.** **[cited]**
<https://docs.unity3d.com/6000.5/Documentation/Manual/wizard-RagdollWizard.html> — titled
"Create a ragdoll", stamped *"Version: Unity 6.5 (6000.5)"*, built 2026-08-19. Menu path
`GameObject > 3D Object > Ragdoll…`.

**Unity 6 rewrote it.** In 2022.3 it was `class RagdollBuilder : ScriptableWizard`. In
Unity 6 it is an `EditorWindow` with a new **`Animator` field and an "Auto-Fill" button**
**[cited — `Modules/PhysicsEditor/RagdollBuilder.cs`, branch `6000.5` of
`Unity-Technologies/UnityCsReference`]**:

```csharp
internal void AutoAssignTransforms() {
    if (animator == null || animator.avatar == null || !animator.avatar.isHuman) {
        Debug.LogError("Animator or Humanoid Avatar is missing!"); return; }
    ...
    Transform boneTransform = animator.GetBoneTransform(bonePair.Value);
```

**Because Starter Assets is Humanoid, Auto-Fill populates all 13 bone slots in one click.**
That `isHuman` guard is the concrete, mechanical reason to insist on a humanoid rig.

**What it generates** — read from the 6000.5 source, none of it documented:

- **`CharacterJoint` on every bone except the pelvis root**, with
  `joint.enablePreprocessing = false` and the comment *"turn off to handle degenerated
  scenarios, like spawning inside geometry."* **[cited]** (Unity's own stability page gives
  the same advice — see §4.)
- **Colliders:** capsules on hips, knees, arms, elbows; boxes on pelvis and middle spine;
  a sphere on the head with
  `radius = Vector3.Distance(leftArm.position, rightArm.position) / 4`. **[cited]**
- **Feet and hands get nothing.** They are wizard slots but never receive a joint,
  Rigidbody, or collider — they only size the shin/forearm capsule. **[cited]**
  **This matters here:** a Rider whose boots catch the ground and whose hands hold the bars
  needs both authored by hand.
- **Mass:** each bone seeded with a hardcoded density, then globally rescaled **[cited]**:
  ```csharp
  float massScale = totalMass / rootBone.summedMass;
  foreach (BoneInfo bone in bones) bone.anchor.GetComponent<Rigidbody>().mass *= massScale;
  ```
  Seeds: Pelvis 2.5, Hips 1.5, Knee 1.5, Middle Spine 2.5, Arm 1.0, Elbow 1.0, Head 1.0.
  `totalMass` defaults to **20** — set it against the Bike's mass, not in isolation (§4).
- **Default limits** (`minTwist, maxTwist, swing1`; `swing2` is **always 0**): Hips
  (-20, 70, 30); Knee (-80, 0, 0); Middle Spine (-20, 20, 10); Arm (-70, 10, 50); Elbow
  (-90, 0, 0); Head (-40, 25, 25). **[cited]**
- **The `Strength` field is dead code.** The wizard draws it; `strength` is never read
  anywhere in `RagdollBuilder.cs`. Dead in 6000.5 and equally dead in 2022.3. **Don't tune
  it.** **[cited]**

Axis naming, quoted **[cited]**: *"the joint's Twist axis corresponds with the limb's
largest swing axis, the joint's Swing 1 axis corresponds with the limb's smaller swing
axis, and the joint's Swing 2 axis is for twisting the limb. This naming scheme is for
legacy reasons."*

**So the Wizard is a collider-and-limit generator you convert**, not a finished Rider. Run
Auto-Fill, keep the colliders/masses/limits, replace every `CharacterJoint` with a
`ConfigurableJoint` carrying the same limits plus a slerp drive.

---

## 2. Asset Store rigged riders

**Every motocross-specific rider listing is stale, and none declares Unity 6 support.**
That is the headline. **[cited — all five listings below]**

| Asset | Publisher | Price | Published / updated | Unity declared | Verdict |
|---|---|---|---|---|---|
| [Sport Rider Fantasy Stylized Humanoid Rigged](https://assetstore.unity.com/packages/3d/characters/sport-rider-fantasy-stylized-humanoid-rigged-298855) | Pixora | $9.99 | 2024-10-24 | 2022.3.47 | Humanoid ✅, but 1 animation, 0 ratings |
| [Biker Character Pro](https://assetstore.unity.com/packages/3d/characters/biker-character-pro-179179) | TGameAssets | $19.99 | 2020-09-22 | 2019.1.8 | ⚠️ **NOT RIGGED** |
| [Dirt Bike Motocross](https://assetstore.unity.com/packages/3d/vehicles/land/dirt-bike-motocross-174150) | TGameAssets | — | 2020-07-07 | 2019.1.8 | Bike only, no rider |
| [Simple Motocross Physics](https://assetstore.unity.com/packages/tools/physics/simple-motocross-physics-221408) | AiKodex | **$50** | v1.3, **2024-07-24** | 2020.3.14f1 | ⭐ **the interesting one** |
| [Rider zombie animated character](https://assetstore.unity.com/packages/3d/characters/rider-zombie-animated-character-170752) | myGameAssets | — | 2020-06-17 | 2019.2.18 | Irrelevant — it's a zombie |

**Sport Rider Fantasy Stylized** — listing states *"a **humanoid rig with an idle
animation**… perfect for a variety of roles, **from bike riders to adventurers**"*, *"a
total of **12.3k triangles and 6.3k vertices**"*. **[cited]** No bike, no ragdoll, one idle
animation, no riding poses. The listing also **contradicts itself** on pipeline — the
feature block claims URP while the body text twice says "built-in render pipeline".
**[cited]** At $9.99 it is a low-risk experiment, not a foundation.

**Biker Character Pro fails the primary criterion.** The listing says *"**Demo Scene
Included with Character in T-Pose**"* and, decisively, *"**✔️ Character can be Rigged &
Animated using Mixamo.**"* **[cited]** It ships as an unrigged T-pose mesh (~20.7k tris,
modular jacket/cap/mask/glasses). You would rig it through Mixamo yourself.

**⭐ Simple Motocross Physics is the highest-signal find.** $50, **43 reviews** — a real
track record, unlike every character listing above — updated 2024-07-24, **Extension
Asset**. Quoted from the listing **[cited]**:

> *"Pipelines Supported : Standard, HDRP, **URP** and SRP."*
> *"Animation Rigging v1.0.0 is used for customizable IK to fit any style of riding."*
> *"Customize IK Style: **Active Ragdoll like Impact Motion**, Customizable IK Targets,
> Body Damping on slopes, Natural Balancing Motions."*
> *"**Only 3 Humanoid Animations** have been used in the asset, the rest of the motion is
> controlled via IK."*
> *"**Supports any Custom Character: … Works best with Mixamo Rigs. Built on a Mixamo
> Styled Rig**, the asset can easily replace the character with a mixamo rig in a matter of
> minutes using **Editor Scripts**."*

A verified user review (2022-08-11) reads: *"Looked something right out of MX vs ATV. The
cherry on top is the quality of the physical animations of the **ragdoll**, feels quite a
bit like the euphoria mods."* **[cited]**

**How to treat it:** it is a complete motocross physics system, and this project's whole
point is authoring its own Bike/Rider physics — so it is **not** a foundation. But it is
the closest thing to a working reference implementation of exactly this problem, at $50,
with URP support and IK riding poses. **Worth buying as a reference even if none of its
code ships.** **[inferred]** Unity 6 compatibility is **unverified** (declares 2020.3).

**Note the ecosystem convergence:** the strongest motocross asset on the store is *built on
a Mixamo-styled rig and ships editor scripts to swap Mixamo characters in*, and the main
biker character asset tells you to rig it *with Mixamo*. **[cited]** Whatever the licence
anxieties, the store has already standardised on Mixamo-shaped humanoids.

**Synty POLYGON — the humanoid question, answered honestly.** Synty's store text says
**"Characters set up with Mecanim"** **[cited]**
<https://syntystore.com/products/polygon-modular-fantasy-hero-characters> — which is *not*
the same statement as "configured as a Humanoid avatar". Their animation packs are
marketed as working with *"any characters that work with the Mecanim humanoid avatar"*,
which implies their characters qualify. **Assessment: humanoid-mappable, but the store text
stops short of guaranteeing it, and it has varied by pack.** Synty's proportions — large
heads, stubby limbs — are exactly where Unity's auto-mapping needs manual bone assignment.
POLYGON Kids and POLYGON Big Rig use *different* skeletons. Verify the specific pack.
**[cited for the quotes, inferred for the assessment]**

**Not verified:** Kevin Iglesias, MalberS, Infinity PBR, Kubold, "Character Pack: Free
Sample". **[unverified]**

### The tooling assets

| Asset | Publisher | Price | Version / date | Unity declared | Notes |
|---|---|---|---|---|---|
| [**PuppetMaster**](https://assetstore.unity.com/packages/tools/physics/puppetmaster-48977) | RootMotion | **$90** | 1.5, **2026-04-11** | **6000.4.2f1**, 6000.0.72f1, 2022.3, 2021.3; Built-in/URP/HDRP | 5★, 220 reviews. The canonical answer. |
| [**Final IK**](https://assetstore.unity.com/packages/tools/animation/final-ik-14290) | RootMotion | **$90** | 2.5, **2026-04-10** | same four | 876 reviews |
| [**Ragdoll Animator 2**](https://assetstore.unity.com/packages/tools/physics/ragdoll-animator-2-285638) | FImpossible Creations | **$36.99** | 1.0.4.2, **2026-07-13** | orig. 2021.3.34 | Cheapest maintained option |
| [Self-Balancing Active Ragdoll](https://assetstore.unity.com/packages/tools/animation/self-balancing-active-ragdoll-370608) | Frost Punch Games | $19.99 | 1.1.4, 2026-05-08 | 2022.3.62f3+ | ⚠️ listing flags **"Created with AI"** |

**[cited — respective listings]**

Both RootMotion tools were updated within four months of this research, which settles the
recurring worry that PuppetMaster is abandonware. It is not. **[cited]**

From the [RootMotion FAQ](https://rootmotion.freshdesk.com/support/solutions/articles/77000057786-faq)
**[cited]**: PuppetMaster handles *"advanced character physics, blending between animation
and physics, partial ragdolls and physical player-character interactions"*; it **requires
PhysX** (not 2D, not Havok/DOTS); and **Final IK is not required** — they *"were designed
to complement each other."*

**Ragdoll Animator 2** exposes a **`RagdollBlend`** scalar plus discrete states via
`Settings.AnimatingMode` / `User_SwitchFallState()`, and a **"Rotate To Pose Force"**
described by the developer as *"the overall muscle force put into limbs… the main
multiplier for the Configurable Spring."* **[cited — the developer's own forum thread]**
<https://discussions.unity.com/t/released-ragdoll-animator-2-easy-body-physics/949880>

**Version gap:** listings top out at **6000.4.2f1**; this project is **6000.5.9f1**.
Untested. **[open]** Mitigation: Unity's refund policy lists as valid grounds *"The asset is
not compatible with the most recent LTS Unity version, and no information was provided in
the asset description to indicate this."* **[cited]**

### ✅ A Unity 6 joint bug that does *not* affect this project

`ConfigurableJoint` limits were bugged in **6000.0 through 6000.0.4** — Unity issue *"Jerky
initialization of Joints occurs when Configurable Joint Limits are used"*, reproducible in
6000.0.4f1 but not in 2021.3/2022.3. On 2024-07-10 the Ragdoll Animator developer wrote
*"yes I saw it is solved"* and gated their warning behind `#if UNITY_6000_0_9_OR_NEWER`.
**[cited]** → **Fixed by 6000.0.9; 6000.5.9f1 is unaffected.** **[inferred from the version
gate]** Worth knowing because stale forum posts still warn about it.

---

## 3. Licensing

### Unity Asset Store EULA — the actual text

**Source:** <https://unity.com/legal/as-terms>, stating **"Last updated: December 4, 2024"**
(the change being AI/ML training prohibitions plus a jurisdiction move to California).
**[cited]**

**The grant, §2.2.1** **[cited verbatim]**:

> "Licensor hereby grants to the END-USER a non-exclusive, non-transferable, worldwide, and
> perpetual license to the Asset solely: (a) to incorporate the Asset, **together with
> substantial, original content not obtained through the Unity Asset Store**, into an
> electronic application … ("**Licensed Product**") **as an embedded component of that
> Licensed Product, such that the Asset does not comprise a substantial portion of the
> Licensed Product**; (b) to reproduce, publicly display, publicly perform, transmit, and
> distribute the Asset as incorporated and embedded in that Licensed Product; … (d)
> **monetize the Asset within and for use within a Licensed Product**…"

**That is the commercial-release permission.** A shipped motocross game is a Licensed
Product. Two subtle conditions in (a): the Asset must be combined with **substantial
original content of your own**, and must **not comprise a substantial portion** of the
product.

**Restrictions, §2.2.1.1** **[cited verbatim, abridged]**:

> "END-USER may not … (b) **enable a customer or user of a Licensed Product to sell,
> transfer, distribute, lease, or lend the Assets for commercial gain or commercialize
> Assets within a Licensed Product**, (c) without express authorization, **monetize an
> Asset in a Licensed Product where the Licensed Product's primary purpose is to create
> user-generated content**, … (g) use the Unity Asset Store or Assets for purposes such as
> **training an artificial intelligence or machine learning model** without … express
> consent."

**Precision correction to common folklore:** the EULA contains **no clause literally saying
"you may not redistribute the asset in a form that lets others extract it."** The actual
mechanism is §2.2.1(a)/(b) — distribution is licensed *only* "as incorporated and embedded"
— combined with §2.2.1.1(b). The practical effect matches the folklore: a shipped game
build is fine; a Unity template, a mod SDK exposing source FBX, or an asset dump is not.
**[cited — including the verified absence]**

**§2.2.1.1(c) is worth flagging for this project specifically:** if a track editor or
livery creator is ever added *as a primary purpose*, monetising bought assets inside it
needs express authorisation. **[cited]**

**Licence tiers.** §2.3.1: assets install on "an unlimited number of computers"; a
"multi-entity" tier extends to Affiliates and Contractors. §2.3.2: **Extension Assets**
(Editor Extension / Scripting / Services categories) are **single-seat, max 2 computers**,
though "build farm servers and virtual machine instances … do not require separate seat
licenses". §2.4: a Contractor "must have license(s) of its own". **[cited verbatim]**

**For a solo developer, Single Entity is correct and sufficient.** The distinction only
bites on incorporating or hiring. Note **PuppetMaster, Final IK, and Simple Motocross
Physics are all Extension Assets** — seat-limited.

### Mixamo — permissive on paper, unreliable in practice

Adobe's official FAQ **[cited verbatim]**
(<https://helpx.adobe.com/creative-cloud/faq/mixamo-faq.html>; also retrieved via the
[Wayback snapshot of 2026-07-24](http://web.archive.org/web/20260724091310/https://helpx.adobe.com/creative-cloud/faq/mixamo-faq.html)
because Adobe blocks automated fetching):

> "You can use both characters and animations royalty free for personal, commercial, and
> non-profit projects including: … **Create video games**."

> "Mixamo is available free for anyone with an Adobe ID and does not require a subscription
> to Creative Cloud."

Stated restrictions: *"Mixamo is not available for Enterprise and Federated IDs"* and *"not
available for users who have a country code from China."* The auto-rigger is *"for bipedal
humanoids only"*. **[cited]**

**"Create video games" is explicitly enumerated as permitted royalty-free commercial use,
on Adobe's own domain.** That is the clearest sentence available.

**⚠️ An ambiguity that must be flagged.** The widely-quoted redistribution prohibition —
roughly *"you may not create blueprints, templates, or asset packages for video game
engines which redistribute character or animation raw files as the product"* — **is not on
the current official FAQ.** The retrieved HTML was searched for `redistribut`, `blueprint`,
`asset package`, `sell`, and `distribute`: **no matches**. **[cited — verified absence]**
It has either been removed in a past edit or lives on a page that could not be reached.

Two further caveats on that evidence: the FAQ is an AEM accordion whose **question headings
are JavaScript-rendered and absent from static HTML**, so an answer block could have failed
to render; and the page's own header reads **"Last updated on Sep 14, 2021"** — Adobe has
not touched this licensing text in nearly five years. **[cited]**

**Practical position:** the permissive grant is solidly verified. The redistribution
prohibition is unverified-but-almost-certainly-real, consistent with every comparable stock
licence, and **irrelevant here** — this project ships a compiled game, not FBX files. Do
not ship Mixamo raw files as a downloadable pack, mod kit, or Unity template.

**Service status.** No official shutdown announcement exists. **[cited — absence]**
Positive evidence it is alive: the Wayback Machine holds a snapshot of `mixamo.com` dated
**2026-08-14**, and the live site served its app shell during this research. **[cited]**
Negative evidence: community reports put a breakage around 2025-06-16 with prolonged
outages — one Adobe community thread is titled *"Mixamo Is Not 'End of Life' — It's Broken,
and Fixable"* **[cited, community-sourced]**; `mixamo.com/faq` 404s; and the FAQ has been
untouched since 2021. The "maintenance mode" characterisation is **[inferred]**, not
reported.

**Mitigation, if Mixamo is used at all: download everything now and archive it locally with
a dated screenshot of the FAQ.** The FAQ itself advises *"It is recommended that you save
your rigged characters locally"* **[cited]**. The grant is perpetual in character; the
*service* is the risk. Whether the stock characters (Y Bot, Vanguard, Remy…) are still
available could not be verified — the catalogue is behind an authenticated API.
**[unverified]**

Adobe **Fuse**, the companion character *creator*, is discontinued. **[cited, secondary]**
<https://www.neowin.net/news/adobe-revamps-mixamo-and-will-soon-be-discontinuing-support-for-fuse-cc/>

### Other free sources

**✅ Quaternius — CC0, the zero-risk fallback.** **[cited verbatim]**
<https://quaternius.com/faq.html>

> "Yes, these assets can be used **for free** without the need for attribution in
> **commercial, educational, and personal projects**."
> "No, attribution is not necessary. However, credit is always appreciated."

Relevant packs include **Universal Base Characters (Rigged, Retargetable)** and the
Ultimate Modular Men/Women packs. "Rigged, Retargetable" strongly suggests
Humanoid-mappable, but the site never says "Unity Humanoid" — **[inferred]**. CC0 means
zero legal risk, and the stylised low-poly look suits an arcade motocross game.

**⚠️ Sketchfab — two traps.** **[cited]** <https://sketchfab.com/licenses> The **Standard
License** permits commercial use but forbids making the material available *"as a
stand-alone file"* and requires credit *"in equal size and comparable placement"*. The
**Editorial License** *"cannot be used for any commercial or promotional use"* — **do not
use Editorial assets in the game.** Community uploads carry CC0 / CC-BY / CC-BY-NC
individually; **CC-BY-NC forbids commercial use entirely**, which is the most common
Sketchfab mistake.

**🚨 SMPL / Meshcapade — confirmed trap.** **[cited verbatim]**
<https://smpl.is.tue.mpg.de/modellicense.html>

> licensed "for the sole purpose of performing **non-commercial** scientific research,
> **non-commercial** education, or **non-commercial** artistic projects."
> **"Any other use, in particular any use for commercial purposes, is prohibited."**

Commercial use requires a paid Meshcapade licence. Anything derived from SMPL inherits
this. **Avoid entirely.**

**⚠️ VRoid Studio** — commercial use in games is permitted, but you **cannot build an
application that itself generates or outputs 3D models** from VRoid meshes. Irrelevant for
a fixed-roster game; fatal if avatar customisation is ever added. Per-item licences can
override. Sourced from `vroid.com/en/studio/guidelines`, which **403'd to automated
fetch** — treat the wording as paraphrase. **[unverified wording, cited source]**

**⚠️ MakeHuman** — `makehumancommunity.org` refused connection during this research.
**[unverified]** The generally-understood position is that the program is AGPL while
**output meshes are CC0** — confirm on the official licence page before shipping.

**⚠️ Blender Studio rigs (Rain, Snow)** — licence unverified (403). The bigger issue is
practical: these are *animator* rigs, heavy Rigify-derived control rigs with drivers and
constraints that do not export cleanly to a game engine. Not worth it. **[inferred]**

**❌ Poly Haven** — CC0, but HDRIs/textures/props only. **No rigged humanoids.** Excellent
for terrain textures and skies; irrelevant to this ticket.

---

## 4. What a ragdoll-capable rig actually requires

### Unity's own stability page

All **[cited]** from
<https://docs.unity3d.com/6000.5/Documentation/Manual/RagdollStability.html>:

> "Avoid large differences in the masses between Rigidbody components connected by Joints.
> It's okay to have one Rigidbody with twice as much mass as another, but when one mass is
> ten times larger than the other, the simulation can become jittery."

> "Avoid small Joint angles of Angular Y Limit and Angular Z Limit. Depending on your
> setup, the minimum angles should be around 5 to 15 degrees in order to be stable."

> "Uncheck the Joint's Enable Preprocessing property. Disabling preprocessing can help
> prevent Joints from separating or moving erratically."

> "[For jittering] try increasing the Default Solver Iterations value to between 10 and 20."

> "[For inaccurate bounce] try increasing the Default Solver Velocity Iterations value to
> between 10 and 20."

> "Try to avoid scaling different from 1 in the Transform containing Rigidbody or the Joint."

> "If Rigidbody components are overlapping when inserted into the world, and you cannot
> avoid the overlap, try lowering the Rigidbody.maxDepenetrationVelocity."

The velocity-iterations setting names the case directly **[cited]**: *"If you experience
problems with jointed Rigidbody components or Ragdolls moving too much after collisions,
try increasing this value."*

### ⭐ Mass ratio — the Bike↔Rider joint is exactly this problem

The strongest citation is NVIDIA's, not Unity's. [PhysX 5 Joints
documentation](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/Joints.html)
**[cited verbatim]**:

> "the solver can have difficulty converging well when a light object is constrained
> between two heavy objects. **Mass ratios of higher than 10 are best avoided in such
> scenarios.**"

and, directly actionable:

> "when one body is significantly heavier than the other, **make the lighter body the
> second actor in the joint**."

**Mapped to Unity [inferred]:** Unity's `Joint` sits on actor0 and `connectedBody` is
actor1. So the **lighter body should be the `connectedBody`** — meaning the
`ConfigurableJoint` goes **on the Bike** with `connectedBody = riderPelvis`, which is the
opposite of what most people do. `Joint.swapBodies` flips this without re-authoring.

**Unity's own mitigation**, [`Joint.massScale`](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Joint-massScale.html)
**[cited]**: a "scale to apply to the inverse mass and inertia tensor of the body prior to
solving the constraints", which "make[s] the joints solver converge faster, thus resulting
in **less stretch of the limbs of a typical ragdoll**." Caveat, verbatim: *"scaling mass
and inertia is fundamentally nonphysical and momentum won't be conserved."* Unity's own
sample from that page **[cited verbatim]**:

```csharp
// Make sure that both of the connected bodies will be moved by the solver with equal speed
j.massScale = j.connectedBody.mass / root.GetComponent<Rigidbody>().mass;
j.connectedMassScale = 1f;
```

The ConfigurableJoint manual says the same in prose **[cited]**: *"When your connected
Rigidbodies vary in mass, use this property with the Connected Mass Scale property to apply
fake masses to make them roughly equal to each other."*

**⭐ And try the Temporal Gauss-Seidel solver.** Unity 6's Physics settings expose **Solver
Type: Projected Gauss Seidel (default) / Temporal Gauss Seidel**, the latter described as
*"better convergence and a better handling of high-mass ratios … improves the resistance of
joints to overstretch."* **[cited]**
<https://docs.unity3d.com/6000.5/Documentation/Manual/class-PhysicsManager.html>
PhysX adds that *"Since PhysX 5.1 the Temporal Gauss-Seidel solver has become the default.
We recommend using one velocity iteration by default."* **[cited]** — so **Unity ships the
non-default-for-PhysX solver.** Flipping to TGS is a one-click experiment aimed straight at
the Bike/Rider mass disparity, and should be tried *before* hand-scaling masses.
**[inferred]** Caveat: it is **not new in Unity 6** (present since 2019.3), and a Unity
Discussions thread is titled *"PhysX upgrade: Temporal Gauss-Seidel solver much less stable
than PGS"* **[cited]** — so A/B test it, don't adopt on faith.

### This project's current physics settings

Read directly from `ProjectSettings/DynamicsManager.asset` and `TimeManager.asset`
**[cited — the repo]**:

| Setting | Current | Guidance | Action |
|---|---|---|---|
| `m_DefaultSolverIterations` | **6** | 10–20 (Unity); ~30 in Unity's own sample | raise **per-body**, not globally |
| `m_DefaultSolverVelocityIterations` | **1** | 10–20 (Unity); 1 (PhysX, with TGS) | raise per-body; test |
| `m_SolverType` | **0** (PGS) | TGS handles high mass ratios | A/B test TGS |
| `m_DefaultMaxAngularSpeed` | **50** | — | fine (see below) |
| `m_AutoSyncTransforms` | 0 | correct | leave |
| Fixed Timestep | **0.02** | — | may need 0.0167 or 0.01 |

**⚠️ Gotcha: `Physics.defaultSolverIterations` only affects *newly created* Rigidbodies.**
**[cited]** To change an existing ragdoll you must set
[`Rigidbody.solverIterations`](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-solverIterations.html)
per body — do it in `Awake()` on every Rider bone. This also keeps the terrain and Bike
from paying for the Rider's joint chain.

PhysX's ceiling **[cited]**: *"If you find a need to use a setting higher than 30, you may
wish to reconsider the configuration of your simulation."*

### ✅ maxAngularVelocity — a docs contradiction, settled for this project

Unity 6 documents two different defaults: `Rigidbody.maxAngularVelocity` says **7 rad/s**,
while `Physics.defaultMaxAngularSpeed` and the Physics settings page both say **50**.
**[cited — all three]** The widely-repeated "7 rad/s clamps your limbs" folklore appears to
be **stale documentation** carried forward from the pre-`defaultMaxAngularSpeed` era.
**[inferred]**

**This project's `DynamicsManager.asset` reads `m_DefaultMaxAngularSpeed: 50`.** **[cited —
the repo]** That settles it here: 7 rad/s ≈ 1.11 rev/s would visibly clamp a backflipping
Bike and a whipping limb; 50 rad/s ≈ 7.96 rev/s will not. No action needed. (A one-line
runtime `Debug.Log` would confirm empirically if it ever looks wrong.)

### ⭐ Fixed timestep, and a cheaper alternative

[PhysX Best Practices](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/BestPractices.html)
on jointed-object stability **[cited verbatim]**:

> "Use smaller time steps. This can be an effective way to improve joints' behavior,
> although it can be an expensive solution."

> "**Consider creating the same constraints multiple times.** This is similar to increasing
> the number of solver iterations, but the performance impact is **localized to the jointed
> object** rather than the simulation island it is a part of."

**That second quote is the underused trick [inferred]:** attach **two identical
`ConfigurableJoint` components** between the Rider's pelvis and the Bike. You get the
stiffness of doubled solver iterations without paying for it scene-wide — ideal for the one
joint in this game that carries the most load.

**[inferred]** For an active ragdoll bolted to a fast vehicle, 0.02 (50 Hz) is likely too
coarse; 0.01 (100 Hz) is the common landing spot at 2× cost. With V-Sync at 60 Hz, `0.01667`
aligns physics to display and avoids interpolation beat frequencies — often a cheaper fix
for *perceived* jitter than raising the rate.

### Colliders and self-collision

[`Joint.enableCollision`](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Joint-enableCollision.html)
"Enable collision between bodies connected with the joint" — **the docs do not state its
default**; widely held to be `false`. **[unverified]** It governs only *directly jointed*
pairs. Non-adjacent pairs (left hand vs right thigh) collide by default, which is what a
ragdoll wants; adjacent-but-not-jointed pairs interpenetrate at spawn. Tools:
`Physics.IgnoreCollision`, `Rigidbody.excludeLayers`/`includeLayers`, and the layer matrix.
The Rider must also not collide with the Bike it is joined to. **[inferred]**

### Interpolation, and a respawn bug waiting to happen

[`Rigidbody.interpolation`](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-interpolation.html)
**[cited verbatim]**:

> "By default, interpolation is disabled."
> "It is recommended to turn on interpolation for the main character but **disable it for
> everything else**."
> ⚠️ "When interpolation or extrapolation is enabled, **the physics system takes control of
> the Rigidbody's transform**. For this reason, you should follow any direct (non-physics)
> change to the transform with a `Physics.SyncTransforms` call."

**[inferred] consequences:** interpolating *every* ragdoll bone is the naive move and can
*introduce* skinning artifacts — each bone interpolates independently, so mid-step the
chain is not a rigid-consistent pose and limbs appear to stretch at the joints. Interpolate
the root the camera follows (the Bike) and test the Rider's bones both ways.

**The `SyncTransforms` warning is not academic here:** respawning teleports Bike and Rider,
which is a direct non-physics transform change to interpolated bodies. Omit
`Physics.SyncTransforms()` and the first frame after respawn reports stale collider
positions. `m_AutoSyncTransforms` is `0` in this project, which makes this mandatory.
**[cited — the repo + the docs]**

---

## 5. Active ragdoll — the core technique

### The choice is forced: `ConfigurableJoint`

`CharacterJoint`'s **complete** member list is `enableProjection`, `projectionDistance`,
`projectionAngle`, `lowTwistLimit`, `highTwistLimit`, `swing1Limit`, `swing2Limit`,
`swingAxis`, `twistLimitSpring`, `swingLimitSpring`, plus inherited Joint members. **There
is no drive, no motor, no `targetRotation`. Zero.** **[cited]**
<https://docs.unity3d.com/6000.5/Documentation/ScriptReference/CharacterJoint.html>

**A `CharacterJoint` ragdoll can never be driven toward a pose.** Since the Rider must hold
a riding pose and respond to Lean, this is not a preference.

Unity's manual endorses the alternative for exactly this **[cited]**: Configurable Joints
are *"particularly useful when you want to customize the movement of a ragdoll and enforce
certain poses on your characters."*

Also note CharacterJoint's warning about its own projection **[cited]**: *"Projection is
not a physical process and does not preserve momentum or respect collision geometry. It is
best avoided if practical."*

### ⭐ `ArticulationBody` is disqualified — and the reason is specific

Unity's [articulations manual](https://docs.unity3d.com/6000.5/Documentation/Manual/physics-articulations.html)
makes a *surprisingly strong* case first **[cited]**: regular joints may leave "certain
constraints unsatisfied" resulting in "stuttery and unrealistic motion", and articulations
prevent "unsolvable scenarios" that occur "in kinematic chains like **ragdolls** or robotic
arms". Spherical joints are "best suited for simulating humanoid limbs". The Featherstone
reduced-coordinate solver ensures "no unwanted stretch occurs" **[cited]** — precisely the
failure mode ConfigurableJoint chains suffer.

**But the stated limitations kill it here** **[cited]**:

> "An articulation can only have one root"
> "Not allowed to have kinematic loops. **If you need kinematic loops, use regular joints**"
> "The physics engine uses the Unity transform hierarchy to express the parent-child
> relationship"

**A Rider whose pelvis is joined to the Bike *and* whose hands are on the handlebars forms
a kinematic loop** — pelvis → spine → arm → bar → frame → pelvis. That is exactly the case
Unity's own docs tell you to use regular joints for. Worse: Bike and Rider would have to
live in one articulation with one root and a fixed transform hierarchy, so **the Rider
could not cleanly detach on a Crash** — which is the core requirement. And zeroing drive
stiffness does not release reduced-coordinate joints the way disabling a
`ConfigurableJoint` does. **[cited limitations, inferred conclusion]**

Unity's own framing agrees: the articulations page steers the feature at *"commercial and
industrial non-gaming applications that involve joints"*. **[cited]**

**Also flagged:** the manual and the scripting API give **contradictory signs** for the
drive formula — the manual says `stiffness * (target - drivePosition) + damping * (...)`,
the scripting page says `stiffness * (currentPosition - target) - damping * (...)`.
**[cited — both]** The manual's is the standard PD law; the scripting page's would be
positive feedback. **[inferred: the scripting page is wrong.]** Noted so nobody tunes
against a phantom.

### The drive setup

- `rotationDriveMode = RotationDriveMode.Slerp`. Exactly two enum values, `XYAndZ` and
  `Slerp`. **[cited]**
- `slerpDrive` then applies: *"how the joint's rotation will behave around all local axes.
  **Only used if Rotation Drive Mode is Slerp Only**."* **[cited]**
- `JointDrive`: `positionSpring` (*"Strength of a rubber-band pull toward the defined
  direction"*), `positionDamper` (*"Resistance strength against the Position Spring"*),
  `maximumForce` (a clamp), `useAcceleration`. **[cited]**

**⭐ Set `useAcceleration = true`.** PhysX documents the underlying flag as *"If this flag
is set the acceleration (rather than the force) applied by the drive is proportional to the
angle from the target"* **[cited — PhysX 5 Joints]**. An acceleration drive is
mass-independent, so one spring value produces the same *angular response* on a 12 kg torso
and a 0.6 kg forearm. Without it you must hand-tune a per-bone torque profile — which is
exactly what ashleve's `maxJointTorqueProfile[]` exists to work around. **[inferred,
supported]**

**Slerp over XYAndZ [inferred]:** Slerp gives one `JointDrive` per joint — one number to
lerp to zero when going limp, which makes the Crash blend trivial. XYAndZ gives independent
twist vs swing stiffness (useful for a neck or wrist) at the cost of two drives to blend,
and it fights itself on ball joints like shoulders and hips. **[cited docs inconsistency:**
`angularXDrive` is documented as *"Only used if Rotation Drive Mode is Swing & Twist"* while
the enum is `XYAndZ` and the Inspector says "X and YZ" — three names, one mode.**]**

### ⭐ The `targetRotation` coordinate-space trap, fully traced

Unity's scripting docs say, in full: *"This is a Quaternion. It defines the desired rotation
that the joint should rotate into."* **No mention of space at all.** **[cited]** That vacuum
is why a 2008 forum thread became the de-facto standard.

The manual is better **[cited]**: *"The target rotation is relative to the body that the
Joint is attached to, unless the Swap Bodies parameter is set…"* — but still not enough.

**Provenance, fully established:**

- **Origin thread**, started by MatthewW **2008-02-14**:
  <https://discussions.unity.com/threads/quaternion-wizardry-for-configurablejoint.8919/>
  The load-bearing explanation **[cited verbatim]**:
  > "The joint coordinate space is unusual: its axes are inverted, and it always begins with
  > a rotation of Quaternion.identity (no rotation at all) regardless of its transform's
  > local or world rotation."

  Values must be "converted to local or world space before transformations are applied,"
  then "converted back to joint space before being assigned to targetRotation."
- **mstevenson posted the derivation 2013-02-15**, same day as
  [gist 4958837](https://gist.github.com/mstevenson/4958837). The gist carries no licence
  header.
- **The licensed home of identical code:**
  [`mstevenson/UnityExtensionMethods/ConfigurableJointExtensions.cs`](https://github.com/mstevenson/UnityExtensionMethods/blob/master/ConfigurableJointExtensions.cs)
  — **MIT, `LICENSE` reads "Copyright (c) 2012 Michael Stevenson"**. **[cited — licence file
  read directly]** → **Safe to ship under MIT with attribution retained.**

The three things that make it counterintuitive: it is relative to the joint's **initial**
rotation at creation; it is expressed in a **joint space** defined by `joint.axis` ×
`joint.secondaryAxis`, not the transform's local space; and that joint space is the
**inverse** of world space. Hence the sandwich:

```csharp
static void SetTargetRotationInternal (ConfigurableJoint joint,
    Quaternion targetRotation, Quaternion startRotation, Space space)
{
    var right   = joint.axis;
    var forward = Vector3.Cross(joint.axis, joint.secondaryAxis).normalized;
    var up      = Vector3.Cross(forward, right).normalized;
    Quaternion worldToJointSpace = Quaternion.LookRotation(forward, up);

    Quaternion resultRotation = Quaternion.Inverse(worldToJointSpace);

    // Joint space is the inverse of world space, so we need to invert our value
    if (space == Space.World)
        resultRotation *= startRotation * Quaternion.Inverse(targetRotation);
    else
        resultRotation *= Quaternion.Inverse(targetRotation) * startRotation;

    resultRotation *= worldToJointSpace;
    joint.targetRotation = resultRotation;
}
```

**API contract: cache each joint transform's `localRotation` in `Start()`** — before physics
touches it — and pass it as `startLocalRotation` forever after. Once the ragdoll moves the
rest pose is gone. **[cited — the file's own summary]** **Optimisation [inferred]:**
`worldToJointSpace` is constant per joint; precompute it once at `Start()` rather than doing
two cross products and a `LookRotation` per bone per FixedUpdate — ashleve does exactly this
with a `localToJointSpace[]` array.

**⚠️ One more trap [cited]:** per a 2014 forum answer, `targetRotation` is *"relative to the
initial rotation of the Rigidbody"* and **changing the connected rigidbody causes the value
to recalculate**, which can trigger unwanted circular motion. Directly relevant if the
Rider's joint is ever re-parented to the Bike at runtime.
<https://discussions.unity.com/t/configurable-joint-how-does-targetrotation-work/105248>

The per-FixedUpdate loop every reference implementation uses **[inferred from the pattern]**:

```csharp
void FixedUpdate() {
    for (int i = 0; i < joints.Length; i++)
        joints[i].SetTargetRotationLocal(
            animatedBones[i].localRotation,   // from the hidden Animator skeleton
            startLocalRotations[i]);          // cached in Start()
}
```

### Open-source reference implementations

From the GitHub API, live on 2026-08-20 **[cited]**:

| Repo | Stars | Licence | Last push | Technique |
|---|---|---|---|---|
| [sergioabreu-g/active-ragdolls](https://github.com/sergioabreu-g/active-ragdolls) | **361** | **Apache-2.0** | 2021-02-02 | Dual-body; ConfigurableJoint drives + `targetRotation` |
| [hairibar/Hairibar.Ragdoll](https://github.com/hairibar/Hairibar.Ragdoll) | **266** | **MIT** | 2021-04-21 | UPM package; **alpha blending, partial ragdolls** |
| [ashleve/ActiveRagdoll](https://github.com/ashleve/ActiveRagdoll) | **261** | **MIT** | 2023-03-05 | Master/slave **PD controller** |
| [kressdev/RagdollTrainer](https://github.com/kressdev/RagdollTrainer) | 79 | MIT | 2021-12-14 | ML-Agents RL |
| [TildeAsterisk/Physicanim](https://github.com/TildeAsterisk/Physicanim) | 35 | MIT | 2022-11-01 | Toolkit, "Works with Mixamo" |
| [EggyStudio/Unity.Humanoid.ActiveRagdoll](https://github.com/EggyStudio/Unity.Humanoid.ActiveRagdoll) | 0 | MPL-2.0 | **2026-03-25** | Most recently maintained |
| [Jesse-ww/UnityActiveRagdolls](https://github.com/Jesse-ww/UnityActiveRagdolls) | 19 | **GPL-3.0** | 2018-03-06 | ⚠️ **viral — avoid** |
| [edualvarado/unity-antagonistic-controller](https://github.com/edualvarado/unity-antagonistic-controller) | 125 | **none** | 2022-12-16 | SCA 2022 paper |

**Licence warnings [cited licences, inferred consequence]:** "no licence" means
all-rights-reserved — `edualvarado`, `dci05049`, `Oladipupo`, `matieme` may be **read for
technique but not copied**. `Jesse-ww` is **GPL-3.0**: viral, avoid entirely for a
commercial game. The clean pickings are **MIT** (mstevenson, Hairibar, ashleve) and
**Apache-2.0** (Sergio).

**Staleness [cited push dates]:** the three highest-star repos were last touched 2021–2023;
none targets Unity 6. Sergio's README states Unity 2020.1.9f1. Expect API drift — notably
`Rigidbody.velocity` → `linearVelocity`, renamed in Unity 6. **These are technique
references, not dependencies.** That is the strongest argument for PuppetMaster, updated
April 2026. **[inferred]**

**sergioabreu-g/active-ragdolls (361★, Apache-2.0)** — the closest architectural match.
README **[cited]** describes an **animated body** (Animator set to "Always Animate",
visually hidden) and a **physical body** (per-bone Rigidbodies joined by ConfigurableJoints
with angular drives targeting the animated body's rotations). Modules: `AnimationModule`,
`PhysicsModule`, **`GripModule`**, `InputModule`, `CameraModule`. **The `GripModule` /
`Gripper` / `Grippable` pair creates joints at runtime when a hand grabs something — that is
structurally the same problem as a Rider's hands staying on handlebars.** **[cited for the
module, inferred for the mapping]** README credits "Michael Stevenson's Configurable Joint
Extensions". **[cited]**

**ashleve/ActiveRagdoll (261★, MIT)** — the PD reference. From
`AnimationFollowing.cs` **[cited]**: linear motion via
`rb.AddForce(forceSignal, ForceMode.VelocityChange)`, rotation via `slerpDrive` — **not**
`AddTorque`. PD law `signal = P * (error + D * (error - lastError) / Time.fixedDeltaTime)`,
with `PForce = 8f` (range 0–160) and `DForce = 0.01f` (range 0–0.064). Per-bone profiles:
`positionSpring * maxJointTorqueProfile[i]` and `positionDamper * jointDampingProfile[i]`.
`SlaveController` **reduces ragdoll strength during collisions** — directly applicable to
Crash. **The per-bone torque profile is the single most transferable idea here [inferred]:**
a Rider needs a stiff torso and a soft neck, and one global spring will not do.

**hairibar/Hairibar.Ragdoll (266★, MIT)** — the blending reference. README **[cited]**: an
**"alpha" parameter (0–1, where 1 = animation, 0 = physics)**; **partial ragdolls** —
"enable simulation on some bones and leave others purely keyframed"; per-bone
kinematic/powered/unpowered modes; `RagdollWeightDistribution` profiles that "control how
this mass is distributed between the ragdoll bones"; framed as a "post-processing effect"
on any animation system. **Partial ragdolls are directly useful: an upper body that goes
loose on a hard landing while the legs stay gripped is a Crash that hasn't happened yet.**
**[cited for the feature, inferred for the application]**

### PD `AddTorque` as an alternative

**[inferred, grounded in cited docs]**

- **Joint drives are solved *inside* the PhysX constraint solver** — Unity's ArticulationBody
  manual describes drives as "implicit springs" **[cited]**, and implicit integration is
  unconditionally stable at high stiffness.
- **`AddTorque` is explicit** — computed from last frame's error and integrated forward. At
  the stiffness needed to hold a Rider's pose against a hard-accelerating Bike, an explicit
  PD will overshoot and oscillate unless the timestep shrinks substantially.
- **Where `AddTorque` wins:** total control (feed-forward, gravity compensation, balance
  terms), no joint-space quaternion gymnastics, and it works on bodies with *no joint at
  all*.

**⭐ Recommendation [inferred]: slerp drives for the Rider's limb chain, and a separate
explicit PD `AddTorque` on the Rider's pelvis for the airborne rotation the Player
controls.** Lean and Air-save are a free-body orientation problem, not a chain problem.
One mechanism will not serve both well.

---

## 6. How animation and physics posing coexist, and what breaks

### There is no official Unity approach — a firm negative result

- The Unity 6 ragdoll section has exactly three children: *Create a ragdoll*, *Joint and
  ragdoll stability*, *Articulation Body component reference*. **No blending page.**
  **[cited]**
- The only transition guidance Unity publishes is the crude prefab swap: *"immediately
  delete the entire character and replace it with an instantiated wrecked prefab"* — and
  that page **exists in 6000.0 but 404s in 6000.5**. **[cited]**
- **Animation Rigging has no ragdoll or physics constraint** — all its constraints are
  kinematic. Its Bidirectional Motion Transfer explicitly states *"Humanoid is not supported
  at the moment."* **[cited]**
- **The old Standard Assets `RagdollHelper` is gone from everything official.** GitHub code
  search across the entire `Unity-Technologies` org for `RagdollHelper` returns
  `total_count: 0`; org repo search for `ragdoll` returns zero repos. Surviving copies are
  third-party re-uploads with no Unity licence grant. **[cited]**

**Budget for this as real engineering, not integration.**

### The architecture: two skeletons

1. A **hidden proxy skeleton** with the `Animator` playing the riding pose. Kinematic, no
   rigidbodies.
2. The **physical skeleton** — rigidbodies + ConfigurableJoints + colliders — whose
   `targetRotation`s are written each `FixedUpdate` from the proxy's local bone rotations.
   **Bind the SkinnedMeshRenderer to this one**, so there is nothing to re-sync.

**[inferred as architecture; every constraint below is cited]**

### What breaks

**Animator update mode — and the enum was renamed.** `AnimatorUpdateMode` in 6000.5 has
three values: `Normal`, **`Fixed`**, `UnscaledTime`. **`AnimatePhysics` no longer exists** —
its 6000.5 doc page 404s while the 2022.3 one still resolves. Renamed in 2023.1. But **the
manual still labels the Inspector dropdown "Animate Physics"**, so the label and the C#
identifier disagree. Write `AnimatorUpdateMode.Fixed`. **[cited]**

Why: with the Animator on `Normal` (Update) and joint-driving code in `FixedUpdate`, you
read a stale bone pose on some physics steps and skip poses on others. At 50 Hz physics and
144 fps render this is jitter that **no amount of spring tuning will fix**. **[inferred]**

(`Animator.animatePhysics` still exists as a `bool` and is a *different* thing — kinematic
rigidbodies imparting velocity to riders on moving platforms.)

**⭐ Animator culling — the silent killer.** **[cited]**

> `CullUpdateTransforms` — "Retarget, IK and write of Transforms are disabled when renderers
> are not visible."

The proxy skeleton **is** hidden — that is the whole point — so culling stops it writing
bone transforms. The target pose silently freezes at the last written frame and the Rider
goes limp or locks rigid, with **no error, no warning, no exception.** It looks exactly like
a joint-tuning bug. Set `cullingMode = AnimatorCullingMode.AlwaysAnimate`. Sergio's README
calls this out independently. **[cited for the modes and the README; inferred for the
failure chain]** The same bites when the camera cuts away from a Rider mid-Crash.

**Never write transforms directly on jointed kinematic bodies.** RagdollStability states it
flatly: *"Never use direct Transform access with Kinematic Rigidbodies connected by
Joints"*. **[cited]** (That page then says to use `Rigidbody2D.MovePosition`/`MoveRotation`
— almost certainly a docs bug on a 3D page; the intent is `Rigidbody.MovePosition`. Flagged
so it isn't chased.) This is precisely the mistake of letting the Animator drive the
*physical* skeleton and hoping physics catches up. Hence the split.

**Kinematic vs non-kinematic under an Animator:**
- Non-kinematic Rigidbody + Animator on the same bone = **they fight**. PhysX writes the
  pose from simulation, the Animator overwrites it from the clip; the bone snaps visually
  while its velocity state is nonsense and the solver reads garbage. **[inferred]**
- Kinematic Rigidbody + Animator = fine, and is the standard idiom. **[cited — `Is
  Kinematic`: *"the physics system cannot apply forces… Unity can only move and rotate it
  via its Transform"*]**

### ⭐ The Crash blend — five things that go wrong

**[inferred throughout, corroborated by the three blending implementations cited above]**

1. **It is not just spring → 0.** Lerp **both** `slerpDrive.positionSpring` **and**
   `slerpDrive.maximumForce` toward zero. Leaving `maximumForce` high with a low spring
   still lets a large accumulated error deliver a torque spike.
2. **Keep some `positionDamper`.** Damper-only (spring 0, damper > 0) gives a **heavy, dead**
   limb rather than a flailing one — the difference between a convincing unconscious Rider
   and a rubber chicken.
3. **Do NOT set `Animator.enabled = false`.** The classic mistake. The proxy skeleton is what
   you blend *back toward* on recovery; kill it and there is nothing to lerp to, so getting
   up pops. Keep it running (AlwaysAnimate) throughout.
4. **Never disable joint angular limits.** Limits stop elbows inverting and are *independent*
   of drives — a fully limp ragdoll should still have its limits on. Blending touches only
   the drives.
5. **Nothing to re-sync.** Because the mesh is bound to the physical skeleton, there is no
   pop. This is the structural advantage of the two-skeleton design over the prefab-swap
   Unity documents — and the reason to bind the mesh to the *physical* skeleton, not the
   proxy.

Because a Crash is a spectacle to watch rather than a fail-state, the blend-out can be
slower and more theatrical than a typical game's — there is no respawn to rush back to.

**Partial ragdoll is the underused idea:** Hairibar's per-bone kinematic/powered/unpowered
modes mean an upper body can go loose on a hard landing while the legs stay gripped. That is
a spectrum of Crash severity rather than a binary. **[cited feature, inferred application]**

---

## Open questions

1. **⭐ Does PuppetMaster support pinning a puppet to a moving dynamic vehicle?** The
   load-bearing question, and unanswerable here — `root-motion.com` was DNS-unreachable.
   Check the RootMotion forum and `PuppetMaster/_DEMOS` before buying.
2. **PuppetMaster / Final IK / Ragdoll Animator 2 on 6000.5.9f1** — listings top out at
   6000.4.2f1. Untested. (Unity's refund policy covers LTS incompatibility.)
3. **Games that shipped this** — Trials, Descenders, MX vs ATV, Human Fall Flat, Gang
   Beasts. **Not researched**; the session's search budget was exhausted. Deliberately left
   blank rather than speculated. Highest-value follow-up queries: `gdcvault.com` for
   RedLynx/Trials sessions, "Antti Ilvessuo Trials physics interview", "RageSquid Descenders
   devlog rider rig".
4. **Mixamo's redistribution clause** — verified *absent* from the current FAQ, but the FAQ's
   question headings are JS-rendered so a block may not have been retrieved. View it in a
   browser and screenshot it with the date before launch.
5. **Adobe General Terms of Use** as they apply to Mixamo — not read.
6. **Mixamo stock character availability** (Y Bot, Vanguard, Remy…) — catalogue is behind an
   authenticated API. Unverified.
7. **PhysX version in 6000.5** — not stated in any primary Unity source.
8. **Fixed timestep** — whether 0.02 holds the Rider's joint chain is empirical.
9. **TGS vs PGS** — Unity's description favours TGS for high mass ratios and PhysX made it
   *their* default, but Unity ships PGS and instability reports exist. A/B test on the real
   rig.
10. **`Joint.enableCollision` default** — not stated in the docs.
11. **CCD on `ArticulationBody`** — `collisionDetectionMode` exists but no doc states whether
    continuous modes are honoured. Moot given the recommendation against it.
12. **MakeHuman, VRoid guidelines, Blender Studio rig licences** — sites blocked or down.
13. **Ragdoll Wizard slot list** — the 13 slots are named from the source's bone map;
    re-verify in-editor.

---

## Sources

**Unity 6000.5 documentation** — [Create a ragdoll](https://docs.unity3d.com/6000.5/Documentation/Manual/wizard-RagdollWizard.html) ·
[Ragdoll physics section](https://docs.unity3d.com/6000.5/Documentation/Manual/ragdoll-physics-section.html) ·
[Joint and ragdoll stability](https://docs.unity3d.com/6000.5/Documentation/Manual/RagdollStability.html) ·
[Configurable Joint](https://docs.unity3d.com/6000.5/Documentation/Manual/class-ConfigurableJoint.html) ·
[Character Joint](https://docs.unity3d.com/6000.5/Documentation/Manual/class-CharacterJoint.html) ·
[CharacterJoint API](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/CharacterJoint.html) ·
[Physics settings](https://docs.unity3d.com/6000.5/Documentation/Manual/class-PhysicsManager.html) ·
[Articulations](https://docs.unity3d.com/6000.5/Documentation/Manual/physics-articulations.html) ·
[ArticulationBody](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/ArticulationBody.html) ·
[ArticulationDrive](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/ArticulationDrive.html) ·
[Rigidbody](https://docs.unity3d.com/6000.5/Documentation/Manual/class-Rigidbody.html) ·
[Rigidbody.solverIterations](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-solverIterations.html) ·
[Rigidbody.interpolation](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Rigidbody-interpolation.html) ·
[Joint.massScale](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Joint-massScale.html) ·
[Joint.enableCollision](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Joint-enableCollision.html) ·
[ConfigurableJoint.targetRotation](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/ConfigurableJoint-targetRotation.html) ·
[slerpDrive](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/ConfigurableJoint-slerpDrive.html) ·
[RotationDriveMode](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/RotationDriveMode.html) ·
[JointDrive](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/JointDrive.html) ·
[Animator component](https://docs.unity3d.com/6000.5/Documentation/Manual/class-Animator.html) ·
[AnimatorUpdateMode](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/AnimatorUpdateMode.html) ·
[AnimatorCullingMode](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/AnimatorCullingMode.html) ·
[Animation Rigging 1.4](https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.4/manual/index.html)

**NVIDIA PhysX 5** — [Joints](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/Joints.html) ·
[RigidBodyDynamics](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/RigidBodyDynamics.html) ·
[BestPractices](https://nvidia-omniverse.github.io/PhysX/physx/5.1.3/docs/BestPractices.html)

**Unity source** — `Unity-Technologies/UnityCsReference` branch `6000.5`,
`Modules/PhysicsEditor/RagdollBuilder.cs` · [ml-agents](https://github.com/Unity-Technologies/ml-agents) (Apache-2.0) ·
[Standard-Assets-Characters](https://github.com/Unity-Technologies/Standard-Assets-Characters) (UCL)

**Licensing** — [Unity Asset Store EULA](https://unity.com/legal/as-terms) ·
[Unity Companion License](https://unity.com/legal/licenses/unity-companion-license) ·
[Unity-chan guidelines](https://support.unity.com/hc/en-us/articles/29102655807252-Guidelines-regarding-the-use-of-Unity-chan-in-your-app) ·
[Mixamo FAQ](https://helpx.adobe.com/creative-cloud/faq/mixamo-faq.html)
([archived 2026-07-24](http://web.archive.org/web/20260724091310/https://helpx.adobe.com/creative-cloud/faq/mixamo-faq.html)) ·
[SMPL Model License](https://smpl.is.tue.mpg.de/modellicense.html) ·
[Quaternius FAQ](https://quaternius.com/faq.html) ·
[Sketchfab Licenses](https://sketchfab.com/licenses) ·
[Neowin on Fuse CC](https://www.neowin.net/news/adobe-revamps-mixamo-and-will-soon-be-discontinuing-support-for-fuse-cc/)

**Asset Store** — [Starter Assets ThirdPerson](https://assetstore.unity.com/packages/essentials/starter-assets-thirdperson-urp-196526) ·
[Starter Assets Character Controllers](https://assetstore.unity.com/packages/essentials/starter-assets-character-controllers-urp-267961) ·
[Robot Kyle](https://assetstore.unity.com/packages/3d/characters/robots/robot-kyle-urp-4696) ·
[PuppetMaster](https://assetstore.unity.com/packages/tools/physics/puppetmaster-48977) ·
[Final IK](https://assetstore.unity.com/packages/tools/animation/final-ik-14290) ·
[Ragdoll Animator 2](https://assetstore.unity.com/packages/tools/physics/ragdoll-animator-2-285638) ·
[Simple Motocross Physics](https://assetstore.unity.com/packages/tools/physics/simple-motocross-physics-221408) ·
[Sport Rider](https://assetstore.unity.com/packages/3d/characters/sport-rider-fantasy-stylized-humanoid-rigged-298855) ·
[Biker Character Pro](https://assetstore.unity.com/packages/3d/characters/biker-character-pro-179179) ·
[Synty Modular Fantasy Hero](https://syntystore.com/products/polygon-modular-fantasy-hero-characters)

**Community and open source** —
[mstevenson/ConfigurableJointExtensions](https://github.com/mstevenson/UnityExtensionMethods/blob/master/ConfigurableJointExtensions.cs) (MIT) ·
[gist 4958837](https://gist.github.com/mstevenson/4958837) ·
[Quaternion wizardry for ConfigurableJoint (2008)](https://discussions.unity.com/threads/quaternion-wizardry-for-configurablejoint.8919/) ·
[How does targetRotation work](https://discussions.unity.com/t/configurable-joint-how-does-targetrotation-work/105248) ·
[sergioabreu-g/active-ragdolls](https://github.com/sergioabreu-g/active-ragdolls) ·
[hairibar/Hairibar.Ragdoll](https://github.com/hairibar/Hairibar.Ragdoll) ·
[ashleve/ActiveRagdoll](https://github.com/ashleve/ActiveRagdoll) ·
[RootMotion FAQ](https://rootmotion.freshdesk.com/support/solutions/articles/77000057786-faq) ·
[Ragdoll Animator 2 thread](https://discussions.unity.com/t/released-ragdoll-animator-2-easy-body-physics/949880) ·
[TGS vs PGS stability](https://discussions.unity.com/t/physx-upgrade-temporal-gauss-seidel-solver-much-less-stable-than-pgs/770772) ·
[Mixamo Is Not "End of Life"](https://community.adobe.com/t5/mixamo-discussions/mixamo-is-not-end-of-life-it-s-broken-and-fixable/m-p/15379311)

---

## 7. Shipped games — documented developer statements

Answers open question #3 above, which was left blank when the search budget ran out.
Researched separately. Claims below are direct developer quotes unless marked otherwise.

### Trials (RedLynx / Ubisoft) — the closest analogue, and it is explicit

Sebastian Aaltonen, lead graphics programmer at RedLynx, on Trials Fusion,
Digital Foundry tech interview, 3 May 2014:

> "The FMX stunt system is completely physics based. **Our rider is a powered ragdoll
> connected to the bike. We control the rider physics joints, by emulating forces that
> a real human being would do if he wanted to change his pose.** As our whole world is
> already physics based, the trick system didn't require any changes to the core
> physics engine itself."

https://www.digitalfoundry.net/articles/digitalfoundry-2014-trials-fusion-tech-interview

The same team on Trials HD, describing the shift away from faked coupling:

> "We also have physically modelled springs and shock absorbers on the bike, and the
> rider physics are now also simulated when he sits on the bike. **The rider pulls the
> handlebars for real when you lean forward; it's not just a baked animation and a
> faked impulse like it used to be in the past.**"

https://www.digitalfoundry.net/articles/digitalfoundry-tech-interview-trials-hd

**Why this matters for issue #2.** Trials is the most successful physics bike game
ever made, and it resolves the coupling question in favour of real coupling: rider
joint forces act on the bike through the handlebar connection, not through a scripted
impulse. It also confirms they moved *to* this from a faked version, which is evidence
about direction of travel rather than just preference.

Trials runs on **Bullet Physics** inside RedLynx's own engine, and they describe a
"deterministic online replay system", which is a useful precedent for ADR-0002.

**Not documented:** the crash/bail trigger condition. Nobody at RedLynx has published
what releases the rider from the bike. That remains an open design question for
issue #9, with no prior art to copy.

**No RedLynx GDC talk on rider or bike physics exists.** The Digital Foundry
interviews are the primary technical record.

### Skate (EA Black Box) — the per-joint blend weight, described in full

Scott Blackwood, Executive Producer, Gamasutra, 17 October 2008:

> "Our guys took these Drives and turned them into all the joints in your body... **we
> can create a full, physically accurate replica of the human body and all the
> joints**... We did mocap, and we have all that animation in the game, but **animation
> is a target**. With Drives, **I can say that I want to be 100 percent of that target,
> with every joint and limb, or I can be zero, which would be ragdoll**... I could even
> take my right arm and say, 'Well, that's going to be 10 percent, so it's super-limp,
> but my other one's 100.'"

https://www.gamedeveloper.com/design/new-tricks-scott-blackwood-talks-i-skate-i-and-i-skate-2-i-

This is the clearest published description of a rider-on-vehicle active ragdoll with a
**per-joint animation-follow weight**, and it maps directly onto this project's
"assists are tunable floats over an unassisted model" constraint. A per-joint weight of
zero is pure ragdoll; the crash state is the same system at a different value, not a
separate mode.

### MX vs ATV (Rainbow Studios) — same studio, no technical detail published

Rainbow Studios made Motocross Madness. Their later "Rider Reflex" system is documented
only at the level of control design, never physics attachment. Ian Wood, Rainbow
Studios, SPOnG, 15 May 2009: "we really wanted to separate this and be really strict
with the fact that the rider's on the right and the machine, the vehicle, is on the
left."

**No documented statement exists** on whether the MX vs ATV rider is a ragdoll, how it
attaches, or how wrecks trigger. The direct lineage from this project's source material
turns out to be a dead end for implementation detail.

### Gang Beasts (Boneloaf, Unity) — independent confirmation on wheel colliders

Official studio account, Unity forum, 13 August 2016:

> "**We use a custom character system which is a constant ragdoll controlled with
> forces, effectively a 'physics puppet.'**"

And, separately, in the same post: "**We've actively avoided wheel colliders due to
stability issues.**"

https://discussions.unity.com/t/official-physics-improvements/619788/24

That second line is a third independent data point against `WheelCollider`, from a
shipped Unity physics game, reached for unrelated reasons. See
`ground-handling-research.md`.

### Best available engineering talk on the technique

**"Physics Driven Ragdolls and Animation at EA: From Sports to Star Wars"**, Jalpesh
Sachania, Frostbite/EA, GDC 2018.
https://gdcvault.com/play/1025210/Physics-Driven-Ragdolls-and-Animation

Covers driven ragdolls following animation, improving animation follow, reducing bad
output poses, and feedback into the animation system. The most directly applicable
shipped-game active-ragdoll talk in the GDC Vault.

### Where no evidence exists

Searched and found nothing technical: **Descenders** (RageSquid), **Human: Fall Flat**
(No Brakes), **Fall Guys** (Mediatonic), **Steep** (Ubisoft Annecy). Community
reverse-engineering exists for several of these and should not be cited as a source.
