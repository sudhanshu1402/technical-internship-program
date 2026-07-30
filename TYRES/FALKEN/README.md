# Falken Tyres store scraper

Pulls dealers from `falkentyre.in/find-a-store`, which shows nothing until you pick a state from a dropdown and then renders results with JavaScript.

Selenium selects a state, waits for the list, and BeautifulSoup pulls each address block. Output: `falkentyres.csv`, 167 rows.

## Fields

`store_name` from the `<h3>`, `full_address` from the address paragraph, `phone_number` from the element right after it, `brand` hardcoded to `FALKENTYRES`. The rest of the schema is blank on purpose: the locator carries city, state, and pincode inline inside the address string rather than splitting them.

```csv
brand,district,email_id,full_address,manufacturer_unique_id,phone_number,pincode,state,store_name
FALKENTYRES,,,", 1-2 Shreeji Tower Opp. Himalaya Mall Drive-In-Road , Ahmedabad., Gujarat- 380052",,27912501,,,Hemant Tyre Shop
```

## Run

```bash
scrapy crawl falkentyresspider -o falkentyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- **The state loop is unrolled, not looped.** The same select-and-parse block is copy-pasted for dropdown `option[2]`, `option[3]`, and `option[4]`, so it only ever covers three states, not the full site.
- Chrome launches at import time, since the driver is created at module top level.
