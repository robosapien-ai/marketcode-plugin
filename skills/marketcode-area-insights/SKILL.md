---
name: marketcode-area-insights
description: Produce a market brief for any UK area with the MarketCode tools: headline metrics, the published sales and asking indices, volumes, rents and yields, live listing stock, agent activity, planning pipeline, built stock and auction outcomes, compared with neighbours. Use when the user asks "what is the market like in X", wants an area report, a comparison of districts, or a ranking. Every call in this skill is free.
module: insights
---

# Area market brief

You are producing an area-level market brief: what the market is doing,
how it compares with its neighbours, and what to watch. The deliverable is
markdown with tables; every figure names the tool it came from.

Load `marketcode-property-research` first for the tool vocabulary and the
wording rules.

## Credit budget

Every tool below costs 0 credits. Say so, then proceed.

## Inputs

An area: a postcode district (BS5), a town, a borough or a local authority.
Optionally a bedroom band or property type focus, and a comparison set.

## Phase A: evidence (parallel)

1. **Resolve.** `area_resolve(area_name)` once. Use the `area_id` it returns
   for the `area_id` tools and the `area_code` and `area_type` it returns for
   the series tools. If it returns both an `area_id` and an
   `analytics_area_id`, pass `analytics_area_id` where the tool asks for
   analytics.
2. **Prices and rents.** `market_facts` for the area with `max_periods` of 36
   months: price, £/m², volume, rent, yield, stock, turnover, by bedroom
   band where available.
3. **Indices.** `market_index_series` (sales, with the forecast tail
   flagged), `asking_price_index_series`, `asking_rent_index_series` for the
   same area; `commercial_index_series` and `commercial_rent_index_series`
   only if the user asked about commercial.
4. **Volumes.** `market_volume_series` for the district over 36 months;
   `market_volume_ranking` to place it among its county or region.
5. **Supply side.** `listing_market_series` (new listings, reduction share,
   time to reduce, yields, agent concentration) and `listing_stock_series`
   (live stock by day). `agent_activity(area_id)` for who is transacting.
6. **Stock.** `area_stock_profile(area_id)`, `energy_profile(area_id)`.
7. **Pipeline.** `planning_applications(location)`.
8. **Auctions.** `auction_stats(postcode_district)`, `auction_discount`,
   and `distressed_assets` if the user is investing.
9. **Peers.** `market_ranking` for the parent area with the same metrics,
   so the subject sits in a league table of its neighbours. Never loop
   `market_facts` over areas to build a table; `market_ranking` is the tool
   for that.

## Phase B: reading the evidence

- **Verdict first.** One sentence that states the dominant story:
  "Buyer's market for flats, tight for houses, rents still rising."
- **Direction, not just level.** For every headline metric give the latest
  value, the 12-month change and the 36-month change, and the peer median.
- **Thin data.** `market_facts` carries evidence counts and provenance; if
  a cell rests on fewer than 20 transactions, say so and widen to the parent
  area for that metric.
- **Recent months are incomplete.** Land Registry registrations lag by
  weeks to months. Treat the last two months of any sales series as partial
  and say so; the asking indices and listing stock are current.
- **Forecast tails are flagged** in the index response; label them
  "projection" and never present them as observed.
- **Supply meets demand.** Rising new listings with rising reduction share
  and lengthening time to reduce is softening; falling stock with falling
  reductions is tightening. Say which you see.
- **Auctions are the stress signal.** Clearance rate, discount to the
  MarketCode estimate and repeat failures tell you where forced sellers are.
  About a quarter of lots carry an achieved price; read `meta.coverage` and
  say so.

## Brief (markdown, in this order)

1. **Verdict**: one sentence, then three stance tags (sales, rentals, yield).
2. **Headline metrics** table: median price, £/m², gross yield, median rent,
   monthly volume, live stock, with 12-month and 36-month change and the
   peer median.
3. **Price trends**: sales index and asking-price index, 36 months, in a
   table or a chart spec the client can render; note divergence between
   asking and achieved.
4. **Rental market**: rent by bedroom band, asking-rent index, yield band.
5. **Supply**: new listings, reductions, time to reduce, live stock, most
   active agents.
6. **Pipeline**: recent applications by type and status, clustered.
7. **Stock and energy**: built form, tenure mix, EPC distribution.
8. **Auctions**: clearance, discount to estimate, repeat-failure lots.
9. **Peers**: the ranking table.
10. **Watch-outs**: three to five risks, each one line with the evidence.
11. **Sources**: one line per tool, with the period and the coverage note.

## Rules

- Numbers are raw and dated; percentages are percentages, not decimals.
- Omit a section whose tool returned nothing, and say it was omitted;
  never fill a gap with a plausible number.
- Plain English; explain what each finding means for a buyer, a seller or
  a landlord, not only what the data says.
