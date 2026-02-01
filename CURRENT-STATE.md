# Current State

> Reflects actual implementation state as of 2026-02-01.

---

## Platform Overview

**Community Node:** locallane.app (the hub)
**Event Node:** Partner Node template for event organizers
**Stack:** React, Vite, Tailwind CSS, TanStack Query, shadcn/ui, Base44 backend
**Design:** Gold Standard dark theme (slate-950 bg, amber-500 accent)
**Stripe:** SDKs installed (`@stripe/react-stripe-js`, `@stripe/stripe-js`), no integration wired up yet

---

## Pages & Routes

### Main Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/` | Home.jsx | ✅ Done | Landing page |
| `/Events` | Events.jsx | ✅ Done | Browse events with filters |
| `/Directory` | Directory.jsx | ✅ Done | Business listings with search |
| `/Search` | Search.jsx | ✅ Done | Full-text search with sort options |
| `/BusinessProfile/:id` | BusinessProfile.jsx | ✅ Done | Public business page |
| `/CategoryPage/:slug` | CategoryPage.jsx | ✅ Done | Category browse page |
| `/PunchPass` | PunchPass.jsx | ✅ Done | Punch Pass balance and management (demo mode) |
| `/CheckIn` | CheckIn.jsx | ✅ Done | Event check-in with PIN validation |
| `/Recommend/:id` | Recommend.jsx | ✅ Done | Nod/Story submission form |
| `/MyLane` | MyLane.jsx | ⏳ Placeholder | User dashboard — not yet built |

### Business Dashboard

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/business-dashboard` | BusinessDashboard.jsx | ✅ Done | Multi-business selector + widget layout |
| (selected business) | BusinessDashboardDetail.jsx | ✅ Done | Profile editing, locations, tier upgrade |

### Admin Panel

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/Admin` | Admin.jsx | ✅ Done | Steward admin panel |
| `/Admin/settings` | AdminSettingsPanel | ✅ Done | General platform settings |
| `/Admin/events/types` | ConfigSection | ✅ Done | Event type management |
| `/Admin/events/age-groups` | ConfigSection | ✅ Done | Age group management |
| `/Admin/events/durations` | ConfigSection | ✅ Done | Duration preset management |
| `/Admin/events/accessibility` | ConfigSection | ✅ Done | Accessibility features management |
| `/Admin/tiers` | (placeholder) | ⏳ Planned | Tier configuration |
| `/Admin/punch-pass` | (placeholder) | ⏳ Planned | Punch Pass settings |

### Other Registered Pages

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/BuildLane` | BuildLane.jsx | ⏳ Shelved | Feed personalization (future feature) |
| `/SpokeDetails` | SpokeDetails.jsx | ✅ Done | Partner Node management (used in Admin) |

---

## Dashboard Widgets

Dashboard flow:
1. User has no businesses → Shows PersonalDashboard
2. User has businesses, none selected → Shows BusinessCard grid
3. Business selected → Shows archetype-based widget layout

| Widget | File | Status | Notes |
|--------|------|--------|-------|
| OverviewWidget | `widgets/OverviewWidget.jsx` | ✅ Done | Recommendations, Stories, Profile Views |
| EventsWidget | `widgets/EventsWidget.jsx` | ✅ Done | Event list + create/edit/duplicate/cancel/delete via EventEditor |
| StaffWidget | `widgets/StaffWidget.jsx` | ✅ Done | Staff management UI, invites, role assignment |
| FinancialWidget | `widgets/FinancialWidget.jsx` | ✅ Exists | Basic placeholder |
| BusinessDashboardDetail | `BusinessDashboardDetail.jsx` | ✅ Done | Profile editing, locations, tier upgrade |

---

## Recommendation System (Replaced Reviews)

The old star-rating review system has been fully removed and replaced with a trust-first recommendation system. See [DECISIONS.md DEC-021](./DECISIONS.md) for rationale.

### How It Works

| Type | What It Is | User Action | Trust Weight |
|------|-----------|-------------|-------------|
| **Nod** | "I recommend this business" | One click, name attached | 1x |
| **Story** | Detailed experience narrative | Write about service used, what happened | 2x |
| **Vouch** | Verified trust endorsement | Must have existing Nod or Story | 3x |

### Business Entity Fields for Recommendations

```javascript
{
  recommendation_count: number,  // Total nods + stories + vouches
  nod_count: number,
  story_count: number,
  vouch_count: number,
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
1. **Primary:** Weighted recommendation score (nod x1 + story x2 + vouch x3, descending)
2. **Secondary:** Story count (richer engagement, descending)
3. **Tertiary:** Newest first

No paid placement. No tier-based ordering. No boost system.

### Private Concern System

Negative feedback is handled privately through the Concern entity:
- Any authenticated user can submit a concern
- Concerns are routed to platform stewards (admin-only read)
- Not displayed publicly — no public negative reviews
- Admin can update status and add notes

---

## Staff & Instructor System

### What's Built (UI & Data Layer)

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

### What's NOT Built (Permission Enforcement)

| Feature | Status | Notes |
|---------|--------|-------|
| Permission enforcement in app | ❌ Not started | Roles are labels only — not enforced in EventsWidget, etc. |
| Custom permissions per person | ❌ Not started | Override default role permissions |
| Instructor can edit assigned events only | ❌ Not started | Currently no "assigned event" concept |
| Staff member restricted view | ❌ Not started | Staff see same widgets as owners |
| Role-based widget visibility | ❌ Not started | All widgets visible regardless of role |

### Role Permission Matrix (Designed, NOT Enforced)

| Permission | Owner | Manager | Instructor | Staff |
|------------|-------|---------|------------|-------|
| can_create_events | ✅ | ✅ | ❌ | ❌ |
| can_edit_all_events | ✅ | ✅ | ❌ | ❌ |
| can_edit_assigned_events | ✅ | ✅ | ✅ | ❌ |
| can_manage_staff | ✅ | ✅ | ❌ | ❌ |
| can_view_analytics | ✅ | ✅ | Limited | ❌ |
| can_use_checkin | ✅ | ✅ | ✅ | ✅ |

**Current reality:** EventsWidget has a loose `allowEdit` check and the action dropdown checks for `['owner', 'manager']`, but this is a UI-only check, not systematic enforcement. StaffWidget shows "Add Staff" button only for owner/manager. No other permission checks exist.

### Business Roles (Visual)

| Role | Badge Color |
|------|------------|
| Owner | Amber (`bg-amber-500 text-black`) |
| Manager | Purple (`bg-purple-500 text-white`) |
| Instructor | Blue (`bg-blue-500 text-white`) |
| Staff | Outline (`border-slate-500 text-slate-300`) |
| Pending | Amber outline (`border-amber-500 text-amber-500`) |

### Staff Data Storage

```javascript
// Active staff: business.instructors = ['user_id_1', 'user_id_2']
// Roles: AdminSettings key 'staff_roles:{business_id}'
// Invites: AdminSettings key 'staff_invites:{business_id}'
```

---

## Event Management

| Feature | Status | Notes |
|---------|--------|-------|
| Create event | ✅ Done | Full form with all fields via EventEditor |
| Edit event | ✅ Done | Opens EventEditor with existing data |
| Delete event | ✅ Done | Soft delete (is_active: false) with confirmation |
| Cancel event | ✅ Done | Sets status: 'cancelled' with confirmation |
| Duplicate event | ✅ Done | Creates copy with "(Copy)" suffix |
| Recurring events | ✅ Done | Weekly/biweekly with day-of-week selection |
| Event images | ✅ Done | Image upload in EventEditor |
| Punch Pass pricing | ✅ Done | Toggle + punch cost (demo mode) |
| Multi-ticket pricing | ✅ Done | Tiered ticket types |
| Accessibility features | ✅ Done | Checkbox list from admin config |
| Event list with actions | ✅ Done | Desktop table + mobile cards |

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

## Security — Entity Permissions

### Completed (Phase 1 & 2) — 11 of 18 Entities Locked

| Entity | Read | Create | Update | Delete |
|--------|------|--------|--------|--------|
| AdminAuditLog | Admin | Admin | Admin | Disabled |
| Concern | Admin | Authenticated | Admin | Admin |
| PunchPass | Owner | Admin | Admin | Admin |
| PunchPassTransaction | Owner | Admin | Admin | Admin |
| PunchPassUsage | Owner | Admin | Admin | Admin |
| Region | Public | Admin | Admin | Admin |
| Archetype | Public | Admin | Admin | Admin |
| CategoryGroup | Public | Admin | Admin | Admin |
| SubCategory | Public | Admin | Admin | Admin |

### Deleted Entities
- Review (replaced by Recommendation — DEC-021)
- Bump (boost system removed — DEC-022)

### Phase 3 — Deferred (Requires Service Role Functions)

These entities still have overly permissive settings and need code changes before lockdown:

Business, Event, AdminSettings, RSVP, Location, User, Recommendation, Spoke, SpokeEvent, CategoryClick

See DEC-025 for full context.

---

## Stripe Integration Status

| Component | Status | Notes |
|-----------|--------|-------|
| Frontend SDKs | ✅ Installed | `@stripe/react-stripe-js` v3.10, `@stripe/stripe-js` v5.10 |
| Stripe account | ⚠️ Needs setup | Current connected account is not LocalLane — need dedicated account |
| Products/Prices | ❌ None | No products, customers, or subscriptions created |
| Stripe Connect | ❌ Not started | Needed for business payouts (Punch Pass revenue) |
| Checkout flow | ❌ Not started | Punch Pass purchase is demo mode only |
| Webhook handling | ❌ Not started | No server-side Stripe event processing |
| Subscription billing | ❌ Not started | Business tier subscriptions not wired up |

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
│   ├── CheckIn.jsx
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
│   ├── events/
│   │   ├── EventFormV2.jsx
│   │   └── EventTable.jsx
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

### High Priority — Revenue & Safety

| Task | Notes |
|------|-------|
| Stripe Connect integration | Real payments for Punch Pass, business tier subscriptions |
| Concern system UI | ConcernForm component, admin review panel |
| Permission enforcement | Make staff roles actually control what users see and do |

### Medium Priority — User Experience

| Task | Notes |
|------|-------|
| MyLane user dashboard | User RSVPs, recommendations, punch passes, saved events |
| Microbusiness archetype | Onboarding + dashboard for solo operators (DEC-027) |
| Platform Settings (Tiers config) | Configure tier names, prices, features in Admin |
| Punch Pass Settings | Platform fee %, price per punch in Admin |

### Lower Priority — Polish & Scale

| Task | Notes |
|------|-------|
| Security Phase 3 | Service role functions for remaining 10 entities |
| Onboarding configuration | Archetypes, goals, feature matrix in Admin |
| User management in Admin | View/manage users |
| Staff auto-link completion | Full invite flow with email notifications |
| BuildLane feed personalization | Shelved — concept exists |
| Recurring events refinement | Edge cases, cancellation handling |

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
