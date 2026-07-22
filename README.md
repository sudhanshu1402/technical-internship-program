# Technical Internship Program

Archive of the web scrapers I built during a technical internship. Despite the "websites" folder names, nothing here is a website — every project is a [Scrapy](https://scrapy.org/) spider that extracts structured data from a live site and writes it to CSV. Most of the harder targets pair Scrapy with Selenium and BeautifulSoup to drive a real Chrome browser through JavaScript-heavy pages before parsing.

The work splits into three categories: tyre-brand dealer locators, US industrial/retail product catalogs, and a few one-off data-collection jobs.

## What's in here

### TYRES — dealer/store locator scrapers

Each spider hits a tyre manufacturer's "find a dealer" page and pulls the dealer list (store name, address, phone, etc.) into CSV. The JS-driven locators (Apollo, Metzeler, Michelin, Pirelli, and similar) are automated with Selenium: type a city into the search box, wait, scroll/"show more" until the list stops growing, then hand the rendered HTML to BeautifulSoup.

| Brand | Target |
| :--- | :--- |
| APOLLO | apollotyres.com dealer finder (drives search box for a list of cities) |
| BIRLA | birla-tyre.in dealer network |
| BRIDGESTONE | select.bridgestone.co.in |
| CEAT | ceat.com tyre-shop |
| CONTINENTAL | continental-tyres.in car dealers |
| FALKEN | Falken dealer locator |
| FIRESTONE | firestonetyre.co.in stores |
| GOODYEAR | goodyear.co.in store locator |
| MAXXIS | maxxistyres.in dealer locator |
| METZELER | metzeler.com dealer locator |
| MICHELIN | michelin.in — split into `CAR/` (auto) and `BIKE/` (motorbike) locators |
| MRF | mrftyres.com |
| PIRELLI | pirelli.com dealer locator |
| YOKOHAMA | yokohama-india.com store locator |

### RETAIL_WEBSITES — product catalog scrapers

Product-detail scrapers for US supply/e-commerce sites. These walk the site's XML sitemap (`SitemapSpider`) or a fixed list of collection/product URLs, open each product in Selenium, and extract title, SKU, brand, price, description, spec tables, image URLs, and document links.

| Project | Target | Notes |
| :--- | :--- | :--- |
| SUPPLYHOUSE | supplyhouse.com | Crawls product sitemaps, headless Chrome per product |
| AFSUPPLY | afsupply.com | Sitemap-driven |
| MARKETAIR | marketair.com | Fixed list of product pages |
| SUSTAINABLESUPPLY | sustainablesupply.com | HVAC / plumbing / safety / electric-motor collections |

### EXTRA_WEBSITES — one-off data jobs

| Project | Target | What it collects |
| :--- | :--- | :--- |
| PINCODE | indiatvnews.com | India PIN codes, crawled through nested state/area links |
| DISTRICT | instapdf.in | List of all India state districts |
| ANGELMATCH | angelmatch.io | Investor listings, paged with Selenium + BeautifulSoup |

The PINCODE and DISTRICT folders also ship the collected data as `.docx` reference files.

## Layout of a single project

Every folder follows the standard Scrapy project shape (as the inner package, without a `scrapy.cfg` at the folder root):

```
<PROJECT>/
  <name>spider.py      # the Spider — request flow + parsing
  <Name>Item.py        # scrapy.Item defining the output fields
  settings.py          # Scrapy settings (ROBOTSTXT_OBEY = False, LOG_LEVEL = INFO)
  pipelines.py         # pass-through item pipeline
  middlewares.py       # default Scrapy middleware stubs
  <name>.csv           # a saved run's output
```

## Running one

There's no `requirements.txt` or `scrapy.cfg` checked in, so treat these as reference code rather than turnkey projects. To actually run a spider you'd need:

```bash
pip install scrapy selenium beautifulsoup4 lxml
scrapy crawl apollotyresspider -o output.csv
```

Two things will bite you first:

- **Hardcoded Windows chromedriver path.** Every Selenium spider points at `C:/Program Files (x86)/Google/Chrome/Application/chromedriver`. Change it to your local driver (and on modern Selenium, pass it via a `Service` object).
- **Old Selenium API.** The code uses Selenium 3 calls like `find_element_by_xpath(...)`, which were removed in Selenium 4. Either pin Selenium 3 or migrate to `find_element(By.XPATH, ...)`.

## Honest scope note

This is internship-era code, kept for the record. The parsing works and the CSVs are real, but the style shows its age: brittle absolute XPaths, `time.sleep()` instead of explicit waits, a module-level Selenium driver, and per-field try/except with `print` debugging rather than logging. It's a decent snapshot of learning production-style scraping across a lot of different site structures, not a polished library.

---
*Maintained by Sudhanshu Singh*
