# Style

## Convention

- AI 生成的代码必须遵循与项目相同的格式化规范

## C# 代码格式化（.editorconfig + omnisharp.json）

```editorconfig
root = false

[*.cs]
csharp_new_line_before_open_brace = false
csharp_new_line_before_else = false
csharp_new_line_before_catch = false
csharp_new_line_before_finally = false
```

```json
// omnisharp.json
{
    "FormattingOptions": {
        "NewLinesForBracesInLambdaExpressionBody": false,
        "NewLinesForBracesInAnonymousMethods": false,
        "NewLinesForBracesInAnonymousTypes": false,
        "NewLinesForBracesInControlBlocks": false,
        "NewLinesForBracesInTypes": false,
        "NewLinesForBracesInMethods": false,
        "NewLinesForBracesInProperties": false,
        "NewLinesForBracesInObjectCollectionArrayInitializers": false,
        "NewLinesForBracesInAccessors": false,
        "NewLineForElse": false,
        "NewLineForCatch": false,
        "NewLineForFinally": false
    }
}
```

### 关键风格说明

- **大括号**：Egyptian style（所有场景左大括号均不换行）
  - 类型、方法、属性、访问器、控制块、Lambda、匿名方法/类型、对象/集合/数组初始化器
- **else / catch / finally**：紧跟上一个 `}` 同行
