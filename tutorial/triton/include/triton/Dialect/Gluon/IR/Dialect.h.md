# Dialect.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/Gluon/IR/Dialect.h`
- **EN:** Declares the MLIR dialect interface and imports generated declarations for this subsystem.
- **CN:** 声明该子系统的 MLIR 方言接口，并引入生成的声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```cpp
   1: #pragma once
   2: #include "triton/Dialect/Triton/IR/Dialect.h"
   3: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 5-5
```cpp
   5: #include "triton/Dialect/Gluon/IR/Dialect.h.inc"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/Gluon/IR/Dialect.h.inc.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/Gluon/IR/Dialect.h.inc。

### Lines 7-8
```cpp
   7: #define GET_ATTRDEF_CLASSES
   8: #include "triton/Dialect/Gluon/IR/GluonAttrDefs.h.inc"
```
**EN:** This block enables MLIR/TableGen-generated declarations and then includes the generated `.inc` fragments that materialize them.
**CN:** 该代码块先打开 MLIR/TableGen 生成代码所需的宏开关，再引入真正展开声明的 `.inc` 片段。

### Lines 10-11
```cpp
  10: #define GET_OP_CLASSES
  11: #include "triton/Dialect/Gluon/IR/Ops.h.inc"
```
**EN:** This block enables MLIR/TableGen-generated declarations and then includes the generated `.inc` fragments that materialize them.
**CN:** 该代码块先打开 MLIR/TableGen 生成代码所需的宏开关，再引入真正展开声明的 `.inc` 片段。

## Key Concepts / 关键概念
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** Gluon dialect support  
  **CN:** Gluon 方言支持

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
- **Generated includes / 生成代码依赖:**
  - `triton/Dialect/Gluon/IR/Dialect.h.inc`
  - `triton/Dialect/Gluon/IR/GluonAttrDefs.h.inc`
  - `triton/Dialect/Gluon/IR/Ops.h.inc`
