# Development Workflow

> How Claude, Cursor, GitHub, and Base44 work together to build the LocalLane ecosystem consistently.

---

## The Workflow

```
┌─────────────┐    ┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌─────────────┐
│   Claude    │ ─► │  Spec-Repo  │ ─► │ Claude Code  │ ─► │   GitHub    │ ─► │   Base44    │
│   (Chat)    │    │   (Truth)   │    │  + Cursor    │    │  (Version)  │    │   (Live)    │
└─────────────┘    └─────────────┘    └──────────────┘    └─────────────┘    └─────────────┘
       │                                      │
       └──── claude.md ◄─────────────────────┘
              (shared memory)
```

### Step-by-Step

1. **Claude reviews/updates Spec-Repo**
   - Makes architectural decisions
   - Updates DECISIONS.md, ARCHITECTURE.md, STYLE-GUIDE.md
   - Creates new archetype specs
   - Generates files for Cursor to use

2. **Cursor/Agent reads Spec-Repo before coding**
   - ALWAYS reference spec files when implementing
   - Follow STYLE-GUIDE.md for all UI
   - Follow ARCHITECTURE.md for patterns
   - Check DECISIONS.md for constraints

3. **Code changes pushed to GitHub**
   - Each node repo (events-node, community-node, etc.)
   - Commits reference spec changes when applicable

4. **Base44 auto-syncs from GitHub**
   - Changes appear in Base44 editor
   - Use Preview mode to test
   - Manual publish makes them live

---

## Division of Labor

### Claude Chat (Strategy & Architecture)

**Does:**
- Architecture decisions, spec writing, strategic planning
- Cursor/Claude Code prompt generation
- Code review via screenshots
- Spec-repo audit and maintenance
- Pattern detection → skill creation triggers

**Doesn't:**
- Direct code editing or GitHub pushes
- Make implementation decisions without Doron's approval on big items

### Claude Code (Codebase-Aware Implementation)

**Does:**
- Multi-file refactors, audits, bulk changes
- Reads `claude.md` at session start (project conventions)
- Updates `claude.md` when mistakes occur or patterns emerge
- Executes Cursor-style prompts with full codebase awareness

**Doesn't:**
- Make architectural decisions (flags and asks)
- Work on strategic planning (that's Chat)

### Cursor (Targeted Code Implementation)

**Does:**
- Feature builds from copy-paste prompts
- Single-file or focused multi-file changes
- Quick fixes with exact find/replace

**Doesn't:**
- Make architectural decisions alone
- Deviate from spec without updating it

### Doron (Vision & Decisions)

**Does:**
- Final decisions, field testing, business relationships
- Tests in browser, reports with screenshots
- Approves major changes before implementation

**Doesn't:**
- Need to write code or understand implementation details

---

## Consistency Rules

### Before Starting Any Work

1. **Read the Spec-Repo first**
   ```
   Always start by reading:
   - README.md (overview, current status)
   - ARCHITECTURE.md (technical patterns)
   - STYLE-GUIDE.md (UI standards)
   - .cursorrules (component patterns per app repo)
   - DECISIONS.md (constraints)
   - Relevant archetype spec (if building a node)
   ```

2. **Check for existing patterns**
   - Look at the most mature app (currently property-pulse or contractor-daily) for reference implementation
   - Copy shared configs (tailwind.config.js, package.json)
   - Follow established folder structure

3. **Update Spec-Repo when making decisions**
   - New decision? → Add to DECISIONS.md
   - Architecture change? → Update ARCHITECTURE.md
   - Style change? → Update STYLE-GUIDE.md

---

## Starting a Session

### Claude Chat
Starting a new Claude Chat conversation in the LocalLane project automatically loads project knowledge (spec-repo + private docs). State what you're working on today.

### Claude Code
Claude Code reads `claude.md` at the start of every session. The file contains project conventions, known pitfalls, architecture patterns, and the Gold Standard design rules. After every session, update it:

```
Update claude.md if anything should be prevented in future
```

### Session SOP
1. **Plan** — Describe goals in Chat, Claude writes prompts
2. **Build** — Run prompts in Claude Code or Cursor
3. **Test** — Check results in browser, report back
4. **Learn** — Update claude.md with new conventions or pitfalls
5. **Document** — Update spec-repo if decisions were made
6. **Push** — Single descriptive commit to GitHub

---

## Cursor/Agent Instructions

When implementing code:

### 1. Always Reference Spec-Repo

- Read relevant spec files before coding
- Follow patterns exactly as documented in Spec-Repo and app .cursorrules
- When in doubt, check the spec

### 2. Use Shared Configurations

- Copy `tailwind.config.js` from the most mature app (currently property-pulse or contractor-daily)
- Use same `package.json` dependencies
- Follow same folder structure

### 3. Maintain Consistency

- **Colors:** Use STYLE-GUIDE.md palette only
- **Components:** Follow .cursorrules patterns in each app repo
- **API patterns:** Follow ARCHITECTURE.md examples

### 4. Document Deviations

- If you must deviate, update Spec-Repo first
- Explain why in commit message
- Flag for review if significant

---

## Spec-Repo Maintenance

### When to Update Spec-Repo

- ✅ New architectural decision → DECISIONS.md
- ✅ New API pattern → ARCHITECTURE.md
- ✅ Style change → STYLE-GUIDE.md
- ✅ New component pattern → .cursorrules in relevant app repo
- ✅ New archetype → archetypes/[name].md
- ✅ New checklist → checklists/[name].md

### When NOT to Update

- ❌ Small bug fixes (just fix the code)
- ❌ Temporary experiments (document later if kept)
- ❌ Node-specific features (document in node's README)

---

## Base44 Workflow

### Testing Changes

1. Push code to GitHub
2. Wait for Base44 to auto-sync (~1-2 min)
3. In Base44, click "Preview" to test
4. Verify changes work
5. If good, click "Publish"

### Test Data Mode

- Base44 has a test data mode
- Use it to avoid affecting production data
- Toggle in Base44 settings

### Rollback

- If publish breaks something, rollback in Base44
- Or revert commit in GitHub and re-sync

---

## Quality Checklist

Before pushing code:

- [ ] Follows STYLE-GUIDE.md colors and patterns
- [ ] Uses .cursorrules patterns from app repo
- [ ] Matches ARCHITECTURE.md communication patterns
- [ ] Uses shared configs (tailwind, package.json)
- [ ] Follows node folder structure
- [ ] No light backgrounds anywhere
- [ ] Icons are gold or white only
- [ ] Any new decisions documented in DECISIONS.md
- [ ] Tested in Base44 Preview mode
- [ ] Commit message is descriptive

---

## Sync Between Spec-Repo and Google Docs

Since Claude reads Google Docs better:

1. **GitHub Spec-Repo** = Canonical source (version controlled)
2. **Google Docs Mirror** = Copy for Claude to read

### Keeping in Sync

When updating Spec-Repo:
1. Update in GitHub (via Cursor)
2. Copy to Google Docs mirror
3. Verify Claude can read the updated docs

### Files to Mirror

- README.md → Google Doc
- STYLE-GUIDE.md → Google Doc  
- ARCHITECTURE.md → Google Doc
- DECISIONS.md → Google Doc

---

## Troubleshooting

### Claude Can't Read Repo

- Use Google Docs mirror instead
- Paste specific file contents when needed
- Use screenshots for visual questions

### Cursor Not Following Spec

- Explicitly tell it to read the spec file first
- Copy relevant sections into your prompt
- Reference specific line numbers

### Base44 Not Syncing

- Check GitHub connection in Base44 settings
- Try manual sync button
- Verify push actually went through to GitHub

### Style Inconsistencies

- Run through STYLE-GUIDE.md checklist
- Compare against events-node reference
- Check for hardcoded colors (search for `bg-white`, `bg-blue`, etc.)

---

## Audit SOP

Standard operating procedure for new features:

| Stage | What Happens | Who |
|-------|--------------|-----|
| **Plan** | Strategy doc written, entities designed, security modeled | Claude Chat + Doron |
| **Audit (Pre-Build)** | Claude Code reviews spec against codebase patterns, flags gaps | Claude Code |
| **Revise** | Fix issues found in audit, update spec | Claude Chat + Doron |
| **Build** | Implementation in phases per spec | Cursor / Claude Code |
| **Audit (Post-Build)** | Security review, pattern compliance, test coverage | Claude Code |
| **Legal** | Update Terms of Service, Privacy Policy as needed | Doron + Legal review |
| **Ship** | Merge, publish, document in STATUS-TRACKER | Doron |

### Pre-Build Audit Checklist

Claude Code reviews:
1. Entity schema — field types, naming, FKs match existing patterns
2. Security model — RLS rules achievable, admin permissions clear
3. Admin integration — sidebar placement, routes, UI patterns
4. Hooks & data flow — query patterns, pagination, real-time needs
5. UI/UX consistency — placement, mobile, empty states
6. Organism integration — data supports future visualization
7. Gaps & risks — underspecified areas, edge cases, dependencies

### Post-Build Audit Checklist

Claude Code reviews:
1. Security rules implemented correctly
2. No console errors or warnings
3. Mobile responsive
4. Dark theme compliance (STYLE-GUIDE.md)
5. Loading and error states handled
6. Admin tools functional
7. No hardcoded values that should be config

### Legal Documentation Trigger

Any feature that introduces:
- New user-generated content → Terms of Service update
- New data collection → Privacy Policy update
- New third-party services → Privacy Policy disclosure
- Payment or financial data → Both + potential regulatory review

---

*This workflow ensures consistency across all nodes in the LocalLane ecosystem.*
