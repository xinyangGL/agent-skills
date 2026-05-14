---
description: 增量实现下一个任务 —— 构建、测试、验证、提交
---

同时调用 agent-skills:incremental-implementation 和 agent-skills:test-driven-development 技能。

从计划中挑选下一个待处理的任务。对每个任务：

1. 阅读任务的验收标准
2. 加载相关上下文（现有代码、模式、类型）
3. 为预期行为编写失败的测试（RED）
4. 实现使测试通过的最小代码（GREEN）
5. 运行完整测试套件检查回归
6. 运行构建验证编译
7. 带描述性消息提交
8. 标记任务完成并移动到下一个任务

如果任何步骤失败，遵循 agent-skills:debugging-and-error-recovery 技能。