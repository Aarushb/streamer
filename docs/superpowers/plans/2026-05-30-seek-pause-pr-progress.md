# Seek + Pause PR Progress

Date: 2026-05-30
Branch: aarushb/feat/seek-pause

## Scope

- Add server-side seek for the shared radio stream.
- Add server-side pause/resume for the shared radio stream.
- Keep branch rebased on upstream changes.
- Document tooling requirements and fallback behavior.

## Upstream Sync

- Added remote `upstream` => `https://github.com/stevo399/streamer.git`.
- Fetched upstream and rebased feature branch onto `upstream/master`.
- Rebase was clean and preserved local feature commits.

## Implemented: Seek

- Pipeline: added seek action handling via FFmpeg `-ss` and offset-aware elapsed timing.
- API: added `POST /api/tracks/seek`.
- Control panel: added seek form in playback controls.
- Tests: server and pipeline coverage for seek routes/behavior.

## Implemented: Pause/Resume

- State: added global `paused` flag.
- Pipeline: added `request_pause()` and `request_resume()`.
- Pipeline loop: when paused, decoding/output progression is held.
- API: added `POST /api/playback/pause` and `POST /api/playback/resume`.
- Control panel: added Pause Stream / Resume Stream buttons and status indicator.
- State API: includes `paused` for UI polling.
- Tests: state, pipeline, and server coverage for pause/resume.

## ripgrep Requirement + Fallback

- Curator tries `rg` for title-based filesystem search.
- If `rg` is not found, curator falls back to Python recursive scan.
- This keeps title-based requests functional in environments missing ripgrep.

Windows install options for ripgrep:

```powershell
winget install BurntSushi.ripgrep.MSVC
# or
choco install ripgrep
```

## Manual Test Plan

1. Configure environment and media roots.
2. Start server with `uv run streamer`.
3. Open control panel at `http://localhost:8054`.
4. Verify seek:
   - Play a track.
   - Enter a seek time and click Seek.
   - Confirm elapsed time jumps and stream continues.
5. Verify pause/resume:
   - Click Pause Stream.
   - Confirm status shows paused and elapsed stops increasing.
   - Click Resume Stream.
   - Confirm status returns to running and elapsed continues.
6. Verify API endpoints with OpenAPI at `/docs`:
   - `POST /api/tracks/seek`
   - `POST /api/playback/pause`
   - `POST /api/playback/resume`

## Notes

- Pause is global for the radio stream (all listeners).
- This is different from per-client media player pause.
- No data model migration needed (in-memory state only).
