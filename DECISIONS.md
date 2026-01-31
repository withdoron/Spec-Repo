# Decision Log

> Records key architectural and implementation decisions with context.
> Last Updated: 2026-01-30

---

## How to Use This Document

Each decision includes:
- **Date** — When the decision was made
- **Context** — Why we needed to decide
- **Decision** — What we chose
- **Rationale** — Why we chose it
- **Status** — Current state (Active, Superseded, Revisit)

---

## Implementation Decisions (DEC-001 …)

### DEC-001: Three-Tier System Naming

**Date:** 2026-01-28

**Context:** The tier system had inconsistent naming between spec documents and code.

**Decision:** Use code values `basic`, `standard`, `partner` consistently.

**Mapping:**
| Tier Level | Code Value | Display Name |
|------------|------------|--------------|
| 1 | `basic` | Basic |
| 2 | `standard` | Standard |
| 3 | `partner` | Partner |

**Rationale:** Code already used these values. Changing spec to match code is less disruptive than changing code + database.

**Status:** ✅ Active

---

### DEC-002: Tier 1 & 2 in Community Node, Tier 3 as Partner Node

**Date:** 2026-01-28

**Context:** Needed to clarify where each tier's functionality lives.

**Decision:**
- Basic & Standard users manage events within Community Node
- Partner users get their own standalone Partner Node
- Partner Node events sync UP to Community Node

**Rationale:** Simpler for majority of users (Tier 1 & 2). Premium experience for Partners. Consistent data in Community Node for search/discovery.

**Status:** ✅ Active

---

### DEC-003: Event Editor Copied from Event Node

**Date:** 2026-01-29

**Context:** Community Node's Event Editor was incomplete. Event Node had a full-featured editor.

**Decision:** Copy Event Node's EventFormV2 to Community Node, adapt for local data model and hooks.

**Approach:**
- Keep as inline editor (not modal) for MVP
- Use useOrganization() hook for tier checking
- Map field names to Community Node's Event schema
- Add LockedFeature component for tier gating

**Rationale:** Faster than building from scratch. Event Node's editor was already polished.

**Status:** ✅ Active — Phase 1 complete. Phase 2 (modal, recurring, etc.) planned for post-pilot.

---

### DEC-004: Hard Delete vs Soft Delete

**Date:** 2026-01-29

**Context:** Soft delete (`is_deleted: true`) wasn't working — field may not exist in Base44 schema.

**Decision:** Use hard delete for now. Plan to implement soft delete later.

**Rationale:** Needed working delete functionality for pilot. Hard delete is acceptable for test data. Production should use soft delete for data recovery.

**Status:** ⚠️ Active (temporary) — Revisit when Base44 schema can be modified.

---

### DEC-005: AdminSettings Fallback for Config Storage

**Date:** 2026-01-29

**Context:** Needed to store platform configuration (Event Types, Networks, etc.). Creating a new PlatformConfig entity requires Base44 dashboard access.

**Decision:** Use existing AdminSettings entity with key convention:
```
platform_config:{domain}:{config_type}
```

Value is JSON string: `{ items: [...], updated_at: Date }`

**Rationale:** AdminSettings already exists and works. Can migrate to dedicated PlatformConfig entity later without changing the useConfig hook interface.

**Status:** ✅ Active — Works well. Migration to dedicated entity is optional.

---

### DEC-006: Admin Panel Sidebar Navigation

**Date:** 2026-01-29

**Context:** Admin Panel had horizontal tabs. As features grow, tabs would become unwieldy.

**Decision:** Restructure Admin Panel with sidebar navigation organized by domain:
- MANAGEMENT (Businesses, Users, Locations, Partners)
- PLATFORM (Networks, Tiers, Punch Pass, Settings)
- EVENTS (Event Types, Age Groups, Durations)
- ONBOARDING (Business, User)

**Rationale:** Scales better as archetypes and features are added. More intuitive navigation. Supports deep linking via React Router.

**Status:** ✅ Active

---

### DEC-007: Config Inheritance Model

**Date:** 2026-01-29

**Context:** Partner Nodes need access to Event Types, Networks, etc. Should each Partner define their own, or inherit from Community Node?

**Decision:** Community Node owns all config. Partner Nodes inherit (read-only for now).

**Key Points:**
- Ensures congruent data for search/filter across all nodes
- User searches "Live Music" and finds ALL events, regardless of source
- Future: Can allow Partner additions tagged as "partner-specific"

**Rationale:** Data consistency is more important than Partner customization for MVP. Structure supports future flexibility.

**Status:** ✅ Active

---

### DEC-008: Event Editor Uses Dynamic Config

**Date:** 2026-01-30

**Context:** Event Types, Networks, Age Groups were hardcoded in EventEditor.

**Decision:** Replace hardcoded arrays with useConfig() hook calls. EventEditor fetches config from AdminSettings.

**Implementation:**
```javascript
const { data: eventTypes = [] } = useConfig('events', 'event_types');
const { data: networks = [] } = useConfig('platform', 'networks');
// etc.
```

**Rationale:** Admin can now add/edit/delete options without code changes. Single source of truth in Admin Panel.

**Status:** ✅ Active

---

### DEC-009: Purple Accent for Dev Mode

**Date:** 2026-01-28

**Context:** Dev Mode tier switcher in Event Node uses purple accent.

**Decision:** Keep purple for Dev Mode. Do NOT change to amber.

**Rationale:** Purple clearly signals "you're in dev/test mode" — visually distinct from production amber accent.

**Status:** ✅ Active

---

### DEC-010: Pilot Focus on Events + Venues

**Date:** 2026-01-29

**Context:** Multiple archetypes defined (Venue, Organizer, Service, Community, Product). Which to prioritize?

**Decision:** Focus on Events archetype first, then Venues (brick & mortar locations).

**Rationale:** 
- Events are core to LocalLane's value prop
- Venues support local businesses (the "support local" angle)
- Other archetypes (Service, Community, Product) can come post-pilot

**Status:** ✅ Active

---

### DEC-011: Staff/Instructor Roles with Custom Permissions

**Date:** 2026-01-29

**Context:** Instructors work at multiple businesses. Need role-based access.

**Decision:** Default role templates (Manager, Instructor, Staff) with per-person permission overrides (planned).

**Role Defaults:**
| Permission | Manager | Instructor | Staff |
|------------|---------|------------|-------|
| can_create_events | ✅ | ❌ | ❌ |
| can_edit_all_events | ✅ | ❌ | ❌ |
| can_edit_assigned_events | ✅ | ✅ | ❌ |
| can_manage_staff | ✅ | ❌ | ❌ |
| can_view_analytics | ✅ | Limited | ❌ |
| can_use_checkin | ✅ | ✅ | ✅ |

**Rationale:** Flexibility for business owners. One instructor might have different permissions at different businesses.

**Status:** ✅ Roles implemented, ⏳ Custom permissions planned

---

### DEC-012: Staff Invites Stored in AdminSettings

**Date:** 2026-01-30

**Context:** Needed to store pending staff invites for users who don't have accounts yet. The Base44 Business schema doesn't have a `staff` field.

**Decision:** Store pending invites in AdminSettings with key convention:
```
staff_invites:{business_id}
```

Value is JSON array: `[{ email, role, status, invited_at }, ...]`

Active staff (users who have accounts) remain in `business.instructors` array.

**Rationale:** AdminSettings already works for flexible storage. Allows invites without schema changes. When user creates account, we can auto-link and move them from invites to instructors.

**Status:** ✅ Active

---

### DEC-013: Staff Role Badge Colors

**Date:** 2026-01-30

**Context:** Need visual distinction between different staff roles.

**Decision:** 
- **Owner:** Amber badge (bg-amber-500 text-black)
- **Manager:** Purple badge (bg-purple-500 text-white)
- **Instructor:** Blue badge (bg-blue-500 text-white)
- **Staff:** Outline badge (border-slate-500 text-slate-300)
- **Pending:** Amber outline badge (border-amber-500 text-amber-500)
- **TEAM:** Amber outline badge on business cards for staff (not owner)

**Rationale:** Clear visual hierarchy. Owner is gold (most prominent). Manager is purple (distinct, high authority). Instructor is blue (professional). Staff is subtle outline (lowest permissions).

**Status:** ✅ Active

---

### DEC-014: Staff Auto-Link on Dashboard Load

**Date:** 2026-01-30

**Context:** When a user is invited to a business, they need to be automatically added when they create an account. Base44 handles auth, so we can't hook into signup directly.

**Decision:** Check for pending invites when user visits Business Dashboard:

```javascript
// In BusinessDashboard.jsx useEffect on mount:
1. Query all AdminSettings with key starting with 'staff_invites:'
2. For each, check if current user's email matches any invite
3. If match found:
   - Add user to business.instructors
   - Add role to staff_roles:{business_id}
   - Remove invite from staff_invites:{business_id}
4. Invalidate queries to refresh business list
```

**Rationale:** Can't modify Base44 auth flow. Dashboard load is reliable trigger — user will always visit dashboard. Runs once on mount, transparent to user.

**Status:** ✅ Active

---

### DEC-015: 403 Permission Handling for Staff View

**Date:** 2026-01-30

**Context:** When a staff member (not owner) views a business's Staff Widget, fetching User records for other staff fails with 403 (Base44 permission denied).

**Decision:** Gracefully handle 403 errors by showing "Team Member" placeholder:

```javascript
// In staffUsers query:
try {
  const user = await User.filter({ id });
  return user;
} catch (error) {
  // Permission denied - return placeholder
  return { id, email: 'Team Member', _permissionDenied: true };
}
```

**UI Behavior:**
- Owners see full staff details (names, emails, remove buttons)
- Staff members see "Team Member" placeholders (no details, no edit controls)
- Add Staff button only visible to owners

**Rationale:** Staff don't need to see other staff's personal info. Graceful degradation maintains functionality while respecting Base44's permission model.

**Status:** ✅ Active

---

### DEC-016: Staff Roles Stored in AdminSettings

**Date:** 2026-01-30

**Context:** Business.staff field doesn't exist in Base44 schema. Needed to store role (manager/instructor/staff) for each staff member.

**Decision:** Store roles in AdminSettings with key convention:
```
staff_roles:{business_id}
```

Value is JSON array: `[{ user_id, role, added_at }, ...]`

Combined with:
- `business.instructors` — Array of user IDs (active staff)
- `staff_invites:{business_id}` — Pending invites

**Rationale:** AdminSettings is flexible key-value store that already works. Keeps role data separate from Business entity. Can query by business ID.

**Status:** ✅ Active

---

### DEC-017: Red/Orange for Destructive Actions

**Date:** 2026-01-31

**Context:** Style Guide specified no functional color-coding (icons/actions should be gold or white only). However, destructive actions like Delete and Cancel Event need clear visual warning.

**Decision:** Allow red and orange for destructive actions only:
- Delete: `text-red-500` / `bg-red-500`
- Cancel Event: `text-orange-500` / `bg-orange-500`

**Rationale:** User safety trumps strict style consistency. Red/orange are universally understood danger signals. Limited to destructive contexts only—not for general status indicators.

**Status:** ✅ Active

---

## Strategic Decisions (2026-01-27)

### Next Archetype: Nonprofit/Church

**Decision:** The second archetype (after Events) will be nonprofit/church organizations.

**Context:** We need to prove the archetype system works by building a second type. Options included: mechanic shop, online store, restaurant, nonprofit.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Mechanic | Common business type | No immediate test case |
| Online Store | High demand | Complex (inventory, shipping) |
| Restaurant | Many local options | Complex (menus, ordering) |
| **Nonprofit/Church** | Real test case (Doron's church), similar data model to events | Narrower audience |

**Reasoning:** Doron has direct access to a church that currently uses paper/texts/conversations with no digital system. This provides real users with real pain, fast feedback loop, similar features to events (announcements, calendar, RSVPs), and proves the archetype system works.

**Consequences:** Church node becomes priority after Event Node stabilizes. We'll learn what's truly archetype-specific vs. universal.

**Made by:** Doron + Claude

---

### Tier 3 is Earned, Not Purchased

**Decision:** Partner Node status (Tier 3) cannot be bought — it must be earned through demonstrated trustworthiness.

**Context:** Most platforms sell premium tiers. LocalLane's differentiator is trust-based ranking.

**Reasoning:** The entire platform premise is "trust over ads." If businesses can buy their way to Partner status, the message is hollow. Tier 2 generates revenue; Tier 3 is the reward for being genuinely trusted.

**Consequences:** Need to define measurable trust criteria; may slow initial revenue (Tier 2 is main revenue source).

**Made by:** Doron (strong preference) + Claude (agreed)

---

### Punch Pass Launches in Demo Mode

**Decision:** The punch pass system will launch without real payment processing. Users will "purchase" with demo currency.

**Context:** Punch passes are a core feature, but real payments require payment processor integration, Terms of Service, refund policies, liability considerations.

**Reasoning:** Demo mode lets us test the full user experience, gather feedback on pricing and flow, launch faster, and defer payment processor integration to Phase 2.

**Consequences:** No revenue from punch passes in Phase 1; need to track demo transactions as if real for future migration.

**Made by:** Doron + Claude

---

### Spec-Repo is Source of Truth

**Decision:** All major decisions, architecture, and standards live in the Spec-Repo. This is the canonical reference.

**Context:** AI assistants don't have persistent memory. Without a single source of truth, decisions get lost and contradicted.

**Reasoning:** Spec-Repo lives in the same GitHub ecosystem as code repos, can be read by Cursor before making changes, and is the "external brain" for AI assistants.

**Consequences:** Must keep Spec-Repo updated; start new conversations by pointing to Spec-Repo.

**Made by:** Doron + Claude

---

### Repositories Are Public (For Now)

**Decision:** The three GitHub repositories (events-node, community-node, Spec-Repo) are public.

**Context:** Claude needs to review code. Options are public access or manual copy-paste.

**Reasoning:** At this stage (pre-revenue, pre-scale), the risk of code theft is low. Real competitive advantage is local relationships, community trust, and execution speed.

**Consequences:** Faster iteration now; can switch to private when there's real traction. Don't commit secrets/API keys.

**Made by:** Doron + Claude

---

## Decision Template

```markdown
### DEC-XXX: [Title]

**Date:** YYYY-MM-DD

**Context:** [Why we needed to decide]

**Decision:** [What we chose]

**Rationale:** [Why we chose it]

**Status:** [Active | Superseded | Revisit]
```

---

*Add new decisions at the bottom of this document.*
