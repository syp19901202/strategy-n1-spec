# N1 版本状态（硬化 G4，2026-08-24）

防止「现在到底在跑哪个」混掉。

| 名字 | 角色 | 跑纸面？ | 仓库 |
| --- | --- | --- | --- |
| **N1** | **当前纸面主策略** | **是**（长桥 Demo） | [strategy-n1-spec](https://github.com/syp19901202/strategy-n1-spec) |
| E2 | N1 的母版。冻结对照 | 否（已被 N1 接替） | 引擎提交 `70cc50ad965234b99a5a136d4e4f283eedd4281f` |
| F1 | 隔离研究，REJECT | 否 | strategy-f1-spec |
| G | 隔离研究，REJECT | 否 | strategy-g-spec |
| N2 | 隔离研究，REJECT | 否 | strategy-n2-spec |
| N3 | 隔离研究，轻微增强，**不替换** | 否 | strategy-n3-spec |
| N4 | 隔离研究，REJECT | 否 | strategy-n4-spec |
| N5 | 隔离研究，REJECT | 否 | strategy-n5-spec |

N1 身份钉：`Strategy_N1_70cc50a_risk085`。冻结日 2026-08-14。纸盘独立 `state.db`、独立门闩 `N1_CRON_ARMED`。

本规格仓没有运行时、持仓、token、行情。云电脑上的纸盘不要从本仓库改。

提高收益 = 新字母 + 独立仓库 + 先复现 N1 四数。不得回写本仓生效参数。
