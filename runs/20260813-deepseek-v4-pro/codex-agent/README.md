# DeepSeek-V4-Pro 原生 Codex Agent 运行（AutomationBench 5 题）

- 运行方式：在当前 Codex 会话中原生执行，Agent（deepseek-v4-pro）通过工具调用 AutomationBench 模拟环境，完成后再用官方断言打分。
- 结果：5 题全部严格通过（100%）。

| 题目 | 严格通过 | 断言 |
| --- | --- | --- |
| T01 退款处理（support.gorgias_refund_processing） | 是 | 37/37 |
| T02 Demo 排期（support.intercom_demo_scheduling） | 是 | 30/30 |
| T03 消防演练（operations.asana_fire_drill） | 是 | 5/5 |
| T04 负向筛选（sales.negative_selection） | 是 | 7/7 |
| T05 费用异常（finance.expense_anomaly_detection） | 是 | 4/4 |

- 原始断言结果：各 `t0*.json`。
- 复现脚本：`scripts/`（`codex_run_export.py` 一键复现全部 5 题）。
