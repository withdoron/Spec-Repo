# Current Implementation State

> What currently exists in the Community Node codebase.
> Last Updated: 2026-02-01

---

## Overview

The Community Node is the central hub of the LocalLane ecosystem. It serves Tier 1 (Basic) and Tier 2 (Standard) businesses, providing event management, business profiles, a recommendation system, and admin tools. Tier 3 (Partner) businesses operate their own Partner Nodes that sync content up to the Community Node.

**Stack:** React, TanStack Query, Tailwind CSS, shadcn/ui, Base44 backend
**Theme:** Gold Standard dark theme (slate-950 background, amber-500 accent)
**Design System:** See [STYLE-GUIDE.md](./STYLE-GUIDE.md) and [COMPONENTS.md](./COMPONENTS.md)

---

## Pages & Routes

### Public Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/Home` | Home.jsx | ✅ Done | Hero, upcoming events, featured businesses |
| `/Events` | Events.jsx | ✅ Done | Browse/filter events, calendar view |
| `/Directory` | Directory.jsx | ✅ Done | Category grid → business browsing |
| `/CategoryPage` | CategoryPage.jsx | ✅ Done | Subcategory filters, business list |
| `/BusinessProfile` | BusinessProfile.jsx | ✅ Done | Public profile with recommendations |
| `/Search` | Search.jsx | ✅ Done | Full-text search with trust-based ranking |
| `/Recommend` | Recommend.jsx | ✅ Done | Nod/Story submission (replaces old WriteReview) |
| `/PunchPass` | PunchPass.jsx | ✅ Exists | Punch pass management (demo mode) |
| `/MyLane` | MyLane.jsx | ✅ Exists | Personal feed (mock data, placeholder) |

### Business Owner Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/BusinessDashboard` | BusinessDashboard.jsx | ✅ Done | Multi-business selector, archetype-based widgets |
| `/BusinessOnboarding` | BusinessOnboarding.jsx | ✅ Done | 5-step wizard (archetype → details → goals → plan → review) |

### Admin Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/Admin` | Admin.jsx | ✅ Done | Sidebar navigation, routed sections |
| `/Admin/businesses` | AdminBusinessTable | ✅ Done | Business CRUD with cascade delete |
| `/Admin/users` | (placeholder) | ⏳ Planned | User management |
| `/Admin/locations` | AdminLocationsTable | ✅ Done | Location management |
| `/Admin/partners` | SpokeDetails | ✅ Done | Partner Node connections |
| `/Admin/networks` | ConfigSection | ✅ Done | Network config (Recess, TCA, etc.) |
| `/Admin/tiers` | (placeholder) | ⏳ Planned | Tier configuration |
| `/Admin/punch-pass` | (placeholder) | ⏳ Planned | Punch Pass settings |
| `/Admin/settings` | AdminSettingsPanel | ✅ Done | General platform settings |
| `/Admin/events/types` | ConfigSection | ✅ Done | Event type management |
| `/Admin/events/age-groups` | ConfigSection | ✅ Done | Age group management |
| `/Admin/events/durations` | ConfigSection | ✅ Done | Duration preset management |
| `/Admin/events/accessibility` | ConfigSection | ✅ Done | Accessibility features management |

### Other Registered Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/BuildLane` | BuildLane.jsx | ⏳ Shelved | Feed personalization (future feature) |
| `/SpokeDetails` | SpokeDetails.jsx | ✅ Done | Partner Node management (used in Admin) |

---

## Recommendation System (Replaced Reviews)

The old star-rating review system has been fully removed and replaced with a trust-first recommendation system. See [DECISIONS.md DEC-021](./DECISIONS.md) for rationale.

### How It Works

| Type | What It Is | User Action |
|------|-----------|-------------|
| **Nod** | "I recommend this business" | One click, name attached |
| **Story** | Detailed experience narrative | Write about service used, what happened |
| **Vouch** | Weighted trust endorsement | Phase 2 — not yet implemented |

### Recommendation Entity

```javascript
{
  id: UUID,
  business_id: UUID,
  user_id: UUID,
  user_name: string,
  type: 'nod' | 'story',        // 'vouch' in Phase 2
  content: string,               // Story text (stories only)
  service_used: string,          // What they used (stories only)
  photos: string[],              // Photo URLs (stories only)
  is_active: boolean,
  created_date: ISO datetime
}
```

### Business Entity Fields for Recommendations

```javascript
{
  recommendation_count: number,  // Total nods + stories
  nod_count: number,             // Quick nods only
  story_count: number,           // Detailed stories only
}
```

### Recommendation Components

| Component | Location | Purpose |
|-----------|----------|---------|
| TrustSignal | `components/recommendations/TrustSignal.jsx` | Displays nod count + story count badges |
| StoryCard | `components/recommendations/StoryCard.jsx` | Renders a single story with author, content, photos |
| NodAvatars | `components/recommendations/NodAvatars.jsx` | Shows avatar row of people who nodded |

### Ranking Algorithm

Trust-based ranking in `rankingUtils.jsx`:
1. **Primary:** Recommendation count (nods + stories, descending)
2. **Secondary:** Story count (richer engagement, descending)
3. **Tertiary:** Newest first

No paid placement. No tier-based ordering. No boost system.

---

## Event System

### Event Editor

| Feature | Status | Notes |
|---------|--------|-------|
| Basic fields | ✅ Done | Title, description, date, time, location |
| Image upload | ✅ Done | Up to 3 images with hero designation |
| Pricing options | ✅ Done | Free, Single Price, Pay What You Wish |
| Punch Pass | ✅ Done | Toggle + punch cost (1-5), tier-gated |
| Event Type | ✅ Done | Multi-select chips from admin config |
| Networks | ✅ Done | Multi-select chips from admin config |
| Age/Audience | ✅ Done | Dropdown from admin config |
| Duration presets | ✅ Done | Dropdown from admin config |
| Capacity | ✅ Done | Number input |
| Status by tier | ✅ Done | Basic → pending_review, Standard+ → published |
| LockedFeature UI | ✅ Done | Lock icon + upgrade CTA for gated features |
| End time picker | ✅ Done | Duration mode or explicit end time |
| Virtual events | ✅ Done | URL + platform fields |
| Location TBD | ✅ Done | Toggle for undetermined locations |
| Multiple ticket types | ✅ Done | Name, price, limit (tier-gated) |
| Recurring events | ✅ Done | Weekly/biweekly, day selection, end date |
| Accessibility features | ✅ Done | Admin-managed checkboxes |
| Accept RSVPs | ✅ Done | Toggle |
| Additional notes | ✅ Done | Free-text textarea |
| Draft auto-save | ✅ Done | localStorage, 30s interval |
| Save as Draft | ✅ Done | Separate button from Publish |

### Event Entity

```javascript
{
  id: UUID,
  business_id: UUID,
  title: string,
  description: string,
  date: ISO datetime,
  end_date: ISO datetime,
  duration_minutes: number,
  location: string,
  thumbnail_url: string,
  pricing_type: 'free' | 'single_price' | 'pay_what_you_wish',
  price: number,
  is_free: boolean,
  is_pay_what_you_wish: boolean,
  min_price: number,
  punch_pass_accepted: boolean,
  punch_cost: number,           // 1-5
  event_type: string,           // Single value (first from array)
  event_types: string[],        // Multi-select array
  network: string,              // Single value (first from array)
  age_info: string,
  capacity: number,
  status: 'pending_review' | 'published' | 'draft',
  is_active: boolean,
  is_virtual: boolean,
  virtual_url: string,
  virtual_platform: string,
  is_location_tbd: boolean,
  ticket_types: JSON,           // [{ name, price, quantity_limit }]
  is_recurring: boolean,
  recurrence_pattern: string,   // 'weekly' | 'biweekly'
  recurrence_days: string[],    // ['Mon', 'Wed', etc.]
  recurrence_end_date: ISO datetime,
  accessibility_features: string[],
  accepts_rsvps: boolean,
  additional_notes: string,
}
```

---

## Business Dashboard

### Architecture

The dashboard uses archetype-based widget configuration:

```javascript
const DASHBOARD_CONFIG = {
  location/venue: ['Overview', 'Events', 'Staff', 'Financials', 'CheckIn'],
  service/talent: ['Overview', 'Schedule', 'Portfolio', 'Reviews'],
  community:      ['Overview', 'Events', 'Members', 'Donations'],
  organizer:      ['Overview', 'Ticketing', 'Marketing', 'Team'],
};
```

### Dashboard Flow

1. User has no businesses → Shows PersonalDashboard
2. User has businesses, none selected → Shows BusinessCard grid
3. Business selected → Shows archetype-based widget layout

### Dashboard Widgets

| Widget | File | Status | Notes |
|--------|------|--------|-------|
| OverviewWidget | `widgets/OverviewWidget.jsx` | ✅ Done | Recommendations, Stories, Profile Views |
| EventsWidget | `widgets/EventsWidget.jsx` | ✅ Done | Event list + create/edit via EventEditor |
| StaffWidget | `widgets/StaffWidget.jsx` | ✅ Done | Staff management, invites, roles |
| FinancialWidget | `widgets/FinancialWidget.jsx` | ✅ Exists | Basic placeholder |
| BusinessDashboardDetail | `BusinessDashboardDetail.jsx` | ✅ Done | Profile editing, locations, tier upgrade |

### Business Roles

| Role | Badge Color | Can Create Events | Can Edit All Events | Can Manage Staff |
|------|------------|-------------------|---------------------|------------------|
| Owner | Amber | ✅ | ✅ | ✅ |
| Manager | Purple | ✅ | ✅ | ✅ |
| Instructor | Blue | ❌ | Own only | ❌ |
| Staff | Outline | ❌ | ❌ | ❌ |

---

## Staff & Instructor System

| Feature | Status | Notes |
|---------|--------|-------|
| Display staff list | ✅ Done | Shows owner + staff with role badges |
| Add existing user as staff | ✅ Done | Search by email |
| Remove staff | ✅ Done | X button (owner only) |
| Role selection | ✅ Done | Manager, Instructor, Staff |
| Invite non-users | ✅ Done | Email invite stored in AdminSettings |
| Pending invites display | ✅ Done | Separate section with "Pending" badge |
| Auto-link on dashboard load | ✅ Done | Pending invites matched on visit |
| 403 permission handling | ✅ Done | Staff see "Team Member" placeholder |
| Custom permissions per person | ⏳ Planned | Override default role permissions |

### Staff Data Storage

```javascript
// Active staff: business.instructors = ['user_id_1', 'user_id_2']
// Roles: AdminSettings key 'staff_roles:{business_id}'
// Invites: AdminSettings key 'staff_invites:{business_id}'
```

---

## Configuration System

Config is stored in AdminSettings with key convention:

```
platform_config:{domain}:{config_type}
```

### Current Config Keys

| Key | Content |
|-----|---------|
| `platform_config:events:event_types` | Event type options |
| `platform_config:events:age_groups` | Age/audience options |
| `platform_config:events:duration_presets` | Duration dropdown options |
| `platform_config:events:accessibility_features` | Accessibility checkbox options |
| `platform_config:platform:networks` | Network options (Recess, TCA, etc.) |

### Hooks

| Hook | File | Purpose |
|------|------|---------|
| `useConfig` | `hooks/useConfig.js` | Read config from AdminSettings with defaults |
| `useConfigMutation` | `hooks/useConfig.js` | Write config to AdminSettings |
| `useOrganization` | `hooks/useOrganization.js` | Tier checking (tierLevel, canUsePunchPass, etc.) |

---

## Business Entity

```javascript
{
  id: UUID,
  name: string,
  description: string,
  owner_user_id: UUID,
  owner_email: string,
  subscription_tier: 'basic' | 'standard' | 'partner',
  archetype: string,
  main_category: string,
  subcategories: string[],
  city: string,
  state: string,
  status: 'active' | 'inactive',
  instructors: UUID[],          // Staff user IDs
  recommendation_count: number,
  nod_count: number,
  story_count: number,
  photos: string[],
  services: JSON[],             // [{ name, starting_price, description }]
  accepts_silver: boolean,
}
```

---

## Dark Theme Status

All pages and components now use the Gold Standard dark theme. No light-theme holdovers remain.

### Theme Completed

- ✅ Home, Events, Directory, CategoryPage, Search
- ✅ BusinessProfile, BusinessOnboarding, Recommend
- ✅ BusinessDashboard, BusinessDashboardDetail, all widgets
- ✅ OverviewWidget (rewritten from light theme)
- ✅ Admin Panel (sidebar, tables, drawers, config sections)
- ✅ Header/Navigation (dropdown menus, mobile menu)
- ✅ All tier cards and icon backgrounds use `bg-amber-500/20`, `bg-blue-500/20`, `bg-slate-800`

### CSS Variables

Global CSS variables in `index.css` are set for the dark theme. Radix UI tab hover states are overridden globally to prevent white flash.

---

## Dead Code Removed

The following were removed during the Jan 31 – Feb 1 cleanup:

| File | Reason |
|------|--------|
| `StarRating.jsx` | Replaced by TrustSignal |
| `ReviewCard.jsx` | Replaced by StoryCard |
| `OverviewWidget.tsx.jsx` | Duplicate of OverviewWidget.jsx |
| `MigrateCategories.jsx` | One-time migration tool, job done |
| `migrateCategoriesScript.jsx` | Helper for above |
| `cleanupDuplicates.jsx` | Helper for above |
| `resetCategories.jsx` | Helper for above |
| `Categories.jsx` | Replaced by CategoryPage |
| `BusinessSelector.jsx` | Never imported (dashboard uses inline grid) |
| `BusinessSelector.tsx.jsx` | Duplicate of above |
| `src/components/reviews/` folder | Empty after deletions |

Also removed: All Review.filter queries from BusinessProfile and BusinessDashboardDetail. Boost system completely removed (see DEC-022).

---

## File Structure

```
community-node/src/
├── pages/
│   ├── Admin.jsx
│   ├── BusinessDashboard.jsx
│   ├── BusinessOnboarding.jsx
│   ├── BusinessProfile.jsx
│   ├── BuildLane.jsx               # Shelved
│   ├── CategoryPage.jsx
│   ├── Directory.jsx
│   ├── Events.jsx
│   ├── Home.jsx
│   ├── MyLane.jsx                  # Placeholder
│   ├── PunchPass.jsx
│   ├── Recommend.jsx
│   └── Search.jsx
│
├── components/
│   ├── admin/
│   │   ├── AdminSidebar.jsx
│   │   ├── AdminLayout.jsx
│   │   ├── AdminBusinessTable.jsx
│   │   ├── AdminFilters.jsx
│   │   ├── AdminLocationsTable.jsx
│   │   ├── AdminSettingsPanel.jsx
│   │   ├── BusinessEditDrawer.jsx
│   │   └── config/ConfigSection.jsx
│   │
│   ├── business/
│   │   ├── BusinessCard.jsx
│   │   └── rankingUtils.jsx
│   │
│   ├── categories/
│   │   └── categoryData.js
│   │
│   ├── dashboard/
│   │   ├── BusinessCard.jsx
│   │   ├── BusinessDashboardDetail.jsx
│   │   ├── EventEditor.jsx
│   │   ├── LocationsSection.jsx
│   │   └── widgets/
│   │       ├── EventsWidget.jsx
│   │       ├── OverviewWidget.jsx
│   │       ├── FinancialWidget.jsx
│   │       └── StaffWidget.jsx
│   │
│   ├── locations/
│   │   └── formatAddress.js
│   │
│   ├── recommendations/
│   │   ├── TrustSignal.jsx
│   │   ├── StoryCard.jsx
│   │   └── NodAvatars.jsx
│   │
│   ├── search/
│   │   └── SearchResultsSection.jsx
│   │
│   └── ui/
│       ├── LockedFeature.jsx
│       ├── SyncStatusBadge.jsx
│       └── ... (shadcn components)
│
├── hooks/
│   ├── useConfig.js
│   └── useOrganization.js
│
└── utils/
    └── defaultConfig.js
```

---

## Known Issues

| Issue | Priority | Notes |
|-------|----------|-------|
| Admin search filters business list behind drawer | Medium | Staff search input bleeds through |
| Soft delete not working | Low | Hard delete works; Base44 schema limitation |
| Calendar picker light theme | Low | Date picker needs dark override |
| Dialog X button hard to see | Low | Dark on dark |

---

## Remaining Work

### High Priority — Features

| Task | Notes |
|------|-------|
| Concern system (Phase 1.5) | ConcernForm, private routing to steward, admin view |
| Vouch system (Phase 2) | Trust weighting, business owner responses |

### Medium Priority — Platform

| Task | Notes |
|------|-------|
| Platform Settings (Tiers config) | Configure tier names, prices, features in Admin |
| Punch Pass Settings | Platform fee %, price per punch in Admin |
| Onboarding Configuration | Archetypes, goals, feature matrix in Admin |
| User Management in Admin | View/manage users |
| Staff auto-link completion | Full invite flow with email notifications |
| Custom permissions per person | Override default role permissions |

### Lower Priority — Post-Pilot

| Task | Notes |
|------|-------|
| BuildLane feed personalization | Shelved — concept exists |
| Event Editor as modal | Currently inline in widget |
| Recurring events refinement | Edge cases, cancellation handling |
| Venue Configuration | When venues archetype activated |
| Permission enforcement | Check permissions throughout app |

---

## Tier System

| Tier | Code Value | Display | Features |
|------|------------|---------|----------|
| 1 | `basic` | Basic | Events (reviewed), basic listing |
| 2 | `standard` | Standard | Auto-publish, Punch Pass, analytics |
| 3 | `partner` | Partner | Own Partner Node, full autonomy |

See [TIER-SYSTEM.md](./TIER-SYSTEM.md) for complete documentation.

---

*This document reflects the actual implementation state as of 2026-02-01.*
