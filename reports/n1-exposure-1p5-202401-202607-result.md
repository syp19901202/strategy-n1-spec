# N1 exposure backtest — round 3

- Identity: `Strategy_N1_70cc50a_risk085`
- SCALE_FACTOR: **1.5** (Boss freeze 2026-08-28)
- Strong-bull trade_risk: **0.85% (0.0085) UNCHANGED**; E2 toml left at 0.65%
- Engine: n-lift `simulate_variant_history` @ `/workspace/quant/migrate/quant-trader-n-lift`
- Parent commit pin: `70cc50ad965234b99a5a136d4e4f283eedd4281f`
- Requested window: 2024-01-01 .. 2026-07-31 inclusive
- Engine sessions: **2024-01-03 .. 2026-07-31** (frozen isolation first session is 2024-01-03; 2024-01-01 NYSE holiday)
- Fill: first 15m open after 13:35 ET (typically 13:45) × (1+10bp), commission 1bp, same_day
- Stops: initial 2.5× entry ATR, trail 3.0×, 12-day time stop, 1h close trigger
- Selection: monthly PIT stage_250 top 50 common stock; C-layer top 55% then E1B; max 5 + D3 sixth
- Status: **complete**
- Started: 2026-08-28T11:39:29.442766+08:00
- Finished: 2026-08-28T12:45:03.058664+08:00
- Runtime seconds: 3933.6159016969978
- n1_paper_modified: False
- isolated-2024-2025 written: False

## 1. Exposure knobs changed

Every name scaled 等比 by SCALE_FACTOR from current N1 `size_mult`. Caps that would clip large names first are scaled by the same factor. `strong_bull_trade_risk` is not an exposure knob this round.

| Knob | Old | New |
| --- | ---: | ---: |
| `size_mult_high` | 1.15 | 1.725 |
| `size_mult_mid` | 1.0 | 1.5 |
| `size_mult_low` | 0.85 | 1.275 |
| `single_name_cap` | 0.15 | 0.225 |
| `sector_cap` | 0.3 | 0.45 |
| `issuer_cap` | 0.15 | 0.225 |

- Unchanged: `strong_bull_trade_risk` = 0.0085 (0.85%)
- Unchanged: stops, schedule, universe, earnings, fill, slippage, commission, max_new_positions, sixth_slot, regime caps

## 2. Full window 2024-01-01 .. 2026-07-31

| Metric | Value |
| --- | ---: |
| Start NAV | 100000.0000000000 |
| End NAV | 214107.7402377084 |
| Total return | 1.1410774023770878 (114.1077%) |
| CAGR | 0.3457901097540130 (34.58%) |
| MaxDD | 0.1245917715854958 (12.46%) |
| Trade count | 333 |
| Daily rows | 646 |
| First session | 2024-01-03 |
| Last session | 2026-07-31 |
| Complete | True |
| Exposure | 0.6455889487957572 |
| Calmar | 2.775384805542549 |
| Profit factor | 1.9038970573272105 |

CAGR uses engine convention: `(1+total_return)^(252/n_daily_returns)-1`.
MaxDD is magnitude (positive). Start NAV is initial cash 100000 before the first daily return.

## 3. MaxDD 15% accept (this round)

This round's drawdown accept door is **15% MaxDD**. The isolation 7.71% gate is **not** the accept door. CAGR gate is not lowered.

- Full-window MaxDD: 0.1245917715854958 (12.46%)
- Accept threshold: 15.00% (15%)
- MaxDD passed 15% accept: **True**
- CAGR vs frozen-spec gate 21.82% (gate unchanged): run 34.58% ; delta 0.1275901097687524

## 4. Versus gate 21.82% / 7.71% (gate numbers unchanged; 7.71% is not accept this round)

The published CAGR gate is the **2024-01-03 .. 2025-12-31** isolation window value 21.82%. This run **extends** through 2026-07-31. The gate numbers are not modified. Isolation MaxDD 7.71% is reported for comparison only.

| Metric | Gate (2024-2025) | This run (2024-01-03 .. 2026-07-31) | Delta (run − gate) |
| --- | ---: | ---: | ---: |
| CAGR | 0.2181999999852606 (21.82%) | 0.3457901097540130 (34.58%) | 0.1275901097687524 |
| MaxDD | 0.0770780856655126 (7.71%) | 0.1245917715854958 (12.46%) | 0.0475136859199833 |
| 2024 return | 0.2747886912126327 (27.48%) | 0.3297413024100875 (32.97%) | 0.0549526111974548 |
| 2025 return | 0.1613911938486754 (16.14%) | 0.2611410626249568 (26.11%) | 0.0997498687762814 |
| Trade count | 245 | 333 | 88 |

- 2024 slice rounded match vs gate 27.48%: **False**
- 2025 slice rounded match vs gate 16.14%: **False**
- 2024/2025 exact float match: 2024=False, 2025=False

## 5. Year slices

Year return = product of that year's daily returns (same as engine `yearly_returns`). Slice MaxDD walks NAV from the prior session close (2024 uses initial 100000).

| Slice | First | Last | Days | Return | MaxDD | Peak | Trough | End NAV |
| --- | --- | --- | ---: | ---: | ---: | --- | --- | ---: |
| 2024 | 2024-01-03 | 2024-12-31 | 251 | 0.3297413024100875 (32.9741%) | 0.1029212733946371 (10.2921%) | 2024-07-10 | 2024-08-07 | 132974.130241 |
| 2025 | 2025-01-02 | 2025-12-31 | 250 | 0.2611410626249568 (26.1141%) | 0.0815473632382828 (8.1547%) | 2025-02-14 | 2025-04-07 | 167699.135914 |
| 2026-01-01 .. 2026-07-31 | 2026-01-02 | 2026-07-31 | 145 | 0.2767372894979943 (27.6737%) | 0.1245917715854955 (12.4592%) | 2026-06-22 | 2026-07-17 | 214107.740238 |

## 6. Drawdown segments (depth ≥ 1%)

Peak-to-trough on full-window daily NAV. Recovery is the first later session that retakes the peak. Sorted deepest first.

| Peak | Peak NAV | Trough | Trough NAV | Depth | Recovery | Open |
| --- | ---: | --- | ---: | ---: | --- | --- |
| 2026-06-22 | 244580.452054 | 2026-07-17 | 214107.740238 | -0.1245917715854955 (12.4592%) | still open | True |
| 2024-07-10 | 125894.682149 | 2024-08-07 | 112937.441149 | -0.1029212733946371 (10.2921%) | 2024-11-06 | False |
| 2025-02-14 | 147588.256156 | 2025-04-07 | 135552.823021 | -0.0815473632382828 (8.1547%) | 2025-07-03 | False |
| 2024-12-24 | 140107.612286 | 2025-01-14 | 129956.691495 | -0.0724508870378424 (7.2451%) | 2025-02-04 | False |
| 2026-05-28 | 231659.747916 | 2026-06-10 | 219175.233335 | -0.0538916004742542 (5.3892%) | 2026-06-15 | False |
| 2025-08-12 | 152943.460199 | 2025-08-21 | 145180.850769 | -0.0507547653259766 (5.0755%) | 2025-09-09 | False |
| 2026-01-28 | 185221.963374 | 2026-03-06 | 176733.296218 | -0.0458297007611940 (4.5830%) | 2026-04-24 | False |
| 2024-04-11 | 117845.706232 | 2024-05-01 | 112821.788828 | -0.0426313148333264 (4.2631%) | 2024-05-21 | False |
| 2025-11-03 | 171228.995075 | 2025-11-20 | 164026.796104 | -0.0420617954755926 (4.2062%) | 2025-11-25 | False |
| 2024-05-21 | 117944.013661 | 2024-05-30 | 113324.894860 | -0.0391636561916476 (3.9164%) | 2024-06-12 | False |
| 2024-12-16 | 139597.406457 | 2024-12-18 | 134237.998380 | -0.0383918885977682 (3.8392%) | 2024-12-24 | False |
| 2025-11-25 | 171862.226592 | 2025-12-17 | 165435.358292 | -0.0373954674425064 (3.7395%) | 2026-01-06 | False |
| 2025-10-06 | 164886.333780 | 2025-10-10 | 158959.366571 | -0.0359457759369740 (3.5946%) | 2025-10-20 | False |
| 2026-04-24 | 188068.347271 | 2026-04-28 | 182254.106699 | -0.0309155722163615 (3.0916%) | 2026-04-30 | False |
| 2024-11-11 | 131400.484654 | 2024-11-18 | 127662.520689 | -0.0284471094216578 (2.8447%) | 2024-11-22 | False |
| 2026-05-14 | 227056.290534 | 2026-05-19 | 220887.387374 | -0.0271690475757446 (2.7169%) | 2026-05-26 | False |
| 2024-02-16 | 109505.631306 | 2024-02-21 | 106602.115132 | -0.0265147658635666 (2.6515%) | 2024-02-22 | False |
| 2024-12-06 | 136592.557080 | 2024-12-10 | 133342.170152 | -0.0237962228500237 (2.3796%) | 2024-12-13 | False |
| 2026-05-06 | 207472.675082 | 2026-05-07 | 202759.857936 | -0.0227153630908948 (2.2715%) | 2026-05-08 | False |
| 2024-03-12 | 116649.590199 | 2024-03-15 | 114130.671258 | -0.0215938944691785 (2.1594%) | 2024-04-03 | False |
| 2024-01-19 | 105376.776436 | 2024-01-31 | 103159.780394 | -0.0210387536683991 (2.1039%) | 2024-02-02 | False |
| 2025-07-30 | 151518.663672 | 2025-08-01 | 148665.452112 | -0.0188307597941899 (1.8831%) | 2025-08-04 | False |
| 2025-10-29 | 171064.923153 | 2025-10-30 | 168600.092153 | -0.0144087458408362 (1.4409%) | 2025-11-03 | False |
| 2026-01-22 | 182147.133242 | 2026-01-26 | 179595.433100 | -0.0140090052279566 (1.4009%) | 2026-01-28 | False |
| 2024-06-27 | 121306.120980 | 2024-06-28 | 119640.732725 | -0.0137288064379153 (1.3729%) | 2024-07-03 | False |
| 2024-11-26 | 132560.875377 | 2024-11-27 | 130752.435377 | -0.0136423359823444 (1.3642%) | 2024-12-04 | False |
| 2025-07-09 | 149089.457875 | 2025-07-15 | 147132.768574 | -0.0131242633005230 (1.3124%) | 2025-07-18 | False |
| 2024-04-03 | 117048.891231 | 2024-04-04 | 115591.897047 | -0.0124477401531251 (1.2448%) | 2024-04-05 | False |
| 2024-02-02 | 106439.898881 | 2024-02-06 | 105163.901733 | -0.0119879590394417 (1.1988%) | 2024-02-07 | False |
| 2025-08-04 | 151614.257112 | 2025-08-05 | 149819.395553 | -0.0118383428645958 (1.1838%) | 2025-08-11 | False |
| 2026-06-15 | 237448.793335 | 2026-06-16 | 234674.383335 | -0.0116842455210151 (1.1684%) | 2026-06-18 | False |
| 2025-07-18 | 149157.849687 | 2025-07-22 | 147438.163504 | -0.0115293039352938 (1.1529%) | 2025-07-24 | False |
| 2025-09-15 | 156966.601282 | 2025-09-23 | 155205.123296 | -0.0112219922651209 (1.1222%) | 2025-09-25 | False |
| 2024-03-08 | 116060.557759 | 2024-03-11 | 114770.675199 | -0.0111138752468901 (1.1114%) | 2024-03-12 | False |
| 2024-06-18 | 120704.890219 | 2024-06-21 | 119376.968683 | -0.0110013896971217 (1.1001%) | 2024-06-26 | False |
| 2024-02-09 | 108695.227975 | 2024-02-13 | 107565.017296 | -0.0103979788284340 (1.0398%) | 2024-02-14 | False |

- Deepest segment depth: -0.1245917715854955

## 7. Data used

### Cache roots (read-only)

- Isolated 2024-2025 monthly: `/workspace/quant/isolated-2024-2025/evidence/monthly_universe_diagnostic` sha `cb11cbcf067857e8d2fedacb6ce9ed7cb8df4a6eee00f84a3d8d5b69702e5afd`
- 2026 holdout monthly: `/workspace/quant/migrate/evidence/strategy_e2_20260803/frozen_e2_before_2026_holdout/monthly_universe` sha `53330a409f5e343031eecb0479fc0292d2d9927f0a7ee0d8f232615f4b083707`
- 2024-2025 reference: `/workspace/quant/isolated-2024-2025/evidence/selection_2024_2025/reference_evidence.json` sha `f102cebf7f3f3c902ea970a0cbc7648db8ca76334c4848c6311f73131d71fd1a`
- 2026 reference: `/workspace/quant/diagnostic-2026-jan-jul/evidence/reference_evidence.json` sha `305d6d918f7f709cb73bdac9bed91c0bba9dba6997932374775bd711e3d6ce61`
- Monthly plan override (FI.US→MRK.US 2025-06): `/workspace/quant/isolated-2024-2025/evidence/selection_2024_2025/monthly_plan_override.json` sha `0ef946e11bfee87332de091975bc33abf2436b794b7c6c61b2b971e5fb7bfcee`
- Isolation overlay/primary: `/workspace/quant/isolated-2024-2025/history/candidate_c_overlay`, `/workspace/quant/isolated-2024-2025/history/candidate_c_primary`
- 2026 hold overlay/primary: `/workspace/quant/diagnostic-2026-jan-jul/history/candidate_c_overlay`, `/workspace/quant/diagnostic-2026-jan-jul/history/candidate_c_primary`
- July 11-31 fill tree (always concatenated): `/workspace/quant/diagnostic-2026-jan-jul/history/july_2026_07_11_31`
- migrate/history overlay/primary (2026-hold miss fallback): `/workspace/quant/migrate/history/candidate_c_overlay`, `/workspace/quant/migrate/history/candidate_c_primary`
- Macro calendar: `/workspace/quant/isolated-2024-2025/data/macro_calendar/us_macro_events.csv` ∪ `/workspace/quant/diagnostic-2026-jan-jul/data/macro_calendar/us_macro_events.csv`
- Archived C3 marker: `/workspace/quant/migrate/checkpoints/candidate_c_matrix_20260714/9d5b408369df5b6b9b78b8f5ef820278bff85a61789f76a7c33c3ddd7ba23fcf/matrix.complete.json`

Isolation window frozen resolver: `--primary-cache-root` isolated `candidate_c_primary`, `--overlay-cache-root` isolated `candidate_c_overlay`.
2026 hold window frozen resolver: `--primary-cache-root` diagnostic `candidate_c_primary`, `--overlay-cache-root` diagnostic `candidate_c_overlay`.
If a ticker is missing from the 2026 hold roots, fallback is diagnostic primary → diagnostic overlay → migrate primary → migrate overlay.
July 11-31 bars are **always** concatenated from `july_2026_07_11_31` (july wins on overlap). This run does **not** skip concat when overlay already reaches 2026-07-31.
Earnings: isolated + diagnostic `reference_evidence.json` `earnings_blackouts` merged as-is. The 13 isolation-tree pairs after 2026-07-11 were kept. Paper `earnings_updates` were not read.

- Plan months: 31 (2024-01 .. 2026-07)
- Plan symbols (Top50 union after override): 118
- Daily frames: 131
- 1h frames: 131
- 15m frames: 131
- Symbols with 15m: 131
- Symbols missing 15m: []
- 15m existed for isolation fills: **True**
- Frame max NY: 2026-07-31
- Frame min NY: 2022-10-31
- Bars after 2026-07-31 (trimmed off / counted): 0
- Bars after 2026-07-10 (July 11-31 coverage): 44880
- July concat frames: 264
- Kept isolation earnings pairs after 2026-07-11: [{'symbol': 'ELV.US', 'event_date': '2026-07-14'}, {'symbol': 'COF.US', 'event_date': '2026-07-20'}, {'symbol': 'SCHW.US', 'event_date': '2026-07-20'}, {'symbol': 'CMCSA.US', 'event_date': '2026-07-22'}, {'symbol': 'FCX.US', 'event_date': '2026-07-22'}, {'symbol': 'HON.US', 'event_date': '2026-07-22'}, {'symbol': 'VZ.US', 'event_date': '2026-07-23'}, {'symbol': 'CMG.US', 'event_date': '2026-07-28'}, {'symbol': 'SBUX.US', 'event_date': '2026-07-28'}, {'symbol': 'BMY.US', 'event_date': '2026-07-29'}, {'symbol': 'FSLR.US', 'event_date': '2026-07-29'}, {'symbol': 'KKR.US', 'event_date': '2026-07-29'}, {'symbol': 'ETN.US', 'event_date': '2026-07-30'}]
- SPY day holes in window: []
- Cache source counts: {"iso_primary+iso_overlay+diag_primary+diag_overlay+july_data_sha256": 264, "iso_primary+iso_overlay+mig_overlay": 52, "iso_primary+iso_overlay+mig_primary": 77}
- Paper market_cache 15m files: 0 (n1-paper has daily/hourly session files only; not used as isolation 15m)
- Missing dates/tickers (day vs SPY, hole_count>0): [{'symbol': 'ABBV.US', 'hole_count': 15}, {'symbol': 'ABNB.US', 'hole_count': 15}, {'symbol': 'ACN.US', 'hole_count': 15}, {'symbol': 'AMGN.US', 'hole_count': 15}, {'symbol': 'APO.US', 'hole_count': 15}, {'symbol': 'BMY.US', 'hole_count': 15}, {'symbol': 'CEG.US', 'hole_count': 15}, {'symbol': 'CMCSA.US', 'hole_count': 15}, {'symbol': 'CMG.US', 'hole_count': 15}, {'symbol': 'CMI.US', 'hole_count': 15}, {'symbol': 'COF.US', 'hole_count': 15}, {'symbol': 'CVS.US', 'hole_count': 15}, {'symbol': 'DASH.US', 'hole_count': 15}, {'symbol': 'DDOG.US', 'hole_count': 15}, {'symbol': 'DIS.US', 'hole_count': 15}, {'symbol': 'ELV.US', 'hole_count': 15}, {'symbol': 'ETN.US', 'hole_count': 15}, {'symbol': 'F.US', 'hole_count': 15}, {'symbol': 'FCX.US', 'hole_count': 15}, {'symbol': 'FDX.US', 'hole_count': 15}, {'symbol': 'FSLR.US', 'hole_count': 15}, {'symbol': 'GEV.US', 'hole_count': 61}, {'symbol': 'HON.US', 'hole_count': 16}, {'symbol': 'HUM.US', 'hole_count': 15}, {'symbol': 'ISRG.US', 'hole_count': 15}, {'symbol': 'KKR.US', 'hole_count': 15}, {'symbol': 'KO.US', 'hole_count': 15}, {'symbol': 'LIN.US', 'hole_count': 15}, {'symbol': 'LULU.US', 'hole_count': 15}, {'symbol': 'MCD.US', 'hole_count': 15}, {'symbol': 'PEP.US', 'hole_count': 15}, {'symbol': 'PFE.US', 'hole_count': 15}, {'symbol': 'PYPL.US', 'hole_count': 15}, {'symbol': 'SBUX.US', 'hole_count': 15}, {'symbol': 'SCHW.US', 'hole_count': 15}, {'symbol': 'SNDK.US', 'hole_count': 279}, {'symbol': 'SNPS.US', 'hole_count': 15}, {'symbol': 'T.US', 'hole_count': 15}, {'symbol': 'TGT.US', 'hole_count': 15}, {'symbol': 'TMUS.US', 'hole_count': 15}, {'symbol': 'TTD.US', 'hole_count': 15}, {'symbol': 'VST.US', 'hole_count': 15}, {'symbol': 'WDAY.US', 'hole_count': 15}, {'symbol': 'WFC.US', 'hole_count': 15}, {'symbol': 'XYZ.US', 'hole_count': 15}]

No Longbridge login or live fetch was performed in this run. Existing OAuth/caches only.

## 8. Exact commands and output paths

Third-party rerun (does not touch paper/cron/isolated tree writes):

```bash
/workspace/quant-venv/bin/python \
  /workspace/n1-exposure-20260731-r3/out/run_n1_exposure_20240101_20260731.py
```

Working directory is set by the script to `/workspace/quant/migrate/quant-trader-n-lift`.

Outputs:

- Report: `/workspace/n1-exposure-20260731-r3/out/result.md`
- Metrics JSON: `/workspace/n1-exposure-20260731-r3/out/artifacts/n1_metrics.json`
- Daily NAV: `/workspace/n1-exposure-20260731-r3/out/artifacts/n1_nav.csv`
- Drawdown segments: `/workspace/n1-exposure-20260731-r3/out/artifacts/drawdown_segments.json`
- Data inventory: `/workspace/n1-exposure-20260731-r3/out/artifacts/data_inventory.json`
- Cache map: `/workspace/n1-exposure-20260731-r3/out/artifacts/cache_used.json`
- Log: `/workspace/n1-exposure-20260731-r3/out/logs/run.log`
- Trades: `/workspace/n1-exposure-20260731-r3/out/artifacts/trades.csv`
- Runner: `/workspace/n1-exposure-20260731-r3/out/run_n1_exposure_20240101_20260731.py`

Paper guards (read-only stat, never opened for write):

- state.db before: `{'exists': True, 'path': '/home/box/quant-trader-n1-paper/var/paper/state.db', 'size': 516096, 'mtime_ns': 1787859334193762613, 'mtime_utc': '2026-08-27T19:35:34.193763+00:00', 'mode': '-rw-r--r--'}`
- state.db after: `{'exists': True, 'path': '/home/box/quant-trader-n1-paper/var/paper/state.db', 'size': 516096, 'mtime_ns': 1787859334193762613, 'mtime_utc': '2026-08-27T19:35:34.193763+00:00', 'mode': '-rw-r--r--'}`
- N1_CRON_ARMED before: `{'exists': True, 'path': '/home/box/quant-trader-n1-paper/var/paper/N1_CRON_ARMED', 'size': 0, 'mtime_ns': 1787840819152338746, 'mtime_utc': '2026-08-27T14:26:59.152339+00:00', 'mode': '-rw-r--r--'}`
- N1_CRON_ARMED after: `{'exists': True, 'path': '/home/box/quant-trader-n1-paper/var/paper/N1_CRON_ARMED', 'size': 0, 'mtime_ns': 1787840819152338746, 'mtime_utc': '2026-08-27T14:26:59.152339+00:00', 'mode': '-rw-r--r--'}`
