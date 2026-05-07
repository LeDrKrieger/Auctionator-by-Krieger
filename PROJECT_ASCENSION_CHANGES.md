# Auctionator 3.3.5 Project Ascension Changes

This copy of Auctionator is based on the WotLK 3.3.5 addon, with compatibility and visibility changes for Project Ascension / private-server auction houses.

## Why this fork exists

The original full scan path expects the WotLK auction house to support an instant "get all auctions" request:

```lua
QueryAuctionItems("", nil, nil, 0, 0, 0, 0, 0, 0, true)
```

On Project Ascension this request can fail to return, return no data, or return only partial data. In that situation the original addon can sit on "Scanning" indefinitely and never populate the scan database.

This version keeps the fast scan path, but adds safer fallback behavior and clearer scan status.

## Full scan fallback

If instant scanning does not answer quickly, or returns unusable data, Auctionator now falls back to a page-by-page scan.

The fallback:

- queries auction pages one at a time;
- waits for the normal auction query cooldown;
- processes each returned page into Auctionator's scan database;
- times out cleanly if a page stops responding;
- shows page number, elapsed time, and processed auction count while scanning.

This is slower than instant scanning, but it works on servers where the instant full scan is unreliable or disabled.

## Scan status improvements

The scan window now exposes what is happening instead of showing only "Scanning".

Examples of visible states:

- waiting for instant scan response;
- falling back to page-by-page scan;
- scanning page X/Y;
- waiting for auction house response;
- waiting for auction query cooldown;
- processing database;
- scan complete;
- auction house timed out.

The status line was also smoothed so it does not flicker between "waiting" and "scanning" during page scans.

## Evolution tab

A fourth Auctionator tab, `Evolution`, was added.

It lets you enter an item name and view the saved lowest price for that item over the last 14 days.

Important behavior:

- historical data starts when this version records it;
- older scan history cannot be reconstructed if it was never stored;
- prices are recorded from full scans and normal item scans;
- one daily lowest price is stored per item;
- older evolution data is pruned after 30 days.

The Evolution tab includes:

- current price;
- 14-day low;
- 14-day high;
- percent change versus the oldest available value in the 14-day window;
- a compact two-column 14-day board with relative price bars.

## Saved variables

This version adds:

```toc
AUCTIONATOR_PRICE_EVOLUTION_DATABASE
```

It also fixes the saved-variable declaration around:

```toc
AUCTIONATOR_MEAN_PRICE_DATABASE
AUCTIONATOR_LAST_SCAN_TIME
```

The previous declaration was missing a comma, which could prevent clean persistence of scan-related metadata.

## Locale / load cleanup

The `.toc` previously referenced `Locales\svSE.lua`, but that file was not present in the addon folder. The missing file reference was removed to avoid load warnings or errors.

## Quality filtering

The existing Auctionator minimum-quality setting still applies to database writes.

For example, if the addon is configured to ignore gray-quality items, the fallback scan still receives auction pages from the server, but gray items are not saved into Auctionator's price database or Evolution history.

This does not make page-by-page scanning faster, because the auction house API does not provide a reliable "all auctions except gray items" full-scan filter.

## Performance notes

Page-by-page fallback scanning is inherently slow.

Auction house pages contain roughly 50 auctions. If the auction house has 1,791 pages, the addon must issue around 1,791 serialized page queries. At about one second per page, a complete scan is expected to take around 30 minutes.

The bottleneck is not Lua processing; it is the auction house query throttle and server response behavior.

## Possible future improvements

Useful next features for private-server play:

- tracked-item scan mode for a custom watchlist;
- favorite items in the Evolution tab;
- price alerts;
- pause/resume for long full scans;
- category-specific scans;
- market dashboard for watched materials;
- CSV export;
- bargain detection based on recent median price;
- data quality indicators for stale or sparse price history.

## Validation notes

The modified scan-related Lua files were checked with `luac -p` where possible.

`Auctionator.lua` contains old WoW-era texture escape strings that Lua 5.4 rejects, even though the addon uses them in-game. For that file, syntax was checked through a temporary escaped-string copy so the compiler could parse the modified sections.

Final behavior still needs to be verified in-game because the standalone Lua compiler cannot simulate WoW APIs such as `QueryAuctionItems`, `GetNumAuctionItems`, frames, saved variables, or private-server auction house behavior.
