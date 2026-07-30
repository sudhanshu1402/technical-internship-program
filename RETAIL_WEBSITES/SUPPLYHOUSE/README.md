# SupplyHouse product scraper

Pulls product data from [supplyhouse.com](https://www.supplyhouse.com/), a plumbing, heating, and HVAC parts retailer.

Scrapy's `SitemapSpider` reads `sitemap_products_1.xml` and `sitemap_products_2.xml` and hands each product URL to `parse()`, where headless Chrome loads the page. Selenium is needed because price, specs, and images all render client-side, so the raw HTML Scrapy downloads is missing them. Output: `supplyhouse.csv`, a small sample of Viessmann boiler parts.

## Fields

`url`, `title`, `sku`, `brand`, `price`, `description`, `specs` (key/value pairs), `images`, and document URLs.

```
brand:  Viessmann
title:  Gasket (Burner Mesh) CU3A
sku:    7834397
price:  102.99
images: {'image_url_1': 'https://s3.amazonaws.com/.../7834397-1.jpg'}
```

## Run

```bash
scrapy crawl supplyhousespider -o supplyhouse.csv
```

Shared setup and caveats are in [../README.md](../README.md).

## The thing to fix first

**A fresh Chrome driver is created for every product page and never quit.** That leaks browser processes and is the reason this can't be pointed at a real catalog. Combined with a fixed `time.sleep(15)` per page, a full crawl would take days. Fine for the handful of rows it produced, wrong for anything bigger.

`main.py` is leftover IDE template and does nothing.
