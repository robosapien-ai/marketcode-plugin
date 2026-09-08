---
name: marketcode-deal-analysis
description: Appraise a development site or an income-producing scheme with the MarketCode tools: site position, planning constraints, market evidence, GDV (capital comps for build-for-sale, income capitalisation for BTR, PBSA, co-living and industrial), a cost stack from the user's assumptions, residual land value or profit, sensitivities and an investment-committee style recommendation. Use for "appraise this site", "is this scheme viable", "GDV", "RLV", "deal memo".
module: sourcing
---

# Deal analysis and development appraisal

You are producing a first-pass appraisal for a site or scheme, written for
an investment committee: what can be built, what it is worth on completion,
what it costs, what is left for the land or as profit, and how sensitive
that is. MarketCode supplies the site, planning and market evidence; the
cost stack comes from the user's assumptions (with defaults you state).

Load `marketcode-property-research` first. For a residential unit's value on
its own, use `marketcode-valuation`; this skill is for schemes.

## Credit budget (state it before you start)

| Call | Credits |
|---|---|
| `address_resolve` (if text, not a UPRN) | 8 |
| `site_appraisal` (parcel, Tier-0 value, owner, EPC, auctions, designations) | 5 |
| `property_summary` (existing building on site) | 4 |
| `planning_designations`, `planning_applications`, `property_flood_risk`, `market_facts`, `market_index_series`, `asking_rent_index_series`, `commercial_index_series`, `commercial_rent_index_series`, `valuation_full`, `mortgage_rates`, `lha_rate`, `auction_stats` | 0 |

Typical run: 5 to 17 credits. `cost_index` (free) gives the tender / output / deflator series for the cost stack's inflation assumption; `time_adjustment` (free) carries any historic price to today.

## Inputs

Required: the site (address, UPRN or parcel) and the scheme (use class or
type, unit count or GIA, description). Ask for missing required inputs in
one message. Then offer the assumptions you will default and let the user
override in one reply: build cost per m² or per unit, professional fees %,
contingency %, finance rate (default from `mortgage_rates` plus a margin,
stated), programme months, target profit on GDV, land cost if known,
affordable %, exit yield for income schemes. If the user says "use
defaults", proceed.

## Phase A: site and planning (parallel)

1. `site_appraisal(uprn|land_id, include_designations=true)`: parcel area,
   built coverage, indicative residual value, owner, existing EPC, nearby
   auction outcomes, designations in one call.
2. `planning_designations(postcode)` if not included; `planning_applications`
   for the history around the site: what was approved, refused, appealed.
3. `property_flood_risk(uprn)` for the surface-water position.

Grade each constraint: BLOCKER (green belt, functional floodplain), HIGH
(conservation area, AONB, Grade I or II* nearby), MEDIUM (flood zone 2,
Grade II), LOW. State the development sensitivity that follows.

## Phase B: market evidence

- **Build-for-sale residential (C3):** `market_facts` for the district by
  bedroom band (price, £/m², volume, evidence counts); `market_index_series`
  for direction; `valuation_full` on two or three comparable completed units
  nearby if the user names them. GDV = Σ unit mix × achieved price by bed
  band, cross-checked against £/m² × sellable area. Use achieved prices,
  not asking; the asking index tells you the gap.
- **Income-producing (BTR, PBSA, co-living, industrial, office):**
  `market_facts` rents by bedroom band and `asking_rent_index_series` for
  residential income; `commercial_rent_index_series` and
  `commercial_index_series` for commercial. `lha_rate` where affordable or
  LHA-linked units apply. GDV = NOI ÷ exit yield, never capital comps.

Income formulas (state defaults, invite overrides):

| Scheme | Gross income | Opex | Watch |
|---|---|---|---|
| BTR | Σ units × monthly rent × 12 × (1 − 3% void) | 25 to 35% | affordable share at a discount |
| Industrial (FRI) | GIA × ERV × (1 − 5% void) | management ~5% only | tenant pays repairs and insurance |
| PBSA | Σ beds × weekly rent × contract weeks (44) × 97% | 30 to 42% | model contract weeks, not 52 |
| Co-living | studios × all-in monthly rent × 12 × 95% | 25 to 30% | all-in rent is not comparable to bills-exclusive rent |

If a promoter's document states opex below 20% for BTR or co-living, flag
it as lean and run the appraisal at 25% and 30% as well.

## Phase C: the appraisal

Cost stack (user assumptions, defaults stated): build cost, professional
fees (10 to 12%), contingency (5%), planning obligations (CIL and S106 if
the user knows the rate; otherwise say it is excluded and why), marketing
and sales (2 to 3% of GDV), finance (rate × drawn balance × programme),
target profit (15 to 20% on GDV for build-for-sale).

- Residual land value = GDV − costs − target profit.
- With a land cost given: profit = GDV − (land + costs); profit on GDV and
  on cost.
- Sensitivities: build cost ±10%, GDV or exit yield ±50 bps, programme +6
  months, rent growth ±1%. Show a small table.

Cross-check the Tier-0 residual value from `site_appraisal` against your
RLV and explain the difference (it is a generic screen; yours uses the
scheme).

## Report (markdown)

1. **Executive summary**: scheme, GDV, total cost, RLV or profit, profit on
   GDV, headline sensitivity, top three reasons to proceed, top three risks,
   recommendation (proceed, proceed with conditions, decline).
2. **Site and planning position**: constraints table with grades, history,
   what can realistically be built.
3. **Market evidence**: prices or rents by bed band with evidence counts,
   index direction, yield band, comparables used.
4. **Appraisal**: GDV build-up, cost stack, residual or profit, with every
   assumption labelled as user-supplied or default.
5. **Sensitivities** table.
6. **Risks and mitigations**: planning, construction, market, operational.
7. **Sources**: one line per tool and field; one line per assumption.

## Rules

- Distinguish our view from the market's view when they differ.
- No point estimates without a range or a sensitivity.
- Never present a default cost as evidence; it is an assumption the user
  can change, and the report says so.
- MarketCode has no build-cost tool; do not invent one. If the user has no
  cost view, run the appraisal at a low and a high cost and show both.

## Build cost and the commercial report (added 7 Sep 2026)

`build_cost(sector, region, gia_m2, rate_gbp_m2, rate_period)` (free) gives
the sector's cost index with its 1- and 5-year movement, the regional factor
(NATIONAL, LONDON, SOUTH_EAST) and the on-cost percentages; with a GIA and
YOUR base rate (BCIS or in-house, dated by `rate_period`, e.g. 2024Q1) it
rebases the rate through the index, localises it and returns totals at low /
mid / high on-costs. No £/m² benchmark is loaded yet — the tool says
`not_available` rather than inventing one, so always ask the user for a rate.

`commercial_property(uprn | uarn)` (2 credits) is the commercial report:
hereditament (RV, sector, description), the sector rent index across four
lists, the capital index, an `income_basis_value` that capitalises the RV at
the district's commercial auction yield (quote its `yield_basis`, and say
`not_available` when the district has none), auction history, owner, lease.
RV is a rating measure at the antecedent valuation date, not passing rent.

## Strategies, the live lane and assembly (added 7 Sep 2026)

Sourcing is preset-first now. Call `sourcing_strategies(area=<LAD GSS code>)`
FIRST (free): it returns the nine presets with their filters, ranking
components and, per authority, `data_ready`. Then run
`sourcing_search(strategy="short_lease_enfranchisement", lad_code="E09000032")`
(2 credits) — every row comes back with `score_components` saying which term
earned what, and you page with `cursor`, never an offset. Override any
published filter with `strategy_overrides`; a key it does not publish is
rejected, so read the catalogue rather than guessing.

Report `data_ready` honestly. `partial` is usable — say the coverage share
beside the results. Do not run a preset marked `not_ready` without telling
the user which input is missing. If a search answers `features_not_built`,
the nightly feature build has not run: say that, and never present the empty
list as "nothing matches".

`sourcing_live_listings(postcode_districts=[...])` (2 credits) is the
on-market lane: asking against our valuation, yields on both bases, lease
years, EPC gap, days listed, agent, signals. It has NO total by design — page
with `offset` and read `has_more`; never tell a user how many listings exist.

`sourcing_offmarket_owners(lad_code=...)` (2 credits) ranks owners as an
approach list. Companies only, structurally — units held by private
individuals are counted under `not_itemised` and never listed, and you must
not try to identify them another way. Quote `scope.share_itemised`.

`land_assembly(title_number=... | uprn=...)` (2 credits) returns the adjacent
parcels, each owner, and the combined site. A neighbour with no owner is one
no registered title has been linked to, not one nobody owns.

## Development, planning precedent and commercial (added 8 Sep 2026)

`planning_precedents(lat, lon)` or `(lpa_code=...)` (2 credits) is the
decision record nearby: an approval RATE with its denominator, a median
decision time, a breakdown by application type, and examples. TWO
DENOMINATORS — the rate is over approvals plus refusals, the timing over the
~13% of applications carrying both dates. Quote each with its own n. It is a
rate, never a probability: do not turn it into a chance of approval.
Withdrawals and advisory outcomes (observations on another authority's
application, confirmations consent was not required) are excluded and shown
under `excluded_from_rate`. Coverage is the 35 London authorities — elsewhere
an empty answer is a coverage gap, not a quiet planning history.

`site_appraisal(uprn, scheme_gia_m2=..., rate_gbp_m2=...)` now prices the
build: index-rebased, localised by region, with on-costs, as a low/mid/high
range and a cost per unit. You must supply the rate — no benchmark is loaded,
and without one the block reads `not_available` rather than inventing one. It
is CONSTRUCTION COST ONLY: one side of a residual, never a residual. Say so.

`commercial_market(sector, lad_code)` (free) returns the capital index, the
rent index and rated stock. The two indices are NOT comparable — one is
quarterly and thin (check `n_pairs` and `thin_periods`), the other moves once
per rating list. Check `area.index_scope` before calling a series local: only
19 authorities have one, and the national series is substituted and labelled.

`sourcing_commercial(lad_code, sectors=[...])` (2 credits) searches rated
stock. Rateable value is the VOA's estimated annual rental value at the list's
valuation date, NOT passing rent. `income_basis_value` divides it by the
district's auction yield and reads `not_available` where there is none — never
substitute a yield of your own. No £/m² is published and you should not
compute one: the VOA measured area is not loaded.

`parcel_lookup(title_number= | uprn= | inspire_id=)` (1 credit) is one parcel
with its owner, units, plot utilisation and constraints. Constraints are
three-valued — `not_checked` is not `false`, and unbuilt ground is an upper
bound, not a developable area.
