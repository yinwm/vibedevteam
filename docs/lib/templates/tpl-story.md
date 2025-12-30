# {{STORY_ID}} {{STORY_TITLE}}

> 文档路径：`/docs/{{EPIC_DIR}}/story/{{STORY_ID}}-{{STORY_TITLE}}.md`

* EPIC_ID：{{EPIC_ID}}
* EPIC_DIR：`{{EPIC_DIR}}`
* STORY_ID：{{STORY_ID}}
* Story 状态：草稿 / 评审中 / 可开发 / 已完成
* 优先级：Must / Should / Could
* 创建人：{{OWNER}}
* 创建日期：{{YYYY-MM-DD}}

---

## 0. 关联信息（References）

* 关联 Epic：`{{EPIC_ID}} {{EPIC_NAME}}`
* 关联 PRD：`../prd/PRD-{{EPIC_ID}}-v1.md`（填写章节引用）
* 关联 Tech Design：`../tech/` 下 `TECH-*.md`
* 关联 Task：`../task/` 下 `TASK-*.md`

---

## 1. 背景与目标（Background & Goal）

### 1.1 背景

* 现状 / 痛点：
* 上游上下文（biz-overview / PRD 摘要）：
* 为什么必须做：

### 1.2 Story 目标

* 用户视角目标：
* 业务视角目标：

---

## 2. 用户 & 场景（Users & Scenarios）

### 2.1 用户角色

* 主要用户角色：
* 次要参与角色（如有）：

### 2.2 主要使用场景（高层）

* 场景 S1：
* 场景 S2：

---

## 3. 用户故事描述（User Story）

* As a：
* I want：
* So that：

补充（可选）：

* 业务规则 / 约束：

---

## 4. 验收标准（Acceptance Criteria）

* AC1：
* AC2：

## 4.1 状态机与关键状态（必填）

> 至少覆盖：空态 / 加载态 / 成功态 / 失败态（可重试/不可重试）/ 权限不足（如适用）

* 状态列表：
* 状态切换条件：
* 对应 UI 提示/按钮：

## 4.2 契约草案（Contract Draft）（必填）

> 目的：让 TDD/实现有共同的边界；字段名/错误码/状态字段必须在这里先对齐。

* API（最小集）：
  * Endpoint：
  * Request：
  * Response（含状态字段）：
  * Error codes（至少 5 个）：
* 字段表（字段名/含义/必填/默认值/边界）：

## 4.3 UI 证据（Evidence）（推荐）

* 原型：`/docs/{{EPIC_DIR}}/prototypes/index.html`（或截图/录屏链接）

---

## 5. 验收用例（Acceptance Scenarios）

### 场景 S1：{{TITLE}}

* Given：
* When：
* Then：

---

## 6. 数据 & 埋点（Data & Analytics）（可选）

* 本期是否需要埋点/报表：

---

## 7. 非功能要求（NFR for this Story）（可选）

* 性能：
* 可靠性：
* 可维护性：

---

## 8. 风险 & OPEN

* 风险：
* [OPEN]：

---

## 9. 变更记录（Changelog）

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{YYYY-MM-DD}} | {{OWNER}} | 初版 |
