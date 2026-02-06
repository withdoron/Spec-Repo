# Business Dashboard v2 — Strategy Spec

> Complete rebuild spec for the Business Dashboard.
> Replaces INTEGRATION-PLAN.md.
> Follows BUILD-PROTOCOL.md phases 0-6.
> Created: 2026-02-06

---

## Phase 0: Decision Filter

1. **Does this help a real person right now?** Yes — businesses need to see that LocalLane is driving value to them. Without a dashboard, they're flying blind.
2. **Does this make the Organism more alive?** Yes — business vitality signals (scans, revenue, engagement) are core Organism data.
3. **Does this move money closer to flowing?** Yes — the dashboard IS the revenue story. Businesses need to see their pool share to trust the model.

**Verdict:** Critical path. Build it.

---

## Phase 1: Plan

| Item | Answer |
|------|--------|
| Feature name | Business Dashboard v2 |
| One-sentence goal | Business owners log in and immediately understand: families are coming, money is flowing, and they're in control. |
| Who benefits | Business owners and staff (Standard + Partner tiers) |
| Success signal | A business owner can set Joy Coin hours, see their monthly pool share, manage events, and feel confident LocalLane is working — all in under 60 seconds. |
| Dependencies | Joy Coins data foundation (shipped), pricing locked (DEC-032/033/034), Business entity with `accepts_joy_coins` field (shipped) |
| Decisions needed | Access window data model (new entity or fields on Business?), analytics display priorities |

**Design philosophy:** The dashboard isn't a management tool — it's how businesses fall in love with LocalLane. Every screen reinforces: "This is working. Families are coming. Money is flowing. I want more of this."

---

## Phase 2: Scheme

### Existing Entities (No Changes Needed)

- **Business** — has `accepts_joy_coins`, `subscription_tier`, `owner_user_id`, `instructors`
- **Event** — has `joy_coin_enabled`, `joy_coin_cost`, `joy_coin_spots`, `joy_coin_unlimited`
- **RSVP** — has `checked_in`, `checked_in_at`, `joy_coin_total`, `party_size`
- **JoyCoinTransactions** — has `business_id`, `type`, `amount`, `event_id`
- **JoyCoinReservations** — has `business_id`, `status`, `amount`

### New Entity: AccessWindow

Businesses control when they accept Joy Coin members and at what price.

```
AccessWindow {
  id: UUID
  business_id: UUID (FK → Business)
  day_of_week: string (monday|tuesday|...|sunday)
  start_time: string (HH:MM, 24hr)
  end_time: string (HH:MM, 24hr)
  coin_cost: integer (1-3)
  capacity: integer (0 = unlimited)
  is_active: boolean
  label: string (optional — "Morning Open Play", "After School Session")
  created_date: datetime
}
```

**Why a separate entity instead of fields on Business?**
- A business may have multiple windows per day (morning off-peak = 1 coin, afternoon peak = 2 coins)
- Windows can be individually toggled on/off
- Clean query patterns for member-facing "when can I visit?" display
- Analytics can track performance per window

### State Transitions

**Access Window:** created → active ↔ inactive → deleted (soft)

**Revenue calculation flow (existing, no changes):**
```
Monthly cycle:
  Total Community Pass revenue (net after Stripe)
  × 75% = Business Pool
  ÷ Total redemptions (all businesses) = Value per redemption
  × Business's redemptions = Business payout
```

### New Hook: useAccessWindows

```
useAccessWindows(businessId) returns:
  - windows: AccessWindow[]
  - isLoading: boolean
  - createWindow(data): mutation
  - updateWindow(id, data): mutation
  - deleteWindow(id): mutation
  - toggleWindow(id): mutation
```

### New Hook: useBusinessRevenue

```
useBusinessRevenue(businessId, dateRange) returns:
  - totalRedemptions: number
  - totalCoinsRedeemed: number
  - estimatedPayout: number (coins × per-coin value)
  - redemptionsByEvent: { eventId, eventTitle, count, coins }[]
  - redemptionsByDay: { date, count }[]
  - trendDirection: 'up' | 'down' | 'flat'
  - networkComparison: { percentile, avgRedemptions }
  - isLoading: boolean
```

---

## Phase 3: Surface Mapping

| Surface | Shows Here? | What They See |
|---------|-------------|---------------|
| **MyLane** | Future phase | "Businesses near you accepting Joy Coins right now" (uses AccessWindow data) |
| **Business Dashboard** | YES — Primary surface | All 5 sections detailed below |
| **Admin Panel** | YES | Access Window overview per business, revenue reports, manual payout trigger |
| **Directory** | Already shipped | Joy Coins badge on business cards (uses `accepts_joy_coins`) |
| **Events Page** | Already shipped | Joy Coin cost badge on event cards |
| **Business Profile** | YES — Enhancement | Access windows displayed ("Open for Joy Coins: Tue-Thu 9am-12pm") |
| **Event Detail Modal** | Already shipped | Joy Coin cost and RSVP flow |
| **Onboarding Wizard** | YES — Enhancement | "Set your Joy Coin hours" step after goals (Standard tier) |
| **Settings** | Existing | Business profile settings already in dashboard |
| **Notifications** | Future phase | "New Joy Coin check-in at your business" |
| **The Good News** | Future phase | Business success stories from revenue data |

---

## Phase 4: UI/UX Design

### Dashboard Layout

The dashboard uses a **tab-based navigation** within the business context (after business selection). Five sections, each a tab:

```
┌─────────────────────────────────────────────────────────────┐
│  ← Back to Businesses    🏪 The Circuit Climbing    OWNER  │
│                          Storefront Management              │
├─────────────────────────────────────────────────────────────┤
│  [Home]  [Joy Coins]  [Revenue]  [Events]  [Settings]      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Tab content here                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Tab 1: Home (The Story)

**Purpose:** At-a-glance health check. Business owner should feel good within 5 seconds.

**Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  "This month, 47 families visited through LocalLane.        │
│   Your pool share: $176.25"                                 │
│                                                             │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  Families    │  Joy Coins   │  Pool Share  │  Events        │
│  Served      │  Redeemed    │  This Month  │  This Month    │
│     47       │     94       │   $176.25    │      8         │
│   ↑ 12%     │   ↑ 18%     │   ↑ 15%     │   → same       │
├──────────────┴──────────────┴──────────────┴────────────────┤
│                                                             │
│  Upcoming Events                              [See All →]   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Tuesday Open Climb   │ Feb 11, 10am │ 3 RSVPs │ 1🪙 │   │
│  │ Kids Wall Session    │ Feb 12, 9am  │ 7 RSVPs │ 2🪙 │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Recent Check-ins                             [See All →]   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Sarah M. + 2 │ Tuesday Open Climb │ 3 coins │ Today │   │
│  │ The Johnsons  │ Kids Wall Session  │ 4 coins │ Yest. │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  Quick Actions                                              │
│  [Create Event]  [View Access Hours]  [Check In Member]     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Narrative line at top:** Dynamic, changes based on data. Examples:
- First month: "Welcome to LocalLane! Set your Joy Coin hours to start receiving families."
- Active month: "This month, 47 families visited through LocalLane. Your pool share: $176.25"
- Growth: "Great month! Visits are up 18% — your Tuesday morning window is your top performer."
- No activity: "No Joy Coin visits yet this month. Make sure your access hours are set."

**Stat cards:** Each shows current value + trend arrow compared to prior month. Trend is only shown after the second month of data.

**Empty state (new business, no data yet):**
```
Welcome to your LocalLane dashboard!

Here's how to get started:
  1. Set your Joy Coin access hours → [Set Hours]
  2. Create your first event → [Create Event]
  3. Wait for families to discover you

Once members start visiting, you'll see your stats here.
```

---

### Tab 2: Joy Coins (The Controls)

**Purpose:** Business sets when and how they accept Joy Coin members. This is the "access window" management.

**Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Accept Joy Coins    [Toggle: ON]                           │
│                                                             │
│  When you accept Joy Coins, Community Pass families can     │
│  visit during your access hours. You get paid from the      │
│  monthly pool based on check-ins.                           │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Your Access Hours                          [+ Add Window]  │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ 🟢 Morning Open Play                                  │ │
│  │ Tue, Wed, Thu  ·  9:00 AM – 12:00 PM  ·  1 coin      │ │
│  │ Capacity: 10 families                    [Edit] [···] │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ 🟢 After School Session                               │ │
│  │ Mon – Fri  ·  3:00 PM – 5:00 PM  ·  2 coins          │ │
│  │ Capacity: Unlimited                      [Edit] [···] │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ 🟡 Weekend Climb (paused)                             │ │
│  │ Sat, Sun  ·  10:00 AM – 2:00 PM  ·  3 coins          │ │
│  │ Capacity: 5 families                     [Edit] [···] │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  💡 Pricing Insight                                        │
│  Your average coin price: 2.0                               │
│  Network average for similar businesses: 1.5                │
│  Businesses priced at 1 coin see 3.2x more monthly scans.  │
│                                                             │
│  This is your call — just sharing what the data shows.      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Add/Edit Window Modal:**

```
┌─────────────────────────────────────────────────────────────┐
│  Add Access Window                                     [X]  │
│                                                             │
│  Label (optional)                                           │
│  [Morning Open Play_____________________________________]   │
│                                                             │
│  Days                                                       │
│  [Mon] [Tue] [Wed] [Thu] [Fri] [ Sat ] [ Sun ]            │
│         ^^^   ^^^   ^^^                                     │
│                                                             │
│  Hours                                                      │
│  [9:00 AM ] to [12:00 PM]                                  │
│                                                             │
│  Joy Coin Cost                                              │
│  [ 1 coin ▼ ]                                              │
│  Off-peak (1 coin) fills more slots. Peak (2-3) earns more │
│  per visit. Find your sweet spot.                           │
│                                                             │
│  Capacity                                                   │
│  ○ Unlimited                                                │
│  ● Limited: [ 10 ] families per window                     │
│                                                             │
│               [Cancel]  [Save Window]                       │
└─────────────────────────────────────────────────────────────┘
```

**The three-dot menu (···) on each window:** Pause, Resume, Delete.

**Pricing Insight section:** Only shows after the network has enough data (10+ businesses). Before that, show a simpler message: "You set your coin price. 1 coin for off-peak hours, 2-3 for peak. Adjust anytime based on what works."

**Tier gating:** Joy Coins tab only visible for Standard+ tier. Basic tier sees a locked card: "Upgrade to Standard to accept Joy Coins and receive revenue from the Community Pass pool." with an upgrade CTA.

---

### Tab 3: Revenue (The Proof)

**Purpose:** Show the business owner that LocalLane is putting money in their pocket. This is the "fall in love" tab.

**Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Revenue from LocalLane                                     │
│  [January 2026 ▼]                                          │
│                                                             │
├──────────────────────┬──────────────────────────────────────┤
│                      │                                      │
│  Your Pool Share     │  How It Works                        │
│  ┌────────────────┐  │                                      │
│  │   $176.25      │  │  Total membership revenue: $12,500   │
│  │   this month   │  │  75% to business pool:     $9,375   │
│  │                │  │  Total network scans:        2,500   │
│  │   47 check-ins │  │  Per-scan value:             $3.75   │
│  │   94 coins     │  │  Your scans:                   47   │
│  └────────────────┘  │  Your share:               $176.25   │
│                      │                                      │
├──────────────────────┴──────────────────────────────────────┤
│                                                             │
│  Monthly Trend                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  $200 ┤                                              │   │
│  │       │            ╭──╮                              │   │
│  │  $150 ┤      ╭──╮ │  │ ╭──╮                         │   │
│  │       │ ╭──╮ │  │ │  │ │  │                         │   │
│  │  $100 ┤ │  │ │  │ │  │ │  │                         │   │
│  │       │ │  │ │  │ │  │ │  │                         │   │
│  │   $50 ┤ │  │ │  │ │  │ │  │                         │   │
│  │       └─┴──┴─┴──┴─┴──┴─┴──┴─                        │   │
│  │        Oct  Nov  Dec  Jan  Feb                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  By Event                                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Event              │ Check-ins │ Coins │ Est. Share  │   │
│  ├────────────────────┼───────────┼───────┼────────────┤   │
│  │ Tuesday Open Climb │    18     │  18   │   $67.50   │   │
│  │ Kids Wall Session  │    12     │  24   │   $45.00   │   │
│  │ Bouldering Basics  │     9     │  27   │   $33.75   │   │
│  │ Open Play Friday   │     8     │   8   │   $30.00   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  By Day of Week                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Mon ████░░░░░░  4                                   │   │
│  │  Tue ██████████████  14  ← Your best day            │   │
│  │  Wed █████████░░  9                                  │   │
│  │  Thu ████████░░░  8                                  │   │
│  │  Fri ██████░░░░░  6                                  │   │
│  │  Sat ████░░░░░░░  4                                  │   │
│  │  Sun ██░░░░░░░░░  2                                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Network Position                                           │
│  You're in the top 30% of businesses for weekday traffic.   │
│  Businesses with 1-coin off-peak windows average 2.4x more  │
│  scans than 2-coin windows.                                 │
│                                                             │
│  Your busiest window: Tue/Wed/Thu 9-12 (1 coin)            │
│  Consider: Adding a Mon/Fri morning window at 1 coin        │
│            could capture 6-8 additional visits/month.        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key design choices:**
- "How It Works" panel is always visible — businesses should understand the pool math
- Monthly trend chart only shows after 2+ months of data
- "By Event" table helps businesses understand which offerings drive traffic
- "By Day of Week" bar chart reveals the off-peak story visually
- "Network Position" section gives the elasticity signal without being pushy — "This is your call" energy, not "you should lower your price" energy

**Empty state (no revenue yet):**
```
No revenue data yet for this month.

Once Community Pass members start checking in at your events,
you'll see your pool share and analytics here.

Your current setup:
  ✓ Joy Coins: Enabled
  ✓ Access Hours: 3 windows set
  ✓ Events: 4 active events

Everything's ready — revenue will flow once members visit.
```

**Tier gating:** Revenue tab only visible for Standard+ tier. Basic tier sees the same locked upgrade card as Joy Coins tab.

---

### Tab 4: Events (The Engine)

**Purpose:** Create, edit, manage events. This is the existing functionality from INTEGRATION-PLAN.md, enhanced with Joy Coins context.

**What exists today:**
- EventsWidget.jsx — event list with basic info, "coming soon" placeholder for editor
- EventEditor.tsx.jsx — full event form, EXISTS but NOT CONNECTED
- JoyCoinsAnalyticsWidget.jsx — basic analytics

**What needs to happen:**
1. Connect EventEditor to EventsWidget (replace "coming soon" placeholder)
2. Update EventEditor to use Joy Coins terminology (not Punch Pass)
3. Add event status management (pending_review for Basic, auto-publish for Standard+)
4. Add Joy Coin cost prominently on event cards
5. Integrate check-in flow per event

**Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Your Events                                [+ Create Event]│
│                                                             │
│  [All] [Upcoming] [Past] [Drafts]                          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Tuesday Open Climb                                   │   │
│  │ Every Tuesday · 10:00 AM – 12:00 PM                  │   │
│  │ 🪙 1 coin · 3 RSVPs · Published                     │   │
│  │                          [Check In] [Edit] [···]     │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │ Kids Wall Session                                    │   │
│  │ Feb 12, 2026 · 9:00 AM – 11:00 AM                   │   │
│  │ 🪙 2 coins · 7 RSVPs · Published                    │   │
│  │                          [Check In] [Edit] [···]     │   │
│  ├──────────────────────────────────────────────────────┤   │
│  │ Advanced Bouldering                                  │   │
│  │ Feb 15, 2026 · 2:00 PM – 4:00 PM                    │   │
│  │ $15 regular price · 2 RSVPs · Published              │   │
│  │                                    [Edit] [···]      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Three-dot menu (···):** Duplicate, Cancel, Delete.

**Check In flow:** Opens the existing CheckInMode component for that event. Business staff scans or searches for the member, confirms check-in, Joy Coins are redeemed.

**Tier gating on events:**
- Basic: Can create events, status = `pending_review`, no Joy Coin pricing option
- Standard+: Auto-publish, Joy Coin pricing available, check-in available

---

### Tab 5: Settings (The Profile)

**Purpose:** Business profile management, subscription status, staff.

**What exists today:**
- StaffWidget.jsx — full staff management (add, invite, roles, auto-linking)
- Business profile editing in BusinessDashboardDetail.jsx
- Tier upgrade dialog

**Sections:**

1. **Business Profile** — Name, description, category, location, contact, photos, hours
2. **Joy Coins Settings** — Redirect to Joy Coins tab (or inline toggle)
3. **Subscription** — Current tier, renewal date, upgrade/manage CTA
4. **Staff** — Existing StaffWidget
5. **Developer Tier Override** — Hidden in production, for testing

No major changes needed here — mostly wiring up existing components into the new tab structure.

---

## Phase 5: Pre-Build Audit

### What Exists (Reuse)

| Component | File | Status | Action |
|-----------|------|--------|--------|
| BusinessDashboard.jsx | src/pages/ | Multi-business selector works | Refactor to add tab navigation |
| EventsWidget.jsx | src/components/dashboard/widgets/ | Event list works | Connect EventEditor, add Joy Coin context |
| EventEditor.tsx.jsx | src/components/dashboard/ | Full form, unused | Wire into EventsWidget |
| JoyCoinsAnalyticsWidget.jsx | src/components/dashboard/widgets/ | Basic analytics | Enhance for Revenue tab |
| StaffWidget.jsx | src/components/dashboard/widgets/ | Fully functional | Move to Settings tab |
| FinancialWidget.jsx | src/components/dashboard/widgets/ | Exists | Evaluate — may be replaced by Revenue tab |
| OverviewWidget.jsx | src/components/dashboard/widgets/ | Light theme, basic stats | Replace with Home tab |
| CheckInWidget.jsx / CheckInMode.jsx | src/components/dashboard/ | Exists | Integrate into Events tab |
| calculateRevenueShare.js | src/functions/ | Revenue calc logic | Use in Revenue tab |
| BusinessDashboardDetail.jsx | src/components/dashboard/ | Profile + tier dialog | Move profile to Settings, tier dialog reuse |

### What's New (Build)

| Component | Purpose |
|-----------|---------|
| AccessWindow entity | Base44 entity for Joy Coin hours |
| useAccessWindows hook | CRUD for access windows |
| useBusinessRevenue hook | Revenue analytics data |
| AccessWindowManager | Joy Coins tab UI |
| AccessWindowModal | Add/edit window form |
| RevenueOverview | Revenue tab UI |
| DashboardHome | Home tab UI (replaces OverviewWidget) |
| Tab navigation system | Replace widget-stack with tabs |

### What Gets Removed

| Component | Reason |
|-----------|--------|
| OverviewWidget.jsx | Replaced by DashboardHome (Home tab) |
| FinancialWidget.jsx | Replaced by RevenueOverview (Revenue tab) |
| PersonalDashboard fallback | Business Dashboard should redirect to MyLane if no business |

### Conflicts to Watch

- BusinessDashboard.jsx currently renders widgets in a vertical stack based on archetype config. New version uses tabs. This is a structural refactor of the main page.
- EventEditor.tsx.jsx uses some old Punch Pass terminology — needs Joy Coins language update during integration.
- OverviewWidget uses light theme — being replaced entirely, not patched.

---

## Phase 6: Security

### AccessWindow Entity Permissions

| Action | Who |
|--------|-----|
| Create | Business owner, manager |
| Read | Business owner, manager, staff (own business only), admin |
| Update | Business owner, manager |
| Delete | Business owner, manager |
| Public read | Members can see active windows for businesses they browse (directory/profile) |

### Route Protection (Existing)

- /business-dashboard requires: logged in + has organization + organization active
- Tab content filtered by business ownership (user can only see their own data)
- Admin override for support/debugging

### Data Exposure Rules

- Revenue data: business can only see their own pool share, not other businesses'
- Network comparison: aggregated/anonymized ("top 30%", not "Business X earned $500")
- Check-in history: business sees who checked in at their events, not other businesses' check-ins

---

## Build Sequence (For Build Chats)

This spec covers Phases 0-6. When it's time to build (Phase 7+), the sequence is:

**Build 1: Foundation**
- Create AccessWindow entity in Base44
- Create useAccessWindows hook
- Create useBusinessRevenue hook
- Security rules for AccessWindow

**Build 2: Dashboard Structure**
- Refactor BusinessDashboard.jsx to tab-based navigation
- Create DashboardHome (Home tab) — stat cards + narrative + upcoming + recent
- Wire up existing data to Home tab

**Build 3: Joy Coins Tab**
- AccessWindowManager component
- AccessWindowModal (add/edit)
- Pricing insight section
- Tier gating (Standard+ only)

**Build 4: Revenue Tab**
- RevenueOverview component
- Monthly trend chart
- By-event breakdown table
- By-day bar chart
- Network position section
- Empty states

**Build 5: Events Tab**
- Connect EventEditor to EventsWidget
- Update EventEditor Joy Coins terminology
- Add event status management per tier
- Integrate check-in flow

**Build 6: Settings Tab**
- Move profile editing to Settings
- Move StaffWidget to Settings
- Subscription status display
- Dev tier override

**Build 7: Surface Integration**
- Business Profile page — show access windows
- Onboarding Wizard — add "Set Joy Coin hours" step for Standard
- Admin Panel — access window overview, revenue reports

**Build 8: Polish + Post-Build Audit**
- Empty states for all tabs
- Mobile responsiveness
- Dark theme compliance
- Loading/error states
- Full protocol Phase 9-13 walkthrough

---

## Estimated Effort

| Build | Estimated Time |
|-------|---------------|
| Build 1: Foundation | 2-3 hours |
| Build 2: Dashboard Structure | 3-4 hours |
| Build 3: Joy Coins Tab | 3-4 hours |
| Build 4: Revenue Tab | 4-5 hours |
| Build 5: Events Tab | 2-3 hours |
| Build 6: Settings Tab | 1-2 hours |
| Build 7: Surface Integration | 2-3 hours |
| Build 8: Polish + Audit | 2-3 hours |
| **Total** | **19-27 hours** |

---

## Open Questions

| Question | Current Thinking | Decision Needed? |
|----------|-----------------|-----------------|
| Should AccessWindow capacity be per-family or per-person? | Per-person — family sizes vary, per-person gives businesses precise capacity control | Decided — per-person |
| Show actual dollar amounts in Revenue tab before Stripe is live? | Show estimated amounts based on calculation, with a note "Payouts begin when Stripe Connect is active" | Yes — before Build 4 |
| Should the Pricing Insight recommendation engine exist at launch? | No — V1 is just the business choosing 1-3 coins with a simple explanation. Intelligence layer after we have scan data. | Decided — DEC in COMMUNITY-PASS.md |
| Network comparison data — what's the minimum businesses needed? | 10+ businesses in the network before showing percentile/comparison data. Below that, hide the section. | Decided — DEC-038: minimum 10 businesses |

---

*This spec replaces INTEGRATION-PLAN.md. It follows BUILD-PROTOCOL.md phases 0-6. Build chats should reference this document for context and sequence.*
