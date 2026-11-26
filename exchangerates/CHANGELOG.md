# Osysharp.Shop.ExchangeRates — changes

## 1.0.0
- Grouped into classes (`osy restructure`). Calls move:
  - `EcbSheet(…)` → `EcbRates.Sheet(…)`
- Published tools, actions and actors keep their names: what calls them by name is unchanged.
- A project moves when it bumps the kit: `use Osysharp.Shop.ExchangeRates@1;`, then the calls above.
