# Labels: design (draft, 2026-11-10)

A Payload plugin for internal labels: design a label once with field tokens, then print one, a few, or every label that belongs to a document, on a thermal roll or an Avery-style sheet. It works on any Payload site with no other plugin. When Kitting, Restock, the Captive Origins kits code or OneTaxonomy are on the same site, they offer ready-made templates and tokens through a small adapter, with no import between packages.

Package: `@captiveorigins/payload-plugin-labels`, repo `payload-plugin-labels`, MIT. Build starts after tour v3 (owner, 2026-11-10). This is a design, not built code. Nothing here is tested yet.

## What the owner asked for (2026-11-10)

- A "print X label" choice from a dropdown, so not every label for an order prints.
- "Print all labels for this order" and "print all labels for this kit".
- A configurable back end.
- Its own Payload plugin, working with ours without depending on them.
- Both thermal (one label after another) and sheet (Avery-style) printing.
- A label builder in the admin: pick the label size and printer or sheet type, see what's going on, and place field tokens so labels always carry live data, never hardcoded text.

## One layout engine, three outputs

The core is one pure TypeScript function: `layout(template, data, media)` returns a list of positioned marks (text runs, barcodes, images, lines, boxes) in millimeters. Every output draws that same list:

- **SVG** in the browser, for the builder's live preview and the print dialog's preview.
- **PDF** on the server (pdfkit, MIT, already used by Restock), for printing.
- **ZPL** on the server, for Zebra printers (phase 3).

Because the preview and the PDF come from the same list, what you see in the builder is what prints. A test checks this by comparing the mark list behind both.

Barcodes come from bwip-js (MIT, pure JavaScript): Code 128, Code 39, EAN-13, UPC-A, QR and Data Matrix, drawn as vectors so they stay sharp at any printer resolution.

## Data model

### `label-templates` (collection)

| Field | What it holds |
| --- | --- |
| `name`, `key` | A name people see; a key that's unique per site, for code and adapters |
| `collection` | The collection it prints from, such as `orders` or `kits` (a select built from the site's collections) |
| `size` | Width and height, in mm or inches as entered. Stored in mm |
| `media` | Default media (below); the print dialog can change it |
| `elements` | The layout: an ordered list of elements (below) |
| `repeat` | Optional: one label per row of an array field, such as one per order line |
| `copies` | Default copies per document (1) |
| `active`, `notes` | |

Each element has a type (`text`, `barcode`, `qr`, `image`, `line`, `box`), a position and size in mm, a rotation of 0, 90, 180 or 270 degrees, and its own settings:
- **Text:** content with tokens, font, size, weight, italic, alignment, and what happens when it doesn't fit (shrink to fit, cut with an ellipsis, or wrap to a maximum number of lines).
- **Barcode:** symbology, value with tokens, human-readable text on or off.
- **Image:** an upload, or a token naming an upload field.

### `label-media` (collection)

What you print on: a thermal roll or a sheet.

- **Thermal:** label width and height, gap, printer resolution (203, 300 or 600 dpi), and a calibration offset in x and y.
- **Sheet:** page size (US Letter or A4), columns, rows, label width and height, top and left margins, horizontal and vertical pitch, corner radius (for the preview only), and a calibration offset.

The plugin ships presets as code: Avery 5160, 5163, 5167 and 94207, plus A4 L7160 sheets, and the common thermal sizes (4 × 6 in first, then 2 × 1 in, 2.25 × 1.25 in and DYMO 30252). A preset is copied into an editable record only when someone picks it, so nothing appears that nobody asked for. A **calibration page** (rulers and corner marks) shows how far a printer is off, and the offset fixes it.

### `label-prints` (collection, a log)

Who printed which template, for which documents, how many, on which media, and when. It stores ids and counts only, never the label's text, so it holds no copy of names or addresses. It allows reprinting and "printed already" marks. Its retention is a setting (default 180 days), and it can be switched off.

### With `scopeField`

As in Kitting: templates and media belong to a site, and a template with no site is shared by all of them, for a platform admin to manage. Printing only reads documents from the printing person's own site.

## Field tokens

- `{{name}}`, `{{order.customer.email}}`: dotted paths, following relationships up to 3 deep.
- Formatters: `{{createdAt | date:"MMM D, YYYY"}}`, `{{total | money}}`, `{{name | upper}}`, `{{notes | default:"–"}}`, `{{code | slice:0:8}}`.
- Label context: `{{label.number}}` and `{{label.of}}` (for example "3 of 12"), `{{printedAt}}`, `{{printedBy.name}}`, `{{site.name}}`.
- Adapter tokens, namespaced: `{{kitting.lotsUsed}}`, `{{kits.returnBy}}`.

Rules:
- **Tokens never run code.** They're parsed by a small grammar, never by `eval` or a template language that can run code.
- **Tokens read as the person printing** (`overrideAccess: false`). A field they can't read prints empty, and the preview warns. A label can never show more than the person could see in the admin.
- **A missing value is shown in the preview,** in color, never silently blank.

## The builder (admin)

A custom view on each template, laid out like a small design tool:

- **Left: elements.** Text, Field, Barcode, QR, Image, Line, Box. "Field" opens a searchable list of the collection's fields, with relationships expandable, and inserts the token.
- **Center: canvas.** The label at true scale with a zoom control, rulers in mm or inches, a snap grid, and a dashed line for the printer's safe area. Drag to move, handles to resize. The arrow keys nudge by 0.1 mm, or 1 mm with Shift.
- **Right: inspector.** Every property of the selected element as plain form fields: position, size, font, symbology, overflow rule.
- **Layers list.** Every element as a row you can reorder and select. With the layers list and the inspector, everything can be done by keyboard and with a screen reader; the canvas is never the only way.
- **Preview with a real document.** Pick, say, "Order 1042", and the canvas fills its tokens in. Text that doesn't fit, a barcode too dense for the printer's resolution, and a mark outside the safe area each show a warning beside the element.
- **Media switcher.** Thermal or a sheet. The sheet preview shows the whole page, so you can see the label in its grid.

Before the drag-and-drop canvas exists (phase 1), the same template is edited through the inspector and layers list, with the live preview beside it. The canvas is added on top in phase 2 and changes nothing underneath.

## Printing

- **On a document:** a "Print labels" button in the document's controls. Its menu lists:
  - the active templates for this collection ("Print kit label");
  - the **sets** that reach this document ("All labels for this order").
- **On a list:** select rows, then "Print labels".
- **The print dialog:**
  - which template or set;
  - which items, as a checklist (for example, which kits of an order), all ticked by default;
  - copies;
  - media;
  - for sheets, the **start position**: click the first free spot on a picture of the sheet, to use up a partly used one;
  - a preview of every page;
  - then **Download PDF** or **Open to print**.
  The dialog reminds you to print at 100% ("Actual size"), because browsers shrink pages by default. The last media used is remembered per person in Payload's preferences.
- **Sets** are what "all labels for this order" means. A set is a named list of steps on one collection. Each step is a template, either for the document itself or for related documents reached through a relationship: the order's box label, then one tube label for each kit whose `order` is this order. Sets come from presets, or are built in the admin like templates. Without a set, the menu still offers each template on its own.

Server endpoint: `POST /api/labels/print` with `{ template | set, ids, media, start, copies, format }` returns `application/pdf` (or ZPL). It checks the person may read every document and may print, and refuses with a plain message otherwise. Its limit is 2,000 labels per request, and anything larger is refused with that number. Rendering 2,000 labels to PDF should take a few seconds; that number is a guess until it's measured.

Configuration in the plugin options:

```ts
labelsPlugin({
  collections: ['orders', 'kits', 'supply-items'], // where print buttons appear (default: every collection a template targets)
  access: { print, manage },                       // plain Payload access functions
  scopeField: 'tenant',
  formats: { zpl: true },                          // phase 3
  log: { enabled: true, keepDays: 180 },
})
```

## Outputs

| Output | Thermal | Sheet | Notes |
| --- | --- | --- | --- |
| PDF | One label per page, the page the label's size | US Letter or A4 pages in the media's grid, from the start position | Works with Zebra, Rollo, DYMO and Brother drivers on Mac and Windows, and with any office printer for sheets |
| ZPL (phase 3) | Yes | No | Text, lines, boxes, Code 128, QR and Data Matrix. Images are refused in ZPL with a message to use PDF. Downloaded as a `.zpl` file |
| Sending straight to a network printer | Later, opt-in, off by default | | Needs the server to reach the printer. Not in the first versions |

## Adapters: working with our other plugins without depending on them

The same pattern as Kitting and Restock (`config.custom.restock`, `config.custom.kitting`): no package imports another.

- Labels publishes a contract, `src/contracts/labels.ts` (types only, MIT), and reads `config.custom.labels.providers`. That's a plain object any plugin or site can add to, in any plugin order, because Labels reads it at request time:

```ts
type LabelProvider = {
  name: string                       // "Kitting"
  templates?: LabelTemplatePreset[]  // ready-made templates, offered in the template library
  sets?: LabelSetPreset[]            // "All labels for this order"
  tokens?: (args: { collection: string; doc: object; payload; req }) => Promise<Record<string, unknown>> // extra tokens under the provider's namespace
}
```

- Each plugin ships its own provider, in its own repo, behind a check that does nothing when Labels isn't there. Labels ships with none, so it stays agnostic.
- **Presets are offered, never forced.** The template library lists every provider's presets, with "Add". A preset is copied into an editable template when someone clicks it. Nothing is created at startup.

Presets in the first round:

| Provider | Templates | Sets |
| --- | --- | --- |
| Kitting | Built unit (external ID barcode, recipe, built on, lots, use-by), supply lot (item, lot, use-by), shelf label (name, SKU, reorder level, QR to its admin page) | All labels for a build batch |
| Restock | Receiving label (PO number, supplier, item, received on, lot) | All labels for a received order |
| Captive Origins kits (the site's own code, in the platform) | Kit bag label (4 × 6 in thermal: kit code as Code 128 plus text, lot, expiry), kit box label, shipping box label (the box is reused for the return), batch label | All labels for this order (shipping box plus each kit's bag and box), all labels for this kit (bag and box) |
| OneTaxonomy (wherever its Payload side lives) | Specimen or animal label (scientific name in italics, common name, ID, QR) | |

Carrier shipping labels are not this plugin's job: they come from the carrier (EasyPost) as their own PDFs. A set can still list them, through a provider step that hands over an existing PDF. That would be phase 3, and only if wanted.

## Tests (planned)

- **Pure:**
  - token grammar and formatters, including refusing anything that looks like code;
  - unit conversion;
  - sheet grid positions, with every start position and offset;
  - text fitting;
  - barcode values that are valid and invalid per symbology;
  - the same mark list behind SVG and PDF;
  - ZPL output against known strings.
- **Integration**, on a real MongoDB in memory:
  - access: no print without read access, and a field the person can't read prints empty;
  - site isolation;
  - sets across relationships;
  - the 2,000-label limit;
  - the log stores ids only;
  - adding a preset copies it;
  - with no providers, the plugin runs alone;
  - with a provider, its tokens and presets appear.
- **PDF checks:** page count, page size, and the label count per page, read back with pdf-lib.
- **Browser, on made-up data:**
  - building a label by keyboard only;
  - the preview matches the PDF;
  - printing a set;
  - a partly used sheet.
- **Real printers:** a thermal printer and a sheet printer are the owner's hands-on test (it goes on Helm's page).

## Phases and rough time

These are rough guesses of session days, not promises.

| Phase | What | Days |
| --- | --- | --- |
| 0.1 | The provider contract (`src/contracts/labels.ts`, types only) on day 1, so others can build against it; engine (tokens, layout, sheets, PDF, barcodes); the three collections; print endpoint; print button and dialog with sets and start position; editing by inspector and layers list with live preview; presets as code; tests | 5–6 |
| 0.2 | The drag-and-drop canvas with rulers, snapping, zoom and warnings; the calibration page | 3–4 |
| 0.3 | ZPL; providers for Kitting, Restock, the site's kits and OneTaxonomy (crossings, each in its own repo); a set step that includes carrier PDFs, if wanted | 2–3 |
| | Total | about 10–13 |

0.1 already does everything the owner listed except dragging elements on the canvas: you can pick the size and media, place token fields, see the label, and print one, some, or all labels for a document. Phase 2 makes the builder visual.

## Owner's answers (2026-11-10)

- **Order:** phased. 0.1 first, then the canvas in 0.2.
- **Thermal printer:** PDF and ZPL. PDF is the default for every printer; ZPL is a proper output in 0.3.
- **Name:** Labels, `@captiveorigins/payload-plugin-labels`.

The exact words are in [decisions.md](decisions.md).
