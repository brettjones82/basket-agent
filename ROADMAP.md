# Roadmap

What we're working on, what's next and ideas worth trying. Anyone can suggest a change to this page with a pull request, the same way as code.

Last updated: 24 Nov 2026

## How we work

- **Every change is a pull request.** Nothing goes into `main` until Brett approves it. GitHub shows each change side by side, like tracked changes in Word.
- **One owner per item.** Put your name in the Owner column before you start, so two people don't build the same thing.
- **New ideas go in [Ideas](#ideas)** first. They move to Next once we agree they're worth doing.
- **Keep the hard rules.** Never check out, never pay, never ask for or type a password, never empty a basket. Any new store follows the same rules.
- **Say how you checked it.** Each pull request says what was tested, and how: a real shop, a test sheet or only reading it through.

## Where things stand

| | |
|---|---|
| Works today | Woolworths: setup from order history, list to basket, the sheet learns your habits. First used for real by Wiehahn on 23 Nov 2026. |
| Built but not in this repo yet | Wiehahn's version: Checkers Sixty60 price comparison, per-kg comparisons, cheapest basket split across stores including delivery, recipes to a shopping list. His first small basket saved just under 9%. |
| Not yet tested on a real shop | The habit-change rule rewritten on 23 Nov (only checked against 7 worked examples). |
| Known friction | Installing: finding where to upload the skill zip, and connecting Google Sheets. |

## Now

| Item | Why | Owner | Status |
|---|---|---|---|
| Give Wiehahn access to the repo | So he can send his changes as pull requests | Brett | Waiting for his GitHub username |
| Bring in Wiehahn's Checkers, per-kg and recipe changes | They work, and they're the most useful thing built so far | Wiehahn | Not started |
| Make installing easier | It took the longest. Clearer steps with screenshots, plus a ready-made zip on GitHub so nobody has to zip a folder | Anyone | Not started |

## Next

| Item | Why | Owner | Status |
|---|---|---|---|
| One skill, one file per store | Keeps Woolworths solid while adding stores. Each store gets its own `references/<store>-calls.md`, like the Woolworths one today | | |
| Store columns in the sheet | Each item needs a Checkers code next to the Woolies one, plus a per-item store rule ("Woolies only" for meat, or "cheapest"), rather than rules buried in instructions | | |
| Always compare per kg or per unit | Pack prices mislead. Only compare products of a similar size | | |
| Delivery and subscriptions in the totals | Delivery decides whether splitting a basket is worth it. A Sixty60 subscription changes the maths | | |
| First real test of the habit-change rule | It has only been checked against worked examples | | |
| Decide on a name | It's more than a Woolies shopper once Checkers is in | | |

## Ideas

Not agreed yet. Add yours, and say why it matters.

| Idea | Why | Suggested by |
|---|---|---|
| Price watch | The sheet already stores the last price. "Your yoghurt is up 18%" or "your usual coffee is on promotion" | Brett |
| Out-of-stock swaps | When your usual is out, pick the closest match by size and price per kg, and say what changed | Brett |
| Two people, one household | One partner's account often holds the order history. Read both accounts into one sheet | Wiehahn (Ash's account) |
| Budget cap | "Keep it under R800": it picks cheaper options and says what it changed | Brett |
| Week of meals | Suggest a few dinners from recipes you like, then shop for them all at once | |
| More stores | Pick n Pay and others, once two stores work well | |

## Open questions

- **Hobby or product?** Wiehahn suggested launching it as a tool for people. Before anyone spends real time on that: neither store has a public API, so this relies on using their websites as a shopper would, which can break without warning and may conflict with their terms. A product would also need support and accounts. Decide this together before building towards it.
- **Name:** keep "Woolies Personal Shopper", or something broader?

## Decisions

| Date | Decision |
|---|---|
| 23 Nov 2026 | Repo made public under the MIT licence |
| 23 Nov 2026 | The habit-change rule counts purchases since your usual was last bought, or since you chose it, whichever is later. Your own choice sticks |
| 24 Nov 2026 | Changes come in as pull requests, approved by Brett |
