---
description: Look up an EVE Online item or ship (type ID, slots, bonuses, skills, build materials)
argument-hint: <item or ship name>
---

Look up this EVE Online item or ship: $ARGUMENTS

Follow the `eve-data` skill.

1. Call `search_item_by_name` with the name to get the exact name and type ID. If nothing matches, say so and suggest close names.
2. If it is a ship, also call `get_ship_stats` and `get_ship_bonuses` (and `get_hull_facts` if it is a specialist or faction hull).
3. Call `get_required_skills` (purpose `fly` for ships and modules, `build` for blueprints) and summarise the skill requirements.
4. If the user asks how to build it, call `get_production_chain`.

Present a compact summary. Prices: `get_market_price` gives the current Jita price of one item; to value a list of items use `appraise_items`, which gives an ESI estimate without a key or a Janice appraisal with the user's own Janice key. Say which one you got; do not invent prices.
