# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-16 (Mylane Agent v2 + Shell Polish + Smart Routing)

## Current Focus

Mylane agent v2 is live with mandatory tool-call-first protocol — agent hallucination fixed. Smart routing deployed: "show me" queries mount real workspace components via TYPE 1 RENDER. Shell polish complete: fixed CommandBar, overlay containment, hidden fields, tab routing, loading state. Home Canvas spec designed, reviewed by Hyphae, and shelved — existing architecture handles it with smarter agent routing.

## Active Architecture

- **Mylane agent v2 (DEC-149):** Mandatory 4-step protocol: Classify → Execute → Verify → Respond. "Never lie" rule. Failure logging to ServiceFeedback. 2 tools only (agentScopedQuery + agentScopedWrite).
- **Smart routing (DEC-150):** TYPE 1 RENDER for workspace views (mounts real components via MyLaneDrillView), TYPE 2 RENDER_DATA for novel queries only (with HIDDEN_FIELDS filter).
- **Mylane shell containment:** Complete. 9 overlays. useBottomInset hook (HEADER_HEIGHT, MINI_PLAYER_HEIGHT, COMMAND_BAR_HEIGHT). CommandBar pinned to viewport bottom (z-9997).
- **Overlay system:** All overlays use OverlayContainer with dynamic bottomInset. Stacked overlays (BusinessProfile z-50, Network z-55, Recommend z-60) have scroll containment.
- **Agent gating:** MYLANE_AGENT_ALLOWLIST gates agent UI. Doron-only during R&D.
- **Frequency Station:** Phase 3 complete. Mini-player (z-9998) coordinates with CommandBar via useBottomInset.
- **Theme system:** Semantic tokens (98.5% migrated). Three themes: Gold Standard, Cloud, Fallout.
- **Team space:** Production-ready. readTeamData server function (DEC-140).
- **Health score:** 87/100

## What Just Shipped (2026-04-16)

1. **Overlay containment polish:** useBottomInset hook, all 9 overlays fixed, stacked overlay scroll containment
2. **Mylane Agent v2:** Full instruction rewrite, 26 entity tools removed, hallucination fixed
3. **Smart routing:** TYPE 1 for views, TYPE 2 for novel queries, drillview tab routing fixed
4. **Shell polish:** CommandBar pinned to viewport, hidden fields filter (33 fields), loading state fix
5. **RemindersCard read path:** agentScopedQuery bypass for Creator Only RLS, staleTime 30s
6. **Home Canvas spec:** Designed, reviewed, shelved (DEC-151 — spec review protocol established)
7. **DEC-149/150/151:** Agent v2 protocol, smart routing, spec review protocol

## Known Issues

1. **Overlay z-indices hardcoded** — z-50/55/60, refactor to stack-based when third scenario appears
2. **ClaimBusiness route** — page going away (co-presence model), BusinessEditDrawer still generates claim URLs
3. **FrequencyLibraryContext refactor** — prop drilling for favorites/queue should become context provider
4. **Footer still renders** on non-MyLane pages — purpose limited to unauthenticated public pages
5. **Riley TeamMember data** — parent_user_ids shows "$69.00" (data issue, fix in Base44 dashboard)

## Upcoming Priorities

1. Walkthrough verification of today's fixes in live app
2. CLAUDE.md compression pass (Vercel pattern — passive context, dense index)
3. Agent Soul Seed Protocol (shared template for all space agents)
4. ClaimBusiness + BusinessEditDrawer cleanup (co-presence model)
5. Footer removal or strip
6. Admin per-space reframe
7. Newsletter "The Good News" — wake dormant accounts
