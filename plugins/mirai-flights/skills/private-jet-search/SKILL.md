---
name: private-jet-search
description: Price private jet charters and find empty leg flights with the Mirai Flights MCP tools, then give the traveler a booking link. Use when the user asks about chartering a private or business jet, private flight prices, jet charter quotes, flying privately, or empty legs.
---

# Private jet search with Mirai Flights

The `mirai-flights` MCP server prices private jet charters with operators and lists empty leg flights. It never books: the traveler books by opening the link you give them, on mirai.flights or in the Mirai app. Your job is to get them from a question to the right offer and its link with as few back-and-forths as possible.

## 1. Get the trip

You need: from, to, date, approximate departure time, and number of passengers. Ask only for what is missing, in one short question. If the user gives a date but no time, assume 12:00 local and say so. A round trip is two legs; multi-city trips take up to five.

For every place, call `find_location` and pass the returned `ref` to the other tools. Never invent refs.

- Prefer the city ref: it searches all the city's airports and finds more aircraft. Use an airport ref only when the user names an airport.
- When several places match (London, UK and London, Canada), pick by country and context, or ask.

## 2. Search

**Charter:** call `search_charter`. `departure` is local time at the departure airport, `YYYY-MM-DDTHH:mm`. Pass `aircraft_class_id` (from `list_aircraft_classes`) only if the user asks for a class. Operators answer for up to ~40 seconds; if `complete` is false, call `get_charter_results` with the same `task_id` before concluding.

**Empty legs:** call `search_empty_legs`. Every filter is optional. Empty legs are one-way repositioning flights, up to 75% below the regular charter price, but they fly on the operator's route and in a departure window.

**Cross-sell honestly:** for a one-way trip where the user is price-sensitive or flexible on dates, also check `search_empty_legs` on the same route and nearby dates, and mention a match as a cheaper alternative. Don't push empty legs on a fixed round trip.

## 3. Present

- Options come sorted by price. Show three to five, not the whole list: the cheapest, a good-value step up, and the most comfortable that fits the group. For each: aircraft type, seats, flight time, price.
- Quote prices exactly as returned, with their currency. Never convert or round them. When an option has `discount_percent`, mention the discount and `price_before_discount`.
- Empty leg prices in a search are the minimum per jet; the price for the group comes from `get_empty_leg`.
- If your client shows the Mirai Flights view, the traveler already sees the cards with photos: summarize and recommend instead of repeating every option.
- Say that the charter price is fixed when the trip is booked, and that the final confirmation happens on mirai.flights.
- Offer the next step: "Want the booking link for one of these?"

## 4. Close

When the user picks an option:

- Charter: call `get_charter_option` with `task_id`, `suggestion_id` and `company_id`.
- Empty leg: call `get_empty_leg` with `id`, `company_id` and the passenger count, and show the exact price for the group.

Show the `link` prominently, for example **[Book this flight on Mirai Flights](link)**, with the aircraft, route, date and price next to it, so the traveler can confirm at a glance. If the link is empty, point them to https://mirai.flights.

## When nothing fits

Don't stop at "no results". Offer the next best move:

- the city ref instead of a single airport, or a nearby city;
- a different time of day, or the day before or after;
- a smaller or larger aircraft class;
- empty legs on the route, if the dates are flexible.

If a tool says the search expired, run `search_charter` again.

## Language and currency

- Pass `language` (`en`, `ru`, `de`, `es`, `it`, `fr`) matching the conversation.
- Pass the `currency` (`EUR` or `USD`) the user asked for, or `USD` when they talk in dollars, and keep it in every follow-up call of the same search. Prices default to EUR.
