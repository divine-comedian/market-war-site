# Daily 3-hour stream + library-fed site loop

Run the GPU box ~3 hours a day instead of 24/7 (~80% GPU cost cut). The box streams a live
match during its window (10:00–13:00 Mexico City); YouTube auto-archives every stream into the
@divin3comedian library; the rest of the day the site loops the latest archived match. Nothing
to record or upload — YouTube is the archive.

## How the site chooses what to show

`index.html` reads `state.json` at load and picks its director-cam embed:

```json
{ "mode": "live" | "replay", "channelId": "UC…", "videoId": "…" }
```

- `live`   → `embed/live_stream?channel=<channelId>` (whatever is currently live)
- `replay` → `embed/<videoId>?loop=1&playlist=<videoId>` (that archived match, looped)

If `state.json` can't be read, the hardcoded live embed stays (safe default). Fully static —
Vercel just serves the files.

## The daily automation

Two scheduled GitHub Actions in this repo (free — public repo), each also runnable by hand
("Run workflow" / `workflow_dispatch`):

- **`.github/workflows/stream-start.yml`** — 16:00 UTC (10:00 Mexico City): `vastai start
  instance` (box boots → pm2 auto-runs war+stream → live) → write `state.json` `mode:live` →
  commit/push → Vercel redeploys → site shows live.
- **`.github/workflows/stream-stop.yml`** — 19:00 UTC (13:00 Mexico City): look up the
  channel's most recent completed stream via the YouTube Data API → write `state.json`
  `mode:replay` + that `videoId` → commit/push → `vastai stop instance`. Site flips to the
  archive *before* the box stops.

`scripts/update-state.py` does the YouTube lookup + writes `state.json` (stdlib only; never
publishes a blank replay — falls back to `mode:live` if there's no match to show).

## Required repo config (Settings → Secrets and variables → Actions)

Secrets: `VAST_API_KEY`, `YT_API_KEY` (YouTube Data API v3, read-only).
Variables: `VAST_INSTANCE_ID` = `45096377`, `YT_CHANNEL_ID` = `UCTZ2PNoiJvryW6s47uKFTog`.

**These must be on THIS repo (`market-war-site`)** — that's where the workflows run. (If they
were set on the game repo `beyond-all-markets`, move them here.)

## Activation

1. Merge this branch to `main` — scheduled workflows only fire from the default branch, so
   nothing runs until then (safe to review first).
2. Confirm the secrets/vars above are on this repo.
3. Vercel already deploys this repo's `main`, so the state.json commits redeploy automatically.

Times are UTC in cron; Mexico City is UTC-6 year-round (no DST since 2023). To change the
window, edit the two `cron:` lines.

## Watch-outs

- **Archive lag:** a 3-hour stream takes a while for YouTube to finalize into a seekable VOD.
  `stream-stop` uses `eventType=completed`, so if today's isn't finalized it shows the previous
  fully-processed match — always playable, at most a day behind on the worst day.
- **Box persistence:** the workflow uses `stop` (not `destroy`), which preserves the container
  filesystem so `start` brings the whole install back and pm2 resurrects war+stream on boot.
  Never wire `destroy` into the schedule.
