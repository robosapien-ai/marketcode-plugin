---
name: marketcode-property-research
description: How to answer UK property questions well with the MarketCode tools. Resolve an address to a UPRN first, start with property_summary, use the volume tools for counts, prefer a curated tool over query_sql, and state the credit cost before a paid call.
---

# Researching UK property with MarketCode

MarketCode exposes read-only tools over a UK property warehouse: addresses and
UPRNs, per-property records, ownership, planning, valuations, auction outcomes,
energy data and market analytics. Every tool states its credit cost in its
description; most are free.

## Work from the UPRN

1. Turn what the user typed into a UPRN. `address_autocomplete` is free and
   returns ranked candidates; `address_resolve` (8 credits) turns free text
   into a canonical record. If the user already gives a UPRN, skip both.
2. For "tell me about this property", call `property_summary` first (4
   credits). It usually answers the question on its own. Reach for
   `property_lookup` (5 credits, up to ~300 fields) only when the summary is
   not enough.
3. `property_history`, `transactions_by_uprn`, `epc_certificates`,
   `property_flood_risk`, `council_tax_band`, `valuation_estimate` and
   `valuation_full` are free and keyed by UPRN.

## Areas and markets

- Resolve a named place with `area_resolve` (free) before any area tool. Pass
  `analytics_area_id` when the resolve response gives both ids.
- For "how many sales" questions use `market_volume_series` or
  `market_volume_ranking`, never SQL.
- `market_facts` answers for one geography; `market_ranking` compares areas.
  Do not loop `market_facts` to build a league table.
- `market_index_series` is the published house price index for a district or
  local authority; the forecast tail is flagged in the response.

## Auctions, ownership and sourcing

- Auction tools are free and return achieved outcomes. Read `meta.coverage`:
  about a quarter of lots carry a hammer price. Say so when results are thin.
- `ownership_by_title` and `ownership_by_company` (3 credits each) read the
  Land Registry corporate and overseas registers. Individually owned titles
  are a gap in the free register, not a failed lookup.
- `sourcing_search` (8 credits) finds land or units by criteria;
  `sourcing_owners` (5 credits) names owners for the parcels it returns.

## Credits

- The tool description carries the cost. Before a paid call, tell the user what
  it costs. Failed calls are not charged, except `query_semantic` and
  `query_sql`, which are charged when the warehouse did the work.
- If a call fails with an out-of-credits message, relay it and stop. Retrying
  will not help.
- Prefer a curated tool over `query_sql` (25 credits): curated tools are
  cheaper, faster and already correct. `query_sql` reaches only the
  `marts_core`, `marts_facts`, `dimensions` and `location` schemas.

## Wording

- `equity_estimate` is value against last sale price, not equity net of debt.
  Never describe it as loan-to-value.
- `mortgage_rates` is the Bank of England market average, not a quote.
- Valuations are estimates with a range. Quote the range, not only the point.
- Coverage is England and Wales for registry data and UK for addresses; say
  which when it matters.

## Coverage first (added 7 Sep 2026)

For any area-level claim, call `data_coverage(area)` (free) first and state
the field's coverage beside the figure. For a shop, office or other rated
unit, `commercial_property(uprn | uarn)` (2 credits) is the report; its
`income_basis_value` is an indicative capitalisation of the rateable value,
not a valuation — say so.

## Property media (added 8 Sep 2026)

`property_media(uprn)` (1 credit) returns photographs, floorplans, EPC graphs
and brochures for a property. Narrow with `types`.

EVERY LINK IS OURS AND EXPIRES. Do not store them — call again for fresh ones.
The portal's own URL is never returned, so a saved response cannot become a
hotlink later.

COVERAGE IS UNEVEN. We hold the actual files for a minority of floorplans and
brochures only; everything else, including all photographs, is a link that
redirects to the source. `n_held_by_us` against `n_redirected` says which.
`no_media` is as often a UPRN matching gap as an absence of photographs —
about a third of listings never resolve to a UPRN — so do not tell a user a
property has no pictures.

Each item carries the portal and host it came from — worth quoting when a user
asks where a picture came from.
