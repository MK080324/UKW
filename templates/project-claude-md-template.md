# [项目名称] -- 项目指令

## 项目概述
<!-- 从需求文档提炼的项目简述 -->

## 分析结构摘要
<!-- 从 analysis-structure.md 提炼，让所有 Agent 快速理解整体框架 -->

## 写作范围划分
<!-- 明确哪个 Agent 负责哪些文档/章节，确保不重叠 -->
- `output/doc-1.md` -- [Writer 1] 负责
- `output/doc-2.md` -- [Writer 2] 负责

## 写作规范
<!-- 引用 .agents/style-guide/ 中的规范 -->
- 语气风格：见 .agents/style-guide/tone.md
- 术语表：见 .agents/style-guide/glossary.md
- 引用格式：见 .agents/style-guide/citation.md

## Agent Teams 工作流

### 角色
| 角色 | 职责 | 写作范围 |
|------|------|---------|
| **Lead** | 任务分配、Phase 推进、QA 调度、git 操作 | 全局 |
| **[Writer 1]** | ... | `output/...` |
| **[Writer 2]** | ... | `output/...` |
| **QA** (Subagent) | 内容审核：事实核查、逻辑审查、来源验证 | 全局（由 Lead 按需调用） |

### 流程
每个 Phase 按以下步骤执行：
1. **Lead 定义任务、质量标准和审核用例** -- 同时写入 .agents/test-cases/phase-N-test-cases.md
2. **并行写作** -- Writer Agents 并行撰写各自部分（可参考 test-case 理解验收预期）
3. **协调** -- 如需引用其他 Agent 产出的内容，通过 SendMessage 协调
4. **完成通知** -- Writer 通过 SendMessage 向 Lead 发送完成通知
5. **QA 审核** -- Lead 调用 QA Subagent，按 test-case 文件逐条执行，问题写入 .agents/issues/phase-N/
6. **问题修复** -- 不通过时，Lead 通过 SendMessage 告知 Writer 阅读 .agents/issues/phase-N/ 下的 issue 文件自行修复，修复后标记 status: resolved
7. **重新审核** -- QA 重新跑全部 test-case，可新增 issue 或 reopen 旧的
8. 连续 8 个完整"修改->审核"循环不通过时，请求人工介入
9. 通过（无 Critical/Major 级别 open issue） -> git commit -> 更新检查点 -> 下一 Phase

### Git 操作约定
- Lead 负责所有 git 操作
- Writer Agent 和 QA 不做 git 操作
- 每个 Phase 审核通过后，Lead 做一次 commit

### 检查点机制
每个 Phase 审核通过后，Lead 更新 `.agents/status/phase` 文件。
session 中断后从上次完成的 Phase 之后继续。

## 验收标准速查
| Phase | 自动审核（阻塞） | 延迟人工审阅（不阻塞） |
|-------|----------------|---------------------|

## 参考文档
- 需求文档：docs/requirement.md
- 主题调研：docs/topic-research.md
- 分析结构：docs/analysis-structure.md
- 产出物规划：docs/deliverables-plan.md
