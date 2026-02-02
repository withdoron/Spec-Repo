# LocalLane Launch Checklist

> Master checklist from content-first launch through money flowing.
> Replaces: pilot-launch.md, 7-day-plan.md (moved to checklists/archive/)
> Active checklist: farm-program-launch.md feeds into Phase 1 below.
> Updated: 2026-02-02

---

## Phase 1 — Content First (NOW)

*Get real businesses and real users on the platform. Zero code changes needed.*

### Outreach

- [ ] Facebook post published ✅ (done 2026-02-02)
- [ ] Identify 3-5 target farm/food businesses in Eugene area
- [ ] Personal outreach to each (in-person or phone preferred)
- [ ] Pitch: "Free listing, free events, we handle the tech, you bring the community"
- [ ] Collect per business: name, contact, description, logo/photo

### Onboard Each Business

- [ ] Create their account on LocalLane
- [ ] Set up business profile (name, description, category, location, contact)
- [ ] Upload photo or logo
- [ ] Create their first event together
- [ ] Verify event appears on public events page
- [ ] Walk them through business dashboard (events, RSVPs, stats)

### Drive Community

- [ ] Track Facebook post responses
- [ ] Share real events in a follow-up post with links
- [ ] Personal invites to 10-15 people for first farm events
- [ ] Brief any interested local groups (homeschool, community orgs)

### Platform Readiness (Verify Before Onboarding)

- [ ] New user can register and log in
- [ ] Browse Events page shows events with filters
- [ ] Event detail modal opens with full info
- [ ] RSVP button works (confirmation + count update)
- [ ] MyLane shows upcoming RSVPed events
- [ ] Business dashboard shows events and RSVP counts
- [ ] Mobile experience is usable

### Feedback

- [ ] After first week: ask each business what was easy, what was confusing
- [ ] After first event: ask 2-3 attendees how they found it
- [ ] Track feedback in simple list (notes app, Google Doc)
- [ ] Bring feedback to Claude Chat for roadmap decisions

### Phase 1 Done When

- [ ] 3+ businesses have profiles with at least one event each
- [ ] 10+ real RSVPs from community members
- [ ] At least 1 event has happened with real attendees
- [ ] Feedback collected from 2+ businesses and 3+ attendees
- [ ] No critical bugs blocking the core flow

---

## Phase 2 — Stranger Test & Polish

*A stranger should be able to use the platform without you standing next to them.*

### Stranger Walkthrough

- [ ] Have someone who's never seen LocalLane try the full flow cold
- [ ] Can they figure out: what this is, how to browse, how to RSVP?
- [ ] Document every point of confusion
- [ ] Fix the top 3 friction points before continuing

### Professional Basics

- [ ] Set up locallane.com email (even a simple forwarder)
- [ ] Terms of Service page (real content, not placeholder)
- [ ] Privacy Policy page (real content, not placeholder)
- [ ] Footer links point to real pages (Terms, Privacy, Support contact)
- [ ] Support email or contact method visible to users

### Quick Fixes (from STATUS-TRACKER)

- [ ] Landing page routing: "/" → MyLane for logged-in users
- [ ] Autofill white flash fix on dark inputs
- [ ] Test user full_name in published environment
- [ ] Update DECISIONS.md with DEC-028 through DEC-031
- [ ] Replace STRIPE-CONNECT.md with v2

### Mobile Polish

- [ ] Walk through full flow on phone (iOS Safari + Android Chrome)
- [ ] Fix any layout issues, text size problems, or touch targets
- [ ] Verify nav menu works cleanly on mobile

### Phase 2 Done When

- [ ] A stranger completed the full flow without help
- [ ] Terms, Privacy, and Support pages exist with real content
- [ ] Professional email address is live
- [ ] Quick fixes from STATUS-TRACKER are shipped
- [ ] Mobile experience is solid

---

## Phase 3 — Legal & Payments Foundation

*Before real money flows through the platform.*

### Legal Checklist (from LEGAL-RESEARCH.md)

- [ ] Register business entity/DBA with Oregon Secretary of State
- [ ] Obtain EIN
- [ ] Open business operating account
- [ ] Professional legal review (~$100): 30-min consultation confirming Community Pass isn't money transmission
  - Oregon Lawyer Referral Service ($35) or Lewis & Clark Small Business Legal Clinic
  - Bring LEGAL-RESEARCH.md and Deep Research analysis
- [ ] Revenue share agreement template for participating businesses
- [ ] 1099-NEC preparation plan (collect business TIN during Stripe Connect onboarding)
- [ ] Terms of Service: include Community Pass clauses + auto-renewal disclosures
- [ ] Privacy Policy: payment data handling disclosures

### Stripe Setup (Phases A-C from STRIPE-CONNECT.md)

- [ ] **Phase A — Platform Setup:** Stripe account config, create products/prices for tiers, webhook URL
- [ ] **Phase B — Business Tier Subscriptions:** Real billing for Basic/Standard/Partner tiers
- [ ] **Phase C — Business Connect Onboarding:** Stripe Connect accounts, bank linking, identity verification

### Security Completion

- [ ] Phase 3 entity permissions (10 remaining entities from DEC-025)
- [ ] Service role function migration for locked entities
- [ ] Verify no public write access on sensitive entities

### Phase 3 Done When

- [ ] Legal entity registered with EIN
- [ ] Legal review confirms Community Pass model is clear
- [ ] Stripe processes real business tier subscriptions
- [ ] Businesses can link bank accounts via Stripe Connect
- [ ] Revenue share agreements signed with participating businesses
- [ ] All entity permissions are locked down

---

## Phase 4 — Community Pass & The Heart

*The product becomes uniquely LocalLane.*

### Stripe Phases D-F

- [ ] **Phase D — Community Pass Subscriptions:** User membership billing ($25/month)
- [ ] **Phase E — Direct Purchases:** `on_behalf_of` checkout for event tickets, services
- [ ] **Phase F — Revenue Share Distribution:** Monthly pool calculation and payout to businesses

### Organism Phase 1 — The Seed

- [ ] Create `useVitality` hook (calculate from existing MyLane data)
- [ ] Create `CommunityOrganism.jsx` (CSS + SVG animation)
- [ ] Place in MyLane GreetingHeader alongside greeting and punch badge
- [ ] 3-5 visual states: Seed → Sprouting → Growing → Thriving → Resting
- [ ] Verify it reflects real community data, not artificial gamification

### Community Pass Launch

- [ ] Network passes configured (Recess, Creative Alliance, Harvest Network, Gathering Circle)
- [ ] Businesses have designated Community Pass offerings
- [ ] Scan/check-in system for tracking usage
- [ ] Revenue share pool distributing monthly

### Phase 4 Done When

- [ ] Community Pass members are paying and using passes at businesses
- [ ] Revenue share is flowing to participating businesses monthly
- [ ] The Organism is alive in MyLane, reflecting real community vitality
- [ ] Direct purchases work end-to-end with zero commission to LocalLane

---

## Phase 5 — Expand & Scale

*Prove the model, then grow.*

- [ ] Expand beyond farms: use farm program as proof to recruit other verticals
- [ ] Second Cell evaluation (next city/town)
- [ ] Area Steward recruitment and training
- [ ] Organism Phase 2 (network organisms, community organism)
- [ ] Partner Node template ready for Tier 3 clones
- [ ] Newsletter/comms system for Stewards

---

## Decision Filter

Before adding any feature to this roadmap, ask:

1. Does this help a real person right now?
2. Does this make the Organism more alive?
3. Does this move money closer to flowing?

If none of the above → defer it.

---

*This is the single source of truth for launch sequencing. Update as phases complete.*
