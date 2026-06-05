# TritonGPUEnums.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/IR/TritonGPUEnums.td`
- **EN:** Defines TableGen enumerations shared by this subsystem.
- **CN:** 定义该子系统共享使用的 TableGen 枚举。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITONGPU_ENUMS
   2: #define TRITONGPU_ENUMS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-5
```tablegen
   4: include "mlir/IR/EnumAttr.td"
   5: include "triton/Dialect/TritonGPU/IR/TritonGPUDialect.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/EnumAttr.td and triton/Dialect/TritonGPU/IR/TritonGPUDialect.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/EnumAttr.td and triton/Dialect/TritonGPU/IR/TritonGPUDialect.td。

### Lines 7-20
```tablegen
   7: // Bitmask enum describing which memory domains a barrier/fence orders.
   8: def TTG_AddrSpace : I32BitEnumAttr<
   9:     "AddrSpace", "",
  10:     [
  11:       I32BitEnumAttrCase<"None", 0b0000, "none">,
  12:       I32BitEnumAttrCase<"Local", 0b0001, "local">,
  13:       I32BitEnumAttrCase<"GlobalRead", 0b0010, "global_read">,
  14:       I32BitEnumAttrCase<"GlobalWrite", 0b0100, "global_write">,
  15:       I32BitEnumAttrCase<"TensorRead", 0b1000, "tensor_read">,
  16:       I32BitEnumAttrCase<"TensorWrite", 0b10000, "tensor_write">,
  17:       I32BitEnumAttrCase<"All", 0b11111, "all">
  18:     ]> {
  19:   let cppNamespace = "::mlir::triton::gpu";
  20: }
```
**EN:** This TableGen def record defines `TTG_AddrSpace`. It is specialized from `I32BitEnumAttr<`.
**CN:** 该 TableGen def 记录定义了 `TTG_AddrSpace`。 它基于 `I32BitEnumAttr<` 进一步特化。

### Lines 22-22
```tablegen
  22: #endif // TRITONGPU_ENUMS
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** barrier semantics  
  **CN:** 屏障语义
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/EnumAttr.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUDialect.td`
