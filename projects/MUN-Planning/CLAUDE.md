# MUN-Planning -- HFIMUNC 2026 安理会中国代表参会指南

## 项目概述

为 HFIMUNC 2026 安理会（中国代表）定制全套参会指南，包含 5 份文档（D1-D5），总计约 3.2-4.2 万字。目标受众是有 MUN 经验但 3 年未参加的代表，需要唤醒程序记忆并提供 2014 年乌克兰危机的策略深度。会议设定为 2014 年历史模拟，安理会 9 席配置（6 票表决权 + 3 受邀），中国作为 P5 常任理事国的否决权选择是全场博弈核心。

## 分析结构摘要

- **核心立场**：尊重主权与领土完整（P1）、政治解决与外交优先（P2）、不干涉内政（P3）
- **立场张力**：P1 vs. 克里米亚自决权、P2 vs. 安理会僵局行动压力、P3 vs. 中俄关系
- **阵营格局**：西方 4 票（美英法韩）vs. 中俄 2 票 + 3 受邀（EU/德/乌）
- **绝对红线**：R1 不承认克里米亚公投合法性、R2 不公开谴责俄罗斯为侵略者、R3 不支持安理会制裁/军事授权
- **节奏策略**：先守后攻（S1-S2 防守 -> S3-S4 主动 -> S5-S6 收官），MH17 是赞成票转折点
- **可争取对象**：法国（最高优先级）、德国（高）、俄罗斯协调（高）

## 写作范围划分

| Agent | 负责文档 | Phase |
|-------|---------|-------|
| Writer-Procedural | `output/D1-procedure-guide.md`（正文+附录A/B/C） | P1, P4（术语统一+导航索引） |
| Writer-Research | `output/D4-issue-research.md`（正文+附录J/K/L） | P2, P4（来源审校） |
| Writer-Strategy | `output/D5-china-strategy.md`（正文+附录M/N/O/P）、`output/D2-negotiation-guide.md`（正文+附录D/E/F） | P2（D5）, P3（D2） |
| Writer-Language（由 Lead 临时 spawn） | `output/D3-diplomatic-phrases.md`（正文+附录G/H/I） | P3 |

**说明**：Writer-Strategy 横跨 P2 和 P3，串行执行，确保 D5 策略核心先定稿，D2 磋商指南以 D5 为基准。Writer-Language 仅在 P3 阶段需要，由 Lead 在 P3 启动时 spawn。

## 写作规范

所有 Writer Agent 必须严格遵守以下规范：
- 语气风格：`.agents/style-guide/tone.md`
- 术语表：`.agents/style-guide/glossary.md`
- 引用格式：`.agents/style-guide/citation.md`

## Agent Teams 工作流

### 角色

| 角色 | 类型 | 职责 |
|------|------|------|
| **Lead** | -- | 任务分配、Phase 推进、QA 调度、git 操作、交叉一致性检查 |
| **Writer-Procedural** | teammate | D1 参会流程与程序指南；P4 术语统一和导航索引 |
| **Writer-Research** | teammate | D4 议题深度调研报告；P4 来源引用审校 |
| **Writer-Strategy** | teammate | D5 中国立场与策略指引（P2）；D2 磋商与策略指南（P3） |
| **QA** | subagent | 内容审核：事实核查、逻辑审查、来源验证（由 Lead 在每个 Phase 完成后调用） |

### 流程

每个 Phase 按以下步骤执行：

1. **Lead 定义任务和审核用例** -- 读取 `docs/deliverables-plan.md` 中该 Phase 的验收标准，生成 `.agents/test-cases/phase-N-test-cases.md`
2. **分配写作任务** -- Lead 通过 SendMessage 向对应 Writer 发送任务指令，包含：负责文档路径、章节清单、篇幅要求、关键依赖
3. **并行写作**（如该 Phase 支持并行） -- Writer Agents 各自撰写负责的部分
4. **完成通知** -- Writer 通过 SendMessage 向 Lead 报告完成
5. **Lead 交叉检查**（如该 Phase 有多个并行文档） -- Lead 执行文档间一致性检查（T2.5/T3.5）
6. **QA 审核** -- Lead 调用 QA Subagent，按 `.agents/test-cases/phase-N-test-cases.md` 逐条执行，问题写入 `.agents/issues/phase-N/`
7. **问题修复** -- 不通过时，Lead 通过 SendMessage 告知 Writer 阅读 `.agents/issues/phase-N/` 下的 issue 文件自行修复，修复后将 issue 的 `status` 改为 `resolved`
8. **重新审核** -- QA 重新跑全部 test-case，可新增 issue 或 reopen 旧的
9. 连续 8 个完整"修改->审核"循环不通过时，Lead 要求 Writer 说明无法满足的具体条件，然后请求人工介入
10. 通过（无 Critical/Major 级别 open issue）-> Lead 执行 git commit -> 更新检查点 -> 下一 Phase

### Phase 执行计划

#### Phase 0: 基础框架（Lead 独立完成）

**任务清单**：
1. 创建 `output/` 目录结构
2. 复核 `.agents/style-guide/`，如有必要微调
3. 创建术语表初稿 `output/glossary.md`，定义全文档共用的中英术语映射（至少 30 个条目）
4. 为 D1-D5 各创建骨架文件（标题 + 章节框架 + 篇幅标注）
5. 生成 `.agents/test-cases/phase-0-test-cases.md`
6. 调用 QA 审核

**自动验收（阻塞）**：
- `output/` 下存在 D1-D5 骨架文件和 glossary.md
- 每个骨架文件包含规划中列出的所有章节标题（含附录）
- glossary.md 包含至少 30 个术语条目
- `.agents/test-cases/phase-0-test-cases.md` 存在

#### Phase 1: 基础设施文档（D1，不并行）

**执行者**：Writer-Procedural
**任务**：T1.1 撰写 D1 正文第 1-5 章 / T1.2 撰写附录 A/B/C / T1.3 中英对照术语标注 / T1.4 更新 glossary.md

**自动验收（阻塞）**：
- D1 包含全部 5 个正文章节 + 3 个附录（grep 标题）
- 正文 3000-4000 字（wc）
- 附录 2000-3000 字（wc）
- 程序性动议速查表包含至少 15 个中英对照动议（grep）
- 安理会特殊规则章节至少引用 3 个权威来源（grep）
- 明确提及本会场 6 票表决权 + 3 受邀配置（grep）
- D1 中专业术语均在 glossary.md 中有条目（grep 对比）

#### Phase 2: 事实基础 + 策略核心（D4 + D5 并行）

**执行者**：Writer-Research（D4） + Writer-Strategy（D5），并行写作
**并行安全性**：两者事实来源均直接引用 `docs/topic-research.md`，不互相依赖
**Lead 任务**：两份文档均完成后执行 T2.5 交叉引用一致性检查

**D4 自动验收（阻塞）**：
- 6 个正文章节 + 3 个附录（grep）
- 正文 5000-6000 字 / 附录 3000-4000 字（wc）
- 全文至少 25 个不同来源引用（grep）
- 时间线覆盖全部关键事件：Euromaidan、亚努科维奇罢免、克里米亚公投/吞并、东乌冲突爆发、日内瓦声明、MH17、明斯克议定书（grep）
- 各方立场章节覆盖全部 9 个代表团（grep）
- 至少 4 个争议点（grep）
- 术语与 glossary.md 一致（grep）

**D5 自动验收（阻塞）**：
- 7 个正文章节 + 4 个附录（grep）
- 正文 5000-6000 字 / 附录 3000-4000 字（wc）
- 三项核心原则 P1/P2/P3 全覆盖（grep）
- 三条绝对红线 R1/R2/R3 全覆盖及应对预案（grep）
- 策略树 5 个议题分支 A-E 全覆盖（grep）
- 全部 6 个 Session 策略规划（grep）
- 双边互动覆盖全部 8 个其他代表团（grep）
- 最佳/最可能/最坏三个情景分析（grep）
- 至少 15 个不同来源引用（grep）

**D4-D5 交叉一致性验收（阻塞）**：
- 同一事实描述不矛盾（投票记录、日期、人名）
- 关键术语使用一致

#### Phase 3: 战术工具文档（D2 + D3 并行）

**执行者**：Writer-Strategy（D2） + Writer-Language（D3，P3 启动时 spawn），并行写作
**并行安全性**：两者均以 D5（已定稿）为策略基准，不互相参照未定稿内容
**Lead 任务**：两份文档均完成后执行 T3.5 交叉引用一致性检查

**D2 自动验收（阻塞）**：
- 6 个正文章节 + 3 个附录（grep）
- 正文 4000-5000 字 / 附录 2000-3000 字（wc）
- 会场格局速览包含全部 9 个代表团（grep）
- 节奏规划覆盖全部 6 个 Session（grep）
- 投票策略、合作优先级与 D5 不矛盾（grep 对比）
- 最佳/最可能/最坏三个情景（grep）

**D3 自动验收（阻塞）**：
- 5 个正文章节 + 3 个附录（grep）
- 正文 3000-4000 字 / 附录 2000-3000 字（wc）
- 全文至少 50 个中英对照话术表达（grep）
- 红线应对话术覆盖 analysis-structure 第 5.3 节全部 4 个被追问场景（grep）
- 包含 Euromaidan、克里米亚公投、MH17、明斯克议定书的专用表达（grep）
- 话术不违反 D5 红线定义（grep 对比）

**D2-D3-D5 交叉一致性验收（阻塞）**：
- D2 互动策略与 D3 话术建议不矛盾
- D2/D3 均未违反 D5 红线定义
- 术语与 glossary.md 一致

#### Phase 4: 交叉审校与统一（不并行）

**执行者**：Writer-Procedural（T4.1 术语统一 + T4.4 导航索引）、Writer-Research（T4.3 来源审校）、Lead（T4.2 交叉引用 + T4.5 glossary 定稿）

**自动验收（阻塞项目完成）**：
- D1-D5 全文档术语与 glossary.md 一致（grep）
- 所有"参见 D* 第 * 章"类交叉引用指向的章节实际存在（grep）
- 全项目不少于 60 个来源引用（grep）
- D1-D5 正文+附录总计 32,000-42,000 字（wc）
- glossary.md 至少 50 个条目（wc）
- 9 个代表团名称全局一致（grep）

### Git 操作约定

- Lead 负责所有 git 操作
- Writer Agent 和 QA 不做 git 操作
- 每个 Phase 审核通过后，Lead 做一次 commit

### 检查点机制

每个 Phase 审核通过后，Lead 更新 `.agents/status/phase` 文件。session 中断后读取该文件，从上次完成的 Phase 之后继续。

| 检查点值 | 含义 | 下一步 |
|---------|------|--------|
| `B4_COMPLETED` | 工作流设计已通过 | 从 Phase 0 开始 |
| `C_PHASE_0_COMPLETED` | 基础框架已通过 | 从 Phase 1 开始 |
| `C_PHASE_1_COMPLETED` | D1 已通过 | 从 Phase 2 开始 |
| `C_PHASE_2_COMPLETED` | D4+D5 已通过 | 从 Phase 3 开始 |
| `C_PHASE_3_COMPLETED` | D2+D3 已通过 | 从 Phase 4 开始 |
| `C_PHASE_4_COMPLETED` | 交叉审校已通过 | 项目完成 |
| `C_COMPLETED` | 所有 Phase 已通过 | 输出最终交付 |

## 验收标准速查

| Phase | 自动审核（阻塞） | 延迟人工审阅（不阻塞） |
|-------|----------------|---------------------|
| P0 | 目录结构完整、骨架章节完整、术语表 >= 30 条目、test-cases 文件存在 | 术语翻译准确性和自然度 |
| P1 | D1 章节完整（5+3）、正文 3000-4000 字、附录 2000-3000 字、中英对照 >= 15 动议、来源 >= 3、6 票 +3 受邀配置、术语表更新 | 程序性动议优先级排序、语言风格适配度 |
| P2 | D4: 章节完整（6+3）、正文 5000-6000 字、来源 >= 25、时间线全覆盖、9 方立场全覆盖; D5: 章节完整（7+4）、正文 5000-6000 字、P1/P2/P3+R1/R2/R3+A-E 分支+6 Session+8 方互动+3 情景全覆盖、来源 >= 15; 交叉一致性 | D5 策略激进程度、代表风格自然度、D4 客观性 |
| P3 | D2: 章节完整（6+3）、正文 4000-5000 字、9 方格局+6 Session+3 情景+与 D5 一致; D3: 章节完整（5+3）、正文 3000-4000 字、中英对照 >= 50、红线话术 4 场景+2014 语境; 交叉一致性 | D3 话术自然度、D2 磋商贴合度、D3 英文地道程度 |
| P4 | 全文档术语一致、交叉引用有效、来源总量 >= 60、总字数 32000-42000、glossary >= 50 条目、代表团名称全局一致 | 整体阅读体验、代表风格调性、策略激进程度 |

## 参考文档

- 需求文档：`docs/requirement.md`
- 主题调研：`docs/topic-research.md`
- 分析结构：`docs/analysis-structure.md`
- 产出物规划：`docs/deliverables-plan.md`
