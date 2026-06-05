# BufferizableOpInterfaceImpl.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/BufferizableOpInterfaceImpl.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `BufferizableOpInterfaceImpl`.
  - **CN**: 实现 Linalg 方言中围绕 `BufferizableOpInterfaceImpl` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- BufferizableOpInterfaceImpl.cpp - Impl. of BufferizableOpInterface -===//
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

#include "mlir/Dialect/Linalg/Transforms/BufferizableOpInterfaceImpl.h"
#include "mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h"
#include "mlir/Dialect/Bufferization/IR/DstBufferizableOpInterfaceImpl.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"
#include "mlir/IR/Dialect.h"
#include "mlir/IR/Operation.h"
#include "mlir/Interfaces/DestinationStyleOpInterface.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/BufferizableOpInterfaceImpl.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/Dialect/Bufferization/IR/DstBufferizableOpInterfaceImpl.h`, `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/BufferizableOpInterfaceImpl.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/Dialect/Bufferization/IR/DstBufferizableOpInterfaceImpl.h`, `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 18-24
```cpp
using namespace mlir;
using namespace linalg;
using namespace mlir::bufferization;

namespace {

/// Generic conversion for any DestinationStyleOpInterface on tensors.
```
- **EN**: Introduces declarations for `mlir`, `linalg`, `mlir::bufferization`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `linalg`, `mlir::bufferization` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 25-31
```cpp
static LogicalResult bufferizeDestinationStyleOpInterface(
    RewriterBase &rewriter, DestinationStyleOpInterface op,
    const BufferizationOptions &options, const BufferizationState &state) {
  // Take a guard before anything else.
  OpBuilder::InsertionGuard g(rewriter);
  rewriter.setInsertionPoint(op);

```
- **EN**: Implements logic around `bufferizeDestinationStyleOpInterface`, `g`, `setInsertionPoint`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `bufferizeDestinationStyleOpInterface`, `g`, `setInsertionPoint` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 32-40
```cpp
  // Nothing to do. This op is already bufferized.
  if (op.hasPureBufferSemantics())
    return success();

  // Ensure op has only tensors. Allow mixed tensor-buffer mode on a per-need
  // basis.
  if (!op.hasPureTensorSemantics())
    return op->emitError() << "op does not have pure tensor semantics";

```
- **EN**: Implements logic around `hasPureBufferSemantics`, `success`, `hasPureTensorSemantics`, `emitError`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureBufferSemantics`, `success`, `hasPureTensorSemantics`, `emitError` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 41-54
```cpp
  // New input operands for the cloned op.
  SmallVector<Value> newInputBuffers;
  newInputBuffers.reserve(op.getNumDpsInputs());
  for (OpOperand *opOperand : op.getDpsInputOperands()) {
    if (op.isScalar(opOperand)) {
      newInputBuffers.push_back(opOperand->get());
      continue;
    }
    FailureOr<Value> buffer =
        getBuffer(rewriter, opOperand->get(), options, state);
    if (failed(buffer))
      return failure();
    newInputBuffers.push_back(*buffer);
  }
```
- **EN**: Implements logic around `reserve`, `getDpsInputOperands`, `isScalar`, `push_back`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `reserve`, `getDpsInputOperands`, `isScalar`, `push_back`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 55-66
```cpp

  // New output operands for the cloned op.
  SmallVector<Value> newOutputBuffers;
  for (OpResult opResult : op->getOpResults()) {
    OpOperand *opOperand = op.getDpsInitOperand(opResult.getResultNumber());
    FailureOr<Value> resultBuffer =
        getBuffer(rewriter, opOperand->get(), options, state);
    if (failed(resultBuffer))
      return failure();
    newOutputBuffers.push_back(*resultBuffer);
  }

```
- **EN**: Implements logic around `getOpResults`, `getDpsInitOperand`, `getBuffer`, `failed`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getOpResults`, `getDpsInitOperand`, `getBuffer`, `failed`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 67-80
```cpp
  // Merge input/output operands.
  SmallVector<Value> newOperands = newInputBuffers;
  newOperands.append(newOutputBuffers.begin(), newOutputBuffers.end());

  // Set insertion point now that potential alloc/dealloc are introduced.
  rewriter.setInsertionPoint(op);
  // Clone the op, but use the new operands. Move the existing block into the
  // new op. Since the new op does not have any tensor results, it does not
  // return anything.
  assert(op->getNumRegions() == 1 && "expected that op has 1 region");
  OperationState opState(op->getLoc(), op->getName(), newOperands, TypeRange{},
                         op->getAttrs());
  opState.addRegion();
  Operation *newOp = Operation::create(opState);
```
- **EN**: Implements logic around `append`, `setInsertionPoint`, `assert`, `opState`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `append`, `setInsertionPoint`, `assert`, `opState`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 81-87
```cpp
  newOp->getRegion(0).getBlocks().splice(newOp->getRegion(0).begin(),
                                         op->getRegion(0).getBlocks());

  // We don't want the rewriter tracks an incomplete operation, so insert new
  // operation after op was fully constructed.
  rewriter.insert(newOp);

```
- **EN**: Implements logic around `getRegion`, `insert`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getRegion`, `insert` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 88-94
```cpp
  // Replace the results of the old op with the new output buffers.
  replaceOpWithBufferizedValues(rewriter, op, newOutputBuffers);

  return success();
}

/// Bufferization of linalg.generic. Replace with a new linalg.generic that
```
- **EN**: Implements logic around `replaceOpWithBufferizedValues`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceOpWithBufferizedValues`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 95-106
```cpp
/// operates entirely on memrefs.
template <typename OpTy>
struct LinalgOpInterface
    : public DstBufferizableOpInterfaceExternalModel<LinalgOpInterface<OpTy>,
                                                     OpTy> {
  bool bufferizesToMemoryRead(Operation *op, OpOperand &opOperand,
                              const AnalysisState &state) const {
    // Operand is read if it is used in the computation.
    auto linalgOp = cast<linalg::LinalgOp>(op);
    return linalgOp.payloadUsesValueFromOperand(&opOperand);
  }

```
- **EN**: Introduces declarations for `LinalgOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 107-113
```cpp
  bool bufferizesToMemoryWrite(Operation *op, OpOperand &opOperand,
                               const AnalysisState &state) const {
    // Operand is written to if it is not an input/init.
    auto dpsOp = cast<DestinationStyleOpInterface>(op);
    return dpsOp.isDpsInit(&opOperand);
  }

```
- **EN**: Implements logic around `bufferizesToMemoryWrite`, `isDpsInit`.
- **CN**: 围绕 `bufferizesToMemoryWrite`, `isDpsInit` 实现具体逻辑。

### Lines 114-121
```cpp
  bool bufferizesToElementwiseAccess(Operation *op, const AnalysisState &state,
                                     ArrayRef<OpOperand *> opOperands) const {
    auto linalgOp = cast<linalg::LinalgOp>(op);

    // Accesses into sparse data structures are not necessarily elementwise.
    if (sparse_tensor::hasAnySparseOperand(linalgOp))
      return false;

```
- **EN**: Implements logic around `bufferizesToElementwiseAccess`, `LinalgOp>`, `hasAnySparseOperand`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bufferizesToElementwiseAccess`, `LinalgOp>`, `hasAnySparseOperand` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 122-135
```cpp
    // All loops must be parallel.
    if (linalgOp.getNumLoops() != linalgOp.getNumParallelLoops())
      return false;

    // All index maps of tensors must be identity maps.
    SmallVector<AffineMap> indexingMaps = linalgOp.getIndexingMapsArray();
    assert(linalgOp->getNumOperands() == indexingMaps.size() &&
           "unexpected number of indexing maps");
    for (auto [operand, map] :
         llvm::zip(linalgOp->getOpOperands(), indexingMaps)) {
      // Non-tensors do not participate in bufferization, so they can be
      // ignored.
      if (!isa<RankedTensorType, MemRefType>(operand.get().getType()))
        continue;
```
- **EN**: Implements logic around `getNumLoops`, `getIndexingMapsArray`, `assert`, `zip`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumLoops`, `getIndexingMapsArray`, `assert`, `zip`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 136-144
```cpp
      // Only consider operands in `opOperands`.
      if (!llvm::is_contained(opOperands, &operand))
        continue;
      // TODO: This could be generalized to other indexing maps. (All indexing
      // must be the same.)
      if (!map.isIdentity())
        return false;
    }

```
- **EN**: Implements logic around `is_contained`, `isIdentity`.
- **CN**: 围绕 `is_contained`, `isIdentity` 实现具体逻辑。

### Lines 145-155
```cpp
    return true;
  }

  LogicalResult bufferize(Operation *op, RewriterBase &rewriter,
                          const BufferizationOptions &options,
                          BufferizationState &state) const {
    return bufferizeDestinationStyleOpInterface(
        rewriter, cast<DestinationStyleOpInterface>(op), options, state);
  }
};

```
- **EN**: Implements logic around `bufferize`, `bufferizeDestinationStyleOpInterface`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `bufferize`, `bufferizeDestinationStyleOpInterface` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 156-164
```cpp
/// Helper structure that iterates over all LinalgOps in `OpTys` and registers
/// the `BufferizableOpInterface` with each of them.
template <typename... Ops>
struct LinalgOpInterfaceHelper {
  static void registerOpInterface(MLIRContext *ctx) {
    (Ops::template attachInterface<LinalgOpInterface<Ops>>(*ctx), ...);
  }
};

```
- **EN**: Introduces declarations for `LinalgOpInterfaceHelper`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgOpInterfaceHelper` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 165-174
```cpp
struct SoftmaxOpInterface
    : public DstBufferizableOpInterfaceExternalModel<SoftmaxOpInterface,
                                                     linalg::SoftmaxOp> {
  bool bufferizesToMemoryRead(Operation *op, OpOperand &opOperand,
                              const AnalysisState &state) const {
    // Output operand is not read.
    auto softmaxOp = cast<linalg::SoftmaxOp>(op);
    return &opOperand == &softmaxOp.getInputMutable();
  }

```
- **EN**: Introduces declarations for `SoftmaxOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SoftmaxOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 175-188
```cpp
  LogicalResult bufferize(Operation *op, RewriterBase &rewriter,
                          const BufferizationOptions &options,
                          BufferizationState &state) const {
    auto softmaxOp = cast<linalg::SoftmaxOp>(op);
    FailureOr<Value> inputBuffer =
        getBuffer(rewriter, softmaxOp.getInput(), options, state);
    if (failed(inputBuffer))
      return failure();
    FailureOr<Value> outputBuffer =
        getBuffer(rewriter, softmaxOp.getOutput(), options, state);
    if (failed(outputBuffer))
      return failure();
    linalg::SoftmaxOp::create(rewriter, softmaxOp.getLoc(),
                              /*result=*/TypeRange(), *inputBuffer,
```
- **EN**: Implements logic around `bufferize`, `SoftmaxOp>`, `getBuffer`, `failed`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bufferize`, `SoftmaxOp>`, `getBuffer`, `failed`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 189-195
```cpp
                              *outputBuffer, softmaxOp.getDimension());
    replaceOpWithBufferizedValues(rewriter, op, *outputBuffer);
    return success();
  }
};

struct PackOpInterface
```
- **EN**: Introduces declarations for `PackOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PackOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 196-203
```cpp
    : public DstBufferizableOpInterfaceExternalModel<PackOpInterface,
                                                     linalg::PackOp> {
  bool bufferizesToMemoryRead(Operation *op, OpOperand &opOperand,
                              const AnalysisState &state) const {
    auto packOp = cast<linalg::PackOp>(op);
    return !packOp.isDpsInit(&opOperand);
  }

```
- **EN**: Implements logic around `bufferizesToMemoryRead`, `PackOp>`, `isDpsInit`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bufferizesToMemoryRead`, `PackOp>`, `isDpsInit` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 204-217
```cpp
  LogicalResult bufferize(Operation *op, RewriterBase &rewriter,
                          const BufferizationOptions &options,
                          BufferizationState &state) const {
    auto packOp = cast<linalg::PackOp>(op);
    assert(!packOp.hasPureBufferSemantics() && "expected op with tensors");
    if (!packOp.hasPureTensorSemantics())
      return packOp.emitError()
             << "mixed tensor/buffer semantic op not supported yet";
    FailureOr<Value> sourceBuffer =
        getBuffer(rewriter, packOp.getSource(), options, state);
    if (failed(sourceBuffer))
      return failure();
    FailureOr<Value> destBuffer =
        getBuffer(rewriter, packOp.getDest(), options, state);
```
- **EN**: Implements logic around `bufferize`, `PackOp>`, `assert`, `hasPureTensorSemantics`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bufferize`, `PackOp>`, `assert`, `hasPureTensorSemantics`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 218-227
```cpp
    if (failed(destBuffer))
      return failure();

    SmallVector<Value> operands;
    operands.push_back(*sourceBuffer);
    operands.push_back(*destBuffer);
    if (auto val = packOp.getPaddingValue())
      operands.push_back(val);
    llvm::append_range(operands, packOp.getInnerTiles());

```
- **EN**: Implements logic around `failed`, `failure`, `push_back`, `getPaddingValue`, and 1 more symbols.
- **CN**: 围绕 `failed`, `failure`, `push_back`, `getPaddingValue`, and 1 more symbols 实现具体逻辑。

### Lines 228-234
```cpp
    linalg::PackOp::create(rewriter, packOp.getLoc(), TypeRange{}, operands,
                           op->getAttrs());
    replaceOpWithBufferizedValues(rewriter, op, *destBuffer);
    return success();
  }
};

```
- **EN**: Implements logic around `create`, `getAttrs`, `replaceOpWithBufferizedValues`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getAttrs`, `replaceOpWithBufferizedValues`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 235-243
```cpp
struct UnPackOpInterface
    : public DstBufferizableOpInterfaceExternalModel<UnPackOpInterface,
                                                     linalg::UnPackOp> {
  bool bufferizesToMemoryRead(Operation *op, OpOperand &opOperand,
                              const AnalysisState &state) const {
    auto unPackOp = cast<linalg::UnPackOp>(op);
    return !unPackOp.isDpsInit(&opOperand);
  }

```
- **EN**: Introduces declarations for `UnPackOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UnPackOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 244-257
```cpp
  LogicalResult bufferize(Operation *op, RewriterBase &rewriter,
                          const BufferizationOptions &options,
                          BufferizationState &state) const {
    auto unPackOp = cast<linalg::UnPackOp>(op);
    assert(!unPackOp.hasPureBufferSemantics() && "expected op with tensors");
    if (!unPackOp.hasPureTensorSemantics())
      return unPackOp.emitError()
             << "mixed tensor/buffer semantic op not supported yet";
    FailureOr<Value> sourceBuffer =
        getBuffer(rewriter, unPackOp.getSource(), options, state);
    if (failed(sourceBuffer))
      return failure();
    FailureOr<Value> destBuffer =
        getBuffer(rewriter, unPackOp.getDest(), options, state);
```
- **EN**: Implements logic around `bufferize`, `UnPackOp>`, `assert`, `hasPureTensorSemantics`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bufferize`, `UnPackOp>`, `assert`, `hasPureTensorSemantics`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 258-265
```cpp
    if (failed(destBuffer))
      return failure();

    SmallVector<Value> operands;
    operands.push_back(*sourceBuffer);
    operands.push_back(*destBuffer);
    llvm::append_range(operands, unPackOp.getInnerTiles());

```
- **EN**: Implements logic around `failed`, `failure`, `push_back`, `append_range`.
- **CN**: 围绕 `failed`, `failure`, `push_back`, `append_range` 实现具体逻辑。

### Lines 266-273
```cpp
    linalg::UnPackOp::create(rewriter, unPackOp.getLoc(), TypeRange{}, operands,
                             op->getAttrs());
    replaceOpWithBufferizedValues(rewriter, op, *destBuffer);
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `create`, `getAttrs`, `replaceOpWithBufferizedValues`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getAttrs`, `replaceOpWithBufferizedValues`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 274-283
```cpp
void mlir::linalg::registerBufferizableOpInterfaceExternalModels(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, linalg::LinalgDialect *dialect) {
    // Register all Linalg structured ops. `LinalgOp` is an interface and it is
    // not possible to attach an external interface to an existing interface.
    // Therefore, attach the `BufferizableOpInterface` to all ops one-by-one.
    LinalgOpInterfaceHelper<
#define GET_OP_LIST
#include "mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`。

### Lines 284-290
```cpp
        >::registerOpInterface(ctx);

    SoftmaxOp::attachInterface<SoftmaxOpInterface>(*ctx);
    PackOp::attachInterface<PackOpInterface>(*ctx);
    UnPackOp::attachInterface<UnPackOpInterface>(*ctx);
  });
}
```
- **EN**: Implements logic around `registerOpInterface`, `attachInterface`.
- **CN**: 围绕 `registerOpInterface`, `attachInterface` 实现具体逻辑。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/BufferizableOpInterfaceImpl.h`, `mlir/Dialect/Bufferization/IR/BufferizableOpInterface.h`, `mlir/Dialect/Bufferization/IR/DstBufferizableOpInterfaceImpl.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/SparseTensor/IR/SparseTensor.h`, `mlir/IR/Dialect.h`, `mlir/IR/Operation.h`, `mlir/Interfaces/DestinationStyleOpInterface.h`, `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`
