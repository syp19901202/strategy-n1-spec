# 失败 / 归档实验索引（硬化 G3，2026-08-24）

避免把已经试过的刀再砍回 N1。全部是**独立仓库**。纸交易仍是 N1。

替换门（对 N 系列）：同一隔离窗 CAGR **> 21.82%** 且 MaxDD **≤ 7.71%**。即使过门也不自动上纸面。

| 代号 | 仓库 | 只改了什么 | 本窗结果 | 状态 | 不要再做的事 |
| --- | --- | --- | --- | --- | --- |
| **F1** | [strategy-f1-spec](https://github.com/syp19901202/strategy-f1-spec) | 在 E2 上叠仓位/暴露实验（A/B/C） | 正式 **REJECT**。F1-C 年化 13.21%、累计 27.98%，过不了 52% 累计等门。F1-A 年化 22.01% 但回撤 9.28%，且不是 N1 门 | 归档 | 不要把 F1 的暴露/叠加仓叠进 N1 |
| **G** | [strategy-g-spec](https://github.com/syp19901202/strategy-g-spec) | 行业优先；正式变体 G-B = Top50 内最强 2 行业、每行业 1 只 | **REJECT**。G-B 累计 −9.73%、年化 −5.03%、暴露 5.6%。3×滑点利润因子 < 1 | 归档 | 不要改成「两个行业两只龙头」 |
| **N2** | [strategy-n2-spec](https://github.com/syp19901202/strategy-n2-spec) | 体制广度人口：写死 19 只 → 当月 Top50 | **REJECT**。21.72% / 7.71%。2024 与回撤与 N1 相同；2025-02-26 起 N2 多留 5 日 bull，多 3 笔，年化少 0.10pp | 归档 | 不要改 19 只体制人口来「再挤 0.1pp」 |
| **N3** | [strategy-n3-spec](https://github.com/syp19901202/strategy-n3-spec) | 只在 Strong Bull：风险 1.40%，单票 25%，行业 40% | **PASS 但不替换**。23.33% / 7.63%（+1.51pp）。提升主要在 2025 不是 2024。3×滑点 15.71% | 归档参考 | 提升不够换；不要自动切换 N1↔N3 |
| **N4** | [strategy-n4-spec](https://github.com/syp19901202/strategy-n4-spec) | 高风险高集中做成常态：Bull/Strong Bull 1.40%，最多 4 只，单票 28%，行业 45% | **REJECT**。14.03% / 10.06%。两年都更差。3×滑点年化 0.81% | 归档 | **同方向再调风险/集中度：收掉** |
| **N5** | [strategy-n5-spec](https://github.com/syp19901202/strategy-n5-spec) | 只改时间止损 12 → 20 日 | **REJECT**。19.55% / 7.71%（−2.27pp）。回撤没坏，年化掉了 | 归档 | 不要为了「让赢家多跑」把 12 日改成 20 日再写回 N1 |

母版 **E2**（`70cc50ad965234b99a5a136d4e4f283eedd4281f`）不是失败，是 N1 的唯一父版。N1 只动 Strong Bull 0.85%。

---

## 读法

1. **已经试过、证明无用或有害的**：改体制人口（N2）、行业两龙头（G）、加暴露叠仓（F1）、加持有天数（N5）、把高集中当常态（N4）。
2. **试过、略好、不够换**：N3。留作参考，不上纸面。
3. **还没试、也不许借加固去试**：任何回写 N1 生效参数的做法。新刀必须新字母。

详细出处：

- F1：`strategy-f1-spec` `reports/validation/strategy_f1/20260813/08_accept_reject.md`
- G：`strategy-g-spec` `reports/validation/strategy_g/20260813/08_accept_reject.md`
- N2：`strategy-n2-spec` `README.md` 与 `03_n2_failure_causes.md`
- N3：`strategy-n3-spec` `reports/independent/20260821/03_n3_conclusion.md`
- N4：`strategy-n4-spec` `reports/independent/20260822/03_n4_conclusion.md`
- N5：`strategy-n5-spec` `reports/independent/20260823/03_n5_reject.md`
