# Onboarding System

> Defines the configurable onboarding flows for businesses (supply-side) and users (demand-side).
> Updated: 2026-01-29

---

## Overview

LocalLane has two onboarding flows:

1. **Business Onboarding** — For businesses/organizations joining the platform
2. **User Onboarding** — For consumers discovering local events and businesses

Both flows should be configurable via the Admin Panel, not hardcoded.

---

## Business Onboarding (Supply-Side)

### Current Flow

```
Step 1: ARCHETYPE
   "What type of organization are you?"
   → Location/Venue, Event Organizer, Service Provider, etc.

Step 2: GOALS
   "What do you want to do?"
   → Host Events, Accept Punch Pass, List in Directory, etc.
   (Options filtered by archetype selected in Step 1)

Step 3: DETAILS
   "Tell us about your organization"
   → Name, description, contact, location
   (Fields shown based on archetype + goals)

Step 4: PLAN
   "Choose your plan"
   → Basic (Free), Standard ($X/mo), Partner (By invitation)
   (Some goals may require higher tiers)

Step 5: REVIEW
   "Review and confirm"
   → Summary of selections
```

### Configurable Elements

| Element | Config Location | What's Configurable |
|---------|-----------------|---------------------|
| **Archetypes** | `onboarding/archetypes` | Which archetypes are active, labels, icons, sort order |
| **Goals** | `onboarding/goals` | Goals per archetype, which tier unlocks each goal |
| **Detail Fields** | `onboarding/detail_fields` | Which fields show per archetype, required vs optional |
| **Tier Display** | `platform/business_tiers` | Tier names, prices, feature descriptions |
| **Feature Matrix** | `onboarding/feature_matrix` | What features each tier includes |

### Archetype Configuration

```javascript
// PlatformConfig: domain='onboarding', config_type='archetypes'
{
  items: [
    {
      value: 'venue',
      label: 'Location / Venue',
      description: 'Physical location where events or activities happen',
      icon: 'building',
      active: true,
      sort_order: 1,
    },
    {
      value: 'organizer',
      label: 'Event Organizer',
      description: 'Individual or organization that hosts events',
      icon: 'calendar',
      active: true,
      sort_order: 2,
    },
    {
      value: 'service',
      label: 'Service Provider',
      description: 'Offers services like classes, coaching, or consulting',
      icon: 'briefcase',
      active: false,  // Disabled for pilot
      sort_order: 3,
    },
    {
      value: 'community',
      label: 'Community / Non-Profit',
      description: 'Community group, non-profit, or volunteer organization',
      icon: 'heart',
      active: false,
      sort_order: 4,
    },
    {
      value: 'product',
      label: 'Product Seller',
      description: 'Sells products at markets, pop-ups, or online',
      icon: 'package',
      active: false,
      sort_order: 5,
    },
  ]
}
```

### Goals Configuration

```javascript
// PlatformConfig: domain='onboarding', config_type='goals'
{
  items: [
    // Venue goals
    {
      value: 'host_events',
      label: 'Host Events',
      description: 'Create and manage events at your location',
      parent: 'venue',  // Links to archetype
      min_tier: 'basic',
      unlocks_module: 'events',  // Which dashboard module this enables
      active: true,
      sort_order: 1,
    },
    {
      value: 'accept_punch_pass',
      label: 'Accept Punch Pass',
      description: 'Accept Punch Pass payments from customers',
      parent: 'venue',
      min_tier: 'standard',
      unlocks_module: 'punch_pass',
      active: true,
      sort_order: 2,
    },
    {
      value: 'list_directory',
      label: 'List in Directory',
      description: 'Appear in the LocalLane business directory',
      parent: 'venue',
      min_tier: 'basic',
      unlocks_module: 'directory',
      active: true,
      sort_order: 3,
    },
    
    // Organizer goals
    {
      value: 'host_events',
      label: 'Host Events',
      description: 'Create and promote your events',
      parent: 'organizer',
      min_tier: 'basic',
      unlocks_module: 'events',
      active: true,
      sort_order: 1,
    },
    {
      value: 'accept_punch_pass',
      label: 'Accept Punch Pass',
      description: 'Let attendees pay with Punch Pass',
      parent: 'organizer',
      min_tier: 'standard',
      unlocks_module: 'punch_pass',
      active: true,
      sort_order: 2,
    },
    {
      value: 'sell_tickets',
      label: 'Sell Tickets',
      description: 'Sell tickets with multiple price tiers',
      parent: 'organizer',
      min_tier: 'standard',
      unlocks_module: 'ticketing',
      active: true,
      sort_order: 3,
    },
  ]
}
```

### Feature Matrix Configuration

```javascript
// PlatformConfig: domain='onboarding', config_type='feature_matrix'
{
  items: [
    // Basic tier features
    { value: 'create_events', label: 'Create Events', tier: 'basic', included: true },
    { value: 'event_review', label: 'Events reviewed before publishing', tier: 'basic', included: true },
    { value: 'basic_profile', label: 'Basic directory profile', tier: 'basic', included: true },
    { value: 'community_support', label: 'Community support', tier: 'basic', included: true },
    
    // Standard tier features
    { value: 'auto_publish', label: 'Auto-publish events', tier: 'standard', included: true },
    { value: 'punch_pass', label: 'Accept Punch Pass', tier: 'standard', included: true },
    { value: 'analytics', label: 'Event analytics', tier: 'standard', included: true },
    { value: 'multiple_tickets', label: 'Multiple ticket types', tier: 'standard', included: true },
    { value: 'priority_support', label: 'Priority support', tier: 'standard', included: true },
    
    // Partner tier features
    { value: 'own_node', label: 'Your own branded app', tier: 'partner', included: true },
    { value: 'custom_domain', label: 'Custom domain', tier: 'partner', included: true },
    { value: 'api_access', label: 'API access', tier: 'partner', included: true },
    { value: 'white_label', label: 'White-label options', tier: 'partner', included: true },
  ]
}
```

### Tier Pricing Display

```javascript
// PlatformConfig: domain='platform', config_type='business_tiers'
{
  items: [
    {
      value: 'basic',
      label: 'Basic',
      price: 'Free',
      price_amount: 0,
      billing_period: null,
      description: 'Get started with the essentials',
      badge_color: 'slate',
      active: true,
      sort_order: 1,
    },
    {
      value: 'standard',
      label: 'Standard',
      price: '$79/month',
      price_amount: 79,
      billing_period: 'month',
      description: 'Everything you need to grow',
      badge_color: 'amber',
      recommended: true,  // Shows "Recommended" badge
      active: true,
      sort_order: 2,
    },
    {
      value: 'partner',
      label: 'Partner',
      price: '$149/month (by invitation)',
      price_amount: 149,
      billing_period: null,
      description: 'For established businesses ready to scale',
      badge_color: 'purple',
      active: true,
      sort_order: 3,
    },
  ]
}
```

---

## User Onboarding (Demand-Side)

### Current Flow (Simplified)

```
Step 1: INTERESTS
   "What are you interested in?"
   → Live Music, Family Activities, Food & Drink, etc.
   (Multi-select)

Step 2: LOCATION
   "Where are you located?"
   → City/zip or allow location access

Step 3: PREFERENCES
   "How do you want to hear from us?"
   → Newsletter, notifications, frequency

Step 4: MEMBERSHIP (Optional)
   "Upgrade for more benefits"
   → Explorer (Free), Member ($5/mo)
```

### Configurable Elements

| Element | Config Location | What's Configurable |
|---------|-----------------|---------------------|
| **Interests** | `onboarding/user_interests` | Interest categories, icons |
| **Membership Tiers** | `platform/user_tiers` | Tier names, prices, benefits |
| **Preferences** | `onboarding/user_preferences` | Notification options, defaults |

### User Interests Configuration

```javascript
// PlatformConfig: domain='onboarding', config_type='user_interests'
{
  items: [
    { value: 'live_music', label: 'Live Music', icon: 'music', active: true, sort_order: 1 },
    { value: 'family', label: 'Family Activities', icon: 'users', active: true, sort_order: 2 },
    { value: 'food_drink', label: 'Food & Drink', icon: 'utensils', active: true, sort_order: 3 },
    { value: 'fitness', label: 'Fitness & Wellness', icon: 'heart', active: true, sort_order: 4 },
    { value: 'arts', label: 'Arts & Culture', icon: 'palette', active: true, sort_order: 5 },
    { value: 'outdoors', label: 'Outdoor Activities', icon: 'tree', active: true, sort_order: 6 },
    { value: 'learning', label: 'Learning & Workshops', icon: 'book', active: true, sort_order: 7 },
    { value: 'nightlife', label: 'Nightlife', icon: 'moon', active: true, sort_order: 8 },
  ]
}
```

### User Membership Tiers

```javascript
// PlatformConfig: domain='platform', config_type='user_tiers'
{
  items: [
    {
      value: 'explorer',
      label: 'Explorer',
      price: 'Free',
      price_amount: 0,
      description: 'Discover local events and businesses',
      features: [
        'Browse all events',
        'Save favorites',
        'Basic recommendations',
      ],
      active: true,
      sort_order: 1,
    },
    {
      value: 'member',
      label: 'Member',
      price: '$5/month',
      price_amount: 5,
      billing_period: 'month',
      description: 'Get more from your local community',
      features: [
        'Everything in Explorer',
        'Punch Pass discounts',
        'Early access to events',
        'Member-only offers',
        'Ad-free experience',
      ],
      active: true,
      sort_order: 2,
    },
  ]
}
```

---

## How Onboarding Uses Config

### Business Onboarding Component

```javascript
// BusinessOnboarding.jsx

import { useConfig } from '@/hooks/useConfig';

function BusinessOnboarding() {
  // Fetch configuration
  const { data: archetypes } = useConfig('onboarding', 'archetypes');
  const { data: goals } = useConfig('onboarding', 'goals');
  const { data: tiers } = useConfig('platform', 'business_tiers');
  const { data: features } = useConfig('onboarding', 'feature_matrix');
  
  // Filter active archetypes
  const activeArchetypes = archetypes?.filter(a => a.active) || [];
  
  // Get goals for selected archetype
  const goalsForArchetype = goals?.filter(g => 
    g.active && g.parent === selectedArchetype
  ) || [];
  
  // Get features for display
  const featuresForTier = (tier) => features?.filter(f => 
    f.tier === tier && f.included
  ) || [];
  
  // ... render wizard steps
}
```

### Dynamic Field Rendering

```javascript
// Step 3: Details - fields based on archetype + goals

const FIELD_CONFIG = {
  // Base fields for all
  base: ['name', 'description', 'email', 'phone'],
  
  // Archetype-specific fields
  venue: ['address', 'hours', 'capacity', 'amenities'],
  organizer: ['website', 'social_links'],
  service: ['service_area', 'certifications'],
  
  // Goal-specific fields
  host_events: ['event_categories'],
  accept_punch_pass: ['punch_pass_terms'],
  list_directory: ['photos', 'logo'],
};

function getFieldsForSelection(archetype, goals) {
  const fields = [...FIELD_CONFIG.base];
  
  if (FIELD_CONFIG[archetype]) {
    fields.push(...FIELD_CONFIG[archetype]);
  }
  
  goals.forEach(goal => {
    if (FIELD_CONFIG[goal]) {
      fields.push(...FIELD_CONFIG[goal]);
    }
  });
  
  return [...new Set(fields)]; // Dedupe
}
```

---

## Admin Panel: Onboarding Configuration

### Business Onboarding Config Screen

```
┌─────────────────────────────────────────────────────────────────┐
│  Business Onboarding Configuration                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ARCHETYPES                                      [+ Add]        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ ✓ Location / Venue                          [Edit] [↑↓] │   │
│  │ ✓ Event Organizer                           [Edit] [↑↓] │   │
│  │ ○ Service Provider (disabled)               [Edit] [↑↓] │   │
│  │ ○ Community / Non-Profit (disabled)         [Edit] [↑↓] │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  GOALS                                           [+ Add]        │
│  Filter by archetype: [All ▼]                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Host Events          venue, organizer    Basic    [Edit] │   │
│  │ Accept Punch Pass    venue, organizer    Standard [Edit] │   │
│  │ List in Directory    venue               Basic    [Edit] │   │
│  │ Sell Tickets         organizer           Standard [Edit] │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  TIER DISPLAY                                    [Edit Matrix]  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Basic     | Free      | 4 features                      │   │
│  │ Standard  | $29/mo    | 9 features (Recommended)        │   │
│  │ Partner   | Invite    | 13 features                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Validation Rules

### Goal-Tier Validation

When user selects goals in onboarding:

```javascript
function validateGoalsForTier(selectedGoals, selectedTier, goalsConfig) {
  const tierLevel = { basic: 1, standard: 2, partner: 3 };
  const selectedTierLevel = tierLevel[selectedTier];
  
  const invalidGoals = selectedGoals.filter(goalValue => {
    const goal = goalsConfig.find(g => g.value === goalValue);
    const requiredTierLevel = tierLevel[goal.min_tier];
    return requiredTierLevel > selectedTierLevel;
  });
  
  return {
    valid: invalidGoals.length === 0,
    invalidGoals,
    message: invalidGoals.length > 0 
      ? `${invalidGoals.join(', ')} require${invalidGoals.length > 1 ? '' : 's'} a higher tier`
      : null,
  };
}
```

### Show Upgrade Prompt

```javascript
// In Step 2 (Goals), when user selects a goal requiring higher tier:

{goal.min_tier !== 'basic' && (
  <Badge variant="outline" className="text-amber-500 border-amber-500">
    Requires {tierLabels[goal.min_tier]}
  </Badge>
)}
```

---

## Future Enhancements

1. **A/B Testing** — Test different onboarding flows
2. **Skip Logic** — Skip steps based on previous answers
3. **Progress Saving** — Resume incomplete onboarding
4. **Analytics** — Track drop-off points
5. **Personalization** — Different flows for referrals vs organic

---

*This system allows Stewards to customize onboarding without code changes, adapting to new archetypes and features as the platform evolves.*
