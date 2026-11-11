# payload-plugin-labels

Labels for Payload: design labels with field tokens in the admin, then print one, a few or every label for a document, on thermal rolls or Avery-style sheets.

**Status:** design only. No code yet, so nothing is tested. The build starts after a planned site tour, in three phases: 0.1 prints PDFs with form-based editing and a live preview, 0.2 adds a drag-and-drop label editor, 0.3 adds ZPL for Zebra printers. The design is in [docs/design.md](docs/design.md).

## What it will do

- A label builder in the admin: pick the label size and the roll or sheet, then place text, barcodes, QR codes, images and lines. Field tokens such as `{{order.customer.name}}` fill them from the document, so labels always carry live data.
- A "Print labels" menu on each document: print one label, a few, or every label that belongs to it, such as an order's box label and one label per kit.
- Output as PDF for any printer: one label per page for thermal printers, or a sheet grid (Avery 5160 and others) with a start position for partly used sheets. ZPL for Zebra printers comes in 0.3.
- Labels show only fields the person printing may read.
- Works on any Payload site by itself. Other plugins can offer ready-made labels through a small adapter, with no dependency either way.

## On its own, and with others

Nothing here is built yet, so this is the plan from [docs/design.md](docs/design.md).

**On its own:** Labels will work on any Payload site with no other plugin. You design labels in the admin from your own collections' fields, and print them from each document.

**Works with** (planned for 0.3, each through `config.custom.labels.providers`, with no import either way):
- **Kitting:** ready-made labels for built units, supply lots and shelves.
- **Restock:** a receiving label for each received order.
- **Sample Kits:** kit bag, kit box, shipping box and lab batch labels. Its `adapters.labels` is already typed for this.
- **OneTaxonomy:** offered through the same kind of adapter.

**Family:** Labels sits in the Fulfillment family, with Kitting, Restock, Sample Kits, Shipping, Scan, Wizards, Custody and Sequences, though the families are a working draft and Fulfillment is due to be split into different groups.

## Install or build

Not published yet. It will be `@captiveorigins/payload-plugin-labels` on npm.

## License

MIT. See [LICENSE](LICENSE).
