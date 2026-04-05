# Gist

> `Architecture.md` 的实现伴侣。将架构原则转化为可直接参照的目录结构与代码框架。  
> `{Feature}` / `{Entity}` / `{F}` 均为占位符，按实际业务替换。

---

## 目录结构模板

```
Assets/
└── Src_Runtime/
    ├── Launcher/                    # 启动层：AOT编译、引导启动、下载热更DLL
    │   └── ClientLauncher.cs
    ├── HotReload/                   # 热更层：核心游戏逻辑，运行时可热更新
    │   ├── ClientMain.cs            # 热更主入口
    │   ├── ClientHotReload.cs       # 热更触发器
    │   ├── GameContext.cs           # 唯一上下文
    │   ├── Systems_{Feature}/       # 每个系统一个文件夹
    │   │   ├── {F}System.cs
    │   │   ├── {F}SystemState.cs
    │   │   └── {F}SystemEvents.cs
    │   ├── Controllers/             # 跨系统通用控制器
    │   ├── Manager_{Feature}/       # 业务服务（高层）
    │   ├── Modules_{Feature}/       # 基础设施能力（低层）
    │   ├── Entities/                # 实体与组件定义
    │   │   └── {Entity}/
    │   │       ├── {Entity}Entity.cs
    │   │       └── {Entity}{Aspect}Component.cs
    │   └── Common/                  # 热更层内部共享工具
    └── Common/                      # 公共层：跨层共享类型，不含业务逻辑
        ├── DefineConst.cs
        └── Env.cs
```

---

## 程序集划分

| 程序集 | 职责 | 依赖方向 |
|---|---|---|
| **Launcher** | AOT引导；下载并执行热更DLL；初始化Addressables目录 | → Common |
| **HotReload** | 核心游戏逻辑；所有System/Controller/Controller/Manager/Module | → Common |
| **Common** | 枚举、常量、接口、工具函数；不含业务逻辑 | — |
| **Editor** | 编辑器工具；仅在`UNITY_EDITOR`下编译 | → Common |
| **Tests** | 单元测试与集成测试 | → Common, HotReload |

> 规则：Launcher 与 HotReload 之间不直接引用，通过 Addressables 动态加载 ClientMain 解耦。

---

## Context 模板

```csharp
public class GameContext {
    // ── SystemState（各系统状态机） ────────────────────────────────
    public {F}SystemState       state_{F};

    // ── SystemEvents（各系统事件总线） ────────────────────────────
    public {F}SystemEvents      events_{F};

    // ── Module（基础设施，低层） ──────────────────────────────────
    public AssetsModule         assetsModule;
    public InputModule          inputModule;

    // ── Manager（业务服务，高层） ─────────────────────────────────
    public AudioManager         audioManager;
    public VFXManager           vfxManager;
    public L10NManager          l10NManager;
    public FileManager          fileManager;

    // ── Repository（实体仓库） ────────────────────────────────────
    // ⚠ Repository 放在 Context，不放在 SystemState
    public {Entity}Repository   {entity}Repository;

    // ── Service（业务服务实例） ───────────────────────────────────
    // ⚠ Service 放在 Context，不放在 SystemState
    public IDService            idService;
    public PoolService          poolService;

    // ── Entity（全局唯一实体） ────────────────────────────────────
    // ⚠ Entity 放在 Context，不放在 SystemState
    public {Entity}Entity       {entity}Entity;
}
```

> 字段命名规则：`state_*` / `events_*` / `*Module` / `*Manager` / `*Repository` / `*Entity`。

---

## 主循环模板

```csharp
// ClientMain.cs（热更主入口，MonoBehaviour）
public class ClientMain : MonoBehaviour {

    GameContext ctx;

    void Awake()  => Init();
    void Update() { ProcessInput(); Tick(); }
    void FixedUpdate() => FixTick();
    void LateUpdate()  => LateTick();
    void OnDestroy()   => TearDown();

    void Init() {
        ctx = new GameContext();
        // 1. 创建 Module / Manager / Repository
        ctx.assetsModule = new AssetsModule();
        ctx.audioManager = new AudioManager();
        // 2. 处理实例间依赖
        // 3. 加载资源
        // 4. 注册事件（SystemEvents.OnXxx += ...）
        // 5. 进入首个系统
        {F}System.Init(ctx);
    }

    void ProcessInput() {
        ctx.inputModule.Process(Time.deltaTime);
        // 网络通信、系统事件等
    }

    void Tick()     => {F}System.Tick(ctx, Time.deltaTime);
    void FixTick()  => {F}System.FixTick(ctx, Time.fixedDeltaTime);
    void LateTick() => {F}System.LateTick(ctx, Time.deltaTime);

    void TearDown() {
        // 卸载资源、注销事件、销毁上下文
    }
}
```

---

## System 模板

三件套：**静态逻辑类** + **状态类** + **事件类**。

```csharp
// {F}System.cs — 静态，系统级编排
public static class {F}System {

    public static void Init(GameContext ctx) { ... }

    public static void Tick(GameContext ctx, float dt) {
        ProcessInput(ctx, dt);
        PreTick(ctx, dt);
    }

    public static void FixTick(GameContext ctx, float fixdt) { ... }

    public static void LateTick(GameContext ctx, float dt) { ... }

    static void ProcessInput(GameContext ctx, float dt) {
        // 读取 inputModule，调用相应 Controller
    }

    static void PreTick(GameContext ctx, float dt) {
        // 遍历 Repository，逐帧 Tick Controller
        int count = ctx.{entity}Repository.TakeAll(out var entities);
        for (int i = 0; i < count; i++)
            {Entity}Controller.Tick(ctx, entities[i], dt);
    }
}

// {F}SystemState.cs — 有状态，存储系统级 FSM 状态
// ⚠ 不放 Repository / Service / Entity / Module / Manager 等共享资源，这些归 Context
public class {F}SystemState {
    public {F}Phase phase;
    // 其他系统级临时状态（不属于 Entity 的数据）
}

public enum {F}Phase { Idle, Playing, Pausing, Over }

// {F}SystemEvents.cs — 有状态，Action-based 事件总线
public class {F}SystemEvents {
    public Action<{Entity}Entity> OnSpawn;
    public Action<{Entity}Entity> OnUnspawn;
    // 逐事件定义，避免通用消息总线
}
```

---

## Controller 模板

```csharp
// {Entity}Controller.cs — 静态无状态，负责控制逻辑
public static class {Entity}Controller {

    /// <summary>创建实体，分配 ID，存入 Repository，初始化数据，注册事件</summary>
    public static {Entity}Entity Spawn(GameContext ctx, {Entity}SO so) {
        var entity = new {Entity}Entity();
        entity.uniqueID = ctx.idService.Next();
        entity.typeID   = so.typeID;
        // 从配置（SO/json）读取内容，写入 entity 字段
        entity.hp = so.hp;
        ctx.{entity}Repository.Add(entity);
        ctx.events_{F}.OnSpawn?.Invoke(entity);
        return entity;
    }

    /// <summary>销毁实体（或放回对象池），注销事件</summary>
    public static void Unspawn(GameContext ctx, {Entity}Entity entity) {
        ctx.events_{F}.OnUnspawn?.Invoke(entity);
        ctx.{entity}Repository.Remove(entity);
        // entity.Reset() 后归还对象池
    }

    /// <summary>每帧更新</summary>
    public static void Tick(GameContext ctx, {Entity}Entity entity, float dt) {
        // 驱动 Controller 或直接修改 entity 字段
        {Entity}Controller.Move(ctx, entity, dt);
    }
}
```

> Controller 只做**控制**（"让谁来"）。行为算法细节（"做什么"）：简单时直接在 Controller 内新增函数处理；复杂时才抽出 Controller 承载。

---

## Entity + Component 模板

```csharp
// {Entity}Entity.cs — 核心数据载体，字段用 Component 封装
public class {Entity}Entity {
    // 必有字段
    public int            uniqueID;   // 实例间不重复
    public TypeID         typeID;     // 同类相同

    // 非单机时额外添加（可选）
    // public {Entity}Logic  logic;
    // public {Entity}Render render;

    // 业务字段（用 Component 封装相关字段）
    public {Entity}MovementComponent  movement;
    public {Entity}CombatComponent    combat;
    public {Entity}ViewComponent      view;
}

// {Entity}MovementComponent.cs — Component 是 Entity 字段的封装
public class {Entity}MovementComponent {
    public Vector3 position;
    public Vector3 velocity;
    public float   speed;
}
```

**规则**
- Component 用 `class`（纯数据，无行为）
- Entity 不继承其他 Entity，用组合替代继承
- 代码不在 Entity 内定义数据内容（初始值来自配置文件）

---

## SO 模板

SO（ScriptableObject）是 Entity 的配置模板，存储不可变的静态数据内容。

```csharp
// {Entity}SO.cs — 配置模板，不含运行时状态
[CreateAssetMenu(fileName = "So_{Entity}_", menuName = "NJM/{Entity}SO")]
public class {Entity}SO : ScriptableObject {
    public TypeID    typeID;    // 类型标识，与同类 Entity.typeID 对应
    // 配置字段（从此读取，写入 Entity）
    public int       hp;
    public float     speed;
    public Sprite    spr;       // 可引用其他资产
    public {Other}SO sub;       // 可嵌套引用其他 SO（形成层级配置）
}
```

**规则**
- SO 存放于 `TM/` 子目录（Template）
- 文件命名：`So_{Entity}_{Name}.asset`
- SO 只读，不含运行时可变状态
- `Controller.Spawn` 以 SO 为参数，从 SO 读取内容初始化 Entity
- 可实现 `AddToAA()` 编辑器方法，将资产自动注册到 Addressables 组

**关系：SO → Entity**
```
AssetsModule 加载 {Entity}SO
        ↓
Controller.Spawn(ctx, so)
        ↓
Entity 创建 + 字段从 SO 赋值 + 存入 Repository
```

---

## EM 模板

EM（Editor Model）是编辑器工具类，职责是以对人友好的方式编辑 SO，仅在编辑时运行，不参与 Runtime 构建。

```csharp
// {Controller}EM.cs — Editor Model（位于 Src_Editor/，不进入 Runtime 构建）
[ExecuteInEditMode]
public class {Controller}EM : MonoBehaviour {

    [SerializeField] {Controller}SO so;   // 目标 SO

    // 用 SO 直接引用替代抽象的 TypeID，方便编辑器拖拽赋值
    public {Other}SO reference;

    [Button]                          // Odin Inspector 触发
    void WriteTo() {
        so.referenceTypeID = reference.typeID;   // 转换为 TypeID 写回 SO
        EditorUtility.SetDirty(so);
        AssetDatabase.SaveAssets();
    }
}
```

**规则**
- 位于 `Src_Editor/` 目录，不参与 Runtime 构建
- `WriteTo()` 将 EM 上的可视数据转换写回目标 SO
- EM 不持有运行时数据，只做数据转换工具
- 使用 `[Button]`（Odin Inspector）触发转换

---

## Repository 模板

```csharp
// {Entity}Repository.cs
public class {Entity}Repository {

    // 主索引
    readonly Dictionary<int, {Entity}Entity> byID = new();

    // 附加索引（按查询需求添加）
    readonly Dictionary<TypeID, List<{Entity}Entity>> byTypeID = new();

    public void Add({Entity}Entity entity) {
        byID[entity.uniqueID] = entity;

        if (!byTypeID.TryGetValue(entity.typeID, out var list)) {
            list = new List<{Entity}Entity>();
            byTypeID[entity.typeID] = list;
        }
        list.Add(entity);
    }

    public bool TryGet(int uniqueID, out {Entity}Entity entity)
        => byID.TryGetValue(uniqueID, out entity);

    public bool TryGetByType(TypeID typeID, out List<{Entity}Entity> entities)
        => byTypeID.TryGetValue(typeID, out entities);

    public int TakeAll(out {Entity}Entity[] result) {
        // 推荐：写入预分配数组，避免 GC
        ...
    }

    public void Remove({Entity}Entity entity) {
        byID.Remove(entity.uniqueID);
        if (byTypeID.TryGetValue(entity.typeID, out var list))
            list.Remove(entity);
    }
}
```

**接口约定**：`Add` / `TryGet` / `TryGetByXxx` / `TakeAll` / `Remove`。附加索引按实际查询需求添加，勿过早优化。

---

## Module vs Manager

| 维度 | Module（基础设施） | Manager（业务服务） |
|---|---|---|
| **层级** | 低层 | 高层 |
| **职责** | 封装平台/引擎能力（加载资源、采集输入、网络 I/O）| 提供业务功能（音效、特效、本地化、文件读写）|
| **依赖** | 不依赖 Manager | 必要时可依赖 Module |
| **常见库** | Addressables / InputSystem / Telepathy / LiteNetLib | 自行实现（参考 AudioManager、L10NManager）|
| **示例** | `AssetsModule` / `InputModule` / `NetworkModule` | `AudioManager` / `VFXManager` / `L10NManager` / `FileManager` |

> 判断依据：若能被多个业务领域复用且不含游戏业务规则 → Module；若封装了特定业务逻辑 → Manager。

---

## Events 模板

```csharp
// {F}SystemEvents.cs
public class {F}SystemEvents {

    // 每个事件单独声明，语义清晰
    public event Action<{Entity}Entity>    OnSpawn;
    public event Action<{Entity}Entity>    OnUnspawn;
    public event Action<int /*score*/>     OnScoreChanged;

    // 可选：提供触发方法，统一调用入口
    internal void Spawn({Entity}Entity e)     => OnSpawn?.Invoke(e);
    internal void Unspawn({Entity}Entity e)   => OnUnspawn?.Invoke(e);
    internal void ScoreChanged(int score)     => OnScoreChanged?.Invoke(score);
}

// 订阅（在 Init 阶段）
ctx.events_{F}.OnSpawn += OnEntitySpawned;

// 注销（在 TearDown 阶段）
ctx.events_{F}.OnSpawn -= OnEntitySpawned;
```

**规则**
- 禁止通用消息总线（`Dispatch(string eventName, object data)`）；每条事件独立声明，保持类型安全
- 订阅在 `Init`，注销在 `TearDown`，成对出现
- 跨系统通信通过 Context 中各 `SystemEvents` 显式引用，不走全局静态
