# Woolworths website calls

Run these with the browser's JavaScript tool, **inside a tab on `https://www.woolworths.co.za`** after the user has signed in. They use the user's own session (`credentials: 'include'`). They are the same requests the website makes. They are not a public API and can change without notice.

Keep each tool call under about 40 seconds. Long loops: save progress to `localStorage` and continue in the next call.

## A. Order list

```js
const r = await fetch(`/server/purchaseHistory?profileId=${PROFILE_ID}&days=all&limit=100&skip=${SKIP}`, {credentials:'include'});
const { orders } = await r.json();
// each: orderId, submittedDate (YYYY-MM-DD), itemType {food:true} or {other:true}, deliveryType ("Standard", "OnDemand" = Dash)
```

Page with `skip` until fewer than `limit` come back. Ignore orders without `itemType.food`.

## B. Order details (line items)

```js
const j = await (await fetch('/server/orderDetails?orderId=' + ORDER_ID, {credentials:'include'})).json();
const items = (j.items && j.items.food) || [];
// each: productId, productDisplayName, quantity, priceInfo.listPrice (shelf price), priceInfo.amount (paid)
```

Weighed items: the code starts with `2` and is 13 digits. A quantity-0 row carries the average-pack estimate in `listPrice`.

## C. Current user

```js
const u = await (await fetch('/server/currentUser', {credentials:'include'})).json();
// u.profileId
```

## D. Search (codes come from the page's data layer, not its HTML)

```js
// navigate the tab to: https://www.woolworths.co.za/browse?searchterm=<url-encoded term>
// wait ~4 s, then:
const ev = window.dataLayer.filter(e => e.event === 'view_search_results').pop();
ev ? ev.ecommerce.items.slice(0, 10).map(x => [x.item_id, x.item_name, x.price]) : 'no results';
```

Prices are for the address the site currently has set.

## E. Add to basket

```js
const r = await fetch('/server/cartAddItems', {
  method: 'POST', credentials: 'include', headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    deliveryType: DELIVERY_TYPE,            // Settings!B5, e.g. "Standard"
    address: { placeId: PLACE_ID },         // Settings!B3
    items: [{ productId: CODE, catalogRefId: CODE, quantity: QTY, itemListName: 'Product Description Page' }]
  })
});
const j = await r.json();   // j.message === "Items Added Successfully"; j.errorMessage on failure
```

One product per call. Always confirm with F afterwards.

## F. Read the basket

```js
const c = await (await fetch('/server/cartDetails?page=cart&t=' + Date.now(), {credentials:'include'})).json();
const lines = (c.items && c.items.food) || [];
// each: id (line id, e.g. "ci2202056918"), catalogRefId, productDisplayName, quantity, priceInfo.amount
// c.groupSubTotal.cartTotalWithDiscount = total after promotions
```

The `t=` timestamp avoids a cached copy.

## G. Remove a line

```js
await fetch('/server/cartRemoveCommerceItems?commerceItemId=' + LINE_ID, {method: 'DELETE', credentials: 'include'});
```

`LINE_ID` is the line's `id` from F, not the product code.

## H. Saved addresses

```js
const a = await (await fetch('/server/savedAddress', {credentials:'include'})).json();
// find each address's nickname, suburb and placeId; field names may vary, so inspect the response once
```

If this call doesn't return usable addresses, use the site instead: open any product page and click "Add to Cart". The site shows a "Confirm your address" pop-up listing saved addresses. Ask the user to choose, then read `placeId` from the request the site sends to `/server/cartAddItems`.
