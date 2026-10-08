---
description: Find community fits for an EVE Online hull (with author and source credit)
argument-hint: <hull name> [activity such as pve, pvp, abyss]
allowed-tools: WebFetch(domain:data.eve-cyno.dev)
---

Find community fits for: $ARGUMENTS

Community-fit search is a public REST endpoint, not an MCP tool.

1. Resolve the hull name with the `search_item_by_name` MCP tool if it is loose or ambiguous.
2. Fetch `https://data.eve-cyno.dev/v1/fits/search?ship=<hull>&limit=5` (URL-encode the name). Add `activity=pve` or `activity=pvp` and `tag=abyss` style filters when the user asked for them. There is no paging; one query returns one ranked page, at most 24 hits.
3. For each hit show the fit name, key facts, and its `attribution`: author when known, source, `source_url`, licence, plus `credit` and `license_url` when present. Never drop the attribution.
4. If `relaxed` is true, tell the user which filters (`dropped`) were removed to get results.
5. Optionally run the best fit through `validate_fitting` and `compute_fit_stats` and say so clearly, as that is our computation, not the author's claim.

Community fits belong to their authors and sites. Do not bulk-export or scrape the corpus.
