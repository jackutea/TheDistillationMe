# Unity UGUI

## 能力

- 创建 UGUI Prefab（`.prefab` + `.meta`）

## Prefab 通用模式（从 WordsLand 学习）

### 命名规范

| 前缀 | 含义 | 示例 |
|------|------|------|
| `Panel_` | 顶级面板（含 Canvas） | Panel_Bag, Panel_Building, Panel_Login |
| `Widget_` | 可复用子组件 | Widget_StuffOnBag, Widget_Stuff_Option |
| `Img_` | Image 元素 | Img_BG, Img_Border, Img_HR, Img_Icon, Img_Selection |
| `Txt_` | TMP 文本 | Txt_Count, Txt_Name |
| `Btn_` / `Button` | 按钮 | Btn_+, Button |
| `Manager_` | 管理器节点（挂脚本） | Manager_Fuel |

### Panel 根节点标准结构

```
Panel_XXX  (Layer 5)
├── Canvas           (RenderMode: 0 ScreenSpace-Overlay)
├── CanvasScaler     (ScaleMode: 1, Reference: 1920x1080)
├── GraphicRaycaster
└── 自定义脚本        (持有 elePrefab + root 容器引用)
```

- `m_LocalScale: {x: 0, y: 0, z: 0}` — 初始隐藏，代码控制显示

### 通用 UI 元素

**背景 (Img_BG)**：Stretch 铺满，深色 `#222222`，m_Type: 0/1

**边框 (Img_Border)**：Stretch 铺满 + 偏移制造边框效果，浅色 `#EDEDED`

**分隔线 (Img_HR)**：锚定父节点一侧，固定宽度 4px，浅色

**文本 (TextMeshProUGUI)**：m_isOrthographic: 1，m_fontStyle: 1 (Bold)，常用字号 24/30/36

**按钮 (Button)**：透明 Image (alpha=0) 作 RaycastTarget + Button 组件，ColorTint 过渡

**选中态 (Img_Selection)**：Stretch 铺满，黑色，通过 SetActive 切换

### 布局组

- `HorizontalLayoutGroup`：物品横排（Spacing 10，ChildForceExpand: false）
- `VerticalLayoutGroup`：纵向排列
- `GridLayoutGroup`：选项网格（CellSize 220x60，Constraint FixedColumnCount=2）
- `ContentSizeFitter`：常与 LayoutGroup 配对，HorizontalFit/VerticalFit: 2 (PreferredSize)
- `LayoutElement`：主要用于 ignoreLayout，使子元素脱离父 LayoutGroup 的自动排列

### 自定义脚本绑定模式

```
面板脚本字段：elePrefab (Widget prefab 引用) + root_xxx (容器 RectTransform)
Widget 脚本字段：uniqueID + txt_xxx + btn + img_xxx
```

---

## 具体面板实现

### Panel_Bag（背包栏）

```
Panel_Bag (Canvas 根)
└── WindowGroup (底部锚定, h=100)
    ├── Img_Border (Stretch, 浅色边框)
    ├── Img_BG (Stretch, 深色背景)
    ├── TitleGroup (左侧 w=50, Image 背景)
    │   ├── Text(TMP) "背包" (竖排标题)
    │   └── Img_HR (右边缘竖线 4px)
    └── StuffGroup (HorizontalLayoutGroup, PaddingL=64, Spacing=10)
        └── [Widget_StuffOnBag 动态实例化]
```

- 锚定：AnchorMin(0,0) AnchorMax(1,0)，SizeDelta h=100（屏幕底部横条）

### Widget_StuffOnBag（背包物品格子）

```
Widget_StuffOnBag (90x50)
├── Img_BG (Stretch, padding 8)
├── Txt_Count (底部居中, "x5", fontSize=30)
├── Img_Icon (Stretch, 物品图标)
│   └── Txt_Name ("苹果", autoSize, fontSize=34)
└── Button (Stretch, 透明覆盖层)
```

### Panel_Building（建筑面板）

```
Panel_Building (Canvas 根, 1920x1080 居中)
└── WindowGroup
    ├── Manager_Fuel (HLayoutGroup + ContentSizeFitter + FuelManager 脚本)
    │   └── [Widget_Stuff_Fuel 动态实例化, 含 Btn_+ 添加按钮]
    └── Manager_Option (GridLayoutGroup, CellSize=220x60, Cols=2)
        └── [Widget_Stuff_Option 动态实例化]
```

- Panel_Building 脚本持有：bd、fuelManager、optionManager、outputManager

### Widget_Stuff_Option（建筑选项格子）

```
Widget_Stuff_Option (pivot 左上)
├── IconGroup (140x58, Image 深色背景)
│   ├── Img_BG (Stretch)
│   ├── Img_Selection (Stretch, 黑色选中态)
│   └── ImgBtn_Icon (Stretch -8 padding, Image + Button)
│       └── Txt_Name ("[无]", fontSize=36)
├── Txt_Count ("999", fontSize=30)
└── Button (Stretch, 透明覆盖层)
```

### Panel_Login（登录面板）

```
Panel_Login (Canvas 根)
├── Panel (Stretch, 内置 Sprite Sliced, alpha=0.392 半透明)
└── Text(TMP) "The Paper Plain" (左下角, fontSize=24)
```

- 最简面板：半透明背景 + 标题文本
