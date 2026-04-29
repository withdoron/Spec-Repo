# FIELD-SERVICE-TOGGLE-AUDIT.md

> **Purpose.** Ground-truth inventory of every `features_json` toggle in the Field Service space — where it's stored, where it's read, whether the gate is enforced, what the user expectation is, and whether the toggle is doing real work or is dead scaffolding. This audit follows from Doron's dogfood finding (Management Fee toggle does nothing; O&P toggle is the inverse) and the membrane fix shipped earlier today (commits `d201c10` + `e57010f`). Read-only — no code changes in this pass.
> **Scope.** `FieldServiceProfile.features_json`. The seven toggles surfaced in the Settings UI's "Workspace Features" section, plus any others in code or live data. Industry preset system included because it's the bigger frame the toggles point at.
> **Reframe.** These are **not "advanced feature flags"** — they are **industry adapters.** The intent is to hide fields that don't apply to a given kind of business. A general contractor (Bari) sees Permits, Subs, O&P, Xactimate. A dog walker sees none of those. A consulting solo wants only the simplest set. The audit asks not just "are gates enforced?" but **"is the toggle layer actually accomplishing industry adaptation, or is it half-built scaffolding?"**

---

## 1. Executive Summary

- **Seven toggles. Five WORKING, one BROKEN, one DEAD.** The user-reported bug (O&P always shows regardless of toggle) is real and the fix is one line. Management Fee toggle is reaching for a feature that doesn't exist yet — the schema has `FSEstimate.management_fee_pct` but no UI consumer; the toggle is gating zero code. **Trust in the Settings surface is shaky** because two of seven toggles lie or do nothing — fix or remove before the financial workflow spec lands.
- **The headline finding is below the toggle layer: `industry_preset` is completely dead.** It's set to `'general_contractor'` exactly once (in `ensureFieldServiceProfileForBusiness.js:104`), read nowhere, and drives nothing — not feature defaults, not field labels, not template seeding (despite the schema description claiming all three per DEC-090). Bari, a future dog walker, and a consulting solo all currently get the same `general_contractor` workspace because the preset system has no consumers.
- **Per the audit's reframe: all seven toggles are WASTED in the long view.** They're doing manual work that the industry preset should do automatically. For Bari, every "construction-relevant" toggle should default ON (he's a general contractor — he doesn't need to discover that he wants Permits and Subs). For a dog walker, all of them should default OFF and ideally not appear in Settings at all. Today the user has to manually toggle each one regardless of business type. The fix is bigger than fixing the broken gate — it's wiring the preset system the schema already describes.
- **The estimate builder has the most concentrated trust gap.** Three toggles point at it (O&P, Xactimate, Management Fee); two of three lie. Doron will be in the estimate builder for the financial workflow spec session — fixing these before that session means he can trust what he sees while speccing.
- **Two WORKING toggles are subtly partial.** `subs_enabled` hides the People-tab Subs section but doesn't hide the `subcontractor` line-item category in the estimate builder (a sub-line is still selectable). `timeline_enabled` hides the Timeline entry button but the timeline view itself remains reachable through inline navigation paths. Neither is broken in a user-trust-corroding way; both are flagged for completeness.
- **Recommendation: ship a small toggle-fix bundle before the financial workflow spec.** Three changes, all trivial: (a) gate O&P section in estimate builder on `overhead_profit_enabled`, (b) hide Management Fee toggle from Settings UI until the field is wired (deferred to financial spec) OR add a "Coming soon" disabled state, (c) optionally hide the deprecated `insurance_work_enabled` migration shim from Settings. Estimated total: 30 minutes of code + Doron's dogfood pass.
- **The deeper recommendation — wire industry_preset — is medium effort and probably the right move before launching to non-Bari users.** This audit doesn't propose the implementation; it surfaces the gap.

---

## 2. Toggle Inventory

The seven keys defined in `FEATURE_DEFAULTS` (`src/utils/ensureFieldServiceProfileForBusiness.js:26-34` and `src/components/fieldservice/FieldServiceSettings.jsx:47-55`). Plus the deprecated `insurance_work_enabled` migration shim.

| Toggle key                  | Settings UI label              | Default | Read sites (file:line)                                                                 | Gate enforced? | User expectation                                                              | Reality                                                                                                     | Verdict |
|-----------------------------|--------------------------------|---------|----------------------------------------------------------------------------------------|----------------|-------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|---------|
| `permits_enabled`           | "Permits & Inspections"         | true    | `FieldServiceProjects.jsx:1120`                                                        | YES            | Toggle off → permit-tracking UI disappears                                    | Hides the Permits panel inside Project Detail. (No top-level Permits tab exists; this is the only surface.)  | WORKING |
| `subs_enabled`              | "Subcontractor Tracking"        | true    | `FieldServicePeople.jsx:696`                                                           | PARTIAL        | Toggle off → "subcontractor" concept disappears across the workspace          | Hides the Subcontractors section in People tab only. Estimate builder still offers `subcontractor` as a line-item category at `FieldServiceEstimates.jsx:959`. | WORKING (partial) |
| `management_fees_enabled`   | "Management Fees"               | false   | (none)                                                                                 | NO             | Toggle on → a Management Fee % field appears on estimates                     | No UI consumes the toggle. `FSEstimate.management_fee_pct` field exists on the entity but no surface displays or computes against it. The toggle gates nothing. | DEAD    |
| `overhead_profit_enabled`   | "Overhead & Profit (O&P)"       | false   | (none — the two `FieldServiceSettings.jsx:61-62` matches are migration backfill, not gating) | NO             | Toggle off → O&P percentage line disappears from estimate builder             | O&P section is hardcoded "always visible" per the inline comment at `FieldServiceEstimates.jsx:1041`. Toggle state has zero effect on the estimate. | BROKEN  |
| `xactimate_enabled`         | "Xactimate Formatting"          | false   | `FieldServiceSettings.jsx:374`, `FieldServiceEstimates.jsx:887`                        | YES            | Toggle on → Trade Categories appear in Settings; estimates render in Xactimate grouped-by-trade format | Both gates fire correctly. Trade Categories section appears in Settings; Xactimate grouped rendering kicks in on the estimate detail page. | WORKING |
| `payments_enabled`          | "Payment Tracking"              | true    | `FieldServiceProjects.jsx:1109`                                                        | YES            | Toggle off → payment-recording UI disappears                                  | Hides the Payments panel inside Project Detail. (No top-level Payments tab exists.)                          | WORKING |
| `timeline_enabled`          | "Project Timeline"              | true    | `FieldServiceProjects.jsx:1372`                                                        | PARTIAL        | Toggle off → timeline view disappears                                          | Hides the Timeline button on Project Detail. The timeline VIEW itself (`FieldServiceProjects.jsx:540-545`) is not gated, but with the button hidden the user can't reach it through normal navigation. | WORKING (entry-only gate) |
| `insurance_work_enabled`    | (not surfaced)                   | (deprecated) | `FieldServiceSettings.jsx:60-65` (migration only)                                    | n/a            | n/a — deprecated; not visible in Settings UI                                  | Migration shim only. Records with this key set get split into `overhead_profit_enabled` + `xactimate_enabled` on profile load, then the key is deleted from the merged object. Per the DEC documentation, deprecated since the audit identified the split. | DEPRECATED |

### Settings UI rendering

All seven user-visible toggles render from one map in `FieldServiceSettings.jsx:333-341`:

```js
[
  { key: 'permits_enabled',         label: 'Permits & Inspections',  desc: 'Track building permits, inspection logs, and eBuild links' },
  { key: 'subs_enabled',            label: 'Subcontractor Tracking', desc: 'Add subs to estimates, assign to projects, track in People tab' },
  { key: 'management_fees_enabled', label: 'Management Fees',        desc: 'Add a percentage-based management fee to estimates' },
  { key: 'overhead_profit_enabled', label: 'Overhead & Profit (O&P)', desc: 'Add an O&P percentage line to estimates for insurance work' },
  { key: 'xactimate_enabled',       label: 'Xactimate Formatting',   desc: 'Group estimate line items by trade category in Xactimate style' },
  { key: 'payments_enabled',        label: 'Payment Tracking',       desc: 'Track payments received per project' },
  { key: 'timeline_enabled',        label: 'Project Timeline',       desc: 'Chronological view of project activity' },
]
```

This is the single source of UI truth. Adding/removing toggles is a one-line change here. No per-preset filtering, no role gating — every workspace owner sees the same seven toggles with the same labels regardless of business type.

---

## 3. Per-Toggle Deep Dives

### 3.1 `overhead_profit_enabled` — BROKEN

The "always visible" comment is explicit at [FieldServiceEstimates.jsx:1041](community-node/src/components/fieldservice/FieldServiceEstimates.jsx):

```jsx
{/* O&P — always visible (important for insurance work) */}
<div className="flex justify-between items-center">
  <span className="text-foreground-soft">O&P (Overhead & Profit)</span>
  <div className="relative">
    <input ... value={formData.overhead_profit_pct} onChange={...} />
    <span className="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground/70 text-sm">%</span>
  </div>
</div>
{formTotals.opAmount > 0 && (
  <div className="flex justify-between items-center text-sm pt-1">
    <span>O&P Amount</span><span>{fmt(formTotals.opAmount)}</span>
  </div>
)}
```

The `formData.overhead_profit_pct` numeric input is always rendered. The `formTotals.opAmount` row appears whenever the user types a number > 0. None of this consults `features.overhead_profit_enabled`.

**Minimum fix:** wrap the `{/* O&P */}` block in `{features?.overhead_profit_enabled && (...)}`. Estimated 1-line change. Verify Bari's existing estimate (EST-2026-004 has `overhead_profit_pct: 0` per JOBS-MIGRATION-PLAN, so the visual change in the preview is zero either way; he'll only feel it on new estimates).

**Side effect:** the existing estimate detail render at line 486-487 also unconditionally shows the O&P row when `overhead_profit_pct > 0`. That's a separate render path. The fix should also gate the detail-view O&P render on `features.overhead_profit_enabled` to keep the toggle honest end-to-end.

### 3.2 `management_fees_enabled` — DEAD

No code consumes this toggle. The schema field `FSEstimate.management_fee_pct` exists (audit §6.5 / §11.4 of the current-state audit) but no UI surfaces it — there is no "Management Fee" input in the estimate builder, no rendered row in the estimate preview, no calc consumer.

The toggle was added as scaffolding for a feature that hasn't been built. The financial workflow spec being written this afternoon is the natural place for that feature to land — calculated lines, % of subtotal, management fee as a real estimate row.

**Two options:**
1. **Hide the toggle from the Settings UI until the feature ships.** Remove the `management_fees_enabled` row from the `FieldServiceSettings.jsx:333-341` map. Toggle remains in `FEATURE_DEFAULTS` so existing profiles aren't disturbed; it just disappears from the UI. After the financial spec wires the feature, re-add the toggle to the map.
2. **Render with a "Coming soon" disabled state.** Keeps the placeholder visible but marks it clearly as not-yet. More signal-rich, costs slightly more code.

**Decision needed.** Option 1 is honest and trivial (1-line removal). Option 2 telegraphs that something is coming. Doron's preference.

### 3.3 `subs_enabled` — WORKING (partial)

Gates the "Subcontractors" section in People tab at `FieldServicePeople.jsx:696`. That's the user-facing surface for managing subs.

**The partial part:** the estimate builder offers `subcontractor` as one of five line-item categories (`materials | labor | subcontractor | fee | other`) at `FieldServiceEstimates.jsx:959`. That category selection is unconditional. So a contractor with `subs_enabled = false` cannot manage a sub roster but can still tag estimate line items as subcontractor.

**Severity:** low. The `subcontractor` line-item category is a label, not a structural feature; even if it appears, picking it doesn't open any sub-related machinery. Probably fine to leave alone for now. Flag for the financial spec to consider whether `subs_enabled = false` should also remove the `subcontractor` category from the estimate builder's category picker.

### 3.4 `timeline_enabled` — WORKING (entry-only gate)

The Timeline button on Project Detail (`FieldServiceProjects.jsx:1372`) is gated. The Timeline view itself (`FieldServiceProjects.jsx:540-545`, mounted via `view === 'timeline'` state) is not — but it's only reachable by clicking the gated button, so in practice the gate holds.

**Severity:** none today. If a future code path sets `view = 'timeline'` from elsewhere (e.g., a deep-link, a notification CTA), the gate will leak. Defensible architecture would also wrap the view render in the gate.

### 3.5 `xactimate_enabled` — WORKING (canonical example)

Two gate sites, both functional:
- `FieldServiceSettings.jsx:374` — Trade Categories section in Settings appears only when on. Lets the user manage the category list that Xactimate-style estimates group by.
- `FieldServiceEstimates.jsx:887` — estimate detail render switches between standard line-items table and Xactimate grouped-by-trade rendering.

This is the cleanest example in the seven. Both sites consult the toggle. Both surfaces respond to toggle changes immediately on save.

### 3.6 `permits_enabled` and `payments_enabled` — WORKING (top-level OK)

Both gate sub-views inside Project Detail. There are no top-level Permits or Payments tabs in the FS workspace shell (the tabs are home, log, projects, estimates, people, documents, settings — verified at `src/config/workspaceTypes.js:245-251`). Since the gates fire on the only mount sites for the respective components, no other surfaces exist that the toggle would need to gate.

---

## 4. Industry Preset — The Bigger Story

### 4.1 Current state

`FieldServiceProfile.industry_preset` is an enum field with four values in the entity schema:

```js
enum: ['general_contractor', 'specialty_trade', 'auto_mechanic', 'service_provider']
default: 'general_contractor'
description: "Controls default feature toggle configuration, field labels, and document template seeding. DEC-090."
```

**Read sites in code: ZERO.** Verified via `grep -rn "industry_preset" src/ base44/` — only the schema definition and `ensureFieldServiceProfileForBusiness.js:104` (which always sets `'general_contractor'`) appear.

**What the schema description claims it drives:**
1. Default feature toggle configuration — **not wired.** `FEATURE_DEFAULTS` is one constant; same defaults for every preset.
2. Field labels — **not wired.** The Settings UI labels are hardcoded in the toggle map; no preset-driven label customization exists.
3. Document template seeding — **not wired.** `initializeWorkspace` always seeds the same 4 Oregon lien-law templates regardless of preset.

**Net:** every workspace today is a `general_contractor` workspace because the preset is always set that way and nothing reads it. The audit's earlier finding (current-state audit §11.5 / 11.6) framed several toggles as "duplications" or "drift"; the deeper finding here is that those toggles are doing manual labor the preset should be doing automatically.

### 4.2 The preset → features_json mapping that should exist

Sketched below. **Not implementation guidance** — illustrative only, to make the gap concrete:

| Toggle / preset             | general_contractor | specialty_trade | auto_mechanic | service_provider | (future: dog_walker / consulting_solo) |
|-----------------------------|--------------------|-----------------|---------------|------------------|----------------------------------------|
| `permits_enabled`           | true               | true            | false         | false            | false                                  |
| `subs_enabled`              | true               | sometimes       | false         | sometimes        | false                                  |
| `overhead_profit_enabled`   | true               | true            | false         | false            | false                                  |
| `xactimate_enabled`         | sometimes          | sometimes       | false         | false            | false                                  |
| `management_fees_enabled`   | sometimes          | sometimes       | false         | sometimes        | false                                  |
| `payments_enabled`          | true               | true            | true          | true             | true (always)                          |
| `timeline_enabled`          | true               | sometimes       | false         | sometimes        | sometimes                              |

Three observations from this sketch:
1. **Most toggles default the same way for any preset that's not a construction trade.** A clean "preset = construction" vs "preset = service" split would cut the toggle list significantly for non-GCs.
2. **`payments_enabled` is universal.** Every business that takes money tracks payments. Probably should never have been a toggle — should be unconditional.
3. **`xactimate_enabled` and `management_fees_enabled` are "sometimes" within the GC preset itself.** These are real user choices even after preset adaptation — not all GCs do insurance work; not all GCs charge a management fee. These remain as toggles even in a preset-driven world.

### 4.3 Recommended evolution (out of scope for this audit, surfaced for design)

A real preset wiring would:
1. **Move `FEATURE_DEFAULTS` from one constant to a preset-keyed dict.** `FEATURE_DEFAULTS_BY_PRESET = { general_contractor: {...}, auto_mechanic: {...}, ... }`. `ensureFieldServiceProfileForBusiness` reads the preset from the business or from the onboarding wizard's selection and applies the corresponding defaults.
2. **Make Settings UI preset-aware.** Render only the toggles that are meaningfully variable for that preset. `auto_mechanic` doesn't need to see "Permits & Inspections" toggle at all.
3. **Wire `industry_preset` selection into FS onboarding.** Today onboarding (`FieldServiceOnboarding.jsx`) doesn't ask the user what kind of business they run; it just sets `general_contractor`. A simple preset picker on onboarding would close the loop.
4. **Optional: drive document template seeding from preset.** Auto mechanic doesn't need Oregon lien-law templates; a service provider doesn't either. `initializeWorkspace` could seed different template sets per preset.

This is a **medium effort** change — maybe a half-day to a day. It's not in this audit's scope to implement; surfacing the gap.

---

## 5. Recommended Fixes (prioritized by impact / effort ratio)

| # | Verdict | Toggle / Surface              | Recommendation                                                                                              | Effort     |
|---|---------|-------------------------------|-------------------------------------------------------------------------------------------------------------|------------|
| 1 | BROKEN  | `overhead_profit_enabled`     | Wrap O&P section in `FieldServiceEstimates.jsx:1041-1057` (builder) and `:486-487` (detail render) in `features?.overhead_profit_enabled && (...)`. Add `features` prop to FieldServiceEstimates if not already passed (it is — verified). | trivial    |
| 2 | DEAD    | `management_fees_enabled`     | Remove the toggle row from `FieldServiceSettings.jsx:333-341` until the financial workflow spec wires the feature. Keep the key in `FEATURE_DEFAULTS` so existing profiles don't break. Re-add when the field has a UI consumer. | trivial    |
| 3 | PARTIAL | `subs_enabled` ↔ estimate category | Optionally gate the `subcontractor` line-item category in the estimate builder's category picker on `subs_enabled`. Or accept the partial gate and document it. Decision belongs in the financial spec. | trivial    |
| 4 | DEPRECATED | `insurance_work_enabled`   | The migration shim at `FieldServiceSettings.jsx:60-65` is still doing useful work for legacy profiles. Leave alone. Audit recommendation only: when the next migration sweep happens, verify zero records still carry the old key, then delete the shim. | none today |
| 5 | DESIGN  | `industry_preset` system       | Wire the preset → defaults pipeline. See §4.3. Pre-launch decision; out of scope for the financial spec. | medium     |
| 6 | DESIGN  | `payments_enabled`             | Consider whether this should be a toggle at all. Every business takes payments. Likely should be unconditional. Pre-launch cleanup. | trivial-but-decision |
| 7 | DESIGN  | `timeline_enabled` view-render | Defensively wrap the timeline view render at `FieldServiceProjects.jsx:540-545` in the same `features?.timeline_enabled !== false` gate. Cheap insurance against a future deep-link path. | trivial    |

**Bundle recommendation: ship #1 + #2 before the financial workflow spec session.** That pass closes the two toggles that lie. #3, #6, #7 are one-line tweaks that can ride along or wait. #5 is a separate planning conversation.

---

## 6. Open Questions for Mycelia / Doron

1. **Hide `management_fees_enabled` (option 1) or render it disabled with "Coming soon" (option 2)?** Both are honest. Option 1 keeps the Settings surface lean; option 2 telegraphs intent. Probably option 1 unless the financial spec session decides it lands soon enough to leave as-is.
2. **Should `payments_enabled` even be a toggle?** Hard to imagine a business that uses Field Service but doesn't track payments. Removing it would simplify the toggle list.
3. **Should `subs_enabled` also gate the estimate builder's `subcontractor` line-item category?** Today it doesn't, and the inconsistency is small. If the answer is "yes, the toggle should reach there too," the financial spec is the right place — alongside the broader category-picker design (where calculated lines and allowance lines will also live).
4. **`industry_preset` design — full wiring vs targeted wiring?** The full version (§4.3 four-step plan) is medium effort. A minimal version — just preset-keyed `FEATURE_DEFAULTS` plus an onboarding picker — is small. Which version is right for the next external user (Peter at Elite Auto Tech N Tow)?
5. **Audit's earlier `overhead_profit_enabled` finding (current-state audit §16):** the audit framed top-level vs `features_json` as a "duplication" but didn't flag the gate-not-enforced bug at the estimate-builder render. This audit catches it. Does the current-state audit need an amendment, or is the new audit sufficient as the canonical reference?
6. **Does the `xactimate_enabled` Settings → Trade Categories chain hold under a preset model?** Today Trade Categories appears when Xactimate is on. Under a preset model, `auto_mechanic` would never see Xactimate, so Trade Categories would never appear. That's correct behavior — flagging only because it confirms the preset model doesn't break this particular chain.
7. **Should this audit's recommendations ship as one bundle or split?** Bundle is faster to verify (one Bari pass); split is easier to bisect if something regresses. Default recommendation: bundle, because each fix is one line.

---

## 7. What I Decided and Why

- **I marked five toggles WORKING and zero WASTED.** The user's reframe (toggles should be industry-preset-driven) is true but applies to the *whole layer*, not individual toggles. Calling each WORKING toggle "WASTED" would dilute the verdict — the fix isn't "remove this toggle" but "wire industry_preset and let it drive the toggle." I lifted that into §4 as the headline structural finding instead. The verdict column captures whether each gate is honest *today*; the design conversation is separate.
- **I marked `subs_enabled` and `timeline_enabled` PARTIAL but kept them in WORKING.** Both are functional in their primary surfaces; the leak is at the edges. Demoting either to BROKEN would overstate the user impact. The estimate builder's `subcontractor` category leaking through `subs_enabled` is the kind of thing that surfaces in a follow-up dogfood, not a blocker.
- **I did not propose new entity fields, refactored shapes, or storage changes.** The audit framing was clear: V1 fix is making the existing shape work. Industry preset gets surfaced as a design conversation in §4, not a recommendation to ship. The shape of the data model is the financial workflow spec's job.
- **I called the management fee fix "remove from UI until wired" rather than "wire it now."** The financial workflow spec is hours away and the natural place to design what % of subtotal lines look like. Wiring management fees as a one-off feature now would be churn — design once, in the spec, then remove the toggle's hidden state when the field has a consumer.
- **I did not include `insurance_work_enabled` in the toggle inventory user-facing toggles.** It's deprecated migration shim. Listed it once in the table for completeness but tagged DEPRECATED so it doesn't muddy the count. The "seven toggles" count refers to user-visible toggles in Settings.
- **I flagged `payments_enabled` as a candidate for "shouldn't be a toggle at all."** Doron's reframe (toggles are industry adapters) implies that universal capabilities don't belong in Settings. Payments are universal. Surfacing this as a design question rather than a recommendation because removing the toggle is reversible and probably correct — but it's a conversation, not a unilateral call.
- **I did NOT investigate non-FS workspaces.** The audit was scoped to Field Service. Other workspaces (Property Management, Team, Finance) likely have their own toggle layers; whatever pattern emerges from the FS pass should probably propagate via Living Feet (DEC-146). That's a future cross-workspace audit, not this one.

---

*End of audit.*
