# Init Spec

## 目标

1. 基于 AI 目录文档，按需生成代理文件, 但至少包括：
	- main.agent.md
	- git.agent.md
	- architecture.agent.md
	- style.agent.md
- 注: 生成的agent模板(仅参考, 不必一样):
```
---
name: Main Agent
description: "Use when handling architecture distillation, convention compliance, milestone planning, Unity module documentation, and overall AI docs maintenance"
model: Claude Opus 4.6
tools: [read, search, edit, execute, todo, agent]
agents: [Git Agent]
user-invocable: true
---
核心职责
```

## 前置问询（必须先执行）
在开始生成前，先向用户确认以下项：
1. 生成位置是 .github/agents 还是 .claude/agents ？
	- 注: 生成后是放置在用户的工程里, 而非TheDistillationMe工程
2. 是否还要补充agent(例如build.agent,tests.agent)?

若用户未明确回答，禁止进入生成阶段。

## 输入
- 目标目录：.github 或 .claude
- 代理清单：main / git / architecture / style

## 处理流程
1. 收集输入
- 读取用户指定的凭依文档。
- 确认目标目录存在；不存在则先创建。

2. 生成代理
- 每个代理文件需包含：职责描述、使用的模型、工具权限、调用的其他代理、用户可否直接调用。

3. 校验一致性
- 每个代理的职责边界尽量不重叠。
- 工具权限与职责匹配（最小工具原则）。
- 引用路径均可访问。

4. 输出结果
- 返回生成文件路径。
- 返回每个代理的职责摘要。
- 返回建议试用指令（每个代理至少 1 条）。

## 约束
- 使用中文输出。
- 信息简洁，采用大纲式表达。
- 仅在用户确认后生成文件。

## 失败回退
- 若凭依文档缺失：先报告缺失清单，再请求补充。
- 若目标目录不可用：建议切换到另一个目录（.github 或 .claude）。
- 若职责冲突：优先收敛为单一职责，再重新生成。

## 完成定义（DoD）
- 前置问询已完成并有明确答案。
- 所有代理文件已生成。
- 文件内容通过职责与路径校验。
- 已向用户返回结果摘要与可执行下一步。
