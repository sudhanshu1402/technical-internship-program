# Maxxis Tyres dealer scraper

A Scrapy + Selenium spider that pulls the full dealer list from the Maxxis India store locator and writes it to CSV. Built during a technical internship.

## What it does

The Maxxis dealer locator (`https://www.maxxistyres.in/dealer/locator`) renders its dealer list with JavaScript, so a plain HTTP request returns an empty page. This spider drives a real Chrome browser through Selenium to load the page, grabs the rendered `#dealers-list` markup, then parses each dealer out with BeautifulSoup.

For every dealer it captures:

- `store_name`, `dealer_name` (contact person)
- `full_address`, `district`, `state`, `pincode`
- `phone_number`, `email_id`
- `manufacturer_unique_id` (the list item's `value` attribute)
- `brand` (hard-coded to `MAXXIS`)

The included `maxxistyres.csv` is a real run: 2,412 dealer records across India.

## How it's built

This is a standard Scrapy project layout, but the actual page fetching is done by Selenium rather than Scrapy's downloader:

- `maxxistyresspider.py` — the spider. Opens Chrome, waits for the locator page to load, reads `#dealers-list`, and yields one item per `li.item.mx-4`. Field extraction leans heavily on BeautifulSoup sibling walking (`div.p.next_sibling.next_sibling...`) because the source markup has no clean per-field classes. Every field is wrapped in try/except so a missing value falls back to an empty string instead of killing the crawl.
- `MaxxistyresItem.py` — the Scrapy `Item` with the ten fields above.
- `pipelines.py` — pass-through pipeline (no transformation).
- `settings.py` — project settings, `ROBOTSTXT_OBEY = True`, `LOG_LEVEL = INFO`. Middlewares and pipelines are left commented out (Scrapy defaults).
- `middlewares.py` — the default Scrapy middleware scaffold, unused.

## Requirements

- Python 3
- `scrapy`
- `selenium`
- `beautifulsoup4` and `lxml`
- Google Chrome + a matching `chromedriver`

The driver path is hard-coded to a Windows install:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change that line to point at your own `chromedriver` before running.

## Run it

From the Scrapy project root:

```bash
scrapy crawl maxxistyresspider -o maxxistyres.csv
```

Chrome will open, load the locator, and the spider prints its progress to the console as it walks each dealer.

## Output

CSV with these columns:

```
brand,dealer_name,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
```

Note: `email_id` is always blank — the locator page does not expose dealer emails, so the field is kept for schema consistency only.

## Caveats

- The spider fetches the page once with a fixed `time.sleep(5)`; if the site is slow the list may not be fully rendered. There is no scroll/pagination handling beyond whatever the locator loads on open.
- Field parsing depends on the exact sibling order of the current page markup. If Maxxis changes the layout, the sibling-walk selectors will break and fields will silently come back empty.
- Uses `find_element_by_xpath` / `find_element_by_id`, which is Selenium 3 API. On Selenium 4+ you'll need to switch to `find_element(By.XPATH, ...)`.

This was written to get one specific job done, not as a reusable framework. It works against the site as it looked at scrape time.
