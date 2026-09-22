# Koochin TSETMC Daily Report

Generates the client's "daily report" Excel (RTL, YekanWeb, autofilter) for one
symbol: 30-second trade windows above the value threshold, with buyer power
(قدرت خریدار), in the format of the client's sample 14050602.xlsx.

    زمان | نماد | درصد | مالکیت | تعداد | ارزش کل | ارزش متوسط | قدرت خریدار | نوع معامله

## Desktop app (BourseFilter.exe)

`main.py` / `dist/BourseFilter.exe` serves a Persian RTL dashboard
(`app/index.html`) and opens it in the default browser:

- Symbol autocomplete (TSETMC search) + scope: latest session / one session /
  last N sessions.
- Smart filters (the client's rules, thresholds editable): power > 0.85,
  power growth ≥ 0.03 vs previous, individual buy − sell ≥ N (toman/rial),
  درصد < 1.
- Column filters for every field incl. date range and time range; sortable
  columns; sessions summary table; live 30-second polling with per-poll
  power delta; Excel export of the current view to Downloads.

Run from source: `py main.py` (deps: openpyxl).
Build exe: `py -3.12 -m PyInstaller --noconfirm --onefile --windowed --name BourseFilter --add-data "app;app" main.py`

## CLI usage

Requires Python 3.12+ and openpyxl (`pip install openpyxl`).

Latest completed session (auto-detected from the exchange clock):

    py koochin_report.py --symbol کوچین --out koochin.xlsx

Specific session (Gregorian YYYYMMDD):

    py koochin_report.py --symbol کوچین --date 20260824 --out koochin.xlsx

Live during a session, refreshed every 30 seconds:

    py koochin_report.py --symbol کوچین --watch 30 --out koochin_live.xlsx

History - one sheet covering the last N trading sessions (each row gets a
Jalali تاریخ column, e.g. 14050602, so the sheet filters by date too):

    py koochin_report.py --symbol کوچین --history 60 --out koochin_history.xlsx

Extract a symbol from an existing sample workbook (offline):

    py koochin_report.py --from-sample 14050602.xlsx --symbol کوچین --out koochin.xlsx

## How rows are built

- The session trade tape is grouped into 30-second windows; a window becomes a
  row when its total trade value reaches `--min-toman` (default 60 million
  toman). `تعداد` = trades in the window, `ارزش کل` / `ارزش متوسط` = window
  total / per-trade average in **million rials**.
- `درصد` = window's last price vs the previous session close.
- `قدرت خریدار` = individual (حقیقی) buy value / individual sell value for the
  session (TSETMC client-type data). During `--watch` polling this reflects the
  cumulative values at each poll, so it evolves like the client's sample;
  historical intraday client-type series is not published by TSETMC, so for
  past days it is the final session ratio.
- `نوع معامله` uses a tick rule (the public tape does not tag the trade side):
  downtick -> فروش, otherwise خرید; a repeated خرید within 120 seconds of the
  previous one becomes خرید مجدد.

## Data source

`https://cdn.tsetmc.com` JSON API (the current TSETMC frontend):
`GetInstrumentSearch`, `GetClosingPriceDailyList`, `GetTradeHistory/{code}/{date}/false`,
`GetClientTypeHistory/{code}/{date}`, `StaticData/GetTime`. Requires a network
that can reach tsetmc.com (Iran IP or proxy). Verified against the raw tape for
Koochin on 2026-08-24 (window 12:28:30-12:28:59: 105 trades, 16533.48M rials,
last price 15150 -> -2.63%).
