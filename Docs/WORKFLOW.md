# CrabCover — 2-Player Remote Development Workflow

Revision of the original workflow plan. The structure and most of the content are unchanged —
the original was sound. Changes are marked **[REVISED]** or **[NEW]** with the reasoning stated,
so you can see what moved and why.

---

## 1. Tech stack

- **Engine:** Unreal Engine 5.8
- **Version control:** Git + GitHub with **Git LFS** (mandatory — `.uasset`/`.umap` are binary
  and standard Git will break on them)
- **Project management:** Trello or Notion. Kanban: Backlog → To Do → In Progress → Review → Done
- **Communication:** Private Discord — voice, screen share, and a dedicated `#file-checkouts`
  text channel

### [NEW] LFS bandwidth is the constraint, not storage

You get **10 GB/month of bandwidth shared between both developers**, and LFS counts every
download — not every upload. Storage fills slowly; bandwidth burns fast once two people pull
daily against a real asset library.

Current usage is ~96 MB, which is nothing. This becomes live during Phase 4, when textures land.

- Watch **bandwidth**, revisit the plan around 6 GB on either metric
- Never run `git lfs pull` casually — it draws against the shared quota
- If you approach the cap, the fix is GitHub's paid data packs or moving LFS hosting, not
  deleting history

### [NEW] Fix `.gitattributes` before art lands — the fix is not retroactive

Currently tracked through LFS: `uasset, umap, fbx, png, jpg, wav, mp4`

Missing: **`tga, psd, exr, tif, bmp, ogg, mp3`**

This matters more than it looks, because line 1 of `.gitattributes` is `* text=auto`. Any binary
file *not* caught by an LFS rule gets line-ending normalization applied to it. That is silent
corruption, not inefficiency.

LFS tracking only applies to files committed **after** the rule exists. Add the missing patterns
now, while no art has been committed. Doing it later means rewriting history.

---

## 2. The golden rules

Unreal uses binary files. **You cannot merge them.** If you both edit the same map, one of you
loses that work entirely on push.

### Rule 1 — the checkout ping

Before opening a major Blueprint or Map, post in `#file-checkouts`:

> "Checking out `PC_CrabController`."

Your partner does not touch that asset until you push and post:

> "`PC_CrabController` is free."

### [REVISED] Rule 1a — `Lvl_Lobby` is single-owner, because the checkout ping cannot cover it

`Lvl_Lobby` is built on the Open World template and carries **143 World Partition files**:

```
138  Content/__ExternalActors__/Lvl_Lobby/
  5  Content/__ExternalObjects__/Lvl_Lobby/
```

One File Per Actor means two people genuinely *can* edit different actors in that level without
colliding — so "check out the whole map" is stricter than necessary. But the files are
hash-named:

```
Content/__ExternalActors__/Lvl_Lobby/0/5X/JP6HAHYY7TYEP5EAHZELDM.uasset
```

You cannot tell from Discord which actor that is. So the rule is simultaneously **too strict**
(it blocks safe parallel work) and **unenforceable** (you cannot name what you checked out, and
you will not know you collided until you push).

Resolve it one of two ways:

- **Declare `Lvl_Lobby` single-owner.** Simplest, and Rule 2 already implies it — level design
  belongs to Player A. *(Recommended.)*
- **Drop World Partition** and rebuild as a plain empty level, per `PROJECT_CONTEXT.md` §2.8.
  Do this now while the level is nearly empty, or not at all.

### Rule 2 — divide and conquer

- **Player A:** 2D UI (menus, widgets) and 3D level design
- **Player B:** core GameMode logic, networking, RPCs

**[REVISED] Known collision in this split:** `WBP_MainMenu` is simultaneously UI (A) and session
logic (B) — its Host button calls Create Advanced Session and then Open Level. `GI_CrabCover`
has the same problem: it is the game instance (B) but is wired to menu flow (A). `PC_CrabController`
drives both input (A) and replication (B).

Agree ownership on those three assets explicitly before Phase 2, or they will be the first
things you collide on.

### Rule 3 — pair program the hard parts

For genuinely hard integration work — Steam lobby, voice, host migration — get in a Discord
call, share one screen, and solve it together. Do not do this asynchronously.

### [NEW] Rule 4 — close the editor before pulling, and before any `.ini` edit

Pulling with the Unreal Editor open causes file-lock failures. This has **already happened** in
this project — `Saved/Logs/` from 26 Aug shows ten retries and then:

```
LogSavePackage: Error: Error saving '.../Content/BP_CrabCharacter.uasset'
```

It resolved without data loss that time. It will not always.

The sequence is: **close editor → pull → open editor.**

The same applies to `Config/*.ini`. The editor rewrites config files on close and will silently
discard external changes made while it was running.

**After any save, check the Output Log for `LogSavePackage: Error` before you trust it.**

### [NEW] Rule 5 — fresh clones need a build step

`Binaries/`, `Intermediate/`, `Saved/`, and `DerivedDataCache/` are gitignored by design. A fresh
clone will not open. The steps are:

1. Clone, then `git lfs pull` (once — this is the one legitimate use)
2. Right-click `CrabCover.uproject` → **Generate Visual Studio project files**
3. Build the `CrabCoverEditor` target
4. Then open the `.uproject`

---

## 3. [NEW] Branching model

There is currently no branching model — one branch, linear history, everything committed
straight to `main`. That is workable for two people but leaves no recovery point.

**Split the rule by file type, because that is the only split that matters here.**

| File type | Rule | Why |
|---|---|---|
| **Text** — `Config/*.ini`, `Source/`, `Docs/`, `.gitattributes` | Branch + PR freely | These merge cleanly. Review is real. |
| **Binary** — `.uasset`, `.umap` | Commit to `main` behind the Rule 1 checkout ping | Branching buys nothing when you cannot merge. Long-lived asset branches actively hurt. |

**Tag at the end of every phase** (`git tag phase-1-complete`). That gives you a known-good
checkpoint to fall back to, which you do not currently have. With unmergeable binaries and no
tags, a bad push has no cheap undo.

### [NEW] Define what "Review" means

You cannot diff a Blueprint. The Review column has to mean one of:

- **Text changes:** a GitHub PR, reviewed normally
- **Binary changes:** pull the commit and test it in PIE, or demo it on a screen share

Otherwise cards sit in Review meaning nothing.

---

## 4. The learning path

Learn in this order. Do not build final crab models or the beach level on day one.

1. **UE5 basics** — navigate the engine, place actors, use the Blueprint event graph
2. **The multiplayer framework (critical)** — this is the foundation:
   - **GameMode** — host only. Holds secret words and rules.
   - **GameState** — shared by everyone. Holds current timer and phase.
   - **PlayerController** — the player's brain. Local and on the host.
   - **PlayerState** — shared. Holds names, scores, alive/dead.
3. **RPCs** — how a client asks the server to act ("Server, I am pointing my claw at Player 3")
4. **Resources** — YouTube: "Unreal Engine 5 Multiplayer Framework Basics". Stephen Ulibarri and
   Ryan Laley are both solid on UE multiplayer.

### [NEW] Version caveat — you are on 5.8, most tutorials are not

UE 5.8 shipped mid-June 2026. Nearly all tutorial material targets 5.0-5.4. Expect drift in:

- **Enhanced Input vs legacy input.** This project currently uses legacy `AxisMappings` (WASD in
  `DefaultInput.ini`) while the project defaults are set to Enhanced Input classes, and there are
  no IA/IMC assets in `Content/`. Rebindable keybinds and controller support both need Enhanced
  Input — plan to migrate rather than adding more legacy mappings.
- **Advanced Sessions plugin.** Vendored into `Plugins/`, and **unverified for 5.8**. It is a
  community plugin that historically lags engine releases. Confirm which branch was pulled — a
  5.5/5.6 build that happens to compile may break subtly.

When a tutorial and the engine disagree, the engine is right.

---

## 5. The roadmap

### [NEW] Parallel track — start now, independent of all phases

**Get a real Steam AppID.** You are on 480 (Spacewar), which cannot access **Steam Workshop,
Steam Cloud, or Steam Inventory**. That blocks Workshop word packs (PRD §5.6) and all XP/cosmetic
storage (PRD §6) — two entire PRD sections that cannot even be prototyped.

Steam Direct is **$100, plus review, plus a 30-day pre-release hold**. That is the longest lead
time in the project and it is not on the critical path of anything else. Start the clock now.

---

### Phase 1 — Greybox network test  *(~90% done)*

**Goal:** Player 1 hosts, Player 2 joins, both spawn in a grey room as default mannequins.

**Working:** Steam overlay in standalone; host creates session; Steam invite join; two players in
one replicated world; movement replicating.

**Remaining — one task:** the character mesh renders nothing. In `BP_CrabCharacter`:

1. Assign a Skeletal Mesh (`SKM_Quinn` or `SKM_Manny`) to **Mesh (CharacterMesh0)**
2. Set that component's **Rotation Z = -90**, **Location Z = -90** — UE skeletal meshes face +Y
   and pivot at the hips; without this the character faces sideways and sinks into the floor
3. Set **Anim Class** to `ABP_Quinn`, or the mesh T-poses and slides

This single task closes Phase 1 and unblocks all visible multiplayer testing.

---

### Phase 2 — Logic prototype

**Goal:** mannequins sit at a table, receive a secret word, and vote via UI.

**Focus:** GameMode logic, phase timers, role distribution, voting arrays.

**Prerequisite:** `Docs/SPEC_DECISIONS.md` must be filled in first. D1 (loop structure) and D2
(role composition table) are hard blockers — role assignment and the phase machine cannot be
written correctly against two contradictory specs.

#### [REVISED] Voice spike moved here from Phase 3

Voice was Phase 3. It is the highest-risk integration in the project and it dictates architecture
decided in Phase 2, so a **spike** belongs here.

You do not need working voice in Phase 2. You need a one-week spike answering two questions:

1. Can an **EOS Voice** channel run alongside a **Steam** session? (Steam for sessions, EOS for
   voice — they are expected to coexist, but confirm it.)
2. Does the API expose a **per-player amplitude value** you can drive animation from?

Both answers gate real design work: the PRD's proximity voice, VAD-driven animation, and the
§5.8 accessibility fallback all assume particular answers. Discovering otherwise after building
a logic prototype around them is expensive.

---

### Phase 3 — The "Liar's Bar" update

**Goal:** full voice chat, crab models replacing mannequins, mic amplitude driving animation.

**Focus:** audio routing, 3D character animation Blueprints, emotes.

**[REVISED] EOS Voice only — not Vivox.** Vivox is now a commercial Unity product and Epic
documents migrating away from it.

**[REVISED] Animation channels.** Crabs have no jaw, so lip-sync and viseme blendshapes do not
apply — the PRD's "mouth/mandibles move" is humanoid advice misapplied. The channels that work
are **eyestalks** (swivel toward the speaker), **claws** (gesticulation scaled to mic amplitude),
and **body bob** (shell rocking on voice activity).

**Art budget constraint:** four species on **one shared skeleton**, so animations are authored
once and retargeted. Four independent rigs means animating everything four times.

---

### Phase 4 — Polish and juice

**Goal:** beach bucket environment, table physics props, SFX, customization.

**Focus:** lighting, physics, UI polish.

**[REVISED] Steam Workshop integration is gated on the parallel track above.** If the AppID is
not through Steam Direct by the time you reach Phase 4, Workshop cannot be built or tested at
all. This is the reason to start that clock in week one.

---

## 6. Recurring hazards — check these when something breaks

| Symptom | First thing to check |
|---|---|
| Networking fails silently | `Saved/Logs/` — read the most recent log before speculating |
| Steam features dead, sessions fail | Is the Steam client running? OSS falls back to NULL without it, silently |
| Asset changes vanish after save | Output Log for `LogSavePackage: Error` — a file lock (Rule 4) |
| `.ini` change had no effect | Editor was open when you edited it; it rewrote the file on close |
| Fresh clone will not open | Missing the build step (Rule 5) |
| Partner's work disappeared | Rule 1 was skipped on a binary asset. Check the reflog before panicking. |

---

## 7. Summary of changes from the original plan

| # | Change | Why |
|---|---|---|
| 1 | Voice spike moved Phase 3 → Phase 2 | Highest technical risk; dictates Phase 2 architecture |
| 2 | Steam AppID added as a parallel track starting now | 6+ week lead time; blocks two whole PRD sections |
| 3 | LFS **bandwidth** quota documented | The actual constraint; was unmentioned |
| 4 | `.gitattributes` gaps flagged as pre-Phase-4 work | `* text=auto` silently corrupts untracked binaries; not retroactive |
| 5 | Branching model added (split by file type) + phase tags | No recovery point existed |
| 6 | `Lvl_Lobby` single-owner rule | Checkout ping cannot address hash-named World Partition files |
| 7 | "Close the editor before pulling" rule added | Already caused a save failure on 26 Aug |
| 8 | Fresh-clone build steps added | Clone alone does not produce an openable project |
| 9 | "Review" defined for binary assets | Cannot diff a Blueprint |
| 10 | Vivox removed; UE 5.8 tutorial-drift caveat added | Vivox is no longer viable; 5.8 postdates most material |
