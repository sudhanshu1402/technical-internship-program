# AngelMatch scraper

Scrapes angel investor profiles from [angelmatch.io](https://angelmatch.io), which lists them through an Algolia-backed search UI rendered entirely client-side.

Selenium loads each search page, waits for the `ais-hits` cards to render, and hands the page source to BeautifulSoup. It loops the first 6 result pages (`?q=&hPP=3&idx=demo_INVESTORS&p=0..5`) with a 3-second sleep after each load.

## Fields

`name`, `company_name`, `email_id` (the `href` off the title anchor), `locations`, `website`, plus numbered dicts for `links`, `investment_focuses`, `companies_invested_in`, and `companies_invested_in_links`.

## Run

```bash
scrapy crawl angelmatchspider -o investors.json
```

Shared setup and the chromedriver path fix are in [../README.md](../README.md).

## Notes

- **Selectors match on inline `style` attributes** like `"font-size:0.7rem"` rather than stable classes, because that's what the site's HTML gives you. Any markup change breaks extraction.
- **One `AngelmatchItem` instance is reused** across every card, overwritten each loop. It's yielded per card and Scrapy's synchronous item handling means it happens to work, but it's the same anti-pattern as the Metzeler spider. A fresh item per card is the fix.
- After yielding, the spider navigates to the investor's website and prints the page source. Leftover follow-through, stored nowhere.
