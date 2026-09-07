---
name: marketcode-valuation
description: Produce an indicative valuation report for one UK residential address with the MarketCode tools. Six-method triangulation (subject's own sale, bedroom-matched £/sqft, latest same-building sale, building £/sqm fingerprint, ceiling comp, rental sanity check) reconciled against the MarketCode estimate, with a stance-driven range and every figure cited to the tool it came from. Use when the user asks "what is this worth", wants a valuation or marketing-price recommendation, or wants to check an agent's quote.
module: valuation
---

# Valuation report

You are producing a customer-ready **indicative** opinion of value for one UK
residential property. It is a desk-based estimate, not a RICS Red Book
valuation, and the report says so. The method is a six-method triangulation
reconciled against the MarketCode estimate; the output is markdown the user
can paste, print or hand to a client.

Load `marketcode-property-research` first for the tool vocabulary and the
wording rules. This skill adds the method.

## Credit budget (state it before you start)

| Call | Credits | Needed |
|---|---|---|
| `address_autocomplete` then `address_resolve` | 0 then 8 | only if the user gave text, not a UPRN |
| `property_summary` | 4 | always |
| `property_comps` | 6 | always |
| `listing_history` | 3 | when the property has been marketed |
| `comps_adjustments` | 2 | when the reader needs the working behind the estimate: each comp's sale, its index adjustment to today, relisting penalty and the IQR fence |
| `subject_transaction_anchor`, `time_adjustment`, `valuation_accuracy` | 0 | anchor the subject's own sale to today; carry any price to today; quote our published forward-validated error (`no_rows_yet` until October 2026 — say so, never a made-up figure) |
| `valuation_full`, `transactions_by_uprn`, `property_history`, `epc_certificates`, `council_tax_band`, `property_flood_risk`, `planning_designations`, `market_index_series`, `market_facts`, `equity_estimate` | 0 | always |

Typical run: 10 to 23 credits. Say so in one line, then proceed. Never ask
permission again mid-run.

## Inputs

Required: the property (address or UPRN) and a **stance**: `low`, `mid`
(default) or `high`. Optional hints that change the answer materially:
floor area, bedrooms, floor level, outside space, internal condition, an
agent's quote. If the user did not give a stance, use `mid` and say so. Do
not stop to ask for optional hints; reconcile them from the data and list
the gaps under Risks.

## Phase A: evidence (call these in parallel where you can)

1. **Identify.** `address_autocomplete` to get candidates (free), then
   `address_resolve` if the user typed an address. Confirm the UPRN once in
   prose. Never invent a UPRN.
2. **Subject.** `property_summary(uprn)`: type, tenure, beds, floor area,
   EPC, last sale, building. Then `epc_certificates(uprn)` for the declared
   floor area and rating history, `council_tax_band(uprn)`.
3. **Estimate.** `valuation_full(uprn)`: point, range, confidence, rental
   estimate and the back-series re-based to the published index. This is
   sanity check S6 and the time-adjustment source.
4. **Subject's own sales.** `transactions_by_uprn(uprn)` for M0.
   `property_history(uprn)` for the merged sale and listing timeline
   (bedrooms on the listing beat inferred bedrooms).
5. **Comparables.** `property_comps(uprn)`: same-building and street sales
   with floor area and bedrooms backfilled. This feeds M1 to M4.
6. **Market.** `market_index_series` for the district (pass `area_code` and
   `area_type` exactly as `property_summary` or `area_resolve` returns
   them) for the time-adjustment factor; `market_facts` for the area's
   gross yield and price distribution by bedroom band.
7. **Context.** `planning_designations(postcode)`, `property_flood_risk(uprn)`,
   `listing_history(uprn)` if marketed, `equity_estimate([uprn])`.

Read `*_source` and `*_confidence` on every field you use. Where two
sources disagree (floor area is the usual one), say which you used and why.

## Phase B: the six methods

Work at the confirmed (or assumed, and labelled) floor area and bedroom
count. Every method shows an equation, its inputs and one sentence of
reasoning. Numbers without arithmetic are not acceptable.

**M0 · Subject's own sale, time-adjusted.**
`m0 = last_sale_price × (index_now / index_at_sale)` from
`transactions_by_uprn` and `market_index_series`. Treat M0 as a **strong
anchor** when the sale is within 60 months, a standard arms-length sale, and
within ±30% of the building's £/sqm at the time; weight it 2× if under 24
months, 1× otherwise. Otherwise quote it as **diagnostic** and exclude it
from the central; the gap between M0 and the comp cluster is itself the
finding (related-party transfer, wrong floor area, refurbishment since, or
an area repricing the index under-corrected). If M0 is a strong anchor and
diverges from the M1 to M4 mean by more than 50%, stop and resolve which is
wrong before recommending a price.

**M1 · Bedroom-matched £/sqft × subject sqft.** From `property_comps` rows
with the same bedroom count and floor area within ±30% of the subject.
Bedroom matching is mandatory: the all-bed median sits 5 to 8% above the
bedroom-matched median in mixed blocks.

**M2 · Most recent same-bed sale, time-adjusted.**
`comp_£/sqm × subject_sqm × time_factor`. Time-adjust anchors older than 24
months using the index drift between the sale month and now; quote the
drift and the date span.

**M3 · Building £/sqm fingerprint.** From non-outlier comps with declared
floor area: n, median, P25, P75, min, max. Choose the percentile by stance
(below) and multiply by subject sqm.

**M4 · Ceiling £/sqft.** The highest non-outlier £/sqft in the building ×
subject sqft. Name any feature of the ceiling sale the subject lacks.

**S5 · Rental sanity check (quoted, not in the central).**
`rent_pcm × 12 ÷ gross_yield`, rent from `valuation_full`, yield from
`market_facts` for the same bedroom band. Note the circularity: the yield
is derived from observed price and rent pairs. Within ±15% of the central is
reassurance; beyond that, say what is special about the subject.

**S6 · MarketCode estimate (quoted, not in the central).** The point and
range from `valuation_full`. If it diverges from the bottom-up central by
more than 20%, explain which you trust and why: usually a floor-area
disagreement. Treat it as a floor or a ceiling, never ignore it.

**Outliers.** Exclude penthouses, combined units, distressed and non-market
transfers from the fingerprint, and any method value more than 2× any other.
List every exclusion and its reason.

## Stance

- `low`: P25 £/sqm, the higher-yield end of the observed distribution, fully
  time-adjusted comps.
- `mid`: median £/sqm, median yield, half time-adjusted comps.
- `high`: P75 £/sqm, the lower-yield end, un-time-adjusted recent comps.

Lower yield means higher capital value, so `high` uses the low yield.

## Range and recommended price (deterministic; show the working)

`mean` = equal-weighted mean of the independent methods (M0 when strong,
M1, M2, M3, M4) after outlier exclusion.

`size_premium` = the subject's position in the building's same-bed floor
area distribution, as a fraction capped to [-0.10, +0.10]; 0 if the floor
area was assumed.

Multiplier: `low` 0.95 + size_premium/2 · `mid` 1.02 + size_premium/2 ·
`high` 1.07 + size_premium.

- Recommended marketing price = mean × multiplier, rounded to £5,000.
- Expected achieved = mean × (multiplier − 0.04).
- Range = mean × max(multiplier − 0.12, 0.85) to mean × min(multiplier + 0.05, 1.20).

Write the sentence out: "Four independent methods cluster at £A to £B with a
mean of £M (rental sanity £S5, MarketCode estimate £S6). On a MID stance with
the subject at P-x of the building's same-bed size distribution, the
multiplier is 1.02 + z = MM. Recommended marketing price £R, expected
achieved £E, range £L to £H."

## Report (markdown, in this order)

1. **Summary** with the recommended price, expected achieved, range,
   confidence and one paragraph of reasoning.
2. **Subject** table: address, UPRN, type, tenure, beds, floor area (source
   and confidence), EPC, council tax band, last sale.
3. **Methodology** and stance, in plain English.
4. **Comparables** table: address, date, price, £/sqm, beds, sqft, anchor
   or outlier and why.
5. **Six methods** table, then one paragraph per method with the equation.
6. **Reconciliation**: mean, S5, S6, divergences and what you did about them.
7. **Market context**: index trend for the district, yield band, anything
   from the flood or planning position that bears on value.
8. **Risks**: the five gaps ranked by £ impact (floor area, floor level,
   condition, lease term, outside space).
9. **Disclosures**: indicative desk-based opinion, not a Red Book
   valuation; a formal valuation by an RICS Registered Valuer should be
   commissioned for a transaction; data as at the date of the tool calls.
10. **Sources**: one line per tool called, with the fields used.

## Rules

- Every number in the report traces to a tool call and a field. Say "from
  `property_comps`, 7 same-building sales, 4 with declared floor area".
- Quote the range, never only the point. Label assumed floor areas.
- If the user gave an agent's quote more than 50% above the building
  benchmark, say so plainly and show which method would have to be wrong for
  the quote to hold.
- Commercial subject (the summary's classification is commercial): the same
  method applies with the rental yield method promoted to the central and the
  commercial indices from `commercial_index_series`; say that the residential
  comps logic does not apply.
- Do not attempt to render a PDF. The markdown is the deliverable; the user's
  client renders it.
