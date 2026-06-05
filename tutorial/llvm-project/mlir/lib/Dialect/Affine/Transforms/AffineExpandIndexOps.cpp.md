# AffineExpandIndexOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/AffineExpandIndexOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to expand affine index ops into one or more more fundamental operations.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineExpandIndexOps.cpp - Affine expand index ops pass ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-15
```cpp
//
// This file implements a pass to expand affine index ops into one or more more
// fundamental operations.
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 16-22
```cpp
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/Transforms/Transforms.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

namespace mlir {
namespace affine {
```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 23-30
```cpp
#define GEN_PASS_DEF_AFFINEEXPANDINDEXOPS
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

using namespace mlir;
using namespace mlir::affine;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 31-37
```cpp
/// Given a basis (in static and dynamic components), return the sequence of
/// suffix products of the basis, including the product of the entire basis,
/// which must **not** contain an outer bound.
///
/// If excess dynamic values are provided, the values at the beginning
/// will be ignored. This allows for dropping the outer bound without
/// needing to manipulate the dynamic value array. `knownPositive`
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 38-46
```cpp
/// indicases that the values being used to compute the strides are known
/// to be non-negative.
static SmallVector<Value> computeStrides(Location loc, RewriterBase &rewriter,
                                         ValueRange dynamicBasis,
                                         ArrayRef<int64_t> staticBasis,
                                         bool knownNonNegative) {
  if (staticBasis.empty())
    return {};

```
- **EN**: Implements logic around `computeStrides`, `empty`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `computeStrides`, `empty` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 47-60
```cpp
  SmallVector<Value> result;
  result.reserve(staticBasis.size());
  size_t dynamicIndex = dynamicBasis.size();
  Value dynamicPart = nullptr;
  int64_t staticPart = 1;
  // The products of the strides can't have overflow by definition of
  // affine.*_index.
  arith::IntegerOverflowFlags ovflags = arith::IntegerOverflowFlags::nsw;
  if (knownNonNegative)
    ovflags = ovflags | arith::IntegerOverflowFlags::nuw;
  for (int64_t elem : llvm::reverse(staticBasis)) {
    if (ShapedType::isDynamic(elem)) {
      // Note: basis elements and their products are, definitionally,
      // non-negative, so `nuw` is justified.
```
- **EN**: Implements logic around `reserve`, `size`, `reverse`, `isDynamic`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `reserve`, `size`, `reverse`, `isDynamic` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 61-71
```cpp
      if (dynamicPart)
        dynamicPart =
            arith::MulIOp::create(rewriter, loc, dynamicPart,
                                  dynamicBasis[dynamicIndex - 1], ovflags);
      else
        dynamicPart = dynamicBasis[dynamicIndex - 1];
      --dynamicIndex;
    } else {
      staticPart *= elem;
    }

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 72-85
```cpp
    if (dynamicPart && staticPart == 1) {
      result.push_back(dynamicPart);
    } else {
      Value stride =
          rewriter.createOrFold<arith::ConstantIndexOp>(loc, staticPart);
      if (dynamicPart)
        stride =
            arith::MulIOp::create(rewriter, loc, dynamicPart, stride, ovflags);
      result.push_back(stride);
    }
  }
  std::reverse(result.begin(), result.end());
  return result;
}
```
- **EN**: Implements logic around `push_back`, `ConstantIndexOp>`, `create`, `reverse`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `push_back`, `ConstantIndexOp>`, `create`, `reverse` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 86-95
```cpp

/// Broadcast a scalar value to match the given type. If the type is already
/// scalar, returns the value as-is. For vector types, uses vector.broadcast.
static Value broadcastToMatchType(RewriterBase &rewriter, Location loc,
                                  Value value, Type targetType) {
  if (value.getType() == targetType)
    return value;
  return vector::BroadcastOp::create(rewriter, loc, targetType, value);
}

```
- **EN**: Implements logic around `broadcastToMatchType`, `getType`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `broadcastToMatchType`, `getType`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 96-105
```cpp
LogicalResult
affine::lowerAffineDelinearizeIndexOp(RewriterBase &rewriter,
                                      AffineDelinearizeIndexOp op) {
  Location loc = op.getLoc();
  Value linearIdx = op.getLinearIndex();
  unsigned numResults = op.getNumResults();
  ArrayRef<int64_t> staticBasis = op.getStaticBasis();
  if (numResults == staticBasis.size())
    staticBasis = staticBasis.drop_front();

```
- **EN**: Implements logic around `lowerAffineDelinearizeIndexOp`, `getLoc`, `getLinearIndex`, `getNumResults`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `lowerAffineDelinearizeIndexOp`, `getLoc`, `getLinearIndex`, `getNumResults`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并执行仿射推理或形状/边界处理。

### Lines 106-116
```cpp
  if (numResults == 1) {
    rewriter.replaceOp(op, linearIdx);
    return success();
  }

  SmallVector<Value> results;
  results.reserve(numResults);
  SmallVector<Value> strides =
      computeStrides(loc, rewriter, op.getDynamicBasis(), staticBasis,
                     /*knownNonNegative=*/true);

```
- **EN**: Implements logic around `replaceOp`, `success`, `reserve`, `computeStrides`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `success`, `reserve`, `computeStrides` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 117-125
```cpp
  // Broadcast strides and zero to match the linear index type (needed for
  // vector types where the strides are scalar but the index is a vector).
  Type indexType = linearIdx.getType();
  for (Value &stride : strides)
    stride = broadcastToMatchType(rewriter, loc, stride, indexType);

  Value zero =
      arith::ConstantOp::create(rewriter, loc, rewriter.getZeroAttr(indexType));

```
- **EN**: Implements logic around `getType`, `broadcastToMatchType`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getType`, `broadcastToMatchType`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 126-139
```cpp
  Value initialPart =
      arith::FloorDivSIOp::create(rewriter, loc, linearIdx, strides.front());
  results.push_back(initialPart);

  auto emitModTerm = [&](Value stride) -> Value {
    Value remainder = arith::RemSIOp::create(rewriter, loc, linearIdx, stride);
    Value remainderNegative = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::slt, remainder, zero);
    // If the correction is relevant, this term is <= stride, which is known
    // to be positive in `index`. Otherwise, while 2 * stride might overflow,
    // this branch won't be taken, so the risk of `poison` is fine.
    Value corrected = arith::AddIOp::create(rewriter, loc, remainder, stride,
                                            arith::IntegerOverflowFlags::nsw);
    Value mod = arith::SelectOp::create(rewriter, loc, remainderNegative,
```
- **EN**: Implements logic around `create`, `push_back`; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `create`, `push_back` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 140-153
```cpp
                                        corrected, remainder);
    return mod;
  };

  // Generate all the intermediate parts
  for (size_t i = 0, e = strides.size() - 1; i < e; ++i) {
    Value thisStride = strides[i];
    Value nextStride = strides[i + 1];
    Value modulus = emitModTerm(thisStride);
    // We know both inputs are positive, so floorDiv == div.
    // This could potentially be a divui, but it's not clear if that would
    // cause issues.
    Value divided = arith::DivSIOp::create(rewriter, loc, modulus, nextStride);
    results.push_back(divided);
```
- **EN**: Implements logic around `size`, `emitModTerm`, `create`, `push_back`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `size`, `emitModTerm`, `create`, `push_back` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 154-161
```cpp
  }

  results.push_back(emitModTerm(strides.back()));

  rewriter.replaceOp(op, results);
  return success();
}

```
- **EN**: Implements logic around `push_back`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `push_back`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 162-170
```cpp
LogicalResult affine::lowerAffineLinearizeIndexOp(RewriterBase &rewriter,
                                                  AffineLinearizeIndexOp op) {
  // Should be folded away, included here for safety.
  if (op.getMultiIndex().empty()) {
    rewriter.replaceOpWithNewOp<arith::ConstantOp>(
        op, rewriter.getZeroAttr(op.getLinearIndex().getType()));
    return success();
  }

```
- **EN**: Implements logic around `lowerAffineLinearizeIndexOp`, `getMultiIndex`, `ConstantOp>`, `getZeroAttr`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `lowerAffineLinearizeIndexOp`, `getMultiIndex`, `ConstantOp>`, `getZeroAttr`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子，并执行仿射推理或形状/边界处理。

### Lines 171-178
```cpp
  Location loc = op.getLoc();
  ValueRange multiIndex = op.getMultiIndex();
  Type indexType = op.getLinearIndex().getType();
  size_t numIndexes = multiIndex.size();
  ArrayRef<int64_t> staticBasis = op.getStaticBasis();
  if (numIndexes == staticBasis.size())
    staticBasis = staticBasis.drop_front();

```
- **EN**: Implements logic around `getLoc`, `getMultiIndex`, `getLinearIndex`, `size`, and 2 more symbols.
- **CN**: 围绕 `getLoc`, `getMultiIndex`, `getLinearIndex`, `size`, and 2 more symbols 实现具体逻辑。

### Lines 179-186
```cpp
  SmallVector<Value> strides =
      computeStrides(loc, rewriter, op.getDynamicBasis(), staticBasis,
                     /*knownNonNegative=*/op.getDisjoint());

  // Broadcast strides to match the index type (needed for vector types).
  for (Value &stride : strides)
    stride = broadcastToMatchType(rewriter, loc, stride, indexType);

```
- **EN**: Implements logic around `computeStrides`, `getDisjoint`, `broadcastToMatchType`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `computeStrides`, `getDisjoint`, `broadcastToMatchType` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 187-200
```cpp
  SmallVector<std::pair<Value, int64_t>> scaledValues;
  scaledValues.reserve(numIndexes);

  // Note: strides doesn't contain a value for the final element (stride 1)
  // and everything else lines up. We use the "mutable" accessor so we can get
  // our hands on an `OpOperand&` for the loop invariant counting function.
  for (auto [stride, idxOp] :
       llvm::zip_equal(strides, llvm::drop_end(op.getMultiIndexMutable()))) {
    Value scaledIdx = arith::MulIOp::create(rewriter, loc, idxOp.get(), stride,
                                            arith::IntegerOverflowFlags::nsw);
    int64_t numHoistableLoops = numEnclosingInvariantLoops(idxOp);
    scaledValues.emplace_back(scaledIdx, numHoistableLoops);
  }
  scaledValues.emplace_back(
```
- **EN**: Implements logic around `reserve`, `zip_equal`, `create`, `numEnclosingInvariantLoops`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `reserve`, `zip_equal`, `create`, `numEnclosingInvariantLoops`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 201-208
```cpp
      multiIndex.back(),
      numEnclosingInvariantLoops(op.getMultiIndexMutable()[numIndexes - 1]));

  // Sort by how many enclosing loops there are, ties implicitly broken by
  // size of the stride.
  llvm::stable_sort(scaledValues,
                    [&](auto l, auto r) { return l.second > r.second; });

```
- **EN**: Implements logic around `back`, `numEnclosingInvariantLoops`, `stable_sort`.
- **CN**: 围绕 `back`, `numEnclosingInvariantLoops`, `stable_sort` 实现具体逻辑。

### Lines 209-218
```cpp
  Value result = scaledValues.front().first;
  for (auto [scaledValue, numHoistableLoops] : llvm::drop_begin(scaledValues)) {
    std::ignore = numHoistableLoops;
    result = arith::AddIOp::create(rewriter, loc, result, scaledValue,
                                   arith::IntegerOverflowFlags::nsw);
  }
  rewriter.replaceOp(op, result);
  return success();
}

```
- **EN**: Implements logic around `front`, `drop_begin`, `create`, `replaceOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `front`, `drop_begin`, `create`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 219-228
```cpp
namespace {
struct LowerDelinearizeIndexOps
    : public OpRewritePattern<AffineDelinearizeIndexOp> {
  using OpRewritePattern<AffineDelinearizeIndexOp>::OpRewritePattern;
  LogicalResult matchAndRewrite(AffineDelinearizeIndexOp op,
                                PatternRewriter &rewriter) const override {
    return affine::lowerAffineDelinearizeIndexOp(rewriter, op);
  }
};

```
- **EN**: Introduces declarations for `LowerDelinearizeIndexOps`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerDelinearizeIndexOps` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 229-236
```cpp
struct LowerLinearizeIndexOps final : OpRewritePattern<AffineLinearizeIndexOp> {
  using OpRewritePattern::OpRewritePattern;
  LogicalResult matchAndRewrite(AffineLinearizeIndexOp op,
                                PatternRewriter &rewriter) const override {
    return affine::lowerAffineLinearizeIndexOp(rewriter, op);
  }
};

```
- **EN**: Introduces declarations for `LowerLinearizeIndexOps`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LowerLinearizeIndexOps` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 237-250
```cpp
class ExpandAffineIndexOpsPass
    : public affine::impl::AffineExpandIndexOpsBase<ExpandAffineIndexOpsPass> {
public:
  ExpandAffineIndexOpsPass() = default;

  void runOnOperation() override {
    MLIRContext *context = &getContext();
    RewritePatternSet patterns(context);
    populateAffineExpandIndexOpsPatterns(patterns);
    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
      return signalPassFailure();
  }
};

```
- **EN**: Introduces declarations for `ExpandAffineIndexOpsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ExpandAffineIndexOpsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 251-258
```cpp
} // namespace

void mlir::affine::populateAffineExpandIndexOpsPatterns(
    RewritePatternSet &patterns) {
  patterns.insert<LowerDelinearizeIndexOps, LowerLinearizeIndexOps>(
      patterns.getContext());
}

```
- **EN**: Implements logic around `populateAffineExpandIndexOpsPatterns`, `LowerLinearizeIndexOps>`, `getContext`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `populateAffineExpandIndexOpsPatterns`, `LowerLinearizeIndexOps>`, `getContext` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 259-261
```cpp
std::unique_ptr<Pass> mlir::affine::createAffineExpandIndexOpsPass() {
  return std::make_unique<ExpandAffineIndexOpsPass>();
}
```
- **EN**: Implements logic around `createAffineExpandIndexOpsPass`, `make_unique`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `createAffineExpandIndexOpsPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/Transforms/Transforms.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
