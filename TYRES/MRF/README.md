# MRF Tyres store-locator scraper

A Scrapy + Selenium spider that opens the MRF Tyres store locator and starts walking its state-by-state dealer list. One of the tyre-brand scrapers built during a technical internship.

## What it does

`mrftyresspider.py` drives a real Chrome browser (via Selenium) to `https://www.mrftyres.com/`, closes the site's popup, scrolls the store-locator section into view, and locates the first state entry (`andaman-and-nicobar`) in the location list. The intent was to iterate every state and pull dealer details into Scrapy items.

Be honest about the state of it: this is an unfinished practice scraper. The spider stops after scrolling to the first state — it never extracts dealer data or yields items. The Scrapy plumbing around it (item, pipeline) is still boilerplate:

- `MrftyresItem.py` — empty `Item` class, no fields defined.
- `pipelines.py` — pass-through pipeline that returns items unchanged.
- `settings.py` — default Scrapy config; `ROBOTSTXT_OBEY = True`, `LOG_LEVEL = "INFO"`.

## Stack

- Python
- [Scrapy](https://scrapy.org/) — spider framework, item/pipeline scaffolding
- [Selenium](https://www.selenium.dev/) — drives Chrome for the JavaScript-rendered store locator
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/) — imported for HTML parsing (not yet used in the spider)

There is no `requirements.txt` in this folder. Install what the code imports:

```bash
pip install scrapy selenium beautifulsoup4
```

## Run it

This folder holds only the spider and Scrapy module files (`spiders/`, `items`, `pipelines`, `settings`), not a full Scrapy project tree. To run it you'd drop these into a Scrapy project named `mrftyres` (the imports and `SPIDER_MODULES` expect that package layout) and launch:

```bash
scrapy crawl mrftyresspider
```

Before it can run you also need:

- A local ChromeDriver. The path is hardcoded to Windows:

  ```python
  driver = webdriver.Chrome("C:/Program Files (x86)/Google/Chrome/Application/chromedriver")
  ```

  Change this to your own ChromeDriver path (and matching Chrome version) on any other machine.

## Notes

- The navigation logic is brittle by design of the site: it relies on absolute XPaths (e.g. `/html/body/div[5]/div[1]/a` to close the popup) and fixed `time.sleep(3)` waits rather than explicit waits, so any layout change on mrftyres.com breaks it.
- Uses `driver.find_element_by_xpath(...)`, which is the old Selenium 3 API. On Selenium 4+ you'd switch to `driver.find_element(By.XPATH, ...)`.
- The state list is keyed by slugged names (`andaman-and-nicobar`), which is the hook a full version would loop over.

## Scope

Learning exercise / internship task — a starting point for scraping MRF's dealer network, not a finished dataset tool. Kept here as an archived record of the work.
