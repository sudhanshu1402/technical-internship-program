# Bridgestone dealer scraper

A Scrapy + Selenium spider that pulls Bridgestone tyre dealer listings from the Bridgestone India store locator (https://select.bridgestone.co.in/) and writes them to CSV. Built during a technical internship.

## What it does

The store locator is a JavaScript page: you type a district into a search box, hit enter, and it renders a paginated list of dealers. Plain HTTP requests get you nothing, so the spider drives a real Chrome browser through Selenium to do the searching and paging, then hands each dealer's detail-page URL back to Scrapy to fetch and parse.

Flow:

1. Selenium opens the locator and, for each district in a hardcoded list (`Mumbai`, `Akola`, `Dhule`), clears the search box, types the district, and presses Enter.
2. It reads the results container's outer HTML and parses it with BeautifulSoup to find every dealer link (`ul.address-sub > a`).
3. Each dealer link is yielded as a `scrapy.Request` to `parse_items`.
4. It clicks the "next" pagination button and repeats until the button becomes `disabled`.
5. `parse_items` scrapes each dealer page with XPath and yields a structured item.

## Fields collected

Defined in `BridgestonetyresItem.py`:

| Field | Source |
|---|---|
| `store_name` | dealer page heading |
| `full_address` | address line (tabs/newlines stripped) |
| `phone_number` | contact line |
| `google_maps_direction_url` | "get directions" link |
| `appointment_booking_url` | booking link |
| `website` | the dealer page URL itself |
| `brand` | constant `"BRIDGE STONE"` |
| `manufacturer_unique_id`, `email_id`, `state`, `district`, `pincode` | declared but left blank — the source page doesn't expose them |

## Stack

- Python
- [Scrapy](https://scrapy.org/) — request scheduling, item pipeline, CSV export
- [Selenium](https://www.selenium.dev/) — drives Chrome to render and page through the JS locator
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) + lxml — parse the rendered HTML for dealer links

No `requirements.txt` ships with this snapshot. Install the deps directly:

```bash
pip install scrapy selenium beautifulsoup4 lxml itemadapter
```

You also need Chrome and a matching **chromedriver**.

## Run

This is a snapshot of the inner package of a Scrapy project — the files here (`bridgestonetyresspider.py`, `settings.py`, `middlewares.py`, `pipelines.py`, `BridgestonetyresItem.py`) normally sit inside a `bridgestonetyres/` package with a `scrapy.cfg` alongside it. The imports (`from bridgestonetyres.BridgestonetyresItem import ...`) and `SPIDER_MODULES = ["bridgestonetyres.spiders"]` assume that layout. To run it, restore that structure and from the project root:

```bash
scrapy crawl bridgestonetyresspider -o dealers.csv
```

Before running, fix the hardcoded chromedriver path in `bridgestonetyresspider.py`:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Point it at your own chromedriver (or put chromedriver on `PATH` and drop the argument).

## Sample output

`bridge.csv` and `bridgestonetyres.csv` are captured runs. One row (Mumbai):

```
store_name:                 (from locator)
full_address:               8/88, Bharat Nagar, Juhu Versova Link Rd, opp. Banana Leaf
                            Restaurant, Four Bungalows, Mumbai, Maharashtra - 400053
brand:                      BRIDGE STONE
google_maps_direction_url:  https://goo.gl/maps/hargruL99AqfmTQ66
appointment_booking_url:    https://bridgestonebookmyservice.in/
```

`bridge.csv` has ~42 dealer rows, `bridgestonetyres.csv` ~12.

## Notes and caveats

- **Selenium runs at module import time.** The `webdriver.Chrome(...)` call sits at the top of the spider file, so Chrome launches the moment the module loads, not when the crawl starts. Fragile, but it works for a one-off scrape.
- **Old Selenium API.** Uses `find_element_by_xpath`, removed in Selenium 4. Run with Selenium 3.x or port these calls to `find_element(By.XPATH, ...)`.
- **`time.sleep(3)` everywhere** to wait for the JS to settle — brittle against slow pages, but simpler than explicit waits.
- **Hardcoded XPaths** (`/html/body/section[3]/...`) tie the parser to the exact page structure at scrape time; a site redesign breaks it.
- **Fixed district list.** Only Mumbai, Akola, and Dhule are searched. Edit the `districts` list to cover more.
- Most `try/except` blocks in `parse_items` wrap constant assignments (e.g. `brand = "BRIDGE STONE"`) that can't actually throw — leftover scaffolding from a scraper template.
- `pipelines.py` and `middlewares.py` are the default Scrapy stubs; neither is enabled in `settings.py`.

## Scope

A working internship scraper for a single site, written to a fixed page layout with a hardcoded district list. It did its job (the CSVs are the proof) but it's a point-in-time tool, not a maintained library.
