# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-30 (Field Service Phase 1 functionally complete — 8 commits, 5 new DECs; pending dogfood test of `317950e` and Estimate Types prompts)

## Current Focus

**Field Service Phase 1 is functionally complete.** Eight commits in community-node today (`32ccb92`, `e72e28c`, `ef14ae9`, `c55504c`, `3c218d4`, `db138bf`, `148290d`, `317950e`) shipped the full estimate-to-CO-to-payment pipeline end-to-end: change-order math + signing + recompute, CO form unification with the estimate builder, draft-CO edit, Log Sub Payment + Client Payment types, Project Detail four-metric financial header, the feature-flag wiring fix that closed an asymmetric-failure class of bug (DEC-194), Management Fee separated from O&P (DEC-195), the polish bundle (Tax toggle + CurrencyInput + Cancel button + PDF branding + CO Delete/Void with paired Base44 prompt), and the Settings-persistence cache-key fix (DEC-196) plus scroll-to-top after save plus CurrencyInput format-while-typing. Five new DECs ratified (DEC-193 through DEC-197). One paired Base44 schema arc applied across FSPayment, FSProject, FSChangeOrder, FSEstimate.

**Phase 4.2-tiles remains the parallel workstream** (last shipped state: tiles-1 through tiles-4 + cleanup, 2026-04-28). Tiles-5 (Settings + Profile workspace surfaces + pricing-structure design) is queued but not blocking Field Service work. The two workstreams ran in parallel today and resolve in the same Spec-Repo/community-node trees without conflict.

**Tomorrow's first move:** dogfood verification of `317950e` (Settings persistence + scroll + format-while-typing across all surfaces) → Estimate Types prompts (Base44 + Hyphae) → Bari's first real estimate entry against Phase 1. Parallel: Dan Sikes logo variants via Gemini (separate workstream, doesn't block).

## Active Architecture

### Field Service Phase 1 (shipped today)

- **CO math primitive (DEC-193):** `signChangeOrder` + `voidChangeOrder` server functions. FSProject.total_budget recompute reads CO `amount` (canonical), not CO `total` (display). Two values diverge once Mgmt Fee / O&P / Tax / Other modifiers land on a CO; conflating them would silently under-bill the parent project.
- **Feature flag canonicalization (DEC-194):** `features_json` is the single source of truth on FieldServiceProfile. All reads through `getFeatures(profile)` (`src/utils/fsFeatures.js`). Top-level legacy boolean fields deprecated; optional Base44 cleanup prompt staged at `community-node/base44-prompts/PHASE-1-DEPRECATE-LEGACY-FEATURE-FLAGS.md`. Eight flags in FEATURE_DEFAULTS (permits, subs, management_fees, overhead_profit, xactimate, tax, payments, timeline).
- **Management Fee distinct from O&P (DEC-195):** Two first-class features. Management Fee for GC cost-plus-management billing (Bari's mode); O&P for insurance-scope work (Xactimate-style). Both subtotal-only basis; never stack. Display order locked across all five surfaces: **Subtotal → Management Fee → O&P → Other → Tax → Total**.
- **Cache invalidation discipline (DEC-196):** `invalidateFSProfiles(queryClient, userId)` helper in `src/utils/fsFeatures.js`. Targets the actual subscriber queryKey (`['mylane-profiles-v2', userId]` per DEC-130), not the previously-stale `['fs-profiles']` key. RQ v5 invalidation against unsubscribed keys is a silent no-op — Living Feet helper prevents drift across 10 invalidation sites.
- **Opt-in default for billing-shape toggles (DEC-197):** Management Fee, O&P, Xactimate, Sales Tax, Insurance Work all default `false` on new profiles. Standard infrastructure flags (Permits, Subs, Payments, Timeline) default `true`. No industry-preset auto-defaults; no location-based heuristics.
- **Universal capture surface (Log) — Phase 1 Item 4:** Type picker — Daily Log + Sub Payment + Client Payment. Sub/Client Payment write FSPayment with proper `direction` + party fields. New `useFSPayments` shared query hook.
- **Project Detail financial header (Phase 1 Item 5):** Contract / Received / Paid Out / Net Cash four-metric banner above the existing Budget breakdown. Daily check-in framing per FINANCIAL-WORKFLOW-SPEC §2.6.
- **Polish primitives:** `CurrencyInput` (format-while-typing, cursor-managed via digit-and-dot-count invariant, applied at 10 sites). `scrollToTopOf(startEl)` (walks to closest scrollable ancestor and resets, used by Settings save + Log mount). Both extracted as Living Feet (DEC-146) on second consumer.
- **CO Void schema + UX (Phase 1 polish):** New `voided` status enum + `voided_at` + `voided_reason` fields on FSChangeOrder. Two-step typed-VOID confirmation. Voided COs render line-through + muted gray badge + `Voided {date} — {reason}` info line. Filter `signed | accepted` naturally excludes voided records — no filter changes anywhere downstream.

### Phase 4.2-tiles (parallel workstream, last shipped 2026-04-28)

- **Phase 4.2-tiles structurally complete (2026-04-28)** — tiles-1 (generic Tile primitive at `src/components/ui/Tile.jsx`), tiles-2 (BreadcrumbPath at `src/components/ui/BreadcrumbPath.jsx`), tiles-3 (TilesCockpit at `src/components/mylane/TilesCockpit.jsx`, default for all users with allowlist-gated picker), tiles-4 (per-business folder rendering, space-type catalog at `src/config/spaceTypes.js`, DEC-148/DEC-168 retired for tile users) plus cleanup (Field Service removed from Personal — Desk is a business-only space per the city/buildings metaphor).
- **Space-type catalog (DEC-190)** — `src/config/spaceTypes.js` is the third living-feet config in the codebase (alongside `folderTree.js` and `folderPredicates.js`). Eight entries. Profile + Settings flagged `universal: true` per DEC-186. Helper `resolveBusinessSpaces(business)` unions universal pair with `enabled_spaces` and dedupes.
- **Cockpit picker pattern (DEC-188)** — `COCKPIT_PICKER_ALLOWLIST` in `MyLaneSurface.jsx` parallels `MYLANE_AGENT_ALLOWLIST`. Allowlisted users see the cockpit toggle; non-allowlisted users see no toggle. Force-migration in MyLaneSurface mount effect bumps non-allowlisted users from spinner/compass to tiles.
- **Uniform navigation pattern (DEC-191)** — every root tile descends into the render layer for tile cockpit users. Spinner cockpit users (allowlisted only) keep DEC-148 overlays + DEC-168 switcher.
- **Engagement entity built in Base44 (2026-04-28)** — design fully closed via DEC-192. First instance (Doron-Bari retainer) pending entity availability + UI build. Read permission deviation: row-level scoping in query logic.

### Cross-cutting

- **Multi-machine infrastructure live (DEC-181)** — Mac mini primary, MacBook Pro 2017 secondary. Today's session ran from the mini.
- **Single-source documentation policy (DEC-182)** — Spec-Repo is canonical. `community-node/context/` mirrors are refreshed by drift-sync passes. This ship-it commit syncs the mirror.
- **Schema-conformance discipline (DEC-167 / DEC-177 / DEC-178)** — code-level Base44 entity changes ship paired with Base44 prompts. Today's Phase 1 schema work shipped paired throughout.
- **Mylane Agent v2, smart routing, shell containment** — all live, no regressions from today's Phase 1 work.
- **Health score:** 87/100 (unchanged).

## Migration Plan (DEC-175 — Pattern C+)

- **Trigger:** Phase 6 (Region foundation backfill window). One fragility cost, not two.
- **Target stack:** Supabase + Vercel.
- **Sandbox:** `lanecountyrecess.com`. `locallane.app` stays on Base44 during construction.
- **AI:** retire all 8 Base44 agents at migration. Build a single warm-presence companion fresh, designed from Bari's "AI tour guide" framing.
- **Seam hardening:** Build F bundled the SDK wrap into `src/api/`, validated end-to-end. `updateProfile()` wraps the server-function write path per DEC-177.
- **Reference:** `community-node/docs/migration-research.md` (commit `78fc8c7`; flagged for cleanup).

## Field Instrument (sibling product, seed-stage, separate from LocalLane)

- **FIELD-INSTRUMENT-SEED.md committed to `private/` root 2026-04-26** (`e00bcda`). Originally drafted 2026-04-12.
- **v0.1 protocol live-tested 2026-04-24** with Pedrom Rejai. Successful — both AIs hit the protocol's posture. Pedrom's AI flagged a framing-bias failure mode.
- **Standalone LLC under Mycelia LLC**, sibling to LocalLane.
- **Companion artifacts** still on laptop; commit alongside `alibi-protocol` GitHub repo setup once Pedrom responds.

## Multi-Machine Infrastructure (DEC-181)

- **Primary:** Mac mini.
- **Secondary:** MacBook Pro 2017 (mobility, field visits to Bari).
- **Discipline:** pull as the first action of any session, push after every commit.

## What Just Shipped (2026-04-30 — Field Service Phase 1 build sprint)

**Eight commits in community-node:**

1. **CO math + signing flow + total_budget recompute (`32ccb92`):** `signChangeOrder` server function, portal token fields on FSChangeOrder, RLS relaxations on FSChangeOrder + FSProject, per-project sequential CO numbers, lightweight Project Detail / Client Portal CO blocks. Architectural primitive: signed CO `amount` is canonical for `total_budget` recompute (DEC-193).
2. **CO form unification + RQ v5 fix (`e72e28c`):** Extracted `LineItemsEditor` + `calcTotals` + `CATEGORIES` into shared utils. Percentage fields added to FSChangeOrder. Bare-array invalidation form fixed across 5 sites (RQ v5 silent no-op).
3. **Edit-Draft on COs (`ef14ae9`):** `editingCOId` state, `openEditCOForm` helper, `saveCOMutation` create/update branch. Edit only on draft status.
4. **Log payment types + Project Detail financial header (`c55504c`):** Sub Payment + Client Payment entries. `useFSPayments` shared hook. Four-metric Contract / Received / Paid Out / Net Cash banner. Financial Ledger Estimated extends to signed CO line items.
5. **Feature flag wiring fix (`3c218d4`):** `getFeatures(profile)` helper extracted into `src/utils/fsFeatures.js`. `features_json` becomes canonical (DEC-194). Asymmetric failure pattern documented.
6. **Management Fee separated from O&P + Log scroll fix (`db138bf`):** `management_fee_pct` + `management_fee_amount` on FSEstimate + FSChangeOrder. Settings toggle (default off). `calcTotals` accepts 5th arg. Display order locked: Subtotal → Mgmt Fee → O&P → Other → Tax → Total (DEC-195). Log mount-effect scroll-to-top.
7. **Polish bundle (`148290d`):** Sales Tax toggle (default off, gates inputs + renders). `CurrencyInput` extracted, applied to 10 sites. Estimate Cancel button. PDF branding (LocalLane title + per-print title swaps). CO Delete (drafts) + Void (signed/accepted) with new `voidChangeOrder` server function. Paired Base44 prompt for void schema.
8. **Settings persistence + scroll + format-while-typing (`317950e`):** `invalidateFSProfiles` helper targeting the actual `['mylane-profiles-v2', userId]` cache (DEC-196). `scrollToTopOf` helper extracted on second consumer. CurrencyInput format-while-typing with cursor-managed digit-and-dot-count invariant.

**Paired Base44 schema (applied via dashboard):**

- **FSPayment** — direction, party_type, party_name, party_id, method (merged enum), reference, notes
- **FSProject** — original_budget immutable + total_budget derived (documented); Update RLS relaxed; Bari's Holman project backfilled `original_budget = $121,657.57`
- **FSChangeOrder** — status enum extended (`awaiting_signature`, `signed`, `voided`); new fields signed_at, signature_data, amount, voided_at, voided_reason; portal fields portal_token, portal_link_active, sent_for_signature_at, recalled_at; Update RLS relaxed; signing + void server functions published
- **FSEstimate** — `management_fee_amount` field added (management_fee_pct already existed)
- **FSChangeOrder** percentage fields — management_fee_pct, management_fee_amount, overhead_profit_pct, tax_rate, tax_amount, other_amount

**Decisions ratified today:**

- **DEC-193** — FSChangeOrder `total` vs `amount` distinction (display vs canonical recompute)
- **DEC-194** — `features_json` as canonical FieldServiceProfile feature flag store
- **DEC-195** — Management Fee distinct from O&P (two first-class features, subtotal-only basis, never stack)
- **DEC-196** — Cache invalidation must target subscriber's actual queryKey (RQ v5 silent-no-op gotcha)
- **DEC-197** — Fee/insurance toggles default off (opt-in only, no industry-preset auto-defaults)

## Strategic Clarifications (still in force)

- **Phase 3.5 (Direct Doors) deferred to post-migration** per DEC-179.
- **Phase 5 (Pre-Migration Cleanup)** between Phase 4.5 and Phase 6 per DEC-180.
- **Payment infrastructure deferred to post-migration.** Membership gate (DEC-155) and Stripe Connect both move to the post-migration window.

## Field Service Node — Production-Shaped (Phase 1 Complete)

Bari is the first external paying user. Multi-category classification working end-to-end after Build F. Score ~95/100. **Phase 1 of the financial workflow now shipped end-to-end** — estimate spine + signed CO amendments + Sub/Client Payment in Log + Project Detail financial header + CO Void with audit preservation. Bari's day-to-day workflow is supported architecturally; tomorrow opens on real-estimate use against the new surface. **NODE-LAB-MODEL.md phase-review note still owed** (private repo, deliberate review pending — flag persists).

**Per-business Desk wiring still owed for tiles workstream:** Field Service workspace assumes personal scope today. `MyLaneDrillView.jsx:53` resolves `fieldServiceProfiles?.[0]` — the user's first PERSONAL profile. Per-business Desk wiring needs a business-scoped resolver. Today's Phase 1 work didn't touch this; remains a tiles-5+ flag.

## Known Issues

1. **`community-node/docs/migration-research.md`** — supposed to be deleted post-DEC-175; still present.
2. **DECISIONS.md drift** between Spec-Repo and community-node — pre-existing.
3. **Stray Cursor references in Spec-Repo** outside today's PROJECT-BRAIN scope. Queued for May 4 Monthly Sharpening.
4. **Persistent Base44 SDK 404 console spam** on every page load — source unknown, not user-visible.
5. **Duplicate `DC` key React warning** in Radix Select — cosmetic seedling.
6. **`Business.categories` field empty in Base44** (DEC-176) — pending Phase 5 cleanup.
7. **FSDocumentTemplate `rls.update` creator-only** — workaround documented.
8. **Phase 2 FieldServiceProfile + User `security.update: true` with no RLS** — wide-open by design for migration; post-migration membership gate re-tightens.
9. **Base44 publish blocker** — escalation request `95a004a0` still open. Workaround in place: Kathy at Base44 ran a checkpoint reset that cleared today's Phase 1 publish path. Today's Base44 schema work + server functions published cleanly under that workaround.
10. **Base44 auto-push behavior** — DEC-162 working agreement mitigates.
11. **Overlay z-indices hardcoded** — z-50/55/60; refactor when third stacked-overlay scenario appears.
12. **ClaimBusiness + BusinessEditDrawer cleanup** pending (Phase 5 candidate per DEC-180).
13. **Footer renders on non-MyLane pages** — limited purpose now.
14. **JoinFieldService welcome-card "Go to desk"** soft-broken (gracefully degrades to no-op).
15. **Engagement Read permission** set to authenticated; row-level scoping in query logic. RLS replaces post-Supabase-migration.
16. **`['fs-profile']` (singular) cache** — separate narrower cache used by FieldServiceHome `guide_dismissed` and one Settings invitee handler. Not part of `invalidateFSProfiles` helper. Worth a separate audit pass when convenient.
17. **Legacy top-level feature flag fields** on FieldServiceProfile entity — deprecated by DEC-194; optional Base44 cleanup prompt staged but not yet applied.

## Organism Milestones (this session — 2026-04-30)

- **Field Service Phase 1 functionally complete.** Eight commits in one day shipped the full estimate-to-CO-to-payment pipeline plus polish bundle plus Settings persistence fix. Five new DECs (DEC-193 through DEC-197) ratified to lock in the architectural calls before they drift.
- **Two latent bugs resolved by diagnosis-led fixes** — `3c218d4` (feature flag wiring, asymmetric failure pattern) and `317950e` (cache key silent-no-op, masked by 5-min staleTime). Both ratified into DECs so the next equivalent bug is precluded structurally.
- **CurrencyInput cursor algorithm.** Built by hand using digit-and-dot-count-before-cursor invariant. ~40 lines + glue. No new dep added (`react-number-format` rejected for this scope).
- **Living Feet (DEC-146) applied twice today** on second consumer — `CurrencyInput` and `scrollToTopOf` both extracted from previous inline patterns.

## In Flight

None. Session closed in a settled state pending tomorrow's dogfood verification of `317950e`.

## Active Blockers

None. Base44 publish blocker workaround in place (Kathy's checkpoint reset).

## Upcoming Priorities

1. **Dogfood test of `317950e` (TOMORROW'S FIRST MOVE).** Settings persistence across all 8 toggles, scroll-to-top after save, format-while-typing on every CurrencyInput site. Apply paired Base44 prompt `community-node/base44-prompts/PHASE-1-CO-VOID-STATUS.md` if not yet applied.
2. **Estimate Types prompts (Base44 + Hyphae).** Adds `estimate_type` enum + 4 supporting fields to FSEstimate and FSChangeOrder. Drafted and queued in chat. Supports the three first-class billing models per FINANCIAL-WORKFLOW-INTENT §2.
3. **Bari's first real estimate entry** against the new Phase 1 surface.
4. **Dan Sikes logo variants via Gemini** — parallel workstream, doesn't block.
5. **Phase 4.2-tiles-5 — Settings + Profile workspace surfaces + pricing-structure design.** Pre-build design conversation about Profile/Settings split-point + pricing-model shape. Pricing field architecture in `spaceTypes.js` (null for now); charging deferred until Stripe integration. Then BusinessSettings lifted from the dashboard tab into a Settings space surface; BusinessProfilePage wired as the Profile space surface.
6. **Phase 4.2-tiles-6** — Cockpit picker allowlist gate cleanup or extension.
7. **Phase 4.2-tiles-7 (queued)** — Personal Profile + Settings — architectural symmetry. Adds `enabled_spaces` field to User entity. Public/private toggle. Foundation for user reviews.
8. **Per-business workspace wiring** for Profile/Desk/Finance/Team. Field Service specifically needs a business-scoped resolver (`MyLaneDrillView.jsx:53`).
9. **Permits library enhancement** — saved per-profile portal links with last-used surfacing. Phase 2 work; seedling.
10. **E-sign hardening (deferred)** — email magic link auth. Wait until real risk surfaces (bigger CO amounts, less-known clients, or first dispute).
11. **Base44 legacy field cleanup (staged)** — `community-node/base44-prompts/PHASE-1-DEPRECATE-LEGACY-FEATURE-FLAGS.md`. Run when ready; not blocking.
12. **`['fs-profile']` cache audit** — separate narrower pass when convenient.
13. **Phase 4.6** — Resurface pass (Admin folder, feedback affordance, others) per Section 20 sweep.
14. **Phase 4.7** — Desk rename mechanical pass (~92 strings, ~40 files). Last in Phase 4 per DEC-183.
15. **Phase 5 (NEW)** — Pre-migration cleanup per DEC-180.
16. **Phase 6** — Onboarding fork + Region backfill + Pattern C+ migration window (DEC-175).
17. **Post-migration** — Membership gate (DEC-155); Direct Doors (DEC-179); Stripe Connect.
18. **Engagement scoped-query server function** owed during tiles-5+ area; will retire during Supabase migration.
19. **Field Instrument as first Supabase + Vercel build (DEC-184)** — develops in slow hours alongside Phase 4/5.
20. **May 4 Monthly Sharpening** — Cursor reference sweep; `community-node/docs/migration-research.md` cleanup; DECISIONS.md drift audit + merge session.
21. **NODE-LAB-MODEL.md phase-review note** for Field Service crossing production-shaped (private repo).
22. **Newsletter "The Good News"** — wake dormant accounts.
