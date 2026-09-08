---
name: marketcode-planning-check
description: Check the planning position of a UK site with the MarketCode tools: every designation and constraint at the address, graded by severity; recent and decided applications nearby; the flood position; and what it means for a stated proposal or application type, with the documents the proposal will likely need. Use for "can I build here", "what are the constraints", "is it in a conservation area", or the first pass before a planning application. Every call is free.
module: sourcing
---

# Planning check

You are producing the planning position of a site: what constrains it,
what has been decided around it, and what that means for the user's
proposal. It is the first pass a planner does before advice; it is not
planning advice, and the report says so.

Load `marketcode-property-research` first for the tool vocabulary and the
wording rules.

## Credit budget

`planning_designations`, `planning_check`, `planning_applications`, `property_flood_risk`,
`epc_certificates` and `transactions_by_uprn` cost 0. `property_summary`
costs 4 and is optional (for the existing building). `address_resolve`
costs 8 if the user typed an address. Say so, then proceed.

## Inputs

The site (address, postcode or UPRN). Optionally the proposal: application
type (householder extension, full residential, change of use, HMO, listed
building consent, prior approval) and a one-line description. Without a
proposal, deliver the position and say which proposals it would bear on.

## Phase A: gather (parallel)

1. `planning_designations(postcode or location)`: thirteen checks in one call; `planning_check(checks=[...])` when only some are needed, or for SSSI and agricultural land grade, which the bundle does not include. Both free.
   `planning_designations`: thirteen checks in one
   call (conservation area, listed buildings, Article 4, TPO, green belt,
   AONB, national park, flood zones, SSSI, ancient woodland, scheduled
   monuments, brownfield register, heritage at risk).
2. `planning_applications(postcode or location)`: recent applications
   with type, status and decision.
3. `property_flood_risk(uprn)`: surface-water position, which the
   designations' fluvial and tidal zones do not cover.
4. Optional: `property_summary(uprn)` for the existing building (type,
   year, floor area, tenure) and `epc_certificates(uprn)` where energy
   measures are part of the proposal.

## Phase B: grade and interpret

Grade each present constraint:

| Grade | Meaning | Typical |
|---|---|---|
| BLOCKER | development very unlikely | green belt, functional floodplain |
| HIGH | significant extra requirements | conservation area, AONB, Grade I or II* nearby |
| MEDIUM | some extra requirements | flood zone 2, Grade II, TPO |
| LOW | minor considerations | low flood risk |

Then the site's development sensitivity: very high (a blocker), high
(several HIGH or green belt), medium (one HIGH or a conservation area),
standard.

For the nearby applications: cluster by type, note the approval and
refusal pattern, and pull out any decision that is a direct precedent for
the user's proposal (same type, same street or neighbours).

## Phase C: what it means for the proposal

Map constraints to consequences in plain English, for the stated
application type:

- Conservation area: design and materials scrutiny; permitted development
  rights reduced; a heritage statement is usually needed.
- Listed building or setting: listed building consent for works to fabric;
  a heritage statement; the setting matters for neighbours' listings too.
- Article 4: the permitted development route the user may be counting on
  is withdrawn for the direction's classes; check which.
- TPO: any works within the root protection area need consent; an
  arboricultural report.
- Flood zone 2 or 3: sequential and exception tests; a flood risk
  assessment; vulnerable uses at or below flood level are the issue.
- Green belt: very special circumstances; expect refusal for new build.
- AONB or national park: landscape impact; major development resisted.
- Brownfield register entry: a positive signal for residential; note the
  register's permission-in-principle route.

Documents the proposal will likely need, as a checklist keyed to the
constraints above (planning statement; design and access statement for
major or conservation-area schemes; heritage statement; flood risk
assessment; arboricultural report; ecology where SSSI or ancient woodland
is near). Mark each: needed, probably needed, not needed.

## Report (markdown)

1. **Position in one sentence** and the sensitivity grade.
2. **Constraints** table: designation, present or not, grade, source.
3. **Flood** position: fluvial, tidal, surface water.
4. **Nearby decisions**: the cluster summary and the precedents.
5. **For your proposal**: consequences and the document checklist.
6. **Next steps**: pre-application advice with the local planning
   authority where sensitivity is high; the portal to check; what to
   commission first.
7. **Sources**: one line per tool.

## Rules

- Distinguish "the authority almost always accepts X" from "X is
  compliant"; you have the constraints, not the policy text.
- If a likely refusal is visible, say so and suggest a pivot (reduce
  scope, add mitigation, seek pre-app).
- Do not cite policy numbers you have not read; name the policy area
  instead and tell the user where the local plan lives.
- This is a desk check, not planning advice; recommend the local planning
  authority's pre-application service for anything graded HIGH or above.

## Planning history at the property (added 7 Sep 2026)

`property_planning_history(uprn)` (1 credit) is the history AT the address:
applications linked to the unit or its building by the mart's entity links,
each with decision, dates, appeal, CIL liability, development type and a
`match_level` (`unit`, `building`, or `nearby` within 25 m when nothing is
linked — say which). `consent_context` reads the designation flags
three-valued: report `applies` as constraints and `unchecked` as unknown,
never as clear. `coverage.covered` is false outside the 33 London boroughs
today; `status: not_covered` is a different answer from `no_applications`,
and it is not charged. The `_batch` twin takes up to 100 UPRNs.

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
