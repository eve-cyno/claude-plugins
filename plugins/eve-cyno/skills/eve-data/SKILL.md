---
name: eve-data
description: Use when the user asks about EVE Online ships, modules, fittings (EFT blocks), DPS/EHP/cap stats, skills to fly or build something, manufacturing materials, reprocessing, routes between systems, or NPC stations. Explains which EVE-Cyno MCP tool to call and in what order, plus the attribution to keep.
---

# EVE Online data via the EVE-Cyno MCP server

The `eve-cyno` MCP server (`https://data.eve-cyno.dev/v1/mcp`) gives deterministic EVE Online data and calculations. No language model is behind it: the same input gives the same answer. Tool names appear as `mcp__eve-cyno__<tool>` (or `mcp__plugin_eve-cyno_eve-cyno__<tool>` when installed as a plugin). Never estimate DPS, EHP, slot counts or type IDs from memory when a tool can answer.

## Which tool when

| Question | Tool |
|----------|------|
| Name to type ID, or "does this item exist" | `search_item_by_name` (accepts several names at once) |
| Slots, hardpoints, CPU/PG of a hull | `get_ship_stats` |
| Hull and role bonuses (which weapon system to use) | `get_ship_bonuses` |
| Weapon-locked or unfamiliar hulls (Triglavian, pirate, faction) | `get_hull_facts` (when `compatible_weapons` is non-empty, use only those) |
| Correct module name for a family ("Steel Plates", "Heat Sink") | `find_canonical_module` (with `size` Small/Medium/Large) |
| Is this EFT fit legal (CPU, PG, slots)? | `validate_fitting` |
| DPS, volley, EHP, speed, align, cap stability of an EFT fit | `compute_fit_stats` |
| Skills needed to fly a ship or build an item | `get_required_skills` (`purpose`: fly or build) |
| Materials to manufacture an item | `get_production_chain` (`me_level`, `runs`, rig and cost index optional) |
| Reprocessing output | `get_reprocessing_yield` |
| Jumps and route between two systems | `get_jumps_between` (`prefer_high_sec`) |
| Systems of a region by security band | `get_systems_in_region` |
| NPC stations in a system | `get_npc_stations` |
| ISK value of items | `appraise_items` (see limits: needs the user's own Janice key) |

## Recommended flows

- **Building or reviewing a fit:** `get_ship_bonuses` (and `get_hull_facts` on specialist hulls) to pick the bonused weapon system, `find_canonical_module` for any name you are not sure of, write the EFT block, `validate_fitting`, fix every violation and unresolved module, then `compute_fit_stats` and report its numbers. State the stats basis it reports (all skills V, no implants, default T1 ammo when none is on the EFT line).
- **Item questions:** resolve names with `search_item_by_name` first, then use the exact names in other tools.
- **Industry:** `get_required_skills` with `purpose: build`, then `get_production_chain`. Only pass rig level or cost index when the user mentioned a structure or system.

## Limits and tiers

- Public tools need no key: everything above except `appraise_items`. Do not ask the user for an EVE-Cyno API key; none is issued.
- `appraise_items` uses the user's own Janice key, entered as the optional "Janice API key" when the plugin was installed (sent as `X-Janice-Key`). If the call fails with `janice_key_required`, tell the user they can add their Janice key in the plugin settings (`/plugin`), say prices are unavailable, and do not guess ISK values.
- Rate limits apply per client IP (MCP: 60 requests per minute). On `rate_limited` wait for `Retry-After` and do not loop.
- Request bodies are capped at 64 KiB on the MCP endpoint.
- Errors come back as `{"error":{"code","message"}}`; report the code rather than inventing an answer.

## Not available

No live market prices or price history (only Janice appraisal with the user's key), no character or corporation data, no killboard, no sovereignty or system-activity data on the public tier, no community-fit search over MCP (the `/eve-fits` command uses the public REST endpoint instead). If a question needs one of these, say so.

## Attribution (keep it)

- Game data is Fenris Creations data under the EVE Developer License Agreement. When you present results in a document, post or shared output, include: "EVE Online and all related marks are trademarks of Fenris Creations hf. Used with limited permission; no official affiliation or endorsement. Data via EVE-Cyno (https://eve-cyno.dev)." A short note is enough in plain chat.
- Text taken from the EVE University Wiki is CC BY-SA 4.0 (https://wiki.eveuniversity.org/); credit it and keep the licence on derived text.
- Community fits belong to their authors and sites. Every fit carries an `attribution` object (`source`, `source_url`, `author` when known, `license`, `credit`, `license_url`). Show the author and source link next to every fit you show, and never present a community fit as your own.
- Tool results report their sources (the MCP `_meta` attribution); cite them when asked where a number comes from.
- Terms of the service: https://data.eve-cyno.dev/terms
