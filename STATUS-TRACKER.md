# LocalLane Status Tracker

> Complete view of where we are and what's ahead.
> Updated: 2026-02-23

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
- [x] Joy Coins balance badge
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
- [x] Feedback button system (floating button + Admin Panel review)
- [x] Config system (event types, age groups, durations, accessibility, networks)
- [x] Platform settings
- [x] Sidebar navigation with sections
- [x] "Users (soon)" label removed from Admin sidebar

### Events & Business Management
- [x] Event CRUD (create, edit, delete, cancel, duplicate)
- [x] Recurring events with day-of-week selection
- [x] Event images, Joy Coins pricing, multi-ticket
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
| Manager role can't edit events | 30 min | manageEvent needs role differentiation per DEC-011 |
| Duplicate recommendations per user | 15 min | Add uniqueness check in Recommend.jsx |
| Admin panel no recommendation visibility | 30 min | Add recommendations section to admin |
| Co-owner support | TBD | Only one owner_user_id per business currently |

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
- [ ] Place in MyLane GreetingHeader alongside greeting and Joy Coins badge
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

### Phase 3 Security Audit (DEC-025) — COMPLETE ✅
- [x] Entity security lockdown — 24 entities set to Restricted (2026-02-18)
- [x] AdminSettings — service role migration + permissions locked (Phase 3a)
- [x] Location — permissions locked (Phase 3a)
- [x] Spoke — permissions locked (Phase 3a)
- [x] SpokeEvent — permissions locked (Phase 3a)
- [x] CategoryClick — permissions locked (Phase 3a)
- [x] Business — service role migration + permissions locked (Phase 3b)
- [x] Event — service role migration + permissions locked (Phase 3b)
- [x] User — service role migration + permissions locked (Phase 3c)
- [x] RSVP — service role migration + permissions locked (Phase 3d)
- [x] Recommendation — service role migration + permissions locked (Phase 3d)

**Server functions shipped:**
- updateAdminSettings.ts (filter, create, update, check_my_invites, accept_invite)
- updateBusiness.ts (update, add_staff_from_invite, update_counters)
- manageEvent.ts (create, update, delete, cancel)
- updateUser.ts (update_profile — owner-only with field allowlist)
- manageRecommendation.ts (create, update, remove, create_concern — owner-only + admin)
- manageRSVP.ts (rsvp, cancel, checkin, no_show — full Joy Coin orchestration)

**Bugs fixed during audit:**
- Staff auto-link invite cleanup (stale cache invalidation)
- accept_invite action for non-owner users accepting staff invites

**Priority:** HIGH — must complete before pilot

### Mobile Polish Pass — COMPLETE ✅
- [x] Full mobile audit: 103 findings (20 HIGH, 44 MED, 39 LOW)
- [x] All 103 findings resolved across 7 fix phases
- [x] shadcn primitives mobile-safe (dialog, sheet, tabs, drawer, select)
- [x] All tables horizontally scrollable
- [x] Admin sidebar hamburger menu on mobile
- [x] Touch targets 44px minimum (28 files updated)
- [x] Dashboard layouts stack on mobile
- **Completed:** 2026-02-19

### Onboarding — User ✅
- [x] Welcome screen after signup (before MyLane)
- [x] Display name capture ("What should we call you?")
- [x] Network interest capture (toggle cards from admin config)
- [x] Community Pass interest capture (yes / maybe later)
- [x] Data stored on User entity (onboarding_complete, network_interests, community_pass_interest, full_name)
- [x] Skippable — user can go straight to MyLane
- [x] localStorage gate prevents repeat showing
- **Completed:** 2026-02-19

### Network Events (DEC-050)
- [x] Build 1: MyLane "My Networks" toggle section — users follow/unfollow networks
- [x] Build 2: Network-only events — toggle in EventEditor, client-side visibility filtering
- [ ] Build 3: Network info pages at /networks/:slug
- **Spec:** NETWORK-EVENTS-SPEC.md (private repo)
- **Priority:** HIGH — enables network-specific communication for Recess and Harvest launches

### Community Node — Phase 1 Status
- **Phase 1: Platform Readiness** — Core complete ✅
  - User onboarding wizard: shipped
  - Business profile editing: shipped
  - Business card redesign: shipped
  - Newsletter system: shipped (footer, onboarding, post-RSVP, admin)
  - Security audit: 6/6 critical resolved, medium items documented
  - Mobile audit: 103/103 findings resolved
  - Remaining: broken links walkthrough, newsletter platform selection
- **Next up:** Map views for network pages, photo carousel consideration

---

## 🔒 SECURITY HARDENING (Post-Launch)

- [ ] **Admin business management via server function** — Business entity Update is currently "No restrictions" to unblock admin during pilot. Needs server function that checks platform admin role before allowing updates to businesses the admin didn't create. Then tighten Business Update back to "Creator only."
- [ ] **Audit all Base44 entity permissions** — verify Read/Update/Delete settings match intended access patterns for: Business, Event, RSVP, JoyCoins, AccessWindow, NewsletterSubscriber, AdminSettings, User, HouseholdMember, Recommendation
- [ ] **Server-side authorization review** — confirm all write operations go through authenticated server functions with proper role checks, not direct client-side entity updates

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

### Network Posts (DEC-051)
- [ ] Phase 1: Announcements — NetworkPost entity, admin compose UI, network page + MyLane integration
- [ ] Phase 2: Reactions + Comments — NetworkPostReaction + NetworkPostComment entities, interaction UI
- [ ] Phase 3: Polls + Organism Signals — NetworkPostPoll + NetworkPostPollVote entities, engagement tracking
- [ ] Phase 4: Polish + Notifications — unread indicators, admin analytics, UX refinements
- **Spec:** NETWORK-POSTS.md (private repo)
- **Entities:** 5 new (NetworkPost, NetworkPostReaction, NetworkPostComment, NetworkPostPoll, NetworkPostPollVote)
- **Priority:** Post-pilot — build after real users are active
- **Estimated:** 7-8 builds across 4 phases

### MyLane Dynamic Layout
- [ ] Build 1: User state detection + section reordering + hide empty sections
- [ ] Build 2: Network compact mode for returning users
- **Spec:** MYLANE-DYNAMIC-LAYOUT.md (private repo)
- **Priority:** High post-launch — directly improves new user experience
- **Estimated:** 2 builds

### Dashboard Workspaces (DEC-053)
- [ ] Phase 1 Build 1: Team + TeamMember entities, team creation wizard, Dashboard landing shows workspaces
- [ ] Phase 1 Build 2: Playbook + Play entities, coach CRUD, image upload for diagrams
- [ ] Phase 1 Build 3: Play viewer (mobile-first card swiper), PlayView tracking, memorization indicators
- [ ] Phase 1 Build 4: Schedule + Roster tabs, TeamEvent + Attendance entities, invite link
- [ ] Phase 2 Build 5: Team organism pulse, health indicators on dashboard cards
- [ ] Phase 2 Build 6: Sharing and discovery — team in Recess network, public profile
- **Spec:** DASHBOARD-WORKSPACES.md (private repo)
- **Entities:** 7 new (Team, TeamMember, Playbook, Play, PlayView, TeamEvent, TeamEventAttendance)
- **Priority:** High post-launch — Doron is user zero (flag football team)
- **Estimated:** 6 builds across 2 phases

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
- [ ] Shopping list with Joy Coins integration
- **Spec:** MEAL-PLANNING-CONCEPT.md

### Multi-Region Expansion
- [ ] Independent community nodes per region
- [ ] User "home community" drives data (already in Settings)
- [ ] Visitor mode for browsing other regions
- [ ] Region-specific businesses, events, trust networks

### Community Scholarship Fund
- [ ] Gifting mechanism for Joy Coins
- [ ] Fund for underserved youth
- [ ] Donation flow through platform
- **Status:** Concept only

### Event Node Enhancements
- [ ] Partner Node template improvements
- [ ] Sync reliability hardening
- [ ] Multi-event series support

---

## 📍 ROUTES & PAGES

### Main Pages

| Route | Component | Status |
|-------|-----------|--------|
| `/` | Home.jsx | ✅ Done |
| `/MyLane` | MyLane.jsx | ✅ Done |
| `/Events` | Events.jsx | ✅ Done |
| `/Directory` | Directory.jsx | ✅ Done |
| `/Search` | Search.jsx | ✅ Done |
| `/BusinessProfile/:id` | BusinessProfile.jsx | ✅ Done |
| `/CategoryPage/:slug` | CategoryPage.jsx | ✅ Done |
| `/PunchPass` | PunchPass.jsx (redirects to Joy Coins) | ✅ Done |
| `/CheckIn` | CheckIn.jsx | ✅ Done |
| `/Recommend/:id` | Recommend.jsx | ✅ Done |
| `/Settings` | Settings.jsx | ✅ Done |

### Business Dashboard

| Route | Component | Status |
|-------|-----------|--------|
| `/business-dashboard` | BusinessDashboard.jsx | ✅ Done |

### Admin Panel

| Route | Component | Status |
|-------|-----------|--------|
| `/Admin` | Admin.jsx | ✅ Done |
| `/Admin/businesses` | AdminBusinessTable | ✅ Done |
| `/Admin/concerns` | AdminConcernsPanel | ✅ Done |
| `/Admin/users` | AdminUsersSection | ✅ Done |
| `/Admin/settings` | AdminSettingsPanel | ✅ Done |
| `/Admin/events/*` | ConfigSection | ✅ Done |
| `/Admin/networks` | ConfigSection | ✅ Done |

---

## 📄 SPEC REPO FILES

| File | Purpose | Status |
|------|---------|--------|
| ARCHITECTURE.md | System architecture overview | ✅ Current |
| STATUS-TRACKER.md | Project status and roadmap | ✅ New 2026-02-01 |
| STYLE-GUIDE.md | Design system reference | ✅ Current |
| TIER-SYSTEM.md | Business tier definitions | ✅ Current |
| DECISIONS.md | Architecture decision records | ✅ Current |
| MYLANE.md | MyLane spec (3 phases) | ✅ Current |
| USER-TIERS.md | User tier definitions | ✅ Current |
| ADMIN-ARCHITECTURE.md | Admin panel structure | ✅ Current |
| BUILD-PROTOCOL.md | Universal build sequence | ✅ Current |
| STRIPE-CONNECT.md | Stripe Connect spec | ✅ Current |
| MAP-VIEW.md | Network page map view spec | ✅ New 2026-02-20 |

Strategy and concept docs maintained in private repository.

---

## Session Log

| Date | Summary |
|------|---------|
| 2026-02-23 | Category Architecture (DEC-055): Phases 1-2 shipped — categoryData.jsx rewritten, useCategories() hook, 12 consumers migrated. Pilot fixes: onboarding category save, optional event type, RSVP auto-close, dashboard RSVP counts, admin drawer save persistence. Roles testing: Tests 1-5 passed. Staff search blocked (Base44 User entity server-side lookup). Team Management feature-guarded as Coming Soon. |
| 2026-02-22 | Full repo audit (spec-repo + private repo). DEC-054 Play Trainer spec written from user research with boys. DASHBOARD-WORKSPACES-IMPLEMENTATION.md written — reconciles DEC-053 workspace model + DEC-054 Play Trainer into single build-ready spec. Audit findings cataloged: 8 files with stale Punch Pass terminology, ARCHITECTURE.md and README.md most stale, ENTITY-SYSTEM.md candidate for archive. |
| 2026-02-21 | Pre-launch cleanup, pricing reset, spec work. Business Dashboard: Coming Soon states, Founding Member, Community Pass interest. DEC-051 (Network Posts), DEC-052 (Pricing Reset), MyLane Dynamic Layout spec. Greeting display_name fix. |
| 2026-02-20 | Major build session: 10+ items shipped. Newsletter system complete (footer capture, post-RSVP prompt, onboarding sync, admin section). Business profile editing (Settings + admin drawer). Business card redesign (vertical, clickable). Security: Business/AccessWindow/Location writes migrated to server functions, entity permissions locked. Full codebase audit (56 findings, 6 critical resolved). Console.log cleanup, toggle knob fixes. |
| 2026-02-19 | User onboarding wizard shipped (3 steps: welcome, network interests, community pass interest). MyLane "My Networks" section with toggle cards. Mobile audit: 103 findings resolved. Network-only events (DEC-050 Build 2). Onboarding data visible in admin user drawer. Session crashed mid-build. |

### Session Log — 2026-02-23

**Focus:** Category architecture consolidation, pilot role testing, bug fixes

**Shipped:**
1. DEC-055 Phase 1: categoryData.jsx rewritten with 5 curated main categories, useCategories() hook created
2. DEC-055 Phase 2: 12 consumer files migrated to useCategories() hook (Group A: 3 Base44 entity readers, Group B: 9 direct importers)
3. BusinessSettings: Grouped category dropdown with section headers + save persistence fix
4. BusinessEditDrawer: Same grouped dropdown pattern + save persistence
5. main_category added to PROFILE_ALLOWLIST in updateBusiness.ts
6. Onboarding category save: main_category included in Business.create() payload
7. Event Type made optional (removed required validation from EventEditor)
8. RSVP auto-close: modal auto-closes after 2 seconds + Done button
9. Dashboard Home tab: RSVP count per event in Upcoming Events list
10. Dashboard Events tab (EventsWidget): RSVP count per event card
11. Business Settings profile card: shows actual category label instead of archetype fallback
12. Staff search: migrated from direct User.filter() to server function (search_user_by_email action)
13. Dashboard nav link: useUserStaffBusinesses hook + Layout.jsx shows Dashboard for staff/co-owners
14. associated_businesses: populated on invite acceptance and direct staff add in updateBusiness.ts
15. Team Management section: feature-guarded as Coming Soon in BusinessSettings

**Roles testing results:**
- Test 1 (Owner creates business): ✅ Pass
- Test 2 (Owner edits business): ✅ Pass
- Test 3 (Owner creates event): ✅ Pass
- Test 4 (Owner sees RSVPs): ✅ Pass (after fix)
- Test 5 (Admin edits any business): ✅ Pass (after fix)
- Test 6 (Co-owner access): ⏸️ Blocked — server-side User entity lookup returns empty
- Test 7 (Unauthorized user blocked): ✅ Pass (per code audit — data-scoped, server functions check auth)

**Key discovery:** Base44 User entity is base44.auth on client side (src/api/entities.js line 9: export const User = base44.auth). Server-side access via base44.asServiceRole.entities.User confirmed in docs but filter/list returning empty in practice. Needs Base44 support investigation.

**Bugs found and fixed:**
- Onboarding category not saving (main_category missing from create payload)
- BusinessSettings category dropdown was flat list (switched to grouped with SelectGroup/SelectLabel)
- BusinessSettings category save not persisting (main_category missing from PROFILE_ALLOWLIST)
- BusinessEditDrawer same two issues (flat list + save not persisting)
- Staff search 403 (User.filter called client-side by non-admin — migrated to server function)
- RSVP modal required manual close
- Dashboard didn't show RSVP counts per event

**Decisions made:**
- DEC-055: Category Architecture Consolidation (Active — Phases 1-2 complete)
- Event Type made optional for pilot (businesses don't always fit standard types)
- Team Management feature-guarded (staff search blocked by Base44 User entity access)
- CategoryClick counts to be reset when Phase 3 runs (no real data exists)

**Blockers:**
- Staff search: Base44 asServiceRole.entities.User.filter() and .list() not returning users despite docs confirming the API. Blocks co-owner and staff invite flow. Feature-guarded for pilot.

**Technical debt noted:**
- search_user_by_email uses User.list() + client-side find (won't scale past ~100 users)
- 3 locations still vulnerable to case-sensitive email matching (validateJoyCoins.ts:42, BusinessEditDrawer.jsx:275, BusinessEditDrawer.jsx:319)
- PunchPass references remain in AdminUsersSection.jsx line 59 and useUserState.js line 23
- deleteBusinessCascade.js uses direct entity calls without server-side auth

**Next session priorities:**
- Business formation + launch strategy (Topic 3 from today — not reached)
- Resolve Base44 User entity lookup with Base44 support
- DEC-055 Phases 3-5 when real businesses exist
- Remove debug logging from updateAdminSettings.ts search_user_by_email

---

### Session Log — 2026-02-22

**Focus:** Play Trainer spec (DEC-054), Dashboard Workspace implementation spec, full repo audit

**Shipped:**
1. PLAY-TRAINER-SPEC-v2.md — full 6-phase spec from user research with Doron's flag football team
2. DASHBOARD-WORKSPACES-IMPLEMENTATION.md — reconciled DEC-053 + DEC-054 into build-ready spec
3. Full repo audit: reviewed all files in spec-repo and private repo
4. Audit report generated with findings, priorities, and cleanup plan
5. DEC-054 added to DECISIONS.md
6. SEEDLING-TRACKER.md updated with Play Trainer as active node
7. NODE-PLAYBOOK.md updated with Play Trainer entry

**Decisions made:**
- DEC-054: Play Trainer — 6-phase build, 8 entities, Team workspace type
- Playbook entity dropped in favor of side field on Play (Offense/Defense as first-level organizer)
- DEC-053 entity model superseded by DEC-054's richer model where they conflict
- Dashboard Workspace build sequence: Foundation 1 (engine) → Foundation 2 (team + roster) → Team Builds 1-5

**Audit findings (for future cleanup):**
- Punch Pass terminology in ~8 files (ARCHITECTURE.md, README.md, .cursorrules, MYLANE.md, ADMIN-ARCHITECTURE.md, ORGANISM-ARCHITECTURE-EVOLUTION.md, STATUS-TRACKER.md)
- ARCHITECTURE.md needs major refresh (pre-Node Lab sync patterns, stale entity schemas)
- README.md needs rebuild (stale status sections, missing file references)
- ENTITY-SYSTEM.md candidate for archive (superseded by DEC-053 workspace model)
- MYLANE.md behind built reality (missing onboarding wizard, networks, newsletter)
- ADMIN-ARCHITECTURE.md missing Newsletter section, still has Punch Pass references
- .cursorrules and claude.md need DEC updates and reference fixes

**Next session priorities:**
- Priority 2 from audit: Terminology sweep (Punch Pass → Joy Coins across all spec-repo files)
- Priority 3: Stale doc refreshes (README.md, ARCHITECTURE.md, MYLANE.md)
- Or: Begin Foundation Build 1 (workspace engine) if Doron wants to start building

### Session Log — 2026-02-21

**Focus:** Pre-launch cleanup, pricing reset, spec work

**Shipped:**
1. Network Posts spec written (DEC-051) — 4-phase feature for network announcements, reactions, comments, polls
2. Business Dashboard: Joy Coins upgrade gate → Coming Soon with Community Pass education
3. Business Dashboard: Revenue upgrade gate → Coming Soon messaging
4. Business Dashboard: Settings subscription card → Founding Member acknowledgment, upgrade button removed
5. All pricing stripped from dashboard ($79 Standard, $149 Partner references removed)
6. Community Pass interest capture on Joy Coins tab — single "Yes, I'm interested" button, saves to Business entity
7. Business Dashboard greeting fixed to use display_name instead of email
8. MyLane Dynamic Layout spec written — 3 user states (new/active/power) with conditional section ordering
9. FinancialWidget upgrade nudge softened to "More features coming soon"
10. Dashboard Workspaces spec written (DEC-053) — workspace model formalizing Dashboard as multi-type producer view, Team workspace as first new type with Playbook feature

**New entities/fields:**
- Business entity: community_pass_interest (string, nullable) added in Base44

**Key decisions:**
- Pricing model under active rethink — $79/$149 tiers removed from UI, three revenue layers emerging ($9 supporter, ~$49 Community Pass, business tiers TBD)
- Founding Member concept adopted for early businesses
- "Maybe later" removed from Community Pass interest — single positive opt-in only
- MyLane Dynamic Layout specced for post-launch (3 user states, hide empty sections, compact networks for returning users)

**Specs written:**
- NETWORK-POSTS.md (private repo) — DEC-051
- MYLANE-DYNAMIC-LAYOUT.md (private repo)
- DASHBOARD-WORKSPACES.md (private repo) — DEC-053

**Next session:**
- Mobile device walkthrough (real phone testing)
- Broken links / dead ends sweep
- Fix anything found during walkthrough
- Publish and go live to users

### Session Log — 2026-02-20

**Focus:** Newsletter system, business settings/cards, security migrations, codebase audit, MyLane networks redesign

**Shipped:**
1. Good News newsletter system — footer email capture, onboarding wizard opt-in, post-RSVP prompt, admin subscriber list (source badges, status dots), NewsletterSubscriber entity with permissions
2. Business Settings upgrade — category dropdown (matches onboarding), working logo upload, structured address fields with state dropdown and map toggle, shared US_STATES list
3. Business card redesign — vertical layout, whole-card clickable, image on top, category fallback when no description, removed call button and View Profile button
4. Business update security — server function with owner/co-owner/admin auth, field allowlist, slug collision handling (appends -2, -3, etc.)
5. AccessWindow + Location security — writes migrated to updateBusiness server function with auth, entity permissions locked (Update/Delete: Creator Only)
6. Business entity permissions locked in Base44 (Update/Delete: Creator Only)
7. Full codebase audit — 56 findings across 6 categories (language, dead code, security, style, data consistency, accessibility). 6 critical security findings identified and resolved.
8. Console.log cleanup — 25+ debug logs removed across 10 components
9. Toggle knob color fix — 6 instances of bg-slate-100 corrected to bg-slate-300
10. MyLane networks redesign — two-state section (discovery for new users, navigation for followers), follow/unfollow moved to network page, back button added
11. Newsletter bug fixes — .filter().list() syntax, field name mismatch (active vs is_active), missing entity fields, async toast flow, duplicate check (list + client match)

**Decisions made:**
- Newsletter platform: Buttondown ($9/mo) chosen for The Good News
- MyLane networks: removed follow toggles, added discovery state for new users, follow action lives on network page
- Business cards: vertical layout with whole-card clickable, removed phone call button (available on profile)
- Duplicate check pattern: NewsletterSubscriber uses .list() + client-side .some() match (Base44 .filter() unreliable for exact match)

**Specced (ready to build):**
- MAP-VIEW.md — network page map view with Leaflet + Nominatim geocoding, geocode-on-save, privacy-aware pins
- Good News newsletter header and footer graphics created (Gemini, PNW botanical style)
- MyLane networks redesign prompt written and executed

**Key discoveries:**
- Base44 entity .filter() returns array directly, no .list() chaining
- Base44 .filter({ field: value }) may not match exactly — safer to .list() + client filter at small scale
- Code deploys require Base44 publish after GitHub sync
- Entity field mismatches cause silent create failures (no error thrown, record partially created)

**Security posture after today:**
- Business writes: server function with auth ✅
- AccessWindow writes: server function with auth ✅
- Location writes: server function with auth ✅
- Business entity: locked (Creator Only) ✅
- AccessWindow entity: locked (Creator Only) ✅
- Location entity: locked (Creator Only) ✅
- Remaining: Spoke, CategoryClick, Business create (onboarding) — medium priority, tracked in audit

**Next up:**
- Buttondown account setup, custom domain (goodnews@locallane.app), first issue draft
- Map view build (Phase 1: geocoding infrastructure)
- Medium audit items: centralize tier config, clarify category systems, staff role badge colors

---

### Session Log — 2026-02-19

**Focus:** Network Events spec, user onboarding, mobile audit resolution, config system fix

**Shipped:**
1. Network Events spec written (DEC-050) — 3-build sequence for network membership
2. Admin network assignment on Business edit drawer with Save Changes pattern
3. Config save fix — direct AdminSettings.update() replacing broken server function (500 errors)
4. User onboarding wizard — 3-step flow (Welcome → Networks → Community Pass) with display name
5. MyLane onboarding redirect — synchronous Navigate check with localStorage gate
6. MyLane "My Networks" section with toggle cards (DEC-050 Build 1)
7. Admin Users section shows community_pass_interest and network_interests
8. Mobile audit — 103 findings identified and all 103 resolved across 7 phases
9. Mobile header now shows "Local Lane" text
10. Onboarding Step 2 subtitle: "You can change these anytime from your dashboard"

**Key decisions:**
- DEC-050: Network Events & User Network Membership (Active)
- Config save uses direct AdminSettings.update() instead of server function
- localStorage approach for onboarding gate (avoids existing user edge cases)
- Deactivating all networks except Recess and Harvest for pilot

**Next session:**
- DEC-050 Build 2: Network-only events (EventEditor toggle + client-side filtering)
- Mobile stranger test (onboarding → MyLane → network toggles)
- Onboarding polish: hide nav header during wizard
- DEC-050 Build 3: Network info pages

---

## Session Log — 2026-02-19 (Evening Build Sprint)

### 2026-02-19 — DEC-050 Builds 2 & 3, User Flow Fixes

**What shipped:**

DEC-050 Build 2: Network-Only Events
- EventEditor: "Network Members Only" toggle tile (pure CSS, DEC-018 pattern) below network selector, only visible when network selected
- Events page: client-side filtering hides network_only events from users not following that network
- MyLane Happening Soon: same filtering with currentUser cache invalidation fix
- EventCard: "Recess Only" / "Harvest Only" pill badge on network-only events
- EventDetailModal: gate for non-followers with "Join [Network]" CTA, sign-in prompt for logged-out users
- Fix: network_only field was missing from event save payload (added to handleSubmit eventData object)

DEC-050 Build 3: Network Info Pages + Admin Fields + EventEditor Permissions
- Admin network editor: added tagline and description fields to network config
- EventEditor: network selector restricted to business's assigned networks (admin sees all)
- Network info pages at /networks/:slug with header, tagline, join/leave button, description, filtered events, filtered businesses
- Networks index at /networks with cards for each active network
- Routes added to App.jsx for /networks and /networks/:slug
- Network page shows ALL events including network_only (removed filter that hid them on the network's own page)

Navigation Cleanup
- Removed "Networks" from top nav bar (was Directory | Events | Networks | Dashboard)
- Nav is now: Directory | Events | Dashboard
- MyLane network cards are clickable links to /networks/:slug
- Follow/unfollow toggle works independently via stopPropagation
- "Browse all networks →" link below network cards

Conditional Business Dashboard
- "Business Dashboard" only shows in dropdown if user owns a business or is admin
- New useUserOwnedBusinesses hook with shared cache key
- MyLane shows "Run a local business?" CTA card for users without businesses
- "Get Started" links to business onboarding wizard

Auto-Publish for Network-Assigned Businesses
- Events auto-publish if: admin, OR business has network_ids and event tagged with assigned network, OR standard/partner tier
- Only basic tier with no network assignment goes to pending_review

Onboarding Fixes
- Name saves to both display_name and full_name; greeting reads display_name first
- Server-side onboarding_complete check prevents wizard reappearing across devices/sessions
- HappeningSoon cache invalidation: MyNetworksSection calls onUpdate to invalidate currentUser query

Other Fixes
- Business entity permissions changed to No-Restrictions for update (was Creator-Only, blocked admin edits)
- Business card overflow fix in NewInCommunitySection
- PWA cache issue documented (home screen app needs re-add after deploys)

**Network config:**
- Recess tagline: "Move your body, build your crew"
- Harvest tagline: "Know your farmer, feed your family"
- Only Recess and Harvest active for pilot

**Test data:**
- Troy's Eggs: test microbusiness under test1 user, assigned to Harvest network

**Known issues for next session:**
1. Business profile editing does not exist anywhere — not in owner dashboard Settings, not in admin panel. The ONLY place to edit a business profile is during initial business onboarding. Once completed, name/description/address/phone/hours/photos cannot be changed. This is the highest priority fix.
2. No address completion nudge for network businesses (needed for map)
3. Network page needs map view for businesses with addresses (Leaflet/OpenStreetMap)
4. Console.logs to clean up: HappeningSoonSection, GreetingHeader, EventEditor, BusinessEditDrawer, NetworkPage
5. Onboarding wizard still shows nav header (should be full-screen)
6. Mobile stranger test still pending
7. Business dashboard shows full tabs (Joy Coins, Revenue, etc.) for microbusinesses — may need simplified view

**What's next (in order):**
1. Business profile editing — build edit capability in both: (a) owner's dashboard Settings tab so business owners can update their own profile, and (b) admin BusinessEditDrawer so admin can edit any business. Currently neither surface supports editing business profile fields (name, description, address, phone, hours, photos).
2. Address nudge for network businesses ("Complete your address to appear on the network map")
3. Map view on network pages (Leaflet/OpenStreetMap, plot businesses with addresses)
4. Console.log cleanup pass
5. Mobile stranger test
6. Onboarding nav header fix
7. Continue user flow testing — walking through as real users and fixing what breaks

**Key files from this session:**
- src/components/dashboard/EventEditor.jsx — network_only toggle, network permission, auto-publish logic
- src/pages/NetworkPage.jsx — network info page with events/businesses
- src/pages/Networks.jsx — networks index
- src/components/mylane/MyNetworksSection.jsx — clickable cards, cache invalidation
- src/pages/MyLane.jsx — business CTA, onboarding server check
- src/hooks/useUserOwnedBusinesses.js — business ownership check hook
- src/Layout.jsx — conditional nav, networks removed from top bar
- src/pages/UserOnboarding.jsx — name saves to display_name + full_name
- src/components/admin/BusinessEditDrawer.jsx — debug logging for save
- functions/updateUser.ts — display_name added to allowed onboarding fields

**Architecture notes:**
- Network page shows all events including network_only (unlike Events page and Happening Soon which filter)
- Business.update() permissions changed to No-Restrictions in Base44 dashboard; UI gates editing behind isAppAdmin
- useUserOwnedBusinesses uses query key ['ownedBusinesses', userId] shared with BusinessDashboard
- currentUser query invalidation required after network follow/unfollow to update filtering

---

### Session Log — 2026-02-18

**Focus:** Pilot readiness — security lockdown, feedback system, feature gating, onboarding refactor

**Shipped:**
1. **Feature gating** — Joy Coins, Networks, Recurring Events behind isAppAdmin in EventEditor
2. **Feedback system** — floating button (Feedback/Bug tabs) + Admin Panel review section with type filtering and delete
3. **Entity security lockdown** — all 24 entities set to Restricted in Base44 dashboard (Creator Only writes, No Restrictions reads for core; Creator Only everything for Joy Coins entities; special rules for CategoryClick and FeedbackLog)
4. **GreetingHeader cleanup** — Silver, Passes, Tickets badges gated behind isAppAdmin
5. **MyLane cleanup** — Joy Coins badge and My Household section gated behind isAppAdmin
6. **Phone number auto-formatting** on Settings page ((XXX) XXX-XXXX)
7. **Onboarding wizard** refactored to config-driven architecture (src/config/onboardingConfig.js). Pilot: 3 steps (Type → Details → Review), 3 archetypes (Location/Venue, Event Organizer, Community/Non-Profit). Goals, Plan, and 2 archetypes dormant in config — flip active flags to enable, no code rewrite needed.

**Key decision:** Base44 AI assistant reverted manual entity permissions when asked to set security via schema files. Always use the dashboard UI manually for entity permissions.

**New file:** src/config/onboardingConfig.js — single source of truth for wizard steps, archetypes, goals, tiers, and pilot defaults.

**Entity created:** FeedbackLog (user_id, user_email, user_role, page_url, what_happened, what_expected, screenshot, feedback_type, created_at)

---

### Session Log — 2026-02-17

**Focus:** Strategic consolidation, Recess Network launch, logo update, Base44 architecture research

**Shipped:**
1. Homeschool Recess post published — Movement Mornings at The Circuit, Tuesdays and Thursdays 9-10 AM, first visit free, 30 for 30 after that
2. Flyer created via Gemini/Nana Banana Pro in Forging Resilience style
3. Header logo updated from storefront icon to new winding lane logo
4. Header text fixed from "Recess" to "Local Lane" with amber Gold Standard color
5. Header hover state added (white default, gold on hover)
6. Footer logo updated to match header

**Decisions made:**
- DEC-049: LocalLane Consolidation Strategy — build verticals inside Community Node (Phase A), migrate to Base44 Backend Platform (Phase B)
- Base44 confirmed apps are isolated, no cross-app shared auth or entities
- Base44 Backend Platform supports shared backend with multiple frontends, custom domains, entity migration
- Feature gate Joy Coins, networks, and recurring events for pilot launch
- Focus on LocalLane Community Node as primary build target going forward
- Standalone nodes continue serving current users but new development goes into Community Node
- Property Pulse paused — Doron reconsidering property manager role

**Key contacts from this week:**
- Troy Dean (Bushnell University Center for Calling & Community) — weekly student community service placements, potential student facilitator for Circuit sessions
- Orion Lawrenz (Lane County Farmers Market) — 100+ vendors, SNAP/EBT Double Up, Harvest Network connection

**Next up:**
- Feature gate Joy Coins, networks, recurring events in EventEditor
- Add feedback button (FeedbackLog entity + UI) to Community Node
- Stranger test: full mobile walkthrough before onboarding first groups
- First Movement Mornings session at The Circuit
- Await Base44 response on staging environment timeline
- HARVEST-MARKET-STOREFRONT.md captured in private — online storefront concept for Lane County Farmers Market with youth fulfillment, Stripe/EBT payment phases, and NW OG Farm booth as physical hub
- NW OG Farm post published to homeschool Facebook group — recruiting families for farm work and farmers market involvement
- Forage identified as SNAP/EBT online payment processor for Phase 2 (USDA-certified TPP, API-based, 6-month authorization timeline)

---

### Property Pulse — Session Log — 2026-02-15

**Focus:** LLC formation planning and Family Property Management Plan V4

**Shipped today:**
1. Family Property Management Plan V4 (PDF) — complete plan for Oregon duplex LLC formation
2. SSI benefits protection research — identified resource limit ($2,000) and income limit risks for Mom's 50% ownership
3. Legal aid resources compiled for Madras, Oregon (Mom's location): DRO, Oregon Public Benefits Hotline, LASO, Halpern Law Group, OSB Referral Service
4. Four resolution options documented: disclaimer, special needs trust, fair-value sale (flagged as problematic), do nothing
5. Financial waterfall modeled: $2,690 gross → $708 fixed → $269 mgmt fee → $404 reserves → $1,309 net distribution
6. Distribution structure: Sierran $654.50/mo, Doron $596.25/mo (mgmt + Mom's share), Aria $327.25/mo, Mom $0 (distributes her share)
7. Oregon property management licensing research — ORS 696.030(27) exemption confirmed for managing member of LLC

**Key findings:**
- Mom's 50% property ownership likely already violates SSI $2,000 resource limit (not yet flagged by SSA)
- Transfer penalty risk: gifting property = up to 36 months SSI ineligibility
- Installment sale does NOT solve SSI problem (note is countable resource + payments are income)
- Co-ownership hardship exclusion does not apply (neither owner lives in duplex)
- Berrey v. Real Estate Agency: partial ownership doesn't exempt property management licensing if managing for others for compensation — LLC resolves this

**Decisions made:**
- DEC-PP-009: LLC formation proceeds with Sierran as primary member, Doron as managing member. Mom's membership deferred pending SSI attorney consultation.
- DEC-PP-010: SSI resolution is a prerequisite for property title transfer into LLC. LLC setup (articles, EIN, bank account) can proceed in parallel.
- DEC-PP-011: Management fee set at 10% of gross rent ($269/mo). Reserves: 10% maintenance + 5% emergency (target $1,500). Family labor at $20/hr general, local handyman estimated $35/hr.

**Documents created:**
- Family_Property_Management_Plan_V4.pdf (sent to Mom and Sierran)

**Next up:**
- Sierran responds with 2054 Marion photos/details → draft rental listing
- Mom contacts legal aid resources in Madras (or hires Doron to coordinate)
- Find local handyman in North Bend/Coos Bay area
- Property Pulse Phase 3 planning (financial reporting, bank reconciliation)
- Contractor Daily Build 17b debugging (Base44 routing issue)

---

### Financial Node — Session Log — 2026-02-14

**Phase 1 COMPLETE. All 6 builds shipped in one session.**

**Builds shipped:**
1. Build 1 (prior session): Backbone — pages, nav, entities, pdf.js, Cash Log
2. Build 2: SELCO PDF parser v3 — rewrote for continuous text (pdf.js returns no newlines), cleanVendorName(), extractTransactionType(), vendorRules.js with 16 vendor patterns. Result: 20 transactions parsed, 18/20 auto-categorized
3. Build 3: Real Dashboard — month selector, income/expense/net summary cards, category breakdown with CSS bars, transaction timeline grouped by date
4. Build 4: Manual transaction entry — reusable TransactionForm component, modal from Transactions and Dashboard pages, edit support
5. Build 5: Debt tracker — Debts and DebtPayments entities, progress bars, priority ordering, payment logging with balance reduction, paid-off detection
6. Build 6: The Enough Number — Priority Stack with 5 tiers (Survival → Growth), auto-pull debt minimums from Debt Tracker, seed data on first visit, inline editing, stacked bar visualization, spending comparison

**Polish fixes shipped:**
- Amount input $ prefix: migrated from absolute positioning to flex layout across all 3 form files (TransactionForm, DebtForm, PaymentForm)
- Dark theme dropdowns and date picker (colorScheme: dark)
- DebtForm focus loss fix (inline component was recreating on every render)
- Delete debt with confirmation and cascading DebtPayment cleanup
- Edit button styling fix (white → Gold Standard slate secondary)
- useRef missing import fix on Enough page

**Technical discoveries:**
- Base44 .filter().list() returns 0 results — documented in ADR-001, all queries use client-side filtering
- pdf.js returns continuous text with NO newlines — parser must use regex on single string
- React components defined inside render functions cause focus loss on every keystroke

**Entities created:**
- Transactions, StatementUploads, CashEntries (Build 1)
- Debts, DebtPayments (Build 5)
- EnoughItems (Build 6)
- VendorRules (Build 2, local file — not Base44 entity)

**Phase 1 "Done When" criteria — ALL MET:**
- [x] Log a month of transactions (SELCO parser + manual entry)
- [x] See where money went (Dashboard with categories)
- [x] Know the Enough Number (Priority Stack page)
- [x] Track debt paydown progress (Debt Tracker with payments)

**New files created:**
- src/utils/statementParser.js (v3)
- src/utils/vendorRules.js
- src/components/TransactionForm.jsx
- src/components/DebtForm.jsx
- src/components/PaymentForm.jsx
- src/components/PriorityStack.jsx
- src/pages/Dashboard.jsx (real, replacing placeholder)
- src/pages/Debts.jsx
- src/pages/EnoughNumber.jsx
- docs/adr/001-base44-filter-workaround.md
- docs/debts-entities.md
- docs/enough-items-entity.md

**Strategic additions:**
- BJJ Ranked Queue concept captured as seed packet (BJJ-RANKED-QUEUE.md in private repo)
- YouTube-to-Claude Chrome extension noted for future tool-for-the-builder build
- Gemini Pro evaluated for logo/marketing asset generation

**Decisions made:**
- DEC-FIN-BUILD2-001: Client-side filtering pattern for all Base44 queries
- DEC-FIN-BUILD2-002: Vendor rules as local JS file, not Base44 entity
- DEC-FIN-BUILD2-003: Aggressive description cleanup with original preserved
- Financial Node moves from Phase 0 to Phase 1 COMPLETE in Node Registry

**Next up:**
- Phase 1 polish: fill in real Enough Number amounts
- Phase 2 planning: recurring transactions, benefits modeling, cross-node imports
- Upload February SELCO statement when available
- BJJ Ranked Queue: paper prototype planning (Phase 0)

---

### Contractor Daily — Session Log — 2026-02-12

**Build 17 complete. Pre-ship audit done. Shipped to Dan.**

**Builds shipped:**
1. Build 17a: Settings project assignments read-only with "managed on Clients page" link
2. Build 17b: Preview as Client — sessionStorage approach (Base44 strips query params from URLs)
3. Build 17c: Notes & Replies — threaded conversations via FeedbackLog parent_id, admin reply on FeedbackReview, client conversation view on ClientDashboard, nav badge for unanswered notes, filter pills
4. Build 17d: Camera icon on report list cards for logs with photos
5. Reports list card polish: two-row layout, spacing fix, edit/share/delete icons separated

**Bug fixes during session:**
- Preview as Client blank screen: switched from query params to sessionStorage
- Badge not decrementing after admin reply: CustomEvent dispatch + Layout listener refresh
- Replies showing as standalone cards: filtered parent_id records from All view
- Client "Your Notes" empty: fixed rootNotes filter and email matching in preview mode
- FeedbackLog delete permission: changed from Creator Only to No Restrictions

**Decisions made:**
- DEC-CD-029 through DEC-CD-035 (Notes/Replies, Settings read-only, Client Preview, Worker Audit, No-Role Landing, Onboarding Wizard, Field Service Engine Origin)

**Post-build audit:** 4 Critical, 9 High, 8 Medium, 8 Low findings. 5 critical/high fixes shipped pre-Dan. Remaining queued for post-field-test cleanup.

**Contractor Daily is now stable.** Dan is field testing. All future development (Build 18: multi-tenant isolation, Build 19: onboarding wizard, audit cleanup) moves to the Field Service Engine clone per DEC-CD-035.

**Second vertical signal:** Peter (Elite Auto Tech N Tow, Eugene) — mechanic and tow operator, 25+ years experience, runs on phone and memory, no job tracking or client system. Same workflow gaps as Dan pre-Contractor Daily. Vocabulary swap: Projects → Vehicles/Jobs, Materials → Parts, Daily Log → Work Order. Research conversation pending.

---

## Session Log — 2026-02-11

**Focus:** Strategic foundation — Node Lab Model

**Shipped today:**
1. NODE-LAB-MODEL.md (private repo) — strategic foundation for standalone node development
2. NODE-PLAYBOOK.md (private repo) — operational SOP with Feedback Kit specification, replaces new-node.md
3. Consolidation sweep: archived microbusiness-node.md and new-node.md, updated ARCHITECTURE.md, event-node.md, README.md
4. Node Registry established tracking all nodes and their current phase
5. Feedback Kit designed — portable FeedbackLog entity + UI for field testing nodes

**Strategic shift:**
- Nodes are now developed as standalone apps, not as sync-dependent children of Community Node
- Lab model: build independently, field test with real users, merge when mature
- Tier 4 (Enterprise) captured as horizon concept — entities that interoperate with LocalLane without being contained by it
- Research-first principle codified: study the domain, find the expert frameworks, talk to the real user before writing a spec
- "Find the Dan" principle: every node needs a named real person who will test it
- Feedback Kit: portable FeedbackLog entity + feedback button added to Phase 3 standard kit for all field-testing nodes

**Documents affected:**
- ARCHITECTURE.md: Node Communication Patterns marked as future integration reference
- event-node.md: Flagged for reassessment against Community Node current state
- microbusiness-node.md: Archived (concepts absorbed into Property Pulse and individual nodes)
- new-node.md: Archived, replaced by NODE-PLAYBOOK.md (private repo)
- README.md: Updated file references and repo list

**Current node status:**
- Contractor Daily: Phase 3 complete, Dan invited for field testing, client view button-up + Feedback Kit next
- Property Pulse: Phase 2 complete, on hold pending revisit
- Event Node: Needs assessment — Community Node may have surpassed it
- Donut Shop: Phase 0 research — phone call with brother pending
- Financial Node: Phase 0 concept — next active build after Contractor Daily ships
- Community Node: Pilot-ready, blocked on legal review

**Decisions:**
- DEC-047: Node Lab Model adopted as development approach
- DEC-048: Research-first principle — domain research and expert frameworks required before spec

**Next up:**
- Button up Contractor Daily: client permissions (DEC-CD-024) + Feedback Kit build
- Phone call with brother for Donut Shop Phase 0
- Event Node assessment: what does it have that Community Node doesn't?

---

## Session Log — 2026-02-10 (Contractor Daily Phase 3)

**Phase 3 complete. Multi-user access shipped.**

**Builds shipped:**
1. Contractor Daily Build 10: Role System Foundation — RoleContext, useRole(), auto-link on login, Settings User Management with per-person permission toggles
2. Contractor Daily Build 11: Client Read-Only View — Project Timeline + Reports filtered to assigned projects, summary-only cost visibility (never line items), receipt-split log filtering
3. Contractor Daily Build 12: Worker View — permission-gated Daily Log sections, assigned projects only, own labor scoping
4. Contractor Daily Build 13: Nav & Route Gating — roleAccess.js page matrix, RoleGate on all routes, login redirect per role, sign out for all roles
5. Phase 3 audit (Claude Code) — read-only review, 5 findings
6. Phase 3 audit fixes — client nav link, worker cost leak in Reports, sign out button, receipt-split filtering in Reports list

**Decisions activated:**
- DEC-CD-005, DEC-CD-017 through DEC-CD-020 moved from Planned to Active

**Key patterns established:**
- Configurable per-person permissions with role defaults (matches LocalLane DEC-046 architecture)
- Summary-only client cost view — progress framing, never line items (cortisol-conscious design)
- Receipt-split auto-log filtering for clean client experience
- Role system portable across all LocalLane archetypes (same pattern in Property Pulse, Contractor Daily, future nodes)

**Next up:**
- Multi-user testing with test accounts
- Field testing with Dan Sikes
- New archetype scoping: Donut Shop, Fitness/Recess

---

### Contractor Daily — Session Log — 2026-02-10 (Phase 2)

**Phase 2 core completed.**

**Builds shipped:**
1. Quick fix: Edit icon (Pencil) added to Reports list cards — edit/share/delete icon row
2. Build 8: Project Timeline — chronological feed of all logs per project with running totals, photo grid with lightbox, "View Full Report" links, accessed via "View Timeline" button on Projects detail
3. Build 9: Running project totals on Dashboard — per-project materials/labor/total on active project cards, summary bar with total across all projects, "View Timeline" links per card

**New files created:**
- src/pages/ProjectTimeline.jsx

**Modified files:**
- src/pages/Reports.jsx (edit icon on list cards)
- src/pages/Projects.jsx (View Timeline button)
- src/pages/Dashboard.jsx (running totals, summary bar, timeline links)
- src/pages.config.js (ProjectTimeline route)

**Key patterns:**
- ProjectTimeline loads all logs + materials + labor + photos per project, aggregates totals
- Photo lightbox: pure CSS/React, no library — dark overlay, centered image, close button
- formatDateRange helper for "Feb 8 – Feb 10, 2026" display
- stopPropagation pattern: card-level Link + icon-level Links coexist without conflict

**Phase 2 status:**
- Build 7: Receipt Split ✅
- Build 7-fix: Receipt Split save bug ✅
- Build 8: Project Timeline ✅
- Build 9: Running Totals on Dashboard ✅
- Build 10: Before/After Photos — deferred (marketing, not daily workflow)
- Build 11: Report Enhancements — deferred (logo/PDF export, not blocking)

**Next up:**
- Claude Code post-build audit (Phase 2)
- Phase 3 planning: Role system, client/worker views, Organism signals

---

### Contractor Daily — Session Log — 2026-02-10

**Phase 1 completed. Phase 2 started.**

**Builds shipped:**
1. Build 6: Settings polish — worker editing, toast feedback, all profile fields
2. Build 6a: Phase 1 audit fixes — iOS input zoom, deep links, hover states, shared formatCurrency utility, dead code cleanup
3. Nav hover fix — Layout.jsx desktop/mobile nav updated to Gold Standard amber hover
4. Build 7: Receipt split — multi-project material entry from single receipt
5. Build 7-fix: Receipt split save bug — materials now route to correct project logs, UUID address guard

**Phase 1 completion audit (Claude Code):**
- 2 PASS / 6 WARNING / 1 FAIL
- FAIL (iOS input zoom) fixed in Build 6a
- All 6 warnings addressed or documented
- BUGS.md reconciled against audit findings

**Decisions made:**
- DEC-CD-015: Receipt Split for multi-project materials
- DEC-CD-016: Shared utility pattern (formatCurrency.js)

**New files created:**
- src/utils/formatCurrency.js

**Key patterns established:**
- Independent audit workflow: Claude Code reads repo → reports findings → Claude Desktop writes fix prompts → Cursor executes
- Receipt split: opt-in toggle, per-line project tagging, auto-create logs for other projects
- Shared utilities in src/utils/ imported across pages

**Next up:**
- Build 8: Project Timeline View
- Field testing with Dan Sikes on mobile
- Base44 plan evaluation (Pro → Builder)

---

## Session Log — 2026-02-08 (Strategy + Co-Owner Build)

**Strategy work:**
- ORGANISM-ARCHITECTURE-EVOLUTION.md written (private repo) — co-ownership, archetype dashboards, tier philosophy reframe, multi-community Forest concept
- SILVER-DIME-LAYER.md written (private repo) — parallel physical currency layer, cross-community bridge, Earn-Your-Pass integration, bullion dealer as organism node, legal firewall analysis
- Analog quest pattern discussed — Doron's kids already running silver dime quests at The Circuit. Life as a game in the real world, not gamification on a screen. Quests section ready to add to Silver Dime doc.

**Shipped today (co-owner role):**
1. Server auth: co-owner role added to canEditBusiness, canEditEvent, canManageEvent across 3 server functions (0df37d6)
2. UI: co-owner role in StaffWidget, BusinessEditDrawer, BusinessCard — dropdown, badge (amber outline), permission checks (5bf3454)
3. Bug fix: addStaffMutation was not writing role to staff_roles AdminSettings for direct-add users. Fixed (131f24d)
4. Verified end-to-end: role saves correctly, badge renders, data confirmed in Base44

**Decisions made:**
- DEC-043: Co-owner role via staff_roles (complete)
- DEC-044: TAB_CONFIG archetype dashboard pattern (designed)
- DEC-045: Silver dime layer as parallel physical currency (designed)
- DEC-046: Custom role permissions (designed, deferred)

**Private repo docs created:**
- ORGANISM-ARCHITECTURE-EVOLUTION.md
- SILVER-DIME-LAYER.md

**Bugs found and fixed:**
- addStaffMutation role write missing for direct-add flow (fixed, 131f24d)

**Next session priorities:**
- Lock DEC-043 co-owner decision in DECISIONS.md ← (this prompt)
- Add Quests section to SILVER-DIME-LAYER.md
- Mobile nav + device testing
- Settings display name bug
- Business onboarding wizard simplification

---

## Session Log — 2026-02-08

**Shipped today:**
- Security audit Phase 3b: Business + Event service role migration (f6502c1)
- Security audit Phase 3c: User + Recommendation + Concern service role migration (f94b962)
- Security audit Phase 3d: RSVP + Joy Coin orchestration service role migration (f979527)
- All 10 Phase 3 entities locked in Base44 dashboard
- 6 server functions total: updateAdminSettings, updateBusiness, manageEvent, updateUser, manageRecommendation, manageRSVP

**Security audit final status:**
- 18 of 18 entities secured (Phase 1: 2 deleted + 2 locked, Phase 2: 7 locked, Phase 3: 10 locked via service role migration)
- 6 server functions enforce authorization (admin, owner, staff, self-only)
- Zero client-side entity writes for protected operations

**Bugs noted (non-blocking):**
- Dashboard greeting shows email instead of display name for non-owner users
- manageRecommendation user_name reads full_name instead of display_name
- Duplicate vouches/recommendations allowed (no uniqueness check)
- Home tab events list not interactive (UX, not regression)

**Decisions made:**
- DEC-042 updated: Phase 3 complete

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
- **Spec updates:** community-org-engine.md major update — 8 sub-types, nonprofit pricing tiers, expanded data model, Burrito Brigade and Ophelia's Place test cases, implementation phases, Organism connection
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
