# EVE-Cyno plugins for Claude Code

A Claude Code plugin marketplace with one plugin, `eve-cyno`: the public EVE Online data tools of [EVE-Cyno](https://eve-cyno.dev) in one install. It adds a remote MCP server, an `eve-data` skill that teaches the model which tool to use when, and three slash commands.

## Install

```
/plugin marketplace add eve-cyno/claude-plugins
/plugin install eve-cyno@eve-cyno
```

Or, MCP server only, without the skill and commands:

```
claude mcp add --transport http eve-cyno https://data.eve-cyno.dev/v1/mcp
```

## What you get

MCP server `eve-cyno` (`https://data.eve-cyno.dev/v1/mcp`, streamable HTTP, stateless):

- Items and ships: `search_item_by_name`, `get_ship_stats`, `get_ship_bonuses`, `get_hull_facts`, `find_canonical_module`
- Fits: `validate_fitting`, `compute_fit_stats` (DPS, EHP, speed, align, capacitor from the dogma engine)
- Skills and industry: `get_required_skills`, `get_production_chain`, `get_reprocessing_yield`
- Space: `get_jumps_between`, `get_systems_in_region`, `get_npc_stations`
- Live data from EVE's public ESI (shared cache): `get_market_price` (Jita best sell/buy and order counts), `get_type_info`, `get_sovereignty`, `get_system_activity` (recent kills and jumps)
- `appraise_items` needs your own Janice key: at install Claude Code asks for an optional **Janice API key** (stored as a sensitive value; leave it empty if you don't have one, and add it later in the plugin settings under `/plugin`). It is sent only to `data.eve-cyno.dev`, which forwards it to Janice for that one request and never stores or logs it

Skill `eve-data`: tool selection, recommended flows (validate then compute), limits, what is not available, and the attribution rules.

Commands:

- `/eve-fit <EFT block>`: validate a fit and compute its stats
- `/eve-item <name>`: look up an item or ship
- `/eve-fits <hull> [activity]`: community fits for a hull, with author credit (uses the public REST endpoint)

## Privacy and limits

No EVE-Cyno API key and no account are needed (the Janice key above is optional and only for appraisals). Requests go to `data.eve-cyno.dev` and are rate limited per client IP (60 MCP requests per minute; the live ESI tools also 10 calls per minute). What you send to a tool, such as an EFT block or a system name, is processed by that service. See the [terms](https://data.eve-cyno.dev/terms). Docs: <https://data.eve-cyno.dev/docs/>, [`llms.txt`](https://data.eve-cyno.dev/llms.txt), [OpenAPI](https://data.eve-cyno.dev/v1/openapi.yaml).

## Support

Questions, feedback and bug reports: the [EVE-Cyno Discord](https://discord.gg/XZzkH38dt9).

## Notices

© 2014 Fenris Creations hf. All rights reserved. "EVE", "EVE Online", "Fenris Creations", and all related logos and images are trademarks or registered trademarks of Fenris Creations hf. This material is used with limited permission of Fenris Creations hf. (formerly CCP hf., rebranded May 2026). No official affiliation or endorsement by Fenris Creations is stated or implied. Created under the EVE Developer License Agreement.

Some reference text served by the tools is taken or condensed from the [EVE University Wiki](https://wiki.eveuniversity.org/), © EVE University and its wiki contributors, licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Community fits belong to their authors and sites and carry their own attribution.

## Licence

MIT, Copyright (c) 2026 Bythlak. See [LICENSE](LICENSE).
