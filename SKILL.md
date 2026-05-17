# Season Votes Feature — COMPLETE

## Status
**Fully built and tested.** This skill documents the implementation as it exists in `index.html`.

## When to Use
Apply this skill when working on anything related to:
- Season vote tallies or leaderboard
- `seasonVotes` Firebase path
- The 🗳️ Votes Admin tab
- Vote history, per-match vote breakdowns, or vote editing

## Overview
Tracks cumulative weighted POTM votes (3pts/2pts/1pt) per player across the whole season.
Admin-only. Retroactive — edits recalculate totals via diff, not full recompute.

## Firebase Data Model
```
seasonVotes/
├── totals/
│   ├── "Nathan": 128       ← cumulative weighted votes across all matches
│   └── "Scott": 88
└── matches/
    └── "vs_Tigers___Monday___25_05_26"/   ← safe(matchName) key
        ├── label: "vs Tigers – Monday – 25/05/26"
        ├── squad: ["Nathan", "Scott", "Agus", ...]   ← players in that match
        └── votes/
            ├── "Mark":   { first: 3, second: 5, third: 3, pts: 22 }
            └── "Clint":  { first: 4, second: 2, third: 2, pts: 18 }
            ← players with 0 pts are OMITTED from votes{} entirely
```
Key formula: `pts = first×3 + second×2 + third×1`

Key function: `safe = s => s.replace(/[^a-zA-Z0-9_\-]/g, '_')`
Match key format: `safe(matchName)` e.g. `vs_Tigers___Monday___25_05_26`

## Implemented Functions

### Write / archive hooks
- `writeSeasonVotesForMatch(matchKey, label, tally)` — called from `resetAndArchive()` on archive
- `diffAndUpdateSeasonVotes(matchKey, label, oldTally, newTally)` — called from `saveResultChanges()` on edit
- `removeSeasonVotesForMatch(matchKey)` — called from `deleteArchiveEntry()` on delete; reverses totals

### Render
- `renderSeasonVotesAdmin()` — entry point; renders leaderboard + 3,2,1 table + per-match cards
- `renderSeasonStandings(elId)` — bar-chart leaderboard sorted by total votes; **requires `if (!el) return;` guard at top**
- `renderSVMatchList()` — per-match breakdown cards; filters `pts > 0` before display

### Editor
- `openSVEditor(matchKey)` — toggles inline edit table for a match card
- `recalcSVRow(input)` — live pts recalc as admin types; updates `.sv-row-pts-cell`
- `saveSVEdits(matchKey, matchLabel)` — reads editor inputs, diffs vs old tally, updates Firebase totals, re-renders

## UI — Votes Admin Tab (tab order: Voting → Votes → Avail → Fines → Results)
- **🏆 Season Votes** — bar chart leaderboard (cumulative, ranked by total votes)
- **📊 3, 2, 1 Match Votes** — table: how many times each player placed 1st/2nd/3rd overall per match
- **📋 Per-Match Breakdown** — one card per archived match, sorted chronologically
  - Each card shows: match label, voter count, squad size, total votes
  - Players with `pts > 0` shown with 🥇🥈🥉 chips and vote count
  - 0-pt players NOT shown in card (stored in Firebase under `squad[]`, shown in editor)
  - ✏️ Edit button opens inline editor
- **🗑 Clear Season Data** — danger button at bottom; wipes `seasonVotes/` entirely

## Editor Behaviour
- Players with votes appear above the "LATE / NO VOTES" divider
- All squad members appear below the divider with 0/0/0 inputs
- Clicking into a 0 field selects all text — typing immediately replaces it
  - Implemented as `onfocus="this.select()"` — **never use `onfocus="if(this.value==='0')..."` inside a JS string literal** (single-quote collision breaks the script)
- Points recalculate live as inputs change
- 💾 Save Changes: computes diff vs previous tally, applies delta to `totals/`, stores new tally
- Players left at 0/0/0 are removed from `votes{}` in Firebase (not stored as zero records)
- "Add a player" dropdown: shows players not already in the editor (for off-roster visitors)

## Display Terminology
- Column header and chip both say **VOTES** (not "pts") — matches the season leaderboard label
- Editor "Total" column says "pts" — acceptable since it's inside the editing context

## Known Issues / History
- **Stray test entries**: Always clean up test archives from both `archive/` and `seasonVotes/matches/`
  and reverse any `seasonVotes/totals/` deltas. Use `db.ref('path').remove()` in console.
- **playerStats inflation**: After test games, verify `playerStats/` values are correct.
  Safe reset: `firebase.database().ref('playerStats').set({ Nathan: {...}, ... })`
- **renderSeasonStandings called with removed element**: The season leaderboard was removed from
  the Voting tab but the function is still called — the `if (!el) return;` guard prevents errors.
