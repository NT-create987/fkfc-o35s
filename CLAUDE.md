# FKFC O35s — Project Memory

## Project Overview
Single-page web app for an over-35s soccer team. One file: `index.html`.
Firebase Realtime Database (v9.23.0 compat) for all state — no localStorage anywhere.

## Deployment
- **Repo:** GitHub → `NT-create987/fkfc-o35s`
- **Hosting:** Vercel — auto-deploys on every push to main branch
- **Live site:** `https://fkfc-o35s.vercel.app`
- **Workflow:** Edit `index.html` → push to GitHub → Vercel deploys automatically
- Always push to main unless told otherwise

## Tech Stack
- **Frontend:** Vanilla HTML/CSS/JS — no build tools, no frameworks
- **Database:** Firebase Realtime Database (v9.23.0 compat)
- **Fonts:** Bebas Neue (headings), DM Sans (body) via Google Fonts
- **Single file:** Everything lives in `index.html`

## Firebase Config
- Project: `fkfc-o35s`
- Database URL: `https://fkfc-o35s-default-rtdb.asia-southeast1.firebasedatabase.app`
- API Key: `AIzaSyCnPIRl-t9yuZLUT7bOZSPb6Z85L87xbmE`
- Use v9.23.0 compat SDK only — do NOT use v10+ or ES module imports

## Script Architecture (order matters)
1. **Tab + wrapper script** (before Firebase) — defines `showTab`, `showAdminTab`, delegation wrappers for all `onclick` functions
2. **Firebase compat CDN scripts** — `firebase-app-compat.js` + `firebase-database-compat.js`
3. **App logic script** (after Firebase) — all init, state, listeners, render functions

## Critical Rules
- All `onclick`-callable functions use two-layer delegation: `window.fn` → `window._impl_fn`
- This prevents silent failures on slow mobile connections — never skip this pattern
- Firebase key safety: sanitise with `s.replace(/[^a-zA-Z0-9_\-]/g, '_')` before using as path keys
- `state.roster` can arrive as JS array or plain object — always handle both:
  `Array.isArray(v) ? v : Object.values(v)`
- Never use localStorage — Firebase only
- Never add a build step, bundler, or second file — everything stays in `index.html`

## App Structure
```
FKFC O35s
├── ⚽ Vote          (public — loads first)
├── 📋 Availability  (public)
├── 📊 Results       (public)
│   ├── Match Results
│   └── Player Stats
└── 🏆 Admin         (PIN: 1234 — gold tab)
    ├── ⚽ Voting Admin
    ├── 🗳️ Votes Admin         ← BUILT: season leaderboard + per-match editor
    ├── 📋 Availability Admin  ← includes Formation Builder
    ├── 💸 Fines Admin
    └── ✏️ Results Admin
```
Tab order matters — Voting and Votes sit next to each other by design.

## Key Firebase Paths
| Path | Description |
|------|-------------|
| `match` | Current match: name, opponent, matchDate, squad[], open |
| `votes` | All votes keyed by `safe(voter) + '_' + safe(matchName)` |
| `seasonPts` | Cumulative season points per player (from POTM voting) |
| `seasonVotes/totals` | `{ playerName: weightedPts }` — cumulative across all matches |
| `seasonVotes/matches/{key}` | `{ label, squad[], votes: { name: { first, second, third, pts } } }` |
| `archive` | Past matches with scorelines, goals, stats |
| `matchFines` | Current match fines |
| `seasonFines` | Cumulative fines per player |
| `availability` | `{ playerName: 'available'│'injured'│'away'│'pending' }` |
| `nextGame` | `{ dateRaw, timeRaw, date, time, location, maps }` |
| `lineup` | Formation builder output: `{ formation, positions[], bench[], posted }` |
| `finesClosedMatch` | Match name for which voting fines have been issued |
| `availFinesDone` | Whether availability fines issued this cycle |
| `matchResult` | `{ fkfc, opp, result, goals[] }` |
| `playerStats` | `{ playerName: { appearances, goals, assists } }` |

## Voting Workflow
1. Open Voting → 2. (Optional) Issue Voting Fines → 3. Close Voting → 4. Reset & Archive
- Close for Fines is OPTIONAL — Close Voting must always be available without it
- On archive: `writeSeasonVotesForMatch()` is called to tally and store POTM votes

## Season Votes Feature (BUILT — see SKILL.md)
Tracks cumulative weighted POTM votes (1st=3, 2nd=2, 3rd=1) per player across the season.

### Key functions
- `renderSeasonVotesAdmin()` — renders leaderboard + per-match card list
- `renderSeasonStandings(elId)` — renders the bar-chart leaderboard; has `if (!el) return;` guard
- `renderSVMatchList()` — renders per-match breakdown cards; filters 0-pt players from display
- `writeSeasonVotesForMatch(matchKey, label, tally)` — writes on archive
- `diffAndUpdateSeasonVotes(matchKey, label, oldTally, newTally)` — diffs on edit
- `removeSeasonVotesForMatch(matchKey)` — reverses totals on archive delete
- `openSVEditor(matchKey)` — toggles inline edit panel
- `recalcSVRow(input)` — live pts recalc inside editor
- `saveSVEdits(matchKey, matchLabel)` — saves edited tally, diffs totals, re-renders

### Display terminology
- Per-match card column: "VOTES" (not "PTS") — consistent with season leaderboard

## Critical JS Gotchas Learned

### 1. onfocus in JS string literals — NEVER use quoted string comparisons
BAD (breaks single-quoted JS string):
```js
'<input onfocus="if(this.value===\'0\')this.value=\'\'">'
```
SAFE (no nested quotes):
```js
'<input onfocus="this.select()">'
```
`this.select()` achieves the same UX (selecting all text so typing replaces it).

### 2. renderSeasonStandings guard
Always check element exists — the element was removed from the Voting tab but the function is still called:
```js
function renderSeasonStandings(elId) {
  const el = document.getElementById(elId);
  if (!el) return;   // ← required
  ...
}
```

### 3. Zero-pt players in SV editor
- 0-pt players are stored in `seasonVotes/matches/{key}/votes` when saved
- BUT: if all three counts are 0, the player is **omitted entirely** from `votes` in Firebase
- They are shown in the editor below the LATE / NO VOTES divider (read from `squad[]`)
- They are **filtered out** of the display card (only shown if `pts > 0`)

### 4. Stale test data
When wiping test games, also check:
- `seasonVotes/matches/` for stray entries
- `seasonVotes/totals/` for stray player scores
- `playerStats/` for inflated goals/assists
- `archive/` for stray entries
Clean with `db.ref('path').remove()` and manually reverse the totals diff.

## Design Tokens
- Green: `#1e7a35` (FKFC brand)
- Navy: `#1a2a4a` (pitch background)
- Gold: `#c9a227` (admin tab)
- GK jersey: `#e8826a` (salmon/pink)
- Bench bib: `#f5c518` (yellow)

## Pending Features
- **Optional Fines** — see `.claude/skills/optional-fines/SKILL.md`
