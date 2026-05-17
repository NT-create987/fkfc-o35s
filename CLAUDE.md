# FKFC O35s — Project Memory

## Project Overview
Single-page web app for an over-35s soccer team. One file: `index_42.html`.
Firebase Realtime Database (v9.23.0 compat) for all state — no localStorage anywhere.

## Deployment
- **Repo:** GitHub (auto-push on changes)
- **Hosting:** Vercel — auto-deploys on every push to main branch
- **Workflow:** Edit `index_42.html` → push to GitHub → Vercel deploys automatically
- Always push to main unless told otherwise

## Tech Stack
- **Frontend:** Vanilla HTML/CSS/JS — no build tools, no frameworks
- **Database:** Firebase Realtime Database (v9.23.0 compat)
- **Fonts:** Bebas Neue (headings), DM Sans (body) via Google Fonts
- **Single file:** Everything lives in `index_42.html`

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
- Never add a build step, bundler, or second file — everything stays in `index_42.html`

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
    ├── 📋 Availability Admin  ← includes Formation Builder
    ├── 💸 Fines Admin
    ├── ✏️ Results Admin
    └── 🗳️ Votes Admin         ← season vote leaderboard (to be built)
```

## Key Firebase Paths
| Path | Description |
|------|-------------|
| `match` | Current match: name, opponent, matchDate, squad[], open |
| `votes` | All votes keyed by `voter_matchName` (sanitised) |
| `seasonPts` | Cumulative season points per player |
| `seasonVotes` | Season vote tallies — totals + per-match breakdown |
| `archive` | Past matches with scorelines, goals, stats |
| `matchFines` | Current match fines |
| `seasonFines` | Cumulative fines per player |
| `availability` | `{ playerName: 'available'│'injured'│'away'│'pending' }` |
| `nextGame` | `{ dateRaw, timeRaw, date, time, location, maps }` |
| `lineup` | Formation builder output: `{ formation, positions[], bench[], posted }` |
| `finesClosedMatch` | Match name for which voting fines have been issued |
| `availFinesDone` | Whether availability fines issued this cycle |
| `matchResult` | `{ fkfc, opp, result, goals[] }` |
| `playerStats` | `{ playerName: { played, goals, assists } }` |

## Voting Workflow
1. Open Voting → 2. (Optional) Issue Voting Fines → 3. Close Voting → 4. Reset & Archive
- Close for Fines is OPTIONAL — Close Voting must always be available without it

## Pending Features
- **Season Votes** — see `.claude/skills/season-votes/SKILL.md`
- **Optional Fines** — see `.claude/skills/optional-fines/SKILL.md`

## Design Tokens
- Green: `#1e7a35` (FKFC brand)
- Navy: `#1a2a4a` (pitch background)
- Gold: `#c9a227` (admin tab)
- GK jersey: `#e8826a` (salmon/pink)
- Bench bib: `#f5c518` (yellow)
