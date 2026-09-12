# Fuel — working notes

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
- **Restore merges, never overwrites.** Newest `updatedAt` wins and tombstones are
  respected, so an old backup cannot delete newer entries or resurrect deleted ones.

## Testing

Harness lives outside the repo (Playwright against headless Chromium at 390×844):
an app suite, install checks, and backup round-trip checks. Serve locally with
`python3 -m http.server 4400` and open `http://localhost:4400/`.
