# Pirelli dealer scraper

Pulls authorised dealers from the [Pirelli India locator](https://www.pirelli.com/tyres/en-in/car/find-your-dealer/dealer-locator).

Selenium dismisses the consent popup, types a city, arrow-downs to the first autocomplete option, presses Enter, clicks search, and reads `#results-panel-box`. BeautifulSoup parses each dealer `<li>`, de-duplicating on `data-dealerid`.

Cities: Mumbai, Akola, Dhule, Delhi, Surat, Bangalore. Output: `pirellityres.csv`, 21 dealers, of which Bangalore returned 14 and the rest returned a handful each.

## Fields

Standard schema. `manufacturer_unique_id` is `data-dealerid`, `full_address` comes from `.dlAddress`, `district` is the searched city. `email_id`, `phone_number`, `state`, and `pincode` stay blank because the locator list doesn't expose them.

```csv
brand,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
PIRELLI,Mumbai,,"Shop 4 & 7, Sahajanand Society, TPS 6, near, Milan Subway Rd, Navpada, MSEB Colony, Santacruz West, 400054 MUMBAI",IN0000000076,,,,BOMBAY TYRE
```

## Run

```bash
scrapy crawl pirellityresspider -o pirellityres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- Chrome launches at import time, since the driver is created outside the spider class.
- The consent popup id (`#cp-accept`) is pinned to Pirelli's markup at the time of writing.
