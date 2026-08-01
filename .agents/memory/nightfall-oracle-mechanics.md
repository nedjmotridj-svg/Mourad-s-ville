---
name: Nightfall Oracle — mechanics decisions
description: Key design decisions for gameplay upgrades (Loup Bavard modal, GM voting, undo stack, Solo Loup Noir).
---

## Bavard pre-vote modal (Phase 1)
- The Loup Bavard word-check was **removed from VotePanel** and moved to **DawnPanel** as a pre-vote modal.
- Night 1: no check (bavard was inactive). Day 2+: modal "A-t-il prononcé son mot?" appears when GM taps the vote button.
  - Oui → `goToVote(state)` → VotePanel as normal.
  - Non → `executeTalkativeWolfAndSkip(state)` → kills bavard, logs it, skips to next night.
- `executeTalkativeWolfAndSkip` is exported from `engine.ts`. It clones state, calls `killPlayer`, pushes a log entry, then calls `checkVictory → startNight`.

## Solo Loup Noir — combined step (Phase 2, updated)
- When Loup Noir is the sole active werewolf, `buildNightSteps` pushes a **single combined step** with `roleId: "loup-noir"`, `mode: "blackwolf"`, `soloKill: true`.
- `Step` interface has `soloKill?: boolean` flag to distinguish combined vs normal blackwolf step.
- NightPanel `blackwolf` mode checks `step.soloKill`: if true, shows a victim picker (`sel` state) at top; contaminate checkbox unlocks once a victim is selected; silence picker below; button sends `{ targetId: sel[0], yes: infect, muteId }`.
- `submitStep` `loup-noir` case: if `payload.targetId && !s.round.attackedId`, sets `s.round.attackedId = payload.targetId` (solo kill) before contaminate/silence logic.
- New locale keys: `noirSoloVictimTitle` ("Désigner la victime") and `noirSoloConfirm` ("Loup Noir frappe") in FR/EN/AR.
- **Why:** merging kill + special abilities into one step is cleaner UX than two sequential steps for solo Loup Noir.

## GM-driven voting (Phase 2)
- VotePanel tally (+/−) is now **display-only** — no auto-compute of winner.
- GM manually picks eliminees via `PlayerPicker` (immune players excluded).
- `confirmElim()` drives the outcome: 1 pick → `submitVote`; 2+ picks + Judge → judge panel; 2+ + no judge + revoteRound=0 → revote; revoteRound=1 → `eliminateTied`.
- Tally logged to `state.log` as `"Jour N — Vote: [A×2, B×1] → Éliminé(s): X"`.
- `talkativeSpoke` is always `true` in VotePanel calls (bavard fate decided before this screen).

## Undo / history stack (Phase 2)
- `stateHistory: GameState[]` in `GamePage`, max 30 snapshots via `updateState(next)`.
- `undo()` pops history and restores state.
- `canUndo` / `onUndo` props threaded to NightPanel, DawnPanel, VotePanel.
- Each panel shows a `RotateCcw` "← Annuler" button when `canUndo`.

## i18n
- New keys added to all 3 locales (fr/en/ar): `bavardPreVoteTitle`, `bavardPreVoteAsk`, `bavardPreVoteYes`, `bavardPreVoteNo`, `gmSelectElim`, `gmSelectElimHint`, `gmConfirmElim`, `gmRevoteAction`, `undoStep`.
- AR uses RTL-appropriate arrow for undo: `"تراجع →"`.
- `Dictionary` type is derived from `fr.ui`, so adding keys to fr.ts automatically enforces them in en/ar at compile time.
