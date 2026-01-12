# Changelog

All notable changes to the Vibedev Agents workflow will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0] - 2025-01-12

### Added
- **beads 任务管理强制集成**: 所有 agent 强制使用 beads 管理任务状态和依赖
  - TASK 文档新增 `BEADS_ID` 字段，任务状态以 beads 为准
  - 新增 3 个自动化 skills: `vibedevteam-init`, `vibedevteam-sync`, `vibedevteam-graph`
  - proj.md 新增完整 beads 操作指南（§9）和无限 AI Dev 场景策略（§10）

- **依赖类型区分**: 区分"硬依赖"和"接口依赖"以提升并行度
  - **硬依赖**: 代码必须的依赖（如 Handler 调用 Service），设置 `bd dep add`
  - **接口依赖**: 联调需要的依赖（如前端调用后端 API），采用契约先行策略
  - tpl-tech-epic.md 新增依赖分析表格

- **提交流程硬化**: dev 提交流程强化为"生命线"
  - 强制先 review 后 commit（违反需返工）
  - 明确违反后果：代码需重新 review、可能返工、不符合 DoD

- **测试基建强制要求**:
  - 前端测试框架必须配置（Vitest/Jest）
  - 后端测试框架必须配置（testify）
  - 测试覆盖率目标：70%
  - 真数据真流程验证（P0/P1 强制）

### Changed
- **tech.md 文档创建职责**: 明确 tech agent 负责创建 TASK 文档（不仅是建议）

- **tpl-prd.md 精简结构**:
  - 删除"需求层可观测性"独立章节
  - 删除"发布策略"独立章节
  - 合并到"业务约束与风险"
  - 从 14 章节减少到 10 章节

- **tpl-task.md 依赖标注**:
  - 上游依赖分为：硬依赖、接口依赖、无依赖
  - 新增"禁止使用 mock"强制说明
  - 状态以 beads 为准，文档只回写证据

- **tpl-tech-epic.md 接口契约强制化**:
  - 新增完整 API 契约模板
  - 新增接口契约检查点（契约不完整禁止输出 TECH）
  - 新增 Mock 禁止检查点

### Fixed
- **proj.md 调度指令模板**: 新增标准 dev 调度指令模板，确保不遗漏提交流程提醒

---

## [0.2.0] - 2025-01-07

### Changed
- **dev.md 精简优化**: 从 1018 行精简至 655 行（减少 36%）
  - 删除所有教程式多语言代码示例
  - 保留核心原则和可执行的检查点
  - 优化为"老手工程师"定位，聚焦可执行标准而非入门教程

- **TDD 工作流程明确**: 区分 tech 和 dev 在测试中的职责
  - **tech**: 在 TECH 文档中定义测试策略和测试点
  - **dev**: 执行 TDD，写具体测试代码（红 → 绿 → 重构）

- **测试优先级调整**: 按优先级排序
  1. 真实实现优先: Test Container（数据库）、测试环境 API
  2. 集成测试: 真实流程串联，少量 mock（只 mock 不可控的外部依赖）
  3. 单元测试: 只在必要时 mock，mock 必须和真实契约对齐

### Removed
- **dev.md 教程式内容**:
  - 原则 2: 多语言假实现示例（~120 行）
  - 原则 3: 可见性表格和多语言示例（~62 行）
  - 原则 4: 方法签名多语言示例（~116 行）
  - 原则 5: 日志规范多语言示例（~46 行）
  - 原则 6: 并发安全详细示例（~38 行）
  - 能力卡片中的重复内容（~60 行）

### Fixed
- **dev.md 提交前自检清单**: 添加 TDD 检查项（P0/P1 是否遵循 TDD？先写测试再写实现）

---

## [0.1.0] - 2024-12-20

### Added
- **初始版本**: Vibedev Agents 工作流系统
  - 6 个核心 agent: biz-owner, prd, ux, tech, proj, dev
  - 完整的 Gate 门槛和 Rebaseline 机制
  - TDD 硬性要求（P0/P1 默认先写测试）

---
