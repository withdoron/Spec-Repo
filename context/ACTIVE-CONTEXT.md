# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-23 (Phase 2 shipped — Mycelia tree anchored)

## Current Focus

Round 1 Phase 2 complete. Mycelia, LLC now exists as a production Business record with LocalLane (exempt), TCA, and reparented Recess as children. Bari's Red Umbrella promoted from his FieldServiceProfile to a first-class Business with all brand fields preserved. Doron's test FS sandbox archived. Dan Sikes Construction preserved as unclaimed orphan. Bari flagged `is_legacy_user: true`. 9 AuditLog rows written, all reversible. Phase 3 (business switcher) queued for a future session.

## Active Architecture

- **Production Business tree:** `Mycelia, LLC` (hidden, `subscription_exempt: true`, `parent_business_id: null`) → `LocalLane` (exempt) + `The Camel Academy` + `Recess` (reparented). `Red Umbrella` (Bari) exists as a peer of Mycelia, not a child. Other originals (Spetzler Designs, NH systems, Danny Sikes Construction) untouched.
- **Per-entity $9 pricing (DEC-155):** every LocalLane entity — user personal account OR public-facing business — pays $9/month from its own books. LocalLane-the-brand is exempt because charging itself is circular. Hidden holding entities (Mycelia, LLC) don't count. Supersedes DEC-115.
- **Business-as-scoped-peer (DEC-156):** tools (Desk/Clients/Finance/Events) stay top-level with a `business_id` filter instead of physically nesting. Same UX, cheaper to build.
- **Legacy user grace pattern (DEC-159):** `is_legacy_user` boolean marks pre-v4 users. `legacy_grace_until` datetime stays null until Round 2 billing goes live — then a migration will populate it to `billing_live_date + 30 days` for everyone flagged. Bari flagged today.
- **Migration machinery:** `reparentBusiness` + `migrationHelpers` Base44 server fns (in main, published). Shared `MIGRATION_SECRET` gate + `asServiceRole` for all I/O. Admin-role gating dropped because Base44 API-key auth doesn't populate `caller.role`. AuditLog `user_id` = Doron per DEC-139 server-authoritative identity.
- **DEC-095 clarified:** `security.update` and `rls.update` are two independent layers in Base44 entities. Both must be open (or the `rls.update` key absent) for `asServiceRole` writes to land. Previously documented as a one-layer fix; today's migration surfaced the second layer.
- **Base44 working agreement (DEC-162):** applied-directly prompts with four-category confirmation. Report-don't-fix is a standing rule to prevent auto-lint-fix overreach during schema changes.
- **Cockpit library (DEC-152):** spinner + compass variants via `ll_cockpit` preference. Doron field-testing compass.
- **Mylane agent v2 (DEC-149), smart routing (DEC-150), shell containment:** all live, no regressions from Phase 2 migration (backend-only, no UI touched).
- **Health score:** 87/100 (unchanged from 2026-04-17).

## What Just Shipped (2026-04-23)

1. **Phase 1 schema foundation** (cfdcdb9 — 2026-04-22): Business/User/FS family field additions + AuditLog entity + TEMPLATE.js canonical migration pattern.
2. **Phase 1.5 schema additions:** User.is_legacy_user, User.legacy_grace_until, Business.subscription_exempt.
3. **reparentBusiness + migrationHelpers + phase-2-production-migration.js** (cc051a9 + 210d087): reparent/rollback, 8-action helper surface with idempotency and dry-run.
4. **Phase 2 production migration applied:** 4 new Business records (Mycelia/LocalLane/TCA/Red Umbrella), Recess reparented, Bari FS profile linked via business_id, Doron sandbox archived, Bari marked legacy, 9 AuditLog rows.
5. **Reports to private repo** (177e915): PHASE-2-DRY-RUN-2026-04-23 + PHASE-2-POSTMIGRATION-REPORT-2026-04-23.
6. **DEC-155 through DEC-162 + DEC-095 amendment** (this session).
7. **Cleanup:** MIGRATION_SECRET removed from Base44 env and local .env.

## Known Issues

1. **DECISIONS.md drift** between Spec-Repo and community-node — Spec-Repo skipped 146-148; both have conflicting DEC-152. Today's new decisions went to Spec-Repo only. Future: dedicated audit/merge session to reconcile.
2. **Base44 auto-push behavior** — "File changes" commits to main with uninformative messages, occasional unrelated lint fixes bundled with intended schema changes. DEC-162 mitigates in prompts; not eliminated.
3. **Overlay z-indices hardcoded** — z-50/55/60, refactor to stack-based when third scenario appears.
4. **DevLab physics sliders don't affect compass** — compass uses CSS transitions. Harmless.
5. **BEARING label weighting** in compass chrome row — soft polish flag for field test.
6. **ClaimBusiness route** — page going away (co-presence model). BusinessEditDrawer still generates claim URLs.
7. **FrequencyLibraryContext refactor** — prop drilling for favorites/queue should become context provider.
8. **Footer still renders** on non-MyLane pages — purpose limited to unauthenticated public.
9. **Riley TeamMember `$69.00`** data issue — fix in Base44 dashboard.
10. **Dan Sikes Construction orphan** — `owner_user_id: null`, unclaimed. Onboarding fork (Phase 6) should offer the existing record when Dan signs in instead of creating new.

## In Flight

None. Phase 2 shipped in a settled state. Awaiting Doron's device-level Bari workspace verification as a final smoke test.

## Active Blockers

None.

## Upcoming Priorities

1. Bari workspace smoke test (Doron, when convenient)
2. Phase 3 — business switcher (UI, handles parent-child rendering + exempt badging)
3. Phase 4 — Desk rename + business-scoped rendering
4. Phase 5 — membership gate ($9/mo per entity)
5. Round 2 — Stripe Connect (prerequisite for real money flow + legacy grace population)
6. DECISIONS.md drift audit + merge session
7. Field testing compass (Doron, ongoing)
8. Walkthrough verification of 2026-04-16/17 fixes in live app
9. Newsletter "The Good News" — wake dormant accounts
