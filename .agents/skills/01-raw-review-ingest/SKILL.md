---
name: 01-raw-review-ingest
description: 将手动复制的公开应用商店评论整理为 ai-app-feedback-analysis 项目 data/raw_reviews.csv 行。支持预览模式和明确授权后的安全追加模式；不做标注、不生成分析结论、不覆盖已有 CSV。
---

# 01 Raw Review Ingest

## 用途

本技能用于项目“AI 应用用户反馈与产品体验分析”的原始评论入库阶段。它把手动复制的公开应用商店评论整理为 `data/raw_reviews.csv` 行。

本技能不是爬虫，不调用外部 API，不做情绪标注、问题分类、优先级判断或产品建议。

## 输出 Schema

字段顺序固定为：

```csv
id,source,app_name,review_date,rating,review_text,notes
```

## 两种模式

### Mode A：预览模式

当用户说“先输出让我检查”时使用。

- 只输出 append-ready CSV rows。
- 不写入任何文件。
- 不包含表头，除非用户明确要求。

### Mode B：直接追加模式

只有当用户明确说“直接写入 data/raw_reviews.csv”或“追加到 raw_reviews.csv”时使用。

- 先生成 CSV rows。
- 使用 `scripts/append_raw_reviews.py` 追加到 `data/raw_reviews.csv`。
- 保留现有表头和已有行。
- 如果文件不存在，脚本会创建正确表头。
- 如果任一生成的 `id` 已存在，必须停止并提示用户，不追加任何行。
- 不覆盖、不重排已有行。

推荐命令：

```bash
python3 scripts/append_raw_reviews.py
```

脚本从标准输入读取要追加的 CSV rows。

## 默认输出规则 / Concise Result Output Rules

默认保持结果导向，避免输出中间过程。

- 直接追加模式下，不输出完整 CSV 内容。
- 不列出所有正常行。
- 不展示 `data/raw_reviews.csv` 的完整内容。
- 将问题、警告、跳过 ID 和需要用户处理的动作放在最前面。
- 如果没有问题，只说：`未发现需要处理的问题。`
- 回复保持简短，聚焦可执行结果。

直接追加模式写入后只汇报：

- 新增 id。
- 跳过 id 及原因。
- 重复 id 或格式问题。
- 是否成功写入。

不要打印全部生成的 CSV rows，除非用户明确要求预览模式。预览模式只输出 append-ready CSV rows，不附加长解释。

## 接受的数据来源

`source` 只能使用以下值之一：

- `App Store`
- `应用宝`
- `小米应用商店`
- `华为应用市场`
- `OPPO应用商店`
- `vivo应用商店`

如果用户给出其他来源，不要自行改写；应提示用户确认来源名称。

## ID 规则

- 豆包：`DB_001`、`DB_002`
- DeepSeek：`DS_001`、`DS_002`
- Kimi：`KIMI_001`、`KIMI_002`

当前主分析 App 只使用豆包、DeepSeek、Kimi。若用户提供其他 App 评论，先提示确认是否作为补充样本，不要混入主分析样本。

如果用户提供 `starting_id`，从该 ID 开始连续递增。不要跳号，除非用户明确要求。

## 元数据规则

- 已提供 `review_date` 时原样保留。
- 已提供 `rating` 时原样保留。
- 缺少 `review_date` 时留空。
- 缺少 `rating` 时留空。
- 不从评论文本推断日期或评分。
- 缺少 `notes` 时填写 `手动公开采样`。
- 不生成用户没有提供的元数据。

## 隐私和清洗规则

- 不保留用户名、头像、主页链接、用户 ID、设备 ID、手机号、邮箱、订单号等私人信息。
- 保持 `review_text` 含义不变。
- 去除开头和结尾多余空格。
- 保留中文标点和用户原始表达。
- 保留强烈措辞，除非其中包含私人信息。

## CSV 转义规则

- `review_text` 必须使用英文双引号包裹。
- `review_text` 中的英文双引号 `"` 必须写成 `""`。
- `notes` 如包含标点、逗号、换行或引号，也使用英文双引号包裹。
- 输出必须是合法 CSV，不使用 Markdown 表格。

## 示例

预览模式输入：

```text
source: 小米应用商店
app_name: 豆包
starting_id: DB_011
notes: 小米应用商店评论页手动采样；安卓端补充样本

评论1：
review_date: 2026-05-10
rating: 2
review_text: 回答有时候不准，问同一个问题前后说法不一样。
```

预览输出：

```csv
DB_011,小米应用商店,豆包,2026-05-10,2,"回答有时候不准，问同一个问题前后说法不一样。","小米应用商店评论页手动采样；安卓端补充样本"
```

直接追加模式提示词示例：

```text
请使用 .agents/skills/01-raw-review-ingest/SKILL.md 的 direct append mode，把以下评论直接写入 data/raw_reviews.csv。写入后只汇报新增 id、跳过 id、重复 id 或格式问题，不要输出全量 CSV 内容。
```
