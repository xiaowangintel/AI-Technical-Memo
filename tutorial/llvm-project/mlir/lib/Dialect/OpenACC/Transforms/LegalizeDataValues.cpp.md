# LegalizeDataValues.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/LegalizeDataValues.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements OpenACC dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `LegalizeDataValues`.
  - **CN**: 实现 OpenACC 方言中围绕 `LegalizeDataValues` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LegalizeDataValues.cpp - -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-17
```cpp

#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/IR/Dominance.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Dominance.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Dominance.h`。

### Lines 18-24
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_LEGALIZEDATAVALUESINREGION
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 25-32
```cpp
using namespace mlir;

namespace {

static bool insideAccComputeRegion(mlir::Operation *op) {
  return op->getParentOfType<ACC_COMPUTE_CONSTRUCT_OPS>();
}

```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 33-46
```cpp
static void collectVars(mlir::ValueRange operands,
                        llvm::SmallVector<std::pair<Value, Value>> &values,
                        bool hostToDevice) {
  for (auto operand : operands) {
    Value var = acc::getVar(operand.getDefiningOp());
    Value accVar = acc::getAccVar(operand.getDefiningOp());
    if (var && accVar) {
      if (hostToDevice)
        values.push_back({var, accVar});
      else
        values.push_back({accVar, var});
    }
  }
}
```
- **EN**: Implements logic around `collectVars`, `getVar`, `getAccVar`, `push_back`.
- **CN**: 围绕 `collectVars`, `getVar`, `getAccVar`, `push_back` 实现具体逻辑。

### Lines 47-60
```cpp

template <typename Op>
static void replaceAllUsesInAccComputeRegionsWith(Value orig, Value replacement,
                                                  Region &outerRegion) {
  for (auto &use : llvm::make_early_inc_range(orig.getUses())) {
    if (outerRegion.isAncestor(use.getOwner()->getParentRegion())) {
      if constexpr (std::is_same_v<Op, acc::DataOp> ||
                    std::is_same_v<Op, acc::DeclareOp>) {
        // For data construct regions, only replace uses in contained compute
        // regions.
        if (insideAccComputeRegion(use.getOwner())) {
          use.set(replacement);
        }
      } else {
```
- **EN**: Implements logic around `replaceAllUsesInAccComputeRegionsWith`, `make_early_inc_range`, `isAncestor`, `constexpr`, and 2 more symbols.
- **CN**: 围绕 `replaceAllUsesInAccComputeRegionsWith`, `make_early_inc_range`, `isAncestor`, `constexpr`, and 2 more symbols 实现具体逻辑。

### Lines 61-71
```cpp
        use.set(replacement);
      }
    }
  }
}

template <typename Op>
static void replaceAllUsesInUnstructuredComputeRegionWith(
    Op &op, llvm::SmallVector<std::pair<Value, Value>> &values,
    DominanceInfo &domInfo, PostDominanceInfo &postDomInfo) {

```
- **EN**: Implements logic around `set`, `replaceAllUsesInUnstructuredComputeRegionWith`.
- **CN**: 围绕 `set`, `replaceAllUsesInUnstructuredComputeRegionWith` 实现具体逻辑。

### Lines 72-82
```cpp
  SmallVector<Operation *> exitOps;
  if constexpr (std::is_same_v<Op, acc::DeclareEnterOp>) {
    // For declare enter/exit pairs, collect all exit ops
    for (auto *user : op.getToken().getUsers()) {
      if (auto declareExit = dyn_cast<acc::DeclareExitOp>(user))
        exitOps.push_back(declareExit);
    }
    if (exitOps.empty())
      return;
  }

```
- **EN**: Implements logic around `constexpr`, `getToken`, `DeclareExitOp>`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `constexpr`, `getToken`, `DeclareExitOp>`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 83-92
```cpp
  for (auto p : values) {
    Value hostVal = std::get<0>(p);
    Value deviceVal = std::get<1>(p);
    for (auto &use : llvm::make_early_inc_range(hostVal.getUses())) {
      Operation *owner = use.getOwner();

      // Check It's the case that the acc entry operation dominates the use.
      if (!domInfo.dominates(op.getOperation(), owner))
        continue;

```
- **EN**: Implements logic around `get`, `make_early_inc_range`, `getOwner`, `dominates`.
- **CN**: 围绕 `get`, `make_early_inc_range`, `getOwner`, `dominates` 实现具体逻辑。

### Lines 93-102
```cpp
      // Check It's the case that at least one of the acc exit operations
      // post-dominates the use
      bool hasPostDominatingExit = false;
      for (auto *exit : exitOps) {
        if (postDomInfo.postDominates(exit, owner)) {
          hasPostDominatingExit = true;
          break;
        }
      }

```
- **EN**: Implements logic around `postDominates`.
- **CN**: 围绕 `postDominates` 实现具体逻辑。

### Lines 103-111
```cpp
      if (!hasPostDominatingExit)
        continue;

      if (insideAccComputeRegion(owner))
        use.set(deviceVal);
    }
  }
}

```
- **EN**: Implements logic around `insideAccComputeRegion`, `set`.
- **CN**: 围绕 `insideAccComputeRegion`, `set` 实现具体逻辑。

### Lines 112-118
```cpp
template <typename Op>
static void
collectAndReplaceInRegion(Op &op, bool hostToDevice,
                          DominanceInfo *domInfo = nullptr,
                          PostDominanceInfo *postDomInfo = nullptr) {
  llvm::SmallVector<std::pair<Value, Value>> values;

```
- **EN**: Implements logic around `collectAndReplaceInRegion`.
- **CN**: 围绕 `collectAndReplaceInRegion` 实现具体逻辑。

### Lines 119-132
```cpp
  if constexpr (std::is_same_v<Op, acc::LoopOp>) {
    collectVars(op.getReductionOperands(), values, hostToDevice);
    collectVars(op.getPrivateOperands(), values, hostToDevice);
  } else {
    collectVars(op.getDataClauseOperands(), values, hostToDevice);
    if constexpr (!std::is_same_v<Op, acc::KernelsOp> &&
                  !std::is_same_v<Op, acc::DataOp> &&
                  !std::is_same_v<Op, acc::DeclareOp> &&
                  !std::is_same_v<Op, acc::HostDataOp> &&
                  !std::is_same_v<Op, acc::DeclareEnterOp>) {
      collectVars(op.getReductionOperands(), values, hostToDevice);
      collectVars(op.getPrivateOperands(), values, hostToDevice);
      collectVars(op.getFirstprivateOperands(), values, hostToDevice);
    }
```
- **EN**: Implements logic around `constexpr`, `collectVars`.
- **CN**: 围绕 `constexpr`, `collectVars` 实现具体逻辑。

### Lines 133-146
```cpp
  }

  if constexpr (std::is_same_v<Op, acc::DeclareEnterOp>) {
    assert(domInfo && postDomInfo &&
           "Dominance info required for DeclareEnterOp");
    replaceAllUsesInUnstructuredComputeRegionWith<Op>(op, values, *domInfo,
                                                      *postDomInfo);
  } else {
    for (auto p : values) {
      replaceAllUsesInAccComputeRegionsWith<Op>(std::get<0>(p), std::get<1>(p),
                                                op.getRegion());
    }
  }
}
```
- **EN**: Implements logic around `constexpr`, `assert`, `replaceAllUsesInUnstructuredComputeRegionWith`, `replaceAllUsesInAccComputeRegionsWith`, and 1 more symbols.
- **CN**: 围绕 `constexpr`, `assert`, `replaceAllUsesInUnstructuredComputeRegionWith`, `replaceAllUsesInAccComputeRegionsWith`, and 1 more symbols 实现具体逻辑。

### Lines 147-154
```cpp

class LegalizeDataValuesInRegion
    : public acc::impl::LegalizeDataValuesInRegionBase<
          LegalizeDataValuesInRegion> {
public:
  using LegalizeDataValuesInRegionBase<
      LegalizeDataValuesInRegion>::LegalizeDataValuesInRegionBase;

```
- **EN**: Introduces declarations for `LegalizeDataValuesInRegion`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LegalizeDataValuesInRegion` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 155-163
```cpp
  void runOnOperation() override {
    func::FuncOp funcOp = getOperation();
    bool replaceHostVsDevice = this->hostToDevice.getValue();

    // Initialize dominance info
    DominanceInfo domInfo;
    PostDominanceInfo postDomInfo;
    bool computedDomInfo = false;

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `getValue`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `getValue` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 164-170
```cpp
    funcOp.walk([&](Operation *op) {
      if (!isa<ACC_COMPUTE_CONSTRUCT_AND_LOOP_OPS>(*op) &&
          !(isa<ACC_DATA_CONSTRUCT_STRUCTURED_OPS>(*op) &&
            applyToAccDataConstruct) &&
          !isa<acc::DeclareEnterOp>(*op))
        return;

```
- **EN**: Implements logic around `walk`, `DeclareEnterOp>`.
- **CN**: 围绕 `walk`, `DeclareEnterOp>` 实现具体逻辑。

### Lines 171-184
```cpp
      if (auto parallelOp = dyn_cast<acc::ParallelOp>(*op)) {
        collectAndReplaceInRegion(parallelOp, replaceHostVsDevice);
      } else if (auto serialOp = dyn_cast<acc::SerialOp>(*op)) {
        collectAndReplaceInRegion(serialOp, replaceHostVsDevice);
      } else if (auto kernelsOp = dyn_cast<acc::KernelsOp>(*op)) {
        collectAndReplaceInRegion(kernelsOp, replaceHostVsDevice);
      } else if (auto loopOp = dyn_cast<acc::LoopOp>(*op)) {
        collectAndReplaceInRegion(loopOp, replaceHostVsDevice);
      } else if (auto dataOp = dyn_cast<acc::DataOp>(*op)) {
        collectAndReplaceInRegion(dataOp, replaceHostVsDevice);
      } else if (auto declareOp = dyn_cast<acc::DeclareOp>(*op)) {
        collectAndReplaceInRegion(declareOp, replaceHostVsDevice);
      } else if (auto hostDataOp = dyn_cast<acc::HostDataOp>(*op)) {
        collectAndReplaceInRegion(hostDataOp, replaceHostVsDevice);
```
- **EN**: Implements logic around `ParallelOp>`, `collectAndReplaceInRegion`, `SerialOp>`, `KernelsOp>`, and 4 more symbols.
- **CN**: 围绕 `ParallelOp>`, `collectAndReplaceInRegion`, `SerialOp>`, `KernelsOp>`, and 4 more symbols 实现具体逻辑。

### Lines 185-198
```cpp
      } else if (auto declareEnterOp = dyn_cast<acc::DeclareEnterOp>(*op)) {
        if (!computedDomInfo) {
          domInfo = DominanceInfo(funcOp);
          postDomInfo = PostDominanceInfo(funcOp);
          computedDomInfo = true;
        }
        collectAndReplaceInRegion(declareEnterOp, replaceHostVsDevice, &domInfo,
                                  &postDomInfo);
      } else {
        llvm_unreachable("unsupported acc region op");
      }
    });
  }
};
```
- **EN**: Implements logic around `DeclareEnterOp>`, `DominanceInfo`, `PostDominanceInfo`, `collectAndReplaceInRegion`.
- **CN**: 围绕 `DeclareEnterOp>`, `DominanceInfo`, `PostDominanceInfo`, `collectAndReplaceInRegion` 实现具体逻辑。

### Lines 199-200
```cpp

} // end anonymous namespace
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
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Dominance.h`, `mlir/Pass/Pass.h`, `mlir/Transforms/RegionUtils.h`, `llvm/Support/ErrorHandling.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), pass infrastructure and registration support / Pass 基础设施与注册支持 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
