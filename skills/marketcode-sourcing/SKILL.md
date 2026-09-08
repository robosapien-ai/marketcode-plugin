---
name: marketcode-sourcing
description: Find land or units matching investment criteria with the MarketCode tools, score and shortlist them, resolve the registered owners and produce a recipient list ready for outreach. Covers off-market sourcing over ownership, planning and the built stock, long-held high-equity owners, and distressed auction stock. Use when the user wants to find sites, plots, buildings or units by criteria, or asks who owns what.
module: sourcing
---

# Site and unit sourcing

You are producing a scored shortlist of land parcels or property units that
match the user's criteria, with the registered owner resolved for each and a
recipient list the user can take into outreach. The deliverable is markdown
tables plus a CSV block.

Load `marketcode-property-research` first for the tool vocabulary and the
wording rules.

## Credit budget (state it before you start)

| Call | Credits | Needed |
|---|---|---|
| `sourcing_search` | 8 | always, once per criteria set |
| `sourcing_owners` | 5 | for the agreed shortlist |
| `long_held_high_equity` | 8 | when the criterion is hold length and gain |
| `site_appraisal` | 5 each | for the two or three top candidates only |
| `owner_profile` | 5 | to see a company's other holdings |
| `ownership_by_company`, `ownership_by_title` | 3 | to confirm a title |
| `plot_utilisation`, `mixed_use_buildings` | 6 | alternative land screens |
| `distressed_assets`, `auction_lots`, `valuation_full`, `equity_estimate`, `planning_designations`, `market_facts` | 0 | context |

Typical run: 13 to 30 credits. Quote the number, then proceed.

## Inputs

The asset class (`land` or `units`), the area, and the criteria. Ask for
anything missing in one message; do not call `sourcing_search` until the
area and asset class are confirmed. Multi-district asks go in ONE call with
a comma-separated `location`, never one call per district.

## Criteria vocabulary for `sourcing_search`

- **Land** (`asset="land"`): `landuse` (a land-use tier such as
  residential, commercial, vacant; never "brownfield", which is a register
  flag, not a use), `max_built_coverage`, `tenure`, `owner_types`,
  `exclude_owner_types`, `min_years_owned`, `company_status` (e.g.
  `["dissolved"]`), `min_score`, `min_rlv`, `opportunities_only`,
  `include_undevelopable`. Score = undeveloped share, hold length, owner
  type, corporate distress.
- **Units** (`asset="units"`): `property_types`, `min_beds`, `max_beds`,
  `epc_in` (`["F","G"]` for MEES pressure), `tenure`, `owner_types`,
  `min_years_owned`, `company_status`, `min_score`. Score = hold length (no
  modern sale record scores highest), owner type, distress, EPC, uplift
  since purchase.
- One-click recipes worth offering: underdeveloped land over 0.5 ha;
  distressed corporate owners; long-held 15 years or more; EPC F and G
  units; units not sold in 15 years, where never-traded counts.

Every row carries a cheap, deterministic Tier-0 value (an indicative
residual land value for parcels, the MarketCode estimate and equity since
purchase for units) and an opportunity flag. Rank and triage on those.
Run `site_appraisal` or `valuation_full` only on the two or three the user
shortlists; label Tier-0 numbers "indicative".

## Phase A: search and shortlist

1. `sourcing_search` with the confirmed criteria. Present the top rows as a
   table: id, address or title, area or floor area, score, Tier-0 value,
   signals (undeveloped share, years owned, owner type, company status,
   EPC), opportunity flag.
2. Refine once if the list is too long or too thin: relax or tighten one
   criterion and say which. If a filter returns a `filter_warning`, relay it
   and offer criteria that work.
3. Agree the shortlist (up to 25) before spending on owners.

## Phase B: owners

4. `sourcing_owners(parcel_ids, asset)` with the `id` values from the search
   result, never title numbers. Company and public-body owners come back
   with company number, tenure and title number; individually owned titles
   return no bulk match because the free registers exclude individuals.
   Report the match tier honestly.
5. For company owners the user cares about, `owner_profile(company_number)`
   for their other holdings, and `ownership_by_company` to confirm.
6. For individually owned parcels, the recipient is "The Owner" at the
   property address (the occupier route). Say that the official copy from
   HM Land Registry is the paid way to name them, and that MarketCode does
   not buy it for you.

## Phase C: deliverable

- **Shortlist** table (above) with one line of thesis per candidate.
- **Owners** table: parcel, owner name, type, company number, status,
  registered office, title number, match tier.
- **Recipients** as a CSV block: parcel_id, address, recipient_name,
  recipient_address, source (land_registry | companies_house | occupier),
  complete (yes/no). Prefer complete recipients.
- **Next steps**: what a letter should reference per parcel (planning
  position, hold length, company status) so it is site-specific rather than
  mail-merge; where to check designations (`planning_designations`) before
  approaching; and the GDPR note below.

## Rules

- Do not search live portals or scrape listings from this skill. If the user
  wants on-market stock, use `listing_stock_series` for the market picture
  and point them at the portals; MarketCode's tools are registers and
  analytics, not a listings feed.
- Personal data: registered-owner data comes from public registers under
  licence; letters to individuals must state the source and offer an
  opt-out. Say so once.
- Never claim a buyer or funding exists in draft outreach unless the user
  said so.
- Cite the tool for every figure; label Tier-0 values as indicative.

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

## Change, reports and saved searches (added 8 Sep 2026)

`property_report(uprn)` and `area_report(lad_code, postcode_district)` (5
credits each) are the composed documents. ALWAYS read `sections_degraded`
before summarising — a section that could not answer says which KIND of
absence it is, and reporting a document as complete when two sections failed
is the thing to avoid. For `area_report`, pass BOTH a LAD GSS code and a
postcode district: readiness, commercial and planning are keyed on the
authority, the live listing lane on the district.

`property_rental_estimate(uprn)` (1 credit) always carries a `basis`. Quote it.
`area_asking` is a locality median applied to one property — give the p25–p75
range with it. The yield is GROSS. If `plausibility` reads `implausible`, the
area cell and the property are not comparable and the yield is withheld: say
so rather than reaching for the number.

`property_energy_retrofit(uprn)` (1 credit): MEES is a legal bar on letting,
not a discount. The measures list is not available and must not be replaced
with a generic one.

`changes_since(cursor=...)` (1 credit) is the portal change feed. The cursor is
an event id, not a time. `added` and `price_reduced` are live; `delisted` has
one row in the entire table, so withdrawal is NOT observed — an empty result
there is not evidence that nothing was withdrawn. Planning decisions, ownership
changes, lease crossings, new EPCs and AVM moves have no feed at all.

**Saved searches are the only tools that WRITE.** `saved_search_create`,
`_update` and `_delete` change the user's account and `_run` advances a stored
cursor. Confirm before creating or deleting, and name the search rather than
its id when you do. Managing them is free; only `_run` spends, and it spends
what the underlying search costs. `_update` REPLACES params — send the whole
object, or filters the user thinks they kept will be dropped.
