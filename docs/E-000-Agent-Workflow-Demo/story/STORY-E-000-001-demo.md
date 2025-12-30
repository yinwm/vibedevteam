# STORY-E-000-001 Demo: 浏览示例 Epic

> 文档路径：`/docs/E-000-Agent-Workflow-Demo/story/STORY-E-000-001-demo.md`

* EPIC_ID：E-000
* EPIC_DIR：`E-000-Agent-Workflow-Demo`
* STORY_ID：STORY-E-000-001
* Story 状态：可开发
* 优先级：Must
* 创建人：@demo
* 创建日期：2024-01-01

---

## 0. 关联信息（References）

* 关联 Epic：`E-000 Agent Workflow Demo`
* 关联 PRD：`../prd/PRD-E-000-v0.md`（6.1 UI 证据，9 AC）
* 关联 Tech Design：`../tech/TECH-E-000-v0.md`
* 关联 Task：`../task/TASK-E-000-DOC-001.md`

---

## 1. 背景与目标（Background & Goal）

### 1.1 背景

* 现状 / 痛点：新成员不知道“Gate B/C 合格的文档+原型”长什么样。
* 上游上下文：PRD 需要一份可运行原型与 Traceability 样例。
* 为什么必须做：提供一刀竖切闭环示例，降低首次落地成本。

### 1.2 Story 目标

* 用户视角目标：打开示例页面即可理解主路径与关键状态，并找到对应文档。
* 业务视角目标：确保后续 Epic 可以直接复制路径与模板。

---

## 2. 用户 & 场景（Users & Scenarios）

### 2.1 用户角色

* 主要用户角色：新入组的 PRD/Tech/Dev。
* 次要参与角色：外部参考者。

### 2.2 主要使用场景（高层）

* 场景 S1：打开原型，切换状态，确认 UI 证据。
* 场景 S2：点击文档链接，跳转到 PRD/Story/Slice/Tech/Task/Proj 示例。

---

## 3. 用户故事描述（User Story）

* As a：新加入的交付成员
* I want：在一个页面看到“主路径 + 关键状态 + 文档入口”
* So that：可以按 Gate B/C 要求复制出自己的 Epic

补充：

* 业务规则 / 约束：无需登录、无需真实数据；所有链接必须在仓库内可点。

---

## 4. 验收标准（Acceptance Criteria）

* AC1：页面默认展示空态，并提示“点击开始 Demo”。
* AC2：点击“展示成功态”后，能看到主路径步骤与文档链接列表。
* AC3：点击“模拟失败”出现错误提示，支持重试返回成功态。

## 4.1 状态机与关键状态（必填）

* 状态列表：空态 → 加载态 → 成功态 / 失败态。
* 状态切换条件：点击主按钮进入加载态；模拟成功后进入成功态；模拟失败后进入失败态；失败态点击重试回到加载态。
* 对应 UI 提示/按钮：主按钮文本根据状态变化（开始 / Loading / 重试）。

## 4.2 契约草案（Contract Draft）（必填）

* API（最小集）：示例为纯静态，无后端 API。
* 事件：页面按钮事件在前端内切换状态。
* 字段表：无持久化字段；状态枚举 `["empty","loading","success","error"]`。

## 4.3 UI 证据（Evidence）（推荐）

* 原型：`/docs/E-000-Agent-Workflow-Demo/prototypes/index.html`

---

## 5. 验收用例（Acceptance Scenarios）

### 场景 S1：查看成功态

* Given：打开原型，默认空态
* When：点击“展示成功态”
* Then：看到主路径卡片、文档链接列表、状态徽标为“成功”

### 场景 S2：模拟失败并重试

* Given：在成功态
* When：点击“模拟失败”
* Then：看到错误提示和重试按钮；点击重试回到加载→成功。

---

## 6. 数据 & 埋点（Data & Analytics）

* 本期不接入埋点。

---

## 7. 非功能要求（NFR for this Story）

* 性能：本地打开 < 1s。
* 可靠性：无网络依赖。

---

## 8. 风险 & OPEN

* 风险：示例内容未来与真实流程脱节。
* [OPEN]：是否需要添加动效录屏供离线查看。
