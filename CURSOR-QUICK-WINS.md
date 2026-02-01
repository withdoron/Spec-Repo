# Quick Win Fixes — Chrome UX Walkthrough

Copy each fix to Cursor independently. All four are low-risk, standalone changes.

---

## Status: ✅ ALL 4 FIXES IMPLEMENTED (2026-02-01)

Verified in codebase:
- Fix 1: Punch Pass + Settings in Layout.jsx (dropdown + mobile)
- Fix 2: Price badge in EventDetailModal when no hero image
- Fix 3: Category colors in categoryData.jsx (dark theme)
- Fix 4: Event type display — EventCard/EventDetailModal use `replace(/_+/g, ' ')`

---

## Fix 1: Add Punch Pass and Settings to User Dropdown Menu

**GOAL:** Logged-in users can access Punch Pass and a future Settings page directly from the user dropdown, on both desktop and mobile.

**WHERE:** `src/Layout.jsx`

**FIND** the desktop dropdown section — the `DropdownMenuSeparator` just before Log Out:

```jsx
<DropdownMenuSeparator className="bg-slate-800" />
<DropdownMenuItem onClick={handleLogout} className="text-red-400 hover:text-red-300 !bg-transparent hover:!bg-slate-800 focus:text-red-300 focus:!bg-slate-800 cursor-pointer">
  <LogOut className="h-4 w-4 mr-2" />
  Log Out
</DropdownMenuItem>
```

**ADD these two items BEFORE that separator** (after Business Dashboard / Admin Panel items):

```jsx
<DropdownMenuItem asChild className="text-slate-300 hover:text-amber-500 !bg-transparent hover:!bg-slate-800 focus:text-amber-500 focus:!bg-slate-800 cursor-pointer">
  <Link to={createPageUrl('PunchPass')} className="flex items-center">
    <Ticket className="h-4 w-4 mr-2" />
    Punch Pass
  </Link>
</DropdownMenuItem>
<DropdownMenuItem asChild className="text-slate-300 hover:text-amber-500 !bg-transparent hover:!bg-slate-800 focus:text-amber-500 focus:!bg-slate-800 cursor-pointer">
  <Link to={createPageUrl('MyLane')} className="flex items-center">
    <Settings className="h-4 w-4 mr-2" />
    Settings
  </Link>
</DropdownMenuItem>
```

**ALSO** add `Ticket` to the lucide-react imports at the top (Settings is already imported):
```jsx
import { Store, User, LogOut, LayoutDashboard, Shield, Calendar, Menu, Sparkles, Ticket, Settings } from "lucide-react";
```

**ALSO** add the same two items to the **mobile Sheet menu**. Find the mobile navigation section inside the `<SheetContent>` where My Lane, Business Dashboard etc. appear. Add Punch Pass and Settings links using the same `sheetLinkClass` pattern:

```jsx
<SheetClose asChild>
  <Link to={createPageUrl('PunchPass')} className={sheetLinkClass('PunchPass')}>
    <Ticket className="h-5 w-5" />
    Punch Pass
  </Link>
</SheetClose>
<SheetClose asChild>
  <Link to={createPageUrl('MyLane')} className={sheetLinkClass('MyLane')}>
    <Settings className="h-5 w-5" />
    Settings
  </Link>
</SheetClose>
```

**CONSTRAINTS:**
- Follow exact className patterns of existing dropdown/sheet items
- Settings links to MyLane for now (will get its own page later)
- Icons inherit color from text class (slate-300 default, amber-500 on hover)

---

## Fix 2: Add Price/Free Badge to Event Detail Modal When No Image

**GOAL:** The FREE / price / punch badge should always appear in the event detail modal, even when there's no hero image.

**WHERE:** `src/components/events/EventDetailModal.jsx`

**THE ISSUE:** Price badges only render inside the hero image container:
```jsx
{event.thumbnail_url && (
  <div className="relative rounded-xl ...">
    <img ... />
    <div className="absolute top-4 right-4 flex flex-col gap-2">
      {/* badges only show here */}
    </div>
  </div>
)}
```

When `thumbnail_url` is null, no badge shows at all.

**FIX:** Add a standalone badge row when there is NO hero image. Place this **immediately after** the hero image conditional block and **before** the Date & Time section (`<div className="bg-slate-800/50 rounded-xl p-5 space-y-2">`):

```jsx
{/* Price/Free badge when no hero image */}
{!event.thumbnail_url && !isCancelled && (
  <div className="flex gap-2">
    {punchPassEligible && (
      <Badge className="bg-amber-500 text-black border-0 rounded-full px-3 py-1 font-semibold">
        {punchCount === 1 ? '1 Punch' : `${punchCount} Punches`}
      </Badge>
    )}
    {!punchPassEligible && !isFree && event.price > 0 && (
      <Badge className="bg-amber-500 text-black border-0 rounded-full px-3 py-1 font-semibold">
        ${event.price.toFixed(2)}
      </Badge>
    )}
    {isFree && (
      <Badge className="bg-emerald-500 text-white border-0 rounded-full px-3 py-1 font-semibold">
        FREE
      </Badge>
    )}
  </div>
)}
```

**CONSTRAINTS:**
- Same badge classes as the existing badges in the hero overlay
- Only renders when `!event.thumbnail_url` (no duplicates when image exists)
- Don't show price badges when cancelled (cancelled badge is handled elsewhere)
- Variables `isFree`, `isCancelled`, `punchPassEligible`, `punchCount` are already defined at the top of the component

---

## Fix 3: Fix Category Card Light Backgrounds

**GOAL:** Category tiles on the Home page should use dark theme colors, not light backgrounds like `bg-yellow-50` that flash against the dark page.

**WHERE:** `src/components/categories/categoryData.jsx`

**THE ISSUE:** Every category in `mainCategories` has a `color` property with light-theme classes:
```javascript
color: 'bg-yellow-50 text-yellow-600 hover:bg-yellow-100'  // Bullion
color: 'bg-blue-50 text-blue-600 hover:bg-blue-100'        // Home Services
color: 'bg-pink-50 text-pink-600 hover:bg-pink-100'        // Childcare
// etc.
```

**FIX:** Replace the `color` value for EVERY category in the `mainCategories` array with the same Gold Standard dark theme class:

```javascript
color: 'bg-slate-800 text-amber-500 hover:bg-slate-700'
```

This applies to ALL categories — no per-category color coding (per STYLE-GUIDE.md Gold Standard rules: icons are amber or white only).

Categories to update (replace each one's `color` property):
- home_services
- auto_transportation
- farms_food
- health_fitness
- beauty_personal_care
- pets_animals
- professional_services
- bullion_coins
- education_tutoring
- childcare_family
- events_entertainment
- shopping_retail
- moving_misc

**ALSO:** Check `src/pages/Home.jsx` to see if it applies the `color` class from categoryData to the category tiles. If Home.jsx uses `category.color` for the tile container/icon wrapper, this fix cascades automatically. If Home.jsx has its own hardcoded light colors, update those to match.

**CONSTRAINTS:**
- All categories use identical dark theme color classes
- The icon inside each tile should render as `text-amber-500` (inherited from the color class)
- Hover state: `hover:bg-slate-700` for subtle lift while staying dark

---

## Fix 4: Clean Up Event Type Formatting Bug

**GOAL:** Event category tag showing "sports & & & active" should display properly as "Sports & Active" or "Sports Active".

**WHERE:** Check the event's raw data first, then the display logic.

**INVESTIGATE:**
1. Look at the event data in Base44 — what is the actual `event_type` value stored? It should be something like `sports_active`.
2. In `EventCard.jsx`, event types display with: `event.event_type.replace(/_/g, ' ')` — so `sports_active` → "sports active" (correct).
3. In the newer `EventCard.jsx` there's an `EVENT_TYPE_LABELS` map that gives proper display names: `sports_active: "Sports & Active"`.
4. If the stored value is malformed (like `sports_&_&_&_active`), fix the data.
5. If both EventCard versions exist, make sure the one being used for the public Events page uses `EVENT_TYPE_LABELS` for display.

**LIKELY FIX:** This is a data issue — one test event has a bad `event_type` value. Update it in Base44 admin to the correct value (probably `sports_active`). No code change needed.

**RESOLUTION (implemented):** EventDetailModal was using `replace(/_/g, ' & ')` which turns each underscore into " & ". So `sports___active` displayed as "sports & & & active". Changed to `replace(/_+/g, ' ')` in both EventDetailModal and EventCard to collapse multiple underscores to a single space.

---

*This spec is part of the LocalLane Spec-Repo. All 4 fixes have been implemented.*
