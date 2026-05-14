# 在 GitHub Copilot 中使用 agent-skills

## 设置

### Copilot 指令

Copilot 支持使用仓库中的 `.github/skills`、`.claude/skills` 或 `.agents/skills` 目录创建代理技能。

```bash
mkdir -p .github

# 为基本技能创建文件
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .github/skills/test-driven-development/SKILL.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md > .github/skills/code-review-and-quality/SKILL.md
```

更多细节，请参阅 [为 GitHub Copilot 创建代理技能](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills)。

### 代理角色（agents.md）

Copilot 支持专门的代理角色。使用 agent-skills 代理：

```bash
# 复制代理定义
cp /path/to/agent-skills/agents/code-reviewer.md .github/agents/code-reviewer.md
cp /path/to/agent-skills/agents/test-engineer.md .github/agents/test-engineer.md
cp /path/to/agent-skills/agents/security-auditor.md .github/agents/security-auditor.md
```

在 Copilot 聊天中调用代理：
- `@code-reviewer 审查这个 PR`
- `@test-engineer 分析此模块的测试覆盖率`
- `@security-auditor 检查此端点的漏洞`

### 自定义指令（用户级）

对于你希望跨所有仓库使用的技能：

1. 打开 VS Code → 设置 → GitHub Copilot → 自定义指令
2. 添加你最常用的技能摘要

## 推荐配置

### .github/copilot-instructions.md

GitHub Copilot 通过 `.github/copilot-instructions.md` 支持项目级指令。

```markdown
# 项目编码标准

## 测试
- 先编写测试后编写代码（TDD）
- 对于 bug：先编写失败的测试，然后修复（Prove-It 模式）
- 测试层次结构：单元 > 集成 > e2e（使用能捕获行为的最低层级）
- 每次变更后运行 `npm test`

## 代码质量
- 从五个轴审查：正确性、可读性、架构、安全性、性能
- 每个 PR 必须通过：lint、类型检查、测试、构建
- 代码或版本控制中无密钥

## 实现
- 以小的、可验证的增量构建
- 每个增量：实现 → 测试 → 验证 → 提交
- 永远不要将格式变更与行为变更混合

## 边界
- 始终：提交前运行测试、验证用户输入
- 先询问：数据库模式变更、新依赖
- 绝不：提交密钥、移除失败的测试、跳过验证
```

### 专门代理

在 Copilot 聊天中使用代理进行针对性的审查工作流。

## 使用提示

1. **保持指令简洁** —— Copilot 指令在专注时效果最好。总结关键规则而不是包含完整技能文件。
2. **使用代理进行审查** —— code-reviewer、test-engineer 和 security-auditor 代理专为 Copilot 的代理模型设计。
3. **在聊天中引用** —— 处理特定阶段时，将相关技能内容粘贴到 Copilot 聊天中以提供上下文。
4. **与 PR 审查结合** —— 设置 Copilot 使用 code-reviewer 代理角色审查 PR。