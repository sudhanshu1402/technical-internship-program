# Falken Tyres Store Scraper

Scrapes the Falken India dealer/store locator and dumps every listing to CSV. One of a set of tyre-brand scrapers built during a technical internship.

## What it does

The Falken store locator at `http://falkentyre.in/find-a-store` doesn't show dealers until you pick a state from a dropdown, and the results load into the page with JavaScript. A plain HTTP request gets you an empty list, so this uses Selenium to drive a real Chrome browser: open the page, select a state, wait for the dealer list to render, then hand the resulting HTML to BeautifulSoup to pull out each address block.

Each dealer becomes a row with these fields:

- `store_name` — dealer name (from the `<h3>`)
- `full_address` — the address paragraph
- `phone_number` — the element right after the address paragraph
- `brand` — hardcoded to `FALKENTYRES`
- `manufacturer_unique_id`, `email_id`, `state`, `district`, `pincode` — declared but left blank; the source page doesn't expose them separately

## Stack

- Python 3
- [Scrapy](https://scrapy.org/) — spider framework and CSV export
- [Selenium](https://www.selenium.dev/) — drives Chrome to trigger the JS-rendered dealer list
- [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/) (with `lxml`) — parses the rendered HTML

## Files

| File | Purpose |
|------|---------|
| `falkentyresspider.py` | The spider — Selenium automation + parsing |
| `FalkentyresItem.py` | The `scrapy.Item` field definitions |
| `settings.py` | Scrapy project settings (`ROBOTSTXT_OBEY = True`, `LOG_LEVEL = INFO`) |
| `pipelines.py` | Default pass-through pipeline (no transforms) |
| `middlewares.py` | Default Scrapy middleware boilerplate |
| `falkentyres.csv` | Sample output — 167 dealer rows |

## Setup

Selenium here talks to a local Chrome via `chromedriver`. You need:

1. Google Chrome installed.
2. A matching `chromedriver` binary. The spider hardcodes a Windows path:
   ```python
   driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
   ```
   Change that to wherever your `chromedriver` lives before running.

Install the libraries:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

This directory holds the spider and its item/settings files. To actually run it, the files need to sit inside a Scrapy project laid out as the imports expect (`from falkentyres.FalkentyresItem import FalkentyresItem`, and `SPIDER_MODULES = ["falkentyres.spiders"]` in `settings.py`).

## Run

From the Scrapy project root:

```bash
scrapy crawl falkentyresspider -o falkentyres.csv
```

Chrome opens, walks through the state dropdown, and rows stream into `falkentyres.csv`.

## Sample output

```csv
brand,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
FALKENTYRES,,,", 1-2 Shreeji Tower Opp. Himalaya Mall Drive-In-Road , Ahmedabad., Gujarat- 380052",,27912501,,,Hemant Tyre Shop
```

## Notes on the implementation

- **The state loop is unrolled, not looped.** The spider handles three states by copy-pasting the same select-and-parse block for dropdown `option[2]`, `option[3]`, and `option[4]`. It only covers those three options, so it doesn't scrape every state on the site.
- **The Selenium driver is created at import time**, at module top level, not inside the spider. That means Chrome launches as soon as the module is imported.
- **Uses the old Selenium 3 API** (`find_element_by_xpath`), which was removed in Selenium 4. Pin an older Selenium or port these calls to `find_element(By.XPATH, ...)`.
- **Timing is hardcoded `time.sleep(3)`** between steps rather than explicit waits — brittle if the page loads slower or the site markup changes.
- **Blank fields are intentional.** Several fields are set to `""` on purpose because the locator page doesn't split them out; the address string carries city/state/pincode inline.

## Scope

An internship-era scraper for a specific site as it existed at build time. It's tied to that page's structure, the old Selenium API, and a hardcoded chromedriver path, so treat it as a working reference rather than something to run as-is today. The committed `falkentyres.csv` shows what a full run produced.
