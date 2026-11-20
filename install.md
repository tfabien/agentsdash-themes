# Install and create themes

## Install from the Marketplace

Open the agentsdash page (http://127.0.0.1:8765/), choose the palette button of the bottom bar, then open the **Marketplace** tab. The default source is `tfabien/agentsdash-themes`.

The local API can also install a theme directly:

```bash
curl -X POST http://127.0.0.1:8765/market/install \
  -H 'Content-Type: application/json' \
  -d '{"source":"tfabien/agentsdash-themes","name":"fantasy-rpg"}'
```

Other routes: `GET /market` lists the themes, `POST /market/update` with `{}` updates every installed theme, `POST /market/remove` with `{"name":"<name>"}` removes one. Installed themes update on their own every 6 hours when their `version` grows.

## Install by hand

Copy `<name>/theme.json` to `~/.agentsdash/themes/<name>.json` and add `"iconPack": "<name>"`. Copy `<name>/icons/` to `~/.agentsdash/icons/<name>/`. Press **Reload** in the theme dialog, or run `curl -X POST http://127.0.0.1:8765/reload`.

## Create a theme

Follow [docs/create-theme.md](docs/create-theme.md). It specifies the folder, `theme.json`, the color keys and contrast rule, the 8 role pictures and their art rules, `pack.json`, the preview and the checks. [docs/format.md](docs/format.md) is the full reference of what the hub accepts.

In short:

- One folder `<name>/` with `theme.json`, `icons/` and an optional `preview.png`. Nothing else.
- `theme.json` has `name` (equal to the folder), `label`, `description` (1 to 200 characters), `keywords` (3 to 8), `author`, `version` (`1.0.0`, raised on every change) and `colors`. No `iconPack`.
- `TEXT` reaches a contrast of 4.5 against `PANEL` and `SCREEN`, in the base colors and in each variant.
- One transparent picture per role: `lead`, `general`, `explore`, `plan`, `review`, `test`, `code`, `generic`, at 108x126 (or SVG with `viewBox="0 0 36 42"`).
- `node tools/check.mjs` prints `PASS` before you open a pull request.

To generate a theme with Claude or ChatGPT, use the skill [`.claude/skills/create-agentsdash-theme/SKILL.md`](.claude/skills/create-agentsdash-theme/SKILL.md). Claude Code loads it on its own inside a clone of this repository. Other assistants take the file as instructions or as an attachment.
