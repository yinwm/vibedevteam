# Changelog

All notable changes to the Vibedev Agents workflow will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
