# SKU Onboarder

A multi-step wizard for turning a KW/IMU export into SFCC-ready catalog and price book XML, with a persisted classification rules engine to cut down repeat manual review week over week.

Part of the [ch-tools](https://github.com/caitlinsc/ch-tools) monorepo.

## Workflow

The tool walks through six steps:

1. **Upload** — drop in a KW/IMU export (`.xls`, `.xlsx`, or `.csv`). If the export includes department/class enrichment columns, filtering is more complete; otherwise you'll see a prompt to add dept/class to the query for full filtering.
2. **Cascade** — items are auto-classified into drop / auto-upload / manual-review buckets based on saved rules.
3. **Review** — manually confirm or adjust items that didn't match an existing rule. Auto-approved items arrive pre-selected.
4. **Group Variations** — combine related SKUs (e.g. color/size variants of the same product) into an SFCC variation group, choosing Color Only, Size Only, or Color + Size.
5. **Edit & Categorize** — assign each product or variation group a category from the full SFCC category tree (searchable, with breadcrumb display), edit display name/description, and review pricing.
6. **Export** — download the generated catalog XML and price book XML.

## SFCC import steps (shown in-app)

1. **Administration → Site Development → Import & Export** — upload the catalog XML.
2. **Import Catalog** — select the file, **MERGE mode**. Creates products and variations.
3. **Import Price Book** — upload and import the price book XML, MERGE mode.
4. **Verify** — search the imported SKUs in staging; check variations and pricing landed correctly.

## What the export contains

- **Catalog XML** (`catalog-id="master-cheshirehorse"`) — one `<product>` per SKU or variation group, with display name, description, online/searchable flags, UPC, manufacturer SKU, brand, tax class, and weight (as a custom attribute). Variation groups include a `<variations>` block with color/size attributes mapped per-category (see below), plus `<category-assignment>` entries for every categorized product or group.
- **Price book XML** (`pricebook-id="ch-list-prices"`) — one `<price-table>` per SKU with a selected retail price.

### Category → variation attribute mapping

SFCC uses different custom attribute IDs for color/size depending on product category (e.g. `size_footwear` vs `size_apparel` vs `size_helmet`). This mapping is hardcoded in `index.html` (`VARIATION_ATTR_MAP` and `VARIATION_ATTR_OVERRIDES`) — if a new category needs its own variation attributes, that's the dictionary to edit.

## Rules engine

Classification rules persist via `window.storage` (key `ch-onboarder-rules`), so once you've classified an item or class as auto-upload/drop, it's remembered for next time and the manual review pile should shrink.

**⚠️ Important:** `window.storage` is the persistence API provided by the Claude Artifacts runtime — it is *not* a standard browser API. If this tool is opened as a static file from GitHub Pages (outside Claude.ai), `window.storage` will be undefined and rule persistence will silently fail (caught by a try/catch, so the tool won't crash, but rules won't save between sessions). Variation-grouping preferences (`ch-var-prefs`) have the same dependency. If you need this to work standalone, swap these calls for `localStorage` or another browser-native storage mechanism.

## Stray file in this folder

`sku-onboarder.jsx` is not part of this tool — it's an unrelated React component for reviewing RPH exports (uses `rph-rules`/`rph-history` storage keys, references "Upload RPH Export"). It looks like it was saved into the wrong folder. Safe to move or delete unless something elsewhere depends on it.

## Built with

React 18 + Babel Standalone (both via CDN, no build step) and [SheetJS (xlsx)](https://github.com/SheetJS/sheetjs) for reading the KW/IMU export.
