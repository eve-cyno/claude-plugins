---
description: Validate an EFT fit and compute its DPS, EHP, speed and capacitor with the EVE-Cyno tools
argument-hint: <EFT block, or a path to a file containing one>
---

Analyse this EVE Online fit: $ARGUMENTS

Follow the `eve-data` skill. If the argument is a file path, read the file; if it is empty, ask the user to paste an EFT block.

1. Call the `validate_fitting` MCP tool with the full EFT text. List every violation, warning and unresolved module. If modules are unresolved, use `find_canonical_module` to suggest the correct names.
2. If the fit is valid (or after the user accepts corrections), call `compute_fit_stats`.
3. Report: ship, DPS (per weapon group and charge), volley, EHP per layer with resists, speed, align time, capacitor stability, CPU/PG load. Say which assumptions the tool reports (skills, ammo defaults).
4. Give two or three concrete, short improvement ideas grounded in `get_ship_bonuses`, and re-validate any changed EFT before presenting it.

Never estimate the numbers by hand. Keep the EVE Online data notice from the skill when the output is meant to be shared.
