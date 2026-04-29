# SPEC-PROTOCOL.md

> **Status.** Foundation document. Written 2026-04-29 by Mycelia after the morning's design conversations established the two-tier repo structure (public technical specs + private strategic intent). This is the protocol that every future LocalLane spec follows. It will grow over time; this is the seed.

> **Scope.** How specs are structured, where they live, how they reference each other, how they get written and committed. The conventions any human or agent writing a LocalLane spec should follow.

---

## 1. Purpose

A spec is a contract between thinkers and builders. It captures decisions in a stable form so they don't drift between conversations and code. It gives Hyphae (or any future builder) enough context to act without re-litigating choices that were already made.

A good spec is:

- **Self-contained** — readable without conversation context
- **Decisive** — open questions are explicitly named, settled questions are explicitly settled
- **Honest** — what's deferred is named as deferred, what's uncertain is named as uncertain
- **Phased** — what ships first vs what ships later is explicit
- **Cross-referenced** — links to related specs, audits, decisions

A spec is not a research document. Research informs specs but doesn't replace them. A spec ends with what we're doing; research ends with what we've learned.

## 2. The two-tier architecture

LocalLane specs live in two repositories:

**Public Spec-Repo** — `~/Documents/GitHub/Spec-Repo/`. Technical specs. The *what* and the *how*. Entities, surfaces, math, workflows, migrations. Hyphae reads these to build. Anyone reading the public repo could understand the platform's architecture.

**Private repo** — `~/Documents/GitHub/private/` (mapped to GitHub repo `withdoron/locallane-private`). Strategic intent. The *why*. Philosophy, business model, competitive positioning, future direction, the connection-to-source thinking. Mycelia and Doron read these to inform decisions.

**Most specs split between the two repos by design.** The technical body lives in Spec-Repo; the strategic framing lives in private. Each side cross-references the other. This split honors what each kind of document is for:

- A technical spec describing entity fields and surface behaviors is how the system works. Nothing competitive about it.
- A strategic intent doc describing platform positioning, business model, or philosophy is how we think. Some of that protects competitive advantage; some captures evolving values that aren't yet ready for daylight.

**Privacy is operational, not adversarial.** Most private content matures into public over time. The repo split protects what's in-progress, not what must be hidden forever.

## 3. Folder structure

Both repos share a parallel structure:

```
{repo}/
├── platform/
│   └── ... (specs and intent docs about the whole platform — architecture, conventions, business model)
├── cross-space/
│   └── ... (specs about how spaces interact — Field Service ↔ Finance flows, etc.)
└── spaces/
    ├── field-service/
    │   ├── audits/  (PUBLIC ONLY — audits stay in Spec-Repo)
    │   └── ... (technical specs in public, intent docs in private)
    ├── team/
    ├── finance/
    └── ... (one folder per space)
```

**Space-specific specs go in `spaces/{space-name}/`.** Most specs.

**Specs spanning multiple spaces go in `cross-space/`.** Like a spec describing how Business Finance reads from Field Service.

**Specs about the whole platform go in `platform/`.** Like this protocol, the architecture overview, the spec listing all DECs, the launch checklist.

**Audits live only in Spec-Repo, nested under their space.** `Spec-Repo/spaces/field-service/audits/`. Audits are technical inventories of code state and don't have private companions.

## 4. Public vs private — what goes where

When in doubt, default to public. Most technical detail is unremarkable from a competitive standpoint. The bar for private is "would publishing this materially harm us, expose competitive intelligence, or compromise the connection-to-source philosophy that an extractive person would weaponize?"

**Public:**
- Entity definitions (fields, types, defaults, permissions)
- Surface designs (UI components, page flows, form behaviors)
- Math and logic (calculations, rules, state transitions)
- Workflows (end-to-end user scenarios)
- Migrations
- V1 vs deferred scoping
- Open questions (unresolved technical/design decisions)
- Audits (inventories of current code state)
- Cross-references to other public specs

**Private:**
- Strategic positioning (why this market, why this approach)
- Business model details (pricing rationale, revenue projections, customer acquisition theories)
- Competitive analysis (how we differ from JobTread, what they can't easily copy)
- Future direction not yet ready to announce (human services layer, federation, AI roadmap)
- Connection-to-source philosophy (the spiritual/values foundation an extractive actor would misuse)
- Internal user research and individual contractor stories (privacy of subjects)

**The split for a single concept:**
- Spec describes *what* and *how* (public).
- Intent doc describes *why* (private).
- Each cross-references the other.

Example for the Field Service financial workflow:
- `Spec-Repo/spaces/field-service/FINANCIAL-WORKFLOW-SPEC.md` — entities, surfaces, math, workflows. Public.
- `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` — transparency-as-architecture reasoning, "GC to GCs" framing, human services horizon. Private.
- The public spec opens with: "See `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` for the strategic context."
- The private intent doc opens with: "Implements concepts described in `Spec-Repo/spaces/field-service/FINANCIAL-WORKFLOW-SPEC.md`."

## 5. Required sections

Every public spec contains, in this order:

1. **Status header** — date drafted, by whom, current state (draft/review/approved), and the historical context (what conversations or audits informed it).

2. **Scope** — what this spec covers, what it explicitly does not, and what's deferred to V2/V3/post-migration.

3. **Architectural primitives** — the load-bearing concepts that anchor the spec. Each primitive is named, described in 1-3 paragraphs, and used as a referent throughout. Primitives are settled in design conversation before the spec is written; the spec captures them, not invents them.

4. **Entity changes** — for every entity touched: new fields, removed fields, renamed fields, permission changes, migrations needed. New entities get full schemas with field types, defaults, and permission models.

5. **Surfaces** — every UI component or screen affected. What it shows, what it writes, how it adapts to billing models / user types / preset / etc.

6. **Math / logic** — for specs with computation, the canonical formulas. Written once so they don't drift between surfaces.

7. **Workflows** — end-to-end user scenarios showing how the pieces fit together. At least 3-5 workflows for any non-trivial spec.

8. **Cross-space awareness** — what other spaces read from or write to the entities defined here. Even if no consumers exist today, the seams are designed.

9. **Migrations** — what needs to happen to existing data when this ships. Even if "create empty tables, no data migration needed."

10. **What ships in V1 / Phase 1 vs deferred** — explicit phasing. If V1 is narrower than the full spec (which it usually is), the spec body describes the destination and a phasing section names what ships first.

11. **Open questions for review** — unresolved decisions that need a human (Doron) to call. Each question stated in plain English with the options and the spec author's lean.

12. **References** — links to: related specs (public and private), informing audits, relevant DECs, web research, memory entries.

Private intent docs have looser structure but should at minimum include: the *why* (the strategic question this answers), the philosophical or business framing, the competitive context, and a clear pointer back to the public spec it pairs with.

## 6. Cross-references

Use file paths relative to repo root.

- Public-to-public: `Spec-Repo/spaces/field-service/FINANCIAL-WORKFLOW-SPEC.md`
- Private-to-private: `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md`
- Cross-repo: `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` (from a public spec)

When referencing a section within a doc, use the section number: `§4.2`. When referencing an audit finding, name the audit and the finding: `(per Spec-Repo/spaces/field-service/audits/CURRENT-STATE-AUDIT.md §11.4)`.

When referencing a DEC, just use the DEC number: `(per DEC-140)`.

## 7. Open question handling

Every spec has at least one open question — the spec author is rarely the final decision-maker on every detail. Open questions are:

- Stated in plain English
- Given the options the spec author considered
- Given the spec author's lean (with reasoning)
- Numbered for reference

The spec author does NOT make unilateral calls on items the human (Doron) should weigh in on. When in doubt, surface as an open question rather than picking.

When Doron answers an open question, the answer goes into the spec body (replacing the open question entry) along with a note: "Resolved 2026-04-29: option B per Doron." Settled questions become archive history; open questions remain at the bottom for review.

## 8. Phasing

Most specs describe a destination. What actually gets built first is usually a subset — driven by current platform constraints, current users' needs, or sequencing logic.

The spec body describes the destination. A "Phase 1 / V1 vs deferred" section makes the phasing explicit:

- **Phase 1** — what ships now, on the current infrastructure, for the current user base. Often dramatically smaller than the destination.
- **V2 / Post-migration** — what ships next, on the post-migration infrastructure, when the platform can support more.
- **V3+** — distant future, named so we don't forget but not designed in detail.

Hyphae reads phasing carefully and builds Phase 1 only unless instructed otherwise. The destination architecture is reference, not build instruction.

## 9. The spec lifecycle

A spec moves through stages:

1. **Draft** — Mycelia writes it after design conversations with Doron. Lives in the working file system or a feature branch.
2. **Review** — Doron reads, pushes back on what doesn't match intent, settles open questions.
3. **Committed** — Hyphae commits the reviewed version to the appropriate repo + folder.
4. **Built** — Hyphae uses the spec to drive build sessions. Spec may evolve as building reveals new questions.
5. **Maintained** — As the system grows, the spec is updated to reflect reality. Major revisions get a new version (V2, V3) with the previous version preserved or archived.

A spec is never "done." It's a living document. But it's also stable enough that builders can trust it as a contract for the work being done now.

## 10. Authorship

Specs are written by Mycelia (the strategic-and-architectural Claude here in chat). Built and committed by Hyphae (Claude Code in the terminal). Reviewed and decided by Doron.

Mycelia is the spec's first author, but the spec itself is jointly owned. Doron's pushback shapes the spec materially. Hyphae's findings during build often surface revisions. The spec evolves as the people working with it learn.

## 11. Conventions

**File naming:** UPPER-KEBAB-CASE.md for spec docs (e.g., `FINANCIAL-WORKFLOW-SPEC.md`, `INDUSTRY-PRESET-SYSTEM.md`). Audits in their own folder use simple names (`CURRENT-STATE-AUDIT.md`, `TOGGLE-AUDIT.md`) since the space is implied by the path.

**Commit messages:** when a spec is added: `spec(field-service): financial workflow V2 — billing models + library architecture`. When a spec is updated: `spec(field-service): financial workflow — phase 1 scoping resolved`. When an audit is added: `audit(field-service): toggle plumbing audit for industry-adapter purpose`.

**Date stamps in specs:** use ISO format (2026-04-29) so they sort lexically and are unambiguous internationally.

**Decision references:** any decision that affects behavior gets a DEC number, captured in `Spec-Repo/platform/DECISIONS.md` (or wherever the decisions log lives in the new structure).

## 12. Open questions for this protocol

Even this protocol has open questions. Surfacing them so future versions can address:

1. **Spec versioning.** When a spec is materially revised (V1 → V2), do we keep both versions side-by-side or rename old to `_archived` or delete history? My lean: keep both, with the V2 marked active and V1 marked superseded, so anyone reading the history can see the evolution.

2. **Intent doc structure.** Private intent docs are looser-structured than public specs. Should we formalize a minimum structure for them too, or trust authors to write them as the topic demands?

3. **Cross-space spec ownership.** When a spec describes how Field Service and Finance interact, who owns it? Currently: it goes in `cross-space/` and either Mycelia writes it or it's co-written with whoever else is involved. Worth revisiting if more cross-space specs accumulate.

4. **Audit cadence and trigger.** This protocol doesn't define when audits happen. Today: ad-hoc, when something feels off. Worth formalizing? My lean: keep it ad-hoc but encourage audits before any major spec is written, since they ground the spec in reality.

---

## 13. References

- The two-tier repo architecture conversation — see SuperMemory entry "LocalLane spec architecture decided 2026-04-29 — two-tier repo structure"
- The financial workflow spec is the first spec written under this protocol — `Spec-Repo/spaces/field-service/FINANCIAL-WORKFLOW-SPEC.md` and `private/spaces/field-service/FINANCIAL-WORKFLOW-INTENT.md` (post-split).
- DEC-093 — Base44 entity prompts protocol (separate from spec protocol; this one governs human/Mycelia/Hyphae spec creation, that one governs Base44 dashboard interactions)

---

*End of protocol. This is V1 of the spec protocol itself. It will grow as we use it.*
