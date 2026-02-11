# Archetype: Microbusiness

**Created:** 2026-02-07
**Status:** Draft — Ready for review
**Owner:** Doron
**Decision Reference:** DEC-027

---

## Overview

### What Is This Archetype?

A **Microbusiness** is a solo operator or small family operation (typically 1-5 people) providing services, products, or experiences from their home, a mobile setup, or a client's location. They have no storefront. They are too small for enterprise tools and too real for just an Instagram page. LocalLane is their digital home.

This is the third archetype for LocalLane (after Events and Nonprofit/Church). It validates whether the platform can serve the largest and most underserved segment of local commerce: the people who make a community run but have no digital presence beyond word of mouth.

### Why This Archetype Matters to LocalLane

Microbusinesses are the connective tissue of local economies. Every dog walker, home baker, tutor, and handyman is both a provider AND a community member. On LocalLane, they exist on both sides of the platform — they use Community Pass as families while listing services as providers. This dual identity strengthens the network effect more than any other archetype.

The $9/month price point (see Pricing below) makes the platform accessible to a teenager mowing lawns, a retired handyman picking up weekend jobs, or a home baker selling at the Saturday market. Volume matters here — 200 microbusinesses at $9/month ($1,800/month) generates meaningful revenue while flooding the directory with exactly the kind of local, trusted services that make the platform indispensable.

### Core Philosophy

**These are not "gig workers."** Gig platforms extract — they take a cut, own the customer relationship, and commoditize the worker. LocalLane does the opposite: the microbusiness owns their listing, their reputation, and their customer relationships. LocalLane provides visibility, tools, and community connection. The money stays between neighbors.

---

## Sub-Types

Microbusinesses aren't one thing. The archetype must flex across fundamentally different operating models. Each sub-type has unique needs, but all share: no storefront, small scale, local focus, and a need for simple tools.

### Home Food & Cottage Industry

**Who:** Home bakers, jam makers, candy makers, meal preppers, specialty food producers

**Oregon Legal Context:**
- **Cottage Food Exemption** (most common path): No license or inspection required. Sales capped at $50,000/year (raised from $20,000 in 2024 via SB 643). Must be shelf-stable, non-refrigerated foods only. Food handler card required ($10 online). Labeling required including: producer name/phone, product name, ingredients, allergens, net weight, and the statement: *"This product is homemade, is not prepared in an inspected food establishment and must be stored and displayed separately if merchandised by a retailer."* Pet disclosure required if pets in home.
- **Domestic Kitchen License**: For those wanting to sell beyond cottage limits or sell perishable items. Requires ODA licensing and inspection.
- **Farm Direct**: For farmers selling products made from their own grown ingredients.
- Allowed cottage foods: breads, cakes, cookies, brownies, muffins, scones, tortillas, bagels, donuts, wedding cakes, candies, jams, jellies, dried fruits, granola, popcorn, roasted nuts, honey, freeze-dried foods, and more.
- Cannot sell to institutions (restaurants, schools, hospitals, nursing homes).
- Can sell: from home, online, at farmers markets, at events, and through retailers (with separate display requirements).

**LocalLane Value:** Listing with product catalog, order management, farmers market schedule, labeling compliance checklist, customer communication.

**Harvest Network connection:** Cottage food producers are natural Harvest Network members — their products show up in market discovery, meal planning integration, and CSA-style direct ordering.

### Home Services & Repair

**Who:** Handypeople, cleaners, pressure washers, painters, landscapers/lawn care, gutter cleaners, organizers, junk haulers

**Oregon Legal Context:**
- No "handyman license" exists in Oregon. Minor repairs under $1,000 (labor + materials) can be performed without a contractor's license.
- Jobs exceeding $1,000 or affecting structural integrity require a Residential Limited Contractor license from the Oregon Construction Contractors Board (CCB): 16 hours classroom training, exam (70% to pass), $15,000 surety bond, general liability insurance, $325 licensing fee, valid 24 months.
- Unlicensed handypeople cannot advertise as "contractors." Annual gross from construction work must stay under $40,000 to remain in handyman territory.
- Electrical and plumbing work requires separate trade licenses regardless of dollar amount.
- Eugene requires a city business license for all businesses operating within city limits. Home-based businesses may need a home occupation permit and zoning verification.

**LocalLane Value:** Service listing with specialties, service area map, availability calendar, job request matching, reputation via Nods/Stories/Vouches (more trustworthy than anonymous reviews), compliance awareness.

### Personal & Professional Services

**Who:** Tutors, music teachers, personal trainers, yoga instructors, life coaches, photographers, pet sitters, dog walkers, mobile groomers, personal chefs, babysitters/nannies, massage therapists, hair stylists (home-based)

**Oregon Legal Context:**
- Most personal services require only a Eugene business license.
- Some professions require state-level licensing: massage therapists (Oregon Board of Massage Therapists), cosmetologists (Oregon Health Licensing Agency), child care providers (Office of Child Care registration).
- Pet care services are generally unregulated at state level but may need city business license.
- Personal trainers and coaches have no state licensing requirement but industry certifications add credibility.

**LocalLane Value:** Portfolio/showcase, booking calendar, client communication, session packages, testimonials via recommendation system.

### Property Management & Short-Term Rentals

**Who:** Airbnb/VRBO hosts, long-term rental property managers, vacation rental operators, house sitters

**Oregon Legal Context:**
- Short-term rentals: Eugene requires a Short-Term Rental Permit. Transient Room Tax applies (currently 4.5% city + 1.8% TRT). Must comply with occupancy limits and safety requirements.
- Long-term rentals: Oregon landlord-tenant law (ORS Chapter 90) governs. Security deposit limits, required disclosures, notice requirements for entry and termination.
- Property managers handling others' properties may need a real estate license if managing for compensation (ORS 696).
- All rental income is reportable. 1099 requirements apply for service providers paid $600+.

**LocalLane Value:**
- *Short-term rental hosts:* Guest welcome guide tied to LocalLane discovery ("here's what's alive in Eugene this week"), property listing, cleaning/maintenance scheduling, financial tracking.
- *Long-term property managers:* Crabtree-model financial dashboards (10% maintenance reserve, 5% emergency fund, distribution tracking), tenant communication, maintenance request system, lease/document management, multi-property portfolio views.
- *Guest-to-community bridge:* Every short-term rental guest becomes a temporary LocalLane user. The host sends a LocalLane welcome link with their booking confirmation. The guest discovers local businesses, events, and experiences. Local businesses get foot traffic from visitors. This is the circulation model in action — the rental host is a gateway to the local economy.

### Creative & Maker Economy

**Who:** Artists, jewelers, woodworkers, potters, candle makers, soap makers, knitters/crocheters, digital creators, print-on-demand sellers

**LocalLane Value:** Portfolio/gallery, market/event schedule, commission request system, Creative Alliance network connection, workshop/class listing capability.

### Local Delivery & Community Services

**Who:** Personal shoppers, grocery/farmers market runners, local delivery drivers, errand services, pet taxi, senior companion services, yard sale organizers

**This is the "Local Uber/GrubHub" concept — but community-scale.** Not gig economy extraction. Neighbors serving neighbors, with money staying local.

**LocalLane Value:** Service area definition, availability windows, request matching, real-time job board, reputation system. Community Pass integration: families could use Joy Coins for local delivery services during off-peak windows.

---

## Data Model

### Core Fields (All Microbusiness Sub-Types)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Auto | Unique identifier |
| name | string | Yes | Business/service name |
| description | text | Yes | What you do, in your words |
| sub_type | enum | Yes | home_food, home_services, personal_services, property_management, creative_maker, local_delivery |
| owner_user_id | string | Yes | Link to LocalLane user account |
| service_area | object | No | Radius, neighborhoods, or "client location" |
| contact_phone | string | No | Business phone |
| contact_email | string | Yes | Business email |
| website | string | No | Website or social link |
| profile_image | string | No | Photo or logo |
| portfolio_images | array | No | Work samples, product photos |
| categories | array | Yes | Service/product categories |
| networks | array | No | LocalLane networks (Harvest, Recess, Creative Alliance, Gathering Circle) |
| availability | object | No | Schedule/hours/seasons |
| accepts_joy_coins | boolean | No | Participates in Community Pass |
| pricing_display | text | No | How they describe their pricing (freeform) |
| compliance_checklist | object | No | Self-reported licensing/certification status |
| created_at | datetime | Auto | Listing creation date |

### Sub-Type Specific Fields

**Home Food:**

| Field | Type | Description |
|-------|------|-------------|
| food_path | enum | cottage_food, domestic_kitchen, farm_direct |
| product_catalog | array | Products with names, descriptions, prices, allergens |
| food_handler_card | boolean | Has current food handler certification |
| market_schedule | array | Which farmers markets / events they attend |
| accepts_orders | boolean | Takes pre-orders |
| order_lead_time | string | How far in advance to order |

**Home Services:**

| Field | Type | Description |
|-------|------|-------------|
| license_status | enum | unlicensed_under_1k, ccb_residential, ccb_general |
| insurance_status | boolean | Has liability insurance |
| specialties | array | Specific services offered |
| service_area_radius | number | Miles from base |

**Property Management:**

| Field | Type | Description |
|-------|------|-------------|
| rental_type | enum | short_term, long_term, both |
| property_count | number | Number of managed properties |
| properties | array | Individual property records |
| financial_tracking | object | Income, expenses, reserves per Crabtree model |
| str_permit_number | string | Short-term rental permit (if applicable) |

**Personal Services:**

| Field | Type | Description |
|-------|------|-------------|
| certifications | array | Professional certifications |
| session_types | array | Service offerings with duration and price |
| booking_enabled | boolean | Accepts online booking |
| location_type | enum | client_home, my_home, mobile, virtual, varies |

---

## Pricing: The $9/Month Microbusiness Tier

### The Gap in the Current Tier System

| Current Tier | Price | Who It's For |
|-------------|-------|--------------|
| Free User | $0 | Community members browsing and discovering |
| Member (User Tier) | ~$5-10/mo (TBD) | Families wanting deeper access |
| **Microbusiness** | **$9/mo** | **Solo providers who are BOTH users and businesses** |
| Basic Business | Free | Businesses with a storefront wanting basic listing |
| Standard Business | $79/mo | Full-featured businesses in the Community Pass network |
| Partner Business | $149/mo | Established businesses with own node |

The $9/month tier fills a critical gap. A dog walker doesn't need a $79/month analytics dashboard. A home baker doesn't need staff management. But they need more than a free listing — they need tools that help them run their operation and connect with customers.

### What $9/Month Gets You

**Listing & Discovery:**
- Professional service/product listing in LocalLane directory
- Portfolio/gallery (up to 20 images)
- Service area visibility
- Network tagging (show up in Harvest, Recess, Creative Alliance, Gathering Circle feeds)
- Nod/Story/Vouch reputation (same trust system as all businesses)

**Operations:**
- Simple booking/scheduling calendar (personal services, property management)
- Order management (home food)
- Availability windows
- Client messaging
- Basic financial tracking (income, expenses, profit — Crabtree principles applied simply)

**Community:**
- Included as a Community Pass participant (accept Joy Coins if desired)
- Appear in MyLane discovery feeds
- Listed in The Good News newsletter rotation
- Access to microbusiness network events and workshops

**What $9/Month Does NOT Include (Standard tier features):**
- Staff management
- Advanced analytics dashboard
- Auto-publish events (events are reviewed)
- Revenue share pool analytics
- Priority listing

### Why $9 Works

- **Accessible:** A teenager mowing lawns can afford it. A retired handyman can afford it. A home baker testing the market can afford it.
- **Volume play:** 200 microbusinesses × $9/month = $1,800/month platform revenue. 500 = $4,500/month. These numbers add up alongside business tier subscriptions.
- **Upgrade path:** As a microbusiness grows, they naturally upgrade. The home baker who outgrows cottage food and opens a commercial kitchen becomes a Standard tier business. The handyman who gets a CCB license and hires helpers becomes Standard. The rental host who manages 5+ properties needs Standard or Partner tools.
- **Dual identity:** Microbusiness subscribers are also community members. Many will also hold a Community Pass for their family. $9 + $49 = $58/month total — they're deeply invested in the ecosystem.

---

## Features by Tier

### Free (Basic Listing Only)

| Feature | Description |
|---------|-------------|
| Directory listing | Name, description, contact, one photo |
| Category tagging | Appear in relevant categories |
| Basic Nod/Story/Vouch | Receive trust signals |

### Microbusiness ($9/month)

| Feature | Description |
|---------|-------------|
| Everything in Free | |
| Full portfolio | Up to 20 photos/work samples |
| Product/service catalog | Detailed listings with pricing |
| Booking calendar | Simple scheduling for appointments/jobs |
| Order management | Pre-orders for food producers |
| Client messaging | In-platform communication |
| Basic financials | Income/expense tracking, profit visibility |
| Network visibility | Appear in network-specific feeds |
| Community Pass participation | Accept Joy Coins (optional) |
| Compliance tracker | Self-service licensing/certification checklist |
| The Good News inclusion | Eligible for newsletter features |

### Upgrade to Standard ($79/month) — When You Outgrow Microbusiness

Triggers for upgrade:
- Hiring employees or regular contractors
- Opening a physical location
- Exceeding cottage food / handyman thresholds
- Wanting advanced analytics, auto-publish, staff tools
- Wanting deeper Community Pass integration (revenue share analytics, capacity management)

---

## Onboarding Flow

```
Step 1: ARCHETYPE
  "How do you serve the community?"
  → User selects "Microbusiness / Solo Provider"

Step 2: SUB-TYPE
  "What kind of work do you do?"
  → Home Food & Baking
  → Home Services & Repair
  → Personal & Professional Services
  → Property Management & Rentals
  → Creative & Maker
  → Local Delivery & Community Services

Step 3: DETAILS (varies by sub-type)
  Home Food → Food path (cottage/domestic/farm), products, markets
  Home Services → Specialties, service area, license status
  Personal Services → Service types, certifications, location type
  Property Management → Rental type, property count
  Creative → Medium, markets/events, commission availability
  Local Delivery → Service types, availability, service area

Step 4: COMPLIANCE AWARENESS
  Based on sub-type, show relevant Oregon requirements:
  → "As a cottage food producer in Oregon, here's what you need to know..."
  → "As a handyman in Oregon, jobs under $1,000 don't require a license, but..."
  → NOT legal advice — awareness and links to official sources

Step 5: LISTING PREVIEW
  "Here's how you'll appear in the LocalLane directory."
  → Preview with edit capability

Step 6: PLAN SELECTION
  "Ready to go? Choose your plan."
  → Free listing (basic)
  → Microbusiness tier ($9/month) — recommended, with feature comparison
```

---

## Integration Points

### Community Pass & Joy Coins

Microbusinesses can opt into the Community Pass network. When they do:
- They appear in Joy Coin-eligible discovery feeds
- Families can use Joy Coins for their services (e.g., 2 Joy Coins for a dog walking session, 3 for a baking class)
- The microbusiness receives revenue share from the Community Pass pool based on check-ins
- This works identically to how Standard tier businesses participate — same pool, same scan-based allocation

### MyLane Discovery

Microbusiness listings appear in MyLane based on:
- User interests and network membership
- Geographic proximity (service area overlap)
- Trust signals (Nods/Stories/Vouches from neighbors)
- Season and availability (a lawn care service is more visible in spring)

### The Good News Newsletter

Microbusinesses rotate through newsletter features:
- "Meet Your Neighbor" profiles
- Seasonal spotlights (holiday bakers, spring landscapers)
- New listing announcements
- Success stories (microbusiness that grew into Standard tier)

### Harvest Network (Home Food Specific)

- Cottage food producers appear in Harvest Network feeds
- Market schedule integration: "Sarah's Sourdough is at the Saturday Market this week"
- Pre-order capability: "Order by Thursday for Saturday pickup"
- Future: Meal planner integration — browse what's available from local producers this week, plan meals around local ingredients

### Guest Welcome (Property Management Specific)

- Short-term rental hosts can send a LocalLane welcome link with booking confirmations
- Guests get a curated view: "What's alive in Eugene this week"
- Local businesses get foot traffic from visitors who would otherwise default to chains
- The host's listing includes a "Guest Experience" section showing what their visitors can discover

### Job Request Board (Local Delivery & Services)

- Community members post needs: "I need help moving a couch Saturday" or "Can someone pick up my CSA box?"
- Matching microbusiness providers see relevant requests
- Acceptance, scheduling, and completion tracking
- Trust signals inform who responds to what

---

## Oregon Compliance Reference

This section is NOT legal advice. It provides awareness of common requirements to help microbusiness operators know what to research. LocalLane should link to official sources and encourage operators to verify requirements for their specific situation.

### Universal Requirements (All Microbusinesses in Eugene)

- **City of Eugene Business License:** Required for all businesses operating within city limits. Fee based on gross receipts. Businesses under $50,000 pay $50. Apply through the City of Eugene.
- **Oregon Secretary of State Registration:** Required for LLCs and corporations ($100 filing fee). Sole proprietors don't need to register unless using a DBA ($50).
- **Home Occupation Permit:** Eugene may require zoning verification for home-based businesses. Check with City of Eugene Planning and Development.
- **Tax Registration:** Oregon has no sales tax, but income tax applies. Business income reported on personal return for sole proprietors.

### Sub-Type Specific Requirements

| Sub-Type | Key Requirements | Official Source |
|----------|-----------------|-----------------|
| Cottage Food | Food handler card, labeling, $50K cap | Oregon Dept. of Agriculture |
| Domestic Kitchen | ODA license, inspection, expanded products | Oregon Dept. of Agriculture |
| Handyman (<$1K jobs) | Business license recommended, no CCB license | Oregon CCB |
| Contractor (>$1K jobs) | CCB Residential Limited Contractor license, bond, insurance | Oregon CCB |
| Massage Therapist | State license required | OR Board of Massage Therapists |
| Child Care | Office of Child Care registration | Oregon OCC |
| Cosmetology | State license required | OR Health Licensing Agency |
| Short-Term Rental | Eugene STR permit, transient room tax | City of Eugene |
| Long-Term Rental | Landlord-tenant law compliance (ORS Ch. 90) | Oregon Legislature |
| Pet Care | Business license (no state license) | City of Eugene |

---

## User Workflows

### Workflow 1: Home Baker Lists Products (Onboarding)

**Actor:** Cottage food producer

**Steps:**
1. Signs up for LocalLane as a user
2. Selects "I also provide services/products" during onboarding (or from MyLane settings)
3. Chooses Microbusiness → Home Food & Baking
4. Selects food path: Cottage Food Exemption
5. Adds products with names, descriptions, prices, allergens
6. Sets market schedule (Saturday Market, Holiday Market, etc.)
7. Sees compliance awareness: food handler card, labeling requirements, $50K cap
8. Previews listing
9. Selects Microbusiness tier ($9/month) or Free listing
10. Listing goes live in directory and Harvest Network feed

**Success criteria:** Product listing visible, compliance items acknowledged, orders can be placed (if $9 tier)

### Workflow 2: Neighbor Finds a Handyman

**Actor:** Community member needing home repair

**Steps:**
1. Opens MyLane or browses directory
2. Searches "handyman" or browses Home Services category
3. Sees local handyman listings with trust signals (3 Nods from neighbors, 1 Vouch)
4. Views profile: specialties, service area, availability, compliance status
5. Sends message or books through calendar
6. After service, gives a Nod ("Jake fixed our fence gate — quick and fair")

**Success criteria:** Customer found trusted provider through neighbor recommendations, not anonymous reviews

### Workflow 3: Airbnb Host Connects Guests to Community

**Actor:** Short-term rental host

**Steps:**
1. Host has Microbusiness listing for their rental property
2. Configures "Guest Welcome" with curated local recommendations
3. Sends LocalLane welcome link with booking confirmation
4. Guest arrives, opens link, sees: "Welcome to Eugene — here's what's alive this week"
5. Guest discovers Saturday Market, a climbing gym, a pottery studio
6. Guest uses Community Pass day passes (future feature) or visits directly
7. Local businesses get a visitor who was otherwise invisible to them

**Success criteria:** Guest engages with local businesses, host provides differentiated guest experience

### Workflow 4: Property Manager Tracks Finances

**Actor:** Family property manager (2 rental properties)

**Steps:**
1. Lists as Microbusiness → Property Management → Long-Term
2. Adds two properties with addresses, rent amounts, tenants
3. Monthly: records income and expenses per property
4. Dashboard shows per-property breakdown (Crabtree model):
   - Gross rent: $1,500
   - Operating expenses: $500
   - Maintenance reserve (10%): $150
   - Emergency reserve (5%): $75
   - Net distribution: $775
5. Year-end: export for tax preparation

**Success criteria:** Clear financial picture per property, reserves tracked, distribution calculated

---

## Comparison: Microbusiness vs Other Archetypes

| Aspect | Event/Venue | Nonprofit | Microbusiness |
|--------|-------------|-----------|---------------|
| Physical location | Yes (venue) | Often yes | No (home/mobile/client) |
| Employees | Often yes | Volunteers | Usually solo |
| Revenue model | Tickets, passes, direct | Donations, dues | Direct service/product |
| Key features | Event creation, capacity | Announcements, directory | Portfolio, booking, orders |
| Tier sweet spot | Standard ($79) | Basic/Standard | Microbusiness ($9) |
| Community Pass | Primary participant | Optional | Optional participant |
| Trust system | Nods for events | Nods for org | Nods for person/work |

---

## Test Cases

### Test Case 1: Home Baker (Cottage Food)

- Produces sourdough bread, cookies, and seasonal pies
- Sells at Eugene Saturday Market and from home
- Takes pre-orders via text/Instagram currently
- Wants: product catalog, order management, market schedule, customer communication
- Compliance: food handler card, cottage labeling

### Test Case 2: Handyman

- Retired, does weekend repair work for neighbors
- Stays under $1,000/job, no CCB license
- Gets work through word of mouth
- Wants: listing with specialties, availability, reputation from neighbors
- Compliance: city business license, $1K job limit awareness

### Test Case 3: Doron's Family Rental Properties

- 2 long-term rental properties, $3,000/month combined income
- Family-managed, no property management company
- Wants: financial tracking (Crabtree model), maintenance logging, tenant communication
- Compliance: ORS Chapter 90 awareness, tax reporting

### Test Case 4: Airbnb Host

- 1 vacation rental near UO campus
- High turnover, seasonal demand
- Wants: guest welcome guide with local discovery, cleaning schedule, financial tracking
- Compliance: Eugene STR permit, transient room tax

### Test Case 5: Dog Walker / Pet Sitter

- College student walking dogs between classes
- 5-8 regular clients in South Eugene
- Wants: availability calendar, client messaging, route planning
- Compliance: city business license

---

## Open Questions

| Question | Status | Notes |
|----------|--------|-------|
| Should $9/month include Community Pass personal membership? | Open | Would make the dual identity seamless — you're both provider and member for $9. But might undercut the $49 Community Pass. |
| Can microbusinesses upgrade to Standard mid-month? | Open | Probably yes, prorated. Stripe handles this. |
| How do we handle microbusinesses that operate in multiple sub-types? | Open | A person who bakes AND does handyman work. One listing or two? |
| Should there be a "Founding Microbusiness" incentive? | Open | First 3 months free, like business tier founding deal. Low risk, builds volume. |
| Job request board — separate feature or baked into microbusiness? | Open | Could be a standalone feature any community member uses, with microbusiness providers as responders. |
| Meal planner integration timeline? | Open | Depends on Harvest Network launch. Document concept, build later. |
| Guest day pass for short-term rental visitors? | Open | Would need a lightweight Community Pass variant. Future feature. |

---

## Implementation Approach

### Phase 1: Core Listing (Microbusiness MVP)

- [ ] Sub-type selection in onboarding wizard
- [ ] Basic listing fields per sub-type
- [ ] Portfolio/gallery upload
- [ ] Directory visibility with category and network filtering
- [ ] Compliance awareness content (static, links to official sources)
- [ ] Free tier only (no payment yet)

### Phase 2: $9/Month Tier + Tools

- [ ] Stripe subscription product for Microbusiness tier
- [ ] Booking calendar (personal services)
- [ ] Order management (home food)
- [ ] Basic financial tracking (property management)
- [ ] Client messaging
- [ ] Community Pass opt-in for microbusinesses

### Phase 3: Network Integration

- [ ] Harvest Network feed for food producers
- [ ] Guest Welcome system for rental hosts
- [ ] Job request board for service providers
- [ ] The Good News newsletter integration
- [ ] MyLane discovery algorithm inclusion

### Phase 4: Growth Features

- [ ] Meal planner integration (Harvest)
- [ ] Multi-property portfolio dashboard
- [ ] Guest day passes (short-term rental bridge)
- [ ] Upgrade prompts when microbusiness outgrows tier

---

## Relationship to Other Documents

| Document | Connection |
|----------|------------|
| DECISIONS.md DEC-027 | Decision to prioritize Microbusiness archetype |
| ENTITY-SYSTEM.md | Microbusiness is a new archetype in the entity system |
| ONBOARDING-SYSTEM.md | Onboarding flow needs new sub-type paths |
| TIER-SYSTEM.md | $9/month tier sits between User and Business tiers |
| COMMUNITY-PASS.md | Microbusinesses can participate in Joy Coins network |
| STRIPE-CONNECT.md | $9/month subscription product needed |
| USER-TIERS.md | Dual identity: microbusiness operators are also users |
| MARKETING-DECK.md | Needs microbusiness pitch section |
| ORGANISM-CONCEPT.md | Microbusiness activity feeds the community organism |
| nonprofit-node.md | Sister archetype spec for reference |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-07 | Initial draft with Oregon research | Claude (Mycelia) |

---

*This archetype spec is drafted and ready for review. Implementation follows Events archetype stabilization.*
