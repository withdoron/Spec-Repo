# LAUNCH-CHECKLIST.md — App Launch Readiness

> What the app needs before real users arrive.
> Not outreach, not content seeding — just the app itself.
> Updated: 2026-02-02
> Status: Active

---

## Phase 1: Legal Foundation (No Budget Required)

### Terms of Service page — Terms.jsx with route /terms

- Content drafted in Chat (Oregon auto-renewal disclosures, Community Pass placeholder)
- Cursor prompt written (legal-pages-cursor-prompt.md)
- Footer link exists but page may not — VERIFY
- Session: ~30 min

### Privacy Policy page — Privacy.jsx with route /privacy

- Content drafted alongside Terms
- Same verification needed
- Session: bundled with Terms above

### Footer links functional — Terms, Privacy, Support mailto

- Footer.jsx updated with Link components ✅
- Verify all three links resolve correctly
- Session: 5 min verification

---

## Phase 2: First Impressions (Quick Fixes)

### Landing page routing — Logged-in users → MyLane, not Directory

- 15 min Cursor fix
- Currently everyone lands on Directory regardless of auth state

### Logged-out landing page — Clear value prop, sign-up CTA

- New users see what LocalLane is before signing up
- 1 session

### Autofill white flash fix — Login form background flash on browser autofill

- CSS fix, ~15 min

### Test user full_name — Ensure display name renders everywhere

- Quick verification pass

---

## Phase 3: Support Infrastructure

### Email infrastructure — hello@locallane.com, support@locallane.com

- Options: Google Workspace (~$7/mo) or Zoho Mail (free tier)
- 20-minute setup once provider chosen
- Must happen before real user onboarding

### /Support page — FAQ + feedback form + bug report + contact

- Repurposes existing Concern entity as the backend
- Types: Bug Report, Feature Request, General Feedback, Concern about Business
- Routes to AdminConcernsPanel (already built)
- 1 session

### Concern workflow enhancements (spec: CONCERN-WORKFLOW.md)

- Enhancement 1: Auto-acknowledgment with reference #CON-{id} and 48hr timeline
- Enhancement 2: Severity category (safety/billing/experience/other)
- Enhancement 3: Admin notification badge (in-app, not email)
- Enhancement 4: Reporter status visibility ("My Concerns" in Settings)
- 2 sessions total, can be split

---

## Phase 4: Navigation & Flow Audit

### Chrome walkthrough — Doron navigates, Claude reviews

- New user: signup → browse → event detail → RSVP → MyLane
- Returning user: login → MyLane → recommendations → settings
- Business owner: login → admin → create event → manage staff
- Mobile: iOS Safari, Android Chrome spot check
- 1 session together

### Business onboarding flow polish — Audit wizard before outreach

- Ensure the path from signup to first event is smooth
- 1 session

---

## Phase 5: Pre-Payment Legal (When Money Gets Close)

### Professional legal review (~$35-100)

- Revenue share agreement templates
- Terms of Service update for Community Pass
- Oregon money transmitter question for subscription model
- 1099-NEC preparation requirements

### Stripe Connect setup — Unlocks all financial functionality

- See STRIPE-CONNECT.md for 5-phase plan
- Blocked by legal review above

---

## Not On This Checklist (Tracked Elsewhere)

- **Content seeding** (real businesses, events) — Doron's domain
- **Marketing materials** (Canva flyer, business card) — Doron's domain
- **Organism Phase 1** — Separate spec, no dependencies, can happen anytime
- **Security Phase 3** (10 remaining entities) — Deferred per DEC-025
- **Community Pass subscription system** — Post-Stripe

---

## Decision Filter

Before adding anything to this list:

1. Does this help the next 10 users?
2. Does this make the organism more alive?
3. Does this move money closer to flowing?

If none → defer.
