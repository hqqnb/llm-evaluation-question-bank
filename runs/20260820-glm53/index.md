# GLM-5.3 发布归档：20260820-glm53

本页是 GLM-5.3 本轮测评的统一入口。原始回答、模型可见 Agent 轨迹和未运行说明
均进入 `llm-evaluation-question-bank`；C01-C07 的可运行 Coding 交付物另有
GitHub Pages 预览。

本轮不发布分数、排名或正式结论。Coding 预览只做机械提取，不修复模型原始代码。
Agent 页面只保留模型可见内容，不公开隐藏推理。

## 推理题

| 题目 | 状态 | 原始回答 |
| --- | --- | --- |
| `reasoning-r01` | `success_no_visible_answer` | [查看](./responses/reasoning-r01/glm-5.3.md) |
| `reasoning-r02` | `success` | [查看](./responses/reasoning-r02/glm-5.3.md) |
| `reasoning-r03` | `success` | [查看](./responses/reasoning-r03/glm-5.3.md) |
| `reasoning-r04` | `success` | [查看](./responses/reasoning-r04/glm-5.3.md) |
| `reasoning-r05` | `success` | [查看](./responses/reasoning-r05/glm-5.3.md) |
| `reasoning-r06` | `success` | [查看](./responses/reasoning-r06/glm-5.3.md) |
| `reasoning-r07` | `success` | [查看](./responses/reasoning-r07/glm-5.3.md) |
| `reasoning-r08` | `success_no_visible_answer` | [查看](./responses/reasoning-r08/glm-5.3.md) |

R01 和 R08 均为 HTTP 200，但在官方 `131072` 输出上限处结束且没有可见最终答案；
归档页不代填参考答案。

## Coding 题

| 题目 | 状态 | 原始回答 | GitHub Pages 预览 |
| --- | --- | --- | --- |
| `coding-c01` | `success` | [查看](./responses/coding-c01/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c01/glm-5.3/) |
| `coding-c02` | `success` | [查看](./responses/coding-c02/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c02/glm-5.3/) |
| `coding-c03` | `success` | [查看](./responses/coding-c03/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c03/glm-5.3/) |
| `coding-c04` | `success` | [查看](./responses/coding-c04/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c04/glm-5.3/) |
| `coding-c05` | `success` | [查看](./responses/coding-c05/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c05/glm-5.3/) |
| `coding-c06` | `success` | [查看](./responses/coding-c06/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c06/glm-5.3/) |
| `coding-c07` | `success` | [查看](./responses/coding-c07/glm-5.3.md) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260820-glm53/coding-c07/glm-5.3/) |
| `coding-c08` | `not_run` | [未运行说明](./responses/coding-c08/glm-5.3.md) | — |

C08 按用户要求未运行，因此没有模型代码或预览链接。

## Agent 题

| 题目 | 状态 | 模型可见轨迹 |
| --- | --- | --- |
| `agent-t01` | `success` | [查看](./responses/agent-t01/glm-5.3.md) |
| `agent-t02` | `success` | [查看](./responses/agent-t02/glm-5.3.md) |
| `agent-t03` | `success` | [查看](./responses/agent-t03/glm-5.3.md) |
| `agent-t04` | `success` | [查看](./responses/agent-t04/glm-5.3.md) |
| `agent-t05` | `success` | [查看](./responses/agent-t05/glm-5.3.md) |
| `agent-t06` | `success` | [查看](./responses/agent-t06/glm-5.3.md) |
| `agent-t07` | `success` | [查看](./responses/agent-t07/glm-5.3.md) |
| `agent-t08` | `success` | [查看](./responses/agent-t08/glm-5.3.md) |

Agent 页面仅归档模型可见工具闭环，未写入正式分数或排名。

## 多模态题

| 题目 | 状态 | 说明页 |
| --- | --- | --- |
| `multimodal-mm01` | `not_run` | [未运行说明](./responses/multimodal-mm01/glm-5.3.md) |
| `multimodal-mm02` | `not_run` | [未运行说明](./responses/multimodal-mm02/glm-5.3.md) |
| `multimodal-mm03` | `not_run` | [未运行说明](./responses/multimodal-mm03/glm-5.3.md) |
| `multimodal-mm04` | `not_run` | [未运行说明](./responses/multimodal-mm04/glm-5.3.md) |

MM01-MM04 本轮未运行；说明页不代表模型失败或能力结论。
