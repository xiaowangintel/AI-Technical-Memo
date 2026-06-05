# OuterProductFusion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/ArmSME/Transforms/OuterProductFusion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements rewrites that fuse 'arm_sme.outerproduct' operations into the 2-way or 4-way widening outerproduct operations.
  - **CN**: 实现 ArmSME 方言与 Arm SME tile/vector 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- OuterProductFusion.cpp - Fuse 'arm_sme.outerproduct' ops -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements rewrites that fuse 'arm_sme.outerproduct' operations
// into the 2-way or 4-way widening outerproduct operations.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-23
```cpp

#include "mlir/Dialect/ArmSME/IR/ArmSME.h"
#include "mlir/Dialect/ArmSME/Transforms/Passes.h"
#include "mlir/Dialect/ArmSME/Transforms/Transforms.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/TypeSwitch.h"

#define DEBUG_TYPE "arm-sme-outerproduct-fusion"

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 24-33
```cpp
namespace mlir::arm_sme {
#define GEN_PASS_DEF_OUTERPRODUCTFUSION
#include "mlir/Dialect/ArmSME/Transforms/Passes.h.inc"
} // namespace mlir::arm_sme

using namespace mlir;
using namespace mlir::arm_sme;

namespace {

```
- **EN**: Introduces declarations for `mlir::arm_sme`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir::arm_sme` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 34-47
```cpp
// Common match failure reasons.
static constexpr StringLiteral
    kMatchFailureNoAccumulator("no accumulator operand");
static constexpr StringLiteral kMatchFailureExpectedOuterProductDefOp(
    "defining op of accumulator must be 'arm_sme.outerproduct'");
static constexpr StringLiteral kMatchFailureInconsistentCombiningKind(
    "combining kind (add or sub) of outer products must match");
static constexpr StringLiteral kMatchFailureInconsistentMasking(
    "unsupported masking, either both outerproducts are masked "
    "or neither");
static constexpr StringLiteral kMatchFailureOuterProductNotSingleUse(
    "outer product(s) not single use and cannot be removed, no benefit to "
    "fusing");

```
- **EN**: Implements logic around `kMatchFailureNoAccumulator`, `kMatchFailureExpectedOuterProductDefOp`, `kMatchFailureInconsistentCombiningKind`, `kind`, and 3 more symbols.
- **CN**: 围绕 `kMatchFailureNoAccumulator`, `kMatchFailureExpectedOuterProductDefOp`, `kMatchFailureInconsistentCombiningKind`, `kind`, and 3 more symbols 实现具体逻辑。

### Lines 48-62
```cpp
// An outer product is compatible if all of the following are true:
// - the result type matches `resultType`.
// - the defining operation of LHS is of the type `LhsExtOp`.
// - the defining operation of RHS is of the type `RhsExtOp`.
// - the input types of the defining operations are identical and match
//   `inputType`.
template <typename LhsExtOp, typename RhsExtOp = LhsExtOp>
static LogicalResult isCompatible(PatternRewriter &rewriter,
                                  arm_sme::OuterProductOp op,
                                  VectorType resultType, VectorType inputType) {
  if (op.getResultType() != resultType)
    return rewriter.notifyMatchFailure(op.getLoc(), [&](Diagnostic &diag) {
      diag << "unsupported result type, expected " << resultType;
    });

```
- **EN**: Implements logic around `isCompatible`, `getResultType`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isCompatible`, `getResultType`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 63-73
```cpp
  auto lhsDefOp = op.getLhs().getDefiningOp<LhsExtOp>();
  auto rhsDefOp = op.getRhs().getDefiningOp<RhsExtOp>();

  if (!lhsDefOp || !rhsDefOp)
    return rewriter.notifyMatchFailure(
        op, "defining op of outerproduct operands must be one of: "
            "'arith.extf' or 'arith.extsi' or 'arith.extui'");

  auto lhsInType = cast<VectorType>(lhsDefOp.getIn().getType());
  auto rhsInType = cast<VectorType>(rhsDefOp.getIn().getType());

```
- **EN**: Implements logic around `getLhs`, `getRhs`, `notifyMatchFailure`, `cast`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLhs`, `getRhs`, `notifyMatchFailure`, `cast` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 74-93
```cpp
  if (lhsInType != inputType || rhsInType != inputType)
    return rewriter.notifyMatchFailure(op.getLoc(), [&](Diagnostic &diag) {
      diag << "unsupported input type, expected " << inputType;
    });

  return success();
}

// Fuse two 'arm_sme.outerproduct' operations that are chained via the
// accumulator into 2-way outer product operation.
//
// For example:
//
//  %a0_ext = arith.extf %a0 : vector<[4]xf16> to vector<[4]xf32>
//  %b0_ext = arith.extf %b0 : vector<[4]xf16> to vector<[4]xf32>
//  %0 = arm_sme.outerproduct %a0_ext, %b0_ext : vector<[4]xf32>,
//                                               vector<[4]xf32>
//
//  %a1_ext = arith.extf %a1 : vector<[4]xf16> to vector<[4]xf32>
//  %b1_ext = arith.extf %b1 : vector<[4]xf16> to vector<[4]xf32>
```
- **EN**: Implements logic around `notifyMatchFailure`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `notifyMatchFailure`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 94-103
```cpp
//  %1 = arm_sme.outerproduct %a1_ext, %b1_ext, %0 : vector<[4]xf32>,
//                                                   vector<[4]xf32>
//
// Becomes:
//
//  %a_packed = vector.interleave %a0, %a1 : vector<[4]xf16> -> vector<[8]xf16>
//  %b_packed = vector.interleave %b0, %b1 : vector<[4]xf16> -> vector<[8]xf16>
//  %0 = arm_sme.fmopa_2way %a_packed, %b_packed
//    : vector<[8]xf16>, vector<[8]xf16> into vector<[4]x[4]xf32>
class OuterProductFusion2Way
```
- **EN**: Introduces declarations for `OuterProductFusion2Way`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `OuterProductFusion2Way` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 104-113
```cpp
    : public OpRewritePattern<arm_sme::OuterProductOp> {
public:
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(arm_sme::OuterProductOp op,
                                PatternRewriter &rewriter) const override {
    Value acc = op.getAcc();
    if (!acc)
      return rewriter.notifyMatchFailure(op, kMatchFailureNoAccumulator);

```
- **EN**: Implements logic around `matchAndRewrite`, `getAcc`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getAcc`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 114-123
```cpp
    arm_sme::OuterProductOp op1 = acc.getDefiningOp<arm_sme::OuterProductOp>();
    arm_sme::OuterProductOp op2 = op;
    if (!op1)
      return rewriter.notifyMatchFailure(
          op, kMatchFailureExpectedOuterProductDefOp);

    if (op1.getKind() != op2.getKind())
      return rewriter.notifyMatchFailure(
          op, kMatchFailureInconsistentCombiningKind);

```
- **EN**: Implements logic around `OuterProductOp>`, `notifyMatchFailure`, `getKind`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `OuterProductOp>`, `notifyMatchFailure`, `getKind` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 124-133
```cpp
    if (!op1->hasOneUse()) {
      // If the first outer product has uses other than as the input to another
      // outer product, it can't be erased after fusion.
      return rewriter.notifyMatchFailure(op,
                                         kMatchFailureOuterProductNotSingleUse);
    }

    if (bool(op1.getLhsMask()) != bool(op2.getLhsMask()))
      return rewriter.notifyMatchFailure(op, kMatchFailureInconsistentMasking);

```
- **EN**: Implements logic around `hasOneUse`, `notifyMatchFailure`, `bool`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `hasOneUse`, `notifyMatchFailure`, `bool` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 134-146
```cpp
    if (failed(canFuseOuterProducts(rewriter, op1, op2)))
      return failure();

    auto loc = op.getLoc();
    auto packInputs = [&](Value lhs, Value rhs) {
      return vector::InterleaveOp::create(rewriter, loc, lhs, rhs);
    };

    auto lhs = packInputs(op1.getLhs().getDefiningOp()->getOperand(0),
                          op2.getLhs().getDefiningOp()->getOperand(0));
    auto rhs = packInputs(op1.getRhs().getDefiningOp()->getOperand(0),
                          op2.getRhs().getDefiningOp()->getOperand(0));

```
- **EN**: Implements logic around `failed`, `failure`, `getLoc`, `create`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `failed`, `failure`, `getLoc`, `create`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 147-166
```cpp
    Value lhsMask, rhsMask;
    if (op1.getLhsMask() || op2.getLhsMask()) {
      lhsMask = packInputs(op1.getLhsMask(), op2.getLhsMask());
      rhsMask = packInputs(op1.getRhsMask(), op2.getRhsMask());
    }

    auto *extOp = op.getLhs().getDefiningOp();

    arm_sme::CombiningKind kind = op.getKind();
    if (kind == arm_sme::CombiningKind::Add) {
      TypeSwitch<Operation *>(extOp)
          .Case([&](arith::ExtFOp) {
            rewriter.replaceOpWithNewOp<arm_sme::FMopa2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
          })
          .Case([&](arith::ExtSIOp) {
            rewriter.replaceOpWithNewOp<arm_sme::SMopa2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
```
- **EN**: Implements logic around `getLhsMask`, `packInputs`, `getLhs`, `getKind`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLhsMask`, `packInputs`, `getLhs`, `getKind`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 167-186
```cpp
          })
          .Case([&](arith::ExtUIOp) {
            rewriter.replaceOpWithNewOp<arm_sme::UMopa2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
          })
          .DefaultUnreachable("unexpected extend op!");
    } else if (kind == arm_sme::CombiningKind::Sub) {
      TypeSwitch<Operation *>(extOp)
          .Case([&](arith::ExtFOp) {
            rewriter.replaceOpWithNewOp<arm_sme::FMops2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
          })
          .Case([&](arith::ExtSIOp) {
            rewriter.replaceOpWithNewOp<arm_sme::SMops2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
          })
          .Case([&](arith::ExtUIOp) {
```
- **EN**: Implements logic around `Case`, `UMopa2WayOp>`, `getResultType`, `getAcc`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `Case`, `UMopa2WayOp>`, `getResultType`, `getAcc`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 187-198
```cpp
            rewriter.replaceOpWithNewOp<arm_sme::UMops2WayOp>(
                op2, op.getResultType(), lhs, rhs, lhsMask, rhsMask,
                op1.getAcc());
          })
          .DefaultUnreachable("unexpected extend op!");
    } else {
      llvm_unreachable("unexpected arm_sme::CombiningKind!");
    }

    return success();
  }

```
- **EN**: Implements logic around `UMops2WayOp>`, `getResultType`, `getAcc`, `DefaultUnreachable`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `UMops2WayOp>`, `getResultType`, `getAcc`, `DefaultUnreachable`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 199-218
```cpp
private:
  // A pair of outer product can be fused if all of the following are true:
  // - input and result types match.
  // - the defining operations of the inputs are identical extensions,
  //   specifically either:
  //     - a signed or unsigned extension for integer types.
  //     - a floating-point extension for floating-point types.
  // - the types and extension are supported, i.e. there's a 2-way operation
  //   they can be fused into.
  LogicalResult canFuseOuterProducts(PatternRewriter &rewriter,
                                     arm_sme::OuterProductOp op1,
                                     arm_sme::OuterProductOp op2) const {
    // Supported result types.
    auto nxnxv4i32 =
        VectorType::get({4, 4}, rewriter.getI32Type(), {true, true});
    auto nxnxv4f32 =
        VectorType::get({4, 4}, rewriter.getF32Type(), {true, true});
    // Supported input types.
    // Note: this is before packing so these have half the number of elements
    // of the input vector types of the 2-way operations.
```
- **EN**: Implements logic around `canFuseOuterProducts`, `get`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `canFuseOuterProducts`, `get` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 219-238
```cpp
    auto nxv4i16 = VectorType::get({4}, rewriter.getI16Type(), true);
    auto nxv4f16 = VectorType::get({4}, rewriter.getF16Type(), true);
    auto nxv4bf16 = VectorType::get({4}, rewriter.getBF16Type(), true);
    if ((failed(
             isCompatible<arith::ExtFOp>(rewriter, op1, nxnxv4f32, nxv4f16)) ||
         failed(
             isCompatible<arith::ExtFOp>(rewriter, op2, nxnxv4f32, nxv4f16))) &&
        (failed(
             isCompatible<arith::ExtFOp>(rewriter, op1, nxnxv4f32, nxv4bf16)) ||
         failed(isCompatible<arith::ExtFOp>(rewriter, op2, nxnxv4f32,
                                            nxv4bf16))) &&
        (failed(
             isCompatible<arith::ExtSIOp>(rewriter, op1, nxnxv4i32, nxv4i16)) ||
         failed(isCompatible<arith::ExtSIOp>(rewriter, op2, nxnxv4i32,
                                             nxv4i16))) &&
        (failed(
             isCompatible<arith::ExtUIOp>(rewriter, op1, nxnxv4i32, nxv4i16)) ||
         failed(
             isCompatible<arith::ExtUIOp>(rewriter, op2, nxnxv4i32, nxv4i16))))
      return failure();
```
- **EN**: Implements logic around `get`, `failed`, `ExtFOp>`, `ExtSIOp>`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `get`, `failed`, `ExtFOp>`, `ExtSIOp>`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 239-248
```cpp

    return success();
  }
};

// Fuse four 'arm_sme.outerproduct' operations that are chained via the
// accumulator into 4-way outer product operation.
class OuterProductFusion4Way
    : public OpRewritePattern<arm_sme::OuterProductOp> {
public:
```
- **EN**: Introduces declarations for `OuterProductFusion4Way`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `OuterProductFusion4Way` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 249-268
```cpp
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(arm_sme::OuterProductOp op,
                                PatternRewriter &rewriter) const override {
    SmallVector<arm_sme::OuterProductOp, 4> outerProductChain;
    outerProductChain.push_back(op);

    for (int i = 0; i < 3; ++i) {
      auto currentOp = outerProductChain.back();
      auto acc = currentOp.getAcc();
      if (!acc)
        return rewriter.notifyMatchFailure(op, kMatchFailureNoAccumulator);
      auto previousOp = acc.getDefiningOp<arm_sme::OuterProductOp>();
      if (!previousOp)
        return rewriter.notifyMatchFailure(
            op, kMatchFailureExpectedOuterProductDefOp);
      if (!previousOp->hasOneUse())
        return rewriter.notifyMatchFailure(
            op, kMatchFailureOuterProductNotSingleUse);
      if (previousOp.getKind() != currentOp.getKind())
```
- **EN**: Implements logic around `matchAndRewrite`, `push_back`, `back`, `getAcc`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `push_back`, `back`, `getAcc`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 269-279
```cpp
        return rewriter.notifyMatchFailure(
            op, kMatchFailureInconsistentCombiningKind);
      if (bool(previousOp.getLhsMask()) != bool(currentOp.getLhsMask()))
        return rewriter.notifyMatchFailure(
            op, kMatchFailureInconsistentCombiningKind);
      outerProductChain.push_back(previousOp);
    }

    if (failed(canFuseOuterProducts(rewriter, outerProductChain)))
      return failure();

```
- **EN**: Implements logic around `notifyMatchFailure`, `bool`, `push_back`, `failed`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `notifyMatchFailure`, `bool`, `push_back`, `failed`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 280-289
```cpp
    arm_sme::OuterProductOp op1 = outerProductChain[3];
    arm_sme::OuterProductOp op2 = outerProductChain[2];
    arm_sme::OuterProductOp op3 = outerProductChain[1];
    arm_sme::OuterProductOp op4 = outerProductChain[0];

    auto loc = op.getLoc();
    auto packInputs = [&](Value lhs, Value rhs) {
      return vector::InterleaveOp::create(rewriter, loc, lhs, rhs);
    };

```
- **EN**: Implements logic around `getLoc`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 290-301
```cpp
    auto lhs0 = packInputs(op1.getLhs().getDefiningOp()->getOperand(0),
                           op3.getLhs().getDefiningOp()->getOperand(0));
    auto lhs1 = packInputs(op2.getLhs().getDefiningOp()->getOperand(0),
                           op4.getLhs().getDefiningOp()->getOperand(0));
    auto lhs = packInputs(lhs0, lhs1);

    auto rhs0 = packInputs(op1.getRhs().getDefiningOp()->getOperand(0),
                           op3.getRhs().getDefiningOp()->getOperand(0));
    auto rhs1 = packInputs(op2.getRhs().getDefiningOp()->getOperand(0),
                           op4.getRhs().getDefiningOp()->getOperand(0));
    auto rhs = packInputs(rhs0, rhs1);

```
- **EN**: Implements logic around `packInputs`, `getLhs`, `getRhs`.
- **CN**: 围绕 `packInputs`, `getLhs`, `getRhs` 实现具体逻辑。

### Lines 302-313
```cpp
    Value lhsMask, rhsMask;
    if (op1.getLhsMask() || op2.getLhsMask() || op3.getLhsMask() ||
        op4.getLhsMask()) {
      auto lhs0Mask = packInputs(op1.getLhsMask(), op3.getLhsMask());
      auto lhs1Mask = packInputs(op2.getLhsMask(), op4.getLhsMask());
      lhsMask = packInputs(lhs0Mask, lhs1Mask);

      auto rhs0Mask = packInputs(op1.getRhsMask(), op3.getRhsMask());
      auto rhs1Mask = packInputs(op2.getRhsMask(), op4.getRhsMask());
      rhsMask = packInputs(rhs0Mask, rhs1Mask);
    }

```
- **EN**: Implements logic around `getLhsMask`, `packInputs`.
- **CN**: 围绕 `getLhsMask`, `packInputs` 实现具体逻辑。

### Lines 314-333
```cpp
    auto *lhsExtOp = op.getLhs().getDefiningOp();
    auto *rhsExtOp = op.getRhs().getDefiningOp();

    arm_sme::CombiningKind kind = op.getKind();
    if (kind == arm_sme::CombiningKind::Add) {
      if (isa<arith::ExtSIOp>(lhsExtOp) && isa<arith::ExtSIOp>(rhsExtOp)) {
        // signed
        rewriter.replaceOpWithNewOp<arm_sme::SMopa4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtUIOp>(lhsExtOp) &&
                 isa<arith::ExtUIOp>(rhsExtOp)) {
        // unsigned
        rewriter.replaceOpWithNewOp<arm_sme::UMopa4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtSIOp>(lhsExtOp) &&
                 isa<arith::ExtUIOp>(rhsExtOp)) {
        // signed by unsigned
        rewriter.replaceOpWithNewOp<arm_sme::SuMopa4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtUIOp>(lhsExtOp) &&
```
- **EN**: Implements logic around `getLhs`, `getRhs`, `getKind`, `ExtSIOp>`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLhs`, `getRhs`, `getKind`, `ExtSIOp>`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 334-353
```cpp
                 isa<arith::ExtSIOp>(rhsExtOp)) {
        // unsigned by signed
        rewriter.replaceOpWithNewOp<arm_sme::UsMopa4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else {
        llvm_unreachable("unexpected extend op!");
      }
    } else if (kind == arm_sme::CombiningKind::Sub) {
      if (isa<arith::ExtSIOp>(lhsExtOp) && isa<arith::ExtSIOp>(rhsExtOp)) {
        // signed
        rewriter.replaceOpWithNewOp<arm_sme::SMops4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtUIOp>(lhsExtOp) &&
                 isa<arith::ExtUIOp>(rhsExtOp)) {
        // unsigned
        rewriter.replaceOpWithNewOp<arm_sme::UMops4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtSIOp>(lhsExtOp) &&
                 isa<arith::ExtUIOp>(rhsExtOp)) {
        // signed by unsigned
```
- **EN**: Implements logic around `ExtSIOp>`, `UsMopa4WayOp>`, `getResultType`, `llvm_unreachable`, and 3 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `ExtSIOp>`, `UsMopa4WayOp>`, `getResultType`, `llvm_unreachable`, and 3 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 354-367
```cpp
        rewriter.replaceOpWithNewOp<arm_sme::SuMops4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else if (isa<arith::ExtUIOp>(lhsExtOp) &&
                 isa<arith::ExtSIOp>(rhsExtOp)) {
        // unsigned by signed
        rewriter.replaceOpWithNewOp<arm_sme::UsMops4WayOp>(
            op4, op.getResultType(), lhs, rhs, lhsMask, rhsMask, op1.getAcc());
      } else {
        llvm_unreachable("unexpected extend op!");
      }
    } else {
      llvm_unreachable("unexpected arm_sme::CombiningKind!");
    }

```
- **EN**: Implements logic around `SuMops4WayOp>`, `getResultType`, `ExtUIOp>`, `ExtSIOp>`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `SuMops4WayOp>`, `getResultType`, `ExtUIOp>`, `ExtSIOp>`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 368-387
```cpp
    return success();
  }

private:
  // Four outer products can be fused if all of the following are true:
  // - input and result types match.
  // - the defining operations of the inputs are identical extensions,
  //   specifically either:
  //     - a signed or unsigned extension for integer types.
  //     - a floating-point extension for floating-point types.
  // - the types and extension are supported, i.e. there's a 4-way operation
  //   they can be fused into.
  LogicalResult
  canFuseOuterProducts(PatternRewriter &rewriter,
                       ArrayRef<arm_sme::OuterProductOp> ops) const {
    // Supported result types.
    auto nxnxv4i32 =
        VectorType::get({4, 4}, rewriter.getI32Type(), {true, true});
    auto nxnxv2i64 =
        VectorType::get({2, 2}, rewriter.getI64Type(), {true, true});
```
- **EN**: Implements logic around `success`, `canFuseOuterProducts`, `get`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `success`, `canFuseOuterProducts`, `get` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 388-406
```cpp

    // Supported input types.
    // Note: this is before packing so these have 1/4 the number of elements
    // of the input vector types of the 4-way operations.
    auto nxv4i8 = VectorType::get({4}, rewriter.getI8Type(), true);
    auto nxv2i16 = VectorType::get({2}, rewriter.getI16Type(), true);

    auto failedToMatch = [&](VectorType resultType, VectorType inputType,
                             auto lhsExtendOp, auto rhsExtendOp) {
      using LhsExtendOpTy = decltype(lhsExtendOp);
      using RhsExtendOpTy = decltype(rhsExtendOp);
      for (auto op : ops) {
        if (failed(isCompatible<LhsExtendOpTy, RhsExtendOpTy>(
                rewriter, op, resultType, inputType)))
          return true;
      }
      return false;
    };

```
- **EN**: Implements logic around `get`, `decltype`, `failed`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `get`, `decltype`, `failed` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 407-416
```cpp
    if (failedToMatch(nxnxv4i32, nxv4i8, arith::ExtSIOp{}, arith::ExtSIOp{}) &&
        failedToMatch(nxnxv4i32, nxv4i8, arith::ExtUIOp{}, arith::ExtUIOp{}) &&
        failedToMatch(nxnxv4i32, nxv4i8, arith::ExtSIOp{}, arith::ExtUIOp{}) &&
        failedToMatch(nxnxv4i32, nxv4i8, arith::ExtUIOp{}, arith::ExtSIOp{}) &&
        failedToMatch(nxnxv2i64, nxv2i16, arith::ExtSIOp{}, arith::ExtSIOp{}) &&
        failedToMatch(nxnxv2i64, nxv2i16, arith::ExtUIOp{}, arith::ExtUIOp{}) &&
        failedToMatch(nxnxv2i64, nxv2i16, arith::ExtSIOp{}, arith::ExtUIOp{}) &&
        failedToMatch(nxnxv2i64, nxv2i16, arith::ExtUIOp{}, arith::ExtSIOp{}))
      return failure();

```
- **EN**: Implements logic around `failedToMatch`, `failure`.
- **CN**: 围绕 `failedToMatch`, `failure` 实现具体逻辑。

### Lines 417-432
```cpp
    return success();
  }
};

// Rewrites: vector.extract(arith.extend) -> arith.extend(vector.extract).
//
// This transforms IR like:
//   %0 = arith.extsi %src : vector<4x[8]xi8> to vector<4x[8]xi32>
//   %1 = vector.extract %0[0] : vector<[8]xi32> from vector<4x[8]xi32>
// Into:
//   %0 = vector.extract %src[0] : vector<[8]xi8> from vector<4x[8]xi8>
//   %1 = arith.extsi %0 : vector<[8]xi8> to vector<[8]xi32>
//
// This enables outer product fusion in the `-arm-sme-outer-product-fusion`
// pass when the result is the input to an outer product.
struct SwapVectorExtractOfArithExtend
```
- **EN**: Introduces declarations for `SwapVectorExtractOfArithExtend`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SwapVectorExtractOfArithExtend` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 433-442
```cpp
    : public OpRewritePattern<vector::ExtractOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::ExtractOp extractOp,
                                PatternRewriter &rewriter) const override {
    VectorType resultType = llvm::dyn_cast<VectorType>(extractOp.getType());
    if (!resultType)
      return rewriter.notifyMatchFailure(extractOp,
                                         "extracted type is not a vector type");

```
- **EN**: Implements logic around `matchAndRewrite`, `dyn_cast`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `dyn_cast`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 443-453
```cpp
    auto numScalableDims = resultType.getNumScalableDims();
    if (numScalableDims != 1)
      return rewriter.notifyMatchFailure(
          extractOp, "extracted type is not a 1-D scalable vector type");

    auto *extendOp = extractOp.getSource().getDefiningOp();
    if (!isa_and_present<arith::ExtSIOp, arith::ExtUIOp, arith::ExtFOp>(
            extendOp))
      return rewriter.notifyMatchFailure(extractOp,
                                         "extract not from extend op");

```
- **EN**: Implements logic around `getNumScalableDims`, `notifyMatchFailure`, `getSource`, `ExtFOp>`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getNumScalableDims`, `notifyMatchFailure`, `getSource`, `ExtFOp>` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 454-465
```cpp
    auto loc = extractOp.getLoc();
    StringAttr extendOpName = extendOp->getName().getIdentifier();
    Value extendSource = extendOp->getOperand(0);

    // Create new extract from source of extend.
    Value newExtract = vector::ExtractOp::create(rewriter, loc, extendSource,
                                                 extractOp.getMixedPosition());

    // Extend new extract to original result type.
    Operation *newExtend =
        rewriter.create(loc, extendOpName, Value(newExtract), resultType);

```
- **EN**: Implements logic around `getLoc`, `getName`, `getOperand`, `create`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getName`, `getOperand`, `create`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 466-483
```cpp
    rewriter.replaceOp(extractOp, newExtend);

    return success();
  }
};

// Same as above, but for vector.scalable.extract.
//
// This transforms IR like:
//   %0 = arith.extsi %src : vector<[8]xi8> to vector<[8]xi32>
//   %1 = vector.scalable.extract %0[0] : vector<[4]xi32> from vector<[8]xi32>
// Into:
//   %0 = vector.scalable.extract %src[0] : vector<[4]xi8> from vector<[8]xi8>
//   %1 = arith.extsi %0 : vector<[4]xi8> to vector<[4]xi32>
//
// This enables outer product fusion in the `-arm-sme-outer-product-fusion`
// pass when the result is the input to an outer product.
struct SwapVectorScalableExtractOfArithExtend
```
- **EN**: Introduces declarations for `SwapVectorScalableExtractOfArithExtend`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SwapVectorScalableExtractOfArithExtend` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 484-494
```cpp
    : public OpRewritePattern<vector::ScalableExtractOp> {
  using OpRewritePattern::OpRewritePattern;

  LogicalResult matchAndRewrite(vector::ScalableExtractOp extractOp,
                                PatternRewriter &rewriter) const override {
    auto *extendOp = extractOp.getSource().getDefiningOp();
    if (!isa_and_present<arith::ExtSIOp, arith::ExtUIOp, arith::ExtFOp>(
            extendOp))
      return rewriter.notifyMatchFailure(extractOp,
                                         "extract not from extend op");

```
- **EN**: Implements logic around `matchAndRewrite`, `getSource`, `ExtFOp>`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `matchAndRewrite`, `getSource`, `ExtFOp>`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 495-509
```cpp
    auto loc = extractOp.getLoc();
    VectorType resultType = extractOp.getResultVectorType();

    Value extendSource = extendOp->getOperand(0);
    StringAttr extendOpName = extendOp->getName().getIdentifier();
    VectorType extendSourceVectorType =
        cast<VectorType>(extendSource.getType());

    // Create new extract from source of extend.
    VectorType extractResultVectorType =
        resultType.clone(extendSourceVectorType.getElementType());
    Value newExtract = vector::ScalableExtractOp::create(
        rewriter, loc, extractResultVectorType, extendSource,
        extractOp.getPos());

```
- **EN**: Implements logic around `getLoc`, `getResultVectorType`, `getOperand`, `getName`, and 4 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getLoc`, `getResultVectorType`, `getOperand`, `getName`, and 4 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 510-519
```cpp
    // Extend new extract to original result type.
    Operation *newExtend =
        rewriter.create(loc, extendOpName, Value(newExtract), resultType);

    rewriter.replaceOp(extractOp, newExtend);

    return success();
  }
};

```
- **EN**: Implements logic around `create`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 520-531
```cpp
struct OuterProductFusionPass
    : public arm_sme::impl::OuterProductFusionBase<OuterProductFusionPass> {

  void runOnOperation() override {
    RewritePatternSet patterns(&getContext());
    populateOuterProductFusionPatterns(patterns);

    if (failed(applyPatternsGreedily(getOperation(), std::move(patterns))))
      signalPassFailure();
  }
};

```
- **EN**: Introduces declarations for `OuterProductFusionPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `OuterProductFusionPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 532-542
```cpp
} // namespace

void mlir::arm_sme::populateOuterProductFusionPatterns(
    RewritePatternSet &patterns) {
  MLIRContext *context = patterns.getContext();
  // Note: High benefit to ensure extract(extend) are swapped first.
  patterns.add<SwapVectorExtractOfArithExtend,
               SwapVectorScalableExtractOfArithExtend>(context, 1024);
  patterns.add<OuterProductFusion2Way, OuterProductFusion4Way>(context);
}

```
- **EN**: Implements logic around `populateOuterProductFusionPatterns`, `getContext`, `SwapVectorScalableExtractOfArithExtend>`, `OuterProductFusion4Way>`.
- **CN**: 围绕 `populateOuterProductFusionPatterns`, `getContext`, `SwapVectorScalableExtractOfArithExtend>`, `OuterProductFusion4Way>` 实现具体逻辑。

### Lines 543-545
```cpp
std::unique_ptr<Pass> mlir::arm_sme::createOuterProductFusionPass() {
  return std::make_unique<OuterProductFusionPass>();
}
```
- **EN**: Implements logic around `createOuterProductFusionPass`, `make_unique`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `createOuterProductFusionPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/ArmSME/IR/ArmSME.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h`, `mlir/Dialect/ArmSME/Transforms/Transforms.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/TypeSwitch.h`, `mlir/Dialect/ArmSME/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR IR core abstractions / MLIR IR 核心抽象 (1), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
