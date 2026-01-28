# Archetype: Nonprofit / Church / Charity

**Created:** 2026-01-27  
**Status:** Draft (Next to Build)  
**Owner:** Doron

---

## Overview

### What Is This Archetype?

A **Nonprofit/Church** is a community organization focused on member engagement, communication, and coordination rather than commercial transactions. This includes churches, charities, community groups, volunteer organizations, and similar entities.

This is the second archetype for LocalLane. It will validate that the archetype system is flexible enough to handle different types of organizations.

### Example Organizations

- Local church (Doron's church — test case)
- Community charity
- Volunteer fire department auxiliary
- Parent-teacher organization
- Neighborhood association
- Youth sports league (non-commercial)

### User Personas

**Organization Admin (Leader):**
- Pastor, director, or volunteer coordinator
- Wants to communicate with members without relying on texts/paper
- Needs to organize events, volunteers, announcements
- Currently using fragmented tools (texts, paper, Facebook)
- Not very tech-savvy, needs simple interface

**Member:**
- Church member, volunteer, or participant
- Wants to know what's happening
- Needs to sign up for events and volunteer opportunities
- May want to see member directory
- May have family members who also participate

---

## Data Model

### Core Fields (All Tiers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string | Auto | Unique identifier |
| name | string | Yes | Organization name |
| description | text | Yes | About this organization |
| type | enum | Yes | church, charity, community_group, volunteer_org, other |
| address | string | No | Physical location |
| phone | string | No | Contact phone |
| email | string | Yes | Contact email |
| website | string | No | Website URL |
| logoUrl | string | No | Organization logo |
| networks | array | No | Networks (TCA, Homeschool, etc.) |
| categories | array | No | Categories (religious, volunteer, etc.) |

### Tier 2 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| memberCount | number | Tracked member count |
| memberDirectory | boolean | Is directory enabled? |
| volunteerOpportunities | array | Active volunteer needs |
| announcementHistory | array | Past announcements |

### Tier 3 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| customBranding | object | Logo, colors, custom styling |
| memberGroups | array | Small groups, committees |
| donationLink | string | External donation page |
| customPages | array | Additional content pages |

---

## Features by Tier

### Tier 1 (Free)

| Feature | Description |
|---------|-------------|
| Basic listing | Appears in Community Node directory |
| Contact info | Phone, email, website, location |
| Public events | Post events (via Event system) |
| Basic description | About page visible to public |

### Tier 2 (Pro)

| Feature | Description |
|---------|-------------|
| Everything in Tier 1 | |
| Announcements | Post updates visible to members |
| Member directory | Opt-in list of members |
| Volunteer coordination | Post needs, track sign-ups |
| Event integration | Enhanced event features |
| Email notifications | Notify members of updates |

### Tier 3 (Partner Node)

| Feature | Description |
|---------|-------------|
| Everything in Tier 2 | |
| Own application | Dedicated Nonprofit Node |
| Custom branding | Own logo, colors |
| Member groups | Small groups, committees, teams |
| Full member management | Add/remove, roles, permissions |
| Attendance tracking | Track participation |
| Priority in Community Node | Higher in directory |
| Trust badge | "Verified Community Partner" |

---

## User Workflows

### Workflow 1: Post Announcement (Admin)

**Actor:** Organization Admin (Tier 2+)

**Goal:** Share an update with all members

**Steps:**
1. Log into Nonprofit Node (or Community Node if Tier 2)
2. Go to "Announcements"
3. Click "New Announcement"
4. Enter title and content
5. Choose visibility (all members, specific groups)
6. Choose notification method (in-app only, email too)
7. Click "Publish"
8. Members see announcement in their feed / receive email

**Success criteria:** Members can view announcement, engagement tracked

---

### Workflow 2: Volunteer Sign-Up (Member)

**Actor:** Member

**Goal:** Sign up to volunteer for an opportunity

**Steps:**
1. View organization page (in Community Node or Nonprofit Node)
2. See "Volunteer Opportunities" section
3. Browse open opportunities
4. Click "Sign Up" on one
5. See confirmation and details (when, where, what to bring)
6. Opportunity appears in "My Volunteer Commitments"
7. Admin sees updated sign-up list

**Success criteria:** 
- Member confirmed for opportunity
- Admin sees volunteer count +1

---

### Workflow 3: View Member Directory (Member)

**Actor:** Member (at organization with directory enabled)

**Goal:** Find contact info for another member

**Steps:**
1. Log into organization's page
2. Go to "Member Directory"
3. Search or browse members (only those who opted in)
4. View member's profile (name, contact info they chose to share)
5. Can message/contact them

**Success criteria:** Member finds who they're looking for

**Privacy considerations:**
- Members must opt-in to directory
- Members control what info is visible
- Must be logged in and a member to view

---

### Workflow 4: Create Event (Admin)

**Actor:** Organization Admin

**Goal:** Create an event for the organization

**Steps:**
1. Log into Nonprofit Node
2. Click "Create Event"
3. Fill in event details (leverages Event archetype)
4. Choose audience (public, members only, specific groups)
5. Publish
6. Event appears in organization calendar AND Community Node (if public)

**Success criteria:** Event visible to appropriate audience

---

## Integration Points

### Data Pushed to Community Node

| Data | Trigger | Endpoint |
|------|---------|----------|
| Organization profile | Admin updates | POST /api/v1/businesses |
| Public events | Event published | POST /api/v1/events |
| Volunteer opportunities | Opportunity posted | POST /api/v1/volunteer |

### Data Pulled from Community Node

| Data | Frequency | Endpoint |
|------|-----------|----------|
| Config (networks, categories) | On startup + every 5 min | GET /api/v1/config |
| User accounts | On login | Auth system |

### Punch Pass Integration

- [ ] This archetype typically does NOT use punch passes
- Exception: Some nonprofits might run paid programming
- Could be enabled per-organization if needed

---

## UI/UX Considerations

### Key Screens

1. **Organization Home** — Overview, recent announcements, upcoming events
2. **Announcements Feed** — All announcements, filterable
3. **Member Directory** — Searchable member list (opt-in)
4. **Volunteer Board** — Current opportunities
5. **Events Calendar** — Organization-specific events
6. **Admin Dashboard** — Member management, analytics

### Mobile Considerations

- Primary use case is mobile (checking announcements on phone)
- Push notifications for important announcements
- Quick volunteer sign-up flow

### Accessibility Considerations

- Older demographic may have accessibility needs
- Large text option
- High contrast mode
- Simple navigation

---

## Technical Notes

### Special Requirements

- Email notification system (for announcements)
- Privacy controls (directory opt-in)
- Group/committee management (Tier 3)

### Known Challenges

- Privacy: Members must control their own visibility
- Permissions: Different roles (admin, leader, member)
- Notifications: Don't spam, respect preferences

### Dependencies

- Community Node for config sync
- Event system (shared with Event archetype)
- User authentication

---

## Comparison: Nonprofit vs Event Archetype

| Aspect | Event Archetype | Nonprofit Archetype |
|--------|-----------------|---------------------|
| Primary purpose | Host events | Manage community |
| Members | Attendees (transient) | Members (ongoing) |
| Revenue model | Punch passes, tickets | Donations, dues |
| Key features | Event creation, RSVPs | Announcements, directory |
| Data sensitivity | Low | Higher (member info) |

---

## Test Case: Doron's Church

### Current State
- Communication via paper, texts, conversations
- No central system
- Hard to know what's happening
- Volunteer coordination is ad-hoc

### Target State with Nonprofit Node
- Announcements visible to all members
- Event calendar everyone can access
- Volunteer opportunities posted, sign-ups tracked
- Optional member directory
- Admin can see engagement

### Success Metrics for Pilot
- [ ] Admin can post announcement in < 2 minutes
- [ ] Members can see announcements on their phone
- [ ] At least one volunteer opportunity posted and filled
- [ ] 50%+ of active members using the system

---

## Open Questions

| Question | Status | Answer |
|----------|--------|--------|
| Should members pay dues through the platform? | Open | Probably out of scope for Phase 1 |
| How to handle family accounts (parent + kids)? | Open | Need to design |
| What member info is appropriate to share? | Open | Privacy policy needed |
| Should there be "groups" in Tier 2 or only Tier 3? | Open | Simplicity vs. utility |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-01-27 | Initial draft | Claude |

---

*This archetype spec is drafted and ready for implementation after Event Node is stable.*
