# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-26 (Phase 4 warmup shipped; DEC-182 ratified; Cursor retired from canonical docs; Field Instrument seed committed)

## Current Focus

**Phase 4 warmup is complete.** Today's session planned light (one warmup prompt to sync community-node's drifted context layer with Spec-Repo canonical) and compounded into four shipped pieces of work: the warmup itself, the architectural call ratifying single-source-with-sync as policy (DEC-182), the Cursor-retirement cleanup of canonical docs in both repos, and the long-overdue commit of `FIELD-INSTRUMENT-SEED.md` to the private repo. Pedrom thread brought current with v0.1 protocol outcome (live test 2026-04-24 — both AIs hit posture, framing-bias failure mode flagged). Phase 4 main work has not started; this was foundation-laying. Phase 4 main work is next.

## Active Architecture

- **Phase 3 closed (2026-04-25)** — all eight community-node Phase 3 builds (1-patch, 2, B, C, D, E, F, G, H) shipped. No regressions today. Cockpit-native business switcher, directory visibility, profile editors, Desk vocabulary, cockpit + workspace centering, structured `service_area`, multi-category `subcategories[]`, SDK wrap in `src/api/`, Desk tile icon, workspace centering — all live, all stable.
- **Multi-machine infrastructure live (DEC-181)** — Mac mini primary, MacBook Pro 2017 secondary. Both machines on `~/Documents/GitHub/`, all four repos on SSH. Today's full session ran from the mini.
- **Single-source documentation policy ratified (DEC-182, today)** — Spec-Repo is canonical for project documentation. Tool-specific repos reach across via explicit `~/Documents/GitHub/Spec-Repo/...` paths and do not mirror reference docs. The narrow exception is `community-node/context/` (PROJECT-BRAIN, ACTIVE-CONTEXT, SESSION-LOG), which is mirrored as a lean read-only copy refreshed on scheduled drift-sync passes (one Hyphae prompt per Phase or per Monthly Sharpening). `CLAUDE.md` and `AGENTS.md` are community-node-native, not mirrors.
- **Cursor retired from canonical docs (today)** — Spec-Repo/.cursorrules deleted; PROJECT-BRAIN.md drops Cursor IDE from AI Tools, drops Base44+Cursor from the tooling-stack progression, removes the "Cursor Prompt Format" section, removes `.cursorrules` from the File Map. community-node/context/PROJECT-BRAIN.md re-synced from canonical. Six remaining Cursor references identified across `WORKFLOW.md` (~17), `ARCHITECTURE.md:156`, `README.md:261/265/277`, `checklists/farm-program-launch.md:60` — queued for May 4 Monthly Sharpening sweep. `MAP-VIEW.md:9` and `DECISIONS.md:93` (DEC-099 historical) intentionally left as append-only history.
- **Schema-conformance discipline (DEC-167 / DEC-177 / DEC-178)** — code-level Base44 entity changes ship paired with Base44 prompts; write-path conformance enforced (not just field-shape); audit dashboard schema before any write-path change.
- **Mylane Agent v2, smart routing, shell containment** — all live, no regressions.
- **Health score:** 87/100 (unchanged).

## Migration Plan (DEC-175 — Pattern C+)

- **Trigger:** Phase 6 (Region foundation backfill window). One fragility cost, not two.
- **Target stack:** Supabase + Vercel.
- **Sandbox:** `lanecountyrecess.com`. `locallane.app` stays on Base44 during construction.
- **AI:** retire all 8 Base44 agents at migration. Build a single warm-presence companion fresh, designed from Bari's "AI tour guide" framing.
- **Seam hardening:** Build F bundled the SDK wrap into `src/api/`, validated end-to-end (multi-category was the first consumer). `updateProfile()` wraps the server-function write path per DEC-177.
- **Reference:** `community-node/docs/migration-research.md` (commit `78fc8c7`; flagged for cleanup).

## Field Instrument (sibling product, seed-stage, separate from LocalLane)

- **FIELD-INSTRUMENT-SEED.md committed to `private/` root today** (`e00bcda`). 244 lines. Originally drafted 2026-04-12 in Mycelia thread, lived on the laptop until today.
- **v0.1 protocol live-tested 2026-04-24** with Pedrom Rejai. Successful — both AIs hit the posture the protocol asked for. Pedrom's AI flagged a framing-bias failure mode in the protocol — the single most valuable output of v0.1.
- **Standalone LLC under Mycelia LLC**, sibling to LocalLane. LocalLane remains the primary plant.
- **Companion artifacts** (HYPHAE briefings, SELF-PORTRAIT-TEMPLATE, PARTICIPATION-GUIDELINES) remain on laptop; commit alongside `alibi-protocol` GitHub repo setup once moved to mini.
- **Next:** post-meeting follow-up email to Pedrom (drafted, two variants); Pedrom face-to-face when his Junction City client visit dates land; v0.2 of protocol when bandwidth allows.

## Multi-Machine Infrastructure (DEC-181)

- **Primary:** Mac mini (more powerful, eventual Clawbot host).
- **Secondary:** MacBook Pro 2017 (mobility, field visits to Bari).
- **Both machines:** GitHub Desktop, Claude Desktop, Claude Code (Hyphae) v2.1.119, Node.js v24.15.0 with `~/.npm-global` prefix, SSH keys to github.com/withdoron, all four repos at `~/Documents/GitHub/` (community-node, Spec-Repo, private, ephraim-games), all remotes SSH.
- **Discipline:** pull as the first action of any session, push after every commit. Now load-bearing for multi-machine.
- **Lesson recorded today:** `git rm` stages deletions but the Edit tool leaves modifications unstaged. Always stage all modifications via `git add` before commit. Manifested as a two-commit Spec-Repo split (`4cb0e3d` deletion + `6638c29` modifications) when intent was a single commit.

## What Just Shipped (2026-04-26)

1. **Phase 4 warmup — context layer drift sync (`fe9fad9` community-node):** CLAUDE.md, AGENTS.md, PROJECT-BRAIN, ACTIVE-CONTEXT, SESSION-LOG synced to Spec-Repo canonical; broken `@`-imports fixed; Hyphae three-gardener model and DEC-167/177/178/181 references added.
2. **`.cursorrules` deletion (`4cb0e3d` Spec-Repo).**
3. **PROJECT-BRAIN Cursor retirement + DEC-182 (`6638c29` Spec-Repo).**
4. **community-node Cursor + stale-files cleanup (`b7b1a19` community-node):** PROJECT-BRAIN re-synced from canonical; `PUNCH-PASS-AUDIT.md` and `DEC-025-ENTITY-PERMISSIONS.md` removed.
5. **FIELD-INSTRUMENT-SEED.md committed (`e00bcda` private):** 244 lines at private repo root.

## Strategic Clarifications (still in force from 2026-04-25)

- **Phase 3.5 (Direct Doors) deferred to post-migration** per DEC-179.
- **Phase 5 (Pre-Migration Cleanup)** between Phase 4.5 and Phase 6 per DEC-180.
- **Payment infrastructure deferred to post-migration.** Membership gate (DEC-155) and Stripe Connect both move to the post-migration window.

## Field Service Node — Production-Shaped

Bari is the first external paying user. Multi-category classification working end-to-end after Build F. Score nudged to ~95/100. Node now considered production-shaped given paying-user dependency. **NODE-LAB-MODEL.md phase-review note still owed** (private repo, deliberate review pending — flagged again today, not updated as a side-effect of ship-it).

## Known Issues

1. **`community-node/docs/migration-research.md`** — supposed to be deleted post-DEC-175; still present as of `78fc8c7`. Carryover from 2026-04-25.
2. **DECISIONS.md drift** between Spec-Repo and community-node — pre-existing, not touched today. Future: dedicated audit/merge session.
3. **Stray Cursor references in Spec-Repo** outside today's PROJECT-BRAIN scope — `WORKFLOW.md` (~17 occurrences), `ARCHITECTURE.md:156`, `README.md:261/265/277`, `checklists/farm-program-launch.md:60`. Queued for May 4 Monthly Sharpening sweep.
4. **Persistent Base44 SDK 404 console spam** on every page load — source unknown, not user-visible, root-cause investigation seedling logged.
5. **Duplicate `DC` key React warning** in Radix Select (likely state-code Select) — cosmetic seedling logged.
6. **`Business.categories` field empty in Base44** (DEC-176) — added in error during Build F verify, left in place pending Phase 5 cleanup.
7. **FSDocumentTemplate `rls.update` creator-only** — workaround documented; logged in private/TECH-DEBT.md.
8. **Phase 2 FieldServiceProfile + User `security.update: true` with no RLS** — wide-open by design for migration; post-migration membership gate re-tightens. Logged.
9. **Base44 publish blocker** — escalation request `95a004a0` still open. Affects DEC-115 deploy of Mylane write capability.
10. **Base44 auto-push behavior** — DEC-162 working agreement mitigates; expect "File changes" commits any entity-change session.
11. **Overlay z-indices hardcoded** — z-50/55/60; refactor when third stacked-overlay scenario appears.
12. **ClaimBusiness route + BusinessEditDrawer claim URLs** — co-presence model retires this; cleanup pending (Phase 5 candidate per DEC-180).
13. **Footer renders on non-MyLane pages** — limited purpose now.

## Organism Milestones (this session)

- **DEC-182 ratified (2026-04-26):** Single-source documentation policy now codified architecture, not just convention. Demonstrated in same-session sync from Spec-Repo to community-node mirror.
- **First explicit Spec-Repo → community-node downstream sync flow** (`b7b1a19`): the policy DEC-182 establishes was applied in the same prompt that established it.
- **FIELD-INSTRUMENT-SEED.md finally on disk in version control** after two weeks of bouncing between laptop outputs and chat context.

## In Flight

None. Session closed in a settled state.

## Active Blockers

None.

## Upcoming Priorities

1. **Phase 4 main work** — Desk implementation rename + business-scoped rendering (DEC-160, DEC-156); MyLaneSurface hook reordering; MyLaneDrillView latent bug fix; eslint-plugin-react-hooks exhaustive-deps enforcement; resurfacing buried surfaces per DEC-173.
2. **Phase 4.5** — Seedlings (Admin Impersonation, post-migration welcome flow).
3. **Phase 5 (NEW)** — Pre-migration cleanup per DEC-180.
4. **Phase 6** — Onboarding fork + Region backfill + Pattern C+ migration window (DEC-175).
5. **Post-migration** — Membership gate (DEC-155); Direct Doors (DEC-179); Stripe Connect.
6. **May 4 Monthly Sharpening** — Cursor reference sweep across `WORKFLOW.md`, `ARCHITECTURE.md`, `README.md`, `checklists/farm-program-launch.md`.
7. **Field Instrument** — when Pedrom responds: GitHub repo `alibi-protocol` setup with Pedrom as collaborator; companion artifacts moved from laptop to mini and committed alongside seed doc.
8. **Standalone cleanup walk session** (Doron's notes-taking pass when bandwidth allows).
9. **NODE-LAB-MODEL.md phase-review note** for Field Service crossing production-shaped (private repo, deliberate review).
10. **Carryovers** — `community-node/docs/migration-research.md` cleanup; DECISIONS.md drift audit + merge session.
11. **Newsletter "The Good News"** — wake dormant accounts.
