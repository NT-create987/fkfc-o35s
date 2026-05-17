# Optional Fines Feature

## Status
**Pending — not yet built.**

## When to Use
Apply this skill when working on anything related to:
- Optional or configurable fines
- The 💸 Fines Admin tab

## Overview
Allow the admin to configure which fines are active each week (e.g. turn off availability fines for a bye round, adjust fine amounts, add custom one-off fines). Currently all fines are hardcoded and always active.

## Tab Location
Admin → 💸 Fines (4th sub-tab, after Avail.)

## To Be Defined
Implementation details TBD. When building, follow the existing admin tab aesthetic:
- Dark cards, green accents, gold headings
- Use same CSS variables already in `index.html`
- Two-layer `window.fn` → `window._impl_fn` delegation for all onclick handlers
- Firebase write helpers: `dbSet`, `dbUpdate`, `dbRemove`
