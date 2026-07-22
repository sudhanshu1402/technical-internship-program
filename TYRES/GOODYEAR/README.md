# Goodyear India store scraper

A Scrapy spider that collects tyre-dealer listings from the Goodyear India store locator (`goodyear.co.in/store`) and writes them to a CSV.

## What it does

The store locator paginates through a JavaScript-rendered list of dealers. Because the list is rendered client-side, plain Scrapy requests can't see it, so this project drives a real Chrome browser with Selenium to load the page and click through the pages, then hands the individual store URLs back to Scrapy to fetch and parse.

Flow:

1. Selenium opens the store locator, reads the `#store-list` container, and clicks the "next" pagination link in a loop.
2. For every store card it finds, it builds the store detail URL and yields a `scrapy.Request`.
3. `parse_items` pulls fields out of each detail page with XPath and yields a `GoodyeartyresItem`.

The scraped data is in `goodyeartyres.csv` (1,260 dealers).

## Fields

Defined in `GoodyeartyresItem.py`:

| Field | Source |
| --- | --- |
| `store_name` | dealer page heading |
| `full_address` | dealer page address line |
| `email_id` | dealer page (blank if absent) |
| `phone_number` | dealer page |
| `goodyear_zone` | `1` if the page reads "Goodyear Retailer", else `0` |
| `brand` | hardcoded `"Good Year"` |
| `website` | the dealer detail URL |
| `manufacturer_unique_id`, `state`, `district`, `pincode` | present in the schema but never populated — always blank |

The empty columns are placeholders that were part of the target schema; the source pages don't expose those values, so they come out empty in the CSV.

## Stack

- Python
- [Scrapy](https://scrapy.org/) — request scheduling, item pipeline, CSV export
- [Selenium](https://www.selenium.dev/) — driving Chrome for the paginated, JS-rendered list
- BeautifulSoup + lxml — parsing the store-list HTML pulled from the browser

## Running it

This is one spider extracted from a Scrapy project (`goodyeartyres`). The imports (`from goodyeartyres.GoodyeartyresItem import ...`, `SPIDER_MODULES = ["goodyeartyres.spiders"]`) assume it lives inside that project layout, so to run it you'd drop these files back into a Scrapy project of that name.

Prerequisites:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

You also need Chrome and a matching `chromedriver`. The driver path is hardcoded in `goodyeartyresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change that to your own chromedriver path.

Then, from the project root:

```bash
scrapy crawl goodyeartyresspider -o goodyeartyres.csv
```

## Files

- `goodyeartyresspider.py` — the spider (Selenium pagination + XPath parsing)
- `GoodyeartyresItem.py` — item schema
- `settings.py` — Scrapy settings (`ROBOTSTXT_OBEY = True`, `LOG_LEVEL = "INFO"`)
- `pipelines.py` — pass-through pipeline (no transformation)
- `middlewares.py` — default Scrapy scaffold
- `goodyeartyres.csv` — scraped output
- `README` — the original scrape target notes

## Notes

- Written against the old Selenium 3 API (`find_element_by_xpath`, `find_element_by_css_selector`, and passing the driver path to `webdriver.Chrome()`). On Selenium 4+ these are removed — you'd need `Service` and `By` to make it run.
- The pagination loop has no explicit stop condition; it relies on `next_page.click()` eventually raising when there's no next link.
- XPaths are absolute and tied to the site's DOM at the time of scraping, so they'll break if the site changes.

This was a web-scraping exercise done during a technical internship program, one of several tyre-brand dealer scrapers. It's a one-off data-collection script, not a maintained tool.
