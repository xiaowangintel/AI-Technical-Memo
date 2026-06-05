# ROCDLOps.td — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `mlir/include/mlir/Dialect/LLVMIR/ROCDLOps.td` | `mlir/include/mlir/Dialect/LLVMIR/ROCDLOps.td` |
| Repository | `llvm/llvm-project` | `llvm/llvm-project` |
| Purpose | This is the ROCDL IR operation definition file. | 该文件的主要内容为：This is the ROCDL IR operation definition file。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```tablegen
   1: //===-- ROCDLOps.td - ROCDL IR dialect op definition file --*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This is the ROCDL IR operation definition file.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1:** This comment states: “===-- ROCDLOps.td - ROCDL IR dialect op definition file --*- tablegen -*-===”, documenting the intent of the surrounding code.
  **CN L1:** 该注释写道：“===-- ROCDLOps.td - ROCDL IR dialect op definition file --*- tablegen -*-===”，用于说明周围代码的意图。
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
- **EN L9:** This comment states: “This is the ROCDL IR operation definition file.”, documenting the intent of the surrounding code.
  **CN L9:** 该注释写道：“This is the ROCDL IR operation definition file.”，用于说明周围代码的意图。
- **EN L10:** This comment documents context for the surrounding code.
  **CN L10:** 该注释为周围代码提供上下文说明。
- **EN L11:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L11:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L12:** Blank line used to separate nearby declarations and improve readability.
  **CN L12:** 该空行用于分隔相邻声明并提升可读性。

### Lines 13-24 / 第 13-24 行

```tablegen
  13: #ifndef ROCDLIR_OPS
  14: #define ROCDLIR_OPS
  15: 
  16: include "mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td"
  17: include "mlir/Dialect/LLVMIR/LLVMOpBase.td"
  18: include "mlir/Interfaces/SideEffectInterfaces.td"
  19: 
  20: //===----------------------------------------------------------------------===//
  21: // ROCDL dialect definitions
  22: //===----------------------------------------------------------------------===//
  23: 
  24: def ROCDL_Dialect : Dialect {
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L13:** This preprocessor directive manages `ROCDLIR_OPS` as part of the file's conditional compilation boundary.
  **CN L13:** 该预处理指令管理 `ROCDLIR_OPS`，作为文件条件编译边界的一部分。
- **EN L14:** This preprocessor directive manages `ROCDLIR_OPS` as part of the file's conditional compilation boundary.
  **CN L14:** 该预处理指令管理 `ROCDLIR_OPS`，作为文件条件编译边界的一部分。
- **EN L15:** Blank line used to separate nearby declarations and improve readability.
  **CN L15:** 该空行用于分隔相邻声明并提升可读性。
- **EN L16:** This TableGen include reuses records from `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td`.
  **CN L16:** 该 TableGen include 复用了 `mlir/Dialect/GPU/IR/CompilationAttrInterfaces.td` 中的记录。
- **EN L17:** This TableGen include reuses records from `mlir/Dialect/LLVMIR/LLVMOpBase.td`.
  **CN L17:** 该 TableGen include 复用了 `mlir/Dialect/LLVMIR/LLVMOpBase.td` 中的记录。
- **EN L18:** This TableGen include reuses records from `mlir/Interfaces/SideEffectInterfaces.td`.
  **CN L18:** 该 TableGen include 复用了 `mlir/Interfaces/SideEffectInterfaces.td` 中的记录。
- **EN L19:** Blank line used to separate nearby declarations and improve readability.
  **CN L19:** 该空行用于分隔相邻声明并提升可读性。
- **EN L20:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L20:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L21:** This comment states: “ROCDL dialect definitions”, documenting the intent of the surrounding code.
  **CN L21:** 该注释写道：“ROCDL dialect definitions”，用于说明周围代码的意图。
- **EN L22:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L22:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L23:** Blank line used to separate nearby declarations and improve readability.
  **CN L23:** 该空行用于分隔相邻声明并提升可读性。
- **EN L24:** This TableGen `def` record introduces `ROCDL_Dialect`, which later participates in generated MLIR code.
  **CN L24:** 该 TableGen `def` 记录引入了 `ROCDL_Dialect`，后续会参与生成的 MLIR 代码。

### Lines 25-36 / 第 25-36 行

```tablegen
  25:   let name = "rocdl";
  26:   let cppNamespace = "::mlir::ROCDL";
  27:   let dependentDialects = ["LLVM::LLVMDialect"];
  28:   let summary = "Dialect for wrapping LLVM AMDGPU backend intrinsics and attributes";
  29:   let hasOperationAttrVerify = 1;
  30: 
  31:   let description = [{
  32:     The ROCDL dialect, like the other platform-specific LLVM dialects, serves
  33:     as the location of wrappers around the AMD-specific intrinsics and attributes
  34:     in LLVM.
  35: 
  36:     This dialect, like other GPU lowering targets, also contains the infrastructure
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L25:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L25:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L26:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L26:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L27:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L27:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L28:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L28:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L29:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L29:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L30:** Blank line used to separate nearby declarations and improve readability.
  **CN L30:** 该空行用于分隔相邻声明并提升可读性。
- **EN L31:** This line contributes implementation detail or declarative structure to the file.
  **CN L31:** 这一行为文件补充了实现细节或声明式结构。
- **EN L32:** This line contributes implementation detail or declarative structure to the file.
  **CN L32:** 这一行为文件补充了实现细节或声明式结构。
- **EN L33:** This line contributes implementation detail or declarative structure to the file.
  **CN L33:** 这一行为文件补充了实现细节或声明式结构。
- **EN L34:** This line contributes implementation detail or declarative structure to the file.
  **CN L34:** 这一行为文件补充了实现细节或声明式结构。
- **EN L35:** Blank line used to separate nearby declarations and improve readability.
  **CN L35:** 该空行用于分隔相邻声明并提升可读性。
- **EN L36:** This line contributes implementation detail or declarative structure to the file.
  **CN L36:** 这一行为文件补充了实现细节或声明式结构。

### Lines 37-48 / 第 37-48 行

```tablegen
  37:     used by the built-in compilation/offloading framework to compile AMD-specific
  38:     LLVM IR into binaries.
  39: 
  40:     # Dialect inclusion criteria and guidelines
  41: 
  42:     The operations in this dialect are 1:1 wrappers around their corresponding
  43:     LLVM intrinsics. Operations that do not correspond to intrinsics should not
  44:     be placed in this dialect.
  45: 
  46:     The definition of a ROCDL op should match its LLVM counterpart. If the
  47:     argument and result types are fixed, they should be specified as type
  48:     constraints, including by overriding the default variadic type on LLVM
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L37:** This line contributes implementation detail or declarative structure to the file.
  **CN L37:** 这一行为文件补充了实现细节或声明式结构。
- **EN L38:** This line contributes implementation detail or declarative structure to the file.
  **CN L38:** 这一行为文件补充了实现细节或声明式结构。
- **EN L39:** Blank line used to separate nearby declarations and improve readability.
  **CN L39:** 该空行用于分隔相邻声明并提升可读性。
- **EN L40:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L40:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L41:** Blank line used to separate nearby declarations and improve readability.
  **CN L41:** 该空行用于分隔相邻声明并提升可读性。
- **EN L42:** This line contributes implementation detail or declarative structure to the file.
  **CN L42:** 这一行为文件补充了实现细节或声明式结构。
- **EN L43:** This line contributes implementation detail or declarative structure to the file.
  **CN L43:** 这一行为文件补充了实现细节或声明式结构。
- **EN L44:** This line contributes implementation detail or declarative structure to the file.
  **CN L44:** 这一行为文件补充了实现细节或声明式结构。
- **EN L45:** Blank line used to separate nearby declarations and improve readability.
  **CN L45:** 该空行用于分隔相邻声明并提升可读性。
- **EN L46:** This line contributes implementation detail or declarative structure to the file.
  **CN L46:** 这一行为文件补充了实现细节或声明式结构。
- **EN L47:** This line contributes implementation detail or declarative structure to the file.
  **CN L47:** 这一行为文件补充了实现细节或声明式结构。
- **EN L48:** This line contributes implementation detail or declarative structure to the file.
  **CN L48:** 这一行为文件补充了实现细节或声明式结构。

### Lines 49-60 / 第 49-60 行

```tablegen
  49:     intrinsics by doing a `let results` in the operation definition.
  50: 
  51:     LLVM attributes do not need to be replicated exactly if it wouldn't be
  52:     easy to do so, but pure operations and ones that read/write memory should
  53:     be annotated as such.
  54: 
  55:     While LLVM intrinsics currently don't allow constraining the values an
  56:     `any_type` can take, it is acceptable (but not required) to impose such
  57:     constraints if they are known.
  58: 
  59:     When an LLVM intrinsic uses an `immarg`, this corresponds to an attribute
  60:     in MLIR.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L49:** This line contributes implementation detail or declarative structure to the file.
  **CN L49:** 这一行为文件补充了实现细节或声明式结构。
- **EN L50:** Blank line used to separate nearby declarations and improve readability.
  **CN L50:** 该空行用于分隔相邻声明并提升可读性。
- **EN L51:** This line contributes implementation detail or declarative structure to the file.
  **CN L51:** 这一行为文件补充了实现细节或声明式结构。
- **EN L52:** This line contributes implementation detail or declarative structure to the file.
  **CN L52:** 这一行为文件补充了实现细节或声明式结构。
- **EN L53:** This line contributes implementation detail or declarative structure to the file.
  **CN L53:** 这一行为文件补充了实现细节或声明式结构。
- **EN L54:** Blank line used to separate nearby declarations and improve readability.
  **CN L54:** 该空行用于分隔相邻声明并提升可读性。
- **EN L55:** This line contributes implementation detail or declarative structure to the file.
  **CN L55:** 这一行为文件补充了实现细节或声明式结构。
- **EN L56:** This line contributes to the declaration or call of `acceptable`.
  **CN L56:** 这一行为 `acceptable` 的声明或调用提供内容。
- **EN L57:** This line contributes implementation detail or declarative structure to the file.
  **CN L57:** 这一行为文件补充了实现细节或声明式结构。
- **EN L58:** Blank line used to separate nearby declarations and improve readability.
  **CN L58:** 该空行用于分隔相邻声明并提升可读性。
- **EN L59:** This line contributes implementation detail or declarative structure to the file.
  **CN L59:** 这一行为文件补充了实现细节或声明式结构。
- **EN L60:** This line contributes implementation detail or declarative structure to the file.
  **CN L60:** 这一行为文件补充了实现细节或声明式结构。

### Lines 61-72 / 第 61-72 行

```tablegen
  61: 
  62:     Human-readable assembly formats (those that, for example, explicitly indicate
  63:     parameter names) may be used, and are encouraged for intrinsics that have
  64:     complex argument schemes and don't have any higher-level wrapper (such as
  65:     in the `amdgpu` dialect).
  66: 
  67:     While not all existing operations follow this convention, new operations should
  68:     generally provide argument and result types except in cases where they are
  69:     clearly redundant (such as with operations like `rocdl.fmed3`, which doesn't
  70:     need to reiterate the single type at issue multiple times). This convention
  71:     enhances the readability of low-level IR and prevents programmers from needing
  72:     to find non-local type information.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L61:** Blank line used to separate nearby declarations and improve readability.
  **CN L61:** 该空行用于分隔相邻声明并提升可读性。
- **EN L62:** This line contributes to the declaration or call of `formats`.
  **CN L62:** 这一行为 `formats` 的声明或调用提供内容。
- **EN L63:** This line contributes implementation detail or declarative structure to the file.
  **CN L63:** 这一行为文件补充了实现细节或声明式结构。
- **EN L64:** This line contributes to the declaration or call of `wrapper`.
  **CN L64:** 这一行为 `wrapper` 的声明或调用提供内容。
- **EN L65:** This line contributes implementation detail or declarative structure to the file.
  **CN L65:** 这一行为文件补充了实现细节或声明式结构。
- **EN L66:** Blank line used to separate nearby declarations and improve readability.
  **CN L66:** 该空行用于分隔相邻声明并提升可读性。
- **EN L67:** This line contributes implementation detail or declarative structure to the file.
  **CN L67:** 这一行为文件补充了实现细节或声明式结构。
- **EN L68:** This line contributes implementation detail or declarative structure to the file.
  **CN L68:** 这一行为文件补充了实现细节或声明式结构。
- **EN L69:** This line contributes to the declaration or call of `redundant`.
  **CN L69:** 这一行为 `redundant` 的声明或调用提供内容。
- **EN L70:** This line contributes implementation detail or declarative structure to the file.
  **CN L70:** 这一行为文件补充了实现细节或声明式结构。
- **EN L71:** This line contributes implementation detail or declarative structure to the file.
  **CN L71:** 这一行为文件补充了实现细节或声明式结构。
- **EN L72:** This line contributes implementation detail or declarative structure to the file.
  **CN L72:** 这一行为文件补充了实现细节或声明式结构。

### Lines 73-84 / 第 73-84 行

```tablegen
  73: 
  74:     Dialect-defined discardable attributes (any attribute starting with `rocdl.`
  75:     that has special handling) need to correspond to AMD-specific attributes, metadata,
  76:     or other entities (such as calling conventions) in LLVM, or be needed for
  77:     GPU compilation management. Outside of the compilation infrastructure,
  78:     dialect-specific enums or attributes are extmelely unlikely to be needed
  79:     and should be avoided.
  80: 
  81:     Operation documentation should specify when the operation was introduced
  82:     (if relevant) and include usage examples. Operations should have
  83:     parser/printer tests in `mlir/test/Dialect/LLVMIR/rocdl.mlir` and
  84:     lowering tests in `mlir/test/Target/LLVMIR/rocdl.mlir`.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L73:** Blank line used to separate nearby declarations and improve readability.
  **CN L73:** 该空行用于分隔相邻声明并提升可读性。
- **EN L74:** This line contributes to the declaration or call of `attributes`.
  **CN L74:** 这一行为 `attributes` 的声明或调用提供内容。
- **EN L75:** This line contributes implementation detail or declarative structure to the file.
  **CN L75:** 这一行为文件补充了实现细节或声明式结构。
- **EN L76:** This line contributes to the declaration or call of `entities`.
  **CN L76:** 这一行为 `entities` 的声明或调用提供内容。
- **EN L77:** This line contributes implementation detail or declarative structure to the file.
  **CN L77:** 这一行为文件补充了实现细节或声明式结构。
- **EN L78:** This line contributes implementation detail or declarative structure to the file.
  **CN L78:** 这一行为文件补充了实现细节或声明式结构。
- **EN L79:** This line contributes implementation detail or declarative structure to the file.
  **CN L79:** 这一行为文件补充了实现细节或声明式结构。
- **EN L80:** Blank line used to separate nearby declarations and improve readability.
  **CN L80:** 该空行用于分隔相邻声明并提升可读性。
- **EN L81:** This line contributes implementation detail or declarative structure to the file.
  **CN L81:** 这一行为文件补充了实现细节或声明式结构。
- **EN L82:** This line contributes implementation detail or declarative structure to the file.
  **CN L82:** 这一行为文件补充了实现细节或声明式结构。
- **EN L83:** This line contributes implementation detail or declarative structure to the file.
  **CN L83:** 这一行为文件补充了实现细节或声明式结构。
- **EN L84:** This line contributes implementation detail or declarative structure to the file.
  **CN L84:** 这一行为文件补充了实现细节或声明式结构。

### Lines 85-96 / 第 85-96 行

```tablegen
  85: 
  86:     # General documentation (What does this op do?)
  87: 
  88:     While rocdl ops sometimes carry their own documentation, there is no
  89:     expectation that such documentation will exist (or be kept up to date).
  90: 
  91:     Since ROCDL operations correspond to LLVM intrinsics, the semantics and
  92:     behavior of these operations can be determined by investigating the
  93:     documentation for the corresponding intrinsic. This documentation
  94:     can be found in
  95:     - `llvm/docs/AMDGPUUsage.rst` and
  96:     - The comments of `llvm/include/llvm/IR/IntrinsicsAMDGPU.td`, which
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L85:** Blank line used to separate nearby declarations and improve readability.
  **CN L85:** 该空行用于分隔相邻声明并提升可读性。
- **EN L86:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L86:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L87:** Blank line used to separate nearby declarations and improve readability.
  **CN L87:** 该空行用于分隔相邻声明并提升可读性。
- **EN L88:** This line contributes implementation detail or declarative structure to the file.
  **CN L88:** 这一行为文件补充了实现细节或声明式结构。
- **EN L89:** This line contributes to the declaration or call of `exist`.
  **CN L89:** 这一行为 `exist` 的声明或调用提供内容。
- **EN L90:** Blank line used to separate nearby declarations and improve readability.
  **CN L90:** 该空行用于分隔相邻声明并提升可读性。
- **EN L91:** This line contributes implementation detail or declarative structure to the file.
  **CN L91:** 这一行为文件补充了实现细节或声明式结构。
- **EN L92:** This line contributes implementation detail or declarative structure to the file.
  **CN L92:** 这一行为文件补充了实现细节或声明式结构。
- **EN L93:** This line contributes implementation detail or declarative structure to the file.
  **CN L93:** 这一行为文件补充了实现细节或声明式结构。
- **EN L94:** This line contributes implementation detail or declarative structure to the file.
  **CN L94:** 这一行为文件补充了实现细节或声明式结构。
- **EN L95:** This line contributes implementation detail or declarative structure to the file.
  **CN L95:** 这一行为文件补充了实现细节或声明式结构。
- **EN L96:** This line contributes implementation detail or declarative structure to the file.
  **CN L96:** 这一行为文件补充了实现细节或声明式结构。

### Lines 97-108 / 第 97-108 行

```tablegen
  97:       is where details of the meaning of certain bitfields or of how an
  98:       intrinsic corresponds to hardware instructions are most likely to
  99:       be found.
 100: 
 101:     Since many intrinsics are themselves minimal wrappers around hardware
 102:     instructions, these documentation sources often do not repeat hardware
 103:     documentation. If an intrinsic appears undocumented, information about
 104:     its behavior will often be available in published ISA descriptions or
 105:     (sometimes known as shader programming guides).
 106: 
 107:     If an operation doesn't provide usage examples, it is likely that they
 108:     can be found in `mlir/test/Dialect/LLVMIR/rocdl.mlir` (op syntax and
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L97:** This line contributes implementation detail or declarative structure to the file.
  **CN L97:** 这一行为文件补充了实现细节或声明式结构。
- **EN L98:** This line contributes implementation detail or declarative structure to the file.
  **CN L98:** 这一行为文件补充了实现细节或声明式结构。
- **EN L99:** This line contributes implementation detail or declarative structure to the file.
  **CN L99:** 这一行为文件补充了实现细节或声明式结构。
- **EN L100:** Blank line used to separate nearby declarations and improve readability.
  **CN L100:** 该空行用于分隔相邻声明并提升可读性。
- **EN L101:** This line contributes implementation detail or declarative structure to the file.
  **CN L101:** 这一行为文件补充了实现细节或声明式结构。
- **EN L102:** This line contributes implementation detail or declarative structure to the file.
  **CN L102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L103:** This line contributes implementation detail or declarative structure to the file.
  **CN L103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L104:** This line contributes implementation detail or declarative structure to the file.
  **CN L104:** 这一行为文件补充了实现细节或声明式结构。
- **EN L105:** This line contributes implementation detail or declarative structure to the file.
  **CN L105:** 这一行为文件补充了实现细节或声明式结构。
- **EN L106:** Blank line used to separate nearby declarations and improve readability.
  **CN L106:** 该空行用于分隔相邻声明并提升可读性。
- **EN L107:** This line contributes implementation detail or declarative structure to the file.
  **CN L107:** 这一行为文件补充了实现细节或声明式结构。
- **EN L108:** This line contributes implementation detail or declarative structure to the file.
  **CN L108:** 这一行为文件补充了实现细节或声明式结构。

### Lines 109-120 / 第 109-120 行

```tablegen
 109:     verification) or `mlir/test/Target/LLVMIR/rocdl.mlir` (translation
 110:     to LLVM IR).
 111:   }];
 112: 
 113:   let extraClassDeclaration = [{
 114:     /// Get the name of the attribute used to annotate external kernel
 115:     /// functions.
 116:     static StringRef getKernelFuncAttrName() { return "rocdl.kernel"; }
 117:     static constexpr ::llvm::StringLiteral getFlatWorkGroupSizeAttrName() {
 118:       return ::llvm::StringLiteral("rocdl.flat_work_group_size");
 119:     }
 120:     static constexpr ::llvm::StringLiteral getReqdWorkGroupSizeAttrName() {
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L109:** This line contributes implementation detail or declarative structure to the file.
  **CN L109:** 这一行为文件补充了实现细节或声明式结构。
- **EN L110:** This line contributes implementation detail or declarative structure to the file.
  **CN L110:** 这一行为文件补充了实现细节或声明式结构。
- **EN L111:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L111:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L112:** Blank line used to separate nearby declarations and improve readability.
  **CN L112:** 该空行用于分隔相邻声明并提升可读性。
- **EN L113:** This line contributes implementation detail or declarative structure to the file.
  **CN L113:** 这一行为文件补充了实现细节或声明式结构。
- **EN L114:** This comment states: “Get the name of the attribute used to annotate external kernel”, documenting the intent of the surrounding code.
  **CN L114:** 该注释写道：“Get the name of the attribute used to annotate external kernel”，用于说明周围代码的意图。
- **EN L115:** This comment states: “functions.”, documenting the intent of the surrounding code.
  **CN L115:** 该注释写道：“functions.”，用于说明周围代码的意图。
- **EN L116:** This line contributes to the declaration or call of `getKernelFuncAttrName`.
  **CN L116:** 这一行为 `getKernelFuncAttrName` 的声明或调用提供内容。
- **EN L117:** This line contributes to the declaration or call of `getFlatWorkGroupSizeAttrName`.
  **CN L117:** 这一行为 `getFlatWorkGroupSizeAttrName` 的声明或调用提供内容。
- **EN L118:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L118:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L119:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L119:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L120:** This line contributes to the declaration or call of `getReqdWorkGroupSizeAttrName`.
  **CN L120:** 这一行为 `getReqdWorkGroupSizeAttrName` 的声明或调用提供内容。

### Lines 121-132 / 第 121-132 行

```tablegen
 121:       return ::llvm::StringLiteral("rocdl.reqd_work_group_size");
 122:     }
 123:     /// MLIR's gpu-related infrastructure effectively assume uniform workgroup
 124:     /// sizes, so this attribute defaults to "true" on `rocdl.kernel` functions.
 125:     /// It is provided here to allow overriding this assumption.
 126:     static constexpr ::llvm::StringLiteral getUniformWorkGroupSizeAttrName() {
 127:       return ::llvm::StringLiteral("rocdl.uniform_work_group_size");
 128:     }
 129: 
 130:     /// The address space value that represents global memory.
 131:     static constexpr unsigned kGlobalMemoryAddressSpace = 1;
 132:     /// The address space value that represents shared memory.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L121:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L121:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L122:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L122:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L123:** This comment states: “MLIR's gpu-related infrastructure effectively assume uniform workgroup”, documenting the intent of the surrounding code.
  **CN L123:** 该注释写道：“MLIR's gpu-related infrastructure effectively assume uniform workgroup”，用于说明周围代码的意图。
- **EN L124:** This comment states: “sizes, so this attribute defaults to "true" on `rocdl.kernel` functions.”, documenting the intent of the surrounding code.
  **CN L124:** 该注释写道：“sizes, so this attribute defaults to "true" on `rocdl.kernel` functions.”，用于说明周围代码的意图。
- **EN L125:** This comment states: “It is provided here to allow overriding this assumption.”, documenting the intent of the surrounding code.
  **CN L125:** 该注释写道：“It is provided here to allow overriding this assumption.”，用于说明周围代码的意图。
- **EN L126:** This line contributes to the declaration or call of `getUniformWorkGroupSizeAttrName`.
  **CN L126:** 这一行为 `getUniformWorkGroupSizeAttrName` 的声明或调用提供内容。
- **EN L127:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L127:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L128:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L128:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L129:** Blank line used to separate nearby declarations and improve readability.
  **CN L129:** 该空行用于分隔相邻声明并提升可读性。
- **EN L130:** This comment states: “The address space value that represents global memory.”, documenting the intent of the surrounding code.
  **CN L130:** 该注释写道：“The address space value that represents global memory.”，用于说明周围代码的意图。
- **EN L131:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L131:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L132:** This comment states: “The address space value that represents shared memory.”, documenting the intent of the surrounding code.
  **CN L132:** 该注释写道：“The address space value that represents shared memory.”，用于说明周围代码的意图。

### Lines 133-144 / 第 133-144 行

```tablegen
 133:     static constexpr unsigned kSharedMemoryAddressSpace = 3;
 134:     /// The address space value that represents constant memory.
 135:     static constexpr unsigned kConstantMemoryAddressSpace = 4;
 136:     /// The address space value that represents private memory.
 137:     static constexpr unsigned kPrivateMemoryAddressSpace = 5;
 138:   }];
 139: 
 140:   let discardableAttrs = (ins
 141:      "::mlir::UnitAttr":$kernel,
 142:      "::mlir::DenseI32ArrayAttr":$reqd_work_group_size,
 143:      "::mlir::StringAttr":$flat_work_group_size,
 144:      "::mlir::IntegerAttr":$max_flat_work_group_size,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L133:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L133:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L134:** This comment states: “The address space value that represents constant memory.”, documenting the intent of the surrounding code.
  **CN L134:** 该注释写道：“The address space value that represents constant memory.”，用于说明周围代码的意图。
- **EN L135:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L135:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L136:** This comment states: “The address space value that represents private memory.”, documenting the intent of the surrounding code.
  **CN L136:** 该注释写道：“The address space value that represents private memory.”，用于说明周围代码的意图。
- **EN L137:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L137:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L138:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L138:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L139:** Blank line used to separate nearby declarations and improve readability.
  **CN L139:** 该空行用于分隔相邻声明并提升可读性。
- **EN L140:** This line contributes implementation detail or declarative structure to the file.
  **CN L140:** 这一行为文件补充了实现细节或声明式结构。
- **EN L141:** This line contributes implementation detail or declarative structure to the file.
  **CN L141:** 这一行为文件补充了实现细节或声明式结构。
- **EN L142:** This line contributes implementation detail or declarative structure to the file.
  **CN L142:** 这一行为文件补充了实现细节或声明式结构。
- **EN L143:** This line contributes implementation detail or declarative structure to the file.
  **CN L143:** 这一行为文件补充了实现细节或声明式结构。
- **EN L144:** This line contributes implementation detail or declarative structure to the file.
  **CN L144:** 这一行为文件补充了实现细节或声明式结构。

### Lines 145-156 / 第 145-156 行

```tablegen
 145:      "::mlir::IntegerAttr":$waves_per_eu,
 146:      "::mlir::BoolAttr":$unsafe_fp_atomics,
 147:      // Correspond to LLVM metadata of the same name
 148:      "::mlir::UnitAttr":$last_use,
 149:      "::mlir::UnitAttr":$no_remote_memory,
 150:      "::mlir::UnitAttr":$no_fine_grained_memory,
 151:      "::mlir::UnitAttr":$ignore_denormal_mode
 152:   );
 153: 
 154:   let useDefaultAttributePrinterParser = 1;
 155: }
 156: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L145:** This line contributes implementation detail or declarative structure to the file.
  **CN L145:** 这一行为文件补充了实现细节或声明式结构。
- **EN L146:** This line contributes implementation detail or declarative structure to the file.
  **CN L146:** 这一行为文件补充了实现细节或声明式结构。
- **EN L147:** This comment states: “Correspond to LLVM metadata of the same name”, documenting the intent of the surrounding code.
  **CN L147:** 该注释写道：“Correspond to LLVM metadata of the same name”，用于说明周围代码的意图。
- **EN L148:** This line contributes implementation detail or declarative structure to the file.
  **CN L148:** 这一行为文件补充了实现细节或声明式结构。
- **EN L149:** This line contributes implementation detail or declarative structure to the file.
  **CN L149:** 这一行为文件补充了实现细节或声明式结构。
- **EN L150:** This line contributes implementation detail or declarative structure to the file.
  **CN L150:** 这一行为文件补充了实现细节或声明式结构。
- **EN L151:** This line contributes implementation detail or declarative structure to the file.
  **CN L151:** 这一行为文件补充了实现细节或声明式结构。
- **EN L152:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L152:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L153:** Blank line used to separate nearby declarations and improve readability.
  **CN L153:** 该空行用于分隔相邻声明并提升可读性。
- **EN L154:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L154:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L155:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L155:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L156:** Blank line used to separate nearby declarations and improve readability.
  **CN L156:** 该空行用于分隔相邻声明并提升可读性。

### Lines 157-168 / 第 157-168 行

```tablegen
 157: //===----------------------------------------------------------------------===//
 158: // ROCDL attribute definitions
 159: //===----------------------------------------------------------------------===//
 160: 
 161: class ROCDL_Attr<string attrName, string attrMnemonic, list<Trait> traits = []>
 162:     : AttrDef<ROCDL_Dialect, attrName, traits> {
 163:   let mnemonic = attrMnemonic;
 164: }
 165: 
 166: 
 167: //===----------------------------------------------------------------------===//
 168: // ROCDL op definitions
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L157:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L157:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L158:** This comment states: “ROCDL attribute definitions”, documenting the intent of the surrounding code.
  **CN L158:** 该注释写道：“ROCDL attribute definitions”，用于说明周围代码的意图。
- **EN L159:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L159:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L160:** Blank line used to separate nearby declarations and improve readability.
  **CN L160:** 该空行用于分隔相邻声明并提升可读性。
- **EN L161:** This TableGen `class` record introduces `ROCDL_Attr`, which later participates in generated MLIR code.
  **CN L161:** 该 TableGen `class` 记录引入了 `ROCDL_Attr`，后续会参与生成的 MLIR 代码。
- **EN L162:** This line contributes implementation detail or declarative structure to the file.
  **CN L162:** 这一行为文件补充了实现细节或声明式结构。
- **EN L163:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L163:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L164:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L164:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L165:** Blank line used to separate nearby declarations and improve readability.
  **CN L165:** 该空行用于分隔相邻声明并提升可读性。
- **EN L166:** Blank line used to separate nearby declarations and improve readability.
  **CN L166:** 该空行用于分隔相邻声明并提升可读性。
- **EN L167:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L167:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L168:** This comment states: “ROCDL op definitions”, documenting the intent of the surrounding code.
  **CN L168:** 该注释写道：“ROCDL op definitions”，用于说明周围代码的意图。

### Lines 169-180 / 第 169-180 行

```tablegen
 169: //===----------------------------------------------------------------------===//
 170: 
 171: class ROCDL_Op<string mnemonic, list<Trait> traits = []> :
 172:   LLVM_OpBase<ROCDL_Dialect, mnemonic, traits> {
 173: }
 174: 
 175: class ROCDL_IntrPure1Op<string mnemonic> :
 176:   LLVM_IntrOpBase<ROCDL_Dialect, mnemonic,
 177:   "amdgcn_" # !subst(".", "_", mnemonic), [], [], [Pure], 1>;
 178: 
 179: class ROCDL_IntrOp<string mnemonic, list<int> overloadedResults,
 180:   list<int> overloadedOperands, list<Trait> traits, int numResults,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L169:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L169:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L170:** Blank line used to separate nearby declarations and improve readability.
  **CN L170:** 该空行用于分隔相邻声明并提升可读性。
- **EN L171:** This TableGen `class` record introduces `ROCDL_Op`, which later participates in generated MLIR code.
  **CN L171:** 该 TableGen `class` 记录引入了 `ROCDL_Op`，后续会参与生成的 MLIR 代码。
- **EN L172:** This line contributes implementation detail or declarative structure to the file.
  **CN L172:** 这一行为文件补充了实现细节或声明式结构。
- **EN L173:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L173:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L174:** Blank line used to separate nearby declarations and improve readability.
  **CN L174:** 该空行用于分隔相邻声明并提升可读性。
- **EN L175:** This TableGen `class` record introduces `ROCDL_IntrPure1Op`, which later participates in generated MLIR code.
  **CN L175:** 该 TableGen `class` 记录引入了 `ROCDL_IntrPure1Op`，后续会参与生成的 MLIR 代码。
- **EN L176:** This line contributes implementation detail or declarative structure to the file.
  **CN L176:** 这一行为文件补充了实现细节或声明式结构。
- **EN L177:** This line contributes to the declaration or call of `subst`.
  **CN L177:** 这一行为 `subst` 的声明或调用提供内容。
- **EN L178:** Blank line used to separate nearby declarations and improve readability.
  **CN L178:** 该空行用于分隔相邻声明并提升可读性。
- **EN L179:** This TableGen `class` record introduces `ROCDL_IntrOp`, which later participates in generated MLIR code.
  **CN L179:** 该 TableGen `class` 记录引入了 `ROCDL_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L180:** This line contributes implementation detail or declarative structure to the file.
  **CN L180:** 这一行为文件补充了实现细节或声明式结构。

### Lines 181-192 / 第 181-192 行

```tablegen
 181:   int requiresAccessGroup = 0, int requiresAliasAnalysis = 0,
 182:   int requiresArgAndResultAttrs = 0,
 183:   list<int> immArgPositions = [],
 184:   list<string> immArgAttrNames = []> :
 185:   LLVM_IntrOpBase<ROCDL_Dialect,  mnemonic,
 186:     "amdgcn_" # !subst(".", "_", mnemonic), overloadedResults,
 187:     overloadedOperands, traits, numResults, requiresAccessGroup,
 188:     requiresAliasAnalysis, 0, requiresArgAndResultAttrs, 0,
 189:     immArgPositions, immArgAttrNames>;
 190: 
 191: // Subclass to save typing and ease readibility when there aren't overloaded
 192: // operands or memory accesses.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
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
- **EN L186:** This line contributes to the declaration or call of `subst`.
  **CN L186:** 这一行为 `subst` 的声明或调用提供内容。
- **EN L187:** This line contributes implementation detail or declarative structure to the file.
  **CN L187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L188:** This line contributes implementation detail or declarative structure to the file.
  **CN L188:** 这一行为文件补充了实现细节或声明式结构。
- **EN L189:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L189:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L190:** Blank line used to separate nearby declarations and improve readability.
  **CN L190:** 该空行用于分隔相邻声明并提升可读性。
- **EN L191:** This comment states: “Subclass to save typing and ease readibility when there aren't overloaded”, documenting the intent of the surrounding code.
  **CN L191:** 该注释写道：“Subclass to save typing and ease readibility when there aren't overloaded”，用于说明周围代码的意图。
- **EN L192:** This comment states: “operands or memory accesses.”, documenting the intent of the surrounding code.
  **CN L192:** 该注释写道：“operands or memory accesses.”，用于说明周围代码的意图。

### Lines 193-204 / 第 193-204 行

```tablegen
 193: class ROCDL_ConcreteNonMemIntrOp<string mnemonic, list<Trait> traits,
 194:     int numResults, list<int> immArgPositions = [],
 195:     list<string> immArgNames = []>
 196:   : ROCDL_IntrOp<mnemonic, [], [], traits, numResults, 0, 0, 0,
 197:       immArgPositions, immArgNames>;
 198: //===----------------------------------------------------------------------===//
 199: // ROCDL special register op definitions
 200: //===----------------------------------------------------------------------===//
 201: 
 202: class ROCDL_SpecialIdRegisterOp<string mnemonic> :
 203:     ROCDL_IntrPure1Op<mnemonic>,
 204:     Arguments<(ins OptionalAttr<LLVM_ConstantRangeAttr>:$range)> {
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L193:** This TableGen `class` record introduces `ROCDL_ConcreteNonMemIntrOp`, which later participates in generated MLIR code.
  **CN L193:** 该 TableGen `class` 记录引入了 `ROCDL_ConcreteNonMemIntrOp`，后续会参与生成的 MLIR 代码。
- **EN L194:** This line contributes implementation detail or declarative structure to the file.
  **CN L194:** 这一行为文件补充了实现细节或声明式结构。
- **EN L195:** This line contributes implementation detail or declarative structure to the file.
  **CN L195:** 这一行为文件补充了实现细节或声明式结构。
- **EN L196:** This line contributes implementation detail or declarative structure to the file.
  **CN L196:** 这一行为文件补充了实现细节或声明式结构。
- **EN L197:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L197:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L198:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L198:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L199:** This comment states: “ROCDL special register op definitions”, documenting the intent of the surrounding code.
  **CN L199:** 该注释写道：“ROCDL special register op definitions”，用于说明周围代码的意图。
- **EN L200:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L200:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L201:** Blank line used to separate nearby declarations and improve readability.
  **CN L201:** 该空行用于分隔相邻声明并提升可读性。
- **EN L202:** This TableGen `class` record introduces `ROCDL_SpecialIdRegisterOp`, which later participates in generated MLIR code.
  **CN L202:** 该 TableGen `class` 记录引入了 `ROCDL_SpecialIdRegisterOp`，后续会参与生成的 MLIR 代码。
- **EN L203:** This line contributes implementation detail or declarative structure to the file.
  **CN L203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L204:** This line contributes implementation detail or declarative structure to the file.
  **CN L204:** 这一行为文件补充了实现细节或声明式结构。

### Lines 205-216 / 第 205-216 行

```tablegen
 205:   string llvmBuilder = baseLlvmBuilder # setRangeRetAttrCode # baseLlvmBuilderCoda;
 206:   string mlirBuilder = baseMlirBuilder # importRangeRetAttrCode # baseMlirBuilderCoda;
 207: 
 208:   let assemblyFormat = "(`range` $range^)? attr-dict `:` type($res)";
 209: 
 210:     // Temporaly builder until Nvidia ops also support range attributes.
 211:   let builders = [
 212:     OpBuilder<(ins "Type":$resultType), [{
 213:       build($_builder, $_state, resultType, ::mlir::LLVM::ConstantRangeAttr{});
 214:     }]>
 215:   ];
 216:   let description = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L205:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L205:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L206:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L206:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L207:** Blank line used to separate nearby declarations and improve readability.
  **CN L207:** 该空行用于分隔相邻声明并提升可读性。
- **EN L208:** This line contributes to the declaration or call of `type`.
  **CN L208:** 这一行为 `type` 的声明或调用提供内容。
- **EN L209:** Blank line used to separate nearby declarations and improve readability.
  **CN L209:** 该空行用于分隔相邻声明并提升可读性。
- **EN L210:** This comment states: “Temporaly builder until Nvidia ops also support range attributes.”, documenting the intent of the surrounding code.
  **CN L210:** 该注释写道：“Temporaly builder until Nvidia ops also support range attributes.”，用于说明周围代码的意图。
- **EN L211:** This line contributes implementation detail or declarative structure to the file.
  **CN L211:** 这一行为文件补充了实现细节或声明式结构。
- **EN L212:** This line contributes implementation detail or declarative structure to the file.
  **CN L212:** 这一行为文件补充了实现细节或声明式结构。
- **EN L213:** This line contributes to the declaration or call of `build`.
  **CN L213:** 这一行为 `build` 的声明或调用提供内容。
- **EN L214:** This line contributes implementation detail or declarative structure to the file.
  **CN L214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L215:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L215:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L216:** This line contributes implementation detail or declarative structure to the file.
  **CN L216:** 这一行为文件补充了实现细节或声明式结构。

### Lines 217-228 / 第 217-228 行

```tablegen
 217:     Read a hardware register for thread/workgroup/cluster identification.
 218:     An optional `range` attribute can constrain the returned value.
 219: 
 220:     Example:
 221:     ```mlir
 222:     // Read the workitem id in the x dimension.
 223:     %0 = rocdl.workitem.id.x : i32
 224: 
 225:     // Read with a known range constraint.
 226:     %1 = rocdl.workitem.id.x range <i32, 0, 64> : i32
 227:     ```
 228:   }];
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
- **EN L222:** This comment states: “Read the workitem id in the x dimension.”, documenting the intent of the surrounding code.
  **CN L222:** 该注释写道：“Read the workitem id in the x dimension.”，用于说明周围代码的意图。
- **EN L223:** This line contributes implementation detail or declarative structure to the file.
  **CN L223:** 这一行为文件补充了实现细节或声明式结构。
- **EN L224:** Blank line used to separate nearby declarations and improve readability.
  **CN L224:** 该空行用于分隔相邻声明并提升可读性。
- **EN L225:** This comment states: “Read with a known range constraint.”, documenting the intent of the surrounding code.
  **CN L225:** 该注释写道：“Read with a known range constraint.”，用于说明周围代码的意图。
- **EN L226:** This line contributes implementation detail or declarative structure to the file.
  **CN L226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L227:** This line contributes implementation detail or declarative structure to the file.
  **CN L227:** 这一行为文件补充了实现细节或声明式结构。
- **EN L228:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L228:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 229-240 / 第 229-240 行

```tablegen
 229: }
 230: 
 231: //===----------------------------------------------------------------------===//
 232: // ROCDL vector types definitions
 233: //===----------------------------------------------------------------------===//
 234: 
 235: class ROCDL_NamedType<string name> {
 236:   string typeName = name;
 237: }
 238: 
 239: class ROCDL_ConcreteVector<Type elem, int length> :
 240:   FixedVectorOfLengthAndType<[length], [elem]>,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L229:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L229:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L230:** Blank line used to separate nearby declarations and improve readability.
  **CN L230:** 该空行用于分隔相邻声明并提升可读性。
- **EN L231:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L231:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L232:** This comment states: “ROCDL vector types definitions”, documenting the intent of the surrounding code.
  **CN L232:** 该注释写道：“ROCDL vector types definitions”，用于说明周围代码的意图。
- **EN L233:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L233:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L234:** Blank line used to separate nearby declarations and improve readability.
  **CN L234:** 该空行用于分隔相邻声明并提升可读性。
- **EN L235:** This TableGen `class` record introduces `ROCDL_NamedType`, which later participates in generated MLIR code.
  **CN L235:** 该 TableGen `class` 记录引入了 `ROCDL_NamedType`，后续会参与生成的 MLIR 代码。
- **EN L236:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L236:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L237:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L237:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L238:** Blank line used to separate nearby declarations and improve readability.
  **CN L238:** 该空行用于分隔相邻声明并提升可读性。
- **EN L239:** This TableGen `class` record introduces `ROCDL_ConcreteVector`, which later participates in generated MLIR code.
  **CN L239:** 该 TableGen `class` 记录引入了 `ROCDL_ConcreteVector`，后续会参与生成的 MLIR 代码。
- **EN L240:** This line contributes implementation detail or declarative structure to the file.
  **CN L240:** 这一行为文件补充了实现细节或声明式结构。

### Lines 241-252 / 第 241-252 行

```tablegen
 241:   BuildableType<
 242:     "::mlir::VectorType::get({" # length # "} ,"
 243:       # elem.builderCall # ")">,
 244:   ROCDL_NamedType<"vector<" # length # "x"
 245:     # !tolower(!cast<string>(elem)) # ">">;
 246: 
 247: class ROCDL_Scalar<Type elem> :
 248:   Type<elem.predicate, elem.summary>,
 249:   BuildableType<elem.builderCall>,
 250:   ROCDL_NamedType<!tolower(!cast<string>(elem))>;
 251: 
 252: def ROCDL_V2I16Type : ROCDL_ConcreteVector<I16, 2>;
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L241:** This line contributes implementation detail or declarative structure to the file.
  **CN L241:** 这一行为文件补充了实现细节或声明式结构。
- **EN L242:** This line contributes to the declaration or call of `get`.
  **CN L242:** 这一行为 `get` 的声明或调用提供内容。
- **EN L243:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L243:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L244:** This line contributes implementation detail or declarative structure to the file.
  **CN L244:** 这一行为文件补充了实现细节或声明式结构。
- **EN L245:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L245:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L246:** Blank line used to separate nearby declarations and improve readability.
  **CN L246:** 该空行用于分隔相邻声明并提升可读性。
- **EN L247:** This TableGen `class` record introduces `ROCDL_Scalar`, which later participates in generated MLIR code.
  **CN L247:** 该 TableGen `class` 记录引入了 `ROCDL_Scalar`，后续会参与生成的 MLIR 代码。
- **EN L248:** This line contributes implementation detail or declarative structure to the file.
  **CN L248:** 这一行为文件补充了实现细节或声明式结构。
- **EN L249:** This line contributes implementation detail or declarative structure to the file.
  **CN L249:** 这一行为文件补充了实现细节或声明式结构。
- **EN L250:** This line contributes to the declaration or call of `tolower`.
  **CN L250:** 这一行为 `tolower` 的声明或调用提供内容。
- **EN L251:** Blank line used to separate nearby declarations and improve readability.
  **CN L251:** 该空行用于分隔相邻声明并提升可读性。
- **EN L252:** This TableGen `def` record introduces `ROCDL_V2I16Type`, which later participates in generated MLIR code.
  **CN L252:** 该 TableGen `def` 记录引入了 `ROCDL_V2I16Type`，后续会参与生成的 MLIR 代码。

### Lines 253-264 / 第 253-264 行

```tablegen
 253: def ROCDL_V2F16Type : ROCDL_ConcreteVector<F16, 2>;
 254: def ROCDL_V2I32Type : ROCDL_ConcreteVector<I32, 2>;
 255: def ROCDL_V2BF16Type : ROCDL_ConcreteVector<BF16, 2>;
 256: def ROCDL_V2F32Type : ROCDL_ConcreteVector<F32, 2>;
 257: def ROCDL_V3I32Type : ROCDL_ConcreteVector<I32, 3>;
 258: def ROCDL_V4I32Type : ROCDL_ConcreteVector<I32, 4>;
 259: def ROCDL_V6I32Type : ROCDL_ConcreteVector<I32, 6>;
 260: def ROCDL_V8I32Type : ROCDL_ConcreteVector<I32, 8>;
 261: def ROCDL_V8BF16Type : ROCDL_ConcreteVector<BF16, 8>;
 262: def ROCDL_V8F16Type : ROCDL_ConcreteVector<F16, 8>;
 263: def ROCDL_V8F32Type : ROCDL_ConcreteVector<F32, 8>;
 264: def ROCDL_V16BF16Type : ROCDL_ConcreteVector<BF16, 16>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L253:** This TableGen `def` record introduces `ROCDL_V2F16Type`, which later participates in generated MLIR code.
  **CN L253:** 该 TableGen `def` 记录引入了 `ROCDL_V2F16Type`，后续会参与生成的 MLIR 代码。
- **EN L254:** This TableGen `def` record introduces `ROCDL_V2I32Type`, which later participates in generated MLIR code.
  **CN L254:** 该 TableGen `def` 记录引入了 `ROCDL_V2I32Type`，后续会参与生成的 MLIR 代码。
- **EN L255:** This TableGen `def` record introduces `ROCDL_V2BF16Type`, which later participates in generated MLIR code.
  **CN L255:** 该 TableGen `def` 记录引入了 `ROCDL_V2BF16Type`，后续会参与生成的 MLIR 代码。
- **EN L256:** This TableGen `def` record introduces `ROCDL_V2F32Type`, which later participates in generated MLIR code.
  **CN L256:** 该 TableGen `def` 记录引入了 `ROCDL_V2F32Type`，后续会参与生成的 MLIR 代码。
- **EN L257:** This TableGen `def` record introduces `ROCDL_V3I32Type`, which later participates in generated MLIR code.
  **CN L257:** 该 TableGen `def` 记录引入了 `ROCDL_V3I32Type`，后续会参与生成的 MLIR 代码。
- **EN L258:** This TableGen `def` record introduces `ROCDL_V4I32Type`, which later participates in generated MLIR code.
  **CN L258:** 该 TableGen `def` 记录引入了 `ROCDL_V4I32Type`，后续会参与生成的 MLIR 代码。
- **EN L259:** This TableGen `def` record introduces `ROCDL_V6I32Type`, which later participates in generated MLIR code.
  **CN L259:** 该 TableGen `def` 记录引入了 `ROCDL_V6I32Type`，后续会参与生成的 MLIR 代码。
- **EN L260:** This TableGen `def` record introduces `ROCDL_V8I32Type`, which later participates in generated MLIR code.
  **CN L260:** 该 TableGen `def` 记录引入了 `ROCDL_V8I32Type`，后续会参与生成的 MLIR 代码。
- **EN L261:** This TableGen `def` record introduces `ROCDL_V8BF16Type`, which later participates in generated MLIR code.
  **CN L261:** 该 TableGen `def` 记录引入了 `ROCDL_V8BF16Type`，后续会参与生成的 MLIR 代码。
- **EN L262:** This TableGen `def` record introduces `ROCDL_V8F16Type`, which later participates in generated MLIR code.
  **CN L262:** 该 TableGen `def` 记录引入了 `ROCDL_V8F16Type`，后续会参与生成的 MLIR 代码。
- **EN L263:** This TableGen `def` record introduces `ROCDL_V8F32Type`, which later participates in generated MLIR code.
  **CN L263:** 该 TableGen `def` 记录引入了 `ROCDL_V8F32Type`，后续会参与生成的 MLIR 代码。
- **EN L264:** This TableGen `def` record introduces `ROCDL_V16BF16Type`, which later participates in generated MLIR code.
  **CN L264:** 该 TableGen `def` 记录引入了 `ROCDL_V16BF16Type`，后续会参与生成的 MLIR 代码。

### Lines 265-276 / 第 265-276 行

```tablegen
 265: def ROCDL_V16F16Type : ROCDL_ConcreteVector<F16, 16>;
 266: def ROCDL_V16F32Type : ROCDL_ConcreteVector<F32, 16>;
 267: def ROCDL_V32F16Type : ROCDL_ConcreteVector<F16, 32>;
 268: def ROCDL_V32BF16Type : ROCDL_ConcreteVector<BF16, 32>;
 269: def ROCDL_V32F32Type : ROCDL_ConcreteVector<F32, 32>;
 270: 
 271: //===----------------------------------------------------------------------===//
 272: // Wave-level primitives
 273: //===----------------------------------------------------------------------===//
 274: 
 275: class ROCDL_MbcntOp<string mnemonic> :
 276:     ROCDL_IntrOp<"mbcnt." # mnemonic, [], [], [Pure], 1,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L265:** This TableGen `def` record introduces `ROCDL_V16F16Type`, which later participates in generated MLIR code.
  **CN L265:** 该 TableGen `def` 记录引入了 `ROCDL_V16F16Type`，后续会参与生成的 MLIR 代码。
- **EN L266:** This TableGen `def` record introduces `ROCDL_V16F32Type`, which later participates in generated MLIR code.
  **CN L266:** 该 TableGen `def` 记录引入了 `ROCDL_V16F32Type`，后续会参与生成的 MLIR 代码。
- **EN L267:** This TableGen `def` record introduces `ROCDL_V32F16Type`, which later participates in generated MLIR code.
  **CN L267:** 该 TableGen `def` 记录引入了 `ROCDL_V32F16Type`，后续会参与生成的 MLIR 代码。
- **EN L268:** This TableGen `def` record introduces `ROCDL_V32BF16Type`, which later participates in generated MLIR code.
  **CN L268:** 该 TableGen `def` 记录引入了 `ROCDL_V32BF16Type`，后续会参与生成的 MLIR 代码。
- **EN L269:** This TableGen `def` record introduces `ROCDL_V32F32Type`, which later participates in generated MLIR code.
  **CN L269:** 该 TableGen `def` 记录引入了 `ROCDL_V32F32Type`，后续会参与生成的 MLIR 代码。
- **EN L270:** Blank line used to separate nearby declarations and improve readability.
  **CN L270:** 该空行用于分隔相邻声明并提升可读性。
- **EN L271:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L271:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L272:** This comment states: “Wave-level primitives”, documenting the intent of the surrounding code.
  **CN L272:** 该注释写道：“Wave-level primitives”，用于说明周围代码的意图。
- **EN L273:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L273:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L274:** Blank line used to separate nearby declarations and improve readability.
  **CN L274:** 该空行用于分隔相邻声明并提升可读性。
- **EN L275:** This TableGen `class` record introduces `ROCDL_MbcntOp`, which later participates in generated MLIR code.
  **CN L275:** 该 TableGen `class` 记录引入了 `ROCDL_MbcntOp`，后续会参与生成的 MLIR 代码。
- **EN L276:** This line contributes implementation detail or declarative structure to the file.
  **CN L276:** 这一行为文件补充了实现细节或声明式结构。

### Lines 277-288 / 第 277-288 行

```tablegen
 277:     0, 0, /*requiresArgAndResultAttrs=*/1> {
 278:   dag args = (ins I32:$in0, I32:$in1);
 279:   let arguments = !con(args, baseArgs);
 280:   let results = (outs I32:$res);
 281:   let assemblyFormat = [{
 282:     $in0 `,` $in1  attr-dict `:` `(` type($in0) `,` type($in1) `)` `->` type($res)
 283:    }];
 284:   let description = [{
 285:     Masked bit count of threads below the current lane in a wavefront.
 286: 
 287:     `in0` is a 32-bit mask that is AND-ed with the relevant half of the
 288:     execution mask and the bits below the current lane; `in1` is added
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L277:** This line contributes implementation detail or declarative structure to the file.
  **CN L277:** 这一行为文件补充了实现细节或声明式结构。
- **EN L278:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L278:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L279:** This line contributes to the declaration or call of `con`.
  **CN L279:** 这一行为 `con` 的声明或调用提供内容。
- **EN L280:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L280:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L281:** This line contributes implementation detail or declarative structure to the file.
  **CN L281:** 这一行为文件补充了实现细节或声明式结构。
- **EN L282:** This line contributes to the declaration or call of `type`.
  **CN L282:** 这一行为 `type` 的声明或调用提供内容。
- **EN L283:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L283:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L284:** This line contributes implementation detail or declarative structure to the file.
  **CN L284:** 这一行为文件补充了实现细节或声明式结构。
- **EN L285:** This line contributes implementation detail or declarative structure to the file.
  **CN L285:** 这一行为文件补充了实现细节或声明式结构。
- **EN L286:** Blank line used to separate nearby declarations and improve readability.
  **CN L286:** 该空行用于分隔相邻声明并提升可读性。
- **EN L287:** This line contributes implementation detail or declarative structure to the file.
  **CN L287:** 这一行为文件补充了实现细节或声明式结构。
- **EN L288:** This line contributes implementation detail or declarative structure to the file.
  **CN L288:** 这一行为文件补充了实现细节或声明式结构。

### Lines 289-300 / 第 289-300 行

```tablegen
 289:     to the resulting popcount:
 290: 
 291:     - **lo**: `in1 + popcount(in0 & exec_lo & ((1 << min(lane_id, 32)) - 1))`
 292:     - **hi**: `in1 + popcount(in0 & exec_hi & ((1 << saturating_usub(lane_id, 32)) - 1))`
 293: 
 294:     To obtain a unique thread index within a wave64, chain the two ops
 295:     with `in0 = -1` (all bits set):
 296: 
 297:     Example:
 298:     ```mlir
 299:     %all_ones = arith.constant -1 : i32
 300:     %zero = arith.constant 0 : i32
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L289:** This line contributes implementation detail or declarative structure to the file.
  **CN L289:** 这一行为文件补充了实现细节或声明式结构。
- **EN L290:** Blank line used to separate nearby declarations and improve readability.
  **CN L290:** 该空行用于分隔相邻声明并提升可读性。
- **EN L291:** This line contributes to the declaration or call of `popcount`.
  **CN L291:** 这一行为 `popcount` 的声明或调用提供内容。
- **EN L292:** This line contributes to the declaration or call of `popcount`.
  **CN L292:** 这一行为 `popcount` 的声明或调用提供内容。
- **EN L293:** Blank line used to separate nearby declarations and improve readability.
  **CN L293:** 该空行用于分隔相邻声明并提升可读性。
- **EN L294:** This line contributes implementation detail or declarative structure to the file.
  **CN L294:** 这一行为文件补充了实现细节或声明式结构。
- **EN L295:** This line contributes implementation detail or declarative structure to the file.
  **CN L295:** 这一行为文件补充了实现细节或声明式结构。
- **EN L296:** Blank line used to separate nearby declarations and improve readability.
  **CN L296:** 该空行用于分隔相邻声明并提升可读性。
- **EN L297:** This line contributes implementation detail or declarative structure to the file.
  **CN L297:** 这一行为文件补充了实现细节或声明式结构。
- **EN L298:** This line contributes implementation detail or declarative structure to the file.
  **CN L298:** 这一行为文件补充了实现细节或声明式结构。
- **EN L299:** This line contributes implementation detail or declarative structure to the file.
  **CN L299:** 这一行为文件补充了实现细节或声明式结构。
- **EN L300:** This line contributes implementation detail or declarative structure to the file.
  **CN L300:** 这一行为文件补充了实现细节或声明式结构。

### Lines 301-312 / 第 301-312 行

```tablegen
 301: 
 302:     // Count active threads below this lane in the low 32 lanes.
 303:     %lo = rocdl.mbcnt.lo %all_ones, %zero : (i32, i32) -> i32
 304: 
 305:     // Add the count from the high 32 lanes to get the full lane index.
 306:     %hi = rocdl.mbcnt.hi %all_ones, %lo : (i32, i32) -> i32
 307:     ```
 308:   }];
 309: }
 310: 
 311: def ROCDL_MbcntLoOp : ROCDL_MbcntOp<"lo">;
 312: def ROCDL_MbcntHiOp : ROCDL_MbcntOp<"hi">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L301:** Blank line used to separate nearby declarations and improve readability.
  **CN L301:** 该空行用于分隔相邻声明并提升可读性。
- **EN L302:** This comment states: “Count active threads below this lane in the low 32 lanes.”, documenting the intent of the surrounding code.
  **CN L302:** 该注释写道：“Count active threads below this lane in the low 32 lanes.”，用于说明周围代码的意图。
- **EN L303:** This line contributes implementation detail or declarative structure to the file.
  **CN L303:** 这一行为文件补充了实现细节或声明式结构。
- **EN L304:** Blank line used to separate nearby declarations and improve readability.
  **CN L304:** 该空行用于分隔相邻声明并提升可读性。
- **EN L305:** This comment states: “Add the count from the high 32 lanes to get the full lane index.”, documenting the intent of the surrounding code.
  **CN L305:** 该注释写道：“Add the count from the high 32 lanes to get the full lane index.”，用于说明周围代码的意图。
- **EN L306:** This line contributes implementation detail or declarative structure to the file.
  **CN L306:** 这一行为文件补充了实现细节或声明式结构。
- **EN L307:** This line contributes implementation detail or declarative structure to the file.
  **CN L307:** 这一行为文件补充了实现细节或声明式结构。
- **EN L308:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L308:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L309:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L309:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L310:** Blank line used to separate nearby declarations and improve readability.
  **CN L310:** 该空行用于分隔相邻声明并提升可读性。
- **EN L311:** This TableGen `def` record introduces `ROCDL_MbcntLoOp`, which later participates in generated MLIR code.
  **CN L311:** 该 TableGen `def` 记录引入了 `ROCDL_MbcntLoOp`，后续会参与生成的 MLIR 代码。
- **EN L312:** This TableGen `def` record introduces `ROCDL_MbcntHiOp`, which later participates in generated MLIR code.
  **CN L312:** 该 TableGen `def` 记录引入了 `ROCDL_MbcntHiOp`，后续会参与生成的 MLIR 代码。

### Lines 313-324 / 第 313-324 行

```tablegen
 313: 
 314: def ROCDL_DsSwizzleOp : ROCDL_ConcreteNonMemIntrOp<"ds_swizzle", [], 1>,
 315:   Arguments<(ins I32:$src,
 316:                  I32:$offset)> {
 317:   let results = (outs I32:$res);
 318:   let assemblyFormat = [{
 319:     $src `,` $offset  attr-dict `:` `(` type($src) `,` type($offset) `)` `->` type($res)
 320:    }];
 321:   let description = [{
 322:     Perform a data-sharing swizzle operation within a wavefront.
 323: 
 324:     The `offset` operand encodes the *swizzle pattern* that will be placed in the
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L313:** Blank line used to separate nearby declarations and improve readability.
  **CN L313:** 该空行用于分隔相邻声明并提升可读性。
- **EN L314:** This TableGen `def` record introduces `ROCDL_DsSwizzleOp`, which later participates in generated MLIR code.
  **CN L314:** 该 TableGen `def` 记录引入了 `ROCDL_DsSwizzleOp`，后续会参与生成的 MLIR 代码。
- **EN L315:** This line contributes implementation detail or declarative structure to the file.
  **CN L315:** 这一行为文件补充了实现细节或声明式结构。
- **EN L316:** This line contributes implementation detail or declarative structure to the file.
  **CN L316:** 这一行为文件补充了实现细节或声明式结构。
- **EN L317:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L317:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L318:** This line contributes implementation detail or declarative structure to the file.
  **CN L318:** 这一行为文件补充了实现细节或声明式结构。
- **EN L319:** This line contributes to the declaration or call of `type`.
  **CN L319:** 这一行为 `type` 的声明或调用提供内容。
- **EN L320:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L320:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L321:** This line contributes implementation detail or declarative structure to the file.
  **CN L321:** 这一行为文件补充了实现细节或声明式结构。
- **EN L322:** This line contributes implementation detail or declarative structure to the file.
  **CN L322:** 这一行为文件补充了实现细节或声明式结构。
- **EN L323:** Blank line used to separate nearby declarations and improve readability.
  **CN L323:** 该空行用于分隔相邻声明并提升可读性。
- **EN L324:** This line contributes implementation detail or declarative structure to the file.
  **CN L324:** 这一行为文件补充了实现细节或声明式结构。

### Lines 325-336 / 第 325-336 行

```tablegen
 325:     instruction's `offset` field (i.e., the pattern used by `ds_swizzle_b32`).
 326:     See https://llvm.org/docs/AMDGPUModifierSyntax.html#swizzle-pattern for
 327:     how this 16-bit pattern is constructed.
 328: 
 329:     Example:
 330:     ```mlir
 331:     // Swizzle data within a wavefront.
 332:     %0 = rocdl.ds_swizzle %src, %offset : (i32, i32) -> i32
 333:     ```
 334:   }];
 335: }
 336: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L325:** This line contributes to the declaration or call of `field`.
  **CN L325:** 这一行为 `field` 的声明或调用提供内容。
- **EN L326:** This line contributes implementation detail or declarative structure to the file.
  **CN L326:** 这一行为文件补充了实现细节或声明式结构。
- **EN L327:** This line contributes implementation detail or declarative structure to the file.
  **CN L327:** 这一行为文件补充了实现细节或声明式结构。
- **EN L328:** Blank line used to separate nearby declarations and improve readability.
  **CN L328:** 该空行用于分隔相邻声明并提升可读性。
- **EN L329:** This line contributes implementation detail or declarative structure to the file.
  **CN L329:** 这一行为文件补充了实现细节或声明式结构。
- **EN L330:** This line contributes implementation detail or declarative structure to the file.
  **CN L330:** 这一行为文件补充了实现细节或声明式结构。
- **EN L331:** This comment states: “Swizzle data within a wavefront.”, documenting the intent of the surrounding code.
  **CN L331:** 该注释写道：“Swizzle data within a wavefront.”，用于说明周围代码的意图。
- **EN L332:** This line contributes implementation detail or declarative structure to the file.
  **CN L332:** 这一行为文件补充了实现细节或声明式结构。
- **EN L333:** This line contributes implementation detail or declarative structure to the file.
  **CN L333:** 这一行为文件补充了实现细节或声明式结构。
- **EN L334:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L334:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L335:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L335:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L336:** Blank line used to separate nearby declarations and improve readability.
  **CN L336:** 该空行用于分隔相邻声明并提升可读性。

### Lines 337-348 / 第 337-348 行

```tablegen
 337: def ROCDL_DsBpermuteOp : ROCDL_ConcreteNonMemIntrOp<"ds_bpermute", [], 1>,
 338:   Arguments<(ins I32:$index,
 339:                  I32:$src)> {
 340:   let results = (outs I32:$res);
 341:   let assemblyFormat = [{
 342:     $index `,` $src  attr-dict `:` `(` type($index) `,` type($src) `)` `->` type($res)
 343:    }];
 344:   let description = [{
 345:     Perform a backward permute (pull) operation across lanes using DS/LDS permute hardware.
 346: 
 347:     Each lane reads the value of `src` from the lane whose byte address is
 348:     given by `index` (i.e. lane id = `index / 4`).
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L337:** This TableGen `def` record introduces `ROCDL_DsBpermuteOp`, which later participates in generated MLIR code.
  **CN L337:** 该 TableGen `def` 记录引入了 `ROCDL_DsBpermuteOp`，后续会参与生成的 MLIR 代码。
- **EN L338:** This line contributes implementation detail or declarative structure to the file.
  **CN L338:** 这一行为文件补充了实现细节或声明式结构。
- **EN L339:** This line contributes implementation detail or declarative structure to the file.
  **CN L339:** 这一行为文件补充了实现细节或声明式结构。
- **EN L340:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L340:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L341:** This line contributes implementation detail or declarative structure to the file.
  **CN L341:** 这一行为文件补充了实现细节或声明式结构。
- **EN L342:** This line contributes to the declaration or call of `type`.
  **CN L342:** 这一行为 `type` 的声明或调用提供内容。
- **EN L343:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L343:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L344:** This line contributes implementation detail or declarative structure to the file.
  **CN L344:** 这一行为文件补充了实现细节或声明式结构。
- **EN L345:** This line contributes to the declaration or call of `permute`.
  **CN L345:** 这一行为 `permute` 的声明或调用提供内容。
- **EN L346:** Blank line used to separate nearby declarations and improve readability.
  **CN L346:** 该空行用于分隔相邻声明并提升可读性。
- **EN L347:** This line contributes implementation detail or declarative structure to the file.
  **CN L347:** 这一行为文件补充了实现细节或声明式结构。
- **EN L348:** This line contributes implementation detail or declarative structure to the file.
  **CN L348:** 这一行为文件补充了实现细节或声明式结构。

### Lines 349-360 / 第 349-360 行

```tablegen
 349: 
 350:     This is “backward” (pull) in contrast to `ds_permute_b32`, which is
 351:     “forward” (push/scatter).
 352: 
 353:     Example:
 354:     ```mlir
 355:     // Backward permute across lanes (pull from selected lane).
 356:     %0 = rocdl.ds_bpermute %index, %src : (i32, i32) -> i32
 357:     ```
 358:   }];
 359: }
 360: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L349:** Blank line used to separate nearby declarations and improve readability.
  **CN L349:** 该空行用于分隔相邻声明并提升可读性。
- **EN L350:** This line contributes implementation detail or declarative structure to the file.
  **CN L350:** 这一行为文件补充了实现细节或声明式结构。
- **EN L351:** This line contributes implementation detail or declarative structure to the file.
  **CN L351:** 这一行为文件补充了实现细节或声明式结构。
- **EN L352:** Blank line used to separate nearby declarations and improve readability.
  **CN L352:** 该空行用于分隔相邻声明并提升可读性。
- **EN L353:** This line contributes implementation detail or declarative structure to the file.
  **CN L353:** 这一行为文件补充了实现细节或声明式结构。
- **EN L354:** This line contributes implementation detail or declarative structure to the file.
  **CN L354:** 这一行为文件补充了实现细节或声明式结构。
- **EN L355:** This comment states: “Backward permute across lanes (pull from selected lane).”, documenting the intent of the surrounding code.
  **CN L355:** 该注释写道：“Backward permute across lanes (pull from selected lane).”，用于说明周围代码的意图。
- **EN L356:** This line contributes implementation detail or declarative structure to the file.
  **CN L356:** 这一行为文件补充了实现细节或声明式结构。
- **EN L357:** This line contributes implementation detail or declarative structure to the file.
  **CN L357:** 这一行为文件补充了实现细节或声明式结构。
- **EN L358:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L358:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L359:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L359:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L360:** Blank line used to separate nearby declarations and improve readability.
  **CN L360:** 该空行用于分隔相邻声明并提升可读性。

### Lines 361-372 / 第 361-372 行

```tablegen
 361: def ROCDL_BallotOp :
 362:   ROCDL_IntrOp<"ballot", [0], [], [], 1>,
 363:   Arguments<(ins I1:$pred)> {
 364:   let summary = "Vote across thread group";
 365: 
 366:   let description = [{
 367:       Ballot provides a bit mask containing the 1-bit predicate value from each lane.
 368:       The nth bit of the result contains the 1 bit contributed by the nth warp lane.
 369: 
 370:       Example:
 371:       ```mlir
 372:       // Ballot across thread group.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L361:** This TableGen `def` record introduces `ROCDL_BallotOp`, which later participates in generated MLIR code.
  **CN L361:** 该 TableGen `def` 记录引入了 `ROCDL_BallotOp`，后续会参与生成的 MLIR 代码。
- **EN L362:** This line contributes implementation detail or declarative structure to the file.
  **CN L362:** 这一行为文件补充了实现细节或声明式结构。
- **EN L363:** This line contributes implementation detail or declarative structure to the file.
  **CN L363:** 这一行为文件补充了实现细节或声明式结构。
- **EN L364:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L364:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L365:** Blank line used to separate nearby declarations and improve readability.
  **CN L365:** 该空行用于分隔相邻声明并提升可读性。
- **EN L366:** This line contributes implementation detail or declarative structure to the file.
  **CN L366:** 这一行为文件补充了实现细节或声明式结构。
- **EN L367:** This line contributes implementation detail or declarative structure to the file.
  **CN L367:** 这一行为文件补充了实现细节或声明式结构。
- **EN L368:** This line contributes implementation detail or declarative structure to the file.
  **CN L368:** 这一行为文件补充了实现细节或声明式结构。
- **EN L369:** Blank line used to separate nearby declarations and improve readability.
  **CN L369:** 该空行用于分隔相邻声明并提升可读性。
- **EN L370:** This line contributes implementation detail or declarative structure to the file.
  **CN L370:** 这一行为文件补充了实现细节或声明式结构。
- **EN L371:** This line contributes implementation detail or declarative structure to the file.
  **CN L371:** 这一行为文件补充了实现细节或声明式结构。
- **EN L372:** This comment states: “Ballot across thread group.”, documenting the intent of the surrounding code.
  **CN L372:** 该注释写道：“Ballot across thread group.”，用于说明周围代码的意图。

### Lines 373-384 / 第 373-384 行

```tablegen
 373:       %0 = rocdl.ballot %pred : i64
 374:       ```
 375:   }];
 376: 
 377:   let assemblyFormat = "$pred attr-dict `:` type($res)";
 378: }
 379: 
 380: def ROCDL_ReadfirstlaneOp : ROCDL_IntrOp<"readfirstlane", [], [0], [AllTypesMatch<["res", "src"]>], 1>,
 381:   Arguments<(ins LLVM_Type:$src)> {
 382:   let results = (outs LLVM_Type:$res);
 383:   let summary = "Get the value in first active lane.";
 384: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L373:** This line contributes implementation detail or declarative structure to the file.
  **CN L373:** 这一行为文件补充了实现细节或声明式结构。
- **EN L374:** This line contributes implementation detail or declarative structure to the file.
  **CN L374:** 这一行为文件补充了实现细节或声明式结构。
- **EN L375:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L375:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L376:** Blank line used to separate nearby declarations and improve readability.
  **CN L376:** 该空行用于分隔相邻声明并提升可读性。
- **EN L377:** This line contributes to the declaration or call of `type`.
  **CN L377:** 这一行为 `type` 的声明或调用提供内容。
- **EN L378:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L378:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L379:** Blank line used to separate nearby declarations and improve readability.
  **CN L379:** 该空行用于分隔相邻声明并提升可读性。
- **EN L380:** This TableGen `def` record introduces `ROCDL_ReadfirstlaneOp`, which later participates in generated MLIR code.
  **CN L380:** 该 TableGen `def` 记录引入了 `ROCDL_ReadfirstlaneOp`，后续会参与生成的 MLIR 代码。
- **EN L381:** This line contributes implementation detail or declarative structure to the file.
  **CN L381:** 这一行为文件补充了实现细节或声明式结构。
- **EN L382:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L382:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L383:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L383:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L384:** Blank line used to separate nearby declarations and improve readability.
  **CN L384:** 该空行用于分隔相邻声明并提升可读性。

### Lines 385-396 / 第 385-396 行

```tablegen
 385:   let description = [{
 386:     Returns the value in the lowest active lane of the input operand.
 387: 
 388:     Example:
 389:     ```mlir
 390:     // Scalar readfirstlane.
 391:     %0 = rocdl.readfirstlane %src0 : f32
 392: 
 393:     // Vector readfirstlane.
 394:     %1 = rocdl.readfirstlane %src1 : vector<2xf32>
 395:     ```
 396:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L385:** This line contributes implementation detail or declarative structure to the file.
  **CN L385:** 这一行为文件补充了实现细节或声明式结构。
- **EN L386:** This line contributes implementation detail or declarative structure to the file.
  **CN L386:** 这一行为文件补充了实现细节或声明式结构。
- **EN L387:** Blank line used to separate nearby declarations and improve readability.
  **CN L387:** 该空行用于分隔相邻声明并提升可读性。
- **EN L388:** This line contributes implementation detail or declarative structure to the file.
  **CN L388:** 这一行为文件补充了实现细节或声明式结构。
- **EN L389:** This line contributes implementation detail or declarative structure to the file.
  **CN L389:** 这一行为文件补充了实现细节或声明式结构。
- **EN L390:** This comment states: “Scalar readfirstlane.”, documenting the intent of the surrounding code.
  **CN L390:** 该注释写道：“Scalar readfirstlane.”，用于说明周围代码的意图。
- **EN L391:** This line contributes implementation detail or declarative structure to the file.
  **CN L391:** 这一行为文件补充了实现细节或声明式结构。
- **EN L392:** Blank line used to separate nearby declarations and improve readability.
  **CN L392:** 该空行用于分隔相邻声明并提升可读性。
- **EN L393:** This comment states: “Vector readfirstlane.”, documenting the intent of the surrounding code.
  **CN L393:** 该注释写道：“Vector readfirstlane.”，用于说明周围代码的意图。
- **EN L394:** This line contributes implementation detail or declarative structure to the file.
  **CN L394:** 这一行为文件补充了实现细节或声明式结构。
- **EN L395:** This line contributes implementation detail or declarative structure to the file.
  **CN L395:** 这一行为文件补充了实现细节或声明式结构。
- **EN L396:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L396:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 397-408 / 第 397-408 行

```tablegen
 397: 
 398:   let assemblyFormat = [{
 399:     $src attr-dict `:` type($res)
 400:   }];
 401: }
 402: 
 403: def ROCDL_ReadlaneOp : ROCDL_IntrOp<"readlane", [], [0], [AllTypesMatch<["res", "src0"]>], 1>,
 404:   Arguments<(ins LLVM_Type:$src0,
 405:                  I32:$src1)> {
 406:   let results = (outs LLVM_Type:$res);
 407:   let summary = "Get the value in the specific lane.";
 408: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L397:** Blank line used to separate nearby declarations and improve readability.
  **CN L397:** 该空行用于分隔相邻声明并提升可读性。
- **EN L398:** This line contributes implementation detail or declarative structure to the file.
  **CN L398:** 这一行为文件补充了实现细节或声明式结构。
- **EN L399:** This line contributes to the declaration or call of `type`.
  **CN L399:** 这一行为 `type` 的声明或调用提供内容。
- **EN L400:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L400:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L401:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L401:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L402:** Blank line used to separate nearby declarations and improve readability.
  **CN L402:** 该空行用于分隔相邻声明并提升可读性。
- **EN L403:** This TableGen `def` record introduces `ROCDL_ReadlaneOp`, which later participates in generated MLIR code.
  **CN L403:** 该 TableGen `def` 记录引入了 `ROCDL_ReadlaneOp`，后续会参与生成的 MLIR 代码。
- **EN L404:** This line contributes implementation detail or declarative structure to the file.
  **CN L404:** 这一行为文件补充了实现细节或声明式结构。
- **EN L405:** This line contributes implementation detail or declarative structure to the file.
  **CN L405:** 这一行为文件补充了实现细节或声明式结构。
- **EN L406:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L406:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L407:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L407:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L408:** Blank line used to separate nearby declarations and improve readability.
  **CN L408:** 该空行用于分隔相邻声明并提升可读性。

### Lines 409-420 / 第 409-420 行

```tablegen
 409:   let description = [{
 410:     Get the value in lane `src1` from input `src0`.
 411: 
 412:     Example:
 413:     ```mlir
 414:     // Scalar readlane.
 415:     %0 = rocdl.readlane %src0, %idx : (f32, i32) -> f32
 416: 
 417:     // Vector readlane.
 418:     %1 = rocdl.readlane %src1, %idx : (vector<2xf32>, i32) -> vector<2xf32>
 419:     ```
 420:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L409:** This line contributes implementation detail or declarative structure to the file.
  **CN L409:** 这一行为文件补充了实现细节或声明式结构。
- **EN L410:** This line contributes implementation detail or declarative structure to the file.
  **CN L410:** 这一行为文件补充了实现细节或声明式结构。
- **EN L411:** Blank line used to separate nearby declarations and improve readability.
  **CN L411:** 该空行用于分隔相邻声明并提升可读性。
- **EN L412:** This line contributes implementation detail or declarative structure to the file.
  **CN L412:** 这一行为文件补充了实现细节或声明式结构。
- **EN L413:** This line contributes implementation detail or declarative structure to the file.
  **CN L413:** 这一行为文件补充了实现细节或声明式结构。
- **EN L414:** This comment states: “Scalar readlane.”, documenting the intent of the surrounding code.
  **CN L414:** 该注释写道：“Scalar readlane.”，用于说明周围代码的意图。
- **EN L415:** This line contributes implementation detail or declarative structure to the file.
  **CN L415:** 这一行为文件补充了实现细节或声明式结构。
- **EN L416:** Blank line used to separate nearby declarations and improve readability.
  **CN L416:** 该空行用于分隔相邻声明并提升可读性。
- **EN L417:** This comment states: “Vector readlane.”, documenting the intent of the surrounding code.
  **CN L417:** 该注释写道：“Vector readlane.”，用于说明周围代码的意图。
- **EN L418:** This line contributes implementation detail or declarative structure to the file.
  **CN L418:** 这一行为文件补充了实现细节或声明式结构。
- **EN L419:** This line contributes implementation detail or declarative structure to the file.
  **CN L419:** 这一行为文件补充了实现细节或声明式结构。
- **EN L420:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L420:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 421-432 / 第 421-432 行

```tablegen
 421: 
 422:   let assemblyFormat = [{
 423:     $src0 `,` $src1  attr-dict `:` `(` type($src0) `,` type($src1) `)` `->` type($res)
 424:   }];
 425: }
 426: 
 427: //===----------------------------------------------------------------------===//
 428: // Thread, Block and Cluster index
 429: //===----------------------------------------------------------------------===//
 430: 
 431: def ROCDL_ThreadIdXOp : ROCDL_SpecialIdRegisterOp<"workitem.id.x">;
 432: def ROCDL_ThreadIdYOp : ROCDL_SpecialIdRegisterOp<"workitem.id.y">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L421:** Blank line used to separate nearby declarations and improve readability.
  **CN L421:** 该空行用于分隔相邻声明并提升可读性。
- **EN L422:** This line contributes implementation detail or declarative structure to the file.
  **CN L422:** 这一行为文件补充了实现细节或声明式结构。
- **EN L423:** This line contributes to the declaration or call of `type`.
  **CN L423:** 这一行为 `type` 的声明或调用提供内容。
- **EN L424:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L424:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L425:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L425:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L426:** Blank line used to separate nearby declarations and improve readability.
  **CN L426:** 该空行用于分隔相邻声明并提升可读性。
- **EN L427:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L427:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L428:** This comment states: “Thread, Block and Cluster index”, documenting the intent of the surrounding code.
  **CN L428:** 该注释写道：“Thread, Block and Cluster index”，用于说明周围代码的意图。
- **EN L429:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L429:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L430:** Blank line used to separate nearby declarations and improve readability.
  **CN L430:** 该空行用于分隔相邻声明并提升可读性。
- **EN L431:** This TableGen `def` record introduces `ROCDL_ThreadIdXOp`, which later participates in generated MLIR code.
  **CN L431:** 该 TableGen `def` 记录引入了 `ROCDL_ThreadIdXOp`，后续会参与生成的 MLIR 代码。
- **EN L432:** This TableGen `def` record introduces `ROCDL_ThreadIdYOp`, which later participates in generated MLIR code.
  **CN L432:** 该 TableGen `def` 记录引入了 `ROCDL_ThreadIdYOp`，后续会参与生成的 MLIR 代码。

### Lines 433-444 / 第 433-444 行

```tablegen
 433: def ROCDL_ThreadIdZOp : ROCDL_SpecialIdRegisterOp<"workitem.id.z">;
 434: 
 435: def ROCDL_BlockIdXOp : ROCDL_SpecialIdRegisterOp<"workgroup.id.x">;
 436: def ROCDL_BlockIdYOp : ROCDL_SpecialIdRegisterOp<"workgroup.id.y">;
 437: def ROCDL_BlockIdZOp : ROCDL_SpecialIdRegisterOp<"workgroup.id.z">;
 438: 
 439: def ROCDL_ClusterIdXOp : ROCDL_SpecialIdRegisterOp<"cluster.id.x">;
 440: def ROCDL_ClusterIdYOp : ROCDL_SpecialIdRegisterOp<"cluster.id.y">;
 441: def ROCDL_ClusterIdZOp : ROCDL_SpecialIdRegisterOp<"cluster.id.z">;
 442: 
 443: def ROCDL_ClusterWorkgroupIdXOp : ROCDL_SpecialIdRegisterOp<"cluster.workgroup.id.x">;
 444: def ROCDL_ClusterWorkgroupIdYOp : ROCDL_SpecialIdRegisterOp<"cluster.workgroup.id.y">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L433:** This TableGen `def` record introduces `ROCDL_ThreadIdZOp`, which later participates in generated MLIR code.
  **CN L433:** 该 TableGen `def` 记录引入了 `ROCDL_ThreadIdZOp`，后续会参与生成的 MLIR 代码。
- **EN L434:** Blank line used to separate nearby declarations and improve readability.
  **CN L434:** 该空行用于分隔相邻声明并提升可读性。
- **EN L435:** This TableGen `def` record introduces `ROCDL_BlockIdXOp`, which later participates in generated MLIR code.
  **CN L435:** 该 TableGen `def` 记录引入了 `ROCDL_BlockIdXOp`，后续会参与生成的 MLIR 代码。
- **EN L436:** This TableGen `def` record introduces `ROCDL_BlockIdYOp`, which later participates in generated MLIR code.
  **CN L436:** 该 TableGen `def` 记录引入了 `ROCDL_BlockIdYOp`，后续会参与生成的 MLIR 代码。
- **EN L437:** This TableGen `def` record introduces `ROCDL_BlockIdZOp`, which later participates in generated MLIR code.
  **CN L437:** 该 TableGen `def` 记录引入了 `ROCDL_BlockIdZOp`，后续会参与生成的 MLIR 代码。
- **EN L438:** Blank line used to separate nearby declarations and improve readability.
  **CN L438:** 该空行用于分隔相邻声明并提升可读性。
- **EN L439:** This TableGen `def` record introduces `ROCDL_ClusterIdXOp`, which later participates in generated MLIR code.
  **CN L439:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterIdXOp`，后续会参与生成的 MLIR 代码。
- **EN L440:** This TableGen `def` record introduces `ROCDL_ClusterIdYOp`, which later participates in generated MLIR code.
  **CN L440:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterIdYOp`，后续会参与生成的 MLIR 代码。
- **EN L441:** This TableGen `def` record introduces `ROCDL_ClusterIdZOp`, which later participates in generated MLIR code.
  **CN L441:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterIdZOp`，后续会参与生成的 MLIR 代码。
- **EN L442:** Blank line used to separate nearby declarations and improve readability.
  **CN L442:** 该空行用于分隔相邻声明并提升可读性。
- **EN L443:** This TableGen `def` record introduces `ROCDL_ClusterWorkgroupIdXOp`, which later participates in generated MLIR code.
  **CN L443:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterWorkgroupIdXOp`，后续会参与生成的 MLIR 代码。
- **EN L444:** This TableGen `def` record introduces `ROCDL_ClusterWorkgroupIdYOp`, which later participates in generated MLIR code.
  **CN L444:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterWorkgroupIdYOp`，后续会参与生成的 MLIR 代码。

### Lines 445-456 / 第 445-456 行

```tablegen
 445: def ROCDL_ClusterWorkgroupIdZOp : ROCDL_SpecialIdRegisterOp<"cluster.workgroup.id.z">;
 446: 
 447: def ROCDL_WaveId : ROCDL_SpecialIdRegisterOp<"wave.id">;
 448: def ROCDL_WavefrontSizeOp : ROCDL_SpecialIdRegisterOp<"wavefrontsize">;
 449: 
 450: //===----------------------------------------------------------------------===//
 451: // Synchronization primitives
 452: //===----------------------------------------------------------------------===//
 453: 
 454: // Emits the waintcnt instruction. The bitfield's semantics depend
 455: // on the target chipset
 456: def ROCDL_SWaitcntOp : ROCDL_ConcreteNonMemIntrOp<"s.waitcnt", [], 0, [0], ["bitfield"]>,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L445:** This TableGen `def` record introduces `ROCDL_ClusterWorkgroupIdZOp`, which later participates in generated MLIR code.
  **CN L445:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterWorkgroupIdZOp`，后续会参与生成的 MLIR 代码。
- **EN L446:** Blank line used to separate nearby declarations and improve readability.
  **CN L446:** 该空行用于分隔相邻声明并提升可读性。
- **EN L447:** This TableGen `def` record introduces `ROCDL_WaveId`, which later participates in generated MLIR code.
  **CN L447:** 该 TableGen `def` 记录引入了 `ROCDL_WaveId`，后续会参与生成的 MLIR 代码。
- **EN L448:** This TableGen `def` record introduces `ROCDL_WavefrontSizeOp`, which later participates in generated MLIR code.
  **CN L448:** 该 TableGen `def` 记录引入了 `ROCDL_WavefrontSizeOp`，后续会参与生成的 MLIR 代码。
- **EN L449:** Blank line used to separate nearby declarations and improve readability.
  **CN L449:** 该空行用于分隔相邻声明并提升可读性。
- **EN L450:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L450:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L451:** This comment states: “Synchronization primitives”, documenting the intent of the surrounding code.
  **CN L451:** 该注释写道：“Synchronization primitives”，用于说明周围代码的意图。
- **EN L452:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L452:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L453:** Blank line used to separate nearby declarations and improve readability.
  **CN L453:** 该空行用于分隔相邻声明并提升可读性。
- **EN L454:** This comment states: “Emits the waintcnt instruction. The bitfield's semantics depend”, documenting the intent of the surrounding code.
  **CN L454:** 该注释写道：“Emits the waintcnt instruction. The bitfield's semantics depend”，用于说明周围代码的意图。
- **EN L455:** This comment states: “on the target chipset”, documenting the intent of the surrounding code.
  **CN L455:** 该注释写道：“on the target chipset”，用于说明周围代码的意图。
- **EN L456:** This TableGen `def` record introduces `ROCDL_SWaitcntOp`, which later participates in generated MLIR code.
  **CN L456:** 该 TableGen `def` 记录引入了 `ROCDL_SWaitcntOp`，后续会参与生成的 MLIR 代码。

### Lines 457-468 / 第 457-468 行

```tablegen
 457:   Arguments<(ins I32Attr:$bitfield)> {
 458:   let assemblyFormat = "attr-dict $bitfield";
 459:   let description = [{
 460:     Wait for outstanding memory operations to complete, as specified by a
 461:     bitfield whose semantics depend on the target chipset.
 462: 
 463:     Example:
 464:     ```mlir
 465:     // Wait for all counters to reach zero.
 466:     rocdl.s.waitcnt 0
 467:     ```
 468:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L457:** This line contributes implementation detail or declarative structure to the file.
  **CN L457:** 这一行为文件补充了实现细节或声明式结构。
- **EN L458:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L458:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L459:** This line contributes implementation detail or declarative structure to the file.
  **CN L459:** 这一行为文件补充了实现细节或声明式结构。
- **EN L460:** This line contributes implementation detail or declarative structure to the file.
  **CN L460:** 这一行为文件补充了实现细节或声明式结构。
- **EN L461:** This line contributes implementation detail or declarative structure to the file.
  **CN L461:** 这一行为文件补充了实现细节或声明式结构。
- **EN L462:** Blank line used to separate nearby declarations and improve readability.
  **CN L462:** 该空行用于分隔相邻声明并提升可读性。
- **EN L463:** This line contributes implementation detail or declarative structure to the file.
  **CN L463:** 这一行为文件补充了实现细节或声明式结构。
- **EN L464:** This line contributes implementation detail or declarative structure to the file.
  **CN L464:** 这一行为文件补充了实现细节或声明式结构。
- **EN L465:** This comment states: “Wait for all counters to reach zero.”, documenting the intent of the surrounding code.
  **CN L465:** 该注释写道：“Wait for all counters to reach zero.”，用于说明周围代码的意图。
- **EN L466:** This line contributes implementation detail or declarative structure to the file.
  **CN L466:** 这一行为文件补充了实现细节或声明式结构。
- **EN L467:** This line contributes implementation detail or declarative structure to the file.
  **CN L467:** 这一行为文件补充了实现细节或声明式结构。
- **EN L468:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L468:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 469-480 / 第 469-480 行

```tablegen
 469: }
 470: 
 471: def ROCDL_SSleepOp : ROCDL_ConcreteNonMemIntrOp<"s.sleep", [], 0, [0], ["count"]>,
 472:   Arguments<(ins I32Attr:$count)> {
 473:   let assemblyFormat = "attr-dict $count";
 474:   let description = [{
 475:     Sleep for a number of clock cycles.
 476: 
 477:     Example:
 478:     ```mlir
 479:     // Sleep for a minimum duration.
 480:     rocdl.s.sleep 0
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L469:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L469:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L470:** Blank line used to separate nearby declarations and improve readability.
  **CN L470:** 该空行用于分隔相邻声明并提升可读性。
- **EN L471:** This TableGen `def` record introduces `ROCDL_SSleepOp`, which later participates in generated MLIR code.
  **CN L471:** 该 TableGen `def` 记录引入了 `ROCDL_SSleepOp`，后续会参与生成的 MLIR 代码。
- **EN L472:** This line contributes implementation detail or declarative structure to the file.
  **CN L472:** 这一行为文件补充了实现细节或声明式结构。
- **EN L473:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L473:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L474:** This line contributes implementation detail or declarative structure to the file.
  **CN L474:** 这一行为文件补充了实现细节或声明式结构。
- **EN L475:** This line contributes implementation detail or declarative structure to the file.
  **CN L475:** 这一行为文件补充了实现细节或声明式结构。
- **EN L476:** Blank line used to separate nearby declarations and improve readability.
  **CN L476:** 该空行用于分隔相邻声明并提升可读性。
- **EN L477:** This line contributes implementation detail or declarative structure to the file.
  **CN L477:** 这一行为文件补充了实现细节或声明式结构。
- **EN L478:** This line contributes implementation detail or declarative structure to the file.
  **CN L478:** 这一行为文件补充了实现细节或声明式结构。
- **EN L479:** This comment states: “Sleep for a minimum duration.”, documenting the intent of the surrounding code.
  **CN L479:** 该注释写道：“Sleep for a minimum duration.”，用于说明周围代码的意图。
- **EN L480:** This line contributes implementation detail or declarative structure to the file.
  **CN L480:** 这一行为文件补充了实现细节或声明式结构。

### Lines 481-492 / 第 481-492 行

```tablegen
 481:     ```
 482:   }];
 483: }
 484: 
 485: def ROCDL_SNopOp : ROCDL_ConcreteNonMemIntrOp<"s.nop", [], 0, [0], ["count"]>,
 486:   Arguments<(ins I16Attr:$count)> {
 487:   let assemblyFormat = "attr-dict $count";
 488:   let description = [{
 489:     Insert a number of NOP cycles.
 490: 
 491:     Example:
 492:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L481:** This line contributes implementation detail or declarative structure to the file.
  **CN L481:** 这一行为文件补充了实现细节或声明式结构。
- **EN L482:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L482:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L483:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L483:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L484:** Blank line used to separate nearby declarations and improve readability.
  **CN L484:** 该空行用于分隔相邻声明并提升可读性。
- **EN L485:** This TableGen `def` record introduces `ROCDL_SNopOp`, which later participates in generated MLIR code.
  **CN L485:** 该 TableGen `def` 记录引入了 `ROCDL_SNopOp`，后续会参与生成的 MLIR 代码。
- **EN L486:** This line contributes implementation detail or declarative structure to the file.
  **CN L486:** 这一行为文件补充了实现细节或声明式结构。
- **EN L487:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L487:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L488:** This line contributes implementation detail or declarative structure to the file.
  **CN L488:** 这一行为文件补充了实现细节或声明式结构。
- **EN L489:** This line contributes implementation detail or declarative structure to the file.
  **CN L489:** 这一行为文件补充了实现细节或声明式结构。
- **EN L490:** Blank line used to separate nearby declarations and improve readability.
  **CN L490:** 该空行用于分隔相邻声明并提升可读性。
- **EN L491:** This line contributes implementation detail or declarative structure to the file.
  **CN L491:** 这一行为文件补充了实现细节或声明式结构。
- **EN L492:** This line contributes implementation detail or declarative structure to the file.
  **CN L492:** 这一行为文件补充了实现细节或声明式结构。

### Lines 493-504 / 第 493-504 行

```tablegen
 493:     // Insert a no-op.
 494:     rocdl.s.nop 0
 495:     ```
 496:   }];
 497: }
 498: 
 499: def ROCDL_SBarrierOp : ROCDL_ConcreteNonMemIntrOp<"s.barrier", [], 0> {
 500:   let assemblyFormat = "attr-dict";
 501:   let description = [{
 502:     Insert a workgroup barrier without memory fences.
 503: 
 504:     Available on gfx9 and later but deprecated on gfx12+; see
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L493:** This comment states: “Insert a no-op.”, documenting the intent of the surrounding code.
  **CN L493:** 该注释写道：“Insert a no-op.”，用于说明周围代码的意图。
- **EN L494:** This line contributes implementation detail or declarative structure to the file.
  **CN L494:** 这一行为文件补充了实现细节或声明式结构。
- **EN L495:** This line contributes implementation detail or declarative structure to the file.
  **CN L495:** 这一行为文件补充了实现细节或声明式结构。
- **EN L496:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L496:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L497:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L497:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L498:** Blank line used to separate nearby declarations and improve readability.
  **CN L498:** 该空行用于分隔相邻声明并提升可读性。
- **EN L499:** This TableGen `def` record introduces `ROCDL_SBarrierOp`, which later participates in generated MLIR code.
  **CN L499:** 该 TableGen `def` 记录引入了 `ROCDL_SBarrierOp`，后续会参与生成的 MLIR 代码。
- **EN L500:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L500:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L501:** This line contributes implementation detail or declarative structure to the file.
  **CN L501:** 这一行为文件补充了实现细节或声明式结构。
- **EN L502:** This line contributes implementation detail or declarative structure to the file.
  **CN L502:** 这一行为文件补充了实现细节或声明式结构。
- **EN L503:** Blank line used to separate nearby declarations and improve readability.
  **CN L503:** 该空行用于分隔相邻声明并提升可读性。
- **EN L504:** This line contributes implementation detail or declarative structure to the file.
  **CN L504:** 这一行为文件补充了实现细节或声明式结构。

### Lines 505-516 / 第 505-516 行

```tablegen
 505:     `rocdl.s.barrier.signal` and `rocdl.s.barrier.wait` instead.
 506: 
 507:     Example:
 508:     ```mlir
 509:     // Synchronize threads within a workgroup.
 510:     rocdl.s.barrier
 511:     ```
 512:   }];
 513: }
 514: 
 515: def ROCDL_BarrierOp : ROCDL_Op<"barrier"> {
 516:   string llvmBuilder = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L505:** This line contributes implementation detail or declarative structure to the file.
  **CN L505:** 这一行为文件补充了实现细节或声明式结构。
- **EN L506:** Blank line used to separate nearby declarations and improve readability.
  **CN L506:** 该空行用于分隔相邻声明并提升可读性。
- **EN L507:** This line contributes implementation detail or declarative structure to the file.
  **CN L507:** 这一行为文件补充了实现细节或声明式结构。
- **EN L508:** This line contributes implementation detail or declarative structure to the file.
  **CN L508:** 这一行为文件补充了实现细节或声明式结构。
- **EN L509:** This comment states: “Synchronize threads within a workgroup.”, documenting the intent of the surrounding code.
  **CN L509:** 该注释写道：“Synchronize threads within a workgroup.”，用于说明周围代码的意图。
- **EN L510:** This line contributes implementation detail or declarative structure to the file.
  **CN L510:** 这一行为文件补充了实现细节或声明式结构。
- **EN L511:** This line contributes implementation detail or declarative structure to the file.
  **CN L511:** 这一行为文件补充了实现细节或声明式结构。
- **EN L512:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L512:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L513:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L513:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L514:** Blank line used to separate nearby declarations and improve readability.
  **CN L514:** 该空行用于分隔相邻声明并提升可读性。
- **EN L515:** This TableGen `def` record introduces `ROCDL_BarrierOp`, which later participates in generated MLIR code.
  **CN L515:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierOp`，后续会参与生成的 MLIR 代码。
- **EN L516:** This line contributes implementation detail or declarative structure to the file.
  **CN L516:** 这一行为文件补充了实现细节或声明式结构。

### Lines 517-528 / 第 517-528 行

```tablegen
 517:     llvm::LLVMContext &llvmContext = builder.getContext();
 518:     builder.CreateFence(llvm::AtomicOrdering::Release,
 519:                         llvmContext.getOrInsertSyncScopeID("workgroup"));
 520:     createIntrinsicCall(builder, llvm::Intrinsic::amdgcn_s_barrier);
 521:     builder.CreateFence(llvm::AtomicOrdering::Acquire,
 522:                         llvmContext.getOrInsertSyncScopeID("workgroup"));
 523:   }];
 524:   let description = [{
 525:     An operation with the same expansion as HIP's __synchthreads();
 526: 
 527:     **DEPRECATION NOTICE**: Use `gpu.barrier`, which will expand to these
 528:     operations, instead.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L517:** This line contributes to the declaration or call of `getContext`.
  **CN L517:** 这一行为 `getContext` 的声明或调用提供内容。
- **EN L518:** This line contributes to the declaration or call of `CreateFence`.
  **CN L518:** 这一行为 `CreateFence` 的声明或调用提供内容。
- **EN L519:** This line contributes to the declaration or call of `getOrInsertSyncScopeID`.
  **CN L519:** 这一行为 `getOrInsertSyncScopeID` 的声明或调用提供内容。
- **EN L520:** This line contributes to the declaration or call of `createIntrinsicCall`.
  **CN L520:** 这一行为 `createIntrinsicCall` 的声明或调用提供内容。
- **EN L521:** This line contributes to the declaration or call of `CreateFence`.
  **CN L521:** 这一行为 `CreateFence` 的声明或调用提供内容。
- **EN L522:** This line contributes to the declaration or call of `getOrInsertSyncScopeID`.
  **CN L522:** 这一行为 `getOrInsertSyncScopeID` 的声明或调用提供内容。
- **EN L523:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L523:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L524:** This line contributes implementation detail or declarative structure to the file.
  **CN L524:** 这一行为文件补充了实现细节或声明式结构。
- **EN L525:** This line contributes to the declaration or call of `__synchthreads`.
  **CN L525:** 这一行为 `__synchthreads` 的声明或调用提供内容。
- **EN L526:** Blank line used to separate nearby declarations and improve readability.
  **CN L526:** 该空行用于分隔相邻声明并提升可读性。
- **EN L527:** This comment states: “*DEPRECATION NOTICE**: Use `gpu.barrier`, which will expand to these”, documenting the intent of the surrounding code.
  **CN L527:** 该注释写道：“*DEPRECATION NOTICE**: Use `gpu.barrier`, which will expand to these”，用于说明周围代码的意图。
- **EN L528:** This line contributes implementation detail or declarative structure to the file.
  **CN L528:** 这一行为文件补充了实现细节或声明式结构。

### Lines 529-540 / 第 529-540 行

```tablegen
 529: 
 530:     Example:
 531:     ```mlir
 532:     // Workgroup barrier with acquire/release fences.
 533:     rocdl.barrier
 534:     ```
 535:   }];
 536:   let assemblyFormat = "attr-dict";
 537: }
 538: 
 539: def ROCDL_WaveBarrierOp : ROCDL_ConcreteNonMemIntrOp<"wave.barrier", [], 0> {
 540:   let assemblyFormat = "attr-dict";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L529:** Blank line used to separate nearby declarations and improve readability.
  **CN L529:** 该空行用于分隔相邻声明并提升可读性。
- **EN L530:** This line contributes implementation detail or declarative structure to the file.
  **CN L530:** 这一行为文件补充了实现细节或声明式结构。
- **EN L531:** This line contributes implementation detail or declarative structure to the file.
  **CN L531:** 这一行为文件补充了实现细节或声明式结构。
- **EN L532:** This comment states: “Workgroup barrier with acquire/release fences.”, documenting the intent of the surrounding code.
  **CN L532:** 该注释写道：“Workgroup barrier with acquire/release fences.”，用于说明周围代码的意图。
- **EN L533:** This line contributes implementation detail or declarative structure to the file.
  **CN L533:** 这一行为文件补充了实现细节或声明式结构。
- **EN L534:** This line contributes implementation detail or declarative structure to the file.
  **CN L534:** 这一行为文件补充了实现细节或声明式结构。
- **EN L535:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L535:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L536:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L536:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L537:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L537:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L538:** Blank line used to separate nearby declarations and improve readability.
  **CN L538:** 该空行用于分隔相邻声明并提升可读性。
- **EN L539:** This TableGen `def` record introduces `ROCDL_WaveBarrierOp`, which later participates in generated MLIR code.
  **CN L539:** 该 TableGen `def` 记录引入了 `ROCDL_WaveBarrierOp`，后续会参与生成的 MLIR 代码。
- **EN L540:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L540:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 541-552 / 第 541-552 行

```tablegen
 541:   let description = [{
 542:     Insert a wave-level (subgroup) barrier. Synchronizes lanes within a
 543:     single wave/wavefront without any memory ordering guarantees.
 544: 
 545:     Example:
 546:     ```mlir
 547:     rocdl.wave.barrier
 548:     ```
 549:   }];
 550: }
 551: 
 552: def ROCDLGlobalBuffer : LLVM_PointerInAddressSpace<1>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L541:** This line contributes implementation detail or declarative structure to the file.
  **CN L541:** 这一行为文件补充了实现细节或声明式结构。
- **EN L542:** This line contributes to the declaration or call of `level`.
  **CN L542:** 这一行为 `level` 的声明或调用提供内容。
- **EN L543:** This line contributes implementation detail or declarative structure to the file.
  **CN L543:** 这一行为文件补充了实现细节或声明式结构。
- **EN L544:** Blank line used to separate nearby declarations and improve readability.
  **CN L544:** 该空行用于分隔相邻声明并提升可读性。
- **EN L545:** This line contributes implementation detail or declarative structure to the file.
  **CN L545:** 这一行为文件补充了实现细节或声明式结构。
- **EN L546:** This line contributes implementation detail or declarative structure to the file.
  **CN L546:** 这一行为文件补充了实现细节或声明式结构。
- **EN L547:** This line contributes implementation detail or declarative structure to the file.
  **CN L547:** 这一行为文件补充了实现细节或声明式结构。
- **EN L548:** This line contributes implementation detail or declarative structure to the file.
  **CN L548:** 这一行为文件补充了实现细节或声明式结构。
- **EN L549:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L549:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L550:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L550:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L551:** Blank line used to separate nearby declarations and improve readability.
  **CN L551:** 该空行用于分隔相邻声明并提升可读性。
- **EN L552:** This TableGen `def` record introduces `ROCDLGlobalBuffer`, which later participates in generated MLIR code.
  **CN L552:** 该 TableGen `def` 记录引入了 `ROCDLGlobalBuffer`，后续会参与生成的 MLIR 代码。

### Lines 553-564 / 第 553-564 行

```tablegen
 553: def ROCDLBufferLDS : LLVM_PointerInAddressSpace<3>;
 554: 
 555: def ROCDL_BarrierInitOp : ROCDL_IntrOp<"s.barrier.init", [], [], [], 0, 0, 0, 0, [1], ["memberCnt"]>,
 556:   Arguments<(ins Arg<ROCDLBufferLDS, "", []>:$ptr, I32Attr:$memberCnt)> {
 557:   let description = [{
 558:     Available on gfx1250+.
 559: 
 560:     Example:
 561:     ```mlir
 562:     // Initialize a named barrier with member count.
 563:     rocdl.s.barrier.init %ptr member_cnt = 1 : !llvm.ptr<3>
 564:     ```
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L553:** This TableGen `def` record introduces `ROCDLBufferLDS`, which later participates in generated MLIR code.
  **CN L553:** 该 TableGen `def` 记录引入了 `ROCDLBufferLDS`，后续会参与生成的 MLIR 代码。
- **EN L554:** Blank line used to separate nearby declarations and improve readability.
  **CN L554:** 该空行用于分隔相邻声明并提升可读性。
- **EN L555:** This TableGen `def` record introduces `ROCDL_BarrierInitOp`, which later participates in generated MLIR code.
  **CN L555:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierInitOp`，后续会参与生成的 MLIR 代码。
- **EN L556:** This line contributes implementation detail or declarative structure to the file.
  **CN L556:** 这一行为文件补充了实现细节或声明式结构。
- **EN L557:** This line contributes implementation detail or declarative structure to the file.
  **CN L557:** 这一行为文件补充了实现细节或声明式结构。
- **EN L558:** This line contributes implementation detail or declarative structure to the file.
  **CN L558:** 这一行为文件补充了实现细节或声明式结构。
- **EN L559:** Blank line used to separate nearby declarations and improve readability.
  **CN L559:** 该空行用于分隔相邻声明并提升可读性。
- **EN L560:** This line contributes implementation detail or declarative structure to the file.
  **CN L560:** 这一行为文件补充了实现细节或声明式结构。
- **EN L561:** This line contributes implementation detail or declarative structure to the file.
  **CN L561:** 这一行为文件补充了实现细节或声明式结构。
- **EN L562:** This comment states: “Initialize a named barrier with member count.”, documenting the intent of the surrounding code.
  **CN L562:** 该注释写道：“Initialize a named barrier with member count.”，用于说明周围代码的意图。
- **EN L563:** This line contributes implementation detail or declarative structure to the file.
  **CN L563:** 这一行为文件补充了实现细节或声明式结构。
- **EN L564:** This line contributes implementation detail or declarative structure to the file.
  **CN L564:** 这一行为文件补充了实现细节或声明式结构。

### Lines 565-576 / 第 565-576 行

```tablegen
 565:   }];
 566:   let results = (outs);
 567:   let assemblyFormat = "$ptr `member_cnt` `=` $memberCnt attr-dict `:` qualified(type($ptr))";
 568: }
 569: 
 570: def ROCDL_BarrierSignalOp : ROCDL_ConcreteNonMemIntrOp<"s.barrier.signal", [], 0, [0], ["id"]>,
 571:   Arguments<(ins I32Attr:$id)> {
 572:   let results = (outs);
 573:   let assemblyFormat = "`id` `=` $id attr-dict";
 574:   let description = [{
 575:     Signal a barrier by id. Available on gfx1250+.
 576: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L565:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L565:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L566:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L566:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L567:** This line contributes to the declaration or call of `qualified`.
  **CN L567:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L568:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L568:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L569:** Blank line used to separate nearby declarations and improve readability.
  **CN L569:** 该空行用于分隔相邻声明并提升可读性。
- **EN L570:** This TableGen `def` record introduces `ROCDL_BarrierSignalOp`, which later participates in generated MLIR code.
  **CN L570:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierSignalOp`，后续会参与生成的 MLIR 代码。
- **EN L571:** This line contributes implementation detail or declarative structure to the file.
  **CN L571:** 这一行为文件补充了实现细节或声明式结构。
- **EN L572:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L572:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L573:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L573:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L574:** This line contributes implementation detail or declarative structure to the file.
  **CN L574:** 这一行为文件补充了实现细节或声明式结构。
- **EN L575:** This line contributes implementation detail or declarative structure to the file.
  **CN L575:** 这一行为文件补充了实现细节或声明式结构。
- **EN L576:** Blank line used to separate nearby declarations and improve readability.
  **CN L576:** 该空行用于分隔相邻声明并提升可读性。

### Lines 577-588 / 第 577-588 行

```tablegen
 577:     Example:
 578:     ```mlir
 579:     // Signal barrier with id -1 (all barriers).
 580:     rocdl.s.barrier.signal id = -1
 581:     ```
 582:   }];
 583: }
 584: 
 585: def ROCDL_BarrierSignalVarOp : ROCDL_IntrOp<"s.barrier.signal.var", [], [], [], 0, 0, 0, 0, [1], ["memberCnt"]>,
 586:   Arguments<(ins Arg<ROCDLBufferLDS, "", []>:$ptr, I32Attr:$memberCnt)> {
 587:   let description = [{
 588:     Available on gfx1250+.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L577:** This line contributes implementation detail or declarative structure to the file.
  **CN L577:** 这一行为文件补充了实现细节或声明式结构。
- **EN L578:** This line contributes implementation detail or declarative structure to the file.
  **CN L578:** 这一行为文件补充了实现细节或声明式结构。
- **EN L579:** This comment states: “Signal barrier with id -1 (all barriers).”, documenting the intent of the surrounding code.
  **CN L579:** 该注释写道：“Signal barrier with id -1 (all barriers).”，用于说明周围代码的意图。
- **EN L580:** This line contributes implementation detail or declarative structure to the file.
  **CN L580:** 这一行为文件补充了实现细节或声明式结构。
- **EN L581:** This line contributes implementation detail or declarative structure to the file.
  **CN L581:** 这一行为文件补充了实现细节或声明式结构。
- **EN L582:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L582:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L583:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L583:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L584:** Blank line used to separate nearby declarations and improve readability.
  **CN L584:** 该空行用于分隔相邻声明并提升可读性。
- **EN L585:** This TableGen `def` record introduces `ROCDL_BarrierSignalVarOp`, which later participates in generated MLIR code.
  **CN L585:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierSignalVarOp`，后续会参与生成的 MLIR 代码。
- **EN L586:** This line contributes implementation detail or declarative structure to the file.
  **CN L586:** 这一行为文件补充了实现细节或声明式结构。
- **EN L587:** This line contributes implementation detail or declarative structure to the file.
  **CN L587:** 这一行为文件补充了实现细节或声明式结构。
- **EN L588:** This line contributes implementation detail or declarative structure to the file.
  **CN L588:** 这一行为文件补充了实现细节或声明式结构。

### Lines 589-600 / 第 589-600 行

```tablegen
 589: 
 590:     If `memberCnt` is 0, the member count is retained from a previous initialization.
 591: 
 592:     Example:
 593:     ```mlir
 594:     // Signal a named barrier with variable ID.
 595:     rocdl.s.barrier.signal.var %ptr member_cnt = 1 : !llvm.ptr<3>
 596:     ```
 597:   }];
 598:   let results = (outs);
 599:   let assemblyFormat = "$ptr `member_cnt` `=` $memberCnt attr-dict `:` qualified(type($ptr))";
 600: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L589:** Blank line used to separate nearby declarations and improve readability.
  **CN L589:** 该空行用于分隔相邻声明并提升可读性。
- **EN L590:** This line contributes implementation detail or declarative structure to the file.
  **CN L590:** 这一行为文件补充了实现细节或声明式结构。
- **EN L591:** Blank line used to separate nearby declarations and improve readability.
  **CN L591:** 该空行用于分隔相邻声明并提升可读性。
- **EN L592:** This line contributes implementation detail or declarative structure to the file.
  **CN L592:** 这一行为文件补充了实现细节或声明式结构。
- **EN L593:** This line contributes implementation detail or declarative structure to the file.
  **CN L593:** 这一行为文件补充了实现细节或声明式结构。
- **EN L594:** This comment states: “Signal a named barrier with variable ID.”, documenting the intent of the surrounding code.
  **CN L594:** 该注释写道：“Signal a named barrier with variable ID.”，用于说明周围代码的意图。
- **EN L595:** This line contributes implementation detail or declarative structure to the file.
  **CN L595:** 这一行为文件补充了实现细节或声明式结构。
- **EN L596:** This line contributes implementation detail or declarative structure to the file.
  **CN L596:** 这一行为文件补充了实现细节或声明式结构。
- **EN L597:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L597:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L598:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L598:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L599:** This line contributes to the declaration or call of `qualified`.
  **CN L599:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L600:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L600:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 601-612 / 第 601-612 行

```tablegen
 601: 
 602: def ROCDL_BarrierJoinOp : ROCDL_IntrOp<"s.barrier.join", [], [], [], 0>,
 603:   Arguments<(ins Arg<ROCDLBufferLDS, "", []>:$ptr)> {
 604:   let description = [{
 605:     Available on gfx1250+.
 606: 
 607:     Example:
 608:     ```mlir
 609:     // Join a named barrier.
 610:     rocdl.s.barrier.join %ptr : !llvm.ptr<3>
 611:     ```
 612:   }];
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L601:** Blank line used to separate nearby declarations and improve readability.
  **CN L601:** 该空行用于分隔相邻声明并提升可读性。
- **EN L602:** This TableGen `def` record introduces `ROCDL_BarrierJoinOp`, which later participates in generated MLIR code.
  **CN L602:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierJoinOp`，后续会参与生成的 MLIR 代码。
- **EN L603:** This line contributes implementation detail or declarative structure to the file.
  **CN L603:** 这一行为文件补充了实现细节或声明式结构。
- **EN L604:** This line contributes implementation detail or declarative structure to the file.
  **CN L604:** 这一行为文件补充了实现细节或声明式结构。
- **EN L605:** This line contributes implementation detail or declarative structure to the file.
  **CN L605:** 这一行为文件补充了实现细节或声明式结构。
- **EN L606:** Blank line used to separate nearby declarations and improve readability.
  **CN L606:** 该空行用于分隔相邻声明并提升可读性。
- **EN L607:** This line contributes implementation detail or declarative structure to the file.
  **CN L607:** 这一行为文件补充了实现细节或声明式结构。
- **EN L608:** This line contributes implementation detail or declarative structure to the file.
  **CN L608:** 这一行为文件补充了实现细节或声明式结构。
- **EN L609:** This comment states: “Join a named barrier.”, documenting the intent of the surrounding code.
  **CN L609:** 该注释写道：“Join a named barrier.”，用于说明周围代码的意图。
- **EN L610:** This line contributes implementation detail or declarative structure to the file.
  **CN L610:** 这一行为文件补充了实现细节或声明式结构。
- **EN L611:** This line contributes implementation detail or declarative structure to the file.
  **CN L611:** 这一行为文件补充了实现细节或声明式结构。
- **EN L612:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L612:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 613-624 / 第 613-624 行

```tablegen
 613:   let results = (outs);
 614:   let assemblyFormat = "$ptr attr-dict `:` qualified(type($ptr))";
 615: }
 616: 
 617: def ROCDL_BarrierLeaveOp : ROCDL_ConcreteNonMemIntrOp<"s.barrier.leave", [], 0, [0], ["id"]>,
 618:   Arguments<(ins I16Attr:$id)> {
 619:   let description = [{
 620:     Available on gfx1250+.
 621: 
 622:     Example:
 623:     ```mlir
 624:     // Leave a named barrier by id.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L613:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L613:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L614:** This line contributes to the declaration or call of `qualified`.
  **CN L614:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L615:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L615:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L616:** Blank line used to separate nearby declarations and improve readability.
  **CN L616:** 该空行用于分隔相邻声明并提升可读性。
- **EN L617:** This TableGen `def` record introduces `ROCDL_BarrierLeaveOp`, which later participates in generated MLIR code.
  **CN L617:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierLeaveOp`，后续会参与生成的 MLIR 代码。
- **EN L618:** This line contributes implementation detail or declarative structure to the file.
  **CN L618:** 这一行为文件补充了实现细节或声明式结构。
- **EN L619:** This line contributes implementation detail or declarative structure to the file.
  **CN L619:** 这一行为文件补充了实现细节或声明式结构。
- **EN L620:** This line contributes implementation detail or declarative structure to the file.
  **CN L620:** 这一行为文件补充了实现细节或声明式结构。
- **EN L621:** Blank line used to separate nearby declarations and improve readability.
  **CN L621:** 该空行用于分隔相邻声明并提升可读性。
- **EN L622:** This line contributes implementation detail or declarative structure to the file.
  **CN L622:** 这一行为文件补充了实现细节或声明式结构。
- **EN L623:** This line contributes implementation detail or declarative structure to the file.
  **CN L623:** 这一行为文件补充了实现细节或声明式结构。
- **EN L624:** This comment states: “Leave a named barrier by id.”, documenting the intent of the surrounding code.
  **CN L624:** 该注释写道：“Leave a named barrier by id.”，用于说明周围代码的意图。

### Lines 625-636 / 第 625-636 行

```tablegen
 625:     rocdl.s.barrier.leave id = 1
 626:     ```
 627:   }];
 628:   let results = (outs);
 629:   let assemblyFormat = "`id` `=` $id attr-dict";
 630: }
 631: 
 632: def ROCDL_BarrierWaitOp : ROCDL_ConcreteNonMemIntrOp<"s.barrier.wait", [], 0, [0], ["id"]>,
 633:   Arguments<(ins I16Attr:$id)> {
 634:   let results = (outs);
 635:   let assemblyFormat = "`id` `=` $id attr-dict";
 636:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L625:** This line contributes implementation detail or declarative structure to the file.
  **CN L625:** 这一行为文件补充了实现细节或声明式结构。
- **EN L626:** This line contributes implementation detail or declarative structure to the file.
  **CN L626:** 这一行为文件补充了实现细节或声明式结构。
- **EN L627:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L627:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L628:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L628:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L629:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L629:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L630:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L630:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L631:** Blank line used to separate nearby declarations and improve readability.
  **CN L631:** 该空行用于分隔相邻声明并提升可读性。
- **EN L632:** This TableGen `def` record introduces `ROCDL_BarrierWaitOp`, which later participates in generated MLIR code.
  **CN L632:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierWaitOp`，后续会参与生成的 MLIR 代码。
- **EN L633:** This line contributes implementation detail or declarative structure to the file.
  **CN L633:** 这一行为文件补充了实现细节或声明式结构。
- **EN L634:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L634:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L635:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L635:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L636:** This line contributes implementation detail or declarative structure to the file.
  **CN L636:** 这一行为文件补充了实现细节或声明式结构。

### Lines 637-648 / 第 637-648 行

```tablegen
 637:     Wait on a barrier by id. Available on gfx1200+.
 638: 
 639:     Example:
 640:     ```mlir
 641:     // Wait on barrier with id -1 (all barriers).
 642:     rocdl.s.barrier.wait id = -1
 643:     ```
 644:   }];
 645: }
 646: 
 647: def ROCDL_BarrierSignalIsfirstOp : ROCDL_ConcreteNonMemIntrOp<"s.barrier.signal.isfirst", [], 1, [0], ["id"]>,
 648:   Arguments<(ins I32Attr:$id)> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L637:** This line contributes implementation detail or declarative structure to the file.
  **CN L637:** 这一行为文件补充了实现细节或声明式结构。
- **EN L638:** Blank line used to separate nearby declarations and improve readability.
  **CN L638:** 该空行用于分隔相邻声明并提升可读性。
- **EN L639:** This line contributes implementation detail or declarative structure to the file.
  **CN L639:** 这一行为文件补充了实现细节或声明式结构。
- **EN L640:** This line contributes implementation detail or declarative structure to the file.
  **CN L640:** 这一行为文件补充了实现细节或声明式结构。
- **EN L641:** This comment states: “Wait on barrier with id -1 (all barriers).”, documenting the intent of the surrounding code.
  **CN L641:** 该注释写道：“Wait on barrier with id -1 (all barriers).”，用于说明周围代码的意图。
- **EN L642:** This line contributes implementation detail or declarative structure to the file.
  **CN L642:** 这一行为文件补充了实现细节或声明式结构。
- **EN L643:** This line contributes implementation detail or declarative structure to the file.
  **CN L643:** 这一行为文件补充了实现细节或声明式结构。
- **EN L644:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L644:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L645:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L645:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L646:** Blank line used to separate nearby declarations and improve readability.
  **CN L646:** 该空行用于分隔相邻声明并提升可读性。
- **EN L647:** This TableGen `def` record introduces `ROCDL_BarrierSignalIsfirstOp`, which later participates in generated MLIR code.
  **CN L647:** 该 TableGen `def` 记录引入了 `ROCDL_BarrierSignalIsfirstOp`，后续会参与生成的 MLIR 代码。
- **EN L648:** This line contributes implementation detail or declarative structure to the file.
  **CN L648:** 这一行为文件补充了实现细节或声明式结构。

### Lines 649-660 / 第 649-660 行

```tablegen
 649:   let description = [{
 650:     Available on gfx1200+.
 651: 
 652:     Example:
 653:     ```mlir
 654:     // Signal barrier and check if this wave is first to arrive.
 655:     %0 = rocdl.s.barrier.signal.isfirst id = 1 -> i1
 656:     ```
 657:   }];
 658:   let results = (outs I1:$res);
 659:   let assemblyFormat = "`id` `=` $id attr-dict `->` type($res)";
 660: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L649:** This line contributes implementation detail or declarative structure to the file.
  **CN L649:** 这一行为文件补充了实现细节或声明式结构。
- **EN L650:** This line contributes implementation detail or declarative structure to the file.
  **CN L650:** 这一行为文件补充了实现细节或声明式结构。
- **EN L651:** Blank line used to separate nearby declarations and improve readability.
  **CN L651:** 该空行用于分隔相邻声明并提升可读性。
- **EN L652:** This line contributes implementation detail or declarative structure to the file.
  **CN L652:** 这一行为文件补充了实现细节或声明式结构。
- **EN L653:** This line contributes implementation detail or declarative structure to the file.
  **CN L653:** 这一行为文件补充了实现细节或声明式结构。
- **EN L654:** This comment states: “Signal barrier and check if this wave is first to arrive.”, documenting the intent of the surrounding code.
  **CN L654:** 该注释写道：“Signal barrier and check if this wave is first to arrive.”，用于说明周围代码的意图。
- **EN L655:** This line contributes implementation detail or declarative structure to the file.
  **CN L655:** 这一行为文件补充了实现细节或声明式结构。
- **EN L656:** This line contributes implementation detail or declarative structure to the file.
  **CN L656:** 这一行为文件补充了实现细节或声明式结构。
- **EN L657:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L657:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L658:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L658:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L659:** This line contributes to the declaration or call of `type`.
  **CN L659:** 这一行为 `type` 的声明或调用提供内容。
- **EN L660:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L660:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 661-672 / 第 661-672 行

```tablegen
 661: 
 662: def ROCDL_GetBarrierStateOp : ROCDL_ConcreteNonMemIntrOp<"s.get.barrier.state", [], 1, [0], ["id"]>,
 663:   Arguments<(ins I32Attr:$id)> {
 664:   let description = [{
 665:     Available on gfx1200+.
 666: 
 667:     Example:
 668:     ```mlir
 669:     // Query barrier state by id.
 670:     %0 = rocdl.s.get.barrier.state id = 1 -> i32
 671:     ```
 672:   }];
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L661:** Blank line used to separate nearby declarations and improve readability.
  **CN L661:** 该空行用于分隔相邻声明并提升可读性。
- **EN L662:** This TableGen `def` record introduces `ROCDL_GetBarrierStateOp`, which later participates in generated MLIR code.
  **CN L662:** 该 TableGen `def` 记录引入了 `ROCDL_GetBarrierStateOp`，后续会参与生成的 MLIR 代码。
- **EN L663:** This line contributes implementation detail or declarative structure to the file.
  **CN L663:** 这一行为文件补充了实现细节或声明式结构。
- **EN L664:** This line contributes implementation detail or declarative structure to the file.
  **CN L664:** 这一行为文件补充了实现细节或声明式结构。
- **EN L665:** This line contributes implementation detail or declarative structure to the file.
  **CN L665:** 这一行为文件补充了实现细节或声明式结构。
- **EN L666:** Blank line used to separate nearby declarations and improve readability.
  **CN L666:** 该空行用于分隔相邻声明并提升可读性。
- **EN L667:** This line contributes implementation detail or declarative structure to the file.
  **CN L667:** 这一行为文件补充了实现细节或声明式结构。
- **EN L668:** This line contributes implementation detail or declarative structure to the file.
  **CN L668:** 这一行为文件补充了实现细节或声明式结构。
- **EN L669:** This comment states: “Query barrier state by id.”, documenting the intent of the surrounding code.
  **CN L669:** 该注释写道：“Query barrier state by id.”，用于说明周围代码的意图。
- **EN L670:** This line contributes implementation detail or declarative structure to the file.
  **CN L670:** 这一行为文件补充了实现细节或声明式结构。
- **EN L671:** This line contributes implementation detail or declarative structure to the file.
  **CN L671:** 这一行为文件补充了实现细节或声明式结构。
- **EN L672:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L672:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 673-684 / 第 673-684 行

```tablegen
 673:   let results = (outs I32:$res);
 674:   let assemblyFormat = "`id` `=` $id attr-dict `->` type($res)";
 675: }
 676: 
 677: def ROCDL_GetNamedBarrierStateOp : ROCDL_ConcreteNonMemIntrOp<"s.get.named.barrier.state", [], 1, [], []>,
 678:   Arguments<(ins Arg<ROCDLBufferLDS, "", []>:$ptr)> {
 679:   let description = [{
 680:     Available on gfx1250+.
 681: 
 682:     Example:
 683:     ```mlir
 684:     // Query named barrier state by pointer.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L673:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L673:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L674:** This line contributes to the declaration or call of `type`.
  **CN L674:** 这一行为 `type` 的声明或调用提供内容。
- **EN L675:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L675:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L676:** Blank line used to separate nearby declarations and improve readability.
  **CN L676:** 该空行用于分隔相邻声明并提升可读性。
- **EN L677:** This TableGen `def` record introduces `ROCDL_GetNamedBarrierStateOp`, which later participates in generated MLIR code.
  **CN L677:** 该 TableGen `def` 记录引入了 `ROCDL_GetNamedBarrierStateOp`，后续会参与生成的 MLIR 代码。
- **EN L678:** This line contributes implementation detail or declarative structure to the file.
  **CN L678:** 这一行为文件补充了实现细节或声明式结构。
- **EN L679:** This line contributes implementation detail or declarative structure to the file.
  **CN L679:** 这一行为文件补充了实现细节或声明式结构。
- **EN L680:** This line contributes implementation detail or declarative structure to the file.
  **CN L680:** 这一行为文件补充了实现细节或声明式结构。
- **EN L681:** Blank line used to separate nearby declarations and improve readability.
  **CN L681:** 该空行用于分隔相邻声明并提升可读性。
- **EN L682:** This line contributes implementation detail or declarative structure to the file.
  **CN L682:** 这一行为文件补充了实现细节或声明式结构。
- **EN L683:** This line contributes implementation detail or declarative structure to the file.
  **CN L683:** 这一行为文件补充了实现细节或声明式结构。
- **EN L684:** This comment states: “Query named barrier state by pointer.”, documenting the intent of the surrounding code.
  **CN L684:** 该注释写道：“Query named barrier state by pointer.”，用于说明周围代码的意图。

### Lines 685-696 / 第 685-696 行

```tablegen
 685:     %0 = rocdl.s.get.named.barrier.state %ptr : !llvm.ptr<3> -> i32
 686:     ```
 687:   }];
 688:   let results = (outs I32:$res);
 689:   let assemblyFormat = "$ptr attr-dict `:` qualified(type($ptr)) `->` type($res)";
 690: }
 691: 
 692: def ROCDL_WakeupBarrierOp : ROCDL_ConcreteNonMemIntrOp<"s.wakeup.barrier", [], 0, [], []>,
 693:   Arguments<(ins Arg<ROCDLBufferLDS, "", []>:$ptr)> {
 694:   let description = [{
 695:     Wakes up waves associated with a given named barrier. Note, This op does not release waves waiting
 696:     at the barrier. It just signal other waves in the same work-group waiting on the indicated named barrier
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L685:** This line contributes implementation detail or declarative structure to the file.
  **CN L685:** 这一行为文件补充了实现细节或声明式结构。
- **EN L686:** This line contributes implementation detail or declarative structure to the file.
  **CN L686:** 这一行为文件补充了实现细节或声明式结构。
- **EN L687:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L687:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L688:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L688:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L689:** This line contributes to the declaration or call of `qualified`.
  **CN L689:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L690:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L690:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L691:** Blank line used to separate nearby declarations and improve readability.
  **CN L691:** 该空行用于分隔相邻声明并提升可读性。
- **EN L692:** This TableGen `def` record introduces `ROCDL_WakeupBarrierOp`, which later participates in generated MLIR code.
  **CN L692:** 该 TableGen `def` 记录引入了 `ROCDL_WakeupBarrierOp`，后续会参与生成的 MLIR 代码。
- **EN L693:** This line contributes implementation detail or declarative structure to the file.
  **CN L693:** 这一行为文件补充了实现细节或声明式结构。
- **EN L694:** This line contributes implementation detail or declarative structure to the file.
  **CN L694:** 这一行为文件补充了实现细节或声明式结构。
- **EN L695:** This line contributes implementation detail or declarative structure to the file.
  **CN L695:** 这一行为文件补充了实现细节或声明式结构。
- **EN L696:** This line contributes implementation detail or declarative structure to the file.
  **CN L696:** 这一行为文件补充了实现细节或声明式结构。

### Lines 697-708 / 第 697-708 行

```tablegen
 697:     to wake up.
 698:     Available on gfx1250+.
 699: 
 700:     Example:
 701:     ```mlir
 702:     // Wake up waves waiting on a named barrier.
 703:     rocdl.s.wakeup.barrier %ptr : !llvm.ptr<3>
 704:     ```
 705:   }];
 706:   let assemblyFormat = "$ptr attr-dict `:` qualified(type($ptr))";
 707: }
 708: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L697:** This line contributes implementation detail or declarative structure to the file.
  **CN L697:** 这一行为文件补充了实现细节或声明式结构。
- **EN L698:** This line contributes implementation detail or declarative structure to the file.
  **CN L698:** 这一行为文件补充了实现细节或声明式结构。
- **EN L699:** Blank line used to separate nearby declarations and improve readability.
  **CN L699:** 该空行用于分隔相邻声明并提升可读性。
- **EN L700:** This line contributes implementation detail or declarative structure to the file.
  **CN L700:** 这一行为文件补充了实现细节或声明式结构。
- **EN L701:** This line contributes implementation detail or declarative structure to the file.
  **CN L701:** 这一行为文件补充了实现细节或声明式结构。
- **EN L702:** This comment states: “Wake up waves waiting on a named barrier.”, documenting the intent of the surrounding code.
  **CN L702:** 该注释写道：“Wake up waves waiting on a named barrier.”，用于说明周围代码的意图。
- **EN L703:** This line contributes implementation detail or declarative structure to the file.
  **CN L703:** 这一行为文件补充了实现细节或声明式结构。
- **EN L704:** This line contributes implementation detail or declarative structure to the file.
  **CN L704:** 这一行为文件补充了实现细节或声明式结构。
- **EN L705:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L705:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L706:** This line contributes to the declaration or call of `qualified`.
  **CN L706:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L707:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L707:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L708:** Blank line used to separate nearby declarations and improve readability.
  **CN L708:** 该空行用于分隔相邻声明并提升可读性。

### Lines 709-720 / 第 709-720 行

```tablegen
 709: def ROCDL_WaitDscntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.dscnt", [], 0, [0], ["count"]>,
 710:   Arguments<(ins I16Attr:$count)> {
 711:   let summary = "Wait until DSCNT is less than or equal to `count`";
 712:   let description = [{
 713:       Wait for the counter specified to be less-than or equal-to the `count`
 714:       before continuing.
 715: 
 716:       Available on gfx12+.
 717: 
 718:       Example:
 719:       ```mlir
 720:       // Wait for data-sharing counter to drain.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L709:** This TableGen `def` record introduces `ROCDL_WaitDscntOp`, which later participates in generated MLIR code.
  **CN L709:** 该 TableGen `def` 记录引入了 `ROCDL_WaitDscntOp`，后续会参与生成的 MLIR 代码。
- **EN L710:** This line contributes implementation detail or declarative structure to the file.
  **CN L710:** 这一行为文件补充了实现细节或声明式结构。
- **EN L711:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L711:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L712:** This line contributes implementation detail or declarative structure to the file.
  **CN L712:** 这一行为文件补充了实现细节或声明式结构。
- **EN L713:** This line contributes implementation detail or declarative structure to the file.
  **CN L713:** 这一行为文件补充了实现细节或声明式结构。
- **EN L714:** This line contributes implementation detail or declarative structure to the file.
  **CN L714:** 这一行为文件补充了实现细节或声明式结构。
- **EN L715:** Blank line used to separate nearby declarations and improve readability.
  **CN L715:** 该空行用于分隔相邻声明并提升可读性。
- **EN L716:** This line contributes implementation detail or declarative structure to the file.
  **CN L716:** 这一行为文件补充了实现细节或声明式结构。
- **EN L717:** Blank line used to separate nearby declarations and improve readability.
  **CN L717:** 该空行用于分隔相邻声明并提升可读性。
- **EN L718:** This line contributes implementation detail or declarative structure to the file.
  **CN L718:** 这一行为文件补充了实现细节或声明式结构。
- **EN L719:** This line contributes implementation detail or declarative structure to the file.
  **CN L719:** 这一行为文件补充了实现细节或声明式结构。
- **EN L720:** This comment states: “Wait for data-sharing counter to drain.”, documenting the intent of the surrounding code.
  **CN L720:** 该注释写道：“Wait for data-sharing counter to drain.”，用于说明周围代码的意图。

### Lines 721-732 / 第 721-732 行

```tablegen
 721:       rocdl.s.wait.dscnt 0
 722:       ```
 723:   }];
 724:   let results = (outs);
 725:   let assemblyFormat = "$count attr-dict";
 726: }
 727: 
 728: def ROCDL_WaitLoadcntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.loadcnt", [], 0, [0], ["count"]>,
 729:   Arguments<(ins I16Attr:$count)> {
 730:   let summary = "Wait until LOADCNT is less than or equal to `count`";
 731:   let description = [{
 732:       Wait for the counter specified to be less-than or equal-to the `count`
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L721:** This line contributes implementation detail or declarative structure to the file.
  **CN L721:** 这一行为文件补充了实现细节或声明式结构。
- **EN L722:** This line contributes implementation detail or declarative structure to the file.
  **CN L722:** 这一行为文件补充了实现细节或声明式结构。
- **EN L723:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L723:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L724:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L724:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L725:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L725:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L726:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L726:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L727:** Blank line used to separate nearby declarations and improve readability.
  **CN L727:** 该空行用于分隔相邻声明并提升可读性。
- **EN L728:** This TableGen `def` record introduces `ROCDL_WaitLoadcntOp`, which later participates in generated MLIR code.
  **CN L728:** 该 TableGen `def` 记录引入了 `ROCDL_WaitLoadcntOp`，后续会参与生成的 MLIR 代码。
- **EN L729:** This line contributes implementation detail or declarative structure to the file.
  **CN L729:** 这一行为文件补充了实现细节或声明式结构。
- **EN L730:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L730:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L731:** This line contributes implementation detail or declarative structure to the file.
  **CN L731:** 这一行为文件补充了实现细节或声明式结构。
- **EN L732:** This line contributes implementation detail or declarative structure to the file.
  **CN L732:** 这一行为文件补充了实现细节或声明式结构。

### Lines 733-744 / 第 733-744 行

```tablegen
 733:       before continuing.
 734: 
 735:       Available on gfx12+.
 736: 
 737:       Example:
 738:       ```mlir
 739:       // Wait for load counter to drain.
 740:       rocdl.s.wait.loadcnt 0
 741:       ```
 742:   }];
 743:   let results = (outs);
 744:   let assemblyFormat = "$count attr-dict";
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L733:** This line contributes implementation detail or declarative structure to the file.
  **CN L733:** 这一行为文件补充了实现细节或声明式结构。
- **EN L734:** Blank line used to separate nearby declarations and improve readability.
  **CN L734:** 该空行用于分隔相邻声明并提升可读性。
- **EN L735:** This line contributes implementation detail or declarative structure to the file.
  **CN L735:** 这一行为文件补充了实现细节或声明式结构。
- **EN L736:** Blank line used to separate nearby declarations and improve readability.
  **CN L736:** 该空行用于分隔相邻声明并提升可读性。
- **EN L737:** This line contributes implementation detail or declarative structure to the file.
  **CN L737:** 这一行为文件补充了实现细节或声明式结构。
- **EN L738:** This line contributes implementation detail or declarative structure to the file.
  **CN L738:** 这一行为文件补充了实现细节或声明式结构。
- **EN L739:** This comment states: “Wait for load counter to drain.”, documenting the intent of the surrounding code.
  **CN L739:** 该注释写道：“Wait for load counter to drain.”，用于说明周围代码的意图。
- **EN L740:** This line contributes implementation detail or declarative structure to the file.
  **CN L740:** 这一行为文件补充了实现细节或声明式结构。
- **EN L741:** This line contributes implementation detail or declarative structure to the file.
  **CN L741:** 这一行为文件补充了实现细节或声明式结构。
- **EN L742:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L742:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L743:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L743:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L744:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L744:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 745-756 / 第 745-756 行

```tablegen
 745: }
 746: 
 747: def ROCDL_WaitStorecntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.storecnt", [], 0, [0], ["count"]>,
 748:   Arguments<(ins I16Attr:$count)> {
 749:   let summary = "Wait until STORECNT is less than or equal to `count`";
 750:   let description = [{
 751:       Wait for the counter specified to be less-than or equal-to the `count`
 752:       before continuing.
 753: 
 754:       Available on gfx12+.
 755: 
 756:       Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L745:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L745:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L746:** Blank line used to separate nearby declarations and improve readability.
  **CN L746:** 该空行用于分隔相邻声明并提升可读性。
- **EN L747:** This TableGen `def` record introduces `ROCDL_WaitStorecntOp`, which later participates in generated MLIR code.
  **CN L747:** 该 TableGen `def` 记录引入了 `ROCDL_WaitStorecntOp`，后续会参与生成的 MLIR 代码。
- **EN L748:** This line contributes implementation detail or declarative structure to the file.
  **CN L748:** 这一行为文件补充了实现细节或声明式结构。
- **EN L749:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L749:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L750:** This line contributes implementation detail or declarative structure to the file.
  **CN L750:** 这一行为文件补充了实现细节或声明式结构。
- **EN L751:** This line contributes implementation detail or declarative structure to the file.
  **CN L751:** 这一行为文件补充了实现细节或声明式结构。
- **EN L752:** This line contributes implementation detail or declarative structure to the file.
  **CN L752:** 这一行为文件补充了实现细节或声明式结构。
- **EN L753:** Blank line used to separate nearby declarations and improve readability.
  **CN L753:** 该空行用于分隔相邻声明并提升可读性。
- **EN L754:** This line contributes implementation detail or declarative structure to the file.
  **CN L754:** 这一行为文件补充了实现细节或声明式结构。
- **EN L755:** Blank line used to separate nearby declarations and improve readability.
  **CN L755:** 该空行用于分隔相邻声明并提升可读性。
- **EN L756:** This line contributes implementation detail or declarative structure to the file.
  **CN L756:** 这一行为文件补充了实现细节或声明式结构。

### Lines 757-768 / 第 757-768 行

```tablegen
 757:       ```mlir
 758:       // Wait for store counter to drain.
 759:       rocdl.s.wait.storecnt 0
 760:       ```
 761:   }];
 762:   let results = (outs);
 763:   let assemblyFormat = "$count attr-dict";
 764: }
 765: 
 766: def ROCDL_WaitExpcntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.expcnt", [], 0, [0], ["count"]>,
 767:   Arguments<(ins I16Attr:$count)> {
 768:   let summary = "Wait until EXPCNT is less than or equal to `count`";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L757:** This line contributes implementation detail or declarative structure to the file.
  **CN L757:** 这一行为文件补充了实现细节或声明式结构。
- **EN L758:** This comment states: “Wait for store counter to drain.”, documenting the intent of the surrounding code.
  **CN L758:** 该注释写道：“Wait for store counter to drain.”，用于说明周围代码的意图。
- **EN L759:** This line contributes implementation detail or declarative structure to the file.
  **CN L759:** 这一行为文件补充了实现细节或声明式结构。
- **EN L760:** This line contributes implementation detail or declarative structure to the file.
  **CN L760:** 这一行为文件补充了实现细节或声明式结构。
- **EN L761:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L761:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L762:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L762:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L763:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L763:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L764:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L764:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L765:** Blank line used to separate nearby declarations and improve readability.
  **CN L765:** 该空行用于分隔相邻声明并提升可读性。
- **EN L766:** This TableGen `def` record introduces `ROCDL_WaitExpcntOp`, which later participates in generated MLIR code.
  **CN L766:** 该 TableGen `def` 记录引入了 `ROCDL_WaitExpcntOp`，后续会参与生成的 MLIR 代码。
- **EN L767:** This line contributes implementation detail or declarative structure to the file.
  **CN L767:** 这一行为文件补充了实现细节或声明式结构。
- **EN L768:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L768:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 769-780 / 第 769-780 行

```tablegen
 769:   let description = [{
 770:       Wait for the counter specified to be less-than or equal-to the `count`
 771:       before continuing.
 772: 
 773:       Available on gfx12+.
 774: 
 775:       Example:
 776:       ```mlir
 777:       // Wait for export counter to drain.
 778:       rocdl.s.wait.expcnt 0
 779:       ```
 780:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L769:** This line contributes implementation detail or declarative structure to the file.
  **CN L769:** 这一行为文件补充了实现细节或声明式结构。
- **EN L770:** This line contributes implementation detail or declarative structure to the file.
  **CN L770:** 这一行为文件补充了实现细节或声明式结构。
- **EN L771:** This line contributes implementation detail or declarative structure to the file.
  **CN L771:** 这一行为文件补充了实现细节或声明式结构。
- **EN L772:** Blank line used to separate nearby declarations and improve readability.
  **CN L772:** 该空行用于分隔相邻声明并提升可读性。
- **EN L773:** This line contributes implementation detail or declarative structure to the file.
  **CN L773:** 这一行为文件补充了实现细节或声明式结构。
- **EN L774:** Blank line used to separate nearby declarations and improve readability.
  **CN L774:** 该空行用于分隔相邻声明并提升可读性。
- **EN L775:** This line contributes implementation detail or declarative structure to the file.
  **CN L775:** 这一行为文件补充了实现细节或声明式结构。
- **EN L776:** This line contributes implementation detail or declarative structure to the file.
  **CN L776:** 这一行为文件补充了实现细节或声明式结构。
- **EN L777:** This comment states: “Wait for export counter to drain.”, documenting the intent of the surrounding code.
  **CN L777:** 该注释写道：“Wait for export counter to drain.”，用于说明周围代码的意图。
- **EN L778:** This line contributes implementation detail or declarative structure to the file.
  **CN L778:** 这一行为文件补充了实现细节或声明式结构。
- **EN L779:** This line contributes implementation detail or declarative structure to the file.
  **CN L779:** 这一行为文件补充了实现细节或声明式结构。
- **EN L780:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L780:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 781-792 / 第 781-792 行

```tablegen
 781:   let results = (outs);
 782:   let assemblyFormat = "$count attr-dict";
 783: }
 784: 
 785: def ROCDL_WaitAsynccntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.asynccnt", [], 0, [0], ["count"]>,
 786:   Arguments<(ins I16Attr:$count)> {
 787:   let summary = "Wait until ASYNCCNT is less than or equal to `count`";
 788:   let description = [{
 789:       Wait for the counter specified to be less-than or equal-to the `count`
 790:       before continuing.
 791: 
 792:       Available on gfx1250+.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L781:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L781:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L782:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L782:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L783:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L783:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L784:** Blank line used to separate nearby declarations and improve readability.
  **CN L784:** 该空行用于分隔相邻声明并提升可读性。
- **EN L785:** This TableGen `def` record introduces `ROCDL_WaitAsynccntOp`, which later participates in generated MLIR code.
  **CN L785:** 该 TableGen `def` 记录引入了 `ROCDL_WaitAsynccntOp`，后续会参与生成的 MLIR 代码。
- **EN L786:** This line contributes implementation detail or declarative structure to the file.
  **CN L786:** 这一行为文件补充了实现细节或声明式结构。
- **EN L787:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L787:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L788:** This line contributes implementation detail or declarative structure to the file.
  **CN L788:** 这一行为文件补充了实现细节或声明式结构。
- **EN L789:** This line contributes implementation detail or declarative structure to the file.
  **CN L789:** 这一行为文件补充了实现细节或声明式结构。
- **EN L790:** This line contributes implementation detail or declarative structure to the file.
  **CN L790:** 这一行为文件补充了实现细节或声明式结构。
- **EN L791:** Blank line used to separate nearby declarations and improve readability.
  **CN L791:** 该空行用于分隔相邻声明并提升可读性。
- **EN L792:** This line contributes implementation detail or declarative structure to the file.
  **CN L792:** 这一行为文件补充了实现细节或声明式结构。

### Lines 793-804 / 第 793-804 行

```tablegen
 793: 
 794:       Example:
 795:       ```mlir
 796:       // Wait for async counter to drain.
 797:       rocdl.s.wait.asynccnt 0
 798:       ```
 799:   }];
 800:   let results = (outs);
 801:   let assemblyFormat = "$count attr-dict";
 802: }
 803: 
 804: def ROCDL_WaitTensorcntOp: ROCDL_ConcreteNonMemIntrOp<"s.wait.tensorcnt", [], 0, [0], ["count"]>,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L793:** Blank line used to separate nearby declarations and improve readability.
  **CN L793:** 该空行用于分隔相邻声明并提升可读性。
- **EN L794:** This line contributes implementation detail or declarative structure to the file.
  **CN L794:** 这一行为文件补充了实现细节或声明式结构。
- **EN L795:** This line contributes implementation detail or declarative structure to the file.
  **CN L795:** 这一行为文件补充了实现细节或声明式结构。
- **EN L796:** This comment states: “Wait for async counter to drain.”, documenting the intent of the surrounding code.
  **CN L796:** 该注释写道：“Wait for async counter to drain.”，用于说明周围代码的意图。
- **EN L797:** This line contributes implementation detail or declarative structure to the file.
  **CN L797:** 这一行为文件补充了实现细节或声明式结构。
- **EN L798:** This line contributes implementation detail or declarative structure to the file.
  **CN L798:** 这一行为文件补充了实现细节或声明式结构。
- **EN L799:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L799:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L800:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L800:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L801:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L801:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L802:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L802:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L803:** Blank line used to separate nearby declarations and improve readability.
  **CN L803:** 该空行用于分隔相邻声明并提升可读性。
- **EN L804:** This TableGen `def` record introduces `ROCDL_WaitTensorcntOp`, which later participates in generated MLIR code.
  **CN L804:** 该 TableGen `def` 记录引入了 `ROCDL_WaitTensorcntOp`，后续会参与生成的 MLIR 代码。

### Lines 805-816 / 第 805-816 行

```tablegen
 805:   Arguments<(ins I16Attr:$count)> {
 806:   let summary = "Wait until TENSORCNT is less than or equal to `count`";
 807:   let description = [{
 808:       Wait for the counter specified to be less-than or equal-to the `count`
 809:       before continuing.
 810: 
 811:       Available on gfx1250+.
 812: 
 813:       Example:
 814:       ```mlir
 815:       // Wait for tensor counter to drain.
 816:       rocdl.s.wait.tensorcnt 0
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L805:** This line contributes implementation detail or declarative structure to the file.
  **CN L805:** 这一行为文件补充了实现细节或声明式结构。
- **EN L806:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L806:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L807:** This line contributes implementation detail or declarative structure to the file.
  **CN L807:** 这一行为文件补充了实现细节或声明式结构。
- **EN L808:** This line contributes implementation detail or declarative structure to the file.
  **CN L808:** 这一行为文件补充了实现细节或声明式结构。
- **EN L809:** This line contributes implementation detail or declarative structure to the file.
  **CN L809:** 这一行为文件补充了实现细节或声明式结构。
- **EN L810:** Blank line used to separate nearby declarations and improve readability.
  **CN L810:** 该空行用于分隔相邻声明并提升可读性。
- **EN L811:** This line contributes implementation detail or declarative structure to the file.
  **CN L811:** 这一行为文件补充了实现细节或声明式结构。
- **EN L812:** Blank line used to separate nearby declarations and improve readability.
  **CN L812:** 该空行用于分隔相邻声明并提升可读性。
- **EN L813:** This line contributes implementation detail or declarative structure to the file.
  **CN L813:** 这一行为文件补充了实现细节或声明式结构。
- **EN L814:** This line contributes implementation detail or declarative structure to the file.
  **CN L814:** 这一行为文件补充了实现细节或声明式结构。
- **EN L815:** This comment states: “Wait for tensor counter to drain.”, documenting the intent of the surrounding code.
  **CN L815:** 该注释写道：“Wait for tensor counter to drain.”，用于说明周围代码的意图。
- **EN L816:** This line contributes implementation detail or declarative structure to the file.
  **CN L816:** 这一行为文件补充了实现细节或声明式结构。

### Lines 817-828 / 第 817-828 行

```tablegen
 817:       ```
 818:   }];
 819:   let results = (outs);
 820:   let assemblyFormat = "$count attr-dict";
 821: }
 822: 
 823: def ROCDL_AsyncmarkOp : ROCDL_ConcreteNonMemIntrOp<"asyncmark", [], 0>,
 824:     Arguments<(ins)> {
 825:   let summary = "Mark the end of a group of asynchronous operations";
 826:   let description = [{
 827:       This operation, in conjunction with `rocdl.wait.asyncmark`, forms the
 828:       compiler-provided framework for tracking explicitly asynchronous
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L817:** This line contributes implementation detail or declarative structure to the file.
  **CN L817:** 这一行为文件补充了实现细节或声明式结构。
- **EN L818:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L818:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L819:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L819:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L820:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L820:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L821:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L821:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L822:** Blank line used to separate nearby declarations and improve readability.
  **CN L822:** 该空行用于分隔相邻声明并提升可读性。
- **EN L823:** This TableGen `def` record introduces `ROCDL_AsyncmarkOp`, which later participates in generated MLIR code.
  **CN L823:** 该 TableGen `def` 记录引入了 `ROCDL_AsyncmarkOp`，后续会参与生成的 MLIR 代码。
- **EN L824:** This line contributes implementation detail or declarative structure to the file.
  **CN L824:** 这一行为文件补充了实现细节或声明式结构。
- **EN L825:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L825:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L826:** This line contributes implementation detail or declarative structure to the file.
  **CN L826:** 这一行为文件补充了实现细节或声明式结构。
- **EN L827:** This line contributes implementation detail or declarative structure to the file.
  **CN L827:** 这一行为文件补充了实现细节或声明式结构。
- **EN L828:** This line contributes implementation detail or declarative structure to the file.
  **CN L828:** 这一行为文件补充了实现细节或声明式结构。

### Lines 829-840 / 第 829-840 行

```tablegen
 829:       memory operations, such as copies to LDS that use async intrinsics
 830:       and gfx1250's tensor loads.
 831: 
 832:       Details of its behavior can be found in
 833:       [the LLVM documentation on async tracking](/llvm/docs/AMDGPUAsyncOperations.rst).
 834: 
 835:       See `rocdl.wait.asyncmark`'s documentation for a usage example.
 836: 
 837:       Example:
 838:       ```mlir
 839:       // Mark the end of an async operation group.
 840:       rocdl.asyncmark
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L829:** This line contributes implementation detail or declarative structure to the file.
  **CN L829:** 这一行为文件补充了实现细节或声明式结构。
- **EN L830:** This line contributes implementation detail or declarative structure to the file.
  **CN L830:** 这一行为文件补充了实现细节或声明式结构。
- **EN L831:** Blank line used to separate nearby declarations and improve readability.
  **CN L831:** 该空行用于分隔相邻声明并提升可读性。
- **EN L832:** This line contributes implementation detail or declarative structure to the file.
  **CN L832:** 这一行为文件补充了实现细节或声明式结构。
- **EN L833:** This line contributes implementation detail or declarative structure to the file.
  **CN L833:** 这一行为文件补充了实现细节或声明式结构。
- **EN L834:** Blank line used to separate nearby declarations and improve readability.
  **CN L834:** 该空行用于分隔相邻声明并提升可读性。
- **EN L835:** This line contributes implementation detail or declarative structure to the file.
  **CN L835:** 这一行为文件补充了实现细节或声明式结构。
- **EN L836:** Blank line used to separate nearby declarations and improve readability.
  **CN L836:** 该空行用于分隔相邻声明并提升可读性。
- **EN L837:** This line contributes implementation detail or declarative structure to the file.
  **CN L837:** 这一行为文件补充了实现细节或声明式结构。
- **EN L838:** This line contributes implementation detail or declarative structure to the file.
  **CN L838:** 这一行为文件补充了实现细节或声明式结构。
- **EN L839:** This comment states: “Mark the end of an async operation group.”, documenting the intent of the surrounding code.
  **CN L839:** 该注释写道：“Mark the end of an async operation group.”，用于说明周围代码的意图。
- **EN L840:** This line contributes implementation detail or declarative structure to the file.
  **CN L840:** 这一行为文件补充了实现细节或声明式结构。

### Lines 841-852 / 第 841-852 行

```tablegen
 841:       ```
 842: 
 843:       Available on gfx9 and later.
 844:   }];
 845:   let results = (outs);
 846:   let assemblyFormat = "attr-dict";
 847: }
 848: 
 849: def ROCDL_WaitAsyncmarkOp: ROCDL_ConcreteNonMemIntrOp<"wait.asyncmark", [], 0, [0], ["count"]>,
 850:     Arguments<(ins I16Attr:$count)> {
 851:   let summary = "Wait until N or fewer async operation groups are unexecuted";
 852:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L841:** This line contributes implementation detail or declarative structure to the file.
  **CN L841:** 这一行为文件补充了实现细节或声明式结构。
- **EN L842:** Blank line used to separate nearby declarations and improve readability.
  **CN L842:** 该空行用于分隔相邻声明并提升可读性。
- **EN L843:** This line contributes implementation detail or declarative structure to the file.
  **CN L843:** 这一行为文件补充了实现细节或声明式结构。
- **EN L844:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L844:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L845:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L845:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L846:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L846:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L847:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L847:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L848:** Blank line used to separate nearby declarations and improve readability.
  **CN L848:** 该空行用于分隔相邻声明并提升可读性。
- **EN L849:** This TableGen `def` record introduces `ROCDL_WaitAsyncmarkOp`, which later participates in generated MLIR code.
  **CN L849:** 该 TableGen `def` 记录引入了 `ROCDL_WaitAsyncmarkOp`，后续会参与生成的 MLIR 代码。
- **EN L850:** This line contributes implementation detail or declarative structure to the file.
  **CN L850:** 这一行为文件补充了实现细节或声明式结构。
- **EN L851:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L851:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L852:** This line contributes implementation detail or declarative structure to the file.
  **CN L852:** 这一行为文件补充了实现细节或声明式结构。

### Lines 853-864 / 第 853-864 行

```tablegen
 853:       This operation, along with `rocdl.asyncmark`, forms the compiler-provided
 854:       framework for explicitly tracking asynchronous operations.
 855: 
 856:       At the point where a wait.asyncmark operation is executed, all async operations
 857:       that were parts of any async group (established by asyncmark in program order)
 858:       other than the `count` previously-added ones will have finished executing.
 859: 
 860:       For more detail, including on how this mechanism composes with function calls,
 861:       see [the LLVM documentation on async tracking](/llvm/docs/AMDGPUAsyncOperations.rst).
 862: 
 863:       Available on gfx9 and later.
 864: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L853:** This line contributes implementation detail or declarative structure to the file.
  **CN L853:** 这一行为文件补充了实现细节或声明式结构。
- **EN L854:** This line contributes implementation detail or declarative structure to the file.
  **CN L854:** 这一行为文件补充了实现细节或声明式结构。
- **EN L855:** Blank line used to separate nearby declarations and improve readability.
  **CN L855:** 该空行用于分隔相邻声明并提升可读性。
- **EN L856:** This line contributes implementation detail or declarative structure to the file.
  **CN L856:** 这一行为文件补充了实现细节或声明式结构。
- **EN L857:** This line contributes to the declaration or call of `group`.
  **CN L857:** 这一行为 `group` 的声明或调用提供内容。
- **EN L858:** This line contributes implementation detail or declarative structure to the file.
  **CN L858:** 这一行为文件补充了实现细节或声明式结构。
- **EN L859:** Blank line used to separate nearby declarations and improve readability.
  **CN L859:** 该空行用于分隔相邻声明并提升可读性。
- **EN L860:** This line contributes implementation detail or declarative structure to the file.
  **CN L860:** 这一行为文件补充了实现细节或声明式结构。
- **EN L861:** This line contributes implementation detail or declarative structure to the file.
  **CN L861:** 这一行为文件补充了实现细节或声明式结构。
- **EN L862:** Blank line used to separate nearby declarations and improve readability.
  **CN L862:** 该空行用于分隔相邻声明并提升可读性。
- **EN L863:** This line contributes implementation detail or declarative structure to the file.
  **CN L863:** 这一行为文件补充了实现细节或声明式结构。
- **EN L864:** Blank line used to separate nearby declarations and improve readability.
  **CN L864:** 该空行用于分隔相邻声明并提升可读性。

### Lines 865-876 / 第 865-876 行

```tablegen
 865:       Example:
 866:       ```mlir
 867:       // Wait until at most N async groups remain outstanding.
 868:       rocdl.wait.asyncmark 1
 869:       ```
 870: 
 871:       Usage example:
 872:       ```mlir
 873:       rocdl.tensor.load.to.lds ...
 874:       rocdl.global.async.load.to.lds ...
 875: 
 876:       rocdl.asyncmark
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L865:** This line contributes implementation detail or declarative structure to the file.
  **CN L865:** 这一行为文件补充了实现细节或声明式结构。
- **EN L866:** This line contributes implementation detail or declarative structure to the file.
  **CN L866:** 这一行为文件补充了实现细节或声明式结构。
- **EN L867:** This comment states: “Wait until at most N async groups remain outstanding.”, documenting the intent of the surrounding code.
  **CN L867:** 该注释写道：“Wait until at most N async groups remain outstanding.”，用于说明周围代码的意图。
- **EN L868:** This line contributes implementation detail or declarative structure to the file.
  **CN L868:** 这一行为文件补充了实现细节或声明式结构。
- **EN L869:** This line contributes implementation detail or declarative structure to the file.
  **CN L869:** 这一行为文件补充了实现细节或声明式结构。
- **EN L870:** Blank line used to separate nearby declarations and improve readability.
  **CN L870:** 该空行用于分隔相邻声明并提升可读性。
- **EN L871:** This line contributes implementation detail or declarative structure to the file.
  **CN L871:** 这一行为文件补充了实现细节或声明式结构。
- **EN L872:** This line contributes implementation detail or declarative structure to the file.
  **CN L872:** 这一行为文件补充了实现细节或声明式结构。
- **EN L873:** This line contributes implementation detail or declarative structure to the file.
  **CN L873:** 这一行为文件补充了实现细节或声明式结构。
- **EN L874:** This line contributes implementation detail or declarative structure to the file.
  **CN L874:** 这一行为文件补充了实现细节或声明式结构。
- **EN L875:** Blank line used to separate nearby declarations and improve readability.
  **CN L875:** 该空行用于分隔相邻声明并提升可读性。
- **EN L876:** This line contributes implementation detail or declarative structure to the file.
  **CN L876:** 这一行为文件补充了实现细节或声明式结构。

### Lines 877-888 / 第 877-888 行

```tablegen
 877: 
 878:       rocdl.tensor.load.to.lds ...
 879:       rocdl.global.async.load.to.lds ...
 880: 
 881:       rocdl.asyncmark
 882: 
 883:       rocdl.wait.asyncmark 1 // First group of loads completes after this
 884:       ```
 885:   }];
 886:   let results = (outs);
 887:   let assemblyFormat = "$count attr-dict";
 888: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L877:** Blank line used to separate nearby declarations and improve readability.
  **CN L877:** 该空行用于分隔相邻声明并提升可读性。
- **EN L878:** This line contributes implementation detail or declarative structure to the file.
  **CN L878:** 这一行为文件补充了实现细节或声明式结构。
- **EN L879:** This line contributes implementation detail or declarative structure to the file.
  **CN L879:** 这一行为文件补充了实现细节或声明式结构。
- **EN L880:** Blank line used to separate nearby declarations and improve readability.
  **CN L880:** 该空行用于分隔相邻声明并提升可读性。
- **EN L881:** This line contributes implementation detail or declarative structure to the file.
  **CN L881:** 这一行为文件补充了实现细节或声明式结构。
- **EN L882:** Blank line used to separate nearby declarations and improve readability.
  **CN L882:** 该空行用于分隔相邻声明并提升可读性。
- **EN L883:** This line contributes implementation detail or declarative structure to the file.
  **CN L883:** 这一行为文件补充了实现细节或声明式结构。
- **EN L884:** This line contributes implementation detail or declarative structure to the file.
  **CN L884:** 这一行为文件补充了实现细节或声明式结构。
- **EN L885:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L885:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L886:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L886:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L887:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L887:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L888:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L888:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 889-900 / 第 889-900 行

```tablegen
 889: 
 890: def ROCDL_SetPrioOp : ROCDL_ConcreteNonMemIntrOp<"s.setprio", [], 0, [0], ["priority"]>,
 891:   Arguments<(ins I16Attr:$priority)> {
 892:   let assemblyFormat = "$priority attr-dict";
 893:   let description = [{
 894:     Set the wavefront scheduling priority.
 895: 
 896:     Example:
 897:     ```mlir
 898:     // Set priority to 0.
 899:     rocdl.s.setprio 0
 900:     ```
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L889:** Blank line used to separate nearby declarations and improve readability.
  **CN L889:** 该空行用于分隔相邻声明并提升可读性。
- **EN L890:** This TableGen `def` record introduces `ROCDL_SetPrioOp`, which later participates in generated MLIR code.
  **CN L890:** 该 TableGen `def` 记录引入了 `ROCDL_SetPrioOp`，后续会参与生成的 MLIR 代码。
- **EN L891:** This line contributes implementation detail or declarative structure to the file.
  **CN L891:** 这一行为文件补充了实现细节或声明式结构。
- **EN L892:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L892:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L893:** This line contributes implementation detail or declarative structure to the file.
  **CN L893:** 这一行为文件补充了实现细节或声明式结构。
- **EN L894:** This line contributes implementation detail or declarative structure to the file.
  **CN L894:** 这一行为文件补充了实现细节或声明式结构。
- **EN L895:** Blank line used to separate nearby declarations and improve readability.
  **CN L895:** 该空行用于分隔相邻声明并提升可读性。
- **EN L896:** This line contributes implementation detail or declarative structure to the file.
  **CN L896:** 这一行为文件补充了实现细节或声明式结构。
- **EN L897:** This line contributes implementation detail or declarative structure to the file.
  **CN L897:** 这一行为文件补充了实现细节或声明式结构。
- **EN L898:** This comment states: “Set priority to 0.”, documenting the intent of the surrounding code.
  **CN L898:** 该注释写道：“Set priority to 0.”，用于说明周围代码的意图。
- **EN L899:** This line contributes implementation detail or declarative structure to the file.
  **CN L899:** 这一行为文件补充了实现细节或声明式结构。
- **EN L900:** This line contributes implementation detail or declarative structure to the file.
  **CN L900:** 这一行为文件补充了实现细节或声明式结构。

### Lines 901-912 / 第 901-912 行

```tablegen
 901:   }];
 902: }
 903: 
 904: def ROCDL_SchedBarrier : ROCDL_ConcreteNonMemIntrOp<"sched.barrier", [], 0, [0],["mask"]>,
 905:   Arguments<(ins I32Attr:$mask)> {
 906:   let assemblyFormat = "$mask attr-dict";
 907:   let description = [{
 908:     Insert a scheduling barrier with the given mask. The mask is a
 909:     bitfield that controls which instruction types may be scheduled
 910:     across the barrier (e.g. `0x0000` = no instructions may cross,
 911:     `0x0001` = ALU only, `0x0010` = all VMEM, etc.). See
 912:     https://github.com/llvm/llvm-project/blob/main/llvm/include/llvm/IR/IntrinsicsAMDGPU.td#L349
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L901:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L901:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L902:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L902:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L903:** Blank line used to separate nearby declarations and improve readability.
  **CN L903:** 该空行用于分隔相邻声明并提升可读性。
- **EN L904:** This TableGen `def` record introduces `ROCDL_SchedBarrier`, which later participates in generated MLIR code.
  **CN L904:** 该 TableGen `def` 记录引入了 `ROCDL_SchedBarrier`，后续会参与生成的 MLIR 代码。
- **EN L905:** This line contributes implementation detail or declarative structure to the file.
  **CN L905:** 这一行为文件补充了实现细节或声明式结构。
- **EN L906:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L906:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L907:** This line contributes implementation detail or declarative structure to the file.
  **CN L907:** 这一行为文件补充了实现细节或声明式结构。
- **EN L908:** This line contributes implementation detail or declarative structure to the file.
  **CN L908:** 这一行为文件补充了实现细节或声明式结构。
- **EN L909:** This line contributes implementation detail or declarative structure to the file.
  **CN L909:** 这一行为文件补充了实现细节或声明式结构。
- **EN L910:** This line contributes to the declaration or call of `barrier`.
  **CN L910:** 这一行为 `barrier` 的声明或调用提供内容。
- **EN L911:** This line contributes implementation detail or declarative structure to the file.
  **CN L911:** 这一行为文件补充了实现细节或声明式结构。
- **EN L912:** This line contributes implementation detail or declarative structure to the file.
  **CN L912:** 这一行为文件补充了实现细节或声明式结构。

### Lines 913-924 / 第 913-924 行

```tablegen
 913:     for the full list of mask values.
 914: 
 915:     Example:
 916:     ```mlir
 917:     // Scheduling barrier with mask 0.
 918:     rocdl.sched.barrier 0
 919:     ```
 920:   }];
 921: }
 922: 
 923: def ROCDL_SchedGroupBarrier
 924:   : ROCDL_ConcreteNonMemIntrOp<"sched.group.barrier", [], 0,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L913:** This line contributes implementation detail or declarative structure to the file.
  **CN L913:** 这一行为文件补充了实现细节或声明式结构。
- **EN L914:** Blank line used to separate nearby declarations and improve readability.
  **CN L914:** 该空行用于分隔相邻声明并提升可读性。
- **EN L915:** This line contributes implementation detail or declarative structure to the file.
  **CN L915:** 这一行为文件补充了实现细节或声明式结构。
- **EN L916:** This line contributes implementation detail or declarative structure to the file.
  **CN L916:** 这一行为文件补充了实现细节或声明式结构。
- **EN L917:** This comment states: “Scheduling barrier with mask 0.”, documenting the intent of the surrounding code.
  **CN L917:** 该注释写道：“Scheduling barrier with mask 0.”，用于说明周围代码的意图。
- **EN L918:** This line contributes implementation detail or declarative structure to the file.
  **CN L918:** 这一行为文件补充了实现细节或声明式结构。
- **EN L919:** This line contributes implementation detail or declarative structure to the file.
  **CN L919:** 这一行为文件补充了实现细节或声明式结构。
- **EN L920:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L920:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L921:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L921:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L922:** Blank line used to separate nearby declarations and improve readability.
  **CN L922:** 该空行用于分隔相邻声明并提升可读性。
- **EN L923:** This TableGen `def` record introduces `ROCDL_SchedGroupBarrier`, which later participates in generated MLIR code.
  **CN L923:** 该 TableGen `def` 记录引入了 `ROCDL_SchedGroupBarrier`，后续会参与生成的 MLIR 代码。
- **EN L924:** This line contributes implementation detail or declarative structure to the file.
  **CN L924:** 这一行为文件补充了实现细节或声明式结构。

### Lines 925-936 / 第 925-936 行

```tablegen
 925:       [0, 1, 2], ["mask", "size", "groupId"]>,
 926:     Arguments<(ins I32Attr:$mask, I32Attr:$size, I32Attr:$groupId)> {
 927:   let assemblyFormat = "$mask `,` $size `,` $groupId attr-dict";
 928:   let description = [{
 929:     Insert a scheduling group barrier.
 930: 
 931:     Example:
 932:     ```mlir
 933:     // Schedule group barrier with mask, size, and group id.
 934:     rocdl.sched.group.barrier 8, 1, 0
 935:     ```
 936:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L925:** This line contributes implementation detail or declarative structure to the file.
  **CN L925:** 这一行为文件补充了实现细节或声明式结构。
- **EN L926:** This line contributes implementation detail or declarative structure to the file.
  **CN L926:** 这一行为文件补充了实现细节或声明式结构。
- **EN L927:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L927:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L928:** This line contributes implementation detail or declarative structure to the file.
  **CN L928:** 这一行为文件补充了实现细节或声明式结构。
- **EN L929:** This line contributes implementation detail or declarative structure to the file.
  **CN L929:** 这一行为文件补充了实现细节或声明式结构。
- **EN L930:** Blank line used to separate nearby declarations and improve readability.
  **CN L930:** 该空行用于分隔相邻声明并提升可读性。
- **EN L931:** This line contributes implementation detail or declarative structure to the file.
  **CN L931:** 这一行为文件补充了实现细节或声明式结构。
- **EN L932:** This line contributes implementation detail or declarative structure to the file.
  **CN L932:** 这一行为文件补充了实现细节或声明式结构。
- **EN L933:** This comment states: “Schedule group barrier with mask, size, and group id.”, documenting the intent of the surrounding code.
  **CN L933:** 该注释写道：“Schedule group barrier with mask, size, and group id.”，用于说明周围代码的意图。
- **EN L934:** This line contributes implementation detail or declarative structure to the file.
  **CN L934:** 这一行为文件补充了实现细节或声明式结构。
- **EN L935:** This line contributes implementation detail or declarative structure to the file.
  **CN L935:** 这一行为文件补充了实现细节或声明式结构。
- **EN L936:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L936:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 937-948 / 第 937-948 行

```tablegen
 937: }
 938: 
 939: def ROCDL_IglpOpt : ROCDL_ConcreteNonMemIntrOp<"iglp.opt", [], 0, [0], ["variant"]>,
 940:   Arguments<(ins I32Attr:$variant)> {
 941:   let assemblyFormat = "$variant attr-dict";
 942:   let description = [{
 943:     Instruction-group-level parallelism optimization hint.
 944: 
 945:     Example:
 946:     ```mlir
 947:     // IGLP optimization hint variant 0.
 948:     rocdl.iglp.opt 0
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L937:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L937:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L938:** Blank line used to separate nearby declarations and improve readability.
  **CN L938:** 该空行用于分隔相邻声明并提升可读性。
- **EN L939:** This TableGen `def` record introduces `ROCDL_IglpOpt`, which later participates in generated MLIR code.
  **CN L939:** 该 TableGen `def` 记录引入了 `ROCDL_IglpOpt`，后续会参与生成的 MLIR 代码。
- **EN L940:** This line contributes implementation detail or declarative structure to the file.
  **CN L940:** 这一行为文件补充了实现细节或声明式结构。
- **EN L941:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L941:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L942:** This line contributes implementation detail or declarative structure to the file.
  **CN L942:** 这一行为文件补充了实现细节或声明式结构。
- **EN L943:** This line contributes implementation detail or declarative structure to the file.
  **CN L943:** 这一行为文件补充了实现细节或声明式结构。
- **EN L944:** Blank line used to separate nearby declarations and improve readability.
  **CN L944:** 该空行用于分隔相邻声明并提升可读性。
- **EN L945:** This line contributes implementation detail or declarative structure to the file.
  **CN L945:** 这一行为文件补充了实现细节或声明式结构。
- **EN L946:** This line contributes implementation detail or declarative structure to the file.
  **CN L946:** 这一行为文件补充了实现细节或声明式结构。
- **EN L947:** This comment states: “IGLP optimization hint variant 0.”, documenting the intent of the surrounding code.
  **CN L947:** 该注释写道：“IGLP optimization hint variant 0.”，用于说明周围代码的意图。
- **EN L948:** This line contributes implementation detail or declarative structure to the file.
  **CN L948:** 这一行为文件补充了实现细节或声明式结构。

### Lines 949-960 / 第 949-960 行

```tablegen
 949:     ```
 950:   }];
 951: }
 952: 
 953: //===---------------------------------------------------------------------===//
 954: // Xdlops intrinsics
 955: 
 956: class ROCDL_Mfma_IntrOp<string mnemonic, ROCDL_NamedType ABType, ROCDL_NamedType CDType> :
 957:   ROCDL_IntrOp<mnemonic, [], [], [], 1, 0, 0, 0, [3, 4, 5], ["cbsz", "abid", "blgp"]>,
 958:   Arguments<(ins
 959:              ABType:$a,
 960:              ABType:$b,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L949:** This line contributes implementation detail or declarative structure to the file.
  **CN L949:** 这一行为文件补充了实现细节或声明式结构。
- **EN L950:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L950:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L951:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L951:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L952:** Blank line used to separate nearby declarations and improve readability.
  **CN L952:** 该空行用于分隔相邻声明并提升可读性。
- **EN L953:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L953:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L954:** This comment states: “Xdlops intrinsics”, documenting the intent of the surrounding code.
  **CN L954:** 该注释写道：“Xdlops intrinsics”，用于说明周围代码的意图。
- **EN L955:** Blank line used to separate nearby declarations and improve readability.
  **CN L955:** 该空行用于分隔相邻声明并提升可读性。
- **EN L956:** This TableGen `class` record introduces `ROCDL_Mfma_IntrOp`, which later participates in generated MLIR code.
  **CN L956:** 该 TableGen `class` 记录引入了 `ROCDL_Mfma_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L957:** This line contributes implementation detail or declarative structure to the file.
  **CN L957:** 这一行为文件补充了实现细节或声明式结构。
- **EN L958:** This line contributes implementation detail or declarative structure to the file.
  **CN L958:** 这一行为文件补充了实现细节或声明式结构。
- **EN L959:** This line contributes implementation detail or declarative structure to the file.
  **CN L959:** 这一行为文件补充了实现细节或声明式结构。
- **EN L960:** This line contributes implementation detail or declarative structure to the file.
  **CN L960:** 这一行为文件补充了实现细节或声明式结构。

### Lines 961-972 / 第 961-972 行

```tablegen
 961:              CDType:$c,
 962:              I32Attr:$cbsz,
 963:              I32Attr:$abid,
 964:              I32Attr:$blgp)> {
 965:   let results = (outs CDType:$res);
 966:   let assemblyFormat = [{
 967:     $a `,` $b `,` $c `,` $cbsz `,` $abid `,` $blgp attr-dict `:` functional-type(operands, $res)
 968:   }];
 969:   let description = [{
 970:     Matrix fused multiply-add (MFMA) intrinsic. Computes `D = A * B + C`
 971:     with matrix operands. The `cbsz`, `abid`, and `blgp` attributes control
 972:     broadcast and block layout modes.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L961:** This line contributes implementation detail or declarative structure to the file.
  **CN L961:** 这一行为文件补充了实现细节或声明式结构。
- **EN L962:** This line contributes implementation detail or declarative structure to the file.
  **CN L962:** 这一行为文件补充了实现细节或声明式结构。
- **EN L963:** This line contributes implementation detail or declarative structure to the file.
  **CN L963:** 这一行为文件补充了实现细节或声明式结构。
- **EN L964:** This line contributes implementation detail or declarative structure to the file.
  **CN L964:** 这一行为文件补充了实现细节或声明式结构。
- **EN L965:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L965:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L966:** This line contributes implementation detail or declarative structure to the file.
  **CN L966:** 这一行为文件补充了实现细节或声明式结构。
- **EN L967:** This line contributes to the declaration or call of `type`.
  **CN L967:** 这一行为 `type` 的声明或调用提供内容。
- **EN L968:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L968:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L969:** This line contributes implementation detail or declarative structure to the file.
  **CN L969:** 这一行为文件补充了实现细节或声明式结构。
- **EN L970:** This line contributes to the declaration or call of `add`.
  **CN L970:** 这一行为 `add` 的声明或调用提供内容。
- **EN L971:** This line contributes implementation detail or declarative structure to the file.
  **CN L971:** 这一行为文件补充了实现细节或声明式结构。
- **EN L972:** This line contributes implementation detail or declarative structure to the file.
  **CN L972:** 这一行为文件补充了实现细节或声明式结构。

### Lines 973-984 / 第 973-984 行

```tablegen
 973: 
 974:     Example:
 975:     ```mlir
 976:     %r0 = }] # mnemonic # [{ %a0, %b0, %c0, 0, 0, 0 : (}] # ABType.typeName
 977:     # [{, }] # ABType.typeName # [{, }] # CDType.typeName # [{) -> }]
 978:     # CDType.typeName # [{
 979:     ```}];
 980: }
 981: 
 982: class ROCDL_Mfma_Scale_IntrOp<string mnemonic, Type AB, Type CD> :
 983:   ROCDL_IntrOp<mnemonic, [], [0, 1], [], 1, 0, 0, 0, [3, 4, 5, 7], ["cbsz", "blgp", "opselA", "opselB"]>,
 984:   Arguments<(ins
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L973:** Blank line used to separate nearby declarations and improve readability.
  **CN L973:** 该空行用于分隔相邻声明并提升可读性。
- **EN L974:** This line contributes implementation detail or declarative structure to the file.
  **CN L974:** 这一行为文件补充了实现细节或声明式结构。
- **EN L975:** This line contributes implementation detail or declarative structure to the file.
  **CN L975:** 这一行为文件补充了实现细节或声明式结构。
- **EN L976:** This line contributes implementation detail or declarative structure to the file.
  **CN L976:** 这一行为文件补充了实现细节或声明式结构。
- **EN L977:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L977:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L978:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L978:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L979:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L979:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L980:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L980:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L981:** Blank line used to separate nearby declarations and improve readability.
  **CN L981:** 该空行用于分隔相邻声明并提升可读性。
- **EN L982:** This TableGen `class` record introduces `ROCDL_Mfma_Scale_IntrOp`, which later participates in generated MLIR code.
  **CN L982:** 该 TableGen `class` 记录引入了 `ROCDL_Mfma_Scale_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L983:** This line contributes implementation detail or declarative structure to the file.
  **CN L983:** 这一行为文件补充了实现细节或声明式结构。
- **EN L984:** This line contributes implementation detail or declarative structure to the file.
  **CN L984:** 这一行为文件补充了实现细节或声明式结构。

### Lines 985-996 / 第 985-996 行

```tablegen
 985:              LLVM_VectorOf<AB>:$a,
 986:              LLVM_VectorOf<AB>:$b,
 987:              LLVM_VectorOf<CD>:$c,
 988:              I32Attr:$cbsz,
 989:              I32Attr:$blgp,
 990:              I32Attr:$opselA,
 991:              I32:$scaleA,
 992:              I32Attr:$opselB,
 993:              I32:$scaleB)> {
 994:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
 995:   let assemblyFormat = [{
 996:     $a `,` $b `,` $c `,` $cbsz `,` $blgp `,` $opselA `,` $scaleA `,` $opselB `,` $scaleB attr-dict `:` functional-type(operands, $res)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L985:** This line contributes implementation detail or declarative structure to the file.
  **CN L985:** 这一行为文件补充了实现细节或声明式结构。
- **EN L986:** This line contributes implementation detail or declarative structure to the file.
  **CN L986:** 这一行为文件补充了实现细节或声明式结构。
- **EN L987:** This line contributes implementation detail or declarative structure to the file.
  **CN L987:** 这一行为文件补充了实现细节或声明式结构。
- **EN L988:** This line contributes implementation detail or declarative structure to the file.
  **CN L988:** 这一行为文件补充了实现细节或声明式结构。
- **EN L989:** This line contributes implementation detail or declarative structure to the file.
  **CN L989:** 这一行为文件补充了实现细节或声明式结构。
- **EN L990:** This line contributes implementation detail or declarative structure to the file.
  **CN L990:** 这一行为文件补充了实现细节或声明式结构。
- **EN L991:** This line contributes implementation detail or declarative structure to the file.
  **CN L991:** 这一行为文件补充了实现细节或声明式结构。
- **EN L992:** This line contributes implementation detail or declarative structure to the file.
  **CN L992:** 这一行为文件补充了实现细节或声明式结构。
- **EN L993:** This line contributes implementation detail or declarative structure to the file.
  **CN L993:** 这一行为文件补充了实现细节或声明式结构。
- **EN L994:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L994:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L995:** This line contributes implementation detail or declarative structure to the file.
  **CN L995:** 这一行为文件补充了实现细节或声明式结构。
- **EN L996:** This line contributes to the declaration or call of `type`.
  **CN L996:** 这一行为 `type` 的声明或调用提供内容。

### Lines 997-1008 / 第 997-1008 行

```tablegen
 997:   }];
 998:   let description = [{
 999:     Scaled matrix fused multiply-add (MFMA) intrinsic with per-operand scaling.
1000:     The `opselA`/`opselB` and `scaleA`/`scaleB` arguments control the scaling
1001:     of input operands.
1002: 
1003:     Example:
1004:     ```mlir
1005:     // Scaled MFMA with fp8 * fp8 inputs.
1006:     %r0 = rocdl.mfma.scale.f32.32x32x64.f8f6f4 %a, %a, %c, 0, 0, 0, %scaleA, 0, %scaleB :
1007:       (vector<8xi32>, vector<8xi32>, vector<16xf32>, i32, i32) -> vector<16xf32>
1008: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L997:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L997:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L998:** This line contributes implementation detail or declarative structure to the file.
  **CN L998:** 这一行为文件补充了实现细节或声明式结构。
- **EN L999:** This line contributes to the declaration or call of `add`.
  **CN L999:** 这一行为 `add` 的声明或调用提供内容。
- **EN L1000:** This line contributes implementation detail or declarative structure to the file.
  **CN L1000:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1001:** This line contributes implementation detail or declarative structure to the file.
  **CN L1001:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1002:** Blank line used to separate nearby declarations and improve readability.
  **CN L1002:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1003:** This line contributes implementation detail or declarative structure to the file.
  **CN L1003:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1004:** This line contributes implementation detail or declarative structure to the file.
  **CN L1004:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1005:** This comment states: “Scaled MFMA with fp8 * fp8 inputs.”, documenting the intent of the surrounding code.
  **CN L1005:** 该注释写道：“Scaled MFMA with fp8 * fp8 inputs.”，用于说明周围代码的意图。
- **EN L1006:** This line contributes implementation detail or declarative structure to the file.
  **CN L1006:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1007:** This line contributes implementation detail or declarative structure to the file.
  **CN L1007:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1008:** Blank line used to separate nearby declarations and improve readability.
  **CN L1008:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1009-1020 / 第 1009-1020 行

```tablegen
1009:     // Scaled MFMA with fp8 * bf8 inputs.
1010:     %r1 = rocdl.mfma.scale.f32.32x32x64.f8f6f4 %a, %a, %c, 0, 1, 0, %scaleA, 0, %scaleB :
1011:       (vector<8xi32>, vector<8xi32>, vector<16xf32>, i32, i32) -> vector<16xf32>
1012: 
1013:     // Scaled MFMA with fp8 * fp6 inputs (6xi32 operand B).
1014:     %r2 = rocdl.mfma.scale.f32.32x32x64.f8f6f4 %a, %b6, %c, 0, 2, 0, %scaleA, 0, %scaleB :
1015:       (vector<8xi32>, vector<6xi32>, vector<16xf32>, i32, i32) -> vector<16xf32>
1016:     ```
1017:   }];
1018: }
1019: 
1020: class ROCDL_Smfmac_IntrOp<string mnemonic, Type AType, Type BType, Type CDType> :
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1009:** This comment states: “Scaled MFMA with fp8 * bf8 inputs.”, documenting the intent of the surrounding code.
  **CN L1009:** 该注释写道：“Scaled MFMA with fp8 * bf8 inputs.”，用于说明周围代码的意图。
- **EN L1010:** This line contributes implementation detail or declarative structure to the file.
  **CN L1010:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1011:** This line contributes implementation detail or declarative structure to the file.
  **CN L1011:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1012:** Blank line used to separate nearby declarations and improve readability.
  **CN L1012:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1013:** This comment states: “Scaled MFMA with fp8 * fp6 inputs (6xi32 operand B).”, documenting the intent of the surrounding code.
  **CN L1013:** 该注释写道：“Scaled MFMA with fp8 * fp6 inputs (6xi32 operand B).”，用于说明周围代码的意图。
- **EN L1014:** This line contributes implementation detail or declarative structure to the file.
  **CN L1014:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1015:** This line contributes implementation detail or declarative structure to the file.
  **CN L1015:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1016:** This line contributes implementation detail or declarative structure to the file.
  **CN L1016:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1017:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1017:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1018:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1018:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1019:** Blank line used to separate nearby declarations and improve readability.
  **CN L1019:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1020:** This TableGen `class` record introduces `ROCDL_Smfmac_IntrOp`, which later participates in generated MLIR code.
  **CN L1020:** 该 TableGen `class` 记录引入了 `ROCDL_Smfmac_IntrOp`，后续会参与生成的 MLIR 代码。

### Lines 1021-1032 / 第 1021-1032 行

```tablegen
1021:   ROCDL_IntrOp<mnemonic, [], [], [], 1, 0, 0, 0, [4, 5], ["cbsz", "abid"]>,
1022:   Arguments<(ins
1023:              AType:$a,
1024:              BType:$b,
1025:              CDType:$c,
1026:              I32:$index,
1027:              I32Attr:$cbsz,
1028:              I32Attr:$abid)> {
1029:   let results = (outs CDType:$res);
1030:   let assemblyFormat = [{
1031:     $a `,` $b `,` $c `,` $index `,` $cbsz `,` $abid attr-dict `:` functional-type(operands, $res)
1032:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1021:** This line contributes implementation detail or declarative structure to the file.
  **CN L1021:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1022:** This line contributes implementation detail or declarative structure to the file.
  **CN L1022:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1023:** This line contributes implementation detail or declarative structure to the file.
  **CN L1023:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1024:** This line contributes implementation detail or declarative structure to the file.
  **CN L1024:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1025:** This line contributes implementation detail or declarative structure to the file.
  **CN L1025:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1026:** This line contributes implementation detail or declarative structure to the file.
  **CN L1026:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1027:** This line contributes implementation detail or declarative structure to the file.
  **CN L1027:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1028:** This line contributes implementation detail or declarative structure to the file.
  **CN L1028:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1029:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1029:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1030:** This line contributes implementation detail or declarative structure to the file.
  **CN L1030:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1031:** This line contributes to the declaration or call of `type`.
  **CN L1031:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1032:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1032:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1033-1044 / 第 1033-1044 行

```tablegen
1033:   let description = [{
1034:     Sparse matrix fused multiply-accumulate (SMFMAC) intrinsic with 2:4
1035:     structured sparsity. The `index` operand provides the sparsity metadata,
1036:     and `cbsz`/`abid` control broadcast modes.
1037: 
1038:     Example:
1039:     ```mlir
1040:     // SMFMAC with f16 inputs.
1041:     %r0 = rocdl.smfmac.f32.16x16x32.f16 %a0, %b0, %c0, %idx, 0, 0 :
1042:       (vector<4xf16>, vector<8xf16>, vector<4xf32>, i32) -> vector<4xf32>
1043: 
1044:     // SMFMAC with bf16 inputs.
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN L1033:** This line contributes implementation detail or declarative structure to the file.
  **CN L1033:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1034:** This line contributes to the declaration or call of `accumulate`.
  **CN L1034:** 这一行为 `accumulate` 的声明或调用提供内容。
- **EN L1035:** This line contributes implementation detail or declarative structure to the file.
  **CN L1035:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1036:** This line contributes implementation detail or declarative structure to the file.
  **CN L1036:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1037:** Blank line used to separate nearby declarations and improve readability.
  **CN L1037:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1038:** This line contributes implementation detail or declarative structure to the file.
  **CN L1038:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1039:** This line contributes implementation detail or declarative structure to the file.
  **CN L1039:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1040:** This comment states: “SMFMAC with f16 inputs.”, documenting the intent of the surrounding code.
  **CN L1040:** 该注释写道：“SMFMAC with f16 inputs.”，用于说明周围代码的意图。
- **EN L1041:** This line contributes implementation detail or declarative structure to the file.
  **CN L1041:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1042:** This line contributes implementation detail or declarative structure to the file.
  **CN L1042:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1043:** Blank line used to separate nearby declarations and improve readability.
  **CN L1043:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1044:** This comment states: “SMFMAC with bf16 inputs.”, documenting the intent of the surrounding code.
  **CN L1044:** 该注释写道：“SMFMAC with bf16 inputs.”，用于说明周围代码的意图。

### Lines 1045-1056 / 第 1045-1056 行

```tablegen
1045:     %r1 = rocdl.smfmac.f32.16x16x32.bf16 %a1, %b1, %c0, %idx, 0, 0 :
1046:       (vector<4xi16>, vector<8xi16>, vector<4xf32>, i32) -> vector<4xf32>
1047: 
1048:     // SMFMAC with i8 inputs and i32 accumulator.
1049:     %r2 = rocdl.smfmac.i32.16x16x64.i8 %a2, %b2, %c2, %idx, 0, 0 :
1050:       (vector<2xi32>, vector<4xi32>, vector<4xi32>, i32) -> vector<4xi32>
1051: 
1052:     // SMFMAC with fp8 inputs.
1053:     %r3 = rocdl.smfmac.f32.16x16x64.fp8.fp8 %a2, %b2, %c0, %idx, 0, 0 :
1054:       (vector<2xi32>, vector<4xi32>, vector<4xf32>, i32) -> vector<4xf32>
1055:     ```
1056:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1045:** This line contributes implementation detail or declarative structure to the file.
  **CN L1045:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1046:** This line contributes implementation detail or declarative structure to the file.
  **CN L1046:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1047:** Blank line used to separate nearby declarations and improve readability.
  **CN L1047:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1048:** This comment states: “SMFMAC with i8 inputs and i32 accumulator.”, documenting the intent of the surrounding code.
  **CN L1048:** 该注释写道：“SMFMAC with i8 inputs and i32 accumulator.”，用于说明周围代码的意图。
- **EN L1049:** This line contributes implementation detail or declarative structure to the file.
  **CN L1049:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1050:** This line contributes implementation detail or declarative structure to the file.
  **CN L1050:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1051:** Blank line used to separate nearby declarations and improve readability.
  **CN L1051:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1052:** This comment states: “SMFMAC with fp8 inputs.”, documenting the intent of the surrounding code.
  **CN L1052:** 该注释写道：“SMFMAC with fp8 inputs.”，用于说明周围代码的意图。
- **EN L1053:** This line contributes implementation detail or declarative structure to the file.
  **CN L1053:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1054:** This line contributes implementation detail or declarative structure to the file.
  **CN L1054:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1055:** This line contributes implementation detail or declarative structure to the file.
  **CN L1055:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1056:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1056:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1057-1068 / 第 1057-1068 行

```tablegen
1057: }
1058: 
1059: // Available on all CDNA.
1060: def ROCDL_mfma_f32_32x32x1f32 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x1f32", /*Type AB=*/ROCDL_Scalar<F32>, /*Type CD=*/ROCDL_ConcreteVector<F32, 32>>;
1061: def ROCDL_mfma_f32_16x16x1f32 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x1f32", ROCDL_Scalar<F32>, ROCDL_ConcreteVector<F32, 16>>;
1062: def ROCDL_mfma_f32_4x4x1f32 : ROCDL_Mfma_IntrOp<"mfma.f32.4x4x1f32", ROCDL_Scalar<F32>, ROCDL_ConcreteVector<F32, 4>>;
1063: def ROCDL_mfma_f32_32x32x2f32 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x2f32", ROCDL_Scalar<F32>, ROCDL_ConcreteVector<F32, 16>>;
1064: def ROCDL_mfma_f32_16x16x4f32 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x4f32", ROCDL_Scalar<F32>, ROCDL_ConcreteVector<F32, 4>>;
1065: def ROCDL_mfma_f32_32x32x4f16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x4f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F32, 32>>;
1066: def ROCDL_mfma_f32_16x16x4f16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x4f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F32, 16>>;
1067: def ROCDL_mfma_f32_4x4x4f16 : ROCDL_Mfma_IntrOp<"mfma.f32.4x4x4f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F32, 4>>;
1068: def ROCDL_mfma_f32_32x32x8f16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x8f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F32, 16>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1057:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1057:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1058:** Blank line used to separate nearby declarations and improve readability.
  **CN L1058:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1059:** This comment states: “Available on all CDNA.”, documenting the intent of the surrounding code.
  **CN L1059:** 该注释写道：“Available on all CDNA.”，用于说明周围代码的意图。
- **EN L1060:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x1f32`, which later participates in generated MLIR code.
  **CN L1060:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x1f32`，后续会参与生成的 MLIR 代码。
- **EN L1061:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x1f32`, which later participates in generated MLIR code.
  **CN L1061:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x1f32`，后续会参与生成的 MLIR 代码。
- **EN L1062:** This TableGen `def` record introduces `ROCDL_mfma_f32_4x4x1f32`, which later participates in generated MLIR code.
  **CN L1062:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_4x4x1f32`，后续会参与生成的 MLIR 代码。
- **EN L1063:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x2f32`, which later participates in generated MLIR code.
  **CN L1063:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x2f32`，后续会参与生成的 MLIR 代码。
- **EN L1064:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x4f32`, which later participates in generated MLIR code.
  **CN L1064:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x4f32`，后续会参与生成的 MLIR 代码。
- **EN L1065:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x4f16`, which later participates in generated MLIR code.
  **CN L1065:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x4f16`，后续会参与生成的 MLIR 代码。
- **EN L1066:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x4f16`, which later participates in generated MLIR code.
  **CN L1066:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x4f16`，后续会参与生成的 MLIR 代码。
- **EN L1067:** This TableGen `def` record introduces `ROCDL_mfma_f32_4x4x4f16`, which later participates in generated MLIR code.
  **CN L1067:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_4x4x4f16`，后续会参与生成的 MLIR 代码。
- **EN L1068:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x8f16`, which later participates in generated MLIR code.
  **CN L1068:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x8f16`，后续会参与生成的 MLIR 代码。

### Lines 1069-1080 / 第 1069-1080 行

```tablegen
1069: def ROCDL_mfma_f32_16x16x16f16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x16f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F32, 4>>;
1070: def ROCDL_mfma_i32_32x32x4i8 : ROCDL_Mfma_IntrOp<"mfma.i32.32x32x4i8", ROCDL_Scalar<I32>, ROCDL_ConcreteVector<I32, 32>>;
1071: def ROCDL_mfma_i32_16x16x4i8 : ROCDL_Mfma_IntrOp<"mfma.i32.16x16x4i8", ROCDL_Scalar<I32>, ROCDL_ConcreteVector<I32, 16>>;
1072: def ROCDL_mfma_i32_4x4x4i8 : ROCDL_Mfma_IntrOp<"mfma.i32.4x4x4i8", ROCDL_Scalar<I32>, ROCDL_ConcreteVector<I32, 4>>;
1073: def ROCDL_mfma_i32_32x32x8i8 : ROCDL_Mfma_IntrOp<"mfma.i32.32x32x8i8", ROCDL_Scalar<I32>, ROCDL_ConcreteVector<I32, 16>>;
1074: def ROCDL_mfma_i32_16x16x16i8 : ROCDL_Mfma_IntrOp<"mfma.i32.16x16x16i8", ROCDL_Scalar<I32>, ROCDL_ConcreteVector<I32, 4>>;
1075: def ROCDL_mfma_f32_32x32x2bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x2bf16", ROCDL_ConcreteVector<I16, 2>, ROCDL_ConcreteVector<F32, 32>>;
1076: def ROCDL_mfma_f32_16x16x2bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x2bf16", ROCDL_ConcreteVector<I16, 2>, ROCDL_ConcreteVector<F32, 16>>;
1077: def ROCDL_mfma_f32_4x4x2bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.4x4x2bf16", ROCDL_ConcreteVector<I16, 2>, ROCDL_ConcreteVector<F32, 4>>;
1078: def ROCDL_mfma_f32_32x32x4bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x4bf16", ROCDL_ConcreteVector<I16, 2>, ROCDL_ConcreteVector<F32, 16>>;
1079: def ROCDL_mfma_f32_16x16x8bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x8bf16", ROCDL_ConcreteVector<I16, 2>, ROCDL_ConcreteVector<F32, 4>>;
1080: // New in gfx90a.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1069:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x16f16`, which later participates in generated MLIR code.
  **CN L1069:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x16f16`，后续会参与生成的 MLIR 代码。
- **EN L1070:** This TableGen `def` record introduces `ROCDL_mfma_i32_32x32x4i8`, which later participates in generated MLIR code.
  **CN L1070:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_32x32x4i8`，后续会参与生成的 MLIR 代码。
- **EN L1071:** This TableGen `def` record introduces `ROCDL_mfma_i32_16x16x4i8`, which later participates in generated MLIR code.
  **CN L1071:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_16x16x4i8`，后续会参与生成的 MLIR 代码。
- **EN L1072:** This TableGen `def` record introduces `ROCDL_mfma_i32_4x4x4i8`, which later participates in generated MLIR code.
  **CN L1072:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_4x4x4i8`，后续会参与生成的 MLIR 代码。
- **EN L1073:** This TableGen `def` record introduces `ROCDL_mfma_i32_32x32x8i8`, which later participates in generated MLIR code.
  **CN L1073:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_32x32x8i8`，后续会参与生成的 MLIR 代码。
- **EN L1074:** This TableGen `def` record introduces `ROCDL_mfma_i32_16x16x16i8`, which later participates in generated MLIR code.
  **CN L1074:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_16x16x16i8`，后续会参与生成的 MLIR 代码。
- **EN L1075:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x2bf16`, which later participates in generated MLIR code.
  **CN L1075:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x2bf16`，后续会参与生成的 MLIR 代码。
- **EN L1076:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x2bf16`, which later participates in generated MLIR code.
  **CN L1076:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x2bf16`，后续会参与生成的 MLIR 代码。
- **EN L1077:** This TableGen `def` record introduces `ROCDL_mfma_f32_4x4x2bf16`, which later participates in generated MLIR code.
  **CN L1077:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_4x4x2bf16`，后续会参与生成的 MLIR 代码。
- **EN L1078:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x4bf16`, which later participates in generated MLIR code.
  **CN L1078:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x4bf16`，后续会参与生成的 MLIR 代码。
- **EN L1079:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x8bf16`, which later participates in generated MLIR code.
  **CN L1079:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x8bf16`，后续会参与生成的 MLIR 代码。
- **EN L1080:** This comment states: “New in gfx90a.”, documenting the intent of the surrounding code.
  **CN L1080:** 该注释写道：“New in gfx90a.”，用于说明周围代码的意图。

### Lines 1081-1092 / 第 1081-1092 行

```tablegen
1081: def ROCDL_mfma_f32_32x32x4bf16_1k : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x4bf16.1k", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<F32, 32>>;
1082: def ROCDL_mfma_f32_16x16x4bf16_1k : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x4bf16.1k", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<F32, 16>>;
1083: def ROCDL_mfma_f32_4x4x4bf16_1k : ROCDL_Mfma_IntrOp<"mfma.f32.4x4x4bf16.1k", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<F32, 4>>;
1084: def ROCDL_mfma_f32_32x32x8bf16_1k : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x8bf16.1k", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<F32, 16>>;
1085: def ROCDL_mfma_f32_16x16x16bf16_1k : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x16bf16.1k", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<F32, 4>>;
1086: // Note: in gfx94x, unlike in gfx90a, the f64 xdlops use the "blgp" argument as
1087: // a NEG bitfield. See IntrinsicsAMDGPU.td for more info.
1088: def ROCDL_mfma_f64_16x16x4f64 : ROCDL_Mfma_IntrOp<"mfma.f64.16x16x4f64", ROCDL_Scalar<F64>, ROCDL_ConcreteVector<F64, 4>>;
1089: def ROCDL_mfma_f64_4x4x4f64 : ROCDL_Mfma_IntrOp<"mfma.f64.4x4x4f64", ROCDL_Scalar<F64>, ROCDL_Scalar<F64>>;
1090: // New in gfx94x.
1091: def ROCDL_mfma_i32_16x16x32_i8 : ROCDL_Mfma_IntrOp<"mfma.i32.16x16x32.i8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<I32, 4>>;
1092: def ROCDL_mfma_i32_32x32x16_i8 : ROCDL_Mfma_IntrOp<"mfma.i32.32x32x16.i8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<I32, 16>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1081:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x4bf16_1k`, which later participates in generated MLIR code.
  **CN L1081:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x4bf16_1k`，后续会参与生成的 MLIR 代码。
- **EN L1082:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x4bf16_1k`, which later participates in generated MLIR code.
  **CN L1082:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x4bf16_1k`，后续会参与生成的 MLIR 代码。
- **EN L1083:** This TableGen `def` record introduces `ROCDL_mfma_f32_4x4x4bf16_1k`, which later participates in generated MLIR code.
  **CN L1083:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_4x4x4bf16_1k`，后续会参与生成的 MLIR 代码。
- **EN L1084:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x8bf16_1k`, which later participates in generated MLIR code.
  **CN L1084:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x8bf16_1k`，后续会参与生成的 MLIR 代码。
- **EN L1085:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x16bf16_1k`, which later participates in generated MLIR code.
  **CN L1085:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x16bf16_1k`，后续会参与生成的 MLIR 代码。
- **EN L1086:** This comment states: “Note: in gfx94x, unlike in gfx90a, the f64 xdlops use the "blgp" argument as”, documenting the intent of the surrounding code.
  **CN L1086:** 该注释写道：“Note: in gfx94x, unlike in gfx90a, the f64 xdlops use the "blgp" argument as”，用于说明周围代码的意图。
- **EN L1087:** This comment states: “a NEG bitfield. See IntrinsicsAMDGPU.td for more info.”, documenting the intent of the surrounding code.
  **CN L1087:** 该注释写道：“a NEG bitfield. See IntrinsicsAMDGPU.td for more info.”，用于说明周围代码的意图。
- **EN L1088:** This TableGen `def` record introduces `ROCDL_mfma_f64_16x16x4f64`, which later participates in generated MLIR code.
  **CN L1088:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f64_16x16x4f64`，后续会参与生成的 MLIR 代码。
- **EN L1089:** This TableGen `def` record introduces `ROCDL_mfma_f64_4x4x4f64`, which later participates in generated MLIR code.
  **CN L1089:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f64_4x4x4f64`，后续会参与生成的 MLIR 代码。
- **EN L1090:** This comment states: “New in gfx94x.”, documenting the intent of the surrounding code.
  **CN L1090:** 该注释写道：“New in gfx94x.”，用于说明周围代码的意图。
- **EN L1091:** This TableGen `def` record introduces `ROCDL_mfma_i32_16x16x32_i8`, which later participates in generated MLIR code.
  **CN L1091:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_16x16x32_i8`，后续会参与生成的 MLIR 代码。
- **EN L1092:** This TableGen `def` record introduces `ROCDL_mfma_i32_32x32x16_i8`, which later participates in generated MLIR code.
  **CN L1092:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_32x32x16_i8`，后续会参与生成的 MLIR 代码。

### Lines 1093-1104 / 第 1093-1104 行

```tablegen
1093: def ROCDL_mfma_f32_16x16x8_xf32 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x8.xf32", ROCDL_ConcreteVector<F32, 2>, ROCDL_ConcreteVector<F32, 4>>;
1094: def ROCDL_mfma_f32_32x32x4_xf32 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x4.xf32", ROCDL_ConcreteVector<F32, 2>, ROCDL_ConcreteVector<F32, 16>>;
1095: def ROCDL_mfma_f32_16x16x32_bf8_bf8 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.bf8.bf8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 4>>;
1096: def ROCDL_mfma_f32_16x16x32_bf8_fp8 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.bf8.fp8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 4>>;
1097: def ROCDL_mfma_f32_16x16x32_fp8_bf8 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.fp8.bf8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 4>>;
1098: def ROCDL_mfma_f32_16x16x32_fp8_fp8 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.fp8.fp8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 4>>;
1099: def ROCDL_mfma_f32_32x32x16_bf8_bf8 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.bf8.bf8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 16>>;
1100: def ROCDL_mfma_f32_32x32x16_bf8_fp8 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.bf8.fp8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 16>>;
1101: def ROCDL_mfma_f32_32x32x16_fp8_bf8 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.fp8.bf8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 16>>;
1102: def ROCDL_mfma_f32_32x32x16_fp8_fp8 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.fp8.fp8", ROCDL_Scalar<I64>, ROCDL_ConcreteVector<F32, 16>>;
1103: // New in gfx950.
1104: def ROCDL_mfma_f32_16x16x32_bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.bf16", ROCDL_ConcreteVector<BF16, 8>, ROCDL_ConcreteVector<F32, 4>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1093:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x8_xf32`, which later participates in generated MLIR code.
  **CN L1093:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x8_xf32`，后续会参与生成的 MLIR 代码。
- **EN L1094:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x4_xf32`, which later participates in generated MLIR code.
  **CN L1094:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x4_xf32`，后续会参与生成的 MLIR 代码。
- **EN L1095:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1095:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1096:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1096:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1097:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1097:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1098:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1098:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1099:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1099:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1100:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1100:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1101:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1101:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1102:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1102:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1103:** This comment states: “New in gfx950.”, documenting the intent of the surrounding code.
  **CN L1103:** 该注释写道：“New in gfx950.”，用于说明周围代码的意图。
- **EN L1104:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1104:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_bf16`，后续会参与生成的 MLIR 代码。

### Lines 1105-1116 / 第 1105-1116 行

```tablegen
1105: def ROCDL_mfma_i32_16x16x64_i8 : ROCDL_Mfma_IntrOp<"mfma.i32.16x16x64.i8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 4>>;
1106: def ROCDL_mfma_f32_16x16x32_f16 : ROCDL_Mfma_IntrOp<"mfma.f32.16x16x32.f16", ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F32, 4>>;
1107: def ROCDL_mfma_f32_32x32x16_bf16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.bf16", ROCDL_ConcreteVector<BF16, 8>, ROCDL_ConcreteVector<F32, 16>>;
1108: def ROCDL_mfma_i32_32x32x32_i8 : ROCDL_Mfma_IntrOp<"mfma.i32.32x32x32.i8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 16>>;
1109: def ROCDL_mfma_f32_32x32x16_f16 : ROCDL_Mfma_IntrOp<"mfma.f32.32x32x16.f16", ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F32, 16>>;
1110: 
1111: def ROCDL_mfma_scale_f32_16x16x128_f8f6f4 : ROCDL_Mfma_Scale_IntrOp<"mfma.scale.f32.16x16x128.f8f6f4", I32, F32>;
1112: def ROCDL_mfma_scale_f32_32x32x64_f8f6f4 : ROCDL_Mfma_Scale_IntrOp<"mfma.scale.f32.32x32x64.f8f6f4", I32, F32>;
1113: 
1114: // 2:4 Sparsity ops (GFX94x)
1115: def ROCDL_smfmac_f32_16x16x32_f16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x32.f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F32, 4>>;
1116: def ROCDL_smfmac_f32_32x32x16_f16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x16.f16", ROCDL_ConcreteVector<F16, 4>, ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F32, 16>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1105:** This TableGen `def` record introduces `ROCDL_mfma_i32_16x16x64_i8`, which later participates in generated MLIR code.
  **CN L1105:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_16x16x64_i8`，后续会参与生成的 MLIR 代码。
- **EN L1106:** This TableGen `def` record introduces `ROCDL_mfma_f32_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1106:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_16x16x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1107:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_bf16`, which later participates in generated MLIR code.
  **CN L1107:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1108:** This TableGen `def` record introduces `ROCDL_mfma_i32_32x32x32_i8`, which later participates in generated MLIR code.
  **CN L1108:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_i32_32x32x32_i8`，后续会参与生成的 MLIR 代码。
- **EN L1109:** This TableGen `def` record introduces `ROCDL_mfma_f32_32x32x16_f16`, which later participates in generated MLIR code.
  **CN L1109:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_f32_32x32x16_f16`，后续会参与生成的 MLIR 代码。
- **EN L1110:** Blank line used to separate nearby declarations and improve readability.
  **CN L1110:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1111:** This TableGen `def` record introduces `ROCDL_mfma_scale_f32_16x16x128_f8f6f4`, which later participates in generated MLIR code.
  **CN L1111:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_scale_f32_16x16x128_f8f6f4`，后续会参与生成的 MLIR 代码。
- **EN L1112:** This TableGen `def` record introduces `ROCDL_mfma_scale_f32_32x32x64_f8f6f4`, which later participates in generated MLIR code.
  **CN L1112:** 该 TableGen `def` 记录引入了 `ROCDL_mfma_scale_f32_32x32x64_f8f6f4`，后续会参与生成的 MLIR 代码。
- **EN L1113:** Blank line used to separate nearby declarations and improve readability.
  **CN L1113:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1114:** This comment states: “2:4 Sparsity ops (GFX94x)”, documenting the intent of the surrounding code.
  **CN L1114:** 该注释写道：“2:4 Sparsity ops (GFX94x)”，用于说明周围代码的意图。
- **EN L1115:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1115:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1116:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x16_f16`, which later participates in generated MLIR code.
  **CN L1116:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x16_f16`，后续会参与生成的 MLIR 代码。

### Lines 1117-1128 / 第 1117-1128 行

```tablegen
1117: def ROCDL_smfmac_f32_16x16x32_bf16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x32.bf16", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<I16, 8>, ROCDL_ConcreteVector<F32, 4>>;
1118: def ROCDL_smfmac_f32_32x32x16_bf16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x16.bf16", ROCDL_ConcreteVector<I16, 4>, ROCDL_ConcreteVector<I16, 8>, ROCDL_ConcreteVector<F32, 16>>;
1119: def ROCDL_smfmac_i32_16x16x64_i8 : ROCDL_Smfmac_IntrOp<"smfmac.i32.16x16x64.i8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 4>>;
1120: def ROCDL_smfmac_i32_32x32x32_i8 : ROCDL_Smfmac_IntrOp<"smfmac.i32.32x32x32.i8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 16>>;
1121: def ROCDL_smfmac_f32_16x16x64_bf8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.bf8.bf8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 4>>;
1122: def ROCDL_smfmac_f32_16x16x64_bf8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.bf8.fp8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 4>>;
1123: def ROCDL_smfmac_f32_16x16x64_fp8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.fp8.bf8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 4>>;
1124: def ROCDL_smfmac_f32_16x16x64_fp8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.fp8.fp8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 4>>;
1125: def ROCDL_smfmac_f32_32x32x32_bf8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.bf8.bf8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 16>>;
1126: def ROCDL_smfmac_f32_32x32x32_bf8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.bf8.fp8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 16>>;
1127: def ROCDL_smfmac_f32_32x32x32_fp8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.fp8.bf8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 16>>;
1128: def ROCDL_smfmac_f32_32x32x32_fp8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.fp8.fp8", ROCDL_ConcreteVector<I32, 2>, ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<F32, 16>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1117:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1117:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1118:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x16_bf16`, which later participates in generated MLIR code.
  **CN L1118:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x16_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1119:** This TableGen `def` record introduces `ROCDL_smfmac_i32_16x16x64_i8`, which later participates in generated MLIR code.
  **CN L1119:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_i32_16x16x64_i8`，后续会参与生成的 MLIR 代码。
- **EN L1120:** This TableGen `def` record introduces `ROCDL_smfmac_i32_32x32x32_i8`, which later participates in generated MLIR code.
  **CN L1120:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_i32_32x32x32_i8`，后续会参与生成的 MLIR 代码。
- **EN L1121:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1121:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1122:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1122:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1123:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1123:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1124:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1124:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1125:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1125:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1126:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1126:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1127:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1127:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1128:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1128:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_fp8_fp8`，后续会参与生成的 MLIR 代码。

### Lines 1129-1140 / 第 1129-1140 行

```tablegen
1129: // New in gfx950.
1130: def ROCDL_smfmac_f32_16x16x64_bf16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.bf16", ROCDL_ConcreteVector<BF16, 8>, ROCDL_ConcreteVector<BF16, 16>, ROCDL_ConcreteVector<F32, 4>>;
1131: def ROCDL_smfmac_f32_16x16x64_f16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x64.f16", ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F16, 16>, ROCDL_ConcreteVector<F32, 4>>;
1132: def ROCDL_smfmac_i32_16x16x128_i8 : ROCDL_Smfmac_IntrOp<"smfmac.i32.16x16x128.i8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<I32, 4>>;
1133: def ROCDL_smfmac_f32_16x16x128_bf8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x128.bf8.bf8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 4>>;
1134: def ROCDL_smfmac_f32_16x16x128_bf8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x128.bf8.fp8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 4>>;
1135: def ROCDL_smfmac_f32_16x16x128_fp8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x128.fp8.bf8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 4>>;
1136: def ROCDL_smfmac_f32_16x16x128_fp8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.16x16x128.fp8.fp8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 4>>;
1137: def ROCDL_smfmac_f32_32x32x32_bf16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.bf16", ROCDL_ConcreteVector<BF16, 8>, ROCDL_ConcreteVector<BF16, 16>, ROCDL_ConcreteVector<F32, 16>>;
1138: def ROCDL_smfmac_f32_32x32x32_f16 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x32.f16", ROCDL_ConcreteVector<F16, 8>, ROCDL_ConcreteVector<F16, 16>, ROCDL_ConcreteVector<F32, 16>>;
1139: def ROCDL_smfmac_i32_32x32x64_i8 : ROCDL_Smfmac_IntrOp<"smfmac.i32.32x32x64.i8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<I32, 16>>;
1140: def ROCDL_smfmac_f32_32x32x64_bf8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x64.bf8.bf8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 16>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1129:** This comment states: “New in gfx950.”, documenting the intent of the surrounding code.
  **CN L1129:** 该注释写道：“New in gfx950.”，用于说明周围代码的意图。
- **EN L1130:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_bf16`, which later participates in generated MLIR code.
  **CN L1130:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1131:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x64_f16`, which later participates in generated MLIR code.
  **CN L1131:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x64_f16`，后续会参与生成的 MLIR 代码。
- **EN L1132:** This TableGen `def` record introduces `ROCDL_smfmac_i32_16x16x128_i8`, which later participates in generated MLIR code.
  **CN L1132:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_i32_16x16x128_i8`，后续会参与生成的 MLIR 代码。
- **EN L1133:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x128_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1133:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x128_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1134:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x128_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1134:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x128_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1135:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x128_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1135:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x128_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1136:** This TableGen `def` record introduces `ROCDL_smfmac_f32_16x16x128_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1136:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_16x16x128_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1137:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_bf16`, which later participates in generated MLIR code.
  **CN L1137:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1138:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x32_f16`, which later participates in generated MLIR code.
  **CN L1138:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1139:** This TableGen `def` record introduces `ROCDL_smfmac_i32_32x32x64_i8`, which later participates in generated MLIR code.
  **CN L1139:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_i32_32x32x64_i8`，后续会参与生成的 MLIR 代码。
- **EN L1140:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x64_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1140:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x64_bf8_bf8`，后续会参与生成的 MLIR 代码。

### Lines 1141-1152 / 第 1141-1152 行

```tablegen
1141: def ROCDL_smfmac_f32_32x32x64_bf8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x64.bf8.fp8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 16>>;
1142: def ROCDL_smfmac_f32_32x32x64_fp8_bf8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x64.fp8.bf8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 16>>;
1143: def ROCDL_smfmac_f32_32x32x64_fp8_fp8 : ROCDL_Smfmac_IntrOp<"smfmac.f32.32x32x64.fp8.fp8", ROCDL_ConcreteVector<I32, 4>, ROCDL_ConcreteVector<I32, 8>, ROCDL_ConcreteVector<F32, 16>>;
1144: 
1145: 
1146: //===---------------------------------------------------------------------===//
1147: // WMMA intrinsics
1148: class ROCDL_WMMA_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1149:     [0], [0], [], 1, 0, 0, 0, [], []>,
1150:   Arguments<(ins
1151:              LLVM_ScalarOrVectorOf<AB>:$a,
1152:              LLVM_ScalarOrVectorOf<AB>:$b,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1141:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x64_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1141:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x64_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1142:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x64_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1142:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x64_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1143:** This TableGen `def` record introduces `ROCDL_smfmac_f32_32x32x64_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1143:** 该 TableGen `def` 记录引入了 `ROCDL_smfmac_f32_32x32x64_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1144:** Blank line used to separate nearby declarations and improve readability.
  **CN L1144:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1145:** Blank line used to separate nearby declarations and improve readability.
  **CN L1145:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1146:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1146:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1147:** This comment states: “WMMA intrinsics”, documenting the intent of the surrounding code.
  **CN L1147:** 该注释写道：“WMMA intrinsics”，用于说明周围代码的意图。
- **EN L1148:** This TableGen `class` record introduces `ROCDL_WMMA_IntrOp`, which later participates in generated MLIR code.
  **CN L1148:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1149:** This line contributes implementation detail or declarative structure to the file.
  **CN L1149:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1150:** This line contributes implementation detail or declarative structure to the file.
  **CN L1150:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1151:** This line contributes implementation detail or declarative structure to the file.
  **CN L1151:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1152:** This line contributes implementation detail or declarative structure to the file.
  **CN L1152:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1153-1164 / 第 1153-1164 行

```tablegen
1153:              LLVM_ScalarOrVectorOf<CD>:$c)> {
1154:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1155:   let assemblyFormat = [{
1156:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1157:   }];
1158:   let description = [{
1159:     Wave Matrix Multiply-Accumulate (WMMA) intrinsic.
1160: 
1161:     Example:
1162:     ```mlir
1163:     // WMMA with f16 inputs and f32 accumulator.
1164:     %r = rocdl.wmma.f32.16x16x16.f16 %a, %b, %c :
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1153:** This line contributes implementation detail or declarative structure to the file.
  **CN L1153:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1154:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1154:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1155:** This line contributes implementation detail or declarative structure to the file.
  **CN L1155:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1156:** This line contributes to the declaration or call of `type`.
  **CN L1156:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1157:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1157:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1158:** This line contributes implementation detail or declarative structure to the file.
  **CN L1158:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1159:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1159:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1160:** Blank line used to separate nearby declarations and improve readability.
  **CN L1160:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1161:** This line contributes implementation detail or declarative structure to the file.
  **CN L1161:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1162:** This line contributes implementation detail or declarative structure to the file.
  **CN L1162:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1163:** This comment states: “WMMA with f16 inputs and f32 accumulator.”, documenting the intent of the surrounding code.
  **CN L1163:** 该注释写道：“WMMA with f16 inputs and f32 accumulator.”，用于说明周围代码的意图。
- **EN L1164:** This line contributes implementation detail or declarative structure to the file.
  **CN L1164:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1165-1176 / 第 1165-1176 行

```tablegen
1165:       (vector<16xf16>, vector<16xf16>, vector<8xf32>) -> vector<8xf32>
1166:     ```
1167:   }];
1168: }
1169: 
1170: class ROCDL_WMMA_Opsel_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1171:     [0], [1], [], 1, 0, 0, 0, [3], ["opsel"]>,
1172:   Arguments<(ins
1173:              LLVM_ScalarOrVectorOf<AB>:$a,
1174:              LLVM_ScalarOrVectorOf<AB>:$b,
1175:              LLVM_ScalarOrVectorOf<CD>:$c,
1176:              DefaultValuedAttr<I1Attr, "0">:$opsel)> {
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1165:** This line contributes implementation detail or declarative structure to the file.
  **CN L1165:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1166:** This line contributes implementation detail or declarative structure to the file.
  **CN L1166:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1167:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1167:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1168:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1168:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1169:** Blank line used to separate nearby declarations and improve readability.
  **CN L1169:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1170:** This TableGen `class` record introduces `ROCDL_WMMA_Opsel_IntrOp`, which later participates in generated MLIR code.
  **CN L1170:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_Opsel_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1171:** This line contributes implementation detail or declarative structure to the file.
  **CN L1171:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1172:** This line contributes implementation detail or declarative structure to the file.
  **CN L1172:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1173:** This line contributes implementation detail or declarative structure to the file.
  **CN L1173:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1174:** This line contributes implementation detail or declarative structure to the file.
  **CN L1174:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1175:** This line contributes implementation detail or declarative structure to the file.
  **CN L1175:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1176:** This line contributes implementation detail or declarative structure to the file.
  **CN L1176:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1177-1188 / 第 1177-1188 行

```tablegen
1177:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1178:   let assemblyFormat = [{
1179:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1180:   }];
1181:   let description = [{
1182:     Wave Matrix Multiply-Accumulate (WMMA) with output operand selection.
1183: 
1184:     Example:
1185:     ```mlir
1186:     // WMMA f16 with opsel control.
1187:     %r = rocdl.wmma.f16.16x16x16.f16 %a, %b, %c {opsel = false} :
1188:       (vector<16xf16>, vector<16xf16>, vector<16xf16>) -> vector<16xf16>
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1177:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1177:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1178:** This line contributes implementation detail or declarative structure to the file.
  **CN L1178:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1179:** This line contributes to the declaration or call of `type`.
  **CN L1179:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1180:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1180:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1181:** This line contributes implementation detail or declarative structure to the file.
  **CN L1181:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1182:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1182:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1183:** Blank line used to separate nearby declarations and improve readability.
  **CN L1183:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1184:** This line contributes implementation detail or declarative structure to the file.
  **CN L1184:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1185:** This line contributes implementation detail or declarative structure to the file.
  **CN L1185:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1186:** This comment states: “WMMA f16 with opsel control.”, documenting the intent of the surrounding code.
  **CN L1186:** 该注释写道：“WMMA f16 with opsel control.”，用于说明周围代码的意图。
- **EN L1187:** This line contributes implementation detail or declarative structure to the file.
  **CN L1187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1188:** This line contributes implementation detail or declarative structure to the file.
  **CN L1188:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1189-1200 / 第 1189-1200 行

```tablegen
1189:     ```
1190:   }];
1191: }
1192: 
1193: class ROCDL_WMMA_IU_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1194:     [0], [1], [], 1, 0, 0, 0, [0, 2, 5], ["signA", "signB", "clamp"]>,
1195:   Arguments<(ins
1196:              DefaultValuedAttr<I1Attr, "0">:$signA,
1197:              LLVM_ScalarOrVectorOf<AB>:$a,
1198:              DefaultValuedAttr<I1Attr, "0">:$signB,
1199:              LLVM_ScalarOrVectorOf<AB>:$b,
1200:              LLVM_ScalarOrVectorOf<CD>:$c,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1189:** This line contributes implementation detail or declarative structure to the file.
  **CN L1189:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1190:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1190:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1191:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1191:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1192:** Blank line used to separate nearby declarations and improve readability.
  **CN L1192:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1193:** This TableGen `class` record introduces `ROCDL_WMMA_IU_IntrOp`, which later participates in generated MLIR code.
  **CN L1193:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_IU_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1194:** This line contributes implementation detail or declarative structure to the file.
  **CN L1194:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1195:** This line contributes implementation detail or declarative structure to the file.
  **CN L1195:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1196:** This line contributes implementation detail or declarative structure to the file.
  **CN L1196:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1197:** This line contributes implementation detail or declarative structure to the file.
  **CN L1197:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1198:** This line contributes implementation detail or declarative structure to the file.
  **CN L1198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1199:** This line contributes implementation detail or declarative structure to the file.
  **CN L1199:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1200:** This line contributes implementation detail or declarative structure to the file.
  **CN L1200:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1201-1212 / 第 1201-1212 行

```tablegen
1201:              DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1202:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1203:   let assemblyFormat = [{
1204:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1205:   }];
1206:   let description = [{
1207:     Wave Matrix Multiply-Accumulate (WMMA) for integer types with
1208:     sign and clamp control.
1209: 
1210:     Example:
1211:     ```mlir
1212:     // WMMA i32 with unsigned i8 inputs.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1201:** This line contributes implementation detail or declarative structure to the file.
  **CN L1201:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1202:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1202:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1203:** This line contributes implementation detail or declarative structure to the file.
  **CN L1203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1204:** This line contributes to the declaration or call of `type`.
  **CN L1204:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1205:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1205:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1206:** This line contributes implementation detail or declarative structure to the file.
  **CN L1206:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1207:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1207:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1208:** This line contributes implementation detail or declarative structure to the file.
  **CN L1208:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1209:** Blank line used to separate nearby declarations and improve readability.
  **CN L1209:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1210:** This line contributes implementation detail or declarative structure to the file.
  **CN L1210:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1211:** This line contributes implementation detail or declarative structure to the file.
  **CN L1211:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1212:** This comment states: “WMMA i32 with unsigned i8 inputs.”, documenting the intent of the surrounding code.
  **CN L1212:** 该注释写道：“WMMA i32 with unsigned i8 inputs.”，用于说明周围代码的意图。

### Lines 1213-1224 / 第 1213-1224 行

```tablegen
1213:     %r = rocdl.wmma.i32.16x16x16.iu8 %a, %b, %c
1214:       {signA = false, signB = false, clamp = false} :
1215:       (vector<4xi32>, vector<4xi32>, vector<8xi32>) -> vector<8xi32>
1216:     ```
1217:   }];
1218: }
1219: 
1220: class ROCDL_WMMA_ModsAll_Reuse_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1221:     [0], [1], [], 1, 0, 0, 0, [0, 2, 4, 6, 7], ["signA", "signB","modC","reuseA","reuseB"]>,
1222:   Arguments<(ins
1223:              DefaultValuedAttr<I1Attr, "0">:$signA,
1224:              LLVM_ScalarOrVectorOf<AB>:$a,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1213:** This line contributes implementation detail or declarative structure to the file.
  **CN L1213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1214:** This line contributes implementation detail or declarative structure to the file.
  **CN L1214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1215:** This line contributes implementation detail or declarative structure to the file.
  **CN L1215:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1216:** This line contributes implementation detail or declarative structure to the file.
  **CN L1216:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1217:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1217:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1218:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1218:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1219:** Blank line used to separate nearby declarations and improve readability.
  **CN L1219:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1220:** This TableGen `class` record introduces `ROCDL_WMMA_ModsAll_Reuse_IntrOp`, which later participates in generated MLIR code.
  **CN L1220:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_ModsAll_Reuse_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1221:** This line contributes implementation detail or declarative structure to the file.
  **CN L1221:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1222:** This line contributes implementation detail or declarative structure to the file.
  **CN L1222:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1223:** This line contributes implementation detail or declarative structure to the file.
  **CN L1223:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1224:** This line contributes implementation detail or declarative structure to the file.
  **CN L1224:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1225-1236 / 第 1225-1236 行

```tablegen
1225:              DefaultValuedAttr<I1Attr, "0">:$signB,
1226:              LLVM_ScalarOrVectorOf<AB>:$b,
1227:              DefaultValuedAttr<I16Attr, "0">:$modC,
1228:              LLVM_ScalarOrVectorOf<CD>:$c,
1229:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1230:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1231:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1232:   let assemblyFormat = [{
1233:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1234:   }];
1235:   let description = [{
1236:     Wave Matrix Multiply-Accumulate (WMMA) with sign, modC, and reuse controls.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1225:** This line contributes implementation detail or declarative structure to the file.
  **CN L1225:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1226:** This line contributes implementation detail or declarative structure to the file.
  **CN L1226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1227:** This line contributes implementation detail or declarative structure to the file.
  **CN L1227:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1228:** This line contributes implementation detail or declarative structure to the file.
  **CN L1228:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1229:** This line contributes implementation detail or declarative structure to the file.
  **CN L1229:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1230:** This line contributes implementation detail or declarative structure to the file.
  **CN L1230:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1231:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1231:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1232:** This line contributes implementation detail or declarative structure to the file.
  **CN L1232:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1233:** This line contributes to the declaration or call of `type`.
  **CN L1233:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1234:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1234:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1235:** This line contributes implementation detail or declarative structure to the file.
  **CN L1235:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1236:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1236:** 这一行为 `Accumulate` 的声明或调用提供内容。

### Lines 1237-1248 / 第 1237-1248 行

```tablegen
1237: 
1238:     Example:
1239:     ```mlir
1240:     // WMMA f32 with f16 inputs and reuse controls.
1241:     %r = rocdl.wmma.f32.16x16x32.f16 %a, %b, %c :
1242:       (vector<16xf16>, vector<16xf16>, vector<8xf32>) -> vector<8xf32>
1243:     ```
1244:   }];
1245: }
1246: 
1247: class ROCDL_WMMA_ModsC_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1248:     [0], [0], [], 1, 0, 0, 0, [2, 4, 5], ["modC","reuseA","reuseB"]>,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1237:** Blank line used to separate nearby declarations and improve readability.
  **CN L1237:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1238:** This line contributes implementation detail or declarative structure to the file.
  **CN L1238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1239:** This line contributes implementation detail or declarative structure to the file.
  **CN L1239:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1240:** This comment states: “WMMA f32 with f16 inputs and reuse controls.”, documenting the intent of the surrounding code.
  **CN L1240:** 该注释写道：“WMMA f32 with f16 inputs and reuse controls.”，用于说明周围代码的意图。
- **EN L1241:** This line contributes implementation detail or declarative structure to the file.
  **CN L1241:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1242:** This line contributes implementation detail or declarative structure to the file.
  **CN L1242:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1243:** This line contributes implementation detail or declarative structure to the file.
  **CN L1243:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1244:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1244:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1245:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1245:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1246:** Blank line used to separate nearby declarations and improve readability.
  **CN L1246:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1247:** This TableGen `class` record introduces `ROCDL_WMMA_ModsC_IntrOp`, which later participates in generated MLIR code.
  **CN L1247:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_ModsC_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1248:** This line contributes implementation detail or declarative structure to the file.
  **CN L1248:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1249-1260 / 第 1249-1260 行

```tablegen
1249:   Arguments<(ins
1250:              LLVM_ScalarOrVectorOf<AB>:$a,
1251:              LLVM_ScalarOrVectorOf<AB>:$b,
1252:              DefaultValuedAttr<I16Attr, "0">:$modC,
1253:              LLVM_ScalarOrVectorOf<CD>:$c,
1254:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1255:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1256:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1257:   let assemblyFormat = [{
1258:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1259:   }];
1260:   let description = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1249:** This line contributes implementation detail or declarative structure to the file.
  **CN L1249:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1250:** This line contributes implementation detail or declarative structure to the file.
  **CN L1250:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1251:** This line contributes implementation detail or declarative structure to the file.
  **CN L1251:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1252:** This line contributes implementation detail or declarative structure to the file.
  **CN L1252:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1253:** This line contributes implementation detail or declarative structure to the file.
  **CN L1253:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1254:** This line contributes implementation detail or declarative structure to the file.
  **CN L1254:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1255:** This line contributes implementation detail or declarative structure to the file.
  **CN L1255:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1256:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1256:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1257:** This line contributes implementation detail or declarative structure to the file.
  **CN L1257:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1258:** This line contributes to the declaration or call of `type`.
  **CN L1258:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1259:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1259:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1260:** This line contributes implementation detail or declarative structure to the file.
  **CN L1260:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1261-1272 / 第 1261-1272 行

```tablegen
1261:     Wave Matrix Multiply-Accumulate (WMMA) with modC and reuse controls.
1262: 
1263:     Example:
1264:     ```mlir
1265:     // WMMA f32 with fp8 inputs and modC/reuse controls.
1266:     %r = rocdl.wmma.f32.16x16x64.fp8_fp8 %a, %b, %c :
1267:       (vector<16xi32>, vector<16xi32>, vector<8xf32>) -> vector<8xf32>
1268:     ```
1269:   }];
1270: }
1271: 
1272: class ROCDL_WMMA_ModsC_Diff_IntrOp<string mnemonic, Type AB, Type C, Type D> : ROCDL_IntrOp<mnemonic,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1261:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1261:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1262:** Blank line used to separate nearby declarations and improve readability.
  **CN L1262:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1263:** This line contributes implementation detail or declarative structure to the file.
  **CN L1263:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1264:** This line contributes implementation detail or declarative structure to the file.
  **CN L1264:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1265:** This comment states: “WMMA f32 with fp8 inputs and modC/reuse controls.”, documenting the intent of the surrounding code.
  **CN L1265:** 该注释写道：“WMMA f32 with fp8 inputs and modC/reuse controls.”，用于说明周围代码的意图。
- **EN L1266:** This line contributes implementation detail or declarative structure to the file.
  **CN L1266:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1267:** This line contributes implementation detail or declarative structure to the file.
  **CN L1267:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1268:** This line contributes implementation detail or declarative structure to the file.
  **CN L1268:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1269:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1269:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1270:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1270:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1271:** Blank line used to separate nearby declarations and improve readability.
  **CN L1271:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1272:** This TableGen `class` record introduces `ROCDL_WMMA_ModsC_Diff_IntrOp`, which later participates in generated MLIR code.
  **CN L1272:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_ModsC_Diff_IntrOp`，后续会参与生成的 MLIR 代码。

### Lines 1273-1284 / 第 1273-1284 行

```tablegen
1273:     [0], [0, 3], [], 1, 0, 0, 0, [2, 4, 5], ["modC","reuseA","reuseB"]>,
1274:   Arguments<(ins
1275:              LLVM_ScalarOrVectorOf<AB>:$a,
1276:              LLVM_ScalarOrVectorOf<AB>:$b,
1277:              DefaultValuedAttr<I16Attr, "0">:$modC,
1278:              LLVM_ScalarOrVectorOf<C>:$c,
1279:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1280:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1281:   let results = (outs LLVM_ScalarOrVectorOf<D>:$res);
1282:   let assemblyFormat = [{
1283:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1284:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1273:** This line contributes implementation detail or declarative structure to the file.
  **CN L1273:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1274:** This line contributes implementation detail or declarative structure to the file.
  **CN L1274:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1275:** This line contributes implementation detail or declarative structure to the file.
  **CN L1275:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1276:** This line contributes implementation detail or declarative structure to the file.
  **CN L1276:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1277:** This line contributes implementation detail or declarative structure to the file.
  **CN L1277:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1278:** This line contributes implementation detail or declarative structure to the file.
  **CN L1278:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1279:** This line contributes implementation detail or declarative structure to the file.
  **CN L1279:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1280:** This line contributes implementation detail or declarative structure to the file.
  **CN L1280:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1281:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1281:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1282:** This line contributes implementation detail or declarative structure to the file.
  **CN L1282:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1283:** This line contributes to the declaration or call of `type`.
  **CN L1283:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1284:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1284:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1285-1296 / 第 1285-1296 行

```tablegen
1285:   let description = [{
1286:     Wave Matrix Multiply-Accumulate (WMMA) with different C and D types.
1287: 
1288:     Example:
1289:     ```mlir
1290:     // WMMA bf16 output from f32 accumulator with bf16 inputs.
1291:     %r = rocdl.wmma.bf16f32.16x16x32.bf16 %a, %b, %c :
1292:       (vector<16xbf16>, vector<16xbf16>, vector<8xf32>) -> vector<16xbf16>
1293:     ```
1294:   }];
1295: }
1296: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1285:** This line contributes implementation detail or declarative structure to the file.
  **CN L1285:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1286:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1286:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1287:** Blank line used to separate nearby declarations and improve readability.
  **CN L1287:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1288:** This line contributes implementation detail or declarative structure to the file.
  **CN L1288:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1289:** This line contributes implementation detail or declarative structure to the file.
  **CN L1289:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1290:** This comment states: “WMMA bf16 output from f32 accumulator with bf16 inputs.”, documenting the intent of the surrounding code.
  **CN L1290:** 该注释写道：“WMMA bf16 output from f32 accumulator with bf16 inputs.”，用于说明周围代码的意图。
- **EN L1291:** This line contributes implementation detail or declarative structure to the file.
  **CN L1291:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1292:** This line contributes implementation detail or declarative structure to the file.
  **CN L1292:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1293:** This line contributes implementation detail or declarative structure to the file.
  **CN L1293:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1294:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1294:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1295:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1295:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1296:** Blank line used to separate nearby declarations and improve readability.
  **CN L1296:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1297-1308 / 第 1297-1308 行

```tablegen
1297: class ROCDL_WMMA_ModsABClamp_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1298:     [0], [1], [], 1, 0, 0, 0, [0, 2, 5, 6, 7], ["signA", "signB", "reuseA","reuseB", "clamp"]>,
1299:   Arguments<(ins
1300:              DefaultValuedAttr<I1Attr, "0">:$signA,
1301:              LLVM_ScalarOrVectorOf<AB>:$a,
1302:              DefaultValuedAttr<I1Attr, "0">:$signB,
1303:              LLVM_ScalarOrVectorOf<AB>:$b,
1304:              LLVM_ScalarOrVectorOf<CD>:$c,
1305:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1306:              DefaultValuedAttr<I1Attr, "0">:$reuseB,
1307:              DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1308:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1297:** This TableGen `class` record introduces `ROCDL_WMMA_ModsABClamp_IntrOp`, which later participates in generated MLIR code.
  **CN L1297:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_ModsABClamp_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1298:** This line contributes implementation detail or declarative structure to the file.
  **CN L1298:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1299:** This line contributes implementation detail or declarative structure to the file.
  **CN L1299:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1300:** This line contributes implementation detail or declarative structure to the file.
  **CN L1300:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1301:** This line contributes implementation detail or declarative structure to the file.
  **CN L1301:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1302:** This line contributes implementation detail or declarative structure to the file.
  **CN L1302:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1303:** This line contributes implementation detail or declarative structure to the file.
  **CN L1303:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1304:** This line contributes implementation detail or declarative structure to the file.
  **CN L1304:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1305:** This line contributes implementation detail or declarative structure to the file.
  **CN L1305:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1306:** This line contributes implementation detail or declarative structure to the file.
  **CN L1306:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1307:** This line contributes implementation detail or declarative structure to the file.
  **CN L1307:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1308:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1308:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1309-1320 / 第 1309-1320 行

```tablegen
1309:   let assemblyFormat = [{
1310:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1311:   }];
1312:   let description = [{
1313:     Wave Matrix Multiply-Accumulate (WMMA) for integer types with
1314:     sign, reuse, and clamp controls.
1315: 
1316:     Example:
1317:     ```mlir
1318:     // WMMA i32 with unsigned i8 inputs and reuse controls.
1319:     %r = rocdl.wmma.i32.16x16x64.iu8 %a, %b, %c
1320:       {signA = false, signB = false, reuseA = false, reuseB = false, clamp = false} :
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1309:** This line contributes implementation detail or declarative structure to the file.
  **CN L1309:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1310:** This line contributes to the declaration or call of `type`.
  **CN L1310:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1311:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1311:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1312:** This line contributes implementation detail or declarative structure to the file.
  **CN L1312:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1313:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1313:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1314:** This line contributes implementation detail or declarative structure to the file.
  **CN L1314:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1315:** Blank line used to separate nearby declarations and improve readability.
  **CN L1315:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1316:** This line contributes implementation detail or declarative structure to the file.
  **CN L1316:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1317:** This line contributes implementation detail or declarative structure to the file.
  **CN L1317:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1318:** This comment states: “WMMA i32 with unsigned i8 inputs and reuse controls.”, documenting the intent of the surrounding code.
  **CN L1318:** 该注释写道：“WMMA i32 with unsigned i8 inputs and reuse controls.”，用于说明周围代码的意图。
- **EN L1319:** This line contributes implementation detail or declarative structure to the file.
  **CN L1319:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1320:** This line contributes implementation detail or declarative structure to the file.
  **CN L1320:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1321-1332 / 第 1321-1332 行

```tablegen
1321:       (vector<8xi32>, vector<8xi32>, vector<8xi32>) -> vector<8xi32>
1322:     ```
1323:   }];
1324: }
1325: 
1326: // Overloaded operands: [1, 3] refers to LLVM intrinsic parameter positions where
1327: // A is at position 1 and B is at position 3 (after format parameters).
1328: class ROCDL_WMMA_Scale_IntrOp<string mnemonic, Type AB, Type CD, Type ScaleExpTy> : ROCDL_IntrOp<mnemonic,
1329:     [0], [1, 3], [], 1, 0, 0, 0, [0, 2, 4, 6, 7, 9, 10, 12, 13],
1330:     ["fmtA", "fmtB", "modC", "scaleAType", "fmtScaleA",
1331:      "scaleBType", "fmtScaleB", "reuseA", "reuseB"]>,
1332:   Arguments<(ins
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1321:** This line contributes implementation detail or declarative structure to the file.
  **CN L1321:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1322:** This line contributes implementation detail or declarative structure to the file.
  **CN L1322:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1323:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1323:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1324:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1324:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1325:** Blank line used to separate nearby declarations and improve readability.
  **CN L1325:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1326:** This comment states: “Overloaded operands: [1, 3] refers to LLVM intrinsic parameter positions where”, documenting the intent of the surrounding code.
  **CN L1326:** 该注释写道：“Overloaded operands: [1, 3] refers to LLVM intrinsic parameter positions where”，用于说明周围代码的意图。
- **EN L1327:** This comment states: “A is at position 1 and B is at position 3 (after format parameters).”, documenting the intent of the surrounding code.
  **CN L1327:** 该注释写道：“A is at position 1 and B is at position 3 (after format parameters).”，用于说明周围代码的意图。
- **EN L1328:** This TableGen `class` record introduces `ROCDL_WMMA_Scale_IntrOp`, which later participates in generated MLIR code.
  **CN L1328:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_Scale_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1329:** This line contributes implementation detail or declarative structure to the file.
  **CN L1329:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1330:** This line contributes implementation detail or declarative structure to the file.
  **CN L1330:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1331:** This line contributes implementation detail or declarative structure to the file.
  **CN L1331:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1332:** This line contributes implementation detail or declarative structure to the file.
  **CN L1332:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1333-1344 / 第 1333-1344 行

```tablegen
1333:              DefaultValuedAttr<I32Attr, "0">:$fmtA,
1334:              LLVM_ScalarOrVectorOf<AB>:$a,
1335:              DefaultValuedAttr<I32Attr, "0">:$fmtB,
1336:              LLVM_ScalarOrVectorOf<AB>:$b,
1337:              DefaultValuedAttr<I16Attr, "0">:$modC,
1338:              LLVM_ScalarOrVectorOf<CD>:$c,
1339:              DefaultValuedAttr<I32Attr, "0">:$scaleAType,
1340:              DefaultValuedAttr<I32Attr, "0">:$fmtScaleA,
1341:              ScaleExpTy:$scaleA,
1342:              DefaultValuedAttr<I32Attr, "0">:$scaleBType,
1343:              DefaultValuedAttr<I32Attr, "0">:$fmtScaleB,
1344:              ScaleExpTy:$scaleB,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1333:** This line contributes implementation detail or declarative structure to the file.
  **CN L1333:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1334:** This line contributes implementation detail or declarative structure to the file.
  **CN L1334:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1335:** This line contributes implementation detail or declarative structure to the file.
  **CN L1335:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1336:** This line contributes implementation detail or declarative structure to the file.
  **CN L1336:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1337:** This line contributes implementation detail or declarative structure to the file.
  **CN L1337:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1338:** This line contributes implementation detail or declarative structure to the file.
  **CN L1338:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1339:** This line contributes implementation detail or declarative structure to the file.
  **CN L1339:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1340:** This line contributes implementation detail or declarative structure to the file.
  **CN L1340:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1341:** This line contributes implementation detail or declarative structure to the file.
  **CN L1341:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1342:** This line contributes implementation detail or declarative structure to the file.
  **CN L1342:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1343:** This line contributes implementation detail or declarative structure to the file.
  **CN L1343:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1344:** This line contributes implementation detail or declarative structure to the file.
  **CN L1344:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1345-1356 / 第 1345-1356 行

```tablegen
1345:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1346:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1347:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1348:   let assemblyFormat = [{
1349:     $a `,` $b `,` $c `,` $scaleA `,` $scaleB attr-dict `:` functional-type(operands, $res)
1350:   }];
1351:   let description = [{
1352:     Scaled Wave Matrix Multiply-Accumulate (WMMA) with per-operand scaling.
1353: 
1354:     Example:
1355:     ```mlir
1356:     // Scaled WMMA with f8f6f4 format inputs.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1345:** This line contributes implementation detail or declarative structure to the file.
  **CN L1345:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1346:** This line contributes implementation detail or declarative structure to the file.
  **CN L1346:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1347:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1347:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1348:** This line contributes implementation detail or declarative structure to the file.
  **CN L1348:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1349:** This line contributes to the declaration or call of `type`.
  **CN L1349:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1350:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1350:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1351:** This line contributes implementation detail or declarative structure to the file.
  **CN L1351:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1352:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1352:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1353:** Blank line used to separate nearby declarations and improve readability.
  **CN L1353:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1354:** This line contributes implementation detail or declarative structure to the file.
  **CN L1354:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1355:** This line contributes implementation detail or declarative structure to the file.
  **CN L1355:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1356:** This comment states: “Scaled WMMA with f8f6f4 format inputs.”, documenting the intent of the surrounding code.
  **CN L1356:** 该注释写道：“Scaled WMMA with f8f6f4 format inputs.”，用于说明周围代码的意图。

### Lines 1357-1368 / 第 1357-1368 行

```tablegen
1357:     %r = rocdl.wmma.scale.f32.16x16x128.f8f6f4 %a, %b, %c, %scaleA, %scaleB :
1358:       (vector<16xi32>, vector<16xi32>, vector<8xf32>, i32, i32) -> vector<8xf32>
1359:     ```
1360:   }];
1361: }
1362: 
1363: class ROCDL_WMMA_Scale_F4_IntrOp<string mnemonic, Type AB, Type CD, Type ScaleExpTy> : ROCDL_IntrOp<mnemonic,
1364:     [0], [0, 1], [], 1, 0, 0, 0, [2, 4, 5, 7, 8, 10, 11],
1365:     ["modC", "scaleAType", "fmtScaleA",
1366:      "scaleBType", "fmtScaleB", "reuseA", "reuseB"]>,
1367:   Arguments<(ins
1368:              LLVM_ScalarOrVectorOf<AB>:$a,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1357:** This line contributes implementation detail or declarative structure to the file.
  **CN L1357:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1358:** This line contributes implementation detail or declarative structure to the file.
  **CN L1358:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1359:** This line contributes implementation detail or declarative structure to the file.
  **CN L1359:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1360:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1360:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1361:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1361:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1362:** Blank line used to separate nearby declarations and improve readability.
  **CN L1362:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1363:** This TableGen `class` record introduces `ROCDL_WMMA_Scale_F4_IntrOp`, which later participates in generated MLIR code.
  **CN L1363:** 该 TableGen `class` 记录引入了 `ROCDL_WMMA_Scale_F4_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1364:** This line contributes implementation detail or declarative structure to the file.
  **CN L1364:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1365:** This line contributes implementation detail or declarative structure to the file.
  **CN L1365:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1366:** This line contributes implementation detail or declarative structure to the file.
  **CN L1366:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1367:** This line contributes implementation detail or declarative structure to the file.
  **CN L1367:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1368:** This line contributes implementation detail or declarative structure to the file.
  **CN L1368:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1369-1380 / 第 1369-1380 行

```tablegen
1369:              LLVM_ScalarOrVectorOf<AB>:$b,
1370:              DefaultValuedAttr<I16Attr, "0">:$modC,
1371:              LLVM_ScalarOrVectorOf<CD>:$c,
1372:              DefaultValuedAttr<I32Attr, "0">:$scaleAType,
1373:              DefaultValuedAttr<I32Attr, "0">:$fmtScaleA,
1374:              ScaleExpTy:$scaleA,
1375:              DefaultValuedAttr<I32Attr, "0">:$scaleBType,
1376:              DefaultValuedAttr<I32Attr, "0">:$fmtScaleB,
1377:              ScaleExpTy:$scaleB,
1378:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1379:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1380:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1369:** This line contributes implementation detail or declarative structure to the file.
  **CN L1369:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1370:** This line contributes implementation detail or declarative structure to the file.
  **CN L1370:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1371:** This line contributes implementation detail or declarative structure to the file.
  **CN L1371:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1372:** This line contributes implementation detail or declarative structure to the file.
  **CN L1372:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1373:** This line contributes implementation detail or declarative structure to the file.
  **CN L1373:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1374:** This line contributes implementation detail or declarative structure to the file.
  **CN L1374:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1375:** This line contributes implementation detail or declarative structure to the file.
  **CN L1375:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1376:** This line contributes implementation detail or declarative structure to the file.
  **CN L1376:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1377:** This line contributes implementation detail or declarative structure to the file.
  **CN L1377:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1378:** This line contributes implementation detail or declarative structure to the file.
  **CN L1378:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1379:** This line contributes implementation detail or declarative structure to the file.
  **CN L1379:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1380:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1380:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1381-1392 / 第 1381-1392 行

```tablegen
1381:   let assemblyFormat = [{
1382:     $a `,` $b `,` $c `,` $scaleA `,` $scaleB attr-dict `:` functional-type(operands, $res)
1383:   }];
1384:   let description = [{
1385:     Scaled Wave Matrix Multiply-Accumulate (WMMA) for F4 format inputs.
1386: 
1387:     Example:
1388:     ```mlir
1389:     // Scaled WMMA with f4 format inputs.
1390:     %r = rocdl.wmma.scale.f32.16x16x128.f4 %a, %b, %c, %scaleA, %scaleB :
1391:       (vector<8xi32>, vector<8xi32>, vector<8xf32>, i32, i32) -> vector<8xf32>
1392:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1381:** This line contributes implementation detail or declarative structure to the file.
  **CN L1381:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1382:** This line contributes to the declaration or call of `type`.
  **CN L1382:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1383:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1383:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1384:** This line contributes implementation detail or declarative structure to the file.
  **CN L1384:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1385:** This line contributes to the declaration or call of `Accumulate`.
  **CN L1385:** 这一行为 `Accumulate` 的声明或调用提供内容。
- **EN L1386:** Blank line used to separate nearby declarations and improve readability.
  **CN L1386:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1387:** This line contributes implementation detail or declarative structure to the file.
  **CN L1387:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1388:** This line contributes implementation detail or declarative structure to the file.
  **CN L1388:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1389:** This comment states: “Scaled WMMA with f4 format inputs.”, documenting the intent of the surrounding code.
  **CN L1389:** 该注释写道：“Scaled WMMA with f4 format inputs.”，用于说明周围代码的意图。
- **EN L1390:** This line contributes implementation detail or declarative structure to the file.
  **CN L1390:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1391:** This line contributes implementation detail or declarative structure to the file.
  **CN L1391:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1392:** This line contributes implementation detail or declarative structure to the file.
  **CN L1392:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1393-1404 / 第 1393-1404 行

```tablegen
1393:   }];
1394: }
1395: 
1396: // Available from gfx11
1397: def ROCDL_wmma_f32_16x16x16_f16 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.f16", /*Type AB=*/F16, /*Type CD=*/F32>;
1398: def ROCDL_wmma_f32_16x16x16_bf16 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.bf16", AnyInteger, F32>;
1399: def ROCDL_wmma_f16_16x16x16_f16 : ROCDL_WMMA_Opsel_IntrOp<"wmma.f16.16x16x16.f16", F16, F16>;
1400: def ROCDL_wmma_bf16_16x16x16_bf16 : ROCDL_WMMA_Opsel_IntrOp<"wmma.bf16.16x16x16.bf16", AnyInteger, AnyInteger>;
1401: def ROCDL_wmma_i32_16x16x16_iu8 : ROCDL_WMMA_IU_IntrOp<"wmma.i32.16x16x16.iu8", AnyInteger, AnyInteger>;
1402: def ROCDL_wmma_i32_16x16x16_iu4 : ROCDL_WMMA_IU_IntrOp<"wmma.i32.16x16x16.iu4", AnyInteger, AnyInteger>;
1403: // Available from gfx12
1404: def ROCDL_wmma_f32_16x16x16_fp8_fp8 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.fp8_fp8", AnyInteger, F32>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1393:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1393:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1394:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1394:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1395:** Blank line used to separate nearby declarations and improve readability.
  **CN L1395:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1396:** This comment states: “Available from gfx11”, documenting the intent of the surrounding code.
  **CN L1396:** 该注释写道：“Available from gfx11”，用于说明周围代码的意图。
- **EN L1397:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_f16`, which later participates in generated MLIR code.
  **CN L1397:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_f16`，后续会参与生成的 MLIR 代码。
- **EN L1398:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_bf16`, which later participates in generated MLIR code.
  **CN L1398:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1399:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x16_f16`, which later participates in generated MLIR code.
  **CN L1399:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x16_f16`，后续会参与生成的 MLIR 代码。
- **EN L1400:** This TableGen `def` record introduces `ROCDL_wmma_bf16_16x16x16_bf16`, which later participates in generated MLIR code.
  **CN L1400:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_bf16_16x16x16_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1401:** This TableGen `def` record introduces `ROCDL_wmma_i32_16x16x16_iu8`, which later participates in generated MLIR code.
  **CN L1401:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_i32_16x16x16_iu8`，后续会参与生成的 MLIR 代码。
- **EN L1402:** This TableGen `def` record introduces `ROCDL_wmma_i32_16x16x16_iu4`, which later participates in generated MLIR code.
  **CN L1402:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_i32_16x16x16_iu4`，后续会参与生成的 MLIR 代码。
- **EN L1403:** This comment states: “Available from gfx12”, documenting the intent of the surrounding code.
  **CN L1403:** 该注释写道：“Available from gfx12”，用于说明周围代码的意图。
- **EN L1404:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1404:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_fp8_fp8`，后续会参与生成的 MLIR 代码。

### Lines 1405-1416 / 第 1405-1416 行

```tablegen
1405: def ROCDL_wmma_f32_16x16x16_fp8_bf8 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.fp8_bf8", AnyInteger, F32>;
1406: def ROCDL_wmma_f32_16x16x16_bf8_bf8 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.bf8_bf8", AnyInteger, F32>;
1407: def ROCDL_wmma_f32_16x16x16_bf8_fp8 : ROCDL_WMMA_IntrOp<"wmma.f32.16x16x16.bf8_fp8", AnyInteger, F32>;
1408: def ROCDL_wmma_i32_16x16x32_iu4 : ROCDL_WMMA_IU_IntrOp<"wmma.i32.16x16x32.iu4", AnyInteger, AnyInteger>;
1409: // Available from gfx1250
1410: def ROCDL_wmma_f32_16x16x4_f32 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x4.f32", F32, F32>;
1411: def ROCDL_wmma_f32_16x16x32_bf16 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x32.bf16", BF16, F32>;
1412: def ROCDL_wmma_f32_16x16x32_f16 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x32.f16", F16, F32>;
1413: def ROCDL_wmma_f16_16x16x32_f16 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x32.f16", F16, F16>;
1414: def ROCDL_wmma_bf16_16x16x32_bf16 : ROCDL_WMMA_ModsC_IntrOp<"wmma.bf16.16x16x32.bf16", BF16, BF16>;
1415: def ROCDL_wmma_bf16f32_16x16x32_bf16 : ROCDL_WMMA_ModsC_Diff_IntrOp<"wmma.bf16f32.16x16x32.bf16", BF16, /*Type C=*/F32, /*Type D=*/BF16>;
1416: def ROCDL_wmma_f32_16x16x64_fp8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x64.fp8_fp8", AnyInteger, F32>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1405:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1405:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1406:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1406:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1407:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x16_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1407:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x16_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1408:** This TableGen `def` record introduces `ROCDL_wmma_i32_16x16x32_iu4`, which later participates in generated MLIR code.
  **CN L1408:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_i32_16x16x32_iu4`，后续会参与生成的 MLIR 代码。
- **EN L1409:** This comment states: “Available from gfx1250”, documenting the intent of the surrounding code.
  **CN L1409:** 该注释写道：“Available from gfx1250”，用于说明周围代码的意图。
- **EN L1410:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x4_f32`, which later participates in generated MLIR code.
  **CN L1410:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x4_f32`，后续会参与生成的 MLIR 代码。
- **EN L1411:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1411:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1412:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1412:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1413:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1413:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1414:** This TableGen `def` record introduces `ROCDL_wmma_bf16_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1414:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_bf16_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1415:** This TableGen `def` record introduces `ROCDL_wmma_bf16f32_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1415:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_bf16f32_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1416:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x64_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1416:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x64_fp8_fp8`，后续会参与生成的 MLIR 代码。

### Lines 1417-1428 / 第 1417-1428 行

```tablegen
1417: def ROCDL_wmma_f32_16x16x64_fp8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x64.fp8_bf8", AnyInteger, F32>;
1418: def ROCDL_wmma_f32_16x16x64_bf8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x64.bf8_fp8", AnyInteger, F32>;
1419: def ROCDL_wmma_f32_16x16x64_bf8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x64.bf8_bf8", AnyInteger, F32>;
1420: def ROCDL_wmma_f16_16x16x64_fp8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x64.fp8_fp8", AnyInteger, F16>;
1421: def ROCDL_wmma_f16_16x16x64_fp8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x64.fp8_bf8", AnyInteger, F16>;
1422: def ROCDL_wmma_f16_16x16x64_bf8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x64.bf8_fp8", AnyInteger, F16>;
1423: def ROCDL_wmma_f16_16x16x64_bf8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x64.bf8_bf8", AnyInteger, F16>;
1424: def ROCDL_wmma_f32_16x16x128_fp8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x128.fp8_fp8", AnyInteger, F32>;
1425: def ROCDL_wmma_f32_16x16x128_fp8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x128.fp8_bf8", AnyInteger, F32>;
1426: def ROCDL_wmma_f32_16x16x128_bf8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x128.bf8_fp8", AnyInteger, F32>;
1427: def ROCDL_wmma_f32_16x16x128_bf8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f32.16x16x128.bf8_bf8", AnyInteger, F32>;
1428: def ROCDL_wmma_f16_16x16x128_fp8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x128.fp8_fp8", AnyInteger, F16>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1417:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x64_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1417:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x64_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1418:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x64_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1418:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x64_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1419:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x64_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1419:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x64_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1420:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x64_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1420:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x64_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1421:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x64_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1421:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x64_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1422:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x64_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1422:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x64_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1423:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x64_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1423:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x64_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1424:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x128_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1424:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x128_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1425:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x128_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1425:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x128_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1426:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x128_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1426:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x128_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1427:** This TableGen `def` record introduces `ROCDL_wmma_f32_16x16x128_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1427:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f32_16x16x128_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1428:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x128_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1428:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x128_fp8_fp8`，后续会参与生成的 MLIR 代码。

### Lines 1429-1440 / 第 1429-1440 行

```tablegen
1429: def ROCDL_wmma_f16_16x16x128_fp8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x128.fp8_bf8", AnyInteger, F16>;
1430: def ROCDL_wmma_f16_16x16x128_bf8_fp8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x128.bf8_fp8", AnyInteger, F16>;
1431: def ROCDL_wmma_f16_16x16x128_bf8_bf8 : ROCDL_WMMA_ModsC_IntrOp<"wmma.f16.16x16x128.bf8_bf8", AnyInteger, F16>;
1432: def ROCDL_wmma_i32_16x16x64_iu8 : ROCDL_WMMA_ModsABClamp_IntrOp<"wmma.i32.16x16x64.iu8", AnyInteger, AnyInteger>;
1433: 
1434: // Scaled wmma intrinsics (available from gfx1250)
1435: def ROCDL_wmma_scale_f32_16x16x128_f8f6f4   : ROCDL_WMMA_Scale_IntrOp<"wmma.scale.f32.16x16x128.f8f6f4", AnyInteger, F32, I32>;
1436: def ROCDL_wmma_scale16_f32_16x16x128_f8f6f4 : ROCDL_WMMA_Scale_IntrOp<"wmma.scale16.f32.16x16x128.f8f6f4", AnyInteger, F32, I64>;
1437: def ROCDL_wmma_scale_f32_32x16x128_f4       : ROCDL_WMMA_Scale_F4_IntrOp<"wmma.scale.f32.32x16x128.f4", AnyInteger, F32, I32>;
1438: def ROCDL_wmma_scale16_f32_32x16x128_f4     : ROCDL_WMMA_Scale_F4_IntrOp<"wmma.scale16.f32.32x16x128.f4", AnyInteger, F32, I64>;
1439: 
1440: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1429:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x128_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1429:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x128_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1430:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x128_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1430:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x128_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1431:** This TableGen `def` record introduces `ROCDL_wmma_f16_16x16x128_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1431:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_f16_16x16x128_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1432:** This TableGen `def` record introduces `ROCDL_wmma_i32_16x16x64_iu8`, which later participates in generated MLIR code.
  **CN L1432:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_i32_16x16x64_iu8`，后续会参与生成的 MLIR 代码。
- **EN L1433:** Blank line used to separate nearby declarations and improve readability.
  **CN L1433:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1434:** This comment states: “Scaled wmma intrinsics (available from gfx1250)”, documenting the intent of the surrounding code.
  **CN L1434:** 该注释写道：“Scaled wmma intrinsics (available from gfx1250)”，用于说明周围代码的意图。
- **EN L1435:** This TableGen `def` record introduces `ROCDL_wmma_scale_f32_16x16x128_f8f6f4`, which later participates in generated MLIR code.
  **CN L1435:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_scale_f32_16x16x128_f8f6f4`，后续会参与生成的 MLIR 代码。
- **EN L1436:** This TableGen `def` record introduces `ROCDL_wmma_scale16_f32_16x16x128_f8f6f4`, which later participates in generated MLIR code.
  **CN L1436:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_scale16_f32_16x16x128_f8f6f4`，后续会参与生成的 MLIR 代码。
- **EN L1437:** This TableGen `def` record introduces `ROCDL_wmma_scale_f32_32x16x128_f4`, which later participates in generated MLIR code.
  **CN L1437:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_scale_f32_32x16x128_f4`，后续会参与生成的 MLIR 代码。
- **EN L1438:** This TableGen `def` record introduces `ROCDL_wmma_scale16_f32_32x16x128_f4`, which later participates in generated MLIR code.
  **CN L1438:** 该 TableGen `def` 记录引入了 `ROCDL_wmma_scale16_f32_32x16x128_f4`，后续会参与生成的 MLIR 代码。
- **EN L1439:** Blank line used to separate nearby declarations and improve readability.
  **CN L1439:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1440:** Blank line used to separate nearby declarations and improve readability.
  **CN L1440:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1441-1452 / 第 1441-1452 行

```tablegen
1441: //===---------------------------------------------------------------------===//
1442: // Dot product intrinsics (v_dot*)
1443: class ROCDL_Dot_IntrOp<string mnemonic, ROCDL_NamedType A, ROCDL_NamedType B,
1444:                        ROCDL_NamedType C> :
1445:     ROCDL_ConcreteNonMemIntrOp<mnemonic, [Pure], 1, [3], ["clamp"]>,
1446:   Arguments<(ins A:$a, B:$b, C:$c,
1447:                  DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1448:   let results = (outs C:$res);
1449:   let assemblyFormat = [{
1450:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1451:   }];
1452:   let description = [{
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1441:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1441:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1442:** This comment states: “Dot product intrinsics (v_dot*)”, documenting the intent of the surrounding code.
  **CN L1442:** 该注释写道：“Dot product intrinsics (v_dot*)”，用于说明周围代码的意图。
- **EN L1443:** This TableGen `class` record introduces `ROCDL_Dot_IntrOp`, which later participates in generated MLIR code.
  **CN L1443:** 该 TableGen `class` 记录引入了 `ROCDL_Dot_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1444:** This line contributes implementation detail or declarative structure to the file.
  **CN L1444:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1445:** This line contributes implementation detail or declarative structure to the file.
  **CN L1445:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1446:** This line contributes implementation detail or declarative structure to the file.
  **CN L1446:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1447:** This line contributes implementation detail or declarative structure to the file.
  **CN L1447:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1448:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1448:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1449:** This line contributes implementation detail or declarative structure to the file.
  **CN L1449:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1450:** This line contributes to the declaration or call of `type`.
  **CN L1450:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1451:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1451:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1452:** This line contributes implementation detail or declarative structure to the file.
  **CN L1452:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1453-1464 / 第 1453-1464 行

```tablegen
1453:     Packed intra-lane dot-product with optional result clamping (`clamp`).
1454:     Computes `res = sum_i a[i]*b[i] + c`, where `a` and `b` hold packed
1455:     4/8/16-bit data (for `dot2`,`dot4`,`dot8`).
1456: 
1457:     Example:
1458:     ```mlir
1459:     %r = rocdl.}] # mnemonic # [{ %a, %b, %c {clamp = true} :
1460:          (}] # A.typeName # [{, }] # B.typeName # [{, }] # C.typeName # [{) -> }]
1461:          # C.typeName # [{
1462:     ```
1463:   }];
1464: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1453:** This line contributes to the declaration or call of `clamping`.
  **CN L1453:** 这一行为 `clamping` 的声明或调用提供内容。
- **EN L1454:** This line contributes implementation detail or declarative structure to the file.
  **CN L1454:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1455:** This line contributes to the declaration or call of `data`.
  **CN L1455:** 这一行为 `data` 的声明或调用提供内容。
- **EN L1456:** Blank line used to separate nearby declarations and improve readability.
  **CN L1456:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1457:** This line contributes implementation detail or declarative structure to the file.
  **CN L1457:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1458:** This line contributes implementation detail or declarative structure to the file.
  **CN L1458:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1459:** This line contributes implementation detail or declarative structure to the file.
  **CN L1459:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1460:** This line contributes implementation detail or declarative structure to the file.
  **CN L1460:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1461:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L1461:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L1462:** This line contributes implementation detail or declarative structure to the file.
  **CN L1462:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1463:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1463:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1464:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1464:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 1465-1476 / 第 1465-1476 行

```tablegen
1465: 
1466: class ROCDL_Dot_NoClamp_IntrOp<string mnemonic, ROCDL_NamedType A,
1467:                                ROCDL_NamedType B, ROCDL_NamedType C> :
1468:     ROCDL_ConcreteNonMemIntrOp<mnemonic, [Pure], 1, [], []>,
1469:   Arguments<(ins A:$a, B:$b, C:$c)> {
1470:   let results = (outs C:$res);
1471:   let assemblyFormat = [{
1472:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
1473:   }];
1474:   let description = [{
1475:     Packed intra-lane dot-product with no clamp control.
1476:     Computes `res = sum_i a[i]*b[i] + c`. Covers the full-f16/bf16
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1465:** Blank line used to separate nearby declarations and improve readability.
  **CN L1465:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1466:** This TableGen `class` record introduces `ROCDL_Dot_NoClamp_IntrOp`, which later participates in generated MLIR code.
  **CN L1466:** 该 TableGen `class` 记录引入了 `ROCDL_Dot_NoClamp_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1467:** This line contributes implementation detail or declarative structure to the file.
  **CN L1467:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1468:** This line contributes implementation detail or declarative structure to the file.
  **CN L1468:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1469:** This line contributes implementation detail or declarative structure to the file.
  **CN L1469:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1470:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1470:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1471:** This line contributes implementation detail or declarative structure to the file.
  **CN L1471:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1472:** This line contributes to the declaration or call of `type`.
  **CN L1472:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1473:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1473:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1474:** This line contributes implementation detail or declarative structure to the file.
  **CN L1474:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1475:** This line contributes implementation detail or declarative structure to the file.
  **CN L1475:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1476:** This line contributes implementation detail or declarative structure to the file.
  **CN L1476:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1477-1488 / 第 1477-1488 行

```tablegen
1477:     accumulator forms (`fdot2.f16.f16`, `fdot2.bf16.bf16`) and the
1478:     FP8/BF8 `dot4.f32.*` variants, whose hardware instructions have no
1479:     CLAMP bit in their modifier word.
1480: 
1481:     Example:
1482:     ```mlir
1483:     %r = rocdl.}] # mnemonic # [{ %a, %b, %c : (}] # A.typeName # [{, }]
1484:          # B.typeName # [{, }] # C.typeName # [{) -> }] # C.typeName # [{
1485:     ```
1486:   }];
1487: }
1488: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1477:** This line contributes to the declaration or call of `forms`.
  **CN L1477:** 这一行为 `forms` 的声明或调用提供内容。
- **EN L1478:** This line contributes implementation detail or declarative structure to the file.
  **CN L1478:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1479:** This line contributes implementation detail or declarative structure to the file.
  **CN L1479:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1480:** Blank line used to separate nearby declarations and improve readability.
  **CN L1480:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1481:** This line contributes implementation detail or declarative structure to the file.
  **CN L1481:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1482:** This line contributes implementation detail or declarative structure to the file.
  **CN L1482:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1483:** This line contributes implementation detail or declarative structure to the file.
  **CN L1483:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1484:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L1484:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L1485:** This line contributes implementation detail or declarative structure to the file.
  **CN L1485:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1486:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1486:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1487:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1487:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1488:** Blank line used to separate nearby declarations and improve readability.
  **CN L1488:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1489-1500 / 第 1489-1500 行

```tablegen
1489: class ROCDL_Sudot_IntrOp<string mnemonic> :
1490:     ROCDL_ConcreteNonMemIntrOp<mnemonic, [Pure], 1, [0, 2, 5],
1491:                                ["signA", "signB", "clamp"]>,
1492:   Arguments<(ins DefaultValuedAttr<I1Attr, "0">:$signA,
1493:                  I32:$a,
1494:                  DefaultValuedAttr<I1Attr, "0">:$signB,
1495:                  I32:$b,
1496:                  I32:$c,
1497:                  DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1498:   let results = (outs I32:$res);
1499:   let assemblyFormat = [{
1500:     $a `,` $b `,` $c attr-dict `:` functional-type(operands, $res)
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1489:** This TableGen `class` record introduces `ROCDL_Sudot_IntrOp`, which later participates in generated MLIR code.
  **CN L1489:** 该 TableGen `class` 记录引入了 `ROCDL_Sudot_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1490:** This line contributes implementation detail or declarative structure to the file.
  **CN L1490:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1491:** This line contributes implementation detail or declarative structure to the file.
  **CN L1491:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1492:** This line contributes implementation detail or declarative structure to the file.
  **CN L1492:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1493:** This line contributes implementation detail or declarative structure to the file.
  **CN L1493:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1494:** This line contributes implementation detail or declarative structure to the file.
  **CN L1494:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1495:** This line contributes implementation detail or declarative structure to the file.
  **CN L1495:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1496:** This line contributes implementation detail or declarative structure to the file.
  **CN L1496:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1497:** This line contributes implementation detail or declarative structure to the file.
  **CN L1497:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1498:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1498:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1499:** This line contributes implementation detail or declarative structure to the file.
  **CN L1499:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1500:** This line contributes to the declaration or call of `type`.
  **CN L1500:** 这一行为 `type` 的声明或调用提供内容。

### Lines 1501-1512 / 第 1501-1512 行

```tablegen
1501:   }];
1502:   let description = [{
1503:     Mixed-signedness packed dot-product with per-operand sign controls.
1504:     Computes `res = sum_i a[i]*b[i] + c`. Each lane of `a` is treated as
1505:     signed when `signA = true`; when `signA = false`, the unsigned lane
1506:     value is zero-extended into a wider signed integer. `signB` controls
1507:     the same for `b`. `clamp` controls result clamping.
1508: 
1509:     These ops correspond to RDNA's unified mixed-sign `v_dot4_i32_iu8`
1510:     and `v_dot8_i32_iu4` instructions (gfx11+). 
1511: 
1512:     Example:
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1501:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1501:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1502:** This line contributes implementation detail or declarative structure to the file.
  **CN L1502:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1503:** This line contributes implementation detail or declarative structure to the file.
  **CN L1503:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1504:** This line contributes implementation detail or declarative structure to the file.
  **CN L1504:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1505:** This line contributes implementation detail or declarative structure to the file.
  **CN L1505:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1506:** This line contributes implementation detail or declarative structure to the file.
  **CN L1506:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1507:** This line contributes implementation detail or declarative structure to the file.
  **CN L1507:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1508:** Blank line used to separate nearby declarations and improve readability.
  **CN L1508:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1509:** This line contributes implementation detail or declarative structure to the file.
  **CN L1509:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1510:** This line contributes to the declaration or call of `instructions`.
  **CN L1510:** 这一行为 `instructions` 的声明或调用提供内容。
- **EN L1511:** Blank line used to separate nearby declarations and improve readability.
  **CN L1511:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1512:** This line contributes implementation detail or declarative structure to the file.
  **CN L1512:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1513-1524 / 第 1513-1524 行

```tablegen
1513:     ```mlir
1514:     %r = rocdl.}] # mnemonic # [{ %a, %b, %c
1515:            {signA = true, signB = false, clamp = true} :
1516:          (i32, i32, i32) -> i32
1517:     ```
1518:   }];
1519: }
1520: 
1521: // Available from gfx906.
1522: def ROCDL_fdot2 : ROCDL_Dot_IntrOp<"fdot2",
1523:     ROCDL_V2F16Type, ROCDL_V2F16Type, ROCDL_Scalar<F32>>;
1524: def ROCDL_sdot2 : ROCDL_Dot_IntrOp<"sdot2",
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1513:** This line contributes implementation detail or declarative structure to the file.
  **CN L1513:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1514:** This line contributes implementation detail or declarative structure to the file.
  **CN L1514:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1515:** This line contributes implementation detail or declarative structure to the file.
  **CN L1515:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1516:** This line contributes implementation detail or declarative structure to the file.
  **CN L1516:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1517:** This line contributes implementation detail or declarative structure to the file.
  **CN L1517:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1518:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1518:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1519:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1519:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1520:** Blank line used to separate nearby declarations and improve readability.
  **CN L1520:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1521:** This comment states: “Available from gfx906.”, documenting the intent of the surrounding code.
  **CN L1521:** 该注释写道：“Available from gfx906.”，用于说明周围代码的意图。
- **EN L1522:** This TableGen `def` record introduces `ROCDL_fdot2`, which later participates in generated MLIR code.
  **CN L1522:** 该 TableGen `def` 记录引入了 `ROCDL_fdot2`，后续会参与生成的 MLIR 代码。
- **EN L1523:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1523:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1524:** This TableGen `def` record introduces `ROCDL_sdot2`, which later participates in generated MLIR code.
  **CN L1524:** 该 TableGen `def` 记录引入了 `ROCDL_sdot2`，后续会参与生成的 MLIR 代码。

### Lines 1525-1536 / 第 1525-1536 行

```tablegen
1525:     ROCDL_V2I16Type, ROCDL_V2I16Type, ROCDL_Scalar<I32>>;
1526: def ROCDL_udot2 : ROCDL_Dot_IntrOp<"udot2",
1527:     ROCDL_V2I16Type, ROCDL_V2I16Type, ROCDL_Scalar<I32>>;
1528: def ROCDL_sdot4 : ROCDL_Dot_IntrOp<"sdot4",
1529:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<I32>>;
1530: def ROCDL_udot4 : ROCDL_Dot_IntrOp<"udot4",
1531:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<I32>>;
1532: def ROCDL_sdot8 : ROCDL_Dot_IntrOp<"sdot8",
1533:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<I32>>;
1534: def ROCDL_udot8 : ROCDL_Dot_IntrOp<"udot8",
1535:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<I32>>;
1536: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1525:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1525:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1526:** This TableGen `def` record introduces `ROCDL_udot2`, which later participates in generated MLIR code.
  **CN L1526:** 该 TableGen `def` 记录引入了 `ROCDL_udot2`，后续会参与生成的 MLIR 代码。
- **EN L1527:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1527:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1528:** This TableGen `def` record introduces `ROCDL_sdot4`, which later participates in generated MLIR code.
  **CN L1528:** 该 TableGen `def` 记录引入了 `ROCDL_sdot4`，后续会参与生成的 MLIR 代码。
- **EN L1529:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1529:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1530:** This TableGen `def` record introduces `ROCDL_udot4`, which later participates in generated MLIR code.
  **CN L1530:** 该 TableGen `def` 记录引入了 `ROCDL_udot4`，后续会参与生成的 MLIR 代码。
- **EN L1531:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1531:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1532:** This TableGen `def` record introduces `ROCDL_sdot8`, which later participates in generated MLIR code.
  **CN L1532:** 该 TableGen `def` 记录引入了 `ROCDL_sdot8`，后续会参与生成的 MLIR 代码。
- **EN L1533:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1533:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1534:** This TableGen `def` record introduces `ROCDL_udot8`, which later participates in generated MLIR code.
  **CN L1534:** 该 TableGen `def` 记录引入了 `ROCDL_udot8`，后续会参与生成的 MLIR 代码。
- **EN L1535:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1535:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1536:** Blank line used to separate nearby declarations and improve readability.
  **CN L1536:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1537-1548 / 第 1537-1548 行

```tablegen
1537: // Available from gfx11.
1538: def ROCDL_fdot2_f16_f16   : ROCDL_Dot_NoClamp_IntrOp<"fdot2.f16.f16",
1539:     ROCDL_V2F16Type,  ROCDL_V2F16Type,  ROCDL_Scalar<F16>>;
1540: def ROCDL_fdot2_bf16_bf16 : ROCDL_Dot_NoClamp_IntrOp<"fdot2.bf16.bf16",
1541:     ROCDL_V2BF16Type, ROCDL_V2BF16Type, ROCDL_Scalar<BF16>>;
1542: def ROCDL_sudot4 : ROCDL_Sudot_IntrOp<"sudot4">;
1543: def ROCDL_sudot8 : ROCDL_Sudot_IntrOp<"sudot8">;
1544: 
1545: // Available from gfx11 and gfx950.
1546: def ROCDL_fdot2_f32_bf16 : ROCDL_Dot_IntrOp<"fdot2.f32.bf16",
1547:     ROCDL_V2BF16Type, ROCDL_V2BF16Type, ROCDL_Scalar<F32>>;
1548: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1537:** This comment states: “Available from gfx11.”, documenting the intent of the surrounding code.
  **CN L1537:** 该注释写道：“Available from gfx11.”，用于说明周围代码的意图。
- **EN L1538:** This TableGen `def` record introduces `ROCDL_fdot2_f16_f16`, which later participates in generated MLIR code.
  **CN L1538:** 该 TableGen `def` 记录引入了 `ROCDL_fdot2_f16_f16`，后续会参与生成的 MLIR 代码。
- **EN L1539:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1539:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1540:** This TableGen `def` record introduces `ROCDL_fdot2_bf16_bf16`, which later participates in generated MLIR code.
  **CN L1540:** 该 TableGen `def` 记录引入了 `ROCDL_fdot2_bf16_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1541:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1541:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1542:** This TableGen `def` record introduces `ROCDL_sudot4`, which later participates in generated MLIR code.
  **CN L1542:** 该 TableGen `def` 记录引入了 `ROCDL_sudot4`，后续会参与生成的 MLIR 代码。
- **EN L1543:** This TableGen `def` record introduces `ROCDL_sudot8`, which later participates in generated MLIR code.
  **CN L1543:** 该 TableGen `def` 记录引入了 `ROCDL_sudot8`，后续会参与生成的 MLIR 代码。
- **EN L1544:** Blank line used to separate nearby declarations and improve readability.
  **CN L1544:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1545:** This comment states: “Available from gfx11 and gfx950.”, documenting the intent of the surrounding code.
  **CN L1545:** 该注释写道：“Available from gfx11 and gfx950.”，用于说明周围代码的意图。
- **EN L1546:** This TableGen `def` record introduces `ROCDL_fdot2_f32_bf16`, which later participates in generated MLIR code.
  **CN L1546:** 该 TableGen `def` 记录引入了 `ROCDL_fdot2_f32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1547:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1547:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1548:** Blank line used to separate nearby declarations and improve readability.
  **CN L1548:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1549-1560 / 第 1549-1560 行

```tablegen
1549: // Available from gfx12.
1550: def ROCDL_dot4_f32_fp8_fp8 : ROCDL_Dot_NoClamp_IntrOp<"dot4.f32.fp8.fp8",
1551:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<F32>>;
1552: def ROCDL_dot4_f32_fp8_bf8 : ROCDL_Dot_NoClamp_IntrOp<"dot4.f32.fp8.bf8",
1553:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<F32>>;
1554: def ROCDL_dot4_f32_bf8_fp8 : ROCDL_Dot_NoClamp_IntrOp<"dot4.f32.bf8.fp8",
1555:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<F32>>;
1556: def ROCDL_dot4_f32_bf8_bf8 : ROCDL_Dot_NoClamp_IntrOp<"dot4.f32.bf8.bf8",
1557:     ROCDL_Scalar<I32>, ROCDL_Scalar<I32>, ROCDL_Scalar<F32>>;
1558: 
1559: 
1560: //===---------------------------------------------------------------------===//
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1549:** This comment states: “Available from gfx12.”, documenting the intent of the surrounding code.
  **CN L1549:** 该注释写道：“Available from gfx12.”，用于说明周围代码的意图。
- **EN L1550:** This TableGen `def` record introduces `ROCDL_dot4_f32_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1550:** 该 TableGen `def` 记录引入了 `ROCDL_dot4_f32_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1551:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1551:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1552:** This TableGen `def` record introduces `ROCDL_dot4_f32_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1552:** 该 TableGen `def` 记录引入了 `ROCDL_dot4_f32_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1553:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1553:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1554:** This TableGen `def` record introduces `ROCDL_dot4_f32_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1554:** 该 TableGen `def` 记录引入了 `ROCDL_dot4_f32_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1555:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1555:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1556:** This TableGen `def` record introduces `ROCDL_dot4_f32_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1556:** 该 TableGen `def` 记录引入了 `ROCDL_dot4_f32_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1557:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1557:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1558:** Blank line used to separate nearby declarations and improve readability.
  **CN L1558:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1559:** Blank line used to separate nearby declarations and improve readability.
  **CN L1559:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1560:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1560:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 1561-1572 / 第 1561-1572 行

```tablegen
1561: // SWMMAC intrinsics
1562: class ROCDL_SWMMAC_V0_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1563:     [0], [0, 1, 3], [], 1, 0, 0, 0, [], []>,
1564:   Arguments<(ins
1565:              LLVM_VectorOf<AB>:$a,
1566:              LLVM_VectorOf<AB>:$b,
1567:              LLVM_VectorOf<CD>:$c,
1568:              I32:$index)> {
1569:   let results = (outs LLVM_VectorOf<CD>:$res);
1570:   let assemblyFormat = [{
1571:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
1572:   }];
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1561:** This comment states: “SWMMAC intrinsics”, documenting the intent of the surrounding code.
  **CN L1561:** 该注释写道：“SWMMAC intrinsics”，用于说明周围代码的意图。
- **EN L1562:** This TableGen `class` record introduces `ROCDL_SWMMAC_V0_IntrOp`, which later participates in generated MLIR code.
  **CN L1562:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_V0_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1563:** This line contributes implementation detail or declarative structure to the file.
  **CN L1563:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1564:** This line contributes implementation detail or declarative structure to the file.
  **CN L1564:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1565:** This line contributes implementation detail or declarative structure to the file.
  **CN L1565:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1566:** This line contributes implementation detail or declarative structure to the file.
  **CN L1566:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1567:** This line contributes implementation detail or declarative structure to the file.
  **CN L1567:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1568:** This line contributes implementation detail or declarative structure to the file.
  **CN L1568:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1569:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1569:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1570:** This line contributes implementation detail or declarative structure to the file.
  **CN L1570:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1571:** This line contributes to the declaration or call of `type`.
  **CN L1571:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1572:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1572:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1573-1584 / 第 1573-1584 行

```tablegen
1573: }
1574: 
1575: class ROCDL_SWMMAC_V1_IntrOp<string mnemonic, Type A, Type B, Type CD> : ROCDL_IntrOp<mnemonic,
1576:     [0], [0, 1, 3], [], 1, 0, 0, 0, [], []>,
1577:   Arguments<(ins
1578:              LLVM_ScalarOrVectorOf<A>:$a,
1579:              LLVM_ScalarOrVectorOf<B>:$b,
1580:              LLVM_ScalarOrVectorOf<CD>:$c,
1581:              I32:$index)> {
1582:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1583:   let assemblyFormat = [{
1584:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1573:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1573:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1574:** Blank line used to separate nearby declarations and improve readability.
  **CN L1574:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1575:** This TableGen `class` record introduces `ROCDL_SWMMAC_V1_IntrOp`, which later participates in generated MLIR code.
  **CN L1575:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_V1_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1576:** This line contributes implementation detail or declarative structure to the file.
  **CN L1576:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1577:** This line contributes implementation detail or declarative structure to the file.
  **CN L1577:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1578:** This line contributes implementation detail or declarative structure to the file.
  **CN L1578:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1579:** This line contributes implementation detail or declarative structure to the file.
  **CN L1579:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1580:** This line contributes implementation detail or declarative structure to the file.
  **CN L1580:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1581:** This line contributes implementation detail or declarative structure to the file.
  **CN L1581:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1582:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1582:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1583:** This line contributes implementation detail or declarative structure to the file.
  **CN L1583:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1584:** This line contributes to the declaration or call of `type`.
  **CN L1584:** 这一行为 `type` 的声明或调用提供内容。

### Lines 1585-1596 / 第 1585-1596 行

```tablegen
1585:   }];
1586: }
1587: 
1588: class ROCDL_SWMMAC_V1_Reuse_IntrOp<string mnemonic, Type A, Type B, Type CD> : ROCDL_IntrOp<mnemonic,
1589:     [0], [0, 1, 3], [], 1, 0, 0, 0, [4, 5], ["reuseA", "reuseB"]>,
1590:   Arguments<(ins
1591:              LLVM_ScalarOrVectorOf<A>:$a,
1592:              LLVM_ScalarOrVectorOf<B>:$b,
1593:              LLVM_ScalarOrVectorOf<CD>:$c,
1594:              I32:$index,
1595:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1596:              DefaultValuedAttr<I1Attr, "0">:$reuseB
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1585:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1585:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1586:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1586:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1587:** Blank line used to separate nearby declarations and improve readability.
  **CN L1587:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1588:** This TableGen `class` record introduces `ROCDL_SWMMAC_V1_Reuse_IntrOp`, which later participates in generated MLIR code.
  **CN L1588:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_V1_Reuse_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1589:** This line contributes implementation detail or declarative structure to the file.
  **CN L1589:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1590:** This line contributes implementation detail or declarative structure to the file.
  **CN L1590:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1591:** This line contributes implementation detail or declarative structure to the file.
  **CN L1591:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1592:** This line contributes implementation detail or declarative structure to the file.
  **CN L1592:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1593:** This line contributes implementation detail or declarative structure to the file.
  **CN L1593:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1594:** This line contributes implementation detail or declarative structure to the file.
  **CN L1594:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1595:** This line contributes implementation detail or declarative structure to the file.
  **CN L1595:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1596:** This line contributes implementation detail or declarative structure to the file.
  **CN L1596:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1597-1608 / 第 1597-1608 行

```tablegen
1597:              )> {
1598:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1599:   let assemblyFormat = [{
1600:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
1601:   }];
1602: }
1603: 
1604: class ROCDL_SWMMAC_IU_IntrOp<string mnemonic, Type AB, Type CD> : ROCDL_IntrOp<mnemonic,
1605:     [0], [1, 3, 5], [], 1, 0, 0, 0, [0, 2, 6], ["signA", "signB", "clamp"]>,
1606:   Arguments<(ins
1607:              DefaultValuedAttr<I1Attr, "0">:$signA,
1608:              LLVM_ScalarOrVectorOf<AB>:$a,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1597:** This line contributes implementation detail or declarative structure to the file.
  **CN L1597:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1598:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1598:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1599:** This line contributes implementation detail or declarative structure to the file.
  **CN L1599:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1600:** This line contributes to the declaration or call of `type`.
  **CN L1600:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1601:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1601:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1602:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1602:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1603:** Blank line used to separate nearby declarations and improve readability.
  **CN L1603:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1604:** This TableGen `class` record introduces `ROCDL_SWMMAC_IU_IntrOp`, which later participates in generated MLIR code.
  **CN L1604:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_IU_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1605:** This line contributes implementation detail or declarative structure to the file.
  **CN L1605:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1606:** This line contributes implementation detail or declarative structure to the file.
  **CN L1606:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1607:** This line contributes implementation detail or declarative structure to the file.
  **CN L1607:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1608:** This line contributes implementation detail or declarative structure to the file.
  **CN L1608:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1609-1620 / 第 1609-1620 行

```tablegen
1609:              DefaultValuedAttr<I1Attr, "0">:$signB,
1610:              LLVM_ScalarOrVectorOf<AB>:$b,
1611:              LLVM_ScalarOrVectorOf<CD>:$c,
1612:              I32:$index,
1613:              DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1614:   let results = (outs LLVM_ScalarOrVectorOf<CD>:$res);
1615:   let assemblyFormat = [{
1616:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
1617:   }];
1618: }
1619: 
1620: class ROCDL_SWMMAC_ModsAB_IntrOp<string mnemonic, Type AB, Type C, Type D> : ROCDL_IntrOp<mnemonic,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1609:** This line contributes implementation detail or declarative structure to the file.
  **CN L1609:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1610:** This line contributes implementation detail or declarative structure to the file.
  **CN L1610:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1611:** This line contributes implementation detail or declarative structure to the file.
  **CN L1611:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1612:** This line contributes implementation detail or declarative structure to the file.
  **CN L1612:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1613:** This line contributes implementation detail or declarative structure to the file.
  **CN L1613:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1614:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1614:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1615:** This line contributes implementation detail or declarative structure to the file.
  **CN L1615:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1616:** This line contributes to the declaration or call of `type`.
  **CN L1616:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1617:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1617:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1618:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1618:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1619:** Blank line used to separate nearby declarations and improve readability.
  **CN L1619:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1620:** This TableGen `class` record introduces `ROCDL_SWMMAC_ModsAB_IntrOp`, which later participates in generated MLIR code.
  **CN L1620:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_ModsAB_IntrOp`，后续会参与生成的 MLIR 代码。

### Lines 1621-1632 / 第 1621-1632 行

```tablegen
1621:     [0], [1, 3, 5], [], 1, 0, 0, 0, [0, 2, 6, 7], ["signA", "signB", "reuseA", "reuseB"]>,
1622:   Arguments<(ins
1623:              DefaultValuedAttr<I1Attr, "0">:$signA,
1624:              LLVM_ScalarOrVectorOf<AB>:$a,
1625:              DefaultValuedAttr<I1Attr, "0">:$signB,
1626:              LLVM_ScalarOrVectorOf<AB>:$b,
1627:              LLVM_ScalarOrVectorOf<C>:$c,
1628:              I32:$index,
1629:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1630:              DefaultValuedAttr<I1Attr, "0">:$reuseB)> {
1631:   let results = (outs LLVM_ScalarOrVectorOf<D>:$res);
1632:   let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1621:** This line contributes implementation detail or declarative structure to the file.
  **CN L1621:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1622:** This line contributes implementation detail or declarative structure to the file.
  **CN L1622:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1623:** This line contributes implementation detail or declarative structure to the file.
  **CN L1623:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1624:** This line contributes implementation detail or declarative structure to the file.
  **CN L1624:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1625:** This line contributes implementation detail or declarative structure to the file.
  **CN L1625:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1626:** This line contributes implementation detail or declarative structure to the file.
  **CN L1626:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1627:** This line contributes implementation detail or declarative structure to the file.
  **CN L1627:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1628:** This line contributes implementation detail or declarative structure to the file.
  **CN L1628:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1629:** This line contributes implementation detail or declarative structure to the file.
  **CN L1629:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1630:** This line contributes implementation detail or declarative structure to the file.
  **CN L1630:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1631:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1631:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1632:** This line contributes implementation detail or declarative structure to the file.
  **CN L1632:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1633-1644 / 第 1633-1644 行

```tablegen
1633:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
1634:   }];
1635: }
1636: 
1637: class ROCDL_SWMMAC_ModsABClamp_IntrOp<string mnemonic, Type AB, Type C, Type D> : ROCDL_IntrOp<mnemonic,
1638:     [0], [1, 3, 5], [], 1, 0, 0, 0, [0, 2, 6, 7, 8], ["signA", "signB", "reuseA", "reuseB", "clamp"]>,
1639:   Arguments<(ins
1640:              DefaultValuedAttr<I1Attr, "0">:$signA,
1641:              LLVM_ScalarOrVectorOf<AB>:$a,
1642:              DefaultValuedAttr<I1Attr, "0">:$signB,
1643:              LLVM_ScalarOrVectorOf<AB>:$b,
1644:              LLVM_ScalarOrVectorOf<C>:$c,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1633:** This line contributes to the declaration or call of `type`.
  **CN L1633:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1634:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1634:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1635:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1635:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1636:** Blank line used to separate nearby declarations and improve readability.
  **CN L1636:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1637:** This TableGen `class` record introduces `ROCDL_SWMMAC_ModsABClamp_IntrOp`, which later participates in generated MLIR code.
  **CN L1637:** 该 TableGen `class` 记录引入了 `ROCDL_SWMMAC_ModsABClamp_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1638:** This line contributes implementation detail or declarative structure to the file.
  **CN L1638:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1639:** This line contributes implementation detail or declarative structure to the file.
  **CN L1639:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1640:** This line contributes implementation detail or declarative structure to the file.
  **CN L1640:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1641:** This line contributes implementation detail or declarative structure to the file.
  **CN L1641:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1642:** This line contributes implementation detail or declarative structure to the file.
  **CN L1642:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1643:** This line contributes implementation detail or declarative structure to the file.
  **CN L1643:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1644:** This line contributes implementation detail or declarative structure to the file.
  **CN L1644:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1645-1656 / 第 1645-1656 行

```tablegen
1645:              I32:$index,
1646:              DefaultValuedAttr<I1Attr, "0">:$reuseA,
1647:              DefaultValuedAttr<I1Attr, "0">:$reuseB,
1648:              DefaultValuedAttr<I1Attr, "0">:$clamp)> {
1649:   let results = (outs LLVM_ScalarOrVectorOf<D>:$res);
1650:   let assemblyFormat = [{
1651:     $a `,` $b `,` $c `,` $index attr-dict `:` functional-type(operands, $res)
1652:   }];
1653: }
1654: 
1655: // Available from gfx12
1656: def ROCDL_swmmac_f32_16x16x32_f16 : ROCDL_SWMMAC_V0_IntrOp<"swmmac.f32.16x16x32.f16", F16, F32>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1645:** This line contributes implementation detail or declarative structure to the file.
  **CN L1645:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1646:** This line contributes implementation detail or declarative structure to the file.
  **CN L1646:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1647:** This line contributes implementation detail or declarative structure to the file.
  **CN L1647:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1648:** This line contributes implementation detail or declarative structure to the file.
  **CN L1648:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1649:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1649:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1650:** This line contributes implementation detail or declarative structure to the file.
  **CN L1650:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1651:** This line contributes to the declaration or call of `type`.
  **CN L1651:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1652:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1652:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1653:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1653:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1654:** Blank line used to separate nearby declarations and improve readability.
  **CN L1654:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1655:** This comment states: “Available from gfx12”, documenting the intent of the surrounding code.
  **CN L1655:** 该注释写道：“Available from gfx12”，用于说明周围代码的意图。
- **EN L1656:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1656:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_f16`，后续会参与生成的 MLIR 代码。

### Lines 1657-1668 / 第 1657-1668 行

```tablegen
1657: def ROCDL_swmmac_f32_16x16x32_bf16 : ROCDL_SWMMAC_V0_IntrOp<"swmmac.f32.16x16x32.bf16", AnyInteger, F32>;
1658: def ROCDL_swmmac_f16_16x16x32_f16 : ROCDL_SWMMAC_V0_IntrOp<"swmmac.f16.16x16x32.f16", F16, F16>;
1659: def ROCDL_swmmac_bf16_16x16x32_bf16 : ROCDL_SWMMAC_V0_IntrOp<"swmmac.bf16.16x16x32.bf16", AnyInteger, AnyInteger>;
1660: def ROCDL_swmmac_i32_16x16x32_iu8 : ROCDL_SWMMAC_IU_IntrOp<"swmmac.i32.16x16x32.iu8", AnyInteger, AnyInteger>;
1661: def ROCDL_swmmac_i32_16x16x32_iu4 : ROCDL_SWMMAC_IU_IntrOp<"swmmac.i32.16x16x32.iu4", AnyInteger, AnyInteger>;
1662: def ROCDL_swmmac_i32_16x16x64_iu4 : ROCDL_SWMMAC_IU_IntrOp<"swmmac.i32.16x16x64.iu4", AnyInteger, AnyInteger>;
1663: def ROCDL_swmmac_f32_16x16x32_fp8_fp8 : ROCDL_SWMMAC_V1_IntrOp<"swmmac.f32.16x16x32.fp8.fp8", AnyInteger, AnyInteger, F32>;
1664: def ROCDL_swmmac_f32_16x16x32_fp8_bf8 : ROCDL_SWMMAC_V1_IntrOp<"swmmac.f32.16x16x32.fp8.bf8", AnyInteger, AnyInteger, F32>;
1665: def ROCDL_swmmac_f32_16x16x32_bf8_fp8 : ROCDL_SWMMAC_V1_IntrOp<"swmmac.f32.16x16x32.bf8.fp8", AnyInteger, AnyInteger, F32>;
1666: def ROCDL_swmmac_f32_16x16x32_bf8_bf8 : ROCDL_SWMMAC_V1_IntrOp<"swmmac.f32.16x16x32.bf8.bf8", AnyInteger, AnyInteger, F32>;
1667: 
1668: // Available from gfx1250
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1657:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1657:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1658:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x32_f16`, which later participates in generated MLIR code.
  **CN L1658:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x32_f16`，后续会参与生成的 MLIR 代码。
- **EN L1659:** This TableGen `def` record introduces `ROCDL_swmmac_bf16_16x16x32_bf16`, which later participates in generated MLIR code.
  **CN L1659:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_bf16_16x16x32_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1660:** This TableGen `def` record introduces `ROCDL_swmmac_i32_16x16x32_iu8`, which later participates in generated MLIR code.
  **CN L1660:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_i32_16x16x32_iu8`，后续会参与生成的 MLIR 代码。
- **EN L1661:** This TableGen `def` record introduces `ROCDL_swmmac_i32_16x16x32_iu4`, which later participates in generated MLIR code.
  **CN L1661:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_i32_16x16x32_iu4`，后续会参与生成的 MLIR 代码。
- **EN L1662:** This TableGen `def` record introduces `ROCDL_swmmac_i32_16x16x64_iu4`, which later participates in generated MLIR code.
  **CN L1662:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_i32_16x16x64_iu4`，后续会参与生成的 MLIR 代码。
- **EN L1663:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1663:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1664:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1664:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1665:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1665:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1666:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x32_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1666:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x32_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1667:** Blank line used to separate nearby declarations and improve readability.
  **CN L1667:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1668:** This comment states: “Available from gfx1250”, documenting the intent of the surrounding code.
  **CN L1668:** 该注释写道：“Available from gfx1250”，用于说明周围代码的意图。

### Lines 1669-1680 / 第 1669-1680 行

```tablegen
1669: def ROCDL_swmmac_f32_16x16x64_f16 : ROCDL_SWMMAC_ModsAB_IntrOp<"swmmac.f32.16x16x64.f16", F16, F32, F32>;
1670: def ROCDL_swmmac_f32_16x16x64_bf16 : ROCDL_SWMMAC_ModsAB_IntrOp<"swmmac.f32.16x16x64.bf16", BF16, F32, F32>;
1671: def ROCDL_swmmac_f16_16x16x64_f16 : ROCDL_SWMMAC_ModsAB_IntrOp<"swmmac.f16.16x16x64.f16", F16, F16, F16>;
1672: def ROCDL_swmmac_bf16_16x16x64_bf16 : ROCDL_SWMMAC_ModsAB_IntrOp<"swmmac.bf16.16x16x64.bf16", BF16, BF16, BF16>;
1673: def ROCDL_swmmac_bf16f32_16x16x64_bf16 : ROCDL_SWMMAC_ModsAB_IntrOp<"swmmac.bf16f32.16x16x64.bf16", BF16, BF16, BF16>;
1674: def ROCDL_swmmac_f32_16x16x128_fp8_fp8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f32.16x16x128.fp8.fp8", AnyInteger, AnyInteger, F32>;
1675: def ROCDL_swmmac_f32_16x16x128_fp8_bf8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f32.16x16x128.fp8.bf8", AnyInteger, AnyInteger, F32>;
1676: def ROCDL_swmmac_f32_16x16x128_bf8_fp8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f32.16x16x128.bf8.fp8", AnyInteger, AnyInteger, F32>;
1677: def ROCDL_swmmac_f32_16x16x128_bf8_bf8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f32.16x16x128.bf8.bf8", AnyInteger, AnyInteger, F32>;
1678: def ROCDL_swmmac_f16_16x16x128_fp8_fp8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f16.16x16x128.fp8.fp8", AnyInteger, AnyInteger, F16>;
1679: def ROCDL_swmmac_f16_16x16x128_fp8_bf8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f16.16x16x128.fp8.bf8", AnyInteger, AnyInteger, F16>;
1680: def ROCDL_swmmac_f16_16x16x128_bf8_fp8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f16.16x16x128.bf8.fp8", AnyInteger, AnyInteger, F16>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1669:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x64_f16`, which later participates in generated MLIR code.
  **CN L1669:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x64_f16`，后续会参与生成的 MLIR 代码。
- **EN L1670:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x64_bf16`, which later participates in generated MLIR code.
  **CN L1670:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x64_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1671:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x64_f16`, which later participates in generated MLIR code.
  **CN L1671:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x64_f16`，后续会参与生成的 MLIR 代码。
- **EN L1672:** This TableGen `def` record introduces `ROCDL_swmmac_bf16_16x16x64_bf16`, which later participates in generated MLIR code.
  **CN L1672:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_bf16_16x16x64_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1673:** This TableGen `def` record introduces `ROCDL_swmmac_bf16f32_16x16x64_bf16`, which later participates in generated MLIR code.
  **CN L1673:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_bf16f32_16x16x64_bf16`，后续会参与生成的 MLIR 代码。
- **EN L1674:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x128_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1674:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x128_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1675:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x128_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1675:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x128_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1676:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x128_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1676:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x128_bf8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1677:** This TableGen `def` record introduces `ROCDL_swmmac_f32_16x16x128_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1677:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f32_16x16x128_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1678:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x128_fp8_fp8`, which later participates in generated MLIR code.
  **CN L1678:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x128_fp8_fp8`，后续会参与生成的 MLIR 代码。
- **EN L1679:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x128_fp8_bf8`, which later participates in generated MLIR code.
  **CN L1679:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x128_fp8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1680:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x128_bf8_fp8`, which later participates in generated MLIR code.
  **CN L1680:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x128_bf8_fp8`，后续会参与生成的 MLIR 代码。

### Lines 1681-1692 / 第 1681-1692 行

```tablegen
1681: def ROCDL_swmmac_f16_16x16x128_bf8_bf8 : ROCDL_SWMMAC_V1_Reuse_IntrOp<"swmmac.f16.16x16x128.bf8.bf8", AnyInteger, AnyInteger, F16>;
1682: def ROCDL_swmmac_i32_16x16x128_iu8 : ROCDL_SWMMAC_ModsABClamp_IntrOp<"swmmac.i32.16x16x128.iu8", AnyInteger, AnyInteger, AnyInteger>;
1683: 
1684: 
1685: //===---------------------------------------------------------------------===//
1686: // LDS transpose intrinsics (available in GFX950)
1687: 
1688: class ROCDL_LDS_Read_Tr_IntrOp<string mnemonic> :
1689:   ROCDL_IntrOp<mnemonic, [1], [], [], 1, 0, 1> {
1690:   dag args = (ins Arg<ROCDLBufferLDS, "", [MemRead]>:$ptr);
1691:   let arguments = !con(args, baseArgs);
1692:   let assemblyFormat = "$ptr attr-dict `:` type($ptr) `->` type($res)";
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1681:** This TableGen `def` record introduces `ROCDL_swmmac_f16_16x16x128_bf8_bf8`, which later participates in generated MLIR code.
  **CN L1681:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_f16_16x16x128_bf8_bf8`，后续会参与生成的 MLIR 代码。
- **EN L1682:** This TableGen `def` record introduces `ROCDL_swmmac_i32_16x16x128_iu8`, which later participates in generated MLIR code.
  **CN L1682:** 该 TableGen `def` 记录引入了 `ROCDL_swmmac_i32_16x16x128_iu8`，后续会参与生成的 MLIR 代码。
- **EN L1683:** Blank line used to separate nearby declarations and improve readability.
  **CN L1683:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1684:** Blank line used to separate nearby declarations and improve readability.
  **CN L1684:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1685:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1685:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1686:** This comment states: “LDS transpose intrinsics (available in GFX950)”, documenting the intent of the surrounding code.
  **CN L1686:** 该注释写道：“LDS transpose intrinsics (available in GFX950)”，用于说明周围代码的意图。
- **EN L1687:** Blank line used to separate nearby declarations and improve readability.
  **CN L1687:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1688:** This TableGen `class` record introduces `ROCDL_LDS_Read_Tr_IntrOp`, which later participates in generated MLIR code.
  **CN L1688:** 该 TableGen `class` 记录引入了 `ROCDL_LDS_Read_Tr_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L1689:** This line contributes implementation detail or declarative structure to the file.
  **CN L1689:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1690:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1690:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1691:** This line contributes to the declaration or call of `con`.
  **CN L1691:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1692:** This line contributes to the declaration or call of `type`.
  **CN L1692:** 这一行为 `type` 的声明或调用提供内容。

### Lines 1693-1704 / 第 1693-1704 行

```tablegen
1693:   let extraClassDefinition = [{
1694:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1695:       return {getPtr()};
1696:     }
1697:   }];
1698: }
1699: 
1700: def ROCDL_ds_read_tr4_b64 : ROCDL_LDS_Read_Tr_IntrOp<"ds.read.tr4.b64">;
1701: def ROCDL_ds_read_tr8_b64 : ROCDL_LDS_Read_Tr_IntrOp<"ds.read.tr8.b64">;
1702: def ROCDL_ds_read_tr6_b96 : ROCDL_LDS_Read_Tr_IntrOp<"ds.read.tr6.b96">;
1703: def ROCDL_ds_read_tr16_b64 : ROCDL_LDS_Read_Tr_IntrOp<"ds.read.tr16.b64">;
1704: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1693:** This line contributes implementation detail or declarative structure to the file.
  **CN L1693:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1694:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1694:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1695:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1695:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1696:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1696:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1697:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1697:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1698:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1698:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1699:** Blank line used to separate nearby declarations and improve readability.
  **CN L1699:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1700:** This TableGen `def` record introduces `ROCDL_ds_read_tr4_b64`, which later participates in generated MLIR code.
  **CN L1700:** 该 TableGen `def` 记录引入了 `ROCDL_ds_read_tr4_b64`，后续会参与生成的 MLIR 代码。
- **EN L1701:** This TableGen `def` record introduces `ROCDL_ds_read_tr8_b64`, which later participates in generated MLIR code.
  **CN L1701:** 该 TableGen `def` 记录引入了 `ROCDL_ds_read_tr8_b64`，后续会参与生成的 MLIR 代码。
- **EN L1702:** This TableGen `def` record introduces `ROCDL_ds_read_tr6_b96`, which later participates in generated MLIR code.
  **CN L1702:** 该 TableGen `def` 记录引入了 `ROCDL_ds_read_tr6_b96`，后续会参与生成的 MLIR 代码。
- **EN L1703:** This TableGen `def` record introduces `ROCDL_ds_read_tr16_b64`, which later participates in generated MLIR code.
  **CN L1703:** 该 TableGen `def` 记录引入了 `ROCDL_ds_read_tr16_b64`，后续会参与生成的 MLIR 代码。
- **EN L1704:** Blank line used to separate nearby declarations and improve readability.
  **CN L1704:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1705-1716 / 第 1705-1716 行

```tablegen
1705: 
1706: 
1707: //===---------------------------------------------------------------------===//
1708: // Glb/DS load-transpose intrinsics (available in GFX1250+)
1709: 
1710: class AddrKind<string n, int s> {
1711:   string name = n;
1712:   int space = s;
1713: }
1714: def GlobalAddrKind : AddrKind<"global", 1>;
1715: def DSAddrKind : AddrKind<"ds", 3>;
1716: 
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1705:** Blank line used to separate nearby declarations and improve readability.
  **CN L1705:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1706:** Blank line used to separate nearby declarations and improve readability.
  **CN L1706:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1707:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1707:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1708:** This comment states: “Glb/DS load-transpose intrinsics (available in GFX1250+)”, documenting the intent of the surrounding code.
  **CN L1708:** 该注释写道：“Glb/DS load-transpose intrinsics (available in GFX1250+)”，用于说明周围代码的意图。
- **EN L1709:** Blank line used to separate nearby declarations and improve readability.
  **CN L1709:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1710:** This TableGen `class` record introduces `AddrKind`, which later participates in generated MLIR code.
  **CN L1710:** 该 TableGen `class` 记录引入了 `AddrKind`，后续会参与生成的 MLIR 代码。
- **EN L1711:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1711:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1712:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1712:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1713:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1713:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1714:** This TableGen `def` record introduces `GlobalAddrKind`, which later participates in generated MLIR code.
  **CN L1714:** 该 TableGen `def` 记录引入了 `GlobalAddrKind`，后续会参与生成的 MLIR 代码。
- **EN L1715:** This TableGen `def` record introduces `DSAddrKind`, which later participates in generated MLIR code.
  **CN L1715:** 该 TableGen `def` 记录引入了 `DSAddrKind`，后续会参与生成的 MLIR 代码。
- **EN L1716:** Blank line used to separate nearby declarations and improve readability.
  **CN L1716:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1717-1728 / 第 1717-1728 行

```tablegen
1717: class ROCDL_TrLoadOpMeta<AddrKind kind, int inElemBits, int outElemBits> {
1718:   AddrKind addrKind = kind;
1719:   string inBits = !cast<string>(inElemBits);
1720:   string outBits = !cast<string>(outElemBits);
1721:   string inBitsEnc = !if(!eq(addrKind.space, 1),
1722:                      !if(!or(!eq(inElemBits, 8), !eq(inElemBits, 16)), "", inBits), inBits);
1723:   string mnemonic = addrKind.name # ".load.tr" # inBitsEnc # ".b" # outBits;
1724: }
1725: 
1726: class ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta meta> :
1727:   ROCDL_IntrOp<meta.mnemonic, [1], [], [], 1, 0, 1> {
1728: 
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1717:** This TableGen `class` record introduces `ROCDL_TrLoadOpMeta`, which later participates in generated MLIR code.
  **CN L1717:** 该 TableGen `class` 记录引入了 `ROCDL_TrLoadOpMeta`，后续会参与生成的 MLIR 代码。
- **EN L1718:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1718:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1719:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1719:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1720:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1720:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1721:** This line contributes to the declaration or call of `if`.
  **CN L1721:** 这一行为 `if` 的声明或调用提供内容。
- **EN L1722:** This line contributes to the declaration or call of `if`.
  **CN L1722:** 这一行为 `if` 的声明或调用提供内容。
- **EN L1723:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1723:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1724:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1724:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1725:** Blank line used to separate nearby declarations and improve readability.
  **CN L1725:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1726:** This TableGen `class` record introduces `ROCDL_TrLoadOp`, which later participates in generated MLIR code.
  **CN L1726:** 该 TableGen `class` 记录引入了 `ROCDL_TrLoadOp`，后续会参与生成的 MLIR 代码。
- **EN L1727:** This line contributes implementation detail or declarative structure to the file.
  **CN L1727:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1728:** Blank line used to separate nearby declarations and improve readability.
  **CN L1728:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1729-1740 / 第 1729-1740 行

```tablegen
1729:   dag args = (ins Arg<LLVM_PointerInAddressSpace<meta.addrKind.space>, "", [MemRead]>:$ptr);
1730:   let arguments = !con(args, baseArgs);
1731:   let summary = "Loads and transposes a matrix from " # meta.addrKind.name # " memory to registers (available in gfx1250+).";
1732:   let description = [{
1733:     Load a matrix of }] # meta.inBits # [{-bit data from the }] # meta.addrKind.name # [{ memory,
1734:     transpose data between row-major and column-major order,
1735:     and store the result into a }] # meta.outBits # [{-bit vector register.
1736: 
1737:     Available in gfx1250+.
1738: 
1739:     Example (concrete mnemonics depend on address space and element size):
1740:     ```mlir
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1729:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1729:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1730:** This line contributes to the declaration or call of `con`.
  **CN L1730:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1731:** This line contributes to the declaration or call of `registers`.
  **CN L1731:** 这一行为 `registers` 的声明或调用提供内容。
- **EN L1732:** This line contributes implementation detail or declarative structure to the file.
  **CN L1732:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1733:** This line contributes implementation detail or declarative structure to the file.
  **CN L1733:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1734:** This line contributes implementation detail or declarative structure to the file.
  **CN L1734:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1735:** This line contributes implementation detail or declarative structure to the file.
  **CN L1735:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1736:** Blank line used to separate nearby declarations and improve readability.
  **CN L1736:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1737:** This line contributes implementation detail or declarative structure to the file.
  **CN L1737:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1738:** Blank line used to separate nearby declarations and improve readability.
  **CN L1738:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1739:** This line contributes to the declaration or call of `Example`.
  **CN L1739:** 这一行为 `Example` 的声明或调用提供内容。
- **EN L1740:** This line contributes implementation detail or declarative structure to the file.
  **CN L1740:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1741-1752 / 第 1741-1752 行

```tablegen
1741:     // 64-bit transpose load from global memory.
1742:     %0 = rocdl.global.load.tr4.b64 %ptr : !llvm.ptr<1> -> vector<2xi32>
1743: 
1744:     // 128-bit transpose load from global memory with f16 result.
1745:     %1 = rocdl.global.load.tr.b128 %ptr : !llvm.ptr<1> -> vector<8xf16>
1746: 
1747:     // 64-bit transpose load from LDS.
1748:     %2 = rocdl.ds.load.tr4.b64 %ptr : !llvm.ptr<3> -> vector<2xi32>
1749: 
1750:     // 128-bit transpose load from LDS with bf16 result.
1751:     %3 = rocdl.ds.load.tr16.b128 %ptr : !llvm.ptr<3> -> vector<8xbf16>
1752:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1741:** This comment states: “64-bit transpose load from global memory.”, documenting the intent of the surrounding code.
  **CN L1741:** 该注释写道：“64-bit transpose load from global memory.”，用于说明周围代码的意图。
- **EN L1742:** This line contributes implementation detail or declarative structure to the file.
  **CN L1742:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1743:** Blank line used to separate nearby declarations and improve readability.
  **CN L1743:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1744:** This comment states: “128-bit transpose load from global memory with f16 result.”, documenting the intent of the surrounding code.
  **CN L1744:** 该注释写道：“128-bit transpose load from global memory with f16 result.”，用于说明周围代码的意图。
- **EN L1745:** This line contributes implementation detail or declarative structure to the file.
  **CN L1745:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1746:** Blank line used to separate nearby declarations and improve readability.
  **CN L1746:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1747:** This comment states: “64-bit transpose load from LDS.”, documenting the intent of the surrounding code.
  **CN L1747:** 该注释写道：“64-bit transpose load from LDS.”，用于说明周围代码的意图。
- **EN L1748:** This line contributes implementation detail or declarative structure to the file.
  **CN L1748:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1749:** Blank line used to separate nearby declarations and improve readability.
  **CN L1749:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1750:** This comment states: “128-bit transpose load from LDS with bf16 result.”, documenting the intent of the surrounding code.
  **CN L1750:** 该注释写道：“128-bit transpose load from LDS with bf16 result.”，用于说明周围代码的意图。
- **EN L1751:** This line contributes implementation detail or declarative structure to the file.
  **CN L1751:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1752:** This line contributes implementation detail or declarative structure to the file.
  **CN L1752:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1753-1764 / 第 1753-1764 行

```tablegen
1753:   }];
1754:   let assemblyFormat = "$ptr attr-dict `:` qualified(type($ptr)) `->` type($res)";
1755:   let extraClassDefinition = [{
1756:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1757:       return {getPtr()};
1758:     }
1759:   }];
1760: }
1761: 
1762: def ROCDL_GlobalLoadTr4_B64 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<GlobalAddrKind, 4, 64>>;
1763: def ROCDL_GlobalLoadTr8_B64 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<GlobalAddrKind, 8, 64>>;
1764: def ROCDL_GlobalLoadTr6_B96 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<GlobalAddrKind, 6, 96>>;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1753:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1753:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1754:** This line contributes to the declaration or call of `qualified`.
  **CN L1754:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1755:** This line contributes implementation detail or declarative structure to the file.
  **CN L1755:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1756:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1756:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1757:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1757:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1758:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1758:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1759:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1759:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1760:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1760:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1761:** Blank line used to separate nearby declarations and improve readability.
  **CN L1761:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1762:** This TableGen `def` record introduces `ROCDL_GlobalLoadTr4_B64`, which later participates in generated MLIR code.
  **CN L1762:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadTr4_B64`，后续会参与生成的 MLIR 代码。
- **EN L1763:** This TableGen `def` record introduces `ROCDL_GlobalLoadTr8_B64`, which later participates in generated MLIR code.
  **CN L1763:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadTr8_B64`，后续会参与生成的 MLIR 代码。
- **EN L1764:** This TableGen `def` record introduces `ROCDL_GlobalLoadTr6_B96`, which later participates in generated MLIR code.
  **CN L1764:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadTr6_B96`，后续会参与生成的 MLIR 代码。

### Lines 1765-1776 / 第 1765-1776 行

```tablegen
1765: def ROCDL_GlobalLoadTr8_B128 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<GlobalAddrKind, 16, 128>>;
1766: 
1767: def ROCDL_DsLoadTr4_B64 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<DSAddrKind, 4, 64>>;
1768: def ROCDL_DsLoadTr8_B64 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<DSAddrKind, 8, 64>>;
1769: def ROCDL_DsLoadTr6_B96 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<DSAddrKind, 6, 96>>;
1770: def ROCDL_DsLoadTr16_B128 : ROCDL_TrLoadOp<ROCDL_TrLoadOpMeta<DSAddrKind, 16, 128>>;
1771: 
1772: //===---------------------------------------------------------------------===//
1773: // Load to LDS intrinsic (available in GFX9 and GFX10)
1774: //===---------------------------------------------------------------------===//
1775: 
1776: def ROCDL_LoadToLDSOp :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1765:** This TableGen `def` record introduces `ROCDL_GlobalLoadTr8_B128`, which later participates in generated MLIR code.
  **CN L1765:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadTr8_B128`，后续会参与生成的 MLIR 代码。
- **EN L1766:** Blank line used to separate nearby declarations and improve readability.
  **CN L1766:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1767:** This TableGen `def` record introduces `ROCDL_DsLoadTr4_B64`, which later participates in generated MLIR code.
  **CN L1767:** 该 TableGen `def` 记录引入了 `ROCDL_DsLoadTr4_B64`，后续会参与生成的 MLIR 代码。
- **EN L1768:** This TableGen `def` record introduces `ROCDL_DsLoadTr8_B64`, which later participates in generated MLIR code.
  **CN L1768:** 该 TableGen `def` 记录引入了 `ROCDL_DsLoadTr8_B64`，后续会参与生成的 MLIR 代码。
- **EN L1769:** This TableGen `def` record introduces `ROCDL_DsLoadTr6_B96`, which later participates in generated MLIR code.
  **CN L1769:** 该 TableGen `def` 记录引入了 `ROCDL_DsLoadTr6_B96`，后续会参与生成的 MLIR 代码。
- **EN L1770:** This TableGen `def` record introduces `ROCDL_DsLoadTr16_B128`, which later participates in generated MLIR code.
  **CN L1770:** 该 TableGen `def` 记录引入了 `ROCDL_DsLoadTr16_B128`，后续会参与生成的 MLIR 代码。
- **EN L1771:** Blank line used to separate nearby declarations and improve readability.
  **CN L1771:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1772:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1772:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1773:** This comment states: “Load to LDS intrinsic (available in GFX9 and GFX10)”, documenting the intent of the surrounding code.
  **CN L1773:** 该注释写道：“Load to LDS intrinsic (available in GFX9 and GFX10)”，用于说明周围代码的意图。
- **EN L1774:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1774:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1775:** Blank line used to separate nearby declarations and improve readability.
  **CN L1775:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1776:** This TableGen `def` record introduces `ROCDL_LoadToLDSOp`, which later participates in generated MLIR code.
  **CN L1776:** 该 TableGen `def` 记录引入了 `ROCDL_LoadToLDSOp`，后续会参与生成的 MLIR 代码。

### Lines 1777-1788 / 第 1777-1788 行

```tablegen
1777:   ROCDL_IntrOp<"load.to.lds", [], [0], [], 0, 0, 1, 0, [2, 3, 4], ["size", "offset", "aux"]> {
1778:   dag args = (ins Arg<LLVM_AnyPointer, "", [MemRead]>:$globalPtr,
1779:                  Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1780:                  I32Attr:$size,
1781:                  I32Attr:$offset,
1782:                  I32Attr:$aux);
1783:   let arguments = !con(args, baseArgs);
1784:   let assemblyFormat = [{
1785:     $globalPtr `,`  $ldsPtr `,` $size `,` $offset `,` $aux
1786:     attr-dict `:` type($globalPtr)
1787:   }];
1788:   let extraClassDefinition = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1777:** This line contributes implementation detail or declarative structure to the file.
  **CN L1777:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1778:** This line contributes implementation detail or declarative structure to the file.
  **CN L1778:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1779:** This line contributes implementation detail or declarative structure to the file.
  **CN L1779:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1780:** This line contributes implementation detail or declarative structure to the file.
  **CN L1780:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1781:** This line contributes implementation detail or declarative structure to the file.
  **CN L1781:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1782:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1782:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1783:** This line contributes to the declaration or call of `con`.
  **CN L1783:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1784:** This line contributes implementation detail or declarative structure to the file.
  **CN L1784:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1785:** This line contributes implementation detail or declarative structure to the file.
  **CN L1785:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1786:** This line contributes to the declaration or call of `type`.
  **CN L1786:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1787:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1787:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1788:** This line contributes implementation detail or declarative structure to the file.
  **CN L1788:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1789-1800 / 第 1789-1800 行

```tablegen
1789:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1790:       return {getGlobalPtr(), getLdsPtr()};
1791:     }
1792:   }];
1793: }
1794: 
1795: def ROCDL_LoadAsyncToLDSOp :
1796:   ROCDL_IntrOp<"load.async.to.lds", [], [0], [], 0, 0, 1, 0, [2, 3, 4], ["size", "offset", "aux"]> {
1797:   dag args = (ins Arg<LLVM_AnyPointer, "", [MemRead]>:$globalPtr,
1798:                  Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1799:                  I32Attr:$size,
1800:                  I32Attr:$offset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1789:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1789:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1790:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1790:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1791:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1791:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1792:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1792:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1793:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1793:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1794:** Blank line used to separate nearby declarations and improve readability.
  **CN L1794:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1795:** This TableGen `def` record introduces `ROCDL_LoadAsyncToLDSOp`, which later participates in generated MLIR code.
  **CN L1795:** 该 TableGen `def` 记录引入了 `ROCDL_LoadAsyncToLDSOp`，后续会参与生成的 MLIR 代码。
- **EN L1796:** This line contributes implementation detail or declarative structure to the file.
  **CN L1796:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1797:** This line contributes implementation detail or declarative structure to the file.
  **CN L1797:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1798:** This line contributes implementation detail or declarative structure to the file.
  **CN L1798:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1799:** This line contributes implementation detail or declarative structure to the file.
  **CN L1799:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1800:** This line contributes implementation detail or declarative structure to the file.
  **CN L1800:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1801-1812 / 第 1801-1812 行

```tablegen
1801:                  I32Attr:$aux);
1802:   let arguments = !con(args, baseArgs);
1803:   let assemblyFormat = [{
1804:     $globalPtr `,`  $ldsPtr `,` $size `,` $offset `,` $aux
1805:     attr-dict `:` qualified(type($globalPtr)) `,` qualified(type($ldsPtr))
1806:   }];
1807:   let extraClassDefinition = [{
1808:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1809:       return {getGlobalPtr(), getLdsPtr()};
1810:     }
1811:   }];
1812: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1801:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1801:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1802:** This line contributes to the declaration or call of `con`.
  **CN L1802:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1803:** This line contributes implementation detail or declarative structure to the file.
  **CN L1803:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1804:** This line contributes implementation detail or declarative structure to the file.
  **CN L1804:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1805:** This line contributes to the declaration or call of `qualified`.
  **CN L1805:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1806:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1806:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1807:** This line contributes implementation detail or declarative structure to the file.
  **CN L1807:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1808:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1808:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1809:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1809:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1810:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1810:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1811:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1811:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1812:** Blank line used to separate nearby declarations and improve readability.
  **CN L1812:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1813-1824 / 第 1813-1824 行

```tablegen
1813:   let summary = "Gathering load to LDS that requires explicit async memory tracking";
1814:   let description = [{
1815:     Load `size` bytes (the valid sizes vary by architecture) from the global memory
1816:     pointed to by `globalPtr` and put them at `ldsPtr`, concantenating (and applying
1817:     padding for sizes less than 4 bytes, along with padding out 12-byte reads
1818:     to 16-byte writes). The value of `globalPtr` can vary between lanes, while
1819:     `sharedPtr` must be subgroup-uniform (the values from each lane are concatentated
1820:     before being written to LDS with appropriate padding applied.)
1821: 
1822:     `offset` is a constant offset applied to **both** pointers, and `aux` sets the cache
1823:     policy. Unlike `rocdl.load.to.lds`, the compiler will not automatically inserts waits
1824:     for this load to complete at the point it thinks you're using a region of LDS you've
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1813:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1813:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1814:** This line contributes implementation detail or declarative structure to the file.
  **CN L1814:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1815:** This line contributes to the declaration or call of `bytes`.
  **CN L1815:** 这一行为 `bytes` 的声明或调用提供内容。
- **EN L1816:** This line contributes to the declaration or call of `concantenating`.
  **CN L1816:** 这一行为 `concantenating` 的声明或调用提供内容。
- **EN L1817:** This line contributes implementation detail or declarative structure to the file.
  **CN L1817:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1818:** This line contributes implementation detail or declarative structure to the file.
  **CN L1818:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1819:** This line contributes to the declaration or call of `uniform`.
  **CN L1819:** 这一行为 `uniform` 的声明或调用提供内容。
- **EN L1820:** This line contributes implementation detail or declarative structure to the file.
  **CN L1820:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1821:** Blank line used to separate nearby declarations and improve readability.
  **CN L1821:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1822:** This line contributes implementation detail or declarative structure to the file.
  **CN L1822:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1823:** This line contributes implementation detail or declarative structure to the file.
  **CN L1823:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1824:** This line contributes implementation detail or declarative structure to the file.
  **CN L1824:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1825-1836 / 第 1825-1836 行

```tablegen
1825:     stored values to - you need to use the `rocdl.asyncmark` and `rocdl.wait.asyncmark`
1826:     operations to explicitly group these operations and wait for their completion.
1827: 
1828:     Available on gfx10 and earlier with varying suppported values of `size`.
1829: 
1830:     Example:
1831:     ```mlir
1832:     // Async load 4 bytes from global pointer to LDS.
1833:     rocdl.load.async.to.lds %global, %shared, 4, 0, 0 : !llvm.ptr<1>, !llvm.ptr<3>
1834: 
1835:     // Async load 4 bytes from fat buffer pointer to LDS.
1836:     rocdl.load.async.to.lds %fatBuffer, %shared, 4, 0, 0 : !llvm.ptr<7>, !llvm.ptr<3>
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1825:** This line contributes implementation detail or declarative structure to the file.
  **CN L1825:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1826:** This line contributes implementation detail or declarative structure to the file.
  **CN L1826:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1827:** Blank line used to separate nearby declarations and improve readability.
  **CN L1827:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1828:** This line contributes implementation detail or declarative structure to the file.
  **CN L1828:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1829:** Blank line used to separate nearby declarations and improve readability.
  **CN L1829:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1830:** This line contributes implementation detail or declarative structure to the file.
  **CN L1830:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1831:** This line contributes implementation detail or declarative structure to the file.
  **CN L1831:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1832:** This comment states: “Async load 4 bytes from global pointer to LDS.”, documenting the intent of the surrounding code.
  **CN L1832:** 该注释写道：“Async load 4 bytes from global pointer to LDS.”，用于说明周围代码的意图。
- **EN L1833:** This line contributes implementation detail or declarative structure to the file.
  **CN L1833:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1834:** Blank line used to separate nearby declarations and improve readability.
  **CN L1834:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1835:** This comment states: “Async load 4 bytes from fat buffer pointer to LDS.”, documenting the intent of the surrounding code.
  **CN L1835:** 该注释写道：“Async load 4 bytes from fat buffer pointer to LDS.”，用于说明周围代码的意图。
- **EN L1836:** This line contributes implementation detail or declarative structure to the file.
  **CN L1836:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1837-1848 / 第 1837-1848 行

```tablegen
1837:     ```
1838:   }];
1839: }
1840: 
1841: def ROCDL_GlobalLoadLDSOp :
1842:   ROCDL_IntrOp<"global.load.lds", [], [], [], 0, 0, 1, 0, [2, 3, 4], ["size", "offset", "aux"]> {
1843:   dag args = (ins Arg<ROCDLGlobalBuffer, "", [MemRead]>:$globalPtr,
1844:                  Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1845:                  I32Attr:$size,
1846:                  I32Attr:$offset,
1847:                  I32Attr:$aux);
1848:   let arguments = !con(args, baseArgs);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1837:** This line contributes implementation detail or declarative structure to the file.
  **CN L1837:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1838:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1838:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1839:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1839:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1840:** Blank line used to separate nearby declarations and improve readability.
  **CN L1840:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1841:** This TableGen `def` record introduces `ROCDL_GlobalLoadLDSOp`, which later participates in generated MLIR code.
  **CN L1841:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadLDSOp`，后续会参与生成的 MLIR 代码。
- **EN L1842:** This line contributes implementation detail or declarative structure to the file.
  **CN L1842:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1843:** This line contributes implementation detail or declarative structure to the file.
  **CN L1843:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1844:** This line contributes implementation detail or declarative structure to the file.
  **CN L1844:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1845:** This line contributes implementation detail or declarative structure to the file.
  **CN L1845:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1846:** This line contributes implementation detail or declarative structure to the file.
  **CN L1846:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1847:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1847:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1848:** This line contributes to the declaration or call of `con`.
  **CN L1848:** 这一行为 `con` 的声明或调用提供内容。

### Lines 1849-1860 / 第 1849-1860 行

```tablegen
1849:   let assemblyFormat = [{
1850:     $globalPtr `,`  $ldsPtr `,` $size `,` $offset `,` $aux
1851:     attr-dict
1852:   }];
1853:   let extraClassDefinition = [{
1854:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1855:       return {getGlobalPtr(), getLdsPtr()};
1856:     }
1857:   }];
1858: }
1859: 
1860: def ROCDL_GlobalLoadAsyncLDSOp :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1849:** This line contributes implementation detail or declarative structure to the file.
  **CN L1849:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1850:** This line contributes implementation detail or declarative structure to the file.
  **CN L1850:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1851:** This line contributes implementation detail or declarative structure to the file.
  **CN L1851:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1852:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1852:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1853:** This line contributes implementation detail or declarative structure to the file.
  **CN L1853:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1854:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1854:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1855:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1855:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1856:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1856:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1857:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1857:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1858:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1858:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1859:** Blank line used to separate nearby declarations and improve readability.
  **CN L1859:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1860:** This TableGen `def` record introduces `ROCDL_GlobalLoadAsyncLDSOp`, which later participates in generated MLIR code.
  **CN L1860:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadAsyncLDSOp`，后续会参与生成的 MLIR 代码。

### Lines 1861-1872 / 第 1861-1872 行

```tablegen
1861:   ROCDL_IntrOp<"global.load.async.lds", [], [], [], 0, 0, 1, 0, [2, 3, 4], ["size", "offset", "aux"]> {
1862:   dag args = (ins Arg<ROCDLGlobalBuffer, "", [MemRead]>:$globalPtr,
1863:                  Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1864:                  I32Attr:$size,
1865:                  I32Attr:$offset,
1866:                  I32Attr:$aux);
1867:   let arguments = !con(args, baseArgs);
1868:   let assemblyFormat = [{
1869:     $globalPtr `,`  $ldsPtr `,` $size `,` $offset `,` $aux
1870:     attr-dict `:` qualified(type($globalPtr)) `,` qualified(type($ldsPtr))
1871:   }];
1872:   let extraClassDefinition = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1861:** This line contributes implementation detail or declarative structure to the file.
  **CN L1861:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1862:** This line contributes implementation detail or declarative structure to the file.
  **CN L1862:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1863:** This line contributes implementation detail or declarative structure to the file.
  **CN L1863:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1864:** This line contributes implementation detail or declarative structure to the file.
  **CN L1864:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1865:** This line contributes implementation detail or declarative structure to the file.
  **CN L1865:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1866:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1866:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1867:** This line contributes to the declaration or call of `con`.
  **CN L1867:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1868:** This line contributes implementation detail or declarative structure to the file.
  **CN L1868:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1869:** This line contributes implementation detail or declarative structure to the file.
  **CN L1869:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1870:** This line contributes to the declaration or call of `qualified`.
  **CN L1870:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1871:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1871:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1872:** This line contributes implementation detail or declarative structure to the file.
  **CN L1872:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1873-1884 / 第 1873-1884 行

```tablegen
1873:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1874:       return {getGlobalPtr(), getLdsPtr()};
1875:     }
1876:   }];
1877: 
1878:   let summary = "Version of rocdl.load.async.to.lds specialized to global pointers";
1879:   let description = [{
1880:     This operation works identically to `rocdl.load.async.to.lds` except that the
1881:     global pointer argument is limited to pointers in address space 1 (pure global
1882:     pointers) instead of also allowing fat buffer pointers.
1883: 
1884:     Available on gfx9 and gfx10.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1873:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1873:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1874:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1874:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1875:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1875:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1876:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1876:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1877:** Blank line used to separate nearby declarations and improve readability.
  **CN L1877:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1878:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1878:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1879:** This line contributes implementation detail or declarative structure to the file.
  **CN L1879:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1880:** This line contributes implementation detail or declarative structure to the file.
  **CN L1880:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1881:** This line contributes implementation detail or declarative structure to the file.
  **CN L1881:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1882:** This line contributes implementation detail or declarative structure to the file.
  **CN L1882:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1883:** Blank line used to separate nearby declarations and improve readability.
  **CN L1883:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1884:** This line contributes implementation detail or declarative structure to the file.
  **CN L1884:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1885-1896 / 第 1885-1896 行

```tablegen
1885: 
1886:     For the operation introduced in gfx1250, see `rocdl.global.load.async.to.lds.bN`.
1887: 
1888:     Example:
1889:     ```mlir
1890:     // Async load from global pointer to LDS (address space 1 only).
1891:     rocdl.load.async.to.lds %global, %shared, 4, 0, 0 : !llvm.ptr<1>, !llvm.ptr<3>
1892:     ```
1893:   }];
1894: }
1895: 
1896: //===---------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1885:** Blank line used to separate nearby declarations and improve readability.
  **CN L1885:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1886:** This line contributes implementation detail or declarative structure to the file.
  **CN L1886:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1887:** Blank line used to separate nearby declarations and improve readability.
  **CN L1887:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1888:** This line contributes implementation detail or declarative structure to the file.
  **CN L1888:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1889:** This line contributes implementation detail or declarative structure to the file.
  **CN L1889:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1890:** This comment states: “Async load from global pointer to LDS (address space 1 only).”, documenting the intent of the surrounding code.
  **CN L1890:** 该注释写道：“Async load from global pointer to LDS (address space 1 only).”，用于说明周围代码的意图。
- **EN L1891:** This line contributes implementation detail or declarative structure to the file.
  **CN L1891:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1892:** This line contributes implementation detail or declarative structure to the file.
  **CN L1892:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1893:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1893:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1894:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1894:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1895:** Blank line used to separate nearby declarations and improve readability.
  **CN L1895:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1896:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1896:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 1897-1908 / 第 1897-1908 行

```tablegen
1897: // Async load to LDS intrinsic (available in GFX1250)
1898: //===---------------------------------------------------------------------===//
1899: 
1900: foreach bitsVal = [8, 32, 64, 128] in {
1901:   defvar bitsStr = "b" # !cast<string>(bitsVal);
1902:   def ROCDL_GlobalLoadAsyncToLDS # !toupper(bitsStr) # Op :
1903:     ROCDL_IntrOp<"global.load.async.to.lds." # bitsStr, [], [], [], 0, 0, 1, 0, [2, 3], ["offset", "aux"]> {
1904:     dag args = (ins Arg<ROCDLGlobalBuffer, "", [MemRead]>:$globalPtr,
1905:                    Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1906:                    I32Attr:$offset,
1907:                    I32Attr:$aux);
1908:     let arguments = !con(args, baseArgs);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1897:** This comment states: “Async load to LDS intrinsic (available in GFX1250)”, documenting the intent of the surrounding code.
  **CN L1897:** 该注释写道：“Async load to LDS intrinsic (available in GFX1250)”，用于说明周围代码的意图。
- **EN L1898:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1898:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1899:** Blank line used to separate nearby declarations and improve readability.
  **CN L1899:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1900:** This line contributes implementation detail or declarative structure to the file.
  **CN L1900:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1901:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1901:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1902:** This TableGen `def` record introduces `ROCDL_GlobalLoadAsyncToLDS`, which later participates in generated MLIR code.
  **CN L1902:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalLoadAsyncToLDS`，后续会参与生成的 MLIR 代码。
- **EN L1903:** This line contributes implementation detail or declarative structure to the file.
  **CN L1903:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1904:** This line contributes implementation detail or declarative structure to the file.
  **CN L1904:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1905:** This line contributes implementation detail or declarative structure to the file.
  **CN L1905:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1906:** This line contributes implementation detail or declarative structure to the file.
  **CN L1906:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1907:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1907:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1908:** This line contributes to the declaration or call of `con`.
  **CN L1908:** 这一行为 `con` 的声明或调用提供内容。

### Lines 1909-1920 / 第 1909-1920 行

```tablegen
1909:     let assemblyFormat = [{
1910:       $globalPtr `,`  $ldsPtr `,` $offset `,` $aux
1911:       attr-dict `:` qualified(type($globalPtr)) `,` qualified(type($ldsPtr))
1912:     }];
1913:     let description = [{
1914:       Asynchronously loads }] # !cast<string>(bitsVal) # [{ bits of data from a global memory pointer
1915:       to a Local Data Share (LDS) pointer.
1916: 
1917:       Available on gfx1250+.
1918: 
1919:       Example:
1920:       ```mlir
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1909:** This line contributes implementation detail or declarative structure to the file.
  **CN L1909:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1910:** This line contributes implementation detail or declarative structure to the file.
  **CN L1910:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1911:** This line contributes to the declaration or call of `qualified`.
  **CN L1911:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1912:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1912:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1913:** This line contributes implementation detail or declarative structure to the file.
  **CN L1913:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1914:** This line contributes implementation detail or declarative structure to the file.
  **CN L1914:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1915:** This line contributes to the declaration or call of `Share`.
  **CN L1915:** 这一行为 `Share` 的声明或调用提供内容。
- **EN L1916:** Blank line used to separate nearby declarations and improve readability.
  **CN L1916:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1917:** This line contributes implementation detail or declarative structure to the file.
  **CN L1917:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1918:** Blank line used to separate nearby declarations and improve readability.
  **CN L1918:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1919:** This line contributes implementation detail or declarative structure to the file.
  **CN L1919:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1920:** This line contributes implementation detail or declarative structure to the file.
  **CN L1920:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1921-1932 / 第 1921-1932 行

```tablegen
1921:       // Async }] # !cast<string>(bitsVal) # [{-bit load from global to LDS.
1922:       rocdl.global.load.async.to.lds.}] # bitsStr # [{ %src, %dst, 0, 0 : !llvm.ptr<1>, !llvm.ptr<3>
1923:       ```
1924:     }];
1925: 
1926:     let extraClassDefinition = [{
1927:       ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1928:         return {getGlobalPtr(), getLdsPtr()};
1929:       }
1930:     }];
1931:   }
1932: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1921:** This comment states: “Async }] # !cast<string>(bitsVal) # [{-bit load from global to LDS.”, documenting the intent of the surrounding code.
  **CN L1921:** 该注释写道：“Async }] # !cast<string>(bitsVal) # [{-bit load from global to LDS.”，用于说明周围代码的意图。
- **EN L1922:** This line contributes implementation detail or declarative structure to the file.
  **CN L1922:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1923:** This line contributes implementation detail or declarative structure to the file.
  **CN L1923:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1924:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1924:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1925:** Blank line used to separate nearby declarations and improve readability.
  **CN L1925:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1926:** This line contributes implementation detail or declarative structure to the file.
  **CN L1926:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1927:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1927:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1928:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1928:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1929:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1929:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1930:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1930:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1931:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1931:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1932:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1932:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 1933-1944 / 第 1933-1944 行

```tablegen
1933: 
1934: //===---------------------------------------------------------------------===//
1935: // Async store from LDS intrinsic (available in GFX1250)
1936: //===---------------------------------------------------------------------===//
1937: 
1938: foreach bitsVal = [8, 32, 64, 128] in {
1939:   defvar bitsStr = "b" # !cast<string>(bitsVal);
1940:   def ROCDL_GlobalStoreAsyncFromLDS # !toupper(bitsStr) # Op :
1941:     ROCDL_IntrOp<"global.store.async.from.lds." # bitsStr, [], [], [], 0, 0, 1, 0, [2, 3], ["offset", "aux"]> {
1942:     dag args = (ins Arg<ROCDLGlobalBuffer, "", [MemWrite]>:$globalPtr,
1943:                    Arg<ROCDLBufferLDS, "", [MemRead]>:$ldsPtr,
1944:                    I32Attr:$offset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1933:** Blank line used to separate nearby declarations and improve readability.
  **CN L1933:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1934:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1934:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1935:** This comment states: “Async store from LDS intrinsic (available in GFX1250)”, documenting the intent of the surrounding code.
  **CN L1935:** 该注释写道：“Async store from LDS intrinsic (available in GFX1250)”，用于说明周围代码的意图。
- **EN L1936:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1936:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1937:** Blank line used to separate nearby declarations and improve readability.
  **CN L1937:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1938:** This line contributes implementation detail or declarative structure to the file.
  **CN L1938:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1939:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1939:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1940:** This TableGen `def` record introduces `ROCDL_GlobalStoreAsyncFromLDS`, which later participates in generated MLIR code.
  **CN L1940:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalStoreAsyncFromLDS`，后续会参与生成的 MLIR 代码。
- **EN L1941:** This line contributes implementation detail or declarative structure to the file.
  **CN L1941:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1942:** This line contributes implementation detail or declarative structure to the file.
  **CN L1942:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1943:** This line contributes implementation detail or declarative structure to the file.
  **CN L1943:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1944:** This line contributes implementation detail or declarative structure to the file.
  **CN L1944:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1945-1956 / 第 1945-1956 行

```tablegen
1945:                    I32Attr:$aux);
1946:     let arguments = !con(args, baseArgs);
1947:     let assemblyFormat = [{
1948:       $globalPtr `,`  $ldsPtr `,` $offset `,` $aux
1949:       attr-dict `:` qualified(type($globalPtr)) `,` qualified(type($ldsPtr))
1950:     }];
1951:     let description = [{
1952:       Asynchronously stores }] # !cast<string>(bitsVal) # [{ bits of data from a Local Data Share (LDS)
1953:       pointer to a global memory pointer.
1954: 
1955:       Available on gfx1250+.
1956: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1945:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1945:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1946:** This line contributes to the declaration or call of `con`.
  **CN L1946:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1947:** This line contributes implementation detail or declarative structure to the file.
  **CN L1947:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1948:** This line contributes implementation detail or declarative structure to the file.
  **CN L1948:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1949:** This line contributes to the declaration or call of `qualified`.
  **CN L1949:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1950:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1950:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1951:** This line contributes implementation detail or declarative structure to the file.
  **CN L1951:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1952:** This line contributes to the declaration or call of `Share`.
  **CN L1952:** 这一行为 `Share` 的声明或调用提供内容。
- **EN L1953:** This line contributes implementation detail or declarative structure to the file.
  **CN L1953:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1954:** Blank line used to separate nearby declarations and improve readability.
  **CN L1954:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1955:** This line contributes implementation detail or declarative structure to the file.
  **CN L1955:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1956:** Blank line used to separate nearby declarations and improve readability.
  **CN L1956:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1957-1968 / 第 1957-1968 行

```tablegen
1957:       Example:
1958:       ```mlir
1959:       // Async }] # !cast<string>(bitsVal) # [{-bit store from LDS to global.
1960:       rocdl.global.store.async.from.lds.}] # bitsStr # [{ %dst, %src, 0, 0 : !llvm.ptr<1>, !llvm.ptr<3>
1961:       ```
1962:     }];
1963: 
1964:     let extraClassDefinition = [{
1965:       ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
1966:         return {getGlobalPtr(), getLdsPtr()};
1967:       }
1968:     }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1957:** This line contributes implementation detail or declarative structure to the file.
  **CN L1957:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1958:** This line contributes implementation detail or declarative structure to the file.
  **CN L1958:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1959:** This comment states: “Async }] # !cast<string>(bitsVal) # [{-bit store from LDS to global.”, documenting the intent of the surrounding code.
  **CN L1959:** 该注释写道：“Async }] # !cast<string>(bitsVal) # [{-bit store from LDS to global.”，用于说明周围代码的意图。
- **EN L1960:** This line contributes implementation detail or declarative structure to the file.
  **CN L1960:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1961:** This line contributes implementation detail or declarative structure to the file.
  **CN L1961:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1962:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1962:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1963:** Blank line used to separate nearby declarations and improve readability.
  **CN L1963:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1964:** This line contributes implementation detail or declarative structure to the file.
  **CN L1964:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1965:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1965:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L1966:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1966:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1967:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1967:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1968:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1968:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1969-1980 / 第 1969-1980 行

```tablegen
1969:   }
1970: }
1971: 
1972: foreach bitsVal = [8, 32, 64, 128] in {
1973:   defvar bitsStr = "b" # !cast<string>(bitsVal);
1974:   def ROCDL_ClusterLoadAsyncToLDS # !toupper(bitsStr) # Op :
1975:     ROCDL_IntrOp<"cluster.load.async.to.lds." # bitsStr, [], [], [], 0, 0, 1, 0, [2, 3], ["offset", "cpol"]> {
1976:     dag args = (ins Arg<ROCDLGlobalBuffer, "", [MemRead]>:$globalPtr,
1977:                    Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
1978:                    I32Attr:$offset,
1979:                    I32Attr:$cpol,
1980:                    I32:$mask);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1969:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1969:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1970:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1970:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1971:** Blank line used to separate nearby declarations and improve readability.
  **CN L1971:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1972:** This line contributes implementation detail or declarative structure to the file.
  **CN L1972:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1973:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1973:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1974:** This TableGen `def` record introduces `ROCDL_ClusterLoadAsyncToLDS`, which later participates in generated MLIR code.
  **CN L1974:** 该 TableGen `def` 记录引入了 `ROCDL_ClusterLoadAsyncToLDS`，后续会参与生成的 MLIR 代码。
- **EN L1975:** This line contributes implementation detail or declarative structure to the file.
  **CN L1975:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1976:** This line contributes implementation detail or declarative structure to the file.
  **CN L1976:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1977:** This line contributes implementation detail or declarative structure to the file.
  **CN L1977:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1978:** This line contributes implementation detail or declarative structure to the file.
  **CN L1978:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1979:** This line contributes implementation detail or declarative structure to the file.
  **CN L1979:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1980:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1980:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1981-1992 / 第 1981-1992 行

```tablegen
1981:     let arguments = !con(args, baseArgs);
1982:     let assemblyFormat = [{
1983:       $globalPtr `,`  $ldsPtr `,` $offset `,` $cpol `,` $mask
1984:       attr-dict `:` qualified(type($globalPtr)) `,` qualified(type($ldsPtr))
1985:     }];
1986:     let description = [{
1987:       Broadcasts memory load of }] # !cast<string>(bitsVal) # [{ bits of data for a cluster of workgroups.
1988: 
1989:       Available on gfx1250+.
1990: 
1991:       Example:
1992:       ```mlir
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1981:** This line contributes to the declaration or call of `con`.
  **CN L1981:** 这一行为 `con` 的声明或调用提供内容。
- **EN L1982:** This line contributes implementation detail or declarative structure to the file.
  **CN L1982:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1983:** This line contributes implementation detail or declarative structure to the file.
  **CN L1983:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1984:** This line contributes to the declaration or call of `qualified`.
  **CN L1984:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L1985:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1985:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1986:** This line contributes implementation detail or declarative structure to the file.
  **CN L1986:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1987:** This line contributes implementation detail or declarative structure to the file.
  **CN L1987:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1988:** Blank line used to separate nearby declarations and improve readability.
  **CN L1988:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1989:** This line contributes implementation detail or declarative structure to the file.
  **CN L1989:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1990:** Blank line used to separate nearby declarations and improve readability.
  **CN L1990:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1991:** This line contributes implementation detail or declarative structure to the file.
  **CN L1991:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1992:** This line contributes implementation detail or declarative structure to the file.
  **CN L1992:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1993-2004 / 第 1993-2004 行

```tablegen
1993:       // Cluster broadcast }] # !cast<string>(bitsVal) # [{-bit load to LDS.
1994:       rocdl.cluster.load.async.to.lds.}] # bitsStr # [{ %src, %dst, 0, 0, %mask : !llvm.ptr<1>, !llvm.ptr<3>
1995:       ```
1996:     }];
1997: 
1998:     let extraClassDefinition = [{
1999:       ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2000:         return {getGlobalPtr(), getLdsPtr()};
2001:       }
2002:     }];
2003:   }
2004: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1993:** This comment states: “Cluster broadcast }] # !cast<string>(bitsVal) # [{-bit load to LDS.”, documenting the intent of the surrounding code.
  **CN L1993:** 该注释写道：“Cluster broadcast }] # !cast<string>(bitsVal) # [{-bit load to LDS.”，用于说明周围代码的意图。
- **EN L1994:** This line contributes implementation detail or declarative structure to the file.
  **CN L1994:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1995:** This line contributes implementation detail or declarative structure to the file.
  **CN L1995:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1996:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1996:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1997:** Blank line used to separate nearby declarations and improve readability.
  **CN L1997:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1998:** This line contributes implementation detail or declarative structure to the file.
  **CN L1998:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1999:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L1999:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2000:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2000:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2001:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2001:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2002:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2002:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2003:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2003:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2004:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2004:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2005-2016 / 第 2005-2016 行

```tablegen
2005: 
2006: //===---------------------------------------------------------------------===//
2007: // Tensor load/store intrinsics (available in GFX1250)
2008: //===---------------------------------------------------------------------===//
2009: 
2010: // Base class for tensor load/store operations with 4 descriptor groups.
2011: class ROCDL_TensorLDSIntrOp<string mnemonic> :
2012:   ROCDL_IntrOp<mnemonic, [], [], [], 0, 0, 1, 0, [5], ["cachePolicy"]> {
2013:   dag args = (ins ROCDL_V4I32Type:$dgroup0, ROCDL_V8I32Type:$dgroup1,
2014:                   ROCDL_V4I32Type:$dgroup2, ROCDL_V4I32Type:$dgroup3,
2015:                   ROCDL_V8I32Type:$dgroup4, I32Attr:$cachePolicy);
2016:   let arguments = !con(args, baseArgs);
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2005:** Blank line used to separate nearby declarations and improve readability.
  **CN L2005:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2006:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2006:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2007:** This comment states: “Tensor load/store intrinsics (available in GFX1250)”, documenting the intent of the surrounding code.
  **CN L2007:** 该注释写道：“Tensor load/store intrinsics (available in GFX1250)”，用于说明周围代码的意图。
- **EN L2008:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2008:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2009:** Blank line used to separate nearby declarations and improve readability.
  **CN L2009:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2010:** This comment states: “Base class for tensor load/store operations with 4 descriptor groups.”, documenting the intent of the surrounding code.
  **CN L2010:** 该注释写道：“Base class for tensor load/store operations with 4 descriptor groups.”，用于说明周围代码的意图。
- **EN L2011:** This TableGen `class` record introduces `ROCDL_TensorLDSIntrOp`, which later participates in generated MLIR code.
  **CN L2011:** 该 TableGen `class` 记录引入了 `ROCDL_TensorLDSIntrOp`，后续会参与生成的 MLIR 代码。
- **EN L2012:** This line contributes implementation detail or declarative structure to the file.
  **CN L2012:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2013:** This line contributes implementation detail or declarative structure to the file.
  **CN L2013:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2014:** This line contributes implementation detail or declarative structure to the file.
  **CN L2014:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2015:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2015:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2016:** This line contributes to the declaration or call of `con`.
  **CN L2016:** 这一行为 `con` 的声明或调用提供内容。

### Lines 2017-2028 / 第 2017-2028 行

```tablegen
2017:   let summary = "Base class for ROCDL tensor load/store to/from LDS.";
2018:   let description = [{
2019:     Moves tiles of tensor data between global memory and LDS. The tile is
2020:     described by the $dgroup descriptors. 5 $dgroup descriptors allows for
2021:     movement of up to 5D tensors. $cachePolicy describes the memory scope and an
2022:     indicator of expected data re-use.
2023: 
2024:     This op is for gfx1250+ architectures.
2025: 
2026:     Example:
2027:     ```mlir
2028:     // Tensor load from global memory to LDS using 4 descriptor groups.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2017:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2017:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2018:** This line contributes implementation detail or declarative structure to the file.
  **CN L2018:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2019:** This line contributes implementation detail or declarative structure to the file.
  **CN L2019:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2020:** This line contributes implementation detail or declarative structure to the file.
  **CN L2020:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2021:** This line contributes implementation detail or declarative structure to the file.
  **CN L2021:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2022:** This line contributes implementation detail or declarative structure to the file.
  **CN L2022:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2023:** Blank line used to separate nearby declarations and improve readability.
  **CN L2023:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2024:** This line contributes implementation detail or declarative structure to the file.
  **CN L2024:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2025:** Blank line used to separate nearby declarations and improve readability.
  **CN L2025:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2026:** This line contributes implementation detail or declarative structure to the file.
  **CN L2026:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2027:** This line contributes implementation detail or declarative structure to the file.
  **CN L2027:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2028:** This comment states: “Tensor load from global memory to LDS using 4 descriptor groups.”, documenting the intent of the surrounding code.
  **CN L2028:** 该注释写道：“Tensor load from global memory to LDS using 4 descriptor groups.”，用于说明周围代码的意图。

### Lines 2029-2040 / 第 2029-2040 行

```tablegen
2029:     rocdl.tensor.load.to.lds %dg0, %dg1, %dg2, %dg3 cachepolicy 0 : vector<4xi32>, vector<8xi32>
2030: 
2031:     // Tensor store from LDS to global memory using 4 descriptor groups.
2032:     rocdl.tensor.store.from.lds %dg0, %dg1, %dg2, %dg3 cachepolicy 0 : vector<4xi32>, vector<8xi32>
2033:     ```
2034:   }];
2035:   let assemblyFormat = [{
2036:     attr-dict operands `cachepolicy` $cachePolicy `:` type($dgroup0) `,` type($dgroup1)
2037:   }];
2038:   let extraClassDefinition = [{
2039:     SmallVector<Value> $cppClass::getAccessedOperands() {
2040:       return {getDgroup0(), getDgroup1(), getDgroup2(), getDgroup3()};
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2029:** This line contributes implementation detail or declarative structure to the file.
  **CN L2029:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2030:** Blank line used to separate nearby declarations and improve readability.
  **CN L2030:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2031:** This comment states: “Tensor store from LDS to global memory using 4 descriptor groups.”, documenting the intent of the surrounding code.
  **CN L2031:** 该注释写道：“Tensor store from LDS to global memory using 4 descriptor groups.”，用于说明周围代码的意图。
- **EN L2032:** This line contributes implementation detail or declarative structure to the file.
  **CN L2032:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2033:** This line contributes implementation detail or declarative structure to the file.
  **CN L2033:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2034:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2034:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2035:** This line contributes implementation detail or declarative structure to the file.
  **CN L2035:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2036:** This line contributes to the declaration or call of `type`.
  **CN L2036:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2037:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2037:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2038:** This line contributes implementation detail or declarative structure to the file.
  **CN L2038:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2039:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2039:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2040:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2040:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2041-2052 / 第 2041-2052 行

```tablegen
2041:     }
2042:   }];
2043: }
2044: 
2045: // Tensor load and store operations
2046: def ROCDL_TensorLoadToLDSOp : ROCDL_TensorLDSIntrOp<"tensor.load.to.lds">;
2047: def ROCDL_TensorStoreFromLDSOp : ROCDL_TensorLDSIntrOp<"tensor.store.from.lds">;
2048: 
2049: //===---------------------------------------------------------------------===//
2050: // Operations on raw buffer resources (stride of 0, bounds checks either off or in
2051: // raw buffer mode).
2052: //===---------------------------------------------------------------------===//
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2041:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2041:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2042:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2042:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2043:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2043:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2044:** Blank line used to separate nearby declarations and improve readability.
  **CN L2044:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2045:** This comment states: “Tensor load and store operations”, documenting the intent of the surrounding code.
  **CN L2045:** 该注释写道：“Tensor load and store operations”，用于说明周围代码的意图。
- **EN L2046:** This TableGen `def` record introduces `ROCDL_TensorLoadToLDSOp`, which later participates in generated MLIR code.
  **CN L2046:** 该 TableGen `def` 记录引入了 `ROCDL_TensorLoadToLDSOp`，后续会参与生成的 MLIR 代码。
- **EN L2047:** This TableGen `def` record introduces `ROCDL_TensorStoreFromLDSOp`, which later participates in generated MLIR code.
  **CN L2047:** 该 TableGen `def` 记录引入了 `ROCDL_TensorStoreFromLDSOp`，后续会参与生成的 MLIR 代码。
- **EN L2048:** Blank line used to separate nearby declarations and improve readability.
  **CN L2048:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2049:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2049:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2050:** This comment states: “Operations on raw buffer resources (stride of 0, bounds checks either off or in”, documenting the intent of the surrounding code.
  **CN L2050:** 该注释写道：“Operations on raw buffer resources (stride of 0, bounds checks either off or in”，用于说明周围代码的意图。
- **EN L2051:** This comment states: “raw buffer mode).”, documenting the intent of the surrounding code.
  **CN L2051:** 该注释写道：“raw buffer mode).”，用于说明周围代码的意图。
- **EN L2052:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2052:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 2053-2064 / 第 2053-2064 行

```tablegen
2053: 
2054: def ROCDLBufferRsrc : LLVM_PointerInAddressSpace<8>;
2055: 
2056: def ROCDL_MakeBufferRsrcOp :
2057:   ROCDL_IntrOp<"make.buffer.rsrc", [0], [0], [Pure], 1>,
2058:   Arguments<(ins LLVM_AnyPointer:$base,
2059:                  I16:$stride,
2060:                  I64:$numRecords,
2061:                  I32:$flags)> {
2062:   let results = (outs LLVM_AnyPointer:$res);
2063:   let assemblyFormat = "operands attr-dict `:` type($base) `to` type($res)";
2064: }
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2053:** Blank line used to separate nearby declarations and improve readability.
  **CN L2053:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2054:** This TableGen `def` record introduces `ROCDLBufferRsrc`, which later participates in generated MLIR code.
  **CN L2054:** 该 TableGen `def` 记录引入了 `ROCDLBufferRsrc`，后续会参与生成的 MLIR 代码。
- **EN L2055:** Blank line used to separate nearby declarations and improve readability.
  **CN L2055:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2056:** This TableGen `def` record introduces `ROCDL_MakeBufferRsrcOp`, which later participates in generated MLIR code.
  **CN L2056:** 该 TableGen `def` 记录引入了 `ROCDL_MakeBufferRsrcOp`，后续会参与生成的 MLIR 代码。
- **EN L2057:** This line contributes implementation detail or declarative structure to the file.
  **CN L2057:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2058:** This line contributes implementation detail or declarative structure to the file.
  **CN L2058:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2059:** This line contributes implementation detail or declarative structure to the file.
  **CN L2059:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2060:** This line contributes implementation detail or declarative structure to the file.
  **CN L2060:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2061:** This line contributes implementation detail or declarative structure to the file.
  **CN L2061:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2062:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2062:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2063:** This line contributes to the declaration or call of `type`.
  **CN L2063:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2064:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2064:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2065-2076 / 第 2065-2076 行

```tablegen
2065: 
2066: def ROCDL_RawPtrBufferLoadOp :
2067:   ROCDL_IntrOp<"raw.ptr.buffer.load", [0], [], [], 1, 0, 1> {
2068:   dag args = (ins Arg<ROCDLBufferRsrc, "", [MemRead]>:$rsrc,
2069:                   I32:$offset,
2070:                   I32:$soffset,
2071:                   I32:$aux);
2072:   let arguments = !con(args, baseArgs);
2073:   let assemblyFormat = "operands attr-dict `:` type($res)";
2074:   let extraClassDefinition = [{
2075:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2076:       return {getRsrc()};
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2065:** Blank line used to separate nearby declarations and improve readability.
  **CN L2065:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2066:** This TableGen `def` record introduces `ROCDL_RawPtrBufferLoadOp`, which later participates in generated MLIR code.
  **CN L2066:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferLoadOp`，后续会参与生成的 MLIR 代码。
- **EN L2067:** This line contributes implementation detail or declarative structure to the file.
  **CN L2067:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2068:** This line contributes implementation detail or declarative structure to the file.
  **CN L2068:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2069:** This line contributes implementation detail or declarative structure to the file.
  **CN L2069:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2070:** This line contributes implementation detail or declarative structure to the file.
  **CN L2070:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2071:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2071:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2072:** This line contributes to the declaration or call of `con`.
  **CN L2072:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2073:** This line contributes to the declaration or call of `type`.
  **CN L2073:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2074:** This line contributes implementation detail or declarative structure to the file.
  **CN L2074:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2075:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2075:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2076:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2076:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2077-2088 / 第 2077-2088 行

```tablegen
2077:     }
2078:   }];
2079: }
2080: 
2081: def ROCDL_RawPtrBufferLoadLdsOp :
2082:   ROCDL_IntrOp<"raw.ptr.buffer.load.lds", [], [], [], 0, 0, 1> {
2083:   dag args = (ins Arg<ROCDLBufferRsrc, "", [MemRead]>:$rsrc,
2084:                   Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
2085:                   I32:$size,
2086:                   I32:$voffset,
2087:                   I32:$soffset,
2088:                   I32:$offset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2077:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2077:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2078:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2078:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2079:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2079:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2080:** Blank line used to separate nearby declarations and improve readability.
  **CN L2080:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2081:** This TableGen `def` record introduces `ROCDL_RawPtrBufferLoadLdsOp`, which later participates in generated MLIR code.
  **CN L2081:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferLoadLdsOp`，后续会参与生成的 MLIR 代码。
- **EN L2082:** This line contributes implementation detail or declarative structure to the file.
  **CN L2082:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2083:** This line contributes implementation detail or declarative structure to the file.
  **CN L2083:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2084:** This line contributes implementation detail or declarative structure to the file.
  **CN L2084:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2085:** This line contributes implementation detail or declarative structure to the file.
  **CN L2085:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2086:** This line contributes implementation detail or declarative structure to the file.
  **CN L2086:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2087:** This line contributes implementation detail or declarative structure to the file.
  **CN L2087:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2088:** This line contributes implementation detail or declarative structure to the file.
  **CN L2088:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2089-2100 / 第 2089-2100 行

```tablegen
2089:                   I32:$aux);
2090:   let arguments = !con(args, baseArgs);
2091:   let assemblyFormat = "operands attr-dict";
2092:   let extraClassDefinition = [{
2093:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2094:       return {getRsrc(), getLdsPtr()};
2095:     }
2096:   }];
2097: }
2098: 
2099: def ROCDL_RawPtrBufferLoadAsyncLdsOp :
2100:   ROCDL_IntrOp<"raw.ptr.buffer.load.async.lds", [], [], [], 0, 0, 1> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2089:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2089:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2090:** This line contributes to the declaration or call of `con`.
  **CN L2090:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2091:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2091:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2092:** This line contributes implementation detail or declarative structure to the file.
  **CN L2092:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2093:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2093:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2094:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2094:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2095:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2095:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2096:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2096:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2097:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2097:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2098:** Blank line used to separate nearby declarations and improve readability.
  **CN L2098:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2099:** This TableGen `def` record introduces `ROCDL_RawPtrBufferLoadAsyncLdsOp`, which later participates in generated MLIR code.
  **CN L2099:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferLoadAsyncLdsOp`，后续会参与生成的 MLIR 代码。
- **EN L2100:** This line contributes implementation detail or declarative structure to the file.
  **CN L2100:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2101-2112 / 第 2101-2112 行

```tablegen
2101:   dag args = (ins Arg<ROCDLBufferRsrc, "", [MemRead]>:$rsrc,
2102:                   Arg<ROCDLBufferLDS, "", [MemWrite]>:$ldsPtr,
2103:                   I32:$size,
2104:                   I32:$voffset,
2105:                   I32:$soffset,
2106:                   I32:$offset,
2107:                   I32:$aux);
2108:   let arguments = !con(args, baseArgs);
2109:   let assemblyFormat = "operands attr-dict";
2110:   let extraClassDefinition = [{
2111:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2112:       return {getRsrc(), getLdsPtr()};
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2101:** This line contributes implementation detail or declarative structure to the file.
  **CN L2101:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2102:** This line contributes implementation detail or declarative structure to the file.
  **CN L2102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2103:** This line contributes implementation detail or declarative structure to the file.
  **CN L2103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2104:** This line contributes implementation detail or declarative structure to the file.
  **CN L2104:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2105:** This line contributes implementation detail or declarative structure to the file.
  **CN L2105:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2106:** This line contributes implementation detail or declarative structure to the file.
  **CN L2106:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2107:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2107:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2108:** This line contributes to the declaration or call of `con`.
  **CN L2108:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2109:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2109:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2110:** This line contributes implementation detail or declarative structure to the file.
  **CN L2110:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2111:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2111:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2112:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2112:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2113-2124 / 第 2113-2124 行

```tablegen
2113:     }
2114:   }];
2115:   let summary = "Async variant of raw.ptr.buffer.load.lds";
2116:   let description = [{
2117:     Load from a buffer resource `rsrc` to `ldsPtr`, which must be uniform.
2118: 
2119:     See `rocdl.load.async.to.lds` for overall semantics of such loads, noting that
2120:     here `voffset` can be lane-varying and that `rsrc` (which holds the base addres)
2121:     must, as always, be uniform.
2122: 
2123:     Available on gfx9 and gfx10.
2124: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2113:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2113:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2114:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2114:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2115:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2115:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2116:** This line contributes implementation detail or declarative structure to the file.
  **CN L2116:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2117:** This line contributes implementation detail or declarative structure to the file.
  **CN L2117:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2118:** Blank line used to separate nearby declarations and improve readability.
  **CN L2118:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2119:** This line contributes implementation detail or declarative structure to the file.
  **CN L2119:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2120:** This line contributes implementation detail or declarative structure to the file.
  **CN L2120:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2121:** This line contributes implementation detail or declarative structure to the file.
  **CN L2121:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2122:** Blank line used to separate nearby declarations and improve readability.
  **CN L2122:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2123:** This line contributes implementation detail or declarative structure to the file.
  **CN L2123:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2124:** Blank line used to separate nearby declarations and improve readability.
  **CN L2124:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2125-2136 / 第 2125-2136 行

```tablegen
2125:     Example:
2126:     ```mlir
2127:     // Async buffer load to LDS via buffer resource pointer.
2128:     rocdl.raw.ptr.buffer.load.async.lds %rsrc, %ldsPtr, %size, %voffset, %soffset, %offset, %aux
2129:     ```
2130:   }];
2131: }
2132: 
2133: def ROCDL_RawPtrBufferStoreOp :
2134:   ROCDL_IntrOp<"raw.ptr.buffer.store", [], [0], [], 0, 0, 1> {
2135:   dag args = (ins LLVM_Type:$vdata,
2136:                   Arg<ROCDLBufferRsrc, "", [MemWrite]>:$rsrc,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2125:** This line contributes implementation detail or declarative structure to the file.
  **CN L2125:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2126:** This line contributes implementation detail or declarative structure to the file.
  **CN L2126:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2127:** This comment states: “Async buffer load to LDS via buffer resource pointer.”, documenting the intent of the surrounding code.
  **CN L2127:** 该注释写道：“Async buffer load to LDS via buffer resource pointer.”，用于说明周围代码的意图。
- **EN L2128:** This line contributes implementation detail or declarative structure to the file.
  **CN L2128:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2129:** This line contributes implementation detail or declarative structure to the file.
  **CN L2129:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2130:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2130:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2131:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2131:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2132:** Blank line used to separate nearby declarations and improve readability.
  **CN L2132:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2133:** This TableGen `def` record introduces `ROCDL_RawPtrBufferStoreOp`, which later participates in generated MLIR code.
  **CN L2133:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferStoreOp`，后续会参与生成的 MLIR 代码。
- **EN L2134:** This line contributes implementation detail or declarative structure to the file.
  **CN L2134:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2135:** This line contributes implementation detail or declarative structure to the file.
  **CN L2135:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2136:** This line contributes implementation detail or declarative structure to the file.
  **CN L2136:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2137-2148 / 第 2137-2148 行

```tablegen
2137:                   I32:$offset,
2138:                   I32:$soffset,
2139:                   I32:$aux);
2140:   let arguments = !con(args, baseArgs);
2141:   let assemblyFormat = "operands attr-dict `:` type($vdata)";
2142:   let extraClassDefinition = [{
2143:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2144:       return {getRsrc()};
2145:     }
2146:   }];
2147: 
2148: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2137:** This line contributes implementation detail or declarative structure to the file.
  **CN L2137:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2138:** This line contributes implementation detail or declarative structure to the file.
  **CN L2138:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2139:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2139:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2140:** This line contributes to the declaration or call of `con`.
  **CN L2140:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2141:** This line contributes to the declaration or call of `type`.
  **CN L2141:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2142:** This line contributes implementation detail or declarative structure to the file.
  **CN L2142:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2143:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2143:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2144:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2144:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2145:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2145:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2146:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2146:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2147:** Blank line used to separate nearby declarations and improve readability.
  **CN L2147:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2148:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2148:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2149-2160 / 第 2149-2160 行

```tablegen
2149: 
2150: def ROCDL_RawPtrBufferAtomicCmpSwap :
2151:   ROCDL_IntrOp<"raw.ptr.buffer.atomic.cmpswap",
2152:     [0], [], [AllTypesMatch<["res", "src", "cmp"]>], 1, 0, 1> {
2153:   dag args = (ins LLVM_Type:$src,
2154:                   LLVM_Type:$cmp,
2155:                   Arg<ROCDLBufferRsrc, "", [MemRead, MemWrite]>:$rsrc,
2156:                   I32:$offset,
2157:                   I32:$soffset,
2158:                   I32:$aux);
2159:   let arguments = !con(args, baseArgs);
2160:   let assemblyFormat = "operands attr-dict `:` type($res)";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2149:** Blank line used to separate nearby declarations and improve readability.
  **CN L2149:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2150:** This TableGen `def` record introduces `ROCDL_RawPtrBufferAtomicCmpSwap`, which later participates in generated MLIR code.
  **CN L2150:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferAtomicCmpSwap`，后续会参与生成的 MLIR 代码。
- **EN L2151:** This line contributes implementation detail or declarative structure to the file.
  **CN L2151:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2152:** This line contributes implementation detail or declarative structure to the file.
  **CN L2152:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2153:** This line contributes implementation detail or declarative structure to the file.
  **CN L2153:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2154:** This line contributes implementation detail or declarative structure to the file.
  **CN L2154:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2155:** This line contributes implementation detail or declarative structure to the file.
  **CN L2155:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2156:** This line contributes implementation detail or declarative structure to the file.
  **CN L2156:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2157:** This line contributes implementation detail or declarative structure to the file.
  **CN L2157:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2158:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2158:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2159:** This line contributes to the declaration or call of `con`.
  **CN L2159:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2160:** This line contributes to the declaration or call of `type`.
  **CN L2160:** 这一行为 `type` 的声明或调用提供内容。

### Lines 2161-2172 / 第 2161-2172 行

```tablegen
2161:   let extraClassDefinition = [{
2162:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2163:       return {getRsrc()};
2164:     }
2165:   }];
2166: }
2167: 
2168: class ROCDL_RawPtrBufferAtomicNoRet<string op> :
2169:   ROCDL_IntrOp<"raw.ptr.buffer.atomic." # op, [], [0], [], 0, 0, 1> {
2170:   dag args = (ins LLVM_Type:$vdata,
2171:                   Arg<ROCDLBufferRsrc, "", [MemRead, MemWrite]>:$rsrc,
2172:                   I32:$offset,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2161:** This line contributes implementation detail or declarative structure to the file.
  **CN L2161:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2162:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2162:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2163:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2163:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2164:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2164:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2165:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2165:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2166:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2166:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2167:** Blank line used to separate nearby declarations and improve readability.
  **CN L2167:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2168:** This TableGen `class` record introduces `ROCDL_RawPtrBufferAtomicNoRet`, which later participates in generated MLIR code.
  **CN L2168:** 该 TableGen `class` 记录引入了 `ROCDL_RawPtrBufferAtomicNoRet`，后续会参与生成的 MLIR 代码。
- **EN L2169:** This line contributes implementation detail or declarative structure to the file.
  **CN L2169:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2170:** This line contributes implementation detail or declarative structure to the file.
  **CN L2170:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2171:** This line contributes implementation detail or declarative structure to the file.
  **CN L2171:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2172:** This line contributes implementation detail or declarative structure to the file.
  **CN L2172:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2173-2184 / 第 2173-2184 行

```tablegen
2173:                   I32:$soffset,
2174:                   I32:$aux);
2175:   let arguments = !con(args, baseArgs);
2176:   let assemblyFormat = "operands attr-dict `:` type($vdata)";
2177:   let extraClassDefinition = [{
2178:     ::llvm::SmallVector<::mlir::Value> $cppClass::getAccessedOperands() {
2179:       return {getRsrc()};
2180:     }
2181:   }];
2182: }
2183: 
2184: def ROCDL_RawPtrBufferAtomicFmaxOp : ROCDL_RawPtrBufferAtomicNoRet<"fmax">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2173:** This line contributes implementation detail or declarative structure to the file.
  **CN L2173:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2174:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2174:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2175:** This line contributes to the declaration or call of `con`.
  **CN L2175:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2176:** This line contributes to the declaration or call of `type`.
  **CN L2176:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2177:** This line contributes implementation detail or declarative structure to the file.
  **CN L2177:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2178:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2178:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2179:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2179:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2180:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2180:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2181:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2181:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2182:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2182:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2183:** Blank line used to separate nearby declarations and improve readability.
  **CN L2183:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2184:** This TableGen `def` record introduces `ROCDL_RawPtrBufferAtomicFmaxOp`, which later participates in generated MLIR code.
  **CN L2184:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferAtomicFmaxOp`，后续会参与生成的 MLIR 代码。

### Lines 2185-2196 / 第 2185-2196 行

```tablegen
2185: def ROCDL_RawPtrBufferAtomicSmaxOp : ROCDL_RawPtrBufferAtomicNoRet<"smax">;
2186: def ROCDL_RawPtrBufferAtomicUminOp : ROCDL_RawPtrBufferAtomicNoRet<"umin">;
2187: // Note: not supported on all architectures
2188: def ROCDL_RawPtrBufferAtomicFaddOp : ROCDL_RawPtrBufferAtomicNoRet<"fadd">;
2189: 
2190: //===---------------------------------------------------------------------===//
2191: // Raw buffer load/store intrinsics
2192: 
2193: def ROCDL_RawBufferLoadOp :
2194:   ROCDL_IntrOp<"raw.buffer.load", [0], [], [], 1>,
2195:   Arguments<(ins LLVM_Type:$rsrc,
2196:                  LLVM_Type:$offset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2185:** This TableGen `def` record introduces `ROCDL_RawPtrBufferAtomicSmaxOp`, which later participates in generated MLIR code.
  **CN L2185:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferAtomicSmaxOp`，后续会参与生成的 MLIR 代码。
- **EN L2186:** This TableGen `def` record introduces `ROCDL_RawPtrBufferAtomicUminOp`, which later participates in generated MLIR code.
  **CN L2186:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferAtomicUminOp`，后续会参与生成的 MLIR 代码。
- **EN L2187:** This comment states: “Note: not supported on all architectures”, documenting the intent of the surrounding code.
  **CN L2187:** 该注释写道：“Note: not supported on all architectures”，用于说明周围代码的意图。
- **EN L2188:** This TableGen `def` record introduces `ROCDL_RawPtrBufferAtomicFaddOp`, which later participates in generated MLIR code.
  **CN L2188:** 该 TableGen `def` 记录引入了 `ROCDL_RawPtrBufferAtomicFaddOp`，后续会参与生成的 MLIR 代码。
- **EN L2189:** Blank line used to separate nearby declarations and improve readability.
  **CN L2189:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2190:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2190:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2191:** This comment states: “Raw buffer load/store intrinsics”, documenting the intent of the surrounding code.
  **CN L2191:** 该注释写道：“Raw buffer load/store intrinsics”，用于说明周围代码的意图。
- **EN L2192:** Blank line used to separate nearby declarations and improve readability.
  **CN L2192:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2193:** This TableGen `def` record introduces `ROCDL_RawBufferLoadOp`, which later participates in generated MLIR code.
  **CN L2193:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferLoadOp`，后续会参与生成的 MLIR 代码。
- **EN L2194:** This line contributes implementation detail or declarative structure to the file.
  **CN L2194:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2195:** This line contributes implementation detail or declarative structure to the file.
  **CN L2195:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2196:** This line contributes implementation detail or declarative structure to the file.
  **CN L2196:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2197-2208 / 第 2197-2208 行

```tablegen
2197:                  LLVM_Type:$soffset,
2198:                  LLVM_Type:$aux)> {
2199:   let hasCustomAssemblyFormat = 1;
2200: }
2201: 
2202: def ROCDL_RawBufferStoreOp :
2203:   ROCDL_IntrOp<"raw.buffer.store", [], [0], [], 0>,
2204:   Arguments<(ins LLVM_Type:$vdata,
2205:                  LLVM_Type:$rsrc,
2206:                  LLVM_Type:$offset,
2207:                  LLVM_Type:$soffset,
2208:                  LLVM_Type:$aux)>{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2197:** This line contributes implementation detail or declarative structure to the file.
  **CN L2197:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2198:** This line contributes implementation detail or declarative structure to the file.
  **CN L2198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2199:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2199:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2200:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2200:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2201:** Blank line used to separate nearby declarations and improve readability.
  **CN L2201:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2202:** This TableGen `def` record introduces `ROCDL_RawBufferStoreOp`, which later participates in generated MLIR code.
  **CN L2202:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferStoreOp`，后续会参与生成的 MLIR 代码。
- **EN L2203:** This line contributes implementation detail or declarative structure to the file.
  **CN L2203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2204:** This line contributes implementation detail or declarative structure to the file.
  **CN L2204:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2205:** This line contributes implementation detail or declarative structure to the file.
  **CN L2205:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2206:** This line contributes implementation detail or declarative structure to the file.
  **CN L2206:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2207:** This line contributes implementation detail or declarative structure to the file.
  **CN L2207:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2208:** This line contributes implementation detail or declarative structure to the file.
  **CN L2208:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2209-2220 / 第 2209-2220 行

```tablegen
2209:   let hasCustomAssemblyFormat = 1;
2210: }
2211: 
2212: def ROCDL_RawBufferAtomicCmpSwap :
2213:   ROCDL_IntrOp<"raw.buffer.atomic.cmpswap", [], [0], [AllTypesMatch<["res", "src", "cmp"]>], 1>,
2214:   Arguments<(ins LLVM_Type:$src,
2215:                  LLVM_Type:$cmp,
2216:                  LLVM_Type:$rsrc,
2217:                  I32:$offset,
2218:                  I32:$soffset,
2219:                  I32:$aux)>{
2220:   let assemblyFormat = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2209:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2209:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2210:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2210:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2211:** Blank line used to separate nearby declarations and improve readability.
  **CN L2211:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2212:** This TableGen `def` record introduces `ROCDL_RawBufferAtomicCmpSwap`, which later participates in generated MLIR code.
  **CN L2212:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferAtomicCmpSwap`，后续会参与生成的 MLIR 代码。
- **EN L2213:** This line contributes implementation detail or declarative structure to the file.
  **CN L2213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2214:** This line contributes implementation detail or declarative structure to the file.
  **CN L2214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2215:** This line contributes implementation detail or declarative structure to the file.
  **CN L2215:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2216:** This line contributes implementation detail or declarative structure to the file.
  **CN L2216:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2217:** This line contributes implementation detail or declarative structure to the file.
  **CN L2217:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2218:** This line contributes implementation detail or declarative structure to the file.
  **CN L2218:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2219:** This line contributes implementation detail or declarative structure to the file.
  **CN L2219:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2220:** This line contributes implementation detail or declarative structure to the file.
  **CN L2220:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2221-2232 / 第 2221-2232 行

```tablegen
2221:     attr-dict `(` operands `)` `:` type($res) `,` type($rsrc)
2222:   }];
2223: }
2224: 
2225: //===---------------------------------------------------------------------===//
2226: // Memory prefetch intrinsics
2227: 
2228: def ROCDL_GlobalPrefetchOp :
2229:   ROCDL_IntrOp<"global.prefetch", [], [], [], 0, 0, 1, 0, [1], ["scope"]> {
2230:   dag args = (ins Arg<LLVM_PointerInAddressSpace<1>, "", [MemWrite, MemRead]>:$ptr,
2231:                   I32Attr:$scope);
2232:   let arguments = !con(args, baseArgs);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2221:** This line contributes to the declaration or call of `type`.
  **CN L2221:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2222:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2222:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2223:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2223:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2224:** Blank line used to separate nearby declarations and improve readability.
  **CN L2224:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2225:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2225:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2226:** This comment states: “Memory prefetch intrinsics”, documenting the intent of the surrounding code.
  **CN L2226:** 该注释写道：“Memory prefetch intrinsics”，用于说明周围代码的意图。
- **EN L2227:** Blank line used to separate nearby declarations and improve readability.
  **CN L2227:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2228:** This TableGen `def` record introduces `ROCDL_GlobalPrefetchOp`, which later participates in generated MLIR code.
  **CN L2228:** 该 TableGen `def` 记录引入了 `ROCDL_GlobalPrefetchOp`，后续会参与生成的 MLIR 代码。
- **EN L2229:** This line contributes implementation detail or declarative structure to the file.
  **CN L2229:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2230:** This line contributes implementation detail or declarative structure to the file.
  **CN L2230:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2231:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2231:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2232:** This line contributes to the declaration or call of `con`.
  **CN L2232:** 这一行为 `con` 的声明或调用提供内容。

### Lines 2233-2244 / 第 2233-2244 行

```tablegen
2233:   let description = [{
2234:     Prefetches 1 byte of data per lane from global memory into the WGP-cache or L2-cache.
2235:     Available on gfx1250+.
2236: 
2237:     Example:
2238:     ```mlir
2239:     // Prefetch from global memory into cache.
2240:     rocdl.global.prefetch %ptr, scope 0 : !llvm.ptr<1>
2241:     ```
2242:   }];
2243:   let results = (outs);
2244:   let assemblyFormat = "$ptr `,` `scope` $scope attr-dict `:` qualified(type($ptr))";
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2233:** This line contributes implementation detail or declarative structure to the file.
  **CN L2233:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2234:** This line contributes implementation detail or declarative structure to the file.
  **CN L2234:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2235:** This line contributes implementation detail or declarative structure to the file.
  **CN L2235:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2236:** Blank line used to separate nearby declarations and improve readability.
  **CN L2236:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2237:** This line contributes implementation detail or declarative structure to the file.
  **CN L2237:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2238:** This line contributes implementation detail or declarative structure to the file.
  **CN L2238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2239:** This comment states: “Prefetch from global memory into cache.”, documenting the intent of the surrounding code.
  **CN L2239:** 该注释写道：“Prefetch from global memory into cache.”，用于说明周围代码的意图。
- **EN L2240:** This line contributes implementation detail or declarative structure to the file.
  **CN L2240:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2241:** This line contributes implementation detail or declarative structure to the file.
  **CN L2241:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2242:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2242:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2243:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2243:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2244:** This line contributes to the declaration or call of `qualified`.
  **CN L2244:** 这一行为 `qualified` 的声明或调用提供内容。

### Lines 2245-2256 / 第 2245-2256 行

```tablegen
2245:   let extraClassDefinition = [{
2246:     SmallVector<Value> $cppClass::getAccessedOperands() {
2247:       return {getPtr()};
2248:     }
2249:   }];
2250: }
2251: 
2252: def ROCDL_FlatPrefetchOp :
2253:   ROCDL_IntrOp<"flat.prefetch", [], [], [], 0, 0, 1, 0, [1], ["scope"]> {
2254:   dag args = (ins Arg<LLVM_PointerInAddressSpace<0>, "", [MemWrite, MemRead]>:$ptr,
2255:                   I32Attr:$scope);
2256:   let arguments = !con(args, baseArgs);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2245:** This line contributes implementation detail or declarative structure to the file.
  **CN L2245:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2246:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2246:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2247:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2247:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2248:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2248:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2249:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2249:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2250:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2250:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2251:** Blank line used to separate nearby declarations and improve readability.
  **CN L2251:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2252:** This TableGen `def` record introduces `ROCDL_FlatPrefetchOp`, which later participates in generated MLIR code.
  **CN L2252:** 该 TableGen `def` 记录引入了 `ROCDL_FlatPrefetchOp`，后续会参与生成的 MLIR 代码。
- **EN L2253:** This line contributes implementation detail or declarative structure to the file.
  **CN L2253:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2254:** This line contributes implementation detail or declarative structure to the file.
  **CN L2254:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2255:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2255:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2256:** This line contributes to the declaration or call of `con`.
  **CN L2256:** 这一行为 `con` 的声明或调用提供内容。

### Lines 2257-2268 / 第 2257-2268 行

```tablegen
2257:   let description = [{
2258:     Prefetches 1 byte of data per lane using flat-memory addresses into the WGP-cache or L2-cache.
2259:     Available on gfx1250+.
2260: 
2261:     Example:
2262:     ```mlir
2263:     // Prefetch from flat memory into cache.
2264:     rocdl.flat.prefetch %ptr, scope 0 : !llvm.ptr
2265:     ```
2266:   }];
2267:   let results = (outs);
2268:   let assemblyFormat = "$ptr `,` `scope` $scope attr-dict `:` qualified(type($ptr))";
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2257:** This line contributes implementation detail or declarative structure to the file.
  **CN L2257:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2258:** This line contributes implementation detail or declarative structure to the file.
  **CN L2258:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2259:** This line contributes implementation detail or declarative structure to the file.
  **CN L2259:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2260:** Blank line used to separate nearby declarations and improve readability.
  **CN L2260:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2261:** This line contributes implementation detail or declarative structure to the file.
  **CN L2261:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2262:** This line contributes implementation detail or declarative structure to the file.
  **CN L2262:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2263:** This comment states: “Prefetch from flat memory into cache.”, documenting the intent of the surrounding code.
  **CN L2263:** 该注释写道：“Prefetch from flat memory into cache.”，用于说明周围代码的意图。
- **EN L2264:** This line contributes implementation detail or declarative structure to the file.
  **CN L2264:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2265:** This line contributes implementation detail or declarative structure to the file.
  **CN L2265:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2266:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2266:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2267:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2267:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2268:** This line contributes to the declaration or call of `qualified`.
  **CN L2268:** 这一行为 `qualified` 的声明或调用提供内容。

### Lines 2269-2280 / 第 2269-2280 行

```tablegen
2269:   let extraClassDefinition = [{
2270:     SmallVector<Value> $cppClass::getAccessedOperands() {
2271:       return {getPtr()};
2272:     }
2273:   }];
2274: }
2275: 
2276: //===---------------------------------------------------------------------===//
2277: // Atomic barrier intrinsic (LDS memory barriers).
2278: 
2279: def ROCDL_DsAtomicBarrierArriveRtnOp :
2280:   ROCDL_IntrOp<"ds.atomic.barrier.arrive.rtn.b64", [], [], [], 1, 0, 1, 0, [], []> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2269:** This line contributes implementation detail or declarative structure to the file.
  **CN L2269:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2270:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2270:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2271:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2271:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2272:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2272:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2273:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2273:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2274:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2274:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2275:** Blank line used to separate nearby declarations and improve readability.
  **CN L2275:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2276:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2276:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2277:** This comment states: “Atomic barrier intrinsic (LDS memory barriers).”, documenting the intent of the surrounding code.
  **CN L2277:** 该注释写道：“Atomic barrier intrinsic (LDS memory barriers).”，用于说明周围代码的意图。
- **EN L2278:** Blank line used to separate nearby declarations and improve readability.
  **CN L2278:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2279:** This TableGen `def` record introduces `ROCDL_DsAtomicBarrierArriveRtnOp`, which later participates in generated MLIR code.
  **CN L2279:** 该 TableGen `def` 记录引入了 `ROCDL_DsAtomicBarrierArriveRtnOp`，后续会参与生成的 MLIR 代码。
- **EN L2280:** This line contributes implementation detail or declarative structure to the file.
  **CN L2280:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2281-2292 / 第 2281-2292 行

```tablegen
2281:   dag args = (ins Arg<ROCDLBufferLDS, "", [MemRead, MemWrite]>:$barrierPtr,
2282:                   I64:$val);
2283:   let arguments = !con(args, baseArgs);
2284:   let description = [{
2285:     Waits on a given DS barrier and decrements its pending count by a given value. Note, the barrier state
2286:     is given as a 64-bit structure containing pending count, phase and init count. The op returns the old
2287:     barrier state. The op is executed as an ordinary LDS operations and it is ordered with other LDS operations.
2288:     Thus, check DSCNT to determine when this instruction has executed.
2289:     Available on gfx1250+.
2290: 
2291:     Example:
2292:     ```mlir
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2281:** This line contributes implementation detail or declarative structure to the file.
  **CN L2281:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2282:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2282:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2283:** This line contributes to the declaration or call of `con`.
  **CN L2283:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2284:** This line contributes implementation detail or declarative structure to the file.
  **CN L2284:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2285:** This line contributes implementation detail or declarative structure to the file.
  **CN L2285:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2286:** This line contributes implementation detail or declarative structure to the file.
  **CN L2286:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2287:** This line contributes implementation detail or declarative structure to the file.
  **CN L2287:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2288:** This line contributes implementation detail or declarative structure to the file.
  **CN L2288:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2289:** This line contributes implementation detail or declarative structure to the file.
  **CN L2289:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2290:** Blank line used to separate nearby declarations and improve readability.
  **CN L2290:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2291:** This line contributes implementation detail or declarative structure to the file.
  **CN L2291:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2292:** This line contributes implementation detail or declarative structure to the file.
  **CN L2292:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2293-2304 / 第 2293-2304 行

```tablegen
2293:     // Atomic barrier arrive with return of old barrier state.
2294:     %res = rocdl.ds.atomic.barrier.arrive.rtn.b64 %ptr, %val : !llvm.ptr<3>, i64 -> i64
2295:     ```
2296:   }];
2297:   let results = (outs I64:$res);
2298:   let assemblyFormat = "$barrierPtr `,` $val attr-dict `:` qualified(type($barrierPtr)) `,` type($val) `->` type($res)";
2299:   let extraClassDefinition = [{
2300:     SmallVector<Value> $cppClass::getAccessedOperands() {
2301:       return {getBarrierPtr()};
2302:     }
2303:   }];
2304: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2293:** This comment states: “Atomic barrier arrive with return of old barrier state.”, documenting the intent of the surrounding code.
  **CN L2293:** 该注释写道：“Atomic barrier arrive with return of old barrier state.”，用于说明周围代码的意图。
- **EN L2294:** This line contributes implementation detail or declarative structure to the file.
  **CN L2294:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2295:** This line contributes implementation detail or declarative structure to the file.
  **CN L2295:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2296:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2296:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2297:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2297:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2298:** This line contributes to the declaration or call of `qualified`.
  **CN L2298:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L2299:** This line contributes implementation detail or declarative structure to the file.
  **CN L2299:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2300:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2300:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2301:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2301:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2302:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2302:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2303:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2303:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2304:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2304:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2305-2316 / 第 2305-2316 行

```tablegen
2305: 
2306: def ROCDL_DsAtomicAsyncBarrierArriveOp :
2307:   ROCDL_IntrOp<"ds.atomic.async.barrier.arrive.b64", [], [], [], 0, 0, 1, 0, [], []> {
2308:   dag args = (ins Arg<ROCDLBufferLDS, "", [MemWrite]>:$barrierPtr);
2309:   let arguments = !con(args, baseArgs);
2310:   let description = [{
2311:     Waits on a given DS barrier and decrements pending count by -1.
2312:     Stays in order with ASYNC loads to LDS, and uses ASYNCcnt to track its completion.
2313:     Available on gfx1250+.
2314: 
2315:     Example:
2316:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2305:** Blank line used to separate nearby declarations and improve readability.
  **CN L2305:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2306:** This TableGen `def` record introduces `ROCDL_DsAtomicAsyncBarrierArriveOp`, which later participates in generated MLIR code.
  **CN L2306:** 该 TableGen `def` 记录引入了 `ROCDL_DsAtomicAsyncBarrierArriveOp`，后续会参与生成的 MLIR 代码。
- **EN L2307:** This line contributes implementation detail or declarative structure to the file.
  **CN L2307:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2308:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2308:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2309:** This line contributes to the declaration or call of `con`.
  **CN L2309:** 这一行为 `con` 的声明或调用提供内容。
- **EN L2310:** This line contributes implementation detail or declarative structure to the file.
  **CN L2310:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2311:** This line contributes implementation detail or declarative structure to the file.
  **CN L2311:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2312:** This line contributes implementation detail or declarative structure to the file.
  **CN L2312:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2313:** This line contributes implementation detail or declarative structure to the file.
  **CN L2313:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2314:** Blank line used to separate nearby declarations and improve readability.
  **CN L2314:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2315:** This line contributes implementation detail or declarative structure to the file.
  **CN L2315:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2316:** This line contributes implementation detail or declarative structure to the file.
  **CN L2316:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2317-2328 / 第 2317-2328 行

```tablegen
2317:     // Async atomic barrier arrive (fire-and-forget).
2318:     rocdl.ds.atomic.async.barrier.arrive.b64 %ptr : !llvm.ptr<3>
2319:     ```
2320:   }];
2321:   let results = (outs);
2322:   let assemblyFormat = "$barrierPtr attr-dict `:` qualified(type($barrierPtr))";
2323:   let extraClassDefinition = [{
2324:     SmallVector<Value> $cppClass::getAccessedOperands() {
2325:       return {getBarrierPtr()};
2326:     }
2327:   }];
2328: }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2317:** This comment states: “Async atomic barrier arrive (fire-and-forget).”, documenting the intent of the surrounding code.
  **CN L2317:** 该注释写道：“Async atomic barrier arrive (fire-and-forget).”，用于说明周围代码的意图。
- **EN L2318:** This line contributes implementation detail or declarative structure to the file.
  **CN L2318:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2319:** This line contributes implementation detail or declarative structure to the file.
  **CN L2319:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2320:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2320:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2321:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2321:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2322:** This line contributes to the declaration or call of `qualified`.
  **CN L2322:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L2323:** This line contributes implementation detail or declarative structure to the file.
  **CN L2323:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2324:** This line contributes to the declaration or call of `getAccessedOperands`.
  **CN L2324:** 这一行为 `getAccessedOperands` 的声明或调用提供内容。
- **EN L2325:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2325:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2326:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2326:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2327:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2327:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2328:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2328:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2329-2340 / 第 2329-2340 行

```tablegen
2329: 
2330: //===---------------------------------------------------------------------===//
2331: // MI-100 and MI-200 buffer atomic floating point add intrinsic
2332: 
2333: def ROCDL_RawBufferAtomicFAddOp :
2334:   ROCDL_IntrOp<"raw.buffer.atomic.fadd", [], [0], [], 0>,
2335:   Arguments<(ins LLVM_Type:$vdata,
2336:                  LLVM_Type:$rsrc,
2337:                  LLVM_Type:$offset,
2338:                  LLVM_Type:$soffset,
2339:                  LLVM_Type:$aux)>{
2340:   let hasCustomAssemblyFormat = 1;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2329:** Blank line used to separate nearby declarations and improve readability.
  **CN L2329:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2330:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2330:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2331:** This comment states: “MI-100 and MI-200 buffer atomic floating point add intrinsic”, documenting the intent of the surrounding code.
  **CN L2331:** 该注释写道：“MI-100 and MI-200 buffer atomic floating point add intrinsic”，用于说明周围代码的意图。
- **EN L2332:** Blank line used to separate nearby declarations and improve readability.
  **CN L2332:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2333:** This TableGen `def` record introduces `ROCDL_RawBufferAtomicFAddOp`, which later participates in generated MLIR code.
  **CN L2333:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferAtomicFAddOp`，后续会参与生成的 MLIR 代码。
- **EN L2334:** This line contributes implementation detail or declarative structure to the file.
  **CN L2334:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2335:** This line contributes implementation detail or declarative structure to the file.
  **CN L2335:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2336:** This line contributes implementation detail or declarative structure to the file.
  **CN L2336:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2337:** This line contributes implementation detail or declarative structure to the file.
  **CN L2337:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2338:** This line contributes implementation detail or declarative structure to the file.
  **CN L2338:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2339:** This line contributes implementation detail or declarative structure to the file.
  **CN L2339:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2340:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2340:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2341-2352 / 第 2341-2352 行

```tablegen
2341: }
2342: 
2343: //===---------------------------------------------------------------------===//
2344: // Buffer atomic floating point max intrinsic. GFX9 does not support fp32.
2345: 
2346: def ROCDL_RawBufferAtomicFMaxOp :
2347:   ROCDL_IntrOp<"raw.buffer.atomic.fmax", [], [0], [], 0>,
2348:   Arguments<(ins LLVM_Type:$vdata,
2349:                  LLVM_Type:$rsrc,
2350:                  LLVM_Type:$offset,
2351:                  LLVM_Type:$soffset,
2352:                  LLVM_Type:$aux)>{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2341:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2341:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2342:** Blank line used to separate nearby declarations and improve readability.
  **CN L2342:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2343:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2343:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2344:** This comment states: “Buffer atomic floating point max intrinsic. GFX9 does not support fp32.”, documenting the intent of the surrounding code.
  **CN L2344:** 该注释写道：“Buffer atomic floating point max intrinsic. GFX9 does not support fp32.”，用于说明周围代码的意图。
- **EN L2345:** Blank line used to separate nearby declarations and improve readability.
  **CN L2345:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2346:** This TableGen `def` record introduces `ROCDL_RawBufferAtomicFMaxOp`, which later participates in generated MLIR code.
  **CN L2346:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferAtomicFMaxOp`，后续会参与生成的 MLIR 代码。
- **EN L2347:** This line contributes implementation detail or declarative structure to the file.
  **CN L2347:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2348:** This line contributes implementation detail or declarative structure to the file.
  **CN L2348:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2349:** This line contributes implementation detail or declarative structure to the file.
  **CN L2349:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2350:** This line contributes implementation detail or declarative structure to the file.
  **CN L2350:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2351:** This line contributes implementation detail or declarative structure to the file.
  **CN L2351:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2352:** This line contributes implementation detail or declarative structure to the file.
  **CN L2352:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2353-2364 / 第 2353-2364 行

```tablegen
2353:   let hasCustomAssemblyFormat = 1;
2354: }
2355: 
2356: //===---------------------------------------------------------------------===//
2357: // Buffer atomic signed integer max intrinsic.
2358: 
2359: def ROCDL_RawBufferAtomicSMaxOp :
2360:   ROCDL_IntrOp<"raw.buffer.atomic.smax", [], [0], [], 0>,
2361:   Arguments<(ins LLVM_Type:$vdata,
2362:                  LLVM_Type:$rsrc,
2363:                  LLVM_Type:$offset,
2364:                  LLVM_Type:$soffset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2353:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2353:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2354:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2354:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2355:** Blank line used to separate nearby declarations and improve readability.
  **CN L2355:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2356:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2356:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2357:** This comment states: “Buffer atomic signed integer max intrinsic.”, documenting the intent of the surrounding code.
  **CN L2357:** 该注释写道：“Buffer atomic signed integer max intrinsic.”，用于说明周围代码的意图。
- **EN L2358:** Blank line used to separate nearby declarations and improve readability.
  **CN L2358:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2359:** This TableGen `def` record introduces `ROCDL_RawBufferAtomicSMaxOp`, which later participates in generated MLIR code.
  **CN L2359:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferAtomicSMaxOp`，后续会参与生成的 MLIR 代码。
- **EN L2360:** This line contributes implementation detail or declarative structure to the file.
  **CN L2360:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2361:** This line contributes implementation detail or declarative structure to the file.
  **CN L2361:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2362:** This line contributes implementation detail or declarative structure to the file.
  **CN L2362:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2363:** This line contributes implementation detail or declarative structure to the file.
  **CN L2363:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2364:** This line contributes implementation detail or declarative structure to the file.
  **CN L2364:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2365-2376 / 第 2365-2376 行

```tablegen
2365:                  LLVM_Type:$aux)>{
2366:   let hasCustomAssemblyFormat = 1;
2367: }
2368: 
2369: //===---------------------------------------------------------------------===//
2370: // Buffer atomic unsigned integer min intrinsic.
2371: 
2372: def ROCDL_RawBufferAtomicUMinOp :
2373:   ROCDL_IntrOp<"raw.buffer.atomic.umin", [], [0], [], 0>,
2374:   Arguments<(ins LLVM_Type:$vdata,
2375:                  LLVM_Type:$rsrc,
2376:                  LLVM_Type:$offset,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2365:** This line contributes implementation detail or declarative structure to the file.
  **CN L2365:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2366:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2366:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2367:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2367:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2368:** Blank line used to separate nearby declarations and improve readability.
  **CN L2368:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2369:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2369:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2370:** This comment states: “Buffer atomic unsigned integer min intrinsic.”, documenting the intent of the surrounding code.
  **CN L2370:** 该注释写道：“Buffer atomic unsigned integer min intrinsic.”，用于说明周围代码的意图。
- **EN L2371:** Blank line used to separate nearby declarations and improve readability.
  **CN L2371:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2372:** This TableGen `def` record introduces `ROCDL_RawBufferAtomicUMinOp`, which later participates in generated MLIR code.
  **CN L2372:** 该 TableGen `def` 记录引入了 `ROCDL_RawBufferAtomicUMinOp`，后续会参与生成的 MLIR 代码。
- **EN L2373:** This line contributes implementation detail or declarative structure to the file.
  **CN L2373:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2374:** This line contributes implementation detail or declarative structure to the file.
  **CN L2374:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2375:** This line contributes implementation detail or declarative structure to the file.
  **CN L2375:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2376:** This line contributes implementation detail or declarative structure to the file.
  **CN L2376:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2377-2388 / 第 2377-2388 行

```tablegen
2377:                  LLVM_Type:$soffset,
2378:                  LLVM_Type:$aux)>{
2379:   let hasCustomAssemblyFormat = 1;
2380: }
2381: 
2382: // DPP Update intrinsic
2383: def ROCDL_DPPUpdateOp : ROCDL_IntrOp<"update.dpp", [], [0],
2384:     [AllTypesMatch<["res", "src", "old"]>], 1, 0, 0, 0,
2385:       [2, 3, 4, 5], ["dppCtrl", "rowMask", "bankMask", "boundCtrl"]>,
2386:   Arguments<(ins LLVM_Type:$old, LLVM_Type:$src, I32Attr:$dppCtrl, I32Attr:$rowMask,
2387:       I32Attr:$bankMask, I1Attr:$boundCtrl)> {
2388:   let results = (outs LLVM_Type:$res);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2377:** This line contributes implementation detail or declarative structure to the file.
  **CN L2377:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2378:** This line contributes implementation detail or declarative structure to the file.
  **CN L2378:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2379:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2379:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2380:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2380:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2381:** Blank line used to separate nearby declarations and improve readability.
  **CN L2381:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2382:** This comment states: “DPP Update intrinsic”, documenting the intent of the surrounding code.
  **CN L2382:** 该注释写道：“DPP Update intrinsic”，用于说明周围代码的意图。
- **EN L2383:** This TableGen `def` record introduces `ROCDL_DPPUpdateOp`, which later participates in generated MLIR code.
  **CN L2383:** 该 TableGen `def` 记录引入了 `ROCDL_DPPUpdateOp`，后续会参与生成的 MLIR 代码。
- **EN L2384:** This line contributes implementation detail or declarative structure to the file.
  **CN L2384:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2385:** This line contributes implementation detail or declarative structure to the file.
  **CN L2385:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2386:** This line contributes implementation detail or declarative structure to the file.
  **CN L2386:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2387:** This line contributes implementation detail or declarative structure to the file.
  **CN L2387:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2388:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2388:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2389-2400 / 第 2389-2400 行

```tablegen
2389:   let assemblyFormat = [{
2390:     attr-dict $old `,` $src `with` $dppCtrl `,` $rowMask `,` $bankMask `,` $boundCtrl `:` type($src)
2391:   }];
2392: }
2393: 
2394: // PermLaneX16 intrinsic operation
2395: def ROCDL_PermlaneX16Op : ROCDL_IntrOp<"permlanex16", [], [0],
2396:     [AllTypesMatch<["res", "old", "src0"]>, AllTypesMatch<["src1", "src2"]>], 1, 0, 0, 0,
2397:     [4, 5], ["fi", "boundControl"]>,
2398:   Arguments<(ins LLVM_Type:$old, LLVM_Type:$src0, LLVM_Type:$src1, LLVM_Type:$src2,
2399:              I1Attr:$fi, I1Attr:$boundControl)> {
2400:   let results = (outs LLVM_Type:$res);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2389:** This line contributes implementation detail or declarative structure to the file.
  **CN L2389:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2390:** This line contributes to the declaration or call of `type`.
  **CN L2390:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2391:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2391:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2392:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2392:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2393:** Blank line used to separate nearby declarations and improve readability.
  **CN L2393:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2394:** This comment states: “PermLaneX16 intrinsic operation”, documenting the intent of the surrounding code.
  **CN L2394:** 该注释写道：“PermLaneX16 intrinsic operation”，用于说明周围代码的意图。
- **EN L2395:** This TableGen `def` record introduces `ROCDL_PermlaneX16Op`, which later participates in generated MLIR code.
  **CN L2395:** 该 TableGen `def` 记录引入了 `ROCDL_PermlaneX16Op`，后续会参与生成的 MLIR 代码。
- **EN L2396:** This line contributes implementation detail or declarative structure to the file.
  **CN L2396:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2397:** This line contributes implementation detail or declarative structure to the file.
  **CN L2397:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2398:** This line contributes implementation detail or declarative structure to the file.
  **CN L2398:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2399:** This line contributes implementation detail or declarative structure to the file.
  **CN L2399:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2400:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2400:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2401-2412 / 第 2401-2412 行

```tablegen
2401:   let assemblyFormat = [{
2402:     attr-dict $old `,` $src0 `,` $src1 `,` $src2 `,` $fi `,` $boundControl `:` type($src0) `,` type($src1)
2403:   }];
2404:   let description = [{
2405:     Performs a `permlanex16` operation with the given operands, applying the
2406:     permutation specified by $fi to the provided inputs.
2407: 
2408:     Example:
2409:     ```mlir
2410:     // Scalar permlanex16.
2411:     %ret0 = rocdl.permlanex16 %src0, %src0, %sel, %sel, 0, -1 : f32, i32
2412: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2401:** This line contributes implementation detail or declarative structure to the file.
  **CN L2401:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2402:** This line contributes to the declaration or call of `type`.
  **CN L2402:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2403:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2403:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2404:** This line contributes implementation detail or declarative structure to the file.
  **CN L2404:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2405:** This line contributes implementation detail or declarative structure to the file.
  **CN L2405:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2406:** This line contributes implementation detail or declarative structure to the file.
  **CN L2406:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2407:** Blank line used to separate nearby declarations and improve readability.
  **CN L2407:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2408:** This line contributes implementation detail or declarative structure to the file.
  **CN L2408:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2409:** This line contributes implementation detail or declarative structure to the file.
  **CN L2409:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2410:** This comment states: “Scalar permlanex16.”, documenting the intent of the surrounding code.
  **CN L2410:** 该注释写道：“Scalar permlanex16.”，用于说明周围代码的意图。
- **EN L2411:** This line contributes implementation detail or declarative structure to the file.
  **CN L2411:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2412:** Blank line used to separate nearby declarations and improve readability.
  **CN L2412:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2413-2424 / 第 2413-2424 行

```tablegen
2413:     // Vector permlanex16.
2414:     %ret1 = rocdl.permlanex16 %src1, %src1, %sel, %sel, 0, -1 : vector<2xf32>, i32
2415:     ```
2416:   }];
2417: }
2418: 
2419: class ROCDL_ConcretePair<Type elem0, Type elem1> :
2420:   Type<And<[
2421:     LLVM_AnyStruct.predicate,
2422:     SubstLeaves<
2423:          "$_self",
2424:          "::llvm::cast<::mlir::LLVM::LLVMStructType>($_self).getBody()[0]",
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2413:** This comment states: “Vector permlanex16.”, documenting the intent of the surrounding code.
  **CN L2413:** 该注释写道：“Vector permlanex16.”，用于说明周围代码的意图。
- **EN L2414:** This line contributes implementation detail or declarative structure to the file.
  **CN L2414:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2415:** This line contributes implementation detail or declarative structure to the file.
  **CN L2415:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2416:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2416:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2417:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2417:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2418:** Blank line used to separate nearby declarations and improve readability.
  **CN L2418:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2419:** This TableGen `class` record introduces `ROCDL_ConcretePair`, which later participates in generated MLIR code.
  **CN L2419:** 该 TableGen `class` 记录引入了 `ROCDL_ConcretePair`，后续会参与生成的 MLIR 代码。
- **EN L2420:** This line contributes implementation detail or declarative structure to the file.
  **CN L2420:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2421:** This line contributes implementation detail or declarative structure to the file.
  **CN L2421:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2422:** This line contributes implementation detail or declarative structure to the file.
  **CN L2422:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2423:** This line contributes implementation detail or declarative structure to the file.
  **CN L2423:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2424:** This line contributes to the declaration or call of `getBody`.
  **CN L2424:** 这一行为 `getBody` 的声明或调用提供内容。

### Lines 2425-2436 / 第 2425-2436 行

```tablegen
2425:          elem0.predicate>,
2426:       SubstLeaves<
2427:          "$_self",
2428:          "::llvm::cast<::mlir::LLVM::LLVMStructType>($_self).getBody()[1]",
2429:          elem1.predicate>
2430:   ]>,
2431:   "LLVM dialect-compatible struct of " # elem0.summary # "and" # elem1.summary,
2432:   "::mlir::LLVM::LLVMStructType">,
2433:   BuildableType<"::mlir::LLVM::LLVMStructType::getLiteral($_builder.getContext(), "
2434:   "{" # elem0.builderCall # ", " # elem1.builderCall # "})">;
2435: 
2436: // Permlane16 swap intrinsic operation
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2425:** This line contributes implementation detail or declarative structure to the file.
  **CN L2425:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2426:** This line contributes implementation detail or declarative structure to the file.
  **CN L2426:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2427:** This line contributes implementation detail or declarative structure to the file.
  **CN L2427:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2428:** This line contributes to the declaration or call of `getBody`.
  **CN L2428:** 这一行为 `getBody` 的声明或调用提供内容。
- **EN L2429:** This line contributes implementation detail or declarative structure to the file.
  **CN L2429:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2430:** This line contributes implementation detail or declarative structure to the file.
  **CN L2430:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2431:** This line contributes implementation detail or declarative structure to the file.
  **CN L2431:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2432:** This line contributes implementation detail or declarative structure to the file.
  **CN L2432:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2433:** This line contributes to the declaration or call of `getLiteral`.
  **CN L2433:** 这一行为 `getLiteral` 的声明或调用提供内容。
- **EN L2434:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2434:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2435:** Blank line used to separate nearby declarations and improve readability.
  **CN L2435:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2436:** This comment states: “Permlane16 swap intrinsic operation”, documenting the intent of the surrounding code.
  **CN L2436:** 该注释写道：“Permlane16 swap intrinsic operation”，用于说明周围代码的意图。

### Lines 2437-2448 / 第 2437-2448 行

```tablegen
2437: def ROCDL_Permlane16SwapOp : ROCDL_IntrOp<"permlane16.swap", [], [],
2438:     [], 1, 0, 0, 0,
2439:     [2, 3], ["fi", "boundControl"]>,
2440:   Arguments<(ins I32:$old, I32:$src, I1Attr:$fi, I1Attr:$boundControl)> {
2441:   let results = (outs ROCDL_ConcretePair<I32, I32>:$res);
2442:   let assemblyFormat = [{
2443:     attr-dict $old `,` $src `,` $fi `,` $boundControl `:` `(` type($old) `,` type($src) `)` `->` type($res)
2444:   }];
2445:   let description = [{
2446:     Performs a `permlane16.swap` operation with the given operands, applying the
2447:     permutation specified by $fi to the provided inputs.
2448: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2437:** This TableGen `def` record introduces `ROCDL_Permlane16SwapOp`, which later participates in generated MLIR code.
  **CN L2437:** 该 TableGen `def` 记录引入了 `ROCDL_Permlane16SwapOp`，后续会参与生成的 MLIR 代码。
- **EN L2438:** This line contributes implementation detail or declarative structure to the file.
  **CN L2438:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2439:** This line contributes implementation detail or declarative structure to the file.
  **CN L2439:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2440:** This line contributes implementation detail or declarative structure to the file.
  **CN L2440:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2441:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2441:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2442:** This line contributes implementation detail or declarative structure to the file.
  **CN L2442:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2443:** This line contributes to the declaration or call of `type`.
  **CN L2443:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2444:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2444:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2445:** This line contributes implementation detail or declarative structure to the file.
  **CN L2445:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2446:** This line contributes implementation detail or declarative structure to the file.
  **CN L2446:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2447:** This line contributes implementation detail or declarative structure to the file.
  **CN L2447:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2448:** Blank line used to separate nearby declarations and improve readability.
  **CN L2448:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2449-2460 / 第 2449-2460 行

```tablegen
2449:     Example:
2450:     ```mlir
2451:     // Swap lanes between groups of 16 threads.
2452:     %res = rocdl.permlane16.swap %src, %src, 0, -1 : (i32, i32) -> !llvm.struct<(i32, i32)>
2453:     ```
2454:   }];
2455: }
2456: 
2457: // Permlane32 swap intrinsic operation
2458: def ROCDL_Permlane32SwapOp : ROCDL_IntrOp<"permlane32.swap", [], [],
2459:     [], 1, 0, 0, 0,
2460:     [2, 3], ["fi", "boundControl"]>,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2449:** This line contributes implementation detail or declarative structure to the file.
  **CN L2449:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2450:** This line contributes implementation detail or declarative structure to the file.
  **CN L2450:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2451:** This comment states: “Swap lanes between groups of 16 threads.”, documenting the intent of the surrounding code.
  **CN L2451:** 该注释写道：“Swap lanes between groups of 16 threads.”，用于说明周围代码的意图。
- **EN L2452:** This line contributes implementation detail or declarative structure to the file.
  **CN L2452:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2453:** This line contributes implementation detail or declarative structure to the file.
  **CN L2453:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2454:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2454:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2455:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2455:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2456:** Blank line used to separate nearby declarations and improve readability.
  **CN L2456:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2457:** This comment states: “Permlane32 swap intrinsic operation”, documenting the intent of the surrounding code.
  **CN L2457:** 该注释写道：“Permlane32 swap intrinsic operation”，用于说明周围代码的意图。
- **EN L2458:** This TableGen `def` record introduces `ROCDL_Permlane32SwapOp`, which later participates in generated MLIR code.
  **CN L2458:** 该 TableGen `def` 记录引入了 `ROCDL_Permlane32SwapOp`，后续会参与生成的 MLIR 代码。
- **EN L2459:** This line contributes implementation detail or declarative structure to the file.
  **CN L2459:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2460:** This line contributes implementation detail or declarative structure to the file.
  **CN L2460:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2461-2472 / 第 2461-2472 行

```tablegen
2461:   Arguments<(ins I32:$old, I32:$src, I1Attr:$fi, I1Attr:$boundControl)> {
2462:   let results = (outs ROCDL_ConcretePair<I32, I32>:$res);
2463:   let assemblyFormat = [{
2464:     attr-dict $old `,` $src `,` $fi `,` $boundControl `:` `(` type($old) `,` type($src) `)` `->` type($res)
2465:   }];
2466:   let description = [{
2467:     Performs a `permlane32.swap` operation with the given operands, applying the
2468:     permutation specified by $fi to the provided inputs.
2469: 
2470:     Example:
2471:     ```mlir
2472:     // Swap lanes between groups of 32 threads.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2461:** This line contributes implementation detail or declarative structure to the file.
  **CN L2461:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2462:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2462:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2463:** This line contributes implementation detail or declarative structure to the file.
  **CN L2463:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2464:** This line contributes to the declaration or call of `type`.
  **CN L2464:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2465:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2465:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2466:** This line contributes implementation detail or declarative structure to the file.
  **CN L2466:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2467:** This line contributes implementation detail or declarative structure to the file.
  **CN L2467:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2468:** This line contributes implementation detail or declarative structure to the file.
  **CN L2468:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2469:** Blank line used to separate nearby declarations and improve readability.
  **CN L2469:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2470:** This line contributes implementation detail or declarative structure to the file.
  **CN L2470:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2471:** This line contributes implementation detail or declarative structure to the file.
  **CN L2471:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2472:** This comment states: “Swap lanes between groups of 32 threads.”, documenting the intent of the surrounding code.
  **CN L2472:** 该注释写道：“Swap lanes between groups of 32 threads.”，用于说明周围代码的意图。

### Lines 2473-2484 / 第 2473-2484 行

```tablegen
2473:     %res = rocdl.permlane32.swap %src, %src, 0, -1 : (i32, i32) -> !llvm.struct<(i32, i32)>
2474:     ```
2475:   }];
2476: }
2477: 
2478: //===---------------------------------------------------------------------===//
2479: // 16-bit float intrinsics
2480: //===---------------------------------------------------------------------===//
2481: def ROCDL_CvtPkRtz:
2482:     ROCDL_IntrOp<"cvt.pkrtz", [], [], [Pure], 1>,
2483:     Arguments<(ins F32:$srcA, F32:$srcB)> {
2484:   let summary = "Convert two f32 input into a vector<2xf16>";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2473:** This line contributes implementation detail or declarative structure to the file.
  **CN L2473:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2474:** This line contributes implementation detail or declarative structure to the file.
  **CN L2474:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2475:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2475:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2476:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2476:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2477:** Blank line used to separate nearby declarations and improve readability.
  **CN L2477:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2478:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2478:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2479:** This comment states: “16-bit float intrinsics”, documenting the intent of the surrounding code.
  **CN L2479:** 该注释写道：“16-bit float intrinsics”，用于说明周围代码的意图。
- **EN L2480:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2480:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2481:** This TableGen `def` record introduces `ROCDL_CvtPkRtz`, which later participates in generated MLIR code.
  **CN L2481:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkRtz`，后续会参与生成的 MLIR 代码。
- **EN L2482:** This line contributes implementation detail or declarative structure to the file.
  **CN L2482:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2483:** This line contributes implementation detail or declarative structure to the file.
  **CN L2483:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2484:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2484:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2485-2496 / 第 2485-2496 行

```tablegen
2485:   let description = [{
2486:     Convert two f32 values into a packed vector<2xf16>.
2487: 
2488:     Example:
2489:     ```mlir
2490:     // Pack two f32 values into a vector<2xf16> with round-to-zero.
2491:     %0 = rocdl.cvt.pkrtz %a, %b : vector<2xf16>
2492:     ```
2493:   }];
2494:   let assemblyFormat = [{
2495:     attr-dict $srcA `,` $srcB `:` type($res)
2496:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2485:** This line contributes implementation detail or declarative structure to the file.
  **CN L2485:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2486:** This line contributes implementation detail or declarative structure to the file.
  **CN L2486:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2487:** Blank line used to separate nearby declarations and improve readability.
  **CN L2487:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2488:** This line contributes implementation detail or declarative structure to the file.
  **CN L2488:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2489:** This line contributes implementation detail or declarative structure to the file.
  **CN L2489:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2490:** This comment states: “Pack two f32 values into a vector<2xf16> with round-to-zero.”, documenting the intent of the surrounding code.
  **CN L2490:** 该注释写道：“Pack two f32 values into a vector<2xf16> with round-to-zero.”，用于说明周围代码的意图。
- **EN L2491:** This line contributes implementation detail or declarative structure to the file.
  **CN L2491:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2492:** This line contributes implementation detail or declarative structure to the file.
  **CN L2492:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2493:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2493:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2494:** This line contributes implementation detail or declarative structure to the file.
  **CN L2494:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2495:** This line contributes to the declaration or call of `type`.
  **CN L2495:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2496:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2496:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2497-2508 / 第 2497-2508 行

```tablegen
2497: }
2498: 
2499: //===---------------------------------------------------------------------===//
2500: // 8-bit float intrinsics
2501: //===---------------------------------------------------------------------===//
2502: def ROCDL_CvtF32Bf8Op :
2503:     ROCDL_ConcreteNonMemIntrOp<"cvt.f32.bf8", [Pure], 1, [1], ["byteSel"]>,
2504:     Arguments<(ins I32:$srcA, I32Attr:$byteSel)> {
2505:   let summary = "Convert bf8 to f32";
2506:   let description = [{
2507:     Convert 8-bit bf8 value from the `byteSel`th bit of `srcA` to fp32.
2508: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2497:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2497:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2498:** Blank line used to separate nearby declarations and improve readability.
  **CN L2498:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2499:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2499:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2500:** This comment states: “8-bit float intrinsics”, documenting the intent of the surrounding code.
  **CN L2500:** 该注释写道：“8-bit float intrinsics”，用于说明周围代码的意图。
- **EN L2501:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2501:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2502:** This TableGen `def` record introduces `ROCDL_CvtF32Bf8Op`, which later participates in generated MLIR code.
  **CN L2502:** 该 TableGen `def` 记录引入了 `ROCDL_CvtF32Bf8Op`，后续会参与生成的 MLIR 代码。
- **EN L2503:** This line contributes implementation detail or declarative structure to the file.
  **CN L2503:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2504:** This line contributes implementation detail or declarative structure to the file.
  **CN L2504:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2505:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2505:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2506:** This line contributes implementation detail or declarative structure to the file.
  **CN L2506:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2507:** This line contributes implementation detail or declarative structure to the file.
  **CN L2507:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2508:** Blank line used to separate nearby declarations and improve readability.
  **CN L2508:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2509-2520 / 第 2509-2520 行

```tablegen
2509:     Example:
2510:     ```mlir
2511:     // Convert bf8 byte 0 to f32.
2512:     %0 = rocdl.cvt.f32.bf8 %src[0] : f32
2513:     ```
2514:   }];
2515:   let assemblyFormat = [{
2516:     attr-dict $srcA `[` $byteSel `]` `:` type($res)
2517:   }];
2518: }
2519: 
2520: def ROCDL_CvtF32Fp8Op :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2509:** This line contributes implementation detail or declarative structure to the file.
  **CN L2509:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2510:** This line contributes implementation detail or declarative structure to the file.
  **CN L2510:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2511:** This comment states: “Convert bf8 byte 0 to f32.”, documenting the intent of the surrounding code.
  **CN L2511:** 该注释写道：“Convert bf8 byte 0 to f32.”，用于说明周围代码的意图。
- **EN L2512:** This line contributes implementation detail or declarative structure to the file.
  **CN L2512:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2513:** This line contributes implementation detail or declarative structure to the file.
  **CN L2513:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2514:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2514:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2515:** This line contributes implementation detail or declarative structure to the file.
  **CN L2515:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2516:** This line contributes to the declaration or call of `type`.
  **CN L2516:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2517:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2517:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2518:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2518:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2519:** Blank line used to separate nearby declarations and improve readability.
  **CN L2519:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2520:** This TableGen `def` record introduces `ROCDL_CvtF32Fp8Op`, which later participates in generated MLIR code.
  **CN L2520:** 该 TableGen `def` 记录引入了 `ROCDL_CvtF32Fp8Op`，后续会参与生成的 MLIR 代码。

### Lines 2521-2532 / 第 2521-2532 行

```tablegen
2521:     ROCDL_ConcreteNonMemIntrOp<"cvt.f32.fp8", [Pure], 1, [1], ["byteSel"]>,
2522:     Arguments<(ins I32:$srcA, I32Attr:$byteSel)> {
2523:   let summary = "Convert fp8 to f32";
2524:   let description = [{
2525:     Convert 8-bit fp8 value from the `byteSel`th bit of `srcA` to fp32.
2526: 
2527:     Example:
2528:     ```mlir
2529:     // Convert fp8 byte 0 to f32.
2530:     %0 = rocdl.cvt.f32.fp8 %src[0] : f32
2531:     ```
2532:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2521:** This line contributes implementation detail or declarative structure to the file.
  **CN L2521:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2522:** This line contributes implementation detail or declarative structure to the file.
  **CN L2522:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2523:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2523:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2524:** This line contributes implementation detail or declarative structure to the file.
  **CN L2524:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2525:** This line contributes implementation detail or declarative structure to the file.
  **CN L2525:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2526:** Blank line used to separate nearby declarations and improve readability.
  **CN L2526:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2527:** This line contributes implementation detail or declarative structure to the file.
  **CN L2527:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2528:** This line contributes implementation detail or declarative structure to the file.
  **CN L2528:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2529:** This comment states: “Convert fp8 byte 0 to f32.”, documenting the intent of the surrounding code.
  **CN L2529:** 该注释写道：“Convert fp8 byte 0 to f32.”，用于说明周围代码的意图。
- **EN L2530:** This line contributes implementation detail or declarative structure to the file.
  **CN L2530:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2531:** This line contributes implementation detail or declarative structure to the file.
  **CN L2531:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2532:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2532:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2533-2544 / 第 2533-2544 行

```tablegen
2533:   let assemblyFormat = [{
2534:     attr-dict $srcA `[` $byteSel `]` `:` type($res)
2535:   }];
2536: }
2537: 
2538: def ROCDL_CvtPkF32Fp8Op :
2539:     ROCDL_ConcreteNonMemIntrOp<"cvt.pk.f32.fp8", [Pure], 1, [1], ["wordSel"]>,
2540:     Arguments<(ins I32:$src, I1Attr:$wordSel)> {
2541:   let summary = "Convert packed fp8 to packed f32";
2542:   let description = [{
2543:     Convert `src` based on $wordSel to packed fp32.
2544: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2533:** This line contributes implementation detail or declarative structure to the file.
  **CN L2533:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2534:** This line contributes to the declaration or call of `type`.
  **CN L2534:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2535:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2535:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2536:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2536:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2537:** Blank line used to separate nearby declarations and improve readability.
  **CN L2537:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2538:** This TableGen `def` record introduces `ROCDL_CvtPkF32Fp8Op`, which later participates in generated MLIR code.
  **CN L2538:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkF32Fp8Op`，后续会参与生成的 MLIR 代码。
- **EN L2539:** This line contributes implementation detail or declarative structure to the file.
  **CN L2539:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2540:** This line contributes implementation detail or declarative structure to the file.
  **CN L2540:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2541:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2541:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2542:** This line contributes implementation detail or declarative structure to the file.
  **CN L2542:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2543:** This line contributes implementation detail or declarative structure to the file.
  **CN L2543:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2544:** Blank line used to separate nearby declarations and improve readability.
  **CN L2544:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2545-2556 / 第 2545-2556 行

```tablegen
2545:     Example:
2546:     ```mlir
2547:     // Unpack fp8 word to packed f32.
2548:     %0 = rocdl.cvt.pk.f32.fp8 %src[false] : vector<2xf32>
2549:     ```
2550:   }];
2551:   let assemblyFormat = [{
2552:     attr-dict $src `[` $wordSel `]` `:` type($res)
2553:   }];
2554: }
2555: 
2556: def ROCDL_CvtPkF32Bf8Op :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2545:** This line contributes implementation detail or declarative structure to the file.
  **CN L2545:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2546:** This line contributes implementation detail or declarative structure to the file.
  **CN L2546:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2547:** This comment states: “Unpack fp8 word to packed f32.”, documenting the intent of the surrounding code.
  **CN L2547:** 该注释写道：“Unpack fp8 word to packed f32.”，用于说明周围代码的意图。
- **EN L2548:** This line contributes implementation detail or declarative structure to the file.
  **CN L2548:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2549:** This line contributes implementation detail or declarative structure to the file.
  **CN L2549:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2550:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2550:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2551:** This line contributes implementation detail or declarative structure to the file.
  **CN L2551:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2552:** This line contributes to the declaration or call of `type`.
  **CN L2552:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2553:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2553:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2554:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2554:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2555:** Blank line used to separate nearby declarations and improve readability.
  **CN L2555:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2556:** This TableGen `def` record introduces `ROCDL_CvtPkF32Bf8Op`, which later participates in generated MLIR code.
  **CN L2556:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkF32Bf8Op`，后续会参与生成的 MLIR 代码。

### Lines 2557-2568 / 第 2557-2568 行

```tablegen
2557:     ROCDL_ConcreteNonMemIntrOp<"cvt.pk.f32.bf8", [Pure], 1, [1], ["wordSel"]>,
2558:     Arguments<(ins I32:$src, I1Attr:$wordSel)> {
2559:   let summary = "Convert packed bf8 to packed f32";
2560:   let description = [{
2561:     Convert `src` based on $wordSel to packed fp32.
2562: 
2563:     Example:
2564:     ```mlir
2565:     // Unpack bf8 word to packed f32.
2566:     %0 = rocdl.cvt.pk.f32.bf8 %src[false] : vector<2xf32>
2567:     ```
2568:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2557:** This line contributes implementation detail or declarative structure to the file.
  **CN L2557:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2558:** This line contributes implementation detail or declarative structure to the file.
  **CN L2558:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2559:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2559:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2560:** This line contributes implementation detail or declarative structure to the file.
  **CN L2560:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2561:** This line contributes implementation detail or declarative structure to the file.
  **CN L2561:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2562:** Blank line used to separate nearby declarations and improve readability.
  **CN L2562:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2563:** This line contributes implementation detail or declarative structure to the file.
  **CN L2563:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2564:** This line contributes implementation detail or declarative structure to the file.
  **CN L2564:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2565:** This comment states: “Unpack bf8 word to packed f32.”, documenting the intent of the surrounding code.
  **CN L2565:** 该注释写道：“Unpack bf8 word to packed f32.”，用于说明周围代码的意图。
- **EN L2566:** This line contributes implementation detail or declarative structure to the file.
  **CN L2566:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2567:** This line contributes implementation detail or declarative structure to the file.
  **CN L2567:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2568:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2568:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2569-2580 / 第 2569-2580 行

```tablegen
2569:   let assemblyFormat = [{
2570:     attr-dict $src `[` $wordSel `]` `:` type($res)
2571:   }];
2572: }
2573: 
2574: def ROCDL_CvtPkBf8F32Op :
2575:     ROCDL_ConcreteNonMemIntrOp<"cvt.pk.bf8.f32", [Pure], 1, [3], ["wordSel"]>,
2576:     Arguments<(ins F32:$srcA, F32:$srcB, I32:$old, I1Attr:$wordSel)> {
2577:   let summary = "Convert two f32's to bf8";
2578:   let description = [{
2579:     Convert `srcA` and `srcB` to bf8 and store into the low/high word of
2580:     `old`, preserving the other word.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2569:** This line contributes implementation detail or declarative structure to the file.
  **CN L2569:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2570:** This line contributes to the declaration or call of `type`.
  **CN L2570:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2571:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2571:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2572:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2572:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2573:** Blank line used to separate nearby declarations and improve readability.
  **CN L2573:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2574:** This TableGen `def` record introduces `ROCDL_CvtPkBf8F32Op`, which later participates in generated MLIR code.
  **CN L2574:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkBf8F32Op`，后续会参与生成的 MLIR 代码。
- **EN L2575:** This line contributes implementation detail or declarative structure to the file.
  **CN L2575:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2576:** This line contributes implementation detail or declarative structure to the file.
  **CN L2576:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2577:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2577:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2578:** This line contributes implementation detail or declarative structure to the file.
  **CN L2578:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2579:** This line contributes implementation detail or declarative structure to the file.
  **CN L2579:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2580:** This line contributes implementation detail or declarative structure to the file.
  **CN L2580:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2581-2592 / 第 2581-2592 行

```tablegen
2581: 
2582:     Example:
2583:     ```mlir
2584:     // Pack two f32 values into bf8 in the low word of old.
2585:     %0 = rocdl.cvt.pk.bf8.f32 %a, %b -> %old[false] : i32
2586:     ```
2587:   }];
2588:   let assemblyFormat = [{
2589:     attr-dict $srcA `,` $srcB `->` $old `[` $wordSel `]` `:` type($res)
2590:   }];
2591: }
2592: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2581:** Blank line used to separate nearby declarations and improve readability.
  **CN L2581:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2582:** This line contributes implementation detail or declarative structure to the file.
  **CN L2582:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2583:** This line contributes implementation detail or declarative structure to the file.
  **CN L2583:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2584:** This comment states: “Pack two f32 values into bf8 in the low word of old.”, documenting the intent of the surrounding code.
  **CN L2584:** 该注释写道：“Pack two f32 values into bf8 in the low word of old.”，用于说明周围代码的意图。
- **EN L2585:** This line contributes implementation detail or declarative structure to the file.
  **CN L2585:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2586:** This line contributes implementation detail or declarative structure to the file.
  **CN L2586:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2587:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2587:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2588:** This line contributes implementation detail or declarative structure to the file.
  **CN L2588:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2589:** This line contributes to the declaration or call of `type`.
  **CN L2589:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2590:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2590:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2591:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2591:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2592:** Blank line used to separate nearby declarations and improve readability.
  **CN L2592:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2593-2604 / 第 2593-2604 行

```tablegen
2593: def ROCDL_CvtPkFp8F32Op :
2594:     ROCDL_ConcreteNonMemIntrOp<"cvt.pk.fp8.f32", [Pure], 1, [3], ["wordSel"]>,
2595:     Arguments<(ins F32:$srcA, F32:$srcB, I32:$old, I1Attr:$wordSel)> {
2596:   let summary = "Convert two f32's to fp8";
2597:   let description = [{
2598:     Convert `srcA` and `srcB` to fp8 and store into the low/high word of
2599:     `old`, preserving the other word.
2600: 
2601:     Example:
2602:     ```mlir
2603:     // Pack two f32 values into fp8 in the low word of old.
2604:     %0 = rocdl.cvt.pk.fp8.f32 %a, %b -> %old[false] : i32
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2593:** This TableGen `def` record introduces `ROCDL_CvtPkFp8F32Op`, which later participates in generated MLIR code.
  **CN L2593:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkFp8F32Op`，后续会参与生成的 MLIR 代码。
- **EN L2594:** This line contributes implementation detail or declarative structure to the file.
  **CN L2594:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2595:** This line contributes implementation detail or declarative structure to the file.
  **CN L2595:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2596:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2596:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2597:** This line contributes implementation detail or declarative structure to the file.
  **CN L2597:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2598:** This line contributes implementation detail or declarative structure to the file.
  **CN L2598:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2599:** This line contributes implementation detail or declarative structure to the file.
  **CN L2599:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2600:** Blank line used to separate nearby declarations and improve readability.
  **CN L2600:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2601:** This line contributes implementation detail or declarative structure to the file.
  **CN L2601:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2602:** This line contributes implementation detail or declarative structure to the file.
  **CN L2602:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2603:** This comment states: “Pack two f32 values into fp8 in the low word of old.”, documenting the intent of the surrounding code.
  **CN L2603:** 该注释写道：“Pack two f32 values into fp8 in the low word of old.”，用于说明周围代码的意图。
- **EN L2604:** This line contributes implementation detail or declarative structure to the file.
  **CN L2604:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2605-2616 / 第 2605-2616 行

```tablegen
2605:     ```
2606:   }];
2607:   let assemblyFormat = [{
2608:     attr-dict $srcA `,` $srcB `->` $old `[` $wordSel `]` `:` type($res)
2609:   }];
2610: }
2611: 
2612: def ROCDL_CvtSrBf8F32Op :
2613:     ROCDL_ConcreteNonMemIntrOp<"cvt.sr.bf8.f32", [Pure], 1, [3], ["byteSel"]>,
2614:     Arguments<(ins F32:$srcA, I32:$srcB, I32:$old, I32Attr:$byteSel)> {
2615:   let summary = "Convert f32 to bf8, stochiastic rounding";
2616:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2605:** This line contributes implementation detail or declarative structure to the file.
  **CN L2605:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2606:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2606:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2607:** This line contributes implementation detail or declarative structure to the file.
  **CN L2607:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2608:** This line contributes to the declaration or call of `type`.
  **CN L2608:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2609:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2609:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2610:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2610:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2611:** Blank line used to separate nearby declarations and improve readability.
  **CN L2611:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2612:** This TableGen `def` record introduces `ROCDL_CvtSrBf8F32Op`, which later participates in generated MLIR code.
  **CN L2612:** 该 TableGen `def` 记录引入了 `ROCDL_CvtSrBf8F32Op`，后续会参与生成的 MLIR 代码。
- **EN L2613:** This line contributes implementation detail or declarative structure to the file.
  **CN L2613:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2614:** This line contributes implementation detail or declarative structure to the file.
  **CN L2614:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2615:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2615:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2616:** This line contributes implementation detail or declarative structure to the file.
  **CN L2616:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2617-2628 / 第 2617-2628 行

```tablegen
2617:     Convert `srcA` to bf8, adding the rounding factor from `srcB`,
2618:     and store into the `byteSel`th byte of `old`, preserving the others.
2619: 
2620:     Example:
2621:     ```mlir
2622:     // Stochastic rounding convert f32 to bf8 in byte 2 of old.
2623:     %0 = rocdl.cvt.sr.bf8.f32 %val, %stoch -> %old[2] : i32
2624:     ```
2625:   }];
2626:   let assemblyFormat = [{
2627:     attr-dict $srcA `,` $srcB `->` $old `[` $byteSel `]` `:` type($res)
2628:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2617:** This line contributes implementation detail or declarative structure to the file.
  **CN L2617:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2618:** This line contributes implementation detail or declarative structure to the file.
  **CN L2618:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2619:** Blank line used to separate nearby declarations and improve readability.
  **CN L2619:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2620:** This line contributes implementation detail or declarative structure to the file.
  **CN L2620:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2621:** This line contributes implementation detail or declarative structure to the file.
  **CN L2621:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2622:** This comment states: “Stochastic rounding convert f32 to bf8 in byte 2 of old.”, documenting the intent of the surrounding code.
  **CN L2622:** 该注释写道：“Stochastic rounding convert f32 to bf8 in byte 2 of old.”，用于说明周围代码的意图。
- **EN L2623:** This line contributes implementation detail or declarative structure to the file.
  **CN L2623:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2624:** This line contributes implementation detail or declarative structure to the file.
  **CN L2624:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2625:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2625:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2626:** This line contributes implementation detail or declarative structure to the file.
  **CN L2626:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2627:** This line contributes to the declaration or call of `type`.
  **CN L2627:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2628:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2628:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2629-2640 / 第 2629-2640 行

```tablegen
2629: }
2630: 
2631: def ROCDL_CvtSrFp8F32Op :
2632:     ROCDL_ConcreteNonMemIntrOp<"cvt.sr.fp8.f32", [Pure], 1, [3], ["byteSel"]>,
2633:     Arguments<(ins F32:$srcA, I32:$srcB, I32:$old, I32Attr:$byteSel)> {
2634:   let summary = "Convert f32 to fp8, stochiastic rounding";
2635:   let description = [{
2636:     Convert `srcA` to fp8, adding the rounding factor from `srcB`,
2637:     and store into the `byteSel`th byte of `old`, preserving the others.
2638: 
2639:     Example:
2640:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2629:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2629:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2630:** Blank line used to separate nearby declarations and improve readability.
  **CN L2630:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2631:** This TableGen `def` record introduces `ROCDL_CvtSrFp8F32Op`, which later participates in generated MLIR code.
  **CN L2631:** 该 TableGen `def` 记录引入了 `ROCDL_CvtSrFp8F32Op`，后续会参与生成的 MLIR 代码。
- **EN L2632:** This line contributes implementation detail or declarative structure to the file.
  **CN L2632:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2633:** This line contributes implementation detail or declarative structure to the file.
  **CN L2633:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2634:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2634:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2635:** This line contributes implementation detail or declarative structure to the file.
  **CN L2635:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2636:** This line contributes implementation detail or declarative structure to the file.
  **CN L2636:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2637:** This line contributes implementation detail or declarative structure to the file.
  **CN L2637:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2638:** Blank line used to separate nearby declarations and improve readability.
  **CN L2638:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2639:** This line contributes implementation detail or declarative structure to the file.
  **CN L2639:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2640:** This line contributes implementation detail or declarative structure to the file.
  **CN L2640:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2641-2652 / 第 2641-2652 行

```tablegen
2641:     // Stochastic rounding convert f32 to fp8 in byte 3 of old.
2642:     %0 = rocdl.cvt.sr.fp8.f32 %val, %stoch -> %old[3] : i32
2643:     ```
2644:   }];
2645:   let assemblyFormat = [{
2646:     attr-dict $srcA `,` $srcB `->` $old `[` $byteSel `]` `:` type($res)
2647:   }];
2648: }
2649: 
2650: //===---------------------------------------------------------------------===//
2651: // Scaled float conversion intrinsics
2652: //
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2641:** This comment states: “Stochastic rounding convert f32 to fp8 in byte 3 of old.”, documenting the intent of the surrounding code.
  **CN L2641:** 该注释写道：“Stochastic rounding convert f32 to fp8 in byte 3 of old.”，用于说明周围代码的意图。
- **EN L2642:** This line contributes implementation detail or declarative structure to the file.
  **CN L2642:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2643:** This line contributes implementation detail or declarative structure to the file.
  **CN L2643:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2644:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2644:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2645:** This line contributes implementation detail or declarative structure to the file.
  **CN L2645:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2646:** This line contributes to the declaration or call of `type`.
  **CN L2646:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2647:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2647:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2648:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2648:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2649:** Blank line used to separate nearby declarations and improve readability.
  **CN L2649:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2650:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2650:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2651:** This comment states: “Scaled float conversion intrinsics”, documenting the intent of the surrounding code.
  **CN L2651:** 该注释写道：“Scaled float conversion intrinsics”，用于说明周围代码的意图。
- **EN L2652:** This comment documents context for the surrounding code.
  **CN L2652:** 该注释为周围代码提供上下文说明。

### Lines 2653-2664 / 第 2653-2664 行

```tablegen
2653: // These are using some tablegen trickery to avoid repetitive documentation
2654: //===---------------------------------------------------------------------===//
2655: 
2656: // Pair used so we can iterate over types..
2657: class ScaleArgInfo<TypeConstraint argTyVal, string typeName> {
2658:   TypeConstraint type = argTyVal;
2659:   string name = !tolower(typeName);
2660:   string nameForOp = typeName;
2661: }
2662: 
2663: //===---------------------------------------------------------------------===//
2664: // Scaled {fp4,bf8,fp8} to {bf16,f16,f32} conversion intrinsics
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2653:** This comment states: “These are using some tablegen trickery to avoid repetitive documentation”, documenting the intent of the surrounding code.
  **CN L2653:** 该注释写道：“These are using some tablegen trickery to avoid repetitive documentation”，用于说明周围代码的意图。
- **EN L2654:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2654:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2655:** Blank line used to separate nearby declarations and improve readability.
  **CN L2655:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2656:** This comment states: “Pair used so we can iterate over types..”, documenting the intent of the surrounding code.
  **CN L2656:** 该注释写道：“Pair used so we can iterate over types..”，用于说明周围代码的意图。
- **EN L2657:** This TableGen `class` record introduces `ScaleArgInfo`, which later participates in generated MLIR code.
  **CN L2657:** 该 TableGen `class` 记录引入了 `ScaleArgInfo`，后续会参与生成的 MLIR 代码。
- **EN L2658:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2658:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2659:** This line contributes to the declaration or call of `tolower`.
  **CN L2659:** 这一行为 `tolower` 的声明或调用提供内容。
- **EN L2660:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2660:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2661:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2661:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2662:** Blank line used to separate nearby declarations and improve readability.
  **CN L2662:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2663:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2663:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2664:** This comment states: “Scaled {fp4,bf8,fp8} to {bf16,f16,f32} conversion intrinsics”, documenting the intent of the surrounding code.
  **CN L2664:** 该注释写道：“Scaled {fp4,bf8,fp8} to {bf16,f16,f32} conversion intrinsics”，用于说明周围代码的意图。

### Lines 2665-2676 / 第 2665-2676 行

```tablegen
2665: //===---------------------------------------------------------------------===//
2666: foreach smallT = [
2667:   ScaleArgInfo<I32, "Fp4">,
2668:   ScaleArgInfo<ROCDL_V2I32Type, "Fp8">,
2669:   ScaleArgInfo<ROCDL_V2I32Type, "Bf8">
2670: ] in {
2671:   foreach largeT = [
2672:     ScaleArgInfo<ROCDL_V8F16Type, "F16">,
2673:     ScaleArgInfo<ROCDL_V8BF16Type, "Bf16">,
2674:     ScaleArgInfo<ROCDL_V8F32Type, "F32">,
2675:   ] in {
2676: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2665:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2665:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2666:** This line contributes implementation detail or declarative structure to the file.
  **CN L2666:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2667:** This line contributes implementation detail or declarative structure to the file.
  **CN L2667:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2668:** This line contributes implementation detail or declarative structure to the file.
  **CN L2668:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2669:** This line contributes implementation detail or declarative structure to the file.
  **CN L2669:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2670:** This line contributes implementation detail or declarative structure to the file.
  **CN L2670:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2671:** This line contributes implementation detail or declarative structure to the file.
  **CN L2671:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2672:** This line contributes implementation detail or declarative structure to the file.
  **CN L2672:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2673:** This line contributes implementation detail or declarative structure to the file.
  **CN L2673:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2674:** This line contributes implementation detail or declarative structure to the file.
  **CN L2674:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2675:** This line contributes implementation detail or declarative structure to the file.
  **CN L2675:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2676:** Blank line used to separate nearby declarations and improve readability.
  **CN L2676:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2677-2688 / 第 2677-2688 行

```tablegen
2677:     // Up-scaling
2678:     def ROCDL_CvtPkScalePk8 # largeT.nameForOp # smallT.nameForOp # Op :
2679:           ROCDL_ConcreteNonMemIntrOp<"cvt.scale.pk8." # largeT.name # "." # smallT.name,
2680:           [Pure], 1, [2], ["scaleSel"]>,
2681:         Arguments<(ins smallT.type:$src, I32:$scale, I32Attr:$scaleSel)> {
2682: 
2683:       let summary = "Scales 8 " # smallT.name # " and converts them to 8 " # largeT.name # ".";
2684:       let description = [{
2685:         Available on gfx1250+.
2686:       }];
2687:       let results = (outs largeT.type:$res);
2688:       let assemblyFormat = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2677:** This comment states: “Up-scaling”, documenting the intent of the surrounding code.
  **CN L2677:** 该注释写道：“Up-scaling”，用于说明周围代码的意图。
- **EN L2678:** This TableGen `def` record introduces `ROCDL_CvtPkScalePk8`, which later participates in generated MLIR code.
  **CN L2678:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkScalePk8`，后续会参与生成的 MLIR 代码。
- **EN L2679:** This line contributes implementation detail or declarative structure to the file.
  **CN L2679:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2680:** This line contributes implementation detail or declarative structure to the file.
  **CN L2680:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2681:** This line contributes implementation detail or declarative structure to the file.
  **CN L2681:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2682:** Blank line used to separate nearby declarations and improve readability.
  **CN L2682:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2683:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2683:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2684:** This line contributes implementation detail or declarative structure to the file.
  **CN L2684:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2685:** This line contributes implementation detail or declarative structure to the file.
  **CN L2685:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2686:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2686:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2687:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2687:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2688:** This line contributes implementation detail or declarative structure to the file.
  **CN L2688:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2689-2700 / 第 2689-2700 行

```tablegen
2689:         attr-dict $src `,` $scale `[` $scaleSel `]` `:` type($res)
2690:       }];
2691:     }
2692: 
2693:     // Down-scaling
2694:     def ROCDL_CvtScaleF32Pk8 # smallT.nameForOp # largeT.nameForOp # Op :
2695:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk8." # smallT.name # "." # largeT.name,
2696:           [Pure], 1>,
2697:         Arguments<(ins largeT.type:$src, F32:$scale)> {
2698:       let results = (outs smallT.type:$res);
2699:       let summary = "Scale and convert packed "
2700:         # largeT.name # " to packed " # smallT.name ;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2689:** This line contributes to the declaration or call of `type`.
  **CN L2689:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2690:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2690:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2691:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2691:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2692:** Blank line used to separate nearby declarations and improve readability.
  **CN L2692:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2693:** This comment states: “Down-scaling”, documenting the intent of the surrounding code.
  **CN L2693:** 该注释写道：“Down-scaling”，用于说明周围代码的意图。
- **EN L2694:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk8`, which later participates in generated MLIR code.
  **CN L2694:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk8`，后续会参与生成的 MLIR 代码。
- **EN L2695:** This line contributes implementation detail or declarative structure to the file.
  **CN L2695:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2696:** This line contributes implementation detail or declarative structure to the file.
  **CN L2696:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2697:** This line contributes implementation detail or declarative structure to the file.
  **CN L2697:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2698:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2698:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2699:** This line contributes implementation detail or declarative structure to the file.
  **CN L2699:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2700:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2700:** 该预处理指令在主体声明处理前调整编译行为。

### Lines 2701-2712 / 第 2701-2712 行

```tablegen
2701:      let description = [{
2702:         Convert 8 packed }] # largeT.name # [{ values to packed }]
2703:         # smallT.name # [{, multiplying by the exponent part of `scale`
2704:         before doing so. This op is for gfx1250+ arch.
2705:       }];
2706:       let assemblyFormat = [{
2707:         attr-dict $src `,` $scale `:` type($res)
2708:       }];
2709:     }
2710: 
2711: 
2712:     def ROCDL_CvtScaleF32SrPk8 # smallT.nameForOp # largeT.nameForOp # Op :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2701:** This line contributes implementation detail or declarative structure to the file.
  **CN L2701:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2702:** This line contributes implementation detail or declarative structure to the file.
  **CN L2702:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2703:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2703:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2704:** This line contributes implementation detail or declarative structure to the file.
  **CN L2704:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2705:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2705:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2706:** This line contributes implementation detail or declarative structure to the file.
  **CN L2706:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2707:** This line contributes to the declaration or call of `type`.
  **CN L2707:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2708:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2708:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2709:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2709:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2710:** Blank line used to separate nearby declarations and improve readability.
  **CN L2710:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2711:** Blank line used to separate nearby declarations and improve readability.
  **CN L2711:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2712:** This TableGen `def` record introduces `ROCDL_CvtScaleF32SrPk8`, which later participates in generated MLIR code.
  **CN L2712:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32SrPk8`，后续会参与生成的 MLIR 代码。

### Lines 2713-2724 / 第 2713-2724 行

```tablegen
2713:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.sr.pk8." # smallT.name # "." # largeT.name,
2714:           [Pure], 1>,
2715:         Arguments<(ins largeT.type:$src, I32:$seed, F32:$scale)> {
2716:       let results = (outs smallT.type:$res);
2717:       let summary = "Scale and convert packed "
2718:         # largeT.name # " to packed " # smallT.name # " with stochastic rounding";
2719:      let description = [{
2720:         Convert 8 packed }] # largeT.name # [{ values to packed }]
2721:         # smallT.name # [{, multiplying by the exponent part of `scale`
2722:         before doing so and apply stochastic rounding. This op is for gfx1250+ arch.
2723:       }];
2724:       let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2713:** This line contributes implementation detail or declarative structure to the file.
  **CN L2713:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2714:** This line contributes implementation detail or declarative structure to the file.
  **CN L2714:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2715:** This line contributes implementation detail or declarative structure to the file.
  **CN L2715:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2716:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2716:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2717:** This line contributes implementation detail or declarative structure to the file.
  **CN L2717:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2718:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2718:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2719:** This line contributes implementation detail or declarative structure to the file.
  **CN L2719:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2720:** This line contributes implementation detail or declarative structure to the file.
  **CN L2720:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2721:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2721:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2722:** This line contributes implementation detail or declarative structure to the file.
  **CN L2722:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2723:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2723:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2724:** This line contributes implementation detail or declarative structure to the file.
  **CN L2724:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2725-2736 / 第 2725-2736 行

```tablegen
2725:         attr-dict $src `,` $seed `,` $scale `:` type($res)
2726:       }];
2727:     }
2728:   } // foreach largeT
2729: } // foreach smallTOp
2730: 
2731: //===---------------------------------------------------------------------===//
2732: // Scaled {bf6,fp6} to {bf16,f16,f32} conversion intrinsics
2733: //===---------------------------------------------------------------------===//
2734: foreach smallT = [
2735:   ScaleArgInfo<ROCDL_V3I32Type, "Fp6">,
2736:   ScaleArgInfo<ROCDL_V3I32Type, "Bf6">
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2725:** This line contributes to the declaration or call of `type`.
  **CN L2725:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2726:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2726:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2727:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2727:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2728:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2728:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2729:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2729:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2730:** Blank line used to separate nearby declarations and improve readability.
  **CN L2730:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2731:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2731:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2732:** This comment states: “Scaled {bf6,fp6} to {bf16,f16,f32} conversion intrinsics”, documenting the intent of the surrounding code.
  **CN L2732:** 该注释写道：“Scaled {bf6,fp6} to {bf16,f16,f32} conversion intrinsics”，用于说明周围代码的意图。
- **EN L2733:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2733:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2734:** This line contributes implementation detail or declarative structure to the file.
  **CN L2734:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2735:** This line contributes implementation detail or declarative structure to the file.
  **CN L2735:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2736:** This line contributes implementation detail or declarative structure to the file.
  **CN L2736:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2737-2748 / 第 2737-2748 行

```tablegen
2737: ] in {
2738:   foreach largeT = [
2739:     ScaleArgInfo<ROCDL_V16F16Type, "F16">,
2740:     ScaleArgInfo<ROCDL_V16BF16Type, "Bf16">,
2741:     ScaleArgInfo<ROCDL_V16F32Type, "F32">,
2742:   ] in {
2743:     // Up-scaling
2744:     def ROCDL_CvtPkScalePk16 # largeT.nameForOp # smallT.nameForOp # Op :
2745:           ROCDL_ConcreteNonMemIntrOp<"cvt.scale.pk16." # largeT.name # "." # smallT.name,
2746:           [Pure], 1, [2], ["scaleSel"]>,
2747:         Arguments<(ins smallT.type:$src, I32:$scale, I32Attr:$scaleSel)> {
2748: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2737:** This line contributes implementation detail or declarative structure to the file.
  **CN L2737:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2738:** This line contributes implementation detail or declarative structure to the file.
  **CN L2738:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2739:** This line contributes implementation detail or declarative structure to the file.
  **CN L2739:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2740:** This line contributes implementation detail or declarative structure to the file.
  **CN L2740:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2741:** This line contributes implementation detail or declarative structure to the file.
  **CN L2741:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2742:** This line contributes implementation detail or declarative structure to the file.
  **CN L2742:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2743:** This comment states: “Up-scaling”, documenting the intent of the surrounding code.
  **CN L2743:** 该注释写道：“Up-scaling”，用于说明周围代码的意图。
- **EN L2744:** This TableGen `def` record introduces `ROCDL_CvtPkScalePk16`, which later participates in generated MLIR code.
  **CN L2744:** 该 TableGen `def` 记录引入了 `ROCDL_CvtPkScalePk16`，后续会参与生成的 MLIR 代码。
- **EN L2745:** This line contributes implementation detail or declarative structure to the file.
  **CN L2745:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2746:** This line contributes implementation detail or declarative structure to the file.
  **CN L2746:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2747:** This line contributes implementation detail or declarative structure to the file.
  **CN L2747:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2748:** Blank line used to separate nearby declarations and improve readability.
  **CN L2748:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2749-2760 / 第 2749-2760 行

```tablegen
2749:       let summary = "Scales 16 " # smallT.name # " and converts them to 16 " # largeT.name # ".";
2750:       let description = [{
2751:         Available on gfx1250+.
2752:       }];
2753:       let results = (outs largeT.type:$res);
2754:       let assemblyFormat = [{
2755:         attr-dict $src `,` $scale `[` $scaleSel `]` `:` type($res)
2756:       }];
2757: 
2758:     }
2759: 
2760:     // Down-scaling
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2749:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2749:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2750:** This line contributes implementation detail or declarative structure to the file.
  **CN L2750:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2751:** This line contributes implementation detail or declarative structure to the file.
  **CN L2751:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2752:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2752:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2753:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2753:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2754:** This line contributes implementation detail or declarative structure to the file.
  **CN L2754:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2755:** This line contributes to the declaration or call of `type`.
  **CN L2755:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2756:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2756:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2757:** Blank line used to separate nearby declarations and improve readability.
  **CN L2757:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2758:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2758:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2759:** Blank line used to separate nearby declarations and improve readability.
  **CN L2759:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2760:** This comment states: “Down-scaling”, documenting the intent of the surrounding code.
  **CN L2760:** 该注释写道：“Down-scaling”，用于说明周围代码的意图。

### Lines 2761-2772 / 第 2761-2772 行

```tablegen
2761:     def ROCDL_CvtScaleF32Pk16 # smallT.nameForOp # largeT.nameForOp # Op :
2762:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk16." # smallT.name # "." # largeT.name,
2763:           [Pure], 1>,
2764:         Arguments<(ins largeT.type:$src, F32:$scale)> {
2765:       let results = (outs smallT.type:$res);
2766:       let summary = "Scale and convert packed "
2767:         # largeT.name # " to packed " # smallT.name ;
2768:      let description = [{
2769:         Convert 8 packed }] # largeT.name # [{ values to packed }]
2770:         # smallT.name # [{, multiplying by the exponent part of `scale`
2771:         before doing so. This op is for gfx1250+ arch.
2772:       }];
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2761:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk16`, which later participates in generated MLIR code.
  **CN L2761:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk16`，后续会参与生成的 MLIR 代码。
- **EN L2762:** This line contributes implementation detail or declarative structure to the file.
  **CN L2762:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2763:** This line contributes implementation detail or declarative structure to the file.
  **CN L2763:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2764:** This line contributes implementation detail or declarative structure to the file.
  **CN L2764:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2765:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2765:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2766:** This line contributes implementation detail or declarative structure to the file.
  **CN L2766:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2767:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2767:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2768:** This line contributes implementation detail or declarative structure to the file.
  **CN L2768:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2769:** This line contributes implementation detail or declarative structure to the file.
  **CN L2769:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2770:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2770:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2771:** This line contributes implementation detail or declarative structure to the file.
  **CN L2771:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2772:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2772:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2773-2784 / 第 2773-2784 行

```tablegen
2773:       let assemblyFormat = [{
2774:         attr-dict $src `,` $scale `:` type($res)
2775:       }];
2776:     }
2777: 
2778:     def ROCDL_CvtScaleF32SrPk16 # smallT.nameForOp # largeT.nameForOp # Op :
2779:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.sr.pk16." # smallT.name # "." # largeT.name,
2780:           [Pure], 1>,
2781:         Arguments<(ins largeT.type:$src, I32:$seed, F32:$scale)> {
2782:       let results = (outs smallT.type:$res);
2783:       let summary = "Scale and convert packed "
2784:         # largeT.name # " to packed " # smallT.name # " with stochastic rounding";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2773:** This line contributes implementation detail or declarative structure to the file.
  **CN L2773:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2774:** This line contributes to the declaration or call of `type`.
  **CN L2774:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2775:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2775:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2776:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2776:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2777:** Blank line used to separate nearby declarations and improve readability.
  **CN L2777:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2778:** This TableGen `def` record introduces `ROCDL_CvtScaleF32SrPk16`, which later participates in generated MLIR code.
  **CN L2778:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32SrPk16`，后续会参与生成的 MLIR 代码。
- **EN L2779:** This line contributes implementation detail or declarative structure to the file.
  **CN L2779:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2780:** This line contributes implementation detail or declarative structure to the file.
  **CN L2780:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2781:** This line contributes implementation detail or declarative structure to the file.
  **CN L2781:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2782:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2782:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2783:** This line contributes implementation detail or declarative structure to the file.
  **CN L2783:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2784:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2784:** 该预处理指令在主体声明处理前调整编译行为。

### Lines 2785-2796 / 第 2785-2796 行

```tablegen
2785:      let description = [{
2786:         Convert 8 packed }] # largeT.name # [{ values to packed }]
2787:         # smallT.name # [{, multiplying by the exponent part of `scale`
2788:         before doing so and apply stochastic rounding. This op is for gfx1250+ arch.
2789:       }];
2790:       let assemblyFormat = [{
2791:         attr-dict $src `,` $seed `,` $scale `:` type($res)
2792:       }];
2793:     }
2794: 
2795:   } // foreach largeT
2796: } // foreach smallTOp
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2785:** This line contributes implementation detail or declarative structure to the file.
  **CN L2785:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2786:** This line contributes implementation detail or declarative structure to the file.
  **CN L2786:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2787:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2787:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2788:** This line contributes implementation detail or declarative structure to the file.
  **CN L2788:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2789:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2789:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2790:** This line contributes implementation detail or declarative structure to the file.
  **CN L2790:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2791:** This line contributes to the declaration or call of `type`.
  **CN L2791:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2792:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2792:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2793:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2793:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2794:** Blank line used to separate nearby declarations and improve readability.
  **CN L2794:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2795:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2795:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2796:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2796:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2797-2808 / 第 2797-2808 行

```tablegen
2797: 
2798: //===---------------------------------------------------------------------===//
2799: // Scaled 32x6-bit float float conversion intrinsics
2800: //===---------------------------------------------------------------------===//
2801: foreach smallT = [
2802:    // MLIR f6E2M3FN
2803:   ScaleArgInfo<ROCDL_V6I32Type, "Fp6">,
2804:    // MLIR f8E3M2FN
2805:   ScaleArgInfo<ROCDL_V6I32Type, "Bf6">
2806: ] in {
2807:   foreach largeT = [
2808:     ScaleArgInfo<ROCDL_V32F16Type, "F16">,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2797:** Blank line used to separate nearby declarations and improve readability.
  **CN L2797:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2798:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2798:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2799:** This comment states: “Scaled 32x6-bit float float conversion intrinsics”, documenting the intent of the surrounding code.
  **CN L2799:** 该注释写道：“Scaled 32x6-bit float float conversion intrinsics”，用于说明周围代码的意图。
- **EN L2800:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2800:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2801:** This line contributes implementation detail or declarative structure to the file.
  **CN L2801:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2802:** This comment states: “MLIR f6E2M3FN”, documenting the intent of the surrounding code.
  **CN L2802:** 该注释写道：“MLIR f6E2M3FN”，用于说明周围代码的意图。
- **EN L2803:** This line contributes implementation detail or declarative structure to the file.
  **CN L2803:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2804:** This comment states: “MLIR f8E3M2FN”, documenting the intent of the surrounding code.
  **CN L2804:** 该注释写道：“MLIR f8E3M2FN”，用于说明周围代码的意图。
- **EN L2805:** This line contributes implementation detail or declarative structure to the file.
  **CN L2805:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2806:** This line contributes implementation detail or declarative structure to the file.
  **CN L2806:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2807:** This line contributes implementation detail or declarative structure to the file.
  **CN L2807:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2808:** This line contributes implementation detail or declarative structure to the file.
  **CN L2808:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2809-2820 / 第 2809-2820 行

```tablegen
2809:     ScaleArgInfo<ROCDL_V32BF16Type, "Bf16">,
2810:     ScaleArgInfo<ROCDL_V32F32Type, "F32">,
2811:   ] in {
2812:     // Note: rouding down f32 values has a special case where
2813:     // we have to use 2 16xf32 arguments.
2814:     if !ne(largeT.name, "f32") then {
2815:       def ROCDL_CvtScaleF32Pk32 # smallT.nameForOp # largeT.nameForOp # Op :
2816:           ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk32." # smallT.name # "." # largeT.name,
2817:             [Pure], 1>,
2818:           Arguments<(ins largeT.type:$src, F32:$scale)> {
2819:         let results = (outs smallT.type:$res);
2820:         let summary = "Scale and convert packed "
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2809:** This line contributes implementation detail or declarative structure to the file.
  **CN L2809:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2810:** This line contributes implementation detail or declarative structure to the file.
  **CN L2810:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2811:** This line contributes implementation detail or declarative structure to the file.
  **CN L2811:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2812:** This comment states: “Note: rouding down f32 values has a special case where”, documenting the intent of the surrounding code.
  **CN L2812:** 该注释写道：“Note: rouding down f32 values has a special case where”，用于说明周围代码的意图。
- **EN L2813:** This comment states: “we have to use 2 16xf32 arguments.”, documenting the intent of the surrounding code.
  **CN L2813:** 该注释写道：“we have to use 2 16xf32 arguments.”，用于说明周围代码的意图。
- **EN L2814:** This line contributes implementation detail or declarative structure to the file.
  **CN L2814:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2815:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk32`, which later participates in generated MLIR code.
  **CN L2815:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk32`，后续会参与生成的 MLIR 代码。
- **EN L2816:** This line contributes implementation detail or declarative structure to the file.
  **CN L2816:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2817:** This line contributes implementation detail or declarative structure to the file.
  **CN L2817:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2818:** This line contributes implementation detail or declarative structure to the file.
  **CN L2818:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2819:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2819:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2820:** This line contributes implementation detail or declarative structure to the file.
  **CN L2820:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2821-2832 / 第 2821-2832 行

```tablegen
2821:           # largeT.name # " to packed " # smallT.name;
2822:         let description = [{
2823:           Convert 32 packed }] # largeT.name # [{ values to packed }]
2824:           # smallT.name # [{, dividing by the exponent part of `scale`
2825:           before doing so.
2826:         }];
2827:         let assemblyFormat = [{
2828:           attr-dict $src `,` $scale `:` type($res)
2829:         }];
2830:       }
2831:     } // if
2832: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2821:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2821:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2822:** This line contributes implementation detail or declarative structure to the file.
  **CN L2822:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2823:** This line contributes implementation detail or declarative structure to the file.
  **CN L2823:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2824:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2824:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2825:** This line contributes implementation detail or declarative structure to the file.
  **CN L2825:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2826:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2826:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2827:** This line contributes implementation detail or declarative structure to the file.
  **CN L2827:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2828:** This line contributes to the declaration or call of `type`.
  **CN L2828:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2829:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2829:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2830:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2830:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2831:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2831:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2832:** Blank line used to separate nearby declarations and improve readability.
  **CN L2832:** 该空行用于分隔相邻声明并提升可读性。

### Lines 2833-2844 / 第 2833-2844 行

```tablegen
2833:     def ROCDL_CvtScaleF32SrPk32 # smallT.nameForOp # largeT.nameForOp # Op :
2834:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.sr.pk32." # smallT.name # "." # largeT.name,
2835:           [Pure], 1>,
2836:         Arguments<(ins largeT.type:$src, I32:$seed, F32:$scale)> {
2837:       let results = (outs smallT.type:$res);
2838:       let summary = "Scale and convert packed "
2839:         # largeT.name # " to packed " # smallT.name
2840:         # " with stochiastic rounding";
2841:       let description = [{
2842:         Convert 32 packed }] # largeT.name # [{ values to packed }]
2843:         # smallT.name # [{, dividing by the exponent part of `scale`
2844:         before doing so and applying random rounding derived from
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2833:** This TableGen `def` record introduces `ROCDL_CvtScaleF32SrPk32`, which later participates in generated MLIR code.
  **CN L2833:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32SrPk32`，后续会参与生成的 MLIR 代码。
- **EN L2834:** This line contributes implementation detail or declarative structure to the file.
  **CN L2834:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2835:** This line contributes implementation detail or declarative structure to the file.
  **CN L2835:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2836:** This line contributes implementation detail or declarative structure to the file.
  **CN L2836:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2837:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2837:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2838:** This line contributes implementation detail or declarative structure to the file.
  **CN L2838:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2839:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2839:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2840:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2840:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2841:** This line contributes implementation detail or declarative structure to the file.
  **CN L2841:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2842:** This line contributes implementation detail or declarative structure to the file.
  **CN L2842:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2843:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2843:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2844:** This line contributes implementation detail or declarative structure to the file.
  **CN L2844:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2845-2856 / 第 2845-2856 行

```tablegen
2845:         `seed`.
2846:       }];
2847:       let assemblyFormat = [{
2848:         attr-dict $src `,` $seed `,` $scale `:` type($res)
2849:       }];
2850:     }
2851: 
2852:     def ROCDL_CvtScaleF32Pk32 # largeT.nameForOp # smallT.nameForOp # Op :
2853:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk32." # largeT.name # "." # smallT.name,
2854:           [Pure], 1>,
2855:         Arguments<(ins smallT.type:$src, F32:$scale)> {
2856:       let results = (outs largeT.type:$res);
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2845:** This line contributes implementation detail or declarative structure to the file.
  **CN L2845:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2846:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2846:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2847:** This line contributes implementation detail or declarative structure to the file.
  **CN L2847:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2848:** This line contributes to the declaration or call of `type`.
  **CN L2848:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2849:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2849:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2850:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2850:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2851:** Blank line used to separate nearby declarations and improve readability.
  **CN L2851:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2852:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk32`, which later participates in generated MLIR code.
  **CN L2852:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk32`，后续会参与生成的 MLIR 代码。
- **EN L2853:** This line contributes implementation detail or declarative structure to the file.
  **CN L2853:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2854:** This line contributes implementation detail or declarative structure to the file.
  **CN L2854:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2855:** This line contributes implementation detail or declarative structure to the file.
  **CN L2855:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2856:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2856:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2857-2868 / 第 2857-2868 行

```tablegen
2857:       let summary = "Scale and convert packed "
2858:         # smallT.name # " to packed " # largeT.name;
2859:       let description = [{
2860:         Convert 32 packed }] # smallT.name # [{ values to packed }]
2861:         # largeT.name # [{, multiplying by the exponent part of `scale`
2862:         before doing so.
2863:       }];
2864:       let assemblyFormat = [{
2865:         attr-dict $src `,` $scale `:` type($res)
2866:       }];
2867:     }
2868:   } // foreach largeT
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2857:** This line contributes implementation detail or declarative structure to the file.
  **CN L2857:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2858:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2858:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2859:** This line contributes implementation detail or declarative structure to the file.
  **CN L2859:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2860:** This line contributes implementation detail or declarative structure to the file.
  **CN L2860:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2861:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2861:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2862:** This line contributes implementation detail or declarative structure to the file.
  **CN L2862:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2863:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2863:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2864:** This line contributes implementation detail or declarative structure to the file.
  **CN L2864:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2865:** This line contributes to the declaration or call of `type`.
  **CN L2865:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2866:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2866:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2867:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2867:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2868:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2868:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 2869-2880 / 第 2869-2880 行

```tablegen
2869: 
2870:   def ROCDL_CvtScaleF322xPk16 # smallT.nameForOp # F32Op :
2871:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.2xpk16." # smallT.name # ".f32",
2872:         [Pure], 1>,
2873:       Arguments<(ins ROCDL_V16F32Type:$src0, ROCDL_V16F32Type:$src1, F32:$scale)> {
2874:     let results = (outs smallT.type:$res);
2875:     let summary = "Scale and convert two vector<16xf32> to 32 packed " # smallT.name;
2876:     let description = [{
2877:       Convert 32 single-precision float values, packed into two length-16
2878:       vectors that will be logically concanenated, to packed }]
2879:       # smallT.name # [{, dividing by the exponent part of `scale`
2880:       before doing so.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2869:** Blank line used to separate nearby declarations and improve readability.
  **CN L2869:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2870:** This TableGen `def` record introduces `ROCDL_CvtScaleF322xPk16`, which later participates in generated MLIR code.
  **CN L2870:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF322xPk16`，后续会参与生成的 MLIR 代码。
- **EN L2871:** This line contributes implementation detail or declarative structure to the file.
  **CN L2871:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2872:** This line contributes implementation detail or declarative structure to the file.
  **CN L2872:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2873:** This line contributes implementation detail or declarative structure to the file.
  **CN L2873:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2874:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2874:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2875:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2875:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2876:** This line contributes implementation detail or declarative structure to the file.
  **CN L2876:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2877:** This line contributes implementation detail or declarative structure to the file.
  **CN L2877:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2878:** This line contributes implementation detail or declarative structure to the file.
  **CN L2878:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2879:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2879:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2880:** This line contributes implementation detail or declarative structure to the file.
  **CN L2880:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2881-2892 / 第 2881-2892 行

```tablegen
2881:     }];
2882:    let assemblyFormat = [{
2883:       attr-dict $src0 `,` $src1 `,` $scale `:` type($res)
2884:     }];
2885:   }
2886: } // forach smallT
2887: 
2888: //===---------------------------------------------------------------------===//
2889: // Scaled conversions to/from fp8/bf8 (f8E4M3FN / f8E5M2)
2890: //===---------------------------------------------------------------------===//
2891: foreach smallTOp = ["Fp8", "Bf8"] in {
2892:   defvar smallT = !tolower(smallTOp);
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2881:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2881:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2882:** This line contributes implementation detail or declarative structure to the file.
  **CN L2882:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2883:** This line contributes to the declaration or call of `type`.
  **CN L2883:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2884:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2884:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2885:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2885:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2886:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2886:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2887:** Blank line used to separate nearby declarations and improve readability.
  **CN L2887:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2888:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2888:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2889:** This comment states: “Scaled conversions to/from fp8/bf8 (f8E4M3FN / f8E5M2)”, documenting the intent of the surrounding code.
  **CN L2889:** 该注释写道：“Scaled conversions to/from fp8/bf8 (f8E4M3FN / f8E5M2)”，用于说明周围代码的意图。
- **EN L2890:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L2890:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L2891:** This line contributes implementation detail or declarative structure to the file.
  **CN L2891:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2892:** This line contributes to the declaration or call of `tolower`.
  **CN L2892:** 这一行为 `tolower` 的声明或调用提供内容。

### Lines 2893-2904 / 第 2893-2904 行

```tablegen
2893: 
2894:   def ROCDL_CvtScaleF32F16 # smallTOp # Op :
2895:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.f16." # smallT,
2896:       [Pure], 1, [3, 4], ["srcSelIndex", "dstLoHiSel"]>,
2897:     Arguments<(ins ROCDL_V2F16Type:$oldVdst, I32:$src, F32:$scale, I32Attr:$srcSelIndex, I1Attr:$dstLoHiSel)> {
2898:     let results = (outs ROCDL_V2F16Type:$res);
2899:     let summary = "Scaled convert " # smallT # " from packed vector to f16, updating tied result";
2900:     let description = [{
2901:       Convert a }] # smallT # [{ byte from `src`, selected by
2902:       `srcSelIndex`, to f16 while multiplying it by the expontent of `scale`,
2903:       and place it into the `dstLoHiSel`th bit
2904:       of `oldVdst` preserving the other element of that vector in
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2893:** Blank line used to separate nearby declarations and improve readability.
  **CN L2893:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2894:** This TableGen `def` record introduces `ROCDL_CvtScaleF32F16`, which later participates in generated MLIR code.
  **CN L2894:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32F16`，后续会参与生成的 MLIR 代码。
- **EN L2895:** This line contributes implementation detail or declarative structure to the file.
  **CN L2895:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2896:** This line contributes implementation detail or declarative structure to the file.
  **CN L2896:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2897:** This line contributes implementation detail or declarative structure to the file.
  **CN L2897:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2898:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2898:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2899:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2899:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2900:** This line contributes implementation detail or declarative structure to the file.
  **CN L2900:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2901:** This line contributes implementation detail or declarative structure to the file.
  **CN L2901:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2902:** This line contributes implementation detail or declarative structure to the file.
  **CN L2902:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2903:** This line contributes implementation detail or declarative structure to the file.
  **CN L2903:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2904:** This line contributes implementation detail or declarative structure to the file.
  **CN L2904:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2905-2916 / 第 2905-2916 行

```tablegen
2905:       the return value.
2906: 
2907:       The bytes are stored as an `i32` and not a `<4 x i8>`.
2908:     }];
2909:     let assemblyFormat = [{
2910:       attr-dict $src `[` $srcSelIndex `]` `,` $scale `->` $oldVdst `[` $dstLoHiSel `]` `:` type($res)
2911:     }];
2912:   }
2913: 
2914:   def ROCDL_CvtScaleF32F32 # smallTOp # Op :
2915:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.f32." # smallT,
2916:       [Pure], 1, [2], ["srcSelIndex"]>,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2905:** This line contributes implementation detail or declarative structure to the file.
  **CN L2905:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2906:** Blank line used to separate nearby declarations and improve readability.
  **CN L2906:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2907:** This line contributes implementation detail or declarative structure to the file.
  **CN L2907:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2908:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2908:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2909:** This line contributes implementation detail or declarative structure to the file.
  **CN L2909:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2910:** This line contributes to the declaration or call of `type`.
  **CN L2910:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2911:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2911:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2912:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2912:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2913:** Blank line used to separate nearby declarations and improve readability.
  **CN L2913:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2914:** This TableGen `def` record introduces `ROCDL_CvtScaleF32F32`, which later participates in generated MLIR code.
  **CN L2914:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32F32`，后续会参与生成的 MLIR 代码。
- **EN L2915:** This line contributes implementation detail or declarative structure to the file.
  **CN L2915:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2916:** This line contributes implementation detail or declarative structure to the file.
  **CN L2916:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2917-2928 / 第 2917-2928 行

```tablegen
2917:     Arguments<(ins I32:$src, F32:$scale, I32Attr:$srcSelIndex)> {
2918:     let results = (outs F32:$res);
2919:     let summary = "Scaled convert " # smallT # " from packed vector to f32";
2920:     let description = [{
2921:       Convert a }] # smallT # [{ byte from `src`, selected by
2922:       `srcSelIndex`, to f32, multiplying it by the exponent of `scale`.
2923: 
2924:       The bytes are stored in an `i32`, not a `<4 x i8>`.
2925:     }];
2926:     let assemblyFormat = [{
2927:       attr-dict $src `[` $srcSelIndex `]` `,` $scale `:` type($res)
2928:     }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2917:** This line contributes implementation detail or declarative structure to the file.
  **CN L2917:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2918:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2918:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2919:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2919:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2920:** This line contributes implementation detail or declarative structure to the file.
  **CN L2920:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2921:** This line contributes implementation detail or declarative structure to the file.
  **CN L2921:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2922:** This line contributes implementation detail or declarative structure to the file.
  **CN L2922:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2923:** Blank line used to separate nearby declarations and improve readability.
  **CN L2923:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2924:** This line contributes implementation detail or declarative structure to the file.
  **CN L2924:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2925:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2925:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2926:** This line contributes implementation detail or declarative structure to the file.
  **CN L2926:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2927:** This line contributes to the declaration or call of `type`.
  **CN L2927:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2928:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2928:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 2929-2940 / 第 2929-2940 行

```tablegen
2929:   }
2930: 
2931:   def ROCDL_CvtScaleF32Pk # smallTOp # F32Op :
2932:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk." # smallT # ".f32",
2933:       [Pure], 1, [4], ["dstLoHiSel"]>,
2934:     Arguments<(ins ROCDL_V2I16Type:$oldVdst, F32:$src0, F32:$src1, F32:$scale, I1Attr:$dstLoHiSel)> {
2935:     let results = (outs ROCDL_V2I16Type:$res);
2936:     let summary = "Scaled convert two f32 to two " # smallT # ", updating packed vector";
2937:     let description = [{
2938:       Convert two f32 values in `src0` and `src1` to two }] # smallT # [{ bytes,
2939:       dividing by the exponent in `scale`. The bytes are packed into
2940:       a 16-bit value which is inserted into `oldVdst` at the `dstLoHiSel`
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2929:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2929:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2930:** Blank line used to separate nearby declarations and improve readability.
  **CN L2930:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2931:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk`, which later participates in generated MLIR code.
  **CN L2931:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk`，后续会参与生成的 MLIR 代码。
- **EN L2932:** This line contributes implementation detail or declarative structure to the file.
  **CN L2932:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2933:** This line contributes implementation detail or declarative structure to the file.
  **CN L2933:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2934:** This line contributes implementation detail or declarative structure to the file.
  **CN L2934:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2935:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2935:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2936:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2936:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2937:** This line contributes implementation detail or declarative structure to the file.
  **CN L2937:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2938:** This line contributes implementation detail or declarative structure to the file.
  **CN L2938:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2939:** This line contributes implementation detail or declarative structure to the file.
  **CN L2939:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2940:** This line contributes implementation detail or declarative structure to the file.
  **CN L2940:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2941-2952 / 第 2941-2952 行

```tablegen
2941:       position, with the entire updated vector being returned.
2942:     }];
2943:     let assemblyFormat = [{
2944:       attr-dict  $src0 `,` $src1 `,` $scale `->` $oldVdst `[` $dstLoHiSel `]` `:` type($res)
2945:     }];
2946:   }
2947: 
2948:   foreach largeT = [
2949:     ScaleArgInfo<ROCDL_V2F16Type, "F16">,
2950:     ScaleArgInfo<ROCDL_V2BF16Type, "Bf16">,
2951:   ] in {
2952:     def ROCDL_CvtScaleF32Pk # smallTOp # largeT.nameForOp # Op :
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2941:** This line contributes implementation detail or declarative structure to the file.
  **CN L2941:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2942:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2942:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2943:** This line contributes implementation detail or declarative structure to the file.
  **CN L2943:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2944:** This line contributes to the declaration or call of `type`.
  **CN L2944:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2945:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2945:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2946:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2946:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2947:** Blank line used to separate nearby declarations and improve readability.
  **CN L2947:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2948:** This line contributes implementation detail or declarative structure to the file.
  **CN L2948:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2949:** This line contributes implementation detail or declarative structure to the file.
  **CN L2949:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2950:** This line contributes implementation detail or declarative structure to the file.
  **CN L2950:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2951:** This line contributes implementation detail or declarative structure to the file.
  **CN L2951:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2952:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk`, which later participates in generated MLIR code.
  **CN L2952:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk`，后续会参与生成的 MLIR 代码。

### Lines 2953-2964 / 第 2953-2964 行

```tablegen
2953:           ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk." # smallT # "." # largeT.name,
2954:           [Pure], 1, [3], ["dstLoHiSel"]>,
2955:         Arguments<(ins ROCDL_V2I16Type:$oldVdst, largeT.type:$src0, F32:$scale, I1Attr:$dstLoHiSel)> {
2956:       let results = (outs ROCDL_V2I16Type:$res);
2957:       let summary = "Scaled convert two " # largeT.name # "to two " # smallT # ", updating packed vector";
2958:       let description = [{
2959:         Convert two }] # largeT.name # [{ values in `src0` to two }]
2960:         # smallT # [{ bytes, dividing by the exponent in `scale`. The bytes are
2961:         packed into a 16-bit value which is inserted into `oldVdst` at the
2962:         `dstLoHiSel` position, with the entire updated vector being returned.
2963:       }];
2964:       let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2953:** This line contributes implementation detail or declarative structure to the file.
  **CN L2953:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2954:** This line contributes implementation detail or declarative structure to the file.
  **CN L2954:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2955:** This line contributes implementation detail or declarative structure to the file.
  **CN L2955:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2956:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2956:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2957:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2957:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2958:** This line contributes implementation detail or declarative structure to the file.
  **CN L2958:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2959:** This line contributes implementation detail or declarative structure to the file.
  **CN L2959:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2960:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2960:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2961:** This line contributes implementation detail or declarative structure to the file.
  **CN L2961:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2962:** This line contributes implementation detail or declarative structure to the file.
  **CN L2962:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2963:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2963:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2964:** This line contributes implementation detail or declarative structure to the file.
  **CN L2964:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2965-2976 / 第 2965-2976 行

```tablegen
2965:         attr-dict $src0 `,` $scale `->` $oldVdst `[` $dstLoHiSel `]` `:` type($res)
2966:       }];
2967:     }
2968:   } // foreach largeT
2969: 
2970:   foreach largeT = [
2971:     ScaleArgInfo<ROCDL_V2F16Type, "F16">,
2972:     ScaleArgInfo<ROCDL_V2BF16Type, "Bf16">,
2973:     ScaleArgInfo<ROCDL_V2F32Type, "F32">
2974:   ] in {
2975:     def ROCDL_CvtScaleF32Pk # largeT.nameForOp # smallTOp # Op :
2976:           ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk." # largeT.name # "." # smallT,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2965:** This line contributes to the declaration or call of `type`.
  **CN L2965:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2966:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2966:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2967:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2967:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2968:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2968:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2969:** Blank line used to separate nearby declarations and improve readability.
  **CN L2969:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2970:** This line contributes implementation detail or declarative structure to the file.
  **CN L2970:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2971:** This line contributes implementation detail or declarative structure to the file.
  **CN L2971:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2972:** This line contributes implementation detail or declarative structure to the file.
  **CN L2972:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2973:** This line contributes implementation detail or declarative structure to the file.
  **CN L2973:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2974:** This line contributes implementation detail or declarative structure to the file.
  **CN L2974:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2975:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk`, which later participates in generated MLIR code.
  **CN L2975:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk`，后续会参与生成的 MLIR 代码。
- **EN L2976:** This line contributes implementation detail or declarative structure to the file.
  **CN L2976:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2977-2988 / 第 2977-2988 行

```tablegen
2977:           [Pure], 1, [2], ["srcLoHiSel"]>,
2978:         Arguments<(ins I32:$src, F32:$scale, I1Attr:$srcLoHiSel)> {
2979:       let results = (outs largeT.type:$res);
2980:       let summary = "Scaled convert two " # smallT # "to two " # largeT.name #;
2981:       let description = [{
2982:         Convert two packed }] # smallT # [{ values in `src0` to two }]
2983:         # largeT.name # [{ values, multiplying by the exponent in `scale`.
2984:         The two values to be converted are selected from the low or high half
2985:         of `src` (a packed vector represented as an `i32`)
2986:         on the basis of `srcLoHiSel`.
2987:       }];
2988:       let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L2977:** This line contributes implementation detail or declarative structure to the file.
  **CN L2977:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2978:** This line contributes implementation detail or declarative structure to the file.
  **CN L2978:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2979:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2979:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2980:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2980:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2981:** This line contributes implementation detail or declarative structure to the file.
  **CN L2981:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2982:** This line contributes implementation detail or declarative structure to the file.
  **CN L2982:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2983:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L2983:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L2984:** This line contributes implementation detail or declarative structure to the file.
  **CN L2984:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2985:** This line contributes implementation detail or declarative structure to the file.
  **CN L2985:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2986:** This line contributes implementation detail or declarative structure to the file.
  **CN L2986:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2987:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2987:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2988:** This line contributes implementation detail or declarative structure to the file.
  **CN L2988:** 这一行为文件补充了实现细节或声明式结构。

### Lines 2989-3000 / 第 2989-3000 行

```tablegen
2989:         attr-dict $src `[` $srcLoHiSel `]` `,` $scale `:` type($res)
2990:       }];
2991:     }
2992:   } // foreach largeT
2993: 
2994:   foreach largeT = [
2995:     ScaleArgInfo<F32, "F32">,
2996:     ScaleArgInfo<F16, "F16">,
2997:     ScaleArgInfo<BF16, "BF16">
2998:   ] in {
2999:     def ROCDL_CvtScaleF32Sr # smallTOp # largeT.nameForOp # Op :
3000:           ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.sr." # smallT # "." # largeT.name,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L2989:** This line contributes to the declaration or call of `type`.
  **CN L2989:** 这一行为 `type` 的声明或调用提供内容。
- **EN L2990:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L2990:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L2991:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2991:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2992:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L2992:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L2993:** Blank line used to separate nearby declarations and improve readability.
  **CN L2993:** 该空行用于分隔相邻声明并提升可读性。
- **EN L2994:** This line contributes implementation detail or declarative structure to the file.
  **CN L2994:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2995:** This line contributes implementation detail or declarative structure to the file.
  **CN L2995:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2996:** This line contributes implementation detail or declarative structure to the file.
  **CN L2996:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2997:** This line contributes implementation detail or declarative structure to the file.
  **CN L2997:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2998:** This line contributes implementation detail or declarative structure to the file.
  **CN L2998:** 这一行为文件补充了实现细节或声明式结构。
- **EN L2999:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Sr`, which later participates in generated MLIR code.
  **CN L2999:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Sr`，后续会参与生成的 MLIR 代码。
- **EN L3000:** This line contributes implementation detail or declarative structure to the file.
  **CN L3000:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3001-3012 / 第 3001-3012 行

```tablegen
3001:           [Pure], 1, [4], ["dstSelIndex"]>,
3002:         Arguments<(ins I32:$oldVdst, largeT.type:$src0, I32:$seed, F32:$scale, I32Attr:$dstSelIndex)> {
3003:       let results = (outs I32:$res);
3004:       let summary = "Scaled convert " # largeT.name # "to " # smallT # " with stochiastic rounding, updating packed vector";
3005:       let description = [{
3006:         Convert a }] # largeT.name # [{ value in `src0` to a }]
3007:         # smallT # [{ bytes, dividing by the exponent in `scale` and using `seed`
3008:         for stochiastic rounding. Place the resulting byte in the
3009:         `dstSelIndex`th bit of `oldVdst` and return the entire packed vector,
3010:         which is stored as an `i32`.
3011:       }];
3012:       let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3001:** This line contributes implementation detail or declarative structure to the file.
  **CN L3001:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3002:** This line contributes implementation detail or declarative structure to the file.
  **CN L3002:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3003:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3003:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3004:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3004:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3005:** This line contributes implementation detail or declarative structure to the file.
  **CN L3005:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3006:** This line contributes implementation detail or declarative structure to the file.
  **CN L3006:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3007:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L3007:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L3008:** This line contributes implementation detail or declarative structure to the file.
  **CN L3008:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3009:** This line contributes implementation detail or declarative structure to the file.
  **CN L3009:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3010:** This line contributes implementation detail or declarative structure to the file.
  **CN L3010:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3011:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3011:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3012:** This line contributes implementation detail or declarative structure to the file.
  **CN L3012:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3013-3024 / 第 3013-3024 行

```tablegen
3013:         attr-dict $src0 `,` $seed `,` $scale `->` $oldVdst `[` $dstSelIndex `]` `:` type($res)
3014:       }];
3015:     }
3016:   } // foreach largeT
3017: } // foreach smallTOp
3018: 
3019: //===---------------------------------------------------------------------===//
3020: // Scaled conversions to/from fp4 (f4E2M1FN)
3021: //===---------------------------------------------------------------------===//
3022: 
3023: foreach largeT = [
3024:   ScaleArgInfo<ROCDL_V2F16Type, "F16">,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3013:** This line contributes to the declaration or call of `type`.
  **CN L3013:** 这一行为 `type` 的声明或调用提供内容。
- **EN L3014:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3014:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3015:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3015:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3016:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3016:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3017:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3017:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3018:** Blank line used to separate nearby declarations and improve readability.
  **CN L3018:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3019:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3019:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3020:** This comment states: “Scaled conversions to/from fp4 (f4E2M1FN)”, documenting the intent of the surrounding code.
  **CN L3020:** 该注释写道：“Scaled conversions to/from fp4 (f4E2M1FN)”，用于说明周围代码的意图。
- **EN L3021:** This comment states: “===---------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3021:** 该注释写道：“===---------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3022:** Blank line used to separate nearby declarations and improve readability.
  **CN L3022:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3023:** This line contributes implementation detail or declarative structure to the file.
  **CN L3023:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3024:** This line contributes implementation detail or declarative structure to the file.
  **CN L3024:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3025-3036 / 第 3025-3036 行

```tablegen
3025:   ScaleArgInfo<ROCDL_V2BF16Type, "Bf16">,
3026:   ScaleArgInfo<ROCDL_V2F32Type, "F32">,
3027: ] in {
3028:   // Note: rouding down f32 values has a special case where
3029:   // we have to use 2 float arguments.
3030:   if !ne(largeT.name, "f32") then {
3031:     def ROCDL_CvtScaleF32PkFp4 # largeT.nameForOp # Op :
3032:         ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk.fp4." # largeT.name,
3033:           [Pure], 1, [3], ["dstSelIndex"]>,
3034:         Arguments<(ins I32:$oldVdst, largeT.type:$src, F32:$scale, I32Attr:$dstSelIndex)> {
3035:       let results = (outs I32:$res);
3036:       let summary = "Scale and convert two "
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3025:** This line contributes implementation detail or declarative structure to the file.
  **CN L3025:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3026:** This line contributes implementation detail or declarative structure to the file.
  **CN L3026:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3027:** This line contributes implementation detail or declarative structure to the file.
  **CN L3027:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3028:** This comment states: “Note: rouding down f32 values has a special case where”, documenting the intent of the surrounding code.
  **CN L3028:** 该注释写道：“Note: rouding down f32 values has a special case where”，用于说明周围代码的意图。
- **EN L3029:** This comment states: “we have to use 2 float arguments.”, documenting the intent of the surrounding code.
  **CN L3029:** 该注释写道：“we have to use 2 float arguments.”，用于说明周围代码的意图。
- **EN L3030:** This line contributes implementation detail or declarative structure to the file.
  **CN L3030:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3031:** This TableGen `def` record introduces `ROCDL_CvtScaleF32PkFp4`, which later participates in generated MLIR code.
  **CN L3031:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32PkFp4`，后续会参与生成的 MLIR 代码。
- **EN L3032:** This line contributes implementation detail or declarative structure to the file.
  **CN L3032:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3033:** This line contributes implementation detail or declarative structure to the file.
  **CN L3033:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3034:** This line contributes implementation detail or declarative structure to the file.
  **CN L3034:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3035:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3035:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3036:** This line contributes implementation detail or declarative structure to the file.
  **CN L3036:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3037-3048 / 第 3037-3048 行

```tablegen
3037:         # largeT.name # " to packed fp4, updating tied vector";
3038:       let description = [{
3039:         Convert two packed }] # largeT.name # [{ values to packed
3040:         fp4, dividing by the exponent part of `scale`
3041:         before doing so.
3042: 
3043:         The two scaled values are packed  into a byte.
3044:         That byte is used to update the `dstSelIndex`th
3045:         byte of `oldVdst`, which is returned in its entirity.
3046:       }];
3047:       let assemblyFormat = [{
3048:         attr-dict $src `,` $scale `->` $oldVdst `[` $dstSelIndex `]` `:` type($res)
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3037:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L3037:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L3038:** This line contributes implementation detail or declarative structure to the file.
  **CN L3038:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3039:** This line contributes implementation detail or declarative structure to the file.
  **CN L3039:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3040:** This line contributes implementation detail or declarative structure to the file.
  **CN L3040:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3041:** This line contributes implementation detail or declarative structure to the file.
  **CN L3041:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3042:** Blank line used to separate nearby declarations and improve readability.
  **CN L3042:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3043:** This line contributes implementation detail or declarative structure to the file.
  **CN L3043:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3044:** This line contributes implementation detail or declarative structure to the file.
  **CN L3044:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3045:** This line contributes implementation detail or declarative structure to the file.
  **CN L3045:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3046:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3046:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3047:** This line contributes implementation detail or declarative structure to the file.
  **CN L3047:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3048:** This line contributes to the declaration or call of `type`.
  **CN L3048:** 这一行为 `type` 的声明或调用提供内容。

### Lines 3049-3060 / 第 3049-3060 行

```tablegen
3049:       }];
3050:     }
3051:   } // if
3052: 
3053:   def ROCDL_CvtScaleF32SrPkFp4 # largeT.nameForOp # Op :
3054:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.sr.pk.fp4." # largeT.name,
3055:         [Pure], 1, [4], ["dstSelIndex"]>,
3056:       Arguments<(ins I32:$oldVdst, largeT.type:$src, I32:$seed, F32:$scale, I32Attr:$dstSelIndex)> {
3057:     let results = (outs I32:$res);
3058:     let summary = "Scale and convert two "
3059:       # largeT.name # " to packed fp4 with stochiastic rounding, updating tied vector";
3060:     let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3049:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3049:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3050:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3050:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3051:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3051:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3052:** Blank line used to separate nearby declarations and improve readability.
  **CN L3052:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3053:** This TableGen `def` record introduces `ROCDL_CvtScaleF32SrPkFp4`, which later participates in generated MLIR code.
  **CN L3053:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32SrPkFp4`，后续会参与生成的 MLIR 代码。
- **EN L3054:** This line contributes implementation detail or declarative structure to the file.
  **CN L3054:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3055:** This line contributes implementation detail or declarative structure to the file.
  **CN L3055:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3056:** This line contributes implementation detail or declarative structure to the file.
  **CN L3056:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3057:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3057:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3058:** This line contributes implementation detail or declarative structure to the file.
  **CN L3058:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3059:** This preprocessor directive adjusts compilation before the main declarations are processed.
  **CN L3059:** 该预处理指令在主体声明处理前调整编译行为。
- **EN L3060:** This line contributes implementation detail or declarative structure to the file.
  **CN L3060:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3061-3072 / 第 3061-3072 行

```tablegen
3061:       Convert two packed }] # largeT.name # [{ values to packed
3062:       fp4, dividing by the exponent part of `scale`
3063:       before doing so and using `seed` as the random seed for
3064:       stochiastic rounding.
3065: 
3066:       The two scaled values are packed (little-endian)
3067:       into a byte. That byte is used to update the `dstSelIndex`th
3068:       byte of `oldVdst`, which is returned in its entirity.
3069:     }];
3070:     let assemblyFormat = [{
3071:       attr-dict $src `,` $seed `,` $scale `->` $oldVdst `[` $dstSelIndex `]` `:` type($res)
3072:     }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3061:** This line contributes implementation detail or declarative structure to the file.
  **CN L3061:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3062:** This line contributes implementation detail or declarative structure to the file.
  **CN L3062:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3063:** This line contributes implementation detail or declarative structure to the file.
  **CN L3063:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3064:** This line contributes implementation detail or declarative structure to the file.
  **CN L3064:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3065:** Blank line used to separate nearby declarations and improve readability.
  **CN L3065:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3066:** This line contributes to the declaration or call of `packed`.
  **CN L3066:** 这一行为 `packed` 的声明或调用提供内容。
- **EN L3067:** This line contributes implementation detail or declarative structure to the file.
  **CN L3067:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3068:** This line contributes implementation detail or declarative structure to the file.
  **CN L3068:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3069:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3069:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3070:** This line contributes implementation detail or declarative structure to the file.
  **CN L3070:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3071:** This line contributes to the declaration or call of `type`.
  **CN L3071:** 这一行为 `type` 的声明或调用提供内容。
- **EN L3072:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3072:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 3073-3084 / 第 3073-3084 行

```tablegen
3073:   }
3074: 
3075:   def ROCDL_CvtScaleF32Pk # largeT.nameForOp # Fp4Op :
3076:       ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk." # largeT.name # ".fp4",
3077:         [Pure], 1, [2], ["srcSelIndex"]>,
3078:       Arguments<(ins I32:$src, F32:$scale, I32Attr:$srcSelIndex)> {
3079:     let results = (outs largeT.type:$res);
3080:     let summary = "Scale and convert two packed fp4 to packed " # largeT.name;
3081:     let description = [{
3082:       Convert two packed fp4 (f4E2M1) values  stored as one byte of a 32-bit integer
3083:       to packed }] # largeT.name # [{, multiplying by the exponent part of `scale`
3084:       before doing so.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3073:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3073:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3074:** Blank line used to separate nearby declarations and improve readability.
  **CN L3074:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3075:** This TableGen `def` record introduces `ROCDL_CvtScaleF32Pk`, which later participates in generated MLIR code.
  **CN L3075:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32Pk`，后续会参与生成的 MLIR 代码。
- **EN L3076:** This line contributes implementation detail or declarative structure to the file.
  **CN L3076:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3077:** This line contributes implementation detail or declarative structure to the file.
  **CN L3077:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3078:** This line contributes implementation detail or declarative structure to the file.
  **CN L3078:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3079:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3079:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3080:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3080:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3081:** This line contributes implementation detail or declarative structure to the file.
  **CN L3081:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3082:** This line contributes to the declaration or call of `fp4`.
  **CN L3082:** 这一行为 `fp4` 的声明或调用提供内容。
- **EN L3083:** This line contributes implementation detail or declarative structure to the file.
  **CN L3083:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3084:** This line contributes implementation detail or declarative structure to the file.
  **CN L3084:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3085-3096 / 第 3085-3096 行

```tablegen
3085: 
3086:       The byte to convert is chosen by `srcSelIndex`.
3087:     }];
3088:     let assemblyFormat = [{
3089:       attr-dict $src `[` $srcSelIndex `]` `,` $scale `:` type($res)
3090:     }];
3091:   }
3092: } // foreach largeT
3093: 
3094: def ROCDL_CvtScaleF32PkFp4F32Op :
3095:     ROCDL_ConcreteNonMemIntrOp<"cvt.scalef32.pk.fp4.f32",
3096:       [Pure], 1, [4], ["dstSelIndex"]>,
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3085:** Blank line used to separate nearby declarations and improve readability.
  **CN L3085:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3086:** This line contributes implementation detail or declarative structure to the file.
  **CN L3086:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3087:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3087:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3088:** This line contributes implementation detail or declarative structure to the file.
  **CN L3088:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3089:** This line contributes to the declaration or call of `type`.
  **CN L3089:** 这一行为 `type` 的声明或调用提供内容。
- **EN L3090:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3090:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3091:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3091:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3092:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3092:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3093:** Blank line used to separate nearby declarations and improve readability.
  **CN L3093:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3094:** This TableGen `def` record introduces `ROCDL_CvtScaleF32PkFp4F32Op`, which later participates in generated MLIR code.
  **CN L3094:** 该 TableGen `def` 记录引入了 `ROCDL_CvtScaleF32PkFp4F32Op`，后续会参与生成的 MLIR 代码。
- **EN L3095:** This line contributes implementation detail or declarative structure to the file.
  **CN L3095:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3096:** This line contributes implementation detail or declarative structure to the file.
  **CN L3096:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3097-3108 / 第 3097-3108 行

```tablegen
3097:     Arguments<(ins I32:$oldVdst, F32:$src0, F32:$src1, F32:$scale, I32Attr:$dstSelIndex)> {
3098:   let results = (outs I32:$res);
3099:   let summary = "Scale and convert two f32 values to two packed fp4, updating tied vector";
3100:   let description = [{
3101:     Convert two single-precision float values, passed in `src0` and `src1`
3102:     into two fp4 values, dividing them by the expontent part of `scale`
3103:     before doing so.
3104: 
3105:     The two scaled values are packed  into a byte.
3106:     That byte is used to update the `dstSelIndex`th
3107:     byte of `oldVdst`, which is returned in its entirity.
3108: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3097:** This line contributes implementation detail or declarative structure to the file.
  **CN L3097:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3098:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3098:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3099:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3099:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3100:** This line contributes implementation detail or declarative structure to the file.
  **CN L3100:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3101:** This line contributes implementation detail or declarative structure to the file.
  **CN L3101:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3102:** This line contributes implementation detail or declarative structure to the file.
  **CN L3102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3103:** This line contributes implementation detail or declarative structure to the file.
  **CN L3103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3104:** Blank line used to separate nearby declarations and improve readability.
  **CN L3104:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3105:** This line contributes implementation detail or declarative structure to the file.
  **CN L3105:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3106:** This line contributes implementation detail or declarative structure to the file.
  **CN L3106:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3107:** This line contributes implementation detail or declarative structure to the file.
  **CN L3107:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3108:** Blank line used to separate nearby declarations and improve readability.
  **CN L3108:** 该空行用于分隔相邻声明并提升可读性。

### Lines 3109-3120 / 第 3109-3120 行

```tablegen
3109:     Example:
3110:     ```mlir
3111:     // Scaled convert two f32 values to packed fp4 in byte 0 of old.
3112:     %0 = rocdl.cvt.scalef32.pk.fp4.f32 %a, %b, %scale -> %old[0] : i32
3113:     ```
3114:   }];
3115:  let assemblyFormat = [{
3116:     attr-dict $src0 `,` $src1 `,` $scale `->` $oldVdst `[` $dstSelIndex `]` `:` type($res)
3117:   }];
3118: }
3119: 
3120: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3109:** This line contributes implementation detail or declarative structure to the file.
  **CN L3109:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3110:** This line contributes implementation detail or declarative structure to the file.
  **CN L3110:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3111:** This comment states: “Scaled convert two f32 values to packed fp4 in byte 0 of old.”, documenting the intent of the surrounding code.
  **CN L3111:** 该注释写道：“Scaled convert two f32 values to packed fp4 in byte 0 of old.”，用于说明周围代码的意图。
- **EN L3112:** This line contributes implementation detail or declarative structure to the file.
  **CN L3112:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3113:** This line contributes implementation detail or declarative structure to the file.
  **CN L3113:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3114:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3114:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3115:** This line contributes implementation detail or declarative structure to the file.
  **CN L3115:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3116:** This line contributes to the declaration or call of `type`.
  **CN L3116:** 这一行为 `type` 的声明或调用提供内容。
- **EN L3117:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3117:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3118:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3118:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3119:** Blank line used to separate nearby declarations and improve readability.
  **CN L3119:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3120:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3120:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 3121-3132 / 第 3121-3132 行

```tablegen
3121: // FMED3 operations
3122: //===----------------------------------------------------------------------===//
3123: 
3124: def ROCDL_FMed3Op : ROCDL_IntrOp<"fmed3", [0], [], [Pure, AllTypesMatch<["res", "src0", "src1", "src2"]>], 1>,
3125:   Arguments<(ins LLVM_ScalarOrVectorOf<LLVM_AnyFloat>:$src0,
3126:                  LLVM_ScalarOrVectorOf<LLVM_AnyFloat>:$src1,
3127:                  LLVM_ScalarOrVectorOf<LLVM_AnyFloat>:$src2)> {
3128:   let results = (outs LLVM_ScalarOrVectorOf<LLVM_AnyFloat>:$res);
3129:   let summary = "Median of three float/half values";
3130:   let description = [{
3131:     Computes the median of three floating-point values using the AMDGPU fmed3 intrinsic.
3132:     This operation is equivalent to `max(min(a, b), min(max(a, b), c))` but uses the
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3121:** This comment states: “FMED3 operations”, documenting the intent of the surrounding code.
  **CN L3121:** 该注释写道：“FMED3 operations”，用于说明周围代码的意图。
- **EN L3122:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3122:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3123:** Blank line used to separate nearby declarations and improve readability.
  **CN L3123:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3124:** This TableGen `def` record introduces `ROCDL_FMed3Op`, which later participates in generated MLIR code.
  **CN L3124:** 该 TableGen `def` 记录引入了 `ROCDL_FMed3Op`，后续会参与生成的 MLIR 代码。
- **EN L3125:** This line contributes implementation detail or declarative structure to the file.
  **CN L3125:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3126:** This line contributes implementation detail or declarative structure to the file.
  **CN L3126:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3127:** This line contributes implementation detail or declarative structure to the file.
  **CN L3127:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3128:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3128:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3129:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3129:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3130:** This line contributes implementation detail or declarative structure to the file.
  **CN L3130:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3131:** This line contributes implementation detail or declarative structure to the file.
  **CN L3131:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3132:** This line contributes to the declaration or call of `max`.
  **CN L3132:** 这一行为 `max` 的声明或调用提供内容。

### Lines 3133-3144 / 第 3133-3144 行

```tablegen
3133:     hardware-accelerated V_MED3_F16/V_MED3_F32 instruction for better performance.
3134: 
3135:     The operation supports both scalar and vector floating-point types (f16, f32).
3136: 
3137:     Example:
3138:     ```mlir
3139:     // Scalar f32 median
3140:     %result = rocdl.fmed3 %a, %b, %c : f32
3141: 
3142:     // Vector f16 median
3143:     %result = rocdl.fmed3 %va, %vb, %vc : vector<4xf16>
3144:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3133:** This line contributes implementation detail or declarative structure to the file.
  **CN L3133:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3134:** Blank line used to separate nearby declarations and improve readability.
  **CN L3134:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3135:** This line contributes to the declaration or call of `types`.
  **CN L3135:** 这一行为 `types` 的声明或调用提供内容。
- **EN L3136:** Blank line used to separate nearby declarations and improve readability.
  **CN L3136:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3137:** This line contributes implementation detail or declarative structure to the file.
  **CN L3137:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3138:** This line contributes implementation detail or declarative structure to the file.
  **CN L3138:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3139:** This comment states: “Scalar f32 median”, documenting the intent of the surrounding code.
  **CN L3139:** 该注释写道：“Scalar f32 median”，用于说明周围代码的意图。
- **EN L3140:** This line contributes implementation detail or declarative structure to the file.
  **CN L3140:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3141:** Blank line used to separate nearby declarations and improve readability.
  **CN L3141:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3142:** This comment states: “Vector f16 median”, documenting the intent of the surrounding code.
  **CN L3142:** 该注释写道：“Vector f16 median”，用于说明周围代码的意图。
- **EN L3143:** This line contributes implementation detail or declarative structure to the file.
  **CN L3143:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3144:** This line contributes implementation detail or declarative structure to the file.
  **CN L3144:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3145-3156 / 第 3145-3156 行

```tablegen
3145:   }];
3146:   let assemblyFormat = [{
3147:     $src0 `,` $src1 `,` $src2 attr-dict `:` type($res)
3148:   }];
3149: }
3150: 
3151: //===----------------------------------------------------------------------===//
3152: // Math operations
3153: //===----------------------------------------------------------------------===//
3154: 
3155: class ROCDL_Math_IntrOp<string mnemonic, list<Trait> traits = [Pure]> :
3156:   ROCDL_IntrOp<mnemonic, [0], [], traits, 1>,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3145:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3145:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3146:** This line contributes implementation detail or declarative structure to the file.
  **CN L3146:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3147:** This line contributes to the declaration or call of `type`.
  **CN L3147:** 这一行为 `type` 的声明或调用提供内容。
- **EN L3148:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3148:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3149:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3149:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3150:** Blank line used to separate nearby declarations and improve readability.
  **CN L3150:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3151:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3151:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3152:** This comment states: “Math operations”, documenting the intent of the surrounding code.
  **CN L3152:** 该注释写道：“Math operations”，用于说明周围代码的意图。
- **EN L3153:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3153:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3154:** Blank line used to separate nearby declarations and improve readability.
  **CN L3154:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3155:** This TableGen `class` record introduces `ROCDL_Math_IntrOp`, which later participates in generated MLIR code.
  **CN L3155:** 该 TableGen `class` 记录引入了 `ROCDL_Math_IntrOp`，后续会参与生成的 MLIR 代码。
- **EN L3156:** This line contributes implementation detail or declarative structure to the file.
  **CN L3156:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3157-3168 / 第 3157-3168 行

```tablegen
3157:   Arguments<(ins LLVM_AnyFloat:$arg)> {
3158:   let results = (outs LLVM_AnyFloat:$res);
3159:   let description = [{
3160:     Note: In the general case, prefer the conventional `arith`, `math`, or `llvm` ops over this.
3161:     Use this ROCDL-specific operation only when you fully understand its implication and
3162:     when it is strictly necessary. This op is usually chosen when a small loss in precision is
3163:     acceptable in exchange for higher execution speed.
3164: 
3165:     Example:
3166:     ```mlir
3167:     %0 = rocdl.}] # mnemonic # [{ %a f32 -> f32
3168:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3157:** This line contributes implementation detail or declarative structure to the file.
  **CN L3157:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3158:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3158:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3159:** This line contributes implementation detail or declarative structure to the file.
  **CN L3159:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3160:** This line contributes implementation detail or declarative structure to the file.
  **CN L3160:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3161:** This line contributes implementation detail or declarative structure to the file.
  **CN L3161:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3162:** This line contributes implementation detail or declarative structure to the file.
  **CN L3162:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3163:** This line contributes implementation detail or declarative structure to the file.
  **CN L3163:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3164:** Blank line used to separate nearby declarations and improve readability.
  **CN L3164:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3165:** This line contributes implementation detail or declarative structure to the file.
  **CN L3165:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3166:** This line contributes implementation detail or declarative structure to the file.
  **CN L3166:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3167:** This line contributes implementation detail or declarative structure to the file.
  **CN L3167:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3168:** This line contributes implementation detail or declarative structure to the file.
  **CN L3168:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3169-3180 / 第 3169-3180 行

```tablegen
3169:   }];
3170:   let assemblyFormat =
3171:     "$arg qualified(type($arg)) attr-dict `->` qualified(type($res))";
3172: }
3173: 
3174: def ROCDLTanh : ROCDL_Math_IntrOp<"tanh">;
3175: def ROCDLSin : ROCDL_Math_IntrOp<"sin">;
3176: def ROCDLCos : ROCDL_Math_IntrOp<"cos">;
3177: def ROCDLRcp : ROCDL_Math_IntrOp<"rcp">;
3178: def ROCDLExp : ROCDL_Math_IntrOp<"exp">;
3179: def ROCDLExp2 : ROCDL_Math_IntrOp<"exp2">;
3180: def ROCDLLog : ROCDL_Math_IntrOp<"log">;
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3169:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3169:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3170:** This line contributes implementation detail or declarative structure to the file.
  **CN L3170:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3171:** This line contributes to the declaration or call of `qualified`.
  **CN L3171:** 这一行为 `qualified` 的声明或调用提供内容。
- **EN L3172:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3172:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3173:** Blank line used to separate nearby declarations and improve readability.
  **CN L3173:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3174:** This TableGen `def` record introduces `ROCDLTanh`, which later participates in generated MLIR code.
  **CN L3174:** 该 TableGen `def` 记录引入了 `ROCDLTanh`，后续会参与生成的 MLIR 代码。
- **EN L3175:** This TableGen `def` record introduces `ROCDLSin`, which later participates in generated MLIR code.
  **CN L3175:** 该 TableGen `def` 记录引入了 `ROCDLSin`，后续会参与生成的 MLIR 代码。
- **EN L3176:** This TableGen `def` record introduces `ROCDLCos`, which later participates in generated MLIR code.
  **CN L3176:** 该 TableGen `def` 记录引入了 `ROCDLCos`，后续会参与生成的 MLIR 代码。
- **EN L3177:** This TableGen `def` record introduces `ROCDLRcp`, which later participates in generated MLIR code.
  **CN L3177:** 该 TableGen `def` 记录引入了 `ROCDLRcp`，后续会参与生成的 MLIR 代码。
- **EN L3178:** This TableGen `def` record introduces `ROCDLExp`, which later participates in generated MLIR code.
  **CN L3178:** 该 TableGen `def` 记录引入了 `ROCDLExp`，后续会参与生成的 MLIR 代码。
- **EN L3179:** This TableGen `def` record introduces `ROCDLExp2`, which later participates in generated MLIR code.
  **CN L3179:** 该 TableGen `def` 记录引入了 `ROCDLExp2`，后续会参与生成的 MLIR 代码。
- **EN L3180:** This TableGen `def` record introduces `ROCDLLog`, which later participates in generated MLIR code.
  **CN L3180:** 该 TableGen `def` 记录引入了 `ROCDLLog`，后续会参与生成的 MLIR 代码。

### Lines 3181-3192 / 第 3181-3192 行

```tablegen
3181: def ROCDLSqrt : ROCDL_Math_IntrOp<"sqrt">;
3182: def ROCDLRsq : ROCDL_Math_IntrOp<"rsq">;
3183: 
3184: //===----------------------------------------------------------------------===//
3185: // ROCDL target attribute.
3186: //===----------------------------------------------------------------------===//
3187: 
3188: def ROCDL_TargetAttr :
3189:     ROCDL_Attr<"ROCDLTarget", "target"> {
3190:   let description = [{
3191:     ROCDL target attribute for controlling compilation of AMDGPU targets. All
3192:     parameters decay into default values if not present.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L3181:** This TableGen `def` record introduces `ROCDLSqrt`, which later participates in generated MLIR code.
  **CN L3181:** 该 TableGen `def` 记录引入了 `ROCDLSqrt`，后续会参与生成的 MLIR 代码。
- **EN L3182:** This TableGen `def` record introduces `ROCDLRsq`, which later participates in generated MLIR code.
  **CN L3182:** 该 TableGen `def` 记录引入了 `ROCDLRsq`，后续会参与生成的 MLIR 代码。
- **EN L3183:** Blank line used to separate nearby declarations and improve readability.
  **CN L3183:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3184:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3184:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3185:** This comment states: “ROCDL target attribute.”, documenting the intent of the surrounding code.
  **CN L3185:** 该注释写道：“ROCDL target attribute.”，用于说明周围代码的意图。
- **EN L3186:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L3186:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L3187:** Blank line used to separate nearby declarations and improve readability.
  **CN L3187:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3188:** This TableGen `def` record introduces `ROCDL_TargetAttr`, which later participates in generated MLIR code.
  **CN L3188:** 该 TableGen `def` 记录引入了 `ROCDL_TargetAttr`，后续会参与生成的 MLIR 代码。
- **EN L3189:** This line contributes implementation detail or declarative structure to the file.
  **CN L3189:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3190:** This line contributes implementation detail or declarative structure to the file.
  **CN L3190:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3191:** This line contributes implementation detail or declarative structure to the file.
  **CN L3191:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3192:** This line contributes implementation detail or declarative structure to the file.
  **CN L3192:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3193-3204 / 第 3193-3204 行

```tablegen
3193: 
3194:     Examples:
3195: 
3196:     1. Target with default values.
3197:     ```
3198:       gpu.module @mymodule [#rocdl.target] attributes {...} {
3199:         ...
3200:       }
3201:     ```
3202: 
3203:     2. Target with `gfx90a` chip and fast math.
3204:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3193:** Blank line used to separate nearby declarations and improve readability.
  **CN L3193:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3194:** This line contributes implementation detail or declarative structure to the file.
  **CN L3194:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3195:** Blank line used to separate nearby declarations and improve readability.
  **CN L3195:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3196:** This line contributes implementation detail or declarative structure to the file.
  **CN L3196:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3197:** This line contributes implementation detail or declarative structure to the file.
  **CN L3197:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3198:** This line contributes implementation detail or declarative structure to the file.
  **CN L3198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3199:** This line contributes implementation detail or declarative structure to the file.
  **CN L3199:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3200:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3200:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3201:** This line contributes implementation detail or declarative structure to the file.
  **CN L3201:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3202:** Blank line used to separate nearby declarations and improve readability.
  **CN L3202:** 该空行用于分隔相邻声明并提升可读性。
- **EN L3203:** This line contributes implementation detail or declarative structure to the file.
  **CN L3203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3204:** This line contributes implementation detail or declarative structure to the file.
  **CN L3204:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3205-3216 / 第 3205-3216 行

```tablegen
3205:       gpu.module @mymodule [#rocdl.target<chip = "gfx90a", flags = {fast, no_wave64}>] {
3206:         ...
3207:       }
3208:     ```
3209:   }];
3210:   let parameters = (ins
3211:     DefaultValuedParameter<"int", "2", "Optimization level to apply.">:$O,
3212:     StringRefParameter<"Target triple.", "\"amdgcn-amd-amdhsa\"">:$triple,
3213:     StringRefParameter<"Target chip.", "\"gfx900\"">:$chip,
3214:     StringRefParameter<"Target chip features.", "\"\"">:$features,
3215:     // Also update the default builder below and rocdl-attach-target in
3216:     // Dialect/GPU/Transforms/Passes.td .
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3205:** This line contributes implementation detail or declarative structure to the file.
  **CN L3205:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3206:** This line contributes implementation detail or declarative structure to the file.
  **CN L3206:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3207:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3207:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3208:** This line contributes implementation detail or declarative structure to the file.
  **CN L3208:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3209:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3209:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3210:** This line contributes implementation detail or declarative structure to the file.
  **CN L3210:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3211:** This line contributes implementation detail or declarative structure to the file.
  **CN L3211:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3212:** This line contributes implementation detail or declarative structure to the file.
  **CN L3212:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3213:** This line contributes implementation detail or declarative structure to the file.
  **CN L3213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3214:** This line contributes implementation detail or declarative structure to the file.
  **CN L3214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3215:** This comment states: “Also update the default builder below and rocdl-attach-target in”, documenting the intent of the surrounding code.
  **CN L3215:** 该注释写道：“Also update the default builder below and rocdl-attach-target in”，用于说明周围代码的意图。
- **EN L3216:** This comment states: “Dialect/GPU/Transforms/Passes.td .”, documenting the intent of the surrounding code.
  **CN L3216:** 该注释写道：“Dialect/GPU/Transforms/Passes.td .”，用于说明周围代码的意图。

### Lines 3217-3228 / 第 3217-3228 行

```tablegen
3217:     StringRefParameter<"ABI version.", "\"600\"">:$abi,
3218:     OptionalParameter<"DictionaryAttr", "Target specific flags.">:$flags,
3219:     OptionalParameter<"ArrayAttr", "Files to link to the LLVM module.">:$link
3220:   );
3221:   let assemblyFormat = [{
3222:     (`<` struct($O, $triple, $chip, $features, $abi, $flags, $link)^ `>`)?
3223:   }];
3224:   let builders = [
3225:     AttrBuilder<(ins CArg<"int", "2">:$optLevel,
3226:                      CArg<"StringRef", "\"amdgcn-amd-amdhsa\"">:$triple,
3227:                      CArg<"StringRef", "\"gfx900\"">:$chip,
3228:                      CArg<"StringRef", "\"\"">:$features,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3217:** This line contributes implementation detail or declarative structure to the file.
  **CN L3217:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3218:** This line contributes implementation detail or declarative structure to the file.
  **CN L3218:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3219:** This line contributes implementation detail or declarative structure to the file.
  **CN L3219:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3220:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3220:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3221:** This line contributes implementation detail or declarative structure to the file.
  **CN L3221:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3222:** This line contributes to the declaration or call of `struct`.
  **CN L3222:** 这一行为 `struct` 的声明或调用提供内容。
- **EN L3223:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3223:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3224:** This line contributes implementation detail or declarative structure to the file.
  **CN L3224:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3225:** This line contributes implementation detail or declarative structure to the file.
  **CN L3225:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3226:** This line contributes implementation detail or declarative structure to the file.
  **CN L3226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3227:** This line contributes implementation detail or declarative structure to the file.
  **CN L3227:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3228:** This line contributes implementation detail or declarative structure to the file.
  **CN L3228:** 这一行为文件补充了实现细节或声明式结构。

### Lines 3229-3240 / 第 3229-3240 行

```tablegen
3229:                      CArg<"StringRef", "\"600\"">:$abiVersion,
3230:                      CArg<"DictionaryAttr", "nullptr">:$targetFlags,
3231:                      CArg<"ArrayAttr", "nullptr">:$linkFiles), [{
3232:       return Base::get($_ctxt, optLevel, triple, chip, features, abiVersion,
3233:                        targetFlags, linkFiles);
3234:     }]>
3235:   ];
3236:   let skipDefaultBuilders = 1;
3237:   let genVerifyDecl = 1;
3238:   let extraClassDeclaration = [{
3239:     bool hasFlag(StringRef flag) const;
3240:     bool hasWave64() const;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3229:** This line contributes implementation detail or declarative structure to the file.
  **CN L3229:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3230:** This line contributes implementation detail or declarative structure to the file.
  **CN L3230:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3231:** This line contributes implementation detail or declarative structure to the file.
  **CN L3231:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3232:** This line contributes implementation detail or declarative structure to the file.
  **CN L3232:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3233:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3233:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3234:** This line contributes implementation detail or declarative structure to the file.
  **CN L3234:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3235:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3235:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3236:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3236:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3237:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3237:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3238:** This line contributes implementation detail or declarative structure to the file.
  **CN L3238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3239:** This line contributes to the declaration or call of `hasFlag`.
  **CN L3239:** 这一行为 `hasFlag` 的声明或调用提供内容。
- **EN L3240:** This line contributes to the declaration or call of `hasWave64`.
  **CN L3240:** 这一行为 `hasWave64` 的声明或调用提供内容。

### Lines 3241-3252 / 第 3241-3252 行

```tablegen
3241:     bool hasFastMath() const;
3242:     bool hasDaz() const;
3243:     bool hasFiniteOnly() const;
3244:     bool hasUnsafeMath() const;
3245:     bool hasCorrectSqrt() const;
3246:   }];
3247:   let extraClassDefinition = [{
3248:     bool $cppClass::hasFlag(StringRef flag) const {
3249:       if (DictionaryAttr flags = getFlags())
3250:         return flags.get(flag) != nullptr;
3251:       return false;
3252:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3241:** This line contributes to the declaration or call of `hasFastMath`.
  **CN L3241:** 这一行为 `hasFastMath` 的声明或调用提供内容。
- **EN L3242:** This line contributes to the declaration or call of `hasDaz`.
  **CN L3242:** 这一行为 `hasDaz` 的声明或调用提供内容。
- **EN L3243:** This line contributes to the declaration or call of `hasFiniteOnly`.
  **CN L3243:** 这一行为 `hasFiniteOnly` 的声明或调用提供内容。
- **EN L3244:** This line contributes to the declaration or call of `hasUnsafeMath`.
  **CN L3244:** 这一行为 `hasUnsafeMath` 的声明或调用提供内容。
- **EN L3245:** This line contributes to the declaration or call of `hasCorrectSqrt`.
  **CN L3245:** 这一行为 `hasCorrectSqrt` 的声明或调用提供内容。
- **EN L3246:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3246:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3247:** This line contributes implementation detail or declarative structure to the file.
  **CN L3247:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3248:** This line contributes to the declaration or call of `hasFlag`.
  **CN L3248:** 这一行为 `hasFlag` 的声明或调用提供内容。
- **EN L3249:** This line contributes implementation detail or declarative structure to the file.
  **CN L3249:** 这一行为文件补充了实现细节或声明式结构。
- **EN L3250:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3250:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3251:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3251:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3252:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3252:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 3253-3264 / 第 3253-3264 行

```tablegen
3253:     bool $cppClass::hasWave64() const {
3254:       return hasFlag("wave64") || !hasFlag("no_wave64");
3255:     }
3256:     bool $cppClass::hasFastMath() const {
3257:       return hasFlag("fast");
3258:     }
3259:     bool $cppClass::hasDaz() const {
3260:       return hasFlag("daz");
3261:     }
3262:     bool $cppClass::hasFiniteOnly() const {
3263:       return hasFlag("finite_only");
3264:     }
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3253:** This line contributes to the declaration or call of `hasWave64`.
  **CN L3253:** 这一行为 `hasWave64` 的声明或调用提供内容。
- **EN L3254:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3254:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3255:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3255:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3256:** This line contributes to the declaration or call of `hasFastMath`.
  **CN L3256:** 这一行为 `hasFastMath` 的声明或调用提供内容。
- **EN L3257:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3257:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3258:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3258:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3259:** This line contributes to the declaration or call of `hasDaz`.
  **CN L3259:** 这一行为 `hasDaz` 的声明或调用提供内容。
- **EN L3260:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3260:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3261:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3261:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3262:** This line contributes to the declaration or call of `hasFiniteOnly`.
  **CN L3262:** 这一行为 `hasFiniteOnly` 的声明或调用提供内容。
- **EN L3263:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3263:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3264:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3264:** 这一行结束当前作用域，例如命名空间、类或枚举块。

### Lines 3265-3273 / 第 3265-3273 行

```tablegen
3265:     bool $cppClass::hasUnsafeMath() const {
3266:       return hasFlag("unsafe_math");
3267:     }
3268:     bool $cppClass::hasCorrectSqrt() const {
3269:       return !hasFlag("unsafe_sqrt");
3270:     }
3271:   }];
3272: }
3273: #endif // ROCDLIR_OPS
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L3265:** This line contributes to the declaration or call of `hasUnsafeMath`.
  **CN L3265:** 这一行为 `hasUnsafeMath` 的声明或调用提供内容。
- **EN L3266:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3266:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3267:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3267:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3268:** This line contributes to the declaration or call of `hasCorrectSqrt`.
  **CN L3268:** 这一行为 `hasCorrectSqrt` 的声明或调用提供内容。
- **EN L3269:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3269:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3270:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3270:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3271:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L3271:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L3272:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L3272:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L3273:** This directive closes the conditional compilation region guarded by `ROCDLIR_OPS`.
  **CN L3273:** 该指令结束了由 `ROCDLIR_OPS` 保护的条件编译区域。

## Key Concepts / 关键概念

- **ROCDL_Attr**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_Op**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_IntrPure1Op**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_IntrOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_ConcreteNonMemIntrOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_SpecialIdRegisterOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_NamedType**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **ROCDL_ConcreteVector**  
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
