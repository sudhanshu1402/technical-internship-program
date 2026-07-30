# MRF Tyres store-locator scraper

**Unfinished.** It scrolls to the first state and stops. No dealer data is extracted, no items are yielded.

`mrftyresspider.py` drives Chrome to `mrftyres.com`, closes the popup, scrolls the store-locator section into view, and finds the first state entry (`andaman-and-nicobar`). The intent was to iterate every state from there. `MrftyresItem` has no fields, and the pipeline is boilerplate.

## Run

```bash
scrapy crawl mrftyresspider
```

Setup and the chromedriver path fix are in [../README.md](../README.md).

## Where a full version would pick up

The state list is keyed by slugged names (`andaman-and-nicobar`), which is the hook to loop over. Navigation relies on absolute XPaths like `/html/body/div[5]/div[1]/a` to close the popup, so expect those to have drifted.

Kept as an archived record of the work, not a dataset tool.
