# Partition.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonGPU/Transforms/Partition.h`
- **EN:** Declares transformation support utilities centered on `Partition`.
- **CN:** 声明围绕 `Partition` 的变换支持工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TRITONGPU_TRANSFORM_PIPELINE_PARTITION_H_
   2: #define TRITON_TRITONGPU_TRANSFORM_PIPELINE_PARTITION_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-7
```cpp
   4: #include "mlir/Support/LLVM.h"
   5: #include "llvm/ADT/ArrayRef.h"
   6: #include "llvm/ADT/DenseMap.h"
   7: #include "llvm/ADT/SmallVector.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Support/LLVM.h, llvm/ADT/ArrayRef.h, llvm/ADT/DenseMap.h, and llvm/ADT/SmallVector.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Support/LLVM.h, llvm/ADT/ArrayRef.h, llvm/ADT/DenseMap.h, and llvm/ADT/SmallVector.h。

### Lines 9-17
```cpp
   9: namespace mlir {
  10: class Operation;
  11: class OpOperand;
  12: class OpResult;
  13: class Region;
  14: namespace scf {
  15: class ForOp;
  16: } // namespace scf
  17: } // namespace mlir
```
**EN:** This block stores supporting state such as Operation, OpOperand, OpResult, Region, and ForOp, which other APIs in the file consume.
**CN:** 该代码块声明了 Operation, OpOperand, OpResult, Region, and ForOp 等支撑状态，供本文件中的其他 API 使用。

### Lines 19-21
```cpp
  19: //===----------------------------------------------------------------------===//
  20: // PartitionSet
  21: //===----------------------------------------------------------------------===//
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===----------------------------------------------------------------------===// PartitionSet ===----------------------------------------------------------------------===//.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 23-31
```cpp
  23: namespace mlir::triton::gpu {
  24: // A partition has a stage and contains some operation. The stage of a
  25: // partition determines how many cycles the partition's outputs are buffered
  26: // relative to its consumers.
  27: class Partition {
  28: public:
  29:   Partition(int idx, int stage) : idx(idx), stage(stage) {
  30:     assert(idx >= 0 && "A partition index must be nonnegative.");
  31:   }
```
**EN:** This block declares or defines callable APIs such as Partition, idx, and stage, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Partition, idx, and stage 等可调用 API，用来封装这里提供的核心行为。

### Lines 33-38
```cpp
  33:   int getIndex() const { return idx; }
  34:   int getStage() const { return stage; }
  35:   ArrayRef<Operation *> getOps() const { return ops; }
  36:   void addOp(Operation *op) { ops.push_back(op); }
  37:   bool hasOp(Operation *op) const;
  38:   bool empty() const { return ops.empty(); }
```
**EN:** This block declares or defines callable APIs such as getIndex, getStage, getOps, addOp, push_back, hasOp, and empty, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getIndex, getStage, getOps, addOp, push_back, hasOp, and empty 等可调用 API，用来封装这里提供的核心行为。

### Lines 40-61
```cpp
  40:   // Iterate the inputs of the partition. Input values are those that originate
  41:   // from a different partition or a previous iteration of the current
  42:   // partition. E.g. partition B(i) may have inputs from A(i) or B(i-1). Note
  43:   // that the same value may be visited more than once.
  44:   void iterateInputs(scf::ForOp loop,
  45:                      function_ref<void(OpOperand &)> callback) const;
  46:   // Iterate the outputs of the partition. Output values are those that are
  47:   // consumed by a different partition or a future iteration of the current
  48:   // partition. E.g. partition A(i) may have outputs to B(i) or A(i+1). Note
  49:   // that the same value may be visited more than once.
  50:   void
  51:   iterateOutputs(scf::ForOp loop,
  52:                  function_ref<void(Operation *, OpOperand &)> callback) const;
  53:   // Iterate the defining ops of the inputs to the partition in the current and
  54:   // previous iterations, including the distance in the past.
  55:   void iterateDefs(scf::ForOp loop,
  56:                    function_ref<void(OpResult, unsigned)> callback) const;
  57:   // Iterate the uses of all outputs of the partition in the current iteration
  58:   // and in future iterations, including the distance in the future.
  59:   void iterateUses(
  60:       scf::ForOp loop,
  61:       function_ref<void(OpResult, OpOperand &, unsigned)> callback) const;
```
**EN:** This block declares or defines callable APIs such as iterateInputs, void, iterateOutputs, iterateDefs, and iterateUses, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 iterateInputs, void, iterateOutputs, iterateDefs, and iterateUses 等可调用 API，用来封装这里提供的核心行为。

### Lines 63-64
```cpp
  63: private:
  64:   void setIndex(int idx) { this->idx = idx; }
```
**EN:** This block declares or defines callable APIs such as setIndex, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setIndex 等可调用 API，用来封装这里提供的核心行为。

### Lines 66-72
```cpp
  66:   // The partition number.
  67:   int idx;
  68:   // The stage of the partition.
  69:   int stage;
  70:   // The ops in the partition.
  71:   SmallVector<Operation *> ops;
  72: };
```
**EN:** This block stores supporting state such as idx, stage, and ops, which other APIs in the file consume.
**CN:** 该代码块声明了 idx, stage, and ops 等支撑状态，供本文件中的其他 API 使用。

### Lines 74-80
```cpp
  74: // A partition set divides a loop into multiple partitions. Ops in a loop are
  75: // assigned at most one partition. A partition set represents asynchronous
  76: // execution of the loop body, where partitions may execute simultaneously.
  77: class PartitionSet {
  78: public:
  79:   // Get WarpSpecialization tag
  80:   int getTag() const { return tag; }
```
**EN:** This block introduces `PartitionSet`, the main class/struct defined here. Within the declaration, methods such as getTag expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `PartitionSet`。 其中 getTag 等方法构成了它的主要接口。

### Lines 82-83
```cpp
  82:   // Create a new partition with a stage.
  83:   Partition *addPartition(unsigned stage);
```
**EN:** This block declares or defines callable APIs such as addPartition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addPartition 等可调用 API，用来封装这里提供的核心行为。

### Lines 85-94
```cpp
  85:   // Get the partition at the index.
  86:   Partition *getPartition(unsigned idx);
  87:   // Get the partition at the index.
  88:   const Partition *getPartition(unsigned idx) const;
  89:   // Return an iterator range over the partitions.
  90:   auto getPartitions() { return llvm::make_pointee_range(partitions); }
  91:   // Return an iterator range over the partitions.
  92:   auto getPartitions() const { return llvm::make_pointee_range(partitions); }
  93:   // Get the number of partitions.
  94:   unsigned getNumPartitions() const { return partitions.size(); }
```
**EN:** This block declares or defines callable APIs such as getPartition, getPartitions, make_pointee_range, getNumPartitions, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartition, getPartitions, make_pointee_range, getNumPartitions, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 96-98
```cpp
  96:   // Deserialize a partition set from an `scf.for` op using the attributes
  97:   // tagged on operations in its body.
  98:   static FailureOr<PartitionSet> fromLoop(scf::ForOp loop);
```
**EN:** This block declares or defines callable APIs such as fromLoop, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 fromLoop 等可调用 API，用来封装这里提供的核心行为。

### Lines 100-101
```cpp
 100:   // Debug dump the partition set.
 101:   LLVM_DUMP_METHOD void dump() const;
```
**EN:** This block declares or defines callable APIs such as dump, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dump 等可调用 API，用来封装这里提供的核心行为。

### Lines 103-104
```cpp
 103:   // Utility to be used when the op is known to belong to one partition
 104:   Partition *getPartition(Operation *op);
```
**EN:** This block declares or defines callable APIs such as getPartition, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartition 等可调用 API，用来封装这里提供的核心行为。

### Lines 106-111
```cpp
 106: private:
 107:   // WarpSpecialization tag
 108:   int tag;
 109:   // Partitions are numbered [0, N).
 110:   SmallVector<std::unique_ptr<Partition>> partitions;
 111: };
```
**EN:** This block stores supporting state such as tag and partitions, which other APIs in the file consume.
**CN:** 该代码块声明了 tag and partitions 等支撑状态，供本文件中的其他 API 使用。

### Lines 113-123
```cpp
 113: // Annotate the op with the partition index or indices, and add the op
 114: // to the partitions it belongs to.
 115: void setPartition(Operation *op, Partition *partition);
 116: void setPartition(Operation *op, const SetVector<Partition *> &partitions);
 117: // Annotate the op with the partition indices. It should only be used in a pass
 118: // which does not work with Partition instances and iterate* functions, since
 119: // it does not keep the op attributes and the op list of a partition in sync.
 120: void setPartition(Operation *op, const SetVector<int> &partitionIds);
 121: void setPartitionOutputs(Operation *op,
 122:                          ArrayRef<SetVector<int>> partitionOutputsIds);
 123: void setWarpSpecializeTag(Operation *op, int tag);
```
**EN:** This block declares or defines callable APIs such as setPartition, setPartitionOutputs, and setWarpSpecializeTag, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 setPartition, setPartitionOutputs, and setWarpSpecializeTag 等可调用 API，用来封装这里提供的核心行为。

### Lines 125-125
```cpp
 125: } // namespace mlir::triton::gpu
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 127-127
```cpp
 127: #endif // TRITON_TRITONGPU_TRANSFORM_PIPELINE_PARTITION_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** compiler passes  
  **CN:** 编译 pass
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** software pipelining  
  **CN:** 软件流水化
- **EN:** partitioning  
  **CN:** 分区

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Support/LLVM.h`
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallVector.h`
