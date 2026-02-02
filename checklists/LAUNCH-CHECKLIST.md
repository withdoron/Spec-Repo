# LocalLane Launch Checklist

> Comprehensive app readiness checklist — what the platform needs before real users.
> Replaces the farm-program-specific version. Outreach lives in farm-program-launch.md.
> Updated: 2026-02-02

---

## Phase 1 — Platform Readiness

*Can a stranger use this app cold without hand-holding?*

### Core User Flow

- [x] New user can register and log in
- [x] Browse Events page shows events with filters
- [x] Event detail modal opens with full info
- [x] RSVP button works (confirmation, count updates)
- [x] MyLane shows upcoming RSVPed events
- [x] User Settings page (name, email, phone, home community)
- [x] Business dashboard shows events and RSVP counts

### Navigation & Routing

- [x] Landing page routes logged-in users to MyLane (not Directory) — needs verification
- [x] Footer links wired to real pages
- [x] User dropdown has Settings, Business Dashboard, Admin links
- [ ] All nav links work on mobile (hamburger menu tested)

### Legal Pages

- [x] Terms of Service page built (Terms.jsx)
- [x] Privacy Policy page built (Privacy.jsx)
- [x] Terms content reviewed — all 7 Community Pass clauses from LEGAL-RESEARCH.md Part 3 confirmed, Oregon auto-renewal disclosures (ORS 646A.293–.295) added in Section 4
- [x] Privacy content reviewed — payment data handling updated (no access to full card/bank data, subscription metadata, business TIN for 1099-NEC), works as pre-Stripe placeholder and post-Stripe disclosure
- [ ] Both pages accessible without login

### Support

- [x] Support link in footer (mailto)
- [ ] Decide support mechanism: email only? Contact form? FAQ page?
- [ ] Confirm support email address works (see Email Infrastructure below)

### Email Infrastructure

- [ ] Choose email provider (Google Workspace ~$7/mo, Zoho free tier, or domain registrar)
- [ ] Set up hello@locallane.com (primary contact, used in Terms/Privacy/Support)
- [ ] Set up support@locallane.com (support inbox)
- [ ] Update Terms.jsx, Privacy.jsx, and Footer.jsx if email differs from placeholder
- [ ] Test that emails actually arrive

### Visual Polish

- [x] Autofill white flash on dark inputs fixed (CSS autofill overrides) — needs verification
- [x] Dark theme consistent across all pages
- [x] Mobile-responsive layout
- [ ] Test on actual mobile devices (iOS Safari, Android Chrome)
- [x] Empty states have helpful messages — all sections (MyLane, Events, Directory, Search, Home) show friendly copy with next-step CTAs
- [x] Error states show friendly messages — errorMessages.js maps network/timeout/auth/5xx/404/429 errors to user-facing copy, raw error.message replaced in toasts and auth

### Phase 1 Done When

- [ ] Someone who has never seen LocalLane can register, browse events, RSVP, and find support — without asking you for help
- [ ] Terms and Privacy pages reviewed and accurate for current state
- [ ] Email infrastructure live (hello@ and support@ receiving mail)
- [ ] No broken links or dead-end pages

---

## Phase 2 — Content & Community

*Real businesses, real events, real users.*

### Business Onboarding

- [ ] 3+ businesses have profiles with at least one event each
- [ ] Business profiles have name, description, category, location, contact, photo
- [ ] Events appear on the public Browse Events page
- [ ] Business dashboards show their events and RSVP counts

### Community Engagement

- [ ] 10+ real RSVPs from community members
- [ ] At least 1 event has actually happened with real attendees
- [ ] Feedback collected from at least 2 businesses and 3 attendees

### Phase 2 Done When

- [ ] Real people are using the platform for real events
- [ ] Feedback is informing what to build next
- [ ] No critical bugs blocking the core flow

---

## Phase 3 — Legal & Payments Foundation

*Before real money flows through the platform.*

### Business Formation

- [ ] Register entity/DBA with Oregon Secretary of State
- [ ] Obtain EIN
- [ ] Open business operating account

### Legal Review

- [ ] Professional legal review confirming Community Pass is not money transmission (~$100, Oregon Lawyer Referral $35 or Lewis & Clark clinic)
- [ ] Revenue share agreement template for participating businesses
- [ ] 1099-NEC preparation plan (collect business TIN during Stripe Connect onboarding)
- [ ] Terms of Service updated with all 7 Community Pass clauses + auto-renewal disclosures
- [ ] Privacy Policy updated with payment data handling disclosures

### Security

- [ ] Phase 3 entity lockdown (10 remaining Base44 entities — requires service role function migration)
- [ ] Auth flow edge cases tested
- [ ] Data access patterns verified

### Stripe Setup (Phases A-C)

- [ ] Phase A — Platform Setup: Stripe account, products, webhook URL, API keys in Base44
- [ ] Phase B — Business Tier Subscriptions: checkout sessions, webhook handlers, subscription lifecycle
- [ ] Phase C — Business Connect Onboarding: connected accounts, bank account linking, account.updated webhook

### Phase 3 Done When

- [ ] Legal structure confirmed by counsel
- [ ] Stripe can accept business tier subscriptions
- [ ] Businesses can link bank accounts via Connect
- [ ] Security audit complete (all 18 entities locked)

---

## Phase 4 — Community Pass & The Heart

*Money flows. The Organism wakes up.*

### Stripe (Phases D-F)

- [ ] Phase D — Community Pass Subscriptions: user membership billing ($25/month)
- [ ] Phase E — Direct Purchases: on_behalf_of checkout for event tickets and services
- [ ] Phase F — Revenue Share Distribution: monthly pool calculation and payout to businesses

### Organism Phase 1 — The Seed

- [ ] Create useVitality hook (calculate from existing MyLane data)
- [ ] Create CommunityOrganism.jsx (CSS + SVG animation)
- [ ] Place in MyLane GreetingHeader alongside greeting and punch badge
- [ ] 3-5 visual states: Seed → Sprouting → Growing → Thriving → Resting
- [ ] Reflects real community data, not artificial gamification

### Community Pass Launch

- [ ] Network passes configured (Recess, Creative Alliance, Harvest Network, Gathering Circle)
- [ ] Businesses have designated Community Pass offerings
- [ ] Scan/check-in system for tracking usage
- [ ] Revenue share pool distributing monthly

### Phase 4 Done When

- [ ] Community Pass members are paying and using passes at businesses
- [ ] Revenue share flowing to participating businesses monthly
- [ ] The Organism is alive in MyLane, reflecting real community vitality
- [ ] Direct purchases work end-to-end

---

## Phase 5 — Expand & Scale

*Prove the model, then grow.*

- [ ] Expand beyond initial vertical — use proof from Phase 2 to recruit other business types
- [ ] Second Cell evaluation (next city/town)
- [ ] Area Steward recruitment and training
- [ ] Organism Phase 2 (network organisms, community organism)
- [ ] Partner Node template ready for Tier 3 clones
- [ ] Newsletter/comms system for Stewards

---

## Open Items (Not Yet Assigned to a Phase)

These surfaced in conversation but need decisions before they land on the roadmap:

- [ ] Notification system — email first, PWA push later. Deferred to post-pilot (decision made 2026-02-01)
- [ ] Profile photo upload — stubbed with placeholder, needs image storage decision
- [ ] Mobile PWA considerations — bottom nav, install prompt
- [ ] Ownership badges ("Locally Owned", "Local Franchise") — concept exists, not scoped
- [ ] Silver Barter Network — concept doc written, Phase 1 is badge + filter (near-term)

---

## Decision Filter

Before adding any feature to this roadmap:

1. Does this help a real person right now?
2. Does this make the Organism more alive?
3. Does this move money closer to flowing?

If none → defer it.

---

*This is the single source of truth for launch sequencing. Update as phases complete.*
