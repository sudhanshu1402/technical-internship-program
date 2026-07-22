# Continental Tyres store-locator scraper

A Scrapy project that drives a real Chrome browser (Selenium) through the Continental Tyres India store locator to pull dealer listings for a given city. Built during a technical internship as a web-scraping exercise.

## What it does

The target site (`continental-tyres.in`) loads its store locator through JavaScript, so a plain HTTP request returns an empty shell. This spider works around that by:

1. Opening `https://www.continental-tyres.in/car` in a Chrome window via Selenium.
2. Dismissing the intro/consent button on the landing page.
3. Opening the store-locator search box.
4. Typing a city name (currently `Mumbai`) and pressing Enter.
5. Waiting for results to render, then grabbing the full rendered `page_source` and parsing it with BeautifulSoup.

At the moment the spider prints the parsed HTML (`soup`) to the console. The extraction step that turns that HTML into structured dealer records is not written yet, so `ContinentaltyresItem` is an empty item and the pipeline is a pass-through.

## Status

Partial / learning project. The browser automation runs end to end, but nothing is scraped into fields or saved. Treat it as a working skeleton for the "get the JS-rendered page" half of the job, not a finished scraper.

## Stack

- **Scrapy** — project scaffolding (spider, items, pipeline, middlewares, settings)
- **Selenium** + **chromedriver** — drives Chrome to render JavaScript and interact with the page
- **BeautifulSoup** (`lxml` parser) — parses the rendered HTML

There is no `requirements.txt` in this folder. Install the dependencies directly:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

You also need Google Chrome and a matching **chromedriver**.

## Configuration you must change before running

The spider hardcodes a Windows chromedriver path in `continentaltyresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Point that at your own chromedriver location for it to start.

Also note the imports assume the standard Scrapy layout — the spider imports from `continentaltyres.ContinentaltyresItem` and `settings.py` references `continentaltyres.spiders`. These files sit flat in this folder, so to actually run it you'd place them in a Scrapy project named `continentaltyres` with `continentaltyresspider.py` under a `spiders/` package.

## Files

| File | What it is |
|------|-----------|
| `continentaltyresspider.py` | The spider — all the Selenium browser-driving logic lives here |
| `ContinentaltyresItem.py` | Scrapy item definition (currently empty, no fields defined) |
| `pipelines.py` | Item pipeline (pass-through, no processing yet) |
| `middlewares.py` | Default Scrapy spider/downloader middleware stubs |
| `settings.py` | Scrapy settings — `ROBOTSTXT_OBEY = True`, `LOG_LEVEL = "INFO"` |
| `README` | Original note: target site and start URL |

## Run

From the project root (once the layout above is in place):

```bash
scrapy crawl continentaltyresspider
```

A Chrome window opens, walks through the store locator for Mumbai, and the parsed page HTML is printed to the console.

## To make it actually useful

- Add real fields to `ContinentaltyresItem` (dealer name, address, phone, etc.).
- Replace the `print(soup)` line with selectors that pull those fields out of the rendered HTML and `yield` items.
- Loop over more than one city — the `districts` list already exists for that; it just has one entry.
- Enable an output feed (e.g. `scrapy crawl continentaltyresspider -o dealers.json`) once items are being yielded.
