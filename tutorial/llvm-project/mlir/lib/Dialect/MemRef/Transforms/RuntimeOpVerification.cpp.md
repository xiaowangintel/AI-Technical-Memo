# RuntimeOpVerification.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/MemRef/Transforms/RuntimeOpVerification.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MemRef dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `RuntimeOpVerification`.
  - **CN**: 实现 MemRef 方言中围绕 `RuntimeOpVerification` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- RuntimeOpVerification.cpp - Op Verification ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/MemRef/Transforms/RuntimeOpVerification.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MemRef/Transforms/RuntimeOpVerification.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MemRef/Transforms/RuntimeOpVerification.h`。

### Lines 11-20
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlow.h"
#include "mlir/Dialect/ControlFlow/IR/ControlFlowOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/MemRef/Utils/MemRefUtils.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Interfaces/RuntimeVerifiableOpInterface.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ControlFlow/IR/ControlFlow.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ControlFlow/IR/ControlFlow.h`。

### Lines 21-37
```cpp
using namespace mlir;

namespace mlir {
namespace memref {
namespace {
/// Generate a runtime check for lb <= value < ub.
Value generateInBoundsCheck(OpBuilder &builder, Location loc, Value value,
                            Value lb, Value ub) {
  Value inBounds1 = builder.createOrFold<arith::CmpIOp>(
      loc, arith::CmpIPredicate::sge, value, lb);
  Value inBounds2 = builder.createOrFold<arith::CmpIOp>(
      loc, arith::CmpIPredicate::slt, value, ub);
  Value inBounds =
      builder.createOrFold<arith::AndIOp>(loc, inBounds1, inBounds2);
  return inBounds;
}

```
- **EN**: Introduces declarations for `mlir`, `memref`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `memref` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 38-57
```cpp
struct AssumeAlignmentOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<
          AssumeAlignmentOpInterface, AssumeAlignmentOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto assumeOp = cast<AssumeAlignmentOp>(op);
    Value ptr = ExtractAlignedPointerAsIndexOp::create(builder, loc,
                                                       assumeOp.getMemref());
    Value rest = arith::RemUIOp::create(
        builder, loc, ptr,
        arith::ConstantIndexOp::create(builder, loc, assumeOp.getAlignment()));
    Value isAligned =
        arith::CmpIOp::create(builder, loc, arith::CmpIPredicate::eq, rest,
                              arith::ConstantIndexOp::create(builder, loc, 0));
    cf::AssertOp::create(
        builder, loc, isAligned,
        generateErrorMessage(op, "memref is not aligned to " +
                                     std::to_string(assumeOp.getAlignment())));
```
- **EN**: Introduces declarations for `AssumeAlignmentOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `AssumeAlignmentOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 58-70
```cpp
  }
};

struct CastOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<CastOpInterface,
                                                         CastOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto castOp = cast<CastOp>(op);
    auto srcType = cast<BaseMemRefType>(castOp.getSource().getType());

```
- **EN**: Introduces declarations for `CastOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `CastOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 71-86
```cpp
    // Nothing to check if the result is an unranked memref.
    auto resultType = dyn_cast<MemRefType>(castOp.getType());
    if (!resultType)
      return;

    if (isa<UnrankedMemRefType>(srcType)) {
      // Check rank.
      Value srcRank = RankOp::create(builder, loc, castOp.getSource());
      Value resultRank =
          arith::ConstantIndexOp::create(builder, loc, resultType.getRank());
      Value isSameRank = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, srcRank, resultRank);
      cf::AssertOp::create(builder, loc, isSameRank,
                           generateErrorMessage(op, "rank mismatch"));
    }

```
- **EN**: Implements logic around `getType`, `create`, `generateErrorMessage`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `create`, `generateErrorMessage` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 87-103
```cpp
    // Get source offset and strides. We do not have an op to get offsets and
    // strides from unranked memrefs, so cast the source to a type with fully
    // dynamic layout, from which we can then extract the offset and strides.
    // (Rank was already verified.)
    int64_t dynamicOffset = ShapedType::kDynamic;
    SmallVector<int64_t> dynamicShape(resultType.getRank(),
                                      ShapedType::kDynamic);
    auto stridedLayout = StridedLayoutAttr::get(builder.getContext(),
                                                dynamicOffset, dynamicShape);
    auto dynStridesType =
        MemRefType::get(dynamicShape, resultType.getElementType(),
                        stridedLayout, resultType.getMemorySpace());
    Value helperCast =
        CastOp::create(builder, loc, dynStridesType, castOp.getSource());
    auto metadataOp =
        ExtractStridedMetadataOp::create(builder, loc, helperCast);

```
- **EN**: Implements logic around `dynamicShape`, `get`, `getMemorySpace`, `create`.
- **CN**: 围绕 `dynamicShape`, `get`, `getMemorySpace`, `create` 实现具体逻辑。

### Lines 104-114
```cpp
    // Check dimension sizes.
    for (const auto &it : llvm::enumerate(resultType.getShape())) {
      // Static dim size -> static/dynamic dim size does not need verification.
      if (auto rankedSrcType = dyn_cast<MemRefType>(srcType))
        if (!rankedSrcType.isDynamicDim(it.index()))
          continue;

      // Static/dynamic dim size -> dynamic dim size does not need verification.
      if (resultType.isDynamicDim(it.index()))
        continue;

```
- **EN**: Implements logic around `enumerate`, `isDynamicDim`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `enumerate`, `isDynamicDim` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 115-126
```cpp
      Value srcDimSz =
          DimOp::create(builder, loc, castOp.getSource(), it.index());
      Value resultDimSz =
          arith::ConstantIndexOp::create(builder, loc, it.value());
      Value isSameSz = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, srcDimSz, resultDimSz);
      cf::AssertOp::create(
          builder, loc, isSameSz,
          generateErrorMessage(op, "size mismatch of dim " +
                                       std::to_string(it.index())));
    }

```
- **EN**: Implements logic around `create`, `generateErrorMessage`, `to_string`.
- **CN**: 围绕 `create`, `generateErrorMessage`, `to_string` 实现具体逻辑。

### Lines 127-144
```cpp
    // Get result offset and strides.
    int64_t resultOffset;
    SmallVector<int64_t> resultStrides;
    if (failed(resultType.getStridesAndOffset(resultStrides, resultOffset)))
      return;

    // Check offset.
    if (resultOffset != ShapedType::kDynamic) {
      // Static/dynamic offset -> dynamic offset does not need verification.
      Value srcOffset = metadataOp.getResult(1);
      Value resultOffsetVal =
          arith::ConstantIndexOp::create(builder, loc, resultOffset);
      Value isSameOffset = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, srcOffset, resultOffsetVal);
      cf::AssertOp::create(builder, loc, isSameOffset,
                           generateErrorMessage(op, "offset mismatch"));
    }

```
- **EN**: Implements logic around `failed`, `getResult`, `create`, `generateErrorMessage`; this block checks operation invariants or verification rules.
- **CN**: 围绕 `failed`, `getResult`, `create`, `generateErrorMessage` 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 145-164
```cpp
    // Check strides.
    for (const auto &it : llvm::enumerate(resultStrides)) {
      // Static/dynamic stride -> dynamic stride does not need verification.
      if (it.value() == ShapedType::kDynamic)
        continue;

      Value srcStride =
          metadataOp.getResult(2 + resultType.getRank() + it.index());
      Value resultStrideVal =
          arith::ConstantIndexOp::create(builder, loc, it.value());
      Value isSameStride = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, srcStride, resultStrideVal);
      cf::AssertOp::create(
          builder, loc, isSameStride,
          generateErrorMessage(op, "stride mismatch of dim " +
                                       std::to_string(it.index())));
    }
  }
};

```
- **EN**: Implements logic around `enumerate`, `value`, `getResult`, `create`, and 2 more symbols; this block checks operation invariants or verification rules.
- **CN**: 围绕 `enumerate`, `value`, `getResult`, `create`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则。

### Lines 165-177
```cpp
struct CopyOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<CopyOpInterface,
                                                         CopyOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto copyOp = cast<CopyOp>(op);
    BaseMemRefType sourceType = copyOp.getSource().getType();
    BaseMemRefType targetType = copyOp.getTarget().getType();
    auto rankedSourceType = dyn_cast<MemRefType>(sourceType);
    auto rankedTargetType = dyn_cast<MemRefType>(targetType);

```
- **EN**: Introduces declarations for `CopyOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `CopyOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 178-197
```cpp
    // TODO: Verification for unranked memrefs is not supported yet.
    if (!rankedSourceType || !rankedTargetType)
      return;

    assert(sourceType.getRank() == targetType.getRank() && "rank mismatch");
    for (int64_t i = 0, e = sourceType.getRank(); i < e; ++i) {
      // Fully static dimensions in both source and target operand are already
      // verified by the op verifier.
      if (!rankedSourceType.isDynamicDim(i) &&
          !rankedTargetType.isDynamicDim(i))
        continue;
      auto getDimSize = [&](Value memRef, MemRefType type,
                            int64_t dim) -> Value {
        return type.isDynamicDim(dim)
                   ? DimOp::create(builder, loc, memRef, dim).getResult()
                   : arith::ConstantIndexOp::create(builder, loc,
                                                    type.getDimSize(dim))
                         .getResult();
      };
      Value sourceDim = getDimSize(copyOp.getSource(), rankedSourceType, i);
```
- **EN**: Implements logic around `assert`, `getRank`, `isDynamicDim`, `create`, and 2 more symbols; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `getRank`, `isDynamicDim`, `create`, and 2 more symbols 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 198-208
```cpp
      Value targetDim = getDimSize(copyOp.getTarget(), rankedTargetType, i);
      Value sameDimSize = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, sourceDim, targetDim);
      cf::AssertOp::create(
          builder, loc, sameDimSize,
          generateErrorMessage(op, "size of " + std::to_string(i) +
                                       "-th source/target dim does not match"));
    }
  }
};

```
- **EN**: Implements logic around `getDimSize`, `create`, `generateErrorMessage`.
- **CN**: 围绕 `getDimSize`, `create`, `generateErrorMessage` 实现具体逻辑。

### Lines 209-225
```cpp
struct DimOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<DimOpInterface,
                                                         DimOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto dimOp = cast<DimOp>(op);
    Value rank = RankOp::create(builder, loc, dimOp.getSource());
    Value zero = arith::ConstantIndexOp::create(builder, loc, 0);
    cf::AssertOp::create(
        builder, loc,
        generateInBoundsCheck(builder, loc, dimOp.getIndex(), zero, rank),
        generateErrorMessage(op, "index is out of bounds"));
  }
};

```
- **EN**: Introduces declarations for `DimOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DimOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 226-237
```cpp
/// Verifies that the indices on load/store ops are in-bounds of the memref's
/// index space: 0 <= index#i < dim#i
template <typename LoadStoreOp>
struct LoadStoreOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<
          LoadStoreOpInterface<LoadStoreOp>, LoadStoreOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto loadStoreOp = cast<LoadStoreOp>(op);

```
- **EN**: Introduces declarations for `LoadStoreOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LoadStoreOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 238-257
```cpp
    auto memref = loadStoreOp.getMemref();
    auto rank = memref.getType().getRank();
    if (rank == 0) {
      return;
    }
    auto indices = loadStoreOp.getIndices();

    auto zero = arith::ConstantIndexOp::create(builder, loc, 0);
    Value assertCond;
    for (auto i : llvm::seq<int64_t>(0, rank)) {
      Value dimOp = builder.createOrFold<memref::DimOp>(loc, memref, i);
      Value inBounds =
          generateInBoundsCheck(builder, loc, indices[i], zero, dimOp);
      assertCond =
          i > 0 ? builder.createOrFold<arith::AndIOp>(loc, assertCond, inBounds)
                : inBounds;
    }
    cf::AssertOp::create(builder, loc, assertCond,
                         generateErrorMessage(op, "out-of-bounds access"));
  }
```
- **EN**: Implements logic around `getMemref`, `getType`, `getIndices`, `create`, and 5 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getMemref`, `getType`, `getIndices`, `create`, and 5 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 258-269
```cpp
};

struct SubViewOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<SubViewOpInterface,
                                                         SubViewOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto subView = cast<SubViewOp>(op);
    MemRefType sourceType = subView.getSource().getType();

```
- **EN**: Introduces declarations for `SubViewOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SubViewOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 270-280
```cpp
    // For each dimension, assert that:
    // For empty slices (size == 0)   : 0 <= offset <= dim_size
    // For non-empty slices (size > 0): 0 <= offset < dim_size
    //                                  0 <= offset + (size - 1) * stride
    //                                  dim_size
    Value zero = arith::ConstantIndexOp::create(builder, loc, 0);
    Value one = arith::ConstantIndexOp::create(builder, loc, 1);

    auto metadataOp =
        ExtractStridedMetadataOp::create(builder, loc, subView.getSource());

```
- **EN**: Implements logic around `create`.
- **CN**: 围绕 `create` 实现具体逻辑。

### Lines 281-292
```cpp
    for (int64_t i : llvm::seq<int64_t>(0, sourceType.getRank())) {
      // Reset insertion point to before the operation for each dimension.
      builder.setInsertionPoint(subView);

      Value offset = getValueOrCreateConstantIndexOp(
          builder, loc, subView.getMixedOffsets()[i]);
      Value size = getValueOrCreateConstantIndexOp(builder, loc,
                                                   subView.getMixedSizes()[i]);
      Value stride = getValueOrCreateConstantIndexOp(
          builder, loc, subView.getMixedStrides()[i]);
      Value dimSize = metadataOp.getSizes()[i];

```
- **EN**: Implements logic around `seq`, `setInsertionPoint`, `getValueOrCreateConstantIndexOp`, `getMixedOffsets`, and 3 more symbols.
- **CN**: 围绕 `seq`, `setInsertionPoint`, `getValueOrCreateConstantIndexOp`, `getMixedOffsets`, and 3 more symbols 实现具体逻辑。

### Lines 293-312
```cpp
      // Verify that offset is in-bounds (conditional on slice size).
      Value sizeIsZero = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, size, zero);
      auto offsetCheckIf = scf::IfOp::create(
          builder, loc, sizeIsZero,
          [&](OpBuilder &b, Location loc) {
            // For empty slices, offset can be at the boundary: 0 <= offset <=
            // dimSize.
            Value offsetGEZero = arith::CmpIOp::create(
                b, loc, arith::CmpIPredicate::sge, offset, zero);
            Value offsetLEDimSize = arith::CmpIOp::create(
                b, loc, arith::CmpIPredicate::sle, offset, dimSize);
            Value emptyOffsetValid =
                arith::AndIOp::create(b, loc, offsetGEZero, offsetLEDimSize);
            scf::YieldOp::create(b, loc, emptyOffsetValid);
          },
          [&](OpBuilder &b, Location loc) {
            // For non-empty slices, offset must be a valid index: 0 <= offset
            // dimSize.
            Value offsetInBounds =
```
- **EN**: Implements logic around `create`; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 313-322
```cpp
                generateInBoundsCheck(b, loc, offset, zero, dimSize);
            scf::YieldOp::create(b, loc, offsetInBounds);
          });

      Value offsetCondition = offsetCheckIf.getResult(0);
      cf::AssertOp::create(builder, loc, offsetCondition,
                           generateErrorMessage(op, "offset " +
                                                        std::to_string(i) +
                                                        " is out-of-bounds"));

```
- **EN**: Implements logic around `generateInBoundsCheck`, `create`, `getResult`, `generateErrorMessage`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `generateInBoundsCheck`, `create`, `getResult`, `generateErrorMessage`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 323-342
```cpp
      // Verify that the slice endpoint is in-bounds (only for non-empty
      // slices).
      Value sizeIsNonZero = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::sgt, size, zero);
      auto ifOp = scf::IfOp::create(
          builder, loc, sizeIsNonZero,
          [&](OpBuilder &b, Location loc) {
            // Verify that slice does not run out-of-bounds.
            Value sizeMinusOne = arith::SubIOp::create(b, loc, size, one);
            Value sizeMinusOneTimesStride =
                arith::MulIOp::create(b, loc, sizeMinusOne, stride);
            Value lastPos =
                arith::AddIOp::create(b, loc, offset, sizeMinusOneTimesStride);
            Value lastPosInBounds =
                generateInBoundsCheck(b, loc, lastPos, zero, dimSize);
            scf::YieldOp::create(b, loc, lastPosInBounds);
          },
          [&](OpBuilder &b, Location loc) {
            Value trueVal =
                arith::ConstantOp::create(b, loc, b.getBoolAttr(true));
```
- **EN**: Implements logic around `create`, `generateInBoundsCheck`; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `generateInBoundsCheck` 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

### Lines 343-355
```cpp
            scf::YieldOp::create(b, loc, trueVal);
          });

      Value finalCondition = ifOp.getResult(0);
      cf::AssertOp::create(
          builder, loc, finalCondition,
          generateErrorMessage(op,
                               "subview runs out-of-bounds along dimension " +
                                   std::to_string(i)));
    }
  }
};

```
- **EN**: Implements logic around `create`, `getResult`, `generateErrorMessage`, `to_string`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `generateErrorMessage`, `to_string` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 356-375
```cpp
struct ExpandShapeOpInterface
    : public RuntimeVerifiableOpInterface::ExternalModel<ExpandShapeOpInterface,
                                                         ExpandShapeOp> {
  void
  generateRuntimeVerification(Operation *op, OpBuilder &builder, Location loc,
                              function_ref<std::string(Operation *, StringRef)>
                                  generateErrorMessage) const {
    auto expandShapeOp = cast<ExpandShapeOp>(op);

    // Verify that the expanded dim sizes are a product of the collapsed dim
    // size.
    for (const auto &it :
         llvm::enumerate(expandShapeOp.getReassociationIndices())) {
      Value srcDimSz =
          DimOp::create(builder, loc, expandShapeOp.getSrc(), it.index());
      int64_t groupSz = 1;
      bool foundDynamicDim = false;
      for (int64_t resultDim : it.value()) {
        if (expandShapeOp.getResultType().isDynamicDim(resultDim)) {
          // Keep this assert here in case the op is extended in the future.
```
- **EN**: Introduces declarations for `ExpandShapeOpInterface`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ExpandShapeOpInterface` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 376-395
```cpp
          assert(!foundDynamicDim &&
                 "more than one dynamic dim found in reassoc group");
          (void)foundDynamicDim;
          foundDynamicDim = true;
          continue;
        }
        groupSz *= expandShapeOp.getResultType().getDimSize(resultDim);
      }
      Value staticResultDimSz =
          arith::ConstantIndexOp::create(builder, loc, groupSz);
      // staticResultDimSz must divide srcDimSz evenly.
      Value mod =
          arith::RemSIOp::create(builder, loc, srcDimSz, staticResultDimSz);
      Value isModZero = arith::CmpIOp::create(
          builder, loc, arith::CmpIPredicate::eq, mod,
          arith::ConstantIndexOp::create(builder, loc, 0));
      cf::AssertOp::create(
          builder, loc, isModZero,
          generateErrorMessage(op, "static result dims in reassoc group do not "
                                   "divide src dim evenly"));
```
- **EN**: Implements logic around `assert`, `getResultType`, `create`, `generateErrorMessage`.
- **CN**: 围绕 `assert`, `getResultType`, `create`, `generateErrorMessage` 实现具体逻辑。

### Lines 396-415
```cpp
    }
  }
};
} // namespace
} // namespace memref
} // namespace mlir

void mlir::memref::registerRuntimeVerifiableOpInterfaceExternalModels(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, memref::MemRefDialect *dialect) {
    AssumeAlignmentOp::attachInterface<AssumeAlignmentOpInterface>(*ctx);
    AtomicRMWOp::attachInterface<LoadStoreOpInterface<AtomicRMWOp>>(*ctx);
    CastOp::attachInterface<CastOpInterface>(*ctx);
    CopyOp::attachInterface<CopyOpInterface>(*ctx);
    DimOp::attachInterface<DimOpInterface>(*ctx);
    ExpandShapeOp::attachInterface<ExpandShapeOpInterface>(*ctx);
    GenericAtomicRMWOp::attachInterface<
        LoadStoreOpInterface<GenericAtomicRMWOp>>(*ctx);
    LoadOp::attachInterface<LoadStoreOpInterface<LoadOp>>(*ctx);
    StoreOp::attachInterface<LoadStoreOpInterface<StoreOp>>(*ctx);
```
- **EN**: Introduces declarations for `memref`, `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `memref`, `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 416-423
```cpp
    SubViewOp::attachInterface<SubViewOpInterface>(*ctx);
    // Note: There is nothing to verify for ReinterpretCastOp.

    // Load additional dialects of which ops may get created.
    ctx->loadDialect<affine::AffineDialect, arith::ArithDialect,
                     cf::ControlFlowDialect>();
  });
}
```
- **EN**: Implements logic around `attachInterface`, `ControlFlowDialect>`; this block checks operation invariants or verification rules; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `attachInterface`, `ControlFlowDialect>` 实现具体逻辑；该代码块检查操作不变式或验证规则，并协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/MemRef/Transforms/RuntimeOpVerification.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/ControlFlow/IR/ControlFlow.h`, `mlir/Dialect/ControlFlow/IR/ControlFlowOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Utils/MemRefUtils.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Interfaces/RuntimeVerifiableOpInterface.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (9), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
