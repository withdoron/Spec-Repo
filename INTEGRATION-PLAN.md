# Integration Plan: Event Module into Community Node

> Step-by-step instructions for integrating Event Node features into Community Node's Business Dashboard.

---

## Overview

**Goal:** Tier 1 & 2 users should be able to create and manage events within the Community Node, using the same UI/UX that exists in the Event Node.

**Approach:** Copy working components from Event Node, adapt for Community Node's data model, add tier-based visibility.

---

## Pre-Work: Decisions Made

**Tier Naming:** Using 3-tier system
- **Standard** (Free) — Tier 1
- **Storefront** (Paid) — Tier 2  
- **Partner** (Earned + Paid) — Tier 3

**Wizard Step Order:** Goals before Details
- New: Archetype → Goals → Details → Plan → Review

**Dev Mode Styling:** Purple accents are intentional
- Purple differentiates dev/test mode from production
- Do NOT change to amber — this is by design

---

## Pre-Work: Questions to Confirm

Before starting, confirm with Doron:

1. **Business Dashboard Route:** What's the current route? `/business-dashboard`? `/organizer`? Does it exist?

2. **Pricing:** What are the actual prices for Storefront tier? (Standard is free, Partner is earned)

---

## Phase 1: Audit & Preparation

### Step 1.1: Audit Community Node Business Dashboard

**Tell Cursor:**
```
In the Community Node repo, find and list all files related to the Business Dashboard. 
Check:
- Routes that include "business" or "dashboard" (excluding admin)
- Components in src/components that relate to business management
- Any existing event management code

Report what exists and what's missing.
```

### Step 1.2: Audit Event Node Components

**Tell Cursor:**
```
In the Event Node repo, list all components that should be copied to Community Node:
- Dashboard components (stats cards, quick actions, earnings)
- Event list/table components
- Event form components (create/edit)
- Settings components (especially tier-gating patterns)
- Any shared utilities

Create a mapping of what to copy and where it should go.
```

---

## Phase 2: Create Business Dashboard Structure

### Step 2.1: Create Business Dashboard Route & Layout

**Tell Cursor:**
```
In Community Node, create the Business Dashboard at /business-dashboard (or /organizer).

Structure:
- Layout with sidebar navigation
- Tabs/sections for: Dashboard Home, My Events, Create Event, Settings
- Pull the current user's organization/business data
- Show loading state while fetching

Reference the Event Node's layout for styling consistency.
Follow STYLE-GUIDE.md for all colors and components.
```

### Step 2.2: Create Dashboard Home Module

**Tell Cursor:**
```
Copy the Dashboard home from Event Node to Community Node's Business Dashboard.

Include:
- Stats cards (Total Events, Upcoming, RSVPs, This Month)
- Quick Actions section
- Punch Pass Earnings section (show only for Tier 2+)

Data should come from Community Node's database, not Event Node.
Add tier-based visibility: hide earnings section for Tier 1.
```

---

## Phase 3: Event Management

### Step 3.1: Create Events List View

**Tell Cursor:**
```
Copy the Events list/table from Event Node to Community Node's Business Dashboard.

Include:
- Search bar
- Filter dropdowns
- Events table with: Title, Date, RSVPs, Status, Actions
- Pagination if needed
- "Create Event" button

Filter events to only show the current user's organization's events.
```

### Step 3.2: Create Event Form (Create/Edit)

**Tell Cursor:**
```
Copy the Event creation form from Event Node to Community Node.

Include all sections:
- Basic Info (title, description)
- Date & Time
- Location
- Image upload
- Categorization (Event Type, Networks, Age/Audience, Capacity)
- Pricing & Tickets

CRITICAL: Implement tier-based feature gating:
- Tier 1: No "Multiple Tickets", no "Punch Pass Eligible"
- Tier 2+: All features unlocked

Copy the exact UI pattern from Event Node (lock icon, "Upgrade" link, grayed out state).
```

### Step 3.3: Wire Up Event CRUD

**Tell Cursor:**
```
Connect the event form to Community Node's database:

- CREATE: Save new events directly to Community Node's events table
  - Set status to 'pending_review' for Tier 1
  - Set status to 'published' for Tier 2+
  
- READ: Fetch events for the current organization
  
- UPDATE: Allow editing own events
  
- DELETE: Allow deleting own events (soft delete preferred)

Events created here do NOT need to sync anywhere - they're already in the Community Node.
```

---

## Phase 4: Tier-Based Gating

### Step 4.1: Create Tier Context/Hook

**Tell Cursor:**
```
Create a hook or context that provides the current organization's tier:

useTier() or useOrganization() should return:
- tier: 'standard' | 'storefront' | 'partner'
- tierLevel: 1 | 2 | 3
- canUsePunchPass: boolean (Storefront+)
- canAutoPublish: boolean (Storefront+)
- isPartner: boolean
- etc.

This should read from the organization record in the database.
Include a dev mode override (like Event Node's Settings) for testing.
```

### Step 4.2: Apply Tier Gating Throughout

**Tell Cursor:**
```
Apply tier-based visibility throughout Business Dashboard:

STANDARD (Tier 1):
- Can create events (go to review queue)
- Show "Pending Review" badge on submitted events
- Hide Punch Pass section
- Hide "Multiple Tickets" option
- Hide analytics/stats (or show limited)
- Show upgrade prompts where features are locked

STOREFRONT (Tier 2):
- All Standard features
- Events auto-publish (no review)
- Punch Pass Eligible option available
- Full analytics
- Check-in feature

Use the same UI patterns as Event Node (lock icon, upgrade CTA).
```

---

## Phase 5: Settings & Profile

### Step 5.1: Create Organization Settings

**Tell Cursor:**
```
Create a Settings page in Business Dashboard:

Sections:
- Organization Profile (name, description, contact, photos)
- Subscription Status (current tier, renewal date)
- Developer Tier Override (for testing, hide in production)
- Notification preferences

Copy the Settings layout from Event Node, adapt for Community Node's data model.
```

---

## Phase 6: Navigation & Access Control

### Step 6.1: Update User Menu

**Tell Cursor:**
```
Update the user dropdown menu in Community Node header:

If user has an organization:
- Show "Business Dashboard" link → goes to /business-dashboard

If user is admin:
- Show "Admin Panel" link (already exists)
- Show "Spokes" link (already exists)

If user has no organization:
- Show "Create Organization" link → goes to onboarding wizard
```

### Step 6.2: Protect Routes

**Tell Cursor:**
```
Add route protection to Business Dashboard:

- /business-dashboard/* requires:
  1. User is logged in
  2. User has an organization
  3. Organization status is 'active'

If not logged in → redirect to login
If no organization → redirect to onboarding wizard
If organization inactive → show "pending approval" message
```

---

## Phase 7: Fix Existing Issues

### Step 7.1: Fix Organizer Name Display

**Tell Cursor:**
```
In the event detail display (Community Node), fix the organizer name:

Current: Shows "Event Spoke" 
Should: Show the actual business/organization name

Trace where this comes from:
- If event is from Partner Node sync, use the organization name from sync payload
- If event is created in Community Node, use the organization name from the events table

Update both the event card and event detail modal/page.
```

### Step 7.2: Fix Wizard Step Order

**Tell Cursor:**
```
In the onboarding wizard, change the step order:

Current: Archetype → Details → Goals → Plan → Review
New: Archetype → Goals → Details → Plan → Review

Goals determines what Details to show, so Goals must come first.

Update:
- Step numbers in the wizard UI
- Navigation logic
- Any step completion validation
```

---

## Phase 8: Testing Checklist

After implementation, test:

### Business Dashboard
- [ ] Can access Business Dashboard from user menu
- [ ] Dashboard shows correct stats for my organization
- [ ] Can see list of my events only
- [ ] Can create a new event
- [ ] Can edit an existing event
- [ ] Can delete an event

### Standard Tier Flow
- [ ] Punch Pass option is locked with upgrade CTA
- [ ] Multiple Tickets is locked
- [ ] Created event shows "Pending Review" status
- [ ] Cannot access check-in feature

### Storefront Tier Flow
- [ ] Punch Pass option is available
- [ ] Can set punch cost
- [ ] Created event auto-publishes
- [ ] Can see Punch Pass earnings
- [ ] Check-in feature accessible

### Integration
- [ ] Events created in Business Dashboard appear in Browse Events
- [ ] Organizer name shows correctly (not "Event Spoke")
- [ ] Event detail shows all correct information

---

## File Structure After Integration

```
community-node/
├── src/
│   ├── pages/
│   │   ├── BusinessDashboard/
│   │   │   ├── index.jsx           # Main dashboard layout
│   │   │   ├── DashboardHome.jsx   # Stats and overview
│   │   │   ├── EventsList.jsx      # Event management list
│   │   │   ├── EventForm.jsx       # Create/Edit form
│   │   │   ├── Settings.jsx        # Organization settings
│   │   │   └── CheckIn.jsx         # Check-in feature (Tier 2+)
│   │   └── ...
│   ├── components/
│   │   ├── business/
│   │   │   ├── StatsCard.jsx
│   │   │   ├── EventsTable.jsx
│   │   │   ├── TierGate.jsx        # Wrapper for tier-locked features
│   │   │   ├── UpgradePrompt.jsx   # "Upgrade to unlock" UI
│   │   │   └── ...
│   │   └── ...
│   ├── hooks/
│   │   ├── useOrganization.js      # Current org data + tier
│   │   └── useTier.js              # Tier checking utilities
│   └── ...
└── ...
```

---

## Estimated Time

| Phase | Time |
|-------|------|
| Phase 1: Audit | 1-2 hours |
| Phase 2: Structure | 2-3 hours |
| Phase 3: Event Management | 4-6 hours |
| Phase 4: Tier Gating | 2-3 hours |
| Phase 5: Settings | 1-2 hours |
| Phase 6: Navigation | 1 hour |
| Phase 7: Fixes | 1-2 hours |
| Phase 8: Testing | 2 hours |
| **Total** | **14-21 hours** |

---

## Quick Reference for Cursor

**Key Commands:**

1. "Read the Spec-Repo files first, especially STYLE-GUIDE.md and CURRENT-STATE.md"

2. "In Event Node, find the component for [X] and copy it to Community Node"

3. "Add tier-based visibility: show only for Tier 2+ organizations"

4. "Follow the exact UI pattern from Event Node for locked features"

5. "Ensure all styles follow the Gold Standard - no purple, no light backgrounds"

---

*This plan provides a structured approach to integrating Event Node features into Community Node.*
