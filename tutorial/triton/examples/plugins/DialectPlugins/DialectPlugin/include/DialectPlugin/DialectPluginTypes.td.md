# DialectPluginTypes.td — Code Analysis / 代码分析

## Source / 来源
- **Path:** `examples/plugins/DialectPlugins/DialectPlugin/include/DialectPlugin/DialectPluginTypes.td`
- **Purpose:** Defines custom plugin dialect types in TableGen and describes their syntax and parameters.
- **用途：** 在 TableGen 中定义插件方言自定义类型，并描述其语法和参数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: TableGen include guard
**EN:** The guard keeps the type definition file reusable across other `.td` includes.

**CN:** 保护宏使该类型定义文件能够被其他 `.td` 文件安全复用。

### Lines 4-5: Base includes
**EN:** `AttrTypeBase.td` provides the TableGen machinery for attributes and types, while `DialectPluginDialect.td` provides the owning dialect record needed by `TypeDef`.

**CN:** `AttrTypeBase.td` 提供属性和类型的 TableGen 基础设施，而 `DialectPluginDialect.td` 则提供 `TypeDef` 所需的所属方言记录。

### Lines 7-9: Section banner
**EN:** The comment block marks the beginning of the dialect type definitions and follows standard LLVM formatting conventions.

**CN:** 这一段注释标记了方言类型定义的开始，并遵循 LLVM 的常见格式风格。

### Lines 11-14: Reusable type base class
**EN:** `class DialectPlugin_Type<...> : TypeDef<DialectPlugin_Dialect, name, traits>` creates a reusable wrapper around `TypeDef` for this dialect. It also assigns the textual mnemonic via `let mnemonic = typeMnemonic;`.

**CN:** `class DialectPlugin_Type<...> : TypeDef<DialectPlugin_Dialect, name, traits>` 为该方言创建了一个可复用的 `TypeDef` 包装类。同时通过 `let mnemonic = typeMnemonic;` 设定文本形式中的类型助记符。

### Lines 16-21: Custom type definition
**EN:** `DialectPlugin_CustomType` defines a custom type named `Custom` with assembly mnemonic `custom`. It has summary/description metadata, one string parameter called `value`, and an assembly format of ``<value>`` so textual IR prints as something like `!plugin.custom<...>`.

**CN:** `DialectPlugin_CustomType` 定义了一个名为 `Custom`、文本助记符为 `custom` 的自定义类型。它包含摘要和描述元数据，带有一个名为 `value` 的字符串参数，并通过 ``<value>`` 的 assembly format 使文本 IR 表示形如 `!plugin.custom<...>`。

### Line 19: Parameter modeling
**EN:** `StringRefParameter<"the custom value">:$value` declares a typed parameter stored in the type instance. TableGen uses it to generate parameter accessors, storage logic, and parse/print support.

**CN:** `StringRefParameter<"the custom value">:$value` 声明了一个存储在类型实例中的强类型参数。TableGen 会利用它生成参数访问器、存储逻辑以及解析/打印支持。

### Line 20: Assembly format DSL
**EN:** The declarative format limits the textual syntax to angle brackets around the parameter value, relying on dialect/type context for the rest of the spelling.

**CN:** 这个声明式格式将文本语法限制为参数值外包一层尖括号，其余拼写部分则由方言/类型上下文补足。

### Line 23: Guard end
**EN:** Ends the type definition file.

**CN:** 结束该类型定义文件。

## Key Concepts / 关键概念
- **`TypeDef` in MLIR TableGen:** Declaratively defines dialect-owned custom types.
- **Mnemonic-driven syntax:** The mnemonic participates in textual spelling such as `!plugin.custom<...>`.
- **Parameterized custom types:** Type parameters become part of uniquing, storage, and generated API accessors.
- **`TypeDef`：** 用声明式方式定义由方言拥有的自定义类型。
- **助记符驱动语法：** 助记符会进入文本表示，例如 `!plugin.custom<...>`。
- **参数化自定义类型：** 类型参数会参与唯一化、存储和自动生成的 API 访问器。

## Dependencies / 依赖关系
- Depends on `mlir/IR/AttrTypeBase.td` and `DialectPluginDialect.td`.
- Feeds generated headers such as `DialectPluginOpsTypes.h.inc` and dialect type parser/printer support.
- Is included by `DialectPluginOps.td`, letting operations reference the custom type definitions.
- 为插件方言的类型注册与文本语法生成提供源元数据。

