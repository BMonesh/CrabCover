# CrabCover — Project Context & Handoff

Carries forward the state of the project as of this handoff, so a new session does not
have to rediscover it. Sections marked **VERIFIED** were checked directly against the
repository or official documentation. Sections marked **UNVERIFIED** are open.

---

## 1. Build state

**Working:**
- Git + Git LFS repository, `.gitattributes` routing `.uasset`/`.umap` through LFS.
- Project converted to C++ hybrid (required for the Advanced Sessions plugin to compile).
- Steam plugins enabled: `OnlineSubsystemSteam`, `SteamShared`, `SteamSockets` (VERIFIED
  in `CrabCover.uproject`).
- Steam overlay confirmed in Standalone Game via AppID 480.
- `WBP_MainMenu` with a Host button calling Create Advanced Session, then Open Level
  (by Name) with the `listen` option.
- `GI_CrabCover` reparented to `AdvancedFriendsGameInstance`, handling
  `OnSessionInviteAccepted` → `Join Session`.
- Two machines confirmed in the same replicated world; character movement replicates.

**Assets present** (VERIFIED in `Content/`):
`Lvl_MainMenu.umap`, `Lvl_Lobby.umap`, `Untitled.umap`, `WBP_MainMenu`, `GI_CrabCover`,
`GM_CrabGame`, `PC_CrabController`, `BP_CrabCharacter`, `Untitled_HLOD0_Instancing`.

**Not built yet:** visible character meshes, seating, role/word assignment, clue phase,
voting, defense moment, elimination, ghost spectators, voice chat, cosmetics,
progression, word pack import.

---

## 2. Pending fixes (highest value first)

### 2.1 Character mesh invisible — BLOCKING
`BP_CrabCharacter` spawns and moves but renders nothing. Three steps, all in the
Blueprint editor:
1. Assign a Skeletal Mesh (`SKM_Quinn` or `SKM_Manny`) to **Mesh (CharacterMesh0)**.
2. Set that mesh component's transform: **Rotation Z = -90**, **Location Z = -90**.
   UE skeletal meshes face +Y and pivot at the hips; without this the character faces
   sideways and sinks into the floor.
3. Set **Anim Class** to `ABP_Quinn`, otherwise the mesh T-poses and slides.

### 2.2 Net driver fallback is a wrong class name
`Config/DefaultEngine.ini` currently has:
```ini
DriverClassNameFallback="/Script/SteamSockets.SteamNetSocketsNetDriver"
```
The SteamSockets module ships `SteamSocketsNetDriver`; `SteamNetSocketsNetDriver`
appears to be a transposition. Effect: if SteamSockets fails to initialise there is no
working fallback and networking dies silently. The configuration in current community
use is:
```ini
[/Script/Engine.GameEngine]
!NetDriverDefinitions=ClearArray
+NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="/Script/SteamSockets.SteamSocketsNetDriver",DriverClassNameFallback="OnlineSubsystemUtils.IpNetDriver")
```
Verify the exact class names against
`Engine/Plugins/Online/SteamSockets/Source` before applying.

Also consider adding under `[OnlineSubsystemSteam]`:
```ini
bAllowP2PPacketRelay=true
P2PConnectionTimeout=90
```
Relay is the fallback when a player's NAT will not punch through. Without it, joins fail
intermittently in a way that looks random.

The existing `[/Script/OnlineSubsystemSteam.SteamNetDriver]` block is dead config — it
targets the driver that `!NetDriverDefinitions=ClearArray` removes.

### 2.3 `GameDefaultMap` points at an engine template
```ini
GameDefaultMap=/Engine/Maps/Templates/OpenWorld
```
Packaged builds boot into an engine template instead of the main menu. Point it at
`Lvl_MainMenu`.

### 2.4 Widget variables are disabled
```ini
[/Script/Engine.UserInterfaceSettings]
bAuthorizeAutomaticWidgetVariableCreation=False
```
This is why UI elements do not appear in the Variables panel and require ticking
"Is Variable" individually. Set to `True`.

### 2.5 `.gitattributes` — line-ending hazard and gaps
First line is `* text=auto`, which normalises line endings on anything Git considers
text. Any binary not caught by an LFS rule can be silently corrupted. Current LFS
coverage: `uasset, umap, fbx, png, jpg, wav, mp4`. Missing: `tga, psd, exr, tif, bmp,
ogg, mp3, uexp, ubulk`. Fix before importing art.

### 2.6 Rendering settings over-specced for the art direction
`r.RayTracing=True` and `r.Substrate=True` are enabled. The target is bright stylized
cartoon crabs at a $2.99-$4.99 price point on ordinary consumer hardware. Ray tracing
inflates shader compilation and packaging time on every build; Substrate complicates
every material authored. Recommend disabling both before assets depend on them. Lumen
(`r.DynamicGlobalIlluminationMethod=1`) is defensible and can stay.

### 2.7 Repo debris
`Untitled.umap` and `Untitled_HLOD0_Instancing.uasset` are committed alongside
`Lvl_Lobby.umap`. Delete so nobody opens the wrong map.

### 2.8 `Lvl_Lobby` is built on the Open World template
World Partition is enabled, producing ~200 One-File-Per-Actor `.uasset` files under
`Content/__ExternalActors__/Lvl_Lobby/` for an essentially empty landscape.

Trade-off: OFPA genuinely helps a two-person team, because two people can edit different
actors in the same level without a binary merge collision. But landscape streaming is
built for kilometre-scale worlds and this game is a table in a bucket — it costs load
time, build time, and file churn. If switching to a plain empty level, do it now while
the level is nearly empty.

### 2.9 `OnlineSubsystem` commented out in `Build.cs`
```csharp
// PrivateDependencyModuleNames.Add("OnlineSubsystem");
```
Harmless while everything is Blueprint-side. The first C++ touching sessions or voice
will fail to link. At that point uncomment it and add `"OnlineSubsystemUtils"` and
`"Voice"`.

`Source/CrabCover/MyClass.h/.cpp` is the empty stub from C++ conversion. It does not
inherit `UObject` and does nothing. Safe to delete.

---

## 3. Documentation conflict — two sources of truth

`Docs/GameFlowSpec.md` is newer and better than PRD v3.1 (which lives outside the repo).
GameFlowSpec resolves several earlier gaps well: the role validation rule
(`Mr. Blank + Undercover < Civilians`) with host override, Skip/Pass voting, and runoff
tie resolution.

But the two documents now contradict each other:

| Topic | PRD v3.1 | GameFlowSpec |
|---|---|---|
| Minimum players | 4 | 3 |
| Role reveal timing | at elimination | at match end only |
| Discussion rounds | 1-3, host-configurable | until win condition |
| Undercover win | survives to final two | parity with civilians |
| Mr. Blank guess mechanic | central | absent |
| Turn timer | 30s + 10s defense | none specified |
| Disconnect handling | instant elimination | unaddressed |

**Two conflicts matter most:**

- **Timers vanished.** With no clock, one silent or AFK player stalls the table
  indefinitely. This was a solved problem in the PRD and is now reopened.
- **Mr. Blank's guess is gone.** That mechanic was what made the role playable rather
  than merely hard.

The stated project owner intent is **4+ players minimum**, which contradicts the spec's 3.

**Recommended next action:** merge both into one document, resolve each row above
explicitly, and delete the loser. The GameMode cannot be implemented correctly against
two conflicting specs.

---

## 4. Open design questions

### 4.1 Text clues (highest impact)
GameFlowSpec §6 has each player's word appear as text above their head and in a
persistent top-of-screen panel. The mechanism is unspecified.

- If players **type** clues, the round becomes a typing game and the 3D character layer
  loses the moments that justify it.
- If it is **live transcription**, that is a real service dependency needing scope.
- A middle path: voice is primary, and text is either a short confirmation the speaker
  taps after speaking aloud, or a host-toggled accessibility option.

### 4.2 Animation set
Established direction: animations should be driven by game actions rather than a generic
emote hotbar. The claw vote is the strongest candidate — a physical claw extended across
the table in real time, visibly wavering between targets before committing, gives
readable social information that a UI click cannot.

Crabs have no jaw, so lip sync and viseme blendshapes do not apply — the PRD's
"mouth/mandibles move" and Oculus LipSync references are humanoid advice misapplied.
Expression channels that do work: **eyestalks** (two bones each; swivel toward the
speaker, retract when nervous), **claws** (gesticulation scaled to mic amplitude),
**body bob** (shell rocking on voice activity).

**Critical art-budget constraint:** four species on **one shared skeleton**, so the
animation set is authored once and retargeted. Four independent rigs means animating
everything four times. Species chosen to differ in silhouette while sharing bones:
common crab, hermit crab (shell to hide in — a real animation, not a metaphor), fiddler
crab (one oversized claw for pointing), lobster (long body, antennae).

Priority animation list, all game-driven:
1. Idle / seated with eyestalk look-at
2. Speaking — VAD-driven, three intensity tiers
3. Vote extend — claw reach with hold/waver state
4. Vote lock — commitment
5. Receive word — identical across all roles so it leaks nothing
6. Defense moment — alone, spotlit
7. Elimination — seagull or shovel (still TBD)
8. Ghost — translucent, can nudge props

Player-triggered emotes come after these. Four or five, not ten.

Fidget props only matter if replicated and visible to others. Otherwise cut them.

### 4.3 Cold start
A paid game requiring four simultaneous buyers is the hardest launch problem in this
category. No matchmaking is planned — discovery is via Steam browser, room code, or
invite. Standard mitigation is a free demo that can join but not host: one person buys,
brings friends, some convert. Undecided. Party-pack pricing also undecided.

### 4.4 Steam AppID — longest lead time
Still on 480 (Spacewar). AppID 480 cannot access **Steam Workshop, Steam Cloud saves, or
Steam Inventory**. Workshop word packs and XP/cosmetic storage therefore cannot be
prototyped at all until a real AppID exists. Steam Direct is $100 plus review plus a
30-day pre-release hold. This clock should be started independent of build progress.

---

## 5. Verified external research

- **UE 5.8** released mid-June 2026. Epic states it is the last planned major UE5
  release as UE6 ramps up, with continued support for bug fixes and regressions. Good
  position: no forced mid-project upgrade.
  https://www.unrealengine.com/news/unreal-engine-5-8-is-now-available
- **Online Subsystem Steam is still supported in 5.8.** Epic describes the newer Online
  Services plugin as intending to "eventually replace" Online Subsystem, but OSS is what
  Advanced Sessions and current tutorials target. Stay on OSS.
- **Voice: use the EOS Voice Chat plugin.** It implements the engine's `IVoiceChat` and
  `IVoiceChatUser` interfaces, is enabled from the Plugin Browser, integrates via a
  Lobbies method or a Trusted Server method, and explicitly does not require programming
  against the EOS SDK directly. Lobbies method needs no self-hosted backend. Vivox is now
  a commercial Unity product; Epic documents migrating away from it to EOS.
  https://dev.epicgames.com/documentation/en-us/unreal-engine/voice-chat-with-epic-online-services
  EOS Voice and Steam sessions coexist: Steam for sessions/matchmaking, EOS for voice
  transport. The PRD's one-line "EOS Voice RTC or Vivox layered over Steam Lobby" should
  be replaced with this.
- **Advanced Sessions plugin — UNVERIFIED for 5.8.** No confirmed 5.8 build was found.
  It is a community plugin (mordentral) that historically lags engine releases. It is
  vendored into `Plugins/AdvancedSessionsPlugin/` in this repo, which is good for the
  partner's setup but version-locks it. Confirm which branch was pulled; a 5.5/5.6 build
  that happens to compile may break subtly.
- **Git LFS quota:** 10 GB storage and 10 GB bandwidth included, currently 0.1 GB of
  each. GitHub is fine for now. Watch **bandwidth**, not storage — LFS counts every
  download, so two developers pulling daily after art lands consumes it faster than
  storage fills. Revisit around 6 GB on either.

---

## 6. Immediate sequence

1. Start the Steam AppID process (longest lead time, independent of everything else).
2. Fix the character mesh (§2.1) — unblocks all visible multiplayer testing.
3. Apply config fixes §2.2-2.4.
4. Merge PRD and GameFlowSpec into one spec, resolving every row in §3.
5. Decide the text-clue question (§4.1) — it shapes the whole clue phase.
6. Lock the role composition table for every player count 4-10, and specify a win-condition
   check that runs after every elimination.
7. Prototype voice with EOS Voice Chat before committing to Phase 3 scope.
