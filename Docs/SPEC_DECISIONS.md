# CrabCover — Spec Decision Sheet

> **History only — do not build from this file.** The rules now live in `Docs/GameSpec.md`, which
> also contains later decisions that change some rows here (notably the voting flow in D6-D8).
> Keep this file for the reasoning behind each decision.

**Purpose:** resolve every contradiction between `PRD v2.0` and `Docs/GameFlowSpec.md` so the
GameMode can be implemented against a single source of truth.

**How to use:** work through this with both developers in one sitting. Fill in the
`**Decision:**` line for each row. When every row is filled, the two source documents get
merged into one spec and the loser is deleted — do not leave three documents alive.

**Status:** ✅ 17 of 18 decided. D18 (word pack file) postponed until before the first word pack is
shared or imported. Ready to merge into one spec.

> **PRD version:** **v2.0 is the official PRD**, saved in `Docs/PRD.md`. A "v3.1" is mentioned in
> `PROJECT_CONTEXT.md`, but no copy can be found, so it is not used.

---

## Blockers — nothing in the GameMode can be written until these two are settled

### D1. Loop structure

The single most consequential row. These are different state machines, not different tunings.

| | Position |
|---|---|
| **PRD v2 §3.3-3.4** | 1-3 configurable clue rounds, **then one vote** |
| **GameFlowSpec §6-7** | Clue round → **vote** → eliminate/skip → check win → repeat |

**Note — the PRD is internally inconsistent here.** §3 reads as a single linear pass with no
loop back, but §3.6's win condition ("Undercover survives to the final two") requires multiple
eliminations, which requires looping. As written, the PRD cannot produce its own win condition.

**Cost:** GameFlowSpec's version is the standard Undercover loop and is fully specified.
The PRD's version needs its loop-back defined before it can be built at all.

**Recommendation — adopt GameFlowSpec's skeleton, graft the PRD's pacing onto it.**
Vote every round, loop until a win condition fires. Preserve the PRD's host-configurable
round count by reinterpreting it as **clue rounds before each vote** (default 1) — this keeps
host control over pacing without breaking the elimination ladder.

**Decision: ✅ DECIDED — Option A, vote after every clue round.**
Every awake player gives a clue, then every awake player votes, then repeat until a win condition
fires. Voters may choose **Skip**, so a round can end with no elimination (see D6).

**Always exactly 1 clue round before each vote.** The PRD's host setting for 1-3 rounds is
dropped for now (can be added later).

---

### D2. Role composition table

**Neither document contains one.** The PRD gives ranges (Undercover 1-2, Mr. Blank 0-1,
Jester 0-1); GameFlowSpec gives a validation rule and says counts are "auto-suggested" but
never says what the suggestion is. Role assignment is the first function in the GameMode and
it currently has no specification.

Validation rule (GameFlowSpec §2): `Mr. Blank + Undercover < Civilians`

**Decision: ✅ DECIDED** — all four tables below are approved. No exceptions.

Rules behind the tables:
- **Undercover:** 1 at 4-6 players, 2 at 7-8 players, **3 at 9-10 players** (see D17). The same
  in every setup.
- **Mr. Blank:** optional, host opt-in, **6+ players only**, always exactly 1. **Neutral — on no
  one's side** (see D5). Takes a Civilian's seat.
- **Jester:** optional, host opt-in, **5+ players only**, always exactly 1. **Neutral — on no one's
  side** (see D16). Takes a Civilian's seat.
- **Table check:** `Undercover < Civilians` (Mr. Blank and Jester not counted). This **replaces**
  GameFlowSpec §2's rule `Mr. Blank + Undercover < Civilians`, which assumed Mr. Blank was on the
  Undercover side.

The **Starts at** column shows the D5 win count at the start of the match: Undercover vs everyone
else. In every setup the other players can survive at least one wrong vote.

**Table 1 — Mr. Blank off, Jester off:**

| Players | Civilians | Undercover | Check (U < C) | Starts at |
|---:|---:|---:|---|---|
| 4 | 3 | 1 | 1 < 3 ✓ | 1 vs 3 |
| 5 | 4 | 1 | 1 < 4 ✓ | 1 vs 4 |
| 6 | 5 | 1 | 1 < 5 ✓ | 1 vs 5 |
| 7 | 5 | 2 | 2 < 5 ✓ | 2 vs 5 |
| 8 | 6 | 2 | 2 < 6 ✓ | 2 vs 6 |
| 9 | 6 | 3 | 3 < 6 ✓ | 3 vs 6 |
| 10 | 7 | 3 | 3 < 7 ✓ | 3 vs 7 |

**Table 2 — Mr. Blank on** (6+ players):

| Players | Civilians | Undercover | Blank | Check (U < C) | Starts at |
|---:|---:|---:|---:|---|---|
| 6 | 4 | 1 | 1 | 1 < 4 ✓ | 1 vs 5 |
| 7 | 5 | 1 | 1 | 1 < 5 ✓ | 1 vs 6 |
| 8 | 5 | 2 | 1 | 2 < 5 ✓ | 2 vs 6 |
| 9 | 5 | 3 | 1 | 3 < 5 ✓ | 3 vs 6 |
| 10 | 6 | 3 | 1 | 3 < 6 ✓ | 3 vs 7 |

**Table 3 — Jester on** (5+ players):

| Players | Civilians | Undercover | Jester | Check (U < C) | Starts at |
|---:|---:|---:|---:|---|---|
| 5 | 3 | 1 | 1 | 1 < 3 ✓ | 1 vs 4 |
| 6 | 4 | 1 | 1 | 1 < 4 ✓ | 1 vs 5 |
| 7 | 4 | 2 | 1 | 2 < 4 ✓ | 2 vs 5 |
| 8 | 5 | 2 | 1 | 2 < 5 ✓ | 2 vs 6 |
| 9 | 5 | 3 | 1 | 3 < 5 ✓ | 3 vs 6 |
| 10 | 6 | 3 | 1 | 3 < 6 ✓ | 3 vs 7 |

**Table 4 — Mr. Blank and Jester both on** (6+ players):

| Players | Civilians | Undercover | Blank | Jester | Check (U < C) | Starts at |
|---:|---:|---:|---:|---:|---|---|
| 6 | 3 | 1 | 1 | 1 | 1 < 3 ✓ | 1 vs 5 |
| 7 | 4 | 1 | 1 | 1 | 1 < 4 ✓ | 1 vs 6 |
| 8 | 4 | 2 | 1 | 1 | 2 < 4 ✓ | 2 vs 6 |
| 9 | 4 | 3 | 1 | 1 | 3 < 4 ✓ | 3 vs 6 |
| 10 | 5 | 3 | 1 | 1 | 3 < 5 ✓ | 3 vs 7 |

---

## Core loop rows

### D3. Minimum players

PRD §3.1 and `CLAUDE.md`: **4**. GameFlowSpec §4: **3**.

At 3 players the composition is 2 civilians + 1 undercover, and a single wrong vote ends it —
there is no deduction, only a coin flip.

**Recommendation: 4.** Matches the PRD, `CLAUDE.md`, and the stated owner intent. Update
GameFlowSpec §4 and grey out Start Match below 4.

**Decision: ✅ DECIDED — minimum 4 players.** Start Match is disabled until 4 players are in the
lobby.

---

### D4. Role reveal timing

PRD §3.6: revealed **at elimination**. GameFlowSpec §7: hidden **until match end**.

- **Reveal at elimination** — the table learns whether it guessed right. Standard Undercover
  rule. Faster feedback, more readable for streamers.
- **Hide until end** — preserves mystery, but an eliminated civilian's death teaches the table
  nothing, which can make later rounds feel arbitrary.

**Recommendation: reveal at elimination.** Medium confidence — this is the most taste-driven
row on the sheet and the one where I would most readily defer to you. Note that when the
*undercover* is eliminated the match ends immediately either way, so this row only really
governs what happens when a **civilian** is voted out.

**Decision: ✅ DECIDED — host setting.** The host chooses in the lobby settings:

- **Show on elimination** — a voted-out player's role is revealed straight away. **(Default.)**
- **Show at match end** — roles stay hidden until the winners screen.

Either way, every role and both words are shown on the winners screen (GameFlowSpec §8).

---

### D5. Undercover win condition

PRD §3.6: undercover **survives to the final two**. GameFlowSpec §7: **parity** — undercover +
Blank combined equals or outnumbers remaining civilians.

The PRD's version breaks with 2 undercovers, which the PRD itself allows in §4: "final two" is
undefined when both survivors are undercover.

**Recommendation: parity (GameFlowSpec).** High confidence. It is the standard rule, it scales
to any undercover count, and it handles Mr. Blank cleanly. The PRD's phrasing only works in the
single-undercover case.

**Decision: ✅ DECIDED — parity, counted like Among Us.**

After every elimination the game counts who is still in the match (sleeping crabs count, per
D15):

- **Undercover** = Undercover still in the match
- **Everyone else** = Civilians + Mr. Blank + Jester still in the match

If the Undercover count is **equal to or greater than** everyone else, **the Undercover wins.**
If every Undercover **and** Mr. Blank has been eliminated, **the Civilians win.**

**Three sides, like Among Us:**

| Side | Who | Wins when |
|---|---|---|
| **Civilians** | Civilians | Every Undercover **and** Mr. Blank is eliminated |
| **Undercover** | Undercover | Undercover count ≥ everyone else |
| **Neutral — Mr. Blank** | Mr. Blank, alone | Voted out **and** guesses the Civilians' word (D10) |
| **Neutral — Jester** | Jester, alone | Voted out (D16) |

- Mr. Blank and the Jester **count with everyone else** when checking whether the Undercover has
  caught up — the same way neutral roles count in Among Us.
- But they are **neutral**: they **never share a win** with the Civilians or the Undercover. When
  the Civilians win or the Undercover wins, Mr. Blank and the Jester lose.
- The Civilians oppose every other role.

This differs from GameFlowSpec §7, which put Mr. Blank with the Undercover.

**Order of checks after every elimination:**
1. Was the Jester voted out? → **Jester wins**, match ends.
2. Was Mr. Blank voted out? → Mr. Blank guesses. Correct → **Mr. Blank wins**, match ends. Wrong →
   Mr. Blank is out, continue.
3. Are every Undercover **and** Mr. Blank out? → **Civilians win**, match ends.
4. Undercover count ≥ everyone else? → **Undercover wins**, match ends.
5. Otherwise, start a new clue round.

**Mr. Blank still in the match after the last Undercover is out:** the match **continues**. The
Civilians must also vote out Mr. Blank, who then gets their guess. Reason: the Civilians' word has
not been said out loud yet, so Mr. Blank still has to work it out.

Two edge cases, both decided:
- **(a) Endless tie.** When only Mr. Blank and one other player remain (ghosts not counted),
  **Mr. Blank must guess immediately**. Otherwise the two would vote for each other forever, since
  D7 ties end with no elimination.
- **(b) Sleeping Mr. Blank.** If a disconnected Mr. Blank is voted out, it **counts as a wrong
  guess**.

---

### D6. Skip / Pass vote

GameFlowSpec §7 offers it. PRD §5.2 does not.

Without Skip, every round forces an elimination — at 4 players the match is over in two rounds.
Skip is the pressure valve that lets a table say "we don't know yet."

**Recommendation: keep Skip, with a stall guard.** Cap consecutive skips (suggest 2) or cap
total rounds, then force an elimination vote. Unbounded skipping is how a match hangs forever.

**Decision: ✅ DECIDED — Skip is kept, with a limit.**

- Players can vote Skip. If Skip wins, the round ends with no elimination and a new clue round
  starts.
- **Stall guard:** after **2 Skips in a row**, Skip is not available in the next vote.
- **Missed votes:** every awake player must vote. A player who has not voted when the voting
  timer runs out **counts as Skip**.

---

### D7. Tie resolution

PRD §5.2: defense for each tied player → single revote among tied → **if still tied, no
elimination, continue**. GameFlowSpec §7: runoff "repeats until a single player has the most
votes" — which can loop forever with a deadlocked table.

**Recommendation: PRD's version.** High confidence. The PRD is simply the better-engineered
document on this row; GameFlowSpec's has no termination guarantee.

**Decision: ✅ DECIDED — PRD's version.** On a tie, each tied player gets a defense moment (D8),
then one re-vote between only the tied players. If it is still tied, nobody is eliminated and a
new clue round starts.

---

### D8. Defense moment

PRD §3.5: most-voted player gets a timed plea (default 10s) before elimination is finalized.
GameFlowSpec: **absent**.

**Recommendation: adopt.** Cheap to build (one timed state in the phase machine) and it directly
serves the PRD's stated value proposition — this is the clip-worthy moment.

**Decision: ✅ DECIDED — yes, 30 seconds** (the PRD's 10s default is replaced). Sleeping crabs
get no defense moment (D15c).

---

### D9. Turn timer

PRD §5.1: ~30s soft timer per turn, plus AFK skip (§5.9). GameFlowSpec: **absent**.

Without a clock, one silent or AFK player stalls the table indefinitely. This is a solved
problem in the PRD that GameFlowSpec reopened.

**Recommendation: adopt the PRD's timer and AFK handling.** High confidence — this is not
optional for a party game.

**Decision: ✅ DECIDED.**

- **Clue turn:** 30 seconds per player.
- **Voting:** one shared voting period for the whole table, like Among Us. Players discuss and vote
  during the same window.
  - **Length:** host chooses **60, 90 or 120 seconds**. Default **90**.
  - **Changing votes:** allowed until time runs out. Everyone can see the claws move.
  - A player with no vote when time runs out counts as **Skip** (D6).

---

### D10. Mr. Blank guess mechanic

PRD §4: Blank survives the vote and **guesses the word at the end**. GameFlowSpec §5: no guess —
Blank only bluffs.

Without the guess, Mr. Blank is pure disadvantage with no upside, and the role stops being
playable — it becomes merely hard.

**Recommendation: adopt the PRD's guess.** High confidence. Define *when* the guess happens
(on elimination? at match end? on reaching final two?) — the PRD says "at the end" without
specifying which.

**Decision: ✅ DECIDED — yes.** When Mr. Blank is voted out, they get **one guess** at the
Civilians' word. A correct guess wins the match for Mr. Blank.

- A correct guess wins for **Mr. Blank alone**, and the match ends. Nobody else wins.
- A wrong guess: Mr. Blank is out and the match continues.
- Mr. Blank is **neutral** (D5): does **not** win when the Civilians win or when the Undercover
  wins.

Reason: if Mr. Blank's win also counted for the Civilians, the Civilians would play to help Mr.
Blank win instead of hunting the Undercover.

---

### D11. Text clue delivery

The question `PROJECT_CONTEXT.md` §4.1 calls highest-impact. Both documents already answer it —
differently.

| | Position |
|---|---|
| **PRD §5.8** | Text is an **accessibility fallback** — mic-less players type or pick a preset phrase. Voice is primary. |
| **GameFlowSpec §6** | Text is **universal and always-on** — every clue appears above every head and in a persistent top panel. |

If every player types every clue, the round becomes a typing game and the 3D character layer
loses the moments that justify building it.

**Recommendation: the PRD's version.** High confidence, and it matches what `PROJECT_CONTEXT.md`
§4.1 independently recommended. Voice primary; typed/preset clues for players without a mic;
optionally a host toggle for always-on text as an accessibility setting.

**Decision: ✅ DECIDED — spoken, with typing as an option for anyone.** Clues are given by voice.
Any player may choose to type their clue instead, and it appears in a speech bubble above their
crab. GameFlowSpec §6's "every clue always shown as text" is dropped.

---

## Secondary rows

### D12. Ghost participation

PRD §5.7: ghosts cast a "ghost vote." GameFlowSpec §7: ghosts cannot vote or participate.

Letting eliminated players influence a match they have lost is a balance problem; but the PRD's
*intent* — keep dead players engaged during downtime — is legitimate.

**Recommendation: split the difference.** Ghosts cast a **non-binding prediction** visible only
to other ghosts. Full engagement, zero effect on the outcome. Ghosts keep the physics-nudging
from PRD §5.7 either way.

**Decision: ✅ DECIDED — ghosts cannot vote.** Normal eliminated players become ghosts who watch
but do not vote or give clues. The PRD's "ghost vote" is dropped.

**New role added by this decision — "Spirit"** (not in the PRD or GameFlowSpec):

- **Optional, host setting**, like Mr. Blank and the Jester. Proposed minimum: 5+ players.
- One random **Civilian** secretly gets the Spirit role. It does not change the D2 tables — the
  Spirit is still one of the Civilians.
- The player **knows they are the Spirit from the start** (other players do not).
- If voted out, the Spirit becomes a ghost who **can still speak and vote**, but **no longer gives
  clues**.
- After being voted out, the Spirit **does not count** as in the match for the D5 win check. Their
  vote still counts.
- The Spirit is a Civilian and wins with the Civilians.

---

### D13. Seating

PRD §3.1: players **claim a seat** in the lobby. GameFlowSpec §5: seats **randomized** each
match, deliberately different from lineup order.

**Recommendation: randomized.** Claimed seats let friends sit together and coordinate, and
GameFlowSpec §8's rematch flow already assumes re-randomization.

**Decision: ✅ DECIDED — random seats every match.**

---

### D14. Turn order

PRD §5.1: "turn order is fixed per match." GameFlowSpec §6: clockwise, but the **starting player
rotates** each round.

Easily reconciled — order fixed, start point rotates.

**Recommendation: GameFlowSpec's rotating start.** Going first is the hardest job (least
information); rotating it spreads that burden.

**Decision: ✅ DECIDED — rotating start.** Turns go clockwise, and the first player moves one seat
each round.

---

### D15. Disconnect handling

PRD §7: client can **rejoin their seat within a grace window**; host migration as a stretch goal.
GameFlowSpec: **unaddressed**.

**Decision: ✅ DECIDED — "sleeping crab."**

- When a player disconnects, their crab **falls asleep** at the table. The match continues.
- The remaining players **may vote out the sleeping crab** if they want to, like any other player.
- If the player **reconnects before being voted out**, they wake up in the same seat, with the
  same role and word, and carry on playing.
- There is **no timer** — a sleeping crab stays in the match until it reconnects or is voted out.

**Host disconnect is not covered by this.** The host *is* the server on a listen server, so if
the host drops, the match ends for everyone. Ship a clean "host left" screen with a fast
re-lobby. Host migration stays a stretch goal.

**Sleeping-crab rules** — all decided:

| # | Question | Decision |
|---|---|---|
| a | During the clue phase, is the sleeping crab's turn skipped? | **Yes** — skipped instantly |
| b | Does a sleeping crab get a vote? | **No** — counts as not voting (not as Skip) |
| c | If a sleeping crab is the most-voted, does it get a defense moment (D8)? | **No** — straight to elimination |
| d | Does a sleeping crab count as "alive" for the parity win check (D5)? | **Yes** |
| e | If a **sleeping Jester** is voted out, does the Jester win? | **Yes** — the Jester wins as normal. Accepted knowingly: a Jester may disconnect on purpose to bait a vote. |
| f | How does a reconnecting player see their word? | **The word card is viewable at any time**, by every player, for the whole match (matches GameFlowSpec §5 "viewable on demand"). A reconnecting player simply opens it. |

---

### D16. Does the Jester count toward the validation rule?

Undefined in both documents. The rule is `Blank + Undercover < Civilians`; the Jester is neither
side and wins independently.

**Recommendation: Jester counts as neither.** It occupies a seat that would otherwise be a
civilian, and is excluded from both sides of the inequality.

**Decision: ✅ DECIDED — it depends on which check.**

- **Table check** (setting up roles, D2): the Jester is **left out**.
- **Win check** (after each elimination, D5): the Jester **counts with everyone else** against the
  Undercover, but is **neutral** and never shares a Civilian or Undercover win.
- If the Jester is voted out, the **Jester wins** and the match ends immediately. This is checked
  **before** the win check above.

---

### D17. Undercover cap at high player counts

PRD §4 caps Undercover at 2. At 9-10 players that is 2 against 7-8 — thin, and the table can
brute-force it.

**Recommendation: allow 3 Undercover at 9+ players.** Requires amending the PRD's stated range.

**Decision: ✅ DECIDED — 3 Undercover at 9-10 players.** This changes PRD §4, which says 1-2.
---

### D18. Word pack schema — decide before any pack ships

PRD §5.6 adds host-tunable difficulty (word-pair similarity). `CLAUDE.md` documents the pack
format as two columns: `civilian_word,undercover_word`. **There is nowhere for difficulty to
live.**

This must be settled *now*. Adding a third column later breaks every existing pack — including
community Workshop packs, which you cannot migrate.

**Recommendation: adopt `civilian_word,undercover_word,difficulty` now**, even if difficulty is
ignored by the first implementation. A column you don't read yet is free; a migration is not.

**Decision: ⏸ POSTPONED.** Not needed to start Phase 2 — the prototype can use a small built-in
list of word pairs. Must be decided **before the first word pack is shared or imported**.

---

## Rows where the PRD conflicts with what is already built

These are not design decisions — they are documentation defects. Fix the PRD.

| # | PRD says | Reality |
|---|---|---|
| A | §7: EOS for NAT punch-through, lobbies, Voice **and Data** | Steam OSS + SteamSockets, verified in `DefaultEngine.ini` and confirmed in `Saved/Logs/`. Correct architecture is **Steam for sessions, EOS for voice only** (`PROJECT_CONTEXT.md` §5). |
| B | Phase 3: "EOS Voice **or Vivox**" | Vivox is now a commercial Unity product; Epic documents migrating away from it. EOS only. |
| C | §5.3: VAD drives "mouth/mandibles move" | Crabs have no jaw. Viseme/lip-sync does not apply. Expression channels that work: **eyestalks, claws, body bob** (`PROJECT_CONTEXT.md` §4.2). |
| D | §5.6 Workshop, §6 progression/XP | Both require Steam Cloud / Inventory / Workshop, **none of which exist on AppID 480**. Blocked until Steam Direct completes. |

---

## Once every row is filled

1. Merge PRD v2.0 and `GameFlowSpec.md` into one document. Delete the loser — do not keep both.
2. Write the resolved role composition table (D2) into the merged spec as a literal table, not
   a rule to be re-derived.
3. Specify the **win-condition check** explicitly: it runs after every elimination, and the
   Jester short-circuit (§7) is evaluated **before** the parity check.
4. Update `CLAUDE.md` to point at the merged spec and drop the "min-player count is disputed"
   note once D3 is settled.
5. Fix the four documentation defects in the table above.
