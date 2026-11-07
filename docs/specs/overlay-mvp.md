# Overlay MVP Spec

Status: **Proposal** for [#1](https://github.com/kacky-code/kacky-live/issues/1). Covers the Phase 1 widgets
[#4](https://github.com/kacky-code/kacky-live/issues/4)–[#8](https://github.com/kacky-code/kacky-live/issues/8).

Field names and behaviour below were checked against the production API on 2026-11-07, during
Kacky Reloaded 7 (event `26`, 10 public servers).

## 1. Context & Surface

- **Target surface:** OBS browser source, `1920x1080`, transparent background.
- **Use case:** A streamer hunting maps on one Kacky server wants the current map, its records, the
  upcoming queue and the top supporters on screen without opening another window.

## 2. URL Parameters

```text
/overlay?server=<server-login>&player=<player-name>
```

| Param | Required | Default | Validation |
|---|---|---|---|
| `server` | yes | – | Non-empty. Missing → render nothing (transparent). `/config` checks it against `GET /servers/live`. |
| `player` | no | – | Matched case-insensitively against a record's `nickname`. No match → no highlight. |

`player` is a nickname and not a login because record rows carry no login (see Open Questions).

## 3. Data & Polling

| Widget | Endpoint | Fields used | Poll |
|---|---|---|---|
| Event header | `GET /events/active` | `name`, `edition`, `startTime`, `endTime`, `id` | 60 s |
| Map card + queue | `GET /servers/{login}/maps` | `currentMap`, `nextMaps[]` | 20 s |
| Map timing | `GET /servers/live` (row with matching `login`) | `mapEndsAt` | 20 s |
| Map detail | `GET /maps/{mapNumber}` | `totalFinishers` | On map change |
| Top 5 + WR | `GET /maps/{mapNumber}/records?eventId={id}&limit=5` | `rank`, `nickname`, `score` | 20 s |
| Top donors | `GET /supporters/top-donors?days=90&limit=5` | `rank`, `name`, `amount` | 60 s |

Notes from the live API:

- **Records need `eventId`.** Without it, `/maps/{n}/records` returns `{ data: [], meta: { total: 0 } }`
  for every map, even maps with 1000+ finishers. The event id comes from `/events/active`.
- **WR is rank 1** of the same records call; no extra request.
- **`currentMap` from `/servers/{login}/maps`** already holds `authorNickname` and `authorTime`, so map
  detail is only needed for `totalFinishers` (which is `null` in the server payload).
- **Response shapes differ:** `/servers/live` and `/supporters/top-donors` return plain arrays;
  only records and leaderboard use `{ data, meta }`.

## 4. Layout & Visuals

Base wireframe as in the README. Widgets sit along the top edge and the right edge; the centre and
bottom stay clear for the car, the speedometer and the in-game timers.

```text
+-------------------------------------------------------------------------+
| KACKY RELOADED 7 · DAY 8 OF 30                                          |
| +---------------------+   +---------------------+   +-----------------+ |
| | MAP #478            |   | TOP 5               |   | UP NEXT         | |
| | by striderin        |   | 1. iFusical  21.136 |   | #486  in 4:12   | |
| | AT  0:20.764        |   | 2. ...              |   | #492            | |
| | WR  0:21.136        |   | 3. You       22.410 |   | #495            | |
| | 951 finishers       |   +---------------------+   +-----------------+ |
| +---------------------+                             +-----------------+ |
|                                                     | TOP SUPPORTERS  | |
|            (transparent game area)                  | catfright  €500 | |
|                                                     +-----------------+ |
+-------------------------------------------------------------------------+
```

- **Panels:** Dark translucent background (≈ `rgba(10,10,14,0.72)`), light text. Readable on bright
  stadium concrete and on dirt without relying on the game behind it.
- **Type:** One sans-serif family, tabular numerals for all times so digits don't jump.
- **Highlight:** The `player` row gets the accent colour (`#FF7800`) and nothing else.
- **Motion:** No animation during runs. Values swap in place; at most a short fade on map change.

### Formatting rules

- **Names:** Strip Trackmania codes from every map name, event name and nickname before rendering:
  colours `$rgb` (1–3 hex digits), styles `$o $i $w $n $t $s $g $z $m`, links `$l[...]`/`$h[...]`,
  and render `$$` as a literal `$`. Example: `$o$i$aa0Kack$05ay Re$09alo$6a0ad$aa0ed $4f07` → `Kacky Reloaded 7`.
- **Nicknames:** Use the plain `nickname`. `customNickname` is ignored in the MVP.
- **Map number:** Display `#478` from `mapNumber`, not the parsed name.
- **Times:** `authorTime` and `score` are milliseconds → `m:ss.mmm`, minutes dropped under 1 min in the
  top-5 list (`21.136`).
- **Day counter:** Day 1 starts at `startTime`; days roll over every 24 h from `startTime` (not at
  local midnight). `M` = whole days between `startTime` and `endTime`.
- **Countdown:** Only the first queue entry gets a countdown, from `mapEndsAt`. Later maps have no
  reliable start time, so they show the number only.
- **Donors:** `name: null` → `Anonymous`. Amounts in EUR, no decimals when whole (`€500`).

## 5. UI States & Fallbacks

| Widget | Loading | Error / offline | Empty | Overflow |
|---|---|---|---|---|
| Event header | Hidden | Hold last good | `404` (no event) → hidden | Ellipsis at panel width |
| Map card | Hidden | Hold last good | `currentMap: null` → hidden | Ellipsis on author |
| Top 5 | Hidden | Hold last good | No records → "No finishes yet" | Ellipsis on nickname |
| Up next | Hidden | Hold last good | Empty queue → hidden | Max 3 entries |
| Top donors | Hidden | Hold last good | Hidden | Ellipsis on name |

- **Never** render error text, status codes or stack traces.
- Unknown server (`404` on `/servers/{login}/maps`) → whole overlay stays transparent; `/config`
  catches this before the link is shared.
- After 5 minutes without one successful response, a widget hides instead of showing stale data.

## 6. Acceptance & Testing

- Renders at 1920x1080 in an OBS browser source with no scrollbars and a transparent background.
- Holds the last good state through a simulated 2-minute network outage, then recovers.
- Name stripping has unit tests for the cases above, including `$$` and nested colour codes.
- Day counter has unit tests around the `startTime` rollover.
- Polling pauses while the document is hidden and backs off on repeated errors.
- No measurable FPS impact in OBS during a 30-minute session.

## Open Questions

1. **Player highlight key:** Records only expose `playerId`, `nickname` and `customNickname`. Should
   `player` stay a nickname, or can the API add the Trackmania account login to record rows?
2. **CORS:** The API currently sends no `Access-Control-Allow-Origin` header, also not for
   `http://localhost:5173`. Local dev will use a Vite proxy; production needs [#9](https://github.com/kacky-code/kacky-live/issues/9).
3. **Widget placement:** Streamers, please confirm the top row doesn't cover anything you need
   (checkpoint splits, chat, webcam).
