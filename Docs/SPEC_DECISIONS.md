# CrabCover — Spec Decision Sheet

**Purpose:** resolve every contradiction between `PRD v2.0` and `Docs/GameFlowSpec.md` so the
GameMode can be implemented against a single source of truth.

**How to use:** work through this with both developers in one sitting. Fill in the
`**Decision:**` line for each row. When every row is filled, the two source documents get
merged into one spec and the loser is deleted — do not leave three documents alive.

**Status:** unresolved. No decision below has been made.

> **Version warning.** This sheet was written against **PRD v2.0**. `PROJECT_CONTEXT.md` §3
> reviews "PRD v3.1". If v3.1 exists, re-check every row before deciding. Six of §3's seven
> rows match v2.0 exactly; one (disconnect handling) does not — see D15.

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

**Decision:** _______________

---

### D2. Role composition table

**Neither document contains one.** The PRD gives ranges (Undercover 1-2, Mr. Blank 0-1,
Jester 0-1); GameFlowSpec gives a validation rule and says counts are "auto-suggested" but
never says what the suggestion is. Role assignment is the first function in the GameMode and
it currently has no specification.

Validation rule (GameFlowSpec §2): `Mr. Blank + Undercover < Civilians`

**Proposed default table** — Mr. Blank off, Jester off:

| Players | Civilians | Undercover | Check |
|---:|---:|---:|---|
| 4 | 3 | 1 | 1 < 3 ✓ |
| 5 | 4 | 1 | 1 < 4 ✓ |
| 6 | 5 | 1 | 1 < 5 ✓ |
| 7 | 5 | 2 | 2 < 5 ✓ |
| 8 | 6 | 2 | 2 < 6 ✓ |
| 9 | 7 | 2 | 2 < 7 ✓ |
| 10 | 8 | 2 | 2 < 8 ✓ |

**With Mr. Blank enabled** (host opt-in; disabled below 6 players — both documents agree Blank
is unplayable at low counts):

| Players | Civilians | Undercover | Blank | Check |
|---:|---:|---:|---:|---|
| 6 | 4 | 1 | 1 | 2 < 4 ✓ |
| 7 | 5 | 1 | 1 | 2 < 5 ✓ |
| 8 | 5 | 2 | 1 | 3 < 5 ✓ |
| 9 | 6 | 2 | 1 | 3 < 6 ✓ |
| 10 | 7 | 2 | 1 | 3 < 7 ✓ |

**With Jester enabled:** Jester takes one seat; reduce Civilians by 1. See D16 for whether the
Jester counts toward the validation rule.

**Decision:** _______________

---

## Core loop rows

### D3. Minimum players

PRD §3.1 and `CLAUDE.md`: **4**. GameFlowSpec §4: **3**.

At 3 players the composition is 2 civilians + 1 undercover, and a single wrong vote ends it —
there is no deduction, only a coin flip.

**Recommendation: 4.** Matches the PRD, `CLAUDE.md`, and the stated owner intent. Update
GameFlowSpec §4 and grey out Start Match below 4.

**Decision:** _______________

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

**Decision:** _______________

---

### D5. Undercover win condition

PRD §3.6: undercover **survives to the final two**. GameFlowSpec §7: **parity** — undercover +
Blank combined equals or outnumbers remaining civilians.

The PRD's version breaks with 2 undercovers, which the PRD itself allows in §4: "final two" is
undefined when both survivors are undercover.

**Recommendation: parity (GameFlowSpec).** High confidence. It is the standard rule, it scales
to any undercover count, and it handles Mr. Blank cleanly. The PRD's phrasing only works in the
single-undercover case.

**Decision:** _______________

---

### D6. Skip / Pass vote

GameFlowSpec §7 offers it. PRD §5.2 does not.

Without Skip, every round forces an elimination — at 4 players the match is over in two rounds.
Skip is the pressure valve that lets a table say "we don't know yet."

**Recommendation: keep Skip, with a stall guard.** Cap consecutive skips (suggest 2) or cap
total rounds, then force an elimination vote. Unbounded skipping is how a match hangs forever.

**Decision:** _______________ &nbsp;&nbsp; Stall guard: _______________

---

### D7. Tie resolution

PRD §5.2: defense for each tied player → single revote among tied → **if still tied, no
elimination, continue**. GameFlowSpec §7: runoff "repeats until a single player has the most
votes" — which can loop forever with a deadlocked table.

**Recommendation: PRD's version.** High confidence. The PRD is simply the better-engineered
document on this row; GameFlowSpec's has no termination guarantee.

**Decision:** _______________

---

### D8. Defense moment

PRD §3.5: most-voted player gets a timed plea (default 10s) before elimination is finalized.
GameFlowSpec: **absent**.

**Recommendation: adopt.** Cheap to build (one timed state in the phase machine) and it directly
serves the PRD's stated value proposition — this is the clip-worthy moment.

**Decision:** _______________ &nbsp;&nbsp; Length: ______ s

---

### D9. Turn timer

PRD §5.1: ~30s soft timer per turn, plus AFK skip (§5.9). GameFlowSpec: **absent**.

Without a clock, one silent or AFK player stalls the table indefinitely. This is a solved
problem in the PRD that GameFlowSpec reopened.

**Recommendation: adopt the PRD's timer and AFK handling.** High confidence — this is not
optional for a party game.

**Decision:** _______________ &nbsp;&nbsp; Length: ______ s

---

### D10. Mr. Blank guess mechanic

PRD §4: Blank survives the vote and **guesses the word at the end**. GameFlowSpec §5: no guess —
Blank only bluffs.

Without the guess, Mr. Blank is pure disadvantage with no upside, and the role stops being
playable — it becomes merely hard.

**Recommendation: adopt the PRD's guess.** High confidence. Define *when* the guess happens
(on elimination? at match end? on reaching final two?) — the PRD says "at the end" without
specifying which.

**Decision:** _______________ &nbsp;&nbsp; Guess timing: _______________

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

**Decision:** _______________

---

## Secondary rows

### D12. Ghost participation

PRD §5.7: ghosts cast a "ghost vote." GameFlowSpec §7: ghosts cannot vote or participate.

Letting eliminated players influence a match they have lost is a balance problem; but the PRD's
*intent* — keep dead players engaged during downtime — is legitimate.

**Recommendation: split the difference.** Ghosts cast a **non-binding prediction** visible only
to other ghosts. Full engagement, zero effect on the outcome. Ghosts keep the physics-nudging
from PRD §5.7 either way.

**Decision:** _______________

---

### D13. Seating

PRD §3.1: players **claim a seat** in the lobby. GameFlowSpec §5: seats **randomized** each
match, deliberately different from lineup order.

**Recommendation: randomized.** Claimed seats let friends sit together and coordinate, and
GameFlowSpec §8's rematch flow already assumes re-randomization.

**Decision:** _______________

---

### D14. Turn order

PRD §5.1: "turn order is fixed per match." GameFlowSpec §6: clockwise, but the **starting player
rotates** each round.

Easily reconciled — order fixed, start point rotates.

**Recommendation: GameFlowSpec's rotating start.** Going first is the hardest job (least
information); rotating it spreads that burden.

**Decision:** _______________

---

### D15. Disconnect handling

PRD §7: client can **rejoin their seat within a grace window**; host migration as a stretch goal.
GameFlowSpec: **unaddressed**.

> **Correction to `PROJECT_CONTEXT.md` §3.** That table lists the PRD's position as "instant
> elimination." PRD v2.0 §7 says the opposite. The handoff doc's row is wrong and should be fixed.

**Recommendation: adopt rejoin-into-seat.** Treat **host migration as a genuine stretch goal**
— it is hard on a UE5 listen server and has sunk party games before. Ship the clean "host left"
screen with a fast re-lobby as the real behavior; migrate only if a spike proves it viable.

**Decision:** _______________ &nbsp;&nbsp; Grace window: ______ s

---

### D16. Does the Jester count toward the validation rule?

Undefined in both documents. The rule is `Blank + Undercover < Civilians`; the Jester is neither
side and wins independently.

**Recommendation: Jester counts as neither.** It occupies a seat that would otherwise be a
civilian, and is excluded from both sides of the inequality.

**Decision:** _______________

---

### D17. Undercover cap at high player counts

PRD §4 caps Undercover at 2. At 9-10 players that is 2 against 7-8 — thin, and the table can
brute-force it.

**Recommendation: allow 3 Undercover at 9+ players.** Requires amending the PRD's stated range.

**Decision:** _______________

---

### D18. Word pack schema — decide before any pack ships

PRD §5.6 adds host-tunable difficulty (word-pair similarity). `CLAUDE.md` documents the pack
format as two columns: `civilian_word,undercover_word`. **There is nowhere for difficulty to
live.**

This must be settled *now*. Adding a third column later breaks every existing pack — including
community Workshop packs, which you cannot migrate.

**Recommendation: adopt `civilian_word,undercover_word,difficulty` now**, even if difficulty is
ignored by the first implementation. A column you don't read yet is free; a migration is not.

**Decision:** _______________

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
