---
description: 执行五轴代码审查 —— 正确性、可读性、架构、安全性、性能
---

调用 agent-skills:code-review-and-quality 技能。

从五个轴审查当前变更（暂存或最近的提交）：

1. **正确性** —— 是否符合规格？处理了边界情况吗？测试充分吗？
2. **可读性** —— 命名清晰吗？逻辑直接吗？组织良好吗？
3. **架构** —— 遵循现有模式吗？边界清晰吗？抽象层次恰当吗？
4. **安全性** —— 输入已验证吗？密钥安全吗？认证检查了吗？（使用 security-and-hardening 技能）
5. **性能** —— 没有 N+1 查询？没有无界操作？（使用 performance-optimization 技能）

将发现分类为 Critical、Important 或 Suggestion。
输出结构化审查，带具体的 file:line 引用和修复建议。