# Goodyear India store scraper

Pulls dealers from the Goodyear India locator at `goodyear.co.in/store`.

Selenium reads `#store-list` and clicks through the pagination, building a detail URL per store card and handing each back to Scrapy as a `Request`. `parse_items` then pulls fields off the detail page with XPath. Output: `goodyeartyres.csv`, 1,260 dealers, the largest haul in this set after Maxxis.

## Fields

Standard schema plus `goodyear_zone` (`1` when the page reads "Goodyear Retailer") and `website` (the detail URL). `brand` is `"Good Year"`. `manufacturer_unique_id`, `state`, `district`, and `pincode` are in the schema but never populated, since the detail pages don't expose them.

## Run

```bash
scrapy crawl goodyeartyresspider -o goodyeartyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotcha

The pagination loop has no stop condition. It relies on `next_page.click()` eventually raising when there's no next link, which is how the crawl ends.
