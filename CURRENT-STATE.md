# Current Implementation State

> What currently exists in the LocalLane ecosystem.
> Last Updated: 2026-01-30 (End of Session)

---

## Summary

The Community Node is functional with:
- ✅ Event browsing and display
- ✅ Business Dashboard with Event Editor
- ✅ Admin Panel with sidebar navigation
- ✅ Admin-managed configuration (Event Types, Networks, Age Groups, Durations)
- ✅ Tier-based feature gating
- ✅ Three-tier system (Basic/Standard/Partner)

---

## Community Node

**URL:** locallane.app (or Base44 preview)
**Repo:** github.com/withdoron/community-node

### Architecture Overview

```
Community Node
├── PUBLIC PAGES
│   ├── Home/Landing
│   ├── Browse Events
│   ├── Event Detail
│   └── Browse Directory
│
├── USER PAGES (Logged In)
│   ├── My Lane (personal dashboard)
│   ├── Dashboard (user dashboard)
│   └── Saved Events
│
├── BUSINESS DASHBOARD
│   ├── Business Selector (if multiple businesses)
│   ├── Overview Widget
│   ├── Events Widget + Event Editor
│   ├── Staff Widget
│   ├── Financial Widget
│   └── Settings (tier display)
│
└── ADMIN PANEL (Sidebar Navigation)
    ├── MANAGEMENT
    │   ├── Businesses ✅
    │   ├── Users (placeholder)
    │   ├── Locations ✅
    │   └── Partners ✅
    ├── PLATFORM
    │   ├── Networks ✅
    │   ├── Tiers (placeholder)
    │   ├── Punch Pass (placeholder)
    │   └── Settings ✅
    ├── EVENTS
    │   ├── Event Types ✅
    │   ├── Age Groups ✅
    │   └── Durations ✅
    └── ONBOARDING
        ├── Business (placeholder)
        └── User (placeholder)
```

---

## Completed Features

### Admin Panel (Restructured)

| Feature | Status | Notes |
|---------|--------|-------|
| Sidebar navigation | ✅ Done | Organized by domain (Management, Platform, Events, Onboarding) |
| Businesses management | ✅ Done | List, search, filter, edit, delete, tier management |
| Locations management | ✅ Done | Geographic areas |
| Partners (Spokes) | ✅ Done | Partner Node management |
| Event Types config | ✅ Done | Add/Edit/Delete, persists to AdminSettings |
| Age Groups config | ✅ Done | Add/Edit/Delete |
| Durations config | ✅ Done | Add/Edit/Delete |
| Networks config | ✅ Done | Add/Edit/Delete |
| Settings panel | ✅ Done | General settings |

### Business Dashboard

| Feature | Status | Notes |
|---------|--------|-------|
| Business selector | ✅ Done | Shows businesses owned by user |
| Owner linking | ✅ Done | Links businesses by owner_user_id |
| Event Editor | ✅ Done | Full form with all fields |
| Tier gating | ✅ Done | Punch Pass locked for basic tier |
| Dynamic config | ✅ Done | Loads Event Types, Networks, etc. from Admin config |
| Delete business | ✅ Done | Hard delete with confirmation |

### Event Editor

| Feature | Status | Notes |
|---------|--------|-------|
| Basic fields | ✅ Done | Title, description, date, time, location |
| Image upload | ✅ Done | Single image via Base44 UploadFile |
| Pricing options | ✅ Done | Free, Single Price, Pay What You Wish |
| Punch Pass | ✅ Done | Toggle + punch cost (1-5), tier-gated |
| Event Type | ✅ Done | Dropdown from admin config |
| Networks | ✅ Done | Multi-select chips from admin config |
| Age/Audience | ✅ Done | Dropdown from admin config |
| Duration presets | ✅ Done | Dropdown from admin config |
| Capacity | ✅ Done | Number input |
| Status by tier | ✅ Done | Basic → pending_review, Standard+ → published |
| LockedFeature UI | ✅ Done | Lock icon + upgrade CTA for gated features |
| Modal display | ✅ Done | Opens as dialog overlay |

### Configuration System

| Feature | Status | Notes |
|---------|--------|-------|
| useConfig hook | ✅ Done | Reads from AdminSettings with fallback to defaults |
| useConfigMutation | ✅ Done | Writes to AdminSettings |
| Default config | ✅ Done | Fallback values in defaultConfig.js |
| Admin UI (ConfigSection) | ✅ Done | Reusable list with Add/Edit/Delete |

### Staff & Instructor System

| Feature | Status | Notes |
|---------|--------|-------|
| Display staff list | ✅ Done | Shows owner + staff with role badges |
| Add existing user as staff | ✅ Done | Search by email, add to business.instructors |
| Remove staff | ✅ Done | X button removes from instructors array |
| Role selection | ✅ Done | Manager (purple), Instructor (blue), Staff (outline) |
| Invite non-users | ✅ Done | Stored in AdminSettings as `staff_invites:{business_id}` |
| Pending invites display | ✅ Done | Shows in separate section with "Pending" badge |
| Remove pending invites | ✅ Done | X button removes from AdminSettings |
| Admin staff management | ✅ Done | Admins can manage staff for any business |
| Staff sees businesses | ✅ Done | Staff members see businesses with "TEAM" badge |
| Auto-link on dashboard load | ✅ Done | When user visits dashboard, pending invites are auto-linked |
| 403 permission handling | ✅ Done | Staff see "Team Member" placeholder for other users |
| Role storage (AdminSettings) | ✅ Done | `staff_roles:{business_id}` stores role per user |
| Permission-based UI | ✅ Done | Only owners see Add Staff button and remove buttons |
| Custom permissions per person | ⏳ Planned | Override default role permissions |
| Permission checking in app | ⏳ Planned | Enforce permissions throughout app |

### Business Entity

```javascript
{
  id: UUID,
  name: string,
  description: string,
  owner_user_id: UUID,        // Links to User
  owner_email: string,
  subscription_tier: 'basic' | 'standard' | 'partner',
  archetype: string,
  city: string,
  status: 'active' | 'inactive',
  is_deleted: boolean,        // Soft delete (not fully working)
  // ... other fields
}
```

### Event Entity

```javascript
{
  id: UUID,
  business_id: UUID,
  title: string,
  description: string,
  date: ISO datetime,         // Start date/time
  end_date: ISO datetime,
  duration_minutes: number,
  location: string,
  thumbnail_url: string,      // Hero image
  pricing_type: 'free' | 'single_price' | 'pay_what_you_wish',
  price: number,
  is_free: boolean,
  is_pay_what_you_wish: boolean,
  min_price: number,
  punch_pass_accepted: boolean,
  punch_cost: number,         // 1-5
  event_type: string,         // Single value
  network: string,            // Single value (first from array)
  age_info: string,
  capacity: number,
  status: 'pending_review' | 'published' | 'draft',
  is_active: boolean,
}
```

### AdminSettings (Config & Invites Storage)

```javascript
// Platform Config
{
  id: UUID,
  key: string,    // e.g., 'platform_config:events:event_types'
  value: string,  // JSON string: { items: [...], updated_at: Date }
}

// Staff Invites (per business)
{
  id: UUID,
  key: string,    // e.g., 'staff_invites:{business_id}'
  value: string,  // JSON array: [{ email, role, status, invited_at }, ...]
}
```

### Staff Data Structure

```javascript
// Active staff stored in Business.instructors (array of user IDs)
business.instructors = ['user_id_1', 'user_id_2', ...]

// Pending invites stored in AdminSettings
// Key: staff_invites:{business_id}
// Value: JSON array
[
  {
    email: 'invited@example.com',
    role: 'instructor',      // 'manager' | 'instructor' | 'staff'
    status: 'invited',
    invited_at: '2026-01-30T...',
  },
  // ...
]

// Role badge colors
// Manager: purple (bg-purple-500)
// Instructor: blue (bg-blue-500)
// Staff: outline (border-slate-500)
```

---

## File Structure

```
community-node/src/
├── pages/
│   ├── Admin.jsx                    # Admin panel with routes
│   ├── BusinessDashboard.jsx        # Business management
│   ├── Events.jsx                   # Public event browsing
│   └── MyLane.jsx                   # User dashboard
│
├── components/
│   ├── admin/
│   │   ├── AdminSidebar.jsx         # Sidebar navigation
│   │   ├── AdminLayout.jsx          # Layout wrapper
│   │   ├── config/
│   │   │   └── ConfigSection.jsx    # Reusable config list
│   │   ├── AdminBusinessTable.jsx
│   │   ├── AdminFilters.jsx
│   │   ├── AdminLocationsTable.jsx
│   │   ├── AdminSettingsPanel.jsx
│   │   └── BusinessEditDrawer.jsx
│   │
│   ├── dashboard/
│   │   ├── EventEditor.jsx          # Event creation/edit form
│   │   ├── widgets/
│   │   │   ├── EventsWidget.jsx
│   │   │   ├── OverviewWidget.jsx
│   │   │   ├── FinancialWidget.jsx
│   │   │   └── StaffWidget.jsx
│   │   └── BusinessCard.jsx
│   │
│   └── ui/
│       ├── LockedFeature.jsx        # Tier gating UI
│       └── ... (shadcn components)
│
├── hooks/
│   ├── useConfig.js                 # Config read/write hooks
│   └── useOrganization.js          # Tier checking hook
│
└── utils/
    └── defaultConfig.js             # Default config values
```

---

## Known Issues & Future Work

### Issues to Address

| Issue | Priority | Notes |
|-------|----------|-------|
| Admin search filters business list | Medium | Typing in staff search input filters business list behind drawer |
| Soft delete not working | Low | Hard delete works; Base44 schema may not have is_deleted field |
| Business.staff field doesn't exist | Low | Using AdminSettings for roles/invites as workaround |
| Calendar picker light theme | Low | Needs dark theme styling |
| Dialog X button hard to see | Low | Dark on dark |
| Some white flash on hover | Low | Minor style issues |

### Staff System Complete ✅

The full staff management system is now functional:
- Owners can add/remove staff with role selection
- Non-users can be invited by email
- Invites auto-link when user creates account and visits dashboard
- Staff see businesses they're part of with "TEAM" badge
- Permission-based UI (staff can't modify team, only view)
- 403 errors handled gracefully (Team Member placeholder)

### Phase 2 (Upcoming)

| Task | Priority | Notes |
|------|----------|-------|
| Recurring Events | High | "This event repeats" pattern |
| Multiple Tickets | High | Ticket types with different prices |
| Event End Time Picker | Medium | Currently duration-only |
| Custom permissions per person | Medium | Override default role permissions |
| Permission checking in app | Medium | Enforce permissions throughout app |
| Platform Settings (Tiers config) | Medium | Configure tier names, prices, features |
| Punch Pass Settings | Medium | Platform fee %, price per punch |
| Onboarding Configuration | Medium | Archetypes, goals, feature matrix |

### Post-Feature Work

| Task | Priority | Notes |
|------|----------|-------|
| Permission Gating | Medium | Who can create/edit events, manage staff, etc. |
| Tier Gating | Medium | Which features unlock at each tier |
| Venue Configuration | Low | When venues archetype activated |
| Polish Pass | Low | Fix all style issues |

---

## Tier System

| Tier | Code Value | Display | Features |
|------|------------|---------|----------|
| 1 | `basic` | Basic | Events (reviewed), basic listing |
| 2 | `standard` | Standard | Auto-publish, Punch Pass, analytics |
| 3 | `partner` | Partner | Own Partner Node, full autonomy |

See [TIER-SYSTEM.md](./TIER-SYSTEM.md) for complete documentation.

---

## Configuration Keys

Config is stored in AdminSettings with this key convention:

```
platform_config:{domain}:{config_type}

Current keys:
- platform_config:events:event_types
- platform_config:events:age_groups
- platform_config:events:duration_presets
- platform_config:platform:networks
```

See [CONFIG-SYSTEM.md](./CONFIG-SYSTEM.md) for complete documentation.

---

## Admin Panel Routes

| Route | Content |
|-------|---------|
| `/Admin` | Redirects to /Admin/businesses |
| `/Admin/businesses` | Business management |
| `/Admin/users` | User management (placeholder) |
| `/Admin/locations` | Location management |
| `/Admin/partners` | Partner Node management |
| `/Admin/networks` | Networks config |
| `/Admin/tiers` | Tiers config (placeholder) |
| `/Admin/punch-pass` | Punch Pass config (placeholder) |
| `/Admin/settings` | General settings |
| `/Admin/events/types` | Event Types config |
| `/Admin/events/age-groups` | Age Groups config |
| `/Admin/events/durations` | Durations config |
| `/Admin/onboarding/business` | Business onboarding config (placeholder) |
| `/Admin/onboarding/user` | User onboarding config (placeholder) |

---

*This document reflects the actual implementation state as of 2026-01-30.*
