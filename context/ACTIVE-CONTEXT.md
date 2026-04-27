# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-26 (evening — Phase 4 migration plan in fully-planned state; Section 8 closes all seven open questions; DEC-183 path-walking cadence + DEC-184 alibi-first-Supabase ratified; tomorrow's first move = Phase 4.1 entity prompt)

## Current Focus

**Phase 4 is fully planned, not yet executing.** Morning session shipped the warmup (community-node doc-drift sync, DEC-182, Cursor retirement, Field Instrument seed). Afternoon shipped Hyphae's pre-Phase-4 audit as `PHASE-4-MIGRATION-PLAN.md` (Spec-Repo `1ed1a6c`, 355 lines) — mapped current `community-node/src/` against v4.1's folder model, surfaced seven open questions, proposed sequence 4.1 → 4.2a → 4.2b → 4.3 → 4.4 → 4.5 → 4.6 → 4.7. Evening planning conversation closed all seven questions in Section 8 of the migration plan (`b519ef3` Spec-Repo, +115 lines). Bari's working list captured in new `private/users/bari/` folder structure (`b651c91` private). Two architectural decisions ratified: **DEC-183 (Walk the Path Before Sinking Thought)** — path-walking cadence between sub-phase shipments, smaller steps used between beat large pre-sequenced blocks; **DEC-184 (Greenfield Alibi Project as First Supabase + Vercel Build)** — alibi develops in slow hours alongside Phase 4/5 work, informs Phase 6 LocalLane migration. Tomorrow's first move: Phase 4.1 entity prompt.

## Active Architecture

- **Phase 3 closed (2026-04-25)** — all eight community-node Phase 3 builds (1-patch, 2, B, C, D, E, F, G, H) shipped. No regressions. Cockpit-native business switcher, directory visibility, profile editors, Desk vocabulary, cockpit + workspace centering, structured `service_area`, multi-category `subcategories[]`, SDK wrap in `src/api/`, Desk tile icon, workspace centering — all live, all stable.
- **Phase 4 fully-planned state (2026-04-26 evening)** — `Spec-Repo/PHASE-4-MIGRATION-PLAN.md` (commit `1ed1a6c` audit, `b519ef3` Section 8 amendment) is the canonical reference for what Phase 4 ships and in what order. Section 8's ten subsections close all seven open questions Hyphae's audit raised. Future Phase 4 execution prompts read this plan before being written.
- **Path-walking cadence (DEC-183, today)** — Phase 4 (and future phases) ships in path-walking cadence, not sequential-execution. Each sub-phase ships, gets used, generates real-world friction signal, and only then informs the next sub-phase's spec. Smaller steps used between beat large pre-sequenced blocks shipped back-to-back. Applied immediately: tomorrow's first move (Phase 4.1) is the smallest possible move that earns the right to think about Phase 4.2.
- **Alibi-first Supabase + Vercel (DEC-184, today)** — the greenfield Field Instrument (alibi) project is the first build on Supabase + Vercel, developing in slow hours alongside Phase 4/5 LocalLane work. Operationalizes DEC-183 at platform-migration scale: greenfield isolates platform-learning unknown so Phase 6 LocalLane migration doesn't compound platform + migration mechanics simultaneously.
- **Multi-machine infrastructure live (DEC-181)** — Mac mini primary, MacBook Pro 2017 secondary. Both machines on `~/Documents/GitHub/`, all four repos on SSH. Today's full session ran from the mini.
- **Single-source documentation policy (DEC-182)** — Spec-Repo is canonical for project documentation. Tool-specific repos reach across via explicit `~/Documents/GitHub/Spec-Repo/...` paths and do not mirror reference docs. The narrow exception is `community-node/context/` (PROJECT-BRAIN, ACTIVE-CONTEXT, SESSION-LOG), refreshed on scheduled drift-sync passes (one Hyphae prompt per Phase or per Monthly Sharpening). `CLAUDE.md` and `AGENTS.md` are community-node-native, not mirrors.
- **Cursor retired from canonical docs (2026-04-26 morning)** — Spec-Repo/.cursorrules deleted; PROJECT-BRAIN.md drops Cursor IDE from AI Tools, drops Base44+Cursor from the tooling-stack progression. Six remaining Cursor references queued for May 4 Monthly Sharpening sweep.
- **Schema-conformance discipline (DEC-167 / DEC-177 / DEC-178)** — code-level Base44 entity changes ship paired with Base44 prompts; write-path conformance enforced (not just field-shape); audit dashboard schema before any write-path change. Phase 4.1 (`Business.enabled_spaces`) is the next paired-change.
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

**Morning (warmup + DEC-182 + Cursor retirement + Field Instrument seed):**

1. **Phase 4 warmup — context layer drift sync (`fe9fad9` community-node):** CLAUDE.md, AGENTS.md, PROJECT-BRAIN, ACTIVE-CONTEXT, SESSION-LOG synced to Spec-Repo canonical; broken `@`-imports fixed; Hyphae three-gardener model and DEC-167/177/178/181 references added.
2. **`.cursorrules` deletion (`4cb0e3d` Spec-Repo).**
3. **PROJECT-BRAIN Cursor retirement + DEC-182 (`6638c29` Spec-Repo).**
4. **community-node Cursor + stale-files cleanup (`b7b1a19` community-node):** PROJECT-BRAIN re-synced from canonical; `PUNCH-PASS-AUDIT.md` and `DEC-025-ENTITY-PERMISSIONS.md` removed.
5. **FIELD-INSTRUMENT-SEED.md committed (`e00bcda` private):** 244 lines at private repo root.
6. **Earlier ship-it commits (`9fa698f` Spec-Repo, `e4d0fa2` private):** session-end docs sync for above.

**Afternoon / evening (Phase 4 plan + Bari working list + Section 8 amendment + DEC-183/184):**

7. **`PHASE-4-MIGRATION-PLAN.md` shipped (`1ed1a6c` Spec-Repo):** 355 lines. Hyphae's pre-Phase-4 audit. Maps current `community-node/src/` against v4.1 folder model. Sequence proposal: 4.1 (entity) → 4.2a (root folders) → 4.2b (Businesses-as-folder) → 4.3 (Home → Desk) → 4.4 (preview pulse) → 4.5 (spaces add/remove) → 4.6 (resurface pass) → 4.7 (Desk rename mechanical sweep). Surfaces seven open questions for Mycelia + Doron planning.
8. **`private/users/bari/BARI-WORK-LIST.md` (`b651c91` private):** new per-collaborator folder structure. Captures Bari's named priorities: custom contracts (active), profile polish (shelf), e-sign + invites (shelf), estimates with change-order workflow + dual-direction payment ledger (active alongside Phase 4).
9. **Phase 4 migration plan Section 8 amendment (`b519ef3` Spec-Repo):** +115 lines. Closes all seven open questions:
   - 8.1 Preview pulse silence-by-default, signal-when-warranted.
   - 8.2 Folder placement static for Phase 4; user-determined shortcuts deferred.
   - 8.3 Multiple businesses under Businesses folder; holding hierarchy in data not display.
   - 8.4 Apple Finder spatial model as reference (with explicit borrow/reject lists).
   - 8.5 URL nesting deferred; behavior works as if nested.
   - 8.6 Desk inherits Home's content as-is for Phase 4.
   - 8.7 Discover stays as fifth root folder.
   - 8.8 Admin contextual root in Phase 4 (Thing 1 only); admin lens deferred to 4.5.
   - 8.9 Playmaker contextual root, default placement.
   - 8.10 `enabled_spaces` backfill: `["profile"]` for everyone except Doron's businesses (self-activate) and Bari's (inspect actual usage).
10. **DEC-183 (Walk the Path Before Sinking Thought) + DEC-184 (Greenfield Alibi Project as First Supabase + Vercel Build):** ratified tonight. Recorded in this session-end commit.

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

1. **Phase 4.1 — entity prompt (TOMORROW'S FIRST MOVE).** `Business.enabled_spaces` field added in Base44 (paired prompt + code per DEC-178). Backfill existing businesses per Section 8.10 of `PHASE-4-MIGRATION-PLAN.md`: `["profile"]` for everyone except Doron's businesses (self-activate) and Bari's Red Umbrella (inspect actual usage). Smallest possible move that earns the right to think about Phase 4.2. Mycelia drafts the prompt fresh tomorrow morning. Path-walking cadence per DEC-183 — ship 4.1, use it, then write 4.2a's spec from that use.
2. **Phase 4.2a / 4.2b / 4.3 / 4.4 / 4.5 / 4.6 / 4.7** — sequenced per `PHASE-4-MIGRATION-PLAN.md`. Each ships, gets used, then informs the next. No back-to-back execution.
3. **Phase 4.5** — Seedlings (Admin Impersonation, post-migration welcome flow). Admin lens inside spaces lands here per Section 8.8.
4. **Phase 5 (NEW)** — Pre-migration cleanup per DEC-180.
5. **Phase 6** — Onboarding fork + Region backfill + Pattern C+ migration window (DEC-175). By this point, alibi-first Supabase + Vercel work (DEC-184) has produced platform-learning fluency that informs the migration.
6. **Post-migration** — Membership gate (DEC-155); Direct Doors (DEC-179); Stripe Connect.
7. **Field Instrument as first Supabase + Vercel build (DEC-184)** — develops in slow hours alongside Phase 4/5. When Pedrom responds: GitHub repo `alibi-protocol` setup with Pedrom as collaborator; companion artifacts moved from laptop to mini and committed alongside seed doc; first Supabase + Vercel scaffolding when the relationship and project shape warrants it.
8. **May 4 Monthly Sharpening** — Cursor reference sweep across `WORKFLOW.md`, `ARCHITECTURE.md`, `README.md`, `checklists/farm-program-launch.md`. Plus `community-node/docs/migration-research.md` cleanup (carryover from 2026-04-25). Plus DECISIONS.md drift audit + merge session.
9. **NODE-LAB-MODEL.md phase-review note** for Field Service crossing production-shaped (private repo, deliberate review).
10. **Standalone cleanup walk session** (Doron's notes-taking pass when bandwidth allows).
11. **Newsletter "The Good News"** — wake dormant accounts.
