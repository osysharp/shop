# Osysharp.Shop — an online shop's backend, and the parts of its pages every shop needs

## Use case

You sell physical things, and you want a shop you own rather than rent: your domain, your data, your design, and no
monthly fee per feature. This package covers what every shop needs and nobody wants to write twice. That means the
catalogue, accounts, the bag, orders that reserve stock, card and Klarna payments, receipts, returns and refunds, the
shop's mail and the staff desk. Your app supplies the design: the pages, the theme and the photographs.

The same package runs a bicycle workshop, a fashion label and a second-hand shop:

| | | |
|---|---|---|
| ![Vall, a fashion shop](docs/fashion-store-atelier.jpg) | ![Rail, a second-hand shop](docs/thrift-store-rail.jpg) | ![Rust & Chrome, vintage bicycles](docs/rust-and-chrome.jpg) |

Each of these is a **template** you can start from:

```console
osy init shop --style atelier
```

## What it ships

| area | what you get |
|---|---|
| **Catalogue** | categories as a tree, products with variants (size, colour) and their own stock and price, a spec sheet per product, photos, draft/listed/archived |
| **Accounts** | sign-up, sign-in, password reset by mail, change email and password; one principal with a Customer or Staff role; guest checkout still works |
| **The bag** | kept per visit and saved on the account, merged at sign-in, re-checked against stock on return; "mail me when it is back" |
| **Orders** | placing an order reserves its goods; an unpaid order lapses after the shop's hold days and puts them back; paid → sent → refunded, each step mailed |
| **Payments** | Stripe Checkout (card, wallets) with its signed webhook; Klarna's hosted page (charged when the order is sent); bank transfer; pay on collection |
| **Receipts** | a PDF per order, kept private and handed to its owner as a signed link |
| **Returns** | the customer returns lines from their order history within the shop's return window; staff receive them; the refund goes back the way the money came |
| **Figures** | top sellers, most viewed, slow movers, totals over a period |
| **Clearance** | mark a product down by a percentage until a date; a sale shelf |
| **Mail** | confirmations, dispatch, resets and returns through Resend, or held in an outbox the desk can read until a key is set |
| **The desk** | one staff page, `ShopDesk`, with tabs for figures, products, stock, returns, customers, mail and settings |

The storefront pieces ship as components that take your theme's tokens, so they look like your shop: `ShopSearchPage`,
`ShopProductView`, `ShopNotifyWhenBack`, `ShopSignIn`, `ShopSignUp`, `ShopForgotPassword`, `ShopAccountPanel`,
`ShopOrderHistory`, `ShopPolicy`, `ShopPolicyLinks`, `ShopAccountButton` and `ShopDesk`.

## Install and use

```osy
app MyShop {
  use Osysharp.Ui;
  use Osysharp.Workflow;
  use Osysharp.Storage;
  use Osysharp.Http;
  // The shop reaches Stripe, Klarna and Resend and reads their keys. This is where the app allows each one.
  use Osysharp.Shop@0 {
    egress "api.stripe.com"; egress "api.resend.com"; egress "api.klarna.com"; egress "api.playground.klarna.com";
    secret "StripeApiKey"; secret "StripeWebhookSecret"; secret "ResendApiKey";
    secret "KlarnaUsername"; secret "KlarnaPassword";
  }
  use Osysharp.Stripe@0 { egress "api.stripe.com"; }
  use Osysharp.Klarna@0 { egress "api.klarna.com"; egress "api.playground.klarna.com"; }
  use Osysharp.Resend@0 { egress "api.resend.com"; }
  model "model/**/*.osy";
}
```

Then three pieces of wiring, which are the app's to state because a package does not open routes or choose a login
page for you:

```osy
using Osysharp.Shop;

app.Auth = new PasswordAuth { LoginField = Email, PasswordField = PasswordHash };
app.AuthBootstrap = new AuthBootstrap {
  Role = ShopRole.Authenticator, LoginPage = AccountSignIn,          // AccountSignIn is YOUR page, around ShopSignIn()
  Login = ShopLogin, Signup = ShopSignup,
  PasswordResetRequest = ShopRequestPasswordReset, PasswordReset = ShopResetPassword,
  PasswordChange = ShopChangePassword, EmailChange = ShopChangeEmail,
};

app.Apis = [
  new RestApi("Stripe") { Route = "stripe", Auth = new ApiAuth { Anonymous = true },
    Endpoints = [ new Endpoint(ShopStripeEvents) { Method = HttpMethod.Post, Path = "/events", Raw = true,
                                                   Headers = ["Stripe-Signature"] } ] },
  new RestApi("Klarna") { Route = "klarna", Auth = new ApiAuth { Anonymous = true },
    Endpoints = [ new Endpoint(ShopKlarnaEvents) { Method = HttpMethod.Post, Path = "/events", Raw = true } ] },
];
```

A payment method is offered only once it is configured, so a new shop takes bank transfers and nothing else:

```console
osy secret set StripeApiKey          # sk_test_… while you build
osy secret set StripeWebhookSecret   # whsec_…, for Stripe's webhook to https://your.shop/stripe/events
osy secret set KlarnaUsername        # and KlarnaPassword; the market is set in the desk
osy secret set ResendApiKey          # and "Mail from" in the desk's settings
osy user add you@yourshop.se --role Staff
```

## Extend it, or take it over

**Extend it first.** These are the seams, and none of them needs you to touch the package:

- **Your goods.** A spec sheet is data, so staff add "Frame size" or "Material" in the desk without a release. A
  subtype of your own, `entity Bike : Product { … }`, adds the fields every product you sell has.
- **Your pages.** Every page is yours. Place the kit's components where your design wants them, or write your own
  page against the same functions (`BagFor`, `PlaceOrder`, `MyOrders`, `TopSellers`, …). They are the same calls the
  kit's components make.
- **Your look.** The components read your theme's tokens (`Bg`, `Surface`, `Primary`, `TextMuted`, your fonts). Change
  the theme and the account pages, the search and the desk change with it.
- **Your prices and words.** How a price is written, the order prefix, the hold and return days, delivery, VAT and the
  terms, privacy and returns pages are all settings, edited in the desk.

**Take it over when you outgrow it.** When your shop needs to work differently at its core, copy this package's
`model/` into your app, drop the `use Osysharp.Shop` line and the grants, and it is your code. From then on you are off
the package: you get no updates, but you can change anything. Nothing in it is privileged. It is ordinary Osy#, so
the copy compiles exactly as the package did.

## Source and package

The source is this repository, `github.com/osysharp/shop`, and each version is a GitHub release carrying the
package as `package.zip`. `use Osysharp.Shop@0;` resolves it. It depends on `Osysharp.Stripe`, `Osysharp.Klarna`,
`Osysharp.Resend` and `Osysharp.PdfWriter`, which resolve the same way.

`tests/` is the producer's own project: 84 tests over the whole flow (placing, paying, lapsing, sending, refunding,
returning, the desk and the account pages), with Stripe, Klarna and Resend stubbed. It does not ship with the package.

```console
$ cd tests && osy test
✓ 84 passed.
```
