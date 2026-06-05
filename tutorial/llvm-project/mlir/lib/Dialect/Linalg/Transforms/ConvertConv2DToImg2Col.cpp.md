# ConvertConv2DToImg2Col.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/ConvertConv2DToImg2Col.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Linalg dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ConvertConv2DToImg2Col`.
  - **CN**: 实现 Linalg 方言中围绕 `ConvertConv2DToImg2Col` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```cpp
//===- ConvertConv2DToImg2Col.cpp - im2col implementation -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/Dialect/Tensor/IR/Tensor.h"
#include "mlir/Dialect/Utils/IndexingUtils.h"
#include "mlir/Dialect/Utils/ReshapeOpsUtils.h"
#include "mlir/Dialect/Utils/StructuredOpsUtils.h"
#include "mlir/IR/AffineExpr.h"
#include "mlir/IR/AffineMap.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/BuiltinTypes.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include <cassert>
#include <utility>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Linalg/IR/Linalg.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Linalg/IR/Linalg.h`。

### Lines 27-41
```cpp
namespace mlir {
namespace linalg {
static bool hasAllOneValues(DenseIntElementsAttr attr) {
  return llvm::all_of(
      attr, [](const APInt &element) { return element.getSExtValue() == 1; });
}

static Value createAdd(Location loc, Value x, Value y, OpBuilder &builder) {
  if (isa<IntegerType>(x.getType()))
    return arith::AddIOp::create(builder, loc, x, y);
  if (isa<ComplexType>(x.getType()))
    return complex::AddOp::create(builder, loc, x, y);
  return arith::AddFOp::create(builder, loc, x, y);
}

```
- **EN**: Introduces declarations for `mlir`, `linalg`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `linalg` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 42-55
```cpp
static Value createMul(Location loc, Value x, Value y, Type accType,
                       OpBuilder &builder) {
  // Linalg named ops specify signed extend for named ops.
  Value xConvert =
      convertScalarToDtype(builder, loc, x, accType, /*isUnsignedCast=*/false);
  Value yConvert =
      convertScalarToDtype(builder, loc, y, accType, /*isUnsignedCast=*/false);
  if (isa<ComplexType>(accType))
    return complex::MulOp::create(builder, loc, xConvert, yConvert);
  if (isa<IntegerType>(accType))
    return arith::MulIOp::create(builder, loc, xConvert, yConvert);
  return arith::MulFOp::create(builder, loc, xConvert, yConvert);
}

```
- **EN**: Implements logic around `createMul`, `convertScalarToDtype`, `create`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createMul`, `convertScalarToDtype`, `create` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 56-71
```cpp
// Generate the affine expression to compute the convolved index
// for the input as `oIndex * stride + fIndex`,
// where oIndex: output iterator; fIndex: filter iterator.
static AffineExpr getConvolvedExpr(OpBuilder &b, int64_t stride,
                                   bool useSymbols = true) {
  AffineExpr oExpr, fExpr;
  if (useSymbols)
    bindSymbols(b.getContext(), oExpr, fExpr);
  else
    bindDims(b.getContext(), oExpr, fExpr);
  return AffineExpr(stride * oExpr + fExpr);
}

// Stores the affine expressions to map the iteration space of the im2col matrix
// to the corresponding indices of the output and filter matrices
struct Im2ColToOperandsExprs {
```
- **EN**: Introduces declarations for `Im2ColToOperandsExprs`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `Im2ColToOperandsExprs` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 72-87
```cpp
  AffineExpr fhIndex;
  AffineExpr fwIndex;
  AffineExpr icIndex;
  AffineExpr ohIndex;
  AffineExpr owIndex;
};

// Stores the affine expressions to map the iteration space of the im2col matrix
// to the input matrix indices
struct Im2ColToInputDimsExprs {
  AffineExpr bIndex;
  AffineExpr hIndex;
  AffineExpr wIndex;
  AffineExpr cIndex;
};

```
- **EN**: Introduces declarations for `Im2ColToInputDimsExprs`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `Im2ColToInputDimsExprs` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 88-115
```cpp
/// Construct the affine expressions that map the indices of the im2col matrix
/// to the corresponding input tensor indices for a 2D convolution with the the
/// provided strides.
///
/// @param exprs      Affine expressions for output and filter indices.
/// @param strides    [height, width] stride values for the convolution.
/// @param rewriter   Pattern rewriter.
/// @return           Affine expressions mapping im2col matrix indices to input
/// offsets.
static Im2ColToInputDimsExprs
getIm2ColInputExpressions(Im2ColToOperandsExprs exprs,
                          ArrayRef<int64_t> strides, RewriterBase &rewriter) {
  // maps the iteration space of the im2col matrix to (output_y, filter_y)
  auto hIndicesMap = AffineMap::inferFromExprList(
      {ArrayRef{exprs.ohIndex, exprs.fhIndex}}, rewriter.getContext())[0];
  // maps the iteration space of the im2col matrix to (output_x, filter_x)
  auto wIndicesMap = AffineMap::inferFromExprList(
      {ArrayRef{exprs.owIndex, exprs.fwIndex}}, rewriter.getContext())[0];
  // Compute the input indexing map, to map the indices of the im2col matrix to
  // the original input offsets. Each element of the im2col matrix corresponds
  // to a pair of (out_element, filter_element). First, we build the expressions
  // to compute the input (ix, iy) indices from [out_x/y, filter_x/y] pairs;
  // then we compose them with the maps that map the im2col matrix elements to
  // the (out_element, filter_element) pairs.
  auto bIndexExpr = rewriter.getAffineDimExpr(0U);
  auto hIndexExpr = getConvolvedExpr(rewriter, strides[0],
                                     /*useSymbols*/ false);
  hIndexExpr = hIndexExpr.compose(hIndicesMap);
```
- **EN**: Implements logic around `getIm2ColInputExpressions`, `inferFromExprList`, `getContext`, `getAffineDimExpr`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIm2ColInputExpressions`, `inferFromExprList`, `getContext`, `getAffineDimExpr`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 116-132
```cpp
  auto wIndexExpr = getConvolvedExpr(rewriter, strides[1],
                                     /*useSymbols*/ false);
  wIndexExpr = wIndexExpr.compose(wIndicesMap);
  auto cIndexExpr = exprs.icIndex;
  return {bIndexExpr, hIndexExpr, wIndexExpr, cIndexExpr};
}

FailureOr<std::pair<Operation *, Operation *>>
rewriteInIm2Col(RewriterBase &rewriter, linalg::Conv2DNhwcHwcfOp convOp) {
  auto inputType = cast<ShapedType>(convOp.getInputs()[0].getType());
  auto filterType = cast<ShapedType>(convOp.getInputs()[1].getType());
  auto outputType = cast<ShapedType>(convOp.getOutputs()[0].getType());

  if (!convOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        convOp, "expected op to have pure tensor semantics");

```
- **EN**: Implements logic around `getConvolvedExpr`, `compose`, `rewriteInIm2Col`, `getInputs`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getConvolvedExpr`, `compose`, `rewriteInIm2Col`, `getInputs`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 133-150
```cpp
  if (!filterType.hasStaticShape())
    return rewriter.notifyMatchFailure(
        convOp, "expected a static shape for the filter");

  if (!inputType.hasStaticShape())
    return rewriter.notifyMatchFailure(convOp,
                                       "expected a static shape for the input");

  // TODO: Support dilation.
  if (!hasAllOneValues(convOp.getDilations()))
    return rewriter.notifyMatchFailure(convOp,
                                       "expected all ones for dilations");

  MLIRContext *context = rewriter.getContext();
  Value input = convOp.getInputs()[0];
  Value filter = convOp.getInputs()[1];
  Value output = convOp.getOutputs()[0];

```
- **EN**: Implements logic around `hasStaticShape`, `notifyMatchFailure`, `hasAllOneValues`, `getContext`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasStaticShape`, `notifyMatchFailure`, `hasAllOneValues`, `getContext`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 151-167
```cpp
  ArrayRef<int64_t> filterShape = filterType.getShape();
  ArrayRef<int64_t> outputShape = outputType.getShape();

  int64_t n = outputShape[0];
  int64_t oh = outputShape[1];
  int64_t ow = outputShape[2];
  int64_t oc = outputShape[3];
  int64_t fh = filterShape[0];
  int64_t fw = filterShape[1];
  int64_t ic = filterShape[2];

  Location loc = convOp.getLoc();

  assert(isa<RankedTensorType>(filterType) &&
         "expected filter type to be a ranked tensor");
  auto tensorFilterType = cast<RankedTensorType>(filterType);

```
- **EN**: Implements logic around `getShape`, `getLoc`, `assert`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getShape`, `getLoc`, `assert` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 168-181
```cpp
  // Reshape output and filter to the LHS and result of a (B)MNK matmul.
  SmallVector<ReassociationIndices> filterReassocIndices = {{0, 1, 2}, {3}};
  auto reshapedFilterType =
      RankedTensorType::get({fh * fw * ic, oc}, filterType.getElementType(),
                            tensorFilterType.getEncoding());
  Value reshapedFilter = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedFilterType, filter, filterReassocIndices);

  SmallVector<ReassociationIndices> outputReassocIndices = {{0}, {1, 2}, {3}};
  RankedTensorType reshapedOutputType =
      RankedTensorType::get({n, oh * ow, oc}, outputType.getElementType());
  Value reshapedOutput = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedOutputType, output, outputReassocIndices);

```
- **EN**: Implements logic around `get`, `getEncoding`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `getEncoding`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 182-205
```cpp
  SmallVector<int64_t> colTensorShape = {n, oh * ow, fh * fw * ic};
  Value colTensor = tensor::EmptyOp::create(rewriter, loc, colTensorShape,
                                            inputType.getElementType());

  // Convert the input to a (BMK) column tensor.
  auto nloops = colTensorShape.size();

  auto parallel = utils::IteratorType::parallel;
  auto reduction = utils::IteratorType::reduction;
  SmallVector<utils::IteratorType> img2colIterators(nloops, parallel);

  // Given an index of the im2col matrix, retrieve the corresponding indices of
  // the output and filter matrices
  auto mIndicesExprs =
      delinearize(rewriter.getAffineDimExpr(1U), ArrayRef<int64_t>{ow, 1});
  auto kIndicesExprs = delinearize(rewriter.getAffineDimExpr(2U),
                                   ArrayRef<int64_t>{fw * ic, ic, 1});
  Im2ColToOperandsExprs i2cToOperExprs;
  i2cToOperExprs.fhIndex = kIndicesExprs[0];
  i2cToOperExprs.fwIndex = kIndicesExprs[1];
  i2cToOperExprs.icIndex = kIndicesExprs[2];
  i2cToOperExprs.ohIndex = mIndicesExprs[0];
  i2cToOperExprs.owIndex = mIndicesExprs[1];

```
- **EN**: Implements logic around `create`, `getElementType`, `size`, `img2colIterators`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getElementType`, `size`, `img2colIterators`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 206-225
```cpp
  // im2col[n, oh*ow, fh*fw*ic] = input[n, sh*oh + fh, sw*ow + fw, ic]
  Im2ColToInputDimsExprs inExprs = getIm2ColInputExpressions(
      i2cToOperExprs, llvm::to_vector(convOp.getStrides().getValues<int64_t>()),
      rewriter);
  auto inMap =
      AffineMap::inferFromExprList({ArrayRef{inExprs.bIndex, inExprs.hIndex,
                                             inExprs.wIndex, inExprs.cIndex}},
                                   rewriter.getContext())[0];

  SmallVector<AffineMap> img2colIndexingMaps = {
      inMap, AffineMap::getMultiDimIdentityMap(nloops, context)};

  auto img2ColTensor = linalg::GenericOp::create(
      rewriter, loc, colTensor.getType(),
      /*inputs=*/input, /*outputs=*/colTensor, img2colIndexingMaps,
      img2colIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        linalg::YieldOp::create(nestedBuilder, nestedLoc, args[0]);
      });

```
- **EN**: Implements logic around `getIm2ColInputExpressions`, `to_vector`, `inferFromExprList`, `getContext`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIm2ColInputExpressions`, `to_vector`, `inferFromExprList`, `getContext`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 226-250
```cpp
  // Because the filter does not share the same batch dimension,
  // the batch dimension is only used in indexing the input and output. Thus
  // we cannot use existing linalg named ops like linalg.batch_matmul.
  // i.e. (B x) M x K * K x N = (B x) M x N
  AffineExpr bDim, mDim, nDim, kDim;
  bindDims(context, bDim, mDim, nDim, kDim);
  auto lhsMap = AffineMap::get(4, 0, {bDim, mDim, kDim}, context);
  auto rhsMap = AffineMap::get(4, 0, {kDim, nDim}, context);
  auto resultMap = AffineMap::get(4, 0, {bDim, mDim, nDim}, context);
  SmallVector<utils::IteratorType> genericIterators = {parallel, parallel,
                                                       parallel, reduction};

  auto genericOp = linalg::GenericOp::create(
      rewriter, loc, reshapedOutputType,
      /*inputs=*/ValueRange{img2ColTensor.getResult(0), reshapedFilter},
      /*outputs=*/ValueRange{reshapedOutput},
      ArrayRef<AffineMap>{lhsMap, rhsMap, resultMap}, genericIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        Value mul =
            createMul(loc, args[0], args[1], args[2].getType(), nestedBuilder);
        Value add = createAdd(loc, mul, args[2], nestedBuilder);
        linalg::YieldOp::create(nestedBuilder, nestedLoc, add);
      });
  Value result = genericOp.getResults().front();

```
- **EN**: Implements logic around `bindDims`, `get`, `create`, `getResult`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bindDims`, `get`, `create`, `getResult`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 251-266
```cpp
  auto reshapedResult = tensor::ExpandShapeOp::create(
      rewriter, loc, outputType, result, outputReassocIndices);

  rewriter.replaceOp(convOp, ArrayRef<Value>{reshapedResult});

  return std::make_pair(img2ColTensor.getOperation(),
                        reshapedResult.getOperation());
}

FailureOr<std::pair<Operation *, Operation *>>
rewriteInIm2Col(RewriterBase &rewriter,
                linalg::DepthwiseConv2DNhwcHwcOp convOp) {
  auto inputType = cast<RankedTensorType>(convOp.getInputs()[0].getType());
  auto filterType = cast<RankedTensorType>(convOp.getInputs()[1].getType());
  auto outputType = cast<RankedTensorType>(convOp.getOutputs()[0].getType());

```
- **EN**: Implements logic around `create`, `replaceOp`, `make_pair`, `getOperation`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `replaceOp`, `make_pair`, `getOperation`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 267-283
```cpp
  if (!convOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        convOp, "expected op to have pure tensor semantics");

  if (!filterType.hasStaticShape())
    return rewriter.notifyMatchFailure(
        convOp, "expected a static shape for the filter");

  if (!inputType.hasStaticShape())
    return rewriter.notifyMatchFailure(convOp,
                                       "expected a static shape for the input");

  // TODO: Support dilation.
  if (!hasAllOneValues(convOp.getDilations()))
    return rewriter.notifyMatchFailure(convOp,
                                       "expected all ones for dilations");

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `notifyMatchFailure`, `hasStaticShape`, `hasAllOneValues`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureTensorSemantics`, `notifyMatchFailure`, `hasStaticShape`, `hasAllOneValues` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 284-298
```cpp
  Location loc = convOp.getLoc();

  auto transposeOperand = [&](Value operand, ArrayRef<int64_t> indices) {
    auto operandTensorType = cast<RankedTensorType>(operand.getType());
    auto nloops = indices.size();
    ArrayRef<int64_t> inputShape = operandTensorType.getShape();

    SmallVector<AffineExpr> exprs =
        llvm::map_to_vector<4>(indices, [&](int64_t index) -> AffineExpr {
          return rewriter.getAffineDimExpr(index);
        });

    SmallVector<int64_t> targetShape = llvm::map_to_vector<4>(
        indices, [&](int64_t index) -> int64_t { return inputShape[index]; });

```
- **EN**: Implements logic around `getLoc`, `getType`, `size`, `getShape`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getLoc`, `getType`, `size`, `getShape`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 299-317
```cpp
    Value outputTensor = tensor::EmptyOp::create(
        rewriter, loc, targetShape, operandTensorType.getElementType());

    SmallVector<utils::IteratorType> loopAttributeTypes(
        nloops, utils::IteratorType::parallel);

    SmallVector<AffineMap> indexingMaps = {
        inversePermutation(
            AffineMap::get(nloops, 0, exprs, rewriter.getContext())),
        AffineMap::getMultiDimIdentityMap(nloops, rewriter.getContext())};

    auto transposedOp = linalg::GenericOp::create(
        rewriter, loc, outputTensor.getType(),
        /*inputs=*/operand, /*outputs=*/outputTensor, indexingMaps,
        loopAttributeTypes,
        [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
          linalg::YieldOp::create(nestedBuilder, nestedLoc, args[0]);
        });

```
- **EN**: Implements logic around `create`, `getElementType`, `loopAttributeTypes`, `inversePermutation`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getElementType`, `loopAttributeTypes`, `inversePermutation`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 318-331
```cpp
    return transposedOp.getResult(0);
  };

  Value input = convOp.getInputs()[0];
  Value filter = convOp.getInputs()[1];
  Value output = convOp.getOutputs()[0];

  // Transpose input, filter so channels are outermost
  Value inputT = transposeOperand(input, {0, 3, 1, 2});
  Value filterT = transposeOperand(filter, {2, 0, 1});
  ArrayRef<int64_t> filterTShape =
      cast<RankedTensorType>(filterT.getType()).getShape();
  ArrayRef<int64_t> outputShape = outputType.getShape();

```
- **EN**: Implements logic around `getResult`, `getInputs`, `getOutputs`, `transposeOperand`, and 2 more symbols.
- **CN**: 围绕 `getResult`, `getInputs`, `getOutputs`, `transposeOperand`, and 2 more symbols 实现具体逻辑。

### Lines 332-349
```cpp
  int n = outputShape[0];
  int oh = outputShape[1];
  int ow = outputShape[2];
  int c = outputShape[3];
  int fh = filterTShape[1];
  int fw = filterTShape[2];

  SmallVector<int64_t> colTensorShape = {n, c, oh, ow, fh, fw};
  Value transposedOutputTensor = transposeOperand(output, {0, 3, 1, 2});

  AffineExpr nDim, cDim, ohDim, owDim, khDim, kwDim;
  bindDims(rewriter.getContext(), nDim, cDim, ohDim, owDim, khDim, kwDim);

  AffineExpr shSym = rewriter.getAffineConstantExpr(
      convOp.getStrides().getValues<int64_t>()[0]);
  AffineExpr swSym = rewriter.getAffineConstantExpr(
      convOp.getStrides().getValues<int64_t>()[1]);

```
- **EN**: Implements logic around `transposeOperand`, `bindDims`, `getAffineConstantExpr`, `getStrides`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `transposeOperand`, `bindDims`, `getAffineConstantExpr`, `getStrides` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 350-364
```cpp
  SmallVector<AffineExpr> inputExprs = {nDim, cDim, ohDim * shSym + khDim,
                                        owDim * swSym + kwDim};

  auto nloops = colTensorShape.size();

  SmallVector<utils::IteratorType> loopAttributeTypes(
      nloops, utils::IteratorType::parallel);

  SmallVector<AffineMap> indexingMaps = {
      AffineMap::get(nloops, 0, inputExprs, rewriter.getContext()),
      AffineMap::getMultiDimIdentityMap(nloops, rewriter.getContext())};

  Value colTensor = tensor::EmptyOp::create(rewriter, loc, colTensorShape,
                                            inputType.getElementType());

```
- **EN**: Implements logic around `size`, `loopAttributeTypes`, `get`, `getMultiDimIdentityMap`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `size`, `loopAttributeTypes`, `get`, `getMultiDimIdentityMap`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 365-378
```cpp
  auto img2ColTensor = linalg::GenericOp::create(
      rewriter, loc, colTensor.getType(),
      /*inputs=*/inputT, /*outputs=*/colTensor, indexingMaps,
      loopAttributeTypes,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        linalg::YieldOp::create(nestedBuilder, nestedLoc, args[0]);
      });

  SmallVector<ReassociationIndices> img2ColTensorReassocIndices = {
      {0, 1}, {2, 3}, {4, 5}};
  SmallVector<ReassociationIndices> filterReassociationIndice = {{0}, {1, 2}};
  SmallVector<ReassociationIndices> outputReassociationIndice = {{0, 1},
                                                                 {2, 3}};

```
- **EN**: Implements logic around `create`, `getType`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getType` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 379-395
```cpp
  auto reshapedImg2ColTensorType = RankedTensorType::get(
      {n * c, oh * ow, fh * fw}, inputType.getElementType());
  auto reshapedFilterTensorType =
      RankedTensorType::get({c, fh * fw}, filterType.getElementType());
  auto reshapedOutputTensorType =
      RankedTensorType::get({n * c, oh * ow}, outputType.getElementType());

  Value reshapedImg2ColTensor = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedImg2ColTensorType, img2ColTensor.getResult(0),
      img2ColTensorReassocIndices);
  Value reshapedFilterTensor =
      tensor::CollapseShapeOp::create(rewriter, loc, reshapedFilterTensorType,
                                      filterT, filterReassociationIndice);
  Value reshapedoutputTensor = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedOutputTensorType, transposedOutputTensor,
      outputReassociationIndice);

```
- **EN**: Implements logic around `get`, `getElementType`, `create`, `getResult`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `getElementType`, `create`, `getResult` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 396-410
```cpp
  auto batchMatVecResult = linalg::BatchMatvecOp::create(
      rewriter, loc, TypeRange{reshapedoutputTensor.getType()},
      ValueRange{reshapedImg2ColTensor, reshapedFilterTensor},
      ValueRange{reshapedoutputTensor});

  SmallVector<ReassociationIndices> batchMatVecReassociationIndice = {{0, 1},
                                                                      {2, 3}};

  auto batchMatVecResultReshaped = tensor::ExpandShapeOp::create(
      rewriter, loc, transposedOutputTensor.getType(),
      batchMatVecResult.getResult(0), batchMatVecReassociationIndice);

  Value transposedResult =
      transposeOperand(batchMatVecResultReshaped, {0, 2, 3, 1});

```
- **EN**: Implements logic around `create`, `getType`, `getResult`, `transposeOperand`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getType`, `getResult`, `transposeOperand` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 411-425
```cpp
  rewriter.replaceOp(convOp, ArrayRef<Value>{transposedResult});
  return std::make_pair(img2ColTensor.getOperation(),
                        transposedResult.getDefiningOp());
}

FailureOr<std::pair<Operation *, Operation *>>
rewriteInIm2Col(RewriterBase &rewriter, linalg::Conv2DNchwFchwOp convOp) {
  auto inputType = cast<ShapedType>(convOp.getInputs()[0].getType());
  auto filterType = cast<ShapedType>(convOp.getInputs()[1].getType());
  auto outputType = cast<ShapedType>(convOp.getOutputs()[0].getType());

  if (!convOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        convOp, "expected op to have pure tensor semantics");

```
- **EN**: Implements logic around `replaceOp`, `make_pair`, `getDefiningOp`, `rewriteInIm2Col`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceOp`, `make_pair`, `getDefiningOp`, `rewriteInIm2Col`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 426-442
```cpp
  if (!filterType.hasStaticShape())
    return rewriter.notifyMatchFailure(
        convOp, "expected a static shape for the filter");

  if (!inputType.hasStaticShape())
    return rewriter.notifyMatchFailure(convOp,
                                       "expected a static shape for the input");

  // TODO: Support dilation.
  if (!hasAllOneValues(convOp.getDilations()))
    return rewriter.notifyMatchFailure(convOp,
                                       "expected all ones for dilations");

  Value input = convOp.getInputs()[0];
  Value filter = convOp.getInputs()[1];
  Value output = convOp.getOutputs()[0];

```
- **EN**: Implements logic around `hasStaticShape`, `notifyMatchFailure`, `hasAllOneValues`, `getInputs`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasStaticShape`, `notifyMatchFailure`, `hasAllOneValues`, `getInputs`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 443-456
```cpp
  auto filterShape = filterType.getShape();
  auto outputShape = outputType.getShape();

  int64_t n = outputShape[0];
  int64_t oc = outputShape[1];
  int64_t oh = outputShape[2];
  int64_t ow = outputShape[3];
  int64_t ic = filterShape[1];
  int64_t fh = filterShape[2];
  int64_t fw = filterShape[3];

  auto loc = convOp.getLoc();
  MLIRContext *context = rewriter.getContext();

```
- **EN**: Implements logic around `getShape`, `getLoc`, `getContext`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getShape`, `getLoc`, `getContext` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 457-473
```cpp
  assert(isa<RankedTensorType>(filterType) &&
         "expected filter type to be a ranked tensor");
  auto tensorFilterType = cast<RankedTensorType>(filterType);

  SmallVector<ReassociationIndices> filterReassocIndices = {{0}, {1, 2, 3}};
  auto reshapedFilterType =
      RankedTensorType::get({oc, ic * fh * fw}, inputType.getElementType(),
                            tensorFilterType.getEncoding());
  Value reshapedFilter = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedFilterType, filter, filterReassocIndices);

  SmallVector<ReassociationIndices> outputReassocIndices = {{0}, {1}, {2, 3}};
  auto reshapedOutputType =
      RankedTensorType::get({n, oc, oh * ow}, outputType.getElementType());
  Value reshapedOutput = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedOutputType, output, outputReassocIndices);

```
- **EN**: Implements logic around `assert`, `get`, `getEncoding`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `get`, `getEncoding`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 474-501
```cpp
  // Convert the input to a (BKN) tensor.
  SmallVector<int64_t, 4> colTensorShape = {n, ic * fh * fw, oh * ow};
  Value colTensor = tensor::EmptyOp::create(rewriter, loc, colTensorShape,
                                            inputType.getElementType());

  auto nloops = colTensorShape.size();

  auto parallel = utils::IteratorType::parallel;
  auto reduction = utils::IteratorType::reduction;
  SmallVector<utils::IteratorType, 3> img2colIterators(nloops, parallel);

  // Recover the original iteration indices from the problem/input sizes:
  // given an index of the im2col matrix, retrieve the corresponding indices of
  // the output and filter matrices
  auto kIndicesExprs = delinearize(rewriter.getAffineDimExpr(1U),
                                   ArrayRef<int64_t>{fh * fw, fw, 1});
  auto mIndicesExprs =
      delinearize(rewriter.getAffineDimExpr(2U), ArrayRef<int64_t>{ow, 1});
  Im2ColToOperandsExprs i2cToOperExprs;
  i2cToOperExprs.icIndex = kIndicesExprs[0];
  i2cToOperExprs.fhIndex = kIndicesExprs[1];
  i2cToOperExprs.fwIndex = kIndicesExprs[2];
  i2cToOperExprs.ohIndex = mIndicesExprs[0];
  i2cToOperExprs.owIndex = mIndicesExprs[1];
  Im2ColToInputDimsExprs inExprs = getIm2ColInputExpressions(
      i2cToOperExprs, llvm::to_vector(convOp.getStrides().getValues<int64_t>()),
      rewriter);
  auto inMap =
```
- **EN**: Implements logic around `create`, `getElementType`, `size`, `img2colIterators`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getElementType`, `size`, `img2colIterators`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 502-516
```cpp
      AffineMap::inferFromExprList({ArrayRef{inExprs.bIndex, inExprs.cIndex,
                                             inExprs.hIndex, inExprs.wIndex}},
                                   rewriter.getContext())[0];
  // im2col[n, ic*fh*fw, oh*ow] = input[n, ic, sh*oh + fh, sw*ow + fw]
  SmallVector<AffineMap> img2colIndexingMaps = {
      inMap, AffineMap::getMultiDimIdentityMap(nloops, context)};

  auto img2ColTensor = linalg::GenericOp::create(
      rewriter, loc, colTensor.getType(),
      /*inputs=*/input, /*outputs=*/colTensor, img2colIndexingMaps,
      img2colIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        linalg::YieldOp::create(nestedBuilder, nestedLoc, args[0]);
      });

```
- **EN**: Implements logic around `inferFromExprList`, `getContext`, `getMultiDimIdentityMap`, `create`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `inferFromExprList`, `getContext`, `getMultiDimIdentityMap`, `create`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 517-540
```cpp
  // Because the filter does not share the same batch dimension,
  // the batch dimension is only used in indexing the input and output. Thus
  // we cannot use existing linalg named ops like linalg.batch_matmul.
  // i.e. M x K * (B x) K x N = (B x) M x N
  AffineExpr bDim, mDim, nDim, kDim;
  bindDims(context, bDim, mDim, nDim, kDim);
  auto lhsMap = AffineMap::get(4, 0, {mDim, kDim}, context);
  auto rhsMap = AffineMap::get(4, 0, {bDim, kDim, nDim}, context);
  auto resultMap = AffineMap::get(4, 0, {bDim, mDim, nDim}, context);
  SmallVector<utils::IteratorType> genericIterators = {parallel, parallel,
                                                       parallel, reduction};
  auto genericOp = linalg::GenericOp::create(
      rewriter, loc, reshapedOutputType,
      /*inputs=*/ValueRange{reshapedFilter, img2ColTensor.getResult(0)},
      /*outputs=*/ValueRange{reshapedOutput},
      ArrayRef<AffineMap>{lhsMap, rhsMap, resultMap}, genericIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        Value mul =
            createMul(loc, args[0], args[1], args[2].getType(), nestedBuilder);
        Value add = createAdd(loc, mul, args[2], nestedBuilder);
        linalg::YieldOp::create(nestedBuilder, nestedLoc, add);
      });
  Value result = genericOp.getResults().front();

```
- **EN**: Implements logic around `bindDims`, `get`, `create`, `getResult`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bindDims`, `get`, `create`, `getResult`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 541-555
```cpp
  auto reshapedResult = tensor::ExpandShapeOp::create(
      rewriter, loc, outputType, result, outputReassocIndices);

  rewriter.replaceOp(convOp, ArrayRef<Value>{reshapedResult});

  return std::make_pair(img2ColTensor.getOperation(),
                        reshapedResult.getOperation());
}

FailureOr<std::pair<Operation *, Operation *>>
rewriteInIm2Col(RewriterBase &rewriter, linalg::Conv2DNhwcFhwcOp convOp) {
  auto inputType = cast<ShapedType>(convOp.getInputs()[0].getType());
  auto filterType = cast<ShapedType>(convOp.getInputs()[1].getType());
  auto outputType = cast<ShapedType>(convOp.getOutputs()[0].getType());

```
- **EN**: Implements logic around `create`, `replaceOp`, `make_pair`, `getOperation`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `replaceOp`, `make_pair`, `getOperation`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 556-572
```cpp
  if (!convOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        convOp, "expected op to have pure tensor semantics");

  if (!filterType.hasStaticShape())
    return rewriter.notifyMatchFailure(
        convOp, "expected a static shape for the filter");

  if (!inputType.hasStaticShape())
    return rewriter.notifyMatchFailure(convOp,
                                       "expected a static shape for the input");

  // TODO: Support dilation.
  if (!hasAllOneValues(convOp.getDilations()))
    return rewriter.notifyMatchFailure(convOp,
                                       "expected all ones for dilations");

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `notifyMatchFailure`, `hasStaticShape`, `hasAllOneValues`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `hasPureTensorSemantics`, `notifyMatchFailure`, `hasStaticShape`, `hasAllOneValues` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 573-588
```cpp
  MLIRContext *context = rewriter.getContext();
  Value input = convOp.getInputs()[0];
  Value filter = convOp.getInputs()[1];
  Value output = convOp.getOutputs()[0];

  ArrayRef<int64_t> filterShape = filterType.getShape();
  ArrayRef<int64_t> outputShape = outputType.getShape();

  int64_t n = outputShape[0];
  int64_t oh = outputShape[1];
  int64_t ow = outputShape[2];
  int64_t oc = outputShape[3];
  int64_t fh = filterShape[1];
  int64_t fw = filterShape[2];
  int64_t ic = filterShape[3];

```
- **EN**: Implements logic around `getContext`, `getInputs`, `getOutputs`, `getShape`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getContext`, `getInputs`, `getOutputs`, `getShape` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 589-603
```cpp
  Location loc = convOp.getLoc();

  assert(isa<RankedTensorType>(filterType) &&
         "expected filter type to be a ranked tensor");
  auto tensorFilterType = cast<RankedTensorType>(filterType);

  // Reshape output and filter to the LHS and result of a "row-wise" matrix
  // multiplication.
  SmallVector<ReassociationIndices> filterReassocIndices = {{0}, {1, 2, 3}};
  auto reshapedFilterType =
      RankedTensorType::get({oc, fh * fw * ic}, filterType.getElementType(),
                            tensorFilterType.getEncoding());
  Value reshapedFilter = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedFilterType, filter, filterReassocIndices);

```
- **EN**: Implements logic around `getLoc`, `assert`, `get`, `getEncoding`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLoc`, `assert`, `get`, `getEncoding`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 604-617
```cpp
  SmallVector<ReassociationIndices> outputReassocIndices = {{0}, {1, 2}, {3}};
  RankedTensorType reshapedOutputType =
      RankedTensorType::get({n, oh * ow, oc}, outputType.getElementType());
  Value reshapedOutput = tensor::CollapseShapeOp::create(
      rewriter, loc, reshapedOutputType, output, outputReassocIndices);

  // Shape of the Toeplitz matrix produced by Im2col.
  SmallVector<int64_t> colTensorShape = {n, oh * ow, fh * fw * ic};
  Value colTensor = tensor::EmptyOp::create(rewriter, loc, colTensorShape,
                                            inputType.getElementType());

  // Convert the input to a (BMK) column tensor.
  auto nloops = colTensorShape.size();

```
- **EN**: Implements logic around `get`, `create`, `getElementType`, `size`; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `get`, `create`, `getElementType`, `size` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 618-634
```cpp
  auto parallel = utils::IteratorType::parallel;
  auto reduction = utils::IteratorType::reduction;
  SmallVector<utils::IteratorType> img2colIterators(nloops, parallel);

  // Given an index of the im2col matrix, retrieve the corresponding indices of
  // the output and filter matrices
  auto mIndicesExprs =
      delinearize(rewriter.getAffineDimExpr(1U), ArrayRef<int64_t>{ow, 1});
  auto kIndicesExprs = delinearize(rewriter.getAffineDimExpr(2U),
                                   ArrayRef<int64_t>{fw * ic, ic, 1});
  Im2ColToOperandsExprs i2cToOperExprs;
  i2cToOperExprs.fhIndex = kIndicesExprs[0];
  i2cToOperExprs.fwIndex = kIndicesExprs[1];
  i2cToOperExprs.icIndex = kIndicesExprs[2];
  i2cToOperExprs.ohIndex = mIndicesExprs[0];
  i2cToOperExprs.owIndex = mIndicesExprs[1];

```
- **EN**: Implements logic around `img2colIterators`, `delinearize`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `img2colIterators`, `delinearize` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 635-653
```cpp
  // im2col[n, oh*ow, fh*fw*ic] = input[n, sh*oh + fh, sw*ow + fw, ic]
  Im2ColToInputDimsExprs inExprs = getIm2ColInputExpressions(
      i2cToOperExprs, llvm::to_vector(convOp.getStrides().getValues<int64_t>()),
      rewriter);
  auto inMap =
      AffineMap::inferFromExprList({ArrayRef{inExprs.bIndex, inExprs.hIndex,
                                             inExprs.wIndex, inExprs.cIndex}},
                                   rewriter.getContext())[0];
  SmallVector<AffineMap> img2colIndexingMaps = {
      inMap, AffineMap::getMultiDimIdentityMap(nloops, context)};

  auto img2ColTensor = linalg::GenericOp::create(
      rewriter, loc, colTensor.getType(),
      /*inputs=*/input, /*outputs=*/colTensor, img2colIndexingMaps,
      img2colIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        linalg::YieldOp::create(nestedBuilder, nestedLoc, args[0]);
      });

```
- **EN**: Implements logic around `getIm2ColInputExpressions`, `to_vector`, `inferFromExprList`, `getContext`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIm2ColInputExpressions`, `to_vector`, `inferFromExprList`, `getContext`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 654-677
```cpp
  // Because we didn't transpose the filters we don't actually have a batched
  // matrix multiply. Instead, we have an operation consisting of "row-wise" dot
  // products.
  AffineExpr bDim, mDim, nDim, kDim;
  bindDims(context, bDim, mDim, nDim, kDim);
  auto lhsMap = AffineMap::get(4, 0, {bDim, mDim, kDim}, context);
  auto rhsMap = AffineMap::get(4, 0, {nDim, kDim}, context);
  auto resultMap = AffineMap::get(4, 0, {bDim, mDim, nDim}, context);
  SmallVector<utils::IteratorType> genericIterators = {parallel, parallel,
                                                       parallel, reduction};

  auto genericOp = linalg::GenericOp::create(
      rewriter, loc, reshapedOutputType,
      /*inputs=*/ValueRange{img2ColTensor.getResult(0), reshapedFilter},
      /*outputs=*/ValueRange{reshapedOutput},
      ArrayRef<AffineMap>{lhsMap, rhsMap, resultMap}, genericIterators,
      [&](OpBuilder &nestedBuilder, Location nestedLoc, ValueRange args) {
        Value mul =
            createMul(loc, args[0], args[1], args[2].getType(), nestedBuilder);
        Value add = createAdd(loc, mul, args[2], nestedBuilder);
        linalg::YieldOp::create(nestedBuilder, nestedLoc, add);
      });
  Value result = genericOp.getResults().front();

```
- **EN**: Implements logic around `bindDims`, `get`, `create`, `getResult`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `bindDims`, `get`, `create`, `getResult`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 678-691
```cpp
  auto reshapedResult = tensor::ExpandShapeOp::create(
      rewriter, loc, outputType, result, outputReassocIndices);

  rewriter.replaceOp(convOp, ArrayRef<Value>{reshapedResult});

  return std::make_pair(img2ColTensor.getOperation(),
                        reshapedResult.getOperation());
}

namespace {

class ConvertConv2DNhwcHwcf final
    : public OpRewritePattern<linalg::Conv2DNhwcHwcfOp> {
public:
```
- **EN**: Introduces declarations for `ConvertConv2DNhwcHwcf`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertConv2DNhwcHwcf` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 692-706
```cpp
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(linalg::Conv2DNhwcHwcfOp convOp,
                                PatternRewriter &rewriter) const override {
    if (failed(rewriteInIm2Col(rewriter, convOp)))
      return failure();
    return success();
  }
};

class ConvertDepthwiseConv2DNhwcHwc final
    : public OpRewritePattern<linalg::DepthwiseConv2DNhwcHwcOp> {
public:
  using OpRewritePattern<linalg::DepthwiseConv2DNhwcHwcOp>::OpRewritePattern;

```
- **EN**: Introduces declarations for `ConvertDepthwiseConv2DNhwcHwc`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertDepthwiseConv2DNhwcHwc` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 707-727
```cpp
  LogicalResult matchAndRewrite(linalg::DepthwiseConv2DNhwcHwcOp convOp,
                                PatternRewriter &rewriter) const override {
    if (failed(rewriteInIm2Col(rewriter, convOp)))
      return failure();
    return success();
  }
};

class ConvertConv2DNchwFchw final
    : public OpRewritePattern<linalg::Conv2DNchwFchwOp> {
public:
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(linalg::Conv2DNchwFchwOp convOp,
                                PatternRewriter &rewriter) const override {
    if (failed(rewriteInIm2Col(rewriter, convOp)))
      return failure();
    return success();
  }
};

```
- **EN**: Introduces declarations for `ConvertConv2DNchwFchw`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertConv2DNchwFchw` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 728-741
```cpp
class ConvertConv2DNhwcFhwc final
    : public OpRewritePattern<linalg::Conv2DNhwcFhwcOp> {
public:
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(linalg::Conv2DNhwcFhwcOp convOp,
                                PatternRewriter &rewriter) const override {
    if (failed(rewriteInIm2Col(rewriter, convOp)))
      return failure();
    return success();
  }
};
} // end anonymous namespace

```
- **EN**: Introduces declarations for `ConvertConv2DNhwcFhwc`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertConv2DNhwcFhwc` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 742-748
```cpp
void populateConvertConv2DToImg2ColPatterns(RewritePatternSet &patterns) {
  MLIRContext *context = patterns.getContext();
  patterns.insert<ConvertConv2DNhwcHwcf, ConvertDepthwiseConv2DNhwcHwc,
                  ConvertConv2DNchwFchw, ConvertConv2DNhwcFhwc>(context);
}
} // end namespace linalg
} // end namespace mlir
```
- **EN**: Introduces declarations for `linalg`, `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `linalg`, `mlir` 等声明，建立后续使用的方言级类型或模式记录。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/Dialect/Tensor/IR/Tensor.h`, `mlir/Dialect/Utils/IndexingUtils.h`, `mlir/Dialect/Utils/ReshapeOpsUtils.h`, `mlir/Dialect/Utils/StructuredOpsUtils.h`, `mlir/IR/AffineExpr.h` ... (+5 more)
- **Standard-library headers / 标准库头文件**: `<cassert>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (9), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (5), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
