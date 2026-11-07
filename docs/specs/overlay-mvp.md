# Streamer Overlay Spec

Status: **Proposal** for [#1](https://github.com/kacky-code/kacky-live/issues/1). Replaces the original
five-widget wireframe.

Checked against the production API and KackyGG `dev` on 2026-11-07, during the live phase of
Kacky Reloaded 7 (event `26`).

## 1. Context & Surface

- **Target surface:** OBS browser source, `1920x1080`, transparent background. Later the same data
  feeds a Twitch panel (Phase 2).
- **Audience:** Viewers of one streamer. The overlay answers what chat asks most: *when is the
  streamer's map up?* and *how far are they?*
- **Not in scope:** Current map, top 5 records, linked servers and donors. Kontrol already shows
  them in game, next to where this overlay would sit.

### Why favourites drive it

A streamer focuses on 3–4 maps at a time. With 25 maps left, a list of every unfinished map is
noise. The streamer marks maps as favourites in game (Kontrol), with a type: `generic`, `easy`,
`medium`, `hard`, `impossible`. The overlay shows only the types the streamer picks.

### Event phases

| Phase | Example | Overlay |
|---|---|---|
| Live edition | KR7, `id 26`, type `KR` | Stats + up next |
| Hunting | `id 99`, type `KRHX` | Stats only, for the edition being hunted (proposal, see 2). Hunting servers play by vote, so there is no schedule. |
| Between events | – | Overall stats only, or hidden |

The overlay never assumes a server count (it changes during an edition, e.g. 15 early, 10 later).

### Placement

The page is a transparent `1920x1080` canvas with compact widgets, designed to look right at 1080p.
Where they end up is the streamer's choice: OBS already positions, crops and scales a browser
source. The overlay does not try to fit every streamer's layout.

## 2. Widgets

### Up next

The streamer's favourite maps on the live edition servers, soonest first, at most `count`.

```text
+------------------------------+
| UP NEXT                      |
| #464  S8   live · 4:12 left  |
| #503  S7   in 9:40           |
| #486  S1   in 31:05          |
+------------------------------+
```

- One row per map: map number, server tag, state and time.
- States: **live** (time left), **next** on its server, **later** (time until start).
- A row without a time (server clock unknown) shows `next` or `3 maps away`.
- No favourite on any server → widget hidden.

### Stats strip

```text
KR7 · 51 / 75 finished · rank #3 · 2 WR        898 finishes overall
```

- Event finishes, map count, event rank, world records, overall finishes.
- Between events, only overall finishes.

**Hunting (proposal):** When the streamer grinds a map from an older edition, show their
finishes in that edition next to the overall total:

```text
Hunting KR6 · 60 / 75 finished                 898 finishes overall
```

- Finishes in the edition: `GET /players/{id}/records?eventId={edition id}` (rows with
  `isFinished`); KR6 is event `11`.
- The edition comes from the map number: each edition covers a fixed range (KR6 `#376–#450`,
  KR7 `#451–#525`), listed by `GET /maps?eventId={id}`.
- The current map would come from the server the streamer is on (`serverLogin` on
  `GET /players/{id}`). Not verified yet: the field was empty during the live phase. Fallback: the
  streamer sets the edition in the URL (`&edition=KR6`).

## 3. Data

### 3.1 Public endpoints (no key)

| Data | Endpoint | Fields | Poll |
|---|---|---|---|
| Active event | `GET /events/active` | `id`, `name`, `eventType`, `endTime` | 60 s |
| Event rank, WRs | `GET /leaderboard?eventId={id}&search={nickname}` | `rank`, `finishCount`, `worldRecords` | 60 s |
| Overall finishes | `GET /players/{id}/stats` | `overall.totalFinishes`, `currentEvent.finishes` | 60 s |
| Map count | `GET /maps?eventId={id}&limit=1` | `meta.total` | On event change |

### 3.2 New endpoint (KackyGG, keyed)

The streamer's favourites are private. Their schedule comes from KackyGG's `ServerSchedule`
(on `dev`), which already powers the `!nextmaps` chat command. It reads every server's full
playlist from kontrol and adds the time limit plus a 60 s gap per map change.

```text
GET /v1/overlay/nextmaps?types=hard,impossible&count=4
X-Api-Key: kgg_...
```

| Query | Default | Rule |
|---|---|---|
| `types` | all types | Comma list of favourite types |
| `count` | `4` | 1–10 |

```jsonc
{
  "generatedAt": "2026-11-07T14:40:00Z",   // server clock, for client skew
  "edition": { "eventId": 26, "name": "Kacky Reloaded 7" },   // null between editions and in hunting
  "maps": [
    {
      "mapNumber": 464,
      "mapUid": "…",
      "favoriteType": "hard",
      "family": "KR",
      "server": { "login": "…", "tag": "S8" },
      "state": "live",            // live | next | later
      "position": -1,             // -1 live, 0 next, 1+ later
      "startsAt": null,           // estimated; null when live or unknown
      "endsAt": "2026-11-07T14:44:12Z"
    }
  ]
}
```

- Times are absolute, so the client counts down locally between polls. Poll every 20 s.
- **Key:** a new read-only streamer key with scope `overlay:read`, made on the kacky.gg profile
  next to the chat key. It can read only its owner's favourites and schedule. The chat key
  (`clips:write`) is not accepted.
- **CORS:** the overlay origin is added to the API's allowed origins (`Cors:OverlayUrl`). Closes
  [#9](https://github.com/kacky-code/kacky-live/issues/9).

## 4. URL

```text
/overlay?player=842&types=hard,impossible&count=4#key=kgg_...
```

| Param | Required | Default | Rule |
|---|---|---|---|
| `player` | yes | – | KackyGG player id, picked on `/config` by name or login |
| `types` | no | all | Passed through to the endpoint |
| `count` | no | `4` | 1–10 |
| `edition` | no | detected | Hunting only, e.g. `KR6`. Overrides the detected edition (proposal) |
| `key` (fragment) | no | – | Without it, only the stats strip shows |

The key sits in the URL fragment, so browsers never send it to the overlay host. OBS loads the
page without the key. The page's JavaScript then reads it from `location.hash` and calls the API
with `fetch()`, putting the key in the `X-Api-Key` header. OBS needs no header setting for this:
the browser source is Chromium, and the page makes the API call itself. The key still sits in the
OBS settings, which is why it is read-only.

## 5. Formatting

- **Names:** Strip Trackmania codes from event names and nicknames: colours `$rgb` (1–3 hex
  digits), styles `$o $i $w $n $t $s $g $z $m`, links `$l[...]`/`$h[...]`, `$$` → `$`.
- **Nicknames:** The game shows `customNickname` (e.g. `Worel` for `TheWoreL`). Show it stripped,
  fall back to `nickname`.
- **Times:** `m:ss`, `h:mm:ss` past an hour, as the chat commands do.
- **Visuals:** Dark translucent panels, tabular numerals, no motion except the value changes.

## 6. UI States & Fallbacks

| State | Up next | Stats strip |
|---|---|---|
| Loading | Hidden | Hidden |
| Error / offline | Keep last good, keep counting down | Keep last good |
| No key / bad key | Hidden | Shown |
| No edition running | Hidden | Overall finishes only |
| No favourite on a server | Hidden | Shown |
| 5 min without a good response | Hidden | Hidden |

Never render error text, status codes or stack traces.

## 7. Acceptance & Testing

- Renders at 1920x1080 in OBS with no scrollbars and a transparent background.
- Countdowns stay right across a 2-minute network outage and resync after.
- Unit tests: name stripping (incl. `$$`), time format, fallback states, countdown from `endsAt`.
- The key never appears in a request to the overlay host.

## Later

- **Alert:** a row flashes once when its map is 30 s from starting.
- **Overlapping favourites:** two favourites live or starting within a few minutes of each other
  on different servers. Decide how to show the clash (grouping, a marker, or order only).
- **Twitch panel:** the same data for viewers below the stream.

## Open Questions

1. **Hunting: current map.** Is `serverLogin` on `GET /players/{id}` filled while a player is
   connected to a hunting server? If yes, the hunted edition is detected automatically; if not, the
   streamer sets `&edition=` in the URL.
2. **Hunting stats:** Is "finishes in the hunted edition + overall finishes" the right pair, or do
   hunters care about something else?
