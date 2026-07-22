---
name: flight-scout
description: Research flight availability and pricing on Google Flights via Chrome browser automation. Use when the user asks to find, compare, or price flights between airports, evaluate alternate airports or split itineraries, or set up flight price tracking. Gathers origin/destination/dates/passengers (asks if missing), prices every option, recommends the best, and offers price-tracking alerts.
---

# Flight Scout

Research flight pricing and availability in the user's Chrome via Google Flights, compare all reasonable options, recommend the best one, and offer price-tracking alerts.

## Personal defaults (optional — edit me)

If you (the skill's owner) want standing defaults, fill these in; Claude will use them instead of asking. Leave blank to be asked each time.

- Home airport(s), in order of preference:
- Airports to always exclude:
- Usual party (adults / children+ages / infants lap-or-seat):
- Cabin preference:

## Step 0 — Prerequisites

1. Invoke the `claude-in-chrome` skill first (required before any `mcp__claude-in-chrome__*` tool). This needs the Claude in Chrome extension installed, connected, and granted permission for google.com. If browser tools are unavailable or the user declines permissions, say so and offer a degraded fallback (WebSearch for typical fares) — do not fake results.
2. Load browser tools in ONE ToolSearch call: `tabs_context_mcp, navigate, computer, read_page, tabs_create_mcp, javascript_tool, browser_batch, get_page_text, find`.
3. Get tab context; create a new tab unless the user says to reuse one.

## Step 1 — Gather inputs (ask only for what's missing)

Required — take from the request or the Personal defaults above; ask via AskUserQuestion for anything still missing:
- **Origin airport(s)**: primary + acceptable alternates + any excluded airports.
- **Destination** (airport or city; if the city has multiple airports, ask or search the city code).
- **Dates**: exact dates or candidate windows; if several windows, price each one.
- **Passengers**: adults / children with ages / infants, and lap vs. seat for infants.

Optional (use sensible defaults, don't interrogate): cabin = economy; round trip unless told otherwise; nonstops preferred, but always show best connections when nonstops don't exist or the user asks.

Do not guess airports from vague place names ("the city", "down south") — ask.

## Step 2 — Search method (Google Flights)

Base URL: `https://www.google.com/travel/flights` (append `?gl=XX&hl=xx` matching the user's country/language only if results come back in the wrong locale; prices display in the currency Google chooses for the session — report whatever currency symbol appears, never assume USD).

**Fast path — natural-language q URL** (best for the FIRST search of each route):
`.../travel/flights?q=nonstop%20flights%20from%20AAA%20to%20BBB%20on%20YYYY-MM-DD%20returning%20YYYY-MM-DD`
Also works with "one way ... on DATE". Parses origin/dest/dates/stops filter reliably.

**Passengers**: the q URL always resets to 1 adult. After the first load, click the passenger-count dropdown in the search header, click + on Adults / Children / Infants-in-seat / Infants-on-lap as needed, then Done. The setting then PERSISTS while you edit fields and dates in the UI — so run all subsequent variations by editing the loaded page, not by loading fresh q URLs.

**Changing dates**: use the ‹ › stepper arrows beside each date field (one day per click, auto-reruns the search) or open the calendar, click a day, then Done. After each change wait ~8s before extracting; if extraction returns nothing or the wrong date, the page was mid-load — wait and re-extract.

**Changing airports**: click the field, select-all (cmd+a on Mac, ctrl+a otherwise), type the airport code, wait ~2s for the autocomplete, press Return. GOTCHA: changing the origin on a results page often bounces back to the home form and CLEARS the destination — after any origin change, screenshot, refill "Where to?" if empty, and click Search.

**Locating controls**: never rely on remembered pixel coordinates across page states or window sizes — screenshot first, then click what you see. Layouts also change; if a documented control is missing, screenshot and adapt.

**Extraction**: don't screenshot-read result lists; run javascript_tool over `document.body.innerText`. Find `'Top flights'` (one-way) or `'Top departing flights'` (round trip) — fall back to `'All flights'` — slice ~1500 chars, and capture the `departing YYYY-MM-DD` string from the tracking text to verify which date actually loaded. Collapsed rows include: times, airline, duration, route, stops + layover airport/length, price. Note Google's price-level banner ("prices are typical/low/high") when shown.

**Displayed prices are TOTALS for all passengers in the search**, taxes included, bag fees excluded. Say this in the report.

## Step 3 — What to price

1. Primary origin, each date window: nonstops first. If none exist, say so explicitly (that's a finding, not a failure) and show the best connecting options with layover airport + duration. Flag connections under ~50 minutes as risky — more so with children, strollers, or checked bags.
2. Each allowed alternate origin: same treatment.
3. Split / positioning itineraries (separate one-ways, overnight en route) — only when the user proposes one or asks for creative savings: price every leg on every feasible date; verify the budget carrier actually FLIES each route on each day (many ultra-low-cost routes are not daily — an empty result on one date does not mean the route doesn't exist); add realistic extras (positioning-city hotel, ground transport, one-way car logistics if outbound and return use different home airports); then compare honestly against the simple round trip. Warn: separately ticketed legs carry no missed-connection protection, and ULCC base fares exclude all bags and seat selection.

## Step 4 — Report format

- Lead with the bottom line: cheapest workable option AND your recommendation — which may differ; weigh price against practicality (total travel time, departure-time sanity for the party, connection risk) and say why.
- One table per origin/plan: date window | airline | schedule | duration | stops+layover | total price (with currency).
- Rank the plans and state the price gap between winner and runners-up.
- Caveats: fares move constantly, bag fees and basic-economy restrictions vary by airline, and quotes this far out will change.

## Step 5 — Price tracking alerts

Always offer at the end. If accepted:
- Requires a signed-in Google account (check for the profile avatar top-right). If not signed in, tell the user to sign in themselves first — never handle their credentials.
- On the exact search (route/dates/passengers set), toggle **"Track prices"** on the results page; alerts email the signed-in account. Confirm the toggle flipped via screenshot. Repeat per date window they want tracked.
- This changes their Google account state — flip it only after an explicit yes.

## House rules

- Never book, hold, or purchase; never enter payment, login, or personal data into any site. Research and price-tracking only.
- All reported prices = total for the whole party, in the currency shown on the page; say so.
- If the site blocks a JS read (e.g. "[BLOCKED: ...]"), fall back to UI interaction — don't retry the blocked call.
- If pages stop responding or results won't load after 2–3 tries, stop and tell the user what happened rather than guessing.
- Show progress as you go (one short line per leg searched); deliver the full comparison at the end.
