# 工作流总览：以 UI 证据 + Slice + TDD 驱动交付

> 目的：修复"文档写很全 → 任务拆很细 → 后端做完才看见页面 → 发现不是想要的"的结构性问题。
>
> 核心变化：
> 1) **UI 证据前置**（可运行最小 HTML 原型/截图/录屏），尽早对齐页面形态与主路径；
> 2) **Slice Spec** 作为 STORY 与 TASK 之间的硬输入，避免 TECH 直接拆 TASK 失真；
> 3) **TDD/Gate/Rebaseline** 机制写进 `PROJ`，把"能不能开工/能不能 DONE"制度化。

**相关文档**：
- 模板映射表：`/docs/lib/template-mapping.md`（各 Agent 使用的模板及对应关系）
- 各 Agent 配置：`.claude/agents/*.md`（详见各 Agent 的 `## 0.1 对应模板说明`）

---

## 1. 角色分工（新增 ux，可选）

* `biz-owner`：定方向与边界（In/Out、MVP、止损信号、成功指标）。
* `prd`：产出 PRD v0/v1、厚 STORY、（推荐）SLICE；在 PRD 中固化 UI 证据与可测试 AC。
* `ux`（可选但推荐）：做可运行最小原型（`/docs/{{EPIC_DIR}}/prototypes/...`），把“页面长什么样”变成证据回填 PRD。
* `tech`：基于厚 STORY/SLICE 输出最小可落地方案与 trade-off；不在缺少输入时输出可执行 TASK 清单。
* `proj`：写 Gate 与 Rebaseline；决定本期纳入清单与里程碑；用 Gate 控制“能不能进入实现/能不能 DONE”。
* `dev`：唯一允许改代码/配置；以 TDD/契约优先交付并回写证据。

---

## 2. 文档地图（新增 Slice / Prototypes）

```text
/docs
  /lib/                         # 可复用基础设施（可迁移到新项目）
    workflow-overview.md         # 工作流规范（本文档）
    template-mapping.md          # 模板映射表（Agent ↔ Template）
    templates/                   # 文档模板
      tpl-biz-overview.md
      tpl-prd.md
      tpl-story.md
      tpl-slice-spec.md
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
      PRD-{{EPIC_ID}}-v0.md
      PRD-{{EPIC_ID}}-v1.md
    story/
      STORY-{{EPIC_ID}}-*.md
    prototypes/
      index.html
    slice/
      SLICE-{{EPIC_ID}}-001.md
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

产物：
* `/docs/_project/biz-overview.md`（允许大量 `[OPEN]`）
* 明确本 Epic 的 In/Out、MVP、止损信号

Gate A（进入 PRD v0 前必须满足）：
* 目标/范围/非目标能被复述，且存在止损信号。

### Phase B：PRD v0（prd + ux）

目标：用最小成本把“页面形态 + 主路径”定住。

产物：
* `PRD-{{EPIC_ID}}-v0.md`（短：主路径 + 关键状态 + 分叉点 + `[OPEN]`）
* UI 证据入口：
  * 推荐：`/docs/{{EPIC_DIR}}/prototypes/index.html`
  * 或截图/录屏链接

Gate B（进入实现前必须满足）：
* 你看完 UI 证据后能说“方向对了/差不多就是这样”。

### Phase C：厚 STORY + Slice Spec（prd 主导，tech 参与）

厚 STORY（最低标准）必须包含：
* 主路径步骤（可被手工验收复现）
* 状态机与关键状态（空/加载/失败/成功/部分成功/权限不足）
* 可测试 AC（步骤 + 期望结果）
* 边界/错误态与用户提示
* 接口契约草案（请求/响应/错误码/状态字段）
* UI 证据引用

Slice Spec（SLICE-001）建议只做“一刀竖切闭环”：
* 交付闭环定义（能用/能验收）
* 最小接口契约
* TDD 计划（先测什么）
* Out of scope（明确延后）

Gate C（允许拆 TASK）：
* 至少 1 个厚 STORY + 1 份 SLICE-001。

### Phase D：TECH v0/v1（tech）

TECH v0（可选）：只写关键分叉决策与最小架构。

TECH v1：对齐 SLICE，补齐数据/API/迁移/回滚/可观测。

硬护栏：
* 未满足 Gate C 时，tech 不输出“可执行 TASK 清单”；只输出风险与需要补齐的输入。

### Phase E：PROJ + TASK（proj + tech 建议）

proj 做两件事：
1) 决定本期只纳入“闭环 P0”还是加上部分增强（P1/P2）。
2) 把 Gate 写进 `PROJ`，并把“完成定义”落到每个 TASK。

**新增硬规则：Story→Slice→Task 可追溯性（Traceability）**

* 本期纳入的每个 `TASK` 必须能追溯到：
  * `STORY_ID`（来源需求）
  * `SLICE_ID`（属于哪个竖切闭环）
* 允许例外：技术债/纯重构/基础设施类任务可以标注 `NO_STORY`，但必须写清：
  * 为什么本期要做
  * 如何验收（可测试/真流程）

proj 必须在 `PROJ-*.md` 维护一张对齐表（本期纳入清单的“硬证据入口”）：

* `STORY_ID` → `SLICE_ID` → `TASK_ID 列表` → 本期纳入/不纳入 → 验收责任人

同时，proj 必须维护一张执行进度表（Progress Table）：
* 以 `TASK` 为粒度，跟踪 `TODO/DOING/BLOCKED/DONE`、Owner、预计与阻塞点；
* 证据入口必须链接到对应 `TASK-*.md`（测试/真流程/回滚/观测点都在 Task 文档回写）。

### Phase F：实现（dev，TDD 优先）

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
* UI 原型允许使用 mock 数据（为了快速对齐交互与状态机）；
* 交付验证（测试）必须包含"真数据真流程"的验证：
  * 最少要求：对本期纳入的 P0/P1 闭环，跑一遍可复现的端到端流程（Staging/测试租户/沙箱环境皆可）；
  * 单测/契约测试可以使用 stub/mock（用于覆盖边界与错误分支），但**不得**作为唯一验证手段；
* 所有证据回写到 `TASK-*.md`。

#### 🔧 文档与代码同步闭环（强制）

**当 Dev 发现文档假设错误时的处理流程**：

开发时如果发现文档（PRD/TECH/SLICE）中的假设与实际代码不符，**必须**按以下流程处理：

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
   - 影响 PRD/TECH/SLICE？→ 必须同步更新
    ↓
4. 更新上游文档（如需要）
   - 更新 PRD/TECH/SLICE 中的错误假设
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

## 4. TDD 作为 Gate（建议写入 PROJ）

* P0/P1 Task 进入 DONE 前必须有：
  * 对应 AC 的测试（自动化优先）与可复现的运行命令 + 结果
  * 无法自动化时的原因说明 + 手工验收脚本与证据
  * 回滚方案 + 上线观测点

---

## 5. 快速使用指南（最小集）

* 新建 Epic 时先创建：
  * `PRD-{{EPIC_ID}}-v0.md`
  * `docs/{{EPIC_DIR}}/prototypes/index.html`（可选但推荐）
  * `SLICE-{{EPIC_ID}}-001.md`
* 先做 SLICE-001 的闭环 P0，再迭代增强。
