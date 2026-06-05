# BlockPackMatmul.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/BlockPackMatmul.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `BlockPackMatmul`.
  - **CN**: 实现 Linalg 方言中围绕 `BlockPackMatmul` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- BlockPackMatmul.cpp - Linalg matmul block packing ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`。

### Lines 11-20
```cpp
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/SmallVector.h"

#include <optional>

namespace mlir {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/IR/PatternMatch.h`。

### Lines 21-40
```cpp
#define GEN_PASS_DEF_LINALGBLOCKPACKMATMUL
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::linalg;

/// Return constant range span or nullopt, otherwise.
static std::optional<int64_t> getConstantRange(const Range &range) {
  std::optional<int64_t> stride = getConstantIntValue(range.stride);
  if (!stride || *stride != 1)
    return std::nullopt;
  std::optional<int64_t> offset = getConstantIntValue(range.offset);
  if (!offset)
    return std::nullopt;
  std::optional<int64_t> size = getConstantIntValue(range.size);
  if (!size)
    return std::nullopt;
  return (*size - *offset);
}
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 41-54
```cpp

/// Return true if all dimensions are fully divisible by the respective tiles.
static bool validateFullTilesOnDims(linalg::LinalgOp linalgOp,
                                    ArrayRef<OpFoldResult> tiles,
                                    ArrayRef<int64_t> dims) {
  if (dims.size() != tiles.size() || tiles.empty())
    return false;

  FailureOr<ContractionDimensions> contractDims =
      inferContractionDims(linalgOp);
  if (failed(contractDims))
    return false;
  unsigned batchDimsOffset = contractDims->batch.size();

```
- **EN**: Implements logic around `validateFullTilesOnDims`, `size`, `inferContractionDims`, `failed`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `validateFullTilesOnDims`, `size`, `inferContractionDims`, `failed` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 55-65
```cpp
  // Skip the batch dimension if present.
  // Offset all dimensions accordingly.
  SmallVector<int64_t, 3> offsetDims(dims);
  for (int64_t &offsetDim : offsetDims)
    offsetDim += batchDimsOffset;

  auto tileOp = cast<TilingInterface>(linalgOp.getOperation());
  OpBuilder builder(tileOp);
  OpBuilder::InsertionGuard guard(builder);
  SmallVector<Range> iterationDomain = tileOp.getIterationDomain(builder);

```
- **EN**: Implements logic around `offsetDims`, `getOperation`, `builder`, `guard`, and 1 more symbols.
- **CN**: 围绕 `offsetDims`, `getOperation`, `builder`, `guard`, and 1 more symbols 实现具体逻辑。

### Lines 66-78
```cpp
  for (auto dim : llvm::enumerate(offsetDims)) {
    if (dim.value() >= static_cast<int64_t>(iterationDomain.size()))
      return false;

    std::optional<int64_t> tileSize = getConstantIntValue(tiles[dim.index()]);
    std::optional<int64_t> rangeOnDim =
        getConstantRange(iterationDomain[dim.value()]);

    // If the tile factor or the range are non-constant, the tile size is
    // considered to be invalid.
    if (!tileSize || !rangeOnDim)
      return false;

```
- **EN**: Implements logic around `enumerate`, `value`, `getConstantIntValue`, `getConstantRange`.
- **CN**: 围绕 `enumerate`, `value`, `getConstantIntValue`, `getConstantRange` 实现具体逻辑。

### Lines 79-96
```cpp
    // The dimension must be fully divisible by the tile.
    if (*rangeOnDim % *tileSize != 0)
      return false;
  }

  return true;
}

/// Return failure or packed matmul with one of its operands transposed.
static FailureOr<PackTransposeResult>
transposePackedMatmul(RewriterBase &rewriter, linalg::LinalgOp linalgOp,
                      linalg::PackOp packOp, AffineMap operandMap,
                      ArrayRef<unsigned> blocksStartDimPos,
                      bool transposeOuterBlocks, bool transposeInnerBlocks) {
  // TODO: Support Memref PackOp. Temporarily return failure.
  if (!packOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `transposePackedMatmul`, `hasPureTensorSemantics`, `failure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transposePackedMatmul`, `hasPureTensorSemantics`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 97-114
```cpp
  assert(operandMap.getNumDims() >= 4 &&
         "expected at least 4D prepacked matmul");
  assert(blocksStartDimPos.size() >= 2 &&
         "expected starting outer and inner block positions");

  // Bias toward innermost dimensions.
  unsigned outerBlockPos = operandMap.getNumResults() - 4;
  unsigned innerBlockPos = operandMap.getNumResults() - 2;

  // Transpose control options define the desired block and element layout.
  // Block transposition (outer dimensions) or element transposition (inner
  // dimensions) may not be necessary depending on the original matmul data
  // layout.
  bool isOuterTransposed =
      operandMap.getDimPosition(outerBlockPos) != blocksStartDimPos.end()[-2];
  bool isInnerTransposed =
      operandMap.getDimPosition(innerBlockPos) != blocksStartDimPos.back();

```
- **EN**: Implements logic around `assert`, `getNumResults`, `getDimPosition`.
- **CN**: 围绕 `assert`, `getNumResults`, `getDimPosition` 实现具体逻辑。

### Lines 115-132
```cpp
  // Transpose only the dimensions that need that to conform to the provided
  // transpotion settings.
  SmallVector<int64_t> innerPerm = {0, 1};
  if (isInnerTransposed != transposeInnerBlocks)
    innerPerm = {1, 0};
  SmallVector<int64_t> outerPerm = {0, 1};
  if (isOuterTransposed != transposeOuterBlocks)
    outerPerm = {1, 0};

  // Leave the outer dimensions, like batch, unchanged by offsetting all
  // outer dimensions permutations.
  SmallVector<int64_t> offsetPerms;
  for (auto i : llvm::seq(0u, outerBlockPos))
    offsetPerms.push_back(i);
  for (auto perm : outerPerm)
    offsetPerms.push_back(perm + outerBlockPos);
  outerPerm = offsetPerms;

```
- **EN**: Implements logic around `seq`, `push_back`.
- **CN**: 围绕 `seq`, `push_back` 实现具体逻辑。

### Lines 133-152
```cpp
  FailureOr<PackTransposeResult> packTransposedMatmul =
      packTranspose(rewriter, packOp, linalgOp,
                    /*maybeUnPackOp=*/nullptr, outerPerm, innerPerm);

  return packTransposedMatmul;
}

/// Pack a matmul operation into blocked 4D layout.
FailureOr<PackResult>
linalg::blockPackMatmul(RewriterBase &rewriter, linalg::LinalgOp linalgOp,
                        const ControlBlockPackMatmulFn &controlPackMatmul) {
  // Check to not let go the batch_matmul with extended semantic, through this
  // transform.
  if (auto *batchMatmulOp = dyn_cast<linalg::BatchMatmulOp>(&linalgOp)) {
    if (batchMatmulOp->hasUserDefinedMaps()) {
      return rewriter.notifyMatchFailure(
          *batchMatmulOp,
          "only batch_matmul ops with non-extended semantics are supported");
    }
  }
```
- **EN**: Implements logic around `packTranspose`, `blockPackMatmul`, `BatchMatmulOp>`, `hasUserDefinedMaps`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `packTranspose`, `blockPackMatmul`, `BatchMatmulOp>`, `hasUserDefinedMaps`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 153-163
```cpp

  if (linalgOp.hasPureBufferSemantics())
    return rewriter.notifyMatchFailure(linalgOp, "require tensor semantics");

  std::optional<BlockPackMatmulOptions> options = controlPackMatmul(linalgOp);
  if (!options)
    return rewriter.notifyMatchFailure(linalgOp, "invalid packing options");

  if (options->blockFactors.size() != 3)
    return rewriter.notifyMatchFailure(linalgOp, "require 3 tile factors");

```
- **EN**: Implements logic around `hasPureBufferSemantics`, `notifyMatchFailure`, `controlPackMatmul`, `size`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureBufferSemantics`, `notifyMatchFailure`, `controlPackMatmul`, `size` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 164-173
```cpp
  SmallVector<OpFoldResult> mnkTiles =
      getAsOpFoldResult(rewriter.getI64ArrayAttr(options->blockFactors));

  // If padding is disabled, make sure that dimensions can be packed cleanly.
  if (!options->allowPadding &&
      !validateFullTilesOnDims(linalgOp, mnkTiles, options->mnkOrder)) {
    return rewriter.notifyMatchFailure(linalgOp,
                                       "expect packing full tiles only");
  }

```
- **EN**: Implements logic around `getAsOpFoldResult`, `validateFullTilesOnDims`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getAsOpFoldResult`, `validateFullTilesOnDims`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 174-187
```cpp
  OpBuilder::InsertionGuard guard(rewriter);
  // The op is replaced, we need to set the insertion point after it.
  rewriter.setInsertionPointAfter(linalgOp);

  // Pack the matmul operation into blocked layout with two levels of
  // subdivision:
  //   - major 2D blocks - outer dimensions, consist of minor blocks
  //   - minor 2D blocks - inner dimensions, consist of scalar elements
  FailureOr<PackResult> packedMatmul = packMatmulGreedily(
      rewriter, linalgOp, mnkTiles, options->mnkPaddedSizesNextMultipleOf,
      options->mnkOrder);
  if (failed(packedMatmul))
    return failure();

```
- **EN**: Implements logic around `guard`, `setInsertionPointAfter`, `packMatmulGreedily`, `failed`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `guard`, `setInsertionPointAfter`, `packMatmulGreedily`, `failed`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 188-197
```cpp
  assert(packedMatmul->packOps.size() == 3 &&
         "invalid number of pack ops after matmul packing");
  assert(packedMatmul->unPackOps.size() == 1 &&
         "invalid number of unpack ops after matmul packing");

  FailureOr<ContractionDimensions> contractDims =
      inferContractionDims(packedMatmul->packedLinalgOp);
  if (failed(contractDims))
    return failure();

```
- **EN**: Implements logic around `assert`, `inferContractionDims`, `failed`, `failure`.
- **CN**: 围绕 `assert`, `inferContractionDims`, `failed`, `failure` 实现具体逻辑。

### Lines 198-209
```cpp
  auto genericOp =
      dyn_cast<linalg::GenericOp>(packedMatmul->packedLinalgOp.getOperation());
  SmallVector<AffineMap> maps = genericOp.getIndexingMapsArray();

  // Transpose LHS matrix according to the options.
  FailureOr<PackTransposeResult> packedLhs = transposePackedMatmul(
      rewriter, packedMatmul->packedLinalgOp, packedMatmul->packOps[0], maps[0],
      contractDims->m, options->lhsTransposeOuterBlocks,
      options->lhsTransposeInnerBlocks);
  if (failed(packedLhs))
    return failure();

```
- **EN**: Implements logic around `GenericOp>`, `getIndexingMapsArray`, `transposePackedMatmul`, `failed`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `GenericOp>`, `getIndexingMapsArray`, `transposePackedMatmul`, `failed`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 210-221
```cpp
  // Update results.
  packedMatmul->packOps[0] = packedLhs->transposedPackOp;
  packedMatmul->packedLinalgOp = packedLhs->transposedLinalgOp;

  // Transpose RHS matrix according to the options.
  FailureOr<PackTransposeResult> packedRhs = transposePackedMatmul(
      rewriter, packedMatmul->packedLinalgOp, packedMatmul->packOps[1], maps[1],
      contractDims->k, options->rhsTransposeOuterBlocks,
      options->rhsTransposeInnerBlocks);
  if (failed(packedRhs))
    return failure();

```
- **EN**: Implements logic around `transposePackedMatmul`, `failed`, `failure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `transposePackedMatmul`, `failed`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 222-231
```cpp
  // Update results.
  packedMatmul->packOps[1] = packedRhs->transposedPackOp;
  packedMatmul->packedLinalgOp = packedRhs->transposedLinalgOp;

  return packedMatmul;
}

namespace {
template <typename OpTy>
struct BlockPackMatmul : public OpRewritePattern<OpTy> {
```
- **EN**: Introduces declarations for `BlockPackMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BlockPackMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 232-244
```cpp
  BlockPackMatmul(MLIRContext *context, ControlBlockPackMatmulFn fun,
                  PatternBenefit benefit = 1)
      : OpRewritePattern<OpTy>(context, benefit), controlFn(std::move(fun)) {}

  LogicalResult matchAndRewrite(OpTy linalgOp,
                                PatternRewriter &rewriter) const override {
    FailureOr<PackResult> packedMatmul =
        blockPackMatmul(rewriter, linalgOp, controlFn);
    if (failed(packedMatmul))
      return failure();
    return success();
  }

```
- **EN**: Implements logic around `BlockPackMatmul`, `OpRewritePattern`, `matchAndRewrite`, `blockPackMatmul`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `BlockPackMatmul`, `OpRewritePattern`, `matchAndRewrite`, `blockPackMatmul`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 245-256
```cpp
private:
  ControlBlockPackMatmulFn controlFn;
};

template <>
struct BlockPackMatmul<linalg::GenericOp>
    : public OpRewritePattern<linalg::GenericOp> {
  BlockPackMatmul(MLIRContext *context, ControlBlockPackMatmulFn fun,
                  PatternBenefit benefit = 1)
      : OpRewritePattern<linalg::GenericOp>(context, benefit),
        controlFn(std::move(fun)) {}

```
- **EN**: Introduces declarations for `BlockPackMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `BlockPackMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 257-268
```cpp
  LogicalResult matchAndRewrite(linalg::GenericOp linalgOp,
                                PatternRewriter &rewriter) const override {
    // Match suitable generics.
    if (!linalg::isaContractionOpInterface(linalgOp)) {
      return rewriter.notifyMatchFailure(linalgOp, "not a contraction");
    }

    using MapList = ArrayRef<ArrayRef<AffineExpr>>;
    auto infer = [&](MapList m) {
      return AffineMap::inferFromExprList(m, linalgOp.getContext());
    };

```
- **EN**: Implements logic around `matchAndRewrite`, `isaContractionOpInterface`, `notifyMatchFailure`, `inferFromExprList`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `isaContractionOpInterface`, `notifyMatchFailure`, `inferFromExprList` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 269-279
```cpp
    AffineExpr i, j, k;
    bindDims(linalgOp->getContext(), i, j, k);
    SmallVector<AffineMap> maps = linalgOp.getIndexingMapsArray();

    // For now, only match simple matmuls.
    if (!(maps == infer({{i, k}, {k, j}, {i, j}}) ||
          maps == infer({{k, i}, {k, j}, {i, j}}) ||
          maps == infer({{i, k}, {j, k}, {i, j}}))) {
      return rewriter.notifyMatchFailure(linalgOp, "not a suitable matmul");
    }

```
- **EN**: Implements logic around `bindDims`, `getIndexingMapsArray`, `infer`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `bindDims`, `getIndexingMapsArray`, `infer`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 280-290
```cpp
    FailureOr<PackResult> packedMatmul =
        blockPackMatmul(rewriter, linalgOp, controlFn);
    if (failed(packedMatmul))
      return failure();
    return success();
  }

private:
  ControlBlockPackMatmulFn controlFn;
};

```
- **EN**: Implements logic around `blockPackMatmul`, `failed`, `failure`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `blockPackMatmul`, `failed`, `failure`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 291-310
```cpp
/// Convert linalg matmul ops to block layout and back.
struct LinalgBlockPackMatmul
    : public impl::LinalgBlockPackMatmulBase<LinalgBlockPackMatmul> {
  using LinalgBlockPackMatmulBase::LinalgBlockPackMatmulBase;

  void runOnOperation() override {
    Operation *op = getOperation();
    RewritePatternSet patterns(&getContext());

    ControlBlockPackMatmulFn controlFn =
        [&](linalg::LinalgOp op) -> BlockPackMatmulOptions {
      BlockPackMatmulOptions options;
      options.blockFactors = SmallVector<int64_t>{*blockFactors};
      options.allowPadding = allowPadding;
      options.mnkPaddedSizesNextMultipleOf =
          SmallVector<int64_t>{*mnkPaddedSizesNextMultipleOf};
      if (!mnkOrder.empty())
        options.mnkOrder = SmallVector<int64_t>{*mnkOrder};
      options.lhsTransposeOuterBlocks = lhsTransposeOuterBlocks;
      options.lhsTransposeInnerBlocks = lhsTransposeInnerBlocks;
```
- **EN**: Introduces declarations for `LinalgBlockPackMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `LinalgBlockPackMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 311-322
```cpp
      options.rhsTransposeOuterBlocks = rhsTransposeOuterBlocks;
      options.rhsTransposeInnerBlocks = rhsTransposeInnerBlocks;
      return options;
    };

    linalg::populateBlockPackMatmulPatterns(patterns, controlFn);
    if (failed(applyPatternsGreedily(op, std::move(patterns))))
      return signalPassFailure();
  }
};
} // namespace

```
- **EN**: Implements logic around `populateBlockPackMatmulPatterns`, `failed`, `signalPassFailure`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateBlockPackMatmulPatterns`, `failed`, `signalPassFailure` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 323-329
```cpp
void linalg::populateBlockPackMatmulPatterns(
    RewritePatternSet &patterns, const ControlBlockPackMatmulFn &controlFn) {
  patterns.add<BlockPackMatmul<linalg::GenericOp>,
               BlockPackMatmul<linalg::MatmulOp>,
               BlockPackMatmul<linalg::BatchMatmulOp>>(patterns.getContext(),
                                                       controlFn);
}
```
- **EN**: Implements logic around `populateBlockPackMatmulPatterns`, `BatchMatmulOp>>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateBlockPackMatmulPatterns`, `BatchMatmulOp>>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Linalg/Utils/Utils.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/SmallVector.h`, `mlir/Dialect/Linalg/Passes.h.inc`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
