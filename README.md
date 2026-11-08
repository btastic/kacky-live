# Kacky Live

[![License: GPL-3.0](https://img.shields.io/badge/license-GPL--3.0-FF7800)](LICENSE)

Clean, read-only stream overlays for [Kacky](https://kacky.gg) events, built for viewers. Kacky Live answers what chat asks most: *when is the streamer's map up?* and *how far are they?* It shows the streamer's favourite maps coming up on the servers, plus their event progress, right inside OBS.

It complements Kontrol, the in-game server UI, rather than repeating it: the current map, top records, linked servers and donors are already on screen in game.

Runs entirely in the browser on top of the KackyGG API ([`docs/API-NOTES.md`](docs/API-NOTES.md)). Stats come from public endpoints; the favourites schedule uses a read-only key the streamer creates on their kacky.gg profile.

**Current Status:** Planning ([Phase 0](https://github.com/kacky-code/kacky-live/milestone/1)). The spec ([`docs/specs/overlay-mvp.md`](docs/specs/overlay-mvp.md)) is up for review before scaffolding the app. Check the [open issues](https://github.com/kacky-code/kacky-live/issues) to weigh in.

## Roadmap

| Phase | Goal | Status |
|---|---|---|
| [Phase 0: spec and mockup](https://github.com/kacky-code/kacky-live/milestone/1) | Agree on widgets, the overlay key, polling and fallback states | In progress |
| [Phase 1: overlay MVP](https://github.com/kacky-code/kacky-live/milestone/2) | Ship the 1920x1080 OBS browser source with "Up next" and the stats strip | Planned |
| [Phase 2: Twitch panel extension](https://github.com/kacky-code/kacky-live/milestone/3) | Viewer-facing Twitch panel reusing the Phase 1 client | Planned |

## What We're Building

### Phase 1: OBS Overlay

A `1920x1080` transparent browser source for OBS, configured via URL parameters:

```text
https://<host>/overlay?widget=schedule&player=<player-id>#key=<overlay-key>
https://<host>/overlay?widget=progress&player=<player-id>&stats=progress,milestone,rank
```

- `widget` (optional): `schedule` or `progress`. Each widget is its own browser source, so streamers place them independently around their camera, chat and Kontrol HUD. Without it, both widgets show on one 1920x1080 layer.
- `player` (required): The streamer's KackyGG player id, picked by name on the `/config` page.
- `count` (optional): How many maps to show, 1–10. Default: 4.
- `stats` (optional): Which stats the strip shows, in this order: `progress`, `milestone`, `rank`, `avg`, `wr`. Default: `progress,milestone,rank`.
- `key` (after `#`, schedule only): The streamer's read-only overlay key. The progress widget needs none. Being after `#`, it is never sent to the overlay host.

```text
+------------------------------+
| UP NEXT                      |
| #464  S8   live · 4:12 left  |
| #503  S7   in 9:40           |
| #486  S1   in 31:05          |
+------------------------------+

KR7  51 / 75 ━━━━━━  Silver · 9 to Gold  Rank #3
```

#### Widgets

- **Up next:** The streamer's favourite maps on the live edition servers, soonest first, with server and countdown. Streamers mark the maps of their current rotation as favourites in Kontrol, so the list stays short. Every favourite counts, whatever marker it has, and there is nothing to configure.
- **Stats strip:** The streamer's progress in the current event, with the stats they pick: finished maps, the next milestone (Plastic, Bronze, Silver, Gold, Kacky), leaderboard rank, average rank and world records.

During hunting, servers are jukeboxed only a few maps ahead, so there is no schedule: the overlay shows the stats strip for the hunting event only.

#### Stream-Safe by Design

- **Quiet visuals:** High contrast for readability over Trackmania stadium concrete and dirt, with zero distracting animations during runs.
- **Graceful fallbacks:** If an API request times out or a server restarts, the overlay holds the last known good state and keeps counting down. It never flashes error banners or stack traces on stream.
- **Safe to leak:** The overlay key is read-only. If it ends up in a screenshot of OBS settings, nobody can change anything with it.

### Phase 2: Twitch Panel Extension

Once the OBS overlay is proven on live streams, Phase 2 reuses the same client layer to power an interactive viewer-facing Twitch panel extension.

## Documentation

- [`CONTRIBUTING.md`](CONTRIBUTING.md) – Workflow and PR guidelines.
- [`docs/API-NOTES.md`](docs/API-NOTES.md) – Public KackyGG `/v1` endpoints, fields, and polling intervals.
- [`docs/SPEC-TEMPLATE.md`](docs/SPEC-TEMPLATE.md) – Structure for layout and widget specs.
- [`docs/specs/overlay-mvp.md`](docs/specs/overlay-mvp.md) – Phase 1 streamer overlay spec (proposal for #1).
- [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) – Community guidelines.

## Contributing

We want Kacky Live to be easy and fun to build together. Jump into the spec discussion in Issue [#1](https://github.com/kacky-code/kacky-live/issues/1), or pick up documentation in Issue [#10: OBS setup guide & URL generator page](https://github.com/kacky-code/kacky-live/issues/10).

- Browse [`good first issue`](https://github.com/kacky-code/kacky-live/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) and [`help wanted`](https://github.com/kacky-code/kacky-live/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22).
- Check [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request.

## Maintainers

Maintained by [@Thorbenl](https://github.com/Thorbenl) and [@bompi-me](https://github.com/bompi-me).

## Security & Licence

Please report security issues privately via [SECURITY.md](SECURITY.md). Licensed under the [GNU General Public License v3.0](LICENSE).
