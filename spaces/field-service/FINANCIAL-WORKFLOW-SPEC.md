# FIELD-SERVICE-FINANCIAL-WORKFLOW-SPEC.md

> **Status.** V2 spec, Phase 1 explicit. Drafted 2026-04-29 by Mycelia after a sequence of design conversations that established the architectural primitives, the three-billing-models commitment, the library-as-speed concept, and the layer-aware cost tracking principle. Then narrowed to Phase 1 scope after Doron clarified that the full destination ships post-migration; Phase 1 is what Bari needs now on existing Base44 infrastructure.

> **Scope.** The complete destination architecture for the financial spine of the Field Service workspace, with Phase 1 explicitly scoped as the subset that ships before Supabase + Vercel migration. Three pricing models as first-class citizens (fixed-price, cost-plus, time-and-materials). Signed estimate as contract. Change orders as additive amendments. Payments in/out with line linkage and allocation states. Line-item-level state. Project P&L. Allowance reconciliation. Variance visibility. Plus the structural pieces that make the spine fast: a contractor's library (cost items, assemblies, templates) so estimating gets faster every time it's used. Plus Log as universal capture surface, calculated lines (Management Fee + Insurance), preset-aware defaults, cross-space awareness with future Business Finance space.

> **Companion.** See `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` for the strategic context — why three billing models are first-class, why the library is the speed, why LocalLane is becoming a general contractor to general contractors, and the connection-to-source philosophy that anchors transparency as architecture.

> **Out of scope for this spec entirely.** Industry preset wiring (separate spec — but financial workflow is preset-aware). Business Finance space (separate spec — but financial workflow is cross-space-aware). Live local supplier pricing feed (separate spec — but library shape anticipates it). Open Postings / Bid Responses (separate spec — but FSEstimate shape anticipates it). AI estimate generation (separate spec — but data shape reserved). Stripe / online payment processing. Multi-currency.

> **Deferred to post-Phase-6 migration.** Field Service rename to "Jobs" — SQL ALTER at Supabase migration time.

---

## 1. Why this spec exists

Bari Swartz is the only live Field Service user. He has a $121,657 signed estimate with Dr. Holman (EST-2026-004) and a project tracking it. Everything else — the change orders he's already done, the payments he's already received from Dr. Holman, the payments he's made to subs and vendors, the running profit math — lives in an Excel spreadsheet because the platform doesn't support it yet.

Bari is also a *type*, not the audience. The same financial spine — signed estimate as contract, amendments, payments in, payments out, project profit — applies to any service business doing multi-stage work with multi-party money flow. A wedding photographer tracking retainer/shooting/album payments. A structural engineer billing in stages. A piano teacher selling lesson packages. A landscape designer doing a six-month installation.

This spec defines that financial spine **once**, generically, then leans on the industry preset system (separate spec) to adapt the visible shape per business type. Bari is the validation user, not the design target.

The deeper purpose: **Field Service is being built as the contractor's complete operating system**, not just a project tracker. Branches from this hub — Directory presence, Business Finance, Live Local Supplier Pricing, Open Postings / Bid Responses — are real spaces and sub-systems in their own right. Field Service is the operating room; those are the connective tissue around it. The financial workflow spec is the foundation everything else builds on.

For the strategic depth — why this matters beyond the technical mechanics, what LocalLane is becoming through this work — see the companion intent doc in private.

---

## 2. Architectural primitives

Seven primitives anchor this spec. Each was settled in design conversation; each is restated here for the build phase to reference.

### 2.1 The estimate is the project spine

When a client signs an estimate, the estimate becomes the contract for the project. Every subsequent piece of financial data — labor logged, materials purchased, payments received, payments made out, change orders, allowance reconciliations — references the estimate, its line items, or its amendments. **There is no loose financial activity in Field Service.** Project-less expenses belong in the future Business Finance space.

The estimate is `FSEstimate`. The signed amount is the **Original Contract Amount** and is immutable from the moment of signing forward.

### 2.2 Three billing models, all first-class

LocalLane's transparency philosophy and many contractors' actual practice both demand that cost-plus and time-and-materials be first-class citizens, not edge cases. The destination spec is **billing-model-agnostic at the core** with each model's specifics described explicitly.

`FSProject.billing_model` is a project-level enum (post-migration):

- **`fixed_price`** — Client pays the agreed contract amount regardless of actual costs. Contractor's margin is hidden. Variance lives entirely on the contractor's side. Estimate is the bill.
- **`cost_plus`** — Client pays actual costs plus an agreed markup (typically a percentage). Total revenue depends on actual costs. Client sees costs because they're the basis for the bill. Actuals are the bill.
- **`time_and_materials`** — Client pays an hourly rate plus material reimbursement at agreed rates. Bills computed from logged hours and materials. Logs and receipts are the bill.

Most contractors use one model for most jobs but exceptions matter. Bari might do cost-plus for a $150K barn and fixed-price for a $5K deck repair. The model is set at project creation (defaulting from estimate-time selection) and is generally not changed mid-project.

Each model has a distinct estimate emphasis, billing flow, and client portal view (Section 4). The data model underneath is the same; what changes is what's visible and how bills are computed.

**Phase 1 only ships fixed-price.** Bari uses fixed-price-with-management-fee exclusively. Cost-plus and T&M ship post-migration when the data model can fully support them and when we have users who need them. Phase 1 doesn't even include the field — projects are implicitly fixed-price. The field gets added at the start of the post-migration build.

### 2.3 Change orders are additive amendments, not replacements

When work scope changes mid-project — discovered conditions, client requests, deductive scope reductions — the original signed contract is not modified or replaced. A separately-numbered, separately-signed amendment is created (`FSChangeOrder`). The signed contract continues to exist as the legal record of what the client agreed to. The change order extends or reduces the contract by its own dollar amount.

**Contract Total = Original Contract Amount + sum of approved Change Orders.**

This matches industry practice across every credible construction-management platform researched (JobTread, Buildertrend, Houzz Pro, Procore). Replacement-style change orders destroy legal evidence and create dispute exposure; the additive pattern preserves the audit trail.

### 2.4 Line items carry three numbers

Every line on an estimate carries:
- **Estimated** — what the client agreed to pay for this line item (or what the contractor expects to spend, in cost-plus)
- **Billed** — what's been invoiced or allocated against this line (cumulative)
- **Cost** — what the contractor has spent against this line (sum of expense logs attached)

Variance is the difference. The system makes variance *visible*; the contractor chooses the resolution path (change order, eat the margin, allowance reconciliation, refund). The platform doesn't auto-resolve variance — that's a contractor judgment call rooted in conversations with the client.

**Layer-aware tracking.** Cost granularity matches the contractor's layer in the supply chain. A general contractor like Bari who subs everything tracks line cost as the sum of his sub payments — he doesn't break that $5K sub payment into "$2.8K lumber, $2.2K labor" because that's the sub's job. A sub doing the actual work tracks materials and labor separately because they're touching real costs. A supplier tracks neither. **Phase 1 honors Bari's layer**: line cost in his world is the sum of FSPayment records (direction=paid) tied to that line. FSMaterialEntry and FSLaborEntry exist but are optional in Phase 1; they become more central post-migration when subs start using LocalLane themselves.

In cost-plus and T&M (post-migration), the client sees all three columns. In fixed-price (Phase 1), the client only sees Estimated and Billed; Cost is the contractor's internal view.

### 2.5 The library is the speed (post-migration)

Estimating gets faster every time it's used because the contractor's personal library grows. Three layers of reusable building blocks accumulate:

- **Cost Items** — saved single line entries ("Re-roof tear-off, 30-year shingle, $4.50/sq ft")
- **Assemblies** — multi-item bundles for common scopes ("Cedar deck — 200 sq ft" includes lumber + hardware + framing labor + decking labor + cleanup; scales with dimensions)
- **Estimate Templates** — full project skeletons ("Pole Barn Template" with sections pre-loaded)

The first estimate Bari builds takes longer than typing in Excel. The fiftieth takes a fraction of the time. By the hundredth, bidding becomes "fill in dimensions, tweak the obvious things, send." This is the unlock that makes Open Postings (future) viable — contractors with libraries can respond to public bids in minutes, not hours.

The library is also the seam for live local supplier pricing (future): each cost item references a material, and when the supplier feed ships, prices update from real Eugene-area suppliers automatically. Today: typed price. Tomorrow: live price. Same data shape.

The library is also the seam for AI generation (future): when AI estimate creation ships, the AI generates into the same Cost Item / Assembly structure. Manual entry, library pickers, and AI all converge on one shape.

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

**FSEstimate** — line item shape gets calculated-line flags. All default to false/null; existing line items render identically.
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

Once on Supabase + Vercel infrastructure, the destination architecture lands. Hyphae implements Phase 1 only and leaves the destination work for the post-migration build sequence.

#### 3.2.1 New entity: FSCostItem (post-migration)

The contractor's reusable line-item library. Single saved entries that get pulled into estimates by name.

Fields:
- `id`, `profile_id` (FK to FieldServiceProfile)
- `name`: string — e.g. "Re-roof tear-off, 30-year shingle"
- `description`: text, nullable
- `category`: enum — `materials | labor | subcontractor | fee | other`
- `default_unit`: enum, nullable — `sq_ft | linear_ft | each | hours | day | lump`
- `default_rate`: number — canonical price per unit
- `default_qty`: number, nullable — for assemblies that include this item with a fixed qty
- `cost_basis`: number, nullable — optional contractor's actual cost (for cost-plus calculations)
- `trade_category`: string, nullable — for Xactimate grouping
- `tags`: array of strings — free-form tags for discovery
- `archived_at`: datetime, nullable
- `supplier_link_id`: string, nullable — V1 reserved, V2 references live supplier feed
- `ai_generated`: boolean — V1 reserved, V2 marks AI-created items
- `created_at`, `updated_at`, `created_by`

**Permissions:** Read = Authenticated Users (DEC-140 pattern), with server-side scoping on profile_id. Create/Update/Delete = Creator Only.

**Save flow:** When a user types a line item in the estimate builder, a "Save to library" button appears. One click saves it as an FSCostItem. The library accumulates as work happens, lowest possible friction.

#### 3.2.2 New entity: FSAssembly (post-migration)

Multi-item bundles that drop into estimates as a unit and expand into individual line items.

Fields:
- `id`, `profile_id` (FK)
- `name`: string — e.g. "Cedar deck — per sq ft"
- `description`: text, nullable
- `scaling_unit`: enum, nullable — `sq_ft | linear_ft | each | hours | null` (null = unscaled)
- `default_dimension`: number, nullable — suggested input value
- `cost_items`: array of references, each containing:
  - `cost_item_id`: string (FK to FSCostItem)
  - `qty_per_unit`: number — qty of this item per scaling_unit
  - `qty_fixed`: number, nullable — alternative to per-unit, for non-scaling items
  - `notes`: string, nullable
- `trade_category`: string, nullable
- `tags`: array of strings
- `archived_at`: datetime, nullable
- `ai_generated`: boolean
- `created_at`, `updated_at`, `created_by`

**Permissions:** same pattern as FSCostItem.

**Drop-into-estimate flow:** User picks an assembly, enters the dimension (e.g. "200 sq ft"), the assembly expands into individual line items in the estimate, each scaled appropriately. The user can then tweak any line. The estimate optionally remembers which assembly the lines came from (display-only) so the contractor and client can see them grouped under the assembly name.

#### 3.2.3 New entity: FSEstimateTemplate (post-migration)

Full project skeletons. Pre-built estimate structures with assemblies and cost items pre-loaded.

Fields:
- `id`, `profile_id` (FK)
- `name`: string — e.g. "Pole Barn — Standard"
- `description`: text, nullable
- `project_type`: string, nullable — for filtering by project archetype
- `default_billing_model`: enum — `fixed_price | cost_plus | time_and_materials`
- `sections`: array of section objects, each containing:
  - `name`: string — e.g. "Foundation"
  - `line_items`: array — same shape as `FSEstimate.line_items` (Section 3.2.4), pre-populated with sensible defaults
- `default_management_fee_pct`: number, nullable
- `default_overhead_profit_pct`: number, nullable
- `default_tax_rate`: number, nullable
- `default_terms`: text, nullable
- `archived_at`: datetime, nullable
- `ai_generated`: boolean
- `created_at`, `updated_at`, `created_by`

**Permissions:** same pattern.

**Use flow:** User picks "New Estimate from Template", picks the template, picks the client, the estimate is created with the template's structure. User fills in dimensions, tweaks lines, sends.

#### 3.2.4 FSEstimate destination shape (post-migration)

Line items become fully structured. Each line carries:
- `id`: string — stable id for cross-referencing from Log
- `description`: string
- `category`: enum — `materials | labor | subcontractor | fee | other | calculated | allowance`
- `qty`: number, nullable — null for lump-sum lines
- `unit`: string, nullable — `sq_ft | linear_ft | each | hours | day | etc.`
- `rate`: number, nullable — null for lump-sum lines
- `amount`: number — canonical estimated amount for this line
- `cost_item_id`: string, nullable — FK to FSCostItem if pulled from library
- `assembly_id`: string, nullable — FK to FSAssembly if line came from an assembly
- `assembly_group_id`: string, nullable — when multiple lines came from the same assembly drop, share an id for display grouping
- `is_allowance`: boolean — true = placeholder amount, reconciled at close
- `is_calculated`: boolean — true = % of subtotal (Phase 1: already exists)
- `calculation_basis`: enum, nullable — `subtotal | subtotal_plus_op | labor_only | materials_only | null` (Phase 1: already exists)
- `calculation_pct`: number, nullable (Phase 1: already exists)
- `trade_category`: string, nullable — for Xactimate grouping
- `cost_basis`: number, nullable — for cost-plus / T&M; contractor's actual or expected cost
- `notes`: string, nullable

Top-level FSEstimate additions (post-migration):
- `billing_model`: enum — `fixed_price | cost_plus | time_and_materials` (default: `fixed_price`)
- `template_id`: string, nullable — if created from a template
- `generation_source`: enum — `manual | library | template | ai`
- `show_costs_to_client`: boolean — derived from billing_model but overridable

**Lump-sum vs detailed mixing:** A line can be either qty × rate OR a single lump amount. The shape supports both: when qty/rate are populated, amount = qty × rate; when both are null, amount stands alone. The contractor can mix freely on the same estimate — detailed lines for things they know, lump sums for things they don't, allowances for things TBD.

#### 3.2.5 FSPayment destination shape (post-migration)

Phase 1 fields plus:
- `line_item_id`: string, nullable — payment can be allocated to a specific line OR float at project level
- `change_order_id`: string, nullable — payment can be against a specific CO
- `allocation_status`: enum — `unallocated | allocated | partially_allocated`
- `parent_payment_id`: string, nullable — when a payment is split via allocation, child records reference parent
- `recipient_profile_id`: string, nullable — reserved; when filled, mirrors payment to recipient's FieldServiceProfile (cross-tier connection)

**Allocation states explained.** When a deposit comes in at signing, the client hasn't paid for any specific work yet — the money is held against the contract as a whole. The payment is recorded with `line_item_id = null` and `allocation_status = unallocated`. As work completes, the contractor can allocate portions of the deposit to specific lines. Allocation either:

- Marks the entire payment as allocated to one line (single-line scenarios)
- **Splits the payment** into multiple child records, each tied to a line, with the parent marked `partially_allocated` and showing the unallocated remainder

The "Received" total at the project level always reflects the full sum regardless of allocation state — money is money. Allocation just determines how it shows up in line-level rollups.

**Treatment at project close.** When a project moves to `current_status = complete`, any remaining unallocated deposit is auto-allocated to the project as "general contract value" — the leftover counts toward the contract total revenue without specific line attachment.

#### 3.2.6 FSDailyLog / FSLaborEntry / FSMaterialEntry — line linkage (post-migration)

Add to each:
- `line_item_id`: string, nullable — FK to a specific line on the project's estimate. When populated, the entry's cost flows into that line's Cost rollup. When null, cost flows into a project-level "unallocated cost" bucket.
- `cost_item_id`: string, nullable — FK to FSCostItem (FSMaterialEntry primarily)
- `notes` text field if not already present

When Bari logs labor, he picks the line "Roofing" if it applies; the four hours at $45/hr = $180 flows into the Roofing line's Cost. If he logs labor not tied to a specific line (general site cleanup, supervisory hours), `line_item_id` stays null and the cost flows into a project-level "unallocated cost" bucket visible in the P&L.

#### 3.2.7 New entity: FSAllowanceReconciliation (post-migration)

When an allowance line's actual cost differs from the allowance amount, the variance gets reconciled.

Fields:
- `id`, `project_id`, `estimate_id`, `line_item_id`
- `allowance_amount`: number — original placeholder
- `actual_amount`: number — final cost
- `variance`: number — actual minus allowance
- `resolution`: enum — `client_pays_difference | contractor_absorbs | deductive_co | credit_to_client`
- `reconciled_at`: datetime
- `reconciled_by`: user_id
- `notes`: text
- `generated_co_id`: string, nullable — if resolution generated a CO, link to it
- `generated_payment_id`: string, nullable — if resolution generated a credit payment

When a project moves to `current_status = complete`, the system surfaces unreconciled allowance lines and prompts for resolution. Each resolution either generates a CO (client_pays_difference, deductive_co), a credit FSPayment (credit_to_client), or just logs the absorption (contractor_absorbs).

#### 3.2.8 FSProject destination additions (post-migration)

Phase 1 fields plus:
- `billing_model`: enum — `fixed_price | cost_plus | time_and_materials` (defaults from source estimate)
- `current_status`: enum — `active | on_hold | complete | cancelled`
- `estimate_id`: string — FK to source FSEstimate
- `show_costs_to_client`: boolean — inherits from estimate but overridable per project

Helper view (computed, not stored): `contract_total` is an alias for `total_budget`.

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
        │     └── FSProject (original_budget locked at acceptance [PHASE 1]; billing_model inherited post-migration)
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

**Change Order Builder.** Existing CO builder creates records (audit confirmed). Phase 1 wiring:

- **Numbering** — auto-increment co_number per project on creation
- **Signing flow** — the CO send-for-signature path needs to mirror FSEstimate's existing flow. Status transitions: draft → awaiting_signature → signed.
- **Math wiring** — when CO transitions to signed status, recompute FSProject.total_budget = original_budget + sum(signed COs). Server-function trigger or recalc on read; choose whichever pattern is simpler given Base44 constraints.

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

### 4.2 Estimate Builder — destination (post-migration)

Existing builder gets these additions on top of what Phase 1 already shipped:

**Top-level controls:**
- **Billing model selector** (default: fixed-price). Choosing changes the form's emphasis and the client portal preview.
- **"Start from Template" button** opens a picker showing the user's templates, filtered by project type if selected.
- **"From Library" pill** next to "Add Line" — opens an assembly/cost-item picker as a slide-in panel.

**Line item picker enriched (beyond Phase 1's calculated type).** Adds:
- `allowance` category — when picked, the line gets a visual marker (italicized description, an "Allowance" tag); same input shape as a regular line but flagged. Description should default to a placeholder like "Allowance — actual TBD."

**Library integration in line item:**
- **"Save to library" inline button** appears on each line as the user types. One click saves the line as an FSCostItem.
- **"Pick from library" search** in line description — typing matches existing FSCostItem records and offers them as autocomplete. Picking one pre-fills the line.
- **Assembly drop zone** — dragging an assembly from the library panel onto the estimate inserts the assembly's expanded line items as a group.

**"Convert to Project" button on accepted estimates.** When estimate status moves to `accepted`, a button appears: "Create Project from this Estimate." Clicking creates an `FSProject` with `original_budget = estimate.total`, `billing_model = estimate.billing_model`, `current_status = active`, the estimate's line items copied as the project's reference shape.

### 4.3 Library Surface (new tab in workspace shell, post-migration)

Settings → Library, or a dedicated Library tab depending on UX direction. Three sub-tabs:

- **Cost Items** — table of all saved single-line items. Columns: name, category, default unit, default rate, last used, tag chips. Inline edit. Bulk archive.
- **Assemblies** — list of multi-item bundles. Click expands to show component cost items. Drag-to-reorder. Edit assembly composition.
- **Templates** — list of full project skeletons. Click opens a preview.

**Library accumulation.** No upfront task to "build the library." It accumulates from the save buttons in the estimate builder. The library tab shows what's been collected over time.

**Industry-preset seed.** When a user adds Field Service to a business, the system optionally seeds a starter library based on the inferred industry preset. For `general_contractor`: maybe 10 cost items + 3 common assemblies. For `service_provider`: maybe 5 cost items + 1 common assembly. Seed quality matters less than seed simplicity — these are starting points, not finished libraries.

### 4.4 Change Order Builder — destination (post-migration)

Beyond Phase 1's signing flow + math wiring:

**Library-powered.** CO line items use the same library as estimates. "Discovered framing rot" CO can drop in the same "Demo of damaged framing" cost item, the same lumber items, the same labor lines.

**CO line items.** Same shape as FSEstimate's. Each line is a discrete work scope being added or removed. Net of the CO's lines = the CO's `amount`. Negative lines (deductive scope) are allowed; the math handles them naturally.

### 4.5 Log — universal capture (destination, post-migration)

Replaces today's Log tab. New shape:

**Top of form: scope pickers.** Always visible.
- **Project** (required) — dropdown of active projects for this profile
- **Client** (auto-fills from project's client; editable if needed)

**Type picker.** Required. Dropdown of:
- `Labor` — who, hours, rate, line item (optional), notes, photo (optional)
- `Expense / Material` — what, vendor, amount, line item (optional), receipt photo (optional), library cost item link (optional), notes
- `Sub Payment` — who got paid, amount, line item (optional), check/method, reference, notes
- `Client Payment` — amount received, method, reference, line item (optional) or "general progress" or "deposit", notes
- `Photo` — just a photo + caption, line item (optional)
- `Note` — text-only, milestone toggle, line item (optional)
- `Permit Update` — permit number, status, inspection date, notes (creates/updates FSPermit)

**Form adapts.** When type picker changes, the form below changes to fit the type's fields. Project and Client stay locked at the top.

**Photo support universal.** Every entry type supports an optional photo upload. Photos are stored on `FSDailyPhoto` and linked back to the originating entry.

**Quick re-entry.** After saving an entry, a "Log another for this project" button stays focused on the same project + client + type, just clears the type-specific fields. Bari logging six expenses in five minutes feels possible.

**Voice input (existing).** Stays. Voice input already works on text fields per existing audit.

**Routing under the hood.** Log is one component but writes to the existing entities: `FSLaborEntry` for Labor, `FSMaterialEntry` for Expense, `FSPayment` (direction-tagged) for Sub Payment / Client Payment, `FSDailyLog` + `FSDailyPhoto` for Photo and Note, `FSPermit` for Permit Update. The user sees one surface; the data lands in the right table.

### 4.6 Project Detail — destination (post-migration)

Beyond Phase 1's header and basic line view:

**Header banner enhancements.** The four-metric banner becomes:
- **Contract Total:** `$X` (Original `$Y` + COs `$Z`)
- **Received:** `$A` of `$X` (`p%`)
- **Cost:** `$B` (`q%` of contract) — replaces "Paid Out" with full line-level cost rollup including FSLaborEntry / FSMaterialEntry not just FSPayment
- **Running Profit:** `$C` (`Received - Cost`, color-coded) — replaces "Net Cash" once line-level cost is real

**Pre-work state.** When a project has `original_budget` set but no logged labor/materials, the header shows status "Awaiting kickoff" instead of "Active". Cost shows `$0`, Running Profit shows `—` with caption "Begins when work starts." Em-dashes signal "nothing's happened yet" rather than "we measured zero."

**Unallocated deposit banner.** When the project has unallocated FSPayment records, a prominent banner appears: "$X deposit held — ready to allocate" with an "Allocate" button. The banner stays visible until allocation is complete or until the project closes (at which point the leftover auto-allocates to general contract value).

**Three-column line ledger.** A new section showing every line on the contract (estimate + signed COs combined) with three columns: **Estimated**, **Billed**, **Cost**. Variance shown on the right. Click a line to drill in and see all logs/payments tied to it.

**Original contract block + CO blocks.** The line ledger is visually split: the original contract block shows the signed estimate's lines and totals; each signed CO appears as its own block below, color-distinct (info-blue), with its own header and totals. A final "Contract total (with COs)" roll-up sits at the bottom.

**Payments section moves to Log.** The standalone Payments section in Project Detail is removed. Payments now flow through Log; Project Detail has a *view* of all payments for this project (the Payment History section) but no input form.

**Allowance prompts.** When project status moves to `complete`, an inline section appears: "X allowance lines unreconciled. Settle each one to close the books on this project." Each unreconciled line gets a quick-action chooser (Client Pays Difference / Contractor Absorbs / Generate Deductive CO / Credit to Client) that creates the FSAllowanceReconciliation record.

### 4.7 Settings — Workspace Features (post-toggle-fix, post-preset-wiring future)

Today (post-fix bundle 905c968): six toggles. Permits, Subs, O&P, Xactimate, Payments, Timeline.

Future (with industry preset wired, eventual separate spec): Settings becomes layered:

- **Industry Preset** at the top (read from Business.category/archetype, confirmable by user). Drives the visible toggle set.
- **Advanced Settings** below — collapsible section showing the in-type toggles relevant to this preset. Most users never expand it. The ones who do are making genuine choices (this GC doesn't do insurance work; that GC does charge management fees).

This spec doesn't ship the preset wiring — that's separate. But the financial workflow surfaces (estimate builder, log, project detail, library) are written to read both the preset and the toggles, so when preset wiring lands, no refactor is needed.

### 4.8 Client Portal — three views, one per billing model (post-migration)

Phase 1 ships only the fixed-price view (with CO blocks added). Post-migration adds two more variants based on `FSEstimate.billing_model`:

**Fixed-price view (Phase 1 default):**
- Summary metrics: Contract Total, You've paid, Remaining balance
- Contract block: line items with their estimated amounts; subtotal/calculated lines/tax/total. **No Cost column.**
- CO blocks: same shape as contract block, with description and approved date
- Payment History: dates, descriptions, methods, amounts; deposits show "Held — not yet allocated" badge when applicable (post-migration)
- Signed PDF links for original contract and each CO

**Cost-plus view (transparency model, post-migration):**
- Summary metrics: Contract Total (so far), Costs to date, Markup (Y%), You've paid, Balance
- Contract block: line items with **three columns visible — Estimated, Costs to date, Billed**. Markup line clearly broken out. Math shows: subtotal of costs → markup → total billed.
- Each line shows the contractor's actual cost as it accumulates, with a "What's this?" link that explains the cost-plus model.
- CO blocks: same three-column treatment.
- Payment History: same as fixed-price, plus a clear "Costs and markup are recalculated as work progresses" note.

**Time-and-materials view (post-migration):**
- Summary metrics: Hours logged, Materials sourced, Total billed, You've paid, Balance
- Hours block: list of labor entries (date, description, hours, rate, total) — possibly grouped by day or week
- Materials block: list of materials (date, description, qty, rate, total)
- Markup if applicable
- Payment History: same shape

**Across all three:** the client portal shows exactly the same financial reality the contractor sees in their respective views. No information asymmetry. This is what makes signing trustworthy — the client sees what they're agreeing to, and as work progresses, what they're being billed for.

### 4.9 Cross-space surface seams (V1 reads, write hooks for future)

The financial entities are designed so future spaces read them cleanly. V1 doesn't ship any of the *consuming* surfaces, but the data shape is set up so they're additive when their specs land:

- **Business Finance space** reads FSPayment (income + expense flows), FSMaterialEntry, FSLaborEntry. No duplication; FS is the canonical source.
- **Directory** reads from FieldServiceProfile (services, branding) and Business (location, category). The estimate builder's "Send to client" surface uses Directory-level branding.
- **Live Local Supplier Pricing** reads from FSCostItem.supplier_link_id (V1: field exists, no consumer; V2: feed connects).
- **Open Postings / Bid Responses** uses FSEstimate as the bid format (V1: field `generation_source` reserved; V2: public posting flow creates draft FSEstimate records that contractors complete and submit).

---

## 5. Math

This section defines the canonical calculations to prevent drift between client portal renders, in-app estimate views, project detail dashboards, and any future reporting surfaces.

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

### 5.3 Post-migration project-time math (with COs and live data)

```
originalContract = signed estimate total (FSProject.original_budget)

changeOrderAdjustment = sum of FSChangeOrder.amount where status = 'signed' and project_id = this project

contractTotal = originalContract + changeOrderAdjustment
              = FSProject.total_budget

billedToDate = depends on billing model:
  fixed_price: derived from line.billed allocations (V2 may add explicit invoicing)
  cost_plus: sum of line.cost values * (1 + markup_pct / 100), plus calculated lines, plus tax
  time_and_materials: sum of (FSLaborEntry hours * rate) + (FSMaterialEntry costs * markup)

receivedToDate = sum of FSPayment.amount where direction = 'received' and project_id = this project

costToDate = sum of FSLaborEntry costs + FSMaterialEntry costs + FSPayment.amount where direction = 'paid'
             all scoped to this project

outstanding = billedToDate - receivedToDate
            (what the client still owes)

remainingToBill = contractTotal - billedToDate
                (what we can still invoice for; in fixed-price)
                (in cost-plus / T&M, this is a moving target)

runningProfit = receivedToDate - costToDate
              (color-coded on dashboard)

projectedFinalProfit = contractTotal - costToDate (assuming we'll be paid the full contract)
```

### 5.4 Line-level math (post-migration)

For each line item on the estimate (and across signed COs):

```
line.estimated = line.amount

line.billed = depends on billing model:
  fixed_price: sum of payments allocated to this line where direction = 'received'
  cost_plus: line.cost * (1 + markup_pct / 100) — actuals drive billing
  time_and_materials: sum of logged hours * rate + materials sourced for this line + markup

line.cost = sum of FSLaborEntry/FSMaterialEntry/FSPayment(direction=paid)
            where line_item_id = this line

line.variance = line.estimated - line.cost
              (positive = under budget; negative = over budget on this line)
```

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

### 6.4 Post-migration workflow — Bari builds a library-powered estimate

Bari has been using Field Service for 6 months. His library now includes a "Pole Barn" template, plus standalone assemblies for "Standard roof framing — per sq ft", "Concrete slab — per sq ft", "Electrical service to outbuilding", and ~40 cost items.

He opens Estimates → New Estimate from Template → "Pole Barn — Standard". Picks the new client. The template populates: 8 sections, ~30 line items pre-filled with Bari's standard rates. He enters dimensions ("30 × 50 = 1500 sq ft"), the scaling assemblies recalculate. He tweaks specific items: roof type to "metal" (drops in metal-roof assembly via library picker), adds two roll-up doors line, removes the foundation wall section since the slab template applies.

He sets billing model: fixed-price. Adds Management Fee as calculated line (25% of subtotal — pulled from his template defaults). Adds Insurance as calculated line (1.2% of subtotal+OP). Reviews the math: subtotal $89,250, calculated $23,652, OP $0, tax $9,615 → total $122,517. Saves draft.

Total time elapsed: ~12 minutes. Without the library this would have taken 90 minutes.

### 6.5 Post-migration workflow — cost-plus estimate

Same library, different model. New client is Sarah, building a custom porch — she wants the work done but doesn't want a fixed price; she wants to pay actuals plus 25%. Bari opens New Estimate, picks Sarah, sets billing_model = `cost_plus`, sets markup = 25%.

He builds the estimate using the same library — porch framing assembly, decking material item, railing line, etc. — but the math is different: each line shows his expected cost, and the total reflects cost + 25%. Sarah signs the cost-plus contract. Throughout the project, every receipt Bari logs flows into the line cost; the bills Sarah receives are computed from actuals + markup.

When she signs, the FSEstimate carries `billing_model = cost_plus` and `show_costs_to_client = true`. The client portal she sees is the cost-plus view with three columns visible.

### 6.6 Post-migration workflow — Bari closes a project with allowances

Holman barn approaches completion. The Lighting line was an allowance for $3,500. The actual final cost was $4,200. Bari moves the project to `current_status = complete`. The allowance prompt appears: "Lighting — Estimated $3,500, Actual $4,200, Variance +$700. How do you want to resolve?" Bari picks "Client Pays Difference" → a CO-002 is drafted for $700, ready for the client to sign. Once signed, contract_total bumps by $700. Allowance is reconciled, FSAllowanceReconciliation record created, generated_co_id linked.

If Bari had picked "Contractor Absorbs," no CO would generate; the variance would log as eaten margin in the FSAllowanceReconciliation record.

### 6.7 Post-migration workflow — Bari handles an unallocated deposit through to allocation

Dr. Holman pays a $46K deposit on signing. Bari logs it via Log → Client Payment → amount $46,000, method "Check #4421", line item field set to "General Progress" (which translates to `line_item_id = null`, `allocation_status = unallocated`). Project Detail shows "$46,000 deposit held — ready to allocate" banner.

Two weeks later, Bari finishes the roofing line. He opens the deposit record (or clicks the Allocate button on the banner), allocates $10K to the Roofing line. Behind the scenes: original $46K record marked `partially_allocated`, child record created for $10K with `line_item_id = roofing_line_id`, `parent_payment_id` linked.

Project Detail's line ledger now shows Roofing with $10K Billed; the deposit banner updates to "$36,000 remaining — ready to allocate".

### 6.8 Post-migration workflow — Doron sets up Consulting with Doron (preset-aware onboarding)

Doron has a Business "Consulting with Doron" with `category = "services"`, `archetype = "service_provider"`. He adds Field Service space. The wizard reads the Business and proposes preset = `service_provider`. Doron confirms.

Field Service workspace initializes:
- Default toggle set: no Permits, no Subs, no Xactimate, no O&P, yes Payments, no Timeline
- Starter library (V1 light): a few generic service-provider cost items, no contractor-specific assemblies
- Estimate builder shows a clean form with line items, optional Management Fee (he toggles it on in Advanced Settings — yes, he charges it), no insurance line, simple total

When industry preset is fully wired (separate spec), this entire flow happens automatically. For now, the preset is read but the toggle defaults are applied uniformly; user can adjust in Advanced Settings.

---

## 7. Cross-space awareness

The financial entities are designed so future spaces can read them as canonical:

- **Income flow.** Every `FSPayment` with `direction = received` is income to the business. Business Finance's eventual reconciliation feed reads from FSPayment without modification — the existing entity is sufficient.
- **Expense flow.** Every `FSPayment` with `direction = paid` plus every `FSMaterialEntry` plus every `FSLaborEntry` is a project-attached expense. Business Finance can roll these up by month, by project, by category.
- **Project-less expenses.** Out of scope for Field Service. Business Finance is the home for those when it ships.
- **Tax and accounting handoff.** The `reference` field on FSPayment + the timestamps + the project linkage gives Business Finance enough to drive an export-to-accounting flow (QuickBooks, etc.) at year-end.
- **Directory.** The estimate builder's "Send to client" UI uses the Business record's branding. When the Directory space ships, that branding becomes the canonical source for how the contractor presents themselves; the estimate inherits.
- **Live Local Supplier Pricing.** FSCostItem.supplier_link_id is reserved; when the supplier feed ships, items reference real suppliers and prices update from the feed. Estimates built today using typed prices will not auto-migrate; new estimates can use feed-linked items.
- **Open Postings / Bid Responses.** FSEstimate.generation_source enum has `manual | library | template | ai` reserved. When public bid posting ships, a new value `public_posting` can be added; estimates created from public postings carry that source for analytics and routing purposes.
- **Cross-tier payment connection.** FSPayment.recipient_profile_id is reserved. When Bari pays a sub who also uses LocalLane, payment can mirror to sub's space as income. The platform's value compounds across the contractor tier graph as more layers adopt it.

This means Field Service is not building "a payments tracker that has to be migrated to Finance later." It's building a payments tracker that *is* the canonical source for project-related transactions; Finance reads from it.

---

## 8. Migrations

### 8.1 Phase 1 migrations

- **FSProject.original_budget backfill.** Every existing FSProject where original_budget is null gets it set from the linked signed estimate's total. One Base44 prompt at deployment. Affects Bari's Holman project.
- **FSPayment.direction default.** New field. All existing records default to `direction = received` (today's payments are all client-received). One Base44 prompt.
- **FSPayment.party_type default.** All existing records default to `party_type = client`. Backfill `party_name` from FSClient.name where the link can be resolved. One Base44 prompt.
- **FSChangeOrder.co_number backfill.** Existing CO records (Bari has zero per audit) get sequential numbering. Likely no records to touch.
- **Line item shape upgrades on FSEstimate.** Add `is_calculated`, `calculation_basis`, `calculation_pct` defaults to existing line items (false/null/null). Bari's existing line items render identically.

### 8.2 Post-migration migrations

- **FSProject.billing_model default.** New field. Default `fixed_price` for all existing projects (matches today's behavior).
- **FSEstimate.billing_model + show_costs_to_client default.** New fields. Default `billing_model = fixed_price`, `show_costs_to_client = false` for all existing estimates.
- **FSPayment.allocation_status default.** New field. For existing records with project_id but no line_item_id, default `allocated` (treats them as "allocated to the project") to avoid noise from "deposit held" banners on closed projects. New records that come in unallocated will be tagged correctly.
- **Line item shape upgrades on FSEstimate.** Add the rest of the destination fields (`is_allowance`, `cost_item_id`, `assembly_id`, etc.) — defaults `false / null`. Bari's existing line items remain valid.
- **FSDailyLog / FSLaborEntry / FSMaterialEntry line linkage.** Add `line_item_id` and `cost_item_id` fields, defaults null.
- **New entities.** FSCostItem, FSAssembly, FSEstimateTemplate, FSAllowanceReconciliation. Empty tables. Records appear as users use the system.

---

## 9. Phasing — what ships when

### Phase 1 — pre-migration (Bari today)

Six concrete items, all on existing Base44 infrastructure:

1. **Calculated line wiring on FSEstimate.** Phase 1 adds the line type picker option, the math pipeline update, the surface render in estimate builder + estimate detail + client portal.

2. **Change order math.** When FSChangeOrder.status transitions to signed, recompute FSProject.total_budget. Plus co_number auto-increment, plus signing flow mirroring FSEstimate's pattern.

3. **FSPayment direction + party fields.** Add direction, party_type, party_name, method, reference, notes via Base44 entity prompt.

4. **Log adds Sub Payment + Client Payment types.** New entries in the Log type picker. Form adapts. Writes FSPayment with appropriate fields.

5. **Project Detail financial header.** Contract Total / Received / Paid Out / Net Cash banner. Plus Payments section grouped by direction.

6. **Client Portal CO blocks.** Render signed FSChangeOrder records as their own blocks below the original contract. Add "Contract total with approved changes" roll-up.

**FSProject.original_budget immutability.** Set once at project creation (when an estimate is accepted), cannot change thereafter. Backfill existing records.

That's Phase 1. Six features plus one immutability constraint. Doesn't introduce new entities. Doesn't change the estimate builder structure beyond adding one line type. Doesn't change Log structure beyond adding two type-picker entries.

### Post-migration — destination architecture

Once on Supabase + Vercel, the full destination ships:

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

9. **Calculated line basis options for the destination.** Are there other bases beyond the four named (`subtotal`, `subtotal_plus_op`, `labor_only`, `materials_only`)? Insurance is sometimes calculated against a labor-only basis in commercial work. Custom-formula bases are V2.

10. **Insurance V2 model.** The destination handles Insurance as a calculated line. The V2 model (annual policy allocated by project value / annual revenue) is queued. Confirm the calculated-line approach is the right starting point.

11. **Allowance reconciliation defaults (post-migration).** When Bari picks "Client Pays Difference" or "Generate Deductive CO" at project close, the system drafts a CO. Does the CO go to the client with the standard CO signing flow (probably yes) or get auto-signed since Bari is just settling a known allowance (probably no — clients still need to consent)?

12. **Project closure gate (post-migration).** Should a project be allowed to move to `complete` with unreconciled allowances, or should the system block closure until each is settled? My lean: warn but don't block. Unallocated deposits at close auto-allocate to "general contract value"; allowances surface a prompt but don't block.

13. **Library seeding by industry preset (post-migration).** V1 light: seed 5-10 cost items + 1-3 assemblies per preset. Is that right, or should V1 ship with empty libraries (let users build organically) to avoid cluttering with items that don't fit specific contractors' practices?

14. **Cross-space data model conflicts.** When Business Finance ships, it'll want to categorize income by source. Should FSPayment carry an additional `revenue_category` field for tax/reporting purposes? Or does Business Finance derive that from project type + payment line linkage?

15. **Display grouping for assembly-sourced lines (post-migration).** When an assembly drops 8 line items into an estimate, should the estimate display them collapsed under the assembly name (with a "show details" expand) or always flat? Probably default to flat (consistent with the line-by-line model) but allow contractor to toggle.

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
- Research: JobTread three-layer library model + Houzz Pro AI assist (web research 2026-04-29)

---

*End of spec. Awaiting Doron's review and Hyphae build sequence for Phase 1.*
