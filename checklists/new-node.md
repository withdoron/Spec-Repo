# Checklist: Setting Up a New Node

> Step-by-step procedure for creating a new node in the LocalLane ecosystem.

---

## Pre-Flight

- [ ] **Archetype spec complete** — Does `/ARCHETYPES/[name].md` exist and have all sections filled?
- [ ] **Decision made on tier scope** — Which tier features are we building first?
- [ ] **Test case identified** — Do we have a real user/business to test with?

---

## Phase 1: Repository Setup

### Create the Repo

- [ ] Create new repo in GitHub: `withdoron/[node-name]-node`
- [ ] Initialize with README
- [ ] Make repo public (for now)

### Clone Base44 Template (If Applicable)

- [ ] Create new app in Base44
- [ ] Connect to GitHub repo
- [ ] Confirm sync works (make small change, push, verify in Base44)

### Set Up in Cursor

- [ ] Clone repo to Cursor workspace
- [ ] Verify you can edit and push

---

## Phase 2: Foundation Code

### Copy Shared Configuration

From an existing node (events-node), copy:

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

## Phase 3: Core Features

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

## Phase 4: Styling & Polish

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

## Phase 5: Testing

### Manual Testing

- [ ] Create test data
- [ ] Walk through all workflows in archetype spec
- [ ] Test on mobile device
- [ ] Test with slow network (throttle in dev tools)

### Integration Testing

- [ ] Verify data syncs to Community Node
- [ ] Verify config pulls from Community Node
- [ ] Test what happens if Community Node is down

### User Testing

- [ ] Have test user try key workflows
- [ ] Document feedback
- [ ] Prioritize fixes

---

## Phase 6: Launch

### Pre-Launch Checklist

- [ ] All critical bugs fixed
- [ ] README updated with node-specific info
- [ ] Archetype spec updated with implementation status
- [ ] DECISIONS.md updated with any decisions made

### Soft Launch

- [ ] Publish to Base44
- [ ] Share with test users only
- [ ] Monitor for issues
- [ ] Gather feedback

### Full Launch

- [ ] Announce to broader audience
- [ ] Update Community Node to display new node type
- [ ] Monitor metrics for first week
- [ ] Schedule feedback review session

---

## Post-Launch

- [ ] Document lessons learned
- [ ] Update archetype spec with any changes
- [ ] Plan Phase 2 features
- [ ] Consider what can be reused for next archetype

---

*Use this checklist every time you create a new node. Copy to a new file and check off as you go.*
