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

**Done when:** Stranger test passes — someone can register, browse, RSVP, find support, on mobile.

---

## Phase 2: Content & Community
*Real businesses, real events, real users*

- [ ] 5 businesses onboarded with complete profiles
- [ ] 10+ events in system (mix of one-time and recurring)
- [ ] Farm Program pilot live (Deck Family Farm)
- [ ] Event RSVP flow tested end-to-end
- [ ] Business claim flow tested
- [ ] At least 1 non-Doron admin user

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

**Stripe Setup (D-F):**
- [ ] D. Community Pass subscription product created
- [ ] E. Webhook handling for subscription events
- [ ] F. Revenue share pool logic implemented

**Organism Phase 1:**
- [ ] Personal creature concept defined
- [ ] First visual element in MyLane
- [ ] Creature responds to real participation data

**Done when:** First paying Community Pass member, Organism visible.

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
