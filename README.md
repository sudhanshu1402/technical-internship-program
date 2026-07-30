# technical-internship-program

Archive of the web scrapers I built during a technical internship. The folder names say "websites", but nothing here is a website: every project is a [Scrapy](https://scrapy.org/) spider that pulls structured data off a live site into CSV. The harder targets pair Scrapy with Selenium and BeautifulSoup to drive real Chrome through JavaScript-heavy pages before parsing.

Three groups: tyre dealer locators, US product catalogs, and one-off data jobs.

## TYRES, dealer locators

Each spider hits a manufacturer's "find a dealer" page and pulls store name, address, and phone into CSV. The JS-driven ones (Apollo, Metzeler, Michelin, Pirelli) are automated with Selenium: type a city, wait, scroll or click "show more" until the list stops growing, hand the rendered HTML to BeautifulSoup.

Brands covered: Apollo, Birla, Bridgestone, CEAT, Continental, Falken, Firestone, Goodyear, Maxxis, Metzeler, Michelin (split into `CAR/` and `BIKE/`), MRF, Pirelli, Yokohama.

## RETAIL_WEBSITES, product catalogs

Product-detail scrapers for US supply and e-commerce sites. They walk the site's XML sitemap via `SitemapSpider` or a fixed URL list, open each product in Selenium, and extract title, SKU, brand, price, description, spec tables, images, and document links.

| Project | Target | Notes |
|---|---|---|
| SUPPLYHOUSE | supplyhouse.com | Sitemap crawl, headless Chrome per product |
| AFSUPPLY | afsupply.com | Sitemap-driven |
| MARKETAIR | marketair.com | Fixed list of product pages |
| SUSTAINABLESUPPLY | sustainablesupply.com | HVAC, plumbing, safety, electric-motor collections |

## EXTRA_WEBSITES, one-offs

| Project | Target | Collects |
|---|---|---|
| PINCODE | indiatvnews.com | India PIN codes, through nested state and area links |
| DISTRICT | instapdf.in | All India state districts |
| ANGELMATCH | angelmatch.io | Investor listings, paged with Selenium |

PINCODE and DISTRICT also ship their collected data as `.docx`.

## Project shape

Standard Scrapy layout, as the inner package, with no `scrapy.cfg` at the folder root:

```
<PROJECT>/
  <name>spider.py      the Spider, request flow + parsing
  <Name>Item.py        scrapy.Item output fields
  settings.py          ROBOTSTXT_OBEY = False, LOG_LEVEL = INFO
  pipelines.py         pass-through
  <name>.csv           a saved run's output
```

## Running one

No `requirements.txt` or `scrapy.cfg` is checked in, so treat these as reference code, not turnkey projects.

```bash
pip install scrapy selenium beautifulsoup4 lxml
scrapy crawl apollotyresspider -o output.csv
```

Two things bite immediately: every Selenium spider hardcodes a Windows chromedriver path (`C:/Program Files (x86)/...`), and the code uses Selenium 3 calls like `find_element_by_xpath(...)` that were removed in Selenium 4. Pin Selenium 3 or migrate to `find_element(By.XPATH, ...)`.

## Scope

Internship-era code, kept for the record. The parsing works and the CSVs are real, but the style shows its age: brittle absolute XPaths, `time.sleep()` instead of explicit waits, a module-level driver, and per-field try/except with `print` debugging. A snapshot of learning production-style scraping across a lot of different site structures, not a polished library.
