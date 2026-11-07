# Kacky Live

[![License: GPL-3.0](https://img.shields.io/badge/license-GPL--3.0-FF7800)](LICENSE)

Clean, read-only broadcast overlays for [Kacky](https://kacky.gg) events. Kacky Live gives streamers an always-current board for the active map, top records, server queue, and top donors—right inside OBS without needing an extra browser window open.

Runs entirely in the browser on top of the public KackyGG API ([`docs/API-NOTES.md`](docs/API-NOTES.md)). Because it is client-only and read-only, anyone can run it locally, test against live data, and contribute widgets without needing backend credentials or server access.

**Current Status:** Planning ([Phase 0](https://github.com/kacky-code/kacky-live/milestone/1)). We are finalizing the layout spec and wireframe before scaffolding the app. Check the [open issues](https://github.com/kacky-code/kacky-live/issues) to weigh in.

## Roadmap

| Phase | Goal | Status |
|---|---|---|
| [Phase 0: spec and mockup](https://github.com/kacky-code/kacky-live/milestone/1) | Finalize HUD layout, polling rates, and fallback states | In progress |
| [Phase 1: overlay MVP](https://github.com/kacky-code/kacky-live/milestone/2) | Ship the 1920x1080 OBS browser source with core widgets | Planned |
| [Phase 2: Twitch panel extension](https://github.com/kacky-code/kacky-live/milestone/3) | Viewer-facing Twitch panel reusing the Phase 1 client | Planned |

## What We're Building

### Phase 1: OBS Overlay

A `1920x1080` transparent browser source for OBS, configured via URL parameters:

```text
https://<host>/overlay?server=<server-login>&player=<player-login>
```

- `server` (required): The Kacky server login to track.
- `player` (optional): Highlights the streamer's personal best in the top 5 records.

```text
+-------------------------------------------------------------------------+
| [KACKY #7 - DAY 3 OF 15]                                   1920 x 1080  |
|                                                                         |
| +---------------------+   +---------------------+   +-----------------+ |
| | CURRENT MAP #42     |   | TOP 5 RECORDS       |   | UP NEXT         | |
| | Author: 0:14.210    |   | 1. PlayerOne  13.4s |   | #43 (5 min)     | |
| | WR:     0:13.412    |   | 2. PlayerTwo  13.8s |   | #44             | |
| +---------------------+   | 3. You        14.1s |   +-----------------+ |
|                           +---------------------+                       |
|                                                     +-----------------+ |
|                                                     | TOP SUPPORTERS  | |
| (Transparent game area for Trackmania runs)         | SupporterA €50  | |
|                                                     +-----------------+ |
+-------------------------------------------------------------------------+
```

#### Core Widgets

- **Event Header:** Active event title and progress (`Day N of M`).
- **Map Card:** Current map number, author time, and world record.
- **Top 5 Records:** Live leaderboard for the active map.
- **Server Queue:** Next maps coming up on the server.
- **Top Donors:** Community supporters over the last 90 days.

#### Stream-Safe by Design

- **Quiet visuals:** High contrast for readability over Trackmania stadium concrete and dirt, with zero distracting animations during runs.
- **Graceful fallbacks:** If an API request times out or a server restarts, the overlay holds the last known good state. It never flashes error banners or stack traces on stream.

### Phase 2: Twitch Panel Extension

Once the OBS overlay is proven on live streams, Phase 2 reuses the same client layer to power an interactive viewer-facing Twitch panel extension.

## Documentation

- [`CONTRIBUTING.md`](CONTRIBUTING.md) – Workflow and PR guidelines.
- [`docs/API-NOTES.md`](docs/API-NOTES.md) – Public KackyGG `/v1` endpoints, fields, and polling intervals.
- [`docs/SPEC-TEMPLATE.md`](docs/SPEC-TEMPLATE.md) – Structure for layout and widget specs.
- [`docs/specs/overlay-mvp.md`](docs/specs/overlay-mvp.md) – Phase 1 overlay spec (proposal for #1).
- [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) – Community guidelines.

## Contributing

We want Kacky Live to be easy and fun to build together. Jump into the layout discussion in Issue [#1](https://github.com/kacky-code/kacky-live/issues/1), or pick up documentation in Issue [#10: OBS setup guide & URL generator page](https://github.com/kacky-code/kacky-live/issues/10).

- Browse [`good first issue`](https://github.com/kacky-code/kacky-live/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) and [`help wanted`](https://github.com/kacky-code/kacky-live/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22).
- Check [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request.

## Maintainers

Maintained by [@Thorbenl](https://github.com/Thorbenl) and [@bompi-me](https://github.com/bompi-me).

## Security & Licence

Please report security issues privately via [SECURITY.md](SECURITY.md). Licensed under the [GNU General Public License v3.0](LICENSE).
