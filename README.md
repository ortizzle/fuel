# Fuel — food, calories, macros and how you eat

Personal food log for a phone: scan a barcode, photograph a plate or a Nutrition Facts label, or add by hand; see calories and macros against targets; understand *how* you eat over time (where, when, which meals), with Claude summaries. Built in the ortizzle family-app mold: one `index.html`, no build step, localStorage as the source of truth, Arizona dates. Pixel / Chrome first, Safari still supported.

**Live app:** https://ortizzle.github.io/fuel/ (after the one-time Pages setting in *Deploying*).

## Install on the phone

Open the live URL in Chrome, tap the ⋮ menu → **Add to Home screen** (or **Install app**). It runs full-screen with its own icon; the camera and storage permissions are granted once.

## What it does

| Capture | How | Fallbacks built in |
|---|---|---|
| **Scan** a package | Live camera → the browser's native barcode detector on the Pixel → Open Food Facts lookup → product card with servings or grams → log | USDA FoodData Central lookup when Open Food Facts misses (works out of the box on USDA's shared demo key, ~30 lookups an hour; add your own free key for unlimited) · ZXing decoder on browsers without a native detector · type the digits · decode a *photo* of the barcode · not found → photograph the Nutrition Facts label · custom entry tied to the barcode and remembered for next time |
| **Photo** of a plate or a label | Photo shrunk on the phone → Claude vision with your key, straight from the phone → itemized estimate with a confidence badge per item → adjust portions, pick meal and context → log | optional hint ("6 oz chicken, olive oil dressing") · re-analyze · describe it in words instead · **the photo's own capture time is used**, so a shot from last night lands last night |
| **Add** by hand | **Recent** (anything you have logged before — tap a row to add it alone at the portion you last used, or check several and add them together) · **Quick** (~60 common foods, offline) · **Describe** in a sentence → Claude estimate · **Custom** numbers (calories computed from macros if blank) | USDA search · saved meals |

Every entry carries a **meal** (Breakfast, Lunch, Dinner, Snack, Pre-workout, Post-workout) and a **context** (Home-cooked, Takeout, Restaurant, At work, On the go, Packaged, Social / event, Other). Hand-added foods default to the last context you used; anything scanned defaults to Packaged and leaves that manual default alone. Rows are editable; deletes are tombstones so a future sync can't resurrect them.

**Serving sizes.** Some packages have no serving size on file, so the figures come back per 100 g and the scan card says so. Either type what you actually ate into "grams eaten", or open the entry and correct the gram weight: every nutrition number rescales with it, and the portion rescales the other way, so the day's total never shifts just because you fixed a label.

**Recents and saved meals.** The Recent tab is derived from your own log, so the second time you eat something is one tap at the portion you used last, with a repeat count beside it. Check several rows instead and they add together in one go, each still at its own last-used portion and context, under a single Meal you pick for the batch — logging a whole plate without opening each item. A whole meal can also be saved under a name from the day it happened, then logged again in one tap. Saved meals are records like everything else, so they back up, restore and tombstone the same way.

**Photo timing.** A photo carries the moment it was taken in its EXIF data, so an entry added from an older shot is stamped with the time you ate rather than the time you logged it, and the meal is guessed from that hour. When the photo is more than ten minutes old the panel says when it was taken and offers a "Right now" override, and the log jumps to that day so you can see where the entry landed. A photo taken seconds ago asks nothing.

**Insights** (7 / 14 / 30 days): calories per day against target, macro averages and split, calories by context, by meal, by time of day, most-logged foods, weekday vs weekend, and, when The Howlers is connected, run days vs rest days. Every chart has a table view and tap-to-read values. "Analyze these days" asks Claude for patterns, wins, watch-outs and one experiment; "Summarize today" does the same for a single day. Only aggregated numbers are sent, never photos.

**Summaries stay put.** What Claude writes is saved with the day it describes, so stepping to another date, closing the app or rebooting the phone all bring it back rather than throwing it away — one summary per day, one analysis per range, each stamped with when it ran. Log something after the fact and the card stays on screen with a note that the day has moved on, so you choose whether to spend another call re-running it. Summaries ride along in backups like everything else, and day summaries older than 90 days are dropped to keep the file small.

**Move — energy out.** A fourth capture button logs a ride, walk, strength session, mobility, run or anything else: minutes, and for rides miles and output, with the bike's own calorie figure when you have it. Workouts sit in the day's timeline beside the meals (a 06:10 ride above breakfast). Energy out gets its own ring inside the calorie ring, on the same scale but run the other way round the dial: intake climbs clockwise from the top, burn falls away counter-clockwise, so the gap between the two ends is the day's deficit at a glance. The center stays intake — burn figures are estimates, and eating them back is how a deficit evaporates — with the blue "~480 out" in the strip beneath. Insights keeps the same accounting: the calories chart is intake, and the table adds in, out and net columns for any range with movement. Calories out are the source's number where there is one; otherwise a rough MET estimate marked `~`, and only once a weight is entered in Settings — with no weight, a walk shows minutes rather than a made-up figure. Peloton history imports from the CSV the site exports (columns matched by name, ids from the ride's timestamp, so re-importing never duplicates and a deleted ride stays deleted), and Howlers history imports in one tap ahead of that app's sunset. Insights gains a Movement card (minutes per day, active days, by kind) and the run-vs-rest comparison becomes active-vs-rest from Fuel's own records.

**Targets from a goal.** Settings → *Set targets from a goal* takes weight, height, age and day-to-day activity, estimates what you burn in a day (Mifflin-St Jeor × an activity factor), lets you pick a deficit — steady loss, faster, maintain, or a typed number — and derives a macro split with the reasoning beside each line: protein first (0.7 g per lb, or 0.8 g per lb of a goal weight; the lever that holds muscle while you cut), fat at a quarter of calories with a 40 g floor, carbs from what is left, fiber at 14 g per 1,000 kcal. A typed target below the commonly used floor is flagged, not blocked. Nothing here is a prescription: once calories and protein are set, no split is provably better than another, and every number stays editable. *Ask Claude about this mix* sends the proposal plus your profile and recent movement for a few specific notes.

**What Claude knows about you.** Settings holds a short training profile — equipment, what you are open to, what is off the table for now, a goal, and an optional weight for estimates. It is context, not a plan: every day summary and range analysis carries it, so anything Claude says about exercise starts from what you are actually working with.

**The Howlers hook** (read-only): on a phone that also runs The Howlers, Fuel picks up the Howlers Gist keys from the shared `ortizzle.github.io` storage and shows the day's workout under the ring (run day / training day / rest day), splits Insights into run vs rest days, and tells Claude which days were runs. On another device, paste the Gist ID and token in Settings. Nothing is ever written to the Howlers Gist.

## The look

"Harbor": deep blue and brown. Light mode is warm cream with navy ink, navy buttons and a brown accent (the calorie ring, active states); dark mode is navy surfaces with cream ink and a caramel accent. One sans (Manrope), hairline borders, no ornament. Color otherwise carries only data: protein red, carbs gold, fat blue, fiber green, validated for color-blind separation and contrast on both surfaces; the P / C / F letters always sit beside the numbers so identity never rides on color alone. Appearance follows the phone (System) with Light / Dark overrides in Settings.

## Settings and keys

- **Daily targets** for calories, protein, carbs, fat, fiber.
- **Claude API key** (photos, descriptions, summaries). Stored only in this browser; calls go straight from the phone to Anthropic. Set a monthly spend limit on the key. Model: Sonnet 5 by default, Opus 5 or Haiku 4.5 selectable.
- **USDA key** (optional, free): removes the shared demo-key limit for barcode fallback and search.
- **Daily targets**: calories, macros, fiber, minutes moved — or *Set targets from a goal* to derive them from weight, height, age, activity and a deficit.
- **Movement**: equipment, open to / not right now / goal, weight for estimates only, a daily minutes target (with the other targets), Peloton CSV import, Howlers import.
- **The Howlers**: connection status, refresh, whose workouts.
- **Backup**: save a dated JSON file holding the log, targets and remembered barcodes. On Android the share sheet sends it straight to Drive; elsewhere it downloads. Restoring merges by entry, newest wins, so an old file can never delete newer work or resurrect something you deleted. The panel shows how long it has been and turns amber after two weeks.
- **Data**: load two sample weeks, copy the JSON, clear this device.

## Data model (sync-ready)

```
localStorage fuel_data      { records: { <id>: entry } }
entry = { id, type:'entry', date:'YYYY-MM-DD' (AZ), meal, context, time,
          name, brand, source:'upc'|'photo'|'manual'|'ai'|'quick'|'usda', barcode,
          servings, serving:{ label, grams }, per:{ kcal, protein, carbs, fat, fiber },
          note, createdAt, updatedAt, deleted? }
savedmeal = { id, type:'savedmeal', name, meal, items:[...], createdAt, updatedAt, deleted? }
summary  = { id:'sum_day_<date>' | 'sum_range_<7|14|30>', type:'summary', kind:'day'|'range',
             date | range + endDate, model, fingerprint, result, createdAt, updatedAt, deleted? }
workout  = { id (wk_pel_<date>_<hhmm>_<kind> for Peloton imports, wk_howl_… for Howlers), type:'workout',
             date, time, kind:'ride'|'walk'|'strength'|'mobility'|'run'|'other', title, minutes,
             miles?, output?, kcal?, avgHr?, source:'manual'|'peloton'|'howlers', note, createdAt, updatedAt, deleted? }
localStorage fuel_settings  { targets, scheme, model }
localStorage fuel_products  { <barcode>: product }          remembered scans / custom products
localStorage fuel_howlers_cache                              workouts by date (read-only mirror)
localStorage fuel_anthropic_key, fuel_usda_key, fuel_howlers  entered in Settings, never in the HTML
```

Records carry `id` / `updatedAt` / tombstones, so the canonical Gist safe-merge can be added without a migration. Because every ortizzle app shares the `ortizzle.github.io` origin, data logged under one path survives a move to another path on the same host.

## Repo layout

- `index.html` — the whole app (CSS and JS inline; ZXing loads from a CDN only when a browser lacks a native barcode detector).
- `manifest.webmanifest` — the web app manifest. It must be a real file at a real URL: Chrome's
  install service fetches it (and the icons) from the server to build the home-screen app.
- `icon.svg`, `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png` — the app mark.
- `sw.js` — service worker: offline shell, network-first for the page so deploys land immediately.
- `.nojekyll` — tells GitHub Pages to serve the files as they are.
- `.claude/launch.json` — local run config (`python3 -m http.server 4400`, open `http://localhost:4400/`).

## Deploying

GitHub Pages serves `main` directly, like the other ortizzle apps. One-time setup: **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `(root)` → Save.** After that every push to `main` goes live within a minute or two. Chrome caches hard: verify a change with `?v=<timestamp>` appended to the URL, or reopen the installed app after a few minutes.

## Known limits and challenges

- **Barcode coverage** is the biggest practical risk: Open Food Facts misses a share of US store brands and new products; the USDA fallback and the remembered-products memory cover most of the rest, and "photograph the label" covers the tail.
- **Photo estimates** are estimates: good at identifying items and reading labels, typically 20–30% off on mixed plates, blind to oils and sauces. The hint line and confidence badges exist for that; a kitchen scale plus the grams box beats any AI.
- **Trends only count logged days**, so consistency matters more than precision; recents / favorites / saved meals is the next feature for that reason.
- **Storage** is on-device. Chrome keeps it unless browsing data is cleared, and the app requests persistent storage. Back up from Settings every couple of weeks; the panel nags when it is stale. Fully automatic off-device backup needs Gist sync, which is not built yet.
- **Workouts** come from The Howlers until that season ends in January; Strava or Garmin export can replace it later.

## Roadmap

Recents / favorites / saved meals → training-day targets (more carbs on long-run days, driven by the Howlers hook) → Gist sync with the canonical safe-merge → weekly review → service worker for offline shell → Strava or Garmin export as the workout source after January.

**Parked: weight as an outcome measure.** Not a daily weigh-in, which Chris does not want and
which is mostly noise anyway. The point is closing the loop: with intake already logged and
workouts already read from The Howlers, two weight readings roughly a month apart are enough
to derive actual maintenance calories (about 3,500 kcal per pound of change) and say whether
the calorie target is really a deficit. Design notes when it is picked up: weight is a second
record type alongside `entry`, so it inherits backup, restore-merge and tombstones for free;
log it from the Log screen rather than Settings, which holds only goal and units; show a
seven-day average with daily readings faint behind it, and report change as pounds per week
fitted across the range, never as today versus yesterday.
