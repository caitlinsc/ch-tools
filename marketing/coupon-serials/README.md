# Coupon Serials

**Status: idea / not yet built.** This folder is a placeholder — there is no tool here yet, just the concept.

Part of the [ch-tools](https://github.com/caitlinsc/ch-tools) monorepo.

## The idea

Confirm that coupon serial numbers are present and valid in Salesforce — i.e. check a batch of coupon serials (from a vendor file, a print run, etc.) against what's actually loaded in SFCC, to catch missing or mismatched serials before a promotion goes live.

## What's needed before this can be built

- A concrete source format for the serial list to check (vendor export? manual paste? specific file type?)
- Confirmation of what "valid in Salesforce" means in this context — does the serial need to exist on a specific coupon code, be unredeemed, be within a date range, or something else?
- Sample data from a real run, so the matching logic can be built against the actual export shape rather than guessed at

If you pick this back up, it'll probably follow the same shape as the other product-matching tools in this repo (drop a file in, get a CSV of mismatches out, all client-side).
