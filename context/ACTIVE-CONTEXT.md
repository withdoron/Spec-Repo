# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-17 (Cockpit Library shipped — spinner + compass)

## Current Focus

Cockpit library shipped. Spinner (preserved) + compass (new) available via `ll_cockpit` localStorage preference and data-cockpit DOM attribute, mirroring theme plumbing. Picker in AccountOverlay, Dark Until Discovered. Doron field-testing compass. Changing focus — ready for next direction.

## Active Architecture

- **Cockpit library (DEC-152):** `ll_cockpit` preference with `data-cockpit` attribute + MutationObserver. SpaceSpinner's VARIANT_MAP is the registry. `resolveVariant()` picks variant from cockpit + theme + reduced-motion. No React provider until cockpit #3 proves the pattern needs formalization.
- **Compass variant:** Chrome row (affordance only: `BEARING` left, degrees right), dial strip (stations slide under fixed needle, active station lit in place via accent color + size + weight), orientation arc (macro position with scaled active dot).
- **Mylane agent v2 (DEC-149):** Mandatory 4-step protocol: Classify → Execute → Verify → Respond. 2 tools only (agentScopedQuery + agentScopedWrite).
- **Smart routing (DEC-150):** TYPE 1 RENDER for workspace views, TYPE 2 RENDER_DATA for novel queries only.
- **Mylane shell containment:** Complete. 9 overlays. useBottomInset hook. CommandBar pinned to viewport bottom (z-9997).
- **Overlay system:** OverlayContainer with dynamic bottomInset. Stacked overlays (BusinessProfile z-50, Network z-55, Recommend z-60).
- **Agent gating:** MYLANE_AGENT_ALLOWLIST gates agent UI. Doron-only during R&D.
- **Frequency Station:** Phase 3. Mini-player (z-9998) coordinates with CommandBar via useBottomInset.
- **Theme system:** Semantic tokens (98.5% migrated). Three themes: Gold Standard, Cloud, Fallout. Now orthogonal to cockpit.
- **Team space:** Production-ready. readTeamData server function (DEC-140).
- **Health score:** 87/100

## What Just Shipped (2026-04-17)

1. **Cockpit library plumbing (c44bb21):** ll_cockpit localStorage, data-cockpit attribute, pre-paint in main.jsx, useCockpit hook, resolveVariant function.
2. **Compass variant (c44bb21):** renderCompass in VARIANT_MAP, COMPASS_BEARINGS map, OrientationArc SVG helper, Fallout monospace rule in index.css.
3. **Cockpit picker (c44bb21):** Two-state cycle button in AccountOverlay next to theme.
4. **Compass polish v1 (b4d187e):** Removed redundant readouts (hid active station on strip, removed global dot indicator row under compass).
5. **Compass polish v2 (ad04eb1):** Course correction — restored active station on strip with color-in-place treatment, simplified chrome row to affordance-only framing.
6. **DEC-152/153/154:** Cockpit Library Pattern, Color-in-Place Over Out-of-Band Readout, Iterate on the Live Surface.

## Known Issues

1. **Overlay z-indices hardcoded** — z-50/55/60, refactor to stack-based when third scenario appears.
2. **DevLab physics sliders don't affect compass** — compass uses CSS transitions, not the friction loop. Harmless. Low-priority cleanup would gate DevLab visually by cockpit.
3. **BEARING label in compass chrome row** may be slightly over-weighted relative to degrees readout after active name moved back to strip. Soft polish flag for field test.
4. **ClaimBusiness route** — page going away (co-presence model), BusinessEditDrawer still generates claim URLs.
5. **FrequencyLibraryContext refactor** — prop drilling for favorites/queue should become context provider.
6. **Footer still renders** on non-MyLane pages — purpose limited to unauthenticated public pages.
7. **Riley TeamMember data** — parent_user_ids shows "$69.00" (data issue, fix in Base44 dashboard).

## In Flight

None. Cockpit library paused in shipped, field-testable state. Ready for next direction from Doron.

## Active Blockers

None related to this work.

## Upcoming Priorities

1. Field testing compass (Doron)
2. Walkthrough verification of 2026-04-16 fixes in live app
3. CLAUDE.md compression pass (Vercel pattern — passive context, dense index)
4. Agent Soul Seed Protocol (shared template for all space agents)
5. ClaimBusiness + BusinessEditDrawer cleanup (co-presence model)
6. Footer removal or strip
7. Admin per-space reframe
8. Newsletter "The Good News" — wake dormant accounts
