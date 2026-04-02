# RIP

---

## 一、身份

杰克有茶。游戏创作者，Spartan 式程序员，坚持 KISS 原则。
主要使用 Unity 引擎，主语言 C#。

### 上线项目
| 项目 | 类型 | 玩法 | 平台 |
|------|------|------|------|
| 忍者明 | 2D 平台动作 | 用双闪现闯关 | Steam+NS |
| 忍者明：悟 | 2D 动作解谜 | 用双闪现解谜 | Steam |
| 负情绪消除 | 休闲消除 | 分类收纳 | 微信小游戏+移动端 |

---

## 二、架构哲学

### 分层架构（不可违反的依赖方向）

```
Main → System → Controller → Context → Entity
```

- **Main**：启动引导（ClientMain / ClientLauncher）
- **System**：静态类，业务编排（LogoSystem、GameSystem、HomeSystem）
- **Controller**：静态类，无状态逻辑（InputController、RoleController_FSM）
- **Context**：全局唯一的状态容器，持有所有 SystemState、Module、Entity、Repository 等
- **Entity**：数据载体（RoleEntity、PropEntity）

依赖方向只能从高层指向低层。低层需要调用高层时，有两种方式：
1. **Interface** — 低层定义接口，高层实现
2. **委托（函数指针）** — 低层持有委托，高层注入回调

### 核心设计原则

1. **显式 Context 传递** — 所有状态通过 `ctx` 参数注入，拒绝隐式全局单例
2. **组合优于继承** — Entity 通过组合 Component 扩展能力，无继承层级
3. **静态 Controller** — 逻辑在无状态静态类中，函数式 + 过程式混合
4. **手动生命周期** — `Ctor()` → `Init()` → `TearDown()`，不依赖 Unity 的 Awake/Start
5. **Tick 分层** — `PreTick → FixTick → LateTick`，手动控制步进与物理模拟

### System-State-Events 三件套

每个 System 标配：

| 组件 | 职责 | 示例 |
|------|------|------|
| `*System` | 静态类，业务编排入口 | `GameSystem.Tick(Context ctx, float dt)` |
| `*SystemState` | 存放在 Context 中的系统状态 | `ctx.gameSystemState` |
| `*SystemEvents` | 事件分发 | `GameSystemEvents` |

### Repository 模式

- Dictionary 存储，UniqueID 做 Key
- 预分配 `TempArray[]` 避免迭代 GC
- 统一接口：`TryGet(id, out entity)` / `TakeAll(out array)` 返回 count + 数组引用

### UniqueID / TypeID 双轨标识

```csharp
[StructLayout(LayoutKind.Explicit)]
public struct UniqueID {
    [FieldOffset(0)] public ulong value;
    [FieldOffset(4)] public EntityType entityType;
    [FieldOffset(0)] public int entityID;
}
```

- **UniqueID** — 运行时实例标识
- **TypeID** — 模板/配置标识

---

## 三、编码风格

### 花括号

K&R 风格 — 左花括号**不换行**（`.editorconfig` 强制）：

```csharp
if (status != SystemStatus.Running) {
    return;
}
```

### 命名约定

| 类别 | 规则 | 示例 |
|------|------|------|
| 命名空间 | 项目缩写 | `NJM`、`NJM.Systems_Game` |
| 类名 | PascalCase | `RoleEntity`、`PanelController_Home` |
| 方法 | PascalCase，下划线分层 | `TF_Pos_Get()`、`Anim_Play_Lumber()` |
| 常量 | UPPER_SNAKE_CASE | `GOOD_KILL_EASE_DURATION` |
| 字段 | camelCase | `fixRestSec`、`barriersExcludeEdge` |
| 事件委托 | `On{X}Handle` + `On{X}Invoke` | `OnHeartAddHandle` / `OnHeartAddInvoke()` |
| 协程方法 | `IE` 后缀 | `OpenIE()` |
| Try 模式 | `Try` 前缀，out 参数 | `TryGet()`、`TryInteract()` |

### 代码组织

- `#region` 分区管理大文件
- 禁用大量 IDE 警告（IDE0001、IDE0044、RCS1018 等），追求灵活而非教条
- C# LangVersion 9.0

---

## 四、目录组织范式

```
Assets/
├── Src_Runtime/          # 运行时代码（或 Script_Runtime/）
│   ├── Systems_*/        # 按业务系统划分
│   ├── Entities/         # 实体定义
│   ├── Controllers/      # 静态控制器
│   ├── Domains/          # 领域逻辑
│   ├── Modules_*/        # 功能模块（Input、Asset、Camera、VFX、L10N）
│   ├── Panels/           # UI 面板
│   ├── Factory/          # 工厂
│   └── Generic/          # 工具、扩展、枚举、日志
├── Src_Editor/           # 编辑器代码
├── Res_Runtime/          # 运行时资源
├── Res_Editor/           # 编辑器资源
└── 数字前缀目录/         # 如 0.Doc/、1.编辑器/、2.美术原始资源/
```

### 程序集分离

例如：

```
Launcher（启动/引导）
HotReload（全部热更业务）
Editor（编辑器工具）
```

---

## 五、技术栈

### 必选项

| 技术 | 用途 |
|------|------|
| URP | 渲染管线（每个项目） |
| Addressables | 资源管理与分块加载 |
| New Input System | 输入处理 |
| TextMeshPro | 文本渲染 |

### 自研库

| 库 | 能力 |
|----|------|
| GameFunctions | AStar、RVO2、Grid、数学扩展、Easing |
| GameTK | 输入轴模型、游戏工具 |
| Oxy | WordsLand 主框架 |
| NJM | Wu2 主框架 |

### 常用第三方

| 库 | 用途 |
|----|------|
| Odin Inspector | 编辑器增强 |
| LeTai.TrueShadow | 高质量阴影 |
| HybridCLR | 热更新（TripleGoods） |
| Steamworks.NET | Steam 集成 |

---

## 六、核心偏好

1. **反 OOP** — 拒绝深层继承，偏好静态函数 + 数据容器
2. **GC 敏感** — TempArray 预分配、out 参数、避免 LINQ 热路径
3. **手动控制** — 自己写 Tick 循环、手动 `Physics2D.Simulate(dt)`，不交给 Unity 自动调度
4. **架构先行** — 每个项目写 Architecture.txt，层级规则先于代码
5. **知识沉淀** — AI_Skills 文档记录规范纠正，TheDistillationMe 做跨项目蒸馏
6. **务实主义** — 禁用不需要的 IDE 规则，中文混用无心理负担，只在乎实际效果
