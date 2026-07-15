# Beyond All Markets — live site

Single-page landing site for the **Beyond All Markets** live stream: a 24/7 autonomous
Beyond All Reason RTS broadcast where real crypto and index order flow commands
the armies.

- `index.html` — self-contained (inline CSS, Google Fonts, baked poster). No build step.
- The stream embeds the current live broadcast on YouTube channel `UCTZ2PNoiJvryW6s47uKFTog`
  via `embed/live_stream?channel=…`, so it auto-follows whatever is live.

Deploys as a static site (Vercel / any static host) — no config needed.
