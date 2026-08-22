# B3 持仓 / 权重 / regime

只读来源：`state.db` snapshots / strategy_state / `config/strategy_e2/20260803.json` / shadow 13:35 / hourly cache 收盘价。

## 身份与体制

- strategy_version（shadow 42/42）：`Strategy_N1_70cc50a_risk085`
- IDENTITY.json version_label：`Strategy_N1_70cc50a_risk085`
- strong_bull_trade_risk（strategy_e.toml）：`0.0085`
- market_regime_confirmation：`{"active": "bull", "last_observation_date": "2026-08-21", "pending": null, "pending_days": 0}`
- risk_state：`[('normal', 'paper_drawdown_from_peak:81535.270', '2026-08-21T19:35:32.440346+00:00')]`
- 体制字段取值：`active=bull` / `observed=bull`，属于 {bull,neutral,bear}
- 体制锚：config/strategy.toml 含 19 mega + SPY/QQQ（另有 IWM 与行业 ETF 仅作参考，不作持仓）
- 19 mega 清单：AAPL.US, AMZN.US, AVGO.US, BAC.US, CAT.US, COST.US, CVX.US, GE.US, GOOGL.US, HD.US, JPM.US, LLY.US, META.US, MSFT.US, NVDA.US, TSLA.US, UNH.US, WMT.US, XOM.US

## 最新 snapshot

- snapshot id=417 created_at=2026-08-21T19:35:36.018225+00:00 et=2026-08-21T15:35:36.018225-04:00
- NAV=77714.650 cash=37850.26 open_orders=[]

| symbol | qty | cost | daily_close | bar_time_raw | mkt_value | weight_NAV | in_Aug_Top50 | ETF? | sector | industry | days_held | entry_atr | initial_stop | max_r | trailing |
|---|---:|---:|---:|---|---:|---:|---|---|---|---|---:|---:|---:|---:|---|
| PLTR.US | 27 | 177.480 | 173.900 | 2026-08-21T04:00:00+08:00 | 4695.300 | 6.04% | True | False | 信息技术 | 应用软件 | 8 | 8.093189558269126 | 157.2470261043271850 | 0.2451443878480357807376108319 | None |
| PANW.US | 14 | 365.070 | 350.085 | 2026-08-21T04:00:00+08:00 | 4901.190 | 6.31% | True | False | 信息技术 | 系统软件 | 11 | 15.599202481123331 | 326.0719937971916725 | 0.6472125746311208886447538657 | None |
| MSFT.US | 15 | 498.955 | 482.955 | 2026-08-21T04:00:00+08:00 | 7244.325 | 9.32% | True | False | 信息技术 | 系统软件 | 9 | 14.331177417137178 | 463.1270564571570550 | 0.4122759650532798094984271439 | None |
| JPM.US | 31 | 347.410 | 353.445 | 2026-08-21T04:00:00+08:00 | 10956.795 | 14.10% | True | False | 金融 | 多元化银行 | 21 | 7.436454251469598 | 328.8188643713260050 | 0.9899879365956047814527968628 | None |
| CRM.US | 28 | 205.540 | 207.695 | 2026-08-21T04:00:00+08:00 | 5815.460 | 7.48% | True | False | 信息技术 | 应用软件 | 3 | 7.890352789310694 | 185.8141180267232650 | 0.2798353963324993758237735039 | None |
| AMZN.US | 23 | 271.790 | 259.810 | 2026-08-21T04:00:00+08:00 | 5975.630 | 7.69% | True | False | 可选消费 | 零售商 | 11 | 9.450735559728123 | 248.1631611006796925 | 0.3534116449340250609858062658 | None |

- 仓位数：6（D3 上限 6；基础 5）
- 持仓集合：['AMZN.US', 'CRM.US', 'JPM.US', 'MSFT.US', 'PANW.US', 'PLTR.US']
- V 是否仍在仓：否（8/19 已出）

### 行业集中（按 daily close × qty / NAV）

- 信息技术: 22656.275 / NAV 77714.65 = 29.15%
- 金融: 10956.795 / NAV 77714.65 = 14.10%
- 可选消费: 5975.630 / NAV 77714.65 = 7.69%
- 信息技术占比 29.15%（单行业 40% 软帽仅作观察，此处不改参）
- 计价说明：`daily__2026-08-21__*.json` 最后一根 `2026-08-21T04:00:00+08:00`（= 16:00 ET 8/20，或 8/21 盘中未完成日线；JPM 当日量 1.91M 明显低于前几日 6–7M）。 snapshot 权益 NAV-cash=39864.39，六票 daily×qty 合计 39588.70，差 275.69，权重是近似。JPM 14.10% 最接近 15% 单票帽，未超过。
- snapshots 表没有 16:10 ET 行；上表历史日用的是当日最后一条 snapshot（多为 15:35 ET）。shadow_daily_nav.csv 另有 16:10 ET NAV： 8/13 78713.670；8/17 77734.630；8/18 77840.290；8/19 77845.900；8/20 77295.210；8/21 77674.030（8/14 无 16:10 行）。

### 历史日尾盘快照（16:10 ET close-snapshot 优先，否则当日最后一条）

| date_et | source_ts_et | n | symbols | nav | cash | any_ETF | all_in_Top50 |
|---|---|---:|---|---:|---:|---|---|
| 2026-08-13 | 2026-08-13T15:35:35.514619-04:00 | 6 | V.US,PLTR.US,PANW.US,MSFT.US,JPM.US,AMZN.US | 78702.605 | 36219.80 | False | True |
| 2026-08-14 | 2026-08-14T14:35:31.599445-04:00 | 6 | V.US,PLTR.US,PANW.US,MSFT.US,JPM.US,AMZN.US | 78379.615 | 36219.80 | False | True |
| 2026-08-17 | 2026-08-17T15:35:34.197889-04:00 | 6 | V.US,PLTR.US,PANW.US,MSFT.US,JPM.US,AMZN.US | 77745.295 | 36233.20 | False | True |
| 2026-08-18 | 2026-08-18T15:35:34.933637-04:00 | 6 | V.US,PLTR.US,PANW.US,MSFT.US,JPM.US,AMZN.US | 77898.681 | 36233.20 | False | True |
| 2026-08-19 | 2026-08-19T15:35:34.234693-04:00 | 6 | PLTR.US,PANW.US,MSFT.US,JPM.US,CRM.US,AMZN.US | 77893.940 | 37854.58 | False | True |
| 2026-08-20 | 2026-08-20T15:35:37.346978-04:00 | 6 | PLTR.US,PANW.US,MSFT.US,JPM.US,CRM.US,AMZN.US | 77427.756 | 37854.58 | False | True |
| 2026-08-21 | 2026-08-21T15:35:36.018225-04:00 | 6 | PLTR.US,PANW.US,MSFT.US,JPM.US,CRM.US,AMZN.US | 77714.650 | 37850.26 | False | True |

### 不该有的持仓检查

- 未发现：非 Top50 / ETF / V 残留 / 仓位>6 / 单票>15%。
- 观察：shadow 的 `strategy_e2.feature_set.e3_sixth_slot=false`，但实盘维持 6 仓（D3 上限）。8/19 卖 V 后买 CRM，是替换不是第 7 仓。SPEC 允许 D3 到 6，本条不因此判不过。
- 观察：JPM `days_held=21` > 12，但 `max_r≈0.99≥0.5`，代码 time_stop 条件是 `days_held>=12 AND max_r<0.5`，故未出。这是实现多出来的 max_r 门，不是「不该有的标的」。
