# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-02 (mega session end)

## Current Focus

CommandBar copilot shipped. Three themes live. Responsive architecture in place. Friday 5 PM demo with Coach Rick on iPhone.

## Active Architecture

- **Navigation:** SpaceSpinner (horizontal gallery) + native scroll priorities
- **Copilot:** CommandBar (atomic queries, no history). Mylane is the only user-facing agent.
- **Themes:** Gold Standard (dark), Cloud (light), Fallout. CSS override propagation covers all 35 workspace files.
- **Responsive:** useBreakpoint hook + container queries. Desktop: right-panel CommandBar (300px). Mobile: bottom-bar.
- **Audio:** FrequencyContext with persistent audio element and master power switch.

## Friday Demo Path

1. locallane.app on iPhone, sign in as Coach Rick
2. Team space: roster count, next practice, Playbook Pro below stats
3. CommandBar: chip queries should trigger RENDER_DATA cards
4. Dark mode (Gold Standard) for demo

## Upcoming Priorities

1. Verify CommandBar → Mylane agent → RENDER_DATA end-to-end
2. Base44 publish (getMyLaneProfiles server function)
3. Creature/mascot design (simple form, personality through motion)
4. Finance/EuDash for SNAP submission
5. Semantic Tailwind migration (organic per DEC-132)
6. Ephemeral conversation cleanup for CommandBar
