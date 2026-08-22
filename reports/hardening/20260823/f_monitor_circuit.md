# N1 监控与熔断（硬化 F，2026-08-23）

**性质**：只写文档和现行行为。不是提高收益任务。  
**纸交易仍是 N1。** 未改选股、风险网格、止损、regime。未下单。未改 0.85%。未写 `202609.json`。未读钥匙。未克隆 Mini。  
**新装自动平仓熔断：没有。** 新动作先写在本页「待确认」，不上盘。  
**完成**：2026-08-23（Asia/Shanghai）。

相关：
- 清单：`docs/n1_hardening_checklist.md`
- 红线：`docs/n1_redlines.md`
- 抗压：`reports/hardening/20260823/d4_stress_archive.md`
- 纸面 B：`reports/hardening/20260823/b_report.md`

---

## 一句话

每天一眼看懂五件事（regime / 暴露 / 持仓数 / 当日盈亏 / 是否止损）。告警阈值写死。现行代码门只暂停新开，不自动清仓。新的降风险或停开要等 Friday 确认才上盘。

---

## 现盘锚（写文档时的只读快照）

| 项 | 值 | 来源 |
|---|---|---|
| snapshot | id=417，2026-08-21 15:35 ET | `state.db` snapshots |
| NAV | 77714.650 | 同上 |
| cash | 37850.26 | 同上 |
| 持仓 | AMZN / CRM / JPM / MSFT / PANW / PLTR（6） | 同上 |
| 暴露 | 51.30% = (NAV−cash)/NAV | 计算 |
| regime | active=`bull`，observed=`bull`，pending=null | `strategy_state.market_regime_confirmation` |
| peak NAV | 81535.270 | `strategy_state.paper_peak_nav` |
| 自峰回撤 | 4.69% | (81535.270−77714.650)/81535.270 |
| risk_mode | `normal`（未到 8%） | `paper.py` `_risk_mode` |
| 8/3–8/21 单日最大跌幅 | −0.81%（8/17） | 日尾 snapshot |
| 累计 vs 80000 | −2.86% | Boss 命名基准，无入金流水 |

---

## F1 日报字段（固定输出）

每天（交易日隔夜、上海 08:15 巡检）必须能一眼看到这五行，缺一行不算交：

| 字段 | 口径 | 现行有没有 |
|---|---|---|
| regime | `strategy_state.market_regime_confirmation.active`（bull/neutral/bear）+ 是否 pending | 早报未固定；本页起写入 08:15 模板 |
| 暴露 | (NAV−cash)/NAV；并报最高单票%、最高行业% | 早报未固定 |
| 持仓数 | 开仓只数 + 名单 | 有 |
| 当日盈亏 | 当日最后 snapshot NAV − 上一交易日最后 snapshot NAV（美元和%） | 有（相对前值） |
| 是否触发止损 | 当日 13:35 `reason_for_exit` / `forced_exit` 是否含 hourly_stop 或 time_stop；没有就写「无」 | 有单才报，未固定成每天一行 |

沿用、不新造：
- 08:15「云端日常巡检」：调度 / armed / 信号 / 持仓 / 相对 80000 累计 / 异常
- 01:45「N1 1335 三件」：targets、下单、armed、atr20、meta
- acceptance：preflight 13:25 / post-signal 13:40 / final 14:00 ET
- 缺 ATR：`atr20` 空 → 0 targets（fail-closed）

累计盈亏一律相对 Boss 给定基准 **80000**，不要写「找不到入金」。

---

## F2 告警阈值（写死）

黄=报 Friday（监测例行）。红=报 Friday + Boss。满仓 `max_new=0`、周末、美股休市 **不算**「无法交易」。

| 项 | 黄 | 红 | 依据 |
|---|---|---|---|
| 单日大亏 | 当日 NAV ≤ −1.5% | 当日 NAV ≤ −2.0% | 代码已有 `daily_loss_limit=0.02`，红线对齐这道门；近 15 个交易日最大单日 −0.81%，黄线在现况之上 |
| 暴露异常 | 单票 ≥ 14%；或行业 ≥ 28%；或暴露比对应 regime 上限差 10pp 以内 | 单票 > 15%；或行业 > 30%；或持仓 > 6；或出现 ETF / 非当月 Top50 | 红线单票 15%、行业 30%；D3 上限 6；B3 现况 JPM 14.10%、信息技术 29.15% |
| 连续无法交易 | 连续 **2** 个交易日 | 连续 **3** 个交易日 | 「无法」= atr20 全 None，或 armed 掉，或 signal OAuth/rc=1，或有 targets 无单。不含 0 targets 满仓 |
| 回撤接近隔离 MaxDD 7.71% | 自峰回撤 ≥ **6.0%** | 自峰回撤 ≥ **7.0%** | 隔离 MaxDD 7.71%；D1 3×滑点 8.24%。要赶在代码 8% `reduce_drawdown` 之前被人看见 |
| 事故（无黄，直接红） | — | 有 targets 无单；或 armed 文件消失；或 meta 有仓无 entry_atr/initial_stop | 原事故口径，不变 |

现盘对照：回撤 4.69% < 6.0%，单日未到 −1.5%，持仓 6/6 合法。**现在不应响。**

---

## F3 熔断动作（预先定义）

### 已经在跑（不是新装）

这些是现盘代码/调度，本页只记录，不改参数：

| 条件 | 现行动作 | 证据 |
|---|---|---|
| `atr20` 空或 ≤0 | 该票不进候选；可导致 0 targets | `paper.py` `_strategy_c_candidates`；B5：8/13–18 全 None → 0 开 |
| `N1_CRON_ARMED` 不在 | 当日 cron 全部 no-op | `run_cron_job.sh`；8/12 11:05–11:34 ET |
| 当日亏损 ≤ −2% | `daily_loss_locked`，**不再新开** | `paper.py` 1589–1603；`strategy.toml` `daily_loss_limit=0.02` |
| 自峰回撤 ≥ 8% | `RiskMode.REDUCED`，新开用 `reduced_trade_risk` | `risk.py` `drawdown_mode`；`strategy.toml` `reduce_drawdown=0.08` |
| 自峰回撤 ≥ 12% | `RiskMode.FROZEN`，**不再新开**（`risk_fraction=0`） | 同上 `freeze_drawdown=0.12` |
| 自峰回撤 ≥ 15% | `RiskMode.DISABLED`，**不再新开** | 同上 `disable_drawdown=0.15` |
| SPEC 止损 | 按 2.5×ATR / 3.0×跟踪 / 12 日 / 1h close 平该票 | 红线第 3 条；不在本页改 |

`systemctl` FileNotFound / user bus 假失败 **不是** 熔断，不当链路断。

**已经在跑的门都不自动清掉现仓。** FROZEN/DISABLED/daily_loss_locked 只挡住新开。

### 待确认才上盘（现在不上）

| 触发 | 提议动作 | 状态 |
|---|---|---|
| F2 黄（单日 −1.5%、回撤 6%、连续 2 日无法交易、暴露黄） | **只告警** | 文档已写死；08:15 / 01:45 按表报。不改代码 |
| F2 红（单日 −2%、回撤 7%、连续 3 日无法交易、暴露红） | **只告警** + 等人工 | 不上自动降 0.85%、不自动拆 armed、不自动平仓 |
| 有 targets 无单 | 立刻报 Friday（已在跑） | 不新装 |
| 想「红线后暂停开仓」超出代码已有的 2%/12%/15% | 必须 Friday 点头才能改代码或拆 armed | **停在这里** |
| 想「红线后降风险」改 0.85% 或 risk 网格 | 禁止借本页做；要另开字母 | 红线 |

一句话：出问题先告警。代码里已经会停新开的，维持原样。任何新的自动平仓或改 0.85%，本页写了也不许上盘。

---

## F4 人工介入清单

必须等人（Boss 或 Friday 点头）才能动：

1. 有 targets 却没有 `lb_papertrading` 单（事故）。监测只报，不补单。
2. `N1_CRON_ARMED` 掉了，要不要写回。
3. 自峰回撤触及 7.0%（红）或代码即将进 8% REDUCED，要不要维持 N1 还是只盯。
4. 单日触及 −2% `daily_loss_locked` 之后，要不要次日继续让代码自己解锁。
5. `paper_position_meta` 和现仓对不上（有仓无 entry_atr/initial_stop，或已平票还挂着）。
6. 任何想改 0.85% / 止损倍数 / 选股 / regime / 提前写 `202609.json`。
7. 任何想新装「触发后自动全平」的熔断。
8. universe cron 再出现 `rc=1`（ImportError 已修过，只报复发）。

监测可以自己做、不必等人：
- 08:15 五行日报
- 01:45 三件 + atr20
- 0 targets 满仓：8 月闷着；9 月起仍报 Friday 三样（atr20 / 成交 / meta），不当门
- 缺 ATR fail-closed：记一行，不开仓

---

## 清单 B 实际结论（一并勾）

| 项 | 结论 | 不装完 |
|---|---|---|
| B1 信号时间 | **部分过** | N1 三路齐全仅 7 个交易日，不装成 20 |
| B2 成交 | **部分过** | 无 15m，回测假设价全部未核到 |
| B3 持仓/权重/regime | **过** | |
| B4 止损 | **不过** | 可分类 hourly/time 只有 1 次，不把未分类 forced_exit 算 10 次 |
| B5 失败闭环 | **过** | |

证据：`reports/hardening/20260823/b_report.md` 及 b1–b5。

---

## 第三方怎么核

```
# 现盘锚
sqlite3 var/paper/state.db "select id, created_at, json_extract(payload,'$.nav'), json_extract(payload,'$.cash') from snapshots order by id desc limit 1"
sqlite3 var/paper/state.db "select key, payload from strategy_state where key in ('paper_peak_nav','market_regime_confirmation')"
# 现行门
rg -n "daily_loss_limit|reduce_drawdown|freeze_drawdown|disable_drawdown" config/strategy.toml
rg -n "daily_loss_locked|RiskMode.FROZEN" src/quant_trader/paper.py
# 日报是否五行
# 看 08:15 巡检是否同时有 regime / 暴露 / 持仓数 / 当日盈亏 / 是否止损
```

以上只读。
