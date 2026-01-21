# 技术基线（Tech Baseline）

> **项目**：VibedevTeam - 多 AI Dev 并行协作平台
> **最后更新**：2026-01-20
> **状态**：Phase 1 验证期

---

## 📋 文档目的

本文档定义项目级技术基线，包括：
- 技术栈选型（语言、框架、依赖管理）
- 开发规范（API、DB、日志、可观测性）
- 质量门槛（测试覆盖率、代码审查）

**适用范围**：整个项目（Phase 1 验证期及未来演进）

**包管理器说明**：
- **前端**：统一使用 `pnpm`（更快、更省空间、更严格）
- **Rust**：使用 `cargo`（Rust 内置包管理器）

**更新原则**：
- Phase 1 优先"快速验证"：选择最成熟、学习成本最低的方案
- Phase 2+ 如需调整，通过 ADR 记录决策理由

---

## 🎯 Phase 1 目标

**核心目标**：验证"多 AI 并行协作价值"，让用户能舒适管理 4 个并行 dev。

**质量门槛**：
- ✅ 能跑、不崩、能验证核心价值
- ✅ 代码清晰（遵守 YAGNI 和 KISS 原则）
- ✅ 测试覆盖率目标 50%（非 70%）
- ✅ 单文件不超过 600 行

**不做的事**（Phase 2 再考虑）：
- ❌ 远程控制功能（Happy 已存在）
- ❌ 复杂数据库（文件系统 + 内存缓存够用）
- ❌ 过度抽象和可扩展性（先验证价值）

---

## 🛠️ 技术栈总览

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
│  HTTP 服务：axum（可选，Phase 1 不需要）                   │
│  进程管理：std::process + 人工监听                        │
│  文件监听：notify                                         │
├─────────────────────────────────────────────────────────┤
│  数据层：文件系统（docs/ 目录）                            │
│  格式：Markdown + Frontmatter                             │
│  缓存：内存 DAG（有向无环图）                              │
└─────────────────────────────────────────────────────────┘
```

---

## 📦 详细技术选型

### 1. 桌面框架：Tauri v2

**选型**：Tauri v2 (Rust + Webview)

**理由**：
- ✅ 系统级能力：可启动/杀死进程（Electron 做不到）
- ✅ 跨平台：macOS + Linux + Windows
- ✅ 打包体积小（< 10MB，Electron > 100MB）
- ✅ 安全性高（Rust 内存安全）
- ✅ 学习成本低：前端用熟悉的 Web 技术栈

**替代方案**：
- Electron：体积大、无系统级能力 ❌
- Flutter：学习成本高、生态不如 Web ❌

**文档**：https://tauri.app/v2/guides/

---

### 2. 前端框架：React + TypeScript + Vite

**选型**：React 18+ + TypeScript 5+ + Vite 5+

**理由**：
- ✅ 生态最成熟：组件库、工具链、教程最多
- ✅ 学习成本低：社区资源丰富，遇到问题容易搜到答案
- ✅ TypeScript：类型安全，减少运行时错误
- ✅ Vite：开发体验好（HMR 快、构建快）

**替代方案**：
- Vue：生态不如 React，组件库少 ❌
- Svelte：生态不成熟，招聘困难 ❌

**文档**：
- React：https://react.dev/
- TypeScript：https://www.typescriptlang.org/docs/
- Vite：https://vitejs.dev/guide/

---

### 3. UI 组件库：TailwindCSS + Shadcn/UI

**选型**：TailwindCSS 3+ + Shadcn/UI

**理由**：
- ✅ TailwindCSS：不用写 CSS 文件，类名即样式
- ✅ Shadcn/UI：复制代码到项目，完全可控（非第三方包）
- ✅ 现代化设计：符合 2026 年审美
- ✅ 可定制性强：不像 Ant Design 那样"黑盒"

**示例代码**：
```tsx
// 安装：pnpm add -D tailwindcss @tailwindcss/forms
// 复制组件：npx shadcn-ui@latest add button

import { Button } from '@/components/ui/button';

<Button variant="default">点击我</Button>
```

**文档**：
- TailwindCSS：https://tailwindcss.com/docs/installation
- Shadcn/UI：https://ui.shadcn.com/

---

### 4. 状态管理：Zustand

**选型**：Zustand 4+

**理由**：
- ✅ API 简单：比 Redux 少 80% 样板代码
- ✅ 无需 Provider：直接 import 使用
- ✅ TypeScript 支持好：类型推导自动
- ✅ 轻量：打包体积 < 1KB

**示例代码**：
```typescript
import { create } from 'zustand';

interface AgentStore {
  agents: Agent[];
  addAgent: (agent: Agent) => void;
}

const useAgentStore = create<AgentStore>((set) => ({
  agents: [],
  addAgent: (agent) => set((state) => ({ agents: [...state.agents, agent] })),
}));

// 在组件中使用
function AgentList() {
  const agents = useAgentStore((state) => state.agents);
  return <div>{agents.map(...)}</div>;
}
```

**替代方案**：
- Redux Toolkit：样板代码多，学习成本高 ❌
- React Context：性能差，不适合高频更新 ❌

**文档**：https://zustand-demo.pmnd.rs/

---

### 5. 知识图谱可视化：Cytoscape.js

**选型**：Cytoscape.js 3+

**理由**：
- ✅ API 简单：像 jQuery 一样直观，20 分钟上手
- ✅ 内置树形布局：`breadthfirst` 或 `dagre` 开箱即用
- ✅ 性能好：能处理 10 万节点（Phase 1 不会有超过 1000 个）
- ✅ 文档清晰：例子多，社区活跃

**示例代码**：
```typescript
import cytoscape from 'cytoscape';

const cy = cytoscape({
  container: document.getElementById('graph'),
  elements: [
    { data: { id: 'E-001', label: 'Epic 1' } },
    { data: { id: 'STORY-001', label: 'Story 1' } },
    { data: { source: 'STORY-001', target: 'E-001' } }
  ],
  layout: { name: 'breadthfirst' },  // 树形布局
  style: [
    {
      selector: 'node',
      style: { 'background-color': '#666', 'label': 'data(label)' }
    }
  ]
});
```

**替代方案**：
- D3.js：学习曲线陡峭，代码量多 5-10 倍 ❌
- vis.js：性能不如 Cytoscape ❌

**文档**：https://js.cytoscape.org/

---

### 6. 终端模拟器：xterm.js + node-pty

**选型**：xterm.js 5+ + node-pty（本地 Node.js 服务）

**架构**：
```
前端 (xterm.js 渲染)
    ↓ WebSocket
Tauri 后端 (Rust)
    ↓ 启动进程
Node.js 服务 (node-pty 创建 PTY)
    ↓ PTY
Shell (bash/zsh)
```

**理由**：
- ✅ `node-pty` 是 VS Code 同款，被千万人验证过
- ✅ 开发速度快：Node.js 生态熟悉
- ✅ 稳定性高：VS Code 用了 10 年

**代价**：
- ❌ 打包体积增加 20MB+（桌面应用可接受）

**替代方案**：
- Rust 原生 PTY：学习成本高，生态不成熟 ❌

**文档**：
- xterm.js：https://xtermjs.org/docs/
- node-pty：https://nodepty.org/

---

### 7. 前端路由：React Router v7

**选型**：React Router v7

**理由**：
- ✅ 最成熟：React 生态标准路由库，用的人最多
- ✅ 文档最全：问题最容易搜到答案
- ✅ 学习成本低：API 简单直观

**示例代码**：
```tsx
import { BrowserRouter, Routes, Route } from 'react-router';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/agents/:id" element={<AgentDetail />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**替代方案**：
- TanStack Router：太新，文档少 ❌

**文档**：https://reactrouter.com/

---

### 8. 日期处理：dayjs

**选型**：dayjs 1+

**理由**：
- ✅ API 和 Moment.js 一样：如果用过 Moment，无缝迁移
- ✅ 轻量：打包体积 2KB（Moment.js 67KB）
- ✅ 不可变数据：避免 bug
- ✅ TypeScript 支持好

**示例代码**：
```typescript
import dayjs from 'dayjs';

const now = dayjs();
const formatted = now.format('YYYY-MM-DD HH:mm:ss');
const diff = now.diff('2026-01-01', 'day');
```

**替代方案**：
- date-fns：函数式风格，学习成本高 ❌
- Moment.js：已废弃，体积大 ❌

**文档**：https://day.js.org/

---

### 9. Rust HTTP 服务器：axum（可选）

**选型**：axum 0.7+

**理由**：
- ✅ 官方推荐：由 Tokio 团队维护（Tokio 是 Rust 异步运行时标准）
- ✅ API 简单：Extractor 模式易理解
- ✅ 生态好：和 Tower/Tokio 无缝集成
- ✅ 学习成本低：文档最全，例子最多

**示例代码**：
```rust
use axum::{Router, routing::get, Json};

async fn hello() -> Json<&'static str> {
    Json("Hello, World!")
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(hello));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();

    axum::serve(listener, app).await.unwrap();
}
```

**注意**：Phase 1 可能不需要 HTTP 服务器（除非做远程控制）

**替代方案**：
- actix-web：性能更好，但 API 复杂 ❌

**文档**：https://docs.rs/axum/

---

### 10. 数据层：文件系统

**选型**：文件系统（监听 `docs/` 目录）+ 内存缓存

**架构**：
```
docs/ 目录
    ├── E-001-xxx/
    │   ├── prd/PRD-E-001-v1.md
    │   ├── story/STORY-*.md
    │   └── task/TASK-*.md
    └── ...

↓ Rust notify 库监听文件变化

↓ 解析 Markdown Frontmatter

↓ 内存 DAG（有向无环图）
```

**理由**：
- ✅ 简单：无需数据库配置、迁移、备份
- ✅ Git 作为天然备份：版本控制、回滚容易
- ✅ Phase 1 够用：不会有超过 1000 个文档

**Rust 依赖**：
```toml
[dependencies]
notify = "6.1"           # 文件监听
serde = { version = "1.0", features = ["derive"] }
serde_yaml = "0.9"       # 解析 Frontmatter
```

**注意**：Phase 2 如果需要复杂查询（如"查找所有延期任务"），再考虑嵌入 SQLite

---

### 11. 测试框架

**选型**：
- 前端：Vitest
- Rust：cargo test（内置）

**理由**：
- ✅ Vitest：和 Vite 无缝集成，速度快，API 和 Jest 类似
- ✅ cargo test：Rust 内置，无需配置

**覆盖率目标**：
- Phase 1：50%（核心业务逻辑）
- Phase 2+：70%（全量）

**文档**：
- Vitest：https://vitest.dev/
- cargo test：https://doc.rust-lang.org/book/ch11-00-testing.html

---

## 📐 开发规范

### API 契约规范（如果 Phase 2 暴露 HTTP API）

**RESTful 风格**（带版本号）：
```
GET    /api/v1/agents          # 列出所有 Agent
GET    /api/v1/agents/:id      # 获取单个 Agent
POST   /api/v1/agents          # 创建 Agent
PUT    /api/v1/agents/:id      # 更新 Agent
DELETE /api/v1/agents/:id      # 删除 Agent
POST   /api/v1/agents/:id/start  # 启动 Agent
POST   /api/v1/agents/:id/kill   # 杀死 Agent
```

**响应格式**（成功）：
```json
{
  "code": 0,
  "data": { "id": "agent-1", "status": "running" },
  "message": "success"
}
```

**响应格式**（失败）：
```json
{
  "code": 1001,
  "data": null,
  "message": "Agent not found"
}
```

**错误码规范**：
```typescript
// 成功
0: 成功

// 客户端错误（1000-1999）
1001: Agent not found
1002: Invalid parameters
1003: Agent already running
1004: Agent not running
1005: Missing required field

// 服务器错误（5000-5999）
5001: Internal server error
5002: Failed to start process
5003: File system error
5004: Process timeout
```

**HTTP 状态码**：
- 200：成功（包括 `code = 0`）
- 400：客户端错误（`code` 在 1000-1999）
- 404：资源不存在（`code = 1001`）
- 500：服务器错误（`code` 在 5000-5999）

---

### 日志规范

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

---

### 代码组织规范

**前端目录结构**：
```
src/
  ├── components/     # 可复用组件
  │   ├── ui/         # Shadcn/UI 组件
  │   └── agents/     # Agent 相关组件
  ├── pages/          # 页面组件
  ├── stores/         # Zustand stores
  ├── lib/            # 工具函数
  ├── types/          # TypeScript 类型定义
  └── App.tsx
```

**Rust 目录结构**：
```
src-tauri/
  ├── src/
  │   ├── main.rs           # 入口
  │   ├── commands/         # Tauri commands
  │   ├── process/          # 进程管理
  │   ├── watcher/          # 文件监听
  │   └── graph/            # DAG 构建
  └── Cargo.toml
```

---

### 命名规范

**TypeScript**：
- 组件：PascalCase（`AgentList.tsx`）
- 函数/变量：camelCase（`createAgent`）
- 常量：UPPER_SNAKE_CASE（`MAX_AGENTS`）
- 接口/类型：PascalCase（`interface AgentStore`）

**Rust**：
- 函数/变量：snake_case（`create_agent`）
- 结构体/枚举：PascalCase（`struct Agent`）
- 常量：UPPER_SNAKE_CASE（`MAX_AGENTS: usize`）

---

## 🔒 安全规范

### 认证与授权（Phase 2 如需远程控制）

**方案**：
- JWT Token：用户登录后返回 Token
- API 鉴权：每个请求 Header 带 `Authorization: Bearer <token>`
- 权限控制：RBAC（Role-Based Access Control）

**示例**：
```typescript
// 前端请求
fetch('/api/agents', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

---

### 数据加密

**本地数据**（文件系统）：
- ✅ 无需加密（用户本地文件）
- ✅ Git 作为备份（云端由 GitHub/GitLab 加密）

**远程通信**（Phase 2 如需）：
- ✅ 强制 HTTPS（TLS 1.3）
- ✅ WebSocket 用 WSS（加密 WebSocket）

---

## 📊 可观测性规范

### 日志收集（Phase 2）

**方案**：
- 前端：集成 Sentry（错误监控）
- Rust：集成 tracing（结构化日志）

**示例**：
```typescript
// 前端错误上报
import * as Sentry from '@sentry/react';

Sentry.captureException(error);
```

```rust
// Rust 结构化日志
use tracing::{info, error, Level};

info!(agent_id = %id, "启动 Agent");
error!(error = %err, "启动失败");
```

---

### 指标监控（Phase 2）

**关键指标**：
- Agent 数量：当前运行的 Agent 数
- 响应时间：API 响应时间（P50/P95/P99）
- 错误率：Agent 启动失败率

**实现**：Prometheus + Grafana（可选）

---

## 🚀 部署规范

### 打包与分发

**工具**：Tauri CLI

**命令**：
```bash
# 开发模式
pnpm run tauri dev    # 或省略 run：pnpm tauri dev

# 构建（打包成 .app / .exe / .deb）
pnpm run tauri build  # 或省略 run：pnpm tauri build
```

**产物**：
- macOS：`.app`（拖拽到 Applications 即可）
- Linux：`.deb` / `.AppImage`
- Windows：`.exe` / `.msi`

---

### 自动更新（Phase 2）

**方案**：Tauri 内置更新器

**配置**：
```rust
// src-tauri/tauri.conf.json
{
  "tauri": {
    "updater": {
      "active": true,
      "endpoints": ["https://releases.yourdomain.com/{{target}}/{{current_version}}"]
    }
  }
}
```

---

## 📚 依赖管理

### 前端依赖管理

**包管理器**：pnpm（本项目统一使用 pnpm）

**为什么选择 pnpm？**
- ✅ 更快（硬链接节省安装时间）
- ✅ 更省空间（避免重复下载）
- ✅ 更严格（避免幽灵依赖）

**版本策略**：
- 生产依赖：锁定版本（如 `react@18.3.1`）
- 开发依赖：允许补丁更新（如 `@types/react@^18.3.1`）

**常用命令**：
```bash
# 安装依赖
pnpm install

# 添加依赖
pnpm add react
pnpm add -D @types/react

# 更新依赖
pnpm update

# 运行脚本（可省略 run）
pnpm run tauri dev   # 或 pnpm tauri dev
pnpm run tauri build # 或 pnpm tauri build
```

---

### Rust 依赖管理

**包管理器**：Cargo

**版本策略**：
- 生产依赖：锁定版本（`Cargo.lock`）
- 允许补丁更新（`1.0.x` → `1.0.1`）

**命令**：
```bash
# 添加依赖
cargo add notify

# 更新依赖
cargo update

# 检查过期依赖
cargo outdated
```

---

## 🎓 学习资源

**必读文档**（按优先级）：
1. Tauri 官方教程：https://tauri.app/v2/guides/
2. React 官方教程：https://react.dev/learn
3. Shadcn/UI 组件示例：https://ui.shadcn.com/examples
4. Cytoscape.js 入门：https://js.cytoscape.org/#getting-started
5. Zustand 示例：https://zustand-demo.pmnd.rs/

**社区资源**：
- Tauri Discord：https://discord.gg/tauri
- Rust Discord：https://discord.gg/rust-lang
- Stack Overflow：[tauri] 标签

---

## 🔄 版本历史

| 版本 | 日期 | 变更内容 | 作者 |
|------|------|---------|------|
| v1.1 | 2026-01-20 | 修正 API 规范：添加版本号 v1，响应格式改用 code/message，统一使用 pnpm | tech agent |
| v1.0 | 2026-01-20 | 初始版本，定义 Phase 1 技术基线 | tech agent |

---

## 📌 附录

### A. 技术选型决策记录（ADR）

当前无偏离基线的决策。

如需调整技术栈，在 `/docs/_project/adr/` 下创建 ADR，说明：
- 为什么需要调整？
- 替代方案有哪些？
- 为什么选择这个方案？
- 对现有代码的影响？

### B. Phase 2 技术演进方向

**可能的技术升级**（Phase 2 根据需求决定）：
- 数据库：文件系统 → SQLite（复杂查询需求）
- 远程控制：本地 → Cloudflare Tunnel（公网访问）
- 实时同步：Git 手动同步 → WebSocket 自动同步
- 测试覆盖率：50% → 70%（生产级质量）

**不变的原则**：
- 代码清晰 > 聪明
- 增量演进 > 全部重构
- 成熟方案 > 实验性技术
