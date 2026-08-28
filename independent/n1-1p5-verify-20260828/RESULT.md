# Independent N1 1.5x verify — 2024-01-03 .. 2026-07-31

Identity: `Strategy_N1_70cc50a_risk085`  
Only knobs vs 1.0x: size_mult 1.725 / 1.5 / 1.275; caps 0.225 / 0.45 / 0.225.  
`strong_bull_trade_risk` stays 0.85%. Paper / cron / live N1 not touched.

This is an independent reproduction, **not** an official spec. Do not merge to main until Friday / Boss say so. Do not treat PR #4 sketch CAGR / MaxDD as alignment targets.

Isolation gate 21.82% / 7.71% applies to **1.0x 2024–2025 only**.  
Friday 1.5 gate: **all MaxDD ≤ 15%**. CAGR is report-only.

## Three-way (closed)

| Run | CAGR | MaxDD | End NAV | Trades | complete | risk_settings_error | ≤15% |
| --- | ---: | ---: | ---: | ---: | --- | ---: | --- |
| 1.5 main (rerun) | 34.58% | 12.46% | 214107.74 | 333 | true | 0 | pass |
| 3× slip (30bp / 1bp, delay false) | 27.03% | 12.67% | 184657.19 | 333 | true | 0 | pass |
| delay 1 session (10bp / 1bp, delay true) | 17.32% | 12.00% | 150612.87 | 287 | true | 0 | pass |

Overall three-way MaxDD ≤ 15%: **过**. CAGR is report-only.

Exact floats:
- main CAGR `0.34579010975401303` MaxDD `0.12459177158549584` end `214107.74023770838`
- 3× CAGR `0.2703037172945406` MaxDD `0.12672005586705826` end `184657.19252524248`
- delay1 CAGR `0.17322885908410401` MaxDD `0.11997678750345886` end `150612.87412920824`

## vs 1.0x independent (same window)

1.0x: CAGR 22.96% / MaxDD 10.06% / 313 trades / end 169880.38 / complete=false  
1.5x main: 34.58% / 12.46% / 333 / 214107.74 / complete=true

Peak both 2026-06-22. 1.5 trough/end 2026-07-17 / 07-31.

## Merge fix (why first 1.5 was not official)

First 1.5 main had complete=false, 313 trades, 51 `risk_settings_error` tags (2026-01-02..2026-03-16). Cause: `merge_environments` always `keep_from=2026-01-01`, so 2026-only names lost 2022+ overlay lookback. Runner patch: if isolated primary has no base, load verified isolated/overlay frame; apply `keep_from=2026-01-01` only when a 2024–2025 base exists. Official numbers are the rerun above.

The first broken 1.5 (33.69 / 11.67 / 313 / complete=false) is **NOT official**.

## Knobs

```json
{
  "size_mult_high": 1.725,
  "size_mult_mid": 1.5,
  "size_mult_low": 1.275,
  "single_name_cap": 0.225,
  "sector_cap": 0.45,
  "issuer_cap": 0.225,
  "strong_bull_trade_risk": 0.0085,
  "slippage_bps_main": 10,
  "slippage_bps_3x": 30,
  "fee_bps": 1,
  "signal_delay_main": false
}
```
