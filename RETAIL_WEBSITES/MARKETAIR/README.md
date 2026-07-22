# MARKETAIR spider

A Scrapy spider that scrapes product listings from [marketair.com](https://www.marketair.com/), an HVAC accessories retailer.

## What it does

Starts from 16 category pages (one per product line: AcustiFlex, DeflectAir, DrainHide, DrainMate, DripShield, DSS Switch, EasyBend, LinePort, Perfect Pitch, Pipe Prop, ReversaLine, RoughInBox, SnapFix, SnowShield, SuperSleeve, ValveShield). For each category it follows the individual product links, opens every product page, and pulls out five fields:

- `title`
- `price` (the `$` is stripped off)
- `url`
- `desc` — the product description block, with `\r \n \t [ ] '` characters cleaned out
- `images` — a dict of `image_url_1`, `image_url_2`, ... built from the main image plus additional images

Each scraped product is emitted as a `MarketairItem`. A sample of the output is checked in as `marketairspider.csv` (82 product rows, columns: `desc, images, price, title, url`).

This was built during a technical internship program as one of several retail-site scraping exercises. It's a real, working scraper for one specific site, not a general framework.

## Files

| File | Purpose |
|------|---------|
| `marketairspider.py` | The spider: start URLs, link following, and per-product parsing |
| `MarketairItem.py` | The Scrapy `Item` defining the five output fields |
| `settings.py` | Scrapy project settings (`ROBOTSTXT_OBEY = False`, most defaults left commented) |
| `pipelines.py` | Pass-through item pipeline (no transformation) |
| `middlewares.py` | Default Scrapy spider/downloader middleware scaffolding |
| `marketairspider.csv` | Sample scraped output |
| `main.py` | Unused PyCharm boilerplate — not part of the scraper |

## Stack

- Python
- [Scrapy](https://scrapy.org/)
- itemadapter (used by the pipeline)

There is no `requirements.txt` in this folder. Install what it needs directly:

```bash
pip install scrapy itemadapter
```

## Running it

These files are the guts of a Scrapy project package (`marketairspider.py` belongs in `marketair/spiders/`, and imports reference `marketair.MarketairItem` and `marketair.spiders`). To run, place them in a Scrapy project laid out like:

```
marketair/
  __init__.py
  MarketairItem.py
  settings.py
  pipelines.py
  middlewares.py
  spiders/
    __init__.py
    marketairspider.py
scrapy.cfg
```

Then, from the project root:

```bash
# print items to the console
scrapy crawl marketairspider

# or dump to CSV, like the sample in this folder
scrapy crawl marketairspider -o marketairspider.csv
```

## How the parsing works

The spider uses two callbacks:

1. `parse` — on each category page, grabs product links from `div.vm-product-media-container a` and requests each one (the site runs VirtueMart, hence the `vm-` selectors).
2. `parse_items` — on each product page, extracts the five fields. Every extraction is wrapped in its own `try/except` so one missing field doesn't kill the whole item; on failure the field is set to empty and the exception is printed.

Field selectors:

- title: `//*[@id="sidecontent"]/div[2]/h1/text()`
- price: `span.PricebasePrice::text`
- desc: `div.product-description`
- images: `div.main-image a::attr(href)` + `div.additional-images a::attr(href)`

## Caveats

- Selectors are pinned to marketair.com's markup as it was when this was written. If the site changed, they'll return empty fields.
- `ROBOTSTXT_OBEY` is off and there's no download delay set, so add throttling before running it at scale against a live site.
- `main.py` is leftover IDE scaffolding and does nothing for the scraper.
