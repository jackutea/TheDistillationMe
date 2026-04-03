# L10N Module

命名空间: `NJM`  
路径: `Assets/Src_Runtime/HotReload/Manager_L10N/`

---

## 职责

运行时多语言文本/资源本地化管理。支持文字、图片、音频、Prefab 的多语言切换，并对未覆盖语言自动 fallback 到英文。

---

## 类结构

```
L10NSO (ScriptableObject)
    └── L10NOneLangTC[] langs        // 每种语言的资源集合

L10NManager
    ├── L10NRepository               // 按语言索引的顶层存储
    │   └── Dictionary<SystemLanguage, L10NEntity>
    └── L10NEntity                   // 单语言下的所有词条
        └── Dictionary<EntityTypeID, L10NOneLangTC>
```

### L10NSO
- `entityTypeID: EntityTypeID` — 词条唯一标识（由 `EntityType` + `TypeID` 组成）
- `desc: string` — 编辑器备注
- `langs: L10NOneLangTC[]` — 各语言内容
- `AddToAA()` (Editor) — 自动重命名资产并注册到 Addressables `L10N` 组

### L10NOneLangTC
```csharp
struct L10NOneLangTC {
    SystemLanguage lang;
    List<string>      txts;
    List<Sprite>      images;
    List<AudioClip>   audioClips;
    List<GameObject>  prefabs;
}
```
一个词条支持同时携带多种类型资源（列表形式，按约定下标取用）。

### L10NOneLangFlag
用于标记词条包含哪些资源类型（Flags 枚举），目前定义：
- `Txt_Title / Txt_Description / Txt_Detail`
- `Image`

### L10NManager
| 方法 | 说明 |
|---|---|
| `AddAll(ICollection<L10NSO>)` | 批量载入 SO，写入 Repository |
| `Language_Set(SystemLanguage)` | 切换当前语言 |
| `TryGet(L10NSO, out L10NOneLangTC)` | 按 SO 查询当前语言词条 |
| `TryGet(EntityTypeID, out L10NOneLangTC)` | 按 ID 查询，先查当前语言，失败则 fallback 到英文 |

**语言检测**（构造函数）：
- 默认 `ChineseSimplified`
- 微信小游戏平台（`UNITY_WXGAME && !UNITY_EDITOR`）：读取 `WX.GetSystemInfoSync().language`，仅映射 `zh_CN → ChineseSimplified`，其余 → `lang_fallback (English)`

---

## 数据流

```
AssetsModule.L10N_GetAll()
    ↓
L10NManager.AddAll()          // 初始化阶段（ClientMain.cs）
    ↓
各 Panel / Controller 调用
l10NManager.TryGet(so, out l10n)
    ↓
l10n.txts[0] / l10n.images[0] ...  // 按下标取资源
```

---

## EntityTypeID

```csharp
struct EntityTypeID {
    EntityType entityType;   // FieldOffset(0)
    TypeID     typeID;       // FieldOffset(4)
}
```
作为词条 key，用 `StructLayout(LayoutKind.Explicit)` 保证内存布局。

---

## 使用规范

- 所有词条在编辑器中通过 `L10NSO.AddToAA()` 统一注册到 AA `L10N` 组
- 调用方只持 `L10NSO` 引用或 `EntityTypeID`，不直接操作 Repository/Entity
- 资源列表下标含义由业务侧约定（模块本身不限制）
