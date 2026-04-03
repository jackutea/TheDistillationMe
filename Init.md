# Init Spec

## 目标

1. 基于 AI 目录文档，生成两个代理文件：
	- main.agent.md
	- git.agent.md
2. 基于 AI 目录文档, 按需生成代理文件
- 注: 生成的agent模板(仅参考, 不必一样):
```
---
name: Main Agent
description: "Use when handling architecture distillation, convention compliance, milestone planning, Unity module documentation, and overall AI docs maintenance"
model: Claude Opus 4.6
tools: [read, search, edit, todo, agent]
agents: [Git Agent]
user-invocable: true
---
核心职责
```

## 前置问询（必须先执行）
在开始生成前，先向用户确认以下三项：
1. 需要哪些目录作为凭依(不必确认Main.md和Git.md, 因为必须包含), 这一步要给出选项(AI目录下的所有第一级目录)？
2. 生成位置是 .github 还是 .claude ？
3. 是否还要补充agent(例如build.agent,tests.agent)?

若用户未明确回答，禁止进入生成阶段。

## 输入
- 文档输入：用户指定的凭依文档列表
- 目标目录：.github 或 .claude
- 代理清单：main / git

## 处理流程
1. 收集输入
- 读取用户指定的凭依文档。
- 确认目标目录存在；不存在则先创建。

2. 生成代理
- 生成 main.agent.md：负责文档体系统筹（架构、公约、里程碑、模块文档）。
- 生成 git.agent.md：仅处理 Git 任务（提交、分支、合并、日志、冲突）。

3. 校验一致性
- 两个代理的职责边界不重叠。
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
- 两个代理文件已生成。
- 文件内容通过职责与路径校验。
- 已向用户返回结果摘要与可执行下一步。
