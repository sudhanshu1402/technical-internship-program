# Birla Tyres dealer scraper

Scrapes the Birla Tyres dealer-locator page and dumps every listed dealer to CSV.

## What it does

The [birla-tyre.in dealer network page](https://www.birla-tyre.in/dealer-network/) has a "search by state or city" box. There's no public API, and the results are rendered by JavaScript, so a plain HTTP request returns nothing useful. This project drives a real Chrome browser through Selenium to type a city into the search box, wait for the dealer cards to render, then parse them.

For each dealer it pulls out the store name, split address lines, pincode, phone number, and state, tags the row with the searched city and the brand `BIRLA`, and yields a Scrapy item. The finished rows are written to `birlatyres.csv`.

It's a Scrapy project only in structure — the actual page fetching is done by Selenium inside the spider's `parse` method, not by Scrapy's downloader.

## Scope

Internship exercise (part of a batch of tyre-brand dealer scrapers). It works against the site as it existed when written, but it's brittle by design:

- Cities are hardcoded: `Mumbai, Akola, Dhule, Delhi, Surat, Bangalore`. Edit the `postals` list in the spider to change them.
- Field extraction walks the DOM by chained `.next_sibling` calls off a `<br>` tag, so any markup change on the site breaks it.
- Timing is handled with fixed `time.sleep(3)` pauses rather than explicit waits.
- Uses the old Selenium 3 API (`find_element_by_xpath`), which was removed in Selenium 4.
- The chromedriver path is a hardcoded Windows path.

The two CSVs in the folder (`birlatyres.csv`, `birlatyresspider.csv`, 50 rows each) are sample output from a real run.

## Stack

- Python
- [Scrapy](https://scrapy.org/) — project layout, item definitions, CSV export
- [Selenium](https://www.selenium.dev/) — drives Chrome to render the JS page
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) + lxml — parses the dealer HTML

## Files

| File | Purpose |
|------|---------|
| `birlatyresspider.py` | The spider — Selenium automation + BeautifulSoup parsing |
| `BirlatyresItem.py` | Scrapy `Item` defining the 11 output fields |
| `pipelines.py` | Pass-through pipeline (no transforms) |
| `settings.py` | Scrapy defaults; `ROBOTSTXT_OBEY = True`, `LOG_LEVEL = "INFO"` |
| `middlewares.py` | Boilerplate Scrapy middleware (unused) |
| `birlatyres.csv` | Sample scraped output |

## Fields collected

`manufacturer_unique_id`, `store_name`, `full_address`, `address_line_one`, `address_line_two`, `email_id`, `phone_number`, `state`, `district`, `pincode`, `brand`

(`manufacturer_unique_id` and `email_id` are always blank — the site doesn't expose them.)

## Run it

This folder holds the spider and support files but not a full Scrapy project skeleton (`scrapy.cfg`, the outer package dir). To run it, place these files under a Scrapy project named `birlatyres` with the spider in `birlatyres/spiders/`, then:

```bash
pip install scrapy selenium beautifulsoup4 lxml

# fix the chromedriver path in birlatyresspider.py first
scrapy crawl birlatyresspider -o birlatyres.csv
```

You need Chrome installed and a matching chromedriver. Update the hardcoded path near the top of `birlatyresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

A browser window opens, gets maximized, and you'll see console lines like `GIVING INPUT IN THE SEARCH BAR --> Mumbai` as it steps through each city.

## Sample output

```
store_name,phone_number,pincode,state,district,brand
M.M.TYRES, 9322351917,400068,MAHARASHTRA,Mumbai,BIRLA
```
