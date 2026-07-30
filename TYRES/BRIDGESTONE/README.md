# Bridgestone dealer scraper

Pulls dealers from the Bridgestone India locator at `select.bridgestone.co.in`.

This one is a genuine hybrid: Selenium searches and pages, collects dealer detail links (`ul.address-sub > a`), and hands each back to Scrapy as a `Request`. `parse_items` then scrapes the detail page with XPath. It clicks "next" until the button goes `disabled`.

Districts searched (hardcoded): Mumbai, Akola, Dhule. Output: `bridge.csv` (~42 rows) and `bridgestonetyres.csv` (~12).

## Fields

Standard schema plus `google_maps_direction_url`, `appointment_booking_url`, and `website` (the dealer detail URL). `brand` is the constant `"BRIDGE STONE"`. `manufacturer_unique_id`, `email_id`, `state`, `district`, and `pincode` are declared but never populated.

## Run

```bash
scrapy crawl bridgestonetyresspider -o dealers.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- **Chrome launches at import time.** The `webdriver.Chrome(...)` call sits at module top level, so the browser opens the moment the module loads rather than when the crawl starts.
- Absolute XPaths (`/html/body/section[3]/...`) tie the parser to the exact page structure at scrape time.
- Most `try/except` blocks in `parse_items` wrap constant assignments that can't throw. Leftover template scaffolding.
