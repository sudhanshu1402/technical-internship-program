# Firestone Tyre Dealer Scraper

A Scrapy + Selenium spider that pulls Firestone's dealer/store listings for India from their store-locator page and writes them to CSV.

## What it does

The store locator at `firestonetyre.co.in/our-stores.php` is a JavaScript form: you pick a state, then a city, then hit search, and the matching dealers render into hidden result tables on the page. There's no plain API behind it, so a normal Scrapy HTTP request just gets the empty form.

This spider drives a real Chrome browser with Selenium to work the dropdowns, then hands the rendered HTML to BeautifulSoup to pull the fields out. For every state option it loops through every city option, clicks search, and reads three result containers on the page:

- `#dealerloc`
- `#dealerlocfs100`
- `#dealerloc_bridge`

Each container is parsed the same way. Rows that say "No Dealer Found!" are skipped, and a running `keys` list of already-seen addresses de-duplicates so the same dealer isn't emitted twice across city/state passes.

Fields collected per dealer (`FirestonetyreItem`):

| Field | Source |
|---|---|
| `store_name` | first `<td>` text |
| `full_address` | sibling `<td>` text, with the phone block stripped out |
| `phone_number` | mobile number after the `<br>` in `div.show-info` |
| `tel_number` | landline (the `show-info` text minus the mobile), `Tel. No.:` / `NA` stripped |
| `district` | `td.dis-info` text |
| `brand` | hardcoded `"FIRE STONE"` |
| `manufacturer_unique_id`, `email_id`, `state`, `pincode` | present on the item but left blank — the page doesn't expose them |

`firestonetyre.csv` is a sample of the output (~71 rows), e.g.:

```
brand,district,full_address,phone_number,store_name,tel_number
FIRE STONE,Hyderabad ,"Road No# 7, Plot No# 16/C, Ida, Nacharam, Ranga Reddy Dist.,Hyderabad 500076", Mob. No.: 9848887755,Srikanth Associates,
```

## Stack

- Python
- [Scrapy](https://scrapy.org/) — item model, spider framework, CSV export
- [Selenium](https://www.selenium.dev/) — drives Chrome to render the JS dropdowns
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) (`lxml` parser) — HTML parsing

There's no `requirements.txt` in this folder. Install what it imports:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

You also need Chrome and a matching **chromedriver**. The path is hardcoded near the top of the spider:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Change that to your own chromedriver location before running.

## Layout

This folder holds the spider plus the standard Scrapy project files, but it's a fragment of a larger `firestonetyre` Scrapy project, not a self-contained one:

- `firestonetyrespider.py` — the spider (belongs in `firestonetyre/spiders/`)
- `FirestonetyreItem.py` — the `FirestonetyreItem` field definitions
- `settings.py` — Scrapy settings (`ROBOTSTXT_OBEY = True`, `LOG_LEVEL = INFO`)
- `pipelines.py` — pass-through pipeline (no transforms)
- `middlewares.py` — Scrapy's default middleware boilerplate, unmodified
- `firestonetyre.csv` — sample scraped output
- `README` — the original one-liner with the target URLs

The imports (`from firestonetyre.FirestonetyreItem import ...`, `SPIDER_MODULES = ["firestonetyre.spiders"]`) expect the normal Scrapy package structure. To run it you'd drop these files back into a `firestonetyre` project laid out the usual way and launch with:

```bash
scrapy crawl firestonetyrespider -o firestonetyre.csv
```

## Notes

- The dropdown-walking is O(states × cities) with `time.sleep()` between every click, so a full run is slow by design — it's waiting on the page's AJAX, not throttling politely.
- The per-field `try/except` blocks around constant assignments (e.g. `state = ""` wrapped in a try) are dead scaffolding, kept uniform across every field so the pattern is copy-pasteable.
- The three result-container blocks are near-identical duplicated code, one per table id the page uses.

Scope: this is scraper work from an internship batch of tyre-brand dealer-locator jobs (see the sibling folders under `TYRES/`). It targets one live site's DOM as it existed then, so selectors may have since drifted.
