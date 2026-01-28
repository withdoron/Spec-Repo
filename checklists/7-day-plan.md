# 7-Day Execution Plan

> Daily goals to get from "spec complete" to "pilot ready"
> Start Date: 2026-01-28 (Tomorrow)

---

## Overview

**Goal:** Event Node + Community Node working together, ready for homeschool group pilot

**Success Criteria:**
- [ ] Event created in Event Node appears in Community Node
- [ ] Punch pass (demo mode) can be "purchased" and "redeemed"
- [ ] At least one real user (homeschool group contact) can use it
- [ ] You can explain the system confidently to potential partners

---

## Day 1 (Tuesday): Foundation & Flow Verification

### Morning: Verify the Development Flow

**Goal:** Confirm Cursor → GitHub → Base44 works end-to-end

- [ ] Open Cursor with both repos (events-node, community-node)
- [ ] Make a tiny visible change in Event Node (e.g., change a button label)
- [ ] Save in Cursor
- [ ] Push to GitHub
- [ ] Open Base44
- [ ] Use Preview mode to verify the change appears
- [ ] If it works: Publish the change
- [ ] Verify it's live

**If this fails:** Stop and troubleshoot. Document any issues.

### Afternoon: Set Up Spec-Repo

- [ ] Clone Spec-Repo to Cursor
- [ ] Copy all the files I created (README.md, ARCHITECTURE.md, etc.) into it
- [ ] Push to GitHub
- [ ] Verify all files are visible at https://github.com/withdoron/Spec-Repo

### Evening: Document What Exists

- [ ] Take screenshots of current Event Node UI (for reference)
- [ ] Take screenshots of current Community Node UI
- [ ] Note what works well vs. what needs fixing
- [ ] Add notes to the archetype specs

**Day 1 Deliverable:** Working dev flow confirmed, Spec-Repo populated

---

## Day 2 (Wednesday): Event Node Polish

### Focus: Make Event Node solid before worrying about Community Node

**Goal:** Event creation and display work flawlessly

- [ ] Test event creation flow end-to-end
- [ ] Fix any bugs in event creation
- [ ] Verify events sync to Community Node
- [ ] Test event editing (does it update in Community Node?)
- [ ] Test event deletion (does it remove from Community Node?)
- [ ] Verify network tagging works (Recess, Homeschool, etc.)

### If Time Permits:

- [ ] Review Event Node code structure
- [ ] Document any technical debt or cleanup needed
- [ ] Add any missing loading/error states

**Day 2 Deliverable:** Event Node create/edit/delete fully functional

---

## Day 3 (Thursday): Community Node Assessment

### Morning: Review Current State

**Goal:** Understand what needs to happen to Community Node

- [ ] Open Community Node in Base44 preview
- [ ] Walk through every screen
- [ ] List what's working well
- [ ] List what's broken or confusing
- [ ] List what shouldn't exist (cruft)

### Afternoon: Make the Call

Based on review, decide:

**Option A: Salvage**
- Keep existing structure
- Fix the broken parts
- Delete the cruft
- Estimated effort: 2-3 days

**Option B: Rebuild Core**
- Keep UI components you like
- Rebuild the page structure
- Fresh start on navigation
- Estimated effort: 4-5 days

- [ ] Document decision in DECISIONS.md
- [ ] Create task list for chosen option

**Day 3 Deliverable:** Clear plan for Community Node (salvage or rebuild)

---

## Day 4 (Friday): Community Node Work (Part 1)

### Based on Day 3 Decision

**If Salvage:**
- [ ] Delete identified cruft
- [ ] Fix broken navigation
- [ ] Ensure events from Event Node display correctly
- [ ] Verify basic user account works

**If Rebuild:**
- [ ] Set up clean page structure
- [ ] Implement home page
- [ ] Implement events listing page
- [ ] Connect to Event Node data

### Must Have by End of Day:

- [ ] Home page loads without errors
- [ ] Events from Event Node are visible
- [ ] Basic navigation works

**Day 4 Deliverable:** Community Node showing events from Event Node

---

## Day 5 (Saturday): Punch Pass (Demo Mode)

### Focus: Get punch pass UX working without real payments

**Morning: Purchase Flow**
- [ ] User can view available punch passes
- [ ] User can "purchase" a pass (fake money, full UX)
- [ ] Pass appears in user's wallet/account
- [ ] Balance is tracked

**Afternoon: Redemption Flow**
- [ ] Event can be marked as "accepts punch pass"
- [ ] User can select punch pass at RSVP
- [ ] Balance is deducted
- [ ] Confirmation shown to user

**Keep It Simple:**
- No QR codes yet (manual selection is fine)
- No complex validation (trust the user for now)
- Demo prices shown but no real payment

**Day 5 Deliverable:** Punch pass purchase and redemption working (demo mode)

---

## Day 6 (Sunday): Integration & Testing

### Morning: End-to-End Testing

Walk through complete user journeys:

**Journey 1: Event Discovery**
- [ ] Open Community Node
- [ ] Browse events
- [ ] Filter by network (Recess, Homeschool)
- [ ] View event details
- [ ] RSVP to event

**Journey 2: Event Creation**
- [ ] Open Event Node
- [ ] Create new event
- [ ] Add to Recess network
- [ ] Enable punch pass
- [ ] Publish
- [ ] Verify appears in Community Node

**Journey 3: Punch Pass**
- [ ] Purchase punch pass on Community Node
- [ ] Find event that accepts passes
- [ ] Use punch pass to "pay" for RSVP
- [ ] Verify balance updated

### Afternoon: Fix Critical Bugs

- [ ] Fix any blockers found in testing
- [ ] Don't worry about polish — focus on "it works"

**Day 6 Deliverable:** All three journeys work end-to-end

---

## Day 7 (Monday): Pilot Prep

### Morning: Final Polish

- [ ] Fix any remaining critical bugs
- [ ] Ensure error messages are user-friendly
- [ ] Test on mobile device
- [ ] Test with slow network

### Afternoon: Prepare for Users

- [ ] Create 2-3 test events (real upcoming events if possible)
- [ ] Set up a test punch pass
- [ ] Write simple instructions for homeschool group contact
- [ ] Prepare talking points for demo

### Evening: Soft Launch

- [ ] Share link with homeschool group contact
- [ ] Walk them through creating their first event
- [ ] Get initial feedback
- [ ] Document any issues

**Day 7 Deliverable:** First real user creating real events!

---

## Daily Standup Template

At the start of each day, review:

1. **Yesterday:** What did I complete?
2. **Today:** What am I focusing on?
3. **Blockers:** What's stopping me?

Document in a simple daily log if helpful.

---

## If You Fall Behind

**Priority order (must-haves for pilot):**

1. Dev flow works (Day 1) — Can't do anything without this
2. Event creation works (Day 2) — Core feature
3. Events show in Community Node (Day 4) — Integration proof
4. Basic user can browse (Day 4) — Users need to see something

**Nice-to-haves (can defer):**
- Punch pass (can demo later)
- Advanced filtering
- Polish and animations

---

## Support

When stuck:
1. Check the Spec-Repo for guidance
2. Start a new Claude conversation, share the spec, ask for help
3. For Base44 issues: Check their docs or support

---

*Update this plan daily based on actual progress. Reality > Plan.*
