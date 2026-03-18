# Active Context

> What's happening right now. Updated each session.
> Last updated: 2026-03-18

## Current Phase

Platform live with organic traction. Field Service V2 shipped — five builds in one day, Bari Swartz confirmed as first real user. Six workspace types operational (Business, Team, Finance, Field Service, Property Management, plus workspace engine). Team workspace go-live ready with 12 players loaded. First paying customer conversations active. Music/creative branch growing.

## What Just Shipped (2026-03-18)

Field Service V2 — five builds for Bari demo:
- Feature toggles: 6 workspace settings (permits, subs, mgmt fees, insurance/O&P, payments, timeline)
- Unified estimate line items: single table with category tags (materials/labor/sub/fee/other), auto-calc, reorder
- Professional preview: two-column header, customer block, signature block, print CSS
- Payment terms: dropdown (Due on Receipt, Net 10-60, Custom), prepared_by field
- Change Orders: FSChangeOrder entity, auto-numbering, budget breakdown (Original + COs = Current)
- Estimate locking: Draft→Sent→Accepted (locked, read-only), Reopen option
- Xactimate insurance format: toggle, trade category grouping, per-category subtotals, 18 default categories
- Old estimates auto-migrate to unified format on read
- 12 new Base44 fields + 1 new entity (FSChangeOrder)

Bari meeting — confirmed as first user:
- 78 years old, humble, same church, similar frequency
- Works with insurance adjusters, needs Xactimate formatting
- Gave Doron $112K Ag Building Remodel estimate to enter
- Free during partnership phase — value from relationship
- Dan Sikes identified as second user (estimates first)

## What's In Progress

- Bari Swartz — first Field Service user, shadowing his workflow next
- Dan Sikes — second Field Service user, introduce estimates
- EIN for MYCELIA LLC — IRS system blocked twice, SS-4 form ready to fax
- Tyler (contractor friend) — potential tester and liaison
- Get Air (Charlie) — Community Pass proposal emailed
- Defy (Amber/Mike) — proposal emailed
- Music branch — 20+ songs, production workflow active

## What's Queued

1. Wednesday: Fresh conversation, admin panel audit, Claude Code orientation
2. EIN: try online, fax SS-4 backup
3. Get Bari's logo, enter his Ag Building Remodel estimate
4. Shadow Bari's workflow when he starts using workspace
5. Dan Sikes: introduce Field Service workspace for estimates
6. Register claimWorkspaceSpot.ts server function in Base44
7. YouTube first video concept
8. LinkedIn post (wolf/lion/AI cliff)
9. Newsletter Issue 1: "Come play"
10. 3-6-9 spec document for private repo
11. Finance V2 field test with real SELCO data
12. PM workspace field test with real property data
13. Add Mycelia LLC Business context (after EIN)
14. CLAUDE.md conventions update
15. DEC-064/065 Play Trainer builds
16. Music: Surrender, In The Dirt, Two Witnesses — hard rap versions
17. Business seeding: Horai, NW OG Farm, Grassward Dairy

## Active Decisions

- DEC-059: Claude Code is primary dev tool
- DEC-068: Field Service Workspace — V2 shipped, Bari confirmed
- DEC-069: Property Management Workspace — V1 COMPLETE
- DEC-070: Team roles simplified (Coach + Player only)
- DEC-071: FSClient entity as first-class object
- DEC-072: Client visibility controls (breakdown toggle)
- DEC-073: Worker/sub access model (invite + claim + role gating)
- DEC-074: Sovereign Worker Network concept
- DEC-075: Music/creative branch ("The OG")
- DEC-076: Business finance dashboard architecture (three levels)
- DEC-077: Unified estimate line items (single table, category tags)
- DEC-078: Estimate locking on acceptance (changes via Change Orders)
- DEC-079: Xactimate insurance format (toggle, trade categories)
- DEC-080: Feature toggles (6 workspace settings by archetype)
- DEC-081: Free pricing during partnership phase

## Key Context for Next Session

- Field Service is PRODUCTION-GRADE — V2 with unified estimates, change orders, Xactimate, feature toggles, client visibility, worker access
- Bari is FIRST REAL USER — enter his $112K Ag Building Remodel estimate, get his logo, shadow workflow
- Dan Sikes is SECOND USER — introduce estimates first, expand from there
- Change Order system is the first sub-entity workflow — pattern reusable across workspace types
- Feature toggles make workspace archetype-driven at settings level — same code, different presentations
- Xactimate format: "same data, different presentation" — one source of truth, multiple views
- LFS declined — seed fell on sand, no reply needed
- EIN blocked twice — SS-4 fax ready as backup
- claimWorkspaceSpot.ts still needs registering in Base44
- invite_code field still needs adding to FieldServiceProfile in Base44
- Community Pulse: 9 Members, 4 Businesses, 10 Subscribers
