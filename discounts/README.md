# Osysharp.Shop.Discounts

Discount codes for [Osysharp.Shop](https://osyrin.com/templates/kits/shop/): percent off, an amount off, or free delivery, with a minimum spend, an end
date, a limit in all and a limit per customer. An add-on — list it, put the code field in your checkout and the list on
your desk, and the shop prices every code with the basket.

- **A field at the checkout** — `CheckoutCode(visitId, onChange)`. Typed codes are normalised (case and spaces do not
  matter); a code that is unknown, ended, not yet valid, used up or under its minimum spend says which. Shown only
  while the shop has a code running.
- **Priced with the basket, every time** — a code is checked again whenever the checkout or the order is priced, so a
  code that ran out between the bag and the order takes nothing off rather than something it no longer may.
- **Limits** — in all (`MaxUses`) and per customer (`MaxPerCustomer`). A guest's email is known only when the order is
  placed, so a guest who has had their uses is told then, BEFORE anything is priced: an order is never placed at a total
  the customer has not seen.
- **Counted once used** — the order keeps the code as one of its adjustments, the code's `Uses` goes up, and a
  reservation called off gives its use back.
- **Codes nobody can list** — a code is looked up by its keyed fingerprint (`Security.Fingerprint`); the code itself is
  readable by staff alone, and who used it is counted by a fingerprint of their email.
- **In a currency the shop sells in besides its own** — an amount off is a promise in ONE currency ("100 kr off" on a
  flyer is 100 kr, not a figure converted at each day's rate), so an amount-off code applies in a sold currency only
  once staff give it an amount there (`SetCodeAmountIn`, the desk's "Other currencies…"); until then a shopper buying
  in it is told which currencies it is for. A percent and a free delivery apply in any currency; their minimum spend is
  the one set for the currency, else the shop's converted at the day's rate — a threshold, so to the cent.
- **The desk** — `DeskDiscountCodes()`: every code with what it gives and how often it was used, start and stop, and a
  form for a new one.

## Use it

```osy
// app.osy
use Osysharp.Shop@0 { egress "api.resend.com"; secret "ResendApiKey"; }
use Osysharp.Shop.Discounts@0;
```

That `use` line is all: the shop finds the add-on (`DiscountCodes`) and prices every basket with it.

```osy
// the checkout: re-price when a code is applied or removed
CheckoutCode(visitId, Offer);
```

Staff make codes on the kit's own page, `/staff/codes` (`DiscountCodesPage`), which stands in the store's staff frame
(`WorkspaceShell`) and its navigation (`[Nav]`); the desk's "Issue a gift card" points to it too (the shop's
`GiftCardGoodwill` slot). The panel itself is `DeskDiscountCodes()`, for a page of your own.

From code: `CreateCode(code, label, kind, value, minSpend, endsAt, maxUses, maxPerCustomer)` (staff),
`ApplyCode(visitId, code)`, `BagCodes(visitId)`, `RemoveCode(applied)`, `CodesRunning()`.

A card payment through Stripe carries the discount as a coupon on Stripe's page, so the customer sees the same total
there. The discount is kept on the order and the invoice as an adjustment, and the VAT inside the price is worked out
on what was actually paid.

## How it is built

It is an ordinary `IShopAddOn`: `Adjust(basket)` gives what comes off and `Refuses(basket)` refuses a basket that may not
be ordered as it is. The uses are counted by two `[On]` handlers of the shop's events — `OrderPlaced` counts them,
`OrderCancelled` gives them back. Loyalty points, gift cards or a bundle price are
built the same way, in a kit of their own.
