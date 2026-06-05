# OffloadTargetVerifier.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/OffloadTargetVerifier.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass verifies that values and symbols used within offload regions are legal for the target execution model.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- OffloadTargetVerifier.cpp ------------------------------------------===//
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
// This pass verifies that values and symbols used within offload regions are
// legal for the target execution model.
//
// Overview:
// ---------
// Offload regions execute on a target device (e.g., GPU) where not all values
// and symbols from the host context are accessible. This pass checks that
// live-in values (values defined outside but used inside the region) and
// symbol references are valid for device execution.
//
// The pass operates on any operation implementing `OffloadRegionOpInterface`,
// which includes OpenACC compute constructs (`acc.parallel`, `acc.kernels`,
// `acc.serial`) as well as GPU operations like `gpu.launch`.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
//
// Verification:
// -------------
// For each offload region, the pass checks:
//
// 1. Live-in Values: Values flowing into the region must be valid for device
//    use. This includes checking that data has been properly mapped via
//    OpenACC data clauses (copyin, copyout, present, etc.) or is a scalar
//    that can be passed by value.
//
// 2. Symbol References: Symbols referenced inside the region must be
//    accessible on the device. This includes checking for proper `declare`
//    attributes on globals or device-resident data attributes.
//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-49
```cpp
// Requirements:
// -------------
// 1. Target Region Identification: Operations representing offload regions
//    must implement `acc::OffloadRegionOpInterface`.
//
// 2. OpenACCSupport Analysis: The pass relies on the `OpenACCSupport`
//    analysis to determine value and symbol validity. This analysis provides
//    dialect-specific hooks for checking legality through `isValidValueUse`
//    and `isValidSymbolUse` methods. Custom dialect support can be registered
//    by providing a derived `OpenACCSupport` analysis before running this
//    pass.
//
// 3. Device Type: The `device_type` option specifies the target device.
//    For `host` or `multicore` targets, verification of ACC compute
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 50-61
```cpp
//    constructs is not yet implemented.
//
//===----------------------------------------------------------------------===//

#include "mlir/Analysis/Liveness.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/OpenACC/Transforms/Passes.h"
#include "mlir/IR/SymbolTable.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Analysis/Liveness.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Analysis/Liveness.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`。

### Lines 62-68
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_OFFLOADTARGETVERIFIER
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 69-75
```cpp
#define DEBUG_TYPE "offload-target-verifier"

using namespace mlir;

namespace {

class OffloadTargetVerifier
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 76-85
```cpp
    : public acc::impl::OffloadTargetVerifierBase<OffloadTargetVerifier> {
public:
  using OffloadTargetVerifierBase::OffloadTargetVerifierBase;

  /// Returns true if the target device type corresponds to host execution.
  bool isHostTarget() const {
    return deviceType == acc::DeviceType::Host ||
           deviceType == acc::DeviceType::Multicore;
  }

```
- **EN**: Implements logic around `isHostTarget`.
- **CN**: 围绕 `isHostTarget` 实现具体逻辑。

### Lines 86-93
```cpp
  /// Check live-in values for legality.
  SmallVector<Value>
  getIllegalLiveInValues(Region &region, Liveness &liveness,
                         acc::OpenACCSupport &accSupport) const {
    auto isInvalid = [&](Value val) -> bool {
      return !accSupport.isValidValueUse(val, region);
    };

```
- **EN**: Implements logic around `getIllegalLiveInValues`, `isValidValueUse`.
- **CN**: 围绕 `getIllegalLiveInValues`, `isValidValueUse` 实现具体逻辑。

### Lines 94-100
```cpp
    SmallVector<Value> illegalValues(llvm::make_filter_range(
        liveness.getLiveIn(&region.front()), isInvalid));

    return illegalValues;
  }

  /// Check symbol uses for legality.
```
- **EN**: Implements logic around `illegalValues`, `getLiveIn`.
- **CN**: 围绕 `illegalValues`, `getLiveIn` 实现具体逻辑。

### Lines 101-108
```cpp
  SmallVector<SymbolTable::SymbolUse>
  getIllegalUsedSymbols(Region &region, acc::OpenACCSupport &accSupport) const {
    auto symUses = SymbolTable::getSymbolUses(&region);

    // When there are no symbols used in the region, there are no illegal ones.
    if (!symUses.has_value())
      return {};

```
- **EN**: Implements logic around `getIllegalUsedSymbols`, `getSymbolUses`, `has_value`.
- **CN**: 围绕 `getIllegalUsedSymbols`, `getSymbolUses`, `has_value` 实现具体逻辑。

### Lines 109-120
```cpp
    auto isInvalidSymbol = [&](const SymbolTable::SymbolUse &symUse) -> bool {
      Operation *definingOp = nullptr;
      return !accSupport.isValidSymbolUse(symUse.getUser(),
                                          symUse.getSymbolRef(), &definingOp);
    };

    auto invalidSyms =
        llvm::make_filter_range(symUses.value(), isInvalidSymbol);
    SmallVector<SymbolTable::SymbolUse> invalidSymsList(invalidSyms);
    return invalidSymsList;
  }

```
- **EN**: Implements logic around `isValidSymbolUse`, `getSymbolRef`, `make_filter_range`, `invalidSymsList`.
- **CN**: 围绕 `isValidSymbolUse`, `getSymbolRef`, `make_filter_range`, `invalidSymsList` 实现具体逻辑。

### Lines 121-130
```cpp
  /// Check if the region has illegal live-in values.
  bool hasIllegalLiveInValues(Operation *regionOp,
                              acc::OpenACCSupport &accSupport) const {
    if (regionOp->getNumRegions() == 0)
      return false;

    Liveness liveness(regionOp);
    SmallVector<Value> invalidValues =
        getIllegalLiveInValues(regionOp->getRegion(0), liveness, accSupport);

```
- **EN**: Implements logic around `hasIllegalLiveInValues`, `getNumRegions`, `liveness`, `getIllegalLiveInValues`.
- **CN**: 围绕 `hasIllegalLiveInValues`, `getNumRegions`, `liveness`, `getIllegalLiveInValues` 实现具体逻辑。

### Lines 131-144
```cpp
    bool hasIllegalValues = !invalidValues.empty();

    if (hasIllegalValues) {
      if (softCheck) {
        // Emit warnings for each illegal value.
        auto diag = regionOp->emitWarning("offload target verifier: ")
                    << invalidValues.size() << " illegal live-in value(s)";
        for (auto [idx, invalidValue] : llvm::enumerate(invalidValues)) {
          diag.attachNote(invalidValue.getLoc()) << "value: " << invalidValue;
        }
      } else {
        accSupport.emitNYI(regionOp->getLoc(),
                           "offload target verifier failed due to " +
                               Twine(invalidValues.size()) +
```
- **EN**: Implements logic around `empty`, `emitWarning`, `size`, `enumerate`, and 3 more symbols.
- **CN**: 围绕 `empty`, `emitWarning`, `size`, `enumerate`, and 3 more symbols 实现具体逻辑。

### Lines 145-151
```cpp
                               " illegal live-in value(s)");
      }
    }

    return hasIllegalValues;
  }

```
- **EN**: Implements logic around `value`.
- **CN**: 围绕 `value` 实现具体逻辑。

### Lines 152-160
```cpp
  /// Check if the region has illegal symbol uses.
  bool hasIllegalSymbolUses(Operation *regionOp,
                            acc::OpenACCSupport &accSupport) const {
    if (regionOp->getNumRegions() == 0)
      return false;

    SmallVector<SymbolTable::SymbolUse> invalidSyms =
        getIllegalUsedSymbols(regionOp->getRegion(0), accSupport);

```
- **EN**: Implements logic around `hasIllegalSymbolUses`, `getNumRegions`, `getIllegalUsedSymbols`.
- **CN**: 围绕 `hasIllegalSymbolUses`, `getNumRegions`, `getIllegalUsedSymbols` 实现具体逻辑。

### Lines 161-169
```cpp
    bool hasIllegalSymbols = !invalidSyms.empty();

    if (hasIllegalSymbols) {
      auto getSymName = [&](SymbolTable::SymbolUse symUse) -> std::string {
        return symUse.getSymbolRef().getLeafReference().str();
      };
      std::string invalidString =
          llvm::join(llvm::map_range(invalidSyms, getSymName), ", ");

```
- **EN**: Implements logic around `empty`, `getSymbolRef`, `join`.
- **CN**: 围绕 `empty`, `getSymbolRef`, `join` 实现具体逻辑。

### Lines 170-180
```cpp
      // Emit only warnings when softCheck is enabled.
      if (softCheck)
        regionOp->emitWarning("offload target verifier: illegal symbol(s): ")
            << invalidString;
      else
        accSupport.emitNYI(regionOp->getLoc(),
                           "offload target verifier failed due to illegal "
                           "symbol(s): " +
                               invalidString);
    }

```
- **EN**: Implements logic around `emitWarning`, `emitNYI`, `symbol`.
- **CN**: 围绕 `emitWarning`, `emitNYI`, `symbol` 实现具体逻辑。

### Lines 181-187
```cpp
    return hasIllegalSymbols;
  }

  void runOnOperation() override {
    LLVM_DEBUG(llvm::dbgs() << "Enter OffloadTargetVerifier()\n");
    func::FuncOp func = getOperation();

```
- **EN**: Implements logic around `runOnOperation`, `dbgs`, `getOperation`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `dbgs`, `getOperation` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 188-194
```cpp
    // Try to get cached parent analysis first, fall back to local analysis.
    auto cachedAnalysis =
        getCachedParentAnalysis<acc::OpenACCSupport>(func->getParentOp());
    acc::OpenACCSupport &accSupport = cachedAnalysis
                                          ? cachedAnalysis->get()
                                          : getAnalysis<acc::OpenACCSupport>();

```
- **EN**: Implements logic around `OpenACCSupport>`, `get`.
- **CN**: 围绕 `OpenACCSupport>`, `get` 实现具体逻辑。

### Lines 195-201
```cpp
    bool hasErrors = false;

    func.walk([&](Operation *op) {
      // Only process offload region operations.
      if (!isa<acc::OffloadRegionOpInterface>(op))
        return WalkResult::advance();

```
- **EN**: Implements logic around `walk`, `OffloadRegionOpInterface>`, `advance`.
- **CN**: 围绕 `walk`, `OffloadRegionOpInterface>`, `advance` 实现具体逻辑。

### Lines 202-210
```cpp
      // TODO: Host/multicore verification for ACC compute constructs is not yet
      // implemented.
      if (isHostTarget() && isa<ACC_COMPUTE_CONSTRUCT_OPS>(op)) {
        accSupport.emitNYI(op->getLoc(),
                           "host/multicore verification for ACC compute "
                           "constructs");
        return WalkResult::advance();
      }

```
- **EN**: Implements logic around `isHostTarget`, `emitNYI`, `advance`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `isHostTarget`, `emitNYI`, `advance` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 211-220
```cpp
      // Check for illegal live-in values.
      bool hasIllegalValues = hasIllegalLiveInValues(op, accSupport);
      if (hasIllegalValues)
        hasErrors = true;

      // Check for illegal symbol uses.
      bool hasIllegalSyms = hasIllegalSymbolUses(op, accSupport);
      if (hasIllegalSyms)
        hasErrors = true;

```
- **EN**: Implements logic around `hasIllegalLiveInValues`, `hasIllegalSymbolUses`.
- **CN**: 围绕 `hasIllegalLiveInValues`, `hasIllegalSymbolUses` 实现具体逻辑。

### Lines 221-229
```cpp
      if (!hasIllegalValues && !hasIllegalSyms && softCheck)
        op->emitRemark("offload target verifier: passed validity check");

      return WalkResult::advance();
    });

    if (hasErrors && !softCheck)
      signalPassFailure();

```
- **EN**: Implements logic around `emitRemark`, `advance`, `signalPassFailure`.
- **CN**: 围绕 `emitRemark`, `advance`, `signalPassFailure` 实现具体逻辑。

### Lines 230-234
```cpp
    LLVM_DEBUG(llvm::dbgs() << "Exit OffloadTargetVerifier()\n");
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/Liveness.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/IR/SymbolTable.h`, `llvm/Support/Debug.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
