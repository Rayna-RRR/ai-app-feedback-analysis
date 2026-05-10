---
name: 03-label-quality-check
description: 检查 ai-app-feedback-analysis 项目 data/clean_reviews.csv 的标注质量。默认 result-only mode 只输出 QA 报告，不修改 review 数据；apply-fixes mode 仅按 docs/qa_report.md 中明确建议移出的 ID 从 clean_reviews.csv 移出并归档。
---

# 03 Label Quality Check

## 用途

本技能用于标注后的质量检查。默认 `result-only mode` 运行本地 QA 脚本，检查 `data/clean_reviews.csv` 是否存在明显数据质量问题，并写入 `docs/qa_report.md`。

当用户明确要求 `apply-fixes mode` 时，本技能可以根据 `docs/qa_report.md` 中明确列为“建议移出 clean_reviews.csv”或“需要移出 clean_reviews.csv”的评论 ID，从 `data/clean_reviews.csv` 移出对应行，并归档到 `data/excluded_reviews.csv`。

它不同于 `02-clean-review-labeling`：

- `02-clean-review-labeling` 负责给 raw rows 打标签。
- `03-label-quality-check` 负责检查已经打好的标签是否一致、完整、可用。

## 何时使用

- 完成一批 `data/clean_reviews.csv` 标注后。
- 写分析报告前。
- 追加新 clean rows 后，需要复核标签质量时。
- 怀疑存在非法枚举值、空字段、过泛建议或优先级不一致时。
- 用户明确要求 `result-only mode` 时，只做质检和报告更新。
- 用户明确要求 `apply-fixes mode`，且 `docs/qa_report.md` 已明确列出建议移出或需要移出 `clean_reviews.csv` 的 ID 时，执行移出和归档。

## 何时不要使用

- 不用于采集评论。
- 不用于生成最终分析结论。
- 不用于替代人工标注判断。
- 不用于调用外部 API 或 LLM API。
- `result-only mode` 不用于修改 `data/raw_reviews.csv` 或 `data/clean_reviews.csv`。
- `apply-fixes mode` 也不得修改 `data/raw_reviews.csv`，只允许处理 `data/clean_reviews.csv` 和 `data/excluded_reviews.csv`。

## 模式

### result-only mode

默认模式。用于检查 `data/clean_reviews.csv`，更新 `docs/qa_report.md`，并输出简洁 QA 结果。

- 不修改 `data/raw_reviews.csv`。
- 不修改 `data/clean_reviews.csv`。
- 不修改 `data/excluded_reviews.csv`。
- 只报告问题 ID、问题原因、建议处理和补样需求。

### apply-fixes mode

只有当用户明确说 `apply-fixes mode` 时使用。

用途：根据 `docs/qa_report.md` 中明确列为“建议移出 clean_reviews.csv”或“需要移出 clean_reviews.csv”的评论 ID，处理 `data/clean_reviews.csv`，并将移出的评论归档到 `data/excluded_reviews.csv`。

固定规则：

1. 不修改 `data/raw_reviews.csv`。
2. 只处理 `docs/qa_report.md` 中明确建议移出 `clean_reviews.csv` 的 ID。
3. 从 `data/clean_reviews.csv` 中移除这些 ID 对应的行。
4. 不重排 `data/clean_reviews.csv` 中其他行。
5. 不改写其他正常评论。
6. 如果 `data/excluded_reviews.csv` 不存在，则创建它。
7. `data/excluded_reviews.csv` 字段固定为：

```csv
id,source,app_name,review_date,rating,review_text,exclude_reason,notes
```

8. 将移出的 `id`、`source`、`app_name`、`review_date`、`rating`、`review_text`、移出原因写入 `data/excluded_reviews.csv`。
9. 完成后只汇报：
   - 移出了哪些 id。
   - 每个 id 的移出原因。
   - 需要补充几条替换评论。
   - 建议补充的 `app_name`、`source` 和评论类型。
10. 不输出全量 `clean_reviews.csv` 内容。
11. 不输出全量 `excluded_reviews.csv` 内容。
12. 如果 `docs/qa_report.md` 中没有明确建议移出的 ID，只回复：`未发现需要移出 clean_reviews.csv 的评论。`

如果 QA 报告只是写“建议复核”“可能需要检查”“可保留当前标签”，不得按 apply-fixes mode 移出。

## 默认输出规则 / Concise Result Output Rules

默认只输出结果，不展示中间过程。

- QA 输出默认采用 result-only 结构。
- 不列出所有正常行。
- 不输出 `data/clean_reviews.csv` 的完整内容。
- 将问题、警告、建议移出 clean 的 ID、建议复核的 ID 和需要补充评论的动作放在最前面。
- 如果所有检查行都通过，只说：`未发现需要处理的问题。`
- 回复保持简短，聚焦可执行处理。

默认只报告：

- 发现问题的 id。
- 问题原因。
- 建议处理。
- 建议移出 `clean_reviews.csv` 的 id。
- 建议复核的 id。
- 建议重写 `product_suggestion` 的 id。
- 需要补充或替换评论的数量、`app_name`、`source` 和评论类型。

`docs/qa_report.md` 也应使用同样的简洁结果结构：只记录问题、原因、建议处理和补样需求，不记录正常评论清单。

## 检查内容

QA 会检查：

- 必需列是否缺失。
- `usage_scene`、`issue_category`、`ai_issue_type`、`user_sentiment`、`priority` 是否为合法值。
- `review_text`、`user_need`、`product_suggestion` 是否为空。
- `product_suggestion` 是否包含“优化体验”“提升质量”“增强用户粘性”“完善功能”等模糊表达。
- P0/P2 是否和严重问题关键词明显冲突。
- 评论是否信息量过低。
- `ai_issue_type` 为 `不明确/其他` 的比例是否过高。
- 是否有无关或低信息量评论混入 clean 数据。
- 是否存在 App 与样本来源明显不匹配的问题，例如非主分析 App 混入豆包、DeepSeek、Kimi 样本。
- 是否存在明显非 AI 产品体验评论，例如奶茶、点餐、外卖、购物、配送等生活服务评论混入 AI 应用样本。

## 使用方式

在项目根目录运行：

```bash
python3 scripts/label_quality_check.py
```

输出：

- 终端 QA 摘要。
- `docs/qa_report.md`。

## 如何解读

- 缺失列和非法枚举值优先修正。
- 模糊建议需要改写为具体、可执行、可验证的建议。
- 优先级警告只提示风险，不自动代表标注错误，需要人工复核。
- “不明确/其他”比例过高通常说明标签口径需要复盘。
- 低信息量评论应判断是否从正式分析样本中移除。

## 示例提示词

result-only mode：

```text
请使用 .agents/skills/03-label-quality-check/SKILL.md 的 result-only mode，对 data/clean_reviews.csv 进行第二轮质检，并更新 docs/qa_report.md。
```

apply-fixes mode：

```text
请使用 .agents/skills/03-label-quality-check/SKILL.md 的 apply-fixes mode，根据 docs/qa_report.md 处理需要移出 clean_reviews.csv 的评论，并归档到 data/excluded_reviews.csv。
```
