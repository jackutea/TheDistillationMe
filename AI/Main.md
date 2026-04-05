# Main

## 职责

- 确保基于公约进行人机交互
- 调用 milestone.agent
- 若 milestone.agent 不存在，则创建 milestone.agent
    - 创建时要问询: 项目是做什么的, 基于哪些模型
- 完成TODO时, 调用 git.agent

## 基础

- [Git](Git.md)
- [架构规范](/AI/Architecture/Architecture.md)
- [架构实现（Gist）](/AI/Architecture/Architecture_Gist.md)

## 公约

### 沟通

- 使用中文交流
- 信息传达保持简洁，采用大纲式表达
- 文档中禁止出现具体项目名称

### 纠错

- 被纠正时，若相关文档存在对应内容，须同步更新文档

### 任务管理

- 每个新Session默认为Plan模式（先规划再执行）
- 任务分为两个级别：
	- **Milestone**：包含一批TODO，由AI自主拆分
	- **TODO**：具体的一件事
- 所有任务（无论级别）均须记录到Milestone文档，便于回顾与总结

### 输入法

- 用户使用五笔输入法，遇到疑似错别字时，尝试用五笔编码猜测正确的字

### 编程偏好

- 偏好C语言风格，希望编程语言基于C语言扩展：
	- 不要.h和.c分离
	- 增加namespace功能
	- 支持namespace.f(o)
	- 支持o.f()
	- 支持操作符重载

### 文档归属

- AI生成的文档统一归类为AI文档

## Implement

- [模块](Implement/Modules/)
