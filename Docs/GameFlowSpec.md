# CrabCover — Game Flow Specification

Detailed functional overview of each screen/state in the game flow, from launch through match rematch. Intended as a reference for implementation and for AI-assisted coding — each section documents what a screen/state does, its UI elements, triggers, and exit conditions, so behavior stays consistent and nothing gets forgotten.

---

## 1. Main Menu

**Type:** Local, client-only. No network session exists yet.

**On entry:**
- Player spawns into a 3D scene (no gameplay logic active).
- Main Menu UI (widget) is shown.

**UI Elements:**
- **Host Game** (button) — initiates hosting flow.
- **Join Game** (button) — initiates joining flow.
- **Customize** (button) — opens crab cosmetic customization (not yet specified — separate section later).
- **Settings** (button) — opens settings panel.
  - **Audio tab:** Master volume, Voice Chat volume, Music volume, SFX volume.
  - **Video tab:** Graphics quality preset, fullscreen toggle, brightness.
  - **Controls tab:** Rebindable keybinds (movement, push-to-talk, emote wheel/menu).
- **Steam Overlay Invite (Shift+Tab):** available at any point on this screen; accepting an invite routes into the Join flow (see Section 3) directly, bypassing the Join Game button/server browser.

**Exit conditions:**
- Host Game → Section 2 (Host Flow)
- Join Game → Section 3 (Join Flow)
- Accept Steam Invite → Section 3 (Join Flow), skips server browser

---

## 2. Host Flow

**Trigger:** Player clicks "Host Game" from Main Menu.

**Step 1 — Lobby Setup Screen:**

Shown before the Steam Session is created. Host configures the match here.

- **Room Name** (text input) — host names the lobby.
- **Lobby Visibility** (dropdown) — public / friends-only / invite-only.
- **Max Players:** fixed at 10.
- **Role Settings:**
  - **Mr. Blank count** — auto-suggested based on total player count (game recommends a value, e.g. suggests 0 Mr. Blanks at low player counts like 4, since Mr. Blank is very hard to play with few players). Fully host-overridable — Mr. Blank is optional and can be set to 0.
  - **Undercover count** — auto-suggested based on total player count. Host-overridable, but **at least 1 Undercover is required** at all times (cannot be set to 0).
  - **Jester** (toggle) — on/off. If on, exactly 1 Jester is included in the match.
  - **Validation rule:** `Mr. Blank count + Undercover count < Civilian count` must always hold (i.e. special roles combined must not reach or exceed the number of civilians — otherwise the round has no real "cover" to find). Setup UI blocks confirming/starting if this is violated.
- **Word Pack Selection:**
  - Choose from preset word packs (dropdown/list).
  - **Add Word Pack** (inline action) — host imports a custom word pack directly in this screen via **CSV**. Each CSV row is a pair: civilian word, undercover word. Newly added packs become available for selection immediately and persist for future lobbies.

**Step 2 — Session Creation:**
- On confirming setup, client requests creation of a Steam Session using the configured room name/visibility.
- **On success:** Host performs a server travel to the "Lineup Lobby" map, becoming the authoritative server. Lobby settings (roles, word pack) travel with the host as session state.
- **On failure:** A popup/error dialog is shown (e.g. "Failed to create session, try again"). Host remains on the Lobby Setup screen.

**Settings persistence:** Lobby settings (roles, word pack) are set at creation and used for the match. They **can be changed by the host after a match ends**, when choosing to "Play Again" (see Section 8 — End Match & Rematch) — not locked in permanently, editable between rounds.

---

## 3. Join Flow

**Trigger:** Player clicks "Join Game" from Main Menu, **or** accepts a Steam Invite via Shift+Tab overlay (from anywhere on the Main Menu — see Section 1).

**Path A — Server Browser:**
- Shown when "Join Game" is clicked.
- Lists joinable lobbies. Only **public** and **friends-only** lobbies (per host's visibility setting) appear here — **invite-only** lobbies never show up in the browser and can only be reached via direct Steam Invite or entering their room code.
- Each row displays: **Room Name**, **Current/Max Players** (e.g. "6/10"), **Visibility** tag (Public / Friends).
- **Room Code entry field:** alternative to browsing — every lobby (regardless of visibility) has a room code. Entering a valid code joins directly, same as clicking a browser row.
- Selecting a row (or submitting a valid code) → attempts to join.

**Path B — Steam Invite:**
- Player accepts an invite from the Steam overlay.
- Bypasses the Server Browser entirely — goes straight to a join attempt against the inviting host's session.

**Join Attempt (both paths converge here):**
- Client sends a Join Steam Session request targeting the selected/invited session.
- **On failure:** popup/error dialog shown with a specific message per case:
  - Session full → "Lobby is full"
  - Invalid/unknown room code → "Room not found"
  - Unreachable/other → "Could not join lobby"
  Player remains on Main Menu / Server Browser, free to retry.
- **On success:** server checks whether a match is currently in progress for that session:
  - **No match in progress** → client travels to the "Lineup Lobby" map (see Section 4), joining as a normal lobby participant.
  - **Match in progress** → client travels to the "Gameplay Table" map directly, joining as a **late-join spectator** rather than a seated player (see Section 6 — Gameplay Initialization for spectator behavior).

**Spectator cap:** No separate spectator limit — spectators still count against the lobby's overall max of **10 players total**. Once a session (seated players + spectators) reaches 10, further joins fail with "Lobby is full."

---

## 4. Lineup Lobby

**Type:** Networked, 3D shared space. All connected players (host + joined clients) are present here simultaneously, synced across the session.

**On entry:**
- All players (whether they arrived by hosting or joining) spawn in a line, facing the camera.
- Voice/mic check runs automatically per player — a status icon indicates whether each player's mic is active.

**What players can do here:**
- **Customize outfit** — same cosmetic customization as available from Main Menu, usable live in this space so others can see changes.
- **View lobby settings (read-only for non-host):** role counts (Mr. Blank / Undercover / Jester on-off) and which word pack(s) are in use. Actual words are never shown/listed — only pack names/metadata.
- **Host-only: Edit settings panel** — the host can open the same settings panel from Lobby Setup (Section 2) directly in this space to adjust roles/word pack before starting. This is the same panel players return to after a match when the host chooses "Play Again" (see Section 8), so changes between rounds happen here without leaving the lobby.

**Start Match:**
- **"Start Match"** button is visible **only to the Host**.
- The host can press Start Match at any time — **no ready-check is required** from other players; readiness of others does not gate the button.
- **Minimum player count: 3.** Start Match is disabled/greyed out until at least 3 players (including host) are in the lobby.
- Pressing it triggers Gameplay Initialization (Section 5).

---

## 5. Gameplay Initialization

**Trigger:** Host presses Start Match in the Lineup Lobby.

**Steps:**
- Server travel: all connected clients move from "Lineup Lobby" map to the "Gameplay Table" map.
- **Seating:** players are seated around a round table in **randomized** order (not the same order as the lineup) each match.
- **Role & word assignment** — server assigns each player one of:
  - **Civilian** — gets the "civilian" word from the active word pack. Not told they are a Civilian; role identity is never displayed.
  - **Undercover** — gets the paired "undercover" word from the pack. Also not explicitly told their role — same as Civilian, they only see their own word and must infer/deduce from play.
  - **Mr. Blank** (if enabled) — receives **no word at all** (blank card). Must bluff through the round without any word to go on.
  - **Jester** (if enabled) — receives a word like a Civilian/Undercover, but is a distinct win condition (wins if voted out — see Section 7).
  - No player is ever shown an explicit role label (e.g. "You are Undercover") — only their word (or blank, for Mr. Blank) is shown.
- **Personal word card:** each player has a private card, viewable on demand (e.g. swipe/pull gesture) showing only their own word. Not visible to other players.

**Late joiners during an in-progress match:**
- A client that joins via Section 3 while a match is already running spawns directly at the Gameplay Table as a **spectator ghost** — translucent, non-seated, cannot interact with cards/voting/table objects.
- Waits until the match ends and a new round begins (Section 8) to be seated.

---

## 6. Round Loop — Clue Phase

**Table environment:**
- Players are seated around a round table.
- A small interactive object sits on the table that players can idly interact with/play with during the round (fidget prop — purely social/flavor, not a game-mechanic item).
- Character heads turn to look at whichever player is currently speaking/acting — characters look at **people**, never at another player's card (cards are always private).

**Clue-giving:**
- Each round has a **turn order**: clockwise around the table, but the **starting player rotates each round** (round 1 might start at seat 3, round 2 at seat 6, etc.) — not always the same player going first.
- On their turn, a player states their word out loud (voice chat is live — players can talk, react, laugh, banter throughout).
- In addition to saying it by voice, the player's word is **written/displayed as text**:
  - Appears above their character's head in the 3D scene.
  - Also appears in a small persistent UI panel at the top of the screen, visible to everyone, so the clue is always legible.
  - **Why text is shown alongside voice:** relying on voice alone risks confusion (accents, cross-talk, background noise) when everyone is meant to track every clue — the text readout keeps the round unambiguous even if the room gets loud/chaotic.
- Play proceeds clockwise until every seated (non-eliminated) player has given their clue for the round.

**After a clue round:**
- Moves into the Voting Phase (Section 7).
- If the vote results in no elimination (players choose to pass — see Section 7), play returns to another Clue Phase round instead of ending the match.

---

## 7. Voting & Elimination

**Trigger:** End of a Clue Phase round (Section 6).

**Voting mechanic:**
- Each surviving (non-eliminated, non-spectator) player casts one vote by pointing their claw/hand at either:
  - **Another player** (accusing them), or
  - **Skip/Pass** (no accusation).
- Votes are **live/visible** — every player sees claws/hands pointing in real-time as others vote, not hidden until a final reveal. Enables social pressure/bandwagoning as part of the gameplay.

**Resolving the vote:**
- **Most votes on "Skip/Pass"** → counts as a collective skip. No elimination. Play returns to another Clue Phase round (Section 6).
- **Most votes on a single player** → that player is eliminated.
- **Tie between two or more players** (and that tie beats Skip/Pass) → a **runoff vote** happens, restricted to choosing only among the tied players (Skip is not an option in the runoff). Repeats until a single player has the most votes.

**Elimination:**
- The eliminated player is visually removed from the table — carried off by a seagull.
- They become a **spectator ghost** (same state as a late-joining spectator, Section 5) — can still watch the rest of the match but can no longer give clues, vote, or otherwise participate.
- The eliminated player's role/word is **not revealed** at elimination time — kept hidden from the table until the match fully ends and winners are revealed (Section 8). Preserves mystery/deduction for the remaining rounds.

**After elimination:**
- Server checks win conditions (below). If met, the match ends and proceeds to Section 8 (winner reveal).
- Otherwise, play returns to another Clue Phase round (Section 6) with the remaining seated players.

**Win Conditions (standard Undercover rules):**
- **Civilians win** when all Undercover and Mr. Blank players have been eliminated.
- **Undercover/Mr. Blank win** when their combined remaining count reaches parity with (equals or outnumbers) the remaining Civilian count.
- **Jester wins independently** if they are the one voted out/eliminated by the table — this is checked at the moment of elimination and ends the match immediately regardless of other roles' state, since the Jester's win condition is being eliminated, not surviving.

---

## 8. End Match & Rematch

**Trigger:** A win condition is met (Section 7).

**Winners Reveal screen:**
- Shows who won (which role/side, and the specific winning players).
- Shows every player's role.
- **Reveals the actual words used that match:** both the Civilian word and the Undercover word (the pack's word pair), so everyone can see what the "real" word vs. the "undercover" word was — not just abstractly who won.

**Play Again:**
- **"Play Again"** button, visible only to the Host (same privilege pattern as Start Match).
- On press, the server resets the Gameplay Table state:
  - Clears prior roles/words/votes.
  - **All spectators are seated for the new round** — this includes both players eliminated during the previous match and any players who joined late as spectators (Section 5) — bounded by the existing 10-player session cap.
  - Seats are re-randomized (per Section 5 seating rule).
- **All players** (not just the host) are sent back to the **Lineup Lobby** (Section 4) — the same space where they can change outfits and view lobby settings. The Host additionally has access to the Edit Settings panel there to review/change roles and word pack before the next round.
- From there, flow re-enters Section 4 (Lineup Lobby) → host presses Start Match again → Section 5 (Gameplay Initialization) for the new round.

---
