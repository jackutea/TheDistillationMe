# Git 规范

## 格式

```
<type> action: description
```

## 提交类型（type）

| 类型 | 含义 |
|------|------|
| `<feature>` | 程序功能 |
| `<refactor>` | 重构 |
| `<perf>` | 性能优化 |
| `<asset>` | 资源文件 |
| `<doc>` | 文档 |
| `<version>` | 版本号 |
| `<ai>` | AI 相关 |

## 变更动作（action）

`add` / `fix` / `modify` / `remove` / `upgrade`（可用中文）

## 描述

简洁明了，无约束。

## 示例

```
<asset> add: 柜子1-1
<asset> modify: 柜子1-1参数
<feature> add: 拖拽消除
<feature> fix: 拖拽消除的bug
<perf> add: 优化拖拽消除的性能
<refactor> modify: 提取公共输入处理逻辑
<ai> add: Git提交规范文档
```

## 指令

1. **底层指令优先**：必须使用真实具体的 `git` 命令行指令（如 `git status`、`git diff` 等）获取变更文件等状态，而不能依赖大模型自身的检索或者猜测工具。
