# Decisions

Dated decisions for this repo, newest last.

- 2026-11-10: created from the ops starter kit, area `captive-origins`.
- 2026-11-10 (owner, relayed verbatim by Helm from its chat, about 1:40 PM MST): "Another thing we will need: a label generation tool for our internal labels. I've used a site to upload csv data to do it programmatically before, but I'd rather just be able to click "print X label" from a drop down (so not all order labels are printed) but also a print all labels associated with this order / print all labels associated with this kit, etc. and then have that backend piece be configurable. I'm thinking this will be its own payload plugin too, but it should tightly interface with our existing ones agnostically. So others can use it without our other plugins, but it should just work seamlessly"
- 2026-11-10 (owner, same relay), on what it prints on: "Thermal label printer, Sheets (Avery-style), Support both thermal sequential printing AND sheet printing so we have options and so do the others. We should have a label builder in the UI so you can visualize what's going on, select the label size and printer / sheet type, and then be able to natively use field tokens to dynamically populate areas so that it always has dynamic info vs hardcoded"
- 2026-11-10 (owner): build it after tour v3 ("After the tour (Recommended)"). MIT, like our general Payload plugins. Design in [design.md](design.md); not built yet.
- 2026-11-10 (owner, relayed verbatim by Helm from its chat, about 1:55 PM MST), answering design.md's three questions:
  - Order: "Phased (Recommended)". 0.1 ships printing with inspector editing and live preview; the drag-and-drop canvas follows in 0.2.
  - Thermal printer: "PDF + ZPL (if it's widely used)". ZPL is Zebra's language and the most common one for thermal label printers in shipping and labs, so ZPL is a proper output in 0.3. PDF stays the default for every printer.
  - Name: "Labels (Recommended)": `@captiveorigins/payload-plugin-labels`.
- 2026-11-10 (Helm): this session (Kitting) leads the repo. The build starts after tour v3.
- 2026-11-10 (owner, relayed verbatim by Helm from its chat, about 2:10 PM MST): "No private repos because CI". So this repo is public on GitHub under CaptiveOrigins from its first push, MIT, with the first-public-push audit ([first-public-push-audit.md](first-public-push-audit.md)) and the leak check in CI. It replaces the earlier plan to keep the repo local and private until the build.
- 2026-11-10 (owner, relayed by Helm): there's no QR on the tube. The kit labels are the kit's outer bag (about 4 × 6 in), the kit's box, and the shipping box, which is reused for the return. So the 4 × 6 in thermal preset comes first, and the kits presets are bag, box, shipping box and batch.
- 2026-11-10: the provider contract moves to the first day of 0.1 (it was 0.3), so Kits can build its templates against it while Labels is built.

## 2026-11-10: decision feed acknowledged (ops/decisions/all.md, captive-origins.md)

Settled decisions from the owner, through Helm's feed. The feed holds the owner's exact words.

- **D1** Settled decisions arrive through the feed and are acted on, not re-argued.
- **D2** Build it right: agnostic plugins with optional adapters, best architecture first. This repo imports no other Captive Origins plugin.
- **D3** Owner steps and tests go to `~/Code/ops/inbox/` per `ops/rules/owner-items.md`, never straight to needs-you.md or tests.md.
- **D4** Priority: Docker media, Minecraft server, then all Captive Origins.
- **D5** No private repos to save CI minutes. This repo is public, with the Actions-minutes rules.
- **D6, D21** Plugin names need the owner's OK before work starts (ops/plugin-names.md). "Labels" is approved (2026-11-10).
- **D7** No launch dates: work follows the priority list in ops/timeline-captive-origins.md. A freeze point is cut only when the owner says so.
- **D8** Seed World hears about every collection-structure change ahead of time, with versions, and audits when it lands.
- **D12** No plugin-family tags yet. None are in `package.json` keywords.
- **D13** The activation QR goes on the kit bag label only, not the kit box label. The Kits presets in [design.md](design.md) say so.
- Not for this repo: D9 (PM doc), D10 (tour before studbook), D11 (kids decide on guardian suggestions), D14 (plan B), D15 (Husbandry.Pro link), D16 (both views).
- 2026-11-11 (Plugins Lead): categories are `admin-ui` and `ecommerce` (the fixed list in captive-origins `docs/plugin-tags.md`). GitHub topics set by hand for now, since there's no `package.json` yet: payload, payload-plugin, payloadcms, admin-ui, ecommerce, labels, printing, barcode, qr-code, thermal-printer, pdf. When the package is created, its `keywords` must be exactly these, in this order, so `packages:sync` keeps them.
