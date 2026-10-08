# Osysharp.Shop.Subscriptions

Selling on a cadence, for software and for goods alike — an add-on to [Osysharp.Shop](https://osyrin.com/templates/kits/shop/).

- **Plans and add-ons, priced per interval** — every two or four weeks, monthly, yearly. A yearly price below twelve
  monthly ones is how a yearly discount is written: data, not code.
- **What a plan grants** — features (`feature.ha`) and amounts of a unit (`unit.storage-gb`: 10). `EntitlementsOf`
  answers what a customer has, limits summed over plans and add-ons; `IsEntitled` answers one key. This is what another
  system asks to unlock features.
- **Changes priced to the day** — an upgrade or an add-on mid-period charges the difference for the days left; removing
  one credits them; a downgrade can wait for the renewal instead.
- **Renewal** — the subscription's workflow checks daily; a period that has ended renews at today's prices (a waiting
  downgrade applied), or the subscription ends when it was set to.
- **A ledger of charges** — renewals and changes, each with what it was for: what an invoice and a payment are made from.
- **Per currency** — a plan has its price in the shop's own currency and, for a currency the shop SELLS in besides
  its own (Osysharp.Shop's "Sell in EUR"), a price of its own there (`PlanPrice.Currency`, set with `SchedulePriceIn`
  or a plan import's `PlanStartingPrice.Currency`); without one, its own price converted at the day's rate and rounded
  by the currency's rule, as a variant's (`PlanAmountIn`). A subscription is bought in the visit's currency and
  RENEWS in it (`Subscription.Currency`): renewal orders, their invoices and books (each at its own invoice's rate),
  the kept card's charge and every price notice are in it. A converted price is fixed when agreed and moves only when
  the shop moves its own price (told first, `SubscriptionItem.NextUnitPrice`); a renewal whose currency is not sold
  now, or whose rate is out of date, waits a day at a time (`Subscription.RenewalWaits`) — it is never charged in
  another currency.
- **When the shop stops selling a currency** (the desk's "Only show it", stopping the rates, or the app no longer
  offering it — `IShopAddOn.CurrencyNoLongerSold`; a rate that merely goes STALE moves nothing) every subscription in it
  MOVES to the shop's own currency: each item's price converted at that day's rate and rounded by the shop's own rule
  (`ShopSettings.Rounding`) into `SubscriptionItem.MovedPrice`, the subscriber mailed in their account's language
  (`SubscriptionCurrencyMoveMail`) and the other system sent a `CurrencyMoving` notice (`NewPrice`, `NewCurrency`,
  `From`), with the price notice's lead time (`PriceNoticeDays`). From the first renewal on or after
  `Subscription.MovesOn` it renews in the shop's currency; until then it renews as agreed if the shop sells in the old
  one again, else it waits — and a renewal that waited for the day starts its period that day (the wait is not
  charged). The moved price is then fixed (`SubscriptionItem.PriceFixed`) until the shop moves the plan's own price.
  The desk says so beside each one ("Moves to SEK on 29 Oct 2026: 147 kr, every four weeks").

Plan keys are the contract: stable, what another system pushes and reads. A retired plan is no longer sold, and its
subscribers keep it until they change.

## Paid, shipped and self-served

- **Started at the checkout** — a customer chooses a plan in the shop (`ChooseSubscription`), and its first period is
  an ordinary order; staff or another system can start one too (`Subscribe`).
- **Charged to the kept card** — every period and every priced change is placed as an order, so its invoice and books
  are the shop's own, and paid with the card the customer asked the shop to keep. A refused payment is tried again
  every few days; paid, the subscription is active again, and an order that lapses unpaid ends it.
- **Shipped plans** — a plan that sells goods is a delivery each period, and the customer can change how many.
- **Self-service** — a customer asks (`AskAboutSubscription`) to cancel at the end of the period, keep renewing,
  pause, resume, change plan or cadence at renewal, skip the next one, or change the quantity. They never write the
  subscription; the shop applies the ask by its own rules and keeps it with its answer.
- **Notices to another system** — every period drawn (paid or refused), every start, change, pause and end is posted
  as signed JSON to the endpoint the app names (`SubscriptionSetup.NoticeUrl`), retried until taken, so a control
  plane or a CRM can keep its own state from them.

## The staff desk

`/staff/subscriptions` is the kit's own page (`SubscriptionsDeskPage`), standing in the store's working shell and listed
in its staff navigation like the shop's desk pages — a store writes nothing to have it. It lists every subscription:
who, which plans, what a period costs in the subscription's own currency, where it stands (Active, Ending, Paused, Past
due, Cancelled), when the next period starts, what the subscriber last asked, and — for one moving currency — when and
at what price. Staff act on one from its row: **Pause** and **Resume** (`PauseSubscription`, `ResumeSubscription`),
**Stop at period end** and **Keep it going** (`CancelSubscription(sub, true)`, `KeepSubscription`), **Stop now**. Above
the list, **Change a price** schedules a plan's price from a date, in the shop's currency or one it sells in, telling
its subscribers or keeping them on what they pay (`SchedulePriceIn`). To tailor the page, take it over by name; to
leave it out, `use Osysharp.Shop.Subscriptions@0 { without SubscriptionsDeskPage; }`; to place the desk on a page of
your own, `SubscriptionsDesk()`.

Plans are priced per interval. Metered usage and prepaid top-ups are not part of the kit.
