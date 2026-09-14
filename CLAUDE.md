# Fuel — working notes

## Before you edit

This app is worked on from two places: the local clone in `~/Downloads/projects/fuel`, and
Claude Code cloud sessions, which clone fresh and push straight to GitHub. Whichever copy is
in front of you may not be the newest.

Run `git fetch origin && git status -sb` before changing any file, and say plainly where
things stand — behind, ahead, or carrying uncommitted work. If the clone is behind, pull
first rather than after. A merge git resolves cleanly can still be wrong: git only stops on
changes that *overlap*, so two sessions editing the same function from different sides merge
quietly into something broken.

## What this is

Single-file PWA. `index.html` holds all CSS and JS. No build step, no framework.
Follows the ortizzle family-app standards: createElement only, never `innerHTML` with
data, no `alert`/`confirm`/`prompt`, 44px tap targets, Arizona dates.

## The Harbor palette

Deep blue and brown. Light is warm cream with navy ink and a leather accent; dark is
navy with bone ink and a brass accent. Colour is reserved for data and state — the
interface itself is navy, cream and brown only.

| Role | Name | Light | Dark |
|---|---|---|---|
| Page ground | Sailcloth / Deepwater | `#f3eee5` | `#0e1626` |
| Card surface | Chalk / Hull | `#fffcf7` | `#172236` |
| Raised surface | Drift / Berth | `#f2ece1` | `#1e2b44` |
| Sunken surface | Dune / Keel | `#e7dfd0` | `#283753` |
| Primary ink | Harbor Navy / Bone | `#172a45` | `#f1e9dc` |
| Secondary ink | Slate / Driftwood | `#3f4e6e` | `#c3bbac` |
| Muted ink | Fog / Ash | `#6f7a8c` | `#8b91a0` |
| Accent | Leather / Brass | `#8a5a2b` | `#d2a06f` |
| Protein | Rust / Coral | `#b8302f` | `#d9455e` |
| Carbs | Ochre / Amber | `#a37a00` | `#bd8b1f` |
| Fat | Cobalt / Periwinkle | `#4257d9` | `#7488ff` |
| Fibre | Pine / Sea Green | `#2f8a4e` | `#3aae6c` |
| Danger | — | `#c0392b` | `#ff6b5f` |
| Good | — | `#2e8b57` | `#4cc98a` |
| Warning | — | `#b7791f` | `#f0b73a` |

Type is Manrope throughout. Radius 12px, 18px for cards.

The four macro colours were validated with the dataviz palette checker against both
surfaces: colour-blind separation, a normal-vision distinctness floor, and 3:1 contrast.
**Do not eyeball replacements** — re-run the validator if you change one. The P / C / F
letters always sit beside the numbers so identity never depends on colour alone.

## Things that will bite you

- **The manifest and icons must be real files at real URLs.** Generating them at runtime
  as a `blob:` URL or `data:` URI makes Chrome fall back to a badged bookmark shortcut
  instead of installing the app.
- **All `ortizzle.github.io` apps share one localStorage origin.** That is how the
  Howlers workout hook reads its Gist keys without any plumbing. It also means two apps
  on that host share storage on a given device.
- **Every storage read and write goes through `Store`**, which namespaces with `fuel_`.
  That single choke point is what would make per-person profiles cheap.
- **Dates go through `AZ`.** Never `toISOString().split('T')[0]` — it drifts to UTC.
- **EXIF is read before the resize, never after.** `shrinkImage` redraws through a canvas,
  which drops all metadata, so `photoTimestamp` must see the original `File`. EXIF
  `DateTimeOriginal` is wall-clock at capture with no timezone, so use it as-is rather than
  passing it through `AZ`.
- **Scans default to Packaged** and deliberately do not write `last_context`, so scanning a
  bar never changes what your next hand-added meal defaults to.
- **`serving.label` is free text; `serving.grams` is the number that matters.** Editing the
  gram weight rescales `per` proportionally and rescales `servings` inversely, so correcting
  a wrong serving size never changes the day's total. Open Food Facts entries with no serving
  size on file come back flagged `serving.unknown` with per-100g figures.
- **Never hand a conditional child to a native `append()`** — it renders the literal text
  "null". Use `addKids(parent, ...)` or the `el()` helper, both of which filter.
- **`updateEntry` mutates the record in place.** Capture any before/after comparison *before*
  calling it, or you will compare a value against itself.
- **`.brand` is the app wordmark** (22px, weight 800, scoped to `header.top`). A log entry's
  brand uses `.ebrand` — the two shared a class name and every brand in the log rendered at
  wordmark size. Check for collisions before reusing a short class name.
- **Never write into a field that has focus.** Servings and "grams eaten" drive each other, and
  normalising the box mid-keystroke swallows a half-typed decimal: "20." becomes "20", so the
  next key lands as "205" and 20.5 g logs as 205.1 g. Linked fields update each other, never
  themselves — use `setNum`, which no-ops on `document.activeElement`, and tidy the typed box
  on `blur` only when it no longer matches the portion.
- **The stepper keeps three decimals, `fmtServ` shows two.** A portion typed in grams rarely
  lands on a round fraction of a serving (20.5 g of a 15 g serving is 1.367), and rounding the
  stored value to 1.37 would read back as 20.6 g. Grams are the honest number; the servings
  count is a label.
- **Recents are derived, never stored** — `recentFoods()` folds the log by barcode, or by name
  plus brand when there is none. Saved meals are a second record type, `savedmeal`, so they
  inherit backup, restore-merge and tombstones. Everything that reads the log filters on
  `type === 'entry'`, so new record types are additive.
- **Editing an entry re-remembers the product** behind its barcode, otherwise the next scan
  hands back the numbers you just corrected.
- **Restore merges, never overwrites.** Newest `updatedAt` wins and tombstones are
  respected, so an old backup cannot delete newer entries or resurrect deleted ones.
- **Claude summaries are records, not DOM.** `render()` rebuilds the whole log view on every
  date change and every edit, so anything that lives only in a local div is gone the moment
  you step to another day. Day summaries save as `sum_day_<date>` and range analyses as
  `sum_range_<7|14|30>` — deterministic ids, so re-running overwrites instead of piling up.
  Each carries a `fingerprint` of what was actually summarized (entry count, rounded kcal,
  newest `updatedAt`, workout count); when it no longer matches, the card is marked stale
  rather than hidden. Summaries untouched for `SUMMARY_KEEP_DAYS` are pruned on the next save
  — by when they were *written*, not the day they cover, so summarizing an old day still keeps it.
- **Workouts are records too** (`type:'workout'`), so they ride along in backups, restore-merge and
  tombstones without any extra code. `dayMovement(date)` is the one place that decides what a day's
  movement is: native records if there are any, else The Howlers' read-only cache — never both, so
  importing Howlers history cannot double-count. Everything that reads the log still filters on
  `type === 'entry'`, so entries and workouts never leak into each other's totals.
- **Energy out is blue (`--move`), energy in is brass.** The inner ring is *minutes* against
  `targets.move`, not calories — minutes are measured, calories are guessed. `workoutKcal()` returns
  the source's figure when there is one, a MET estimate flagged `estimated` only when a weight is on
  file, and `null` otherwise; the UI shows minutes rather than inventing a number.
- **Peloton ids come from the row's timestamp** (`wk_pel_<date>_<hhmm>_<kind>`), so re-importing the
  same export adds nothing and a ride the user deleted stays a tombstone. Columns are matched by
  header name, not position. The parser was written against the documented export format; confirm it
  against a real file the first time.
- **`estimateTargets()` is pure and deterministic** — Mifflin-St Jeor, an activity factor, a deficit,
  then protein → fat → carbs → fiber in that order, each with a floor or cap. Keep it that way: the
  sheet shows its reasoning line by line, and a test pins the arithmetic to hand-computed values
  (220 lb, 5'10", 45, male, sedentary → 2,267 maintenance, 1,767 steady). A target below the floor
  (1,500 men / 1,200 women) is flagged, never blocked. `profileContext()` folds maintenance and the
  current target in, so every prompt knows whether the user is cutting and by how much.
- **The training profile is context, not a plan.** `profileContext()` is injected into every AI
  prompt. The old hardcoded "training for a half marathon" line is gone — do not put personal
  assumptions in prompt text; they belong in Settings where the user can change them.
- **`.field label` is `display:block` and ties with any `.x label` rule.** A label that needs to be
  flex (the equipment checkboxes) must outrank it — `.checks label.check` — and a checkbox inside a
  sheet needs the global input rule's `min-height: 44px` and padding switched off explicitly.
- **`seedSampleDays` only logs meals that have already happened**, so before the first sample
  meal of the morning today would be blank. It backfills one coffee at the current time —
  without that the preview build looks broken at 6am and day counts drift by one.

## Testing

Harness lives outside the repo (Playwright against headless Chromium at 390×844):
an app suite, install checks, and backup round-trip checks. Serve locally with
`python3 -m http.server 4400` and open `http://localhost:4400/`.
