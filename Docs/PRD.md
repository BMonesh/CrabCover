# Product Requirements Document (PRD) — v2

> **Official PRD for CrabCover.** Confirmed as the current version. Where this document and
> `Docs/GameFlowSpec.md` disagree, see `Docs/SPEC_DECISIONS.md` for the resolved decision.

**Game Title:** Crab Cover
**Genre:** 3D Tabletop Social Deduction / Party Game
**Engine:** Unreal Engine 5 (UE5)
**Document Status:** Approved / Ready for Design Phase
**Version:** 2.0 — expanded loop, retention, accessibility & safety

**Changelog from v1:** Added multi-round discussion, tie-vote resolution, defense moment,
progression/retention system, accessibility & non-voice support, anti-grief via host disconnect
(no vote-kick), Steam Workshop content, difficulty tuning, and reconnection handling. New/changed
items are tagged [NEW in v2] or [UPDATED].

---

## 1. Product Vision & Elevator Pitch

**Elevator Pitch:** Crab Cover is a 3D multiplayer tabletop party game where players embody goofy,
highly-animated crustacean characters playing a high-stakes word game. Using proximity voice chat,
expressive avatars, and physics-based table interactions, players must bluff, deduce, and argue
their way to survival to find the "Undercover" crab hiding among them.

**Core Value Proposition (USP):** Combining the proven, highly engaging mechanics of word-based
social deduction (like Undercover or Spyfall) with the hilarious, highly-clippable 3D physical
roleplay of games like Liar's Bar and Lethal Company, set in a bright, colorful, and chaotic beach
environment.

---

## 2. Target Audience & Platforms

- **Target Audience:** PC Gamers, Streamers, and friend groups (ages 13–35) looking for a casual,
  highly social, and funny party game.
- **Primary Platform:** PC (Steam).
- **Input:** Mouse and Keyboard (UI navigation and emote hotkeys). [Open question flagged in §10 —
  controller support undecided.]
- **Monetization Strategy:** Premium (paid upfront, aggressive "impulse buy" indie pricing e.g.,
  $2.99–$4.99) to lower the barrier for friend groups, with potential for cosmetic DLC (hats, claw
  skins, shell types). [UPDATED] All progression unlocks are cosmetic only — no pay-to-win, no
  gameplay behind purchase.

---

## 3. Core Gameplay Loop

A single match lasts approximately 5–10 minutes and consists of the following phases:

1. **Lobby:** 4 to 10 players join a host's room. Players select their crustacean species and claim
   a seat. The Host selects the Word Pack (Preset, Custom, or Workshop) and match settings (rounds,
   roles, difficulty).
2. **Word Distribution:** The game secretly assigns roles and a secret word to each player (e.g.,
   Civilians get "Apple", Undercover gets "Pear").
3. **The Discussion Phase (The Core) — [UPDATED]:** Players take 1–3 configurable rounds of
   turn-based describing (see §5). Each round, players describe their word via voice chat without
   saying it, using 3D emotes, snapping claws, and table physics.
4. **The Vote — [UPDATED]:** After the final discussion round, players point their claws to vote.
   Ties trigger the resolution flow (see §5).
5. **Defense Moment — [NEW in v2]:** Before elimination is finalized, the most-voted player gets a
   brief timed plea (default 10s) to defend themselves. Highly clip-worthy.
6. **Reveal & Resolution:** The eliminated player's role is revealed. If the Undercover is caught,
   Civilians win. If the Undercover survives to the final two, they win.

---

## 4. Roles & Game Modes

- **The Citizens (Majority):** Receive the main secret word. Goal: Find the Undercover.
- **The Undercover (1–2 players):** Receives a slightly different, related word. Goal: Survive the
  vote and figure out the Citizens' word.
- **Mr. Blank / The Clueless (1 player — Optional):** Receives NO word. Goal: Blend in by
  listening, survive the vote, and guess the word at the end.
- **The Jester (1 player — Optional):** Goal: Act suspicious and get voted out on purpose. Adds
  chaos.

**Game Modes — [NEW in v2, placeholder]:** Beyond the default match, the game reserves an
"Alternate Modes" slot in the lobby as an expansion surface (e.g., time-limited, team-based, or
twist variants). Specific modes are intentionally undefined at this stage and will be designed in a
later pass.

---

## 5. Key Features & Mechanics

### 5.1 Discussion Rounds — [UPDATED]

- Host-configurable 1–3 description rounds before the vote, set in the lobby.
  - 1 round: fast games / experienced groups.
  - 2 rounds (default): Round 1 = vaguer clue, Round 2 = sharper clue; rewards reading the room.
  - 3 rounds: deeper deduction for larger or hardcore lobbies.
- Turn order is fixed per match; a visible indicator shows whose turn it is (ties into the Control
  Rig "look-at" so crabs turn to the speaker).
- Per-turn soft timer (default ~30s) keeps pacing tight and prevents stalling.

### 5.2 Voting & Tie Resolution — [NEW in v2]

- Players vote by pointing claws at a suspect after the final round.
- On a tie: each tied player gets a defense moment (timed plea), followed by a single re-vote among
  only the tied players.
- If still tied after the re-vote: no elimination that round — the game proceeds to another
  discussion round (or ends the match per win conditions). This avoids random/coin-flip
  eliminations while preventing infinite stalls.

### 5.3 Voice Chat Integration

- Spatial audio via Epic Online Services (EOS).
- Voice Activity Animation (VAD): Avatars procedurally animate (mouth/mandibles move, bubbles emit)
  when the mic detects audio.

### 5.4 Expressive Emote System

- Hotkeys (1–5) trigger animations: accusatory claw point, hiding inside a shell, nervous
  sweating/shaking, slamming the table.

### 5.5 Fidget Physics

- Players pick up and drop small physics objects on the table (pearls, plastic shovels, seaweed) via
  mouse drag-and-drop to keep hands busy while thinking.

### 5.6 Custom, Preset & Workshop Word Packs — [UPDATED]

- Ships with robust default categories ("Everyday Items," "Hardcore," "Animals," etc.).
- In-UI Custom Pack creator — host types their own word pairs before starting.
- Steam Workshop support — [NEW in v2]: community-created packs for effectively infinite free
  content and long-tail engagement.
- Difficulty tuning — [NEW in v2]: host sets how similar the Undercover word is to the Citizens'
  word (e.g., Easy = "Apple/Orange", Hard = "Apple/Pear"), tuning deduction challenge.

### 5.7 Ghost Interactions (Spectator Mode)

- Eliminated players become "ghosts" (hovering seagulls or translucent floating crabs). Ghosts
  gently interact with physics objects (knocking over cups) and participate in a secondary "ghost
  vote" to stay engaged during downtime.

### 5.8 Accessibility & Non-Voice Support — [NEW in v2]

Voice is core, but a large slice of the audience (streamers' friends, mic-less players,
hard-of-hearing) must remain fully playable:

- Text / Quick-Phrase describe mode: players without a mic can type a short clue or pick from
  preset phrases during their turn; delivered via a speech bubble above their crab.
- Push-to-describe: clear turn indicator + push-to-talk so mic-shy players speak only on their turn.
- Captions/subtitles where technically feasible for voice, plus on-screen text for all system
  prompts.
- Colorblind-friendly UI and remappable keybinds.

### 5.9 Anti-Grief & Social Safety — [NEW in v2]

- Host-only disconnect: the host can remove a disruptive player from the room.
- Explicitly NO vote-kick — decided against, as vote-kick is easily abused (players kick whoever is
  winning or whoever they suspect).
- Per-player local mute (client-side) so anyone can silence a noisy player without affecting the
  match.
- Basic AFK handling: idle players are skipped on their turn and flagged.

---

## 6. Progression & Retention — [NEW in v2]

A premium party game needs a reason to relaunch. All rewards are cosmetic and earned through play
(no purchase required, no gameplay advantage):

- Match XP & leveling with cosmetic unlocks (hats, claw skins, shell types, emotes).
- "Crab of the Match" award at the end of a game (e.g., best bluffer, sharpest detective).
- Post-game stats screen: "You fooled 4 players," "Caught the Undercover 3 games in a row,"
  win/loss and role-based stats.

Feeds naturally into the existing cosmetic DLC strategy — earned cosmetics sit alongside premium
ones.

---

## 7. Technical Architecture (Unreal Engine 5)

- **Engine Framework:** UE5 using Blueprints for game logic and UI.
- **Networking Topology:** Listen Server (Client-Host). One player hosts, others connect.
- **Relay / Matchmaking:** Epic Online Services (EOS) for NAT punch-through, lobbies, and seamless
  Steam integration (Voice and Data).
- **Audio Sync:** Unreal Engine Audio / EOS Voice RTC.
- **Animation:** UE5 Control Rig for procedural idle animations and look-at targets (crabs turn
  their heads to whoever is speaking).
- **Reconnection Handling — [NEW in v2]:** Graceful handling of drops on a listen-server:
  - Client drop: player can rejoin an in-progress match into their seat within a grace window.
  - Host migration (stretch goal): if the host drops, attempt to migrate host to another player
    rather than killing the match — a known failure mode for listen-server party games. If
    migration proves infeasible in UE5/EOS, fall back to a clean "host left" screen with a fast
    re-lobby.

---

## 8. Art & Audio Direction

- **Visual Style:** Bright, colorful, goofy. Stylized cartoon graphics (Party Animals, Fall Guys,
  Crash Bandicoot).
- **Environment Ideas:** Inside a giant colorful plastic beach bucket, a sunny tide pool, or on top
  of a discarded cooler box on a vibrant beach.
- **Avatar Customization:** Distinct crustacean species with unique silhouettes and animations
  (standard Crab, Hermit Crab dragging objects, Lobster, Fiddler Crab with one giant claw).
- **Audio:** Upbeat beachy ambience (seagulls, waves) with satisfying, exaggerated "clacks" for claw
  movements and physics objects hitting plastic/wooden surfaces.

---

## 9. Backlog / TBD (Design Phase)

Pending final creative decisions, tackled during prototyping:

- **The Elimination Animation:** What happens to a voted-out player? (Prototype ideas: a giant
  seagull swoops and grabs them, a kid's plastic shovel scoops them up, or a giant plastic beach
  vacuum sucks them up.)
- **Specific Alternate Modes (§4)** — to be designed once the core loop is validated.
- **Host migration feasibility (§7)** — spike during prototyping.

---

## 10. Open Questions / Decisions Needed

These are flagged for your input; defaults chosen where noted:

- **Controller support:** PRD currently says Mouse & Keyboard only, but drag-and-drop physics +
  emote hotkeys could map to a controller. Steam party games lean heavily on controller/couch play.
  Decision needed.
- **Defense moment length** — defaulted to 10s. Tune during prototyping.
- **Per-turn timer length** — defaulted to ~30s. Tune during prototyping.
- **Captions for live voice** — technical feasibility (real-time transcription) to be validated;
  may ship as post-MVP.
