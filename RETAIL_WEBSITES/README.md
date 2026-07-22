# RETAIL_WEBSITES

Scrapy spiders that pull product data off four B2B supply/HVAC retail sites. Built during an internship; the job each time was the same: crawl a catalog, extract product details, dump them to CSV (or MongoDB).

## What's here

Each folder is a standalone Scrapy project targeting one website. They share the same shape — an `Item` class defining the fields, a spider that walks the site and parses product pages, and the usual Scrapy scaffolding (`settings.py`, `pipelines.py`, `middlewares.py`). A sample CSV of scraped output sits next to each spider.

| Folder | Site | How it finds products | Fields scraped | Sample rows |
|--------|------|----------------------|----------------|-------------|
| `MARKETAIR` | marketair.com | 16 hardcoded category pages → follows product links | title, price, url, desc, images | 82 |
| `SUPPLYHOUSE` | supplyhouse.com | product sitemaps + Selenium | url, title, sku, brand, price, desc, specs, images, docs | 6 |
| `AFSUPPLY` | afsupply.com | 3 sitemaps, XPath + BeautifulSoup for images | title, sku, price, retail_price, stock, mpn, upc, brand, description, specs, url, docs, image_urls | 24 |
| `SUSTAINABLESUPPLY` | sustainablesupply.com | 5 collection pages + Selenium pagination | url, title, sku, brand, mpn, weight, weight_unit, price, desc, specs, images, docs | 14 |

## Two scraping approaches

The four spiders split by how the target site serves its pages:

- **Static HTML (`MARKETAIR`, `AFSUPPLY`)** — plain Scrapy requests with CSS/XPath selectors. `AFSUPPLY` adds a `requests` + BeautifulSoup pass to grab product images that only appear after the main parse, and reads product URLs straight from the site's XML sitemaps via `SitemapSpider`.
- **JavaScript-rendered (`SUPPLYHOUSE`, `SUSTAINABLESUPPLY`)** — these pages load content client-side, so the spider drives a headless Chrome through Selenium, waits for the DOM, then reads it. `SUSTAINABLESUPPLY` also clicks the "next page" button in a loop to walk paginated collections; `SUPPLYHOUSE` pulls its URL list from sitemaps.

Every field is wrapped in its own `try/except` that falls back to an empty string. Ugly, but on a catalog of thousands of products one missing `<span>` shouldn't kill the crawl — the row just comes out partial.

## Stack

- Python + [Scrapy](https://scrapy.org/)
- Selenium + ChromeDriver (SUPPLYHOUSE, SUSTAINABLESUPPLY)
- requests + BeautifulSoup (AFSUPPLY, for images)

No `requirements.txt` ships with these. You'd need:

```bash
pip install scrapy selenium beautifulsoup4 requests
```

## Running one

These are Scrapy projects, so from inside a project folder (with the standard `scrapy.cfg` present) it's the normal run command:

```bash
cd MARKETAIR
scrapy crawl marketairspider -o output.csv
```

Spider names are `marketairspider`, `supplyhousespider`, `afsupplyspider`, `sustainablesupplyspider`.

Caveats before anything runs:

- The Selenium spiders hardcode a Windows ChromeDriver path (`C:/Program Files (x86)/Google/Chrome/Application/chromedriver`). Change it to your local driver.
- `AFSUPPLY`'s item import points at `Hub.models.voomi.AfsupplyItem` — a path from the internship's larger project tree, not this folder. Fix the import to the local `AfsupplyItem` to run it standalone.
- `ROBOTSTXT_OBEY = False` is set. These were internal data-collection tasks; be deliberate about where and how you point them.

## Example output

A row from `AFSUPPLY/afsupplyspider.csv`, one product per line, columns:

```
brand,description,docs,image_urls,mpn,price,retail_price,sku,specs,stock,title,upc,url
```

## Scope

Internship work, archived as-is. The code does its job but shows its age: hardcoded absolute paths, `print`-based logging, brittle positional XPaths, `try/except pass` on every field, and a `main.py` in `MARKETAIR` that's still the PyCharm "Hi, PyCharm" starter file. Kept here as a record of the scrapers actually built, not as a polished library.
