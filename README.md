# AI 应用用户反馈与产品体验分析

## 项目简介

本项目基于公开应用商店评论，分析真实 AI 应用用户在使用过程中的产品体验问题，并将用户反馈转化为问题分类、优先级判断和可执行的产品改进建议。

分析对象为三款 AI 应用：豆包、DeepSeek、Kimi。项目围绕 AI 产品运营、产品分析、运营分析和数据产品助理岗位常见工作，展示以下能力：

- 从非结构化用户评论中识别真实使用场景和问题信号。
- 区分模型能力、性能稳定性、交互体验、内容安全、数据信任和商业化限制等问题类型。
- 提取用户需求，并判断问题优先级。
- 将评论证据转化为产品建议、运营动作和可观察指标。
- 使用轻量 QA 流程检查人工标注质量。

## 项目背景与问题

AI 应用的用户反馈和传统工具类 App 不完全相同。用户不仅会反馈卡顿、入口难找、限制不清楚等通用体验问题，也会直接指出回答不准、幻觉、上下文记忆差、指令跟随差、搜索可信度不足、生成质量不稳定等 AI 特有问题。

本项目希望回答的问题包括：

- 用户在真实 AI 应用评论中最常反馈哪些产品体验问题？
- 哪些问题属于模型能力，哪些属于交互、性能、内容安全或信任问题？
- 评论中体现了哪些高价值使用场景和用户期待？
- 哪些问题应优先处理，哪些可以作为产品或运营优化机会？

## 数据与样本说明

当前有效样本来自 `data/clean_reviews.csv`，共 30 条，覆盖三款 AI 应用，每款 App 各 10 条。

| 维度 | 样本说明 |
| --- | --- |
| App 范围 | 豆包、DeepSeek、Kimi |
| 样本量 | 30 条有效评论 |
| iOS 来源 | App Store |
| Android 来源 | 应用宝；Kimi 使用小米应用商店作为 Android 侧补充来源 |
| 排除样本 | 5 条千问相关样本已归档到 `data/excluded_reviews.csv` |

样本采集方式为手动公开采样，数据处理过程只保留评论正文、评分、日期、来源等分析所需字段。初始备选应用千问因公开评论中有效 AI 产品体验反馈不足，且部分评论与 AI 应用体验关联较弱，最终未纳入主分析样本。

## 分析流程

1. 选择分析对象：确定豆包、DeepSeek、Kimi 三款 AI 应用。
2. 收集公开评论样本：手动整理 App Store、应用宝和小米应用商店中的公开评论。
3. 录入原始评论：将评论保存到 `data/raw_reviews.csv`，对应工具为 `.agents/skills/01-raw-review-ingest/`。
4. 人工标注样本：按编码指南将原始评论标注为 `data/clean_reviews.csv`，对应工具为 `.agents/skills/02-clean-review-labeling/`。
5. 检查标注质量：检查非法标签、低信息量评论、可疑优先级和模糊建议，对应脚本为 `scripts/label_quality_check.py`，对应工具为 `.agents/skills/03-label-quality-check/`。
6. 汇总分析：基于 `issue_category`、`priority`、`user_need`、`product_suggestion` 输出问题分布和分析报告。
7. 形成产品建议：将高频问题整理为具体动作、预期改善和可观察指标。

## 标签与分析口径

本项目以 `issue_category` 作为主分析层级，避免在小样本中过度拆分标签。

主要问题大类包括：

- 模型能力问题
- 交互体验问题
- 性能与稳定性问题
- 会员与商业化问题
- 内容安全与合规问题
- 账号、隐私与数据问题
- 用户预期与产品定位问题

`ai_issue_type` 作为辅助说明，用于补充回答不准、幻觉、指令跟随差、上下文记忆差、响应速度慢等 AI 体验细节。主要 Top 问题排序依据为 `issue_category`。

`不明确/其他` 在本项目中作为小样本 MVP 的保守标签使用，维持现有大类口径。正向能力样本单独作为用户期待和产品机会证据。

## 核心分析产出

主要报告文件：

- `docs/analysis_report.md`：第一版完整分析报告。
- `docs/coding_guide.md`：评论标注规则和标签口径。
- `docs/qa_report.md`：标注质量检查结果。

当前分析报告中的主要发现包括：

- Top 1 问题大类为模型能力问题，占 36.7%。
- 性能与稳定性问题占 20.0%，在 Kimi 样本中较突出。
- 交互体验问题和用户预期与产品定位问题各占 13.3%。
- P0 高优先级问题主要涉及事实误导、幻觉、内容丢失和闪退。
- 正向样本显示用户认可长上下文、复杂任务处理、模糊搜索、代码辅助求证和情绪价值。

## 项目结构

```text
ai-app-feedback-analysis/
├── README.md
├── data/
│   ├── raw_reviews.csv
│   ├── clean_reviews.csv
│   └── excluded_reviews.csv
├── docs/
│   ├── coding_guide.md
│   ├── qa_report.md
│   └── analysis_report.md
├── scripts/
│   ├── append_raw_reviews.py
│   ├── append_clean_reviews.py
│   ├── label_quality_check.py
│   └── summarize_reviews.py
├── .agents/
│   └── skills/
│       ├── 01-raw-review-ingest/
│       ├── 02-clean-review-labeling/
│       └── 03-label-quality-check/
└── assets/
    └── charts/
        └── .gitkeep
```

## 如何查看

建议按以下顺序阅读：

1. `docs/analysis_report.md`：查看完整分析结论、问题洞察和产品建议。
2. `docs/coding_guide.md`：了解标签定义、标注规则和分析口径。
3. `docs/qa_report.md`：查看标注质量检查结果。
4. `data/clean_reviews.csv`：查看结构化后的有效样本。
5. `scripts/summarize_reviews.py`：查看基础统计脚本。

可在项目根目录运行：

```bash
python3 scripts/label_quality_check.py
python3 scripts/summarize_reviews.py
```

## 项目价值

这个项目模拟了 AI 应用产品与运营分析中的一个常见任务：面对分散、情绪化、非结构化的用户反馈，如何提炼出可讨论、可排序、可跟踪的问题。

它重点展示以下岗位能力：

- AI 产品运营：理解 AI 应用用户体验，识别模型能力和内容安全相关反馈。
- 产品分析：把用户评论转化为问题分类、用户需求和产品改进方向。
- 运营分析：从公开评论中发现高频痛点、正向场景和可跟踪运营动作。
- 数据产品助理：设计数据字段、维护标注口径、建立轻量 QA 和汇总流程。

## 说明与局限

- 样本量较小，目前仅 30 条有效评论，适合用于展示分析方法和初步观察。
- 公开应用商店评论可能偏向强情绪用户、近期版本用户和低评分用户。
- 人工编码存在主观性，尤其是复合问题评论需要选择主要问题大类。
- 本项目无法访问应用内部埋点、留存、转化、任务完成率等数据，因此产品建议属于基于公开反馈的初步分析。
