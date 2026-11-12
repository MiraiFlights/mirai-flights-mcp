<img src="assets/icon.svg" alt="Mirai Flights" width="72" height="72">

# Mirai Flights MCP server

[Mirai Flights](https://mirai.flights) is a private aviation booking service. This free, public MCP server lets any AI assistant search private jet charter prices from operators worldwide and discounted empty-leg flights, then hand the traveler a link that opens the same offer on mirai.flights or in the Mirai app, where the booking happens.

Use the hosted endpoint. Nothing needs to be installed, and there is no account, sign-up or API key.

```text
https://mcp.mirai.flights/mcp
```

## Install

- **Claude (claude.ai and Claude Desktop):** Customize → Connectors → **+ Add** → **Add custom connector**, enter a name and `https://mcp.mirai.flights/mcp`, choose **No sign in** under Authentication, then turn the connector on in a chat from **+** → Connectors. On Team and Enterprise plans an owner adds it under Organization settings → Connectors.
- **ChatGPT:** turn on Developer Mode (Settings → Apps & Connectors → Advanced), create a connector with the URL above and **No authentication**, then enable it in a chat from the **+** menu. Custom connectors need a paid ChatGPT plan.
- **Claude Code:**

  ```bash
  claude mcp add --transport http mirai-flights https://mcp.mirai.flights/mcp
  ```

- **Claude Code plugin** (the same server, plus a skill that tells Claude how to run a search end to end):

  ```bash
  claude plugin marketplace add MiraiFlights/mirai-flights-mcp
  claude plugin install mirai-flights@mirai-flights
  ```

- **Cursor:** add to `~/.cursor/mcp.json`:

  ```json
  { "mcpServers": { "mirai-flights": { "url": "https://mcp.mirai.flights/mcp" } } }
  ```

- **VS Code (GitHub Copilot):**

  ```bash
  code --add-mcp '{"name":"mirai-flights","type":"http","url":"https://mcp.mirai.flights/mcp"}'
  ```

- **Any other client** that supports remote MCP servers over Streamable HTTP: point it at the endpoint and leave authentication empty.

## What you can ask

- "Find a private jet from Nice to London next Friday at noon for 4 people."
- "Any empty legs out of Dubai this week for 2 passengers?"
- "Price a round trip Geneva → Mykonos on 3 July, back on 10 July, 6 passengers, heavy jets only."

## Tools

| Tool | What it does |
|---|---|
| `find_location` | Finds cities and airports by name or IATA/ICAO code and returns a `ref` for each. Every other tool takes places as refs. A city ref covers all its airports. |
| `search_charter` | Prices a charter route of 1–5 legs (local departure time, passengers, optional luggage and aircraft class). Operators answer for up to ~40 seconds; progress is streamed, and all options come back sorted by price. |
| `get_charter_results` | Collects more options for a search that returned `complete: false`. |
| `get_charter_option` | Details of the option the traveler picked, and its booking link. |
| `search_empty_legs` | Searches empty-leg flights by route, dates, passengers, price and aircraft class. All filters are optional. |
| `get_empty_leg` | Details of one empty leg, the exact price for a given number of passengers, and its booking link. |
| `list_aircraft_classes` | Aircraft classes (light, midsize, heavy…) with example types, for filtering. |

A typical flow is `find_location` → `search_charter` or `search_empty_legs` → `get_charter_option` or `get_empty_leg` for the flight the traveler chooses. Booking links are created only in that last step.

## Connection details

| | |
|---|---|
| Endpoint | `https://mcp.mirai.flights/mcp` |
| Authentication | None. |
| Transport | Streamable HTTP, `POST` only, stateless. No `Mcp-Session-Id` is issued. `GET` returns `405`. |
| Protocol versions | `2026-07-28`, `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. |
| Languages | `en`, `ru`, `de`, `es`, `it`, `fr`. Each tool takes an optional `language`; without it the server uses English when the client's `Accept-Language` allows it, otherwise the first supported language listed there. |
| Currencies | `EUR` (default) or `USD`, per call via `currency`. |
| Rate limits | Fair-use limits apply per client. Requests over the limit get HTTP `429`; charter searches over the limit return a tool error asking to retry in a minute. |
| Payments | None. No tool books or takes payment; booking happens on mirai.flights. |
| Privacy policy | https://mirai.flights/privacy |
| Terms of service | https://mirai.flights/terms |
| Support | info@mirai.flights |

## Repository contents

This repository holds the public listing and client packages for the hosted server; the server itself runs at the endpoint above.

- `server.json`: entry for the [official MCP Registry](https://registry.modelcontextprotocol.io).
- `.claude-plugin/marketplace.json` and `plugins/mirai-flights`: Claude Code plugin with the server and a usage skill.

## License

MIT, see [LICENSE](LICENSE).
