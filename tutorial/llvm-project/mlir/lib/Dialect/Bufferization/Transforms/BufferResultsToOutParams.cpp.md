# BufferResultsToOutParams.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Bufferization/Transforms/BufferResultsToOutParams.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for bufferization infrastructure and tensor-to-buffer lowering.
  - **CN**: 实现 Bufferization 基础设施与张量到缓冲区的 lowering 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- BufferResultsToOutParams.cpp - Calling convention conversion -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Bufferization/IR/AllocationOpInterface.h"
#include "mlir/Dialect/Bufferization/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Bufferization/IR/AllocationOpInterface.h`, `mlir/Dialect/Bufferization/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Bufferization/IR/AllocationOpInterface.h`, `mlir/Dialect/Bufferization/Transforms/Passes.h`。

### Lines 12-22
```cpp
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/IR/Operation.h"

namespace mlir {
namespace bufferization {
#define GEN_PASS_DEF_BUFFERRESULTSTOOUTPARAMSPASS
#include "mlir/Dialect/Bufferization/Transforms/Passes.h.inc"
} // namespace bufferization
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `bufferization`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `bufferization` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 23-41
```cpp
using namespace mlir;
using AllocationFn = bufferization::BufferResultsToOutParamsOpts::AllocationFn;
using MemCpyFn = bufferization::BufferResultsToOutParamsOpts::MemCpyFn;
using AllocDynamicSizesMap =
    llvm::DenseMap<func::FuncOp, SmallVector<SmallVector<Value>>>;

/// Return `true` if the given MemRef type has a fully dynamic layout.
static bool hasFullyDynamicLayoutMap(MemRefType type) {
  int64_t offset;
  SmallVector<int64_t, 4> strides;
  if (failed(type.getStridesAndOffset(strides, offset)))
    return false;
  if (!llvm::all_of(strides, ShapedType::isDynamic))
    return false;
  if (ShapedType::isStatic(offset))
    return false;
  return true;
}

```
- **EN**: Implements logic around `hasFullyDynamicLayoutMap`, `failed`, `all_of`, `isStatic`; this block moves data between tensor-style values and explicit buffers; works with symbol tables or function-like operations.
- **CN**: 围绕 `hasFullyDynamicLayoutMap`, `failed`, `all_of`, `isStatic` 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理符号表或类函数操作。

### Lines 42-60
```cpp
/// Return `true` if the given MemRef type has a static identity layout (i.e.,
/// no layout).
static bool hasStaticIdentityLayout(MemRefType type) {
  return type.getLayout().isIdentity();
}

/// Return the dynamic shapes of the `memref` based on the defining op. If the
/// complete dynamic shape fails to be captured, return an empty value.
/// Currently, only function block arguments are supported for capturing.
static SmallVector<Value> getDynamicSize(Value memref, func::FuncOp funcOp) {
  Operation *defOp = memref.getDefiningOp();
  if (!defOp)
    return {};
  auto operands = defOp->getOperands();
  SmallVector<Value> dynamicSizes;
  for (Value size : operands) {
    if (!isa<IndexType>(size.getType()))
      continue;

```
- **EN**: Implements logic around `hasStaticIdentityLayout`, `getLayout`, `getDynamicSize`, `getDefiningOp`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers; manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `hasStaticIdentityLayout`, `getLayout`, `getDynamicSize`, `getDefiningOp`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 61-72
```cpp
    BlockArgument sizeSrc = dyn_cast<BlockArgument>(size);
    if (!sizeSrc)
      return {};
    auto arguments = funcOp.getArguments();
    auto iter = llvm::find(arguments, sizeSrc);
    if (iter == arguments.end())
      return {};
    dynamicSizes.push_back(*iter);
  }
  return dynamicSizes;
}

```
- **EN**: Implements logic around `dyn_cast`, `getArguments`, `find`, `end`, and 1 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `dyn_cast`, `getArguments`, `find`, `end`, and 1 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 73-91
```cpp
/// Returns the dynamic sizes at the callee, through the call relationship
/// between the caller and callee.
static SmallVector<Value> mapDynamicSizeAtCaller(func::CallOp call,
                                                 func::FuncOp callee,
                                                 ValueRange dynamicSizes) {
  SmallVector<Value> mappedDynamicSizes;
  for (Value size : dynamicSizes) {
    for (auto [src, dst] :
         llvm::zip_first(call.getOperands(), callee.getArguments())) {
      if (size != dst)
        continue;
      mappedDynamicSizes.push_back(src);
    }
  }
  assert(mappedDynamicSizes.size() == dynamicSizes.size() &&
         "could not find all dynamic sizes");
  return mappedDynamicSizes;
}

```
- **EN**: Implements logic around `mapDynamicSizeAtCaller`, `zip_first`, `push_back`, `assert`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `mapDynamicSizeAtCaller`, `zip_first`, `push_back`, `assert` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 92-102
```cpp
// Updates the func op and entry block.
//
// Any args appended to the entry block are added to `appendedEntryArgs`.
// If `addResultAttribute` is true, adds the unit attribute `bufferize.result`
// to each newly created function argument.
static LogicalResult
updateFuncOp(func::FuncOp func,
             SmallVectorImpl<BlockArgument> &appendedEntryArgs,
             bool addResultAttribute) {
  auto functionType = func.getFunctionType();

```
- **EN**: Implements logic around `updateFuncOp`, `getFunctionType`; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `updateFuncOp`, `getFunctionType` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 103-120
```cpp
  // Collect information about the results will become appended arguments.
  SmallVector<Type, 6> erasedResultTypes;
  BitVector erasedResultIndices(functionType.getNumResults());
  for (const auto &resultType : llvm::enumerate(functionType.getResults())) {
    if (auto memrefType = dyn_cast<MemRefType>(resultType.value())) {
      if (!hasStaticIdentityLayout(memrefType) &&
          !hasFullyDynamicLayoutMap(memrefType)) {
        // Only buffers with static identity layout can be allocated. These can
        // be casted to memrefs with fully dynamic layout map. Other layout maps
        // are not supported.
        return func->emitError()
               << "cannot create out param for result with unsupported layout";
      }
      erasedResultIndices.set(resultType.index());
      erasedResultTypes.push_back(memrefType);
    }
  }

```
- **EN**: Implements logic around `erasedResultIndices`, `enumerate`, `dyn_cast`, `hasStaticIdentityLayout`, and 4 more symbols.
- **CN**: 围绕 `erasedResultIndices`, `enumerate`, `dyn_cast`, `hasStaticIdentityLayout`, and 4 more symbols 实现具体逻辑。

### Lines 121-138
```cpp
  // Add the new arguments to the function type.
  auto newArgTypes = llvm::to_vector<6>(
      llvm::concat<const Type>(functionType.getInputs(), erasedResultTypes));
  auto newFunctionType = FunctionType::get(func.getContext(), newArgTypes,
                                           functionType.getResults());
  func.setType(newFunctionType);

  // Transfer the result attributes to arg attributes.
  auto erasedIndicesIt = erasedResultIndices.set_bits_begin();
  for (int i = 0, e = erasedResultTypes.size(); i < e; ++i, ++erasedIndicesIt) {
    func.setArgAttrs(functionType.getNumInputs() + i,
                     func.getResultAttrs(*erasedIndicesIt));
    if (addResultAttribute)
      func.setArgAttr(functionType.getNumInputs() + i,
                      StringAttr::get(func.getContext(), "bufferize.result"),
                      UnitAttr::get(func.getContext()));
  }

```
- **EN**: Implements logic around `to_vector`, `Type>`, `get`, `getResults`, and 6 more symbols.
- **CN**: 围绕 `to_vector`, `Type>`, `get`, `getResults`, and 6 more symbols 实现具体逻辑。

### Lines 139-149
```cpp
  // Erase the results.
  if (failed(func.eraseResults(erasedResultIndices)))
    return failure();

  // Add the new arguments to the entry block if the function is not external.
  if (func.isExternal())
    return success();
  Location loc = func.getLoc();
  for (Type type : erasedResultTypes)
    appendedEntryArgs.push_back(func.front().addArgument(type, loc));

```
- **EN**: Implements logic around `failed`, `failure`, `isExternal`, `success`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `failed`, `failure`, `isExternal`, `success`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 150-169
```cpp
  return success();
}

// Updates all ReturnOps in the scope of the given func::FuncOp by either
// keeping them as return values or copying the associated buffer contents into
// the given out-params.
static LogicalResult
updateReturnOps(func::FuncOp func, ArrayRef<BlockArgument> appendedEntryArgs,
                AllocDynamicSizesMap &map,
                const bufferization::BufferResultsToOutParamsOpts &options) {
  auto res = func.walk([&](func::ReturnOp op) {
    SmallVector<Value, 6> copyIntoOutParams;
    SmallVector<Value, 6> keepAsReturnOperands;
    for (Value operand : op.getOperands()) {
      if (isa<MemRefType>(operand.getType()))
        copyIntoOutParams.push_back(operand);
      else
        keepAsReturnOperands.push_back(operand);
    }
    OpBuilder builder(op);
```
- **EN**: Implements logic around `success`, `updateReturnOps`, `walk`, `getOperands`, and 3 more symbols; this block moves data between tensor-style values and explicit buffers; works with symbol tables or function-like operations.
- **CN**: 围绕 `success`, `updateReturnOps`, `walk`, `getOperands`, and 3 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理符号表或类函数操作。

### Lines 170-189
```cpp
    SmallVector<SmallVector<Value>> dynamicSizes;
    for (auto [orig, arg] : llvm::zip(copyIntoOutParams, appendedEntryArgs)) {
      bool hoistStaticAllocs =
          options.hoistStaticAllocs &&
          cast<MemRefType>(orig.getType()).hasStaticShape();
      bool hoistDynamicAllocs =
          options.hoistDynamicAllocs &&
          !cast<MemRefType>(orig.getType()).hasStaticShape();
      if ((hoistStaticAllocs || hoistDynamicAllocs) &&
          isa_and_nonnull<bufferization::AllocationOpInterface>(
              orig.getDefiningOp())) {
        orig.replaceAllUsesWith(arg);
        if (hoistDynamicAllocs) {
          SmallVector<Value> dynamicSize = getDynamicSize(orig, func);
          dynamicSizes.push_back(dynamicSize);
        }
        orig.getDefiningOp()->erase();
      } else {
        if (failed(options.memCpyFn(builder, op.getLoc(), orig, arg)))
          return WalkResult::interrupt();
```
- **EN**: Implements logic around `zip`, `cast`, `AllocationOpInterface>`, `getDefiningOp`, and 5 more symbols.
- **CN**: 围绕 `zip`, `cast`, `AllocationOpInterface>`, `getDefiningOp`, and 5 more symbols 实现具体逻辑。

### Lines 190-202
```cpp
      }
    }
    func::ReturnOp::create(builder, op.getLoc(), keepAsReturnOperands);
    op.erase();
    auto dynamicSizePair =
        std::pair<func::FuncOp, SmallVector<SmallVector<Value>>>(func,
                                                                 dynamicSizes);
    map.insert(dynamicSizePair);
    return WalkResult::advance();
  });
  return failure(res.wasInterrupted());
}

```
- **EN**: Implements logic around `create`, `erase`, `SmallVector`, `insert`, and 2 more symbols; this block works with symbol tables or function-like operations; uses explicit diagnostics or recoverable failure signaling.
- **CN**: 围绕 `create`, `erase`, `SmallVector`, `insert`, and 2 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作，并使用显式诊断或可恢复失败信号。

### Lines 203-222
```cpp
// Updates all CallOps in the scope of the given ModuleOp by allocating
// temporary buffers for newly introduced out params.
static LogicalResult
updateCalls(ModuleOp module, const AllocDynamicSizesMap &map,
            const bufferization::BufferResultsToOutParamsOpts &options) {
  bool didFail = false;
  SymbolTable symtab(module);
  module.walk([&](func::CallOp op) {
    auto callee = symtab.lookup<func::FuncOp>(op.getCallee());
    if (!callee) {
      op.emitError() << "cannot find callee '" << op.getCallee() << "' in "
                     << "symbol table";
      didFail = true;
      return;
    }
    if (!options.filterFn(&callee))
      return;
    if (callee.isPublic() && !options.modifyPublicFunctions)
      return;
    if (callee.isExternal())
```
- **EN**: Implements logic around `updateCalls`, `symtab`, `walk`, `FuncOp>`, and 4 more symbols; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `updateCalls`, `symtab`, `walk`, `FuncOp>`, and 4 more symbols 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 223-242
```cpp
      return;

    SmallVector<Value, 6> replaceWithNewCallResults;
    SmallVector<Value, 6> replaceWithOutParams;
    for (OpResult result : op.getResults()) {
      if (isa<MemRefType>(result.getType()))
        replaceWithOutParams.push_back(result);
      else
        replaceWithNewCallResults.push_back(result);
    }
    SmallVector<Value, 6> outParams;
    OpBuilder builder(op);
    SmallVector<SmallVector<Value>> dynamicSizes = map.lookup(callee);
    size_t dynamicSizesIndex = 0;
    for (Value memref : replaceWithOutParams) {
      SmallVector<Value> dynamicSize = dynamicSizes.size() > dynamicSizesIndex
                                           ? dynamicSizes[dynamicSizesIndex]
                                           : SmallVector<Value>();
      bool memrefStaticShape =
          cast<MemRefType>(memref.getType()).hasStaticShape();
```
- **EN**: Implements logic around `getResults`, `isa`, `push_back`, `builder`, and 4 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `getResults`, `isa`, `push_back`, `builder`, and 4 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 243-253
```cpp
      if (!memrefStaticShape && dynamicSize.empty()) {
        op.emitError()
            << "cannot create out param for dynamically shaped result";
        didFail = true;
        return;
      }
      auto memrefType = cast<MemRefType>(memref.getType());
      auto allocType =
          MemRefType::get(memrefType.getShape(), memrefType.getElementType(),
                          AffineMap(), memrefType.getMemorySpace());

```
- **EN**: Implements logic around `empty`, `emitError`, `cast`, `get`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `empty`, `emitError`, `cast`, `get`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并在张量风格值与显式缓冲区之间移动数据。

### Lines 254-273
```cpp
      if (memrefStaticShape) {
        dynamicSize = {};
      } else {
        ++dynamicSizesIndex;
        dynamicSize = mapDynamicSizeAtCaller(op, callee, dynamicSize);
      }
      auto maybeOutParam =
          options.allocationFn(builder, op.getLoc(), allocType, dynamicSize);
      if (failed(maybeOutParam)) {
        op.emitError() << "failed to create allocation op";
        didFail = true;
        return;
      }
      Value outParam = maybeOutParam.value();
      if (!hasStaticIdentityLayout(memrefType)) {
        // Layout maps are already checked in `updateFuncOp`.
        assert(hasFullyDynamicLayoutMap(memrefType) &&
               "layout map not supported");
        outParam =
            memref::CastOp::create(builder, op.getLoc(), memrefType, outParam);
```
- **EN**: Implements logic around `mapDynamicSizeAtCaller`, `allocationFn`, `failed`, `emitError`, and 4 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `mapDynamicSizeAtCaller`, `allocationFn`, `failed`, `emitError`, and 4 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 274-289
```cpp
      }
      memref.replaceAllUsesWith(outParam);
      outParams.push_back(outParam);
    }

    auto newOperands = llvm::to_vector<6>(op.getOperands());
    newOperands.append(outParams.begin(), outParams.end());
    auto newResultTypes = llvm::map_to_vector<6>(
        replaceWithNewCallResults, [](Value v) { return v.getType(); });
    auto newCall = func::CallOp::create(
        builder, op.getLoc(), op.getCalleeAttr(), newResultTypes, newOperands);
    for (auto t : llvm::zip(replaceWithNewCallResults, newCall.getResults()))
      std::get<0>(t).replaceAllUsesWith(std::get<1>(t));
    op.erase();
  });

```
- **EN**: Implements logic around `replaceAllUsesWith`, `push_back`, `to_vector`, `append`, and 7 more symbols; this block moves data between tensor-style values and explicit buffers; works with symbol tables or function-like operations.
- **CN**: 围绕 `replaceAllUsesWith`, `push_back`, `to_vector`, `append`, and 7 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理符号表或类函数操作。

### Lines 290-309
```cpp
  return failure(didFail);
}

LogicalResult mlir::bufferization::promoteBufferResultsToOutParams(
    ModuleOp module,
    const bufferization::BufferResultsToOutParamsOpts &options) {
  // It maps the shape source of the dynamic shape memref returned by each
  // function.
  AllocDynamicSizesMap map;
  for (auto func : module.getOps<func::FuncOp>()) {
    if (func.isPublic() && !options.modifyPublicFunctions)
      continue;
    if (func.isExternal())
      continue;
    if (!options.filterFn(&func))
      continue;
    SmallVector<BlockArgument, 6> appendedEntryArgs;
    if (failed(
            updateFuncOp(func, appendedEntryArgs, options.addResultAttribute)))
      return failure();
```
- **EN**: Implements logic around `failure`, `promoteBufferResultsToOutParams`, `FuncOp>`, `isPublic`, and 4 more symbols; this block moves data between tensor-style values and explicit buffers; works with symbol tables or function-like operations; uses explicit diagnostics or recoverable failure signaling.
- **CN**: 围绕 `failure`, `promoteBufferResultsToOutParams`, `FuncOp>`, `isPublic`, and 4 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据，并处理符号表或类函数操作，并使用显式诊断或可恢复失败信号。

### Lines 310-319
```cpp
    if (failed(updateReturnOps(func, appendedEntryArgs, map, options))) {
      return failure();
    }
  }
  if (failed(updateCalls(module, map, options)))
    return failure();
  return success();
}

namespace {
```
- **EN**: Implements logic around `failed`, `failure`, `success`.
- **CN**: 围绕 `failed`, `failure`, `success` 实现具体逻辑。

### Lines 320-335
```cpp
struct BufferResultsToOutParamsPass
    : bufferization::impl::BufferResultsToOutParamsPassBase<
          BufferResultsToOutParamsPass> {
  using Base::Base;

  void runOnOperation() override {
    // Convert from pass options in tablegen to BufferResultsToOutParamsOpts.
    if (addResultAttribute)
      options.addResultAttribute = true;
    if (hoistStaticAllocs)
      options.hoistStaticAllocs = true;
    if (hoistDynamicAllocs)
      options.hoistDynamicAllocs = true;
    if (modifyPublicFunctions)
      options.modifyPublicFunctions = true;

```
- **EN**: Introduces declarations for `BufferResultsToOutParamsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `BufferResultsToOutParamsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 336-344
```cpp
    if (failed(bufferization::promoteBufferResultsToOutParams(getOperation(),
                                                              options)))
      return signalPassFailure();
  }

private:
  bufferization::BufferResultsToOutParamsOpts options;
};
} // namespace
```
- **EN**: Implements logic around `failed`, `signalPassFailure`.
- **CN**: 围绕 `failed`, `signalPassFailure` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Bufferization / 缓冲区化**:
  - **EN**: Bridges tensor-style IR to explicit memory buffers and ownership-aware updates.
  - **CN**: 在张量风格 IR 与显式内存缓冲区、所有权感知更新之间建立桥接。
- **Tensor/buffer boundary / 张量/缓冲区边界**:
  - **EN**: Tracks how abstract tensor values are converted into explicit memory effects and memref-based IR.
  - **CN**: 跟踪抽象张量值如何转换成显式内存效应与基于 memref 的 IR。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Bufferization/IR/AllocationOpInterface.h`, `mlir/Dialect/Bufferization/Transforms/Passes.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/IR/Operation.h`, `mlir/Dialect/Bufferization/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
