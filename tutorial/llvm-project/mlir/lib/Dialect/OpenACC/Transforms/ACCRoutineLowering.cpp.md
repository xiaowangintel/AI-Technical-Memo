# ACCRoutineLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCRoutineLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass handles `acc routine` directive by creating specialized functions with appropriate parallelism information that can be used for eventual creation of device function.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ACCRoutineLowering.cpp - Wrap ACC routines in compute_region -------===//
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
// This pass handles `acc routine` directive by creating specialized
// functions with appropriate parallelism information that can be used for
// eventual creation of device function.
//
// Overview:
// ---------
// For each acc.routine that is not bound by name, the pass creates a new
// function (the "device" copy) whose body is a single acc.compute_region
// containing a clone of the original (host) function body. Parallelism is
// expressed by one acc.par_width derived from the routine's clauses (seq,
// vector, worker, gang). The device copy created is simply a staging
// place for eventual move to device module level function.
//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
// Transformations:
// ----------------
// 1. Device function: Same signature as the host; attributes copied except
//    acc.routine_info. The acc.specialized_routine attribute is set with the
//    routine symbol, par level, and original function name.
//
// 2. Body: One acc.par_width, one acc.compute_region that clones the host
//    body. Multi-block host bodies are wrapped in scf.execute_region inside
//    the compute_region.
//
// 3. Finalization: acc.routine's func_name is updated to the device function.
//    For nohost routines, all uses of the host symbol are replaced with the
//    device symbol and the host function is erased.
//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-49
```cpp
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/OpenACC/OpenACCParMapping.h"
#include "mlir/Dialect/OpenACC/OpenACCUtilsCG.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/IR/Value.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCParMapping.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCParMapping.h`。

### Lines 50-56
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCROUTINELOWERING
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 57-63
```cpp
#define DEBUG_TYPE "acc-routine-lowering"

using namespace mlir;
using namespace mlir::acc;

namespace {

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 64-77
```cpp
/// Compute the ParLevel from an acc.routine op for specialization.
static ParLevel computeParLevel(RoutineOp routineOp, DeviceType deviceType) {
  auto gangDim = routineOp.getGangDimValue(deviceType);
  if (!gangDim)
    gangDim = routineOp.getGangDimValue();
  if (gangDim) {
    switch (*gangDim) {
    case 1:
      return ParLevel::gang_dim1;
    case 2:
      return ParLevel::gang_dim2;
    case 3:
      return ParLevel::gang_dim3;
    default:
```
- **EN**: Implements logic around `computeParLevel`, `getGangDimValue`.
- **CN**: 围绕 `computeParLevel`, `getGangDimValue` 实现具体逻辑。

### Lines 78-89
```cpp
      break;
    }
  }
  if (routineOp.hasGang(deviceType) || routineOp.hasGang())
    return ParLevel::gang_dim1;
  if (routineOp.hasWorker(deviceType) || routineOp.hasWorker())
    return ParLevel::worker;
  if (routineOp.hasVector(deviceType) || routineOp.hasVector())
    return ParLevel::vector;
  return ParLevel::seq;
}

```
- **EN**: Implements logic around `hasGang`, `hasWorker`, `hasVector`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasGang`, `hasWorker`, `hasVector` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 90-100
```cpp
/// Collect return operands from the function (first block with func.return).
static void getReturnValues(func::FuncOp func, SmallVectorImpl<Value> &result) {
  result.clear();
  for (Block &block : func.getBody().getBlocks()) {
    if (auto returnOp = dyn_cast<func::ReturnOp>(block.getTerminator())) {
      result.assign(returnOp.operand_begin(), returnOp.operand_end());
      break;
    }
  }
}

```
- **EN**: Implements logic around `getReturnValues`, `clear`, `getBody`, `ReturnOp>`, and 1 more symbols.
- **CN**: 围绕 `getReturnValues`, `clear`, `getBody`, `ReturnOp>`, and 1 more symbols 实现具体逻辑。

### Lines 101-114
```cpp
/// Create the device function with the same signature as the host, set
/// specialized_routine, and add a single block with the same block arguments.
static func::FuncOp createFunctionForDeviceStaging(func::FuncOp hostFunc,
                                                   RoutineOp routineOp,
                                                   ParLevel parLevel,
                                                   MLIRContext *ctx,
                                                   IRRewriter &rewriter) {
  Location loc = hostFunc.getLoc();
  FunctionType funcType = hostFunc.getFunctionType();
  func::FuncOp deviceFunc =
      func::FuncOp::create(rewriter, loc, hostFunc.getName(), funcType);
  deviceFunc->setAttrs(hostFunc->getAttrs());
  deviceFunc->removeAttr(getRoutineInfoAttrName());
  deviceFunc->setAttr(getSpecializedRoutineAttrName(),
```
- **EN**: Implements logic around `createFunctionForDeviceStaging`, `getLoc`, `getFunctionType`, `create`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createFunctionForDeviceStaging`, `getLoc`, `getFunctionType`, `create`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 115-124
```cpp
                      SpecializedRoutineAttr::get(
                          ctx, SymbolRefAttr::get(ctx, routineOp.getSymName()),
                          ParLevelAttr::get(ctx, parLevel),
                          StringAttr::get(ctx, hostFunc.getName())));

  Block *sourceBlock = &hostFunc.getBody().front();
  Block *newBlock = rewriter.createBlock(&deviceFunc.getRegion());
  for (BlockArgument arg : sourceBlock->getArguments())
    newBlock->addArgument(arg.getType(), hostFunc.getLoc());

```
- **EN**: Implements logic around `get`, `getBody`, `createBlock`, `getArguments`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `get`, `getBody`, `createBlock`, `getArguments`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 125-138
```cpp
  return deviceFunc;
}

/// Fill the device function body: one acc.par_width, one acc.compute_region
/// (cloning the host body with inputArgsToMap), then func.return.
static LogicalResult
buildRoutineBody(func::FuncOp deviceFunc, func::FuncOp hostFunc,
                 ArrayRef<Value> funcReturnVals, ParLevel parLevel,
                 DefaultACCToGPUMappingPolicy &policy, IRRewriter &rewriter) {
  Block *newBlock = &deviceFunc.getBody().front();
  Block *sourceBlock = &hostFunc.getBody().front();
  Location loc = hostFunc.getLoc();
  MLIRContext *ctx = rewriter.getContext();

```
- **EN**: Implements logic around `buildRoutineBody`, `getBody`, `getLoc`, `getContext`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `buildRoutineBody`, `getBody`, `getLoc`, `getContext` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 139-152
```cpp
  rewriter.setInsertionPointToStart(newBlock);
  GPUParallelDimAttr parDim = policy.map(ctx, parLevel);
  Value parWidthVal = ParWidthOp::create(rewriter, loc, Value(), parDim);
  SmallVector<Value, 4> inputArgs(newBlock->getArguments().begin(),
                                  newBlock->getArguments().end());

  // Normally the region passed to buildComputeRegion is something in the
  // current function. Here we pass the body of the original (host) function as
  // an optimization to avoid cloning twice (once for a staged device copy and
  // again when creating the compute region). Since we clone only once, we must
  // also provide the original function's arguments so the mapping is correct
  // when cloning the body.
  ValueRange sourceArgsToMap = sourceBlock->getArguments();

```
- **EN**: Implements logic around `setInsertionPointToStart`, `map`, `create`, `inputArgs`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper; uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPointToStart`, `map`, `create`, `inputArgs`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并使用重写模式基础设施变换操作。

### Lines 153-162
```cpp
  IRMapping mapping;
  rewriter.setInsertionPointAfter(parWidthVal.getDefiningOp());
  ComputeRegionOp computeRegion = buildComputeRegion(
      loc, {parWidthVal}, inputArgs, RoutineOp::getOperationName(),
      hostFunc.getBody(), rewriter, mapping,
      /*output=*/funcReturnVals, /*kernelFuncName=*/{},
      /*kernelModuleName=*/{}, /*stream=*/{}, sourceArgsToMap);
  if (!computeRegion)
    return failure();

```
- **EN**: Implements logic around `setInsertionPointAfter`, `buildComputeRegion`, `getOperationName`, `getBody`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPointAfter`, `buildComputeRegion`, `getOperationName`, `getBody`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 163-171
```cpp
  rewriter.setInsertionPointAfter(computeRegion);
  if (funcReturnVals.empty())
    func::ReturnOp::create(rewriter, loc);
  else
    func::ReturnOp::create(rewriter, loc, computeRegion.getResults());

  return success();
}

```
- **EN**: Implements logic around `setInsertionPointAfter`, `empty`, `create`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPointAfter`, `empty`, `create`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 172-180
```cpp
/// Update acc.routine refs and optionally erase host for nohost routines.
static LogicalResult finalizeRoutines(
    SmallVectorImpl<std::tuple<func::FuncOp, func::FuncOp, RoutineOp>>
        &accRoutineInfo,
    ModuleOp mod, MLIRContext *ctx) {
  for (auto &[hostFunc, deviceFunc, routineOp] : accRoutineInfo) {
    routineOp.setFuncNameAttr(SymbolRefAttr::get(ctx, deviceFunc.getName()));
    routineOp->moveBefore(deviceFunc);

```
- **EN**: Implements logic around `finalizeRoutines`, `setFuncNameAttr`, `moveBefore`.
- **CN**: 围绕 `finalizeRoutines`, `setFuncNameAttr`, `moveBefore` 实现具体逻辑。

### Lines 181-193
```cpp
    if (routineOp.getNohost()) {
      if (failed(SymbolTable::replaceAllSymbolUses(
              StringAttr::get(ctx, hostFunc.getName()),
              StringAttr::get(ctx, deviceFunc.getName()), mod))) {
        routineOp.emitError("cannot replace symbol uses for acc routine");
        return failure();
      }
      hostFunc->erase();
    }
  }
  return success();
}

```
- **EN**: Implements logic around `getNohost`, `failed`, `get`, `emitError`, and 3 more symbols.
- **CN**: 围绕 `getNohost`, `failed`, `get`, `emitError`, and 3 more symbols 实现具体逻辑。

### Lines 194-206
```cpp
class ACCRoutineLowering
    : public acc::impl::ACCRoutineLoweringBase<ACCRoutineLowering> {
public:
  using ACCRoutineLoweringBase::ACCRoutineLoweringBase;

  void runOnOperation() override {
    ModuleOp mod = getOperation();
    if (mod.getOps<RoutineOp>().empty()) {
      LLVM_DEBUG(llvm::dbgs()
                 << "Skipping ACCRoutineLowering - no acc.routine ops\n");
      return;
    }

```
- **EN**: Introduces declarations for `ACCRoutineLowering`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ACCRoutineLowering` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 207-215
```cpp
    SymbolTable symTab(mod);
    MLIRContext *ctx = mod.getContext();
    IRRewriter rewriter(ctx);
    DefaultACCToGPUMappingPolicy policy;

    // Tuple: host function, device function, routine operation
    SmallVector<std::tuple<func::FuncOp, func::FuncOp, RoutineOp>, 4>
        accRoutineInfo;

```
- **EN**: Implements logic around `symTab`, `getContext`, `rewriter`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `symTab`, `getContext`, `rewriter` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 216-229
```cpp
    for (RoutineOp routineOp : mod.getOps<RoutineOp>()) {
      if (routineOp.getBindNameValue() ||
          routineOp.getBindNameValue(deviceType))
        continue;

      func::FuncOp hostFunc = symTab.lookup<func::FuncOp>(
          routineOp.getFuncName().getLeafReference());
      if (!hostFunc) {
        routineOp.emitError("acc routine function not found in symbol table");
        return signalPassFailure();
      }
      if (hostFunc.isExternal())
        continue;

```
- **EN**: Implements logic around `getOps`, `getBindNameValue`, `FuncOp>`, `getFuncName`, and 3 more symbols.
- **CN**: 围绕 `getOps`, `getBindNameValue`, `FuncOp>`, `getFuncName`, and 3 more symbols 实现具体逻辑。

### Lines 230-240
```cpp
      SmallVector<Value, 4> funcReturnVals;
      getReturnValues(hostFunc, funcReturnVals);

      OpBuilder::InsertionGuard guard(rewriter);
      ParLevel parLevel = computeParLevel(routineOp, deviceType);
      func::FuncOp deviceFunc = createFunctionForDeviceStaging(
          hostFunc, routineOp, parLevel, ctx, rewriter);
      if (failed(buildRoutineBody(deviceFunc, hostFunc, funcReturnVals,
                                  parLevel, policy, rewriter)))
        return signalPassFailure();

```
- **EN**: Implements logic around `getReturnValues`, `guard`, `computeParLevel`, `createFunctionForDeviceStaging`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getReturnValues`, `guard`, `computeParLevel`, `createFunctionForDeviceStaging`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 241-249
```cpp
      accRoutineInfo.push_back({hostFunc, deviceFunc, routineOp});
      symTab.insert(deviceFunc);
    }

    if (failed(finalizeRoutines(accRoutineInfo, mod, ctx)))
      return signalPassFailure();
  }
};

```
- **EN**: Implements logic around `push_back`, `insert`, `failed`, `signalPassFailure`.
- **CN**: 围绕 `push_back`, `insert`, `failed`, `signalPassFailure` 实现具体逻辑。

### Lines 250-250
```cpp
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
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCParMapping.h`, `mlir/Dialect/OpenACC/OpenACCUtilsCG.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/IRMapping.h`, `mlir/IR/PatternMatch.h`, `mlir/IR/SymbolTable.h`, `mlir/IR/Value.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (5)
