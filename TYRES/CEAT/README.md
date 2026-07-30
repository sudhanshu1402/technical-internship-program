# CEAT dealer scraper

Pulls dealers from the CEAT store locator at [ceat.com/tyre-shop.html](https://www.ceat.com/tyre-shop.html), searching one pincode at a time.

Selenium opens the page, closes the callback popup, clicks into the locator, then loops the pincode list entering each and hitting Enter. The rendered `#parentNode` goes to BeautifulSoup.

Pincodes searched, a small sample across Mumbai, Delhi, and Punjab:

```
400059, 400069, 400037, 110032, 110033, 144205, 144206, 152116, 152117
```

Output: `ceattyres.csv` and `ceattyresspider.csv`, ~70 dealers each.

## Fields

Standard schema plus `ceat_shoppe` (`1` when the "CEAT Shoppe" label is visible). `manufacturer_unique_id` is the dealer code from the WhatsApp icon's `data-dealercode`, which also de-duplicates dealers appearing under two adjacent pincodes. `email_id`, `state`, and `district` are always blank.

## Run

```bash
scrapy crawl ceattyresspider -o dealers.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotcha

Several selectors are absolute paths like `/html/body/div[5]/...`, so any CEAT layout change snaps them.
