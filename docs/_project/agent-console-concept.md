# 产品立项书：Agent 编排控制台
> **项目代号**: "Commander Console" (指挥官控制台)
> **版本**: v1.0 (基于 V5 讨论定稿)
> **核心属性**: 知识图谱驱动 · 具名 Agent 实例 · 并行编排

## 1. 业务背景与痛点

你目前作为 4 个并行 Agent 的“人肉系统总线”，面临以下不可持续的瓶颈：
1.  **认知过载**: 需要手动在 4 个终端之间切换上下文，大脑过载。
2.  **信息孤岛**: Task 很多，但经常忘了“这个 Task 是为了哪个 PRD 服务的”。
3.  **调度低效**: 需要人工盯着终端看它跑完没有，才能派发下一个任务。

我们不是在做一个简单的“聊天壳子”，我们是在做一个 **“SaaS 化的项目经理与调度器”**。

## 2. 核心价值主张 (Core Value)

### 2.1 全链路知识图谱 (Knowledge Graph)
我们将分散的文档结构化为可视化树，解决“上下文迷失”问题：
*   **链路**: `EPIC` -> `PRD/TECH` (双支柱) -> `SLICE` (交付切片) -> `TASK` (执行单元)。
*   **逻辑**:
    *   **PRD** 定义“做什么” (In/Out 范围)。
    *   **TECH** 定义“怎么做” (API/DB Schema)。
    *   **SLICE** 是汇合点：一个完整的功能切片必须包含 Story 和 Tech Spec。
    *   **状态联动**: 上游文档未定稿 (Draft)，下游任务自动阻塞 (Blocked)。

### 2.2 具名 Agent 实例 (Named Agents)
我们将抽象的“AI 角色”具象化为可管理的“数字员工”：
*   **身份**: 不再是通用的 Dev Agent，而是 `Dave-1` (后端开发), `Dave-2` (前端开发), `Tiger` (架构师)。
*   **持久化**: 每个 Agent 有独立的上下文窗口和 Shell 会话。
*   **调度**: 你可以明确指派 `Dave-1` 去领 `TASK-01`，而 `Dave-2` 并行处理 `TASK-02`。

### 2.3 隔离与解耦 (Decoupling)
*   **聊天 (Brain)**: 负责规划和生成文档。不占用 Shell 资源。
*   **执行 (Muscle)**: 负责读取文档并跑命令。只认 TASK 状态。

## 3. 解决方案架构

我们将在 `agent-orchestrator` 目录初始化一个全新的 **Mac Native App**：

*   **框架**: Tauri v2 (Rust + Webview) - 保证高性能和系统级能力 (PTY)。
*   **前端**: React + TypeScript + Vite - 现代化开发流。
*   **UI 组件**: TailwindCSS + Shadcn/UI - 快速构建 Vibe 风格仪表盘。
*   **终端内核**: `xterm.js` + `node-pty` (via Rust) - 在前端渲染真实的 Shell。
*   **数据源**: `file-system` - 直接监听 `docs/` 目录下的 Markdown 文件变化，实时驱动图谱更新。

## 4. MVP 交付范围 (In-Scope)

1.  **Commander Dashboard (仪表盘)**:
    *   左右分屏布局。
    *   左侧：多 Tab 聊天 (Chat with Tiger/Dave)。
    *   右侧上：实时渲染的文档依赖树 (基于 Markdown Frontmatter)。
    *   右侧下：多 Tab 终端 (Agent Runners)。

2.  **文档解析器**:
    *   能解析 `PRD`, `TECH`, `SLICE`, `TASK` 的 Markdown 文件头，构建 DAG (有向无环图)。

3.  **Agent Runner**:
    *   封装 `claude code` CLI。
    *   支持“一键启动任务” (将 Task 上下文注入 Prompt)。

## 5. 待办 (Out-Scope)

*   **全自动无人值守调度**: MVP 阶段仍需人工点击“开始任务”，暂不实现“昨晚睡觉今早收菜”的全自动模式。
*   **云端同步**: 仅限本地运行，利用 Git 同步。

---
*本文档由 Biz Owner (AI) 协助生成，作为开发实施的唯一真理来源。*
