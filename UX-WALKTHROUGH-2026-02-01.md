# UX Walkthrough Findings — 2026-02-01

> Chrome extension walkthrough as test user (doron.bsg+test@gmail.com)
> Overall Score: 7.5 / 10

---

## What's Working Well

- ✅ Gold Standard dark theme — consistent, professional, no white flash
- ✅ Business profiles are comprehensive (services, pricing, contact, tabs)
- ✅ Navigation is intuitive — clear nav, proper page highlighting, back buttons work
- ✅ Hover states clean across all tested elements (Phase 1 fixes confirmed)
- ✅ Filter chips on Events page work correctly with amber active state
- ✅ Search bar on Directory filters in real-time with amber focus ring

---

## Action Items

### High Priority (Before Pilot)

| # | Finding | Fix | Effort | Blocked By |
|---|---------|-----|--------|------------|
| 1 | Event detail modal has no RSVP/Attend button | Add RSVP button to event detail modal | Medium | RSVP entity/system needed |
| 2 | Punch Pass access is buried (only via Dashboard "0 Passes" button) | Add "Punch Pass" to user dropdown menu | Small | None |
| 3 | "FREE" badge shown on event cards but disappears in detail modal | Add price/free badge to event detail modal | Small | None |

### Medium Priority (Polish)

| # | Finding | Fix | Effort | Blocked By |
|---|---------|-----|--------|------------|
| 4 | User menu has no Settings option | Add "Settings" link to user dropdown (can link to placeholder page initially) | Small | None |
| 5 | "Bullion Dealers & Coin Shops" category card has light/cream background that's visually jarring | Check image or override background on that category tile | Small | None |
| 6 | Event category showing "sports & & & active" | Fix test data formatting (data cleanup, not code bug) | Small | None |

### Addressed by MyLane Build

| # | Finding | Resolution |
|---|---------|------------|
| 7 | No personal home base for users | MyLane Phase 1 — the entire spec addresses this |
| 8 | Punch Pass not easily discoverable | MyLane GreetingHeader includes Punch Pass balance card |
| 9 | No clear "what should I do next" for logged-in users | MyLane progressive sections guide naturally |

---

## Quick Wins (Could Ship Today)

Items 2, 3, 5, and 6 are all small fixes that don't depend on anything else. Could be a single Cursor prompt.

---

*Filed from Claude in Chrome UX walkthrough, 2026-02-01*
