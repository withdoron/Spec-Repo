# Phase 4 Migration Plan — Pre-Build Audit

> Authored by Hyphae 2026-04-26. Amended 2026-04-26 (evening) with Mycelia + Doron planning decisions in Section 8.
> Maps the current `community-node/src/` structure against the folder model in
> `LOCALLANE-CORE-ARCHITECTURE-v4-1.md` (Sections 10.1, 13.1, 14, 17, 20).
> Docs-only artifact. No code touched. Input to future Phase 4 execution prompts.

---

## Section 1 — v4.1's Folder Model (Summarized)

v4.1 reframes navigation as a **folder tree** rendered by a **cockpit**. Every folder either contains sub-folders or a workspace; the cockpit is the chrome that lets the user move through that tree. At the root, the cockpit shows **universal folders** (Directory, Events, Personal, Businesses) plus **contextual folders** the user has earned (Admin if admin, Playmaker if any Playmaker role, Networks if joined any). Inside a folder, the cockpit shows that folder's contents — which may themselves be folders (Businesses → your businesses → one business → its spaces) or, eventually, workspaces.

The viewport below the cockpit changes meaning with depth. **At leaves** (a specific space of a specific business: Desk, Finance, Profile, Settings) the area below the cockpit is the actual workspace. **Above leaves** the area below shows a **preview pulse** — living-tile language describing what's alive in the centered folder, so the user has context before entering it. Different cockpit styles (Spinner, Compass, future Voice) render the same folder tree in their own visual languages.

Every folder and workspace is **URL-addressable** (Section 14.3): `/mylane`, `/mylane/personal`, `/mylane/personal/finance`, `/mylane/businesses`, `/b/{slug}`, `/b/{slug}/desk`. URL is the **primary source of scope** — `useActiveBusiness` reads URL first, localStorage second. The `/b/{slug}` form is a "direct door" — same URL, auth-aware rendering (public profile for visitors, tools for operators). Per **DEC-179** (logged 2026-04-25, after v4.1 was written), the path-based direct doors are deferred to post-migration so we don't pay the fragility cost twice; the folder-tree restructure proceeds independently inside `/mylane`.

Three load-bearing sub-principles tie this together:

- **Cockpits as folder renderers (10.1).** The cockpit is one component that knows how to render a folder, not a screen-per-space.
- **Hidden entities (13.1).** `listed_in_directory: false` is the sole hide mechanism. Directory queries filter on `!== false`. Owners' surfaces never apply the filter.
- **Resurface before rebuild (20).** When a new architecture needs a previously-built surface (admin, feedback affordance), find the existing component first and rewire it. Only rebuild if genuinely lost. This audit is the first concrete application of that principle to Phase 4.

---

## Section 2 — Current MyLane Structure Inventory

This walks the files involved in MyLane navigation, surface rendering, drill-through, business-context switching, and the dashboard tab structure. One line each — what it does, no critique.

### Router & shell

- `src/App.jsx` (237 lines) — React Router root. PUBLIC_PAGES set. Wraps every route in `<Layout>` unless a route opts out. Authenticated `/` redirects to `/MyLane`. Routes: `/`, `/{PageName}` (PascalCase, from `pages.config.js`), `/Events/:eventId`, `/my-lane/transactions`, `/networks`, `/networks/:slug`, `/claim-business`, `/join/:inviteCode`, `/door/:slug`, `/join-field-service/:inviteCode`, `/join-pm/:inviteCode`, `/client-portal/...`, `/frequency`, `/frequency/:slug`. **No `/b/{slug}` or `/u/{slug}` routes.** **No nested `/mylane/...` routes.**
- `src/Layout.jsx` (284 lines) — Top header chrome (logo, Directory/Events nav links, avatar dropdown to MyLane / Admin / Settings / Logout, mobile sheet). Hidden on `/MyLane` and on the unauthenticated `/Home` page. Footer also hidden on `/MyLane`.
- `src/pages.config.js` (60 lines) — PAGES map. `mainPage: "MyLane"`. `Layout: __Layout`. Top-level note: `BusinessDashboard retired (DEC-131)`.

### MyLane page + surface

- `src/pages/MyLane.jsx` (364 lines) — Wraps `<MyLaneSurface>`. Inline welcome flow for cold users (name capture, `onboarding_complete` write). Pulls workspace-profile data via `getMyLaneProfiles` server fn (DEC-130 single-call pattern), with per-entity fallback. Redirects to invite/door routes if pending invite is in localStorage.
- `src/components/mylane/MyLaneSurface.jsx` (1385 lines) — The big surface. Holds:
  - `OverlayContainer` (the DEC-148 containment primitive — backdrop + panel below header, above bottom UI stack).
  - `AccountOverlay` (Settings, Theme picker, Cockpit picker, Sound, Newsletter, Philosophy, Support, Terms, Privacy, Logout — rendered inside the avatar overlay).
  - `OV` constant (FREQ, DIR, EVT, ACCT, PHILOSOPHY, SUPPORT, NETWORK) — single source of overlay names per DEC-146.
  - `SPACE_CONFIG` map and `buildSpinnerItems()` — produces the cockpit's spinner items from active workspace profiles + ownedBusinesses (home, meal-prep, field-service[Desk], finance, team, business, property-pulse, discover, dev-lab[admin]).
  - `buildBusinessSwitcherItems()` — recasts ownedBusinesses as spinner items for switcher mode (DEC-168).
  - Header (logo, Music icon, Directory pill, Events pill, avatar) with "operating as" label in switcher mode.
  - State for `switcherMode`, `spinnerIndex`, `drilledTab`, `activeOverlay`, plus stacked overlays `overlayBusinessId` / `overlayRecommend` / `overlayNetworkSlug`.
  - Renders `<SpaceSpinner>` (cockpit), then `<HomeFeed>` / `<DiscoverPosition>` / `<MyLaneDrillView>` / `<DevLab>` based on which spinner position is active.
  - Lazy-imports overlay pages (`Directory`, `Events`, `Settings`, `BusinessProfile`, `Philosophy`, `Support`, `Recommend`, `NetworkPage`, `FrequencyStation`).
- `src/components/mylane/SpaceSpinner.jsx` (870 lines) — The cockpit itself. `data-cockpit` attribute + MutationObserver mirror the theme plumbing (DEC-152). Variant render functions: `renderFlat`, `renderDrum`, `renderCoverFlow`, `renderCompass`. `VARIANT_MAP` registers them. Per-theme physics (mass + friction). Audio tick + haptics on each crossing. Switcher mode reuses the same component — businesses ride the same spinner as spaces.
- `src/components/mylane/MyLaneDrillView.jsx` (265 lines) — Renders a workspace's tabs inside MyLane. `WORKSPACE_KEY_MAP` translates drill workspace name → `WORKSPACE_TYPES` key. Pulls the right profile from props or `useActiveBusiness`. For business workspace: fetches business events, RSVP counts, revenue, builds scope, renders `TabComponent`. Tab bar at top, tab content below.
- `src/components/mylane/HomeFeed.jsx` (271 lines) — Home position content. Three tabs: Attention | This week | Spaces. Each tab feeds a `PrioritySpinner` of items derived from the user's profiles and urgency signals (per DEC-131).
- `src/components/mylane/DiscoverPosition.jsx` (184 lines) — Discover position content (the dim spinner item).
- `src/components/mylane/DevLab.jsx` (165 lines) — Admin-only Dev Lab spinner position content (physics tuner, etc.).
- `src/components/mylane/CommandBar.jsx` (238 lines) — Bottom-docked command/agent input. Allowlist-gated to MYLANE_AGENT_ALLOWLIST per DEC-147.
- `src/components/mylane/PrioritySpinner.jsx` (122 lines) — Vertical scroll/spin used inside HomeFeed.
- `src/components/mylane/HouseholdManager.jsx` (300 lines) — Household manager UI; imported by `pages/Settings.jsx`.
- `src/components/mylane/SectionWrapper.jsx` (26 lines) — Tiny presentational wrapper.
- `src/components/mylane/ConfirmationCard.jsx` (147 lines) — Renders agent's TYPE 3 RENDER_CONFIRM cards.
- `src/components/mylane/parseRenderInstruction.js` (100 lines) — Parses HTML-comment render instructions (TYPE 1 RENDER, TYPE 2 RENDER_DATA) from agent messages, per DEC-111.
- `src/components/mylane/renderEntityView.jsx` (372 lines) — Universal renderer for TYPE 2 RENDER_DATA. Holds the `HIDDEN_FIELDS` Set (33 internal fields excluded from cards).
- `src/components/mylane/useMyLaneState.js` (224 lines) — Card-reorder state, time-aware urgency tracking, last-visited tracking.
- `src/components/mylane/cards/*` (7 files, 41–176 lines each) — Card components: `ActiveProjectsCard`, `EnoughNumberCard`, `PendingEstimatesCard`, `PlayerReadinessCard`, `PropertyOverviewCard`, `RecipeBookCard`, `RemindersCard`. Registered by `myLaneRegistry.js` (in `config/`, **not** `components/mylane/`).

### Business-context machinery

- `src/hooks/useActiveBusiness.js` (81 lines) — Single source of truth for "which business am I operating as." Reads localStorage key `locallane.activeBusinessId`. Validates against owned-businesses query on every render. Falls back to oldest owned by `created_date`. Cross-component updates via `locallane-active-business-change` event. **localStorage is the primary source today.**
- `src/hooks/useMylane.js` (65 lines) — Mylane access tier (`basic` / `beta` / `admin`).
- `src/hooks/useBusinessRevenue.js` — Business revenue analytics (consumed by `MyLaneDrillView`).
- `src/components/business/BusinessCard.jsx` — Generic business card preview.
- `src/components/business/JoyCoinHours.jsx` — Joy coin hours widget.
- `src/components/business/SlugMultiSelect.jsx` — Multi-slug select (DEC-176, used for `subcategories[]` and `service_area`).
- `src/components/business/rankingUtils.jsx` — Directory ranking utilities.

### Workspace types config + tab implementations

- `src/config/workspaceTypes.js` (302 lines) — Tab registry per workspace type. Maps each workspace type (business, team, finance, fieldservice, property_management, meal_prep) to its tab list (id, label, icon, component, getProps). `BUSINESS_TABS = [Home, Joy Coins, Revenue, Events, Settings]`. `ARCHETYPE_TITLES` for subtitle copy.
- `src/components/dashboard/DashboardHome.jsx` (203 lines) — Business-workspace "Home" tab (stats, upcoming events, quick actions).
- `src/components/dashboard/BusinessSettings.jsx` (1469 lines) — Business-workspace "Settings" tab. **Holds the `listed_in_directory` toggle (line 247).** Holds the universal services editor, tagline, gallery, accepts toggles (Build B + Build E + Build F).
- `src/components/dashboard/AccessWindowManager.jsx` (366 lines) — "Joy Coins" tab.
- `src/components/dashboard/AccessWindowModal.jsx` (225 lines) — Joy Coins access-window modal.
- `src/components/dashboard/RevenueOverview.jsx` (241 lines) — "Revenue" tab.
- `src/components/dashboard/widgets/EventsWidget.jsx` (541 lines) — "Events" tab.
- `src/components/dashboard/widgets/StaffWidget.jsx` (612 lines) — Staff management widget.
- `src/components/dashboard/EventEditor.jsx` (1694 lines) — Event editor modal.
- `src/components/dashboard/IdeasBoard.jsx` (672 lines) — Ideas board.
- `src/components/dashboard/DashboardSettings.jsx` (48 lines) — Dashboard-level settings shim.
- `src/components/dashboard/BusinessCard.jsx` (97 lines) — Dashboard-flavored business card.
- Per-workspace folders (not catalogued line-by-line, but their tab components are imported by `workspaceTypes.js`): `src/components/team/*`, `src/components/finance/*`, `src/components/fieldservice/*`, `src/components/propertymgmt/*`, `src/components/mealprep/*`.

### Pages used by MyLane (rendered as overlays per DEC-148, also reachable as standalone routes)

- `src/pages/Home.jsx` (384 lines) — Public landing page (unauthenticated). Authenticated users redirect to `/MyLane`.
- `src/pages/Directory.jsx` (289 lines) — Directory surface. Filters via `directoryVisibility.js`.
- `src/pages/Events.jsx` (306 lines) — Events surface.
- `src/pages/BusinessProfile.jsx` (730 lines) — A business's public profile (the future "Profile space"). Stacked overlay drill-in target.
- `src/pages/Settings.jsx` (302 lines) — User-level settings (name, phone, region, household).
- `src/pages/Admin.jsx` (357 lines) — Admin panel; bird's-eye admin hub. Standalone page; reachable via avatar-dropdown link. Not folded into MyLane today.
- `src/pages/Networks.jsx`, `src/pages/NetworkPage.jsx` — Networks list + detail.
- `src/pages/FrequencyStation.jsx`, `src/pages/SongDetail.jsx` — Frequency Station.
- `src/pages/Recommend.jsx`, `src/pages/Philosophy.jsx`, `src/pages/Support.jsx`, `src/pages/Privacy.jsx`, `src/pages/Terms.jsx` — Identity + legal pages.
- `src/pages/JoyCoinsHistory.jsx` — Joy Coins history.
- Onboarding pages: `BusinessOnboarding.jsx` (517 lines, creates Business but does **not** set any `enabled_spaces` field), `UserOnboarding.jsx`, `TeamOnboarding.jsx`, `FinanceOnboarding.jsx`, `FieldServiceOnboarding.jsx`, `PropertyManagementOnboarding.jsx`, `MealPrepOnboarding.jsx`.
- `src/pages/JoinTeam.jsx`, `src/pages/JoinFieldService.jsx`, `src/pages/JoinPM.jsx`, `src/pages/ClaimBusiness.jsx`, `src/pages/ClientPortal.jsx` — Invite/join flows.

### Registry + utils

- `src/config/myLaneRegistry.js` (98 lines) — MyLane card registry (7 entries; cards live under `components/mylane/cards/` but the registry lives in `config/`).
- `src/utils/directoryVisibility.js` (~22 lines) — Single filter helper for `listed_in_directory !== false`. Used by Directory, NetworkPage, Home.

### Files mentioned in the brief that **do not exist** at expected paths

The brief's prior-session list named the following — confirmed absent at those paths and not relocated:

- `MyLaneBreadcrumb.jsx` — does not exist anywhere in the codebase.
- `WhatsChangedBar.jsx` — does not exist anywhere in the codebase.
- `BusinessDashboard.jsx` — retired (DEC-131); replaced by `MyLaneDrillView` + `WORKSPACE_TYPES.business`.
- `OverlayContainer.jsx` — exists, but as an inline component **inside** `MyLaneSurface.jsx` (lines 98–153), not as a separate file.
- `BusinessSwitcher.jsx` — does not exist as a separate component. Switcher mode is state inside `MyLaneSurface` that re-uses `SpaceSpinner` with switcher items (DEC-168).

These five "missing" names are not gaps in v4.1 — they're stale references in earlier session logs. Phase 4 should not assume any of them exist.

---

## Section 3 — Mapping Current Files to v4.1 Concepts

Each row pairs a v4.1 concept with the current file(s) and a category: **DIRECT** (reuse as-is or with light edits), **MOVE/RENAME** (move or rename), **PARTIAL** (split, augment, or refactor), **NET-NEW** (no current implementation), **RETIRE** (concern is gone in v4.1).

### Cockpit / folder rendering (Section 10.1)

| v4.1 concept | Current file(s) | Category | Notes |
|---|---|---|---|
| Cockpit as folder renderer (one component, not screen-per-space) | `components/mylane/SpaceSpinner.jsx`, `components/mylane/MyLaneSurface.jsx` | **PARTIAL** | SpaceSpinner already renders any item array as cockpit chrome (spaces, businesses in switcher mode). The "folder" abstraction isn't named, but the structural pattern is already there. MyLaneSurface decides which items to feed it — that decision logic is the part that needs the folder-tree shape. |
| Cockpit registry (Spinner / Compass / future Voice) | `SpaceSpinner.jsx` `VARIANT_MAP` | **DIRECT** | DEC-152 already implemented: `VARIANT_MAP = { flat, drum, coverFlow, compass }`. Adding a new cockpit is a render function and a COCKPITS entry. Nothing to change. |
| Folder tree as data structure (root → folder → sub-folder → workspace) | none | **NET-NEW** | The current code computes spinner items imperatively from profile arrays inside `buildSpinnerItems`. There is no declared tree. Phase 4 needs a tree representation (likely a config or builder fn) that the cockpit renders. |
| "Cockpit dissolves laterally into sibling-selection" (switcher mode) | `MyLaneSurface.jsx` `switcherMode` state + `buildBusinessSwitcherItems` | **DIRECT** | DEC-168 cockpit-native switcher already does this for businesses. Same pattern generalizes to any folder where lateral switching makes sense (the v4.1 hint at the end of 10.1). Reuse, don't rebuild. |
| Workspace at the leaf | `components/mylane/MyLaneDrillView.jsx` + `config/workspaceTypes.js` tabs | **DIRECT** | When a space is the active spinner item and the user drills into it, MyLaneDrillView renders the workspace's tabs below the cockpit. v4.1's "leaf = workspace below cockpit" is what's already shipping. |
| Preview pulse at non-leaf levels | none | **NET-NEW** | Today, when no space is "drilled," the home position shows `HomeFeed` (Attention/Week/Spaces) and the Discover position shows `DiscoverPosition`. There is no per-folder preview-pulse component. **Ambiguous:** Hyphae cannot tell from v4.1 alone whether `HomeFeed` is meant to remain as the Home space's content or whether its spirit (composite urgency view) is what the preview-pulse should be. Flag for planning. |

### Root-level folders (Section 14.1)

| v4.1 root folder | Current implementation | Category | Notes |
|---|---|---|---|
| Directory (universal) | `pages/Directory.jsx`, opened as overlay via OV.DIR + lazy-loaded inside MyLaneSurface; also reachable at `/Directory` | **PARTIAL** | The page exists and works inside the shell as an overlay. v4.1 wants it as a top-level cockpit folder, not an overlay tap on a header pill. The page content can be reused; what changes is the entry pattern (cockpit position rather than header chip). |
| Events (universal) | `pages/Events.jsx`, opened as overlay via OV.EVT + lazy-loaded inside MyLaneSurface; also `/Events` and `/Events/:eventId` | **PARTIAL** | Same as Directory — the page is reused; the entry shifts from header pill to cockpit folder. |
| Personal (universal for members) | none, conceptually | **NET-NEW (assembly)** | The constituent spaces of Personal exist (Home → HomeFeed; Personal Finance → finance workspace scoped to user; Meal Prep → mealprep workspace; Networks → Networks page; Playmaker → team-as-playmaker context). What's missing is the **Personal folder as a container** — a cockpit position whose contents are these spaces, distinct from business-scoped versions of the same spaces. Today, finance/meal-prep show up at the root spinner unscoped. |
| Businesses (universal for members) | `useActiveBusiness` hook + switcher mode in MyLaneSurface | **PARTIAL** | The list of owned businesses is computed; the switcher is the lateral-navigation primitive. What's missing is the **Businesses folder containing one folder per business**. Today, switcher mode is entered by tapping the space-name pill while operating in business context — the businesses-as-a-folder concept isn't represented. |
| Admin (contextual) | `pages/Admin.jsx`, accessed via avatar-dropdown link to `/Admin/*`; never inside MyLane | **MOVE/PARTIAL** | The admin panel is a complete standalone surface and should be resurfaced (DEC-173) as a contextual root folder rather than rebuilt. The page itself is reusable; what changes is its entry — MyLane root folder instead of Layout dropdown link. **Ambiguous:** v4.1 14.1 says contextual root; v4.1 14.2 also implies "admin lens inside every space" (Phase 4.5). The two surfaces are different. |
| Playmaker (contextual) | Currently surfaces as the `team` spinner item in MyLane (when user has a team profile) | **PARTIAL** | The team workspace is the Playmaker substrate today. Phase 4 needs to consider whether "Playmaker" becomes a contextual root folder distinct from "Team within Personal," and what gates its appearance. Flag for planning. |
| Networks (contextual) | `pages/Networks.jsx`, `pages/NetworkPage.jsx` (live as standalone routes + as overlay-stacked drill-ins via `overlayNetworkSlug`) | **PARTIAL** | Standalone pages exist. v4.1 wants Networks as a contextual root folder containing the user's joined networks. Page content is reusable; the wrapping shifts. |

### Header elements (Section 14.2)

| v4.1 element | Current implementation | Category | Notes |
|---|---|---|---|
| Left: LocalLane logo, tap returns to root | `MyLaneSurface.jsx` "Local Lane" wordmark, `handleLogoClick` | **DIRECT** | Already exists. May need behavior change to "back to root" when nested. |
| Right: Frequency Station glyph + small gap + avatar | Music icon (OV.FREQ toggle) + Directory pill + Events pill + avatar (OV.ACCT toggle) | **PARTIAL** | The Frequency glyph and avatar both exist. **Directory and Events pills move out of the header** under v4.1 — they become root cockpit positions, not header chips. That's a deletion in the header, not a new build. |
| Avatar opens settings/themes/account/feedback/sign out | `AccountOverlay` inside `MyLaneSurface.jsx` | **DIRECT** | Already does Settings, Theme, Cockpit, Sound, Newsletter, Philosophy, Support, Terms, Privacy, Logout. **Feedback affordance is missing** (DEC-137 routes feedback through Mylane companion; v4.1 14.2 implies a persistent feedback affordance — that's the resurface candidate per DEC-173). |

### URL addressability (Section 14.3)

| v4.1 URL | Current routing | Category | Notes |
|---|---|---|---|
| `/mylane` (root) | `/MyLane` (PascalCase from `pages.config.js`) | **PARTIAL** | Works, but case differs and there's no nesting under it. |
| `/mylane/personal`, `/mylane/personal/finance`, `/mylane/businesses` | none | **NET-NEW** | App.jsx has no nested routing under `/mylane`. The folder tree has no URL representation today. |
| `/b/{slug}`, `/b/{slug}/desk`, `/b/{slug}/profile` | none | **NET-NEW (deferred)** | **Per DEC-179, path-based direct doors are deferred to post-migration.** Phase 4 is not the place for this. v4.1 written before DEC-179. |
| URL as primary source of scope (Section 14.3 final paragraph) | `useActiveBusiness` reads localStorage first, no URL awareness | **PARTIAL** | The hook's contract should remain (DEC-146), but its source-of-truth flips. Until nested URLs exist (NET-NEW above), there's no URL to read — so this can't change in Phase 4 cleanly without the routing-restructure prerequisite. |

### Descent, return, lateral switching (Section 14.4)

| v4.1 | Current | Category | Notes |
|---|---|---|---|
| Descent: tap centered folder to enter | SpaceSpinner `onCenterTap` (switcher mode) + `commitSwitcher` in MyLaneSurface | **DIRECT** | The mechanic exists. Generalize beyond switcher. |
| Return: logo to root, back affordance up one level | Logo click goes to `/MyLane`; "‹ back" appears in header in switcher mode only | **PARTIAL** | Logo behavior is right. The "back one level" affordance only exists for switcher; needs to generalize for nested folders. |
| Lateral switching via space-name pill (DEC-168) | switcherMode in MyLaneSurface | **DIRECT** | Already implemented for business switching. Reuse for any folder. |

### Workspace at leaf (Section 14.5)

| v4.1 | Current | Category | Notes |
|---|---|---|---|
| Cockpit stays at top, workspace below, swipe between sibling spaces | MyLaneDrillView renders tabs below SpaceSpinner; spaces switch via spinner | **DIRECT** | Today's drill view already matches this shape. |

### Business: spaces & defaults (Section 6)

| v4.1 concept | Current | Category | Notes |
|---|---|---|---|
| `enabled_spaces: array<string>` field on Business | none | **NET-NEW** | Confirmed via grep — no occurrence anywhere in `src/`. Needs Base44 prompt + code (DEC-178 paired update). |
| Default `["desk", "finance", "profile", "settings"]` at business creation | none | **NET-NEW** | `pages/BusinessOnboarding.jsx` creates a Business record but does not set `enabled_spaces` (line 104). Needs both onboarding-flow update and a backfill strategy for existing businesses. |
| "+ Add Space" / remove space mechanic in Business Settings | none | **NET-NEW** | Confirmed via grep — no add-space UI exists. Adjacent UI (BusinessSettings.jsx is 1469 lines) is the natural home. |
| Backfill: inspect actual data per business to seed `enabled_spaces` | none | **NET-NEW** | Will need a one-off migration script (live alongside existing `src/scripts/migrations/phase-2-production-migration.js` pattern). |
| Dark-until-explored for spaces (Section 6.5) | `buildSpinnerItems` already gates spinner items by profile presence | **DIRECT** | The principle is enforced today. Once `enabled_spaces` lands, the gate moves from "has profile?" to "is in `enabled_spaces`?" but the principle is identical. |

### Business: Profile + directory presence (Sections 6.3, 13.1)

| v4.1 concept | Current | Category | Notes |
|---|---|---|---|
| Profile space (logo, tagline, services, gallery, contact) | `pages/BusinessProfile.jsx` (730 lines) | **DIRECT** | Already a complete public-profile page. v4.1 imagines it as a space inside the Business; the page itself is reusable. |
| `listed_in_directory` toggle in Business Settings | `components/dashboard/BusinessSettings.jsx` line 247 (Build 2) | **DIRECT** | **Already shipped.** v4.1 explicitly listed this as a Phase 4 item (it was Phase 3 Build 2, absorbed into Phase 4 per Section 17). Treat as done — no rebuild. |
| `directoryVisibility.js` filter helper | `src/utils/directoryVisibility.js` | **DIRECT** | Already shipped (DEC-146 single source per Build 2). |
| Hidden entities (Mycelia, LLC) — no separate `is_hidden` field needed | Already enforced by Phase 2 production migration | **DIRECT** | Mycelia exists as `listed_in_directory: false` per Phase 2. |

### Home → Desk collapse (Section 14, DEC-170)

| v4.1 / DEC-170 | Current | Category | Notes |
|---|---|---|---|
| Vocabulary: "Jobsite" → "Desk" in user-visible strings | Build C (DEC-170) did MyLane label + welcome map + HomeFeed priority spinner. Comment in BUILD-PROTOCOL says "FieldServiceAgent persona text intentionally preserved." | **PARTIAL** | The user-visible vocabulary in three high-traffic spots is already swapped. The full pass is the "~92 strings, ~40 files" mechanical job called out in v4.1 Section 17. |
| Structural: no separate Home inside a business — Desk is the home of business work | The business workspace tabs still include `home: DashboardHome`. Inside `BUSINESS_TABS` Home is the first tab. | **PARTIAL** | Vocabulary is half-done; structural collapse is undone. The Home tab's stats may need to migrate into Desk-as-home, or DashboardHome may need to become Desk's landing. **Ambiguous:** v4.1 doesn't specify whether DashboardHome's content moves into Desk verbatim or is rethought. Flag. |

### Resurface candidates (Section 20, DEC-173)

| v4.1 mention | Current | Category | Notes |
|---|---|---|---|
| Persistent feedback affordance | The OLD floating bug button was deleted (DEC-104, DEC-137); feedback flows through Mylane companion as a "Have feedback?" chip | **PARTIAL — RESURFACE** | The chip exists in some space positions but isn't persistent across all surfaces. The "resurface" task is to make a single feedback affordance that appears in every space, not invent one. |
| Admin panel as contextual root | `pages/Admin.jsx` + 17 `components/admin/*` files | **MOVE** | Page is whole and complete (DEC-173). Move means folding the entry into the cockpit's contextual root folder list, not rebuilding. |
| Other buried surfaces | TBD (Section 20 says "others yet to be identified") | **OPEN QUESTION** | Phase 4.5 sweep candidate. Out of scope for the structural Phase 4. |

---

## Section 4 — Net-New Work in Phase 4

Pulled from Section 3, the discrete pieces that aren't represented in current code at all and have to be built fresh. Listed in approximate dependency order, not delivery order.

1. **Folder-tree representation.** A declarative or builder-fn shape that lists root folders (Directory, Events, Personal, Businesses + contextual Admin / Playmaker / Networks) and, for each, what their contents are (sub-folders or workspaces). The cockpit consumes this. Today the equivalent is `buildSpinnerItems` returning a flat array; the new shape needs depth.
2. **Cockpit handling of folder vs leaf.** SpaceSpinner already renders an array of items; what's new is teaching MyLaneSurface to decide what to render below — preview pulse for folders, drill view for leaves. Today MyLaneSurface picks between HomeFeed / DiscoverPosition / MyLaneDrillView / DevLab based on which spinner item is active; the new logic generalizes to any folder vs leaf.
3. **Preview pulse component.** A living-tile renderer that takes a folder's contents and renders a pulse view — what's alive in this folder right now. New component. **Possibly reuses `HomeFeed`'s spirit and the `cards/` collection** — that's the resurface lens. Worth specifying before building from scratch.
4. **`Business.enabled_spaces` field.** Base44 prompt to add `array<string>` field. Default at create: `["desk", "finance", "profile", "settings"]`. Backfill script for existing businesses (inspect FS profiles, finance profiles, etc., and seed accordingly).
5. **Default-spaces-at-business-creation logic.** `BusinessOnboarding.jsx` writes `enabled_spaces` on Business.create (line ~104).
6. **"+ Add Space" / remove space UI in Business Settings.** New section in `BusinessSettings.jsx`. Reads/writes `enabled_spaces`. Surfaces the available-but-not-enabled spaces (Events, Network, Property Pulse, etc.).
7. **Personal folder as a container.** Today Finance/Meal Prep show up at the root spinner unscoped. The Personal folder needs to be the parent of those when the user is in personal context — distinct from business-scoped versions.
8. **Businesses folder as a container.** A folder whose contents are one sub-folder per owned business, each containing that business's `enabled_spaces`. The switcher pattern (DEC-168) becomes the descent mechanic into a specific business.
9. **Contextual root: Admin as a folder.** Wire `pages/Admin.jsx` into the cockpit's root folder list when `currentUser.role === 'admin'`. Don't rebuild; resurface.
10. **Contextual root: Networks as a folder.** Wire `pages/Networks.jsx` + `NetworkPage.jsx` into the cockpit's root folder list when the user has joined any network. Containing folder lists their joined networks.
11. **Contextual root: Playmaker as a folder.** Decide whether Playmaker is distinct from "Team inside Personal" and how the gate works. **Open question.**
12. **Persistent feedback affordance.** Resurface the existing "Have feedback?" chip pattern as a once-per-shell element, not a per-space element.
13. **Desk rename — full pass.** ~92 strings across ~40 files (per v4.1 Section 17). Mechanical find-replace, but careful: agent persona text intentionally preserved per DEC-170.

### Items the brief listed as candidates that turn out to be **not Phase 4**

- **Direct doors path-based routing (`/b/{slug}`).** Deferred to post-migration per DEC-179.
- **Auth-aware rendering on `/b/{slug}` URLs.** Same — deferred per DEC-179.
- **`listed_in_directory` toggle in Business Settings.** Already shipped in Build 2.
- **Home → Desk vocabulary swap** in three high-traffic spots — already done in Build C; what remains is the wider mechanical pass.

### Items the brief listed as candidates that **need clarification before building**

- **Home → Desk collapse (structural).** Vocabulary is partly done. Whether `DashboardHome`'s stats move into Desk verbatim, get rethought, or stay where they are is unspecified.

---

## Section 5 — Sequencing Recommendations

The brief proposed 4.1–4.7. The audit confirms the broad shape but suggests two adjustments and one removal:

**Confirmed sequence:**

- **4.1 — Entity work (Base44).** Add `Business.enabled_spaces` field. Paired Base44 prompt + code per DEC-178. Backfill existing businesses. **Note:** `listed_in_directory` is already done — no Base44 work needed for that.
- **4.2 — Folder structure refactor.** The structural move:
  - Define folder-tree shape.
  - Teach MyLaneSurface to render folder vs leaf based on tree depth, not based on a flat spinner-items array.
  - Wire Personal, Businesses as containers. Wire Directory and Events as root cockpit positions (rather than header pills — and remove the Directory/Events pills from the header). Wire Admin / Networks / Playmaker as contextual root folders.
  - Preserves the cockpit code (SpaceSpinner) untouched — only changes what MyLaneSurface feeds it.
- **4.3 — Home → Desk collapse (structural + remaining vocab).** Decide structural shape, do the ~92-string mechanical pass last (so structural moves don't invalidate it). **Note:** the brief's instinct to put the rename pass last is right — confirmed.
- **4.4 — Preview pulse.** New component for non-leaf cockpit positions. Probably reuses HomeFeed's spirit and the cards/ collection (resurface lens). Needs spec before build because v4.1 leaves the implementation underspecified.
- **4.5 — Spaces add/remove mechanic UI.** Section in BusinessSettings to toggle items in `enabled_spaces`. Cleanest after 4.1 (field exists) and 4.2 (folder rendering uses the field).
- **4.6 — Resurface pass.** Persistent feedback affordance + any other buried surfaces identified during 4.2–4.4 work. Per Section 20 / DEC-173.

**Recommended adjustments:**

- **Drop "4.6 Directory toggle UI" — already shipped in Build 2.** The brief's 4.6 is duplicative.
- **Reorder 4.4 (Preview pulse) before 4.5 (Spaces add/remove).** The preview pulse is what the user sees while navigating; the add-space mechanic is configuration. Building the navigation experience before the configuration UI lets the configuration UI be informed by what the navigation actually looks like.
- **Consider splitting 4.2 (Folder structure refactor) into 4.2a (root folders) and 4.2b (Businesses-as-folder + per-business sub-folders).** 4.2a is mostly mechanical (Directory, Events, Personal as cockpit positions). 4.2b touches the switcher pattern (DEC-168) and the business-context machinery — higher risk because it's where Bari's daily flow lives. Splitting lets 4.2a ship and stabilize before 4.2b lands.

**Phase 4.0 added 2026-04-27.** Before any Phase 4 UI work, the folder tree is established as a declarative configuration (in `src/config/folderTree.js`) read by a small predicate registry (in `src/config/folderPredicates.js`). MyLaneSurface's imperative `buildSpinnerItems()` is refactored to filter the config rather than build items inline. This is invisible to users (same folders, same conditions, same order) but converts the cockpit from "stone stairs" (every new folder = a code edit) to "living feet" (every new folder = a config entry). Phase 4.0 makes everything after it cheaper and makes the eventual Engagements addition (planned but not yet a DEC) a one-line config plug-in.

**Phase 4.2-tiles added 2026-04-28.** Smoke testing 4.2a surfaced friction with the spinner-as-cockpit pattern when folder-vs-leaf rendering was introduced. Tiles become the primary cockpit pattern; the spinner is preserved as a selectable alternate cockpit (DEC-152 pattern, behind a dev allowlist for v1 per DEC-147). Phase 4.2-tiles absorbs the original 4.2b (Businesses-as-folder), 4.3 (Home → Desk collapse), 4.4 (Preview pulse), and 4.5 (Spaces add/remove UI) into a unified sequence of six sub-builds. Each ships separately per DEC-183 path-walking — ship one, use it, then write the next prompt. 4.6 and 4.7 remain as planned. Full design captured in Section 8.13.

**Recommended order:**

```
4.0    ✅ Folder tree as configuration                                  [shipped 2026-04-27, 73a3d53/077c89a]
4.1    ✅ Entity (Business.enabled_spaces + backfill)                   [functional 2026-04-27, one record pending Base44 support]
4.2a   ✅ Universal root folders, header pills removed                  [shipped 2026-04-27, d47ac7d/304e538]

Phase 4.2-tiles — design pivot, absorbs 4.2b/4.3/4.4/4.5
4.2-tiles-1 ✅ Generic Tile primitive extracted from BusinessCard      [shipped 2026-04-28, a3ac463]
4.2-tiles-2 ✅ Breadcrumb component (cockpit-agnostic, hybrid modes)   [shipped 2026-04-28, caae822]
4.2-tiles-3 ✅ Tile cockpit rendering at root                          [shipped 2026-04-28, 77571b7]
4.2-tiles-4 ✅ Per-business folder rendering (first enabled_spaces consumer)  [shipped 2026-04-28, 84a9889]
4.2-tiles-5   Settings space surface (lifted from BusinessSettings.jsx)
4.2-tiles-6   Cockpit picker allowlist gate, tile becomes default      [DEC-147 pattern]

4.6    Resurface pass (Admin folder, feedback affordance, others)      [Section 20 sweep]
4.7    Desk rename mechanical pass                                      [last, ~92 strings, ~40 files]
```

---

## Section 6 — Dependencies and Risks

**Dependencies on work outside Phase 4:**

- **Phase 3.5 Direct Doors are no longer a Phase 4 dependency.** Per DEC-179, path-based direct doors deferred to post-migration. v4.1 was written before that decision; Phase 4 should not assume `/b/{slug}` routing. The "URL as primary source of scope" line in v4.1 §14.3 cannot be fulfilled in Phase 4 without nested `/mylane/...` routing — which can be built without `/b/{slug}` if desired, but isn't a hard requirement.
- **Phase 5 (Pre-Migration Cleanup, DEC-180)** is downstream and consumes nothing from Phase 4.
- **Phase 6 (Region foundation)** needs `region_id` on records but is downstream. Not a Phase 4 dependency.

**Internal Phase 4 dependencies:**

- 4.5 (Spaces add/remove UI) **requires** 4.1 (`enabled_spaces` field). Hard.
- 4.2b (Businesses-as-folder) **builds on** 4.2a (root folders) but isn't strictly blocked.
- 4.4 (Preview pulse) **needs spec** before build — v4.1 underspecifies and HomeFeed's spirit overlaps. Recommend a Mycelia + Doron design pass before 4.4 starts.
- 4.7 (Desk rename pass) **must be last**: structural moves rename strings before the mechanical pass would have caught them.

**Risks:**

- **Bari is the only paying user** ($500 retainer 2026-04-24). Anything that touches his business-context flow is real-cost risk. The DEC-168 switcher and `useActiveBusiness` are the load-bearing pieces. 4.2b is the highest-risk Phase 4 step. Strong recommendation: ship 4.2a first, verify Bari's flow uninterrupted, then 4.2b.
- **The OV constant + overlay containment (DEC-148)** is the current way pages-rendered-as-overlays work. Phase 4 changes the entry pattern (cockpit folder, not header chip) but the underlying overlay primitive remains. Care needed not to rebuild overlay machinery while restructuring entry.
- **Hardcoded z-indices (z-50 / 55 / 60) for stacked overlays** are flagged tech debt in ACTIVE-CONTEXT. Phase 4 may add stacking patterns (folder drill-in might be a stack). If a third stacked-overlay scenario appears during Phase 4, refactor z-index to stack-based assignment in the same pass (per the audit's own guidance in DEC-148 footnote).
- **`MyLaneSurface.jsx` is 1385 lines.** Phase 4 work concentrates here. There's a real risk of accidentally breaking adjacent state (overlays, switcher mode, Escape stack handling) while moving folder-render logic in. ACTIVE-CONTEXT names "MyLaneSurface hook reordering" as a Phase 4 priority — that suggests prior known instability that should be addressed before or during Phase 4.2.
- **The "MyLaneDrillView latent bug fix"** named in ACTIVE-CONTEXT is unspecified here but listed as a Phase 4 priority. If it shipped first, it might inform 4.2b sequencing. Worth pulling forward to scope.
- **Eslint exhaustive-deps enforcement** (also a Phase 4 priority per ACTIVE-CONTEXT) is orthogonal to the folder refactor but typically generates churn in MyLaneSurface and the hooks adjacent to it. Recommend running it before 4.2 starts so the diffs in 4.2 are pure folder work.

---

## Section 7 — Open Questions

These are the spots where the audit found enough ambiguity to warrant a Mycelia + Doron planning conversation before execution prompts get written.

1. **Preview pulse — what is it concretely?** v4.1 says "living-tile language describing what's alive in the folder." HomeFeed already does something close to that for the Home position (Attention / This week / Spaces tabs feeding a PrioritySpinner). Is the preview pulse:
   - HomeFeed's pattern, generalized per folder?
   - A new compact tile-based view (one tile per child)?
   - Something else entirely?
   The answer shapes 4.4's scope and whether it's a build or a resurface.

2. **Home → Desk structural collapse — what content moves?** Vocabulary is partly done. Today `DashboardHome` (the Business workspace's "Home" tab) shows stats, upcoming events, quick actions. v4.1 / DEC-170 says no separate Home inside a business. Does that mean:
   - DashboardHome's content becomes Desk's landing tab (rename + structure-preserve)?
   - DashboardHome retires and Desk's existing landing absorbs the relevant pieces?
   - The stats and quick actions move elsewhere and Desk's landing is reconceived?

3. **Playmaker as contextual root — what's the gate?** v4.1 14.1 says "appears for users with any Playmaker role." Today the team workspace surfaces in MyLane for users with a TeamMember row. The "Playmaker role" concept implies parent-coach-player distinctions per DEC-123. Is Playmaker a separate root folder distinct from "Team inside Personal," and what role-check determines visibility?

4. **Admin contextual root vs. admin lens inside spaces.** v4.1 14.1 places Admin as a contextual root folder. Phase 4.5 in Section 17 talks about "Admin lens inside every space (authority-aware affordances)." Different surfaces. Phase 4 builds the root folder; Phase 4.5 builds the lens. Confirm split.

5. **Personal folder vs. unscoped spinner items.** Today Finance and Meal Prep show up at the root spinner without scope (they're personal-flavored by default since user's the only operator). v4.1 says these live inside Personal. Is the migration:
   - Move them under Personal and surface Personal at root, OR
   - Keep them at root but mark them Personal-flavored, OR
   - Both (Personal contains them, but they also appear at root for fast access)?

6. **URL nesting in Phase 4 vs. waiting for migration.** v4.1 14.3 wants `/mylane/personal/finance` etc. Building nested routing is independent of `/b/{slug}` direct doors (which are deferred). Should Phase 4 add the nested `/mylane/...` routes, or wait until the post-migration stack? Building them on Base44 is throwaway if they get rewritten on Supabase/Vercel.

7. **`enabled_spaces` backfill specificity.** Inspecting actual data per business is straightforward for FS / Finance / PM (existence of profile = space was used). Less obvious for Events, Network, Property Pulse if they're not already represented as separate profile entities. Need to map each space type to a backfill signal before writing the migration script.

---

## Section 8 — Planning Decisions (Mycelia + Doron, 2026-04-26 evening)

After Hyphae's audit landed (commit `1ed1a6c`), Mycelia and Doron walked the seven open questions in Section 7 and made the following decisions. These close the planning loop and feed directly into Phase 4 execution prompts.

### 8.1 — Preview pulse: silence by default, signal when warranted

The area below the cockpit at non-leaf folder levels is **conditional, not constant**. When there is nothing alive that needs attention in the centered folder, the area is calm — possibly a single line of label text indicating folder contents, possibly empty space. The pulse appears only when there is something to pulse about: an unsigned estimate, a parent who hasn't confirmed, a transaction that needs categorizing, an event starting soon.

This resolves Open Question #1. It rejects the temptation to fill the space with synthetic activity feeds or stat panels. Filler-content is anti-circulation; it teaches the user to scan rather than to act.

The implementation question becomes: what's the trigger logic for "something needs attention in this folder"? Each folder type has its own answer, derived from the entities it contains. That logic gets specified when the 4.4 (Preview pulse) execution prompt is written, not here.

### 8.2 — Folder placement: shortcuts (user-determined), not dynamic rules (system-determined)

The temptation surfaced during planning to make folder placement dynamic — Playmaker at root for users with one role, nested under Personal for users with many things going on. After discussion, this was rejected in favor of a **user-determined shortcut layer**.

The principle: the user decides what matters most to them. The system does not algorithmically guess.

Phase 4 ships v4.1's universal + contextual root folders as currently described — Directory, Events, Discover, Personal, Businesses (universal); Admin, Playmaker, Networks (contextual). Placement is static.

Future work (post-Phase-4): a shortcut/pin primitive that lets users pull any item from anywhere in the folder tree to either their personal Home, or to root-spinner-level fast-access. This is a meaningful addition to v4.1 and earns its own DEC and design pass when it gets built.

The folder-tree representation in Phase 4 should be designed so that this future shortcut layer doesn't require a refactor. Specifically: the tree should be a function (not a static config) so placement rules can evolve.

### 8.3 — Multiple businesses live under Businesses; holding hierarchy lives in data, not display

Confirmed v4.1's existing structure. A user with three businesses (Doron's case: TCA, Recess, Consulting, plus hidden Mycelia and exempt LocalLane) does not see them at root. They live under the Businesses folder. The user enters Businesses and selects which business to operate as. The DEC-168 switcher pattern handles this exactly right.

The holding-company hierarchy (Mycelia, LLC parents TCA / Recess / Consulting / LocalLane) lives in **data, not display**. Finance roll-up shows the consolidated parent view for tax filing. The dashboard does not represent parent-child as nested folders. All the businesses appear as siblings in the Businesses folder; Mycelia is hidden via `listed_in_directory: false`; LocalLane is exempt from its own fee.

This separation is clean: user mental model is "I have N businesses." Accounting model is "my parent owns these and rolls them up." Both true; live in different places.

### 8.4 — Apple Finder spatial model is the reference (with explicit deltas)

Decades of HCI research have established the folder mental model: tap to descend, back to ascend, breadcrumb shows depth, lateral switching at the same level, folders distinguished from leaves visually. LocalLane should use what users already know rather than reinvent it.

**What to borrow from Finder:**

- Tap-to-descend, back-to-ascend
- Breadcrumb showing path depth
- Lateral switching at the same level (matches DEC-168 switcher)
- Folders are containers (places with stuff), not screens (pages with content)
- Visual distinction in the cockpit between folders (descend-able) and leaves (workspaces — no further descent)

**What NOT to borrow from Finder:**

- Tree view in a sidebar showing full hierarchy. v4.1 explicitly rejects panoptic views ("the map is alive, relative, and never panoptic"). The user only ever sees the folder they're in plus its siblings, never the full tree.
- Static thumbnails. We want preview *pulse* — living signal, not frozen snapshot.
- The desktop metaphor (files on a desk dragged into folders). LocalLane is a navigable garden, not a workspace metaphor.

The folder-leaf visual distinction in cockpit chrome is a small design move worth surfacing during 4.2 build planning. Currently SpaceSpinner doesn't distinguish — every spinner item looks the same. v4.1 names lateral switching (DEC-168) but doesn't explicitly name folder/leaf chrome.

### 8.5 — URL nesting deferred; behavior works as if nested

Per DEC-179 (path-based direct doors deferred to post-migration), Phase 4 does NOT build nested `/mylane/personal/finance` style URLs. Building them on Base44's router would mean writing code that gets rewritten when we migrate to Supabase + Vercel.

However, the *behavior* of nested folders is preserved: descent, return, breadcrumb navigation, lateral switching. The folder tree exists in component state; the URL is a side-effect that Phase 4 does not yet wire up. Post-migration, when the routing layer is something we keep, URL nesting lands naturally without a structural refactor.

This resolves Open Question #6.

### 8.6 — Desk inherits Home's content as-is for Phase 4

The Home → Desk vocabulary swap renames the tab. The content (stats, upcoming events, quick actions per `DashboardHome.jsx`) stays the same in Phase 4. No structural redesign of what Desk shows.

A future design possibility was surfaced and noted: a **pin/shortcut pattern** where items created in a business default to pinning on the Desk, with the user able to unpin or move later. This would align Desk with the preview-pulse principle — Desk shows what's actively being touched, not all the data in the business. This is a Phase 5 or 6 candidate, not a Phase 4 build.

This resolves Open Question #2.

### 8.7 — Discover stays as a fifth root folder

Universal root folders are: **Directory, Events, Discover, Personal, Businesses**. Discover is included alongside the other four (v4.1's Section 14.1 listed only four; this addition reflects what's currently shipping and what users have already learned).

Discover's deliberate dimness in the cockpit is preserved — it's an entry point for exploration without commitment. Users tap into Discover and explore from there.

### 8.8 — Admin: contextual root folder in Phase 4 (Thing 1 only)

Phase 4 ships **Admin as a contextual root folder** for users with admin role. Tap the Admin folder, enter the existing `Admin.jsx` page (resurfaced per DEC-173). This is a wiring task, not a build.

The "admin lens inside every space" concept (per v4.1 Section 17, Phase 4.5) — additional admin-aware affordances on every other space's screen — is **deferred to Phase 4.5**. That's a different surface and shipping it as part of Phase 4 would expand scope unnecessarily.

This resolves Open Question #4.

### 8.9 — Playmaker: contextual root folder, default placement

Phase 4 ships **Playmaker as a contextual root folder** for users with any TeamMember role. This matches what's currently shipping. Placement is static (at root, by default) for Phase 4. Future shortcut/pin work (per Section 8.2) may let users move it.

This resolves Open Question #3 with the simpler answer: keep current shipping behavior, defer fancier placement logic.

### 8.10 — `enabled_spaces` backfill specificity

The migration script for adding `enabled_spaces` to existing businesses uses these rules:

- **Doron's businesses (Mycelia LLC, LocalLane, TCA, Recess, Consulting):** Doron self-activates the spaces he uses on each. The migration script seeds with `["profile"]` and Doron toggles others on via the Add Space UI when 4.5 ships. (Or, if 4.5 isn't shipped yet by the time he needs full activation, the field can be edited directly in Base44.)
- **Bari's business (Red Umbrella):** Inspect actual usage. Currently using at minimum: Profile, Desk (Field Service), Settings. Finance if there's a finance profile. Seed accordingly per actual data inspection.
- **All other businesses in the system:** Seed with `["profile"]` only. They are directory-listed but not actively using other spaces. The 4.5 Add Space UI lets them activate more when they want.

This resolves Open Question #7. The backfill is concrete and the migration script can be specced from this directly.

A useful side-effect: the seed values surface real data about the platform — how many businesses are directory-only vs actively using multiple spaces. That's a circulation signal worth tracking.

### 8.11 — Living feet at the architectural level

DEC-146 ("living feet, not stone stairs") was originally captured as a philosophical principle — the foundation moves with the organisms living on it. After tonight's Engagements concept conversation surfaced the question of how new contextual roots get added to the cockpit, the principle generalizes to the engineering layer: **the codebase should accept new folders, predicates, and (eventually) entity-relationship types by configuration rather than by net-new code**.

Phase 4.0 (added to the sequence) enforces this for the folder tree specifically. The folder tree becomes a declarative config; visibility predicates become a registry; MyLaneSurface becomes a thin renderer over both. Future contextual roots — Admin (already planned for Phase 4.6), Engagements (concept under design), future ones we don't yet know — plug in by adding a config entry plus an optional predicate. No core rewrite.

Adjacent living-feet patterns already shipped in the codebase: `VARIANT_MAP` for cockpit variants (DEC-152), `WORKSPACE_TYPES` for tab structures, `myLaneRegistry` for cards. Phase 4.0 extends this established pattern to the folder tree itself.

The deeper question — whether the relationship-shape entities (TeamMember, Client, Subcontractor, future Engagement) should unify into a single `Relationship` primitive — is parked. Engagements MVP can build on existing entity shapes for now. The unification question is a Phase 5 or Phase 6 architectural call when it becomes load-bearing.

### 8.12 — Phase 4.2a shipped (2026-04-27 afternoon)

Universal root folders moved into the cockpit. Five new entries land on the root spinner: **Directory**, **Events**, **Personal**, **Businesses**, plus the existing **Discover**. Directory and Events lose their header-pill chrome; descent into them via center-tap on the cockpit opens the existing OV.DIR / OV.EVT overlay machinery (resurface, not rebuild — DEC-173). Personal becomes the static folder containing today's personal-flavored leaves (home, meal-prep/Kitchen, field-service/Desk, finance, team, property-pulse). Businesses replaces the singular `business` leaf as the descent point for the DEC-168 switcher.

**Folder tree representation chosen: nested `children` arrays** in `src/config/folderTree.js`. Picked over `parent_id` because the tree shape is immediately visible from the file (one glance shows the hierarchy) and adding a child entry can't typo a parent reference. Helpers exported alongside the data — `rootItems`, `folderItems`, `locateLeaf`, `allLeaves` — so MyLaneSurface stays a thin renderer. Phase 4.0's predicate registry (`folderPredicates.js`) remained unchanged — every new folder uses an existing predicate (`always`, `has_owned_business`, `is_admin`).

**The Phase 4.0 abstraction held.** The folder-tree config absorbed the new universal roots without a rewrite — only additive entries plus the small `children` extension. MyLaneSurface gained ~42 lines (1373 → 1415) — descent state, center-tap descent gesture, cross-folder `navigateToId` helper, folder-vs-leaf branch in `renderContent`. SpaceSpinner stayed untouched per DEC-173. The `kind: 'folder' | 'leaf'` field introduced in 4.0 became load-bearing in 4.2a as predicted.

**Switcher mode (DEC-168) preserved without structural changes.** The space-name pill still triggers `enterSwitcher` exactly as before. Phase 4.2a adds a second entry path — center-tapping the Businesses folder on the root cockpit also enters switcher mode — but the existing pill-tap path is untouched. Switcher commit now lands on Personal/Home (descended) rather than spinnerIndex 0 of a flat list, preserving the post-commit "user is at Home of new business context" UX.

**Rendering at folder positions: silence-by-default per Section 8.1.** When the centered cockpit item is a folder (kind === 'folder'), the area below renders an empty 24px-min spacer. No preview pulse machinery built. Directory and Events open their existing full-page overlays on descent — those overlays are the folder's "content," in effect.

**Logo behavior generalized.** The "Local Lane" wordmark click now clears descended state, exits switcher, AND snaps spinnerIndex to 0 — matches v4.1 §14.2's "logo returns to root at any depth." Escape key also unwinds the descent stack as its final step.

**One judgment call flagged for review:** the welcome card's `WELCOME_LABELS` map still includes `'business': 'business'` from before 4.2a. There is no longer a leaf with id `business` — the welcome flow's "Go to <space>" button for that key is now a no-op (graceful degradation; no crash). The card path is rare and runs through invite-state localStorage; chose to flag rather than redesign the welcome flow inside this pass. Revisit in 4.2b or 4.6.

**Cold-open landing position is now Directory** (root index 0). Today's cold-open landed on Home (Personal/home). Doron will path-walk this — if root[0] = Directory feels jarring, adjust by reordering folderTree.js or initializing spinnerIndex to point at Personal. No hard data on which is right; preserved spec order from v4.1 §14.1 for now.

**Risk to paying user remains effectively zero.** Bari is not yet using LocalLane for daily work. Doron is the smoke-test for both switcher preservation and the descent gestures.

### 8.13 — Tile cockpit design pivot (2026-04-27 evening through 2026-04-28 morning)

After Phase 4.2a shipped, smoke testing surfaced three real frictions with the spinner-as-cockpit pattern: (1) the pill-switcher's contextual meaning shifts confusingly with depth — clicking the "finances" pill enters business switcher mode anywhere it appears; (2) folder-centered positions render to silence below the cockpit (per Section 8.1) which is fine philosophically but felt empty in real use; (3) the "what's inside this folder" question doesn't have a natural spinner answer — the spinner gives you one centered thing, but a folder is many things.

The pivot: tiles become the primary cockpit pattern. The current spinner area gets repurposed visually for breadcrumb path navigation. The spinner cockpit is preserved as a selectable alternate (not deleted) — the existing DEC-152 cockpit library pattern handles this naturally.

A pre-build investigation (2026-04-27 evening) inspected codebase reusability for the new pattern and surfaced findings that shape the build. The locked decisions:

**Tile primitive:** A generic `<Tile>` component takes `{ id, label, sublabel?, icon?, accentColor?, kind, onClick }` and renders the visual shell — `rounded-lg`, `bg-gradient-to-br from-secondary to-secondary/90`, `border-l-4` with accent, hover lift (`hover:-translate-y-0.5`), 300ms ease-out transition, 0.08 opacity primary glow on hover. `BusinessCard` becomes a thin wrapper that adds Business-specific decoration (network chips, tier badges, vitality state) and resolves Business-specific data (category labels via `useCategories()`, network chips via `useConfig()`). Folder tiles, leaf tiles, and any future tile kind use the generic Tile directly.

**Settings as a universal space:** Settings is never listed in `enabled_spaces`. Profile and Settings render unconditionally for every business alongside whatever opt-in spaces the business has activated. `enabled_spaces` means "additional opt-in spaces" only. No follow-up backfill is needed — Phase 4.1's seeded values stay as they are. Reasoning: a business with no Settings is unreachable (no way to edit name, toggle directory listing, or delete). It's part of what *being a business in LocalLane* means, not a feature flag. The 4.5 Add/Remove Space UI manages only the opt-in subset, not the universal pair.

**Category-driven accents preserved:** No per-business `brand_color` field added to Business in v1. The existing `resolveCategoryAccent()` pattern in `BusinessCard.jsx` (keyed on `main_category` slug, with archetype fallback) stays. Visual story remains "category at a glance," not "personalized brand color." Revisit if path-walking surfaces a real owner-personalization need.

**Folder tile accents:** Folder tiles (Directory, Events, Personal, Businesses, Discover, Admin, Engagements) need their own accent assignments alongside the existing category palette. Defer to the first build prompt — Hyphae proposes a five-to-eight-color palette from the existing `border-l-{color}-700` family that complements the theme and doesn't collide with category accents.

**Cockpit picker pattern (DEC-147 + DEC-152):** Tile cockpit is the v1 default for everyone. The spinner and compass cockpits are gated behind an email allowlist constant — `COCKPIT_PICKER_ALLOWLIST` — same pattern as `MYLANE_AGENT_ALLOWLIST` in MyLaneSurface.jsx. Allowlisted users see the AccountOverlay's "Cockpit" toggle with all three options. Non-allowlisted users see no cockpit toggle (no placeholder, no "coming soon" — DEC-147 says nothing). localStorage drives the choice (`ll_cockpit` key already exists from DEC-152). No User entity changes for v1. Upgrade path when tile cockpit is proven: drop the allowlist gate, optionally promote `ll_cockpit` to a server-stored field on User if cross-device persistence becomes worth it.

**Breadcrumb component (cockpit-agnostic, hybrid modes):** The breadcrumb is its own component, separate from any cockpit. It reads navigation state and renders breadcrumb segments with current-segment highlight and click-to-jump-back. Two presentation modes via prop:

- `mode="primary"` for tile cockpit — breadcrumb takes the prominent position where the spinner area was, large segments, central to the experience
- `mode="adjacent"` for spinner cockpit and future cockpits — small "you are here" affordance, less prominent, supporting the cockpit's primary navigation

Same data, same logic, different visual weight per cockpit. Future cockpits inherit the breadcrumb for free. The spinner cockpit gains nested-folder support as a side effect, extending its useful life for users who prefer it.

This is genuinely net-new UI; the `MyLaneBreadcrumb.jsx` referenced in older session logs as if it existed was a stale reference.

**Space-type catalog:** When Phase 4.2-tiles-4 ships per-business folder rendering, the renderer must read from a space-type catalog config (likely `src/config/spaceTypes.js`), never from inline conditionals. Each space type maps to its tile metadata — label, icon, accent, route. The renderer iterates `enabled_spaces`, unions with the universal `['profile', 'settings']`, and looks up each type-string in the catalog to produce tiles. Adding a new space type later means one config entry. This is the same living-feet pattern as `folderTree.js`, `folderPredicates.js`, `VARIANT_MAP`, and `WORKSPACE_TYPES` — the codebase already has the discipline; tiles-4 applies it to space types.

**Cold-open synthetic root:** The user lands on a synthetic "Home" root that shows all universal + contextual root folder tiles. The breadcrumb shows "Home" highlighted. From there: tap any tile to descend. Click "Home" in the breadcrumb (or the LocalLane logo top-left) to return to root from any depth.

**`enabled_spaces` is unconsumed code:** Phase 4.1 added the field on Business and seeded most records, but no `src/` code reads or writes it yet. The first consumer is Phase 4.2-tiles-4. Worth verifying directly in Base44 before tiles-4 starts whether the seeding is current.

**Breadcrumb plus tile-tap descent supersedes the spinner's center-tap-descent gesture from 4.2a.** The descent state machine added in 4.2a (per Section 8.12) is rewireable to the tile model — tile-tap is the descent gesture, breadcrumb segments are the ascent gesture, the underlying state shape is preserved. Not net-new state, just rewired.

### 8.14 — Phase 4.2-tiles-1 shipped (2026-04-28)

Generic `<Tile>` primitive landed at `src/components/ui/Tile.jsx` (121 lines). The visual signature of the Directory tile — rounded card, `bg-gradient-to-br from-secondary to-secondary/90`, `border-l-4` accent, hover lift, 300ms ease-out transition — now lives in one place. `BusinessCard.jsx` was refactored to wrap `<Tile>` (201 → 184 lines) by passing business-specific resolved values (label, sublabel, accentClass) and rendering business-specific decoration (network chips, tier badges, location, product tags) as Tile children. The smaller-than-anticipated drop on BusinessCard reflects that the file's bulk was always business-specific resolution and decoration — the visual shell itself was ~17 lines, all of which moved out cleanly.

**Public API preserved.** `BusinessCard` default export and `resolveCategoryAccent` named export both kept unchanged; the three consumers (`Directory.jsx`, `NetworkPage.jsx`, `BusinessProfile.jsx`) needed no edits. Hover effects gate on interactivity (only apply when `onClick` or `href` is provided), so static-tile usage in future builds (folder tiles, leaf tiles) gets the right affordance for free. The `data-tile-kind` attribute carries the `kind` prop through to the DOM as a hook for any future per-kind styling. No behavior change observed at the Directory or NetworkPage surfaces — the refactor is structural only.

This unlocks tiles-2 (breadcrumb component) and tiles-3 (tile cockpit at root); both will use `<Tile>` directly rather than building their own visual shell.

### 8.15 — Phase 4.2-tiles-2 shipped (2026-04-28)

Cockpit-agnostic `<BreadcrumbPath>` primitive landed at `src/components/ui/BreadcrumbPath.jsx` (121 lines). Takes a `segments` array (ordered root → current) plus a `mode` prop (`"primary"` or `"adjacent"`) and renders a horizontal pill trail. The last segment is the current location, rendered non-interactively with `aria-current="page"`; earlier segments fire their own `onClick` callbacks when tapped. Pure rendering — no state, no navigation logic, the consumer owns both. Both modes share the same warm-on-hover pill family (`bg-primary/10 text-primary border-primary/30` for the active segment, matching the network chips in `BusinessCard.jsx` and the `<Tile>` hover state); they differ only in size — primary uses `text-sm px-4 py-1.5` with `py-3` container padding for prominence, adjacent uses `text-xs px-2 py-0.5` with tighter `py-1` padding to sit lighter alongside another nav primitive.

**Resurface, not rebuild (DEC-173).** Composes the shadcn primitives in `src/components/ui/breadcrumb.jsx` (`Breadcrumb`, `BreadcrumbList`, `BreadcrumbItem`, `BreadcrumbPage`, `BreadcrumbSeparator`) for the a11y wrapping, then overrides shadcn's default classes to match LocalLane's pill language. **Naming flag:** the file is `BreadcrumbPath.jsx` rather than `Breadcrumb.jsx` because the lowercase shadcn `breadcrumb.jsx` already exists at the same path; APFS is case-insensitive, so the two would collide. The "Path" suffix also signals the segments-as-path API.

**Not yet wired in.** Component exists as a tested standalone. tiles-3 will be the first real consumer — mounted inside the tile cockpit in `mode="primary"`. Future cockpits (spinner, future Voice) can mount it `mode="adjacent"` for free. No existing files were modified beyond this migration plan.

### 8.16 — Phase 4.2-tiles-3 shipped (2026-04-28)

Tile cockpit landed and became the v1 default for everyone. New `TilesCockpit.jsx` at `src/components/mylane/TilesCockpit.jsx` (158 lines) composes the tiles-1 `<Tile>` primitive and the tiles-2 `<BreadcrumbPath>` primitive into a coherent cockpit experience. Pure rendering — `TilesCockpit` receives `state`, `descendedFolderId`, `tileLeafSelected`, and the leaf-currently-selected as props; emits `onDescendFolder`/`onSelectLeaf`/`onAscend` callbacks. The composition is the point: this file is small because the foundations were laid in tiles-1 and tiles-2.

**Force-migration + allowlist (DEC-147).** New constant `COCKPIT_PICKER_ALLOWLIST = ['doron.bsg@gmail.com']` in `MyLaneSurface.jsx` (paralleling the existing `MYLANE_AGENT_ALLOWLIST`). The cockpit toggle in `AccountOverlay` is wrapped in `cockpitPickerEnabled && (…)` — non-allowlisted users see no toggle at all. The toggle cycle is `tiles → spinner → compass → tiles` for allowlisted devs. Force-migration runs in a `useEffect` keyed on `currentUser?.email` inside `MyLaneSurface`: if the user is non-allowlisted and `ll_cockpit` is `'spinner'` or `'compass'`, overwrites localStorage to `'tiles'` and updates the `data-cockpit` DOM attribute. Allowlisted users keep their preference.

**Default-cockpit bootstrap.** `src/main.jsx` now sets `ll_cockpit='tiles'` and `data-cockpit='tiles'` on first paint when no value is stored — flipping the v1 default from implicit spinner to explicit tiles for new users.

**Cockpit detection at the surface level.** `MyLaneSurface.jsx` gained a local `useCockpit`-style hook (MutationObserver on `data-cockpit`, mirrors the SpaceSpinner internal pattern from DEC-152) so the surface can branch on `currentCockpit === 'tiles'` without depending on SpaceSpinner internals. `SpaceSpinner.jsx` was not modified.

**Tile-leaf-selected state.** New `tileLeafSelected` boolean state in `MyLaneSurface` distinguishes "browsing folder children as tiles" from "a leaf has been tapped, render its workspace below." The `descendedFolderId` state from 4.2a is preserved unchanged; tile-leaf-selected is additive. Logo-click and Escape-key handlers were extended to clear the new state alongside descent.

**Folder-tile accent palette.** Hardcoded inside `TilesCockpit.jsx` as `FOLDER_ACCENTS = { directory: 'border-l-gray-700', events: 'border-l-purple-700', personal: 'border-l-teal-700', businesses: 'border-l-amber-700', discover: 'border-l-blue-700', 'dev-lab': 'border-l-muted-foreground' }`. Drawn from the same `border-l-{color}-700` family as the Directory category accents in `BusinessCard.jsx` (DEC-060). Net-new colors (gray, purple, blue) avoid collision with existing category accents (amber/teal/violet/sky/rose). Flag: if a fourth/fifth folder lands and the hardcode becomes a stone, move accents into `folderTree.js` as a per-entry field.

**No icons.** Tiles render with label + accent border only; the `icon` prop on `<Tile>` stays unused for now (deferred per scope).

**No pinned favorites.** Zone 2 collapses entirely at root (silence by default per Section 8.1). Pinning is deferred until use surfaces real need.

**Untouched per scope:** `SpaceSpinner.jsx`, the compass cockpit code (internal to SpaceSpinner's VARIANT_MAP), `Tile.jsx`, `BreadcrumbPath.jsx`, `folderTree.js`, `folderPredicates.js`. Hook reordering and eslint exhaustive-deps are also untouched (separate Phase 4 priorities, owed but out of scope).

This is the first build that puts tiles-1 and tiles-2 in front of users. Path-walking begins.

### 8.17 — Phase 4.2-tiles-4 shipped (2026-04-28)

Tile cockpit gained uniform navigation and its first per-business descent surface. New `src/config/spaceTypes.js` (123 lines) holds the space-type catalog — eight entries for now (`profile`, `settings`, `desk`, `finance`, `team`, `kitchen`, `property`, `events`), with `profile` and `settings` flagged `universal: true` so they render for every business regardless of `enabled_spaces` (per Section 8.13's "Settings as universal space" decision). The `resolveBusinessSpaces(business)` helper unions the universal pair with the business's `enabled_spaces` array, deduplicates, and returns ordered tile metadata. This is the first build that consumes `enabled_spaces` from the Business entity (added in Phase 4.1, dormant until now). Bari's `["profile", "desk", "settings"]` resolves to four tiles total: Profile, Settings (both universal), Desk (from his array). Other businesses with `["profile"]` resolve to two tiles: Profile + Settings.

**Three patterns retired for tile cockpit users (preserved for spinner cockpit).** DEC-148 overlay shortcut for Directory/Events: tile cockpit now descends into them (sets `descendedFolderId`), and the page content renders inline using the same `.overlay-page-content` wrapper class so the existing page-header suppression carries forward (DEC-173 resurface, no page rebuild). DEC-168 lateral switcher for Businesses: tile cockpit descends to a tile grid of owned businesses, each rendered as a `<Tile>` with category accent via `resolveCategoryAccent` (DEC-060 visual continuity). Tap a business → `setActiveBusiness` is called (mirrors the DEC-168 commit logic exactly) AND `descendedBusinessId` is set, descending into that business's spaces tile grid. Dev Lab removed from `folderTree.js` and the orphaned `space.id === 'dev-lab'` branch cleaned out of `renderContent` — the `DevLab` component itself stays as the admin physics tuner toggle, unrelated to the folder tree. Every tile cockpit descent is now uniform: tap → set state → render breadcrumb + content. Spinner cockpit's `handleCenterTap` is untouched; allowlisted users who toggle to spinner still get the overlays and switcher.

**Per-business space tile-tap renders a v1 placeholder for every space type** (`BusinessSpacePlaceholder`, inline in MyLaneSurface). The placeholder reads label + sublabel from the SPACE_TYPES catalog so the tile and the placeholder share one source of label truth. Real workspace surfaces — `BusinessProfilePage` for Profile, the lifted `BusinessSettings` for Settings (tiles-5), business-scoped `MyLaneDrillView` paths for Desk/Finance/Team/etc. — are deferred to tiles-5+. v1 ships uniform navigation; later builds wire the workspaces with careful per-business profile scoping. Each tile renders honestly: the user can see the space exists, what it's for, and that the workspace surface is on the way.

**Two new descent state slots in `MyLaneSurface`:** `descendedBusinessId` (null or business id) and `descendedSpaceId` (null or SPACE_TYPES id). Both are tile-cockpit-only — spinner cockpit never reads them. Logo-click clears all descent state; Escape unwinds level-by-level (space → business → leaf → folder → root). The `tileLeafSelected` slot from tiles-3 remains for Personal-folder leaf workspace rendering.

**Cleanup follow-up (community-node `2c01950`, 2026-04-28):** Removed `field-service` (Desk) from Personal's children — Desk is a business-only space per the city/buildings architectural metaphor; it lives inside each business via `enabled_spaces`, never under Personal. Dropped the orphaned `has_field_service_profile` predicate, the `Briefcase` icon import in folderTree.js, and the Estimates card-builder block in HomeFeed that was navigating to the now-retired leaf. The `WORKSPACE_TYPES.field_service` entry, the Field Service workspace components, all invite/agent/admin flows referencing `field-service`, and the `desk` entry in `spaceTypes.js` were intentionally left untouched — they'll be reached when tiles-5+ wires the per-business Desk space.

### Summary of resolution status (Open Questions, Section 7)

| Question | Status |
|---|---|
| #1 Preview pulse concretely | **Superseded by Phase 4.2-tiles design (Section 8.13)** — silence-by-default (8.1) was the answer for the spinner cockpit; the tile cockpit replaces folder-position rendering entirely with a grid of child tiles, so preview pulse is no longer the question being asked at non-leaf levels. |
| #2 Home → Desk content collapse | **Superseded by Phase 4.2-tiles design (Section 8.13)** — the structural collapse question is now part of the tile cockpit's per-business folder rendering (Phase 4.2-tiles-4). Desk inherits its content as-is per the original 8.6 resolution; what changed is the surface that surrounds it. |
| #3 Playmaker contextual root vs Team-inside-Personal | **Resolved** — contextual root, default placement, current shipping behavior (8.9) |
| #4 Admin contextual root vs admin lens in spaces | **Resolved** — Thing 1 (root folder) in Phase 4; Thing 2 (lens) deferred to 4.5 (8.8) |
| #5 Personal folder vs unscoped at root | **Implicit** — Personal contains personal-flavored items (Home, Finance, Meal Prep, etc.); fast-access at root is a future shortcut concern (8.2) |
| #6 URL nesting in Phase 4 vs wait | **Superseded by Phase 4.2-tiles design (Section 8.13)** — original 8.5 resolution still holds (URL nesting waits for post-migration); the tile cockpit + breadcrumb provides the as-if-nested behavior, replacing what 4.2b would have built. |
| #7 `enabled_spaces` backfill spec | **Resolved** — concrete per-business rules (8.10). First consumer is Phase 4.2-tiles-4 (8.13). |

---

## Appendix — Files Inspected but Out of Scope for Phase 4

For completeness, files inspected during the audit but not relevant to the folder-architecture refactor:

- Per-workspace tab implementations (`components/team/*`, `components/finance/*`, `components/fieldservice/*`, `components/propertymgmt/*`, `components/mealprep/*`) — internal to leaf workspaces; structural change happens above them.
- `components/admin/*` (17+ files) — internal to the Admin page; resurface task is wiring entry, not editing internals.
- `components/onboarding/*`, per-workspace onboarding pages — not folder-tree related.
- `components/events/*`, `components/categories/*`, `components/locations/*`, `components/recommendations/*`, `components/region/*`, `components/frequency/*`, `components/field/*` — feature internals, unaffected by Phase 4.
- `src/api/*`, `src/lib/*`, `src/scripts/*` — infra; not folder-tree related.

---

*End of plan. Future Phase 4 execution prompts read this as the canonical reference for what's already standing, what gets reused, what gets moved, and what needs to be built fresh.*
