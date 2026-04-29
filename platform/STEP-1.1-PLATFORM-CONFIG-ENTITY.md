# Step 1.1: PlatformConfig Entity — Report & Recommendation

> Phase 1: Admin Panel restructure with configurable settings.  
> Check Base44 entity options and recommend approach for PlatformConfig.

---

## 1. What’s in the repo

### Base44 usage (community-node)

- **Client:** `createClient()` from `@base44/sdk` in `src/api/base44Client.js`; app is wired via `appId`, `serverUrl`, `token` from `app-params`.
- **Entities:** Used as `base44.entities.<EntityName>` (e.g. `Business`, `Event`, `AdminSettings`, `Location`, `Spoke`, `User`, `Archetype`, `CategoryGroup`, `SubCategory`, `Review`, `JoyCoins` (formerly PunchPass), etc.). No entity definitions live in this repo.
- **Entity definitions:** Base44 docs say entities are defined with **JSON Schema** and can be:
  - Managed in the **Base44 dashboard**: Data → select entity → Schema.
  - Or defined in a **`base44/entities/`** directory (JSON/JSONC) and deployed with the **`entities push`** CLI.
- **This repo:** There is **no `base44/entities/`** folder. So entities are either defined in the Base44 dashboard for this app or in another repo that runs the CLI.

### Existing “config-like” entity: AdminSettings

- **Where:** `src/components/admin/AdminSettingsPanel.jsx`
- **Usage:** `base44.entities.AdminSettings.list()`, `.create({ key, value })`, `.update(id, { value })`
- **Shape:** Key-value: `key` (string), `value` (string). Used for things like `show_accepts_silver_badge`, `default_boost_duration_days`; values are JSON strings.
- **Not a fit for:** A first-class `domain` + `config_type` + `items` model. We could emulate it with key conventions and JSON in `value`, but that’s a fallback, not the intended design.

---

## 2. Required PlatformConfig shape

From CONFIG-SYSTEM.md, we need at least:

```javascript
{
  id: UUID,
  domain: string,      // 'platform' | 'events' | 'venues' | 'onboarding'
  config_type: string, // 'event_types' | 'networks' | 'age_groups' | etc.
  items: array,       // Array of config items
  updated_at: Date,
}
```

Optional for later: `created_at`, `updated_by`, `allow_partner_additions`, `allow_partner_disable`. Each **item** can have `value`, `label`, `icon`, `description`, `active`, `sort_order`, `min_tier`, `parent`, `metadata`.

---

## 3. Recommendation

### Preferred: Create a dedicated PlatformConfig entity

- **Why:** Matches CONFIG-SYSTEM.md and ADMIN-ARCHITECTURE.md: one row per `(domain, config_type)` with an `items` array. Clean API, clear semantics, good for Admin UI and for `useConfig(domain, configType)`.
- **How:**
  1. **If you use the Base44 dashboard:** In the app’s Data section, create a new entity and set its schema to the JSON Schema below (or the one in `Spec-Repo/schemas/PlatformConfig.schema.json`).
  2. **If you use the CLI and `base44/entities/`:** Add `base44/entities/PlatformConfig.json` (or `.jsonc`) with the same schema and run `entities push` (or the equivalent from Base44 docs).
- **Schema:** A JSON Schema for Base44 is in **`Spec-Repo/schemas/PlatformConfig.schema.json`**. It defines `domain`, `config_type`, `items`, and optional `allow_partner_additions` / `allow_partner_disable`. Use it when creating the entity in the dashboard or in `base44/entities/`.

After the entity exists, the app will expose `base44.entities.PlatformConfig` (filter, create, update, list) with no code changes in the SDK usage.

---

## 4. Fallback: Use AdminSettings with a key convention

If you **cannot** create a new entity (no dashboard access and no CLI / `base44/entities/`), you can emulate PlatformConfig with **AdminSettings**:

- **Key format:** `platform_config:{domain}:{config_type}`  
  Examples: `platform_config:events:event_types`, `platform_config:platform:networks`.
- **Value:** JSON string: `JSON.stringify({ items, updated_at })`.
- **Usage:** One AdminSettings row per (domain, config_type). Frontend parses `value` and presents it as config; writes back the same key with updated `items` and `updated_at`.

**Pros:** No new entity; works with current setup.  
**Cons:** Key namespace overlaps with other settings; no dedicated schema or validation; list/filter by domain is by key prefix, not a real relation. Acceptable only as a temporary workaround until PlatformConfig exists.

---

## 5. Summary

| Option | When to use | Action |
|--------|-------------|--------|
| **Create PlatformConfig entity** | You have Base44 dashboard or CLI / `base44/entities/` | Create entity with `Spec-Repo/schemas/PlatformConfig.schema.json`; then use `base44.entities.PlatformConfig`. |
| **Use AdminSettings** | You cannot create a new entity yet | Use keys `platform_config:{domain}:{config_type}` and JSON in `value`; implement a thin adapter in the app that mirrors the intended PlatformConfig API. |

**Recommendation:** Create the **PlatformConfig** entity using the schema in **`Spec-Repo/schemas/PlatformConfig.schema.json`**. If creation is blocked, use the AdminSettings fallback and plan to migrate to PlatformConfig once the entity is available.

---

## 6. Schema file location

- **`Spec-Repo/schemas/PlatformConfig.schema.json`** — Base44 JSON Schema for the PlatformConfig entity (dashboard or `base44/entities/`).

After the entity is created, Step 1.2 can add the seed data and Step 1.3 can wire the Admin UI and `useConfig` to `base44.entities.PlatformConfig`.
