# Agent Skills

**面向 AI 编程助手的生产级工程技能。**

技能封装了高级工程师在构建软件时使用的工作流、质量门禁和最佳实践。这些技能被打包，以便 AI 助手在开发的每个阶段都能一致地遵循它们。

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

---

## 命令

7 个映射到开发生命周期的斜杠命令。每个命令自动激活相应的技能。

| 你在做什么 | 命令 | 核心原则 |
|-----------|------|---------|
| 定义要构建什么 | `/spec` | 先规格后代码 |
| 规划如何构建 | `/plan` | 小而原子的任务 |
| 增量构建 | `/build` | 一次一个切片 |
| 证明它有效 | `/test` | 测试即证明 |
| 合并前审查 | `/review` | 改善代码健康 |
| 简化代码 | `/code-simplify` | 清晰胜过巧妙 |
| 发布到生产 | `/ship` | 更快更安全 |

技能还会根据你的工作内容自动激活——设计 API 会触发 `api-and-interface-design`，构建 UI 会触发 `frontend-ui-engineering`，依此类推。

---

## 快速开始

<details>
<summary><b>Claude Code（推荐）</b></summary>

**Marketplace 安装：**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH 错误？** Marketplace 通过 SSH 克隆仓库。如果你没有在 GitHub 上设置 SSH 密钥，要么[添加你的 SSH 密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)，要么使用完整的 HTTPS URL 强制使用 HTTPS 克隆：
> ```bash
> /plugin marketplace add https://github.com/addyosmani/agent-skills.git
> /plugin install agent-skills@addy-agent-skills
> ```

**本地 / 开发：**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

将任意 `SKILL.md` 复制到 `.cursor/rules/`，或引用完整的 `skills/` 目录。详见 [docs/cursor-setup.md](docs/cursor-setup.md)。

</details>

<details>
<summary><b>Gemini CLI</b></summary>

安装为原生技能以自动发现，或添加到 `GEMINI.md` 作为持久上下文。详见 [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md)。

**从仓库安装：**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**从本地克隆安装：**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Windsurf</b></summary>

将技能内容添加到 Windsurf 规则配置中。详见 [docs/windsurf-setup.md](docs/windsurf-setup.md)。

</details>

<details>
<summary><b>OpenCode</b></summary>

通过 AGENTS.md 和 `skill` 工具使用代理驱动的技能执行。

详见 [docs/opencode-setup.md](docs/opencode-setup.md)。

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

使用 `agents/` 中的代理定义作为 Copilot 角色，在 `.github/copilot-instructions.md` 中使用技能内容。详见 [docs/copilot-setup.md](docs/copilot-setup.md)。

</details>

<details>
  <summary><b>Kiro IDE & CLI </b></summary>
  Kiro 的技能位于 ".kiro/skills/" 下，可以存储在 Project 或 Global 级别。Kiro 也支持 Agents.md。详见 Kiro 文档 https://kiro.dev/docs/skills/
</details>

<details>
<summary><b>Codex / 其他代理</b></summary>

技能是纯 Markdown——它们适用于任何接受系统提示或指令文件的代理。详见 [docs/getting-started.md](docs/getting-started.md)。

</details>

---

## 全部 23 个技能

上面的命令是入口点。本包共包含 23 个技能——22 个生命周期技能加上 `using-agent-skills` 元技能。每个技能都是结构化的工作流，包含步骤、验证门禁和反合理化表格。你也可以直接引用任意技能。

### 元技能 - 发现哪个技能适用

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | 将传入工作映射到正确的技能工作流，并定义共享运行规则 | 开始会话或决定哪个技能适用时 |

### 定义 - 明确要构建什么

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [interview-me](skills/interview-me/SKILL.md) | 一次一问的面试流程，提取用户真正想要的，而不是他们以为应该要的，直到 ~95% 置信度 | 需求不明确，或用户调用 "interview me" / "grill me" 时 |
| [idea-refine](skills/idea-refine/SKILL.md) | 结构化的发散/收敛思维，将模糊想法转化为具体提案 | 你有大致概念需要探索时 |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | 在编写任何代码之前，编写涵盖目标、命令、结构、代码风格、测试和边界的 PRD | 启动新项目、功能或重大变更时 |

### 计划 - 拆解

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | 将规格拆解为小的、可验证的任务，包含验收标准和依赖排序 | 你有规格需要可执行的单元时 |

### 构建 - 编写代码

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | 薄垂直切片——实现、测试、验证、提交。特性标志、安全默认值、可回滚的变更 | 任何影响多个文件的变更 |
| [test-driven-development](skills/test-driven-development/SKILL.md) | 红-绿-重构、测试金字塔（80/15/5）、测试大小、DAMP 优于 DRY、Beyonce 规则、浏览器测试 | 实现逻辑、修复 Bug 或更改行为时 |
| [context-engineering](skills/context-engineering/SKILL.md) | 在正确的时间向代理提供正确的信息——规则文件、上下文打包、MCP 集成 | 开始会话、切换任务或输出质量下降时 |
| [source-driven-development](skills/source-driven-development/SKILL.md) | 每个框架决策都以官方文档为依据——验证、引用来源、标记未验证的内容 | 你想要任何框架或库的权威、有来源引用的代码时 |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.md) | 对每个非平凡决策进行对抗性新上下文审查——CLAIM → EXTRACT → DOUBT → RECONCILE → STOP，可选用户授权的跨模型升级 | 高风险（生产、安全、不可逆）、处理不熟悉的代码，或者一个自信的输出的验证成本低于后续调试成本时 |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | 组件架构、设计系统、状态管理、响应式设计、WCAG 2.1 AA 无障碍 | 构建或修改面向用户的界面时 |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | 契约优先设计、Hyrum 定律、One-Version 规则、错误语义、边界验证 | 设计 API、模块边界或公共接口时 |

### 验证 - 证明它有效

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Chrome DevTools MCP 用于实时运行时数据——DOM 检查、控制台日志、网络追踪、性能分析 | 构建或调试任何在浏览器中运行的内容时 |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | 五步分诊：复现、定位、简化、修复、防护。停线规则、安全回退 | 测试失败、构建中断或行为异常时 |

### 审查 - 合并前的质量门禁

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | 五轴审查、变更大小（~100 行）、严重性标签（Nit/Optional/FYI）、审查速度规范、拆分策略 | 合并任何变更之前 |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton 栅栏、500 规则、降低复杂度同时保持精确行为 | 代码能运行但比应有的更难阅读或维护时 |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | OWASP Top 10 预防、认证模式、密钥管理、依赖审计、三层边界系统 | 处理用户输入、认证、数据存储或外部集成时 |
| [performance-optimization](skills/performance-optimization/SKILL.md) | 测量优先方法——Core Web Vitals 目标、分析工作流、包分析、反模式检测 | 存在性能要求或你怀疑有性能回退时 |

### 发布 - 放心部署

| 技能 | 作用 | 使用时机 |
|------|------|---------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | 主干开发、原子提交、变更大小（~100 行）、提交即保存点模式 | 进行任何代码变更时（始终适用） |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | 左移、更快更安全、特性标志、质量门禁流水线、失败反馈循环 | 设置或修改构建和部署流水线时 |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | 代码即负债心态、强制性与建议性弃用、迁移模式、僵尸代码清理 | 移除旧系统、迁移用户或下线功能时 |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | 架构决策记录、API 文档、内联文档标准——记录 *为什么* | 做架构决策、更改 API 或发布功能时 |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | 发布前检查清单、特性标志生命周期、分阶段发布、回滚流程、监控设置 | 准备部署到生产环境时 |

---

## 代理角色

预配置的专业角色，用于针对性审查：

| 代理 | 角色 | 视角 |
|------|------|------|
| [code-reviewer](agents/code-reviewer.md) | 高级 Staff 工程师 | 五轴代码审查，"Staff 工程师会批准这个吗？"标准 |
| [test-engineer](agents/test-engineer.md) | QA 专家 | 测试策略、覆盖分析和 Prove-It 模式 |
| [security-auditor](agents/security-auditor.md) | 安全工程师 | 漏洞检测、威胁建模、OWASP 评估 |

---

## 参考检查清单

按需由技能引用的快速参考材料：

| 参考 | 涵盖 |
|------|------|
| [testing-patterns.md](references/testing-patterns.md) | 测试结构、命名、Mock、React/API/E2E 示例、反模式 |
| [security-checklist.md](references/security-checklist.md) | 提交前检查、认证、输入验证、头部、CORS、OWASP Top 10 |
| [performance-checklist.md](references/performance-checklist.md) | Core Web Vitals 目标、前端/后端检查清单、测量命令 |
| [accessibility-checklist.md](references/accessibility-checklist.md) | 键盘导航、屏幕阅读器、视觉设计、ARIA、测试工具 |

---

## 技能如何工作

每个技能遵循一致的结构：

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: Guides agents through [task].│  │
│  │              Use when…                    │  │
│  └───────────────────────────────────────────┘  │
│  Overview         → 此技能的作用                │
│  When to Use      → 触发条件                    │
│  Process          → 逐步工作流                  │
│  Rationalizations → 借口 + 反驳                 │
│  Red Flags        → 出现问题的迹象              │
│  Verification     → 证据要求                    │
└─────────────────────────────────────────────────┘
```

**关键设计选择：**

- **流程，不是散文。** 技能是代理遵循的工作流，不是阅读参考文档。每个都有步骤、检查点和退出标准。
- **反合理化。** 每个技能包含一个表格，列出了代理用来跳过步骤的常见借口（例如"我以后再添加测试"），并附有论证反驳。
- **验证不可妥协。** 每个技能以证据要求结束——测试通过、构建输出、运行时数据。"看起来对"永远不够。
- **渐进式披露。** `SKILL.md` 是入口。辅助参考资料仅在需要时加载，保持 token 使用量最小。

---

## 项目结构

```
agent-skills/
├── skills/                            # 23 个技能（22 个生命周期 + 1 个元技能）
│   ├── interview-me/                  #   定义
│   ├── idea-refine/                   #   定义
│   ├── spec-driven-development/       #   定义
│   ├── planning-and-task-breakdown/   #   计划
│   ├── incremental-implementation/    #   构建
│   ├── context-engineering/           #   构建
│   ├── source-driven-development/     #   构建
│   ├── doubt-driven-development/      #   构建
│   ├── frontend-ui-engineering/       #   构建
│   ├── test-driven-development/       #   构建
│   ├── api-and-interface-design/      #   构建
│   ├── browser-testing-with-devtools/ #   验证
│   ├── debugging-and-error-recovery/  #   验证
│   ├── code-review-and-quality/       #   审查
│   ├── code-simplification/          #   审查
│   ├── security-and-hardening/        #   审查
│   ├── performance-optimization/      #   审查
│   ├── git-workflow-and-versioning/   #   发布
│   ├── ci-cd-and-automation/          #   发布
│   ├── deprecation-and-migration/     #   发布
│   ├── documentation-and-adrs/        #   发布
│   ├── shipping-and-launch/           #   发布
│   └── using-agent-skills/            #   元技能：如何使用本包
├── agents/                            # 3 个专业角色
├── references/                        # 4 个补充检查清单
├── hooks/                             # 会话生命周期钩子
├── .claude/commands/                  # 7 个斜杠命令（Claude Code）
├── .gemini/commands/                  # 7 个斜杠命令（Gemini CLI）
└── docs/                              # 每个工具的设置指南
```

---

## 为什么需要 Agent Skills？

AI 编程助手默认选择最短路径——这通常意味着跳过规格、测试、安全审查和使软件可靠的最佳实践。Agent Skills 为代理提供了结构化的工作流，强制执行高级工程师对生产代码所遵循的相同纪律。

每个技能都编码了来之不易的工程判断：*何时*写规格、*什么*需要测试、*如何*审查，以及*何时*发布。这些不是通用提示——它们是区分生产级工作和原型级工作的有主见、流程驱动的工作流。

技能内置了来自 Google 工程文化的最佳实践——包括来自[《Google 软件工程》](https://abseil.io/resources/swe-book)和 Google[工程实践指南](https://google.github.io/eng-practices/)的概念。你会在 API 设计中找到 Hyrum 定律，在测试中找到 Beyonce 规则和测试金字塔，在代码审查中找到变更大小和审查速度规范，在简化中找到 Chesterton 栅栏，在 Git 工作流中找到主干开发，在 CI/CD 中找到左移和特性标志，以及一个将代码视为负债的专用弃用技能。这些不是抽象原则——它们直接嵌入代理遵循的逐步工作流中。

---

## 贡献

技能应该**具体**（可操作的步骤，不是模糊的建议）、**可验证**（清晰的退出标准和证据要求）、**经过实战检验**（基于真实工作流）、**精简**（仅引导代理所需的内容）。

详见 [docs/skill-anatomy.md](docs/skill-anatomy.md) 的格式规范和 [CONTRIBUTING.md](CONTRIBUTING.md) 的指南。

---

## 许可证

MIT — 在你的项目、团队和工具中自由使用这些技能。
