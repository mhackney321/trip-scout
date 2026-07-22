# trip-scout — a Claude Code skill

Turns Claude Code + the Claude in Chrome extension into a trip-research assistant: it researches in your own browser — starting with Google Flights and Google Hotels for fast comparison, then going directly to airline, resort, hotel, and rental sites (Airbnb, Vrbo, property booking engines) whenever the aggregators don't cover something — prices every reasonable option, recommends the best ones, can set price-tracking alerts, and — if you want — researches your destination and sketches a day-by-day itinerary.

## What it does

**Flights**
- Asks for origin(s), destination, dates, and passenger mix — or uses defaults you hard-code.
- Checks nonstops first; falls back to connections with layover details when nonstops don't exist.
- Compares alternate airports and creative split itineraries honestly (including hotel nights, bag fees, and missed-connection risk).
- Reports totals for your whole party in the currency Google shows, ranked, with a clear recommendation.

**Lodging** (skipped if yours is handled) — hotels, resorts, all-inclusives, and vacation rentals (Google Vacation Rentals / Vrbo listings, or Airbnb directly on request)
- Asks lodging type, budget, area, and must-haves once, then shortlists 3–5 options for your exact dates and party — comparing true totals (taxes, resort fees, cleaning fees), cross-checking red flags, and checking resorts' own sites for direct-booking deals and room restrictions.

**Itinerary** (opt-in)
- One quick interview (interests, restaurant style, pace), then a day-by-day plan built around your real arrival/departure times, clustered by neighborhood, with booking-ahead flags and a rough total trip cost.

**Price tracking**
- Offers to toggle Google "Track prices" alerts on flights (and hotels where available) — requires you to be signed in to Google.

It never books, holds, pays, or enters personal data. Research only.

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI or desktop app)
- Google Chrome with the **Claude in Chrome** extension installed and connected
- Extension permission granted for google.com

## Install

Copy this folder into your skills directory:

- Personal (all your projects): `~/.claude/skills/trip-scout/`
- Or per-project (shared with a repo's users): `<repo>/.claude/skills/trip-scout/`

New Claude Code sessions will pick it up automatically.

## How to trigger it

Two ways:

**1. Explicitly** — type the slash command in any Claude Code session:

```
/trip-scout
```

Claude will ask for whatever it needs (origins, destination, dates, party) and take it from there. You can also pass the trip in the same breath:

```
/trip-scout BOS to Lisbon, May 12-19, 2 adults
```

**2. Naturally** — just describe what you want; the skill triggers on trip-shaped requests. Examples that work:

| You say... | What runs |
|---|---|
| "Find me flights from BOS to Lisbon in May" | Flights phase |
| "What's the cheapest week to fly ORD → Tokyo this fall?" | Flights across date windows |
| "Compare flying out of Austin vs. Dallas to Denver, March 3–8" | Multi-airport comparison |
| "Would it be cheaper to fly to Fort Lauderdale first and hop to the Bahamas?" | Split-itinerary analysis |
| "Find us an all-inclusive resort in Punta Cana, first week of February, 2 adults + 2 kids (6 and 9)" | Lodging phase |
| "Find an Airbnb in Lisbon near Alfama under $200/night" | Lodging via Airbnb direct |
| "Plan our whole trip: flights from JFK, somewhere warm in January, hotel and things to do" | All three phases |
| "What should we do for 5 days in Kyoto with a toddler?" | Itinerary phase |
| "Set up price alerts on those dates" | Price tracking |

Tips: give your party up front ("2 adults + 1 lap infant") for accurate totals, and name any airports you refuse to use — Claude treats exclusions as hard constraints. A budget ("keep it under $4k total", or just "budget/standard/luxury") gets tracked across flights, lodging, and activities. Undecided on where to go? Say so — "somewhere warm in January, short flight from NYC" gets you 2–3 recommended destinations to pick from before any searching starts. If you skip details, Claude asks instead of guessing.

## Customize (optional)

Copy `defaults.example.md` to `defaults.local.md` and fill in your standing preferences (home airports, airports to avoid, usual party, cabin, lodging taste, interests). Claude reads it before asking questions. The file is gitignored, so your personal details never end up in a fork or pull request.

## Example output

A real run (family of 2 adults + 1 lap infant, Northeast US → Cancun, 7 nights):

> **Bottom line:** JetBlue JFK round trip, 10:30 AM out / afternoon return, **$864 total** for the family — pairs with the Jan 6–13 resort window.
>
> | Origin plan | Best option | Total | Verdict |
> |---|---|---|---|
> | JFK nonstop | JetBlue 10:30 AM, 4h25m | $864 | ✅ winner |
> | BDL (no nonstops exist) | United via IAD, 6h32m | $825 | +hotel risk, longer day |
> | Split via Tampa (Avelo + Breeze) | 2 travel days | $938 + hotel night | loses on cost AND comfort |
>
> No nonstop BDL→CUN service exists in January — that's a finding, not a failure. The split itinerary loses because the budget carriers don't fly daily and separately ticketed legs carry no protection.

## License

MIT — see [LICENSE](LICENSE).

## Notes

- Prices shown are Google Flights totals for all passengers, taxes included, bag fees excluded.
- Price-tracking alerts email the Google account signed in to Chrome; Claude will ask before flipping the toggle and will never sign in for you.
- Google changes its UI occasionally; the skill instructs Claude to screenshot-and-adapt rather than rely on fixed layouts, but if a step breaks, re-run or file an issue where you got the skill.
