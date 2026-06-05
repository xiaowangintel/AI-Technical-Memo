# MemorySlot.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SCF/IR/MemorySlot.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements SCF dialect support for dialect IR definitions such as operations, attributes, types, and parsers/printers, centered on `MemorySlot`.
  - **CN**: 实现 SCF 方言中围绕 `MemorySlot` 的方言 IR 定义，如操作、属性、类型以及解析/打印逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MemorySlot.cpp - Memory Slot interface implementations for SCF -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp

#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Interfaces/Utils/MemorySlotUtils.h"

using namespace mlir;
using namespace mlir::scf;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Interfaces/Utils/MemorySlotUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Interfaces/Utils/MemorySlotUtils.h`。

### Lines 15-23
```cpp
//===----------------------------------------------------------------------===//
// ExecuteRegionOp
//===----------------------------------------------------------------------===//

bool ExecuteRegionOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                         bool hasValueStores) {
  return true;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 24-36
```cpp
void ExecuteRegionOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  regionsToProcess.insert({&getRegion(), reachingDef});
}

Value ExecuteRegionOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  if (!hasValueStores)
    return reachingDef;

```
- **EN**: Implements logic around `setupPromotion`, `insert`, `finalizePromotion`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `insert`, `finalizePromotion` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 37-45
```cpp
  // Update the yield terminators to return the newly defined reaching
  // definition.
  for (Block &block : getRegion().getBlocks())
    if (isa<YieldOp>(block.getTerminator()))
      memoryslot::updateTerminator(&block, reachingDef, reachingAtBlockEnd);

  SmallVector<Type> resultTypes(getResultTypes());
  resultTypes.push_back(slot.elemType);

```
- **EN**: Implements logic around `getRegion`, `getTerminator`, `updateTerminator`, `resultTypes`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getRegion`, `getTerminator`, `updateTerminator`, `resultTypes`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 46-52
```cpp
  IRRewriter rewriter(builder);
  Operation *newOp =
      memoryslot::replaceWithNewResults(rewriter, getOperation(), resultTypes);
  return newOp->getResults().back();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `rewriter`, `replaceWithNewResults`, `getResults`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `rewriter`, `replaceWithNewResults`, `getResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 53-60
```cpp
// ForOp
//===----------------------------------------------------------------------===//

bool ForOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                               bool hasValueStores) {
  return true;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 61-69
```cpp
void ForOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  Region &bodyRegion = getBodyRegion();
  if (!hasValueStores) {
    regionsToProcess.insert({&bodyRegion, reachingDef});
    return;
  }

```
- **EN**: Implements logic around `setupPromotion`, `getBodyRegion`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `getBodyRegion`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 70-81
```cpp
  getInitArgsMutable().append(reachingDef);
  bodyRegion.addArgument(slot.elemType, slot.ptr.getLoc());
  regionsToProcess.insert({&bodyRegion, bodyRegion.getArguments().back()});
}

Value ForOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  if (!hasValueStores)
    return reachingDef;

```
- **EN**: Implements logic around `getInitArgsMutable`, `addArgument`, `insert`, `finalizePromotion`.
- **CN**: 围绕 `getInitArgsMutable`, `addArgument`, `insert`, `finalizePromotion` 实现具体逻辑。

### Lines 82-88
```cpp
  // Update the yield terminator to return the newly defined reaching
  // definition.
  memoryslot::updateTerminator(getBody(), reachingDef, reachingAtBlockEnd);

  SmallVector<Type> resultTypes(getResultTypes());
  resultTypes.push_back(slot.elemType);

```
- **EN**: Implements logic around `updateTerminator`, `resultTypes`, `push_back`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `updateTerminator`, `resultTypes`, `push_back` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 89-95
```cpp
  IRRewriter rewriter(builder);
  Operation *newOp =
      memoryslot::replaceWithNewResults(rewriter, getOperation(), resultTypes);
  return newOp->getResults().back();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `rewriter`, `replaceWithNewResults`, `getResults`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `rewriter`, `replaceWithNewResults`, `getResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 96-105
```cpp
// ForallOp
//===----------------------------------------------------------------------===//

bool ForallOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                  bool hasValueStores) {
  // The ForallOp body can be ran in parallel, thus does not support sequenced
  // value passing. Therefore only loads can be handled.
  return !hasValueStores;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 106-112
```cpp
void ForallOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  assert(!hasValueStores && "ForallOp does not support stores");
  regionsToProcess.insert({&getBodyRegion(), reachingDef});
}

```
- **EN**: Implements logic around `setupPromotion`, `assert`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `assert`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 113-120
```cpp
Value ForallOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  assert(!hasValueStores && "ForallOp does not support stores");
  return reachingDef;
}

```
- **EN**: Implements logic around `finalizePromotion`, `assert`.
- **CN**: 围绕 `finalizePromotion`, `assert` 实现具体逻辑。

### Lines 121-129
```cpp
//===----------------------------------------------------------------------===//
// IfOp
//===----------------------------------------------------------------------===//

bool IfOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                              bool hasValueStores) {
  return true;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 130-136
```cpp
void IfOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  regionsToProcess.insert({&getThenRegion(), reachingDef});
  regionsToProcess.insert({&getElseRegion(), reachingDef});
}

```
- **EN**: Implements logic around `setupPromotion`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 137-143
```cpp
Value IfOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  if (!hasValueStores)
    return reachingDef;

```
- **EN**: Implements logic around `finalizePromotion`.
- **CN**: 围绕 `finalizePromotion` 实现具体逻辑。

### Lines 144-157
```cpp
  IRRewriter rewriter(builder);

  // Update the yield terminators to return the newly defined reaching
  // definition.
  memoryslot::updateTerminator(&getThenRegion().back(), reachingDef,
                               reachingAtBlockEnd);
  if (getElseRegion().hasOneBlock()) {
    memoryslot::updateTerminator(&getElseRegion().back(), reachingDef,
                                 reachingAtBlockEnd);
  } else {
    OpBuilder::InsertionGuard guard(rewriter);
    rewriter.createBlock(&getElseRegion());
    YieldOp::create(rewriter, getOperation()->getLoc(), reachingDef);
  }
```
- **EN**: Implements logic around `rewriter`, `updateTerminator`, `getElseRegion`, `guard`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `rewriter`, `updateTerminator`, `getElseRegion`, `guard`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 158-166
```cpp

  SmallVector<Type> resultTypes(getResultTypes());
  resultTypes.push_back(slot.elemType);

  Operation *newOp =
      memoryslot::replaceWithNewResults(rewriter, getOperation(), resultTypes);
  return newOp->getResults().back();
}

```
- **EN**: Implements logic around `resultTypes`, `push_back`, `replaceWithNewResults`, `getResults`; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `resultTypes`, `push_back`, `replaceWithNewResults`, `getResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性。

### Lines 167-175
```cpp
//===----------------------------------------------------------------------===//
// IndexSwitchOp
//===----------------------------------------------------------------------===//

bool IndexSwitchOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                       bool hasValueStores) {
  return true;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 176-183
```cpp
void IndexSwitchOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  regionsToProcess.insert({&getDefaultRegion(), reachingDef});
  for (Region &caseRegion : getCaseRegions())
    regionsToProcess.insert({&caseRegion, reachingDef});
}

```
- **EN**: Implements logic around `setupPromotion`, `insert`, `getCaseRegions`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `insert`, `getCaseRegions` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 184-190
```cpp
Value IndexSwitchOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  if (!hasValueStores)
    return reachingDef;

```
- **EN**: Implements logic around `finalizePromotion`.
- **CN**: 围绕 `finalizePromotion` 实现具体逻辑。

### Lines 191-200
```cpp
  IRRewriter rewriter(builder);

  // Update the yield terminators to return the newly defined reaching
  // definition.
  memoryslot::updateTerminator(&getDefaultRegion().back(), reachingDef,
                               reachingAtBlockEnd);
  for (Region &caseRegion : getCaseRegions())
    memoryslot::updateTerminator(&caseRegion.back(), reachingDef,
                                 reachingAtBlockEnd);

```
- **EN**: Implements logic around `rewriter`, `updateTerminator`, `getCaseRegions`; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `rewriter`, `updateTerminator`, `getCaseRegions` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性。

### Lines 201-208
```cpp
  SmallVector<Type> resultTypes(getResultTypes());
  resultTypes.push_back(slot.elemType);

  Operation *newOp =
      memoryslot::replaceWithNewResults(rewriter, getOperation(), resultTypes);
  return newOp->getResults().back();
}

```
- **EN**: Implements logic around `resultTypes`, `push_back`, `replaceWithNewResults`, `getResults`; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `resultTypes`, `push_back`, `replaceWithNewResults`, `getResults` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性。

### Lines 209-219
```cpp
//===----------------------------------------------------------------------===//
// ParallelOp
//===----------------------------------------------------------------------===//

bool ParallelOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                    bool hasValueStores) {
  // The ParallelOp body can be ran in parallel, thus does not support sequenced
  // value passing. Therefore only loads can be handled.
  return !hasValueStores;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 220-226
```cpp
void ParallelOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  assert(!hasValueStores && "ParallelOp does not support stores");
  regionsToProcess.insert({&getBodyRegion(), reachingDef});
}

```
- **EN**: Implements logic around `setupPromotion`, `assert`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `assert`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 227-234
```cpp
Value ParallelOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  assert(!hasValueStores && "ParallelOp does not support stores");
  return reachingDef;
}

```
- **EN**: Implements logic around `finalizePromotion`, `assert`.
- **CN**: 围绕 `finalizePromotion`, `assert` 实现具体逻辑。

### Lines 235-245
```cpp
//===----------------------------------------------------------------------===//
// ReduceOp
//===----------------------------------------------------------------------===//

bool ReduceOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                  bool hasValueStores) {
  // The ReduceOp body can be ran in parallel, thus does not support sequenced
  // value passing. Therefore only loads can be handled.
  return !hasValueStores;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 246-253
```cpp
void ReduceOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  assert(!hasValueStores && "ReduceOp does not support stores");
  for (Region &reduction : getReductions())
    regionsToProcess.insert({&reduction, reachingDef});
}

```
- **EN**: Implements logic around `setupPromotion`, `assert`, `getReductions`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `assert`, `getReductions`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 254-261
```cpp
Value ReduceOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  assert(!hasValueStores && "ReduceOp does not support stores");
  return reachingDef;
}

```
- **EN**: Implements logic around `finalizePromotion`, `assert`.
- **CN**: 围绕 `finalizePromotion`, `assert` 实现具体逻辑。

### Lines 262-270
```cpp
//===----------------------------------------------------------------------===//
// WhileOp
//===----------------------------------------------------------------------===//

bool WhileOp::isRegionPromotable(const MemorySlot &slot, Region *region,
                                 bool hasValueStores) {
  return true;
}

```
- **EN**: Implements logic around `isRegionPromotable`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isRegionPromotable` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 271-281
```cpp
void WhileOp::setupPromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    llvm::SmallMapVector<Region *, Value, 2> &regionsToProcess) {
  Region &beforeRegion = getBefore();
  Region &afterRegion = getAfter();
  if (!hasValueStores) {
    regionsToProcess.insert({&beforeRegion, reachingDef});
    regionsToProcess.insert({&afterRegion, reachingDef});
    return;
  }

```
- **EN**: Implements logic around `setupPromotion`, `getBefore`, `getAfter`, `insert`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setupPromotion`, `getBefore`, `getAfter`, `insert` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 282-290
```cpp
  getInitsMutable().append(reachingDef);

  beforeRegion.addArgument(slot.elemType, slot.ptr.getLoc());
  regionsToProcess.insert({&beforeRegion, beforeRegion.getArguments().back()});

  afterRegion.addArgument(slot.elemType, slot.ptr.getLoc());
  regionsToProcess.insert({&afterRegion, afterRegion.getArguments().back()});
}

```
- **EN**: Implements logic around `getInitsMutable`, `addArgument`, `insert`.
- **CN**: 围绕 `getInitsMutable`, `addArgument`, `insert` 实现具体逻辑。

### Lines 291-297
```cpp
Value WhileOp::finalizePromotion(
    const MemorySlot &slot, Value reachingDef, bool hasValueStores,
    const llvm::DenseMap<Block *, Value> &reachingAtBlockEnd,
    OpBuilder &builder) {
  if (!hasValueStores)
    return reachingDef;

```
- **EN**: Implements logic around `finalizePromotion`.
- **CN**: 围绕 `finalizePromotion` 实现具体逻辑。

### Lines 298-305
```cpp
  // Update the yield terminators to return the newly defined reaching
  // definition.
  memoryslot::updateTerminator(&getBefore().back(),
                               getBefore().getArguments().back(),
                               reachingAtBlockEnd);
  memoryslot::updateTerminator(
      &getAfter().back(), getAfter().getArguments().back(), reachingAtBlockEnd);

```
- **EN**: Implements logic around `updateTerminator`, `getBefore`, `getAfter`.
- **CN**: 围绕 `updateTerminator`, `getBefore`, `getAfter` 实现具体逻辑。

### Lines 306-313
```cpp
  SmallVector<Type> resultTypes(getResultTypes());
  resultTypes.push_back(slot.elemType);

  IRRewriter rewriter(builder);
  Operation *newOp =
      memoryslot::replaceWithNewResults(rewriter, getOperation(), resultTypes);
  return newOp->getResults().back();
}
```
- **EN**: Implements logic around `resultTypes`, `push_back`, `rewriter`, `replaceWithNewResults`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `resultTypes`, `push_back`, `rewriter`, `replaceWithNewResults`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Interfaces/Utils/MemorySlotUtils.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (1), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
