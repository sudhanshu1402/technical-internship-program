# PINCODE

A Scrapy spider that collects Indian postal PIN codes by crawling the pincode directory on indiatvnews.com.

## What it does

The spider starts at the India PIN codes index page, follows the links down two levels (state/region -> area lists), then reads the PIN code column out of the tables it lands on. Each new PIN it finds gets appended to an in-memory list, with duplicates skipped.

This is a small scraping exercise, not a finished dataset. A few honest caveats:

- The spider prints the collected PIN codes to the console. It does not write them anywhere. `pincodes.csv` is present but empty.
- `pipelines.py` defines a pass-through pipeline that isn't enabled in `settings.py`, and `items.py` defines a `PincodeItem` the spider never uses — it collects into a module-level `postals` list instead of yielding items.
- The XPath selectors are absolute paths tied to indiatvnews.com's page structure at the time this was written. If that site changed its markup, the selectors will need updating.
- `main.py` is the default PyCharm sample stub and is unrelated to the spider.

## Layout

This is a standard Scrapy project split across the usual files:

| File | Role |
|------|------|
| `pincodespider.py` | The spider. Three-stage crawl: `parse` -> `parse2` -> `parse_items`. |
| `items.py` | `PincodeItem` field definitions (`pin`, `postals`). Currently unused by the spider. |
| `pipelines.py` | Pass-through `PincodePipeline`. Not enabled. |
| `middlewares.py` | Default spider/downloader middleware scaffolding from `scrapy startproject`. |
| `settings.py` | Project settings. `ROBOTSTXT_OBEY = False`; most other settings left at defaults. |
| `main.py` | Leftover PyCharm stub, unrelated. |
| `pincodes.csv` | Empty output placeholder. |
| `All Pincodes in India List Form.docx` | Reference document. |

## Requirements

- Python 3
- [Scrapy](https://scrapy.org/)

```bash
pip install scrapy
```

There is no `requirements.txt` in this directory despite what the old README claimed.

## Run

This directory holds the project files, but a Scrapy project normally has a `scrapy.cfg` and a package layout (`pincode/spiders/`). `settings.py` expects the spider under `pincode.spiders`, and `pincodespider.py` imports `from pincode.items import PincodeItem`. To run it, place these files in that package structure, then:

```bash
scrapy crawl pincodespider
```

Watch the console — the discovered PIN codes are printed there via the `postals` list.

## If you wanted to actually save the output

The pieces are already here; they're just not wired up. You'd:

1. Have `parse_items` `yield` a `PincodeItem` per PIN instead of appending to the global list.
2. Enable `ITEM_PIPELINES` in `settings.py` (or just run with `-o pincodes.csv`).

```bash
scrapy crawl pincodespider -o pincodes.csv
```

## Scope

Learning project for practicing Scrapy's multi-level link following and XPath extraction. It crawls and prints; it does not persist, deduplicate across runs, or handle site-structure changes.
