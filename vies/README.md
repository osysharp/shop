# Osysharp.Shop.Vies — EU VAT-number checks for a shop that sells VAT-free to businesses in other EU countries

## Use case

A business customer in another EU country buys without VAT (reverse charge) — but only once its VAT number is
confirmed. `Osysharp.Shop` asks whoever its `ShopSetup.VatNumbers` names; this kit answers from the European
Commission's VIES register, which every member state's VAT numbers are checked against. It returns whether the number
is valid, and the registered name and address where the member state publishes them.

## Install and use

```osy
app MyShop {
  use Osysharp.Shop@1 { … }
  use Osysharp.Shop.Vies@0 { egress "ec.europa.eu"; }
  …
}
```

```osy
using Osysharp.Shop.Vies;

app.Shop = new ShopSetup { …, VatNumbers = new ViesCheck() };
```

Staff then make an account wholesale with its VAT number checked, and the shop prices that account's orders without
VAT where the rules say so. A VIES answer that is not a success is refused with the register's status, never taken as
"valid"; a member state that withholds the name and address is answered with both empty.

## Source and package

- Source: `model/Vies.osy` (`ViesCheck : IVatNumberCheck`).
- Tests: `tests/vies.test.osy`, with every call to VIES stubbed.
- Package: `use Osysharp.Shop.Vies@0;` — `osy lock` fetches the release archive from this repository's releases.
