# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-24 (Phase 3 marathon shipped — Builds 1-patch, 2, B, C, D, E + arch v4.1 amendments + first revenue)

## Current Focus

Round 1 Phase 3 is ~95% complete after a six-build marathon. Switcher works end-to-end, public surfaces filter on `listed_in_directory`, profile Settings has editors for every previously-uneditable rendered field, "Jobsite" is gone from the user-visible vocabulary in favor of "Desk," cockpit centers cleanly on wide viewports, and `service_area` is now a structured array of Lane County town slugs. **First external revenue arrived today** — Bari Swartz paid a $500 retainer check at the 2026-04-24 morning meeting. Pattern C+ migration plan resolved (DEC-175): defer Supabase + Vercel migration to Phase 6, combined with Region backfill, sandbox on `lanecountyrecess.com`. Build F (multi-category + SDK wrap into `src/api/`) closes Phase 3; Phase 3.5 (direct doors `/b/{slug}`) follows.

## Active Architecture

- **Cockpit-native business switcher live (DEC-168):** `useActiveBusiness()` hook + `localStorage` key `locallane.activeBusinessId`. Spinner renders nested switcher spinner accessed by tapping the space-name pill; only wakes when `isMultiBusiness`. Build 1 commit-handler patch (`c0d7c3f`) made center-tile commits fire reliably.
- **Directory visibility (Build 2):** `src/utils/directoryVisibility.js` is the single filter helper. Public surfaces (`Directory.jsx`, `NetworkPage.jsx`, `Home.jsx`) all read through it. Owners always see their full tree (per architecture §13.1). Mycelia, LLC stays hidden from public.
- **Profile Settings editors (Build B):** universal `services[]` structured editor, tagline, photos gallery, accepts toggles, empty-state nudge. Closes the Phase 2 origination gap. Subtitle ladder on `BusinessProfile`: tagline → category label → "New to LocalLane."
- **Desk vocabulary (Build C, DEC-170):** user-visible "Jobsite" replaced with "Desk" in MyLane label + welcome map + HomeFeed priority spinner. `FieldServiceAgent` persona text intentionally preserved. Phase 4 will rename the implementation tier (DEC-160).
- **Cockpit centering (Build D):** removed the dead `.mylane-content-area.panel-open { margin-right: 300px }` rule reserving width for an absolute-overlay agent panel. Clean centering at 375 → 2560 px.
- **Service area structured (Build E, DEC-174):** `src/config/laneCountyTowns.js` exports a 21-town curated list (incorporated cities + notable unincorporated communities). `src/components/business/TownMultiSelect.jsx` is the reusable curated multi-select chip component. `service_area` on Business is now `array<slug>`. Universal across archetypes — no longer gated to `service_provider`. Legacy strings preserved verbatim with owner-only "(legacy)" annotation. `serviceAreaMutation` writes immediately on chip toggle.
- **Architecture v4.1 closure (`955f8e2`):** DEC-169 (folder architecture), DEC-170 (Home collapses into Desk), DEC-171 (path-based direct doors Round 1, custom domains Round 2+), DEC-172 (Region as first-class entity with seed/sprouting/established lifecycle, "carried in the wind" growth), DEC-173 (resurface before rebuild — standing rule).
- **Production Business tree (Phase 2 state preserved):** Mycelia, LLC → LocalLane / TCA / Recess; Red Umbrella is Bari's peer; sandbox archived. 9 Phase 2 AuditLog rows still reversible.
- **Two-tier FSDocumentTemplate (DEC-163), business-first branding (DEC-164), template preview (DEC-165), Bari's contracts loaded (DEC-166), schema-conformance audit protocol (DEC-167)** — all live from 2026-04-23.
- **Mylane Agent v2, smart routing, shell containment** — all live, no regressions from any Phase 3 build (UI-additive only).
- **Health score:** 87/100 (unchanged).

## Migration Plan (DEC-175 — Pattern C+)

- **Trigger:** Phase 6 (Region foundation backfill window). One fragility cost, not two.
- **Target stack:** Supabase + Vercel.
- **Sandbox:** `lanecountyrecess.com`. `locallane.app` stays on Base44 during construction.
- **AI:** retire all 8 Base44 agents at migration. Build a single warm-presence companion fresh, designed from Bari's "AI tour guide" framing.
- **Seam hardening:** Phases 4-5 contain no new direct CRUD. Build F (next) bundles the SDK wrap into `src/api/`, validated by the multi-category build as first consumer.
- **Reference:** `community-node/docs/migration-research.md` (commit `78fc8c7`; flagged for cleanup).

## What Just Shipped (2026-04-24, Phase 3 marathon)

1. **Build 1 commit-handler patch (`c0d7c3f`):** SpaceSpinner pointer-capture fix; `onCenterTap` prop. Switcher functional end-to-end. Preceded by `e6b4c4c` (TDZ fix) and `87a8ee4` (diagnostic).
2. **Build 2 (`65e3fd7`):** directory visibility helper + Settings toggle + `listed_in_directory` allowlist.
3. **Build B (`6521232`):** profile Settings editors + tagline render + photos + accepts toggles + empty-state nudge. Diagnostic commit `57db9d6` preceded.
4. **Build C (`9598128`, DEC-170):** Jobsite → Desk vocabulary.
5. **Build D (`47d1af2`):** cockpit centering on wide viewports.
6. **Build E (`7c21c3e`, DEC-174):** structured service area editor + `laneCountyTowns.js` constants + `TownMultiSelect` reusable component.
7. **Architecture v4.1 amendments (`955f8e2`, Spec-Repo):** DEC-169 through DEC-173.
8. **First external revenue:** Bari $500 retainer check.
9. **DEC-174 + DEC-175 recorded** in this Spec-Repo ship-it commit.

## Known Issues

1. **`community-node/docs/migration-research.md`** was supposed to be deleted post-decision per its own ephemeral note; still present in main as of `78fc8c7`. Cleanup owed in a future community-node commit.
2. **DECISIONS.md drift** between Spec-Repo and community-node — pre-existing, not touched today. Future: dedicated audit/merge session.
3. **FSDocumentTemplate `rls.update` creator-only** — workaround documented; logged in private/TECH-DEBT.md.
4. **Phase 2 FieldServiceProfile + User `security.update: true` with no RLS** — wide-open by design for migration; Phase 5 re-tightens. Logged.
5. **Base44 publish blocker** — escalation request `95a004a0` still open. Affects DEC-115 deploy of Mylane write capability. No new agent writes blocked by this for the Phase 3 builds (all UI/data-shape changes).
6. **Base44 auto-push behavior** — DEC-162 working agreement mitigates; expect "File changes" commits any entity-change session.
7. **Overlay z-indices hardcoded** — z-50/55/60; refactor when third stacked-overlay scenario appears.
8. **ClaimBusiness route + BusinessEditDrawer claim URLs** — co-presence model retires this; cleanup pending.
9. **FrequencyLibraryContext refactor** — prop drilling, planned wrap.
10. **Footer renders on non-MyLane pages** — limited purpose now.

## Organism Milestones (this session)

- **First external revenue:** Bari $500 retainer (2026-04-24). Pricing structure agreed: $45/mo founding platform (locked vs $69 launch standard), $45/hr dogfood, $90/hr service, $40 service call, $350 half-day, $650 full-day.
- **Active paying members: 1** (Bari).

## In Flight

None. Phase 3 marathon shipped in a settled state.

## Active Blockers

None.

## Upcoming Priorities

1. **Build F** — multi-category support + SDK wrap into `src/api/` (closes Phase 3; validates the wrapper shape with first real consumer; first migration-prep seam hardening per DEC-175)
2. **Build G** — Desk tile icon swap (cosmetic, ~15 min)
3. **Build H** — workspace content centering on wide monitors (cosmetic; defer past Build F)
4. **Phase 3.5** — direct doors `/b/{slug}` (DEC-171)
5. **Phase 4** — Desk implementation rename + business-scoped rendering (DEC-160, DEC-156)
6. **Phase 5** — membership gate ($9/mo per entity, DEC-155) — re-tightens FSProfile/User `rls.update`
7. **Phase 6** — onboarding fork + Region backfill + **Pattern C+ migration window** (DEC-175)
8. **Round 2** — Stripe Connect (real money flow + legacy grace population per DEC-159)
9. **`community-node/docs/migration-research.md` cleanup** (carryover)
10. **DECISIONS.md drift audit + merge session** (carryover)
11. **Newsletter "The Good News"** — wake dormant accounts
