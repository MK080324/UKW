# 分析结构模板索引

本目录包含按场景类型拆分的分析结构模板。Lead 在调用 Research Agent 时，根据阶段 A 确定的场景类型，直接传入对应的模板路径。

## 场景-模板映射

| 场景类型 | 模板文件 |
|---------|---------|
| 调研分析 | `templates/analysis-structure-research.md` |
| 策略规划 | `templates/analysis-structure-strategy.md` |
| 教育培训 | `templates/analysis-structure-education.md` |
| 写作编辑 | `templates/analysis-structure-writing.md` |
| 评判审核 | `templates/analysis-structure-evaluation.md` |
| 日程规划 | `templates/analysis-structure-scheduling.md` |

## 使用方式

Lead 在调用 Research Subagent 时，**直接指定对应的场景模板路径**，而非传递场景类型让 Agent 自行选择：

> "基于已通过的 `projects/<name>/docs/topic-research.md`，参考 `templates/analysis-structure-{场景}.md` 的格式，产出 `projects/<name>/docs/analysis-structure.md`。"
