# Archetype Template

> Use this template when defining a new archetype (business type) for the LocalLane ecosystem.

---

## Archetype: [NAME]

**Created:** YYYY-MM-DD  
**Status:** Draft | In Progress | Complete  
**Owner:** [Name]

---

## Overview

### What Is This Archetype?

[One paragraph describing this type of business/organization. Who are they? What do they do?]

### Example Businesses

- Example 1
- Example 2
- Example 3

### User Personas

**Business Owner:** [Who runs this type of business? What are their goals?]

**Customer/Member:** [Who uses this business? What do they need?]

---

## Data Model

### Core Fields (All Tiers)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | Yes | Business name |
| description | text | Yes | About this business |
| address | string | No | Physical location |
| phone | string | No | Contact phone |
| email | string | Yes | Contact email |
| website | string | No | Website URL |
| [archetype-specific] | | | |

### Tier 2 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| [field] | | |

### Tier 3 Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| [field] | | |

---

## Features by Tier

### Tier 1 (Free)

| Feature | Description |
|---------|-------------|
| Basic listing | Appears in community directory |
| Contact info | Phone, email, website displayed |
| [archetype-specific] | |

### Tier 2 (Pro)

| Feature | Description |
|---------|-------------|
| Everything in Tier 1 | |
| Accept punch passes | Can accept community punch passes |
| Analytics | View engagement metrics |
| [archetype-specific] | |

### Tier 3 (Partner Node)

| Feature | Description |
|---------|-------------|
| Everything in Tier 2 | |
| Own application | Dedicated Partner Node |
| Custom branding | Full control over look/feel |
| Priority placement | Higher in search results |
| Trust badge | Verified partner indicator |
| [archetype-specific] | |

---

## User Workflows

### Workflow 1: [Name]

**Actor:** [User type]

**Goal:** [What they're trying to accomplish]

**Steps:**
1. Step one
2. Step two
3. Step three

**Success criteria:** [How do we know this worked?]

---

### Workflow 2: [Name]

**Actor:** [User type]

**Goal:** [What they're trying to accomplish]

**Steps:**
1. Step one
2. Step two
3. Step three

**Success criteria:** [How do we know this worked?]

---

## Integration Points

### Data Pushed to Community Node

| Data | Trigger | Endpoint |
|------|---------|----------|
| [data type] | [when it happens] | [API endpoint] |

### Data Pulled from Community Node

| Data | Frequency | Endpoint |
|------|-----------|----------|
| Config (networks, categories) | On startup + every 5 min | GET /api/v1/config |
| [other data] | | |

### Punch Pass Integration

- [ ] This archetype can accept punch passes
- Accepted pass types: [list]
- Typical punch cost: [number]

---

## UI/UX Considerations

### Key Screens

1. **[Screen name]** — [purpose]
2. **[Screen name]** — [purpose]
3. **[Screen name]** — [purpose]

### Mobile Considerations

[Any specific mobile requirements for this archetype]

### Accessibility Considerations

[Any specific accessibility requirements]

---

## Technical Notes

### Special Requirements

[Any technical requirements unique to this archetype]

### Known Challenges

[Anticipated technical or UX challenges]

### Dependencies

[Other nodes, services, or features this depends on]

---

## Open Questions

| Question | Status | Answer |
|----------|--------|--------|
| [question] | Open | |
| [question] | Resolved | [answer] |

---

## Changelog

| Date | Change | Author |
|------|--------|--------|
| YYYY-MM-DD | Initial draft | [name] |

---

*Copy this template to create a new archetype spec. Save as `ARCHETYPES/[archetype-name].md`*
