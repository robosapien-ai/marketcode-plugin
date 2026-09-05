# MarketCode plugin

UK property data inside your assistant, and the skills that turn it into
deliverables. This plugin connects the
[MarketCode MCP server](https://marketcode.ai/data/mcp) and bundles eight
skills: a research method, a sign-in guide, and six that each produce a
piece of work a property professional would otherwise commission.

## What you get

Read-only tools over MarketCode's UK property warehouse: address and UPRN
resolution, property records, sold-price history, ownership, planning
designations, valuations, auction outcomes, energy data, market indices and
volumes, listing stock, and a governed query layer. The live count and
every price are published at https://marketcode.ai/data/mcp; most tools are
free, including valuations. The priced ones return licensed content (OS
AddressBase, HM Land Registry corporate and overseas registers) or let you
shape the query.

## Skills

A skill is a markdown file that teaches the model which tools to call, in
what order, at what cost, and how to lay out the result with its sources.
The same files render at https://marketcode.ai/skills.

| Skill | Produces |
|---|---|
| `marketcode-property-research` | The method for any UK property question, and the wording rules |
| `marketcode-setup` | A connected, signed-in assistant |
| `marketcode-valuation` | An indicative valuation report: six methods, reconciled, with a stance-driven range |
| `marketcode-area-insights` | A market brief for any UK area against its neighbours |
| `marketcode-sourcing` | A scored shortlist of land or units with owners resolved and a recipient list |
| `marketcode-deal-analysis` | A first-pass appraisal of a site or scheme with sensitivities and a recommendation |
| `marketcode-due-diligence` | A due diligence report with cross-reference findings and a traffic light |
| `marketcode-planning-check` | The planning position of a site and what it means for a proposal |

`skills/index.json` is the machine-readable index of the same list.

## Install

Claude Code:

```
/plugin marketplace add robosapien-ai/marketcode-plugin
/plugin install marketcode@marketcode
/mcp   # choose marketcode, then Authenticate
```

The server uses OAuth 2.1 sign-in. There is no API key to paste. First sign-in
creates a MarketCode account with 100 free credits.

## Other clients

Claude, ChatGPT, Codex and Cursor connect to the same server without this
plugin; steps for each are at https://marketcode.ai/data/mcp#connect. The
skills are plain markdown: paste the one you need into the client's project
instructions, a custom GPT, or a rules file.

## Privacy, terms, support

- Privacy policy: https://marketcode.ai/privacy
- Terms: https://marketcode.ai/terms
- Support: https://marketcode.ai/contact-us

## Licence

MIT. The plugin manifest and skills are open source; the MCP server and the data
behind it are MarketCode's.
