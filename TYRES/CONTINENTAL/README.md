# Continental Tyres store-locator scraper

**Unfinished.** The browser automation works end to end, but nothing is extracted or saved.

Targets `continental-tyres.in`, which loads its locator through JavaScript. Selenium opens `/car`, dismisses the consent button, opens the locator search box, types a city (only `Mumbai` is in the `districts` list), and grabs the rendered `page_source` for BeautifulSoup.

Then it just prints the soup. `ContinentaltyresItem` has no fields defined and the pipeline is a pass-through, so there's no CSV.

## Run

```bash
scrapy crawl continentaltyresspider
```

Chrome opens, walks the locator for Mumbai, and dumps parsed HTML to the console. Setup and the chromedriver path fix are in [../README.md](../README.md).

## To finish it

Add real fields to `ContinentaltyresItem`, replace the `print(soup)` with selectors that yield items, extend the one-entry `districts` list, and add an output feed.

Kept as a working skeleton for the "render the JS page" half of the job.
