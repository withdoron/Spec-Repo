# Checklist: Setting Up a New Node

> Step-by-step procedure for creating a new node in the LocalLane ecosystem.
> Use this for both: (1) Cloning events-node for Tier 3 partners, or (2) Creating a new archetype from scratch.

---

## Which Scenario?

**Scenario A: Clone events-node for Tier 3 Partner** (Most Common)
- Business/organization earned Tier 3 status
- They get their own node cloned from events-node template
- **Use:** Phase 1-8 below (skip Phase 2 "Foundation Code")

**Scenario B: Create New Archetype** (Less Common)
- Building a new node type (e.g., nonprofit-node, different from events-node)
- Need to implement new features/data models
- **Use:** All phases below

---

## Pre-Flight

### For Tier 3 Partner Clone:
- [ ] **Tier 3 status confirmed** — Business has met all Tier 3 criteria
- [ ] **Partner information ready** — Business name, logo, branding preferences
- [ ] **Community Node ready** — Can accept new partner node connections
- [ ] **API credentials ready** — Will need LOCAL_LANE_API_KEY and SPOKE_ID

### For New Archetype:
- [ ] **Archetype spec complete** — Does `/archetypes/[name].md` exist and have all sections filled?
- [ ] **Decision made on tier scope** — Which tier features are we building first?
- [ ] **Test case identified** — Do we have a real user/business to test with?

---

## Phase 1: Base44 & Repository Setup

### Option A: Clone events-node (Tier 3 Partner)

- [ ] In Base44, open the **events-node** app
- [ ] Use Base44's "Duplicate App" feature
- [ ] Name it: `[business-name]-node` (e.g., `recess-node`, `jiujitsu-node`)
- [ ] Verify duplicate includes all functions, collections, pages, styling
- [ ] Set up Organization entity with partner's details:
  - Name: `[Business Name]`
  - Logo URL (if available)
  - Subscription tier: `partner` or `enterprise`
  - Networks: `["recess", "tca", etc.]`

### Option B: Create New App (New Archetype)

- [ ] Create new app in Base44
- [ ] Name it: `[archetype-name]-node` (e.g., `nonprofit-node`)
- [ ] Set up initial Organization entity

### GitHub Repository

- [ ] Create new repo in GitHub: `withdoron/[node-name]-node`
- [ ] Initialize with README
- [ ] Make repo **public** (for now, can go private later)

### Connect Base44 to GitHub

- [ ] In Base44, go to **App Settings** → **Deployment** → **GitHub**
- [ ] Connect the new GitHub repo
- [ ] Verify sync works (make small change, push, verify in Base44)

### Clone to Cursor

- [ ] In Cursor terminal, navigate to LocalLane directory:
  ```bash
  cd /Users/bombsquadgrowers/Documents/LocalLane
  ```
- [ ] Clone the repo:
  ```bash
  git clone https://github.com/withdoron/[node-name]-node.git
  ```
- [ ] Open folder in Cursor
- [ ] Copy `.cursorrules` from Spec-Repo if missing
- [ ] Run `npm install` to install dependencies

---

## Phase 2: Foundation Code (New Archetype Only)

**Skip this phase if cloning events-node** — the code is already there.

### Copy Shared Configuration

From events-node, copy:

- [ ] `tailwind.config.js` — Use exact same config for consistency
- [ ] `package.json` — Same dependencies
- [ ] `vite.config.js` — Same build setup
- [ ] `eslint.config.js` — Same linting rules
- [ ] `.gitignore` — Same ignore patterns

### Set Up Folder Structure

Create the standard node structure:

```
src/
├── components/      # Reusable UI pieces
├── pages/           # Screen-level components
├── api/             # API calls to parent/children
├── config/          # Dynamic config (pulled from parent)
├── hooks/           # Shared React hooks
└── utils/           # Helper functions
functions/           # Backend/serverless functions
```

- [ ] Create folders
- [ ] Add placeholder files as needed

### Configure Parent Node Connection

- [ ] Set up config sync (pull from Community Node)
- [ ] Implement config caching (localStorage)
- [ ] Test config fetch works

---

## Phase 3: Configuration (Tier 3 Partner Clone)

**For cloning events-node, configure these:**

### Environment Variables

Create `.env.local` file:

```bash
VITE_BASE44_APP_ID=[new_app_id_from_base44]
VITE_BASE44_APP_BASE_URL=[new_base_url_from_base44]
```

**Important:** These are different from events-node!

### Base44 Secrets (Set in Base44 Dashboard)

Configure these secrets in Base44 → App Settings → Secrets:

- [ ] `LOCAL_LANE_API_KEY` — API key for Community Node (same for all nodes)
- [ ] `SPOKE_ID` — **Unique ID for this partner node** (get from Community Node admin)
- [ ] `SPOKE_WEBHOOK_SECRET` — **Unique secret for webhooks** (generate new one)

### Update Code References (If Needed)

- [ ] **README.md** — Update with partner name and description
- [ ] **Organization name** — Update fallback text if not using dynamic (currently "Recess")
- [ ] Verify `SPOKE_ID` is read from env vars (should already be in functions)

### Files That Must Stay Identical

These should match events-node exactly:

- ✅ `tailwind.config.js` — Must match for consistency
- ✅ `package.json` — Same dependencies
- ✅ `vite.config.js` — Same build config
- ✅ `src/components/` — All component patterns
- ✅ `src/api/base44Client.js` — Same API setup
- ✅ All styling — Follows STYLE-GUIDE.md

---

## Phase 4: Core Features (New Archetype Only)

**Skip this phase if cloning events-node** — features already exist.

### Implement Data Model

Based on archetype spec:

- [ ] Define data schema
- [ ] Set up Base44 collections/tables
- [ ] Create CRUD operations

### Build Key Screens

Based on archetype spec "Key Screens":

- [ ] Screen 1: _______________
- [ ] Screen 2: _______________
- [ ] Screen 3: _______________
- [ ] Screen 4: _______________

### Implement API Integration

- [ ] Push data to Community Node (POST)
- [ ] Pull config from Community Node (GET)
- [ ] Handle errors gracefully (retry, cache)

---

## Phase 5: Styling & Polish

### Apply Style Guide

- [ ] Verify colors match STYLE-GUIDE.md
- [ ] Verify typography is consistent
- [ ] Verify spacing follows patterns
- [ ] Mobile responsive works

### Add Loading & Error States

- [ ] Loading spinners/skeletons
- [ ] Error messages (user-friendly)
- [ ] Empty states ("No events yet")

---

## Phase 6: Community Node Registration

### Register Partner Node

- [ ] In Community Node admin, register new partner node:
  - Node ID: `[node-name]-node` (matches GitHub repo name)
  - SPOKE_ID: `[unique_id]` (matches Base44 secret)
  - Business Name: `[Partner Name]`
  - API Key: `[LOCAL_LANE_API_KEY]`
  - Webhook URL: `https://[node-name].base44.app/functions/handleLocalLaneWebhook`
  - Webhook Secret: `[SPOKE_WEBHOOK_SECRET]`

### Test Connection

- [ ] Test config sync:
  - Should fetch networks, categories from Community Node
  - Should cache config locally
- [ ] Test data sync:
  - Create test item (event, listing, etc.)
  - Verify appears in Community Node
  - Verify sync status shows "synced"

---

## Phase 7: Testing

### Local Testing

- [ ] Run `npm run dev` locally
- [ ] Verify app loads without errors
- [ ] Test key workflows (create, edit, delete)
- [ ] Test sync to Community Node
- [ ] Test config sync from Community Node

### Base44 Preview

- [ ] Push code to GitHub
- [ ] Wait for Base44 sync (~1-2 min)
- [ ] Use Base44 Preview mode
- [ ] Test all key workflows
- [ ] Verify styling matches Gold Standard

### Integration Testing

- [ ] Create item → Verify appears in Community Node
- [ ] Update item → Verify updates in Community Node
- [ ] Delete item → Verify removed from Community Node
- [ ] Test what happens if Community Node is down
- [ ] Test on mobile device
- [ ] Test with slow network

### User Testing

- [ ] Have test user try key workflows
- [ ] Document feedback
- [ ] Prioritize fixes

---

## Phase 8: Launch

### Pre-Launch Checklist

- [ ] All critical bugs fixed
- [ ] README updated with node-specific info
- [ ] Archetype spec updated with implementation status (if new archetype)
- [ ] DECISIONS.md updated with any decisions made
- [ ] Partner has reviewed and approved (if Tier 3 clone)
- [ ] Community Node shows node as active

### Publish

- [ ] In Base44, click **Publish**
- [ ] Verify live URL works
- [ ] Share URL with users/partner
- [ ] Monitor for issues first 24 hours

---

## Post-Launch

### Documentation

- [ ] Update Spec-Repo with new node info
- [ ] Document any customizations made
- [ ] Note any issues encountered for future reference

### Monitoring

- [ ] Check sync status regularly first week
- [ ] Monitor for errors in Base44 logs
- [ ] Gather user feedback
- [ ] Plan improvements

---

## Common Issues & Solutions

### Issue: Sync Not Working

**Check:**
- [ ] SPOKE_ID matches Community Node registration
- [ ] LOCAL_LANE_API_KEY is correct
- [ ] Community Node is accessible
- [ ] Base44 secrets are set correctly

### Issue: Config Not Loading

**Check:**
- [ ] Community Node `/api/v1/config` endpoint works
- [ ] Partner node can reach Community Node URL
- [ ] Config cache is being used if parent is down

### Issue: Items Not Appearing in Community Node

**Check:**
- [ ] Item status is "published" (not "draft")
- [ ] Sync function is being called
- [ ] Community Node is receiving POST requests
- [ ] Check Base44 function logs for errors

---

## Quick Reference

### Environment Variables Needed

```bash
# In .env.local (for local dev)
VITE_BASE44_APP_ID=[app_id]
VITE_BASE44_APP_BASE_URL=[base_url]

# In Base44 Secrets (for production)
LOCAL_LANE_API_KEY=[shared_key]
SPOKE_ID=[unique_per_partner]  # Only for Tier 3 clones
SPOKE_WEBHOOK_SECRET=[unique_per_partner]  # Only for Tier 3 clones
```

### Key Files to Update (Tier 3 Clone)

1. `.env.local` — Local development
2. Base44 Secrets — Production environment
3. `README.md` — Documentation
4. Organization entity in Base44 — Partner details

### Key Files to Keep Identical

1. `tailwind.config.js` — Style consistency
2. `package.json` — Dependency consistency
3. Component patterns — Follow COMPONENTS.md
4. API patterns — Follow ARCHITECTURE.md

---

*Use this checklist every time you create a new node. For Tier 3 partners, clone events-node. For new archetypes, build from scratch following the archetype spec.*
