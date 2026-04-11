# ACTIVE-CONTEXT.md

> What's happening RIGHT NOW. This file gets overwritten each session, not appended.
> Last updated: 2026-04-10 (Frequency Station Build 1 + Build 2 ship-it)

## Current Focus

Frequency Station end-to-end loop is functional: seed → wizard → admin workbench → Suno transform → deliver to submitter's library → lock-screen playback. First real song: "Grow, Little Seedlings" by The OG. Debug chain resolved RLS permissions, field name mismatches, and wizard state bugs. Polish items queued.

## Active Architecture

- **Frequency Station:** Phase 3 in progress. Pip-Boy radio model (DEC-142). Studio + Library + Ownership (DEC-143). RLS loosened (DEC-144). Provider at App.jsx root. Single `<audio playsInline>`. MediaSession wired. Persistent mini-player.
- **Agent writes:** DEC-139 — identity fields (user_id, owner_id) are server-authoritative on all agentScopedWrite calls.
- **Theme system:** Semantic tokens (98.5% migrated). Three themes: Gold Standard, Cloud, Fallout.
- **Spinner:** 3D variant architecture. Friction + mass physics. Ratchet snap.
- **Team space (7 tabs):** Production-ready. Coach Rick confirmed. readTeamData server function (DEC-140).
- **Query optimization:** staleTime 5min global default (DEC-130). getMyLaneProfiles batches 6 queries into 1.
- **Agent architecture:** DEC-107 enforced — space agents use agentScopedQuery only. DEC-136 — entity permissions default Creator Only.
- **Auth:** Single source of truth — AuthContext seeds React Query cache.
- **Error isolation:** WorkspaceErrorBoundary wraps each workspace drill view.
- **Health score:** 87/100 (from 68 on 2026-04-04 audit)

## What Just Shipped (2026-04-10)

1. **Build 1 — Pip-Boy Radio:** FrequencyProvider at root, single audio element, MediaSession, mini-player, localStorage persistence. Background playback verified on iPhone.
2. **Build 2 — Studio & Library:** SubmitWizard (3-step), AdminWorkbench (Suno boxes + delivery), MyLibrary (public/private toggle), FrequencyArtist CRUD, NotificationBell, ownership model (owner_user_id + is_public).
3. **Debug chain:** ListenTab infinite loop fix, wizard Enter-key fix, RLS permission loosening (DEC-144), field name corrections (FrequencyArtist owner_user_id, FrequencyNotification body, DeliveryForm owner chain).
4. **Team space visibility:** Coach Rick phone-confirmed. readTeamData server function + 8 entity permissions relaxed (DEC-140).

## Known Issues (Frequency Station — Polish Queue)

1. **Notification bell polish:** Badge may not render or display correctly — needs field test
2. **Duplicate song on delivery:** Two FrequencySong records created per delivery — investigate double-submit
3. **FrequencyMood sort_order not respected:** Moods display in creation order, not intended order
4. **FrequencyMood color_hex not used:** Mood pills use generic primary styling instead of mood colors
5. **~300 lines dead code in FrequencyStation.jsx:** Old SubmitTab, SongCreationForm, QueueTab — replaced by extracted components but not deleted
6. **DeliveryForm doesn't write mood_tag or artist_id** on delivered FrequencySong records
7. **Old SongCreationForm creates songs without owner_user_id/is_public** — dead code but dangerous if triggered
8. **EditSeedForm still used by MySeedsTab** — needs update to match new submission fields

## Upcoming Priorities

1. Frequency Station polish — work through known issues list
2. Newsletter "The Good News" — wake dormant accounts
3. Ephraim Pip-Boy design session
4. Bari visit for feedback chip demo
5. Base44 discussion-mode pattern for future entity audits
