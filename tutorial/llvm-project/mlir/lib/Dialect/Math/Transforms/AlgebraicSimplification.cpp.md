# AlgebraicSimplification.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Math/Transforms/AlgebraicSimplification.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements rewrites based on the basic rules of algebra (Commutativity, associativity, etc...) and strength reductions for math operations.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Math/Transforms`，围绕 Math 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AlgebraicSimplification.cpp - Simplify algebraic expressions -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp
//
// This file implements rewrites based on the basic rules of algebra
// (Commutativity, associativity, etc...) and strength reductions for math
// operations.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 15-24
```cpp
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Math/IR/Math.h"
#include "mlir/Dialect/Math/Transforms/Passes.h"
#include "mlir/Dialect/Vector/IR/VectorOps.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/TypeUtilities.h"
#include <climits>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`。

### Lines 25-31
```cpp
using namespace mlir;

//----------------------------------------------------------------------------//
// PowFOp strength reduction.
//----------------------------------------------------------------------------//

namespace {
```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 32-40
```cpp
struct PowFStrengthReduction : public OpRewritePattern<math::PowFOp> {
public:
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(math::PowFOp op,
                                PatternRewriter &rewriter) const final;
};
} // namespace

```
- **EN**: Introduces declarations for `PowFStrengthReduction`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PowFStrengthReduction` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 41-47
```cpp
LogicalResult
PowFStrengthReduction::matchAndRewrite(math::PowFOp op,
                                       PatternRewriter &rewriter) const {
  Location loc = op.getLoc();
  Value x = op.getLhs();
  arith::FastMathFlags fmf = op.getFastmathAttr().getValue();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getLhs`, `getFastmathAttr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getLhs`, `getFastmathAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 48-58
```cpp
  FloatAttr scalarExponent;
  DenseFPElementsAttr vectorExponent;

  bool isScalar = matchPattern(op.getRhs(), m_Constant(&scalarExponent));
  bool isVector = matchPattern(op.getRhs(), m_Constant(&vectorExponent));

  // Returns true if exponent is a constant equal to `value`.
  auto isExponentValue = [&](double value) -> bool {
    if (isScalar)
      return scalarExponent.getValue().isExactlyValue(value);

```
- **EN**: Implements logic around `matchPattern`, `getValue`.
- **CN**: 围绕 `matchPattern`, `getValue` 实现具体逻辑。

### Lines 59-66
```cpp
    if (isVector && vectorExponent.isSplat())
      return vectorExponent.getSplatValue<FloatAttr>()
          .getValue()
          .isExactlyValue(value);

    return false;
  };

```
- **EN**: Implements logic around `isSplat`, `getSplatValue`, `getValue`, `isExactlyValue`.
- **CN**: 围绕 `isSplat`, `getSplatValue`, `getValue`, `isExactlyValue` 实现具体逻辑。

### Lines 67-73
```cpp
  // Maybe broadcasts scalar value into vector type compatible with `op`.
  auto bcast = [&](Value value) -> Value {
    if (auto vec = dyn_cast<VectorType>(op.getType()))
      return vector::BroadcastOp::create(rewriter, loc, vec, value);
    return value;
  };

```
- **EN**: Implements logic around `getType`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getType`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 74-85
```cpp
  // Replace `pow(x, 1.0)` with `x`.
  if (isExponentValue(1.0)) {
    rewriter.replaceOp(op, x);
    return success();
  }

  // Replace `pow(x, 2.0)` with `x * x`.
  if (isExponentValue(2.0)) {
    rewriter.replaceOpWithNewOp<arith::MulFOp>(op, x, x, fmf);
    return success();
  }

```
- **EN**: Implements logic around `isExponentValue`, `replaceOp`, `success`, `MulFOp>`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isExponentValue`, `replaceOp`, `success`, `MulFOp>` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 86-92
```cpp
  // Replace `pow(x, 3.0)` with `x * x * x`.
  if (isExponentValue(3.0)) {
    Value square = arith::MulFOp::create(rewriter, loc, x, x, fmf);
    rewriter.replaceOpWithNewOp<arith::MulFOp>(op, x, square, fmf);
    return success();
  }

```
- **EN**: Implements logic around `isExponentValue`, `create`, `MulFOp>`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isExponentValue`, `create`, `MulFOp>`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 93-101
```cpp
  // Replace `pow(x, -1.0)` with `1.0 / x`.
  if (isExponentValue(-1.0)) {
    Value one = arith::ConstantOp::create(
        rewriter, loc,
        rewriter.getFloatAttr(getElementTypeOrSelf(op.getType()), 1.0));
    rewriter.replaceOpWithNewOp<arith::DivFOp>(op, bcast(one), x, fmf);
    return success();
  }

```
- **EN**: Implements logic around `isExponentValue`, `create`, `getFloatAttr`, `DivFOp>`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isExponentValue`, `create`, `getFloatAttr`, `DivFOp>`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 102-113
```cpp
  // Replace `pow(x, 0.5)` with `sqrt(x)`.
  if (isExponentValue(0.5)) {
    rewriter.replaceOpWithNewOp<math::SqrtOp>(op, x, fmf);
    return success();
  }

  // Replace `pow(x, -0.5)` with `rsqrt(x)`.
  if (isExponentValue(-0.5)) {
    rewriter.replaceOpWithNewOp<math::RsqrtOp>(op, x, fmf);
    return success();
  }

```
- **EN**: Implements logic around `isExponentValue`, `SqrtOp>`, `success`, `RsqrtOp>`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isExponentValue`, `SqrtOp>`, `success`, `RsqrtOp>` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 114-121
```cpp
  // Replace `pow(x, 0.75)` with `sqrt(sqrt(x)) * sqrt(x)`.
  if (isExponentValue(0.75)) {
    Value powHalf = math::SqrtOp::create(rewriter, loc, x, fmf);
    Value powQuarter = math::SqrtOp::create(rewriter, loc, powHalf, fmf);
    rewriter.replaceOpWithNewOp<arith::MulFOp>(op, powHalf, powQuarter, fmf);
    return success();
  }

```
- **EN**: Implements logic around `isExponentValue`, `create`, `MulFOp>`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `isExponentValue`, `create`, `MulFOp>`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 122-128
```cpp
  return failure();
}

//----------------------------------------------------------------------------//
// FPowIOp/IPowIOp strength reduction.
//----------------------------------------------------------------------------//

```
- **EN**: Implements logic around `failure`.
- **CN**: 围绕 `failure` 实现具体逻辑。

### Lines 129-135
```cpp
namespace {
template <typename PowIOpTy, typename DivOpTy, typename MulOpTy>
struct PowIStrengthReduction : public OpRewritePattern<PowIOpTy> {

  unsigned exponentThreshold;

public:
```
- **EN**: Introduces declarations for `PowIStrengthReduction`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PowIStrengthReduction` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 136-146
```cpp
  PowIStrengthReduction(MLIRContext *context, unsigned exponentThreshold = 3,
                        PatternBenefit benefit = 1,
                        ArrayRef<StringRef> generatedNames = {})
      : OpRewritePattern<PowIOpTy>(context, benefit, generatedNames),
        exponentThreshold(exponentThreshold) {}

  LogicalResult matchAndRewrite(PowIOpTy op,
                                PatternRewriter &rewriter) const final;
};
} // namespace

```
- **EN**: Implements logic around `PowIStrengthReduction`, `OpRewritePattern`, `exponentThreshold`, `matchAndRewrite`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `PowIStrengthReduction`, `OpRewritePattern`, `exponentThreshold`, `matchAndRewrite` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 147-153
```cpp
template <typename PowIOpTy, typename DivOpTy, typename MulOpTy>
LogicalResult
PowIStrengthReduction<PowIOpTy, DivOpTy, MulOpTy>::matchAndRewrite(
    PowIOpTy op, PatternRewriter &rewriter) const {
  Location loc = op.getLoc();
  Value base = op.getLhs();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getLhs`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getLhs` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 154-167
```cpp
  IntegerAttr scalarExponent;
  DenseIntElementsAttr vectorExponent;

  bool isScalar = matchPattern(op.getRhs(), m_Constant(&scalarExponent));
  bool isVector = matchPattern(op.getRhs(), m_Constant(&vectorExponent));

  // Simplify cases with known exponent value.
  int64_t exponentValue = 0;
  if (isScalar)
    exponentValue = scalarExponent.getInt();
  else if (isVector && vectorExponent.isSplat())
    exponentValue = vectorExponent.getSplatValue<IntegerAttr>().getInt();
  else
    return failure();
```
- **EN**: Implements logic around `matchPattern`, `getInt`, `isSplat`, `getSplatValue`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `matchPattern`, `getInt`, `isSplat`, `getSplatValue`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理。

### Lines 168-176
```cpp

  // Compute abs(exponent) and check the threshold before creating any IR,
  // so that returning failure() here does not violate the pattern API contract.
  bool exponentIsNegative = false;
  if (exponentValue < 0) {
    exponentIsNegative = true;
    exponentValue *= -1;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 177-187
```cpp
  // Bail out if `abs(exponent)` exceeds the threshold (exponent==0 is free).
  if (exponentValue != 0 && exponentValue > exponentThreshold)
    return failure();

  // Maybe broadcasts scalar value into vector type compatible with `op`.
  auto bcast = [&loc, &op, &rewriter](Value value) -> Value {
    if (auto vec = dyn_cast<VectorType>(op.getType()))
      return vector::BroadcastOp::create(rewriter, loc, vec, value);
    return value;
  };

```
- **EN**: Implements logic around `failure`, `getType`, `create`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `failure`, `getType`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 188-201
```cpp
  Value one;
  Type opType = getElementTypeOrSelf(op.getType());
  if constexpr (std::is_same_v<PowIOpTy, math::FPowIOp>) {
    one = arith::ConstantOp::create(rewriter, loc,
                                    rewriter.getFloatAttr(opType, 1.0));
  } else if constexpr (std::is_same_v<PowIOpTy, complex::PowiOp>) {
    auto complexTy = cast<ComplexType>(opType);
    Type elementType = complexTy.getElementType();
    auto realPart = rewriter.getFloatAttr(elementType, 1.0);
    auto imagPart = rewriter.getFloatAttr(elementType, 0.0);
    one = complex::ConstantOp::create(
        rewriter, loc, complexTy, rewriter.getArrayAttr({realPart, imagPart}));
  } else {
    one = arith::ConstantOp::create(rewriter, loc,
```
- **EN**: Implements logic around `getElementTypeOrSelf`, `constexpr`, `create`, `getFloatAttr`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getElementTypeOrSelf`, `constexpr`, `create`, `getFloatAttr`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 202-210
```cpp
                                    rewriter.getIntegerAttr(opType, 1));
  }

  // Replace `[fi]powi(x, 0)` with `1`.
  if (exponentValue == 0) {
    rewriter.replaceOp(op, bcast(one));
    return success();
  }

```
- **EN**: Implements logic around `getIntegerAttr`, `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getIntegerAttr`, `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 211-224
```cpp
  Value result = base;
  // Transform to naive sequence of multiplications:
  //   * For positive exponent case replace:
  //       `[fi]powi(x, positive_exponent)`
  //     with:
  //       x * x * x * ...
  //   * For negative exponent case replace:
  //       `[fi]powi(x, negative_exponent)`
  //     with:
  //       (1 / x) * (1 / x) * (1 / x) * ...
  auto buildMul = [&](Value lhs, Value rhs) {
    if constexpr (std::is_same_v<PowIOpTy, complex::PowiOp>)
      return MulOpTy::create(rewriter, loc, op.getType(), lhs, rhs,
                             op.getFastmathAttr());
```
- **EN**: Implements logic around `constexpr`, `create`, `getFastmathAttr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `constexpr`, `create`, `getFastmathAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 225-238
```cpp
    else
      return MulOpTy::create(rewriter, loc, lhs, rhs);
  };
  for (unsigned i = 1; i < exponentValue; ++i)
    result = buildMul(result, base);

  // Inverse the base for negative exponent, i.e. for
  // `[fi]powi(x, negative_exponent)` set `x` to `1 / x`.
  if (exponentIsNegative) {
    if constexpr (std::is_same_v<PowIOpTy, complex::PowiOp>)
      result = DivOpTy::create(rewriter, loc, op.getType(), bcast(one), result,
                               op.getFastmathAttr());
    else
      result = DivOpTy::create(rewriter, loc, bcast(one), result);
```
- **EN**: Implements logic around `create`, `buildMul`, `constexpr`, `getFastmathAttr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `buildMul`, `constexpr`, `getFastmathAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 239-246
```cpp
  }

  rewriter.replaceOp(op, result);
  return success();
}

//----------------------------------------------------------------------------//

```
- **EN**: Implements logic around `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 247-255
```cpp
void mlir::populateMathAlgebraicSimplificationPatterns(
    RewritePatternSet &patterns) {
  patterns.add<
      PowFStrengthReduction,
      PowIStrengthReduction<math::IPowIOp, arith::DivSIOp, arith::MulIOp>,
      PowIStrengthReduction<math::FPowIOp, arith::DivFOp, arith::MulFOp>,
      PowIStrengthReduction<complex::PowiOp, complex::DivOp, complex::MulOp>>(
      patterns.getContext(), /*exponentThreshold=*/8);
}
```
- **EN**: Implements logic around `populateMathAlgebraicSimplificationPatterns`, `MulOp>>`, `getContext`.
- **CN**: 围绕 `populateMathAlgebraicSimplificationPatterns`, `MulOp>>`, `getContext` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Complex/IR/Complex.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`, `mlir/Dialect/Vector/IR/VectorOps.h`, `mlir/IR/Builders.h`, `mlir/IR/Matchers.h`, `mlir/IR/TypeUtilities.h`
- **Standard-library headers / 标准库头文件**: `<climits>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3)
