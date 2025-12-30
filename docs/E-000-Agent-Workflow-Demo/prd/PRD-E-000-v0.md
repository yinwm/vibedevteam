# Agent Workflow Demo PRD - v0（E-000）

> 文档路径：`/docs/E-000-Agent-Workflow-Demo/prd/PRD-E-000-v0.md`
>
> * EPIC_ID：E-000
> * EPIC_DIR：`E-000-Agent-Workflow-Demo`
> * 文档状态：草稿
> * 版本：v0
> * 创建人：@demo
> * 创建日期：2024-01-01
> * 更新日期：2024-01-01

---

## 0. 关联信息（References）

* biz-overview：`/docs/_project/biz-overview.md`（示例，未创建）
* Story：`../story/STORY-E-000-001-demo.md`
* Tech Design：`../tech/TECH-E-000-v0.md`
* Proj Plan：`../proj/PROJ-E-000-v1.md`

---

## 1. 背景与动机（Background）

* 现状：新人或外部用户不知道如何按 Gate B/C 方式落地一个 Epic。
* 问题：缺少可运行的 UI 证据与填好的样例文档。
* 影响：启动成本高，容易绕过 UI 证据和 Slice 约束。

## 2. 目标与成功标准（Goals & Success Metrics）

* 本 Epic 业务目标：让读者可以在 5 分钟内看懂并复制一套示例 Epic。
* 本 Epic 交付目标：提供一份填好的 PRD/Story/Slice/Tech/Task 以及可运行原型。

## 3. 范围与非目标（Scope & Non-goals）

* In Scope：示例文档、可运行 HTML 原型、Traceability 表。
* Out of Scope：真实后端实现、权限/合规细节、CI/CD 集成。

## 4. 用户与场景（Users & Scenarios）

* 用户角色：新加入的产品/技术同学、外部参考者。
* 关键场景：5 分钟浏览一圈 Demo → 复制模板 → 按 Gate 开始自己的 Epic。

## 5. 功能需求（Functional Requirements）

### 5.1 功能 F1：浏览示例 Epic

* 描述：能在 UI 原型中看到主路径、关键状态与文档链接。
* 业务规则：展示 Gate B（UI 证据）与 Gate C（Story+Slice）通过的最小闭环。
* 权限：公开。
* 边界条件：无后端依赖。

### 5.2 功能 F2：复制模板入口

* 描述：从页面跳转到各文档的原始模板与示例。
* 业务规则：提供相对路径，保证在本仓库内可点击。
* 权限：公开。
* 边界条件：需在离线环境也可阅读。

## 6. 交互与信息结构（UX/UI）

* 页面/入口：单页原型展示“状态切换 + 文档链接”。
* 信息结构：顶部状态切换，中间展示主路径卡片，底部列出文档链接。
* ASCII 草图（可选）：
  ```
  [状态: 空/加载/成功/失败] [主按钮: 打开 Demo]
  ------------------------------------------------
  | 主路径步骤 | UI 证据卡片 | 文档链接列表 |
  ------------------------------------------------
  ```

## 6.1 UI 证据（Evidence）

* 原型：`/docs/E-000-Agent-Workflow-Demo/prototypes/index.html`
* 截图/录屏（可选）：[OPEN]
* 关键状态说明：
  * 空态：无输入，页面提示“点击开始”。
  * 加载态：按钮禁用并显示“Building sample...”（模拟）。
  * 失败态：显示错误提示与重试按钮。
  * 成功态：展示主路径卡片与文档链接。

## 7. 数据与报表（Data）

* 字段/口径：不涉及埋点（示例）。
* 统计项：无。

## 8. 非功能需求（NFR）

* 性能：纯静态，首屏 < 1s。
* 可靠性：无网络依赖，离线可读。
* 可观测性：示例不接入日志/指标。
* 安全：无用户数据。

## 9. 验收标准（Acceptance Criteria）

* AC1：原型页面能演示空/成功/失败三种状态切换。
* AC2：PRD/Story/Slice/Tech/Task/Proj 示例文件可点击并存在。
* AC3：Story 与 Task 显示了 Traceability（Story → Slice → Task）。

## 10. 灰度与发布（Rollout）

* 开关/配置：无。
* 回滚方案：删除示例目录或恢复模板。

## 11. 风险与 OPEN

* 风险：示例过时与真实流程偏离。
* [OPEN]：是否需要配套 biz-overview 示例。

## 12. 变更记录（Changelog）

| 版本 | 日期 | 修改人 | 修改内容 |
|------|------|--------|----------|
| v0 | 2024-01-01 | @demo | 初版 |
