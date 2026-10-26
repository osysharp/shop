# Osysharp.Shop.ExchangeRates — the European Central Bank's daily rates, for prices in a visitor's own currency

## Use case

A Swedish shop keeps its books in kronor, and a visitor from Oslo or Berlin still wants to know what a bag of coffee
costs in their money. `Osysharp.Shop` shows it as an estimate ("≈ €14.58", and "charged as 165 kr" wherever money
changes hands) from rates a provider supplies. This kit is that provider: the ECB's euro foreign exchange reference
rates, published once a working day at about 16:00 CET — public, free, no key.

Source: <https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html>,
read from <https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml>. The ECB publishes these rates for
information purposes only, which is how the shop uses them: nothing is charged at them.

## Install and use

```osy
app MyShop {
  use Osysharp.Shop@1 { egress "api.resend.com"; secret "ResendApiKey"; }
  use Osysharp.Shop.ExchangeRates@0 { egress "www.ecb.europa.eu"; }
  …
}
```

```osy
using Osysharp.Shop.ExchangeRates;

app.Shop = new ShopSetup { …, ExchangeRates = new EcbRates(), DisplayCurrencies = ["EUR", "NOK", "DKK", "USD"] };
```

The shop's rate job asks the ECB as it starts and every six hours; cross rates against the shop's own currency are
worked out from the euro sheet (1 SEK in USD = USD per euro ÷ SEK per euro). A sheet with no date or no rates — a
maintenance page served as 200 — is refused rather than stored.

## Source and tests

`model/Ecb.osy` (`EcbRates : IExchangeRates`, `EcbSheet(xml)`), and tests in `tests/ecb.test.osy` with every call to
the ECB stubbed.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.ExchangeRates;

[Test]
void the_daily_sheet_is_read_against_the_euro_with_its_date() {
  List<string> asked = [];
  Http.Get.Stub((url, headers) => { asked.Add(url); return new HttpResponse { StatusCode = 200, Body = EcbDaily() }; });
  var sheet = new EcbRates().Fetch();
  Assert.Equal("https://www.ecb.europa.eu/stats/eurofxref/eurofxref-daily.xml", asked[0]);
  Assert.Equal("EUR", sheet.Base);
  Assert.Equal(new DateTime(2026, 9, 28), sheet.Date);
  Assert.Equal(5, sheet.Quotes.Count);
  Assert.Equal(11.3205m, sheet.Quotes.Single(q => q.Currency == "SEK").Rate);
  Assert.Equal(178.50m, sheet.Quotes.Single(q => q.Currency == "JPY").Rate);
  Assert.Equal("European Central Bank", new EcbRates().Source);
}
```

From [`tests/ecb.test.osy`](tests/ecb.test.osy).

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.ExchangeRates;

[Test]
void an_unreachable_bank_says_so() {
  Http.Get.Stub((url, headers) => new HttpResponse { StatusCode = 503, Body = "" });
  Assert.Throws<ExternalServiceException>(() => new EcbRates().Fetch());
}
```

From [`tests/ecb.test.osy`](tests/ecb.test.osy).

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.ExchangeRates;

[Test]
void a_page_that_is_not_the_sheet_is_not_a_day_of_rates() {
  Http.Get.Stub((url, headers) => new HttpResponse { StatusCode = 200, Body = "<html><body>Down for maintenance</body></html>" });
  Assert.Throws<ExternalServiceException>(() => new EcbRates().Fetch());
  Assert.Throws<ExternalServiceException>(() => EcbSheet("<Cube time='2026-09-28'></Cube>"));
}
```

From [`tests/ecb.test.osy`](tests/ecb.test.osy).

## Learn more

- [Osysharp.Shop.ExchangeRates in the package catalogue](https://osysharp.com/packages/#osysharp-shop-exchangerates)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
