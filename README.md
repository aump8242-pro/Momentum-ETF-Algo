# ETF Momentum Algo

A Python backtester and paper-trading dashboard for a single INR portfolio of 51 Indian ETFs and 29 US-listed ETFs. The strategy ranks funds monthly and holds up to six positions.

[Setup](#setup) · [Run the backtest](#run-the-backtest) · [Quant Momentum Fund comparison](#quant-momentum-fund-comparison) · [Paper dashboard](#paper-dashboard)

## Strategy

- Rank ETFs by 1-, 3-, 6-, and 12-month price momentum with weights of 15%, 40%, 30%, and 15%. The 1-month signal skips the five most recent trading sessions.
- Rotate out of holdings that fall outside the top six. New entries must clear their 200-day moving average. The strategy uses a 15% hard stop and an 18% trailing stop.
- Use prior-session signals on the NSE calendar. The historical replay uses each ETF's market open as a fill proxy and converts US prices using the latest earlier daily USD/INR close.

## Setup

Use Python 3.12. From the repository root, create a virtual environment and install the dependencies in PowerShell.

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

The replay runs from saved data without API credentials. For a fresh download or connected paper dashboard, create a `../.env` file **one folder above the repository** with the credentials you need.

```dotenv
UPSTOX_ACCESS_TOKEN=your_upstox_access_token
APCA_API_KEY_ID=your_alpaca_key_id
APCA_API_SECRET_KEY=your_alpaca_secret_key
```

Keep the real `.env` out of Git. The downloader also accepts these variables from the shell. The paper dashboard reads the Upstox token from `../.env` and uses Yahoo Finance for US data and FX.

## Run the backtest

The supplied five-year replay is fixed to **23 Nov 2021–23 Nov 2026**. US bars end on **20 Nov 2026**, the latest completed US session used in this run.

1. If the daily CSVs are already present under `Backtest_2021-11-23_to_2026-11-23/bars/`, go to step 2. To fetch them again, run the downloader. It uses the instrument master at `data/instruments_NSE.json.gz`, prefers Upstox for Indian ETFs and Alpaca SIP for US ETFs, and uses Yahoo Finance for eligible gaps and USD/INR.

   ```powershell
   .\.venv\Scripts\python.exe download_five_year.py
   ```

   If no QQQ bars are available from Alpaca, the downloader requires the sibling archive `../QQQ_Alpaca_Backtest_2020_2026-11-17/data/` with `qqq_daily.csv` and its matching `manifest.json`. A fresh download without that archive requires working Alpaca access for QQQ. The published run used the archive for QQQ, Yahoo Finance for the remaining US ETF data, and Upstox with Yahoo Finance gap fills for Indian ETFs.

2. Replay the strategy and generate the report.

   ```powershell
   .\.venv\Scripts\python.exe run_five_year.py
   ```

3. Open [report.html](Backtest_2021-11-23_to_2026-11-23/report.html) in a local browser for the interactive charts. The same folder contains the [trade log](Backtest_2021-11-23_to_2026-11-23/trades.csv), [daily equity](Backtest_2021-11-23_to_2026-11-23/equity.csv), and [data coverage](Backtest_2021-11-23_to_2026-11-23/data_coverage.csv).

The five-year run grew **₹10 lakh to ₹20.91 lakh**, a **109.08% total return** and **15.90% CAGR**, with a **22.88% maximum drawdown**. It recorded 268 trades and ₹1,948.93 in estimated statutory and regulatory charges. Brokerage and slippage are set to zero. The replay uses the current ETF universe and price returns, which introduces selection bias and excludes cash dividends.

## Quant Momentum Fund comparison

The report compares the ETF strategy with **Quant Momentum Fund – Direct Plan, Growth** from **21 Nov 2023**, the fund's first published NAV, through **20 Nov 2026**. It rebases both portfolios to ₹10 lakh on the first of **739 shared valuation dates** and shows their equity curves, drawdowns, and year-wise returns. Fund returns reflect expenses through the published NAV. Strategy returns come from the historical replay. [AMFI NAV history, scheme 152189](https://www.amfiindia.com/net-asset-value/nav-download)

| Matched-period metric | ETF Momentum | Quant Momentum Direct Growth |
| --- | ---: | ---: |
| Final value from ₹10 lakh | ₹20.73 lakh | ₹15.59 lakh |
| Total return | 107.34% | 55.90% |
| CAGR | 27.54% | 15.96% |
| Maximum drawdown | −22.88% | −22.67% |
| Annualized volatility | 21.90% | 16.97% |

| Calendar year | ETF Momentum | Quant Momentum Direct Growth |
| --- | ---: | ---: |
| 2023, from 21 Nov | 12.29% | 12.83% |
| 2024 | 11.36% | 26.44% |
| 2025 | 60.97% | 3.56% |
| 2026, through 20 Nov | 3.00% | 5.52% |

Preserve [quant_momentum_direct_growth_nav.csv](Backtest_2021-11-23_to_2026-11-23/quant_momentum_direct_growth_nav.csv) in the output folder when regenerating the report. The report generator reads it to build the comparison.

## Paper dashboard

```powershell
.\.venv\Scripts\python.exe etf_fronttest/server.py --port 8767
```

Open [http://127.0.0.1:8767](http://127.0.0.1:8767). The dashboard simulates all orders locally. Add `--offline` to view cached history with paper execution disabled. See the [dashboard documentation](etf_fronttest/README.md) for market hours, FX handling, and the paper ledger.

Run the tests from the repository root.

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```
