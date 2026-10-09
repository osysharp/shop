# Osysharp.Shop.Ecb — the European Central Bank's daily rates, for prices in a visitor's own currency

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
  use Osysharp.Shop.Ecb@0 { egress "www.ecb.europa.eu"; }
  …
}
```

```osy
using Osysharp.Shop.Ecb;

app.Shop = new ShopSetup { …, ExchangeRates = new EcbRates(), DisplayCurrencies = ["EUR", "NOK", "DKK", "USD"] };
```

The shop's rate job asks the ECB as it starts and every six hours; cross rates against the shop's own currency are
worked out from the euro sheet (1 SEK in USD = USD per euro ÷ SEK per euro). A sheet with no date or no rates — a
maintenance page served as 200 — is refused rather than stored.

## Source and tests

`model/Ecb.osy` (`EcbRates : IExchangeRates`, `EcbSheet(xml)`), and tests in `tests/ecb.test.osy` with every call to
the ECB stubbed.
