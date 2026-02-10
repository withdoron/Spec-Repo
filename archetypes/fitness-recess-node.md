# Fitness / Recess Node — Archetype Scope

> Light scoping document. Captures initial ideas for future development.
> Created: 2026-02-10
> Status: Idea
> Owner: Doron (user zero for Recess network pass)

---

## Overview

Workout tracking and gamification archetype tied to the Recess network pass. Doron is user zero — he wants this for himself. The key differentiator from existing fitness apps: a ranking system modeled after Rocket League (casual/ranked tiers) that feeds the personal Organism.

## What This Archetype Covers

Personal fitness tracking with community gamification. Not a gym management tool (that's a different archetype) — this is for the individual who works out and wants progression feedback tied to their community.

## Who Uses It

**Individual (Doron is user zero):** Tracks workouts, sees progression, participates in casual or ranked challenges. Their fitness activity feeds their personal Organism's vitality.

**Community:** Recess network pass holders can see aggregate activity (the Recess network organism), participate in community challenges, and discover local fitness businesses through the network.

## The Ranking System — Rocket League Model

**Casual Mode:**
- Log workouts freely
- No pressure, no ranking
- Activity feeds personal Organism
- Good for beginners, recovery periods, people who just want to track

**Ranked Mode:**
- Opt-in competitive tier system
- Tiers based on consistency + intensity (not just volume)
- Weekly or monthly seasons
- Tier movement based on sustained activity, not single bursts
- De-ranking is gentle (mirrors Organism philosophy — rest is healthy, not failure)

**Key design principle:** This is NOT a leaderboard. It's personal progression with community context. You're not competing against others — you're growing your own fitness organism. The ranking reflects YOUR consistency and growth, visible to you. Community aggregate shows "the Recess network is energetic this month" without individual comparison.

This aligns with the Organism Concept: mirror, don't manipulate. Growth, not points. Seasons, not streaks. Connection, not competition.

## Potential Features by Phase

**Phase 1: Personal Tracking**
- Log workouts (type, duration, intensity, notes)
- Voice input for quick logging (same pattern as Contractor Daily)
- Weekly/monthly activity summary
- Simple progression view (am I doing more than last month?)

**Phase 2: Organism Integration**
- Workout data feeds personal Organism vitality
- Recess network organism reflects aggregate community activity
- Seasonal patterns visible (summer outdoor peak, winter gym steady)
- Rest periods reflected as healthy cycles, not decline

**Phase 3: Ranking System**
- Casual/ranked mode toggle
- Tier definitions based on consistency metrics
- Season system (monthly? quarterly?)
- Tier badges on personal profile (visible in LocalLane)
- De-ranking grace periods (vacation, illness, life happens)

**Phase 4: Community & Business Integration**
- Discover Recess network businesses (gyms, yoga studios, climbing walls, martial arts)
- Community challenges (opt-in, seasonal)
- Check-ins at Recess businesses feed both personal Organism and business vitality
- Community Pass / Joy Coins integration for Recess activities

## Organism Signals

| Activity | Signal | Level |
|----------|--------|-------|
| Workout logged | Participation | Personal |
| Consistent weekly activity | Vitality growth | Personal |
| Ranked tier maintained | Sustained engagement | Personal |
| Check-in at Recess business | Cross-pollination | Personal + Business |
| Community challenge participation | Community engagement | Network |
| Seasonal activity patterns | Natural rhythm | Network |

## The Recess Network Pass Connection

This app IS the Recess pass experience for individuals. The Recess network pass (one of four under Community Pass) covers movement and play: yoga, gyms, climbing, dance, martial arts, outdoor adventures, sports leagues. This fitness node is how an individual interacts with that network on a daily basis.

From ORGANISM-CONCEPT.md:
- Creature character: Energetic, bouncy, dynamic. More movement = more alive.
- Seasonal quality: Peaks in summer (outdoor activities), stays active year-round (indoor fitness)
- Color palette: Greens and blues — vitality, nature, motion

## Real Test Case

Doron as user zero. Real daily use, real feedback loop. Same pattern as Property Pulse (Doron's rental properties) and Contractor Daily (Dan's construction business).

## Key Design Questions

| Question | Status |
|----------|--------|
| What constitutes a "workout"? (Broad: walking counts? Narrow: structured exercise only?) | Open — probably broad, mirrors Organism philosophy |
| How are tiers calculated? (Frequency? Duration? Intensity? Combination?) | Open — needs design |
| Season length? (Weekly too short, yearly too long) | Open — monthly feels right |
| Integration with existing fitness APIs? (Apple Health, Google Fit, Strava) | Future — Phase 1 is manual logging |
| Standalone Base44 app or built into Community Node? | Open — standalone first, same as PP and CD |
| How does ranking avoid becoming a leaderboard? | Core design challenge — personal tiers, not comparative ranking |

## Dependencies

- Base44 platform (standalone app)
- Gold Standard design system
- Organism Concept (vitality calculation)
- Recess network pass definition (from COMMUNITY-PASS.md)

---

*This is a holding doc. Full spec will be written using archetypes/_template.md when we're ready to build.*
