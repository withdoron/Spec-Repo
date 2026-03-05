# Active Context

> What's happening right now. Updated each session.
> Last updated: 2026-03-04

## Current Focus

Play Trainer — Play Builder shipped, polish pass next.

## Last Session (2026-03-04 evening)

20-item Play Builder build. Visual play creation with SVG football field, route templates (slant, out, post, fly, curl, drag, flat, corner, hitch, block), freehand drawing, custom positions. 8 new files, 8 modified. Base44 entity fields added (Play: use_renderer, custom_positions; PlayAssignment: movement_type, route_path, start_x, start_y). 7 bug fixes including .filter().list() chains, toggle knob colors, JSON stringify fix. DEC-061: Play Builder pulled forward from Build 7 to Build 4.

## What's Queued

1. Play Builder polish (field aspect ratio in Study Mode, mobile padding, phone testing)
2. Quiz Engine (Team Build 5)
3. Newsletter Issue 1 draft
4. Seed directory with Horai and NW OG Farm (Claim Your Business)
5. Android Chrome mobile testing

## Active Decisions

- DEC-059: Claude Code is primary dev tool (Cursor cancelled)
- DEC-060: Typographic cards with ambient vitality
- DEC-061: Play Builder — visual play creation pulled forward from Build 7 to Build 4

## Key Context for Next Session

- Play Builder works end-to-end: create, save, view in Study Mode
- Known polish: field aspect ratio compressed in Study Mode, mobile padding cutoff in overlay, needs actual phone testing
- Base44 JSON fields take raw arrays — do NOT stringify (route_path, custom_positions)
- Claude Code worktree branch issue: .claude/worktrees/ accumulation causes branch creation, needs cleanup + CLAUDE.md hard rule
- community-node repo path: ~/Documents/LocalLane/community-node
- Boys are actively creating plays — real user testing happening
