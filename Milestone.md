# Milestone

## M1 - 基础框架搭建 ✅

- [x] 建立 Human/AI 双目录结构
- [x] 制定公约（Convention）
- [x] 创建AI主入口（Main）
- [x] 蒸馏架构规范（Architecture）

## M2 - Gist.md ✅

- [x] 目录结构模板（Launcher / HotReload / Common 分层）
- [x] 程序集划分（五层职责与边界）
- [x] Context 模板（字段按类型分组）
- [x] 主循环模板（Init → ProcessInput → Tick → TearDown）
- [x] System 模板（静态类 + SystemState + SystemEvents 三件套）
- [x] Controller 模板（Spawn / Unspawn / Tick 签名规范）
- [x] Entity + Component 模板（组合式数据容器）
- [x] Repository 模板（Add / TryGet / Remove + 多索引示例）
- [x] Module vs Manager 对比（边界说明）
- [x] Events 模板（Action-based pubsub 框架）
- [x] SO 模板（配置模板与 Entity 关系）
- [x] EM 模板（编辑器工具类）

## M3 - Unity模块文档蒸馏

- [ ] InputModule — 统一输入处理
- [ ] AssetsModule — Addressable资源加载
- [ ] AudioManager — 音频管理与音频池
- [ ] VFXManager — 特效管理与特效池
- [ ] FileManager — 本地文件读写与持久化
- [ ] OSSManager — 云对象存储服务
- [ ] ObjectPool — 对象池模式
- [ ] L10NModule — 本地化

## M4 - 领域知识文档

- [ ] 数学 — 向量、矩阵、四元数、插值
- [ ] 图形学 — 渲染管线、Shader基础、URP
- [ ] 设计模式 — 状态机、观察者、对象池等
- [ ] 数据结构 — 空间分区、优先队列等

## M5 - 架构文档完善

- [ ] 理念补充 — 数据内容格式示例（.prefab, .json, .csv, .asset, .txt, .png, .fbx）；"控制"即"让谁来"、"行为"即"做什么"
- [ ] Controller职责细节 — Spawn/Unspawn/Tick
- [ ] Module常用库推荐 — Addressables / InputSystem / Telepathy / LiteNetLib / Steamworks.NET
- [ ] Functions具体命名 — GFEasing / Newtonsoft.JSON / GFEncoder

## M6 - 仓库说明文档 ✅

- [x] 根目录新增 README.md，说明“仓库是什么”与“如何使用”
- [x] README.md 补充 .github 目录职责与使用入口说明
- [x] Init.md 扩展为 AI 可读的结构化初始化规范
