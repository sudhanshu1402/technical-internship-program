# Michelin dealer locator scraper

Two spiders against `michelin.in`, one for car tyres (`CAR/`) and one for motorbike (`BIKE/`). The locator is a JS app where cards load after you type a city and click "load more".

Selenium closes the intro popup, types the city, takes the first autocomplete suggestion, then clicks "load more" until it goes `display: none;`. BeautifulSoup parses every `<li class="b2c-dl-result-card">` out of `#b2c-dl-result-list`, de-duplicating on `data-dealer-id`.

Cities: Mumbai, Akola, Dhule, Delhi, Surat, Bangalore. Both CSVs hold 312 dealer rows.

## Fields

Standard schema plus `michelin_certified_centre` (`1` when the card carries `--recommended`), `google_maps_direction_url`, `website`, `appointment_booking_url`, and `brand_website_store_url`. `manufacturer_unique_id` is `data-dealer-id`. `email_id`, `state`, and `pincode` are always empty since the cards don't carry them.

## Run

```bash
scrapy crawl michelincartyresspider -o michelincartyres.csv
scrapy crawl michelinbiketyresspider -o michelinbiketyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md). Note both variants need their own restored Scrapy project (`michelincartyres`, `michelinbiketyres`).

## Brand-specific gotchas

- **Real copy-paste bug:** the `appointment_booking_url` branch checks for the *website* link but reads the *appointment* link, so it can throw and fall back to empty.
- The car and bike projects are about 95% duplicated code. Only the name, item class, and start URL differ.
