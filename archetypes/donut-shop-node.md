# Donut Shop Node — Archetype Scope

> Light scoping document. Captures initial ideas for future development.
> Created: 2026-02-10
> Status: Idea
> Owner: Doron

---

## Overview

Brick-and-mortar retail/food service archetype. Doron's brother owns a couple of donut shops. Same standalone Base44 app pattern as Property Pulse and Contractor Daily — build it as a standalone tool that merges into LocalLane as an archetype.

## What This Archetype Covers

Any small retail food service operation: donut shops, bakeries, coffee shops, juice bars, ice cream shops. The common thread is walk-in customers, daily production, and simple inventory.

## Who Uses It

**Owner:** Runs one or more locations. Needs to track daily sales, manage simple inventory (what to make each day), schedule staff, and understand what's selling.

**Staff:** Works the counter or kitchen. Needs to know what to make, clock in/out, and handle customer orders.

**Customer:** Walks in, buys donuts. Might want to pre-order, check hours, or see today's menu.

## Key Questions to Answer (When We Build)

- What does daily production planning look like? (How many of each type to make)
- Is there a pre-order flow? (Call ahead, online ordering)
- How does inventory work for perishable goods? (Made fresh daily, waste tracking)
- Multi-location management — what does the owner need across 2+ shops?
- POS integration or standalone? (Does this replace or complement existing POS?)
- Staff scheduling — simple shifts or complex?
- What data feeds the Organism? (Customer visits, production volume, waste reduction)

## Potential Features by Phase

**Phase 1: Daily Operations**
- Daily production log (what was made, how many, what sold, what wasted)
- Simple menu management (items, prices, categories)
- Staff list with roles and contact info
- Daily sales summary (manual entry or simple register)

**Phase 2: Planning & Analytics**
- Production planning based on historical sales data
- Waste tracking and reduction insights
- Popular items and trends
- Multi-location dashboard (if applicable)

**Phase 3: Customer-Facing**
- Today's menu (what's available right now)
- Pre-order / call-ahead
- Hours and location info
- Loyalty program (ties into Community Pass / Joy Coins)

**Phase 4: LocalLane Integration**
- Business listing in community directory
- Community Pass acceptance
- Events (donut decorating classes, tastings)
- Harvest Network connection (local ingredients sourcing)

## Organism Signals

| Activity | Signal | Level |
|----------|--------|-------|
| Daily production logged | Business vitality | Business |
| Customer purchases | Transaction activity | Business + Community |
| Waste reduced week over week | Sustainability | Business |
| Local ingredient sourcing | Harvest Network connection | Network |
| Community events hosted | Community participation | Business + Community |

## Real Test Case

Doron's brother's donut shops. Direct access to a real operator with real pain points. Same advantage we had with Dan Sikes for Contractor Daily and Doron's own properties for Property Pulse.

## Dependencies

- Base44 platform (standalone app)
- Gold Standard design system
- Role system pattern (from PP and CD)

## Open Questions

| Question | Status |
|----------|--------|
| What POS does brother currently use? | Unknown |
| How many locations? | 2+ (confirm) |
| What's the biggest daily pain point? | Need to ask |
| Does he track inventory now? How? | Need to ask |
| Staff size per location? | Need to ask |

---

*This is a holding doc. Full spec will be written using archetypes/_template.md when we're ready to build.*
