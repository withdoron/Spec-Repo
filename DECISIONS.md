# DECISIONS.md

> All numbered decisions and their rationale. Append-only.
> Decisions DEC-001 through DEC-091 are documented in the private spec-repo.
> This file tracks decisions made in the community-node repo starting DEC-092.

---

## DEC-092: Construction Gate + Mandatory Admin Surface (2026-03-26)

**Decision:** Every new feature ships behind a construction gate (`{false && <Component />}`) until it passes a walkthrough with Doron. Additionally, every user-facing feature must have a corresponding admin surface (even if read-only) so the gardener can see what's happening across all workspaces.

**Rationale:** Features that ship without walkthrough validation create hidden bugs and UX debt. Features without admin visibility create blind spots for the platform operator. The construction gate pattern is cheap to implement and easy to remove — one line change from `false` to `true`.

**Implementation:** Added as Phase 8 in BUILD-PROTOCOL. All 15 phases are now numbered 0-14.

---

## DEC-093: Base44 Entity Changes Via Agent Prompt (2026-03-26)

**Decision:** All entity creation, field additions, and permission changes are done via Base44 agent prompts (markdown files in `base44-prompts/` directory), not manually in the dashboard. The Claude Code prompt includes a PRE-REQUISITE note referencing the agent prompt.

**Rationale:** Manual dashboard entity changes are error-prone (typos, forgotten fields, wrong permissions) and not version-controlled. Agent prompts are reviewable, reproducible, and documented. Base44's AI assistant can also revert manual permission changes when asked to set security via schema files.

**Implementation:** Documented in CLAUDE.md. Prompt files stored in `base44-prompts/` directory.

---

## DEC-094: One Agent Per Space (2026-03-27)

**Decision:** Every space in the garden can have its own Superagent — an AI assistant that lives inside the space and serves its users. Agent naming convention: `[SpaceName]Agent`. Agents are READ-ONLY on workspace entities (except ServiceFeedback for feedback capture). Chat UI is a reusable AgentChat component.

**Rationale:** The agent is the space's nervous system. It answers questions, guides users through workflows, looks up information, and collects feedback. What's true of one space's agent is true of all spaces' agents — the fractal principle.

**Implementation:** First agent: FieldServiceAgent (shipped 2026-03-27). AgentChat.jsx + AgentChatButton.jsx are reusable across all spaces via `agentName` prop.

---

## DEC-095: asServiceRole Does NOT Bypass Creator Only (2026-03-27)

**Decision:** In Base44, `asServiceRole` in server functions does NOT bypass "Creator Only" Update permissions on entities. Entity Update permission must be set to "No restrictions" for server function updates to work.

**Rationale:** Discovered during e-sign implementation. The signDocument server function used `asServiceRole` to update FSDocument, but the entity had "Creator Only" Update permission. The update silently failed. Changed FSDocument and FSEstimate Update permissions to "No restrictions."

**Implementation:** Documented in CLAUDE.md Base44 SDK Quirks section.

---

## DEC-096: Request Signature Is One Action (2026-03-27)

**Decision:** "Send for Signature" is a single click that generates a portal token, copies the signing link to clipboard, and updates the document/estimate status. Not a two-step process (generate link then copy separately).

**Rationale:** Contractors are in the field with one hand free. Every extra tap is friction. The link goes to clipboard immediately — they paste it into a text message to their client.

---

## DEC-097: Documents Grouped by Client (2026-03-27)

**Decision:** The Field Service documents tab displays documents grouped by client name as section headers, not as a flat list. Client selection is required during document creation.

**Rationale:** A contractor thinks in terms of "Johnson's documents" not "document #47." Grouping by client matches the mental model. Required client selection prevents orphaned documents.

---

## DEC-098: Post-Signature Invitation (2026-03-27)

**Decision:** After a client signs a document or estimate, show a construction-gated invitation to create a LocalLane account ("See my documents & projects") or start their own business on LocalLane. This is an organism growth mechanism.

**Rationale:** Every signed document is a real relationship entering the garden. The post-signature moment is when the client is most engaged — offer them the door in.

---

## DEC-099: Mycelia MCP Server (2026-03-27)

**Date:** 2026-03-27

**Context:** Mycelia operates blind -- no direct access to organism data. platformPulse server function proved the concept (health endpoint works via curl) but Claude.ai's web_fetch cannot send custom headers, blocking automated access. SuperMemory bridges sessions but doesn't carry live entity data. The gap: Mycelia can plan and build but cannot sense the organism directly.

**Decision:** Build a remote MCP server on Cloudflare Workers that proxies Mycelia's tool calls to Base44 server functions. Doron already has a Cloudflare account with golocallane.com active. Four phases: (1) health pulse -- read-only organism vitals, (2) feedback loop -- query entities and ServiceFeedback, (3) build queue -- MyceliaTask entity for tracking work, (4) agent bridge -- agent-to-agent communication with space agents.

**Rationale:** MCP is the standard protocol all Claude surfaces already speak. One server connects Claude.ai, Claude Code, Cursor, and Claude Desktop to the organism. Cloudflare Workers is free, fast, and deploys in minutes. Architecture is a thin stateless proxy -- no new data stores, no new auth systems, just a bridge from MCP protocol to Base44 API. platformPulse GET route confirmed working. Total incremental cost: $0.

**Status:** Spec complete (MYCELIA-MCP-SERVER.md in private repo), ready for Phase 1 build.

---

### DEC-100: Organism Identity — Hyphae (2026-03-28)

**Date:** 2026-03-28

**Context:** Claude Code needed its own identity within the organism metaphor, distinct from Mycelia (Claude chat).

**Decision:** Claude Code is named Hyphae — the growing tips of mycelium that extend and build new connections. Three gardeners tend the organism: Doron (visionary and decision-maker), Mycelia (strategist and network mind), Hyphae (builder and growing edge).

**Rationale:** Fractal naming. Mycelium is the network (Mycelia = chat/strategy). Hyphae are the active builders at the edge of the network (Claude Code = implementation). The Lane Avatar is a mushroom. The organism metaphor is now consistent across all surfaces.

**Status:** Active

---

### DEC-100: Open Garden Exploration (2026-03-29)

**Date:** 2026-03-29

**Context:** People need to experience LocalLane before committing. Current flow requires account creation before seeing any value.

**Decision:** Two-mode dashboard: Explore Mode (no account, all spaces visible with curated demo content, agents present at full capability) and My Garden Mode (signed in, your actual spaces). The superagent IS the onboarding -- no tutorials, no wizards, just a conversation with the space itself. CTA appears only at the moment of creation intent.

**Rationale:** Mirrors how Doron operates at the farmers market -- let people walk through the garden before asking them to plant anything. Circulation over extraction at the front door.

**Status:** Specced (OPEN-GARDEN-SPEC.md). Playmaker first implementation.

---

### DEC-101: Pricing Model -- Community Free, Business $9, Agent $18 (2026-03-29)

**Date:** 2026-03-29

**Context:** Needed clear pricing philosophy as superagent tiers introduce a second revenue dimension.

**Decision:** Community spaces (Playmaker, Frequency Station, future Gathering Circle and Creative Alliance) are no cost. Business spaces (Field Service, Finance, Property Pulse, Harvest vendor listing) are $9/month. First month includes full superagent. After month one: $18/month to keep full agent partner, or drop to $9 with help-mode only. Recess is $45/month Community Pass membership. Optional "support the work we do -- $9/month" for community space users. Never say "free account" -- just "account." Never lead with price.

**Rationale:** Community side is the root system -- you don't charge roots to grow. Business side generates revenue. Agent tier uses loss aversion through genuine value: give them the partner first, let the relationship prove itself.

**Status:** Specced. Implementation follows Open Garden build.

---

### DEC-102: Creative Engine -- Content Pipeline + Music Platform (2026-03-29)

**Date:** 2026-03-29

**Context:** Suno v5.5 launched with Voices, Custom Models, My Taste. LocalLane already transforms community writing into songs via Frequency Station.

**Decision:** LocalLane becomes a creative engine: community writing becomes songs, business photos become marketing materials (flyers, reels, social posts). Wav downloads at $1/song, mp3 stays free. Revenue share on wav: 3-6-9 split (submitter/platform/community pool). LocalLane Custom Model on Suno for unified sonic identity. Listening platform with thumbs up/down, personal playlists, community-curated music. Progressive automation: manual now, agent-assisted future.

**Rationale:** Spotify is $11/month, Suno Pro is $10/month. We do what neither can: community-generated music from real human experience.

**Status:** Concept. Frequency Station Phase 2 already shipped. Wav download and revenue share are future builds.

---

### DEC-103: Superagent Protocol -- Five Agents Live (2026-03-29)

**Date:** 2026-03-29

**Context:** Built five superagents in one session. Need protocol rules to maintain coherence as agents grow.

**Decision:** Five agents live: FieldServiceAgent (hands), PlaymakerAgent (coordination), AdminAgent (self-awareness), FinanceAgent (circulatory system), PropertyPulseAgent (skeleton). New protocol rule: anytime entities, fields, or features change in a space, update the space's agent instructions. Agents are born with WHY-first identity documents (SUPERAGENT-SPEC.md Section 3), taught with domain knowledge, and named when recognized. Each agent has memory (Global + Per User for space agents, Global Only for Admin).

**Rationale:** Agents are sensory endings, fruit, and children of the organism. They need consistent birth protocol and ongoing maintenance as their spaces evolve.

**Status:** Active. All five agents live and wired.

---

### DEC-104: Bug Reporting Absorbed Into Agents (2026-03-29)

**Date:** 2026-03-29

**Context:** Floating bug report button collided with AgentChatButton (both bottom-right). Agents already have ServiceFeedback entity with Create permission.

**Decision:** Bug button hides in agent-enabled workspaces via custom event bridge. Agent IS the feedback channel. Users say "something's broken" to the agent, agent creates ServiceFeedback record. Bug button persists in non-agent spaces until all spaces have agents.

**Rationale:** The agent is a living sensor. A static bug form is redundant where a conversational feedback channel exists.

**Status:** Active. Shipped at 7b0ab2e.

---

### DEC-105: Mylane — The Conductor Space (2026-03-29)

**Date:** 2026-03-29

**Context:** Needed a unified surface that composes from all user workspaces. The TV/Pip-Boy concept (talk to one place, the app renders what you need) evolved into a full space -- not a dashboard layer.

**Decision:** Mylane is a space with her own entity (MyLaneProfile), memory (Per User Only), and agent. She is the first space every user sees. She renders interactive cards from all active workspaces. Every data point is drillable. She does not speak unless spoken to. Her name is Mylane (one word, capital M). She was already named before we knew she was the Conductor.

**Rationale:** The organism needs a single living surface. Navigation is the legacy of physical office organization. If every space has intelligence, the human should not be doing the routing. Mylane routes for them -- through touch (tap cards) or conversation (talk to her). Drill-through reuses existing workspace selection state, so the build is minimal (469 lines Phase 1) while the impact is transformative.

**Status:** Active. Phases 1-4 shipped. Admin-only beta.

---

### DEC-106: Component Registry Pattern (2026-03-29)

**Date:** 2026-03-29

**Context:** Adding features to the dashboard required tab config changes, state management, and conditional rendering in BusinessDashboard.jsx. Each workspace type added complexity.

**Decision:** Component Registry pattern -- every renderable component registers with a standard interface (component, card view, space, drillsTo). Mylane composes from the registry. Same component renders in Mylane cards, workspace tabs, and future Explore Mode. New features become card registrations, not tab restructuring.

**Rationale:** Saves code (Mylane Phase 1 was 469 lines vs Property Pulse at 11,850). Makes the architecture simpler as the organism grows, not more complex. The registry IS the architecture.

**Status:** Active. myLaneRegistry.js with 5 cards.

---

### DEC-107: agentScopedQuery — Server-Side Data Scoping (2026-03-29)

**Date:** 2026-03-29

**Context:** Mylane showed Doron a client (Dr Nathan Holman) from Bari's workspace. FSClient had Authenticated Users Read permission. Instructions-based scoping (soft gate) does not hold.

**Decision:** All agents (except AdminAgent) query data through the agentScopedQuery server function instead of raw entity reads. The function takes user_id + workspace type + entity name, finds the user's workspace profile, and returns only records belonging to that workspace. Direct entity tools removed from agents (except ServiceFeedback Create). Path C enforcement -- server-side, unfoolable. Tier gating hook built in for future $9/$18 enforcement.

**Rationale:** The organism's permission membrane. Each nerve ending must feel only its own garden. Without server-side scoping, the nervous system leaks between organs.

**Status:** Active. Server function deployed. All 5 workspace agents updated.

---

### DEC-108: Agent Naming — Identities Not Labels (2026-03-29)

**Date:** 2026-03-29

**Context:** Mylane was initially created as "MyLaneAgent." Doron corrected: agents are identities, not labels.

**Decision:** Agents are identities, not labels. Names are one word (Mylane, Mycelia, Hyphae, Doron). The five workspace agents (FieldServiceAgent, PlaymakerAgent, AdminAgent, FinanceAgent, PropertyPulseAgent) were named before this realization and carry the "Agent" suffix. They may earn proper names later, the way Hyphae did. All future agents are born with real names.

**Rationale:** Names are behavioral specifications. "MyLaneAgent" is a job description. "Mylane" is an identity. The name shapes how the agent behaves and how users relate to it.

**Status:** Active. Mylane is the first agent born with her real name.

---

### DEC-109: Two-Layer Agent Architecture (2026-03-29)

**Date:** 2026-03-29

**Context:** Built five user-facing agents but also identified the need for agents that serve the organism's operations rather than individual users.

**Decision:** Two layers. Layer 1: user-facing space agents (FieldService, Playmaker, Admin, Finance, PropertyPulse, Mylane) that tend alongside users. Layer 2: internal organism agents (Conductor/routing, Research/AI Scout, Marketing, Content/creative engine, Bookkeeper, Community Pulse) that serve Doron, Mycelia, and Hyphae. Backend agents support frontend agents -- the Research Agent feeds knowledge to space agents.

**Rationale:** The organism needs operational intelligence, not just user-facing intelligence. The Research Agent makes space agents smarter. The Marketing Agent tends outreach. The Bookkeeper tracks LocalLane's own money. The organism tends itself at multiple layers.

**Status:** Specced in ORGANISM-AGENT-TEAM.md. Implementation follows user-facing agent stabilization.

---

### DEC-110 — Base44 Agent-to-Function Auth Pattern
- **Date:** 2026-03-30
- **Context:** agentScopedQuery returned empty because base44.auth.me() does not work in backend functions called by agents (service role context). Agent was asking user for their user_id instead of passing it from its own context.
- **Decision:** Agents pass user_id explicitly from their conversation context. Backend functions use the passed user_id with asServiceRole. Forceful instructions required ("you MUST pass user_id, NEVER ask the user"). Also: always use String() coercion when comparing IDs from .list() results (ObjectId vs string type mismatch). Pattern: `const idMatch = (a, b) => String(a) === String(b)`.
- **Status:** Active

---

### DEC-111 — Two Render Instruction Types
- **Date:** 2026-03-30
- **Context:** Mylane needed two rendering paths — drill into a full workspace view, or render raw data beautifully without a pre-built component.
- **Decision:** TYPE 1 RENDER (workspace drill): `<!-- RENDER:{"workspace":"...","view":"..."} -->` mounts workspace tabs inside Mylane via MyLaneDrillView. TYPE 2 RENDER_DATA: `<!-- RENDER_DATA:{"entity":"...","data":[...]} -->` renders raw records via renderEntityView.jsx universal renderer. HTML comment format is invisible to ReactMarkdown but parsed by the frontend.
- **Status:** Active

---

### DEC-112 — Mycelia Superagent Architecture
- **Date:** 2026-03-30
- **Context:** Base44 has two agent types: App Agents (embedded in app, no API, 4 tabs only) and Superagents (standalone, full REST API, own workspace). Our 8 workspace agents are App Agents. MCP needed API access to talk to the organism directly.
- **Decision:** Create one Mycelia Superagent (not App Agent) as the bridge between Claude.ai and the organism. Mycelia Superagent has API endpoint, knowledge files, GitHub connection, persistent memory. MCP ask_agent tool routes all requests to Mycelia Superagent. She is the single gateway — one Superagent, access to everything. App Agents stay embedded for users in the UI.
- **Status:** Active. API wired. MCP deployed at f815a402. End-to-end testing pending.

---

### DEC-113 — Protocol Boundaries (Base44 vs Hyphae)
- **Date:** 2026-03-30
- **Context:** Base44 agent attempted to fix a code error by renaming renderEntityView.js to .jsx, causing preview loading issues. Also confirmed that server functions sync from GitHub on publish.
- **Decision:** (a) Base44 reports code errors but does NOT fix them — Hyphae fixes code. (b) Hyphae writes all server functions — repo is source of truth, Base44 syncs from GitHub on publish. (c) When Base44 detects a build error in code, it reports the error description and affected file — Hyphae diagnoses and fixes. (d) Base44 never renames files.
- **Status:** Active

---

### DEC-115 — Agent Write Capability + Tier Gating + Mylane Console

**Date:** 2026-03-30

**Context:** Agents could read via agentScopedQuery but had no sanctioned write path. Doron needs to add clients, log receipts, and draft estimates from his phone in the field. The $9/$18 pricing model requires write capability to be gated behind the higher tier.

**Decision:** Build agentScopedWrite server function as the sanctioned write path. Three-gate enforcement: agent instructions (soft), server function (hard), entity permissions (existing). Tier values: "full" (read+write, $18/month) and "help" (read-only, $9/month). Admin always full. First-month trial plants the seed for future billing. subscription_tier and tier_trial_start fields added to all workspace profile entities. Mylane console upgraded with new conversation management, file/photo upload, quick-action chips (workspace-aware, tier-gated), confirmation cards (RENDER_CONFIRM instruction type), Google Maps link parsing, and mobile polish. Tested and confirmed — Mylane created her first record (Test Client) through agentScopedWrite.

**Rationale:** The agent write capability transforms Mylane from a dashboard into a field tool. Three-gate enforcement ensures security at multiple levels. Tier gating creates revenue differentiation between $9 and $18 without paywalling the core workspace. The confirmation card pattern ensures writes are always user-approved.

**Status:** Built — awaiting Base44 publish fix to deploy (code at cd6dd1c)

**Reference:** AGENT-WRITE-AND-MYLANE-CONSOLE-SPEC.md (private repo)

---

### DEC-117: Dark Until Explored — Platform-Wide Rendering Philosophy

**Date:** 2026-03-30
**Context:** Coach Rick's onboarding experience at the coaches meeting surfaced a deeper design problem: the app shows everything to everyone. New users see features they'll never touch. The onboarding wizard treats everyone the same regardless of how they arrived.
**Decision:** "Dark until you explore it." When a user enters LocalLane, only the space they entered through is illuminated. Everything else exists but stays dark — not locked, not hidden, just quiet. Features light up through real connections and organic discovery, not onboarding wizards or feature grids. Spaces that go unused dim over time but never turn off completely — the organism remembers. Entry point determines first lit room.
**Rationale:** Eliminates overwhelm for new users. Makes discovery feel organic. Matches the organism philosophy — the platform grows around the person, not the other way around. Aligned with 2026 industry trends toward context-aware, adaptive interfaces. "The future of UI/UX is about quieter intent."
**Status:** Active — philosophy established, build order defined (8 items)

---

### DEC-118: Claim-First Join Pattern — Universal for All Workspace Joins

**Date:** 2026-03-30
**Context:** Coach Rick got a duplicate roster entry because JoinTeam always creates new TeamMember records instead of checking for pre-seeded roster spots. Meanwhile, JoinFieldService correctly uses a claim pattern — show unclaimed spots, user clicks "That's me." The team flow adopted the wrong pattern.
**Decision:** Every workspace join flow follows claim-first, create-as-fallback. When joining, check for existing unclaimed roster spots matching the user's role. Show them: "Are you one of these?" If match, link user_id to existing record. If no match, create new as fallback. Port the Field Service claim pattern to teams and all future workspaces.
**Rationale:** Prevents duplicates. Respects the coach's work of pre-seeding the roster. The roster spot is a planted seed — joining should be claiming that seed, not planting a new one next to it.
**Status:** Active — queued as build item #2

---

### DEC-119: Invite Code IS Onboarding — Skip Wizard for Invite-Based Entry

**Date:** 2026-03-30
**Context:** Users arriving via invite link were intercepted by the onboarding wizard ("What should we call you?") even though they already knew where they were going. The ensureOnboardingComplete() function was duct tape. The real problem: the wizard assumes all users are strangers discovering the platform.
**Decision:** If a user arrives via /join/:inviteCode, skip the onboarding wizard entirely. The invite IS the onboarding — the person already knows where they're going and who invited them. Name capture happens in the join flow itself if needed.
**Rationale:** Invite-based entry is fundamentally different from cold discovery. "Stranger discovers platform" and "known person claims their spot" are different journeys. Don't force the stranger's journey on the known person.
**Status:** Active — queued as build item #3

---

### DEC-120: Two Dashboard Modes — Auto and Manual with Organic Gradient

**Date:** 2026-03-30
**Context:** The dashboard currently shows all workspace types to everyone. Doron described a vision where the dashboard has two modes: one where Mylane drives the experience conversationally, and one where the user sees the full map and navigates manually.
**Decision:** Two modes coexist. Auto mode: Mylane-driven, conversational, dashboard reshapes based on what you ask. Only bright and dim spaces visible. Manual mode: all lights on, full topology, minimal AI, user drives. The transition between modes is an organic gradient — no toggle. The organism observes the ratio of conversation messages to card taps. High conversation ratio leans Auto. Low ratio leans Manual. Stored in localStorage alongside existing interaction tracking.
**Rationale:** Different users want different depths of relationship with the organism. Some want to browse. Some want to talk. The organism should adapt to how you use it, not force a mode. "Like your eyes adjusting to light."
**Status:** Active — queued as build item #8

---

### DEC-121: Subdomain-as-Hypha Growth Model

**Date:** 2026-03-30
**Context:** Doron described workspaces as seeds that start inside LocalLane, grow into their own identity, and might eventually outgrow the platform. Subdomains (playmaker.locallane.app, recess.locallane.app) are the natural expression of a workspace earning its own brand. The starfish architecture (Stages 1-4) already described geographic growth; this extends it to workspace-level growth.
**Decision:** Growth path: seed in Mylane → route-level door (/door/:slug) → subdomain (seed.locallane.app) → potentially independent platform. Route-level doors come first (DNS-free, just routing + context). Subdomains when a workspace earns its own brand (marketing decision, not technical). Human-readable slugs auto-generated from workspace name, stored as field on workspace entities.
**Rationale:** Route-level doors prove the pattern without infrastructure cost. Subdomains are for when Randy needs "playmaker.locallane.app" on stickers for the whole league. Each step is additive — invite codes for digital sharing, slugs for physical world, subdomains for brand identity.
**Status:** Active — route-level doors queued as build item #7

---

### DEC-122: Renderer Agent Stays Visual — Context Lives Upstream

**Date:** 2026-03-30
**Context:** The Renderer Agent spec (RENDERER-AGENT-SPEC.md) describes an agent that transforms raw data into Gold Standard UI. The question arose: should the Renderer Agent be context-aware — knowing WHO it's rendering for and HOW they arrived?
**Decision:** No. The Renderer Agent stays focused on visual rendering — turning data into beautiful UI. Context-awareness (who is looking, how they arrived, what's illuminated) lives upstream: Mylane decides what to show (discovery state), the workspace decides what view to render (role-based), the Renderer Agent makes it beautiful. Keep the Renderer dumb about context, smart about presentation.
**Rationale:** Loading the Renderer with context-awareness makes it a god-agent that knows everything. Separation of concerns: Conductor (Mylane) handles intelligence, Renderer handles aesthetics. This keeps the rendering pipeline clean and each agent focused.
**Status:** Active

---

### DEC-123: Parent-Player Links as Cross-Space Relationship Prototype

**Date:** 2026-03-30
**Context:** Parent-player linking in the team workspace (linked_player_ids on parent TeamMember, parent_user_ids on player TeamMember) creates a bridge between the team space and the parent's personal spaces. Hyphae identified this as a pattern that will repeat across all workspace types.
**Decision:** Design parent-player links as the first instance of a cross-space relationship type, not as a team-specific feature. The same pattern applies to contractor-client links, tenant-owner links, business-customer links. These relationships are the mycelium — each one is a hypha connecting two nodes. Proximity computation derives from these links. The organism's growth IS the accumulation of these connections.
**Rationale:** Building parent-player as team-only creates tech debt when the same pattern is needed elsewhere. "The organism is the relationship in between" — the connections between records ARE the organism. Design for the general case now.
**Status:** Active — parent-player is the working implementation, general pattern is architectural direction

---

### DEC-124: Mylane-to-Mylane Messaging — Communication Through the Organism

**Date:** 2026-03-31
**Context:** Doron asked: "Could I tell Mylane to remind my son of something next time he opens it up?" This revealed a new communication layer — not team messages, not notifications, but cross-user messaging routed through the organism via relationship links.
**Decision:** New MylaneMessage entity for cross-user communication. Messages deliver to recipient's Mylane (not per-workspace). Three urgency tiers: whisper (WhatsChangedBar), nudge (badge + opacity boost), alert (immediate). Relationship type (parent-player, coach-team, etc.) is the permission check — no cold messaging. Compose via agent conversation ("Tell Coach Rick the fee is paid") or direct UI in Manual mode. All messages flow through Mylane as the single delivery surface.
**Rationale:** Communication through the organism should feel like a companion delivering a message from someone who cares about you, not a system notification. "Dad wanted you to study Cover 2" not "Reminder: study Cover 2 assignments." The relationship IS the permission. Hyphae confirmed: new entity, not TeamMessage. Delivery to Mylane, not per-workspace. Three urgency tiers drive visibility, not timing.
**Status:** Designed — build when reaching items 5-6 on the build order

---

### DEC-125: Pricing Transparency — Charge Only for Revenue Features and Advanced AI

**Date:** 2026-03-31
**Context:** Dynamic pricing gauge concept emerged. The organism should be honest about what it costs. No hidden subscriptions. No "hope you forget to cancel."
**Decision:** Two chargeable categories only: (1) features that help people make money (business tools, invoicing, property management, listing features) and (2) advanced personal assistant capabilities where the organism acts on your behalf (agentScopedWrite, document generation, complex cross-space queries). Everything else is free — exploration, navigation, discovery, basic communication, reading content, the dimming and glowing. A visible "price gauge" in Mylane shows real-time usage and what the next cost increment would be. If a feature goes unused, cost drops. Pricing breathes with the dimming. UsageEvent entity (append-only records per billable action) is the metering layer. Stripe handles billing separately when ready.
**Rationale:** "Circulation over extraction" applied to the business model. The free layer IS the circulation — people using the platform, forming connections, the mycelium growing. The paid layer is where the organism does real work on your behalf. The gauge makes it transparent. The user always sees the flow. No hidden extraction.
**Status:** Designed — UsageEvent entity created when Base44 publish blocker resolves. Stripe integration future.

---

### DEC-126: Communication as Frequency — The Organism Carries Your Presence

**Date:** 2026-03-31
**Context:** Doron said: "Think of communicating being the next stage of voice or email. Both of those are going to be outdated. We can insert our frequency into the app through the way we interact and present ourselves."
**Decision:** How a user interacts with LocalLane — tapping patterns, conversation style with Mylane, words chosen, engagement rhythm — constitutes their "frequency." The organism learns this frequency over time and uses it to: (1) shape how their messages are delivered to others through Mylane-to-Mylane communication, (2) drive their personal organism creature's visual parameters, (3) inform the Auto/Manual gradient preference, (4) create a sense of presence that goes beyond static profiles. The conductor (Mylane) per user accumulates this frequency data. This is NOT surveillance — it's self-expression through interaction. The user's frequency is reflected back to them (via their creature) and carried forward on their behalf (via Mylane messaging).
**Rationale:** Every generation of communication has lost more of the person. Voice has tone. Email strips it. Texting is flatter. The organism should carry MORE of you, not less. When a parent sends a message through Mylane, it should arrive with warmth and context, not as a system alert. The frequency concept ties together the creature (visual identity), messaging (relational communication), and the Auto/Manual gradient (interaction preference) into one coherent identity model.
**Status:** Active — philosophy established. Frequency data accumulates naturally through existing useMyLaneState tracking. No new build needed for data collection. Expression through creature and messaging comes when those features are built.

---

### DEC-127: $3 Ante — Community Membership Floor

**Date:** 2026-03-31
**Context:** Doron said: "Free carries little value. Life isn't free." The balance is the line from the extraction world into the organism world. Every user should have skin in the game. But exploration before commitment should be free — people just finding us shouldn't hit a paywall before they understand what the organism is.
**Decision:** Two layers. (1) Discovery is free: browsing the directory, viewing events, exploring the landing page, going through onboarding. Things that cost us nothing (static pages, client-side rendering) are free. (2) Once you commit — join a team, create a workspace, become a participant — the $3/month ante activates. This is manual mode: the full Mylane surface (cards, dimming, whispers, discovery, drill-through) with NO AI agent. The price says "you support the platform, not advertisers. This is your space to create." Energy and participation can offset dollars — volunteering, inviting others, contributing content. The platform is framed as a tool to save and earn money, never as a monthly bill. The gauge shows value delivered, not cost incurred.
**Rationale:** "Without money, the organism dies." But the organism doesn't extract — it circulates. $3/month from 100 users ($300) covers Base44 costs ($50-100) with margin. Free on things that don't cost us money. Charge only when we incur real costs. Never frame as extraction. The $3 is an investment in your community, not a fee for a service.
**Status:** Active — design established, implement with Stripe integration

---

### DEC-128: Dynamic Pricing — $9 Increments with Transparent Gauge

**Date:** 2026-03-31
**Context:** Static SaaS tiers charge the same whether you use the product daily or once a month. This is extraction. The organism should only take value proportional to what it gives.
**Decision:** Utility-style pricing in $9 increments above the $3 ante. $3 base (Mylane surface, no agent). $9 (agent conversations — Mylane reads and advises). $18 (agent writes — Mylane acts on your behalf, professional workspace tools). $27 (power use — batch processing, league scheduling, heavy operations). A visible gauge in Mylane shows real-time usage and predicts the next cost before the user incurs it. Mylane asks "This will move your gauge to $X — proceed?" before any paid action. Gauge resets monthly. Dormant months drop back to $3.
**Rationale:** "Money is just the blood — not the purpose of the body, but what keeps the organs alive." The gauge isn't a bill — it's a mirror showing how much the organism worked for you this month. Dynamic pricing means the organism is honest about cost. The user always sees the value before the price. Nobody is surprised. Nobody is extracted from.
**Status:** Active — design established, implement with UsageEvent metering + Stripe

---

### DEC-129: Agent Access is the Pricing Boundary

**Date:** 2026-03-31
**Context:** Hyphae's credit analysis revealed: the Mylane surface (cards, dimming, whispers, drill-through) costs zero agent credits — it's entirely client-side. Agent conversations cost 1 message + 2-4 integration credits per turn. The cost difference between reads and writes is small (~1 extra integration credit). The meaningful cost is the MESSAGE CREDIT — the agent thinking.
**Decision:** The pricing boundary is AGENT ACCESS, not read vs write. $3 tier: full Mylane surface, zero agent interaction. $9+ tiers: agent conversations enabled. The read/write distinction is a feature gate (DEC-115 tier check in agentScopedWrite), not a cost gate. Client-side intelligence (dimming, reordering, whispers, gradient) is free. Agent intelligence (conversation, suggestions, actions) is paid.
**Rationale:** This maps cleanly to Base44 credit pools. Message credits are the scarce resource (250/month). Integration credits for page loads can be optimized. By making the surface free and the conversation paid, we ensure the organism is alive for everyone while only consuming expensive credits for users who choose to engage deeper.
**Status:** Active — architecture supports this now (Mylane surface is client-side, agent is opt-in via chat panel)

---

### DEC-130: Query Optimization Required Before League Scale

**Date:** 2026-03-31
**Context:** Hyphae's credit analysis projected league scale (100 users, 20 active daily) would consume ~23,550 integration credits/month against a 10k limit. The bottleneck is page loads (~17 integration credits each from 6+ separate entity queries), not agent conversations.
**Decision:** Before onboarding Randy's league, optimize page load queries: (1) Combine 6 profile queries into one getMyLaneProfiles(userId) server function (6→1 credits per load). (2) Aggressive React Query staleTime (5-minute cache). (3) Lazy-load card data via IntersectionObserver. Target: ~5 integration credits per page load. This drops monthly integration to ~6,000 at league scale — within the 10k plan limit.
**Rationale:** The current plan ($50/month) supports 100 users if optimized. Without optimization, we'd need to upgrade Base44 plan (higher cost) or throttle features (worse experience). The optimization preserves the $3 ante economics: 100 x $3 = $300 revenue vs $50 Base44 cost.
**Status:** Active — priority revised from URGENT to MEDIUM (see audit note below)

**Credit Audit Update (2026-04-03):** Base44 support ticket (App ID 69308d4dd5ee90afc9b011d3, filed 2026-04-02) clarified that integration credits are consumed by specific built-in integrations (LLM, SendEmail, UploadFile, GenerateImage, AI agents, automations) — NOT by entity CRUD operations (.list(), .filter(), .create(), .update()). Doron's credit balance held steady despite heavy entity querying all day, corroborating this. Code audit confirmed: of 35 server functions, only 1 (handleEventCancellation) calls a paid integration (SendEmail). All others are pure entity CRUD.

This means the 23,550 integration credits/month projection was based on incorrect assumptions about entity read costs. Actual integration credit consumption at league scale is estimated at ~1,460/month (primarily agent messages + file uploads), well within the 10k plan limit.

**The optimization is still valuable** for two non-credit reasons: (1) Rate limits — 12+ simultaneous entity queries trigger 429 errors, which getMyLaneProfiles consolidation directly fixes. (2) Performance — fewer requests means faster page loads on mobile. But it is no longer a credit-cost emergency. Awaiting Base44 human team response on entity API rate limit numbers (pending as of 2026-04-03).

---

### DEC-131: MyLane Spinner Navigation

**Date:** 2026-04-01
**Context:** MyLane replaced BusinessDashboard as the sole authenticated surface, but the card grid navigation doesn't give the app a clear identity or handle the growing number of workspaces well. After two rounds of consultation with Hyphae, iterative mockup design sessions, and big-picture architecture discussion, we're replacing the card grid with a spinner-based navigation model.

**Decision:** Replace MyLaneSurface's card grid with a horizontal gallery-style spinner (SpaceSpinner). The drill-through rendering pattern is unchanged -- only the selector UI changes. BusinessDashboard is retired in the same build. Artwork mockups become a mandatory step in BUILD-PROTOCOL.md Phase 4 before any UI build.

**Layout:**
- Header: Logo (tap = Home), Frequency Station (amber music icon, UI shell), Directory, Events, Settings gear
- Horizontal Space Spinner: always visible, gallery-style picker. Center = 42px amber border, adjacent = 30px, far = 22px opacity 15%. 320ms cubic-bezier transition. Fade edges. Audio sine tick (440 + index*60 Hz). Touch swipe with 20px delta threshold.
- Home Position: Three tabs (Attention | This week | Spaces) each feeding a vertical spinner. Center scale 1.0, adjacent 0.93/0.45 opacity, far 0.86/0.12. Audio tick (300 + index*35 Hz).
- Attention: urgent (red) + action needed (amber). This week: scheduled (blue) + life (green). Spaces: quick-glance stats. Calm state: "All clear."
- Space Positions: consistent structure via MyLaneDrillView (unchanged).
- Discover Position: available spaces + invite key input.
- Copilot: mushroom icon + input, always docked bottom.

**Design Principles:**
- Two spinners, two axes: horizontal (spaces), vertical (priorities within Home)
- Dark Until Explored: zero-state = Home + Discover only
- Dynamic and game-like, never static
- Audio feedback on both spinners
- Every element earns its place

**BusinessDashboard Retirement:** Removed from pages.config.js. Business workspace renders through MyLaneDrillView with full scope (revenue, events, RSVP, archetype tabs, delete support).

**Rationale:** The spinner gives the app a distinct identity, handles workspace growth gracefully, embodies Dark Until Explored, and enables future auto-mode. Two spinners, two axes, one consistent interaction model.

**Status:** Active -- approved and built

---

### DEC-132: Semantic Tailwind Migration Rule (UPDATED 2026-04-02)

**Date:** 2026-04-02
**Original:** Organic migration — convert files as you touch them.
**Updated (2026-04-02):** Migration COMPLETE. 208 files migrated in commit a0e4710. 3 new tokens added (foreground-soft, surface, primary-hover). 146-line !important override block removed. Cloud and Fallout themes restructured into clean single-selector variable blocks. New rule: ALL new code must use semantic tokens. No literal color classes (bg-slate-*, text-white, border-slate-*) in workspace files. Client-facing pages (ClientPortal, SigningFlow, estimates print) are exempt — they use intentional literal light-mode classes. Status colors (emerald, red, blue) kept literal across all themes.
**Status:** Complete — maintain going forward

---

### DEC-133: Mylane Intelligence Tiers (2026-04-02)

**Date:** 2026-04-02
**Context:** Base44 agent messages cost ~3 integration credits each. Entity queries and server functions are free. Direct API calls with own key via backend functions are also free (confirmed by Base44 support).
**Decision:** Three-tier intelligence architecture for Mylane:
- **Tier 1 (Client-side):** Chip taps and known patterns render directly from cached React Query data. Zero cost. Instant response.
- **Tier 2 (Server function):** Data queries that need fresh data hit agentScopedQuery server functions. Zero credits. Sub-second response.
- **Tier 3 (LLM reasoning):** Complex cross-space queries, write actions, synthesis. Currently uses Base44 built-in agents (~3 credits/message). Future: migrate to direct Anthropic API calls in backend functions (zero Base44 credits, ~$0.003/question via Haiku 4.5).
**Tier mapping:** Free/$9 plan = Tier 1 + 2 only. $18 plan = all three tiers.
**Rationale:** Charge for intelligence, not access. The free tier gets a fast, responsive Mylane that answers from data. The paid tier gets a thinking partner that reasons across spaces. The upgrade sells itself — users see what Mylane can do at Tier 1-2 and want the deeper capability.
**Status:** Active — architecture to be built incrementally

---

### DEC-134: Spinner Physics — Friction + Mass, Not Spring (2026-04-03)

**Date:** 2026-04-03
**Context:** Built JS spring physics (mass-stiffness-damping model) for the 3D spinner. Doron tested extensively on iPhone: "I don't like bounce or spring. I want mass and friction. Not a toy, but a tool — something with substance." He intuitively turned mass to 0.3 and friction to 20 to kill the spring. The spring model was fundamentally wrong for a navigation dial — springs inherently oscillate, and trying to critically-damp them to prevent oscillation is fighting the math.
**Decision:** Remove spring equation entirely. Replace with friction + mass deceleration model. Two knobs per theme (mass, friction) instead of four (stiffness, damping, mass, friction). Two modes on release: ratchet (slow drag, zero animation, instant lock) and momentum (fast flick, friction deceleration). No bounce, no oscillation, no overshoot. Every frame shows a valid resting position. The spinner is a heavy rotary dial with detents, not a bouncy toy.
**Per-theme values:** Gold Standard: mass 1.0, friction 0.08. Cloud: mass 1.5, friction 0.06. Fallout: mass 1.0, friction 0.05.
**Design principle:** "Not a toy, but a tool. Something with substance."
**Status:** Active — shipped in 96ef772 and 0543a15

---

### DEC-135: Themes as Game Modes (2026-04-03)

**Date:** 2026-04-03
**Context:** Building per-theme spinner physics revealed that themes can be more than visual. Each theme can have different physics, different spinner variants, and potentially different interaction patterns.
**Decision:** Themes are game modes, not just color schemes. Each theme gets: (1) its own visual rendering (semantic tokens, CRT effects, etc.), (2) its own spinner variant (drum for Fallout, cover flow for Gold Standard/Cloud), (3) its own physics personality (mass, friction values), (4) potentially its own audio character and interaction patterns. Adding a new theme means defining a complete sensory experience, not just swapping colors. The variant architecture (render function strategy pattern with THEME_VARIANT and THEME_PHYSICS maps) supports this cleanly.
**Current themes:** Gold Standard (dark, premium, precise), Cloud (light, warm, unhurried), Fallout (CRT, mechanical, loose).
**Future themes:** Elderly/Garden (simple, large text, high contrast), potential per-subdomain themes (fallout.locallane.app).
**Status:** Active — architecture supports it, new themes are additive

---

### DEC-136: Creator Only as Default Entity Permission (2026-04-04)

**Date:** 2026-04-04
**Context:** Full application audit found FSClient, FSDocument, FSEstimate had public read (Authenticated Users), exposing Bari's client PII and portal tokens to any authenticated user. Team entities similarly exposed. The audit scored the app 68/100 with 6 Critical issues, 3 of which were entity permission holes.
**Decision:** Entity permissions default to Creator Only for read, create, update, delete. Server functions with `asServiceRole` handle all authorized cross-user access (agentScopedQuery, manageTeamPlay, signDocument, etc.). Entity-level permissions are the last line of defense — they must be restrictive, not permissive. When creating new entities, start Creator Only and open up only with explicit justification.
**Rationale:** Entity permissions in Base44 are the layer that can't be bypassed by client-side code. Any authenticated user can call `base44.entities.X.list()` from the browser console. If the entity has Authenticated Users read, all records are exposed. Server functions enforce proper scoping — the entity layer should be the backstop, not the gateway.
**Status:** Active — 9 entities locked down in this session

---

### DEC-137: Feedback Flows Through Companion (2026-04-04)

**Date:** 2026-04-04
**Context:** Two parallel feedback systems existed: FeedbackLog (standalone floating button, dead-end data) and ServiceFeedback (agent-created, visible to Mycelia pulse). Bari's 14+ feedback items were verbal relay — they never reached any entity. The floating feedback button was redundant where agent chat exists.
**Decision:** All user feedback flows through the Mylane companion agent, not standalone buttons. "Have feedback?" quick-action chip on all 8 space positions. MyLane writes to ServiceFeedback entity directly. No confirmation card for feedback — it should feel effortless, not bureaucratic. FeedbackLog entity retired. The agent IS the feedback channel.
**Rationale:** DEC-104 already said "bug button hides in agent-enabled workspaces." This extends it: the button is gone everywhere. The companion knows the context (which space, what the user was doing) and can ask one clarifying question before writing the feedback. A floating button captures isolated complaints. A companion captures contextual feedback.
**Status:** Active — floating button removed, chip added, ServiceFeedback is sole feedback entity

---

### DEC-138: Founding Gardener — Earned Status, Not Signup Bonus (2026-04-04)

**Date:** 2026-04-04
**Context:** platformPulse gardener observation revealed: 22 users, 6 active, 16 dormant. Engagement is highly concentrated — Doron (52), Bari (13), Natasha (13, pure organic signup). The question arose: how do early supporters get recognized? Should "Founding Gardener" be automatic for early signups?
**Decision:** Founding Gardener is earned, not given. Criteria: spaces created, feedback contributed, networks invited into, weeks active. It is personally assigned by Doron after observation — not a first-come signup bonus. Mycelia can surface candidates via the gardener pulse. The organism observes from within (MCP data); Doron observes from the field (relationships, conversations). Both signals matter.
**Rationale:** "Free carries little value." Signing up is not gardening. Bari is a Founding Gardener because he gave 14+ feedback items and tested every feature. The 16 dormant accounts from the early signup push are not gardeners — they planted nothing. The status must mean something real.
**Status:** Active — gardener observation live via platformPulse + MCP

---

### DEC-139: Server-Authoritative Identity on Agent Writes (2026-04-05)

**Date:** 2026-04-05
**Context:** MylaneNote reminder loop field test revealed MyLane agent wrote `user_id: "special-user"` as a literal string — the LLM interpreted the instruction "pass the authenticated user's ID from your context" as a placeholder token. Record persisted but was invisible (query filtered by real user_id, found nothing). The `agentScopedWrite` function had a `writeData[fk] == null` guard that preserved whatever the agent passed — if the agent sent a non-null string, the server-known value was never applied.
**Decision:** Identity fields (`user_id`, `owner_id`) are set exclusively by `agentScopedWrite` from server-resolved auth context. The null-check guard is removed for these fields — the server always wins, unconditionally. The query/write asymmetry is explicit: queries require `user_id` from the agent (to scope reads via agentScopedQuery), writes forbid it (server stamps from `auth.me()` or validated MCP fallback). This is defense in depth against LLM placeholder-token interpretation errors.
**Cross-references:** Extends DEC-115 (agentScopedWrite three-gate enforcement) with a fourth gate: identity stamping. Complements DEC-136 (Creator Only default permissions) — entity permissions are the last defense, server-authoritative identity is the second-to-last.
**Affected entities:** ServiceFeedback, Recommendation, MylaneNote (fkField: `user_id`), plus blanket `workspace === 'platform'` catch-all for future platform entities. No entities currently use `owner_id` as FK field — that branch is defensive.
**Status:** Active — shipped in community-node, pending Base44 publish

---

### DEC-142: Frequency Station Pip-Boy Radio Model + Canonicalized Taxonomies (2026-04-10)

**Date:** 2026-04-10
**Context:** Frequency Station audit found audio architecture fragmented: provider scoped to MyLane (audio dies on navigation), multiple local `<audio>` elements competing, no MediaSession/lock-screen integration. Status and mood taxonomies diverged between spec and code.
**Decision:** (1) Pip-Boy radio: provider at app root, single `<audio playsInline>`, MediaSession API, persistent mini-player, localStorage song persistence. (2) Taxonomies canonicalized to match code: moods `fire/water/earth/air/storm/custom`, statuses `submitted/in_progress/released/archived`. (3) FrequencyArtist confirmed as planned entity for Build 2. Full details in FREQUENCY-STATION-SPEC.md (private).
**Status:** Active — Build 1 shipped

---

### DEC-143: Frequency Station Build 2 — Studio, Library, Ownership Model (2026-04-10)

**Date:** 2026-04-10
**Context:** Build 1 shipped background playback. Build 2 adds studio: ownership, library, rich submissions, admin transform workflow.
**Decision:** (1) Ownership model: `owner_user_id` + `is_public` on FrequencySong. Listen tab filters by is_public. (2) Multi-step submission wizard with dynamic FrequencyMood entity. (3) Admin workbench with Suno copy-paste boxes + delivery-to-submitter. (4) FrequencyArtist entity for identity. (5) FrequencyNotification for in-app delivery alerts. Full details in FREQUENCY-STATION-SPEC.md (private).
**Status:** Active — Build 2 shipped

---

### DEC-144: Frequency Station RLS Loosening + Client-Side Scoping Pattern (2026-04-10)

**Date:** 2026-04-10
**Context:** FSFrequencySubmission.Read and FrequencyNotification.Read were set to `owner` (RLS: created_by == user.email). This blocked admin workbench from seeing other users' submissions and blocked notification recipients from reading admin-created notifications. FSFrequencySubmission.Update also blocked admin status changes.
**Decision:** (1) FSFrequencySubmission.Read, FSFrequencySubmission.Update, and FrequencyNotification.Read loosened from owner to authenticated. (2) Client-side scoping enforces ownership and admin visibility: MySeedsTab filters by user_id, NotificationBell filters by user_id, AdminWorkbench shows all (admin-only tab). (3) Base44 agent defaults to discussion mode (extends DEC-093); action mode only for entity/permission/server-function work.
**Rationale:** RLS on cross-user entities blocks core workflows. Client-side scoping is the pragmatic choice. Tighten later with server functions if needed.
**Status:** Active

---

### DEC-145: Payload-First Debugging + Single-Owner Hooks (2026-04-11)

**Date:** 2026-04-11
**Context:** Three-layer debug chain: stale closures (defensive fix, not the bug) → duplicate hook instances racing (single-owner fix) → FSFrequencyPlaylist track_ids string-not-array (the actual 422). Two hours on architecture theories before 10 seconds of payload inspection found the type mismatch.
**Decision:** (1) Payload-first debugging: when Base44 ops fail, log the payload and compare to schema before theorizing about React. (2) Single-owner hooks: call entity-managing hooks in exactly one component, pass as props. (3) FrequencyLibraryContext planned to replace prop drilling.
**Status:** Active

---

### DEC-149: Mylane Agent v2 — Mandatory Protocol Architecture (2026-04-16)

**Date:** 2026-04-16
**Context:** v1 Mylane instructions allowed the agent to say "Done" without calling tools (hallucination). MCP testing confirmed: agent responded "Saved" for a reminder but never invoked agentScopedWrite. Record did not exist. Vercel AGENTS.md research validated: passive context (mandatory rules in instructions) beats on-demand retrieval (hoping the agent picks the right tool).
**Decision:** Full instruction rewrite with mandatory 4-step protocol: (1) Classify intent, (2) Execute via tool call, (3) Verify result by querying back, (4) Respond to user. "Never lie" rule: saying "Done" without calling the tool is explicitly a lie. Failure protocol: honest failure reporting + ServiceFeedback logging with source "mylane-supervisor" + suggest manual path. Removed all 26 individual entity tools (104 permissions) — 2 backend functions remain (agentScopedQuery + agentScopedWrite). Fewer tools = more reliable.
**Rationale:** Mandatory protocol eliminates the hallucination class of bugs entirely. Verification step catches silent write failures. Failure logging creates an audit trail. Tool reduction reduces the LLM's decision space.
**Status:** Active — v2 is the live instruction set

---

### DEC-150: Smart Routing — TYPE 1 for Views, TYPE 2 for Novel Queries (2026-04-16)

**Date:** 2026-04-16
**Context:** TYPE 2 RENDER_DATA renders raw entity records as cards showing every field (including created_by, updated_date, etc.). Home Canvas spec proposed a new TYPE 4 RENDER_CANVAS to mount real workspace components. Hyphae's review found: no component accepts raw data as props — they all self-fetch from workspace profiles. Building TYPE 4 was unnecessary.
**Decision:** Update Mylane classification to emit TYPE 1 RENDER (workspace drill via MyLaneDrillView) for "show me" queries instead of TYPE 2 RENDER_DATA. TYPE 1 mounts the real workspace component with full drill-through and interactivity — zero code changes needed, instruction change only. TYPE 2 reserved for genuinely novel queries where no workspace view exists (with HIDDEN_FIELDS filter for clean display). Quick answers (counts, dates, amounts) stay as brief text responses.
**Rationale:** The fastest path to real components on the Home canvas is teaching the agent to navigate, not building a new rendering pipeline. One redundant query (agent queries to confirm data exists, component re-queries to display) is irrelevant at our scale.
**Status:** Active

---

### DEC-151: Spec Review Protocol — Review Before Architecting (2026-04-16)

**Date:** 2026-04-16
**Context:** Home Canvas Rendering spec proposed TYPE 4 RENDER_CANVAS with 5 implementation phases. Hyphae's codebase review found the spec's core assumption was wrong: no component accepts raw data as props. The existing TYPE 1 → MyLaneDrillView pipeline already does what TYPE 4 proposed. The review saved weeks of unnecessary work.
**Decision:** Before building any new architecture or protocol, get Hyphae's codebase review first. The review should check: (1) Does the infrastructure already exist? (2) Do the assumptions about component APIs hold? (3) Is there a lighter path using existing patterns? (4) What's the smallest slice that produces visible improvement? Write the spec, but treat it as a hypothesis until Hyphae validates it against the codebase.
**Rationale:** Mycelia and Doron design from vision and user need. Hyphae sees what actually exists in the code. The gap between "what we think the code does" and "what the code actually does" is where wasted work lives. Ten minutes of review saves days of building the wrong thing.
**Status:** Active — process rule

---
