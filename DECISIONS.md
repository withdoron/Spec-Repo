# Decision Log

> Records key architectural and implementation decisions with context.
> Last Updated: 2026-02-01

---

## How to Use This Document

Each decision includes:
- **Date** — When the decision was made
- **Context** — Why we needed to decide
- **Decision** — What we chose
- **Rationale** — Why we chose it
- **Status** — Current state (Active, Superseded, Revisit)

---

## Implementation Decisions (DEC-001 …)

### DEC-001: Three-Tier System Naming

**Date:** 2026-01-28

**Context:** The tier system had inconsistent naming between spec documents and code.

**Decision:** Use code values `basic`, `standard`, `partner` consistently.

**Mapping:**
| Tier Level | Code Value | Display Name |
|------------|------------|--------------|
| 1 | `basic` | Basic |
| 2 | `standard` | Standard |
| 3 | `partner` | Partner |

**Rationale:** Code already used these values. Changing spec to match code is less disruptive than changing code + database.

**Status:** ✅ Active

---

### DEC-002: Tier 1 & 2 in Community Node, Tier 3 as Partner Node

**Date:** 2026-01-28

**Context:** Needed to clarify where each tier's functionality lives.

**Decision:**
- Basic & Standard users manage events within Community Node
- Partner users get their own standalone Partner Node
- Partner Node events sync UP to Community Node

**Rationale:** Simpler for majority of users (Tier 1 & 2). Premium experience for Partners. Consistent data in Community Node for search/discovery.

**Status:** ✅ Active

---

### DEC-003: Event Editor Copied from Event Node

**Date:** 2026-01-29

**Context:** Community Node's Event Editor was incomplete. Event Node had a full-featured editor.

**Decision:** Copy Event Node's EventFormV2 to Community Node, adapt for local data model and hooks.

**Approach:**
- Keep as inline editor (not modal) for MVP
- Use useOrganization() hook for tier checking
- Map field names to Community Node's Event schema
- Add LockedFeature component for tier gating

**Rationale:** Faster than building from scratch. Event Node's editor was already polished.

**Status:** ✅ Complete — Full feature parity achieved (2026-01-31). See DEC-019.

---

### DEC-004: Hard Delete vs Soft Delete

**Date:** 2026-01-29

**Context:** Soft delete (`is_deleted: true`) wasn't working — field may not exist in Base44 schema.

**Decision:** Use hard delete for now. Plan to implement soft delete later.

**Rationale:** Needed working delete functionality for pilot. Hard delete is acceptable for test data. Production should use soft delete for data recovery.

**Status:** ⚠️ Active (temporary) — Revisit when Base44 schema can be modified.

---

### DEC-005: AdminSettings Fallback for Config Storage

**Date:** 2026-01-29

**Context:** Needed to store platform configuration (Event Types, Networks, etc.). Creating a new PlatformConfig entity requires Base44 dashboard access.

**Decision:** Use existing AdminSettings entity with key convention:
```
platform_config:{domain}:{config_type}
```

Value is JSON string: `{ items: [...], updated_at: Date }`

**Rationale:** AdminSettings already exists and works. Can migrate to dedicated PlatformConfig entity later without changing the useConfig hook interface.

**Status:** ✅ Active — Works well. Migration to dedicated entity is optional.

---

### DEC-006: Admin Panel Sidebar Navigation

**Date:** 2026-01-29

**Context:** Admin Panel had horizontal tabs. As features grow, tabs would become unwieldy.

**Decision:** Restructure Admin Panel with sidebar navigation organized by domain:
- MANAGEMENT (Businesses, Users, Locations, Partners)
- PLATFORM (Networks, Tiers, Punch Pass, Settings)
- EVENTS (Event Types, Age Groups, Durations)
- ONBOARDING (Business, User)

**Rationale:** Scales better as archetypes and features are added. More intuitive navigation. Supports deep linking via React Router.

**Status:** ✅ Active

---

### DEC-007: Config Inheritance Model

**Date:** 2026-01-29

**Context:** Partner Nodes need access to Event Types, Networks, etc. Should each Partner define their own, or inherit from Community Node?

**Decision:** Community Node owns all config. Partner Nodes inherit (read-only for now).

**Key Points:**
- Ensures congruent data for search/filter across all nodes
- User searches "Live Music" and finds ALL events, regardless of source
- Future: Can allow Partner additions tagged as "partner-specific"

**Rationale:** Data consistency is more important than Partner customization for MVP. Structure supports future flexibility.

**Status:** ✅ Active

---

### DEC-008: Event Editor Uses Dynamic Config

**Date:** 2026-01-30

**Context:** Event Types, Networks, Age Groups were hardcoded in EventEditor.

**Decision:** Replace hardcoded arrays with useConfig() hook calls. EventEditor fetches config from AdminSettings.

**Implementation:**
```javascript
const { data: eventTypes = [] } = useConfig('events', 'event_types');
const { data: networks = [] } = useConfig('platform', 'networks');
// etc.
```

**Rationale:** Admin can now add/edit/delete options without code changes. Single source of truth in Admin Panel.

**Status:** ✅ Active

---

### DEC-009: Purple Accent for Dev Mode

**Date:** 2026-01-28

**Context:** Dev Mode tier switcher in Event Node uses purple accent.

**Decision:** Keep purple for Dev Mode. Do NOT change to amber.

**Rationale:** Purple clearly signals "you're in dev/test mode" — visually distinct from production amber accent.

**Status:** ✅ Active

---

### DEC-010: Pilot Focus on Events + Venues

**Date:** 2026-01-29

**Context:** Multiple archetypes defined (Venue, Organizer, Service, Community, Product). Which to prioritize?

**Decision:** Focus on Events archetype first, then Venues (brick & mortar locations).

**Rationale:** 
- Events are core to LocalLane's value prop
- Venues support local businesses (the "support local" angle)
- Other archetypes (Service, Community, Product) can come post-pilot

**Status:** ✅ Active

---

### DEC-011: Staff/Instructor Roles with Custom Permissions

**Date:** 2026-01-29

**Context:** Instructors work at multiple businesses. Need role-based access.

**Decision:** Default role templates (Manager, Instructor, Staff) with per-person permission overrides (planned).

**Role Defaults:**
| Permission | Manager | Instructor | Staff |
|------------|---------|------------|-------|
| can_create_events | ✅ | ❌ | ❌ |
| can_edit_all_events | ✅ | ❌ | ❌ |
| can_edit_assigned_events | ✅ | ✅ | ❌ |
| can_manage_staff | ✅ | ❌ | ❌ |
| can_view_analytics | ✅ | Limited | ❌ |
| can_use_checkin | ✅ | ✅ | ✅ |

**Rationale:** Flexibility for business owners. One instructor might have different permissions at different businesses.

**Status:** ✅ Roles implemented, ⏳ Custom permissions planned

---

### DEC-012: Staff Invites Stored in AdminSettings

**Date:** 2026-01-30

**Context:** Needed to store pending staff invites for users who don't have accounts yet. The Base44 Business schema doesn't have a `staff` field.

**Decision:** Store pending invites in AdminSettings with key convention:
```
staff_invites:{business_id}
```

Value is JSON array: `[{ email, role, status, invited_at }, ...]`

Active staff (users who have accounts) remain in `business.instructors` array.

**Rationale:** AdminSettings already works for flexible storage. Allows invites without schema changes. When user creates account, we can auto-link and move them from invites to instructors.

**Status:** ✅ Active

---

### DEC-013: Staff Role Badge Colors

**Date:** 2026-01-30

**Context:** Need visual distinction between different staff roles.

**Decision:** 
- **Owner:** Amber badge (bg-amber-500 text-black)
- **Manager:** Purple badge (bg-purple-500 text-white)
- **Instructor:** Blue badge (bg-blue-500 text-white)
- **Staff:** Outline badge (border-slate-500 text-slate-300)
- **Pending:** Amber outline badge (border-amber-500 text-amber-500)
- **TEAM:** Amber outline badge on business cards for staff (not owner)

**Rationale:** Clear visual hierarchy. Owner is gold (most prominent). Manager is purple (distinct, high authority). Instructor is blue (professional). Staff is subtle outline (lowest permissions).

**Status:** ✅ Active

---

### DEC-014: Staff Auto-Link on Dashboard Load

**Date:** 2026-01-30

**Context:** When a user is invited to a business, they need to be automatically added when they create an account. Base44 handles auth, so we can't hook into signup directly.

**Decision:** Check for pending invites when user visits Business Dashboard:

```javascript
// In BusinessDashboard.jsx useEffect on mount:
1. Query all AdminSettings with key starting with 'staff_invites:'
2. For each, check if current user's email matches any invite
3. If match found:
   - Add user to business.instructors
   - Add role to staff_roles:{business_id}
   - Remove invite from staff_invites:{business_id}
4. Invalidate queries to refresh business list
```

**Rationale:** Can't modify Base44 auth flow. Dashboard load is reliable trigger — user will always visit dashboard. Runs once on mount, transparent to user.

**Status:** ✅ Active

---

### DEC-015: 403 Permission Handling for Staff View

**Date:** 2026-01-30

**Context:** When a staff member (not owner) views a business's Staff Widget, fetching User records for other staff fails with 403 (Base44 permission denied).

**Decision:** Gracefully handle 403 errors by showing "Team Member" placeholder:

```javascript
// In staffUsers query:
try {
  const user = await User.filter({ id });
  return user;
} catch (error) {
  // Permission denied - return placeholder
  return { id, email: 'Team Member', _permissionDenied: true };
}
```

**UI Behavior:**
- Owners see full staff details (names, emails, remove buttons)
- Staff members see "Team Member" placeholders (no details, no edit controls)
- Add Staff button only visible to owners

**Rationale:** Staff don't need to see other staff's personal info. Graceful degradation maintains functionality while respecting Base44's permission model.

**Status:** ✅ Active

---

### DEC-016: Staff Roles Stored in AdminSettings

**Date:** 2026-01-30

**Context:** Business.staff field doesn't exist in Base44 schema. Needed to store role (manager/instructor/staff) for each staff member.

**Decision:** Store roles in AdminSettings with key convention:
```
staff_roles:{business_id}
```

Value is JSON array: `[{ user_id, role, added_at }, ...]`

Combined with:
- `business.instructors` — Array of user IDs (active staff)
- `staff_invites:{business_id}` — Pending invites

**Rationale:** AdminSettings is flexible key-value store that already works. Keeps role data separate from Business entity. Can query by business ID.

**Status:** ✅ Active

---

### DEC-017: Red/Orange for Destructive Actions

**Date:** 2026-01-31

**Context:** Style Guide specified no functional color-coding (icons/actions should be gold or white only). However, destructive actions like Delete and Cancel Event need clear visual warning.

**Decision:** Allow red and orange for destructive actions only:
- Delete: `text-red-500` / `bg-red-500`
- Cancel Event: `text-orange-500` / `bg-orange-500`

**Rationale:** User safety trumps strict style consistency. Red/orange are universally understood danger signals. Limited to destructive contexts only—not for general status indicators.

**Status:** ✅ Active

---

### DEC-018: Replace Radix UI Primitives with Pure Divs in Controlled Toggle Contexts

**Date:** 2026-01-31

**Context:** shadcn/ui Checkbox and Switch components (built on Radix UI primitives) have internal state management that conflicts with controlled mode. When a parent div onClick toggles state → the component re-renders with a new `checked` prop → Radix's internal state sync fires `onCheckedChange` → which triggers another state update → infinite render loop ("Maximum update depth exceeded").

This was hit three times:
1. Accessibility checkboxes in EventEditor (Checkbox component)
2. Recurring events toggle in EventEditor (Switch component)
3. Punch Pass toggle in EventEditor (Switch component)

Adding `pointer-events-none` to the component does NOT fix it — the internal state sync fires regardless of pointer events.

**Decision:** When a parent element handles click to toggle state, replace shadcn Checkbox/Switch with pure CSS visual equivalents:

**Checkbox replacement:**
```jsx
<div className={cn(
  "h-4 w-4 rounded border flex items-center justify-center",
  isChecked ? "bg-amber-500 border-amber-500" : "border-slate-600 bg-transparent"
)}>
  {isChecked && <svg className="h-3 w-3 text-black" viewBox="0 0 24 24"><polyline points="20 6 9 17 4 12" /></svg>}
</div>
```

**Switch replacement:**
```jsx
<div className={cn(
  "relative inline-flex h-6 w-11 items-center rounded-full transition-colors flex-shrink-0 pointer-events-none",
  isOn ? "bg-amber-500" : "bg-slate-600"
)}>
  <span className={cn(
    "inline-block h-5 w-5 rounded-full bg-white shadow-sm transition-transform",
    isOn ? "translate-x-5" : "translate-x-0.5"
  )} />
</div>
```

**Rule:** When a parent div has `onClick` to toggle state, any Checkbox/Switch inside MUST be a pure visual div with zero internal state. Standalone Switches (no parent onClick) can still use the shadcn component.

**Rationale:** Radix primitives are designed as uncontrolled-first with optional controlled mode. Their internal state reconciliation creates render loops in our click-delegation pattern. Pure CSS divs have zero state management overhead.

**Status:** ✅ Active — Applied to all toggle tiles in EventEditor

---

### DEC-019: EventEditor ↔ EventFormV2 Feature Parity Complete

**Date:** 2026-01-31

**Context:** EventEditor (Community Node, used by Tier 1 & 2 businesses) was missing many features present in EventFormV2 (Event Node, used by Partner Nodes). Users switching between the two experienced different UX, which undermined trust and consistency.

**Decision:** Achieve full feature and interaction parity between the two forms. The following were added to EventEditor to match EventFormV2:

**Features added:**
- End time / end date picker (duration mode or end time mode)
- Virtual event support (URL + platform fields)
- Location TBD toggle
- Multiple ticket types (name, price, limit — tier-gated)
- Recurring events (weekly/biweekly, day selection, end date)
- Accessibility features (admin-managed checkboxes)
- Accept RSVPs toggle
- Additional notes textarea
- Draft auto-save (localStorage, 30s interval)
- Save as Draft button
- Multi-image upload (up to 3 with hero designation)
- Event type multi-select chips (replaced dropdown)
- Pricing type 2×2 grid layout (replaced single row)

**Interaction parity:**
- Clickable tile toggles (Recurring, Punch Pass) — click anywhere on the row
- Info banners ("Each occurrence will be created as a separate event")
- Punch Pass warnings ("Cannot be free") and earnings display
- Three-tier button hierarchy: Cancel (ghost) / Save as Draft (outline) / Publish (primary)

**What intentionally differs:**
- EventEditor uses `useOrganization()` hook (Community Node tier check)
- EventFormV2 uses `useOrganizationTier()` hook (Event Node tier check)
- EventEditor sends `punch_pass_accepted` (backend field name), EventFormV2 sends `punch_pass_eligible`
- EventEditor sends `event_type` (single) + `event_types` (array); backend uses single field

**Rationale:** Users should feel like they're using the same product regardless of which node they're in. Consistency builds trust — a core LocalLane principle.

**Status:** ✅ Active — Full parity achieved. Future changes to either form should be mirrored.

---

### DEC-020: Event Entity Schema Expanded

**Date:** 2026-01-31

**Context:** The Event entity gained several new fields during the EventEditor parity work. These need to be documented as the canonical schema.

**Decision:** The Event entity now includes these additional fields beyond the original spec:

| Field | Type | Notes |
|-------|------|-------|
| `is_virtual` | boolean | Whether event is virtual/online |
| `virtual_url` | string | Meeting link (Zoom, etc.) |
| `virtual_platform` | string | Platform name |
| `is_location_tbd` | boolean | Location not yet determined |
| `event_types` | string[] | Multi-select (array), backend also stores first as `event_type` |
| `ticket_types` | JSON | Array of { name, price, quantity_limit } |
| `is_recurring` | boolean | Whether event repeats |
| `recurrence_pattern` | string | 'weekly' or 'biweekly' |
| `recurrence_days` | string[] | ['Mon', 'Wed', etc.] |
| `recurrence_end_date` | ISO datetime | When recurring series ends |
| `accessibility_features` | string[] | From admin-managed config |
| `accepts_rsvps` | boolean | Whether RSVPs are enabled |
| `additional_notes` | string | Free-text notes |

**Backend note:** Base44 schema may not have all these fields defined. Fields are sent in the create/update payload and stored if the schema accepts them. Fields not in the schema are silently dropped.

**Rationale:** Document the full intended schema even if Base44 doesn't enforce all fields. This serves as the target schema for when we move to a more structured database.

**Status:** ✅ Active

---

### DEC-021: Replace Star Ratings with Recommendation System

**Date:** 2026-02-01

**Context:** The original system used traditional 1-5 star ratings with a Review entity. Star ratings are anonymous, gameable, and don't build trust — they tell you what strangers think, not what your neighbors experienced.

**Decision:** Replace the entire star-rating system with a trust-first recommendation model:

| Type | What It Is | Accountability |
|------|-----------|---------------|
| **Nod** | "I recommend this" | Name attached, one-click |
| **Story** | Detailed experience | Name attached, narrative |
| **Vouch** (Phase 2) | Weighted endorsement | Earned trust, higher weight |

**New entity:** `Recommendation` with fields: business_id, user_id, user_name, type, content, service_used, photos, is_active.

**New business fields:** `recommendation_count`, `nod_count`, `story_count` (replace `review_count`, `average_rating`).

**New components:** TrustSignal (badge display), StoryCard (story rendering), NodAvatars (avatar row).

**Ranking:** Recommendations count → Story count → Newest. No paid placement.

**Removed:** StarRating.jsx, ReviewCard.jsx, all Review.filter queries, star rating display from BusinessCard/BusinessProfile/OverviewWidget/BusinessDashboardDetail.

**Rationale:** LocalLane's differentiator is trust over algorithms. Anonymous star ratings undermine that. "Your neighbor Sarah recommends this plumber" is more powerful than "4.2 stars from 47 reviews." Names create accountability, which creates quality.

**Status:** ✅ Active — Phase 1 (Nod + Story) complete. Phase 2 (Vouch) planned.

---

### DEC-022: Remove Boost/Bump Paid Placement System

**Date:** 2026-01-31

**Context:** The codebase contained a boost system allowing businesses to pay for promoted placement in search results. This included boost credits, boost duration, boost badges, and admin settings for boost configuration.

**Decision:** Remove the entire boost system. Files affected:
- rankingUtils.jsx — removed boost scoring and "featured" band
- BusinessCard.jsx — removed boost badge and boost-related props
- AdminSettingsPanel.jsx — removed boost duration setting
- OverviewWidget.jsx — removed "Boost Credits" stat card
- Home.jsx — removed boost-based featured business queries
- SearchResultsSection.jsx — simplified to single grid (no featured/regular bands)

**Rationale:** Paid placement directly contradicts LocalLane's "trust over ads" mission. If businesses can buy visibility, the ranking system is compromised. Revenue comes from tier subscriptions and punch pass transactions, not advertising.

**Status:** ✅ Active — All boost code removed. Zero references remain.

---

### DEC-023: Gold Standard Dark Theme — Complete Coverage

**Date:** 2026-02-01

**Context:** Many pages and components had light-theme holdovers (bg-white, bg-slate-50, text-slate-900, bg-amber-100, etc.) despite the Gold Standard specifying dark-only design.

**Decision:** Systematically convert every page and component to dark theme. Approach was page-by-page audit with targeted find-and-replace prompts.

**Pages converted:** Home, Directory (rebuilt from scratch), CategoryPage, BusinessProfile, Search, BusinessOnboarding (tier icons), BusinessDashboardDetail (full conversion + recommendation migration).

**Global fix:** CSS variables in index.css updated to dark palette. Radix UI tab hover states overridden globally to prevent white background flash.

**Pattern established for tier icon backgrounds:**
- Partner: `bg-amber-500/20` + `text-amber-500`
- Standard: `bg-blue-500/20` + `text-blue-400`
- Basic: `bg-slate-800` + `text-slate-400`

**Rationale:** Visual inconsistency undermines trust. Users seeing white flashes or light-themed sections breaks the premium feel. Dark theme is the brand.

**Status:** ✅ Complete — No light-theme holdovers remain.

---

### DEC-024: Dead Code Cleanup — Remove 10 Unused Files

**Date:** 2026-02-01

**Context:** After the recommendation system migration, boost removal, and CategoryPage rebuild, several files had zero imports anywhere in the codebase.

**Decision:** Delete all files with zero import references:

**Deleted (zero imports):**
1. StarRating.jsx — replaced by TrustSignal
2. ReviewCard.jsx — replaced by StoryCard
3. OverviewWidget.tsx.jsx — exact duplicate
4. MigrateCategories.jsx — one-time migration, completed
5. migrateCategoriesScript.jsx — helper for above
6. cleanupDuplicates.jsx — helper for above
7. resetCategories.jsx — helper for above
8. Categories.jsx — replaced by CategoryPage
9. BusinessSelector.jsx — never imported
10. BusinessSelector.tsx.jsx — duplicate of above

**Kept (confirmed active):**
- SpokeDetails.jsx — used in Admin Partners tab
- BuildLane.jsx — shelved but planned future feature

**Also cleaned:** pages.config.js (removed MigrateCategories and Categories entries), stale Review.filter queries from BusinessProfile and BusinessDashboardDetail.

**Rationale:** Dead code creates confusion for AI agents (Cursor, Claude) scanning the codebase. Clean code = faster iteration.

**Status:** ✅ Complete

---

### DEC-025: Entity Permission Security Audit

**Date:** 2026-02-01

**Context:** All 20 Base44 entities were set to "Public" with full read/write/delete access for any authenticated user. This meant any logged-in user could read, modify, or delete any record in the system.

**Decision:** Three-phase permission lockdown:

- **Phase 1 (Complete):** Delete unused entities (Review, Bump), lock down AdminAuditLog and Concern
- **Phase 2 (Complete):** Lock down PunchPass (3 entities), Region, Archetype, CategoryGroup, SubCategory
- **Phase 3 (Planned):** Remaining 10 entities require service role function migration before permissions can be tightened

**Entities locked down (11 of 18):**
- AdminAuditLog: Admin-only read/create/update, delete disabled
- Concern: Authenticated create, admin-only read/update/delete
- PunchPass: Owner-only read, admin-only create/update/delete
- PunchPassTransaction: Owner-only read, admin-only create/update/delete
- PunchPassUsage: Owner-only read, admin-only create/update/delete
- Region, Archetype, CategoryGroup, SubCategory: Public read, admin-only create/update/delete

**Phase 3 entities (require service role functions):**
Business, Event, AdminSettings, RSVP, Location, User, Recommendation, Spoke, SpokeEvent, CategoryClick

**Rationale:** Critical security vulnerability. Public write access to entities like AdminSettings or Business means any user could modify platform configuration or other businesses' data.

**Status:** ✅ Phase 1 & 2 Complete — ⏳ Phase 3 Deferred (requires code changes)

---

### DEC-026: Dead Code Cleanup — Review/Bump/Boost Remnants

**Date:** 2026-02-01

**Context:** After removing the Review and Bump entities (DEC-025) and migrating to the recommendation system (DEC-021, DEC-022), legacy code references remained in the codebase.

**Decision:** Remove all remaining references to:
- `review_count` fallbacks in OverviewWidget, TrustSignal, rankingUtils
- `average_rating` references
- `Star` icon import in BusinessDashboardDetail (legacy review usage)
- `'reviews'` sort option in Search.jsx
- `src/components/reviews/` folder (if still present)
- Global sweep for `StarRating`, `ReviewCard`, `WriteReview`, `boost_credits`, `boost_duration`

**Rationale:** Dead code confuses AI agents scanning the codebase and creates false signals about system capabilities. Clean code = faster iteration.

**Status:** ✅ Complete

---

### DEC-027: Microbusiness as Next Archetype

**Date:** 2026-02-01

**Context:** After Events archetype stabilizes, need to prove the archetype system scales. Previously decided Nonprofit/Church (see earlier strategic decision). Revisited based on market opportunity.

**Decision:** The next archetype after Events will be Microbusiness — people who sell from home, dog walkers, tutors, lawn care, cottage food operators, etc.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Nonprofit/Church | Real test case (Doron's church) | Narrower audience |
| **Microbusiness** | Huge underserved market, expands user base, aligns with "support local" | No single test case |

**Reasoning:** Microbusinesses are the most underserved segment in local commerce. They don't fit Yelp, they're too small for a website, and they usually just have Instagram or word-of-mouth. LocalLane can be their digital storefront. This archetype also grows the platform's user base directly — every dog walker, home baker, and tutor becomes both a business and a community member.

**Key Differences from Events Archetype:**
- No physical venue (service area or home-based instead)
- Solo operator vs. team (may not need staff management)
- Portfolio/showcase emphasis over event calendar
- Service listings instead of event listings
- Booking/scheduling potential (future)

**Consequences:** Onboarding wizard needs new paths for microbusiness. Will reveal which features are truly universal vs. archetype-specific.

**Status:** ⏳ Planned — Design conversation needed before implementation

---

### DEC-028: Community Pass Replaces Stored-Value Punch Pass

**Date:** 2026-02-02

**Context:** Legal research (LEGAL-RESEARCH.md) revealed that the original Punch Pass design — buy credits redeemable at unaffiliated businesses — constitutes money transmission under Oregon ORS 717. The multi-merchant stored-value model requires a money transmitter license not feasible at bootstrap scale.

**Decision:** Restructure from stored-value credits to a membership subscription model (Community Pass). Users pay a monthly subscription fee to LocalLane. They receive "punches" as membership access tokens, not stored value. Businesses receive revenue share from the membership pool, not per-transaction transfers.

**Key changes from old model:**
- Subscription replaces one-time credit purchase
- Monthly punch expiration replaces 12-month window
- Revenue-share pool replaces per-redemption settlement
- Zero commission on direct purchases (businesses are merchant of record via `on_behalf_of`)
- No unredeemed balance liability (membership benefits, not stored value)

**Rationale:** Eliminates money transmitter licensing requirement. Creates predictable recurring revenue. Aligns with proven models (1Pass, ClassPass, gym memberships). Better product: membership encourages exploration, stored value encourages hoarding.

**Consequences:**
- STRIPE-CONNECT.md must be rewritten to reflect new charge model
- Settlement logic changes from per-redemption to monthly pool distribution
- PunchPass entity fields evolve (see COMMUNITY-PASS.md)
- Need revenue share agreements with participating businesses
- Need professional legal review confirming structure before money flows

**Status:** ✅ Active — Approved direction. Implementation pending Stripe setup.

**Reference:** COMMUNITY-PASS.md (private repo) contains full concept. LEGAL-RESEARCH.md contains the legal analysis.

---

### DEC-029: Organism Concept as North Star

**Date:** 2026-02-02

**Context:** LocalLane needed a differentiating vision beyond "another business directory." The platform's mission (revitalize local commerce, fund youth scholarships) needed a user-facing expression that makes community health visible and meaningful.

**Decision:** Adopt the Organism Concept as the platform's north star vision. The organism is a living, animated visual element that reflects community health at three fractal scales: personal, network, and community. It mirrors real participation rather than creating artificial engagement. Decision filter for new features: "Does this make the organism more alive?"

**Principles:**
- Mirror, don't manipulate (reflect reality, no artificial urgency)
- Growth, not points (no badges, leaderboards, or XP)
- Seasons, not streaks (rest periods are healthy, not failures)
- Connection, not competition (organisms don't compete)

**Rationale:** No competing platform offers anything like this. Creates genuine emotional connection to community health data. Aligns incentives: the organism thrives when the community thrives, not when the algorithm extracts attention.

**Consequences:**
- Phase 1 is buildable now using existing data in MyLane
- Four implementation phases from CSS/SVG → Lottie → Canvas → social organisms
- Network passes (Recess, Creative Alliance, Harvest, Gathering Circle) each get organism identity
- MyLane GreetingHeader is the first home for the organism

**Status:** ✅ Active — North star vision. Build incrementally.

**Reference:** ORGANISM-CONCEPT.md (private repo) contains full vision, lifecycle states, vitality formulas, and technical approach.

---

### DEC-030: Private Repository for Sensitive Strategy

**Date:** 2026-02-02

**Context:** Public spec-repo contained competitive strategy, legal research, and revenue model details. While repos are public for development convenience (DEC: Repositories Are Public), strategy documents give away LocalLane's most differentiated thinking.

**Decision:** Create `private` repository for sensitive documents. Public spec-repo may reference that private concepts exist but must not reproduce strategy details, revenue formulas, competitive analysis, or legal reasoning.

**Documents moved to private:**
- ORGANISM-CONCEPT.md
- COMMUNITY-PASS.md
- LEGAL-RESEARCH.md
- SILVER-BARTER-CONCEPT.md

**Rationale:** Competitive advantage is in the vision and model, not the code. Code repos can stay public. Strategy stays private.

**Consequences:**
- Claude project sync established for private repo collaboration
- Public spec docs must be audited for leaked strategy details
- New sensitive docs default to private repo
- Public docs can say "see private repo" for full context

**Status:** ✅ Active

---

### DEC-031: Status Indicator Color Policy

**Date:** 2026-02-02

**Context:** Codebase audit found inconsistent use of blue, teal, and other colors for status indicators in admin and dashboard contexts. Needed a clear policy that respects the Gold Standard while allowing functional color variety.

**Decision:** Adopt the following color policy for admin/dashboard status indicators:

| Color | Usage | Example |
|-------|-------|---------|
| Amber (`amber-500`) | Primary/active states | Active subscriptions, current selections |
| Blue (`bg-blue-500/20 text-blue-400`) | Informational/neutral status | "Standard" tier badge, info callouts |
| Teal (`bg-teal-500/20 text-teal-400`) | Wizard selection states | Onboarding step selection, toggle active |
| Red/Orange | Destructive actions only | Delete confirmation, error states |

**Scope:** Admin panel and business dashboard only. Public-facing UI (Directory, Events, MyLane) stays strictly amber + white + slate.

**Rationale:** Pure amber-only in admin contexts makes it hard to distinguish between different status types. Functional color provides information hierarchy without violating the Gold Standard on pages users see.

**Status:** ✅ Active

**Reference:** Also encoded in claude.md under "Status Indicator Colors."

---

### DEC-032: Community Pass Pricing Locked

**Date:** 2026-02-05

**Context:** Extensive pricing research completed: Greg Crabtree's Simple Numbers framework, ClassPass competitive analysis, 1Pass Eugene comparable, Eugene gymnastics/climbing/trampoline pricing, off-peak revenue strategy. Needed to lock pricing to move forward with Stripe integration and marketing.

**Decision:** Community Pass pricing locked at:
- Base membership: $49/mo (12 Joy Coins)
- Each additional allocation: $39/mo (12 Joy Coins)
- 75/25 split: 75% of net revenue (after Stripe fees) to business pool, 25% to LocalLane
- Unused coins flow to scholarship pool at month's end
- Monthly billing cycle — coins reset each period

**Household examples:**
- Solo: $49/mo, 12 coins
- Couple / Power User: $88/mo, 24 coins
- Family of 4: $166/mo, 48 coins

**Market validation:**
- The Circuit climbing family membership: $200/mo
- Two kids in gymnastics (Eugene): $160-240/mo for one activity
- ClassPass: $89/mo for one person with limited credits
- 1Pass Eugene: $60 one-time, summer only, kids only — proved scan-based revenue share works in Eugene

**Founding member rates:** TBD — to be decided before launch. Suggested ~$39 base / ~$29 additional.
**Explorer Pass (seasonal):** Higher price, more coins. Summer product. Pricing deferred until base proves out.

**Rationale:** Crabtree principle — price for profit from day one, easier to lower than raise. $166/mo for a family of 4 is still less than most single-activity memberships. Per-coin business payout of $2.50-2.97 is defensible as incremental revenue on empty capacity.

**Status:** ✅ Active — Locked

---

### DEC-033: Joy Coin Pool Model (Coins, Not People)

**Date:** 2026-02-05

**Context:** Original model assigned additional members (people) to the membership. Reframed: the $39 add-on is additional Joy Coins to the household pool, not an additional person.

**Decision:** Memberships are coin allocations, not person assignments. A household buys a base (12 coins) and as many additional allocations (12 coins each) as they want. The family shares one pool. No tracking individuals, no "who's on the plan" management.

If a solo person wants 24 coins for heavy usage, they pay $88. If a family of 6 wants 48 coins, they buy base + 3 add-ons for $166. The membership doesn't care who uses the coins.

**Rationale:** Simpler billing, simpler UX, no awkward "how many people" questions. Promotes family flow — the household decides how movement gets distributed. Aligns with Organism philosophy: the family moves as one unit.

**Status:** ✅ Active

---

### DEC-034: Business Tier Pricing Locked

**Date:** 2026-02-05

**Context:** Business tier pricing had been discussed but never formally locked. Needed to finalize alongside Community Pass pricing. Business tiers are a separate product from Community Pass — they include network membership, community of business owners, events, problem-solving (chamber of commerce energy).

**Decision:** Business tier pricing locked at:

| Tier | Code | Monthly | What They Get |
|------|------|---------|---------------|
| Basic | `basic` | Free | Listing, events (reviewed before publish), directory presence |
| Standard | `standard` | $79/mo | Auto-publish events, Community Pass participation, analytics, revenue share from Joy Coin pool, business network membership |
| Partner | `partner` | $149/mo | Earned status, own branded node, full autonomy, enhanced features |

**Founding business deal:** First 3 months free at Standard tier. Gives LocalLane time to prove value before businesses see a charge.

**Partner tier remains earned** (see existing strategic decision). Requirements: 6+ months at Standard, 4.5+ rating, engagement threshold, 3+ vouches, zero violations.

**Rationale:** $79/mo Standard is where businesses access the Joy Coin pool — their incentive to upgrade. $149/mo Partner is premium earned tier. Business tiers are a separate value proposition from Community Pass: being part of a local business network with events, problem-solving, visibility, and customer flow.

**Status:** ✅ Active — Locked

---

### DEC-035: Earn-Your-Pass Program

**Date:** 2026-02-05

**Context:** Community Pass includes a scholarship program (unused coins flow to families in need). But Doron identified a deeper opportunity: kids can earn their pass through community contribution rather than only receiving it as charity.

**Decision:** Establish an Earn-Your-Pass program where kids work at participating farms and/or farmers markets to earn their monthly Community Pass access. A month of consistent contribution earns a month of pass access.

**Initial pathways:**
- Farm work during growing season
- Farmers market assistance (setup, booths, cleanup)
- Future: expand to other community contribution pathways

**Principles:**
- This is participation in the Organism, not charity
- "Teach a man to fish" — opportunity over handout
- The farm feeds the market, the market feeds the community, the community feeds the kids, the kids feed the farm. Circulation.
- Tracking is manual at first (farm coordinator confirms contribution)

**Rationale:** Aligns with LocalLane's core philosophy of circulation over extraction. Kids aren't passive recipients — they're active contributors to the community. Creates deeper connection between youth, local agriculture, and the broader network. Powerful story for The Good News newsletter and community building.

**Status:** ✅ Active — Initial program design. Implementation details TBD.

---

### DEC-036: AccessWindow Capacity Per-Person

**Date:** 2026-02-06

**Context:** The AccessWindow entity lets businesses set Joy Coin hours with optional capacity limits. Question was whether capacity should be per-family (household unit) or per-person (individual headcount).

**Decision:** Per-person. Family sizes vary — a family of 2 and a family of 6 have very different capacity impact. Per-person gives businesses precise control over how many bodies are in their space at any given time.

**Status:** ✅ Active

---

### DEC-037: Show Estimated Revenue Before Stripe Connect

**Date:** 2026-02-06

**Context:** The Revenue tab displays pool share amounts, but Stripe Connect won't be active until a later phase. Question was whether to show dollar amounts before real payouts are possible.

**Decision:** Yes — show estimated revenue with a persistent banner: "Estimated payout · Deposits begin when Stripe Connect activates." Businesses need to see the value story building in real time, not discover it after payments are wired up.

**Status:** ✅ Active

---

### DEC-038: Network Comparison Minimum 10 Businesses

**Date:** 2026-02-06

**Context:** The Revenue tab includes a Network Position section showing percentile rankings and elasticity signals. With few businesses, these metrics are meaningless and potentially feel invasive.

**Decision:** Require 10+ businesses in the network before showing percentile rankings, network averages, or comparison data. Below 10, hide the Network Position section entirely. Individual business stats (their own revenue, scans, trends) always show regardless of network size.

**Status:** ✅ Active

---

### DEC-039: Microbusiness Archetype Spec Complete

**Date:** 2026-02-07

**Context:** DEC-027 identified Microbusiness as the next archetype. This spec documents six sub-types (Home Food, Home Services, Personal Services, Property Management, Creative/Maker, Local Delivery), a $9/month tier between free users and $79 Standard businesses, Oregon-specific compliance research, and integration points with Community Pass, Harvest Network, and the Guest Welcome system for short-term rental hosts.

**Decision:** Microbusiness archetype spec is complete and ready for implementation after Events archetype stabilizes. Introduces a new $9/month pricing tier for solo operators.

**Key Design Choices:**
- Six sub-types with shared core fields and sub-type-specific extensions
- $9/month fills the gap between free user tier and $79 business tier
- Dual identity: microbusiness operators are both providers and community members
- Property management (including short-term rental) included as a sub-type
- Oregon compliance awareness built into onboarding (not legal advice, links to official sources)
- Community Pass participation optional for microbusinesses

**Status:** ✅ Spec Complete — ⏳ Implementation follows Events stabilization

---

### DEC-040: Nonprofit Pricing — 50% Discount on Business Tiers

**Date:** 2026-02-07

**Context:** Eugene-Springfield has 2,443 registered nonprofits. Nonprofits are high-value nodes (volunteers, members, community connections) but cash-strapped. Need pricing that's accessible without devaluing the platform or creating sustainability problems.

**Decision:** Nonprofits receive a 50% discount on standard business tier pricing:

| Tier | Nonprofit Price | Business Price | Discount |
|------|----------------|----------------|----------|
| Basic | Free | Free | Same |
| Nonprofit Standard | $39/mo | $79/mo | 50% |
| Nonprofit Partner | $79/mo | $149/mo (earned) | 47% |

**Qualification:** 501(c)(3) EIN or Oregon nonprofit registration. Self-reported during onboarding, spot-checked if needed. Binary qualification — no sliding scale.

**Founding nonprofit deal:** First 10 nonprofits get 3-6 months free to prove platform value.

**Rationale:**
- $39/month is realistic for orgs with operational budgets and grants
- Keeps pricing clean: businesses $79, nonprofits $39, microbusinesses $9
- Free tier already covers orgs that only need visibility
- 50 nonprofits × $39 = $1,950/month — meaningful revenue
- Signals partnership, not charity
- Partner tier discount applies to fee, not qualification bar (still earned)

**Revenue projection (conservative Year 1):** 38 nonprofits, $702/month ($8,424 annual). Optimistic: 62 nonprofits, $1,603/month ($19,236 annual).

**Made by:** Doron + Claude

**Status:** ✅ Active

---

### DEC-041: Joy Coins Non-Transferable; Expired Coins Fund Scholarship Pool

**Date:** 2026-02-07

**Context:** Legal research established non-transferability as a key guardrail keeping Community Pass outside money transmitter regulation (ORS 717) and stored value classification (FinCEN). The Joy Coins system had a P2P transfer feature (JoyCoinsTransfer.jsx) that conflicted with this guardrail. Additionally, the ClassPass lawsuit (Blackburn v. ClassPass, July 2025) demonstrated litigation risk around expiring credits — making the scholarship pool flow from expired coins both a legal defense and a community benefit.

**Decision:**
- Joy Coins are non-transferable between members
- JoyCoinsTransfer.jsx and all transfer code removed from codebase
- Unused Joy Coins expire monthly and flow to Community Scholarship Pool
- Scholarship Pool funds Community Pass access for youth and families in need
- Terms of Service updated: non-transferability confirmed, scholarship pool language added
- LEGAL-RESEARCH.md updated with CARD Act / EFTA analysis

**Alternatives considered:**
- Keep P2P transfers with limits — rejected (regulatory risk, complexity)
- Allow gifting to scholarship pool only — unnecessary (automatic flow is simpler)

**Status:** Active

---

### DEC-042: Phase 3 Security Audit — Service Role Migration

**Date:** 2026-02-07

**Context:** 10 entities remained publicly writable by any authenticated user (DEC-025). Base44 confirmed support for base44.asServiceRole in server functions to bypass entity-level permissions.

**Decision:** Migrate all client-side entity writes to server functions with authorization checks, then lock entity permissions to Admin only. Server functions verify the calling user's role (admin, owner, staff) before executing writes with elevated permissions.

**Server functions created:**
- **updateAdminSettings.ts** — Handles all AdminSettings writes with role-based authorization. Actions: filter, create, update, check_my_invites, accept_invite.
- **updateBusiness.ts** — Handles Business entity writes. Actions: update (owner/admin/manager), add_staff_from_invite (invited users), update_counters (any authenticated user for recommendations).
- **manageEvent.ts** — Handles Event entity writes. Actions: create, update, delete, cancel. Authorization: admin, owner, or business staff.
- **updateUser.ts** — User profile updates. Action: update_profile (self-only with field allowlist: display_name, phone, home_region).
- **manageRecommendation.ts** — Recommendation + Concern writes. Actions: create, update, remove (owner-only + admin), create_concern (any authenticated).
- **manageRSVP.ts** — Full RSVP lifecycle with Joy Coin orchestration. Actions: rsvp (any authenticated), cancel (self-only with refund/forfeit logic), checkin (admin/owner/staff), no_show (admin/owner/staff).

**Entities locked (Phase 3a + 3b + 3c + 3d):**
- AdminSettings: Read Authenticated, Write Admin only
- Location: Read Public, Write Admin only
- Spoke: Read Public, Write Admin only
- SpokeEvent: Read Public, Write Admin only
- CategoryClick: Read Admin only, Create Authenticated
- Business: Read Public, Write Admin only
- Event: Read Public, Write Admin only
- User: Read Authenticated, Write Admin only
- Recommendation: Read Public, Write Admin only
- Concern: Read Authenticated, Write Admin only
- RSVP: Read Authenticated, Write Admin only

**Rationale:** Eliminates the ability for any authenticated user to modify any business, event, or platform setting. Server functions enforce business logic (ownership, staff roles) that entity-level permissions alone cannot express.

**Status:** ✅ Complete — all 10 entities secured

**Noted for future:**
- Co-owner role not currently supported. Only one owner_user_id per business. Multiple owners may be needed for pilot (e.g. husband/wife running a business together). Would require either an owners array on Business or a new role level in staff_roles that grants owner-equivalent permissions.
- Manager role cannot currently edit events. manageEvent.ts checks business.instructors array but doesn't differentiate permissions by role. DEC-011 specifies managers should have can_create_events and can_edit_all_events.

---

### DEC-043: Co-Owner Role via Staff Roles System

**Date:** 2026-02-08

**Context:** Businesses often have multiple people who need owner-level access — spouses running a family business, partners in a gym, co-founders. The Business entity has a single `owner_user_id` field. Adding an `owners` array would require schema changes and break existing authorization logic.

**Decision:** Add `co-owner` as a new role in the existing `staff_roles` system (stored in AdminSettings as `staff_roles:{business_id}`). Co-owners get owner-equivalent permissions with two exceptions: cannot delete the business, cannot change Stripe account settings. Multiple co-owners supported.

**Implementation:**
- Server functions updated: `canEditBusiness` (updateBusiness.ts), `canEditEvent` (manageEvent.ts), `canManageEvent` (manageRSVP.ts) all check for `co-owner` role
- UI updated: StaffWidget.jsx, BusinessEditDrawer.jsx, BusinessCard.jsx all support co-owner role selection and badge rendering
- Badge: amber outline (`border-amber-500 text-amber-500`) — visually linked to Owner's solid amber badge
- Bug fix: `addStaffMutation` was not writing role to `staff_roles` AdminSettings for directly-added users (only worked via invite flow). Fixed in commit 131f24d.
- No schema changes required — uses existing `instructors` array + `staff_roles` AdminSettings pattern

**Commits:** 0df37d6 (server auth), 5bf3454 (UI), 131f24d (role write fix)

**Badge color spec (extends DEC-013):**
- Co-Owner: `border-amber-500 text-amber-500` (outline)

**Status:** ✅ Complete

---

### DEC-044: TAB_CONFIG Archetype Dashboard Pattern

**Date:** 2026-02-08

**Context:** The Business Dashboard currently renders tabs for the Events archetype only. As new archetypes are added (Property Management, Construction, Home Food, Nonprofit), each needs different tabs while sharing the same dashboard skeleton (header, tab bar, Home tab, Settings tab).

**Decision:** Define a `TAB_CONFIG` object keyed by `archetype.value` that maps each archetype to its tab set. Dashboard renders tabs from config, not hardcoded JSX.

**Tab mapping:**
- Events: Home, Joy Coins, Revenue, Events, Settings
- Property Management: Home, Properties, Finances, Maintenance, Settings
- Construction/Trades: Home, Bids, Jobs, Billing, Settings
- Home Food: Home, Products, Orders, Markets, Settings
- Nonprofit: Home, Programs, Events, Volunteers, Settings

**Shared skeleton:** Header (business name, tier badge, organism placement), tab bar, Home tab, Settings tab, admin visibility controls.

**Rationale:** Architecture scales to N archetypes without duplicating dashboard code. Each archetype's unique tabs are separate components loaded by config. Shared components stay DRY.

**Status:** 🔲 Designed — Build when second archetype is ready

---

### DEC-045: Silver Dime Layer — Parallel Physical Currency

**Date:** 2026-02-08

**Context:** Joy Coins are non-transferable digital membership benefits with no cash value (critical for ORS 717 compliance). Community members also want a physical medium of exchange that embodies local economic sovereignty. Pre-1965 silver dimes (90% silver, 0.0723 troy oz, ~$2-3 at spot) fit the scale of everyday local transactions.

**Decision:** Two parallel layers, legally separate, philosophically aligned:

**Digital Layer (Joy Coins):** Non-transferable, no cash value, community-specific, platform-mediated, monthly reset with expired coins flowing to Scholarship Pool. Legal status: membership access token.

**Physical Layer (Silver Dimes):** Pre-1965 dimes as barter commodity, intrinsic metal value, portable across communities, hand-to-hand between willing parties. LocalLane is directory only — never custodies, exchanges, or sets prices for silver.

**Critical firewall:** No exchange rate, no conversion, no equivalence between Joy Coins and silver dimes. They coexist but never convert.

**Cross-community bridge:** Joy Coins are community-specific (Eugene pool, Bend pool). Silver dimes are portable — the physical passport between communities.

**Earn-Your-Pass integration:** Kids earn both Community Pass access (digital) AND silver dimes per shift (physical). Teaches work has value, money has history, community circulation matters.

**Bullion dealer role:** Liquidity provider, reference price display, dime supplier, authentication, education hub (Sound Money 101 workshops).

**Lawyer questions flagged:** Does directory create obligation? Does Scholarship Pool buying dimes change legal status? Does displaying reference price = price-setting?

**Full concept doc:** SILVER-DIME-LAYER.md (private repo)

**Status:** 🔲 Designed — Phase 0 foundation exists (accepts_silver field, filter, bullion category)

---

### DEC-046: Custom Role Permissions (Deferred)

**Date:** 2026-02-08

**Context:** Current role system is coarse — co-owner and manager have identical permissions, instructor and staff have limited differentiation. DEC-011 originally specified granular manager permissions (`can_create_events`, `can_edit_all_events`) that were never built. Real businesses with larger teams will need per-person permission toggles.

**Decision:** Design accepted, build deferred. Architecture:
- Each role gets a default permission object (e.g., manager defaults to `can_edit_business: true, can_create_events: true, can_manage_staff: false`)
- Owner can toggle individual permissions per staff member, overriding role defaults
- Server functions check specific permission flags instead of role names
- Permission object stored alongside role in `staff_roles` AdminSettings entries

**Rationale:** No pilot business currently needs this granularity. Co-owner (DEC-043) handles the immediate need (spouse/partner access). Build when a business with 5+ distinct staff roles requests it.

**Status:** 🔲 Designed, Deferred — Build when real business need emerges

---

## Strategic Decisions (2026-01-27)

### Next Archetype: Nonprofit/Church (Superseded by DEC-027)

**Decision:** The second archetype (after Events) will be nonprofit/church organizations.

**Context:** We need to prove the archetype system works by building a second type. Options included: mechanic shop, online store, restaurant, nonprofit.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Mechanic | Common business type | No immediate test case |
| Online Store | High demand | Complex (inventory, shipping) |
| Restaurant | Many local options | Complex (menus, ordering) |
| **Nonprofit/Church** | Real test case (Doron's church), similar data model to events | Narrower audience |

**Reasoning:** Doron has direct access to a church that currently uses paper/texts/conversations with no digital system. This provides real users with real pain, fast feedback loop, similar features to events (announcements, calendar, RSVPs), and proves the archetype system works.

**Consequences:** Church node becomes priority after Event Node stabilizes. We'll learn what's truly archetype-specific vs. universal.

**Made by:** Doron + Claude

---

### Tier 3 is Earned, Not Purchased

**Decision:** Partner Node status (Tier 3) cannot be bought — it must be earned through demonstrated trustworthiness.

**Context:** Most platforms sell premium tiers. LocalLane's differentiator is trust-based ranking.

**Reasoning:** The entire platform premise is "trust over ads." If businesses can buy their way to Partner status, the message is hollow. Tier 2 generates revenue; Tier 3 is the reward for being genuinely trusted.

**Consequences:** Need to define measurable trust criteria; may slow initial revenue (Tier 2 is main revenue source).

**Made by:** Doron (strong preference) + Claude (agreed)

---

### Punch Pass Launches in Demo Mode

**Decision:** The punch pass system will launch without real payment processing. Users will "purchase" with demo currency.

**Context:** Punch passes are a core feature, but real payments require payment processor integration, Terms of Service, refund policies, liability considerations.

**Reasoning:** Demo mode lets us test the full user experience, gather feedback on pricing and flow, launch faster, and defer payment processor integration to Phase 2.

**Consequences:** No revenue from punch passes in Phase 1; need to track demo transactions as if real for future migration.

**Made by:** Doron + Claude

---

### Spec-Repo is Source of Truth

**Decision:** All major decisions, architecture, and standards live in the Spec-Repo. This is the canonical reference.

**Context:** AI assistants don't have persistent memory. Without a single source of truth, decisions get lost and contradicted.

**Reasoning:** Spec-Repo lives in the same GitHub ecosystem as code repos, can be read by Cursor before making changes, and is the "external brain" for AI assistants.

**Consequences:** Must keep Spec-Repo updated; start new conversations by pointing to Spec-Repo.

**Made by:** Doron + Claude

---

### Repositories Are Public (For Now)

**Decision:** The three GitHub repositories (events-node, community-node, Spec-Repo) are public.

**Context:** Claude needs to review code. Options are public access or manual copy-paste.

**Reasoning:** At this stage (pre-revenue, pre-scale), the risk of code theft is low. Real competitive advantage is local relationships, community trust, and execution speed.

**Consequences:** Faster iteration now; can switch to private when there's real traction. Don't commit secrets/API keys.

**Made by:** Doron + Claude

---

### DEC-036: User Groups as Phase 2 Feature

**Date:** 2026-02-03

**Context:** Users need a way to coordinate with people they already know — parents planning around kids' friend groups, neighbors organizing block activities, friend groups scheduling hangouts. This is distinct from public community features.

**Decision:** Implement User Groups as a Phase 2 feature (now Phase 4 in checklist). Groups are fully private, invite-only, with chat, private events, and shared awareness of public event attendance. The group organism pulse tracks real-world gatherings, not message volume.

**Key Design Choices:**
- Fully private (no discoverability, no public groups in V1)
- Private events are the heartbeat (gatherings feed the creature, not chat volume)
- Flat chat only (no threading in V1)
- Admin tools for moderation (suspend group, remove member)
- MyLane placement (groups are part of your lane, not a separate app section)

**Rationale:** Differentiates from generic messaging apps by focusing on real-world coordination. Aligns with Organism Concept — the group creature reflects actual gatherings, not engagement metrics. Builds trust through privacy-first design.

**Consequences:**
- 6 new Base44 entities required
- Admin panel gets Groups section under MANAGEMENT
- Organism pulse infrastructure built before visualization
- Future expansion points architected (business groups, threaded messages, invite links)

**Status:** ✅ Planned — Phase 4 in launch sequence

**Reference:** USER-GROUPS-v1.md (private repo) contains full spec.

---

### DEC-047: Node Lab Model

**Date:** 2026-02-11

**Context:** Nodes were originally designed as sync-dependent children of Community Node (config flows down, events flow up). This created coupling before there was real traffic. Property Pulse and Contractor Daily proved that standalone apps with real users generate better feedback and more mature code than speculative features built inside LocalLane.

**Decision:** Adopt the Node Lab Model as the primary development approach. Nodes are standalone Base44 apps that serve real people independently. Each node goes through: Research → Spec → Build → Field Test → Iterate → Maturity Assessment → Integration Planning. Nodes merge into LocalLane only when both the node and LocalLane are ready. The sync architecture (ARCHITECTURE.md) is preserved as the future integration reference. Portable Feedback Kit (FeedbackLog entity + UI) ships with every node entering field testing.

**Tier 4 (Enterprise) captured as horizon:** Entities that operate their own tree while interoperating with LocalLane. Not a build target — an architectural aim that informs how we design node interoperability.

**Rationale:** Independence forces quality. Real users generate real feedback. Parallel testing with focused building prevents attention fragmentation. Merge from strength, not hope.

**Reference:** NODE-LAB-MODEL.md and NODE-PLAYBOOK.md (private repo)

**Status:** ✅ Active

---

### DEC-048: Research-First Development

**Date:** 2026-02-11

**Context:** Contractor Daily's photo-first philosophy came from CompanyCam research. Community Pass pricing came from Crabtree's Simple Numbers framework. The best design decisions emerged from studying experts and understanding the domain before building.

**Decision:** Codify research as Phase 0 of every node. Before writing a spec: study the domain experts (books, frameworks, competing tools), talk to the real user about their actual workflow, and document findings. The spec emerges from research. Expert frameworks get cited as design principles. This applies to every new node and every major feature.

**Research library maintained in NODE-PLAYBOOK.md** (private repo) tracks which sources informed which nodes.

**Rationale:** Building from imagination produces generic tools. Building from understanding produces tools that feel like they were made by someone who gets it. The research investment pays for itself in fewer iterations.

**Status:** ✅ Active

---

### DEC-049: LocalLane Consolidation Strategy

**Date:** 2026-02-17

**Context:** Standalone nodes (Contractor Daily, Property Pulse, Financial Node) were built as independent Base44 apps following the Node Lab Model (DEC-047). The original plan assumed these could be "integrated" into LocalLane when mature. Research with Base44 support confirmed that Base44 apps are fully isolated — no shared auth, no shared entities, no cross-app data access. Integration would require rebuilding features from specs inside the Community Node.

However, Base44's new Backend Platform (announced Feb 2026) supports a shared backend with multiple frontends — shared auth, shared entities, separate UIs deployed to separate URLs. This is the architecture LocalLane originally envisioned.

**Decision:** Adopt a two-phase consolidation strategy:

- **Phase A (immediate):** Build all new vertical features inside the Community Node with feature gating (role-based, tier-based, or admin-only visibility). Get the platform in front of real users for events and directory. Standalone nodes continue serving their current users unchanged.

- **Phase B (when Base44 Backend Platform is ready):** Migrate to a shared Base44 backend with multiple frontends deployed as subdomains (contractor.locallane.app, property.locallane.app, finance.locallane.app, etc.). Shared auth and entities, separate purpose-built UIs. Proven entity designs from standalone nodes port directly into the shared backend.

**What transfers from standalone nodes:** Entity schemas, data models, role/permission patterns, UX patterns, field-tested workflows. The specs and proven code are the artifacts that carry forward — not the Base44 app containers themselves.

**What does NOT transfer:** Base44 app-level configuration, deployment settings, organization entities. These are recreated in the new architecture.

**Base44 confirmed (2026-02-17):**
- Multiple frontends connect to one backend via shared appId in SDK
- Entity schemas exportable from current apps (Dashboard > Data > Export), push to new backend with `base44 entities push`
- Data migration: CSV export + bulkCreate() — manual but doable
- User accounts cannot be exported (privacy) — users re-register or get invited
- Custom domains fully supported per frontend
- No staging environment yet, on their roadmap
- Docs available: Backend Platform Introduction, Backend-only Quickstart, React Quickstart

**Rationale:** Phase A avoids rebuilding from scratch by using the existing Community Node as the consolidation point. Phase B provides the clean multi-frontend architecture when the tooling supports it. Neither phase wastes work done in the other — entities and components port forward in both directions.

**Consequences:**
- NODE-LAB-MODEL.md updated to reflect consolidation path
- New vertical features built inside Community Node with feature gating
- Standalone nodes remain operational for current users (Dan on Contractor Daily, Doron on Financial Node and Property Pulse)
- Base44 Backend Platform migration planned pending team readiness and user scale

**Status:** Active — Phase A in progress.

---

### DEC-050: Network Events & User Network Membership

**Date:** 2026-02-19

**Context:** Community interest growing via Facebook but no platform-native way to communicate with network-specific audiences. Platform has events and networks but no connection between users and networks.

**Decision:** Three features in sequence: (1) User network interest toggles on MyLane, stored as network_interests array on User entity. (2) Network-only boolean on events with client-side visibility filtering. (3) Network info pages at /networks/:slug with join button, filtered events, and filtered businesses.

**Build Status:**
- Build 1: MyLane My Networks section ✅ (2026-02-19)
- Build 2: Network-only events ⏳
- Build 3: Network info pages ⏳

**Rationale:** Uses events as communication channel instead of messaging. Network membership adds value to platform users. Private events create natural tier differentiation for future Community Pass.

**Status:** ✅ Active — Build 1 complete, Build 2 next

---

### DEC-051: Network Posts

**Date:** 2026-02-21

**Context:** Networks can publish events but have no way to share announcements, updates, or questions with followers. This makes networks feel like event calendars rather than living communities.

**Decision:** Build Network Posts as a phased feature: announcements (Phase 1) → reactions and comments (Phase 2) → polls and organism signals (Phase 3) → polish and notifications (Phase 4). Admin-authored only in V1. Each phase ships a working feature. 5 new entities total: NetworkPost, NetworkPostReaction, NetworkPostComment, NetworkPostPoll, NetworkPostPollVote.

**Key Design Choices:**
- Admin-authored only (V1) — member posts are a future consideration requiring moderation
- Reactions (like/heart/interested) + flat comments — lightweight interaction without messaging complexity
- Polls as post attachments, not standalone — keeps the post as the atomic unit
- Engagement signals documented for Organism integration but not wired yet
- Non-followers can read posts on network pages but cannot react/comment until they follow

**Rationale:** Posts transform networks from event broadcasters into conversation surfaces. The interaction layer differentiates this from "events without a date." Phased delivery means Phase 1 ships quickly and real usage guides depth. Patterns borrowed from User Groups spec (server function auth, entity permissions, expansion points).

**Status:** Draft — build after pilot launch

**Reference:** NETWORK-POSTS.md (private repo)

---

### DEC-052: Pricing Model Reset

**Date:** 2026-02-21

**Context:** Original business tier pricing ($79 Standard, $149 Partner) was set before any market validation. Doron wants more users at lower price points and is rethinking the entire revenue model.

**Decision:** Remove all pricing references from the UI. Three revenue layers under consideration: (1) $9/month platform supporter tier for community believers, (2) ~$49/month Community Pass membership with 75% flowing to businesses via revenue-share pool, (3) Business tiers with pricing TBD. No pricing will be shown until the model is finalized with real user feedback.

**Rationale:** Anchoring early users on speculative pricing creates expectations that may not match the final model. Better to onboard with "founding member" messaging and introduce pricing when there's proof of value.

**Status:** Active — pricing TBD pending real user feedback

---

## Decision Template

```markdown
### DEC-XXX: [Title]

**Date:** YYYY-MM-DD

**Context:** [Why we needed to decide]

**Decision:** [What we chose]

**Rationale:** [Why we chose it]

**Status:** [Active | Superseded | Revisit]
```

---

*Add new decisions at the bottom of this document.*
