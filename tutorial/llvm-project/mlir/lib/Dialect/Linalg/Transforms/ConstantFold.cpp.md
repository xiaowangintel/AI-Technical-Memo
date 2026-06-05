# ConstantFold.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/ConstantFold.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements constant folding on Linalg operations.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ConstantFold.cpp - Implementation of constant folding on Linalg ops ===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-20
```cpp
//
// This file implements constant folding on Linalg operations.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Matchers.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Matchers.h`, `mlir/IR/PatternMatch.h`。

### Lines 21-27
```cpp
using namespace mlir;
using namespace mlir::linalg;

namespace {
/// Base class for constant folding linalg structured ops with N inputs, 1
/// output, and permutation indexing maps.
///
```
- **EN**: Introduces declarations for `mlir`, `mlir::linalg`, `for`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::linalg`, `for` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 28-34
```cpp
/// `ConcreteType` should provide methods with signatures
///
/// ```c++
///   bool matchIndexingMaps(LinalgOp linalgOp) const;
///   RegionComputationFn getRegionComputeFn(LinalgOp) const;
/// ```
///
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 35-41
```cpp
/// The latter inspects the region and returns the computation inside as a
/// functor. The functor will be invoked with constant elements for all inputs
/// and should return the corresponding computed constant element for output.
template <typename ConcreteType>
class FoldConstantBase : public OpInterfaceRewritePattern<LinalgOp> {
public:
  struct APIntOrFloat {
```
- **EN**: Introduces declarations for `FoldConstantBase`, `APIntOrFloat`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldConstantBase`, `APIntOrFloat` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 42-51
```cpp
    std::optional<APInt> apInt;
    std::optional<APFloat> apFloat;
  };
  struct APIntOrFloatArray {
    SmallVector<APInt> apInts;
    SmallVector<APFloat> apFloats;
  };
  using RegionComputationFn =
      std::function<APIntOrFloat(const APIntOrFloatArray &)>;

```
- **EN**: Introduces declarations for `APIntOrFloatArray`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `APIntOrFloatArray` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 52-62
```cpp
  FoldConstantBase(MLIRContext *context, const ControlFusionFn &controlFn,
                   PatternBenefit benefit = 1)
      : OpInterfaceRewritePattern<LinalgOp>(context, benefit),
        controlFn(controlFn) {}

  LogicalResult matchAndRewrite(LinalgOp linalgOp,
                                PatternRewriter &rewriter) const override {
    // Mixed and buffer sematics aren't supported.
    if (!linalgOp.hasPureTensorSemantics())
      return failure();

```
- **EN**: Implements logic around `FoldConstantBase`, `OpInterfaceRewritePattern`, `controlFn`, `matchAndRewrite`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `FoldConstantBase`, `OpInterfaceRewritePattern`, `controlFn`, `matchAndRewrite`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 63-72
```cpp
    // Only support ops generating one output for now.
    if (linalgOp.getNumDpsInits() != 1)
      return failure();

    auto outputType = dyn_cast<ShapedType>(linalgOp->getResultTypes().front());
    // Require the output types to be static given that we are generating
    // constants.
    if (!outputType || !outputType.hasStaticShape())
      return failure();

```
- **EN**: Implements logic around `getNumDpsInits`, `failure`, `getResultTypes`, `hasStaticShape`.
- **CN**: 围绕 `getNumDpsInits`, `failure`, `getResultTypes`, `hasStaticShape` 实现具体逻辑。

### Lines 73-85
```cpp
    if (!llvm::all_of(linalgOp.getDpsInputs(), [](Value input) {
          return isa<ShapedType>(input.getType());
        }))
      return failure();

    // Make sure all element types are the same.
    auto getOperandElementType = [](Value value) {
      return cast<ShapedType>(value.getType()).getElementType();
    };
    if (!llvm::all_equal(
            llvm::map_range(linalgOp->getOperands(), getOperandElementType)))
      return failure();

```
- **EN**: Implements logic around `all_of`, `getType`, `failure`, `all_equal`, and 1 more symbols.
- **CN**: 围绕 `all_of`, `getType`, `failure`, `all_equal`, and 1 more symbols 实现具体逻辑。

### Lines 86-99
```cpp
    // We can only handle the case where we have int/float elements.
    auto elementType = outputType.getElementType();
    if (!elementType.isIntOrFloat())
      return failure();

    // Require all indexing maps to be permutations for now. This is common and
    // it simplifies input/output access greatly: we can do the data shuffling
    // entirely in the compiler, without needing to turn all indices into
    // Values, and then do affine apply on them, and then match back the
    // constant again.
    if (!llvm::all_of(linalgOp.getIndexingMapsArray(),
                      [](AffineMap map) { return map.isPermutation(); }))
      return failure();

```
- **EN**: Implements logic around `getElementType`, `isIntOrFloat`, `failure`, `all_of`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getElementType`, `isIntOrFloat`, `failure`, `all_of`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 100-108
```cpp
    for (OpOperand &operand : linalgOp.getDpsInitsMutable()) {
      if (linalgOp.payloadUsesValueFromOperand(&operand))
        return failure();
    }

    // Further check the indexing maps are okay for the ConcreteType.
    if (!static_cast<const ConcreteType *>(this)->matchIndexingMaps(linalgOp))
      return failure();

```
- **EN**: Implements logic around `getDpsInitsMutable`, `payloadUsesValueFromOperand`, `failure`, `matchIndexingMaps`.
- **CN**: 围绕 `getDpsInitsMutable`, `payloadUsesValueFromOperand`, `failure`, `matchIndexingMaps` 实现具体逻辑。

### Lines 109-115
```cpp
    // Defer to the concrete type to check the region and discover the
    // computation inside.
    RegionComputationFn computeFn =
        static_cast<const ConcreteType *>(this)->getRegionComputeFn(linalgOp);
    if (!computeFn)
      return failure();

```
- **EN**: Implements logic around `getRegionComputeFn`, `failure`.
- **CN**: 围绕 `getRegionComputeFn`, `failure` 实现具体逻辑。

### Lines 116-124
```cpp
    // All inputs should be constants.
    int numInputs = linalgOp.getNumDpsInputs();
    SmallVector<DenseTypedElementsAttr> inputValues(numInputs);
    for (const auto &en : llvm::enumerate(linalgOp.getDpsInputOperands())) {
      if (!matchPattern(en.value()->get(),
                        m_Constant(&inputValues[en.index()])))
        return failure();
    }

```
- **EN**: Implements logic around `getNumDpsInputs`, `inputValues`, `enumerate`, `matchPattern`, and 2 more symbols.
- **CN**: 围绕 `getNumDpsInputs`, `inputValues`, `enumerate`, `matchPattern`, and 2 more symbols 实现具体逻辑。

### Lines 125-131
```cpp
    // Identified this as a potential candidate for folding. Now check the
    // policy to see whether we are allowed to proceed.
    for (OpOperand *operand : linalgOp.getDpsInputOperands()) {
      if (!controlFn(operand))
        return failure();
    }

```
- **EN**: Implements logic around `getDpsInputOperands`, `controlFn`, `failure`.
- **CN**: 围绕 `getDpsInputOperands`, `controlFn`, `failure` 实现具体逻辑。

### Lines 132-144
```cpp
    SmallVector<int64_t, 4> loopBounds = linalgOp.getStaticLoopRanges();
    int64_t numElements = outputType.getNumElements();

    // Use APInt/APFloat instead of Attribute here for constructing the output.
    // This helps to avoid blowing up compiler memory usage: Attributes would
    // unify the following cases but they have lifetime as the MLIRContext.
    SmallVector<APInt> intOutputValues;
    SmallVector<APFloat> fpOutputValues;
    if (isa<FloatType>(elementType))
      fpOutputValues.resize(numElements, APFloat(0.f));
    else
      intOutputValues.resize(numElements);

```
- **EN**: Implements logic around `getStaticLoopRanges`, `getNumElements`, `resize`.
- **CN**: 围绕 `getStaticLoopRanges`, `getNumElements`, `resize` 实现具体逻辑。

### Lines 145-154
```cpp
    // Return the constant dim positions from the given permutation map.
    auto getDimPositions = [](AffineMap map) {
      SmallVector<unsigned> dims;
      dims.reserve(map.getNumResults());
      for (AffineExpr result : map.getResults()) {
        dims.push_back(cast<AffineDimExpr>(result).getPosition());
      }
      return dims;
    };

```
- **EN**: Implements logic around `reserve`, `getResults`, `push_back`.
- **CN**: 围绕 `reserve`, `getResults`, `push_back` 实现具体逻辑。

### Lines 155-168
```cpp
    SmallVector<SmallVector<unsigned>> inputDims;
    for (int i = 0; i < numInputs; ++i)
      inputDims.push_back(getDimPositions(linalgOp.getIndexingMapsArray()[i]));
    auto outputDims = getDimPositions(linalgOp.getIndexingMapsArray().back());
    auto outputShape = outputType.getShape();

    // Allocate small vectors for index delinearization. Initial values do not
    // matter here as they will be overwritten later.
    SmallVector<uint64_t> indices(loopBounds.size(), 0);
    SmallVector<uint64_t> dstIndices(loopBounds.size(), 0);
    SmallVector<SmallVector<uint64_t>> srcIndices(
        numInputs, SmallVector<uint64_t>(loopBounds.size(), 0));
    SmallVector<uint64_t> srcLinearIndices(numInputs, 0);
    uint64_t dstLinearIndex = 0;
```
- **EN**: Implements logic around `push_back`, `getDimPositions`, `getShape`, `indices`, and 4 more symbols.
- **CN**: 围绕 `push_back`, `getDimPositions`, `getShape`, `indices`, and 4 more symbols 实现具体逻辑。

### Lines 169-178
```cpp

    // Allocate spaces for compute function inputs. Initial values do not matter
    // here as they will be overwritten later.
    APIntOrFloatArray computeFnInputs;

    auto inputShapes =
        llvm::map_to_vector<4>(linalgOp.getDpsInputs(), [](Value value) {
          return cast<ShapedType>(value.getType()).getShape();
        });

```
- **EN**: Implements logic around `map_to_vector`, `getType`.
- **CN**: 围绕 `map_to_vector`, `getType` 实现具体逻辑。

### Lines 179-188
```cpp
    // Given a `linearIndex`, remap it to a linear index to access linalg op
    // inputs/ouputs. This mutates `indices`, `srcIndices`, `dstIndices`,
    // `srcLinearIndices`, `dstLinearIndex` in place.
    auto computeRemappedLinearIndex = [&](int linearIndex) {
      int totalCount = linearIndex;
      for (int dim = loopBounds.size() - 1; dim >= 0; --dim) {
        indices[dim] = totalCount % loopBounds[dim];
        totalCount /= loopBounds[dim];
      }

```
- **EN**: Implements logic around `size`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `size` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 189-198
```cpp
      for (int dim = loopBounds.size() - 1; dim >= 0; --dim) {
        for (int i = 0; i < numInputs; ++i)
          srcIndices[i][dim] = indices[inputDims[i][dim]];
        dstIndices[dim] = indices[outputDims[dim]];
      }

      dstLinearIndex = dstIndices.front();
      for (int i = 0; i < numInputs; ++i)
        srcLinearIndices[i] = srcIndices[i].front();

```
- **EN**: Implements logic around `size`, `front`.
- **CN**: 围绕 `size`, `front` 实现具体逻辑。

### Lines 199-206
```cpp
      for (int dim = 1; dim < outputType.getRank(); ++dim) {
        dstLinearIndex = dstLinearIndex * outputShape[dim] + dstIndices[dim];
        for (int i = 0; i < numInputs; ++i)
          srcLinearIndices[i] =
              srcLinearIndices[i] * inputShapes[i][dim] + srcIndices[i][dim];
      }
    };

```
- **EN**: Implements logic around `getRank`.
- **CN**: 围绕 `getRank` 实现具体逻辑。

### Lines 207-214
```cpp
    bool isFloat = isa<FloatType>(elementType);
    if (isFloat) {
      SmallVector<DenseElementsAttr::iterator_range<APFloat>> inFpRanges;
      for (int i = 0; i < numInputs; ++i)
        inFpRanges.push_back(inputValues[i].getValues<APFloat>());

      computeFnInputs.apFloats.resize(numInputs, APFloat(0.f));

```
- **EN**: Implements logic around `push_back`, `resize`.
- **CN**: 围绕 `push_back`, `resize` 实现具体逻辑。

### Lines 215-224
```cpp
      // Transpose the input constant. Because we don't know its rank in
      // advance, we need to loop over the range [0, element count) and
      // delinearize the index.
      for (int linearIndex = 0; linearIndex < numElements; ++linearIndex) {
        computeRemappedLinearIndex(linearIndex);

        // Collect constant elements for all inputs at this loop iteration.
        for (int i = 0; i < numInputs; ++i)
          computeFnInputs.apFloats[i] = inFpRanges[i][srcLinearIndices[i]];

```
- **EN**: Implements logic around `computeRemappedLinearIndex`.
- **CN**: 围绕 `computeRemappedLinearIndex` 实现具体逻辑。

### Lines 225-233
```cpp
        // Invoke the computation to get the corresponding constant output
        // element.
        fpOutputValues[dstLinearIndex] = *computeFn(computeFnInputs).apFloat;
      }
    } else {
      SmallVector<DenseElementsAttr::iterator_range<APInt>> inIntRanges;
      for (int i = 0; i < numInputs; ++i)
        inIntRanges.push_back(inputValues[i].getValues<APInt>());

```
- **EN**: Implements logic around `computeFn`, `push_back`.
- **CN**: 围绕 `computeFn`, `push_back` 实现具体逻辑。

### Lines 234-241
```cpp
      computeFnInputs.apInts.resize(numInputs);

      // Transpose the input constant. Because we don't know its rank in
      // advance, we need to loop over the range [0, element count) and
      // delinearize the index.
      for (int linearIndex = 0; linearIndex < numElements; ++linearIndex) {
        computeRemappedLinearIndex(linearIndex);

```
- **EN**: Implements logic around `resize`, `computeRemappedLinearIndex`.
- **CN**: 围绕 `resize`, `computeRemappedLinearIndex` 实现具体逻辑。

### Lines 242-251
```cpp
        // Collect constant elements for all inputs at this loop iteration.
        for (int i = 0; i < numInputs; ++i)
          computeFnInputs.apInts[i] = inIntRanges[i][srcLinearIndices[i]];

        // Invoke the computation to get the corresponding constant output
        // element.
        intOutputValues[dstLinearIndex] = *computeFn(computeFnInputs).apInt;
      }
    }

```
- **EN**: Implements logic around `computeFn`.
- **CN**: 围绕 `computeFn` 实现具体逻辑。

### Lines 252-259
```cpp
    DenseElementsAttr outputAttr =
        isFloat ? DenseElementsAttr::get(outputType, fpOutputValues)
                : DenseElementsAttr::get(outputType, intOutputValues);

    rewriter.replaceOpWithNewOp<arith::ConstantOp>(linalgOp, outputAttr);
    return success();
  }

```
- **EN**: Implements logic around `get`, `ConstantOp>`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `get`, `ConstantOp>`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 260-266
```cpp
private:
  ControlFusionFn controlFn;
};

// Folds linalg.transpose (and linalg.generic ops that are actually transposes)
// on constant values.
struct FoldConstantTranspose : public FoldConstantBase<FoldConstantTranspose> {
```
- **EN**: Introduces declarations for `FoldConstantTranspose`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `FoldConstantTranspose` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 267-274
```cpp

  using FoldConstantBase::FoldConstantBase;

  bool matchIndexingMaps(LinalgOp linalgOp) const {
    // We should have one input and one output.
    return linalgOp.getIndexingMapsArray().size() == 2;
  }

```
- **EN**: Implements logic around `matchIndexingMaps`, `getIndexingMapsArray`.
- **CN**: 围绕 `matchIndexingMaps`, `getIndexingMapsArray` 实现具体逻辑。

### Lines 275-283
```cpp
  RegionComputationFn getRegionComputeFn(LinalgOp linalgOp) const {
    // Make sure the region only contains a yield op.
    Block &body = linalgOp->getRegion(0).front();
    if (!llvm::hasSingleElement(body))
      return nullptr;
    auto yieldOp = dyn_cast<linalg::YieldOp>(body.getTerminator());
    if (!yieldOp)
      return nullptr;

```
- **EN**: Implements logic around `getRegionComputeFn`, `getRegion`, `hasSingleElement`, `YieldOp>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getRegionComputeFn`, `getRegion`, `hasSingleElement`, `YieldOp>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 284-292
```cpp
    // The yield op should return the block argument corresponds to the input.
    for (Value yieldVal : yieldOp.getValues()) {
      auto yieldArg = dyn_cast<BlockArgument>(yieldVal);
      if (!yieldArg || yieldArg.getOwner() != &body)
        return nullptr;
      if (yieldArg.getArgNumber() != 0)
        return nullptr;
    }

```
- **EN**: Implements logic around `getValues`, `getOwner`, `getArgNumber`.
- **CN**: 围绕 `getValues`, `getOwner`, `getArgNumber` 实现具体逻辑。

### Lines 293-300
```cpp
    // No computation; just return the orginal value.
    return [](const APIntOrFloatArray &inputs) {
      if (inputs.apFloats.empty())
        return APIntOrFloat{inputs.apInts.front(), std::nullopt};
      return APIntOrFloat{std::nullopt, inputs.apFloats.front()};
    };
  }

```
- **EN**: Implements logic around `empty`, `front`.
- **CN**: 围绕 `empty`, `front` 实现具体逻辑。

### Lines 301-309
```cpp
  ControlFusionFn controlFn;
};
} // namespace

void mlir::linalg::populateConstantFoldLinalgOperations(
    RewritePatternSet &patterns, const ControlFusionFn &controlFn) {
  MLIRContext *context = patterns.getContext();
  patterns.insert<FoldConstantTranspose>(context, controlFn);
}
```
- **EN**: Implements logic around `populateConstantFoldLinalgOperations`, `getContext`, `insert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateConstantFoldLinalgOperations`, `getContext`, `insert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/Matchers.h`, `mlir/IR/PatternMatch.h`, `mlir/Support/LLVM.h`, `llvm/ADT/SmallVectorExtras.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
