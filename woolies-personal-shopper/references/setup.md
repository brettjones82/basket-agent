# First-run setup

Goal: a Google Sheet called **Woolies: Usual Items** (the Woolies Personal Shopper's memory) that knows what this household usually buys, plus the few account settings the skill needs. This takes about 10 minutes, mostly waiting for order history to load.

Tell the user up front:

> First time, so I'll set things up. I'll open Woolworths for you to sign in, read your past online orders, and build a sheet of what you usually buy. I won't check out or change anything on your account. About 10 minutes.

Write `Settings!B8` (Setup status) as you go, so an interrupted setup can resume.

## 1. Check tools

- Is there a browser you can drive? Load its skill.
- Are Google Sheets and Drive connected?
- If either is missing, say which one, how it helps, and stop.

## 2. Sign in

- Open a new tab at `https://www.woolworths.co.za/dashboard/purchases/purchase-history`.
- If it redirects to a URL with `login=true`, say: "Please sign in to Woolworths in the browser, then tell me when you're in." Never ask for the password in chat.
- When they're in, the page shows "My Orders".

## 3. Find the account

- Call C (`/server/currentUser`) and read `profileId`.
- Check it: call A with that `profileId` should return orders. If it returns none or an error, read the browser's network requests on the purchase-history page for `purchaseHistory?profileId=` and use the number from there.
- Store it in `Settings!B4`. Don't show it to the user, and don't store their name or email.

## 4. Pick the delivery address

- Call H (`/server/savedAddress`). Show the user their saved addresses by nickname and suburb only, and ask which one to use for groceries.
- Store that address's `placeId` in `Settings!B3`.
- If there are no saved addresses, ask them to add one on the website first, then continue.

## 5. Pull order history

- Call A in pages of 100 until it returns fewer than 100. Keep orders where `itemType.food` is true.
- Call B for each order, about 8 at a time. Save progress in `localStorage` after each batch, because the tool call can time out and a page navigation wipes page variables. Stop each call after about 35 seconds and run another to continue.
- Expect about 5 seconds per order the slow way and well under 1 second in parallel. 100 orders takes 1 to 3 minutes.
- For each line item keep: code (`productId`), name, quantity, `priceInfo.listPrice`, and the order date.

**Weighed items** (meat, some veg): the code starts with `2` and is 13 digits long, and the name usually has "Avg". These appear twice in an order: once with quantity 0, whose `listPrice` is the estimate for the average pack (use this as the shelf price), and once with quantity 1 at the actual weighed price.

## 6. Build the sheet

Aggregate per product code:
- orders (all time)
- orders in the last 12 months (only for choosing defaults below; don't store it in the sheet)
- first ordered and last ordered dates
- the dates of its 5 most recent orders, newest first, one date per order (weighed items appear twice in an order; count it once)
- last shelf price
- sold by weight (yes or blank)

**Which rows to keep:** anything ordered at least twice, or at least once in the last 12 months. Leave out old one-off buys; step 2 of Shop adds new items as they appear.

**Keywords:** give each kept product the one word or short phrase a person would write on a shopping list for it: "milk", "bread", "apples", "baby tomatoes", "chicken fillets", "soy sauce". Use your judgement from the product name, not pattern rules. Products that are really the same thing in different sizes share a keyword. Dried fruit is not "apples"; crisps flavoured like chicken are not "chicken".

**Defaults:** within each keyword, mark `Default = Yes` on the product with the most orders in the last 12 months. Break ties on all-time orders, then the most recent date. Exactly one default per keyword. Set **Default set** to today's date on each default row and leave it blank on the others.

**Create the sheet** in the user's Drive titled `Woolies: Usual Items`:

Tab `Items`, sorted by Last ordered (newest first), then by Orders (all time), with a header row:

| Keyword | Default | Product | Code | Orders (all time) | First ordered | Last ordered | Last shelf price (R) | Sold by weight | Notes | Default set | Recent orders |

- **Recent orders** holds the 5 dates from above, comma-separated (`2026-11-08, 2026-10-30`).
- Freeze the header row and add a filter.
- Format the Code column as a plain number (pattern `0`) so long codes don't turn into scientific notation.
- Format Default set and Recent orders as plain text (pattern `@`) so Sheets doesn't change the dates.

Tab `Settings`:

| | A | B |
|---|---|---|
| 1 | Setting | Value |
| 2 | Last order synced | newest order date |
| 3 | Delivery address placeId | from step 4 |
| 4 | Woolies profileId | from step 3 |
| 5 | Delivery type for adding | `Standard` |
| 6 | Basket item limit | blank, or 35 if they use Dash |
| 7 | Basket shows in app | `untested` |
| 8 | Setup status | `complete` when done |
| 9 | How Default works | One row per keyword has Default = Yes. That product is added when your list says the keyword. Move the Yes to change your usual. |

## 7. Check the defaults with the user

Show the 20 most-bought keywords with their default product, as a compact list. Ask: "Anything here wrong, or anything you call by a different name?" Apply their answers (see the correction rules in SKILL.md).

## 8. Test the basket

- Ask: "Do you check out on the website, or in the Woolies app (for example Dash)?" If Dash, set `Settings!B6` to 35.
- Add one cheap item they buy often (call E).
- If the site shows an address or delivery pop-up, show the user the choice. Pick the address from step 4 and the delivery option they confirm.
- Ask them to check the basket where they normally check out:
  - **They see it:** set `Settings!B7` to `yes`, ask them to remove the test item (or remove it with call G if they say so), and finish.
  - **They don't see it** (common with Dash): try once more with the delivery type the app uses. Read `deliveryType` from their most recent order (call A); Dash orders show `OnDemand`. Add the item with that value and ask them to check again. If that works, store it in `Settings!B5`. If not, set `Settings!B7` to `no`, tell them the basket will be on the website only, and remove the test item.

## 9. Finish

- Set `Settings!B8` to `complete`.
- Delete your `localStorage` keys.
- Tell them: the sheet name, how many items it holds, and that next time they can just send a list (a photo, typed or spoken).
