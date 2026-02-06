# LocalLane Launch Checklist

> Living checklist for app readiness. Updated after each work session.
> Last updated: 2026-02-05

---

## Phase 1: Platform Readiness (Stranger Test)
*Can someone who's never seen this use it cold?*

- [x] Landing → MyLane routing for logged-in users
- [x] Empty states show friendly messages (not blank screens)
- [x] Error states show human-readable messages (not raw errors)
- [x] Terms of Service reviewed (Community Pass clauses present)
- [x] Privacy Policy reviewed (payment data handling ready)
- [x] Terms/Privacy accessible without login
- [x] Autofill white flash fixed
- [x] Email infrastructure live (hello@ and support@locallane.app)
- [ ] Support mechanism decided (email-only for now — revisit if volume grows)
- [ ] Mobile nav links work on phone
- [ ] Full mobile device testing (iOS Safari, Android Chrome)
- [ ] No broken links or dead ends

**Known Bugs:**
- [ ] Settings: Display name update doesn't save / drawer doesn't close

**Onboarding — Business (Events Archetype Only):**
- [ ] Simplify wizard to Events archetype only (hide other archetypes for now)
- [ ] Fix step order: Archetype → Goals → Details → Tier → Review
- [ ] Goals step shows: Host Events, Accept Community Pass, List in Directory
- [ ] Tier recommendation based on goals (Accept Community Pass → Standard)
- [ ] Success screen with "Create your first event" CTA
- [ ] Mobile-friendly wizard (test on phone)

**Onboarding — User (Lightweight):**
- [ ] Welcome screen after signup (before MyLane)
- [ ] "What brings you to LocalLane?" interest capture (checkboxes)
- [ ] "Interested in Community Pass when it launches?" (Yes / Maybe later)
- [ ] Data stored for future outreach
- [ ] Skippable — user can go straight to MyLane

**The Good News (Newsletter & Community Pulse):**
- [ ] Email capture in footer (simple: email input + "Join The Good News")
- [ ] One-time prompt after first RSVP ("You're in! Want community wins in your inbox?")
- [ ] Newsletter list created (Mailchimp, Buttondown, or similar)
- [ ] First issue drafted — community health, scholarship stories, new businesses, seasonal rhythms
- [ ] Scholarship visibility: show where unused Joy Coins flowed and who they helped (with permission)
- [ ] Real photos, real families, real movement. No stock images, no promotional copy.
- [ ] Tone: community health report, not marketing newsletter

**Language Audit — Joy Coins & Community Pass (LEGAL):**
- [ ] Full codebase search for "punch" — replace with Joy Coin / Community Pass terminology
- [ ] UI audit: no references to "punch card", "punch pass", or "stored value"
- [ ] Database fields: document any legacy field names (don't rename yet, just document)
- [ ] Terms of Service: verify Community Pass + Joy Coin language (not stored value)
- [ ] Privacy Policy: verify membership framing (not prepaid balance)
- [ ] User-facing copy audit: MyLane, Business Dashboard, Event Detail
- [ ] Confirm "Joy Coins" used consistently (not "tokens", "credits", "punches", "tickets")

**Community Pass Foundation (Phase 1):**

### Legal & Structure
- [ ] Professional legal review for membership model (~$100-200)
- [ ] Revenue share agreement template drafted
- [ ] Terms of Service updated for membership framing (not stored value)
- [ ] Oregon-specific review for revenue share pool
- [ ] Ticket transfer language reviewed

### Pricing — LOCKED (DEC-032, DEC-033, DEC-034)
- [x] Community Pass pricing locked: $49 base / $39 per additional allocation (12 coins each)
- [x] Joy Coin pool model: coins not people, household shares pool (DEC-033)
- [x] Business tier pricing locked: Free / $79 Standard / $149 Partner (DEC-034)
- [x] Founding business deal: 3 months free at Standard
- [x] Revenue split locked: 75% business pool / 25% LocalLane (after Stripe fees)
- [x] Market research complete: Crabtree, ClassPass, 1Pass, Eugene gym pricing, off-peak strategy
- [ ] Founding member rate finalized (suggested ~$39/$29 — decide before launch)
- [ ] Talk to 5+ "interested" respondents about pricing
- [ ] Confirm access window model makes sense to partner businesses

### Brand Foundation
- [ ] MISSION-VISION-VALUES.md published to Spec-Repo
- [ ] Tagline finalized: "Local Lane — Where community be-comes alive."
- [ ] Core messaging reviewed against book frameworks
- [ ] No discount/savings language in any copy

**Done when:** Stranger test passes — someone can register, browse, RSVP, find support, on mobile. No "punch card" language visible anywhere.

**Earn-Your-Pass Program (DEC-035):**
- [ ] Farm program partnership confirmed (which farms participate)
- [ ] Farmers market participation pathway defined
- [ ] Tracking mechanism decided (manual coordinator tracking for V1)
- [ ] Contribution-to-pass conversion rate set (how much work = 1 month access)
- [ ] Coordinator identified for farm program
- [ ] Youth eligibility criteria defined (age range, parental consent)
- [ ] Story captured for first issue of The Good News

---

## Phase 2: Content & Community
*Real businesses, real events, real users*

- [ ] 5 businesses onboarded with complete profiles
- [ ] 10+ events in system (mix of one-time and recurring)
- [ ] Farm Program pilot live (Deck Family Farm)
- [ ] Event RSVP flow tested end-to-end
- [ ] Business claim flow tested
- [ ] At least 1 non-Doron admin user

**Access Window Implementation (Phase 2):**

### Event Archetype Updates
- [ ] Add Community Pass settings to event creation
  - [ ] Accept Community Pass toggle
  - [ ] Days/hours available
  - [ ] Capacity per window
  - [ ] Ticket cost (1-5, set by business)
  - [ ] Access type (free entry / discounted / special)
  - [ ] RSVP required toggle
  - [ ] No-show policy (Flexible / Moderate / Strict)
- [ ] Access windows display in business dashboard
- [ ] Access windows appear in member discovery (MyLane)

### Ticket System
- [ ] Member ticket balance tracking
- [ ] Ticket deduction on scan/booking
- [ ] Monthly ticket reset (no rollover)
- [ ] Ticket transfer (member-to-member) flow
- [ ] Cancellation handling per no-show policy

**Done when:** Platform has enough real content to feel alive.

---

## Phase 3: Legal & Payments Foundation
*Before any real money flows*

- [ ] LLC formation (Oregon)
- [ ] EIN obtained
- [ ] Business bank account opened
- [ ] Professional legal review (~$100 for Terms/Privacy)
- [ ] Revenue share agreement template drafted
- [ ] 1099-NEC preparation understood

**Stripe Setup (A-C):**
- [ ] A. Stripe account created & verified
- [ ] B. Connect onboarding for first business
- [ ] C. Test transaction successful

**Done when:** Legal entity exists, Stripe can accept test payments.

---

## Phase 4: User Groups
*Private coordination for people who know each other*

**Spec:** USER-GROUPS-v1.md (private repo)

### Phase 4A: Foundation
- [ ] Create user_groups entity in Base44
- [ ] Create user_group_members entity
- [ ] Create user_group_messages entity
- [ ] Create user_group_events entity
- [ ] Create user_group_event_rsvps entity
- [ ] Create user_group_invites entity
- [ ] Security rules for all 6 entities (user + admin)
- [ ] Basic group CRUD (create, view, soft delete)
- [ ] Member management (add existing users, remove, leave)
- [ ] Invite flow (email invites, accept/decline)

### Phase 4B: Communication
- [ ] Group chat UI (flat, no threading)
- [ ] Message display with sender + timestamp
- [ ] Chat input with send
- [ ] MyLane "My Groups" section

### Phase 4C: Private Events
- [ ] Create private event form
- [ ] Event display in group view
- [ ] RSVP buttons (going/maybe/can't)
- [ ] Private events appear in member's MyLane calendar
- [ ] Gathering detection (for organism pulse)

### Phase 4D: Shared Awareness
- [ ] Query public RSVPs for group members
- [ ] Display "X members going to [Event]" in group view
- [ ] Link to public event detail

### Phase 4E: Organism Infrastructure
- [ ] Pulse calculation logic (cron or on-demand)
- [ ] Streak tracking
- [ ] Data ready for future visualization

### Phase 4F: Admin Tools
- [ ] GroupsSection list view with filters
- [ ] GroupDetailDrawer (members, activity, moderation tabs)
- [ ] Suspend/unsuspend group action
- [ ] Remove member action (admin override)
- [ ] AdminSidebar updated with Groups link

**Done when:** Users can create private groups, invite friends, chat, create private events, RSVP. Admins can view and moderate. Organism pulse calculating in background.

---

## Phase 5: Joy Coins System
*The mechanics that make Community Pass work*

**Data Foundation:**
- [x] JoyCoins entity created (balance, transactions, reservations)
- [x] Transaction states implemented (available, reserved, spent, refunded, forfeited, expired)
- [x] Relationship to User, Event, Business, RSVP entities defined

**Event Creator Controls:**
- [ ] Network selection (pulls from admin config)
- [ ] Joy Coin cost setting per event
- [ ] Capacity controls (total vs Joy Coin member spots)
- [ ] Frequency limits (per day/week/month)
- [ ] Refund policy selection

**Member Experience:**
- [ ] Joy Coin balance display in MyLane
- [ ] Cost visibility on events
- [ ] Reservation status tracking
- [ ] Transaction history

**RSVP Integration:**
- [ ] Reserve Joy Coins on RSVP
- [ ] Deduct on check-in/attendance
- [ ] Refund on cancellation (per policy)
- [ ] Forfeit on no-show

**Business Analytics:**
- [ ] Redemption dashboard
- [ ] Capacity utilization metrics
- [ ] Revenue share tracking

**Done when:** A member can RSVP to an event using Joy Coins, attend, and have it properly tracked. Business can see redemption data.

---

## Phase 6: Community Pass & The Heart
*Revenue flowing, Organism comes alive*

**Community Pass Pricing Research (Before Any Pricing in UI):**
- [ ] Comparable models research (co-ops, CSAs, community memberships)
- [ ] Value-to-member math (what savings justify what price?)
- [ ] Revenue share sustainability analysis
- [ ] Eugene market pricing tolerance check
- [ ] Talk to 5+ "Interested in Community Pass" respondents from Phase 1
- [ ] Pricing decision documented in COMMUNITY-PASS.md

**Stripe Setup (D-F):**
- [ ] D. Community Pass subscription product created (price from research above)
- [ ] E. Webhook handling for subscription events
- [ ] F. Revenue share pool logic implemented

**Community Pass Purchase Flow:**
- [ ] Upgrade prompt in MyLane for free users
- [ ] Community Pass benefits explanation screen
- [ ] Stripe Checkout integration for subscription
- [ ] Success state → MyLane with pass badge visible
- [ ] Pass benefits reflected in UI (discounts shown at participating businesses)

**Community Pass Launch (Phase 6):**

### Member Experience
- [ ] Tier selection during signup (Individual/Duo/Family/Family+)
- [ ] Founding member pricing flag
- [ ] Ticket balance visible in MyLane
- [ ] Available access windows in discovery feed
- [ ] Booking flow with ticket cost display
- [ ] Scan/check-in at business
- [ ] Monthly "your impact" summary (not savings-focused)

### Business Experience
- [ ] Revenue share dashboard
- [ ] Scan history and attribution
- [ ] Access window management
- [ ] Capacity and booking visibility
- [ ] No-show tracking

### Revenue System
- [ ] Stripe Connect for revenue share payouts
- [ ] 75/25 split calculation
- [ ] Scan-based allocation logic
- [ ] Monthly payout cycle
- [ ] Business payment history

### Network Identity
- [ ] Four networks defined (Recess, Creative Alliance, Harvest, Gathering Circle)
- [ ] Network tagging for businesses/events
- [ ] Member interest mapping to networks
- [ ] Merch strategy documented (future)

**Organism Phase 1:**
- [ ] Personal creature concept defined
- [ ] First visual element in MyLane
- [ ] Creature responds to real participation data

**Done when:** First paying Community Pass member, Organism visible, pricing validated by research.

---

## Phase 7: Expand & Scale
*Growth beyond pilot*

- [ ] Second business vertical (beyond Farm Program)
- [ ] Second Cell (geographic or interest-based)
- [ ] Steward role defined and first steward active
- [ ] Notification system (email digests, RSVP reminders)
- [ ] PWA / mobile app consideration

---

## Open Items (Parking Lot)
*Not blocking launch, but don't forget*

- Profile photos (placeholder shows "coming soon")
- Badges/achievements system
- Silver Barter concept exploration
- Security Phase 3 (10 remaining entities need service role migration)
- hub.locallane.com URL in LockedFeature.jsx — is this real or cleanup needed?

---

## Session Log

| Date | Summary |
|------|---------|
| 2025-02-02 | Email infrastructure complete (IONOS, hello@ + support@ forwarding). Terms/Privacy email refs updated to .app. Landing page routing and autofill flash verified. Settings display name bug identified. |
| 2025-02-01 | Checklist rewritten from farm-focused to app-focused. Phase 1 code complete: empty states, error states, Terms/Privacy reviews. |
