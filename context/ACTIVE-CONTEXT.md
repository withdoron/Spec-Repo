# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-11 (Frequency Station B1 + B1.5 + B2 polish + payload debug chain)

## Current Focus

Frequency Station is a three-section personal library with favorites, queue, and a Spotify-shaped listening surface. 17 songs in Doron's library via bulk upload. End-to-end flows working: submit → transform → deliver → favorite → queue → play. Payload-first debugging principle established (DEC-145).

## Active Architecture

- **Frequency Station:** Phase 3 complete. Three-section My Library (My Songs / Favorites / Queue). Single-owner hook pattern: favorites + queue hooks in FrequencyStation.jsx, passed as props to children. SongRow shared component everywhere. Pip-Boy radio (DEC-142), Studio + Library (DEC-143), RLS (DEC-144), payload-first (DEC-145).
- **Frequency hooks:** `useFrequencyFavorites` (FSFrequencyFavorite CRUD), `useFrequencyQueue` (FSFrequencyPlaylist title='queue'). Both use refs to avoid stale closures. All payloads explicitly String()-typed. track_ids sent as real arrays, not JSON strings.
- **Agent writes:** DEC-139 — identity fields server-authoritative on all agentScopedWrite calls.
- **Theme system:** Semantic tokens (98.5% migrated). Three themes: Gold Standard, Cloud, Fallout.
- **Team space:** Production-ready. Coach Rick confirmed. readTeamData server function (DEC-140).
- **Query optimization:** staleTime 5min global default (DEC-130).
- **Health score:** 87/100

## What Just Shipped (2026-04-11)

1. **Prompt A — Polish:** Lock-screen metadata fix (stale closure in onPlay), delivery double-click guard, dead code removal (515 lines), mood_tag + artist_id on delivered songs
2. **B1 — Wizard + Tabs:** Single-page wizard with drafts, tab rename/reorder (My Library > Explore > Submit > My Submissions), per-user station default
3. **B1.5 — Bulk Upload:** BulkUploadModal with filename parsing, 17 songs imported
4. **B2 — Library Restructure:** Three sections, SongRow shared component, favorites via FSFrequencyFavorite, queue via FSFrequencyPlaylist, heart + add-to-queue buttons everywhere
5. **Debug chain:** SongRow useNavigate fix → single-owner hook pattern → queue track_ids string→array (422 fix) → favorites String() defensive defaults

## Known Issues (Remaining Polish)

1. **FrequencyLibraryContext refactor** — favorites/queue hooks should move from prop-drilling to a context provider (planned for next touch of this area)
2. **EditSeedForm uses old field set** — still writes legacy fields, not Build 2 fields
3. **AdminUploadForm dead code** in MyLibrary.jsx — function defined but no longer rendered
4. **Old SongCard component** in FrequencyStation.jsx — replaced by SongRow but function still defined
5. **FrequencyMood sort_order not respected** in wizard (mood removed from wizard, but entity still exists)
6. **Bulk upload doesn't extract cover art** — jsmediatags removed due to Base44 deployment incompatibility

## Upcoming Priorities

1. FrequencyLibraryContext refactor (eliminate prop drilling for favorites/queue)
2. Newsletter "The Good News" — wake dormant accounts
3. Ephraim Pip-Boy design session
4. Responsive polish pass for Frequency Station mobile
5. Base44 discussion-mode pattern documentation
