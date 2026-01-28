# Current Implementation State

> What currently exists in the LocalLane ecosystem as of 2026-01-28.

---

## Community Node

**URL:** locallane.app (or Base44 preview)
**Repo:** github.com/withdoron/community-node

### Pages & Features

| Page | Route | Status | Notes |
|------|-------|--------|-------|
| **Home/Landing** | `/` | ✅ Built | Public landing page |
| **Browse Events** | `/events` | ✅ Built | Event listing with filters |
| **Event Detail** | `/events/:id` | ✅ Built | Modal/page with full details |
| **Browse Directory** | `/directory` | ⚠️ Exists | May need cleanup for pilot |
| **My Lane** | `/my-lane` | ✅ Built | User personal dashboard |
| **Dashboard** | `/dashboard` | ✅ Built | Welcome page with cards |
| **Business Dashboard** | `/business-dashboard` | ⚠️ Unclear | Needs Event Module integration |
| **Admin Panel** | `/admin` | ✅ Built | Full admin functionality |
| **Onboarding Wizard** | `/onboarding` | ✅ Built | 5-step wizard |

### User Dashboard ("My Lane" / "Dashboard")

Cards/Sections:
- ✅ Build Your Local Lane (preferences/interests)
- ✅ My Saved Items
- ✅ Punch Pass Network (shows membership status)
- ✅ Host Center (Create Organization CTA)
- ✅ Membership Status
- ✅ Magic Planner (teased feature)
- ✅ Quick filters: For You, This Weekend, Tonight, Near Me, Exclusive Offers

### Admin Panel

Tabs:
- ✅ Businesses — List/search/filter, tier management, status toggles
- ✅ Locations — Geographic area management
- ✅ Settings — Platform configuration
- ✅ Spokes — Partner Node management

Features per business:
- Tier dropdown (Standard, Storefront, etc.)
- Boosted toggle
- Silver toggle
- Local Franchise toggle
- Status (Active/Pending/etc.)
- Edit button

### Onboarding Wizard

Steps (current order):
1. Archetype — What type of entity
2. Details — Basic info (should be after Goals)
3. Goals — What they want to do (should be step 2)
4. Plan — Tier selection
5. Review — Confirmation

**TODO:** Swap Goals and Details order

Archetypes available:
- Location / Venue
- Event Organizer
- Service Provider
- Community / Non-Profit
- Product Seller

### Public Event Display

Event cards show:
- ✅ Image
- ✅ Title
- ✅ Date/time
- ✅ Location
- ✅ Punch cost badge
- ✅ Network badges
- ✅ Category badges

Event detail shows:
- ✅ All card info expanded
- ✅ Duration
- ✅ Age/Audience
- ✅ "Punch Pass Eligible" badge
- ⚠️ Organizer name (shows "Event Spoke" instead of business name)

---

## Event Node (Tier 3 Partner Template)

**URL:** Base44 preview
**Repo:** github.com/withdoron/events-node

### Pages & Features

| Page | Route | Status | Notes |
|------|-------|--------|-------|
| **Dashboard** | `/` or `/dashboard` | ✅ Built | Stats, quick actions, earnings |
| **Events** | `/events` | ✅ Built | Event list with management |
| **Create Event** | `/events/new` | ✅ Built | Full form with tier-gating |
| **Edit Event** | `/events/:id/edit` | ✅ Built | Same form, edit mode |
| **Settings** | `/settings` | ✅ Built | Profile, subscription, dev mode |
| **Check-in** | `/check-in` | ✅ Built | Event check-in feature |

### Dashboard Features

Stats cards:
- ✅ Total Events
- ✅ Upcoming Events
- ✅ Total RSVPs
- ✅ This Month

Punch Pass Earnings section:
- ✅ Shows 85% business share (correct)
- ✅ Per-redemption calculation

### Event Creation (Tier-Gated Features)

**Available to All Tiers:**
- Basic event info (title, description, date, time, location)
- Image upload
- Categories (Markets & Fairs, Live Music, Food & Drink, etc.)
- Networks/Communities selection
- Age/Audience targeting
- Capacity
- Pricing types: Free, Single Price, Pay What You Wish

**Tier 2+ Only (locked with upgrade CTA for Tier 1):**
- Multiple Tickets pricing
- Punch Pass Eligible toggle
- Punch Cost selection (1-5 punches)

### Settings Page

Features:
- ✅ Developer Tier Override (switch tiers for testing)
- ✅ Subscription Status display
- ✅ Manage Billing link

Tier options in Dev Mode (needs updating):
- Basic ($29/month) → Should be: **Standard** (Free)
- Community ($79/month) → Should be: **Storefront** ($X/month)
- Plus ($129/month) → Remove (consolidate into Partner)
- Enterprise (Custom) → Should be: **Partner** (Earned + $Y/month)

**TODO:** Update tier names to match 3-tier system (Standard/Storefront/Partner)

**Note:** Purple accent in Dev Mode is intentional — differentiates dev/test mode from production UI

### Sync to Community Node

- ✅ Events created in Event Node appear in Community Node
- ✅ Network tags sync
- ✅ Punch Pass info syncs
- ⚠️ Business name shows as "Event Spoke" instead of actual name

---

## What's Working

1. ✅ Event creation in Event Node
2. ✅ Event sync to Community Node
3. ✅ Event display in Community Node
4. ✅ Tier-based feature gating in Event Node
5. ✅ Admin Panel for managing businesses
6. ✅ User Dashboard with personalization
7. ✅ Onboarding Wizard flow
8. ✅ Network/Category tagging
9. ✅ Punch Pass UI (display, not redemption)

---

## What Needs Work

### Critical for Pilot

1. **Business Dashboard in Community Node** — Copy Event Node features for Tier 1/2
2. **Organizer name display** — Shows "Event Spoke" instead of business name
3. **Wizard step order** — Goals should come before Details

### Nice to Have

4. **Tier naming alignment** — Basic/Community/Plus/Enterprise → Standard/Storefront/Partner (3-tier system)
5. **User RSVP flow** — Not yet functional
6. **Punch Pass purchase/redemption** — Demo mode needed

### Post-Pilot

7. **Directory cleanup** — Decide what to show/hide
8. **Magic Planner** — Currently teased, not functional
9. **Real payments** — Stripe integration

---

## Data Flow Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         COMMUNITY NODE                                   │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │  User       │  │  Admin      │  │  Business   │  │  Public     │   │
│  │  Dashboard  │  │  Panel      │  │  Dashboard  │  │  Pages      │   │
│  │  (My Lane)  │  │             │  │  (TODO)     │  │  (Events)   │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
│         │                │                │                │           │
│         ▼                ▼                ▼                ▼           │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    COMMUNITY NODE DATABASE                       │  │
│  │  • Users          • Businesses      • Events        • Config    │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                   ▲                                    │
│                                   │ SYNC (events push up)              │
└───────────────────────────────────┼────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │       EVENT NODE (Tier 3)     │
                    │                               │
                    │  ┌──────────┐ ┌──────────┐   │
                    │  │Dashboard │ │ Events   │   │
                    │  └──────────┘ └──────────┘   │
                    │  ┌──────────┐ ┌──────────┐   │
                    │  │Settings  │ │ Check-in │   │
                    │  └──────────┘ └──────────┘   │
                    │                               │
                    │  ┌─────────────────────────┐ │
                    │  │   EVENT NODE DATABASE   │ │
                    │  └─────────────────────────┘ │
                    └───────────────────────────────┘
```

---

## Config Currently in Admin

Categories (Event Type):
- Markets & Fairs
- Live Music
- Food & Drink
- Workshops & Classes
- Sports & Active
- Art & Culture
- Meetings & Gatherings
- Other

Networks/Communities:
- Homeschool Community
- Recess
- Creative Alliance
- Local Parents Network
- Arts Council
- Fitness Coalition

Age/Audience:
- All Ages
- 5-12
- 13-17
- Adults (18+)
- Seniors (65+)
- Custom

---

*This document reflects the actual implementation state for development reference.*
