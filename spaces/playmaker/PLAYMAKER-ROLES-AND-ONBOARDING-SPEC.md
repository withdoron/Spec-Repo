# Playmaker Role Capability Matrix & Onboarding Flows

## Purpose
Define roles, capabilities, and onboarding flows for team spaces. Each relationship has exactly one correct entry path. The person with the right to add another person is defined by their role, not their account. This is the membrane principle applied to onboarding.

## Roles

### Head Coach (`role: coach`)
Team creator by default. Exactly one per team. Root of trust.

**Can:** create/edit/delete any team member, play, event, message, photo; send Coach Link, Family Link, League Link; promote an assistant to Head Coach (atomic swap); edit team settings; delete the team.

**Cannot:** be removed by an assistant.

### Assistant Coach (`role: assistant_coach`)
Trusted delegate. Coaching capability without destructive power over others' work.

**Can:** create/edit plays, events, messages; **delete plays, events, and messages they themselves created**; edit player info (jersey name, number, position); send Family Link; upload photos; view everything.

**Cannot:** delete team members; delete content created by others; send Coach Link or League Link; edit team settings; promote/demote; delete the team.

### Parent (`role: parent`)
Views the team, manages their own child(ren) only.

**Can:** view roster/playbook/schedule/messages/photos; RSVP for their linked child's events; edit their linked child's info; upload photos (delete only their own); claim a player row via Family Link. A parent may be linked to multiple children on the same team.

**Cannot:** see Coach Link or League Link; edit other children; edit plays/events/team settings.

### Player (`role: player`)
Future — most youth players don't have accounts. Deferred.

## Onboarding Flows

Exactly three ways a person becomes a TeamMember. No other paths.

### Flow 1 — Add Player (Head Coach or Assistant Coach)
- UI: "Add Player" button on Roster tab
- Fields: `jersey_name` (required), `jersey_number`, `position`
- Creates TeamMember: `role: player`, `status: active`, `user_id: null`, `pending_claim: true`
- Pending badge visible to coaches only (informational — see Pending Rule)
- Coach can delete at any time

### Flow 2 — Invite Coach (Head Coach only)
- UI: "Invite Coach" button on Roster tab. **No role dropdown** — always creates an assistant coach. Head coach is earned via promotion, never via invite.
- Modal prompts for `jersey_name` only
- Creates TeamMember: `role: assistant_coach`, `pending_claim: true`, `user_id: null`
- Generates Coach Link (shareable URL) to send to the person
- Recipient opens link → sees pending assistant coach rows they can claim (name must match)
- On claim: `user_id` set, `pending_claim: false`

### Flow 3 — Claim Child (Parent, via Family Link)
- Parent opens Family Link URL, lands on team onboarding
- Selects their child from the existing player list
- Creates TeamMember: `role: parent`, `user_id: self`, `jersey_name: parent's display name`
- Adds parent's `user_id` to the player row's `parent_user_ids` array
- Multiple parents may link to the same player (mom + dad + stepparent)

### Pending Claim Rule
`pending_claim: true` is **informational to coaches only** — it tells coaches which rows have not yet been claimed by a real user account. It is not a functional restriction on roster visibility. It clears when: the row is claimed via matching invite link, the head coach manually clears it, or the row is deleted.

## Head Coach Promotion

- UI: "Promote to Head Coach" action on assistant coach rows, visible only to the current head coach
- Confirmation modal: "This will swap roles. You will become Assistant Coach. Continue?"
- On confirm (atomic swap): promoter → `assistant_coach`, promoted → `coach`
- No modal asking the outgoing head coach if they want to leave the team. If they want to leave entirely, they delete themselves from their own Pip-Boy (users always control their own exit).
- Exactly one head coach per team at all times.

## Removing an Assistant Coach

When a head coach removes an assistant coach:
- **If the assistant has a linked child on the team** → demote to `role: parent`. They retain access via their parent relationship.
- **If they have no linked child** → delete the TeamMember row entirely.

This preserves the parent relationship when one exists. A former coach who is also a dad does not lose access to his kid's team.

## Roster Sort Order (Family Grouping)

The roster displays families as cohesive units, sorted top-to-bottom:

1. **Head Coach** — followed immediately by their linked children
2. **Assistant Coaches** — each followed immediately by their linked children
3. **Parents** — grouped with their linked children. When two parents share children (both on the platform), they appear together followed by their shared kids.

Families stay together visually. The roster reads as relationships, not a flat list.

Example:
```
Coach Rick (Head Coach)
  └─ Lincoln (Player)
Doron (Assistant Coach)
Amy (Parent of Egan, Elek)
  └─ Egan (Player)
  └─ Elek (Player)
Jeslyn (Parent of Riley)
  └─ Riley (Player)
```

## Message Attribution with Family Context

Messages display both the author's name and their family relationship when applicable:

- Coach: `Rick (Head Coach)`
- Assistant coach who is also a parent: `Doron (Assistant Coach, parent of Egan and Elek)`
- Parent: `Amy (Parent of Egan and Elek)`

Messages become relational, not just identity-based. The sender is not just a person — they are the parent of these specific kids on this team.

## Season Lifecycle

Playmaker does not auto-archive or auto-rollover seasons. At the end of a season:

1. **Head coach cleans the team space** — deletes old player rows, old events, stale messages as appropriate.
2. **Head coach re-launches for the new season** — adds the new roster, sends fresh Family Links, invites returning or new assistant coaches.
3. **The team itself persists** — team name, head coach identity, playbook (optional reuse), and any assets the coach wants to carry forward stay in place. Only roster and schedule reset.
4. **Pending rows from the previous season** are cleared when the head coach deletes the underlying TeamMember rows. No expiration automation.

Future seasons, league rollover, and roster carryover are out of scope for v1. Manual cleanup is the deliberate choice — it keeps the coach in control and avoids hidden automation that could surprise a returning user.

## Capability Enforcement

Two layers, both required:

1. **UI layer** — Hide/show buttons based on current user's role. Delete icons only render for head coach; Coach Link button only for head coach; Promote action only for head coach viewing an assistant row.
2. **Server function layer** — Every mutation verifies the caller's role via membership check, then enforces capability rules. **UI hiding is not sufficient — the server function is the actual membrane.**

## UI Labels

- `role: coach` → "Head Coach" (gold badge)
- `role: assistant_coach` → "Assistant Coach" (amber badge)
- `role: parent` → "Parent" (neutral badge)
- `role: player` → "Player" (neutral badge)

## Out of Scope (v1)

League tier and play library forking; player accounts; transferring ownership to a brand new person (only promotion of existing assistants is supported); multi-team coaches; template forking between teams; season archive/rollover automation.

---

*Source of truth for the Hyphae audit and phased build. Capture any changes directly in this file before handing off.*
