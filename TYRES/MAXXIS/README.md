# Maxxis Tyres dealer scraper

Pulls the full dealer list from `maxxistyres.in/dealer/locator`, which renders in JavaScript.

Selenium loads the page, waits, and reads the rendered `#dealers-list`. BeautifulSoup then yields one item per `li.item.mx-4`. Output: `maxxistyres.csv`, 2,412 dealer records across India, the biggest run in this set.

## Fields

Standard schema plus `dealer_name` (the contact person). `manufacturer_unique_id` is the list item's `value` attribute. `email_id` is always blank, kept only for schema consistency.

```
brand,dealer_name,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
```

## Run

```bash
scrapy crawl maxxistyresspider -o maxxistyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- **One shot, no pagination.** The page is fetched once behind a fixed `time.sleep(5)`. If the site is slow the list may not have fully rendered, and there's no scroll or paging beyond whatever loads on open.
- Field extraction leans on sibling walking (`div.p.next_sibling.next_sibling...`) because the markup has no clean per-field classes. A layout change makes fields come back silently empty rather than erroring.
