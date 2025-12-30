# 模板映射表

本文档说明各 Agent 使用的文档模板及其对应关系。

## 变更同步链（重要）

**当本文档或任何模板文件变更时，必须同步更新以下文件：**

1. **`.claude/agents/*.md`**（各 Agent 配置）
   - 每个 Agent 的 `## 0.1 对应模板说明` 章节
   - 确保模板引用、变量说明、内容结构保持一致

2. **`/docs/lib/workflow-overview.md`**
   - `## 2. 文档地图` 章节：保持 `/docs/lib/` 和 `/docs/lib/templates/` 结构同步
   - 模板映射说明段落（指向 `/docs/lib/template-mapping.md`）

**变更检查清单**：
- [ ] 更新了 `template-mapping.md` 的映射表？
- [ ] 同步更新了对应 Agent 的 `## 0.1 对应模板说明`？
- [ ] 同步更新了 `workflow-overview.md` 的文档地图？

---

## 全局变量约定

| 变量 | 说明 | 示例 |
|------|------|------|
| `{{EPIC_ID}}` | Epic 编号 | `E-001` |
| `{{EPIC_DIR}}` | Epic 目录名（建议格式：ID-名称） | `E-001-履约群健康看板-V1` |
| `{{EPIC_NAME}}` | Epic 名称 | `履约群健康看板` |
| `{{VERSION}}` 或 `{{N}}` | 版本号 | `v1`, `v0.1` |
| `{{OWNER}}` | 作者/负责人 | `@username` |
| `{{YYYY-MM-DD}}` | 日期 | `2024-01-15` |

## Agent → Template 映射

| Agent | 模板文件 | 输出文档路径 | 用途说明 |
|-------|---------|-------------|---------|
| **biz-owner** | `tpl-biz-overview.md` | `/docs/_project/biz-overview.md` | 业务概览（商业模式、目标、Epic 列表） |
| **prd** | `tpl-prd.md` | `/docs/{{EPIC_DIR}}/prd/PRD-{{EPIC_ID}}-v{{N}}.md` | Epic PRD（v0 探索版、v1 可开发版） |
| **prd** | `tpl-story.md` | `/docs/{{EPIC_DIR}}/story/STORY-*.md` | 用户故事（厚 Story：主路径/状态机/AC/边界/契约） |
| **prd** | `tpl-slice-spec.md` | `/docs/{{EPIC_DIR}}/slice/SLICE-{{EPIC_ID}}-*.md` | 竖切闭环规格（驱动 TASK 拆解） |
| **tech** | `tpl-tech-epic.md` | `/docs/{{EPIC_DIR}}/tech/TECH-{{EPIC_ID}}-v{{N}}.md` | Epic 技术方案（现状/方案/设计/NFR/TASK 建议） |
| **tech/proj** | `tpl-task.md` | `/docs/{{EPIC_DIR}}/task/TASK-*.md` | 任务卡片（可执行、可验收） |
| **tech** | `tpl-code-review-tech.md | 评审时使用 | 代码评审检查项（复用/基线/架构/迁移/NFR） |
| **proj** | `tpl-proj-epic.md` | `/docs/{{EPIC_DIR}}/proj/PROJ-{{EPIC_ID}}-v{{N}}.md` | Epic 项目计划（范围/排期/里程碑/Gate） |
| **proj** | `tpl-proj-roadmap.md` | `/docs/_project/proj-roadmap.md` | 业务线 Roadmap（多 Epic 优先级/排期） |
| **ux** | `tpl-prototype-index.html` | `/docs/{{EPIC_DIR}}/prototypes/index.html` | 可运行 HTML 原型（UI 证据） |

## 模板内容结构速查

### tpl-biz-overview.md（业务概览）
- 背景与核心问题
- 商业模式与目标用户
- 业务结果指标与止损信号
- Epic 列表与优先级
- [OPEN]/[TBD] 未决项

### tpl-prd.md（PRD）
- 文档元信息（EPIC_ID、状态、关联文档）
- 背景与动机
- 目标与成功标准（引用 biz-overview）
- 范围与非目标（In/Out）
- 用户与场景
- 功能需求（F1/F2/... 编号，含规则/权限/边界）
- 交互与信息结构（ASCII 线框可选）
- **UI 证据**（原型/截图/录屏，Gate B 硬要求）
- 非功能需求（性能/权限/合规）
- 需求层可观测性（埋点/日志/报表）
- 发布策略（需求层：开关/灰度/回滚）
- AI Native 附录（可选，仅涉及 AI 时）
- 版本验收标准
- [OPEN]/[TBD]

### tpl-story.md（用户故事 - 厚 STORY）
- Story ID / EPIC_ID / EPIC_DIR
- 关联 PRD 相对路径
- User Story 文本
- 背景与动机
- 主要使用场景（主路径步骤）
- **状态机**（关键状态与跃迁，含恢复路径）
- 验收标准（AC 列表，可测试）
- 边界与异常（空/失败/权限/重复/超时）
- 接口契约草案（请求/响应示例）
- UI 证据引用
- 依赖与备注

### tpl-slice-spec.md（竖切闭环规格）
- Slice ID / EPIC_ID
- 关联 Story 列表
- 竖切说明（为什么这样切）
- 闭环定义（从哪到哪可独立验收）
- 包含的 AC 清单
- 技术依赖与风险
- 验收方式

### tpl-tech-epic.md（技术方案）
- 文档元信息（EPIC_ID、状态、关联文档）
- 目标与范围对齐（**引用上游，不重写**）
- 现状与约束（**必须先读代码**）
  - 相关代码/系统现状
  - **复用清单**（避免重复造轮子）
  - 硬约束
- 方案总览（1 页能讲清：一句话/架构变化/trade-off/影响面）
- 详细设计
  - 模块/服务边界与职责
  - 数据模型（表结构/索引/迁移）
  - API 契约（请求/响应/错误码）
  - 可观测性（日志/指标/tracing/报表）
  - 安全与权限（鉴权/审计/合规）
- NFR（性能/可靠性/兼容性）
- 迁移与上线策略（数据迁移/回滚/灰度）
- 风险与预案
- **TASK 拆解建议**（基于 Story/SLICE，标注 P0/P1/P2）
- [CONFLICT_WITH_BASELINE]（如有偏离基线）
- [OPEN]/[TBD]

### tpl-task.md（任务卡片）
- Task ID / EPIC_ID
- 关联 STORY_ID / SLICE_ID
- 标题与描述
- 验收标准（引用 Story AC，细化到可执行）
- 技术要点（引用 TECH 中的方案）
- 实现记录（开发时的技术决策/坑点）
- 测试记录（单测/集成/真流程验证）
- 状态（TODO/DOING/BLOCKED/DONE）
- 依赖与风险

### tpl-proj-epic.md（项目计划）
- 文档元信息（EPIC_ID、状态、关联文档）
- 项目概述（名称/目标/指标）
- 范围说明
  - 本期包含的 Story/Task
  - **Story → Slice → Task 对齐表**（必填）
  - **执行进度表**（必填）
- 资源配置（角色/人数/时间）
- 时间计划（上线目标/里程碑）
- 里程碑完成定义（DoD）
- 任务拆解与优先级
- 风险与预案
- **Gate 检查点**（Gate A/B/C）
- 变更记录

### tpl-proj-roadmap.md（业务线 Roadmap）
- 业务线概览
- Epic 列表与优先级
- 时间规划（Q1/Q2/Q3/Q4 或具体月份）
- 资源与依赖
- 风险与预案
- 变更记录

### tpl-code-review-tech.md（代码评审检查项）
- 复用性检查
- 基线符合性检查
- 架构边界检查
- 迁移与回滚可行性
- NFR 检查（日志/性能/安全）
- 评审结果与修改建议

## 模板文件位置

所有模板文件位于：`/docs/lib/templates/`

## 使用建议

1. **Agent 优先**：各 Agent 在输出文档前，应先读取对应模板了解结构
2. **变量替换**：输出时将 `{{VAR}}` 替换为实际值
3. **可选章节**：模板中标注"可选"的章节，根据实际情况决定是否填充
4. **引用而非重复**：Epic 级文档引用项目级基线，不要重复粘贴
5. **未决项管理**：用 `[OPEN]` 标记问题，`[TBD]` 标记待定值，`[ASSUMPTION]` 标记假设
