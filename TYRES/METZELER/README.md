# Metzeler Dealer Scraper

A Scrapy + Selenium spider that pulls Metzeler motorcycle-tyre dealer listings from the India dealer locator and writes them to CSV.

## What it does

The [Metzeler India dealer locator](https://www.metzeler.com/en-in/dealer-locator) renders its dealer list with JavaScript and hides each dealer's full details behind a "goto" link that opens a detail card. Plain HTTP requests don't see that content, so this spider drives a real Chrome browser through Selenium: it loads the page, dismisses the cookie/consent popup, then walks every dealer entry — clicking into the detail card, reading the fields, and clicking back out.

Each dealer is emitted as one row with these fields (see `MetzelerItem.py`):

- `manufacturer_unique_id` — Metzeler's internal dealer ID (e.g. `IN0000000041`)
- `store_name`
- `full_address`
- `phone_number`
- `district`
- `email_id`, `state`, `pincode` — collected as empty; the locator's detail card doesn't expose them
- `brand` — hardcoded to `METZELER`

The scraped output lives in `metzelertyres.csv` and `metzelertyresspider.csv` (46 dealers).

## Stack

- Python
- [Scrapy](https://scrapy.org/) — project scaffolding, item model, CSV export
- [Selenium](https://www.selenium.dev/) with ChromeDriver — the actual page interaction, since the site needs a JS-capable browser

Note: the spider subclasses `scrapy.Spider` but does the real work in Selenium inside `parse()`. Scrapy's own request/response cycle is largely along for the ride here.

## Setup

```bash
pip install scrapy selenium
```

You also need Chrome and a matching ChromeDriver. The spider currently hardcodes a Windows path:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change that line to your own chromedriver location before running.

## Run

This is a spider from a Scrapy project (`SPIDER_MODULES = ["metzeler.spiders"]`). Placed inside that project layout, run:

```bash
scrapy crawl metzelertyresspider -o dealers.csv
```

A visible Chrome window will open and step through the locator. It runs headed on purpose — the flow depends on clicking real elements.

## How the scrape works

The logic in `metzelertyresspider.py`:

1. Load the locator, maximize the window, click `#cp-decline` to close the consent popup.
2. Grab every `#goto-details` element (one per dealer in the list).
3. For each one: move to it, click it, wait for the detail card, then read the input fields (`#scheda-rivenditore/input[1..4]`, `#dealer_phone`) by their `value` attribute.
4. Every field read is wrapped in try/except so a missing element doesn't kill the crawl — it just yields an empty string for that field.
5. Click the back link (`#scheda-rivenditore-outer/div[1]/a`) and move to the next dealer.

`time.sleep()` calls (3–5s) between steps give the page time to render; there's no explicit wait-for-element strategy beyond one `implicitly_wait(5)`.

## Scope and caveats

This is an internship-era scraping exercise, and it shows:

- The item object is created once outside the loop and mutated each iteration, so `yield items` yields the same dict repeatedly. Scrapy serializes on yield so the CSV came out correct, but it's a bug waiting to bite — build a fresh `MetzelerItem()` per dealer.
- It uses Selenium's old `find_element_by_xpath` API, removed in Selenium 4. This runs on Selenium 3.x; on Selenium 4+ you'd switch to `find_element(By.XPATH, ...)`.
- XPaths and element IDs are pinned to the site's layout at scrape time. If Metzeler redesigns the locator, the selectors break.
- `email_id`, `state`, and `pincode` are placeholders — the detail card doesn't surface them, so they're written blank.
- The pipeline (`pipelines.py`) is the default pass-through and isn't enabled in `settings.py`.

Files: `metzelertyresspider.py` (spider), `MetzelerItem.py` (item schema), `settings.py`, `pipelines.py`, `middlewares.py` (Scrapy defaults), plus the two output CSVs.
