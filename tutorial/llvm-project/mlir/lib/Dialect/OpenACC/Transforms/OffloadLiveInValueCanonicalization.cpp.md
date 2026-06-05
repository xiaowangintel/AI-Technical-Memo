# OffloadLiveInValueCanonicalization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/OffloadLiveInValueCanonicalization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass canonicalizes live-in values for regions destined for offloading.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- OffloadLiveInValueCanonicalization.cpp -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-21
```cpp
//
// This pass canonicalizes live-in values for regions destined for offloading.
//
// Overview:
// ---------
// When a region is outlined (extracted into a separate function for device
// execution), values defined outside the region but used inside become
// arguments to the outlined function. However, some values cannot be passed
// as arguments because they represent synthetic types (e.g., shape metadata,
// field indices) or are better handled by recreating them inside the region.
//
// This pass identifies such values and either:
// 1. Sinks the defining operation into the region (if all uses are inside)
// 2. Rematerializes (clones) the operation inside the region (if there are
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
//    uses both inside and outside)
//
// Transforms:
// -----------
// The pass performs two main transformations on live-in values:
//
// 1. Sinking: If a candidate operation's result is only used inside the
//    offload region, the operation is moved into the region.
//
// 2. Rematerialization: If a candidate operation's result is used both
//    inside and outside the region, the operation is cloned inside the
//    region and uses within the region are updated to use the clone.
//
// Candidate operations are:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-49
```cpp
// - Constants (matching arith.constant, etc.)
// - Operations implementing `acc::OutlineRematerializationOpInterface`
// - Address-of operations (`acc::AddressOfGlobalOpInterface`) referencing
//   symbols that are valid in GPU regions or constant globals
//
// The pass traces through view-like operations (`ViewLikeOpInterface`) and
// partial entity access operations (`acc::PartialEntityAccessOpInterface`)
// to find the original defining operation before making candidate decisions.
//
// Requirements:
// -------------
// To use this pass in a pipeline, the following requirements must be met:
//
// 1. Target Region Identification: Operations representing offload regions
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 50-63
```cpp
//    must implement `acc::OffloadRegionOpInterface`. This interface marks
//    regions that will be outlined for device execution.
//
// 2. Rematerialization Candidates: Operations producing values that should
//    be rematerialized (rather than passed as arguments) should implement
//    `acc::OutlineRematerializationOpInterface`. Examples include operations
//    producing shape metadata, field indices, or other synthetic types.
//
// 3. Analysis Registration (Optional): If custom behavior is needed for
//    symbol validation (e.g., determining if a global is valid on device),
//    pre-register `acc::OpenACCSupport` analysis on the parent module.
//    If not registered, default behavior will be used.
//
// 4. View-Like Operations: Operations that create views or casts should
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 64-70
```cpp
//    implement `ViewLikeOpInterface` or `acc::PartialEntityAccessOpInterface`
//    to allow the pass to trace through to the original defining operation.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`。

### Lines 71-84
```cpp
#include "mlir/Analysis/TopologicalSortUtils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/Region.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/IR/Value.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "mlir/Interfaces/ViewLikeInterface.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LLVM.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`。

### Lines 85-93
```cpp
#include "mlir/Transforms/RegionUtils.h"

namespace mlir {
namespace acc {
#define GEN_PASS_DEF_OFFLOADLIVEINVALUECANONICALIZATION
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Transforms/RegionUtils.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Transforms/RegionUtils.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 94-100
```cpp
#define DEBUG_TYPE "offload-livein-value-canonicalization"

using namespace mlir;

namespace {

/// Returns true if all users of the given value are inside the region.
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 101-107
```cpp
static bool allUsersAreInsideRegion(Value val, Region &region) {
  for (Operation *user : val.getUsers())
    if (!region.isAncestor(user->getParentRegion()))
      return false;
  return true;
}

```
- **EN**: Implements logic around `allUsersAreInsideRegion`, `getUsers`, `isAncestor`.
- **CN**: 围绕 `allUsersAreInsideRegion`, `getUsers`, `isAncestor` 实现具体逻辑。

### Lines 108-121
```cpp
/// Traces through view-like and partial entity access operations to find the
/// original defining value.
static Value getOriginalValue(Value val) {
  Value prev;
  while (val && val != prev) {
    prev = val;
    if (auto viewLikeOp = val.getDefiningOp<ViewLikeOpInterface>())
      val = viewLikeOp.getViewSource();
    if (auto partialAccess =
            val.getDefiningOp<acc::PartialEntityAccessOpInterface>()) {
      Value base = partialAccess.getBaseEntity();
      if (base)
        val = base;
    }
```
- **EN**: Implements logic around `getOriginalValue`, `getDefiningOp`, `getViewSource`, `PartialEntityAccessOpInterface>`, and 1 more symbols.
- **CN**: 围绕 `getOriginalValue`, `getDefiningOp`, `getViewSource`, `PartialEntityAccessOpInterface>`, and 1 more symbols 实现具体逻辑。

### Lines 122-128
```cpp
  }
  return val;
}

/// Returns true if the operation is a candidate for rematerialization.
/// Candidates are operations that:
/// 1. Match the constant pattern (arith.constant, etc.)
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 129-140
```cpp
/// 2. Implement OutlineRematerializationOpInterface
/// 3. Are address-of operations referencing valid symbols or constant globals
/// The function traces through view-like operations (casts, reinterpret_cast)
/// to find the original defining operation before making the determination.
static bool isRematerializationCandidate(Value val,
                                         acc::OpenACCSupport &accSupport) {
  // Trace through view-like operations to find the original value.
  Value origVal = getOriginalValue(val);
  Operation *definingOp = origVal.getDefiningOp();
  if (!definingOp && !(definingOp = val.getDefiningOp()))
    return false;

```
- **EN**: Implements logic around `isRematerializationCandidate`, `getOriginalValue`, `getDefiningOp`.
- **CN**: 围绕 `isRematerializationCandidate`, `getOriginalValue`, `getDefiningOp` 实现具体逻辑。

### Lines 141-148
```cpp
  LLVM_DEBUG(llvm::dbgs() << "\tChecking candidate: " << *definingOp << "\n");

  // Constants are trivial and useful to rematerialize.
  if (matchPattern(definingOp, m_Constant())) {
    LLVM_DEBUG(llvm::dbgs() << "\t\t-> constant pattern matched\n");
    return true;
  }

```
- **EN**: Implements logic around `dbgs`, `matchPattern`.
- **CN**: 围绕 `dbgs`, `matchPattern` 实现具体逻辑。

### Lines 149-161
```cpp
  // Operations implementing OutlineRematerializationOpInterface are candidates.
  if (isa<acc::OutlineRematerializationOpInterface>(definingOp)) {
    LLVM_DEBUG(llvm::dbgs() << "\t\t-> OutlineRematerializationOpInterface\n");
    return true;
  }

  // Address-of operations referencing globals that are valid in GPU regions
  // or referencing constant globals should be rematerialized.
  if (auto addrOfOp = dyn_cast<acc::AddressOfGlobalOpInterface>(definingOp)) {
    SymbolRefAttr symbol = addrOfOp.getSymbol();
    LLVM_DEBUG(llvm::dbgs()
               << "\t\tAddressOfGlobalOpInterface, symbol: " << symbol << "\n");

```
- **EN**: Implements logic around `OutlineRematerializationOpInterface>`, `dbgs`, `AddressOfGlobalOpInterface>`, `getSymbol`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `OutlineRematerializationOpInterface>`, `dbgs`, `AddressOfGlobalOpInterface>`, `getSymbol` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 162-170
```cpp
    // If the symbol is already valid in GPU regions (e.g., has acc.declare),
    // rematerializing ensures the address refers to the device copy.
    Operation *globalOp = nullptr;
    if (accSupport.isValidSymbolUse(definingOp, symbol, &globalOp)) {
      LLVM_DEBUG(llvm::dbgs() << "\t\t-> isValidSymbolUse: true\n");
      return true;
    }
    LLVM_DEBUG(llvm::dbgs() << "\t\t-> isValidSymbolUse: false\n");

```
- **EN**: Implements logic around `isValidSymbolUse`, `dbgs`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isValidSymbolUse`, `dbgs` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 171-183
```cpp
    // If the referenced global is constant, prefer rematerialization so the
    // constant can be placed in GPU memory.
    if (globalOp) {
      if (auto globalVarOp =
              dyn_cast<acc::GlobalVariableOpInterface>(globalOp)) {
        if (globalVarOp.isConstant()) {
          LLVM_DEBUG(llvm::dbgs() << "\t\t-> constant global\n");
          return true;
        }
      }
    }
  }

```
- **EN**: Implements logic around `GlobalVariableOpInterface>`, `isConstant`, `dbgs`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `GlobalVariableOpInterface>`, `isConstant`, `dbgs` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 184-196
```cpp
  // An op implementing both ViewLikeOpInterface and
  // OutlineRematerializationOpInterface may have been traced through by
  // getOriginalValue. If the traced op is not a candidate, check the direct
  // defining op of the live-in value.
  if (origVal != val) {
    if (isa_and_nonnull<acc::OutlineRematerializationOpInterface>(
            val.getDefiningOp())) {
      LLVM_DEBUG(llvm::dbgs()
                 << "\t\t-> OutlineRematerializationOpInterface (direct)\n");
      return true;
    }
  }

```
- **EN**: Implements logic around `OutlineRematerializationOpInterface>`, `getDefiningOp`, `dbgs`, `OutlineRematerializationOpInterface`.
- **CN**: 围绕 `OutlineRematerializationOpInterface>`, `getDefiningOp`, `dbgs`, `OutlineRematerializationOpInterface` 实现具体逻辑。

### Lines 197-204
```cpp
  LLVM_DEBUG(llvm::dbgs() << "\t\t-> not a candidate\n");
  return false;
}

class OffloadLiveInValueCanonicalization
    : public acc::impl::OffloadLiveInValueCanonicalizationBase<
          OffloadLiveInValueCanonicalization> {
public:
```
- **EN**: Introduces declarations for `OffloadLiveInValueCanonicalization`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `OffloadLiveInValueCanonicalization` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 205-218
```cpp
  using acc::impl::OffloadLiveInValueCanonicalizationBase<
      OffloadLiveInValueCanonicalization>::
      OffloadLiveInValueCanonicalizationBase;

  /// Canonicalizes live-in values for a region by sinking or rematerializing
  /// operations. Returns true if any changes were made.
  bool canonicalizeLiveInValues(Region &region,
                                acc::OpenACCSupport &accSupport) {
    // 1) Collect live-in values.
    SetVector<Value> liveInValues;
    getUsedValuesDefinedAbove(region, liveInValues);
    LLVM_DEBUG(llvm::dbgs()
               << "\tFound " << liveInValues.size() << " live-in value(s)\n");

```
- **EN**: Implements logic around `canonicalizeLiveInValues`, `getUsedValuesDefinedAbove`, `dbgs`, `size`.
- **CN**: 围绕 `canonicalizeLiveInValues`, `getUsedValuesDefinedAbove`, `dbgs`, `size` 实现具体逻辑。

### Lines 219-227
```cpp
    auto isSinkCandidate = [&region, &accSupport](Value val) -> bool {
      return isRematerializationCandidate(val, accSupport) &&
             allUsersAreInsideRegion(val, region);
    };
    auto isCloneCandidate = [&region, &accSupport](Value val) -> bool {
      return isRematerializationCandidate(val, accSupport) &&
             !allUsersAreInsideRegion(val, region);
    };

```
- **EN**: Implements logic around `isRematerializationCandidate`, `allUsersAreInsideRegion`.
- **CN**: 围绕 `isRematerializationCandidate`, `allUsersAreInsideRegion` 实现具体逻辑。

### Lines 228-237
```cpp
    // 2) Filter values into two sets - sink and rematerialization candidates.
    SmallVector<Value> sinkCandidates(
        llvm::make_filter_range(liveInValues, isSinkCandidate));
    SmallVector<Value> rematerializationCandidates(
        llvm::make_filter_range(liveInValues, isCloneCandidate));

    LLVM_DEBUG(llvm::dbgs() << "\tSink candidates: " << sinkCandidates.size()
                            << ", clone candidates: "
                            << rematerializationCandidates.size() << "\n");

```
- **EN**: Implements logic around `sinkCandidates`, `make_filter_range`, `rematerializationCandidates`, `dbgs`, and 1 more symbols.
- **CN**: 围绕 `sinkCandidates`, `make_filter_range`, `rematerializationCandidates`, `dbgs`, and 1 more symbols 实现具体逻辑。

### Lines 238-251
```cpp
    if (rematerializationCandidates.empty() && sinkCandidates.empty())
      return false;

    LLVM_DEBUG(llvm::dbgs() << "\tCanonicalizing values into "
                            << *region.getParentOp() << "\n");

    // 3) Handle the sink set by moving the operations into the region.
    for (Value sinkCandidate : sinkCandidates) {
      Operation *sinkOp = sinkCandidate.getDefiningOp();
      assert(sinkOp && "must have op to be considered");
      sinkOp->moveBefore(&region.front().front());
      LLVM_DEBUG(llvm::dbgs() << "\t\tSunk: " << *sinkOp << "\n");
    }

```
- **EN**: Implements logic around `empty`, `dbgs`, `getParentOp`, `getDefiningOp`, and 2 more symbols.
- **CN**: 围绕 `empty`, `dbgs`, `getParentOp`, `getDefiningOp`, and 2 more symbols 实现具体逻辑。

### Lines 252-265
```cpp
    // 4) Handle the rematerialization set by copying the operations into
    // the region.
    OpBuilder builder(region);
    SmallVector<Operation *> opsToRematerialize;
    for (Value rematerializationCandidate : rematerializationCandidates) {
      Operation *rematerializationOp =
          rematerializationCandidate.getDefiningOp();
      assert(rematerializationOp && "must have op to be considered");
      opsToRematerialize.push_back(rematerializationOp);
    }
    computeTopologicalSorting(opsToRematerialize);
    for (Operation *rematerializationOp : opsToRematerialize) {
      Operation *clonedOp = builder.clone(*rematerializationOp);
      for (auto [oldResult, newResult] : llvm::zip(
```
- **EN**: Implements logic around `builder`, `getDefiningOp`, `assert`, `push_back`, and 3 more symbols.
- **CN**: 围绕 `builder`, `getDefiningOp`, `assert`, `push_back`, and 3 more symbols 实现具体逻辑。

### Lines 266-274
```cpp
               rematerializationOp->getResults(), clonedOp->getResults())) {
        replaceAllUsesInRegionWith(oldResult, newResult, region);
      }
      LLVM_DEBUG(llvm::dbgs() << "\t\tCloned: " << *clonedOp << "\n");
    }

    return true;
  }

```
- **EN**: Implements logic around `getResults`, `replaceAllUsesInRegionWith`, `dbgs`.
- **CN**: 围绕 `getResults`, `replaceAllUsesInRegionWith`, `dbgs` 实现具体逻辑。

### Lines 275-287
```cpp
  void runOnOperation() override {
    LLVM_DEBUG(llvm::dbgs() << "Enter OffloadLiveInValueCanonicalization\n");

    // Since OpenACCSupport is normally registered on modules, attempt to
    // get it from the parent module first (if available), then fallback
    // to the per-function analysis.
    acc::OpenACCSupport *accSupportPtr = nullptr;
    if (auto parentAnalysis = getCachedParentAnalysis<acc::OpenACCSupport>())
      accSupportPtr = &parentAnalysis->get();
    else
      accSupportPtr = &getAnalysis<acc::OpenACCSupport>();
    acc::OpenACCSupport &accSupport = *accSupportPtr;

```
- **EN**: Implements logic around `runOnOperation`, `dbgs`, `OpenACCSupport>`, `get`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `dbgs`, `OpenACCSupport>`, `get` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 288-297
```cpp
    func::FuncOp func = getOperation();
    LLVM_DEBUG(llvm::dbgs()
               << "Processing function: " << func.getName() << "\n");

    func.walk([&](Operation *op) {
      if (isa<acc::OffloadRegionOpInterface>(op)) {
        LLVM_DEBUG(llvm::dbgs()
                   << "Found offload region: " << op->getName() << "\n");
        assert(op->getNumRegions() == 1 && "must have 1 region");

```
- **EN**: Implements logic around `getOperation`, `dbgs`, `getName`, `walk`, and 2 more symbols.
- **CN**: 围绕 `getOperation`, `dbgs`, `getName`, `walk`, and 2 more symbols 实现具体逻辑。

### Lines 298-310
```cpp
        // Canonicalization of values changes live-in set.
        // Rerun the algorithm until convergence.
        bool changes = false;
        [[maybe_unused]] int iteration = 0;
        do {
          LLVM_DEBUG(llvm::dbgs() << "\tIteration " << iteration++ << "\n");
          changes = canonicalizeLiveInValues(op->getRegion(0), accSupport);
        } while (changes);
        LLVM_DEBUG(llvm::dbgs()
                   << "\tConverged after " << iteration << " iteration(s)\n");
      }
    });

```
- **EN**: Implements logic around `dbgs`, `canonicalizeLiveInValues`, `iteration`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `dbgs`, `canonicalizeLiveInValues`, `iteration` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 311-315
```cpp
    LLVM_DEBUG(llvm::dbgs() << "Exit OffloadLiveInValueCanonicalization\n");
  }
};

} // namespace
```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

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
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Analysis/TopologicalSortUtils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Builders.h`, `mlir/IR/Matchers.h`, `mlir/IR/Operation.h`, `mlir/IR/Region.h`, `mlir/IR/SymbolTable.h` ... (+7 more)
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (6), dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2), pass infrastructure and registration support / Pass 基础设施与注册支持 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
