# Vibedev Agents 工作流

本仓库提供一套"人格化子 Agent + 文档驱动"的交付工作流，重点是文档与模板，没有业务代码。目标是把 **UI 证据、Slice 规格和 TDD** 作为交付中心。

## 快速导航

| 文档 | 用途 |
|------|------|
| `docs/lib/workflow-overview.md` | 工作流规范（必读：Gate 机制、Phase 分工） |
| `docs/lib/templates/` | 所有模板（PRD/Story/Slice/Tech/Task/Proj/Prototype） |
| `docs/lib/template-mapping.md` | Agent ↔ 模板对应关系 |
| `.claude/agents/*.md` | 各 Agent 的详细配置 |

---

## 安装到你的项目

### 前置条件

- 已安装 [Claude Code](https://claude.com/claude-code)
- 你的项目是一个 Git 仓库

### 安装步骤

```bash
# 1. 克隆本仓库到你的项目旁边（不是项目内部）
git clone https://github.com/vibedevteam/vibedevteam.git

# 2. 复制核心内容到你的项目
cp -r vibedevteam/docs 你的项目根目录/
mkdir -p 你的项目根目录/.claude
cp -r vibedevteam/.claude/agents 你的项目根目录/.claude/

# 3. 可选：删除本仓库
rm -rf vibedevteam
```

### 目录结构（安装后）

你的项目根目录会多出：
```
你的项目/
  ├── .claude/
  │   └── agents/          # 子 Agent 配置
  │       ├── biz-owner.md
  │       ├── prd.md
  │       ├── ux.md
  │       ├── tech.md
  │       ├── proj.md
  │       └── dev.md
  └── docs/
      ├── lib/             # 可复用基础设施（模板、工作流规范）
      ├── _project/        # 项目级配置（你自己的）
      └── E-000-Agent-Workflow-Demo/   # 纯 demo，可删除
```

### 开始使用

安装完成后，直接在 Claude Code 中对话即可：

```
使用 biz-owner agent，帮我梳理这个想法：...
```

---

## 三种典型场景（基于 Claude Code）

### 场景一：从脑暴开始（biz-owner）

**适用情况**：你有一个模糊想法，不知道值不值得做、先做哪个、MVP 到哪里收手。

**在 Claude Code 中这样调用**：

```
使用 biz-owner agent，帮我梳理这个想法：

[描述你的想法，比如：想做履约群健康看板，但不确定 V1 应该做到什么程度]
```

**biz-owner 会做什么**：
- 不断问你问题（JTBD、单位经济、止损信号、优先级）
- 帮你把"一锅粥"收缩成"少数关键战役"
- 你说"差不多了"后，输出 `docs/_project/biz-overview.md` 草稿

**典型对话流**：
```
你: 使用 biz-owner agent，我有个想法...

biz-owner: 好的，先问几个问题：
1. 你最想解决的业务问题是什么？
2. 如果只能做一件事，你会选哪个？
3. 止损信号是什么？

[多轮对话后]

biz-owner: 当前共识小结：
- P1 问题：...
- MVP 方向：...
- 止损信号：...

你: 差不多了，帮我整理成 biz-overview.md

biz-owner: [生成文档内容，保存到 docs/_project/biz-overview.md]
```

---

### 场景二：确定产品，需要实现（prd → tech → dev）

**适用情况**：产品方向已确定，需要把需求变成可执行的代码任务。

**完整流程（按 Gate 顺序）**：

#### Step 1：PRD + UI 证据（prd + ux）

```
使用 prd agent，基于 docs/_project/biz-overview.md，写 PRD v0 和厚 Story，需要包含 UI 证据。

如果缺原型，先使用 ux agent 按 docs/lib/templates/tpl-prototype-index.html 生成一个可运行的 HTML 原型。
```

**Gate B（进入实现前必须满足）**：看完 UI 证据后能说"方向对了"。

#### Step 2：Slice Spec（prd）

```
使用 prd agent，补一份 SLICE-{{EPIC_ID}}-001.md（一刀竖切闭环），列闭环 AC。
```

#### Step 3：技术方案（tech）

```
使用 tech agent，读取 docs/{{EPIC_DIR}}/prd/PRD-{{EPIC_ID}}-v1.md 和 Story/Slice，产出 TECH-{{EPIC_ID}}-v1.md，注明复用/风险/Task 建议。
```

**Gate C（允许拆 TASK）**：至少 1 个厚 STORY + 1 份 SLICE。

#### Step 4：项目计划 + 任务卡片（proj + tech/proj）

```
使用 proj agent，写 PROJ-{{EPIC_ID}}-v1.md，填 Story → Slice → Task 对齐表、进度表，设 Gate 检查项。

使用 tech agent 或 proj agent，基于 Slice 建 TASK-*.md，标明 STORY_ID/SLICE_ID。
```

#### Step 5：开发与验证（dev）

```
使用 dev agent，基于 TASK-*.md 执行开发：
1. 先写测试或测试计划（TDD）
2. 再实现代码
3. 更新 TASK 文档（实现记录、测试记录、上线说明）
4. 标记 TASK 为 DOING
5. 请求 tech review（此时不要 commit！）
```

---

### 场景三：改 bug 直接改（dev → tech review）

**适用情况**：明确的 bug 修复，不需要走完整的 PRD 流程。

**简化的提交流程**：

```
使用 dev agent，帮我修复这个 bug：[描述 bug]

dev: [定位问题、实现修复、写测试、更新 TASK 文档]

dev: 标记 TASK 为 DOING，请求 tech review

[切换到 tech review 窗口]

使用 tech agent，review TASK-XXX 的代码变更：
- 检查 Commit 格式是否引用 TASK ID
- 检查文档同步（代码 + 文档原子提交）
- 检查复用性、基线符合性
- 给出 review 结果（通过/不通过）
```

**review 不通过时**：
```
tech: [列出必须修改项]

dev: [按意见修改，重新标记 DOING，请求 review]
[重复直到通过]
```

**review 通过时**：
```
tech: review 通过，可以执行 Git Commit 并标记 TASK 为 DONE

dev: [执行 commit，标记 DONE]
```

---

## dev + tech 双窗口交替工作流

**核心原则**：
- **dev 窗口**：负责写代码、改配置、更新文档（可写）
- **tech 窗口**：负责代码 review（只读，不直接改代码）

### 操作方式（两个 Claude Code 窗口）

| 窗口 | Agent | 职责 | 权限 |
|------|-------|------|------|
| 窗口 A | dev | 开发、测试、更新文档 | 可读写代码/配置 |
| 窗口 B | tech | 代码 review | 只读，不直接修改 |

### 标准流程

1. **dev 窗口开始工作**
```
使用 dev agent，基于 TASK-XXX 开始开发：
1. 先写测试/计划
2. 实现代码
3. 更新 TASK-XXX.md 文档
4. 标记状态为 DOING
```

2. **dev 请求 review**
```
dev: 代码和文档已更新完成，TASK-XXX 标记为 DOING，请求 tech review
```

3. **切换到 tech 窗口做 review**
```
使用 tech agent，review 以下变更：
- 对应的 TASK-XXX
- 检查点：
  1. 是否引用 TASK ID
  2. 文档是否同步更新
  3. 是否复用既有实现
  4. 是否符合技术基线
  5. 迁移/回滚是否可行
```

4. **tech 给出结果**

**review 不通过**：
```
tech: [列出必须修改项]
- 问题 1：...
- 问题 2：...
- [建议项]：...

请修改后重新 review。
```

**review 通过**：
```
tech: review 通过，可以执行 Git Commit 并标记 TASK-XXX 为 DONE。

Commit 格式参考：
git commit -m "fix(scope): description - refs TASK-XXX

- Fix summary
- Test coverage: X%

Docs: Updated TASK-XXX to DONE"
```

5. **dev 窗口执行 commit**
```
[执行 git commit]
[标记 TASK-XXX 为 DONE]
```

### 关键规则

1. **禁止提前 commit**：review 通过前，绝对不要执行 `git commit`
2. **代码 + 文档原子提交**：一次 commit 包含代码变更和文档变更
3. **Commit 必须引用 TASK ID**：格式 `refs TASK-XXX`
4. **review 不通过时**：按 tech 意见修改，重新从步骤 2 开始

---

## 目录结构

```text
/docs
  /lib/                         # 可复用基础设施
    workflow-overview.md     # 工作流规范（必读）
    template-mapping.md         # Agent ↔ 模板映射
    templates/                   # 所有模板
  /_project/                    # 项目级配置
    biz-overview.md             # 业务概览（biz-owner）
    tech-baseline.md            # 技术基线（tech）
    arch-overview.md            # 架构总览
    conventions/                # 规范（API/DB/日志等）
    adr/                        # 架构决策记录
  /{{EPIC_DIR}}                 # Epic 文档（例如 E-001-示例）
    prd/                        # 需求文档
    story/                      # 用户故事
    prototypes/                 # UI 原型
    slice/                      # Slice 规格
    tech/                       # 技术方案
    task/                       # 任务卡片
    proj/                       # 项目计划
```

---

## Agent 角色速查

| Agent | 职责 | 产物 |
|-------|------|------|
| **biz-owner** | 定方向/边界 | `docs/_project/biz-overview.md` |
| **prd** | 写 PRD/Story/Slice | PRD/Story/Slice 文档 |
| **ux** | 做 UI 原型 | `prototypes/index.html` |
| **tech** | 写技术方案、拆任务建议 | TECH 文档、TASK 建议 |
| **proj** | 维护 Gate、对齐表、进度 | PROJ 文档 |
| **dev** | 写代码、测试、更新文档 | 代码、TASK 文档回写 |

**重要约束**：
- 只有 **dev** 可以直接修改仓库代码/配置
- **tech** 只能做代码 review（只读），不能直接改代码
- 代码必须先通过 tech review 才能进仓库

---

## 核心概念

### Gate 机制（控制流程质量）

| Gate | 触发时机 | 检查项 |
|------|----------|--------|
| **Gate A** | 进入 PRD v1 前 | 目标/范围/非目标能复述，有止损信号 |
| **Gate B** | 进入实现前 | UI 证据存在（原型/截图/录屏），"方向对了" |
| **Gate C** | 允许拆 TASK | 至少 1 个厚 STORY + 1 份 SLICE-001 |

### Story → Slice → Task 可追溯性

每个 TASK 必须能追溯到：
- `STORY_ID`（来源需求）
- `SLICE_ID`（属于哪个竖切闭环）

例外：技术债/纯重构类任务可标注 `NO_STORY`，但需写清验收方式。

### TDD 规则

- **P0/P1**：默认先写测试（或至少先写测试计划）再写实现
- **UI 原型**：允许 mock 数据（为了快速对齐）
- **交付验证**：必须包含"真数据真流程"（Staging/测试租户/沙箱）

### Rebaseline（任何角色可触发）

触发条件：发现"不是想要的"或关键分叉决策改变

必须动作：
- PRD/TECH/PROJ 升版本并记录变更点
- 更新本期纳入清单（取消/延期/新增 TASK）
- 已完成能力明确归类（继续复用/后续增强/不再使用）

---

## 从零开始创建 Epic

> 注：`docs/E-000-Agent-Workflow-Demo/` 是一个纯 demo 展示，仅供参考。实际使用时，直接与 agent 对话即可，无需手动复制。

**你不需要手动创建目录**，agent 会自动处理。

### 对话流

```
你: 使用 prd agent，帮我创建一个新 Epic E-001：[描述你的功能]

prd: 好的，我先问几个问题...
[多轮对话，澄清需求、边界、主路径]

prd: 差不多了，我现在创建 docs/E-001-xxx/ 目录并生成 PRD v0、Story、Slice 文档
```

### 检查清单

Epic 创建完成后，确认：
- [ ] `docs/E-XXX-*/` 目录已自动创建
- [ ] PRD v0 有 UI 证据链接（原型/截图/录屏）
- [ ] 至少 1 个厚 STORY（主路径/状态机/AC）
- [ ] 至少 1 份 SLICE-001（闭环定义）
- [ ] 每个 TASK 能追溯到 STORY_ID + SLICE_ID

---

## 贡献规范

- 修改 `docs/lib/templates/` 时，同步更新 `docs/lib/template-mapping.md` 与各 Agent 的 `## 0.1 对应模板说明`
- 若 `docs/lib/workflow-overview.md` 变更，映射表与 Agent 说明也要同步
- 新增 Epic 请放到 `docs/E-XXX-*/`，不要直接改基础模板
