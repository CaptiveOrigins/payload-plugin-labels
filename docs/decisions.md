# Decisions

Dated decisions for this repo, newest last.

- 2026-11-10: the owner asked for a label tool for internal labels: print one label from a menu, or every label that belongs to an order or a kit, with a configurable backend. It is its own Payload plugin and works with or without the other plugins in this set.
- 2026-11-10: it prints to thermal label printers in sequence and to Avery-style sheets. A label builder in the admin shows the label, its size and its printer or sheet type, and fills areas from field tokens so the content always comes from the document.
- 2026-11-10: the build starts after the guided tour work. MIT, like the other general Payload plugins. Design in [design.md](design.md); not built yet.
- 2026-11-10: answers to design.md's three questions:
  - Order: phased. 0.1 ships printing with inspector editing and live preview; the drag-and-drop canvas follows in 0.2.
  - Thermal printer: PDF and ZPL. ZPL is Zebra's language and the most common one for thermal label printers in shipping and labs, so ZPL is a proper output in 0.3. PDF stays the default for every printer.
  - Name: Labels.
- 2026-11-10: the repo is public from its first push, MIT, with the first-public-push audit ([first-public-push-audit.md](first-public-push-audit.md)) and the leak check in CI.
- 2026-11-10: there's no QR on the tube. The kit labels are the kit's outer bag (about 4 × 6 in), the kit's box, and the shipping box, which is reused for the return. So the 4 × 6 in thermal preset comes first, and the kits presets are bag, box, shipping box and batch. The activation QR goes on the kit bag label only.
- 2026-11-10: the provider contract moves to the first day of 0.1 (it was 0.3), so a kits plugin can build its templates against it while Labels is built.
- 2026-11-10: build it right. Labels imports no other plugin; other plugins reach it through optional adapters. No plugin-family tags in `package.json` keywords yet.
- 2026-11-11: categories are `admin-ui` and `ecommerce`. GitHub topics: payload, payload-plugin, payloadcms, admin-ui, ecommerce, labels, printing, barcode, qr-code, thermal-printer, pdf. When the package is created, its `keywords` are exactly these, in this order.
- 2026-11-14: moved to the ExtantApe GitHub organization. The package will be `@extantape/payload-plugin-labels`.
