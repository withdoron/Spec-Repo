# Phase 1 Audit: Community Node Business Dashboard

> Comprehensive audit of existing Business Dashboard implementation in Community Node
> Date: 2026-01-28

---

## Executive Summary

**Route:** `/business-dashboard` ✅ EXISTS  
**Status:** Partially implemented — structure exists, Event Module incomplete  
**Key Finding:** Event editor exists (`EventEditor.tsx.jsx`) but is NOT connected to `EventsWidget`

---

## What Exists

### 1. Main Dashboard Page

**File:** `src/pages/BusinessDashboard.jsx`

**Structure:**
- ✅ Multi-business selector (if user has multiple businesses)
- ✅ Business card grid view
- ✅ Selected business detail view
- ✅ Archetype-based widget configuration
- ✅ Role-based access (Owner, Manager, Instructor, Editor)

**Widgets by Archetype:**
- `location`/`venue`: Overview, Events, Staff, Financials, CheckIn
- `service`/`talent`: Overview, Schedule, Portfolio, Reviews
- `community`: Overview, Events, Members, Donations
- `organizer`: Overview, Ticketing, Marketing, Team

**Current Flow:**
1. If no businesses → Shows `PersonalDashboard`
2. If businesses exist but none selected → Shows business card grid
3. If business selected → Shows widgets based on archetype

**Issues:**
- ⚠️ Uses light theme in some widgets (OverviewWidget)
- ⚠️ No tier-based feature gating
- ⚠️ No goal-based module visibility

---

### 2. Events Widget

**File:** `src/components/dashboard/widgets/EventsWidget.jsx`

**What Works:**
- ✅ Fetches events for business (`base44.entities.Event.filter`)
- ✅ Displays event list with basic info
- ✅ Shows "Create Event" button
- ✅ Edit/Delete buttons (for Owner/Manager)
- ✅ Event count display

**What's Missing:**
- ❌ Event editor NOT connected — shows placeholder "Event editor coming soon"
- ❌ No event creation form
- ❌ No tier-based gating (Punch Pass, Multiple Tickets)
- ❌ No status management (pending_review vs published)
- ❌ No analytics/stats cards

**Current Implementation:**
```jsx
if (showEditor) {
  return (
    <Card className="p-6">
      <h2>{editingEvent ? 'Edit Event' : 'Create New Event'}</h2>
      <p className="text-sm text-slate-600">Event editor coming soon</p>
      <Button onClick={() => setShowEditor(false)}>Back to Events</Button>
    </Card>
  );
}
```

---

### 3. Event Editor Component (Unused)

**File:** `src/components/dashboard/EventEditor.tsx.jsx`

**Status:** ✅ EXISTS but NOT IMPORTED/USED

**Features:**
- ✅ Full event form with all fields
- ✅ Date/time picker
- ✅ Location selection
- ✅ Instructor selection
- ✅ Network partnership (Recess/TCA)
- ✅ Pricing options
- ✅ Punch Pass toggle
- ✅ Boost credits display
- ✅ Audience tags
- ✅ Event type/category

**Issues:**
- ❌ Not connected to EventsWidget
- ⚠️ Uses old tier names (not Standard/Storefront/Partner)
- ⚠️ No tier-based feature gating
- ⚠️ Missing some Event Node features (image upload, capacity, etc.)

---

### 4. Overview Widget

**File:** `src/components/dashboard/widgets/OverviewWidget.jsx`

**Status:** ✅ EXISTS

**Features:**
- ✅ Average Rating card
- ✅ Total Reviews card
- ✅ Profile Views card
- ✅ Boost Credits card

**Issues:**
- ⚠️ Uses LIGHT THEME (`bg-slate-50`, `text-slate-900`) — violates Gold Standard
- ⚠️ Uses colored backgrounds (amber-100, blue-100, etc.) — should be dark
- ❌ Missing event stats (Total Events, Upcoming, RSVPs, This Month)
- ❌ Missing Punch Pass earnings (should show for Tier 2+)

---

### 5. Other Widgets

**Files:**
- `FinancialWidget.jsx` ✅ EXISTS
- `StaffWidget.jsx` ✅ EXISTS
- `LocationsSection.jsx` ✅ EXISTS (used in BusinessDashboardDetail)

**Status:** Basic implementations exist, may need updates for Event Module integration

---

### 6. Business Dashboard Detail

**File:** `src/components/dashboard/BusinessDashboardDetail.jsx`

**Status:** ✅ EXISTS (but may be legacy/unused)

**Features:**
- ✅ Profile editing
- ✅ Location management
- ✅ Reviews display
- ✅ Tier upgrade dialog

**Issues:**
- ⚠️ Uses LIGHT THEME throughout
- ⚠️ Old tier names (Basic/Standard/Partner with wrong prices)
- ❌ Not integrated into main BusinessDashboard flow

---

## What's Missing

### Critical Missing Features

1. **Event Creation Form**
   - EventEditor exists but not connected
   - Need to integrate EventEditor into EventsWidget
   - Need to add missing fields from Event Node (image upload, capacity, etc.)

2. **Tier Checking Hook/Context**
   - No `useTier()` or `useOrganization()` hook
   - Tier info scattered across components
   - Need centralized tier checking

3. **Tier-Based Feature Gating**
   - No tier checks for Punch Pass features
   - No tier checks for Multiple Tickets
   - No auto-publish vs review queue logic
   - No upgrade prompts/CTAs

4. **Goal-Based Module Visibility**
   - No goal checking
   - All widgets show regardless of goals
   - Need to hide/show modules based on selected goals

5. **Dashboard Stats**
   - Missing: Total Events, Upcoming Events, Total RSVPs, This Month
   - Missing: Punch Pass Earnings (Tier 2+)
   - OverviewWidget only shows rating/reviews

6. **Event Status Management**
   - No "Pending Review" badge for Tier 1
   - No status field handling
   - Events created don't set status based on tier

---

## Data Model Analysis

### Business Entity Fields Used

```javascript
business {
  id: UUID,
  name: string,
  subscription_tier: 'basic' | 'standard' | 'partner',  // ⚠️ Old names
  archetype: string,  // ✅ Used for widget config
  owner_user_id: UUID,  // ✅ Used for role checking
  instructors: UUID[],  // ✅ Used for role checking
  associated_businesses: UUID[],  // ✅ Used in user record
  // ... other fields
}
```

### Event Entity Fields Used

```javascript
event {
  id: UUID,
  business_id: UUID,  // ✅ Used for filtering
  title: string,
  description: string,
  date: Date,
  end_date: Date,
  location: string,
  price: number,
  is_active: boolean,  // ✅ Used for filtering
  // ... other fields
}
```

**Missing Event Fields (from Event Node):**
- `status` (pending_review, published, etc.)
- `punch_cost` (1-5 punches)
- `punch_pass_eligible` (boolean)
- `image_url` (string)
- `capacity` (number)
- `event_type` (category)
- `networks` (array)
- `age_audience` (string)

---

## Route & Navigation

### Current Routes

| Route | Component | Status |
|-------|-----------|--------|
| `/business-dashboard` | `BusinessDashboard` | ✅ Exists |
| `/business-onboarding` | `BusinessOnboarding` | ✅ Exists |
| `/business-profile` | `BusinessProfile` | ✅ Exists |

### Navigation Links

**In Layout.jsx:**
- ✅ "Dashboard" link → `/business-dashboard`
- ✅ "Host Dashboard" link → `/business-dashboard` (if `is_business_owner`)
- ✅ "Start Hosting" link → `/business-onboarding` (if not owner)

**Access Control:**
- ⚠️ No route protection (anyone can access)
- ⚠️ No organization requirement check
- ⚠️ No active status check

---

## Styling Issues

### Light Theme Violations

**Files using light theme (need dark theme update):**
1. `OverviewWidget.jsx` — `bg-slate-50`, `text-slate-900`
2. `BusinessDashboardDetail.jsx` — Entire file uses light theme
3. `EventEditor.tsx.jsx` — Uses dark theme ✅ (correct)

**Files using correct dark theme:**
- `BusinessDashboard.jsx` ✅
- `EventsWidget.jsx` ✅
- `EventEditor.tsx.jsx` ✅

---

## Tier System Analysis

### Current Tier Values

**In Code:**
- `'basic'` — Tier 1 (but should be `'standard'`)
- `'standard'` — Tier 2 (but should be `'storefront'`)
- `'partner'` — Tier 3 ✅ (correct)

**In Admin Panel:**
- Uses: `basic`, `standard`, `partner` ✅

**Mismatch:**
- Spec says: Standard (Tier 1), Storefront (Tier 2), Partner (Tier 3)
- Code uses: basic (Tier 1), standard (Tier 2), partner (Tier 3)

**Action Needed:** Align tier names OR update Spec-Repo to match current implementation

---

## Component Mapping: Event Node → Community Node

### What to Copy from Event Node

| Event Node Component | Community Node Target | Status |
|----------------------|----------------------|--------|
| `Dashboard.jsx` (stats) | `OverviewWidget.jsx` | ⚠️ Partial — missing event stats |
| `Dashboard.jsx` (earnings) | New `PunchPassWidget.jsx` | ❌ Missing |
| `Events.jsx` (list) | `EventsWidget.jsx` | ✅ Exists (basic) |
| `EventForm.jsx` | `EventEditor.tsx.jsx` | ✅ Exists (unused) |
| `Settings.jsx` | New `SettingsWidget.jsx` | ❌ Missing |
| `CheckIn.jsx` | Placeholder exists | ⚠️ Placeholder only |

---

## Recommendations

### Immediate Actions (Phase 1 Complete)

1. ✅ **Connect EventEditor to EventsWidget**
   - Import EventEditor into EventsWidget
   - Replace placeholder with actual form
   - Wire up save/cancel handlers

2. ✅ **Add Missing Event Fields**
   - Image upload
   - Capacity
   - Event type/category dropdown
   - Networks selection (from admin config)
   - Age/Audience selection

3. ✅ **Create Tier Hook**
   - `useOrganization()` hook
   - Returns tier, tierLevel, canUsePunchPass, canAutoPublish
   - Reads from business.subscription_tier

4. ✅ **Update OverviewWidget**
   - Convert to dark theme
   - Add event stats cards
   - Add Punch Pass earnings (Tier 2+)

5. ✅ **Add Tier Gating**
   - Lock Punch Pass option for Tier 1
   - Lock Multiple Tickets for Tier 1
   - Set event status based on tier
   - Show upgrade prompts

---

## File Structure Summary

```
community-node/src/
├── pages/
│   └── BusinessDashboard.jsx          ✅ EXISTS (main page)
│
├── components/
│   ├── dashboard/
│   │   ├── BusinessCard.jsx           ✅ EXISTS
│   │   ├── BusinessDashboardDetail.jsx ✅ EXISTS (legacy?)
│   │   ├── EventEditor.tsx.jsx        ✅ EXISTS (UNUSED)
│   │   └── widgets/
│   │       ├── EventsWidget.jsx       ✅ EXISTS (incomplete)
│   │       ├── OverviewWidget.jsx     ✅ EXISTS (needs update)
│   │       ├── FinancialWidget.jsx    ✅ EXISTS
│   │       └── StaffWidget.jsx        ✅ EXISTS
│   └── ...
│
└── hooks/
    └── (no tier hooks)                 ❌ MISSING
```

---

## Next Steps (Phase 2)

Based on this audit, proceed with:

1. **Connect EventEditor** — Wire up existing form to EventsWidget
2. **Enhance EventEditor** — Add missing fields from Event Node
3. **Create Tier Hook** — `useOrganization()` for tier checking
4. **Update OverviewWidget** — Dark theme + event stats
5. **Add Tier Gating** — Lock features based on tier
6. **Add Goal-Based Visibility** — Show/hide modules by goals

---

*This audit provides the foundation for Phase 2 implementation.*
