# AFSUPPLY spider

Sitemap crawler for [afsupply.com](https://www.afsupply.com/), a plumbing and hardware retailer on Magento. Static HTML, so plain Scrapy requests with XPath.

Reads three sitemaps (`sitemap_1.xml` through `sitemap_3.xml`), visits every product URL, and scrapes each page. Output: `afsupplyspider.csv`, 24 products.

## Fields

`title`, `sku`, `brand`, `mpn`, `upc`, `price` and `retail_price` (both stripped of `$` and commas), `stock`, `description`, a `specs` dict from the additional-attributes table, `docs` (spec-sheet link), `image_urls`, and `url`.

## Run

```bash
scrapy crawl afsupplyspider -o output.csv
```

Shared setup and caveats are in [../README.md](../README.md). One extra step for this spider: the item import points at `Hub.models.voomi.AfsupplyItem`, a path from the internship's larger project tree. Change it to the local `AfsupplyItem` to run standalone.

## Implementation notes

- **Fallback XPaths.** `sku` and `stock` try several selectors in turn, because product pages don't all use the same markup.
- **`title is None` short-circuits with `exit()`.** Pages that don't parse as products are dropped rather than yielding empty rows. Blunt, but it keeps the CSV clean.
- **Images come from a second fetch.** Rather than reading them off the Scrapy response, `parse()` re-fetches the page with `requests` and filters `<img>` tags by the Magento cache path prefix. It works, and it doubles the request count per page.
