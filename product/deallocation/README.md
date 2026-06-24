# Deallocation File Generator

Builds the fixed-width inventory deallocation `.txt` file used to release stock allocations in Eagle, with a step to cross-check against ShipStation open orders so nothing still in flight gets deallocated by mistake.

Part of the [ch-tools](https://github.com/caitlinsc/ch-tools) monorepo.

## How it works

1. **File settings** — set the file date and store number for the export.

2. **Import from spreadsheet** — drop in an order export (`.xls`, `.xlsx`, or `.csv`). The tool auto-detects `Order ID`, `SKU`, `Quantity`, and `Order Date` columns (plus optional `Store` and `Description`), groups rows into orders by Order ID, and skips `SHIP` line items and empty/zero-quantity rows automatically. You can review the detected orders and choose to append them to or replace your current list.

3. **Check against open orders** — drop in a ShipStation open-orders export (same file types). The tool matches order numbers in your list against the open-orders file by exact match (no fuzzy matching) and flags any that are still open. You can then remove those flagged orders from your list with one click so they don't get deallocated while still in transit.

4. **Orders** — a manual editor for the order list: add/remove orders, edit order number, date, and SKU/quantity lines by hand. This is also where you land after import or the open-orders check, so you can fix anything before generating the file. State is saved to the browser automatically, so a refresh won't lose your work.

5. **Generate** — preview the exact output line-by-line (with character counts), then download the `.txt` file or copy it to the clipboard. The download button is disabled — with a warning — until at least one order has a number, a date, and at least one SKU with a quantity greater than 0.

## File format

Output is a fixed-width text file with two line types:

- **Header line (`H`)** — one per order. Encodes the date (`MMDDYYYY`), store number, total quantity across the order's SKUs, and the order number. Padded to exactly 517 characters.
- **Detail line (`D`)** — one per SKU in the order. Encodes the SKU (right-padded to 64 characters) and quantity. Padded to exactly 89 characters.

The tool logs a console warning if a generated line doesn't hit its expected length — worth checking if Eagle ever rejects a file.

Filename: `inventoryDeallocation-MM-DD-YY.txt`, dated from the file settings date.

## Notes

- Everything runs client-side in the browser — nothing is uploaded anywhere.
- This tool is provided for internal convenience and is not affiliated with, endorsed by, or supported by Epicor. The fixed-width format was reverse-engineered from observed file requirements and may need adjustment if Eagle's import spec changes. Review generated files before relying on them.

## Built with

[SheetJS (xlsx)](https://github.com/SheetJS/sheetjs) for reading order and open-orders spreadsheets.
