# StackToShared.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenMP/Transforms/StackToShared.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements transforms to swap stack allocations on the target device with device shared memory where applicable.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenMP/Transforms`，围绕 OpenMP 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- StackToShared.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-12
```cpp
//
// This file implements transforms to swap stack allocations on the target
// device with device shared memory where applicable.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 13-21
```cpp

#include "mlir/Dialect/OpenMP/Transforms/Passes.h"

#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/Dialect/OpenMP/OpenMPDialect.h"
#include "mlir/Dialect/OpenMP/Utils/Utils.h"
#include "mlir/Pass/Pass.h"
#include "llvm/ADT/STLExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenMP/Transforms/Passes.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/Dialect/OpenMP/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenMP/Transforms/Passes.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/Dialect/OpenMP/Utils/Utils.h`。

### Lines 22-28
```cpp
namespace mlir {
namespace omp {
#define GEN_PASS_DEF_STACKTOSHAREDPASS
#include "mlir/Dialect/OpenMP/Transforms/Passes.h.inc"
} // namespace omp
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`。

### Lines 29-33
```cpp
using namespace mlir;

/// Tell whether to replace an operation representing a stack allocation with a
/// device shared memory allocation/deallocation pair based on the location of
/// the allocation and its uses.
```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 34-40
```cpp
static bool shouldReplaceAllocaWithDeviceSharedMem(Operation &op) {
  return omp::opInSharedDeviceContext(op) &&
         llvm::any_of(op.getResults(), [&](Value result) {
           return omp::allocaUsesRequireSharedMem(result);
         });
}

```
- **EN**: Implements logic around `shouldReplaceAllocaWithDeviceSharedMem`, `opInSharedDeviceContext`, `any_of`, `allocaUsesRequireSharedMem`.
- **CN**: 围绕 `shouldReplaceAllocaWithDeviceSharedMem`, `opInSharedDeviceContext`, `any_of`, `allocaUsesRequireSharedMem` 实现具体逻辑。

### Lines 41-50
```cpp
/// Based on the location of the definition of the given value representing the
/// result of a device shared memory allocation, find the corresponding points
/// where its deallocation should be placed and introduce `omp.free_shared_mem`
/// ops at those points.
static void insertDeviceSharedMemDeallocation(OpBuilder &builder,
                                              TypeAttr elemType,
                                              Value arraySize,
                                              IntegerAttr alignment,
                                              Value allocVal) {
  Block *allocaBlock = allocVal.getParentBlock();
```
- **EN**: Implements logic around `insertDeviceSharedMemDeallocation`, `getParentBlock`.
- **CN**: 围绕 `insertDeviceSharedMemDeallocation`, `getParentBlock` 实现具体逻辑。

### Lines 51-60
```cpp
  DominanceInfo domInfo;
  for (Block &block : allocVal.getParentRegion()->getBlocks()) {
    Operation *terminator = block.getTerminator();
    if (!terminator->hasSuccessors() &&
        domInfo.dominates(allocaBlock, &block)) {
      builder.setInsertionPoint(terminator);
      omp::FreeSharedMemOp::create(builder, allocVal.getLoc(), elemType,
                                   arraySize, alignment, allocVal);
    }
  }
```
- **EN**: Implements logic around `getParentRegion`, `getTerminator`, `hasSuccessors`, `dominates`, and 2 more symbols.
- **CN**: 围绕 `getParentRegion`, `getTerminator`, `hasSuccessors`, `dominates`, and 2 more symbols 实现具体逻辑。

### Lines 61-66
```cpp
}

namespace {
class StackToSharedPass
    : public omp::impl::StackToSharedPassBase<StackToSharedPass> {
public:
```
- **EN**: Introduces declarations for `StackToSharedPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `StackToSharedPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 67-72
```cpp
  StackToSharedPass() = default;

  void runOnOperation() override {
    MLIRContext *context = &getContext();
    OpBuilder builder(context);

```
- **EN**: Implements logic around `StackToSharedPass`, `runOnOperation`, `getContext`, `builder`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `StackToSharedPass`, `runOnOperation`, `getContext`, `builder` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 73-77
```cpp
    LLVM::LLVMFuncOp funcOp = getOperation();
    auto offloadIface = funcOp->getParentOfType<omp::OffloadModuleInterface>();
    if (!offloadIface || !offloadIface.getIsTargetDevice())
      return;

```
- **EN**: Implements logic around `getOperation`, `OffloadModuleInterface>`, `getIsTargetDevice`.
- **CN**: 围绕 `getOperation`, `OffloadModuleInterface>`, `getIsTargetDevice` 实现具体逻辑。

### Lines 78-84
```cpp
    llvm::SmallVector<Operation *> toBeDeleted;
    funcOp->walk([&](LLVM::AllocaOp allocaOp) {
      if (!shouldReplaceAllocaWithDeviceSharedMem(*allocaOp))
        return;
      // Replace llvm.alloca with omp.alloc_shared_mem.
      Type resultType = allocaOp.getResult().getType();

```
- **EN**: Implements logic around `walk`, `shouldReplaceAllocaWithDeviceSharedMem`, `getResult`.
- **CN**: 围绕 `walk`, `shouldReplaceAllocaWithDeviceSharedMem`, `getResult` 实现具体逻辑。

### Lines 85-92
```cpp
      // TODO: The handling of non-default address spaces might need to be
      // improved. This currently only handles the case where an alloca to
      // non-default address space is only used by a single addrspacecast to
      // default address space.
      bool nonDefaultAddrSpace = false;
      if (auto llvmPtrType = dyn_cast<LLVM::LLVMPointerType>(resultType))
        nonDefaultAddrSpace = llvmPtrType.getAddressSpace() != 0;

```
- **EN**: Implements logic around `LLVMPointerType>`, `getAddressSpace`.
- **CN**: 围绕 `LLVMPointerType>`, `getAddressSpace` 实现具体逻辑。

### Lines 93-102
```cpp
      builder.setInsertionPoint(allocaOp);
      auto sharedAllocOp = omp::AllocSharedMemOp::create(
          builder, allocaOp->getLoc(), LLVM::LLVMPointerType::get(context),
          allocaOp.getElemTypeAttr(), allocaOp.getArraySize(),
          allocaOp.getAlignmentAttr());
      if (nonDefaultAddrSpace) {
        assert(allocaOp->hasOneUse() && " unsupported non-default address "
                                        "space alloca with multiple uses");
        auto asCastOp =
            cast<LLVM::AddrSpaceCastOp>(*allocaOp->getUsers().begin());
```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `getLoc`, `getElemTypeAttr`, and 3 more symbols.
- **CN**: 围绕 `setInsertionPoint`, `create`, `getLoc`, `getElemTypeAttr`, and 3 more symbols 实现具体逻辑。

### Lines 103-112
```cpp
        asCastOp.replaceAllUsesWith(sharedAllocOp.getOperation());
        // Delete later because we can't delete the cast op before the top-level
        // iteration visits it. Also, the alloca can't be deleted before because
        // it's used by it.
        toBeDeleted.push_back(asCastOp);
        toBeDeleted.push_back(allocaOp);
      } else {
        allocaOp.replaceAllUsesWith(sharedAllocOp.getOperation());
        allocaOp.erase();
      }
```
- **EN**: Implements logic around `replaceAllUsesWith`, `push_back`, `erase`.
- **CN**: 围绕 `replaceAllUsesWith`, `push_back`, `erase` 实现具体逻辑。

### Lines 113-122
```cpp

      // Create a new omp.free_shared_mem for the allocated buffer prior to
      // exiting the region.
      insertDeviceSharedMemDeallocation(
          builder, sharedAllocOp.getMemElemTypeAttr(),
          sharedAllocOp.getMemArraySize(), sharedAllocOp.getMemAlignmentAttr(),
          sharedAllocOp.getResult());
    });
    for (Operation *op : toBeDeleted)
      op->erase();
```
- **EN**: Implements logic around `insertDeviceSharedMemDeallocation`, `getMemElemTypeAttr`, `getMemArraySize`, `getResult`, and 1 more symbols.
- **CN**: 围绕 `insertDeviceSharedMemDeallocation`, `getMemElemTypeAttr`, `getMemArraySize`, `getResult`, and 1 more symbols 实现具体逻辑。

### Lines 123-125
```cpp
  }
};
} // namespace
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenMP/Transforms/Passes.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/Dialect/OpenMP/Utils/Utils.h`, `mlir/Pass/Pass.h`, `llvm/ADT/STLExtras.h`, `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), pass infrastructure and registration support / Pass 基础设施与注册支持 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
