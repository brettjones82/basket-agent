# Woolies Personal Shopper

**Send Claude your shopping list. It fills your Woolworths basket with the things you usually buy.**

Take a photo of the list on the fridge, type it, or just say it:

> "Add milk, bread, apples, mince x2 and the butter chicken sauce to my Woolies basket."

Claude adds *your* milk, *your* bread and *your* apples, because it has learnt what you normally buy from your past orders. You check the basket and pay as normal, on the website or in the Woolies app.

It never checks out for you.

---

## What you need

- **The Claude desktop app** (Mac or Windows), with its built-in browser. Claude in Chrome also works.
- **Google Drive connected to Claude.** Your list of usual items lives in a Google Sheet in your own Drive.
- **A Woolworths online account** (woolworths.co.za) with some past online orders.

## Install

1. Download this repo as a zip (green **Code** button, then **Download ZIP**).
2. In Claude, open **Settings → Capabilities → Skills** and upload the `woolies-personal-shopper` folder (zipped).
3. Start a new chat and say: **"Do my Woolies shop."**

## The first time (about 10 minutes)

Claude sets itself up:

1. **It opens Woolworths and asks you to sign in.** You type your password yourself, in the browser. Claude never sees or asks for it.
2. **It reads your past online orders.**
3. **It makes a Google Sheet called "Woolies: Usual Items"** in your Drive. Each row is a product you buy, with the word you'd write on a list ("milk", "bread"). One product per word is marked as your usual.
4. **It shows you your top 20 and asks if anything's wrong.** "No, we always get the low-fat one" is the kind of thing to say here.
5. **It adds one test item** so you can check it shows up where you normally check out, then removes it.

That's it. From then on, just send a list.

## Every time after that

1. Send the list: a photo, typed or spoken.
2. Claude picks your usual products and adds them. If it isn't sure ("apples": the 1 kg bag or the 1.5 kg one?), it asks.
3. It tells you what went in, what it couldn't find, and the total.
4. You open the basket, check it, and pay.

## It gets better as you use it

You don't need to maintain anything. Each shop, Claude:

- **adds new things you've started buying** to your sheet;
- **remembers your words.** Say "when I say the good bread I mean the seeded loaf" once, and it remembers;
- **notices when your habits change.** Buy a different brand a few times and it becomes your new usual. Claude tells you when it switches;
- **asks at most 3 quick questions** when it genuinely can't tell.

You can also edit the sheet yourself any time. Move the "Yes" in the Default column to change your usual.

## Good to know

- **Unofficial.** This isn't made or supported by Woolworths. It uses the Woolworths website the way the site itself does, so it may stop working if Woolworths changes their site.
- **Your data stays yours.** Your order history goes into a sheet in your own Google Drive and nowhere else.
- **It never pays.** It doesn't check out, book delivery slots or touch your payment details.
- **Meat and veg sold by weight** show an estimated price until your order is packed.
- **Only online orders count.** Things you buy in store won't show up as "usual" unless you add them to the sheet.
- **Dash users:** Dash baskets are limited to 35 items. Claude warns you when you go over.

## What's in this repo

```
woolies-personal-shopper/
  SKILL.md                     the instructions Claude follows
  references/setup.md          the first-time setup steps
  references/woolies-calls.md  how Claude talks to the Woolworths website
```
