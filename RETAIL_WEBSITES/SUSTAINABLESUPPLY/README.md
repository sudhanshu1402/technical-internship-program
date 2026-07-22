# SustainableSupply product scraper

A Scrapy + Selenium spider that pulls product data from [sustainablesupply.com](https://www.sustainablesupply.com/) across five collections and writes it to CSV. Built as part of a technical internship program.

## What it does

The site loads its collection listings with JavaScript (AngularJS/SearchSpring), so plain Scrapy requests can't see the product links. The spider works around this by driving a real Chrome browser:

1. Selenium opens each collection URL and waits for the listing to render.
2. It reads the product anchor tags off the rendered page and builds absolute product URLs.
3. Scrapy fetches each product page and parses fields with XPath.
4. Selenium clicks the "next" pagination button to walk through the rest of the listing.

Collections scraped:

- restroom-supplies
- electric-motors
- hvac-r-supplies
- plumbing-supplies
- safety-supplies

## Fields extracted

Defined in `SustainablesupplyItem.py`:

| Field | Notes |
|---|---|
| `url` | product page URL |
| `title` | product name |
| `sku` | store SKU |
| `brand` | manufacturer |
| `mpn` | manufacturer part number |
| `weight` | numeric portion only (regex-stripped) |
| `weight_unit` | non-numeric portion of the weight string |
| `price` | dollar sign removed |
| `desc` | falls back to a `<strong>` variant when the plain paragraph is empty |
| `specs` | spec table scraped into a dict |
| `images` | `og:image:secure_url` meta tags, numbered `image_url_1..n` |
| `docs` | linked document URLs (spec sheets etc.), numbered `doc_url_1..n` |

Sample output is checked in: `sustainablesupplyspider.csv` (~29 rows) and `sustainablesupply.csv`. Column order in the CSV is alphabetical.

## Stack

- Python
- [Scrapy](https://scrapy.org/) — crawl engine, item pipeline, XPath extraction
- [Selenium](https://www.selenium.dev/) + ChromeDriver — renders the JS listings and handles pagination clicks

There is no `requirements.txt` in this folder. Install the two libraries directly:

```bash
pip install scrapy selenium
```

You also need Chrome and a matching ChromeDriver.

## Run

This is a Scrapy spider, so it runs with `scrapy crawl` — not `python main.py` (that file is just an unused PyCharm stub). The spider expects to sit inside a standard Scrapy project layout (a `sustainablesupply` package with a `spiders/` module), which matches the imports in the code:

```bash
scrapy crawl sustainablesupplyspider -o output.csv
```

Before running, fix the hard-coded ChromeDriver path near the top of `sustainablesupplyspider.py`:

```python
driver = webdriver.Chrome(
    "C:/Program Files (x86)/Google/Chrome/Application/chromedriver"
)
```

Point it at your own ChromeDriver location.

## Files

- `sustainablesupplyspider.py` — the spider (Selenium setup, three parse callbacks, pagination)
- `SustainablesupplyItem.py` — the item schema
- `pipelines.py` — pass-through pipeline (no transforms)
- `settings.py` — Scrapy settings; `ROBOTSTXT_OBEY = True`, `LOG_LEVEL = "INFO"`
- `middlewares.py` — default Scrapy middleware boilerplate, unmodified
- `main.py` — leftover PyCharm sample, not used
- `*.csv` — sample scraped output

## Honest notes

This is internship / learning work aimed at one specific site, so expect the rough edges of that kind of project:

- **Selenium is a module-level global.** A Chrome window opens the moment the file is imported, and there's no `driver.quit()` — the browser is left running.
- **XPaths are brittle absolute paths** (e.g. `.../div[2]/div/div[1]/div/h1`). They break if the site's markup shifts.
- **The site depends on it.** The XPaths and the SearchSpring listing structure reflect sustainablesupply.com as it was when this was written; there's no guarantee it still matches the live site.
- The Selenium API calls (`find_elements_by_xpath`, `find_element_by_class_name`) are the pre-4.x style and would need updating for current Selenium.
- Every field is wrapped in try/except that falls back to an empty value, so a layout change degrades quietly rather than crashing.

It does the job it was built for — pull structured product data off a JS-rendered storefront — and it's a reasonable example of combining Scrapy with Selenium when a site won't render server-side.
