---
name: workflow-overview
description: 工作流总览：UI 证据 + 用户旅程 + TDD 驱动交付
version: 0.4.0
author: 大铭 <yinwm@outlook.com>
updated: 2026-01-20
---

# 工作流总览：以 UI 证据 + 用户旅程 + TDD 驱动交付

> 目的：修复"文档写很全 → 任务拆很细 → 后端做完才看见页面 → 发现不是想要的"的结构性问题。
>
> 核心变化：
> 1) **UI 证据前置**（可运行最小 HTML 原型/截图/录屏），尽早对齐页面形态与主路径；
> 2) **用户旅程** 在 PRD v0 中合并，帮助理解用户体验；**厚 STORY** 作为交付规格，避免 TECH 直接拆 TASK 失真；
> 3) **TDD/Gate/Rebaseline** 机制写进 `PROJ`，把"能不能开工/能不能 DONE"制度化。

**相关文档**：
- 模板映射表：`/docs/lib/template-mapping.md`（各 Agent 使用的模板及对应关系）
- 各 Agent 配置：`.claude/agents/*.md`（详见各 Agent 的 `## 0.1 对应模板说明`）

---

## 1. 角色分工

* `biz-owner`：定方向与边界（In/Out、MVP、止损信号、成功指标）。
* `prd`：产出 PRD v0/v1、厚 STORY；在 PRD 中固化 UI 证据与可测试 AC，包含用户旅程帮助理解用户体验。**按需调用 ux 能力**产出可运行原型（`/docs/{{EPIC_DIR}}/prototypes/...`）。
* `tech`：基于厚 STORY 输出最小可落地方案与 trade-off；不在缺少输入时输出可执行 TASK 清单。
* `proj`：写 Gate 与 Rebaseline；决定本期纳入清单与里程碑；用 Gate 控制"能不能进入实现/能不能 DONE"。
* `dev`：唯一允许改代码/配置；以 TDD/契约优先交付并回写证据。

---

## 2. 文档地图（新增用户旅程 / Prototypes）

```text
/docs
  /lib/                         # 可复用基础设施（可迁移到新项目）
    workflow-overview.md         # 工作流规范（本文档）
    template-mapping.md          # 模板映射表（Agent ↔ Template）
    templates/                   # 文档模板
      tpl-biz-overview.md
      tpl-prd.md
      tpl-story.md
      tpl-tech-epic.md
      tpl-task.md
      tpl-proj-epic.md
      tpl-proj-roadmap.md
      tpl-prototype-index.html
      tpl-code-review-tech.md
  /_project/                    # 项目特定配置（当前仓库专属）
    biz-overview.md             # 业务概览（biz-owner）
    tech-baseline.md            # 技术基线（tech）
    arch-overview.md            # 架构总览（tech）
    adr/                        # 架构决策记录
  /{{EPIC_DIR}}                 # Epic 文档
    prd/
      PRD-{{EPIC_ID}}-v0.md        # 包含用户旅程
      PRD-{{EPIC_ID}}-v1.md
    story/
      STORY-{{EPIC_ID}}-*.md
    prototypes/
      index.html                    # UI 证据
    tech/
      TECH-{{EPIC_ID}}-v0.md
      TECH-{{EPIC_ID}}-v1.md
    task/
      TASK-{{EPIC_ID}}-*.md
    proj/
      PROJ-{{EPIC_ID}}-v1.md
```

**模板映射说明**：
- 各 Agent 使用哪些模板，详见 `/docs/lib/template-mapping.md`
- 每个 Agent 配置文件（`.claude/agents/*.md`）的 `## 0.1 对应模板说明` 章节也有详细说明

---

## 3. 流程（带 Gate）

### Phase A：方向对齐（biz-owner）

**产物**：
* `/docs/_project/biz-overview.md`（项目级，允许大量 `[OPEN]`）
* `/docs/{{EPIC_DIR}}/biz/BIZ-E-{{EPIC_ID}}-v0.md`（Epic 级，可选）
  * 复杂 Epic 需要先写 Epic 级 biz-overview
  * 简单 Epic 可以跳过，直接从 PRD v0 开始
* 明确本 Epic 的 In/Out、MVP、止损信号

**Gate A（项目级，进入任何 Epic 前必须满足）**：
* 目标/范围/非目标能被复述，且存在止损信号

**Gate A（Epic 级，可选）**：
* 对于复杂 Epic，需要先通过 Epic 级 Gate A（在 `BIZ-E-{{EPIC_ID}}-v0.md` 中定义）
* 简单 Epic 可以跳过 Epic 级 Gate A

### Phase B：PRD v0 + 用户旅程 + UI 证据（prd，按需调用 ux 两步走）

**目标**：用最小成本把"页面形态 + 主路径 + 用户体验"定住。

**两步走流程**：
1. **ASCII 线框图**（必填）：prd 按需调用 ux，用 ASCII 线框图快速确认布局与主路径
2. **HTML 原型**（可选）：方向确认后，使用 `frontend-design` skill 生成高保真原型

**产物**：
* `PRD-{{EPIC_ID}}-v0.md`（包含：用户旅程 + 主路径 + 关键状态 + 分叉点 + `[OPEN]`）
  * **用户旅程**：简化版，帮助理解用户在特定场景下的体验流程
  * **主路径**：产品的主要交互路径
  * **UI 证据**：ASCII 线框图 +（可选）HTML 原型链接

**Gate B**（进入实现前必须满足）：
* 你看完 UI 证据（至少 ASCII 线框图）后能说"方向对了/差不多就是这样"。

### Phase C：厚 STORY（prd 主导，tech 参与）

厚 STORY（最低标准）必须包含：
* 主路径步骤（可被手工验收复现）
* 状态机与关键状态（空/加载/失败/成功/部分成功/权限不足）
* 可测试 AC（步骤 + 期望结果）
* 边界/错误态与用户提示
* 接口契约草案（请求/响应/错误码/状态字段）
* UI 证据引用

Gate C（允许拆 TASK）：
* 至少 1 个厚 STORY。

### Phase D：TECH v0/v1（tech）

TECH v0（可选）：只写关键分叉决策与最小架构。

TECH v1：基于厚 STORY 补齐数据/API/迁移/回滚/可观测方案。

**代码验证优先（tech 必须遵守）**：
- 在输出任何设计前，必须先读代码验证：
  - 读取数据模型定义（验证字段真实存在）
  - 搜索现有实现（理解数据关联方式）
  - 追踪关键变量生命周期（从创建到使用的完整路径）
- 禁止基于"应该有这个字段"的假设设计
- 所有未验证内容必须标记 `[ASSUMPTION]`

**数据流追踪法**：
- 选取关键变量（配置、上下文等）
- 追踪变量在代码中的完整生命周期：创建→传递→使用→销毁
- 检查点：
  - [ ] 变量是否在所有需要的地方都可访问？
  - [ ] 是否有作用域断裂（局部变量无法跨阶段）？
  - [ ] 是否有重复声明导致的数据丢失？

硬护栏：
* 未满足 Gate C 时，tech 不输出"可执行 TASK 清单"；只输出风险与需要补齐的输入。

### Phase E：PROJ + TASK（proj + tech 建议）

proj 做两件事：
1) 决定本期只纳入“闭环 P0”还是加上部分增强（P1/P2）。
2) 把 Gate 写进 `PROJ`，并把“完成定义”落到每个 TASK。

**新增硬规则：Story→Task 可追溯性（Traceability）**

* 本期纳入的每个 `TASK` 必须能追溯到：
  * `STORY_ID`（来源需求）
* 允许例外：技术债/纯重构/基础设施类任务可以标注 `NO_STORY`，但必须写清：
  * 为什么本期要做
  * 如何验收（可测试/真流程）

proj 必须在 `PROJ-*.md` 维护一张对齐表（本期纳入清单的"硬证据入口"）：

* `STORY_ID` → `TASK_ID 列表` → 本期纳入/不纳入 → 验收责任人

同时，proj 必须维护一张执行进度表（Progress Table）：
* 以 `TASK` 为粒度，跟踪 `TODO/DOING/BLOCKED/DONE`、Owner、预计与阻塞点；
* 证据入口必须链接到对应 `TASK-*.md`（测试/真流程/回滚/观测点都在 Task 文档回写）。

### 依赖类型（tech 拆解，proj 设置）

tech 在拆解任务时区分两种依赖类型：

| 类型 | 定义 | 示例 | beads 设置 | 并行策略 |
|------|------|------|-----------|---------|
| **硬依赖** | 代码直接 import 了其他任务的模块 | Handler 调用 Service | `bd dep add` | 必须等待 |
| **接口依赖** | 只需调用接口，不依赖具体实现 | 前端调用后端 API | 不设置依赖 | 契约先行，可并行 |

**接口依赖的前置条件**：
- 必须先定义接口契约（类型签名、参数、返回值）
- 被依赖任务必须在 TECH 中明确接口契约
- 允许使用桩实现（空实现但类型对齐），禁止使用 mock 数据

### Phase F：实现（dev，TDD 优先）

#### 🟢 特殊情况：快速修复模式（Quick Fix Mode）

**适用场景**：
- 紧急 BUG 修复
- 小改动（几行代码）
- 不需要走 Epic/Story/Task 流程的临时修改

**工作流程**：
1. 修改代码
2. 等待用户确认
   - 用户说"提交了" → git commit
   - 用户说"推了" → git push
   - 用户说"算了" → git restore
3. 不要自动 commit/push

**与 Epic 开发的区别**：
| | 快速修复模式 | Epic 开发模式 |
|---|---|---|
| 触发 | 用户说"先改了再说" | 有 TASK 文档 |
| 流程 | 修改 → 等确认 → commit → push | review → commit → push → bd close |
| beads | 不涉及 | 必须同步 |
| 文档 | 可选 | 必须更新 |

⚠️ **注意**：快速修复模式只适用于真正的小改动。任何需要设计、测试、上线的功能都应该走 Epic 开发模式。

---

#### 标准提交流程（Epic 开发模式）

**提交流程（硬性约束）**：
1. 完成代码 + 文档更新
2. **标记 TASK 为 DOING，请求 tech review（此时不要 commit）**
3. **tech review 通过后，才执行 Git Commit**
4. commit 完成后，标记 TASK 为 DONE
5. review 不通过时，按 tech 意见修改，重新从步骤 2 开始

**Git Commit 要求（review 通过后执行）**：
* **代码 + 文档必须作为原子动作提交**
* 每个 Git Commit 必须：
  * 引用对应的 TASK ID（如 `refs TASK-LOG-001`）
  * 包含代码变更和文档变更
  * 在 commit message 中说明文档更新状态
* 提交前自检：
  ```bash
  - 我修改了哪个 TASK 的代码？
  - 对应的 TASK 文档更新了吗？
  - 验收标准打勾了吗？
  - tech review 通过了吗？
  ```

**TDD 规则**：
* P0/P1 默认先写测试（或至少先写可运行的测试计划与断言清单）再写实现；
* **测试覆盖率目标：70%**（运行 `go test -cover` 或 `pnpm test --coverage`）；
* UI 原型允许使用 mock 数据（为了快速对齐交互与状态机）；
* 交付验证（测试）必须包含"真数据真流程"的验证：
  * 最少要求：对本期纳入的 P0/P1 闭环，跑一遍可复现的端到端流程（Staging/测试租户/沙箱环境皆可）；
  * 单测/契约测试可以使用 stub/mock（用于覆盖边界与错误分支），但**不得**作为唯一验证手段；
* 所有证据回写到 `TASK-*.md`。

#### 🔧 文档与代码同步闭环（强制）

**当 Dev 发现文档假设错误时的处理流程**：

开发时如果发现文档（PRD/TECH/Story）中的假设与实际代码不符，**必须**按以下流程处理：

```
发现文档假设错误
    ↓
1. 先修改代码（确保能跑通）
    ↓
2. 在 TASK 文档中记录问题
   - 记录：文档假设 vs 实际发现
   - 标注：[ISSUE] 文档假设错误
    ↓
3. 判断影响范围
   - 只影响当前 TASK？→ 只更新 TASK 文档
   - 影响 PRD/TECH/Story？→ 必须同步更新
    ↓
4. 更新上游文档（如需要）
   - 更新 PRD/TECH/Story 中的错误假设
   - 标注变更原因和实际发现
    ↓
5. 在 commit message 中说明
   - "fix: 修正文档假设，实际代码采用 X 方案"
   - "docs: 更新 TECH-v1.md 数据模型描述"
    ↓
6. 请求 tech review
   - 重点说明文档修正点
```

**强制检查清单**（Dev 在提交前必须确认）：

| 检查项 | 说明 | 示例 |
|--------|------|------|
| [ ] **代码与文档一致** | 代码实现是否与 TASK/TECH 文档描述一致？ | 文档说调用 API，代码硬编码了 → ❌ |
| [ ] **假设已标记** | 文档中所有假设是否已标记 `[ASSUMPTION]` 或 `[VERIFIED]`？ | 未验证的 SQL 必须标记 `[ASSUMPTION]` |
| [ ] **修正已回写** | 发现文档错误时，是否已更新上游文档？ | 数据模型错误必须同步更新 TECH |
| [ ] **变更已记录** | commit message 是否引用了 TASK 并说明文档变更？ | `fix: 修正数据模型假设，docs: 更新 TECH-v1` |

**文档修正模板**（在 TASK 文档中记录）：

```markdown
## 实现中发现的问题

### [ISSUE] 文档假设错误

**文档假设**：
- TECH 文档假设 `contact_roles` 表有 `room_id` 字段

**实际发现**：
- 读取代码后发现表中没有 `room_id` 字段
- 实际关联路径：通过 `room_members` 表关联

**采用的方案**：
- 复用 `ContactRoleService.GetRoleMembersInRoom()` 方法
- 引用：internal/services/contact_role_service.go:123

**同步更新的文档**：
- [ ] TECH-E-001-v1.md：更新数据模型描述
- [ ] TASK-E-001-BE-002.md：修正实现说明
```

### Phase G：Rebaseline（任何角色可触发，proj 负责落地）

触发条件：
* 你看到 UI/联调结果后发现“不是想要的”
* 关键分叉决策改变（例如：建表策略、授权策略、页面形态）

必须动作：
* PRD/TECH/PROJ 升版本并记录变更点
* 更新本期纳入清单（取消/延期/新增 TASK）
* 已完成能力明确归类：
  * 继续复用
  * 变为后续增强
  * 明确不再使用（但不强制删除）

---

## 5. TDD 作为 Gate（建议写入 PROJ）

* P0/P1 Task 进入 DONE 前必须有：
  * 对应 AC 的测试（自动化优先）与可复现的运行命令 + 结果
  * 无法自动化时的原因说明 + 手工验收脚本与证据
  * 回滚方案 + 上线观测点

---

## 6. 模拟运行 Dry Run（proj 负责）

### 核心目标

从 STORY 出发，模拟完整开发链路，发现隐藏问题。

```
PRD → STORY → TECH → TASK → 验收 → 上线
   ↓         ↓       ↓
  需求      接口    实现
```

### 触发时机

* **STORY 验收前**：验证 STORY 是否能追溯到 PRD
* **TASK 拆解后**：验证 TASK 是否覆盖 STORY AC
* **Phase 切换前**：验证完整链路是否通顺
* **Rebaseline 前**：验证变更影响范围

### 执行方式

```bash
使用 /vibedevteam:dryrun skill 执行模拟运行
```

### 链路检查（6 步）

| 步骤 | 检查 | 发现问题 |
|------|------|----------|
| 1 | PRD → STORY | STORY 是否有需求来源 |
| 2 | STORY → TECH | 接口是否定义完整 |
| 3 | STORY → TASK | AC 是否可拆成 TASK |
| 4 | TASK 依赖 | 是否有循环依赖 |
| 5 | TASK 验收 → STORY AC | 验收是否覆盖 AC |
| 6 | 真流程验证 | 是否有前置依赖 |

### 问题级别

| 级别 | 定义 | 处理 |
|------|------|------|
| **P0** | 阻塞，无法继续开发 | 必须修复 |
| **P1** | 严重，后续大量返工 | 建议修复 |
| **P2** | 警告，可后续优化 | 可选修复 |

### 报告输出

模拟运行输出结构化报告，包含：
- 6 步链路检查结果
- P0/P1/P2 问题列表
- 改进建议
- 整体结论（通过 / 需修复后通过）

---

## 7. 快速使用指南（最小集）

* 新建 Epic 时先创建：
  * `PRD-{{EPIC_ID}}-v0.md`（包含用户旅程）
  * `docs/{{EPIC_DIR}}/prototypes/index.html`（可选但推荐）
  * `STORY-{{EPIC_ID}}-001.md`
* 先做一个垂直切片验证核心价值，再迭代增强。
