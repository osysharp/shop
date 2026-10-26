# Osysharp.Shop.VatNumbers — EU VAT-number checks for a shop that sells VAT-free to businesses in other EU countries

## Use case

A business customer in another EU country buys without VAT (reverse charge) — but only once its VAT number is
confirmed. `Osysharp.Shop` asks whoever its `ShopSetup.VatNumbers` names; this kit answers from the European
Commission's VIES register, which every member state's VAT numbers are checked against. It returns whether the number
is valid, and the registered name and address where the member state publishes them.

## Install and use

```osy
app MyShop {
  use Osysharp.Shop@1 { … }
  use Osysharp.Shop.VatNumbers@0 { egress "ec.europa.eu"; }
  …
}
```

```osy
using Osysharp.Shop.VatNumbers;

app.Shop = new ShopSetup { …, VatNumbers = new ViesCheck() };
```

Staff then make an account wholesale with its VAT number checked, and the shop prices that account's orders without
VAT where the rules say so. A VIES answer that is not a success is refused with the register's status, never taken as
"valid"; a member state that withholds the name and address is answered with both empty.

## Source and package

- Source: `model/Vies.osy` (`ViesCheck : IVatNumberCheck`).
- Tests: `tests/vies.test.osy`, with every call to VIES stubbed.
- Package: `use Osysharp.Shop.VatNumbers@0;` — `osy lock` fetches the release archive from this repository's releases.

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.VatNumbers;

[Test]
void a_number_is_checked_with_its_country_and_the_company_named() {
  var posted = new List<string>();
  var urls = new List<string>();
  Http.Post.Stub((url, body) => {
    urls.Add(url); posted.Add(body);
    return new HttpResponse { StatusCode = 200, Body = "{\"countryCode\":\"DE\",\"vatNumber\":\"123456789\",\"valid\":true,\"name\":\"Muster GmbH \",\"address\":\"Hauptstraße 1\"}" };
  });
  var result = new ViesCheck().Check("DE", "123456789");
  Assert.Equal("https://ec.europa.eu/taxation_customs/vies/rest-api/check-vat-number", urls[0]);
  Assert.Equal("{\"countryCode\":\"DE\",\"vatNumber\":\"123456789\"}", posted[0]);
  Assert.True(result.Valid);
  Assert.Equal("Muster GmbH", result.Name);
}
```

From [`tests/vies.test.osy`](tests/vies.test.osy).

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.VatNumbers;

[Test]
void a_withheld_name_is_empty_and_an_invalid_number_is_not_valid() {
  Http.Post.Stub((url, body) => new HttpResponse { StatusCode = 200, Body = "{\"valid\":false,\"name\":\"---\",\"address\":\"---\"}" });
  var result = new ViesCheck().Check("FR", "00000000000");
  Assert.False(result.Valid);
  Assert.Equal("", result.Name);
}
```

From [`tests/vies.test.osy`](tests/vies.test.osy).

```osy
using Osysharp.Http;
using Osysharp.Shop;
using Osysharp.Shop.VatNumbers;

[Test]
void an_unreachable_register_says_so() {
  Http.Post.Stub((url, body) => new HttpResponse { StatusCode = 503, Body = "" });
  Assert.Throws<ExternalServiceException>(() => new ViesCheck().Check("DE", "123456789"));
}
```

From [`tests/vies.test.osy`](tests/vies.test.osy).

## Learn more

- [Osysharp.Shop.VatNumbers in the package catalogue](https://osysharp.com/packages/#osysharp-shop-vatnumbers)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
