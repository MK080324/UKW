---
name: Agent发现机制绑定会话启动目录
description: Claude Code的Agent工具发现subagent_type是绑定会话启动时的primary working directory，Bash中cd不会改变。这导致阶段C无法从项目目录发现项目级agent。
type: feedback
discovered: 2026-03-24
status: open
---

Claude Code 的 Agent 工具发现可用 subagent_type 的机制是**绑定会话启动时的 primary working directory**，而非 Bash 运行时的 cwd。在 Bash 中执行 `cd` 到另一个 git 仓库目录，不会使 Agent 工具发现该目录下 `.claude/agents/` 中的 agent 配置。

**Why:** 在 MUN-Planning 项目中发现。工作流设计要求阶段 C cd 到项目目录（独立 git 仓库），使用项目级 agent（writer-procedural 等）。但由于会话从元仓库启动，Agent 工具始终只能发现元仓库的 agent（research-agent、qa-agent、design-agent），无法发现项目级 agent。

**How to apply:**
1. 工作流 CLAUDE.md 的阶段 C 设计需要修改——不能假设 cd 后 Agent 工具会发现新目录的 agent
2. 可能的解决方案：
   - 方案 A：阶段 C 需要在项目目录下启动新的 Claude Code 会话（`cd projects/<name> && claude`）
   - 方案 B：项目级 agent 配置放在元仓库的 `.claude/agents/` 下（但这样不同项目的 agent 会冲突）
   - 方案 C：探索是否有 Claude Code API/机制可以动态切换 agent 发现的根目录
3. 在 B4 工作流设计阶段需要提前告知用户这个限制，并在设计中预设解决方案
