# EXTRA_WEBSITES

A few small [Scrapy](https://scrapy.org/) scrapers written during the internship program. Some were practice, some pulled data that made another task easier. Nothing here is a product; they're throwaway crawlers kept for reference.

Each folder is a separate Scrapy project (its own `settings.py`, `items.py`, `pipelines.py`, `middlewares.py`) with a single spider.

## What's in here

| Folder | Target site | What it scrapes | Notes |
|--------|-------------|-----------------|-------|
| `DISTRICT` | instapdf.in | Names of all districts in India, one flat list | Output saved to `dist.csv`; source list also in `All Districts in India Dict Form.docx` |
| `PINCODE` | indiatvnews.com | Indian postal (PIN) codes, crawled state → area → table | `pincodes.csv` is empty (never produced a clean run); reference list in `All Pincodes in India List Form.docx` |
| `ANGELMATCH` | angelmatch.io | Angel investor profiles (name, company, email, location, links, investment focus, portfolio) | Uses Selenium + BeautifulSoup, not Scrapy's own downloader |

## How they work

### DISTRICT
Plain Scrapy. The spider (`districtspider.py`) hits one page and pulls the second column of a table via a hardcoded XPath tied to the WordPress post id (`#post-23838`). One request, one item, a list of district names. The XPath breaks if the page markup changes.

### PINCODE
A three-level crawl in `pincodespider.py`: `parse` follows state links, `parse2` follows area links, `parse_items` reads the PIN column out of the results table. It dedupes into a module-level `postals` list and prints as it goes. It only prints — there's no pipeline writing to disk, which is why `pincodes.csv` is empty. `main.py` is the untouched PyCharm "Hi, PyCharm" stub, unrelated to the spider.

### ANGELMATCH
This one bypasses Scrapy's request engine. It drives a real Chrome through Selenium (`webdriver.Chrome(...)` with a hardcoded Windows chromedriver path), loops the first 6 result pages, waits with `time.sleep(3)` for the JS-rendered `ais-hits--item` cards to load, then parses each card with BeautifulSoup. Every field is wrapped in its own `try/except` that falls back to an empty value, so a missing element on one card doesn't kill the run. Selector rules are the site's inline styles (e.g. matching on `style="font-size:0.7rem"`), which is brittle by nature.

## Running one

These predate a pinned `requirements.txt`. To run any of them you need Python 3 plus:

```bash
pip install scrapy beautifulsoup4 lxml selenium
```

Then, from a folder configured as a proper Scrapy project (each spider expects to live under a `<project>/spiders/` package — the `SPIDER_MODULES` in each `settings.py` point at e.g. `pincode.spiders`):

```bash
scrapy crawl districtspider -o dist.csv
scrapy crawl pincodespider
scrapy crawl angelmatchspider
```

ANGELMATCH additionally needs Chrome and a matching chromedriver, and the path in `angelmatchspider.py` (`C:/Program Files (x86)/Google/Chrome/Application/chromedriver`) is Windows-specific — change it for your machine.

## Honest scope

Learning-grade code. XPaths are absolute and tied to specific page layouts, so they've likely rotted as the source sites changed. PINCODE never wrote its output. Field extraction leans on inline-style selectors that any redesign would break. Useful as examples of a plain Scrapy crawl, a multi-level link crawl, and a Selenium-driven scrape of a JavaScript-rendered page — not as tools you'd point at these sites today.
