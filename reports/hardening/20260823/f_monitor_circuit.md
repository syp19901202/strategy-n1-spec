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

## F1–F4 见同目录全文

全文与 `docs/n1_monitor_circuit.md` 同步。
