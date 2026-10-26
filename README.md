# FidMAS Bloomberg pull plan

## Purpose

This repository specifies the Bloomberg data pull for the FidMAS evaluation. The data are licensed from Bloomberg and **not included**; this repository contains only identifiers, field mnemonics and request settings. Repeating the pull requires a Bloomberg Terminal.

## Universe and period

- 22 US equities (`config/universe.yaml`) and 16 macro and market-context series (`config/macro_series.yaml`).
- Pulled: 2010-01-01 to 2025-12-31.
- The evaluation uses 2012-01-03 to 2025-09-30. Observations from before 2012 only supply the values that are current at the start.

## Fields and series

- `config/equity_fields.yaml` lists the 22 equity fields (15 daily, 7 quarterly) with category, periodicity, availability rule, and whether the evaluation uses each one.
- `config/macro_series.yaml` lists each macro series with its fields, cadence, publication lag and role. The evaluation reads PX_LAST of every macro series. FEDL01 Index also serves as the risk-free rate.

## Pull method

The data were pulled with Excel `BDH` formulas through the Bloomberg Excel add-in, so a Bloomberg Terminal is required. `pull/excel_bdh_formulas.csv` specifies one formula per security × field (530 pairs) with the following BDH options:

- **Start and end dates:** 2010-01-01 and 2025-12-31.
- **Periodicity:** daily, except `Per=Q` for fundamentals and `Per=M` for USGG3M and USGG10YR.
- **Other options:** `Days=A`, `Fill=B`, `FX=USD` and `QuoteType=P` on every formula. `CDR=US` is set on the daily equity and daily macro requests.

Each formula is specified with its own date column (`Dts=S`). The formulas use English Excel syntax; a German-locale Excel needs `;` as the argument separator.

## Output format and availability rules

The sheets are converted to long format, one row per observation that has a value: `date;ticker;field;value;freq;available_from`. The files use a semicolon separator, decimal comma and `DD.MM.YYYY` dates.

`available_from` is the first date an observation may be used. All forward fills join on `available_from`, never on `date`.

| Data | Rule |
|---|---|
| Prices and other daily fields | `available_from = date` |
| Quarterly fundamentals | period end + 45 calendar days |
| Short interest (SHORT_INT) | `date` + 10 calendar days |
| CPI YOY Index | reference month end + 20 calendar days |
| FEDL01, CESIUSD, BFCIUS | `date` + 1 business day |
| Other macro series | `available_from = date` |

These lags are conservative proxies, not reported announcement dates.

## License

MIT (see `LICENSE`). The license covers this repository only, not Bloomberg data.
