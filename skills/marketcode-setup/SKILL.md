---
name: marketcode-setup
description: Connect and sign in to the MarketCode MCP server after installing this plugin. Use when MarketCode tools are missing, return an authentication error, or the user asks how to connect MarketCode.
---

# Connecting MarketCode

This plugin declares one remote MCP server, `https://mcp.marketcode.ai/mcp`.
It uses OAuth 2.1 sign-in. There is no API key to paste.

## First use

1. Run `/mcp` and choose **marketcode**, then **Authenticate**.
2. A browser opens on the MarketCode sign-in. Sign in with an email link,
   Google or Microsoft. A new account is created on first sign-in with 100
   free credits.
3. Approve the request. The scope is `property:read`: read-only property data.
   The connection cannot touch keys, billing or account settings.
4. Back in the client, the MarketCode tools appear. Tool descriptions state
   their credit cost.

## If a tool returns an authentication error

The access token lasts one hour and refreshes automatically. If a call still
returns "Authentication required", run `/mcp` and authenticate again.

## Disconnecting

Sign in at https://marketcode.ai/api-hub/dashboard/connections and remove the
connection under Connected apps. An access token already issued keeps working
for up to an hour.

## Do not

- Do not put an `mc_live_...` API key in this plugin's configuration or in an
  `Authorization` header for the MCP server. API keys are for the REST API at
  https://api.marketcode.ai and leak from client config files.
