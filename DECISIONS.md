# Decision Log

> Every significant decision, with context and reasoning.  
> This is how we maintain institutional memory across conversations.

---

## How to Use This Document

When a decision is made:

1. Add a new entry at the TOP of the log (newest first)
2. Include: Date, Decision, Context, Options Considered, Reasoning, Consequences
3. Tag who made the decision

---

## Decisions

---

### 2026-01-27 | Next Archetype: Nonprofit/Church

**Decision:** The second archetype (after Events) will be nonprofit/church organizations.

**Context:** We need to prove the archetype system works by building a second type. Options included: mechanic shop, online store, restaurant, nonprofit.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Mechanic | Common business type | No immediate test case |
| Online Store | High demand | Complex (inventory, shipping) |
| Restaurant | Many local options | Complex (menus, ordering) |
| **Nonprofit/Church** | Real test case (Doron's church), similar data model to events | Narrower audience |

**Reasoning:** Doron has direct access to a church that currently uses paper/texts/conversations with no digital system. This provides:
- Real users with real pain
- Fast feedback loop
- Similar features to events (announcements, calendar, RSVPs)
- Proves the archetype system works

**Consequences:**
- Church node becomes priority after Event Node stabilizes
- We'll learn what's truly archetype-specific vs. universal

**Made by:** Doron + Claude

---

### 2026-01-27 | Tier 3 is Earned, Not Purchased

**Decision:** Partner Node status (Tier 3) cannot be bought — it must be earned through demonstrated trustworthiness.

**Context:** Most platforms sell premium tiers. LocalLane's differentiator is trust-based ranking.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Tier 3 purchasable | Revenue, simple | Undermines trust message |
| Tier 3 earned only | Strong differentiator, real trust | Slower revenue, need clear criteria |
| **Hybrid (earned + fee)** | Best of both? | Muddy message |

**Reasoning:** The entire platform premise is "trust over ads." If businesses can buy their way to Partner status, the message is hollow. Tier 2 generates revenue; Tier 3 is the reward for being genuinely trusted.

**Consequences:**
- Need to define measurable trust criteria
- Need to decide if criteria are universal or archetype-specific
- May slow initial revenue (Tier 2 is main revenue source)

**Made by:** Doron (strong preference) + Claude (agreed)

---

### 2026-01-27 | Punch Pass Launches in Demo Mode

**Decision:** The punch pass system will launch without real payment processing. Users will "purchase" with demo currency.

**Context:** Punch passes are a core feature, but real payments require: payment processor integration, Terms of Service, refund policies, liability considerations.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Skip punch pass for pilot | Simpler | Missing core feature |
| **Demo mode** | Full UX, no legal/payment complexity | Users might expect it to be free forever |
| Real payments immediately | Real revenue | Delays launch, legal risk |

**Reasoning:** Demo mode lets us:
- Test the full user experience
- Gather feedback on pricing, flow, acceptance
- Launch faster
- Defer payment processor integration to Phase 2

**Mitigation for "free expectation" risk:** Show real prices, generate receipts, make it clear this is "coming soon" pricing.

**Consequences:**
- No revenue from punch passes in Phase 1
- Need to track demo transactions as if real (for future migration)
- Phase 2 must include real payment integration

**Made by:** Doron + Claude

---

### 2026-01-27 | Spec-Repo is Source of Truth

**Decision:** All major decisions, architecture, and standards live in the Spec-Repo. This is the canonical reference.

**Context:** AI assistants (Claude, Cursor) don't have persistent memory. Every conversation starts fresh. Without a single source of truth, decisions get lost and contradicted.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Notion/Google Docs | Familiar, easy editing | Another tool, not in code workflow |
| README in each repo | Close to code | Fragmented, easy to miss |
| **Dedicated Spec-Repo** | Single source, in GitHub workflow, Cursor can read it | Extra repo to maintain |

**Reasoning:** Spec-Repo:
- Lives in the same GitHub ecosystem as code repos
- Can be read by Cursor before making changes
- Forces documentation discipline
- Is the "external brain" for AI assistants

**Consequences:**
- Must keep Spec-Repo updated (discipline required)
- Start new Claude conversations by pointing to Spec-Repo
- Cursor should read spec before implementing

**Made by:** Doron + Claude

---

### 2026-01-27 | Repositories Are Public (For Now)

**Decision:** The three GitHub repositories (events-node, community-node, Spec-Repo) are public.

**Context:** Claude needs to review code. Options are public access or manual copy-paste.

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| **Public repos** | Claude can access, faster iteration | Code visible to anyone |
| Private repos | Code protected | Slower workflow (copy-paste) |
| Temporarily public | Flexible | Hassle to toggle |

**Reasoning:** At this stage (pre-revenue, pre-scale), the risk of code theft is low. Real competitive advantage is:
- Local relationships
- Community trust
- Execution speed

Someone copying code would still need to build all of that.

**Consequences:**
- Faster iteration now
- Can switch to private when there's real traction
- Don't commit secrets/API keys to repos

**Made by:** Doron + Claude

---

## Template for New Decisions

```markdown
### YYYY-MM-DD | [Short Title]

**Decision:** [One sentence summary]

**Context:** [Why this decision was needed]

**Options Considered:**

| Option | Pros | Cons |
|--------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |

**Reasoning:** [Why we chose what we chose]

**Consequences:** [What this means for the project]

**Made by:** [Names]
```
