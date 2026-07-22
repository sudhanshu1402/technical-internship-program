# SupplyHouse Product Scraper

A Scrapy + Selenium spider that pulls product data from [supplyhouse.com](https://www.supplyhouse.com/) — an online plumbing, heating, and HVAC parts retailer.

Built during a technical internship program as part of a set of retail-site scrapers. It's a working practice project, not a production crawler.

## What it does

The spider walks SupplyHouse's product sitemaps, visits every product URL it finds, and scrapes the details off each page. For each product it collects:

- URL
- Title
- SKU
- Brand
- Price
- Description
- Spec table (key/value pairs)
- Image URLs
- Manual/document URLs

Output is written to a CSV (see `supplyhouse.csv` for a sample run of a few Viessmann boiler parts).

## How it works

Two pieces do the work:

1. **Scrapy `SitemapSpider`** — reads the two product sitemaps (`sitemap_products_1.xml`, `sitemap_products_2.xml`) and hands each product URL to `parse()`.
2. **Selenium (headless Chrome)** — inside `parse()`, a Chrome driver actually loads each page, waits for it to render, and extracts fields via XPath.

Selenium is used instead of plain Scrapy responses because the product pages render content with JavaScript, so the raw HTML Scrapy downloads doesn't contain the price, specs, or images.

Every field is wrapped in its own try/except and defaults to an empty value on failure, so a missing element on one page doesn't kill the crawl.

## Tech / stack

- Python
- [Scrapy](https://scrapy.org/) — sitemap crawling and the item/pipeline framework
- [Selenium](https://www.selenium.dev/) — headless Chrome for JS-rendered pages
- `itemadapter` (Scrapy dependency)

There's no `requirements.txt` in this folder. Install what you need directly:

```bash
pip install scrapy selenium itemadapter
```

You also need Google Chrome and a matching **chromedriver** binary.

## Files

| File | Purpose |
|------|---------|
| `supplyhousespider.py` | The spider — sitemap crawl + Selenium scraping logic |
| `SupplyhouseItem.py` | Scrapy `Item` defining the 9 scraped fields |
| `settings.py` | Scrapy project settings (`ROBOTSTXT_OBEY = True`, `LOG_LEVEL = INFO`) |
| `pipelines.py` | Pass-through item pipeline (no transforms) |
| `middlewares.py` | Default Scrapy spider/downloader middleware scaffolding (unused) |
| `supplyhouse.csv` | Sample output from a run |
| `main.py` | Unrelated PyCharm "Hi, PyCharm" stub — not part of the scraper |

## Build & run

This is one spider extracted from a Scrapy project. To run it, place these files back into a standard Scrapy layout (the imports expect a `supplyhouse` package with a `spiders` module, per `settings.py`), then:

```bash
scrapy crawl supplyhousespider -o supplyhouse.csv
```

Before running, fix the hardcoded chromedriver path in `supplyhousespider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Point that at your own chromedriver.

## Sample output

One row from `supplyhouse.csv`:

```
brand:  Viessmann
title:  Gasket (Burner Mesh) CU3A
sku:    7834397
price:  102.99
images: {'image_url_1': 'https://s3.amazonaws.com/.../7834397-1.jpg'}
url:    https://www.supplyhouse.com/Viessmann-7834397-Gasket-Burner-Mesh-CU3A
```

## Honest notes / caveats

- The Selenium XPaths are brittle absolute paths (e.g. `/html/body/main/div[2]/...`) tied to the site's DOM at the time. If the page layout changed, they'd break.
- A fresh Chrome driver is created for **every** product page and never quit — that leaks browser processes and is slow. Fine for a learning exercise, not for a large crawl.
- There's a fixed `time.sleep(15)` per page, so crawls are slow by design.
- `main.py` is a leftover IDE template and does nothing for the scraper.
- The old plain-text `README` just lists the site link and sitemap URLs.

Practice/portfolio project — treat it as a reference for combining Scrapy sitemap crawling with Selenium page rendering, not a maintained tool.
