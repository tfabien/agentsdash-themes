---
name: create-agentsdash-theme
description: Use when asked to create, generate, design or fix a theme or icon pack for agentsdash (the agent dashboard, formerly standup-bot), or a folder of the agentsdash-themes marketplace repository, including theme.json colors, variants, looks, role sprites (lead, general, explore, plan, review, test, code, generic) and pack.json tint or palette swaps.
---

# Create an agentsdash theme

An agentsdash theme is one folder: a `theme.json` (interface colors) and an `icons/` folder (one character picture per agent role). This file holds the whole specification, so it works without the repository. When the repository is available, `docs/format.md` and `docs/create-theme.md` are the long references and `node tools/check.mjs` is the validator.

## Procedure

1. Settle the brief with the user: subject (pirates, cats, robots...), art style (pixel art, line SVG, flat vector, soft 3D), mood (dark or light page), and whether they want alternate characters per role. Pick defaults if they do not care: pixel art, dark page, one character per role.
2. Choose the rendering mode from the table in "pack.json". It decides the file format of the icons.
3. Write `theme.json` (section "theme.json"). Compute the contrast of `TEXT` against `PANEL` and `SCREEN` for the base and for each variant; every ratio must be 4.5 or more.
4. Produce the 8 role pictures (section "Icons"). Generate or draw one file per role. Remove the background, crop and resize each to the canvas size, and save it under its exact file name.
5. Add optional files: `icons/pack.json`, blink frames, numbered alternates with one look each, `preview.png`.
6. Validate with the checklist at the end. In the repository, run `node tools/check.mjs` and fix every `FAIL` line.
7. Deliver the folder (or a zip of it) and tell the user how to install it (section "Install and test").

## Folder

```text
<name>/
  theme.json                    required
  preview.png                   optional: png, webp, jpg or gif, 1 MB max
  icons/
    pack.json                   optional, 4 KB max
    <role>.<ext>                required, one per role
    <role>-<N>.<ext>            optional alternate character, N = 2..99
    <role>[-<N>]-blink.<ext>    optional blink frame
```

- `<name>`: 1 to 32 characters among `a-z 0-9 -`, not `line`.
- `<role>`: `lead general explore plan review test code generic`. All 8 are required.
- `<ext>`: `png webp gif jpg svg`.
- Nothing else in the folder. No other sub-folder than `icons/`, no other file in `icons/` than role files and `pack.json`.
- At most 100 files and 20 MB per theme. Raster icons 1 MB each, SVG icons and `theme.json` 256 KB each.

## theme.json

```json
{
  "name": "harbor",
  "label": "Harbor",
  "description": "Pixel-art dock workers with role-specific tools.",
  "keywords": ["harbor", "pixel", "sea"],
  "author": "Your Name",
  "version": "1.0.0",
  "colors": {
    "SCREEN": "#0b1a24", "BAR": "#10222e", "PANEL": "#0f2230", "ROW": "#173041",
    "FRAME": "#7a5230", "CHIP": "#2c4a5e",
    "TEXT": "#f4e9d0", "LIGHT": "#d5c3a0", "FAINT": "#8f9ea4",
    "WORKING": "#6fcf97", "NEEDS": "#f2a33a", "ERROR": "#e5574f", "YOURTURN": "#6fb4e8", "WAITING": "#79d0c8",
    "ACCENT": "#f2cc60", "ACCENT_DIM": "#5a4a22", "NEEDS_DIM": "#7a4e18"
  },
  "variants": {
    "sand": { "label": "Sand", "colors": {
      "SCREEN": "#f7f0df", "BAR": "#eadcc1", "PANEL": "#eadcc1", "ROW": "#dfcba7",
      "FRAME": "#9a7447", "CHIP": "#d3bd95", "TEXT": "#2e2118", "LIGHT": "#5e4937", "FAINT": "#75604d" } }
  }
}
```

| Field | Rule |
|---|---|
| `name` | Equal to the folder name. |
| `label` | Card title, non-empty, 32 characters max. |
| `description` | One sentence, 1 to 200 characters: the characters and the art style. |
| `keywords` | 3 to 8 lowercase words matching `^[a-z0-9-]{1,24}$`: subject plus style (`pixel`, `line`, `vector`, `3d`). |
| `author` | Non-empty text. |
| `version` | Dotted numbers with at least one dot. Start at `1.0.0`, raise it on every change. |
| `iconPack` | Never in a marketplace folder. |
| `colors` | Required. Known keys only, values `#rrggbb`. `TEXT`, `PANEL`, `SCREEN` required. |
| `accent` | Optional `#rrggbb`, tints icons when `pack.json` has `tint` `svg` or `hue`. |
| `variants` | Optional, up to 12, keys `a-z0-9-`, each `{label, colors (partial), accent}`. |
| `looks` | Optional, up to 12, keys `a-z0-9-`, each `{label, sprite 1..99, accent, palette}`. |
| `defaultLooks` | Optional list of keys of `looks`, active until the user picks. |

### Color keys

| Key | Use |
|---|---|
| `SCREEN` | Page background; text color on `ACCENT` buttons. |
| `BAR` | Bottom bar. |
| `PANEL` | Session, plan and question panels. |
| `ROW` | Agent rows of the team list. |
| `FRAME` | Session title band, selected row, borders. |
| `CHIP` | Empty rings and steps, buttons, chips. |
| `TEXT` | Main text. |
| `LIGHT` | Secondary text. |
| `FAINT` | Hints and counters. |
| `ACCENT` | Section labels, running plan step, selected answers. |
| `ACCENT_DIM` | Darker `ACCENT`, low point of a pulse. |
| `WORKING` | Green family: agent working, steps done. |
| `NEEDS` | Orange family: agent needs the user. |
| `NEEDS_DIM` | Darker `NEEDS`, low point of a pulse. |
| `ERROR` | Red family: error, usage over 80%. |
| `YOURTURN` | Blue family: session waits for a prompt. |
| `WAITING` | Violet or cyan family: waiting for sub-agents. |

Color rules:

- Set all 17 keys. Missing keys fall back to built-in dark blue colors.
- Contrast ratio (WCAG relative luminance) of `TEXT` on `PANEL` and on `SCREEN`: 4.5 or more, for the base and for each variant merged over the base.
- Keep `SCREEN` text readable on `ACCENT`.
- Keep the 5 state colors in their hue family and distinct from each other.
- A light variant changes `SCREEN BAR PANEL ROW FRAME CHIP TEXT LIGHT FAINT`. It also changes `ACCENT` and `ACCENT_DIM` when `SCREEN` text on `ACCENT` drops under 4.5, and darkens the state colors that become hard to read on `ROW` and `PANEL`.
- The `accent` field and the `ACCENT` color key are independent. `accent` only tints icons (packs with `tint` `svg` or `hue`). `ACCENT` only colors the interface. Set both in the same hue for a coherent look.
- An accent variant recolors the icons only: `{"label": "...", "colors": {}, "accent": "#rrggbb"}`. Add `"colors": {"ACCENT": "...", "ACCENT_DIM": "..."}` to recolor the interface labels too.

## pack.json

| Artwork | `icons/pack.json` | Theme side |
|---|---|---|
| Multi-color raster (most pixel art) | none | nothing |
| SVG line or flat art | `{"tint": "svg"}`; the SVGs draw tintable parts with `currentColor` | `accent`, accent variants or looks |
| Single-color raster | `{"tint": "hue", "hue": <base hue, integer 0-359>}` | `accent` |
| Raster with recolorable areas | `{"tint": "none", "palettes": {"<name>": {"#rrggbb": "#rrggbb"}}}` | looks with `"palette": "<name>"` |

- `tint` is required when the file exists.
- Palettes: up to 12, each 1 to 32 exact color pairs. Matching is exact RGB, so the source colors must be flat, unantialiased and used nowhere else in the picture. PNG or lossless WebP only.
- Every `palette` of a look must exist in `pack.json`.

## Icons

### Role meanings

The page picks the role picture from the agent's current activity, then from its type. Draw the same character doing the activity, with one prop that reads at a glance.

| File | Shown when | Prop ideas |
|---|---|---|
| `lead` | Main session agent | Crown, captain hat, badge |
| `general` | General-purpose sub-agent | Neutral pose, satchel |
| `explore` | Reading or searching files and the web | Magnifier, map, lantern |
| `plan` | Writing a plan or todo list | Clipboard, scroll, blueprint |
| `review` | Review sub-agent | Glasses, checklist, quill |
| `test` | Running tests | Flask, test tube, target |
| `code` | Editing files | Laptop, keyboard, hammer |
| `generic` | Any other agent | Plainest version of the character |

### Picture rules

- Canvas: 108x126 pixels (3x the 36x42 box) for PNG and WebP. SVG uses `viewBox="0 0 36 42"` and may set `width="108" height="126"`.
- The page draws the box at 36x42, 24x28 and 16x19 screen units. Bold silhouette, few details, readable at 16x19.
- Transparent background. No scenery, frame, ground shadow, text or letters.
- One character per file, centered horizontally, feet near the bottom, about 2 px margin.
- Same style, palette, scale and light direction for all 8 roles.
- PNG or lossless WebP for raster art, SVG for line art. No JPG (no transparency).
- When an image generator returns a sheet with several characters, split it into one file per role, then remove the background, trim, pad to the 108x126 ratio and resize with nearest-neighbor for pixel art. Clean the edges at final size; crops from large sheets keep halos and broken outlines.

Image generation prompt, one per role:

```text
A single <subject> character, <role prop and action>, <art style>, full body, centered,
facing the viewer, bold readable silhouette for a 36x42 pixel icon, limited palette
<palette hex colors>, transparent background, no text, no scenery, no shadow.
```

### Alternates, blinks, looks

- `<role>-2`, `<role>-3`... are different characters (another species, outfit or team), not recolors. Recolors go in `palettes` or `hue`.
- Keep number N the same team across the 8 roles, and add one look per team: `"looks": {"humans": {"label": "Humans", "sprite": 1}, "elves": {"label": "Elves", "sprite": 2}}`.
- Without `defaultLooks`, every look is active. Set it only to hide some looks by default.
- `<role>-blink` is the identical picture with closed eyes. Any other change makes the sprite jump. A numbered file blinks with `<role>-<N>-blink`; without it, that character does not blink.

### SVG example (`tint: svg`)

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 36 42" width="108" height="126" color="#898cfa">
  <g fill="none" stroke="currentColor" stroke-width="1.65" stroke-linecap="round" stroke-linejoin="round">
    <path d="m11 12-1.2-7 4.3 3 3.9-5 3.9 5 4.3-3-1.2 7"/>
    <rect x="4.5" y="12.4" width="27" height="19.7" rx="7.5"/>
    <rect x="11.4" y="33.4" width="13.2" height="6.8" rx="3.4"/>
    <g fill="currentColor" stroke="none"><circle cx="13.8" cy="22" r="1.95"/><circle cx="22.2" cy="22" r="1.95"/></g>
  </g>
</svg>
```

This is a `lead` robot: the crown is the role prop. The `color` attribute is the fallback; the page replaces `currentColor` with the accent.

## Install and test

- Marketplace: put the folder at the root of a clone of `tfabien/agentsdash-themes` (or of the user's own themes repository), run `node tools/check.mjs`, add a row to the README table and open a pull request. The agentsdash page installs it from the **Theme** dialog, **Marketplace** tab, once merged.
- Personal: copy `theme.json` to `~/.agentsdash/themes/<name>.json` and add `"iconPack": "<name>"`, copy `icons/` to `~/.agentsdash/icons/<name>/`, then press **Reload** in the theme dialog (or `curl -X POST http://127.0.0.1:8765/reload`).
- On the page, check each variant swatch, each look chip and the three sprite sizes.

## Checklist

- [ ] Folder name equals `name`, `a-z0-9-`, 32 characters max.
- [ ] `label`, `description` (200 max), `keywords` (3 to 8), `author`, `version` `1.0.0` present; no `iconPack`.
- [ ] All 17 color keys set as `#rrggbb`; `TEXT` contrast 4.5+ on `PANEL` and `SCREEN`, base and every variant.
- [ ] 8 plain role files present with exact names; only allowed files in the folder.
- [ ] Pictures transparent, 108x126 (or SVG `0 0 36 42`), one character each, readable at 16x19.
- [ ] Numbered files are different characters; each number has a look; `defaultLooks` keys exist.
- [ ] `pack.json`, if present, has `tint`; `tint: svg` comes with `currentColor` in the SVGs; every look palette exists.
- [ ] `node tools/check.mjs` prints `PASS` (when the repository is available).

## Common mistakes

| Mistake | Effect | Fix |
|---|---|---|
| `"version": "1"` | Checker fails | `"1.0.0"` |
| `iconPack` in a marketplace folder | Checker fails | Remove it |
| Only some color keys | Built-in dark blue leaks into a light theme | Set all 17 |
| `-2` files that are recolors | Looks show the same character twice | Draw a new character, or use a palette |
| Opaque or white background | A box around every agent | Remove the background, save PNG with alpha |
| `explore-1.png` | Invalid name | The first character is the plain `explore.png` |
| Extra files (`notes.txt`, `src/`) | Checker fails | Keep sources outside the theme folder |
| Antialiased hair with a palette | Edges keep the old color | Flat colors in swap areas |
