# LabLunar — a lunar outpost simulator for junior astronauts

Entry for the **NASA Space Apps Challenge 2026**, challenge *"Build a mission simulator for junior astronauts"*.

The player runs a small lunar outpost for **56 days** (two lunar day/night cycles of 14 days each) and
keeps three numbers alive: **energy**, **oxygen** and **radiation dose**. Solar storms and equipment
failures force decisions every day. Every choice has a cost — there is no perfect base.

- **One file, no server.** Everything is in `index.html`: HTML, CSS and JavaScript.
- **No login, no install, no backend.** Double-click and play.
- **Language:** English, written for ages 7–12, with terms explained where they appear.
  (The first build was in Brazilian Portuguese for a local audience; the interface is now English-only.)
- **Only external resource:** Google Fonts (the page still works without it).

---

## How to run

1. Open `index.html` in a current browser (Chrome, Edge, Firefox, Safari).
2. Choose the base, then play one day at a time, or use *“Avançar até algo importante”* to skip to the
   next warning, storm or blackout.

That is all. JSON files are **embedded** in the HTML by `tools/build.py` because browsers block
`fetch()` of local files under `file://`; the files in `data/` are the source of truth and the
deliverable, the copy inside the HTML is what the page reads.

Optional local server (not required):

```bash
python -m http.server 8000      # then open http://localhost:8000/index.html
```

### Simulation mode (balance testing)

Open `index.html?sim=1000` (or run `LabLunar.runAll(1000)` in the DevTools console). The result is
written to `window.__SIM_RESULTS` and to the console; nothing changes on screen.

---

## Repository layout

```
index.html                  the whole game (data embedded)
data/game_constants.json    design numbers: energy, oxygen, radiation, mass, failure rates
data/donki_snapshot.json    NASA DONKI: SEP events (storms) and FLR alerts (warnings only)
data/osdr_snapshot.json     NASA OSDR: dataset references + dose values (illustrative)
data/artemis_snapshot.json  lunar day/night length and surface temperatures
data/sources.md             every request: exact URL, date, HTTP code, result
tools/fetch_donki.py        real DONKI download (requires NASA_API_KEY)
tools/build_data.py         builds data/*.json + sources.md from the raw download
tools/build.py              embeds data/*.json into index.html
CHANGELOG.md                what changed, and the fact that no prototype was provided
```

---

## How each dataset is used

| Dataset | What it provides | How the game uses it |
|---|---|---|
| **NASA DONKI** — `SEP` | Real solar energetic particle events (date, instruments, link) | **Only source of solar storms.** Each event becomes a storm on its real date inside the 56-day window; the storm lasts 2 days and raises radiation. |
| **NASA DONKI** — `notifications?type=FLR` | Real flare alerts with class (e.g. M5.9, X1.9) | **Warning label only**, shown the day before a storm. A flare never creates a storm. |
| **NASA OSDR** | Public metadata for space-radiation studies (e.g. OSD-570 / Inspiration4) | Inspires the *accumulated dose* model. Dose numbers are **illustrative**, not medical data. |
| **NASA Moon Facts** | Lunar cycle ≈ 29.5 days; 127 °C day, −173 °C night | 14 days of sun + 14 of night (derived: half of 29.5, rounded). |
| `data/game_constants.json` | Design numbers | Mass budget, kWh, oxygen rates, failure chance. **All illustrative**, tuned by the balance tests. |

### Data honesty rules used in this project

- No endpoint, parameter or event ID was invented. Endpoints were called for real; see `data/sources.md`.
- Data that could not be downloaded — or could not be reproduced — is marked `"status": "pending_download"`
  and the game **does not** present it as real.
- `SEP` is the primary storm source; `FLR` is used for warnings only.
- Any number without a real source is labelled **illustrative** in the JSON and in the game text.

### Current data status (2026-11-15)

| Source | Status | Detail |
|---|---|---|
| DONKI SEP | **`ok`** | **27 real events** downloaded from **2024-04-25 to 2024-06-18** (HTTP 200, personal `NASA_API_KEY`). Storms in the game use these real dates. |
| DONKI FLR | **`ok`** | **88 real flare alerts** from the same window (HTTP 200) — used only for the one-day-ahead warning label. |
| OSDR | ok | Search API returned HTTP 200, 107 hits (e.g. GSE104121, GSE107057). |
| Moon Facts | ok | Cycle length and temperatures taken from the official page. |

Why this window: a sweep of 13 consecutive 55-day windows (2024-03 to 2025-12) showed solar activity was
uneven; 2024-04-25..2024-06-18 had the most SEP events, so a 56-day mission inside it contains three or
four storms on real dates. Every request is listed in `data/sources.md`.

Earlier exploratory calls used the shared `DEMO_KEY` and hit **HTTP 429**; that history is kept in
`data/sources.md` and is not used as game data.

### Completing the solar data

```bash
export NASA_API_KEY="your-key"        # Git Bash; PowerShell: $env:NASA_API_KEY="your-key"
py -3 tools/fetch_donki.py --start 2024-04-25 --end 2024-06-18
py -3 tools/build_data.py
py -3 tools/build.py
```

Alternatively keep the key in a private file (already git-ignored) and pass `--key-file secrets/nasa_key.txt`;
the script prints only the key length, never the value.

`fetch_donki.py` reads the key from the environment or from `--key-file`, refuses to run without one,
never writes it to a file or log, rejects the placeholder text, and splits the range into 55-day
windows (the API refuses more than 60 days per call). When the SEP endpoint returns HTTP 200, the
snapshot flips to `status: "ok"` and storms appear with their real dates.

Get a free key at <https://api.nasa.gov> (sign-up form, key arrives by email; it is personal, so never
commit it).

---

## Balance results

**1,000 simulated missions per configuration**, player policy = careful: switch to *economia* when the
battery drops below 25%, to *rigoroso* below 8%, and shelter during a storm whenever the base has a shield.

| Configuration | Win rate |
|---|---|
| Default base, careful play (6 panels, 6 batteries, 1 greenhouse, recycler, water wall — 86/100 mass) | **84.1 %** |
| Default base, **ignoring storm warnings** (never shelters) | **43.3 %** (radiation caused 486 of the 567 losses) |
| No recycler | 0 % |
| No greenhouse | 0 % |
| Small battery (3 batteries) | 0 % |
| No radiation shield | 0 % |

Defeat causes in the default base: oxygen 136/1000, energy 23/1000.

Measured with the **real SEP calendar** (2024-04-25..2024-06-18), 1,000 missions per configuration.
Each mission uses three or four storms on real DONKI dates, selected from the window by the simulator;
the mission always starts three days before the first storm, so the warning always has a day to appear.
This is why ignoring the warning costs 40 percentage points: a shielded base that shelters during a
storm absorbs about 6 dose points per storm, while an unsheltered one absorbs about 26.

How to reproduce: open `index.html?sim=1000`, or in DevTools:

```js
LabLunar.runAll(1000)
```

---

## Game rules (short)

- **56 days**, repeating 14 days of sun and 14 of night. Panels only produce during the day; batteries
  are the only source at night.
- **Mass budget 100.** Panels 4, batteries 6, greenhouse 8, recycler 6, shield 0 / 5 / 9 / 12.
  The design screen previews daily production, consumption, night demand and how many nights the
  batteries cover.
- **Daily choices:** energy-saving mode (normal / economia / rigoroso — less power use, less oxygen
  production) and whether to shelter the crew (only possible with a shield).
- **Storms:** warning one day ahead, high radiation for two days, partly blocked by shield and shelter.
- **Failures:** random equipment damage; repairs consume spare parts (3 at start).
- **Defeat:** oxygen at 0, radiation dose above 100 %, or three consecutive days with an empty battery.
  **Victory:** survive 56 days.
- **Final screen:** result, plain-language post-mortem, one chart per resource, replay.

---

## Accessibility and constraints

- Works on phone and desktop; large touch targets; no colour-only meaning.
- No 3D, no multiplayer, no story mode, no ranking, no ads, no analytics.
- **No NASA or Space Apps logo** is used, copied or redrawn.

---

## Data attribution

Displayed in the footer and on the final screen:

> Data: NASA DONKI (snapshot of 15/11/2026); NASA OSDR; NASA Artemis.

Radiation values in the game are **illustrative** and are **not medical advice**.

---

## Declaration of AI use

This project was built with an AI coding assistant (Freebuff/Buffy) under human direction:

- **Code, prose and balance tuning:** written with the assistant, reviewed by the author.
- **Data:** every NASA number shown in the game comes from a real request recorded in
  `data/sources.md`, or is explicitly marked *illustrative*. No event, ID or endpoint was invented.
- **Provenance:** the assistant performed the API calls, saved the raw responses, and labelled what it
  could not obtain as `pending_download` instead of filling the gap.
- **No AI-generated imagery** is used; there are no images at all.

---

## License

- **Code and text:** MIT — see `LICENSE`.
- **NASA data:** NASA content is generally not subject to copyright in the United States; follow the
  [NASA media guidelines](https://www.nasa.gov/nasa-brand-center/images-and-media/). The DONKI and OSDR
  services are used through their public APIs; OSDR controlled-access data was **not** downloaded.
- **Fonts:** Nunito, served by Google Fonts under the SIL Open Font License.

---

## Known limitations

1. **Storms come from one real window** (2024-04-25..2024-06-18). A different period needs another
   `fetch_donki.py` run; the game never fabricates an event when the snapshot is missing.
2. The 14/14 split is a round number derived from 29.5 days; reality is ~14.75.
3. Dose, mass, kWh and failure rates are game design values, not measurements.
4. There was **no reference prototype** in the project folder, so `index.html` was written from the
   brief; see `CHANGELOG.md`.
