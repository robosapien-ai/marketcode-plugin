# MarketCode plugin

UK property data inside your assistant. This plugin connects the
[MarketCode MCP server](https://marketcode.ai/data/mcp) and bundles two skills:
one that teaches the model how to research property with the tools, and one
that walks through sign-in.

## What you get

53 read-only tools over MarketCode's UK property warehouse: address and UPRN
resolution, property records, sold-price history, ownership, planning
designations, valuations, auction outcomes, energy data, market indices and
volumes, and a governed query layer. Every tool states its credit cost. Most
are free; the priced ones return licensed content (OS AddressBase, HM Land
Registry corporate and overseas registers) or run a model.

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

ChatGPT, Codex, claude.ai and Cursor connect to the same server without this
plugin. Steps for each are at https://marketcode.ai/data/mcp#connect.

## Privacy, terms, support

- Privacy policy: https://marketcode.ai/privacy
- Terms: https://marketcode.ai/terms
- Support: https://marketcode.ai/contact-us

## Licence

MIT. The plugin manifest and skills are open source; the MCP server and the data
behind it are MarketCode's.
