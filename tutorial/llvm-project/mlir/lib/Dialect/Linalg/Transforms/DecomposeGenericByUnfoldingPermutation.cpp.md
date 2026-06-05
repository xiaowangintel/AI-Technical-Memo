# DecomposeGenericByUnfoldingPermutation.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/DecomposeGenericByUnfoldingPermutation.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `DecomposeGenericByUnfoldingPermutation`.
  - **CN**: 实现 Linalg 方言中围绕 `DecomposeGenericByUnfoldingPermutation` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- DecomposeGenericByUnfoldingPermutation.cpp                   -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-16
```cpp
//
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include <map>
#include <utility>

using namespace mlir;
using namespace mlir::linalg;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `map`, `utility`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `map`, `utility`。

### Lines 17-23
```cpp
namespace {

/// This pattern decomposes the input operand(s) of a linalg.generic that has
/// a `transpose`, `broadcast`, or a mixture of two, into explicit transpose
/// and broadcast. Having them folded into the linalg.generic is a good
/// optimization but sometimes we may want to unwrap, i.e., `unfold` them as
/// explicit transpose and broadcast. This rewrite pattern helps do it for
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 24-30
```cpp
/// each input operand. This is useful for instance when trying to recognize
/// named ops.
///
/// The transpose, broadcast, or mixture of both, are expressed in the affine
/// map of the operand. Technically it is essentially `projected permutation`.
///
///  Example
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 31-37
```cpp
///
/// ```mlir
///
/// #projection = affine_map<(d0, d1, d2, d3, d4) -> (d2, d3, d1)>
/// #identity   = affine_map<(d0, d1, d2, d3, d4) -> (d0, d1, d2, d3, d4)>
/// ...
///    %res = linalg.generic
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 38-44
```cpp
///       { indexing_maps = [#projection, #identity, #identity],
///       iterator_types = ["parallel", "parallel", "parallel",
///                         "parallel", "parallel"]}
///       ins(%x, %y : tensor<7x8x9xf32>, tensor<5x9x7x8x10xf32>)
///       outs(%z : tensor<5x9x7x8x10xf32>) {
///         ^bb0(%in: f32, %in_1: f32, %out: f32):
///              %div = arith.divf %in, %in_1 : f32
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 45-51
```cpp
///              linalg.yield %div : f32
///    } -> tensor<5x9x7x8x10xf32>
/// ```
///
/// In the above IR operand `%x` map is a projected-permutation. This can be
/// unfolded as:
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 52-58
```cpp
/// ```mlir
///   ...
///   %x_trans = linalg.transpose
///                   ins(%x : tensor<7x8x9xf32>)
///                   outs(%e1 : tensor<9x7x8xf32>) permutation = [2, 0, 1]
///   ...
///   %x_trans_bc = linalg.broadcast
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 59-65
```cpp
///                   ins(%x_trans : tensor<9x7x8xf32>)
///                   outs(%e2 : tensor<5x9x7x8x10xf32>) dimensions = [0, 4]
///   %2 = linalg.div
///           ins(%x_trans_bc, %y :
///                  tensor<5x9x7x8x10xf32>, tensor<5x9x7x8x10xf32>)
///           outs(%arg2 : tensor<5x9x7x8x10xf32>) -> tensor<5x9x7x8x10xf32>
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 66-72
```cpp
/// Note that linalg.generic has been 'specialized' to linalg.div.
///
/// To unfold it, it is more optimal to transpose first and then do the
/// broadcast. However, if transpose is done first, the permutation map needs
/// to be expressed in terms of reduced dimension as broadcast hasn't happened
/// yet. Also, the broadcast dimensions in a linalg.generic come from other
/// operands (those not broadcasted along that particular dimension). We work
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 73-82
```cpp
/// this out by computing the convex-polyhedron shape of the linalg.generic
/// iteration space from shapes of all the operands, both inputs and outputs.
///
struct DecomposeProjectedPermutation : public OpRewritePattern<GenericOp> {
  using OpRewritePattern<GenericOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(GenericOp genericOp,
                                PatternRewriter &rewriter) const override;
};

```
- **EN**: Introduces declarations for `DecomposeProjectedPermutation`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `DecomposeProjectedPermutation` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 83-91
```cpp
/// For the given `map`, determine what dimensions are transposed and what
/// dimensions are broadcasted.
/// Returns :
///   transpose-permutation, broadcast-dimensions` (empty if not needed)
///
std::pair<SmallVector<int64_t>, SmallVector<int64_t>>
computeTransposeBroadcast(AffineMap &map) {
  assert(map.isProjectedPermutation(false) && "not a projection");

```
- **EN**: Implements logic around `computeTransposeBroadcast`, `assert`.
- **CN**: 围绕 `computeTransposeBroadcast`, `assert` 实现具体逻辑。

### Lines 92-101
```cpp
  // As the map is a projection it likely operates on a smaller set of
  // dimensions as far as the transpose is concerned (rest are broadcast).
  int64_t minorSize = map.getNumResults();

  SmallVector<int64_t> minorResult;
  for (int64_t i = 0; i < minorSize; ++i) {
    auto expr = cast<AffineDimExpr>(map.getResults()[i]);
    minorResult.push_back(expr.getPosition());
  }

```
- **EN**: Implements logic around `getNumResults`, `getResults`, `push_back`.
- **CN**: 围绕 `getNumResults`, `getResults`, `push_back` 实现具体逻辑。

### Lines 102-115
```cpp
  // If dims are not monotonically increasing then transpose is present.
  SmallVector<int64_t> sortedResMap(minorResult);
  llvm::sort(sortedResMap);
  bool hasTranspose = !std::equal(minorResult.begin(), minorResult.end(),
                                  sortedResMap.begin(), sortedResMap.end());

  // Walk the sorted map result to determine which dimensions are broadcasted.
  SmallVector<int64_t> broadcast;
  for (int64_t i = 0, j = 0; i < map.getNumInputs(); ++i) {
    if (j < minorSize && sortedResMap[j] == i) {
      j++;
      continue;
    }
    broadcast.push_back(i);
```
- **EN**: Implements logic around `sortedResMap`, `sort`, `equal`, `begin`, and 2 more symbols.
- **CN**: 围绕 `sortedResMap`, `sort`, `equal`, `begin`, and 2 more symbols 实现具体逻辑。

### Lines 116-129
```cpp
  }

  SmallVector<int64_t> permutation;
  if (hasTranspose) {
    // Consider an operand `x : tensor<7x8x9>` of a genericOp that has
    // affine map `affine_map<(d0, d1, d2, d3, d4) -> (d2, d3, d1)>`
    // `x`s access is both transposed and broadcast. But when specifying
    // the `linalg.transpose(x : tensor<7x8x9>)` the dimensions need to be
    // specified as `affine_map<(d0,d1,d2) -> (d1, d2, d0)` instead of
    // refering to d3, d4. Therefore, re-base the transpose dimensions so
    // that they start from d0.
    permutation.resize(minorSize);
    std::map<int64_t, int64_t> minorMap;
    for (int64_t i = 0; i < minorSize; ++i)
```
- **EN**: Implements logic around `resize`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `resize` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 130-136
```cpp
      minorMap.insert({sortedResMap[i], i});

    // Re-map the dimensions.
    SmallVector<int64_t> remappedResult(minorSize);
    for (int64_t i = 0; i < minorSize; ++i)
      remappedResult[i] = minorMap[minorResult[i]];

```
- **EN**: Implements logic around `insert`, `remappedResult`.
- **CN**: 围绕 `insert`, `remappedResult` 实现具体逻辑。

### Lines 137-144
```cpp
    /// Calculate the permutation for the transpose.
    for (unsigned i = 0; i < minorSize; ++i) {
      permutation[remappedResult[i]] = i;
    }
  }
  return {permutation, broadcast};
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 145-158
```cpp
LogicalResult DecomposeProjectedPermutation::matchAndRewrite(
    GenericOp op, PatternRewriter &rewriter) const {
  if (!op.hasPureTensorSemantics() || op.isSingleInputOutput() ||
      op.isSingleYieldOp() || !op.isAllParallelLoops())
    return failure();

  // If the map of an operand is not a `projected permutation` then
  // it cannot be decomposed to mere transpose and broadcast.
  // The requirement that all maps be `projected permutation` may be
  // over-restrictive but since we need to determine shape of the
  // iteration space as well, reject if any map violates assumption.
  for (auto &opOperand : op->getOpOperands()) {
    auto map = op.getMatchingIndexingMap(&opOperand);
    if (!map.isProjectedPermutation(false))
```
- **EN**: Implements logic around `matchAndRewrite`, `hasPureTensorSemantics`, `isSingleYieldOp`, `failure`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `hasPureTensorSemantics`, `isSingleYieldOp`, `failure`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 159-171
```cpp
      return failure();
  }

  // Decomposing linalg.generic involves creating `tensor.empty`
  // which can have dynamic shapes but then we would have to work
  // out which operand can supply that runtime-value (tensor.dim).
  // Leaving it as a future TODO.
  if (llvm::any_of(op->getOpOperands(), [](OpOperand &oper) {
        auto opType = cast<RankedTensorType>(oper.get().getType());
        return ShapedType::isDynamicShape(opType.getShape());
      }))
    return failure();

```
- **EN**: Implements logic around `failure`, `any_of`, `get`, `isDynamicShape`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `any_of`, `get`, `isDynamicShape` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 172-178
```cpp
  auto outputShape = op.getStaticLoopRanges();

  auto loc = op.getLoc();
  bool isChanged = false;
  SmallVector<Value> newInitValues = op.getDpsInputs();
  SmallVector<AffineMap> newMap = op.getIndexingMapsArray();

```
- **EN**: Implements logic around `getStaticLoopRanges`, `getLoc`, `getDpsInputs`, `getIndexingMapsArray`.
- **CN**: 围绕 `getStaticLoopRanges`, `getLoc`, `getDpsInputs`, `getIndexingMapsArray` 实现具体逻辑。

### Lines 179-185
```cpp
  // Walk over each input operand and unfold if it is transposed, broadcast
  // or mix of two via operand's affine-map.
  for (int64_t i = 0; i < op.getNumDpsInputs(); ++i) {
    auto &map = newMap[i];
    auto inputRTType = cast<RankedTensorType>(newInitValues[i].getType());
    auto elType = inputRTType.getElementType();

```
- **EN**: Implements logic around `getNumDpsInputs`, `getType`, `getElementType`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getNumDpsInputs`, `getType`, `getElementType` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 186-194
```cpp
    /// Nothing to do if map is already an identity.
    if (map.isIdentity())
      continue;

    auto [permutation, broadcastedDims] = computeTransposeBroadcast(map);

    // Does it need transpose?
    if (!permutation.empty()) {
      /// linalg.transpose permutes the dimensions of input using
```
- **EN**: Implements logic around `isIdentity`, `computeTransposeBroadcast`, `empty`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isIdentity`, `computeTransposeBroadcast`, `empty` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 195-202
```cpp
      /// rule: dim(result, i) = dim(input, permutation[i])
      SmallVector<int64_t> transposedShape(map.getNumResults());
      for (int64_t i = 0; i < map.getNumResults(); ++i)
        transposedShape[i] = inputRTType.getShape()[permutation[i]];

      Value emptyTensor =
          tensor::EmptyOp::create(rewriter, loc, transposedShape, elType);

```
- **EN**: Implements logic around `transposedShape`, `getNumResults`, `getShape`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transposedShape`, `getNumResults`, `getShape`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 203-214
```cpp
      auto transposeOp = TransposeOp::create(rewriter, loc, newInitValues[i],
                                             emptyTensor, permutation);
      newInitValues[i] = transposeOp->getResult(0);
      isChanged = true;
    }

    // Does it require broadcast?
    if (!broadcastedDims.empty()) {
      assert(!broadcastedDims.empty() && "should have non size broadcast");
      Value emptyTensor = tensor::EmptyOp::create(rewriter, loc, outputShape,
                                                  inputRTType.getElementType());

```
- **EN**: Implements logic around `create`, `getResult`, `empty`, `assert`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `empty`, `assert`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 215-223
```cpp
      auto broadcastOp = linalg::BroadcastOp::create(
          rewriter, loc, newInitValues[i], emptyTensor, broadcastedDims);

      newInitValues[i] = broadcastOp->getResult(0);
      isChanged = true;
    }
    newMap[i] = rewriter.getMultiDimIdentityMap(map.getNumDims());
  }

```
- **EN**: Implements logic around `create`, `getResult`, `getMultiDimIdentityMap`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `getMultiDimIdentityMap` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 224-237
```cpp
  if (!isChanged)
    return failure();

  SmallVector<Value> operands = op->getOperands();
  ValueRange operandsRef(operands);

  auto newOp = linalg::GenericOp::create(
      rewriter,
      /*location=*/op.getLoc(),
      /*resultTensorTypes=*/op->getResultTypes(),
      /*inputs=*/newInitValues,
      /*outputs=*/operandsRef.drop_front(op.getNumDpsInputs()),
      /*indexingMaps=*/newMap,
      /*iteratorTypes=*/op.getIteratorTypesArray());
```
- **EN**: Implements logic around `failure`, `getOperands`, `operandsRef`, `create`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `getOperands`, `operandsRef`, `create`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 238-244
```cpp
  newOp.getRegion().takeBody(op->getRegion(0));
  rewriter.replaceOp(op, newOp->getResults());
  return success();
}

} // namespace

```
- **EN**: Implements logic around `getRegion`, `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getRegion`, `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 245-248
```cpp
void mlir::linalg::populateDecomposeProjectedPermutationPatterns(
    RewritePatternSet &patterns) {
  patterns.insert<DecomposeProjectedPermutation>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateDecomposeProjectedPermutationPatterns`, `insert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateDecomposeProjectedPermutationPatterns`, `insert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`
- **Standard-library headers / 标准库头文件**: `<map>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2)
