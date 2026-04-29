# Tier System

> Defines the business/organization tier system for LocalLane.
> Updated: 2026-02-05

---

## Overview

LocalLane uses a 3-tier system for businesses and organizations. Tiers determine feature access, pricing, and platform privileges.

---

## Tier Definitions

| Tier Level | Code Value | Display Name | Cost | Description |
|------------|------------|--------------|------|-------------|
| **Tier 1** | `basic` | Basic | Free | Entry level — listing, events (reviewed), directory presence |
| **Tier 2** | `standard` | Standard | $79/month | Full features, auto-publish, Community Pass pool, analytics, business network |
| **Tier 3** | `partner` | Partner | $149/month (earned) | Own Partner Node, full autonomy, enhanced features |

**Founding deal:** Standard tier businesses get first 3 months free.

### What Businesses Get (Beyond Features)

Business tiers are a separate product from Community Pass. Standard and Partner businesses are part of a **local business network** — think Eugene executive membership or chamber of commerce energy, but connected to real customer flow.

**Network benefits include:**
- Community Pass foot traffic (Joy Coin scans = revenue from pool)
- Business events and problem-solving workshops
- Visibility in the LocalLane directory and newsletter (The Good News)
- Analytics dashboard with network-wide insights
- Connection to other local business owners
- Organic marketing from member social media and word of mouth

**The pitch isn't just "list your business." It's "join a living network that sends you customers and connects you with other owners who get it."**

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
| Joy Coins Eligible | ❌ | ✅ | ✅ |
| Joy Coins Revenue | ❌ | ✅ | ✅ |
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
  canUseJoyCoins,     // true for standard+
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
const canUseJoyCoins = tierLevel >= 2;
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
  <JoyCoinsToggle />
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

*Always use the exact tier strings (`basic`, `standard`, `partner`) in code and database.*
