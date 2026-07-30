# RETAIL_WEBSITES

Scrapy spiders that pull product data off four B2B supply and HVAC retail sites. Same job each time: crawl a catalog, extract product details, dump to CSV.

| Folder | Site | Finds products via | Rows |
|---|---|---|---|
| MARKETAIR | marketair.com | 16 hardcoded category pages | 82 |
| AFSUPPLY | afsupply.com | 3 XML sitemaps | 24 |
| SUPPLYHOUSE | supplyhouse.com | product sitemaps + Selenium | 6 |
| SUSTAINABLESUPPLY | sustainablesupply.com | 5 collection pages + Selenium pagination | 14 |

## Two approaches

They split by how the target serves its pages.

**Static HTML** (MARKETAIR, AFSUPPLY): plain Scrapy requests with CSS and XPath selectors. AFSUPPLY reads product URLs from the site's sitemaps via `SitemapSpider` and adds a `requests` plus BeautifulSoup pass for images that don't appear in the main parse.

**JavaScript-rendered** (SUPPLYHOUSE, SUSTAINABLESUPPLY): the spider drives headless Chrome, waits for the DOM, then reads it. SUSTAINABLESUPPLY also clicks through paginated collections.

Every field is wrapped in its own `try/except` falling back to an empty string. Ugly, but across thousands of products one missing `<span>` shouldn't kill a crawl. The row just comes out partial.

## Setup

```bash
pip install scrapy selenium beautifulsoup4 requests
```

Plus Chrome and a matching chromedriver for the two Selenium spiders.

```bash
cd MARKETAIR
scrapy crawl marketairspider -o output.csv
```

Spider names: `marketairspider`, `supplyhousespider`, `afsupplyspider`, `sustainablesupplyspider`.

## Caveats that apply across the set

- **Hardcoded Windows chromedriver path** in both Selenium spiders. Change it first.
- **Pre-4.x Selenium API** (`find_elements_by_xpath`, `find_element_by_class_name`). Pin Selenium 3 or port the calls.
- **Brittle positional XPaths** like `/html/body/main/div[2]/...`, tied to each site's DOM at scrape time.
- **`ROBOTSTXT_OBEY = False`** on some of these. They were internal data-collection tasks. Be deliberate about where you point them.
- **`print`-based logging** rather than Scrapy's logger.
- A leftover PyCharm `main.py` stub sits in a few folders and does nothing.

Internship work, archived as-is. The CSVs are the deliverable.
