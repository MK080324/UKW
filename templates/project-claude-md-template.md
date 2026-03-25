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

### 整合审核

所有 Phase 完成后、最终交付前，Lead 调用 QA 对 `output/` 下所有产出物做跨文档整体性审查：
1. Lead 生成 `.agents/test-cases/integration-test-cases.md`
2. QA 检查：论述一致性、重复内容、逻辑衔接、术语一致性、交叉引用准确性
3. 问题写入 `.agents/issues/integration/`，由对应 Writer 修复
4. 通过后在 phase 文件写入 `C_INTEGRATION_REVIEWED`

### 补充调研流程

Writer Agent 禁止自行使用 WebSearch/WebFetch。当 Writer 发现需要补充信息时：

1. Writer 通过 SendMessage 向 Lead 发送调研请求（说明需要什么信息、用于哪个章节）
2. Lead 自行使用 WebSearch/WebFetch 完成调研
3. Lead 将调研结果追加到 `docs/supplementary-research.md`（如不存在则创建），格式：
   ```markdown
   ## [请求主题]
   - 请求来源: [Writer Agent 名称]
   - 请求时间: YYYY-MM-DD
   - 调研结果:
     [调研内容，附来源 URL]
   ```
4. Lead 通过 SendMessage 回复 Writer，告知结果已写入，Writer 从文件中读取

### 检查点与恢复机制

每个 Phase 审核通过后，Lead 更新 `.agents/status/phase` 文件。检查点值：

```
C_PHASE_0_COMPLETED        -> 基础框架已通过
C_PHASE_1_COMPLETED        -> Phase 1 已通过
C_PHASE_N_COMPLETED        -> Phase N 已通过
C_INTEGRATION_REVIEWED     -> 整合审核已通过
C_COMPLETED                -> 项目完成
```

**恢复逻辑：** session 中断后，新 session 启动时读取 phase 文件：
- `C_PHASE_N_COMPLETED`：跳到 Phase N+1 继续。如果 N 是最后一个 Phase，进入整合审核
- `C_INTEGRATION_REVIEWED`：整合审核已通过，直接进入最终交付
- `C_COMPLETED`：项目已完成，告知人类

## 验收标准速查
| Phase | 自动审核（阻塞） | 延迟人工审阅（不阻塞） |
|-------|----------------|---------------------|

## 参考文档
- 需求文档：docs/requirement.md
- 主题调研：docs/topic-research.md
- 分析结构：docs/analysis-structure.md
- 产出物规划：docs/deliverables-plan.md
