# Local Ownership Badges — Feature Spec

> Helping users identify and prioritize locally owned businesses at a glance.
> "Shop local, on purpose."

---

## Status: 📋 READY FOR NEAR-TERM BUILD

Last updated: 2026-02-01

---

## The Idea

LocalLane's mission is to revitalize local commerce. Right now we have a single boolean field (`is_locally_owned_franchise`) that distinguishes locally-owned franchise businesses. But "local" is more nuanced than that, and users should be able to filter by it.

A coffee shop owned by someone in Springfield is different from a Starbucks. A locally-owned Subway franchise is different from a corporate-managed one. An Oregon-based company with 12 locations is different from a national chain. Users deserve to know.

---

## What Exists Today

| Field | Type | Where Used |
|-------|------|------------|
| `is_locally_owned_franchise` | boolean | BusinessCard badge, BusinessEditDrawer toggle, AdminBusinessTable toggle, AdminSettingsPanel visibility control |

The existing badge shows "Local Franchise" on BusinessCard as a small overlay badge, controlled by an admin setting (`show_locally_owned_franchise_badge`).

---

## Proposed: Ownership Type Field

Replace the boolean with a richer classification. The Business entity gets a new field:

```javascript
ownership_type: 'locally_owned' | 'local_franchise' | 'oregon_based' | 'national_chain' | 'nonprofit' | null
```

| Value | What It Means | Badge Text | Example |
|-------|--------------|------------|---------|
| `locally_owned` | Majority owned and operated by someone in the community | 🏠 Locally Owned | Mom's Diner, Springfield Auto |
| `local_franchise` | Franchise brand, but this location is locally owned/operated | 🏪 Local Franchise | Locally-owned Subway, local Ace Hardware |
| `oregon_based` | Company is headquartered in Oregon (may have multiple locations) | 🌲 Oregon Based | Dutch Bros, Bi-Mart |
| `national_chain` | National/corporate-owned chain location | (no badge) | Safeway, Target |
| `nonprofit` | Registered nonprofit / community organization | 💛 Nonprofit | Eugene YMCA, Food Bank |
| `null` | Not yet classified | (no badge) | New businesses before classification |

### Key Design Decision: National chains get NO badge

We don't penalize national chains — Safeway is on the platform and that's fine. But we don't celebrate them either. The badge system highlights what makes a business *locally connected*. No badge just means "we don't know" or "it's a chain." No negative signaling.

---

## Badge Display

### On BusinessCard (image overlay area)

Replace the current `is_locally_owned_franchise` badge logic:

```jsx
{/* Current */}
{business.is_locally_owned_franchise && showFranchiseBadge && (
  <span className="...">
    <Store className="h-2 w-2 mr-1" />
    Local Franchise
  </span>
)}

{/* New */}
{business.ownership_type && business.ownership_type !== 'national_chain' && showOwnershipBadge && (
  <span className="...">
    <OwnershipIcon type={business.ownership_type} className="h-2 w-2 mr-1" />
    {OWNERSHIP_LABELS[business.ownership_type]}
  </span>
)}
```

### On BusinessProfile (header area)

Show alongside the tier badge and "Accepts Silver" badge:

```jsx
{business.ownership_type && business.ownership_type !== 'national_chain' && (
  <Badge variant="outline" className="border-emerald-500/30 text-emerald-500 bg-emerald-500/10">
    <OwnershipIcon type={business.ownership_type} className="h-3 w-3 mr-1" />
    {OWNERSHIP_LABELS[business.ownership_type]}
  </Badge>
)}
```

### Badge Colors (Gold Standard compliant)

All ownership badges use the same subtle style — no color-coding per type:

```
Overlay badge (BusinessCard):
  bg-black/20 backdrop-blur-sm text-white/90 text-[9px] font-normal px-1.5 py-0.5 rounded

Profile badge (BusinessProfile):
  border-emerald-500/30 text-emerald-500 bg-emerald-500/10
```

Emerald (green tint) for all ownership badges — it's the "local/community" color, distinct from amber (action/accent) and the tier system.

---

## Filtering

### Directory Page

Add an "Ownership" filter alongside the existing category and "Accepts Silver" filters:

```jsx
{/* New filter button in the filter bar */}
<button
  onClick={() => setOwnershipFilter(ownershipFilter === 'local' ? null : 'local')}
  className={ownershipFilter === 'local' ? activeFilterClass : inactiveFilterClass}
>
  <Home className="h-3.5 w-3.5" />
  Locally Owned
</button>
```

Filter options:
- **Locally Owned** — shows `locally_owned` only
- **Local** — shows `locally_owned` + `local_franchise` + `oregon_based` + `nonprofit` (everything except `national_chain`)
- **Oregon Based** — shows `oregon_based` only

Implementation: a simple dropdown or pill group in the filter bar.

### Search Page

`SearchResultsSection` should respect the same ownership filter if active.

### MyLane (Future)

HappeningSoon and NewInCommunity sections could optionally filter/weight toward local businesses.

---

## Admin Controls

### BusinessEditDrawer

Replace the `is_locally_owned_franchise` toggle with a dropdown:

```jsx
<Select
  value={editData.ownership_type || ''}
  onValueChange={(value) => handleFieldChange('ownership_type', value || null, 'ownership_type_change')}
>
  <SelectItem value="">Not classified</SelectItem>
  <SelectItem value="locally_owned">Locally Owned</SelectItem>
  <SelectItem value="local_franchise">Local Franchise</SelectItem>
  <SelectItem value="oregon_based">Oregon Based</SelectItem>
  <SelectItem value="national_chain">National Chain</SelectItem>
  <SelectItem value="nonprofit">Nonprofit</SelectItem>
</Select>
```

### AdminBusinessTable

Replace the franchise toggle column with an ownership type column showing the current value as a badge or text.

### AdminSettingsPanel

Replace the `show_locally_owned_franchise_badge` toggle with `show_ownership_badges` (single toggle for all ownership badges).

### Business Onboarding

Add ownership type selection to Step 2 (Details) in `BusinessOnboarding.jsx`:

```
"How would you describe your business ownership?"
○ Locally Owned — I own and operate this business in the community
○ Local Franchise — This is a franchise location that I own locally
○ Oregon-Based Company — Our company is headquartered in Oregon
○ National Chain — This location is part of a national company
○ Nonprofit — We're a registered nonprofit organization
○ Skip for now
```

---

## Migration

Existing data migration from the boolean field:

```javascript
// For each business with is_locally_owned_franchise === true:
//   Set ownership_type = 'local_franchise'
// For all others:
//   Leave ownership_type = null (unclassified)
```

The old `is_locally_owned_franchise` field can be deprecated after migration, but no rush to remove it.

---

## Implementation Order

1. Add `ownership_type` field to Business entity in Base44
2. Run migration from `is_locally_owned_franchise` boolean
3. Update BusinessEditDrawer with dropdown (replaces toggle)
4. Update AdminBusinessTable column
5. Update BusinessCard badge rendering
6. Update BusinessProfile badge rendering
7. Add Directory filter
8. Update Business Onboarding step
9. Update AdminSettingsPanel toggle

Steps 1-6 are straightforward. Step 7 (filtering) is the user-facing value. Steps 8-9 are polish.

---

*This spec is part of the LocalLane Spec-Repo. Reference before implementing.*
