# N1 加固清单 B — 纸面执行一致性（只读）

- 核查日：2026-08-23（Asia/Shanghai）
- 根：`/home/box/quant-trader-n1-paper`
- 身份：`Strategy_N1_70cc50a_risk085`（IDENTITY.json + shadow 42/42）
- 约束：未改纸盘文件、未下单、未碰 N1_CRON_ARMED、未改 0.85%、未写 202609.json、未读密钥/token/env、未 clone。
- 输出：`/workspace/n1-hardening-b-20260823/`

## 总表

| 项 | 结论 | 一句话 |
|---|---|---|
| B1 信号时间 | **部分过** | N1 三路齐全仅 7 个交易日（要求抽样 20）。拼 decisions morning-1335 可覆盖 33 个交易日。不装成 20 天三路已齐。 |
| B2 成交 | **部分过** | 纸面成交价与决策 ref 可核；market_cache/evidence 无任何 15m 文件，回测假设价全部「未核到」。 |
| B3 持仓/权重/regime | **过** | 6 仓皆 8 月 Top50 普通股，无 ETF，无 V 残留，单票<15%，regime=bull。 |
| B4 止损 | **不过** | 可分类为 hourly_stop/time_stop 的只有 1 次（8/19 V time_stop）。forced_exit 记录 18 次但 reasons 未写明类型。不足 10 次可分类止损。 |
| B5 失败闭环 | **过** | 8/13–18 缺 ATR → 0 targets 成立；鉴权/下单失败有日志；acceptance 例行在；systemctl 假失败已单独剥离。Friday 专向事故口径偏弱，但不构成 fail-closed 破裂。 |

禁止装完：B1 不把 7 天三路说成 20 天；B2 不编 15m；B4 不把未分类 forced_exit 算成 10 次 hourly/time stop。

## B1 信号时间

- 结论：**部分过**
- 规格：每交易日 13:35 America/New_York
- shadow_signals.jsonl 行数：42（全是 N1 身份）；其中 13:35 行：7 日
- cron-logs `signal-*T173501Z.log`（13:35 ET）：['2026-08-13', '2026-08-14', '2026-08-17', '2026-08-18', '2026-08-19', '2026-08-20', '2026-08-21']
- state.db decisions 带 13:35 的交易日：33

### 覆盖

- N1 活体交易日（2026-08-13..21，美东）：['2026-08-13', '2026-08-14', '2026-08-17', '2026-08-18', '2026-08-19', '2026-08-20', '2026-08-21']
- 其中三路都有 13:35：['2026-08-13', '2026-08-14', '2026-08-17', '2026-08-18', '2026-08-19', '2026-08-20', '2026-08-21'] → **7 天**
- 拼 inherited decisions morning-1335：33 天（2026-07-07 .. 2026-08-21）
- Aug 12 是交易日，但 N1 13:35 没有：当日 11:05–11:34 ET 日志全是 `N1_CRON_ARMED missing` no-op；武装发生在 20:51 UTC / 16:51 ET（收盘后）。记 **漏日（部署日未武装）**，不是运行后漏跑。
- Aug 15/16 周末，不计入。

### 偏差

- N1 shadow 相对 13:35:00 延迟秒：[1.508, 1.872, 2.323, 1.915, 2.223, 1.842, 1.858]；均值 1.934s
- N1 signal 日志 start 一律 `et=13:35:01`（cron 文件名 T173501Z），rc=0
- inherited E2 decisions 13:35 多数在 13:35:37–38（约 +37s），不是 +10 分钟
- 周一全缺的日子：无
- 无「每天晚 10 分钟」或「周一常漏」的系统偏差。

### 为何不是「过」

规格要求连续抽样 **至少 20 个美东交易日的 13:35** 且三路对。N1 活体从 8/13 起算，到 8/21 只有 7 个交易日；shadow 与 signal-*.log 都盖不住 20 天。拼 decisions 能到 33 天（含 8/12 之外的 Jul 7–Aug 21 交易日），但那是 E2 继承库，不是 N1 三路。按「不够 20 天不要装」→ **部分过（数据不足）**。

复核：`b1_signal_calendar.csv`

## B2 成交

- 结论：**部分过**
- 规格：隔离回测成交 = 13:45 ET 15m open × (1+sign×10bp)，佣金 1bp；纸盘 = 长桥 Demo 限价 limit_deviation_bps=10，以 executions 价为准。
- executions 笔数：48
- 有决策 ref 的笔数：48
- 15m 核到的笔数：**0**（`var/paper/market_cache` 只有 daily/hourly/trading_days；evidence 无 15m 文件；全树文件名无 15m/15min）

### 近期 V → CRM（2026-08-19 13:35 ET）

| symbol | side | qty | fill | ref | limit | fill-ref bps | 15m |
|---|---|---:|---:|---:|---:|---:|---|
| V.US | sell | 20 | 368.825 | 368.67 | 368.74 | 4.20 | 未核到 |
| CRM.US | buy | 28 | 205.54 | 205.8 | 205.63 | -12.63 | 未核到 |

- V 卖：fill 368.825 vs ref 368.67 = **+4.20 bps**（卖得略贵，非逆向）
- CRM 买：fill 205.54 vs ref 205.8 = **-12.63 bps**（买得更便宜，非买贵）
- 限价：V 368.74 / CRM 205.63，与 `limit_deviation_bps=10` 同量级（相对 ref 约 +1.9 / -8.3 bps）
- 回测假设价：未核到，不编。

### 相对决策 ref 的总体

- 全样本 fill-vs-ref 均值 -10.45 bps，|bps| 均值 28.89，n=48
- 有 side 的不利偏离均值 17.08 bps，n=48
- |bps|>30 异常 10 笔（几乎全是 7 月–8/11 继承成交；N1 活体 V/CRM 都不在此列）
- 继承成交里常见「买贵/卖便宜」大偏离，例如 7/8 TSLA 卖 fill 393.82 vs 决策 ref 402.9；7/9 AAPL 买 314.68 vs 312.84。这些是 E2 纸盘历史，不是 8/19 之后的 N1 新单。
- `shadow_trades.csv` 前 59 行 `intended_order_time` 全是 `2026-08-13T10:35:01Z`，那是 N1 接手时的历史回放戳，**不要当成 8/13 10:35 真的下了 59 笔**。只有最后两行 8/19 17:35Z 是 N1 活体。

复核：`b2_fills.csv`

## B3 持仓 / 权重 / regime

- 结论：**过**
- 最新仓位 6 只：['AMZN.US', 'CRM.US', 'JPM.US', 'MSFT.US', 'PANW.US', 'PLTR.US']
- 全部 ∈ `config/strategy_e2/20260803.json` Top50，无 ETF。
- V 已出（8/19 time_stop），现仓无 V。
- 单票市值/NAV 均未超 15%（daily close 近似，JPM 14.10% 最高，见 b3_positions.md）。
- regime 字段 `bull`/`bull`，合法。
- Strong Bull 风险预算配置值 0.85%（`strategy_e.toml` `strong_bull_trade_risk=0.0085`），本次未改。

复核：`b3_positions.md`

## B4 止损

- 结论：**不过**
- 规格：initial=entry-2.5×entry_atr；trailing=peak-3.0×entry_atr 且 max_r≥1；时间止损 12 交易日；触发价=1h close。
- 找到 forced_exit 事件 **18** 次（决策 reasons）。
- 其中能标成 `hourly_stop` / `time_stop` 的：**1** 次。

### 8/19 V time_stop（唯一完整样本）

- 来源：shadow `reason_for_exit={V.US: time_stop}` + 备份库 `state.db.pre-crm-meta.20260819T180830Z`
- entry_price=365.750 entry_atr=7.672278554669094 days_held=12 max_r=0.2788741290809769253167672235
- initial_stop=346.5693036133272650 ；2.5×ATR 期望 346.5693036133272650 → **相符**
- days_held=12 ≥ 12 → **相符**；max_r≈0.279 < 0.5（代码额外门）
- trailing_stop=null（max_r<1，未开 3.0× 跟踪）
- 成交 fill=368.825，initial_stop=346.57，不是跌破 initial 的 hourly_stop

### 其余 forced_exit（不能装成已分类）

决策只写 `forced_exit:SYMBOL`，N1 树里 **零处** 出现 `hourly_stop` 字符串。risk 槽（如 7/21 14:35、7/30 11:35、8/06 11:35）按代码路径更像 hourly_stop，但没有当时的 reason_for_exit / 1h close 证据，**不分类**。

不足 10 次可验证 hourly_stop/time_stop → **B4 不过**。

复核：`b4_stops.csv`

## B5 失败闭环

- 结论：**过**
- 缺 ATR：8/13–18 的 13:35 atr20 全 None，targets=0，没有开仓；8/19 ATR 恢复后才动。fail-closed 成立。
- 下单失败：order_events 有记录；N1 活体 8/19 对账成功。
- 鉴权失败：8/13 09:35 signal 日志 OAuth rc=1，有完整 traceback。
- 告警例行：acceptance 每日三阶段 + operations 复盘规范存在；两个周五 final 报告存在。缺少独立 Friday 事故口径周报。
- systemctl 假失败：fallback/watchdog 找不到 `systemctl` / 无 user bus，rc=1。**不当成 fail-closed 破了**。正式 13:35 signal 仍 rc=0。

复核：`b5_failclosed.md`

## 第三方复核路径

```
# B1
python3 -c "import json; ..."  # shadow_signals.jsonl 滤 13:35 ET
ls var/paper/cron-logs/signal-*T173501Z.log
sqlite3 var/paper/state.db "select id, created_at, json_extract(payload,'$.decision_id'), json_extract(payload,'$.time') from decisions"
# B2
sqlite3 var/paper/state.db "select created_at, payload from executions"
ls var/paper/market_cache | rg 15m   # 期望空
# B3
sqlite3 var/paper/state.db "select payload from snapshots order by id desc limit 1"
python3 -c "import json; print([x['symbol'] for x in json.load(open('config/strategy_e2/20260803.json'))['top50']])"
# B4
rg time_stop reports/runtime/shadow/strategy_n1/20260812/shadow_signals.jsonl
sqlite3 var/paper/backups/state.db.pre-crm-meta.20260819T180830Z "select payload from strategy_state where key='paper_position_meta'"
# B5
rg -n atr20 reports/runtime/shadow/strategy_n1/20260812/shadow_signals.jsonl | head
sed -n '1,20p' var/paper/cron-logs/signal-20260813T133501Z.log
```

以上命令只读。
