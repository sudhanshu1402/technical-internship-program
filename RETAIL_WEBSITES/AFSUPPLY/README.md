# AFSUPPLY spider

A Scrapy sitemap crawler that pulls product data from [afsupply.com](https://www.afsupply.com/) (a plumbing/hardware retailer running Magento).

## What it does

The spider reads the site's XML sitemaps, visits every URL they list, and scrapes each product page into a structured item. For each product it extracts:

- `title`, `sku`, `brand`, `mpn`, `upc`
- `price` and `retail_price` (old/list price), cleaned of `$` and commas
- `stock` (availability text)
- `description`, plus a `specs` dict built from the "additional attributes" table
- `docs` (spec-sheet link, if present) and `image_urls` (product image gallery)
- `url` (the source page)

Output is one row per product. A sample run is checked in as `afsupplyspider.csv` (24 products).

This was built during a technical internship program as one of several retail-site scrapers, so it targets afsupply's exact Magento page structure — the XPaths are hard-coded to that layout.

## Files

| File | Purpose |
|------|---------|
| `afsupplyspider.py` | The spider. `SitemapSpider` subclass, all parsing logic lives in `parse()`. |
| `AfsupplyItem.py` | `scrapy.Item` defining the 13 scraped fields. |
| `pipelines.py` | Pass-through pipeline (no transformation). |
| `middlewares.py` | Default Scrapy spider/downloader middleware stubs. |
| `settings.py` | Project settings — `ROBOTSTXT_OBEY = False`, `LOG_LEVEL = "INFO"`. |
| `afsupplyspider.csv` | Example scrape output. |

## Tech

- Python
- [Scrapy](https://scrapy.org/) — `SitemapSpider`, XPath selectors
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) + [requests](https://requests.readthedocs.io/) — used only to grab the product image URLs from a second fetch of the page

## Run it

This module was written to live inside a larger Scrapy project (note the import `from Hub.models.voomi.AfsupplyItem import AfsupplyItem`), so it isn't standalone as-is. To run it on its own:

```bash
pip install scrapy beautifulsoup4 requests

# fix the item import to be local, e.g.:
#   from AfsupplyItem import AfsupplyItem
scrapy runspider afsupplyspider.py -o output.csv
```

`scrapy runspider` needs a proper project layout for `settings.py` to apply; within a full project you'd instead run `scrapy crawl afsupplyspider -o output.csv`.

## Sitemaps crawled

```
https://www.afsupply.com/sitemap_1.xml
https://www.afsupply.com/sitemap_2.xml
https://www.afsupply.com/sitemap_3.xml
```

## Notes on the implementation

- **Fallback XPaths.** `sku` and `stock` try several selectors in turn, because product pages don't all use the same markup. If the primary path returns nothing, it falls back to the next.
- **`title is None` short-circuit.** If no title is found the spider calls `exit()` — pages that don't parse as products are dropped rather than yielding empty rows.
- **Images via a separate request.** Instead of reading images from the Scrapy response, `parse()` re-fetches the page with `requests` and filters `<img>` tags by the Magento cache path prefix. Works, but doubles the request count per page.
- **Every extraction is wrapped in try/except** with a printed message, so one broken field doesn't kill the whole scrape.

## Scope

A single-site scraper coupled to afsupply's current page layout. If the site's HTML changes, the XPaths need updating. Not a general-purpose tool — it's a targeted data-collection script.
