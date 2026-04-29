# FIELD-SERVICE-FINANCIAL-WORKFLOW-SPEC.md

> **Status.** V2 spec, Phase 1 explicit. Drafted 2026-04-29 by Mycelia after a sequence of design conversations that established the architectural primitives, the three-billing-models commitment, the library-as-speed concept, and the layer-aware cost tracking principle. Then narrowed to Phase 1 scope after Doron clarified that the full V1 ships post-migration; Phase 1 is what Bari needs now on existing Base44 infrastructure.

> **Scope.** The destination architecture for the financial spine of the Field Service workspace, with Phase 1 explicitly scoped as the subset that ships before Supabase + Vercel migration.

> **Companion.** See `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` for the strategic context — why three billing models are first-class, why the library is the speed, why LocalLane is becoming a general contractor to general contractors, and the connection-to-source philosophy that anchors transparency as architecture.

> **Out of scope for this spec entirely.** Industry preset wiring (separate spec — but financial workflow is preset-aware). Business Finance space (separate spec — but financial workflow is cross-space-aware). Live local supplier pricing feed (separate spec — but library shape anticipates it). Open Postings / Bid Responses (separate spec — but FSEstimate shape anticipates it). AI estimate generation (separate spec — but data shape reserved). Stripe / online payment processing. Multi-currency.

> **Deferred to post-Phase-6 migration.** Field Service rename to "Jobs" — SQL ALTER at Supabase migration time.

---

## 1. Why this spec exists

Bari Swartz is the only live Field Service user. He has a $121,657 signed estimate with Dr. Holman (EST-2026-004) and a project tracking it. Everything else — the change orders he's already done, the payments he's already received from Dr. Holman, the payments he's made to subs and vendors, the running profit math — lives in an Excel spreadsheet because the platform doesn't support it yet.

Bari is also a *type*, not the audience. The same financial spine — signed estimate as contract, amendments, payments in, payments out, project profit — applies to any service business doing multi-stage work with multi-party money flow. A wedding photographer tracking retainer/shooting/album payments. A structural engineer billing in stages. A piano teacher selling lesson packages. A landscape designer doing a six-month installation.

This spec defines that financial spine **once**, generically, then leans on the industry preset system (separate spec) to adapt the visible shape per business type. Bari is the validation user, not the design target.

For the strategic depth — why this matters beyond the technical mechanics, what LocalLane is becoming through this work — see the companion intent doc in private.

---

## 2. Architectural primitives

Seven primitives anchor this spec. Each was settled in design conversation; each is restated here for the build phase to reference.

### 2.1 The estimate is the project spine

When a client signs an estimate, the estimate becomes the contract for the project. Every subsequent piece of financial data — labor logged, materials purchased, payments received, payments made out, change orders, allowance reconciliations — references the estimate, its line items, or its amendments. **There is no loose financial activity in Field Service.** Project-less expenses belong in the future Business Finance space.

The estimate is `FSEstimate`. The signed amount is the **Original Contract Amount** and is immutable from the moment of signing forward.

### 2.2 Three billing models, all first-class (post-migration)

Three pricing models exist as first-class citizens in the destination architecture:

- **`fixed_price`** — Client pays the agreed contract amount regardless of actual costs. Contractor's margin is hidden. Estimate is the bill.
- **`cost_plus`** — Client pays actual costs plus an agreed markup. Total revenue depends on actual costs. Client sees costs because they're the basis for the bill. Actuals are the bill.
- **`time_and_materials`** — Client pays an hourly rate plus material reimbursement at agreed rates. Logs and receipts are the bill.

`FSProject.billing_model` is a project-level enum. Each model has a distinct estimate emphasis, billing flow, and client portal view.

**Phase 1 only ships fixed-price.** Bari uses fixed-price-with-management-fee exclusively. Cost-plus and T&M ship post-migration when the data model can fully support them and when we have users who need them. Phase 1 doesn't even include the field — projects are implicitly fixed-price. The field gets added at the start of the post-migration build.

### 2.3 Change orders are additive amendments, not replacements

When work scope changes mid-project — discovered conditions, client requests, deductive scope reductions — the original signed contract is not modified or replaced. A separately-numbered, separately-signed amendment is created (`FSChangeOrder`). The signed contract continues to exist as the legal record of what the client agreed to. The change order extends or reduces the contract by its own dollar amount.

**Contract Total = Original Contract Amount + sum of approved Change Orders.**

This matches industry practice across every credible construction-management platform researched (JobTread, Buildertrend, Houzz Pro, Procore). Replacement-style change orders destroy legal evidence and create dispute exposure; the additive pattern preserves the audit trail.

### 2.4 Line items carry three numbers

Every line on an estimate carries:
- **Estimated** — what the client agreed to pay for this line item
- **Billed** — what's been invoiced or allocated against this line (cumulative)
- **Cost** — what the contractor has spent against this line (sum of expense logs attached)

Variance is the difference. The system makes variance *visible*; the contractor chooses the resolution path (change order, eat the margin, allowance reconciliation, refund). The platform doesn't auto-resolve variance — that's a contractor judgment call rooted in conversations with the client.

**Layer-aware tracking.** Cost granularity matches the contractor's layer in the supply chain. A general contractor like Bari who subs everything tracks line cost as the sum of his sub payments — he doesn't break that $5K sub payment into "$2.8K lumber, $2.2K labor" because that's the sub's job. A sub doing the actual work tracks materials and labor separately because they're touching real costs. A supplier tracks neither. **Phase 1 honors Bari's layer**: line cost in his world is the sum of FSPayment records (direction=paid) tied to that line. FSMaterialEntry and FSLaborEntry exist but are optional in Phase 1; they become more central post-migration when subs start using LocalLane themselves.

In cost-plus and T&M (post-migration), the client sees all three columns. In fixed-price (Phase 1), the client only sees Estimated and Billed; Cost is the contractor's internal view.

### 2.5 The library is the speed (post-migration)

Estimating gets faster every time it's used because the contractor's personal library grows. Three layers of reusable building blocks accumulate:

- **Cost Items** — saved single line entries
- **Assemblies** — multi-item bundles that scale with dimensions
- **Estimate Templates** — full project skeletons

The library is also the seam for live local supplier pricing (each cost item references a material; supplier feed updates prices automatically) and AI generation (when AI estimate creation ships, AI generates into the same Cost Item / Assembly structure).

**Phase 1 does not ship the library.** Bari's volume (~10 estimates per year) doesn't justify the overhead of building and managing a library now. Phase 1 estimating uses the existing free-form line-item builder. Library entities (FSCostItem, FSAssembly, FSEstimateTemplate) ship post-migration.

### 2.6 Log is the universal capture surface

Every financial event in the workspace flows through one entry point: the Log tab. Labor hours, expenses, payments received, payments paid out, photos, notes, milestones — all captured through Log with a type picker that adapts the form to the entry type. Project + Client are picked once at the top; type determines the rest.

Underlying entities remain distinct (`FSLaborEntry`, `FSPayment`, `FSMaterialEntry`, etc.) — the unification is in the **input surface**, not the data model. Logs link to specific line items where applicable, which is what makes line-level rollup possible.

**Phase 1 expands Log incrementally.** Today's Log handles labor, materials, photos, notes, daily entries. Phase 1 adds Sub Payment and Client Payment as type-picker entries (which write to FSPayment with appropriate direction). The full universal-capture vision (every type unified through one form, voice-input everywhere, quick-re-entry, etc.) ships post-migration.

### 2.7 Business is the unit of truth

Every space in a business inherits canonical facts from the Business record (and, when relevant, from sibling spaces). The Field Service space pre-fills industry preset, branding, contact info, and other shared facts from Business. New facts learned in Field Service that are canonical (a phone number that becomes the customer-facing number, branding details set in the workspace) propagate back to the Business record. Cross-space data flows are explicit and additive.

For this spec, the load-bearing implications are:

- When a user adds Field Service to a business, the wizard reads `Business.category`, `Business.archetype`, and other identity fields and pre-fills `FieldServiceProfile`. The user confirms; they don't re-enter.
- The financial entities (`FSEstimate`, `FSPayment`, `FSChangeOrder`, etc.) are designed so a future Business Finance space can read them as the canonical source for income/expense flows. No duplication, no authority conflicts — Business Finance reads, Field Service writes.
- The library (post-migration) is owned by the FieldServiceProfile but cross-references can be made to Business-level branding, contact info, and preset.

---

## 3. Entity changes

### 3.1 Phase 1 — minimal changes to existing entities

**FSEstimate** — line item shape gets two new flags. Both default to false; existing line items render identically.
- `is_calculated`: boolean — true = % of subtotal (Management Fee, Insurance)
- `calculation_basis`: enum, nullable — `subtotal | subtotal_plus_op | labor_only | materials_only | null`
- `calculation_pct`: number, nullable

**FSPayment** — gains direction and party fields:
- `direction`: enum — `received | paid` (default: `received` for existing records)
- `party_type`: enum — `client | subcontractor | vendor | other` (default: `client` for existing records)
- `party_name`: string — denormalized; required on new records, backfilled from FSClient.name for existing AR records
- `method`: enum — `check | cash | ach | card | other` (if not already present)
- `reference`: string, nullable — check number, transaction ID, etc.
- `notes`: text, nullable

**FSChangeOrder** — wires into project budget math:
- `co_number`: string — sequential per project (CO-001, CO-002, ...)
- existing `status` enum gains `awaiting_signature` and `signed` (already added in cleanup commit e57010f for FSEstimate; mirror for FSChangeOrder)
- `signed_at`: datetime, nullable
- `signature_data`: text, nullable — JSON.stringify'd signature blob, mirrors FSDocument pattern
- `amount`: number — already exists on the entity per audit; net contract adjustment

**FSProject** — `original_budget` becomes immutable; `total_budget` becomes derived:
- `original_budget` — set ONCE when an estimate is accepted and project is created; immutable thereafter. Backfilled from signed estimate total for existing projects.
- `total_budget` — = `original_budget + sum(FSChangeOrder.amount where status = signed and project_id = this project)`. Recomputed on every CO state change.

That's the entire Phase 1 entity scope. No new entities. Six new fields across three existing entities. One field semantic change (`original_budget` immutability).

### 3.2 Post-migration — destination architecture

Once on Supabase + Vercel infrastructure, the destination architecture lands. The remainder of this section describes that architecture; Hyphae implements Phase 1 only and leaves the destination work for the post-migration build sequence.

#### 3.2.1 New entity: FSCostItem (post-migration)

The contractor's reusable line-item library. Single saved entries that get pulled into estimates by name.

Fields:
- `id`, `profile_id` (FK to FieldServiceProfile)
- `name`: string
- `description`: text, nullable
- `category`: enum — `materials | labor | subcontractor | fee | other`
- `default_unit`: enum, nullable — `sq_ft | linear_ft | each | hours | day | lump`
- `default_rate`: number
- `default_qty`: number, nullable — for assemblies that include this item with a fixed qty
- `cost_basis`: number, nullable — contractor's actual cost (for cost-plus)
- `trade_category`: string, nullable — for Xactimate grouping
- `tags`: array of strings
- `archived_at`: datetime, nullable
- `supplier_link_id`: string, nullable — reserved for live supplier feed
- `ai_generated`: boolean — reserved for AI-created items
- `created_at`, `updated_at`, `created_by`

#### 3.2.2 New entity: FSAssembly (post-migration)

Multi-item bundles that drop into estimates as a unit and expand into individual line items.

Fields:
- `id`, `profile_id` (FK)
- `name`: string
- `description`: text, nullable
- `scaling_unit`: enum, nullable — `sq_ft | linear_ft | each | hours | null`
- `default_dimension`: number, nullable
- `cost_items`: array of references, each with:
  - `cost_item_id`: string (FK to FSCostItem)
  - `qty_per_unit`: number — qty per scaling_unit
  - `qty_fixed`: number, nullable — alternative to per-unit
  - `notes`: string, nullable
- `trade_category`: string, nullable
- `tags`: array of strings
- `archived_at`: datetime, nullable
- `ai_generated`: boolean
- `created_at`, `updated_at`, `created_by`

#### 3.2.3 New entity: FSEstimateTemplate (post-migration)

Full project skeletons. Pre-built estimate structures with assemblies and cost items pre-loaded.

Fields:
- `id`, `profile_id` (FK)
- `name`: string
- `description`: text, nullable
- `project_type`: string, nullable
- `default_billing_model`: enum
- `sections`: array of section objects, each with `name` and `line_items` array (same shape as FSEstimate.line_items)
- `default_management_fee_pct`: number, nullable
- `default_overhead_profit_pct`: number, nullable
- `default_tax_rate`: number, nullable
- `default_terms`: text, nullable
- `archived_at`: datetime, nullable
- `ai_generated`: boolean
- `created_at`, `updated_at`, `created_by`

#### 3.2.4 FSEstimate destination shape (post-migration)

Line items become fully structured. Each line carries:
- `id`: string — stable id for cross-referencing from Log
- `description`: string
- `category`: enum — `materials | labor | subcontractor | fee | other | calculated | allowance`
- `qty`: number, nullable
- `unit`: string, nullable
- `rate`: number, nullable
- `amount`: number
- `cost_item_id`: string, nullable — FK to FSCostItem
- `assembly_id`: string, nullable — FK to FSAssembly
- `assembly_group_id`: string, nullable — for display grouping
- `is_allowance`: boolean
- `is_calculated`: boolean (Phase 1: already exists)
- `calculation_basis`: enum, nullable (Phase 1: already exists)
- `calculation_pct`: number, nullable (Phase 1: already exists)
- `trade_category`: string, nullable
- `cost_basis`: number, nullable
- `notes`: string, nullable

Top-level FSEstimate additions:
- `billing_model`: enum — `fixed_price | cost_plus | time_and_materials`
- `template_id`: string, nullable
- `generation_source`: enum — `manual | library | template | ai`
- `show_costs_to_client`: boolean

#### 3.2.5 FSPayment destination shape (post-migration)

Phase 1 fields plus:
- `line_item_id`: string, nullable — payment can be allocated to a specific line OR float at project level
- `change_order_id`: string, nullable — payment can be against a specific CO
- `allocation_status`: enum — `unallocated | allocated | partially_allocated`
- `parent_payment_id`: string, nullable — when a payment is split via allocation, child records reference parent
- `recipient_profile_id`: string, nullable — reserved; when filled, mirrors payment to recipient's FieldServiceProfile (cross-tier connection)

#### 3.2.6 FSDailyLog / FSLaborEntry / FSMaterialEntry — line linkage (post-migration)

Add to each:
- `line_item_id`: string, nullable — FK to a specific line on the project's estimate
- `cost_item_id`: string, nullable — FK to FSCostItem (FSMaterialEntry primarily)

#### 3.2.7 New entity: FSAllowanceReconciliation (post-migration)

When an allowance line's actual cost differs from the allowance amount.

Fields:
- `id`, `project_id`, `estimate_id`, `line_item_id`
- `allowance_amount`: number
- `actual_amount`: number
- `variance`: number
- `resolution`: enum — `client_pays_difference | contractor_absorbs | deductive_co | credit_to_client`
- `reconciled_at`: datetime
- `reconciled_by`: user_id
- `notes`: text
- `generated_co_id`: string, nullable
- `generated_payment_id`: string, nullable

### 3.3 Relationship summary (destination, post-migration)

```
Business
  └── FieldServiceProfile (industry_preset reads from Business.category/archetype)
        ├── FSCostItem (library — reusable single line items) [POST-MIGRATION]
        ├── FSAssembly (library — reusable multi-item bundles) [POST-MIGRATION]
        ├── FSEstimateTemplate (library — full project skeletons) [POST-MIGRATION]
        ├── FSClient
        ├── FSEstimate (line_items reference cost_items/assemblies; billing_model declared post-migration;
        │              status: draft → sent → awaiting_signature → signed → accepted → declined → expired)
        │     └── FSProject (original_budget locked at acceptance; billing_model inherited post-migration)
        │           ├── FSChangeOrder (additive amendments; line_items use same library post-migration;
        │           │                  contributes to total_budget when signed) [WIRED IN PHASE 1]
        │           ├── FSPayment (direction: received|paid [PHASE 1]; line_item_id linkage,
        │           │              allocation_status, parent_payment_id [POST-MIGRATION])
        │           ├── FSLaborEntry (line_item_id linkage post-migration)
        │           ├── FSMaterialEntry (line_item_id + cost_item_id linkage post-migration)
        │           ├── FSDailyLog (photos, notes, milestones)
        │           ├── FSPermit (existing, unchanged)
        │           └── FSAllowanceReconciliation (created at project close) [POST-MIGRATION]
        └── FSDocument (templates and signed docs; not financial-spine but shares signing infrastructure)
```

---

## 4. Surfaces

### 4.1 Phase 1 surface changes

**Estimate Builder.** The existing builder (Labor / Subcontractor / Fee categories, qty / unit price / amount, voice input on description) stays structurally intact. Phase 1 additions:

- **Calculated line type** — adds a new line category option ("Management Fee" / "Insurance" / "Custom %"). When picked, the line collapses qty/rate fields and shows: percentage input + basis dropdown (default: subtotal). Amount auto-computes. Common cases (Mgmt Fee at 25%, Insurance at 1.2%) pre-fill the percentage.
- **Math pipeline update** — `calcTotals` accounts for calculated lines: sum line subtotal first, apply each calculated line against its declared basis, then O&P (already gated post toggle-fix), then tax, then total.
- **Subscription cost field on FieldServiceProfile** — already exists per audit; surface in Settings stays as-is.

**Change Order Builder.** Existing CO builder creates records (audit confirmed). Phase 1 wiring:

- **Numbering** — auto-increment co_number per project on creation
- **Signing flow** — the CO send-for-signature path needs to mirror FSEstimate's existing flow. Status transitions: draft → awaiting_signature → signed.
- **Math wiring** — when CO transitions to signed status, recompute FSProject.total_budget = original_budget + sum(signed COs). This is a server-function trigger or a recalc on read; choose whichever pattern is simpler given Base44 constraints.

**Log — adds Sub Payment and Client Payment.** Today's Log handles labor, material, photo, daily-log, daily-photo writes. Phase 1 adds two type-picker entries:

- **Sub Payment** — captures payment paid out. Form fields: project (already scoped at top), payee name, amount, method, reference, notes, optional receipt photo. Writes FSPayment with direction=paid, party_type=subcontractor, party_name=payee.
- **Client Payment** — captures payment received from client. Form fields: project (scoped), amount, method, reference, notes, optional photo. Writes FSPayment with direction=received, party_type=client, party_id=project.client_id, party_name=client.name.

The full Log universal-capture surface (all types unified through one adaptive form, quick re-entry, voice-input everywhere) stays post-migration. Phase 1 just adds the two new entry types alongside what's already there.

**Project Detail.** Header banner shows four metrics:
- Contract Total: `$X` (caption: "Original `$Y` + `N` CO" if any signed COs)
- Received: `$A` of `$X` (`p%`)
- Paid Out: `$B` (sum of FSPayment direction=paid for this project)
- Net Cash: `$C` = Received − Paid Out (color-coded)

Below the header, a basic line view lists the contract's line items with their amounts. No three-column ledger yet — that ships post-migration when line linkage is wired. Phase 1 just shows the line items as the contract describes them.

A Payments section lists all FSPayment records for the project, grouped by direction (Received / Paid Out), with a link to Log for new entries.

**Client Portal.** Existing portal renders the original signed estimate. Phase 1 adds: **CO blocks below the original contract**. Each signed CO appears as its own block, color-distinct, showing description + line items + amount + signed date + link to view-signed-PDF. Below all CO blocks, a "Contract total (with approved changes)" roll-up. Payment History section already exists; verify it renders FSPayment records correctly across both directions (and decide whether sub payments should be visible to the client — likely not in Phase 1 fixed-price; that's transparency-mode territory).

### 4.2 Post-migration surface destination

The full destination surface design — library tab, three client portal views (one per billing model), pre-work and unallocated deposit visual treatment, allowance reconciliation prompts at project close, etc. — is held in Section 4 of the V2 spec we drafted. Reproduced here in summary; details live in the original spec body for reference during the post-migration build sequence.

**Library Surface (post-migration):** new tab with Cost Items / Assemblies / Templates sub-tabs. Save-from-estimate buttons in the builder. Drag-to-drop assembly picker.

**Estimate Builder (post-migration enhancements):** "Start from Template", "From Library" pill, save-to-library inline buttons, billing model selector with three options.

**Project Detail (post-migration enhancements):** three-column line ledger (Estimated / Billed / Cost), pre-work state with em-dashes, unallocated deposit banner, allowance-prompts-at-close section.

**Client Portal (post-migration):** three view variants per billing model. Fixed-price hides Cost. Cost-plus shows three columns. T&M shows hours and materials breakdowns.

---

## 5. Math

### 5.1 Phase 1 estimate-time math (fixed-price only)

```
lineSubtotal = sum of line items where category != 'calculated'

calculatedTotal = sum over calculated lines of:
                    line.amount = (basis_value * calculation_pct / 100)
                    where basis_value depends on calculation_basis:
                      'subtotal' → lineSubtotal
                      'subtotal_plus_op' → lineSubtotal + opAmount
                      'labor_only' → sum of labor lines
                      'materials_only' → sum of materials lines

opAmount = lineSubtotal * (overhead_profit_pct / 100)
           if features.overhead_profit_enabled = true; 0 otherwise

taxableBase = lineSubtotal + calculatedTotal + opAmount

taxAmount = taxableBase * (tax_rate / 100)

estimateTotal = taxableBase + taxAmount + (other_amount or 0)
```

Order matters because Management Fee at 25% is computed against the work subtotal, not against itself plus tax. Insurance is similar. The new math handles this.

### 5.2 Phase 1 project-time math

```
originalContract = signed estimate total = FSProject.original_budget (immutable post-signing)

changeOrderAdjustment = sum of FSChangeOrder.amount where status = 'signed' and project_id = this project

contractTotal = originalContract + changeOrderAdjustment
              = FSProject.total_budget

receivedToDate = sum of FSPayment.amount where direction = 'received' and project_id = this project

paidOutToDate = sum of FSPayment.amount where direction = 'paid' and project_id = this project

netCash = receivedToDate - paidOutToDate
        (Phase 1 surfaces this; "running profit" with proper line-level cost rollup is post-migration)

outstanding = contractTotal - receivedToDate
            (in Phase 1 fixed-price, this is "what client still owes overall"; line-level outstanding is post-migration)
```

### 5.3 Post-migration math

Three billing models add their own billing computation. Cost-plus computes line.billed = line.cost * (1 + markup). T&M computes from logged hours and materials. Line-level Estimated/Billed/Cost rollup becomes possible once `line_item_id` linkage is on FSPayment / FSLaborEntry / FSMaterialEntry. The full math is in Section 5 of the V2 spec, held for the post-migration build phase.

---

## 6. Workflows

### 6.1 Phase 1 workflow — Bari builds a fixed-price estimate

Bari opens Estimates → New Estimate. Picks Dr. Holman as client. Adds line items in the existing form: Labor / Subcontractor / Fee categories, types description, qty, unit price. Reaches the Management Fee — picks the new "Calculated" type, picks "Management Fee" preset, percentage pre-fills to 25%, basis to subtotal. Same for Insurance at 1.2%. Reviews math: subtotal, calculated lines, tax, total.

Saves draft. Sends for signature. Status → awaiting_signature. Dr. Holman signs in client portal. Status → signed → accepted. Bari clicks "Create Project from Estimate" → FSProject created with original_budget set immutably to the signed total.

### 6.2 Phase 1 workflow — Bari logs activity

Bari pays Crawford Door $1,200 cash for the garage door deposit. Opens Log → picks "Holman Barn" project (auto-fills client) → picks type "Sub Payment" → enters payee "Crawford Door", amount $1,200, method "cash", optional photo of receipt. Saves. FSPayment created with direction=paid, party_type=subcontractor.

Project Detail's Paid Out total updates immediately. Net Cash recalculates.

When Dr. Holman pays him, same flow: Log → Client Payment → amount, method, reference. FSPayment created with direction=received. Received total updates.

### 6.3 Phase 1 workflow — Bari handles a change order

Bari opens up a wall and finds rot. Adds $4,500 of unforeseen framing and demo work. He goes to the Holman project, opens Change Orders, clicks "New CO". Adds line items, enters the work scope and amount in the existing CO form. CO-001 drafted. Clicks "Send to Client for Signature" → status = awaiting_signature. Dr. Holman opens client portal, sees the CO block beneath the original contract, signs. Status = signed.

FSProject.total_budget recalculates from $122K to $126.5K. Header on Project Detail updates: Contract Total now shows the new amount with "+1 CO" indicator. Client portal shows the original contract block + the signed CO block + new contract total.

### 6.4 Post-migration workflows

Cost-plus estimating, library-powered estimate construction, deposit allocation, allowance reconciliation, three client portal variants, etc. Held for post-migration build sequence. Full workflows in Section 6 of the V2 spec.

---

## 7. Cross-space awareness

Reserved seams that V1 doesn't ship but the data shape anticipates:

- **Business Finance space.** Reads FSPayment (income + expense), FSMaterialEntry, FSLaborEntry. FS is canonical source.
- **Directory.** Reads from FieldServiceProfile (services, branding) and Business (location, category) for the contractor's public presence.
- **Live Local Supplier Pricing.** FSCostItem.supplier_link_id is the seam (post-migration).
- **Open Postings / Bid Responses.** FSEstimate.generation_source enum has a value reserved for `public_posting` (post-migration).
- **Cross-tier payment connection.** FSPayment.recipient_profile_id is reserved (post-migration). When Bari pays a sub who also uses LocalLane, payment mirrors to sub's space as income.

---

## 8. Migrations

### 8.1 Phase 1 migrations

- **FSProject.original_budget backfill.** Every existing FSProject where original_budget is null gets it set from the linked signed estimate's total. One Base44 prompt at deployment. Affects Bari's Holman project.
- **FSPayment.direction default.** New field. All existing records default to `direction = received` (today's payments are all client-received). One Base44 prompt.
- **FSPayment.party_type default.** All existing records default to `party_type = client`. Backfill `party_name` from FSClient.name where party_id (or equivalent) links. One Base44 prompt.
- **FSChangeOrder.co_number backfill.** Existing CO records (Bari has zero per audit) get sequential numbering. Likely no records to touch.
- **Line item shape upgrades on FSEstimate.** Add `is_calculated`, `calculation_basis`, `calculation_pct` defaults to existing line items (false/null/null). Bari's existing line items render identically.

### 8.2 Post-migration migrations

Library entities, allocation states, line linkage on all expense entities, FSAllowanceReconciliation entity creation. Held for post-migration build sequence.

---

## 9. Phasing — what ships when

### Phase 1 — pre-migration (Bari today)

Six concrete items, all on existing Base44 infrastructure:

1. **Calculated line wiring on FSEstimate.** Schema fields already shipped per cleanup commit e57010f. Phase 1 adds the line type picker option, the math pipeline update, the surface render in estimate builder + estimate detail + client portal.

2. **Change order math.** When FSChangeOrder.status transitions to signed, recompute FSProject.total_budget. Plus co_number auto-increment, plus signing flow mirroring FSEstimate's pattern.

3. **FSPayment direction + party fields.** Add direction, party_type, party_name, method, reference, notes via Base44 entity prompt.

4. **Log adds Sub Payment + Client Payment types.** New entries in the Log type picker. Form adapts. Writes FSPayment with appropriate fields.

5. **Project Detail financial header.** Contract Total / Received / Paid Out / Net Cash banner. Plus Payments section grouped by direction.

6. **Client Portal CO blocks.** Render signed FSChangeOrder records as their own blocks below the original contract. Add "Contract total with approved changes" roll-up.

**FSProject.original_budget immutability.** Set once at project creation (when an estimate is accepted), cannot change thereafter. Backfill existing records.

That's Phase 1. Six features plus one immutability constraint. Doesn't introduce new entities. Doesn't change the estimate builder structure beyond adding one line type. Doesn't change Log structure beyond adding two type-picker entries.

### Post-migration — destination architecture

Once on Supabase + Vercel, the full V1 destination ships:

- Three billing models (cost_plus, time_and_materials added alongside fixed_price)
- Library architecture (FSCostItem, FSAssembly, FSEstimateTemplate)
- Full structured line items (cost_item_id, assembly_id, allowance flag)
- FSPayment full shape (line_item_id, allocation_status, parent_payment_id, recipient_profile_id)
- Line linkage on FSLaborEntry / FSMaterialEntry / FSDailyLog
- FSAllowanceReconciliation entity + close-prompts flow
- Three client portal views (one per billing model)
- Project Detail three-column line ledger
- Pre-work and unallocated deposit visual treatment
- Log full universal capture (all entry types, voice everywhere, quick re-entry)
- Library tab in workspace shell

### Deferred V2+ (post-destination)

- Stripe / online payment processing
- Payment schedules / draws automation
- Invoice generation as distinct entity
- Auto-flagged "this looks like a change order" detection
- Lien rights and retainage handling
- Multi-currency
- Auto-export to QuickBooks / accounting platforms

### Deferred to post-Phase-6

- Field Service rename to "Jobs" (SQL ALTER at Supabase migration)

---

## 10. Open questions for review

These need Doron's call before Hyphae starts building Phase 1:

1. **Calculated line basis options for Phase 1.** Spec defines `subtotal`, `subtotal_plus_op`, `labor_only`, `materials_only`. Does Phase 1 ship all four or just `subtotal` (which is what Bari needs for Mgmt Fee + Insurance)? My lean: ship all four, the math is the same shape regardless of which option populates.

2. **Tax convention.** I've assumed `taxableBase = lineSubtotal + calculatedTotal + opAmount` per typical convention. Bari may be in a jurisdiction (Oregon) where labor isn't taxable, or where the convention differs. Worth confirming during dogfood.

3. **CO signing flow gate.** Phase 1 mirrors FSEstimate's send-for-signature path for FSChangeOrder. Does the CO approval require any second-party authentication (e.g. project owner approves before sending) or is it the same as estimate sending? My lean: same as estimate.

4. **Sub payment visibility in client portal.** Phase 1 fixed-price doesn't show paid-out sub payments to the client. This stays internal to Bari. Confirm this is correct — under cost-plus (post-migration) sub payments would be visible since they're the basis for billing.

5. **Net Cash vs Running Profit framing.** Phase 1 surfaces "Net Cash" (Received − Paid Out) on Project Detail. Post-migration with line-level cost tracking, this becomes "Running Profit." The Phase 1 number is honest (cash in minus cash out for this project) but doesn't capture all costs (no labor/material entries). Worth naming it carefully so it doesn't imply more than it shows. My lean: "Net Cash" with caption "(receipts − payouts on this project)".

6. **FSPayment migration default for `party_type`.** Phase 1 backfills existing records as `party_type = client`. Is that universally correct or do any existing FSPayment records reflect non-client parties? If Bari's existing records are all client-received, this is safe.

7. **Phase 1 build sequencing.** Six items can ship as one mega-PR or as smaller chained PRs. My lean: chain them. Each one is testable independently. Order: (1) FSPayment fields → (2) Log type-picker additions → (3) calculated lines → (4) CO math wiring → (5) Project Detail header → (6) Client Portal CO blocks. Each PR is small, dogfood-testable, and rollback-able.

8. **Existing line_items shape compatibility.** Bari's signed estimate (EST-2026-004) has line items in the current loose shape. Adding the calculated line type adds a new option but doesn't break existing items. Confirmed safe by the audit, but worth Hyphae verifying when she touches that code path.

---

## 11. References

- `Spec-Repo/spaces/field-service/audits/CURRENT-STATE-AUDIT.md` (commit e8ce739) — entity inventory and gap analysis
- `Spec-Repo/spaces/field-service/audits/TOGGLE-AUDIT.md` (commit 032ab65) — toggle plumbing inventory
- Toggle fix bundle (commit 905c968) — six toggles render, O&P gate enforced, Management Fee removed from UI
- Membrane fix + schema cleanup (commits d201c10 + e57010f) — DEC-140 RLS pattern, duplicate fields collapsed, status enums aligned including `awaiting_signature` and `signed` on FSEstimate
- `JOBS-MIGRATION-PLAN.md` — deferred but informational; data-model ground truth
- DEC-090 — industry presets (claimed, not wired; this spec is preset-aware)
- DEC-093 — Base44 entity prompts protocol
- DEC-115 — subscription tier migration pattern
- DEC-140 — Authenticated Read + server-side scoping pattern
- `Spec-Repo/platform/SPEC-PROTOCOL.md` — the protocol this spec follows
- `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` — strategic context, philosophy, the "GC to GCs" framing, transparency-as-architecture, the connection-to-source thinking

---

*End of spec. Awaiting Doron's review and Hyphae build sequence for Phase 1.*
