# Community Organization Engine — Archetype Spec

**Created:** 2026-01-27
**Updated:** 2026-02-14
**Status:** Phase 0 — Research + Spec (seedling tray)
**Owner:** Doron
**Real User:** River Road Baptist Church (House of Worship archetype)
**Repo:** TBD (will be standalone Base44 app per Node Lab Model)

---

## Overview

### What Is This?

The Community Organization Engine is a standalone node serving mission-driven community organizations — churches, nonprofits, mutual aid networks, youth programs, and community groups. It follows the same Node Lab Model (DEC-047) as the Field Service Engine: built independently, field tested with real users, merged into LocalLane when mature.

The core insight: what Contractor Daily is to field service businesses, this engine is to community organizations. Same permission-based role system (DEC-ROLE-001), same archetype vocabulary configuration (DEC-FSE-003), same fractal pattern — different domain.

### How It Fits the Organism

Community organizations are among the most vital organs in the Organism. Their activity patterns — volunteer mobilization, food distribution, youth engagement, spiritual care — are direct indicators of community health. Eugene-Springfield has 2,443 registered nonprofits employing 17,711 people with $2 billion in annual revenue. This is not a niche — it is the circulatory system itself.

When this node matures and merges into LocalLane:
- A family discovers Burrito Brigade through the Harvest Network
- A church member signs up to volunteer at FOOD for Lane County's Youth Farm
- A youth at Ophelia's Place finds a summer program through the Gathering Circle
- The Organism visualizes these connections as living circulation

LocalLane doesn't create these connections — it makes them visible. Burrito Brigade's 55 Little Free Pantry locations already ARE the Organism. This node gives them a digital nervous system.

### The Shared Archetype: Mission-Driven Community Organization (MDCO)

All organization types share common DNA:

| Characteristic | Church | Nonprofit | Mutual Aid | Community Org |
|----------------|--------|-----------|------------|---------------|
| Primary currency | Relationships + spiritual care | Impact + outcomes | Aid + solidarity | Connection + belonging |
| Membership model | Congregants, visitors, members | Donors, beneficiaries, volunteers | Neighbors, participants | Residents, participants |
| Governance | Elders/board + pastoral | Board + executive director | Flat/consensus | Steering committee |
| Revenue | Tithes, offerings, donations | Grants, donations, fees | Mutual contributions | Dues, grants, sponsorships |
| Volunteer criticality | Very high | High | Very high | Very high |
| Care model | Prayer, pastoral care, benevolence | Client services, case management | Mutual aid, neighbor care | Neighbor care |
| Recurring gatherings | Weekly worship, small groups | Board meetings, programs | Regular meetups | Regular meetups, programs |

The universal elements (people, groups, events, communication, roles) are built once. The configurable elements (terminology, care workflows, governance formality, group types) adapt per sub-type.

---

## Sub-Types

| Sub-Type | Examples | Primary Needs | Test Case |
|----------|----------|---------------|-----------|
| **Houses of Worship** | Churches, temples, mosques | Announcements, member directory, event calendar, volunteer coordination, pastoral care | River Road Baptist Church |
| **Food Security & Mutual Aid** | Food pantries, meal programs, food rescue | Volunteer scheduling, distribution mapping, cross-referral | Burrito Brigade |
| **Youth Programs** | After-school, mentoring, empowerment | Program enrollment, parent communication, attendance | Ophelia's Place |
| **Environmental** | Recyclers, land trusts, watershed groups | Volunteer events, education programs | BRING Recycling |
| **Community Infrastructure** | Tool libraries, tiny home villages, warming centers | Resource management, volunteer shifts | ToolBox Project |
| **Social Services** | Clinics, housing providers, crisis response | Service listing, referral networks, hours/location | White Bird Clinic |
| **Arts & Culture** | Mural projects, cultural orgs | Event promotion, program listings | 20x21 Mural Project |
| **Advocacy & Civil Rights** | Community organizing, identity support | Event coordination, resource directories | CALC, NAACP Eugene |

---

## User Personas

### Organization Admin (Leader)

Pastor, executive director, volunteer coordinator, board member. Wants to communicate with members and community without relying on texts, paper, or Facebook. Needs to organize events, volunteers, announcements, programs. Currently using fragmented tools. Tech comfort varies widely — interface must be simple.

**The "one thing":** Post an announcement and know people saw it.

### Member / Volunteer

Church member, volunteer, participant, donor. Wants to know what's happening and how to help. Needs to sign up for events and volunteer opportunities. May have family members who also participate.

**The "one thing":** See what's happening this week and sign up.

### Community Beneficiary

Person served by the organization (food pantry client, youth program participant). May interact through LocalLane discovery without formal membership. Privacy is paramount — no tracking of service usage without explicit consent.

**The "one thing":** Find help without being tracked.

---

## Role System

Follows the permission-based architecture from DEC-ROLE-001. Roles are permission sets, not hardcoded enforcement keys.

### Role Hierarchy

| Role | Description | Typical Users |
|------|-------------|---------------|
| platform_admin | Platform operator (Doron). Sees all organizations. Email match, not role entry. | LocalLane admin |
| org_admin | Full access within the organization. Manages settings, members, all content. | Pastor, executive director, IT lead |
| staff | Broad access. Can manage most modules. Cannot change org settings. | Associate pastor, program director |
| leader | Manages specific group(s) and their members/events/volunteers. | Ministry director, team lead, small group leader |
| volunteer | Active volunteer. Can see schedule, check in, view assigned tasks. | Serving team members |
| member | Can view directory, register for events, access resources, submit requests. | Engaged members |
| visitor | Limited access. Can update own profile, see public events. | New visitors, community browsers |

### Permission Template (House of Worship)

```
org_admin:
  can_manage_settings: true
  can_manage_members: true
  can_manage_all_groups: true
  can_manage_all_events: true
  can_view_all_care_cases: true
  can_manage_volunteers: true
  can_send_announcements: true
  can_view_analytics: true
  can_view_directory_full: true

staff:
  can_manage_members: false
  can_manage_all_groups: true
  can_manage_all_events: true
  can_view_assigned_care_cases: true
  can_manage_volunteers: true
  can_send_announcements: true
  can_view_analytics: true
  can_view_directory_full: true

leader:
  can_manage_own_group: true
  can_manage_group_events: true
  can_view_group_care_cases: true
  can_manage_group_volunteers: true
  can_send_group_announcements: true
  can_view_directory_contact: true

volunteer:
  can_view_own_schedule: true
  can_check_in: true
  can_view_directory_basic: true

member:
  can_register_events: true
  can_submit_care_request: true
  can_view_directory_basic: true
  can_view_announcements: true

visitor:
  can_view_public_events: true
  can_update_own_profile: true
```

### Contextual Scoping

Permissions can be scoped to:
- Organization-wide (applies everywhere)
- Group-specific (limited to a ministry, committee, or team)
- Event-specific (e.g., event coordinator for one event)

Example: "Sarah has can_manage_group_volunteers scoped to the Hospitality Ministry group."

### Data Sensitivity Tiers

| Tier | Visible Fields | Who Can Access |
|------|----------------|----------------|
| Tier 1 (Public) | Name, photo, ministry involvement | All members |
| Tier 2 (Contact) | + Email, phone | Staff, leaders, by consent |
| Tier 3 (Personal) | + Address, birth date, household | Staff only, by consent |
| Tier 4 (Sensitive) | + Care cases, giving history | Authorized roles only, audit logged |

Every access to Tier 3-4 data is logged. This is non-negotiable for organizational trust.

### Vocabulary Configuration (Per Sub-Type)

| Universal Role | Church | Food Security | Youth Program |
|---------------|--------|---------------|---------------|
| org_admin | Pastor / Elder | Executive Director | Program Director |
| staff | Associate Pastor | Program Manager | Coordinator |
| leader | Ministry Director / Deacon | Site Lead / Shift Captain | Group Facilitator |
| volunteer | Serving Team Member | Star Supporter / Cook / Driver | Mentor / Tutor |
| member | Church Member | Community Supporter / Donor | Parent / Guardian |
| visitor | Visitor / Guest | Community Browser | Prospective Family |

Same permission skeleton. Different labels. One JSON config per sub-type.

---

## Core Modules

### Module 1: People & Households (Always On)

Central registry of individuals and households. The hub entity — everything else connects back to people.

**Features:**
- Individual profiles with contact info, photo, communication preferences
- Household grouping with relationships (head, spouse, child, other)
- Membership status tracking with configurable journey stages
- Privacy controls — members control their own visibility per field
- Communication preferences and consent tracking (email, SMS, push)
- Minor flagging with guardian linkage
- Merge/deduplication tools (admin only)
- Visitor-to-member journey tracking

**Privacy Model:**
- Default: users see only Tier 1 data
- Each member opts in to what they share and with whom
- Directory visibility is always opt-in, never default
- Service beneficiaries are NEVER tracked without explicit consent
- Bulk export requires elevated permissions and is audit-logged

**Church-specific:** Membership classes, baptism/confirmation dates (opt-in custom fields)

**Food Security-specific:** Beneficiary records completely separate from volunteer/donor records. No cross-linking.

### Module 2: Groups & Teams (Always On)

Organize people into functional and relational units.

**Features:**
- Configurable group types (ministry team, small group, committee, program, cohort)
- Leadership roles per group (leader, co-leader, admin)
- Membership management with join/leave tracking
- Group-scoped communication (announcements within group)
- Group calendar linked to org calendar
- Subgroups and hierarchy (up to 3 levels: Ministry > Team > Sub-team)
- Public vs. private groups
- Seasons/terms for time-bounded groups (fall semester, program year)

**Church-specific:** Small groups, ministries, Bible studies, worship teams, elder board
**Food Security-specific:** Volunteer shifts by program (Weekend Brigade, Pantry Hosts, Delivery Drivers)
**Youth-specific:** Age-based cohorts, program sessions, after-school groups

### Module 3: Events & Calendar (Always On)

Manage all organizational activities and gatherings.

**Features:**
- Event creation with rich details (title, description, date/time, location, capacity)
- Recurrence patterns (weekly service, monthly board meeting, annual retreat)
- Registration (optional, with configurable fields per event type)
- Capacity management with waitlist
- RSVP tracking
- Personal calendar sync (Google, Outlook, Apple — future integration)
- Public vs. member-only vs. invited-only events
- Event types with templates (worship service auto-fills differently than potluck)
- Multi-session events (VBS, course series, multi-day programs)
- Location/room booking (for organizations with multiple spaces)

**LocalLane Integration:** Public events push to Community Node calendar. Members discover org events through LocalLane directory. Joy Coins accepted for paid programming (workshops, classes).

### Module 4: Announcements & Communication (Always On)

Multi-channel messaging to individuals and groups.

**Features:**
- Announcement posting with title, content, visibility scope
- Audience targeting (all members, specific groups, event registrants)
- Notification method selection (in-app, email, both)
- Template library for recurring communications
- Consent enforcement (never email someone who opted out)
- Frequency governance (configurable max per week)
- Read tracking (aggregate, not individual surveillance)
- Urgent flag for time-sensitive announcements

**Future:** SMS via Twilio, push notifications. Not MVP.

**Church-specific:** Weekly bulletin, prayer chain distribution, sermon follow-up
**Food Security-specific:** Volunteer shift reminders, pantry restock alerts

### Module 5: Volunteer Management (Standard Tier)

Full volunteer lifecycle from interest to appreciation.

**Features:**
- Volunteer interest capture (sign-up form)
- Role-based opportunities (typed roles per group: Greeter, Sound Tech, Nursery, Cook, Driver)
- Availability capture (weekly matrix or per-event)
- Schedule creation (manual, with auto-suggest future)
- Shift management with open/filled/confirmed status
- Swap and substitute requests between volunteers
- Check-in for serving (self or coordinator confirms)
- Hours tracking (automatic from check-in)
- Appreciation automation (milestone celebrations: "John served 10 times!")
- Offboarding with reason tracking (sabbatical, stepping down)

**Screening (configurable per role):**
- Application form (configurable fields)
- Background check tracking (manual entry for MVP; integration with Checkr/Protect My Ministry future)
- Training tracking with expiration dates and renewal alerts
- Approval workflow (leader approves before volunteer is scheduled)

**Child Safety (non-negotiable for child-involved roles):**
- Background check required flag
- Training requirements (Safe Church, first aid)
- Two-adult rule enforcement in scheduling (system warns if only one adult assigned)
- Training expiration blocks scheduling until renewed

### Module 6: Care & Prayer (Standard Tier)

Manage pastoral care, prayer requests, and case-based support with appropriate confidentiality.

**This is the killer feature.** Churches genuinely need this and cannot get it from Facebook groups or generic tools. The confidentiality model is what makes it trustworthy.

**Features:**
- Multi-channel intake (web form, app, staff entry, during service)
- Configurable case types (prayer request, pastoral care, benevolence, crisis, mutual aid)
- Confidentiality tiers with strict enforcement
- Assignment and routing (auto-assign to care team or manual pastoral assignment)
- Follow-up workflows with reminder cadence per case type
- Status tracking (new, in progress, follow-up, closed)
- Outcome tracking (resolved, ongoing, referred, closed-unresolved)
- "Praying" button for prayer team members (anonymous count visible to requester)
- Encouragement messaging (prayer partner can send note to requester)
- Privacy-preserving reporting (aggregate only: "47 requests this month, 89% followed up within 48 hours")
- Related resource linking (connect to benevolence fund, community resources, referrals)

**Confidentiality Tiers:**

| Tier | Label | Who Sees | Example |
|------|-------|----------|---------|
| 0 | SEALED | Senior pastor / designated only | Abuse disclosures, legal matters |
| 1 | PASTORAL | Pastoral staff only | Marital issues, mental health, confessions |
| 2 | CARE TEAM | Trained care team | Illness, grief, general struggles |
| 3 | PRAYER CHAIN | Opted-in prayer team | General prayer requests |
| 4 | PUBLIC | Bulletin, website | Celebrations, general needs |

**Sanitization workflow:** Pastor receives Tier 1 request with sensitive details. Creates a separate Tier 3 version with consent ("Please pray for Jane as she faces health challenges") linked to the original (link visible only to pastoral staff). Prayer team sees sanitized version only.

**Audit logging:** Every view of a care case is logged with viewer, timestamp, and case ID. Non-negotiable. This is what builds trust with congregants who share vulnerable information.

**Non-church sub-types:**
- Food Security: "Need" requests from community (food, supplies) — Tier 2 default, staff-only
- Youth: Parent concerns, behavioral notes — Tier 1 default, program director only
- Social Services: Client case management — Tier 0-1, strict access controls

### Module 7: Check-In & Attendance (Standard Tier)

Track attendance at events and ensure child safety.

**Features:**
- Self-check-in (QR code, name lookup)
- Coordinator check-in (search and confirm)
- Headcount tracking for services/events
- Attendance history per person
- Room rosters for multi-room events

**Child Safety Check-In (for child-involved events):**
- Parent/guardian check-in with authorized pickup list
- Secure pickup codes per household (e.g., "SMITH-7294")
- Name tag printing with allergy alerts and pickup code
- Guardian photo on coordinator tablet for pickup verification
- Check-out requires pickup code + visual verification
- Parent notification on checkout ("Sofia picked up by Grandma Elena at 12:05pm")
- Allergy/medical alerts visible on room roster (red flag icon, not full details unless tapped)

**This is non-negotiable for any organization serving minors.** The liability and safety implications make this a trust requirement, not a nice-to-have.

### Module 8: Meetings & Governance (Partner Tier)

Support formal and informal meetings with accountability.

**Features:**
- Meeting types with templates (board meeting, staff meeting, committee, 1:1)
- Agenda management (items with time allocation, presenters, attachments)
- Attendance tracking with quorum monitoring (configurable)
- Note-taking during meeting
- Action item extraction (linked to people with due dates)
- Minutes approval workflow (draft → chair review → approved → distributed)
- Voting and motions (optional, enable per meeting type)
- Confidentiality settings per meeting (normal, confidential, executive session)
- Recurring meeting support
- Outstanding action items auto-added to next meeting agenda

**Church-specific:** Elder/deacon board meetings, congregational meetings with voting
**Nonprofit-specific:** Board of directors meetings with formal minutes and quorum requirements
**Community org:** Steering committee meetings, informal with action items

---

## Pricing

Nonprofits receive a 50% discount on standard business tier pricing (DEC-040).

| Tier | Code | Monthly | Business Equivalent | What They Get |
|------|------|---------|---------------------|---------------|
| **Basic** | `basic` | Free | Free | Directory listing, contact info, public events (reviewed), sub-type tagging |
| **Nonprofit Standard** | `standard` | **$39/mo** | $79/mo | + Auto-publish events, announcements, member directory, volunteer coordination, program listings, email notifications, care/prayer module, check-in/attendance, Community Pass participation, Joy Coin acceptance, newsletter features, basic analytics |
| **Nonprofit Partner** | `partner` | **$79/mo** | $149/mo (earned) | + Own Partner Node app, custom branding, member groups/committees, full member management, meetings & governance module, multi-program dashboard, attendance tracking, priority listing, trust badge |

**Qualification:** 501(c)(3) EIN or Oregon nonprofit registration. Self-reported during onboarding, spot-checked if needed.

**Founding nonprofit deal:** First 10 nonprofits get 3-6 months free to prove platform value before any charge.

**Partner tier remains earned** — same criteria as business Partner (6+ months at Standard, engagement threshold, vouches, zero violations). Discount applies to fee, not qualification bar.

---

## Data Model

### Core Entities

#### OrganizationProfile

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | Unique identifier |
| owner_user_id | string | Yes | Base44 user ID of creator |
| name | string | Yes | Organization name |
| description | text | Yes | About this organization |
| sub_type | enum | Yes | house_of_worship, food_security, youth_program, environmental, community_infrastructure, social_services, arts_culture, advocacy |
| ein | string | No | 501(c)(3) EIN for discount verification |
| denomination | string | No | Religious denomination (house_of_worship only) |
| address | string | No | Physical location |
| phone | string | No | Contact phone |
| email | string | Yes | Contact email |
| website | string | No | Website URL |
| logo_url | string | No | Organization logo |
| networks | array | No | LocalLane networks (Recess, Creative Alliance, Harvest, Gathering Circle) |
| categories | array | No | Categories (religious, volunteer, food, youth, etc.) |
| service_area | string | No | Geographic area served |
| hours | object | No | Operating hours |
| subscription_tier | enum | Yes | basic, standard, partner |
| is_active | boolean | Yes | Active status |
| settings | object | No | Org-specific configuration (vocabulary, enabled modules, defaults) |
| hub_org_id | string | No | LocalLane Community Node business ID (future integration) |

#### Person

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | Which organization |
| user_id | string | No | Base44 user ID (null until account linked) |
| first_name | string | Yes | |
| last_name | string | Yes | |
| email | string | No | Unique per org |
| phone | string | No | |
| address | object | No | JSON (street, city, state, zip) |
| photo_url | string | No | |
| birth_date | date | No | |
| is_minor | boolean | Yes | Default false |
| membership_status | enum | Yes | visitor, regular, member, leader, inactive |
| membership_date | date | No | When became member |
| communication_preferences | object | No | JSON (email_opt_in, sms_opt_in, frequency) |
| privacy_settings | object | No | JSON (directory_visible, show_email, show_phone, show_address) |
| custom_fields | object | No | JSON for sub-type-specific data |
| hub_person_id | string | No | LocalLane user ID (future integration) |

#### Household

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| name | string | Yes | "The Smith Family" |
| primary_address | object | No | JSON |
| primary_phone | string | No | |

#### HouseholdMember

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| household_id | FK | Yes | |
| person_id | FK | Yes | |
| relationship_type | enum | Yes | head, spouse, child, other |
| is_primary_contact | boolean | Yes | |
| is_guardian | boolean | Yes | For minors — authorized for check-in/out |

#### Group

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| parent_group_id | FK | No | For hierarchy |
| group_type | enum | Yes | ministry, small_group, committee, program, team, cohort |
| name | string | Yes | |
| description | text | No | |
| visibility | enum | Yes | public, members, private |
| status | enum | Yes | active, inactive, archived |
| season | string | No | "Fall 2026", "Q1 2026" |
| settings | object | No | Type-specific settings |

#### GroupMembership

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| group_id | FK | Yes | |
| person_id | FK | Yes | |
| role | enum | Yes | member, leader, admin |
| status | enum | Yes | active, pending, inactive |
| joined_at | datetime | Yes | |
| left_at | datetime | No | |

#### Event

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| event_type | enum | Yes | worship_service, meeting, program_session, social, outreach, training, other |
| name | string | Yes | |
| description | text | No | |
| location | object | No | JSON (name, address, room) |
| start_datetime | datetime | Yes | |
| end_datetime | datetime | Yes | |
| recurrence_rule | string | No | RRULE string |
| parent_event_id | FK | No | For series |
| capacity | number | No | |
| registration_required | boolean | Yes | Default false |
| visibility | enum | Yes | public, members, invited |
| status | enum | Yes | draft, published, cancelled |
| group_id | FK | No | Owning group |
| check_in_mode | enum | Yes | none, simple, secure (secure = child safety) |
| accepts_joy_coins | boolean | No | For paid programming |
| joy_coin_cost | number | No | |

#### Announcement

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| author_id | FK | Yes | Person who posted |
| title | string | Yes | |
| content | text | Yes | Rich text |
| visibility | enum | Yes | all_members, group_specific, public |
| target_group_ids | array | No | If group_specific |
| notification_method | enum | Yes | in_app, email, both |
| is_urgent | boolean | Yes | Default false |
| published_at | datetime | No | |
| expires_at | datetime | No | Auto-archive |

#### CareCase

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| case_type | enum | Yes | prayer_request, pastoral_care, benevolence, crisis, mutual_aid |
| person_id | FK | Yes | Who the case is about |
| submitted_by_id | FK | No | Who submitted (null if self) |
| title | string | Yes | |
| description | text | Yes | Encrypted at rest |
| confidentiality_tier | integer | Yes | 0-4 |
| status | enum | Yes | new, in_progress, follow_up, closed |
| priority | enum | Yes | low, normal, high, urgent |
| assigned_to_id | FK | No | Person assigned |
| assigned_team_id | FK | No | Group assigned |
| follow_up_days | integer | No | Days between follow-ups |
| outcome | text | No | Resolution summary |
| closed_at | datetime | No | |
| sanitized_case_id | FK | No | Link to sanitized version (pastoral staff only) |

#### CareCaseNote

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| care_case_id | FK | Yes | |
| author_id | FK | Yes | |
| content | text | Yes | Encrypted at rest |
| is_internal | boolean | Yes | Not visible to requester if true |

#### VolunteerProfile

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| person_id | FK | Yes | |
| organization_id | FK | Yes | |
| status | enum | Yes | interested, screening, active, inactive, removed |
| availability | object | No | Weekly matrix JSON |
| skills | array | No | |
| background_check_status | enum | No | not_required, pending, passed, failed, expired |
| background_check_date | date | No | |
| background_check_expires | date | No | |

#### TrainingRecord

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| person_id | FK | Yes | |
| organization_id | FK | Yes | |
| training_type | string | Yes | "Safe Church", "First Aid", "Food Safety" |
| completed_at | date | Yes | |
| expires_at | date | No | |
| certificate_url | string | No | |

#### ScheduledShift

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| event_id | FK | Yes | |
| group_id | FK | Yes | The team |
| role_name | string | Yes | "Greeter", "Sound Tech", "Nursery" |
| person_id | FK | No | Assigned volunteer (null = open) |
| status | enum | Yes | open, filled, confirmed, completed, no_show |
| confirmed_at | datetime | No | |

#### EventRegistration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| event_id | FK | Yes | |
| person_id | FK | Yes | |
| household_id | FK | No | For household registrations |
| status | enum | Yes | registered, waitlisted, cancelled |
| registration_data | object | No | Form responses JSON |
| checked_in_at | datetime | No | |
| checked_out_at | datetime | No | |
| pickup_code | string | No | For secure child checkout |

#### Meeting (Partner Tier)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| meeting_type | enum | Yes | board, staff, committee, one_on_one |
| group_id | FK | Yes | Owning group |
| title | string | Yes | |
| scheduled_start | datetime | Yes | |
| scheduled_end | datetime | Yes | |
| location | object | No | JSON |
| video_link | string | No | |
| status | enum | Yes | scheduled, in_progress, completed, cancelled |
| confidentiality | enum | Yes | normal, confidential, executive_session |
| quorum_required | integer | No | |

#### ActionItem

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| meeting_id | FK | No | Source meeting |
| title | string | Yes | |
| description | text | No | |
| assigned_to_id | FK | Yes | |
| due_date | date | Yes | |
| status | enum | Yes | pending, in_progress, completed, cancelled |

#### AuditLog

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | UUID | Auto | |
| organization_id | FK | Yes | |
| user_id | FK | Yes | Who performed action |
| action | string | Yes | "viewed_care_case", "exported_directory", "updated_member" |
| entity_type | string | Yes | "CareCase", "Person", etc. |
| entity_id | string | Yes | |
| metadata | object | No | Additional context JSON |
| timestamp | datetime | Yes | |

---

## Sub-Type Specific Fields

These extend OrganizationProfile.settings or use dedicated entities:

### House of Worship

| Field | Type | Description |
|-------|------|-------------|
| denomination | string | Baptist, Methodist, Non-denominational, etc. |
| service_times | array | [{day: "Sunday", times: ["9:00 AM", "10:30 AM"]}] |
| pastoral_staff | array | Named pastoral roles |
| sacramental_records | boolean | Track baptism/confirmation/marriage (opt-in, denomination-specific) |
| sermon_archive_url | string | YouTube or podcast link |

### Food Security & Mutual Aid

| Field | Type | Description |
|-------|------|-------------|
| distribution_locations | array | Pantry/distribution point locations with addresses |
| distribution_schedule | object | When food is available |
| food_rescue_partners | array | Partner businesses |
| delivery_radius | string | Delivery area |
| volunteer_roles | array | Typed roles (driver, stocker, cook, sorter) |

### Youth Programs

| Field | Type | Description |
|-------|------|-------------|
| age_range | object | min_age, max_age |
| program_sessions | array | Named sessions with schedules |
| enrollment_open | boolean | Currently accepting |
| parent_communication | boolean | Parent notifications enabled |
| certifications | array | Org certifications held |

### Community Infrastructure

| Field | Type | Description |
|-------|------|-------------|
| resource_inventory | array | Available resources |
| checkout_enabled | boolean | Lending system active |
| capacity | number | People/units served |
| locations | array | Multiple addresses |

---

## Test Cases

### Test Case 1: River Road Baptist Church (House of Worship)

**Organization:** River Road Baptist Church
**Location:** 1105 River Road, Eugene, OR 97404
**Phone:** (541) 688-6071
**Website:** rrbceugene.org
**History:** 70+ years in Eugene
**Services:** Sunday School 9am, Worship 10am, Hispanic Sunday School 11am, Hispanic Worship 12pm
**Sermons:** YouTube channel

**Current State:**
- Communication via paper, texts, conversations. No central digital system.
- Volunteer coordination is ad-hoc
- No member directory accessible outside of church
- Hard to know what's happening between Sundays
- Hispanic ministry operates alongside English ministry — bilingual needs

**Target State:**
- Announcements visible to all members on their phones
- Event calendar everyone can access
- Volunteer opportunities posted, sign-ups tracked
- Optional member directory (opt-in)
- Prayer request submission with appropriate confidentiality
- Bilingual support for Hispanic ministry communications

**Success Metrics:**
- [ ] Admin can post announcement in < 2 minutes
- [ ] Members can see announcements on their phone
- [ ] At least one volunteer opportunity posted and filled
- [ ] 50%+ of active members using the system within 3 months
- [ ] Prayer requests submitted and followed up within 48 hours
- [ ] Hispanic ministry members can access same features

**Research Questions for Elder Board (Doron to conduct):**
1. What is RRBC's governance model? (Elder-led, pastor-led, congregational?)
2. How many active members / regular attenders?
3. How many distinct ministry areas? (Worship, Children, Youth, Outreach, etc.)
4. What's the current child protection policy? (Background checks, training, two-adult rule?)
5. Who handles pastoral care currently? How formalized?
6. How do you schedule volunteers? What breaks every week?
7. What tools do you use now? (Google apps, email platform, giving platform?)
8. What's the congregation's tech comfort level? Smartphone penetration?
9. What is the single biggest admin headache right now?
10. How does the Hispanic ministry coordinate with the English ministry?

### Test Case 2: Burrito Brigade (Food Security)

**Organization:** Burrito Brigade, 501(c)(3)
**EIN:** 47-3100942
**Location:** 1775 W 6th Ave, Eugene
**Executive Director:** Jennifer Denson
**Contact:** hello@burritobrigade.org, (541) 632-3239

**Current State:** Three programs (Weekend Burrito Brigade, Little Free Pantries, Waste to Taste), 55+ pantry locations, 300K+ meals served, volunteer-powered, average donation $5-10.

**Target State:**
- Volunteer coordination across three programs with role-based sign-ups
- Little Free Pantry locations visible on map (55+ points — this IS the Organism)
- Community awareness — neighbors discover nearby pantries
- Event scheduling for weekend production shifts
- Cross-referral connections to FOOD for Lane County, White Bird, Black Thistle

**Success Metrics:**
- [ ] Volunteers can sign up for specific roles (Star Supporter, Pantry Host, Weekend Cook)
- [ ] All 55+ pantry locations discoverable by neighborhood
- [ ] Weekend production shifts fill via platform sign-ups
- [ ] At least 3 cross-referral connections active

### Test Case 3: Ophelia's Place (Youth Program)

**Current State:** Prevention-based nonprofit for girl-identified youth ages 10-18. Programs across Lane & Linn counties. Facing federal funding cuts as of early 2026.

**Target State:**
- Program listings with enrollment info
- Event calendar for community events and fundraisers
- Parent communication channel
- Cross-referral to SASS, Trans*Ponder, Youth Era

---

## Implementation Phases

Following the Node Lab Model lifecycle:

### Phase 0: Research (Current — Doron conducting)

- [x] Market research complete (NONPROFIT-LANDSCAPE-EUGENE.md)
- [x] Spec drafted (this document)
- [ ] Interview RRBC elder board and pastor on pain points
- [ ] Study competing tools in the church management space
- [ ] Interview Burrito Brigade on volunteer coordination workflow
- [ ] Document findings in private repo

**Expert Research Library (to study before building):**

| Source | What to Learn |
|--------|--------------|
| Planning Center | Market leader in church management. What do they do well? What's missing? What's their pricing? |
| Breeze ChMS | Simpler alternative. Good UX reference for small churches. |
| Church Community Builder | Enterprise-level. Overkill for RRBC but shows what mature looks like. |
| Elvanto (Tithely) | Mid-range. Recently acquired by Tithely. |
| SignUpGenius | What people use when they don't have a real system — reveals the minimum viable feature set |
| Galaxy Digital | Volunteer management platform. Shows what mature volunteer scheduling looks like. |
| Salesforce Nonprofit Cloud | Enterprise case management. Too complex, but the care workflow patterns are instructive. |

### Phase 1: Core Build (After Phase 0 research validates direction)

**Scope:** Data model, core workflows, admin-only. Following BUILD-PROTOCOL.md.

- OrganizationProfile entity with owner_user_id (multi-tenant from day one)
- Person and Household entities
- Announcements (create, publish, view)
- Events (create, RSVP, calendar view)
- Basic member directory (opt-in)
- Admin dashboard (member count, recent activity)
- Gold Standard design system
- First-login setup wizard (mirrors FSE pattern)

**Done when:** Doron can create RRBC as an org, add members, post announcements, create events, and see a directory — all from the admin view.

### Phase 2: Multi-User Access

**Scope:** Role-based views, invite flow, Feedback Kit.

- Permission-based role system (roleTemplates.js with org_admin, staff, leader, member, visitor)
- Member view (see announcements, directory, events, register)
- Leader view (manage own group, see group members)
- Invite flow (email → role entry → auto-link on login)
- Feedback Kit (portable FeedbackLog entity)
- Invite RRBC pastor or key leader as field tester

**Done when:** A non-admin RRBC member can log in, see announcements, register for events, and view the directory. A leader can manage their group.

### Phase 3: Care & Volunteer Modules

**Scope:** The features that differentiate this from "just use a Facebook group."

- Volunteer management (opportunities, sign-ups, scheduling, check-in)
- Care/prayer request submission and workflow
- Confidentiality tiers with audit logging
- Follow-up reminders
- Background check and training tracking (manual entry)
- Child safety check-in for child-involved events

**Done when:** A church member can submit a prayer request, a pastor can triage and assign it, a prayer team member can see the sanitized version and tap "Praying." Volunteers can sign up for Sunday roles and check in.

### Phase 4: Iteration

**Scope:** Respond to field testing feedback from RRBC.

- Fix what's confusing
- Build what they asked for (not what we guessed)
- Polish, edge cases, mobile optimization
- Accessibility improvements for older demographics

### Phase 5: Maturity Assessment

- Is RRBC using it weekly?
- Is the data model stable?
- Has the role system survived real use?
- Are there other churches or orgs asking to use it?

### Phase 6: Integration Planning (Future)

When this node merges into LocalLane:

| Community Org Engine | LocalLane System |
|---------------------|-----------------|
| OrganizationProfile | Business entity (sub_type: nonprofit) |
| Person (member) | User entity with org membership |
| org_admin role | owner_user_id + staff_role |
| staff/leader/volunteer/member | staff_roles with permission flags |
| Events | Events system (already exists in Community Node) |
| Care Cases | New — unique to this node, stays within org |
| Volunteer scheduling | New — could become shared infrastructure |
| Announcements | Business Dashboard communications tab |
| Groups | New — could extend to business teams |

**What LocalLane gains:**
1. Care/prayer workflow (no equivalent exists)
2. Volunteer management (applicable beyond nonprofits)
3. Household model (useful for Community Pass family accounts)
4. Group/committee management
5. Meeting governance tools

**What this node gains from LocalLane:**
1. Tier system and billing infrastructure
2. Community Pass / Joy Coins for paid programming
3. Directory discovery (families find RRBC through LocalLane)
4. Cross-referral network (org-to-org connections visible)
5. Event promotion to broader community
6. Organism visualization of community health

---

## Cross-Referral Network

Nonprofits are already interconnected through informal referral networks. LocalLane makes these visible.

**Burrito Brigade's connections:**
- FOOD for Lane County (shared food rescue)
- White Bird Clinic (referral)
- Black Thistle Street Aid (referral)
- 86 Hunger (Sunday partnership)

**Ophelia's Place connections:**
- SASS (sexual assault support)
- Trans*Ponder (gender identity support)
- Youth Era (youth services)
- Looking Glass (youth services)

**Model:** OrganizationRelationship entity (mirrors BusinessRelationship from DEC-FSE-005)

| Field | Type | Description |
|-------|------|-------------|
| from_org_id | FK | Organization initiating |
| to_org_id | FK | Related organization |
| relationship_type | enum | referral_partner, shared_program, coalition |
| status | enum | active, pending, inactive |
| description | text | "We refer families needing food assistance" |

This is the nonprofit equivalent of the Supplier Super Node. Instead of materials flowing between businesses, care and services flow between organizations. Same circulation pattern.

---

## Organism Connection

**What a healthy community org organism looks like:**
- Active volunteer sign-ups and fulfilled shifts
- Regular events with community attendance
- Cross-referral connections lighting up between organizations
- Care cases being followed up within 48 hours
- Member engagement with announcements
- Geographic coverage (pantry network, service locations)

**What an unhealthy one looks like:**
- Unfilled volunteer shifts
- Events with low attendance
- No cross-referral connections
- Stale announcements
- Care cases going dormant

**Organism signals this node could emit:**
- volunteer_shifts_filled_ratio (health of volunteer mobilization)
- care_response_time_avg (speed of pastoral/community care)
- cross_referral_count (network density between organizations)
- event_attendance_trend (community gathering vitality)
- member_engagement_score (are people active or just listed?)

Burrito Brigade's 55 Little Free Pantries would light up across the Organism map as pantries are stocked and restocked — the real-time pulse of neighborhood food access. A church's weekly service attendance shows the rhythm of spiritual community. These are the heartbeats of the Organism.

---

## Decisions

### DEC-CO-001: Community Organization Engine as Node Lab Node

**Date:** 2026-02-14
**Decision:** Build as standalone Base44 app following Node Lab Model (DEC-047). Not as a module within Community Node. Independent until mature, then merge.
**Rationale:** Same reasoning as Field Service Engine. Independence forces quality. Real users (RRBC) generate real feedback. Permission system ports from FSE architecture.
**Status:** Active

### DEC-CO-002: Permission-Based Roles Port from FSE

**Date:** 2026-02-14
**Decision:** Use identical permission-based role architecture from DEC-ROLE-001. roleTemplates.js with default permission objects per role, hasPermission(flag) enforcement, per-person overrides.
**Rationale:** Fractal consistency. What works for contractor → owner → worker → client works for org_admin → staff → leader → member. Same skeleton, different vocabulary.
**Status:** Active

### DEC-CO-003: Care Confidentiality Tiers Are Non-Negotiable

**Date:** 2026-02-14
**Decision:** The 5-tier confidentiality model (Sealed → Pastoral → Care Team → Prayer Chain → Public) ships in Phase 3. Every care case view is audit logged. No exceptions, no shortcuts.
**Rationale:** This is the feature that builds trust. A pastor who shares sensitive prayer requests publicly destroys trust. A system that enforces confidentiality BY DEFAULT protects both the person sharing and the person caring. This is what makes the platform trustworthy enough for churches to actually use.
**Status:** Active

### DEC-CO-004: Child Safety Check-In Is Non-Negotiable

**Date:** 2026-02-14
**Decision:** Any event flagged as involving minors requires secure check-in mode: guardian verification, pickup codes, allergy alerts, audit trail. This is enforced by the system, not optional per event.
**Rationale:** Liability, safety, and trust. Churches that serve children without proper check-in systems face real legal and safety risks. The platform should make the safe path the easy path.
**Status:** Active

### DEC-CO-005: Cross-Referral as Circulation (Not Just Listing)

**Date:** 2026-02-14
**Decision:** OrganizationRelationship entity mirrors BusinessRelationship (DEC-FSE-005). Organizations connect to each other with typed relationships and permissions. This is the nonprofit equivalent of the Supplier Super Node — care and services circulating between organizations instead of materials between businesses.
**Rationale:** Eugene's nonprofits already cross-refer informally. Making these connections visible and functional is what makes LocalLane's Organism real for the nonprofit sector.
**Status:** Active — Design only, build when cross-referral UX is needed

---

## Open Questions

| Question | Status | Notes |
|----------|--------|-------|
| RRBC governance model? | Pending | Doron to ask elder board |
| RRBC congregation size? | Pending | Doron to research |
| Current child protection policy? | Pending | Critical for Phase 3 scope |
| Hispanic ministry coordination? | Pending | Bilingual UX implications |
| Family account / household model? | Open | Community Pass household model may apply |
| Volunteer hours for Earn-Your-Pass? | Open | Youth Farm connection |
| Giving integration scope? | Deferred | External for Phase 1-2 (Tithe.ly, Stripe) |
| Planning Center migration path? | Future | If RRBC or other churches use it |
| Real-time pantry stocking status? | Future | Simple check-in mechanism for hosts |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-01-27 | Initial draft — church-focused | Claude |
| 2026-02-07 | Major update — sub-types, nonprofit pricing, expanded data model, test cases, implementation phases, Organism connection | Claude + Doron |
| 2026-02-14 | Reconciled spec — merged Google Doc MDCO framework with existing nonprofit-node.md. Added: permission-based role system (aligned to FSE/DEC-ROLE-001), care confidentiality tiers, child safety check-in, meetings & governance module, household model, volunteer lifecycle, audit logging, cross-referral network (OrganizationRelationship), RRBC research questions, expert research library, FSE integration mapping, 5 decision records (DEC-CO-001 through DEC-CO-005) | Claude + Doron |

---

*This spec replaces the previous nonprofit-node.md. It is the authoritative reference for the Community Organization Engine node.*
