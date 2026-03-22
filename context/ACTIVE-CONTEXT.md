# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Any AI surface reads this to know the current state without searching conversation history.
> Last updated: 2026-03-22

---

## Current Phase

Full app audit complete (68/100). Immediate security fixes shipped (ErrorBoundary, XSS sanitization, photo validation, delete cascades). Frequency Station Phase 2 live — first song "Come Alive" published. PM workspace at ~95/100. Finance at ~78/100. App health ~75/100 after security fixes.

## What Just Shipped (March 22)

1. **PM Workspace Phases 1-4** — security guards, settlement locking, multi-user (claimPMSpot.ts, JoinPM, triple invite codes, PMWorkspaceMember, PMTenant), Polish (init seeds, form validation, toast errors, photo validation). Score 35→95.
2. **Finance Workspace All Phases** — ownership guards, manageFinanceWorkspace.ts, init seeds V2 categories, mobile fixes. Score 52→78.
3. **Full App Audit** — 68/100 overall. 10 sections audited. Server functions 9/10. Foundation 8/10. Cross-workspace consistency 5/10. Garden alignment 6/10.
4. **Immediate Security Fixes** — React ErrorBoundary, DOMPurify XSS on all input points, validateFile() on all uploads, manageBusinessWorkspace.ts server cascade, manageFieldServiceWorkspace.ts server cascade.
5. **Frequency Station Phase 2** — anonymous privacy fix, admin song creation with audio upload, audio player (compact+full), Listen tab wired to FrequencySong, SongDetail page at /frequency/:slug, share links.
6. **Property deed** — Joyce Ann Fletcher warranty deed filed (2054/2064 Marion St, North Bend). Medicaid concern flagged.
7. **Ephraim's Game Lab** — separate Claude project with fractal philosophy.

## Active Nodes

| Node | Status | Score | What's Next |
|------|--------|-------|-------------|
| Community Node | Pilot-ready, blocked on legal | ~75/100 | Newsletter Issue 1, admin pagination, open event creation |
| Field Service Workspace | Phase 6 complete (e-sign) | ~85/100 | Phase 7: Payments & Invoicing (blocked on LLC/EIN) |
| Property Management Workspace | Phase 4 complete | ~95/100 | Field test with real property data (deed filed, Medicaid review needed) |
| Personal Finance | V1 complete | ~78/100 | Field test with real data |
| Play Trainer (Team) | Go-live ready | ~95/100 | Share invite with Coach Rick and players, load plays with boys |
| Frequency Station | Phase 2 shipped | Functional | First song published, Listen tab fix deployed |

## What's In Flight

- **"Come Alive"** — first song published on Frequency Station, pending Listen tab verification after Base44 sync
- **Boys with Doron** — Friday through Thursday, loading plays and testing Team workspace
- **Bari** — Field Service active user, Oregon ORS confirmation pending
- **Ephraim** — Game Lab project created, returns for next session
- **LLC/EIN** — paper filing pending, blocks Stripe Connect

## Current Blockers

- **LLC/EIN paper filing** — blocks Stripe Connect, blocks Phase 7 Payments & Invoicing
- **AdminSettings platform_stats:member_count** — needs manual Base44 creation for CommunityPulse headcount
- **Base44 .filter().list() quirk** — confirmed recurring issue. Safe pattern: .list() then client-side filter.
- **Medicaid review** — deed transfer may trigger 5-year Medicaid lookback. Need legal guidance before active property management.

## What's Next

- Verify "Come Alive" on Listen tab (post-sync)
- Doron Frequency Station feedback
- Share team invite codes with Coach Rick and parents
- Load remaining plays with boys
- Newsletter Issue 1 send
- High priority audit fixes: initializeWorkspace ownership checks, audit logging
- Open event creation to community (Garden Door migration)
- Surface PM Listings to directory
- Admin pagination (audit finding)
- Mycelia testimony draft (5/19 trial)

## Garden Migration Status

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1: Language | Garden metaphors in UI copy | ✅ Shipped (Shaping the Garden, Frequency Station) |
| Phase 2: Door | Open events to all, directory visibility controls | 🔜 Next |
| Phase 3: Pulse | CSS vitality hooks, CommunityPulse data wiring | Planned |
| Phase 4: Guide | Workspace walkthrough system | ✅ Shipped (all 5 types) |
| Phase 5: Game | Superpowers, Quests, Organism Phase 1 | Planned |

## Time-Sensitive Items

- **Custody trial:** 2026-05-19 — revenue and stable employment needed before this date
- **Boys with Doron:** This week (Friday through Thursday)
- **Ephraim:** Returns for Game Lab session

---

*Overwrite this file at the end of every session. Commit and push to spec-repo so all surfaces read current state.*
