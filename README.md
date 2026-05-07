# Auctionator 3.3.5 - Project Ascension Compatibility Fork

This is a WotLK 3.3.5-compatible Auctionator copy adjusted for Project Ascension / private-server auction house behavior.

The main changes are:

- safer full-scan fallback when instant full scan does not work;
- clearer scan progress and timeout messages;
- smoother page-scan status display;
- a new `Evolution` tab for 14-day item price history;
- daily price-history persistence for scanned items;
- saved-variable and `.toc` cleanup.

For implementation details and rationale, see:

- `PROJECT_ASCENSION_CHANGES.md`

## Notes

Evolution history starts from scans made with this version. It cannot reconstruct older daily history that was never stored.

Page-by-page full scanning can be slow on large auction houses because the WoW auction API serializes page requests.
