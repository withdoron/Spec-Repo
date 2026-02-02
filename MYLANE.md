# MyLane — User Dashboard

> The personal home base for every LocalLane community member.
> "Your community, organized around you."

---

## Status: 📋 SPEC COMPLETE — Ready for Phase 1 Build

Last updated: 2026-02-01

---

## Philosophy

MyLane is the answer to "I just logged in — now what?" Every other page on LocalLane serves *the platform* (Directory, Events, Admin). MyLane serves *the person*. It's where your upcoming events live, where your recommendations are tracked, where your punch pass balance is visible, and where you discover what's new in your community.

MyLane is NOT:
- A repeat of the Directory with different styling
- A feed algorithm optimized for engagement
- A place that hides content behind upgrade walls

MyLane IS:
- Useful on day one, even with zero activity
- Progressively richer as the user engages
- The place where user-tier features naturally surface
- Built on real data, never mock content

From the Blueprint:
> "LocalLane monetizes by being so useful to the community member that they're willing to pay for deeper access to the community itself."

---

## Progressive Depth: Three User States

MyLane adapts based on what the user has *done*, not how long they've been on the platform. No onboarding quiz required. No gates.

### State 1: Explorer (New User — No Activity)

**Who:** Just created an account. No RSVPs, no recommendations, no punch pass.

**What they see:**
- Greeting with their name
- "Happening Soon" — next 5-10 events in their region
- "New in Your Community" — businesses added in last 30 days
- "Browse by Category" — quick-nav tiles to Directory categories
- Punch Pass card (shows "0 balance — learn more" linking to info)
- Gentle nudge: "Start exploring — recommend a business you love"

**What they DON'T see:**
- Empty sections with "nothing here yet" messages
- Upgrade prompts
- Preference quizzes or setup flows

**Design principle:** The page feels useful immediately. A new user sees real community activity without having done anything. Walking into a town square, not filling out a form.

**State 1: Explorer** — Organism state: Seed (small, still, dim — a point of potential)

### State 2: Engaged (Has Activity)

**Who:** Has RSVP'd to events, given Nods or Stories, followed businesses, used Punch Pass.

**What they see (in addition to Explorer content):**
- "Your Upcoming Events" — events they've RSVP'd to, sorted by date
- "Your Recommendations" — businesses they've Nodded or written Stories for
- "From Businesses You Follow" — new events/updates from followed businesses
- Punch Pass card with real balance
- "Happening Soon" now weighted toward categories they've engaged with

**Design principle:** MyLane starts reflecting *you*. It earned the right to personalize because it has real signal — not survey answers.

**State 2: Engaged** — Organism state: Sprouting or Growing (form taking shape, colors developing)

### State 3: Connected (Regular User)

**Who:** Multiple recommendations, regular event attendance, active punch pass usage.

**What they see (in addition to Engaged content):**
- "Your Trust Network" — visual of connections through shared recommendations
- Smart notifications summary ("3 new events match your interests")
- "Outside Your Lane" — deliberate discovery from unfamiliar categories
- BuildLane preference refinement (available as optional tool, not required)
- Deeper community stats (events attended, businesses supported)

**Design principle:** Full personalization. The platform knows you well enough to surface both what you want AND what you didn't know you'd enjoy.

**State 3: Connected** — Organism state: Thriving (vibrant, complex, animated)

---

## Architecture: Section-Based Widget Layout

MyLane uses a **section/widget pattern** — each block of content is a self-contained component. The page assembles sections based on user state and tier.

```
┌─────────────────────────────────────────────┐
│  MyLane                                      │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  GreetingHeader                         │ │
│  │  "Good morning, Doron"                  │ │
│  │  [Punch Pass Balance]  [Settings gear]  │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  UpcomingEventsSection                  │ │
│  │  (only if user has RSVPs)               │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  HappeningSoonSection                   │ │
│  │  Next 5-10 events in region             │ │
│  │  [Filter pills: All, This Weekend,     │ │
│  │   Free, Family, Near Me]               │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  NewInCommunitySection                  │ │
│  │  Businesses added in last 30 days       │ │
│  │  (horizontal scroll)                    │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  YourRecommendationsSection             │ │
│  │  (only if user has Nods/Stories)        │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  DiscoverSection                        │ │
│  │  Category tiles / "Outside Your Lane"   │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ │
│  │  [Future Member Sections]               │ │
│  │  MarketBasketSection                    │ │
│  │  SmartNotificationsSection              │ │
│  │  TrustNetworkSection                    │ │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘ │
└─────────────────────────────────────────────┘
```

### Section Visibility Rules

| Section | Explorer | Engaged | Connected | Member Tier |
|---------|----------|---------|-----------|-------------|
| GreetingHeader | ✅ | ✅ | ✅ | ✅ |
| PunchPassCard | ✅ (0 balance) | ✅ | ✅ | ✅ |
| UpcomingEventsSection | ❌ (no RSVPs) | ✅ | ✅ | ✅ |
| HappeningSoonSection | ✅ | ✅ (weighted) | ✅ (weighted) | ✅ |
| NewInCommunitySection | ✅ | ✅ | ✅ | ✅ |
| YourRecommendationsSection | ❌ (no recs) | ✅ | ✅ | ✅ |
| FollowedBusinessesSection | ❌ (no follows) | ✅ | ✅ | ✅ |
| DiscoverSection | ✅ (categories) | ✅ | ✅ (+ Outside Your Lane) | ✅ |
| MarketBasketSection | ❌ | ❌ | ❌ | 🔮 Future |
| SmartNotificationsSection | ❌ | ❌ | ❌ | 🔮 Future |
| TrustNetworkSection | ❌ | ❌ | ✅ (basic) | 🔮 Enhanced |

### State Detection Logic

```javascript
// Determine user state based on actual activity
const getUserState = (user, rsvps, recommendations, follows) => {
  const hasRSVPs = rsvps.length > 0;
  const hasRecommendations = recommendations.length > 0;
  const hasFollows = follows?.length > 0;
  const hasPunchPass = user.punch_pass_balance > 0;
  
  const activityCount = [hasRSVPs, hasRecommendations, hasFollows, hasPunchPass]
    .filter(Boolean).length;
  
  if (activityCount >= 3) return 'connected';
  if (activityCount >= 1) return 'engaged';
  return 'explorer';
};
```

---

## New Entity Visibility: The "New in Your Community" Shelf

### The Problem

The ranking algorithm (`rankingUtils.jsx`) sorts by trust score. New businesses with zero recommendations always appear last. This is a death sentence for newcomers.

### The Solution

**Dedicated spotlight section** — not mixed into the ranked list.

"New in Your Community" is a horizontal-scroll section on MyLane, Directory, and Events showing entities added in the last 30 days (or until they reach a threshold of 3+ recommendations, whichever comes first).

### Rules

- **Eligibility:** Business `created_date` within last 30 days AND `recommendation_count` < 3
- **Display:** Horizontal scroll of BusinessCards with a "New to LocalLane" badge
- **Graduation:** Once a business hits 3 recommendations, it exits the shelf and joins the trust-ranked main list
- **Ordering within shelf:** Newest first (no trust ranking here — everyone gets equal visibility)
- **Events from new businesses:** Show normally in the events list (events are time-bound so freshness is inherent)

### Where It Appears

| Page | Placement |
|------|-----------|
| MyLane | Between HappeningSoon and YourRecommendations |
| Directory | Above the main business grid |
| Home | Optional — as a "Welcome our newest" section |

### Badge Design

```
┌──────────────────────────────────┐
│  🆕 New to LocalLane             │
│  [Business Name]                 │
│  [Category]                      │
│  Be the first to recommend →     │
└──────────────────────────────────┘
```

The "Be the first to recommend" CTA turns the visibility problem into a community action.

---

## Search and Filtering at Scale

### The Problem

A town with 200 businesses and 50 events per week overwhelms users without strong filtering. Current Events and Directory pages have basic search and category filters — these need to evolve.

### MyLane Filtering (Contextual, Lightweight)

MyLane uses **smart section headers with filter pills**, not a search bar. Filtering here is about narrowing sections, not searching the whole platform.

```
Happening Soon
[All] [This Weekend] [Free] [Family] [Near Me]

┌──────┐ ┌──────┐ ┌──────┐
│Event │ │Event │ │Event │  → horizontal scroll or grid
└──────┘ └──────┘ └──────┘
```

Filter pills for HappeningSoon:
- **All** — everything upcoming
- **This Weekend** — Fri-Sun events
- **Today / Tonight** — same-day events
- **Free** — no ticket cost
- **Family** — age group includes children
- **Near Me** — if location available, within radius

### Events Page Filtering (Power Filtering)

As scale grows, the Events page needs multi-dimensional, combinable filters:

| Filter | Type | Options |
|--------|------|---------|
| Date range | Preset + Custom | This weekend, Next week, Next 30 days, Custom range |
| Category/Type | Multi-select | From admin-configured event types |
| Age group | Multi-select | From admin-configured age groups |
| Time of day | Preset | Morning, Afternoon, Evening |
| Price | Preset | Free, Under $10, Under $25, Under $50, Any |
| Distance | Slider | 5mi, 10mi, 25mi, 50mi (if location available) |

Implementation: Filter pills at top (quick access to common combos), "More Filters" drawer for full power-user filtering. Filters are combinable — "Free family events this weekend within 10 miles."

### Directory Page Filtering (Power Filtering)

| Filter | Type | Options |
|--------|------|---------|
| Category | Single-select | Main categories from categoryData.js |
| Subcategory | Multi-select | Subcategories within selected category |
| Accepts Punch Pass | Toggle | Yes/No |
| Has upcoming events | Toggle | Yes/No |
| Distance | Slider | 5mi, 10mi, 25mi, 50mi |
| Sort | Dropdown | Recommended (trust-ranked), Newest, Alphabetical |

### Saved Filters (Future — Member Feature)

Member-tier users can save filter combinations. "My Saturday search" = Family + Free + This Weekend + Within 10mi. One tap to re-apply.

---

## The "Something New" Factor: Outside Your Lane

### The Problem

If MyLane only shows content matching past behavior, users get trapped in a filter bubble. Someone who attends live music never sees the pottery class.

### The Solution

A deliberate **"Outside Your Lane"** section that surfaces content from categories the user hasn't engaged with.

### Rules

- Only appears for Connected-state users (needs enough activity data to know what's "outside")
- Shows 2-3 items from categories the user has NOT interacted with
- Framed as community discovery, not algorithm: "Your neighbors are also checking out..."
- Rotates on each visit — different suggestions each time
- Never shows categories the user has explicitly dismissed (future: "not interested" option)

### Why Community Framing Matters

"You might also like" → algorithmic, feels like ads
"Your neighbors are also checking out..." → community-driven, feels like word of mouth

This matches LocalLane's core identity. Discovery comes through the community, not through an algorithm.

---

## Dependencies

### Required for Phase 1

| Dependency | Status | Notes |
|------------|--------|-------|
| RSVP data | ⚠️ Needed | Event detail modal needs RSVP button (confirmed in Chrome walkthrough). UpcomingEvents section depends on this. |
| Recommendation data | ✅ Exists | Recommendation entity with Nod/Story types already built |
| Business created_date | ✅ Exists | Used for NewInCommunity filtering |
| Region filtering | ✅ Exists | useActiveRegion hook already implemented |
| Event data | ✅ Exists | Events page queries already work |

### Required for Phase 2

| Dependency | Status | Notes |
|------------|--------|-------|
| Follow/save businesses | ⚠️ Needed | New user action — save business to "My Businesses" |
| User preferences storage | ⚠️ Needed | Store filter preferences, interests on User entity |
| Category engagement tracking | ⚠️ Needed | Track which categories user interacts with |

### Required for Phase 3

| Dependency | Status | Notes |
|------------|--------|-------|
| User tiers | 🔮 Future | See USER-TIERS.md |
| BuildLane integration | ⏳ Shelved | Preference quiz exists but not connected |
| Stripe Connect | 📋 Spec'd | Required for member-tier payments |

---

## File Structure

```
src/
├── pages/
│   └── MyLane.jsx                    # Main page — assembles sections
│
├── components/
│   └── mylane/
│       ├── GreetingHeader.jsx         # Name, punch pass, settings link
│       ├── PunchPassCard.jsx          # Balance display, link to PunchPass page
│       ├── UpcomingEventsSection.jsx   # RSVP'd events (Engaged+)
│       ├── HappeningSoonSection.jsx    # Upcoming events with filter pills
│       ├── NewInCommunitySection.jsx   # Businesses < 30 days, < 3 recs
│       ├── YourRecommendationsSection.jsx  # Nods and Stories given
│       ├── DiscoverSection.jsx        # Category tiles + Outside Your Lane
│       └── SectionWrapper.jsx         # Shared section layout (title, "see all" link)
│
└── hooks/
    └── useUserState.js               # Determines explorer/engaged/connected
```

---

## Phase 1 Build (Now)

Replace the current MyLane.jsx placeholder. Real data, no mocks.

### Components to Build

1. **MyLane.jsx** — Page shell. Fetches user data, determines state, renders sections.
2. **GreetingHeader** — "Good morning, {name}" + Punch Pass balance badge + settings gear icon

> **Organism Placement (DEC-029):** The GreetingHeader is the first home for the personal organism (Phase 1). The organism component sits alongside the greeting and punch badge, providing an ambient visual reflection of the user's community vitality. See ORGANISM-CONCEPT.md (private repo) for the full vision and implementation phases.
3. **HappeningSoonSection** — Queries upcoming events in region, filter pills, grid/scroll
4. **NewInCommunitySection** — Queries businesses where `created_date` > 30 days ago AND `recommendation_count` < 3. Horizontal scroll with "New to LocalLane" badge.
5. **UpcomingEventsSection** — Queries user's RSVP'd events. Conditional: only renders if RSVPs exist. (Depends on RSVP implementation.)
6. **YourRecommendationsSection** — Queries Recommendations where `user_id` = current user. Conditional: only renders if any exist.
7. **DiscoverSection** — Category tiles linking to CategoryPage. Simple grid.
8. **useUserState hook** — Returns 'explorer' | 'engaged' | 'connected' based on activity counts.

### What Phase 1 Skips

- RSVP functionality (UpcomingEventsSection will be conditional — appears once RSVP exists)
- Follow/save businesses
- Preference-based weighting
- Outside Your Lane
- User tier gating
- Saved filters

---

## Phase 2 Build (Soon After)

- Add Follow/Save business action (heart icon on BusinessCard)
- FollowedBusinessesSection on MyLane
- Category engagement tracking (log which categories user views)
- HappeningSoon weighting based on engagement data
- Event page advanced filtering (multi-dimensional)
- RSVP button on event detail → UpcomingEventsSection activates

---

## Phase 3 Build (Scale)

- BuildLane preference integration
- Outside Your Lane section
- Saved filter combinations (member feature)
- Smart notifications section (member feature)
- Trust network visualization (member feature)
- User tier conditional sections

---

## Design Notes

### Colors (Gold Standard)

- Page background: `bg-slate-950`
- Section cards: `bg-slate-900 border border-slate-800`
- Section titles: `text-slate-100 text-xl font-bold`
- "See all" links: `text-amber-500 hover:text-amber-400`
- Filter pills active: `bg-amber-500 text-black`
- Filter pills inactive: `bg-slate-800 text-slate-300 border-slate-700`
- Punch Pass badge: `bg-amber-500/20 text-amber-500 border border-amber-500/30`
- "New to LocalLane" badge: `bg-emerald-500/20 text-emerald-400 border border-emerald-500/30`

### Empty State Philosophy

No section should ever show "Nothing here yet" with a sad face. Instead:

- UpcomingEventsSection: simply doesn't render (Explorer state)
- YourRecommendationsSection: simply doesn't render (Explorer state)
- HappeningSoonSection: if somehow zero events, show "Your community is getting started — check back soon!"
- NewInCommunitySection: if no new businesses, section doesn't render

### Mobile Considerations

- Horizontal scroll sections need touch-friendly snap scrolling
- Filter pills need horizontal scroll on small screens
- Greeting header should stack (name above, badges below) on mobile
- Each section should have clear "See all →" tap targets

---

## Decision Record

This spec establishes the following decisions:

- **MyLane uses section-based widget architecture** — mirroring the business dashboard pattern
- **User state is activity-based, not time-based** — no onboarding gates
- **New entities get dedicated spotlight** — not buried in trust-ranked lists
- **Discovery includes deliberate diversity** — "Outside Your Lane" prevents filter bubbles
- **Architecture supports future user tiers** — sections conditionally render based on tier
- **Organism lives in MyLane GreetingHeader** — Phase 1 placement per DEC-029 and ORGANISM-CONCEPT.md

See also: [USER-TIERS.md](./USER-TIERS.md) for the user tier philosophy and roadmap.

---

*This spec is part of the LocalLane Spec-Repo. Reference before implementing.*
