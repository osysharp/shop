# Osysharp.Shop.Reviews

Product reviews for [Osysharp.Shop](https://osyrin.com/templates/kits/shop/): written by customers who bought the product, read by staff before they go
up, averaged on the product page and in what a search engine reads. An add-on — list it, put the panel on your product
page and the desk list on a staff page.

- **Only a customer who bought it** — a signed-in customer with a paid order holding the product (paid, being packed,
  sent or delivered — not one merely reserved). One review each, which they may change. The rule is the entity's own
  `security { }`, so it holds however a review arrives: the page, a function, the API.
- **Staff read them first** — a new review waits until staff publish it, or the shop publishes at once (a switch on
  the desk; default: wait). Staff hide one with a reason, which its author sees. A customer never publishes their own:
  the review's state belongs to its workflow, and publishing is an event only staff may raise.
- **An edit goes back for reading** — a published review its author changes waits for staff again (unless the shop
  publishes at once). Nobody else can change it.
- **The average counts published reviews only** — `ReviewSummaryOf(product)`: the average to one decimal and how many.
- **A page at a time** — the product page shows the newest twenty reviews and a "Show more reviews" button for the rest,
  so a product with thousands of reviews opens as fast as one with three.
- **In the search result** — the product's structured data (`ProductStructuredData`) carries schema.org
  `aggregateRating` once it has published reviews, through the shop's add-on seam: nothing to wire.
- **Erased with the customer** — a customer who asks to be erased takes their reviews with them.

## Use it

```osy
// app.osy
use Osysharp.Shop@1 { egress "api.resend.com"; secret "ResendApiKey"; }
use Osysharp.Shop.Reviews@0;
```

That `use` line is all: the shop finds the add-on (`ProductReviews`) for every product's rating.

```osy
// the product page: the average, the reviews, and the form for a customer who bought it
ProductReviewsPanel(product, heading: "What people say");
```

Staff read and publish the reviews on the kit's own page, `/staff/reviews` (`ReviewsDeskPage`), which stands in the
store's staff frame (`WorkspaceShell`) and its navigation (`[Nav]`). The panel itself is `DeskReviews()`, for a page of
your own.

The panel is written in the theme's tokens — `Font.Display` for its heading and each review's title, `FontSize.Title`,
`Colors.Surface` behind the customer's own review, `Colors.Border` between reviews — so a store's theme dresses it. A
store that wants its own composition builds it from the same functions.

From code: `WriteReview(product, rating, title, text)`, `ReviseReview(review, rating, title, text)`, `MyReview(product)`,
`BoughtIt(product)`, `PublishedReviews(product, take = 20)` (newest first), `ReviewSummaryOf(product)`; for staff `ApproveReview(review)`,
`HideReview(review, reason)`, `SetReviewsGoLiveAtOnce(bool)`.

## How it is built

`ProductReview` is one row per customer and product (`[Unique(Product, Author)]`). Its `security { }` lets a customer
create only their own review of a product they bought (`HasBought`), and nothing else: its `State` is tracked by
`ReviewLifecycle` — Waiting, Published, Hidden — whose `Approve` and `Hide` only staff may raise and whose `Revise`
only the author may. `ProductReviews` is an ordinary `IShopAddOn`, found by the shop: `Rating(product)` answers the aggregate for the
structured data and `CustomerErased(account)` removes the customer's reviews.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Shop.Reviews;
using Osysharp.Shop;

[Test(Reviewed)]
void the_average_reads_the_same_however_many_reviews_a_product_has() {
  var tea = Called("tea");
  var beans = Called("beans");
  ReviewSummaryOf(Called("cake"));   // the first read sets up what every later one reuses — not what this compares
  var mark = Cost.Mark();
  var two = ReviewSummaryOf(tea);
  var twoRows = Cost.RowsReadSince(mark);
  mark = Cost.Mark();
  var thirty = ReviewSummaryOf(beans);
  var thirtyRows = Cost.RowsReadSince(mark);
  Log.Information("The average: " + twoRows + " rows read for two reviews, " + thirtyRows + " for thirty");
  Assert.Equal(2, two.Count);
  Assert.Equal(5m, two.Average);
  Assert.Equal(30, thirty.Count);
  Assert.Equal(3m, thirty.Average);   // ratings 2,3,4,5,1 six times over: 90 / 30
  Assert.Equal(twoRows, thirtyRows);
}
```

From [`tests/cost.test.osy`](tests/cost.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Shop.Reviews;
using Osysharp.Shop;
using Osysharp.Workflow;

[Test(Shop)]
void a_buyer_writes_a_review_that_waits_until_staff_publish_it() {
  MajaBoughtBeans();
  runas (Maja) {
    var mine = WriteReview(Beans(), 5, "Bright and sweet", "Like blackcurrant jam on toast.");
    Assert.Equal(ReviewState.Waiting, mine.State);
    Assert.Equal("Maja B.", mine.ShownAs);                             // signed by the workflow, never typed
    Assert.Equal("Bright and sweet", MyReview(Beans()).Title);        // she sees her own while it waits
  }
  Assert.Equal(0, PublishedReviews(Beans()).Count);                  // nobody else does yet
  Assert.Equal(0, ProductReview.Count());
  Assert.Equal(0, ReviewSummaryOf(Beans()).Count);
  runas (Desk) { ApproveReview(ProductReview.Single()); }
  var shown = PublishedReviews(Beans());
  Assert.Equal(1, shown.Count);
  Assert.Equal("Like blackcurrant jam on toast.", shown[0].Text);
  Assert.Equal(5m, ReviewSummaryOf(Beans()).Average);
  Assert.NotNull(shown[0].PublishedAt);
}
```

From [`tests/reviews.test.osy`](tests/reviews.test.osy).

```osy
using Osysharp.Shop.Reviews;
using Osysharp.Shop;
using Osysharp.Ui;

[Test(Shop)]
void the_kit_announces_its_reviews_section_to_staff_only() {
  runas (Desk) {
    var link = AnnouncedSettings().Single(l => l.To == "/settings/reviews");
    Assert.Equal("Reviews", link.Label);
    Assert.Equal("Shop", link.Group);
  }
  runas (Maja) { Assert.False(AnnouncedSettings().Any(l => l.To == "/settings/reviews")); }
}
```

From [`tests/settings.test.osy`](tests/settings.test.osy).

## Learn more

- [Osysharp.Shop.Reviews in the package catalogue](https://osysharp.com/packages/#osysharp-shop-reviews)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
