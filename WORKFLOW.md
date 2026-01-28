# Development Workflow

> How Claude, Cursor, GitHub, and Base44 work together to build the LocalLane ecosystem consistently.

---

## The Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Claude    │ ─► │  Spec-Repo  │ ─► │   Cursor    │ ─► │   GitHub    │ ─► │   Base44    │
│  (Strategy) │    │   (Truth)   │    │   (Code)    │    │  (Version)  │    │   (Live)    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
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

### Claude (Brain - Strategy & Architecture)

**Does:**
- Architecture decisions
- Spec writing and updates
- Strategic planning
- Code review via screenshots/paste
- Generating files for Cursor

**Doesn't:**
- Direct code editing
- GitHub pushes
- Browse repos (technical limitation)

### Cursor (Hands - Code Implementation)

**Does:**
- Reading all code files directly
- Implementing features
- Following Spec-Repo patterns
- Finding unused code
- Refactoring
- Pushing to GitHub

**Doesn't:**
- Make architectural decisions alone
- Deviate from spec without updating it

### Doron (Vision - Decisions & Testing)

**Does:**
- Final decisions on big questions
- Field testing with users
- Sales and relationship building
- Approving major changes

**Doesn't:**
- Need to write code
- Need to understand technical details

---

## Consistency Rules

### Before Starting Any Work

1. **Read the Spec-Repo first**
   ```
   Always start by reading:
   - README.md (overview, current status)
   - ARCHITECTURE.md (technical patterns)
   - STYLE-GUIDE.md (UI standards)
   - COMPONENTS.md (component patterns)
   - DECISIONS.md (constraints)
   - Relevant archetype spec (if building a node)
   ```

2. **Check for existing patterns**
   - Look at events-node for reference implementation
   - Copy shared configs (tailwind.config.js, package.json)
   - Follow established folder structure

3. **Update Spec-Repo when making decisions**
   - New decision? → Add to DECISIONS.md
   - Architecture change? → Update ARCHITECTURE.md
   - Style change? → Update STYLE-GUIDE.md

---

## Claude Conversation Starter

When starting a new Claude conversation, paste this:

```
I'm working on the LocalLane ecosystem. 

Spec-Repo Google Docs:
- README: [paste link]
- STYLE-GUIDE: [paste link]
- ARCHITECTURE: [paste link]

Current status:
- Community Node: Events display working, needs user accounts
- Event Node: Functional, syncing to Community Node
- Pilot target: Homeschool group

Before making any recommendations, please review these docs to understand:
1. The node-based architecture
2. The tier system (Standard → Storefront → Partner)
3. The "Gold Standard" design (dark theme, amber accent)
4. Existing decisions and constraints

Today I'm working on: [YOUR TASK]
```

---

## Cursor/Agent Instructions

When implementing code:

### 1. Always Reference Spec-Repo

- Read relevant spec files before coding
- Follow patterns exactly as documented
- When in doubt, check the spec

### 2. Use Shared Configurations

- Copy `tailwind.config.js` from events-node
- Use same `package.json` dependencies
- Follow same folder structure

### 3. Maintain Consistency

- **Colors:** Use STYLE-GUIDE.md palette only
- **Components:** Follow COMPONENTS.md patterns
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
- ✅ New component pattern → COMPONENTS.md
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
- [ ] Uses COMPONENTS.md patterns
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

*This workflow ensures consistency across all nodes in the LocalLane ecosystem.*
