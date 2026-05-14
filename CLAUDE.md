# agent-skills

这是 agent-skills 项目——面向 AI 编程助手的生产级工程技能集合。

## 项目结构

```
skills/       → 核心技能（每个目录一个 SKILL.md）
agents/       → 可复用的代理角色（code-reviewer、test-engineer、security-auditor）
hooks/        → 会话生命周期钩子
.claude/commands/ → 斜杠命令（/spec、/plan、/build、/test、/review、/code-simplify、/ship）
references/   → 补充检查清单（测试、性能、安全、无障碍）
docs/         → 不同工具的设置指南
```

## 按阶段分类的技能

**定义：** interview-me、idea-refine、spec-driven-development
**计划：** planning-and-task-breakdown
**构建：** incremental-implementation、test-driven-development、context-engineering、source-driven-development、doubt-driven-development、frontend-ui-engineering、api-and-interface-design
**验证：** browser-testing-with-devtools、debugging-and-error-recovery
**审查：** code-review-and-quality、code-simplification、security-and-hardening、performance-optimization
**发布：** git-workflow-and-versioning、ci-cd-and-automation、deprecation-and-migration、documentation-and-adrs、shipping-and-launch

## 约定

- 每个技能位于 `skills/<name>/SKILL.md`
- 带有 `name` 和 `description` 字段的 YAML frontmatter
- Description 以技能的用途开头（第三人称），后跟触发条件（"Use when..."）
- 每个技能包含：Overview、When to Use、Process、Common Rationalizations、Red Flags、Verification
- 参考资料在 `references/` 中，不在技能目录内
- 仅当内容超过 100 行时才创建辅助文件

## 命令

- `npm test` — 不适用（这是一个文档项目）
- 验证：检查所有 SKILL.md 文件是否具有有效的包含 name 和 description 的 YAML frontmatter

## 边界

- 始终：遵循 skill-anatomy.md 格式创建新技能
- 切勿：添加模糊的建议而非可执行流程的技能
- 切勿：在技能之间重复内容——改为引用其他技能
