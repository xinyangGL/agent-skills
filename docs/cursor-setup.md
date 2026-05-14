# 在 Cursor 中使用 agent-skills

## 设置

### 选项 1：规则目录（推荐）

Cursor 支持用于项目特定规则的 `.cursor/rules/` 目录：

```bash
# 创建规则目录
mkdir -p .cursor/rules

# 复制你希望作为规则的技能
cp /path/to/agent-skills/skills/test-driven-development/SKILL.md .cursor/rules/test-driven-development.md
cp /path/to/agent-skills/skills/code-review-and-quality/SKILL.md .cursor/rules/code-review-and-quality.md
cp /path/to/agent-skills/skills/incremental-implementation/SKILL.md .cursor/rules/incremental-implementation.md
```

此目录中的规则会自动加载到 Cursor 的上下文中。

### 选项 2：.cursorrules 文件

在项目根目录创建 `.cursorrules` 文件，内联基本技能：

```bash
# 生成组合规则文件
cat /path/to/agent-skills/skills/test-driven-development/SKILL.md > .cursorrules
echo "\n---\n" >> .cursorrules
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> .cursorrules
```

## 推荐配置

### 基本技能（始终加载）

将这些添加到 `.cursor/rules/`：

1. `test-driven-development.md` —— TDD 工作流和 Prove-It 模式
2. `code-review-and-quality.md` —— 五轴审查
3. `incremental-implementation.md` —— 以小的可验证切片构建

### 阶段特定技能（按需加载）

对于阶段特定的工作，根据需要创建额外的规则文件：

- `spec-development.md` -> `spec-driven-development/SKILL.md`
- `frontend-ui.md` -> `frontend-ui-engineering/SKILL.md`
- `security.md` -> `security-and-hardening/SKILL.md`
- `performance.md` -> `performance-optimization/SKILL.md`

在处理相关任务时将这些添加到 `.cursor/rules/`，完成后移除以管理上下文限制。

## 使用提示

1. **不要一次性加载所有技能** - Cursor 有上下文限制。将 2-3 个基本技能作为规则加载，并按需添加阶段特定技能。
2. **显式引用技能** - 告诉 Cursor"对此变更遵循 test-driven-development 规则"以确保它读取已加载的规则。
3. **使用代理进行审查** - 复制 `agents/code-reviewer.md` 内容并告诉 Cursor"使用此代码审查框架审查此 diff。"
4. **按需加载参考** - 处理性能时，将 `performance.md` 添加到 `.cursor/rules/` 或直接粘贴检查清单内容。