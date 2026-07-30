# PINCODE

Crawls Indian postal PIN codes from the indiatvnews.com pincode directory. A three-stage crawl: `parse` follows state links, `parse2` follows area links, `parse_items` reads the PIN column out of the results table, skipping duplicates.

**It never saves anything.** The spider prints to the console and collects into a module-level `postals` list. `pincodes.csv` is present but empty. `PincodeItem` exists and is never used, and the pass-through pipeline isn't enabled in `settings.py`.

The reference data it was meant to produce is in `All Pincodes in India List Form.docx`.

## Run

```bash
scrapy crawl pincodespider
```

Shared setup is in [../README.md](../README.md). Watch the console.

## Wiring up the output

The pieces are all here, just not connected:

1. Have `parse_items` yield a `PincodeItem` per PIN instead of appending to the global list.
2. Run with `-o pincodes.csv`, or enable `ITEM_PIPELINES`.

## Note

XPaths are absolute paths tied to indiatvnews.com's structure at the time. `main.py` is a PyCharm stub, unrelated.
