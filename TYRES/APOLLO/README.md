# Apollo Tyres dealer scraper

Pulls dealer listings from the Apollo "Find a Dealer" locator at `apollotyres.com/en-in/car-suv-van/dealer-finder/find-a-dealer/`.

It's an Angular page with no plain HTML list, so Selenium types a city, presses Down then Enter to take the first autocomplete suggestion, scrolls `#mCSB_1` to the bottom, and clicks "Show more" until that link goes `ng-hide`. The rendered `#dealer-result` HTML then goes to BeautifulSoup.

Cities searched (hardcoded): Mumbai, Akola, Dhule, Delhi, Surat, Bangalore. No CSV committed.

## Fields

Standard schema plus `apollo_zone` (`"1"` if the card is flagged an Apollo Zone). `manufacturer_unique_id` comes from the card's `data-id`, which also drives in-run de-duplication so a dealer appearing under two cities is yielded once.

## Run

```bash
scrapy crawl apollotyresspider -o dealers.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- `state` and `pincode` are left blank in the code despite being in the schema.
- `email_id` is read from a repurposed schema.org `endDate` attribute, which is as brittle as it sounds.
