# 使用 agent-skills 入门

agent-skills 与任何接受 Markdown 指令的 AI 编程代理配合使用。本指南介绍通用方法。对于特定工具的设置，请参阅专用指南。

## 技能如何工作

每个技能是一个 Markdown 文件（`SKILL.md`），描述特定的工程工作流。当加载到代理的上下文中时，代理遵循工作流 —— 包括验证步骤、要避免的反模式和退出标准。

**技能不是参考文档。** 它们是代理遵循的逐步流程。

## 快速开始（任何代理）

### 1. 克隆仓库

```bash
git clone https://github.com/addyosmani/agent-skills.git
```

### 2. 选择技能

浏览 `skills/` 目录。每个子目录包含一个 `SKILL.md`，带有：
- **何时使用** —— 指示此技能适用的触发条件
- **流程** —— 逐步工作流
- **验证** —— 如何确认工作完成
- **常见合理化** —— 代理可能用来跳过步骤的借口
- **危险信号** —— 技能被违反的迹象

### 3. 将技能加载到你的代理

将相关的 `SKILL.md` 内容复制到代理的系统提示、规则文件或对话中。最常见的方法：

**系统提示：** 在会话开始时粘贴技能内容。

**规则文件：** 将技能内容添加到项目的规则文件（CLAUDE.md、.cursorrules 等）。

**对话：** 在给出指令时引用技能："对此变更遵循 test-driven-development 流程。"

### 4. 使用元技能进行发现

从加载 `using-agent-skills` 技能开始。它包含一个流程图，将任务类型映射到适当的技能。

## 推荐设置

### 最小化（从这里开始）

将三个基本技能加载到规则文件：

1. **spec-driven-development** —— 用于定义要构建什么
2. **test-driven-development** —— 用于证明它有效
3. **code-review-and-quality** —— 用于在合并前验证质量

这三个覆盖了 AI 辅助开发中最关键的质量缺口。

### 完整生命周期

为了全面覆盖，按阶段加载技能：

```
启动项目时：  spec-driven-development → planning-and-task-breakdown
开发期间：    incremental-implementation + test-driven-development
合并前：      code-review-and-quality + security-and-hardening
部署前：      shipping-and-launch
```

### 上下文感知加载

不要一次性加载所有技能 —— 这会浪费上下文。加载与当前任务相关的技能：

- 处理 UI？加载 `frontend-ui-engineering`
- 调试？加载 `debugging-and-error-recovery`
- 设置 CI？加载 `ci-cd-and-automation`

## 技能结构

每个技能遵循相同的结构：

```
YAML frontmatter (name, description)
├── Overview — 此技能的作用
├── When to Use — 触发条件和情况
├── Core Process — 逐步工作流
├── Examples — 代码示例和模式
├── Common Rationalizations — 借口和反驳
├── Red Flags — 技能被违反的迹象
└── Verification — 退出标准检查清单
```

参见 [skill-anatomy.md](skill-anatomy.md) 了解完整规格。

## 使用代理

`agents/` 目录包含预配置的代理角色：

| 代理 | 用途 |
|-------|---------|
| `code-reviewer.md` | 五轴代码审查 |
| `test-engineer.md` | 测试策略和编写 |
| `security-auditor.md` | 漏洞检测 |

当你需要专门的审查时，加载代理定义。例如，要求你的编程代理"使用 code-reviewer 代理角色审查此变更"并提供代理定义。

## 使用命令

`.claude/commands/` 目录包含 Claude Code 的斜杠命令：

| 命令 | 调用的技能 |
|---------|---------------|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/review` | code-review-and-quality |
| `/ship` | shipping-and-launch |

## 使用参考

`references/` 目录包含补充检查清单：

| 参考 | 与...配合使用 |
|-----------|----------|
| `testing-patterns.md` | test-driven-development |
| `performance-checklist.md` | performance-optimization |
| `security-checklist.md` | security-and-hardening |
| `accessibility-checklist.md` | frontend-ui-engineering |

当你需要技能未涵盖的详细模式时，加载参考。

## 规格和任务工件

`/spec` 和 `/plan` 命令创建工作工件（`SPEC.md`、`tasks/plan.md`、`tasks/todo.md`）。将它们视为**活文档**，在工作进行中：

- 在开发期间将它们保留在版本控制中，以便人类和代理有共享的事实来源。
- 当范围或决策变更时更新它们。
- 如果你的仓库不希望长期保留这些文件，在合并前删除它们或将文件夹添加到 `.gitignore` —— 工作流不要求它们是永久的。

## 提示

1. **对于任何非平凡工作，从 spec-driven-development 开始**
2. **编写代码时始终加载 test-driven-development**
3. **不要跳过验证步骤** —— 这就是重点
4. **选择性加载技能** —— 更多上下文不一定更好
5. **使用代理进行审查** —— 不同视角发现不同问题