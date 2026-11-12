<img src="assets/icon.png" alt="Mirai Flights" width="72" height="72">

# Mirai Flights MCP server: private jet charter and empty leg search for AI assistants

Let Claude, ChatGPT, Cursor or any MCP client price a private jet charter, find discounted empty leg flights and hand the traveler a booking link, all from a chat. Free, hosted, no account or API key.

```text
https://mcp.mirai.flights/mcp
```

[Mirai Flights](https://mirai.flights) is a fully digital private aviation broker, launched in 2019 and used by clients from 45 countries in 2025. Travelers book a business jet on mirai.flights or in the Mirai app at a fixed price calculated at the time of booking.

## Why use it

- **Live charter prices.** A search asks operators about the exact route, date and group size and returns priced options by aircraft type, cheapest first, usually within seconds.
- **Empty legs up to 75% below the regular charter price.** One-way repositioning flights you can filter by route, dates, group size, price and aircraft class.
- **100+ aircraft models,** from light jets for short hops to heavy long-range jets.
- **From question to booking in one conversation.** The offer the traveler picks comes with a link that opens the same flight and price on mirai.flights or in the Mirai app.
- **Multi-leg trips.** Round trips and itineraries of up to five legs in one search.
- **Six languages, two currencies.** English, Russian, German, Spanish, Italian and French; prices in EUR or USD.

## Try it

Ask your assistant things like:

- "Find a private jet from Nice to London next Friday at noon for 4 people."
- "What does a heavy jet from Dubai to Geneva cost on 20 December for 8 passengers?"
- "Price a round trip Geneva → Mykonos on 3 July, back on 10 July, 6 passengers."
- "Any empty legs out of Miami this week for 2 people under $10,000?"
- "Найди частный самолёт из Ниццы в Женеву на завтра на троих."

## Install

No installation, no keys. Add the endpoint to your client:

| Client | How |
|---|---|
| **Claude** (claude.ai, Claude Desktop) | Customize → Connectors → **+ Add** → **Add custom connector**, enter `https://mcp.mirai.flights/mcp`, choose **No sign in**, then turn it on in a chat from **+** → Connectors. On Team and Enterprise plans an owner adds it under Organization settings → Connectors. |
| **ChatGPT** | Turn on Developer Mode (Settings → Apps & Connectors → Advanced), create a connector with the URL above and **No authentication**, then enable it in a chat from the **+** menu. Needs a paid plan. |
| **Claude Code** | `claude mcp add --transport http mirai-flights https://mcp.mirai.flights/mcp` |
| **Claude Code plugin** (server plus a skill that walks Claude through search and booking) | `claude plugin marketplace add MiraiFlights/mirai-flights-mcp`, then `claude plugin install mirai-flights@mirai-flights` |
| **Cursor** | Add `{ "mcpServers": { "mirai-flights": { "url": "https://mcp.mirai.flights/mcp" } } }` to `~/.cursor/mcp.json` |
| **VS Code** (GitHub Copilot) | `code --add-mcp '{"name":"mirai-flights","type":"http","url":"https://mcp.mirai.flights/mcp"}'` |
| **Any MCP client** | Streamable HTTP at the URL above, no authentication. |

## How a booking happens

1. **Find places.** `find_location` turns "Nice" or "LHR" into a city or airport reference. A city covers all its airports.
2. **Search.** `search_charter` prices the route; `search_empty_legs` lists matching empty legs.
3. **Pick.** The traveler chooses an option; `get_charter_option` or `get_empty_leg` returns its details and a booking link.
4. **Book.** The traveler opens the link and confirms the flight on mirai.flights or in the Mirai app.

The assistant never books or pays on the traveler's behalf.

## Tools

| Tool | What it does |
|---|---|
| `find_location` | Finds cities and airports by name or IATA/ICAO code and returns a `ref` for each. Every other tool takes places as refs. |
| `search_charter` | Prices a charter route of 1–5 legs: local departure time, passengers, optional luggage and aircraft class. Progress is streamed while operators answer; options come back sorted by price. |
| `get_charter_results` | Collects more options for a search that returned `complete: false`. |
| `get_charter_option` | Details of the charter option the traveler picked, and its booking link. |
| `search_empty_legs` | Searches empty legs by route, dates, passengers, price and aircraft class. All filters are optional. |
| `get_empty_leg` | Details of one empty leg, the exact price for the group, and its booking link. |
| `list_aircraft_classes` | Aircraft classes (light, midsize, heavy and more) with example types, for filtering. |

## FAQ

**Is it free?** Yes. Searching is free and anonymous; travelers pay only for the flight they book with Mirai Flights.

**Does the assistant book the flight?** No. It finds and prices flights and gives a link; the traveler books on mirai.flights or in the Mirai app.

**How accurate are the prices?** Charter prices come from operators at the time of the search, and the fixed price is confirmed when the traveler books. `get_empty_leg` recalculates an empty leg price for the exact number of passengers.

**Where does it work?** Worldwide: any city or airport that `find_location` returns.

**Which languages and currencies?** `en`, `ru`, `de`, `es`, `it`, `fr`, and `EUR` or `USD`. Tools take `language` and `currency` per call; without `language` the server follows the client's `Accept-Language`, preferring English.

## Connection details

| | |
|---|---|
| Endpoint | `https://mcp.mirai.flights/mcp` |
| Authentication | None. |
| Transport | Streamable HTTP, `POST` only, stateless. No `Mcp-Session-Id` is issued. `GET` returns `405`. |
| Protocol versions | `2026-07-28`, `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. |
| Languages | `en`, `ru`, `de`, `es`, `it`, `fr`. |
| Currencies | `EUR` (default), `USD`. |
| Rate limits | Fair-use limits apply per client. Requests over the limit get HTTP `429`; charter searches over the limit return a tool error asking to retry in a minute. |
| Payments | None. No tool books or takes payment. |
| Privacy policy | https://mirai.flights/privacy |
| Terms of service | https://mirai.flights/terms |
| Support | info@mirai.flights |

## Repository contents

This repository holds the public listing and client packages for the hosted server; the server itself runs at the endpoint above.

- `server.json`: entry for the [official MCP Registry](https://registry.modelcontextprotocol.io).
- `.claude-plugin/marketplace.json` and `plugins/mirai-flights`: Claude Code plugin with the server and the `private-jet-search` skill.

## License

MIT, see [LICENSE](LICENSE).
