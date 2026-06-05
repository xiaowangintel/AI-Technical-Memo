# SparseTensor.h — Code Analysis / 代码分析

## Source / 来源
- File: `mlir/include/mlir-c/Dialect/SparseTensor.h`
- Repository: `llvm-project` (MLIR)
- Purpose (EN): This header in `mlir/include/mlir-c/Dialect` declares infrastructure centered on `MlirSparseTensorLevelFormat`, `MlirSparseTensorLevelPropertyNondefault`, `mlirAttributeIsASparseTensorEncodingAttr`, and `mlirSparseTensorEncodingAttrGet`.
- 用途 (CN): 该 头文件 位于 `mlir/include/mlir-c/Dialect`，围绕 `MlirSparseTensorLevelFormat`、`MlirSparseTensorLevelPropertyNondefault`、`mlirAttributeIsASparseTensorEncodingAttr`、`mlirSparseTensorEncodingAttrGet` 等核心符号组织声明，用于支撑对应子系统的接口、类型或配置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
   1: //===-- mlir-c/Dialect/SparseTensor.h - C API for SparseTensor ----*- C -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM
   4: // Exceptions.
   5: // See https://llvm.org/LICENSE.txt for license information.
   6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   7: //
   8: //===----------------------------------------------------------------------===//
   9: 
  10: #ifndef MLIR_C_DIALECT_SPARSETENSOR_H
```
- EN:
  - Line 1: standard LLVM file banner or section divider.
  - Lines 2-7: comments documenting the surrounding code: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions. See https://llvm.or...`.
  - Line 8: standard LLVM file banner or section divider.
  - Line 9: blank separation between logical blocks.
  - Line 10: start of include guard `MLIR_C_DIALECT_SPARSETENSOR_H`.
- CN:
  - 第1行：LLVM 标准文件横幅或分节注释。
  - 第2-7行：通过注释说明周围代码：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions. See https://llvm.or...`。
  - 第8行：LLVM 标准文件横幅或分节注释。
  - 第9行：用于分隔逻辑块的空行。
  - 第10行：头文件保护宏 `MLIR_C_DIALECT_SPARSETENSOR_H` 的开始。

### Lines 11-20
```cpp
  11: #define MLIR_C_DIALECT_SPARSETENSOR_H
  12: 
  13: #include "mlir-c/AffineMap.h"
  14: #include "mlir-c/IR.h"
  15: 
  16: #ifdef __cplusplus
  17: extern "C" {
  18: #endif
  19: 
  20: MLIR_DECLARE_CAPI_DIALECT_REGISTRATION(SparseTensor, sparse_tensor);
```
- EN:
  - Line 11: definition of include-guard macro `MLIR_C_DIALECT_SPARSETENSOR_H`.
  - Line 12: blank separation between logical blocks.
  - Lines 13-14: direct C++ dependencies `mlir-c/AffineMap.h`, `mlir-c/IR.h`.
  - Line 15: blank separation between logical blocks.
  - Line 16: conditional preprocessor branch for `__cplusplus`.
  - Line 17: opening a new scope for the surrounding declaration or initializer.
  - Line 18: end of a conditional preprocessor region.
  - Line 19: blank separation between logical blocks.
  - Line 20: macro invocation `MLIR_DECLARE_CAPI_DIALECT_REGISTRATION` for declarative or generated behavior.
- CN:
  - 第11行：定义头文件保护宏 `MLIR_C_DIALECT_SPARSETENSOR_H`。
  - 第12行：用于分隔逻辑块的空行。
  - 第13-14行：直接包含的 C++ 依赖 `mlir-c/AffineMap.h`, `mlir-c/IR.h`。
  - 第15行：用于分隔逻辑块的空行。
  - 第16行：针对 `__cplusplus` 的条件预处理分支。
  - 第17行：为周围声明或初始化打开新的作用域。
  - 第18行：条件预处理区域的结束。
  - 第19行：用于分隔逻辑块的空行。
  - 第20行：调用宏 `MLIR_DECLARE_CAPI_DIALECT_REGISTRATION` 以附加声明式或生成式行为。

### Lines 21-30
```cpp
  21: 
  22: /// Dimension level types (and properties) that define sparse tensors.
  23: /// See the documentation in SparseTensorAttrDefs.td for their meaning.
  24: ///
  25: /// These correspond to SparseTensorEncodingAttr::LevelType in the C++ API.
  26: /// If updating, keep them in sync and update the static_assert in the impl
  27: /// file.
  28: typedef uint64_t MlirSparseTensorLevelType;
  29: 
  30: enum MlirSparseTensorLevelFormat {
```
- EN:
  - Line 21: blank separation between logical blocks.
  - Lines 22-27: comments documenting the surrounding code: `Dimension level types (and properties) that define sparse tensors. See the documentation in Spars...`.
  - Line 28: continuation of the surrounding declaration or initialization: `typedef uint64_t MlirSparseTensorLevelType;`.
  - Line 29: blank separation between logical blocks.
  - Line 30: beginning of enum `MlirSparseTensorLevelFormat`.
- CN:
  - 第21行：用于分隔逻辑块的空行。
  - 第22-27行：通过注释说明周围代码：`Dimension level types (and properties) that define sparse tensors. See the documentation in Spars...`。
  - 第28行：延续周围的声明或初始化：`typedef uint64_t MlirSparseTensorLevelType;`。
  - 第29行：用于分隔逻辑块的空行。
  - 第30行：枚举 `MlirSparseTensorLevelFormat` 的开始。

### Lines 31-40
```cpp
  31:   MLIR_SPARSE_TENSOR_LEVEL_DENSE = 0x000000010000,
  32:   MLIR_SPARSE_TENSOR_LEVEL_BATCH = 0x000000020000,
  33:   MLIR_SPARSE_TENSOR_LEVEL_COMPRESSED = 0x000000040000,
  34:   MLIR_SPARSE_TENSOR_LEVEL_SINGLETON = 0x000000080000,
  35:   MLIR_SPARSE_TENSOR_LEVEL_LOOSE_COMPRESSED = 0x000000100000,
  36:   MLIR_SPARSE_TENSOR_LEVEL_N_OUT_OF_M = 0x000000200000,
  37: };
  38: 
  39: enum MlirSparseTensorLevelPropertyNondefault {
  40:   MLIR_SPARSE_PROPERTY_NON_UNIQUE = 0x0001,
```
- EN:
  - Line 31: enum member `MLIR_SPARSE_TENSOR_LEVEL_DENSE`.
  - Line 32: enum member `MLIR_SPARSE_TENSOR_LEVEL_BATCH`.
  - Line 33: enum member `MLIR_SPARSE_TENSOR_LEVEL_COMPRESSED`.
  - Line 34: enum member `MLIR_SPARSE_TENSOR_LEVEL_SINGLETON`.
  - Line 35: enum member `MLIR_SPARSE_TENSOR_LEVEL_LOOSE_COMPRESSED`.
  - Line 36: enum member `MLIR_SPARSE_TENSOR_LEVEL_N_OUT_OF_M`.
  - Line 37: closing the current scope or type definition.
  - Line 38: blank separation between logical blocks.
  - Line 39: beginning of enum `MlirSparseTensorLevelPropertyNondefault`.
  - Line 40: enum member `MLIR_SPARSE_PROPERTY_NON_UNIQUE`.
- CN:
  - 第31行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_DENSE`。
  - 第32行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_BATCH`。
  - 第33行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_COMPRESSED`。
  - 第34行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_SINGLETON`。
  - 第35行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_LOOSE_COMPRESSED`。
  - 第36行：枚举成员 `MLIR_SPARSE_TENSOR_LEVEL_N_OUT_OF_M`。
  - 第37行：关闭当前作用域或类型定义。
  - 第38行：用于分隔逻辑块的空行。
  - 第39行：枚举 `MlirSparseTensorLevelPropertyNondefault` 的开始。
  - 第40行：枚举成员 `MLIR_SPARSE_PROPERTY_NON_UNIQUE`。

### Lines 41-50
```cpp
  41:   MLIR_SPARSE_PROPERTY_NON_ORDERED = 0x0002,
  42:   MLIR_SPARSE_PROPERTY_SOA = 0x0004,
  43: };
  44: 
  45: //===----------------------------------------------------------------------===//
  46: // SparseTensorEncodingAttr
  47: //===----------------------------------------------------------------------===//
  48: 
  49: /// Checks whether the given attribute is a `sparse_tensor.encoding` attribute.
  50: MLIR_CAPI_EXPORTED bool
```
- EN:
  - Line 41: enum member `MLIR_SPARSE_PROPERTY_NON_ORDERED`.
  - Line 42: enum member `MLIR_SPARSE_PROPERTY_SOA`.
  - Line 43: closing the current scope or type definition.
  - Line 44: blank separation between logical blocks.
  - Line 45: standard LLVM file banner or section divider.
  - Line 46: comments documenting the surrounding code: `SparseTensorEncodingAttr`.
  - Line 47: standard LLVM file banner or section divider.
  - Line 48: blank separation between logical blocks.
  - Line 49: comments documenting the surrounding code: `Checks whether the given attribute is a `sparse_tensor.encoding` attribute.`.
  - Line 50: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED bool`.
- CN:
  - 第41行：枚举成员 `MLIR_SPARSE_PROPERTY_NON_ORDERED`。
  - 第42行：枚举成员 `MLIR_SPARSE_PROPERTY_SOA`。
  - 第43行：关闭当前作用域或类型定义。
  - 第44行：用于分隔逻辑块的空行。
  - 第45行：LLVM 标准文件横幅或分节注释。
  - 第46行：通过注释说明周围代码：`SparseTensorEncodingAttr`。
  - 第47行：LLVM 标准文件横幅或分节注释。
  - 第48行：用于分隔逻辑块的空行。
  - 第49行：通过注释说明周围代码：`Checks whether the given attribute is a `sparse_tensor.encoding` attribute.`。
  - 第50行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED bool`。

### Lines 51-60
```cpp
  51: mlirAttributeIsASparseTensorEncodingAttr(MlirAttribute attr);
  52: 
  53: /// Creates a `sparse_tensor.encoding` attribute with the given parameters.
  54: MLIR_CAPI_EXPORTED MlirAttribute mlirSparseTensorEncodingAttrGet(
  55:     MlirContext ctx, intptr_t lvlRank,
  56:     MlirSparseTensorLevelType const *lvlTypes, MlirAffineMap dimToLvl,
  57:     MlirAffineMap lvlTodim, int posWidth, int crdWidth,
  58:     MlirAttribute explicitVal, MlirAttribute implicitVal);
  59: 
  60: MLIR_CAPI_EXPORTED MlirStringRef mlirSparseTensorEncodingAttrGetName(void);
```
- EN:
  - Line 51: function or method declaration `mlirAttributeIsASparseTensorEncodingAttr`.
  - Line 52: blank separation between logical blocks.
  - Line 53: comments documenting the surrounding code: `Creates a `sparse_tensor.encoding` attribute with the given parameters.`.
  - Line 54: part of a multi-line declaration or signature: `MLIR_CAPI_EXPORTED MlirAttribute mlirSparseTensorEncodingAttrGet(`.
  - Line 55: continuation of the surrounding declaration or initialization: `MlirContext ctx, intptr_t lvlRank,`.
  - Line 56: continuation of the surrounding declaration or initialization: `MlirSparseTensorLevelType const *lvlTypes, MlirAffineMap dimToLvl,`.
  - Line 57: continuation of the surrounding declaration or initialization: `MlirAffineMap lvlTodim, int posWidth, int crdWidth,`.
  - Line 58: part of a multi-line declaration or signature: `MlirAttribute explicitVal, MlirAttribute implicitVal);`.
  - Line 59: blank separation between logical blocks.
  - Line 60: function or method declaration `mlirSparseTensorEncodingAttrGetName`.
- CN:
  - 第51行：函数或方法声明 `mlirAttributeIsASparseTensorEncodingAttr`。
  - 第52行：用于分隔逻辑块的空行。
  - 第53行：通过注释说明周围代码：`Creates a `sparse_tensor.encoding` attribute with the given parameters.`。
  - 第54行：多行声明或签名的一部分：`MLIR_CAPI_EXPORTED MlirAttribute mlirSparseTensorEncodingAttrGet(`。
  - 第55行：延续周围的声明或初始化：`MlirContext ctx, intptr_t lvlRank,`。
  - 第56行：延续周围的声明或初始化：`MlirSparseTensorLevelType const *lvlTypes, MlirAffineMap dimToLvl,`。
  - 第57行：延续周围的声明或初始化：`MlirAffineMap lvlTodim, int posWidth, int crdWidth,`。
  - 第58行：多行声明或签名的一部分：`MlirAttribute explicitVal, MlirAttribute implicitVal);`。
  - 第59行：用于分隔逻辑块的空行。
  - 第60行：函数或方法声明 `mlirSparseTensorEncodingAttrGetName`。

### Lines 61-70
```cpp
  61: 
  62: /// Returns the level-rank of the `sparse_tensor.encoding` attribute.
  63: MLIR_CAPI_EXPORTED intptr_t
  64: mlirSparseTensorEncodingGetLvlRank(MlirAttribute attr);
  65: 
  66: /// Returns a specified level-type of the `sparse_tensor.encoding` attribute.
  67: MLIR_CAPI_EXPORTED MlirSparseTensorLevelType
  68: mlirSparseTensorEncodingAttrGetLvlType(MlirAttribute attr, intptr_t lvl);
  69: 
  70: /// Returns a specified level-format of the `sparse_tensor.encoding` attribute.
```
- EN:
  - Line 61: blank separation between logical blocks.
  - Line 62: comments documenting the surrounding code: `Returns the level-rank of the `sparse_tensor.encoding` attribute.`.
  - Line 63: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED intptr_t`.
  - Line 64: function or method declaration `mlirSparseTensorEncodingGetLvlRank`.
  - Line 65: blank separation between logical blocks.
  - Line 66: comments documenting the surrounding code: `Returns a specified level-type of the `sparse_tensor.encoding` attribute.`.
  - Line 67: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirSparseTensorLevelType`.
  - Line 68: function or method declaration `mlirSparseTensorEncodingAttrGetLvlType`.
  - Line 69: blank separation between logical blocks.
  - Line 70: comments documenting the surrounding code: `Returns a specified level-format of the `sparse_tensor.encoding` attribute.`.
- CN:
  - 第61行：用于分隔逻辑块的空行。
  - 第62行：通过注释说明周围代码：`Returns the level-rank of the `sparse_tensor.encoding` attribute.`。
  - 第63行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED intptr_t`。
  - 第64行：函数或方法声明 `mlirSparseTensorEncodingGetLvlRank`。
  - 第65行：用于分隔逻辑块的空行。
  - 第66行：通过注释说明周围代码：`Returns a specified level-type of the `sparse_tensor.encoding` attribute.`。
  - 第67行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirSparseTensorLevelType`。
  - 第68行：函数或方法声明 `mlirSparseTensorEncodingAttrGetLvlType`。
  - 第69行：用于分隔逻辑块的空行。
  - 第70行：通过注释说明周围代码：`Returns a specified level-format of the `sparse_tensor.encoding` attribute.`。

### Lines 71-80
```cpp
  71: MLIR_CAPI_EXPORTED enum MlirSparseTensorLevelFormat
  72: mlirSparseTensorEncodingAttrGetLvlFmt(MlirAttribute attr, intptr_t lvl);
  73: 
  74: /// Returns the dimension-to-level mapping of the `sparse_tensor.encoding`
  75: /// attribute.
  76: MLIR_CAPI_EXPORTED MlirAffineMap
  77: mlirSparseTensorEncodingAttrGetDimToLvl(MlirAttribute attr);
  78: 
  79: /// Returns the level-to-dimension mapping of the `sparse_tensor.encoding`
  80: /// attribute.
```
- EN:
  - Line 71: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED enum MlirSparseTensorLevelFormat`.
  - Line 72: function or method declaration `mlirSparseTensorEncodingAttrGetLvlFmt`.
  - Line 73: blank separation between logical blocks.
  - Lines 74-75: comments documenting the surrounding code: `Returns the dimension-to-level mapping of the `sparse_tensor.encoding` attribute.`.
  - Line 76: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirAffineMap`.
  - Line 77: function or method declaration `mlirSparseTensorEncodingAttrGetDimToLvl`.
  - Line 78: blank separation between logical blocks.
  - Lines 79-80: comments documenting the surrounding code: `Returns the level-to-dimension mapping of the `sparse_tensor.encoding` attribute.`.
- CN:
  - 第71行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED enum MlirSparseTensorLevelFormat`。
  - 第72行：函数或方法声明 `mlirSparseTensorEncodingAttrGetLvlFmt`。
  - 第73行：用于分隔逻辑块的空行。
  - 第74-75行：通过注释说明周围代码：`Returns the dimension-to-level mapping of the `sparse_tensor.encoding` attribute.`。
  - 第76行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirAffineMap`。
  - 第77行：函数或方法声明 `mlirSparseTensorEncodingAttrGetDimToLvl`。
  - 第78行：用于分隔逻辑块的空行。
  - 第79-80行：通过注释说明周围代码：`Returns the level-to-dimension mapping of the `sparse_tensor.encoding` attribute.`。

### Lines 81-90
```cpp
  81: MLIR_CAPI_EXPORTED MlirAffineMap
  82: mlirSparseTensorEncodingAttrGetLvlToDim(MlirAttribute attr);
  83: 
  84: /// Returns the position bitwidth of the `sparse_tensor.encoding` attribute.
  85: MLIR_CAPI_EXPORTED int
  86: mlirSparseTensorEncodingAttrGetPosWidth(MlirAttribute attr);
  87: 
  88: /// Returns the coordinate bitwidth of the `sparse_tensor.encoding` attribute.
  89: MLIR_CAPI_EXPORTED int
  90: mlirSparseTensorEncodingAttrGetCrdWidth(MlirAttribute attr);
```
- EN:
  - Line 81: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirAffineMap`.
  - Line 82: function or method declaration `mlirSparseTensorEncodingAttrGetLvlToDim`.
  - Line 83: blank separation between logical blocks.
  - Line 84: comments documenting the surrounding code: `Returns the position bitwidth of the `sparse_tensor.encoding` attribute.`.
  - Line 85: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED int`.
  - Line 86: function or method declaration `mlirSparseTensorEncodingAttrGetPosWidth`.
  - Line 87: blank separation between logical blocks.
  - Line 88: comments documenting the surrounding code: `Returns the coordinate bitwidth of the `sparse_tensor.encoding` attribute.`.
  - Line 89: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED int`.
  - Line 90: function or method declaration `mlirSparseTensorEncodingAttrGetCrdWidth`.
- CN:
  - 第81行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirAffineMap`。
  - 第82行：函数或方法声明 `mlirSparseTensorEncodingAttrGetLvlToDim`。
  - 第83行：用于分隔逻辑块的空行。
  - 第84行：通过注释说明周围代码：`Returns the position bitwidth of the `sparse_tensor.encoding` attribute.`。
  - 第85行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED int`。
  - 第86行：函数或方法声明 `mlirSparseTensorEncodingAttrGetPosWidth`。
  - 第87行：用于分隔逻辑块的空行。
  - 第88行：通过注释说明周围代码：`Returns the coordinate bitwidth of the `sparse_tensor.encoding` attribute.`。
  - 第89行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED int`。
  - 第90行：函数或方法声明 `mlirSparseTensorEncodingAttrGetCrdWidth`。

### Lines 91-100
```cpp
  91: 
  92: /// Returns the explicit value of the `sparse_tensor.encoding` attribute.
  93: MLIR_CAPI_EXPORTED MlirAttribute
  94: mlirSparseTensorEncodingAttrGetExplicitVal(MlirAttribute attr);
  95: 
  96: /// Returns the implicit value of the `sparse_tensor.encoding` attribute.
  97: MLIR_CAPI_EXPORTED MlirAttribute
  98: mlirSparseTensorEncodingAttrGetImplicitVal(MlirAttribute attr);
  99: 
 100: MLIR_CAPI_EXPORTED unsigned
```
- EN:
  - Line 91: blank separation between logical blocks.
  - Line 92: comments documenting the surrounding code: `Returns the explicit value of the `sparse_tensor.encoding` attribute.`.
  - Line 93: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirAttribute`.
  - Line 94: function or method declaration `mlirSparseTensorEncodingAttrGetExplicitVal`.
  - Line 95: blank separation between logical blocks.
  - Line 96: comments documenting the surrounding code: `Returns the implicit value of the `sparse_tensor.encoding` attribute.`.
  - Line 97: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirAttribute`.
  - Line 98: function or method declaration `mlirSparseTensorEncodingAttrGetImplicitVal`.
  - Line 99: blank separation between logical blocks.
  - Line 100: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED unsigned`.
- CN:
  - 第91行：用于分隔逻辑块的空行。
  - 第92行：通过注释说明周围代码：`Returns the explicit value of the `sparse_tensor.encoding` attribute.`。
  - 第93行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirAttribute`。
  - 第94行：函数或方法声明 `mlirSparseTensorEncodingAttrGetExplicitVal`。
  - 第95行：用于分隔逻辑块的空行。
  - 第96行：通过注释说明周围代码：`Returns the implicit value of the `sparse_tensor.encoding` attribute.`。
  - 第97行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirAttribute`。
  - 第98行：函数或方法声明 `mlirSparseTensorEncodingAttrGetImplicitVal`。
  - 第99行：用于分隔逻辑块的空行。
  - 第100行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED unsigned`。

### Lines 101-110
```cpp
 101: mlirSparseTensorEncodingAttrGetStructuredN(MlirSparseTensorLevelType lvlType);
 102: 
 103: MLIR_CAPI_EXPORTED unsigned
 104: mlirSparseTensorEncodingAttrGetStructuredM(MlirSparseTensorLevelType lvlType);
 105: 
 106: MLIR_CAPI_EXPORTED MlirSparseTensorLevelType
 107: mlirSparseTensorEncodingAttrBuildLvlType(
 108:     enum MlirSparseTensorLevelFormat lvlFmt,
 109:     const enum MlirSparseTensorLevelPropertyNondefault *properties,
 110:     unsigned propSize, unsigned n, unsigned m);
```
- EN:
  - Line 101: function or method declaration `mlirSparseTensorEncodingAttrGetStructuredN`.
  - Line 102: blank separation between logical blocks.
  - Line 103: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED unsigned`.
  - Line 104: function or method declaration `mlirSparseTensorEncodingAttrGetStructuredM`.
  - Line 105: blank separation between logical blocks.
  - Line 106: continuation of the surrounding declaration or initialization: `MLIR_CAPI_EXPORTED MlirSparseTensorLevelType`.
  - Line 107: part of a multi-line declaration or signature: `mlirSparseTensorEncodingAttrBuildLvlType(`.
  - Line 108: beginning of enum `MlirSparseTensorLevelFormat`.
  - Line 109: continuation of the surrounding declaration or initialization: `const enum MlirSparseTensorLevelPropertyNondefault *properties,`.
  - Line 110: part of a multi-line declaration or signature: `unsigned propSize, unsigned n, unsigned m);`.
- CN:
  - 第101行：函数或方法声明 `mlirSparseTensorEncodingAttrGetStructuredN`。
  - 第102行：用于分隔逻辑块的空行。
  - 第103行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED unsigned`。
  - 第104行：函数或方法声明 `mlirSparseTensorEncodingAttrGetStructuredM`。
  - 第105行：用于分隔逻辑块的空行。
  - 第106行：延续周围的声明或初始化：`MLIR_CAPI_EXPORTED MlirSparseTensorLevelType`。
  - 第107行：多行声明或签名的一部分：`mlirSparseTensorEncodingAttrBuildLvlType(`。
  - 第108行：枚举 `MlirSparseTensorLevelFormat` 的开始。
  - 第109行：延续周围的声明或初始化：`const enum MlirSparseTensorLevelPropertyNondefault *properties,`。
  - 第110行：多行声明或签名的一部分：`unsigned propSize, unsigned n, unsigned m);`。

### Lines 111-118
```cpp
 111: 
 112: #ifdef __cplusplus
 113: }
 114: #endif
 115: 
 116: #include "mlir/Dialect/SparseTensor/Transforms/Passes.capi.h.inc"
 117: 
 118: #endif // MLIR_C_DIALECT_SPARSETENSOR_H
```
- EN:
  - Line 111: blank separation between logical blocks.
  - Line 112: conditional preprocessor branch for `__cplusplus`.
  - Line 113: closing the current scope or type definition.
  - Line 114: end of a conditional preprocessor region.
  - Line 115: blank separation between logical blocks.
  - Line 116: direct C++ dependencies `mlir/Dialect/SparseTensor/Transforms/Passes.capi.h.inc`.
  - Line 117: blank separation between logical blocks.
  - Line 118: end of the file-level include guard.
- CN:
  - 第111行：用于分隔逻辑块的空行。
  - 第112行：针对 `__cplusplus` 的条件预处理分支。
  - 第113行：关闭当前作用域或类型定义。
  - 第114行：条件预处理区域的结束。
  - 第115行：用于分隔逻辑块的空行。
  - 第116行：直接包含的 C++ 依赖 `mlir/Dialect/SparseTensor/Transforms/Passes.capi.h.inc`。
  - 第117行：用于分隔逻辑块的空行。
  - 第118行：文件级头文件保护的结束。

## Key Concepts / 关键概念
- `MlirSparseTensorLevelFormat` — Enum / 枚举.
- `MlirSparseTensorLevelPropertyNondefault` — Enum / 枚举.
- `mlirAttributeIsASparseTensorEncodingAttr` — Function / 函数.
- `mlirSparseTensorEncodingAttrGet` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetName` — Function / 函数.
- `mlirSparseTensorEncodingGetLvlRank` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetLvlType` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetLvlFmt` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetDimToLvl` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetLvlToDim` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetPosWidth` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetCrdWidth` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetExplicitVal` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetImplicitVal` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetStructuredN` — Function / 函数.
- `mlirSparseTensorEncodingAttrGetStructuredM` — Function / 函数.
- `mlirSparseTensorEncodingAttrBuildLvlType` — Function / 函数.

## Dependencies / 依赖关系
- Direct includes / 直接包含:
  - `mlir-c/AffineMap.h`
  - `mlir-c/IR.h`
  - `mlir/Dialect/SparseTensor/Transforms/Passes.capi.h.inc`
- Primary symbols / 主要符号:
  - `MlirSparseTensorLevelFormat`
  - `MlirSparseTensorLevelPropertyNondefault`
  - `mlirAttributeIsASparseTensorEncodingAttr`
  - `mlirSparseTensorEncodingAttrGet`
  - `mlirSparseTensorEncodingAttrGetName`
  - `mlirSparseTensorEncodingGetLvlRank`
  - `mlirSparseTensorEncodingAttrGetLvlType`
  - `mlirSparseTensorEncodingAttrGetLvlFmt`
- Subsystem / 子系统: `mlir/include/mlir-c/Dialect`
- Notes / 说明: Dependencies were inferred from textual includes, namespace scopes, and major declarations. / 依赖关系基于文本中的 include、命名空间作用域与主要声明推断。
