# Osysharp.Shop.Looks.Swap

**A look for shops: draws `Osysharp.Site.Looks` and `Osysharp.Shop.Looks`.** Its own design of all seven parts of a store (the frame and the pages) and of the hero, the footer's words and the product row; the other block types fall back to the shop kit's plain designs, in this look's theme. Its own block: “Every piece”.

A **look** for a shop — "the swap sheet". White, black IBM Plex Sans Condensed with Plex Mono for lot numbers, sizes and prices, the shelves always open in a column, and one accent — signal red — for the thing to press. The catalogue is the front page.

Install it in a store beside other looks, and the people who run the store pick it per version of the site at the page
desk (`/staff/pages`): drafted, previewed as a whole site, switched on a date. Orders, stock, customers and everything
staff arranged stay put — the Swap sheet redraws the same words and photographs in its own design.

![The Second Round shop's front page in the Swap sheet, wide](docs/front-wide.jpg)
![A piece's page in the Swap sheet, on a phone](docs/piece-phone.jpg)

- **The frame:** the kit's CatalogueShell — the whole shelf tree open in a column beside the page, the bag at its foot; the menu (`SiteMenu("menu", …)`), the strip across the top (`Region("announcement")`)
  and the footer's words (`Region("footer")`), each staff's to arrange, under the names every look uses — so what staff
  arranged comes along whichever look the store wears.
- **The pages:** the front page (a region of blocks), a listing, the grid, a piece's page, and a "not here".
- **Its designs of the block types:** the hero (the title at poster size beside a count of the pieces — no photograph), the product row and the footer's words. A type it has no design of is drawn in the shop kit's plain design.
- **Its own block:** Every piece — the whole rail as one grid — drawn only while a version wears the Swap sheet; the desk lists it before a switch to another look.
- **Its theme and fonts:** the palette, type scale, radii and motion, served only while the store wears the Swap sheet
  (`:root[data-look="swap"]`); IBM Plex Sans Condensed and IBM Plex Mono, registered in this kit's `osyrin.lock` and shipped with the store that installs it.
- **Whatever the visitor's system is set to:** every colour the shop kit and its add-ons paint with is the look's own —
  raised panels and wells (`Elevated`, `Sunken`) and the four statuses included — so a visitor in dark mode sees the
  look as it was designed, not the platform's dark menus inside it. `osy kit tokens --app` lists nothing left at the floor.
- **The store's name** in the wordmark and the footer's sign-off is the shop's own (`ShopSettings.Name`, set at the desk).

It draws the general site contract (`Osysharp.Site.Looks`) and the shop's (`Osysharp.Shop.Looks`), with exactly their
parameters — the compiler checks each design like a class against an interface. It carries **no data and no logic** and
needs no egress and no secret: it reads what a store's page reads, as the visitor. Its default front page names the
Second Round catalogue's pieces and photographs (`/_osy/files/public/products/…`); staff replace them at the desk.

- Worn by the Second Round thrift store, whose tests walk the shop in every look at a phone's width and a wide screen's (`osy test --pixels`).
- `osy docs ui-looks` — looks, contracts, and switching a version's look.
- `osy docs ui-building-a-look` — building a look of your own, from an empty kit.
