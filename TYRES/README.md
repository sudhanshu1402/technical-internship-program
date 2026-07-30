# TYRES

Dealer-locator scrapers for 14 tyre brands sold in India. Each folder is one brand's Scrapy project, and the committed CSVs are the dealer data actually collected.

Almost every one of these locators is a JavaScript page where you type a city or pincode and results load into the DOM, so a plain HTTP request returns nothing. The shape is the same everywhere: a Scrapy spider is the entry point, Selenium drives real Chrome to search and expand results, the rendered HTML goes to BeautifulSoup, each dealer becomes a Scrapy `Item`, and Scrapy's feed export writes the CSV. De-duplication is in-spider, tracking a dealer code in a `keys` list.

## Brands

| Folder | Source | Search by | Rows |
|---|---|---|---|
| APOLLO | apollotyres.com dealer finder | city | no CSV |
| BIRLA | birla-tyre.in dealer network | city | 49 |
| BRIDGESTONE | select.bridgestone.co.in | district | 42 |
| CEAT | ceat.com tyre shop | pincode | 70 |
| CONTINENTAL | continental-tyres.in | city | unfinished |
| FALKEN | falkentyre.in find-a-store | state | 167 |
| FIRESTONE | firestonetyre.co.in our stores | state + city | 71 |
| GOODYEAR | goodyear.co.in store | pagination | 1,260 |
| MAXXIS | maxxistyres.in dealer locator | none, one page | 2,412 |
| METZELER | metzeler.com dealer locator | per-dealer detail card | 46 |
| MICHELIN/CAR | michelin.in auto locator | pincode | 312 |
| MICHELIN/BIKE | michelin.in motorbike locator | area | 312 |
| MRF | mrftyres.com | state | unfinished |
| PIRELLI | pirelli.com dealer locator | city | 21 |
| YOKOHAMA | yokohama-india.com store locator | state + city + radius | 28 |

## Common item schema

```
manufacturer_unique_id, store_name, full_address,
address_line_one, address_line_two, email_id,
phone_number, state, district, pincode, brand
```

Some brands add a flag of their own (`ceat_shoppe`, `apollo_zone`, `goodyear_zone`, `michelin_certified_centre`, `yokohama_zone`). Fields the source page doesn't expose are written as empty strings.

## Setup

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

Plus Chrome and a matching chromedriver.

Each folder is a fragment of a Scrapy project (spider, `Item`, `settings.py`, `pipelines.py`, `middlewares.py`), not a ready-to-run tree. The imports assume a package name like `birlatyres.spiders`, so to run one, put the spider under a real Scrapy project's `spiders/` directory and point `SPIDER_MODULES` at it:

```bash
scrapy crawl birlatyresspider -o birlatyres.csv
```

## Caveats that apply to every spider here

The individual brand READMEs only list what's unique to them. These are shared:

- **Hardcoded Windows chromedriver path.** Every spider pins `webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")`. Change it before anything runs.
- **Selenium 3 API.** `find_element_by_xpath(...)` and friends were removed in Selenium 4. Pin Selenium 3 or port to `find_element(By.XPATH, ...)`.
- **`time.sleep(3)` instead of explicit waits.** Simple, slow, occasionally flaky.
- **Brittle selectors.** Absolute XPaths and long `next_sibling` chains against the DOM as it was. Store locators get redesigned, so expect drift.
- **`ROBOTSTXT_OBEY` varies per brand.** Check the brand's `settings.py` before running.
- **Hardcoded search lists.** Cities and pincodes are literal lists in each spider, usually Mumbai, Akola, Dhule, Delhi, Surat, Bangalore.

Internship-era code, kept as an archive. The CSVs are the deliverable.
