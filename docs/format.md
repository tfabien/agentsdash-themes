# Theme format

Reference for authors of themes and icon packs for [agentsdash](https://github.com/tfabien/agentsdash). It describes what the hub accepts. The marketplace of this repository is stricter: [create-theme.md](create-theme.md) lists the extra rules of `tools/check.mjs` and the art guidelines, and [install.md](../install.md) covers installation.

Validation code: [`hub/web/theme_schema.js`](https://github.com/tfabien/agentsdash/blob/main/hub/web/theme_schema.js) (themes, looks, palettes) and [`hub/themes_loader.mjs`](https://github.com/tfabien/agentsdash/blob/main/hub/themes_loader.mjs) (files) in agentsdash.

## Two layouts

The same theme exists in two layouts. Only the location of the icons and the `iconPack` field differ.

| | Marketplace (this repository) | Personal theme (your machine) |
|---|---|---|
| Theme file | `<name>/theme.json` | `~/.agentsdash/themes/<name>.json` |
| Icons | `<name>/icons/` | `~/.agentsdash/icons/<pack>/` |
| `iconPack` | forbidden: the pack is the theme's own `icons/` folder | required, names the folder under `icons/` (missing: `robot`) |
| Preview | `<name>/preview.(png\|webp\|jpg\|gif)`, optional | none |

The marketplace install converts the first layout into the second: it writes `themes/<name>.json` with `iconPack` set to `<name>` and copies the icons to `icons/<name>/`. Press **Reload** in the theme dialog, or `curl -X POST http://127.0.0.1:8765/reload`, after editing a personal theme.

## Theme file

```json
{
  "name": "harbor",
  "label": "Harbor",
  "description": "Pixel-art dock workers with role-specific tools.",
  "keywords": ["harbor", "pixel", "sea"],
  "author": "me",
  "version": "1.0.0",
  "accent": "#f2cc60",
  "colors": { "SCREEN": "#0b1a24", "PANEL": "#0f2230", "TEXT": "#f4e9d0" }
}
```

- `name`: 1 to 32 characters among `a-z`, `0-9`, `-`. It becomes a file and folder name.
- `label`: the title on the theme card, cut to 32 characters. Defaults to `name`.
- `description`: one sentence, cut to 200 characters. The marketplace shows and searches it.
- `keywords`: a list of words, at most 8, each cut to 24 characters. The marketplace shows them as tags and searches them.
- `author`, `version`: text. The marketplace updates an installed theme when `version` grows, so raise it on every change.
- `colors`: required object of `KEY: "#rrggbb"`, keys in the table below. A missing or invalid key keeps the built-in color.
- `accent`: optional `#rrggbb`. It tints the icons of packs that declare a tint mode in `pack.json`.
- `variants`, `looks`, `defaultLooks`: optional, see below.
- A file over 256 KB, invalid JSON, a bad `name` or a missing `colors` object is skipped. The hub lists the reason in `errors` of the catalog.

### Color keys

All values are `#rrggbb`. Default values come from `hub/web/screen.js`.

| Key | Where it shows |
|---|---|
| `SCREEN` | Page background. Text drawn on `ACCENT` buttons uses it too. |
| `BAR` | Bottom bar background. |
| `PANEL` | Background of session panels, plan and question areas. |
| `ROW` | Background of an agent row in the team list. |
| `FRAME` | Session title band, selected row, panel borders. |
| `CHIP` | Empty progress rings and plan steps, buttons, chips. |
| `TEXT` | Main text. Must reach a contrast of 4.5 against `PANEL` and `SCREEN`. |
| `LIGHT` | Secondary text (plan steps not started, details). |
| `FAINT` | Dim text: hints, counters, "+3 more". |
| `ACCENT` | Section labels (PLAN, BEFORE), the running plan step, selected answers. |
| `ACCENT_DIM` | Low end of the pulse of a running plan step (pulses to `ACCENT`). |
| `WORKING` | State "working", finished plan steps, usage under 60%. |
| `NEEDS` | State "needs you", stale plan, usage from 60%. |
| `NEEDS_DIM` | Low end of the pulse of the "needs you" bar of a row. |
| `ERROR` | State "error", usage from 80%. |
| `YOURTURN` | State "your turn" (the session waits for a prompt). |
| `WAITING` | State "waiting for sub-agents". |

Keep the five state colors (`WORKING NEEDS ERROR YOURTURN WAITING`) distinct from each other and readable on `ROW` and `PANEL`. A light variant usually changes `SCREEN BAR PANEL ROW FRAME CHIP TEXT LIGHT FAINT` and keeps the state colors.

## Variants: interface palettes

```json
{ "variants": {
    "paper": { "label": "Paper", "colors": { "SCREEN": "#f6f8fa", "PANEL": "#ffffff", "TEXT": "#1f2328" }, "accent": "#3b6fd4" },
    "amber": { "label": "Amber", "colors": {}, "accent": "#f2cc60" } } }
```

- `variants`: up to 12 entries, keys `a-z0-9-`. Each has an optional `label` (32 characters), a partial `colors` object merged over the theme colors, and an optional `accent` (falls back to the theme accent). An invalid variant, color or accent is dropped.
- The theme card shows one swatch per variant (hub setting `themeVariant`). Only one variant is active at a time.
- Order of application: theme colors, variant colors, then the user's edits from the color editor.

## Icon pack

A pack is a folder of pictures, one or more per role.

```
icons/lead.png     lead-blink.png   lead-2.png   lead-2-blink.png
      general.png  explore.png      plan.png     review.png
      test.png     code.png         generic.png  pack.json
```

File name: `<role>[-N][-blink].<ext>`.

- `role`: `lead general explore plan review test code generic`. Other file names are ignored by the hub.
- `-N`: optional, 2 to 99. Numbered files are alternate characters for the role (see "Which picture an agent gets").
- `-blink`: optional frame shown while the agent blinks. Without it, the plain picture stays.
- `ext`: `png`, `webp`, `gif`, `jpg` or `svg`. When one role has several formats for the same number, the order is png, webp, gif, jpg, svg.
- Sizes: raster files up to 1 MB, SVG up to 256 KB. A larger file is skipped. A pack with no valid file is not listed.
- A missing role falls back to the `robot` pack for that role.

### Drawing box

- Every picture is drawn in a 36x42 box (width x height), scaled by the page. Draw rasters at 2x or 3x: 72x84 or 108x126 pixels.
- SVG files use `viewBox="0 0 36 42"`.
- The page draws the box at three sizes, in units of the 480x272 screen: 36x42 in the lead and detail panels, 24x28 in team rows, 16x19 in compact team rows. The silhouette must stay readable at 16x19.
- While an agent works, the page squeezes the picture vertically by up to 4.5% for a breathing effect and shows the `-blink` frame on one redraw out of 15.
- An animated GIF or WebP plays where the browser has `ImageDecoder` (Chrome, Edge). Elsewhere the first frame shows. Frame delays under 20 ms count as 20 ms.

### Which picture an agent gets

The role file follows what the agent does now, then its type (`hub/web/screen.js`, `activityIndex` and `roleIndex`):

| Role file | Shown when |
|---|---|
| `explore` | The current tool is Read, Grep, Glob, LS, WebFetch or WebSearch, or the agent type contains "explor". |
| `code` | The current tool is Edit, MultiEdit, Write or NotebookEdit, or the agent type contains "code" or "dev". |
| `plan` | The current tool is TodoWrite, EnterPlanMode or ExitPlanMode, or the agent type contains "plan". |
| `test` | The current tool is Bash running a test command (test, jest, vitest, pytest, mocha, rspec, phpunit), or the agent type contains "test". |
| `lead` | The main session agent. |
| `general` | Agent type starting with "general" (general-purpose). |
| `review` | Agent type containing "review". |
| `generic` | Any other agent type. |

Among the numbered files of that role (`explore`, `explore-2`, `explore-3`...), a hash of the agent id picks one, so one agent keeps the same character and two explorers of a session usually differ. The lead picks by the session id. Looks narrow this choice.

## Looks: several characters, ticked by the user

A variant changes the whole page and only one is active. Looks change how agents are drawn, and several can be active: each agent takes one of the active looks by a hash of its id.

```json
{ "looks": {
    "classic": { "label": "Classic" },
    "ember":   { "label": "Ember", "accent": "#f2a33a" },
    "night":   { "label": "Night", "sprite": 2, "palette": "hair-black" } },
  "defaultLooks": ["classic", "ember"] }
```

- `looks`: up to 12 entries, keys `a-z0-9-`. `label` is the chip text (32 characters).
- `sprite`: optional integer 1 to 99. `1` is the plain file (`lead.png`), `2` is `lead-2.png`. A role without that number draws its plain file.
- `accent`: optional `#rrggbb`. It tints the agent through the pack's tint mode. A pack with `tint: none` ignores it. A look without `accent` uses the theme or variant accent.
- `palette`: optional name of a palette of the pack's `pack.json`. A look without it draws the original colors.
- A look may combine `sprite`, `palette` and `accent`.
- `defaultLooks`: the looks active until the user chooses. Missing or empty: all of them.
- The theme card shows one chip per look when the theme has more than one. The choice is the hub setting `themeLooks`. **Save as theme** keeps the active looks.
- A theme without `looks` lets the hash pick among all the numbered files of each role.

## `pack.json`: tint and palettes

Optional file `icons/pack.json`, 4 KB at most, rendering metadata only.

```json
{ "tint": "svg" }
{ "tint": "hue", "hue": 30 }
{ "tint": "none", "palettes": { "hair-blond": { "#3b241c": "#6b4b12", "#6e432b": "#b27a22" } } }
```

### Tint modes

- `svg`: SVG files take the accent wherever they draw with `currentColor` (strokes, fills). Use it for line and minimal packs.
- `hue`: every picture gets a hue rotation from `hue` (the base hue of the artwork, integer 0 to 359) to the hue of the accent. It changes every color, so it suits single-color artwork. It needs canvas `filter` (Chrome, Edge).
- `none`, or no `pack.json`: the accent is ignored. Use it for multi-color artwork.

### Palette swaps

A look can recolor part of a raster picture (for example only the hair) without a second sprite.

- `palettes`: up to 12 names (`a-z0-9-`). Each palette maps 1 to 32 source colors to target colors, `"#rrggbb": "#rrggbb"`. A bad palette is dropped and listed in the catalog errors.
- Matching is exact: a pixel changes only when its RGB equals a source color. Alpha is kept. The swap runs at the image's own resolution.
- Order for one agent: the look picks the numbered sprite, the palette applies to it, then the `hue` tint.
- Use PNG or lossless WebP, and paint the recolored areas in flat colors without antialiasing. JPEG and lossy WebP shift colors by a few units, so those pixels never match.
- SVG files ignore palettes (use `currentColor` with `tint: svg`). Animated GIF and WebP are drawn unmodified.
- An uninstalled marketplace card may preview the base artwork. Installed themes draw every palette look.

| Tool | Changes |
|---|---|
| `hue` tint | Every color of the picture. |
| `currentColor` + `tint: svg` | The marked parts of vector files. |
| `palette` | The listed colors only. |
| numbered `sprite` | The whole drawing: a different character. |

## Color editor and saved themes

The **Edit colors** button of the active theme card opens an editor: one input per color key, one for the accent, a live preview and a warning when `TEXT` contrast against `PANEL` or `SCREEN` is under 4.5. Edits are stored in the hub setting `themeOverrides` (key `<theme>` or `<theme>/<variant>`). **Reset edits** drops them.

**Save as theme** writes the current look (theme, variant and edits, flattened, without variants) to `~/.agentsdash/themes/<name>.json` through `POST /themes` (`{ name, label?, theme, variant?, override?, overwrite? }`). A built-in name and a name installed from the marketplace are refused. An existing user theme needs `overwrite: true` (the route answers 409 otherwise).
