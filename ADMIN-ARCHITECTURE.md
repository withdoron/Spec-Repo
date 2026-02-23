# Admin Panel Architecture

> Defines the structure, navigation, and organization of the LocalLane Admin Panel.
> Updated: 2026-01-29

---

## Overview

The Admin Panel is the control center for platform Stewards to manage businesses, users, configuration, and platform settings. As LocalLane expands to support multiple archetypes (Events, Venues, Services, etc.), the Admin Panel must scale without becoming unwieldy.

**Design Principles:**
1. **Organized by domain** — Not one giant settings page
2. **Intuitive navigation** — Sidebar for complex hierarchy
3. **Scalable** — Easy to add new archetype configurations
4. **Configurable over hardcoded** — Stewards can modify options without code changes

---

## Navigation Structure

### Sidebar Navigation (Recommended)

```
┌─────────────────────────────────────────────────────────────────┐
│  LocalLane Admin                                    [Back to Site]
├─────────────────┬───────────────────────────────────────────────┤
│                 │                                               │
│  MANAGEMENT     │   [Content Area]                              │
│  ├─ Businesses  │                                               │
│  ├─ Users       │   Shows the selected section's content        │
│  ├─ Locations   │                                               │
│  └─ Partners    │                                               │
│                 │                                               │
│  PLATFORM       │                                               │
│  ├─ Networks    │                                               │
│  ├─ Tiers       │                                               │
│  ├─ Joy Coins    │                                               │
│  └─ Branding    │                                               │
│                 │                                               │
│  EVENTS         │                                               │
│  ├─ Event Types │                                               │
│  ├─ Age Groups  │                                               │
│  └─ Durations   │                                               │
│                 │                                               │
│  VENUES         │                                               │
│  ├─ Venue Types │                                               │
│  ├─ Amenities   │                                               │
│  └─ Capacities  │                                               │
│                 │                                               │
│  ONBOARDING     │                                               │
│  ├─ Business    │                                               │
│  └─ User        │                                               │
│                 │                                               │
└─────────────────┴───────────────────────────────────────────────┘
```

---

## Section Breakdown

### MANAGEMENT

Existing functionality for managing entities.

| Section | Purpose | Status |
|---------|---------|--------|
| **Businesses** | List, search, filter, edit businesses. Tier management, status, badges. | ✅ Exists |
| **Users** | List, search users. Membership tier, status, preferences. | ⏳ New |
| **Locations** | Geographic areas/cities the platform serves. | ✅ Exists |
| **Partners** | Partner Nodes (Tier 3). Manage spokes, sync status. | ✅ Exists (as "Spokes") |

---

### PLATFORM

Global configuration that applies across the entire platform.

| Section | Purpose | Configurable Items |
|---------|---------|-------------------|
| **Networks** | Community networks/groups that can be tagged on events and businesses | Add/edit/delete networks, set active status, sort order |
| **Tiers** | Business and User tier definitions | Tier names, prices, features included per tier |
| **Joy Coins** | Joy Coins / Community Pass settings | Coin pricing range, revenue share %, monthly reset rules |
| **Branding** | Platform visual identity | Logo, accent color (future: per-community branding) |
| **Newsletter** | Subscriber management, send history, capture points | Subscriber list, email drafts (future), footer/RSVP/onboarding capture settings |

---

### EVENTS (Archetype Configuration)

Configuration specific to the Events archetype.

| Section | Purpose | Configurable Items |
|---------|---------|-------------------|
| **Event Types** | Categories for events | Add/edit/delete types (Markets, Live Music, etc.), icons, sort order |
| **Age Groups** | Audience targeting options | Add/edit/delete age ranges (All Ages, 5-12, etc.) |
| **Durations** | Preset duration options | Add/edit duration presets (30min, 1hr, 2hr, etc.) |

---

### VENUES (Archetype Configuration — Phase 2)

Configuration specific to the Venues/Locations archetype.

| Section | Purpose | Configurable Items |
|---------|---------|-------------------|
| **Venue Types** | Categories for venues | Restaurant, Cafe, Gym, Park, etc. |
| **Amenities** | Features a venue can have | WiFi, Parking, Wheelchair Access, etc. |
| **Capacities** | Capacity tier options | Intimate (1-20), Small (20-50), etc. |

---

### ONBOARDING

Configuration for the onboarding wizards (both supply and demand side).

| Section | Purpose | Configurable Items |
|---------|---------|-------------------|
| **Business Onboarding** | Supply-side wizard configuration | Active archetypes, goals per archetype, tier pricing display, feature matrix |
| **User Onboarding** | Demand-side wizard configuration | Interest categories, membership tiers, preference options |

#### Business Onboarding Config

```javascript
{
  archetypes: [
    { value: 'venue', label: 'Location / Venue', active: true, icon: 'building' },
    { value: 'organizer', label: 'Event Organizer', active: true, icon: 'calendar' },
    { value: 'service', label: 'Service Provider', active: false, icon: 'briefcase' },
    // ...
  ],
  goals: {
    venue: [
      { value: 'host_events', label: 'Host Events', min_tier: 'basic' },
      { value: 'accept_joy_coins', label: 'Accept Joy Coins', min_tier: 'standard' },
      // ...
    ],
    organizer: [
      { value: 'host_events', label: 'Host Events', min_tier: 'basic' },
      { value: 'sell_tickets', label: 'Sell Tickets', min_tier: 'standard' },
      // ...
    ],
  },
  tier_display: [
    { tier: 'basic', name: 'Basic', price: 'Free', features: [...] },
    { tier: 'standard', name: 'Standard', price: '$79/mo', features: [...] },
    { tier: 'partner', name: 'Partner', price: 'By invitation', features: [...] },
  ],
}
```

#### User Onboarding Config

```javascript
{
  interests: [
    { value: 'live_music', label: 'Live Music', icon: 'music' },
    { value: 'family_activities', label: 'Family Activities', icon: 'users' },
    { value: 'food_drink', label: 'Food & Drink', icon: 'utensils' },
    // ...
  ],
  membership_tiers: [
    { tier: 'explorer', name: 'Explorer', price: 'Free', features: [...] },
    { tier: 'member', name: 'Member', price: '$5/mo', features: [...] },
  ],
  preferences: [
    { value: 'newsletter', label: 'Weekly newsletter', default: true },
    { value: 'notifications', label: 'Event notifications', default: true },
    // ...
  ],
}
```

---

## UI Components

### Config List Component

Standard pattern for all configurable lists:

```
┌─────────────────────────────────────────────────────────────┐
│  Event Types                                    [+ Add Type] │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐│
│  │ ≡  Markets & Fairs                          [Edit] [🗑] ││
│  ├─────────────────────────────────────────────────────────┤
│  │ ≡  Live Music                               [Edit] [🗑] ││
│  ├─────────────────────────────────────────────────────────┤
│  │ ≡  Food & Drink                             [Edit] [🗑] ││
│  ├─────────────────────────────────────────────────────────┤
│  │ ≡  Workshops & Classes                      [Edit] [🗑] ││
│  └─────────────────────────────────────────────────────────┘│
│                                                             │
│  Drag to reorder • Click Edit to modify • Deleted items     │
│  are hidden, not removed (can be restored)                  │
└─────────────────────────────────────────────────────────────┘
```

Features:
- Drag-and-drop reordering (sort_order)
- Inline edit or modal edit
- Soft delete (set active: false)
- Search/filter for long lists

### Config Edit Modal

```
┌─────────────────────────────────────────────────┐
│  Edit Event Type                            [X] │
├─────────────────────────────────────────────────┤
│                                                 │
│  Label *                                        │
│  ┌─────────────────────────────────────────┐   │
│  │ Live Music                              │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  Value (system key)                            │
│  ┌─────────────────────────────────────────┐   │
│  │ live_music                              │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  Icon (optional)                               │
│  ┌─────────────────────────────────────────┐   │
│  │ music                               [▼] │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ☑ Active (visible in dropdowns)               │
│                                                 │
│           [Cancel]  [Save Changes]             │
└─────────────────────────────────────────────────┘
```

---

## Implementation Phases

### Phase 1: Foundation (Current Sprint)

1. **Restructure Admin Panel navigation** — Sidebar with sections
2. **Create PlatformConfig entity** — See CONFIG-SYSTEM.md
3. **Build Event Configuration section** — Event Types, Age Groups, Durations
4. **Update EventEditor** — Fetch config instead of hardcoded

### Phase 2: Platform & Onboarding

5. **Build Platform Settings section** — Networks, Tiers, Joy Coins
6. **Build Onboarding Configuration** — Business wizard config
7. **Update Business Onboarding** — Use config for archetypes, goals, tiers

### Phase 3: Additional Archetypes

8. **Build Venue Configuration** — When venues archetype is activated
9. **Build User Management section** — User list, tiers, preferences
10. **Build User Onboarding Configuration** — Interests, membership tiers

---

## File Structure

```
src/
├── pages/
│   └── Admin.jsx                    # Main admin page with sidebar
│
├── components/
│   └── admin/
│       ├── AdminSidebar.jsx         # Sidebar navigation
│       ├── AdminLayout.jsx          # Layout wrapper
│       │
│       ├── management/
│       │   ├── BusinessesSection.jsx    # Existing
│       │   ├── UsersSection.jsx         # New
│       │   ├── LocationsSection.jsx     # Existing
│       │   └── PartnersSection.jsx      # Existing (Spokes)
│       │
│       ├── platform/
│       │   ├── NetworksConfig.jsx
│       │   ├── TiersConfig.jsx
│       │   ├── JoyCoinsConfig.jsx
│       │   └── BrandingConfig.jsx
│       │
│       ├── events/
│       │   ├── EventTypesConfig.jsx
│       │   ├── AgeGroupsConfig.jsx
│       │   └── DurationsConfig.jsx
│       │
│       ├── venues/
│       │   ├── VenueTypesConfig.jsx
│       │   ├── AmenitiesConfig.jsx
│       │   └── CapacitiesConfig.jsx
│       │
│       ├── onboarding/
│       │   ├── BusinessOnboardingConfig.jsx
│       │   └── UserOnboardingConfig.jsx
│       │
│       └── shared/
│           ├── ConfigList.jsx       # Reusable list component
│           ├── ConfigEditModal.jsx  # Reusable edit modal
│           └── ConfigItem.jsx       # Single item row
│
└── hooks/
    └── useConfig.js                 # Hook to fetch config by domain/type
```

---

## Routing

```javascript
// Admin routes
/admin                      → Redirects to /admin/businesses
/admin/businesses           → BusinessesSection
/admin/users                → UsersSection
/admin/locations            → LocationsSection
/admin/partners             → PartnersSection

/admin/platform/networks    → NetworksConfig
/admin/platform/tiers       → TiersConfig
/admin/platform/joy-coins   → JoyCoinsConfig
/admin/platform/branding    → BrandingConfig

/admin/events/types         → EventTypesConfig
/admin/events/age-groups    → AgeGroupsConfig
/admin/events/durations     → DurationsConfig

/admin/venues/types         → VenueTypesConfig
/admin/venues/amenities     → AmenitiesConfig
/admin/venues/capacities    → CapacitiesConfig

/admin/onboarding/business  → BusinessOnboardingConfig
/admin/onboarding/user      → UserOnboardingConfig
```

---

## Styling

Follow STYLE-GUIDE.md:

- **Sidebar:** `bg-slate-900` with `border-r border-slate-700`
- **Active item:** `bg-slate-800` with `border-l-2 border-amber-500`
- **Section headers:** `text-slate-400 text-xs uppercase tracking-wider`
- **Content area:** `bg-slate-800` 
- **Cards:** `bg-slate-900 border-slate-700`

---

*This architecture supports current needs while scaling for future archetypes and features.*
