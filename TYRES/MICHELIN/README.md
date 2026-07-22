# Michelin Dealer Locator Scraper (India)

Scrapy + Selenium spiders that pull Michelin's dealer/store listings for India off `michelin.in` and dump them to CSV. Two variants: car tyres (`CAR/`) and motorbike tyres (`BIKE/`).

Built as part of a technical internship program.

## What it does

Michelin's dealer locator is a JavaScript app — the store cards aren't in the initial HTML, they load after you type a city and click through "load more". Plain Scrapy can't see them. So these spiders drive a real Chrome window with Selenium to do the interaction, then hand the rendered HTML to BeautifulSoup for parsing.

For each of a fixed list of cities (`Mumbai`, `Akola`, `Dhule`, `Delhi`, `Surat`, `Bangalore`) the spider:

1. Opens the store locator page and closes the intro pop-up.
2. Types the city into the search box, picks the first autocomplete suggestion (arrow-down + enter).
3. Clicks the "load more" button in a loop until it disappears (`display: none;`), so all results for that city are on the page.
4. Grabs the `#b2c-dl-result-list` HTML and parses every `<li class="b2c-dl-result-card">`.
5. Deduplicates on `data-dealer-id` (a `keys` list) so a dealer showing up under two cities is only emitted once.

Each dealer becomes a Scrapy item with these fields:

| Field | Source |
|---|---|
| `manufacturer_unique_id` | `data-dealer-id` on the card |
| `store_name` | dealer name div |
| `full_address` | address div (newlines stripped) |
| `phone_number` | phone div, blank if missing |
| `district` | the search city being scraped |
| `michelin_certified_centre` | `1` if card has the `--recommended` class, else `0` |
| `brand` | hardcoded `MICHELIN` |
| `google_maps_direction_url` | the "get direction" link |
| `website` | the dealer website link, if present |
| `appointment_booking_url` | the appointment link |
| `brand_website_store_url` | dealer detail page on michelin.in |
| `email_id`, `state`, `pincode` | always emitted empty (not on the cards) |

Every extraction is wrapped in its own `try/except` that falls back to an empty string, so a single missing element on one card doesn't kill the run.

## Stack

- **Python** + **Scrapy** — spider framework and item/field definitions
- **Selenium** — drives Chrome to render the JS page and click through pagination
- **BeautifulSoup** (`lxml` parser) — parses the rendered card HTML
- No `requirements.txt` in the repo; you'll need `scrapy`, `selenium`, `beautifulsoup4`, and `lxml` installed.

## Layout note

This is a flattened archive of two separate Scrapy projects. The spiders import from package paths that assume the original project structure (`michelincartyres.spiders`, `michelinbiketyres.spiders`, and the settings' `SPIDER_MODULES`). To actually run them you'd restore that structure — each project needs its `spiders/` package and the item file importable as `michelin<car|bike>tyres.Michelin<Car|Bike>tyresItem`.

```
MICHELIN/
├── CAR/
│   ├── michelincartyresspider.py    # the spider
│   ├── MichelincartyresItem.py      # Scrapy Item (14 fields)
│   ├── pipelines.py                 # pass-through pipeline
│   ├── middlewares.py
│   ├── settings.py                  # ROBOTSTXT_OBEY = True
│   ├── michelincartyres.csv         # scraped output, 312 dealers
│   └── README                       # site + start URL
└── BIKE/                            # same shape, bike variant
```

The car and bike spiders are near-identical; the bike one just has a different `name`, item class, and start URL.

## Run

You need Chrome plus a matching `chromedriver`. The driver path is hardcoded to a Windows location at the top of each spider:

```python
driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
```

Edit that to your own chromedriver path (and OS) before running. Then, from inside a restored Scrapy project:

```bash
scrapy crawl michelincartyresspider -o michelincartyres.csv
# or, for bikes:
scrapy crawl michelinbiketyresspider -o michelinbiketyres.csv
```

A Chrome window opens and drives itself — don't touch it while it runs. Expect it to be slow: there are fixed `time.sleep(2..5)` waits after every action.

## Sample output

A row from `CAR/michelincartyres.csv`:

```
store_name:               SIGN TYRES
full_address:             264, Panchshil, Shop No. 1 & 2, OPP SION HOSPITAL... 400022 MUMBAI
phone_number:             9711999394
district:                 Mumbai
michelin_certified_centre: 1
manufacturer_unique_id:   132115069
google_maps_direction_url: https://www.google.com/maps?daddr=19.076,72.878
```

Both CSVs in the repo hold 312 dealer rows.

## Honest scope

Practice/internship scraper, not production. Known rough edges:

- Chromedriver path and city list are hardcoded — no config or CLI args.
- Timing is fixed `sleep()` calls rather than proper waits, so it's brittle if the page is slow.
- `state`, `pincode`, `email_id` are collected as empty on purpose (the cards don't carry them).
- The `appointment_booking_url` branch checks for the *website* link but reads the *appointment* link, so it can throw and fall back to empty — a small copy-paste bug.
- The two projects are 95% duplicated code.
- Site markup (`b2c-dl-*` classes, XPaths) drifts over time; selectors will eventually go stale.
