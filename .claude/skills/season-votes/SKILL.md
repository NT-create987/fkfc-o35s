# Season Votes Feature

## When to Use
Apply this skill when working on anything related to:
- Season vote tallies or leaderboard
- `seasonVotes` Firebase path
- The 🗳️ Votes Admin tab
- Vote history, per-match vote breakdowns, or vote editing

## Overview
Tracks cumulative weighted POTM votes (3pts/2pts/1pt) per player across the whole season.
Admin-only. Retroactive — edits recalculate totals via diff.

## Firebase Data Model
```
seasonVotes/
├── totals/
│   ├── "Nathan": 47        ← cumulative weighted pts, all matches
│   └── "Scott": 31
└── matches/
    └── "vs_DY___Tuesday___28_04_26"/   ← sanitised match key
        ├── label: "vs DY – Tuesday – 28/04/26"
        └── votes/
            ├── "Nathan": { first: 2, second: 1, third: 0, pts: 8 }
            └── "Scott":  { first: 0, second: 2, third: 1, pts: 5 }
```
Points: 1st = 3pts · 2nd = 2pts · 3rd = 1pt

## Key Functions to Add
- `tallyVotesForMatch(votesSnap, matchName)` — tally raw votes into per-player `{ first, second, third, pts }`
- `writeSeasonVotesForMatch(matchKey, label, tally)` — write on archive
- `diffAndUpdateSeasonVotes(matchKey, label, oldTally, newTally)` — diff on edit
- `removeSeasonVotesForMatch(matchKey)` — reverse on archive delete
- `renderSeasonVotesAdmin()` — renders leaderboard + per-match list
- `openSVEditor(matchKey)` — toggle inline edit panel for a match
- `recalcSVRow(input)` — live pts recalc as admin edits vote counts
- `saveSVEdits(matchKey, label)` — save edited tally, diff totals, re-render

## Hooks into Existing Functions
- `resetAndArchive()` — call `writeSeasonVotesForMatch()` after archiving
- `saveResultChanges()` — call `diffAndUpdateSeasonVotes()` after editing
- `deleteArchiveEntry()` — call `removeSeasonVotesForMatch()` before deleting

## UI Location
Admin → 🗳️ Votes Admin (5th sub-tab, after Results Admin)
- Top: season leaderboard with bar chart (ranked by total pts)
- Below: per-match cards showing 🥇🥈🥉 counts and pts per player
- ✏️ Edit button on each card opens inline table editor with number inputs
- Points recalculate live as inputs change
- 💾 Save Changes diffs and updates Firebase, re-renders leaderboard

## Styling
Match existing admin tab aesthetic — dark cards, green accents, gold headings.
Use same colour variables already in the file.

## Backfill (one-time)
After implementing, run this in the browser console to populate past matches:
- Read all `votes` node entries
- Group by `matchName`
- Tally per player and write to `seasonVotes/matches/` and `seasonVotes/totals/`
