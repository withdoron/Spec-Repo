# Current State

> Reflects actual implementation state as of 2026-02-01 (end of day).

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
| `/` | Home.jsx | ✅ Done | Landing page (currently routes to Directory for logged-in users — needs fix) |
| `/MyLane` | MyLane.jsx | ✅ Done | User community dashboard — Phase 1 complete |
| `/Events` | Events.jsx | ✅ Done | Browse events with filters |
| `/Directory` | Directory.jsx | ✅ Done | Business listings with search |
| `/Search` | Search.jsx | ✅ Done | Full-text search with sort options |
| `/BusinessProfile/:id` | BusinessProfile.jsx | ✅ Done | Public business page |
| `/CategoryPage/:slug` | CategoryPage.jsx | ✅ Done | Category browse page |
| `/PunchPass` | PunchPass.jsx | ✅ Done | Punch Pass balance and management (demo mode) |
| `/CheckIn` | CheckIn.jsx | ✅ Done | Event check-in with PIN validation |
| `/Recommend/:id` | Recommend.jsx | ✅ Done | Nod/Story submission form |
| `/Settings` | Settings.jsx | ✅ Done | User profile settings (name, phone, community, account info) |

### Business Dashboard

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/business-dashboard` | BusinessDashboard.jsx | ✅ Done | Multi-business selector + widget layout |
| (selected business) | BusinessDashboardDetail.jsx | ✅ Done | Profile editing, locations, tier upgrade |

### Admin Panel

| Route | Component | Status | Notes |
|-------|-----------|--------|-------|
| `/Admin` | Admin.jsx | ✅ Done | Steward admin panel |
| `/Admin/businesses` | AdminBusinessTable | ✅ Done | Business management |
| `/Admin/concerns` | AdminConcernsPanel | ✅ Done | Private concern review |
| `/Admin/users` | AdminUsersSection | ✅ Done | User list, detail drawer, admin controls |
| `/Admin/settings` | AdminSettingsPanel | ✅ Done | General platform settings |
| `/Admin/events/types` | ConfigSection | ✅ Done | Event type management |
| `/Admin/events/age-groups` | ConfigSection | ✅ Done | Age group management |
| `/Admin/events/durations` | ConfigSection | ✅ Done | Duration preset management |
| `/Admin/events/accessibility` | ConfigSection | ✅ Done | Accessibility features management |
| `/Admin/networks` | ConfigSection | ✅ Done | Network configuration |
| `/Admin/tiers` | (placeholder) | ⏳ Planned | Tier configuration |
| `/Admin/punch-pass` | (placeholder) | ⏳ Planned | Punch Pass settings |

---

## MyLane (User Dashboard)

Built 2026-02-01. Progressive depth architecture — sections show/hide based on user engagement level.

### Components

| Component | Status | Notes |
|-----------|--------|-------|
| MyLane.jsx | ✅ Done | Page shell, assembles sections |
| GreetingHeader.jsx | ✅ Done | Time-aware greeting, first name, Punch Pass badge |
| UpcomingEventsSection.jsx | ✅ Done | Shows RSVP'd future events, hides when empty |
| HappeningSoonSection.jsx | ✅ Done | Upcoming events with filter pills (All, This Weekend, Today, Free) |
| NewInCommunitySection.jsx | ✅ Done | Businesses < 30 days old with < 3 recommendations |
| YourRecommendationsSection.jsx | ✅ Done | User's Nods and Stories, hides when empty |
| DiscoverSection.jsx | ✅ Done | Category tiles with amber icons |
| SectionWrapper.jsx | ✅ Done | Shared section layout with title and "See all" link |

### Hooks

| Hook | Status | Notes |
|------|--------|-------|
| useUserState.js | ✅ Done | Determines Explorer/Engaged/Connected state |
| useRSVP.js | ✅ Done | RSVP status, attendee count, going/cancel mutations |

---

## RSVP System

Built 2026-02-01. Full RSVP lifecycle for events.

| Feature | Status | Notes |
|---------|--------|-------|
| RSVP entity | ✅ Done | Base44 entity: event_id, user_id, user_name, status, is_active, created_date |
| useRSVP hook | ✅ Done | Fetch RSVP status, attendee count/names, going/cancel mutations |
| EventDetailModal RSVP button | ✅ Done | "I'm Going" / "You're Going — Tap to Cancel" / sign-in prompt / past event state |
| RSVP celebration + auto-close | ✅ Done | "You're in! See you there." (1.2s) then modal closes |
| Cache invalidation | ✅ Done | Invalidates modal queries + MyLane queries on RSVP change |
| UpcomingEventsSection | ✅ Done | Queries user RSVPs, matches to events, shows in MyLane |

---

## User Settings

Built 2026-02-01. Self-service profile management.

| Feature | Status | Notes |
|---------|--------|-------|
| Display Name | ✅ Done | Editable, shows in greeting and across platform |
| Email | ✅ Done | Read-only with lock icon |
| Phone | ✅ Done | Optional |
| Home Community | ✅ Done | Dropdown — Greater Eugene active, others "Coming Soon" |
| Member Since | ✅ Done | Display only, formatted as Month Year |
| Account Type | ✅ Done | Display only, shows "Free" (tier from user.data) |
| Profile Photo | ⏳ Stub | Shows initials circle with "coming soon" text |
| Save Changes | ✅ Done | Saves to user.data, success/error toasts |

---

## Admin User Management

Built 2026-02-01. Replaces "coming soon" placeholder.

| Feature | Status | Notes |
|---------|--------|-------|
| User list table | ✅ Done | Name, Email, Joined, Status columns |
| Search | ✅ Done | Client-side filter by name or email |
| User detail drawer | ✅ Done | Profile info, activity counts, linked businesses |
| Admin controls | ✅ Done | Account status, user tier, admin notes |
| Activity counts | ✅ Done | Recommendations, RSVPs, Punch Pass balance |
| Linked businesses | ✅ Done | Shows owned + staff businesses |

---

## Recommendation System (Trust Architecture)

| Feature | Status | Notes |
|---------|--------|-------|
| Nods (quick endorsements) | ✅ Done | One-tap recommendation |
| Stories (detailed narratives) | ✅ Done | Full recommendation with text |
| Vouches (weighted trust) | ✅ Done | Verified endorsements |
| Private Concerns | ✅ Done | Routes to admin, not public |
| AdminConcernsPanel | ✅ Done | Review and manage concerns |

---

## Event Management

| Feature | Status | Notes |
|---------|--------|-------|
| Create event | ✅ Done | Full form with all fields via EventEditor |
| Edit event | ✅ Done | Opens EventEditor with existing data |
| Delete event | ✅ Done | Soft delete with confirmation |
| Cancel event | ✅ Done | Sets status: 'cancelled' with confirmation |
| Duplicate event | ✅ Done | Creates copy with "(Copy)" suffix |
| Recurring events | ✅ Done | Weekly/biweekly with day-of-week selection |
| Event images | ✅ Done | Image upload in EventEditor |
| Punch Pass pricing | ✅ Done | Toggle + punch cost (demo mode) |
| Multi-ticket pricing | ✅ Done | Tiered ticket types |
| Accessibility features | ✅ Done | Checkbox list from admin config |
| Event list with actions | ✅ Done | Desktop table + mobile cards |
| RSVP system | ✅ Done | Full lifecycle with celebration UX |

---

## Staff Management

| Feature | Status | Notes |
|---------|--------|-------|
| Add existing users | ✅ Done | Search by email, add to business |
| Invite new users | ✅ Done | Email invite for users not on platform |
| Assign roles | ✅ Done | Manager, Instructor, Staff |
| Manage pending invites | ✅ Done | View/cancel pending invitations |
| Auto-linking | ✅ Done | Invited users auto-connect on dashboard visit |
| Role badges | ✅ Done | Color-coded: Owner, Manager, Instructor, Staff |

---

## Known Issues

| Issue | Severity | Notes |
|-------|----------|-------|
| Landing page routes to Directory instead of MyLane for logged-in users | Medium | Routing fix needed |
| Browser autofill flashes white on dark inputs | Low | CSS autofill override needed |
| Test user (doron.bsg+test) has no full_name in published environment | Low | Data issue — fix via Settings page |
| Published vs Preview data discrepancy | Low | Different environments may have different data |

---

*Updated 2026-02-01 — MyLane Phase 1, RSVP system, User Settings, Admin Users shipped.*
