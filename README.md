# GridDown map packs

Pre-built map packs for [GridDown](https://github.com/pl0xuee/griddown), cut from
the [Protomaps](https://protomaps.com) daily planet build.

There is no source code here. The packs are built by the app's own extractor —
`src-tauri/src/bin/build-pack.rs` in the app repo — so a pack cut here is
byte-for-byte the kind of archive the app would have produced itself.

## Why this exists

The app used to build each pack on the phone, reading only the bytes it needed
out of the 100 GB planet archive. That works, but the shape of the work is
brutal: a rectangular area is not a contiguous run of tiles, and the planet
deduplicates identical tiles, so the reads scatter. Measured on the real thing:

| | Rhode Island | Oregon |
|---|---|---|
| data | 78 MB | 534 MB |
| requests | 650 | **65,160** |
| mean request | 120 KB | **8.2 KB** |

Sixty-five thousand round trips is over twenty minutes of overhead on a phone
connection before any data arrives — and every install repeated it against
somebody else's free server.

Doing it once here turns that into a single sequential download, and cuts the
total traffic to Protomaps enormously.

## Releases

Each release is tagged `packs-YYYYMMDD` and holds:

- `<STATE>.pmtiles` — one pack per state
- `starter.pmtiles` — the whole US at low zoom, ~11 MB, bundled in the app so a
  fresh install opens on a real map
- `packs.json` — the manifest the app reads

```json
{
  "planetBuild": "https://build.protomaps.com/20260719.pmtiles",
  "generated": "2026-07-20T06:00:00Z",
  "packs": {
    "RI": {
      "bytes": 78123456,
      "sha256": "…",
      "url": "https://github.com/pl0xuee/griddown-packs/releases/download/packs-20260720/RI.pmtiles"
    }
  }
}
```

`sha256` is not decoration: a 500 MB download that truncates silently becomes a
corrupt map, which is worse than one that fails.

## Building

Actions → **Build map packs** → Run workflow, with a comma-separated list of
state codes (`DC,RI`) or `starter` for the overview pack.

Adding states later merges into the existing release rather than replacing it.

## Data

Map data © OpenStreetMap contributors, ODbL, via Protomaps.
