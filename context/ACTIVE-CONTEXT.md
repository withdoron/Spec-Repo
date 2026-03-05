# Active Context

> What's happening right now. Updated each session.
> Last updated: 2026-03-04

## Current Focus

Pilot readiness — mobile polish and stranger test in progress.

## Last Session (2026-03-04 afternoon)

Shipped 20 items: onboarding gate fix (localStorage → server-side), onboarding redirect loop fix (optimistic cache), network active flag enforcement with admin toggle, network living tiles (DEC-060 on all network surfaces), contextual back navigation, clickable network chips on business cards/profiles, directory cleanup (Browse by Category removed, mobile polish), banner image upload in Settings, BusinessProfile hero refactor (fallback logic, overlap removed, full visibility), share button wired, heart icon removed, settings save → home tab navigation, upcoming events section on business profiles, Recess banner created and uploaded.

## What's Queued

1. Newsletter Issue 1 draft
1. Seed directory with Horai and NW OG Farm (Claim Your Business)
1. Test shareable links end-to-end
1. Android Chrome mobile testing
1. Broken links walkthrough

## Active Decisions

- DEC-059: Claude Code is primary dev tool (Cursor cancelled)
- DEC-060: Typographic cards with ambient vitality

## Key Context for Next Session

- Business profile images: logo_url (square, for cards) vs photos[0] (banner, for hero). Both uploadable from Settings.
- Banner overlap removed — clean mt-4 gap, no gradient overlay
- Network active toggle in AdminSettings ConfigSection — amber switch, strikethrough + Hidden badge for inactive
- Onboarding gate is now purely server-side (onboarding_complete field on user record). No localStorage involvement.
- Claude Code creates branches despite CLAUDE.md instruction. Merge via: cd community-node && git merge claude/BRANCH-NAME && git push origin main
- Horai bakery = top target for first real business seeding. NW OG Farm second. Both via Claim Your Business flow.
- Newsletter subscribers at 6 organic, 2 organic accounts, zero marketing.
