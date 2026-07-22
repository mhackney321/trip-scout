---
name: trip-scout
description: Plan trips end-to-end via Chrome browser automation and web research - flights (Google Flights first, airline sites direct when needed), lodging of any kind (hotels, resorts, all-inclusives, vacation rentals/Airbnb - via Google or the property's own site), and optional activity/restaurant itineraries. Use when the user asks to find, compare, or price flights or places to stay, evaluate alternate airports or split itineraries, set up price tracking, or plan what to do at a destination. Gathers origin/destination/dates/party/budget/style (asks if missing), recommends destinations when the user is undecided, prices every option, tracks total budget across phases, recommends the best, and offers price alerts and a day-by-day itinerary.
---

# Trip Scout

Research a trip in the user's Chrome: flights, lodging of any kind, and (opt-in) an activity + restaurant itinerary. Compare options, recommend, offer price tracking.

Run the phases in order, but only the ones the trip needs: flights-only requests stop after Phase 1; "plan my trip" runs all three. Each phase ends with a short report before the next begins.

## Core principle — Google first, direct sites when needed

Google Flights / Google Hotels are the fast, comparable STARTING POINT — not the boundary of the search. They are aggregators with coverage gaps: some airlines, resorts, and all Airbnb listings don't appear there. Absence from a Google surface is never proof something doesn't exist or isn't available. Go directly to an airline's, resort's, rental platform's, or hotel's own website (in the same Chrome session) whenever:

- The user names a specific carrier, property, or platform that doesn't show in Google results.
- A route/property is known or suspected to be served by companies missing from the aggregator.
- Google shows no availability but the finding matters — confirm on the operator's own booking engine before reporting "unavailable".
- You're about to recommend something — direct sites can have promos, member rates, bundle pricing, or restrictions (age limits, occupancy rules, fare rules) the aggregator hides.

On direct sites use the same techniques (fill the search form, extract from page text, screenshot to locate controls) and the same safety rules (no logins, no payments, no personal data). Report which source each price came from.

## Parse the request first

Before any phase, extract these fields from the user's message (and `defaults.local.md`); ask in one question only for required fields still missing:

- **Destination**: country/city/airport. If undecided ("somewhere warm", "not sure where"), don't stall — ask for their criteria (climate, vibe, flight-time tolerance, budget) in one question, recommend 2–3 candidate destinations with a sentence of reasoning each, let them pick, then proceed.
- **Duration & dates**: how many days; departure/return dates or candidate windows. Flexible dates = price multiple windows.
- **Party**: solo/couple/family/friends + headcount; children's ages; infants lap-or-seat.
- **Budget** (optional): total trip budget or level (Budget/Standard/Luxury). Feeds every phase: flight-price ceiling, lodging tier, activity costs. If a total is given, track the running spend across phases and flag when a recommendation would blow it.
- **Travel style** (optional): relaxation/adventure/cultural/culinary — steers lodging type and the itinerary without re-asking in Phase 3.
- **Special requests** (optional): must-visit places, exclusions (airports, airlines, neighborhoods, activities), accessibility needs. Treat exclusions as hard constraints, never suggestions.

Don't re-ask anything already extracted; later phases reuse these fields.

## Personal defaults (optional)

Before asking the user for inputs, check for a `defaults.local.md` file next to this SKILL.md and read it if present — it holds the owner's standing preferences (home airports, excluded airports, usual party, cabin, lodging taste, interests/dietary notes) and is gitignored so it never gets published. Use whatever it provides instead of asking; ask only for what's still missing. To set it up, copy `defaults.example.md` to `defaults.local.md` and fill it in.

## Phase 0 — Prerequisites

This skill needs an assistant that can drive the user's Chrome browser (navigate, click, type, screenshot, run page JavaScript) and search the web. Set up whatever your environment requires before starting. In Claude Code specifically: invoke the `claude-in-chrome` skill first (required before any `mcp__claude-in-chrome__*` tool), which needs the Claude in Chrome extension installed, connected, and granted permission for google.com; load the browser tools in one ToolSearch call (`tabs_context_mcp, navigate, computer, read_page, tabs_create_mcp, javascript_tool, browser_batch, get_page_text, find`); get tab context and create a new tab unless the user says to reuse one. In other environments, use the equivalent browser-automation tooling.

If browser automation is unavailable or the user declines permissions, say so and offer a degraded fallback (web search for typical prices) — do not fake results.

Wherever this skill says "ask in one question", use your environment's structured-question tool if it has one (AskUserQuestion in Claude Code); otherwise just ask in chat.

## Phase 1 — Flights

### Inputs (ask only for what's missing)

Destination, dates, and party come from "Parse the request first". Still needed here:
- **Origin airport(s)**: primary + acceptable alternates + any excluded airports.
- **Destination airport**: if the destination city has multiple airports, ask or search the city code.

Optional (use sensible defaults, don't interrogate): cabin = economy; round trip unless told otherwise; nonstops preferred, but always show best connections when nonstops don't exist or the user asks.

Do not guess airports from vague place names ("the city", "down south") — ask.

### Search method — Google Flights first, airlines direct when needed

Base URL: `https://www.google.com/travel/flights` (append `?gl=XX&hl=xx` matching the user's country/language only if results come back in the wrong locale; report prices in whatever currency the page shows — never assume USD).

Per the core principle: if a carrier the user cares about is missing from results, if a low-cost or regional airline is suspected to serve the route unlisted, or if Google shows nothing on a route that should exist — search the airline's own website directly (destination pages usually publish route maps and operating days). Also verify the fare on the airline's site before the user books, since direct prices and bundles can differ. Note per-airline gotchas honestly (e.g. some ULCCs sell bags/seats only at booking).

**Fast path — natural-language q URL** (best for the FIRST search of each route):
`.../travel/flights?q=nonstop%20flights%20from%20AAA%20to%20BBB%20on%20YYYY-MM-DD%20returning%20YYYY-MM-DD`
Also works with "one way ... on DATE". Parses origin/dest/dates/stops filter reliably.

**Passengers**: the q URL always resets to 1 adult. After the first load, click the passenger-count dropdown in the search header, click + on Adults / Children / Infants-in-seat / Infants-on-lap as needed, then Done. The setting then PERSISTS while you edit fields and dates in the UI — so run all subsequent variations by editing the loaded page, not by loading fresh q URLs.

**Changing dates**: use the ‹ › stepper arrows beside each date field (one day per click, auto-reruns the search) or open the calendar, click a day, then Done. After each change wait ~8s before extracting; if extraction returns nothing or the wrong date, the page was mid-load — wait and re-extract.

**Changing airports**: click the field, select-all (cmd+a on Mac, ctrl+a otherwise), type the airport code, wait ~2s for the autocomplete, press Return. GOTCHA: changing the origin on a results page often bounces back to the home form and CLEARS the destination — after any origin change, screenshot, refill "Where to?" if empty, and click Search.

**Locating controls**: never rely on remembered pixel coordinates across page states or window sizes — screenshot first, then click what you see. Layouts also change; if a documented control is missing, screenshot and adapt.

**Extraction**: don't screenshot-read result lists; run javascript_tool over `document.body.innerText`. Find `'Top flights'` (one-way) or `'Top departing flights'` (round trip) — fall back to `'All flights'` — slice ~1500 chars, and capture the `departing YYYY-MM-DD` string from the tracking text to verify which date actually loaded. Collapsed rows include: times, airline, duration, route, stops + layover airport/length, price. Note Google's price-level banner ("prices are typical/low/high") when shown.

**Displayed prices are TOTALS for all passengers in the search**, taxes included, bag fees excluded. Say this in the report.

### What to price

1. Primary origin, each date window: nonstops first. If none exist, say so explicitly (that's a finding, not a failure) and show the best connecting options with layover airport + duration. Flag connections under ~50 minutes as risky — more so with children, strollers, or checked bags.
2. Each allowed alternate origin: same treatment.
3. Split / positioning itineraries (separate one-ways, overnight en route) — only when the user proposes one or asks for creative savings: price every leg on every feasible date; verify the budget carrier actually FLIES each route on each day (many ultra-low-cost routes are not daily — an empty result on one date does not mean the route doesn't exist); add realistic extras (positioning-city hotel, ground transport, one-way car logistics if outbound and return use different home airports); then compare honestly against the simple round trip. Warn: separately ticketed legs carry no missed-connection protection, and ULCC base fares exclude all bags and seat selection.

### Flight report

- Lead with the bottom line: cheapest workable option AND your recommendation — which may differ; weigh price against practicality (total travel time, departure-time sanity for the party, connection risk) and say why.
- One table per origin/plan: date window | airline | schedule | duration | stops+layover | total price (with currency).
- Rank the plans and state the price gap between winner and runners-up.
- Caveats: fares move constantly, bag fees and basic-economy restrictions vary by airline, and quotes far out will change.

## Phase 2 — Lodging (skip if the user already has it)

"Lodging" means anywhere the user rests: hotels, resorts, all-inclusives, vacation rentals (Airbnb/Vrbo-style homes and condos), B&Bs, hostels. Ask once: "Do you need a place to stay, or is lodging handled?" If handled, skip to Phase 3.

### Inputs

Ask (in one question, only for what's missing): **lodging type** (multiSelect: hotel · resort/all-inclusive · vacation rental/Airbnb · no preference), nightly budget or total, star/quality level, area or landmark to stay near (if they know the destination), must-haves (multiSelect: pool, breakfast, kitchen, free cancellation, family rooms/cribs, parking, accessibility).

### Where to search (by lodging type)

- **Hotels, resorts, all-inclusives** → `https://www.google.com/travel/hotels`. Resorts are indexed here; use the amenity/class filters and the "All-inclusive" filter when relevant.
- **Vacation rentals** → Google's "Vacation rentals" tab (sibling of the Hotels tab, or `https://www.google.com/travel/search?...` — click over from Hotels). This covers Vrbo/Booking-listed homes. NOTE: **Airbnb does not list on Google** — if the user wants Airbnb specifically, search `airbnb.com` directly in the browser: set destination, dates, guests (adults/children/infants), use the price-display toggle for totals, and extract listing name, type, total price, rating/review count, and location.
- Search every type the user selected; "no preference" = hotels/resorts + vacation rentals side by side.
- **Any specific property or platform the user names** (a resort, a chain, Vrbo, Booking.com, a boutique's own site) → search it directly, whether or not it appears on Google. Per the core principle, Google absence ≠ unavailable: property booking engines are the ground truth for availability, room categories, and restrictions.

### Site playbooks (field-tested)

**Google Hotels** (`google.com/travel/hotels`):
- The q-URL works here too (`?q=resorts%20in%20Cancun`) and even auto-applies category filters from the query — but it defaults to ~tomorrow's dates, 1 night, 2 adults. ALWAYS set dates and party before reading any prices.
- Date fields accept TYPED dates: click the field, select-all, type e.g. `Wed, Jan 6 2027` — jumps the calendar instantly, far faster than clicking through months. GOTCHA: after typing one date, calendar clicks can REPLACE check-in instead of setting checkout. Reliable sequence: click the check-in field → click its day → click the check-out field → click its day → verify BOTH dates show in the header → Done.
- Adding a child defaults its age to 12 — always set the real age (the age dropdown scrolls; younger ages are above the fold). Ages change pricing.
- Displayed prices are NIGHTLY. Get stay totals via the "What you'll pay" control or the property page before comparing.
- Extraction anchor: search innerText for `N results`. Each card yields: name, nightly price, deal badges ("22% less than usual"), rating, review count, star class, amenities list.

**Google Vacation Rentals**: it's the sibling tab next to Hotels — switching tabs PRESERVES dates and party. Cards yield name, nightly price, rating/reviews, sleeps/bedrooms/baths, sq ft, and useful tags like "Kid-friendly" and "Beach access".

**Airbnb** (no login needed for search): build the URL directly —
`https://www.airbnb.com/s/<City>--<Country>/homes?checkin=YYYY-MM-DD&checkout=YYYY-MM-DD&adults=N&children=N&infants=N`
- Prices display as all-fees-included TOTALS for the stay ("Prices include all fees"), unlike Google's nightly figures — don't mix the two up when comparing.
- A one-time modal ("Now you'll see one price...") may block the page — dismiss it ("Got it") before interacting.
- innerText is NOISY: repeated "Prices include all fees" lines and duplicated accessibility text — dedupe when extracting. Cards yield: type ("Condo in Cancún"), title, rating + review count, bedrooms/beds/baths, strikethrough original vs. deal price, "for 7 nights", "Free cancellation", "Guest favorite" badge.
- Filter chips (Crib, Pool, Kitchen, Free cancellation...) sit above results — use them for must-haves.

### Method (all sources)

- Set the SAME dates and party as the chosen flight option (children's ages matter for room pricing).
- Apply the user's filters via the UI (price slider, star rating, amenities). Sort or scan for best value.
- Extract with the same innerText technique. Capture for each candidate: name, type, nightly + TOTAL price (with currency), rating and review count, area/landmark distance, cancellation policy if shown.
- Compare totals, not nightly rates — rentals add cleaning/service fees, hotels add taxes/resort fees. Use each site's "total price" toggle where offered and say what's included.
- Cross-check the top 2–3 candidates with a quick WebSearch for red flags (resort fees, location issues, recent complaints).
- **Resorts**: also check the property's own booking site — direct promos, member rates, or suite-category perks often beat aggregator prices, and some room categories (e.g. adults-only sections, age restrictions) only show their rules there. Flag any occupancy/age restrictions against the user's party.
- Shortlist 3–5 options across the price range and types, recommend one, and give the trip subtotal (flights + lodging) for the winning combination.
- Offer price tracking where the site shows a track/alert option, same consent rules as flights.

## Phase 3 — Activities & itinerary (opt-in)

After flights/hotels are settled, ask exactly one simple question: "Want me to research things to do in <destination> and sketch a day-by-day itinerary?" If no, stop.

### Quick interview

If yes, ask in one question (multi-select where noted):
1. **Interests** (multiSelect): food & drink · beaches/outdoors · culture/museums/history · adventure/sports · nightlife · shopping · kid-friendly · relax/spa.
2. **Restaurants**: splurge-worthy dining · solid local spots · quick & cheap · mix; plus note any cuisines or dietary restrictions in the option descriptions and let "Other" catch specifics.
3. **Pace**: packed days · one anchor activity per day · mostly unstructured.

### Research

- Use WebSearch (and WebFetch for promising pages) — recent sources only; prefer local/official tourism pages, recent reviews, and current opening hours. Verify anything seasonal against the actual travel dates (closures, weather, festivals, hurricane/rainy season, wildlife seasons — e.g. recommending a whale-shark tour outside its season is a classic failure).
- Query patterns that work well (field-tested): `<destination> things to do with <party type>` · `<specific area> day trip <constraint>` (e.g. "with baby", "ferry", "wheelchair") · `family friendly restaurants <area> <current year> local food`. Search the party's actual constraints, not just the destination.
- Keep the sources — the itinerary must cite them as markdown links so the user can verify details and book.
- Match to the party: don't propose all-day hikes for a group with a lap infant; note stroller/accessibility issues; flag activities needing advance booking and typical prices.
- Restaurants: 2 per day max in the plan (lunch/dinner), matched to stated prefs, near that day's activities; note reservation difficulty.

### Itinerary output

- Day-by-day for the actual trip dates, starting after the flight's real arrival time and ending before departure (Day 1 after a long flight = light; departure day = nothing that risks the flight).
- Each day: morning / afternoon / evening, one line each, with neighborhood clustering so the day doesn't zigzag; estimated costs where known.
- Mark which items need booking ahead, with links.
- Keep it a suggestion, not a schedule — include a "swap ideas" list of 3–5 alternates.
- End with a rough total trip cost: flights + hotel + estimated activities/food.

## Price tracking alerts (flights, and hotels where offered)

Always offer at the end of the relevant phase. If accepted:
- Requires a signed-in Google account (check for the profile avatar top-right). If not signed in, tell the user to sign in themselves first — never handle their credentials.
- On the exact search (route/dates/passengers set), toggle **"Track prices"** on the results page; alerts email the signed-in account. Confirm the toggle flipped via screenshot. Repeat per date window they want tracked.
- This changes their Google account state — flip it only after an explicit yes.

## House rules

- Never book, hold, or purchase; never enter payment, login, or personal data into any site. Research and price-tracking only.
- All reported prices = total for the whole party, in the currency shown on the page; say so.
- If the site blocks a JS read (e.g. "[BLOCKED: ...]"), fall back to UI interaction — don't retry the blocked call.
- If pages stop responding or results won't load after 2–3 tries, stop and tell the user what happened rather than guessing.
- Show progress as you go (one short line per leg/hotel/topic searched); deliver the full comparison at the end of each phase.
