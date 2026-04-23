# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-23 (Bari-prep shipped — Red Umbrella workspace ready for 2026-04-24)

## Current Focus

Bari-prep session closed. Red Umbrella Services LLC workspace is ready for Bari's 2026-04-24 morning meeting: two attorney-drafted templates loaded, two-section template UI shipped, preview modal live, legal disclaimer on system templates, branded letterhead composited from his Business record. Phase 3 (business switcher) is the next queued session — and it will resolve the multi-business template visibility edge case Doron hit during dogfood, where the current-business signal lives in a client-side state rather than a proper switcher.

## Active Architecture

- **Two-tier FSDocumentTemplate (DEC-163):** `business_id: null` = system template (the 4 LocalLane-drafted Oregon lien templates), `business_id` set = business-scoped user template. Visible to all users with access to that Business. Client-side partition (DEC-140 pattern). Two-section UI on the Documents tab.
- **Business-first branding (DEC-164):** `buildMergeData` reads Business record first with FSProfile fallback. 14 new `{{business_*}}` merge fields including `logo_url` and `banner_url`. Legacy `{{company_*}}` preserved. Branded letterhead (logo + name + tagline) in DocumentDetail and preview modal.
- **Template preview before commit (DEC-165):** Every template card opens a preview modal. Business branding composited from viewer's Business; per-client fields shown as `[Bracketed]` placeholders. "Use this Template" routes to existing CreateDocumentFlow with `initialTemplate` prop.
- **Legal disclaimer on system templates:** banner in preview modal + small italic footer on rendered FSDocument. User-owned templates do not receive it (user's content, user's responsibility).
- **Bari's Red Umbrella templates loaded (DEC-166):** `69ea7974c5f30ff25c860702` (General Construction Contract) and `69ea797533163127a73aeef3` (Subcontractor Agreement). Both `business_id: 69ea5590481b7e15af7216b6`, `profile_id: 69baba55a6b9cca0c7d5700b`, `is_system: false`. Loaded via `load-bari-templates.js` + new `migrationHelpers.create_fs_document_template` action. Source typos preserved verbatim.
- **Business.video_url field:** plumbing only. Paste URL in BusinessSettings → saves. No render yet; deferred to BusinessProfile redesign session.
- **Schema-conformance audit protocol (DEC-167):** any write-mutation session now audits the full payload against the entity schema, not just the diff. Surfaced after a Phase-4-era dormant `merge_fields` JSON.stringify bug hit Doron's dogfood.
- **Production Business tree (Phase 2 state preserved):** Mycelia, LLC → LocalLane / TCA / Recess; Red Umbrella is Bari's peer (not nested). 9 AuditLog rows from Phase 2 remain reversible.
- **Mylane Agent v2, smart routing, shell containment:** all live, no regressions from Bari-prep (UI-additive only, no agent-path changes).
- **Health score:** 87/100 (unchanged).

## What Just Shipped (2026-04-23, Bari-prep session)

1. **Track A — feature surface (e3ba53a):** Two-section template UI, preview modal, legal disclaimer constant + render sites, `buildMergeData` business-first with logo/banner in merge fields, branded letterhead in DocumentDetail, Business.video_url input + PROFILE_ALLOWLIST + jsonc reference copy.
2. **Track B cleanup (4c6ceda):** Strip italic `*(Note: ...)*` implementer annotations from loaded content. Source .md files keep annotations for repo docs; Base44 content is clean.
3. **Track B load (c3a7091):** `base44-prompts/assets/` with both contract markdown files, `migrationHelpers.create_fs_document_template` action, `load-bari-templates.js` idempotent loader.
4. **Bari's templates applied** via migration script (dry-run + --apply). 2 `FSDocumentTemplate` records, 2 `AuditLog` rows.
5. **Regression fix (9e349be):** `TemplateEditor.handleSave` — dropped `JSON.stringify` wrapper on `merge_fields` write. Pre-existing dormant bug from Phase 4 (DEC-085), surfaced by dogfood. Not a Track A regression.
6. **DEC-163 through DEC-167 recorded.** Three seedlings and three tech-debt entries in private repo.

## Known Issues

1. **DECISIONS.md drift** between Spec-Repo and community-node — pre-existing, not touched today. Future: dedicated audit/merge session.
2. **FSDocumentTemplate `rls.update` creator-only** — business teammates can't edit each other's templates; Bari can't edit the two migration-loaded templates (service role is creator). Workaround: fresh "+ Custom" copy. Logged in private/TECH-DEBT.md.
3. **Phase 2 FieldServiceProfile + User `security.update: true` with no RLS** — wide-open by design for Phase 2 migration. Phase 5 (membership gate) re-tightens. Logged in private/TECH-DEBT.md.
4. **Base44 auto-push behavior** — "File changes" commits to main with uninformative messages. DEC-162 mitigates in prompts; expect it any entity-change session.
5. **Overlay z-indices hardcoded** — z-50/55/60, refactor when third stacked-overlay scenario appears.
6. **ClaimBusiness route** — page going away (co-presence model). BusinessEditDrawer still generates claim URLs.
7. **FrequencyLibraryContext refactor** — prop drilling for favorites/queue should become context provider.
8. **Footer still renders** on non-MyLane pages — purpose limited to unauthenticated public.
9. **Phase 2 migration scripts eslint-env node placement bug** — cosmetic, no runtime impact. Logged in private/TECH-DEBT.md.

## Dogfood Observations (2026-04-23, Bari-prep)

- **Template creation "+ Custom" flow** hit the `merge_fields` stringify bug — fixed tonight (9e349be).
- **Multi-business template visibility** edge case: current-business signal on the Documents tab reads `profile.business_id` as the source of truth. Works for Doron's single-business today; resolves cleanly in Phase 3 when the business switcher becomes the authoritative source.
- **Admin impersonation need surfaced twice** during dogfood: verifying Bari's workspace view, pasting a video URL to Bari's settings. Neither was possible without the feature. Logged as seedling (private/SEEDLING-TRACKER.md).
- **Doron's test template** from dogfood (in his own Business workspace, `business_id: 69ea4eb5d5b69f39bbcc404f`) deleted post-session.

## In Flight

None. Bari-prep shipped in a settled state.

## Active Blockers

None.

## Upcoming Priorities

1. Bari workspace live check from Bari's device (2026-04-24 morning)
2. Phase 3 — business switcher (UI, handles parent-child rendering + exempt badging + current-business source of truth)
3. Phase 4 — Desk rename + business-scoped rendering (DEC-160, DEC-156)
4. Phase 5 — membership gate ($9/mo per entity, DEC-155) — also re-tightens FSProfile/User `rls.update`
5. Round 2 — Stripe Connect (prerequisite for real money flow + legacy grace population per DEC-159)
6. BusinessProfile website-shaped redesign (video render becomes live at that point)
7. DECISIONS.md drift audit + merge session
8. Field testing compass (Doron, ongoing)
9. Newsletter "The Good News" — wake dormant accounts
