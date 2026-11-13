# GADUIN MCP connector

Remote, read-only [Model Context Protocol](https://modelcontextprotocol.io) server for live transport status: flights, airports, airlines, train stations, maritime chokepoints, and prediction markets on transport outcomes.

> **This repository holds connector metadata and installation docs only.** It is not the server's source code. The server is hosted by GADUIN at the URL below; there is nothing to clone, build or run locally.

| | |
|---|---|
| Endpoint | `https://gaduin.com/mcp` |
| Transport | Streamable HTTP |
| Access | Read-only. 12 public tools work with no key and no sign-in. |
| Optional auth | `Authorization: Bearer <api-key>` unlocks 2 account tools (positions, balance). Get a key at <https://gaduin.com/account/ai>. |
| Metadata | [`server.json`](server.json) (MCP registry descriptor) |
| Website | <https://gaduin.com/agents> |
| Support | support@gaduin.com |

No tool places trades, moves money or changes anything. Data is informational, not advice.

## Install

Every example connects straight to the remote endpoint. Leave out the `Authorization` header unless you want the two account tools.

### Claude Code

Per the [Claude Code MCP docs](https://code.claude.com/docs/en/mcp#option-1-add-a-remote-http-server):

```bash
claude mcp add --transport http gaduin https://gaduin.com/mcp
```

With an API key for the account tools:

```bash
claude mcp add --transport http gaduin https://gaduin.com/mcp \
  --header "Authorization: Bearer YOUR_GADUIN_API_KEY"
```

Check the connection with `claude mcp get gaduin`.

### Claude (claude.ai, Claude Desktop, Cowork)

Per [Get started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp) (individual plans; Team and Enterprise owners add it under **Organization settings > Connectors**):

1. Open **Customize > Connectors**.
2. Click **+ Add**, then **Add custom connector**.
3. Name: `GADUIN`. Remote MCP server URL: `https://gaduin.com/mcp`. Click **Continue**.
4. Under **Authentication**, choose **No sign in**.
5. Optional, for the account tools: under **Request headers**, add `Authorization` with the value `Bearer YOUR_GADUIN_API_KEY`.
6. Click **Add**, then enable it in a chat from **+ > Connectors**.

Claude reaches custom connectors from Anthropic's cloud, not from your device.

### Cline

Per the [Cline MCP docs](https://docs.cline.bot/mcp/mcp-overview):

**IDE extension, Remote Servers tab:** open **MCP Servers > Remote Servers**, enter Server Name `gaduin`, Server URL `https://gaduin.com/mcp`, Transport Type **Streamable HTTP**, then **Add Server**.

**IDE extension or CLI, JSON settings** (`cline_mcp_settings.json`; for the CLI it is `~/.cline/data/settings/cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "gaduin": {
      "type": "streamableHttp",
      "url": "https://gaduin.com/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Set `"type": "streamableHttp"` explicitly; Cline treats a remote entry without `type` as legacy SSE. For the account tools, add:

```json
      "headers": {
        "Authorization": "Bearer YOUR_GADUIN_API_KEY"
      },
```

**Cline CLI:**

```bash
cline mcp add gaduin https://gaduin.com/mcp --transport http --yes
```

### Other MCP clients

Any client that supports remote Streamable HTTP servers can use `https://gaduin.com/mcp` directly. To check the endpoint from a terminal:

```bash
curl -s https://gaduin.com/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Tools

All tools are read-only.

### Public (no key)

| Tool | What it returns |
|---|---|
| `get_flight_status` | Live status of a flight by number (e.g. `BA117`): times in UTC and local time, delay, last update, link to its page. |
| `get_flight_history` | How often a flight ran late over the last 30 days: on-time, late and cancelled counts. |
| `search_flights` | Today's departures from an airport still to leave, optionally to one destination or on one airline. |
| `get_airline_stats` | An airline's last 30 days: on-time share, cancellations, diversions, median and 90th-percentile delay. |
| `get_airport_status` | How an airport is running today: status, disruption score, delayed and cancelled shares for departures and arrivals. |
| `list_disrupted_airports` | Today's most disrupted airports, worst first. |
| `get_station_board` | Live departures and arrivals at a railway station by name or code: times, platform, status, delay. |
| `list_chokepoints` | Every maritime chokepoint with a published daily ship count, furthest below normal first. |
| `get_chokepoint_status` | Ship traffic through one chokepoint: recent daily mean against normal, weekly and daily counts. |
| `list_markets` | Prediction markets on transport outcomes, filtered by status, category and free text. |
| `get_market` | One market: question, settlement rule, outcomes with current prices, status. |
| `get_market_price_history` | How a market's prices moved over its life. |

### Account (needs `Authorization: Bearer <api-key>`)

| Tool | What it returns |
|---|---|
| `get_my_positions` | Your open positions across all markets. |
| `get_balance` | Your current balance. |

Keyless clients see only the public tools in `tools/list`.

## Prompts

Server version 0.3.0 adds six prompts for clients that support MCP prompts:

| Prompt | Arguments |
|---|---|
| `check_flight`: Is my flight delayed? | `flight` |
| `find_flights`: Flights between two airports today | `from`, `to` |
| `check_airport`: Is this airport delayed today? | `airport` |
| `disrupted_airports`: Which airports are disrupted today? | none |
| `check_station`: Are trains running late at my station? | `station` |
| `check_chokepoint`: Is this shipping chokepoint open? | `chokepoint` |

## Example questions

- "Is BA117 delayed today, and how often has it been late this month?"
- "Which flights from MAD to Lisbon are still to leave today?"
- "Which airports are most disrupted right now?"
- "Are trains running late at London Paddington?"
- "How does ship traffic through the Suez Canal compare with normal?"
- "Which prediction markets are open on flights from JFK?"

## Support

Email support@gaduin.com.
