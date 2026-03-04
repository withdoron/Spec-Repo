# Active Context

> What's happening right now. Updated each session.
> Last updated: 2026-03-03

## Current Focus

Pilot readiness — making the platform feel alive for first-time visitors and enabling organic sharing.

## Last Session (2026-03-03)

Shipped 7 items: DEC-060 typographic cards (no images, equal weight), living tiles (5 layers of ambient life — accent bars, hover warmth, lift, depth, typography), recurring events fix (all 4 patterns), shareable event links (/events/:eventId with clipboard copy), category accent bar fix, DEC-060 pointer in spec-repo DECISIONS.md.

Living homepage prompt written and ready to run: Happening Soon events section, random business rotation, section reorder.

## What's Queued

1. Run living homepage prompt (Happening Soon, random businesses, section reorder, value prop polish)
1. Seed directory with Farmers Market vendors
1. Test shareable event links end-to-end
1. Recurring events edge case polish

## Active Decisions

- DEC-059: Claude Code is primary dev tool (Cursor cancelled)
- DEC-060: Typographic cards with ambient vitality — no images on cards, equal weight, vitality CSS hooks planted

## Key Context for Next Session

- Vitality CSS hooks are in index.css (data-vitality warm/cool/neutral) — cards respond to backend state without component changes
- Shareable event links work: /events/:eventId opens modal, share button copies to clipboard
- Homepage business selection is random rotation (shuffled per page load, capped at 4)
- Homepage events are soonest-first, public only, sections hide when empty
- All card surfaces (business, event, value prop) should share the same living tile aesthetic
