# CEAT Tyre Dealer Scraper

A Scrapy + Selenium spider that scrapes CEAT tyre dealer listings from the store locator on [ceat.com](https://www.ceat.com/tyre-shop.html), one pincode at a time, and dumps them to CSV.

## What it does

CEAT's store locator (`https://www.ceat.com/tyre-shop.html`) is a JavaScript page — you type a pincode, it loads nearby dealers into the DOM. Plain HTTP requests get you nothing, so this spider drives a real Chrome browser through Selenium to:

1. Open the tyre-shop page.
2. Close the callback popup.
3. Click into the store locator.
4. Loop over a hardcoded list of pincodes, entering each into the search box and hitting Enter.
5. Grab the rendered dealer list (`#parentNode`), parse it with BeautifulSoup, and pull each dealer's details.
6. De-duplicate by dealer code (a dealer near two adjacent pincodes only gets saved once) and yield a Scrapy item.

The pincodes it covers are a small sample spanning a few regions (Mumbai `400xxx`, Delhi `110xxx`, Punjab `144xxx`/`152xxx`):

```
400059, 400069, 400037, 110032, 110033, 144205, 144206, 152116, 152117
```

## Fields collected

Defined in `CeattyresItem.py`, one row per dealer:

| Field | Source |
|---|---|
| `manufacturer_unique_id` | dealer code from the WhatsApp icon's `data-dealercode` |
| `store_name` | `.store-name` |
| `full_address` | `.address` |
| `phone_number` | `.contact-no` |
| `pincode` | the search pincode being queried |
| `ceat_shoppe` | `1` if the "CEAT Shoppe" label is visible, else `0` |
| `brand` | constant `"CEAT"` |
| `email_id`, `state`, `district` | present in the schema but always left blank (not on the page) |

Two sample outputs are checked in: `ceattyres.csv` and `ceattyresspider.csv` (~70 dealers each).

## Stack

- **Python** with **Scrapy** — spider framework, item model, CSV export
- **Selenium** (Chrome via `chromedriver`) — drives the JS page
- **BeautifulSoup4** — parses the dealer HTML pulled off the live DOM

There is no `requirements.txt`; install the three libraries directly.

## Run it

This is a single spider from a Scrapy project (the surrounding project layout expects it under `ceattyres/spiders/`).

```bash
pip install scrapy selenium beautifulsoup4

# from the Scrapy project root
scrapy crawl ceattyresspider -o dealers.csv
```

Before running you need a matching **chromedriver**. The path is hardcoded near the top of `ceattyresspider.py`:

```python
driver = webdriver.Chrome(
    "C:/Program Files (x86)/Google/Chrome/Application/chromedriver",
    chrome_options=chrome_options,
)
```

Edit that to your own chromedriver location (and it's a Windows path — change it for macOS/Linux).

## Notable details

- **Hybrid approach.** Scrapy's request engine isn't really used here — the whole scrape happens inside `parse` via a module-level Selenium driver. Scrapy is along for the item model and the CSV feed export.
- **`time.sleep(3)` between every step.** Waits are fixed sleeps, not explicit Selenium waits, so it's slow and can break if the page is slower than 3 seconds.
- **XPaths are brittle.** Several selectors are absolute paths like `/html/body/div[5]/...`; any CEAT layout change will snap them.
- **De-dup is in-memory.** A `keys` list holds seen dealer codes for the run, so overlapping pincode results don't produce duplicate rows.

## Scope

Practice/internship scraping exercise from a tyre-dealer data collection task. It targets one live site with a fixed sample of pincodes and a hardcoded driver path — useful as a reference for the Scrapy-plus-Selenium pattern, not as a maintained tool. Selectors will need updating to match the current CEAT site.
