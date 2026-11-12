---
name: private-jet-search
description: Search private jet charter prices and empty-leg flights with the Mirai Flights MCP tools and give the traveler a booking link. Use when the user asks about chartering a private jet, private flight prices, or empty legs.
---

# Private jet search with Mirai Flights

The `mirai-flights` MCP server searches charter operators and empty-leg flights. It never books: the traveler books by opening the link you give them.

## Places

1. Call `find_location` for every place the user names, and pass the returned `ref` to the other tools. Never invent refs.
2. When several results match (London, UK and London, Canada), pick by country and context, or ask.
3. A city ref searches all its airports; an airport ref searches only that airport. Prefer the city unless the user names an airport.

## Charter

1. Call `search_charter` with one leg per flight: `departure` is local time at the departure airport, `YYYY-MM-DDTHH:mm`. A round trip is two legs. Pass `aircraft_class_id` (from `list_aircraft_classes`) only if the user asks for a class.
2. The search can take up to ~40 seconds. If `complete` is false, call `get_charter_results` with the same `task_id` before saying nothing was found.
3. Options come sorted by price. Show a short list: aircraft type, seats, price, flight time.
4. When the user picks an option, call `get_charter_option` with `task_id`, `suggestion_id` and `company_id`, and show its `link`.

## Empty legs

1. Call `search_empty_legs`; every filter is optional. Empty legs are one-way, fly in a departure window rather than at a fixed time, and are cheaper because the aircraft would otherwise fly empty.
2. When the user picks a flight, call `get_empty_leg` with its `id`, `company_id` and the passenger count, and show the exact price and `link`.

## Language and currency

- Pass `language` (`en`, `ru`, `de`, `es`, `it`, `fr`) matching the conversation.
- Pass `currency` (`EUR` or `USD`) the user asked for, and keep the same currency in every follow-up call of one search; prices default to EUR.

## Answering

- Quote prices exactly as returned, with their currency; never convert them yourself.
- Prices are indicative until the operator confirms on mirai.flights.
- If a tool says the search expired, run `search_charter` again.
