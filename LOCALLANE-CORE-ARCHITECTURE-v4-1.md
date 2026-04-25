# LocalLane Core Architecture — v4.1

> Plain-English blueprint for how LocalLane works going forward.
> Date: 2026-04-22 (original), amendments 2026-04-24
> Version: 4.1 (minor refinement of v4; supersedes v1, v2, v3, v4)
> Incorporates v4 changes plus: per-entity $9 model ($9 per user account + $9 per business), LocalLane brand exempt from its own fee, processing fees as visible line items

## Revision history

- **2026-04-22** — v4.1 originally authored.
- **2026-04-23** — Section 10 amended post-Phase 3 Build 1 (outward cockpit extension, DEC-168).
- **2026-04-24** — Amendments: new subsection 3.1 (Region as scoping layer); major rework of Section 6 (Spaces belong to scopes, `enabled_spaces`); new subsection 10.1 (Cockpits as folder renderers); new subsection 13.1 (Hidden entities); full rewrite of Section 14 (folder-tree MyLane navigation); full restructure of Section 17 (Phases 1–10+). New Sections 18 (Regions & Federation), 19 (Direct Doors), 20 (Resurface Before Rebuild) inserted; the prior Section 18 (Deferred), 19 (Known Issues), 20 (Open Questions) are preserved and renumbered to 21/22/23 respectively. DEC-169 through DEC-173 logged in `DECISIONS.md`.

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

### 3.1 — Region as a scoping layer

Above Business and User sits Region. Every user has a primary region. Every business belongs to a region. Every event belongs to a region. Region is a first-class entity, not a derived filter on address strings.

Round 1 begins with a single region: Lane County, Oregon. All existing users, businesses, and events are backfilled to this region. As LocalLane grows beyond Lane County, each new community becomes its own region — with its own directory, its own events, its own network roster, and eventually its own local lead.

Regions have lifecycle states:

- **Seed.** One or more interested neighbors have signed up, but the community is not yet active. Directory and events for this region are empty. The region exists as a placeholder so that users landing there have a home. Seed users can use their Personal surface and browse other active regions.
- **Sprouting.** A threshold of interest has been reached — some number of neighbors plus a local lead willing to nurture the region. Businesses can begin joining. The directory populates. Events begin.
- **Established.** The region functions fully. Full directory, active events, networks operational, community pass functional.

The transition between states is gardened, not automated. A human at LocalLane (or eventually, a regional representative) decides when a seed becomes sprouting, and when sprouting becomes established. Living feet: the structure supports any pace a community chooses.

This is how the organism spreads. A user visits Eugene, experiences LocalLane, returns home to Salem where no region yet exists. Their interest creates Salem-as-seed. They invite their neighbors. Salem grows into sprouting, then established — at its own pace, with its own local shape. LocalLane does not "expand to Salem" by corporate decision. Salem grows into LocalLane by community energy.

The same data model supports the ten-year arc toward federation. Today all regions share one database, scoped by `region_id`. Tomorrow, any region that grows enough can be peeled off to its own instance, with federation protocols syncing aggregate data back to a shared directory layer. No rewrite required — the `region_id` on every record is already the seam.

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

### 6.1 — Spaces belong to scopes

Every space exists within a scope. The same space (Finance, Events) renders differently depending on which scope it's inside.

Scopes, in order of containment:
- **Personal scope.** Spaces belonging to the user themselves.
- **Business scope.** Spaces belonging to a specific business the user operates.
- **Network scope (Round 4+).** Spaces belonging to a network the user has joined.

A user with three businesses has Finance inside Personal (their personal books), Finance inside Business A, Finance inside Business B, Finance inside Business C — four distinct Finance surfaces, each rendering only its scope's data.

### 6.2 — Personal spaces

The defaults inside Personal:

- **Home.** The user's landing surface. Attention items, this week, personal organism pulse.
- **Personal Finance.** Books scoped to the user.
- **Meal Prep.** Household food planning.
- **Playmaker (contextual).** Appears only for users with a Playmaker role (parent, coach, player).
- **Networks (contextual).** Appears for users who have joined any network — one folder that nests their joined networks.

Additional personal spaces may be enabled over time. The default set stays small.

### 6.3 — Business spaces

The defaults inside a business, at creation:

- **Desk.** The operational home of the business. Clients, projects, logs, estimates, documents, people. Combines what was previously called Jobsite and Home inside a business — there is no separate "Home" inside a business because Desk is the home of business work.
- **Finance.** Books scoped to this business.
- **Profile.** The business's public-facing directory presence. Logo, tagline, services, gallery, contact, financing, blog. Eventually, the full public website.
- **Settings.** Business-level configuration. Listing visibility, branding, enabled spaces, billing.

Additional business spaces, available via "+ Add Space":

- **Events.** Event organizing for businesses that run them (Recess, Harvest Network member farms, art studios).
- **Network.** For businesses that operate their own network (Recess, future Harvest Hub, future Creative Alliance).
- **Property Pulse.** For businesses managing rentals.
- **Storefront (future).** For businesses with physical retail or e-commerce.
- **Kitchen (future).** For food businesses with menu, inventory, supply chain.

A business starts lean with four spaces. As its needs grow, it adds spaces. Each space is data — enabled or disabled per business via its `enabled_spaces` field — not code. Adding a space for your business does not require a deploy.

### 6.4 — The `enabled_spaces` field on Business

Business gains a field: `enabled_spaces: array<string>`. Default at creation: `["desk", "finance", "profile", "settings"]`. Users can add or remove spaces in business Settings. Data for disabled spaces is retained (not deleted), simply hidden from the spinner until re-enabled.

Backfill strategy for existing businesses: inspect actual data. A business with FS data gets Desk. A business with finance transactions gets Finance. Every business gets Profile and Settings. Additional spaces added by explicit inspection of what each business currently uses.

### 6.5 — Dark-until-explored applied to spaces

Per DEC-117, spaces follow dark-until-explored. A business that has never used Events doesn't see Events in its spinner until explicitly adding it. A user who has not joined any network doesn't see Networks in their Personal folder. The cockpit shows what the user has, not what they could have.

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

### 10.1 — Cockpits as folder renderers

The cockpit is the UI chrome for navigating a folder tree. Each folder in the tree contains either sub-folders or a workspace. The cockpit renders whichever it finds.

At the root, the cockpit shows universal folders (Directory, Events, Personal, Businesses) plus any contextual folders the user has earned (Playmaker if they have a role, Networks if they've joined any, Admin if they are one).

Inside a folder, the cockpit shows that folder's contents — which may themselves be folders (Businesses → your businesses → one business → its spaces) or workspaces (a specific space's tools).

Workspace renders below the cockpit only at the leaf level. At non-leaf levels, the area below the cockpit shows a preview pulse — living-tile language describing what's alive in the folder currently centered on the spinner. This gives the user context about what they're navigating toward before they enter.

This extends section 10's principle of "spinner dissolves as you descend" to include lateral dissolution: the cockpit can also dissolve into a sibling-selection mode (the business switcher, per DEC-168) when the user wants to switch which branch of the tree they're operating within.

Different cockpit styles (Spinner, Bearing, future Voice) each render folder navigation in their own visual language. The folder tree is identical across cockpits; the rendering differs. A user switches cockpits in Settings and sees the same underlying structure through a different lens.

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

**Coverage as structured data (DEC-174).** `service_area` on Business is `array<slug>` (curated Lane County town list in Round 1). Every searchable directory field is structured for the same reason: freeform text doesn't filter. Slugs are forward-compatible with the Region/Town promotion in §18 — when towns become first-class entities in Phase 6, `array<slug>` becomes `array<id>` with no Business-side schema change. Legacy freeform strings on pre-Build-E records are preserved verbatim for read-only display; owners migrate themselves via the Settings editor.

**Visibility is immediate.** A business appears as soon as basic onboarding completes. Stripe setup shown as a badge ("Accepting payments") but doesn't gate listing.

**Parent-child shown subtly** ("A Mycelia, LLC brand" — optional).

**Network participation shown as badges** (Recess, Harvest, etc.).

**Tile-first applies to users too** when we build user public pages in Round 2. A solo yoga teacher appears as a living tile — "Teaching at Recess Wed 6pm" — drill-in to see full bio and reviews.

**Browsing remains free for everyone.**

### 13.1 — Hidden entities

`listed_in_directory: false` is the mechanism for hidden entities. Mycelia, LLC is the first hidden entity — visible operationally to its owner (for consolidated books and tax filing) but absent from the public directory and from other users' surfaces. No separate `is_hidden` field is needed.

Directory queries must filter on `listed_in_directory !== false`. The owner's own surfaces do not apply this filter — owners see their full tree including hidden entities.

---

## 14. MyLane Navigation

MyLane is the user's navigable view of their slice of the organism. It is structured as a folder tree. Every path through MyLane is a path through the tree. The URL reflects the path.

### 14.1 — The folder tree

At the root:
- **Directory** (universal). Browse businesses across the organism, scoped to the user's region by default.
- **Events** (universal). Browse events across the organism, scoped to the user's region by default.
- **Personal** (universal for members). The user's personal spaces: Home, Personal Finance, Meal Prep, Networks (if joined), Playmaker (if applicable).
- **Businesses** (universal for members). The user's businesses. Nested: each business is a folder containing that business's spaces.
- **Admin** (contextual). Appears for users with admin role. The bird's-eye admin hub.
- **Playmaker** (contextual). Appears for users with any Playmaker role.
- **Networks** (contextual). Appears when the user has joined any network. Nests their joined networks.

### 14.2 — Header elements

- **Left: LocalLane logo.** Tap returns to root at any depth.
- **Right: Frequency Station glyph, small gap, avatar.** Frequency Station is tier-gated in its full functionality; avatar opens settings, themes, account, feedback (when resurfaced), sign out.

### 14.3 — Addressable navigation (direct doors)

Every folder and every workspace in MyLane is addressable by URL. Full paths include the scope's slug. Examples:

- `/mylane` — root
- `/mylane/personal` — Personal folder
- `/mylane/personal/finance` — user's Personal Finance workspace
- `/mylane/businesses` — Businesses folder
- `/b/red-umbrella` — Red Umbrella's folder (or its public profile, depending on auth)
- `/b/red-umbrella/desk` — Red Umbrella's Desk workspace (authenticated operators)
- `/b/red-umbrella/profile` — Red Umbrella's Profile workspace / public page

The `/b/{slug}` form is the canonical direct door into a business. It renders public profile for unauthenticated visitors and tools for authenticated operators — same URL, auth-aware rendering.

URL is the primary source of scope. `useActiveBusiness` and related hooks read from URL first, localStorage second.

### 14.4 — Descent, return, and lateral switching

- **Descent:** tap the centered folder in the cockpit to enter it. The cockpit reforms to show that folder's contents.
- **Return:** tap the LocalLane logo (back to root) or the back affordance (up one level) to ascend.
- **Lateral switching within a folder:** per DEC-168, tap the space-name pill to switch which sibling branch you're operating within, using the cockpit's native visual language. This is most useful for business switching but applies anywhere lateral movement makes sense.

### 14.5 — Workspace at the leaf

When the user descends to a space (leaf), the cockpit stays at the top of the viewport as the space selector within that business. The area below becomes the actual workspace. Swipe / tap to switch between spaces within the business; the workspace below updates accordingly.

Only at leaves does the cockpit occupy the same screen as workspace content. Above leaves, the screen is cockpit + preview pulse.

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

**Phase 1 (shipped).** Entity schema foundation. Nullable `business_id` on all FS entities. `AuditLog` entity. Public-page-reserved fields on User. Brand fields on Business.

**Phase 2 (shipped).** Schema foundation + reparenting. Mycelia, LLC created as hidden parent. LocalLane, TCA, Recess reparented under it. Red Umbrella established as Bari's first-class business. Bari flagged `is_legacy_user: true`.

**Phase 3 (shipped).** Cockpit-native business switcher (DEC-168). Shared `useActiveBusiness` hook. `MyLaneDrillView` drill fix. OverlayContainer close button. TDZ patch, commit-handler patch.

**Phase 3.5 (upcoming — direct doors foundation).** Slug backfill and reserved-word enforcement. `/b/{slug}` path routing. Public vs authenticated rendering on business URLs. Minimum-viable public Profile view. `react-helmet` meta tag management. Bari's first shareable URL.

**Phase 4 (upcoming — folder architecture).** Top-level cockpit restructure (universal + contextual root folders). Nested navigation (root → folder → sub-folder → workspace). Home→Desk collapse. Preview pulse at non-leaf levels. Spaces add/remove mechanic. Default business spaces at creation. Directory toggle (formerly Phase 3 Build 2, absorbed here). Desk rename pass (92 strings across ~40 files).

**Phase 4.5 (upcoming — admin & feedback).** Admin root folder (contextual, Doron-only until authority model extends). Admin lens inside every space (authority-aware affordances). Feedback resurfaced as persistent affordance. Other buried surfaces resurfaced as found.

**Phase 5 (upcoming — custom domains & profile growth).** Custom domain support per-business (white-glove config). Profile space grows to full business-website capability. SSR or pre-render strategy for SEO-dependent businesses. Business-website replacement pitch becomes deliverable.

**Phase 6 (upcoming — region foundation).** Region entity added. Backfill Lane County on all existing records. Region-scoped queries on Directory and Events. Region lifecycle states (seed, sprouting, established). No cross-region UI yet, no regional leads — just foundation.

**Phase 7 (upcoming — membership gate).** Legacy grace period. $9 membership activation. $18 personal assistant tier. Workspace billing for revenue tools. DEC-115's gating pattern live.

**Phase 8+ (upcoming — national expansion).** Cross-region browsing UI. Regional lead authority (specialization of the gardener/authority model). Regional branding overlays. Seed-to-sprouting transition automation (or semi-automation).

**Phase 10+ (ten-year arc).** Federation. Regional instances on local hardware. Aggregate sync protocols. Community-owned data.

### Phases as Hyphae-sized sessions

Each phase is scoped to fit a single focused Hyphae session (under 400 messages). Between phases, you test in the browser and verify before proceeding.

---

## 18. Regions & Federation

### 18.1 — Region as data

The Region entity has these fields at minimum: `name`, `slug`, `state`, `founded_date`, `region_lead_user_id` (nullable), `is_active`, `lifecycle_state` (seed / sprouting / established), `created_date`.

Every User has `region_id` (their primary region). Every Business has `region_id` (the region they operate in). Every Event has `region_id` (the region they happen in). Every Network may have `region_id` (for regional networks) or be cross-regional (national networks).

Round 1 begins with one region: Lane County, Oregon. All existing records backfill to it. This is a Phase 6 migration (as restructured in Section 17).

### 18.2 — Region-scoped queries

Directory and Events default to querying within the user's primary region. Cross-region browsing is allowed via explicit action — a toggle or a path-prefixed URL like `/r/multnomah/directory`. The default experience is local; the optional experience is broader.

### 18.3 — The carried-in-the-wind mechanic

When a user accesses LocalLane from a location outside any active region, they are not turned away. They become a seed for their location. Their interest plants a region. The region exists in seed state until threshold is reached and a local lead is identified.

This is the growth mechanic. The organism spreads by interest, not by corporate expansion. A visitor to Eugene who experiences LocalLane and returns home becomes the founder of their hometown's LocalLane simply by caring about it.

### 18.4 — Region leads

When a region reaches sprouting state, it needs a local lead — a neighbor willing to cultivate the community within it. Region leads have elevated authority within their region (can curate, invite, moderate) but do not have authority outside it. This is an instance of the general authority/gardener model parked in Section 8, scoped to region.

### 18.5 — Federation (Round 10+)

Long-term, each active region may choose to run its own LocalLane instance on modest local hardware — a community data center — federated to the wider organism via aggregate sync protocols (similar in spirit to Mastodon or Matrix, adapted to the organism's shape). This is explicitly deferred to Round 10+. What Round 1 must do is make it possible: every record is tagged with `region_id`, so a region's data can be extracted cleanly when the time comes. No rewrite required.

### 18.6 — The brand

"LocalLane" operates nationally by letting "local" mean the user's local. Each region's LocalLane is that region's LocalLane. Regional tagline variants are permissible (LocalLane · Lane County, LocalLane · Multnomah County) without changing the brand. The word "Lane" carries the semantic of "a way, a path, your way" — naturally scaling beyond its Lane-County origin.

---

## 19. Direct Doors

### 19.1 — Purpose

Every business (and eventually, every user and every network) in LocalLane has an addressable URL that serves as both a public-facing surface for unauthenticated visitors and an authenticated entry for operators. These are called direct doors. They make LocalLane addressable the same way the open web is addressable.

Direct doors serve users as much as they serve the platform. A business shares their door with their clients; clients reach the business without the platform being in the way. The platform, meanwhile, gains every direct door as a pointer back — a business sharing their URL 50 times a year is 50 new impressions for the organism.

### 19.2 — URL structure (Round 1 — path-based)

Because wildcard subdomains are not currently supported on Base44's infrastructure, Round 1 direct doors are path-based:

- `/b/{business-slug}` — a business's direct door
- `/u/{user-slug}` — a user's direct door (Round 2+)
- `/n/{network-slug}` — a network's direct door (Round 4+)
- `/e/{event-slug}` — an event's direct door (Round 2+)
- `/r/{region-slug}` — a region's direct door (Round 6+)

The prefix pattern (`/b/`, `/u/`, etc.) is explicit and extensible. Reserved top-level paths are the app's own (`/mylane`, `/directory`, `/events`, `/settings`, etc.). Slug generation must check against a reserved-word list and against other slugs within the same scope.

### 19.3 — URL structure (Round 2+ — custom domains)

Base44 supports custom domains per-business. A business may point their own domain at Base44; the app recognizes the domain and renders that business's surface. Example: Bari points `redumbrellaservices.com` at Base44; visits to that domain render Red Umbrella's business surface as if they had gone to `/b/red-umbrella`, but with the custom domain in the URL bar.

Setup is white-glove per-business in Round 2 (manual DNS on business's side, manual dashboard config on our side). Self-service flow is Round 4+.

### 19.4 — Public vs authenticated rendering

Same URL, different view based on auth state.

- **Unauthenticated visitor** to `/b/red-umbrella`: sees public profile — logo, tagline, services, gallery, contact form. No login prompt, no redirect. This is an open door.
- **Authenticated operator** of Red Umbrella at `/b/red-umbrella`: sees business tools — Desk, Finance, etc. This is the operator's workplace.
- **Authenticated non-operator** (another LocalLane user) at `/b/red-umbrella`: sees the public profile with additional member-context (contact directly via LocalLane message system when that ships, see full profile).

This requires Base44's App Visibility be set to Public at the platform level, with per-component auth-aware rendering inside.

### 19.5 — SEO & discoverability

Base44 currently supports client-side rendering only; no SSR. This affects how search engines index business profiles. Round 1 strategy:

- Use `react-helmet` or equivalent to manage per-route meta tags dynamically.
- Ensure every public profile has a title, description, Open Graph tags, and canonical URL.
- Base44's auto-generated sitemap at `/sitemap.xml` includes public routes.
- Accept that rankings will be slower to build than server-rendered sites.
- Track SSR availability on Base44's roadmap; upgrade when available.

Businesses currently operating their own SEO-ranked site (like Bari's WordPress site) should be advised transparently: replacing their current site with LocalLane in Round 1 may affect search rankings until SSR is solved. The full website replacement pitch is Round 2+ when we have a pre-render or SSR story.

### 19.6 — Business Profile space as eventual website

The Profile space inside a business is the long-term target for replacing standalone business websites. Round 1 Profile is minimum-viable (logo, tagline, services, gallery, contact). Round 2+ Profile grows to include blog, extended process descriptions, service detail pages, financing, testimonials — the full feature set of a traditional small-business website.

When a business reaches the point where their LocalLane Profile matches or exceeds their current site, they can switch: point their domain at LocalLane, retire their WordPress hosting, operate their public presence inside the organism.

---

## 20. Resurface Before Rebuild

LocalLane has accumulated useful surfaces during iteration that got buried as the architecture evolved. A persistent feedback affordance. An admin panel. Others yet to be identified.

When a previously-built surface is needed in the current architecture, the first action is to find the existing component in the codebase and wire it into the new cockpit structure. The action is not to rebuild from scratch.

This preserves accumulated design intent. It respects the work already done. And it catches buried capabilities before they become forgotten entirely.

Operationally: when Doron says "we used to have this," Hyphae's first step is to grep the codebase for the relevant component. If found, wire it into the current architecture. If not found, reconstruct from git history before rebuilding. Only rebuild if the original is genuinely lost.

This principle applies to entire surfaces (admin panel, feedback form) and to smaller affordances (keyboard shortcuts that used to work, empty states that used to have helpful copy). The resurface pass is part of Phase 4.5 work but the principle applies throughout.

---

## 21. Deferred

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

## 22. Known Issues — Separate Investigation (Completed)

Hyphae's directory investigation (April 22, 2026) resolved:

- Recess exists as a Business record with banner and `network_ids: ["recess"]` intact. Hidden from MyLane by `[0]` hardcode, but discoverable in Directory surface.
- Directory navigation bug: affordance gap on mobile. Overlay has no visible close button; users can't navigate away without force-quit. Fixable with a persistent close button on OverlayContainer.
- No Network entity exists in code — "networks" are currently a tag system (`network_ids` array on Business). Proper Network entity is Round 4 work.
- Business Settings has no directory visibility toggle today — `is_active` is conflated with delete.

**Pre-Round-1 patch (optional):** Ship the close-button affordance on OverlayContainer. Tiny PR, <1 hour, fixes active mobile pain, doesn't touch Round 1 work. Doron's call whether to patch now or bundle into Phase 3.

---

## 23. Open Questions — Things We'll Learn by Doing

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
