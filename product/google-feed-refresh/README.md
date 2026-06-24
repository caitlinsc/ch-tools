# Google Shopping Feed Builder (SFCC → GMC)

Generates a Google Shopping-compliant TSV feed from Salesforce Commerce Cloud (B2C) catalog and price book XML exports, for manual upload to Google Merchant Center.

Built as a patch solution while the automated SFTP feed job is broken.

Part of the [ch-tools](https://github.com/caitlinsc/ch-tools) monorepo.

## Usage — browser tool (recommended)

Open `index.html`. Drop in the catalog XML and price book XML exports, click **Build Feed**, review the summary/preview, and download the resulting `.tsv`. Runs entirely client-side — nothing is uploaded anywhere — and mirrors the same parsing and filtering logic as `build_feed.py` below.

## Usage — Python script (alternative / scriptable)

```bash
python build_feed.py catalog.xml pricebook.xml output.tsv
```

### Example

```bash
# Export both files from SFCC Business Manager, then:
python build_feed.py \
  all-skus-google-shopping-feed.xml \
  pricebooks.xml \
  google_shopping_feed.tsv
```

`start_feed_builder.bat` runs the script with a double-click on Windows (prompts for input via the script's own CLI handling).

Then upload `google_shopping_feed.tsv` as a **primary feed** in Google Merchant Center.

## What it does

1. Parses the SFCC catalog XML (products, variants, images, categories)
2. Parses price book XML (list prices + sale prices from consignment/closeout books)
3. Links variants to their master products (inherits name, description, images, brand, category)
4. Filters to **online + available** products with a name, price, and image
5. Outputs a TSV with all required Google Shopping fields

## Feed fields

| Field | Source |
|-------|--------|
| `id` | SFCC product ID (variant-level) |
| `title` | Display name + color/size for variants |
| `description` | Short description (HTML stripped) |
| `link` | `/p/{slug}/{master-id}.html` |
| `image_link` | Hashless static URL (Google crawler resolves real images from og:image) |
| `price` | From `ch-list-prices` price book |
| `sale_price` | From consignment/closeout price books (when lower than list) |
| `brand` | Brand field |
| `gtin` | UPC code |
| `mpn` | Manufacturer SKU |
| `item_group_id` | Master product ID (groups variants) |
| `color` / `size` | Custom attributes |
| `product_type` | Classification category |

## How to get the export files from Business Manager

### Catalog XML
1. **Merchant Tools → Products and Catalogs → Export**
2. Select catalog `master-cheshirehorse`
3. Export as XML

### Price Book XML
1. **Merchant Tools → Products and Catalogs → Price Books**
2. Select all price books → **Export**

## Notes

- **Images**: The feed uses hashless SFCC static URLs as placeholders. Google's crawler picks up the real images from `og:image` tags on your product pages within 24-72 hours.
- **Shipping**: Configure shipping rules at the account level in GMC (Settings → Shipping and returns) rather than in the feed.
- **No dependencies**: The Python script uses only the standard library (xml, csv, argparse). Python 3.6+. The browser tool has no dependencies at all — just open `index.html`.

## Site configuration

Both the Python script and the browser tool define the same constants near the top of the file (`SITE_URL`, `IMAGE_BASE`, `LIST_PRICE_BOOK`, `SALE_PRICE_BOOKS`, `SITE_ID`). Edit both `build_feed.py` and `index.html` if adapting this for a different SFCC site — they aren't shared, so a change in one won't propagate to the other.
