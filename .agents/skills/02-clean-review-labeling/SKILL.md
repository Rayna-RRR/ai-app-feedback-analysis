---
name: 02-clean-review-labeling
description: 将 ai-app-feedback-analysis 项目 data/raw_reviews.csv 中的有效评论标注为 data/clean_reviews.csv 行。支持预览模式和明确授权后的安全追加模式；可按指定 ID、未标注 ID 或用户粘贴的 raw rows 处理。
---

# 02 Clean Review Labeling

## 用途

本技能用于项目“AI 应用用户反馈与产品体验分析”的评论标注阶段。它把 `data/raw_reviews.csv` 中的有效 raw rows 转换为 `data/clean_reviews.csv` 可追加行。

本技能不采集数据、不改写原评论含义、不生成最终分析结论、不重标已有 clean rows，除非用户明确要求。

## 输入 Schema

```csv
id,source,app_name,review_date,rating,review_text,notes
```

输入可以来自：

- 用户粘贴的 raw CSV rows。
- `data/raw_reviews.csv` 中指定 ID，例如 `KIMI_001-KIMI_005`。
- `data/raw_reviews.csv` 中尚未出现在 `data/clean_reviews.csv` 的 rows。

## 输出 Schema

```csv
id,source,app_name,review_date,rating,review_text,usage_scene,issue_category,ai_issue_type,user_sentiment,user_need,priority,product_suggestion,notes
```

## 两种模式

### Mode A：预览模式

当用户说“先输出让我检查”时使用。

- 只输出 append-ready clean rows。
- 不写入文件。
- 如果有跳过评论，在 CSV 行后列出 `跳过：id - 原因`。

### Mode B：直接追加模式

只有当用户明确说“直接写入 data/clean_reviews.csv”或“追加到 clean_reviews.csv”时使用。

- 先完成相关性检查和标注。
- 使用 `scripts/append_clean_reviews.py` 追加到 `data/clean_reviews.csv`。
- 保留现有表头和已有行。
- 如果文件不存在，脚本会创建正确表头。
- 如果任一待追加 `id` 已存在，必须停止并提示用户，不追加任何行。
- 不覆盖、不重排已有行。
- 跳过项不写入 clean CSV，只在回复中列出原因。

推荐命令：

```bash
python3 scripts/append_clean_reviews.py
```

脚本从标准输入读取要追加的 clean CSV rows。

## 默认输出规则 / Concise Result Output Rules

默认保持结果导向，避免输出中间过程。

- 直接追加模式下，不输出完整 CSV 内容。
- 不列出所有正常行。
- 不展示 `data/raw_reviews.csv` 或 `data/clean_reviews.csv` 的完整内容。
- 将问题、警告、跳过 ID、需要人工复核的 ID 和需要用户处理的动作放在最前面。
- 回复保持简短，聚焦可执行结果。

直接追加模式写入后只汇报：

- 新增到 `clean_reviews.csv` 的 id。
- 被跳过的 id 及原因。
- 需要人工复核的 id。
- 是否还需要补充评论。

在 direct append mode 下，如果没有跳过项、重复 id 或需要人工复核的评论，不要只回复 `未发现需要处理的问题。`，而应简短汇报：

- 已成功写入 `clean_reviews.csv` 的 id。
- 跳过项：无。
- 需要复核项：无。
- 是否还需要补充评论。

不要输出全部 labeled rows，除非用户明确要求预览模式。不要把问题汇总放到最后；跳过项和问题 ID 必须优先出现。

## 相关性检查

只标注与 AI 应用产品体验相关的评论，包括：

- AI 回答质量、回答不准、幻觉或信息不准确。
- 指令跟随、上下文记忆、多轮对话。
- 搜索、引用和信息可信度。
- 写作、学习、办公、翻译、代码、图片/视频生成等使用场景。
- 新手引导、功能入口、功能发现。
- 性能、速度、崩溃、加载、稳定性。
- 会员、免费次数、付费功能限制，且评论明确提到相关体验。
- 账号、隐私、数据、登录。
- 用户预期或产品定位。

跳过以下评论：

- 无意义或过短，无法分析。
- 只有表情、广告、刷屏、重复内容。
- 与 AI 应用体验无关。
- 只谈手机存储、下载渠道或安装问题，且无法推断产品体验问题。
- 点餐、奶茶、外卖、购物、配送、票务等生活服务评论，且没有明确 AI 助手能力、对话体验或任务执行问题。
- 含私人信息且无法安全清除。
- 重复评论。

跳过项不要写入 `data/clean_reviews.csv`，应返回简短列表：`跳过：id - 原因`。

## 标签取值

`usage_scene`：

- `写作`
- `搜索`
- `学习`
- `办公`
- `聊天`
- `图片/视频`
- `代码`
- `综合/不明确`

`issue_category`：

- `模型能力问题`
- `交互体验问题`
- `性能与稳定性问题`
- `会员与商业化问题`
- `内容安全与合规问题`
- `账号、隐私与数据问题`
- `用户预期与产品定位问题`

`ai_issue_type`：

- `回答不准`
- `幻觉 / 编造信息`
- `指令跟随差`
- `上下文记忆差`
- `搜索引用不足`
- `生成内容质量不稳定`
- `多轮对话体验差`
- `响应速度慢`
- `功能入口难找`
- `会员限制影响核心体验`
- `不明确/其他`

`user_sentiment`：

- `正向`
- `负向`
- `中性`
- `混合`
- `不明确`

`priority`：

- `P0`
- `P1`
- `P2`

## 优先级规则

- `P0`：核心功能不可用，严重崩溃、登录、数据、隐私、支付、安全问题，或高风险误导性 AI 输出。
- `P1`：明确影响核心 AI 使用、新用户激活、留存、信任或重复产品体验。
- `P2`：轻微建议、弱信号、低影响偏好或模糊反馈。

## product_suggestion 规则

- 必须具体。
- 必须与评论内容直接相关。
- 不使用“优化体验”“提升质量”“增强用户粘性”“完善功能”等空泛表达。
- 如果评论太模糊，给出保守建议，并在 `notes` 中说明不确定性。

## 去重和增量规则

- 不重标已经存在于 `data/clean_reviews.csv` 的 `id`，除非用户明确要求。
- 当用户要求“只标注未清洗的新 raw rows”时，应读取 raw 和 clean，选择 raw 中存在但 clean 中不存在的 ID。
- 新发现评论必须先进入 `data/raw_reviews.csv`，再进入 `data/clean_reviews.csv`。

## CSV 输出规则

- 每行严格对齐 clean schema。
- `review_text`、`user_need`、`product_suggestion`、`notes` 使用英文双引号包裹。
- 文本内部英文双引号写成 `""`。
- 默认不包含表头，除非用户明确要求。
- 不使用 Markdown 表格。

## 示例提示词

直接追加模式：

```text
请使用 .agents/skills/02-clean-review-labeling/SKILL.md 的 direct append mode，只标注 data/raw_reviews.csv 中尚未进入 clean_reviews.csv 的评论，并直接写入 data/clean_reviews.csv。写入后只汇报新增 id、跳过 id、需要复核的 id，不要输出全量 CSV 内容。
```
