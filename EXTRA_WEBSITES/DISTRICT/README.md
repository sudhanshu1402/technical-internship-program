# DISTRICT

Scrapes the list of all districts in India from [instapdf.in/india-all-state-district-name-list](https://instapdf.in/india-all-state-district-name-list/). Plain Scrapy, one request, no browser.

```python
district_list = dist.xpath(
    '//*[@id="post-23838"]/div[4]/table/tbody//tr/td[2]/text()'
).extract()
items["district_list"] = district_list
```

## The output shape is wrong, on purpose

`extract()` returns every district name at once, and that whole list goes into the single `district_list` field. So `dist.csv` is one column holding one big comma-joined cell, not one row per district. Yielding one item per name instead would fix it.

The same data in dictionary form is in `All Districts in India Dict Form.docx`.

## Run

```bash
scrapy crawl districtspider -o dist.csv
```

Shared setup is in [../README.md](../README.md).

## Note

The XPath is pinned to that page's exact DOM (`post-23838`, fixed div and table indexes), so it stops matching the moment the source page is rebuilt.
