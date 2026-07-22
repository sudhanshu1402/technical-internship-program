# DISTRICT

A small Scrapy spider that scrapes the list of all districts in India from a public webpage.

## What it does

`districtspider.py` fetches [instapdf.in/india-all-state-district-name-list](https://instapdf.in/india-all-state-district-name-list/), reads the HTML table on that page, and pulls the district-name column out of every row. The scraped names end up in `dist.csv`.

This is a practice web-scraping exercise, not a maintained tool. It learns the Scrapy basics: a spider, an item, a pipeline, and settings.

## Files

| File | What it is |
|------|-----------|
| `districtspider.py` | The spider. Parses the table via XPath and yields district names. |
| `items.py` | Defines `DistrictItem` with a single field, `district_list`. |
| `pipelines.py` | `DistrictPipeline` — pass-through stub, does nothing to the item. |
| `settings.py` | Scrapy config. Mostly defaults; `ROBOTSTXT_OBEY = True`. |
| `middlewares.py` | Generated Scrapy middleware boilerplate (unused). |
| `dist.csv` | The scrape output — every district name, comma-joined into one cell. |
| `All Districts in India Dict Form.docx` | Reference copy of the same data in dictionary form. |

## Stack

- Python
- [Scrapy](https://scrapy.org/)

## Run it

These files are the spider-level pieces of a Scrapy project (they import from the `district` package). To run them, place `districtspider.py` under `district/spiders/` and the rest under the `district/` package, add a `scrapy.cfg`, then:

```bash
pip install scrapy
scrapy crawl districtspider -o dist.csv
```

The spider prints each batch of scraped names to the console as it runs.

## How the parse works

```python
table = response.xpath('//*[@id="post-23838"]/div[4]/table/tbody')
district_list = dist.xpath(
    '//*[@id="post-23838"]/div[4]/table/tbody//tr/td[2]/text()'
).extract()
items["district_list"] = district_list
```

`extract()` returns a list of all district names at once, and that whole list is stored in the single `district_list` field. So the output CSV is one column with one big cell rather than one row per district — see `dist.csv`. Splitting it into a row per district would mean yielding one item per name instead.

## Notes

- The XPath is pinned to that page's exact DOM (`post-23838`, fixed div/table indexes). If the source page changes its markup, the spider stops matching.
- `settings.py` and `middlewares.py` are the default Scrapy scaffolding; the pipeline is a no-op.
- Practice project. The scraped data is already saved in `dist.csv` and the `.docx`.
