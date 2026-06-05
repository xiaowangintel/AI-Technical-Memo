# XeVMOps.td — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `mlir/include/mlir/Dialect/LLVMIR/XeVMOps.td` | `mlir/include/mlir/Dialect/LLVMIR/XeVMOps.td` |
| Repository | `llvm/llvm-project` | `llvm/llvm-project` |
| Purpose | This file is licensed under the Apache License v2.0 with LLVM Exceptions. | 该文件的主要内容为：This file is licensed under the Apache License v2.0 with LLVM Exceptions。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```tablegen
   1: //===-- XeVMOps.td - XeVM dialect definition ---------------*- tablegen -*-===//
   2: //
   3: // This file is licensed under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: #ifndef XEVMIR_OPS
   9: #define XEVMIR_OPS
  10: 
  11: include "mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td"
  12: include "mlir/Dialect/LLVMIR/LLVMOpBase.td"
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN L1:** This comment states: “===-- XeVMOps.td - XeVM dialect definition ---------------*- tablegen -*-===”, documenting the intent of the surrounding code.
  **CN L1:** 该注释写道：“===-- XeVMOps.td - XeVM dialect definition ---------------*- tablegen -*-===”，用于说明周围代码的意图。
- **EN L2:** This comment documents context for the surrounding code.
  **CN L2:** 该注释为周围代码提供上下文说明。
- **EN L3:** This comment states: “This file is licensed under the Apache License v2.0 with LLVM Exceptions.”, documenting the intent of the surrounding code.
  **CN L3:** 该注释写道：“This file is licensed under the Apache License v2.0 with LLVM Exceptions.”，用于说明周围代码的意图。
- **EN L4:** This comment states: “See https://llvm.org/LICENSE.txt for license information.”, documenting the intent of the surrounding code.
  **CN L4:** 该注释写道：“See https://llvm.org/LICENSE.txt for license information.”，用于说明周围代码的意图。
- **EN L5:** This comment states: “SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”, documenting the intent of the surrounding code.
  **CN L5:** 该注释写道：“SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”，用于说明周围代码的意图。
- **EN L6:** This comment documents context for the surrounding code.
  **CN L6:** 该注释为周围代码提供上下文说明。
- **EN L7:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L7:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L8:** This preprocessor directive manages `XEVMIR_OPS` as part of the file's conditional compilation boundary.
  **CN L8:** 该预处理指令管理 `XEVMIR_OPS`，作为文件条件编译边界的一部分。
- **EN L9:** This preprocessor directive manages `XEVMIR_OPS` as part of the file's conditional compilation boundary.
  **CN L9:** 该预处理指令管理 `XEVMIR_OPS`，作为文件条件编译边界的一部分。
- **EN L10:** Blank line used to separate nearby declarations and improve readability.
  **CN L10:** 该空行用于分隔相邻声明并提升可读性。
- **EN L11:** This TableGen include reuses records from `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td`.
  **CN L11:** 该 TableGen include 复用了 `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td` 中的记录。
- **EN L12:** This TableGen include reuses records from `mlir/Dialect/LLVMIR/LLVMOpBase.td`.
  **CN L12:** 该 TableGen include 复用了 `mlir/Dialect/LLVMIR/LLVMOpBase.td` 中的记录。

### Lines 13-24 / 第 13-24 行

```tablegen
  13: include "mlir/Interfaces/SideEffectInterfaces.td"
  14: 
  15: include "mlir/IR/OpBase.td"
  16: include "mlir/IR/EnumAttr.td"
  17: 
  18: def XeVM_Dialect : Dialect {
  19:   let name = "xevm";
  20:   let cppNamespace = "::mlir::xevm";
  21:   let summary = "The XeVM dialect that extends LLVM dialect and models Intel "
  22:                 "GPU's hardware features.";
  23:   let description = [{
  24:     The XeVM dialect is extension to the LLVM dialect that models hardware
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L13:** This TableGen include reuses records from `mlir/Interfaces/SideEffectInterfaces.td`.
  **CN L13:** 该 TableGen include 复用了 `mlir/Interfaces/SideEffectInterfaces.td` 中的记录。
- **EN L14:** Blank line used to separate nearby declarations and improve readability.
  **CN L14:** 该空行用于分隔相邻声明并提升可读性。
- **EN L15:** This TableGen include reuses records from `mlir/IR/OpBase.td`.
  **CN L15:** 该 TableGen include 复用了 `mlir/IR/OpBase.td` 中的记录。
- **EN L16:** This TableGen include reuses records from `mlir/IR/EnumAttr.td`.
  **CN L16:** 该 TableGen include 复用了 `mlir/IR/EnumAttr.td` 中的记录。
- **EN L17:** Blank line used to separate nearby declarations and improve readability.
  **CN L17:** 该空行用于分隔相邻声明并提升可读性。
- **EN L18:** This TableGen `def` record introduces `XeVM_Dialect`, which later participates in generated MLIR code.
  **CN L18:** 该 TableGen `def` 记录引入了 `XeVM_Dialect`，后续会参与生成的 MLIR 代码。
- **EN L19:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L19:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L20:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L20:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L21:** This line contributes implementation detail or declarative structure to the file.
  **CN L21:** 这一行为文件补充了实现细节或声明式结构。
- **EN L22:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L22:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L23:** This line contributes implementation detail or declarative structure to the file.
  **CN L23:** 这一行为文件补充了实现细节或声明式结构。
- **EN L24:** This line contributes implementation detail or declarative structure to the file.
  **CN L24:** 这一行为文件补充了实现细节或声明式结构。

### Lines 25-36 / 第 25-36 行

```tablegen
  25:     features of Intel GPUs. The dialect is designed to work with the Xe
  26:     architecture for Intel GPUs, supporting advanced operations like 2D block
  27:     loads, stores, prefetch and matrix multiply-add (MMA) operations.
  28:   }];
  29:   let dependentDialects = ["LLVM::LLVMDialect"];
  30: 
  31:   let extraClassDeclaration = [{
  32:     /// Get the name for the attribute used to specify cache control
  33:     /// decorations.
  34:     static constexpr ::llvm::StringRef getCacheControlsAttrName() {
  35:       return ::llvm::StringLiteral("xevm.DecorationCacheControl");
  36:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L25:** This line contributes implementation detail or declarative structure to the file.
  **CN L25:** 这一行为文件补充了实现细节或声明式结构。
- **EN L26:** This line contributes implementation detail or declarative structure to the file.
  **CN L26:** 这一行为文件补充了实现细节或声明式结构。
- **EN L27:** This line contributes to the declaration or call of `add`.
  **CN L27:** 这一行为 `add` 的声明或调用提供内容。
- **EN L28:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L28:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L29:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L29:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L30:** Blank line used to separate nearby declarations and improve readability.
  **CN L30:** 该空行用于分隔相邻声明并提升可读性。
- **EN L31:** This line contributes implementation detail or declarative structure to the file.
  **CN L31:** 这一行为文件补充了实现细节或声明式结构。
- **EN L32:** This comment states: “Get the name for the attribute used to specify cache control”, documenting the intent of the surrounding code.
  **CN L32:** 该注释写道：“Get the name for the attribute used to specify cache control”，用于说明周围代码的意图。
- **EN L33:** This comment states: “decorations.”, documenting the intent of the surrounding code.
  **CN L33:** 该注释写道：“decorations.”，用于说明周围代码的意图。
- **EN L34:** This line contributes to the declaration or call of `getCacheControlsAttrName`.
  **CN L34:** 这一行为 `getCacheControlsAttrName` 的声明或调用提供内容。
- **EN L35:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L35:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L36:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L36:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 37-48 / 第 37-48 行

```tablegen
  37:   }];
  38: 
  39:   let useDefaultAttributePrinterParser = 1;
  40: }
  41: 
  42: class XeVM_Attr<string attrName, string attrMnemonic, list<Trait> traits = []>
  43:     : AttrDef<XeVM_Dialect, attrName, traits> {
  44:   let mnemonic = attrMnemonic;
  45: }
  46: 
  47: class XeVM_Op<string mnemonic, list<Trait> traits = []>
  48:     : LLVM_OpBase<XeVM_Dialect, mnemonic, traits> {
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L37:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L37:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L38:** Blank line used to separate nearby declarations and improve readability.
  **CN L38:** 该空行用于分隔相邻声明并提升可读性。
- **EN L39:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L39:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L40:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L40:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L41:** Blank line used to separate nearby declarations and improve readability.
  **CN L41:** 该空行用于分隔相邻声明并提升可读性。
- **EN L42:** This TableGen `class` record introduces `XeVM_Attr`, which later participates in generated MLIR code.
  **CN L42:** 该 TableGen `class` 记录引入了 `XeVM_Attr`，后续会参与生成的 MLIR 代码。
- **EN L43:** This line contributes implementation detail or declarative structure to the file.
  **CN L43:** 这一行为文件补充了实现细节或声明式结构。
- **EN L44:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L44:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L45:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L45:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L46:** Blank line used to separate nearby declarations and improve readability.
  **CN L46:** 该空行用于分隔相邻声明并提升可读性。
- **EN L47:** This TableGen `class` record introduces `XeVM_Op`, which later participates in generated MLIR code.
  **CN L47:** 该 TableGen `class` 记录引入了 `XeVM_Op`，后续会参与生成的 MLIR 代码。
- **EN L48:** This line contributes implementation detail or declarative structure to the file.
  **CN L48:** 这一行为文件补充了实现细节或声明式结构。

### Lines 49-60 / 第 49-60 行

```tablegen
  49: 
  50:   code extraBaseClassDeclaration = [{
  51:     void printProperties(::mlir::MLIRContext *ctx,
  52:             ::mlir::OpAsmPrinter &p, const Properties &prop,
  53:             ::mlir::ArrayRef<::llvm::StringRef> elidedProps) {
  54:       Attribute propAttr = getPropertiesAsAttr(ctx, prop);
  55:       if (propAttr)
  56:         p << "<" << propAttr << ">";
  57:     }
  58: 
  59:     static ::mlir::ParseResult parseProperties(::mlir::OpAsmParser &parser,
  60:                                      ::mlir::OperationState &result) {
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L49:** Blank line used to separate nearby declarations and improve readability.
  **CN L49:** 该空行用于分隔相邻声明并提升可读性。
- **EN L50:** This line contributes implementation detail or declarative structure to the file.
  **CN L50:** 这一行为文件补充了实现细节或声明式结构。
- **EN L51:** This line contributes to the declaration or call of `printProperties`.
  **CN L51:** 这一行为 `printProperties` 的声明或调用提供内容。
- **EN L52:** This line contributes implementation detail or declarative structure to the file.
  **CN L52:** 这一行为文件补充了实现细节或声明式结构。
- **EN L53:** This line contributes implementation detail or declarative structure to the file.
  **CN L53:** 这一行为文件补充了实现细节或声明式结构。
- **EN L54:** This line contributes to the declaration or call of `getPropertiesAsAttr`.
  **CN L54:** 这一行为 `getPropertiesAsAttr` 的声明或调用提供内容。
- **EN L55:** This line contributes implementation detail or declarative structure to the file.
  **CN L55:** 这一行为文件补充了实现细节或声明式结构。
- **EN L56:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L56:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L57:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L57:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L58:** Blank line used to separate nearby declarations and improve readability.
  **CN L58:** 该空行用于分隔相邻声明并提升可读性。
- **EN L59:** This line contributes to the declaration or call of `parseProperties`.
  **CN L59:** 这一行为 `parseProperties` 的声明或调用提供内容。
- **EN L60:** This line contributes implementation detail or declarative structure to the file.
  **CN L60:** 这一行为文件补充了实现细节或声明式结构。

### Lines 61-72 / 第 61-72 行

```tablegen
  61:       if (mlir::succeeded(parser.parseOptionalLess())) {
  62:         if (parser.parseAttribute(result.propertiesAttr) || parser.parseGreater())
  63:           return failure();
  64:       }
  65:       return success();
  66:     }
  67: 
  68:   }];
  69: }
  70: 
  71: def XeVM_ElemType : AnyTypeOf<[AnyI8, AnyI16, AnyI32, F32, TF32, F16, BF16]>;
  72: def XeVM_1DBlockElemType : AnyTypeOf<[I8, I16, I32, I64]>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L61:** This line contributes implementation detail or declarative structure to the file.
  **CN L61:** 这一行为文件补充了实现细节或声明式结构。
- **EN L62:** This line contributes implementation detail or declarative structure to the file.
  **CN L62:** 这一行为文件补充了实现细节或声明式结构。
- **EN L63:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L63:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L64:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L64:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L65:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L65:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L66:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L66:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L67:** Blank line used to separate nearby declarations and improve readability.
  **CN L67:** 该空行用于分隔相邻声明并提升可读性。
- **EN L68:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L68:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L69:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L69:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L70:** Blank line used to separate nearby declarations and improve readability.
  **CN L70:** 该空行用于分隔相邻声明并提升可读性。
- **EN L71:** This TableGen `def` record introduces `XeVM_ElemType`, which later participates in generated MLIR code.
  **CN L71:** 该 TableGen `def` 记录引入了 `XeVM_ElemType`，后续会参与生成的 MLIR 代码。
- **EN L72:** This TableGen `def` record introduces `XeVM_1DBlockElemType`, which later participates in generated MLIR code.
  **CN L72:** 该 TableGen `def` 记录引入了 `XeVM_1DBlockElemType`，后续会参与生成的 MLIR 代码。

### Lines 73-84 / 第 73-84 行

```tablegen
  73: 
  74: //===----------------------------------------------------------------------===//
  75: // XeVM Load Cache Control
  76: //   L1, L2, L3 - cache levels
  77: //   uc - uncached
  78: //   c - cached
  79: //   s - streaming
  80: //   ir - invalidated after read
  81: //===----------------------------------------------------------------------===//
  82: 
  83: def LoadCacheControl_Use_Default
  84:     : I32EnumAttrCase<"USE_DEFAULT", 0, "Use_Default">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L73:** Blank line used to separate nearby declarations and improve readability.
  **CN L73:** 该空行用于分隔相邻声明并提升可读性。
- **EN L74:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L74:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L75:** This comment states: “XeVM Load Cache Control”, documenting the intent of the surrounding code.
  **CN L75:** 该注释写道：“XeVM Load Cache Control”，用于说明周围代码的意图。
- **EN L76:** This comment states: “L1, L2, L3 - cache levels”, documenting the intent of the surrounding code.
  **CN L76:** 该注释写道：“L1, L2, L3 - cache levels”，用于说明周围代码的意图。
- **EN L77:** This comment states: “uc - uncached”, documenting the intent of the surrounding code.
  **CN L77:** 该注释写道：“uc - uncached”，用于说明周围代码的意图。
- **EN L78:** This comment states: “c - cached”, documenting the intent of the surrounding code.
  **CN L78:** 该注释写道：“c - cached”，用于说明周围代码的意图。
- **EN L79:** This comment states: “s - streaming”, documenting the intent of the surrounding code.
  **CN L79:** 该注释写道：“s - streaming”，用于说明周围代码的意图。
- **EN L80:** This comment states: “ir - invalidated after read”, documenting the intent of the surrounding code.
  **CN L80:** 该注释写道：“ir - invalidated after read”，用于说明周围代码的意图。
- **EN L81:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L81:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L82:** Blank line used to separate nearby declarations and improve readability.
  **CN L82:** 该空行用于分隔相邻声明并提升可读性。
- **EN L83:** This TableGen `def` record introduces `LoadCacheControl_Use_Default`, which later participates in generated MLIR code.
  **CN L83:** 该 TableGen `def` 记录引入了 `LoadCacheControl_Use_Default`，后续会参与生成的 MLIR 代码。
- **EN L84:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L84:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 85-96 / 第 85-96 行

```tablegen
  85: def LoadCacheControl_L1uc_L2uc_L3uc
  86:     : I32EnumAttrCase<"L1UC_L2UC_L3UC", 1, "L1uc_L2uc_L3uc">;
  87: def LoadCacheControl_L1uc_L2uc_L3c
  88:     : I32EnumAttrCase<"L1UC_L2UC_L3C", 2, "L1uc_L2uc_L3c">;
  89: def LoadCacheControl_L1uc_L2c_L3uc
  90:     : I32EnumAttrCase<"L1UC_L2C_L3UC", 3, "L1uc_L2c_L3uc">;
  91: def LoadCacheControl_L1uc_L2c_L3c
  92:     : I32EnumAttrCase<"L1UC_L2C_L3C", 4, "L1uc_L2c_L3c">;
  93: def LoadCacheControl_L1c_L2uc_L3uc
  94:     : I32EnumAttrCase<"L1C_L2UC_L3UC", 5, "L1c_L2uc_L3uc">;
  95: def LoadCacheControl_L1c_L2uc_L3c
  96:     : I32EnumAttrCase<"L1C_L2UC_L3C", 6, "L1c_L2uc_L3c">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L85:** This TableGen `def` record introduces `LoadCacheControl_L1uc_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L85:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1uc_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L86:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L86:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L87:** This TableGen `def` record introduces `LoadCacheControl_L1uc_L2uc_L3c`, which later participates in generated MLIR code.
  **CN L87:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1uc_L2uc_L3c`，后续会参与生成的 MLIR 代码。
- **EN L88:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L88:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L89:** This TableGen `def` record introduces `LoadCacheControl_L1uc_L2c_L3uc`, which later participates in generated MLIR code.
  **CN L89:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1uc_L2c_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L90:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L90:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L91:** This TableGen `def` record introduces `LoadCacheControl_L1uc_L2c_L3c`, which later participates in generated MLIR code.
  **CN L91:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1uc_L2c_L3c`，后续会参与生成的 MLIR 代码。
- **EN L92:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L92:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L93:** This TableGen `def` record introduces `LoadCacheControl_L1c_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L93:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1c_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L94:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L94:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L95:** This TableGen `def` record introduces `LoadCacheControl_L1c_L2uc_L3c`, which later participates in generated MLIR code.
  **CN L95:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1c_L2uc_L3c`，后续会参与生成的 MLIR 代码。
- **EN L96:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L96:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 97-108 / 第 97-108 行

```tablegen
  97: def LoadCacheControl_L1c_L2c_L3uc
  98:     : I32EnumAttrCase<"L1C_L2C_L3UC", 7, "L1c_L2c_L3uc">;
  99: def LoadCacheControl_L1c_L2c_L3c
 100:     : I32EnumAttrCase<"L1C_L2C_L3C", 8, "L1c_L2c_L3c">;
 101: def LoadCacheControl_L1s_L2uc_L3uc
 102:     : I32EnumAttrCase<"L1S_L2UC_L3UC", 9, "L1s_L2uc_L3uc">;
 103: def LoadCacheControl_L1s_L2uc_L3c
 104:     : I32EnumAttrCase<"L1S_L2UC_L3C", 10, "L1s_L2uc_L3c">;
 105: def LoadCacheControl_L1s_L2c_L3uc
 106:     : I32EnumAttrCase<"L1S_L2C_L3UC", 11, "L1s_L2c_L3uc">;
 107: def LoadCacheControl_L1s_L2c_L3c
 108:     : I32EnumAttrCase<"L1S_L2C_L3C", 12, "L1s_L2c_L3c">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L97:** This TableGen `def` record introduces `LoadCacheControl_L1c_L2c_L3uc`, which later participates in generated MLIR code.
  **CN L97:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1c_L2c_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L98:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L98:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L99:** This TableGen `def` record introduces `LoadCacheControl_L1c_L2c_L3c`, which later participates in generated MLIR code.
  **CN L99:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1c_L2c_L3c`，后续会参与生成的 MLIR 代码。
- **EN L100:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L100:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L101:** This TableGen `def` record introduces `LoadCacheControl_L1s_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L101:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1s_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L102:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L102:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L103:** This TableGen `def` record introduces `LoadCacheControl_L1s_L2uc_L3c`, which later participates in generated MLIR code.
  **CN L103:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1s_L2uc_L3c`，后续会参与生成的 MLIR 代码。
- **EN L104:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L104:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L105:** This TableGen `def` record introduces `LoadCacheControl_L1s_L2c_L3uc`, which later participates in generated MLIR code.
  **CN L105:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1s_L2c_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L106:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L106:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L107:** This TableGen `def` record introduces `LoadCacheControl_L1s_L2c_L3c`, which later participates in generated MLIR code.
  **CN L107:** 该 TableGen `def` 记录引入了 `LoadCacheControl_L1s_L2c_L3c`，后续会参与生成的 MLIR 代码。
- **EN L108:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L108:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 109-120 / 第 109-120 行

```tablegen
 109: def LoadCacheControlInvalidateRead
 110:     : I32EnumAttrCase<"INVALIDATE_READ", 13, "ir">;
 111: 
 112: def XeVM_LoadCacheControl
 113:     : I32EnumAttr<
 114:           "LoadCacheControl", "XeVM load ops cache control",
 115:           [LoadCacheControl_Use_Default,
 116:            LoadCacheControl_L1uc_L2uc_L3uc, LoadCacheControl_L1uc_L2uc_L3c,
 117:            LoadCacheControl_L1uc_L2c_L3uc, LoadCacheControl_L1uc_L2c_L3c,
 118:            LoadCacheControl_L1c_L2uc_L3uc, LoadCacheControl_L1c_L2uc_L3c,
 119:            LoadCacheControl_L1c_L2c_L3uc, LoadCacheControl_L1c_L2c_L3c,
 120:            LoadCacheControl_L1s_L2uc_L3uc, LoadCacheControl_L1s_L2uc_L3c,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L109:** This TableGen `def` record introduces `LoadCacheControlInvalidateRead`, which later participates in generated MLIR code.
  **CN L109:** 该 TableGen `def` 记录引入了 `LoadCacheControlInvalidateRead`，后续会参与生成的 MLIR 代码。
- **EN L110:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L110:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L111:** Blank line used to separate nearby declarations and improve readability.
  **CN L111:** 该空行用于分隔相邻声明并提升可读性。
- **EN L112:** This TableGen `def` record introduces `XeVM_LoadCacheControl`, which later participates in generated MLIR code.
  **CN L112:** 该 TableGen `def` 记录引入了 `XeVM_LoadCacheControl`，后续会参与生成的 MLIR 代码。
- **EN L113:** This line contributes implementation detail or declarative structure to the file.
  **CN L113:** 这一行为文件补充了实现细节或声明式结构。
- **EN L114:** This line contributes implementation detail or declarative structure to the file.
  **CN L114:** 这一行为文件补充了实现细节或声明式结构。
- **EN L115:** This line contributes implementation detail or declarative structure to the file.
  **CN L115:** 这一行为文件补充了实现细节或声明式结构。
- **EN L116:** This line contributes implementation detail or declarative structure to the file.
  **CN L116:** 这一行为文件补充了实现细节或声明式结构。
- **EN L117:** This line contributes implementation detail or declarative structure to the file.
  **CN L117:** 这一行为文件补充了实现细节或声明式结构。
- **EN L118:** This line contributes implementation detail or declarative structure to the file.
  **CN L118:** 这一行为文件补充了实现细节或声明式结构。
- **EN L119:** This line contributes implementation detail or declarative structure to the file.
  **CN L119:** 这一行为文件补充了实现细节或声明式结构。
- **EN L120:** This line contributes implementation detail or declarative structure to the file.
  **CN L120:** 这一行为文件补充了实现细节或声明式结构。

### Lines 121-132 / 第 121-132 行

```tablegen
 121:            LoadCacheControl_L1s_L2c_L3uc, LoadCacheControl_L1s_L2c_L3c,
 122:            LoadCacheControlInvalidateRead]> {
 123:   let cppNamespace = "::mlir::xevm";
 124:   let genSpecializedAttr = 0;
 125: }
 126: 
 127: def XeVM_LoadCacheControlAttr
 128:     : EnumAttr<XeVM_Dialect, XeVM_LoadCacheControl, "load_cache_control"> {
 129:   let summary = [{Describe the cache settings for load operators}];
 130:   let assemblyFormat = "`<` $value `>`";
 131: }
 132: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L121:** This line contributes implementation detail or declarative structure to the file.
  **CN L121:** 这一行为文件补充了实现细节或声明式结构。
- **EN L122:** This line contributes implementation detail or declarative structure to the file.
  **CN L122:** 这一行为文件补充了实现细节或声明式结构。
- **EN L123:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L123:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L124:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L124:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L125:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L125:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L126:** Blank line used to separate nearby declarations and improve readability.
  **CN L126:** 该空行用于分隔相邻声明并提升可读性。
- **EN L127:** This TableGen `def` record introduces `XeVM_LoadCacheControlAttr`, which later participates in generated MLIR code.
  **CN L127:** 该 TableGen `def` 记录引入了 `XeVM_LoadCacheControlAttr`，后续会参与生成的 MLIR 代码。
- **EN L128:** This line contributes implementation detail or declarative structure to the file.
  **CN L128:** 这一行为文件补充了实现细节或声明式结构。
- **EN L129:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L129:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L130:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L130:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L131:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L131:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L132:** Blank line used to separate nearby declarations and improve readability.
  **CN L132:** 该空行用于分隔相邻声明并提升可读性。

### Lines 133-144 / 第 133-144 行

```tablegen
 133: //===----------------------------------------------------------------------===//
 134: // XeVM Store Cache Control
 135: //   L1, L2, L3 - cache levels
 136: //   uc - uncached
 137: //   wb - write-back
 138: //   wt - write-through
 139: //   s - streaming
 140: //===----------------------------------------------------------------------===//
 141: 
 142: 
 143: def StoreCacheControl_Use_Default
 144:     : I32EnumAttrCase<"USE_DEFAULT", 0, "Use_Default">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L133:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L133:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L134:** This comment states: “XeVM Store Cache Control”, documenting the intent of the surrounding code.
  **CN L134:** 该注释写道：“XeVM Store Cache Control”，用于说明周围代码的意图。
- **EN L135:** This comment states: “L1, L2, L3 - cache levels”, documenting the intent of the surrounding code.
  **CN L135:** 该注释写道：“L1, L2, L3 - cache levels”，用于说明周围代码的意图。
- **EN L136:** This comment states: “uc - uncached”, documenting the intent of the surrounding code.
  **CN L136:** 该注释写道：“uc - uncached”，用于说明周围代码的意图。
- **EN L137:** This comment states: “wb - write-back”, documenting the intent of the surrounding code.
  **CN L137:** 该注释写道：“wb - write-back”，用于说明周围代码的意图。
- **EN L138:** This comment states: “wt - write-through”, documenting the intent of the surrounding code.
  **CN L138:** 该注释写道：“wt - write-through”，用于说明周围代码的意图。
- **EN L139:** This comment states: “s - streaming”, documenting the intent of the surrounding code.
  **CN L139:** 该注释写道：“s - streaming”，用于说明周围代码的意图。
- **EN L140:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L140:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L141:** Blank line used to separate nearby declarations and improve readability.
  **CN L141:** 该空行用于分隔相邻声明并提升可读性。
- **EN L142:** Blank line used to separate nearby declarations and improve readability.
  **CN L142:** 该空行用于分隔相邻声明并提升可读性。
- **EN L143:** This TableGen `def` record introduces `StoreCacheControl_Use_Default`, which later participates in generated MLIR code.
  **CN L143:** 该 TableGen `def` 记录引入了 `StoreCacheControl_Use_Default`，后续会参与生成的 MLIR 代码。
- **EN L144:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L144:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 145-156 / 第 145-156 行

```tablegen
 145: def StoreCacheControl_L1uc_L2uc_L3uc
 146:     : I32EnumAttrCase<"L1UC_L2UC_L3UC", 1, "L1uc_L2uc_L3uc">;
 147: def StoreCacheControl_L1uc_L2uc_L3wb
 148:     : I32EnumAttrCase<"L1UC_L2UC_L3WB", 2, "L1uc_L2uc_L3wb">;
 149: def StoreCacheControl_L1uc_L2wb_L3uc
 150:     : I32EnumAttrCase<"L1UC_L2WB_L3UC", 3, "L1uc_L2wb_L3uc">;
 151: def StoreCacheControl_L1uc_L2wb_L3wb
 152:     : I32EnumAttrCase<"L1UC_L2WB_L3WB", 4, "L1uc_L2wb_L3wb">;
 153: def StoreCacheControl_L1wt_L2uc_L3uc
 154:     : I32EnumAttrCase<"L1WT_L2UC_L3UC", 5, "L1wt_L2uc_L3uc">;
 155: def StoreCacheControl_L1wt_L2uc_L3wb
 156:     : I32EnumAttrCase<"L1WT_L2UC_L3WB", 6, "L1wt_L2uc_L3wb">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L145:** This TableGen `def` record introduces `StoreCacheControl_L1uc_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L145:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1uc_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L146:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L146:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L147:** This TableGen `def` record introduces `StoreCacheControl_L1uc_L2uc_L3wb`, which later participates in generated MLIR code.
  **CN L147:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1uc_L2uc_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L148:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L148:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L149:** This TableGen `def` record introduces `StoreCacheControl_L1uc_L2wb_L3uc`, which later participates in generated MLIR code.
  **CN L149:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1uc_L2wb_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L150:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L150:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L151:** This TableGen `def` record introduces `StoreCacheControl_L1uc_L2wb_L3wb`, which later participates in generated MLIR code.
  **CN L151:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1uc_L2wb_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L152:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L152:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L153:** This TableGen `def` record introduces `StoreCacheControl_L1wt_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L153:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wt_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L154:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L154:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L155:** This TableGen `def` record introduces `StoreCacheControl_L1wt_L2uc_L3wb`, which later participates in generated MLIR code.
  **CN L155:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wt_L2uc_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L156:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L156:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 157-168 / 第 157-168 行

```tablegen
 157: def StoreCacheControl_L1wt_L2wb_L3uc
 158:     : I32EnumAttrCase<"L1WT_L2WB_L3UC", 7, "L1wt_L2wb_L3uc">;
 159: def StoreCacheControl_L1wt_L2wb_L3wb
 160:     : I32EnumAttrCase<"L1WT_L2WB_L3WB", 8, "L1wt_L2wb_L3wb">;
 161: def StoreCacheControl_L1s_L2uc_L3uc
 162:     : I32EnumAttrCase<"L1S_L2UC_L3UC", 9, "L1s_L2uc_L3uc">;
 163: def StoreCacheControl_L1s_L2uc_L3wb
 164:     : I32EnumAttrCase<"L1S_L2UC_L3WB", 10, "L1s_L2uc_L3wb">;
 165: def StoreCacheControl_L1s_L2wb_L3uc
 166:     : I32EnumAttrCase<"L1S_L2WB_L3UC", 11, "L1s_L2wb_L3uc">;
 167: def StoreCacheControl_L1s_L2wb_L3wb
 168:     : I32EnumAttrCase<"L1S_L2WB_L3WB", 12, "L1s_L2wb_L3wb">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L157:** This TableGen `def` record introduces `StoreCacheControl_L1wt_L2wb_L3uc`, which later participates in generated MLIR code.
  **CN L157:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wt_L2wb_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L158:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L158:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L159:** This TableGen `def` record introduces `StoreCacheControl_L1wt_L2wb_L3wb`, which later participates in generated MLIR code.
  **CN L159:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wt_L2wb_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L160:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L160:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L161:** This TableGen `def` record introduces `StoreCacheControl_L1s_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L161:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1s_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L162:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L162:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L163:** This TableGen `def` record introduces `StoreCacheControl_L1s_L2uc_L3wb`, which later participates in generated MLIR code.
  **CN L163:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1s_L2uc_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L164:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L164:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L165:** This TableGen `def` record introduces `StoreCacheControl_L1s_L2wb_L3uc`, which later participates in generated MLIR code.
  **CN L165:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1s_L2wb_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L166:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L166:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L167:** This TableGen `def` record introduces `StoreCacheControl_L1s_L2wb_L3wb`, which later participates in generated MLIR code.
  **CN L167:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1s_L2wb_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L168:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L168:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 169-180 / 第 169-180 行

```tablegen
 169: def StoreCacheControl_L1wb_L2uc_L3uc
 170:     : I32EnumAttrCase<"L1WB_L2UC_L3UC", 13, "L1wb_L2uc_L3uc">;
 171: def StoreCacheControl_L1wb_L2wb_L3uc
 172:     : I32EnumAttrCase<"L1WB_L2WB_L3UC", 14, "L1wb_L2wb_L3uc">;
 173: def StoreCacheControl_L1wb_L2uc_L3wb
 174:     : I32EnumAttrCase<"L1WB_L2UC_L3WB", 15, "L1wb_L2uc_L3wb">;
 175: 
 176: def XeVM_StoreCacheControl
 177:     : I32EnumAttr<
 178:           "StoreCacheControl", "XeVM store ops cache control",
 179:           [StoreCacheControl_Use_Default,
 180:            StoreCacheControl_L1uc_L2uc_L3uc, StoreCacheControl_L1uc_L2uc_L3wb,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L169:** This TableGen `def` record introduces `StoreCacheControl_L1wb_L2uc_L3uc`, which later participates in generated MLIR code.
  **CN L169:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wb_L2uc_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L170:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L170:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L171:** This TableGen `def` record introduces `StoreCacheControl_L1wb_L2wb_L3uc`, which later participates in generated MLIR code.
  **CN L171:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wb_L2wb_L3uc`，后续会参与生成的 MLIR 代码。
- **EN L172:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L172:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L173:** This TableGen `def` record introduces `StoreCacheControl_L1wb_L2uc_L3wb`, which later participates in generated MLIR code.
  **CN L173:** 该 TableGen `def` 记录引入了 `StoreCacheControl_L1wb_L2uc_L3wb`，后续会参与生成的 MLIR 代码。
- **EN L174:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L174:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L175:** Blank line used to separate nearby declarations and improve readability.
  **CN L175:** 该空行用于分隔相邻声明并提升可读性。
- **EN L176:** This TableGen `def` record introduces `XeVM_StoreCacheControl`, which later participates in generated MLIR code.
  **CN L176:** 该 TableGen `def` 记录引入了 `XeVM_StoreCacheControl`，后续会参与生成的 MLIR 代码。
- **EN L177:** This line contributes implementation detail or declarative structure to the file.
  **CN L177:** 这一行为文件补充了实现细节或声明式结构。
- **EN L178:** This line contributes implementation detail or declarative structure to the file.
  **CN L178:** 这一行为文件补充了实现细节或声明式结构。
- **EN L179:** This line contributes implementation detail or declarative structure to the file.
  **CN L179:** 这一行为文件补充了实现细节或声明式结构。
- **EN L180:** This line contributes implementation detail or declarative structure to the file.
  **CN L180:** 这一行为文件补充了实现细节或声明式结构。

### Lines 181-192 / 第 181-192 行

```tablegen
 181:            StoreCacheControl_L1uc_L2wb_L3uc, StoreCacheControl_L1uc_L2wb_L3wb,
 182:            StoreCacheControl_L1wt_L2uc_L3uc, StoreCacheControl_L1wt_L2uc_L3wb,
 183:            StoreCacheControl_L1wt_L2wb_L3uc, StoreCacheControl_L1wt_L2wb_L3wb,
 184:            StoreCacheControl_L1s_L2uc_L3uc, StoreCacheControl_L1s_L2uc_L3wb,
 185:            StoreCacheControl_L1s_L2wb_L3uc, StoreCacheControl_L1s_L2wb_L3wb,
 186:            StoreCacheControl_L1wb_L2uc_L3uc, StoreCacheControl_L1wb_L2wb_L3uc,
 187:            StoreCacheControl_L1wb_L2uc_L3wb]> {
 188:   let cppNamespace = "::mlir::xevm";
 189:   let genSpecializedAttr = 0;
 190: }
 191: 
 192: def XeVM_StoreCacheControlAttr
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L181:** This line contributes implementation detail or declarative structure to the file.
  **CN L181:** 这一行为文件补充了实现细节或声明式结构。
- **EN L182:** This line contributes implementation detail or declarative structure to the file.
  **CN L182:** 这一行为文件补充了实现细节或声明式结构。
- **EN L183:** This line contributes implementation detail or declarative structure to the file.
  **CN L183:** 这一行为文件补充了实现细节或声明式结构。
- **EN L184:** This line contributes implementation detail or declarative structure to the file.
  **CN L184:** 这一行为文件补充了实现细节或声明式结构。
- **EN L185:** This line contributes implementation detail or declarative structure to the file.
  **CN L185:** 这一行为文件补充了实现细节或声明式结构。
- **EN L186:** This line contributes implementation detail or declarative structure to the file.
  **CN L186:** 这一行为文件补充了实现细节或声明式结构。
- **EN L187:** This line contributes implementation detail or declarative structure to the file.
  **CN L187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L188:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L188:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L189:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L189:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L190:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L190:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L191:** Blank line used to separate nearby declarations and improve readability.
  **CN L191:** 该空行用于分隔相邻声明并提升可读性。
- **EN L192:** This TableGen `def` record introduces `XeVM_StoreCacheControlAttr`, which later participates in generated MLIR code.
  **CN L192:** 该 TableGen `def` 记录引入了 `XeVM_StoreCacheControlAttr`，后续会参与生成的 MLIR 代码。

### Lines 193-204 / 第 193-204 行

```tablegen
 193:     : EnumAttr<XeVM_Dialect, XeVM_StoreCacheControl, "store_cache_control"> {
 194:   let summary = [{Describe the cache settings for store operators}];
 195:   let assemblyFormat = "`<` $value `>`";
 196: }
 197: 
 198: def XeVM_BlockLoadOp
 199:     : XeVM_Op<"blockload">,
 200:       Results<(outs AnyTypeOf<
 201:           [XeVM_1DBlockElemType,
 202:            FixedVectorOfRankAndType<[1], [XeVM_1DBlockElemType]>]>:$res)>,
 203:       Arguments<(ins Arg<LLVM_AnyPointer, "", [MemRead]>:$ptr,
 204:           OptionalAttr<XeVM_LoadCacheControlAttr>:$cache_control)> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L193:** This line contributes implementation detail or declarative structure to the file.
  **CN L193:** 这一行为文件补充了实现细节或声明式结构。
- **EN L194:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L194:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L195:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L195:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L196:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L196:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L197:** Blank line used to separate nearby declarations and improve readability.
  **CN L197:** 该空行用于分隔相邻声明并提升可读性。
- **EN L198:** This TableGen `def` record introduces `XeVM_BlockLoadOp`, which later participates in generated MLIR code.
  **CN L198:** 该 TableGen `def` 记录引入了 `XeVM_BlockLoadOp`，后续会参与生成的 MLIR 代码。
- **EN L199:** This line contributes implementation detail or declarative structure to the file.
  **CN L199:** 这一行为文件补充了实现细节或声明式结构。
- **EN L200:** This line contributes implementation detail or declarative structure to the file.
  **CN L200:** 这一行为文件补充了实现细节或声明式结构。
- **EN L201:** This line contributes implementation detail or declarative structure to the file.
  **CN L201:** 这一行为文件补充了实现细节或声明式结构。
- **EN L202:** This line contributes implementation detail or declarative structure to the file.
  **CN L202:** 这一行为文件补充了实现细节或声明式结构。
- **EN L203:** This line contributes implementation detail or declarative structure to the file.
  **CN L203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L204:** This line contributes implementation detail or declarative structure to the file.
  **CN L204:** 这一行为文件补充了实现细节或声明式结构。

### Lines 205-216 / 第 205-216 行

```tablegen
 205:   let summary = "subgroup block load";
 206:   let description = [{
 207:     Reads one or more components of Result data for each invocation
 208:     in the subgroup from the specified `ptr` as a block operation.
 209:     The data is read strided, so the first value read is:
 210:     ```
 211:       ptr[ SubgroupLocalInvocationId ]
 212:     ```
 213:     and the second value read is:
 214:     ```
 215:       ptr[ SubgroupLocalInvocationId + SubgroupMaxSize ]
 216:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L205:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L205:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L206:** This line contributes implementation detail or declarative structure to the file.
  **CN L206:** 这一行为文件补充了实现细节或声明式结构。
- **EN L207:** This line contributes implementation detail or declarative structure to the file.
  **CN L207:** 这一行为文件补充了实现细节或声明式结构。
- **EN L208:** This line contributes implementation detail or declarative structure to the file.
  **CN L208:** 这一行为文件补充了实现细节或声明式结构。
- **EN L209:** This line contributes implementation detail or declarative structure to the file.
  **CN L209:** 这一行为文件补充了实现细节或声明式结构。
- **EN L210:** This line contributes implementation detail or declarative structure to the file.
  **CN L210:** 这一行为文件补充了实现细节或声明式结构。
- **EN L211:** This line contributes implementation detail or declarative structure to the file.
  **CN L211:** 这一行为文件补充了实现细节或声明式结构。
- **EN L212:** This line contributes implementation detail or declarative structure to the file.
  **CN L212:** 这一行为文件补充了实现细节或声明式结构。
- **EN L213:** This line contributes implementation detail or declarative structure to the file.
  **CN L213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L214:** This line contributes implementation detail or declarative structure to the file.
  **CN L214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L215:** This line contributes implementation detail or declarative structure to the file.
  **CN L215:** 这一行为文件补充了实现细节或声明式结构。
- **EN L216:** This line contributes implementation detail or declarative structure to the file.
  **CN L216:** 这一行为文件补充了实现细节或声明式结构。

### Lines 217-228 / 第 217-228 行

```tablegen
 217:     Result type may be a scalar or vector type of scalar element type.
 218: 
 219:     The parameters are:
 220:       * `ptr` - the base address to load from. Must be uniform across subgroup.
 221:       * `cache_control` - an enumerator that sets the cache behaviour
 222: 
 223:     Example:
 224:     ```mlir
 225:       %loaded_a = xevm.blockload %src,
 226:                       <{cache_control=#xevm.load_cache_control<L1uc_L2uc_L3uc>}>
 227:                     : (!llvm.ptr<1>) -> vector<4xi16>
 228:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L217:** This line contributes implementation detail or declarative structure to the file.
  **CN L217:** 这一行为文件补充了实现细节或声明式结构。
- **EN L218:** Blank line used to separate nearby declarations and improve readability.
  **CN L218:** 该空行用于分隔相邻声明并提升可读性。
- **EN L219:** This line contributes implementation detail or declarative structure to the file.
  **CN L219:** 这一行为文件补充了实现细节或声明式结构。
- **EN L220:** This comment states: “`ptr` - the base address to load from. Must be uniform across subgroup.”, documenting the intent of the surrounding code.
  **CN L220:** 该注释写道：“`ptr` - the base address to load from. Must be uniform across subgroup.”，用于说明周围代码的意图。
- **EN L221:** This comment states: “`cache_control` - an enumerator that sets the cache behaviour”, documenting the intent of the surrounding code.
  **CN L221:** 该注释写道：“`cache_control` - an enumerator that sets the cache behaviour”，用于说明周围代码的意图。
- **EN L222:** Blank line used to separate nearby declarations and improve readability.
  **CN L222:** 该空行用于分隔相邻声明并提升可读性。
- **EN L223:** This line contributes implementation detail or declarative structure to the file.
  **CN L223:** 这一行为文件补充了实现细节或声明式结构。
- **EN L224:** This line contributes implementation detail or declarative structure to the file.
  **CN L224:** 这一行为文件补充了实现细节或声明式结构。
- **EN L225:** This line contributes implementation detail or declarative structure to the file.
  **CN L225:** 这一行为文件补充了实现细节或声明式结构。
- **EN L226:** This line contributes implementation detail or declarative structure to the file.
  **CN L226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L227:** This line contributes implementation detail or declarative structure to the file.
  **CN L227:** 这一行为文件补充了实现细节或声明式结构。
- **EN L228:** This line contributes implementation detail or declarative structure to the file.
  **CN L228:** 这一行为文件补充了实现细节或声明式结构。

### Lines 229-240 / 第 229-240 行

```tablegen
 229:   }];
 230:   let assemblyFormat = [{
 231:     operands prop-dict attr-dict `:` functional-type(operands, results)
 232:   }];
 233:   let hasVerifier = 1;
 234: }
 235: 
 236: def XeVM_BlockStoreOp
 237:     : XeVM_Op<"blockstore">,
 238:       Arguments<(ins Arg<LLVM_AnyPointer, "", [MemWrite]>:$ptr,
 239:           AnyTypeOf<[XeVM_1DBlockElemType,
 240:                      FixedVectorOfRankAndType<[1],
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L229:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L229:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L230:** This line contributes implementation detail or declarative structure to the file.
  **CN L230:** 这一行为文件补充了实现细节或声明式结构。
- **EN L231:** This line contributes to the declaration or call of `type`.
  **CN L231:** 这一行为 `type` 的声明或调用提供内容。
- **EN L232:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L232:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L233:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L233:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L234:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L234:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L235:** Blank line used to separate nearby declarations and improve readability.
  **CN L235:** 该空行用于分隔相邻声明并提升可读性。
- **EN L236:** This TableGen `def` record introduces `XeVM_BlockStoreOp`, which later participates in generated MLIR code.
  **CN L236:** 该 TableGen `def` 记录引入了 `XeVM_BlockStoreOp`，后续会参与生成的 MLIR 代码。
- **EN L237:** This line contributes implementation detail or declarative structure to the file.
  **CN L237:** 这一行为文件补充了实现细节或声明式结构。
- **EN L238:** This line contributes implementation detail or declarative structure to the file.
  **CN L238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L239:** This line contributes implementation detail or declarative structure to the file.
  **CN L239:** 这一行为文件补充了实现细节或声明式结构。
- **EN L240:** This line contributes implementation detail or declarative structure to the file.
  **CN L240:** 这一行为文件补充了实现细节或声明式结构。

### Lines 241-252 / 第 241-252 行

```tablegen
 241:                                               [XeVM_1DBlockElemType]>]>:$val,
 242:           OptionalAttr<XeVM_StoreCacheControlAttr>:$cache_control)> {
 243:   let summary = "subgroup block store";
 244:   let description = [{
 245:     Writes one or more components of `val` for each invocation
 246:     in the subgroup to the specified `ptr` as a block operation.
 247:     The data is written strided, so the first value is written to:
 248:     ```
 249:       ptr[ SubgroupLocalInvocationId ]
 250:     ```
 251:     and the second value is written to:
 252:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L241:** This line contributes implementation detail or declarative structure to the file.
  **CN L241:** 这一行为文件补充了实现细节或声明式结构。
- **EN L242:** This line contributes implementation detail or declarative structure to the file.
  **CN L242:** 这一行为文件补充了实现细节或声明式结构。
- **EN L243:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L243:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L244:** This line contributes implementation detail or declarative structure to the file.
  **CN L244:** 这一行为文件补充了实现细节或声明式结构。
- **EN L245:** This line contributes implementation detail or declarative structure to the file.
  **CN L245:** 这一行为文件补充了实现细节或声明式结构。
- **EN L246:** This line contributes implementation detail or declarative structure to the file.
  **CN L246:** 这一行为文件补充了实现细节或声明式结构。
- **EN L247:** This line contributes implementation detail or declarative structure to the file.
  **CN L247:** 这一行为文件补充了实现细节或声明式结构。
- **EN L248:** This line contributes implementation detail or declarative structure to the file.
  **CN L248:** 这一行为文件补充了实现细节或声明式结构。
- **EN L249:** This line contributes implementation detail or declarative structure to the file.
  **CN L249:** 这一行为文件补充了实现细节或声明式结构。
- **EN L250:** This line contributes implementation detail or declarative structure to the file.
  **CN L250:** 这一行为文件补充了实现细节或声明式结构。
- **EN L251:** This line contributes implementation detail or declarative structure to the file.
  **CN L251:** 这一行为文件补充了实现细节或声明式结构。
- **EN L252:** This line contributes implementation detail or declarative structure to the file.
  **CN L252:** 这一行为文件补充了实现细节或声明式结构。

### Lines 253-264 / 第 253-264 行

```tablegen
 253:       ptr[ SubgroupLocalInvocationId + SubgroupMaxSize ]
 254:     ```
 255:     `val` type may be a scalar or vector type of scalar element type.
 256: 
 257:     The parameters are:
 258:       * `ptr` - the base address to store to. Must be uniform across subgroup.
 259:       * `val` - the value to store
 260:       * `cache_control` - an enumerator that sets the cache behaviour
 261: 
 262:     Example:
 263:     ```mlir
 264:       xevm.blockstore %ptr, %val
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L253:** This line contributes implementation detail or declarative structure to the file.
  **CN L253:** 这一行为文件补充了实现细节或声明式结构。
- **EN L254:** This line contributes implementation detail or declarative structure to the file.
  **CN L254:** 这一行为文件补充了实现细节或声明式结构。
- **EN L255:** This line contributes implementation detail or declarative structure to the file.
  **CN L255:** 这一行为文件补充了实现细节或声明式结构。
- **EN L256:** Blank line used to separate nearby declarations and improve readability.
  **CN L256:** 该空行用于分隔相邻声明并提升可读性。
- **EN L257:** This line contributes implementation detail or declarative structure to the file.
  **CN L257:** 这一行为文件补充了实现细节或声明式结构。
- **EN L258:** This comment states: “`ptr` - the base address to store to. Must be uniform across subgroup.”, documenting the intent of the surrounding code.
  **CN L258:** 该注释写道：“`ptr` - the base address to store to. Must be uniform across subgroup.”，用于说明周围代码的意图。
- **EN L259:** This comment states: “`val` - the value to store”, documenting the intent of the surrounding code.
  **CN L259:** 该注释写道：“`val` - the value to store”，用于说明周围代码的意图。
- **EN L260:** This comment states: “`cache_control` - an enumerator that sets the cache behaviour”, documenting the intent of the surrounding code.
  **CN L260:** 该注释写道：“`cache_control` - an enumerator that sets the cache behaviour”，用于说明周围代码的意图。
- **EN L261:** Blank line used to separate nearby declarations and improve readability.
  **CN L261:** 该空行用于分隔相邻声明并提升可读性。
- **EN L262:** This line contributes implementation detail or declarative structure to the file.
  **CN L262:** 这一行为文件补充了实现细节或声明式结构。
- **EN L263:** This line contributes implementation detail or declarative structure to the file.
  **CN L263:** 这一行为文件补充了实现细节或声明式结构。
- **EN L264:** This line contributes implementation detail or declarative structure to the file.
  **CN L264:** 这一行为文件补充了实现细节或声明式结构。

### Lines 265-276 / 第 265-276 行

```tablegen
 265:         <{cache_control=#xevm.store_cache_control<L1uc_L2uc_L3uc>}>
 266:         : (!llvm.ptr<1>, vector<4xi16>)
 267:     ```
 268:   }];
 269: 
 270:   let assemblyFormat = [{
 271:     operands prop-dict attr-dict `:` `(` type(operands) `)`
 272:   }];
 273:   let hasVerifier = 1;
 274: }
 275: 
 276: def XeVM_BlockLoad2dOp
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L265:** This line contributes implementation detail or declarative structure to the file.
  **CN L265:** 这一行为文件补充了实现细节或声明式结构。
- **EN L266:** This line contributes implementation detail or declarative structure to the file.
  **CN L266:** 这一行为文件补充了实现细节或声明式结构。
- **EN L267:** This line contributes implementation detail or declarative structure to the file.
  **CN L267:** 这一行为文件补充了实现细节或声明式结构。
- **EN L268:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L268:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L269:** Blank line used to separate nearby declarations and improve readability.
  **CN L269:** 该空行用于分隔相邻声明并提升可读性。
- **EN L270:** This line contributes implementation detail or declarative structure to the file.
  **CN L270:** 这一行为文件补充了实现细节或声明式结构。
- **EN L271:** This line contributes to the declaration or call of `type`.
  **CN L271:** 这一行为 `type` 的声明或调用提供内容。
- **EN L272:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L272:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L273:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L273:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L274:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L274:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L275:** Blank line used to separate nearby declarations and improve readability.
  **CN L275:** 该空行用于分隔相邻声明并提升可读性。
- **EN L276:** This TableGen `def` record introduces `XeVM_BlockLoad2dOp`, which later participates in generated MLIR code.
  **CN L276:** 该 TableGen `def` 记录引入了 `XeVM_BlockLoad2dOp`，后续会参与生成的 MLIR 代码。

### Lines 277-288 / 第 277-288 行

```tablegen
 277:     : XeVM_Op<"blockload2d">,
 278:       Results<(outs FixedVectorOfRankAndType<[1], [XeVM_ElemType]>:$res)>,
 279:       Arguments<(ins Arg<LLVM_AnyPointer, "", [MemRead]>:$ptr, I32:$base_width,
 280:           I32:$base_height, I32:$base_pitch, I32:$x, I32:$y,
 281:           I32Attr:$elem_size_in_bits, I32Attr:$tile_width, I32Attr:$tile_height,
 282:           I32Attr:$v_blocks, I1Attr:$transpose, I1Attr:$pack_register,
 283:           OptionalAttr<XeVM_LoadCacheControlAttr>:$cache_control)> {
 284: 
 285:   let summary = "2D block load";
 286: 
 287:   let description = [{
 288:     The `xevm.blockload2d` operation loads a two dimensional matrix tile
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L277:** This line contributes implementation detail or declarative structure to the file.
  **CN L277:** 这一行为文件补充了实现细节或声明式结构。
- **EN L278:** This line contributes implementation detail or declarative structure to the file.
  **CN L278:** 这一行为文件补充了实现细节或声明式结构。
- **EN L279:** This line contributes implementation detail or declarative structure to the file.
  **CN L279:** 这一行为文件补充了实现细节或声明式结构。
- **EN L280:** This line contributes implementation detail or declarative structure to the file.
  **CN L280:** 这一行为文件补充了实现细节或声明式结构。
- **EN L281:** This line contributes implementation detail or declarative structure to the file.
  **CN L281:** 这一行为文件补充了实现细节或声明式结构。
- **EN L282:** This line contributes implementation detail or declarative structure to the file.
  **CN L282:** 这一行为文件补充了实现细节或声明式结构。
- **EN L283:** This line contributes implementation detail or declarative structure to the file.
  **CN L283:** 这一行为文件补充了实现细节或声明式结构。
- **EN L284:** Blank line used to separate nearby declarations and improve readability.
  **CN L284:** 该空行用于分隔相邻声明并提升可读性。
- **EN L285:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L285:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L286:** Blank line used to separate nearby declarations and improve readability.
  **CN L286:** 该空行用于分隔相邻声明并提升可读性。
- **EN L287:** This line contributes implementation detail or declarative structure to the file.
  **CN L287:** 这一行为文件补充了实现细节或声明式结构。
- **EN L288:** This line contributes implementation detail or declarative structure to the file.
  **CN L288:** 这一行为文件补充了实现细节或声明式结构。

### Lines 289-300 / 第 289-300 行

```tablegen
 289:     from a base matrix residing in global memory. The parameters are:
 290:       * `ptr` - the base address of the base matrix containing the tile to load
 291:       * `base_width` - the width of the base matrix in number of bytes.
 292:       * `base_height` - the number of rows in the base matrix
 293:       * `base_pitch` - the physical stride between the first columns of the current
 294:         row and the subsequent row in number of bytes.
 295:       * `x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of
 296:         the tile to load in number of elements.
 297:       * `elem_size_in_bits` - the size in bits of the matrix element type
 298:         - 32 for f32, tf32
 299:         - 16 for f16, int16, bf16
 300:         - 8 for int8
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L289:** This line contributes implementation detail or declarative structure to the file.
  **CN L289:** 这一行为文件补充了实现细节或声明式结构。
- **EN L290:** This comment states: “`ptr` - the base address of the base matrix containing the tile to load”, documenting the intent of the surrounding code.
  **CN L290:** 该注释写道：“`ptr` - the base address of the base matrix containing the tile to load”，用于说明周围代码的意图。
- **EN L291:** This comment states: “`base_width` - the width of the base matrix in number of bytes.”, documenting the intent of the surrounding code.
  **CN L291:** 该注释写道：“`base_width` - the width of the base matrix in number of bytes.”，用于说明周围代码的意图。
- **EN L292:** This comment states: “`base_height` - the number of rows in the base matrix”, documenting the intent of the surrounding code.
  **CN L292:** 该注释写道：“`base_height` - the number of rows in the base matrix”，用于说明周围代码的意图。
- **EN L293:** This comment states: “`base_pitch` - the physical stride between the first columns of the current”, documenting the intent of the surrounding code.
  **CN L293:** 该注释写道：“`base_pitch` - the physical stride between the first columns of the current”，用于说明周围代码的意图。
- **EN L294:** This line contributes implementation detail or declarative structure to the file.
  **CN L294:** 这一行为文件补充了实现细节或声明式结构。
- **EN L295:** This comment states: “`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of”, documenting the intent of the surrounding code.
  **CN L295:** 该注释写道：“`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of”，用于说明周围代码的意图。
- **EN L296:** This line contributes implementation detail or declarative structure to the file.
  **CN L296:** 这一行为文件补充了实现细节或声明式结构。
- **EN L297:** This comment states: “`elem_size_in_bits` - the size in bits of the matrix element type”, documenting the intent of the surrounding code.
  **CN L297:** 该注释写道：“`elem_size_in_bits` - the size in bits of the matrix element type”，用于说明周围代码的意图。
- **EN L298:** This line contributes implementation detail or declarative structure to the file.
  **CN L298:** 这一行为文件补充了实现细节或声明式结构。
- **EN L299:** This line contributes implementation detail or declarative structure to the file.
  **CN L299:** 这一行为文件补充了实现细节或声明式结构。
- **EN L300:** This line contributes implementation detail or declarative structure to the file.
  **CN L300:** 这一行为文件补充了实现细节或声明式结构。

### Lines 301-312 / 第 301-312 行

```tablegen
 301:       * `v_blocks` - number of consecutive tiles in innermost dimension direction to load
 302:       * `transpose` - transpose the tile in registers (useful for 32 bit element type)
 303:       * `pack_register` - pack element types narrower than register bit width.
 304:         [M, N] => [M/factor, N, factor] where factor is register_size_in_bits / elem_size_in_bits
 305:       * `cache_control` - an enumerator that sets the cache behaviour
 306: 
 307:     Notes:
 308:       - the `transpose` and `pack_register` parameters are mutual exclusive
 309:       - transposing the tile loaded is used for A matrix in backward path or used for the B matrix operand
 310:         (D = C + A * B), where A has row-major layout and B should have column-major layout in memory.
 311:       - if the tile loaded contains out of bound elements of the matrix, they are filled with 0.
 312: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L301:** This comment states: “`v_blocks` - number of consecutive tiles in innermost dimension direction to load”, documenting the intent of the surrounding code.
  **CN L301:** 该注释写道：“`v_blocks` - number of consecutive tiles in innermost dimension direction to load”，用于说明周围代码的意图。
- **EN L302:** This comment states: “`transpose` - transpose the tile in registers (useful for 32 bit element type)”, documenting the intent of the surrounding code.
  **CN L302:** 该注释写道：“`transpose` - transpose the tile in registers (useful for 32 bit element type)”，用于说明周围代码的意图。
- **EN L303:** This comment states: “`pack_register` - pack element types narrower than register bit width.”, documenting the intent of the surrounding code.
  **CN L303:** 该注释写道：“`pack_register` - pack element types narrower than register bit width.”，用于说明周围代码的意图。
- **EN L304:** This line contributes implementation detail or declarative structure to the file.
  **CN L304:** 这一行为文件补充了实现细节或声明式结构。
- **EN L305:** This comment states: “`cache_control` - an enumerator that sets the cache behaviour”, documenting the intent of the surrounding code.
  **CN L305:** 该注释写道：“`cache_control` - an enumerator that sets the cache behaviour”，用于说明周围代码的意图。
- **EN L306:** Blank line used to separate nearby declarations and improve readability.
  **CN L306:** 该空行用于分隔相邻声明并提升可读性。
- **EN L307:** This line contributes implementation detail or declarative structure to the file.
  **CN L307:** 这一行为文件补充了实现细节或声明式结构。
- **EN L308:** This line contributes implementation detail or declarative structure to the file.
  **CN L308:** 这一行为文件补充了实现细节或声明式结构。
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
 313:     Example:
 314:     ```mlir
 315:       %base_width_a = arith.constant 32 : i32
 316:       %base_height_a = arith.constant 8 : i32
 317:       %base_pitch_a = arith.constant 32 : i32
 318:       %x = arith.constant 0 : i32
 319:       %y = arith.constant 0 : i32
 320:       %loaded_a = xevm.blockload2d %src, %base_width_a, %base_height_a, %base_pitch_a, %x, %y
 321:                     <{elem_size_in_bits=16 : i32, tile_width=16 : i32, tile_height=8 : i32,
 322:                       v_blocks=1 : i32, transpose=false : i32, pack_register=false,
 323:                       cache_control=#xevm.load_cache_control<Default>}>
 324:                     : (!llvm.ptr<1>, i32, i32, i32, i32, i32) -> vector<8xi16>
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L313:** This line contributes implementation detail or declarative structure to the file.
  **CN L313:** 这一行为文件补充了实现细节或声明式结构。
- **EN L314:** This line contributes implementation detail or declarative structure to the file.
  **CN L314:** 这一行为文件补充了实现细节或声明式结构。
- **EN L315:** This line contributes implementation detail or declarative structure to the file.
  **CN L315:** 这一行为文件补充了实现细节或声明式结构。
- **EN L316:** This line contributes implementation detail or declarative structure to the file.
  **CN L316:** 这一行为文件补充了实现细节或声明式结构。
- **EN L317:** This line contributes implementation detail or declarative structure to the file.
  **CN L317:** 这一行为文件补充了实现细节或声明式结构。
- **EN L318:** This line contributes implementation detail or declarative structure to the file.
  **CN L318:** 这一行为文件补充了实现细节或声明式结构。
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
 325:     ```
 326:   }];
 327: 
 328:   let assemblyFormat = [{
 329:     operands prop-dict attr-dict `:` functional-type(operands, results)
 330:   }];
 331: 
 332:   let extraClassDeclaration = extraBaseClassDeclaration#[{
 333:   }];
 334: 
 335:   let hasVerifier = 1;
 336: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L325:** This line contributes implementation detail or declarative structure to the file.
  **CN L325:** 这一行为文件补充了实现细节或声明式结构。
- **EN L326:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L326:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L327:** Blank line used to separate nearby declarations and improve readability.
  **CN L327:** 该空行用于分隔相邻声明并提升可读性。
- **EN L328:** This line contributes implementation detail or declarative structure to the file.
  **CN L328:** 这一行为文件补充了实现细节或声明式结构。
- **EN L329:** This line contributes to the declaration or call of `type`.
  **CN L329:** 这一行为 `type` 的声明或调用提供内容。
- **EN L330:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L330:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L331:** Blank line used to separate nearby declarations and improve readability.
  **CN L331:** 该空行用于分隔相邻声明并提升可读性。
- **EN L332:** This line contributes implementation detail or declarative structure to the file.
  **CN L332:** 这一行为文件补充了实现细节或声明式结构。
- **EN L333:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L333:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L334:** Blank line used to separate nearby declarations and improve readability.
  **CN L334:** 该空行用于分隔相邻声明并提升可读性。
- **EN L335:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L335:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L336:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L336:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 337-348 / 第 337-348 行

```tablegen
 337: 
 338: def XeVM_BlockStore2dOp
 339:     : XeVM_Op<"blockstore2d">,
 340:       Arguments<(ins Arg<LLVM_AnyPointer, "", [MemWrite]>:$ptr, I32:$base_width,
 341:           I32:$base_height, I32:$base_pitch, I32:$x, I32:$y,
 342:           I32Attr:$elem_size_in_bits, I32Attr:$tile_width, I32Attr:$tile_height,
 343:           FixedVectorOfRankAndType<[1], [XeVM_ElemType]>:$stored_val,
 344:           OptionalAttr<XeVM_StoreCacheControlAttr>:$cache_control)> {
 345: 
 346:   let summary = "2D block store";
 347: 
 348:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L337:** Blank line used to separate nearby declarations and improve readability.
  **CN L337:** 该空行用于分隔相邻声明并提升可读性。
- **EN L338:** This TableGen `def` record introduces `XeVM_BlockStore2dOp`, which later participates in generated MLIR code.
  **CN L338:** 该 TableGen `def` 记录引入了 `XeVM_BlockStore2dOp`，后续会参与生成的 MLIR 代码。
- **EN L339:** This line contributes implementation detail or declarative structure to the file.
  **CN L339:** 这一行为文件补充了实现细节或声明式结构。
- **EN L340:** This line contributes implementation detail or declarative structure to the file.
  **CN L340:** 这一行为文件补充了实现细节或声明式结构。
- **EN L341:** This line contributes implementation detail or declarative structure to the file.
  **CN L341:** 这一行为文件补充了实现细节或声明式结构。
- **EN L342:** This line contributes implementation detail or declarative structure to the file.
  **CN L342:** 这一行为文件补充了实现细节或声明式结构。
- **EN L343:** This line contributes implementation detail or declarative structure to the file.
  **CN L343:** 这一行为文件补充了实现细节或声明式结构。
- **EN L344:** This line contributes implementation detail or declarative structure to the file.
  **CN L344:** 这一行为文件补充了实现细节或声明式结构。
- **EN L345:** Blank line used to separate nearby declarations and improve readability.
  **CN L345:** 该空行用于分隔相邻声明并提升可读性。
- **EN L346:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L346:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L347:** Blank line used to separate nearby declarations and improve readability.
  **CN L347:** 该空行用于分隔相邻声明并提升可读性。
- **EN L348:** This line contributes implementation detail or declarative structure to the file.
  **CN L348:** 这一行为文件补充了实现细节或声明式结构。

### Lines 349-360 / 第 349-360 行

```tablegen
 349:     The `xevm.blockstore2d` operation stores a two dimensional tile into a
 350:     larger matrix residing in global memory. The parameters are:
 351:       * `ptr` - the base address of the target matrix where to store the tile
 352:       * `base_width` - the width of the base matrix in number of bytes.
 353:       * `base_height` - the number of rows in the base matrix
 354:       * `base_pitch` - the physical stride between the first columns of the current
 355:         row and the subsequent row in number of bytes.
 356:       * `x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of the tile to store
 357:       in number of elements.
 358:       * `elem_size_in_bits` - the size in bits of the matrix element
 359:         - 32 for f32, tf32
 360:         - 16 for f16, int16, bf16
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L349:** This line contributes implementation detail or declarative structure to the file.
  **CN L349:** 这一行为文件补充了实现细节或声明式结构。
- **EN L350:** This line contributes implementation detail or declarative structure to the file.
  **CN L350:** 这一行为文件补充了实现细节或声明式结构。
- **EN L351:** This comment states: “`ptr` - the base address of the target matrix where to store the tile”, documenting the intent of the surrounding code.
  **CN L351:** 该注释写道：“`ptr` - the base address of the target matrix where to store the tile”，用于说明周围代码的意图。
- **EN L352:** This comment states: “`base_width` - the width of the base matrix in number of bytes.”, documenting the intent of the surrounding code.
  **CN L352:** 该注释写道：“`base_width` - the width of the base matrix in number of bytes.”，用于说明周围代码的意图。
- **EN L353:** This comment states: “`base_height` - the number of rows in the base matrix”, documenting the intent of the surrounding code.
  **CN L353:** 该注释写道：“`base_height` - the number of rows in the base matrix”，用于说明周围代码的意图。
- **EN L354:** This comment states: “`base_pitch` - the physical stride between the first columns of the current”, documenting the intent of the surrounding code.
  **CN L354:** 该注释写道：“`base_pitch` - the physical stride between the first columns of the current”，用于说明周围代码的意图。
- **EN L355:** This line contributes implementation detail or declarative structure to the file.
  **CN L355:** 这一行为文件补充了实现细节或声明式结构。
- **EN L356:** This comment states: “`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of the tile to store”, documenting the intent of the surrounding code.
  **CN L356:** 该注释写道：“`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of the tile to store”，用于说明周围代码的意图。
- **EN L357:** This line contributes implementation detail or declarative structure to the file.
  **CN L357:** 这一行为文件补充了实现细节或声明式结构。
- **EN L358:** This comment states: “`elem_size_in_bits` - the size in bits of the matrix element”, documenting the intent of the surrounding code.
  **CN L358:** 该注释写道：“`elem_size_in_bits` - the size in bits of the matrix element”，用于说明周围代码的意图。
- **EN L359:** This line contributes implementation detail or declarative structure to the file.
  **CN L359:** 这一行为文件补充了实现细节或声明式结构。
- **EN L360:** This line contributes implementation detail or declarative structure to the file.
  **CN L360:** 这一行为文件补充了实现细节或声明式结构。

### Lines 361-372 / 第 361-372 行

```tablegen
 361:         - 8 for int8
 362:       * `cache_control` - an enumerator that sets the cache behaviour
 363:       * `stored_val` - the tile to store
 364: 
 365:     Example:
 366:     ```mlir
 367:       %base_width_c = arith.constant 64 : i32
 368:       %base_height_c = arith.constant 8 : i32
 369:       %base_pitch_c = arith.constant 64 : i32
 370:       %x = arith.constant 0 : i32
 371:       %y = arith.constant 0 : i32
 372:       xevm.blockstore2d %dst, %base_width_c, %base_height_c, %base_pitch_c, %x, %y, %src
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L361:** This line contributes implementation detail or declarative structure to the file.
  **CN L361:** 这一行为文件补充了实现细节或声明式结构。
- **EN L362:** This comment states: “`cache_control` - an enumerator that sets the cache behaviour”, documenting the intent of the surrounding code.
  **CN L362:** 该注释写道：“`cache_control` - an enumerator that sets the cache behaviour”，用于说明周围代码的意图。
- **EN L363:** This comment states: “`stored_val` - the tile to store”, documenting the intent of the surrounding code.
  **CN L363:** 该注释写道：“`stored_val` - the tile to store”，用于说明周围代码的意图。
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
- **EN L369:** This line contributes implementation detail or declarative structure to the file.
  **CN L369:** 这一行为文件补充了实现细节或声明式结构。
- **EN L370:** This line contributes implementation detail or declarative structure to the file.
  **CN L370:** 这一行为文件补充了实现细节或声明式结构。
- **EN L371:** This line contributes implementation detail or declarative structure to the file.
  **CN L371:** 这一行为文件补充了实现细节或声明式结构。
- **EN L372:** This line contributes implementation detail or declarative structure to the file.
  **CN L372:** 这一行为文件补充了实现细节或声明式结构。

### Lines 373-384 / 第 373-384 行

```tablegen
 373:         <{elem_size_in_bits=32 : i32, tile_width=16 : i32, tile_height=8 : i32,
 374:           cache_control=#xevm.load_cache_control<Default>}>
 375:         : (!llvm.ptr<1>, i32, i32, i32, i32, i32, vector<8xi32>)
 376:     ```
 377:   }];
 378: 
 379:   let assemblyFormat = [{
 380:     operands prop-dict attr-dict `:` `(` type(operands) `)`
 381:   }];
 382: 
 383:   let extraClassDeclaration = extraBaseClassDeclaration#[{
 384:     /// Default value for v_blocks is 1.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L373:** This line contributes implementation detail or declarative structure to the file.
  **CN L373:** 这一行为文件补充了实现细节或声明式结构。
- **EN L374:** This line contributes implementation detail or declarative structure to the file.
  **CN L374:** 这一行为文件补充了实现细节或声明式结构。
- **EN L375:** This line contributes implementation detail or declarative structure to the file.
  **CN L375:** 这一行为文件补充了实现细节或声明式结构。
- **EN L376:** This line contributes implementation detail or declarative structure to the file.
  **CN L376:** 这一行为文件补充了实现细节或声明式结构。
- **EN L377:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L377:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L378:** Blank line used to separate nearby declarations and improve readability.
  **CN L378:** 该空行用于分隔相邻声明并提升可读性。
- **EN L379:** This line contributes implementation detail or declarative structure to the file.
  **CN L379:** 这一行为文件补充了实现细节或声明式结构。
- **EN L380:** This line contributes to the declaration or call of `type`.
  **CN L380:** 这一行为 `type` 的声明或调用提供内容。
- **EN L381:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L381:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L382:** Blank line used to separate nearby declarations and improve readability.
  **CN L382:** 该空行用于分隔相邻声明并提升可读性。
- **EN L383:** This line contributes implementation detail or declarative structure to the file.
  **CN L383:** 这一行为文件补充了实现细节或声明式结构。
- **EN L384:** This comment states: “Default value for v_blocks is 1.”, documenting the intent of the surrounding code.
  **CN L384:** 该注释写道：“Default value for v_blocks is 1.”，用于说明周围代码的意图。

### Lines 385-396 / 第 385-396 行

```tablegen
 385:     constexpr uint32_t getVBlocks() {
 386:       return 1;
 387:     }
 388:   }];
 389: 
 390:   let hasVerifier = 1;
 391: }
 392: 
 393: def MemScopeLane : I32EnumAttrCase<"LANE", 0, "lane">;
 394: def MemScopeSubgroup : I32EnumAttrCase<"SUBGROUP", 1, "subgroup">;
 395: def MemScopeWorkgroup : I32EnumAttrCase<"WORKGROUP", 2, "workgroup">;
 396: def MemScopeCluster : I32EnumAttrCase<"CLUSTER", 3, "cluster">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L385:** This line contributes to the declaration or call of `getVBlocks`.
  **CN L385:** 这一行为 `getVBlocks` 的声明或调用提供内容。
- **EN L386:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L386:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L387:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L387:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L388:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L388:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L389:** Blank line used to separate nearby declarations and improve readability.
  **CN L389:** 该空行用于分隔相邻声明并提升可读性。
- **EN L390:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L390:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L391:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L391:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L392:** Blank line used to separate nearby declarations and improve readability.
  **CN L392:** 该空行用于分隔相邻声明并提升可读性。
- **EN L393:** This TableGen `def` record introduces `MemScopeLane`, which later participates in generated MLIR code.
  **CN L393:** 该 TableGen `def` 记录引入了 `MemScopeLane`，后续会参与生成的 MLIR 代码。
- **EN L394:** This TableGen `def` record introduces `MemScopeSubgroup`, which later participates in generated MLIR code.
  **CN L394:** 该 TableGen `def` 记录引入了 `MemScopeSubgroup`，后续会参与生成的 MLIR 代码。
- **EN L395:** This TableGen `def` record introduces `MemScopeWorkgroup`, which later participates in generated MLIR code.
  **CN L395:** 该 TableGen `def` 记录引入了 `MemScopeWorkgroup`，后续会参与生成的 MLIR 代码。
- **EN L396:** This TableGen `def` record introduces `MemScopeCluster`, which later participates in generated MLIR code.
  **CN L396:** 该 TableGen `def` 记录引入了 `MemScopeCluster`，后续会参与生成的 MLIR 代码。

### Lines 397-408 / 第 397-408 行

```tablegen
 397: def MemScopeDevice : I32EnumAttrCase<"DEVICE", 4, "device">;
 398: def MemScopeSystem : I32EnumAttrCase<"SYSTEM", 5, "system">;
 399: 
 400: def XeVM_MemScope
 401:     : I32EnumAttr<"MemScope", "XeVM memory scope",
 402:                   [MemScopeLane, MemScopeSubgroup, MemScopeWorkgroup,
 403:                    MemScopeCluster, MemScopeDevice, MemScopeSystem]> {
 404:   let genSpecializedAttr = 0;
 405:   let cppNamespace = "::mlir::xevm";
 406: }
 407: def XeVM_MemScopeAttr : EnumAttr<XeVM_Dialect, XeVM_MemScope, "mem_scope"> {
 408:   let summary = [{Describe memory scopes}];
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L397:** This TableGen `def` record introduces `MemScopeDevice`, which later participates in generated MLIR code.
  **CN L397:** 该 TableGen `def` 记录引入了 `MemScopeDevice`，后续会参与生成的 MLIR 代码。
- **EN L398:** This TableGen `def` record introduces `MemScopeSystem`, which later participates in generated MLIR code.
  **CN L398:** 该 TableGen `def` 记录引入了 `MemScopeSystem`，后续会参与生成的 MLIR 代码。
- **EN L399:** Blank line used to separate nearby declarations and improve readability.
  **CN L399:** 该空行用于分隔相邻声明并提升可读性。
- **EN L400:** This TableGen `def` record introduces `XeVM_MemScope`, which later participates in generated MLIR code.
  **CN L400:** 该 TableGen `def` 记录引入了 `XeVM_MemScope`，后续会参与生成的 MLIR 代码。
- **EN L401:** This line contributes implementation detail or declarative structure to the file.
  **CN L401:** 这一行为文件补充了实现细节或声明式结构。
- **EN L402:** This line contributes implementation detail or declarative structure to the file.
  **CN L402:** 这一行为文件补充了实现细节或声明式结构。
- **EN L403:** This line contributes implementation detail or declarative structure to the file.
  **CN L403:** 这一行为文件补充了实现细节或声明式结构。
- **EN L404:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L404:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L405:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L405:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L406:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L406:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L407:** This TableGen `def` record introduces `XeVM_MemScopeAttr`, which later participates in generated MLIR code.
  **CN L407:** 该 TableGen `def` 记录引入了 `XeVM_MemScopeAttr`，后续会参与生成的 MLIR 代码。
- **EN L408:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L408:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 409-420 / 第 409-420 行

```tablegen
 409:   let assemblyFormat = "`<` $value `>`";
 410: }
 411: 
 412: def AddrSpacePrivate : I32EnumAttrCase<"PRIVATE", 0, "private">;
 413: def AddrSpaceGlobal : I32EnumAttrCase<"GLOBAL", 1, "global">;
 414: def AddrSpaceConstant : I32EnumAttrCase<"CONSTANT", 2, "constant">;
 415: def AddrSpaceShared : I32EnumAttrCase<"SHARED", 3, "shared">;
 416: def AddrSpaceGeneric : I32EnumAttrCase<"GENERIC", 4, "generic">;
 417: 
 418: def XeVM_AddrSpace
 419:     : I32EnumAttr<"AddrSpace", "Address spaces",
 420:                   [AddrSpacePrivate, AddrSpaceGlobal, AddrSpaceConstant,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L409:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L409:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L410:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L410:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L411:** Blank line used to separate nearby declarations and improve readability.
  **CN L411:** 该空行用于分隔相邻声明并提升可读性。
- **EN L412:** This TableGen `def` record introduces `AddrSpacePrivate`, which later participates in generated MLIR code.
  **CN L412:** 该 TableGen `def` 记录引入了 `AddrSpacePrivate`，后续会参与生成的 MLIR 代码。
- **EN L413:** This TableGen `def` record introduces `AddrSpaceGlobal`, which later participates in generated MLIR code.
  **CN L413:** 该 TableGen `def` 记录引入了 `AddrSpaceGlobal`，后续会参与生成的 MLIR 代码。
- **EN L414:** This TableGen `def` record introduces `AddrSpaceConstant`, which later participates in generated MLIR code.
  **CN L414:** 该 TableGen `def` 记录引入了 `AddrSpaceConstant`，后续会参与生成的 MLIR 代码。
- **EN L415:** This TableGen `def` record introduces `AddrSpaceShared`, which later participates in generated MLIR code.
  **CN L415:** 该 TableGen `def` 记录引入了 `AddrSpaceShared`，后续会参与生成的 MLIR 代码。
- **EN L416:** This TableGen `def` record introduces `AddrSpaceGeneric`, which later participates in generated MLIR code.
  **CN L416:** 该 TableGen `def` 记录引入了 `AddrSpaceGeneric`，后续会参与生成的 MLIR 代码。
- **EN L417:** Blank line used to separate nearby declarations and improve readability.
  **CN L417:** 该空行用于分隔相邻声明并提升可读性。
- **EN L418:** This TableGen `def` record introduces `XeVM_AddrSpace`, which later participates in generated MLIR code.
  **CN L418:** 该 TableGen `def` 记录引入了 `XeVM_AddrSpace`，后续会参与生成的 MLIR 代码。
- **EN L419:** This line contributes implementation detail or declarative structure to the file.
  **CN L419:** 这一行为文件补充了实现细节或声明式结构。
- **EN L420:** This line contributes implementation detail or declarative structure to the file.
  **CN L420:** 这一行为文件补充了实现细节或声明式结构。

### Lines 421-432 / 第 421-432 行

```tablegen
 421:                    AddrSpaceShared, AddrSpaceGeneric]> {
 422:   let genSpecializedAttr = 0;
 423:   let cppNamespace = "mlir::xevm";
 424: }
 425: def XeVM_AddrSpaceAttr : EnumAttr<XeVM_Dialect, XeVM_AddrSpace, "addr_space"> {
 426:   let summary = [{Describe address spaces}];
 427:   let assemblyFormat = "`<` $value `>`";
 428: }
 429: 
 430: def XeVM_MemfenceOp
 431:     : XeVM_Op<"memfence">,
 432:       Arguments<(ins XeVM_MemScopeAttr:$scope,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L421:** This line contributes implementation detail or declarative structure to the file.
  **CN L421:** 这一行为文件补充了实现细节或声明式结构。
- **EN L422:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L422:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L423:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L423:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L424:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L424:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L425:** This TableGen `def` record introduces `XeVM_AddrSpaceAttr`, which later participates in generated MLIR code.
  **CN L425:** 该 TableGen `def` 记录引入了 `XeVM_AddrSpaceAttr`，后续会参与生成的 MLIR 代码。
- **EN L426:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L426:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L427:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L427:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L428:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L428:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L429:** Blank line used to separate nearby declarations and improve readability.
  **CN L429:** 该空行用于分隔相邻声明并提升可读性。
- **EN L430:** This TableGen `def` record introduces `XeVM_MemfenceOp`, which later participates in generated MLIR code.
  **CN L430:** 该 TableGen `def` 记录引入了 `XeVM_MemfenceOp`，后续会参与生成的 MLIR 代码。
- **EN L431:** This line contributes implementation detail or declarative structure to the file.
  **CN L431:** 这一行为文件补充了实现细节或声明式结构。
- **EN L432:** This line contributes implementation detail or declarative structure to the file.
  **CN L432:** 这一行为文件补充了实现细节或声明式结构。

### Lines 433-444 / 第 433-444 行

```tablegen
 433:           DefaultValuedAttr<XeVM_AddrSpaceAttr,
 434:                             "mlir::xevm::AddrSpace::GENERIC">:$addrspace)> {
 435:   let summary = "Work-item's memory fence.";
 436:   let description = [{
 437:     This operation ensures that all prior memory accesses of this
 438:     work-item to `addrspace` are visible to all other work-items in `scope`.
 439:     Parameters description:
 440:       * `scope` - specify the memory scope at which all other work-items should observe
 441:         memory operations prior to the fence.
 442:       * `addrspace` - specify the address space of work-item's memory accesses
 443:         to be affected by the fence.
 444:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L433:** This line contributes implementation detail or declarative structure to the file.
  **CN L433:** 这一行为文件补充了实现细节或声明式结构。
- **EN L434:** This line contributes implementation detail or declarative structure to the file.
  **CN L434:** 这一行为文件补充了实现细节或声明式结构。
- **EN L435:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L435:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L436:** This line contributes implementation detail or declarative structure to the file.
  **CN L436:** 这一行为文件补充了实现细节或声明式结构。
- **EN L437:** This line contributes implementation detail or declarative structure to the file.
  **CN L437:** 这一行为文件补充了实现细节或声明式结构。
- **EN L438:** This line contributes implementation detail or declarative structure to the file.
  **CN L438:** 这一行为文件补充了实现细节或声明式结构。
- **EN L439:** This line contributes implementation detail or declarative structure to the file.
  **CN L439:** 这一行为文件补充了实现细节或声明式结构。
- **EN L440:** This comment states: “`scope` - specify the memory scope at which all other work-items should observe”, documenting the intent of the surrounding code.
  **CN L440:** 该注释写道：“`scope` - specify the memory scope at which all other work-items should observe”，用于说明周围代码的意图。
- **EN L441:** This line contributes implementation detail or declarative structure to the file.
  **CN L441:** 这一行为文件补充了实现细节或声明式结构。
- **EN L442:** This comment states: “`addrspace` - specify the address space of work-item's memory accesses”, documenting the intent of the surrounding code.
  **CN L442:** 该注释写道：“`addrspace` - specify the address space of work-item's memory accesses”，用于说明周围代码的意图。
- **EN L443:** This line contributes implementation detail or declarative structure to the file.
  **CN L443:** 这一行为文件补充了实现细节或声明式结构。
- **EN L444:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L444:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 445-456 / 第 445-456 行

```tablegen
 445:   let assemblyFormat = [{prop-dict  attr-dict}];
 446: 
 447:   let extraClassDeclaration = extraBaseClassDeclaration#[{
 448:   }];
 449: }
 450: 
 451: def XeVM_PrefetchOp
 452:     : XeVM_Op<"prefetch">,
 453:       Arguments<(ins Arg<AnyTypeOf<[LLVM_PointerInAddressSpace<1>,
 454:                                     LLVM_PointerInAddressSpace<4>]>>:$ptr,
 455:           OptionalAttr<XeVM_LoadCacheControlAttr>:$cache_control)> {
 456:   let summary = "Prefetch data into a cache subsystem.";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L445:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L445:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L446:** Blank line used to separate nearby declarations and improve readability.
  **CN L446:** 该空行用于分隔相邻声明并提升可读性。
- **EN L447:** This line contributes implementation detail or declarative structure to the file.
  **CN L447:** 这一行为文件补充了实现细节或声明式结构。
- **EN L448:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L448:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L449:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L449:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L450:** Blank line used to separate nearby declarations and improve readability.
  **CN L450:** 该空行用于分隔相邻声明并提升可读性。
- **EN L451:** This TableGen `def` record introduces `XeVM_PrefetchOp`, which later participates in generated MLIR code.
  **CN L451:** 该 TableGen `def` 记录引入了 `XeVM_PrefetchOp`，后续会参与生成的 MLIR 代码。
- **EN L452:** This line contributes implementation detail or declarative structure to the file.
  **CN L452:** 这一行为文件补充了实现细节或声明式结构。
- **EN L453:** This line contributes implementation detail or declarative structure to the file.
  **CN L453:** 这一行为文件补充了实现细节或声明式结构。
- **EN L454:** This line contributes implementation detail or declarative structure to the file.
  **CN L454:** 这一行为文件补充了实现细节或声明式结构。
- **EN L455:** This line contributes implementation detail or declarative structure to the file.
  **CN L455:** 这一行为文件补充了实现细节或声明式结构。
- **EN L456:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L456:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 457-468 / 第 457-468 行

```tablegen
 457:   let description = [{
 458:     Work-item issues a prefetch from global memory to cache:
 459:       * `ptr` - LLVM pointer with address space. Address space must be 1 (global)
 460:         or 4 (generic)
 461:       * `cache_control` - specify caching options
 462:   }];
 463:   let assemblyFormat = [{
 464:     operands prop-dict attr-dict `:` `(` type(operands) `)`
 465:   }];
 466: 
 467:   let extraClassDeclaration = extraBaseClassDeclaration#[{
 468:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L457:** This line contributes implementation detail or declarative structure to the file.
  **CN L457:** 这一行为文件补充了实现细节或声明式结构。
- **EN L458:** This line contributes implementation detail or declarative structure to the file.
  **CN L458:** 这一行为文件补充了实现细节或声明式结构。
- **EN L459:** This comment states: “`ptr` - LLVM pointer with address space. Address space must be 1 (global)”, documenting the intent of the surrounding code.
  **CN L459:** 该注释写道：“`ptr` - LLVM pointer with address space. Address space must be 1 (global)”，用于说明周围代码的意图。
- **EN L460:** This line contributes implementation detail or declarative structure to the file.
  **CN L460:** 这一行为文件补充了实现细节或声明式结构。
- **EN L461:** This comment states: “`cache_control` - specify caching options”, documenting the intent of the surrounding code.
  **CN L461:** 该注释写道：“`cache_control` - specify caching options”，用于说明周围代码的意图。
- **EN L462:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L462:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L463:** This line contributes implementation detail or declarative structure to the file.
  **CN L463:** 这一行为文件补充了实现细节或声明式结构。
- **EN L464:** This line contributes to the declaration or call of `type`.
  **CN L464:** 这一行为 `type` 的声明或调用提供内容。
- **EN L465:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L465:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L466:** Blank line used to separate nearby declarations and improve readability.
  **CN L466:** 该空行用于分隔相邻声明并提升可读性。
- **EN L467:** This line contributes implementation detail or declarative structure to the file.
  **CN L467:** 这一行为文件补充了实现细节或声明式结构。
- **EN L468:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L468:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 469-480 / 第 469-480 行

```tablegen
 469: }
 470: 
 471: def XeVM_BlockPrefetch2dOp
 472:     : XeVM_Op<"blockprefetch2d">,
 473:       Arguments<(ins LLVM_AnyPointer:$ptr, I32:$base_width, I32:$base_height,
 474:           I32:$base_pitch, I32:$x, I32:$y, I32Attr:$elem_size_in_bits,
 475:           I32Attr:$tile_width, I32Attr:$tile_height, I32Attr:$v_blocks,
 476:           OptionalAttr<XeVM_LoadCacheControlAttr>:$cache_control)> {
 477: 
 478:   let summary = "2D block prefetch";
 479: 
 480:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L469:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L469:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L470:** Blank line used to separate nearby declarations and improve readability.
  **CN L470:** 该空行用于分隔相邻声明并提升可读性。
- **EN L471:** This TableGen `def` record introduces `XeVM_BlockPrefetch2dOp`, which later participates in generated MLIR code.
  **CN L471:** 该 TableGen `def` 记录引入了 `XeVM_BlockPrefetch2dOp`，后续会参与生成的 MLIR 代码。
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
- **EN L477:** Blank line used to separate nearby declarations and improve readability.
  **CN L477:** 该空行用于分隔相邻声明并提升可读性。
- **EN L478:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L478:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L479:** Blank line used to separate nearby declarations and improve readability.
  **CN L479:** 该空行用于分隔相邻声明并提升可读性。
- **EN L480:** This line contributes implementation detail or declarative structure to the file.
  **CN L480:** 这一行为文件补充了实现细节或声明式结构。

### Lines 481-492 / 第 481-492 行

```tablegen
 481:     The `xevm.blockprefetch2d` operation prefetches a two dimensional tile
 482:     from a larger base matrix residing in global memory. The parameters are:
 483:       * `ptr` - the base address of the base matrix containing the tile to prefetch
 484:       * `base_width` - the width of the base matrix in number of bytes.
 485:       * `base_height` - the number of rows in the base matrix
 486:       * `base_pitch` - the physical stride between the first columns of the current
 487:         row and the subsequent row in number of bytes.
 488:       * `x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of tile
 489:         to prefetch in number of elements.
 490:       * `elem_size_in_bits` - the size in bits of the matrix element
 491:         - 32 for f32, bf32
 492:         - 16 for f16, int16, bf16
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L481:** This line contributes implementation detail or declarative structure to the file.
  **CN L481:** 这一行为文件补充了实现细节或声明式结构。
- **EN L482:** This line contributes implementation detail or declarative structure to the file.
  **CN L482:** 这一行为文件补充了实现细节或声明式结构。
- **EN L483:** This comment states: “`ptr` - the base address of the base matrix containing the tile to prefetch”, documenting the intent of the surrounding code.
  **CN L483:** 该注释写道：“`ptr` - the base address of the base matrix containing the tile to prefetch”，用于说明周围代码的意图。
- **EN L484:** This comment states: “`base_width` - the width of the base matrix in number of bytes.”, documenting the intent of the surrounding code.
  **CN L484:** 该注释写道：“`base_width` - the width of the base matrix in number of bytes.”，用于说明周围代码的意图。
- **EN L485:** This comment states: “`base_height` - the number of rows in the base matrix”, documenting the intent of the surrounding code.
  **CN L485:** 该注释写道：“`base_height` - the number of rows in the base matrix”，用于说明周围代码的意图。
- **EN L486:** This comment states: “`base_pitch` - the physical stride between the first columns of the current”, documenting the intent of the surrounding code.
  **CN L486:** 该注释写道：“`base_pitch` - the physical stride between the first columns of the current”，用于说明周围代码的意图。
- **EN L487:** This line contributes implementation detail or declarative structure to the file.
  **CN L487:** 这一行为文件补充了实现细节或声明式结构。
- **EN L488:** This comment states: “`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of tile”, documenting the intent of the surrounding code.
  **CN L488:** 该注释写道：“`x`, `y`, `tile_width`, `tile_height` - the starting offsets and shape of tile”，用于说明周围代码的意图。
- **EN L489:** This line contributes implementation detail or declarative structure to the file.
  **CN L489:** 这一行为文件补充了实现细节或声明式结构。
- **EN L490:** This comment states: “`elem_size_in_bits` - the size in bits of the matrix element”, documenting the intent of the surrounding code.
  **CN L490:** 该注释写道：“`elem_size_in_bits` - the size in bits of the matrix element”，用于说明周围代码的意图。
- **EN L491:** This line contributes implementation detail or declarative structure to the file.
  **CN L491:** 这一行为文件补充了实现细节或声明式结构。
- **EN L492:** This line contributes implementation detail or declarative structure to the file.
  **CN L492:** 这一行为文件补充了实现细节或声明式结构。

### Lines 493-504 / 第 493-504 行

```tablegen
 493:         - 8 for int8, int4, int2
 494:       * `v_blocks` - number of tiles in innermost dimension direction to prefetch
 495:       * `cache_control` - an enumerator that sets the cache behaviour
 496: 
 497:     Example:
 498:     ```mlir
 499:       xevm.blockprefetch2d %ptr, %base_width, %base_height, %base_pitch, %x, %y
 500:         <{elem_size_in_bits=8 : i32, tile_width=32 : i32, tile_height=8 : i32,
 501:           v_blocks=1 : i32, cache_control=#xevm.load_cache_control<L1uc_L2uc_L3uc>}>
 502:         : (!llvm.ptr<1>, i32, i32, i32, i32, i32)
 503:     ```
 504:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L493:** This line contributes implementation detail or declarative structure to the file.
  **CN L493:** 这一行为文件补充了实现细节或声明式结构。
- **EN L494:** This comment states: “`v_blocks` - number of tiles in innermost dimension direction to prefetch”, documenting the intent of the surrounding code.
  **CN L494:** 该注释写道：“`v_blocks` - number of tiles in innermost dimension direction to prefetch”，用于说明周围代码的意图。
- **EN L495:** This comment states: “`cache_control` - an enumerator that sets the cache behaviour”, documenting the intent of the surrounding code.
  **CN L495:** 该注释写道：“`cache_control` - an enumerator that sets the cache behaviour”，用于说明周围代码的意图。
- **EN L496:** Blank line used to separate nearby declarations and improve readability.
  **CN L496:** 该空行用于分隔相邻声明并提升可读性。
- **EN L497:** This line contributes implementation detail or declarative structure to the file.
  **CN L497:** 这一行为文件补充了实现细节或声明式结构。
- **EN L498:** This line contributes implementation detail or declarative structure to the file.
  **CN L498:** 这一行为文件补充了实现细节或声明式结构。
- **EN L499:** This line contributes implementation detail or declarative structure to the file.
  **CN L499:** 这一行为文件补充了实现细节或声明式结构。
- **EN L500:** This line contributes implementation detail or declarative structure to the file.
  **CN L500:** 这一行为文件补充了实现细节或声明式结构。
- **EN L501:** This line contributes implementation detail or declarative structure to the file.
  **CN L501:** 这一行为文件补充了实现细节或声明式结构。
- **EN L502:** This line contributes implementation detail or declarative structure to the file.
  **CN L502:** 这一行为文件补充了实现细节或声明式结构。
- **EN L503:** This line contributes implementation detail or declarative structure to the file.
  **CN L503:** 这一行为文件补充了实现细节或声明式结构。
- **EN L504:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L504:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 505-516 / 第 505-516 行

```tablegen
 505: 
 506:   let assemblyFormat = [{
 507:     operands prop-dict attr-dict `:` `(` type(operands) `)`
 508:   }];
 509: 
 510:   let extraClassDeclaration = extraBaseClassDeclaration#[{
 511:   }];
 512: 
 513:   let hasVerifier = 1;
 514: }
 515: 
 516: def XeVM_MatrixElemType
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L505:** Blank line used to separate nearby declarations and improve readability.
  **CN L505:** 该空行用于分隔相邻声明并提升可读性。
- **EN L506:** This line contributes implementation detail or declarative structure to the file.
  **CN L506:** 这一行为文件补充了实现细节或声明式结构。
- **EN L507:** This line contributes to the declaration or call of `type`.
  **CN L507:** 这一行为 `type` 的声明或调用提供内容。
- **EN L508:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L508:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L509:** Blank line used to separate nearby declarations and improve readability.
  **CN L509:** 该空行用于分隔相邻声明并提升可读性。
- **EN L510:** This line contributes implementation detail or declarative structure to the file.
  **CN L510:** 这一行为文件补充了实现细节或声明式结构。
- **EN L511:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L511:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L512:** Blank line used to separate nearby declarations and improve readability.
  **CN L512:** 该空行用于分隔相邻声明并提升可读性。
- **EN L513:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L513:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L514:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L514:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L515:** Blank line used to separate nearby declarations and improve readability.
  **CN L515:** 该空行用于分隔相邻声明并提升可读性。
- **EN L516:** This TableGen `def` record introduces `XeVM_MatrixElemType`, which later participates in generated MLIR code.
  **CN L516:** 该 TableGen `def` 记录引入了 `XeVM_MatrixElemType`，后续会参与生成的 MLIR 代码。

### Lines 517-528 / 第 517-528 行

```tablegen
 517:     : AnyTypeOf<[AnyI8, AnyI16, AnyI32, F32, TF32, F16, BF16]>;
 518: 
 519: /// Enum attribute of the different element types.
 520: def XeVM_ET_BF16 : I32EnumAttrCase<"BF16", 8, "bf16">;
 521: def XeVM_ET_F16 : I32EnumAttrCase<"F16", 9, "f16">;
 522: def XeVM_ET_S8 : I32EnumAttrCase<"S8", 10, "s8">;
 523: def XeVM_ET_U8 : I32EnumAttrCase<"U8", 11, "u8">;
 524: def XeVM_ET_S4 : I32EnumAttrCase<"S4", 12, "s4">;
 525: def XeVM_ET_U4 : I32EnumAttrCase<"U4", 13, "u4">;
 526: def XeVM_ET_TF32 : I32EnumAttrCase<"TF32", 14, "tf32">;
 527: def XeVM_ET_F32 : I32EnumAttrCase<"F32", 15, "f32">;
 528: def XeVM_ET_S32 : I32EnumAttrCase<"S32", 16, "s32">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L517:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L517:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L518:** Blank line used to separate nearby declarations and improve readability.
  **CN L518:** 该空行用于分隔相邻声明并提升可读性。
- **EN L519:** This comment states: “Enum attribute of the different element types.”, documenting the intent of the surrounding code.
  **CN L519:** 该注释写道：“Enum attribute of the different element types.”，用于说明周围代码的意图。
- **EN L520:** This TableGen `def` record introduces `XeVM_ET_BF16`, which later participates in generated MLIR code.
  **CN L520:** 该 TableGen `def` 记录引入了 `XeVM_ET_BF16`，后续会参与生成的 MLIR 代码。
- **EN L521:** This TableGen `def` record introduces `XeVM_ET_F16`, which later participates in generated MLIR code.
  **CN L521:** 该 TableGen `def` 记录引入了 `XeVM_ET_F16`，后续会参与生成的 MLIR 代码。
- **EN L522:** This TableGen `def` record introduces `XeVM_ET_S8`, which later participates in generated MLIR code.
  **CN L522:** 该 TableGen `def` 记录引入了 `XeVM_ET_S8`，后续会参与生成的 MLIR 代码。
- **EN L523:** This TableGen `def` record introduces `XeVM_ET_U8`, which later participates in generated MLIR code.
  **CN L523:** 该 TableGen `def` 记录引入了 `XeVM_ET_U8`，后续会参与生成的 MLIR 代码。
- **EN L524:** This TableGen `def` record introduces `XeVM_ET_S4`, which later participates in generated MLIR code.
  **CN L524:** 该 TableGen `def` 记录引入了 `XeVM_ET_S4`，后续会参与生成的 MLIR 代码。
- **EN L525:** This TableGen `def` record introduces `XeVM_ET_U4`, which later participates in generated MLIR code.
  **CN L525:** 该 TableGen `def` 记录引入了 `XeVM_ET_U4`，后续会参与生成的 MLIR 代码。
- **EN L526:** This TableGen `def` record introduces `XeVM_ET_TF32`, which later participates in generated MLIR code.
  **CN L526:** 该 TableGen `def` 记录引入了 `XeVM_ET_TF32`，后续会参与生成的 MLIR 代码。
- **EN L527:** This TableGen `def` record introduces `XeVM_ET_F32`, which later participates in generated MLIR code.
  **CN L527:** 该 TableGen `def` 记录引入了 `XeVM_ET_F32`，后续会参与生成的 MLIR 代码。
- **EN L528:** This TableGen `def` record introduces `XeVM_ET_S32`, which later participates in generated MLIR code.
  **CN L528:** 该 TableGen `def` 记录引入了 `XeVM_ET_S32`，后续会参与生成的 MLIR 代码。

### Lines 529-540 / 第 529-540 行

```tablegen
 529: def XeVM_ET_E2M1 : I32EnumAttrCase<"E2M1", 17, "e2m1">;
 530: def XeVM_ET_BF8 : I32EnumAttrCase<"BF8", 18, "bf8">;
 531: def XeVM_ET_F8 : I32EnumAttrCase<"F8", 19, "f8">;
 532: 
 533: def XeVM_ElemTypeAttr
 534:     : I32EnumAttr<"ElemType", "XeVM element type",
 535:                   [XeVM_ET_BF16, XeVM_ET_F16, XeVM_ET_S8, XeVM_ET_U8,
 536:                    XeVM_ET_S4, XeVM_ET_U4, XeVM_ET_TF32, XeVM_ET_F32,
 537:                    XeVM_ET_S32, XeVM_ET_E2M1, XeVM_ET_BF8, XeVM_ET_F8]> {
 538:   let cppNamespace = "::mlir::xevm";
 539: }
 540: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L529:** This TableGen `def` record introduces `XeVM_ET_E2M1`, which later participates in generated MLIR code.
  **CN L529:** 该 TableGen `def` 记录引入了 `XeVM_ET_E2M1`，后续会参与生成的 MLIR 代码。
- **EN L530:** This TableGen `def` record introduces `XeVM_ET_BF8`, which later participates in generated MLIR code.
  **CN L530:** 该 TableGen `def` 记录引入了 `XeVM_ET_BF8`，后续会参与生成的 MLIR 代码。
- **EN L531:** This TableGen `def` record introduces `XeVM_ET_F8`, which later participates in generated MLIR code.
  **CN L531:** 该 TableGen `def` 记录引入了 `XeVM_ET_F8`，后续会参与生成的 MLIR 代码。
- **EN L532:** Blank line used to separate nearby declarations and improve readability.
  **CN L532:** 该空行用于分隔相邻声明并提升可读性。
- **EN L533:** This TableGen `def` record introduces `XeVM_ElemTypeAttr`, which later participates in generated MLIR code.
  **CN L533:** 该 TableGen `def` 记录引入了 `XeVM_ElemTypeAttr`，后续会参与生成的 MLIR 代码。
- **EN L534:** This line contributes implementation detail or declarative structure to the file.
  **CN L534:** 这一行为文件补充了实现细节或声明式结构。
- **EN L535:** This line contributes implementation detail or declarative structure to the file.
  **CN L535:** 这一行为文件补充了实现细节或声明式结构。
- **EN L536:** This line contributes implementation detail or declarative structure to the file.
  **CN L536:** 这一行为文件补充了实现细节或声明式结构。
- **EN L537:** This line contributes implementation detail or declarative structure to the file.
  **CN L537:** 这一行为文件补充了实现细节或声明式结构。
- **EN L538:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L538:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L539:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L539:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L540:** Blank line used to separate nearby declarations and improve readability.
  **CN L540:** 该空行用于分隔相邻声明并提升可读性。

### Lines 541-552 / 第 541-552 行

```tablegen
 541: def XeVM_MMAShapeAttr : XeVM_Attr<"MMAShape", "mma_shape"> {
 542:   let description = [{
 543:     MMA operation is represented as D=AxB+C, where
 544:       - A has the shape MxK.
 545:       - B has the shape KxN.
 546:       - D and C have the shape MxN.
 547:     This attribute encodes the shape of all matrices that participate in MMA.
 548:   }];
 549:   let parameters = (ins "int":$m, "int":$n, "int":$k);
 550:   let assemblyFormat = "`<` struct(params) `>`";
 551: }
 552: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L541:** This TableGen `def` record introduces `XeVM_MMAShapeAttr`, which later participates in generated MLIR code.
  **CN L541:** 该 TableGen `def` 记录引入了 `XeVM_MMAShapeAttr`，后续会参与生成的 MLIR 代码。
- **EN L542:** This line contributes implementation detail or declarative structure to the file.
  **CN L542:** 这一行为文件补充了实现细节或声明式结构。
- **EN L543:** This line contributes implementation detail or declarative structure to the file.
  **CN L543:** 这一行为文件补充了实现细节或声明式结构。
- **EN L544:** This line contributes implementation detail or declarative structure to the file.
  **CN L544:** 这一行为文件补充了实现细节或声明式结构。
- **EN L545:** This line contributes implementation detail or declarative structure to the file.
  **CN L545:** 这一行为文件补充了实现细节或声明式结构。
- **EN L546:** This line contributes implementation detail or declarative structure to the file.
  **CN L546:** 这一行为文件补充了实现细节或声明式结构。
- **EN L547:** This line contributes implementation detail or declarative structure to the file.
  **CN L547:** 这一行为文件补充了实现细节或声明式结构。
- **EN L548:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L548:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L549:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L549:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L550:** This line contributes to the declaration or call of `struct`.
  **CN L550:** 这一行为 `struct` 的声明或调用提供内容。
- **EN L551:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L551:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L552:** Blank line used to separate nearby declarations and improve readability.
  **CN L552:** 该空行用于分隔相邻声明并提升可读性。

### Lines 553-564 / 第 553-564 行

```tablegen
 553: def XeVM_MMATypesAttr : XeVM_Attr<"MMATypes", "mma_types"> {
 554:   let parameters = (ins "xevm::ElemType":$d, "xevm::ElemType":$a,
 555:       "xevm::ElemType":$b, OptionalParameter<"xevm::ElemType">:$c);
 556:   let assemblyFormat = "`<` struct(params) `>`";
 557: }
 558: 
 559: def XeVM_MMAOp
 560:     : XeVM_Op<"mma">,
 561:       Results<(outs FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$d)>,
 562:       Arguments<(ins FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$a,
 563:           FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$b,
 564:           Optional<FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>>:$c,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L553:** This TableGen `def` record introduces `XeVM_MMATypesAttr`, which later participates in generated MLIR code.
  **CN L553:** 该 TableGen `def` 记录引入了 `XeVM_MMATypesAttr`，后续会参与生成的 MLIR 代码。
- **EN L554:** This line contributes implementation detail or declarative structure to the file.
  **CN L554:** 这一行为文件补充了实现细节或声明式结构。
- **EN L555:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L555:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L556:** This line contributes to the declaration or call of `struct`.
  **CN L556:** 这一行为 `struct` 的声明或调用提供内容。
- **EN L557:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L557:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L558:** Blank line used to separate nearby declarations and improve readability.
  **CN L558:** 该空行用于分隔相邻声明并提升可读性。
- **EN L559:** This TableGen `def` record introduces `XeVM_MMAOp`, which later participates in generated MLIR code.
  **CN L559:** 该 TableGen `def` 记录引入了 `XeVM_MMAOp`，后续会参与生成的 MLIR 代码。
- **EN L560:** This line contributes implementation detail or declarative structure to the file.
  **CN L560:** 这一行为文件补充了实现细节或声明式结构。
- **EN L561:** This line contributes implementation detail or declarative structure to the file.
  **CN L561:** 这一行为文件补充了实现细节或声明式结构。
- **EN L562:** This line contributes implementation detail or declarative structure to the file.
  **CN L562:** 这一行为文件补充了实现细节或声明式结构。
- **EN L563:** This line contributes implementation detail or declarative structure to the file.
  **CN L563:** 这一行为文件补充了实现细节或声明式结构。
- **EN L564:** This line contributes implementation detail or declarative structure to the file.
  **CN L564:** 这一行为文件补充了实现细节或声明式结构。

### Lines 565-576 / 第 565-576 行

```tablegen
 565:           XeVM_MMAShapeAttr:$shape, XeVM_MMATypesAttr:$types)> {
 566: 
 567:   let summary = "Subgroup matrix multiply-add";
 568: 
 569:   let description = [{
 570:     The `xevm.mma` is a cooperative operation where all threads/lanes in
 571:     a subgroup participates and carries out matrix multiplication plus accumulation:
 572: 
 573:       D = C + A x B
 574: 
 575:       where the A, B, C input matrices and the result D have shapes:
 576:         - D : MxN
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L565:** This line contributes implementation detail or declarative structure to the file.
  **CN L565:** 这一行为文件补充了实现细节或声明式结构。
- **EN L566:** Blank line used to separate nearby declarations and improve readability.
  **CN L566:** 该空行用于分隔相邻声明并提升可读性。
- **EN L567:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L567:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L568:** Blank line used to separate nearby declarations and improve readability.
  **CN L568:** 该空行用于分隔相邻声明并提升可读性。
- **EN L569:** This line contributes implementation detail or declarative structure to the file.
  **CN L569:** 这一行为文件补充了实现细节或声明式结构。
- **EN L570:** This line contributes implementation detail or declarative structure to the file.
  **CN L570:** 这一行为文件补充了实现细节或声明式结构。
- **EN L571:** This line contributes implementation detail or declarative structure to the file.
  **CN L571:** 这一行为文件补充了实现细节或声明式结构。
- **EN L572:** Blank line used to separate nearby declarations and improve readability.
  **CN L572:** 该空行用于分隔相邻声明并提升可读性。
- **EN L573:** This line contributes implementation detail or declarative structure to the file.
  **CN L573:** 这一行为文件补充了实现细节或声明式结构。
- **EN L574:** Blank line used to separate nearby declarations and improve readability.
  **CN L574:** 该空行用于分隔相邻声明并提升可读性。
- **EN L575:** This line contributes implementation detail or declarative structure to the file.
  **CN L575:** 这一行为文件补充了实现细节或声明式结构。
- **EN L576:** This line contributes implementation detail or declarative structure to the file.
  **CN L576:** 这一行为文件补充了实现细节或声明式结构。

### Lines 577-588 / 第 577-588 行

```tablegen
 577:         - C : MxN
 578:         - A : MxK
 579:         - B : KxN
 580: 
 581:     Parameters:
 582:       * `a` - vector of matrix A elements.
 583:       * `b` - vector of matrix B elements.
 584:       * `c` - (optional) vector of matrix C elements.
 585:       * `shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.
 586:       * `types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.
 587: 
 588:     Example:
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L577:** This line contributes implementation detail or declarative structure to the file.
  **CN L577:** 这一行为文件补充了实现细节或声明式结构。
- **EN L578:** This line contributes implementation detail or declarative structure to the file.
  **CN L578:** 这一行为文件补充了实现细节或声明式结构。
- **EN L579:** This line contributes implementation detail or declarative structure to the file.
  **CN L579:** 这一行为文件补充了实现细节或声明式结构。
- **EN L580:** Blank line used to separate nearby declarations and improve readability.
  **CN L580:** 该空行用于分隔相邻声明并提升可读性。
- **EN L581:** This line contributes implementation detail or declarative structure to the file.
  **CN L581:** 这一行为文件补充了实现细节或声明式结构。
- **EN L582:** This comment states: “`a` - vector of matrix A elements.”, documenting the intent of the surrounding code.
  **CN L582:** 该注释写道：“`a` - vector of matrix A elements.”，用于说明周围代码的意图。
- **EN L583:** This comment states: “`b` - vector of matrix B elements.”, documenting the intent of the surrounding code.
  **CN L583:** 该注释写道：“`b` - vector of matrix B elements.”，用于说明周围代码的意图。
- **EN L584:** This comment states: “`c` - (optional) vector of matrix C elements.”, documenting the intent of the surrounding code.
  **CN L584:** 该注释写道：“`c` - (optional) vector of matrix C elements.”，用于说明周围代码的意图。
- **EN L585:** This comment states: “`shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.”, documenting the intent of the surrounding code.
  **CN L585:** 该注释写道：“`shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.”，用于说明周围代码的意图。
- **EN L586:** This comment states: “`types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.”, documenting the intent of the surrounding code.
  **CN L586:** 该注释写道：“`types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.”，用于说明周围代码的意图。
- **EN L587:** Blank line used to separate nearby declarations and improve readability.
  **CN L587:** 该空行用于分隔相邻声明并提升可读性。
- **EN L588:** This line contributes implementation detail or declarative structure to the file.
  **CN L588:** 这一行为文件补充了实现细节或声明式结构。

### Lines 589-600 / 第 589-600 行

```tablegen
 589:     ```mlir
 590:       %d = xevm.mma %a, %b, %c { shape=<m=8, n=16, k=16>, types=<d=f32, a=f16, b=f16, c=f32> }
 591:              : (vector<8xi16>, vector<8xi32>, vector<8xf32>) -> vector<8xf32>
 592:     ```
 593:   }];
 594: 
 595:   let assemblyFormat = [{
 596:     $a `,` $b (`,` $c^)? ` `
 597:     `{`
 598:       `shape` `=` $shape `,`
 599:       `types` `=` $types
 600:     `}` attr-dict `:` functional-type(operands, results)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L589:** This line contributes implementation detail or declarative structure to the file.
  **CN L589:** 这一行为文件补充了实现细节或声明式结构。
- **EN L590:** This line contributes implementation detail or declarative structure to the file.
  **CN L590:** 这一行为文件补充了实现细节或声明式结构。
- **EN L591:** This line contributes implementation detail or declarative structure to the file.
  **CN L591:** 这一行为文件补充了实现细节或声明式结构。
- **EN L592:** This line contributes implementation detail or declarative structure to the file.
  **CN L592:** 这一行为文件补充了实现细节或声明式结构。
- **EN L593:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L593:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L594:** Blank line used to separate nearby declarations and improve readability.
  **CN L594:** 该空行用于分隔相邻声明并提升可读性。
- **EN L595:** This line contributes implementation detail or declarative structure to the file.
  **CN L595:** 这一行为文件补充了实现细节或声明式结构。
- **EN L596:** This line contributes to the declaration or call of `b`.
  **CN L596:** 这一行为 `b` 的声明或调用提供内容。
- **EN L597:** This line contributes implementation detail or declarative structure to the file.
  **CN L597:** 这一行为文件补充了实现细节或声明式结构。
- **EN L598:** This line contributes implementation detail or declarative structure to the file.
  **CN L598:** 这一行为文件补充了实现细节或声明式结构。
- **EN L599:** This line contributes implementation detail or declarative structure to the file.
  **CN L599:** 这一行为文件补充了实现细节或声明式结构。
- **EN L600:** This line contributes to the declaration or call of `type`.
  **CN L600:** 这一行为 `type` 的声明或调用提供内容。

### Lines 601-612 / 第 601-612 行

```tablegen
 601:   }];
 602: 
 603:   let hasVerifier = 1;
 604: }
 605: 
 606: def XeVM_TruncfSrcElemTypes
 607:     : I32EnumAttr<
 608:           "TruncfSrcElemTypes",
 609:           "Source element type for xevm.truncf", [XeVM_ET_F16, XeVM_ET_BF16]> {
 610:   let cppNamespace = "::mlir::xevm";
 611: }
 612: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L601:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L601:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L602:** Blank line used to separate nearby declarations and improve readability.
  **CN L602:** 该空行用于分隔相邻声明并提升可读性。
- **EN L603:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L603:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L604:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L604:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L605:** Blank line used to separate nearby declarations and improve readability.
  **CN L605:** 该空行用于分隔相邻声明并提升可读性。
- **EN L606:** This TableGen `def` record introduces `XeVM_TruncfSrcElemTypes`, which later participates in generated MLIR code.
  **CN L606:** 该 TableGen `def` 记录引入了 `XeVM_TruncfSrcElemTypes`，后续会参与生成的 MLIR 代码。
- **EN L607:** This line contributes implementation detail or declarative structure to the file.
  **CN L607:** 这一行为文件补充了实现细节或声明式结构。
- **EN L608:** This line contributes implementation detail or declarative structure to the file.
  **CN L608:** 这一行为文件补充了实现细节或声明式结构。
- **EN L609:** This line contributes implementation detail or declarative structure to the file.
  **CN L609:** 这一行为文件补充了实现细节或声明式结构。
- **EN L610:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L610:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L611:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L611:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L612:** Blank line used to separate nearby declarations and improve readability.
  **CN L612:** 该空行用于分隔相邻声明并提升可读性。

### Lines 613-624 / 第 613-624 行

```tablegen
 613: def XeVM_TruncfSrcElemTypeAttr : XeVM_Attr<"TruncfSrcElemType", "src_etype"> {
 614:   let parameters = (ins "xevm::TruncfSrcElemTypes":$etype);
 615:   let assemblyFormat = "`src_etype` `=` $etype";
 616: }
 617: 
 618: def XeVM_TruncfDstElemTypes
 619:     : I32EnumAttr<"TruncfDstElemTypes",
 620:                   "Destination element type for xevm.truncf",
 621:                   [XeVM_ET_BF8, XeVM_ET_F8, XeVM_ET_E2M1]> {
 622:   let cppNamespace = "::mlir::xevm";
 623: }
 624: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L613:** This TableGen `def` record introduces `XeVM_TruncfSrcElemTypeAttr`, which later participates in generated MLIR code.
  **CN L613:** 该 TableGen `def` 记录引入了 `XeVM_TruncfSrcElemTypeAttr`，后续会参与生成的 MLIR 代码。
- **EN L614:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L614:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L615:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L615:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L616:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L616:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L617:** Blank line used to separate nearby declarations and improve readability.
  **CN L617:** 该空行用于分隔相邻声明并提升可读性。
- **EN L618:** This TableGen `def` record introduces `XeVM_TruncfDstElemTypes`, which later participates in generated MLIR code.
  **CN L618:** 该 TableGen `def` 记录引入了 `XeVM_TruncfDstElemTypes`，后续会参与生成的 MLIR 代码。
- **EN L619:** This line contributes implementation detail or declarative structure to the file.
  **CN L619:** 这一行为文件补充了实现细节或声明式结构。
- **EN L620:** This line contributes implementation detail or declarative structure to the file.
  **CN L620:** 这一行为文件补充了实现细节或声明式结构。
- **EN L621:** This line contributes implementation detail or declarative structure to the file.
  **CN L621:** 这一行为文件补充了实现细节或声明式结构。
- **EN L622:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L622:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L623:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L623:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L624:** Blank line used to separate nearby declarations and improve readability.
  **CN L624:** 该空行用于分隔相邻声明并提升可读性。

### Lines 625-636 / 第 625-636 行

```tablegen
 625: def XeVM_TruncfDstElemTypeAttr : XeVM_Attr<"TruncfDstElemType", "dst_etype"> {
 626:   let parameters = (ins "xevm::TruncfDstElemTypes":$etype);
 627:   let assemblyFormat = "`dst_etype` `=` $etype";
 628: }
 629: 
 630: def XeVM_TruncfOp
 631:     : XeVM_Op<"truncf">,
 632:       Results<(outs AnyTypeOf<[FixedVectorOfRankAndType<[1], [I8, I<4>]>, I8,
 633:                                I<4>]>:$dst)>,
 634:       Arguments<(ins AnyTypeOf<[FixedVectorOfRankAndType<[1], [F16, BF16]>, F16,
 635:                                 BF16]>:$src,
 636:           XeVM_TruncfSrcElemTypeAttr:$src_etype,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L625:** This TableGen `def` record introduces `XeVM_TruncfDstElemTypeAttr`, which later participates in generated MLIR code.
  **CN L625:** 该 TableGen `def` 记录引入了 `XeVM_TruncfDstElemTypeAttr`，后续会参与生成的 MLIR 代码。
- **EN L626:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L626:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L627:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L627:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L628:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L628:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L629:** Blank line used to separate nearby declarations and improve readability.
  **CN L629:** 该空行用于分隔相邻声明并提升可读性。
- **EN L630:** This TableGen `def` record introduces `XeVM_TruncfOp`, which later participates in generated MLIR code.
  **CN L630:** 该 TableGen `def` 记录引入了 `XeVM_TruncfOp`，后续会参与生成的 MLIR 代码。
- **EN L631:** This line contributes implementation detail or declarative structure to the file.
  **CN L631:** 这一行为文件补充了实现细节或声明式结构。
- **EN L632:** This line contributes implementation detail or declarative structure to the file.
  **CN L632:** 这一行为文件补充了实现细节或声明式结构。
- **EN L633:** This line contributes implementation detail or declarative structure to the file.
  **CN L633:** 这一行为文件补充了实现细节或声明式结构。
- **EN L634:** This line contributes implementation detail or declarative structure to the file.
  **CN L634:** 这一行为文件补充了实现细节或声明式结构。
- **EN L635:** This line contributes implementation detail or declarative structure to the file.
  **CN L635:** 这一行为文件补充了实现细节或声明式结构。
- **EN L636:** This line contributes implementation detail or declarative structure to the file.
  **CN L636:** 这一行为文件补充了实现细节或声明式结构。

### Lines 637-648 / 第 637-648 行

```tablegen
 637:           XeVM_TruncfDstElemTypeAttr:$dst_etype)> {
 638:   let summary = "Floating point truncation from f16/bf16 to f8/bf8/f4";
 639:   let description = [{
 640:     The `xevm.truncf` operation truncates a floating point value from
 641:     f16/bf16 to f8/bf8/f4 format.
 642:   }];
 643: 
 644:   let assemblyFormat = [{
 645:     $src ` ` `{` $src_etype `,` $dst_etype `}` attr-dict `:` functional-type(operands, results)
 646:   }];
 647: 
 648:   let hasVerifier = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L637:** This line contributes implementation detail or declarative structure to the file.
  **CN L637:** 这一行为文件补充了实现细节或声明式结构。
- **EN L638:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L638:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L639:** This line contributes implementation detail or declarative structure to the file.
  **CN L639:** 这一行为文件补充了实现细节或声明式结构。
- **EN L640:** This line contributes implementation detail or declarative structure to the file.
  **CN L640:** 这一行为文件补充了实现细节或声明式结构。
- **EN L641:** This line contributes implementation detail or declarative structure to the file.
  **CN L641:** 这一行为文件补充了实现细节或声明式结构。
- **EN L642:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L642:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L643:** Blank line used to separate nearby declarations and improve readability.
  **CN L643:** 该空行用于分隔相邻声明并提升可读性。
- **EN L644:** This line contributes implementation detail or declarative structure to the file.
  **CN L644:** 这一行为文件补充了实现细节或声明式结构。
- **EN L645:** This line contributes to the declaration or call of `type`.
  **CN L645:** 这一行为 `type` 的声明或调用提供内容。
- **EN L646:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L646:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L647:** Blank line used to separate nearby declarations and improve readability.
  **CN L647:** 该空行用于分隔相邻声明并提升可读性。
- **EN L648:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L648:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 649-660 / 第 649-660 行

```tablegen
 649: }
 650: 
 651: def XeVM_MMAMxOp
 652:     : XeVM_Op<"mma_mx">,
 653:       Results<(outs FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$d)>,
 654:       Arguments<(ins FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$a,
 655:           FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>:$b,
 656:           AnyTypeOf<[FixedVectorOfRankAndType<[1], [I8]>, I8]>:$scale_a,
 657:           AnyTypeOf<[FixedVectorOfRankAndType<[1], [I8]>, I8]>:$scale_b,
 658:           Optional<FixedVectorOfRankAndType<[1], [XeVM_MatrixElemType]>>:$c,
 659:           XeVM_MMAShapeAttr:$shape, XeVM_MMATypesAttr:$types)> {
 660: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L649:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L649:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L650:** Blank line used to separate nearby declarations and improve readability.
  **CN L650:** 该空行用于分隔相邻声明并提升可读性。
- **EN L651:** This TableGen `def` record introduces `XeVM_MMAMxOp`, which later participates in generated MLIR code.
  **CN L651:** 该 TableGen `def` 记录引入了 `XeVM_MMAMxOp`，后续会参与生成的 MLIR 代码。
- **EN L652:** This line contributes implementation detail or declarative structure to the file.
  **CN L652:** 这一行为文件补充了实现细节或声明式结构。
- **EN L653:** This line contributes implementation detail or declarative structure to the file.
  **CN L653:** 这一行为文件补充了实现细节或声明式结构。
- **EN L654:** This line contributes implementation detail or declarative structure to the file.
  **CN L654:** 这一行为文件补充了实现细节或声明式结构。
- **EN L655:** This line contributes implementation detail or declarative structure to the file.
  **CN L655:** 这一行为文件补充了实现细节或声明式结构。
- **EN L656:** This line contributes implementation detail or declarative structure to the file.
  **CN L656:** 这一行为文件补充了实现细节或声明式结构。
- **EN L657:** This line contributes implementation detail or declarative structure to the file.
  **CN L657:** 这一行为文件补充了实现细节或声明式结构。
- **EN L658:** This line contributes implementation detail or declarative structure to the file.
  **CN L658:** 这一行为文件补充了实现细节或声明式结构。
- **EN L659:** This line contributes implementation detail or declarative structure to the file.
  **CN L659:** 这一行为文件补充了实现细节或声明式结构。
- **EN L660:** Blank line used to separate nearby declarations and improve readability.
  **CN L660:** 该空行用于分隔相邻声明并提升可读性。

### Lines 661-672 / 第 661-672 行

```tablegen
 661:   let summary = "Subgroup matrix multiply-add with MxN shape and MX scaling";
 662: 
 663:   let description = [{
 664:     The `xevm.mma_mx` is similar to `xevm.mma` has scale operands for A and B matrices.
 665:     It is a cooperative operation where all threads/lanes in a subgroup participates
 666:     and carries out matrix multiplication plus accumulation:
 667: 
 668:       D = C + A x B
 669: 
 670:       where the A, B, C input matrices and the result D have shapes:
 671:         - D : MxN
 672:         - C : MxN
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L661:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L661:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L662:** Blank line used to separate nearby declarations and improve readability.
  **CN L662:** 该空行用于分隔相邻声明并提升可读性。
- **EN L663:** This line contributes implementation detail or declarative structure to the file.
  **CN L663:** 这一行为文件补充了实现细节或声明式结构。
- **EN L664:** This line contributes implementation detail or declarative structure to the file.
  **CN L664:** 这一行为文件补充了实现细节或声明式结构。
- **EN L665:** This line contributes implementation detail or declarative structure to the file.
  **CN L665:** 这一行为文件补充了实现细节或声明式结构。
- **EN L666:** This line contributes implementation detail or declarative structure to the file.
  **CN L666:** 这一行为文件补充了实现细节或声明式结构。
- **EN L667:** Blank line used to separate nearby declarations and improve readability.
  **CN L667:** 该空行用于分隔相邻声明并提升可读性。
- **EN L668:** This line contributes implementation detail or declarative structure to the file.
  **CN L668:** 这一行为文件补充了实现细节或声明式结构。
- **EN L669:** Blank line used to separate nearby declarations and improve readability.
  **CN L669:** 该空行用于分隔相邻声明并提升可读性。
- **EN L670:** This line contributes implementation detail or declarative structure to the file.
  **CN L670:** 这一行为文件补充了实现细节或声明式结构。
- **EN L671:** This line contributes implementation detail or declarative structure to the file.
  **CN L671:** 这一行为文件补充了实现细节或声明式结构。
- **EN L672:** This line contributes implementation detail or declarative structure to the file.
  **CN L672:** 这一行为文件补充了实现细节或声明式结构。

### Lines 673-684 / 第 673-684 行

```tablegen
 673:         - A : MxK
 674:         - B : KxN
 675: 
 676:     Parameters:
 677:       * `a` - vector of matrix A elements.
 678:       * `b` - vector of matrix B elements.
 679:       * `scale_a` - vector or scalar of scaling factors for matrix A.
 680:       * `scale_b` - vector or scalar of scaling factors for matrix B.
 681:       * `c` - (optional) vector of matrix C elements.
 682:       * `shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.
 683:       * `types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.
 684: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L673:** This line contributes implementation detail or declarative structure to the file.
  **CN L673:** 这一行为文件补充了实现细节或声明式结构。
- **EN L674:** This line contributes implementation detail or declarative structure to the file.
  **CN L674:** 这一行为文件补充了实现细节或声明式结构。
- **EN L675:** Blank line used to separate nearby declarations and improve readability.
  **CN L675:** 该空行用于分隔相邻声明并提升可读性。
- **EN L676:** This line contributes implementation detail or declarative structure to the file.
  **CN L676:** 这一行为文件补充了实现细节或声明式结构。
- **EN L677:** This comment states: “`a` - vector of matrix A elements.”, documenting the intent of the surrounding code.
  **CN L677:** 该注释写道：“`a` - vector of matrix A elements.”，用于说明周围代码的意图。
- **EN L678:** This comment states: “`b` - vector of matrix B elements.”, documenting the intent of the surrounding code.
  **CN L678:** 该注释写道：“`b` - vector of matrix B elements.”，用于说明周围代码的意图。
- **EN L679:** This comment states: “`scale_a` - vector or scalar of scaling factors for matrix A.”, documenting the intent of the surrounding code.
  **CN L679:** 该注释写道：“`scale_a` - vector or scalar of scaling factors for matrix A.”，用于说明周围代码的意图。
- **EN L680:** This comment states: “`scale_b` - vector or scalar of scaling factors for matrix B.”, documenting the intent of the surrounding code.
  **CN L680:** 该注释写道：“`scale_b` - vector or scalar of scaling factors for matrix B.”，用于说明周围代码的意图。
- **EN L681:** This comment states: “`c` - (optional) vector of matrix C elements.”, documenting the intent of the surrounding code.
  **CN L681:** 该注释写道：“`c` - (optional) vector of matrix C elements.”，用于说明周围代码的意图。
- **EN L682:** This comment states: “`shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.”, documenting the intent of the surrounding code.
  **CN L682:** 该注释写道：“`shape` - the shape of the matrices, specified as `M`, `N`, and `K` values.”，用于说明周围代码的意图。
- **EN L683:** This comment states: “`types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.”, documenting the intent of the surrounding code.
  **CN L683:** 该注释写道：“`types` - the data types of the matrices, specified as `D`, `A`, `B`, and optionally `C`.”，用于说明周围代码的意图。
- **EN L684:** Blank line used to separate nearby declarations and improve readability.
  **CN L684:** 该空行用于分隔相邻声明并提升可读性。

### Lines 685-696 / 第 685-696 行

```tablegen
 685:   }];
 686: 
 687:   let assemblyFormat = [{
 688:     $a `,` $b `,` $scale_a `,` $scale_b (`,` $c^)? ` `
 689:     `{`
 690:       `shape` `=` $shape `,`
 691:       `types` `=` $types
 692:     `}` attr-dict `:` functional-type(operands, results)
 693:   }];
 694: 
 695:   let hasVerifier = 1;
 696: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L685:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L685:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L686:** Blank line used to separate nearby declarations and improve readability.
  **CN L686:** 该空行用于分隔相邻声明并提升可读性。
- **EN L687:** This line contributes implementation detail or declarative structure to the file.
  **CN L687:** 这一行为文件补充了实现细节或声明式结构。
- **EN L688:** This line contributes to the declaration or call of `scale_b`.
  **CN L688:** 这一行为 `scale_b` 的声明或调用提供内容。
- **EN L689:** This line contributes implementation detail or declarative structure to the file.
  **CN L689:** 这一行为文件补充了实现细节或声明式结构。
- **EN L690:** This line contributes implementation detail or declarative structure to the file.
  **CN L690:** 这一行为文件补充了实现细节或声明式结构。
- **EN L691:** This line contributes implementation detail or declarative structure to the file.
  **CN L691:** 这一行为文件补充了实现细节或声明式结构。
- **EN L692:** This line contributes to the declaration or call of `type`.
  **CN L692:** 这一行为 `type` 的声明或调用提供内容。
- **EN L693:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L693:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L694:** Blank line used to separate nearby declarations and improve readability.
  **CN L694:** 该空行用于分隔相邻声明并提升可读性。
- **EN L695:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L695:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L696:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L696:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 697-708 / 第 697-708 行

```tablegen
 697: 
 698: //===----------------------------------------------------------------------===//
 699: // XeVM target attribute.
 700: //===----------------------------------------------------------------------===//
 701: 
 702: def XeVM_TargetAttr : XeVM_Attr<"XeVMTarget", "target"> {
 703:   let description = [{
 704:     GPU target attribute for controlling compilation of Intel GPU targets. All
 705:     parameters decay into default values if not present.
 706: 
 707:     Examples:
 708: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L697:** Blank line used to separate nearby declarations and improve readability.
  **CN L697:** 该空行用于分隔相邻声明并提升可读性。
- **EN L698:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L698:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L699:** This comment states: “XeVM target attribute.”, documenting the intent of the surrounding code.
  **CN L699:** 该注释写道：“XeVM target attribute.”，用于说明周围代码的意图。
- **EN L700:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L700:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L701:** Blank line used to separate nearby declarations and improve readability.
  **CN L701:** 该空行用于分隔相邻声明并提升可读性。
- **EN L702:** This TableGen `def` record introduces `XeVM_TargetAttr`, which later participates in generated MLIR code.
  **CN L702:** 该 TableGen `def` 记录引入了 `XeVM_TargetAttr`，后续会参与生成的 MLIR 代码。
- **EN L703:** This line contributes implementation detail or declarative structure to the file.
  **CN L703:** 这一行为文件补充了实现细节或声明式结构。
- **EN L704:** This line contributes implementation detail or declarative structure to the file.
  **CN L704:** 这一行为文件补充了实现细节或声明式结构。
- **EN L705:** This line contributes implementation detail or declarative structure to the file.
  **CN L705:** 这一行为文件补充了实现细节或声明式结构。
- **EN L706:** Blank line used to separate nearby declarations and improve readability.
  **CN L706:** 该空行用于分隔相邻声明并提升可读性。
- **EN L707:** This line contributes implementation detail or declarative structure to the file.
  **CN L707:** 这一行为文件补充了实现细节或声明式结构。
- **EN L708:** Blank line used to separate nearby declarations and improve readability.
  **CN L708:** 该空行用于分隔相邻声明并提升可读性。

### Lines 709-720 / 第 709-720 行

```tablegen
 709:     1. Target with default values.
 710:     ```
 711:       gpu.module @mymodule [#xevm.target] attributes {...} {
 712:         ...
 713:       }
 714:     ```
 715:   }];
 716:   let parameters =
 717:       (ins DefaultValuedParameter<"int", "2",
 718:                                   "Optimization level to apply.">:$O,
 719:           StringRefParameter<"Target triple.",
 720:                              "\"spirv64-unknown-unknown\"">:$triple,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L709:** This line contributes implementation detail or declarative structure to the file.
  **CN L709:** 这一行为文件补充了实现细节或声明式结构。
- **EN L710:** This line contributes implementation detail or declarative structure to the file.
  **CN L710:** 这一行为文件补充了实现细节或声明式结构。
- **EN L711:** This line contributes implementation detail or declarative structure to the file.
  **CN L711:** 这一行为文件补充了实现细节或声明式结构。
- **EN L712:** This line contributes implementation detail or declarative structure to the file.
  **CN L712:** 这一行为文件补充了实现细节或声明式结构。
- **EN L713:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L713:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L714:** This line contributes implementation detail or declarative structure to the file.
  **CN L714:** 这一行为文件补充了实现细节或声明式结构。
- **EN L715:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L715:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L716:** This line contributes implementation detail or declarative structure to the file.
  **CN L716:** 这一行为文件补充了实现细节或声明式结构。
- **EN L717:** This line contributes implementation detail or declarative structure to the file.
  **CN L717:** 这一行为文件补充了实现细节或声明式结构。
- **EN L718:** This line contributes implementation detail or declarative structure to the file.
  **CN L718:** 这一行为文件补充了实现细节或声明式结构。
- **EN L719:** This line contributes implementation detail or declarative structure to the file.
  **CN L719:** 这一行为文件补充了实现细节或声明式结构。
- **EN L720:** This line contributes implementation detail or declarative structure to the file.
  **CN L720:** 这一行为文件补充了实现细节或声明式结构。

### Lines 721-732 / 第 721-732 行

```tablegen
 721:           StringRefParameter<"Target chip.", "\"bmg\"">:$chip,
 722:           OptionalParameter<"::mlir::DictionaryAttr",
 723:                             "Target specific flags.">:$flags,
 724:           OptionalParameter<"::mlir::ArrayAttr",
 725:                             "Files to link to the LLVM module.">:$linkFiles);
 726:   let assemblyFormat = [{
 727:     (`<` struct($O, $triple, $chip, $flags, $linkFiles)^ `>`)?
 728:   }];
 729:   let builders = [AttrBuilder<
 730:       (ins CArg<"int", "2">:$optLevel,
 731:           CArg<"::llvm::StringRef", "\"spirv64-unknown-unknown\"">:$triple,
 732:           CArg<"::llvm::StringRef", "\"bmg\"">:$chip,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L721:** This line contributes implementation detail or declarative structure to the file.
  **CN L721:** 这一行为文件补充了实现细节或声明式结构。
- **EN L722:** This line contributes implementation detail or declarative structure to the file.
  **CN L722:** 这一行为文件补充了实现细节或声明式结构。
- **EN L723:** This line contributes implementation detail or declarative structure to the file.
  **CN L723:** 这一行为文件补充了实现细节或声明式结构。
- **EN L724:** This line contributes implementation detail or declarative structure to the file.
  **CN L724:** 这一行为文件补充了实现细节或声明式结构。
- **EN L725:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L725:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L726:** This line contributes implementation detail or declarative structure to the file.
  **CN L726:** 这一行为文件补充了实现细节或声明式结构。
- **EN L727:** This line contributes to the declaration or call of `struct`.
  **CN L727:** 这一行为 `struct` 的声明或调用提供内容。
- **EN L728:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L728:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L729:** This line contributes implementation detail or declarative structure to the file.
  **CN L729:** 这一行为文件补充了实现细节或声明式结构。
- **EN L730:** This line contributes implementation detail or declarative structure to the file.
  **CN L730:** 这一行为文件补充了实现细节或声明式结构。
- **EN L731:** This line contributes implementation detail or declarative structure to the file.
  **CN L731:** 这一行为文件补充了实现细节或声明式结构。
- **EN L732:** This line contributes implementation detail or declarative structure to the file.
  **CN L732:** 这一行为文件补充了实现细节或声明式结构。

### Lines 733-744 / 第 733-744 行

```tablegen
 733:           CArg<"::mlir::DictionaryAttr", "nullptr">:$targetFlags,
 734:           CArg<"::mlir::ArrayAttr", "nullptr">:$linkFiles),
 735:       [{
 736:       return Base::get($_ctxt, optLevel, triple, chip, targetFlags, linkFiles);
 737:     }]>];
 738:   let skipDefaultBuilders = 1;
 739:   let genVerifyDecl = 1;
 740: }
 741: 
 742: //===----------------------------------------------------------------------===//
 743: // XeVM special register op definitions
 744: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L733:** This line contributes implementation detail or declarative structure to the file.
  **CN L733:** 这一行为文件补充了实现细节或声明式结构。
- **EN L734:** This line contributes implementation detail or declarative structure to the file.
  **CN L734:** 这一行为文件补充了实现细节或声明式结构。
- **EN L735:** This line contributes implementation detail or declarative structure to the file.
  **CN L735:** 这一行为文件补充了实现细节或声明式结构。
- **EN L736:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L736:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L737:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L737:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L738:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L738:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L739:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L739:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L740:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L740:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L741:** Blank line used to separate nearby declarations and improve readability.
  **CN L741:** 该空行用于分隔相邻声明并提升可读性。
- **EN L742:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L742:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L743:** This comment states: “XeVM special register op definitions”, documenting the intent of the surrounding code.
  **CN L743:** 该注释写道：“XeVM special register op definitions”，用于说明周围代码的意图。
- **EN L744:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L744:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 745-756 / 第 745-756 行

```tablegen
 745: 
 746: class XeVM_SpecialIdRegisterOp<string mnemonic, list<Trait> traits = []>
 747:     : XeVM_Op<mnemonic, traits>,
 748:       Results<(outs AnyTypeOf<[I32, I64]>:$res)>,
 749:       Arguments<(ins OptionalAttr<LLVM_ConstantRangeAttr>:$range)> {
 750:   let assemblyFormat = "(`range` $range^)? attr-dict `:` type($res)";
 751: }
 752: 
 753: multiclass XeVM_SpecialRegisterXYZ<string mnemonic, list<Trait> traits = []> {
 754:   def XOp : XeVM_SpecialIdRegisterOp<!strconcat(mnemonic, ".x"), traits>;
 755:   def YOp : XeVM_SpecialIdRegisterOp<!strconcat(mnemonic, ".y"), traits>;
 756:   def ZOp : XeVM_SpecialIdRegisterOp<!strconcat(mnemonic, ".z"), traits>;
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L745:** Blank line used to separate nearby declarations and improve readability.
  **CN L745:** 该空行用于分隔相邻声明并提升可读性。
- **EN L746:** This TableGen `class` record introduces `XeVM_SpecialIdRegisterOp`, which later participates in generated MLIR code.
  **CN L746:** 该 TableGen `class` 记录引入了 `XeVM_SpecialIdRegisterOp`，后续会参与生成的 MLIR 代码。
- **EN L747:** This line contributes implementation detail or declarative structure to the file.
  **CN L747:** 这一行为文件补充了实现细节或声明式结构。
- **EN L748:** This line contributes implementation detail or declarative structure to the file.
  **CN L748:** 这一行为文件补充了实现细节或声明式结构。
- **EN L749:** This line contributes implementation detail or declarative structure to the file.
  **CN L749:** 这一行为文件补充了实现细节或声明式结构。
- **EN L750:** This line contributes to the declaration or call of `type`.
  **CN L750:** 这一行为 `type` 的声明或调用提供内容。
- **EN L751:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L751:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L752:** Blank line used to separate nearby declarations and improve readability.
  **CN L752:** 该空行用于分隔相邻声明并提升可读性。
- **EN L753:** This TableGen `multiclass` record introduces `XeVM_SpecialRegisterXYZ`, which later participates in generated MLIR code.
  **CN L753:** 该 TableGen `multiclass` 记录引入了 `XeVM_SpecialRegisterXYZ`，后续会参与生成的 MLIR 代码。
- **EN L754:** This TableGen `def` record introduces `XOp`, which later participates in generated MLIR code.
  **CN L754:** 该 TableGen `def` 记录引入了 `XOp`，后续会参与生成的 MLIR 代码。
- **EN L755:** This TableGen `def` record introduces `YOp`, which later participates in generated MLIR code.
  **CN L755:** 该 TableGen `def` 记录引入了 `YOp`，后续会参与生成的 MLIR 代码。
- **EN L756:** This TableGen `def` record introduces `ZOp`, which later participates in generated MLIR code.
  **CN L756:** 该 TableGen `def` 记录引入了 `ZOp`，后续会参与生成的 MLIR 代码。

### Lines 757-768 / 第 757-768 行

```tablegen
 757: }
 758: 
 759: //===----------------------------------------------------------------------===//
 760: // Workitem index and range
 761: defm XeVM_WorkitemId : XeVM_SpecialRegisterXYZ<"local_id">;
 762: defm XeVM_WorkgroupDim : XeVM_SpecialRegisterXYZ<"local_size">;
 763: 
 764: //===----------------------------------------------------------------------===//
 765: // Workgroup index and range
 766: defm XeVM_WorkgroupId : XeVM_SpecialRegisterXYZ<"group_id">;
 767: defm XeVM_GridDim : XeVM_SpecialRegisterXYZ<"group_count">;
 768: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L757:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L757:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L758:** Blank line used to separate nearby declarations and improve readability.
  **CN L758:** 该空行用于分隔相邻声明并提升可读性。
- **EN L759:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L759:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L760:** This comment states: “Workitem index and range”, documenting the intent of the surrounding code.
  **CN L760:** 该注释写道：“Workitem index and range”，用于说明周围代码的意图。
- **EN L761:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L761:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L762:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L762:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L763:** Blank line used to separate nearby declarations and improve readability.
  **CN L763:** 该空行用于分隔相邻声明并提升可读性。
- **EN L764:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L764:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L765:** This comment states: “Workgroup index and range”, documenting the intent of the surrounding code.
  **CN L765:** 该注释写道：“Workgroup index and range”，用于说明周围代码的意图。
- **EN L766:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L766:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L767:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L767:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L768:** Blank line used to separate nearby declarations and improve readability.
  **CN L768:** 该空行用于分隔相邻声明并提升可读性。

### Lines 769-775 / 第 769-775 行

```tablegen
 769: //===----------------------------------------------------------------------===//
 770: // Lane, Subgroup index and range
 771: def XeVM_LaneIdOp : XeVM_SpecialIdRegisterOp<"lane_id">;
 772: def XeVM_SubgroupIdOp : XeVM_SpecialIdRegisterOp<"subgroup_id">;
 773: def XeVM_SubgroupSizeOp : XeVM_SpecialIdRegisterOp<"subgroup_size">;
 774: 
 775: #endif // XEVMIR_OPS
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L769:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L769:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L770:** This comment states: “Lane, Subgroup index and range”, documenting the intent of the surrounding code.
  **CN L770:** 该注释写道：“Lane, Subgroup index and range”，用于说明周围代码的意图。
- **EN L771:** This TableGen `def` record introduces `XeVM_LaneIdOp`, which later participates in generated MLIR code.
  **CN L771:** 该 TableGen `def` 记录引入了 `XeVM_LaneIdOp`，后续会参与生成的 MLIR 代码。
- **EN L772:** This TableGen `def` record introduces `XeVM_SubgroupIdOp`, which later participates in generated MLIR code.
  **CN L772:** 该 TableGen `def` 记录引入了 `XeVM_SubgroupIdOp`，后续会参与生成的 MLIR 代码。
- **EN L773:** This TableGen `def` record introduces `XeVM_SubgroupSizeOp`, which later participates in generated MLIR code.
  **CN L773:** 该 TableGen `def` 记录引入了 `XeVM_SubgroupSizeOp`，后续会参与生成的 MLIR 代码。
- **EN L774:** Blank line used to separate nearby declarations and improve readability.
  **CN L774:** 该空行用于分隔相邻声明并提升可读性。
- **EN L775:** This directive closes the conditional compilation region guarded by `XEVMIR_OPS`.
  **CN L775:** 该指令结束了由 `XEVMIR_OPS` 保护的条件编译区域。

## Key Concepts / 关键概念

- **XeVM_Attr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **XeVM_Op**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **XeVM_SpecialIdRegisterOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **XeVM_Dialect**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **XeVM_ElemType**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **XeVM_1DBlockElemType**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **LoadCacheControl_Use_Default**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **LoadCacheControl_L1uc_L2uc_L3uc**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。

## Dependencies / 依赖关系

- **`mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/LLVMIR/LLVMOpBase.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/SideEffectInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/OpBase.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/IR/EnumAttr.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
