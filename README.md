# flight-scout — a Claude Code skill

Turns Claude Code + the Claude in Chrome extension into a flight-research assistant: it drives Google Flights in your own browser, prices every reasonable option (multiple airports, date windows, even split "positioning flight" itineraries), recommends the best one, and can set Google Flights price-tracking alerts for you.

## What it does

- Asks for origin(s), destination, dates, and passenger mix — or uses defaults you hard-code.
- Checks nonstops first; falls back to connections with layover details when nonstops don't exist.
- Compares alternate airports and creative split itineraries honestly (including hotel nights, bag fees, and missed-connection risk).
- Reports totals for your whole party in the currency Google shows, ranked, with a clear recommendation.
- Offers to toggle Google Flights "Track prices" alerts (requires you to be signed in to Google).

It never books, holds, pays, or enters personal data. Research only.

## Requirements

- [Claude Code](https://claude.com/claude-code) (CLI or desktop app)
- Google Chrome with the **Claude in Chrome** extension installed and connected
- Extension permission granted for google.com

## Install

Copy this folder into your skills directory:

- Personal (all your projects): `~/.claude/skills/flight-scout/`
- Or per-project (shared with a repo's users): `<repo>/.claude/skills/flight-scout/`

New Claude Code sessions will pick it up automatically. Invoke it by typing `/flight-scout`, or just ask something like "find me flights from BOS to Lisbon in May."

## Customize (optional)

Open `SKILL.md` and fill in the **Personal defaults** section (home airports, airports to avoid, your usual party, cabin preference) so Claude stops asking for them every time. Leave it blank to be prompted per search.

## Notes

- Prices shown are Google Flights totals for all passengers, taxes included, bag fees excluded.
- Price-tracking alerts email the Google account signed in to Chrome; Claude will ask before flipping the toggle and will never sign in for you.
- Google changes its UI occasionally; the skill instructs Claude to screenshot-and-adapt rather than rely on fixed layouts, but if a step breaks, re-run or file an issue where you got the skill.
