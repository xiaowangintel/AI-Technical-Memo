# KernelOutlining.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/KernelOutlining.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the GPU dialect kernel outlining pass.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- KernelOutlining.cpp - Implementation of GPU kernel outlining -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the GPU dialect kernel outlining pass.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-30
```cpp

#include "mlir/Dialect/GPU/Transforms/Passes.h"

#include "mlir/AsmParser/AsmParser.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/DLTI/DLTI.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Utils/GPUUtils.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/RegionUtils.h"
#include <limits>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`。

### Lines 31-45
```cpp
namespace mlir {
#define GEN_PASS_DEF_GPULAUNCHSINKINDEXCOMPUTATIONSPASS
#define GEN_PASS_DEF_GPUKERNELOUTLININGPASS
#include "mlir/Dialect/GPU/Transforms/Passes.h.inc"
} // namespace mlir

using namespace mlir;

template <typename OpTy>
static void createForAllDimensions(OpBuilder &builder, Location loc,
                                   SmallVectorImpl<Value> &values) {
  for (auto dim : {gpu::Dimension::x, gpu::Dimension::y, gpu::Dimension::z})
    values.push_back(OpTy::create(builder, loc, builder.getIndexType(), dim));
}

```
- **EN**: Introduces declarations for `mlir`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 46-65
```cpp
/// Adds operations generating block/thread ids and grid/block dimensions at the
/// beginning of the `launchFuncOpBody` region. Add mapping from argument in
/// entry block of `launchOpBody`, to the corresponding result value of the
/// added operations.
static void injectGpuIndexOperations(Location loc, Region &launchFuncOpBody,
                                     Region &launchOpBody, IRMapping &map,
                                     bool hasCluster = false) {
  OpBuilder builder(loc->getContext());
  Block &firstBlock = launchOpBody.front();
  builder.setInsertionPointToStart(&launchFuncOpBody.front());
  SmallVector<Value> indexOps;
  // The order is important here, as it must match the order of the arguments
  createForAllDimensions<gpu::BlockIdOp>(builder, loc, indexOps);
  createForAllDimensions<gpu::ThreadIdOp>(builder, loc, indexOps);
  createForAllDimensions<gpu::GridDimOp>(builder, loc, indexOps);
  createForAllDimensions<gpu::BlockDimOp>(builder, loc, indexOps);
  if (hasCluster) {
    createForAllDimensions<gpu::ClusterIdOp>(builder, loc, indexOps);
    createForAllDimensions<gpu::ClusterDimOp>(builder, loc, indexOps);
  }
```
- **EN**: Implements logic around `injectGpuIndexOperations`, `builder`, `front`, `setInsertionPointToStart`, and 6 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `injectGpuIndexOperations`, `builder`, `front`, `setInsertionPointToStart`, and 6 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 66-79
```cpp
  // Replace the leading 12 function args with the respective thread/block index
  // operations. Iterate backwards since args are erased and indices change.
  for (const auto &indexOp : enumerate(indexOps))
    map.map(firstBlock.getArgument(indexOp.index()), indexOp.value());
}

/// Identifies operations that are beneficial to sink into kernels. These
/// operations may not have side-effects, as otherwise sinking (and hence
/// duplicating them) is not legal.
static bool isLikelyAnIndexComputation(Operation *op) {
  return matchPattern(op, m_Constant()) ||
         isa<memref::DimOp, arith::SelectOp, arith::CmpIOp>(op);
}

```
- **EN**: Implements logic around `enumerate`, `map`, `isLikelyAnIndexComputation`, `matchPattern`, and 1 more symbols; this block moves data between tensor-style values and explicit buffers; encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `enumerate`, `map`, `isLikelyAnIndexComputation`, `matchPattern`, and 1 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 80-89
```cpp
/// For a given operation `op`, computes whether it is beneficial to sink the
/// operation into the kernel. An operation can be sunk if doing so does not
/// introduce new kernel arguments. Whether a value is already available in the
/// kernel (and hence does not introduce new arguments) is checked by
/// querying `existingDependencies` and `availableValues`.
/// If an operand is not yet available, we recursively check whether it can be
/// made available by siking its defining op.
/// Operations that are indentified for sinking are added to `beneficiaryOps` in
/// the order they should appear in the kernel. Furthermore, `availableValues`
/// is updated with results that will be available after sinking the identified
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 90-101
```cpp
/// ops.
static bool extractBeneficiaryOps(
    Operation *op, const SetVector<Value> &existingDependencies,
    SetVector<Operation *> &beneficiaryOps,
    llvm::SmallPtrSetImpl<Value> &availableValues,
    llvm::function_ref<bool(Operation *)> isSinkingBeneficiary) {
  if (beneficiaryOps.count(op))
    return true;

  if (!isSinkingBeneficiary(op))
    return false;

```
- **EN**: Implements logic around `extractBeneficiaryOps`, `function_ref`, `count`, `isSinkingBeneficiary`.
- **CN**: 围绕 `extractBeneficiaryOps`, `function_ref`, `count`, `isSinkingBeneficiary` 实现具体逻辑。

### Lines 102-121
```cpp
  for (Value operand : op->getOperands()) {
    // It is already visible in the kernel, keep going.
    if (availableValues.count(operand))
      continue;
    // Else check whether it can be made available via sinking or already is a
    // dependency.
    Operation *definingOp = operand.getDefiningOp();
    if ((!definingOp || !extractBeneficiaryOps(definingOp, existingDependencies,
                                               beneficiaryOps, availableValues,
                                               isSinkingBeneficiary)) &&
        !existingDependencies.count(operand))
      return false;
  }
  // We will sink the operation, mark its results as now available.
  beneficiaryOps.insert(op);
  for (Value result : op->getResults())
    availableValues.insert(result);
  return true;
}

```
- **EN**: Implements logic around `getOperands`, `count`, `getDefiningOp`, `extractBeneficiaryOps`, and 2 more symbols.
- **CN**: 围绕 `getOperands`, `count`, `getDefiningOp`, `extractBeneficiaryOps`, and 2 more symbols 实现具体逻辑。

### Lines 122-132
```cpp
LogicalResult mlir::sinkOperationsIntoLaunchOp(
    gpu::LaunchOp launchOp,
    llvm::function_ref<bool(Operation *)> isSinkingBeneficiary) {
  assert(isSinkingBeneficiary);
  Region &launchOpBody = launchOp.getBody();

  // Identify uses from values defined outside of the scope of the launch
  // operation.
  SetVector<Value> sinkCandidates;
  getUsedValuesDefinedAbove(launchOpBody, sinkCandidates);

```
- **EN**: Implements logic around `sinkOperationsIntoLaunchOp`, `function_ref`, `assert`, `getBody`, and 1 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `sinkOperationsIntoLaunchOp`, `function_ref`, `assert`, `getBody`, and 1 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 133-142
```cpp
  SetVector<Operation *> toBeSunk;
  llvm::SmallPtrSet<Value, 4> availableValues;
  for (Value operand : sinkCandidates) {
    Operation *operandOp = operand.getDefiningOp();
    if (!operandOp)
      continue;
    extractBeneficiaryOps(operandOp, sinkCandidates, toBeSunk, availableValues,
                          isSinkingBeneficiary);
  }

```
- **EN**: Implements logic around `getDefiningOp`, `extractBeneficiaryOps`.
- **CN**: 围绕 `getDefiningOp`, `extractBeneficiaryOps` 实现具体逻辑。

### Lines 143-155
```cpp
  // Insert operations so that the defs get cloned before uses.
  IRMapping map;
  OpBuilder builder(launchOpBody);
  for (Operation *op : toBeSunk) {
    Operation *clonedOp = builder.clone(*op, map);
    // Only replace uses within the launch op.
    for (auto pair : llvm::zip(op->getResults(), clonedOp->getResults()))
      replaceAllUsesInRegionWith(std::get<0>(pair), std::get<1>(pair),
                                 launchOp.getBody());
  }
  return success();
}

```
- **EN**: Implements logic around `builder`, `clone`, `zip`, `replaceAllUsesInRegionWith`, and 2 more symbols.
- **CN**: 围绕 `builder`, `clone`, `zip`, `replaceAllUsesInRegionWith`, and 2 more symbols 实现具体逻辑。

### Lines 156-173
```cpp
/// Return the provided KernelDim3 as an array of i32 constants if possible.
static DenseI32ArrayAttr maybeConstantDimsAttr(gpu::KernelDim3 dims) {
  SmallVector<int32_t, 3> constants;
  MLIRContext *ctx = dims.x.getContext();
  for (Value v : {dims.x, dims.y, dims.z}) {
    APInt constValue;
    if (!matchPattern(v, m_ConstantInt(&constValue)))
      return nullptr;
    // In the event someone called for a too-large block or grid dimension,
    // don't set bounds as it is likely to cause more confusing behavior.
    if (constValue.ugt(std::numeric_limits<uint32_t>::max()))
      return nullptr;
    constants.push_back(
        constValue.getLimitedValue(std::numeric_limits<uint32_t>::max()));
  }
  return DenseI32ArrayAttr::get(ctx, constants);
}

```
- **EN**: Implements logic around `maybeConstantDimsAttr`, `getContext`, `matchPattern`, `ugt`, and 3 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `maybeConstantDimsAttr`, `getContext`, `matchPattern`, `ugt`, and 3 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 174-185
```cpp
/// Outline the `gpu.launch` operation body into a kernel function. Replace
/// `gpu.terminator` operations by `gpu.return` in the generated function.
/// Set block and grid size bounds if known.
static gpu::GPUFuncOp outlineKernelFuncImpl(gpu::LaunchOp launchOp,
                                            StringRef kernelFnName,
                                            SetVector<Value> &operands) {
  Location loc = launchOp.getLoc();
  // Create a builder with no insertion point, insertion will happen separately
  // due to symbol table manipulation.
  OpBuilder builder(launchOp.getContext());
  Region &launchOpBody = launchOp.getBody();

```
- **EN**: Implements logic around `outlineKernelFuncImpl`, `getLoc`, `builder`, `getBody`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `outlineKernelFuncImpl`, `getLoc`, `builder`, `getBody` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 186-203
```cpp
  // Identify uses from values defined outside of the scope of the launch
  // operation.
  getUsedValuesDefinedAbove(launchOpBody, operands);

  // Create the gpu.func operation.
  SmallVector<Type, 4> kernelOperandTypes;
  kernelOperandTypes.reserve(operands.size());
  for (Value operand : operands) {
    kernelOperandTypes.push_back(operand.getType());
  }
  FunctionType type =
      FunctionType::get(launchOp.getContext(), kernelOperandTypes, {});
  auto outlinedFunc = gpu::GPUFuncOp::create(
      builder, loc, kernelFnName, type,
      TypeRange(ValueRange(launchOp.getWorkgroupAttributionBBArgs())),
      TypeRange(ValueRange(launchOp.getPrivateAttributions())));
  outlinedFunc.setKernel(true);

```
- **EN**: Implements logic around `getUsedValuesDefinedAbove`, `reserve`, `push_back`, `get`, and 3 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getUsedValuesDefinedAbove`, `reserve`, `push_back`, `get`, and 3 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 204-217
```cpp
  // If we can infer bounds on the grid and/or block sizes from the arguments
  // to the launch op, propagate them to the generated kernel. This is safe
  // because multiple launches with the same body are not deduplicated.
  if (auto blockBounds =
          maybeConstantDimsAttr(launchOp.getBlockSizeOperandValues()))
    outlinedFunc.setKnownBlockSizeAttr(blockBounds);
  if (auto gridBounds =
          maybeConstantDimsAttr(launchOp.getGridSizeOperandValues()))
    outlinedFunc.setKnownGridSizeAttr(gridBounds);
  if (auto clusterSize = launchOp.getClusterSizeOperandValues()) {
    if (auto clusterBounds = maybeConstantDimsAttr(*clusterSize))
      outlinedFunc.setKnownClusterSizeAttr(clusterBounds);
  }

```
- **EN**: Implements logic around `maybeConstantDimsAttr`, `setKnownBlockSizeAttr`, `setKnownGridSizeAttr`, `getClusterSizeOperandValues`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `maybeConstantDimsAttr`, `setKnownBlockSizeAttr`, `setKnownGridSizeAttr`, `getClusterSizeOperandValues`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理 MLIR region、block 或控制流边。

### Lines 218-236
```cpp
  IRMapping map;

  // Map the arguments corresponding to the launch parameters like blockIdx,
  // threadIdx, etc. If cluster is present, then we also generate clusterIdx and
  // clusterDim.
  Region &outlinedFuncBody = outlinedFunc.getBody();
  injectGpuIndexOperations(loc, outlinedFuncBody, launchOpBody, map,
                           launchOp.hasClusterSize());

  // Map memory attributions from the LaunOp op to the GPUFuncOp attributions.
  for (const auto &[launchArg, funcArg] :
       llvm::zip(launchOp.getWorkgroupAttributionBBArgs(),
                 outlinedFunc.getWorkgroupAttributionBBArgs()))
    map.map(launchArg, funcArg);
  for (const auto &[launchArg, funcArg] :
       llvm::zip(launchOp.getPrivateAttributions(),
                 outlinedFunc.getPrivateAttributions()))
    map.map(launchArg, funcArg);

```
- **EN**: Implements logic around `getBody`, `injectGpuIndexOperations`, `hasClusterSize`, `zip`, and 3 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `getBody`, `injectGpuIndexOperations`, `hasClusterSize`, `zip`, and 3 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 237-256
```cpp
  // Map arguments from gpu.launch region to the arguments of the gpu.func
  // operation.
  Block &entryBlock = outlinedFuncBody.front();
  for (const auto &operand : enumerate(operands))
    map.map(operand.value(), entryBlock.getArgument(operand.index()));

  // Clone the region of the gpu.launch operation into the gpu.func operation.
  launchOpBody.cloneInto(&outlinedFuncBody, map);

  // Replace the terminator op with returns.
  for (Block &block : launchOpBody) {
    Block *clonedBlock = map.lookup(&block);
    auto terminator = dyn_cast<gpu::TerminatorOp>(clonedBlock->getTerminator());
    if (!terminator)
      continue;
    OpBuilder replacer(terminator);
    gpu::ReturnOp::create(replacer, terminator->getLoc());
    terminator->erase();
  }

```
- **EN**: Implements logic around `front`, `enumerate`, `map`, `cloneInto`, and 5 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `front`, `enumerate`, `map`, `cloneInto`, and 5 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 257-266
```cpp
  // Splice now the entry block of the gpu.launch operation at the end of the
  // gpu.func entry block and erase the redundant block.
  Block *clonedLaunchOpEntry = map.lookup(&launchOpBody.front());
  entryBlock.getOperations().splice(entryBlock.getOperations().end(),
                                    clonedLaunchOpEntry->getOperations());
  clonedLaunchOpEntry->erase();

  return outlinedFunc;
}

```
- **EN**: Implements logic around `lookup`, `getOperations`, `erase`; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `lookup`, `getOperations`, `erase` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 267-280
```cpp
gpu::GPUFuncOp mlir::outlineKernelFunc(gpu::LaunchOp launchOp,
                                       StringRef kernelFnName,
                                       llvm::SmallVectorImpl<Value> &operands) {
  DenseSet<Value> inputOperandSet;
  inputOperandSet.insert_range(operands);
  SetVector<Value> operandSet(llvm::from_range, operands);
  auto funcOp = outlineKernelFuncImpl(launchOp, kernelFnName, operandSet);
  for (auto operand : operandSet) {
    if (!inputOperandSet.count(operand))
      operands.push_back(operand);
  }
  return funcOp;
}

```
- **EN**: Implements logic around `outlineKernelFunc`, `insert_range`, `operandSet`, `outlineKernelFuncImpl`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules; works with symbol tables or function-like operations.
- **CN**: 围绕 `outlineKernelFunc`, `insert_range`, `operandSet`, `outlineKernelFuncImpl`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理符号表或类函数操作。

### Lines 281-300
```cpp
/// Replace `gpu.launch` operations with an `gpu.launch_func` operation
/// launching `kernelFunc`. The kernel func contains the body of the
/// `gpu.launch` with constant region arguments inlined.
static void convertToLaunchFuncOp(gpu::LaunchOp launchOp,
                                  gpu::GPUFuncOp kernelFunc,
                                  ValueRange operands) {
  OpBuilder builder(launchOp);
  // The launch op has an optional dynamic shared memory size. If it doesn't
  // exist, we use zero.
  Value asyncToken = launchOp.getAsyncToken();
  std::optional<gpu::KernelDim3> clusterSize =
      launchOp.getClusterSizeOperandValues();
  auto launchFunc = gpu::LaunchFuncOp::create(
      builder, launchOp.getLoc(), kernelFunc,
      launchOp.getGridSizeOperandValues(), launchOp.getBlockSizeOperandValues(),
      launchOp.getDynamicSharedMemorySize(), operands,
      asyncToken ? asyncToken.getType() : nullptr,
      launchOp.getAsyncDependencies(), clusterSize);
  if (launchOp.getCooperative())
    launchFunc.setCooperative(true);
```
- **EN**: Implements logic around `convertToLaunchFuncOp`, `builder`, `getAsyncToken`, `getClusterSizeOperandValues`, and 8 more symbols; this block encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `convertToLaunchFuncOp`, `builder`, `getAsyncToken`, `getClusterSizeOperandValues`, and 8 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 301-311
```cpp
  launchOp.replaceAllUsesWith(launchFunc);
  launchOp.erase();
}

namespace {
/// Pass that moves ops which are likely an index computation into gpu.launch
/// body.
class GpuLaunchSinkIndexComputationsPass
    : public impl::GpuLaunchSinkIndexComputationsPassBase<
          GpuLaunchSinkIndexComputationsPass> {
public:
```
- **EN**: Introduces declarations for `GpuLaunchSinkIndexComputationsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuLaunchSinkIndexComputationsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 312-325
```cpp
  void runOnOperation() override {
    Operation *op = getOperation();
    if (op->walk([](gpu::LaunchOp launch) {
            // Pull in instructions that can be sunk
            if (failed(sinkOperationsIntoLaunchOp(launch,
                                                  isLikelyAnIndexComputation)))
              return WalkResult::interrupt();

            return WalkResult::advance();
          }).wasInterrupted())
      signalPassFailure();
  }
};

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `walk`, `failed`, and 4 more symbols; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `walk`, `failed`, and 4 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 326-335
```cpp
/// Pass that moves the kernel of each LaunchOp into its separate nested module.
///
/// This pass moves the kernel code of each LaunchOp into a function created
/// inside a nested module. It also creates an external function of the same
/// name in the parent module.
///
/// The gpu.modules are intended to be compiled to a cubin blob independently in
/// a separate pass. The external functions can then be annotated with the
/// symbol of the cubin accessor function.
class GpuKernelOutliningPass
```
- **EN**: Introduces declarations for `GpuKernelOutliningPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuKernelOutliningPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 336-346
```cpp
    : public impl::GpuKernelOutliningPassBase<GpuKernelOutliningPass> {
public:
  using Base::Base;

  LogicalResult initialize(MLIRContext *context) override {
    // Initialize the data layout specification from the data layout string.
    if (!dataLayoutStr.empty()) {
      Attribute resultAttr = mlir::parseAttribute(dataLayoutStr, context);
      if (!resultAttr)
        return failure();

```
- **EN**: Implements logic around `initialize`, `empty`, `parseAttribute`, `failure`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `initialize`, `empty`, `parseAttribute`, `failure` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 347-366
```cpp
      dataLayoutSpec = dyn_cast<DataLayoutSpecInterface>(resultAttr);
      if (!dataLayoutSpec)
        return failure();
    }

    return success();
  }

  void runOnOperation() override {
    SymbolTable symbolTable(getOperation());
    bool modified = false;
    for (auto func : getOperation().getOps<SymbolOpInterface>()) {
      // Insert just after the function.
      Block::iterator insertPt(func->getNextNode());
      auto funcWalkResult = func.walk([&](gpu::LaunchOp op) {
        SetVector<Value> operands;
        std::string kernelFnName;
        if (op.getFunction()) {
          kernelFnName = op.getFunction()->str();
        } else {
```
- **EN**: Implements logic around `dyn_cast`, `failure`, `success`, `runOnOperation`, and 5 more symbols; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `dyn_cast`, `failure`, `success`, `runOnOperation`, and 5 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则，并处理 MLIR region、block 或控制流边。

### Lines 367-384
```cpp
          kernelFnName =
              Twine(op->getParentOfType<SymbolOpInterface>().getName(),
                    "_kernel")
                  .str();
        }

        gpu::GPUFuncOp outlinedFunc =
            outlineKernelFuncImpl(op, kernelFnName, operands);

        // Create nested module and insert outlinedFunc. The module will
        // originally get the same name as the function, but may be renamed on
        // insertion into the parent module.
        FailureOr<gpu::GPUModuleOp> kernelModule =
            createKernelModule(op, outlinedFunc, symbolTable);
        if (failed(kernelModule))
          return WalkResult::interrupt();
        symbolTable.insert(*kernelModule, insertPt);

```
- **EN**: Implements logic around `Twine`, `str`, `outlineKernelFuncImpl`, `createKernelModule`, and 3 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `Twine`, `str`, `outlineKernelFuncImpl`, `createKernelModule`, and 3 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 385-400
```cpp
        // Potentially changes signature, pulling in constants.
        convertToLaunchFuncOp(op, outlinedFunc, operands.getArrayRef());
        modified = true;
        return WalkResult::advance();
      });
      if (funcWalkResult.wasInterrupted())
        return signalPassFailure();
    }

    // If any new module was inserted in this module, annotate this module as
    // a container module.
    if (modified)
      getOperation()->setAttr(gpu::GPUDialect::getContainerModuleAttrName(),
                              UnitAttr::get(&getContext()));
  }

```
- **EN**: Implements logic around `convertToLaunchFuncOp`, `advance`, `wasInterrupted`, `signalPassFailure`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `convertToLaunchFuncOp`, `advance`, `wasInterrupted`, `signalPassFailure`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 401-420
```cpp
private:
  /// Returns a gpu.module containing kernelFunc and all callees (recursive).
  FailureOr<gpu::GPUModuleOp>
  createKernelModule(gpu::LaunchOp gpuLaunchOp, gpu::GPUFuncOp kernelFunc,
                     const SymbolTable &parentSymbolTable) {
    // TODO: This code cannot use an OpBuilder because it must be inserted into
    // a SymbolTable by the caller. SymbolTable needs to be refactored to
    // prevent manual building of Ops with symbols in code using SymbolTables
    // and then this needs to use the OpBuilder.
    auto *context = getOperation().getContext();
    OpBuilder builder(context);
    std::string kernelModuleName;
    gpu::GPUModuleOp kernelModule;
    if (gpuLaunchOp.getModule()) {
      kernelModuleName = gpuLaunchOp.getModule()->str();
      kernelModule =
          parentSymbolTable.lookup<gpu::GPUModuleOp>(kernelModuleName);
    } else {
      kernelModuleName = kernelFunc.getName();
    }
```
- **EN**: Implements logic around `createKernelModule`, `getOperation`, `builder`, `getModule`, and 2 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `createKernelModule`, `getOperation`, `builder`, `getModule`, and 2 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 421-433
```cpp

    // Check if the module already exists in the symbol table
    if (!kernelModule) {
      // If not found, create a new GPU module
      kernelModule = gpu::GPUModuleOp::create(builder, kernelFunc.getLoc(),
                                              kernelModuleName);
    }

    // If a valid data layout spec was provided, attach it to the kernel module.
    // Otherwise, the default data layout will be used.
    if (dataLayoutSpec)
      kernelModule->setAttr(DLTIDialect::kDataLayoutAttrName, dataLayoutSpec);

```
- **EN**: Implements logic around `create`, `setAttr`; this block encodes accelerator-specific execution or lowering rules; works with symbol tables or function-like operations.
- **CN**: 围绕 `create`, `setAttr` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理符号表或类函数操作。

### Lines 434-453
```cpp
    SymbolTable symbolTable(kernelModule);
    symbolTable.insert(kernelFunc);

    SmallVector<Operation *, 8> symbolDefWorklist = {kernelFunc};
    while (!symbolDefWorklist.empty()) {
      if (std::optional<SymbolTable::UseRange> symbolUses =
              SymbolTable::getSymbolUses(symbolDefWorklist.pop_back_val())) {
        for (SymbolTable::SymbolUse symbolUse : *symbolUses) {
          // Nested symbol references (e.g. @M::@F) cannot be resolved inside
          // the kernel module when @M exists in the parent: @M will not be
          // available inside the outlined module after the transformation.
          // Ignore references whose root does not exist in the parent, as those
          // are phantom references (e.g. in unregistered-op attributes) that
          // were already unresolvable and are simply copied as-is.
          if (!symbolUse.getSymbolRef().getNestedReferences().empty() &&
              parentSymbolTable.lookup(
                  symbolUse.getSymbolRef().getRootReference())) {
            symbolUse.getUser()->emitError("nested symbol reference '")
                << symbolUse.getSymbolRef()
                << "' cannot be resolved inside the outlined kernel module; "
```
- **EN**: Implements logic around `symbolTable`, `insert`, `empty`, `getSymbolUses`, and 3 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `symbolTable`, `insert`, `empty`, `getSymbolUses`, and 3 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 454-472
```cpp
                   "gpu-kernel-outlining does not support cross-module symbol "
                   "references inside gpu.launch bodies";
            kernelModule->erase();
            return failure();
          }
          StringAttr symbolName = symbolUse.getSymbolRef().getLeafReference();
          if (symbolTable.lookup(symbolName))
            continue;

          Operation *symbolDef = parentSymbolTable.lookup(symbolName);
          if (!symbolDef)
            continue;
          Operation *symbolDefClone = symbolDef->clone();
          symbolDefWorklist.push_back(symbolDefClone);
          symbolTable.insert(symbolDefClone);
        }
      }
    }

```
- **EN**: Implements logic around `erase`, `failure`, `getSymbolRef`, `lookup`, and 3 more symbols; this block encodes accelerator-specific execution or lowering rules; works with symbol tables or function-like operations.
- **CN**: 围绕 `erase`, `failure`, `getSymbolRef`, `lookup`, and 3 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则，并处理符号表或类函数操作。

### Lines 473-479
```cpp
    return kernelModule;
  }

  DataLayoutSpecInterface dataLayoutSpec;
};

} // namespace
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/AsmParser/AsmParser.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/DLTI/DLTI.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Utils/GPUUtils.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinAttributes.h` ... (+6 more)
- **Standard-library headers / 标准库头文件**: `<limits>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (8), MLIR IR core abstractions / MLIR IR 核心抽象 (5), MLIR support-library helpers / MLIR Support 库辅助功能 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
