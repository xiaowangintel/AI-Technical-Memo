# LinalgBase.td — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `mlir/include/mlir/Dialect/Linalg/IR/LinalgBase.td` | `mlir/include/mlir/Dialect/Linalg/IR/LinalgBase.td` |
| Repository | `llvm/llvm-project` | `llvm/llvm-project` |
| Purpose | This is the definition file for base linear algebra support. | 该文件的主要内容为：This is the definition file for base linear algebra support。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```tablegen
   1: //===- LinalgBase.td - Linalg dialect base support ---------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This is the definition file for base linear algebra support.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1:** This comment states: “===- LinalgBase.td - Linalg dialect base support ---------*- tablegen -*-===”, documenting the intent of the surrounding code.
  **CN L1:** 该注释写道：“===- LinalgBase.td - Linalg dialect base support ---------*- tablegen -*-===”，用于说明周围代码的意图。
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
- **EN L9:** This comment states: “This is the definition file for base linear algebra support.”, documenting the intent of the surrounding code.
  **CN L9:** 该注释写道：“This is the definition file for base linear algebra support.”，用于说明周围代码的意图。
- **EN L10:** This comment documents context for the surrounding code.
  **CN L10:** 该注释为周围代码提供上下文说明。
- **EN L11:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L11:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L12:** Blank line used to separate nearby declarations and improve readability.
  **CN L12:** 该空行用于分隔相邻声明并提升可读性。

### Lines 13-24 / 第 13-24 行

```tablegen
  13: #ifndef LINALG_BASE
  14: #define LINALG_BASE
  15: 
  16: include "mlir/Dialect/Utils/StructuredOpsUtils.td"
  17: include "mlir/Dialect/Linalg/IR/LinalgEnums.td"
  18: include "mlir/IR/EnumAttr.td"
  19: include "mlir/IR/OpBase.td"
  20: 
  21: def Linalg_Dialect : Dialect {
  22:   let name = "linalg";
  23:   let description = [{
  24:     The `linalg` dialect groups together a set of types, operations and
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L13:** This preprocessor directive manages `LINALG_BASE` as part of the file's conditional compilation boundary.
  **CN L13:** 该预处理指令管理 `LINALG_BASE`，作为文件条件编译边界的一部分。
- **EN L14:** This preprocessor directive manages `LINALG_BASE` as part of the file's conditional compilation boundary.
  **CN L14:** 该预处理指令管理 `LINALG_BASE`，作为文件条件编译边界的一部分。
- **EN L15:** Blank line used to separate nearby declarations and improve readability.
  **CN L15:** 该空行用于分隔相邻声明并提升可读性。
- **EN L16:** This TableGen include reuses records from `mlir/Dialect/Utils/StructuredOpsUtils.td`.
  **CN L16:** 该 TableGen include 复用了 `mlir/Dialect/Utils/StructuredOpsUtils.td` 中的记录。
- **EN L17:** This TableGen include reuses records from `mlir/Dialect/Linalg/IR/LinalgEnums.td`.
  **CN L17:** 该 TableGen include 复用了 `mlir/Dialect/Linalg/IR/LinalgEnums.td` 中的记录。
- **EN L18:** This TableGen include reuses records from `mlir/IR/EnumAttr.td`.
  **CN L18:** 该 TableGen include 复用了 `mlir/IR/EnumAttr.td` 中的记录。
- **EN L19:** This TableGen include reuses records from `mlir/IR/OpBase.td`.
  **CN L19:** 该 TableGen include 复用了 `mlir/IR/OpBase.td` 中的记录。
- **EN L20:** Blank line used to separate nearby declarations and improve readability.
  **CN L20:** 该空行用于分隔相邻声明并提升可读性。
- **EN L21:** This TableGen `def` record introduces `Linalg_Dialect`, which later participates in generated MLIR code.
  **CN L21:** 该 TableGen `def` 记录引入了 `Linalg_Dialect`，后续会参与生成的 MLIR 代码。
- **EN L22:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L22:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L23:** This line contributes implementation detail or declarative structure to the file.
  **CN L23:** 这一行为文件补充了实现细节或声明式结构。
- **EN L24:** This line contributes implementation detail or declarative structure to the file.
  **CN L24:** 这一行为文件补充了实现细节或声明式结构。

### Lines 25-36 / 第 25-36 行

```tablegen
  25:     transformations that are useful to implement a structured abstraction on
  26:     buffers and tensors. These abstractions are useful for transformations and
  27:     can lower to scalar load/store and other operations or to more general
  28:     library calls.
  29: 
  30:     Additional [Linalg Dialect
  31:     Documentation](https://mlir.llvm.org/docs/Dialects/Linalg) and a
  32:     [Rationale
  33:     Document](https://mlir.llvm.org/docs/Rationale/RationaleLinalgDialect) are
  34:     are also available and should be read first before going in the details of
  35:     the op semantics.
  36:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L25:** This line contributes implementation detail or declarative structure to the file.
  **CN L25:** 这一行为文件补充了实现细节或声明式结构。
- **EN L26:** This line contributes implementation detail or declarative structure to the file.
  **CN L26:** 这一行为文件补充了实现细节或声明式结构。
- **EN L27:** This line contributes implementation detail or declarative structure to the file.
  **CN L27:** 这一行为文件补充了实现细节或声明式结构。
- **EN L28:** This line contributes implementation detail or declarative structure to the file.
  **CN L28:** 这一行为文件补充了实现细节或声明式结构。
- **EN L29:** Blank line used to separate nearby declarations and improve readability.
  **CN L29:** 该空行用于分隔相邻声明并提升可读性。
- **EN L30:** This line contributes implementation detail or declarative structure to the file.
  **CN L30:** 这一行为文件补充了实现细节或声明式结构。
- **EN L31:** This line contributes implementation detail or declarative structure to the file.
  **CN L31:** 这一行为文件补充了实现细节或声明式结构。
- **EN L32:** This line contributes implementation detail or declarative structure to the file.
  **CN L32:** 这一行为文件补充了实现细节或声明式结构。
- **EN L33:** This line contributes implementation detail or declarative structure to the file.
  **CN L33:** 这一行为文件补充了实现细节或声明式结构。
- **EN L34:** This line contributes implementation detail or declarative structure to the file.
  **CN L34:** 这一行为文件补充了实现细节或声明式结构。
- **EN L35:** This line contributes implementation detail or declarative structure to the file.
  **CN L35:** 这一行为文件补充了实现细节或声明式结构。
- **EN L36:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L36:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 37-48 / 第 37-48 行

```tablegen
  37:   let cppNamespace = "::mlir::linalg";
  38:   let dependentDialects = [
  39:     "arith::ArithDialect",
  40:     "affine::AffineDialect",
  41:     "math::MathDialect",
  42:     "memref::MemRefDialect",
  43:     "tensor::TensorDialect",
  44:   ];
  45:   let useDefaultAttributePrinterParser = 1;
  46:   let hasCanonicalizer = 1;
  47:   let hasOperationAttrVerify = 1;
  48:   let hasConstantMaterializer = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L37:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L37:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L38:** This line contributes implementation detail or declarative structure to the file.
  **CN L38:** 这一行为文件补充了实现细节或声明式结构。
- **EN L39:** This line contributes implementation detail or declarative structure to the file.
  **CN L39:** 这一行为文件补充了实现细节或声明式结构。
- **EN L40:** This line contributes implementation detail or declarative structure to the file.
  **CN L40:** 这一行为文件补充了实现细节或声明式结构。
- **EN L41:** This line contributes implementation detail or declarative structure to the file.
  **CN L41:** 这一行为文件补充了实现细节或声明式结构。
- **EN L42:** This line contributes implementation detail or declarative structure to the file.
  **CN L42:** 这一行为文件补充了实现细节或声明式结构。
- **EN L43:** This line contributes implementation detail or declarative structure to the file.
  **CN L43:** 这一行为文件补充了实现细节或声明式结构。
- **EN L44:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L44:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L45:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L45:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L46:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L46:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L47:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L47:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L48:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L48:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 49-60 / 第 49-60 行

```tablegen
  49:   let extraClassDeclaration = [{
  50:     /// Attribute name used to memoize indexing maps for named ops.
  51:     constexpr const static ::llvm::StringLiteral
  52:         kMemoizedIndexingMapsAttrName = "linalg.memoized_indexing_maps";
  53: 
  54:     using RegionBuilderFunType = llvm::function_ref<
  55:       void(ImplicitLocOpBuilder &b, Block &, ArrayRef<NamedAttribute>,
  56:            function_ref<InFlightDiagnostic()>)>;
  57:     RegionBuilderFunType getRegionBuilder(StringRef name) {
  58:       return namedStructuredOpRegionBuilders.lookup(name);
  59:     }
  60:     private:
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L49:** This line contributes implementation detail or declarative structure to the file.
  **CN L49:** 这一行为文件补充了实现细节或声明式结构。
- **EN L50:** This comment states: “Attribute name used to memoize indexing maps for named ops.”, documenting the intent of the surrounding code.
  **CN L50:** 该注释写道：“Attribute name used to memoize indexing maps for named ops.”，用于说明周围代码的意图。
- **EN L51:** This line contributes implementation detail or declarative structure to the file.
  **CN L51:** 这一行为文件补充了实现细节或声明式结构。
- **EN L52:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L52:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L53:** Blank line used to separate nearby declarations and improve readability.
  **CN L53:** 该空行用于分隔相邻声明并提升可读性。
- **EN L54:** This `using` declaration introduces `RegionBuilderFunType` as an alias or imported name.
  **CN L54:** 该 `using` 声明把 `RegionBuilderFunType` 引入为别名或可直接使用的名称。
- **EN L55:** This line contributes to the declaration or call of `void`.
  **CN L55:** 这一行为 `void` 的声明或调用提供内容。
- **EN L56:** This line contributes to the declaration or call of `InFlightDiagnostic`.
  **CN L56:** 这一行为 `InFlightDiagnostic` 的声明或调用提供内容。
- **EN L57:** This line contributes to the declaration or call of `getRegionBuilder`.
  **CN L57:** 这一行为 `getRegionBuilder` 的声明或调用提供内容。
- **EN L58:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L58:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L59:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L59:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L60:** This line contributes implementation detail or declarative structure to the file.
  **CN L60:** 这一行为文件补充了实现细节或声明式结构。

### Lines 61-72 / 第 61-72 行

```tablegen
  61:       llvm::StringMap<RegionBuilderFunType> namedStructuredOpRegionBuilders;
  62:   }];
  63: }
  64: 
  65: // Define the attribute enums matching elementwise op kind (e.g., add).
  66: def ElementwiseKindAttr : EnumAttr<Linalg_Dialect,
  67:                                    ElementwiseKind, "elementwise_kind"> {
  68:   let assemblyFormat = "`<` $value `>`";
  69: }
  70: 
  71: // Define the function attribute enums matching the OpDSL functions.
  72: def UnaryFnAttr : EnumAttr<Linalg_Dialect, UnaryFn, "unary_fn"> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L61:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L61:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L62:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L62:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L63:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L63:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L64:** Blank line used to separate nearby declarations and improve readability.
  **CN L64:** 该空行用于分隔相邻声明并提升可读性。
- **EN L65:** This comment states: “Define the attribute enums matching elementwise op kind (e.g., add).”, documenting the intent of the surrounding code.
  **CN L65:** 该注释写道：“Define the attribute enums matching elementwise op kind (e.g., add).”，用于说明周围代码的意图。
- **EN L66:** This TableGen `def` record introduces `ElementwiseKindAttr`, which later participates in generated MLIR code.
  **CN L66:** 该 TableGen `def` 记录引入了 `ElementwiseKindAttr`，后续会参与生成的 MLIR 代码。
- **EN L67:** This line contributes implementation detail or declarative structure to the file.
  **CN L67:** 这一行为文件补充了实现细节或声明式结构。
- **EN L68:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L68:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L69:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L69:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L70:** Blank line used to separate nearby declarations and improve readability.
  **CN L70:** 该空行用于分隔相邻声明并提升可读性。
- **EN L71:** This comment states: “Define the function attribute enums matching the OpDSL functions.”, documenting the intent of the surrounding code.
  **CN L71:** 该注释写道：“Define the function attribute enums matching the OpDSL functions.”，用于说明周围代码的意图。
- **EN L72:** This TableGen `def` record introduces `UnaryFnAttr`, which later participates in generated MLIR code.
  **CN L72:** 该 TableGen `def` 记录引入了 `UnaryFnAttr`，后续会参与生成的 MLIR 代码。

### Lines 73-84 / 第 73-84 行

```tablegen
  73:   let assemblyFormat = "`<` $value `>`";
  74: }
  75: def BinaryFnAttr : EnumAttr<Linalg_Dialect, BinaryFn, "binary_fn"> {
  76:   let assemblyFormat = "`<` $value `>`";
  77: }
  78: def TernaryFnAttr : EnumAttr<Linalg_Dialect, TernaryFn, "ternary_fn"> {
  79:   let assemblyFormat = "`<` $value `>`";
  80: }
  81: def TypeFnAttr : EnumAttr<Linalg_Dialect, TypeFn, "type_fn"> {
  82:   let assemblyFormat = "`<` $value `>`";
  83: }
  84: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L73:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L73:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L74:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L74:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L75:** This TableGen `def` record introduces `BinaryFnAttr`, which later participates in generated MLIR code.
  **CN L75:** 该 TableGen `def` 记录引入了 `BinaryFnAttr`，后续会参与生成的 MLIR 代码。
- **EN L76:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L76:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L77:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L77:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L78:** This TableGen `def` record introduces `TernaryFnAttr`, which later participates in generated MLIR code.
  **CN L78:** 该 TableGen `def` 记录引入了 `TernaryFnAttr`，后续会参与生成的 MLIR 代码。
- **EN L79:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L79:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L80:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L80:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L81:** This TableGen `def` record introduces `TypeFnAttr`, which later participates in generated MLIR code.
  **CN L81:** 该 TableGen `def` 记录引入了 `TypeFnAttr`，后续会参与生成的 MLIR 代码。
- **EN L82:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L82:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L83:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L83:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L84:** Blank line used to separate nearby declarations and improve readability.
  **CN L84:** 该空行用于分隔相邻声明并提升可读性。

### Lines 85-91 / 第 85-91 行

```tablegen
  85: def IteratorTypeEnum : EnumAttr<Linalg_Dialect, IteratorType, "iterator_type"> {
  86:   let assemblyFormat = "`<` $value `>`";
  87: }
  88: def IteratorTypeArrayAttr : TypedArrayAttrBase<IteratorTypeEnum,
  89:   "Iterator type should be an enum.">;
  90: 
  91: #endif // LINALG_BASE
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L85:** This TableGen `def` record introduces `IteratorTypeEnum`, which later participates in generated MLIR code.
  **CN L85:** 该 TableGen `def` 记录引入了 `IteratorTypeEnum`，后续会参与生成的 MLIR 代码。
- **EN L86:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L86:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L87:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L87:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L88:** This TableGen `def` record introduces `IteratorTypeArrayAttr`, which later participates in generated MLIR code.
  **CN L88:** 该 TableGen `def` 记录引入了 `IteratorTypeArrayAttr`，后续会参与生成的 MLIR 代码。
- **EN L89:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L89:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L90:** Blank line used to separate nearby declarations and improve readability.
  **CN L90:** 该空行用于分隔相邻声明并提升可读性。
- **EN L91:** This directive closes the conditional compilation region guarded by `LINALG_BASE`.
  **CN L91:** 该指令结束了由 `LINALG_BASE` 保护的条件编译区域。

## Key Concepts / 关键概念

- **Linalg_Dialect**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ElementwiseKindAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **UnaryFnAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **BinaryFnAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **TernaryFnAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **TypeFnAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IteratorTypeEnum**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **IteratorTypeArrayAttr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。

## Dependencies / 依赖关系

- **`mlir/Dialect/Utils/StructuredOpsUtils.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/Linalg/IR/LinalgEnums.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/EnumAttr.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/OpBase.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
