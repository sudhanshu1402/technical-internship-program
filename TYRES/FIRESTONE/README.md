# Firestone dealer scraper

Pulls dealers from `firestonetyre.co.in/our-stores.php`, a JavaScript form where you pick a state, then a city, then search.

Selenium works the dropdowns for every state and city combination, then BeautifulSoup reads three separate result containers on the page: `#dealerloc`, `#dealerlocfs100`, and `#dealerloc_bridge`. Rows reading "No Dealer Found!" are skipped, and a `keys` list of seen addresses de-duplicates across passes. Output: `firestonetyre.csv`, ~71 rows.

## Fields

Standard schema plus `tel_number` (the landline, which is the `show-info` text minus the mobile, with `Tel. No.:` and `NA` stripped). `district` comes from `td.dis-info`, `brand` is `"FIRE STONE"`. `manufacturer_unique_id`, `email_id`, `state`, and `pincode` stay blank.

## Run

```bash
scrapy crawl firestonetyrespider -o firestonetyre.csv
```

Setup, the chromedriver path fix, and the caveats common to every spider here are in [../README.md](../README.md).

## Brand-specific gotchas

- Walking every state crossed with every city, with a `time.sleep()` between each click, makes a full run genuinely slow. That's waiting on the page's AJAX, not polite throttling.
- The three result-container blocks are near-identical duplicated code, one per table id.
- Per-field `try/except` around constant assignments is dead scaffolding, kept uniform so the pattern stays copy-pasteable.
