# data/

This folder holds the frozen market-data snapshot used by the notebook.

| File | Created by | Purpose |
|---|---|---|
| `market_data.csv` | first live notebook run | Raw Yahoo Finance series used by the analysis |
| `snapshot_info.json` | first live notebook run | Download date and S&P 500 basis actually used |

After the first successful live run, **commit both files** to GitHub.

When `market_data.csv` exists, the notebook uses it and does not contact Yahoo Finance. This makes subsequent runs reproducible and allows the analysis to run offline.

Delete the snapshot files only when you intentionally want to refresh the data.

Source: Yahoo Finance via `yfinance` (an unofficial endpoint; values can occasionally be revised or wrong).
