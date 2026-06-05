# IRDLOps.td — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `mlir/include/mlir/Dialect/IRDL/IR/IRDLOps.td` | `mlir/include/mlir/Dialect/IRDL/IR/IRDLOps.td` |
| Repository | `llvm/llvm-project` | `llvm/llvm-project` |
| Purpose | This file declares the IRDL dialect ops. | 该文件声明了：the IRDL dialect ops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```tablegen
   1: //===- IRDLOps.td - IR Definition Language Dialect ---------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the IRDL dialect ops.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1:** This comment states: “===- IRDLOps.td - IR Definition Language Dialect ---------*- tablegen -*-===”, documenting the intent of the surrounding code.
  **CN L1:** 该注释写道：“===- IRDLOps.td - IR Definition Language Dialect ---------*- tablegen -*-===”，用于说明周围代码的意图。
- **EN L2:** This comment documents context for the surrounding code.
  **CN L2:** 该注释为周围代码提供上下文说明。
- **EN L3:** This comment states: “Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.”, documenting the intent of the surrounding code.
  **CN L3:** 该注释写道：“Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.”，用于说明周围代码的意图。
- **EN L4:** This comment states: “See https://llvm.org/LICENSE.txt for license information.”, documenting the intent of the surrounding code.
  **CN L4:** 该注释写道：“See https://llvm.org/LICENSE.txt for license information.”，用于说明周围代码的意图。
- **EN L5:** This comment states: “SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”, documenting the intent of the surrounding code.
  **CN L5:** 该注释写道：“SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”，用于说明周围代码的意图。
- **EN L6:** This comment documents context for the surrounding code.
  **CN L6:** 该注释为周围代码提供上下文说明。
- **EN L7:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L7:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L8:** This comment documents context for the surrounding code.
  **CN L8:** 该注释为周围代码提供上下文说明。
- **EN L9:** This comment states: “This file declares the IRDL dialect ops.”, documenting the intent of the surrounding code.
  **CN L9:** 该注释写道：“This file declares the IRDL dialect ops.”，用于说明周围代码的意图。
- **EN L10:** This comment documents context for the surrounding code.
  **CN L10:** 该注释为周围代码提供上下文说明。
- **EN L11:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L11:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L12:** Blank line used to separate nearby declarations and improve readability.
  **CN L12:** 该空行用于分隔相邻声明并提升可读性。

### Lines 13-24 / 第 13-24 行

```tablegen
  13: #ifndef MLIR_DIALECT_IRDL_IR_IRDLOPS
  14: #define MLIR_DIALECT_IRDL_IR_IRDLOPS
  15: 
  16: include "mlir/Dialect/IRDL/IR/IRDL.td"
  17: include "mlir/Dialect/IRDL/IR/IRDLAttributes.td"
  18: include "mlir/Dialect/IRDL/IR/IRDLTypes.td"
  19: include "mlir/Dialect/IRDL/IR/IRDLInterfaces.td"
  20: include "mlir/Interfaces/SideEffectInterfaces.td"
  21: include "mlir/Interfaces/InferTypeOpInterface.td"
  22: include "mlir/IR/SymbolInterfaces.td"
  23: include "mlir/IR/BuiltinAttributes.td"
  24: 
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN L13:** This preprocessor directive manages `MLIR_DIALECT_IRDL_IR_IRDLOPS` as part of the file's conditional compilation boundary.
  **CN L13:** 该预处理指令管理 `MLIR_DIALECT_IRDL_IR_IRDLOPS`，作为文件条件编译边界的一部分。
- **EN L14:** This preprocessor directive manages `MLIR_DIALECT_IRDL_IR_IRDLOPS` as part of the file's conditional compilation boundary.
  **CN L14:** 该预处理指令管理 `MLIR_DIALECT_IRDL_IR_IRDLOPS`，作为文件条件编译边界的一部分。
- **EN L15:** Blank line used to separate nearby declarations and improve readability.
  **CN L15:** 该空行用于分隔相邻声明并提升可读性。
- **EN L16:** This TableGen include reuses records from `mlir/Dialect/IRDL/IR/IRDL.td`.
  **CN L16:** 该 TableGen include 复用了 `mlir/Dialect/IRDL/IR/IRDL.td` 中的记录。
- **EN L17:** This TableGen include reuses records from `mlir/Dialect/IRDL/IR/IRDLAttributes.td`.
  **CN L17:** 该 TableGen include 复用了 `mlir/Dialect/IRDL/IR/IRDLAttributes.td` 中的记录。
- **EN L18:** This TableGen include reuses records from `mlir/Dialect/IRDL/IR/IRDLTypes.td`.
  **CN L18:** 该 TableGen include 复用了 `mlir/Dialect/IRDL/IR/IRDLTypes.td` 中的记录。
- **EN L19:** This TableGen include reuses records from `mlir/Dialect/IRDL/IR/IRDLInterfaces.td`.
  **CN L19:** 该 TableGen include 复用了 `mlir/Dialect/IRDL/IR/IRDLInterfaces.td` 中的记录。
- **EN L20:** This TableGen include reuses records from `mlir/Interfaces/SideEffectInterfaces.td`.
  **CN L20:** 该 TableGen include 复用了 `mlir/Interfaces/SideEffectInterfaces.td` 中的记录。
- **EN L21:** This TableGen include reuses records from `mlir/Interfaces/InferTypeOpInterface.td`.
  **CN L21:** 该 TableGen include 复用了 `mlir/Interfaces/InferTypeOpInterface.td` 中的记录。
- **EN L22:** This TableGen include reuses records from `mlir/IR/SymbolInterfaces.td`.
  **CN L22:** 该 TableGen include 复用了 `mlir/IR/SymbolInterfaces.td` 中的记录。
- **EN L23:** This TableGen include reuses records from `mlir/IR/BuiltinAttributes.td`.
  **CN L23:** 该 TableGen include 复用了 `mlir/IR/BuiltinAttributes.td` 中的记录。
- **EN L24:** Blank line used to separate nearby declarations and improve readability.
  **CN L24:** 该空行用于分隔相邻声明并提升可读性。

### Lines 25-36 / 第 25-36 行

```tablegen
  25: class IRDL_Op<string mnemonic, list<Trait> traits = []>
  26:     : Op<IRDL_Dialect, mnemonic, traits>;
  27: 
  28: class AtMostOneChildOf<string op> : ParamNativeOpTrait<"AtMostOneChildOf", op>;
  29: 
  30: //===----------------------------------------------------------------------===//
  31: // Dialect definition
  32: //===----------------------------------------------------------------------===//
  33: 
  34: def IRDL_DialectOp : IRDL_Op<"dialect",
  35:     [IsolatedFromAbove, NoTerminator, Symbol, SymbolTable]> {
  36:   let summary = "Define a new dialect";
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L25:** This TableGen `class` record introduces `IRDL_Op`, which later participates in generated MLIR code.
  **CN L25:** 该 TableGen `class` 记录引入了 `IRDL_Op`，后续会参与生成的 MLIR 代码。
- **EN L26:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L26:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L27:** Blank line used to separate nearby declarations and improve readability.
  **CN L27:** 该空行用于分隔相邻声明并提升可读性。
- **EN L28:** This TableGen `class` record introduces `AtMostOneChildOf`, which later participates in generated MLIR code.
  **CN L28:** 该 TableGen `class` 记录引入了 `AtMostOneChildOf`，后续会参与生成的 MLIR 代码。
- **EN L29:** Blank line used to separate nearby declarations and improve readability.
  **CN L29:** 该空行用于分隔相邻声明并提升可读性。
- **EN L30:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L30:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L31:** This comment states: “Dialect definition”, documenting the intent of the surrounding code.
  **CN L31:** 该注释写道：“Dialect definition”，用于说明周围代码的意图。
- **EN L32:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L32:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L33:** Blank line used to separate nearby declarations and improve readability.
  **CN L33:** 该空行用于分隔相邻声明并提升可读性。
- **EN L34:** This TableGen `def` record introduces `IRDL_DialectOp`, which later participates in generated MLIR code.
  **CN L34:** 该 TableGen `def` 记录引入了 `IRDL_DialectOp`，后续会参与生成的 MLIR 代码。
- **EN L35:** This line contributes implementation detail or declarative structure to the file.
  **CN L35:** 这一行为文件补充了实现细节或声明式结构。
- **EN L36:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L36:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 37-48 / 第 37-48 行

```tablegen
  37:   let description = [{
  38:     The `irdl.dialect` operation defines a dialect. All operations, attributes,
  39:     and types defined inside its region will be part of the dialect.
  40: 
  41:     Example:
  42: 
  43:     ```mlir
  44:     irdl.dialect @cmath {
  45:       ...
  46:     }
  47:     ```
  48: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L37:** This line contributes implementation detail or declarative structure to the file.
  **CN L37:** 这一行为文件补充了实现细节或声明式结构。
- **EN L38:** This line contributes implementation detail or declarative structure to the file.
  **CN L38:** 这一行为文件补充了实现细节或声明式结构。
- **EN L39:** This line contributes implementation detail or declarative structure to the file.
  **CN L39:** 这一行为文件补充了实现细节或声明式结构。
- **EN L40:** Blank line used to separate nearby declarations and improve readability.
  **CN L40:** 该空行用于分隔相邻声明并提升可读性。
- **EN L41:** This line contributes implementation detail or declarative structure to the file.
  **CN L41:** 这一行为文件补充了实现细节或声明式结构。
- **EN L42:** Blank line used to separate nearby declarations and improve readability.
  **CN L42:** 该空行用于分隔相邻声明并提升可读性。
- **EN L43:** This line contributes implementation detail or declarative structure to the file.
  **CN L43:** 这一行为文件补充了实现细节或声明式结构。
- **EN L44:** This line contributes implementation detail or declarative structure to the file.
  **CN L44:** 这一行为文件补充了实现细节或声明式结构。
- **EN L45:** This line contributes implementation detail or declarative structure to the file.
  **CN L45:** 这一行为文件补充了实现细节或声明式结构。
- **EN L46:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L46:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L47:** This line contributes implementation detail or declarative structure to the file.
  **CN L47:** 这一行为文件补充了实现细节或声明式结构。
- **EN L48:** Blank line used to separate nearby declarations and improve readability.
  **CN L48:** 该空行用于分隔相邻声明并提升可读性。

### Lines 49-60 / 第 49-60 行

```tablegen
  49:     The above program defines a `cmath` dialect.
  50:   }];
  51: 
  52:   let arguments = (ins SymbolNameAttr:$sym_name);
  53:   let regions = (region SizedRegion<1>:$body);
  54:   let assemblyFormat =
  55:     "$sym_name attr-dict-with-keyword custom<SingleBlockRegion>($body)";
  56:   let hasVerifier = 1;
  57: }
  58: 
  59: //===----------------------------------------------------------------------===//
  60: // Type and Attribute definition
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L49:** This line contributes implementation detail or declarative structure to the file.
  **CN L49:** 这一行为文件补充了实现细节或声明式结构。
- **EN L50:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L50:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L51:** Blank line used to separate nearby declarations and improve readability.
  **CN L51:** 该空行用于分隔相邻声明并提升可读性。
- **EN L52:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L52:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L53:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L53:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L54:** This line contributes implementation detail or declarative structure to the file.
  **CN L54:** 这一行为文件补充了实现细节或声明式结构。
- **EN L55:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L55:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L56:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L56:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L57:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L57:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L58:** Blank line used to separate nearby declarations and improve readability.
  **CN L58:** 该空行用于分隔相邻声明并提升可读性。
- **EN L59:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L59:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L60:** This comment states: “Type and Attribute definition”, documenting the intent of the surrounding code.
  **CN L60:** 该注释写道：“Type and Attribute definition”，用于说明周围代码的意图。

### Lines 61-72 / 第 61-72 行

```tablegen
  61: //===----------------------------------------------------------------------===//
  62: 
  63: def IRDL_TypeOp : IRDL_Op<"type",
  64:     [HasParent<"DialectOp">, NoTerminator, NoRegionArguments,
  65:      AtMostOneChildOf<"ParametersOp">, Symbol, IsolatedFromAbove]> {
  66:   let summary = "Define a new type";
  67:   let description = [{
  68:     `irdl.type` defines a new type belonging to the `irdl.dialect` parent.
  69: 
  70:     The type parameters can be defined with an `irdl.parameters` operation in
  71:     the optional region.
  72: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L61:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L61:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L62:** Blank line used to separate nearby declarations and improve readability.
  **CN L62:** 该空行用于分隔相邻声明并提升可读性。
- **EN L63:** This TableGen `def` record introduces `IRDL_TypeOp`, which later participates in generated MLIR code.
  **CN L63:** 该 TableGen `def` 记录引入了 `IRDL_TypeOp`，后续会参与生成的 MLIR 代码。
- **EN L64:** This line contributes implementation detail or declarative structure to the file.
  **CN L64:** 这一行为文件补充了实现细节或声明式结构。
- **EN L65:** This line contributes implementation detail or declarative structure to the file.
  **CN L65:** 这一行为文件补充了实现细节或声明式结构。
- **EN L66:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L66:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L67:** This line contributes implementation detail or declarative structure to the file.
  **CN L67:** 这一行为文件补充了实现细节或声明式结构。
- **EN L68:** This line contributes implementation detail or declarative structure to the file.
  **CN L68:** 这一行为文件补充了实现细节或声明式结构。
- **EN L69:** Blank line used to separate nearby declarations and improve readability.
  **CN L69:** 该空行用于分隔相邻声明并提升可读性。
- **EN L70:** This line contributes implementation detail or declarative structure to the file.
  **CN L70:** 这一行为文件补充了实现细节或声明式结构。
- **EN L71:** This line contributes implementation detail or declarative structure to the file.
  **CN L71:** 这一行为文件补充了实现细节或声明式结构。
- **EN L72:** Blank line used to separate nearby declarations and improve readability.
  **CN L72:** 该空行用于分隔相邻声明并提升可读性。

### Lines 73-84 / 第 73-84 行

```tablegen
  73:     Example:
  74: 
  75:     ```mlir
  76:     irdl.dialect @cmath {
  77:       irdl.type @complex {
  78:         %0 = irdl.is i32
  79:         %1 = irdl.is i64
  80:         %2 = irdl.any_of(%0, %1)
  81:         irdl.parameters(%2)
  82:       }
  83:     }
  84:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L73:** This line contributes implementation detail or declarative structure to the file.
  **CN L73:** 这一行为文件补充了实现细节或声明式结构。
- **EN L74:** Blank line used to separate nearby declarations and improve readability.
  **CN L74:** 该空行用于分隔相邻声明并提升可读性。
- **EN L75:** This line contributes implementation detail or declarative structure to the file.
  **CN L75:** 这一行为文件补充了实现细节或声明式结构。
- **EN L76:** This line contributes implementation detail or declarative structure to the file.
  **CN L76:** 这一行为文件补充了实现细节或声明式结构。
- **EN L77:** This line contributes implementation detail or declarative structure to the file.
  **CN L77:** 这一行为文件补充了实现细节或声明式结构。
- **EN L78:** This line contributes implementation detail or declarative structure to the file.
  **CN L78:** 这一行为文件补充了实现细节或声明式结构。
- **EN L79:** This line contributes implementation detail or declarative structure to the file.
  **CN L79:** 这一行为文件补充了实现细节或声明式结构。
- **EN L80:** This line contributes to the declaration or call of `any_of`.
  **CN L80:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L81:** This line contributes to the declaration or call of `parameters`.
  **CN L81:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L82:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L82:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L83:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L83:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L84:** This line contributes implementation detail or declarative structure to the file.
  **CN L84:** 这一行为文件补充了实现细节或声明式结构。

### Lines 85-96 / 第 85-96 行

```tablegen
  85: 
  86:     The above program defines a type `complex` inside the dialect `cmath`. The
  87:     type has a single parameter that should be either `i32` or `i64`.
  88:   }];
  89: 
  90:   let arguments = (ins SymbolNameAttr:$sym_name);
  91:   let regions = (region SizedRegion<1>:$body);
  92:   let assemblyFormat =
  93:     "$sym_name attr-dict-with-keyword custom<SingleBlockRegion>($body)";
  94:   let hasVerifier = 1;
  95: }
  96: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L85:** Blank line used to separate nearby declarations and improve readability.
  **CN L85:** 该空行用于分隔相邻声明并提升可读性。
- **EN L86:** This line contributes implementation detail or declarative structure to the file.
  **CN L86:** 这一行为文件补充了实现细节或声明式结构。
- **EN L87:** This line contributes implementation detail or declarative structure to the file.
  **CN L87:** 这一行为文件补充了实现细节或声明式结构。
- **EN L88:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L88:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L89:** Blank line used to separate nearby declarations and improve readability.
  **CN L89:** 该空行用于分隔相邻声明并提升可读性。
- **EN L90:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L90:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L91:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L91:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L92:** This line contributes implementation detail or declarative structure to the file.
  **CN L92:** 这一行为文件补充了实现细节或声明式结构。
- **EN L93:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L93:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L94:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L94:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L95:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L95:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L96:** Blank line used to separate nearby declarations and improve readability.
  **CN L96:** 该空行用于分隔相邻声明并提升可读性。

### Lines 97-108 / 第 97-108 行

```tablegen
  97: def IRDL_AttributeOp : IRDL_Op<"attribute",
  98:     [HasParent<"DialectOp">, NoTerminator, NoRegionArguments,
  99:      AtMostOneChildOf<"ParametersOp">, Symbol, IsolatedFromAbove]> {
 100:   let summary = "Define a new attribute";
 101:   let description = [{
 102:     `irdl.attribute` defines a new attribute belonging to the `irdl.dialect`
 103:     parent.
 104: 
 105:     The attribute parameters can be defined with an `irdl.parameters` operation
 106:     in the optional region.
 107: 
 108:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L97:** This TableGen `def` record introduces `IRDL_AttributeOp`, which later participates in generated MLIR code.
  **CN L97:** 该 TableGen `def` 记录引入了 `IRDL_AttributeOp`，后续会参与生成的 MLIR 代码。
- **EN L98:** This line contributes implementation detail or declarative structure to the file.
  **CN L98:** 这一行为文件补充了实现细节或声明式结构。
- **EN L99:** This line contributes implementation detail or declarative structure to the file.
  **CN L99:** 这一行为文件补充了实现细节或声明式结构。
- **EN L100:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L100:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L101:** This line contributes implementation detail or declarative structure to the file.
  **CN L101:** 这一行为文件补充了实现细节或声明式结构。
- **EN L102:** This line contributes implementation detail or declarative structure to the file.
  **CN L102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L103:** This line contributes implementation detail or declarative structure to the file.
  **CN L103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L104:** Blank line used to separate nearby declarations and improve readability.
  **CN L104:** 该空行用于分隔相邻声明并提升可读性。
- **EN L105:** This line contributes implementation detail or declarative structure to the file.
  **CN L105:** 这一行为文件补充了实现细节或声明式结构。
- **EN L106:** This line contributes implementation detail or declarative structure to the file.
  **CN L106:** 这一行为文件补充了实现细节或声明式结构。
- **EN L107:** Blank line used to separate nearby declarations and improve readability.
  **CN L107:** 该空行用于分隔相邻声明并提升可读性。
- **EN L108:** This line contributes implementation detail or declarative structure to the file.
  **CN L108:** 这一行为文件补充了实现细节或声明式结构。

### Lines 109-120 / 第 109-120 行

```tablegen
 109: 
 110:     ```mlir
 111:     irdl.dialect @testd {
 112:       irdl.attribute @enum_attr {
 113:         %0 = irdl.is "foo"
 114:         %1 = irdl.is "bar"
 115:         %2 = irdl.any_of(%0, %1)
 116:         irdl.parameters(%2)
 117:       }
 118:     }
 119:     ```
 120: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L109:** Blank line used to separate nearby declarations and improve readability.
  **CN L109:** 该空行用于分隔相邻声明并提升可读性。
- **EN L110:** This line contributes implementation detail or declarative structure to the file.
  **CN L110:** 这一行为文件补充了实现细节或声明式结构。
- **EN L111:** This line contributes implementation detail or declarative structure to the file.
  **CN L111:** 这一行为文件补充了实现细节或声明式结构。
- **EN L112:** This line contributes implementation detail or declarative structure to the file.
  **CN L112:** 这一行为文件补充了实现细节或声明式结构。
- **EN L113:** This line contributes implementation detail or declarative structure to the file.
  **CN L113:** 这一行为文件补充了实现细节或声明式结构。
- **EN L114:** This line contributes implementation detail or declarative structure to the file.
  **CN L114:** 这一行为文件补充了实现细节或声明式结构。
- **EN L115:** This line contributes to the declaration or call of `any_of`.
  **CN L115:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L116:** This line contributes to the declaration or call of `parameters`.
  **CN L116:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L117:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L117:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L118:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L118:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L119:** This line contributes implementation detail or declarative structure to the file.
  **CN L119:** 这一行为文件补充了实现细节或声明式结构。
- **EN L120:** Blank line used to separate nearby declarations and improve readability.
  **CN L120:** 该空行用于分隔相邻声明并提升可读性。

### Lines 121-132 / 第 121-132 行

```tablegen
 121:     The above program defines an `enum_attr` attribute inside the `testd`
 122:     dialect. The attribute has one `StringAttr` parameter that should be
 123:     either a `"foo"` or a `"bar"`.
 124:   }];
 125: 
 126:   let arguments = (ins SymbolNameAttr:$sym_name);
 127:   let regions = (region SizedRegion<1>:$body);
 128:   let assemblyFormat =
 129:     "$sym_name attr-dict-with-keyword custom<SingleBlockRegion>($body)";
 130: 
 131:   let hasVerifier = 1;
 132: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L121:** This line contributes implementation detail or declarative structure to the file.
  **CN L121:** 这一行为文件补充了实现细节或声明式结构。
- **EN L122:** This line contributes implementation detail or declarative structure to the file.
  **CN L122:** 这一行为文件补充了实现细节或声明式结构。
- **EN L123:** This line contributes implementation detail or declarative structure to the file.
  **CN L123:** 这一行为文件补充了实现细节或声明式结构。
- **EN L124:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L124:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L125:** Blank line used to separate nearby declarations and improve readability.
  **CN L125:** 该空行用于分隔相邻声明并提升可读性。
- **EN L126:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L126:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L127:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L127:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L128:** This line contributes implementation detail or declarative structure to the file.
  **CN L128:** 这一行为文件补充了实现细节或声明式结构。
- **EN L129:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L129:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L130:** Blank line used to separate nearby declarations and improve readability.
  **CN L130:** 该空行用于分隔相邻声明并提升可读性。
- **EN L131:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L131:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L132:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L132:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 133-144 / 第 133-144 行

```tablegen
 133: 
 134: def IRDL_ParametersOp : IRDL_Op<"parameters",
 135:     [ParentOneOf<["AttributeOp", "TypeOp"]>]> {
 136:   let summary =
 137:     "Define the constraints on parameters of a type/attribute definition";
 138:   let description = [{
 139:     `irdl.parameters` defines the constraints on parameters of a type or
 140:     attribute definition. Each parameter is named after an identifier.
 141: 
 142:     Example:
 143: 
 144:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L133:** Blank line used to separate nearby declarations and improve readability.
  **CN L133:** 该空行用于分隔相邻声明并提升可读性。
- **EN L134:** This TableGen `def` record introduces `IRDL_ParametersOp`, which later participates in generated MLIR code.
  **CN L134:** 该 TableGen `def` 记录引入了 `IRDL_ParametersOp`，后续会参与生成的 MLIR 代码。
- **EN L135:** This line contributes implementation detail or declarative structure to the file.
  **CN L135:** 这一行为文件补充了实现细节或声明式结构。
- **EN L136:** This line contributes implementation detail or declarative structure to the file.
  **CN L136:** 这一行为文件补充了实现细节或声明式结构。
- **EN L137:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L137:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L138:** This line contributes implementation detail or declarative structure to the file.
  **CN L138:** 这一行为文件补充了实现细节或声明式结构。
- **EN L139:** This line contributes implementation detail or declarative structure to the file.
  **CN L139:** 这一行为文件补充了实现细节或声明式结构。
- **EN L140:** This line contributes implementation detail or declarative structure to the file.
  **CN L140:** 这一行为文件补充了实现细节或声明式结构。
- **EN L141:** Blank line used to separate nearby declarations and improve readability.
  **CN L141:** 该空行用于分隔相邻声明并提升可读性。
- **EN L142:** This line contributes implementation detail or declarative structure to the file.
  **CN L142:** 这一行为文件补充了实现细节或声明式结构。
- **EN L143:** Blank line used to separate nearby declarations and improve readability.
  **CN L143:** 该空行用于分隔相邻声明并提升可读性。
- **EN L144:** This line contributes implementation detail or declarative structure to the file.
  **CN L144:** 这一行为文件补充了实现细节或声明式结构。

### Lines 145-156 / 第 145-156 行

```tablegen
 145:     irdl.dialect @cmath {
 146:       irdl.type @complex {
 147:         %0 = irdl.is i32
 148:         %1 = irdl.is i64
 149:         %2 = irdl.any_of(%0, %1)
 150:         irdl.parameters(elem: %2)
 151:       }
 152:     }
 153:     ```
 154: 
 155:     The above program defines a type `complex` inside the dialect `cmath`. The
 156:     type has a single parameter `elem` that should be either `i32` or `i64`.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L145:** This line contributes implementation detail or declarative structure to the file.
  **CN L145:** 这一行为文件补充了实现细节或声明式结构。
- **EN L146:** This line contributes implementation detail or declarative structure to the file.
  **CN L146:** 这一行为文件补充了实现细节或声明式结构。
- **EN L147:** This line contributes implementation detail or declarative structure to the file.
  **CN L147:** 这一行为文件补充了实现细节或声明式结构。
- **EN L148:** This line contributes implementation detail or declarative structure to the file.
  **CN L148:** 这一行为文件补充了实现细节或声明式结构。
- **EN L149:** This line contributes to the declaration or call of `any_of`.
  **CN L149:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L150:** This line contributes to the declaration or call of `parameters`.
  **CN L150:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L151:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L151:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L152:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L152:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L153:** This line contributes implementation detail or declarative structure to the file.
  **CN L153:** 这一行为文件补充了实现细节或声明式结构。
- **EN L154:** Blank line used to separate nearby declarations and improve readability.
  **CN L154:** 该空行用于分隔相邻声明并提升可读性。
- **EN L155:** This line contributes implementation detail or declarative structure to the file.
  **CN L155:** 这一行为文件补充了实现细节或声明式结构。
- **EN L156:** This line contributes implementation detail or declarative structure to the file.
  **CN L156:** 这一行为文件补充了实现细节或声明式结构。

### Lines 157-168 / 第 157-168 行

```tablegen
 157:   }];
 158: 
 159:   let arguments = (ins Variadic<IRDL_AttributeType>:$args,
 160:                         StrArrayAttr:$names);
 161:   let assemblyFormat = " `` custom<NamedValueList>($args, $names) attr-dict ";
 162:   let hasVerifier = true;
 163: }
 164: 
 165: //===----------------------------------------------------------------------===//
 166: // IRDL Operation definition
 167: //===----------------------------------------------------------------------===//
 168: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L157:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L157:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L158:** Blank line used to separate nearby declarations and improve readability.
  **CN L158:** 该空行用于分隔相邻声明并提升可读性。
- **EN L159:** This line contributes implementation detail or declarative structure to the file.
  **CN L159:** 这一行为文件补充了实现细节或声明式结构。
- **EN L160:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L160:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L161:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L161:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L162:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L162:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L163:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L163:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L164:** Blank line used to separate nearby declarations and improve readability.
  **CN L164:** 该空行用于分隔相邻声明并提升可读性。
- **EN L165:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L165:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L166:** This comment states: “IRDL Operation definition”, documenting the intent of the surrounding code.
  **CN L166:** 该注释写道：“IRDL Operation definition”，用于说明周围代码的意图。
- **EN L167:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L167:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L168:** Blank line used to separate nearby declarations and improve readability.
  **CN L168:** 该空行用于分隔相邻声明并提升可读性。

### Lines 169-180 / 第 169-180 行

```tablegen
 169: def IRDL_OperationOp : IRDL_Op<"operation",
 170:     [HasParent<"DialectOp">, NoTerminator, NoRegionArguments,
 171:     AtMostOneChildOf<"OperandsOp, ResultsOp, AttributesOp, RegionsOp">,
 172:     Symbol, IsolatedFromAbove]> {
 173:   let summary = "Define a new operation";
 174:   let description = [{
 175:     `irdl.operation` defines a new operation belonging to the `irdl.dialect`
 176:     parent.
 177: 
 178:     Operations can define constraints on their operands and results with the
 179:     `irdl.results` and `irdl.operands` operations. If these operations are not
 180:     present in the region, the results or operands are expected to be empty.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L169:** This TableGen `def` record introduces `IRDL_OperationOp`, which later participates in generated MLIR code.
  **CN L169:** 该 TableGen `def` 记录引入了 `IRDL_OperationOp`，后续会参与生成的 MLIR 代码。
- **EN L170:** This line contributes implementation detail or declarative structure to the file.
  **CN L170:** 这一行为文件补充了实现细节或声明式结构。
- **EN L171:** This line contributes implementation detail or declarative structure to the file.
  **CN L171:** 这一行为文件补充了实现细节或声明式结构。
- **EN L172:** This line contributes implementation detail or declarative structure to the file.
  **CN L172:** 这一行为文件补充了实现细节或声明式结构。
- **EN L173:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L173:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L174:** This line contributes implementation detail or declarative structure to the file.
  **CN L174:** 这一行为文件补充了实现细节或声明式结构。
- **EN L175:** This line contributes implementation detail or declarative structure to the file.
  **CN L175:** 这一行为文件补充了实现细节或声明式结构。
- **EN L176:** This line contributes implementation detail or declarative structure to the file.
  **CN L176:** 这一行为文件补充了实现细节或声明式结构。
- **EN L177:** Blank line used to separate nearby declarations and improve readability.
  **CN L177:** 该空行用于分隔相邻声明并提升可读性。
- **EN L178:** This line contributes implementation detail or declarative structure to the file.
  **CN L178:** 这一行为文件补充了实现细节或声明式结构。
- **EN L179:** This line contributes implementation detail or declarative structure to the file.
  **CN L179:** 这一行为文件补充了实现细节或声明式结构。
- **EN L180:** This line contributes implementation detail or declarative structure to the file.
  **CN L180:** 这一行为文件补充了实现细节或声明式结构。

### Lines 181-192 / 第 181-192 行

```tablegen
 181: 
 182:     Example:
 183: 
 184:     ```mlir
 185:     irdl.dialect @cmath {
 186: 
 187:       irdl.type @complex { /* ... */ }
 188: 
 189:       irdl.operation @norm {
 190:         %0 = irdl.any
 191:         %1 = irdl.parametric @cmath::@complex<%0>
 192:         irdl.results(%0)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L181:** Blank line used to separate nearby declarations and improve readability.
  **CN L181:** 该空行用于分隔相邻声明并提升可读性。
- **EN L182:** This line contributes implementation detail or declarative structure to the file.
  **CN L182:** 这一行为文件补充了实现细节或声明式结构。
- **EN L183:** Blank line used to separate nearby declarations and improve readability.
  **CN L183:** 该空行用于分隔相邻声明并提升可读性。
- **EN L184:** This line contributes implementation detail or declarative structure to the file.
  **CN L184:** 这一行为文件补充了实现细节或声明式结构。
- **EN L185:** This line contributes implementation detail or declarative structure to the file.
  **CN L185:** 这一行为文件补充了实现细节或声明式结构。
- **EN L186:** Blank line used to separate nearby declarations and improve readability.
  **CN L186:** 该空行用于分隔相邻声明并提升可读性。
- **EN L187:** This line contributes implementation detail or declarative structure to the file.
  **CN L187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L188:** Blank line used to separate nearby declarations and improve readability.
  **CN L188:** 该空行用于分隔相邻声明并提升可读性。
- **EN L189:** This line contributes implementation detail or declarative structure to the file.
  **CN L189:** 这一行为文件补充了实现细节或声明式结构。
- **EN L190:** This line contributes implementation detail or declarative structure to the file.
  **CN L190:** 这一行为文件补充了实现细节或声明式结构。
- **EN L191:** This line contributes implementation detail or declarative structure to the file.
  **CN L191:** 这一行为文件补充了实现细节或声明式结构。
- **EN L192:** This line contributes to the declaration or call of `results`.
  **CN L192:** 这一行为 `results` 的声明或调用提供内容。

### Lines 193-204 / 第 193-204 行

```tablegen
 193:         irdl.operands(%1)
 194:       }
 195:     }
 196:     ```
 197: 
 198:     The above program defines an operation `norm` inside the dialect `cmath`.
 199:     The operation expects a single operand of base type `cmath.complex`, and
 200:     returns a single result of the element type of the operand.
 201:   }];
 202: 
 203:   let arguments = (ins SymbolNameAttr:$sym_name);
 204:   let regions = (region SizedRegion<1>:$body);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L193:** This line contributes to the declaration or call of `operands`.
  **CN L193:** 这一行为 `operands` 的声明或调用提供内容。
- **EN L194:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L194:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L195:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L195:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L196:** This line contributes implementation detail or declarative structure to the file.
  **CN L196:** 这一行为文件补充了实现细节或声明式结构。
- **EN L197:** Blank line used to separate nearby declarations and improve readability.
  **CN L197:** 该空行用于分隔相邻声明并提升可读性。
- **EN L198:** This line contributes implementation detail or declarative structure to the file.
  **CN L198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L199:** This line contributes implementation detail or declarative structure to the file.
  **CN L199:** 这一行为文件补充了实现细节或声明式结构。
- **EN L200:** This line contributes implementation detail or declarative structure to the file.
  **CN L200:** 这一行为文件补充了实现细节或声明式结构。
- **EN L201:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L201:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L202:** Blank line used to separate nearby declarations and improve readability.
  **CN L202:** 该空行用于分隔相邻声明并提升可读性。
- **EN L203:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L203:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L204:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L204:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 205-216 / 第 205-216 行

```tablegen
 205:   let assemblyFormat =
 206:     "$sym_name attr-dict-with-keyword custom<SingleBlockRegion>($body)";
 207:   let hasRegionVerifier = true;
 208:   let hasVerifier = 1;
 209: }
 210: 
 211: def IRDL_OperandsOp : IRDL_Op<"operands", [HasParent<"OperationOp">]> {
 212:   let summary = "Define the operands of an operation";
 213:   let description = [{
 214:     `irdl.operands` define the operands of the `irdl.operation` parent operation
 215:     definition. Each operand is named after an identifier.
 216: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L205:** This line contributes implementation detail or declarative structure to the file.
  **CN L205:** 这一行为文件补充了实现细节或声明式结构。
- **EN L206:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L206:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L207:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L207:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L208:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L208:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L209:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L209:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L210:** Blank line used to separate nearby declarations and improve readability.
  **CN L210:** 该空行用于分隔相邻声明并提升可读性。
- **EN L211:** This TableGen `def` record introduces `IRDL_OperandsOp`, which later participates in generated MLIR code.
  **CN L211:** 该 TableGen `def` 记录引入了 `IRDL_OperandsOp`，后续会参与生成的 MLIR 代码。
- **EN L212:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L212:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L213:** This line contributes implementation detail or declarative structure to the file.
  **CN L213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L214:** This line contributes implementation detail or declarative structure to the file.
  **CN L214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L215:** This line contributes implementation detail or declarative structure to the file.
  **CN L215:** 这一行为文件补充了实现细节或声明式结构。
- **EN L216:** Blank line used to separate nearby declarations and improve readability.
  **CN L216:** 该空行用于分隔相邻声明并提升可读性。

### Lines 217-228 / 第 217-228 行

```tablegen
 217:     In the following example, `irdl.operands` defines the operands of the
 218:     `mul` operation:
 219: 
 220:     ```mlir
 221:     irdl.dialect @cmath {
 222: 
 223:       irdl.type @complex { /* ... */ }
 224: 
 225:       irdl.operation @mul {
 226:         %0 = irdl.any
 227:         %1 = irdl.parametric @cmath::@complex<%0>
 228:         irdl.results(res: %1)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L217:** This line contributes implementation detail or declarative structure to the file.
  **CN L217:** 这一行为文件补充了实现细节或声明式结构。
- **EN L218:** This line contributes implementation detail or declarative structure to the file.
  **CN L218:** 这一行为文件补充了实现细节或声明式结构。
- **EN L219:** Blank line used to separate nearby declarations and improve readability.
  **CN L219:** 该空行用于分隔相邻声明并提升可读性。
- **EN L220:** This line contributes implementation detail or declarative structure to the file.
  **CN L220:** 这一行为文件补充了实现细节或声明式结构。
- **EN L221:** This line contributes implementation detail or declarative structure to the file.
  **CN L221:** 这一行为文件补充了实现细节或声明式结构。
- **EN L222:** Blank line used to separate nearby declarations and improve readability.
  **CN L222:** 该空行用于分隔相邻声明并提升可读性。
- **EN L223:** This line contributes implementation detail or declarative structure to the file.
  **CN L223:** 这一行为文件补充了实现细节或声明式结构。
- **EN L224:** Blank line used to separate nearby declarations and improve readability.
  **CN L224:** 该空行用于分隔相邻声明并提升可读性。
- **EN L225:** This line contributes implementation detail or declarative structure to the file.
  **CN L225:** 这一行为文件补充了实现细节或声明式结构。
- **EN L226:** This line contributes implementation detail or declarative structure to the file.
  **CN L226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L227:** This line contributes implementation detail or declarative structure to the file.
  **CN L227:** 这一行为文件补充了实现细节或声明式结构。
- **EN L228:** This line contributes to the declaration or call of `results`.
  **CN L228:** 这一行为 `results` 的声明或调用提供内容。

### Lines 229-240 / 第 229-240 行

```tablegen
 229:         irdl.operands(lhs: %1, rhs: %1)
 230:       }
 231:     }
 232:     ```
 233: 
 234:     The `mul` operation will expect two operands of type `cmath.complex`, that
 235:     have the same type, and return a result of the same type.
 236: 
 237:     The operands can also be marked as variadic or optional:
 238:     ```mlir
 239:     irdl.operands(foo: %0, bar: single %1, baz: optional %2, qux: variadic %3)
 240:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L229:** This line contributes to the declaration or call of `operands`.
  **CN L229:** 这一行为 `operands` 的声明或调用提供内容。
- **EN L230:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L230:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L231:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L231:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L232:** This line contributes implementation detail or declarative structure to the file.
  **CN L232:** 这一行为文件补充了实现细节或声明式结构。
- **EN L233:** Blank line used to separate nearby declarations and improve readability.
  **CN L233:** 该空行用于分隔相邻声明并提升可读性。
- **EN L234:** This line contributes implementation detail or declarative structure to the file.
  **CN L234:** 这一行为文件补充了实现细节或声明式结构。
- **EN L235:** This line contributes implementation detail or declarative structure to the file.
  **CN L235:** 这一行为文件补充了实现细节或声明式结构。
- **EN L236:** Blank line used to separate nearby declarations and improve readability.
  **CN L236:** 该空行用于分隔相邻声明并提升可读性。
- **EN L237:** This line contributes implementation detail or declarative structure to the file.
  **CN L237:** 这一行为文件补充了实现细节或声明式结构。
- **EN L238:** This line contributes implementation detail or declarative structure to the file.
  **CN L238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L239:** This line contributes to the declaration or call of `operands`.
  **CN L239:** 这一行为 `operands` 的声明或调用提供内容。
- **EN L240:** This line contributes implementation detail or declarative structure to the file.
  **CN L240:** 这一行为文件补充了实现细节或声明式结构。

### Lines 241-252 / 第 241-252 行

```tablegen
 241: 
 242:     Here, foo and bar are required single operands, baz is an optional operand,
 243:     and qux is a variadic operand.
 244: 
 245:     When more than one operand is marked as optional or variadic, the operation
 246:     will expect a 'operandSegmentSizes' attribute that defines the number of
 247:     operands in each segment.
 248:   }];
 249: 
 250:   let arguments = (ins Variadic<IRDL_AttributeType>:$args,
 251:                        StrArrayAttr:$names,
 252:                        VariadicityArrayAttr:$variadicity);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L241:** Blank line used to separate nearby declarations and improve readability.
  **CN L241:** 该空行用于分隔相邻声明并提升可读性。
- **EN L242:** This line contributes implementation detail or declarative structure to the file.
  **CN L242:** 这一行为文件补充了实现细节或声明式结构。
- **EN L243:** This line contributes implementation detail or declarative structure to the file.
  **CN L243:** 这一行为文件补充了实现细节或声明式结构。
- **EN L244:** Blank line used to separate nearby declarations and improve readability.
  **CN L244:** 该空行用于分隔相邻声明并提升可读性。
- **EN L245:** This line contributes implementation detail or declarative structure to the file.
  **CN L245:** 这一行为文件补充了实现细节或声明式结构。
- **EN L246:** This line contributes implementation detail or declarative structure to the file.
  **CN L246:** 这一行为文件补充了实现细节或声明式结构。
- **EN L247:** This line contributes implementation detail or declarative structure to the file.
  **CN L247:** 这一行为文件补充了实现细节或声明式结构。
- **EN L248:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L248:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L249:** Blank line used to separate nearby declarations and improve readability.
  **CN L249:** 该空行用于分隔相邻声明并提升可读性。
- **EN L250:** This line contributes implementation detail or declarative structure to the file.
  **CN L250:** 这一行为文件补充了实现细节或声明式结构。
- **EN L251:** This line contributes implementation detail or declarative structure to the file.
  **CN L251:** 这一行为文件补充了实现细节或声明式结构。
- **EN L252:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L252:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 253-264 / 第 253-264 行

```tablegen
 253:   let assemblyFormat =
 254:     " `` custom<NamedValueListWithVariadicity>($args, $names, $variadicity) attr-dict";
 255:   let hasVerifier = true;
 256: }
 257: 
 258: def IRDL_ResultsOp : IRDL_Op<"results", [HasParent<"OperationOp">]> {
 259:   let summary = "Define the results of an operation";
 260:   let description = [{
 261:     `irdl.results` define the results of the `irdl.operation` parent operation
 262:     definition. Each result is named after an identifier.
 263: 
 264:     In the following example, `irdl.results` defines the results of the
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L253:** This line contributes implementation detail or declarative structure to the file.
  **CN L253:** 这一行为文件补充了实现细节或声明式结构。
- **EN L254:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L254:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L255:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L255:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L256:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L256:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L257:** Blank line used to separate nearby declarations and improve readability.
  **CN L257:** 该空行用于分隔相邻声明并提升可读性。
- **EN L258:** This TableGen `def` record introduces `IRDL_ResultsOp`, which later participates in generated MLIR code.
  **CN L258:** 该 TableGen `def` 记录引入了 `IRDL_ResultsOp`，后续会参与生成的 MLIR 代码。
- **EN L259:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L259:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L260:** This line contributes implementation detail or declarative structure to the file.
  **CN L260:** 这一行为文件补充了实现细节或声明式结构。
- **EN L261:** This line contributes implementation detail or declarative structure to the file.
  **CN L261:** 这一行为文件补充了实现细节或声明式结构。
- **EN L262:** This line contributes implementation detail or declarative structure to the file.
  **CN L262:** 这一行为文件补充了实现细节或声明式结构。
- **EN L263:** Blank line used to separate nearby declarations and improve readability.
  **CN L263:** 该空行用于分隔相邻声明并提升可读性。
- **EN L264:** This line contributes implementation detail or declarative structure to the file.
  **CN L264:** 这一行为文件补充了实现细节或声明式结构。

### Lines 265-276 / 第 265-276 行

```tablegen
 265:     `get_values` operation:
 266: 
 267:     ```mlir
 268:     irdl.dialect @cmath {
 269: 
 270:       irdl.type @complex { /* ... */ }
 271: 
 272:       /// Returns the real and imaginary parts of a complex number.
 273:       irdl.operation @get_values {
 274:         %0 = irdl.any
 275:         %1 = irdl.parametric @cmath::@complex<%0>
 276:         irdl.results(re: %0, im: %0)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L265:** This line contributes implementation detail or declarative structure to the file.
  **CN L265:** 这一行为文件补充了实现细节或声明式结构。
- **EN L266:** Blank line used to separate nearby declarations and improve readability.
  **CN L266:** 该空行用于分隔相邻声明并提升可读性。
- **EN L267:** This line contributes implementation detail or declarative structure to the file.
  **CN L267:** 这一行为文件补充了实现细节或声明式结构。
- **EN L268:** This line contributes implementation detail or declarative structure to the file.
  **CN L268:** 这一行为文件补充了实现细节或声明式结构。
- **EN L269:** Blank line used to separate nearby declarations and improve readability.
  **CN L269:** 该空行用于分隔相邻声明并提升可读性。
- **EN L270:** This line contributes implementation detail or declarative structure to the file.
  **CN L270:** 这一行为文件补充了实现细节或声明式结构。
- **EN L271:** Blank line used to separate nearby declarations and improve readability.
  **CN L271:** 该空行用于分隔相邻声明并提升可读性。
- **EN L272:** This comment states: “Returns the real and imaginary parts of a complex number.”, documenting the intent of the surrounding code.
  **CN L272:** 该注释写道：“Returns the real and imaginary parts of a complex number.”，用于说明周围代码的意图。
- **EN L273:** This line contributes implementation detail or declarative structure to the file.
  **CN L273:** 这一行为文件补充了实现细节或声明式结构。
- **EN L274:** This line contributes implementation detail or declarative structure to the file.
  **CN L274:** 这一行为文件补充了实现细节或声明式结构。
- **EN L275:** This line contributes implementation detail or declarative structure to the file.
  **CN L275:** 这一行为文件补充了实现细节或声明式结构。
- **EN L276:** This line contributes to the declaration or call of `results`.
  **CN L276:** 这一行为 `results` 的声明或调用提供内容。

### Lines 277-288 / 第 277-288 行

```tablegen
 277:         irdl.operands(complex: %1)
 278:       }
 279:     }
 280:     ```
 281: 
 282:     The operation will expect one operand of the `cmath.complex` type, and two
 283:     results that have the underlying type of the `cmath.complex`.
 284: 
 285:     The results can also be marked as variadic or optional:
 286:     ```mlir
 287:     irdl.results(foo: %0, bar: single %1, baz: optional %2, qux: variadic %3)
 288:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L277:** This line contributes to the declaration or call of `operands`.
  **CN L277:** 这一行为 `operands` 的声明或调用提供内容。
- **EN L278:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L278:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L279:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L279:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L280:** This line contributes implementation detail or declarative structure to the file.
  **CN L280:** 这一行为文件补充了实现细节或声明式结构。
- **EN L281:** Blank line used to separate nearby declarations and improve readability.
  **CN L281:** 该空行用于分隔相邻声明并提升可读性。
- **EN L282:** This line contributes implementation detail or declarative structure to the file.
  **CN L282:** 这一行为文件补充了实现细节或声明式结构。
- **EN L283:** This line contributes implementation detail or declarative structure to the file.
  **CN L283:** 这一行为文件补充了实现细节或声明式结构。
- **EN L284:** Blank line used to separate nearby declarations and improve readability.
  **CN L284:** 该空行用于分隔相邻声明并提升可读性。
- **EN L285:** This line contributes implementation detail or declarative structure to the file.
  **CN L285:** 这一行为文件补充了实现细节或声明式结构。
- **EN L286:** This line contributes implementation detail or declarative structure to the file.
  **CN L286:** 这一行为文件补充了实现细节或声明式结构。
- **EN L287:** This line contributes to the declaration or call of `results`.
  **CN L287:** 这一行为 `results` 的声明或调用提供内容。
- **EN L288:** This line contributes implementation detail or declarative structure to the file.
  **CN L288:** 这一行为文件补充了实现细节或声明式结构。

### Lines 289-300 / 第 289-300 行

```tablegen
 289: 
 290:     Here, foo and bar are required single results, baz is an optional result,
 291:     and qux is a variadic result.
 292: 
 293:     When more than one result is marked as optional or variadic, the operation
 294:     will expect a 'resultSegmentSizes' attribute that defines the number of
 295:     results in each segment.
 296:   }];
 297: 
 298:   let arguments = (ins Variadic<IRDL_AttributeType>:$args,
 299:                         StrArrayAttr:$names,
 300:                         VariadicityArrayAttr:$variadicity);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L289:** Blank line used to separate nearby declarations and improve readability.
  **CN L289:** 该空行用于分隔相邻声明并提升可读性。
- **EN L290:** This line contributes implementation detail or declarative structure to the file.
  **CN L290:** 这一行为文件补充了实现细节或声明式结构。
- **EN L291:** This line contributes implementation detail or declarative structure to the file.
  **CN L291:** 这一行为文件补充了实现细节或声明式结构。
- **EN L292:** Blank line used to separate nearby declarations and improve readability.
  **CN L292:** 该空行用于分隔相邻声明并提升可读性。
- **EN L293:** This line contributes implementation detail or declarative structure to the file.
  **CN L293:** 这一行为文件补充了实现细节或声明式结构。
- **EN L294:** This line contributes implementation detail or declarative structure to the file.
  **CN L294:** 这一行为文件补充了实现细节或声明式结构。
- **EN L295:** This line contributes implementation detail or declarative structure to the file.
  **CN L295:** 这一行为文件补充了实现细节或声明式结构。
- **EN L296:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L296:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L297:** Blank line used to separate nearby declarations and improve readability.
  **CN L297:** 该空行用于分隔相邻声明并提升可读性。
- **EN L298:** This line contributes implementation detail or declarative structure to the file.
  **CN L298:** 这一行为文件补充了实现细节或声明式结构。
- **EN L299:** This line contributes implementation detail or declarative structure to the file.
  **CN L299:** 这一行为文件补充了实现细节或声明式结构。
- **EN L300:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L300:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 301-312 / 第 301-312 行

```tablegen
 301:   let assemblyFormat =
 302:     " `` custom<NamedValueListWithVariadicity>($args, $names, $variadicity) attr-dict";
 303:   let hasVerifier = true;
 304: }
 305: 
 306: def IRDL_AttributesOp : IRDL_Op<"attributes", [HasParent<"OperationOp">]> {
 307:   let summary = "Define the attributes of an operation";
 308: 
 309:   let description = [{
 310:     `irdl.attributes` defines the attributes of the `irdl.operation` parent
 311:     operation definition.
 312: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L301:** This line contributes implementation detail or declarative structure to the file.
  **CN L301:** 这一行为文件补充了实现细节或声明式结构。
- **EN L302:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L302:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L303:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L303:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L304:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L304:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L305:** Blank line used to separate nearby declarations and improve readability.
  **CN L305:** 该空行用于分隔相邻声明并提升可读性。
- **EN L306:** This TableGen `def` record introduces `IRDL_AttributesOp`, which later participates in generated MLIR code.
  **CN L306:** 该 TableGen `def` 记录引入了 `IRDL_AttributesOp`，后续会参与生成的 MLIR 代码。
- **EN L307:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L307:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L308:** Blank line used to separate nearby declarations and improve readability.
  **CN L308:** 该空行用于分隔相邻声明并提升可读性。
- **EN L309:** This line contributes implementation detail or declarative structure to the file.
  **CN L309:** 这一行为文件补充了实现细节或声明式结构。
- **EN L310:** This line contributes implementation detail or declarative structure to the file.
  **CN L310:** 这一行为文件补充了实现细节或声明式结构。
- **EN L311:** This line contributes implementation detail or declarative structure to the file.
  **CN L311:** 这一行为文件补充了实现细节或声明式结构。
- **EN L312:** Blank line used to separate nearby declarations and improve readability.
  **CN L312:** 该空行用于分隔相邻声明并提升可读性。

### Lines 313-324 / 第 313-324 行

```tablegen
 313:     In the following example, `irdl.attributes` defines the attributes of the
 314:     `attr_op` operation:
 315: 
 316:     ```mlir
 317:     irdl.dialect @example {
 318: 
 319:       irdl.operation @attr_op {
 320:         %0 = irdl.any
 321:         %1 = irdl.is i64
 322:         irdl.attibutes {
 323:           "attr1" = %0,
 324:           "attr2" = %1
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L313:** This line contributes implementation detail or declarative structure to the file.
  **CN L313:** 这一行为文件补充了实现细节或声明式结构。
- **EN L314:** This line contributes implementation detail or declarative structure to the file.
  **CN L314:** 这一行为文件补充了实现细节或声明式结构。
- **EN L315:** Blank line used to separate nearby declarations and improve readability.
  **CN L315:** 该空行用于分隔相邻声明并提升可读性。
- **EN L316:** This line contributes implementation detail or declarative structure to the file.
  **CN L316:** 这一行为文件补充了实现细节或声明式结构。
- **EN L317:** This line contributes implementation detail or declarative structure to the file.
  **CN L317:** 这一行为文件补充了实现细节或声明式结构。
- **EN L318:** Blank line used to separate nearby declarations and improve readability.
  **CN L318:** 该空行用于分隔相邻声明并提升可读性。
- **EN L319:** This line contributes implementation detail or declarative structure to the file.
  **CN L319:** 这一行为文件补充了实现细节或声明式结构。
- **EN L320:** This line contributes implementation detail or declarative structure to the file.
  **CN L320:** 这一行为文件补充了实现细节或声明式结构。
- **EN L321:** This line contributes implementation detail or declarative structure to the file.
  **CN L321:** 这一行为文件补充了实现细节或声明式结构。
- **EN L322:** This line contributes implementation detail or declarative structure to the file.
  **CN L322:** 这一行为文件补充了实现细节或声明式结构。
- **EN L323:** This line contributes implementation detail or declarative structure to the file.
  **CN L323:** 这一行为文件补充了实现细节或声明式结构。
- **EN L324:** This line contributes implementation detail or declarative structure to the file.
  **CN L324:** 这一行为文件补充了实现细节或声明式结构。

### Lines 325-336 / 第 325-336 行

```tablegen
 325:         }
 326:       }
 327:     }
 328:     ```
 329: 
 330:     The operation will expect an arbitrary attribute "attr1" and an
 331:     attribute "attr2" with value `i64`.
 332:   }];
 333: 
 334:   let arguments = (ins Variadic<IRDL_AttributeType>:$attributeValues,
 335:                         StrArrayAttr:$attributeValueNames);
 336:   let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L325:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L325:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L326:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L326:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L327:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L327:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L328:** This line contributes implementation detail or declarative structure to the file.
  **CN L328:** 这一行为文件补充了实现细节或声明式结构。
- **EN L329:** Blank line used to separate nearby declarations and improve readability.
  **CN L329:** 该空行用于分隔相邻声明并提升可读性。
- **EN L330:** This line contributes implementation detail or declarative structure to the file.
  **CN L330:** 这一行为文件补充了实现细节或声明式结构。
- **EN L331:** This line contributes implementation detail or declarative structure to the file.
  **CN L331:** 这一行为文件补充了实现细节或声明式结构。
- **EN L332:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L332:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L333:** Blank line used to separate nearby declarations and improve readability.
  **CN L333:** 该空行用于分隔相邻声明并提升可读性。
- **EN L334:** This line contributes implementation detail or declarative structure to the file.
  **CN L334:** 这一行为文件补充了实现细节或声明式结构。
- **EN L335:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L335:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L336:** This line contributes implementation detail or declarative structure to the file.
  **CN L336:** 这一行为文件补充了实现细节或声明式结构。

### Lines 337-348 / 第 337-348 行

```tablegen
 337:     custom<AttributesOp>($attributeValues, $attributeValueNames) attr-dict
 338:   }];
 339: 
 340:   let hasVerifier = true;
 341: }
 342: 
 343: def IRDL_RegionOp : IRDL_Op<"region",
 344:     [HasParent<"OperationOp">, VerifyRegionInterface,
 345:     DeclareOpInterfaceMethods<VerifyRegionInterface>]> {
 346:   let summary = "Define a region of an operation";
 347:   let description = [{
 348:     The irdl.region construct defines a set of characteristics
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L337:** This line contributes implementation detail or declarative structure to the file.
  **CN L337:** 这一行为文件补充了实现细节或声明式结构。
- **EN L338:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L338:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L339:** Blank line used to separate nearby declarations and improve readability.
  **CN L339:** 该空行用于分隔相邻声明并提升可读性。
- **EN L340:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L340:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L341:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L341:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L342:** Blank line used to separate nearby declarations and improve readability.
  **CN L342:** 该空行用于分隔相邻声明并提升可读性。
- **EN L343:** This TableGen `def` record introduces `IRDL_RegionOp`, which later participates in generated MLIR code.
  **CN L343:** 该 TableGen `def` 记录引入了 `IRDL_RegionOp`，后续会参与生成的 MLIR 代码。
- **EN L344:** This line contributes implementation detail or declarative structure to the file.
  **CN L344:** 这一行为文件补充了实现细节或声明式结构。
- **EN L345:** This line contributes implementation detail or declarative structure to the file.
  **CN L345:** 这一行为文件补充了实现细节或声明式结构。
- **EN L346:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L346:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L347:** This line contributes implementation detail or declarative structure to the file.
  **CN L347:** 这一行为文件补充了实现细节或声明式结构。
- **EN L348:** This line contributes implementation detail or declarative structure to the file.
  **CN L348:** 这一行为文件补充了实现细节或声明式结构。

### Lines 349-360 / 第 349-360 行

```tablegen
 349:     that a region of an operation should satify. Each region is named after
 350:     an identifier.
 351: 
 352:     These characteristics include constraints for the entry block arguments
 353:     of the region and the total number of blocks it contains.
 354:     The number of blocks must be a non-zero and non-negative integer,
 355:     and it is optional by default.
 356:     The set of constraints for the entry block arguments may be optional or
 357:     empty. If no parentheses are provided, the set is assumed to be optional,
 358:     and the arguments are not constrained in any way. If parentheses are
 359:     provided with no arguments, it means that the region must have
 360:     no entry block arguments
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L349:** This line contributes implementation detail or declarative structure to the file.
  **CN L349:** 这一行为文件补充了实现细节或声明式结构。
- **EN L350:** This line contributes implementation detail or declarative structure to the file.
  **CN L350:** 这一行为文件补充了实现细节或声明式结构。
- **EN L351:** Blank line used to separate nearby declarations and improve readability.
  **CN L351:** 该空行用于分隔相邻声明并提升可读性。
- **EN L352:** This line contributes implementation detail or declarative structure to the file.
  **CN L352:** 这一行为文件补充了实现细节或声明式结构。
- **EN L353:** This line contributes implementation detail or declarative structure to the file.
  **CN L353:** 这一行为文件补充了实现细节或声明式结构。
- **EN L354:** This line contributes implementation detail or declarative structure to the file.
  **CN L354:** 这一行为文件补充了实现细节或声明式结构。
- **EN L355:** This line contributes implementation detail or declarative structure to the file.
  **CN L355:** 这一行为文件补充了实现细节或声明式结构。
- **EN L356:** This line contributes implementation detail or declarative structure to the file.
  **CN L356:** 这一行为文件补充了实现细节或声明式结构。
- **EN L357:** This line contributes implementation detail or declarative structure to the file.
  **CN L357:** 这一行为文件补充了实现细节或声明式结构。
- **EN L358:** This line contributes implementation detail or declarative structure to the file.
  **CN L358:** 这一行为文件补充了实现细节或声明式结构。
- **EN L359:** This line contributes implementation detail or declarative structure to the file.
  **CN L359:** 这一行为文件补充了实现细节或声明式结构。
- **EN L360:** This line contributes implementation detail or declarative structure to the file.
  **CN L360:** 这一行为文件补充了实现细节或声明式结构。

### Lines 361-372 / 第 361-372 行

```tablegen
 361: 
 362: 
 363:     Example:
 364: 
 365:     ```mlir
 366:     irdl.dialect @example {
 367:       irdl.operation @op_with_regions {
 368:           %r0 = irdl.region
 369:           %r1 = irdl.region()
 370:           %v0 = irdl.is i32
 371:           %v1 = irdl.is i64
 372:           %r2 = irdl.region(%v0, %v1)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L361:** Blank line used to separate nearby declarations and improve readability.
  **CN L361:** 该空行用于分隔相邻声明并提升可读性。
- **EN L362:** Blank line used to separate nearby declarations and improve readability.
  **CN L362:** 该空行用于分隔相邻声明并提升可读性。
- **EN L363:** This line contributes implementation detail or declarative structure to the file.
  **CN L363:** 这一行为文件补充了实现细节或声明式结构。
- **EN L364:** Blank line used to separate nearby declarations and improve readability.
  **CN L364:** 该空行用于分隔相邻声明并提升可读性。
- **EN L365:** This line contributes implementation detail or declarative structure to the file.
  **CN L365:** 这一行为文件补充了实现细节或声明式结构。
- **EN L366:** This line contributes implementation detail or declarative structure to the file.
  **CN L366:** 这一行为文件补充了实现细节或声明式结构。
- **EN L367:** This line contributes implementation detail or declarative structure to the file.
  **CN L367:** 这一行为文件补充了实现细节或声明式结构。
- **EN L368:** This line contributes implementation detail or declarative structure to the file.
  **CN L368:** 这一行为文件补充了实现细节或声明式结构。
- **EN L369:** This line contributes to the declaration or call of `region`.
  **CN L369:** 这一行为 `region` 的声明或调用提供内容。
- **EN L370:** This line contributes implementation detail or declarative structure to the file.
  **CN L370:** 这一行为文件补充了实现细节或声明式结构。
- **EN L371:** This line contributes implementation detail or declarative structure to the file.
  **CN L371:** 这一行为文件补充了实现细节或声明式结构。
- **EN L372:** This line contributes to the declaration or call of `region`.
  **CN L372:** 这一行为 `region` 的声明或调用提供内容。

### Lines 373-384 / 第 373-384 行

```tablegen
 373:           %r3 = irdl.region with size 3
 374: 
 375:           irdl.regions(foo: %r0, bar: %r1, baz: %r2, qux: %r3)
 376:       }
 377:     }
 378:     ```
 379: 
 380:     The above snippet demonstrates an operation named `@op_with_regions`,
 381:     which is constrained to have four regions.
 382: 
 383:     * Region `foo` doesn't have any constraints on the arguments
 384:       or the number of blocks.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L373:** This line contributes implementation detail or declarative structure to the file.
  **CN L373:** 这一行为文件补充了实现细节或声明式结构。
- **EN L374:** Blank line used to separate nearby declarations and improve readability.
  **CN L374:** 该空行用于分隔相邻声明并提升可读性。
- **EN L375:** This line contributes to the declaration or call of `regions`.
  **CN L375:** 这一行为 `regions` 的声明或调用提供内容。
- **EN L376:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L376:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L377:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L377:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L378:** This line contributes implementation detail or declarative structure to the file.
  **CN L378:** 这一行为文件补充了实现细节或声明式结构。
- **EN L379:** Blank line used to separate nearby declarations and improve readability.
  **CN L379:** 该空行用于分隔相邻声明并提升可读性。
- **EN L380:** This line contributes implementation detail or declarative structure to the file.
  **CN L380:** 这一行为文件补充了实现细节或声明式结构。
- **EN L381:** This line contributes implementation detail or declarative structure to the file.
  **CN L381:** 这一行为文件补充了实现细节或声明式结构。
- **EN L382:** Blank line used to separate nearby declarations and improve readability.
  **CN L382:** 该空行用于分隔相邻声明并提升可读性。
- **EN L383:** This comment states: “Region `foo` doesn't have any constraints on the arguments”, documenting the intent of the surrounding code.
  **CN L383:** 该注释写道：“Region `foo` doesn't have any constraints on the arguments”，用于说明周围代码的意图。
- **EN L384:** This line contributes implementation detail or declarative structure to the file.
  **CN L384:** 这一行为文件补充了实现细节或声明式结构。

### Lines 385-396 / 第 385-396 行

```tablegen
 385:     * Region `bar` should have an empty set of arguments.
 386:     * Region `baz` should have two arguments of types `i32` and `i64`.
 387:     * Region `qux` should contain exactly three blocks.
 388:   }];
 389:   let arguments = (ins Variadic<IRDL_AttributeType>:$entryBlockArgs,
 390:                     OptionalAttr<I32Attr>:$numberOfBlocks,
 391:                     UnitAttr:$constrainedArguments);
 392:   let results = (outs IRDL_RegionType:$output);
 393: 
 394:   let assemblyFormat = [{
 395:     ``(`(` $entryBlockArgs $constrainedArguments^ `)`)?
 396:     ``(` ` `with` `size` $numberOfBlocks^)? attr-dict
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L385:** This comment states: “Region `bar` should have an empty set of arguments.”, documenting the intent of the surrounding code.
  **CN L385:** 该注释写道：“Region `bar` should have an empty set of arguments.”，用于说明周围代码的意图。
- **EN L386:** This comment states: “Region `baz` should have two arguments of types `i32` and `i64`.”, documenting the intent of the surrounding code.
  **CN L386:** 该注释写道：“Region `baz` should have two arguments of types `i32` and `i64`.”，用于说明周围代码的意图。
- **EN L387:** This comment states: “Region `qux` should contain exactly three blocks.”, documenting the intent of the surrounding code.
  **CN L387:** 该注释写道：“Region `qux` should contain exactly three blocks.”，用于说明周围代码的意图。
- **EN L388:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L388:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L389:** This line contributes implementation detail or declarative structure to the file.
  **CN L389:** 这一行为文件补充了实现细节或声明式结构。
- **EN L390:** This line contributes implementation detail or declarative structure to the file.
  **CN L390:** 这一行为文件补充了实现细节或声明式结构。
- **EN L391:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L391:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L392:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L392:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L393:** Blank line used to separate nearby declarations and improve readability.
  **CN L393:** 该空行用于分隔相邻声明并提升可读性。
- **EN L394:** This line contributes implementation detail or declarative structure to the file.
  **CN L394:** 这一行为文件补充了实现细节或声明式结构。
- **EN L395:** This line contributes implementation detail or declarative structure to the file.
  **CN L395:** 这一行为文件补充了实现细节或声明式结构。
- **EN L396:** This line contributes implementation detail or declarative structure to the file.
  **CN L396:** 这一行为文件补充了实现细节或声明式结构。

### Lines 397-408 / 第 397-408 行

```tablegen
 397:   }];
 398: 
 399:   let hasVerifier = true;
 400: }
 401: 
 402: def IRDL_RegionsOp : IRDL_Op<"regions", [HasParent<"OperationOp">]> {
 403:   let summary = "Define the regions of an operation";
 404:   let description = [{
 405:     `irdl.regions` defines the regions of an operation by accepting
 406:     values produced by `irdl.region` operation as arguments. Each
 407:     region has an identifier as name.
 408: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L397:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L397:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L398:** Blank line used to separate nearby declarations and improve readability.
  **CN L398:** 该空行用于分隔相邻声明并提升可读性。
- **EN L399:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L399:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L400:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L400:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L401:** Blank line used to separate nearby declarations and improve readability.
  **CN L401:** 该空行用于分隔相邻声明并提升可读性。
- **EN L402:** This TableGen `def` record introduces `IRDL_RegionsOp`, which later participates in generated MLIR code.
  **CN L402:** 该 TableGen `def` 记录引入了 `IRDL_RegionsOp`，后续会参与生成的 MLIR 代码。
- **EN L403:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L403:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L404:** This line contributes implementation detail or declarative structure to the file.
  **CN L404:** 这一行为文件补充了实现细节或声明式结构。
- **EN L405:** This line contributes implementation detail or declarative structure to the file.
  **CN L405:** 这一行为文件补充了实现细节或声明式结构。
- **EN L406:** This line contributes implementation detail or declarative structure to the file.
  **CN L406:** 这一行为文件补充了实现细节或声明式结构。
- **EN L407:** This line contributes implementation detail or declarative structure to the file.
  **CN L407:** 这一行为文件补充了实现细节或声明式结构。
- **EN L408:** Blank line used to separate nearby declarations and improve readability.
  **CN L408:** 该空行用于分隔相邻声明并提升可读性。

### Lines 409-420 / 第 409-420 行

```tablegen
 409:     Example:
 410: 
 411:     ```mlir
 412:     irdl.dialect @example {
 413:       irdl.operation @op_with_regions {
 414:         %r1 = irdl.region with size 3
 415:         %0 = irdl.any
 416:         %r2 = irdl.region(%0)
 417:         irdl.regions(foo: %r1, bar: %r2)
 418:       }
 419:     }
 420:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L409:** This line contributes implementation detail or declarative structure to the file.
  **CN L409:** 这一行为文件补充了实现细节或声明式结构。
- **EN L410:** Blank line used to separate nearby declarations and improve readability.
  **CN L410:** 该空行用于分隔相邻声明并提升可读性。
- **EN L411:** This line contributes implementation detail or declarative structure to the file.
  **CN L411:** 这一行为文件补充了实现细节或声明式结构。
- **EN L412:** This line contributes implementation detail or declarative structure to the file.
  **CN L412:** 这一行为文件补充了实现细节或声明式结构。
- **EN L413:** This line contributes implementation detail or declarative structure to the file.
  **CN L413:** 这一行为文件补充了实现细节或声明式结构。
- **EN L414:** This line contributes implementation detail or declarative structure to the file.
  **CN L414:** 这一行为文件补充了实现细节或声明式结构。
- **EN L415:** This line contributes implementation detail or declarative structure to the file.
  **CN L415:** 这一行为文件补充了实现细节或声明式结构。
- **EN L416:** This line contributes to the declaration or call of `region`.
  **CN L416:** 这一行为 `region` 的声明或调用提供内容。
- **EN L417:** This line contributes to the declaration or call of `regions`.
  **CN L417:** 这一行为 `regions` 的声明或调用提供内容。
- **EN L418:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L418:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L419:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L419:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L420:** This line contributes implementation detail or declarative structure to the file.
  **CN L420:** 这一行为文件补充了实现细节或声明式结构。

### Lines 421-432 / 第 421-432 行

```tablegen
 421: 
 422:     In the snippet above the operation is constrained to have two regions.
 423:     The first region (`foo`) should contain three blocks.
 424:     The second region (`bar`) should have one region with one argument.
 425:   }];
 426: 
 427:   let arguments = (ins Variadic<IRDL_RegionType>:$args, StrArrayAttr:$names);
 428:   let assemblyFormat = " `` custom<NamedValueList>($args, $names) attr-dict ";
 429:   let hasVerifier = true;
 430: }
 431: 
 432: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L421:** Blank line used to separate nearby declarations and improve readability.
  **CN L421:** 该空行用于分隔相邻声明并提升可读性。
- **EN L422:** This line contributes implementation detail or declarative structure to the file.
  **CN L422:** 这一行为文件补充了实现细节或声明式结构。
- **EN L423:** This line contributes to the declaration or call of `region`.
  **CN L423:** 这一行为 `region` 的声明或调用提供内容。
- **EN L424:** This line contributes to the declaration or call of `region`.
  **CN L424:** 这一行为 `region` 的声明或调用提供内容。
- **EN L425:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L425:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L426:** Blank line used to separate nearby declarations and improve readability.
  **CN L426:** 该空行用于分隔相邻声明并提升可读性。
- **EN L427:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L427:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L428:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L428:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L429:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L429:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L430:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L430:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L431:** Blank line used to separate nearby declarations and improve readability.
  **CN L431:** 该空行用于分隔相邻声明并提升可读性。
- **EN L432:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L432:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 433-444 / 第 433-444 行

```tablegen
 433: // IRDL Constraint operations
 434: //===----------------------------------------------------------------------===//
 435: 
 436: class IRDL_ConstraintOp<string mnemonic, list<Trait> traits = []>
 437:     : IRDL_Op<mnemonic, [VerifyConstraintInterface,
 438:         DeclareOpInterfaceMethods<VerifyConstraintInterface>] # traits> {
 439: }
 440: 
 441: def IRDL_IsOp : IRDL_ConstraintOp<"is",
 442:     [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>, Pure]> {
 443:   let summary = "Constraints an attribute/type to be a specific attribute instance";
 444:   let description = [{
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L433:** This comment states: “IRDL Constraint operations”, documenting the intent of the surrounding code.
  **CN L433:** 该注释写道：“IRDL Constraint operations”，用于说明周围代码的意图。
- **EN L434:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L434:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L435:** Blank line used to separate nearby declarations and improve readability.
  **CN L435:** 该空行用于分隔相邻声明并提升可读性。
- **EN L436:** This TableGen `class` record introduces `IRDL_ConstraintOp`, which later participates in generated MLIR code.
  **CN L436:** 该 TableGen `class` 记录引入了 `IRDL_ConstraintOp`，后续会参与生成的 MLIR 代码。
- **EN L437:** This line contributes implementation detail or declarative structure to the file.
  **CN L437:** 这一行为文件补充了实现细节或声明式结构。
- **EN L438:** This line contributes implementation detail or declarative structure to the file.
  **CN L438:** 这一行为文件补充了实现细节或声明式结构。
- **EN L439:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L439:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L440:** Blank line used to separate nearby declarations and improve readability.
  **CN L440:** 该空行用于分隔相邻声明并提升可读性。
- **EN L441:** This TableGen `def` record introduces `IRDL_IsOp`, which later participates in generated MLIR code.
  **CN L441:** 该 TableGen `def` 记录引入了 `IRDL_IsOp`，后续会参与生成的 MLIR 代码。
- **EN L442:** This line contributes implementation detail or declarative structure to the file.
  **CN L442:** 这一行为文件补充了实现细节或声明式结构。
- **EN L443:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L443:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L444:** This line contributes implementation detail or declarative structure to the file.
  **CN L444:** 这一行为文件补充了实现细节或声明式结构。

### Lines 445-456 / 第 445-456 行

```tablegen
 445:     `irdl.is` defines a constraint that only accepts a specific instance of a
 446:     type or attribute.
 447: 
 448:     Example:
 449: 
 450:     ```mlir
 451:     irdl.dialect @cmath {
 452:       irdl.type @complex_i32 {
 453:         %0 = irdl.is i32
 454:         irdl.parameters(%0)
 455:       }
 456:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L445:** This line contributes implementation detail or declarative structure to the file.
  **CN L445:** 这一行为文件补充了实现细节或声明式结构。
- **EN L446:** This line contributes implementation detail or declarative structure to the file.
  **CN L446:** 这一行为文件补充了实现细节或声明式结构。
- **EN L447:** Blank line used to separate nearby declarations and improve readability.
  **CN L447:** 该空行用于分隔相邻声明并提升可读性。
- **EN L448:** This line contributes implementation detail or declarative structure to the file.
  **CN L448:** 这一行为文件补充了实现细节或声明式结构。
- **EN L449:** Blank line used to separate nearby declarations and improve readability.
  **CN L449:** 该空行用于分隔相邻声明并提升可读性。
- **EN L450:** This line contributes implementation detail or declarative structure to the file.
  **CN L450:** 这一行为文件补充了实现细节或声明式结构。
- **EN L451:** This line contributes implementation detail or declarative structure to the file.
  **CN L451:** 这一行为文件补充了实现细节或声明式结构。
- **EN L452:** This line contributes implementation detail or declarative structure to the file.
  **CN L452:** 这一行为文件补充了实现细节或声明式结构。
- **EN L453:** This line contributes implementation detail or declarative structure to the file.
  **CN L453:** 这一行为文件补充了实现细节或声明式结构。
- **EN L454:** This line contributes to the declaration or call of `parameters`.
  **CN L454:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L455:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L455:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L456:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L456:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 457-468 / 第 457-468 行

```tablegen
 457:     ```
 458: 
 459:     The above program defines a `complex_i32` type inside the dialect `cmath`
 460:     that can only have a `i32` as its parameter.
 461:   }];
 462: 
 463:   let arguments = (ins AnyAttr:$expected);
 464:   let results = (outs IRDL_AttributeType:$output);
 465:   let assemblyFormat = " $expected ` ` attr-dict ";
 466: }
 467: 
 468: def IRDL_BaseOp : IRDL_ConstraintOp<"base",
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L457:** This line contributes implementation detail or declarative structure to the file.
  **CN L457:** 这一行为文件补充了实现细节或声明式结构。
- **EN L458:** Blank line used to separate nearby declarations and improve readability.
  **CN L458:** 该空行用于分隔相邻声明并提升可读性。
- **EN L459:** This line contributes implementation detail or declarative structure to the file.
  **CN L459:** 这一行为文件补充了实现细节或声明式结构。
- **EN L460:** This line contributes implementation detail or declarative structure to the file.
  **CN L460:** 这一行为文件补充了实现细节或声明式结构。
- **EN L461:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L461:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L462:** Blank line used to separate nearby declarations and improve readability.
  **CN L462:** 该空行用于分隔相邻声明并提升可读性。
- **EN L463:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L463:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L464:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L464:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L465:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L465:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L466:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L466:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L467:** Blank line used to separate nearby declarations and improve readability.
  **CN L467:** 该空行用于分隔相邻声明并提升可读性。
- **EN L468:** This TableGen `def` record introduces `IRDL_BaseOp`, which later participates in generated MLIR code.
  **CN L468:** 该 TableGen `def` 记录引入了 `IRDL_BaseOp`，后续会参与生成的 MLIR 代码。

### Lines 469-480 / 第 469-480 行

```tablegen
 469:     [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>,
 470:      DeclareOpInterfaceMethods<SymbolUserOpInterface>]> {
 471:   let summary = "Constraints an attribute/type base";
 472:   let description = [{
 473:     `irdl.base` defines a constraint that only accepts a single type
 474:     or attribute base, e.g. an `IntegerType`. The attribute base is defined
 475:     either by a symbolic reference to the corresponding IRDL definition,
 476:     or by the name of the base. Named bases are prefixed with `!` or `#`
 477:     respectively for types and attributes.
 478: 
 479:     Example:
 480: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L469:** This line contributes implementation detail or declarative structure to the file.
  **CN L469:** 这一行为文件补充了实现细节或声明式结构。
- **EN L470:** This line contributes implementation detail or declarative structure to the file.
  **CN L470:** 这一行为文件补充了实现细节或声明式结构。
- **EN L471:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L471:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L472:** This line contributes implementation detail or declarative structure to the file.
  **CN L472:** 这一行为文件补充了实现细节或声明式结构。
- **EN L473:** This line contributes implementation detail or declarative structure to the file.
  **CN L473:** 这一行为文件补充了实现细节或声明式结构。
- **EN L474:** This line contributes implementation detail or declarative structure to the file.
  **CN L474:** 这一行为文件补充了实现细节或声明式结构。
- **EN L475:** This line contributes implementation detail or declarative structure to the file.
  **CN L475:** 这一行为文件补充了实现细节或声明式结构。
- **EN L476:** This line contributes implementation detail or declarative structure to the file.
  **CN L476:** 这一行为文件补充了实现细节或声明式结构。
- **EN L477:** This line contributes implementation detail or declarative structure to the file.
  **CN L477:** 这一行为文件补充了实现细节或声明式结构。
- **EN L478:** Blank line used to separate nearby declarations and improve readability.
  **CN L478:** 该空行用于分隔相邻声明并提升可读性。
- **EN L479:** This line contributes implementation detail or declarative structure to the file.
  **CN L479:** 这一行为文件补充了实现细节或声明式结构。
- **EN L480:** Blank line used to separate nearby declarations and improve readability.
  **CN L480:** 该空行用于分隔相邻声明并提升可读性。

### Lines 481-492 / 第 481-492 行

```tablegen
 481:     ```mlir
 482:     irdl.dialect @cmath {
 483:       irdl.type @complex {
 484:         %0 = irdl.base "!builtin.integer"
 485:         irdl.parameters(%0)
 486:       }
 487: 
 488:       irdl.type @complex_wrapper {
 489:         %0 = irdl.base @cmath::@complex
 490:         irdl.parameters(%0)
 491:       }
 492:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L481:** This line contributes implementation detail or declarative structure to the file.
  **CN L481:** 这一行为文件补充了实现细节或声明式结构。
- **EN L482:** This line contributes implementation detail or declarative structure to the file.
  **CN L482:** 这一行为文件补充了实现细节或声明式结构。
- **EN L483:** This line contributes implementation detail or declarative structure to the file.
  **CN L483:** 这一行为文件补充了实现细节或声明式结构。
- **EN L484:** This line contributes implementation detail or declarative structure to the file.
  **CN L484:** 这一行为文件补充了实现细节或声明式结构。
- **EN L485:** This line contributes to the declaration or call of `parameters`.
  **CN L485:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L486:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L486:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L487:** Blank line used to separate nearby declarations and improve readability.
  **CN L487:** 该空行用于分隔相邻声明并提升可读性。
- **EN L488:** This line contributes implementation detail or declarative structure to the file.
  **CN L488:** 这一行为文件补充了实现细节或声明式结构。
- **EN L489:** This line contributes implementation detail or declarative structure to the file.
  **CN L489:** 这一行为文件补充了实现细节或声明式结构。
- **EN L490:** This line contributes to the declaration or call of `parameters`.
  **CN L490:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L491:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L491:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L492:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L492:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 493-504 / 第 493-504 行

```tablegen
 493:     ```
 494: 
 495:     The above program defines a `cmath.complex` type that expects a single
 496:     parameter, which is a type with base name `builtin.integer`, which is the
 497:     name of an `IntegerType` type.
 498:     It also defines a `cmath.complex_wrapper` type that expects a single
 499:     parameter, which is a type of base type `cmath.complex`.
 500:   }];
 501: 
 502:   let arguments = (ins OptionalAttr<SymbolRefAttr>:$base_ref,
 503:                        OptionalAttr<StrAttr>:$base_name);
 504:   let results = (outs IRDL_AttributeType:$output);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L493:** This line contributes implementation detail or declarative structure to the file.
  **CN L493:** 这一行为文件补充了实现细节或声明式结构。
- **EN L494:** Blank line used to separate nearby declarations and improve readability.
  **CN L494:** 该空行用于分隔相邻声明并提升可读性。
- **EN L495:** This line contributes implementation detail or declarative structure to the file.
  **CN L495:** 这一行为文件补充了实现细节或声明式结构。
- **EN L496:** This line contributes implementation detail or declarative structure to the file.
  **CN L496:** 这一行为文件补充了实现细节或声明式结构。
- **EN L497:** This line contributes implementation detail or declarative structure to the file.
  **CN L497:** 这一行为文件补充了实现细节或声明式结构。
- **EN L498:** This line contributes implementation detail or declarative structure to the file.
  **CN L498:** 这一行为文件补充了实现细节或声明式结构。
- **EN L499:** This line contributes implementation detail or declarative structure to the file.
  **CN L499:** 这一行为文件补充了实现细节或声明式结构。
- **EN L500:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L500:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L501:** Blank line used to separate nearby declarations and improve readability.
  **CN L501:** 该空行用于分隔相邻声明并提升可读性。
- **EN L502:** This line contributes implementation detail or declarative structure to the file.
  **CN L502:** 这一行为文件补充了实现细节或声明式结构。
- **EN L503:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L503:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L504:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L504:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 505-516 / 第 505-516 行

```tablegen
 505:   let assemblyFormat = " ($base_ref^)? ($base_name^)? ` ` attr-dict";
 506: 
 507:   let builders = [
 508:     OpBuilder<(ins "SymbolRefAttr":$base_ref), [{
 509:       build($_builder, $_state, base_ref, {});
 510:     }]>,
 511:     OpBuilder<(ins "StringAttr":$base_name), [{
 512:       build($_builder, $_state, {}, base_name);
 513:     }]>,
 514:   ];
 515: 
 516:   let hasVerifier = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L505:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L505:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L506:** Blank line used to separate nearby declarations and improve readability.
  **CN L506:** 该空行用于分隔相邻声明并提升可读性。
- **EN L507:** This line contributes implementation detail or declarative structure to the file.
  **CN L507:** 这一行为文件补充了实现细节或声明式结构。
- **EN L508:** This line contributes implementation detail or declarative structure to the file.
  **CN L508:** 这一行为文件补充了实现细节或声明式结构。
- **EN L509:** This line contributes to the declaration or call of `build`.
  **CN L509:** 这一行为 `build` 的声明或调用提供内容。
- **EN L510:** This line contributes implementation detail or declarative structure to the file.
  **CN L510:** 这一行为文件补充了实现细节或声明式结构。
- **EN L511:** This line contributes implementation detail or declarative structure to the file.
  **CN L511:** 这一行为文件补充了实现细节或声明式结构。
- **EN L512:** This line contributes to the declaration or call of `build`.
  **CN L512:** 这一行为 `build` 的声明或调用提供内容。
- **EN L513:** This line contributes implementation detail or declarative structure to the file.
  **CN L513:** 这一行为文件补充了实现细节或声明式结构。
- **EN L514:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L514:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L515:** Blank line used to separate nearby declarations and improve readability.
  **CN L515:** 该空行用于分隔相邻声明并提升可读性。
- **EN L516:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L516:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 517-528 / 第 517-528 行

```tablegen
 517: }
 518: 
 519: def IRDL_ParametricOp : IRDL_ConstraintOp<"parametric",
 520:     [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>,
 521:      DeclareOpInterfaceMethods<SymbolUserOpInterface>, Pure]> {
 522:   let summary = "Constraints an attribute/type base and its parameters";
 523:   let description = [{
 524:     `irdl.parametric` defines a constraint that accepts only a single type
 525:     or attribute base. The attribute base is defined by a symbolic reference
 526:     to the corresponding definition. It will additionally constraint the
 527:     parameters of the type/attribute.
 528: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L517:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L517:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L518:** Blank line used to separate nearby declarations and improve readability.
  **CN L518:** 该空行用于分隔相邻声明并提升可读性。
- **EN L519:** This TableGen `def` record introduces `IRDL_ParametricOp`, which later participates in generated MLIR code.
  **CN L519:** 该 TableGen `def` 记录引入了 `IRDL_ParametricOp`，后续会参与生成的 MLIR 代码。
- **EN L520:** This line contributes implementation detail or declarative structure to the file.
  **CN L520:** 这一行为文件补充了实现细节或声明式结构。
- **EN L521:** This line contributes implementation detail or declarative structure to the file.
  **CN L521:** 这一行为文件补充了实现细节或声明式结构。
- **EN L522:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L522:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L523:** This line contributes implementation detail or declarative structure to the file.
  **CN L523:** 这一行为文件补充了实现细节或声明式结构。
- **EN L524:** This line contributes implementation detail or declarative structure to the file.
  **CN L524:** 这一行为文件补充了实现细节或声明式结构。
- **EN L525:** This line contributes implementation detail or declarative structure to the file.
  **CN L525:** 这一行为文件补充了实现细节或声明式结构。
- **EN L526:** This line contributes implementation detail or declarative structure to the file.
  **CN L526:** 这一行为文件补充了实现细节或声明式结构。
- **EN L527:** This line contributes implementation detail or declarative structure to the file.
  **CN L527:** 这一行为文件补充了实现细节或声明式结构。
- **EN L528:** Blank line used to separate nearby declarations and improve readability.
  **CN L528:** 该空行用于分隔相邻声明并提升可读性。

### Lines 529-540 / 第 529-540 行

```tablegen
 529:     Example:
 530: 
 531:     ```mlir
 532:     irdl.dialect @cmath {
 533: 
 534:       irdl.type @complex { /* ... */ }
 535: 
 536:       irdl.operation @norm {
 537:         %0 = irdl.any
 538:         %1 = irdl.parametric @cmath::@complex<%0>
 539:         irdl.operands(%1)
 540:         irdl.results(%0)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L529:** This line contributes implementation detail or declarative structure to the file.
  **CN L529:** 这一行为文件补充了实现细节或声明式结构。
- **EN L530:** Blank line used to separate nearby declarations and improve readability.
  **CN L530:** 该空行用于分隔相邻声明并提升可读性。
- **EN L531:** This line contributes implementation detail or declarative structure to the file.
  **CN L531:** 这一行为文件补充了实现细节或声明式结构。
- **EN L532:** This line contributes implementation detail or declarative structure to the file.
  **CN L532:** 这一行为文件补充了实现细节或声明式结构。
- **EN L533:** Blank line used to separate nearby declarations and improve readability.
  **CN L533:** 该空行用于分隔相邻声明并提升可读性。
- **EN L534:** This line contributes implementation detail or declarative structure to the file.
  **CN L534:** 这一行为文件补充了实现细节或声明式结构。
- **EN L535:** Blank line used to separate nearby declarations and improve readability.
  **CN L535:** 该空行用于分隔相邻声明并提升可读性。
- **EN L536:** This line contributes implementation detail or declarative structure to the file.
  **CN L536:** 这一行为文件补充了实现细节或声明式结构。
- **EN L537:** This line contributes implementation detail or declarative structure to the file.
  **CN L537:** 这一行为文件补充了实现细节或声明式结构。
- **EN L538:** This line contributes implementation detail or declarative structure to the file.
  **CN L538:** 这一行为文件补充了实现细节或声明式结构。
- **EN L539:** This line contributes to the declaration or call of `operands`.
  **CN L539:** 这一行为 `operands` 的声明或调用提供内容。
- **EN L540:** This line contributes to the declaration or call of `results`.
  **CN L540:** 这一行为 `results` 的声明或调用提供内容。

### Lines 541-552 / 第 541-552 行

```tablegen
 541:       }
 542:     }
 543:     ```
 544: 
 545:     The above program defines an operation `norm` inside the dialect `cmath` that
 546:     for any `T` takes a `cmath.complex` with parameter `T` and returns a `T`.
 547:   }];
 548: 
 549:   let arguments = (ins SymbolRefAttr:$base_type,
 550:                        Variadic<IRDL_AttributeType>:$args);
 551:   let results = (outs IRDL_AttributeType:$output);
 552:   let assemblyFormat = " $base_type `<` $args `>` ` ` attr-dict ";
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L541:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L541:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L542:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L542:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L543:** This line contributes implementation detail or declarative structure to the file.
  **CN L543:** 这一行为文件补充了实现细节或声明式结构。
- **EN L544:** Blank line used to separate nearby declarations and improve readability.
  **CN L544:** 该空行用于分隔相邻声明并提升可读性。
- **EN L545:** This line contributes implementation detail or declarative structure to the file.
  **CN L545:** 这一行为文件补充了实现细节或声明式结构。
- **EN L546:** This line contributes implementation detail or declarative structure to the file.
  **CN L546:** 这一行为文件补充了实现细节或声明式结构。
- **EN L547:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L547:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L548:** Blank line used to separate nearby declarations and improve readability.
  **CN L548:** 该空行用于分隔相邻声明并提升可读性。
- **EN L549:** This line contributes implementation detail or declarative structure to the file.
  **CN L549:** 这一行为文件补充了实现细节或声明式结构。
- **EN L550:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L550:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L551:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L551:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L552:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L552:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 553-564 / 第 553-564 行

```tablegen
 553: }
 554: 
 555: def IRDL_AnyOp : IRDL_ConstraintOp<"any",
 556:     [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>]> {
 557:   let summary = "Accept any type or attribute";
 558:   let description = [{
 559:     `irdl.any` defines a constraint that accepts any type or attribute.
 560: 
 561:     Example:
 562: 
 563:     ```mlir
 564:     irdl.dialect @cmath {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L553:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L553:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L554:** Blank line used to separate nearby declarations and improve readability.
  **CN L554:** 该空行用于分隔相邻声明并提升可读性。
- **EN L555:** This TableGen `def` record introduces `IRDL_AnyOp`, which later participates in generated MLIR code.
  **CN L555:** 该 TableGen `def` 记录引入了 `IRDL_AnyOp`，后续会参与生成的 MLIR 代码。
- **EN L556:** This line contributes implementation detail or declarative structure to the file.
  **CN L556:** 这一行为文件补充了实现细节或声明式结构。
- **EN L557:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L557:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L558:** This line contributes implementation detail or declarative structure to the file.
  **CN L558:** 这一行为文件补充了实现细节或声明式结构。
- **EN L559:** This line contributes implementation detail or declarative structure to the file.
  **CN L559:** 这一行为文件补充了实现细节或声明式结构。
- **EN L560:** Blank line used to separate nearby declarations and improve readability.
  **CN L560:** 该空行用于分隔相邻声明并提升可读性。
- **EN L561:** This line contributes implementation detail or declarative structure to the file.
  **CN L561:** 这一行为文件补充了实现细节或声明式结构。
- **EN L562:** Blank line used to separate nearby declarations and improve readability.
  **CN L562:** 该空行用于分隔相邻声明并提升可读性。
- **EN L563:** This line contributes implementation detail or declarative structure to the file.
  **CN L563:** 这一行为文件补充了实现细节或声明式结构。
- **EN L564:** This line contributes implementation detail or declarative structure to the file.
  **CN L564:** 这一行为文件补充了实现细节或声明式结构。

### Lines 565-576 / 第 565-576 行

```tablegen
 565:       irdl.type @complex_flexible {
 566:         %0 = irdl.any
 567:         irdl.parameters(%0)
 568:       }
 569:     }
 570:     ```
 571: 
 572:     The above program defines a type `complex_flexible` inside the dialect
 573:     `cmath` that has a single parameter that can be any attribute.
 574:   }];
 575: 
 576:   let results = (outs IRDL_AttributeType:$output);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L565:** This line contributes implementation detail or declarative structure to the file.
  **CN L565:** 这一行为文件补充了实现细节或声明式结构。
- **EN L566:** This line contributes implementation detail or declarative structure to the file.
  **CN L566:** 这一行为文件补充了实现细节或声明式结构。
- **EN L567:** This line contributes to the declaration or call of `parameters`.
  **CN L567:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L568:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L568:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L569:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L569:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L570:** This line contributes implementation detail or declarative structure to the file.
  **CN L570:** 这一行为文件补充了实现细节或声明式结构。
- **EN L571:** Blank line used to separate nearby declarations and improve readability.
  **CN L571:** 该空行用于分隔相邻声明并提升可读性。
- **EN L572:** This line contributes implementation detail or declarative structure to the file.
  **CN L572:** 这一行为文件补充了实现细节或声明式结构。
- **EN L573:** This line contributes implementation detail or declarative structure to the file.
  **CN L573:** 这一行为文件补充了实现细节或声明式结构。
- **EN L574:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L574:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L575:** Blank line used to separate nearby declarations and improve readability.
  **CN L575:** 该空行用于分隔相邻声明并提升可读性。
- **EN L576:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L576:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 577-588 / 第 577-588 行

```tablegen
 577:   let assemblyFormat = " attr-dict ";
 578: }
 579: 
 580: def IRDL_AnyOfOp : IRDL_ConstraintOp<"any_of",
 581:                   [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>,
 582:                    SameOperandsAndResultType]> {
 583:   let summary = "Constraints to the union of the provided constraints";
 584:   let description = [{
 585:     `irdl.any_of` defines a constraint that accepts any type or attribute that
 586:     satisfies at least one of its provided type constraints.
 587: 
 588:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L577:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L577:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L578:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L578:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L579:** Blank line used to separate nearby declarations and improve readability.
  **CN L579:** 该空行用于分隔相邻声明并提升可读性。
- **EN L580:** This TableGen `def` record introduces `IRDL_AnyOfOp`, which later participates in generated MLIR code.
  **CN L580:** 该 TableGen `def` 记录引入了 `IRDL_AnyOfOp`，后续会参与生成的 MLIR 代码。
- **EN L581:** This line contributes implementation detail or declarative structure to the file.
  **CN L581:** 这一行为文件补充了实现细节或声明式结构。
- **EN L582:** This line contributes implementation detail or declarative structure to the file.
  **CN L582:** 这一行为文件补充了实现细节或声明式结构。
- **EN L583:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L583:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L584:** This line contributes implementation detail or declarative structure to the file.
  **CN L584:** 这一行为文件补充了实现细节或声明式结构。
- **EN L585:** This line contributes implementation detail or declarative structure to the file.
  **CN L585:** 这一行为文件补充了实现细节或声明式结构。
- **EN L586:** This line contributes implementation detail or declarative structure to the file.
  **CN L586:** 这一行为文件补充了实现细节或声明式结构。
- **EN L587:** Blank line used to separate nearby declarations and improve readability.
  **CN L587:** 该空行用于分隔相邻声明并提升可读性。
- **EN L588:** This line contributes implementation detail or declarative structure to the file.
  **CN L588:** 这一行为文件补充了实现细节或声明式结构。

### Lines 589-600 / 第 589-600 行

```tablegen
 589: 
 590:     ```mlir
 591:     irdl.dialect @cmath {
 592:       irdl.type @complex {
 593:         %0 = irdl.is i32
 594:         %1 = irdl.is i64
 595:         %2 = irdl.is f32
 596:         %3 = irdl.is f64
 597:         %4 = irdl.any_of(%0, %1, %2, %3)
 598:         irdl.parameters(%4)
 599:       }
 600:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L589:** Blank line used to separate nearby declarations and improve readability.
  **CN L589:** 该空行用于分隔相邻声明并提升可读性。
- **EN L590:** This line contributes implementation detail or declarative structure to the file.
  **CN L590:** 这一行为文件补充了实现细节或声明式结构。
- **EN L591:** This line contributes implementation detail or declarative structure to the file.
  **CN L591:** 这一行为文件补充了实现细节或声明式结构。
- **EN L592:** This line contributes implementation detail or declarative structure to the file.
  **CN L592:** 这一行为文件补充了实现细节或声明式结构。
- **EN L593:** This line contributes implementation detail or declarative structure to the file.
  **CN L593:** 这一行为文件补充了实现细节或声明式结构。
- **EN L594:** This line contributes implementation detail or declarative structure to the file.
  **CN L594:** 这一行为文件补充了实现细节或声明式结构。
- **EN L595:** This line contributes implementation detail or declarative structure to the file.
  **CN L595:** 这一行为文件补充了实现细节或声明式结构。
- **EN L596:** This line contributes implementation detail or declarative structure to the file.
  **CN L596:** 这一行为文件补充了实现细节或声明式结构。
- **EN L597:** This line contributes to the declaration or call of `any_of`.
  **CN L597:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L598:** This line contributes to the declaration or call of `parameters`.
  **CN L598:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L599:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L599:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L600:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L600:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 601-612 / 第 601-612 行

```tablegen
 601:     ```
 602: 
 603:     The above program defines a type `complex` inside the dialect `cmath` that
 604:     has a single type parameter that can be either `i32`, `i64`, `f32` or
 605:     `f64`.
 606:   }];
 607: 
 608:   let arguments = (ins Variadic<IRDL_AttributeType>:$args);
 609:   let results = (outs IRDL_AttributeType:$output);
 610:   let assemblyFormat = [{ `(` $args `)` ` ` attr-dict }];
 611: }
 612: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L601:** This line contributes implementation detail or declarative structure to the file.
  **CN L601:** 这一行为文件补充了实现细节或声明式结构。
- **EN L602:** Blank line used to separate nearby declarations and improve readability.
  **CN L602:** 该空行用于分隔相邻声明并提升可读性。
- **EN L603:** This line contributes implementation detail or declarative structure to the file.
  **CN L603:** 这一行为文件补充了实现细节或声明式结构。
- **EN L604:** This line contributes implementation detail or declarative structure to the file.
  **CN L604:** 这一行为文件补充了实现细节或声明式结构。
- **EN L605:** This line contributes implementation detail or declarative structure to the file.
  **CN L605:** 这一行为文件补充了实现细节或声明式结构。
- **EN L606:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L606:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L607:** Blank line used to separate nearby declarations and improve readability.
  **CN L607:** 该空行用于分隔相邻声明并提升可读性。
- **EN L608:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L608:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L609:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L609:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L610:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L610:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L611:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L611:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L612:** Blank line used to separate nearby declarations and improve readability.
  **CN L612:** 该空行用于分隔相邻声明并提升可读性。

### Lines 613-624 / 第 613-624 行

```tablegen
 613: def IRDL_AllOfOp : IRDL_ConstraintOp<"all_of",
 614:                  [ParentOneOf<["TypeOp", "AttributeOp", "OperationOp"]>,
 615:                   SameOperandsAndResultType]> {
 616:   let summary = "Constraints to the intersection of the provided constraints";
 617:   let description = [{
 618:     `irdl.all_of` defines a constraint that accepts any type or attribute that
 619:     satisfies all of its provided constraints.
 620: 
 621:     Example:
 622: 
 623:     ```mlir
 624:     irdl.dialect @cmath {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L613:** This TableGen `def` record introduces `IRDL_AllOfOp`, which later participates in generated MLIR code.
  **CN L613:** 该 TableGen `def` 记录引入了 `IRDL_AllOfOp`，后续会参与生成的 MLIR 代码。
- **EN L614:** This line contributes implementation detail or declarative structure to the file.
  **CN L614:** 这一行为文件补充了实现细节或声明式结构。
- **EN L615:** This line contributes implementation detail or declarative structure to the file.
  **CN L615:** 这一行为文件补充了实现细节或声明式结构。
- **EN L616:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L616:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L617:** This line contributes implementation detail or declarative structure to the file.
  **CN L617:** 这一行为文件补充了实现细节或声明式结构。
- **EN L618:** This line contributes implementation detail or declarative structure to the file.
  **CN L618:** 这一行为文件补充了实现细节或声明式结构。
- **EN L619:** This line contributes implementation detail or declarative structure to the file.
  **CN L619:** 这一行为文件补充了实现细节或声明式结构。
- **EN L620:** Blank line used to separate nearby declarations and improve readability.
  **CN L620:** 该空行用于分隔相邻声明并提升可读性。
- **EN L621:** This line contributes implementation detail or declarative structure to the file.
  **CN L621:** 这一行为文件补充了实现细节或声明式结构。
- **EN L622:** Blank line used to separate nearby declarations and improve readability.
  **CN L622:** 该空行用于分隔相邻声明并提升可读性。
- **EN L623:** This line contributes implementation detail or declarative structure to the file.
  **CN L623:** 这一行为文件补充了实现细节或声明式结构。
- **EN L624:** This line contributes implementation detail or declarative structure to the file.
  **CN L624:** 这一行为文件补充了实现细节或声明式结构。

### Lines 625-636 / 第 625-636 行

```tablegen
 625:       irdl.type @complex_f32 {
 626:         %0 = irdl.is i32
 627:         %1 = irdl.is f32
 628:         %2 = irdl.any_of(%0, %1) // is 32-bit
 629: 
 630:         %3 = irdl.is f32
 631:         %4 = irdl.is f64
 632:         %5 = irdl.any_of(%3, %4) // is a float
 633: 
 634:         %6 = irdl.all_of(%2, %5) // is a 32-bit float
 635:         irdl.parameters(%6)
 636:       }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L625:** This line contributes implementation detail or declarative structure to the file.
  **CN L625:** 这一行为文件补充了实现细节或声明式结构。
- **EN L626:** This line contributes implementation detail or declarative structure to the file.
  **CN L626:** 这一行为文件补充了实现细节或声明式结构。
- **EN L627:** This line contributes implementation detail or declarative structure to the file.
  **CN L627:** 这一行为文件补充了实现细节或声明式结构。
- **EN L628:** This line contributes to the declaration or call of `any_of`.
  **CN L628:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L629:** Blank line used to separate nearby declarations and improve readability.
  **CN L629:** 该空行用于分隔相邻声明并提升可读性。
- **EN L630:** This line contributes implementation detail or declarative structure to the file.
  **CN L630:** 这一行为文件补充了实现细节或声明式结构。
- **EN L631:** This line contributes implementation detail or declarative structure to the file.
  **CN L631:** 这一行为文件补充了实现细节或声明式结构。
- **EN L632:** This line contributes to the declaration or call of `any_of`.
  **CN L632:** 这一行为 `any_of` 的声明或调用提供内容。
- **EN L633:** Blank line used to separate nearby declarations and improve readability.
  **CN L633:** 该空行用于分隔相邻声明并提升可读性。
- **EN L634:** This line contributes to the declaration or call of `all_of`.
  **CN L634:** 这一行为 `all_of` 的声明或调用提供内容。
- **EN L635:** This line contributes to the declaration or call of `parameters`.
  **CN L635:** 这一行为 `parameters` 的声明或调用提供内容。
- **EN L636:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L636:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 637-648 / 第 637-648 行

```tablegen
 637:     }
 638:     ```
 639: 
 640:     The above program defines a type `complex` inside the dialect `cmath` that
 641:     has one parameter that must be 32-bit long and a float (in other
 642:     words, that must be `f32`).
 643:   }];
 644: 
 645:   let arguments = (ins Variadic<IRDL_AttributeType>:$args);
 646:   let results = (outs IRDL_AttributeType:$output);
 647:   let assemblyFormat = [{ `(` $args `)` ` ` attr-dict }];
 648: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L637:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L637:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L638:** This line contributes implementation detail or declarative structure to the file.
  **CN L638:** 这一行为文件补充了实现细节或声明式结构。
- **EN L639:** Blank line used to separate nearby declarations and improve readability.
  **CN L639:** 该空行用于分隔相邻声明并提升可读性。
- **EN L640:** This line contributes implementation detail or declarative structure to the file.
  **CN L640:** 这一行为文件补充了实现细节或声明式结构。
- **EN L641:** This line contributes to the declaration or call of `float`.
  **CN L641:** 这一行为 `float` 的声明或调用提供内容。
- **EN L642:** This line contributes implementation detail or declarative structure to the file.
  **CN L642:** 这一行为文件补充了实现细节或声明式结构。
- **EN L643:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L643:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L644:** Blank line used to separate nearby declarations and improve readability.
  **CN L644:** 该空行用于分隔相邻声明并提升可读性。
- **EN L645:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L645:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L646:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L646:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L647:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L647:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L648:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L648:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 649-660 / 第 649-660 行

```tablegen
 649: 
 650: def IRDL_CPredOp : IRDL_Op<"c_pred"> {
 651:   let summary = "Constraints an attribute using a C++ predicate";
 652:   let description = [{
 653:     `irdl.c_pred` defines a constraint that is written in C++.
 654: 
 655:     Dialects using this operation cannot be registered at runtime, as it relies
 656:     on C++ code.
 657: 
 658:     Special placeholders can be used to refer to entities in the context where
 659:     this predicate is used. They serve as "hooks" to the enclosing environment.
 660:     The following special placeholders are supported in constraints for an op:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L649:** Blank line used to separate nearby declarations and improve readability.
  **CN L649:** 该空行用于分隔相邻声明并提升可读性。
- **EN L650:** This TableGen `def` record introduces `IRDL_CPredOp`, which later participates in generated MLIR code.
  **CN L650:** 该 TableGen `def` 记录引入了 `IRDL_CPredOp`，后续会参与生成的 MLIR 代码。
- **EN L651:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L651:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L652:** This line contributes implementation detail or declarative structure to the file.
  **CN L652:** 这一行为文件补充了实现细节或声明式结构。
- **EN L653:** This line contributes implementation detail or declarative structure to the file.
  **CN L653:** 这一行为文件补充了实现细节或声明式结构。
- **EN L654:** Blank line used to separate nearby declarations and improve readability.
  **CN L654:** 该空行用于分隔相邻声明并提升可读性。
- **EN L655:** This line contributes implementation detail or declarative structure to the file.
  **CN L655:** 这一行为文件补充了实现细节或声明式结构。
- **EN L656:** This line contributes implementation detail or declarative structure to the file.
  **CN L656:** 这一行为文件补充了实现细节或声明式结构。
- **EN L657:** Blank line used to separate nearby declarations and improve readability.
  **CN L657:** 该空行用于分隔相邻声明并提升可读性。
- **EN L658:** This line contributes implementation detail or declarative structure to the file.
  **CN L658:** 这一行为文件补充了实现细节或声明式结构。
- **EN L659:** This line contributes implementation detail or declarative structure to the file.
  **CN L659:** 这一行为文件补充了实现细节或声明式结构。
- **EN L660:** This line contributes implementation detail or declarative structure to the file.
  **CN L660:** 这一行为文件补充了实现细节或声明式结构。

### Lines 661-672 / 第 661-672 行

```tablegen
 661: 
 662:     * `$_builder` will be replaced by a mlir::Builder instance.
 663:     * `$_op` will be replaced by the current operation.
 664:     * `$_self` will be replaced with the entity this predicate is attached to.
 665:        Compared to ODS, `$_self` is always of type `mlir::Attribute`, and types
 666:        are manipulated as `TypeAttr` attributes.
 667: 
 668:     Example:
 669:     ```mlir
 670:     irdl.type @op_with_attr {
 671:       %0 = irdl.c_pred "::llvm::isa<::mlir::IntegerAttr>($_self)"
 672:       irdl.parameters(%0)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L661:** Blank line used to separate nearby declarations and improve readability.
  **CN L661:** 该空行用于分隔相邻声明并提升可读性。
- **EN L662:** This comment states: “`$_builder` will be replaced by a mlir::Builder instance.”, documenting the intent of the surrounding code.
  **CN L662:** 该注释写道：“`$_builder` will be replaced by a mlir::Builder instance.”，用于说明周围代码的意图。
- **EN L663:** This comment states: “`$_op` will be replaced by the current operation.”, documenting the intent of the surrounding code.
  **CN L663:** 该注释写道：“`$_op` will be replaced by the current operation.”，用于说明周围代码的意图。
- **EN L664:** This comment states: “`$_self` will be replaced with the entity this predicate is attached to.”, documenting the intent of the surrounding code.
  **CN L664:** 该注释写道：“`$_self` will be replaced with the entity this predicate is attached to.”，用于说明周围代码的意图。
- **EN L665:** This line contributes implementation detail or declarative structure to the file.
  **CN L665:** 这一行为文件补充了实现细节或声明式结构。
- **EN L666:** This line contributes implementation detail or declarative structure to the file.
  **CN L666:** 这一行为文件补充了实现细节或声明式结构。
- **EN L667:** Blank line used to separate nearby declarations and improve readability.
  **CN L667:** 该空行用于分隔相邻声明并提升可读性。
- **EN L668:** This line contributes implementation detail or declarative structure to the file.
  **CN L668:** 这一行为文件补充了实现细节或声明式结构。
- **EN L669:** This line contributes implementation detail or declarative structure to the file.
  **CN L669:** 这一行为文件补充了实现细节或声明式结构。
- **EN L670:** This line contributes implementation detail or declarative structure to the file.
  **CN L670:** 这一行为文件补充了实现细节或声明式结构。
- **EN L671:** This line contributes implementation detail or declarative structure to the file.
  **CN L671:** 这一行为文件补充了实现细节或声明式结构。
- **EN L672:** This line contributes to the declaration or call of `parameters`.
  **CN L672:** 这一行为 `parameters` 的声明或调用提供内容。

### Lines 673-684 / 第 673-684 行

```tablegen
 673:     }
 674:     ```
 675: 
 676:     In this example, @op_with_attr is defined as a type with a single
 677:     parameter, which is an `IntegerAttr`, as constrained by the C++ predicate.
 678:   }];
 679: 
 680:   let arguments = (ins StrAttr:$pred);
 681:   let results = (outs IRDL_AttributeType:$output);
 682:   let assemblyFormat = "$pred ` ` attr-dict";
 683: }
 684: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L673:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L673:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L674:** This line contributes implementation detail or declarative structure to the file.
  **CN L674:** 这一行为文件补充了实现细节或声明式结构。
- **EN L675:** Blank line used to separate nearby declarations and improve readability.
  **CN L675:** 该空行用于分隔相邻声明并提升可读性。
- **EN L676:** This line contributes implementation detail or declarative structure to the file.
  **CN L676:** 这一行为文件补充了实现细节或声明式结构。
- **EN L677:** This line contributes implementation detail or declarative structure to the file.
  **CN L677:** 这一行为文件补充了实现细节或声明式结构。
- **EN L678:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L678:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L679:** Blank line used to separate nearby declarations and improve readability.
  **CN L679:** 该空行用于分隔相邻声明并提升可读性。
- **EN L680:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L680:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L681:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L681:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L682:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L682:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L683:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L683:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L684:** Blank line used to separate nearby declarations and improve readability.
  **CN L684:** 该空行用于分隔相邻声明并提升可读性。

### Lines 685-685 / 第 685-685 行

```tablegen
 685: #endif // MLIR_DIALECT_IRDL_IR_IRDLOPS
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L685:** This directive closes the conditional compilation region guarded by `MLIR_DIALECT_IRDL_IR_IRDLOPS`.
  **CN L685:** 该指令结束了由 `MLIR_DIALECT_IRDL_IR_IRDLOPS` 保护的条件编译区域。

## Key Concepts / 关键概念

- **IRDL_Op**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **AtMostOneChildOf**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_ConstraintOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_DialectOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_TypeOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_AttributeOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_ParametersOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IRDL_OperationOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。

## Dependencies / 依赖关系

- **`mlir/Dialect/IRDL/IR/IRDL.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/IRDL/IR/IRDLAttributes.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/IRDL/IR/IRDLTypes.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/IRDL/IR/IRDLInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/SideEffectInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/InferTypeOpInterface.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/SymbolInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/BuiltinAttributes.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
