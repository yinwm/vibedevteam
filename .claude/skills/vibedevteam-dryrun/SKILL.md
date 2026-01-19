---
name: vibedevteam-dryrun
description: 模拟运行检查，验证 STORY → TECH → TASK 链路是否完整。用于：1) STORY 验收前；2) TASK 拆解后；3) Phase 切换前。触发词：dryrun、模拟运行、链路检查。
---

# VibeDevTeam: 模拟运行检查

## 快速开始

当用户要求 dry run 或模拟运行时：

1. **确认检查范围**：STORY_ID 或 EPIC
2. **读取链路检查**：加载 `链路检查.md`
3. **执行模拟**：从 STORY 出发，模拟完整链路
4. **输出报告**：按 `报告模板.md` 格式输出

## 触发条件

用户说类似：
- "dry run"
- "模拟运行 STORY-001"
- "检查链路"
- "验证 STORY → TASK"

## 模拟运行流程

### Step 1: 确定检查范围

```
单个 STORY：STORY-001
多个 STORY：STORY-001, STORY-002
整个 EPIC：E-001
```

### Step 2: 读取链路

按顺序读取：
1. PRD：`docs/{{EPIC_DIR}}/prd/PRD-{{EPIC_ID}}-v1.md`
2. STORY：`docs/{{EPIC_DIR}}/story/STORY-*.md`
3. TECH：`docs/{{EPIC_DIR}}/tech/TECH-{{EPIC_ID}}-v1.md`
4. PROJ：`docs/{{EPIC_DIR}}/proj/PROJ-{{EPIC_ID}}-v1.md`
5. TASK：`docs/{{EPIC_DIR}}/task/TASK-*.md`

### Step 3: 执行链路检查

使用 `链路检查.md` 中的 6 步检查：
1. PRD → STORY
2. STORY → TECH
3. STORY → TASK
4. TASK 依赖
5. TASK 验收 → STORY AC
6. 真流程验证

### Step 4: 输出报告

按 `报告模板.md` 格式输出，包含：
- 检查结果汇总
- P0/P1/P2 问题列表
- 改进建议

## 问题级别

使用 `问题级别.md` 定义问题严重程度：

| 级别 | 定义 | 处理方式 |
|------|------|----------|
| **P0** | 阻塞，无法继续开发 | 必须修复 |
| **P1** | 严重，后续大量返工 | 建议修复 |
| **P2** | 警告，可后续优化 | 可选修复 |

## 示例对话

```
用户：dry run STORY-001

Claude：
好的，我来做 STORY-001 的模拟运行检查。

[读取 PRD, STORY, TECH, PROJ, TASK]

# 模拟运行报告：STORY-001

## 链路检查结果

### 1. PRD → STORY
✓ PRD 有 F1 描述对应 STORY-001
✓ PRD AC 可追溯到 STORY

### 2. STORY → TECH
✗ STORY-001 提到"获取群列表接口"
✗ TECH 没有定义这个接口
→ P0 问题

...

## 发现的问题

### P0 - 阻塞
1. TECH 缺少"获取群列表接口"定义

### P1 - 严重
1. STORY-001 异常态描述不足

## 建议

1. 补充 TECH-E-001-v1.md 接口契约
2. 细化 STORY-001 边界描述

**整体结论**：需修复后通过
```

## 资源

### references/

- `链路检查.md` - 6 步链路检查定义
- `问题级别.md` - P0/P1/P2 问题定义
- `报告模板.md` - 报告格式模板
