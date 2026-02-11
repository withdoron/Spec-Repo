# Build Protocol

> The universal sequence every feature goes through before, during, and after development.
> This is how LocalLane ships anything. No exceptions.
> Created: 2026-02-06

---

## Why This Exists

Every feature touches multiple surfaces, multiple tiers, and multiple users. Without a repeatable protocol, we ship half-features — the event editor sits unused, the widget says "coming soon" for months, or the admin panel has no controls for something the user can already do.

This protocol catches those gaps before they become debt.

---

## The Sequence

### Phase 0: Decision Filter

Before anything else — should we build this?

1. Does this help a real person right now?
2. Does this make the Organism more alive?
3. Does this move money closer to flowing?

If none of the above: defer unless Doron specifically asks. If yes: proceed to Phase 1.

---

### Phase 1: Plan

**What we're building and why.**

| Item | Answer |
|------|--------|
| Feature name | One clear name used everywhere |
| One-sentence goal | What should be true after this ships? |
| Who benefits | Which user type(s) does this serve? |
| Success signal | How do we know it's working? |
| Dependencies | What must exist before we start? |
| Decisions needed | Any open questions that block implementation? |

**Output:** A short feature brief (can be inline in the strategy chat or a standalone section in the spec doc). No build starts without a clear goal.

---

### Phase 2: Scheme

**Data model, state transitions, relationships.**

Define:
- New entities or fields needed (Base44 schema)
- Relationships to existing entities
- State transitions (e.g., pending → active → expired)
- Calculations or derived data (e.g., revenue share formulas)
- API patterns (hooks, queries, mutations)

Questions to answer:
- Where does the data live?
- Who can create/read/update/delete it?
- What triggers state changes?
- Does this data feed into other features?

**Output:** Entity definitions, field lists, state diagrams (can be text-based). Added to ARCHITECTURE.md or the feature spec.

---

### Phase 3: Surface Mapping

**How and where does this feature show up?**

Every feature potentially touches multiple surfaces. Walk through each and decide: does this feature appear here? If yes, what does the user see?

| Surface | Question |
|---------|----------|
| **MyLane** | Does the community member see anything new or changed? |
| **Business Dashboard** | Does the business owner see/control anything? |
| **Admin Panel** | Does the steward need tools to manage this? |
| **Directory** | Does this affect how businesses display? |
| **Events Page** | Does this change event browsing or detail? |
| **Business Profile** | Does this show on the public business page? |
| **Event Detail Modal** | Does this change the event detail view? |
| **Onboarding Wizard** | Does this require new onboarding steps? |
| **Settings** | User or business settings affected? |
| **Notifications** | Should anyone be notified about this? |
| **The Good News** | Does this generate newsletter-worthy content? |

Mark each as: Shows Here / Not Applicable / Future Phase.

**This is the step that prevents half-features.** If you build Joy Coin settings on the Business Dashboard but forget to show the badge in the Directory, members can't find participating businesses.

**Output:** Surface map table in the feature spec.

---

### Phase 4: UI/UX Design

**What does it look like and feel like?**

- Reference STYLE-GUIDE.md for all visual decisions
- Reference existing component patterns (.cursorrules in each app repo)
- Sketch the user flow: what does the user do, step by step?
- Define empty states (what shows before any data exists?)
- Define loading states
- Define error states
- Define mobile behavior (does this work on a phone?)

Key principle: **intuitive before pretty.** A business owner logging in for the first time should understand what they're looking at within 5 seconds. Labels, hierarchy, and flow matter more than animation.

**Output:** Text-based wireframes or flow descriptions in the feature spec. Reference screenshots of existing patterns where helpful.

---

### Phase 5: Pre-Build Audit

**What exists already? What can we reuse? What conflicts?**

Before writing a single Cursor prompt:

1. Search the codebase for related components, hooks, utils
2. Check if similar patterns already exist (don't reinvent)
3. Identify files that will be modified (not just created)
4. Flag potential conflicts with existing code
5. Note any tech debt this build should clean up along the way

**Output:** Audit findings list. "These 4 files exist and need updates. This hook can be reused. This widget needs to be replaced, not extended."

---

### Phase 6: Security

**Who can see/do what?**

- Base44 entity permissions (RLS rules)
- Role-based access (owner vs staff vs admin vs public)
- Route protection (what happens if someone navigates directly?)
- Data exposure (can a user see another business's data?)
- Input validation (what can go wrong with user input?)

Cross-reference with DEC-025 (entity permission audit) and existing security patterns.

**Output:** Security requirements added to the feature spec. Entity permission settings documented.

---

### Phase 7: Build

**Cursor prompts, implementation, testing.**

This is where the Build chat happens. One feature per chat. Prompts follow the standard format:

```
GOAL: [What should be true after]
WHERE: [File path(s)]
[Instructions]
CONSTRAINTS:
- [What NOT to change]
- Git commit message: "[message]"
```

Build sequence:
1. Data layer first (entities, hooks, utilities)
2. Core component(s) next
3. Integration into surfaces (from Phase 3 map)
4. Tier gating (Phase 8)
5. Test each piece as it ships

**Output:** Working feature in the app.

---

### Phase 8: Tier Gating

**Does this feature behave differently per tier?**

| Tier | Code Value | Monthly | Behavior |
|------|-----------|---------|----------|
| Basic | `basic` | Free | Limited access, content reviewed before publish |
| Standard | `standard` | $79 | Full access, auto-publish, Joy Coin pool, analytics |
| Partner | `partner` | $149 | Own branded node, enhanced features, earned status |

For each tier:
- What does the user see?
- What's locked with an upgrade prompt?
- Does this feature add value to a tier's sales pitch?

If this feature changes the tier value proposition: update TIER-SYSTEM.md and any marketing copy.

**Output:** Tier behavior documented. Upgrade prompts implemented where needed.

---

### Phase 9: Polish

**Edge cases, empty states, mobile, dark theme.**

Checklist:
- [ ] Empty states show friendly messages (not blank screens or "coming soon")
- [ ] Loading states show skeletons or spinners
- [ ] Error states show human-readable messages
- [ ] Mobile-responsive (test on phone-sized viewport)
- [ ] Dark theme compliance (no light backgrounds, STYLE-GUIDE.md)
- [ ] Icons are gold or white only
- [ ] No hardcoded values that should be config
- [ ] Console clean (no errors, no leftover debug logs)
- [ ] Accessible (buttons have labels, contrast is sufficient)

**Output:** Polish fixes shipped.

---

### Phase 10: Post-Build Audit

**Does what shipped match the plan?**

Walk the feature spec from Phase 1-8:
1. Does the data model match the scheme?
2. Does the feature appear on every surface from the map?
3. Are security rules implemented correctly?
4. Do tier gates work as specified?
5. Is the UX intuitive on first encounter?

If anything's missing: fix it now or add it to the Quick Fixes table in STATUS-TRACKER.md with a clear note.

**Output:** Audit findings resolved or documented.

---

### Phase 11: Documentation

**Update the system of record.**

| Document | Update If... |
|----------|-------------|
| STATUS-TRACKER.md | Always — add session log entry, update shipped list |
| CURRENT-STATE.md | Feature adds new components or changes existing ones |
| DECISIONS.md | Any architectural decisions were made during build |
| ARCHITECTURE.md | New entities, hooks, or data patterns introduced |
| LAUNCH-CHECKLIST.md | Feature completes a checklist item |
| TIER-SYSTEM.md | Tier behavior or pricing changed |
| Terms of Service | New data collection, new user-generated content |
| Privacy Policy | New data processing, new third-party services |
| claude.md | New conventions or pitfalls discovered |

---

### Phase 12: Legal Check

**Does this feature change what we collect, store, or process?**

Triggers for legal review:
- New user-generated content → Terms of Service update
- New data collection → Privacy Policy update
- New third-party services → Privacy Policy disclosure
- Payment or financial data → Both + potential regulatory review
- New money flows → Revenue share agreement review

Most features won't trigger this. But when they do, it's critical to catch it before launch.

---

### Phase 13: Organism Signal

**Does this feature generate data that feeds the Organism?**

If yes: document what vitality signals this feature produces. Reference ORGANISM-CONCEPT.md for signal types and weights.

If no: consider whether it should. Not everything needs to feed the Organism, but if a feature generates participation data, that data should eventually flow into the creature.

This phase is often "noted for future" rather than built immediately. The important thing is capturing the signal design so it's ready when the Organism visualization ships.

---

## Quick Reference Card

```
 0. Decision Filter    → Should we build this?
 1. Plan               → What and why?
 2. Scheme             → Data model and state
 3. Surface Mapping    → Where does it show up?
 4. UI/UX Design       → What does it look/feel like?
 5. Pre-Build Audit    → What exists? What can we reuse?
 6. Security           → Who can see/do what?
 7. Build              → Cursor prompts and implementation
 8. Tier Gating        → Different behavior per tier?
 9. Polish             → Edge cases, mobile, dark theme
10. Post-Build Audit   → Does reality match the plan?
11. Documentation      → Update the system of record
12. Legal Check        → Terms/Privacy affected?
13. Organism Signal    → Data for the creature?
```

---

## How to Use This

**For strategy chats:** Walk through Phases 0-6 to produce a complete feature spec before any code.

**For build chats:** Reference the spec from Phases 0-6, then execute Phases 7-9.

**For status checks:** Use Phases 10-13 as a post-ship checklist.

**For Claude:** When Doron says "let's build [X]," run Phase 0 (Decision Filter), then check if a spec exists covering Phases 1-6. If not, produce one before writing Cursor prompts.

---

*This protocol sits alongside WORKFLOW.md (tool mechanics) and the Decision Filter (should we build it). Together they form the complete "how we ship" system.*
