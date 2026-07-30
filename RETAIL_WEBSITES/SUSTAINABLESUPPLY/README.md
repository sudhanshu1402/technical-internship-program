# SustainableSupply product scraper

Pulls products from [sustainablesupply.com](https://www.sustainablesupply.com/) across five collections. The listings load through AngularJS/SearchSpring, so plain Scrapy can't see the product links.

Selenium opens each collection, waits for the listing, reads the product anchors, and builds absolute URLs. Scrapy then fetches and parses each product page, while Selenium clicks "next" to walk the pagination.

Collections: restroom-supplies, electric-motors, hvac-r-supplies, plumbing-supplies, safety-supplies. Output: `sustainablesupplyspider.csv` (~29 rows) and `sustainablesupply.csv`.

## Fields

`url`, `title`, `sku`, `brand`, `mpn`, `price` (`$` removed), `weight` and `weight_unit` (numeric and non-numeric halves, regex-split), `desc`, `specs`, `images` (from `og:image:secure_url` meta tags, numbered), and `docs` (numbered).

`desc` falls back to a `<strong>` variant when the plain paragraph comes back empty.

## Run

```bash
scrapy crawl sustainablesupplyspider -o output.csv
```

Shared setup and caveats are in [../README.md](../README.md). Not `python main.py`, which is an unused PyCharm stub.

## Notes

- **Selenium is a module-level global** with no `driver.quit()`, so importing the file opens Chrome and leaves it running.
- Every field falls back to empty on exception, which means a layout change degrades quietly rather than crashing. Good for finishing a run, bad for noticing the data went hollow.
