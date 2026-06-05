# ACCRoutineToGPUFunc.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCRoutineToGPUFunc.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: The OpenACC `routine` directive defines functions that may be invoked from device code. Those functions need to be available in the device compilation unit. This pass moves materialized acc routines into the GPU module as gpu.func operations so they can be compiled for the device.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===- ACCRoutineToGPUFunc.cpp - Move ACC routines to GPU module ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The OpenACC `routine` directive defines functions that may be invoked from
// device code. Those functions need to be available in the device compilation
// unit. This pass moves materialized acc routines into the GPU module as
// gpu.func operations so they can be compiled for the device.
//
// Overview:
// ---------
// For each acc.routine that is not bound by name, the corresponding
// specialized function (created by ACCRoutineLowering) or the original
// host function (in case of seq) is cloned into theGPU module as a gpu.func.
// Callees referenced from those routines are processed: device-valid callees
// (runtime, intrinsics, other acc routines) are added to the GPU module as
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 21-40
```cpp
// declarations or full clones as needed. Bind-name routines are not moved;
// their acc.routine ops are erased. After cloning, the host copies of
// specialized device functions and nohost routines are removed.
//
// Approach:
// ----------------
// 1. Collect materialized routines (acc.routine without bind(name)); record
//    bind-name routines for erasure. Emit remarks for materialized routines.
//
// 2. Process calls: walk each materialized function; for each call, if the
//    callee is already in the GPU module or is an acc routine (or specialized
//    acc routine), skip; otherwise require OpenACCSupport::isValidSymbolUse.
//    Valid callees are added to the clone set (as declaration or full clone).
//
// 3. Clone into GPU module: each function in the clone set is turned into a
//    gpu.func (body cloned or declaration only). acc.specialized_routine is
//    preserved and symbol uses are updated so the routine name is unchanged.
//
// 4. Cleanup: erase from the host module the specialized device function
//    bodies and any nohost routine (host copy removed after move to device).
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 41-60
```cpp
//
// Example:
// --------
// Before (after ACCRoutineLowering):
//   acc.routine @r_seq func(@foo) seq
//   func.func @foo() attributes {acc.specialized_routine = ...} { ... }
//
// After:
//   acc.routine @r_seq func(@foo) seq
//   gpu.module @acc_gpu_module {
//     gpu.func @foo() attributes {acc.specialized_routine = ...} { ... }
//   }
//   (host @foo erased)
//
// Requirements:
// -------------
// - Must run after `ACCRoutineLowering` pass  which ensures variants for all
//   levels of parallelism are created.
// - Uses OpenACCSupport: getOrCreateGPUModule, isValidSymbolUse, emitRemark,
//   emitNYI. If no custom implementation is registered, the default is used.
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 61-76
```cpp
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/CallInterfaces.h"
#include "llvm/ADT/SetVector.h"
#include <string>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`。

### Lines 77-88
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCROUTINETOGPUFUNC
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

#define DEBUG_TYPE "acc-routine-to-gpu-func"

using namespace mlir;
using namespace mlir::acc;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 89-103
```cpp
namespace {

/// Create a gpu.func from a func.func by cloning the body.
static gpu::GPUFuncOp createGPUFuncFromFunc(OpBuilder &builder,
                                            func::FuncOp sourceFunc) {
  Location loc = sourceFunc.getLoc();
  StringRef name = sourceFunc.getName();
  FunctionType type = sourceFunc.getFunctionType();
  // Do not copy any attributes from the source; specialized_routine is set
  // later when applicable.
  gpu::GPUFuncOp gpuFunc =
      gpu::GPUFuncOp::create(builder, loc, name, type,
                             /*workgroupAttributions=*/TypeRange(),
                             /*privateAttributions=*/TypeRange(), /*attrs=*/{});

```
- **EN**: Implements logic around `createGPUFuncFromFunc`, `getLoc`, `getName`, `getFunctionType`, and 2 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `createGPUFuncFromFunc`, `getLoc`, `getName`, `getFunctionType`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 104-115
```cpp
  Region &sourceBody = sourceFunc.getBody();
  Region &deviceBody = gpuFunc.getBody();
  Block &deviceEntryBlock = deviceBody.front();

  // Map source block arguments to the GPU func's entry block arguments (which
  // GPUFuncOp::create already created).
  IRMapping mapping;
  Block &sourceEntryBlock = sourceBody.front();
  for (auto [srcArg, destArg] : llvm::zip(sourceEntryBlock.getArguments(),
                                          deviceEntryBlock.getArguments()))
    mapping.map(srcArg, destArg);

```
- **EN**: Implements logic around `getBody`, `front`, `zip`, `getArguments`, and 1 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getBody`, `front`, `zip`, `getArguments`, and 1 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 116-125
```cpp
  sourceBody.cloneInto(&deviceBody, mapping);

  // Replace func.return with gpu.return in the cloned blocks.
  gpuFunc.walk([](func::ReturnOp op) {
    OpBuilder replacer(op);
    gpu::ReturnOp gpuReturn = gpu::ReturnOp::create(replacer, op.getLoc());
    gpuReturn->setOperands(op.getOperands());
    op.erase();
  });

```
- **EN**: Implements logic around `cloneInto`, `walk`, `replacer`, `create`, and 2 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `cloneInto`, `walk`, `replacer`, `create`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 126-137
```cpp
  // Splice the cloned entry block's operations into the GPU func's entry block
  // (cloneInto created a separate block for the cloned content), then remove
  // the now-empty cloned block.
  Block *clonedSourceEntry = mapping.lookup(&sourceEntryBlock);
  deviceEntryBlock.getOperations().splice(
      deviceEntryBlock.getOperations().end(),
      clonedSourceEntry->getOperations());
  clonedSourceEntry->erase();

  return gpuFunc;
}

```
- **EN**: Implements logic around `lookup`, `getOperations`, `erase`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookup`, `getOperations`, `erase` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 138-154
```cpp
using CloneCandidate = std::pair<func::FuncOp, RoutineOp>;

/// Collect materialized and bind routines; fill candidate func names and
/// materialized routine set. Emit remarks for materialized routines.
static void collectRoutineCandidates(
    ModuleOp mod, SymbolTable &symTab, acc::DeviceType deviceType,
    OpenACCSupport &accSupport,
    llvm::SmallSetVector<llvm::StringRef, 4> &funcsToCloneCandidates,
    llvm::SmallSetVector<RoutineOp, 4> &materializedAccRoutines,
    llvm::SmallSetVector<RoutineOp, 4> &bindAccRoutines) {
  auto isParallelRoutine = [deviceType](RoutineOp routineOp) {
    return routineOp.hasGang(deviceType) || routineOp.hasGang() ||
           routineOp.hasWorker(deviceType) || routineOp.hasWorker() ||
           routineOp.hasVector(deviceType) || routineOp.hasVector() ||
           routineOp.getGangDimValue(deviceType) || routineOp.getGangDimValue();
  };

```
- **EN**: Implements logic around `collectRoutineCandidates`, `hasGang`, `hasWorker`, `hasVector`, and 1 more symbols.
- **CN**: 围绕 `collectRoutineCandidates`, `hasGang`, `hasWorker`, `hasVector`, and 1 more symbols 实现具体逻辑。

### Lines 155-174
```cpp
  mod.walk([&](RoutineOp op) {
    if (op.getBindNameValue() || op.getBindNameValue(deviceType)) {
      bindAccRoutines.insert(op);
      return;
    }
    func::FuncOp callee =
        symTab.lookup<func::FuncOp>(op.getFuncName().getLeafReference());
    accSupport.emitRemark(
        callee ? callee.getOperation() : op.getOperation(),
        [&op, &isParallelRoutine]() {
          std::string msg = "Generating";
          if (op.getImplicitAttr())
            msg += " implicit";
          msg += " acc routine";
          if (!isParallelRoutine(op))
            msg += " seq";
          return msg;
        },
        DEBUG_TYPE);
    funcsToCloneCandidates.insert(op.getFuncName().getLeafReference());
```
- **EN**: Implements logic around `walk`, `getBindNameValue`, `insert`, `FuncOp>`, and 4 more symbols.
- **CN**: 围绕 `walk`, `getBindNameValue`, `insert`, `FuncOp>`, and 4 more symbols 实现具体逻辑。

### Lines 175-194
```cpp
    materializedAccRoutines.insert(op);
  });
}

/// Process calls in ACC routines: add valid callees to funcsToClone (for
/// declaration or clone). Returns failure() if any call is unsupported.
static LogicalResult processCallsInRoutines(
    SymbolTable &symTab, SymbolTable &gpuSymTab, OpenACCSupport &accSupport,
    const llvm::SmallSetVector<llvm::StringRef, 4> &funcsToCloneCandidates,
    const llvm::SmallSetVector<RoutineOp, 4> &materializedAccRoutines,
    llvm::SmallSetVector<CloneCandidate, 4> &funcsToClone) {
  LogicalResult callCheckResult = success();
  auto processCalls = [&](CallOpInterface callOp) {
    if (!callOp.getCallableForCallee())
      return;
    auto calleeSymbolRef =
        dyn_cast<SymbolRefAttr>(callOp.getCallableForCallee());
    if (!calleeSymbolRef)
      return;

```
- **EN**: Implements logic around `insert`, `processCallsInRoutines`, `success`, `getCallableForCallee`.
- **CN**: 围绕 `insert`, `processCallsInRoutines`, `success`, `getCallableForCallee` 实现具体逻辑。

### Lines 195-204
```cpp
    auto callee =
        symTab.lookup<func::FuncOp>(calleeSymbolRef.getLeafReference());
    if (!callee)
      return;

    if (gpuSymTab.lookup(callee.getName()))
      return;
    if (isAccRoutine(callee) || isSpecializedAccRoutine(callee))
      return;

```
- **EN**: Implements logic around `FuncOp>`, `lookup`, `isAccRoutine`.
- **CN**: 围绕 `FuncOp>`, `lookup`, `isAccRoutine` 实现具体逻辑。

### Lines 205-224
```cpp
    if (!accSupport.isValidSymbolUse(callOp.getOperation(), calleeSymbolRef)) {
      accSupport.emitNYI(callOp->getLoc(), "Unsupported call in acc routine");
      callCheckResult = failure();
      return;
    }
    funcsToClone.insert({callee, RoutineOp{}});
  };

  for (auto [funcName, accRoutine] :
       llvm::zip(funcsToCloneCandidates, materializedAccRoutines)) {
    func::FuncOp func = symTab.lookup<func::FuncOp>(funcName);
    if (!func)
      continue;
    if (!gpuSymTab.lookup(funcName))
      funcsToClone.insert({func, accRoutine});
    func.walk([&](CallOpInterface callOp) { processCalls(callOp); });
    if (failed(callCheckResult))
      return failure();
  }
  return success();
```
- **EN**: Implements logic around `isValidSymbolUse`, `emitNYI`, `failure`, `insert`, and 6 more symbols.
- **CN**: 围绕 `isValidSymbolUse`, `emitNYI`, `failure`, `insert`, and 6 more symbols 实现具体逻辑。

### Lines 225-235
```cpp
}

/// Clone each function in funcsToClone into the GPU module (declaration or
/// full body). Fix up symbol names and specialized_routine attr for ACC
/// routines.
static LogicalResult cloneFuncsToGPUModule(
    ModuleOp mod, OpenACCSupport &accSupport, SymbolTable &gpuSymTab,
    const llvm::SmallSetVector<CloneCandidate, 4> &funcsToClone) {
  MLIRContext *ctx = mod.getContext();
  OpBuilder builder(ctx);

```
- **EN**: Implements logic around `cloneFuncsToGPUModule`, `getContext`, `builder`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `cloneFuncsToGPUModule`, `getContext`, `builder` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 236-246
```cpp
  for (CloneCandidate candidate : funcsToClone) {
    func::FuncOp srcFunc = candidate.first;

    if (srcFunc.isDeclaration()) {
      Operation *cloned = srcFunc->clone();
      gpuSymTab.insert(cloned);
      continue;
    }

    gpu::GPUFuncOp deviceFuncOp = createGPUFuncFromFunc(builder, srcFunc);

```
- **EN**: Implements logic around `isDeclaration`, `clone`, `insert`, `createGPUFuncFromFunc`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isDeclaration`, `clone`, `insert`, `createGPUFuncFromFunc` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 247-261
```cpp
    if (auto specRoutineAttr = srcFunc->getAttrOfType<SpecializedRoutineAttr>(
            getSpecializedRoutineAttrName())) {
      StringAttr funcName = specRoutineAttr.getFuncName();
      if (failed(SymbolTable::replaceAllSymbolUses(
              StringAttr::get(ctx, deviceFuncOp.getName()), funcName, mod))) {
        accSupport.emitNYI(deviceFuncOp.getLoc(),
                           "cannot replace symbol for acc routine");
        return failure();
      }
      deviceFuncOp->setAttr(SymbolTable::getSymbolAttrName(), funcName);
    }
    if (auto specAttr = srcFunc->getAttrOfType<SpecializedRoutineAttr>(
            getSpecializedRoutineAttrName()))
      deviceFuncOp->setAttr(getSpecializedRoutineAttrName(), specAttr);

```
- **EN**: Implements logic around `getAttrOfType`, `getSpecializedRoutineAttrName`, `getFuncName`, `failed`, and 4 more symbols.
- **CN**: 围绕 `getAttrOfType`, `getSpecializedRoutineAttrName`, `getFuncName`, `failed`, and 4 more symbols 实现具体逻辑。

### Lines 262-278
```cpp
    gpuSymTab.insert(deviceFuncOp);
  }
  return success();
}

/// Remove specialized device copies and nohost routines from the host module.
static void
cleanupHostModule(const llvm::SmallSetVector<CloneCandidate, 4> &funcsToClone) {
  for (CloneCandidate candidate : funcsToClone) {
    func::FuncOp funcCandidate = candidate.first;
    RoutineOp routineCandidate = candidate.second;
    if ((routineCandidate && routineCandidate.getNohost()) ||
        acc::isSpecializedAccRoutine(funcCandidate))
      funcCandidate.erase();
  }
}

```
- **EN**: Implements logic around `insert`, `success`, `cleanupHostModule`, `getNohost`, and 2 more symbols.
- **CN**: 围绕 `insert`, `success`, `cleanupHostModule`, `getNohost`, and 2 more symbols 实现具体逻辑。

### Lines 279-292
```cpp
class ACCRoutineToGPUFunc
    : public acc::impl::ACCRoutineToGPUFuncBase<ACCRoutineToGPUFunc> {
public:
  using acc::impl::ACCRoutineToGPUFuncBase<
      ACCRoutineToGPUFunc>::ACCRoutineToGPUFuncBase;

  void runOnOperation() override {
    ModuleOp mod = getOperation();
    if (mod.getOps<RoutineOp>().empty()) {
      LLVM_DEBUG(llvm::dbgs()
                 << "Skipping ACCRoutineToGPUFunc - no acc.routine ops\n");
      return;
    }

```
- **EN**: Introduces declarations for `ACCRoutineToGPUFunc`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ACCRoutineToGPUFunc` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 293-304
```cpp
    OpenACCSupport &accSupport = getAnalysis<OpenACCSupport>();
    std::optional<gpu::GPUModuleOp> gpuModOpt =
        accSupport.getOrCreateGPUModule(mod);
    if (!gpuModOpt) {
      accSupport.emitNYI(mod.getLoc(), "Failed to create GPU module");
      return signalPassFailure();
    }
    gpu::GPUModuleOp gpuMod = *gpuModOpt;

    SymbolTable symTab(mod);
    SymbolTable gpuSymTab(gpuMod);

```
- **EN**: Implements logic around `getAnalysis`, `getOrCreateGPUModule`, `emitNYI`, `signalPassFailure`, and 2 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getAnalysis`, `getOrCreateGPUModule`, `emitNYI`, `signalPassFailure`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 305-318
```cpp
    llvm::SmallSetVector<llvm::StringRef, 4> funcsToCloneCandidates;
    llvm::SmallSetVector<RoutineOp, 4> materializedAccRoutines;
    llvm::SmallSetVector<RoutineOp, 4> bindAccRoutines;

    collectRoutineCandidates(mod, symTab, this->deviceType, accSupport,
                             funcsToCloneCandidates, materializedAccRoutines,
                             bindAccRoutines);

    llvm::SmallSetVector<CloneCandidate, 4> funcsToClone;
    if (failed(processCallsInRoutines(symTab, gpuSymTab, accSupport,
                                      funcsToCloneCandidates,
                                      materializedAccRoutines, funcsToClone)))
      return signalPassFailure();

```
- **EN**: Implements logic around `collectRoutineCandidates`, `failed`, `signalPassFailure`.
- **CN**: 围绕 `collectRoutineCandidates`, `failed`, `signalPassFailure` 实现具体逻辑。

### Lines 319-328
```cpp
    if (failed(cloneFuncsToGPUModule(mod, accSupport, gpuSymTab, funcsToClone)))
      return signalPassFailure();

    cleanupHostModule(funcsToClone);
    for (RoutineOp bindOp : bindAccRoutines)
      bindOp.erase();
  }
};

} // namespace
```
- **EN**: Implements logic around `failed`, `signalPassFailure`, `cleanupHostModule`, `erase`.
- **CN**: 围绕 `failed`, `signalPassFailure`, `cleanupHostModule`, `erase` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/IRMapping.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/CallInterfaces.h`, `llvm/ADT/SetVector.h` ... (+1 more)
- **Standard-library headers / 标准库头文件**: `<string>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
