---
name: eve-data
description: Answers EVE Online questions with deterministic tools instead of guesses: ships, modules, fittings (EFT blocks), DPS/EHP/cap stats, skills to fly or build something, manufacturing materials, reprocessing, routes and jump distances between systems, system search, jump-drive planning for capitals, NPC stations and mission agents, LP store offers, skill plans and training time, build costs, planetary production chains, current market prices and price history, Thera connections, item details, sovereignty and recent system activity. Use it whenever a question needs EVE data or a calculation. Says which EVE-Cyno tool to call and in what order, works over MCP or plain REST, and lists the attribution to keep.
---

# EVE Online data via the EVE-Cyno tools

EVE-Cyno (`https://data.eve-cyno.dev`) gives deterministic EVE Online data and calculations. No language model is behind it: the same input gives the same answer. Never estimate DPS, EHP, slot counts or type IDs from memory when a tool can answer.

Use the `eve-cyno` MCP tools when your client has them (remote MCP server `https://data.eve-cyno.dev/v1/mcp`). The tool names vary by client: `mcp__eve-cyno__<tool>` in Claude Code with the server added by hand, `mcp__plugin_eve-cyno_eve-cyno__<tool>` when installed as the plugin; other clients show their own prefix. Without MCP, call the same tools over REST (see "Without MCP (REST)" below). The tool names and arguments are identical on both.

## Which tool when

| Question | Tool |
|----------|------|
| Name to type ID, or "does this item exist" | `search_item_by_name` (accepts several names at once) |
| Classify a list of items (tech level, faction, Alpha use), or look up abyssal NPCs and filaments | `search_item_by_name` with `details: true` (up to 50 names, one line of facts each) |
| Slots, hardpoints, CPU/PG of a hull | `get_ship_stats` |
| Hull and role bonuses (which weapon system to use) | `get_ship_bonuses` |
| Weapon-locked or unfamiliar hulls (Triglavian, pirate, faction) | `get_hull_facts` (when `compatible_weapons` is non-empty, use only those) |
| Correct module name for a family ("Steel Plates", "Heat Sink") | `find_canonical_module` (with `size` Small/Medium/Large) |
| Modules by slot or meta group ("Tech II mid-slot shield modules") | `find_canonical_module` with `family` (a name part, at least 2 characters) plus `slot` (high, mid, low, rig), `meta` ("Tech II", "Faction", ...), optional `kind` and `limit` (1-30) |
| Is this EFT fit legal (CPU, PG, slots)? | `validate_fitting` |
| DPS, volley, EHP, speed, align, cap stability of an EFT fit | `validate_fitting` on a valid fit already ends with a `Fit stats:` card; `compute_fit_stats` only when it did not (a fixed fit) or to choose charges |
| Browse community fits cheaply | `list_fits`, or `get_fits` with `include_eft: false`; then `get_fits` with `fit_id` for the one fit to copy |
| Skills needed to fly a ship or build an item | `get_required_skills` (`purpose`: fly or build) |
| Skill plan or training time for one or more items | `get_required_skills` with `items` (up to 20) and `training_time: true` (an Omega clone from zero SP, each skill at its best remap 27/21 with no implants: the fastest possible time, not a typical one; pass `attributes` for the character's own five values, all or none) |
| Materials to manufacture an item | `get_production_chain` (`me_level`, `runs`, rig and cost index optional) |
| Build cost of an item, job fee in a system | `get_production_chain` with `include_cost: true` and `system` (live cost index; `facility_tax_pct` for a structure) |
| Planetary production chain (P1-P4 commodity down to raw resources) | `get_production_chain` with the P1-P4 name (`runs` counts cycles of the final schematic; ME and rigs do not apply) |
| Reprocessing output | `get_reprocessing_yield` |
| Jumps and route between two systems | `get_jumps_between` (`prefer_high_sec`; `avoid`, `via`). For the jump count only, pass `detail: "summary"`; ask for the hop list only when the user wants the route |
| Distances from one system to a list of systems | `get_jump_distances` (one call, up to 200 systems, nearest first) |
| Find systems by security, region, faction, wormhole class, stations, belts, planets, ice or distance | `find_systems` (`near` for gate jumps, sort `jumps`) |
| Facts about named systems | `get_system_info` (up to 50 systems; unenterable Jove or market pseudo-space systems are named "Not enterable") |
| Jump-drive route for a Jump Freighter, Rorqual, Black Ops, capital or titan (or an Ansiblex chain: ship "Jump Bridge") | `plan_jump_route` (skills default to 5; `waypoints`, `avoid`, `avoid_regions`, `prefer_stations`; `avoid_incursions` or `live_info` cost one live call) |
| Systems a ship can jump to from X (nearest lowsec station for a Rorqual, titan bridge range, Black Ops reach) | `find_systems` with `jump_range` (`origin`, `ship`) plus the usual filters, sorted by `ly` |
| Systems of a region by security band | `get_systems_in_region` |
| NPC stations in a system | `get_npc_stations` |
| Nearest station with a service, or the nearest mission agent of a level, division or corporation | `get_npc_stations` with `near` (`origin`, `max_jumps`), `services`, `agent` (`level`, `division`, `corporation`, `faction`, `kind`), `limit` (1-50) |
| LP store value of an NPC corporation (best offers by ISK per LP) | `get_lp_offers` (`corporation`; `item`, `sort`, `limit` optional; live ESI, priced at average prices, not live orders: say so) |
| Current price of one item (best sell/buy, order counts); Jita region by default | `get_market_price` (live ESI; `hub` for Jita, Amarr, Dodixie, Rens or Hek main station) |
| Average prices of many items | `get_average_prices` (up to 200 items; an average of recent trades, not live orders: say so) |
| Price history or trend of an item | `get_market_history` (live ESI; per day or week, 1-90 days) |
| Thera or Turnur shortcuts | `get_thera_connections` (live, EvE-Scout; credit "EvE-Scout / Signal Cartel") |
| Item name, description, volume, mass, group | `get_type_info` (answers from the local SDE, falls back to live ESI) |
| Who holds sovereignty (names) | `get_sovereignty` with `systems` (up to 50; the single `system_name` form answers with ids only; live ESI) |
| Activity in the last hour including jumps | `get_system_activity` with `systems` (up to 50; the single `system_name` form has no jumps; live ESI) |
| ISK value of a list of items | `appraise_items` (an estimate from ESI average prices; a Janice appraisal with the user's Janice key) |

## Recommended flows

- **Building or reviewing a fit:** `get_ship_bonuses` (on specialist or faction hulls `get_hull_facts` instead: same bonus lines plus the weapon lock) to pick the bonused weapon system, `find_canonical_module` for any name you are not sure of, write the EFT block, `validate_fitting`, fix every violation and unresolved module, then report. `validate_fitting` on a valid fit already ends with a `Fit stats:` card (the same engine as `compute_fit_stats`): report it. Call `compute_fit_stats` only when validate did not return one (a fit fixed afterwards) or to choose charges. State the stats basis it reports (all skills V, no implants, default T1 ammo when none is on the EFT line).
- **Map questions:** answer "nearest highsec system with each ice type (Clear Icicle, Glacial Mass, ...) to X" with one `find_systems` per ice type asked for, because a list in `ice_types` matches any of them (4 types = 4 calls). Each call: `security.min` 0.45, `ice_types` with that one type, `near.origin`, `sort: jumps`, `limit: 3`. Its lines already carry the jump counts. Add `get_jumps_between` with `detail: "summary"` only when the user asks about the route or its safety.
- **Batch first:** for lists of systems or items (not ice types, see above) prefer one list call over many single calls. A call counts as one request whatever its list length (caps: 200 distances or prices, 50 systems, 100 rows).
- **Item questions:** resolve names with `search_item_by_name` first, then use the exact names in other tools.
- **Industry:** `get_required_skills` with `purpose: build`, then `get_production_chain`. Only pass rig level or cost index when the user mentioned a structure or system. For "what does it cost to build" add `include_cost: true`, and `system` only when the user names where the job runs.
- **LP stores:** `get_lp_offers` ranks a corporation's offers by ISK per LP at ESI average prices, so treat the figures as a guide. An offer with an item that has no average price (a blueprint copy) shows no value and says why.

## Limits and tiers

- Public tools need no key: everything above, `appraise_items` included.
- `analyze_battle` appears only when the user set the optional "EVE-Cyno API key" (sent as `X-API-Key`). If the user asks for a battle report and it is not in your tool list, tell them the key is free: `/apikey create` in the EVE-Cyno Discord (https://discord.gg/XZzkH38dt9) or ai.eve-cyno.dev, Settings, Data API keys; then add it where your client keeps the server settings: under `/plugin` in Claude Code, as the `X-API-Key` header of the MCP server in other clients, or as the `X-API-Key` header of a REST call. Custom connectors in claude.ai and Claude Desktop reach the server from Anthropic's addresses, so without a key every connector user shares one per-client limit; a free key gives the user their own.
- `appraise_items` works without any key: it estimates the value from EVE's average prices (ESI, refreshed daily) and says so; tell the user it is an estimate, not live Jita prices. With the user's own Janice key (the optional "Janice API key" in the plugin settings in Claude Code, or the `X-Janice-Key` header in any other client or REST call) it is a Janice appraisal with Jita buy, sell and split totals.
- Rate limits apply per caller: per API key when one is set, otherwise per client IP (MCP: 20 requests per minute without a key, 120 with one; REST: 10 without, 60 with; 200 requests a day without a key). On `rate_limited` wait for `Retry-After` and do not loop.
- The live tools (`get_market_price`, `get_type_info`, `get_sovereignty`, `get_system_activity`, `get_market_history`, `get_lp_offers`, `plan_jump_route` with `avoid_incursions` or `live_info`, `get_production_chain` with `system` (ESI) and `get_thera_connections` (EvE-Scout)) also share a limit of 5 calls per minute per client without a key, 10 with one (MCP and REST together). `get_type_info` reads the local SDE first and falls back to live ESI, but it still counts against this limit. The ESI tools fail with `upstream_busy` when the shared ESI budget is spent; over the per-caller live limit any live tool returns `rate_limited`. Say live data is unavailable right now and do not retry in a loop. Their values change; quote them as of the call.
- Request bodies are capped at 64 KiB on the MCP and tool endpoints.
- Errors come back as `{"error":{"code","message"}}` (over MCP, an error result with the code first); report the code rather than inventing an answer.

## Without MCP (REST)

If your client has no MCP support, call a tool with `POST https://data.eve-cyno.dev/v1/tool/<name>`. The body is the tool's arguments as a JSON object, sent with `Content-Type: application/json`:

```
curl -sS -X POST https://data.eve-cyno.dev/v1/tool/search_item_by_name \
  -H 'Content-Type: application/json' \
  -d '{"names":["Tritanium"]}'
```

The answer is `{"tool","version","result":{"text","data"},"attribution":[{"name","url","license"}]}`. `result.text` is the answer to read, `result.data` the typed result where the tool has one (otherwise `null`), `attribution` the sources to credit. Example reply: `{"tool":"search_item_by_name","version":"…","result":{"text":"Tritanium: type_id=34","data":null},"attribution":[…]}`.

- Keys are optional: `X-API-Key: <key>` (or `Authorization: Bearer <key>`) for the keyed tools and higher limits; `X-Janice-Key: <key>` for a Janice appraisal in `appraise_items`. Send keys only to `data.eve-cyno.dev`.
- Errors are JSON with a non-2xx status: `{"error":{"code":"...","message":"..."}}`, for example `404 unknown_tool`, `400 invalid_arguments`, `401 api_key_required`, `429 rate_limited` or `daily_quota_exceeded` (wait `Retry-After` seconds), `503 upstream_busy`.
- The full tool list with every argument is at `https://data.eve-cyno.dev/llms-full.txt`; the OpenAPI document is `https://data.eve-cyno.dev/v1/openapi.yaml`. Read the argument list there before calling a tool you have not used.

## Not available

No character or corporation data, no killboard, no wormhole statics, no moon ore, no abyss spawn tables (only the facts of a named abyssal NPC or filament), no training time for a real character (the plan assumes an Omega clone from zero SP, or the attributes you pass), no live order-book LP values (`get_lp_offers` uses average prices), no nullsec ice belts (the ice list covers highsec and lowsec only), no free-text community-fit search over MCP (`get_fits` and `list_fits` filter by hull and activity; free-text search is the REST `q` parameter, which needs an API key; the Claude Code plugin's `/eve-fits` command searches by hull on the public REST endpoint). If a question needs one of these, say so.

## Attribution (keep it)

- Game data is Fenris Creations data under the EVE Developer License Agreement. When you present results in a document, post or shared output, include: "EVE Online and all related marks are trademarks of Fenris Creations hf. Used with limited permission; no official affiliation or endorsement. Data via EVE-Cyno (https://eve-cyno.dev)." A short note is enough in plain chat.
- Text taken from the EVE University Wiki is CC BY-SA 4.0 (https://wiki.eveuniversity.org/); credit it and keep the licence on derived text. This includes the ice-belt data (EVE University, CC BY-SA 4.0).
- Thera and Turnur connection data is EvE-Scout / Signal Cartel (https://www.eve-scout.com/); credit them with the link.
- Community fits belong to their authors and sites. Every fit carries an `attribution` object (`source`, `source_url`, `author` when known, `license`, `credit`, `license_url`). Show the author and source link next to every fit you show, and never present a community fit as your own.
- Tool results report their sources (the MCP `_meta` attribution); cite them when asked where a number comes from.
- Terms of the service: https://data.eve-cyno.dev/terms
