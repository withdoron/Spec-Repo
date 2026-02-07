# Archetype: Nonprofit / Church / Community Organization

**Created:** 2026-01-27
**Updated:** 2026-02-07
**Status:** Draft (Next to Build after Events stabilization)
**Owner:** Doron

---

## Overview

### What Is This Archetype?

A **Nonprofit/Community Organization** is focused on member engagement, communication, coordination, and community service rather than commercial transactions. This archetype serves the full spectrum of nonprofit entities — from churches and food pantries to youth programs, environmental groups, and mutual aid networks.

Eugene-Springfield metro has 2,443 registered nonprofits employing 17,711 people with $2 billion in annual revenue. This archetype is designed to serve that density.

### Sub-Types

| Sub-Type | Examples | Primary Needs |
|----------|----------|---------------|
| **Houses of Worship** | Churches, temples, mosques | Announcements, member directory, event calendar, volunteer coordination |
| **Food Security & Mutual Aid** | Food pantries, meal programs, food rescue, gleaning | Volunteer scheduling, distribution mapping, donor engagement, cross-referral |
| **Youth Programs** | After-school programs, mentoring, youth empowerment | Program enrollment, parent communication, attendance, session scheduling |
| **Environmental** | Recyclers, land trusts, watershed groups | Donation scheduling, volunteer events, education programs |
| **Community Infrastructure** | Tool libraries, tiny home villages, warming centers | Inventory/resource management, volunteer shifts, location info |
| **Social Services** | Clinics, housing providers, crisis response | Service listing, referral networks, hours/location, volunteer coordination |
| **Arts & Culture** | Mural projects, cultural orgs, heritage groups | Event promotion, gallery/portfolio, program listings, community engagement |
| **Advocacy & Civil Rights** | Civil liberties, community organizing, identity support | Event coordination, resource directories, community communication |

### User Personas

**Organization Admin (Leader):**
- Pastor, executive director, volunteer coordinator, board member
- Wants to communicate with members and community without relying on texts/paper/Facebook
- Needs to organize events, volunteers, announcements, programs
- Currently using fragmented tools
- Tech comfort varies widely — interface must be simple

**Member/Volunteer:**
- Church member, volunteer, participant, donor
- Wants to know what's happening and how to help
- Needs to sign up for events and volunteer opportunities
- May have family members who also participate

**Community Beneficiary:**
- Person served by the organization (food pantry client, youth program participant, etc.)
- May interact with organization through LocalLane discovery without formal membership
- Privacy is paramount — no tracking of service usage without explicit consent

---

## Nonprofit Pricing

Nonprofits receive a 50% discount on standard business tier pricing. Qualification requires 501(c)(3) EIN or Oregon nonprofit registration — self-reported during onboarding, spot-checked if needed.

| Tier | Code | Monthly | Business Equivalent | What They Get |
|------|------|---------|---------------------|---------------|
| **Basic** | `basic` | Free | Free (same) | Listing, contact info, public events (reviewed), directory presence |
| **Nonprofit Standard** | `standard` | **$39/mo** | $79/mo business | Auto-publish events, volunteer coordination, announcements, member directory, email notifications, Community Pass participation, Joy Coin acceptance, newsletter features, analytics |
| **Nonprofit Partner** | `partner` | **$79/mo** | $149/mo business (earned) | Own Partner Node, custom branding, member groups/committees, full member management, attendance tracking, multi-program dashboard, priority listing, trust badge |

**Founding nonprofit deal:** First 3-6 months free for the first 10 nonprofits to join. Proves platform value before any charge.

**Why 50% (not free across the board):**
- Signals LocalLane values their work without treating them as charity — they're partners
- $39/month is realistic for orgs with operational budgets and grants
- Keeps pricing architecture clean: businesses $79, nonprofits $39, microbusinesses $9
- Free tier already covers orgs that just need visibility
- Sustainability: 50 nonprofits × $39 = $1,950/month

**Partner tier remains earned** — same criteria as business Partner (6+ months at Standard, engagement threshold, vouches, zero violations). The discount applies to the monthly fee, not the qualification bar.

---

## Data Model

### Core Fields (All Tiers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Auto | Unique identifier |
| name | string | Yes | Organization name |
| description | text | Yes | About this organization |
| sub_type | enum | Yes | house_of_worship, food_security, youth_program, environmental, community_infrastructure, social_services, arts_culture, advocacy |
| ein | string | No | 501(c)(3) EIN (for nonprofit discount verification) |
| address | string | No | Physical location |
| phone | string | No | Contact phone |
| email | string | Yes | Contact email |
| website | string | No | Website URL |
| logoUrl | string | No | Organization logo |
| networks | array | No | Networks (Recess, Creative Alliance, Harvest, Gathering Circle) |
| categories | array | No | Categories (religious, volunteer, food, youth, etc.) |
| service_area | string | No | Geographic area served |
| hours | object | No | Operating hours |

### Standard Tier Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| memberCount | number | Tracked member count |
| memberDirectory | boolean | Is directory enabled? |
| volunteerOpportunities | array | Active volunteer needs |
| announcementHistory | array | Past announcements |
| programs | array | Named programs/services offered |
| accepts_joy_coins | boolean | Participates in Community Pass |

### Partner Tier Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| customBranding | object | Logo, colors, custom styling |
| memberGroups | array | Small groups, committees, teams |
| donationLink | string | External donation page |
| customPages | array | Additional content pages |
| multi_program_dashboard | boolean | Manage multiple programs from one view |

### Sub-Type Specific Fields

**Food Security & Mutual Aid:**

| Field | Type | Description |
|-------|------|-------------|
| distribution_locations | array | Pantry/distribution point locations with addresses |
| distribution_schedule | object | When food is available (days, hours, appointment info) |
| food_rescue_partners | array | Partner businesses for food rescue |
| delivery_radius | string | Delivery area description |
| volunteer_roles | array | Typed roles (driver, stocker, cook, sorter, etc.) |

**Youth Programs:**

| Field | Type | Description |
|-------|------|-------------|
| age_range | object | min_age, max_age |
| program_sessions | array | Named sessions with schedules |
| enrollment_open | boolean | Currently accepting participants |
| parent_communication | boolean | Parent notification features enabled |
| certifications | array | Relevant certifications held |

**Community Infrastructure:**

| Field | Type | Description |
|-------|------|-------------|
| resource_inventory | array | Available resources (tools, spaces, etc.) |
| checkout_enabled | boolean | Resource lending/checkout system |
| capacity | number | People/units served |
| locations | array | Multiple location addresses |

---

## Features by Tier

### Basic (Free)

| Feature | Description |
|---------|-------------|
| Basic listing | Appears in Community Node directory |
| Contact info | Phone, email, website, location |
| Public events | Post events (reviewed before publish) |
| Basic description | About page visible to public |
| Sub-type tagging | Categorized by organizational type |

### Nonprofit Standard ($39/month)

| Feature | Description |
|---------|-------------|
| Everything in Basic | |
| Auto-publish events | No review queue |
| Announcements | Post updates visible to members |
| Member directory | Opt-in list of members |
| Volunteer coordination | Post needs, track sign-ups, role-based |
| Program listings | Named programs with details and schedules |
| Email notifications | Notify members of updates |
| Community Pass participation | Accept Joy Coins, appear in network feeds |
| Newsletter features | Included in The Good News rotation |
| Basic analytics | Member engagement, event attendance |

### Nonprofit Partner ($79/month, earned)

| Feature | Description |
|---------|-------------|
| Everything in Standard | |
| Own application | Dedicated Nonprofit Node |
| Custom branding | Own logo, colors |
| Member groups | Small groups, committees, teams |
| Full member management | Add/remove, roles, permissions |
| Attendance tracking | Track participation across programs |
| Multi-program dashboard | Manage multiple programs from one view |
| Priority in Community Node | Higher in directory |
| Trust badge | "Verified Community Partner" |

---

## User Workflows

### Workflow 1: Post Announcement (Admin)

**Actor:** Organization Admin (Standard+)

**Steps:**
1. Log into Nonprofit Node (or Community Node if Standard)
2. Go to "Announcements"
3. Click "New Announcement"
4. Enter title and content
5. Choose visibility (all members, specific groups)
6. Choose notification method (in-app only, email too)
7. Click "Publish"
8. Members see announcement in their feed / receive email

**Success criteria:** Members can view announcement, engagement tracked

### Workflow 2: Volunteer Sign-Up (Member)

**Actor:** Member

**Steps:**
1. View organization page
2. See "Volunteer Opportunities" section
3. Browse open opportunities (filtered by role type if available)
4. Click "Sign Up" on one
5. See confirmation and details (when, where, what to bring)
6. Opportunity appears in "My Volunteer Commitments"
7. Admin sees updated sign-up list

**Success criteria:** Member confirmed, admin sees volunteer count +1

### Workflow 3: View Member Directory (Member)

**Actor:** Member (at organization with directory enabled)

**Steps:**
1. Log into organization's page
2. Go to "Member Directory"
3. Search or browse members (only those who opted in)
4. View member's profile (name, contact info they chose to share)

**Privacy considerations:**
- Members must opt-in to directory
- Members control what info is visible
- Must be logged in and a member to view

### Workflow 4: Create Event (Admin)

**Actor:** Organization Admin

**Steps:**
1. Log into Nonprofit Node
2. Click "Create Event"
3. Fill in event details (leverages Event archetype)
4. Choose audience (public, members only, specific groups)
5. Publish
6. Event appears in organization calendar AND Community Node (if public)

**Success criteria:** Event visible to appropriate audience

### Workflow 5: Manage Volunteer Roles (Admin — Food Security Sub-Type)

**Actor:** Organization Admin (e.g., Burrito Brigade)

**Steps:**
1. Go to "Volunteer Management"
2. Create role types (Star Supporter, Pantry Host, Weekend Cook, Delivery Driver)
3. Set schedule for each role (recurring or one-time)
4. Post openings with details (time commitment, requirements)
5. Volunteers sign up for specific roles
6. Admin sees roster by role, date, and location

**Success criteria:** Volunteers matched to roles, schedule coverage visible

---

## Integration Points

### Data Pushed to Community Node

| Data | Trigger | Endpoint |
|------|---------|----------|
| Organization profile | Admin updates | POST /api/v1/businesses |
| Public events | Event published | POST /api/v1/events |
| Volunteer opportunities | Opportunity posted | POST /api/v1/volunteer |
| Program listings | Program created/updated | POST /api/v1/programs |

### Data Pulled from Community Node

| Data | Frequency | Endpoint |
|------|-----------|----------|
| Config (networks, categories) | On startup + every 5 min | GET /api/v1/config |
| User accounts | On login | Auth system |

### Community Pass Integration

- Standard+ nonprofits can accept Joy Coins for paid programming (classes, workshops, events)
- Volunteer hours could earn Joy Coin equivalents (future consideration)
- Food security orgs typically don't use Joy Coins for food distribution (that stays free)
- Youth programs may use Joy Coins for enrichment classes

### Cross-Referral Network

Nonprofits already link to each other informally. LocalLane makes these connections visible:
- Organizations can list "Related Organizations" on their profile
- Shared events between organizations show in both calendars
- Resource referral cards: "Need help? These organizations can assist"

---

## Organism Connection

Nonprofits are among the most vital nodes in the community organism. Their activity patterns — volunteer mobilization, food distribution, youth engagement — are direct indicators of community health.

**What a healthy nonprofit organism looks like:**
- Active volunteer sign-ups and fulfilled roles
- Regular events with community attendance
- Cross-referral connections to other organizations
- Member engagement with announcements and programs
- Geographic coverage (e.g., pantry network spanning neighborhoods)

**What an unhealthy one looks like:**
- Unfilled volunteer shifts
- Events with low attendance
- No cross-referral connections
- Stale announcements

The Organism concept should eventually reflect this — a food security network like Burrito Brigade's 55 Little Free Pantries would light up across the map as pantries are stocked and restocked, showing the real-time pulse of neighborhood food access.

---

## UI/UX Considerations

### Key Screens

1. **Organization Home** — Overview, recent announcements, upcoming events
2. **Announcements Feed** — All announcements, filterable
3. **Member Directory** — Searchable member list (opt-in)
4. **Volunteer Board** — Current opportunities, role-based filtering
5. **Events Calendar** — Organization-specific events
6. **Programs** — Named programs with details, schedules, enrollment
7. **Admin Dashboard** — Member management, analytics, volunteer tracking

### Mobile Considerations

- Primary use case is mobile (checking announcements on phone, signing up to volunteer)
- Push notifications for important announcements
- Quick volunteer sign-up flow
- Location-aware for multi-site organizations

### Accessibility Considerations

- Older demographic may have accessibility needs
- Large text option
- High contrast mode
- Simple navigation
- Screen reader compatible

---

## Technical Notes

### Special Requirements

- Email notification system (for announcements)
- Privacy controls (directory opt-in, service beneficiary protection)
- Group/committee management (Partner tier)
- Role-based volunteer management
- Multi-location support (for organizations with distributed sites)

### Known Challenges

- Privacy: Members must control their own visibility
- Permissions: Different roles (admin, leader, member, volunteer)
- Notifications: Don't spam, respect preferences
- Service beneficiaries: Never track or expose who receives services

### Dependencies

- Community Node for config sync
- Event system (shared with Event archetype)
- User authentication
- Stripe for nonprofit tier billing (with nonprofit discount applied)

---

## Test Cases

### Test Case 1: Doron's Church (House of Worship)

**Current State:** Communication via paper, texts, conversations. No central system.

**Target State:**
- Announcements visible to all members
- Event calendar everyone can access
- Volunteer opportunities posted, sign-ups tracked
- Optional member directory

**Success Metrics:**
- [ ] Admin can post announcement in < 2 minutes
- [ ] Members can see announcements on their phone
- [ ] At least one volunteer opportunity posted and filled
- [ ] 50%+ of active members using the system

### Test Case 2: Burrito Brigade (Food Security)

**Current State:** Three programs (Weekend Burrito Brigade, Little Free Pantries, Waste to Taste), 55+ pantry locations, 300K+ meals served, volunteer-powered, average donation $5-10.

**Target State:**
- Volunteer coordination across three programs with role-based sign-ups
- Little Free Pantry locations visible on map
- Community awareness — neighbors discover nearby pantries
- Event scheduling for weekend production shifts
- Cross-referral connections to FOOD for Lane County, White Bird, Black Thistle

**Success Metrics:**
- [ ] Volunteers can sign up for specific roles (Star Supporter, Pantry Host, Weekend Cook)
- [ ] All 55+ pantry locations discoverable by neighborhood
- [ ] Weekend production shifts fill via platform sign-ups
- [ ] At least 3 cross-referral connections active

### Test Case 3: Ophelia's Place (Youth Program)

**Current State:** Multiple programs across Lane & Linn counties — after-school, therapy, school partnerships, training. Serves girl-identified youth ages 10-18.

**Target State:**
- Program listings with enrollment info
- Event calendar for community events and fundraisers
- Parent communication channel
- Cross-referral to SASS, Trans*Ponder, Youth Era

**Success Metrics:**
- [ ] Programs listed with clear descriptions and schedules
- [ ] Community events promoted and RSVPs tracked
- [ ] Cross-referral connections visible

---

## Implementation Phases

### Phase 1: Core Listing (Free tier)
- Sub-type selection during onboarding
- Basic fields, description, contact info
- Directory visibility in Community Node
- Public event posting (reviewed)

### Phase 2: Standard Tier ($39/month)
- Stripe subscription with nonprofit discount
- Announcements system
- Member directory (opt-in)
- Volunteer coordination (basic)
- Program listings
- Community Pass opt-in

### Phase 3: Network Integration
- Cross-referral connections between organizations
- Newsletter features
- Joy Coin acceptance for paid programming
- Multi-location map support
- Role-based volunteer management

### Phase 4: Partner Features ($79/month, earned)
- Own Nonprofit Partner Node
- Custom branding
- Member groups and committees
- Multi-program dashboard
- Advanced analytics

---

## Open Questions

| Question | Status | Answer |
|----------|--------|--------|
| Should members pay dues through the platform? | Deferred | Out of scope for Phase 1-2 |
| How to handle family accounts (parent + kids)? | Open | Household model from Community Pass may apply |
| What member info is appropriate to share? | Open | Privacy policy needed, minimum viable: name + opt-in contact |
| Volunteer hour tracking for Earn-Your-Pass? | Open | Could connect to farm/market program |
| Real-time pantry stocking status? | Future | Requires simple check-in mechanism for hosts |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-01-27 | Initial draft — church-focused | Claude |
| 2026-02-07 | Major update — sub-types, nonprofit pricing (50% discount), expanded data model, test cases (Burrito Brigade, Ophelia's Place), implementation phases, Organism connection | Claude + Doron |

---

*This archetype spec is drafted and ready for implementation after Event Node is stable.*
