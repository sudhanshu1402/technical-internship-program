# AngelMatch scraper

A Scrapy + Selenium spider that scrapes investor profiles from [angelmatch.io](https://angelmatch.io).

## What it does

AngelMatch lists angel investors through an Algolia-backed search UI, so the results
are rendered client-side in JavaScript and never appear in the raw HTML that a plain
HTTP request returns. This spider works around that: it drives a real Chrome browser
with Selenium to load each search page, waits for the JS to render the `ais-hits`
result cards, then hands the rendered page source to BeautifulSoup to pull the fields
out.

For every investor card it collects:

- `name` — investor name
- `company_name` — their firm
- `email_id` — the contact link (`href` off the title anchor)
- `locations`
- `links` — social/external links from the card, as a numbered dict
- `investment_focuses` — focus tags, as a numbered dict
- `companies_invested_in` and `companies_invested_in_links`
- `website`

The fields are defined in `AngelmatchItem.py`. After yielding an item, the spider also
navigates to the investor's `website` and prints the page source (a leftover
follow-through step, not stored anywhere).

## Stack

- Python
- [Scrapy](https://scrapy.org/) — spider framework, item model, settings
- [Selenium](https://www.selenium.dev/) with ChromeDriver — renders the JS pages
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) (`lxml` parser) — HTML parsing

There is no `requirements.txt` in this folder. Install the deps manually:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

You also need Chrome and a matching ChromeDriver.

## Layout

These files are meant to sit inside a standard Scrapy project named `angelmatch`
(the spider imports `from angelmatch.AngelmatchItem import AngelmatchItem` and the
settings reference `angelmatch.spiders`). What's checked in here:

| File | Role |
|------|------|
| `angelmatchspider.py` | the spider — belongs under `angelmatch/spiders/` |
| `AngelmatchItem.py` | the scraped item's field definitions |
| `settings.py` | Scrapy settings |
| `pipelines.py` | pass-through item pipeline (does nothing yet) |
| `middlewares.py` | default Scrapy middleware stubs |

## Running it

The ChromeDriver path is hardcoded to a Windows location at the top of
`angelmatchspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change that to your own ChromeDriver path before running. Then, from the Scrapy
project root:

```bash
scrapy crawl angelmatchspider -o investors.json
```

The spider loops over the first 6 result pages
(`https://angelmatch.io/?q=&hPP=3&idx=demo_INVESTORS&p=0..5`), sleeping 3 seconds
after each page load to let the JS render.

## Notes on scope

This is a working-draft scraper, not a polished tool. A few things are worth knowing:

- **Selectors are brittle.** Fields are matched on inline `style` attributes
  (e.g. `"font-size:0.7rem"`, `"color: purple;..."`) rather than stable classes,
  so any markup change on the site breaks extraction. That's a property of the
  target site's HTML, not a choice.
- **A single `AngelmatchItem` instance is reused** across every card in `parse`,
  with its fields overwritten each loop. It's yielded per card, so with Scrapy's
  default synchronous item handling this happens to work, but it's a known
  anti-pattern — better to create a fresh item per card.
- **The pipeline is a no-op.** Output goes straight through; use `-o` to write a file.
- **ChromeDriver is created at import time** and the paths/pagination are hardcoded.
- Every extraction is wrapped in try/except that falls back to `""`, so missing
  fields won't crash the crawl.

Built as a portfolio/learning scraping project.
