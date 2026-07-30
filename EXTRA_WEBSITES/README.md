# EXTRA_WEBSITES

Three small Scrapy scrapers from the internship. Some were practice, some pulled data that made another task easier. Throwaway crawlers kept for reference.

| Folder | Site | Scrapes | State |
|---|---|---|---|
| DISTRICT | instapdf.in | All India district names | Works, but writes one cell instead of one row per district |
| PINCODE | indiatvnews.com | India PIN codes, state to area to table | Prints only, never persists. `pincodes.csv` is empty |
| ANGELMATCH | angelmatch.io | Angel investor profiles | Works. Selenium + BeautifulSoup, bypasses Scrapy's downloader |

Between them they demo three different shapes: a plain single-request Scrapy crawl, a multi-level link crawl, and a Selenium scrape of a JavaScript-rendered page.

## Setup

```bash
pip install scrapy beautifulsoup4 lxml selenium
```

Each spider expects to live under a `<project>/spiders/` package, matching the `SPIDER_MODULES` in its `settings.py`.

```bash
scrapy crawl districtspider -o dist.csv
scrapy crawl pincodespider
scrapy crawl angelmatchspider
```

ANGELMATCH also needs Chrome and a matching chromedriver, and its hardcoded path (`C:/Program Files (x86)/Google/Chrome/Application/chromedriver`) is Windows-specific.

## Shared caveats

Learning-grade code. Absolute XPaths tied to specific page layouts, so expect rot. Field extraction in ANGELMATCH leans on inline-style selectors that any redesign breaks. PINCODE never wrote its output. Useful as examples, not as tools to point at these sites today.
