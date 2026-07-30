# Yokohama India store-locator scraper

Pulls dealers from `yokohama-india.com/storelocator`. The hardest target in this set: store details only exist inside Google Map info-window popups, rendered client-side, so there's no list to parse at all.

Selenium loops every state, then every city within it, sets the distance filter to the widest radius, searches, then scrolls to each map marker and clicks it to open the popup. BeautifulSoup parses that popup's HTML. De-duplication is on address text, since the same dealer surfaces under overlapping city and radius searches. Output: `yokohamatyres.csv`, 28 stores.

## Fields

Standard schema plus `yokohama_zone` (`1` when a `span.ycn_bg` badge is present), `dealer_name`, and `google_maps_direction_url`. `state`, `district`, and `pincode` are all derived from one text blob after the `<br>` by splitting and stripping.

What reliably came back: `brand`, `store_name`, `full_address`, `phone_number`, `state`, `district`, `pincode`, `yokohama_zone`. Empty in practice: `email_id`, `dealer_name`, `google_maps_direction_url`, `manufacturer_unique_id`.

## Run

```bash
scrapy crawl yokohamatyresspider -o yokohamatyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- Fields are located by deep chained `.find_next("p")` calls and absolute XPaths like `//*[@id="map"]/div/div/div[2]/div[3]/div/div[4]`. Unavoidable when the data only lives in a map popup, but it breaks on any layout change.
- Fixed sleeps mean slow-loading popups get missed entirely, not retried.
- Chrome launches at import time.
