# Event Node — Assessment

> Last assessed: 2026-02-11
> Status: Needs Repositioning
> Owner: Doron

---

## Current Situation

The Event Node (events-node repo) was originally built as a Tier 3 Partner Node that would sync events to/from the Community Node. Since then, the Community Node has achieved full event management capability:

### What Community Node Already Does

- Full EventEditor with all fields (title, description, date/time, location, networks, age groups, capacity, images, Joy Coins pricing, frequency limits, refund policy)
- Event CRUD (create, edit, delete, cancel, duplicate)
- Recurring events with day-of-week selection
- RSVP system with capacity management
- Check-in system (QR code + manual)
- Joy Coins integration (cost per event, spot limits, check-in scans)
- Business Dashboard event management tab
- Admin event configuration (types, age groups, durations)
- Multi-ticket support
- Event analytics (basic)

### What Event Node Has That Community Node Doesn't

- Standalone deployment (could run without Community Node)
- Simpler UI focused purely on event coordination
- Layout reported as "really nice" — may have UX patterns worth porting back

### Open Questions

| Question | Notes |
|----------|-------|
| Does a standalone Event Node serve a real user? | Homeschool coordinators, festival organizers who want their own tool? |
| Should Event Node features be ported INTO Community Node instead? | The nice layout, any unique UX patterns |
| Is this a lab node or an archival candidate? | Depends on finding "the Dan" for events |

## Recommendation

Before any work on Event Node:

1. Identify a specific real user who needs event management beyond what Community Node provides
2. Interview them about their actual workflow (Research-First, DEC-048)
3. If a real gap exists, write a new spec following NODE-PLAYBOOK.md
4. If no gap exists, archive this repo and port any good UX patterns to Community Node

The original spec (Tier 1/2/3 sync-dependent Partner Node) is archived in Git history.

## Node Registry Entry

| Field | Value |
|-------|-------|
| Phase | Needs assessment |
| Real User | TBD — homeschool group? festival organizer? |
| Repo | events-node |
| Spec | This file (assessment only, not a build spec) |

---

*See NODE-PLAYBOOK.md (private repo) for current node development model.*
