# MARKETAIR spider

Scrapes product listings from [marketair.com](https://www.marketair.com/), an HVAC accessories retailer running VirtueMart. Static HTML, plain Scrapy.

Starts from 16 hardcoded category pages, one per product line (AcustiFlex, DeflectAir, DrainHide, DrainMate, DripShield, DSS Switch, EasyBend, LinePort, Perfect Pitch, Pipe Prop, ReversaLine, RoughInBox, SnapFix, SnowShield, SuperSleeve, ValveShield), follows each product link, and parses the page. Output: `marketairspider.csv`, 82 rows.

## Fields

`title`, `price` (`$` stripped), `url`, `desc` (cleaned of `\r \n \t [ ] '`), and `images` as a dict of `image_url_1`, `image_url_2`, and so on from the main plus additional images.

Selectors, all pinned to the VirtueMart markup:

```
title   //*[@id="sidecontent"]/div[2]/h1/text()
price   span.PricebasePrice::text
desc    div.product-description
images  div.main-image a::attr(href) + div.additional-images a::attr(href)
links   div.vm-product-media-container a
```

## Run

```bash
scrapy crawl marketairspider -o marketairspider.csv
```

Shared setup and caveats are in [../README.md](../README.md).

## Notes

- No download delay is set and `ROBOTSTXT_OBEY` is off, so add throttling before pointing this at a live site at any scale.
- `main.py` in this folder is leftover PyCharm boilerplate and does nothing.
