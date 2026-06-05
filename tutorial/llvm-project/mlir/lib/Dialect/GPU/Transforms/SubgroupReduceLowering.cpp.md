# SubgroupReduceLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/SubgroupReduceLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements gradual lowering of `gpu.subgroup_reduce` ops.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- SubgroupReduceLowering.cpp - subgroup_reduce lowering patterns -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implements gradual lowering of `gpu.subgroup_reduce` ops.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-29
```cpp

#include "mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h"
#include "mlir/Dialect/AMDGPU/Utils/Chipset.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Transforms/Passes.h"
#include "mlir/Dialect/GPU/Utils/GPUUtils.h"
#include "mlir/Dialect/LLVMIR/ROCDLDialect.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
#include <cstdint>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`。

### Lines 30-39
```cpp
using namespace mlir;

namespace {

/// Example, assumes `maxShuffleBitwidth` equal to 32:
/// ```
/// %a = gpu.subgroup_reduce add %x : (vector<3xf16>) -> vector<3xf16>
///  ==>
/// %v0 = arith.constant dense<0.0> : vector<3xf16>
/// %e0 = vector.extract_strided_slice %x
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 40-53
```cpp
///   {offsets = [0], sizes = [2], strides = [1}: vector<3xf32> to vector<2xf32>
/// %r0 = gpu.subgroup_reduce add %e0 : (vector<2xf16>) -> vector<2xf16>
/// %v1 = vector.insert_strided_slice %r0, %v0
///   {offsets = [0], strides = [1}: vector<2xf32> into vector<3xf32>
/// %e1 = vector.extract %x[2] : f16 from vector<2xf16>
/// %r1 = gpu.subgroup_reduce add %e1 : (f16) -> f16
/// %a  = vector.insert %r1, %v1[2] : f16 into vector<3xf16>
/// ```
struct BreakDownSubgroupReduce final : OpRewritePattern<gpu::SubgroupReduceOp> {
  BreakDownSubgroupReduce(MLIRContext *ctx, unsigned maxShuffleBitwidth,
                          PatternBenefit benefit)
      : OpRewritePattern(ctx, benefit), maxShuffleBitwidth(maxShuffleBitwidth) {
  }

```
- **EN**: Introduces declarations for `BreakDownSubgroupReduce`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `BreakDownSubgroupReduce` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 54-70
```cpp
  LogicalResult matchAndRewrite(gpu::SubgroupReduceOp op,
                                PatternRewriter &rewriter) const override {
    auto vecTy = dyn_cast<VectorType>(op.getType());
    if (!vecTy || vecTy.getNumElements() < 2)
      return rewriter.notifyMatchFailure(op, "not a multi-element reduction");

    assert(vecTy.getRank() == 1 && "Unexpected vector type");
    assert(!vecTy.isScalable() && "Unexpected vector type");

    Type elemTy = vecTy.getElementType();
    unsigned elemBitwidth = elemTy.getIntOrFloatBitWidth();
    if (elemBitwidth >= maxShuffleBitwidth)
      return rewriter.notifyMatchFailure(
          op, llvm::formatv("element type too large ({0}), cannot break down "
                            "into vectors of bitwidth {1} or less",
                            elemBitwidth, maxShuffleBitwidth));

```
- **EN**: Implements logic around `matchAndRewrite`, `dyn_cast`, `getNumElements`, `notifyMatchFailure`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `dyn_cast`, `getNumElements`, `notifyMatchFailure`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 71-83
```cpp
    unsigned elementsPerShuffle = maxShuffleBitwidth / elemBitwidth;
    assert(elementsPerShuffle >= 1);

    unsigned numNewReductions =
        llvm::divideCeil(vecTy.getNumElements(), elementsPerShuffle);
    assert(numNewReductions >= 1);
    if (numNewReductions == 1)
      return rewriter.notifyMatchFailure(op, "nothing to break down");

    Location loc = op.getLoc();
    Value res =
        arith::ConstantOp::create(rewriter, loc, rewriter.getZeroAttr(vecTy));

```
- **EN**: Implements logic around `assert`, `divideCeil`, `notifyMatchFailure`, `getLoc`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `assert`, `divideCeil`, `notifyMatchFailure`, `getLoc`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 84-100
```cpp
    for (unsigned i = 0; i != numNewReductions; ++i) {
      int64_t startIdx = i * elementsPerShuffle;
      int64_t endIdx =
          std::min(startIdx + elementsPerShuffle, vecTy.getNumElements());
      int64_t numElems = endIdx - startIdx;

      Value extracted;
      if (numElems == 1) {
        extracted =
            vector::ExtractOp::create(rewriter, loc, op.getValue(), startIdx);
      } else {
        extracted = vector::ExtractStridedSliceOp::create(
            rewriter, loc, op.getValue(), /*offsets=*/startIdx,
            /*sizes=*/numElems,
            /*strides=*/1);
      }

```
- **EN**: Implements logic around `min`, `create`, `getValue`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `min`, `create`, `getValue` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 101-112
```cpp
      Value reduce = gpu::SubgroupReduceOp::create(
          rewriter, loc, extracted, op.getOp(), op.getUniform(),
          op.getClusterSize(), op.getClusterStride());
      if (numElems == 1) {
        res = vector::InsertOp::create(rewriter, loc, reduce, res, startIdx);
        continue;
      }

      res = vector::InsertStridedSliceOp::create(
          rewriter, loc, reduce, res, /*offsets=*/startIdx, /*strides=*/1);
    }

```
- **EN**: Implements logic around `create`, `getOp`, `getClusterSize`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `create`, `getOp`, `getClusterSize` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 113-122
```cpp
    rewriter.replaceOp(op, res);
    return success();
  }

private:
  unsigned maxShuffleBitwidth = 0;
};

/// Example:
/// ```
```
- **EN**: Implements logic around `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 123-132
```cpp
/// %a = gpu.subgroup_reduce add %x : (vector<1xf32>) -> vector<1xf32>
///  ==>
/// %e0 = vector.extract %x[0] : f32 from vector<1xf32>
/// %r0 = gpu.subgroup_reduce add %e0 : (f32) -> f32
/// %a = vector.broadcast %r0 : f32 to vector<1xf32>
/// ```
struct ScalarizeSingleElementReduce final
    : OpRewritePattern<gpu::SubgroupReduceOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `ScalarizeSingleElementReduce`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ScalarizeSingleElementReduce` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 133-151
```cpp
  LogicalResult matchAndRewrite(gpu::SubgroupReduceOp op,
                                PatternRewriter &rewriter) const override {
    auto vecTy = dyn_cast<VectorType>(op.getType());
    if (!vecTy || vecTy.getNumElements() != 1)
      return rewriter.notifyMatchFailure(op, "not a single-element reduction");

    assert(vecTy.getRank() == 1 && "Unexpected vector type");
    assert(!vecTy.isScalable() && "Unexpected vector type");
    Location loc = op.getLoc();
    Value extracted =
        vector::ExtractOp::create(rewriter, loc, op.getValue(), 0);
    Value reduce = gpu::SubgroupReduceOp::create(
        rewriter, loc, extracted, op.getOp(), op.getUniform(),
        op.getClusterSize(), op.getClusterStride());
    rewriter.replaceOpWithNewOp<vector::BroadcastOp>(op, vecTy, reduce);
    return success();
  }
};

```
- **EN**: Implements logic around `matchAndRewrite`, `dyn_cast`, `getNumElements`, `notifyMatchFailure`, and 7 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `dyn_cast`, `getNumElements`, `notifyMatchFailure`, and 7 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 152-161
```cpp
struct ClusterInfo {
  unsigned clusterStride;
  unsigned clusterSize;
  unsigned subgroupSize;
};

static FailureOr<ClusterInfo>
getAndValidateClusterInfo(gpu::SubgroupReduceOp op, unsigned subgroupSize) {
  assert(llvm::isPowerOf2_32(subgroupSize));

```
- **EN**: Introduces declarations for `ClusterInfo`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ClusterInfo` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 162-177
```cpp
  std::optional<uint32_t> clusterSize = op.getClusterSize();
  assert(!clusterSize ||
         llvm::isPowerOf2_32(*clusterSize)); // Verifier should've caught this.
  if (clusterSize && *clusterSize > subgroupSize)
    return op.emitOpError()
           << "cluster size " << *clusterSize
           << " is greater than subgroup size " << subgroupSize;
  unsigned effectiveClusterSize = clusterSize.value_or(subgroupSize);

  auto clusterStride = op.getClusterStride();
  assert(llvm::isPowerOf2_32(clusterStride)); // Verifier should've caught this.
  if (clusterStride >= subgroupSize)
    return op.emitOpError()
           << "cluster stride " << clusterStride
           << " is not less than subgroup size " << subgroupSize;

```
- **EN**: Implements logic around `getClusterSize`, `assert`, `isPowerOf2_32`, `emitOpError`, and 2 more symbols.
- **CN**: 围绕 `getClusterSize`, `assert`, `isPowerOf2_32`, `emitOpError`, and 2 more symbols 实现具体逻辑。

### Lines 178-187
```cpp
  return ClusterInfo{clusterStride, effectiveClusterSize, subgroupSize};
}

/// Emits a subgroup reduction using a sequence of shuffles. Uses the `packFn`
/// and `unpackFn` to convert to the native shuffle type and to the reduction
/// type, respectively. For example, with `input` of type `f16`, `packFn` could
/// build ops to cast the value to `i32` to perform shuffles, while `unpackFn`
/// would cast it back to `f16` to perform arithmetic reduction on. Assumes that
/// the subgroup is `subgroupSize` lanes wide and divides it into clusters of
/// `clusterSize` lanes starting at lane 0 with a stride of `clusterStride` for
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 188-207
```cpp
/// lanes within a cluster, reducing all lanes in each cluster in parallel.
Value createSubgroupShuffleReduction(OpBuilder &builder, Location loc,
                                     Value input, gpu::AllReduceOperation mode,
                                     const ClusterInfo &ci,
                                     function_ref<Value(Value)> packFn,
                                     function_ref<Value(Value)> unpackFn) {
  // Lane value always stays in the original type. We use it to perform arith
  // reductions.
  Value laneVal = input;
  // Parallel reduction using butterfly shuffles.
  for (unsigned i = ci.clusterStride; i < ci.clusterStride * ci.clusterSize;
       i <<= 1) {
    Value shuffled = gpu::ShuffleOp::create(builder, loc, packFn(laneVal), i,
                                            /*width=*/ci.subgroupSize,
                                            /*mode=*/gpu::ShuffleMode::XOR)
                         .getShuffleResult();
    laneVal = vector::makeArithReduction(builder, loc,
                                         gpu::convertReductionKind(mode),
                                         laneVal, unpackFn(shuffled));
    assert(laneVal.getType() == input.getType());
```
- **EN**: Implements logic around `createSubgroupShuffleReduction`, `function_ref`, `create`, `getShuffleResult`, and 4 more symbols; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `createSubgroupShuffleReduction`, `function_ref`, `create`, `getShuffleResult`, and 4 more symbols 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 208-221
```cpp
  }

  return laneVal;
}

/// Lowers scalar gpu subgroup reductions to a series of shuffles.
struct ScalarSubgroupReduceToShuffles final
    : OpRewritePattern<gpu::SubgroupReduceOp> {
  ScalarSubgroupReduceToShuffles(MLIRContext *ctx, unsigned subgroupSize,
                                 unsigned shuffleBitwidth, bool matchClustered,
                                 PatternBenefit benefit)
      : OpRewritePattern(ctx, benefit), subgroupSize(subgroupSize),
        shuffleBitwidth(shuffleBitwidth), matchClustered(matchClustered) {}

```
- **EN**: Introduces declarations for `ScalarSubgroupReduceToShuffles`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ScalarSubgroupReduceToShuffles` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 222-231
```cpp
  LogicalResult matchAndRewrite(gpu::SubgroupReduceOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getClusterSize().has_value() != matchClustered) {
      return rewriter.notifyMatchFailure(
          op, llvm::formatv("op is {0}clustered but pattern is configured to "
                            "only match {1}clustered ops",
                            matchClustered ? "non-" : "",
                            matchClustered ? "" : "non-"));
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getClusterSize`, `notifyMatchFailure`, `formatv`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `getClusterSize`, `notifyMatchFailure`, `formatv` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 232-242
```cpp
    auto ci = getAndValidateClusterInfo(op, subgroupSize);
    if (failed(ci))
      return failure();

    Type valueTy = op.getType();
    unsigned elemBitwidth =
        getElementTypeOrSelf(valueTy).getIntOrFloatBitWidth();
    if (!valueTy.isIntOrFloat() || elemBitwidth > shuffleBitwidth)
      return rewriter.notifyMatchFailure(
          op, "value type is not a compatible scalar");

```
- **EN**: Implements logic around `getAndValidateClusterInfo`, `failed`, `failure`, `getType`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getAndValidateClusterInfo`, `failed`, `failure`, `getType`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 243-252
```cpp
    Location loc = op.getLoc();
    // Since this is already a native shuffle scalar, no packing is necessary.
    if (elemBitwidth == shuffleBitwidth) {
      auto identityFn = [](Value v) { return v; };
      rewriter.replaceOp(op, createSubgroupShuffleReduction(
                                 rewriter, loc, op.getValue(), op.getOp(), *ci,
                                 identityFn, identityFn));
      return success();
    }

```
- **EN**: Implements logic around `getLoc`, `replaceOp`, `getValue`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `replaceOp`, `getValue`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 253-267
```cpp
    auto shuffleIntType = rewriter.getIntegerType(shuffleBitwidth);
    auto equivIntType = rewriter.getIntegerType(elemBitwidth);
    auto packFn = [loc, &rewriter, equivIntType,
                   shuffleIntType](Value unpackedVal) -> Value {
      auto asInt =
          arith::BitcastOp::create(rewriter, loc, equivIntType, unpackedVal);
      return arith::ExtUIOp::create(rewriter, loc, shuffleIntType, asInt);
    };
    auto unpackFn = [loc, &rewriter, equivIntType,
                     valueTy](Value packedVal) -> Value {
      auto asInt =
          arith::TruncIOp::create(rewriter, loc, equivIntType, packedVal);
      return arith::BitcastOp::create(rewriter, loc, valueTy, asInt);
    };

```
- **EN**: Implements logic around `getIntegerType`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getIntegerType`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 268-279
```cpp
    rewriter.replaceOp(
        op, createSubgroupShuffleReduction(rewriter, loc, op.getValue(),
                                           op.getOp(), *ci, packFn, unpackFn));
    return success();
  }

private:
  unsigned subgroupSize = 0;
  unsigned shuffleBitwidth = 0;
  bool matchClustered = false;
};

```
- **EN**: Implements logic around `replaceOp`, `createSubgroupShuffleReduction`, `getOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `createSubgroupShuffleReduction`, `getOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 280-298
```cpp
/// Lowers vector gpu subgroup reductions to a series of shuffles.
struct VectorSubgroupReduceToShuffles final
    : OpRewritePattern<gpu::SubgroupReduceOp> {
  VectorSubgroupReduceToShuffles(MLIRContext *ctx, unsigned subgroupSize,
                                 unsigned shuffleBitwidth, bool matchClustered,
                                 PatternBenefit benefit)
      : OpRewritePattern(ctx, benefit), subgroupSize(subgroupSize),
        shuffleBitwidth(shuffleBitwidth), matchClustered(matchClustered) {}

  LogicalResult matchAndRewrite(gpu::SubgroupReduceOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getClusterSize().has_value() != matchClustered) {
      return rewriter.notifyMatchFailure(
          op, llvm::formatv("op is {0}clustered but pattern is configured to "
                            "only match {1}clustered ops",
                            matchClustered ? "non-" : "",
                            matchClustered ? "" : "non-"));
    }

```
- **EN**: Introduces declarations for `VectorSubgroupReduceToShuffles`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `VectorSubgroupReduceToShuffles` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 299-315
```cpp
    auto ci = getAndValidateClusterInfo(op, subgroupSize);
    if (failed(ci))
      return failure();

    auto vecTy = dyn_cast<VectorType>(op.getType());
    if (!vecTy)
      return rewriter.notifyMatchFailure(op, "value type is not a vector");

    unsigned vecBitwidth =
        vecTy.getNumElements() * vecTy.getElementTypeBitWidth();
    if (vecBitwidth > shuffleBitwidth)
      return rewriter.notifyMatchFailure(
          op,
          llvm::formatv("vector type bitwidth too large ({0}), cannot lower "
                        "to shuffles of size {1}",
                        vecBitwidth, shuffleBitwidth));

```
- **EN**: Implements logic around `getAndValidateClusterInfo`, `failed`, `failure`, `dyn_cast`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getAndValidateClusterInfo`, `failed`, `failure`, `dyn_cast`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 316-335
```cpp
    unsigned elementsPerShuffle =
        shuffleBitwidth / vecTy.getElementTypeBitWidth();
    if (elementsPerShuffle * vecTy.getElementTypeBitWidth() != shuffleBitwidth)
      return rewriter.notifyMatchFailure(
          op, "shuffle bitwidth is not a multiple of the element bitwidth");

    Location loc = op.getLoc();

    // If the reduced type is smaller than the native shuffle size, extend it,
    // perform the shuffles, and extract at the end.
    auto extendedVecTy = VectorType::get(
        static_cast<int64_t>(elementsPerShuffle), vecTy.getElementType());
    Value extendedInput = op.getValue();
    if (vecBitwidth < shuffleBitwidth) {
      auto zero = arith::ConstantOp::create(
          rewriter, loc, rewriter.getZeroAttr(extendedVecTy));
      extendedInput = vector::InsertStridedSliceOp::create(
          rewriter, loc, extendedInput, zero, /*offsets=*/0, /*strides=*/1);
    }

```
- **EN**: Implements logic around `getElementTypeBitWidth`, `notifyMatchFailure`, `getLoc`, `get`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getElementTypeBitWidth`, `notifyMatchFailure`, `getLoc`, `get`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 336-350
```cpp
    auto shuffleIntType = rewriter.getIntegerType(shuffleBitwidth);
    auto shuffleVecType = VectorType::get(1, shuffleIntType);

    auto packFn = [loc, &rewriter, shuffleVecType](Value unpackedVal) -> Value {
      auto asIntVec =
          vector::BitCastOp::create(rewriter, loc, shuffleVecType, unpackedVal);
      return vector::ExtractOp::create(rewriter, loc, asIntVec, 0);
    };
    auto unpackFn = [loc, &rewriter, shuffleVecType,
                     extendedVecTy](Value packedVal) -> Value {
      auto asIntVec =
          vector::BroadcastOp::create(rewriter, loc, shuffleVecType, packedVal);
      return vector::BitCastOp::create(rewriter, loc, extendedVecTy, asIntVec);
    };

```
- **EN**: Implements logic around `getIntegerType`, `get`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getIntegerType`, `get`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 351-363
```cpp
    Value res = createSubgroupShuffleReduction(
        rewriter, loc, extendedInput, op.getOp(), *ci, packFn, unpackFn);

    if (vecBitwidth < shuffleBitwidth) {
      res = vector::ExtractStridedSliceOp::create(
          rewriter, loc, res, /*offsets=*/0, /*sizes=*/vecTy.getNumElements(),
          /*strides=*/1);
    }

    rewriter.replaceOp(op, res);
    return success();
  }

```
- **EN**: Implements logic around `createSubgroupShuffleReduction`, `getOp`, `create`, `getNumElements`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createSubgroupShuffleReduction`, `getOp`, `create`, `getNumElements`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 364-383
```cpp
private:
  unsigned subgroupSize = 0;
  unsigned shuffleBitwidth = 0;
  bool matchClustered = false;
};

static FailureOr<Value>
createSubgroupDPPReduction(PatternRewriter &rewriter, gpu::SubgroupReduceOp op,
                           Value input, gpu::AllReduceOperation mode,
                           const ClusterInfo &ci, amdgpu::Chipset chipset) {
  Location loc = op.getLoc();
  Value dpp;
  Value res = input;
  constexpr int allRows = 0xf;
  constexpr int allBanks = 0xf;
  const bool boundCtrl = true;
  if (ci.clusterSize >= 2) {
    // Perform reduction between all lanes N <-> N+1.
    dpp = amdgpu::DPPOp::create(
        rewriter, loc, res.getType(), res, res, amdgpu::DPPPerm::quad_perm,
```
- **EN**: Implements logic around `createSubgroupDPPReduction`, `getLoc`, `create`, `getType`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `createSubgroupDPPReduction`, `getLoc`, `create`, `getType` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 384-403
```cpp
        rewriter.getI32ArrayAttr({1, 0, 3, 2}), allRows, allBanks, boundCtrl);
    res = vector::makeArithReduction(rewriter, loc,
                                     gpu::convertReductionKind(mode), res, dpp);
  }

  if (ci.clusterSize >= 4) {
    // Perform reduction between all lanes N <-> N+2.
    dpp = amdgpu::DPPOp::create(
        rewriter, loc, res.getType(), res, res, amdgpu::DPPPerm::quad_perm,
        rewriter.getI32ArrayAttr({2, 3, 0, 1}), allRows, allBanks, boundCtrl);
    res = vector::makeArithReduction(rewriter, loc,
                                     gpu::convertReductionKind(mode), res, dpp);
  }
  if (ci.clusterSize >= 8) {
    // Perform reduction between all lanes N <-> 7-N,
    // e.g lane[0] <-> lane[7], lane[1] <-> lane[6]..., lane[3] <-> lane[4].
    dpp = amdgpu::DPPOp::create(rewriter, loc, res.getType(), res, res,
                                amdgpu::DPPPerm::row_half_mirror,
                                rewriter.getUnitAttr(), allRows, allBanks,
                                boundCtrl);
```
- **EN**: Implements logic around `getI32ArrayAttr`, `makeArithReduction`, `convertReductionKind`, `create`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getI32ArrayAttr`, `makeArithReduction`, `convertReductionKind`, `create`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 404-423
```cpp
    res = vector::makeArithReduction(rewriter, loc,
                                     gpu::convertReductionKind(mode), res, dpp);
  }
  if (ci.clusterSize >= 16) {
    // Perform reduction between all lanes N <-> 15-N,
    // e.g lane[0] <-> lane[15], lane[1] <-> lane[14]..., lane[7] <-> lane[8].
    dpp = amdgpu::DPPOp::create(
        rewriter, loc, res.getType(), res, res, amdgpu::DPPPerm::row_mirror,
        rewriter.getUnitAttr(), allRows, allBanks, boundCtrl);
    res = vector::makeArithReduction(rewriter, loc,
                                     gpu::convertReductionKind(mode), res, dpp);
  }
  if (ci.clusterSize >= 32) {
    if (chipset.majorVersion <= 9) {
      // Broadcast last value from each row to next row.
      // Use row mask to avoid polluting row 0 (and row 2 if wave-64).
      dpp = amdgpu::DPPOp::create(rewriter, loc, res.getType(), res, res,
                                  amdgpu::DPPPerm::row_bcast_15,
                                  rewriter.getUnitAttr(), 0xa, allBanks,
                                  /*bound_ctrl*/ false);
```
- **EN**: Implements logic around `makeArithReduction`, `convertReductionKind`, `create`, `getType`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `makeArithReduction`, `convertReductionKind`, `create`, `getType`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 424-443
```cpp
      res = vector::makeArithReduction(
          rewriter, loc, gpu::convertReductionKind(mode), res, dpp);

      // For subgroupSize = 64, at this point lanes [16, 32) contain the full
      // reduction over lanes [0, 32), but lanes [0, 16) do not. Similarly,
      // lanes [48, 64) contain the full reduction over lanes [32, 64), but
      // lanes [32, 48) do not.
      //
      // If subgroup size is 64 and cluster size is 64, we don't need lanes [0,
      // 16) and [32, 48) to have the correct cluster-32 reduction values at
      // this point, because only lane 63's value will ultimately be read in
      // this full-cluster case.
      //
      // If subgroup size is 64 and cluster size is 32, we need to ensure that
      // lanes [0, 16) and [32, 48) have the correct final cluster-32 reduction
      // values (subgroup_reduce guarantees that all lanes within each cluster
      // contain the final reduction value). We do this by broadcasting lane
      // 31's value to lanes [0, 16) and lanes 63's value to lanes [32, 48).
      //
      // See https://gpuopen.com/learn/amd-gcn-assembly-cross-lane-operations
```
- **EN**: Implements logic around `makeArithReduction`, `convertReductionKind`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `makeArithReduction`, `convertReductionKind` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 444-463
```cpp
      // for an illustration of how this within-cluster broadcast works with a
      // swizzle.
      if (ci.subgroupSize == 64 && ci.clusterSize == 32) {
        res =
            amdgpu::SwizzleBitModeOp::create(rewriter, loc, res, /*and_mask=*/0,
                                             /*or_mask=*/31,
                                             /*xor_mask=*/0);
      }
    } else if (chipset.majorVersion <= 12) {
      // Use a permute lane to cross rows (row 1 <-> row 0, row 3 <-> row 2).
      Value uint32Max = arith::ConstantOp::create(
          rewriter, loc, rewriter.getI32Type(), rewriter.getI32IntegerAttr(-1));
      dpp = ROCDL::PermlaneX16Op::create(rewriter, loc, res.getType(), res, res,
                                         uint32Max, uint32Max,
                                         /*fi=*/true,
                                         /*boundControl=*/false);
      res = vector::makeArithReduction(
          rewriter, loc, gpu::convertReductionKind(mode), res, dpp);
    } else {
      return rewriter.notifyMatchFailure(
```
- **EN**: Implements logic around `create`, `getI32Type`, `makeArithReduction`, `convertReductionKind`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `create`, `getI32Type`, `makeArithReduction`, `convertReductionKind`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 464-483
```cpp
          op, "Subgroup reduce lowering to DPP not currently supported for "
              "this device.");
    }
    if (ci.subgroupSize == 32) {
      Value lane31 = arith::ConstantOp::create(
          rewriter, loc, rewriter.getI32Type(), rewriter.getI32IntegerAttr(31));
      res =
          ROCDL::ReadlaneOp::create(rewriter, loc, res.getType(), res, lane31);
    }
  }
  if (ci.clusterSize >= 64) {
    if (chipset.majorVersion <= 9) {
      // Broadcast 31st lane value to rows 2 and 3.
      dpp = amdgpu::DPPOp::create(rewriter, loc, res.getType(), res, res,
                                  amdgpu::DPPPerm::row_bcast_31,
                                  rewriter.getUnitAttr(), 0xf, allBanks,
                                  /*bound_ctrl*/ true);
      res = vector::makeArithReduction(
          rewriter, loc, gpu::convertReductionKind(mode), dpp, res);
      // Obtain reduction from last rows, the previous rows are polluted.
```
- **EN**: Implements logic around `create`, `getI32Type`, `getUnitAttr`, `makeArithReduction`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `create`, `getI32Type`, `getUnitAttr`, `makeArithReduction`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 484-503
```cpp
      Value lane63 = arith::ConstantOp::create(
          rewriter, loc, rewriter.getI32Type(), rewriter.getI32IntegerAttr(63));
      res =
          ROCDL::ReadlaneOp::create(rewriter, loc, res.getType(), res, lane63);

    } else if (chipset.majorVersion <= 12) {
      // Assume reduction across 32 lanes has been done.
      // Perform final reduction manually by summing values in lane 0 and
      // lane 32.
      Value lane31 = arith::ConstantOp::create(
          rewriter, loc, rewriter.getI32Type(), rewriter.getI32IntegerAttr(31));
      Value lane63 = arith::ConstantOp::create(
          rewriter, loc, rewriter.getI32Type(), rewriter.getI32IntegerAttr(63));
      lane31 =
          ROCDL::ReadlaneOp::create(rewriter, loc, res.getType(), res, lane31);
      lane63 =
          ROCDL::ReadlaneOp::create(rewriter, loc, res.getType(), res, lane63);
      res = vector::makeArithReduction(
          rewriter, loc, gpu::convertReductionKind(mode), lane31, lane63);
    } else {
```
- **EN**: Implements logic around `create`, `getI32Type`, `makeArithReduction`, `convertReductionKind`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `create`, `getI32Type`, `makeArithReduction`, `convertReductionKind` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 504-513
```cpp
      return rewriter.notifyMatchFailure(
          op, "Subgroup reduce lowering to DPP not currently supported for "
              "this device.");
    }
  }
  assert(res.getType() == input.getType());
  return res;
}

/// Collect a set of patterns to lower `gpu.subgroup_reduce` into `amdgpu.dpp`
```
- **EN**: Implements logic around `notifyMatchFailure`, `assert`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `notifyMatchFailure`, `assert` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 514-523
```cpp
/// ops over scalar types. Assumes that the subgroup has
/// `subgroupSize` lanes. Applicable only to AMD GPUs.
struct ScalarSubgroupReduceToDPP final
    : OpRewritePattern<gpu::SubgroupReduceOp> {
  ScalarSubgroupReduceToDPP(MLIRContext *ctx, unsigned subgroupSize,
                            bool matchClustered, amdgpu::Chipset chipset,
                            PatternBenefit benefit)
      : OpRewritePattern(ctx, benefit), subgroupSize(subgroupSize),
        matchClustered(matchClustered), chipset(chipset) {}

```
- **EN**: Introduces declarations for `ScalarSubgroupReduceToDPP`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ScalarSubgroupReduceToDPP` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 524-536
```cpp
  LogicalResult matchAndRewrite(gpu::SubgroupReduceOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getClusterSize().has_value() != matchClustered) {
      return rewriter.notifyMatchFailure(
          op, llvm::formatv("op is {0}clustered but pattern is configured to "
                            "only match {1}clustered ops",
                            matchClustered ? "non-" : "",
                            matchClustered ? "" : "non-"));
    }
    auto ci = getAndValidateClusterInfo(op, subgroupSize);
    if (failed(ci))
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getClusterSize`, `notifyMatchFailure`, `formatv`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `getClusterSize`, `notifyMatchFailure`, `formatv`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 537-546
```cpp
    if (ci->clusterStride != 1)
      return rewriter.notifyMatchFailure(
          op, "Subgroup reductions using DPP are currently only available for "
              "clusters of contiguous lanes.");

    Type valueTy = op.getType();
    if (!valueTy.isIntOrFloat())
      return rewriter.notifyMatchFailure(
          op, "Value type is not a compatible scalar.");

```
- **EN**: Implements logic around `notifyMatchFailure`, `getType`, `isIntOrFloat`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `notifyMatchFailure`, `getType`, `isIntOrFloat` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 547-556
```cpp
    FailureOr<Value> dpp = createSubgroupDPPReduction(
        rewriter, op, op.getValue(), op.getOp(), *ci, chipset);
    if (failed(dpp))
      return failure();

    rewriter.replaceOp(op, dpp.value());
    return success();
  }

private:
```
- **EN**: Implements logic around `createSubgroupDPPReduction`, `getValue`, `failed`, `failure`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createSubgroupDPPReduction`, `getValue`, `failed`, `failure`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 557-570
```cpp
  unsigned subgroupSize = 0;
  bool matchClustered = false;
  amdgpu::Chipset chipset;
};
} // namespace

void mlir::populateGpuBreakDownSubgroupReducePatterns(
    RewritePatternSet &patterns, unsigned maxShuffleBitwidth,
    PatternBenefit benefit) {
  patterns.add<BreakDownSubgroupReduce>(patterns.getContext(),
                                        maxShuffleBitwidth, benefit);
  patterns.add<ScalarizeSingleElementReduce>(patterns.getContext(), benefit);
}

```
- **EN**: Implements logic around `populateGpuBreakDownSubgroupReducePatterns`, `add`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `populateGpuBreakDownSubgroupReducePatterns`, `add` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 571-586
```cpp
void mlir::populateGpuLowerSubgroupReduceToDPPPatterns(
    RewritePatternSet &patterns, unsigned subgroupSize, amdgpu::Chipset chipset,
    PatternBenefit benefit) {
  patterns.add<ScalarSubgroupReduceToDPP>(patterns.getContext(), subgroupSize,
                                          /*matchClustered=*/false, chipset,
                                          benefit);
}

void mlir::populateGpuLowerClusteredSubgroupReduceToDPPPatterns(
    RewritePatternSet &patterns, unsigned subgroupSize, amdgpu::Chipset chipset,
    PatternBenefit benefit) {
  patterns.add<ScalarSubgroupReduceToDPP>(patterns.getContext(), subgroupSize,
                                          /*matchClustered=*/true, chipset,
                                          benefit);
}

```
- **EN**: Implements logic around `populateGpuLowerSubgroupReduceToDPPPatterns`, `add`, `populateGpuLowerClusteredSubgroupReduceToDPPPatterns`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `populateGpuLowerSubgroupReduceToDPPPatterns`, `add`, `populateGpuLowerClusteredSubgroupReduceToDPPPatterns` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 587-601
```cpp
void mlir::populateGpuLowerSubgroupReduceToShufflePatterns(
    RewritePatternSet &patterns, unsigned subgroupSize,
    unsigned shuffleBitwidth, PatternBenefit benefit) {
  patterns.add<ScalarSubgroupReduceToShuffles, VectorSubgroupReduceToShuffles>(
      patterns.getContext(), subgroupSize, shuffleBitwidth,
      /*matchClustered=*/false, benefit);
}

void mlir::populateGpuLowerClusteredSubgroupReduceToShufflePatterns(
    RewritePatternSet &patterns, unsigned subgroupSize,
    unsigned shuffleBitwidth, PatternBenefit benefit) {
  patterns.add<ScalarSubgroupReduceToShuffles, VectorSubgroupReduceToShuffles>(
      patterns.getContext(), subgroupSize, shuffleBitwidth,
      /*matchClustered=*/true, benefit);
}
```
- **EN**: Implements logic around `populateGpuLowerSubgroupReduceToShufflePatterns`, `VectorSubgroupReduceToShuffles>`, `getContext`, `populateGpuLowerClusteredSubgroupReduceToShufflePatterns`.
- **CN**: 围绕 `populateGpuLowerSubgroupReduceToShufflePatterns`, `VectorSubgroupReduceToShuffles>`, `getContext`, `populateGpuLowerClusteredSubgroupReduceToShufflePatterns` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/GPU/Utils/GPUUtils.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Location.h` ... (+4 more)
- **Standard-library headers / 标准库头文件**: `<cassert>`, `<cstdint>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (8), MLIR IR core abstractions / MLIR IR 核心抽象 (4), LLVM support-library helpers / LLVM Support 库辅助功能 (2)
