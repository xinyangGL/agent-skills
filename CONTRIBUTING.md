# 为 Agent Skills 贡献

感谢你的贡献兴趣！本项目是面向 AI 编程助手的生产级工程技能集合。

## 添加新技能

1. 在 `skills/` 下创建 kebab-case 命名的目录
2. 按照 [docs/skill-anatomy.md](docs/skill-anatomy.md) 中的格式添加 `SKILL.md`
3. 包含带有 `name` 和 `description` 字段的 YAML frontmatter
4. 确保 `description` 以技能的用途开头（第三人称），然后包含一个或多个 `Use when` 触发条件

### 技能质量标准

技能应该：

- **具体** — 可操作的步骤，不是模糊的建议
- **可验证** — 清晰的退出标准和证据要求
- **经过实战检验** — 基于真实的工程工作流，不是理论理想
- **精简** — 仅引导代理所需的内容

### 结构

每个新技能必须包含：

- 技能目录中的 `SKILL.md`
- 带有有效 `name` 和 `description` 的 YAML frontmatter

新技能通常应遵循标准结构：

- **Overview** — 此技能的作用及其重要性
- **When to Use** — 触发条件
- **Process** — 逐步工作流
- **Common Rationalizations** — 代理用来跳过步骤的借口及反驳
- **Red Flags** — 技能被错误应用的警告信号
- **Verification** — 如何确认技能被正确应用

上述 frontmatter 字段是必需的。节的结构是推荐模式：如 `How It Works`、`Workflow` 或 `Core Process` 等等效标题在保持相同意图且使技能易于遵循是可以接受的。

### 不应该做的事

- 不要在技能之间重复内容——改为引用其他技能
- 不要添加模糊建议而非可执行流程的技能
- 不要创建辅助文件除非内容超过 100 行
- 不要为了匹配其他技能而创建空的 `scripts/` 目录——仅当技能包含可运行辅助工具时才添加 `scripts/`
- 不要将参考资料放在技能目录内——使用 `references/`

## 修改现有技能

- 保持变更聚焦和最小化
- 保留现有结构和语气
- 测试编辑后 YAML frontmatter 仍然有效

## 测试钩子

session-start 钩子（`hooks/session-start.sh`）将 `using-agent-skills` 元技能注入每个新的 Claude Code 会话。`hooks/session-start-test.sh` 处的回归测试验证钩子的 JSON 有效负载——无论 `jq` 是否可用。

在提交任何触及以下内容的 PR 之前运行它：

- `hooks/session-start.sh`
- `skills/using-agent-skills/SKILL.md`（钩子嵌入的元技能内容）

```bash
bash hooks/session-start-test.sh
```

预期输出：`session-start JSON payload OK`。脚本在任何断言失败时以非零退出。

### 复现 no-jq 回退

当 `jq` 不在 `PATH` 上时，钩子优雅降级为 `INFO` 优先级的有效负载。要在本地练习该分支，从测试调用的 `PATH` 中剥离 `jq` 的目录：

```bash
JQ_DIR=$(dirname "$(command -v jq)")
PATH=$(echo "$PATH" | tr ':' '\n' | grep -v "^${JQ_DIR}$" | tr '\n' ':' | sed 's/:$//') \
  bash hooks/session-start-test.sh
```

当 `jq` 位于其自己的目录中时（例如 Homebrew 的 `/opt/homebrew/bin`、手动安装的 `/usr/local/bin`），此方法工作良好。如果你的 `jq` 与测试依赖的其他工具共享系统 bin（如 `/usr/bin` 中的 `mktemp`），更简单的方法是通过单独的包管理器安装 `jq`，使其拥有自己的 bin 目录，然后重新运行。

钩子的 `command -v jq` 检查在剥离的 `PATH` 下失败，`INFO` 优先级回退运行，测试断言 `jq is required` 指导消息而不是正常有效负载。

## 报告问题

如果你发现以下问题，请打开 issue：

- 技能给出错误或过时的指导
- 缺少常见工程工作流的覆盖
- 技能之间不一致

## 许可证

通过贡献，你同意你的贡献将按 MIT 许可证授权。
