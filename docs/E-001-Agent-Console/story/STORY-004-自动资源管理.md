# STORY-004：自动资源管理

> **文档路径**：`/docs/E-001-Agent-Console/story/STORY-004-自动资源管理.md`
>
> * **STORY ID**：STORY-004
> * **EPIC_ID**：E-001
> * **EPIC_DIR**：`E-001-Agent-Console`
> * **关联 PRD**：`../prd/PRD-E-001-v0.md#场景4-闲置自动释放`
> * **状态**：待开发
> * **优先级**：P1（高）
> * **创建日期**：2026-01-20
> * **预计完成**：[OPEN]

---

## 1. User Story 文本

**作为一个** 开发者（用户自己）

**我想要** 系统自动释放闲置的 Agent 进程，不需要手动管理

**以便于** 我能从"手动管理进程"到"自动回收资源"，不用担心内存浪费，闲置 Agent 自动释放，重新使用时自动恢复

---

## 2. 背景与动机

### 2.1 痛点（来自 PRD 场景 4）

**P3：进程管理复杂**
- 之前得手动 kill 进程释放内存
- 闲置进程占用内存，得手动管理
- 当前成本：需要手动启动/kill Agent，手动管理 Session

**P1：认知过载**
- 需要记住哪些 Agent 闲置了
- 需要记得手动释放资源

### 2.2 当前成本

- **时间成本**：每次手动 kill 进程需要 10-20 秒
- **认知负荷**：需要记住哪些 Agent 闲置了
- **内存浪费**：闲置进程占用内存（每个进程约 50-100MB）

### 2.3 预期价值

- ✅ 闲置释放方式：从手动 kill 进程 → 自动释放
- ✅ Session 恢复方式：从手动指定 JSONL → 自动恢复
- ✅ 内存浪费：闲置进程占用内存 → 自动释放
- ✅ 误杀率：< 1%（频繁误杀正在使用的 Agent）

---

## 3. 主要使用场景（主路径步骤）

### 3.1 场景：从"手动管理"到"自动回收"

**触发条件**：下午 16:00，Dave-2 完成了 TASK-02，状态变为"空闲"

**主路径**：

1. **Agent 完成任务**
   - Dave-2 完成了 TASK-02，状态变为"空闲"（绿色）
   - 开始计时（闲置时间）
   - **成功标志**：员工档案中 Dave-2 状态变为"空闲"

2. **用户继续忙别的**
   - 用户在忙其他事情（如查看其他 Agent 的日志）
   - 没有给 Dave-2 分派新任务
   - **成功标志**：用户不需要手动操作

3. **系统检测到闲置超过 5 分钟**
   - 系统检测到 Dave-2 闲置超过 5 分钟
   - 自动释放 Dave-2 的进程（发送 SIGTERM）
   - **成功标志**：进程成功终止

4. **员工档案更新**
   - 员工档案中 Dave-2 状态变为"已释放"（灰色）
   - 对话历史正确保存到 JSONL 文件
   - **成功标志**：状态更新，Session 保存成功

5. **用户重新分派任务给 Dave-2**
   - 用户给 Dave-2 分派新任务
   - 系统检测到 Dave-2 已释放
   - **成功标志**：提示"Agent 已释放，是否自动启动？"

6. **系统自动启动并恢复 Session**
   - 系统自动启动 Dave-2 的进程
   - 从 JSONL 文件恢复对话历史
   - **成功标志**：Agent 启动成功，对话历史完整恢复

**完成标志**：从"手动管理"到"自动回收"，用户不需要手动操作

---

## 4. 状态机（关键状态与跃迁）

### 4.1 Agent 生命周期的状态机

```
┌──────────┐  启动Agent  ┌──────────┐  完成任务  ┌──────────┐
│  已释放   │ ────────> │  空闲态   │ ────────> │  闲置态   │
│ (未运行)  │           │ (IDLE)   │           │ (计时中) │
└──────────┘           └──────────┘           └──────────┘
                                                  │
                                                  │ 闲置>5分钟
                                                  ▼
                                          ┌──────────┐
                                          │  释放态   │
                                          │(释放进程) │
                                          └──────────┘
                                                  │
                                                  │ 释放完成
                                                  ▼
                                          ┌──────────┐
                                          │  已释放   │
                                          │ (未运行)  │
                                          └──────────┘
```

**关键跃迁**：
- **已释放 → 空闲态**：用户启动 Agent（手动或自动）
- **空闲态 → 忙碌态**：用户分派任务给 Agent
- **忙碌态 → 空闲态**：Agent 完成任务
- **空闲态 → 闲置态**：Agent 空闲超过 1 分钟，开始计时
- **闲置态 → 释放态**：Agent 空闲超过 5 分钟，开始释放
- **释放态 → 已释放**：进程释放完成

**恢复路径**：
- 如果用户在闲置期间分派任务 → 取消计时，回到忙碌态
- 如果进程释放失败 → 记录错误日志，保持闲置态
- 如果 Session 恢复失败 → 启动新 Session（失去历史对话）

### 4.2 Session 恢复的状态机

```
┌──────────┐  启动Agent  ┌──────────┐  读取JSONL  ┌──────────┐
│  初始态   │ ────────> │  加载态   │ ─────────> │  恢复态   │
│ (无Session)│          │(读取配置) │            │(恢复历史) │
└──────────┘           └──────────┘           └──────────┘
                                                  │
                                                  │ 恢复成功
                                                  ▼
                                          ┌──────────┐
                                          │  完成态   │
                                          │(可用状态) │
                                          └──────────┘
```

**关键跃迁**：
- **初始态 → 加载态**：Agent 启动，开始读取 Session 配置
- **加载态 → 恢复态**：找到 JSONL 文件，开始恢复对话历史
- **恢复态 → 完成态**：对话历史恢复完成，Agent 可用

**恢复路径**：
- 如果 JSONL 文件不存在 → 跳到完成态（启动新 Session）
- 如果 JSONL 文件损坏 → 提示"Session 损坏，是否启动新 Session？"
- 如果恢复超时（10 秒） → 提示"Session 恢复超时，是否启动新 Session？"

---

## 5. 验收标准（Acceptance Criteria）

### AC1：Agent 闲置 5 分钟后自动释放

**步骤**：
1. 用户启动一个 Agent（如 Dave-1）
2. 用户不给 Agent 分派任何任务
3. 用户观察员工档案
4. 用户等待 5 分钟

**期望结果**：
- ✅ Agent 在 5 分钟后自动释放
- ✅ Agent 状态变为"已释放"（灰色）
- ✅ 进程终止（通过任务管理器验证）
- ✅ 对话历史正确保存到 JSONL 文件
- ✅ 释放过程不阻塞用户操作（后台进行）

**测试方式**：手工测试 + 计时 + 进程监控

---

### AC2：Agent 闲置检测准确，误杀率 < 1%

**步骤**：
1. 用户启动一个 Agent（如 Dave-1）
2. 用户在讨论室中查看 Dave-1 的对话历史
3. 用户等待 5 分钟（期间保持查看对话）
4. 用户观察 Agent 是否被释放

**期望结果**：
- ✅ Agent **不会**被释放（因为用户正在使用）
- ✅ 闲置计时在用户查看对话时暂停
- ✅ 用户停止查看后，计时继续
- ✅ 误杀率 < 1%（100 次测试中，误杀 < 1 次）

**测试方式**：手工测试 + 统计误杀率

---

### AC3：Session 自动恢复，恢复成功率 > 95%

**步骤**：
1. 用户启动一个 Agent（如 Dave-1）
2. 用户和 Agent 对话几轮
3. 用户关闭应用（模拟退出）
4. 用户重新打开应用
5. 用户给 Dave-1 分派新任务

**期望结果**：
- ✅ Agent 自动启动
- ✅ 对话历史完整恢复（之前的对话内容都能看到）
- ✅ Session 恢复成功率 > 95%（100 次测试中，成功 > 95 次）
- ✅ 恢复时间 < 3 秒

**测试方式**：手工测试 + 统计成功率 + 计时

---

### AC4：用户能手动启动/停止 Agent

**步骤**：
1. 用户在员工档案中找到已释放的 Agent（如 Dave-2）
2. 用户点击"启动"按钮
3. 等待 Agent 启动
4. 用户点击"停止"按钮
5. 等待 Agent 停止

**期望结果**：
- ✅ Agent 在 3 秒内启动完成
- ✅ Agent 状态变为"空闲"
- ✅ Agent 在 2 秒内停止完成
- ✅ Agent 状态变为"已释放"
- ✅ 手动操作不影响自动释放机制

**测试方式**：手工测试 + 计时

---

### AC5：Session 损坏时，提供修复方案

**步骤**：
1. 用户手动破坏 JSONL 文件（模拟损坏）
2. 用户尝试启动 Agent
3. 用户观察错误提示

**期望结果**：
- ✅ 显示"Session 损坏，是否启动新 Session？"
- ✅ 提供两个选项：
  - 选项 A："启动新 Session"（失去历史对话）
  - 选项 B："尝试修复 Session"
- ✅ 如果用户选择"启动新 Session"，Agent 正常启动（只是没有历史对话）
- ✅ 如果用户选择"尝试修复 Session"，尝试修复 JSONL 文件

**测试方式**：手工测试 + 故意破坏 JSONL 文件

---

### AC6：自动释放不影响其他 Agent

**步骤**：
1. 用户启动 4 个 Agent（Tiger、Dave-1、Dave-2、Terry）
2. 用户让 Dave-1 闲置超过 5 分钟
3. 用户观察其他 Agent 的状态

**期望结果**：
- ✅ Dave-1 被自动释放
- ✅ 其他 Agent（Tiger、Dave-2、Terry）不受影响
- ✅ 其他 Agent 继续正常运行
- ✅ 应用不会崩溃或卡顿

**测试方式**：手工测试 + 进程监控

---

## 6. 边界与异常

### 6.1 Agent 闲置期间用户正在查看对话

**场景**：Agent 闲置超过 5 分钟，但用户正在查看对话历史

**处理方式**：
- 暂不释放，延长计时
- 提示："Dave-2 闲置中，是否继续释放？"
- 提供"继续释放"和"取消释放"按钮

**验收标准**：
- ✅ 不会在用户查看对话时释放 Agent
- ✅ 用户可以选择是否释放

---

### 6.2 Session 恢复失败

**场景**：JSONL 文件损坏，无法恢复对话历史

**错误提示**：显示"Session 损坏，是否启动新 Session？"

**恢复路径**：
- **选项 A**："启动新 Session"
  - Agent 正常启动，但没有历史对话
- **选项 B**："尝试修复 Session"
  - 尝试修复 JSONL 文件（如删除损坏的行）
  - 如果修复失败，提示"无法修复，是否启动新 Session？"

**验收标准**：
- ✅ 不会因为 Session 损坏而无法启动 Agent
- ✅ 提供清晰的恢复路径

---

### 6.3 进程释放失败

**场景**：Agent 进程无法正常终止（如死锁）

**错误提示**：显示"Agent 进程无法终止"

**恢复路径**：
- 提供"强制杀死"按钮（发送 SIGKILL）
- 提供"查看进程状态"按钮（显示 CPU、内存使用情况）
- 如果强制杀死失败，记录错误日志，提示用户手动处理

**验收标准**：
- ✅ 不会因为进程无法终止而阻塞应用
- ✅ 提供"强制杀死"选项

---

### 6.4 Session 文件过大

**场景**：对话历史很长，JSONL 文件超过 10MB

**降级方案**：
- 显示"Session 文件过大，只恢复最近 100 条对话"
- 只恢复最近 100 条对话（按时间戳排序）
- 提供"恢复完整历史"按钮（加载全部对话，可能需要较长时间）

**验收标准**：
- ✅ 不会因为 Session 文件过大而无法恢复
- ✅ 提供降级方案，保证 Agent 能快速启动

---

### 6.5 并发启动多个 Agent

**场景**：用户同时启动 4 个 Agent

**处理方式**：
- 依次启动（避免同时启动导致资源争抢）
- 显示启动进度（如"正在启动 3/4..."）
- 如果某个 Agent 启动失败，不影响其他 Agent

**验收标准**：
- ✅ 所有 Agent 都能成功启动
- ✅ 启动过程不阻塞应用
- ✅ 启动失败不影响其他 Agent

---

### 6.6 闲置时间阈值可配置

**场景**：用户希望自定义闲置时间阈值（如改为 10 分钟）

**处理方式**：
- 提供配置文件（`config.yaml`）
- 用户可以修改 `idle_timeout` 字段（单位：分钟）
- 默认值为 5 分钟

**配置文件格式**：
```yaml
agent_management:
  idle_timeout: 5  # 闲置时间阈值（分钟）
  auto_release: true  # 是否自动释放
```

**验收标准**：
- ✅ 用户可以自定义闲置时间阈值
- ✅ 配置文件修改后，重启应用生效

---

## 7. 接口契约草案（Interface Contract Draft）

> **说明**：本章节用自然语言描述接口的业务字段含义和边界条件。

### 7.1 Rust → 前端：Agent 闲置检测推送

**触发时机**：Agent 闲置时间超过阈值（默认 5 分钟）

**请求结构**（Rust → 前端，通过 Tauri Event）：
```yaml
event_name: "agent:idle_timeout"

payload:
  agent_id: "dave-1"
  idle_duration: 300                        # 闲置时长（秒）
  idle_threshold: 300                       # 闲置阈值（秒），默认 300 秒（5 分钟）
  will_release: true                        # 是否即将释放
  release_after: 0                          # 多少秒后释放（0 表示立即释放）
  timestamp: "2026-01-20T16:05:00Z"
```

**边界条件**：
- **闲置时间 < 阈值**：不推送事件（避免频繁推送）
- **will_release = false**：Agent 不会释放（如用户正在查看对话）

**性能要求**：
- 推送延迟 < 1 秒（从闲置超时到前端收到）

---

### 7.2 Rust → 前端：Agent 状态变更推送

**触发时机**：Agent 状态变化（如从 IDLE → RELEASED）

**请求结构**（Rust → 前端，通过 Tauri Event）：
```yaml
event_name: "agent:status_changed"

payload:
  agent_id: "dave-1"
  old_status: "IDLE"
  new_status: "RELEASED"                    # 状态：IDLE / BUSY / RELEASED
  reason: "idle_timeout"                    # 状态变化原因（可选）
  session_saved: true                       # Session 是否保存成功
  timestamp: "2026-01-20T16:05:00Z"
```

**边界条件**：
- **状态不变**：不推送事件（避免冗余）
- **reason 为空**：返回 `reason: null`

**性能要求**：
- 推送延迟 < 500ms（从状态变化到前端收到）

---

### 7.3 前端 → Rust：手动启动 Agent

**触发时机**：用户点击"启动"按钮

**请求结构**（前端 → Rust，通过 Tauri Command）：
```yaml
command_name: "start_agent"

request:
  agent_id: "dave-1"
  restore_session: true                      # 是否从 JSONL 恢复对话历史
  session_path: "/sessions/dave-1.jsonl"    # Session 文件路径

response:
  success: true
  process_id: 12345                         # Agent 进程 ID
  session_restored: true                    # Session 是否恢复成功
  startup_time: 2500                        # 启动耗时（毫秒）
  error_message: null
```

**边界条件**：
- **Agent 已运行**：返回 `success: false, error_message: "Agent 已在运行"`
- **Session 文件不存在**：返回 `session_restored: false`，但不影响启动
- **Session 文件损坏**：返回 `session_restored: false, error_message: "Session 损坏"`

**性能要求**：
- 启动时间 < 3 秒（从点击到 Agent 可用）

---

### 7.4 前端 → Rust：手动停止 Agent

**触发时机**：用户点击"停止"按钮

**请求结构**（前端 → Rust，通过 Tauri Command）：
```yaml
command_name: "stop_agent"

request:
  agent_id: "dave-1"
  force: false                               # 是否强制杀死（SIGKILL）

response:
  success: true
  session_saved: true                       # Session 是否保存成功
  termination_time: 1200                    # 终止耗时（毫秒）
  error_message: null
```

**边界条件**：
- **Agent 未运行**：返回 `success: false, error_message: "Agent 未运行"`
- **进程无法终止**：返回 `success: false, error_message: "进程无法终止，请使用强制杀死"`
- **force = true**：发送 SIGKILL 信号

**性能要求**：
- 终止时间 < 2 秒（从点击到进程终止）

---

### 7.5 前端 → Rust：获取 Agent 闲置时间

**触发时机**：用户查看员工档案

**请求结构**（前端 → Rust，通过 Tauri Command）：
```yaml
command_name: "get_agent_idle_time"

request:
  agent_id: "dave-1"

response:
  success: true
  idle_time: 180                            # 闲置时长（秒）
  idle_threshold: 300                       # 闲置阈值（秒），默认 300 秒（5 分钟）
  will_release_soon: false                  # 是否即将释放（闲置时间 > 4 分钟）
  estimated_release_time: "2026-01-20T16:10:00Z"  # 预计释放时间（ISO 8601 格式）
  error_message: null
```

**边界条件**：
- **Agent 正在忙碌**：返回 `idle_time: 0, will_release_soon: false`
- **Agent 已释放**：返回 `idle_time: null`
- **无法获取闲置时间**：返回 `success: false, error_message: "无法获取闲置时间"`

**性能要求**：
- 响应时间 < 300ms（从点击到显示）

---

### 7.6 前端 → Rust：修复损坏的 Session

**触发时机**：用户选择"尝试修复 Session"

**请求结构**（前端 → Rust，通过 Tauri Command）：
```yaml
command_name: "repair_session"

request:
  agent_id: "dave-1"
  session_path: "/sessions/dave-1.jsonl"    # Session 文件路径

response:
  success: true
  repaired: true                            # 是否修复成功
  error_lines_removed: 2                    # 删除的错误行数
  valid_lines_restored: 48                  # 恢复的有效行数
  error_message: null
```

**边界条件**：
- **Session 文件不存在**：返回 `success: false, error_message: "Session 文件不存在"`
- **Session 文件未损坏**：返回 `repaired: false`（无需修复）
- **修复失败**：返回 `repaired: false, error_message: "无法修复 Session"`

**性能要求**：
- 修复时间 < 5 秒（从点击到修复完成）

---

## 8. UI 证据引用

### 8.1 原型链接

**可运行 HTML 原型**：`/docs/E-001-Agent-Console/prototypes/index.html`

### 8.2 关键页面截图

#### Agent 闲置提示

```
┌─────────────────────────────────────────────────────────┐
│  ⏰  Dave-2 闲置提示                                      │
│                                                          │
│  Dave-2 已闲置 5 分钟，即将自动释放资源                     │
│                                                          │
│  [继续释放] [取消释放]                                    │
└─────────────────────────────────────────────────────────┘
```

#### Agent 已释放状态

```
┌─────────────────────────────────────────────────────────┐
│  ⚪ Dave-2 (iOS 开发)                                     │
│  状态: 已释放                                              │
│  技能: iOS、Swift                                        │
│  当前任务: 无                                            │
│  最后活动: 10 分钟前                                      │
│                                                          │
│  [启动]                                                 │
└─────────────────────────────────────────────────────────┘
```

#### Session 损坏提示

```
┌─────────────────────────────────────────────────────────┐
│  ⚠️  Session 损坏                                         │
│                                                          │
│  Dave-1 的对话历史文件损坏，无法恢复                        │
│                                                          │
│  [启动新 Session] [尝试修复]                              │
└─────────────────────────────────────────────────────────┘
```

#### Agent 自动启动提示

```
┌─────────────────────────────────────────────────────────┐
│  🔄  Agent 自动启动                                        │
│                                                          │
│  Dave-2 已释放，正在自动启动...                            │
│  正在恢复对话历史... (50%)                                │
│                                                          │
│  [取消]                                                 │
└─────────────────────────────────────────────────────────┘
```

### 8.3 关键状态说明

#### 空态

- **员工档案**：显示"暂无员工"提示，引导用户配置 Agent

#### 加载态

- **员工档案**：Agent 启动中，显示"正在启动..."动画

#### 失败态

- **员工档案**：Agent 启动失败时，显示"无法启动 Agent"错误
- **Session 损坏**：显示"Session 损坏"错误，提供修复方案

#### 成功态

- **员工档案**：显示所有 Agent 的信息，状态实时更新
- **闲置提示**：显示"Agent 即将自动释放"提示

---

## 9. 依赖与备注

### 9.1 技术依赖

**Rust 技术栈**：
- Tauri v2（桌面框架）
- std::process（进程管理）
- serde + serde_json（JSONL 格式）
- tokio（异步任务调度）

**外部依赖**：
- Agent 进程：Claude Code（`claude -p --input-format=stream-json --output-format=stream-json --agent <agent>`）
- Session 格式：JSONL（JSON Lines）

---

### 9.2 STORY 依赖

**前置 STORY**：
- **STORY-001（全局状态可视化）**：依赖员工档案和 Agent 状态
- **STORY-002（任务调度闭环）**：依赖 Agent 启动和消息发送
- **STORY-003（实时进度监控）**：依赖进程状态监听

**后续 STORY**：
- 无（本 STORY 是最后一个核心 STORY）

---

### 9.3 数据依赖

**Session 格式**（JSONL）：
```json
{"timestamp": "2026-01-20T14:30:00Z", "role": "user", "content": "领 TASK-03，开始实现用户登录 API"}
{"timestamp": "2026-01-20T14:30:05Z", "role": "assistant", "content": "好的，开始实现 TASK-03。首先检查 PRD 中的需求..."}
```

**配置文件格式**（`config.yaml`）：
```yaml
agent_management:
  idle_timeout: 5  # 闲置时间阈值（分钟）
  auto_release: true  # 是否自动释放
  session_path: "/sessions"  # Session 文件目录
```

---

### 9.4 风险与未决项

**[OPEN-1] 闲置时间阈值**
- **问题**：默认 5 分钟是否合适？是否需要可配置？
- **建议**：默认 5 分钟，提供配置文件让用户自定义
- **责任方**：prd + tech

**[OPEN-2] Session 恢复成功率**
- **问题**：如何保证 Session 恢复成功率 > 95%？
- **建议**：使用冗余存储（如备份文件），定期校验 JSONL 文件完整性
- **责任方**：tech

**[OPEN-3] 闲置检测准确性**
- **问题**：如何准确检测 Agent 是否闲置？
- **建议**：综合判断（无输出 + 无交互 + 无任务），避免误杀
- **责任方**：tech

---

### 9.5 非目标（Out of Scope）

本 STORY **不包含**：
- ❌ 自动任务调度（L2 级别）：属于 Phase 2
- ❌ 多用户支持：属于 Phase 2
- ❌ 分布式 Session 存储（如数据库）：属于 Phase 2
- ❌ Session 加密：属于 Phase 2

---

## 10. 验收总结

### 10.1 核心验收点

| 验收点 | 验收方式 | 目标值 |
|-------|---------|-------|
| **闲置释放时间** | 计时测试 | 5 分钟（可配置） |
| **误杀率** | 统计测试 | < 1% |
| **Session 恢复成功率** | 统计测试 | > 95% |
| **Agent 启动时间** | 计时测试 | < 3 秒 |
| **Agent 终止时间** | 计时测试 | < 2 秒 |

### 10.2 质量门槛

- ✅ 代码清晰：遵守 YAGNI 和 KISS 原则
- ✅ 测试覆盖率：核心业务逻辑 50%
- ✅ 能跑、不崩、能验证核心价值

### 10.3 用户体验北极星验证

**体验北极星**：从"手动管理"到"自动回收"

**验证方式**：
- ✅ 用户不需要手动启动/kill Agent
- ✅ 闲置资源自动释放，不浪费内存
- ✅ Session 自动恢复，对话历史不丢失
- ✅ 误杀率 < 1%（不会误杀正在使用的 Agent）

---

## 11. 变更记录

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v0.1 | 2026-01-20 | prd agent | 初版 STORY-004，基于 PRD 场景 4 |

---

## 附录：参考文档

- **PRD**：`/docs/E-001-Agent-Console/prd/PRD-E-001-v0.md`
- **Tech Baseline**：`/docs/_project/tech-baseline.md`
- **STORY-001**：`/docs/E-001-Agent-Console/story/STORY-001-全局状态可视化.md`
- **STORY-002**：`/docs/E-001-Agent-Console/story/STORY-002-任务调度闭环.md`
- **STORY-003**：`/docs/E-001-Agent-Console/story/STORY-003-实时进度监控.md`
- **UI 原型**：`/docs/E-001-Agent-Console/prototypes/index.html`
