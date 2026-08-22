# B5 失败闭环（只记现行行为，不改代码）

## 1. 缺 ATR / atr20 None → 0 targets

N1 shadow 13:35 逐日：

| date_et | atr20_null / rankings | decision.targets | reason_for_no_trade | selected |
|---|---:|---:|---|---|
| 2026-08-13 | 50/50 | 0 | no_strategy_e2_candidates |  |
| 2026-08-14 | 50/50 | 0 | no_strategy_e2_candidates |  |
| 2026-08-17 | 50/50 | 0 | no_strategy_e2_candidates |  |
| 2026-08-18 | 50/50 | 0 | no_strategy_e2_candidates |  |
| 2026-08-19 | 0/50 | 2 |  | CRM.US |
| 2026-08-20 | 0/50 | 0 | no_strategy_e2_candidates |  |
| 2026-08-21 | 0/50 | 0 | no_strategy_e2_candidates |  |

证据：
- 2026-08-13/14/17/18 的 13:35：`candidate_rankings` 50/50 的 `atr20=null`，decision `targets=[]`，no_trade=`no_strategy_e2_candidates`。没有新开仓。
- 2026-08-19 13:35：atr20 恢复（0/50 null），才出现 `forced_exit:V.US` + CRM 开仓。
- 代码现行：`paper.py` `_strategy_c_candidates` 对 `atr<=0` 直接 `continue`（fail-closed，不开仓）。
- 8/13–18 符合「缺 ATR 不能开仓」。

## 2. 下单失败是否有记录

- `order_events` 共 11 条，全部是 2026-07-18/20/22 的 cancel / reprice_cancel / weekend_non_regular / broker_terminal_status，有 order_id 与原因。
- N1 活体 8/19 V 卖 / CRM 买：acceptance-final-2026-08-19 = `TRADE_RECONCILED`，无失败事件。
- 8/13–18、8/20–21：decision targets=0，无新单，不构成「下单失败被吞」。

## 3. 接口 / 鉴权失败日志

- `cron-logs/signal-20260813T133501Z.log`：09:35 ET 误触发的 signal 窗口，`LongbridgeSDKAuthRequired: Longbridge SDK OAuth authorization has not completed`，rc=1。这是鉴权失败有日志。
- 同日 13:35 正式 signal（`signal-20260813T173501Z.log`）rc=0。
- 若干 early 日志（risk-20260813T14/153501Z、order-lifecycle-20260813T133701Z、close-snapshot-20260813T161001Z）同样留下 OAuth traceback。
- **未读取** env / token / 密钥文件。

## 4. 告警 / 监测例行 / Friday 事故口径

存在的例行：
- 每日 acceptance 三阶段：preflight 13:25 / post-signal 13:40 / final 14:00 ET，目录 `reports/runtime/operations/acceptance/`。
- 两个周五：`acceptance-final-2026-08-14.md` = NO_TRADE_CONFIRMED；`acceptance-final-2026-08-21.md` = NO_TRADE_CONFIRMED。
- 操作复盘规范：`reports/runtime/operations/README.md` 要求每次信号/下单/watchdog/异常留复盘。
- watchdog JSON：`reports/runtime/watchdog/post_signal_watchdog_20260813*.json` 与 `20260814*.json`（之后 watchdog 进程因 systemctl 缺失而崩，见下）。

缺口：
- 未找到单独的「Friday 事故口径」周报模板（无 事故/incident Friday 专文）。shadow_2w_report.md 仍是占位「尚未到报告生成时间」。
- 监测例行（acceptance + operation review）在，Friday 专向事故口径弱。

## 5. systemctl 假失败（不要当成 fail-closed 破了）

现行 cron 跑在 box 用户 crontab，不是 systemd。
`execution-fallback-20260821T175501Z.log`（及 8/17 起多日同构）：
```
FileNotFoundError: [Errno 2] No such file or directory: 'systemctl'
===== N1 execution-fallback end ... rc=1 =====
```
watchdog 8/14 JSON 也有：`Failed to connect to user scope bus ... $DBUS_SESSION_BUS_ADDRESS and $XDG_RUNTIME_DIR not defined`。
这是 watchdog/fallback **去问 systemd 状态** 的环境假失败，不是券商下单或 ATR fail-closed 被绕过。
对照：同窗口正式 signal 日志 rc=0，8/19 成交对账 TRADE_RECONCILED。

## 6. 其它现行行为

- control_state.armed=[(1,)]（只读，未碰 N1_CRON_ARMED 文件内容以外的存在性确认：文件在 `var/paper/N1_CRON_ARMED`）。
- 8/12 当日 11:05–11:34 ET 的 signal 日志全部 `N1_CRON_ARMED missing; no-op exit 0 (disarmed)`，武装前安全 no-op。
