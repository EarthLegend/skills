# skills

LGND agent skills and plugins for geospatial search over satellite and aerial imagery. This repository is a Claude plugin marketplace named `lgnd`.

## Plugins

| Plugin | Skills | What it does |
|---|---|---|
| [`lgnd-geo`](plugins/lgnd-geo) | `map-in-area` | Connects Claude to the LGND Geo server, which searches, views, classifies and checks places from imagery embeddings. `map-in-area` finds every place of one kind in an area (solar farms, wind turbines, lakes…) and says where they are and roughly how many. |

Each plugin's README covers what it needs, what it costs and its limits. Release notes are in [CHANGELOG.md](CHANGELOG.md).

## Install

**Claude Code:**

```
claude plugin marketplace add EarthLegend/skills
claude plugin install lgnd-geo@lgnd
```

To update later, run `claude plugin marketplace update lgnd`, then `claude plugin update lgnd-geo@lgnd`.

**claude.ai and Claude Desktop:** go to Customize → Plugins → Add marketplace, add `EarthLegend/skills`, then install the plugin you want.

**Other agents:** each plugin's `.mcp.json` names its MCP server, and its `skills/*/SKILL.md` files are plain instructions. Add the server to your agent's MCP configuration and give the agent the skill.

## Layout

```
.claude-plugin/marketplace.json   the marketplace: one entry per plugin
plugins/<plugin>/                 one installable plugin: its manifest, MCP server, README and skills
CHANGELOG.md                      one entry per release
```

`plugins/`, `marketplace.json` and the changelog entries are published from LGND's source repository. Each release replaces them, so edits made to them here would be lost.

To report a problem, open an issue with the plugin and its version, the question you asked, and what came back.

## License

[MIT](LICENSE)
