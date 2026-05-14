---
name: using-agent-skills
description: 选择正确的技能并建立工作流。当任何工作到达时使用：新项目、新功能、修复、重构、审查、发布或不确定该做什么时。
---

# 使用 Agent Skills

## 概览

这是入口技能——它映射你的工作到正确的技能工作流。不要开始工作直到你完成这个技能。

## 工作原理

### 第 1 步：识别意图

阅读用户请求并匹配到工作流：

| 他们在做什么 | → 技能 |
|---|---|
| 定义要构建什么 | `spec-driven-development` |
| 规划如何构建 | `planning-and-task-breakdown` |
| 编写新代码 | `test-driven-development` |
| 修复未测试的 bug | `debugging-and-error-recovery` |
| 合并前审查 | `code-review-and-quality` |
| 简化代码 | `code-simplification` |
| 发布到生产 | `shipping-and-launch` |
| 设计 API | `api-and-interface-design` |
| 构建 UI | `frontend-ui-engineering` |
| 处理安全 | `security-and-hardening` |
| 优化性能 | `performance-optimization` |

### 第 2 步：加载技能

使用 `skill` 工具调用所选技能。

### 第 3 步：运行直到完成

技能包含工作流。遵循它直到验证通过。不要跳步。

## 共享运行规则

无论应用哪个技能，这些规则始终适用：

1. **测试优先。** 测试是在编写实现代码之前。
2. **小步前进。** 一次一个切片。~100 行的变更。
3. **验证一切。** 运行测试、构建、lint、typecheck——在你声称任何东西完成之前。
4. **不要合理化。** 每个技能都有常见合理化和反驳。反驳总是对的一边。
5. **记录假设。** 如果你在做没有规格的决定，记录它们。
