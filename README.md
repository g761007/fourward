# FOURWARD

A one-page retrospective of a single ride: the Taiwan 620 "Four Extremes" challenge,
Fulong → Cape Fugui → Guosheng Lighthouse → Eluanbi, 590.9 km in 26 hours on
6–7 November 2026. The page is rebuilt second by second from the head unit's FIT
recording and is written in Traditional Chinese.

**Live page:** https://g761007.github.io/fourward/

## What the page does

- Scrolling replays the ride. A pinned "head unit" panel shows the map, clock, speed,
  heart rate, power, temperature and gear at that moment.
- The page background is the sky the rider was under: the sun's altitude is computed
  from the GPS position and time, so the page goes from night to dawn to day and back.
- Later sections chart the whole ride, heart-rate and power zones, gear usage, stops,
  and a physics estimate of how much wind and drafting helped.
- It ends with a poster drawn from the ride data that can be saved as a PNG.

`index.html` is the entire site: markup, styles, script and data in one file. It makes
no network requests, so it also works offline when opened from disk.

## Repository contents

| Path | Published | What it is |
|---|---|---|
| `index.html` | yes | The generated page |
| `README.md` | yes | This file |
| `tools/` | no (local only) | The generator, see below |
| `fourward.fit` | no (never) | The raw recording: location and health data |

`tools/` and the FIT file are listed in `.gitignore`. Remove the `tools/` line there to
start versioning the generator.

## Requirements

- **Viewing:** a current browser. The page relies on CSS `@property`, `subgrid` and
  `svh` units, so roughly Chrome/Edge 117+, Safari 16.4+ or Firefox 128+. It was
  checked in Chromium at desktop and phone sizes; Safari and Firefox are untested.
- **Fonts:** system fonts only. It is designed around Songti TC, PingFang TC and
  DIN Condensed (present on Apple platforms) and falls back to Noto / system fonts
  elsewhere, where it will look plainer.
- **Building:** Python 3 with the standard library only. Developed with Python 3.14;
  older 3.x versions are untested.

## Installation

There is nothing to install and no dependencies to fetch. To view the page, open the
live link above or open `index.html` in a browser.

## Configuration

There are no environment variables and no config files. Everything adjustable lives at
the top of `tools/build.py`:

- `STATIONS`, `LEGS`: the rider's checkpoint sheet, matched to stops and laps in the data
- `MASS`, `CDA`, `CRR`, `RHO`, `ETA`: assumptions of the wind/drafting model
- `MOVING_KMH`: the speed below which the wheels count as stopped
- `EAST_COAST`: a hand-drawn outline used only as map context

## Development

The generator needs `tools/` and a FIT file, which are not part of the public repository.

```bash
python3 tools/build.py                      # reads fourward.fit, writes index.html
python3 tools/build.py ride.fit out.html    # explicit input and output
```

| File | Role |
|---|---|
| `tools/build.py` | Decodes the ride, derives every number, assembles the page |
| `tools/story.py` | The prose; every figure in it is computed, none is typed in |
| `tools/fit_decoder.py` | Dependency-free FIT decoder |
| `tools/astro.py` | Sun and moon positions |
| `tools/template.html`, `page.css`, `page.js` | Page source, inlined into `index.html` |

Edit those files, never `index.html` itself: it is overwritten on every build.

## Tests

There is no separate test suite. The build checks itself and stops with an
`AssertionError` if something does not hold:

- per-second sums are compared with the head unit's own session summary
  (distance, pedal revolutions, work, normalized power);
- every claim in the story is asserted against the data before it is written
  (for example "the big ring was never left" or "every heart-rate dropout fell in a rest").

## Deployment

GitHub Pages serves `index.html` from the root of the `main` branch. To update the
live page, rebuild and push:

```bash
python3 tools/build.py
git add index.html
git commit -m "Rebuild page"
git push
```

The page carries `<meta name="robots" content="noindex">`: it is public to anyone with
the link but asks search engines not to list it.

## Troubleshooting

- **`AssertionError` during the build.** The data and the story disagree. `story.py`
  and `STATIONS` are written for this one ride, so a different FIT file will trip them
  until they are adapted.
- **`no long stop near km …`.** A station in `STATIONS` no longer matches a stop of
  three minutes or more in the recording.
- **The page looks different on Windows or Android.** The Apple system fonts are
  missing and the fallbacks are used.
- **The poster does not download.** Some embedded previews block downloads; open the
  page in a normal browser tab.
- **The live page still shows the old version.** GitHub Pages can take a minute to
  deploy; reload without cache.

## Data and method notes

- Times, distance, heart rate, power, cadence, temperature, gear shifts and rider
  position come straight from the recording. Per-leg ascent, normalized power and time
  in zones are the head unit's own figures.
- Sunrise, sunset, twilight and the moon are computed for a sea-level horizon with no
  terrain, good to about a minute for the sun and a few minutes for the moon.
- The "ghost rider" is a power-balance model, not a measurement, and no wind data was
  recorded. Its assumptions and the range they produce are stated on the page.
- The route on the maps is the real GPS track. The east coast outline is a sketch.
- Place names for moments that are not stations were identified from coordinates.
