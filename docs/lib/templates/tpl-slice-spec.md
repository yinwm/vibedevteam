# SLICE 竖切规格 - {{SLICE_ID}}（{{EPIC_ID}}）

> 文档路径：`/docs/{{EPIC_DIR}}/slice/{{SLICE_ID}}.md`
>
> 目的：把“一个可用闭环”定义清楚，作为 TECH/TASK 的硬输入，避免凭想象拆任务。

* EPIC_ID：{{EPIC_ID}}
* EPIC_DIR：`{{EPIC_DIR}}`
* SLICE_ID：{{SLICE_ID}}（示例：`SLICE-{{EPIC_ID}}-001`）
* 状态：草稿 / 评审中 / 可开发 / 已完成
* Owner：{{OWNER}}
* 创建日期：{{YYYY-MM-DD}}

---

## 0. References

* biz-overview：`/docs/_project/biz-overview.md`
* PRD：`../prd/PRD-{{EPIC_ID}}-v1.md`
* Story：`../story/STORY-*.md`
* TECH：`../tech/TECH-{{EPIC_ID}}-v1.md`
* UI 证据（推荐）：
  * 原型：`/docs/{{EPIC_DIR}}/prototypes/index.html`
  * 或截图/录屏链接：

---

## 1. 这刀交付什么（Definition）

* 交付闭环一句话：
* 入口/页面：
* 主要用户：
* 完成定义（用户看到什么结果算“完成”）：
* 本刀不做什么（明确 Out of scope）：

---

## 2. 主路径（Happy Path）

按“步骤 + 期望”写清楚：

1. …
2. …
3. …

---

## 3. 状态机（States）

至少覆盖：
* 空态
* 加载态
* 成功态
* 失败态（可重试/不可重试）
* 权限不足（如适用）

---

## 4. 契约（Contract）

### 4.1 API（最小集）

* Endpoint：
* Request：
* Response（含状态字段）：
* Error codes（最少 5 个）：

### 4.2 数据与字段含义

* 字段表（字段名/含义/必填/默认值/边界）：

---

## 5. 可测试 AC（Acceptance Criteria）

每条 AC 都要能落到测试或手工验收步骤：

* AC1：
* AC2：

---

## 6. TDD 计划（Test First）

* 要先写哪些测试：
* 关键断言是什么：
* 运行命令（后端/前端）：

---

## 7. 风险与 OPEN

* 风险：
* [OPEN]：

---

## 8. Changelog

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v1.0 | {{YYYY-MM-DD}} | {{OWNER}} | 初版 |
