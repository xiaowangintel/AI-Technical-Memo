# PartitionBuilder.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/PartitionBuilder.h`
- **EN:** Declares transformation support utilities centered on `PartitionBuilder`.
- **CN:** 声明围绕 `PartitionBuilder` 的变换支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TRITONGPU_TRANSFORMS_PARTITIONBUILDER_H
   2: #define TRITON_TRITONGPU_TRANSFORMS_PARTITIONBUILDER_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-5
```cpp
   4: #include "mlir/IR/ImplicitLocOpBuilder.h"
   5: #include "llvm/ADT/SetVector.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/ImplicitLocOpBuilder.h and llvm/ADT/SetVector.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/ImplicitLocOpBuilder.h and llvm/ADT/SetVector.h。

### Lines 7-7
```cpp
   7: namespace mlir::triton::gpu {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::gpu.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::gpu 下。

### Lines 9-9
```cpp
   9: class Partition;
```
**EN:** This block introduces `Partition`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `Partition`。

### Lines 11-11
```cpp
  11: using StageCluster = std::optional<std::pair<int, int>>;
```
**EN:** This block introduces type aliases such as using, StageCluster, std::optional<std::pair<int, and int to simplify later declarations.
**CN:** 该代码块引入了 using, StageCluster, std::optional<std::pair<int, and int 等类型别名，以简化后续声明。

### Lines 13-15
```cpp
  13: // Get the stage and cluster for an operation, if it has one assigned.
  14: void setStageCluster(OpBuilder &b, Operation *op, StageCluster stageCluster);
  15: StageCluster getStageCluster(Operation *op);
```
**EN:** This block declares or defines callable APIs such as setStageCluster and getStageCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setStageCluster and getStageCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 17-18
```cpp
  17: struct PartitionBuilder : public ImplicitLocOpBuilder {
  18:   using ImplicitLocOpBuilder::ImplicitLocOpBuilder;
```
**EN:** This block introduces `PartitionBuilder`, the main class/struct defined here. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `PartitionBuilder`。 它还通过继承复用基类能力。

### Lines 20-21
```cpp
  20:   Value intCst(int value, unsigned width = 32);
  21:   Value boolCst(bool value);
```
**EN:** This block declares or defines callable APIs such as intCst and boolCst, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 intCst and boolCst 等可调用 API，用来封装这里提供的核心行为。

### Lines 23-23
```cpp
  23:   void assignPartition(Operation *op, Partition &partition);
```
**EN:** This block declares or defines callable APIs such as assignPartition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 assignPartition 等可调用 API，用来封装这里提供的核心行为。

### Lines 25-33
```cpp
  25:   template <typename OpT, typename... Args>
  26:   auto createInto(Partition &partition, StageCluster stageCluster,
  27:                   Args &&...args) {
  28:     auto op = create<OpT>(std::forward<Args>(args)...);
  29:     assignPartition(op, partition);
  30:     setStageCluster(*this, op, stageCluster);
  31:     return op;
  32:   }
  33: };
```
**EN:** This block declares or defines callable APIs such as createInto, assignPartition, and setStageCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createInto, assignPartition, and setStageCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 35-45
```cpp
  35: template <typename OpT, typename... Args>
  36: OpT createInto(OpBuilder &b, Location loc,
  37:                std::optional<SetVector<int>> partitionSet,
  38:                StageCluster stageCluster, Args &&...args) {
  39:   auto op = OpT::create(b, loc, std::forward<Args>(args)...);
  40:   if (partitionSet) {
  41:     setPartition(op, *partitionSet);
  42:     setStageCluster(b, op, stageCluster);
  43:   }
  44:   return op;
  45: }
```
**EN:** This block declares or defines callable APIs such as createInto, create, setPartition, and setStageCluster, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createInto, create, setPartition, and setStageCluster 等可调用 API，用来封装这里提供的核心行为。

### Lines 47-47
```cpp
  47: } // namespace mlir::triton::gpu
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 49-49
```cpp
  49: #endif // TRITON_TRITONGPU_TRANSFORMS_PARTITIONBUILDER_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** partitioning  
  **CN:** 分区
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/ImplicitLocOpBuilder.h`
  - `llvm/ADT/SetVector.h`
