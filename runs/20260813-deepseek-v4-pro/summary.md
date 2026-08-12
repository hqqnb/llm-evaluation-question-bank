# DeepSeek-V4-Pro 测评汇总（2026-08-13，max 思考强度）

运行方式：venv 直连 API 两遍（r1、r2，15 题，排除 coding-c08）+ Codex Agent 一遍（16 题，含 coding-c08）。多模态题因模型纯文本限制跳过。

## 一、推理题自动判分

| 题目 | r1 | r2 | Codex | 标准答案 |
| --- | --- | --- | --- | --- |
| reasoning-r01 | 20460 ✓ | 20460 ✓ | 20460 ✓ | 20460 |
| reasoning-r02 | 21 ✓ | 21 ✓ | 21 ✓ | 21 |
| reasoning-r03 | 735 ✓ | 735 ✓ | 735 ✓ | 735 |
| reasoning-r04 | 0 ✓ | 0 ✓ | 0 ✓ | 0 |
| reasoning-r05 | 5429515560378 ✓ | 5429515560378 ✓ | 5429515560378 ✓ | 5429515560378 |
| reasoning-r06 | 369 ✓ | 371 ✗ | 369 ✓ | 369 |
| reasoning-r07 | C ✓ | C ✓ | C ✓ | C |
| reasoning-r08 | 29010 ✓ | 29010 ✓ | 29010 ✓ | 29010 |

## 二、Coding 三次运行预览链接

| 题目 | r1 预览 | r2 预览 | Codex 预览 |
| --- | --- | --- | --- |
| coding-c01 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c01/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c01/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c01/codex-deepseek-v4-pro/) |
| coding-c02 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c02/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c02/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c02/codex-deepseek-v4-pro/) |
| coding-c03 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c03/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c03/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c03/codex-deepseek-v4-pro/) |
| coding-c04 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c04/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c04/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c04/codex-deepseek-v4-pro/) |
| coding-c05 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c05/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c05/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c05/codex-deepseek-v4-pro/) |
| coding-c06 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c06/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c06/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c06/codex-deepseek-v4-pro/) |
| coding-c07 | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c07/deepseek-v4-pro-r1/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c07/deepseek-v4-pro-r2/) | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c07/codex-deepseek-v4-pro/) |
| coding-c08 | — | — | [打开](https://hqqnb.github.io/llm-evaluation-previews/20260813-deepseek-v4-pro/coding-c08/codex-deepseek-v4-pro/) |
