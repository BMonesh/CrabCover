# CrabCover

3D multiplayer tabletop social deduction party game. Players are crustaceans around a
table playing a word-based hidden-role game (Undercover / Spyfall mechanics) with
proximity voice chat, expressive avatars, and physics table props.

- **Engine:** Unreal Engine 5.8 (`EngineAssociation` in `CrabCover.uproject`)
- **Logic:** Blueprint-first. C++ hybrid only because custom plugins require a Source module.
- **Platform:** PC / Steam. Currently on Steam AppID 480 (Spacewar test ID).
- **Networking:** Listen Server (client-host), Online Subsystem Steam + SteamSockets.
- **Players:** 4-10 (min-player count is disputed — see `Docs/PROJECT_CONTEXT.md`).
- **Team:** 2 beginners working remotely. Explain reasoning; do not assume UE fluency.

## Hard constraint: most of this project is binary

`.uasset` and `.umap` files are binary and stored in Git LFS. You cannot read or edit
Blueprint graphs, levels, meshes, or materials.

- Never attempt to open, parse, or edit `.uasset` / `.umap`.
- Never run `git lfs pull` unless explicitly asked — it draws against a 10 GB/month
  bandwidth quota shared by two developers.
- When a task requires Blueprint changes, produce step-by-step editor instructions
  (panel names, node names, pin names) rather than attempting a file edit.

## What you can work on

| Area | Path |
|---|---|
| Engine/project config | `Config/*.ini` |
| Design specs | `Docs/` |
| C++ module | `Source/CrabCover/` |
| Build rules | `Source/CrabCover/CrabCover.Build.cs`, `Source/*.Target.cs` |
| Version control config | `.gitattributes`, `.gitignore` |
| Runtime logs (local only, gitignored) | `Saved/Logs/*.log` |
| Word pack data | CSV — format is `civilian_word,undercover_word` per row |

`Saved/Logs/` is the primary debugging surface for networking failures. When a Steam
host or join fails silently, read the most recent log there before speculating.

## Naming conventions

Already established in `Content/` — follow them:

- `BP_` Blueprint actor/character (`BP_CrabCharacter`)
- `GM_` Game Mode (`GM_CrabGame`)
- `PC_` Player Controller (`PC_CrabController`)
- `GI_` Game Instance (`GI_CrabCover`)
- `WBP_` Widget Blueprint (`WBP_MainMenu`)
- `Lvl_` Level (`Lvl_MainMenu`, `Lvl_Lobby`)

## Config editing rules

- `Config/DefaultEngine.ini` is load-bearing for Steam networking. Show a diff and
  explain each line before changing it.
- Never edit `.ini` files while the Unreal Editor is open — the editor rewrites them
  on close and will silently discard your changes.
- After any `DefaultEngine.ini` networking change, the verification step is: launch
  Standalone Game, press Shift+Tab, confirm the Steam overlay appears.

## Git rules

- Binary `.uasset` / `.umap` files cannot be merged. Two people editing the same asset
  loses one person's work. Before suggesting parallel work, check whether it touches
  the same assets.
- `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/` are gitignored by design.
  A fresh clone therefore requires "Generate Visual Studio project files" then a build
  before the project will open.

## Working style

- State assumptions explicitly. If a fact about UE 5.8 is uncertain, say so rather than
  guessing — 5.8 is recent and much online material targets older versions.
- Prefer small verifiable steps with a stated success condition over large refactors.
- Do not add features that are not in `Docs/GameFlowSpec.md`. Where the spec is silent
  or contradictory, flag it instead of inventing an answer.

## Current state

Working: Steam overlay in standalone, host creates session, Steam invite join,
two players in the same replicated world, character movement replicating.

Not yet done: visible character meshes, seating, role assignment, clue phase, voting,
voice chat.

Read `Docs/PROJECT_CONTEXT.md` for the full audit, pending fixes, and open questions.
