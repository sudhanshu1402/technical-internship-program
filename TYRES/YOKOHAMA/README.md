# Yokohama India store-locator scraper

A Scrapy + Selenium spider that walks the Yokohama India dealer locator and pulls out every store's name, address, phone, and zone status into a CSV.

## What it does

The Yokohama India store locator (`https://www.yokohama-india.com/storelocator`) is a
JavaScript form: pick a state, pick a city, pick a search radius, hit search, and it
drops pins on a Google Map. The store details only exist inside the map's info-window
popups, which are rendered client-side — a plain HTTP request never sees them.

So this spider drives a real Chrome browser through Selenium:

1. Loads the store-locator page.
2. Loops over every option in the **state** dropdown.
3. For each state, loops over every option in the **city** dropdown.
4. Sets the distance filter to the 7th option (the widest radius) and clicks search.
5. Scrolls to each result marker, clicks it to open the info window, and reads the
   popup's HTML.
6. Parses that HTML with BeautifulSoup and yields one item per store.

It de-duplicates as it goes: the store's address text is used as a key, and a store
already seen is skipped (the same dealer shows up under overlapping city/radius
searches). Scrapy writes the yielded items out to CSV.

This was built as a tyre-dealer data-collection exercise during a technical internship.
It scrapes one specific site's DOM, so it's tightly coupled to that page's markup as it
existed at the time.

## Fields collected

Defined in `YokohamatyresItem.py`:

| Field | How it's derived |
|---|---|
| `store_name` | `<h3>` inside the info-window |
| `full_address` | first `<p>` in the popup body |
| `phone_number` | a later `<p>` containing "Phone" |
| `email_id` | a later `<p>` containing "Email" |
| `state` | text after the `<br>`, split on comma |
| `district` | same text, with digits/state/punctuation stripped out |
| `pincode` | same text, non-digits stripped via regex |
| `yokohama_zone` | `1` if a `span.ycn_bg` badge is present, else `0` |
| `brand` | hardcoded `"Yokohama"` |
| `google_maps_direction_url` | `href` of the sibling of the address `<p>` |
| `dealer_name` | a `<p>` mentioning "Mr" |
| `manufacturer_unique_id` | always empty (placeholder) |

In the sample output (`yokohamatyres.csv`), the fields that reliably came back are
`brand`, `store_name`, `full_address`, `phone_number`, `state`, `district`, `pincode`,
and `yokohama_zone`. `email_id`, `dealer_name`, `google_maps_direction_url`, and
`manufacturer_unique_id` came back empty because the source popups didn't carry them.

## Tech

- **Scrapy** — project scaffolding, item model, CSV export.
- **Selenium** + **ChromeDriver** — drives the browser through the JS form.
- **BeautifulSoup** (`lxml` parser) — parses each info-window's HTML.

Standard Scrapy files are present: `pipelines.py` is the default pass-through pipeline,
`middlewares.py` is unmodified boilerplate, and `settings.py` keeps defaults with
`ROBOTSTXT_OBEY = True` and `LOG_LEVEL = "INFO"`.

## Running it

There's no `requirements.txt` in this folder. Install the dependencies directly:

```bash
pip install scrapy selenium beautifulsoup4 lxml
```

The spider file expects a full Scrapy project layout (it imports from
`yokohamatyres.YokohamatyresItem` and settings reference `yokohamatyres.spiders`), so
`yokohamatyresspider.py` belongs under `yokohamatyres/spiders/` in that project. From
the project root:

```bash
scrapy crawl yokohamatyresspider -o yokohamatyres.csv
```

**Before it will run, fix the ChromeDriver path.** It's hardcoded to a Windows install:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Point that at your own `chromedriver`, or let Selenium Manager resolve it
(`webdriver.Chrome()` on modern Selenium). A matching Chrome install is required either
way, and the browser opens visibly — this isn't headless.

## Sample output

Rows from `yokohamatyres.csv` (header:
`brand,dealer_name,district,email_id,full_address,google_maps_direction_url,manufacturer_unique_id,phone_number,pincode,state,store_name,yokohama_zone`):

```csv
Yokohama,,Mumbai,,"Opposite Municipal Pumping Station, G.M Road, Next To Ravi Petrol Pump, ChemburMumbai, Maharashtra - 400089",,,Phone: 09867900801 / 09920265789 / 25252080,400089,Maharashtra,Popular tyres,1
Yokohama,,Mumbai,,"Shop No. 6 Eversun Co. Soc Andheri (West) J P Road, Dhake ColoneyMumbai, Maharashtra - 400058",,,Phone: 9820046614,400058,Maharashtra,Tejani Tyre Service,1
```

The committed CSV has 28 store rows.

## Notes and caveats

- **Brittle by design.** Store fields are located by deep chained `.find_next("p")`
  calls and long absolute XPaths (e.g.
  `//*[@id="map"]/div/div/div[2]/div[3]/div/div[4]`). Any change to the site's layout
  breaks extraction. That's inherent to scraping a live third-party DOM.
- **Timing via `sleep`.** Navigation waits on fixed `time.sleep()` calls rather than
  Selenium explicit waits, so runs are slow and can miss slow-loading popups.
- **Selenium version.** Uses the old `find_element_by_id` / `find_elements_by_name`
  API, which was removed in Selenium 4.3+. Run it on an older Selenium, or port the
  calls to `driver.find_element(By.ID, ...)`.
- **Global driver.** The WebDriver is created at import time, so simply importing the
  module launches Chrome.
