# FIELD-SERVICE-CURRENT-STATE-AUDIT.md

> **Purpose.** Pre-spec inventory of the Field Service space as it actually exists in code and live data on 2026-04-28. The financial-workflow spec that follows (estimate → contract → change orders → AR → AP → P&L) will be written against this baseline. No code changes — read-only audit.
> **Repos audited.** `community-node` (frontend at `~/Documents/GitHub/community-node/`), `Spec-Repo` (this repo). Note: the audit prompt referenced `~/Documents/LocalLane/` paths — the actual paths are `~/Documents/GitHub/...` per DEC-181.
> **Authoritative live records.** Bari Swartz's user `69b9f8313b41521cc50c97d8`, FieldServiceProfile `69baba55a6b9cca0c7d5700b` (Red Umbrella `69ea5590481b7e15af7216b6`), FSProject `69ebaa59a3b3dcbc375a7ca9` (Holman barn), FSEstimate `69c6fba24b8b8812fe4f5419` (EST-2026-004, $121,657.57, status `accepted`).

---

## 1. Executive Summary

- **The data model is roughly half of what the financial-workflow spec needs.** Estimates, projects, change orders, payments, and documents all exist as entities with reasonable shapes. What's missing is the **contract-as-distinct-thing** concept, an **AP direction** on payments, a **calculated/percentage line** on estimates, and any **project P&L rollup**. `FSPayment` is single-direction (client → contractor); `FSChangeOrder` exists but does not modify the parent estimate's total; there is no "Contract Total" field anywhere.
- **The component layer is wider than the data model lets on.** ~12.9k lines across 19 jsx files in `src/components/fieldservice/`. Estimate builder (1,670 lines) and Documents (1,902 lines) are the heaviest. Change-order create *is* wired (the prior agent inventory called it a stub — that's incorrect; `createCOMutation` calls `FSChangeOrder.create()` at [FieldServiceProjects.jsx:404-414](community-node/src/components/fieldservice/FieldServiceProjects.jsx)). What is *not* wired is any feedback loop from CO → estimate or → project budget.
- **Bari has exactly one project, one estimate, and one client at the entity layer; everything else is zero.** No `FSPayment`, no `FSChangeOrder`, no `FSDailyLog`, no `FSMaterialEntry`, no `FSLaborEntry`, no `FSPermit`. Three `FSDocument` records exist (two Bari sub-agreements, one Doron-test signed lien notice), and two custom `FSDocumentTemplate` records (his attorney-drafted contracts). Tier is `full`. `features_json` is fully populated with seven flags.
- **Mycelia is currently blind to most of Bari's records.** `agentScopedQuery` and `platformPulse` both call `base44.asServiceRole.entities.X.list()`, which per DEC-140 / DEC-095 does **not** bypass Creator Only RLS in SDK 0.8.23. So FSEstimate, FSDocument, FSClient (all Creator Only Read) return 0 to the agent layer and to the pulse health endpoint, even though the records demonstrably exist (per `JOBS-MIGRATION-PLAN.md` Section 10 verification queries). FSProject (Read=Unrestricted) is visible. **This is the single biggest "the audit is the value" finding** — the financial-workflow spec must not assume the existing read pipes will surface contract/estimate/payment data to the agent.
- **Two parallel client-portal implementations exist** and they aren't equivalent. `src/pages/ClientPortal.jsx` is the public-route signing surface (token-validated, where actual e-sign happens). `src/components/fieldservice/FieldServiceClientPortal.jsx` is an in-shell preview rendered inside the contractor's own workspace ("preview as client"). The financial-workflow spec needs to be clear about which surface is the AR-collection touchpoint for the client.
- **Six FSEstimate fields are dormant.** `management_fee_pct` (no UI), `tax_rate` (duplicate of `tax_rate_pct`), `terms` (duplicate of `payment_terms`), `is_insurance_estimate` (set but not read), `prepared_by` (set but not displayed except in JOBS-MIGRATION-PLAN reference). The math pipeline only reads `overhead_profit_pct`, `tax_rate`, `other_amount`, plus the line-item subtotal. `management_fee_pct` exists on the entity but no component surfaces or computes against it.
- **The Field Service rename to "Jobs" is fully specced and deferred.** `JOBS-MIGRATION-PLAN.md` (133k chars) is the most authoritative document in the repo for FS data model — it contains exact field-by-field migration tables and was the source of truth for reconstructing live record state during this audit. All current code uses `FS*` / `FieldServiceProfile` / `field-service`.

---

## 2. Entity Inventory

Source of truth: `community-node/base44/entities/*.jsonc` (reference copies — Base44 dashboard is canonical, but the .jsonc files are kept current via DEC-093 prompts).

### 2.1 Complete entity set

Twelve "real" Field Service entities + three Frequency Station entities that share the `FS` prefix but belong to a different feature (music/community submissions):

| Entity                  | Purpose                                | Read perm           | Live count for Bari |
|-------------------------|----------------------------------------|---------------------|---------------------|
| `FieldServiceProfile`   | Workspace config + tier + features     | Unrestricted        | **1**               |
| `FSClient`              | Contractor's client roster             | Creator Only        | invisible (≥1)¹     |
| `FSProject`             | Project master record                  | Unrestricted        | **1**               |
| `FSEstimate`            | Estimate / quote                       | Creator Only        | invisible (1)¹      |
| `FSChangeOrder`         | Change order on a project              | Unrestricted        | 0                   |
| `FSPayment`             | Payment (currently AR-only)            | Unrestricted        | 0                   |
| `FSDocument`            | Generated/signed document              | Creator Only        | invisible (≥3)¹     |
| `FSDocumentTemplate`    | Template for documents                 | Unrestricted        | invisible (6)²      |
| `FSDailyLog`            | Daily work log                         | Unrestricted        | 0                   |
| `FSDailyPhoto`          | Photo on a daily log                   | Unrestricted        | 0                   |
| `FSLaborEntry`          | Labor hours on a daily log             | Unrestricted        | 0                   |
| `FSMaterialEntry`       | Materials on a daily log               | Unrestricted        | 0                   |
| `FSPermit`              | Permit tracking                        | Unrestricted        | 0                   |
| `FSFeedback`            | App feedback / client notes            | Unrestricted        | not queried         |
| `FSFrequencyFavorite`   | (Frequency Station — unrelated)        | Creator Only        | n/a                 |
| `FSFrequencyPlaylist`   | (Frequency Station — unrelated)        | Creator Only        | n/a                 |
| `FSFrequencySubmission` | (Frequency Station — unrelated)        | Authenticated       | n/a                 |

¹ Records are demonstrably present (per `JOBS-MIGRATION-PLAN.md` Section 10) but `agentScopedQuery` returns 0 — see §11.1.
² `FSDocumentTemplate` is not in `agentScopedQuery`'s ENTITY_CONFIG allowlist; query returns "Unknown entity" 400.

### 2.2 Per-entity field tables (compact)

**FieldServiceProfile** — workspace root. FK: `user_id` → User (owner), `business_id` → Business (since Phase 2 promotion).

| Field                        | Type    | Notes                                                           |
|------------------------------|---------|-----------------------------------------------------------------|
| `workspace_name`             | string  | Display name. Bari: "Red Umbrella".                             |
| `business_name`, `owner_name`| string  | Owner-side name pair.                                           |
| `license_number`             | string  | Bari: 210412.                                                   |
| `phone`, `email`, `website`  | string  | Branding fields.                                                |
| `service_area`               | string  | **Plain string on FieldServiceProfile** — distinct from Build E's `Business.service_area` array migration (DEC-178). |
| `tagline`                    | string  | Bari: "Home and Garage Contractors".                            |
| `hourly_rate`                | number  | Default for labor entries. Bari: 0.                             |
| `logo_url`, `brand_color`    | string  | Bari: amber `#f59e0b`.                                          |
| `workers_json`               | json    | `[{name, hourly_rate, phone}, …]`. **No surface UI** — populated via Settings.                |
| `user_roles`                 | json    | Roster role list. Bari: `{}`.                                   |
| `default_terms`              | string  | Auto-loaded into estimate `terms` field on new estimates.       |
| `phase_labels`               | json    | Custom photo phase labels per workspace.                        |
| `invite_code`                | string  | Workspace-share code. Bari: `TDAqwuxj`.                         |
| `features_json`              | json    | Per-workspace feature flags. **See §8.**                        |
| `trade_categories_json`      | json    | Custom trade categories (Xactimate context).                    |
| `insurance_work_enabled`     | bool    | **Deprecated** — superseded by `overhead_profit_enabled` + `xactimate_enabled`. Still on entity. |
| `overhead_profit_enabled`    | bool    | Whether O&P% appears on estimates. Bari: false.                 |
| `xactimate_enabled`          | bool    | **Two copies — one on the entity, one inside `features_json`.** Bari has top-level=false, features_json=true. Truthy source ambiguous — see §11.5. |
| `industry_preset`            | enum    | `general_contractor` / `specialty_trade` / `auto_mechanic` / `service_provider`. Drives template seed + field labels. |
| `guide_dismissed`            | bool    | Has the walkthrough been dismissed.                             |
| `linked_business_workspace_id`, `linked_finance_workspace_id` | string | Future cross-workspace links. **Both null in production.**      |
| `subscription_tier`          | string  | `free` / `help` / `full`. Bari: `full`. Tier gating per DEC-115. |
| `tier_trial_start`           | string  | Trial-start timestamp. Bari: null.                              |
| `business_id`                | string  | Phase 2 link to Business. Bari: `69ea5590481b7e15af7216b6`.     |
| `archived_at`, `archived_by` | string  | Soft-archive support.                                           |
| `update.security/rls`        | —       | **Wide-open by design for migration** (no RLS on update). Post-migration membership gate re-tightens. (ACTIVE-CONTEXT issue #8.) |

**FSClient** — Creator Only Read. `workspace_id` (= profile_id) is required; `business_id` is the Phase 2 backfill nullable.

| Field                                            | Notes                                          |
|--------------------------------------------------|------------------------------------------------|
| `name` (req), `email`, `phone`, `address`, `city`, `state` (default `OR`), `zip_code` | Contact block. |
| `company_name`                                   | Distinct from personal `name`.                 |
| `notes`                                          | Internal contractor notes about client.         |
| `status`                                         | `active` / `inactive`.                          |
| `business_id`                                    | Phase 2 backfill (DEC-186 era).                 |

**FSProject** — Unrestricted Read. The hub that everything project-scoped joins through.

| Field                                                        | Notes                                                                           |
|--------------------------------------------------------------|---------------------------------------------------------------------------------|
| `name` (req)                                                 | Bari live: `"Dr. Nathan Holman   "` (verbatim trailing whitespace — preserve).  |
| `client_id` + denormalized `client_name`/`email`/`phone`/`address` | Denormalized client snapshot for fast list rendering.                           |
| `address`                                                    | Job-site address.                                                                |
| `status`                                                     | `active` / `quoting` / `paused` / `completed` / `cancelled`.                    |
| `start_date`, `estimated_end_date`                           | Date pair.                                                                       |
| `original_budget`, `total_budget`                            | **Two budget fields — `original_budget` is null on Bari's project.** Intent unclear; only `total_budget` ($121,657.57) populated. **Purpose unclear: §11.6.** |
| `description`, `notes`                                       | Free text. Bari notes: `"Created from estimate EST-2026-004"`.                  |
| `show_costs_to_clients`, `client_show_breakdown`             | Two related booleans controlling client-portal cost visibility.                  |
| `estimate_id`                                                | FK back to originating estimate. Bari: `69c6fba24b8b8812fe4f5419`.              |
| `business_id`                                                | Phase 2 nullable. Bari: null (not backfilled).                                  |

**FSEstimate** — Creator Only Read. The financial centerpiece today.

| Field                                                  | Notes                                                                              |
|--------------------------------------------------------|------------------------------------------------------------------------------------|
| `client_id` + denormalized `client_name`/`email`/`phone`/`address` | Denormalized client.                                                               |
| `project_id`                                           | Optional FK back to project (estimate may exist before project).                   |
| `estimate_number`                                      | Human-readable. Bari: `EST-2026-004`.                                              |
| `title` (req), `date`, `valid_until`, `due_date`       | Standard quote header.                                                              |
| `status` (req)                                         | `draft` / `sent` / `viewed` / `accepted` / `declined` / `expired`. **`signed` and `awaiting_signature` are also used in the signing flow but not in the canonical enum** — see §11.7. |
| `line_items`                                           | JSON. Schema below.                                                                 |
| `labor_estimate`                                       | JSON. **Legacy** — code unifies into `line_items` on read; new writes set `{items: []}`. |
| `subtotal`                                             | Persisted but always recomputed client-side on render.                              |
| `overhead_profit_pct`                                  | % applied to subtotal. Read by `calcTotals`.                                       |
| `management_fee_pct`                                   | **On entity but no UI / no consumer.** Schema field with zero call sites in `src/components/fieldservice/`. **§11.4.** |
| `tax_rate_pct`, `tax_rate`                             | **Duplicates.** Builder writes `tax_rate`. `tax_rate_pct` appears unused.          |
| `tax_amount`, `other_amount`, `total`                  | Persisted snapshots; recomputed on render.                                         |
| `payment_terms`, `terms`                               | **Duplicates.** Builder writes `terms`; `payment_terms` appears unused.            |
| `conditions`, `notes`, `prepared_by`                   | Free text. Bari live: `prepared_by = "Bari Swartz"`.                              |
| `is_insurance_estimate`                                | Boolean — set but not read in current builder.                                     |
| `client_show_breakdown`                                | Mirror of FSProject field.                                                         |
| `sent_at`, `viewed_at`, `responded_at`                 | Lifecycle timestamps.                                                              |
| `portal_token`, `portal_link_active`                   | Bearer token + active flag for the public client portal.                           |
| `sent_for_signature_at`, `recalled_at`                 | Signature-flow timestamps.                                                         |
| `signed_at`, `signature_data`                          | Client signature.                                                                  |
| `owner_signature_data`, `owner_signed_at`              | Owner countersignature (JSON string field).                                        |

`line_items` JSON shape (unified format, post-migration from legacy split):
```js
{ items: [
  { id, category: 'materials'|'labor'|'subcontractor'|'fee'|'other',
    trade_category_id?, description, quantity, unit, unit_price, amount, sub_name? }
]}
```

**FSChangeOrder** — Unrestricted Read.

| Field                                            | Notes                                                                           |
|--------------------------------------------------|---------------------------------------------------------------------------------|
| `project_id` (req), `estimate_id` (optional)     | CO links to project; optionally to a parent estimate.                           |
| `change_order_number`                            | Human-readable, e.g. `CO-001`. Generated by `generateCONumber(changeOrders)`.   |
| `title` (req), `description`                     | Free text.                                                                      |
| `line_items`                                     | JSON `{ items: [{description, quantity, unit_price, amount}] }`.               |
| `subtotal`, `total`                              | Persisted. **No fees/tax math on COs** — total = subtotal.                      |
| `status`                                         | `draft` / `sent` / `accepted` / `declined`. **No `signed` state.**             |
| `accepted_date`                                  | Set when `status = accepted`.                                                   |

**Critical:** there is no automated link from a CO to its parent FSEstimate's totals or to FSProject.total_budget. CO acceptance does not bump anything. The Project Detail UI lists COs and sums their totals in a panel, but that sum is display-only and does not become a "Contract Total" anywhere persistent. **§6.2.**

**FSPayment** — Unrestricted Read.

| Field                                          | Notes                                                                    |
|------------------------------------------------|--------------------------------------------------------------------------|
| `project_id` (req)                             | All payments scoped to one project. **No multi-project payment.**       |
| `type` (req)                                   | `deposit` / `progress_payment` / `final_payment` / `change_order` / `refund`. |
| `amount` (req), `date` (req)                   | Money + date.                                                            |
| `method` (req)                                 | `cash` / `check` / `card` / `transfer` / `other`.                       |
| `check_number`, `notes`                        |                                                                          |
| `status` (req)                                 | `pending` / `received` / `cleared`.                                     |

**No `direction` field.** Direction is implicit in `type`: deposit/progress/final/change_order = client-paid-us (AR); refund = we-paid-back-client (still an AR-side flow). There is no path for vendor/subcontractor payments (AP). The financial-workflow spec will need to either add a `direction` enum, split this into two entities (`FSPaymentReceived` + `FSPaymentMade`), or model AP through a separate entity. **§6.3.**

**FSDocument** — Creator Only Read. Generated from FSDocumentTemplate, signed via portal flow.

| Field                                                                | Notes                                                                  |
|----------------------------------------------------------------------|------------------------------------------------------------------------|
| `template_id`, `project_id`, `client_id` (req)                       | FK chain.                                                              |
| `title` (req), `content`                                             | Resolved content (merge fields applied).                               |
| `status`                                                             | `draft` / `sent` / `signed` / `archived`. (Note: **`awaiting_signature` is used in code paths but not in the entity enum** — same drift as FSEstimate; §11.7.) |
| `sent_at`, `signed_at`                                               | Lifecycle.                                                             |
| `signature_data`, `owner_signature_data`, `owner_signed_at`          | Dual-signature support (client + contractor).                          |
| `signer_name`, `signer_email`                                        | Captured at signing.                                                   |
| `document_url`                                                       | Generated/uploaded file URL. Often null (content is the persisted form). |
| `portal_token`, `portal_link_active`, `portal_expires_at`            | Bearer token + lifecycle.                                              |
| `recalled`, `recalled_at`                                            | Owner can recall before client signs.                                  |
| `amendment_of`                                                       | Self-FK for amendment chains.                                          |
| `archived`                                                           | Post-signing archive flag.                                             |

**FSDocumentTemplate** — Unrestricted Read; **Update is Creator Only** (a known issue per CLAUDE.md / ACTIVE-CONTEXT #7 — workaround documented).

| Field                                                | Notes                                                              |
|------------------------------------------------------|--------------------------------------------------------------------|
| `profile_id`, `business_id`                          | Two-tier scoping (DEC-163): `business_id`-set = private to that business; `is_system: true` + `business_id: null` = system template. |
| `template_type`                                      | `lien_notice` / `sub_agreement` / `contract` / `custom`.           |
| `title` (req), `description`, `content`              | Template body with `{{merge_field}}` tokens.                       |
| `merge_fields`                                       | Array of token names. **Was bugged 2026-04-23 (passed as JSON string instead of array — fixed in `9e349be`).** |
| `is_system`                                          | true → seeded by `initializeWorkspace`, read-only to user.        |
| `sort_order`                                         |                                                                    |

**FSDailyLog / FSDailyPhoto / FSLaborEntry / FSMaterialEntry / FSPermit** — all Unrestricted Read, all `project_id`-scoped, all "Phase 2 backfill nullable" `business_id` field. Used by Field Service Log tab. Bari has zero records of each. (Field tables omitted for brevity — see `base44/entities/*.jsonc` files.)

### 2.3 Cross-entity diagram

```
FieldServiceProfile (workspace root, business_id → Business)
├── FSClient        (workspace_id = profile_id)
├── FSDocumentTemplate (profile_id, business_id)
├── FSDocument      (profile_id, client_id, project_id?, template_id?)
├── FSEstimate      (profile_id, client_id?, project_id?)
└── FSProject       (profile_id, client_id?, estimate_id?)
    ├── FSChangeOrder    (project_id, estimate_id?)
    ├── FSPayment        (project_id)
    ├── FSPermit         (project_id, profile_id)
    └── FSDailyLog       (project_id, profile_id)
        ├── FSDailyPhoto   (daily_log_id)
        ├── FSLaborEntry   (daily_log_id, project_id)
        └── FSMaterialEntry (daily_log_id, project_id)
```

The relationship graph is mostly clean. The two soft spots:
- **FSEstimate ↔ FSProject is a bidirectional optional link.** `FSEstimate.project_id` is optional; `FSProject.estimate_id` is also optional. Bari's project carries an estimate_id; his estimate carries no project_id (need to confirm — invisible to scoped query).
- **FSChangeOrder.estimate_id is optional and unused in math.** A CO can be created floating (no estimate parent) or linked to an estimate, but neither case modifies the parent's totals.

---

## 3. Component Inventory

`src/components/fieldservice/` — **12,888 lines across 19 jsx files**.

| File                                  | Lines | Mounted as / role                                            | Status                  |
|---------------------------------------|-------|--------------------------------------------------------------|-------------------------|
| `FieldServiceHome.jsx`                | 436   | Home tab (workspace landing)                                 | Live                    |
| `FieldServiceLog.jsx`                 | 1,038 | Log tab (daily work entry)                                   | Live                    |
| `FieldServiceProjects.jsx`            | 1,658 | Projects tab + project detail + CO builder                   | Live                    |
| `FieldServiceEstimates.jsx`           | 1,670 | Estimates tab + estimate builder + preview + signature flow  | Live                    |
| `FieldServiceClientDetail.jsx`        | 526   | Client drill-in from Projects                                | Live                    |
| `FieldServiceClientPortal.jsx`        | 405   | **In-shell** "preview as client" — NOT the public route      | Live (preview only)     |
| `FieldServiceDocuments.jsx`           | 1,902 | Documents tab + template manager + signing flow              | Live                    |
| `FieldServicePayments.jsx`            | 279   | Payments sub-view inside Project Detail                      | Live (AR only)          |
| `FieldServicePeople.jsx`              | 874   | People tab (workers + clients)                               | Live                    |
| `FieldServicePermits.jsx`             | 536   | Permits tab                                                  | Live                    |
| `FieldServicePhotoGallery.jsx`        | 164   | Photo grid sub-view                                          | Live                    |
| `FieldServiceTimeline.jsx`            | 366   | Timeline sub-view (logs + photos)                            | Live                    |
| `FieldServiceReport.jsx`              | 387   | Reports tab                                                  | **Live but minimal** — see §3.7 |
| `FieldServiceSettings.jsx`            | 971   | Settings tab                                                 | Live                    |
| `FieldServiceOnboarding.jsx`          | 395   | Onboarding wizard component                                  | Live                    |
| `AgentChat.jsx`                       | 920   | Generic agent chat (used here for FieldServiceAgent persona) | Live                    |
| `AgentChatButton.jsx`                 | 47    | FAB                                                          | Live                    |
| `ClientSelector.jsx`                  | 228   | Reusable client picker (used in estimate builder, document creation) | Live                    |
| `VoiceInput.jsx`                      | 86    | SpeechRecognition input (notes/log)                          | Live                    |

Plus pages and admin:
- `src/pages/FieldServiceOnboarding.jsx` (5 lines — wraps the component for `/onboarding/field-service`)
- `src/pages/JoinFieldService.jsx` (248 lines — claim-first invite flow per DEC-118)
- `src/pages/ClientPortal.jsx` (~970 lines, ~289-925 are FS-specific) — **the public signing route**
- `src/components/admin/workspaces/FieldServiceDefaultsPanel.jsx` (~250 lines — admin tier-defaults UI)

### 3.1 Home tab — `FieldServiceHome.jsx`
- **Reads:** FSProject (active count), FSEstimate (recent), FSPayment (recent receipts), FSDailyLog (recent), FSDocument (awaiting signature).
- **Writes:** none.
- **Notable:** [line 431](community-node/src/components/fieldservice/FieldServiceHome.jsx) renders `"Financial summary coming soon"` placeholder. This is the only "coming soon" string in the FS layer and the natural seat for the project-financial-rollup view the workflow spec will produce.

### 3.2 Log tab — `FieldServiceLog.jsx`
- **Reads:** FSDailyLog, FSDailyPhoto, FSLaborEntry, FSMaterialEntry, FSProject.
- **Writes:** all four (DailyLog + child entries). Photos uploaded as Base44 attachments.
- **Notable:** uses `VoiceInput` for hands-free note capture. `worker_name` decoupled from `user_id` so a foreman can log labor for the whole crew.

### 3.3 Projects tab — `FieldServiceProjects.jsx`
- **Reads:** FSProject, FSClient, FSEstimate, FSPayment, FSChangeOrder, FSMaterialEntry, FSLaborEntry, FSDailyLog.
- **Writes:** FSProject (`.create`/`.update`/`.delete`), FSChangeOrder (`.create`/`.update`).
- **Sub-views in this file:**
  - **Project list** (top of file) — cards with budget bar, status badge.
  - **Project detail** (selectedProject branch) — vertical scroll with: header, financials block (budget bar + progress + recent activity), CO panel, payments panel (delegates to `FieldServicePayments`), permits, daily logs.
  - **CO builder** — at `setShowCOForm(true)`, renders an inline form ([lines 1143-1217](community-node/src/components/fieldservice/FieldServiceProjects.jsx)) with line-item rows, total computation, and a Submit button wired to `createCOMutation`. The mutation calls `base44.entities.FSChangeOrder.create({ project_id, estimate_id: project.estimate_id, change_order_number, title, description, line_items, subtotal, total, status: 'draft' })`. **The CO builder is functional, not a stub** — correcting prior agent inventory's claim. What the CO builder does *not* do: send for signature, modify the parent estimate, modify FSProject.total_budget.
- **Spend rollup:** `spendByProject` in this component sums `FSMaterialEntry.total_cost + FSLaborEntry.total_cost` per project for the budget bar. Bari's project shows $0 because he has zero log entries.

### 3.4 Estimates tab — `FieldServiceEstimates.jsx`
The financial heart of the current build. 1,670 lines.

- **Reads:** FSEstimate, FSClient, FSProject (for "Accept & Create Project" path).
- **Writes:** FSEstimate (`.create`/`.update`), FSProject (`.create` from accepted estimate at line 1239).
- **Math pipeline (`calcTotals` at line 202):**
  ```
  subtotal       = Σ line_items.amount
  opAmount       = subtotal × (overhead_profit_pct / 100)
  beforeTax      = subtotal + opAmount + other_amount
  taxAmount      = beforeTax × (tax_rate / 100)
  total          = beforeTax + taxAmount
  ```
  **All math is component-side `useMemo` — totals are persisted as denormalized snapshots but recomputed on every render. There is no server-side estimate calculator.**
- **Line item structure** (unified, after migration from legacy split):
  ```
  { id, category, trade_category_id?, description, quantity, unit, unit_price, amount, sub_name? }
  ```
- **Categories** (color-coded badges): `materials`, `labor`, `subcontractor`, `fee`, `other`.
- **Calculated / % of subtotal lines:** **Not supported.** Every line is `quantity × unit_price`. There is no "this line is N% of the subtotal" affordance. `management_fee_pct` exists on the entity (§2.2) but is not surfaced in the builder or used in `calcTotals`.
- **Deposit auto-split:** none. Estimate has free-text `payment_terms`; nothing parses "50% deposit" into a deposit line.
- **Sub-views in this file:**
  - **List view** with status badges + filtering.
  - **Detail / preview** — branded letterhead, line-items table (or Xactimate grouped-by-trade view if `xactimate_enabled`), signature badge, toolbar (Print/PDF, Accept & Create Project, Preview as Client, Copy Link, Request Signature, Recall, Edit). Edit blocked once `signed`.
  - **Builder form** — line-item rows, O&P%, tax%, other amount, terms.
- **Signing flow:** `Request Signature` button generates a UUID `portal_token`, sets `portal_link_active = true`, status → `awaiting_signature`, copies a `/client-portal?...&token=...` URL to clipboard. `Recall` flips `portal_link_active = false` + `recalled_at = now`. The actual client signing is the public `ClientPortal.jsx` route → `signEstimate` server function (§4.2).
- **`Accept & Create Project`:** sets estimate status `accepted` and creates an FSProject with `total_budget = estimate.total`, denormalizing client info onto the project. Bari's Holman project was created this way — `notes` field still says `"Created from estimate EST-2026-004"`.

### 3.5 Payments — `FieldServicePayments.jsx`
- **Reads:** FSPayment scoped to `project_id`.
- **Writes:** FSPayment.create.
- **Direction:** **AR only.** No outgoing-to-vendor path. `type` enum carries the implicit direction.
- **Running balance:** computed client-side. Reference total = `estimate.total || project.total_budget`. Subtracts `received` and `cleared` payments in date order.
- **Missing for the financial workflow spec:** payment schedule (no expected/due-date concept), late-fee tracking, multi-invoice (one balance per project), AP entirely.

### 3.6 Documents — `FieldServiceDocuments.jsx` (1,902 lines, the largest file)
- **Reads:** FSDocument, FSDocumentTemplate, FSClient, FSProject, FSEstimate.
- **Writes:** FSDocument (`.create`/`.update`/`.delete`), FSDocumentTemplate (`.create`/`.update`).
- **Two-tier template UI** (DEC-163): "RED UMBRELLA TEMPLATES" section above "DOCUMENT TEMPLATES" — partition by `business_id`. Bari sees his two attorney-drafted contracts on top, then the four Oregon system templates underneath.
- **Click-to-preview modal on every card** (DEC-165) with branded letterhead + bracketed per-client placeholders.
- **Merge field engine:** `buildMergeData(...)` resolves 50+ tokens — `{{client_name}}`, `{{project_address}}`, `{{business_name}}`, `{{business_logo_url}}`, `{{business_banner_url}}`, plus 14 `{{business_*}}` fields added in DEC-164, plus legacy `{{company_*}}` aliases preserved.
- **Signing flow:** mirrors estimate. Generates portal token, status → `awaiting_signature`, copy link to clipboard. Public route is `ClientPortal.jsx` → `signDocument`.
- **Post-signature:** [DEC-098 invitation card is construction-gated `{false && <InvitationCard />}`](community-node/src/components/fieldservice/FieldServiceDocuments.jsx) — organism-growth mechanism deferred.

### 3.7 Reports tab — `FieldServiceReport.jsx`
387 lines. Mostly summary stats + recent items. **Not a P&L.** No cost-vs-revenue rollup, no margin calc, no per-project profit. Closest thing the codebase has to a financial report and it's still a placeholder.

### 3.8 Settings — `FieldServiceSettings.jsx`
971 lines. Workspace identity (logo, brand color, license, phone, email, website, tagline, service area), workers list (`workers_json`), default terms, hourly rate, phase labels, trade categories, feature toggles surface for `features_json`. Tier display + upgrade hooks (CTA only — no Stripe).

### 3.9 People tab — `FieldServicePeople.jsx`
874 lines. Workers + Clients in one tab. Workers reads `workers_json` from profile; Clients reads FSClient.

### 3.10 Permits — `FieldServicePermits.jsx`
536 lines. Per-project permit list with statuses (`not_applied` / `applied` / `issued` / `expired`), inspections JSON (untyped — adds inline), portal/apply URL fields.

### 3.11 Onboarding — `FieldServiceOnboarding.jsx` + `JoinFieldService.jsx`
- **Onboarding wizard** (`FieldServiceOnboarding.jsx` 395 lines) — collects business name, owner name, license, phone, email, website, tagline, hourly rate, brand color, logo. Calls `ensureFieldServiceProfileForBusiness` from `src/utils/`.
- **Join flow** (`JoinFieldService.jsx` 248 lines) — claim-first per DEC-118.

### 3.12 Workspace shell + agent
- **FieldServiceAgent** is a Base44 superagent (defined in `base44/agents/`). Read-only on workspace entities (DEC-107). Reads through `agentScopedQuery`. Surfaces in the FS workspace via the shared `AgentChat` component. Tier-gated per DEC-115.

### 3.13 Admin surface — `FieldServiceDefaultsPanel.jsx`
Admin > Workspaces > FS tab. Six platform-wide feature toggles (permits, subs, mgmt fees, insurance, payments, timeline) + workspace stats. **The toggles set platform defaults but are not yet wired to per-workspace `FieldServiceProfile.features_json`** — adjusting them updates a default that new workspaces inherit; existing workspaces aren't updated.

---

## 4. Server Function Inventory

`base44/functions/` — Deno-runtime server functions. Source-of-truth is the GitHub repo; Base44 syncs from main on publish (DEC-113).

Ten functions touch FS entities:

### 4.1 `manageFieldServiceWorkspace`
**Path:** `base44/functions/manageFieldServiceWorkspace/entry.ts`
**Auth:** owner-gated (`isFSOwner` resolves caller, verifies they own the FieldServiceProfile).
**Action:** `delete_workspace_cascade` — deletes 14 entity types in dependency order. Returns `{deleted_counts}`. **No transactional rollback on partial failure.** Uses `asServiceRole` to bypass entity perms.
**Returns:** counts per entity type.
**Notable:** sequential delete loop — no concurrency control; mid-cascade failures leave partial state behind.

### 4.2 `signEstimate`
**Path:** `base44/functions/signEstimate/entry.ts`
**Auth:** **public — bearer token.** No `auth.me()`.
**Inputs:** `estimate_id`, `portal_token`, `signature_data`.
**Validation:** estimate status must be in `['awaiting_signature', 'sent']`; `portal_link_active === true`; portal_token must match.
**Effect:** sets status → `signed`, stores `signature_data` (auto-stringifies if object), `signed_at = now`, `portal_link_active = false`.
**Notable:** no PDF generation. No transaction. No replay protection beyond the status check (idempotent on repeat — second call returns 400 because status moved). **No verification that the `signature_data` is "real"** — it's whatever bytes the portal posts.

### 4.3 `signDocument`
**Path:** `base44/functions/signDocument/entry.ts`
**Mirror of `signEstimate`** for FSDocument. Same auth model, same validation, same effect.

### 4.4 `agentScopedQuery`
**Path:** `base44/functions/agentScopedQuery/entry.ts` (192 lines).
**Auth:** `auth.me()` first, falls back to `user_id` body param for MCP calls.
**Inputs:** `entity`, optional `workspace`, optional `filters`.
**Behavior:** ENTITY_CONFIG maps entity → workspace + FK field. Resolves the user's profile in that workspace, then `entities[entity].list()` + client-side filter on `r[fkField] === profileId`. Profile-type entities (FieldServiceProfile, FinancialProfile, etc.) skip the profile-resolution step and filter directly on `user_id`.
**FS coverage:** FSProject, FSEstimate, FSClient (uses `workspace_id` not `profile_id`), FSDocument, FSDailyLog, FSPayment, FSMaterialEntry, FSLaborEntry, FSChangeOrder, FSPermit, FieldServiceProfile.
**Not in allowlist:** FSDocumentTemplate, FSDailyPhoto, FSFeedback. Calling `agentScopedQuery` for these returns 400 Unknown entity.
**Critical SDK quirk:** uses `asServiceRole.entities[entity].list()`. Per DEC-140 / DEC-095 / DEC-176, this does **not** bypass Creator Only RLS in SDK 0.8.23. Result: FSEstimate, FSDocument, FSClient (Creator Only Read) come back empty even when records exist. **§11.1 below.**

### 4.5 `agentScopedWrite`
**Path:** `base44/functions/agentScopedWrite/entry.ts`
**Auth:** `auth.me()` or `user_id` fallback. Admin always allowed; non-admins require `subscription_tier === 'full'` on the workspace profile.
**Inputs:** `action: 'create'|'update'`, `workspace`, `entity`, `data`, `record_id` (for update), `user_id` (fallback).
**Three-gate enforcement** (DEC-115): agent instructions (soft) + this server function (hard) + entity perms (existing).
**Identity stamping** (DEC-139): `user_id`, `owner_id`, `created_by` are always set by the server from auth context — agent cannot override.
**FS whitelist:** FSClient, FSProject, FSDailyLog, FSMaterialEntry, FSLaborEntry, FSDailyPhoto, FSEstimate, FSPayment, FSPermit.
**Excluded from agent writes:** FSDocument, FSDocumentTemplate, FSChangeOrder. (Documents and COs require human-driven flows; templates are admin/owner.)
**Returns:** the created/updated record.

### 4.6 `initializeWorkspace`
**Path:** `base44/functions/initializeWorkspace/entry.ts`
**Auth:** owner-gated for non-admin.
**Action:** seeds default data for any workspace type. **For Field Service:** seeds 4 Oregon lien-law system templates (Information Notice to Owner, Notice of Right to Lien, Pre-Claim Notice, Subcontractor Agreement) — see §5.1.
**Idempotency:** if 4 system templates already exist for the profile, skips (unless `force=true`). Errors in individual creates don't block the rest — partial-success returns include error list.

### 4.7 `migrationHelpers`
**Path:** `base44/functions/migrationHelpers/entry.ts`
**Auth:** shared secret in `X-Migration-Secret` header.
**FS-relevant actions:**
- `create_business_from_fs_profile` — promotes a FieldServiceProfile to a Business (idempotent on `profile.business_id`).
- `archive_fs_profile` / `unarchive_fs_profile` — toggles archive state with AuditLog.
- `create_fs_document_template` — idempotent on `(business_id, title)`.
**Notable:** every mutation writes an AuditLog. Supports `dry_run`. The Phase 2 production migration (`phase-2-production-migration.js`, run 2026-04-23) promoted Bari's FS profile and loaded his two attorney-drafted templates via this function.

### 4.8 `platformPulse`
**Path:** `base44/functions/platformPulse/entry.ts`
**Auth:** API key in `x-api-key` header.
**Actions:** `health` / `feedback` / `documents` / `estimates` / `projects` / `agents` / `gardeners`.
**FS reads:** uses `asServiceRole.entities.X.list()` for FSProject, FSEstimate, FSDocument. **Same SDK quirk as `agentScopedQuery`** — `health` returns `estimates: 0` and `documents: 0` even though records exist. `projects: 1` (visible because FSProject.Read is unrestricted). **§11.1.**
**Used by:** Mycelia MCP server (DEC-099). What Mycelia "sees" of the organism's health is partial because of this.

### 4.9 `updateBusiness`
**Path:** `base44/functions/updateBusiness/entry.ts`
**FS-relevant:** owns the `enabled_spaces` and `document_logo_url` fields on Business. Bari's Red Umbrella has `enabled_spaces: ["profile", "settings", "desk"]` — the "desk" being today's name for what becomes "jobs" post-Phase-6.

### 4.10 `getMyLaneProfiles`
**Path:** `base44/functions/getMyLaneProfiles/entry.ts`
**Behavior:** parallel-fetches FieldServiceProfile + FinancialProfile + PMPropertyProfile + TeamMember + MealPrepProfile in one call (DEC-130). Returns Bari's FieldServiceProfile in the `ownedFSProfiles` array.

### 4.11 Client-side functions in `src/functions/`
Different layer — client-side helpers wrapping server fns. Nothing FS-specific in this folder.

---

## 5. Document Templates and Client Portal Flow

### 5.1 Template seeding
- **System templates** (4 Oregon lien-law docs) seeded on workspace initialization by `initializeWorkspace` action `initialize`, workspace_type `field_service`. Idempotent: skips if already seeded. All marked `is_system: true`, `business_id: null`.
- **Bari's custom templates** (2 attorney-drafted contracts) seeded by the one-off script `src/scripts/migrations/load-bari-templates.js` invoking `migrationHelpers.create_fs_document_template`. IDs:
  - `69ea7974c5f30ff25c860702` — Red Umbrella General Construction Contract (`template_type: contract`)
  - `69ea797533163127a73aeef3` — Red Umbrella Subcontractor Agreement (`template_type: sub_agreement`)
  Both have `is_system: false`, `business_id: 69ea5590481b7e15af7216b6`. Source typos preserved verbatim per the explicit "preserve source verbatim" constraint of the Bari-prep build (Section 6 appears twice in the construction contract; Section 15 of the subcontract reads "fifteen percent (25%)"; duplicate 21.2; missing 22.4 — flag for Bari to review).
- Total seeded for Bari: **6 templates** (4 system + 2 custom).

### 5.2 Two parallel client-portal implementations

**A. Public signing route — `src/pages/ClientPortal.jsx` (~970 lines).**
- Path: `/client-portal?...&estimate=ID&token=TOKEN` and `/client-portal?...&doc=ID&token=TOKEN`.
- Token validated client-side against the entity's `portal_token` field; `portal_link_active` must be true.
- Renders the document or estimate; signature pad → posts to `signEstimate` or `signDocument` server function. Server function performs the second-line token check + status mutation.
- This is where the live Bari portal tokens (`ea14972d-3978-4381-860b-849f4b4d3694` for EST-2026-004, plus the two doc tokens) resolve. JOBS-MIGRATION-PLAN.md explicitly preserves these post-migration.

**B. In-shell preview — `src/components/fieldservice/FieldServiceClientPortal.jsx` (405 lines).**
- Mounted from inside the contractor's workspace via "Preview as Client" button on the estimate detail toolbar.
- Reads project + linked payments + photos + permits; renders read-only "this is what the client sees" view.
- **Cannot accept a signature — preview only.** Client-portal AR collection happens in (A).

The two surfaces are almost-but-not-quite the same shape. The financial-workflow spec needs to decide whether to consolidate or treat them as distinct (preview vs. live) intentionally.

### 5.3 Signing flow end-to-end

```
Owner: Estimates tab → estimate detail → "Request Signature"
       → portal_token = uuid(), portal_link_active = true,
         status → awaiting_signature, sent_for_signature_at = now
       → /client-portal?...&estimate=...&token=... copied to clipboard
Owner sends URL to client (text/email — outside platform)
Client: opens URL → ClientPortal.jsx renders estimate
       → draws signature → POST signEstimate
       → server validates token + status, sets signed_at + signature_data,
         portal_link_active = false, status = signed
Client: optionally lands on post-signature CTA (currently `{false && <InvitationCard />}`, gated per DEC-098)
Owner: returns to estimate detail → sees signature badge
       → clicks "Accept & Create Project" → FSProject created with total_budget = estimate.total
```

EST-2026-004 walked exactly this path on 2026-04-24 (`responded_at: 2026-04-24T17:37:30.168Z`) — but the live record currently shows `status: accepted` and `signed_at: null`, `owner_signature_data: null` per JOBS-MIGRATION-PLAN. Bari accepted it on Dr. Holman's behalf rather than collecting a remote signature, then immediately created the project (`Created from estimate EST-2026-004`). That is the **canonical "real money flow on the platform"** to date.

---

## 6. Financial Logic — What Exists Today

### 6.1 Estimate totals
Component-side, recomputed every render. See §3.4 for the formula. Persisted snapshots on the entity are not authoritative — open the estimate, change a number, totals are recomputed. This is fine for a single-user single-record flow but creates drift risk if ever a server-side process needs to know the "true" total.

**What's there:**
- subtotal, O&P% applied to subtotal, other (flat $), tax% applied to (subtotal+OP+other), grand total.
- Line items typed by category (materials/labor/subcontractor/fee/other).
- Xactimate grouped-by-trade rendering when `xactimate_enabled`.
- Client visibility toggle: `client_show_breakdown` shows materials+labor or only the grand total.

**What's missing for the workflow spec:**
- **No calculated lines.** Cannot say "Management Fee = 10% of subtotal" as a line item.
- **No deposit auto-split.** `payment_terms` is a free-text box.
- **`management_fee_pct` field exists on the entity but no UI consumer.** Schema-level intent visible; component-level absent.
- **No "Contract Total" concept.** Once an estimate is `accepted`, the contract is implicitly the estimate's `total` — but there's no separate field, no stamping, no immutability.

### 6.2 Change orders
- FSChangeOrder entity exists. Builder is wired (§3.3). `total = subtotal` (no fees/tax math on COs).
- **CO does not modify FSEstimate.total.** No "amended estimate" concept.
- **CO does not modify FSProject.total_budget.** No automatic budget bump.
- The Project Detail UI displays a list of COs and shows their summed total in a panel — but this is render-time arithmetic, not persisted.
- **There is no "Contract Total = Original Estimate + Σ Accepted COs" anywhere.** This is the single biggest gap the financial-workflow spec needs to close.

### 6.3 Payment tracking
- FSPayment scoped to one project.
- **No `direction` field.** Implicit direction in `type`. All current code is AR (client → contractor). No path for vendor/subcontractor payments.
- Running balance computed client-side. Reference total = `estimate.total || project.total_budget`. Subtracts received + cleared.
- **No payment schedule / expected dates.** No late-fee tracking. No multi-invoice grouping.

### 6.4 Project financial rollup
- Budget bar in Projects tab: `spendByProject` sums `FSMaterialEntry.total_cost + FSLaborEntry.total_cost`.
- **No COGS breakdown** (labor vs material vs sub).
- **No profit/margin calc** anywhere.
- **No cost-projection** (estimated remaining based on schedule vs spend).
- Home tab placeholder: `"Financial summary coming soon"` ([FieldServiceHome.jsx:431](community-node/src/components/fieldservice/FieldServiceHome.jsx)).

### 6.5 Calculated / % of subtotal lines
**Not supported.** Every line item is `quantity × unit_price`. The fee categories (`fee` line-item category) are flat-dollar lines that the user types in. Adding "% of subtotal" lines requires either a new line-item shape (`{kind: 'percent_of_subtotal', percent}`) or a new estimate-level field (the dormant `management_fee_pct` is a candidate seat).

### 6.6 What "exists" in the financial space, summarized

| Capability                      | Status              | Where it lives                                        |
|---------------------------------|---------------------|-------------------------------------------------------|
| Estimate subtotal + O&P + tax + total | ✅ Live           | `calcTotals` in FieldServiceEstimates.jsx              |
| % of subtotal line items        | ❌ Not built        | —                                                     |
| Management fee field            | ⚠️ Schema-only       | `FSEstimate.management_fee_pct` (no UI)               |
| Deposit auto-split              | ❌ Not built        | `payment_terms` free text only                        |
| Contract Total                  | ❌ Not built        | No field, no concept                                  |
| Change orders (entity + builder) | ✅ Live             | FSChangeOrder + FieldServiceProjects.jsx              |
| CO modifies estimate total      | ❌ Not wired        | —                                                     |
| CO modifies project budget      | ❌ Not wired        | —                                                     |
| Payment recording (AR)          | ✅ Live             | FSPayment + FieldServicePayments.jsx                  |
| Payment direction (AP/AR split) | ❌ Not built        | implicit in `type`                                    |
| AP / vendor payments            | ❌ Not built        | —                                                     |
| Payment schedule / due dates    | ❌ Not built        | —                                                     |
| Project P&L                     | ❌ Placeholder      | FieldServiceHome.jsx:431 "coming soon"                |
| COGS breakdown                  | ❌ Not built        | —                                                     |
| Per-job margin                  | ❌ Not built        | —                                                     |
| Reports / financial dashboard    | ⚠️ Minimal stats    | FieldServiceReport.jsx (387 lines, no P&L)            |
| Stripe / actual money movement  | ❌ Deferred         | DEC-180; post-migration                                |

---

## 7. Bari Reality Check

### 7.1 What Bari actually has on the platform

Pulled live via `agentScopedQuery` and `pulse` 2026-04-28:

| Entity                | Count | Notes                                                                  |
|-----------------------|-------|------------------------------------------------------------------------|
| FieldServiceProfile   | 1     | "Red Umbrella". Tier `full`. `features_json` populated. logo + brand color set. |
| FSProject             | 1     | "Dr. Nathan Holman   " (verbatim trailing whitespace). $121,657.57 budget. |
| FSEstimate            | (1)   | EST-2026-004, status `accepted`, $121,657.57, 30 line items, `prepared_by: "Bari Swartz"`. **Invisible to scoped_query — see §11.1.** |
| FSDocument            | (≥3)  | 2 Bari sub-agreements + 1 Doron-test signed lien notice (pre-Phase-2). **Invisible to scoped_query.** |
| FSDocumentTemplate    | 6     | 4 system + 2 custom (Construction Contract + Sub Agreement, both attorney-drafted). |
| FSClient              | (≥1)  | Dr. Nathan Holman exists (FK from project). **Invisible to scoped_query.** |
| FSPayment             | 0     | **Zero payments recorded.** All payment tracking is currently in his Excel. |
| FSChangeOrder         | 0     | **Zero change orders.** Bari has been doing all CO math in Excel. |
| FSDailyLog            | 0     | No daily logs.                                                          |
| FSMaterialEntry       | 0     | No materials logged.                                                    |
| FSLaborEntry          | 0     | No labor logged.                                                        |
| FSPermit              | 0     | No permits logged.                                                      |

### 7.2 Where the platform meets Bari's current Excel workflow

Without seeing the exact Excel sheet, the natural mapping based on his JobTread-style GC workflow:

| Bari does in Excel today          | Platform supports today                                  |
|-----------------------------------|----------------------------------------------------------|
| Quote a job (line items, totals)   | ✅ FSEstimate + builder. He already used it for EST-2026-004. |
| Send quote for client signature    | ✅ Portal flow + signEstimate (he chose to internally accept on behalf instead). |
| Convert accepted quote to project  | ✅ "Accept & Create Project" wired.                      |
| Track project budget               | ⚠️ Single `total_budget` field; no contract-vs-estimate distinction. |
| Track change orders                | ⚠️ FSChangeOrder exists + builder; **but does not flow into project budget or contract total.** |
| Track money in (deposits, draws)   | ✅ FSPayment.create. AR-side only.                       |
| Track money out (subs, materials)  | ❌ Not supported. Excel-only.                             |
| Per-job profit / margin            | ❌ Not supported. Excel-only.                             |
| Multi-job rollup (this month, this year) | ❌ Not supported. Reports tab is stats only. |

### 7.3 Why Bari has zero payments / COs / logs in the platform

Two reads possible: (a) he hasn't tried; (b) he tried and it didn't fit his Excel. Doron will know. Either way: the financial-workflow spec is the natural moment to make these surfaces compelling enough that he switches off Excel for at least one of them.

---

## 8. `features_json` — Live State

Bari's FieldServiceProfile.features_json (verified by scoped_query):
```json
{
  "permits_enabled": true,
  "subs_enabled": true,
  "management_fees_enabled": true,
  "overhead_profit_enabled": false,
  "xactimate_enabled": true,
  "payments_enabled": true,
  "timeline_enabled": true
}
```

Adjacent top-level booleans on the same profile:
- `overhead_profit_enabled: false`  ← agrees with features_json
- `xactimate_enabled: false`         ← **conflicts with features_json's `true`**
- `insurance_work_enabled: false`    ← deprecated

The duplication (top-level field + features_json key) is unresolved. It's not clear which the UI consults — likely both, in different places. **§11.5.**

The seven flag names also tell us what was *envisioned*: permits, subs, mgmt fees, O&P, Xactimate, payments, timeline. The financial-workflow spec is implicitly an expansion of "payments" into AR/AP + a new "contract" / "p_and_l" / "ar_aging" surface, plus turning the dormant `management_fees_enabled` flag into something with a real consumer.

Not queried in this audit (other workspaces): no other FieldServiceProfiles besides Bari's currently exist beyond a Doron sandbox archived 2026-04-23. So Bari's flag state is essentially the only live datum.

---

## 9. Spec-Repo Cross-Reference

### 9.1 What we found

| Spec                                            | Where                                                         | Date           |
|-------------------------------------------------|---------------------------------------------------------------|----------------|
| `JOBS-MIGRATION-PLAN.md`                        | Spec-Repo root                                                | 2026-04-28     |
| `PHASE-4-MIGRATION-PLAN.md`                     | Spec-Repo root                                                | 2026-04-26 + Section 8 amendments through 2026-04-28 |
| `STATUS-TRACKER.md`                             | Spec-Repo root                                                | through 2026-04-28 |
| `DECISIONS.md`                                  | Spec-Repo root + community-node mirror (DEC-092+)             | through 2026-04-28 |
| `ACTIVE-CONTEXT.md` / `PROJECT-BRAIN.md`        | community-node/context/                                       | mirror of Spec-Repo canonical |
| `audits/DRIFT-REPORT-2026-03-30.md`             | Spec-Repo audits/                                             | 2026-03-30 — references Bari prep |
| `FIELD-SERVICE-WORKSPACE-SPEC.md`               | **does not exist** at Spec-Repo root                          | —              |
| `AGENT-SCOPED-QUERY-SPEC.md`                    | **does not exist** in Spec-Repo (lives in `private/`)         | —              |
| `AGENT-WRITE-AND-MYLANE-CONSOLE-SPEC.md`        | **does not exist** in Spec-Repo (lives in `private/`)         | —              |

### 9.2 What `JOBS-MIGRATION-PLAN.md` asserts vs current code

This is the single most authoritative spec for the FS data model — its Section 10 verification queries are the proof that EST-2026-004 etc. exist as described. Specifically:
- Bari's FSProject id `69ebaa59a3b3dcbc375a7ca9` ✅ verified by scoped_query.
- Bari's FSEstimate id `69c6fba24b8b8812fe4f5419`, EST-2026-004, status `accepted`, total $121,657.57 ✅ verified by JOBS-MIGRATION-PLAN; **invisible to scoped_query** (RLS quirk).
- Three live portal tokens must continue to resolve post-migration ✅ matches code at `src/pages/ClientPortal.jsx`.
- Bari's FieldServiceProfile.business_id = `69ea5590481b7e15af7216b6` ✅ verified by scoped_query.
- Red Umbrella `enabled_spaces: ["profile", "settings", "desk"]` ✅ documented; will become `["profile", "settings", "jobs"]` post-Phase-4 backfill.

Everything in JOBS-MIGRATION-PLAN about the FS data model checks out against the live records and the code.

### 9.3 Decisions that affect Field Service

From the community-node DEC-092+ ledger and the Spec-Repo DEC-001-091 ledger (snapshot, not exhaustive):

- **DEC-085 (private)** — Original Field Service Workspace spec. Not in Spec-Repo (in `private/`).
- **DEC-093** — Entity changes via Base44 prompts.
- **DEC-095** — `asServiceRole` does NOT bypass Creator Only Update. Foundation of the SDK quirk story.
- **DEC-096 / DEC-097 / DEC-098** — One-click Request Signature; documents grouped by client; post-signature invitation gated.
- **DEC-107** — `agentScopedQuery` as the permission membrane.
- **DEC-115** — `agentScopedWrite` + tier gating + Mylane console.
- **DEC-136** — Creator Only as default entity permission.
- **DEC-139** — Server-authoritative identity stamping on agent writes.
- **DEC-140** — `readTeamData` pattern (the same SDK quirk story applied to team entities; the FS layer hasn't yet adopted the same workaround for FSEstimate/FSDocument/FSClient).
- **DEC-163 / DEC-164 / DEC-165 / DEC-166 / DEC-167** — Bari-prep document templates (two-tier, business-first branding, click-to-preview, schema-conformance audit).
- **DEC-170** — "Jobsite → Desk" vocabulary; FieldServiceAgent persona name preserved.
- **DEC-175** — Migration Pattern C+; deferred to Phase 6.
- **DEC-186** — `Business.enabled_spaces` field + Settings as a universal space.
- **DEC-190** — Space-type catalog (`src/config/spaceTypes.js`).

The flow from these: Field Service was built node-first, then promoted into the multi-business architecture via Phase 2, then will be renamed to Jobs and migrated to Supabase via Phase 6. The financial workflow spec is being written **between** those two events — close enough to migration that any new entity should be designed with the Jobs renaming in mind, but soon enough that the build still happens against Base44.

### 9.4 Where the specs match the code

- **Two-tier templates (DEC-163)** — match. Documents UI partitions on `business_id`.
- **Tier gating (DEC-115)** — matches. agentScopedWrite checks `subscription_tier === 'full'` server-side.
- **Server-authoritative identity (DEC-139)** — matches. agentScopedWrite stamps user_id from auth.
- **Three-gate agent enforcement** — matches. agentScopedQuery + agentScopedWrite are the hard gate.
- **Phase 2 Bari promotion** — matches. business_id present on profile; reparenting via migrationHelpers ran 2026-04-23.

### 9.5 Where the specs and the code drift

- **FS-WORKSPACE spec referenced in audit prompt does not exist in Spec-Repo** (it's in `private/` — not audited here per scope).
- **Field Service rename to Jobs** is fully specced (`JOBS-MIGRATION-PLAN.md`) but **completely absent from current code**. All FS-prefix names remain. Doron has confirmed the rename is deferred to post-Phase-6 — this is not drift, it's intent.
- **`management_fee_pct` field exists on FSEstimate** but no spec articulates a use for it, no UI consumes it. **§11.4.**
- **`xactimate_enabled` exists in two places** (top-level + features_json). No spec resolves which is canonical.
- **Client portal preview vs public** — the duality isn't documented anywhere as intentional.
- **`agentScopedQuery` returns 0 for Creator Only entities** — the spec describes it as the permission membrane (and it does enforce scoping correctly), but the spec doesn't address the SDK-quirk read failure. DEC-140 documents the same problem for team entities and provides the workaround (entity perms → Authenticated, scoping in server function). FS layer has not yet adopted this pattern for FSEstimate/FSDocument/FSClient.

---

## 10. Known Issues, Deferrals, and Construction-Gated Territory

Compiled from `ACTIVE-CONTEXT.md` "Known Issues" section + audit walkthrough.

| # | Item | Source | Type |
|---|------|--------|------|
| 1 | `community-node/docs/migration-research.md` supposed to be deleted post-DEC-175; still present | ACTIVE-CONTEXT #1 | Cleanup |
| 2 | DECISIONS.md drift between Spec-Repo and community-node | ACTIVE-CONTEXT #2 | Doc drift |
| 3 | Persistent Base44 SDK 404 console spam on every page load | ACTIVE-CONTEXT #4 | Quirk |
| 4 | `Business.categories` field empty in Base44 (Phase 5 cleanup) | ACTIVE-CONTEXT #6 + DEC-176 | Pending cleanup |
| 5 | **FSDocumentTemplate `rls.update` creator-only** — workaround documented | ACTIVE-CONTEXT #7 | Quirk |
| 6 | **Phase 2 FieldServiceProfile + User `security.update: true` with no RLS** — wide-open by design for migration | ACTIVE-CONTEXT #8 | Intentional, time-boxed |
| 7 | Base44 publish blocker — escalation `95a004a0` open | ACTIVE-CONTEXT #9 | External |
| 8 | Field Service workspace assumes personal scope today (`MyLaneDrillView.jsx:53` resolves `fieldServiceProfiles?.[0]`); per-business Desk wiring needed for tiles-5 | ACTIVE-CONTEXT, DEC-186 | Pending build |
| 9 | **Bari's Estimates surface is dark in HomeFeed until tiles-5 wires per-business Desk** | ACTIVE-CONTEXT | Real-user signal blackout window — load-bearing for the spec |
| 10 | JoinFieldService welcome-card "Go to desk" soft-broken (gracefully no-ops) | ACTIVE-CONTEXT #14 | Cleanup |
| 11 | Construction-gated post-signature invitation card (`{false && <InvitationCard />}`) | DEC-098 | Deferred organism-growth feature |
| 12 | "Financial summary coming soon" placeholder on Home tab | FieldServiceHome.jsx:431 | Natural seat for the workflow spec |
| 13 | FSEstimate.`management_fee_pct` field with no UI consumer | §6.5 | Schema-only |
| 14 | FSEstimate.`tax_rate` ↔ `tax_rate_pct` field duplication | §2.2 | Schema cleanup |
| 15 | FSEstimate.`terms` ↔ `payment_terms` field duplication | §2.2 | Schema cleanup |
| 16 | FieldServiceProfile.`xactimate_enabled` (top-level) ↔ `features_json.xactimate_enabled` (nested) — values disagree on Bari's profile | §8 | Conflict |
| 17 | FSEstimate.`is_insurance_estimate` set but never read in current builder | §2.2 | Dead-flag candidate |
| 18 | FieldServiceDefaultsPanel toggles update platform defaults but don't propagate to existing FieldServiceProfile.features_json | §3.13 | Half-built |
| 19 | FSChangeOrder does not modify parent estimate's `total` or project's `total_budget` | §6.2 | **Major workflow gap** |
| 20 | No "Contract Total" concept anywhere | §6.1 | **Major workflow gap** |
| 21 | FSPayment is AR-only; no AP / vendor / sub payments | §6.3 | **Major workflow gap** |
| 22 | No project P&L / margin / COGS rollup | §6.4 | **Major workflow gap** |
| 23 | `agentScopedQuery` returns 0 for Creator Only entities (FSEstimate, FSDocument, FSClient) due to SDK 0.8.23 RLS quirk | §11.1, DEC-140 pattern | **Major Mycelia visibility gap** |
| 24 | `platformPulse health` returns `estimates: 0, documents: 0` for the same reason | §11.1 | Same root cause |
| 25 | Reports tab is summary stats, not a P&L | §3.7 | Half-built |
| 26 | Two parallel client-portal implementations (preview vs public) — duality not documented | §5.2 | Potential cleanup |
| 27 | `manageFieldServiceWorkspace.delete_workspace_cascade` — no transactional rollback on partial failure | §4.1 | Risk if used at scale |
| 28 | FSChangeOrder.status enum lacks `signed`; FSDocument/FSEstimate code uses `awaiting_signature` not in entity enum | §11.7 | Schema/code drift |

---

## 11. Open Questions for Mycelia

### 11.1 The Creator Only RLS read quirk
`agentScopedQuery` and `platformPulse` both call `asServiceRole.entities.X.list()`. Per DEC-140 / DEC-095 / DEC-176 this does NOT bypass Creator Only RLS in SDK 0.8.23 — confirmed today by the live data: scoped_query returns 1 FSProject (Read=Unrestricted) and 0 FSEstimate / 0 FSDocument / 0 FSClient (Read=Creator Only) for Bari, even though those records demonstrably exist.

**Open question:** does the FS layer adopt the DEC-140 pattern (loosen FSEstimate / FSDocument / FSClient Read to Authenticated, enforce scoping in `agentScopedQuery` server-side) — or wait for Supabase migration? Current state means **Mycelia and the Mylane agent are blind to Bari's signed estimate, his contract documents, and his client roster.** The financial-workflow spec depends on the agent being able to *see* this data.

### 11.2 The two FSProject budget fields
`original_budget` (null on Bari's record) and `total_budget` ($121,657.57). Bari's Holman project came from "Accept & Create Project" which set `total_budget`. **Purpose unclear:** was `original_budget` meant to be set to the same value at project-creation time, with `total_budget` then accumulating COs? If so, the wiring is incomplete (no CO bumps `total_budget`). Or is one of them legacy?

### 11.3 The two client-visibility booleans
`FSProject.show_costs_to_clients` and `FSProject.client_show_breakdown`. Plus `FSEstimate.client_show_breakdown`. Three booleans, two entities. **Purpose unclear** which one drives which surface.

### 11.4 `management_fee_pct`
Field exists on FSEstimate. No UI surfaces it. No `calcTotals` consumer. `features_json.management_fees_enabled = true` for Bari — implying the flag is "on" but the feature isn't wired. **Was this scoped, half-built, then paused? Or was it built and removed?** Git blame on the entity .jsonc would tell us.

### 11.5 `xactimate_enabled` duplication
Top-level boolean on FieldServiceProfile = false; `features_json.xactimate_enabled` = true. **Which wins?** If the estimate-builder reads top-level it disables Xactimate mode for Bari; if it reads features_json it enables it. The financial-workflow spec needs to know where to write the new flag (e.g., `ar_aging_enabled`, `cogs_enabled`).

### 11.6 Bari's zero-state
Bari has zero FSPayment, zero FSChangeOrder, zero FSDailyLog. Has he tried these surfaces and found them lacking, or has he never tried them? The spec strategy depends on the answer.

### 11.7 Status enum drift
FSEstimate entity enum: `draft / sent / viewed / accepted / declined / expired`. Code paths use `awaiting_signature` and `signed` (signEstimate sets status to `signed`). Same for FSDocument. **Are the entity enums out-of-date with code?** Likely — they predate the signing flow build. The financial-workflow spec should not assume the enums are authoritative.

### 11.8 The FSDocumentTemplate access gap
`FSDocumentTemplate` is in the entity allowlist of nothing — not `agentScopedQuery`, not `agentScopedWrite`. The Documents UI reads templates via raw `base44.entities.FSDocumentTemplate.list()` (which works because Read is unrestricted), but the agent layer cannot. Intentional?

---

## 12. What I Decided and Why

A few judgment calls flagged inline:

- **I called the change-order builder "live, not a stub."** The prior agent inventory described it as a stub form with no save path. Reading the code (`FieldServiceProjects.jsx` lines 102-107 + 404-414 + 1143-1217) shows a wired `createCOMutation` that creates FSChangeOrder records. What it does *not* do is feed back into the parent estimate or project budget — that's the real gap. Calling it a stub overstates the absence; calling it a "major workflow gap" (§6.2 / §10 #19) is the accurate framing.
- **I treated FSFrequency* entities as out-of-scope.** They share the FS prefix but belong to Frequency Station (community music feature), not Field Service. Listed in §2.1 for completeness but not analyzed.
- **I did not query the `private/` repo specs** (FIELD-SERVICE-WORKSPACE-SPEC, AGENT-SCOPED-QUERY-SPEC, AGENT-WRITE-AND-MYLANE-CONSOLE-SPEC). The audit prompt referenced them; they live in `~/Documents/GitHub/private/` not Spec-Repo. The DEC-107 / DEC-115 entries in the public DECISIONS.md ledger plus the actual server-function code give us enough to triangulate. If reading the private specs would change a finding, that would be a follow-up.
- **I flagged the agentScopedQuery RLS read quirk as the headline finding.** It's the most consequential thing the audit surfaced because it directly affects whether the financial-workflow spec can assume the agent layer "sees" contracts and payments. Without resolution, every read-side workflow path through Mylane is half-blind to Bari's actual records.
- **I did not attempt to read the Excel spreadsheet.** It wasn't in the repos and the audit prompt referenced it as something Doron has — left for him to bring into the spec session.

## 13. Verification I Couldn't Complete

- **Live entity counts for FSEstimate / FSDocument / FSClient.** Blocked by §11.1. The audit reports inferred counts (1 / ≥3 / ≥1) from JOBS-MIGRATION-PLAN.md.
- **Reading `private/` specs.** Out of scope and the public DECISIONS / function code substituted.
- **Bari's Excel spreadsheet.** Not in any of the audited repos.
- **Whether `features_json.management_fees_enabled = true` causes any code path to behave differently.** Grep finds no consumer; flagged in §11.4.
- **Git blame on FSEstimate.jsonc to date `management_fee_pct`.** Skipped; can be done quickly if needed.
- **The full FSEstimate.line_items JSON for EST-2026-004.** JOBS-MIGRATION-PLAN says "30 line items" — the actual content is invisible to the audit through the agent membrane. The exact structure of Bari's line items would inform the spec but is not strictly required.

---

*End of audit.*
