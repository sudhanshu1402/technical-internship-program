# Metzeler dealer scraper

Pulls motorcycle-tyre dealers from the [Metzeler India locator](https://www.metzeler.com/en-in/dealer-locator), where each dealer's details sit behind a "goto" link that opens a detail card.

Selenium loads the page, dismisses the consent popup (`#cp-decline`), then for every `#goto-details` element clicks in, reads the detail card's input `value` attributes, and clicks back out. Output: `metzelertyres.csv` and `metzelertyresspider.csv`, 46 dealers.

Runs headed on purpose, since the flow depends on clicking real elements.

## Fields

Standard schema. `manufacturer_unique_id` is Metzeler's internal dealer ID (e.g. `IN0000000041`). `email_id`, `state`, and `pincode` come out blank because the detail card doesn't surface them.

## Run

```bash
scrapy crawl metzelertyresspider -o dealers.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotcha, and a real bug

The item object is created **once outside the loop** and mutated each iteration, so `yield items` yields the same object repeatedly. Scrapy serializes on yield, so the CSV came out correct anyway, but this is a bug waiting to bite. Build a fresh `MetzelerItem()` per dealer.
