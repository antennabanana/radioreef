# RadioReef V2

A single-file, zero-dependency RF spectrum reference portal. No build tools, no framework, no PWA — just `index.html` and a handful of self-hosted font files.

Live at: **radioreef.com**

---

## What it is

RadioReef is an interactive reference for the radio frequency spectrum from **15 kHz to 5 GHz**. It covers band definitions, modulation schemes, propagation characteristics, well-known frequencies, shortwave broadcast schedules, RF calculators, and a full glossary — all in one file you can open offline.

---

## Repository layout

```
radioreef/
├── index.html          # Entire application (4,296 lines, ~831 KB)
├── CNAME               # radioreef.com
├── README.md
└── fonts/
    ├── JetBrainsMono-Bold.woff2
    ├── JetBrainsMono-Light.woff2
    ├── JetBrainsMono-Medium.woff2
    ├── JetBrainsMono-Regular.woff2
    └── JetBrainsMono-SemiBold.woff2
```

Everything is baked into `index.html`. There are no external network requests at runtime.

---

## Features

### Spectrum browser
- **7 bands**, **90 sub-bands** covering 15 kHz – 5 GHz
- Each sub-band shows: frequency range, wavelength, modulation types, usage, propagation mode, maximum range, skip zone, and a full description
- Animated wavelength SVG visualisation scales with the selected wavelength

| Band | Range | Sub-bands |
|------|-------|-----------|
| VLF  | 15 – 30 kHz       | 3  |
| LF   | 30 – 300 kHz      | 7  |
| MF   | 300 kHz – 3 MHz   | 5  |
| HF   | 3 – 30 MHz        | 34 |
| VHF  | 30 – 300 MHz      | 14 |
| UHF  | 300 MHz – 3 GHz   | 21 |
| SHF  | 3 – 5 GHz         | 6  |

### Well-Known Frequencies (WKF)
**370 entries** across 10 countries, organised per sub-band. Each entry lists frequency, callsign/identifier, mode, and a description. Automatically filtered to show only entries that fall within the selected sub-band's frequency range.

| Country | Entries |
|---------|---------|
| USA     | 83 |
| UK      | 81 |
| Russia  | 68 |
| Japan   | 23 |
| Germany | 22 |
| Australia | 20 |
| Canada  | 20 |
| France  | 18 |
| Netherlands | 18 |
| India   | 17 |

### Shortwave Broadcast Finder (BC)
Powered by the **EiBi A26 schedule** — 8,919 broadcast entries baked inline.

- Enter a city name (617 cities + all country capitals supported)
- Optionally filter by frequency (accepts kHz or MHz, e.g. `9410 kHz`, `9.41 MHz`)
- Optionally enter a UTC time; defaults to current UTC if left blank
- Results sorted by frequency, capped at 200 rows; day-of-week filter applied automatically
- Shows: frequency, station name, language, UTC time window, days active, ITU country code

### Propagation reference
- Atmosphere layers diagram (SVG): Troposphere, Stratosphere, D/E/F layers with altitude annotations
- Propagation modes table: ground wave, sky wave, NVIS, line-of-sight, tropo ducting, sporadic-E, meteor scatter, EME
- Real-time day/night world map (SVG, UTC-accurate solar terminator)
- Per-band propagation summary table (VLF → SHF)

### Glossary
**263 terms** covering modulation schemes, propagation phenomena, antenna types, operating modes, regulatory bodies, and general RF terminology. Organised into categories.

### RF Calculators
Four calculator tabs:

| Tab | Formula |
|-----|---------|
| Wavelength ↔ Frequency | λ = c / f (c = 299,792,458 m/s) |
| Half-Wave Dipole | half-dipole = 0.475 × λ, quarter-wave = half-dipole / 2 |
| Free-Space Path Loss | FSPL = 20·log₁₀(d_km) + 20·log₁₀(f_MHz) + 32.45 dB |
| Power / Doppler | dBm ↔ watts; Δf = f₀ × v/c |

All inputs accept flexible unit notation (e.g. `14.074 MHz`, `21.3 m`, `2.4 GHz`, `50 km/h`).

### Global search
- Press `/` to focus the search bar from anywhere
- Searches band names, sub-band names/descriptions, glossary terms/definitions, and WKF entries simultaneously
- Band/sub-band results are clickable — navigate directly to the sub-band
- Glossary results link to the Glossary panel
- Results grouped by category with match counts

### Keyboard navigation
- `↑` / `↓` — move between bands and panel buttons (Glossary, Propagation, BC, CALC, Search)
- `→` — enter the first sub-band of the active band
- `←` — return from sub-band to band level
- `↑` / `↓` while a sub-band is selected — step through sub-bands
- `D` / `L` — toggle dark / light theme (or click the theme button)
- `/` — focus global search input
- `Enter` in search input — run search; live results appear after 2 characters

### UTC clock
Live UTC time displayed in the top bar, updated every second.

### Dark / light theme
Persisted to `localStorage`. All colours, borders, and accents have light-mode equivalents.

---

## Data sources

| Dataset | Description | Size |
|---------|-------------|------|
| EiBi A26 | Shortwave broadcast schedule (season A26, 2026) | 8,919 rows |
| WKF | Hand-curated well-known frequencies for 10 countries | 370 entries |
| BANDS | Hand-authored band and sub-band reference data | 7 bands / 90 subs |
| GLOSSARY | RF terminology definitions | 263 terms |
| CITY_MAP | City → EiBi target-region mapping | 617 cities |

---

## V2 build process

V2 was assembled from four source segments concatenated in order:

```
v2_head.html    — HTML skeleton, CSS (all panels), inline data constants
                  (BANDS, GLOSSARY, WKF, CITY_MAP, TARGET_NAMES)
eibi_data.js    — const EIBI=[...] — single line, 495 KB
v2_tail_a.txt   — State model, all JS functions
v2_tail_b.txt   — buildAtmoDiagram(), buildPropModes(),
                  buildWorldMap(), buildPropBandTable()
                  (extracted verbatim from original index.html)
v2_tail_c.html  — Ripple handler, UTC clock, keyboard nav,
                  global search listener, </script></body></html>
```

Final assembly command:
```bash
cat v2_head.html eibi_data.js v2_tail_a.txt v2_tail_b.txt v2_tail_c.html > index.html
```

### EiBi data pipeline
```
eibi_a26.csv  →  parse_eibi.js (Node.js)  →  eibi_data.js
```
`parse_eibi.js` reads the raw EiBi CSV, strips headers, normalises day strings, and emits `const EIBI=[[freq_khz, start_hhmm, end_hhmm, days, itu, station, lang, target], ...]`.

Day encoding: digit string where `1`=Mon … `7`=Sun; empty string or `"1234567"` = every day.

---

## State model

```javascript
let activeView = 'intro';  // 'intro'|'band'|'glossary'|'propagation'|'broadcast'|'calc'|'search'
let activeBand = null;     // band id string | null
let activeSub  = null;     // sub-band name string | null
let lastSearchQuery = '';
```

`goHome()` resets all three to their initial values and calls `showIntro()`.

`refreshCurrentView()` re-renders the current panel (used after theme toggle).

---

## Key design decisions

- **Single file** — no build step, no bundler, works offline by opening `index.html` directly
- **No external requests** — all fonts, data, and code are local
- **CRT / cyberpunk aesthetic** — dark background, green/amber/cyan accents, monospace font throughout
- **Frequency scope** — 15 kHz to 5 GHz; no EHF/mmWave extension
- **EiBi inline** — the 495 KB schedule is baked in rather than fetched, keeping the app fully self-contained
- **WKF filtered per sub-band** — `buildWKF()` applies a ±5% frequency window so only relevant entries appear
- **BC day filter** — JavaScript's `getUTCDay()` (0=Sunday) is remapped to EiBi's convention (7=Sunday) before filtering

---

## CSS architecture

All styles are in a single `<style>` block. Panel-specific class namespaces:

| Prefix | Panel |
|--------|-------|
| `.bc-` | Shortwave Broadcast Finder |
| `.calc-` | RF Calculators |
| `.search-` | Global Search |
| `.wkf-` | Well-Known Frequencies table |
| `.glossary-` | Glossary grid |
| `.band-btn` / `.sub-btn` | Left nav / sub-band list |

Light-mode overrides are scoped under `html.light`.

---

## Testing

Automated checks run against the assembled file using Node.js with a mock DOM (`vm.runInContext`):

- JS syntax validation
- All 10 structural DOM IDs present
- All 35+ CSS panel classes defined
- All 17 key functions present and callable
- Runtime data counts: 7 bands, 90 sub-bands, 8,919 EiBi rows, 617 cities, 263 glossary terms, 370 WKF entries
- `runBroadcast('London', '', '1200')` returns 65 result rows
- `buildWKF('hf', '14.0–14.35 MHz')` returns populated HTML
- `formatDays`, `formatTime`, `fmtFreqKhz` produce correct output
- EiBi day mapping (JS 0=Sun → EiBi 7=Sun) verified

Local preview:
```bash
node -e "
const http=require('http'),fs=require('fs'),path=require('path');
http.createServer((req,res)=>{
  const f=path.join(__dirname,req.url==='/'?'index.html':req.url.slice(1));
  try{res.writeHead(200);res.end(fs.readFileSync(f));}
  catch{res.writeHead(404);res.end();}
}).listen(8765,()=>console.log('http://localhost:8765'));
"
```

---

## Updating the EiBi schedule

EiBi publishes a new shortwave broadcast schedule twice a year:

- **A season** — late March through late October (e.g. A26 = 2026)
- **B season** — late October through late March (e.g. B25 = 2025/26)

The schedule is baked into `index.html` as a single `const EIBI=[[...]]` line (~495 KB). Updating it is a four-step process: download → parse → replace → verify.

### Step 1 — Download the new EiBi CSV

Go to **http://www.eibispace.de** and download the CSV file for the new season. The download is listed under the "sked" section; the file is named `sked-a26.csv`, `sked-b26.csv`, etc.

Save it anywhere accessible, e.g. `/tmp/sked-new.csv`.

The CSV is semicolon-separated with this header and column layout:

```
kHz;Time(UTC);Days;ITU;Station;Lng;Target;Remarks;P;Start;Stop
16.3;0000-2400;;IND;VTX1 Indian Navy;;SAs;v;1;;
603;1500-1600;;CHN;China Radio Int.;VN;SEA;d;1;;
```

| Col | Field | Used | Notes |
|-----|-------|------|-------|
| 0 | kHz | Yes | Transmit frequency in kHz |
| 1 | Time(UTC) | Yes | `HHMM-HHMM` start–end window |
| 2 | Days | Yes | Digit string: `1`=Mon … `7`=Sun; blank = daily; `1234567` = daily |
| 3 | ITU | Yes | Transmitter country (ITU code) |
| 4 | Station | Yes | Station name (truncated to 40 chars) |
| 5 | Lng | Yes | Language code (truncated to 6 chars) |
| 6 | Target | Yes | Target region code (truncated to 8 chars) |
| 7+ | Remarks, P, Start, Stop | No | Not used |

### Step 2 — Parse the CSV into compact JS

Run the parser script below. It reads the CSV, skips the header row, and emits a single-line `const EIBI=[[...]]` JS file.

Save this as `parse_eibi.js` and update the two paths at the top:

```javascript
// parse_eibi.js
const fs = require('fs');

const INPUT  = '/tmp/sked-new.csv';       // ← path to downloaded CSV
const OUTPUT = '/tmp/eibi_data.js';       // ← output path (temporary)

const lines = fs.readFileSync(INPUT, 'utf8').split('\n');
const entries = [];

for (let i = 1; i < lines.length; i++) {
  const line = lines[i].trim();
  if (!line) continue;
  const parts = line.split(';');
  if (parts.length < 7) continue;

  const [freqStr, timeStr, days, itu, station, lang, target] = parts;

  const freq = parseFloat(freqStr);
  if (isNaN(freq)) continue;

  const timeParts = timeStr.split('-');
  if (timeParts.length < 2) continue;
  const startTime = parseInt(timeParts[0], 10);
  const endTime   = parseInt(timeParts[1], 10);
  if (isNaN(startTime) || isNaN(endTime)) continue;

  entries.push([
    freq,
    startTime,
    endTime,
    days.trim().slice(0, 10),
    itu.trim().slice(0, 5),
    station.trim().slice(0, 40),
    lang.trim().slice(0, 6),
    target.trim().slice(0, 8)
  ]);
}

const output = 'const EIBI=' + JSON.stringify(entries) + ';';
fs.writeFileSync(OUTPUT, output);
console.log('Parsed ' + entries.length + ' entries — ' + (output.length / 1024).toFixed(1) + ' KB');
```

Run it:

```bash
node parse_eibi.js
# Parsed 8919 entries — 483.8 KB   ← row count will differ for a new season
```

The output file is a single line with no trailing newline beyond the `;`.

### Step 3 — Replace the EIBI line in `index.html`

The EIBI data occupies exactly one line in `index.html` (line 2,776 in V2). The following Node.js one-liner finds that line by its prefix, replaces it, and writes the file back in place:

```bash
node -e "
const fs      = require('fs');
const newData = fs.readFileSync('/tmp/eibi_data.js', 'utf8').trim();
const html    = fs.readFileSync('index.html', 'utf8');
const lines   = html.split('\n');
const idx     = lines.findIndex(l => l.startsWith('const EIBI='));
if (idx < 0) { console.error('ERROR: EIBI line not found in index.html'); process.exit(1); }
const oldLen  = lines[idx].length;
lines[idx]    = newData;
const updated = lines.join('\n');
fs.writeFileSync('index.html', updated);
console.log('Replaced line ' + (idx + 1));
console.log('Old EIBI line: ' + (oldLen / 1024).toFixed(1) + ' KB');
console.log('New EIBI line: ' + (newData.length / 1024).toFixed(1) + ' KB');
console.log('Total file:    ' + (updated.length / 1024).toFixed(1) + ' KB');
"
```

Run this from the repository root (where `index.html` lives). Adjust `/tmp/eibi_data.js` if you used a different output path in Step 2.

Expected output (numbers will vary by season):

```
Replaced line 2776
Old EIBI line: 483.4 KB
New EIBI line: 491.2 KB
Total file:    835.0 KB
```

### Step 4 — Verify

Confirm the new schedule loaded correctly:

```bash
node -e "
const html  = require('fs').readFileSync('index.html', 'utf8');
const lines = html.split('\n');
const idx   = lines.findIndex(l => l.startsWith('const EIBI='));
const eibi  = JSON.parse(lines[idx].slice('const EIBI='.length, -1));
console.log('EIBI rows:   ', eibi.length);
console.log('First entry: ', JSON.stringify(eibi[0]));
console.log('Last entry:  ', JSON.stringify(eibi[eibi.length - 1]));
// Check structure: each row must have 8 fields
const bad = eibi.filter(r => r.length !== 8);
console.log('Malformed rows:', bad.length === 0 ? 'none ✓' : bad.length + ' PROBLEMS');
"
```

Expected structure for each row:

```
[freq_khz, start_hhmm, end_hhmm, days, itu_code, station_name, language, target_region]
  number    integer     integer   str    str         str           str       str
```

Example:
```json
[9410, 800, 900, "12345", "GBR", "BBC World Service", "E", "Eu"]
```

- `freq_khz` — frequency in kHz (e.g. `9410` = 9.410 MHz)
- `start_hhmm` / `end_hhmm` — UTC time as integer HHMM (e.g. `800` = 08:00, `1430` = 14:30)
- `days` — digit string `"1"–"7"` where 1=Mon, 7=Sun; empty string or `"1234567"` = broadcasts every day
- `itu_code` — ITU transmitter country (e.g. `"GBR"`, `"CHN"`, `"USA"`)
- `station_name` — broadcast station name
- `language` — language code
- `target_region` — EiBi target region code (e.g. `"Eu"`, `"NAm"`, `"SEA"`)

### Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `ERROR: EIBI line not found` | The `const EIBI=` line was split or moved | Search for `const EIBI` in `index.html` manually and check line structure |
| `Malformed rows: N PROBLEMS` | New CSV uses a different column order | Check EiBi header line against the column table in Step 1 and update `parse_eibi.js` accordingly |
| Row count is 0 | CSV path wrong or file is empty | Re-check `INPUT` path in `parse_eibi.js` |
| BC panel returns no results | Day/time filter too strict | Test with a blank time field so the BC panel defaults to current UTC |
| File size grew unexpectedly large | `newData` was not trimmed | Ensure `fs.readFileSync(..., 'utf8').trim()` is used in Step 3 |
