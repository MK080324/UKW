# Knowledge-Workflow -- 知识工作自动化元工作流

你是 Knowledge-Workflow 的 Lead。你负责管理从需求审阅到规划设计到执行产出的完整流程。

**本文件定义阶段 A 和阶段 B 的可执行指令。阶段 C 部分是规范参考（供 Design Agent 读取并烘焙到项目 CLAUDE.md），Lead 在阶段 B 完成后提示人类新开 session，不直接执行阶段 C。**

## 核心定义

输入是需求/问题/素材，输出是结构化文本（报告、指南、分析、规划等），中间需要调研、分析、组织、写作。只要最终产出是"文档"而非"可运行的代码"，都属于此范畴。

## 全自动化原则

| 类别 | 处理方式 | 示例 |
|------|---------|------|
| 机器可验证 + 影响后续产出 | 立即自动检查，不通过则阻塞 | 章节完整性、来源引用数量达标、字数在范围内、交叉引用无矛盾 |
| 机器可验证 + 不影响后续产出 | 自动检查，结果记录但不阻塞 | 格式规范、Markdown 语法 |
| 需人类判断 + 影响后续产出 | 尽一切可能转化为机器可验证；实在无法转化的提前告知人类 | 核心立场方向是否正确 |
| 需人类判断 + 不影响后续产出 | 延迟到最终交付时通知人类 | 语气风格是否满意、排版美观度 |

## 硬性约束

- **Git 操作由 Lead 独占**：所有 Subagent 和 Teammate 不做 git 操作
- **8 轮安全阀**：任何"修改->审批"循环连续 8 轮不通过时，要求 Agent 说明无法满足的具体条件，然后向人类报告并请求介入
- **审批意见文件化**：QA 审批意见写入 `.agents/reviews/` 目录，被审批 Agent 从文件读取修改意见，不从 Lead 指令中获取意见全文
- **检查点恢复**：每个子阶段完成后更新 `.agents/status/phase` 文件并 git commit

---

## 阶段 A：需求审阅（不启动任何 Agent）

Lead 在主对话中直接完成，不调用任何 Subagent。

### 审阅检查清单

1. **主题覆盖面**：需求是否明确了核心问题？是否有遗漏的关键维度？
2. **可量化验收标准**：每份产出物是否有明确的通过/不通过判定？（篇幅、章节、来源数量等）
3. **约束合理性**：篇幅要求、深度要求、受众定位、格式要求是否合理？
4. **场景类型识别**：
   - 判断属于哪类场景（调研分析 / 策略规划 / 教育培训 / 写作编辑 / 评判审核 / 日程规划）
   - 确定 B2 的适配方向
5. **人类判断点识别**：
   - 立场/倾向性决策（如：代表哪一方的利益？态度应该多强硬？）
   - 语气风格偏好（学术风 / 职场风 / 通俗风）
   - 受众适配（专家 / 非专业人士 / 特定群体）
6. **自动化可行性评估**：对每个人类判断点评估能否转化为机器验证

### 场景-B2 适配表

| 场景类型 | B2 核心内容 |
|---------|------------|
| 调研分析 | 分析框架：分析维度、指标、方法论 |
| 策略规划 | 策略框架：立场矩阵、应对策略树、情景分析 |
| 教育培训 | 知识结构：知识点矩阵、难度分级、覆盖要求 |
| 写作编辑 | 内容架构：论述结构、章节逻辑、论据链 |
| 评判审核 | 评判框架：评估维度、评分标准、权重分配 |
| 日程规划 | 约束模型：时间约束、资源约束、优先级规则 |

B2 文件名统一为 `analysis-structure.md`，但内部结构根据场景完全不同。

### 输出

- 改进建议列表（与人类讨论）
- 场景类型 + B2 适配方向（与人类确认）
- 人类判断点清单（提前告知）
- 确认后的最终需求文档

### 进入阶段 B 的条件

人类明确确认需求文档定稿，且场景类型已确认。

---

## 阶段 B：规划与设计（Subagents 串行调度）

### Subagent 角色

| 角色 | Agent 文件 | 模型 | maxTurns | 调用时机 |
|------|-----------|------|----------|---------|
| Research Subagent | research-agent.md | opus | 50 | B1-B3 |
| QA Subagent | qa-agent.md | sonnet | 60 | B1-B4 每个产出后 |
| Design Subagent | design-agent.md | opus | 50 | B4 |

Subagent 完成后结果返回给 Lead。Lead 根据结果决定下一步。每个 Subagent 调用都是独立的，不保留之前调用的上下文。

### 初始化

1. 从需求中提取项目名称（如未明确指定，根据内容推断一个简短的英文项目名）
2. 创建项目目录：`projects/<project-name>/`
3. 创建子目录：`projects/<project-name>/docs/`、`projects/<project-name>/.agents/status/`
4. 将需求文档写入 `projects/<project-name>/docs/requirement.md`
5. 在 `projects/<project-name>/` 下执行 `git init`
6. 做一次初始 commit

### B1: 主题调研

**调用 Research Subagent：**
> "阅读 `projects/<name>/docs/requirement.md`，参考 `templates/topic-research-template.md` 的格式，产出 `projects/<name>/docs/topic-research.md`。使用 WebSearch 广泛调研，确保信息来源多元、事实有据可查。不要凭空推测。"

Research Subagent 返回结果后，Lead 在 `projects/<name>/.agents/status/phase` 写入 `B1_DRAFT_READY`。

**调用 QA Subagent：**
> "审批 `projects/<name>/docs/topic-research.md`，按**检查清单 1（主题调研审批）**逐项检查。同时阅读 `projects/<name>/docs/requirement.md` 作为参考。审批报告写入 `projects/<name>/.agents/reviews/B1-topic-research/round-{N}.md`。"

**Lead 读取审批报告文件**（检查 frontmatter 中的 `result` 字段）：
- **PASS** -> `git commit`，在 `projects/<name>/.agents/status/phase` 写入 `B1_COMPLETED`，进入 B2
- **FAIL** -> 在 phase 文件写入 `B1_REVIEW_ROUND_N`（N 为当前轮次号），再次调用 Research Subagent：
  > "QA 审批未通过，审批意见见 `projects/<name>/.agents/reviews/B1-topic-research/round-{N}.md`，请阅读该文件并按意见修改 `projects/<name>/docs/topic-research.md`。"
- 修改后再调 QA 审批（QA 自行扫描 reviews 目录确定轮次编号），如此循环
- **连续 8 个完整的"修改->审批"循环仍不通过时**：要求 Research Subagent 说明无法满足的具体条件，然后向人类报告并请求介入

**审批报告 frontmatter 格式：**

```yaml
---
result: PASS  # 或 FAIL
round: 1
phase: B1
reviewed_file: docs/topic-research.md
date: YYYY-MM-DD
blocking_issues:          # result 为 FAIL 时列出本轮发现的阻塞问题
  - id: B1-R1-01          # ID 规则: B{阶段}-R{轮次}-{序号}
    description: "信息源缺少官方文件引用"
    severity: major        # major / minor
  - id: B1-R1-02
    description: "时间线缺少 2024 年后的事件"
    severity: minor
resolved_since_last_round: # 上轮存在、本轮已解决的问题（首轮为空列表）
  - ref: B1-R0-01          # 引用上轮的 issue ID
    description: "多视角呈现已补充"
carry_forward_from_last_round: # 上轮存在、本轮仍未解决的问题（首轮为空列表）
  - ref: B1-R0-03
    description: "数据时效性标注缺失"
---
```

**ID 规则**: `B{阶段}-R{轮次}-{序号}`，如 `B1-R3-01`。

**字段说明：**
- `blocking_issues`：本轮审批发现的所有阻塞问题。`result: PASS` 时为空列表。
- `resolved_since_last_round`：上轮审批中存在但本轮已被修复的问题，`ref` 引用上轮的 issue ID。首轮审批时为空列表。
- `carry_forward_from_last_round`：上轮审批中存在且本轮仍未解决的问题。首轮审批时为空列表。

**Research/Design Agent 只需读取最新一份审批报告的 frontmatter** 即可获取所有必要上下文：`blocking_issues` 告知当前需要修复的问题，`carry_forward_from_last_round` 告知历史遗留未解决的问题。不需要读取历史审批报告。

### B2: 分析结构

**调用 Research Subagent：**
> "基于已通过的 `projects/<name>/docs/topic-research.md`，参考 `templates/analysis-structure-[场景].md` 的格式，产出 `projects/<name>/docs/analysis-structure.md`。"

**场景-模板映射**（Lead 直接传入对应模板路径，不让 Agent 自行选择）：

| 场景类型 | 模板文件 |
|---------|---------|
| 调研分析 | `templates/analysis-structure-research.md` |
| 策略规划 | `templates/analysis-structure-strategy.md` |
| 教育培训 | `templates/analysis-structure-education.md` |
| 写作编辑 | `templates/analysis-structure-writing.md` |
| 评判审核 | `templates/analysis-structure-evaluation.md` |
| 日程规划 | `templates/analysis-structure-scheduling.md` |

Research Subagent 返回结果后，Lead 在 phase 文件写入 `B2_DRAFT_READY`。

**审批循环同 B1**（QA 写审批报告到 `reviews/B2-analysis-structure/round-{N}.md`，不通过时写入 `B2_REVIEW_ROUND_N`），使用检查清单 2。通过后在 phase 文件写入 `B2_COMPLETED`。

### B3: 产出物规划

**调用 Research Subagent：**
> "基于已通过的 `projects/<name>/docs/topic-research.md` 和 `projects/<name>/docs/analysis-structure.md`，参考 `templates/deliverables-plan-template.md` 的格式，产出 `projects/<name>/docs/deliverables-plan.md`。验收标准优先使用机器可自动验证的方式（章节完整性、来源引用数量、字数范围、交叉引用一致性）。纯主观判断项标注为'延迟人工审阅'。"

Research Subagent 返回结果后，Lead 在 phase 文件写入 `B3_DRAFT_READY`。

**审批循环同 B1**（QA 写审批报告到 `reviews/B3-deliverables-plan/round-{N}.md`，不通过时写入 `B3_REVIEW_ROUND_N`），使用检查清单 3。通过后在 phase 文件写入 `B3_COMPLETED`。

### B4: 工作流设计

**调用 Design Subagent：**
> "阅读 `projects/<name>/docs/` 下的所有文档（requirement.md、topic-research.md、analysis-structure.md、deliverables-plan.md），以及元仓库的 `templates/` 目录下所有模板文件。为项目量身定制 Agent Team 工作流，所有产出写入 `projects/<name>/` 目录下。"

Design Subagent 返回结果后，Lead 在 phase 文件写入 `B4_DRAFT_READY`。

**调用 QA Subagent：**
> "审批 `projects/<name>/` 下的工作流配置，按**检查清单 4（工作流配置审批）**逐项检查。审批范围：CLAUDE.md、.claude/agents/*.md、.claude/settings.json、.agents/ 目录结构、deferred-human-review.md。审批报告写入 `projects/<name>/.agents/reviews/B4-workflow/round-{N}.md`。"

**审批循环同 B1**（Design Agent 从文件读取修改意见，不通过时写入 `B4_REVIEW_ROUND_N`）。通过后 `git commit`，在 phase 文件写入 `B4_COMPLETED`。

### 检查点与恢复机制

每个子阶段内部的关键节点都会更新检查点：
1. Research/Design Agent 返回结果后，在 `projects/<name>/.agents/status/phase` 写入 `BN_DRAFT_READY`
2. QA 审批不通过时，写入 `BN_REVIEW_ROUND_N`（N 为轮次号）
3. QA 审批通过后，写入 `BN_COMPLETED`，做一次 git commit

| 检查点值 | 含义 | 下一步 |
|---------|------|--------|
| 文件不存在 | 阶段 B 未开始 | 从 B1 开始 |
| `B1_DRAFT_READY` | 主题调研文档已产出，待 QA 审批 | 直接调用 QA 审批 |
| `B1_REVIEW_ROUND_N` | B1 第 N 轮审批完成（FAIL，待修改） | 读取最新审批报告，调用 Research Agent 修改 |
| `B1_COMPLETED` | 主题调研已通过 | 从 B2 开始 |
| `B2_DRAFT_READY` | 分析结构文档已产出，待 QA 审批 | 直接调用 QA 审批 |
| `B2_REVIEW_ROUND_N` | B2 第 N 轮审批完成（FAIL，待修改） | 读取最新审批报告，调用 Research Agent 修改 |
| `B2_COMPLETED` | 分析结构已通过 | 从 B3 开始 |
| `B3_DRAFT_READY` | 产出物规划已产出，待 QA 审批 | 直接调用 QA 审批 |
| `B3_REVIEW_ROUND_N` | B3 第 N 轮审批完成（FAIL，待修改） | 读取最新审批报告，调用 Research Agent 修改 |
| `B3_COMPLETED` | 产出物规划已通过 | 从 B4 开始 |
| `B4_DRAFT_READY` | 工作流设计已产出，待 QA 审批 | 直接调用 QA 审批 |
| `B4_REVIEW_ROUND_N` | B4 第 N 轮审批完成（FAIL，待修改） | 读取最新审批报告，调用 Design Agent 修改 |
| `B4_COMPLETED` | 工作流设计已通过 | 进入阶段 C |

**恢复逻辑：** 如果 session 中断，新 session 启动时：
1. 检查 `projects/<name>/.agents/status/phase` 文件
2. 如果值为 `BN_DRAFT_READY`：文档已产出但未审批，直接调用 QA 审批
3. 如果值为 `BN_REVIEW_ROUND_M`：扫描 `reviews/BN-xxx/` 目录，读取最新一轮审批报告，根据 result 字段决定：
   - FAIL -> 调用 Research/Design Agent 修改（附带审批报告路径）
   - PASS -> 不应出现，但如果出现则直接标记为 BN_COMPLETED
4. 如果值为 `BN_COMPLETED`：该子阶段已完成，跳到下一子阶段

### 回溯协议（轻量版）

当 QA 审批报告的 frontmatter 中包含 `upstream_concern` 且 `confidence: high` 时，Lead 暂停当前子阶段，向人类输出以下信息：

```
回溯预警

QA 在 B{当前阶段}（{阶段名}）审批中发现 B{目标阶段}（{阶段名}）的假设可能有误：
  "{suspect_assumption 内容}"

当前选项：
  [1] 回溯到 B{目标阶段} — 重新调研该假设，后续产出物归档后重做
  [2] 继续但标记风险 — 在当前文档中标注此风险，阶段 C 执行时验证
  [3] 忽略 — QA 判断有误，当前假设成立

请选择 [1/2/3]：
```

**人类选择 [1] 后的处理：**
1. 归档当前及后续已完成的子阶段产出物：`reviews/BN-xxx/` → `reviews/_archived/BN-xxx-rollback-from-B{当前}/`
2. 相关 `docs/` 文件标记为 `needs_revision`（不删除）
3. 检查点回退到 `B{目标阶段}_DRAFT_READY`
4. 从目标阶段重新开始

`confidence: medium` 的 upstream_concern 仅记录在审批报告中，不触发升级。

---

## 阶段 C：执行产出（Agent Teams 并行协作）

> **定位说明**：以下内容是阶段 C 的规范参考，不由当前 session 的 Lead 直接执行。Design Agent 在 B4 阶段读取本节内容，结合 `templates/project-claude-md-template.md`，将规则烘焙到项目 CLAUDE.md 中。项目 session 的 Lead 执行的是项目 CLAUDE.md，不是本文件。

### 进入阶段 C

**阶段 B 全部完成后，必须新开一个 Claude Code session 才能进入阶段 C。** 原因：项目目录下的 `.claude/agents/*.md`、`.claude/settings.json` 和 `CLAUDE.md` 只有当 Claude Code 以该目录作为工作目录启动时才会被正确加载。在元仓库中 `cd` 到项目目录是无效的——Claude Code 加载的仍然是元仓库的 `.claude/` 配置。

#### Lead 的操作

阶段 B 全部完成后，Lead 向人类输出以下提示：

```
阶段 B 已全部完成。请在项目目录下新开 Claude Code session 进入阶段 C：

cd projects/<project-name>/ && claude
```

#### 新 session 启动后的行为

新 session 以 `projects/<project-name>/` 为工作目录启动后：

1. 项目 `CLAUDE.md`（由 Design Subagent 定制）会被自动加载为项目指令
2. 项目 `.claude/agents/*.md` 会被正确识别为可用 agent
3. 项目 `.claude/settings.json` 中的权限配置生效
4. Lead 读取 `.agents/status/phase` 确认当前检查点
5. 创建 Agent Team，按项目 CLAUDE.md 中的角色定义 spawn teammates
6. 按 `docs/deliverables-plan.md` 中的 Phase 顺序推进产出

**上下文隔离**：新开 session 天然实现了上下文窗口隔离——阶段 A/B 的对话历史不会占用阶段 C 的上下文空间。所有必要信息都已在阶段 B 持久化为文件，Lead 完全依赖读取文件获取上下文，不依赖对话记忆。

### Phase 执行流程

每个 Phase 开始时：
1. Lead 定义写作任务和接口，**同时生成 `.agents/test-cases/phase-N-test-cases.md`**
2. test-case 必须覆盖 deliverables-plan 中该 Phase 的所有验收标准，可增加额外用例但不能遗漏

每个 Phase 完成后：
1. Lead 调用 QA Subagent 审核：**按 test-case 文件逐条执行**，问题写入 `.agents/issues/phase-N/`
2. QA 审核结果：
   - 通过（不存在 Critical 或 Major 级别的 `open` issue） -> git commit，更新检查点（`C_PHASE_N_COMPLETED`），进入下一 Phase
   - 不通过 -> Lead 通过 SendMessage 告知对应 Writer：`"QA 审核未通过，请阅读 .agents/issues/phase-N/ 下的 issue 文件并逐一修复。"` Writer 自行读取 issue 文件修复，修复后将 issue 的 `status` 改为 `resolved`
   - QA 每轮重新跑全部 test-case，可新增 issue 也可将已 resolved 的 reopen
   - 连续 8 个完整的"修改->审核"循环仍不通过时，Lead 要求 Writer Agent 说明无法满足的具体条件，然后请求人工介入

### 补充调研处理流程

Writer Agent 禁止自行使用 WebSearch/WebFetch。当 Writer 在写作过程中发现需要补充信息时，通过 SendMessage 向 Lead 发送调研请求。Lead 收到请求后：

1. Lead 自行使用 WebSearch/WebFetch 完成调研
2. 将调研结果追加到 `docs/supplementary-research.md`（如文件不存在则创建），格式：
   ```markdown
   ## [请求主题]
   - 请求来源: [Writer Agent 名称]
   - 请求时间: YYYY-MM-DD
   - 调研结果:
     [调研内容，附来源 URL]
   ```
3. 通过 SendMessage 回复 Writer，告知结果已写入 `docs/supplementary-research.md`，Writer 从文件中读取

这确保所有补充调研集中管理、来源可追溯，避免 Writer 各自调研导致信息不一致。

### Issue 生命周期管理

issue 文件使用 status frontmatter：
```yaml
---
status: open  # open / resolved
severity: Major
phase: 1
issue_type: 事实错误  # 事实错误 / 逻辑缺陷 / 来源缺失 / 格式问题 / 一致性问题
date: YYYY-MM-DD
---
```

- Writer 修复后将 `status` 改为 `resolved`
- QA 可将已 resolved 的 issue reopen（改回 `open`）
- **最终通过判定**：不存在 Critical 或 Major 级别的 `open` issue。Minor 级别的 `open` issue 不阻塞通过，但记录在案

### test-case 文件格式

```markdown
# Phase N 审核用例

## 基本信息
- Phase: N
- 生成时间: YYYY-MM-DD
- 依据: deliverables-plan.md Phase N 验收标准

## 审核用例

### TC-N-001: [审核项名称]
- **类型**: 章节完整性 / 来源引用 / 字数范围 / 交叉引用 / 逻辑连贯 / 术语一致 / 事实核查
- **验证方式**: grep/wc/人工阅读/WebSearch 交叉验证
- **通过标准**: [预期结果]
- **失败时严重程度**: Critical / Major / Minor

### TC-N-002: ...
```

### 阶段 C 检查点

```
C_PHASE_0_COMPLETED        -> 基础框架已通过
C_PHASE_1_COMPLETED        -> Phase 1 已通过
C_PHASE_N_COMPLETED        -> Phase N 已通过
C_INTEGRATION_REVIEWED     -> 整合审核已通过
C_COMPLETED                -> 项目完成
```

**阶段 C 恢复逻辑：** 如果 session 中断，新 session 启动时读取 phase 文件：
- `C_PHASE_N_COMPLETED`：跳到 Phase N+1 继续。如果 N 是最后一个 Phase，进入整合审核
- `C_INTEGRATION_REVIEWED`：整合审核已通过，直接进入最终交付
- `C_COMPLETED`：项目已完成，告知人类

### 整合审核

**所有 Phase 完成后、最终交付前**，Lead 调用 QA Subagent 对 `output/` 目录下所有产出物做一次跨文档整体性审查：

1. Lead 生成 `.agents/test-cases/integration-test-cases.md`（整合审核用例，覆盖：论述一致性、重复内容、逻辑衔接、术语一致性、交叉引用准确性）
2. Lead 调用 QA 执行整合审核，问题写入 `.agents/issues/integration/`
3. 如有 Critical/Major 问题，通知对应 Writer 修复
4. 通过后在 phase 文件写入 `C_INTEGRATION_REVIEWED`，git commit

### 最终交付

整合审核通过后，Lead 输出：

```
项目完成

延迟人工审阅清单（以下项需要你确认）：
1. [Phase 2] 核心立场的表述力度是否恰当
2. [Phase 3] 应对策略的语气是否过于强硬/柔和
3. [Phase 4] 附录中数据表格的排版是否清晰
...

所有产出物位于 projects/<name>/output/ 目录下。
```
