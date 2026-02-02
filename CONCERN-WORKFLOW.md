# CONCERN-WORKFLOW.md — Concern Management Workflow

> Spec for making the Concern system sustainable for a solo steward at pilot scale.
> Created: 2026-02-02
> Status: Spec — Ready to build

---

## Problem

The Concern system works: users submit concerns, admins review them in the AdminConcernsPanel. But the current flow has gaps that break down at scale — even pilot scale:

1. **Silence after submission.** User submits a Concern and sees "Concern Received" — then nothing. No updates, no acknowledgment that anyone is looking at it. This breeds distrust, exactly what the system is supposed to prevent.

2. **No priority signals.** All Concerns look the same in the admin panel. A safety issue sits next to "the barista was slow" with no visual distinction. A solo steward scanning 10 Concerns wastes time figuring out what's urgent.

3. **No proactive notification.** The steward has to remember to check the admin panel. If they're busy onboarding businesses or running the newsletter, Concerns pile up unnoticed.

4. **Reporter has no visibility.** The person who submitted the Concern can't see whether it's been read, reviewed, or resolved. They have no reason to trust the process worked.

---

## Solution: Four Enhancements

### Enhancement 1: Auto-Acknowledgment on Submission

**What happens now:** User submits → sees "Concern Received" confirmation → nothing.

**What should happen:** User submits → sees "Concern Received" → immediately gets a visible record in their own experience confirming it's been logged with a reference number and expected response time.

**Implementation:**

On ConcernForm submission success, the confirmation screen should show:
- "Your concern has been received"
- Reference: `#CON-{id}` (using the Concern entity ID)
- "A community steward will review this within 48 hours"
- "You can check the status of your concern in Settings"

**Changes needed:**
- Update `Recommend.jsx` concern-done state to show reference ID and timeline
- Add a "My Concerns" section to Settings page (or MyLane) — see Enhancement 4

### Enhancement 2: Severity Category on Submission

**What happens now:** Concern form has: "What happened?" (free text), "When?" (optional), "What would you like?" (optional). All Concerns arrive as undifferentiated text.

**What should happen:** Add a category selector that helps the reporter classify their concern AND helps the steward triage without reading every word.

**New field on Concern entity:** `category` (string)

**Category options:**
| Value | Display Label | Admin Priority | Icon |
|-------|--------------|---------------|------|
| `safety` | "Safety or accessibility issue" | 🔴 High | ShieldAlert |
| `billing` | "Billing or payment dispute" | 🟡 Medium | CreditCard |
| `experience` | "Poor experience or service" | 🟢 Standard | MessageCircle |
| `other` | "Something else" | 🟢 Standard | HelpCircle |

**ConcernForm changes:**
- Add category selector between the heading and "What happened?" field
- Display as button group (not dropdown) — 4 options, one required
- Each button shows icon + label
- Selected state: `bg-amber-500/20 border-amber-500 text-amber-500`
- Unselected: `bg-slate-800 border-slate-700 text-slate-400`
- Category is required — description can't submit without it

**AdminConcernsPanel changes:**
- Show category badge next to status badge in the table
- Safety concerns get `bg-red-500/20 text-red-400` badge
- Billing concerns get `bg-amber-500/20 text-amber-400` badge
- Experience/other get `bg-slate-700 text-slate-300` badge
- Default sort: Safety first, then by date (newest)
- Add category to the filter pills (alongside status filter)

### Enhancement 3: Daily Digest Notification

**What happens now:** Steward must manually check AdminConcernsPanel to see new Concerns.

**What should happen:** Once per day, if there are any unresolved Concerns, the steward sees a notification. For pilot, this can be a simple in-app indicator rather than email (email notification is a post-pilot feature).

**Implementation approach — In-app notification badge:**

A notification dot on the Admin Panel nav link and the Concerns sidebar item, showing the count of `status: 'new'` Concerns. This already partially exists — the AdminConcernsPanel shows a count of new concerns. The enhancement is surfacing that count OUTSIDE the admin panel:

- **Header nav:** If user is admin and new concern count > 0, show a red dot on the user avatar or a subtle badge near the nav
- **Admin sidebar:** Already has the Concerns item — add a red count badge like `(3)` when new concerns exist
- **MyLane greeting (admin only):** Add a line: "You have 3 new concerns to review" with a link to the admin panel

**Data source:** Query `Concern.filter({ status: 'new' })` — already available.

**Why not email for now:** Base44 doesn't have a built-in email service, and wiring up a third-party email provider (SendGrid, etc.) is a post-pilot task. The in-app badge gets the steward's attention without infrastructure overhead.

**Future (post-pilot):** Daily email digest at 8am: "You have X new concerns (Y flagged as safety). [View in Admin Panel]"

### Enhancement 4: Reporter Status Visibility

**What happens now:** The reporter submits a Concern and never hears back through the platform. They don't know if it was read, reviewed, or resolved.

**What should happen:** The reporter can check the status of their submitted Concerns.

**Implementation — "My Concerns" in Settings:**

Add a section to the Settings page (below Account, above Save button) that shows the user's submitted Concerns:

```
MY CONCERNS
─────────────────────────────────

#CON-42 · Sunrise Yoga Studio        Safety issue
Submitted Jan 28                      🟡 Under Review

#CON-38 · Main Street Market          Poor experience
Submitted Jan 15                      ✅ Resolved

No public details are shown — just the business name,
category, date, and current status.
```

**Status display mapping:**
| Status | Display | Color |
|--------|---------|-------|
| `new` | "Received" | `text-amber-500` |
| `reviewing` | "Under Review" | `text-blue-400` |
| `resolved` | "Resolved" | `text-emerald-500` |
| `dismissed` | "Reviewed — No Action Needed" | `text-slate-400` |

**Why "Reviewed — No Action Needed" instead of "Dismissed":** The word "dismissed" feels invalidating. The reporter should understand it was reviewed, not ignored.

**Query:** `Concern.filter({ user_id: currentUser.id })` — sorted by created_date descending.

**Privacy:** Only the reporter sees their own Concerns. No other user can see them. Admin sees all via AdminConcernsPanel.

---

## Entity Changes

### Concern Entity — Add `category` Field

```
Field: category
Type: string
Options: safety | billing | experience | other
Required: Yes (for new submissions)
Default: null (existing records won't have it — handle gracefully)
```

Existing Concerns without a category should display as "Uncategorized" in the admin panel — no migration needed.

---

## Build Order

### Step 1: Add category to Concern entity and form
- Add `category` field to Concern entity in Base44 dashboard
- Update ConcernForm.jsx: Add category button group (required)
- Update Recommend.jsx concern-done screen: Show reference ID + 48hr timeline

### Step 2: Update AdminConcernsPanel
- Add category badge to table rows
- Add category filter pills
- Default sort: safety first, then newest
- Show category in detail drawer

### Step 3: Admin notification badge
- Create `useNewConcernCount` hook (queries Concern status: 'new', returns count)
- Add badge to Admin sidebar Concerns item
- Add badge to header nav (admin users only)
- Add "concerns to review" line in MyLane GreetingHeader (admin only)

### Step 4: Reporter status visibility
- Add "My Concerns" section to Settings.jsx
- Query user's own Concerns
- Display status with friendly labels
- Show only if user has submitted at least one Concern (hide section otherwise)

---

## SLA Commitment

The auto-acknowledgment message promises "within 48 hours." This sets the expectation. To enforce it:

- AdminConcernsPanel should highlight Concerns that are > 48 hours old and still `status: 'new'` — amber warning background on the row
- This is a visual reminder, not a system enforcement — the steward sees it and knows they're behind

**Future:** If email notifications are built, escalation rules could auto-notify if 48hr SLA is breached.

---

## What This Doesn't Solve

- **Resolution communication:** The reporter sees "Resolved" but doesn't know what happened. Adding a "resolution note" visible to the reporter is a future enhancement.
- **Email notifications:** Both submission confirmation and status change emails require email infrastructure (post-pilot).
- **Business notification:** The business doesn't know a Concern was filed about them unless the steward reaches out manually. This is intentional — the steward mediates.
- **Appeal process:** If a reporter disagrees with "No Action Needed," there's no formal appeal. At pilot scale, they can submit another Concern or contact the steward directly.

---

## Relationship to Gronk Critique

This addresses critique point #5 (Trust Architecture):
- **"Patterns might not surface fast enough"** → Safety category + admin badge ensures urgent issues get seen within hours, not days
- **"Users could feel gaslit if issues aren't public"** → Reporter visibility shows their Concern is tracked and has a status — they're not shouting into the void
- **"Need response SLA"** → 48-hour commitment with visual enforcement in admin panel

And critique point #9 (Solo execution):
- **"10 Concerns in week one with only one of you"** → Category triage means you read safety issues first, experience issues when you have time. Auto-acknowledgment buys breathing room. Badge notification means you don't have to remember to check.

---

*This spec is part of the LocalLane Spec-Repo. Reference before implementing.*
