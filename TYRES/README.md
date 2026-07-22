# TYRES

Dealer/store-locator scrapers for 14 tyre brands sold in India. Each brand drives its own store-finder page and writes the dealer records to CSV.

## What this is

A set of web scrapers built during a technical internship. The job: collect dealer network data (store name, address, phone, pincode, brand, etc.) from the "find a dealer" pages of major tyre manufacturers. Most of those pages are JavaScript-driven store locators where you type a city or pincode and results load into the DOM, so a plain HTTP request returns nothing useful.

The approach for almost every brand is the same:

1. A Scrapy spider is the entry point (`scrapy.Spider` subclass).
2. Selenium drives a real Chrome window: open the locator, close popups, type each city/pincode into the search box, press Enter, wait, scroll or click "show more" until all results are loaded.
3. The rendered HTML of the results container is handed to BeautifulSoup.
4. Each dealer element is parsed field by field into a Scrapy `Item` and yielded.
5. Scrapy's feed export writes the items to a CSV.

De-duplication (where the site returns overlapping results across searches) is done in-spider by tracking a dealer code in a `keys` list.

## Brands covered

Each folder is a self-contained Scrapy project for one brand. The CSVs are the actual scraped output that shipped with the code.

| Folder | Source page | Search by | Rows scraped |
|---|---|---|---|
| APOLLO | apollotyres.com dealer finder | city | (no CSV committed) |
| BIRLA | birla-tyre.in dealer network | city | 49 |
| BRIDGESTONE | select.bridgestone.co.in | — | 42 |
| CEAT | ceat.com tyre shop | pincode | 70 |
| CONTINENTAL | continental-tyres.in | — | (no CSV committed) |
| FALKEN | falkentyre.in find-a-store | — | 167 |
| FIRESTONE | firestonetyre.co.in our stores | — | 71 |
| GOODYEAR | goodyear.co.in store | — | 1,260 |
| MAXXIS | maxxistyres.in dealer locator | — | 2,412 |
| METZELER | metzeler.com dealer locator | — | 46 |
| MICHELIN/CAR | michelin.in auto dealer locator | pincode | 312 |
| MICHELIN/BIKE | michelin.in motorbike dealer locator | area | 312 |
| MRF | mrftyres.com | — | (no CSV committed) |
| PIRELLI | pirelli.com dealer locator | city | 21 |
| YOKOHAMA | yokohama-india.com store locator | — | 28 |

Michelin is split into two spiders (car and bike locators are separate pages).

## Fields

The common item schema across brands:

```
manufacturer_unique_id, store_name, full_address,
address_line_one, address_line_two, email_id,
phone_number, state, district, pincode, brand
```

Some brands add a brand-specific flag — e.g. CEAT has `ceat_shoppe` (branded outlet yes/no), Apollo has `apollo_zone`. Fields the source page doesn't expose are left as empty strings.

## Tech

- **Python 3**
- **Scrapy** — spider framework and CSV feed export
- **Selenium** — headed Chrome automation for the JS locators
- **BeautifulSoup4** (lxml / html.parser) — HTML parsing

There is no `requirements.txt`. Install what you need:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

You also need Google Chrome plus a matching **chromedriver**.

## Running a spider

Each folder is a fragment of a Scrapy project (spider + `Item` + `settings.py` + `pipelines.py` + `middlewares.py`) rather than a ready-to-run tree — the spider imports assume a package name like `birlatyres.spiders`. To run one, place the spider under a real Scrapy project's `spiders/` directory and point the imports/`SPIDER_MODULES` at it, then:

```bash
scrapy crawl birlatyresspider -o birlatyres.csv
```

Before it works you must fix the hardcoded chromedriver path. Every spider has this line pinned to a Windows install:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change it to your local chromedriver location.

## Honest scope notes

- **Internship-era code, kept as an archive.** It reflects how the sites and the Selenium API looked at the time.
- **Selenium API is dated.** The spiders use `driver.find_element_by_xpath(...)`, removed in Selenium 4. Running on a current Selenium means switching to `driver.find_element(By.XPATH, ...)`.
- **Selectors are brittle by nature.** Much of the location logic is absolute XPaths and long `next_sibling` chains against the DOM as it was; store-locator pages get redesigned, so expect selectors to have drifted.
- **`ROBOTSTXT_OBEY` varies per brand** (True for some, False for others) — check the brand's `settings.py` before running.
- **Timing is `time.sleep(3)` everywhere**, not explicit waits — simple, but slow and occasionally flaky.
- The committed CSVs are the point of the deliverable: they're the dealer data that was actually collected.
