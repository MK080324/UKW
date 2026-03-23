# Knowledge-Workflow

知识工作自动化元工作流。提交需求后，Claude 全自动完成从主题调研到分析构建到内容产出到质量审核的完整流程。

## 它能做什么

输入一个需求，产出结构化文档。覆盖以下场景：

| 场景 | 示例 |
|------|------|
| 调研分析 | 深度调研报告、竞品分析、可行性评估 |
| 策略规划 | 参会指南、立场文件、OKR 体系设计 |
| 教育培训 | 课程大纲、出题出卷、知识图谱梳理 |
| 写作编辑 | 白皮书、提案标书、翻译本地化 |
| 评判审核 | 批改考试、合同审查、合规检查报告 |
| 日程规划 | 会议日程、项目排期 |

## 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- 能访问 Claude Opus 和 Sonnet 模型
- WebSearch / WebFetch 工具权限（用于在线调研）

## 快速开始

```bash
cd Knowledge-Workflow
claude
```

然后直接用自然语言描述你的需求，例如：

> 我需要一份关于 2025 年全球 AI 芯片市场的深度调研报告，目标读者是投资委员会成员，需要覆盖主要玩家、技术路线对比、市场规模预测，篇幅 8000-12000 字。

Lead 会自动进入阶段 A，与你确认需求细节后全自动推进。

## 工作流程

整个流程分三个阶段：

### 阶段 A：需求审阅

Lead 直接与你对话，不启动任何 Agent。它会：

1. 审阅需求的完整性和可执行性
2. 识别场景类型（调研分析 / 策略规划 / 教育培训 / ...）
3. 识别需要你做主观判断的决策点（如立场倾向、语气风格）
4. 提出改进建议

你确认需求定稿后，自动进入阶段 B。

### 阶段 B：规划与设计

全自动，无需人类介入。Lead 串行调度 Subagent 完成四步：

| 步骤 | 产出 | 说明 |
|------|------|------|
| B1 主题调研 | `topic-research.md` | 基于 WebSearch 的广泛调研，所有事实标注来源 |
| B2 分析结构 | `analysis-structure.md` | 根据场景类型构建分析框架（策略树 / 知识矩阵 / 评判维度等） |
| B3 产出物规划 | `deliverables-plan.md` | 交付物清单、章节大纲、Phase 划分、验收标准 |
| B4 工作流设计 | 项目 CLAUDE.md + Agent 配置 | 为本项目量身定制写作团队 |

每步产出都经过 QA Subagent 严格审批，不通过则自动修改重审（最多 8 轮）。

### 阶段 C：执行产出

全自动。Lead 创建 Agent Team，Writer Agents 并行撰写各自负责的文档/章节，QA 按预设的 test-case 审核。

完成后你会收到：
- 所有产出物（位于 `projects/<name>/output/`）
- 延迟人工审阅清单（语气风格、立场力度等主观项）

## 你需要做的

1. **描述需求** — 越具体越好（目标读者、篇幅、关键问题、格式偏好）
2. **确认阶段 A 的产出** — Lead 会提出改进建议和场景判断，你确认即可
3. **等待** — 阶段 B 和 C 全自动执行
4. **审阅延迟清单** — 项目完成后检查主观项是否满意

## 中断恢复

如果 session 中断，重新启动 Claude Code 后 Lead 会自动检测检查点文件（`.agents/status/phase`），从上次完成的步骤继续，不会重复已完成的工作。

## 目录结构

```
Knowledge-Workflow/
├── CLAUDE.md                    # 元工作流 Lead 指令
├── .claude/
│   ├── settings.json            # 权限配置
│   └── agents/
│       ├── research-agent.md    # 调研 Agent (opus)
│       ├── design-agent.md      # 工作流设计 Agent (opus)
│       └── qa-agent.md          # 质量审批 Agent (sonnet)
├── templates/                   # 文档模板（B1-B4 参考格式）
├── projects/                    # 项目目录（每个项目独立 git）
│   └── <project-name>/
│       ├── CLAUDE.md            # 项目专属指令
│       ├── docs/                # 规划文档
│       ├── output/              # 最终交付物
│       └── .agents/             # Agent 协作文件
└── .gitignore                   # 忽略 projects/
```

## 自定义

- **模板**：修改 `templates/` 下的文件可调整各阶段产出的格式和结构
- **Agent 配置**：修改 `.claude/agents/` 下的文件可调整模型、轮次上限、工具权限
- **QA 检查清单**：在 `qa-agent.md` 中修改四套检查清单的审批标准

## License

[MIT](LICENSE)
