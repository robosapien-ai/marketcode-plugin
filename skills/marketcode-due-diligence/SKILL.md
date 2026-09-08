---
name: marketcode-due-diligence
description: Produce a property due diligence report for one UK address with the MarketCode tools: title and ownership, sales and listing history, EPC and council tax, flood and designations, nearby planning activity, cross-source disagreements, and the named cross-reference findings that join the dots (short lease, conservation area with a low EPC, flood risk with a basement). Use for "due diligence", "what should I check before I buy", "red flags on this property".
module: valuation
---

# Property due diligence report

You are producing a structured due diligence report a buyer, lender or
adviser can act on. The value is in joining sources, not listing them:
each triggered cross-reference rule becomes a finding with what it means
and what to check next. The deliverable is markdown with a traffic light.

Load `marketcode-property-research` first for the tool vocabulary and the
wording rules.

## Credit budget (state it before you start)

| Call | Credits | Needed |
|---|---|---|
| `address_resolve` | 8 | if the user typed an address |
| `property_lookup` | 5 | always (300 fields with source and confidence) |
| `property_reconcile` | 3 | always (where sources disagree) |
| `ownership_by_title` | 3 | when the lookup gives a title number |
| `listing_history` | 3 | when the property has been marketed |
| `building_physical` | 3 | flats and converted buildings |
| `transactions_by_uprn`, `property_history`, `epc_certificates`, `council_tax_band`, `property_flood_risk`, `planning_designations`, `planning_applications`, `valuation_full` | 0 | always |

Typical run: 11 to 25 credits.

## Inputs

The property. Optionally the tenure if known and the buyer's purpose
(owner-occupier, landlord, lender), which changes which findings matter.

## Phase A: gather (parallel)

1. `property_lookup(uprn)` for the full record. Read `*_source` and
   `*_confidence`; the tenure, title number, year built, floor area and
   type feed everything below.
2. `property_reconcile(uprn)`: the fields where sources disagree, with each
   source's value. Every disagreement is a finding.
3. `transactions_by_uprn(uprn)` and `property_history(uprn)`: sales and
   listing timeline; `listing_history(uprn)` if it has been marketed.
4. `epc_certificates(uprn)`: rating history, floor area, heating, the
   recommendations.
5. `council_tax_band(uprn)`.
6. `property_flood_risk(uprn)` and `planning_designations(postcode)`.
7. `planning_applications(postcode)`: what is happening around it.
8. `ownership_by_title(title_number)` when the lookup returns a title
   number; corporate and overseas proprietors resolve, individuals do not
   (the free registers exclude them; say so).
9. `building_physical([uprn])` for a flat: storeys, units, form.
10. `valuation_full(uprn)` for the estimate, its range and confidence, so
    the buyer knows where the price sits.

## Phase B: cross-reference rules

Apply each rule; a triggered rule is a finding with a paragraph: what, why
it matters, what to check.

1. **LEASE_SHORT**: leasehold with an unexpired term under 80 years
   (from the lookup where present; otherwise flag the term as unknown and
   name the official copy as the source to obtain).
2. **CONSERVATION_LOW_EPC**: conservation area and EPC below D: external
   insulation and glazing limits constrain retrofit.
3. **LISTED_EPC**: listed building and any EPC measure: consent required.
4. **PRE1919_LOW_EPC**: built before 1919 and EPC below D: sympathetic
   retrofit scope.
5. **FLOOD_BASEMENT**: any flood risk and a lower-ground or basement level.
6. **ARTICLE4_PD**: Article 4 direction and permitted-development style
   applications nearby: precedent and enforcement risk.
7. **PRICE_ABOVE_ESTIMATE**: asking or agreed price more than 10% above the
   estimate range: the buyer is paying for something the data does not see.
8. **SOURCE_DISAGREEMENT**: `property_reconcile` flags on floor area,
   bedrooms, tenure or type: the marketing may be wrong.
9. **LONG_HOLD_NO_MODERN_SALE**: no registered sale since 1995: expect
   older title documents and possibly unregistered land.
10. **CORPORATE_OR_OVERSEAS_OWNER**: proprietor is a company (check status)
    or an overseas entity: extra checks on authority to sell.

## Phase C: nearby activity

Tag each nearby application: impact (positive, negative, neutral),
category (residential, commercial, infrastructure, mixed), status.
Summarise the cluster in two sentences.

## Report (markdown)

1. **Cover**: address, UPRN, tenure, date, and the traffic light with one
   sentence: GREEN proceed, AMBER proceed with conditions, RED do not
   proceed without resolution.
2. **Executive summary**: three to five bullets of what matters most, with
   any missing data stated plainly.
3. **Sections**: Title and tenure · Sales and listing history · Energy and
   council tax · Flood and environment · Planning designations · Nearby
   activity · Cross-reference findings · Price against the estimate.
4. **Recommendations**: practical next steps (obtain the official copy and
   lease, commission a survey, ask the seller for X). Never legal advice
   phrasing; "worth checking", "we would recommend".
5. **Sources**: one line per tool with the fields used; name the checks
   MarketCode cannot do (local land charges, lease terms, searches) so the
   conveyancer's list is complete.

## Rules

- Lead every finding with what it means for the buyer, then the data.
- If a tool returned nothing, say so in the section; never silently skip.
- Every figure cites its tool and field.

## Lease and risk (added 7 Sep 2026)

`property_lease(uprn)` (1 credit) is the governing registered lease with its
receipt: unexpired years today, `is_short_lease` under 80 years, the under-60
flag, the enfranchisement and valuation notes; `status: no_lease` with the
reason when the register holds none. `property_risk_report(uprn)` (1 credit)
is every designation and hazard flag THREE-VALUED with its source — report
`not_checked` and `defaulted_false` as exactly that, never as "clear". The
`_batch` twins take up to 100 UPRNs at 1 credit per property answered.

## Planning history and the commercial report (added 7 Sep 2026)

`property_planning_history(uprn)` (1 credit): the applications AT the
address with decision, dates, appeal and CIL, plus `consent_context`
(three-valued — `unchecked` is not clear) and `coverage` (London boroughs
only today; `not_covered` is free and is not "no applications").
`commercial_property(uprn | uarn)` (2 credits) for a rated unit: RV and
sector, the four-list rent index, an income-basis value with its yield
receipt, auction history, owner and lease; blocks that cannot be derived say
`not_available` with the reason.

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
