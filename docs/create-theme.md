# Create a theme

Specification of a marketplace theme for this repository. A theme that follows it passes `node tools/check.mjs`, installs from the marketplace and reads well on the page. [format.md](format.md) is the full reference of what the hub accepts; this page adds the stricter rules of the checker and the art rules.

AI assistants can follow this page directly. The skill [`.claude/skills/create-agentsdash-theme/SKILL.md`](../.claude/skills/create-agentsdash-theme/SKILL.md) packs the same rules as a step-by-step procedure (see "Generate with an AI assistant" below).

## 1. Deliverable

One folder at the root of the repository:

```text
<name>/
  theme.json                              required
  preview.png                             optional (png, webp, jpg or gif), 1 MB max
  icons/
    pack.json                             optional, 4 KB max
    <role>.<ext>                          required for each of the 8 roles
    <role>-<N>.<ext>                      optional alternate characters, N = 2..99
    <role>[-<N>]-blink.<ext>              optional blink frames
```

- `<name>`: 1 to 32 characters among `a-z`, `0-9`, `-`. It must not be `line` (the built-in theme). Pick a name not used by another folder of this repository.
- Roles: `lead`, `general`, `explore`, `plan`, `review`, `test`, `code`, `generic`.
- `<ext>`: `png`, `webp`, `gif`, `jpg` or `svg`.
- Nothing else is allowed in the folder: no other file, no sub-folder besides `icons/`, no file in `icons/` besides role files and `pack.json`.
- Limits per theme: 100 files, 20 MB in total, raster icons 1 MB each, SVG icons and `theme.json` 256 KB each.

## 2. `theme.json`

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
    "ACCENT": "#f2cc60", "ACCENT_DIM": "#5a4a22"
  }
}
```

The checker requires, beyond [format.md](format.md):

| Field | Rule |
|---|---|
| `name` | Equal to the folder name. |
| `label` | Non-empty text. Keep it under 32 characters (the card cuts it). |
| `description` | One sentence, 1 to 200 characters. Say what the characters are and the art style. |
| `keywords` | 3 to 8 entries, each matching `^[a-z0-9-]{1,24}$`. Include the subject and the art style (`pixel`, `line`, `vector`, `3d`). |
| `author` | Non-empty text. |
| `version` | Dotted numbers with at least one dot: `1.0.0`. Start at `1.0.0`. Raise it on every change, because installed copies update only when it grows. |
| `iconPack` | Forbidden. The pack is the theme's own `icons/` folder. |
| `colors` | Object with known keys only (table in [format.md](format.md#color-keys)), values `#rrggbb`. `TEXT`, `PANEL` and `SCREEN` are required. |
| contrast | `TEXT` against `PANEL` and `TEXT` against `SCREEN` at least 4.5 (WCAG ratio). |
| `accent` | `#rrggbb` when present. |
| `variants` | Up to 12, keys `a-z0-9-`. Each variant merged over the base colors must pass the same contrast rule. |
| `looks` | Up to 12, keys `a-z0-9-`. `sprite` is an integer 1 to 99. `accent` is `#rrggbb`. `palette` must name a palette of `icons/pack.json`. |
| `defaultLooks` | Every key must exist in `looks`. |

Color advice:

- Set all 17 keys when the theme has a strong identity. Keys left out keep the built-in dark blue colors, which can clash with a light background.
- `ACCENT` is also a button background with `SCREEN` text on it: keep that pair readable.
- Keep the 5 state colors (`WORKING` green, `NEEDS` orange, `ERROR` red, `YOURTURN` blue, `WAITING` violet or cyan) recognizable. Shift their tone to the palette, keep their hue family.
- `ACCENT_DIM` and `NEEDS_DIM` are darker versions of `ACCENT` and `NEEDS`, used as the low point of a pulse.

- The `accent` field and the `ACCENT` color key are independent. `accent` tints icons of packs with `tint` `svg` or `hue`. `ACCENT` colors the interface. Use the same hue for both.

Offer 1 to 3 variants:

- A light or dark counterpart changes `SCREEN BAR PANEL ROW FRAME CHIP TEXT LIGHT FAINT`. It also changes `ACCENT` and `ACCENT_DIM` when `SCREEN` text on `ACCENT` drops under 4.5, and darkens state colors that become hard to read on `ROW`.
- For tinted packs, accent variants: `"colors": {}` plus `accent` recolors the icons; add `ACCENT` and `ACCENT_DIM` to `colors` to recolor the interface labels too.

## 3. `icons/pack.json`

Leave it out for multi-color artwork. Otherwise:

| Artwork | `pack.json` | Theme fields |
|---|---|---|
| SVG line or flat art | `{"tint": "svg"}`, and at least one SVG draws with `currentColor` | `accent`, accent-only variants or looks |
| Single-color raster art | `{"tint": "hue", "hue": <base hue 0-359>}` | `accent` |
| Multi-color raster art with recolorable areas | `{"tint": "none", "palettes": {...}}` | looks with `palette` |

`tint` is required when the file exists. `hue` is an integer from 0 to 359. Palettes: up to 12, each with 1 to 32 `"#rrggbb": "#rrggbb"` pairs.

## 4. Icons

### What each role shows

The page picks the role file from what the agent is doing now, then from its type. Draw each role as a character doing that activity, with one object that tells it apart at a glance:

| Role | Shown when | Suggested prop |
|---|---|---|
| `lead` | The main session agent. | Crown, captain hat, badge. |
| `general` | General-purpose sub-agent. | Neutral pose, satchel. |
| `explore` | Reading or searching (Read, Grep, Glob, WebFetch, WebSearch). | Magnifier, map, lantern, telescope. |
| `plan` | Planning (TodoWrite, plan mode). | Clipboard, scroll, blueprint. |
| `review` | Review sub-agent. | Glasses, checklist, quill. |
| `test` | Running tests. | Flask, test tube, target. |
| `code` | Editing files (Edit, Write). | Laptop, keyboard, hammer, wrench. |
| `generic` | Any other agent type. | Simplest version of the character. |

### Drawing rules

- Canvas: 108x126 pixels for rasters (3x the 36x42 box), or 72x84 (2x). SVG: `viewBox="0 0 36 42"`.
- Transparent background. No scenery, no frame, no shadow box, no text or letters.
- One character per file, centered, feet near the bottom edge, about 2 px of free margin.
- Readable at 16x19: bold silhouette, few details, strong outline or flat color areas. Check it by scaling the picture down.
- All 8 roles share one style, palette, scale and light direction. The character must stay recognizable from role to role.
- Formats: PNG or lossless WebP for raster art, SVG for line art. Avoid JPG: it has no transparency.
- Each picture is a separate file drawn for this size. Sprites cropped from a large concept sheet get blurry or broken edges: redraw or clean them at the final size.

### Alternate characters (`-2` to `-99`)

- Numbered files must be different characters (another species, outfit, team), not recolors of the plain file. A recolor belongs in a `palette` or a `hue` tint.
- Give the same number the same character across all 8 roles. Then add one look per number (`"sprite": 2`) so the user can turn each team on or off.
- A role without file `-N` falls back to its plain file for that look.

### Blink frames (optional)

- `<role>-blink.<ext>` (or `<role>-<N>-blink.<ext>`) is the same picture with closed eyes, same size and same position.
- Any other difference makes the character jump when it blinks.

### Palette swaps (optional)

- Paint each recolorable area (hair, cloth) with 1 to 4 exact flat colors, with no antialiasing inside or at the border of that area.
- Use these colors nowhere else in the picture, or they change too.
- Save as PNG or lossless WebP. List the exact colors as source keys in `pack.json`.

## 5. Preview

`preview.png`, at most 1 MB, shown on the marketplace card and in the README. Take a real screenshot of the page with the theme installed, using `hub/tools/page_screenshots.mjs` of the agentsdash repository:

```bash
node hub/tools/page_screenshots.mjs --themes <name>
```

A sheet of the 8 role icons on the theme's `SCREEN` color also works.

## 6. Check and submit

1. Run `node tools/check.mjs` at the root of this repository. It prints `PASS <n> marketplace themes` or one `FAIL <theme>: <reason>` line per problem.
2. Install the theme locally and look at it on the page: copy `<name>/theme.json` to `~/.agentsdash/themes/<name>.json` with `"iconPack": "<name>"` added, copy `<name>/icons/` to `~/.agentsdash/icons/<name>/`, then press **Reload** in the theme dialog.
3. Check every variant swatch, every look chip, the 3 sprite sizes, and text readability.
4. Add a row to the table of [README.md](../README.md) and open a pull request.

## Generate with an AI assistant

- Claude Code: the skill in `.claude/skills/create-agentsdash-theme/` loads on its own when you work in a clone of this repository. Ask "create an agentsdash theme about pirates". To use it elsewhere, copy the folder to `~/.claude/skills/`.
- Claude.ai: zip the `create-agentsdash-theme` folder and upload it under Settings, Capabilities, Skills.
- ChatGPT or another assistant: paste the content of `SKILL.md` as project instructions or as the first message, or attach the file. The skill is self-contained and needs no other file.

Image generators often return one picture with several characters, an opaque background or a size other than 108x126. The skill asks the assistant to generate or clean each role picture separately, remove the background and resize it before writing the files.
