# LocalLane Core Architecture — v4.1

> Plain-English blueprint for how LocalLane works going forward.
> Date: 2026-04-22
> Version: 4.1 (minor refinement of v4; supersedes v1, v2, v3, v4)
> Incorporates v4 changes plus: per-entity $9 model ($9 per user account + $9 per business), LocalLane brand exempt from its own fee, processing fees as visible line items

---

## Changes from v4 to v4.1

- **Membership is per-entity**, not "first entity free then +$9 each." Every account pays $9; every business pays $9. Clean attribution.
- **LocalLane-the-brand is exempt** from its own $9 fee. Charging itself would be circular.
- **Processing fees (Stripe) are visible line items** in all financial dashboards, not silently absorbed.
- Doron's total is $36/mo ($9 personal + $9 TCA + $9 Recess + $9 Consulting; LocalLane and Mycelia exempt).

---

## TL;DR — The whole thing in 90 seconds

**LocalLane is an ocean.** One legal entity (Mycelia, LLC) holds it all. Out of that ocean flow streams: LocalLane, TCA, Recess, Consulting — each a public-facing *business* with its own brand, tools, and relationships. Hidden entities (like Mycelia itself) exist operationally but aren't listed in the directory.

**A user can own many businesses, or none at all.** Mycelia's tree is one user's. Runhub NW is another user's. A yoga teacher with no business of her own is a third user, working across other people's businesses via authority granted by those owners.

**$9/month per entity is the cost of being on LocalLane.** Every member account pays $9. Every public-facing business pays $9. LocalLane the brand is exempt (circular charge). Hidden entities (holding companies) don't count. Network memberships (like Recess's $45 Community Pass) are separate — paid to the operating business, not to LocalLane.

**Each business is its own room with tools inside.** Tap "TCA" and you enter TCA's dashboard. Tools inside: Desk (work management), Finance (books), Events, and others. Tools determine the shape of the business — no archetype labels. A yoga teacher's Desk and a contractor's Desk are the same tool, different fits.

**Networks are one architecture with three settings.** Every network is operated by a business. Settings: cost (free/paid), access (public/private/hybrid), discovery mode (list/map/events). Recess runs on the same code as Runhub NW and Harvest — just with more settings turned on.

**Every space has a gardener.** The owner by default. A specialist can be invited (a CPA gardens a Finance space). Gardeners get paid. Same rate per space across all levels — scale by gardening more or opening more businesses, not by extracting more per space.

**Cockpit vs. theme.** Cockpit = how you navigate (Spinner, Bearing). Theme = what it looks like (Gold Standard). They're independent and already decoupled in code.

**Living tiles, not photos.** Public-facing surfaces are compact tiles showing what's active right now. Photos and rich content appear on drill-in.

**Dark until explored.** DEC-117 preserved. Discover exists as a navigable space — it's the entry point to explore from, not a buffet of everything all at once. Things become more prominent as users engage with them.

**What's built first (Round 1 Business Foundation):** Six phases, starting with Phase 0 decisions already made, then schema foundation, reparenting machinery, business switcher, Desk rename and business-scoped rendering, membership gate, onboarding fork. User public pages come in Round 2. Authority model in a dedicated round thereafter.

---

## 1. The Core Ideas

Seven principles that everything flows from:

1. **A user can own multiple businesses — or none at all.** Users are first-class entities themselves.
2. **Each business is a room with tools inside.** Tools determine the shape; no archetype labels.
3. **Every space has a gardener** — the owner by default, or a paid specialist invited by the owner.
4. **Networks are unified** — one architecture with three settings (cost, access, discovery mode), always operated by a business.
5. **$9/month is the ante to operate.** Exploration is free. Public-facing presence costs $9.
6. **The app separates navigation (cockpit) from appearance (theme).** Each is user-selectable.
7. **Living feet** — configuration lives in data, not code. Every capability is reconfigurable without redeploys.

Plus three principles that shape how the above gets built:

- **Transparent economics** — platform books are open. Anyone can see where the money flows. The platform's non-extractive nature is provable, not just claimed.
- **Living tiles, not photos** — public representations are compact status tiles. Depth revealed on drill-in.
- **Dark until explored** (DEC-117) — we don't give everyone everything at once. Discovery is an active movement through the organism, not passive consumption.

---

## 2. The Ocean and the Streams

**The ocean is Mycelia, LLC** — or whichever legal entity a user operates under. It holds the EIN, bank, Stripe anchor, tax obligation.

**The streams are the brands** flowing from that ocean. In Doron's case: LocalLane, TCA, Recess, Consulting. Each is public-facing; each is its own business on the platform.

**Estuaries are where streams meet** — TCA buying memberships from Recess; The Circuit participating in Recess Network. Commercial and network-participation relationships that don't create ownership hierarchy.

**Other users have their own oceans.** Runhub NW is its own oceanic tree, independently operated. Same platform capabilities serve every ocean identically.

---

## 3. The Hierarchy (Doron's specific tree)

```
Mycelia, LLC (parent — hidden from directory — exempt)
│   EIN, Stripe anchor, one bank, consolidated Finance
│
├── LocalLane (child — listed, public — exempt from own fee)
│   The platform brand
│
├── TCA — The Camel Academy (child — listed, public — $9/mo)
│   Life school
│   Commercial relationship: client of Recess
│
├── Recess (child — listed, public — $9/mo)
│   Physical branch + operates Recess Network (commercial, paid)
│
└── Consulting (child — listed, public — $9/mo)
    Personal leadership consulting (withdoron.com brand)

Doron's total: $9 (personal account) + $9 × 3 (TCA, Recess, Consulting) = $36/mo
LocalLane doesn't charge itself. Mycelia is hidden.
```

Networks, members, participating businesses, events all flow as separate relationships from this tree structure.

**Runhub NW's tree** is independent:
```
[Runhub NW owner]
└── Runhub NW (business — listed, public — $9/mo)
    Operates: Runhub NW Running Community
```

---

## 4. Businesses

**What a business holds:**
- Legal identity: legal name, EIN (or sole prop), registered address
- Brand identity: display name, logo, colors, tagline
- Directory content: categories, hours, photos, bio, contact, social links
- `listed_in_directory`: toggle per business. Hidden businesses still exist and operate; just not discoverable.
- `parent_business_id`: optional. One level of nesting supported.
- Stripe connection: inherited from parent, or set directly.
- Location fields already in schema: latitude, longitude, address lines, geocoded_at.
- `location_precision` (new): `exact` / `neighborhood` / `general_area` — for micro-producer privacy.

**No archetype labels.** A business is a business. The tools it has turned on reveal its shape.

**Three kinds of business-to-business relationships:**

1. **Ownership (nesting)** — `parent_business_id`. One level deep in Round 1.
2. **Network participation** — a business opts into another's network (The Circuit joins Recess Network).
3. **Commerce** — invoices and payments between two businesses. No structural relationship; just transactions.

**Business creation:**

When a user creates a first business, a wizard captures legal identity, brand, and directory content. On a second, the wizard asks: "New standalone business, or brand under an existing one?" Children inherit defaults from parent.

Onboarding *suggests* tools based on the user's description of their business. It doesn't prescribe or label. "Sounds like you'll want Desk and Finance — add now or later?" User confirms or adjusts.

**Reparenting exists.** Existing businesses can be moved under a new parent without data loss. Important for Doron's Recess migration under Mycelia.

**Creating additional businesses is always available to members.** Not gated to signup.

**Stale businesses are fine.** A listing without active tools is valid.

---

## 5. Users

**Users are first-class entities, same as businesses.**

A user has:
- Identity: name, avatar, bio
- A **personal page** that can be public or private (toggle per user)
- Optional geo-location with precision controls
- Reviews/vouches from others they've worked with or alongside
- Roles at various businesses (owned, authority-granted, staff, contractor, client, gardener)

**A public user page costs $9/month — the same ante as a public business.** Private pages are free.

**Why users are first-class:** an individual offering to pull weeds, a yoga teacher teaching at three studios, a fractional leader helping run multiple businesses — these are all real participants in the LocalLane economy. They deserve a presence as real as a business's presence.

A user can be:
- A solo operator with just a public user page (the weed-puller)
- An owner of one or more businesses
- A non-owner working at multiple businesses via granted authority (the yoga teacher, the fractional leader)
- All of the above simultaneously

**Round 1 reserves User entity fields for public pages** (bio, latitude, longitude, page_public_toggle) so no migration is needed later. The actual public page surfaces are Round 2 work.

---

## 6. Spaces and Tools

**A space is where work happens.** Three kinds:

**Business spaces — tools inside a business dashboard:**

- **Desk** (formerly Jobsite / Field Service) — work management. Clients, projects, estimates, daily logs, materials, labor, invoices, payments. Universal across archetypes: contractors, CPAs, therapists, yoga teachers, dog walkers, anyone with clients. Renamed in Phase 4 of Round 1.
- **Finance** (business flavor) — the business's books.
- **Events** — events hosted by this business, public or private.
- **Property Pulse** — rentals. Units, tenants, rent, maintenance.
- **Storefront** (future) — e-commerce for retail.
- **Market Stand** (future) — simple product listing + inventory + payment methods for micro-producers (Harvest Network participants).
- **Network** (future, Round 4) — operate your own network (paid or free) with participating businesses and members.

A business turns tools on and off as needed. A business with just a directory listing and no active tools is valid. No tool is required; none is forced.

**Personal spaces — attached to the user:**

- **Home** — MyLane's entry surface. Automatic.
- **Discover** — navigable space. Entry point to explore from. Not a buffet. Dark-until-explored preserved.
- **Personal Finance** — household books.
- **Meal Prep** — family food.
- **Networks** — aggregated view of all networks the user is a member of (visible when they've joined any).
- **Playmaker** — appears when user has parent/coach/player role somewhere.

**Settings** is accessed via avatar in the app header — not on the spinner. Keeps work separate from configuration.

**Future: user-pinned spaces.** A user can decide which spaces appear on their spinner, hiding what they don't use and pinning favorites. This is Round 3+ work.

---

## 7. Networks — One Unified Architecture

Every network is operated by a business. There is no "free-floating community network." Every network has a parent business, even if that business is small and the network is free.

**A network has three independent configuration dimensions:**

1. **Cost:** free or paid
2. **Access:** public, private, or hybrid (public entry, private interior)
3. **Discovery mode:** list / map / events calendar — which surface opens by default

**Any combination is valid.** A network can be free + public + list (simple running group). Paid + hybrid + events (Recess). Free + public + map (Harvest). Paid + private + list (members-only professional network). Settings change; architecture doesn't.

**What configures per network:**
- Name, identity, branding
- Cost settings (price, billing cycle) if paid
- Access settings (who can see what)
- Discovery mode(s) (map / events / list)
- Joy coin rules if using joy coins
- Revenue share terms to participating businesses if any
- Participation criteria (who can join as a participating business)

**Participating businesses:** Opt into a network. Configure their own participation terms (hours accepted, capacity, joy coin cost per visit, etc.). A business can participate in multiple networks.

**Members:** Users who've joined. For paid networks, subscription flows through the operating business's Stripe account.

**Events** belong to networks (through the parent business) or directly to a business. Each event has a `visibility: public/private` setting. Public events appear in global feeds; private events appear only to members.

**Map view** is a first-class discovery mode. When enabled, users browsing the network see participating businesses as pins, tappable to drill into each business's profile. Used natively by Harvest, relevant for Recess, possible for any network.

**Dogfooding principle:** Recess (most complex case) and Runhub NW (simplest case) and Harvest (spatial case) all run on identical infrastructure. Each has different settings turned on.

---

## 8. Authority and Permissions (Round 3+ — Placeholder in Round 1)

**Authority, not ownership, is what matters for access.**

A user can have authority at a business without owning it. A fractional leader helping run another's business needs owner-level access. A yoga teacher at a studio needs scoped access to her classes. Staff need their permission levels. Contractors get invited as contributors.

**Round 1 does not build this model.** Round 1 keeps simple owner-only semantics. The `parent_business_id` and related schema are sufficient for Phase 1. No multi-user-per-business access yet.

**But Round 1 reserves architectural space** so the authority model can land without migration:

- Plan for a `BusinessRole` entity type that links User ↔ Business ↔ permission level.
- Plan for per-tool permissions that read from that role relationship when it exists.
- Tool-level queries in Round 1 use "is owner?" but are structured so they can later read from a richer permission check.

**When the authority round happens** (probably between Round 2 and Round 3), roles become first-class: Owner / Fractional Admin / Staff / Contractor / Client / Gardener. The yoga teacher, fractional leader, and CPA-gardener cases all get proper implementation.

---

## 9. Membership — $9 Per Entity

**$9/month is the cost of each entity on LocalLane.** Not an ante that covers multiple things — a per-entity fee that each entity pays from its own books.

**What pays $9:**
- Every user's personal LocalLane account (paid as a personal expense)
- Every public-facing business with full business tools (paid as a business expense on that business's books)

**What doesn't pay $9:**
- LocalLane-the-brand (exempt — it is the platform; charging itself is circular)
- Hidden entities like holding companies (they don't have public-facing presence; Mycelia, LLC is an example)
- Non-members browsing without an account (they can still view the directory, events, and public network pages)
- Light-touch guest access (parent viewing an invited child's Playmaker team; client paying an invoice via hosted Stripe link)

**Why per-entity:**

Each business should pay its own platform fee from its own books. This matches how businesses actually work: Red Umbrella LLC pays its own platform fee as a business expense, deductible on its own tax return. Bari doesn't pay his business's platform fee out of personal checking. His LLC does.

At the accounting level: each business's Finance dashboard shows its own $9 as an expense line item. At the bank level: all the money physically flows through the parent entity's Stripe account and lands in the parent's bank account. Attribution happens in software; banking happens at the parent.

**Example — Doron's case:**
- Doron's personal account: $9/mo (personal expense)
- TCA: $9/mo (business expense on TCA's books)
- Recess: $9/mo (business expense on Recess's books)
- Consulting: $9/mo (business expense on Consulting's books)
- LocalLane: exempt
- Mycelia: hidden, doesn't apply
- **Total flowing from Mycelia's bank: $36/month**

**Example — Bari's case:**
- Bari's personal account: $9/mo (personal expense)
- Red Umbrella: $9/mo (business expense on Red Umbrella's books)
- **Total: $18/month**

**Example — solo yoga teacher with no business:**
- Personal account: $9/mo
- **Total: $9/month**

**Network memberships are separate:**

- Recess Community Pass: $45/mo paid to Recess the business (not to LocalLane)
- Future networks set their own pricing
- A user can pay $9 for their LocalLane account AND $45 for a Recess membership simultaneously — different things.

**Dynamic gating per action:** Every build includes a living-feet flag asking "does this action require an active member?" Answered space-by-space. If yes and the user isn't a member, a prompt appears: *"Become a LocalLane member to do this."*

**Cancellation:** Any time. Period-end access. No refunds, no prorations. Cancel the personal account or any business independently.

**Billing cycle:** Starts on signup date. Stripe native.

**DEC-115 deprecation:** The old per-workspace tier model (free/help/full) is retired cleanly since no one is paying on it yet. Anyone using tools today continues as-is until they hit a gated action that triggers a $9 prompt.

**Books open:** Platform financials are transparent. Anyone can see where money flows. The non-extractive principle is provable, not just claimed.

---

## 10. Cockpits and Themes

**Cockpit = how you navigate.** Spinner (horizontal carousel), Bearing (compass of spaces). Spinner is default. Future cockpits possible (voice, minimalist, spatial).

**Theme = what it looks like.** Currently Gold Standard (amber on dark). Future themes for accessibility, life stage, brand expression.

**Already decoupled in code.** `data-cockpit` and `data-theme` applied independently. Compass + Fallout combination renders correctly today.

**Cockpit extends inward into spaces.** A business's dashboard in Spinner renders as tile chips; in Bearing as compass bearings. Each cockpit carries its visual language into the spaces it navigates through. Cockpit-agnostic structure, cockpit-specific chrome.

Cockpits also extend **outward** into business-depth — each cockpit is responsible for rendering how a multi-business user switches between their businesses, using the cockpit's own visual vocabulary. (See DEC-168.)

**Every space has two layers:** structure (data, actions, entities — cockpit-agnostic) and presentation (how structure renders — cockpit-specific). New spaces are designed structurally once, then rendered per cockpit.

**Physics toggle** is experimental. Pending. May grow or retire.

---

## 11. Stripe and Payments

**One Stripe Connect account per parent business.** Children inherit.

Stripe-hosted onboarding collects EIN/SSN, bank info, ID verification. LocalLane never sees PII.

**Two payment flows:**

1. **Platform fees ($9/mo per entity)** — members pay LocalLane. Flows to Mycelia's Stripe account as platform operator.
2. **Direct invoicing + network membership payments** — clients pay businesses directly; network members pay the operating business. LocalLane is the bridge, not merchant of record.

**ACH enabled day one.** Card (~2.9% + $0.30) or bank transfer (~0.8%, capped ~$5). Stripe configuration, not separate build.

**Processing fees are always visible.**

Every financial dashboard shows both what was charged and what was collected. Nothing silently absorbed.

Example: A $45 Recess membership is charged on card.
- Recess's Finance shows: +$45 revenue from subscriptions
- Recess's Finance shows: -$1.61 Stripe processing fee (expense)
- Net effect on Recess's books: +$43.39

This is honest accounting. Books open. Nothing hidden in a processing-fee black box.

**Per-entity payment attribution.**

For a holding-company structure (like Doron's Mycelia tree), each child business pays its own $9 platform fee as its own expense. Physical money moves through the parent's Stripe; accounting records the expense at the child level.

- Mycelia's bank debits $36/month (3 businesses at $9 + Doron's personal $9 account; LocalLane exempt)
- Doron's personal books show: $9 expense
- TCA's books show: $9 expense
- Recess's books show: $9 expense
- Consulting's books show: $9 expense
- Mycelia consolidated view: $36 total, appropriately tagged

Banking happens at the parent. Accounting happens at each entity.

---

## 12. Finance

One tool, two flavors.

- **Personal Finance** — attached to user. Household money.
- **Business Finance** — attached to a business. Business books.

Same engine, different flavor.

**Sibling blindness.**

Each child business's Finance dashboard is scoped to that business only. Recess's Finance shows Recess's books. LocalLane's shows LocalLane's. Siblings are blind to each other — when Doron is inside Recess, he doesn't see TCA's revenue.

The parent (Mycelia) sees all streams rolled up into consolidated books. That's the tax-filing view. Everything rolls up there.

A Finance gardener invited to a specific child tends only that child's books — not siblings. CPA gardens Recess? They see Recess only.

**Roll-up:** Mycelia's Finance shows consolidated revenue/expenses/profit across all child businesses.

**Physically, all money flows through Mycelia's Stripe account and lands in Mycelia's bank.** The business dashboards are accounting views; the bank is one shared pool. Revenue attribution happens at the business-logic layer (which brand earned which payment), not at the bank level.

**Bank data:** Statement upload now. Plaid in Round 3 once membership revenue funds the infrastructure.

**CPAs as Finance gardeners:** Invited by business owners, scoped access, plug-and-play.

---

## 13. The Directory — Living Tiles

Every listed business (public) appears as a **living tile** in the directory. Not a photo-heavy profile — a compact tile showing what's active right now: open/closed, accepting Recess passes, joy coin balance if relevant, current status.

**Drill-in reveals full content:** photos, detailed profile, contact info, events, services.

**Visibility is immediate.** A business appears as soon as basic onboarding completes. Stripe setup shown as a badge ("Accepting payments") but doesn't gate listing.

**Parent-child shown subtly** ("A Mycelia, LLC brand" — optional).

**Network participation shown as badges** (Recess, Harvest, etc.).

**Tile-first applies to users too** when we build user public pages in Round 2. A solo yoga teacher appears as a living tile — "Teaching at Recess Wed 6pm" — drill-in to see full bio and reviews.

**Browsing remains free for everyone.**

---

## 14. MyLane Navigation

**At home:** The cockpit shows:
- Your businesses as chips/tiles/bearings (per active cockpit)
- Your personal spaces (Discover, Finance, Networks if joined, Playmaker if role exists)
- Settings via avatar in header (not on main cockpit)

**Spinner dissolves as you descend.** Tap Businesses → secondary spinner of individual businesses. Tap TCA → TCA dashboard. One level at a time.

**Breadcrumb appears when past home:** `MyLane › TCA › Desk`. Tappable segments; jump back to any level with one tap.

**MyLane button always in header.** One tap returns home regardless of depth.

**Context is always clear.** Visual identity carries through every screen.

---

## 15. Living Feet

Everything above is reconfigurable without code changes:
- Businesses add/remove tools
- Tools are gated at the action level (not hardcoded globally)
- Owners invite/revoke gardeners
- Gardeners customize per-client
- Users switch cockpits and themes
- Users join/leave networks
- Users add businesses over time
- Directory visibility toggles freely
- Page visibility (public/private) toggles freely

Configuration lives in data, not code. Every check happens at render/action time, not at login.

---

## 16. Physical Marketing

LocalLane appears in Eugene's physical world through stickers. Chanterelle mushroom with root network and path. Every active member business gets stickers. Multiple formats, QR code to business profile, optional tenure mark, variant stickers for gardeners and network stewards. Architectural element, not just marketing.

---

## 17. Round 1 — The Business Foundation

Six phases, adapted from Hyphae's design review (rejecting the five-phase version I proposed earlier as too big and mis-sized).

### Phase 0 — Decisions (complete)

All six decisions Hyphae required are settled:

- **Business implementation:** scoped-peer (not nested container). Tools get `business_id` field; render filtered to active business.
- **DEC-115 transition:** clean deprecation. No one is paying; no grandfather complexity.
- **Billing math:** $9 per entity. Each user account, each public-facing business. LocalLane-the-brand is exempt (circular). Hidden entities don't count. Doron's case: $36/mo ($9 personal + $9 each for TCA, Recess, Consulting).
- **DEC-117 preservation:** Discover remains a navigable space but "dark-until-explored" holds. Entry point, not buffet.
- **Settings placement:** stays in avatar. Not on cockpit. Personal profile access secondary (Round 2).
- **FS rename communication:** Doron will explain in person to Bari and Dan. No in-app notice needed for such small user base.

### Phase 1 — Schema Foundation (no UI changes)

- Add to Business: `legal_name`, `display_name`, `listed_in_directory`, `brand_color`, `logo_url`, `tagline`, `location_precision`, `parent_business_id`. All nullable.
- Add to User: `bio`, `latitude`, `longitude`, `page_public_toggle`, `geocoded_at`. All nullable.
- Add `business_id` (nullable) to all 10 FieldService entities: FieldServiceProfile, FSClient, FSProject, FSEstimate, FSDocument, FSPermit, FSDailyLog, FSMaterialEntry, FSLaborEntry, FSPayment.
- Create `AuditLog` entity: entity, entity_id, action, old_value, new_value, user_id, timestamp.
- Add `archived_at`, `archived_by` to FS profiles and Business.
- Commit `src/scripts/migrations/TEMPLATE.ts` showing dry-run + audit-log pattern.
- **First task: produce production record manifest** — Hyphae queries Base44 to produce verified list of all Business records (with owner, state, flags), all FieldServiceProfile records (with owner, state), User counts, Doron's specific records. Flags orphans.
- **Backfill dry-run script:** For each FieldServiceProfile, map to its owner's Business via owner_user_id → businesses they own. Report ambiguous cases (user with >1 business requires manual mapping).
- Deploy schema. Verify Bari and Dan unaffected.

### Phase 2 — Reparenting Machinery (no UI changes)

- `reparentBusiness()` server function with idempotent steps, audit logging, rollback on failure.
- Dry-run framework.
- First real reparent: create Mycelia, LLC record; reparent Doron's Recess under it. This is the production smoke test.

### Phase 3 — Business Switcher + Directory Toggle

- MyLane shell switcher for multi-business users. Persist active business to localStorage.
- Fix `MyLaneDrillView.jsx:53` to use selected business instead of `[0]`.
- Directory query respects `listed_in_directory` (separate from `is_active`).
- Settings toggle for listing visibility.
- Single-business users see no change; multi-business users get the switcher.

### Phase 4 — Desk Rename + Business-Scoped Rendering

Only after FS has `business_id` and switcher exists.

- Rename pass: Jobsite → Desk (92 strings across ~40 files).
- Render Desk inside business-scoped view — the "business dashboard" implemented as filter + layout, not new container component.
- Test against Bari's live workspace first.
- Doron communicates rename in person to Bari and Dan.

### Phase 5 — Membership Gate

- Add `active_member` field on User.
- Gate at action level (per-space configuration). Don't gate browsing.
- Pop-up prompt for non-members hitting gated actions.
- Stripe Connect wiring comes in Round 2 — Phase 5 can operate with temporary manual override until then.

### Phase 6 — Onboarding Fork + Tool Suggestions + Personal Spaces Cleanup

- Onboarding asks: "new business or brand under existing?"
- Tool suggestions based on user's description.
- Verify Home and Settings are correctly already in place. Discover remains a space but respects DEC-117 (dark-until-explored, not auto-populated).

### Phases as Hyphae-sized sessions

Each phase is scoped to fit a single focused Hyphae session (under 400 messages). Between phases, you test in the browser and verify before proceeding.

---

## 18. Deferred

Not blocked. Not built yet.

- Co-owners of a single business
- Grandchildren (deeper nesting than one level)
- Auto-categorization and finance automation
- Multi-gardener coordination in single scope
- Additional cockpits beyond Spinner and Bearing
- Additional themes beyond Gold Standard
- Migration flow between network configurations (free ↔ paid)
- User-pinned spaces on the cockpit
- Authority model (Fractional Admin, Staff, Contractor, Client, Gardener roles) — dedicated round after Round 2
- User public pages surfaces — Round 2
- Network entity as a proper thing — Round 4
- Market Stand tool for micro-producers — Round 4
- Map view discovery mode — Round 4
- Plaid for live bank data — Round 3

---

## 19. Known Issues — Separate Investigation (Completed)

Hyphae's directory investigation (April 22, 2026) resolved:

- Recess exists as a Business record with banner and `network_ids: ["recess"]` intact. Hidden from MyLane by `[0]` hardcode, but discoverable in Directory surface.
- Directory navigation bug: affordance gap on mobile. Overlay has no visible close button; users can't navigate away without force-quit. Fixable with a persistent close button on OverlayContainer.
- No Network entity exists in code — "networks" are currently a tag system (`network_ids` array on Business). Proper Network entity is Round 4 work.
- Business Settings has no directory visibility toggle today — `is_active` is conflated with delete.

**Pre-Round-1 patch (optional):** Ship the close-button affordance on OverlayContainer. Tiny PR, <1 hour, fixes active mobile pain, doesn't touch Round 1 work. Doron's call whether to patch now or bundle into Phase 3.

---

## 20. Open Questions — Things We'll Learn by Doing

- What does Desk look like when a CPA uses it for client engagements vs. a contractor for projects?
- How do network memberships show in Personal Finance (as recurring expenses)?
- What's the welcome-back UX for reactivated members?
- How deep does cockpit-aware rendering go into specific spaces?
- Does LocalLane-the-brand have tools beyond Finance, or is its "work" just running the platform?
- When the authority model lands, what are the default permission bundles per role?

---

## What This Document Is

A **blueprint**. Architecture decisions in plain language. Reference for all future design, build, and strategy work.

Not a technical spec (no schemas, component names).

Not comprehensive (many details layer on top as we build).

**Next steps:**
1. Doron reviews v4 at own pace. Flags anything off.
2. Hyphae runs Phase 1 build in fresh conversation.
3. Phase 1 produces manifest + schema foundation; verifies no breakage.
4. Phase 2-6 follow sequentially, roughly 1-2 sessions apart for testing between.
