# LAUNCH-CHECKLIST.md

> Pre-pilot checklist items. Mark [x] when shipped.

---

## Field Service Workspace

- [x] Document templates seeded by workspace type
- [x] Documents grouped by client
- [x] Client-required document creation
- [x] One-action Send for Signature with clipboard copy
- [x] Client portal e-sign (document)
- [x] Client portal e-sign (estimate)
- [x] Recall flow (documents + estimates)
- [x] Owner signing (documents + estimates)
- [x] Amendment flow
- [x] Archive toggle
- [x] Send estimate with link + copy
- [x] Currency formatting on all monetary values
- [x] Permit apply_url on creation
- [x] Type-specific People add (Worker, Sub, Client)
- [x] Daily log photo gallery upload
- [x] Project financial ledger
- [x] Mobile optimization pass (44px targets, responsive tables)
- [x] FieldServiceAgent Superagent live
- [x] Agent chat with voice input
- [x] Admin document stats
- [ ] Settings tab walkthrough with Bari
- [ ] View permissions customization
- [ ] Industry presets (DEC-090)

## Harvest Network

- [x] Product tags + payment methods
- [x] Tag filtering on grid
- [x] Geocoding on address save
- [x] Admin marketplace panel
- [ ] Map view (construction gated)
- [ ] Network application flow (construction gated)
- [ ] Two egg sellers onboarded

## Agents & MCP

- [x] MCP v2 circuit test — all 5 tools passing (2026-03-30)
- [x] agentScopedWrite server function built and confirmed
- [x] Mylane write capability confirmed (first agent-created record)
- [x] Agent tier gating infrastructure (subscription_tier on profiles)
- [ ] Base44 publish unblocked
- [ ] Mylane console upgrades deployed (new chat, upload, chips, cards)
- [ ] Field test: full Mylane write flow from mobile
- [x] Server-authoritative user_id on agent writes (DEC-139 — 2026-04-05)
- [x] MylaneNote reminder loop end-to-end verified (agent v2 — 2026-04-16)
- [x] Mylane Agent v2 deployed (DEC-149 — mandatory protocol, hallucination fixed — 2026-04-16)
- [x] Smart routing deployed (DEC-150 — TYPE 1 for views, TYPE 2 for novel — 2026-04-16)
- [x] RemindersCard read path fixed (Creator Only RLS bypass via agentScopedQuery — 2026-04-16)
- [ ] Propagate write capability to all workspace agents

## Team / Playmaker

- [x] Team invite flow working end to end
- [x] Personalized invite landing page
- [x] Claim-first join flow (coaches claim pre-seeded roster spots)
- [x] Onboarding skip for invite-based entry
- [x] Door links for physical-world entry (/door/:slug)
- [ ] Coach Rick field test confirmation
- [ ] Randy league demo + scheduling workflow research

## Mylane / Dark Until Explored

- [x] Mylane as default authenticated landing
- [x] Card vitality dimming (continuous opacity curve)
- [x] Discovery whisper ghost cards (proximate spaces)
- [x] Auto/Manual gradient tracking (frequency data)
- [ ] AgentChat dispatch wired for mylane-user-message
- [x] Dead code cleanup (19 dead files deleted, 31 unused imports removed — 2026-04-04)
- [x] Full 13-category audit complete (68 to 87 — 2026-04-04)
- [x] Entity permissions locked to Creator Only (9 entities — 2026-04-04)
- [x] MylaneNote reminders live (2026-04-04)
- [x] Feedback pipeline consolidated to ServiceFeedback (2026-04-04)
- [x] Founding Gardener observation live via MCP (2026-04-04)
- [x] Mylane shell containment audit complete (2026-04-15)
- [x] All 6 known escape points closed (Sessions A+B+C — 2026-04-15)
- [x] Overlay system refactored to OV constant (DEC-146 Living Feet — 2026-04-15)
- [x] Philosophy + Support surfaced inside shell (2026-04-15)
- [x] Newsletter inline form in Account overlay (2026-04-15)
- [x] Backdrop click-to-close on all overlays (2026-04-15)
- [x] 4 dead pages deleted: SpokeDetails, ShapingTheGarden, CategoryPage, Search (~1,170 lines — 2026-04-15)
- [x] Agent gated to R&D allowlist (DEC-147 — 2026-04-15)
- [x] Viewport pinch-to-fit fix (fontSize 16 — 2026-04-15)
- [x] Overlay containment polish (useBottomInset, all 9 overlays — 2026-04-16)
- [x] CommandBar pinned to viewport bottom (position: fixed — 2026-04-16)
- [x] Hidden fields filter for TYPE 2 renders (33 fields — 2026-04-16)
- [x] DrillView tab routing (TYPE 1 view param wired — 2026-04-16)
- [x] Agent loading state cleared on RENDER parse (2026-04-16)
- [x] Home Canvas spec reviewed and shelved (DEC-151 — 2026-04-16)
- [ ] Walkthrough verification: today's fixes in live app (Doron)
- [ ] ClaimBusiness + BusinessEditDrawer cleanup (co-presence model)
- [ ] Footer removal or strip (content now inside shell)
- [ ] Admin per-space reframe

## Platform

- [ ] LLC/EIN paper filing (SSN missing, needs re-fax)
- [ ] Stripe Connect integration
- [ ] Newsletter Issue 1 sent
- [ ] Admin panel audit
- [ ] Community Pass / Recess Pass audit
- [x] SpaceSpinner 3D variants (drum + cover flow) with friction physics
- [x] Semantic Tailwind migration complete (208 files)
- [x] Three themes live (Gold Standard, Cloud, Fallout with CRT effects)
- [x] Frequency Station background playback (Pip-Boy radio model — provider at root, MediaSession, mini-player)
- [x] Frequency Station studio + library (ownership model, submission wizard, admin workbench)
- [x] Frequency Station end-to-end submission→transform→deliver flow
- [x] Frequency Station notification system (in-app bell + dropdown — polish pending)
- [ ] Frequency Station playlist polish (auto-advance working, shuffle + queue UI pending)

## Round 1 — Business Foundation

- [x] **Phase 1: Schema foundation** (2026-04-22) — Business/User/FS family field additions, AuditLog entity, TEMPLATE.js canonical pattern (cfdcdb9, 1bb8936)
- [x] **Phase 1.5: additional fields** — User.is_legacy_user, User.legacy_grace_until, Business.subscription_exempt
- [x] **Phase 2: Reparenting machinery** — reparentBusiness + migrationHelpers server fns + phase-2-production-migration.js Node script (cc051a9, 210d087)
- [x] **Phase 2: Production migration applied** (2026-04-23) — Mycelia/LocalLane/TCA created, Recess reparented, Red Umbrella promoted from Bari FS, Doron sandbox archived, Bari flagged legacy, 9 AuditLog rows reversible
- [ ] **Phase 2: Bari workspace smoke test** (Doron, when convenient — log in from real device and confirm his FieldService workspace loads normally with new `business_id` link)
- [ ] **Phase 3: Business switcher** — UI for parent-child tree; LocalLane exempt badging; Red Umbrella as peer, not under Mycelia
- [ ] **Phase 4: Desk rename + business-scoped rendering** (DEC-160, DEC-156) — "Field Service" becomes "Desk"; tools filter by `business_id` when in a business context
- [ ] **Phase 5: Membership gate** — $9/mo per entity per DEC-155, with LocalLane exempt
- [ ] **Phase 6: Onboarding fork** — invite-based vs. cold entry; Dan Sikes claim path for his orphan Business
- [ ] **Round 2: Stripe Connect** (prerequisite for real money flow + for populating `legacy_grace_until` on all `is_legacy_user: true` users per DEC-159)

### Critical path to real users (next blockers)

1. Phase 3 business switcher (unblocks Doron's multi-business UI)
2. Phase 4 Desk rename + business-scoped rendering
3. Phase 5 membership gate (prerequisite for charging)
4. Round 2 Stripe Connect (prerequisite for real money)

---
