# Y-API price table — recomputed, not copied

**English** · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · [Español](README.es.md) · [Deutsch](README.de.md) · [Português](README.pt.md)

> **Disclosure:** I work on Y-API, so this is a first-party tool reading a first-party file. Everything it prints comes from <https://y-api.bestvirtualgoods.com/pricing.json>, which is public and needs no API key — run the script and check the output against the source yourself.

One script, no dependencies, no API key. It reads Y-API's published price file and renders [table.md](table.md) with **cash** prices — the number that actually leaves your card, and the only one you can compare against another provider's list price.

## Why bother

Y-API quotes prices in **credit**, and right now `$1` paid buys `$20` of credit. Credit is what the balance is charged; it is not what your card is charged. A table that stops at credit makes every model look 20× more expensive than it is, and a table that hard-codes "divide by 20" quietly becomes wrong the day the promotional rate ends. This script reads the rate out of the same file it reads the prices from.

## Run it

```bash
node price-table.mjs > table.md          # fetch the live file
PRICE_JSON=./pricing.json node price-table.mjs   # render a saved copy, offline
```

Requires Node 18+ (for global `fetch`). `table.md` in this repo is regenerated every Monday by a GitHub Action and committed only when it changes, so the commit history *is* the price-change log.

## Two rules the script keeps

Breaking either one produces a table that quietly contradicts the site it came from.

1. **The cash price is recomputed, not copied.** `cash_price` is published in the file, but the script recalculates `credit_price / top_up.quota_rate` and **fails loudly** if the two disagree. A stale rate or a hand-edited number cannot slip into the table.
2. **`vendor_cheaper_on_cached_input` is never printed alone.** That flag says the vendor's cached-input rate is below our cash price — we are the more expensive option for a cache-heavy workload. Some rows carry it while sitting on a vendor rate card marked `historical-price`, and the Y-API site deliberately excludes those from its "we lose on cache" count. Print the flag without the caveat and a reader walks away with the wrong count. So the table ships both columns.

## Reading the output

- **The catalog** — every model, credit and cash, per 1M tokens, sorted by price.
- **Against the vendor's own list price** — the 11 models where the vendor publishes a price we verified and cited, with the multiple, the cache flag, the caveats, the verification date and a link to the vendor's page. The remaining 4 are listed as having no verified vendor price; the file says so explicitly rather than estimating.

`× ours` reads as "the vendor charges this many times our cash price" — a large multiple is Y-API being cheaper, not more expensive.

## Scope, honestly

- The table is only as current as the source file. It carries the source's `synced_at` and the render date in its header; if both are old, nothing here is news.
- This renders one gateway's prices. It does not compare across providers, and it does not track anyone else's price history.
- Prices move, and the top-up rate is promotional with no announced end date. Anything you decide from this table should be re-derived from the source, not from a copy of `table.md`.

---

*Part of [the Y-API profile](https://github.com/zhuo01-l2ab).*
