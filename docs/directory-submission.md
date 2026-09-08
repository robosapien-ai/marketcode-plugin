# Directory submission pack

Everything the ChatGPT plugin portal and the Claude Connectors Directory ask
for, in one place, so the forms can be filled in without re-deriving anything.
Requirements were checked against the two sets of developer docs on
2026-09-04 and the live server on 2026-09-06.

## The server

| | |
|---|---|
| MCP endpoint | `https://mcp.marketcode.ai/mcp` (streamable HTTP) |
| Auth | OAuth 2.1 with PKCE. CIMD and DCR. Issuer `https://api.marketcode.ai`. Protected-resource metadata at `https://mcp.marketcode.ai/.well-known/oauth-protected-resource` |
| Redirects accepted | `https://chatgpt.com/connector_platform_oauth_redirect`, `https://chatgpt.com/connector/oauth/{id}`, `https://claude.ai/api/mcp/auth_callback`, loopback `http://127.0.0.1:{port}/callback` and `http://localhost/callback` (RFC 8252) |
| RFC 9207 `iss` | on every authorization response; advertised in metadata |
| Tools | 83, all read-only. Every tool carries `title` (top-level and `annotations.title`), `annotations.readOnlyHint: true`, `destructiveHint: false`, `openWorldHint: false`, and `_meta["marketcode.ai/credits"]` with its price |
| Discovery | `tools/list` needs no credentials; the tool list is the public catalogue |
| Unpaid call | HTTP 402 whose body links to `https://marketcode.ai/data/mcp#credits`, an information page. The plugin never links to a checkout |

## Listing copy

**Display name** (≤30): `MarketCode UK Property`

**Short description** (≤30): `UK property data and analytics`

**Tagline for Claude** (≤55): `UK property data for valuations, planning and sourcing`

**Long description** (≤2,000, fits both portals):

> MarketCode gives your assistant the UK property record, joined at the
> address: HM Land Registry titles and every sale since 1995, Ordnance Survey
> addresses, planning designations and applications, energy certificates,
> flood zones, council tax bands, company and overseas ownership, live portal
> listings and auction results. On top sit MarketCode's own analytics: a
> valuation model with a range and a confidence score for every UK home, five
> published price and rent indices, transaction volumes, listing stock and
> auction outcomes.
>
> Ask it to value an address and it returns the estimate, the range, the
> comparables and where each figure came from. Ask how a district has moved
> and it returns the index, volumes and stock. Ask what applies to a site and
> it returns every planning designation, recent applications and the flood
> position. Ask who owns a title and it returns the registered proprietor.
>
> Every tool is read-only. Most are free; a call costs credits only where a
> data licence requires it. Sign in with your MarketCode account; new accounts
> start with free credits. Prices are published in every tool description and
> at marketcode.ai/pricing.

**Category**: Data & analytics / Real estate.

## URLs

| | |
|---|---|
| Website | `https://marketcode.ai` |
| Documentation | `https://marketcode.ai/data/mcp` (tools, prices, connection steps per client) |
| Privacy policy | `https://marketcode.ai/privacy` |
| Terms | `https://marketcode.ai/terms` |
| Support | `https://marketcode.ai/contact-us` and `hello@marketcode.ai` |
| Logo (square, 512px) | `https://marketcode.ai/mark-transparent-512.png` |
| Plugin repo (Claude plugin directory) | `https://github.com/robosapien-ai/marketcode-plugin` (MIT) |

## Annotation justification (ChatGPT asks for a sentence)

Every tool reads from MarketCode's property warehouse and writes nothing: no
tool creates, changes or deletes any record, account or setting, and none
sends messages or money. `readOnlyHint` is true and `destructiveHint` false
on all 83. `openWorldHint` is false because every tool answers from
MarketCode's own database rather than the open web.

## Review account

Create it in admin-app → Review Accounts → New review account (door: API/MCP
Hub, 2,000 credits). The page creates the user, sets a password shown once,
provisions the gateway account and grants the credits. The reviewer signs in
with email and password at auth.robosapien.ai when the connector asks; no
mailbox, Google account or MFA is involved. Paste the "For the submission
form" summary the page produces. Retire the password after the review.

## Test cases

Five that should succeed and three that should fail cleanly, using real
tools. Run each once in ChatGPT developer mode and once in claude.ai before
submitting and paste the actual output next to the expectation.

### Should succeed

1. **Resolve an address.** "What is the UPRN for 10 Downing Street, London
   SW1A 2AA?" Expect `address_resolve` to return one UPRN with a match
   confidence and the normalised address. Free.
2. **Value a home.** "Value 12 Example Street, London SW1A 1AA and show me the
   comparables you used." Expect `address_resolve` then `valuation_full`: an
   estimate, a low and high, a confidence band, and comparable sales with
   dates and prices. Free.
3. **Area brief.** "How have sale prices in Hackney moved over the last three
   years, and how many transactions a month?" Expect `area_resolve`,
   `market_index_series` and `market_volume_series`: a monthly index and a
   monthly count for the district. Free.
4. **Planning position.** "What planning designations and constraints apply at
   [a real address]?" Expect `planning_designations`: conservation area,
   listed status, flood zone, green belt and the rest, each true or false with
   its source. Free.
5. **Auction outcomes.** "What was the auction clearance rate in Bristol over
   the last twelve months, and the average discount to estimate?" Expect
   `auction_stats` and `auction_discount` with counts and percentages. Free.

### Should fail cleanly

1. **Outside the UK.** "Value 1600 Pennsylvania Avenue, Washington DC." Expect
   the tool to return a structured error saying only UK addresses are
   supported (the gateway rejects `country != UK` with a 422) and the
   assistant to say so, with no retry loop.
2. **A priced call with no balance.** On an account with zero credits: "List
   every title owned by Tesco Stores Limited." Expect `ownership_by_company`
   to return a 402 whose message names the credit cost and links to
   `marketcode.ai/data/mcp#credits`. The assistant explains the balance and
   offers nothing to buy inside the chat.
3. **A write request.** "Create an API key for me" or "Delete my last
   valuation." Expect no tool call: the server exposes no writing tool, and
   the assistant says the connector is read-only and points to the MarketCode
   account page.

## Checklist

- [x] OAuth 2.1 with CIMD and DCR, ChatGPT and Claude redirect URIs, loopback for Codex
- [x] `title` (top-level and inside `annotations`) and read-only annotations on all 83 tools; `_meta` price on each
- [x] `tools/list` public; discovery and token endpoints answer in under a second
- [x] 402 links to an information page, not a checkout
- [x] Privacy, terms, support pages live; `www.marketcode.ai` resolves
- [x] Claude Code plugin validates (`claude plugin validate`), MIT licence, public repo
- [ ] Review account created (none exists yet: `users.review_password_hash` is NULL for every user)
- [ ] `/.well-known/openai-apps-challenge` served on `mcp.marketcode.ai` with OpenAI's token (set `OPENAI_APPS_CHALLENGE_TOKEN` on the marketcode-mcp Deployment when the portal issues it)
- [ ] The eight test cases above run once in ChatGPT developer mode and once in claude.ai, outputs pasted in
- [ ] Terms and privacy mention credits and the API/MCP explicitly (they mention MCP; they do not mention credits or pricing)
- [ ] Claude Connectors Directory needs a Team or Enterprise claude.ai organisation with the Owner or Directory role
