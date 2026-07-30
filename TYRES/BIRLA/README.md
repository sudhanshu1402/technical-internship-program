# Birla Tyres dealer scraper

Pulls dealers from the [birla-tyre.in dealer network](https://www.birla-tyre.in/dealer-network/) page, which renders results in JavaScript behind a "search by state or city" box.

Selenium types each city and waits for the cards, then BeautifulSoup parses them. Scrapy is here for the item model and CSV export only; the fetching all happens inside the spider's `parse` method.

Cities searched (the `postals` list): Mumbai, Akola, Dhule, Delhi, Surat, Bangalore. Output: `birlatyres.csv` and `birlatyresspider.csv`, ~50 rows each.

## Fields

Standard schema. `manufacturer_unique_id` and `email_id` are always blank because the site doesn't expose them. Sample row:

```
store_name,phone_number,pincode,state,district,brand
M.M.TYRES, 9322351917,400068,MAHARASHTRA,Mumbai,BIRLA
```

## Run

```bash
scrapy crawl birlatyresspider -o birlatyres.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotcha

Field extraction walks the DOM by chained `.next_sibling` calls off a `<br>` tag. Any markup change breaks it outright.
