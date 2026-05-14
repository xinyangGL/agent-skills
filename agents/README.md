# Agent 角色（Personas）

专家级角色，每个角色承担单一职责并提供单一视角。每个角色是一个 Markdown 文件，作为系统提示词被你的调度器（Claude Code、Cursor、Copilot 等）消费。

| 角色 | 职责 | 最佳场景 |
|---------|------|----------|
| [code-reviewer](code-reviewer.md) | 高级资深工程师 | 合并前的五维度审查 |
| [security-auditor](security-auditor.md) | 安全工程师 | 漏洞检测、OWASP 风格审计 |
| [test-engineer](test-engineer.md) | 质量保障工程师 | 测试策略、覆盖率分析、Prove-It 模式 |

## 角色、技能与命令的关系

三个层级，各自承担不同的职责：

| 层级 | 是什么 | 示例 | 组合角色 |
|-------|-----------|---------|------------------|
| **Skill（技能）** | 带有步骤和退出条件的工作流 | `code-review-and-quality` | *如何（how）* —— 在角色或命令内部调用 |
| **Persona（角色）** | 带有视角和输出格式的职责 | `code-reviewer` | *谁（who）* —— 采用某种视角，产出报告 |
| **Command（命令）** | 面向用户的入口点 | `/review`、`/ship` | *何时（when）* —— 组合角色和技能 |

用户（或斜杠命令）是编排者。**角色不会调用其他角色。** 技能是角色工作流中必须经过的环节。

## 何时使用各角色

### 直接角色调用

当你希望从单一视角审视当前变更，且用户在循环中时，选择此项。

- "Review this PR" → 直接调用 `code-reviewer`
- "`auth.ts` 中是否有安全问题？" → 直接调用 `security-auditor`
- "结账流程缺少哪些测试？" → 直接调用 `test-engineer`

### 斜杠命令（背后是单一角色）

当存在可重复的工作流，否则你每次都要重新解释时，选择此项。

- `/review` → 使用项目的 review 技能包装 `code-reviewer`
- `/test` → 使用 TDD 技能包装 `test-engineer`

### 斜杠命令（编排者 —— 扇出）

仅当**独立的**调查可以并行运行，并生成报告由单一 agent 合并时，才选择此项。

- `/ship` → 并行扇出到 `code-reviewer` + `security-auditor` + `test-engineer`，然后将它们的报告综合为 go/no-go 决策

这是本仓库唯一认可的编排模式。完整模式目录和反模式请参阅 [references/orchestration-patterns.md](../references/orchestration-patterns.md)。

## 决策矩阵

```
这项工作是对单一工件的单一视角分析吗？
├── 是 → 直接角色调用
└── 否  → 子任务是否相互独立（无共享可变状态、无顺序依赖）？
         ├── 是 → 带并行扇出的斜杠命令（例如 /ship）
         └── 否  → 由用户按顺序运行的斜杠命令（/spec → /plan → /build → /test → /review）
```

## 示例：有效的编排

`/ship` 是本仓库中扇出编排者的典范：

```
/ship
  ├──（并行）code-reviewer    → 审查报告
  ├──（并行）security-auditor → 审计报告
  └──（并行）test-engineer    → 覆盖率报告
                  ↓
        合并阶段（主 agent）
                  ↓
        go/no-go 决策 + 回滚计划
```

为何有效：
- 每个子 agent 操作同一个 diff，但产出**不同的视角**
- 它们之间没有依赖关系 → 真正的并行化，实际节省墙钟时间
- 每个在独立的上下文窗口中运行 → 主会话保持整洁
- 合并步骤很小且受益于完整上下文，因此保留在主 agent 中

## 示例：无效的编排（不要构建此类）

一个 `meta-orchestrator` 角色，其职责是"决定调用哪个其他角色"：

```
/work-on-pr → meta-orchestrator
                  ↓（决定"这需要审查"）
              code-reviewer
                  ↓（返回结果）
              meta-orchestrator（转述结果）
                  ↓
              用户
```

为何失败：
- 纯路由层，无领域价值
- 增加两次转述跳跃 → 信息丢失 + 2× token 成本
- 用户已经知道自己想要审查；让他们直接调用 `/review`
- 重复了斜杠命令和 `AGENTS.md` 意图映射已经完成的工作

## 角色规则

1. 一个角色是单一职责，对应单一输出格式。如果你发现自己正在添加第二个职责，请创建第二个角色。
2. **角色不会调用其他角色。** 组合是斜杠命令或用户的职责。在 Claude Code 上，这也是一个硬性平台约束 —— *"subagents cannot spawn other subagents"* —— 所以该规则会自动为你强制执行。
3. 角色可以调用技能（*如何*）。
4. 每个角色文件末尾都有一个"组合（Composition）"区块，说明其适用位置。

## Claude Code 互操作

本仓库中的角色设计为可与 Claude Code 子 agent（subagents）和 Agent Teams 队友无缝协作，无需修改：

- **作为子 agent（subagents）：** 启用此插件时自动发现（无需路径配置）。使用 Agent 工具并指定 `subagent_type: code-reviewer`（或 `security-auditor`、`test-engineer`）。`/ship` 是典范示例。
- **作为 Agent Teams 队友**（实验性，需要 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`）：生成队友时引用相同的角色名称。角色的正文会**追加到**队友的系统提示词中作为额外指令（而非替换），因此你的角色文本位于领队安装的团队协调指令（SendMessage、task-list 工具等）之上。

子 agent 仅将结果报告回主 agent。Agent Teams 允许队友直接互相发送消息。当报告足够时使用子 agent；当子 agent 需要相互质疑发现结果时使用 Agent Teams（例如 competing-hypothesis 调试）。完整映射请参阅 [references/orchestration-patterns.md](../references/orchestration-patterns.md)。

插件 agent 不支持 `hooks`、`mcpServers` 或 `permissionMode` frontmatter —— 这些字段会被静默忽略。在此处编写新角色时，请避免依赖它们。

## 添加新角色

1. 使用现有角色相同的 frontmatter 格式创建 `agents/<role>.md`。
2. 定义职责、范围、输出格式和规则。
3. 在底部添加一个 **组合（Composition）** 区块（何时直接调用 / 通过什么调用 / 不要从其他角色调用）。
4. 将此角色添加到此文件顶部的表格中。
5. 如果该角色启用了新的编排模式，请在 `references/orchestration-patterns.md` 中记录该模式，而不是在角色文件本身中创造新模式。
