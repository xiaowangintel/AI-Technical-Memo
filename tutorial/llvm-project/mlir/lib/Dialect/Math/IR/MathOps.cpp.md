# MathOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Math/IR/MathOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Math dialect support for dialect IR definitions such as operations, attributes, types, and parsers/printers, centered on `MathOps`.
  - **CN**: 实现 Math 方言中围绕 `MathOps` 的方言 IR 定义，如操作、属性、类型以及解析/打印逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- MathOps.cpp - MLIR operations for math implementation --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/CommonFolders.h"
#include "mlir/Dialect/Math/IR/Math.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/IR/Builders.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/CommonFolders.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/UB/IR/UBOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/CommonFolders.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/UB/IR/UBOps.h`。

### Lines 16-32
```cpp
using namespace mlir;
using namespace mlir::math;

//===----------------------------------------------------------------------===//
// Common helpers
//===----------------------------------------------------------------------===//

/// Return the type of the same shape (scalar, vector or tensor) containing i1.
static Type getI1SameShape(Type type) {
  auto i1Type = IntegerType::get(type.getContext(), 1);
  if (auto shapedType = llvm::dyn_cast<ShapedType>(type))
    return shapedType.cloneWith(std::nullopt, i1Type);
  if (llvm::isa<UnrankedTensorType>(type))
    return UnrankedTensorType::get(i1Type);
  return i1Type;
}

```
- **EN**: Introduces declarations for `mlir`, `mlir::math`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `mlir::math` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 33-48
```cpp
//===----------------------------------------------------------------------===//
// TableGen'd op method definitions
//===----------------------------------------------------------------------===//

#define GET_OP_CLASSES
#include "mlir/Dialect/Math/IR/MathOps.cpp.inc"

//===----------------------------------------------------------------------===//
// AbsFOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AbsFOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<FloatAttr>(adaptor.getOperands(),
                                     [](const APFloat &a) { return abs(a); });
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Math/IR/MathOps.cpp.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Math/IR/MathOps.cpp.inc`。

### Lines 49-75
```cpp
//===----------------------------------------------------------------------===//
// AbsIOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AbsIOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<IntegerAttr>(adaptor.getOperands(),
                                       [](const APInt &a) { return a.abs(); });
}

//===----------------------------------------------------------------------===//
// AcosOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AcosOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(acos(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(acosf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `abs`, `constFoldUnaryOpConditional`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `abs`, `constFoldUnaryOpConditional`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 76-93
```cpp
//===----------------------------------------------------------------------===//
// AcoshOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AcoshOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(acosh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(acoshf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 94-111
```cpp
//===----------------------------------------------------------------------===//
// AsinOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AsinOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(asin(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(asinf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 112-129
```cpp
//===----------------------------------------------------------------------===//
// AsinhOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AsinhOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(asinh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(asinhf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 130-147
```cpp
//===----------------------------------------------------------------------===//
// AtanOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AtanOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(atan(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(atanf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 148-165
```cpp
//===----------------------------------------------------------------------===//
// AtanhOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::AtanhOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(atanh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(atanhf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 166-187
```cpp
//===----------------------------------------------------------------------===//
// Atan2Op folder
//===----------------------------------------------------------------------===//

OpFoldResult math::Atan2Op::fold(FoldAdaptor adaptor) {
  return constFoldBinaryOpConditional<FloatAttr>(
      adaptor.getOperands(),
      [](const APFloat &a, const APFloat &b) -> std::optional<APFloat> {
        if (a.isZero() && b.isZero())
          return llvm::APFloat::getNaN(a.getSemantics());

        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(atan2(a.convertToDouble(), b.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(atan2f(a.convertToFloat(), b.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldBinaryOpConditional`, `getOperands`, `isZero`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldBinaryOpConditional`, `getOperands`, `isZero`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 188-205
```cpp
//===----------------------------------------------------------------------===//
// CbrtOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CbrtOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(cbrt(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(cbrtf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 206-219
```cpp
//===----------------------------------------------------------------------===//
// CeilOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CeilOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) {
        APFloat result(a);
        result.roundToIntegral(llvm::RoundingMode::TowardPositive);
        return result;
      });
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 220-234
```cpp
// CopySignOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CopySignOp::fold(FoldAdaptor adaptor) {
  return constFoldBinaryOp<FloatAttr>(adaptor.getOperands(),
                                      [](const APFloat &a, const APFloat &b) {
                                        APFloat result(a);
                                        result.copySign(b);
                                        return result;
                                      });
}

//===----------------------------------------------------------------------===//
// CosOp folder
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `fold`, `constFoldBinaryOp`, `result`, `copySign`; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldBinaryOp`, `result`, `copySign` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 235-249
```cpp

OpFoldResult math::CosOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(cos(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(cosf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 250-267
```cpp
//===----------------------------------------------------------------------===//
// CoshOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CoshOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(cosh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(coshf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 268-285
```cpp
//===----------------------------------------------------------------------===//
// SinOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::SinOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(sin(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(sinf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 286-303
```cpp
//===----------------------------------------------------------------------===//
// SinhOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::SinhOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(sinh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(sinhf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 304-327
```cpp
//===----------------------------------------------------------------------===//
// SinCosOp
//===----------------------------------------------------------------------===//

std::optional<SmallVector<int64_t, 4>> math::SincosOp::getShapeForUnroll() {
  if (auto vt = mlir::dyn_cast<VectorType>(getOperand().getType()))
    return llvm::to_vector<4>(vt.getShape());
  return std::nullopt;
}

LogicalResult math::SincosOp::fold(FoldAdaptor adaptor,
                                   SmallVectorImpl<OpFoldResult> &result) {
  auto foldSincos = [](const APFloat &a, double (*fnDouble)(double),
                       float (*fnFloat)(float)) -> std::optional<APFloat> {
    switch (APFloat::SemanticsToEnum(a.getSemantics())) {
    case APFloat::Semantics::S_IEEEdouble:
      return APFloat(fnDouble(a.convertToDouble()));
    case APFloat::Semantics::S_IEEEsingle:
      return APFloat(fnFloat(a.convertToFloat()));
    default:
      return {};
    }
  };

```
- **EN**: Implements logic around `getShapeForUnroll`, `getOperand`, `to_vector`, `fold`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `getShapeForUnroll`, `getOperand`, `to_vector`, `fold`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 328-342
```cpp
  Attribute sinRes = constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(),
      [&](const APFloat &a) { return foldSincos(a, sin, sinf); });
  Attribute cosRes = constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(),
      [&](const APFloat &a) { return foldSincos(a, cos, cosf); });

  if (sinRes && cosRes) {
    result.push_back(sinRes);
    result.push_back(cosRes);
    return success();
  }
  return failure();
}

```
- **EN**: Implements logic around `constFoldUnaryOpConditional`, `getOperands`, `foldSincos`, `push_back`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `constFoldUnaryOpConditional`, `getOperands`, `foldSincos`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 343-356
```cpp
//===----------------------------------------------------------------------===//
// CountLeadingZerosOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CountLeadingZerosOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a) { return APInt(a.getBitWidth(), a.countl_zero()); });
}

//===----------------------------------------------------------------------===//
// CountTrailingZerosOp folder
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `getOperands`, `APInt`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `getOperands`, `APInt` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 357-372
```cpp
OpFoldResult math::CountTrailingZerosOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a) { return APInt(a.getBitWidth(), a.countr_zero()); });
}

//===----------------------------------------------------------------------===//
// CtPopOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::CtPopOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a) { return APInt(a.getBitWidth(), a.popcount()); });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `getOperands`, `APInt`; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `getOperands`, `APInt` 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 373-390
```cpp
//===----------------------------------------------------------------------===//
// ErfOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::ErfOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(erf(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(erff(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 391-408
```cpp
//===----------------------------------------------------------------------===//
// ErfcOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::ErfcOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(erfc(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(erfcf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 409-424
```cpp
//===----------------------------------------------------------------------===//
// IPowIOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::IPowIOp::fold(FoldAdaptor adaptor) {
  return constFoldBinaryOpConditional<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &base, const APInt &power) -> std::optional<APInt> {
        unsigned width = base.getBitWidth();
        auto zeroValue = APInt::getZero(width);
        // i1 folding is ambiguous with signed semantics, don't fold.
        if (width == 1)
          return {};
        APInt oneValue{width, 1ULL, /*isSigned=*/true};
        APInt minusOneValue{width, -1ULL, /*isSigned=*/true};

```
- **EN**: Implements logic around `fold`, `constFoldBinaryOpConditional`, `getOperands`, `getBitWidth`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldBinaryOpConditional`, `getOperands`, `getBitWidth`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 425-442
```cpp
        if (power.isZero())
          return oneValue;

        if (power.isNegative()) {
          // Leave 0 raised to negative power not folded.
          if (base.isZero())
            return {};
          if (base.isOne())
            return oneValue;
          // If abs(base) > 1, then the result is zero.
          if (base.ne(minusOneValue))
            return zeroValue;
          // base == -1:
          //   -1: power is odd
          //    1: power is even
          if (power[0] == 1)
            return minusOneValue;

```
- **EN**: Implements logic around `isZero`, `isNegative`, `isOne`, `ne`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isZero`, `isNegative`, `isOne`, `ne` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 443-459
```cpp
          return oneValue;
        }

        // power is positive.
        APInt result = oneValue;
        APInt curBase = base;
        APInt curPower = power;
        while (true) {
          if (curPower[0] == 1)
            result *= curBase;
          curPower.lshrInPlace(1);
          if (curPower.isZero())
            return result;
          curBase *= curBase;
        }
      });

```
- **EN**: Implements logic around `lshrInPlace`, `isZero`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `lshrInPlace`, `isZero` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 460-483
```cpp
  return Attribute();
}

//===----------------------------------------------------------------------===//
// LogOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::LogOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        if (a.isNegative())
          return {};

        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(log(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(logf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `Attribute`, `fold`, `constFoldUnaryOpConditional`, `getOperands`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `Attribute`, `fold`, `constFoldUnaryOpConditional`, `getOperands`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 484-504
```cpp
//===----------------------------------------------------------------------===//
// Log2Op folder
//===----------------------------------------------------------------------===//

OpFoldResult math::Log2Op::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        if (a.isNegative())
          return {};

        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(log2(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(log2f(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 505-525
```cpp
//===----------------------------------------------------------------------===//
// Log10Op folder
//===----------------------------------------------------------------------===//

OpFoldResult math::Log10Op::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        if (a.isNegative())
          return {};

        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(log10(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(log10f(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 526-547
```cpp
//===----------------------------------------------------------------------===//
// Log1pOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::Log1pOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          if ((a + APFloat(1.0)).isNegative())
            return {};
          return APFloat(log1p(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          if ((a + APFloat(1.0f)).isNegative())
            return {};
          return APFloat(log1pf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 548-566
```cpp
//===----------------------------------------------------------------------===//
// PowFOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::PowFOp::fold(FoldAdaptor adaptor) {
  return constFoldBinaryOpConditional<FloatAttr>(
      adaptor.getOperands(),
      [](const APFloat &a, const APFloat &b) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(pow(a.convertToDouble(), b.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(powf(a.convertToFloat(), b.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldBinaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldBinaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 567-588
```cpp
//===----------------------------------------------------------------------===//
// RsqrtOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::RsqrtOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        if (a.isNegative())
          return {};

        APFloat one(a.getSemantics(), 1);
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return one / APFloat(sqrt(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return one / APFloat(sqrtf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 589-609
```cpp
//===----------------------------------------------------------------------===//
// SqrtOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::SqrtOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        if (a.isNegative())
          return {};

        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(sqrt(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(sqrtf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `isNegative`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 610-627
```cpp
//===----------------------------------------------------------------------===//
// ExpOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::ExpOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(exp(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(expf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 628-645
```cpp
//===----------------------------------------------------------------------===//
// Exp2Op folder
//===----------------------------------------------------------------------===//

OpFoldResult math::Exp2Op::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(exp2(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(exp2f(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 646-663
```cpp
//===----------------------------------------------------------------------===//
// ExpM1Op folder
//===----------------------------------------------------------------------===//

OpFoldResult math::ExpM1Op::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(expm1(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(expm1f(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 664-679
```cpp
//===----------------------------------------------------------------------===//
// IsFiniteOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::IsFiniteOp::fold(FoldAdaptor adaptor) {
  if (auto val = dyn_cast_or_null<FloatAttr>(adaptor.getOperand())) {
    return BoolAttr::get(val.getContext(), val.getValue().isFinite());
  }
  if (auto splat = dyn_cast_or_null<SplatElementsAttr>(adaptor.getOperand())) {
    return DenseElementsAttr::get(
        cast<ShapedType>(getType()),
        APInt(1, splat.getSplatValue<APFloat>().isFinite()));
  }
  return {};
}

```
- **EN**: Implements logic around `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 680-695
```cpp
//===----------------------------------------------------------------------===//
// IsInfOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::IsInfOp::fold(FoldAdaptor adaptor) {
  if (auto val = dyn_cast_or_null<FloatAttr>(adaptor.getOperand())) {
    return BoolAttr::get(val.getContext(), val.getValue().isInfinity());
  }
  if (auto splat = dyn_cast_or_null<SplatElementsAttr>(adaptor.getOperand())) {
    return DenseElementsAttr::get(
        cast<ShapedType>(getType()),
        APInt(1, splat.getSplatValue<APFloat>().isInfinity()));
  }
  return {};
}

```
- **EN**: Implements logic around `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 696-711
```cpp
//===----------------------------------------------------------------------===//
// IsNaNOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::IsNaNOp::fold(FoldAdaptor adaptor) {
  if (auto val = dyn_cast_or_null<FloatAttr>(adaptor.getOperand())) {
    return BoolAttr::get(val.getContext(), val.getValue().isNaN());
  }
  if (auto splat = dyn_cast_or_null<SplatElementsAttr>(adaptor.getOperand())) {
    return DenseElementsAttr::get(
        cast<ShapedType>(getType()),
        APInt(1, splat.getSplatValue<APFloat>().isNaN()));
  }
  return {};
}

```
- **EN**: Implements logic around `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 712-727
```cpp
//===----------------------------------------------------------------------===//
// IsNormalOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::IsNormalOp::fold(FoldAdaptor adaptor) {
  if (auto val = dyn_cast_or_null<FloatAttr>(adaptor.getOperand())) {
    return BoolAttr::get(val.getContext(), val.getValue().isNormal());
  }
  if (auto splat = dyn_cast_or_null<SplatElementsAttr>(adaptor.getOperand())) {
    return DenseElementsAttr::get(
        cast<ShapedType>(getType()),
        APInt(1, splat.getSplatValue<APFloat>().isNormal()));
  }
  return {};
}

```
- **EN**: Implements logic around `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `dyn_cast_or_null`, `get`, `getType`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 728-745
```cpp
//===----------------------------------------------------------------------===//
// TanOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::TanOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(tan(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(tanf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 746-763
```cpp
//===----------------------------------------------------------------------===//
// TanhOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::TanhOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(tanh(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(tanhf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 764-777
```cpp
//===----------------------------------------------------------------------===//
// RoundEvenOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::RoundEvenOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) {
        APFloat result(a);
        result.roundToIntegral(llvm::RoundingMode::NearestTiesToEven);
        return result;
      });
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 778-792
```cpp
// FloorOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::FloorOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOp<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) {
        APFloat result(a);
        result.roundToIntegral(llvm::RoundingMode::TowardNegative);
        return result;
      });
}

//===----------------------------------------------------------------------===//
// RoundOp folder
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOp`, `getOperands`, `result`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 793-807
```cpp

OpFoldResult math::RoundOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(round(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(roundf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 808-825
```cpp
//===----------------------------------------------------------------------===//
// TruncOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::TruncOp::fold(FoldAdaptor adaptor) {
  return constFoldUnaryOpConditional<FloatAttr>(
      adaptor.getOperands(), [](const APFloat &a) -> std::optional<APFloat> {
        switch (APFloat::SemanticsToEnum(a.getSemantics())) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(trunc(a.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(truncf(a.convertToFloat()));
        default:
          return {};
        }
      });
}

```
- **EN**: Implements logic around `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `constFoldUnaryOpConditional`, `getOperands`, `SemanticsToEnum`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 826-842
```cpp
//===----------------------------------------------------------------------===//
// FPowIOp folder
//===----------------------------------------------------------------------===//

OpFoldResult math::FPowIOp::fold(FoldAdaptor adaptor) {
  return constFoldBinaryOpConditional<FloatAttr, IntegerAttr>(
      adaptor.getOperands(),
      [](const APFloat &base, const APInt &exp) -> std::optional<APFloat> {
        const llvm::fltSemantics &sem = base.getSemantics();
        // Fold when the exponent is exactly representable in the
        // floating-point type of the base.
        APFloat fExp(sem);
        if (fExp.convertFromAPInt(exp, /*isSigned=*/true,
                                  APFloat::rmNearestTiesToEven) !=
            APFloat::opOK)
          return {};

```
- **EN**: Implements logic around `fold`, `IntegerAttr>`, `getOperands`, `getSemantics`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `fold`, `IntegerAttr>`, `getOperands`, `getSemantics`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 843-860
```cpp
        switch (APFloat::SemanticsToEnum(sem)) {
        case APFloat::Semantics::S_IEEEdouble:
          return APFloat(pow(base.convertToDouble(), fExp.convertToDouble()));
        case APFloat::Semantics::S_IEEEsingle:
          return APFloat(powf(base.convertToFloat(), fExp.convertToFloat()));
        default:
          return {};
        }
      });
}

/// Materialize an integer or floating point constant.
Operation *math::MathDialect::materializeConstant(OpBuilder &builder,
                                                  Attribute value, Type type,
                                                  Location loc) {
  if (auto poison = dyn_cast<ub::PoisonAttr>(value))
    return ub::PoisonOp::create(builder, loc, type, poison);

```
- **EN**: Implements logic around `SemanticsToEnum`, `APFloat`, `materializeConstant`, `PoisonAttr>`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `SemanticsToEnum`, `APFloat`, `materializeConstant`, `PoisonAttr>`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 861-862
```cpp
  return arith::ConstantOp::materialize(builder, value, type, loc);
}
```
- **EN**: Implements logic around `materialize`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `materialize` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/CommonFolders.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Builders.h`, `mlir/Dialect/Math/IR/MathOps.cpp.inc`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
- **Generated macros / 生成宏**: `GET_OP_CLASSES`
