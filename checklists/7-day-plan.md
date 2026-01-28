# 7-Day Pilot Launch Plan

> Revised execution plan for homeschool pilot launch.
> Updated: 2026-01-28

---

## Goal

**Get a working event flow for the homeschool group pilot:**
- Event organizers can create/manage events
- Events appear on Community Node
- Users can browse and RSVP
- Punch Pass demo mode functional

---

## Day 1 (Today) ✅ COMPLETE

### Objective: Foundation & Documentation

- [x] Spec-Repo populated with all documents
- [x] Push to GitHub
- [x] Style guide complete with Gold Standard
- [x] Components guide created
- [x] Development workflow documented
- [x] Current state assessed via screenshots
- [x] Google Docs mirror set up

### Deliverables
- Spec-Repo live on GitHub
- Google Docs mirror readable by Claude
- Clear understanding of current state

---

## Day 2: Community Node Cleanup

### Objective: Clean up Community Node, fix visual inconsistencies

### Tasks

**Morning: Visual Fixes**
- [ ] Fix Admin Panel to dark theme
  - Change `bg-white` to `bg-slate-950`
  - Update all light-colored elements
  
- [ ] Fix organizer name display
  - Event detail shows "Event Spoke" → should show business name
  - Trace data flow: Event Node → sync → Community Node display
  - Ensure business name is included in sync payload

**Afternoon: Remove Unused Features**
- [ ] Audit existing pages/features in Community Node
- [ ] Hide/remove features not needed for pilot:
  - [ ] Directory (defer to post-pilot)
  - [ ] Boost feature (outdated concept)
  - [ ] Any other cruft identified
  
- [ ] Simplify navigation to pilot essentials:
  - Browse Events
  - My Lane (user dashboard)
  - Dashboard (organizer)

**Evening: Verification**
- [ ] Test event sync still works
- [ ] Verify dark theme throughout
- [ ] Screenshot key screens for review

### Deliverables
- Community Node visually consistent (dark theme)
- Organizer name displays correctly
- Unused features hidden
- Clean navigation

---

## Day 3: User Flow

### Objective: Enable user registration and basic interaction

### Tasks

**Morning: User Authentication**
- [ ] Verify user registration flow works
- [ ] Verify login/logout works
- [ ] Test password reset (if implemented)

**Afternoon: User Features**
- [ ] RSVP to events (basic)
  - User clicks "RSVP" on event
  - Shows confirmation
  - Event shows RSVP count
  
- [ ] "My Events" or "My Saved" view
  - User can see events they've RSVPed to
  - Can cancel RSVP

**Evening: Testing**
- [ ] Create test user account
- [ ] Go through full flow: register → browse → RSVP → view my events
- [ ] Fix any issues

### Deliverables
- Users can register/login
- Users can RSVP to events
- Users can see their RSVPed events

---

## Day 4: Punch Pass Demo Mode

### Objective: Implement Punch Pass purchase and redemption (demo only)

### Tasks

**Morning: Purchase Flow**
- [ ] Create Punch Pass purchase UI
  - Show pack options (10 for $100, 20 for $180, 30 for $255)
  - "Demo Mode" badge clearly visible
  - Mock checkout process (no real payment)
  
- [ ] Store purchased passes in user account
  - Track balance (punches remaining)
  - Track expiration (12 months from "purchase")

**Afternoon: Redemption Flow**
- [ ] At RSVP time, show Punch Pass option
  - "Use 3 Punches" button if user has balance
  - Or "Pay $30" if no balance
  
- [ ] Deduct punches on RSVP
  - Update balance
  - Show confirmation

**Evening: Verification**
- [ ] Test full flow: Buy pack → RSVP with punches → Balance updates
- [ ] Verify FIFO logic (oldest pack used first)
- [ ] Handle edge cases (not enough punches, expired pack)

### Deliverables
- Users can "purchase" punch packs (demo)
- Users can see their balance
- Users can redeem punches at RSVP
- Clear "Demo Mode" indication throughout

---

## Day 5: Event Node Polish

### Objective: Ensure Event Node is production-ready

### Tasks

**Morning: Sync Verification**
- [ ] Create test event in Event Node
- [ ] Verify it appears in Community Node
- [ ] Verify all fields sync correctly:
  - Title, description
  - Date, time, duration
  - Location
  - Image
  - Punch cost
  - Categories
  - Networks (Recess, etc.)
  - **Business name** (not "Event Spoke")

**Afternoon: Edge Cases**
- [ ] Test event update → sync updates
- [ ] Test event delete → removes from Community
- [ ] Test draft events (don't sync until published)

**Evening: Dashboard Polish**
- [ ] Verify stats are accurate
- [ ] Verify Punch Pass earnings display
- [ ] Clean up any visual issues

### Deliverables
- Event sync is bulletproof
- All event fields display correctly
- Event Node dashboard is clean

---

## Day 6: End-to-End Testing

### Objective: Complete flow testing, fix critical bugs

### Tasks

**Morning: Happy Path Testing**
- [ ] **Organizer flow:**
  1. Login to Event Node
  2. Create event with all fields
  3. Publish event
  4. Verify appears in Community Node
  
- [ ] **User flow:**
  1. Register on Community Node
  2. Browse events
  3. View event detail
  4. Purchase Punch Pack (demo)
  5. RSVP with punches
  6. View My Events

**Afternoon: Edge Case Testing**
- [ ] User with no punches tries to use Punch Pass
- [ ] Event at capacity (if implemented)
- [ ] User tries to RSVP twice
- [ ] Mobile browser testing (iOS Safari, Android Chrome)

**Evening: Bug Fixes**
- [ ] Fix any critical bugs found
- [ ] Document known issues (non-blocking)
- [ ] Update Spec-Repo with any decisions made

### Deliverables
- All happy paths work
- No critical bugs
- Known issues documented

---

## Day 7: Pilot Prep & Soft Launch

### Objective: Prepare for real users, soft launch

### Tasks

**Morning: Content**
- [ ] Add 5-10 real events for pilot
  - Mix of free and paid
  - Different categories
  - Different networks
  
- [ ] Verify realistic data displays well
- [ ] Remove any test data

**Afternoon: User Prep**
- [ ] Create simple user guide (1-page)
  - How to browse events
  - How to RSVP
  - How to use Punch Pass
  
- [ ] Set up feedback collection
  - Google Form or similar
  - Link in app footer
  
- [ ] Brief pilot users (homeschool group leaders)

**Evening: Launch**
- [ ] Final verification
- [ ] Publish to production
- [ ] Share link with pilot group
- [ ] Monitor for immediate issues

### Deliverables
- Real events populated
- User guide ready
- Feedback mechanism in place
- Soft launch complete

---

## Success Criteria

By end of Day 7:

- [ ] Event created in Event Node appears in Community Node within 5 minutes
- [ ] User can register, browse, and RSVP
- [ ] Punch Pass demo mode works end-to-end
- [ ] At least 5 real events from homeschool group
- [ ] At least 3 pilot users have successfully used the system
- [ ] No critical bugs blocking basic flow

---

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| Sync breaks | Have manual entry fallback in Community Node |
| Auth issues | Test thoroughly on Day 3, have backup login method |
| Punch Pass complexity | Keep demo mode simple, defer edge cases |
| Time overrun | Cut Punch Pass to Day 8 if needed, launch with just RSVP |

---

## Daily Standup Questions

At the end of each day, answer:
1. What did I complete today?
2. What's blocking me?
3. Am I on track for Day 7 launch?

---

## Post-Pilot (Day 8+)

After successful pilot:
- [ ] Gather feedback
- [ ] Prioritize fixes based on user input
- [ ] Begin Phase 2: Real payments, more features
- [ ] Consider Nonprofit Node archetype

---

*Update this document as tasks complete. Check off items as you go.*
