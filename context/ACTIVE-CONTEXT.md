# Active Context

> What's happening right now. Updated each session.
> Last updated: 2026-03-16

## Current Phase

Platform live with organic traction. Six workspace types operational (Business, Team, Finance, Field Service, Property Management, plus workspace engine). Team workspace button-up complete, roster loaded with 12 real players for spring season. Field Service workspace fully overhauled for Bari demo Tuesday. Music/creative branch opened. Sovereign Worker Network concept documented. First paying customer conversations active.

## What Just Shipped (2026-03-15 / 2026-03-16)

Team Workspace — button-up and go-live ready:
- Play Builder: 12 fixes across 13 files (roster CRUD, format dropdown, config positions, SidelineMode, photo protection)
- Head Coach system (head_coach_member_id, shield badge, Set as Head Coach)
- manageTeamPlay server function (any coach edits any play via asServiceRole)
- Role simplification: Coach and Player only (DEC-070). Legacy assistant_coach normalized.
- 12 real players loaded for spring season. Role audit confirmed safe for sharing.

Field Service Workspace — complete overhaul for Bari demo:
- FSClient entity (DEC-071) — clients as first-class objects anchoring the workflow
- Client visibility controls (DEC-072) — per-project and per-estimate breakdown toggle
- Worker/sub access (DEC-073) — invite code + claim flow, 4 roles, view gating seeds
- People tab (workers, subs, clients), projects grouped by client, log editing, inline photos
- Phone auto-format, number zero-clearing, email/tel links, estimate print CSS, logo upload fix
- Mobile audit fixes, budget bar amber-only, "Save & Copy Link" on estimates
- 11 Base44 entities (FSClient added), ~30 commits across 2 days

Music / Creative:
- 20+ songs written, artist name "The OG - A hand full of seeds", artwork + thumbnails built

## What's In Progress

- Bari Swartz relationship — meeting Tuesday March 18, 1pm, Barry's Espresso (Field Service demo)
- Tyler (contractor friend) — texted, potential tester and liaison
- Travis (NW OG Farm) — farm visit done, subdomain interest, focus on eggs for market
- EIN application — retry Monday (IRS online)
- Get Air (Charlie) — Community Pass proposal emailed, awaiting response
- Defy (Amber/Mike) — proposal emailed, awaiting response
- Tory/LFS — board reviewing Doron's candidacy
- Dustin Castleberry — sovereign worker conversation planted, door open

## What's Queued

1. Monday: Help Dan paint 10am-3pm. EIN retry. Ask Dan about Stringfield pricing workflow.
2. Monday evening: Field Service final testing. Register claimWorkspaceSpot.ts in Base44. Test worker invite/claim.
3. Tuesday 1pm: Bari demo at Barry's Espresso
4. Share Team workspace invite with Coach Rick and players
5. CLAUDE.md update (user_id on creates, dictionary fields as objects, formatPhone, Core.UploadFile)
6. Finance V2 field test with real SELCO data
7. PM workspace field test with real property data
8. Add Mycelia LLC Business context in Finance workspace (after EIN/SELCO)
9. Jamie website (awaiting content)
10. Get Air walk-in
11. Newsletter Issue 1
12. withdoron.com polish
13. DEC-064/065 Play Trainer builds (Creation Station, Coach Mode)
14. View gating enforcement for workers (V2)
15. Wire linked_finance_workspace_id bridge (V2)
16. Business seeding: Horai, NW OG Farm, Grassward Dairy

## Active Decisions

- DEC-059: Claude Code is primary dev tool
- DEC-060: Typographic cards with ambient vitality
- DEC-066: Ideas Board — shipped
- DEC-067: Finance Workspace V2 — shipped
- DEC-068: Field Service Workspace — shipped, overhauled for demo
- DEC-069: Property Management Workspace — V1 COMPLETE
- DEC-070: Team roles simplified (Coach + Player only)
- DEC-071: FSClient entity as first-class object
- DEC-072: Client visibility controls (breakdown toggle)
- DEC-073: Worker/sub access model (invite + claim + role gating)
- DEC-074: Sovereign Worker Network concept
- DEC-075: Music/creative branch ("The OG")
- DEC-076: Business finance dashboard architecture (three levels)

## Key Context for Next Session

- Field Service is DEMO READY — FSClient entity, projects grouped by client, estimate preview with branding, client portal, worker invite flow, all amber psychology
- Team workspace is GO-LIVE READY — 12 real players loaded, manageTeamPlay server function, role audit clean, Coach Rick can share invite
- Bari confirmed Tuesday March 18, 1:00pm, Barry's Espresso SE Eugene
- claimWorkspaceSpot.ts needs registering in Base44 before Monday testing
- invite_code field needs adding to FieldServiceProfile in Base44
- Tyler potential liaison for Stringfield channel — ask Dan about pricing workflow Monday
- Dustin Castleberry: sovereign worker seed planted, don't push, let him come back
- Travis: too many things going, encouraged eggs-for-market focus
- 20+ songs written — "The OG - A hand full of seeds" — creative branch is real
- $9 as organism membrane (Cerberus metaphor) — skin that ensures only the living pass through
- Community Pulse: 9 Members, 4 Businesses, 10 Subscribers
