# Apollo Tyres dealer scraper

A Scrapy spider that pulls Apollo Tyres dealer listings out of the company's "Find a Dealer" store locator. Built during a technical internship.

## What it does

The Apollo Tyres dealer finder (`apollotyres.com/en-in/car-suv-van/dealer-finder/find-a-dealer/`) is an Angular page: you type a city, it queries an API and renders dealer cards on the fly. There's no plain HTML list to scrape, so this spider drives a real Chrome browser with Selenium to load the page, type in a city, expand all results, then hands the rendered HTML to BeautifulSoup to pull the fields.

For each city it collects, per dealer:

- `manufacturer_unique_id` — the dealer's `data-id`
- `store_name`
- `full_address`
- `email_id`
- `phone_number`
- `state`, `district`, `pincode`
- `apollo_zone` — `"1"` if the card is flagged an Apollo Zone, else `"0"`
- `brand` — hardcoded `"APOLLO"`

The cities it searches are hardcoded in the spider: Mumbai, Akola, Dhule, Delhi, Surat, Bangalore.

## How it works

1. Selenium opens Chrome and loads the dealer finder page.
2. For each city, it clears the search box (`input[itemprop='codeCountry']`), types the name, presses Down then Enter to pick the first autocomplete suggestion.
3. It scrolls the results panel (`#mCSB_1`) to the bottom and clicks "Show more" (`a[ng-show='isShowMore']`) in a loop until that link goes `ng-hide`, meaning all dealers are loaded.
4. It grabs the outer HTML of `#dealer-result` and parses it with BeautifulSoup (`lxml`).
5. Each dealer card (`div[itemtype='https://schema.org/location']`) is read into a Scrapy item. A `keys` list de-dupes by `data-id`, so a dealer that shows up under more than one city search is only yielded once.

`time.sleep(3)` sits between every step to wait for the Angular UI to catch up — this is a slow, deliberate scrape, not a fast one.

## Stack

- **Scrapy** — spider framework and item pipeline
- **Selenium** — drives Chrome to render the JS page
- **BeautifulSoup + lxml** — HTML parsing of the rendered result panel
- ChromeDriver

The other files are standard Scrapy scaffolding: `settings.py`, `pipelines.py` (pass-through), `middlewares.py` (unmodified boilerplate), and `ApollotyresItem.py` (the item schema).

## Run it

This module is the `spiders` package of a Scrapy project named `apollotyres` (see `SPIDER_MODULES = ["apollotyres.spiders"]` in `settings.py`). It runs from the project root, not from this folder:

```bash
pip install scrapy selenium beautifulsoup4 lxml
scrapy crawl apollotyresspider -o dealers.csv
```

Before running you must fix the ChromeDriver path in `apollotyresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

That's a hardcoded Windows path. Point it at your own ChromeDriver, and make sure the ChromeDriver version matches your installed Chrome.

## Caveats

- The city list, the ChromeDriver path, and every CSS/XPath selector are hardcoded. Store locators change their markup often, so selectors like `input[itemprop='codeCountry']` or `#mCSB_1` may be stale by now.
- `state`, `pincode`, and `email_id` are read from repurposed schema.org attributes (`endDate` for email, `addressRegion` for address) — brittle by nature; `state` and `pincode` are in fact left blank in the code.
- No throttling beyond the fixed 3-second sleeps. `ROBOTSTXT_OBEY` is off.

This is an internship-era scraping exercise, kept for reference. Selectors and paths would need updating to run against the live site today.
