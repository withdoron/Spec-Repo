# Configuration System

> Defines how platform configuration is stored, managed, and inherited across nodes.
> Updated: 2026-01-29

---

## Overview

LocalLane uses a centralized configuration system that allows Stewards to manage platform options (event types, networks, age groups, etc.) without code changes. Configuration is owned by the Community Node and inherited by Partner Nodes to ensure data congruency across the platform.

**Key Principles:**
1. **Single source of truth** — Community Node owns all config
2. **No hardcoding** — All dropdown options, categories, etc. come from config
3. **Congruent data** — Same options everywhere = consistent search/filter
4. **Future flexibility** — Structure supports Partner customization later

---

## Data Model

### PlatformConfig Entity

```javascript
// Entity: PlatformConfig
{
  id: UUID,
  
  // Classification
  domain: string,       // 'platform' | 'events' | 'venues' | 'services' | 'onboarding'
  config_type: string,  // 'event_types' | 'networks' | 'age_groups' | 'tiers' | etc.
  
  // The actual configuration items
  items: [
    {
      value: string,        // System key (e.g., 'live_music')
      label: string,        // Display name (e.g., 'Live Music')
      icon: string,         // Optional icon name
      description: string,  // Optional description
      active: boolean,      // Whether visible in UI
      sort_order: number,   // Display order
      
      // For tier-gated features
      min_tier: string,     // 'basic' | 'standard' | 'partner' | null
      
      // For nested config (e.g., goals per archetype)
      parent: string,       // Optional parent value
      
      // Metadata
      metadata: object,     // Flexible additional data
    },
  ],
  
  // Partner customization (future)
  allow_partner_additions: boolean,  // Can partners add custom items?
  allow_partner_disable: boolean,    // Can partners hide items?
  
  // Audit
  created_at: Date,
  updated_at: Date,
  updated_by: UUID,
}
```

---

## Domain & Config Type Reference

### Platform Domain

Global configuration that applies everywhere.

| config_type | Purpose | Example Items |
|-------------|---------|---------------|
| `networks` | Community networks/groups | Homeschool Community, Recess, Creative Alliance |
| `business_tiers` | Business subscription tiers | Basic, Standard, Partner |
| `user_tiers` | User membership tiers | Explorer, Member |
| `joy_coin_settings` | Joy Coins / Community Pass configuration | coin_price_range, revenue_share_percent, monthly_reset |

### Events Domain

Configuration for the Events archetype.

| config_type | Purpose | Example Items |
|-------------|---------|---------------|
| `event_types` | Event categories | Markets & Fairs, Live Music, Food & Drink |
| `age_groups` | Audience targeting | All Ages, 5-12, 13-17, Adults |
| `duration_presets` | Duration options | 30 min, 1 hr, 2 hrs, 3 hrs |
| `pricing_types` | Pricing models | Free, Single Price, Pay What You Wish |

### Venues Domain

Configuration for the Venues archetype.

| config_type | Purpose | Example Items |
|-------------|---------|---------------|
| `venue_types` | Venue categories | Restaurant, Cafe, Gym, Park, Gallery |
| `amenities` | Venue features | WiFi, Parking, Wheelchair Access |
| `capacity_tiers` | Capacity ranges | Intimate (1-20), Small (20-50), Large (100+) |

### Onboarding Domain

Configuration for onboarding wizards.

| config_type | Purpose | Example Items |
|-------------|---------|---------------|
| `archetypes` | Business types | Venue, Event Organizer, Service Provider |
| `goals` | What businesses want to do | Host Events, Accept Punch Pass, List in Directory |
| `user_interests` | User preference categories | Live Music, Family Activities, Food & Drink |
| `feature_matrix` | What features each tier unlocks | (see below) |

---

## Config Inheritance Model

```
┌─────────────────────────────────────────────────────────────────┐
│                     COMMUNITY NODE                               │
│                   (Source of Truth)                              │
│                                                                  │
│  PlatformConfig entities stored here                            │
│  ├── platform/networks                                          │
│  ├── platform/business_tiers                                    │
│  ├── events/event_types                                         │
│  ├── events/age_groups                                          │
│  └── ... all config                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ API: GET /config/:domain/:type
                              │ Returns: { items: [...] }
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
   │ Partner A   │   │ Partner B   │   │ Partner C   │
   │             │   │             │   │             │
   │ Fetches     │   │ Fetches     │   │ Fetches     │
   │ config on   │   │ config on   │   │ config on   │
   │ load        │   │ load        │   │ load        │
   │             │   │             │   │             │
   │ Uses same   │   │ Uses same   │   │ Uses same   │
   │ event_types │   │ event_types │   │ event_types │
   │ networks    │   │ networks    │   │ networks    │
   │ age_groups  │   │ age_groups  │   │ age_groups  │
   └─────────────┘   └─────────────┘   └─────────────┘
```

### Why Inheritance Matters

**Scenario without inheritance:**
- Partner A creates event type "Acoustic Night"
- Partner B has no "Acoustic Night" option
- User searches Community Node for "Acoustic Night"
- Only Partner A's events appear — confusing!

**Scenario with inheritance:**
- Community Node defines all event types
- All partners use the same list
- User searches work consistently everywhere
- Data is congruent across the platform

### Future: Partner Additions

When `allow_partner_additions: true`:

```javascript
// Partner can add custom items
{
  value: 'acoustic_night',
  label: 'Acoustic Night',
  source: 'partner',           // Marked as partner-specific
  partner_id: 'partner_abc',   // Which partner added it
  active: true,
}
```

- Custom items appear in that Partner's dropdowns
- When synced to Community Node, tagged as partner-specific
- Community search can filter or include partner-specific types
- **Not implemented now** — structure supports it for future

---

## API Endpoints

### Read Config (Public)

```
GET /api/config/:domain/:type

Example: GET /api/config/events/event_types

Response:
{
  domain: 'events',
  config_type: 'event_types',
  items: [
    { value: 'live_music', label: 'Live Music', icon: 'music', active: true, sort_order: 1 },
    { value: 'markets', label: 'Markets & Fairs', icon: 'store', active: true, sort_order: 2 },
    ...
  ],
  updated_at: '2026-01-29T...'
}
```

### Write Config (Admin Only)

```
PUT /api/config/:domain/:type

Body:
{
  items: [
    { value: 'live_music', label: 'Live Music', ... },
    ...
  ]
}

Response:
{ success: true, updated_at: '2026-01-29T...' }
```

### Get All Config for Domain

```
GET /api/config/:domain

Example: GET /api/config/events

Response:
{
  domain: 'events',
  configs: {
    event_types: { items: [...] },
    age_groups: { items: [...] },
    duration_presets: { items: [...] },
  }
}
```

---

## Frontend Usage

### useConfig Hook

```javascript
// hooks/useConfig.js

import { useQuery } from '@tanstack/react-query';
import { base44 } from '@/api/base44Client';

export function useConfig(domain, configType) {
  return useQuery({
    queryKey: ['config', domain, configType],
    queryFn: async () => {
      const configs = await base44.entities.PlatformConfig.filter({
        domain,
        config_type: configType,
      });
      return configs[0]?.items || [];
    },
    staleTime: 5 * 60 * 1000, // Cache for 5 minutes
  });
}

// Usage in EventEditor:
const { data: eventTypes, isLoading } = useConfig('events', 'event_types');
const { data: networks } = useConfig('platform', 'networks');
const { data: ageGroups } = useConfig('events', 'age_groups');
```

### Fallback to Defaults

During migration or if config fails to load:

```javascript
const DEFAULT_EVENT_TYPES = [
  { value: 'markets_fairs', label: 'Markets & Fairs' },
  { value: 'live_music', label: 'Live Music' },
  // ...
];

const { data: eventTypes = DEFAULT_EVENT_TYPES, isLoading } = useConfig('events', 'event_types');
```

---

## Initial Data Seed

When setting up a new Community Node, seed with default config:

```javascript
// seeds/platformConfig.js

const INITIAL_CONFIG = [
  // Platform - Networks
  {
    domain: 'platform',
    config_type: 'networks',
    items: [
      { value: 'homeschool_community', label: 'Homeschool Community', active: true, sort_order: 1 },
      { value: 'recess', label: 'Recess', active: true, sort_order: 2 },
      { value: 'creative_alliance', label: 'Creative Alliance', active: true, sort_order: 3 },
      { value: 'local_parents', label: 'Local Parents Network', active: true, sort_order: 4 },
      { value: 'arts_council', label: 'Arts Council', active: true, sort_order: 5 },
      { value: 'fitness_coalition', label: 'Fitness Coalition', active: true, sort_order: 6 },
    ],
  },
  
  // Events - Event Types
  {
    domain: 'events',
    config_type: 'event_types',
    items: [
      { value: 'markets_fairs', label: 'Markets & Fairs', active: true, sort_order: 1 },
      { value: 'live_music', label: 'Live Music', active: true, sort_order: 2 },
      { value: 'food_drink', label: 'Food & Drink', active: true, sort_order: 3 },
      { value: 'workshops_classes', label: 'Workshops & Classes', active: true, sort_order: 4 },
      { value: 'sports_active', label: 'Sports & Active', active: true, sort_order: 5 },
      { value: 'art_culture', label: 'Art & Culture', active: true, sort_order: 6 },
      { value: 'meetings_gatherings', label: 'Meetings & Gatherings', active: true, sort_order: 7 },
      { value: 'other', label: 'Other', active: true, sort_order: 99 },
    ],
  },
  
  // Events - Age Groups
  {
    domain: 'events',
    config_type: 'age_groups',
    items: [
      { value: 'all_ages', label: 'All Ages', active: true, sort_order: 1 },
      { value: '5_12', label: '5-12', active: true, sort_order: 2 },
      { value: '13_17', label: '13-17', active: true, sort_order: 3 },
      { value: 'adults', label: 'Adults (18+)', active: true, sort_order: 4 },
      { value: 'seniors', label: 'Seniors (65+)', active: true, sort_order: 5 },
    ],
  },
  
  // Events - Duration Presets
  {
    domain: 'events',
    config_type: 'duration_presets',
    items: [
      { value: '30', label: '30 minutes', metadata: { minutes: 30 }, active: true, sort_order: 1 },
      { value: '60', label: '1 hour', metadata: { minutes: 60 }, active: true, sort_order: 2 },
      { value: '90', label: '1.5 hours', metadata: { minutes: 90 }, active: true, sort_order: 3 },
      { value: '120', label: '2 hours', metadata: { minutes: 120 }, active: true, sort_order: 4 },
      { value: '180', label: '3 hours', metadata: { minutes: 180 }, active: true, sort_order: 5 },
    ],
  },
  
  // Onboarding - Archetypes
  {
    domain: 'onboarding',
    config_type: 'archetypes',
    items: [
      { value: 'venue', label: 'Location / Venue', icon: 'building', active: true, sort_order: 1 },
      { value: 'organizer', label: 'Event Organizer', icon: 'calendar', active: true, sort_order: 2 },
      { value: 'service', label: 'Service Provider', icon: 'briefcase', active: false, sort_order: 3 },
      { value: 'community', label: 'Community / Non-Profit', icon: 'heart', active: false, sort_order: 4 },
      { value: 'product', label: 'Product Seller', icon: 'package', active: false, sort_order: 5 },
    ],
  },
  
  // Onboarding - Goals (with parent archetype)
  {
    domain: 'onboarding',
    config_type: 'goals',
    items: [
      // Venue goals
      { value: 'host_events', label: 'Host Events', parent: 'venue', min_tier: 'basic', active: true, sort_order: 1 },
      { value: 'accept_punch_pass', label: 'Accept Punch Pass', parent: 'venue', min_tier: 'standard', active: true, sort_order: 2 },
      { value: 'list_directory', label: 'List in Directory', parent: 'venue', min_tier: 'basic', active: true, sort_order: 3 },
      
      // Organizer goals
      { value: 'host_events', label: 'Host Events', parent: 'organizer', min_tier: 'basic', active: true, sort_order: 1 },
      { value: 'accept_punch_pass', label: 'Accept Punch Pass', parent: 'organizer', min_tier: 'standard', active: true, sort_order: 2 },
      { value: 'sell_tickets', label: 'Sell Tickets', parent: 'organizer', min_tier: 'standard', active: true, sort_order: 3 },
    ],
  },
];
```

---

## Migration Path

### Step 1: Create Entity

Add `PlatformConfig` entity to Base44 schema.

### Step 2: Seed Initial Data

Run seed script to populate default config.

### Step 3: Build Admin UI

Create config management screens in Admin Panel.

### Step 4: Update Components

Replace hardcoded arrays with `useConfig()` hook.

### Step 5: Partner Node Integration

Add config fetch to Partner Node startup.

---

## Caching Strategy

### Frontend

- **React Query** with 5-minute stale time
- Config rarely changes, so aggressive caching is fine
- Invalidate on admin save

### Partner Nodes

- Fetch config on app startup
- Cache in memory or local storage
- Refresh every 24 hours or on admin notification
- Future: WebSocket push when config changes

---

*This system ensures consistent, maintainable configuration across the LocalLane platform.*
