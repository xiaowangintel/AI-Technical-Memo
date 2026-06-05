# EmulateWideInt.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Arith/Transforms/EmulateWideInt.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the Arith dialect and scalar/vector arithmetic semantics.
  - **CN**: 实现 Arith 方言与标量/向量算术语义 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```cpp
//===- EmulateWideInt.cpp - Wide integer operation emulation ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/Transforms/Passes.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Transforms/WideIntEmulationConverter.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/Func/Transforms/FuncConversions.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/Transforms/DialectConversion.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Transforms/WideIntEmulationConverter.h`, `mlir/Dialect/Arith/Utils/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Transforms/WideIntEmulationConverter.h`, `mlir/Dialect/Arith/Utils/Utils.h`。

### Lines 26-47
```cpp
namespace mlir::arith {
#define GEN_PASS_DEF_ARITHEMULATEWIDEINT
#include "mlir/Dialect/Arith/Transforms/Passes.h.inc"
} // namespace mlir::arith

using namespace mlir;

//===----------------------------------------------------------------------===//
// Common Helper Functions
//===----------------------------------------------------------------------===//

/// Returns N bottom and N top bits from `value`, where N = `newBitWidth`.
/// Treats `value` as a 2*N bits-wide integer.
/// The bottom bits are returned in the first pair element, while the top bits
/// in the second one.
static std::pair<APInt, APInt> getHalves(const APInt &value,
                                         unsigned newBitWidth) {
  APInt low = value.extractBits(newBitWidth, 0);
  APInt high = value.extractBits(newBitWidth, newBitWidth);
  return {std::move(low), std::move(high)};
}

```
- **EN**: Introduces declarations for `mlir::arith`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arith` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 48-65
```cpp
/// Returns the type with the last (innermost) dimension reduced to x1.
/// Scalarizes 1D vector inputs to match how we extract/insert vector values,
/// e.g.:
///   - vector<3x2xi16> --> vector<3x1xi16>
///   - vector<2xi16>   --> i16
static Type reduceInnermostDim(VectorType type) {
  if (type.getShape().size() == 1)
    return type.getElementType();

  auto newShape = to_vector(type.getShape());
  newShape.back() = 1;
  return VectorType::get(newShape, type.getElementType());
}

/// Extracts the `input` vector slice with elements at the last dimension offset
/// by `lastOffset`. Returns a value of vector type with the last dimension
/// reduced to x1 or fully scalarized, e.g.:
///   - vector<3x2xi16> --> vector<3x1xi16>
```
- **EN**: Implements logic around `reduceInnermostDim`, `getShape`, `getElementType`, `to_vector`, and 2 more symbols.
- **CN**: 围绕 `reduceInnermostDim`, `getShape`, `getElementType`, `to_vector`, and 2 more symbols 实现具体逻辑。

### Lines 66-86
```cpp
///   - vector<2xi16>   --> i16
static Value extractLastDimSlice(ConversionPatternRewriter &rewriter,
                                 Location loc, Value input,
                                 int64_t lastOffset) {
  ArrayRef<int64_t> shape = cast<VectorType>(input.getType()).getShape();
  assert(lastOffset < shape.back() && "Offset out of bounds");

  // Scalarize the result in case of 1D vectors.
  if (shape.size() == 1)
    return vector::ExtractOp::create(rewriter, loc, input, lastOffset);

  SmallVector<int64_t> offsets(shape.size(), 0);
  offsets.back() = lastOffset;
  auto sizes = llvm::to_vector(shape);
  sizes.back() = 1;
  SmallVector<int64_t> strides(shape.size(), 1);

  return vector::ExtractStridedSliceOp::create(rewriter, loc, input, offsets,
                                               sizes, strides);
}

```
- **EN**: Implements logic around `extractLastDimSlice`, `cast`, `assert`, `size`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractLastDimSlice`, `cast`, `assert`, `size`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 87-108
```cpp
/// Extracts two vector slices from the `input` whose type is `vector<...x2T>`,
/// with the first element at offset 0 and the second element at offset 1.
static std::pair<Value, Value>
extractLastDimHalves(ConversionPatternRewriter &rewriter, Location loc,
                     Value input) {
  return {extractLastDimSlice(rewriter, loc, input, 0),
          extractLastDimSlice(rewriter, loc, input, 1)};
}

// Performs a vector shape cast to drop the trailing x1 dimension. If the
// `input` is a scalar, this is a noop.
static Value dropTrailingX1Dim(ConversionPatternRewriter &rewriter,
                               Location loc, Value input) {
  auto vecTy = dyn_cast<VectorType>(input.getType());
  if (!vecTy)
    return input;

  // Shape cast to drop the last x1 dimension.
  ArrayRef<int64_t> shape = vecTy.getShape();
  assert(shape.size() >= 2 && "Expected vector with at list two dims");
  assert(shape.back() == 1 && "Expected the last vector dim to be x1");

```
- **EN**: Implements logic around `extractLastDimHalves`, `extractLastDimSlice`, `dropTrailingX1Dim`, `dyn_cast`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractLastDimHalves`, `extractLastDimSlice`, `dropTrailingX1Dim`, `dyn_cast`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 109-127
```cpp
  auto newVecTy = VectorType::get(shape.drop_back(), vecTy.getElementType());
  return vector::ShapeCastOp::create(rewriter, loc, newVecTy, input);
}

/// Performs a vector shape cast to append an x1 dimension. If the
/// `input` is a scalar, this is a noop.
static Value appendX1Dim(ConversionPatternRewriter &rewriter, Location loc,
                         Value input) {
  auto vecTy = dyn_cast<VectorType>(input.getType());
  if (!vecTy)
    return input;

  // Add a trailing x1 dim.
  auto newShape = llvm::to_vector(vecTy.getShape());
  newShape.push_back(1);
  auto newTy = VectorType::get(newShape, vecTy.getElementType());
  return vector::ShapeCastOp::create(rewriter, loc, newTy, input);
}

```
- **EN**: Implements logic around `get`, `create`, `appendX1Dim`, `dyn_cast`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `get`, `create`, `appendX1Dim`, `dyn_cast`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 128-147
```cpp
/// Inserts the `source` vector slice into the `dest` vector at offset
/// `lastOffset` in the last dimension. `source` can be a scalar when `dest` is
/// a 1D vector.
static Value insertLastDimSlice(ConversionPatternRewriter &rewriter,
                                Location loc, Value source, Value dest,
                                int64_t lastOffset) {
  ArrayRef<int64_t> shape = cast<VectorType>(dest.getType()).getShape();
  assert(lastOffset < shape.back() && "Offset out of bounds");

  // Handle scalar source.
  if (isa<IntegerType>(source.getType()))
    return vector::InsertOp::create(rewriter, loc, source, dest, lastOffset);

  SmallVector<int64_t> offsets(shape.size(), 0);
  offsets.back() = lastOffset;
  SmallVector<int64_t> strides(shape.size(), 1);
  return vector::InsertStridedSliceOp::create(rewriter, loc, source, dest,
                                              offsets, strides);
}

```
- **EN**: Implements logic around `insertLastDimSlice`, `cast`, `assert`, `isa`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `insertLastDimSlice`, `cast`, `assert`, `isa`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 148-166
```cpp
/// Constructs a new vector of type `resultType` by creating a series of
/// insertions of `resultComponents`, each at the next offset of the last vector
/// dimension.
/// When all `resultComponents` are scalars, the result type is `vector<NxT>`;
/// when `resultComponents` are `vector<...x1xT>`s, the result type is
/// `vector<...xNxT>`, where `N` is the number of `resultComponents`.
static Value constructResultVector(ConversionPatternRewriter &rewriter,
                                   Location loc, VectorType resultType,
                                   ValueRange resultComponents) {
  llvm::ArrayRef<int64_t> resultShape = resultType.getShape();
  (void)resultShape;
  assert(!resultShape.empty() && "Result expected to have dimensions");
  assert(resultShape.back() == static_cast<int64_t>(resultComponents.size()) &&
         "Wrong number of result components");

  Value resultVec = createScalarOrSplatConstant(rewriter, loc, resultType, 0);
  for (auto [i, component] : llvm::enumerate(resultComponents))
    resultVec = insertLastDimSlice(rewriter, loc, component, resultVec, i);

```
- **EN**: Implements logic around `constructResultVector`, `getShape`, `assert`, `createScalarOrSplatConstant`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `constructResultVector`, `getShape`, `assert`, `createScalarOrSplatConstant`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 167-186
```cpp
  return resultVec;
}

namespace {
//===----------------------------------------------------------------------===//
// ConvertConstant
//===----------------------------------------------------------------------===//

struct ConvertConstant final : OpConversionPattern<arith::ConstantOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::ConstantOp op, OpAdaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Type oldType = op.getType();
    auto newType = getTypeConverter()->convertType<VectorType>(oldType);
    if (!newType)
      return rewriter.notifyMatchFailure(
          op, llvm::formatv("unsupported type: {0}", op.getType()));

```
- **EN**: Introduces declarations for `ConvertConstant`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertConstant` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 187-207
```cpp
    unsigned newBitWidth = newType.getElementTypeBitWidth();
    Attribute oldValue = op.getValueAttr();

    if (auto intAttr = dyn_cast<IntegerAttr>(oldValue)) {
      auto [low, high] = getHalves(intAttr.getValue(), newBitWidth);
      auto newAttr = DenseElementsAttr::get(newType, {low, high});
      rewriter.replaceOpWithNewOp<arith::ConstantOp>(op, newAttr);
      return success();
    }

    if (auto splatAttr = dyn_cast<SplatElementsAttr>(oldValue)) {
      auto [low, high] =
          getHalves(splatAttr.getSplatValue<APInt>(), newBitWidth);
      int64_t numSplatElems = splatAttr.getNumElements();
      SmallVector<APInt> values;
      values.reserve(numSplatElems * 2);
      for (int64_t i = 0; i < numSplatElems; ++i) {
        values.push_back(low);
        values.push_back(high);
      }

```
- **EN**: Implements logic around `getElementTypeBitWidth`, `getValueAttr`, `dyn_cast`, `getHalves`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getElementTypeBitWidth`, `getValueAttr`, `dyn_cast`, `getHalves`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 208-227
```cpp
      auto attr = DenseElementsAttr::get(newType, values);
      rewriter.replaceOpWithNewOp<arith::ConstantOp>(op, attr);
      return success();
    }

    if (auto elemsAttr = dyn_cast<DenseElementsAttr>(oldValue)) {
      int64_t numElems = elemsAttr.getNumElements();
      SmallVector<APInt> values;
      values.reserve(numElems * 2);
      for (const APInt &origVal : elemsAttr.getValues<APInt>()) {
        auto [low, high] = getHalves(origVal, newBitWidth);
        values.push_back(std::move(low));
        values.push_back(std::move(high));
      }

      auto attr = DenseElementsAttr::get(newType, values);
      rewriter.replaceOpWithNewOp<arith::ConstantOp>(op, attr);
      return success();
    }

```
- **EN**: Implements logic around `get`, `ConstantOp>`, `success`, `dyn_cast`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `get`, `ConstantOp>`, `success`, `dyn_cast`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 228-248
```cpp
    return rewriter.notifyMatchFailure(op.getLoc(),
                                       "unhandled constant attribute");
  }
};

//===----------------------------------------------------------------------===//
// ConvertAddI
//===----------------------------------------------------------------------===//

struct ConvertAddI final : OpConversionPattern<arith::AddIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::AddIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

```
- **EN**: Introduces declarations for `ConvertAddI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertAddI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 249-270
```cpp
    Type newElemTy = reduceInnermostDim(newTy);

    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    auto [rhsElem0, rhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getRhs());

    auto lowSum =
        arith::AddUIExtendedOp::create(rewriter, loc, lhsElem0, rhsElem0);
    Value overflowVal =
        arith::ExtUIOp::create(rewriter, loc, newElemTy, lowSum.getOverflow());

    Value high0 = arith::AddIOp::create(rewriter, loc, overflowVal, lhsElem1);
    Value high = arith::AddIOp::create(rewriter, loc, high0, rhsElem1);

    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {lowSum.getSum(), high});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

```
- **EN**: Implements logic around `reduceInnermostDim`, `extractLastDimHalves`, `create`, `constructResultVector`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `reduceInnermostDim`, `extractLastDimHalves`, `create`, `constructResultVector`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 271-290
```cpp
//===----------------------------------------------------------------------===//
// ConvertBitwiseBinary
//===----------------------------------------------------------------------===//

/// Conversion pattern template for bitwise binary ops, e.g., `arith.andi`.
template <typename BinaryOp>
struct ConvertBitwiseBinary final : OpConversionPattern<BinaryOp> {
  using OpConversionPattern<BinaryOp>::OpConversionPattern;
  using OpAdaptor = typename OpConversionPattern<BinaryOp>::OpAdaptor;

  LogicalResult
  matchAndRewrite(BinaryOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = this->getTypeConverter()->template convertType<VectorType>(
        op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

```
- **EN**: Introduces declarations for `ConvertBitwiseBinary`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertBitwiseBinary` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 291-308
```cpp
    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    auto [rhsElem0, rhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getRhs());

    Value resElem0 = BinaryOp::create(rewriter, loc, lhsElem0, rhsElem0);
    Value resElem1 = BinaryOp::create(rewriter, loc, lhsElem1, rhsElem1);
    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {resElem0, resElem1});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertCmpI
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `extractLastDimHalves`, `create`, `constructResultVector`, `replaceOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractLastDimHalves`, `create`, `constructResultVector`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 309-326
```cpp
/// Returns the matching unsigned version of the given predicate `pred`, or the
/// same predicate if `pred` is not a signed.
static arith::CmpIPredicate toUnsignedPredicate(arith::CmpIPredicate pred) {
  using P = arith::CmpIPredicate;
  switch (pred) {
  case P::sge:
    return P::uge;
  case P::sgt:
    return P::ugt;
  case P::sle:
    return P::ule;
  case P::slt:
    return P::ult;
  default:
    return pred;
  }
}

```
- **EN**: Implements logic around `toUnsignedPredicate`.
- **CN**: 围绕 `toUnsignedPredicate` 实现具体逻辑。

### Lines 327-347
```cpp
struct ConvertCmpI final : OpConversionPattern<arith::CmpIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::CmpIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto inputTy =
        getTypeConverter()->convertType<VectorType>(op.getLhs().getType());
    if (!inputTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    arith::CmpIPredicate highPred = adaptor.getPredicate();
    arith::CmpIPredicate lowPred = toUnsignedPredicate(highPred);

    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    auto [rhsElem0, rhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getRhs());

```
- **EN**: Introduces declarations for `ConvertCmpI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertCmpI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 348-372
```cpp
    Value lowCmp =
        arith::CmpIOp::create(rewriter, loc, lowPred, lhsElem0, rhsElem0);
    Value highCmp =
        arith::CmpIOp::create(rewriter, loc, highPred, lhsElem1, rhsElem1);

    Value cmpResult{};
    switch (highPred) {
    case arith::CmpIPredicate::eq: {
      cmpResult = arith::AndIOp::create(rewriter, loc, lowCmp, highCmp);
      break;
    }
    case arith::CmpIPredicate::ne: {
      cmpResult = arith::OrIOp::create(rewriter, loc, lowCmp, highCmp);
      break;
    }
    default: {
      // Handle inequality checks.
      Value highEq = arith::CmpIOp::create(
          rewriter, loc, arith::CmpIPredicate::eq, lhsElem1, rhsElem1);
      cmpResult =
          arith::SelectOp::create(rewriter, loc, highEq, lowCmp, highCmp);
      break;
    }
    }

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 373-394
```cpp
    assert(cmpResult && "Unhandled case");
    rewriter.replaceOp(op, dropTrailingX1Dim(rewriter, loc, cmpResult));
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertMulI
//===----------------------------------------------------------------------===//

struct ConvertMulI final : OpConversionPattern<arith::MulIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::MulIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

```
- **EN**: Introduces declarations for `ConvertMulI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertMulI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 395-412
```cpp
    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    auto [rhsElem0, rhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getRhs());

    // The multiplication algorithm used is the standard (long) multiplication.
    // Multiplying two i2N integers produces (at most) an i4N result, but
    // because the calculation of top i2N is not necessary, we omit it.
    auto mulLowLow =
        arith::MulUIExtendedOp::create(rewriter, loc, lhsElem0, rhsElem0);
    Value mulLowHi = arith::MulIOp::create(rewriter, loc, lhsElem0, rhsElem1);
    Value mulHiLow = arith::MulIOp::create(rewriter, loc, lhsElem1, rhsElem0);

    Value resLow = mulLowLow.getLow();
    Value resHi =
        arith::AddIOp::create(rewriter, loc, mulLowLow.getHigh(), mulLowHi);
    resHi = arith::AddIOp::create(rewriter, loc, resHi, mulHiLow);

```
- **EN**: Implements logic around `extractLastDimHalves`, `create`, `getLow`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractLastDimHalves`, `create`, `getLow` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 413-435
```cpp
    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {resLow, resHi});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertExtSI
//===----------------------------------------------------------------------===//

struct ConvertExtSI final : OpConversionPattern<arith::ExtSIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::ExtSIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

```
- **EN**: Introduces declarations for `ConvertExtSI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertExtSI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 436-457
```cpp
    Type newResultComponentTy = reduceInnermostDim(newTy);

    // Sign-extend the input value to determine the low half of the result.
    // Then, check if the low half is negative, and sign-extend the comparison
    // result to get the high half.
    Value newOperand = appendX1Dim(rewriter, loc, adaptor.getIn());
    Value extended = rewriter.createOrFold<arith::ExtSIOp>(
        loc, newResultComponentTy, newOperand);
    Value operandZeroCst =
        createScalarOrSplatConstant(rewriter, loc, newResultComponentTy, 0);
    Value signBit = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::slt, extended, operandZeroCst);
    Value signValue =
        arith::ExtSIOp::create(rewriter, loc, newResultComponentTy, signBit);

    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {extended, signValue});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

```
- **EN**: Implements logic around `reduceInnermostDim`, `appendX1Dim`, `ExtSIOp>`, `createScalarOrSplatConstant`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `reduceInnermostDim`, `appendX1Dim`, `ExtSIOp>`, `createScalarOrSplatConstant`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 458-475
```cpp
//===----------------------------------------------------------------------===//
// ConvertExtUI
//===----------------------------------------------------------------------===//

struct ConvertExtUI final : OpConversionPattern<arith::ExtUIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::ExtUIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    Type newResultComponentTy = reduceInnermostDim(newTy);

```
- **EN**: Introduces declarations for `ConvertExtUI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertExtUI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 476-493
```cpp
    // Zero-extend the input value to determine the low half of the result.
    // The high half is always zero.
    Value newOperand = appendX1Dim(rewriter, loc, adaptor.getIn());
    Value extended = rewriter.createOrFold<arith::ExtUIOp>(
        loc, newResultComponentTy, newOperand);
    Value zeroCst = createScalarOrSplatConstant(rewriter, loc, newTy, 0);
    Value newRes = insertLastDimSlice(rewriter, loc, extended, zeroCst, 0);
    rewriter.replaceOp(op, newRes);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertMaxMin
//===----------------------------------------------------------------------===//

template <typename SourceOp, arith::CmpIPredicate CmpPred>
struct ConvertMaxMin final : OpConversionPattern<SourceOp> {
```
- **EN**: Introduces declarations for `ConvertMaxMin`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertMaxMin` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 494-517
```cpp
  using OpConversionPattern<SourceOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(SourceOp op, typename SourceOp::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();

    Type oldTy = op.getType();
    auto newTy = dyn_cast_or_null<VectorType>(
        this->getTypeConverter()->convertType(oldTy));
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    // Rewrite Max*I/Min*I as compare and select over original operands. Let
    // the CmpI and Select emulation patterns handle the final legalization.
    Value cmp =
        arith::CmpIOp::create(rewriter, loc, CmpPred, op.getLhs(), op.getRhs());
    rewriter.replaceOpWithNewOp<arith::SelectOp>(op, cmp, op.getLhs(),
                                                 op.getRhs());
    return success();
  }
};

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getType`, `dyn_cast_or_null`, and 7 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getType`, `dyn_cast_or_null`, and 7 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 518-536
```cpp
// Convert IndexCast ops
//===----------------------------------------------------------------------===//

/// Returns true iff the type is `index` or `vector<...index>`.
static bool isIndexOrIndexVector(Type type) {
  if (isa<IndexType>(type))
    return true;

  if (auto vectorTy = dyn_cast<VectorType>(type))
    if (isa<IndexType>(vectorTy.getElementType()))
      return true;

  return false;
}

template <typename CastOp>
struct ConvertIndexCastIntToIndex final : OpConversionPattern<CastOp> {
  using OpConversionPattern<CastOp>::OpConversionPattern;

```
- **EN**: Introduces declarations for `ConvertIndexCastIntToIndex`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertIndexCastIntToIndex` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 537-559
```cpp
  LogicalResult
  matchAndRewrite(CastOp op, typename CastOp::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Type resultType = op.getType();
    if (!isIndexOrIndexVector(resultType))
      return failure();

    Location loc = op.getLoc();
    Type inType = op.getIn().getType();
    auto newInTy =
        this->getTypeConverter()->template convertType<VectorType>(inType);
    if (!newInTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", inType));

    // Discard the high half of the input truncating the original value.
    Value extracted = extractLastDimSlice(rewriter, loc, adaptor.getIn(), 0);
    extracted = dropTrailingX1Dim(rewriter, loc, extracted);
    rewriter.replaceOpWithNewOp<CastOp>(op, resultType, extracted);
    return success();
  }
};

```
- **EN**: Implements logic around `matchAndRewrite`, `getType`, `isIndexOrIndexVector`, `failure`, and 9 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getType`, `isIndexOrIndexVector`, `failure`, and 9 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 560-580
```cpp
template <typename CastOp, typename ExtensionOp>
struct ConvertIndexCastIndexToInt final : OpConversionPattern<CastOp> {
  using OpConversionPattern<CastOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(CastOp op, typename CastOp::Adaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Type inType = op.getIn().getType();
    if (!isIndexOrIndexVector(inType))
      return failure();

    Location loc = op.getLoc();
    auto *typeConverter =
        this->template getTypeConverter<arith::WideIntEmulationConverter>();

    Type resultType = op.getType();
    auto newTy = typeConverter->template convertType<VectorType>(resultType);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", resultType));

```
- **EN**: Introduces declarations for `ConvertIndexCastIndexToInt`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertIndexCastIndexToInt` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 581-598
```cpp
    // Emit an index cast over the matching narrow type.
    Type narrowTy =
        rewriter.getIntegerType(typeConverter->getMaxTargetIntBitWidth());
    if (auto vecTy = dyn_cast<VectorType>(resultType))
      narrowTy = VectorType::get(vecTy.getShape(), narrowTy);

    // Sign or zero-extend the result. Let the matching conversion pattern
    // legalize the extension op.
    Value underlyingVal =
        CastOp::create(rewriter, loc, narrowTy, adaptor.getIn());
    rewriter.replaceOpWithNewOp<ExtensionOp>(op, resultType, underlyingVal);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertSelect
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getIntegerType`, `dyn_cast`, `get`, `create`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getIntegerType`, `dyn_cast`, `get`, `create`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 599-617
```cpp

struct ConvertSelect final : OpConversionPattern<arith::SelectOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::SelectOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    auto [trueElem0, trueElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getTrueValue());
    auto [falseElem0, falseElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getFalseValue());
    Value cond = appendX1Dim(rewriter, loc, adaptor.getCondition());

```
- **EN**: Introduces declarations for `ConvertSelect`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertSelect` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 618-635
```cpp
    Value resElem0 =
        arith::SelectOp::create(rewriter, loc, cond, trueElem0, falseElem0);
    Value resElem1 =
        arith::SelectOp::create(rewriter, loc, cond, trueElem1, falseElem1);
    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {resElem0, resElem1});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertShLI
//===----------------------------------------------------------------------===//

struct ConvertShLI final : OpConversionPattern<arith::ShLIOp> {
  using Base::Base;

```
- **EN**: Introduces declarations for `ConvertShLI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertShLI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 636-654
```cpp
  LogicalResult
  matchAndRewrite(arith::ShLIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();

    Type oldTy = op.getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(oldTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    Type newOperandTy = reduceInnermostDim(newTy);
    // `oldBitWidth` == `2 * newBitWidth`
    unsigned newBitWidth = newTy.getElementTypeBitWidth();

    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    Value rhsElem0 = extractLastDimSlice(rewriter, loc, adaptor.getRhs(), 0);

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getType`, `getTypeConverter`, and 6 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getType`, `getTypeConverter`, and 6 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 655-682
```cpp
    // Assume that the shift amount is < 2 * newBitWidth. Calculate the low and
    // high halves of the results separately:
    //   1. low := LHS.low shli RHS
    //
    //   2. high := a or b or c, where:
    //     a) Bits from LHS.high, shifted by the RHS.
    //     b) Bits from LHS.low, shifted right. These come into play when
    //        RHS < newBitWidth, e.g.:
    //         [0000][llll] shli 3 --> [0lll][l000]
    //                                    ^
    //                                    |
    //                           [llll] shrui (4 - 3)
    //     c) Bits from LHS.low, shifted left. These matter when
    //        RHS > newBitWidth, e.g.:
    //         [0000][llll] shli 7 --> [l000][0000]
    //                                   ^
    //                                   |
    //                          [llll] shli (7 - 4)
    //
    // Because shifts by values >= newBitWidth are undefined, we ignore the high
    // half of RHS, and introduce 'bounds checks' to account for
    // RHS.low > newBitWidth.
    //
    // TODO: Explore possible optimizations.
    Value zeroCst = createScalarOrSplatConstant(rewriter, loc, newOperandTy, 0);
    Value elemBitWidth =
        createScalarOrSplatConstant(rewriter, loc, newOperandTy, newBitWidth);

```
- **EN**: Implements logic around `createScalarOrSplatConstant`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `createScalarOrSplatConstant` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 683-701
```cpp
    Value illegalElemShift = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::uge, rhsElem0, elemBitWidth);

    Value shiftedElem0 =
        arith::ShLIOp::create(rewriter, loc, lhsElem0, rhsElem0);
    Value resElem0 = arith::SelectOp::create(rewriter, loc, illegalElemShift,
                                             zeroCst, shiftedElem0);

    Value cappedShiftAmount = arith::SelectOp::create(
        rewriter, loc, illegalElemShift, elemBitWidth, rhsElem0);
    Value rightShiftAmount =
        arith::SubIOp::create(rewriter, loc, elemBitWidth, cappedShiftAmount);
    Value shiftedRight =
        arith::ShRUIOp::create(rewriter, loc, lhsElem0, rightShiftAmount);
    Value overshotShiftAmount =
        arith::SubIOp::create(rewriter, loc, rhsElem0, elemBitWidth);
    Value shiftedLeft =
        arith::ShLIOp::create(rewriter, loc, lhsElem0, overshotShiftAmount);

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 702-720
```cpp
    Value shiftedElem1 =
        arith::ShLIOp::create(rewriter, loc, lhsElem1, rhsElem0);
    Value resElem1High = arith::SelectOp::create(
        rewriter, loc, illegalElemShift, zeroCst, shiftedElem1);
    Value resElem1Low = arith::SelectOp::create(rewriter, loc, illegalElemShift,
                                                shiftedLeft, shiftedRight);
    Value resElem1 =
        arith::OrIOp::create(rewriter, loc, resElem1Low, resElem1High);

    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {resElem0, resElem1});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertShRUI
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `create`, `constructResultVector`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `constructResultVector`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 721-739
```cpp

struct ConvertShRUI final : OpConversionPattern<arith::ShRUIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::ShRUIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();

    Type oldTy = op.getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(oldTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    Type newOperandTy = reduceInnermostDim(newTy);
    // `oldBitWidth` == `2 * newBitWidth`
    unsigned newBitWidth = newTy.getElementTypeBitWidth();

```
- **EN**: Introduces declarations for `ConvertShRUI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertShRUI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 740-771
```cpp
    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    Value rhsElem0 = extractLastDimSlice(rewriter, loc, adaptor.getRhs(), 0);

    // Assume that the shift amount is < 2 * newBitWidth. Calculate the low and
    // high halves of the results separately:
    //   1. low := a or b or c, where:
    //     a) Bits from LHS.low, shifted by the RHS.
    //     b) Bits from LHS.high, shifted left. These matter when
    //        RHS < newBitWidth, e.g.:
    //         [hhhh][0000] shrui 3 --> [000h][hhh0]
    //                                          ^
    //                                          |
    //                                 [hhhh] shli (4 - 1)
    //     c) Bits from LHS.high, shifted right. These come into play when
    //        RHS > newBitWidth, e.g.:
    //         [hhhh][0000] shrui 7 --> [0000][000h]
    //                                          ^
    //                                          |
    //                                 [hhhh] shrui (7 - 4)
    //
    //   2. high := LHS.high shrui RHS
    //
    // Because shifts by values >= newBitWidth are undefined, we ignore the high
    // half of RHS, and introduce 'bounds checks' to account for
    // RHS.low > newBitWidth.
    //
    // TODO: Explore possible optimizations.
    Value zeroCst = createScalarOrSplatConstant(rewriter, loc, newOperandTy, 0);
    Value elemBitWidth =
        createScalarOrSplatConstant(rewriter, loc, newOperandTy, newBitWidth);

```
- **EN**: Implements logic around `extractLastDimHalves`, `extractLastDimSlice`, `createScalarOrSplatConstant`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractLastDimHalves`, `extractLastDimSlice`, `createScalarOrSplatConstant` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 772-794
```cpp
    Value illegalElemShift = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::uge, rhsElem0, elemBitWidth);

    Value shiftedElem0 =
        arith::ShRUIOp::create(rewriter, loc, lhsElem0, rhsElem0);
    Value resElem0Low = arith::SelectOp::create(rewriter, loc, illegalElemShift,
                                                zeroCst, shiftedElem0);
    Value shiftedElem1 =
        arith::ShRUIOp::create(rewriter, loc, lhsElem1, rhsElem0);
    Value resElem1 = arith::SelectOp::create(rewriter, loc, illegalElemShift,
                                             zeroCst, shiftedElem1);

    Value cappedShiftAmount = arith::SelectOp::create(
        rewriter, loc, illegalElemShift, elemBitWidth, rhsElem0);
    Value leftShiftAmount =
        arith::SubIOp::create(rewriter, loc, elemBitWidth, cappedShiftAmount);
    Value shiftedLeft =
        arith::ShLIOp::create(rewriter, loc, lhsElem1, leftShiftAmount);
    Value overshotShiftAmount =
        arith::SubIOp::create(rewriter, loc, rhsElem0, elemBitWidth);
    Value shiftedRight =
        arith::ShRUIOp::create(rewriter, loc, lhsElem1, overshotShiftAmount);

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 795-813
```cpp
    Value resElem0High = arith::SelectOp::create(
        rewriter, loc, illegalElemShift, shiftedRight, shiftedLeft);
    Value resElem0 =
        arith::OrIOp::create(rewriter, loc, resElem0Low, resElem0High);

    Value resultVec =
        constructResultVector(rewriter, loc, newTy, {resElem0, resElem1});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertShRSI
//===----------------------------------------------------------------------===//

struct ConvertShRSI final : OpConversionPattern<arith::ShRSIOp> {
  using Base::Base;

```
- **EN**: Introduces declarations for `ConvertShRSI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertShRSI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 814-838
```cpp
  LogicalResult
  matchAndRewrite(arith::ShRSIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();

    Type oldTy = op.getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(oldTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", op.getType()));

    Value lhsElem1 = extractLastDimSlice(rewriter, loc, adaptor.getLhs(), 1);
    Value rhsElem0 = extractLastDimSlice(rewriter, loc, adaptor.getRhs(), 0);

    Type narrowTy = rhsElem0.getType();
    int64_t origBitwidth = newTy.getElementTypeBitWidth() * 2;

    // Rewrite this as an bitwise or of `arith.shrui` and sign extension bits.
    // Perform as many ops over the narrow integer type as possible and let the
    // other emulation patterns convert the rest.
    Value elemZero = createScalarOrSplatConstant(rewriter, loc, narrowTy, 0);
    Value signBit = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::slt, lhsElem1, elemZero);
    signBit = dropTrailingX1Dim(rewriter, loc, signBit);

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getType`, `getTypeConverter`, and 7 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getType`, `getTypeConverter`, and 7 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 839-857
```cpp
    // Create a bit pattern of either all ones or all zeros. Then shift it left
    // to calculate the sign extension bits created by shifting the original
    // sign bit right.
    Value allSign = arith::ExtSIOp::create(rewriter, loc, oldTy, signBit);
    Value maxShift =
        createScalarOrSplatConstant(rewriter, loc, narrowTy, origBitwidth);
    Value numNonSignExtBits =
        arith::SubIOp::create(rewriter, loc, maxShift, rhsElem0);
    numNonSignExtBits = dropTrailingX1Dim(rewriter, loc, numNonSignExtBits);
    numNonSignExtBits =
        arith::ExtUIOp::create(rewriter, loc, oldTy, numNonSignExtBits);
    Value signBits =
        arith::ShLIOp::create(rewriter, loc, allSign, numNonSignExtBits);

    // Use original arguments to create the right shift.
    Value shrui =
        arith::ShRUIOp::create(rewriter, loc, op.getLhs(), op.getRhs());
    Value shrsi = arith::OrIOp::create(rewriter, loc, shrui, signBits);

```
- **EN**: Implements logic around `create`, `createScalarOrSplatConstant`, `dropTrailingX1Dim`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `createScalarOrSplatConstant`, `dropTrailingX1Dim` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 858-876
```cpp
    // Handle shifting by zero. This is necessary when the `signBits` shift is
    // invalid.
    Value isNoop = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::eq, rhsElem0, elemZero);
    isNoop = dropTrailingX1Dim(rewriter, loc, isNoop);
    rewriter.replaceOpWithNewOp<arith::SelectOp>(op, isNoop, op.getLhs(),
                                                 shrsi);

    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertSubI
//===----------------------------------------------------------------------===//

struct ConvertSubI final : OpConversionPattern<arith::SubIOp> {
  using Base::Base;

```
- **EN**: Introduces declarations for `ConvertSubI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertSubI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 877-900
```cpp
  LogicalResult
  matchAndRewrite(arith::SubIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op->getLoc();
    auto newTy = getTypeConverter()->convertType<VectorType>(op.getType());
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {}", op.getType()));

    Type newElemTy = reduceInnermostDim(newTy);

    auto [lhsElem0, lhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getLhs());
    auto [rhsElem0, rhsElem1] =
        extractLastDimHalves(rewriter, loc, adaptor.getRhs());

    // Emulates LHS - RHS by [LHS0 - RHS0, LHS1 - RHS1 - CARRY] where
    // CARRY is 1 or 0.
    Value low = arith::SubIOp::create(rewriter, loc, lhsElem0, rhsElem0);
    // We have a carry if lhsElem0 < rhsElem0.
    Value carry0 = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::ult, lhsElem0, rhsElem0);
    Value carryVal = arith::ExtUIOp::create(rewriter, loc, newElemTy, carry0);

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getTypeConverter`, `notifyMatchFailure`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getTypeConverter`, `notifyMatchFailure`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 901-921
```cpp
    Value high0 = arith::SubIOp::create(rewriter, loc, lhsElem1, carryVal);
    Value high = arith::SubIOp::create(rewriter, loc, high0, rhsElem1);

    Value resultVec = constructResultVector(rewriter, loc, newTy, {low, high});
    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertSIToFP
//===----------------------------------------------------------------------===//

struct ConvertSIToFP final : OpConversionPattern<arith::SIToFPOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::SIToFPOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();

```
- **EN**: Introduces declarations for `ConvertSIToFP`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertSIToFP` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 922-940
```cpp
    Value in = op.getIn();
    Type oldTy = in.getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(oldTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", oldTy));

    Value zeroCst = createScalarOrSplatConstant(rewriter, loc, oldTy, 0);

    // To avoid operating on very large unsigned numbers, perform the
    // conversion on the absolute value. Then, decide whether to negate the
    // result or not based on that sign bit. We implement negation by
    // subtracting from zero. Note that this relies on the the other conversion
    // patterns to legalize created ops and narrow the bit widths.
    Value isNeg = arith::CmpIOp::create(rewriter, loc,
                                        arith::CmpIPredicate::slt, in, zeroCst);
    Value neg = arith::SubIOp::create(rewriter, loc, zeroCst, in);
    Value abs = arith::SelectOp::create(rewriter, loc, isNeg, neg, in);

```
- **EN**: Implements logic around `getIn`, `getType`, `getTypeConverter`, `notifyMatchFailure`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getIn`, `getType`, `getTypeConverter`, `notifyMatchFailure`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 941-960
```cpp
    Value absResult = arith::UIToFPOp::create(rewriter, loc, op.getType(), abs);
    Value negResult = arith::NegFOp::create(rewriter, loc, absResult);
    rewriter.replaceOpWithNewOp<arith::SelectOp>(op, isNeg, negResult,
                                                 absResult);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertUIToFP
//===----------------------------------------------------------------------===//

struct ConvertUIToFP final : OpConversionPattern<arith::UIToFPOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::UIToFPOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();

```
- **EN**: Introduces declarations for `ConvertUIToFP`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertUIToFP` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 961-990
```cpp
    Type oldTy = op.getIn().getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(oldTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {0}", oldTy));
    unsigned newBitWidth = newTy.getElementTypeBitWidth();

    auto [low, hi] = extractLastDimHalves(rewriter, loc, adaptor.getIn());
    Value lowInt = dropTrailingX1Dim(rewriter, loc, low);
    Value hiInt = dropTrailingX1Dim(rewriter, loc, hi);
    Value zeroCst =
        createScalarOrSplatConstant(rewriter, loc, hiInt.getType(), 0);

    // The final result has the following form:
    //   if (hi == 0) return uitofp(low)
    //   else         return uitofp(low) + uitofp(hi) * 2^BW
    //
    // where `BW` is the bitwidth of the narrowed integer type. We emit a
    // select to make it easier to fold-away the `hi` part calculation when it
    // is known to be zero.
    //
    // Note 1: The emulation is precise only for input values that have exact
    // integer representation in the result floating point type, and may lead
    // loss of precision otherwise.
    //
    // Note 2: We do not strictly need the `hi == 0`, case, but it makes
    // constant folding easier.
    Value hiEqZero = arith::CmpIOp::create(
        rewriter, loc, arith::CmpIPredicate::eq, hiInt, zeroCst);

```
- **EN**: Implements logic around `getIn`, `getTypeConverter`, `notifyMatchFailure`, `formatv`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getIn`, `getTypeConverter`, `notifyMatchFailure`, `formatv`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并实现验证、折叠、解析或打印钩子。

### Lines 991-1012
```cpp
    Type resultTy = op.getType();
    Type resultElemTy = getElementTypeOrSelf(resultTy);
    Value lowFp = arith::UIToFPOp::create(rewriter, loc, resultTy, lowInt);
    Value hiFp = arith::UIToFPOp::create(rewriter, loc, resultTy, hiInt);

    int64_t pow2Int = int64_t(1) << newBitWidth;
    TypedAttr pow2Attr =
        rewriter.getFloatAttr(resultElemTy, static_cast<double>(pow2Int));
    if (auto vecTy = dyn_cast<VectorType>(resultTy))
      pow2Attr = SplatElementsAttr::get(vecTy, pow2Attr);

    Value pow2Val =
        arith::ConstantOp::create(rewriter, loc, resultTy, pow2Attr);

    Value hiVal = arith::MulFOp::create(rewriter, loc, hiFp, pow2Val);
    Value result = arith::AddFOp::create(rewriter, loc, lowFp, hiVal);

    rewriter.replaceOpWithNewOp<arith::SelectOp>(op, hiEqZero, lowFp, result);
    return success();
  }
};

```
- **EN**: Implements logic around `getType`, `getElementTypeOrSelf`, `create`, `int64_t`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getType`, `getElementTypeOrSelf`, `create`, `int64_t`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1013-1034
```cpp
//===----------------------------------------------------------------------===//
// ConvertFPToSI
//===----------------------------------------------------------------------===//

struct ConvertFPToSI final : OpConversionPattern<arith::FPToSIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::FPToSIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    // Get the input float type.
    Value inFp = adaptor.getIn();
    Type fpTy = inFp.getType();

    Type intTy = op.getType();

    auto newTy = getTypeConverter()->convertType<VectorType>(intTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {}", intTy));

```
- **EN**: Introduces declarations for `ConvertFPToSI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertFPToSI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1035-1054
```cpp
    // Work on the absolute value and then convert the result to signed integer.
    // Defer absolute value to fptoui. If minSInt < fp < maxSInt, i.e. if the fp
    // is representable in signed i2N, emits the correct result. Else, the
    // result is UB.

    TypedAttr zeroAttr = rewriter.getZeroAttr(fpTy);
    Value zeroCst = arith::ConstantOp::create(rewriter, loc, zeroAttr);
    Value zeroCstInt = createScalarOrSplatConstant(rewriter, loc, intTy, 0);

    // Get the absolute value. One could have used math.absf here, but that
    // introduces an extra dependency.
    Value isNeg = arith::CmpFOp::create(
        rewriter, loc, arith::CmpFPredicate::OLT, inFp, zeroCst);
    Value negInFp = arith::NegFOp::create(rewriter, loc, inFp);

    Value absVal = arith::SelectOp::create(rewriter, loc, isNeg, negInFp, inFp);

    // Defer the absolute value to fptoui.
    Value res = arith::FPToUIOp::create(rewriter, loc, intTy, absVal);

```
- **EN**: Implements logic around `getZeroAttr`, `create`, `createScalarOrSplatConstant`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getZeroAttr`, `create`, `createScalarOrSplatConstant` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1055-1077
```cpp
    // Negate the value if < 0 .
    Value neg = arith::SubIOp::create(rewriter, loc, zeroCstInt, res);

    rewriter.replaceOpWithNewOp<arith::SelectOp>(op, isNeg, neg, res);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertFPToUI
//===----------------------------------------------------------------------===//

struct ConvertFPToUI final : OpConversionPattern<arith::FPToUIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::FPToUIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    // Get the input float type.
    Value inFp = adaptor.getIn();
    Type fpTy = inFp.getType();

```
- **EN**: Introduces declarations for `ConvertFPToUI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertFPToUI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1078-1097
```cpp
    Type intTy = op.getType();
    auto newTy = getTypeConverter()->convertType<VectorType>(intTy);
    if (!newTy)
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported type: {}", intTy));
    unsigned newBitWidth = newTy.getElementTypeBitWidth();

    Type newHalfType = IntegerType::get(inFp.getContext(), newBitWidth);
    if (auto vecType = dyn_cast<VectorType>(fpTy))
      newHalfType = VectorType::get(vecType.getShape(), newHalfType);

    // The resulting integer has the upper part and the lower part. This would
    // be interpreted as 2^N * high + low, where N is the bitwidth. Therefore,
    // to calculate the higher part, we emit resHigh = fptoui(fp/2^N). For the
    // lower part, we emit fptoui(fp - resHigh * 2^N). The special cases of
    // overflows including +-inf, NaNs and negative numbers are UB.

    const llvm::fltSemantics &fSemantics =
        cast<FloatType>(getElementTypeOrSelf(fpTy)).getFloatSemantics();

```
- **EN**: Implements logic around `getType`, `getTypeConverter`, `notifyMatchFailure`, `formatv`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getType`, `getTypeConverter`, `notifyMatchFailure`, `formatv`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1098-1124
```cpp
    auto powBitwidth = llvm::APFloat(fSemantics);
    // If the integer does not fit the floating point number, we set the
    // powBitwidth to inf. This ensures that the upper part is set
    // correctly to 0. The opStatus inexact here only occurs when we have an
    // overflow, since the number is always a power of two.
    if (powBitwidth.convertFromAPInt(APInt(newBitWidth * 2, 1).shl(newBitWidth),
                                     false, llvm::RoundingMode::TowardZero) ==
        llvm::detail::opStatus::opInexact)
      powBitwidth = llvm::APFloat::getInf(fSemantics);

    TypedAttr powBitwidthAttr =
        FloatAttr::get(getElementTypeOrSelf(fpTy), powBitwidth);
    if (auto vecType = dyn_cast<VectorType>(fpTy))
      powBitwidthAttr = SplatElementsAttr::get(vecType, powBitwidthAttr);
    Value powBitwidthFloatCst =
        arith::ConstantOp::create(rewriter, loc, powBitwidthAttr);

    Value fpDivPowBitwidth =
        arith::DivFOp::create(rewriter, loc, inFp, powBitwidthFloatCst);
    Value resHigh =
        arith::FPToUIOp::create(rewriter, loc, newHalfType, fpDivPowBitwidth);
    // Calculate fp - resHigh * 2^N by getting the remainder of the division
    Value remainder =
        arith::RemFOp::create(rewriter, loc, inFp, powBitwidthFloatCst);
    Value resLow =
        arith::FPToUIOp::create(rewriter, loc, newHalfType, remainder);

```
- **EN**: Implements logic around `APFloat`, `convertFromAPInt`, `getInf`, `get`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `APFloat`, `convertFromAPInt`, `getInf`, `get`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 1125-1152
```cpp
    Value high = appendX1Dim(rewriter, loc, resHigh);
    Value low = appendX1Dim(rewriter, loc, resLow);

    Value resultVec = constructResultVector(rewriter, loc, newTy, {low, high});

    rewriter.replaceOp(op, resultVec);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertTruncI
//===----------------------------------------------------------------------===//

struct ConvertTruncI final : OpConversionPattern<arith::TruncIOp> {
  using Base::Base;

  LogicalResult
  matchAndRewrite(arith::TruncIOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    // Check if the result type is legal for this target. Currently, we do not
    // support truncation to types wider than supported by the target.
    if (!getTypeConverter()->isLegal(op.getType()))
      return rewriter.notifyMatchFailure(
          loc, llvm::formatv("unsupported truncation result type: {0}",
                             op.getType()));

```
- **EN**: Introduces declarations for `ConvertTruncI`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertTruncI` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1153-1170
```cpp
    // Discard the high half of the input. Truncate the low half, if
    // necessary.
    Value extracted = extractLastDimSlice(rewriter, loc, adaptor.getIn(), 0);
    extracted = dropTrailingX1Dim(rewriter, loc, extracted);
    Value truncated =
        rewriter.createOrFold<arith::TruncIOp>(loc, op.getType(), extracted);
    rewriter.replaceOp(op, truncated);
    return success();
  }
};

//===----------------------------------------------------------------------===//
// ConvertVectorPrint
//===----------------------------------------------------------------------===//

struct ConvertVectorPrint final : OpConversionPattern<vector::PrintOp> {
  using Base::Base;

```
- **EN**: Introduces declarations for `ConvertVectorPrint`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ConvertVectorPrint` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1171-1192
```cpp
  LogicalResult
  matchAndRewrite(vector::PrintOp op, OpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    rewriter.replaceOpWithNewOp<vector::PrintOp>(op, adaptor.getSource());
    return success();
  }
};

//===----------------------------------------------------------------------===//
// Pass Definition
//===----------------------------------------------------------------------===//

struct EmulateWideIntPass final
    : arith::impl::ArithEmulateWideIntBase<EmulateWideIntPass> {
  using ArithEmulateWideIntBase::ArithEmulateWideIntBase;

  void runOnOperation() override {
    if (!llvm::isPowerOf2_32(widestIntSupported) || widestIntSupported < 2) {
      signalPassFailure();
      return;
    }

```
- **EN**: Introduces declarations for `EmulateWideIntPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `EmulateWideIntPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1193-1211
```cpp
    Operation *op = getOperation();
    MLIRContext *ctx = op->getContext();

    arith::WideIntEmulationConverter typeConverter(widestIntSupported);
    ConversionTarget target(*ctx);
    target.addDynamicallyLegalOp<func::FuncOp>([&typeConverter](Operation *op) {
      return typeConverter.isLegal(cast<func::FuncOp>(op).getFunctionType());
    });
    auto opLegalCallback = [&typeConverter](Operation *op) {
      return typeConverter.isLegal(op);
    };
    target.addDynamicallyLegalOp<func::CallOp, func::ReturnOp>(opLegalCallback);
    target.addDynamicallyLegalOp<vector::PrintOp>(opLegalCallback);
    target.addDynamicallyLegalDialect<arith::ArithDialect>(opLegalCallback);
    target.addLegalDialect<vector::VectorDialect>();

    RewritePatternSet patterns(ctx);
    arith::populateArithWideIntEmulationPatterns(typeConverter, patterns);

```
- **EN**: Implements logic around `getOperation`, `getContext`, `typeConverter`, `target`, and 8 more symbols; this block implements verifier, folding, parsing, or printing hooks; works with symbol tables or function-like operations.
- **CN**: 围绕 `getOperation`, `getContext`, `typeConverter`, `target`, and 8 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并处理符号表或类函数操作。

### Lines 1212-1234
```cpp
    // Populate `func.*` conversion patterns.
    populateFunctionOpInterfaceTypeConversionPattern<func::FuncOp>(
        patterns, typeConverter);
    populateCallOpTypeConversionPattern(patterns, typeConverter);
    populateReturnOpTypeConversionPattern(patterns, typeConverter);

    if (failed(applyPartialConversion(op, target, std::move(patterns))))
      signalPassFailure();
  }
};
} // end anonymous namespace

//===----------------------------------------------------------------------===//
// Public Interface Definition
//===----------------------------------------------------------------------===//

arith::WideIntEmulationConverter::WideIntEmulationConverter(
    unsigned widestIntSupportedByTarget)
    : maxIntWidth(widestIntSupportedByTarget) {
  assert(llvm::isPowerOf2_32(widestIntSupportedByTarget) &&
         "Only power-of-two integers with are supported");
  assert(widestIntSupportedByTarget >= 2 && "Integer type too narrow");

```
- **EN**: Implements logic around `FuncOp>`, `populateCallOpTypeConversionPattern`, `populateReturnOpTypeConversionPattern`, `failed`, and 4 more symbols; this block defines or attaches interface behavior; works with symbol tables or function-like operations.
- **CN**: 围绕 `FuncOp>`, `populateCallOpTypeConversionPattern`, `populateReturnOpTypeConversionPattern`, `failed`, and 4 more symbols 实现具体逻辑；该代码块定义或附加接口行为，并处理符号表或类函数操作。

### Lines 1235-1256
```cpp
  // Allow unknown types.
  addConversion([](Type ty) -> std::optional<Type> { return ty; });

  // Scalar case.
  addConversion([this](IntegerType ty) -> std::optional<Type> {
    unsigned width = ty.getWidth();
    if (width <= maxIntWidth)
      return ty;

    // i2N --> vector<2xiN>
    if (width == 2 * maxIntWidth)
      return VectorType::get(2, IntegerType::get(ty.getContext(), maxIntWidth));

    return nullptr;
  });

  // Vector case.
  addConversion([this](VectorType ty) -> std::optional<Type> {
    auto intTy = dyn_cast<IntegerType>(ty.getElementType());
    if (!intTy)
      return ty;

```
- **EN**: Implements logic around `addConversion`, `getWidth`, `get`, `dyn_cast`.
- **CN**: 围绕 `addConversion`, `getWidth`, `get`, `dyn_cast` 实现具体逻辑。

### Lines 1257-1279
```cpp
    unsigned width = intTy.getWidth();
    if (width <= maxIntWidth)
      return ty;

    // vector<...xi2N> --> vector<...x2xiN>
    if (width == 2 * maxIntWidth) {
      auto newShape = to_vector(ty.getShape());
      newShape.push_back(2);
      return VectorType::get(newShape,
                             IntegerType::get(ty.getContext(), maxIntWidth));
    }

    return nullptr;
  });

  // Function case.
  addConversion([this](FunctionType ty) -> std::optional<Type> {
    // Convert inputs and results, e.g.:
    //   (i2N, i2N) -> i2N --> (vector<2xiN>, vector<2xiN>) -> vector<2xiN>
    SmallVector<Type> inputs;
    if (failed(convertTypes(ty.getInputs(), inputs)))
      return nullptr;

```
- **EN**: Implements logic around `getWidth`, `to_vector`, `push_back`, `get`, and 2 more symbols.
- **CN**: 围绕 `getWidth`, `to_vector`, `push_back`, `get`, and 2 more symbols 实现具体逻辑。

### Lines 1280-1313
```cpp
    SmallVector<Type> results;
    if (failed(convertTypes(ty.getResults(), results)))
      return nullptr;

    return FunctionType::get(ty.getContext(), inputs, results);
  });
}

void arith::populateArithWideIntEmulationPatterns(
    const WideIntEmulationConverter &typeConverter,
    RewritePatternSet &patterns) {
  // Populate `arith.*` conversion patterns.
  patterns.add<
      // Misc ops.
      ConvertConstant, ConvertCmpI, ConvertSelect, ConvertVectorPrint,
      // Binary ops.
      ConvertAddI, ConvertMulI, ConvertShLI, ConvertShRSI, ConvertShRUI,
      ConvertMaxMin<arith::MaxUIOp, arith::CmpIPredicate::ugt>,
      ConvertMaxMin<arith::MaxSIOp, arith::CmpIPredicate::sgt>,
      ConvertMaxMin<arith::MinUIOp, arith::CmpIPredicate::ult>,
      ConvertMaxMin<arith::MinSIOp, arith::CmpIPredicate::slt>, ConvertSubI,
      // Bitwise binary ops.
      ConvertBitwiseBinary<arith::AndIOp>, ConvertBitwiseBinary<arith::OrIOp>,
      ConvertBitwiseBinary<arith::XOrIOp>,
      // Extension and truncation ops.
      ConvertExtSI, ConvertExtUI, ConvertTruncI,
      // Cast ops.
      ConvertIndexCastIntToIndex<arith::IndexCastOp>,
      ConvertIndexCastIntToIndex<arith::IndexCastUIOp>,
      ConvertIndexCastIndexToInt<arith::IndexCastOp, arith::ExtSIOp>,
      ConvertIndexCastIndexToInt<arith::IndexCastUIOp, arith::ExtUIOp>,
      ConvertSIToFP, ConvertUIToFP, ConvertFPToUI, ConvertFPToSI>(
      typeConverter, patterns.getContext());
}
```
- **EN**: Implements logic around `failed`, `get`, `populateArithWideIntEmulationPatterns`, `ConvertFPToSI>`, and 1 more symbols.
- **CN**: 围绕 `failed`, `get`, `populateArithWideIntEmulationPatterns`, `ConvertFPToSI>`, and 1 more symbols 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Arithmetic semantics / 算术语义**:
  - **EN**: Defines foldable scalar/vector arithmetic operations and constant semantics.
  - **CN**: 定义可折叠的标量/向量算术操作与常量语义。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/Transforms/Passes.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Transforms/WideIntEmulationConverter.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/TypeUtilities.h`, `mlir/Transforms/DialectConversion.h` ... (+5 more)
- **Standard-library headers / 标准库头文件**: `<cassert>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (8), MLIR IR core abstractions / MLIR IR 核心抽象 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), LLVM support-library helpers / LLVM Support 库辅助功能 (2), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
