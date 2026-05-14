# 在 Windsurf 中使用 agent-skills

## 设置

### 项目规则

Windsurf 使用 `.windsurfrules` 进行项目特定的代理指令：

```bash
# 从你最重要的技能创建组合规则文件
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md >> .windsurfrules
echo "\n---\n" >> .windsurfrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .windsurfrules
```

### 全局规则

对于你希望跨所有项目使用的技能，将它们添加到 Windsurf 的全局规则：

1. 打开 Windsurf → 设置 → AI → 全局规则
2. 粘贴你最常用的技能内容

## 推荐配置

将 `.windsurfrules` 专注于 2-3 个基本技能以保持在上下文限制内：

```
# .windsurfrules
# 此项目的基本 agent-skills

[粘贴 test-driven-development SKILL.md]

---

[粘贴 incremental-implementation SKILL.md]

---

[粘贴 code-review-and-quality SKILL.md]
```

## 使用提示

1. **选择性加载** —— Windsurf 的上下文有限。选择解决你最大质量缺口的技能。
2. **在对话中引用** —— 处理特定阶段时将额外技能内容粘贴到聊天中（例如，构建认证时粘贴 `security-and-hardening`）。
3. **将参考用作检查清单** —— 粘贴 `references/security-checklist.md` 并要求 Windsurf 验证每一项。