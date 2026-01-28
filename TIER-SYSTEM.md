# Tier System

> Defines the business/organization tier system for LocalLane.
> Updated: 2026-01-28

---

## Overview

LocalLane uses a 3-tier system for businesses and organizations. Tiers determine feature access, pricing, and platform privileges.

---

## Tier Definitions

| Tier Level | Code Value | Display Name | Cost | Description |
|------------|------------|--------------|------|-------------|
| **Tier 1** | `basic` | Basic | Free | Entry level, events go to review |
| **Tier 2** | `standard` | Standard | $X/month | Full features, auto-publish, Punch Pass |
| **Tier 3** | `partner` | Partner | Earned + $Y/month | Own Partner Node, full autonomy |

### Database Values

Always use these exact strings in code and database:
```javascript
subscription_tier: 'basic' | 'standard' | 'partner'
```

### Display Names

When showing to users, use friendly names:
- `basic` → "Basic" or "Free"
- `standard` → "Standard" or "Pro"
- `partner` → "Partner"

---

## Feature Matrix

| Feature | Basic | Standard | Partner |
|---------|-------|----------|---------|
| Create Events | ✅ (reviewed) | ✅ (auto-publish) | ✅ (own node) |
| Event Analytics | ❌ | ✅ | ✅ |
| Punch Pass Eligible | ❌ | ✅ | ✅ |
| Punch Pass Earnings | ❌ | ✅ | ✅ |
| Multiple Ticket Types | ❌ | ✅ | ✅ |
| Check-in Feature | ❌ | ✅ | ✅ |
| Directory Listing | Basic | Enhanced | Enhanced + Badge |
| Own Branding | ❌ | ❌ | ✅ |
| Own Domain | ❌ | ❌ | ✅ |
| API Access | ❌ | ❌ | ✅ |

---

## Event Status by Tier

| Tier | Event Status on Create | Approval Required |
|------|------------------------|-------------------|
| Basic | `pending_review` | Yes (Steward reviews) |
| Standard | `published` | No (trusted) |
| Partner | `published` | No (own node) |

---

## Tier Checking in Code

### The `useOrganization()` Hook

```javascript
const { 
  organization,
  tier,           // 'basic' | 'standard' | 'partner'
  tierLevel,      // 1 | 2 | 3
  canUsePunchPass,    // true for standard+
  canAutoPublish,     // true for standard+
  canUseMultipleTickets, // true for standard+
  canUseCheckIn,      // true for standard+
  isPartner,          // true for partner only
} = useOrganization();
```

### Tier Level Mapping

```javascript
const TIER_LEVELS = {
  'basic': 1,
  'standard': 2,
  'partner': 3,
};

const getTierLevel = (tier) => TIER_LEVELS[tier] || 1;
```

### Feature Checks

```javascript
// Check if feature is available
const canUsePunchPass = tierLevel >= 2;
const canAutoPublish = tierLevel >= 2;
const isPartner = tier === 'partner';
```

---

## UI Patterns for Locked Features

### Lock Icon with Upgrade CTA

```jsx
{tierLevel < 2 ? (
  <div className="flex items-center gap-2 text-slate-400">
    <Lock className="h-4 w-4 text-amber-500" />
    <span>Standard tier required</span>
    <Button variant="link" className="text-amber-500">
      Upgrade now
    </Button>
  </div>
) : (
  <PunchPassToggle />
)}
```

### Disabled State

```jsx
<Button 
  disabled={tierLevel < 2}
  className={tierLevel < 2 ? "opacity-50 cursor-not-allowed" : ""}
>
  {tierLevel < 2 && <Lock className="h-4 w-4 mr-2" />}
  Multiple Tickets
</Button>
```

---

## Upgrade Flow

### From Basic to Standard

1. User clicks "Upgrade" CTA
2. Show tier comparison modal
3. Redirect to Stripe checkout (future)
4. On success, update `subscription_tier` to `'standard'`
5. Features unlock immediately

### From Standard to Partner

Partner tier is **earned**, not just purchased:

**Requirements:**
- 6+ months at Standard tier
- 4.5+ star rating
- Engagement threshold met
- 3+ vouches from other partners
- Zero policy violations

**Process:**
1. User applies for Partner status
2. Steward reviews application
3. If approved, Partner Node is created
4. User transitions to Partner Node

---

## Admin Panel

### Tier Dropdown Options

In Admin Panel, the tier dropdown should show:
- `basic` (displayed as "Basic")
- `standard` (displayed as "Standard")
- `partner` (displayed as "Partner")

### Manual Tier Override

Admins can manually change tiers for:
- Testing purposes
- Special arrangements
- Founding 50 promotion

---

## Dev Mode

For development and testing, a **Dev Mode** tier override is available:

- Located in Business Dashboard Settings
- Uses **purple accent** (intentionally different from gold)
- Changes are saved to database
- Features update immediately

This allows testing tier-locked features without actual payment.

---

## Migration Notes

### Terminology Evolution

The tier naming has evolved:

| Old Name | Current Code | Blueprint Name |
|----------|--------------|----------------|
| Basic | `basic` | Standard |
| Community | `standard` | Storefront |
| Plus | (removed) | — |
| Enterprise | `partner` | Partner |

**Current implementation uses:** `basic` / `standard` / `partner`

This may be updated in the future to match Blueprint naming, but for now, use the code values consistently.

---

*Always use the exact tier strings (`basic`, `standard`, `partner`) in code and database.*
