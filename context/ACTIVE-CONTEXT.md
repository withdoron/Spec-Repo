# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Any AI surface reads this to know the current state without searching conversation history.
> Last updated: 2026-03-21

---

## Current Focus

Team workspace go-live ready. 11 ships this session. Fractal SOPs established. DEC-064 Creation Station, DEC-090 Industry Presets, DEC-091 Three-Tier Role System all shipped/specced. Field Service fixes landed. Print Playbook live with three 3-6-9 layouts. Split household support (parent_user_ids array). Claude Code confirmed as primary implementation surface.

## Active Nodes

| Node | Status | Current Build | What's Next |
|------|--------|--------------|-------------|
| Community Node | Pilot-ready, blocked on legal | Community nav + Frequency Station shipped | Fractal audit fixes, Newsletter Issue 1 |
| Field Service | Phase 6 complete (e-sign), DEC-090 specced | Bari active user, industry presets specced | DEC-090 Industry Presets build, test doc seeding, Phase 7 (blocked on LLC/EIN) |
| Property Management | V1 Complete | Doron (port from Property Pulse) | Field test with real property data |
| Personal Finance | V1 complete | Doron | Field test with real data |
| Play Trainer | Go-live ready, 11 ships today | Creation Station, role system, print playbook all shipped | Parent walkthrough test, share invite codes with Coach Rick, load plays, flag Game Day, print Quick Reference |
| Frequency Station | Phase 1 shipped | Submit + My Seeds + Queue | Phase 2: music generation, playlist curation |

## Garden Migration Status

| Phase | Name | Status | What Ships |
|-------|------|--------|-----------|
| 1 | Guide Layer | COMPLETE | All 5 workspace guides, WorkspaceGuide.jsx, useWorkspaceInit |
| 2 | Door Layer | NEXT | Open event creation to all, directory visibility consent toggle |
| 3 | Pulse Layer | Queued | Freshness signal → self-trend → diversity → wire into data-vitality CSS |
| 4 | Play Layer | IN PROGRESS | Shaping the Garden shipped, Frequency Station Phase 1 shipped, community nav shipped |
| 5 | Game Layer | Future | Superpowers, Quests |

## What Just Shipped (2026-03-21)

1. Shaping the Garden moved to /shaping with Dashboard breadcrumb
2. O&P and Xactimate split into two independent toggles
3. Documents guide step verified, debug logs removed from initializeWorkspace
4. DEC-090: Industry Presets for Field Service specced
5. Team Workspace Audit — all 13 sections pass, two minor fixes
6. Head Coach designation removed — all coaches equal, "Transfer Ownership" label
7. DEC-064: Creation Station — 3-6-9 playbook structure (Game Day / Playbook / Creation Station)
8. DEC-091: Three-Tier Role System — Coach/Parent/Player, dual invite codes, claimTeamSpot.ts
9. Print Playbook — Player Card (3), Quick Reference (6), Full Page (9) layouts
10. Pending badge fix, coach-parent claim fix, amber heart toggle
11. Split household support — parent_user_ids JSON array
12. Fractal SOPs established in SuperMemory

## Current Blockers

- **LLC/EIN paper filing** — blocks Stripe Connect, blocks Phase 7 Payments & Invoicing
- **AdminSettings platform_stats:member_count** — needs manual Base44 creation
- **Base44 fields need manual creation** — coach_invite_code on Team, created_by_name on Play, parent_user_ids on TeamMember, industry_preset/overhead_profit_enabled/xactimate_enabled on FieldServiceProfile

## What's Next

1. Test account walkthrough as parent
2. Share invite codes with Coach Rick and team parents
3. Load remaining plays, flag Game Day plays
4. Print Quick Reference for sideline
5. Newsletter Issue 1
6. Frequency Station Phase 2
7. Mycelia testimony draft (5/19 trial)
8. DEC-090 Industry Presets build

## What's In Flight

- Parent walkthrough test (next session)
- Coach Rick invite share
- Plays loading and Game Day flagging
- Newsletter Issue 1
- Frequency Station Phase 2
- Mycelia testimony draft (5/19 trial)

## Time-Sensitive Items

- **Custody trial:** 2026-05-19 — revenue and stable employment needed before this date
- **Boys with Doron:** This week
