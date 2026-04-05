# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-05 (MyLane reminder loop bug fix)

## Current Focus

MylaneNote reminder loop diagnosed and fixed. Server-side user_id enforcement shipped (DEC-139). MyLane instructions updated. Bad record healed. Read path confirmed working. Write path pending Base44 publish.

## Active Architecture

- **Agent writes:** DEC-139 — identity fields (user_id, owner_id) are server-authoritative on all agentScopedWrite calls. Agents cannot set these fields. Query/write asymmetry: queries need user_id for scoping, writes forbid it.
- **Theme system:** Semantic tokens (98.5% migrated). Three themes: Gold Standard, Cloud, Fallout.
- **Spinner:** 3D variant architecture. Friction + mass physics. Ratchet snap.
- **Team space (7 tabs):** Home, Playbook, Schedule, Roster, Messages, Photos, Settings.
- **Query optimization:** staleTime 5min global default (DEC-130). getMyLaneProfiles batches 6 queries into 1.
- **Agent architecture:** DEC-107 enforced — space agents use agentScopedQuery only. DEC-136 — entity permissions default Creator Only.
- **Auth:** Single source of truth — AuthContext seeds React Query cache, refreshUser() syncs both.
- **Error isolation:** WorkspaceErrorBoundary wraps each workspace drill view.
- **Health score:** 87/100 (from 68 on 2026-04-04 audit)

## What Just Shipped (2026-04-05)

1. Server-side fix: agentScopedWrite unconditionally stamps user_id/owner_id from auth context (DEC-139)
2. MyLane instruction updates: removed user_id from write payloads, added query/write asymmetry docs
3. Healed MylaneNote record: "Text Kate from LinkedIn" now visible on Home feed
4. Confirmed: RemindersCard renders, client query path works, MylaneNote entity + permissions correct

## Pending (Blocked on Base44 Publish)

- Server fix live in Base44 runtime
- MyLane instruction updates live in Base44 runtime
- End-to-end write test: create new reminder → display → complete → verify status flip

## Known Issues for Next Session

1. **Date parsing bug:** MyLane parsed "tomorrow" as 2026-04-19 instead of 2026-04-06. One-line instruction fix likely.
2. **MCP user_id fallback:** Weaker path than auth.me(). Future hardening candidate.
3. **Broader user_id flow audit:** Today's cascade suggests more ambiguous identity paths may exist.

## Upcoming Priorities

1. Base44 publish — unblocks reminder write-path verification
2. Coach Rick demo — foundation is stone, Team space production-ready
3. Date parsing bug fix in MyLane instructions
4. Ephraim Pip-Boy design session
5. Newsletter "The Good News" to wake dormant accounts
6. Bari visit for feedback chip demo
7. Remaining polish: StepIndicator extraction, loading states, shared EmptyState, accessibility
