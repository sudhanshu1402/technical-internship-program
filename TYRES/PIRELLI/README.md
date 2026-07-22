# Pirelli Tyre Dealer Scraper

A Scrapy + Selenium spider that pulls Pirelli's authorized tyre dealers from the India dealer-locator page and writes them to CSV. Built during a technical internship, so it's a real working scraper aimed at one site, not a general-purpose tool.

## What it does

Pirelli's [dealer locator](https://www.pirelli.com/tyres/en-in/car/find-your-dealer/dealer-locator) is a JavaScript page: you type a city, pick an autocomplete suggestion, hit search, and dealers render into a results panel. There's no plain HTML page to request, so Scrapy alone can't see the data.

The spider drives a real Chrome browser with Selenium to do the typing and clicking, then hands the rendered results HTML to BeautifulSoup to extract each dealer. For every city in a hard-coded list it:

1. Loads the locator page and dismisses the cookie/consent popup.
2. Types the city name into the search field, arrow-downs to the first autocomplete option, and presses Enter.
3. Clicks the search button and reads the `#results-panel-box` HTML.
4. Parses each dealer `<li>`, dedupes by `data-dealerid`, and yields a `PirellityresItem`.

Cities scraped: Mumbai, Akola, Dhule, Delhi, Surat, Bangalore.

## Fields collected

Each dealer row (see `PirellityresItem.py`) has:

| Field | Source |
|---|---|
| `manufacturer_unique_id` | `data-dealerid` attribute |
| `store_name` | dealer headline text |
| `full_address` | `.dlAddress` text |
| `district` | the city being searched |
| `brand` | constant `"PIRELLI"` |
| `email_id`, `phone_number`, `state`, `pincode` | present in the schema but left blank — the locator list doesn't expose them |

## Tech

- Python + [Scrapy](https://scrapy.org/) — project skeleton, item model, CSV export
- [Selenium](https://www.selenium.dev/) with ChromeDriver — drives the JS page
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) (lxml parser) — parses the rendered results

`settings.py` keeps `ROBOTSTXT_OBEY = True` and `LOG_LEVEL = "INFO"`. The pipeline and middlewares are the default Scrapy stubs — nothing custom.

## Setup & run

There's no `requirements.txt` in this folder. Install the deps directly:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

Then get a ChromeDriver matching your Chrome version and point the spider at it. The path is hard-coded near the top of `pirellityresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Edit that to your local chromedriver path before running.

This folder holds only the spider and support files, not a full Scrapy project tree. Drop it into a standard Scrapy layout (`scrapy startproject pirellityres`, put the spider under `pirellityres/spiders/`) and run:

```bash
scrapy crawl pirellityresspider -o pirellityres.csv
```

A Chrome window will open and visibly click through each city while the console logs every step.

## Sample output

`pirellityres.csv` is the committed result — 21 dealers. One row:

```csv
brand,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
PIRELLI,Mumbai,,"Shop 4 & 7, Sahajanand Society, TPS 6, near, Milan Subway Rd, Navpada, MSEB Colony, Santacruz West, 400054 MUMBAI",IN0000000076,,,,BOMBAY TYRE
```

Bangalore returned the most dealers (14); the other cities returned one to a handful each.

## Notes and limits

- **Selenium runs non-headless.** It launches a visible Chrome and uses `time.sleep(2-3)` between every action instead of explicit waits. Slower and more brittle than `WebDriverWait`, but easy to watch and debug.
- **The driver is created at import time**, outside the spider class, so Chrome opens the moment the module loads.
- **Uses the pre-Selenium-4 API** (`find_element_by_xpath`), which was removed in Selenium 4.3+. Run it against Selenium 3.x, or migrate those calls to `find_element(By.XPATH, ...)`.
- **XPaths and the popup ID (`#cp-accept`) are pinned to Pirelli's markup at the time this was written.** If the site changed, the selectors need updating.
- Dedup is per-run via an in-memory `keys` list, so the same dealer showing up under two nearby cities is only recorded once.

Scope: a single-site scraping exercise from an internship. It does the job for that page and that list of cities and isn't meant to generalize.
