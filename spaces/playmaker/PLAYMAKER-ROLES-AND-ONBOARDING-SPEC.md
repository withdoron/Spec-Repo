# Playmaker Role Capability Matrix & Onboarding Flows (v2)

## Purpose
Define roles, capabilities, and onboarding flows for team spaces. Every person is one row. Every role is derived from a TeamMember field, never duplicated elsewhere. Every entry point requires a pre-seeded pending row. The person with the right to add another person is defined by their role, not their account. This is the membrane principle applied to onboarding.

v2 supersedes v1. Changes from v1: one invite code per team (not two), parent-to-assistant-coach promotion added as the primary assistant entry path, non-parent coaches handled via one-time claim tokens, head coach derived from `role` field (not stored on Team), unified promotion action replaces legacy Transfer Ownership, all adult roles share the `linked_player_ids` array shape, dual identity in message attribution, settings surface audit added as universal build protocol, and the Roster tab exposes a single unified **"Add"** button with a modal decision tree (matches TeamSnap's industry-standard pattern while preserving our tighter membrane).

## Roles

### Head Coach (`role: coach`)
Exactly one per team, derived from the single TeamMember row with `role: coach`. Root of trust. Team creator is the initial head coach by default, but the role is not bound to team creation — it transfers freely via promotion.

**Can:** create, edit, and delete any team member, play, event, message, or photo; promote any parent to assistant coach; promote any assistant coach to head coach (atomic swap); add a non-parent coach directly via one-time claim token; edit team settings; delete the team.

**Cannot:** be removed by an assistant coach.

### Assistant Coach (`role: assistant_coach`)
Trusted delegate. Coaching capability without destructive power over other people's work.

**Can:** create and edit plays, events, and messages; **delete plays, events, and messages they themselves created** (own content only); edit player info (jersey name, number, position); upload photos; view everything on the team.

**Cannot:** delete team members; delete content created by others; generate non-parent coach claim tokens; edit team settings; promote or demote; delete the team.

### Parent (`role: parent`)
Views the team, manages their own linked child(ren).

**Can:** view roster, playbook, schedule, messages, photos; RSVP for their linked children's events; edit their linked children's info (jersey name, number, position); upload photos (delete only their own); claim additional player rows via the team invite code.

**Cannot:** edit other children; edit plays, events, or team settings; promote anyone.

### Player (`role: player`)
Future — most youth players don't have accounts today. Deferred. Player rows currently exist as claim targets for parents, not as login identities.

## Linked Children — Symmetric Array on All Adult Roles

Every adult row (head coach, assistant coach, parent) carries `linked_player_ids: string[]` — an array of TeamMember IDs pointing at player rows on the same team. One row per person, regardless of role.

- A head coach with no child on the team has `linked_player_ids: []`.
- An assistant coach with two children on the team has their two player IDs in the array.
- A parent with one child has one ID. A parent with three children has three.
- Player rows store the reverse pointer in `parent_user_ids: string[]` — a list of user IDs for every adult currently linked to that player. The join and claim flows write both directions symmetrically.

Role changes (promotion, demotion, remove-assistant-with-demote) never touch `linked_player_ids` — only the `role` field flips. A person's family links are durable across every role transition.

## Onboarding Flows

There are exactly three ways a person becomes a TeamMember. No other paths.

### Flow 1 — Claim a Player (Parent entry, default path)
The primary and near-universal entry point for adults on a team. Every new adult comes through here unless they are a non-parent coach (Flow 3).

1. A head coach or assistant coach clicks **"Add"** on the Roster tab and selects **"A player on the team"** in the modal (see "Roster UI — The Add Button" below).
2. Fields: `jersey_name` (required), `jersey_number`, `position`.
3. A player TeamMember is created with `role: player`, `pending_claim: true`, `user_id: null`, `parent_user_ids: []`.
4. The row appears on the roster with a "Pending" badge visible to coaches only. The badge is informational — not a functional restriction.
5. The coach shares the **team invite code** (one code per team, stored on the Team entity, reusable for the life of the team).
6. The parent opens the invite URL, signs in, and is shown the list of pending player rows on the team. Name matching is fuzzy — case-insensitive, whitespace-tolerant. "riley" matches "Riley".
7. If a fuzzy match is found, the parent claims the matching player row. If no fuzzy match is found, the parent is hard-blocked with the message: *"No pending player matches your child's name. Ask the coach to add them first."* The block is enforced server-side, not only in the UI.
8. On successful claim, the parent's TeamMember row is created (or updated, if they are already on the team linking another child) with `role: parent`, `user_id: self`. The claimed player's ID is appended to the parent's `linked_player_ids`. The parent's user ID is appended to the player's `parent_user_ids`. The player's `pending_claim` flips to false.

A parent may return and claim additional player rows via the same flow. Each claim appends to the same parent row — never creates a duplicate row for the same person.

### Flow 2 — Promote an Existing Member (Assistant coach entry, normal path)
Most assistant coaches on youth teams are already parents on the team. Rather than a separate onboarding flow, they are promoted from parent to assistant coach from within the platform.

1. The head coach clicks **"Add"** on the Roster tab and selects **"A coach"** in the modal. When prompted *"Is [name] already a parent on the team?"* the head coach answers **Yes** and picks the existing parent row from a short picker. (Alternatively, the head coach can click the "Promote to Assistant Coach" action directly on a parent row in the roster — both entry points trigger the same server function.)
2. Confirmation modal: *"Grant coaching capabilities to [name]?"*
3. On confirm, the target row's `role` field flips from `parent` to `assistant_coach`. `linked_player_ids` is untouched — they remain linked to their children.
4. The new assistant coach immediately gains coaching capabilities per the role matrix above.

The reverse (demoting an assistant coach back to parent) is available to the head coach via the "Removing an Assistant Coach" rules below.

### Flow 3 — Add Coach Directly (Non-parent coach, edge case)
For the rare case where a non-parent coach needs to be added — a hired assistant, a school coach without a kid on the team, a friend of the head coach volunteering. Head coach only.

1. The head coach clicks **"Add"** on the Roster tab and selects **"A coach"** in the modal. When prompted *"Is [name] already a parent on the team?"* the head coach answers **No**.
2. Field: `jersey_name` (the coach's display name).
3. A TeamMember row is created with `role: assistant_coach`, `pending_claim: true`, `user_id: null`, `linked_player_ids: []`.
4. A **one-time claim token** is generated and tied to that specific TeamMember row. The token is NOT the team invite code — it is single-use, expires 72 hours after generation, and is invalidated on claim.
5. The head coach copies the claim URL (`/claim-coach/:token`) and sends it to the non-parent coach directly.
6. The invited coach opens the URL, signs in, and the server claims the linked row automatically. No pick-list — the token uniquely identifies which row to claim.
7. `user_id` is set, `pending_claim` flips to false, the token is invalidated.

This path is the only way to create an assistant coach with no linked children. It uses a different URL, a different entity field (`coach_claim_token` on TeamMember), and a different server function from the team invite code. The membrane stays tight: no standing URL ever grants coach capability.

## Roster UI — The Add Button

The Roster tab exposes exactly one additive button: **"Add"** (with a + icon). There is no separate "Add Player" or "Add Coach" button. The three Flows above are real distinctions in the data layer and in the server functions, but a head coach looking at the roster thinks in one verb: *"I need to add someone."*

Clicking **"Add"** opens a modal that walks the head coach (or assistant coach, for Flow 1 only) through a short decision tree:

**Question 1: Who are you adding?**
- **A player on the team** → proceeds to the Flow 1 form (jersey name, number, position). Creates a `role: player` pending row. The parent claims the row later via the team invite code.
- **A coach** → proceeds to Question 2. *(Disabled for assistant coaches — only the head coach can add coaches.)*

**Question 2 (coach path only): Is [name] already a parent on the team?**
- **Yes** → shows a picker of current parent rows on the team. The head coach selects the matching parent, confirms the promotion, and Flow 2 executes. No new row is created — the existing parent row's `role` flips to `assistant_coach`.
- **No** → proceeds to the Flow 3 form (coach's display name). Creates a `role: assistant_coach` pending row with empty `linked_player_ids` and generates a one-time claim token. The head coach copies the claim URL and sends it to the non-parent coach directly.

**Alternate entry for Flow 2 only:** a "Promote to Assistant Coach" action is also available inline on parent rows in the roster, as a shortcut when the head coach is already looking at the parent they want to promote. Both entry points invoke the same server function.

This pattern — one button, modal decision tree, three distinct data-layer paths underneath — matches the industry-standard model used by TeamSnap and preserves our tighter membrane (pre-seeded pending rows, fuzzy-match claim, no bare invite links granting coach capability).

## Invite Codes and Claim Rules

- **One invite code per team.** Stored on the Team entity, generated at team creation, reusable for the life of the team. Same code handles all Flow 1 parent claims. No separate Coach Link as a standing URL.
- **Pre-seeded pending row required.** No one claims into a team without a matching pending row created by a coach first. There is no free-pick UI.
- **Hard block on no-match.** If a claimer's name does not fuzzy-match any pending row, they are blocked with an instruction to ask the coach to add them. Enforced server-side, not only in the UI.
- **Fuzzy matching** is case-insensitive and whitespace-tolerant. If multiple rows fuzzy-match, the claimer is shown a short pick-list of matches only — never the full roster.
- **Non-parent coaches** use one-time tokens (Flow 3), never the team invite code.

## Head Coach Derivation

The head coach is **derived at query time** from the single TeamMember row on a given team with `role: coach`. Exactly one by invariant.

- A server-side helper `getHeadCoach(team_id)` returns the head coach TeamMember row.
- Every UI gate and server-side capability check that needs "is this user the head coach of this team?" uses this helper.
- The `owner_id` field on the Team entity is preserved as historical "who originally created this team" and is never used for capability gating.
- No `head_coach_member_id` or `head_coach_user_id` field is stored on the Team entity. The `role` field on TeamMember is the single source of truth.
- This pattern is forward-compatible with the future League tier: league-level queries derive each team's head coach the same way, one layer up, with no sync and no stored pointer at the league level either.

## Promotion and Transfer

Promotion is both the in-season role change mechanism and the setup-to-permanent-holder transfer mechanism. One action, one server function, two contexts.

### Promote to Assistant Coach
- Head coach only.
- Action appears on parent rows in the Roster tab.
- On confirm, the target row's `role` flips from `parent` to `assistant_coach`. Atomic.

### Promote to Head Coach
- Head coach only.
- Action appears on assistant coach rows in the Roster tab AND in the Team Settings surface (see "Settings Surface Consolidation" below).
- Confirmation modal: *"This will swap roles. You will become Assistant Coach. Continue?"*
- On confirm, an atomic swap in a single server function transaction: promoter's `role` → `assistant_coach`, promoted's `role` → `coach`. Both rows updated in one call. No partial state possible.
- No automatic "do you want to leave the team?" modal. If the outgoing head coach wants to exit entirely, they delete themselves from their own MyLane surface — users always control their own exit.

### Setup Transfer (same action, different context)
When a gardener (Doron, a league coordinator, or any team creator) creates a team on behalf of someone else, they become the initial head coach by default. Once the intended permanent head coach has joined the team — via Flow 1 (claiming their own child), Flow 2 (being promoted from parent), or Flow 3 (non-parent coach with one-time token) — the gardener uses the exact same "Promote to Head Coach" action to hand off the role. No separate transfer mechanism. No separate UI. **Promotion IS transfer.**

## Removing an Assistant Coach

When a head coach removes an assistant coach from the team:

- **If the assistant has linked children** (`linked_player_ids` is non-empty) → the row's `role` flips from `assistant_coach` to `parent`. They retain access to the team via their parent relationship. `linked_player_ids` is untouched.
- **If the assistant has no linked children** → the TeamMember row is deleted entirely.

This preserves the parent relationship when one exists. A former coach who is also a dad does not lose access to his kid's team.

## Roster Sort Order — Family Grouping

The roster displays families as cohesive units. Sort order top-to-bottom:

1. **Head Coach** — followed immediately by their linked children, if any
2. **Assistant Coaches** — each followed immediately by their linked children
3. **Parents** — each followed immediately by their linked children

When two adults share one or more children (co-parents both on the team), they may be rendered as a clustered family unit with the shared children grouped underneath both adults. This is a Phase 3 rendering refinement; the underlying data model supports it without change.

Unclaimed player rows (`pending_claim: true`, `parent_user_ids: []`) render at the bottom under a "Pending" section, visible to coaches only.

## Message Attribution with Family Context

Messages display both the author's role and their family relationship when applicable. **Dual identity is not a choice — it is a union.** A coach who is also a parent is both at the same time, and the attribution shows both simultaneously.

- Head coach with no linked children: `Rick (Head Coach)`
- Head coach with linked children: `Rick (Head Coach, parent of Marcus)`
- Assistant coach with no linked children: `Jamie (Assistant Coach)`
- Assistant coach with linked children: `Doron (Assistant Coach, parent of Egan and Elek)`
- Parent: `Amy (Parent of Egan and Elek)`

Messages become relational, not just identity-based. The sender is not just a person — they are this person in this role with these family links on this team.

## Season Lifecycle

Playmaker does not auto-archive or auto-rollover seasons. At the end of a season:

1. The head coach manually cleans the team space — deletes old player rows, old events, stale messages as appropriate.
2. The head coach re-launches for the new season — adds the new roster via Flow 1, re-shares the team invite code, promotes returning or new assistant coaches via Flow 2 (or Flow 3 for non-parent coaches).
3. The Team entity persists. Team name, head coach identity, playbook (optional reuse), and any assets the coach wants to carry forward stay in place. Only roster and schedule reset.
4. The team invite code persists across seasons unless the head coach explicitly rotates it. Rotation is a future capability, not v1.
5. Pending rows from the previous season are cleared when the head coach deletes the underlying TeamMember rows. No expiration automation.

Future automated rollover, roster carryover, and league-wide season transitions are out of scope for v1. Manual cleanup is the deliberate choice — it keeps the coach in control and avoids hidden automation that could surprise a returning user.

## Settings Surface Consolidation

The Team Settings tab currently contains a legacy "Transfer Ownership" UI that mutates `owner_id` on the Team entity. Under this spec, that mechanism is retired entirely.

- The "Transfer Ownership" section in Settings is replaced by a "Promote to Head Coach" section that invokes the same unified promotion server function used by the roster-level action.
- Both surfaces (Roster row action, Settings panel action) invoke the same server function. Same atomic swap, same confirmation modal, same outcome.
- The `owner_id` field is no longer touched on promotion. It remains as historical "who originally created this team."
- Any code currently reading `owner_id` to mean "head coach" is migrated to use the `getHeadCoach()` helper.

**Build protocol addendum (applies to every space, every build):** Every Hyphae audit and build pass must include a sweep of the relevant space's settings surface. Settings tabs accumulate legacy actions and stale toggles that drift silently when new features land on other surfaces. Audit settings alongside the main build target, flag conflicts with the current spec, and consolidate or remove legacy actions as part of the same commit. Not Playmaker-specific.

## Capability Enforcement

Two layers. Both required. Neither sufficient alone.

### UI layer
Hide or show buttons based on the current user's role on the team.

- Delete icons render only for the head coach.
- "Add Coach Directly" renders only for the head coach.
- "Promote to Head Coach" renders only for the head coach viewing an assistant coach row.
- "Promote to Assistant Coach" renders only for the head coach viewing a parent row.
- Coach-gated actions (edit plays, create events, send messages) render for both head coach and assistant coach, but "delete" on plays/events/messages renders only for the creator of that content.

### Server function layer
Every mutation goes through a server function that verifies the caller's role on the team via a membership check, then enforces capability rules. **UI hiding is not sufficient — the server function is the actual membrane.** A technical user calling `base44.entities.TeamMember.delete()` directly from the browser console must be blocked at the server function boundary.

Required server functions for Phase 1:

- `manageTeamMember` — all TeamMember create, update, and delete mutations route through it. Role check on every call.
- `promoteToHeadCoach` — atomic role swap in a single transaction.
- `promoteToAssistantCoach` — parent → assistant coach flip.
- `removeAssistantCoach` — handles demote-to-parent-if-linked or delete-if-unlinked per the rules above.
- `generateCoachClaimToken` — creates one-time token for Flow 3. Head coach only.
- `claimCoachToken` — validates and consumes one-time token. Public endpoint, token is the auth.
- `claimPlayerRow` — handles Flow 1 parent claims with fuzzy match and hard block. Public endpoint, team invite code is the auth.
- `getHeadCoach(team_id)` — read helper used by every capability check.

### Known critical gap to fix
The existing `claimTeamSpot` server function currently hardcodes `role: 'coach'` on the coach-join path (identified by Hyphae's audit at line 198 of the current file). This single line is the most dangerous gap in the current code relative to this spec — every coach who joins via the legacy Coach Link becomes a full head coach. Hyphae's reconciliation pass determines whether the line is fixed in place (change to `assistant_coach`) or the entire legacy coach-join path is removed in favor of Flow 2 and Flow 3. Either way, this line cannot survive Phase 1b.

## UI Labels

- `role: coach` → "Head Coach" (gold badge)
- `role: assistant_coach` → "Assistant Coach" (amber badge)
- `role: parent` → "Parent" (neutral badge)
- `role: player` → "Player" (neutral badge)

## Out of Scope (v1)

League tier and cross-team play library forking; player accounts with their own logins; multi-team coaches (someone who coaches two teams simultaneously); template forking between teams; automated season archive or rollover; custody-restricted parent visibility; invite code rotation; assistant-coach self-promotion (only head coach can promote); demotion of head coach without a replacement (the atomic swap requires a target).

---

*Source of truth for the Hyphae reconciliation audit and phased build. Supersedes v1. Changes to this spec must be captured here before they reach Hyphae.*
