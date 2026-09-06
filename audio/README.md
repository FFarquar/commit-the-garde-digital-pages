Drop the real sound effect files here, named exactly as below (referenced by [src/audio/soundManager.ts](../../src/audio/soundManager.ts)):

- `bugle.mp3` — plays when a modal/popup prompts or informs the player (turn end, reaction window, your-turn notice, order/redeploy/march-facing popovers), and on granting a React marker
- `musketry.mp3` — plays on Fire and Skirmish Fire resolution (including Artillery Reaction Fire)
- `march.mp3` — plays on March, Advance/Retire, Skirmish deploy, and Skirmisher recall
- `melee.mp3` — plays when an Assault actually resolves into close combat (including a Cavalry Opportunity Assault reaction)
- `rally.mp3` — plays on a Rally attempt
- `rout.mp3` — plays on Sauve qui Peut
- `formation.mp3` — plays on Redeploy, Reorder, Support Assault marker placement, and an Infantry Opportunity Square reaction
- `activation.mp3` — plays when a unit is activated (rolls the Unit Actions Table)

A leader Falling plays its own `SoundId` (`leaderFallen`) but currently points at `bugle.mp3` too (human-requested 2026-08-24: reuse the existing cue for now). Drop a `leaderFallen.mp3` here and repoint `SOUND_FILES.leaderFallen` in soundManager.ts to give it a real, distinct sound later.

The five action sounds above (`melee`, `rally`, `rout`, `formation`, `activation`) are wired to callers throughout src/main.ts (local hotseat) and server/handlers/ws/game-action.ts's `SOUND_HINT_BY_KIND` (network games). `melee`, `rally`, `rout`, and `activation` now have real files; `formation.mp3` doesn't exist yet.

Until a file exists, `playSound()` fails silently (caught) — no error, just no sound.

Not wired to a sound at all yet: the leader-phase actions (Move/Stay, Direct Order, Replace Leader) and `transferControl` — these are procedural/meta steps rather than battlefield events, so they were left silent rather than forced into one of the buckets above.

**Melee in network games** (human-reported 2026-09-06, fixed): `SOUND_HINT_BY_KIND` maps by action `kind` alone, which can't tell whether an `assault`/`pickAssaultSkirmisherChoice`/`declineReaction`/`attemptReaction`/`rollDice` message actually just resolved real close combat, opened a reaction window with no combat yet, or resolved as something else entirely (a square formation, musketry, a no-op occupation). Melee now rides a separate signal instead: `server/lib/game.ts`'s `ResolutionPatch.soundHint`, set only at the one place real combat is known to have happened (`finishAssaultCombat`/`finishArmyHqAssaultCombat` — the same moment main.ts's own `computeContinueAssault`/`computeResolveArmyHqAssault` call `playSound("melee")` locally), and bubbled up through every path that can reach it (a fresh Assault, a decline-resume, a Cavalry Opportunity Assault reaction, manual dice mode's `rollDice`, and the 5-second auto-roll timeout). `game-action.ts`/`roll-timeout.ts` pull this off the patch into a closure variable before persisting the rest of the patch (it is NOT a `StoredGame` field) and merge it into the broadcast, preferring it over `SOUND_HINT_BY_KIND[kind]` when present.

Square-formation and musketry reaction cues are still unmapped in network games for the same `kind`-alone reason — `attemptReaction`'s `kind` doesn't say which of the three reaction kinds actually happened, and only melee has had this patch-level signal added so far (local hotseat still plays every cue right regardless, since `computeAttemptReaction` calls `playSound` directly).
