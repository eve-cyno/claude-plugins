# EVE-Cyno plugins for Claude Code

A Claude Code plugin marketplace with one plugin, `eve-cyno`: the public EVE Online data tools of [EVE-Cyno](https://eve-cyno.dev) in one install. It adds a remote MCP server, an `eve-data` skill that teaches the model which tool to use when, and three slash commands.

## Install

```
/plugin marketplace add eve-cyno/claude-plugins
/plugin install eve-cyno@eve-cyno
```

From a shell instead of a session: `claude plugin marketplace add eve-cyno/claude-plugins`, then `claude plugin install eve-cyno@eve-cyno` (pass keys with `--config api_key=…` or `--config janice_api_key=…`).

Both keys are optional; Claude Code asks for them at install and you can change them later under `/plugin`:

- **EVE-Cyno API key**: free; adds `analyze_battle` and free-text fit search and raises the per-minute limits. Get one with `/apikey create` in the [EVE-Cyno Discord](https://discord.gg/XZzkH38dt9) or at [ai.eve-cyno.dev](https://ai.eve-cyno.dev) (Settings, Data API keys). Sent only to `data.eve-cyno.dev`, in `X-API-Key`.
- **Janice API key**: your own, optional: `appraise_items` gives an ESI estimate without it and a Janice appraisal with it (below).

Or, MCP server only, without the skill and commands (add `--header "X-API-Key: ecd_…"` if you have a key):

```
claude mcp add --transport http --scope user eve-cyno https://data.eve-cyno.dev/v1/mcp
```

## Other clients

Every MCP client connects with the URL `https://data.eve-cyno.dev/v1/mcp` (streamable HTTP; add an `X-API-Key` header if you have a key). Setup per client: <https://data.eve-cyno.dev/docs/mcp>.

The same `eve-data` skill works in harnesses that read `~/.agents/skills` (Codex, Gemini CLI, Cursor, VS Code); it uses the MCP tools when the client has them and the REST tool API otherwise:

```
mkdir -p ~/.agents/skills/eve-data && curl -fsSL https://raw.githubusercontent.com/eve-cyno/claude-plugins/main/plugins/eve-cyno/skills/eve-data/SKILL.md -o ~/.agents/skills/eve-data/SKILL.md
```

Claude Code reads `~/.claude/skills/` the same way.

## What you get

MCP server `eve-cyno` (`https://data.eve-cyno.dev/v1/mcp`, streamable HTTP, stateless):

- Items and ships: `search_item_by_name`, `get_type_info` (name, description, volume, mass, group from the SDE, live ESI as fallback), `get_ship_stats`, `get_ship_bonuses`, `get_hull_facts`, `find_canonical_module` (by family, or by `slot` and `meta`); `search_item_by_name` with `details` classifies up to 50 names (tech level, faction, Alpha use, abyssal NPCs and filaments)
- Fits: `validate_fitting`, `compute_fit_stats` (DPS, EHP, speed, align, capacitor from the dogma engine); community fits: `get_fits`, `list_fits`
- Skills and industry: `get_required_skills` (skill plans for up to 20 items, with training time for an Omega clone), `get_production_chain` (materials, build cost with `include_cost`, job fee with `system`, planetary chains for P1-P4 products), `get_reprocessing_yield`
- Space: `get_jumps_between` (summary or route, `avoid`, `via`), `get_jump_distances` (one origin to up to 200 systems), `find_systems` (filter by security, region, faction, wormhole class, stations, belts, planets, ice, distance, or jump-drive reach), `get_system_info`, `get_systems_in_region`, `get_npc_stations` (a system's stations, or the nearest stations by service and mission agents by level, division or corporation), `plan_jump_route` (Jump Freighter, Rorqual, Black Ops, capital, titan or Ansiblex chain)
- Prices and LP: `get_lp_offers` (an NPC corporation's LP store ranked by ISK per LP at average prices), `get_market_price` (best sell/buy and order counts, per trade hub), `get_average_prices` (up to 200 items), `get_market_history`
- Live data from EVE's public ESI (shared cache): `get_market_price`, `get_market_history`, `get_type_info`, `get_sovereignty`, `get_system_activity` (kills, jumps), `get_lp_offers`, `get_production_chain` with `system`, and `plan_jump_route` with `avoid_incursions` or `live_info`; `get_thera_connections` is live data from EvE-Scout / Signal Cartel
- With an EVE-Cyno API key: `analyze_battle`, and free-text fit search for `/eve-fits`
- `appraise_items` gives an estimate from ESI average prices without a key; your own Janice key makes it a Janice appraisal: at install Claude Code asks for an optional **Janice API key** (stored as a sensitive value; leave it empty if you don't have one, and add it later in the plugin settings under `/plugin`). It is sent only to `data.eve-cyno.dev`, which forwards it to Janice for that one request and never stores or logs it

Skill `eve-data`: tool selection, recommended flows (fits: validate, which carries the stat card on a valid fit; map questions; batch first), limits, what is not available, and the attribution rules.

Commands:

- `/eve-fit <EFT block>`: validate a fit and compute its stats
- `/eve-item <name>`: look up an item or ship
- `/eve-fits <hull> [activity]`: community fits for a hull, with author credit (uses the public REST endpoint)

If another plugin uses the same command name, use the namespaced form (`/eve-cyno:eve-fit` and so on). In Claude Code the MCP tools are named `mcp__plugin_eve-cyno_eve-cyno__<tool>`; to allow them all without a prompt, allow `mcp__plugin_eve-cyno_eve-cyno__*`.

## Privacy and limits

No account and no key are needed for the public tools; both keys above are optional. Requests go to `data.eve-cyno.dev` and are rate limited per caller, per API key when one is sent, otherwise per client IP (20 MCP requests per minute without a key, 120 with one, 200 requests a day without a key; the live tools also 5 calls per minute without a key, 10 with one). What you send to a tool, such as an EFT block or a system name, is processed by that service. See the [terms](https://data.eve-cyno.dev/terms). Docs: <https://data.eve-cyno.dev/docs/>, [`llms.txt`](https://data.eve-cyno.dev/llms.txt), [OpenAPI](https://data.eve-cyno.dev/v1/openapi.yaml).

## Support

Questions, feedback and bug reports: the [EVE-Cyno Discord](https://discord.gg/XZzkH38dt9).

## Notices

© 2014 Fenris Creations hf. All rights reserved. "EVE", "EVE Online", "Fenris Creations", and all related logos and images are trademarks or registered trademarks of Fenris Creations hf. This material is used with limited permission of Fenris Creations hf. (formerly CCP hf., rebranded May 2026). No official affiliation or endorsement by Fenris Creations is stated or implied. Created under the EVE Developer License Agreement.

Some reference text served by the tools is taken or condensed from the [EVE University Wiki](https://wiki.eveuniversity.org/), © EVE University and its wiki contributors, licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Community fits belong to their authors and sites and carry their own attribution.

## Licence

MIT, Copyright (c) 2026 Bythlak. See [LICENSE](LICENSE).
