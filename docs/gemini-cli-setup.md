# 在 Gemini CLI 中使用 agent-skills

## 设置

### 选项 1：安装为技能（推荐）

Gemini CLI 具有原生技能系统，自动发现 `.gemini/skills/` 或 `.agents/skills/` 目录中的 `SKILL.md` 文件。每个技能在匹配任务时按需激活。

**从仓库安装：**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**或从本地克隆安装：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
gemini skills install /path/to/agent-skills/skills/
```

**仅为特定工作区安装：**

```bash
gemini skills install /path/to/agent-skills/skills/ --scope workspace
```

工作区范围安装的技能进入 `.gemini/skills/`（或 `.agents/skills/`）。用户级技能进入 `~/.gemini/skills/`。

安装后，使用以下命令验证：

```
/skills list
```

Gemini CLI 自动将技能名称和描述注入提示。当它识别匹配的任务时，它在加载完整指令前请求激活技能的权限。

### 选项 2：GEMINI.md（持久上下文）

对于你希望始终加载为持久项目上下文（而非按需激活）的技能，将它们添加到项目的 `GEMINI.md`：

```bash
# 创建带有基本技能作为持久上下文的 GEMINI.md
cat /path/to/agent-skills/skills/incremental-implementation/SKILL.md > GEMINI.md
echo -e "\n---\n" >> GEMINI.md
cat /path/to/agent-skills/skills/code-review-and-quality/SKILL.md >> GEMINI.md
```

你还可以通过从单独文件导入来模块化：

```markdown
# 项目指令

@skills/test-driven-development/SKILL.md
@skills/incremental-implementation/SKILL.md
```

使用 `/memory show` 验证加载的上下文，使用 `/memory reload` 在变更后刷新。

> **技能 vs GEMINI.md：** 技能是仅在相关时激活的按需专业知识，保持上下文窗口干净。GEMINI.md 提供每个提示加载的持久上下文。对阶段特定工作流使用技能，对始终启用的项目约定使用 GEMINI.md。

## 推荐配置

### 始终启用（GEMINI.md）

将这些添加为每个会话的持久上下文：

- `incremental-implementation` —— 以小的可验证切片构建
- `code-review-and-quality` —— 五轴审查

### 按需（技能）

将这些安装为技能，以便它们仅在相关时激活：

- `test-driven-development` —— 实现逻辑或修复 bug 时激活
- `spec-driven-development` —— 启动新项目或功能时激活
- `frontend-ui-engineering` —— 构建 UI 时激活
- `security-and-hardening` —— 安全审查期间激活
- `performance-optimization` —— 性能工作期间激活

## 高级配置

### MCP 集成

本包中的许多技能利用 [模型上下文协议（MCP）](https://modelcontextprotocol.io/) 工具与环境交互。例如：

- `browser-testing-with-devtools` 使用 `chrome-devtools` MCP 扩展。
- `performance-optimization` 可以从性能相关 MCP 工具中受益。

要启用这些，确保你在 Gemini CLI 配置（`~/.gemini/config.json`）中安装了相关 MCP 扩展。

### 会话钩子

Gemini CLI 支持会话生命周期钩子。你可以使用它们在会话开始时自动注入上下文或运行业验证脚本。

要复制其他工具的 `agent-skills` 体验，你可以配置 `SessionStart` 钩子来提醒你可用技能或加载元技能。

### 显式上下文加载

你可以在提示中使用 `@` 符号显式将任何技能加载到当前会话：

```markdown
使用 @skills/test-driven-development/SKILL.md 技能实现此修复。
```

这在你希望确保遵循特定工作流而不等待自动发现时很有用。

## 斜杠命令

仓库随附 7 个斜杠命令，位于 `.gemini/commands/` 下，映射到开发生命周期。Gemini CLI 在从项目根目录运行时自动发现它们。

| 命令 | 作用 |
|---------|--------------|
| `/spec` | 在编写代码前编写结构化规格 |
| `/planning` | 将工作拆解为小的、可验证的任务 |
| `/build` | 增量实现下一个任务 |
| `/test` | 运行 TDD 工作流 —— 红、绿、重构 |
| `/review` | 五轴代码审查 |
| `/code-simplify` | 在不改变行为的情况下降低复杂度 |
| `/ship` | 通过并行角色扇出运行发布前检查清单 |

每个命令自动调用相应的技能 —— 无需手动加载技能。

> **注意：** 使用 `/planning` 而不是 `/plan` —— `/plan` 与 Gemini CLI 内部命令名称冲突。

## 使用提示

1. **优先使用技能而不是 GEMINI.md** —— 技能按需激活，保持上下文窗口专注。仅当你希望技能始终加载时才将其放入 GEMINI.md。
2. **技能描述很重要** —— 每个 SKILL.md 的 frontmatter 中有 `description` 字段，告诉代理何时激活它。本仓库中的描述针对所有支持工具（Claude Code、Gemini CLI 等）的自动发现进行了优化，通过清晰说明技能*做什么*以及*何时*触发。
3. **使用代理进行审查** —— 请求结构化代码审查时复制 `agents/code-reviewer.md` 内容。
4. **与参考结合** —— 处理特定质量领域（如测试或性能）时，引用 `references/` 中的检查清单。