# Archetype: Event Coordinator

**Created:** 2026-01-27  
**Status:** In Progress  
**Owner:** Doron

---

## Overview

### What Is This Archetype?

An **Event Coordinator** is any individual or organization that hosts events for the community. This could be a professional event planner, a homeschool co-op organizing field trips, a business hosting workshops, or a community group running meetups.

This is the first archetype built for LocalLane and serves as the template for all future archetypes.

### Example Businesses

- Homeschool group organizing weekly park days
- Jiu-jitsu school hosting open mat sessions
- Church running community dinners
- Local business hosting workshops
- Community center with regular programming

### User Personas

**Event Coordinator (Business Owner):**
- Wants to reach more local families
- Currently uses Facebook events (gets buried, hard to find)
- Needs easy way to post events and track RSVPs
- May want to accept punch passes for attendance

**Community Member (Customer):**
- Parent looking for activities for kids
- Wants one place to see all local events
- Needs to filter by interest (sports, education, etc.)
- May have punch pass to use

---

## Data Model

### Core Fields (All Tiers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Auto | Unique identifier |
| title | string | Yes | Event name |
| description | text | Yes | Full event description |
| date | datetime | Yes | Start date/time |
| endDate | datetime | No | End date/time (if different from start) |
| timezone | string | Yes | Timezone (default: America/Los_Angeles) |
| location | string | Yes | Venue name |
| address | string | No | Full address |
| isVirtual | boolean | No | Is this online? |
| virtualLink | string | No | Link if virtual |
| capacity | number | No | Max attendees (null = unlimited) |
| networks | array | No | Networks this event belongs to |
| categories | array | No | Categories (sports, education, etc.) |
| imageUrl | string | No | Event banner/image |
| status | enum | Yes | draft, published, cancelled |
| createdAt | datetime | Auto | When created |
| updatedAt | datetime | Auto | Last update |

### Tier 2 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| acceptsPunchPass | boolean | Can attendees use punch passes? |
| punchPassTypes | array | Which pass types accepted |
| punchCost | number | How many punches to attend |
| requiresRSVP | boolean | Must RSVP to attend? |
| rsvpDeadline | datetime | When RSVPs close |
| waitlistEnabled | boolean | Allow waitlist if full? |

### Tier 3 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| customBranding | object | Logo, colors, header image |
| recurringPattern | object | For repeating events |
| ticketTypes | array | Multiple ticket/attendance types |
| customFields | array | Additional registration questions |

---

## Features by Tier

### Tier 1 (Free)

| Feature | Description |
|---------|-------------|
| Basic event posting | Create and publish events |
| Appears in Community Node | Events show in aggregated calendar |
| Basic RSVP tracking | See who's interested |
| Network tagging | Tag with Recess, Homeschool, etc. |
| Category tagging | Tag with Sports, Education, etc. |

### Tier 2 (Pro)

| Feature | Description |
|---------|-------------|
| Everything in Tier 1 | |
| Accept punch passes | Attendees can "pay" with punches |
| Advanced RSVP | Waitlists, deadlines, capacity limits |
| Analytics | View event performance metrics |
| Recurring events | Set up repeating schedules |
| Event reminders | Automated reminders to RSVPs |

### Tier 3 (Partner Node)

| Feature | Description |
|---------|-------------|
| Everything in Tier 2 | |
| Own Event Node | Dedicated application |
| Custom branding | Own logo, colors, domain (future) |
| Priority in Community Node | Higher placement in calendar |
| Trust badge | "Verified Event Partner" |
| Custom registration fields | Collect additional info |
| Multiple ticket types | Different attendance options |
| Full attendee management | Check-in, communications |

---

## User Workflows

### Workflow 1: Create and Publish Event (Coordinator)

**Actor:** Event Coordinator (any tier)

**Goal:** Post an event that appears in the Community Node

**Steps:**
1. Log into Event Node (or Community Node if Tier 1/2)
2. Click "Create Event"
3. Fill in required fields (title, date, location, description)
4. Select networks (Recess, Homeschool, etc.) — pulled from Community Node config
5. Select categories (Sports, Education, etc.)
6. [Tier 2+] Configure punch pass acceptance
7. Preview event
8. Click "Publish"
9. Event syncs to Community Node automatically

**Success criteria:** Event appears in Community Node calendar within 30 seconds

---

### Workflow 2: RSVP to Event (Community Member)

**Actor:** Community Member

**Goal:** Sign up to attend an event

**Steps:**
1. Browse events in Community Node
2. Filter by network, category, or date
3. Click on event to view details
4. Click "RSVP" button
5. [If not logged in] Prompted to log in or create account
6. [If Tier 2+ event with punch pass] Choose payment method (punch pass or regular RSVP)
7. Confirm RSVP
8. See confirmation with event added to "My Events"

**Success criteria:** 
- User sees confirmation
- Coordinator sees +1 RSVP count
- [If punch pass] Balance deducted

---

### Workflow 3: Redeem Punch Pass at Event (Community Member)

**Actor:** Community Member with punch pass

**Goal:** Use punch pass to attend event

**Steps:**
1. Arrive at event
2. Open Community Node app, go to "My Passes"
3. Select the relevant punch pass
4. Show QR code to coordinator
5. Coordinator scans code (or enters code manually)
6. System verifies pass is valid and has sufficient punches
7. Punches deducted
8. Both see confirmation

**Success criteria:**
- Pass balance reduced by correct amount
- Transaction logged
- Coordinator sees attendee checked in

---

### Workflow 4: View Event Analytics (Coordinator - Tier 2+)

**Actor:** Event Coordinator (Tier 2 or 3)

**Goal:** Understand event performance

**Steps:**
1. Log into Event Node
2. Go to "My Events"
3. Click on past or current event
4. Click "Analytics" tab
5. View: RSVP count, attendance, punch pass usage, traffic sources

**Success criteria:** Coordinator can see actionable metrics

---

## Integration Points

### Data Pushed to Community Node

| Data | Trigger | Endpoint |
|------|---------|----------|
| Event (create) | Coordinator publishes | POST /api/v1/events |
| Event (update) | Coordinator edits | PUT /api/v1/events/:id |
| Event (delete) | Coordinator cancels | DELETE /api/v1/events/:id |
| RSVP count | User RSVPs | POST /api/v1/events/:id/rsvp |

### Data Pulled from Community Node

| Data | Frequency | Endpoint |
|------|-----------|----------|
| Config (networks, categories) | On startup + every 5 min | GET /api/v1/config |
| Punch pass types | With config | GET /api/v1/config |

### Punch Pass Integration

- [x] This archetype can accept punch passes (Tier 2+)
- Accepted pass types: Configurable per event (from Community Node config)
- Typical punch cost: 1-3 punches per event (coordinator sets)

---

## UI/UX Considerations

### Key Screens

1. **Event List** — All events this coordinator has created
2. **Event Creation Form** — Multi-step form to create event
3. **Event Detail View** — Full event info + RSVPs
4. **Analytics Dashboard** — Event metrics (Tier 2+)
5. **Attendee List** — Who's coming, check-in status (Tier 2+)

### Mobile Considerations

- Event creation should work on mobile (coordinators may be on-the-go)
- QR code scanning for punch pass must work smoothly
- RSVP flow must be fast (1-2 taps)

### Accessibility Considerations

- Event descriptions support screen readers
- Date/time in accessible format
- QR codes have text fallback (manual code entry)

---

## Technical Notes

### Special Requirements

- Real-time sync to Community Node (events should appear quickly)
- Offline capability for viewing events (graceful degradation)
- QR code generation for punch pass redemption

### Known Challenges

- Time zones: Events may have attendees in different zones
- Recurring events: Complex to model, keep simple for Phase 1
- Capacity management: Race conditions when event fills up

### Dependencies

- Community Node must be operational for sync
- Punch pass system in Community Node (for Tier 2+ features)
- User authentication (from Community Node or Base44)

---

## Current Implementation Status

> **Status Note (2026-02-11):** This spec was written 2026-01-27 based on the Event Node as a Tier 3 Partner Node syncing to Community Node. Since then, Community Node has achieved full EventEditor feature parity (DEC-019), migrated from Punch Pass to Joy Coins (DEC-028, DEC-041), and built the complete RSVP and check-in system. A full reassessment is needed to determine what Event Node offers beyond Community Node's current capabilities, and whether Event Node should be repositioned as a standalone lab node under the Node Lab Model. See NODE-PLAYBOOK.md (private repo).

### What Exists

- [x] Basic event creation
- [x] Event list view
- [x] Sync to Community Node
- [x] Network tagging
- [x] Punch pass integration (basic)
- [x] UI/Layout (reported as "really nice")

### What Needs Work

- [ ] Analytics dashboard
- [ ] Recurring events
- [ ] Advanced RSVP (waitlist, deadline)
- [ ] Check-in system
- [ ] QR code for punch pass

### What's Deferred to Later

- Custom registration fields (Tier 3)
- Multiple ticket types (Tier 3)
- Custom branding (Tier 3)

---

## Open Questions

| Question | Status | Answer |
|----------|--------|--------|
| Should coordinators see attendee contact info? | Open | Privacy consideration |
| How to handle event cancellation with punch passes? | Open | Refund policy needed |
| What analytics matter most to coordinators? | Open | Need user research |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-01-27 | Initial spec based on existing implementation | Claude |

---

*This is the spec for the Event Coordinator archetype. The Event Node is the Tier 3 implementation.*
