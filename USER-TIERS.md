# User Tiers

> How LocalLane creates enough value that community members choose to pay for deeper access.
> "Value through trust and convenience — not ads."

---

## Status: 🔮 VISION DOCUMENT — Not Yet Implementing

Last updated: 2026-02-01

---

## Philosophy

LocalLane's revenue model has three pillars, none of which involve advertising:

| Pillar | Who Pays | What They Get | Status |
|--------|----------|---------------|--------|
| **Business Tiers** | Businesses | Tools to manage presence, events, analytics | ✅ Implemented (basic/standard/partner) |
| **User Tiers** | Community members | Deeper convenience and community access | 🔮 This document |
| **Transaction Fees** | Both (small cut) | Punch Pass, event tickets, marketplace orders | 📋 Stripe Connect spec'd |

The critical principle: **the platform's incentive is always aligned with the user's.** No advertising means no conflict of interest. Discovery is driven by community trust, not by who paid for placement. User tiers enhance the experience — they never degrade the free experience.

### Why This Matters

Every other local platform monetizes through ads. Yelp sells promoted placement. Eventbrite takes ticket fees that don't serve the user. LocalLane's model says: "We'll make this platform so useful to you personally that you'll want to pay for more of it." The value comes from the trust network underneath, not from content you could get anywhere.

---

## Tier Structure (Draft)

### Free Tier (Everyone)

**Must be genuinely useful on its own.** Free users generate the recommendations that make the entire platform valuable. A crippled free experience is self-defeating.

| Feature | Included |
|---------|----------|
| Browse events and directory | ✅ |
| Give Nods and Stories | ✅ |
| RSVP to events | ✅ |
| MyLane dashboard (Explorer → Connected) | ✅ |
| See trust signals (recommendations, vouches) | ✅ |
| Punch Pass (basic — pay per punch) | ✅ |
| Check in to events | ✅ |
| Search and filter | ✅ |

### Member Tier (Paid Monthly)

**The convenience and deeper access layer.** Not "see more stuff" — "the platform works harder for you."

| Feature | Description | Priority |
|---------|-------------|----------|
| Market Basket | Pre-order from farmers market vendors, scheduled pickup | High |
| Smart Notifications | "3 new events match your saved search" | High |
| Priority RSVP | First access to limited-capacity events | Medium |
| Punch Pass Perks | Bonus punches, better rates, or member pricing | Medium |
| Saved Filters | Save and re-apply search/filter combinations | Medium |
| Community Concierge | "I need a plumber — who do my neighbors recommend?" with faster, filtered results | Low |
| Enhanced Trust Network | Visual map of your community connections | Low |
| BuildLane Preferences | Fine-tune your discovery feed | Low |

**Pricing:** TBD. Initial thinking: $5-10/month. Must feel like obvious value.

---

## Feature Deep Dives

### Market Basket (Flagship Member Feature)

**The concept:** A farmer's market vendor has great produce. A busy parent wants it but can't make the Saturday 10am-2pm window. LocalLane bridges the gap.

**How it works:**
1. Participating vendors list available items for the upcoming market (e.g., "This Saturday: heirloom tomatoes, fresh basil, sourdough bread")
2. Member users browse and place orders by Thursday
3. Vendor prepares the order
4. Member picks up at a dedicated window — or eventually, community delivery

**Why it only works on LocalLane:**
- Trust layer: You know the vendor is legit because 47 neighbors recommend them
- Community connection: It's your neighbor's farm, not Amazon Fresh
- Platform alignment: LocalLane takes a small facilitation fee, vendor gets guaranteed sales, member gets access they couldn't otherwise have

**Dependencies:** Stripe Connect (for payments), business dashboard feature (for vendors to list items), order management system

**Priority:** High — this is the kind of feature that makes someone say "I'd pay $5/month for that."

### Smart Notifications

**The concept:** Instead of checking LocalLane daily, the platform tells you when something matches your interests.

**Examples:**
- "A new pottery class was posted within 5 miles of you"
- "Mountain View Farms (you follow them) just added a Saturday market event"
- "3 new businesses joined in your area this week"

**Implementation:** Push notifications or email digest. Requires user preference storage and category engagement tracking.

**Priority:** High — low friction, high perceived value.

### Priority RSVP

**The concept:** Limited-capacity events (cooking class with 12 spots, workshop with 20 seats) give member users a 24-hour early access window before general availability.

**Why it works:** Creates genuine scarcity value without excluding free users. Members get first pick, but the event still fills normally afterward.

**Priority:** Medium — depends on event volume and capacity limits being implemented.

### Community Concierge (Future)

**The concept:** "I need a plumber this week — who's trusted?" Member users get a filtered, recommendation-weighted search experience. Instead of browsing the full directory, they describe what they need and get a curated shortlist based on neighbor recommendations.

**Why it's different from search:** It's intent-based, not keyword-based. "I need someone to fix my leaky faucet" maps to plumbers, handymen, and general contractors — all filtered by trust score and availability.

**Priority:** Low — requires significant search infrastructure. Worth documenting as north star.

---

## MyLane Integration

User tiers affect MyLane through conditional section rendering. See [MYLANE.md](./MYLANE.md) for the widget architecture.

**Free users see:** All base sections (Greeting, HappeningSoon, NewInCommunity, Discover, etc.)

**Member users see base sections plus:**
- MarketBasketSection (orders, pickup reminders)
- SmartNotificationsSection (matches and alerts)
- Enhanced DiscoverSection (saved filters, Outside Your Lane)
- Enhanced TrustNetworkSection (visual connections)

**Architectural rule:** Member sections are additional components added to the layout. They never replace or modify free-tier sections. Upgrading adds — it doesn't rearrange.

---

## Pricing Considerations

### What We Know

- Must feel like obvious value ("of course I'd pay $5 for that")
- Must not feel like a paywall on core functionality
- Monthly subscription aligns with Stripe Connect billing
- Annual discount option for retention

### What We Don't Know Yet

- Exact price point (needs market testing)
- Whether one member tier or multiple (e.g., Member vs. Premium)
- Whether family/household plans make sense
- Whether there's a community/non-profit discount

### Revenue Model Math (Illustrative)

A community of 5,000 active users with 10% member conversion at $7/month:
- 500 members × $7/month = $3,500/month recurring
- Plus business tier revenue (separate)
- Plus transaction fees on Punch Pass / Market Basket

This is sustainable community infrastructure revenue, not venture-scale growth. That's the point.

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Our Approach |
|-------------|-------------|--------------|
| Crippling free tier | Kills recommendation volume, which is the platform's value | Free tier is genuinely useful |
| "Upgrade to see who recommended" | Weaponizes trust signals | Trust signals are always visible to everyone |
| Paywalling search/filter | Basic discovery is a right, not a premium feature | Core search/filter is free; saved filters are member |
| Showing ads to free users | Conflicts with trust-first mission | No advertising, ever |
| Nagging upgrade banners | Erodes trust and feels desperate | Member features are visible but not pushy |
| Hiding content behind walls | Makes the platform feel empty for free users | All content is visible; member tier adds convenience features |

---

## Relationship to Business Tiers

User tiers and business tiers are independent but complementary:

- A free user can interact with a Partner-tier business
- A Member user gets the same business listing as a free user — just with better tools to find and engage with it
- Business features (analytics, event publishing, staff management) are business-tier gated
- User features (Market Basket, smart notifications) are user-tier gated
- Neither tier system affects the other's experience

---

## Implementation Timeline

| Phase | What | When |
|-------|------|------|
| **Now** | Document philosophy, capture feature ideas (this doc) | ✅ Done |
| **Phase 1** | Build MyLane with section architecture that supports future tiers | 🔜 Next |
| **Phase 2** | Wire Stripe Connect for payments infrastructure | After MyLane |
| **Phase 3** | Implement member tier (Market Basket + Smart Notifications first) | Post-pilot |
| **Phase 4** | Expand member features based on user feedback | Ongoing |

---

## Open Questions

1. **Should member benefits be community-specific?** If LocalLane expands to multiple communities, do member perks differ by region?
2. **Is there a "founding member" incentive?** Early adopters who pay during pilot get lifetime discount or perks?
3. **How do we handle gifting?** Can someone buy a membership for a neighbor?
4. **Should businesses be able to offer member-only perks?** A coffee shop gives 10% off to LocalLane members? This could be powerful but needs careful design to avoid feeling like advertising.

---

## Shareable Recommendation Links (Related Feature)

While not strictly a user-tier feature, this is closely related to community growth:

**Concept:** Every business profile gets a shareable "Recommend" URL: `locallane.app/recommend?businessId=xxx`

**Use case:** A business owner texts a happy customer: "Hey, would you mind sharing your experience on LocalLane?" The link takes them straight to the Nod/Story flow. If not logged in, they create an account first.

**Growth engine:** One link = one recommendation + one potential new user.

**Implementation:** The route already exists (`/Recommend?businessId={id}`). The enhancement is making it work for unauthenticated users — show the recommend flow, prompt account creation before submission, then complete the recommendation.

This should be spec'd separately but is noted here because it's a key part of the user acquisition strategy that feeds into the tier conversion funnel.

---

*This spec is part of the LocalLane Spec-Repo. Reference before implementing.*
