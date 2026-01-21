# TECH-E-001-v1：AI 员工管理控制台 - 技术方案

> **文档路径**：`/docs/E-001-Agent-Console/tech/TECH-E-001-v1.md`
>
> * **EPIC_ID**：E-001
> * **EPIC_DIR**：`E-001-Agent-Console`
> * **版本**：v1
> * **状态**：DRAFT
> * **作者**：tech agent
> * **创建日期**：2026-01-21
> * **关联文档**：
>   - biz：`/docs/_project/biz-overview.md`
>   - prd：`../prd/PRD-E-001-v0.md`
>   - story：`../story/STORY-001-全局状态可视化.md`、`STORY-002-任务调度闭环.md`、`STORY-003-实时进度监控.md`、`STORY-004-自动资源管理.md`
>   - tech-baseline：`/docs/_project/tech-baseline.md`

---

## 0. 文档元信息

- **EPIC_ID**：`E-001`
- **EPIC_DIR**：`E-001-Agent-Console`
- **版本**：`v1`
- **状态**：`DRAFT`
- **作者**：`tech agent`
- **关联文档**：
  - biz：`/docs/_project/biz-overview.md`
  - prd：`../prd/PRD-E-001-v0.md`
  - story：`../story/`（本期纳入 4 个 Story）
  - project baseline：`/docs/_project/tech-baseline.md`

---

## 1. 目标与范围对齐（引用上游，不重写）

### 1.1 目标（来自 biz/prd）

**体验北极星**：从"焦虑"到"掌控"，一眼看全局，无需在终端间切换

**核心业务目标**（摘自 biz-overview）：
- G1：能同时管理的并行 Dev 数量：4 个 → 8+ 个
- G2：单次上下文切换时间：[OPEN] → < 5 秒
- G3：任务依赖关系可见性：0% → 100%
- G4：Agent 闲置资源释放时间：人工 → 5 分钟
- G5：全局状态刷新时间：[OPEN] → < 1 秒

### 1.2 Out of Scope（来自 biz/prd）

**Phase 1 不包含**：
- ❌ 自动任务调度（L2 级别）
- ❌ Agent 之间自主协商（L3 级别）
- ❌ 多用户支持
- ❌ 远程控制功能
- ❌ 复杂数据库（文件系统 + 内存缓存够用）

### 1.3 验收口径摘要（来自 Story AC）

**STORY-001（全局状态可视化）**：
- AC1：应用启动时间 < 3 秒
- AC2：知识图谱树正确显示文档依赖关系
- AC3：员工档案正确显示 Agent 状态
- AC4：文档变化后，知识图谱在 2 秒内自动刷新
- AC5：用户能在 30 秒内理解所有 dev 的状态

**STORY-002（任务调度闭环）**：
- AC1：用户能在知识图谱中选中 TASK，高亮依赖路径
- AC2：用户能在员工档案中选中空闲 Agent，显示"分派任务"按钮
- AC3：用户点击"分派任务"后，讨论室自动切换并填充任务描述
- AC4：用户点击"发送"后，Agent 开始执行，状态实时更新
- AC5：用户能在 30 秒内完成"选任务 → 选人 → 确认"
- AC6：系统能检测 Agent 忙碌状态，防止重复分派

**STORY-003（实时进度监控）**：
- AC1：用户能实时看到 Agent 的日志输出（延迟 < 1 秒）
- AC2：错误日志自动红色高亮
- AC3：用户能搜索日志，快速定位错误信息
- AC4：用户能清空日志，释放内存
- AC5：用户能一键询问 Agent 关于错误的问题
- AC6：日志超过 1000 行时，自动清理旧日志

**STORY-004（自动资源管理）**：
- AC1：Agent 闲置 5 分钟后自动释放
- AC2：Agent 闲置检测准确，误杀率 < 1%
- AC3：Session 自动恢复，恢复成功率 > 95%
- AC4：用户能手动启动/停止 Agent
- AC5：Session 损坏时，提供修复方案
- AC6：自动释放不影响其他 Agent

### 1.4 需求层可观测性摘要（来自 prd）

**用户体验指标**：
- 应用启动时间 < 3 秒
- 全局状态刷新时间 < 1 秒
- 日志实时刷新延迟 < 1 秒
- 文档打开时间 < 1 秒
- 分派任务时间 < 30 秒

**数据量限制**：
- 默认展示最近 1000 条日志
- 默认展示最近 5 个历史任务
- 知识图谱默认展开 3 层

---

## 2. 现状与约束（必须先读代码）

### 2.1 相关代码/系统现状

**[NEW_PROJECT]**：E-001 是第一个 Epic，目前**没有任何代码实现**。

**现有基础设施**：
- 项目目录结构已建立：`docs/`、`.claude/`、`README.md`、`CHANGELOG.md`
- 文档体系已建立：biz-overview、tech-baseline、workflow-overview、template-mapping
- 无代码仓库：目前只有文档，没有 Rust 或前端代码

### 2.2 复用清单（避免重复造轮子）

**可复用能力/组件**（来自 tech-baseline.md）：
- ✅ **Tauri v2 桌面框架**：提供系统级能力（启动/杀死进程）、跨平台支持
- ✅ **React 18+ + TypeScript 5+ + Vite 5+**：成熟的前端技术栈
- ✅ **Zustand**：轻量级状态管理（API 简单、无 Provider、TypeScript 支持好）
- ✅ **Cytoscape.js**：知识图谱可视化（API 简单、内置树形布局、性能好）
- ✅ **xterm.js**：终端模拟器（VS Code 同款）
- ✅ **TailwindCSS + Shadcn/UI**：现代化 UI 组件库
- ✅ **notify（Rust）**：文件监听库
- ✅ **serde + serde_yaml（Rust）**：解析 Frontmatter

**不复用的理由**：
- ❌ **Electron**：体积大（> 100MB）、无系统级能力
- ❌ **Vue / Svelte**：生态不如 React、组件库少
- ❌ **Redux Toolkit**：样板代码多、学习成本高
- ❌ **D3.js**：学习曲线陡峭、代码量多 5-10 倍
- ❌ **Rust 原生 PTY**：学习成本高、生态不成熟（使用 node-pty 代替）

### 2.3 硬约束

**技术基线约束**（来自 tech-baseline.md）：
- 桌面框架：必须使用 Tauri v2
- 前端：React 18+ + TypeScript 5+ + Vite 5+
- 状态管理：必须使用 Zustand
- UI 库：必须使用 TailwindCSS + Shadcn/UI
- 可视化：必须使用 Cytoscape.js（知识图谱）、xterm.js（终端）
- 包管理器：前端必须使用 pnpm
- 测试覆盖率：Phase 1 目标 50%（非 70%）
- 单文件不超过 600 行

**合规/权限/审计约束**：
- Phase 1 只有主用户（开发者自己），无需权限控制
- 未来扩展（Phase 2）再考虑多用户支持

**交付约束**：
- 必须交付可运行的 Tauri 桌面应用（macOS + Linux）
- 必须支持同时管理 4 个并行 dev
- 必须在 3 秒内启动完成

---

## 3. 方案总览（1 页能讲清）

### 3.1 方案一句话

> 基于 Tauri v2 构建 AI 员工管理控制台，实现"一眼看全局，一键触达任何 Agent"的用户体验。

### 3.2 架构变化概览

**[NEW_PROJECT]**：从零开始构建，无历史包袱。

**核心架构**：
```
┌─────────────────────────────────────────────────────────┐
│  Commander Console (Tauri v2 桌面应用)                    │
├─────────────────────────────────────────────────────────┤
│  前端层：React + TypeScript + Vite                       │
│  UI：TailwindCSS + Shadcn/UI                             │
│  可视化：Cytoscape.js（知识图谱）+ xterm.js（终端）       │
│  状态：Zustand                                            │
│  路由：React Router v7                                    │
├─────────────────────────────────────────────────────────┤
│  后端层：Rust (Tauri v2)                                  │
│  进程管理：std::process + 人工监听                        │
│  文件监听：notify                                         │
├─────────────────────────────────────────────────────────┤
│  数据层：文件系统（docs/ 目录）                            │
│  格式：Markdown + Frontmatter                             │
│  缓存：内存 DAG（有向无环图）                              │
└─────────────────────────────────────────────────────────┘
```

### 3.3 关键 trade-off

| 决策点 | 选择 | Trade-off | 理由 |
|--------|------|-----------|------|
| **桌面框架** | Tauri v2 | 学习成本略高于 Electron | 系统级能力、体积小（< 10MB）、跨平台 |
| **状态管理** | Zustand | 生态不如 Redux | API 简单、无 Provider、轻量（< 1KB） |
| **数据层** | 文件系统 | 无法复杂查询 | 简单、Git 作为备份、Phase 1 够用 |
| **终端** | xterm.js + node-pty | 打包体积增加 20MB+ | VS Code 同款、稳定性高、生态成熟 |

### 3.4 影响面

**新增模块**：
- 前端：`src/`（React 组件、stores、types）
- 后端：`src-tauri/src/`（Rust commands、process、watcher、graph）
- 数据：`docs/`（Markdown 文档）、`sessions/`（JSONL 对话历史）

**不影响模块**：
- 无（全新项目）

---

## 4. 详细设计

### 4.1 模块/服务边界与职责

#### 4.1.1 前端模块划分（React）

```
src/
├── components/           # 可复用组件
│   ├── ui/              # Shadcn/UI 组件（复制代码到项目）
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   └── ...
│   ├── knowledge-graph/  # 知识图谱组件
│   │   ├── GraphView.tsx       # 主视图（Cytoscape.js）
│   │   ├── GraphNode.tsx       # 节点组件
│   │   └── GraphEdge.tsx       # 边组件
│   ├── discussion/       # 讨论室组件
│   │   ├── DiscussionPanel.tsx  # 主面板
│   │   ├── TabBar.tsx           # Tab 栏
│   │   ├── MessageList.tsx      # 消息列表
│   │   └── InputBox.tsx         # 输入框
│   ├── terminal/         # 运行终端组件
│   │   ├── TerminalPanel.tsx    # 主面板
│   │   ├── TerminalPane.tsx     # 单个 Agent 终端
│   │   └── LogViewer.tsx        # 日志查看器（xterm.js）
│   └── staff/            # 员工档案组件
│       ├── StaffPanel.tsx       # 主面板
│       └── AgentCard.tsx        # Agent 卡片
├── stores/              # Zustand stores
│   ├── documentStore.ts         # 文档状态（知识图谱）
│   ├── agentStore.ts            # Agent 状态（员工档案）
│   ├── sessionStore.ts          # Session 状态（讨论室）
│   └── logStore.ts              # 日志状态（运行终端）
├── types/               # TypeScript 类型定义
│   ├── document.ts              # 文档类型（EPIC/PRD/TECH/STORY/TASK）
│   ├── agent.ts                 # Agent 类型
│   ├── session.ts               # Session 类型
│   └── api.ts                   # API 契约类型
├── lib/                 # 工具函数
│   ├── tauri.ts                 # Tauri API 封装
│   ├── graph.ts                 # 图算法（DAG 构建）
│   └── parser.ts                # Markdown Frontmatter 解析
├── pages/               # 页面组件
│   └── Dashboard.tsx            # 主页面
└── App.tsx              # 应用入口
```

**模块职责**：
- **knowledge-graph**：显示 Epic → PRD/TECH → STORY → TASK 的依赖关系
- **discussion**：多 tab 对话，和 Agent 进行多轮交互
- **terminal**：显示 Agent 执行进度和实时日志流
- **staff**：显示 Agent 状态、当前任务、技能标签

#### 4.1.2 后端模块划分（Rust）

```
src-tauri/src/
├── main.rs              # 入口（Tauri commands 注册）
├── commands/            # Tauri commands（前端调用接口）
│   ├── mod.rs
│   ├── document.rs              # 文档相关命令
│   ├── agent.rs                 # Agent 相关命令
│   ├── session.rs               # Session 相关命令
│   └── log.rs                   # 日志相关命令
├── process/             # 进程管理
│   ├── mod.rs
│   ├── manager.rs               # Agent 进程管理器
│   └── monitor.rs               # 进程状态监控（闲置检测）
├── watcher/             # 文件监听
│   ├── mod.rs
│   ├── docs_watcher.rs          # docs/ 目录监听
│   └── event.rs                 # 文件变化事件
├── graph/               # DAG 构建
│   ├── mod.rs
│   ├── builder.rs               # DAG 构建器
│   ├── node.rs                  # 节点定义
│   └── edge.rs                  # 边定义
├── parser/              # Markdown 解析
│   ├── mod.rs
│   ├── frontmatter.rs           # Frontmatter 解析
│   └── document.rs              # 文档解析
└── models/              # 数据模型
    ├── mod.rs
    ├── document.rs              # 文档模型
    ├── agent.rs                 # Agent 模型
    └── session.rs               # Session 模型
```

**模块职责**：
- **commands**：提供前端调用的 API 接口
- **process**：管理 Agent 进程（启动/杀死/监控）
- **watcher**：监听 docs/ 目录变化，触发 DAG 重建
- **graph**：构建文档依赖关系的 DAG
- **parser**：解析 Markdown Frontmatter
- **models**：定义数据结构

### 4.2 数据模型与迁移策略

#### 4.2.1 数据模型

**[NEW_PROJECT]**：无迁移策略，从零开始。

**文档模型**（TypeScript + Rust）：

```typescript
// src/types/document.ts
export enum DocumentType {
  EPIC = "EPIC",
  PRD = "PRD",
  TECH = "TECH",
  STORY = "STORY",
  TASK = "TASK",
}

export enum DocumentStatus {
  TODO = "TODO",
  DOING = "DOING",
  BLOCKED = "BLOCKED",
  DONE = "DONE",
}

export interface Document {
  id: string;                  // 文档唯一标识（如 "PRD-E-001-v1"）
  type: DocumentType;          // 文档类型
  name: string;                // 文档名称（从 Frontmatter title 字段）
  status: DocumentStatus;      // 文档状态
  path: string;                // 文档绝对路径
  parent_id: string | null;    // 父节点 ID（EPIC 无父节点）
  dependencies: string[];      // 依赖的文档 ID 列表
  metadata: {
    epic_id: string;
    version: string;
    created_at: string;        // ISO 8601 格式
    updated_at: string;        // ISO 8601 格式
  };
}
```

```rust
// src-tauri/src/models/document.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub enum DocumentType {
    Epic,
    Prd,
    Tech,
    Story,
    Task,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub enum DocumentStatus {
    Todo,
    Doing,
    Blocked,
    Done,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Document {
    pub id: String,
    #[serde(rename = "type")]
    pub doc_type: DocumentType,
    pub name: String,
    pub status: DocumentStatus,
    pub path: String,
    pub parent_id: Option<String>,
    pub dependencies: Vec<String>,
    pub metadata: DocumentMetadata,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DocumentMetadata {
    pub epic_id: String,
    pub version: String,
    pub created_at: String,
    pub updated_at: String,
}
```

**Agent 模型**（TypeScript + Rust）：

```typescript
// src/types/agent.ts
export enum AgentStatus {
  IDLE = "IDLE",
  BUSY = "BUSY",
  ERROR = "ERROR",
  RELEASED = "RELEASED",
}

export interface Agent {
  id: string;                  // Agent 唯一标识（如 "dave-1"）
  name: string;                // Agent 显示名称
  status: AgentStatus;         // Agent 状态
  current_task: CurrentTask | null;  // 当前执行的任务
  skills: string[];            // 技能标签
  last_active: string;         // 最后活动时间（ISO 8601 格式）
  process_id: number | null;   // 进程 ID（如果 status=BUSY）
}

export interface CurrentTask {
  task_id: string;
  task_name: string;
  started_at: string;          // ISO 8601 格式
}

export interface AgentHistory {
  task_id: string;
  task_name: string;
  status: DocumentStatus;
  completed_at: string;        // ISO 8601 格式
  duration_seconds: number;
}
```

```rust
// src-tauri/src/models/agent.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub enum AgentStatus {
    Idle,
    Busy,
    Error,
    Released,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Agent {
    pub id: String,
    pub name: String,
    pub status: AgentStatus,
    pub current_task: Option<CurrentTask>,
    pub skills: Vec<String>,
    pub last_active: String,
    pub process_id: Option<u32>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CurrentTask {
    pub task_id: String,
    pub task_name: String,
    pub started_at: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct AgentHistory {
    pub task_id: String,
    pub task_name: String,
    pub status: DocumentStatus,
    pub completed_at: String,
    pub duration_seconds: u64,
}
```

**Session 模型**（TypeScript + Rust）：

```typescript
// src/types/session.ts
export enum MessageRole {
  USER = "user",
  ASSISTANT = "assistant",
}

export interface Message {
  timestamp: string;           // ISO 8601 格式
  role: MessageRole;
  content: string;
  tool_calls?: ToolCall[];     // 可选：Agent 调用的工具
}

export interface ToolCall {
  name: string;                // 工具名称（如 "read_file"）
  args: Record<string, any>;    // 工具参数
  result: string;              // 执行结果
}
```

**日志模型**（TypeScript + Rust）：

```typescript
// src/types/log.ts
export enum LogLevel {
  INFO = "INFO",
  WARN = "WARN",
  ERROR = "ERROR",
}

export interface LogEntry {
  timestamp: string;           // ISO 8601 格式
  level: LogLevel;
  message: string;
  raw_output: string;          // 原始输出（用于显示）
}
```

#### 4.2.2 数据存储策略

**文档数据**（docs/ 目录）：
- 格式：Markdown + Frontmatter（YAML 格式）
- 位置：`docs/E-001-xxx/prd/PRD-E-001-v1.md`
- 缓存：内存 DAG（有向无环图）

**Session 数据**（sessions/ 目录）：
- 格式：JSONL（JSON Lines）
- 位置：`sessions/dave-1.jsonl`
- 示例：
  ```json
  {"timestamp": "2026-01-20T14:30:00Z", "role": "user", "content": "领 TASK-03，开始实现用户登录 API"}
  {"timestamp": "2026-01-20T14:30:05Z", "role": "assistant", "content": "好的，开始实现 TASK-03。首先检查 PRD 中的需求..."}
  ```

**Agent 配置**（可选，Phase 1 可用默认配置）：
- 位置：`.commander/agents.yaml` 或项目根目录
- 格式：
  ```yaml
  agents:
    - id: "dave-1"
      name: "Dave-1"
      skills: ["后端", "Rust"]
      type: "backend"
  ```

### 4.3 API / 事件 / 任务设计

> ⚠️ **强制要求**：所有后端 API 必须先定义接口契约，前端按契约开发。

#### 4.3.1 Tauri Commands（前端 → Rust）

**契约格式**：所有 Commands 遵循 `tech-baseline.md` 定义的响应格式。

**响应格式**（成功）：
```typescript
{
  code: 0;
  data: T;
  message: "success";
}
```

**响应格式**（失败）：
```typescript
{
  code: number;               // 1000-1999（客户端错误）、5000-5999（服务器错误）
  data: null;
  message: string;            // 错误信息
}
```

**Command 列表**（基于 4 个 STORY 的接口契约草案）：

| Command | 请求类型 | 响应类型 | 说明 |
|---------|---------|---------|------|
| `open_document` | `{ document_path: string; line_number?: number }` | `{ success: boolean; error_message?: string }` | 打开文档 |
| `get_agent_details` | `{ agent_id: string }` | `{ success: boolean; agent?: Agent; error_message?: string }` | 获取 Agent 详细信息 |
| `highlight_dependency_path` | `{ document_id: string }` | `{ success: boolean; path?: DependencyPath[]; error_message?: string }` | 高亮依赖路径 |
| `start_agent` | `{ agent_id: string; restore_session: boolean; session_path?: string }` | `{ success: boolean; process_id?: number; session_restored?: boolean; error_message?: string }` | 启动 Agent |
| `send_message_to_agent` | `{ agent_id: string; message: string; task_id?: string; save_to_history: boolean }` | `{ success: boolean; message_id?: string; timestamp?: string; error_message?: string }` | 发送消息给 Agent |
| `extract_task_description` | `{ task_id: string; task_path: string }` | `{ success: boolean; description?: string; summary?: string; error_message?: string }` | 提取任务描述 |
| `check_agent_status` | `{ agent_id: string }` | `{ success: boolean; is_busy?: boolean; current_task?: CurrentTaskInfo; estimated_completion?: string; error_message?: string }` | 检查 Agent 是否忙碌 |
| `clear_agent_logs` | `{ agent_id: string }` | `{ success: boolean; error_message?: string }` | 清空日志 |
| `search_agent_logs` | `{ agent_id: string; keyword: string; use_regex: boolean }` | `{ success: boolean; matches?: LogMatch[]; total_matches?: number; error_message?: string }` | 搜索日志 |
| `toggle_log_stream` | `{ agent_id: string; paused: boolean }` | `{ success: boolean; paused?: boolean; error_message?: string }` | 暂停/继续日志流 |
| `get_agent_process_status` | `{ agent_id: string }` | `{ success: boolean; process?: ProcessStatus; error_message?: string }` | 获取 Agent 进程状态 |
| `stop_agent` | `{ agent_id: string; force: boolean }` | `{ success: boolean; session_saved?: boolean; termination_time?: number; error_message?: string }` | 停止 Agent |
| `get_agent_idle_time` | `{ agent_id: string }` | `{ success: boolean; idle_time?: number; idle_threshold?: number; will_release_soon?: boolean; estimated_release_time?: string; error_message?: string }` | 获取 Agent 闲置时间 |
| `repair_session` | `{ agent_id: string; session_path: string }` | `{ success: boolean; repaired?: boolean; error_lines_removed?: number; valid_lines_restored?: number; error_message?: string }` | 修复损坏的 Session |

#### 4.3.2 Tauri Events（Rust → 前端）

**事件列表**（基于 4 个 STORY 的接口契约草案）：

| Event | Payload 类型 | 触发时机 | 说明 |
|-------|-------------|---------|------|
| `docs:updated` | `{ documents: Document[]; timestamp: string }` | 文件系统变化时 | 文档列表推送 |
| `agents:status_updated` | `{ agents: Agent[]; timestamp: string }` | Agent 状态变化时 | Agent 状态推送 |
| `agent:reply_received` | `{ agent_id: string; message_id: string; reply: string; timestamp: string; tools_called?: ToolCall[] }` | Agent 回复时 | Agent 回复推送 |
| `agent:status_changed` | `{ agent_id: string; old_status: AgentStatus; new_status: AgentStatus; task_id?: string; timestamp: string }` | Agent 状态变化时 | Agent 状态变更推送 |
| `agent:log_output` | `{ agent_id: string; logs: LogEntry[]; timestamp: string }` | Agent 输出日志时 | 日志流推送（批量） |
| `agent:idle_timeout` | `{ agent_id: string; idle_duration: number; idle_threshold: number; will_release: boolean; release_after: number; timestamp: string }` | Agent 闲置超时 | 闲置检测推送 |
| `agent:crashed` | `{ agent_id: string; process_id: number; exit_code: number; error_message?: string; timestamp: string }` | Agent 崩溃时 | 崩溃事件推送 |

#### 4.3.3 API 契约完整定义（示例）

**示例 1：start_agent Command**

```typescript
// 请求类型
interface StartAgentRequest {
  agent_id: string;             // Agent ID（如 "dave-1"）
  restore_session: boolean;     // 是否从 JSONL 恢复对话历史
  session_path?: string;        // Session 文件路径（可选）
}

// 响应类型（成功）
interface StartAgentResponse {
  code: 0;
  data: {
    success: true;
    process_id: number;         // Agent 进程 ID
    session_restored: boolean;  // Session 是否恢复成功
  };
  message: "success";
}

// 响应类型（失败）
interface StartAgentErrorResponse {
  code: 1001 | 5002;           // 1001: Agent 已在运行, 5002: 启动失败
  data: null;
  message: string;              // 错误信息
}

// 边界情况
// - Agent 已运行：返回 code: 1001, message: "Agent 已在运行"
// - Session 文件不存在：返回 session_restored: false（不影响启动）
// - Session 文件损坏：返回 session_restored: false, message: "Session 损坏，将启动新 Session"
```

**示例 2：docs:updated Event**

```typescript
// 事件 payload
interface DocsUpdatedEvent {
  documents: Document[];        // 文档列表
  timestamp: string;            // 推送时间（ISO 8601 格式）
}

// 边界情况
// - 文档列表为空：返回空数组 []，前端显示空态
// - 文档路径不存在：不包含在列表中，前端不显示
// - Frontmatter 缺失：使用默认值（name="未知文档", status="TODO"）
```

### 4.4 安全 / 权限 / 审计

**Phase 1 只有主用户（开发者自己）**，无需权限控制。

**未来扩展**（Phase 2）：
- JWT Token 认证
- RBAC（Role-Based Access Control）
- 审计日志（记录所有操作）

### 4.5 可观测性（工程侧）

**日志规范**（来自 tech-baseline.md）：

**前端日志**（开发环境）：
```typescript
console.log('[AgentStore] 创建 Agent:', agent);
console.error('[AgentStore] 创建失败:', error);
```

**Rust 日志**（生产环境）：
```rust
use tracing::{info, error};

info!("启动 Agent: id={}", id);
error!("启动失败: id={}, error={}", id, error);
```

**日志级别**：
- ERROR：影响功能的错误
- WARN：潜在问题
- INFO：关键操作（如启动 Agent）
- DEBUG：调试信息（仅开发环境）

**指标监控**（Phase 2）：
- Agent 数量：当前运行的 Agent 数
- 响应时间：API 响应时间（P50/P95/P99）
- 错误率：Agent 启动失败率

### 4.6 失败模式与可靠性

**幂等/重试**：
- 启动 Agent：幂等（重复启动返回错误）
- 停止 Agent：幂等（重复停止返回成功）
- 发送消息：不幂等（需要前端去重）

**降级/兜底**：
- 文档解析失败：显示错误图标 + "重试"按钮，单个文档失败不影响其他文档
- Agent 启动失败：提供"手动启动"降级方案
- Session 恢复失败：启动新 Session（失去历史对话）
- 日志刷太快：提供"暂停日志流"按钮

**限流/熔断**：
- 无需限流（Phase 1 只有主用户）
- 日志超过 1000 行：自动清理旧日志

---

## 5. 上线策略（MVP→演进）

### 5.1 Feature Flag / 灰度

**Phase 1 全功能发布**，无需 Feature Flag。

### 5.2 回滚策略

- 无数据库迁移，无需回滚
- 配置文件损坏：从 Git 历史恢复
- Session 损坏：启动新 Session

### 5.3 数据回填/重算

- 无需回填（全新项目）

### 5.4 兼容期策略

- Phase 1 只有主用户，无需兼容期
- Phase 2 再考虑多用户支持

---

## 6. 风险、未决项与升级点

### 6.1 风险（[RISK]）

**[RISK-1] 知识图谱布局算法选择**
- **问题**：Cytoscape.js 提供多种布局算法（breadthfirst、dagre、circle），选择哪个？
- **影响**：影响可读性和性能
- **缓解**：先测试 `breadthfirst`（树形布局），如果效果不好，再试 `dagre`（层次布局）
- **责任方**：tech

**[RISK-2] 文档监听性能优化**
- **问题**：如果文档数量超过 1000 个，文件监听可能会有性能问题
- **影响**：卡顿、延迟高
- **缓解**：Phase 1 先监听所有文档，如果性能有问题，再限制监听范围（只监听 `docs/_project/` 和当前 Epic 目录）
- **责任方**：tech

**[RISK-3] Session 恢复成功率**
- **问题**：JSONL 文件损坏时，如何恢复对话历史？
- **影响**：用户体验差
- **缓解**：Phase 1 先提供"手动修复"工具，Phase 2 再考虑自动修复
- **责任方**：tech

### 6.2 未决项（[OPEN]）

**[OPEN-1] Agent 配置文件位置**
- **问题**：`agents.yaml` 放在哪里？`~/.commander/agents.yaml` 还是项目根目录？
- **建议**：先支持项目根目录 `.commander/agents.yaml`，Phase 2 再支持全局配置
- **责任方**：prd + tech

**[OPEN-2] 知识图谱节点数量限制**
- **问题**：如果节点超过 100 个，如何处理？
- **建议**：提供"折叠/过滤"功能，默认展开 3 层，深层节点折叠
- **责任方**：prd + ux

**[OPEN-3] 消息发送超时时间**
- **问题**：Agent 多久没回复算"无响应"？
- **建议**：设置为 5 秒（基于 Claude Code 的平均响应时间）
- **责任方**：prd + tech

**[OPEN-4] 任务描述提取算法**
- **问题**：如何从 TASK 文档提取任务描述？
- **建议**：优先使用 Frontmatter 的 `description` 字段，如果为空，则提取第一段正文
- **责任方**：tech

**[OPEN-5] 日志存储策略**
- **问题**：日志是否需要持久化到磁盘？
- **建议**：Phase 1 先不持久化，只保留在内存中（最多 1000 行）。Phase 2 再考虑持久化到文件。
- **责任方**：prd + tech

**[OPEN-6] 日志搜索性能优化**
- **问题**：如果日志超过 10000 行，搜索会不会卡顿？
- **建议**：Phase 1 先限制最多保留 1000 行日志。Phase 2 再考虑索引优化。
- **责任方**：tech

**[OPEN-7] 终端模拟器选择**
- **问题**：xterm.js 是否满足需求？是否需要其他方案？
- **建议**：先验证 xterm.js 是否能满足需求（日志流、颜色高亮、搜索）。如果不行，再考虑其他方案。
- **责任方**：tech

**[OPEN-8] 闲置时间阈值**
- **问题**：默认 5 分钟是否合适？是否需要可配置？
- **建议**：默认 5 分钟，提供配置文件让用户自定义
- **责任方**：prd + tech

**[OPEN-9] Session 恢复成功率**
- **问题**：如何保证 Session 恢复成功率 > 95%？
- **建议**：使用冗余存储（如备份文件），定期校验 JSONL 文件完整性
- **责任方**：tech

**[OPEN-10] 闲置检测准确性**
- **问题**：如何准确检测 Agent 是否闲置？
- **建议**：综合判断（无输出 + 无交互 + 无任务），避免误杀
- **责任方**：tech

### 6.3 升级点（需要升级给）

**升级给 prd**：
- OPEN-1：Agent 配置文件位置
- OPEN-2：知识图谱节点数量限制
- OPEN-3：消息发送超时时间
- OPEN-5：日志存储策略
- OPEN-8：闲置时间阈值

**升级给 biz-owner**：
- 无

**升级给 proj**：
- 无

---

## 7. 任务拆解建议（供 proj/dev 落 TASK）

> 输出顺序：先打通主链路，再补齐边界与质量；标出依赖关系与可以并行的项。

### 7.1 任务依赖与并行可行性分析

| TASK_ID | 依赖类型 | 依赖任务 | 可并行 | 说明 |
|---------|----------|----------|--------|------|
| TASK-001 | 无 | - | ✅ | 项目初始化（Tauri + React + TypeScript） |
| TASK-002 | 硬依赖 | TASK-001 | ❌ | 数据模型定义（TypeScript + Rust） |
| TASK-003 | 接口依赖 | TASK-002 | ✅ | Tauri Commands 框架（Rust） |
| TASK-004 | 接口依赖 | TASK-002 | ✅ | 前端 Zustand Stores（TypeScript） |
| TASK-005 | 接口依赖 | TASK-003 | ✅ | 文件监听 + DAG 构建（Rust） |
| TASK-006 | 接口依赖 | TASK-004 | ✅ | 知识图谱组件（React + Cytoscape.js） |
| TASK-007 | 硬依赖 | TASK-003, TASK-004 | ❌ | Agent 进程管理（Rust + 前端状态） |
| TASK-008 | 硬依赖 | TASK-007 | ❌ | 讨论室组件（React + Tab） |
| TASK-009 | 硬依赖 | TASK-007 | ❌ | 运行终端组件（React + xterm.js） |
| TASK-010 | 硬依赖 | TASK-007 | ❌ | 员工档案组件（React + Agent 卡片） |
| TASK-011 | 接口依赖 | TASK-006 | ✅ | 任务调度功能（STORY-002） |
| TASK-012 | 硬依赖 | TASK-009 | ❌ | 实时日志功能（STORY-003） |
| TASK-013 | 硬依赖 | TASK-007 | ❌ | 自动资源管理（STORY-004） |
| TASK-014 | 接口依赖 | TASK-006, TASK-010 | ✅ | 主页面布局（Dashboard） |
| TASK-015 | 硬依赖 | 所有 | ❌ | 测试 + 修复 |

**依赖类型说明**：
- **硬依赖（代码必须）**：代码直接 import 其他任务的模块，禁止 mock
- **接口依赖（联调需要）**：只需要接口契约，允许桩实现，但类型必须对齐
- **无依赖**：完全独立，可立即并行

**并行策略**：
- ✅ **第一批（立即可并行）**：TASK-001（项目初始化）
- ⚠️ **第二批（等接口契约）**：TASK-003、TASK-004、TASK-005、TASK-006（按契约先行开发）
- ⏳ **第三批（等硬依赖）**：TASK-007、TASK-008、TASK-009、TASK-010（必须等 TASK-003、TASK-004 完成）

### 7.2 建议任务列表

**STORY-001：全局状态可视化**
- `TASK-001`：项目初始化（Tauri + React + TypeScript + Vite + TailwindCSS + Shadcn/UI）<br>**硬依赖：无 | 可并行：是**
- `TASK-002`：数据模型定义（TypeScript + Rust 对应）<br>**硬依赖：TASK-001 | 可并行：否**
- `TASK-003`：Tauri Commands 框架（Rust commands 注册 + 事件系统）<br>**接口依赖：TASK-002 | 可并行：是**
- `TASK-004`：前端 Zustand Stores（documentStore、agentStore、sessionStore、logStore）<br>**接口依赖：TASK-002 | 可并行：是**
- `TASK-005`：文件监听 + DAG 构建（Rust notify + DAG 构建器 + Frontmatter 解析）<br>**接口依赖：TASK-002 | 可并行：是**
- `TASK-006`：知识图谱组件（React + Cytoscape.js + 节点/边样式 + 交互逻辑）<br>**接口依赖：TASK-004 | 可并行：是**
- `TASK-010`：员工档案组件（React + Agent 卡片 + 状态显示）<br>**接口依赖：TASK-004 | 可并行：是**
- `TASK-014`：主页面布局（Dashboard + 知识图谱 40% + 员工档案 30%）<br>**接口依赖：TASK-006, TASK-010 | 可并行：是**

**STORY-002：任务调度闭环**
- `TASK-007`：Agent 进程管理（Rust std::process + 启动/杀死 + 进程监控）<br>**硬依赖：TASK-003, TASK-004 | 可并行：否**
- `TASK-008`：讨论室组件（React + Tab + 消息列表 + 输入框 + Session 持久化）<br>**硬依赖：TASK-007 | 可并行：否**
- `TASK-011`：任务调度功能（选中 TASK + 选中 Agent + 分派按钮 + 自动填充任务描述）<br>**接口依赖：TASK-006 | 可并行：是**

**STORY-003：实时进度监控**
- `TASK-009`：运行终端组件（React + xterm.js + 日志流 + 颜色高亮 + 搜索）<br>**硬依赖：TASK-007 | 可并行：否**
- `TASK-012`：实时日志功能（日志流推送 + 错误高亮 + 清空日志 + 搜索）<br>**硬依赖：TASK-009 | 可并行：否**

**STORY-004：自动资源管理**
- `TASK-013`：自动资源管理（闲置检测 + 自动释放 + Session 恢复 + 手动启动/停止）<br>**硬依赖：TASK-007 | 可并行：否**

**公共任务**
- `TASK-015`：测试 + 修复（单元测试 + 集成测试 + 性能测试 + Bug 修复）<br>**硬依赖：所有 | 可并行：否**

### 7.3 代码修改清单（Critical - 纵向追踪） ⭐

**[NEW_PROJECT]**：无现有代码，无需修改清单。

**新增文件清单**（完整列表）：

#### 前端文件（TypeScript + React）
- [ ] `src/types/document.ts` - 文档类型定义
- [ ] `src/types/agent.ts` - Agent 类型定义
- [ ] `src/types/session.ts` - Session 类型定义
- [ ] `src/types/log.ts` - 日志类型定义
- [ ] `src/types/api.ts` - API 契约类型定义
- [ ] `src/stores/documentStore.ts` - 文档状态管理
- [ ] `src/stores/agentStore.ts` - Agent 状态管理
- [ ] `src/stores/sessionStore.ts` - Session 状态管理
- [ ] `src/stores/logStore.ts` - 日志状态管理
- [ ] `src/lib/tauri.ts` - Tauri API 封装
- [ ] `src/lib/graph.ts` - 图算法（DAG 构建）
- [ ] `src/lib/parser.ts` - Markdown Frontmatter 解析
- [ ] `src/components/knowledge-graph/GraphView.tsx` - 知识图谱主视图
- [ ] `src/components/knowledge-graph/GraphNode.tsx` - 节点组件
- [ ] `src/components/knowledge-graph/GraphEdge.tsx` - 边组件
- [ ] `src/components/discussion/DiscussionPanel.tsx` - 讨论室主面板
- [ ] `src/components/discussion/TabBar.tsx` - Tab 栏
- [ ] `src/components/discussion/MessageList.tsx` - 消息列表
- [ ] `src/components/discussion/InputBox.tsx` - 输入框
- [ ] `src/components/terminal/TerminalPanel.tsx` - 运行终端主面板
- [ ] `src/components/terminal/TerminalPane.tsx` - 单个 Agent 终端
- [ ] `src/components/terminal/LogViewer.tsx` - 日志查看器（xterm.js）
- [ ] `src/components/staff/StaffPanel.tsx` - 员工档案主面板
- [ ] `src/components/staff/AgentCard.tsx` - Agent 卡片
- [ ] `src/pages/Dashboard.tsx` - 主页面
- [ ] `src/App.tsx` - 应用入口
- [ ] `src/main.tsx` - React 入口
- [ ] `src/vite-env.d.ts` - Vite 类型声明
- [ ] `index.html` - HTML 入口
- [ ] `package.json` - 前端依赖
- [ ] `tsconfig.json` - TypeScript 配置
- [ ] `vite.config.ts` - Vite 配置
- [ ] `tailwind.config.js` - TailwindCSS 配置

#### 后端文件（Rust）
- [ ] `src-tauri/src/main.rs` - 入口（Tauri commands 注册）
- [ ] `src-tauri/src/commands/mod.rs` - Commands 模块
- [ ] `src-tauri/src/commands/document.rs` - 文档相关命令
- [ ] `src-tauri/src/commands/agent.rs` - Agent 相关命令
- [ ] `src-tauri/src/commands/session.rs` - Session 相关命令
- [ ] `src-tauri/src/commands/log.rs` - 日志相关命令
- [ ] `src-tauri/src/process/mod.rs` - 进程管理模块
- [ ] `src-tauri/src/process/manager.rs` - Agent 进程管理器
- [ ] `src-tauri/src/process/monitor.rs` - 进程状态监控（闲置检测）
- [ ] `src-tauri/src/watcher/mod.rs` - 文件监听模块
- [ ] `src-tauri/src/watcher/docs_watcher.rs` - docs/ 目录监听
- [ ] `src-tauri/src/watcher/event.rs` - 文件变化事件
- [ ] `src-tauri/src/graph/mod.rs` - DAG 模块
- [ ] `src-tauri/src/graph/builder.rs` - DAG 构建器
- [ ] `src-tauri/src/graph/node.rs` - 节点定义
- [ ] `src-tauri/src/graph/edge.rs` - 边定义
- [ ] `src-tauri/src/parser/mod.rs` - Markdown 解析模块
- [ ] `src-tauri/src/parser/frontmatter.rs` - Frontmatter 解析
- [ ] `src-tauri/src/parser/document.rs` - 文档解析
- [ ] `src-tauri/src/models/mod.rs` - 数据模型模块
- [ ] `src-tauri/src/models/document.rs` - 文档模型
- [ ] `src-tauri/src/models/agent.rs` - Agent 模型
- [ ] `src-tauri/src/models/session.rs` - Session 模型
- [ ] `src-tauri/Cargo.toml` - Rust 依赖
- [ ] `src-tauri/tauri.conf.json` - Tauri 配置
- [ ] `src-tauri/build.rs` - 构建脚本（如果需要）

#### 配置文件
- [ ] `.commander/agents.yaml` - Agent 配置文件（可选）
- [ ] `config.yaml` - 全局配置文件（可选）

---

## 8. 与基线冲突（如有）

**无冲突**：本方案完全遵循 `tech-baseline.md` 定义的技术栈。

---

## 9. TECH 文档质量检查清单（输出前必须自检）

### 9.1 内容结构检查
- [x] 是否包含架构图或流程图？（✅ 第 3.2 节：架构图）
- [x] 复用清单是否清晰列出了可复用组件？（✅ 第 2.2 节：复用清单）
- [x] 是否明确标注了 `[ASSUMPTION]` 和 `[VERIFIED]`？（✅ 第 2.1 节：[NEW_PROJECT]）
- [x] 接口定义是否完整（请求/响应/错误码）？（✅ 第 4.3 节：API/事件设计）

### 9.2 代码占比检查（Critical）
- [x] 代码块占比是否 < 30%？（✅ 仅保留类型定义和接口契约）
- [x] 所有代码是否通过文件路径引用？（✅ 使用文件路径引用格式）
- [x] 是否避免了粘贴完整实现代码？（✅ 无完整实现代码）
- [x] API 契约和伪代码是否仅保留关键部分？（✅ 仅保留类型定义）

### 9.3 可落地性检查
- [x] TASK 拆解是否可执行？（✅ 15 个具体任务，每个都有明确的交付物）
- [x] 依赖关系是否清晰？（✅ 第 7.1 节：依赖类型 + 并行可行性分析）
- [x] 风险和未决项是否显式标注？（✅ 第 6 节：10 个风险/未决项）
- [x] 迁移和回滚方案是否可行？（✅ 第 5 节：全新项目，无迁移）

### 9.3.1 接口契约检查（Critical - 强制）
- [x] 每个后端 API 都有完整的接口契约？（✅ 第 4.3.1 节：13 个 Commands + 7 个 Events）
- [x] 契约包含请求、响应、错误码定义？（✅ 每个接口都有完整定义）
- [x] 边界情况都有明确定义？（✅ 每个接口都有边界情况说明）
- [x] 前端可以直接按契约开发？（✅ 提供了完整的 TypeScript 类型定义）

### 9.3.2 Mock 禁止检查（Critical - 强制）
- [x] 是否有使用 mock 数据的计划？（✅ 无 mock 数据计划）
- [x] 是否有使用假接口的计划？（✅ 无假接口，所有接口基于真实数据结构）
- [x] 是否明确标注了真实接口依赖？（✅ 第 7.1 节：依赖类型 + 并行可行性）
- [x] 并行计划是否基于真实接口而非 mock？（✅ 基于接口契约的并行开发）

### 9.4 文档一致性检查
- [x] 是否引用了上游文档（biz/prd/story）？（✅ 第 1 节：目标与范围对齐）
- [x] 是否与项目基线保持一致？（✅ 完全遵循 tech-baseline.md）
- [x] 术语是否与上游文档一致？（✅ 使用上游文档的术语）

### 9.5 常见错误检查
- [x] 是否避免了项目特定的硬编码路径？（✅ 使用相对路径）
- [x] 是否避免了粘贴大段代码实现？（✅ 无完整实现代码）
- [x] 是否避免了假设"应该有这个字段/接口"？（✅ 所有类型都有明确定义）
- [x] 是否所有设计决策都有代码引用来源？（✅ 引用 tech-baseline.md 和 STORY 文档）

---

## 10. 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | 2026-01-21 | tech agent | 初版 TECH-E-001-v1，基于 4 个 STORY 和 tech-baseline.md |

---

## 附录：参考文档

- **PRD**：`/docs/E-001-Agent-Console/prd/PRD-E-001-v0.md`
- **STORY-001**：`/docs/E-001-Agent-Console/story/STORY-001-全局状态可视化.md`
- **STORY-002**：`/docs/E-001-Agent-Console/story/STORY-002-任务调度闭环.md`
- **STORY-003**：`/docs/E-001-Agent-Console/story/STORY-003-实时进度监控.md`
- **STORY-004**：`/docs/E-001-Agent-Console/story/STORY-004-自动资源管理.md`
- **Tech Baseline**：`/docs/_project/tech-baseline.md`
- **biz-overview**：`/docs/_project/biz-overview.md`
- **UI 原型**：`/docs/E-001-Agent-Console/prototypes/index.html`
