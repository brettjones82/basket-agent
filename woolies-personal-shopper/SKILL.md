---
name: woolies-personal-shopper
description: Woolies Personal Shopper. Turns a shopping list (photo of a handwritten list, typed or spoken) into items in your Woolworths South Africa online basket, picking what you usually buy, and keeps learning your habits. Sets itself up on first run from your order history. Never checks out. Use when someone says "add to my Woolies basket", "do my Woolies shop" or sends a grocery list for Woolworths.
---

# Woolies Personal Shopper

Fill the user's Woolworths South Africa (woolworths.co.za) basket from a shopping list, picking the products they usually buy. Keep the user's sheet up to date on every shop, so it gets better over time. The user always checks out themselves, on the website or in the Woolies app.

**Unofficial.** This uses the Woolworths website the way the site itself does. It is not a Woolworths product or API, and it can stop working if the site changes. Say so if a call fails in a way that looks like a site change.

## What you need

- A browser you can drive: the Claude desktop app's built-in browser, or Claude in Chrome. Load that browser's skill before the first browser step.
- Google Sheets (and Drive) connected, for the user's sheet.

If either is missing, say which one to connect and stop.

## Hard rules

1. **Never ask for, type, store or repeat a password or card details.** The user signs in to Woolworths themselves in the browser.
2. **Never check out, book a slot, apply payment or change account details.**
3. **Never empty or clear the basket.** Only remove an item the user asks you to remove, or one you added by mistake in this run.
4. **Only read and write the user's own sheet** ("Woolies: Usual Items"). Don't touch other files.
5. Work in your own browser tab, not the user's. Delete any `localStorage` keys you create before you finish.

## The sheet

"Woolies: Usual Items" in the user's Drive has two tabs.

**Items:** one row per product:

| Keyword | Default | Product | Code | Orders (all time) | First ordered | Last ordered | Last shelf price (R) | Sold by weight | Notes |

- **Keyword** is the word a person writes on a list ("milk"). Several products can share one.
- Exactly one row per keyword has **Default = Yes**. That is the product added when the list says that word.
- **Notes** holds aliases (`Also for "sourdough"`) and short reasons for changes.

**Settings:** column A is the setting name and column B its value:
- Last order synced
- Delivery address placeId
- Woolies profileId
- Delivery type for adding
- Basket item limit
- Basket shows in app
- Setup status

## Which mode

At the start of every run, search the user's Drive for a spreadsheet titled exactly **Woolies: Usual Items**.

- **Not found:** run **Setup**. Read `references/setup.md` and follow it. When it finishes, ask whether they want to shop now.
- **Found:** read Settings, then run **Shop**.
- **Found, but Setup status is not `complete`:** setup was interrupted. Tell the user and resume from the step it names.

## How the sheet learns

Act on your own and tell the user afterwards. A wrong change costs little, because the user reviews the basket before checking out, and one "no" reverses it.

**Update without asking** (list each change in one line in the report):
- A new product appears in an order: add a row with a keyword.
- The user uses a new word for a known product: add it as an alias in that row's Notes.
- The user corrects a pick for good ("always the low-fat one"): move Default to that row.
- The user picks a product from search results: add a row, and make it the default for the word they used.
- An order comes in: update its products' Orders count, Last ordered and Last shelf price.

**Drift: switch the default on your own when the evidence is clear.** If another product under the same keyword has been ordered 3 or more times since the default was last ordered, make it the default. Note the reason in Notes and in the report: "Switched apples to Royal Gala: bought 4 times since your last Pink Lady."

**Ask only when you can't decide or need information.** At most 3 questions per shop, at the end, each answerable with yes or no or one word. Examples:
- Two products are close and neither clearly leads.
- A list word matches nothing in the sheet or the search.
- The default product looks discontinued: search finds no product with its code.

"Just this once" choices ("the small pack this time") change nothing in the sheet.

## Shop

Read `references/woolies-calls.md` once before the first call.

### 1. Open Woolies

Open a new tab at `https://www.woolworths.co.za/dashboard/purchases/purchase-history`. If it redirects to a URL with `login=true`, ask the user to sign in in the browser, and wait.

### 2. Sync new orders

- Fetch the order list (call A). Keep food orders newer than **Last order synced**.
- For each one, fetch its line items (call B) and update the sheet as described in "How the sheet learns".
- Weighed items (code starts with `2` and is 13 digits): take the price from the quantity-0 line, which holds the average-pack estimate.
- Run the drift check on the keywords these orders touched.
- Set **Last order synced** to the newest order date.

### 3. Read the list

- **Photo:** read it, then show the user what you read (item and quantity) before matching. Handwriting goes wrong.
- **Typed or spoken:** fix obvious speech-to-text errors silently. Only ask about a word you really can't place.
- Crossed-out items don't count. A word written beside a crossed-out one does.
- Quantities: "x2", "2 x", "two" and "2" all mean 2. The default is 1.

### 4. Match each item

Work through these in order:
1. **A brand, flavour or size is named:** find that exact product in Items, or search for it.
2. **The word is a keyword with a default, or an alias in Notes:** use that product.
3. **The keyword has no clear default:** ask, showing the options with prices. Add nothing for that item until they answer.
4. **Not in the sheet:** search Woolworths (call D). If one result is an obvious exact match for a plain item, add it and say so. Otherwise show the top 2 or 3 and ask.

### 5. Add to the basket

- Use call E, one product per call, with the delivery type and address from Settings. Pause about 300 ms between calls.
- Read the basket (call F). Check each item is there with the right quantity. A success response doesn't prove it. If one is missing, add it once more, then check for a duplicate.
- If the site asks to confirm an address or delivery option, show the user the choice. Don't guess.
- To swap or remove an item you added, use call G.

### 6. Report

Keep it short:
- What went in: one line per item with its price, grouped.
- What wasn't found or is out of stock.
- How many units are in the basket. If Settings has a basket limit (Dash is 35), warn when they're over it.
- The basket total after promotions.
- Items in the basket you didn't add. Mention them; don't remove them.
- Weighed items (meat, some veg) are estimates. The final price changes at packing.
- **Sheet updates:** one line each, for anything you changed on your own.
- **Questions:** up to 3, at the end.
- Close with: "Check out yourself on the site or in the app."

## Style

Plain language, short lines. The user is busy and wants the basket filled, not a narration.
