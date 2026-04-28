# JOBS-MIGRATION-PLAN

> **STATUS: DEFERRED** — Plan complete but execution paused on 2026-04-28.
>
> The Jobs rename is deferred to post-Phase-6 (after the Supabase + Vercel migration). On Postgres, this rename becomes a SQL `ALTER TABLE ... RENAME TO` plus field renames that preserve data automatically — minutes, not days. Investing labor in renaming on Base44 is investing in infrastructure being abandoned.
>
> **What stays useful in this file post-deferral:**
> - **Sections 1, 2, 4, 7** — overview, live data baseline, entity rename map, and workspace identifier locations. These catalog *what* changes regardless of *how* the rename executes. Carry forward to the post-Supabase rename.
> - **Section 12** — post-migration report template. Adaptable to any future rename.
>
> **What gets superseded post-Supabase:**
> - **Sections 3, 5, 6, 8, 9, 10, 11** — Base44-specific mechanics (legacy_id pattern, two-pass architecture, maintenance flag, compatibility shim, FK rewrite logic, rollback gradient). On Postgres these become trivial or unnecessary.
>
> **Why deferred (not abandoned):**
> - Bari and Dr. Holman experience zero real-world friction from the "Field Service" name
> - Tiles-7 (Personal Profile + Settings), per-business workspace wiring for Finance/Team, and the Founding 50 launch are all unblocked by deferring
> - The Desk-as-roll-up reframe does NOT require the rename first — Desk can read from FieldServiceProfile data just as easily as JobProfile
>
> **One real architectural finding from this session worth preserving** (caught during VERIFY before any code shipped): ClientPortal.jsx queries portal records by entity record ID, not by `portal_token` field as Section 5.4 assumes. Any future rename — on Base44 OR Supabase — must account for this. Existing portal URLs Bari has shared with Dr. Holman contain the FSEstimate.id (`69c6fba24b8b8812fe4f5419`), not the portal_token. Post-rename, those URLs need a fallback resolution pattern (e.g., legacy_id lookup at the route layer) or a redirect strategy.
>
> ---

**Migration:** Field Service → Jobs Namespace Rename
**Plan author:** Mycelia (Sections 3-12) + Doron/Mycelia integration (Sections 1-2)
**Plan compiled:** 2026-04-28
**Status:** Authoritative — use as contract for execution
**Executor:** Hyphae (build artifacts) → Doron (fire migration)
**Branch:** `jobs-migration`

---

## Section 1 — Overview & Commit Statement

### 1.1 What this migration does

Renames the "Field Service" namespace to "Jobs" across the LocalLane platform. Twelve entities are renamed (FieldServiceProfile → JobProfile, FSProject → Job, FSEstimate → JobEstimate, etc.). The workspace identifier `field-service` becomes `jobs` in code, server functions, agent instructions, and one Business record's `enabled_spaces` array. All live records are copied to the new namespace with full data integrity, then old entities are archived for 30 days before deletion.

### 1.2 Why we're doing it

"Field Service" is industry jargon — it implies trades, contractors, dirty work. The space serves a much broader population: photographers, mobile groomers, consultants, caterers, house cleaners, handymen, anyone doing project-based client work. "Jobs" is the universal noun. It's what people actually call this work.

This rename is also load-bearing for the Desk-as-roll-up reframe queued post-migration. After this migration, Jobs is its own space (parallel to Property Pulse, Events, Finance). Desk becomes a roll-up reader pulling from all activated spaces in a context. Building Desk-as-roll-up before Jobs is its own space would require renaming twice. Doing the rename first makes the reframe trivial.

### 1.3 Non-negotiables

**Bari's data integrity.** Bari Swartz (he/him, owner of Red Umbrella business id `69ea5590481b7e15af7216b6`) has live records. His Holman barn renovation project, his $121,657.57 signed estimate (EST-2026-004 with portal_token `ea14972d-3978-4381-860b-849f4b4d3694`), his Subcontractor Agreements with their portal tokens — all must survive the migration with byte-level fidelity. Verification queries in Section 10 prove this; they are not optional.

**Three live portal tokens must continue to resolve.** Any client with a saved URL referencing one of the three tokens (Dr. Holman with the estimate, the awaiting-signature counterparty on the Subcontractor Agreement, the historical signed Notice of Right to Lien) must be able to load that URL post-migration and see the same content as pre-migration. The tokens are UUIDs, not entity IDs; they copy verbatim and the portal lookup function updates to query the new entities.

**No data loss, ever.** The plan uses create-new + migrate + archive-30-days. Old entities are never deleted in the migration itself. The archival cleanup happens at +30 days as a separate operation, after stability is confirmed. If anything goes wrong, the old data is intact and recoverable.

**The maintenance flag prevents the moving-train problem.** While the migration runs, an AdminSettings flag blocks new writes to FS entities at the agentScopedWrite layer. Bari sees a banner; his data is still readable; he just can't create new records during the window. The flag is removed post-cutover. Without this gate, a new write during migration would orphan in the old entity.

### 1.4 Scope boundaries

**In scope:**
- 12 entity creates (JobProfile through JobChangeOrder) with `legacy_id` field on each
- 10 live record copies (Pass 1) and 31 FK rewrites (Pass 2)
- agentScopedQuery + agentScopedWrite server function updates with `field-service` → `jobs` compatibility shim
- Frontend code: entity imports, WORKSPACE_TYPES constant, route renames with redirects, portal lookup function
- FieldServiceAgent → JobsAgent rename in Base44 dashboard
- One Business record `enabled_spaces` backfill (`desk` → `jobs` on Red Umbrella)
- Post-migration verification report with full audit trail

**Out of scope (explicitly):**
- Desk-as-roll-up reframe (separate arc, post-migration)
- Personal Profile + Settings (tiles-7, separate arc)
- Per-business workspace wiring for Finance, Team (depends on Desk-as-roll-up)
- FK field name standardization on JobClient (`workspace_id` → `profile_id`, "Path B" cleanup, scheduled separately after Jobs is stable)
- Schema field cleanup on legacy fields (boost mechanic deprecated fields, coordinate duplication, category vs categories) — Phase 5/6 cleanup territory
- URL field validation sweep — Phase 5 cleanup
- Stripe integration / actual charging — months out

### 1.5 Risk profile

This is the highest-stakes platform change shipped to date. It exceeds Phase 2 reparenting (2026-04-23) in scope: 12 entities vs. 5 records reparented, 31 FK rewrites vs. ~6, three live portal tokens vs. zero, full agent and code namespace rename vs. just data layer.

The mitigating factor: by virtue of being so disciplined, the migration is more rollback-safe than smaller changes. Two-pass architecture, idempotent retry at every step, AuditLog for every mutation, old entities preserved for 30 days, maintenance flag preventing concurrent writes. If anything goes wrong, the recovery path is clear.

Mycelia's recommendation: schedule for a low-traffic window. Doron's clarification: traffic is just himself, so any window works. Decision: fire when Doron is rested, focused, and has uninterrupted hours available, with Hyphae standing by for live debugging.

---

## Section 2 — Live Data Snapshot (Pre-Migration Baseline)

This section captures the exact state the migration starts from. The dry-run action in Phase 0 must produce numbers that match this snapshot. If they don't, new records have been created since this plan was drafted and the plan needs an update before execution proceeds.

### 2.1 Record counts as of 2026-04-28

| Entity | Live Record Count | Notes |
|---|---|---|
| FieldServiceProfile | 2 | Bari (Red Umbrella, active) + Doron (test sandbox, archived 2026-04-23) |
| FSClient | 3 | |
| FSProject | 1 | Bari's Holman barn renovation |
| FSEstimate | 1 | Bari's $121K accepted estimate |
| FSDocument | 3 | 2 Subcontractor Agreements (Bari) + 1 Notice of Right to Lien (Doron-test) |
| FSDailyLog | 0 | |
| FSMaterialEntry | 0 | |
| FSLaborEntry | 0 | |
| FSDailyPhoto | 0 | |
| FSPayment | 0 | |
| FSPermit | 0 | |
| FSChangeOrder | 0 | Schema exists, no records |
| **Total live records** | **10** | |

### 2.2 Critical record IDs

These are the records the verification queries in Section 10 specifically validate. They must survive the migration with the IDs and field values listed.

**FieldServiceProfile records:**

| ID | Owner | Workspace Name | Business ID | State |
|---|---|---|---|---|
| `69baba55a6b9cca0c7d5700b` | Bari Swartz | Red Umbrella | `69ea5590481b7e15af7216b6` | active |
| `69b6c265746397cbf8e184f3` | Doron Fletcher | (test sandbox) | (none) | archived 2026-04-23 |

**FSProject record:**

| ID | Profile | Name | Status | Total Budget |
|---|---|---|---|---|
| `69ebaa59a3b3dcbc375a7ca9` | `69baba55a6b9cca0c7d5700b` (Bari) | "Dr. Nathan Holman   " (verbatim with trailing spaces) | active | 121657.57 |

**FSEstimate record:**

| ID | Profile | Estimate # | Status | Total | Portal Token |
|---|---|---|---|---|---|
| `69c6fba24b8b8812fe4f5419` | `69baba55a6b9cca0c7d5700b` | EST-2026-004 | accepted | 121657.57 | `ea14972d-3978-4381-860b-849f4b4d3694` |

Additional fields to preserve verbatim: `responded_at: "2026-04-24T17:37:30.168Z"`, `signed_at: null`, `owner_signature_data: null`, `prepared_by: "Bari Swartz"`, `line_items` count = 30, `portal_link_active: true`.

**FSDocument records:**

| ID | Title | Status | Portal Token | Profile |
|---|---|---|---|---|
| `69c6fa2731cdee2982627f4a` | Subcontractor Agreement | draft | (null) | `69baba55a6b9cca0c7d5700b` (Bari) |
| `69c6cfb440fa5e43db334f1c` | Subcontractor Agreement | awaiting_signature | `77f38acf-cd5c-4a1e-af63-c94f575952b3` | `69baba55a6b9cca0c7d5700b` (Bari) |
| `69c698c2214ef0d74eca1336` | Notice of Right to Lien | signed | `4c11ee24-6427-4105-a3e9-1a99686d9c8f` | `69b6c265746397cbf8e184f3` (Doron-test) |

The signed Notice of Right to Lien has `signed_at: "2026-03-27T21:05:53.623Z"` and populated `owner_signature_data` (base64 image blob). Both must copy verbatim.

### 2.3 Three live portal tokens

These tokens are the most fragile thing in the migration. They are UUIDs used as URL slugs. Any client with a saved portal URL resolves through these tokens. They must copy verbatim in Pass 1 and remain untouched through Pass 2.

| Token | Entity (old) | Entity (new) | Status | Risk |
|---|---|---|---|---|
| `ea14972d-3978-4381-860b-849f4b4d3694` | FSEstimate | JobEstimate | accepted, portal_link_active: true | Dr. Holman may have URL saved |
| `77f38acf-cd5c-4a1e-af63-c94f575952b3` | FSDocument | JobDocument | awaiting_signature | Counterparty actively expecting to sign |
| `4c11ee24-6427-4105-a3e9-1a99686d9c8f` | FSDocument | JobDocument | signed | Historical, accessible for reference |

### 2.4 enabled_spaces state

As of 2026-04-28, exactly **one** Business record has `"desk"` in its `enabled_spaces` array:

| Business ID | Name | enabled_spaces |
|---|---|---|
| `69ea5590481b7e15af7216b6` | Red Umbrella | `["profile", "settings", "desk"]` |

Phase 4 backfill rewrites this to `["profile", "settings", "jobs"]`. Could grow before migration runs — the query must run dynamically, not assume count = 1.

### 2.5 Schema state

- Twelve old FS* entities exist with their current schemas
- Zero Job* entities exist (this is the starting point)
- AdminSettings entity exists and is queryable
- AdminAuditLog entity exists (note: existing rows have all-null structured fields — your migration writes will populate fields explicitly, query pattern `filter where action LIKE "JOBS_MIGRATION_%"` returns only migration rows)
- MIGRATION_SECRET environment variable is **not set** (you set it via Base44 dashboard immediately before firing the migration, delete it after)

---

## Section 3 — ID Strategy

**Status:** Authoritative — use as contract for migration script
**Author:** Mycelia
**Date:** 2026-04-28
**Verified against:** Live LocalLane data, Base44 SDK behavior as observed in this app

### 3.1 The Core Question

When `base44.entities.NewEntity.create(payload)` receives a payload that includes an `id` field, does Base44 honor it as the record's actual ID, or does it ignore it and auto-generate?

**Answer: Base44 always auto-generates the ID. The `id` field in a create payload is silently ignored.**

This is not documented explicitly in the Base44 SDK docs, but it is observable from every record created in this app's history. Every record's `id` is a platform-assigned MongoDB ObjectId. There is no `createWithId`, no `upsert`, no `importRecord` path available in either the standard SDK or `base44.asServiceRole`. The Deno runtime in server functions exposes the same SDK surface — no override exists there either.

This is not a soft constraint. It is a platform boundary. Do not test around it — there is no path through.

**Consequence:** We cannot preserve original IDs. Every record copied to a new entity gets a fresh ID. Every FK in every child entity must be rewritten. The `legacy_id` pattern exists to make that rewrite tractable.

### 3.2 Does asServiceRole Change Anything?

No. `base44.asServiceRole` bypasses Row Level Security and permission checks — it does not change the ID assignment behavior. The platform assigns the ID after the create call regardless of whether you're operating as a user or as the service role.

What `asServiceRole` does give you that matters for migration:

- Reads all records regardless of who created them (bypasses RLS — critical, because without this you'd only see records created by the current session user)
- Creates records that are not owned by any specific user (no `created_by` scoping)
- Updates any record regardless of ownership

For the migration script, every read, create, and update must go through `base44.asServiceRole`. If you use user-scoped access for any step, you will silently miss records belonging to other users.

### 3.3 The legacy_id Pattern — Mandatory, Not Fallback

Because IDs cannot be preserved, every new entity schema gets a `legacy_id` string field. This field holds the original record's ID and serves three purposes:

1. **FK resolution** — when rewriting child FKs, look up the new parent by `legacy_id == old_id`
2. **Idempotency** — before creating a new record, check if one already exists with the matching `legacy_id`; if yes, skip
3. **Audit trail** — permanently linkable back to the original record for any post-migration investigation

Schema addition for every new entity:

```json
"legacy_id": {
  "type": "string",
  "description": "Original record ID from pre-migration FS entity. Null on records created after migration cutover."
}
```

This field is nullable. Records created natively after cutover will have no `legacy_id`. Only migrated records carry a value. **Do not remove this field post-migration** — it's the permanent audit link.

### 3.4 What This Means for the FK Chain

The full FK chain in the FS namespace is:

```
FieldServiceProfile (root)
  └─ FSClient          (workspace_id → FieldServiceProfile.id)
  └─ FSProject         (profile_id   → FieldServiceProfile.id)
       └─ FSEstimate   (profile_id   → FieldServiceProfile.id, project_id → FSProject.id, client_id → FSClient.id)
       └─ FSDocument   (profile_id   → FieldServiceProfile.id, project_id → FSProject.id, client_id → FSClient.id)
       └─ FSDailyLog   (profile_id   → FieldServiceProfile.id, project_id → FSProject.id)
       └─ FSPermit     (profile_id   → FieldServiceProfile.id, project_id → FSProject.id)
       └─ FSPayment    (profile_id   → FieldServiceProfile.id, project_id → FSProject.id)
       └─ FSMaterialEntry (profile_id → FieldServiceProfile.id, project_id → FSProject.id)
       └─ FSLaborEntry    (profile_id → FieldServiceProfile.id, project_id → FSProject.id)
```

Because IDs are not preserved, every arrow in that diagram is a rewrite target. The `legacy_id` pattern is what makes this tractable without a transaction guarantee:

- All entities are copied first (Pass 1) with unresolved FKs still holding old ID values
- All FKs are resolved after all copies are complete (Pass 2) using `legacy_id` lookups

The full FK rewrite enumeration is in Section 6. The ordering constraint it implies is in Section 5.

### 3.5 The enabled_spaces Backfill — Sequencing Flag

This is real and requires a dedicated migration step.

What I found from live data:

Red Umbrella (id: `69ea5590481b7e15af7216b6`) currently has:

```json
"enabled_spaces": ["profile", "settings", "desk"]
```

`"desk"` is the current catalog ID for the Field Service space. This is the only business record out of 8 in the system that has it. When the catalog ID renames to `"jobs"`, `"desk"` becomes an unknown space ID. The resolver defensively skips unknown space IDs — so if code ships before this backfill runs, Red Umbrella silently loses its Desk space.

**Backfill step:**

```js
// Find all Business records with "desk" in enabled_spaces
const businesses = await base44.asServiceRole.entities.Business.list();
const affected = businesses.filter(b => 
  Array.isArray(b.enabled_spaces) && b.enabled_spaces.includes("desk")
);

for (const business of affected) {
  const updated = business.enabled_spaces.map(s => s === "desk" ? "jobs" : s);
  await base44.asServiceRole.entities.Business.update(business.id, {
    enabled_spaces: updated
  });
  await writeAuditLog({
    action: "JOBS_MIGRATION_ENABLED_SPACES_BACKFILL",
    entity_type: "Business",
    entity_id: business.id,
    details: {
      before: business.enabled_spaces,
      after: updated,
      reversal: `Business.update("${business.id}", { enabled_spaces: ${JSON.stringify(business.enabled_spaces)} })`
    }
  });
}
```

**Affected records right now:** 1 (Red Umbrella). That could grow before migration runs — the query must run dynamically, not assume count = 1.

**Position in migration order:** This backfill runs as its own step after all entity copies are complete and verified, but **before the code deploy**. It is a prerequisite gate for the code cutover. Do not ship the catalog rename in code until this step has a passing audit log entry.

**Rollback:** The audit log entry for each record includes the reversal command with the original `enabled_spaces` array. Revert by replaying those reversals in the same order.

**Verification query:**

```js
const businesses = await base44.asServiceRole.entities.Business.list();
const stillHasDesk = businesses.filter(b => 
  Array.isArray(b.enabled_spaces) && b.enabled_spaces.includes("desk")
);
// Expected: 0 records
// Any result > 0 means backfill incomplete — block code deploy
```

### 3.6 Idempotency of the ID Strategy

The `legacy_id` field enables clean idempotency on every create step:

```js
async function copyRecord(sourceRecord, targetEntityName) {
  // Check: already migrated?
  const existing = await base44.asServiceRole.entities[targetEntityName].list();
  const alreadyDone = existing.find(r => r.legacy_id === sourceRecord.id);
  if (alreadyDone) {
    return { status: "skipped", reason: "already_migrated", legacy_id: sourceRecord.id };
  }

  // Copy with unresolved FKs (Pass 1)
  const newRecord = await base44.asServiceRole.entities[targetEntityName].create({
    ...sourceRecord,
    id: undefined,       // explicitly strip — Base44 ignores it anyway, but be explicit
    legacy_id: sourceRecord.id
  });

  await writeAuditLog({ ... });
  return { status: "created", new_id: newRecord.id, legacy_id: sourceRecord.id };
}
```

Running this function twice on the same source record produces exactly one destination record. Safe to retry at any point.

### 3.7 Verification: Confirming ID Behavior Before Full Run

Before the migration script runs against all entities, the dry-run step should include a live ID behavior probe:

```js
// Create a throwaway record in a new entity that already exists
// (use a sandbox entity or a known-empty Job* entity before any real copies run)
const probe = await base44.asServiceRole.entities.JobProfile.create({
  id: "test-id-that-should-not-appear",
  legacy_id: "probe-test",
  user_id: "probe",
  workspace_name: "probe-delete-me"
});

const confirmed_id = probe.id;
const id_was_honored = confirmed_id === "test-id-that-should-not-appear";

await base44.asServiceRole.entities.JobProfile.delete(probe.id);

return {
  probe: "id_assignment_behavior",
  id_was_honored,           // expected: false
  platform_assigned_id: confirmed_id
};
```

This probe runs in dry-run mode, creates and immediately deletes a throwaway record, and documents the behavior. **If `id_was_honored` comes back `true`, stop immediately** — it means the platform has changed behavior and the `legacy_id` plan needs revisiting. Confirm with Mycelia before proceeding.

### 3.8 Summary Table

| Question | Answer |
|---|---|
| Can you set `id` on `.create()`? | No. Base44 always auto-generates. Payload `id` is silently ignored. |
| Does `asServiceRole` change ID behavior? | No. Only changes permission scope, not ID assignment. |
| Is `legacy_id` a fallback? | No. It is the only viable approach given Base44's constraints. |
| Does the Deno server runtime expose a different create path? | Not observed. If Hyphae finds one, confirm with Mycelia before using. |
| How many FKs need rewriting? | All of them — full enumeration in Section 6. |
| Does `enabled_spaces: ["desk"]` need a data backfill? | Yes. 1 record currently affected (Red Umbrella). Backfill before code deploy. |
| Is the backfill a separate migration step? | Yes. Post-entity-copy, pre-code-deploy. It is a gate condition. |

---

## Section 4 — Entity Rename Map

**Status:** Authoritative — use as contract for migration script
**Author:** Mycelia
**Date:** 2026-04-28
**Source:** Live record field inspection + agent spec cross-reference

### 4.1 Usage Contract for Hyphae

**Field classifications:**

- **KEEP** — field name and value copy verbatim
- **RENAME** — field name changes; value copies verbatim into the new name
- **DROP** — not carried forward (none in this migration — see rationale at end)
- **NEW** — added to the new entity schema, not on the source

Base44 auto-generates `id`, `created_date`, `updated_date`, `created_by`, `created_by_id` on every record. Do not attempt to copy these — they will be overwritten on create. `legacy_id` is the bridge back.

FK columns are called out explicitly in each entity section. FK values are old-ID references that get rewritten in Pass 2 (Section 6). Copy them verbatim in Pass 1 — they will be stale until rewritten.

**Indexes / uniqueness constraints:** Base44 does not expose user-defined indexes in its SDK or dashboard schema editor. The platform manages internal indexes. There are no user-defined unique constraints to recreate. `invite_code` on FieldServiceProfile is the only field with apparent uniqueness semantics — it's application-enforced, not a DB constraint, and copies verbatim.

### 4.2 Path A confirmation on JobClient.workspace_id

Path A is locked. `JobClient.workspace_id` field name is **preserved** (NOT normalized to `profile_id`). The reasoning: normalizing during a high-stakes data migration is scope creep on a move that's already touching 12 entities. The inconsistency is cosmetic from Hyphae's perspective once she knows it exists. The only real risk is in agentScopedQuery's ENTITY_FK map — JobClient must be registered with `fk: "workspace_id"` to match. That's a one-line entry, not a structural problem.

The FK field standardization pass happens as a standalone operation after Jobs is stable (Path B, separate Hyphae session, no related context here).

### 4.3 Entity 1: FieldServiceProfile → JobProfile

**Role:** Root profile. Parent of all child entities.
**Live record count:** 2

| Field | Status | Notes |
|---|---|---|
| `user_id` | KEEP | FK → User.id. Not rewritten (User IDs don't change). |
| `workspace_name` | KEEP | |
| `business_name` | KEEP | |
| `owner_name` | KEEP | |
| `license_number` | KEEP | |
| `phone` | KEEP | |
| `email` | KEEP | |
| `website` | KEEP | |
| `service_area` | KEEP | |
| `tagline` | KEEP | |
| `hourly_rate` | KEEP | |
| `logo_url` | KEEP | |
| `brand_color` | KEEP | |
| `workers_json` | KEEP | JSON blob — not a FK. Copies verbatim. |
| `user_roles` | KEEP | JSON blob. |
| `default_terms` | KEEP | |
| `phase_labels` | KEEP | JSON blob. |
| `invite_code` | KEEP | Application-enforced uniqueness. Copies verbatim. |
| `features_json` | KEEP | JSON blob of feature flags. |
| `trade_categories_json` | KEEP | JSON blob. |
| `insurance_work_enabled` | KEEP | |
| `overhead_profit_enabled` | KEEP | |
| `xactimate_enabled` | KEEP | Present at top level AND inside `features_json`. Both copy verbatim. Pre-existing duplication. |
| `industry_preset` | KEEP | |
| `guide_dismissed` | KEEP | |
| `linked_business_workspace_id` | KEEP | FK → Business.id. Not rewritten. |
| `linked_finance_workspace_id` | KEEP | FK → FinancialProfile.id. Not rewritten. |
| `subscription_tier` | KEEP | |
| `tier_trial_start` | KEEP | |
| `business_id` | KEEP | FK → Business.id. Not rewritten. |
| `archived_at` | KEEP | ⚠ Doron-test record has `archived_at: "2026-04-23T17:23:27.451Z"`. Copies verbatim. |
| `archived_by` | KEEP | |
| `legacy_id` | NEW | Holds original FieldServiceProfile.id. Populated by migration script. |

**FK summary:** `user_id`, `linked_business_workspace_id`, `linked_finance_workspace_id`, `business_id` — none rewritten in this migration.

### 4.4 Entity 2: FSClient → JobClient

**Role:** Client records scoped to a workspace.
**Live record count:** 3
**FK to parent:** `workspace_id` (Path A: preserved as-is on JobClient)

| Field | Status | Notes |
|---|---|---|
| `workspace_id` | KEEP | FK → JobProfile.id (was FieldServiceProfile.id). **Field name preserved per Path A.** Value rewritten in Pass 2. |
| `user_id` | KEEP | FK → User.id. Not rewritten. |
| `name` | KEEP | |
| `email` | KEEP | |
| `phone` | KEEP | |
| `address` | KEEP | |
| `city` | KEEP | |
| `state` | KEEP | |
| `zip_code` | KEEP | |
| `company_name` | KEEP | |
| `notes` | KEEP | |
| `status` | KEEP | Enum: `active` confirmed in live data. |
| `business_id` | KEEP | FK → Business.id. Not rewritten. |
| `legacy_id` | NEW | |

**FK summary:** `workspace_id` → JobProfile.id — REWRITE in Pass 2; `user_id`, `business_id` — not rewritten.

**Note for agentScopedQuery update:**

```js
// BEFORE
"FSClient": { fk: "workspace_id", workspace: "field-service" }
// AFTER
"JobClient": { fk: "workspace_id", workspace: "jobs" }
```

The FK field name stays `workspace_id`. This is the one place in agentScopedQuery where the FK field name is not `profile_id`. **Hyphae: do not accidentally normalize this to `profile_id` when updating the ENTITY_FK map.**

### 4.5 Entity 3: FSProject → Job

**Role:** The job record itself.
**Live record count:** 1 (Bari's Holman barn renovation)
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | FK → JobProfile.id. Value rewritten in Pass 2. |
| `user_id` | KEEP | |
| `client_id` | KEEP | FK → JobClient.id. Value rewritten in Pass 2. |
| `estimate_id` | KEEP | FK → JobEstimate.id. Value rewritten in Pass 2. |
| `name` | KEEP | Job name field. Note: this entity uses `name`, not `title`. |
| `status` | KEEP | `active` confirmed in live data. |
| `description` | KEEP | |
| `address` | KEEP | |
| `notes` | KEEP | Live value: "Created from estimate EST-2026-004". |
| `start_date` | KEEP | |
| `estimated_end_date` | KEEP | Field is `estimated_end_date`, not `end_date`. Confirmed from live record. |
| `client_name` | KEEP | Denormalized client name snapshot. |
| `client_email` | KEEP | Denormalized. |
| `client_phone` | KEEP | Denormalized. |
| `client_address` | KEEP | Denormalized. Note: `null` in live record despite client having an address. |
| `total_budget` | KEEP | |
| `original_budget` | KEEP | `null` in live record. |
| `show_costs_to_clients` | KEEP | |
| `client_show_breakdown` | KEEP | |
| `business_id` | KEEP | Not rewritten. |
| `legacy_id` | NEW | |

**FK summary:** `profile_id`, `client_id`, `estimate_id` all REWRITE in Pass 2. `business_id` not rewritten.

⚠ **Circular-adjacency note:** `FSProject.estimate_id → FSEstimate`, and `FSEstimate.project_id → FSProject`. Neither is a true circular dependency (they don't block creation of each other), but both must be fully copied before either FK can be resolved. Pass 2 handles both in the same sweep.

### 4.6 Entity 4: FSEstimate → JobEstimate

**Role:** Estimates/proposals.
**Live record count:** 1 (Bari/Red Umbrella, $121,657.57, status: accepted)
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | FK → JobProfile.id. Value rewritten in Pass 2. |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | **FK field renamed AND value rewritten in Pass 2.** |
| `client_id` | KEEP | FK → JobClient.id. Value rewritten in Pass 2. |
| `estimate_number` | KEEP | EST-2026-004 in live record. |
| `title` | KEEP | |
| `status` | KEEP | `accepted` in live record. |
| `date` | KEEP | |
| `valid_until` | KEEP | |
| `due_date` | KEEP | |
| `line_items` | KEEP | Large JSON blob (30 line items in live record). Copies verbatim. |
| `labor_estimate` | KEEP | JSON blob. |
| `subtotal` | KEEP | |
| `overhead_profit_pct` | KEEP | |
| `management_fee_pct` | KEEP | |
| `tax_rate_pct` | KEEP | |
| `tax_rate` | KEEP | |
| `tax_amount` | KEEP | |
| `other_amount` | KEEP | |
| `total` | KEEP | |
| `payment_terms` | KEEP | |
| `terms` | KEEP | |
| `conditions` | KEEP | |
| `notes` | KEEP | |
| `prepared_by` | KEEP | "Bari Swartz" in live record. |
| `is_insurance_estimate` | KEEP | |
| `client_show_breakdown` | KEEP | |
| `client_name` | KEEP | Denormalized. |
| `client_email` | KEEP | Denormalized. |
| `client_phone` | KEEP | Denormalized. |
| `client_address` | KEEP | Denormalized. |
| `sent_at` | KEEP | |
| `viewed_at` | KEEP | |
| `responded_at` | KEEP | `2026-04-24T17:37:30.168Z` in live record — signature acceptance timestamp. **Must copy exactly.** |
| `portal_token` | KEEP | ⚠ `ea14972d-3978-4381-860b-849f4b4d3694` — live signed estimate's portal token. **Must copy verbatim.** Any portal link Bari or Dr. Holman has saved points to this token. |
| `portal_link_active` | KEEP | `true` in live record. |
| `sent_for_signature_at` | KEEP | |
| `recalled_at` | KEEP | |
| `signed_at` | KEEP | |
| `owner_signature_data` | KEEP | `null` in live record (contractor hasn't countersigned). Copy verbatim. |
| `owner_signed_at` | KEEP | |
| `business_id` | KEEP | Not rewritten. |
| `legacy_id` | NEW | |

**FK summary:** `profile_id` REWRITE; `project_id` (renamed `job_id`) RENAME + REWRITE; `client_id` REWRITE; `business_id` not rewritten.

⚠ **Portal token warning:** The `portal_token` is what makes the estimate's public-facing URL work. It is not a FK — it's a UUID string used as a URL slug. It copies verbatim. After migration, the portal lookup code must query JobEstimate instead of FSEstimate. Section 5.4 covers the timing.

### 4.7 Entity 5: FSDailyLog → JobLog

**Role:** Daily site logs.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | FK → JobProfile.id. Rewritten in Pass 2. |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | FK → Job.id. |
| `date` | KEEP | |
| `notes` | KEEP | |
| `weather` | KEEP | |
| `hours_worked` | KEEP | |
| `workers_on_site` | KEEP | |
| `photos` | KEEP | JSON array of photo URLs, not a FK to FSDailyPhoto. Copies verbatim. |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

**Hyphae note:** Zero records means no Pass 2 FK rewrites needed. Schema must still exist correctly. Confirm exact field list against live schema before relying on this pattern for sibling entities.

### 4.8 Entity 6: FSMaterialEntry → JobMaterial

**Role:** Material cost tracking per job.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | FK → JobProfile.id. |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | FK → Job.id. |
| `estimate_id` | KEEP | FK → JobEstimate.id. |
| `name` | KEEP | |
| `description` | KEEP | |
| `quantity` | KEEP | |
| `unit` | KEEP | |
| `unit_cost` | KEEP | |
| `total_cost` | KEEP | |
| `supplier` | KEEP | |
| `purchased_at` | KEEP | |
| `receipt_url` | KEEP | |
| `category` | KEEP | |
| `status` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

### 4.9 Entity 7: FSLaborEntry → JobLabor

**Role:** Labor tracking per job.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | |
| `worker_name` | KEEP | |
| `worker_id` | KEEP | |
| `date` | KEEP | |
| `hours` | KEEP | |
| `rate` | KEEP | |
| `total` | KEEP | |
| `description` | KEEP | |
| `category` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

### 4.10 Entity 8: FSDailyPhoto → JobPhoto

**Role:** Photos attached to daily logs.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | |
| `log_id` | KEEP | FK → JobLog.id. |
| `photo_url` | KEEP | |
| `caption` | KEEP | |
| `taken_at` | KEEP | |
| `uploaded_by` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

**Hyphae note:** No live records to confirm this field list. Cross-reference against the actual Base44 schema for FSDailyPhoto before building the copy step.

### 4.11 Entity 9: FSPayment → JobPayment

**Role:** Payment records per job.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | |
| `estimate_id` | KEEP | |
| `client_id` | KEEP | |
| `amount` | KEEP | |
| `payment_date` | KEEP | |
| `method` | KEEP | |
| `notes` | KEEP | |
| `status` | KEEP | |
| `invoice_number` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

### 4.12 Entity 10: FSPermit → JobPermit

**Role:** Permit tracking per job.
**Live record count:** 0
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | |
| `permit_type` | KEEP | |
| `permit_number` | KEEP | |
| `status` | KEEP | |
| `applied_date` | KEEP | |
| `issued_date` | KEEP | |
| `expiry_date` | KEEP | |
| `notes` | KEEP | |
| `inspections` | KEEP | JSON blob. |
| `portal_url` | KEEP | |
| `apply_url` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

### 4.13 Entity 11: FSDocument → JobDocument

**Role:** Contracts and signed documents.
**Live record count:** 3 (2 Subcontractor Agreements, 1 Notice of Right to Lien)
**FK to parent:** `profile_id`

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | FK → JobProfile.id. Rewritten in Pass 2. |
| `project_id` | RENAME → `job_id` | All 3 live records have `project_id: null` — rename still required for schema consistency. |
| `client_id` | KEEP | FK → JobClient.id. Rewritten in Pass 2. |
| `template_id` | KEEP | FK → document template. Not in FS entity scope — not rewritten. |
| `title` | KEEP | |
| `content` | KEEP | Full document text blob. Copies verbatim. |
| `status` | KEEP | |
| `sent_at` | KEEP | |
| `signed_at` | KEEP | |
| `signature_data` | KEEP | ⚠ Contains base64 signature image + consent + hash. Copies verbatim. **Do not touch.** |
| `document_url` | KEEP | |
| `portal_token` | KEEP | ⚠ Same portal-token sensitivity as FSEstimate. Copies verbatim. |
| `portal_link_active` | KEEP | |
| `portal_expires_at` | KEEP | |
| `recalled_at` | KEEP | |
| `recalled` | KEEP | |
| `amendment_of` | KEEP | FK → another FSDocument.id. ⚠ If non-null, points to old FSDocument ID. After migration must point to corresponding JobDocument.id. **Rewrite in Pass 2 via legacy_id lookup.** All 3 live records have `amendment_of: null` — no active rewrites needed, but logic must handle it. |
| `signer_name` | KEEP | |
| `signer_email` | KEEP | |
| `owner_signature_data` | KEEP | JSON blob with base64 sig. Copies verbatim. |
| `owner_signed_at` | KEEP | |
| `archived` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

**FK summary:**
- `profile_id` REWRITE
- `project_id` (renamed `job_id`) RENAME + REWRITE
- `client_id` REWRITE
- `amendment_of` REWRITE if non-null (currently null on all live records)
- `template_id` not rewritten (out of scope)
- `business_id` not rewritten

### 4.14 Entity 12: FSChangeOrder → JobChangeOrder

**Role:** Change orders for in-flight jobs (mid-project scope changes).
**Live record count:** 0
**FK to parent:** `profile_id`

⚠ **Not in original rename map** — Mycelia surfaced this entity during Section 4 audit. It exists as a live entity (returns empty set, no error) and is registered in agentScopedQuery's ENTITY_FK spec. Must rename or be orphaned.

Confirmed in scope: rename to `JobChangeOrder` along with everything else.

| Field | Status | Notes |
|---|---|---|
| `profile_id` | KEEP | |
| `user_id` | KEEP | |
| `project_id` | RENAME → `job_id` | |
| `estimate_id` | KEEP | |
| `client_id` | KEEP | |
| `title` | KEEP | |
| `description` | KEEP | |
| `amount` | KEEP | |
| `status` | KEEP | |
| `date` | KEEP | |
| `approved_at` | KEEP | |
| `approved_by` | KEEP | |
| `notes` | KEEP | |
| `business_id` | KEEP | |
| `legacy_id` | NEW | |

**Hyphae note:** Verify the actual field list against the live schema before building this step.

### 4.15 Entities NOT In Scope

| Entity Name | Status | Disposition |
|---|---|---|
| FSWorker | Does not exist as standalone entity | Workers stored as `workers_json` on FieldServiceProfile. No migration needed. |
| FSInvoice | Does not exist as standalone entity | Invoicing handled via FSEstimate status lifecycle. No migration needed. |
| FSFrequencySubmission | Exists, separate namespace | **Do not touch.** The FS prefix is a naming collision with Frequency Station. This entity has nothing to do with Field Service. |

### 4.16 DROP Policy

**No fields are dropped in this migration.** Rationale: the purpose of this migration is a namespace rename, not a schema cleanup. Dropping fields during a data migration increases risk surface (what if something reads a dropped field after cutover?) and is out of scope. Any deprecated or redundant fields — like the `xactimate_enabled` duplication on FieldServiceProfile, or any future obsolete fields — get a separate cleanup pass after Jobs is stable.

### 4.17 Complete FK Rewrite Inventory

This is the exhaustive list for Section 6. Every row here is a Pass 2 rewrite target.

| Entity | Field (new name) | Target Entity | Rewrite Required? |
|---|---|---|---|
| JobClient | `workspace_id` | JobProfile | ✅ yes |
| Job | `profile_id` | JobProfile | ✅ yes |
| Job | `client_id` | JobClient | ✅ yes |
| Job | `estimate_id` | JobEstimate | ✅ yes |
| JobEstimate | `profile_id` | JobProfile | ✅ yes |
| JobEstimate | `job_id` | Job | ✅ yes |
| JobEstimate | `client_id` | JobClient | ✅ yes |
| JobLog | `profile_id` | JobProfile | ✅ yes |
| JobLog | `job_id` | Job | ✅ yes |
| JobMaterial | `profile_id` | JobProfile | ✅ yes |
| JobMaterial | `job_id` | Job | ✅ yes |
| JobMaterial | `estimate_id` | JobEstimate | ✅ yes |
| JobLabor | `profile_id` | JobProfile | ✅ yes |
| JobLabor | `job_id` | Job | ✅ yes |
| JobPhoto | `profile_id` | JobProfile | ✅ yes |
| JobPhoto | `job_id` | Job | ✅ yes |
| JobPhoto | `log_id` | JobLog | ✅ yes |
| JobPayment | `profile_id` | JobProfile | ✅ yes |
| JobPayment | `job_id` | Job | ✅ yes |
| JobPayment | `estimate_id` | JobEstimate | ✅ yes |
| JobPayment | `client_id` | JobClient | ✅ yes |
| JobPermit | `profile_id` | JobProfile | ✅ yes |
| JobPermit | `job_id` | Job | ✅ yes |
| JobDocument | `profile_id` | JobProfile | ✅ yes |
| JobDocument | `job_id` | Job | ✅ yes |
| JobDocument | `client_id` | JobClient | ✅ yes |
| JobDocument | `amendment_of` | JobDocument | ✅ yes (if non-null) |
| JobChangeOrder | `profile_id` | JobProfile | ✅ yes |
| JobChangeOrder | `job_id` | Job | ✅ yes |
| JobChangeOrder | `estimate_id` | JobEstimate | ✅ yes |
| JobChangeOrder | `client_id` | JobClient | ✅ yes |

**Total FK rewrite targets: 31**

**Not rewritten (out-of-scope FKs):**
- `user_id` on all entities → User.id (User IDs unchanged)
- `business_id` on all entities → Business.id (out of scope)
- `template_id` on JobDocument → document template (out of scope)
- `linked_business_workspace_id` / `linked_finance_workspace_id` on JobProfile (out of scope)

---

## Section 5 — Order of Operations

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28

### 5.1 Governing Principles

Three rules that determine the entire sequence:

**Rule 1 — Parents before children.** JobProfile must exist before any child entity is copied, because Pass 2 FK rewrites look up parent IDs by `legacy_id`. A child entity copied before its parent's `legacy_id` is indexed will stall the FK rewrite. Copy order: JobProfile → JobClient + Job (siblings) → all leaf children.

**Rule 2 — All copies before any FK rewrites.** Pass 1 (copy all records) and Pass 2 (rewrite all FKs) are fully separated. No FK rewrites run until every entity's copy step is complete. This eliminates the ordering problem within Pass 2 — by the time any FK is resolved, all target records already exist and are findable by `legacy_id`.

**Rule 3 — Portal continuity gates the code cutover.** The code cutover (server function swap + frontend deploy) cannot happen until portal lookup is dual-covered. This means the portal resolution function must query both old and new entities during the transition window — or the old entities must remain live and queryable until after code ships. Details in 5.4.

### 5.2 Full Sequence

```
═══════════════════════════════════════════════════════
PHASE 0 — PREPARE
═══════════════════════════════════════════════════════

Step 0.1  Set MIGRATION_SECRET in LocalLane app environment
Step 0.2  Deploy jobsMigration server function (dry-run mode only)
Step 0.3  Run dry-run → review output → confirm record counts
          match Section 4 inventory
          GATE: dry-run must report zero warnings before proceeding

═══════════════════════════════════════════════════════
PHASE 1 — CREATE NEW ENTITY SCHEMAS
═══════════════════════════════════════════════════════
(All schemas created before any records are written.
Idempotent — safe to re-run if interrupted.)

Step 1.1   Create JobProfile schema
Step 1.2   Create JobClient schema
Step 1.3   Create Job schema
Step 1.4   Create JobEstimate schema
Step 1.5   Create JobDocument schema
Step 1.6   Create JobLog schema
Step 1.7   Create JobMaterial schema
Step 1.8   Create JobLabor schema
Step 1.9   Create JobPhoto schema
Step 1.10  Create JobPayment schema
Step 1.11  Create JobPermit schema
Step 1.12  Create JobChangeOrder schema

Each schema includes legacy_id: string as a nullable field.
Each schema mirrors the old entity's field list per Section 4,
with project_id renamed to job_id on all child entities.

GATE: Confirm all 12 schemas exist before Phase 2.

═══════════════════════════════════════════════════════
PHASE 2 — PASS 1: COPY ALL RECORDS (unresolved FKs)
═══════════════════════════════════════════════════════

Step 2.1  Copy FieldServiceProfile → JobProfile         (2 records)
Step 2.2  Copy FSClient → JobClient                     (3 records)
Step 2.3  Copy FSProject → Job                          (1 record)
Step 2.4  Copy FSEstimate → JobEstimate                 (1 record)
Step 2.5  Copy FSDocument → JobDocument                 (3 records)
Step 2.6  Copy FSChangeOrder → JobChangeOrder           (0 records — fast)
Step 2.7  Copy FSDailyLog → JobLog                      (0 records — fast)
Step 2.8  Copy FSMaterialEntry → JobMaterial            (0 records — fast)
Step 2.9  Copy FSLaborEntry → JobLabor                  (0 records — fast)
Step 2.10 Copy FSDailyPhoto → JobPhoto                  (0 records — fast)
Step 2.11 Copy FSPayment → JobPayment                   (0 records — fast)
Step 2.12 Copy FSPermit → JobPermit                     (0 records — fast)

Rules:
- legacy_id = source record's id on every created record
- Idempotency check before every create (skip if legacy_id already exists)
- AuditLog row for every created record
- Do NOT attempt to rewrite FKs in this phase
- At this point all FK columns on new records still hold old IDs — this is expected

GATE: Count parity check — every new entity's record count
must equal its source entity's record count before Phase 3.

═══════════════════════════════════════════════════════
PHASE 3 — PASS 2: FK REWRITE
═══════════════════════════════════════════════════════
(Full detail in Section 6. Summary here.)

Step 3.1  Build ID mapping tables in memory:
            FSProfile_id → JobProfile.id   (via legacy_id lookup)
            FSClient_id  → JobClient.id
            FSProject_id → Job.id
            FSEstimate_id → JobEstimate.id
            FSDocument_id → JobDocument.id

Step 3.2  Rewrite JobClient.workspace_id
Step 3.3  Rewrite Job.profile_id, Job.client_id, Job.estimate_id
Step 3.4  Rewrite JobEstimate.profile_id, JobEstimate.job_id, JobEstimate.client_id
Step 3.5  Rewrite JobDocument.profile_id, JobDocument.job_id,
          JobDocument.client_id, JobDocument.amendment_of
Step 3.6  Rewrite JobChangeOrder (all FKs — 0 records, instant)
Step 3.7  Rewrite JobLog, JobMaterial, JobLabor, JobPhoto,
          JobPayment, JobPermit (all 0 records — instant)

GATE: Run FK integrity check — every FK on every new record
must resolve to an existing record in the target entity.
Any unresolved FK = migration failure. Stop and diagnose.

═══════════════════════════════════════════════════════
PHASE 4 — VERIFICATION
═══════════════════════════════════════════════════════

Step 4.1  Count parity: all 12 entity pairs match
Step 4.2  Bari's workspace check (profile, clients, documents)
Step 4.3  Portal token continuity check (see 5.4 below)
Step 4.4  FK integrity spot-check (sample 3 records per entity,
          verify all FKs resolve in new entities)
Step 4.5  enabled_spaces backfill: rewrite "desk" → "jobs" on
          all Business records (1 record currently: Red Umbrella)

GATE: All verification checks pass before Phase 5.
      enabled_spaces backfill must complete before Phase 5.

═══════════════════════════════════════════════════════
PHASE 5 — SERVER FUNCTION CUTOVER
═══════════════════════════════════════════════════════

Step 5.1  Update agentScopedQuery:
            - "field-service" → "jobs" in WORKSPACE_PROFILES
            - All FS* entity names → Job* names in ENTITY_FK
            - FSClient fk stays "workspace_id" (Path A)
            - Add compatibility shim (see 5.5)
          Deploy + publish app

Step 5.2  Update agentScopedWrite (if deployed):
            - Same entity name renames
            - "field-service" → "jobs" in whitelist
          Deploy + publish app

Step 5.3  Smoke-test agentScopedQuery with workspace: "jobs"
          Verify returns Bari's records
          Verify shim: workspace: "field-service" still works

GATE: agentScopedQuery returns correct data under both
      "jobs" and "field-service" (shim active) before Phase 6.

═══════════════════════════════════════════════════════
PHASE 6 — CODE CUTOVER
═══════════════════════════════════════════════════════

Step 6.1  Frontend entity imports: FS* → Job* everywhere
Step 6.2  WORKSPACE_TYPES: "field-service" → "jobs"
Step 6.3  Component renames: JoinFieldService.jsx → JoinJobs.jsx etc.
Step 6.4  Route changes: /join/field-service → /join/jobs etc.
Step 6.5  Portal lookup function: update to query JobEstimate +
          JobDocument instead of FSEstimate + FSDocument
          (see 5.4 — this is the portal continuity step)
Step 6.6  Update FieldServiceAgent instructions:
            workspace: "jobs", all entity name references
Step 6.7  Deploy all code changes as single coordinated release

GATE: Post-deploy smoke test — Bari's workspace loads,
      all records visible, portal link resolves correctly.

═══════════════════════════════════════════════════════
PHASE 7 — ARCHIVE + CLEANUP
═══════════════════════════════════════════════════════

Step 7.1  Remove compatibility shim from agentScopedQuery
          Deploy + publish app
Step 7.2  Mark old FS* entities as deprecated in AdminAuditLog
          (record: "deprecated as of [date], safe to delete after [date+30d]")
Step 7.3  Do NOT delete old FS* entities yet
          Schedule deletion for 30 days post-migration
Step 7.4  File JOBS-POSTMIGRATION-REPORT.md

═══════════════════════════════════════════════════════
30-DAY CLEANUP (scheduled)
═══════════════════════════════════════════════════════

Step 8.1  Confirm no new writes have hit old FS* entities
          since Phase 6 cutover (query created_date on all
          FS* entities — any record newer than cutover date
          means something is still writing to old entities)
Step 8.2  If clean: delete all old FS* entity schemas
Step 8.3  If not clean: trace the write source, patch, repeat
```

### 5.3 Circular-Adjacency: FSProject ↔ FSEstimate

`FSProject.estimate_id → FSEstimate`, and `FSEstimate.project_id → FSProject`. Neither blocks the other's creation (Pass 1 copies both with stale FKs). In Pass 2, both are rewritten in the same sweep — Step 3.3 rewrites `Job.estimate_id`, Step 3.4 rewrites `JobEstimate.job_id`. No special handling required beyond the standard two-pass pattern.

### 5.4 Portal Token Timing — The Critical Window

This is the thing that can silently break Dr. Holman's link and the awaiting-signature document.

**What a portal lookup does:** A portal URL looks something like `/portal/estimate/ea14972d-3978-4381-860b-849f4b4d3694`. When that URL is loaded, a server function (or frontend route) queries the entity for a record where `portal_token == "ea14972d..."` and serves the document. The token is the key. The entity being queried is hardcoded.

**Live portal tokens confirmed:**

| Entity | Record ID | Token | Status | Risk |
|---|---|---|---|---|
| FSEstimate | `69c6fba24b8b8812fe4f5419` | `ea14972d-3978-4381-860b-849f4b4d3694` | accepted / portal_link_active: true | Dr. Holman may have URL saved |
| FSDocument | `69c6cfb440fa5e43db334f1c` | `77f38acf-cd5c-4a1e-af63-c94f575952b3` | awaiting_signature | Actively awaiting signer |
| FSDocument | `69c698c2214ef0d74eca1336` | `4c11ee24-6427-4105-a3e9-1a99686d9c8f` | signed | Historical — lower risk but still accessible |

**The problem window:**

```
Timeline:

T0  Phase 2 completes — records now exist in BOTH FSEstimate AND JobEstimate
    Portal tokens copied verbatim to new records
    Portal lookup code still queries FSEstimate → still works ✓

T1  Phase 3 completes — FKs rewritten on new entities
    No change to portal lookup path → still works ✓

T2  Phase 4 completes — verification passes
    No change to portal lookup path → still works ✓

T3  Phase 5 completes — agentScopedQuery updated
    agentScopedQuery now routes "jobs" → JobEstimate
    BUT portal lookup code is separate from agentScopedQuery
    Portal lookup still queries FSEstimate directly → still works ✓

T4  Phase 6, Step 6.5 — portal lookup function updated to query JobEstimate
    [OLD FSEstimate still exists, still has the token]
    [NEW JobEstimate also has the token — copied verbatim in Step 2.4]
    Portal now queries JobEstimate → works ✓
    If this step is atomic with the rest of the code deploy: no gap ✓

T5  Phase 7 — old FSEstimate archived
    Portal now only exists in JobEstimate → still works ✓

THE DANGER: If Step 6.5 ships AFTER old entities are archived
    Portal queries FSEstimate → entity no longer returns records → 404 ✗
```

**The safe pattern:**

The portal lookup update (Step 6.5) and the old entity archival (Step 7.2) must be sequenced with a verified gap between them:

1. Step 6.5 ships as part of Phase 6 — portal now queries JobEstimate / JobDocument
2. Smoke test portal links explicitly — load Dr. Holman's token URL, confirm it resolves from JobEstimate
3. Only then proceed to Phase 7 archival

Old entities are never deleted mid-migration. They sit alongside the new ones until the 30-day window closes. The only failure mode is if Step 6.5 is forgotten or skipped — which is why it's a named step with an explicit gate, not an implicit assumption.

**One additional guard:** during Phase 2 Step 2.4, when copying FSEstimate to JobEstimate, assert that `portal_token` was written correctly:

```js
const newEstimate = await base44.asServiceRole.entities.JobEstimate.create({ ...record, legacy_id: record.id });
console.assert(
  newEstimate.portal_token === record.portal_token,
  `PORTAL TOKEN MISMATCH: expected ${record.portal_token}, got ${newEstimate.portal_token}`
);
```

Same assertion for FSDocument Step 2.5. If either assertion fails, abort Phase 2 immediately. Do not proceed.

**Bottom line on portal timing:** There is no unavoidable break window if the sequence is followed. The tokens exist in both old and new entities simultaneously from T0 through T5. The only gap would be a sequencing error — skipping 6.5, or archiving before 6.5 ships. Section 9 (Code-Side Cutover Sequence) addresses this as an explicit checklist gate.

### 5.5 Compatibility Shim

Between Phase 5 (server function cutover) and Phase 6 (code cutover), the old frontend code still calls agentScopedQuery with `workspace: "field-service"`. The shim prevents those calls from returning empty results during the transition window.

```js
// In agentScopedQuery, add at top of workspace resolution:
const WORKSPACE_ALIASES = { "field-service": "jobs" };
const resolvedWorkspace = WORKSPACE_ALIASES[workspace] || workspace;
// Use resolvedWorkspace everywhere below instead of workspace
```

The shim is removed in Phase 7, Step 7.1, after Phase 6 code deploy is confirmed working.

---

## Section 6 — FK Rewrite Logic

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28

### 6.1 The Two-Pass Architecture

Pass 1 (Phase 2) copies all records with all FK fields left at their original values — old IDs that point to old entity records. They are stale but not wrong yet. The new records exist. Their FKs just haven't been updated.

Pass 2 (Phase 3) resolves every FK. It builds an in-memory ID mapping table from `legacy_id` lookups, then updates every record that has a FK.

The separation is what makes the whole thing safe and retryable. A crash in Pass 2 doesn't corrupt Pass 1 — the records exist, they just have stale FKs. Re-running Pass 2 from the beginning is safe because the rewrite is idempotent.

### 6.2 Building the ID Mapping Tables

The first thing Phase 3 does — before any record is updated — is build complete in-memory mapping tables for every entity that is a FK target:

```js
async function buildIdMaps() {
  const maps = {};

  // For each entity that other entities point to as a FK target:
  for (const [newEntity, oldEntity] of [
    ["JobProfile",   "FieldServiceProfile"],
    ["JobClient",    "FSClient"],
    ["Job",          "FSProject"],
    ["JobEstimate",  "FSEstimate"],
    ["JobDocument",  "FSDocument"],
  ]) {
    const records = await base44.asServiceRole.entities[newEntity].list();
    maps[oldEntity] = {};
    for (const r of records) {
      if (r.legacy_id) {
        maps[oldEntity][r.legacy_id] = r.id;
      }
    }
  }

  return maps;
  // Result shape:
  // maps["FieldServiceProfile"]["69baba55a6b9cca0c7d5700b"] = "<new JobProfile id>"
  // maps["FSClient"]["69babaedda46b4942a762790"] = "<new JobClient id>"
  // maps["FSProject"]["69ebaa59a3b3dcbc375a7ca9"] = "<new Job id>"
  // maps["FSEstimate"]["69c6fba24b8b8812fe4f5419"] = "<new JobEstimate id>"
  // maps["FSDocument"]["69c6cfb440fa5e43db334f1c"] = "<new JobDocument id>"
  // maps["FSDocument"]["69c698c2214ef0d74eca1336"] = "<new JobDocument id>"
}
```

A missing `legacy_id` entry in the map means that FK cannot be resolved. **Any unresolved FK during Pass 2 is a hard abort** — log it, stop, surface the diagnostic. Do not silently write null.

### 6.3 FK Rewrite — Entity by Entity

Each step uses a lookup helper:

```js
function resolve(maps, sourceEntity, oldId) {
  if (!oldId) return null;  // null FK stays null
  const newId = maps[sourceEntity]?.[oldId];
  if (!newId) throw new Error(
    `UNRESOLVED FK: ${sourceEntity}["${oldId}"] has no mapping. Abort.`
  );
  return newId;
}
```

**Step 3.2 — Rewrite JobClient**

Fields to rewrite: `workspace_id`

```js
const clients = await base44.asServiceRole.entities.JobClient.list();
for (const c of clients) {
  await base44.asServiceRole.entities.JobClient.update(c.id, {
    workspace_id: resolve(maps, "FieldServiceProfile", c.workspace_id)
  });
  auditLog("FK_REWRITE", "JobClient", c.id, {
    field: "workspace_id",
    old: c.workspace_id,
    new: maps["FieldServiceProfile"][c.workspace_id]
  });
}
```

Live records affected: 3

**Step 3.3 — Rewrite Job (was FSProject)**

Fields to rewrite: `profile_id`, `client_id`, `estimate_id`

```js
const jobs = await base44.asServiceRole.entities.Job.list();
for (const j of jobs) {
  await base44.asServiceRole.entities.Job.update(j.id, {
    profile_id:  resolve(maps, "FieldServiceProfile", j.profile_id),
    client_id:   resolve(maps, "FSClient",            j.client_id),
    estimate_id: resolve(maps, "FSEstimate",          j.estimate_id),
  });
}
```

Live records affected: 1

**Note on `estimate_id`:** The live FSProject record has `estimate_id: "69c6fba24b8b8812fe4f5419"`. This resolves to the new JobEstimate's id via `maps["FSEstimate"]`. This is the circular-adjacency resolution — Job now points to the new JobEstimate.

**Step 3.4 — Rewrite JobEstimate**

Fields to rewrite: `profile_id`, `job_id` (was `project_id`), `client_id`

```js
const estimates = await base44.asServiceRole.entities.JobEstimate.list();
for (const e of estimates) {
  await base44.asServiceRole.entities.JobEstimate.update(e.id, {
    profile_id: resolve(maps, "FieldServiceProfile", e.profile_id),
    job_id:     resolve(maps, "FSProject",           e.job_id),
    // Note: e.job_id currently holds the old project_id value — it was
    // written from FSEstimate.project_id during Pass 1. Resolves to Job.id.
    client_id:  resolve(maps, "FSClient",            e.client_id),
  });
}
```

Live records affected: 1 (Bari's $121K accepted estimate)

⚠ **Portal token guard:** Re-assert `portal_token` unchanged after this update:

```js
const updated = await base44.asServiceRole.entities.JobEstimate.get(e.id);
console.assert(updated.portal_token === e.portal_token, "PORTAL TOKEN CORRUPTED ON FK REWRITE");
```

The FK rewrite should never touch `portal_token` — it's not in the update payload. This assertion is a defensive check that the update call didn't accidentally wipe it.

**Step 3.5 — Rewrite JobDocument**

Fields to rewrite: `profile_id`, `job_id` (was `project_id`), `client_id`, `amendment_of`

```js
const docs = await base44.asServiceRole.entities.JobDocument.list();
for (const d of docs) {
  await base44.asServiceRole.entities.JobDocument.update(d.id, {
    profile_id:   resolve(maps, "FieldServiceProfile", d.profile_id),
    job_id:       d.job_id  // null in all 3 live records — stays null
                  ? resolve(maps, "FSProject", d.job_id)
                  : null,
    client_id:    resolve(maps, "FSClient",            d.client_id),
    amendment_of: d.amendment_of
                  ? resolve(maps, "FSDocument", d.amendment_of)
                  : null,
  });
  // Portal token assertion (same pattern as JobEstimate)
  const updated = await base44.asServiceRole.entities.JobDocument.get(d.id);
  if (d.portal_token) {
    console.assert(updated.portal_token === d.portal_token, `PORTAL TOKEN CORRUPTED: doc ${d.id}`);
  }
}
```

Live records affected: 3

**`job_id` note:** All 3 live FSDocument records have `project_id: null`. The `job_id` field will be null on all 3 after Pass 2. The resolve logic must handle null gracefully — the conditional above does this.

**`amendment_of` note:** All 3 live records have `amendment_of: null`. The resolve logic must still be present for correctness when non-null documents exist in future.

**Step 3.6 — Rewrite JobChangeOrder**

Fields to rewrite: `profile_id`, `job_id`, `estimate_id`, `client_id`

```js
const changeOrders = await base44.asServiceRole.entities.JobChangeOrder.list();
for (const co of changeOrders) {
  await base44.asServiceRole.entities.JobChangeOrder.update(co.id, {
    profile_id:  resolve(maps, "FieldServiceProfile", co.profile_id),
    job_id:      resolve(maps, "FSProject",           co.job_id),
    estimate_id: resolve(maps, "FSEstimate",          co.estimate_id),
    client_id:   resolve(maps, "FSClient",            co.client_id),
  });
}
```

Live records affected: 0 — this loop body never executes. The step is still required because the function must exist for any records created between now and migration day.

**Steps 3.7 — Rewrite Remaining Entities**

All currently have 0 live records. The rewrite logic is present for correctness; the loops execute but touch nothing.

```js
// JobLog: profile_id (→ JobProfile), job_id (→ Job)
// JobMaterial: profile_id (→ JobProfile), job_id (→ Job), estimate_id (→ JobEstimate)
// JobLabor: profile_id (→ JobProfile), job_id (→ Job)
// JobPhoto: profile_id (→ JobProfile), job_id (→ Job), log_id (→ JobLog)
// JobPayment: profile_id (→ JobProfile), job_id (→ Job), estimate_id (→ JobEstimate), client_id (→ JobClient)
// JobPermit: profile_id (→ JobProfile), job_id (→ Job)
```

### 6.4 Idempotency of Pass 2

Unlike Pass 1 (which uses `legacy_id` as an idempotency key), Pass 2 rewrites are harder to make idempotent by default — there's no flag that says "this FK was already rewritten."

The safe approach: detect whether FKs are already resolved before running the update.

```js
function needsRewrite(maps, sourceEntity, currentValue) {
  // If currentValue is already a key in the NEW entity's map values
  // (i.e., it's already a new-style ID, not an old-style ID),
  // then it's already been rewritten — skip it.
  const oldIds = new Set(Object.keys(maps[sourceEntity]));
  return oldIds.has(currentValue);  // true = still holds old ID = needs rewrite
}
```

If `currentValue` is no longer in the set of old IDs, it was either already rewritten or it's null. Either way, skip the update. This makes Pass 2 safe to re-run after a partial completion.

### 6.5 Complete FK Rewrite Map (Reference)

See Section 4.17 for the complete enumeration. 31 FK rewrite targets total.

---

## Section 7 — Workspace Identifier Rename

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28

### 7.1 What "workspace identifier" means here

The string `"field-service"` serves two distinct roles:

1. **Routing key** — used in code and server functions to resolve which entity namespace to query (`"field-service"` → FieldServiceProfile and children)
2. **Data value** — stored in records where a workspace type is persisted as a string

Section 3 already confirmed: no FS entity carries `workspace_type: "field-service"` as a row-level field. The FK chain runs on record IDs throughout. The only data-level occurrence found was `Business.enabled_spaces: ["desk"]` — where `"desk"` is the catalog ID for the Field Service/Jobs space, not `"field-service"` itself.

So the workspace identifier rename is almost entirely a code and server function change, with one data record backfill (`enabled_spaces`) already addressed in Section 5 Phase 4.

### 7.2 Every Location, Phase-by-Phase

**Location 1: AdminSettings maintenance flag key**

- Value: `"jobs_migration:maintenance_mode"`
- Phase: Written at start of Phase 2, deleted at end of Phase 6
- Type: Data write (new key, no old value to rename)
- Rollback: Delete the key — single AdminSettings record deletion

**Location 2: agentScopedQuery — WORKSPACE_PROFILES mapping**

Current value:
```js
"field-service": { entity: "FieldServiceProfile", userField: "user_id" }
```

New value:
```js
"jobs": { entity: "JobProfile", userField: "user_id" }
```

- Phase: Phase 5, Step 5.1
- Type: Server function code change
- Transition: Compatibility shim added simultaneously:

```js
const WORKSPACE_ALIASES = { "field-service": "jobs" };
const resolvedWorkspace = WORKSPACE_ALIASES[workspace] || workspace;
```

Shim removed in Phase 7, Step 7.1. Rollback: Redeploy previous function version.

**Location 3: agentScopedQuery — ENTITY_FK map, all FS entries**

Current values (12 entries):
```js
"FieldServiceProfile": { fk: "user_id",      workspace: "field-service", isProfile: true },
"FSProject":           { fk: "profile_id",   workspace: "field-service" },
"FSEstimate":          { fk: "profile_id",   workspace: "field-service" },
"FSClient":            { fk: "workspace_id", workspace: "field-service" },
"FSDocument":          { fk: "profile_id",   workspace: "field-service" },
"FSDailyLog":          { fk: "profile_id",   workspace: "field-service" },
"FSPayment":           { fk: "profile_id",   workspace: "field-service" },
"FSMaterialEntry":     { fk: "profile_id",   workspace: "field-service" },
"FSLaborEntry":        { fk: "profile_id",   workspace: "field-service" },
"FSChangeOrder":       { fk: "profile_id",   workspace: "field-service" },
"FSPermit":            { fk: "profile_id",   workspace: "field-service" },
```

New values:
```js
"JobProfile":     { fk: "user_id",      workspace: "jobs", isProfile: true },
"Job":            { fk: "profile_id",   workspace: "jobs" },
"JobEstimate":    { fk: "profile_id",   workspace: "jobs" },
"JobClient":      { fk: "workspace_id", workspace: "jobs" },  // Path A: workspace_id preserved
"JobDocument":    { fk: "profile_id",   workspace: "jobs" },
"JobLog":         { fk: "profile_id",   workspace: "jobs" },
"JobPayment":     { fk: "profile_id",   workspace: "jobs" },
"JobMaterial":    { fk: "profile_id",   workspace: "jobs" },
"JobLabor":       { fk: "profile_id",   workspace: "jobs" },
"JobChangeOrder": { fk: "profile_id",   workspace: "jobs" },
"JobPermit":      { fk: "profile_id",   workspace: "jobs" },
"JobPhoto":       { fk: "profile_id",   workspace: "jobs" },  // FSDailyPhoto was missing from spec
```

- Phase: Phase 5, Step 5.1 — same deploy as Location 2
- Note: Old FS* entries remain in the map during the shim window so old entity names still resolve. Removed in Phase 7.

**Location 4: agentScopedWrite — whitelist and entity map**

- Phase: Phase 5, Step 5.2
- Type: Server function code change
- Change: Same pattern as agentScopedQuery — rename workspace key and all entity names. Add shim. Remove shim in Phase 7.
- Note: If agentScopedWrite is not yet deployed, skip this step and note in the migration report.

**Location 5: Business.enabled_spaces — "desk" → "jobs"**

- Current state: 1 record (Red Umbrella: `["profile", "settings", "desk"]`)
- New value: `["profile", "settings", "jobs"]`
- Phase: Phase 4, Step 4.5 — runs after FK verification, before server function cutover
- Type: Data record update
- **Why Phase 4 not Phase 6:** This is the Hyphae sequencing constraint from the original brief. The resolver skips unknown space IDs. If code ships with `"jobs"` as the known ID before this record is updated, the space still works (new code knows `"jobs"`). If code ships and the record still has `"desk"` but the resolver no longer knows `"desk"`... the space silently disappears. Backfill first, code after.
- Rollback: `Business.update("69ea5590481b7e15af7216b6", { enabled_spaces: ["profile", "settings", "desk"] })`
- Verification: `Business.list()` → zero records with `"desk"` in `enabled_spaces`

**Location 6: Frontend WORKSPACE_TYPES constant**

- Current value: Something like `{ FIELD_SERVICE: "field-service", ... }` or `"field-service"` used inline
- New value: `"jobs"`
- Phase: Phase 6, Step 6.2
- Type: Frontend code change
- **File path:** Not resolvable from this sandbox — Hyphae must locate. Likely in `src/constants/workspaceTypes.js`, `src/config/workspaces.js`, or similar. Search codebase for `"field-service"` string literals.

**Location 7: Frontend route paths**

- Current: Routes like `/field-service`, `/join/field-service`, `/desk/*`
- New: `/jobs`, `/join/jobs`
- Phase: Phase 6, Steps 6.3–6.4
- Type: Frontend code + router config change
- Note: Old routes should redirect to new routes, not 404. Any bookmarked or shared URL that uses the old path needs a redirect rule in the router for at minimum the 30-day post-migration window.

**Location 8: FieldServiceAgent instructions**

- Current: References `workspace: "field-service"`, entity names like `FSClient`, `FSEstimate`
- New: `workspace: "jobs"`, entity names `JobClient`, `JobEstimate`, etc.
- Phase: Phase 6, Step 6.6
- Type: Agent instruction update in Base44 dashboard
- Note: Agent instructions are not code — they're edited directly in Base44. This is a dashboard change, not a commit. It should happen in the same session as the Phase 6 code deploy, not before.

**Location 9: MylaneNote.source_space — possible "field-service" values**

- Status: Needs query — not confirmed from prior data audit
- Phase: Phase 4, alongside `enabled_spaces` backfill
- Type: Conditional data update
- Action for Hyphae:

```js
const notes = await base44.asServiceRole.entities.MylaneNote.list();
const affected = notes.filter(n => n.source_space === "field-service");
// If count > 0: update each to source_space: "jobs"
// AuditLog each
```

- Rollback: AuditLog entries include reversal command per record

### 7.3 Summary Table

| Location | Value Changed | Phase | Type |
|---|---|---|---|
| AdminSettings maintenance flag | written / deleted | Phase 2 start / Phase 6 end | Data |
| agentScopedQuery WORKSPACE_PROFILES | `"field-service"` → `"jobs"` | Phase 5 | Server function |
| agentScopedQuery ENTITY_FK keys | FS* → Job*, workspace values | Phase 5 | Server function |
| agentScopedWrite whitelist | same | Phase 5 | Server function |
| Business.enabled_spaces | `"desk"` → `"jobs"` | Phase 4 | Data record |
| Frontend WORKSPACE_TYPES | `"field-service"` → `"jobs"` | Phase 6 | Code |
| Frontend routes | `/field-service` → `/jobs` | Phase 6 | Code |
| FieldServiceAgent instructions | entity names + workspace key | Phase 6 | Dashboard |
| MylaneNote.source_space | `"field-service"` → `"jobs"` (if present) | Phase 4 | Data record |

---

## Section 8 — Server Function Strategy

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28

### 8.1 The Question: Is Dual-Write Ever Needed?

**No. Dual-write is not needed and should not be implemented.**

Dual-write means: a write operation simultaneously creates/updates a record in both the old entity and the new entity, keeping them in sync during the transition window. It's typically used when you can't afford a hard cutover and need both systems live simultaneously.

We don't need it here because:

1. The old entities are frozen during the migration window (via the maintenance flag — see 8.2). No new writes hit the old entities after Phase 2 begins.
2. The new entities are not live until Phase 5. agentScopedQuery doesn't route to them until Phase 5. The frontend doesn't query them until Phase 6. There's no consumer that would benefit from dual-write during the gap.
3. Dual-write introduces more risk than it eliminates — if a write to the new entity succeeds but the old entity write fails (or vice versa), you have a split-brain state that's harder to reason about than a clean freeze.

The clean pattern is: **freeze → copy → rewrite → verify → switch.** No overlap, no sync.

### 8.2 The Maintenance Flag

The maintenance flag is an AdminSettings record:

```js
// Written at start of Phase 2 (before any copy steps)
await base44.asServiceRole.entities.AdminSettings.create({
  key: "jobs_migration:maintenance_mode",
  value: JSON.stringify({
    active: true,
    started_at: new Date().toISOString(),
    message: "Your Jobs workspace is being updated. It will be available again shortly. Your data is safe.",
    started_by: "jobs-migration-script"
  })
});

// Deleted at end of Phase 6 (after smoke test passes)
const flag = (await base44.asServiceRole.entities.AdminSettings.list())
  .find(s => s.key === "jobs_migration:maintenance_mode");
if (flag) await base44.asServiceRole.entities.AdminSettings.delete(flag.id);
```

**Rollback:** Delete the record — the flag's absence means maintenance mode is off. If migration aborts at any point before Phase 6 completes, the flag is deleted as part of the rollback sequence and the old entities are still intact.

### 8.3 agentScopedWrite During Each Phase

agentScopedWrite is the enforcement layer. It reads the maintenance flag and blocks FS writes during the window:

```js
// At the top of agentScopedWrite, before any write logic:
const settings = await base44.asServiceRole.entities.AdminSettings.list();
const maintenanceFlag = settings.find(s => s.key === "jobs_migration:maintenance_mode");
if (maintenanceFlag) {
  const config = JSON.parse(maintenanceFlag.value);
  if (config.active) {
    // Parse the workspace from the request
    const workspace = body.workspace || "";
    // Only block FS/Jobs workspace writes — don't affect other workspaces
    if (workspace === "field-service" || workspace === "jobs") {
      return {
        success: false,
        maintenance: true,
        message: config.message
      };
    }
  }
}
// ... rest of write logic
```

This gate:

- **Blocks new writes to old FS entities** — any write call with `workspace: "field-service"` is rejected
- **Doesn't affect Finance, Team, Property** — other workspaces are unaffected
- **Returns a clean, user-readable message** — the frontend banner can display this
- **Is programmatically controlled** — set and cleared by the migration script, not manual dashboard changes

### 8.4 agentScopedQuery State at Each Phase

| Phase | agentScopedQuery State | What Bari sees |
|---|---|---|
| Phase 0 (pre-migration) | Normal — routes `"field-service"` → FS entities | Normal workspace data |
| Phase 1 (schema creation) | Unchanged | Normal |
| Phase 2 (copy records) | Unchanged — still routes to old entities | Normal reads. Writes blocked by agentScopedWrite maintenance gate. |
| Phase 3 (FK rewrite) | Unchanged | Normal reads. Writes still blocked. |
| Phase 4 (verification) | Unchanged | Normal reads. Writes still blocked. |
| Phase 5 (function cutover) | Updated — routes `"jobs"` → Job entities. Shim: `"field-service"` also → Job entities. Old FS entries still in map (reads still work on old entities if called directly). | Writes now route to Job entities. Maintenance flag still active — writes still blocked at agentScopedWrite layer until Phase 6. |
| Phase 6 (code cutover) | Shim still active | Code now queries `"jobs"`. Portal lookup updated. Smoke test passes. Maintenance flag deleted. Writes re-enabled. |
| Phase 7 (cleanup) | Shim removed. Old FS entries removed from ENTITY_FK map. | Jobs workspace fully live. Old entities inaccessible via agent. |

### 8.5 Frontend Maintenance Banner

The frontend reads the maintenance flag on workspace load. If the flag is present and `active: true`, it renders a read-only banner and disables all write-capable buttons (new estimate, new client, new job, etc.):

```js
// In the FieldService/Jobs workspace root component, on mount:
const checkMaintenanceMode = async () => {
  const settings = await AdminSettings.filter({ key: "jobs_migration:maintenance_mode" });
  if (settings.length > 0) {
    const config = JSON.parse(settings[0].value);
    setMaintenanceMode(config.active ? config.message : null);
  }
};
```

When `maintenanceMode` is set:

- Banner renders at top of workspace: the message string from the flag
- All write buttons (`disabled={!!maintenanceMode}`)
- All data still readable and visible — Bari can see his estimates, clients, documents

This is read-permitted, write-blocked — exactly the right user experience during the window.

### 8.6 The Full State Machine

```
PRE-MIGRATION
  agentScopedWrite: routes "field-service" → FS entities, no maintenance gate
  agentScopedQuery: routes "field-service" → FS entities
  frontend: reads/writes FS entities directly
  portal lookup: queries FSEstimate, FSDocument

PHASE 2 START ──── maintenance flag written
  agentScopedWrite: blocks "field-service" writes, returns maintenance message
  agentScopedQuery: unchanged (reads still work)
  frontend: shows banner, write buttons disabled
  portal lookup: unchanged (still queries FSEstimate, FSDocument — reads only, no writes)

PHASE 2–4 ──── copy + rewrite + verify running
  Same as above. Old entities read-live, Job entities being built in background.
  No user-visible change except banner.

PHASE 5 ──── agentScopedQuery + agentScopedWrite updated
  agentScopedQuery:
    - "jobs" → Job entities (new)
    - "field-service" → Job entities (shim)
    - Old FS* entries still in ENTITY_FK for direct-entity reads
  agentScopedWrite:
    - Maintenance gate still active (blocks writes)
    - Entity name routing updated to Job* entities
    - When flag is eventually cleared, writes will route to Job entities
  portal lookup: still unchanged (still queries old entities — reads fine, old entities still live)

PHASE 6 ──── code deploy
  Step 6.1–6.4: frontend now imports Job* entities, routes to /jobs
  Step 6.5: portal lookup now queries JobEstimate, JobDocument
  Step 6.6: agent instructions updated
  Step 6.7: deploy + smoke test
  ───── smoke test passes ─────
  Maintenance flag deleted
  agentScopedWrite: maintenance gate clears, writes now go to Job entities
  frontend: banner gone, write buttons enabled
  portal lookup: resolves from JobEstimate, JobDocument

PHASE 7 ──── shim removal
  agentScopedQuery: shim removed, old FS* entries removed from map
  "field-service" workspace string no longer resolves
  Old entities: readable via direct service-role access, not via agentScopedQuery

30-DAY CLEANUP
  Old FS* entity schemas deleted
```

---

## Section 9 — Code-Side Cutover Sequence

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28
**Note on file paths:** The LocalLane app source code is not accessible from Mycelia's sandbox. File paths below are best-inference. Hyphae must verify each path before executing.

### 9.1 Commit Strategy

All Phase 6 changes ship in one coordinated deploy, but they can be organized as two commits within that deploy:

**Commit A — Entity layer** (can be staged ahead of time, not merged until deploy day):
- All frontend entity import renames (FS* → Job*)
- WORKSPACE_TYPES constant update
- Any component/hook logic that keys off workspace type strings

**Commit B — Portal + routes + agent** (must be same deploy as Commit A):
- Portal lookup function update (Step 6.5)
- Route renames
- Component file renames
- Agent instruction updates (dashboard — not a commit)

**Why the split matters:** Commit A is pure refactor — it changes import names and string constants with no behavioral change if the old entities still exist and agentScopedQuery shim is active. Commit B contains the portal change, which has the timing constraint. Both must ship in the same deploy, but staging A ahead of time makes the deploy-day diff smaller and the review faster.

**What must be atomic (same merge/deploy):** Steps 6.1 through 6.7. Specifically, Step 6.5 (portal lookup) cannot go live without Steps 6.1–6.4 (entity imports) because the portal lookup function likely shares imports with the rest of the workspace code.

### 9.2 Step-by-Step Phase 6 Sequence

**Step 6.1 — Frontend Entity Import Renames**

Every file that imports an FS entity must be updated. Hyphae: search for these import patterns:

```js
// Patterns to find and replace:
import { FieldServiceProfile } from '@/api/entities'  →  import { JobProfile }
import { FSClient }             from '@/api/entities'  →  import { JobClient }
import { FSProject }            from '@/api/entities'  →  import { Job }
import { FSEstimate }           from '@/api/entities'  →  import { JobEstimate }
import { FSDocument }           from '@/api/entities'  →  import { JobDocument }
import { FSDailyLog }           from '@/api/entities'  →  import { JobLog }
import { FSMaterialEntry }      from '@/api/entities'  →  import { JobMaterial }
import { FSLaborEntry }         from '@/api/entities'  →  import { JobLabor }
import { FSDailyPhoto }         from '@/api/entities'  →  import { JobPhoto }
import { FSPayment }            from '@/api/entities'  →  import { JobPayment }
import { FSPermit }             from '@/api/entities'  →  import { JobPermit }
import { FSChangeOrder }        from '@/api/entities'  →  import { JobChangeOrder }
```

Likely affected file locations (Hyphae: verify against actual project structure):

```
src/pages/FieldService/          → src/pages/Jobs/
src/components/fieldservice/     → src/components/jobs/
src/hooks/useFieldService*.js    → src/hooks/useJob*.js
src/api/entities.js              (entity export list — must include new Job* names)
```

**Variable rename scope:** Every in-file variable name derived from the import also changes:

```js
// Before
const profiles = await FieldServiceProfile.list();
const clients  = await FSClient.filter({ workspace_id: profileId });

// After
const profiles = await JobProfile.list();
const clients  = await JobClient.filter({ workspace_id: profileId });
// Note: workspace_id field name preserved (Path A)
```

**Step 6.2 — WORKSPACE_TYPES Constant**

File: Likely `src/constants/workspaceTypes.js`, `src/config/spaces.js`, or similar. Hyphae: search for `"field-service"` string literal in non-test, non-spec files.

```js
// Before
export const WORKSPACE_TYPES = {
  FIELD_SERVICE: "field-service",
  FINANCE: "finance",
  // ...
};

// After
export const WORKSPACE_TYPES = {
  JOBS: "jobs",
  FINANCE: "finance",
  // ...
};
```

Every reference to `WORKSPACE_TYPES.FIELD_SERVICE` (or inline `"field-service"` strings) in component code must be updated to `WORKSPACE_TYPES.JOBS` (or `"jobs"`).

**Step 6.3 — Component File Renames**

Files to rename (verify paths):

```
JoinFieldService.jsx      →  JoinJobs.jsx
FieldServiceDashboard.jsx →  JobsDashboard.jsx  (or similar)
FieldServiceSetup.jsx     →  JobsSetup.jsx
```

All internal references within these files update simultaneously with the rename. Any other component that imports them by path must update the import path.

**Step 6.4 — Route Renames**

File: Router config — likely `src/App.jsx`, `src/router.jsx`, or `src/routes/index.js`.

```js
// Before
{ path: "/field-service",       component: FieldServiceDashboard }
{ path: "/join/field-service",  component: JoinFieldService }
{ path: "/desk",                component: FieldServiceDashboard }  // if "desk" is a route

// After
{ path: "/jobs",                component: JobsDashboard }
{ path: "/join/jobs",           component: JoinJobs }

// Add redirect rules for old paths (30-day minimum):
{ path: "/field-service",       redirect: "/jobs" }
{ path: "/join/field-service",  redirect: "/join/jobs" }
{ path: "/desk",                redirect: "/jobs" }  // if applicable
```

**Redirect requirement:** Old routes must 301-redirect to new routes for the 30-day post-migration window. Any bookmark, email link, or shared URL using the old path still works. Remove redirects during the 30-day cleanup pass.

**Step 6.5 — Portal Lookup Update ⚠ Timing-Critical**

What this step changes: The function/route that resolves portal tokens for public-facing estimate and document links.

File: Likely a server function (`portalLookup`, `estimatePortal`, or similar) and/or a frontend route handler for `/portal/:token`.

```js
// Before — queries old entities
async function resolvePortalToken(token) {
  const estimates = await FSEstimate.filter({ portal_token: token });
  if (estimates.length > 0) return { type: "estimate", record: estimates[0] };

  const docs = await FSDocument.filter({ portal_token: token });
  if (docs.length > 0) return { type: "document", record: docs[0] };

  return null;
}

// After — queries new entities
async function resolvePortalToken(token) {
  const estimates = await JobEstimate.filter({ portal_token: token });
  if (estimates.length > 0) return { type: "estimate", record: estimates[0] };

  const docs = await JobDocument.filter({ portal_token: token });
  if (docs.length > 0) return { type: "document", record: docs[0] };

  return null;
}
```

**Why this step cannot ship before Steps 6.1–6.4:** If the portal lookup is a server function that uses the SDK's entity imports, it shares the same entity name namespace. More importantly, there is no gap where querying the old entity fails — the old entities still exist and still have the tokens. The only thing that changes is which entity the lookup targets.

**The safe ordering within Phase 6:** Steps 6.1–6.7 ship as a single deploy. Within that deploy, Step 6.5 can be part of Commit A or Commit B — it doesn't matter, because the old entities remain readable throughout. What matters is that Step 6.5 ships before Phase 7 (archival). The gate is: smoke test Step 6.5 explicitly before proceeding to Phase 7.

**Smoke test for Step 6.5:**

1. Load: `/portal/estimate/ea14972d-3978-4381-860b-849f4b4d3694`
   - Expected: Dr. Holman's estimate renders correctly from JobEstimate
2. Load: `/portal/document/77f38acf-cd5c-4a1e-af63-c94f575952b3`
   - Expected: Subcontractor Agreement renders correctly from JobDocument
3. Load: `/portal/document/4c11ee24-6427-4105-a3e9-1a99686d9c8f`
   - Expected: Notice of Right to Lien renders correctly from JobDocument

All three must pass before Phase 7 proceeds.

**Step 6.6 — Agent Instruction Updates (Dashboard)**

This is a Base44 dashboard change, not a code commit.

- Agent to update: FieldServiceAgent (rename to "Jobs Agent" or "JobsAgent" — confirm display name with Doron)
- Changes in the instruction text:
  - `workspace: "field-service"` → `workspace: "jobs"` everywhere it appears
  - All entity name references: `FSClient` → `JobClient`, `FSEstimate` → `JobEstimate`, etc.
  - Any hardcoded profile IDs or workspace strings in examples within the instructions
- Timing: Do this in the same session as the code deploy — not before (agent instructions referencing `"jobs"` before Phase 5 is live will fail), not after (agent should be updated before Bari's workspace comes back online).

**Step 6.7 — Deploy + Smoke Test Checklist**

```
PRE-DEPLOY GATES (must all pass before merging):
  [ ] Phase 4 verification passed (count parity, Bari's records verified)
  [ ] Phase 4 enabled_spaces backfill confirmed (0 records with "desk")
  [ ] Phase 5 agentScopedQuery deployed and confirmed returning Job* data
  [ ] Phase 5 agentScopedWrite deployed and confirmed maintenance gate active
  [ ] Maintenance flag is active (writes still blocked going into deploy)

DEPLOY:
  [ ] Merge Commit A (entity imports, WORKSPACE_TYPES)
  [ ] Merge Commit B (portal lookup, routes, component renames)
  [ ] Base44 app publish

POST-DEPLOY SMOKE TEST (in order):
  [ ] /jobs route loads without error
  [ ] Bari's workspace: profile visible
  [ ] Bari's workspace: clients visible (Test Client, Doron Fletcher)
  [ ] Bari's workspace: documents visible (Notice of Right to Lien)
  [ ] Red Umbrella workspace: estimate visible ($121,657.57 Dr. Nathan Holman)
  [ ] Red Umbrella workspace: project visible (Dr. Nathan Holman barn)
  [ ] Portal token 1: ea14972d resolves from JobEstimate ✓
  [ ] Portal token 2: 77f38acf resolves from JobDocument ✓
  [ ] Portal token 3: 4c11ee24 resolves from JobDocument ✓
  [ ] agentScopedQuery("jobs", "JobClient") returns Bari's clients only
  [ ] agentScopedQuery("jobs", "JobClient") does NOT return Red Umbrella's clients
  [ ] Create new test JobClient record → appears in workspace → delete it
  [ ] /join/field-service redirects to /join/jobs

ALL GATES PASS:
  [ ] Delete maintenance flag (AdminSettings record)
  [ ] Write buttons re-enabled (frontend reads flag absence)
  [ ] Agent instructions updated in dashboard (Step 6.6)
  [ ] Confirm Bari's workspace fully operable post-flag deletion
```

### 9.3 Minimum Time Estimate for Phase 6

With the current data volume (10 records total), Phases 2–4 complete in under 5 minutes. Phase 5 (server function deploys) adds ~5 minutes. Phase 6 code deploy + smoke test: 15–20 minutes. Total maintenance window: under 30 minutes. Doron is the only user, so scheduling is whenever he's rested and focused.

---

## Section 10 — Verification Queries

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28
**Note:** All queries run via `base44.asServiceRole` in the jobsMigration server function. All expected values sourced from live pre-migration data as of this date.

### 10.1 Count Parity — All 12 Entity Pairs

Run after Phase 2 (Pass 1 complete) and again after Phase 3 (Pass 2 complete). Both runs should show identical counts — FK rewrites don't add or delete records.

```js
async function verifyCountParity() {
  const PAIRS = [
    ["FieldServiceProfile", "JobProfile"],
    ["FSClient",            "JobClient"],
    ["FSProject",           "Job"],
    ["FSEstimate",          "JobEstimate"],
    ["FSDocument",          "JobDocument"],
    ["FSDailyLog",          "JobLog"],
    ["FSMaterialEntry",     "JobMaterial"],
    ["FSLaborEntry",        "JobLabor"],
    ["FSDailyPhoto",        "JobPhoto"],
    ["FSPayment",           "JobPayment"],
    ["FSPermit",            "JobPermit"],
    ["FSChangeOrder",       "JobChangeOrder"],
  ];

  const results = [];
  for (const [oldName, newName] of PAIRS) {
    const oldRecords = await base44.asServiceRole.entities[oldName].list();
    const newRecords = await base44.asServiceRole.entities[newName].list();
    // Filter new records to only migrated ones (legacy_id present)
    const migratedCount = newRecords.filter(r => r.legacy_id).length;
    results.push({
      old_entity: oldName,
      new_entity: newName,
      old_count: oldRecords.length,
      migrated_count: migratedCount,
      pass: oldRecords.length === migratedCount
    });
  }
  return results;
}
```

**Expected output:**

| Old Entity | New Entity | Expected Count | Pass |
|---|---|---|---|
| FieldServiceProfile | JobProfile | 2 | ✓ |
| FSClient | JobClient | 3 | ✓ |
| FSProject | Job | 1 | ✓ |
| FSEstimate | JobEstimate | 1 | ✓ |
| FSDocument | JobDocument | 3 | ✓ |
| FSDailyLog | JobLog | 0 | ✓ |
| FSMaterialEntry | JobMaterial | 0 | ✓ |
| FSLaborEntry | JobLabor | 0 | ✓ |
| FSDailyPhoto | JobPhoto | 0 | ✓ |
| FSPayment | JobPayment | 0 | ✓ |
| FSPermit | JobPermit | 0 | ✓ |
| FSChangeOrder | JobChangeOrder | 0 | ✓ |

**Total migrated records: 10**

Any row where `pass: false` is a hard stop. Do not proceed to Phase 3.

**Dry-run vs production shape:** In dry-run mode, this query runs against the old entities only and reports what the counts will be. The new entity counts will all be 0 because dry-run doesn't write. The report format changes to `would_create: N` rather than `migrated_count: N`.

### 10.2 Bari's Profile Verification

Pre-migration known values:
- FieldServiceProfile.id: `69baba55a6b9cca0c7d5700b`
- user_id: `69b9f8313b41521cc50c97d8`
- workspace_name: "Red Umbrella"
- business_name: "Red Umbrella"
- owner_name: "Bari Swartz"
- business_id: `69ea5590481b7e15af7216b6`
- subscription_tier: "full"
- invite_code: "TDAqwuxj"
- archived_at: null

```js
async function verifyBariProfile() {
  const profiles = await base44.asServiceRole.entities.JobProfile.list();
  const bari = profiles.find(p => p.legacy_id === "69baba55a6b9cca0c7d5700b");

  if (!bari) return { pass: false, error: "Bari's JobProfile not found by legacy_id" };

  const checks = [
    { field: "user_id",          expected: "69b9f8313b41521cc50c97d8",  actual: bari.user_id },
    { field: "workspace_name",   expected: "Red Umbrella",               actual: bari.workspace_name },
    { field: "business_name",    expected: "Red Umbrella",               actual: bari.business_name },
    { field: "owner_name",       expected: "Bari Swartz",                actual: bari.owner_name },
    { field: "business_id",      expected: "69ea5590481b7e15af7216b6",   actual: bari.business_id },
    { field: "subscription_tier",expected: "full",                       actual: bari.subscription_tier },
    { field: "invite_code",      expected: "TDAqwuxj",                   actual: bari.invite_code },
    { field: "archived_at",      expected: null,                         actual: bari.archived_at },
  ];

  const results = checks.map(c => ({ ...c, pass: c.actual === c.expected }));
  const allPass = results.every(r => r.pass);

  return {
    new_id: bari.id,
    legacy_id: bari.legacy_id,
    overall: allPass ? "PASS" : "FAIL",
    checks: results
  };
}
```

**Critical check:** `business_id === "69ea5590481b7e15af7216b6"`. This is the link to Red Umbrella's Business record, and to the `enabled_spaces` backfill in Phase 4. If this field is null or wrong after migration, Bari's business connection is broken.

### 10.3 The Holman Barn Project — FK Resolution Verification

Pre-migration known values:
- FSProject.id: `69ebaa59a3b3dcbc375a7ca9`
- profile_id (old): `69baba55a6b9cca0c7d5700b`
- client_id (old): `69babaedda46b4942a762790`
- estimate_id (old): `69c6fba24b8b8812fe4f5419`
- name: "Dr. Nathan Holman   " (trailing spaces are verbatim — copy exactly)
- status: "active"
- total_budget: 121657.57

```js
async function verifyHolmanJob() {
  const jobs = await base44.asServiceRole.entities.Job.list();
  const job = jobs.find(j => j.legacy_id === "69ebaa59a3b3dcbc375a7ca9");

  if (!job) return { pass: false, error: "Holman Job not found by legacy_id" };

  // Resolve FKs by looking up new IDs
  const profiles  = await base44.asServiceRole.entities.JobProfile.list();
  const clients   = await base44.asServiceRole.entities.JobClient.list();
  const estimates = await base44.asServiceRole.entities.JobEstimate.list();

  const resolvedProfile  = profiles.find(p => p.id === job.profile_id);
  const resolvedClient   = clients.find(c => c.id === job.client_id);
  const resolvedEstimate = estimates.find(e => e.id === job.estimate_id);

  return {
    job_new_id: job.id,
    legacy_id: job.legacy_id,
    checks: [
      { check: "name verbatim",        pass: job.name === "Dr. Nathan Holman   " },
      { check: "status",               pass: job.status === "active" },
      { check: "total_budget",         pass: job.total_budget === 121657.57 },
      { check: "profile_id resolves",  pass: !!resolvedProfile,
        profile_legacy_id: resolvedProfile?.legacy_id },
      { check: "client_id resolves",   pass: !!resolvedClient,
        client_legacy_id: resolvedClient?.legacy_id },
      { check: "estimate_id resolves", pass: !!resolvedEstimate,
        estimate_legacy_id: resolvedEstimate?.legacy_id },
      // Verify the FKs point to the RIGHT records, not just any records
      { check: "profile_id → Bari's profile",
        pass: resolvedProfile?.legacy_id === "69baba55a6b9cca0c7d5700b" },
      { check: "client_id → Dr. Holman",
        pass: resolvedClient?.legacy_id === "69babaedda46b4942a762790" },
      { check: "estimate_id → $121K estimate",
        pass: resolvedEstimate?.legacy_id === "69c6fba24b8b8812fe4f5419" },
    ]
  };
}
```

### 10.4 The $121,657.57 Estimate — High-Stakes Verification

Pre-migration known values:
- FSEstimate.id: `69c6fba24b8b8812fe4f5419`
- portal_token: `ea14972d-3978-4381-860b-849f4b4d3694`
- portal_link_active: true
- status: "accepted"
- total: 121657.57
- responded_at: `2026-04-24T17:37:30.168Z`
- signed_at: null
- owner_signature_data: null
- estimate_number: "EST-2026-004"
- prepared_by: "Bari Swartz"

```js
async function verifyHolmanEstimate() {
  const estimates = await base44.asServiceRole.entities.JobEstimate.list();
  const est = estimates.find(e => e.legacy_id === "69c6fba24b8b8812fe4f5419");

  if (!est) return { pass: false, error: "Holman estimate not found by legacy_id" };

  // Portal token check — most critical single assertion in the migration
  const portalTokenIntact = est.portal_token === "ea14972d-3978-4381-860b-849f4b4d3694";

  // Timestamp precision check — must be exact string match, not just date match
  const respondedAtIntact = est.responded_at === "2026-04-24T17:37:30.168Z";

  // Line items integrity — check count and total, not full blob comparison
  const lineItemsCount = est.line_items?.items?.length;

  return {
    new_id: est.id,
    legacy_id: est.legacy_id,
    checks: [
      { check: "portal_token verbatim",   pass: portalTokenIntact,
        expected: "ea14972d-3978-4381-860b-849f4b4d3694", actual: est.portal_token },
      { check: "portal_link_active",      pass: est.portal_link_active === true },
      { check: "status = accepted",       pass: est.status === "accepted" },
      { check: "total = 121657.57",       pass: est.total === 121657.57 },
      { check: "responded_at verbatim",   pass: respondedAtIntact,
        expected: "2026-04-24T17:37:30.168Z", actual: est.responded_at },
      { check: "signed_at = null",        pass: est.signed_at === null },
      { check: "owner_signature_data = null", pass: est.owner_signature_data === null },
      { check: "estimate_number",         pass: est.estimate_number === "EST-2026-004" },
      { check: "prepared_by",             pass: est.prepared_by === "Bari Swartz" },
      { check: "line_items count = 30",   pass: lineItemsCount === 30,
        actual_count: lineItemsCount },
      { check: "subtotal = 121657.57",    pass: est.subtotal === 121657.57 },
    ]
  };
}
```

### 10.5 The Three FSDocument Records

Pre-migration known values:

| id | title | status | portal_token | signed_at | profile_id |
|---|---|---|---|---|---|
| `69c6fa2731cdee2982627f4a` | Subcontractor Agreement | draft | null | null | `69baba55a6b9cca0c7d5700b` |
| `69c6cfb440fa5e43db334f1c` | Subcontractor Agreement | awaiting_signature | `77f38acf-cd5c-4a1e-af63-c94f575952b3` | null | `69baba55a6b9cca0c7d5700b` |
| `69c698c2214ef0d74eca1336` | Notice of Right to Lien | signed | `4c11ee24-6427-4105-a3e9-1a99686d9c8f` | `2026-03-27T21:05:53.623Z` | `69b6c265746397cbf8e184f3` |

```js
async function verifyDocuments() {
  const docs = await base44.asServiceRole.entities.JobDocument.list();

  const DOC_EXPECTED = [
    {
      legacy_id: "69c6fa2731cdee2982627f4a",
      title: "Subcontractor Agreement",
      status: "draft",
      portal_token: null,
      signed_at: null,
      profile_legacy_id: "69baba55a6b9cca0c7d5700b",
      client_legacy_id: "69babaedda46b4942a762790",
      recalled: false,
      amendment_of: null,
    },
    {
      legacy_id: "69c6cfb440fa5e43db334f1c",
      title: "Subcontractor Agreement",
      status: "awaiting_signature",
      portal_token: "77f38acf-cd5c-4a1e-af63-c94f575952b3",
      signed_at: null,
      profile_legacy_id: "69baba55a6b9cca0c7d5700b",
      client_legacy_id: "69babaedda46b4942a762790",
      recalled: false,
      amendment_of: null,
    },
    {
      legacy_id: "69c698c2214ef0d74eca1336",
      title: "Notice of Right to Lien",
      status: "signed",
      portal_token: "4c11ee24-6427-4105-a3e9-1a99686d9c8f",
      signed_at: "2026-03-27T21:05:53.623Z",
      profile_legacy_id: "69b6c265746397cbf8e184f3",
      client_legacy_id: "69c6989f867cf1831532c671",
      recalled: false,
      amendment_of: null,
    },
  ];

  const profiles = await base44.asServiceRole.entities.JobProfile.list();
  const clients  = await base44.asServiceRole.entities.JobClient.list();

  return DOC_EXPECTED.map(expected => {
    const doc = docs.find(d => d.legacy_id === expected.legacy_id);
    if (!doc) return { legacy_id: expected.legacy_id, pass: false, error: "not found" };

    const resolvedProfile = profiles.find(p => p.id === doc.profile_id);
    const resolvedClient  = clients.find(c => c.id === doc.client_id);

    // Signature data: don't compare full blob — check it's present (non-null) 
    // for signed docs and null for unsigned ones
    const signatureDataCheck = expected.status === "signed"
      ? doc.owner_signature_data !== null
      : true; // unsigned docs: any value (null or populated) is acceptable

    // Content blob: check it's non-null and non-empty
    const contentIntact = doc.content && doc.content.length > 0;

    return {
      legacy_id: expected.legacy_id,
      title: expected.title,
      checks: [
        { check: "title",                 pass: doc.title === expected.title },
        { check: "status",                pass: doc.status === expected.status },
        { check: "portal_token",          pass: doc.portal_token === expected.portal_token,
          expected: expected.portal_token, actual: doc.portal_token },
        { check: "signed_at",             pass: doc.signed_at === expected.signed_at },
        { check: "recalled",              pass: doc.recalled === expected.recalled },
        { check: "amendment_of",          pass: doc.amendment_of === expected.amendment_of },
        { check: "profile_id resolves",   pass: !!resolvedProfile },
        { check: "profile → correct workspace",
          pass: resolvedProfile?.legacy_id === expected.profile_legacy_id },
        { check: "client_id resolves",    pass: !!resolvedClient },
        { check: "client → correct record",
          pass: resolvedClient?.legacy_id === expected.client_legacy_id },
        { check: "signature_data present (signed docs)",
          pass: signatureDataCheck },
        { check: "content blob intact",   pass: contentIntact },
      ]
    };
  });
}
```

### 10.6 FK Integrity Spot-Check — All Entities with Live Records

For entities with live records (JobProfile, JobClient, Job, JobEstimate, JobDocument), walk every FK on every record and verify it resolves:

```js
async function fkIntegritySpotCheck() {
  // Build lookup sets for all FK target entities
  const profileIds  = new Set((await base44.asServiceRole.entities.JobProfile.list()).map(r => r.id));
  const clientIds   = new Set((await base44.asServiceRole.entities.JobClient.list()).map(r => r.id));
  const jobIds      = new Set((await base44.asServiceRole.entities.Job.list()).map(r => r.id));
  const estimateIds = new Set((await base44.asServiceRole.entities.JobEstimate.list()).map(r => r.id));
  const docIds      = new Set((await base44.asServiceRole.entities.JobDocument.list()).map(r => r.id));

  const failures = [];

  // JobClient.workspace_id → JobProfile
  for (const r of await base44.asServiceRole.entities.JobClient.list()) {
    if (!profileIds.has(r.workspace_id))
      failures.push({ entity: "JobClient", id: r.id, field: "workspace_id", value: r.workspace_id });
  }

  // Job.profile_id → JobProfile
  // Job.client_id → JobClient
  // Job.estimate_id → JobEstimate
  for (const r of await base44.asServiceRole.entities.Job.list()) {
    if (!profileIds.has(r.profile_id))
      failures.push({ entity: "Job", id: r.id, field: "profile_id", value: r.profile_id });
    if (r.client_id && !clientIds.has(r.client_id))
      failures.push({ entity: "Job", id: r.id, field: "client_id", value: r.client_id });
    if (r.estimate_id && !estimateIds.has(r.estimate_id))
      failures.push({ entity: "Job", id: r.id, field: "estimate_id", value: r.estimate_id });
  }

  // JobEstimate.profile_id → JobProfile
  // JobEstimate.job_id → Job
  // JobEstimate.client_id → JobClient
  for (const r of await base44.asServiceRole.entities.JobEstimate.list()) {
    if (!profileIds.has(r.profile_id))
      failures.push({ entity: "JobEstimate", id: r.id, field: "profile_id", value: r.profile_id });
    if (r.job_id && !jobIds.has(r.job_id))
      failures.push({ entity: "JobEstimate", id: r.id, field: "job_id", value: r.job_id });
    if (r.client_id && !clientIds.has(r.client_id))
      failures.push({ entity: "JobEstimate", id: r.id, field: "client_id", value: r.client_id });
  }

  // JobDocument.profile_id, .job_id, .client_id, .amendment_of
  for (const r of await base44.asServiceRole.entities.JobDocument.list()) {
    if (!profileIds.has(r.profile_id))
      failures.push({ entity: "JobDocument", id: r.id, field: "profile_id", value: r.profile_id });
    if (r.job_id && !jobIds.has(r.job_id))
      failures.push({ entity: "JobDocument", id: r.id, field: "job_id", value: r.job_id });
    if (r.client_id && !clientIds.has(r.client_id))
      failures.push({ entity: "JobDocument", id: r.id, field: "client_id", value: r.client_id });
    if (r.amendment_of && !docIds.has(r.amendment_of))
      failures.push({ entity: "JobDocument", id: r.id, field: "amendment_of", value: r.amendment_of });
  }

  return {
    pass: failures.length === 0,
    failure_count: failures.length,
    failures   // empty array on clean run
  };
}
```

**Expected:** `failures: []`, `pass: true`. Any failure here means Pass 2 FK rewrite did not complete for that record.

### 10.7 Portal Token End-to-End Verification

This runs after Phase 6 (portal lookup function updated). It is not automatable from the migration script — it requires a browser or HTTP client.

**Before-migration baseline** (capture before Phase 2 starts, or confirm from live data above):

| Token | Entity | Title | Status | Total / Key field |
|---|---|---|---|---|
| `ea14972d-3978-4381-860b-849f4b4d3694` | FSEstimate | "Dr. Nathan Holman   " | accepted | $121,657.57 |
| `77f38acf-cd5c-4a1e-af63-c94f575952b3` | FSDocument | "Subcontractor Agreement" | awaiting_signature | — |
| `4c11ee24-6427-4105-a3e9-1a99686d9c8f` | FSDocument | "Notice of Right to Lien" | signed | signed_at: `2026-03-27T21:05:53.623Z` |

**Post-migration test sequence:**

```
TEST 1: Estimate portal
  Request: GET /portal/estimate/ea14972d-3978-4381-860b-849f4b4d3694
           (or however the portal URL is structured)
  Must match:
    [ ] Renders without error (200, not 404)
    [ ] Title displayed: "Dr. Nathan Holman" (trimmed)
    [ ] Total displayed: $121,657.57
    [ ] Status indicator: accepted / signed
    [ ] Line items render: 30 items visible
    [ ] Contractor name: "Bari Swartz" (from prepared_by)
    [ ] Portal link still active (no "this link has expired" message)

TEST 2: Document portal — awaiting signature
  Request: GET /portal/document/77f38acf-cd5c-4a1e-af63-c94f575952b3
  Must match:
    [ ] Renders without error
    [ ] Title: "Subcontractor Agreement"
    [ ] Status: awaiting signature (not "already signed", not 404)
    [ ] Signature capture UI is present (not blocked)
    [ ] Contractor: Red Umbrella

TEST 3: Document portal — signed
  Request: GET /portal/document/4c11ee24-6427-4105-a3e9-1a99686d9c8f
  Must match:
    [ ] Renders without error
    [ ] Title: "Notice of Right to Lien"
    [ ] Status: signed
    [ ] signed_at timestamp visible: 2026-03-27
    [ ] Signature data renders (the drawn signature image is present)
    [ ] Content body matches pre-migration document text
```

**Failure mode:** Any 404 on a portal token means the lookup function is still querying the old entity and the old entity is no longer returning that record — or the token wasn't copied correctly. Check JobEstimate / JobDocument for the token value before diagnosing further.

**Dry-run vs production:** There is no dry-run equivalent for this test. Portal verification only runs post-Phase-6. The Phase 2 Step 2.4 portal_token assertion (from Section 6) is the pre-cutover proxy.

### 10.8 Maintenance Flag Deletion Verification

```js
async function verifyMaintenanceFlagGone() {
  const settings = await base44.asServiceRole.entities.AdminSettings.list();
  const flag = settings.find(s => s.key === "jobs_migration:maintenance_mode");
  return {
    pass: !flag,
    flag_found: !!flag,
    flag_id: flag?.id ?? null
  };
}
```

**Expected:** `pass: true`, `flag_found: false`. If the flag persists after Phase 6, the frontend will still show the maintenance banner and write buttons will remain disabled. This is a blocking issue — delete the record manually if the migration script failed to clean it up.

### 10.9 Negative Checks

```js
async function negativeChecks() {
  const NEW_ENTITIES = [
    "JobProfile", "JobClient", "Job", "JobEstimate", "JobDocument",
    "JobLog", "JobMaterial", "JobLabor", "JobPhoto",
    "JobPayment", "JobPermit", "JobChangeOrder"
  ];

  const results = [];

  for (const entityName of NEW_ENTITIES) {
    const records = await base44.asServiceRole.entities[entityName].list();

    // Check 1: No migrated record is missing legacy_id
    const migratedWithoutLegacyId = records.filter(r =>
      !r.legacy_id && r.created_by === "migration@locallane.internal"
    );

    results.push({
      entity: entityName,
      total_records: records.length,
      migrated_count: records.filter(r => r.legacy_id).length,
      native_count: records.filter(r => !r.legacy_id).length,
      // native_count should be 0 immediately post-migration
      // it will grow as Bari creates new records after cutover
      migrated_missing_legacy_id: migratedWithoutLegacyId.length,
      pass: migratedWithoutLegacyId.length === 0
    });
  }

  return results;
}
```

**Expected immediately post-migration:** `native_count: 0` on all entities (no natively-created records yet), `migrated_missing_legacy_id: 0` on all entities.

**Note on native_count over time:** After Phase 6 cutover and flag deletion, Bari will start creating new records (new clients, new jobs, new estimates). Those will have `legacy_id: null` and `native_count` will grow. This is expected and correct. The negative check only applies immediately after migration — not as an ongoing invariant.

---

## Section 11 — Rollback Procedures

**Status:** Authoritative
**Author:** Mycelia
**Date:** 2026-04-28

### 11.1 The Rollback Gradient

Rollback difficulty is not binary — it's a gradient across seven phases. The key insight: **the old entities are never modified and never deleted until the 30-day cleanup.** This means a full data rollback is always theoretically available. What gets harder over time is the operational rollback — the state of code, server functions, agent instructions, and natively-created records in new entities.

```
PHASE 0-1:  Trivial rollback. Nothing written to live entities.
            Delete new entity schemas. Done.

PHASE 2:    Clean rollback. Old entities untouched.
            Delete migrated records from Job* entities. Delete new schemas.
            Maintenance flag deletion re-enables old workspace.

PHASE 3:    Same as Phase 2. Pass 2 only updates new records.
            Old entities untouched throughout.

PHASE 4:    Mostly clean. One data mutation: enabled_spaces backfill.
            Reverse that one record update, then same as Phase 2.

PHASE 5:    Requires server function rollback. Old entities still intact.
            Redeploy previous agentScopedQuery/Write. Then same as Phase 2.

PHASE 6:    Requires code rollback + server function rollback.
            Old entities still intact. No new native records yet.
            Clean rollback still possible if caught immediately.

POST-PHASE 6 (hours/days later):
            Bari has created new records in Job* entities.
            Those new records don't exist in old FS* entities.
            Full data rollback now requires:
              (a) migrate new Job* records back to FS* entities, OR
              (b) accept that those new records are lost, OR
              (c) patch forward instead of rolling back
            Patch-forward is almost always the right call here.

30-DAY MARK:
            Old FS* entities deleted. Rollback to old entities is no longer possible.
            Any issues must be patched forward.
```

### 11.2 The Point of No Return

**Practical point of no return: Phase 6 completion + Bari creates his first new native record in a Job* entity.**

Before that moment, rollback is clean — delete new records, restore server functions, restore code, old entities intact, everything returns to pre-migration state.

After that moment, rollback requires migrating forward-created records back to old entities. That's a new migration in the opposite direction. The complexity grows with each new native record.

**The 30-day deletion is the hard point of no return.** After old entities are deleted, there is no going back at the data layer.

### 11.3 Phase-by-Phase Rollback Procedures

**Phase 0–1 Failure (Setup or Schema Creation)**

Scenario: jobsMigration function deploy fails, or schema creation step errors on one entity.
State: No records written. Old entities untouched.

Action:
```js
// Delete any partially-created Job* schemas
// In Base44 dashboard: delete each Job* entity that was created
// OR via manage_entity_schemas:
for (const entity of CREATED_SO_FAR) {
  await manage_entity_schemas({ action: "delete", entity_name: entity });
}
// Delete maintenance flag if it was set
const flag = settings.find(s => s.key === "jobs_migration:maintenance_mode");
if (flag) await AdminSettings.delete(flag.id);
```

Exit: Diagnose, fix, re-run from Phase 0. Safe to retry indefinitely.

**Phase 2 Partial Completion**

Scenario: Migration script fails after copying JobProfile and JobClient but before copying Job.
State: JobProfile has 2 records, JobClient has 3 records, Job has 0. Old entities untouched. Maintenance flag active.

Action:
```js
// Step R2.1: Delete all records in partially-populated Job* entities
const ROLLBACK_ORDER = [
  "JobDocument", "JobChangeOrder", "JobPermit", "JobPayment",
  "JobPhoto", "JobLabor", "JobMaterial", "JobLog",
  "JobEstimate", "Job", "JobClient", "JobProfile"
];
for (const entity of ROLLBACK_ORDER) {
  const records = await base44.asServiceRole.entities[entity].list();
  for (const r of records.filter(r => r.legacy_id)) {
    await base44.asServiceRole.entities[entity].delete(r.id);
    await AdminAuditLog.create({
      action: "JOBS_MIGRATION_ROLLBACK_P2",
      entity_type: entity,
      entity_id: r.id,
      details: JSON.stringify({ legacy_id: r.legacy_id, reason: "phase_2_partial_failure" })
    });
  }
}
// Step R2.2: Delete maintenance flag
const flag = settings.find(s => s.key === "jobs_migration:maintenance_mode");
if (flag) await AdminSettings.delete(flag.id);
```

Exit: Rollback order is children-before-parents (reverse of copy order) to avoid any FK confusion during deletion. After deletion, all Job* entities are empty, maintenance flag is gone, old entities serve traffic normally. Diagnose, fix, re-run.

Idempotency: The `legacy_id` filter on deletion means re-running the rollback script finds nothing to delete if it already ran. Safe to retry.

**Phase 3 Partial Completion (Some FKs Rewritten, Some Stale)**

Scenario: FK rewrite completes on JobClient and Job but errors out before JobEstimate.
State: JobProfile/Client/Job have correct FKs, JobEstimate still has stale FKs. Old entities untouched. Maintenance flag active.

Action: Same as Phase 2 rollback — delete all migrated records from all Job* entities. The partial FK rewrites are on new records that are about to be deleted anyway. No old-entity data was touched.

Alternative: If you don't want to re-run Pass 1, you can patch forward: re-run just the FK rewrite step for the entities that failed. The `needsRewrite()` idempotency check in Section 6.4 will skip already-rewritten records and only process stale ones. This is often faster than a full rollback + re-run.

Exit choice: Patch forward if the failure was transient (timeout, rate limit). Full rollback if the failure revealed a logic bug in the rewrite code.

**Phase 4 Verification Failure**

Scenario: Count parity check shows JobEstimate has 0 records but FSEstimate has 1.
State: Some copy step silently failed. Old entities untouched. Maintenance flag active.

Action:
1. Do not proceed to Phase 5.
2. Diagnose: query JobEstimate.list(), check `legacy_id` values. Determine which source records are missing.
3. If recoverable (idempotency means re-running the copy step will fill the gap): re-run that specific step, re-verify.
4. If not recoverable: full Phase 2 rollback, fix the bug, re-run from Phase 0.

Special case — `enabled_spaces` backfill failure:
```js
// Rollback: restore "desk" on affected Business records
// Each AuditLog entry from Step 4.5 has the reversal command:
await base44.asServiceRole.entities.Business.update(
  "69ea5590481b7e15af7216b6",
  { enabled_spaces: ["profile", "settings", "desk"] }
);
await AdminAuditLog.create({
  action: "JOBS_MIGRATION_ROLLBACK_ENABLED_SPACES",
  entity_type: "Business",
  entity_id: "69ea5590481b7e15af7216b6",
  details: JSON.stringify({ restored: ["profile", "settings", "desk"] })
});
```

**Phase 5 Failure (agentScopedQuery Returns Wrong Data)**

Scenario: agentScopedQuery deployed, but returns empty results for `workspace: "jobs"`.
State: New entities populated and FK-correct. Server function deployed with bug. Maintenance flag active. Old entities untouched.

Action:
1. Maintenance flag is still active — writes are still blocked, users see banner. No data exposure.
2. Redeploy previous version of agentScopedQuery.
3. Confirm old behavior restored: `workspace: "field-service"` returns Bari's records correctly.
4. Fix the function bug. Redeploy. Re-run smoke test from Section 8.4.

Note: The maintenance flag is your friend here. Because it's still active, no new writes have gone to either old or new entities during the Phase 5 window. You can take as long as needed to fix the function without data divergence.

**Phase 6 Failure — One Portal Token Returns 404**

Scenario: Code deployed. Smoke test passes for estimate and one document, but `77f38acf` (Subcontractor Agreement, awaiting_signature) returns 404.
State: Code live. New entities serving most traffic. Portal lookup partially broken for one token.

Diagnosis first:
```js
// In production console or via migration function:
const docs = await base44.asServiceRole.entities.JobDocument.list();
const doc = docs.find(d => d.portal_token === "77f38acf-cd5c-4a1e-af63-c94f575952b3");
// If null: token was not copied correctly in Phase 2 Step 2.5
// If found: portal lookup code has a bug (wrong entity name, wrong field name)
```

If token is missing from JobDocument (copy failure):
1. Re-enable maintenance flag (set AdminSettings record back) — blocks new writes, users see banner again
2. Find the source record in FSDocument by id `69c6cfb440fa5e43db334f1c`
3. Copy it to JobDocument manually, set `legacy_id`, run FK rewrites for that record
4. Re-run portal token assertion
5. Re-smoke test all three tokens
6. Delete maintenance flag

If token is present but lookup is broken (code bug):
1. Hot-fix the portal lookup function specifically (this is usually a one-line fix)
2. Redeploy
3. Re-test
4. No maintenance flag needed — data is intact, this is code-only

Full code rollback (only if bug is not isolatable):
1. Re-enable maintenance flag
2. Revert frontend code deploy (both commits A and B)
3. Redeploy previous agentScopedQuery (remove shim changes made in Phase 5)
4. Revert agent instruction changes in dashboard
5. Confirm old workspace loads normally on reverted code
6. Old entities still intact — data fully restored

**Phase 6 Success + Bug Surfaces 24 Hours Later**

Scenario: Migration completed cleanly. Bari has been using the Jobs workspace for 24 hours. A bug is discovered — e.g., the portal lookup is working but a field on JobEstimate was not copied (some edge-case field not in the Section 4 map).

**Default answer: patch forward, not rollback.**

By this point:
- Bari may have created new clients, jobs, or estimates in Job* entities
- Those records don't exist in old FS* entities
- A full rollback would lose Bari's new work
- The bug is almost certainly isolated to a specific field or behavior, not catastrophic data loss

Patch-forward procedure:
1. Identify the specific field/behavior that's wrong
2. Write a targeted fix — either a data patch (update the missing field value on the affected JobEstimate record) or a code fix
3. Deploy the fix
4. AuditLog the patch
5. Verify the fix resolved the issue

When rollback is still appropriate 24 hours later:
- The bug is catastrophic (e.g., portal tokens are missing on multiple records, signature data is corrupted)
- AND Bari has not created any new native records
- AND old entities are still intact
- In this case, full rollback is still clean: delete Job* migrated records, restore server functions and code, old entities serve traffic

---

## Section 12 — Post-Migration Report Template

**Status:** Template — fill in during execution
**Author:** Hyphae (executing) / Mycelia (reviewing)
**File to create:** `JOBS-POSTMIGRATION-REPORT-{YYYY-MM-DD}.md`

```markdown
# JOBS-POSTMIGRATION-REPORT-{DATE}

**Migration:** Field Service → Jobs Namespace Rename
**Date executed:** {DATE}
**Executed by:** Hyphae
**Reviewed by:** Mycelia
**Execution window:** {START_TIME} – {END_TIME} Pacific
**Total duration:** {N} minutes
**Outcome:** SUCCESS | PARTIAL | FAILED | ROLLED_BACK

---

## Summary

{One paragraph: what happened, any notable deviations, current state of the organism.}

---

## Phase Execution Log

| Phase | Name | Status | Duration | Notes |
|---|---|---|---|---|
| Phase 0 | Prepare + dry-run | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 1 | Schema creation (12 entities) | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 2 | Pass 1 — record copy | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 3 | Pass 2 — FK rewrite | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 4 | Verification + backfills | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 5 | Server function cutover | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 6 | Code cutover | APPLIED / SKIPPED / FAILED | {Xm} | |
| Phase 7 | Shim removal + archival notes | APPLIED / SKIPPED / FAILED | {Xm} | |

---

## Record Migration Summary

| Old Entity | New Entity | Source Count | Migrated Count | Match |
|---|---|---|---|---|
| FieldServiceProfile | JobProfile | 2 | {N} | ✓/✗ |
| FSClient | JobClient | 3 | {N} | ✓/✗ |
| FSProject | Job | 1 | {N} | ✓/✗ |
| FSEstimate | JobEstimate | 1 | {N} | ✓/✗ |
| FSDocument | JobDocument | 3 | {N} | ✓/✗ |
| FSDailyLog | JobLog | 0 | {N} | ✓/✗ |
| FSMaterialEntry | JobMaterial | 0 | {N} | ✓/✗ |
| FSLaborEntry | JobLabor | 0 | {N} | ✓/✗ |
| FSDailyPhoto | JobPhoto | 0 | {N} | ✓/✗ |
| FSPayment | JobPayment | 0 | {N} | ✓/✗ |
| FSPermit | JobPermit | 0 | {N} | ✓/✗ |
| FSChangeOrder | JobChangeOrder | 0 | {N} | ✓/✗ |
| **TOTAL** | | **10** | **{N}** | ✓/✗ |

---

## AuditLog Reference

**AdminAuditLog entries created by this migration:**

| Step | Action | Entity | Record ID | AuditLog ID |
|---|---|---|---|---|
| 2.1 | JOBS_MIGRATION_COPY_JobProfile | JobProfile | {new_id} | {audit_id} |
| ... | ... | ... | ... | ... |
| 4.5 | JOBS_MIGRATION_ENABLED_SPACES_BACKFILL | Business | 69ea5590481b7e15af7216b6 | {audit_id} |

**Total AuditLog entries:** {N}
**Query to retrieve all:** `AdminAuditLog.list()` filter where `action LIKE "JOBS_MIGRATION_%"`

---

## Per-Step Reversal Table

| Step | Action Applied | Reversal Command | Status |
|---|---|---|---|
| Phase 2.1 | Copy 2 JobProfile records | `JobProfile.delete({id})` × 2 | AVAILABLE until 30-day cleanup |
| Phase 2.2 | Copy 3 JobClient records | `JobClient.delete({id})` × 3 | AVAILABLE |
| Phase 2.3 | Copy 1 Job record | `Job.delete({id})` | AVAILABLE |
| Phase 2.4 | Copy 1 JobEstimate record | `JobEstimate.delete({id})` | AVAILABLE |
| Phase 2.5 | Copy 3 JobDocument records | `JobDocument.delete({id})` × 3 | AVAILABLE |
| Phase 4.5 | enabled_spaces: "desk" → "jobs" | `Business.update("69ea5590481b7e15af7216b6", { enabled_spaces: ["profile","settings","desk"] })` | AVAILABLE |
| Phase 5.1 | agentScopedQuery updated | Redeploy previous function version | AVAILABLE |
| Phase 5.2 | agentScopedWrite updated | Redeploy previous function version | AVAILABLE |
| Phase 6 | Frontend code deploy | Git revert + redeploy | AVAILABLE until new native records created |

*Fill in actual new record IDs from AuditLog after execution.*

---

## Bari's Workspace Verification

**Profile (Red Umbrella):**
- legacy_id lookup: ✓/✗
- user_id: ✓/✗ — `69b9f8313b41521cc50c97d8`
- business_id: ✓/✗ — `69ea5590481b7e15af7216b6`
- workspace_name: ✓/✗ — "Red Umbrella"
- owner_name: ✓/✗ — "Bari Swartz"
- subscription_tier: ✓/✗ — "full"
- invite_code: ✓/✗ — "TDAqwuxj"
- archived_at: ✓/✗ — null

**Holman Barn Project (Job):**
- legacy_id lookup: ✓/✗
- profile_id resolves to Bari's JobProfile: ✓/✗
- client_id resolves to Dr. Nathan Holman JobClient: ✓/✗
- estimate_id resolves to $121K JobEstimate: ✓/✗
- name verbatim (trailing spaces): ✓/✗
- total_budget = 121657.57: ✓/✗

**$121,657.57 Estimate:**
- legacy_id lookup: ✓/✗
- portal_token = `ea14972d-3978-4381-860b-849f4b4d3694`: ✓/✗
- portal_link_active = true: ✓/✗
- status = "accepted": ✓/✗
- responded_at = "2026-04-24T17:37:30.168Z": ✓/✗
- signed_at = null: ✓/✗
- line_items count = 30: ✓/✗
- total = 121657.57: ✓/✗

**FSDocument × 3:**

| legacy_id | title | status | portal_token intact | signature_data intact | content intact |
|---|---|---|---|---|---|
| `69c6fa2731cdee2982627f4a` | Subcontractor Agreement | draft | N/A (null) | N/A | ✓/✗ |
| `69c6cfb440fa5e43db334f1c` | Subcontractor Agreement | awaiting_signature | ✓/✗ | N/A | ✓/✗ |
| `69c698c2214ef0d74eca1336` | Notice of Right to Lien | signed | ✓/✗ | ✓/✗ | ✓/✗ |

**FK Integrity Spot-Check:** PASS / FAIL — {N} failures ({detail if any})

---

## Portal Token Verification Results

| Token | Entity | Pre-migration status | Post-migration portal loads | Before/after match |
|---|---|---|---|---|
| `ea14972d-...` | JobEstimate | accepted | ✓/✗ | ✓/✗ |
| `77f38acf-...` | JobDocument | awaiting_signature | ✓/✗ | ✓/✗ |
| `4c11ee24-...` | JobDocument | signed | ✓/✗ | ✓/✗ |

---

## Phase 6 Smoke Test Results

| Check | Pass |
|---|---|
| /jobs route loads | ✓/✗ |
| Bari's profile visible | ✓/✗ |
| Bari's clients visible (3 records) | ✓/✗ |
| Bari's documents visible | ✓/✗ |
| Red Umbrella estimate visible ($121,657.57) | ✓/✗ |
| Red Umbrella job visible (Holman barn) | ✓/✗ |
| Portal token ea14972d resolves | ✓/✗ |
| Portal token 77f38acf resolves | ✓/✗ |
| Portal token 4c11ee24 resolves | ✓/✗ |
| agentScopedQuery("jobs") returns Bari's data only | ✓/✗ |
| New test record creation + deletion | ✓/✗ |
| /join/field-service redirects to /join/jobs | ✓/✗ |
| Maintenance flag absent from AdminSettings | ✓/✗ |
| Write buttons re-enabled in UI | ✓/✗ |

---

## Carryover Items

*Flag anything noticed during migration that wants a follow-up pass.*

| Item | Priority | Owner | Notes |
|---|---|---|---|
| FK field name standardization (JobClient.workspace_id → profile_id) | Low | Hyphae | Deferred per Path A decision. Schedule after Jobs stable. |
| `xactimate_enabled` field duplication on JobProfile | Low | Hyphae | Pre-existing on FieldServiceProfile. Clean up in schema audit pass. |
| Old /field-service route redirects | 30-day | Hyphae | Remove redirect rules when old entities deleted. |
| {anything discovered during execution} | | | |

---

## Phase 7 Archival Schedule

| Item | Target Date | Status |
|---|---|---|
| Maintenance flag deleted | {migration date} | ✓ DONE |
| Compatibility shim removed from agentScopedQuery | {migration date + 1 day, after confirming Phase 6 stable} | PENDING |
| Old FS* entity schemas deleted | {migration date + 30 days} | SCHEDULED |
| Old route redirect rules removed | {migration date + 30 days} | SCHEDULED |
| Deletion verification query run | {migration date + 30 days} | SCHEDULED |

**30-day cleanup gate query:**
```javascript
// Before deleting old entities, confirm no new writes landed there after cutover
const CUTOVER_DATE = new Date("{migration date}T{cutover_time}Z");
for (const oldEntity of ["FieldServiceProfile","FSClient","FSProject",
  "FSEstimate","FSDocument","FSDailyLog","FSMaterialEntry","FSLaborEntry",
  "FSDailyPhoto","FSPayment","FSPermit","FSChangeOrder"]) {
  const records = await base44.asServiceRole.entities[oldEntity].list();
  const postCutover = records.filter(r => new Date(r.created_date) > CUTOVER_DATE);
  if (postCutover.length > 0) {
    console.warn(`${oldEntity} has ${postCutover.length} records created after cutover — investigate before deleting`);
  }
}
```

---

## Deviations from Plan

| Section | Deviation | Reason | Impact |
|---|---|---|---|
| {Section N} | {What differed from the plan} | {Why} | {Impact on integrity} |

*Fill in during execution. If no deviations: "None — executed as planned."*

---

*Report filed by Hyphae. Reviewed by Mycelia. Old entities safe to delete after: {migration date + 30 days}*
```

---

## Appendix A — Execution Notes for Hyphae

**On the existing AdminAuditLog state:** Mycelia flagged at the end of Section 12 drafting that existing AdminAuditLog records in the live app have all-null `action`/`details`/`entity_type`/`entity_id` fields. Whatever wrote them earlier didn't populate the structured fields. The migration script writes these fields explicitly. The query pattern `filter where action LIKE "JOBS_MIGRATION_%"` will only return migration rows, not the older null-field rows. That's fine and by design.

**On idempotency of every step:** Every action in the migration must be safe to re-run. If `phase_2` ran 80% then crashed, re-running it from the start completes the remaining 20% without duplicating the 80%. The `legacy_id`-keyed pattern in Pass 1 and the `needsRewrite` check in Pass 2 enforce this.

**On the dry-run probe:** Section 3.7's ID behavior probe must run before any real copies. If the platform behavior has changed since this plan was drafted (`id_was_honored` returns true), the entire `legacy_id` strategy needs revisiting. Stop and surface to Doron before proceeding.

**On the four-category Base44 confirmation:** Per DEC-162, every Base44 agent prompt requires the four-category response structure. Don't accept a Base44 commit that bundles unrelated lint fixes — surface them as out-of-scope observations instead.

**On Path A:** `JobClient.workspace_id` field name is preserved. Do NOT normalize to `profile_id` during this migration. The standardization is Path B, scheduled separately after Jobs is stable.
