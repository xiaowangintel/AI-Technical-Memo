# SPIRVCanonicalization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/IR/SPIRVCanonicalization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the folders and canonicalization patterns for SPIR-V ops.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SPIRV/IR`，围绕 SPIRV 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```cpp
//===- SPIRVCanonicalization.cpp - MLIR SPIR-V canonicalization patterns --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the folders and canonicalization patterns for SPIR-V ops.
//
//===----------------------------------------------------------------------===//

#include <optional>
#include <utility>

#include "mlir/Dialect/SPIRV/IR/SPIRVOps.h"

#include "mlir/Dialect/CommonFolders.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVTypes.h"
#include "mlir/Dialect/UB/IR/UBOps.h"
#include "mlir/IR/Matchers.h"
#include "mlir/IR/PatternMatch.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallVectorExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `optional`, `utility`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/CommonFolders.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `optional`, `utility`, `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/CommonFolders.h`。

### Lines 26-45
```cpp
using namespace mlir;

//===----------------------------------------------------------------------===//
// Common utility functions
//===----------------------------------------------------------------------===//

/// Returns the boolean value under the hood if the given `boolAttr` is a scalar
/// or splat vector bool constant.
static std::optional<bool> getScalarOrSplatBoolAttr(Attribute attr) {
  if (!attr)
    return std::nullopt;

  if (auto boolAttr = dyn_cast<BoolAttr>(attr))
    return boolAttr.getValue();
  if (auto splatAttr = dyn_cast<SplatElementsAttr>(attr))
    if (splatAttr.getElementType().isInteger(1))
      return splatAttr.getSplatValue<bool>();
  return std::nullopt;
}

```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 46-67
```cpp
// Extracts an element from the given `composite` by following the given
// `indices`. Returns a null Attribute if error happens.
static Attribute extractCompositeElement(Attribute composite,
                                         ArrayRef<unsigned> indices) {
  // Check that given composite is a constant.
  if (!composite)
    return {};
  // Return composite itself if we reach the end of the index chain.
  if (indices.empty())
    return composite;

  if (auto vector = dyn_cast<ElementsAttr>(composite)) {
    assert(indices.size() == 1 && "must have exactly one index for a vector");
    return vector.getValues<Attribute>()[indices[0]];
  }

  if (auto array = dyn_cast<ArrayAttr>(composite)) {
    assert(!indices.empty() && "must have at least one index for an array");
    return extractCompositeElement(array.getValue()[indices[0]],
                                   indices.drop_front());
  }

```
- **EN**: Implements logic around `extractCompositeElement`, `empty`, `assert`, `getValues`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `extractCompositeElement`, `empty`, `assert`, `getValues`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 68-85
```cpp
  return {};
}

static bool isDivZeroOrOverflow(const APInt &a, const APInt &b) {
  bool div0 = b.isZero();
  bool overflow = a.isMinSignedValue() && b.isAllOnes();

  return div0 || overflow;
}

//===----------------------------------------------------------------------===//
// TableGen'erated canonicalizers
//===----------------------------------------------------------------------===//

namespace {
#include "SPIRVCanonicalization.inc"
} // namespace

```
- **EN**: Pulls in the headers needed by this translation unit, including `SPIRVCanonicalization.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `SPIRVCanonicalization.inc`。

### Lines 86-106
```cpp
//===----------------------------------------------------------------------===//
// spirv.AccessChainOp
//===----------------------------------------------------------------------===//

namespace {

/// Combines chained `spirv::AccessChainOp` operations into one
/// `spirv::AccessChainOp` operation.
struct CombineChainedAccessChain final
    : OpRewritePattern<spirv::AccessChainOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(spirv::AccessChainOp accessChainOp,
                                PatternRewriter &rewriter) const override {
    auto parentAccessChainOp =
        accessChainOp.getBasePtr().getDefiningOp<spirv::AccessChainOp>();

    if (!parentAccessChainOp) {
      return failure();
    }

```
- **EN**: Introduces declarations for `CombineChainedAccessChain`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `CombineChainedAccessChain` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 107-124
```cpp
    // Combine indices.
    SmallVector<Value, 4> indices(parentAccessChainOp.getIndices());
    llvm::append_range(indices, accessChainOp.getIndices());

    rewriter.replaceOpWithNewOp<spirv::AccessChainOp>(
        accessChainOp, parentAccessChainOp.getBasePtr(), indices);

    return success();
  }
};
} // namespace

void spirv::AccessChainOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results.add<CombineChainedAccessChain>(context);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `indices`, `append_range`, `AccessChainOp>`, `getBasePtr`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `indices`, `append_range`, `AccessChainOp>`, `getBasePtr`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 125-147
```cpp
// spirv.IAddCarry
//===----------------------------------------------------------------------===//

// We are required to use CompositeConstructOp to create a constant struct as
// they are not yet implemented as constant, hence we can not do so in a fold.
struct IAddCarryFold final : OpRewritePattern<spirv::IAddCarryOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(spirv::IAddCarryOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    Value lhs = op.getOperand1();
    Value rhs = op.getOperand2();
    Type constituentType = lhs.getType();

    // iaddcarry (x, 0) = <0, x>
    if (matchPattern(rhs, m_Zero())) {
      Value constituents[2] = {rhs, lhs};
      rewriter.replaceOpWithNewOp<spirv::CompositeConstructOp>(op, op.getType(),
                                                               constituents);
      return success();
    }

```
- **EN**: Introduces declarations for `as`, `IAddCarryFold`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `as`, `IAddCarryFold` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 148-170
```cpp
    // According to the SPIR-V spec:
    //
    //  Result Type must be from OpTypeStruct.  The struct must have two
    //  members...
    //
    //  Member 0 of the result gets the low-order bits (full component width) of
    //  the addition.
    //
    //  Member 1 of the result gets the high-order (carry) bit of the result of
    //  the addition. That is, it gets the value 1 if the addition overflowed
    //  the component width, and 0 otherwise.
    Attribute lhsAttr;
    Attribute rhsAttr;
    if (!matchPattern(lhs, m_Constant(&lhsAttr)) ||
        !matchPattern(rhs, m_Constant(&rhsAttr)))
      return failure();

    auto adds = constFoldBinaryOp<IntegerAttr>(
        {lhsAttr, rhsAttr},
        [](const APInt &a, const APInt &b) { return a + b; });
    if (!adds)
      return failure();

```
- **EN**: Introduces declarations for `must`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `must` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 171-197
```cpp
    auto carrys = constFoldBinaryOp<IntegerAttr>(
        ArrayRef{adds, lhsAttr}, [](const APInt &a, const APInt &b) {
          APInt zero = APInt::getZero(a.getBitWidth());
          return a.ult(b) ? (zero + 1) : zero;
        });

    if (!carrys)
      return failure();

    Value addsVal =
        spirv::ConstantOp::create(rewriter, loc, constituentType, adds);

    Value carrysVal =
        spirv::ConstantOp::create(rewriter, loc, constituentType, carrys);

    // Create empty struct
    Value undef = spirv::UndefOp::create(rewriter, loc, op.getType());
    // Fill in adds at id 0
    Value intermediate =
        spirv::CompositeInsertOp::create(rewriter, loc, addsVal, undef, 0);
    // Fill in carrys at id 1
    rewriter.replaceOpWithNewOp<spirv::CompositeInsertOp>(op, carrysVal,
                                                          intermediate, 1);
    return success();
  }
};

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getZero`, `ult`, `failure`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getZero`, `ult`, `failure`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 198-219
```cpp
void spirv::IAddCarryOp::getCanonicalizationPatterns(
    RewritePatternSet &patterns, MLIRContext *context) {
  patterns.add<IAddCarryFold>(context);
}

//===----------------------------------------------------------------------===//
// spirv.[S|U]MulExtended
//===----------------------------------------------------------------------===//

// We are required to use CompositeConstructOp to create a constant struct as
// they are not yet implemented as constant, hence we can not do so in a fold.
template <typename MulOp, bool IsSigned>
struct MulExtendedFold final : OpRewritePattern<MulOp> {
  using OpRewritePattern<MulOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(MulOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    Value lhs = op.getOperand1();
    Value rhs = op.getOperand2();
    Type constituentType = lhs.getType();

```
- **EN**: Introduces declarations for `as`, `MulExtendedFold`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `as`, `MulExtendedFold` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 220-242
```cpp
    // [su]mulextended (x, 0) = <0, 0>
    if (matchPattern(rhs, m_Zero())) {
      Value zero = spirv::ConstantOp::getZero(constituentType, loc, rewriter);
      Value constituents[2] = {zero, zero};
      rewriter.replaceOpWithNewOp<spirv::CompositeConstructOp>(op, op.getType(),
                                                               constituents);
      return success();
    }

    // According to the SPIR-V spec:
    //
    // Result Type must be from OpTypeStruct.  The struct must have two
    // members...
    //
    // Member 0 of the result gets the low-order bits of the multiplication.
    //
    // Member 1 of the result gets the high-order bits of the multiplication.
    Attribute lhsAttr;
    Attribute rhsAttr;
    if (!matchPattern(lhs, m_Constant(&lhsAttr)) ||
        !matchPattern(rhs, m_Constant(&rhsAttr)))
      return failure();

```
- **EN**: Introduces declarations for `must`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `must` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 243-260
```cpp
    auto lowBits = constFoldBinaryOp<IntegerAttr>(
        {lhsAttr, rhsAttr},
        [](const APInt &a, const APInt &b) { return a * b; });

    if (!lowBits)
      return failure();

    auto highBits = constFoldBinaryOp<IntegerAttr>(
        {lhsAttr, rhsAttr}, [](const APInt &a, const APInt &b) {
          if (IsSigned) {
            return llvm::APIntOps::mulhs(a, b);
          }
          return llvm::APIntOps::mulhu(a, b);
        });

    if (!highBits)
      return failure();

```
- **EN**: Implements logic around `constFoldBinaryOp`, `failure`, `mulhs`, `mulhu`.
- **CN**: 围绕 `constFoldBinaryOp`, `failure`, `mulhs`, `mulhu` 实现具体逻辑。

### Lines 261-278
```cpp
    Value lowBitsVal =
        spirv::ConstantOp::create(rewriter, loc, constituentType, lowBits);

    Value highBitsVal =
        spirv::ConstantOp::create(rewriter, loc, constituentType, highBits);

    // Create empty struct
    Value undef = spirv::UndefOp::create(rewriter, loc, op.getType());
    // Fill in lowBits at id 0
    Value intermediate =
        spirv::CompositeInsertOp::create(rewriter, loc, lowBitsVal, undef, 0);
    // Fill in highBits at id 1
    rewriter.replaceOpWithNewOp<spirv::CompositeInsertOp>(op, highBitsVal,
                                                          intermediate, 1);
    return success();
  }
};

```
- **EN**: Implements logic around `create`, `CompositeInsertOp>`, `success`; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `create`, `CompositeInsertOp>`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 279-303
```cpp
using SMulExtendedOpFold = MulExtendedFold<spirv::SMulExtendedOp, true>;
void spirv::SMulExtendedOp::getCanonicalizationPatterns(
    RewritePatternSet &patterns, MLIRContext *context) {
  patterns.add<SMulExtendedOpFold>(context);
}

struct UMulExtendedOpXOne final : OpRewritePattern<spirv::UMulExtendedOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(spirv::UMulExtendedOp op,
                                PatternRewriter &rewriter) const override {
    Location loc = op.getLoc();
    Value lhs = op.getOperand1();
    Value rhs = op.getOperand2();
    Type constituentType = lhs.getType();

    // umulextended (x, 1) = <x, 0>
    if (matchPattern(rhs, m_One())) {
      Value zero = spirv::ConstantOp::getZero(constituentType, loc, rewriter);
      Value constituents[2] = {lhs, zero};
      rewriter.replaceOpWithNewOp<spirv::CompositeConstructOp>(op, op.getType(),
                                                               constituents);
      return success();
    }

```
- **EN**: Introduces declarations for `UMulExtendedOpXOne`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UMulExtendedOpXOne` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 304-324
```cpp
    return failure();
  }
};

using UMulExtendedOpFold = MulExtendedFold<spirv::UMulExtendedOp, false>;
void spirv::UMulExtendedOp::getCanonicalizationPatterns(
    RewritePatternSet &patterns, MLIRContext *context) {
  patterns.add<UMulExtendedOpFold, UMulExtendedOpXOne>(context);
}

//===----------------------------------------------------------------------===//
// spirv.UMod
//===----------------------------------------------------------------------===//

// Input:
//    %0 = spirv.UMod %arg0, %const32 : i32
//    %1 = spirv.UMod %0, %const4 : i32
// Output:
//    %0 = spirv.UMod %arg0, %const32 : i32
//    %1 = spirv.UMod %arg0, %const4 : i32

```
- **EN**: Implements logic around `failure`, `getCanonicalizationPatterns`, `UMulExtendedOpXOne>`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `failure`, `getCanonicalizationPatterns`, `UMulExtendedOpXOne>` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 325-357
```cpp
// The transformation is only applied if one divisor is a multiple of the other.

struct UModSimplification final : OpRewritePattern<spirv::UModOp> {
  using Base::Base;

  LogicalResult matchAndRewrite(spirv::UModOp umodOp,
                                PatternRewriter &rewriter) const override {
    auto prevUMod = umodOp.getOperand(0).getDefiningOp<spirv::UModOp>();
    if (!prevUMod)
      return failure();

    TypedAttr prevValue;
    TypedAttr currValue;
    if (!matchPattern(prevUMod.getOperand(1), m_Constant(&prevValue)) ||
        !matchPattern(umodOp.getOperand(1), m_Constant(&currValue)))
      return failure();

    // Ensure that previous divisor is a multiple of the current divisor. If
    // not, fail the transformation.
    bool isApplicable = false;
    if (auto prevInt = dyn_cast<IntegerAttr>(prevValue)) {
      auto currInt = cast<IntegerAttr>(currValue);
      isApplicable = prevInt.getValue().urem(currInt.getValue()) == 0;
    } else if (auto prevVec = dyn_cast<DenseElementsAttr>(prevValue)) {
      auto currVec = cast<DenseElementsAttr>(currValue);
      isApplicable = llvm::all_of(llvm::zip_equal(prevVec.getValues<APInt>(),
                                                  currVec.getValues<APInt>()),
                                  [](const auto &pair) {
                                    auto &[prev, curr] = pair;
                                    return prev.urem(curr) == 0;
                                  });
    }

```
- **EN**: Introduces declarations for `UModSimplification`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `UModSimplification` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 358-375
```cpp
    if (!isApplicable)
      return failure();

    // The transformation is safe. Replace the existing UMod operation with a
    // new UMod operation, using the original dividend and the current divisor.
    rewriter.replaceOpWithNewOp<spirv::UModOp>(
        umodOp, umodOp.getType(), prevUMod.getOperand(0), umodOp.getOperand(1));

    return success();
  }
};

void spirv::UModOp::getCanonicalizationPatterns(RewritePatternSet &patterns,
                                                MLIRContext *context) {
  patterns.add<UModSimplification>(context);
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `failure`, `UModOp>`, `getType`, `success`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `failure`, `UModOp>`, `getType`, `success`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 376-393
```cpp
// spirv.BitcastOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::BitcastOp::fold(FoldAdaptor /*adaptor*/) {
  Value curInput = getOperand();
  if (getType() == curInput.getType())
    return curInput;

  // Look through nested bitcasts.
  if (auto prevCast = curInput.getDefiningOp<spirv::BitcastOp>()) {
    Value prevInput = prevCast.getOperand();
    if (prevInput.getType() == getType())
      return prevInput;

    getOperandMutable().assign(prevInput);
    return getResult();
  }

```
- **EN**: Implements logic around `fold`, `getOperand`, `getType`, `BitcastOp>`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand`, `getType`, `BitcastOp>`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 394-411
```cpp
  // TODO(kuhar): Consider constant-folding the operand attribute.
  return {};
}

//===----------------------------------------------------------------------===//
// spirv.CompositeExtractOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::CompositeExtractOp::fold(FoldAdaptor adaptor) {
  Value compositeOp = getComposite();

  while (auto insertOp =
             compositeOp.getDefiningOp<spirv::CompositeInsertOp>()) {
    if (getIndices() == insertOp.getIndices())
      return insertOp.getObject();
    compositeOp = insertOp.getComposite();
  }

```
- **EN**: Implements logic around `fold`, `getComposite`, `CompositeInsertOp>`, `getIndices`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getComposite`, `CompositeInsertOp>`, `getIndices`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 412-429
```cpp
  if (auto constructOp =
          compositeOp.getDefiningOp<spirv::CompositeConstructOp>()) {
    auto type = cast<spirv::CompositeType>(constructOp.getType());
    if (getIndices().size() == 1 &&
        constructOp.getConstituents().size() == type.getNumElements()) {
      auto i = cast<IntegerAttr>(*getIndices().begin());
      if (i.getValue().getSExtValue() <
          static_cast<int64_t>(constructOp.getConstituents().size()))
        return constructOp.getConstituents()[i.getValue().getSExtValue()];
    }
  }

  auto indexVector = llvm::map_to_vector(getIndices(), [](Attribute attr) {
    return static_cast<unsigned>(cast<IntegerAttr>(attr).getInt());
  });
  return extractCompositeElement(adaptor.getComposite(), indexVector);
}

```
- **EN**: Implements logic around `CompositeConstructOp>`, `CompositeType>`, `getIndices`, `getConstituents`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `CompositeConstructOp>`, `CompositeType>`, `getIndices`, `getConstituents`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 430-456
```cpp
//===----------------------------------------------------------------------===//
// spirv.Constant
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ConstantOp::fold(FoldAdaptor /*adaptor*/) {
  return getValue();
}

//===----------------------------------------------------------------------===//
// spirv.IAdd
//===----------------------------------------------------------------------===//

OpFoldResult spirv::IAddOp::fold(FoldAdaptor adaptor) {
  // x + 0 = x
  if (matchPattern(getOperand2(), m_Zero()))
    return getOperand1();

  // According to the SPIR-V spec:
  //
  // The resulting value will equal the low-order N bits of the correct result
  // R, where N is the component width and R is computed with enough precision
  // to avoid overflow and underflow.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](APInt a, const APInt &b) { return std::move(a) + b; });
}

```
- **EN**: Implements logic around `fold`, `getValue`, `matchPattern`, `getOperand1`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getValue`, `matchPattern`, `getOperand1`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 457-478
```cpp
//===----------------------------------------------------------------------===//
// spirv.IMul
//===----------------------------------------------------------------------===//

OpFoldResult spirv::IMulOp::fold(FoldAdaptor adaptor) {
  // x * 0 == 0
  if (matchPattern(getOperand2(), m_Zero()))
    return getOperand2();
  // x * 1 = x
  if (matchPattern(getOperand2(), m_One()))
    return getOperand1();

  // According to the SPIR-V spec:
  //
  // The resulting value will equal the low-order N bits of the correct result
  // R, where N is the component width and R is computed with enough precision
  // to avoid overflow and underflow.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a, const APInt &b) { return a * b; });
}

```
- **EN**: Implements logic around `fold`, `matchPattern`, `getOperand2`, `getOperand1`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `getOperand2`, `getOperand1`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 479-497
```cpp
//===----------------------------------------------------------------------===//
// spirv.ISub
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ISubOp::fold(FoldAdaptor adaptor) {
  // x - x = 0
  if (getOperand1() == getOperand2())
    return Builder(getContext()).getZeroAttr(getType());

  // According to the SPIR-V spec:
  //
  // The resulting value will equal the low-order N bits of the correct result
  // R, where N is the component width and R is computed with enough precision
  // to avoid overflow and underflow.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](APInt a, const APInt &b) { return std::move(a) - b; });
}

```
- **EN**: Implements logic around `fold`, `getOperand1`, `Builder`, `constFoldBinaryOp`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand1`, `Builder`, `constFoldBinaryOp`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 498-526
```cpp
//===----------------------------------------------------------------------===//
// spirv.SDiv
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SDivOp::fold(FoldAdaptor adaptor) {
  // sdiv (x, 1) = x
  if (matchPattern(getOperand2(), m_One()))
    return getOperand1();

  // According to the SPIR-V spec:
  //
  // Signed-integer division of Operand 1 divided by Operand 2.
  // Results are computed per component. Behavior is undefined if Operand 2 is
  // 0. Behavior is undefined if Operand 2 is -1 and Operand 1 is the minimum
  // representable value for the operands' type, causing signed overflow.
  //
  // So don't fold during undefined behavior.
  bool div0OrOverflow = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (div0OrOverflow || isDivZeroOrOverflow(a, b)) {
          div0OrOverflow = true;
          return a;
        }
        return a.sdiv(b);
      });
  return div0OrOverflow ? Attribute() : res;
}

```
- **EN**: Implements logic around `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 527-562
```cpp
//===----------------------------------------------------------------------===//
// spirv.SMod
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SModOp::fold(FoldAdaptor adaptor) {
  // smod (x, 1) = 0
  if (matchPattern(getOperand2(), m_One()))
    return Builder(getContext()).getZeroAttr(getType());

  // According to SPIR-V spec:
  //
  // Signed remainder operation for the remainder whose sign matches the sign
  // of Operand 2. Behavior is undefined if Operand 2 is 0. Behavior is
  // undefined if Operand 2 is -1 and Operand 1 is the minimum representable
  // value for the operands' type, causing signed overflow. Otherwise, the
  // result is the remainder r of Operand 1 divided by Operand 2 where if
  // r ≠ 0, the sign of r is the same as the sign of Operand 2.
  //
  // So don't fold during undefined behavior
  bool div0OrOverflow = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (div0OrOverflow || isDivZeroOrOverflow(a, b)) {
          div0OrOverflow = true;
          return a;
        }
        APInt c = a.abs().urem(b.abs());
        if (c.isZero())
          return c;
        if (b.isNegative()) {
          APInt zero = APInt::getZero(c.getBitWidth());
          return a.isNegative() ? (zero - c) : (b + c);
        }
        return a.isNegative() ? (b - c) : c;
      });
  return div0OrOverflow ? Attribute() : res;
```
- **EN**: Implements logic around `fold`, `matchPattern`, `Builder`, `constFoldBinaryOp`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `Builder`, `constFoldBinaryOp`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 563-582
```cpp
}

//===----------------------------------------------------------------------===//
// spirv.SRem
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SRemOp::fold(FoldAdaptor adaptor) {
  // x % 1 = 0
  if (matchPattern(getOperand2(), m_One()))
    return Builder(getContext()).getZeroAttr(getType());

  // According to SPIR-V spec:
  //
  // Signed remainder operation for the remainder whose sign matches the sign
  // of Operand 1. Behavior is undefined if Operand 2 is 0. Behavior is
  // undefined if Operand 2 is -1 and Operand 1 is the minimum representable
  // value for the operands' type, causing signed overflow. Otherwise, the
  // result is the remainder r of Operand 1 divided by Operand 2 where if
  // r ≠ 0, the sign of r is the same as the sign of Operand 1.

```
- **EN**: Implements logic around `fold`, `matchPattern`, `Builder`; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `Builder` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 583-604
```cpp
  // Don't fold if it would do undefined behavior.
  bool div0OrOverflow = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](APInt a, const APInt &b) {
        if (div0OrOverflow || isDivZeroOrOverflow(a, b)) {
          div0OrOverflow = true;
          return a;
        }
        return a.srem(b);
      });
  return div0OrOverflow ? Attribute() : res;
}

//===----------------------------------------------------------------------===//
// spirv.UDiv
//===----------------------------------------------------------------------===//

OpFoldResult spirv::UDivOp::fold(FoldAdaptor adaptor) {
  // udiv (x, 1) = x
  if (matchPattern(getOperand2(), m_One()))
    return getOperand1();

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `isDivZeroOrOverflow`, `srem`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `isDivZeroOrOverflow`, `srem`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 605-622
```cpp
  // According to the SPIR-V spec:
  //
  // Unsigned-integer division of Operand 1 divided by Operand 2. Behavior is
  // undefined if Operand 2 is 0.
  //
  // So don't fold during undefined behavior.
  bool div0 = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (div0 || b.isZero()) {
          div0 = true;
          return a;
        }
        return a.udiv(b);
      });
  return div0 ? Attribute() : res;
}

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `isZero`, `udiv`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `isZero`, `udiv`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 623-649
```cpp
//===----------------------------------------------------------------------===//
// spirv.UMod
//===----------------------------------------------------------------------===//

OpFoldResult spirv::UModOp::fold(FoldAdaptor adaptor) {
  // umod (x, 1) = 0
  if (matchPattern(getOperand2(), m_One()))
    return Builder(getContext()).getZeroAttr(getType());

  // According to the SPIR-V spec:
  //
  // Unsigned modulo operation of Operand 1 modulo Operand 2. Behavior is
  // undefined if Operand 2 is 0.
  //
  // So don't fold during undefined behavior.
  bool div0 = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (div0 || b.isZero()) {
          div0 = true;
          return a;
        }
        return a.urem(b);
      });
  return div0 ? Attribute() : res;
}

```
- **EN**: Implements logic around `fold`, `matchPattern`, `Builder`, `constFoldBinaryOp`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `Builder`, `constFoldBinaryOp`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 650-669
```cpp
//===----------------------------------------------------------------------===//
// spirv.SNegate
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SNegateOp::fold(FoldAdaptor adaptor) {
  // -(-x) = 0 - (0 - x) = x
  auto op = getOperand();
  if (auto negateOp = op.getDefiningOp<spirv::SNegateOp>())
    return negateOp->getOperand(0);

  // According to the SPIR-V spec:
  //
  // Signed-integer subtract of Operand from zero.
  return constFoldUnaryOp<IntegerAttr>(
      adaptor.getOperands(), [](const APInt &a) {
        APInt zero = APInt::getZero(a.getBitWidth());
        return zero - a;
      });
}

```
- **EN**: Implements logic around `fold`, `getOperand`, `SNegateOp>`, `constFoldUnaryOp`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand`, `SNegateOp>`, `constFoldUnaryOp`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 670-688
```cpp
//===----------------------------------------------------------------------===//
// spirv.NotOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::NotOp::fold(spirv::NotOp::FoldAdaptor adaptor) {
  // !(!x) = x
  auto op = getOperand();
  if (auto notOp = op.getDefiningOp<spirv::NotOp>())
    return notOp->getOperand(0);

  // According to the SPIR-V spec:
  //
  // Complement the bits of Operand.
  return constFoldUnaryOp<IntegerAttr>(adaptor.getOperands(), [&](APInt a) {
    a.flipAllBits();
    return a;
  });
}

```
- **EN**: Implements logic around `fold`, `getOperand`, `NotOp>`, `constFoldUnaryOp`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand`, `NotOp>`, `constFoldUnaryOp`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 689-707
```cpp
//===----------------------------------------------------------------------===//
// spirv.LogicalAnd
//===----------------------------------------------------------------------===//

OpFoldResult spirv::LogicalAndOp::fold(FoldAdaptor adaptor) {
  if (std::optional<bool> rhs =
          getScalarOrSplatBoolAttr(adaptor.getOperand2())) {
    // x && true = x
    if (*rhs)
      return getOperand1();

    // x && false = false
    if (!*rhs)
      return adaptor.getOperand2();
  }

  return Attribute();
}

```
- **EN**: Implements logic around `fold`, `getScalarOrSplatBoolAttr`, `getOperand1`, `getOperand2`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getScalarOrSplatBoolAttr`, `getOperand1`, `getOperand2`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 708-728
```cpp
//===----------------------------------------------------------------------===//
// spirv.LogicalEqualOp
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::LogicalEqualOp::fold(spirv::LogicalEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [](const APInt &a, const APInt &b) {
        return a == b ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

```
- **EN**: Implements logic around `fold`, `getOperand1`, `get`, `getType`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand1`, `get`, `getType`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 729-749
```cpp
//===----------------------------------------------------------------------===//
// spirv.LogicalNotEqualOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::LogicalNotEqualOp::fold(FoldAdaptor adaptor) {
  if (std::optional<bool> rhs =
          getScalarOrSplatBoolAttr(adaptor.getOperand2())) {
    // x != false -> x
    if (!rhs.value())
      return getOperand1();
  }

  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `fold`, `getScalarOrSplatBoolAttr`, `value`, `getOperand1`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getScalarOrSplatBoolAttr`, `value`, `getOperand1`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 750-775
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [](const APInt &a, const APInt &b) {
        return a == b ? APInt::getZero(1) : APInt::getAllOnes(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.LogicalNot
//===----------------------------------------------------------------------===//

OpFoldResult spirv::LogicalNotOp::fold(FoldAdaptor adaptor) {
  // !(!x) = x
  auto op = getOperand();
  if (auto notOp = op.getDefiningOp<spirv::LogicalNotOp>())
    return notOp->getOperand(0);

  // According to the SPIR-V spec:
  //
  // Complement the bits of Operand.
  return constFoldUnaryOp<IntegerAttr>(adaptor.getOperands(),
                                       [](const APInt &a) {
                                         APInt zero = APInt::getZero(1);
                                         return a == 1 ? zero : (zero + 1);
                                       });
}

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `getZero`, `fold`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `getZero`, `fold`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 776-794
```cpp
void spirv::LogicalNotOp::getCanonicalizationPatterns(
    RewritePatternSet &results, MLIRContext *context) {
  results
      .add<ConvertLogicalNotOfIEqual, ConvertLogicalNotOfINotEqual,
           ConvertLogicalNotOfLogicalEqual, ConvertLogicalNotOfLogicalNotEqual>(
          context);
}

//===----------------------------------------------------------------------===//
// spirv.LogicalOr
//===----------------------------------------------------------------------===//

OpFoldResult spirv::LogicalOrOp::fold(FoldAdaptor adaptor) {
  if (auto rhs = getScalarOrSplatBoolAttr(adaptor.getOperand2())) {
    if (*rhs) {
      // x || true = true
      return adaptor.getOperand2();
    }

```
- **EN**: Implements logic around `getCanonicalizationPatterns`, `ConvertLogicalNotOfLogicalNotEqual>`, `fold`, `getScalarOrSplatBoolAttr`, and 1 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getCanonicalizationPatterns`, `ConvertLogicalNotOfLogicalNotEqual>`, `fold`, `getScalarOrSplatBoolAttr`, and 1 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 795-814
```cpp
    if (!*rhs) {
      // x || false = x
      return getOperand1();
    }
  }

  return Attribute();
}

//===----------------------------------------------------------------------===//
// spirv.SelectOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SelectOp::fold(FoldAdaptor adaptor) {
  // spirv.Select _ x x -> x
  Value trueVals = getTrueValue();
  Value falseVals = getFalseValue();
  if (trueVals == falseVals)
    return trueVals;

```
- **EN**: Implements logic around `getOperand1`, `Attribute`, `fold`, `getTrueValue`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getOperand1`, `Attribute`, `fold`, `getTrueValue`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 815-834
```cpp
  ArrayRef<Attribute> operands = adaptor.getOperands();

  // spirv.Select true  x y -> x
  // spirv.Select false x y -> y
  if (auto boolAttr = getScalarOrSplatBoolAttr(operands[0]))
    return *boolAttr ? trueVals : falseVals;

  // Check that all the operands are constant
  if (!operands[0] || !operands[1] || !operands[2])
    return Attribute();

  // Note: getScalarOrSplatBoolAttr will always return a boolAttr if we are in
  // the scalar case. Hence, we are only required to consider the case of
  // DenseElementsAttr in foldSelectOp.
  auto condAttrs = dyn_cast<DenseElementsAttr>(operands[0]);
  auto trueAttrs = dyn_cast<DenseElementsAttr>(operands[1]);
  auto falseAttrs = dyn_cast<DenseElementsAttr>(operands[2]);
  if (!condAttrs || !trueAttrs || !falseAttrs)
    return Attribute();

```
- **EN**: Implements logic around `getOperands`, `getScalarOrSplatBoolAttr`, `Attribute`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getOperands`, `getScalarOrSplatBoolAttr`, `Attribute` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 835-860
```cpp
  auto elementResults = llvm::to_vector<4>(trueAttrs.getValues<Attribute>());
  auto iters = llvm::zip_equal(elementResults, condAttrs.getValues<BoolAttr>(),
                               falseAttrs.getValues<Attribute>());
  for (auto [result, cond, falseRes] : iters) {
    if (!cond.getValue())
      result = falseRes;
  }

  auto resultType = trueAttrs.getType();
  return DenseElementsAttr::get(cast<ShapedType>(resultType), elementResults);
}

//===----------------------------------------------------------------------===//
// spirv.IEqualOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::IEqualOp::fold(spirv::IEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

```
- **EN**: Implements logic around `to_vector`, `zip_equal`, `getValues`, `getValue`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `to_vector`, `zip_equal`, `getValues`, `getValue`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 861-880
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a == b ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.INotEqualOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::INotEqualOp::fold(spirv::INotEqualOp::FoldAdaptor adaptor) {
  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `getAllOnes`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `getAllOnes`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 881-901
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a == b ? APInt::getZero(1) : APInt::getAllOnes(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.SGreaterThan
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::SGreaterThanOp::fold(spirv::SGreaterThanOp::FoldAdaptor adaptor) {
  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `getZero`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `getZero`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 902-922
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.sgt(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.SGreaterThanEqual
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SGreaterThanEqualOp::fold(
    spirv::SGreaterThanEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `sgt`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `sgt`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 923-943
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.sge(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.UGreaterThan
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::UGreaterThanOp::fold(spirv::UGreaterThanOp::FoldAdaptor adaptor) {
  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `sge`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `sge`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 944-964
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.ugt(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.UGreaterThanEqual
//===----------------------------------------------------------------------===//

OpFoldResult spirv::UGreaterThanEqualOp::fold(
    spirv::UGreaterThanEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `ugt`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `ugt`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 965-984
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.uge(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.SLessThan
//===----------------------------------------------------------------------===//

OpFoldResult spirv::SLessThanOp::fold(spirv::SLessThanOp::FoldAdaptor adaptor) {
  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `uge`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `uge`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 985-1005
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.slt(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.SLessThanEqual
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::SLessThanEqualOp::fold(spirv::SLessThanEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `slt`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `slt`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1006-1025
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.sle(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.ULessThan
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ULessThanOp::fold(spirv::ULessThanOp::FoldAdaptor adaptor) {
  // x == x -> false
  if (getOperand1() == getOperand2()) {
    auto falseAttr = BoolAttr::get(getContext(), false);
    if (isa<IntegerType>(getType()))
      return falseAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, falseAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `sle`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `sle`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1026-1046
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.ult(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.ULessThanEqual
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::ULessThanEqualOp::fold(spirv::ULessThanEqualOp::FoldAdaptor adaptor) {
  // x == x -> true
  if (getOperand1() == getOperand2()) {
    auto trueAttr = BoolAttr::get(getContext(), true);
    if (isa<IntegerType>(getType()))
      return trueAttr;
    if (auto vecTy = dyn_cast<VectorType>(getType()))
      return SplatElementsAttr::get(vecTy, trueAttr);
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `ult`, `fold`, and 3 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `ult`, `fold`, and 3 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1047-1065
```cpp
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), getType(), [](const APInt &a, const APInt &b) {
        return a.ule(b) ? APInt::getAllOnes(1) : APInt::getZero(1);
      });
}

//===----------------------------------------------------------------------===//
// spirv.ShiftLeftLogical
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ShiftLeftLogicalOp::fold(
    spirv::ShiftLeftLogicalOp::FoldAdaptor adaptor) {
  // x << 0 -> x
  if (matchPattern(adaptor.getOperand2(), m_Zero())) {
    return getOperand1();
  }

  // Unfortunately due to below undefined behaviour can't fold 0 for Base.

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `ule`, `fold`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `ule`, `fold`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1066-1083
```cpp
  // Results are computed per component, and within each component, per bit...
  //
  // The result is undefined if Shift is greater than or equal to the bit width
  // of the components of Base.
  //
  // So we can use the APInt << method, but don't fold if undefined behaviour.
  bool shiftToLarge = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (shiftToLarge || b.uge(a.getBitWidth())) {
          shiftToLarge = true;
          return a;
        }
        return a << b;
      });
  return shiftToLarge ? Attribute() : res;
}

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `uge`, `Attribute`; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `uge`, `Attribute` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理。

### Lines 1084-1114
```cpp
//===----------------------------------------------------------------------===//
// spirv.ShiftRightArithmetic
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ShiftRightArithmeticOp::fold(
    spirv::ShiftRightArithmeticOp::FoldAdaptor adaptor) {
  // x >> 0 -> x
  if (matchPattern(adaptor.getOperand2(), m_Zero())) {
    return getOperand1();
  }

  // Unfortunately due to below undefined behaviour can't fold 0, -1 for Base.

  // Results are computed per component, and within each component, per bit...
  //
  // The result is undefined if Shift is greater than or equal to the bit width
  // of the components of Base.
  //
  // So we can use the APInt ashr method, but don't fold if undefined behaviour.
  bool shiftToLarge = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (shiftToLarge || b.uge(a.getBitWidth())) {
          shiftToLarge = true;
          return a;
        }
        return a.ashr(b);
      });
  return shiftToLarge ? Attribute() : res;
}

```
- **EN**: Implements logic around `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1115-1145
```cpp
//===----------------------------------------------------------------------===//
// spirv.ShiftRightLogical
//===----------------------------------------------------------------------===//

OpFoldResult spirv::ShiftRightLogicalOp::fold(
    spirv::ShiftRightLogicalOp::FoldAdaptor adaptor) {
  // x >> 0 -> x
  if (matchPattern(adaptor.getOperand2(), m_Zero())) {
    return getOperand1();
  }

  // Unfortunately due to below undefined behaviour can't fold 0 for Base.

  // Results are computed per component, and within each component, per bit...
  //
  // The result is undefined if Shift is greater than or equal to the bit width
  // of the components of Base.
  //
  // So we can use the APInt lshr method, but don't fold if undefined behaviour.
  bool shiftToLarge = false;
  auto res = constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(), [&](const APInt &a, const APInt &b) {
        if (shiftToLarge || b.uge(a.getBitWidth())) {
          shiftToLarge = true;
          return a;
        }
        return a.lshr(b);
      });
  return shiftToLarge ? Attribute() : res;
}

```
- **EN**: Implements logic around `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `matchPattern`, `getOperand1`, `constFoldBinaryOp`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1146-1166
```cpp
//===----------------------------------------------------------------------===//
// spirv.BitwiseAndOp
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::BitwiseAndOp::fold(spirv::BitwiseAndOp::FoldAdaptor adaptor) {
  // x & x -> x
  if (getOperand1() == getOperand2()) {
    return getOperand1();
  }

  APInt rhsMask;
  if (matchPattern(adaptor.getOperand2(), m_ConstantInt(&rhsMask))) {
    // x & 0 -> 0
    if (rhsMask.isZero())
      return getOperand2();

    // x & <all ones> -> x
    if (rhsMask.isAllOnes())
      return getOperand1();

```
- **EN**: Implements logic around `fold`, `getOperand1`, `matchPattern`, `isZero`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand1`, `matchPattern`, `isZero`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1167-1185
```cpp
    // (UConvert x : iN to iK) & <mask with N low bits set> -> UConvert x
    if (auto zext = getOperand1().getDefiningOp<spirv::UConvertOp>()) {
      int valueBits =
          getElementTypeOrSelf(zext.getOperand()).getIntOrFloatBitWidth();
      if (rhsMask.zextOrTrunc(valueBits).isAllOnes())
        return getOperand1();
    }
  }

  // According to the SPIR-V spec:
  //
  // Type is a scalar or vector of integer type.
  // Results are computed per component, and within each component, per bit.
  // So we can use the APInt & method.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a, const APInt &b) { return a & b; });
}

```
- **EN**: Implements logic around `getOperand1`, `getElementTypeOrSelf`, `zextOrTrunc`, `constFoldBinaryOp`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOperand1`, `getElementTypeOrSelf`, `zextOrTrunc`, `constFoldBinaryOp`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1186-1206
```cpp
//===----------------------------------------------------------------------===//
// spirv.BitwiseOrOp
//===----------------------------------------------------------------------===//

OpFoldResult spirv::BitwiseOrOp::fold(spirv::BitwiseOrOp::FoldAdaptor adaptor) {
  // x | x -> x
  if (getOperand1() == getOperand2()) {
    return getOperand1();
  }

  APInt rhsMask;
  if (matchPattern(adaptor.getOperand2(), m_ConstantInt(&rhsMask))) {
    // x | 0 -> x
    if (rhsMask.isZero())
      return getOperand1();

    // x | <all ones> -> <all ones>
    if (rhsMask.isAllOnes())
      return getOperand2();
  }

```
- **EN**: Implements logic around `fold`, `getOperand1`, `matchPattern`, `isZero`, and 2 more symbols; this block performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `fold`, `getOperand1`, `matchPattern`, `isZero`, and 2 more symbols 实现具体逻辑；该代码块执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义。

### Lines 1207-1227
```cpp
  // According to the SPIR-V spec:
  //
  // Type is a scalar or vector of integer type.
  // Results are computed per component, and within each component, per bit.
  // So we can use the APInt | method.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a, const APInt &b) { return a | b; });
}

//===----------------------------------------------------------------------===//
// spirv.BitwiseXorOp
//===----------------------------------------------------------------------===//

OpFoldResult
spirv::BitwiseXorOp::fold(spirv::BitwiseXorOp::FoldAdaptor adaptor) {
  // x ^ 0 -> x
  if (matchPattern(adaptor.getOperand2(), m_Zero())) {
    return getOperand1();
  }

```
- **EN**: Implements logic around `constFoldBinaryOp`, `getOperands`, `fold`, `matchPattern`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes; performs canonicalization or folding-oriented reasoning; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `constFoldBinaryOp`, `getOperands`, `fold`, `matchPattern`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并执行面向规范化或折叠的推理，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1228-1245
```cpp
  // x ^ x -> 0
  if (getOperand1() == getOperand2())
    return Builder(getContext()).getZeroAttr(getType());

  // According to the SPIR-V spec:
  //
  // Type is a scalar or vector of integer type.
  // Results are computed per component, and within each component, per bit.
  // So we can use the APInt ^ method.
  return constFoldBinaryOp<IntegerAttr>(
      adaptor.getOperands(),
      [](const APInt &a, const APInt &b) { return a ^ b; });
}

//===----------------------------------------------------------------------===//
// spirv.mlir.selection
//===----------------------------------------------------------------------===//

```
- **EN**: Implements logic around `getOperand1`, `Builder`, `constFoldBinaryOp`, `getOperands`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getOperand1`, `Builder`, `constFoldBinaryOp`, `getOperands` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1246-1272
```cpp
namespace {
// Blocks from the given `spirv.mlir.selection` operation must satisfy the
// following layout:
//
//       +-----------------------------------------------+
//       | header block                                  |
//       | spirv.BranchConditionalOp %cond, ^case0, ^case1 |
//       +-----------------------------------------------+
//                            /   \
//                             ...
//
//
//   +------------------------+    +------------------------+
//   | case #0                |    | case #1                |
//   | spirv.Store %ptr %value0 |    | spirv.Store %ptr %value1 |
//   | spirv.Branch ^merge      |    | spirv.Branch ^merge      |
//   +------------------------+    +------------------------+
//
//
//                             ...
//                            \   /
//                              v
//                       +-------------+
//                       | merge block |
//                       +-------------+
//
struct ConvertSelectionOpToSelect final : OpRewritePattern<spirv::SelectionOp> {
```
- **EN**: Introduces declarations for `ConvertSelectionOpToSelect`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ConvertSelectionOpToSelect` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1273-1294
```cpp
  using Base::Base;

  LogicalResult matchAndRewrite(spirv::SelectionOp selectionOp,
                                PatternRewriter &rewriter) const override {
    Operation *op = selectionOp.getOperation();
    Region &body = op->getRegion(0);
    // Verifier allows an empty region for `spirv.mlir.selection`.
    if (body.empty()) {
      return failure();
    }

    // Check that region consists of 4 blocks:
    // header block, `true` block, `false` block and merge block.
    if (llvm::range_size(body) != 4) {
      return failure();
    }

    Block *headerBlock = selectionOp.getHeaderBlock();
    if (!onlyContainsBranchConditionalOp(headerBlock)) {
      return failure();
    }

```
- **EN**: Implements logic around `matchAndRewrite`, `getOperation`, `getRegion`, `empty`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `matchAndRewrite`, `getOperation`, `getRegion`, `empty`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1295-1316
```cpp
    auto brConditionalOp =
        cast<spirv::BranchConditionalOp>(headerBlock->front());

    Block *trueBlock = brConditionalOp.getSuccessor(0);
    Block *falseBlock = brConditionalOp.getSuccessor(1);
    Block *mergeBlock = selectionOp.getMergeBlock();

    if (failed(canCanonicalizeSelection(trueBlock, falseBlock, mergeBlock)))
      return failure();

    Value trueValue = getSrcValue(trueBlock);
    Value falseValue = getSrcValue(falseBlock);
    Value ptrValue = getDstPtr(trueBlock);
    auto storeOpAttributes =
        cast<spirv::StoreOp>(trueBlock->front())->getAttrs();

    auto selectOp = spirv::SelectOp::create(
        rewriter, selectionOp.getLoc(), trueValue.getType(),
        brConditionalOp.getCondition(), trueValue, falseValue);
    spirv::StoreOp::create(rewriter, selectOp.getLoc(), ptrValue,
                           selectOp.getResult(), storeOpAttributes);

```
- **EN**: Implements logic around `BranchConditionalOp>`, `getSuccessor`, `getMergeBlock`, `failed`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `BranchConditionalOp>`, `getSuccessor`, `getMergeBlock`, `failed`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 1317-1336
```cpp
    // `spirv.mlir.selection` is not needed anymore.
    rewriter.eraseOp(op);
    return success();
  }

private:
  // Checks that given blocks follow the following rules:
  // 1. Each conditional block consists of two operations, the first operation
  //    is a `spirv.Store` and the last operation is a `spirv.Branch`.
  // 2. Each `spirv.Store` uses the same pointer and the same memory attributes.
  // 3. A control flow goes into the given merge block from the given
  //    conditional blocks.
  LogicalResult canCanonicalizeSelection(Block *trueBlock, Block *falseBlock,
                                         Block *mergeBlock) const;

  bool onlyContainsBranchConditionalOp(Block *block) const {
    return llvm::hasSingleElement(*block) &&
           isa<spirv::BranchConditionalOp>(block->front());
  }

```
- **EN**: Implements logic around `eraseOp`, `success`, `canCanonicalizeSelection`, `onlyContainsBranchConditionalOp`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `eraseOp`, `success`, `canCanonicalizeSelection`, `onlyContainsBranchConditionalOp`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并涉及目标平台或加速器专用语义。

### Lines 1337-1355
```cpp
  bool isSameAttrList(spirv::StoreOp lhs, spirv::StoreOp rhs) const {
    return lhs->getDiscardableAttrDictionary() ==
               rhs->getDiscardableAttrDictionary() &&
           lhs.getProperties() == rhs.getProperties();
  }

  // Returns a source value for the given block.
  Value getSrcValue(Block *block) const {
    auto storeOp = cast<spirv::StoreOp>(block->front());
    return storeOp.getValue();
  }

  // Returns a destination value for the given block.
  Value getDstPtr(Block *block) const {
    auto storeOp = cast<spirv::StoreOp>(block->front());
    return storeOp.getPtr();
  }
};

```
- **EN**: Implements logic around `isSameAttrList`, `getDiscardableAttrDictionary`, `getProperties`, `getSrcValue`, and 4 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `isSameAttrList`, `getDiscardableAttrDictionary`, `getProperties`, `getSrcValue`, and 4 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1356-1374
```cpp
LogicalResult ConvertSelectionOpToSelect::canCanonicalizeSelection(
    Block *trueBlock, Block *falseBlock, Block *mergeBlock) const {
  // Each block must consists of 2 operations.
  if (llvm::range_size(*trueBlock) != 2 || llvm::range_size(*falseBlock) != 2) {
    return failure();
  }

  auto trueBrStoreOp = dyn_cast<spirv::StoreOp>(trueBlock->front());
  auto trueBrBranchOp =
      dyn_cast<spirv::BranchOp>(*std::next(trueBlock->begin()));
  auto falseBrStoreOp = dyn_cast<spirv::StoreOp>(falseBlock->front());
  auto falseBrBranchOp =
      dyn_cast<spirv::BranchOp>(*std::next(falseBlock->begin()));

  if (!trueBrStoreOp || !trueBrBranchOp || !falseBrStoreOp ||
      !falseBrBranchOp) {
    return failure();
  }

```
- **EN**: Implements logic around `canCanonicalizeSelection`, `range_size`, `failure`, `StoreOp>`, and 1 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `canCanonicalizeSelection`, `range_size`, `failure`, `StoreOp>`, and 1 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1375-1395
```cpp
  // Checks that given type is valid for `spirv.SelectOp`.
  // According to SPIR-V spec:
  // "Before version 1.4, Result Type must be a pointer, scalar, or vector.
  // Starting with version 1.4, Result Type can additionally be a composite type
  // other than a vector."
  bool isScalarOrVector =
      cast<spirv::SPIRVType>(trueBrStoreOp.getValue().getType())
          .isScalarOrVector();

  // Check that each `spirv.Store` uses the same pointer, memory access
  // attributes and a valid type of the value.
  if ((trueBrStoreOp.getPtr() != falseBrStoreOp.getPtr()) ||
      !isSameAttrList(trueBrStoreOp, falseBrStoreOp) || !isScalarOrVector) {
    return failure();
  }

  if ((trueBrBranchOp->getSuccessor(0) != mergeBlock) ||
      (falseBrBranchOp->getSuccessor(0) != mergeBlock)) {
    return failure();
  }

```
- **EN**: Implements logic around `SPIRVType>`, `isScalarOrVector`, `getPtr`, `isSameAttrList`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `SPIRVType>`, `isScalarOrVector`, `getPtr`, `isSameAttrList`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1396-1403
```cpp
  return success();
}
} // namespace

void spirv::SelectionOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                                     MLIRContext *context) {
  results.add<ConvertSelectionOpToSelect>(context);
}
```
- **EN**: Implements logic around `success`, `getCanonicalizationPatterns`, `add`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `success`, `getCanonicalizationPatterns`, `add` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SPIRV/IR/SPIRVOps.h`, `mlir/Dialect/CommonFolders.h`, `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`, `mlir/Dialect/UB/IR/UBOps.h`, `mlir/IR/Matchers.h`, `mlir/IR/PatternMatch.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVectorExtras.h`, `SPIRVCanonicalization.inc`
- **Standard-library headers / 标准库头文件**: `<optional>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2)
