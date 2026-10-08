# lgnd-geo

The LGND Geo MCP server and the skills that use it, installed together. The server searches, views, classifies and checks places on Earth from imagery embeddings. It covers two indexes:

- `naip`: US aerial imagery from 2020 to 2024. Its places are about 150 m across.
- `s2`: Sentinel-2 over the world's land, monthly since 2017. Its places are about 1.2 km across.

## Install

In Claude Code:

```
claude plugin marketplace add EarthLegend/skills
claude plugin install lgnd-geo@lgnd
```

In claude.ai or Claude Desktop, go to Customize → Plugins → Add marketplace, add `EarthLegend/skills`, then install `lgnd-geo`.

The first Geo tool call opens a sign-in to your LGND account. Free accounts have a daily limit on tool calls.

## Skills

### find-in-area

Finds every place of one kind in an area, such as solar farms, wind turbines, lakes or quarries, and answers where they are and roughly how many there are. A search returns only the best few matches. This skill instead builds a classifier from checked examples, labels every place in the area, checks a sample of the result, and groups the labelled places into sites.

You can ask plainly, or call the skill by name. Name a place and roughly how big an area to cover:

```
/lgnd-geo:find-in-area How many wind turbines are there in an area about 10 km across,
around Ellsworth, Illinois? Where are they?
```

Other questions it handles:

- "Find every solar farm in an area about 15 km across, around Woodbridge and Linden, New Jersey."
- "Which lakes within about 20 km of Boulder Junction, Wisconsin, are bigger than 3 km²?"
- "Map the ground burned by the 2024 Line Fire in the San Bernardino Mountains, California. About how much burned?"

Coordinates or a box make the area exact, but you don't need them. A vague area can come out much larger, and larger areas take longer.

What to expect:

- **The answer:** each site with its position and size, a count (a range when it's uncertain), and what was checked or may have been missed.
- **The cost:** in our tests, areas of about 90–300 km² took 9–18 minutes and 34–68 tool calls.
- **A checked count, on request:** when the count is uncertain, the answer offers one, with its cost in calls. It makes those calls only if you ask.
- **Narrow targets:** things much narrower than a place, such as grass airstrips or small pond dams, are mostly missed. The answer says so.

## Notes

- Building a classifier can take a few minutes, so the plugin sets a 6-minute tool timeout in `.mcp.json`.
- To use it with other agents, point your MCP client at `https://geo.lgnd.ai/mcp`, which signs in with OAuth, and give the agent `skills/find-in-area/SKILL.md`.
