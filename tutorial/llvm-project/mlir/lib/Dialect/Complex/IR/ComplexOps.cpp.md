# ComplexOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Complex/IR/ComplexOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the core IR, operations, types, attributes, or interfaces for the Complex dialect and complex-number operations.
  - **CN**: 实现 Complex 方言与复数操作 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- ComplexOps.cpp - MLIR Complex Operations ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`。

### Lines 15-28
```cpp
using namespace mlir;
using namespace mlir::complex;

//===----------------------------------------------------------------------===//
// ConstantOp
//===----------------------------------------------------------------------===//

OpFoldResult ConstantOp::fold(FoldAdaptor adaptor) { return getValue(); }

void ConstantOp::getAsmResultNames(
    function_ref<void(Value, StringRef)> setNameFn) {
  setNameFn(getResult(), "cst");
}

```
- **EN**: Implements logic around `fold`, `getAsmResultNames`, `function_ref`, `setNameFn`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getAsmResultNames`, `function_ref`, `setNameFn` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 29-48
```cpp
bool ConstantOp::isBuildableWith(Attribute value, Type type) {
  if (auto arrAttr = llvm::dyn_cast<ArrayAttr>(value)) {
    auto complexTy = llvm::dyn_cast<ComplexType>(type);
    if (!complexTy || arrAttr.size() != 2)
      return false;
    auto complexEltTy = complexTy.getElementType();
    if (auto fre = llvm::dyn_cast<FloatAttr>(arrAttr[0])) {
      auto im = llvm::dyn_cast<FloatAttr>(arrAttr[1]);
      return im && fre.getType() == complexEltTy &&
             im.getType() == complexEltTy;
    }
    if (auto ire = llvm::dyn_cast<IntegerAttr>(arrAttr[0])) {
      auto im = llvm::dyn_cast<IntegerAttr>(arrAttr[1]);
      return im && ire.getType() == complexEltTy &&
             im.getType() == complexEltTy;
    }
  }
  return false;
}

```
- **EN**: Implements logic around `isBuildableWith`, `dyn_cast`, `size`, `getElementType`, and 1 more symbols.
- **CN**: 围绕 `isBuildableWith`, `dyn_cast`, `size`, `getElementType`, and 1 more symbols 实现具体逻辑。

### Lines 49-68
```cpp
LogicalResult ConstantOp::verify() {
  ArrayAttr arrayAttr = getValue();
  if (arrayAttr.size() != 2) {
    return emitOpError(
        "requires 'value' to be a complex constant, represented as array of "
        "two values");
  }

  auto complexEltTy = getType().getElementType();
  if (!isa<FloatAttr, IntegerAttr>(arrayAttr[0]) ||
      !isa<FloatAttr, IntegerAttr>(arrayAttr[1]))
    return emitOpError(
        "requires attribute's elements to be float or integer attributes");
  auto re = llvm::dyn_cast<TypedAttr>(arrayAttr[0]);
  auto im = llvm::dyn_cast<TypedAttr>(arrayAttr[1]);
  if (complexEltTy != re.getType() || complexEltTy != im.getType()) {
    return emitOpError()
           << "requires attribute's element types (" << re.getType() << ", "
           << im.getType()
           << ") to match the element type of the op's return type ("
```
- **EN**: Implements logic around `verify`, `getValue`, `size`, `emitOpError`, and 5 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getValue`, `size`, `emitOpError`, and 5 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 69-81
```cpp
           << complexEltTy << ")";
  }
  return success();
}

//===----------------------------------------------------------------------===//
// BitcastOp
//===----------------------------------------------------------------------===//

OpFoldResult BitcastOp::fold(FoldAdaptor bitcast) {
  if (getOperand().getType() == getType())
    return getOperand();

```
- **EN**: Implements logic around `success`, `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `success`, `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 82-92
```cpp
  return {};
}

LogicalResult BitcastOp::verify() {
  auto operandType = getOperand().getType();
  auto resultType = getType();

  // We allow this to be legal as it can be folded away.
  if (operandType == resultType)
    return success();

```
- **EN**: Implements logic around `verify`, `getOperand`, `getType`, `success`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `verify`, `getOperand`, `getType`, `success` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 93-105
```cpp
  if (!operandType.isIntOrFloat() && !isa<ComplexType>(operandType)) {
    return emitOpError("operand must be int/float/complex");
  }

  if (!resultType.isIntOrFloat() && !isa<ComplexType>(resultType)) {
    return emitOpError("result must be int/float/complex");
  }

  if (isa<ComplexType>(operandType) == isa<ComplexType>(resultType)) {
    return emitOpError(
        "requires that either input or output has a complex type");
  }

```
- **EN**: Implements logic around `isIntOrFloat`, `emitOpError`, `isa`.
- **CN**: 围绕 `isIntOrFloat`, `emitOpError`, `isa` 实现具体逻辑。

### Lines 106-118
```cpp
  if (isa<ComplexType>(resultType))
    std::swap(operandType, resultType);

  int32_t operandBitwidth = dyn_cast<ComplexType>(operandType)
                                .getElementType()
                                .getIntOrFloatBitWidth() *
                            2;
  int32_t resultBitwidth = resultType.getIntOrFloatBitWidth();

  if (operandBitwidth != resultBitwidth) {
    return emitOpError("casting bitwidths do not match");
  }

```
- **EN**: Implements logic around `isa`, `swap`, `dyn_cast`, `getElementType`, and 2 more symbols.
- **CN**: 围绕 `isa`, `swap`, `dyn_cast`, `getElementType`, and 2 more symbols 实现具体逻辑。

### Lines 119-138
```cpp
  return success();
}

struct MergeComplexBitcast final : OpRewritePattern<BitcastOp> {
  using OpRewritePattern<BitcastOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(BitcastOp op,
                                PatternRewriter &rewriter) const override {
    if (auto defining = op.getOperand().getDefiningOp<BitcastOp>()) {
      if (isa<ComplexType>(op.getType()) ||
          isa<ComplexType>(defining.getOperand().getType())) {
        // complex.bitcast requires that input or output is complex.
        rewriter.replaceOpWithNewOp<BitcastOp>(op, op.getType(),
                                               defining.getOperand());
      } else {
        rewriter.replaceOpWithNewOp<arith::BitcastOp>(op, op.getType(),
                                                      defining.getOperand());
      }
      return success();
    }
```
- **EN**: Introduces declarations for `MergeComplexBitcast`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MergeComplexBitcast` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 139-149
```cpp

    if (auto defining = op.getOperand().getDefiningOp<arith::BitcastOp>()) {
      rewriter.replaceOpWithNewOp<BitcastOp>(op, op.getType(),
                                             defining.getOperand());
      return success();
    }

    return failure();
  }
};

```
- **EN**: Implements logic around `getOperand`, `replaceOpWithNewOp`, `success`, `failure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getOperand`, `replaceOpWithNewOp`, `success`, `failure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 150-160
```cpp
struct MergeArithBitcast final : OpRewritePattern<arith::BitcastOp> {
  using OpRewritePattern<arith::BitcastOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(arith::BitcastOp op,
                                PatternRewriter &rewriter) const override {
    if (auto defining = op.getOperand().getDefiningOp<complex::BitcastOp>()) {
      rewriter.replaceOpWithNewOp<complex::BitcastOp>(op, op.getType(),
                                                      defining.getOperand());
      return success();
    }

```
- **EN**: Introduces declarations for `MergeArithBitcast`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `MergeArithBitcast` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 161-170
```cpp
    return failure();
  }
};

void BitcastOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                            MLIRContext *context) {
  results.add<MergeComplexBitcast, MergeArithBitcast>(context);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `failure`, `getCanonicalizationPatterns`, `MergeArithBitcast>`; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `failure`, `getCanonicalizationPatterns`, `MergeArithBitcast>` 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 171-185
```cpp
// CreateOp
//===----------------------------------------------------------------------===//

OpFoldResult CreateOp::fold(FoldAdaptor adaptor) {
  // Fold complex.create(complex.re(op), complex.im(op)).
  if (auto reOp = getOperand(0).getDefiningOp<ReOp>()) {
    if (auto imOp = getOperand(1).getDefiningOp<ImOp>()) {
      if (reOp.getOperand() == imOp.getOperand()) {
        return reOp.getOperand();
      }
    }
  }
  return {};
}

```
- **EN**: Implements logic around `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 186-199
```cpp
//===----------------------------------------------------------------------===//
// ImOp
//===----------------------------------------------------------------------===//

OpFoldResult ImOp::fold(FoldAdaptor adaptor) {
  ArrayAttr arrayAttr =
      llvm::dyn_cast_if_present<ArrayAttr>(adaptor.getComplex());
  if (arrayAttr && arrayAttr.size() == 2)
    return arrayAttr[1];
  if (auto createOp = getOperand().getDefiningOp<CreateOp>())
    return createOp.getOperand(1);
  return {};
}

```
- **EN**: Implements logic around `fold`, `dyn_cast_if_present`, `size`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `dyn_cast_if_present`, `size`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 200-210
```cpp
namespace {
template <typename OpKind, int ComponentIndex>
struct FoldComponentNeg final : OpRewritePattern<OpKind> {
  using OpRewritePattern<OpKind>::OpRewritePattern;

  LogicalResult matchAndRewrite(OpKind op,
                                PatternRewriter &rewriter) const override {
    auto negOp = op.getOperand().template getDefiningOp<NegOp>();
    if (!negOp)
      return failure();

```
- **EN**: Introduces declarations for `FoldComponentNeg`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `FoldComponentNeg` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 211-224
```cpp
    auto createOp = negOp.getComplex().template getDefiningOp<CreateOp>();
    if (!createOp)
      return failure();

    Type elementType = createOp.getType().getElementType();
    assert(isa<FloatType>(elementType));

    rewriter.replaceOpWithNewOp<arith::NegFOp>(
        op, elementType, createOp.getOperand(ComponentIndex));
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `getComplex`, `failure`, `getType`, `assert`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getComplex`, `failure`, `getType`, `assert`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 225-243
```cpp
void ImOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                       MLIRContext *context) {
  results.add<FoldComponentNeg<ImOp, 1>>(context);
}

//===----------------------------------------------------------------------===//
// ReOp
//===----------------------------------------------------------------------===//

OpFoldResult ReOp::fold(FoldAdaptor adaptor) {
  ArrayAttr arrayAttr =
      llvm::dyn_cast_if_present<ArrayAttr>(adaptor.getComplex());
  if (arrayAttr && arrayAttr.size() == 2)
    return arrayAttr[0];
  if (auto createOp = getOperand().getDefiningOp<CreateOp>())
    return createOp.getOperand(0);
  return {};
}

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `fold`, `dyn_cast_if_present`, `size`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `getCanonicalizationPatterns`, `fold`, `dyn_cast_if_present`, `size`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子。

### Lines 244-258
```cpp
void ReOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                       MLIRContext *context) {
  results.add<FoldComponentNeg<ReOp, 0>>(context);
}

//===----------------------------------------------------------------------===//
// AddOp
//===----------------------------------------------------------------------===//

OpFoldResult AddOp::fold(FoldAdaptor adaptor) {
  // complex.add(complex.sub(a, b), b) -> a
  if (auto sub = getLhs().getDefiningOp<SubOp>())
    if (getRhs() == sub.getRhs())
      return sub.getLhs();

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `fold`, `getLhs`, `getRhs`; this block implements verifier, folding, parsing, or printing hooks; registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `getCanonicalizationPatterns`, `fold`, `getLhs`, `getRhs` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并注册方言实体或方言级钩子。

### Lines 259-272
```cpp
  // complex.add(b, complex.sub(a, b)) -> a
  if (auto sub = getRhs().getDefiningOp<SubOp>())
    if (getLhs() == sub.getRhs())
      return sub.getLhs();

  // complex.add(a, complex.constant<0.0, 0.0>) -> a
  if (auto constantOp = getRhs().getDefiningOp<ConstantOp>()) {
    auto arrayAttr = constantOp.getValue();
    if (llvm::cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&
        llvm::cast<FloatAttr>(arrayAttr[1]).getValue().isZero()) {
      return getLhs();
    }
  }

```
- **EN**: Implements logic around `getRhs`, `getLhs`, `getValue`, `cast`.
- **CN**: 围绕 `getRhs`, `getLhs`, `getValue`, `cast` 实现具体逻辑。

### Lines 273-285
```cpp
  return {};
}

//===----------------------------------------------------------------------===//
// SubOp
//===----------------------------------------------------------------------===//

OpFoldResult SubOp::fold(FoldAdaptor adaptor) {
  // complex.sub(complex.add(a, b), b) -> a
  if (auto add = getLhs().getDefiningOp<AddOp>())
    if (getRhs() == add.getRhs())
      return add.getLhs();

```
- **EN**: Implements logic around `fold`, `getLhs`, `getRhs`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getLhs`, `getRhs` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 286-297
```cpp
  // complex.sub(a, complex.constant<0.0, 0.0>) -> a
  if (auto constantOp = getRhs().getDefiningOp<ConstantOp>()) {
    auto arrayAttr = constantOp.getValue();
    if (llvm::cast<FloatAttr>(arrayAttr[0]).getValue().isZero() &&
        llvm::cast<FloatAttr>(arrayAttr[1]).getValue().isZero()) {
      return getLhs();
    }
  }

  return {};
}

```
- **EN**: Implements logic around `getRhs`, `getValue`, `cast`, `getLhs`.
- **CN**: 围绕 `getRhs`, `getValue`, `cast`, `getLhs` 实现具体逻辑。

### Lines 298-309
```cpp
//===----------------------------------------------------------------------===//
// NegOp
//===----------------------------------------------------------------------===//

OpFoldResult NegOp::fold(FoldAdaptor adaptor) {
  // complex.neg(complex.neg(a)) -> a
  if (auto negOp = getOperand().getDefiningOp<NegOp>())
    return negOp.getOperand();

  return {};
}

```
- **EN**: Implements logic around `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 310-321
```cpp
//===----------------------------------------------------------------------===//
// LogOp
//===----------------------------------------------------------------------===//

OpFoldResult LogOp::fold(FoldAdaptor adaptor) {
  // complex.log(complex.exp(a)) -> a
  if (auto expOp = getOperand().getDefiningOp<ExpOp>())
    return expOp.getOperand();

  return {};
}

```
- **EN**: Implements logic around `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 322-333
```cpp
//===----------------------------------------------------------------------===//
// ExpOp
//===----------------------------------------------------------------------===//

OpFoldResult ExpOp::fold(FoldAdaptor adaptor) {
  // complex.exp(complex.log(a)) -> a
  if (auto logOp = getOperand().getDefiningOp<LogOp>())
    return logOp.getOperand();

  return {};
}

```
- **EN**: Implements logic around `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 334-345
```cpp
//===----------------------------------------------------------------------===//
// ConjOp
//===----------------------------------------------------------------------===//

OpFoldResult ConjOp::fold(FoldAdaptor adaptor) {
  // complex.conj(complex.conj(a)) -> a
  if (auto conjOp = getOperand().getDefiningOp<ConjOp>())
    return conjOp.getOperand();

  return {};
}

```
- **EN**: Implements logic around `fold`, `getOperand`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getOperand` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 346-358
```cpp
//===----------------------------------------------------------------------===//
// MulOp
//===----------------------------------------------------------------------===//

OpFoldResult MulOp::fold(FoldAdaptor adaptor) {
  auto constant = getRhs().getDefiningOp<ConstantOp>();
  if (!constant)
    return {};

  ArrayAttr arrayAttr = constant.getValue();
  APFloat real = cast<FloatAttr>(arrayAttr[0]).getValue();
  APFloat imag = cast<FloatAttr>(arrayAttr[1]).getValue();

```
- **EN**: Implements logic around `fold`, `getRhs`, `getValue`, `cast`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getRhs`, `getValue`, `cast` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 359-368
```cpp
  if (!imag.isZero())
    return {};

  // complex.mul(a, complex.constant<1.0, 0.0>) -> a
  if (real == APFloat(real.getSemantics(), 1))
    return getLhs();

  return {};
}

```
- **EN**: Implements logic around `isZero`, `APFloat`, `getLhs`.
- **CN**: 围绕 `isZero`, `APFloat`, `getLhs` 实现具体逻辑。

### Lines 369-388
```cpp
//===----------------------------------------------------------------------===//
// DivOp
//===----------------------------------------------------------------------===//

OpFoldResult DivOp::fold(FoldAdaptor adaptor) {
  Attribute rhs = adaptor.getRhs();
  Attribute lhs = adaptor.getLhs();

  // complex.div(complex.constant<NaN, NaN>, a) -> complex.constant<NaN, NaN>
  // complex.div(complex.constant<NaN, a>, b) -> complex.constant<NaN, NaN>
  // complex.div(complex.constant<a, NaN>, b) -> complex.constant<NaN, NaN>
  bool isLhsComplexHasNan = false;
  ArrayAttr lhsArrayAttr = dyn_cast_if_present<ArrayAttr>(lhs);
  if (lhsArrayAttr && lhsArrayAttr.size() == 2) {
    APFloat lhsReal = cast<FloatAttr>(lhsArrayAttr[0]).getValue();
    APFloat lhsImag = cast<FloatAttr>(lhsArrayAttr[1]).getValue();
    isLhsComplexHasNan = lhsReal.isNaN() || lhsImag.isNaN();
    if (isLhsComplexHasNan) {
      Attribute nanValue = lhsReal.isNaN() ? lhsArrayAttr[0] : lhsArrayAttr[1];
      return ArrayAttr::get(getContext(), {nanValue, nanValue});
```
- **EN**: Implements logic around `fold`, `getRhs`, `getLhs`, `dyn_cast_if_present`, and 4 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `fold`, `getRhs`, `getLhs`, `dyn_cast_if_present`, and 4 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 389-401
```cpp
    }
  }

  ArrayAttr rhsArrayAttr = dyn_cast_if_present<ArrayAttr>(rhs);
  if (!rhsArrayAttr || rhsArrayAttr.size() != 2)
    return {};

  // Fold only if RHS is complex.constant<1.0, 0.0>
  APFloat rhsImag = cast<FloatAttr>(rhsArrayAttr[1]).getValue();
  APFloat rhsReal = cast<FloatAttr>(rhsArrayAttr[0]).getValue();
  if (!rhsImag.isZero() || rhsReal != APFloat(rhsReal.getSemantics(), 1))
    return {};

```
- **EN**: Implements logic around `dyn_cast_if_present`, `size`, `cast`, `isZero`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `dyn_cast_if_present`, `size`, `cast`, `isZero` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 402-411
```cpp
  // Fold to LHS if it doesn't contains NaNs or fast math flag nan is set
  // complex.div(a, complex.constant<1.0, 0.0>) fastmath<nnan> -> a
  if ((lhsArrayAttr && !isLhsComplexHasNan) ||
      arith::bitEnumContainsAll(getFastmath(), arith::FastMathFlags::nnan))
    return getLhs();

  return {};
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `bitEnumContainsAll`, `getLhs`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `bitEnumContainsAll`, `getLhs` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 412-416
```cpp
// TableGen'd op method definitions
//===----------------------------------------------------------------------===//

#define GET_OP_CLASSES
#include "mlir/Dialect/Complex/IR/ComplexOps.cpp.inc"
```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/PatternMatch.h`, `mlir/Dialect/Complex/IR/ComplexOps.cpp.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (3), MLIR IR core abstractions / MLIR IR 核心抽象 (3)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
