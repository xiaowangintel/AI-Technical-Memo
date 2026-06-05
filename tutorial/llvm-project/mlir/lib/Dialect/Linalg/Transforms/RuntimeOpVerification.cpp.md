# RuntimeOpVerification.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/RuntimeOpVerification.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `RuntimeOpVerification`.
  - **CN**: 实现 Linalg 方言中围绕 `RuntimeOpVerification` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- RuntimeOpVerification.cpp - Op Verification ------------------------===//
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

#include "mlir/Dialect/Linalg/Transforms/RuntimeOpVerification.h"

#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/Index/IR/IndexAttrs.h"
#include "mlir/Dialect/Index/IR/IndexDialect.h"
#include "mlir/Dialect/Index/IR/IndexOps.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/RuntimeOpVerification.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/RuntimeOpVerification.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`。

### Lines 22-28
```cpp
#include "mlir/Interfaces/RuntimeVerifiableOpInterface.h"

namespace mlir {
namespace linalg {
namespace {
/// Verify that the runtime sizes of the operands to linalg structured ops are
/// compatible with the runtime sizes inferred by composing the loop ranges with
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Interfaces/RuntimeVerifiableOpInterface.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Interfaces/RuntimeVerifiableOpInterface.h`。

### Lines 29-40
```cpp
/// the linalg op's indexing maps. This is similar to the verifier except that
/// here we insert IR to perform the verification at runtime.
template <typename T>
struct StructuredOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<
          StructuredOpInterface<T>, T> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto linalgOp = llvm::cast<LinalgOp>(op);

```
- **EN**: Introduces declarations for `StructuredOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `StructuredOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 41-51
```cpp
    SmallVector<Range> loopRanges = linalgOp.createLoopRanges(builder, loc);
    auto [starts, ends, _] = getOffsetsSizesAndStrides(loopRanges);

    auto zero = arith::ConstantIndexOp::create(builder, loc, 0);
    auto one = arith::ConstantIndexOp::create(builder, loc, 1);

    Value iterationDomainIsNonDegenerate;
    for (auto [start, end] : llvm::zip(starts, ends)) {
      auto startValue = getValueOrCreateConstantIndexOp(builder, loc, start);
      auto endValue = getValueOrCreateConstantIndexOp(builder, loc, end);

```
- **EN**: Implements logic around `createLoopRanges`, `getOffsetsSizesAndStrides`, `create`, `zip`, and 1 more symbols.
- **CN**: 围绕 `createLoopRanges`, `getOffsetsSizesAndStrides`, `create`, `zip`, and 1 more symbols 实现具体逻辑。

### Lines 52-65
```cpp
      // Loop Trip count > 0 iff start < end
      Value dimensionHasNonZeroTripCount = index::CmpOp::create(
          builder, loc, index::IndexCmpPredicate::SLT, startValue, endValue);

      if (!iterationDomainIsNonDegenerate) {
        iterationDomainIsNonDegenerate = dimensionHasNonZeroTripCount;
      } else {
        // Iteration domain is non-degenerate iff all dimensions have loop trip
        // count > 0
        iterationDomainIsNonDegenerate =
            arith::AndIOp::create(builder, loc, iterationDomainIsNonDegenerate,
                                  dimensionHasNonZeroTripCount);
      }
    }
```
- **EN**: Implements logic around `create`.
- **CN**: 围绕 `create` 实现具体逻辑。

### Lines 66-73
```cpp

    if (!iterationDomainIsNonDegenerate)
      return;

    auto ifOp = scf::IfOp::create(builder, loc, iterationDomainIsNonDegenerate,
                                  /*withElseRegion=*/false);
    builder.setInsertionPointToStart(&ifOp.getThenRegion().front());

```
- **EN**: Implements logic around `create`, `setInsertionPointToStart`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `setInsertionPointToStart` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 74-86
```cpp
    // Subtract one from the loop ends before composing with the indexing map
    transform(ends, ends.begin(), [&](OpFoldResult end) {
      auto endValue = getValueOrCreateConstantIndexOp(builder, loc, end);
      return builder.createOrFold<index::SubOp>(loc, endValue, one);
    });

    for (OpOperand &opOperand : linalgOp->getOpOperands()) {
      AffineMap indexingMap = linalgOp.getMatchingIndexingMap(&opOperand);
      auto startIndices = affine::makeComposedFoldedMultiResultAffineApply(
          builder, loc, indexingMap, starts);
      auto endIndices = affine::makeComposedFoldedMultiResultAffineApply(
          builder, loc, indexingMap, ends);

```
- **EN**: Implements logic around `transform`, `getValueOrCreateConstantIndexOp`, `SubOp>`, `getOpOperands`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transform`, `getValueOrCreateConstantIndexOp`, `SubOp>`, `getOpOperands`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 87-100
```cpp
      for (auto dim : llvm::seq(linalgOp.getRank(&opOperand))) {
        auto startIndex =
            getValueOrCreateConstantIndexOp(builder, loc, startIndices[dim]);
        auto endIndex =
            getValueOrCreateConstantIndexOp(builder, loc, endIndices[dim]);

        // Generate:
        //   minIndex = min(startIndex, endIndex)
        //   assert(minIndex >= 0)
        // To ensure we do not generate a negative index. We take the minimum of
        // the start and end indices in order to handle reverse loops such as
        // `affine_map<(i) -> (3 - i)>`
        auto min =
            builder.createOrFold<index::MinSOp>(loc, startIndex, endIndex);
```
- **EN**: Implements logic around `seq`, `getValueOrCreateConstantIndexOp`, `MinSOp>`.
- **CN**: 围绕 `seq`, `getValueOrCreateConstantIndexOp`, `MinSOp>` 实现具体逻辑。

### Lines 101-108
```cpp
        auto cmpOp = builder.createOrFold<index::CmpOp>(
            loc, index::IndexCmpPredicate::SGE, min, zero);
        auto msg = generateErrorMessage(
            linalgOp, "unexpected negative result on dimension #" +
                          std::to_string(dim) + " of input/output operand #" +
                          std::to_string(opOperand.getOperandNumber()));
        builder.createOrFold<cf::AssertOp>(loc, cmpOp, msg);

```
- **EN**: Implements logic around `CmpOp>`, `generateErrorMessage`, `to_string`, `AssertOp>`.
- **CN**: 围绕 `CmpOp>`, `generateErrorMessage`, `to_string`, `AssertOp>` 实现具体逻辑。

### Lines 109-116
```cpp
        // Generate:
        //   inferredDimSize = max(startIndex, endIndex) + 1
        //   actualDimSize = dim(operand)
        //   assert(inferredDimSize <= actualDimSize)
        // To ensure that we do not index past the bounds of the operands.
        auto max =
            builder.createOrFold<index::MaxSOp>(loc, startIndex, endIndex);

```
- **EN**: Implements logic around `MaxSOp>`.
- **CN**: 围绕 `MaxSOp>` 实现具体逻辑。

### Lines 117-130
```cpp
        auto inferredDimSize =
            builder.createOrFold<index::AddOp>(loc, max, one);

        auto actualDimSize =
            createOrFoldDimOp(builder, loc, opOperand.get(), dim);

        // Similar to the verifier, when the affine expression in the indexing
        // map is complicated, we just check that the inferred dimension sizes
        // are in the boundary of the operands' size. Being more precise than
        // that is difficult.
        auto predicate = isa<AffineDimExpr>(indexingMap.getResult(dim))
                             ? index::IndexCmpPredicate::EQ
                             : index::IndexCmpPredicate::SLE;

```
- **EN**: Implements logic around `AddOp>`, `createOrFoldDimOp`, `getResult`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `AddOp>`, `createOrFoldDimOp`, `getResult` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 131-144
```cpp
        cmpOp = builder.createOrFold<index::CmpOp>(
            loc, predicate, inferredDimSize, actualDimSize);
        msg = generateErrorMessage(
            linalgOp, "dimension #" + std::to_string(dim) +
                          " of input/output operand #" +
                          std::to_string(opOperand.getOperandNumber()) +
                          " is incompatible with inferred dimension size");
        builder.createOrFold<cf::AssertOp>(loc, cmpOp, msg);
      }
    }
    builder.setInsertionPointAfter(ifOp);
  }
};

```
- **EN**: Implements logic around `CmpOp>`, `generateErrorMessage`, `to_string`, `AssertOp>`, and 1 more symbols.
- **CN**: 围绕 `CmpOp>`, `generateErrorMessage`, `to_string`, `AssertOp>`, and 1 more symbols 实现具体逻辑。

### Lines 145-152
```cpp
template <typename... OpTs>
void attachInterface(MLIRContext *ctx) {
  (OpTs::template attachInterface<StructuredOpInterface<OpTs>>(*ctx), ...);
}
} // namespace
} // namespace linalg
} // namespace mlir

```
- **EN**: Introduces declarations for `linalg`, `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `linalg`, `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 153-160
```cpp
void mlir::linalg::registerRuntimeVerifiableOpInterfaceExternalModels(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, LinalgDialect *) {
    attachInterface<
#define GET_OP_LIST
#include "mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc"
        >(ctx);

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/LinalgStructuredOps.cpp.inc`。

### Lines 161-166
```cpp
    // Load additional dialects of which ops may get created.
    ctx->loadDialect<affine::AffineDialect, arith::ArithDialect,
                     cf::ControlFlowDialect, index::IndexDialect,
                     tensor::TensorDialect>();
  });
}
```
- **EN**: Implements logic around `TensorDialect>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `TensorDialect>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/RuntimeOpVerification.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/Index/IR/IndexAttrs.h`, `mlir/Dialect/Index/IR/IndexDialect.h`, `mlir/Dialect/Index/IR/IndexOps.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/MemRef/IR/MemRef.h` ... (+4 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (13), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`
