# LocalLane Launch Checklist

> Living checklist for app readiness. Updated after each work session.
> Last updated: 2025-02-02

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

**The Good News (Newsletter):**
- [ ] Email capture in footer (simple: email input + "Join The Good News")
- [ ] One-time prompt after first RSVP ("You're in! Want community wins in your inbox?")
- [ ] Newsletter list created (Mailchimp, Buttondown, or similar)
- [ ] First issue drafted (can wait until Phase 2 content exists)

**Language Audit — Community Pass (LEGAL):**
- [ ] Full codebase search for "punch" — replace with Community Pass terminology
- [ ] UI audit: no references to "punch card" or "stored value"
- [ ] Database fields: document any legacy field names (don't rename yet, just document)
- [ ] Terms of Service: verify Community Pass language (not stored value)
- [ ] Privacy Policy: verify membership framing (not prepaid balance)
- [ ] User-facing copy audit: MyLane, Business Dashboard, Event Detail

**Community Pass Foundation (Phase 1):**

### Legal & Structure
- [ ] Professional legal review for membership model (~$100-200)
- [ ] Revenue share agreement template drafted
- [ ] Terms of Service updated for membership framing (not stored value)
- [ ] Oregon-specific review for revenue share pool
- [ ] Ticket transfer language reviewed

### Pricing Validation
- [ ] Talk to 5+ "interested" respondents about pricing
- [ ] Test founding member concept reception
- [ ] Validate ticket quantities feel sufficient per tier
- [ ] Confirm access window model makes sense to partner businesses
- [ ] Document final pricing decision in COMMUNITY-PASS.md

### Brand Foundation
- [ ] MISSION-VISION-VALUES.md published to Spec-Repo
- [ ] Tagline finalized: "Local Lane — Where community be-comes alive."
- [ ] Core messaging reviewed against book frameworks
- [ ] No discount/savings language in any copy

**Done when:** Stranger test passes — someone can register, browse, RSVP, find support, on mobile. No "punch card" language visible anywhere.

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

## Phase 4: Community Pass & The Heart
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

**Community Pass Launch (Phase 4):**

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

## Phase 5: Expand & Scale
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
