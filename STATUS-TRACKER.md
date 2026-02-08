# LocalLane Status Tracker

> Complete view of where we are and what's ahead.
> Updated: 2026-02-01

---

## ✅ SHIPPED (Done)

### Core Platform
- [x] Directory with search, filters, categories
- [x] Events page with filter pills
- [x] Business profiles (services, pricing, contact, tabs)
- [x] Category pages
- [x] Full-text search with sort options
- [x] Gold Standard dark theme (consistent everywhere)
- [x] Mobile-responsive layout

### MyLane (User Dashboard)
- [x] Phase 1 — 8 components, real data, progressive depth
- [x] Time-aware greeting with first name
- [x] Punch Pass balance badge
- [x] Happening Soon with filter pills
- [x] New in Community business cards
- [x] Your Recommendations (hides when empty)
- [x] Upcoming Events from RSVP data (hides when empty)
- [x] Discover category tiles

### RSVP System
- [x] RSVP entity in Base44
- [x] useRSVP hook (status, attendee count, mutations)
- [x] "I'm Going" / cancel button in EventDetailModal
- [x] Celebration UX ("You're in!") + auto-close
- [x] Cache invalidation (modal + MyLane refresh)
- [x] Attendee count display

### User Settings
- [x] Settings page (/Settings)
- [x] Display name, email (read-only), phone
- [x] Home Community dropdown
- [x] Account info (member since, tier)
- [x] Profile photo placeholder
- [x] Settings link in user dropdown + mobile menu

### Admin Panel
- [x] Business management table with filters
- [x] User management (list + detail drawer + admin controls)
- [x] Concerns panel (private feedback review)
- [x] Config system (event types, age groups, durations, accessibility, networks)
- [x] Platform settings
- [x] Sidebar navigation with sections
- [x] "Users (soon)" label removed from Admin sidebar

### Events & Business Management
- [x] Event CRUD (create, edit, delete, cancel, duplicate)
- [x] Recurring events with day-of-week selection
- [x] Event images, Punch Pass pricing, multi-ticket
- [x] Staff management (add, invite, roles, auto-linking)
- [x] Business onboarding wizard
- [x] Recommendation system (Nods, Stories, Vouches)
- [x] Private Concerns routing to admin

### Code Quality
- [x] 42-finding code audit (3 phases complete)
- [x] Dark theme cleanup across all components
- [x] Unused file removal
- [x] Legacy system cleanup

---

## 🔧 QUICK FIXES (Small, can ship anytime)

| Fix | Effort | Notes |
|-----|--------|-------|
| Landing page routing (logged-in → MyLane) | 15 min | Route "/" to MyLane for authenticated users |
| Autofill white flash on dark inputs | 15 min | CSS `autofill:` style overrides |
| Test user full_name in published env | 5 min | Log in as test user → Settings → save name |
| Update DECISIONS.md with DEC-028 through DEC-031 | 15 min | Copy from decisions-additions.md |
| Replace STRIPE-CONNECT.md with v2 | 10 min | Copy STRIPE-CONNECT-v2.md over existing |

---

## 🚧 NEXT UP (Before Pilot)

### 🚀 Launch Roadmap

Three phases, in order. Each unlocks the next.

#### Phase 1: Farm Program (Content-First Launch) — NOW
Real businesses, real events, no code changes needed. The platform is ready for this today.

- [ ] Onboard 3-5 farm/food businesses (create accounts, set up profiles)
- [ ] Each business creates 2-3 real events (farm tours, workshops, markets)
- [ ] Facebook post driving community awareness (posted 2026-02-02)
- [ ] First real users browse, RSVP, attend
- [ ] Collect feedback from businesses and attendees
- [ ] Identify friction points before adding complexity
- **Why first:** Proves the platform works with real people. Generates the activity data the organism will eventually reflect. Builds the relationships that Community Pass revenue share depends on.
- **What's needed:** Doron's time onboarding businesses. Zero code.
- **Success criteria:** 3+ businesses with events, 10+ real RSVPs, feedback collected
- **Checklist:** See `checklists/farm-program-launch.md`

#### Phase 2: Organism Phase 1 — The Seed
Single build session. Makes the platform feel alive before money flows.

- [ ] Create `useVitality` hook (calculate from existing MyLane data)
- [ ] Create `CommunityOrganism.jsx` component (CSS + SVG animation)
- [ ] Place in MyLane GreetingHeader alongside greeting and punch badge
- [ ] 3-5 visual states: Seed → Sprouting → Growing → Thriving → Resting
- **Spec:** ORGANISM-CONCEPT.md (private repo)
- **Dependencies:** None — uses data already queried in MyLane
- **Note:** Decision filter for all features: "Does this make the organism more alive?" (DEC-029)

#### Phase 3: Stripe Payments (Community Pass Model)
Real money. Gated behind legal checklist.

- [ ] Phase A: Platform Setup — Stripe account config, products, webhook URL
- [ ] Phase B: Business Tier Subscriptions — real billing for business tiers
- [ ] Phase C: Business Connect Onboarding — link bank accounts
- [ ] Phase D: Community Pass Subscriptions — user membership billing
- [ ] Phase E: Direct Purchases — `on_behalf_of` checkout flow
- [ ] Phase F: Revenue Share Distribution — monthly pool distribution
- **Spec:** STRIPE-CONNECT.md (revised 2026-02-02, aligned with DEC-028)
- **Legal:** Must complete legal checklist before Phase B (see LEGAL-RESEARCH.md)
- **Priority:** HIGH — blocks real money flowing through platform

### Security Audit Phase 3
- [ ] Base44 permission warnings addressed
- [ ] Auth flow edge cases tested
- [ ] Data access patterns verified
- **Priority:** HIGH — must complete before pilot

### Mobile Polish Pass
- [ ] Test all pages on actual mobile devices
- [ ] Touch targets sized appropriately (44px minimum)
- [ ] Scroll behavior smooth on all sections
- [ ] Bottom nav / PWA considerations
- **Priority:** MEDIUM — important for pilot UX

---

## 📋 PLANNED (Post-Pilot Enhancements)

### MyLane Phase 2
- [ ] Follow/save businesses ("My Businesses")
- [ ] User preference storage
- [ ] Category engagement tracking
- [ ] Personalized recommendations based on activity
- **Spec:** MYLANE.md Phase 2 section

### MyLane Phase 3
- [ ] Member tier features (Market Basket, saved filters)
- [ ] BuildLane integration (preference quiz)
- [ ] Advanced discovery algorithms
- **Spec:** MYLANE.md Phase 3 section

### User Tiers
- [ ] Free → Member upgrade flow
- [ ] Member benefits (Market Basket, meal planning, saved filters)
- [ ] Tier display in UI
- [ ] Payment flow for membership
- **Spec:** USER-TIERS.md

### Notifications
- [ ] Phase 1: Email notifications (RSVP confirmation, 24hr reminder, weekly digest)
- [ ] Phase 2: PWA setup + push notifications
- [ ] Phase 3: SMS via Twilio (if needed)
- **Decision:** Deferred — will build post-pilot

### Ownership Badges
- [ ] "Locally Owned" badge on business cards/profiles
- [ ] "Local Franchise" badge
- [ ] Admin toggle for badge assignment
- **Spec:** Concept in CURSOR-QUICK-WINS.md

### Profile Photo Upload
- [ ] Image upload to Base44 or external storage
- [ ] Avatar display in nav, Settings, recommendations
- [ ] Crop/resize functionality
- **Status:** Stubbed with "coming soon" placeholder

---

## 🔮 VISION (Future Features)

### User Groups (Phase 4)
- [ ] Private groups for friends/family coordination
- [ ] Group chat + private events
- [ ] Shared awareness (see when group members attend public events)
- [ ] Group organism pulse (infrastructure for creature visualization)
- [ ] Admin moderation tools
- **Spec:** USER-GROUPS-v1.md (private repo)
- **Status:** Planned — Phase 4, after Stripe/payments

### Silver Barter Network
- [ ] Phase 1: Badge and filter (near-term, mostly exists)
- [ ] Phase 2: Silver Network dedicated view
- [ ] Phase 3: Bullion dealer integration (live spot price)
- [ ] Phase 4: Community features (stories, meetups, education)
- **Spec:** SILVER-BARTER-CONCEPT.md

### Meal Planning / Market Basket
- [ ] Recipe → ingredient → local business matching
- [ ] Weekly meal plan generation
- [ ] Shopping list with Punch Pass integration
- **Spec:** MEAL-PLANNING-CONCEPT.md

### Multi-Region Expansion
- [ ] Independent community nodes per region
- [ ] User "home community" drives data (already in Settings)
- [ ] Visitor mode for browsing other regions
- [ ] Region-specific businesses, events, trust networks

### Community Punch Fund
- [ ] Gifting mechanism for Punch Passes
- [ ] Fund for underserved youth
- [ ] Donation flow through platform
- **Status:** Concept only

### Event Node Enhancements
- [ ] Partner Node template improvements
- [ ] Sync reliability hardening
- [ ] Multi-event series support

---

## 📄 SPEC REPO FILES

| File | Purpose | Status |
|------|---------|--------|
| ARCHITECTURE.md | System architecture overview | ✅ Current |
| CURRENT-STATE.md | Implementation status | ✅ Updated 2026-02-01 |
| STATUS-TRACKER.md | Project status and roadmap | ✅ New 2026-02-01 |
| STYLE-GUIDE.md | Design system reference | ✅ Current |
| TIER-SYSTEM.md | Business tier definitions | ✅ Current |
| DECISIONS.md | Architecture decision records | ✅ Current |
| MYLANE.md | MyLane spec (3 phases) | ✅ Current |
| USER-TIERS.md | User tier definitions | ✅ Current |
| ADMIN-ARCHITECTURE.md | Admin panel structure | ✅ Current |
| INTEGRATION-PLAN.md | Stripe + platform integration | ✅ Current |
| STRIPE-CONNECT.md | Stripe Connect spec | ✅ Current |
| UX-WALKTHROUGH-2026-02-01.md | Chrome audit findings | ✅ Current |
| CURSOR-QUICK-WINS.md | Quick fix prompts | ✅ Complete |
| CURSOR-MYLANE-PHASE1.md | MyLane build prompt | ✅ Complete |
| CURSOR-RSVP-SYSTEM.md | RSVP build prompt | ✅ Complete |
| CURSOR-SETTINGS-AND-ADMIN-USERS.md | Settings + Admin Users prompt | ✅ Complete |

Strategy and concept docs maintained in private repository.

---

## Session Log — 2026-02-07

### 2026-02-07 — Punch Pass → Joy Coins Migration Complete (Full Stack)

**What shipped:**
- P1: All user-facing text migrated across 10 files (ec940cb)
- P2-P4: Terms.jsx legal text, JoyCoinsTransfer.jsx deleted, PunchPass.jsx page deleted, variable renames, field mapping with dual-write (3f6a211, 695e2dc)
- Base44 field mapping + remaining cleanup: Layout.jsx, Admin.jsx route, EventEditor dual-write, server function docs (695e2dc)
- Server function migration (community-node): setPunchPassPin → setJoyCoinPin, validatePunchPass → validateJoyCoins, handleEventCancellation and searchHubMember rewritten for JoyCoins entities (d026fb7)
- Server function migration (events-node): checkInWithPunchPass → checkInWithJoyCoins, CheckIn.jsx updated (3fa5ab5)
- pin_hash field added to JoyCoins entity in Base44
- DEC-041: Joy Coins non-transferable, expired coins fund Community Scholarship Pool
- Legal update: CARD Act / EFTA risk analysis added to LEGAL-RESEARCH.md (Blackburn v. ClassPass, July 2025)
- DECISIONS.md updated with DEC-041

**PunchPass entities status:**
- PunchPass, PunchPassTransaction, PunchPassUsage entities remain in Base44 as archived data
- Zero code reads or writes them — fully dead
- No migration of data needed (test/demo data only)

**Decisions made:**
- DEC-041: Non-transferable coins, scholarship pool for expired coins
- Stripe path: LLC + EIN → Stripe account → personal bank temporarily → swap to business bank when ready
- Business tier subscriptions can start before business bank account opens
- CARD Act gift certificate disclaimer recommended for Terms of Service (discuss with counsel)

**Next session priorities:**
- Mobile nav + device testing
- Settings display name bug
- Business onboarding wizard simplification (Events archetype only)

---

## Phase 1 Language Audit — Joy Coins & Community Pass

- [x] Full codebase search for "punch" — replaced with Joy Coin / Community Pass terminology
- [x] UI audit: no references to "punch card", "punch pass", or "stored value"
- [x] Database fields: documented legacy field names with mapping comments
- [x] Terms of Service: verified Community Pass + Joy Coin language (not stored value)
- [x] User-facing copy audit: MyLane, Business Dashboard, Event Detail
- [x] Confirm "Joy Coins" used consistently (not "tokens", "credits", "punches", "tickets")
- [ ] Privacy Policy: verify membership framing (not prepaid balance) — still needs review

---

## Session Log — 2026-02-05

**Focus:** Pricing strategy deep dive and decision locking

**Decisions locked (4 new):**
- DEC-032: Community Pass pricing — $49 base / $39 additional / 12 coins each
- DEC-033: Joy Coin pool model — coins not people, household shares pool
- DEC-034: Business tier pricing — Free / $79 Standard / $149 Partner
- DEC-035: Earn-Your-Pass — kids work farms/markets to earn access

**Research completed:**
- Crabtree's Simple Numbers: price for 10-15% profit, pay yourself market rate, protect margins
- ClassPass anti-model: extracts value, $6-15/visit to studios, 94% first-timers, <2% conversion
- 1Pass Eugene: 11K passes, 69K scans, $9.50/scan avg — validates scan-based pool in Eugene
- Eugene gym pricing: climbing $75-91/mo adult, gymnastics $80-120/mo per kid, trampoline $25/mo
- Off-peak strategy: 30-50% empty capacity weekdays, incremental revenue pitch to businesses
- Operating costs modeled: ~$5,450/mo, breakeven ~200 memberships

**Key insight:** Memberships are coin allocations, not person assignments. Simplifies everything — billing, UX, no "who's on the plan" tracking.

**Docs requiring update:**
- COMMUNITY-PASS.md → full rewrite (private repo)
- DECISIONS.md → add DEC-032 through DEC-035
- STRIPE-CONNECT.md → update product tables with real prices
- TIER-SYSTEM.md → lock business pricing
- LAUNCH-CHECKLIST.md → check off pricing items, add new sections

**Next up:**
- Apply doc updates to repos
- Founding member rate decision
- Marketing deck (two versions: families + businesses)
- Launch buildout roadmap

---

## Session Log — 2026-02-07

**2026-02-07 — Microbusiness Archetype Spec**
- Completed comprehensive microbusiness-node.md archetype spec
- Six sub-types: Home Food (cottage law), Home Services (handyman), Personal Services, Property Management & Rentals, Creative/Maker, Local Delivery
- Oregon legal research: cottage food ($50K cap, SB 643), handyman ($1K threshold, CCB licensing), Eugene business license requirements
- Introduced $9/month Microbusiness tier concept between free users and $79 Standard businesses
- Property management sub-type includes Crabtree financial model and guest-to-community bridge for Airbnb hosts
- Decision logged as DEC-039

**2026-02-07 — Nonprofit Archetype Expansion**
- **Focus:** Nonprofit archetype expansion and Eugene landscape research
- **Decisions locked (1 new):** DEC-040: Nonprofit pricing — 50% discount ($39 Standard, $79 Partner)
- **Spec updates:** nonprofit-node.md major update — 8 sub-types, nonprofit pricing tiers, expanded data model, Burrito Brigade and Ophelia's Place test cases, implementation phases, Organism connection
- **Research completed:** NONPROFIT-LANDSCAPE-EUGENE.md new research doc for private repo — 2,443 org ecosystem, detailed profiles, outreach priority list, revenue projections
- Eugene nonprofit density: 2,443 orgs, 17,711 employees, $2B revenue
- Burrito Brigade deep dive: 55 Little Free Pantries, 3 programs, founding partner candidate
- Ophelia's Place: youth empowerment across Lane & Linn counties
- FOOD for Lane County: Youth Farm connects to Earn-Your-Pass
- Grassroots Connect philosophy alignment: "What we practice at a small scale sets the patterns for the whole system"
- **Key insight:** Nonprofits are already interconnected through informal referral networks. LocalLane doesn't create connections — it makes them visible. Burrito Brigade's 55 pantry locations IS the Organism.

---

## Session Log — 2026-02-05 (Build Sprint)

### 2026-02-05 — Business Dashboard v2 Build Sprint (Builds 1-6)

**What shipped:**
- Build 1: AccessWindow entity created in Base44, useAccessWindows and useBusinessRevenue hooks
- Build 2: BusinessDashboard.jsx refactored from widget stack to 5-tab navigation (Home, Joy Coins, Revenue, Events, Settings) with Home tab showing real stat cards and narrative line
- Build 3: AccessWindowManager + AccessWindowModal — full CRUD for Joy Coin access hours with empty states, pause/resume, delete confirmation, pricing tip
- Build 4: RevenueOverview — monthly pool share, DEC-037 estimated payout banner, collapsible "How Pool Share Works", by-event table, by-day bar chart, empty state
- Build 5: Skipped — Events tab already functional with EventEditor and Joy Coin controls
- Build 6: BusinessSettings — business profile card, subscription status with tier display, StaffWidget integration, dev tier override for testing

**Tier gating confirmed working:**
- Basic tier: Lock + upgrade prompt on Joy Coins and Revenue tabs
- Standard tier: Full access to all tabs
- Dev tier override: One-click switching in Settings (dev only)

**New files:**
- src/hooks/useAccessWindows.js
- src/hooks/useBusinessRevenue.js
- src/components/dashboard/AccessWindowManager.jsx
- src/components/dashboard/AccessWindowModal.jsx
- src/components/dashboard/RevenueOverview.jsx
- src/components/dashboard/BusinessSettings.jsx

**New entity:**
- AccessWindow (Base44) — business_id, day_of_week, start_time, end_time, coin_cost, capacity, is_active, label

**Decisions applied:** DEC-036 (capacity per-person), DEC-037 (show estimated revenue), DEC-038 (network comparison min 10)

**Build 8 backlog:**
- Wire CheckInMode overlay (pre-existing bug — state set but component never rendered)
- Remove 6 unused imports from BusinessDashboard.jsx
- Optional: upgrade bar chart to recharts

**Remaining:** Build 7 (Surface Integration) and Build 8 (Polish + Audit) deferred to next session.

---

## Session Log — 2026-02-06

**Focus:** Build Protocol creation + Business Dashboard v2 strategy spec

**Shipped today:**
- BUILD-PROTOCOL.md — universal 13-phase build sequence for all features
- BUSINESS-DASHBOARD-v2.md — complete strategy spec replacing INTEGRATION-PLAN.md
- DEC-036: AccessWindow capacity per-person (not per-family)
- README.md updated with new file references

**Key decisions:**
- Build Protocol established as the standard for how every feature ships
- Business Dashboard restructured around 5 tabs: Home (Story), Joy Coins (Controls), Revenue (Proof), Events (Engine), Settings (Profile)
- AccessWindow entity designed for peak/off-peak Joy Coin hour management
- Capacity set to per-person (family sizes vary)
- Dashboard build estimated at 19-27 hours across 8 sequential builds

**Next up:**
- Build 1: AccessWindow entity + hooks + security (Foundation)
- Decide: show estimated revenue before Stripe is live?
- Decide: minimum business count for network comparison data

---

## Session Log — 2026-02-03

## Session Log — 2026-02-03 (continued)

**Shipped today (afternoon session):**

**Total items shipped 2026-02-03: 63**
- Joy Coins Phases 1-5 complete
- Event Creator Controls complete
- Business-level Joy Coins complete
- Punch Pass → Joy Coins migration started
- UI polish and filters complete
- Language audit backlog documented

*Bug Fixes:*
41. AdminSidebar "Punch Pass" removed (language audit item)
42. Settings/Household "saved automatically" clarification added

*Event Creator Controls (LAUNCH-CHECKLIST Phase 4):*
43. Joy Coins UI already existed in EventEditor.jsx — audit confirmed
44. Added joy_coin_unlimited toggle
45. Added frequency_limit_count + frequency_limit_period (attendance limits)
46. Added adults_only toggle
47. Fixed Joy Coins prefill when editing existing events

*Audits:*
48. Pre-build audit: Mapped EventEditor.jsx structure
49. Post-build audit: Found missing prefill, fixed

*Business-Level Joy Coins:*
50. Added `accepts_joy_coins` field to Business entity (Base44)
51. Added Joy Coins toggle to BusinessDashboardDetail.jsx
52. Added Joy Coins badge to BusinessProfile.jsx
53. Added Joy Coins badge to BusinessCard.jsx (directory listing)
54. Updated FilterModal: "Punch Pass Accepted" → "Accepts Joy Coins"

*Punch Pass Cleanup:*
55. Removed Punch Pass section from EventEditor.jsx (Joy Coins replaces it)

*Audit completed:*
56. Full codebase Punch Pass audit: 24 files, ~140 references identified
57. Language audit backlog created for remaining Punch Pass → Joy Coins migration

*Joy Coins UI Polish:*
58. Compact Joy Coins layout in EventEditor (inline field pairs)

*Joy Coins Filters:*
59. Directory filter: "Accepts Silver" → "Joy Coins", checks `accepts_joy_coins`
60. Events filter: Fixed to check `joy_coin_enabled` (was broken)
61. FilterModal: Updated label and icon to Joy Coins

*Team Identity:*
62. Mycelia + Doron = gardeners tending the Organism. Lane Avatar (mushroom) is the Organism itself.

*Final verification:*
63. Build passed, published to production ✅

**Joy Coins Event Creator Controls status:**
- [x] Network selection (already existed)
- [x] Joy Coin cost setting per event (already existed)
- [x] Capacity controls — joy_coin_spots + joy_coin_unlimited
- [x] Frequency limits — frequency_limit_count + frequency_limit_period
- [x] Refund policy selection (already existed)
- [x] Adults only toggle (bonus)

**Phase 6 still blocked on:**
1. LAUNCH-CHECKLIST Phase 3 Legal items
2. Service role function migration
3. PIN validation server-side

**Next session starts here:**
- [ ] Verify Joy Coins filters work in production (Directory + Events)
- [ ] Test business Joy Coins toggle → badge flow end-to-end
- [ ] Continue LAUNCH-CHECKLIST Phase 4 or move to Phase 5
- [ ] Language audit backlog is documented but deferred (non-blocking)

**Phase 6 blockers (unchanged):**
- Legal review (~$100)
- Service role function migration
- Stripe Connect for payouts

---

## Backlog: Punch Pass → Joy Coins Language Audit

**Status:** Deferred (non-blocking for launch)

Per DEC-028, Joy Coins replaces Punch Pass. Audit found 24 files with ~140 references.

**Priority 1 — User-facing text (5 files):**
- [ ] GreetingHeader.jsx — "Punches" → "Joy Coins"
- [ ] Layout.jsx — "Punch Pass" nav → "Joy Coins" or remove
- [ ] Terms.jsx — Legal text update
- [ ] Support.jsx — "punch-based system" text

**Priority 2 — Field/variable names (5 files):**
- [ ] useUserState.js — punchPass → joyCoins
- [ ] useOrganization.js — canUsePunchPass → canUseJoyCoins
- [ ] EventCard.jsx — remove punch variables
- [ ] EventDetailModal.jsx — remove punch variables

**Priority 3 — Legacy removal (6+ items):**
- [ ] PunchPass.jsx page — delete or redirect
- [ ] pages.config.js — remove PunchPass route
- [ ] functions/setPunchPassPin.ts — migrate to Joy Coins
- [ ] functions/validatePunchPass.ts — replace with Joy Coins
- [ ] AdminUsersSection.jsx — punch balance → Joy Coins balance
- [ ] PunchPass entities — archive or migrate data

---

Shipped today:

**Phase 1 — Data Foundation:**
- JoyCoins entity (balance tracking per user)
- JoyCoinTransactions entity (immutable audit trail)
- JoyCoinReservations entity (RSVP coin holds)
- HouseholdMembers entity (family members for group RSVPs)
- Events entity expanded (10 new Joy Coin fields)
- RSVPs entity expanded (8 new fields: party size, check-in tracking)
- Security rules locked down for all 4 new entities

**Phase 2 — Core Flows:**
8. useJoyCoins hook (balance, transactions, reservations queries)
9. JoyCoinsCard component in MyLane dashboard
10. JoyCoinsHistory page (/my-lane/transactions)
11. RSVP flow: Joy Coin reserve on RSVP, balance check, cost display
12. Cancel flow: refund/forfeit logic per event policy (flexible/moderate/strict)
13. UI: coin badges on EventCard, cost display in EventDetailModal
14. UI: cancel confirmation with refund/forfeit warning for Joy Coin RSVPs

**Phase 3 — Household & Party:**
15. Party size selector in RSVP flow (+/- buttons, max limit)
16. Total Joy Coin cost calculation (partySize × event cost)
17. HouseholdManager component (add/edit/delete family members)
18. Household section in Settings page
19. Household link in MyLane dashboard
20. Optional party composition selector (pick who's going by name)
21. party_composition stored on RSVP when names selected

**Phase 4 — Transfers & Business Tools:**
22. CheckInMode component (attendee list, search, PIN entry)
23. CheckInWidget for business dashboard (upcoming events quick access)
24. Check-in flow: RSVP marked checked_in, reservation redeemed, transaction created
25. JoyCoinsAnalyticsWidget (redemptions, attendance rate, recent activity)
26. Analytics integrated into business dashboard for location/venue archetypes
27. Joy Coin transfer between members (search, send, receive)
28. Transfer UI with search, amount selection, confirmation flow
29. Transfer transactions recorded for both sender and recipient

**Phase 5 — Automation & Polish:**
30. processNoShows.js — batch and single-event no-show processing
31. No-show button in CheckInMode for ended events
32. processMonthlyGrants.js — grant automation (single user, batch, cohort)
33. Monthly grant resets balance (no rollover), tracks expired coins
34. calculateRevenueShare.js — revenue share calculation (75/25 split)
35. Revenue report in admin: per-business breakdown, monthly totals
36. JoyCoinsAdminPanel — admin UI for grants, no-shows, revenue reports
37. Admin sidebar integration under /Admin/joy-coins

**Audit — 2026-02-03:**
38. Claude Code full audit: 46 pass, 9 warn, 0 fail
39. Quick fixes applied: console.error removed, category colors fixed
40. Known limitations documented: atomicity, PIN validation (temporary)

**Joy Coins implementation status:**
- [x] Phase 1: Data Foundation (entities, fields, security)
- [x] Phase 2: Core Flows (RSVP, balance display, transactions, cancel/refund)
- [x] Phase 3: Household & Party (party size selector, household management)
- [x] Phase 4: Transfers & Business Tools (member transfers, check-in mode, analytics)
- [x] Phase 5: Automation & Polish (no-show processing, monthly grants, revenue share)
- [x] Event Creator Controls (UI for businesses to configure Joy Coins on events)
- [ ] Phase 6: Payouts (Stripe Connect — blocked, see dependencies below)

**🎯 NEXT ACTION:** Run Claude Code full codebase review before moving to Phase 6.

Phase 6 (Stripe Connect payouts) is deferred until:
1. Claude Code review complete
2. Phase 3 Legal items resolved (professional legal review, revenue share agreements)

---

## Session Log — 2026-02-01

**Shipped today:**
1. MyLane Phase 1 (8 components, real Base44 data)
2. RSVP system (entity, hook, modal button, celebration UX)
3. UpcomingEventsSection (connected to RSVP data)
4. BusinessCard dark theme fix
5. Grid layout fixes for sparse events
6. User Settings page
7. Admin User Management section
8. RSVP cache invalidation + auto-close
9. Silver Barter concept doc
10. AdminSidebar "Users (soon)" label removed

**Decisions made:**
- Regions = independent nodes, users have "home community"
- Punch Pass stays free tier (drives transaction revenue)
- User Settings captures personal info only, business/tax info goes on Business entity + Stripe Connect
- Notifications deferred to post-pilot

---

## Session Log — 2026-02-02

**Shipped today:**
1. Codebase health audit (44 findings, 31 quick fixes executed)
2. Dark theme compliance fixes (14 violations fixed across 5 files)
3. Dead code cleanup (7 items removed from BusinessOnboarding)
4. Tier hook consistency refactor (3 files using useOrganization())
5. Unused dependencies removed (8 packages)
6. claude.md created for Claude Code sessions
7. claude.md updated with 5 new conventions (DEC-028, toggle knobs, tier hooks, file organization, large file guidance)
8. ORGANISM-CONCEPT.md written (private repo) — north star vision
9. COMMUNITY-PASS.md written (private repo) — membership model
10. STRIPE-CONNECT.md rewritten to align with Community Pass (DEC-028)
11. DECISIONS.md updated with DEC-028 through DEC-031
12. Spec-repo audit completed — all docs checked against north star

**Decisions made:**
- DEC-028: Community Pass replaces stored-value Punch Pass
- DEC-029: Organism Concept adopted as north star
- DEC-030: Private repo for sensitive strategy
- DEC-031: Status indicator color policy for admin/dashboard

**Workflow improvements:**
- claude.md established as bridge between Chat and Claude Code
- Session SOP: plan → build → test → update claude.md → document → push
- Skill creation trigger: flag on second occurrence of a pattern

---

*This tracker is the single source of truth for project status.*
