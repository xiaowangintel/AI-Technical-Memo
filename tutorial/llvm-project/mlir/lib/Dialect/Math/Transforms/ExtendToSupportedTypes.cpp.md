# ExtendToSupportedTypes.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Math/Transforms/ExtendToSupportedTypes.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Math dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ExtendToSupportedTypes`.
  - **CN**: 实现 Math 方言中围绕 `ExtendToSupportedTypes` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
//===- ExtendToSupportedTypes.cpp - Legalize functions on unsupported floats
//----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 9-22
```cpp
//
// This file implements legalizing math operations on unsupported floating-point
// types through arith.extf and arith.truncf.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/Math/IR/Math.h"
#include "mlir/Dialect/Math/Transforms/Passes.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/TypeUtilities.h"
#include "mlir/Transforms/DialectConversion.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`。

### Lines 23-30
```cpp
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SetVector.h"

namespace mlir::math {
#define GEN_PASS_DEF_MATHEXTENDTOSUPPORTEDTYPES
#include "mlir/Dialect/Math/Transforms/Passes.h.inc"
} // namespace mlir::math

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `mlir/Dialect/Math/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h`, `mlir/Dialect/Math/Transforms/Passes.h.inc`。

### Lines 31-42
```cpp
using namespace mlir;

namespace {
struct ExtendToSupportedTypesRewritePattern final : ConversionPattern {
  ExtendToSupportedTypesRewritePattern(const TypeConverter &converter,
                                       MLIRContext *context)
      : ConversionPattern(converter, MatchAnyOpTypeTag{}, 1, context) {}
  LogicalResult
  matchAndRewrite(Operation *op, ArrayRef<Value> operands,
                  ConversionPatternRewriter &rewriter) const override;
};

```
- **EN**: Introduces declarations for `mlir`, `ExtendToSupportedTypesRewritePattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `ExtendToSupportedTypesRewritePattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 43-52
```cpp
struct ExtendToSupportedTypesPass
    : mlir::math::impl::MathExtendToSupportedTypesBase<
          ExtendToSupportedTypesPass> {
  using math::impl::MathExtendToSupportedTypesBase<
      ExtendToSupportedTypesPass>::MathExtendToSupportedTypesBase;

  void runOnOperation() override;
};
} // namespace

```
- **EN**: Introduces declarations for `ExtendToSupportedTypesPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ExtendToSupportedTypesPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 53-63
```cpp
void mlir::math::populateExtendToSupportedTypesTypeConverter(
    TypeConverter &typeConverter, const SetVector<Type> &sourceTypes,
    Type targetType) {

  typeConverter.addConversion(
      [](Type type) -> std::optional<Type> { return type; });
  typeConverter.addConversion(
      [&sourceTypes, targetType](FloatType type) -> std::optional<Type> {
        if (!sourceTypes.contains(type))
          return targetType;

```
- **EN**: Implements logic around `populateExtendToSupportedTypesTypeConverter`, `addConversion`, `contains`; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `populateExtendToSupportedTypesTypeConverter`, `addConversion`, `contains` 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 64-71
```cpp
        return std::nullopt;
      });
  typeConverter.addConversion(
      [&sourceTypes, targetType](ShapedType type) -> std::optional<Type> {
        if (auto elemTy = dyn_cast<FloatType>(type.getElementType()))
          if (!sourceTypes.contains(elemTy))
            return type.clone(targetType);

```
- **EN**: Implements logic around `addConversion`, `getElementType`, `contains`, `clone`; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `addConversion`, `getElementType`, `contains`, `clone` 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 72-81
```cpp
        return std::nullopt;
      });
  typeConverter.addTargetMaterialization(
      [](OpBuilder &b, Type target, ValueRange input, Location loc) {
        auto extFOp = arith::ExtFOp::create(b, loc, target, input);
        extFOp.setFastmath(arith::FastMathFlags::contract);
        return extFOp;
      });
}

```
- **EN**: Implements logic around `addTargetMaterialization`, `create`, `setFastmath`; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `addTargetMaterialization`, `create`, `setFastmath` 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 82-92
```cpp
void mlir::math::populateExtendToSupportedTypesConversionTarget(
    ConversionTarget &target, TypeConverter &typeConverter) {
  target.markUnknownOpDynamicallyLegal([&typeConverter](Operation *op) -> bool {
    if (isa<MathDialect>(op->getDialect()))
      return typeConverter.isLegal(op);
    return true;
  });
  target.addLegalOp<FmaOp>();
  target.addLegalOp<arith::ExtFOp, arith::TruncFOp>();
}

```
- **EN**: Implements logic around `populateExtendToSupportedTypesConversionTarget`, `markUnknownOpDynamicallyLegal`, `getDialect`, `isLegal`, and 2 more symbols; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `populateExtendToSupportedTypesConversionTarget`, `markUnknownOpDynamicallyLegal`, `getDialect`, `isLegal`, and 2 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 93-102
```cpp
LogicalResult ExtendToSupportedTypesRewritePattern::matchAndRewrite(
    Operation *op, ArrayRef<Value> operands,
    ConversionPatternRewriter &rewriter) const {
  Location loc = op->getLoc();
  const TypeConverter *converter = getTypeConverter();
  FailureOr<Operation *> legalized =
      convertOpResultTypes(op, operands, *converter, rewriter);
  if (failed(legalized))
    return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getLoc`, `getTypeConverter`, `convertOpResultTypes`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `matchAndRewrite`, `getLoc`, `getTypeConverter`, `convertOpResultTypes`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调方言转换或 lowering 决策。

### Lines 103-115
```cpp
  SmallVector<Value> results = (*legalized)->getResults();
  for (auto [result, newType, origType] : llvm::zip_equal(
           results, (*legalized)->getResultTypes(), op->getResultTypes())) {
    if (newType != origType) {
      auto truncFOp = arith::TruncFOp::create(rewriter, loc, origType, result);
      truncFOp.setFastmath(arith::FastMathFlags::contract);
      result = truncFOp.getResult();
    }
  }
  rewriter.replaceOp(op, results);
  return success();
}

```
- **EN**: Implements logic around `getResults`, `zip_equal`, `getResultTypes`, `create`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getResults`, `zip_equal`, `getResultTypes`, `create`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 116-125
```cpp
void mlir::math::populateExtendToSupportedTypesPatterns(
    RewritePatternSet &patterns, const TypeConverter &typeConverter) {
  patterns.add<ExtendToSupportedTypesRewritePattern>(typeConverter,
                                                     patterns.getContext());
}

void ExtendToSupportedTypesPass::runOnOperation() {
  Operation *op = getOperation();
  MLIRContext *ctx = &getContext();

```
- **EN**: Implements logic around `populateExtendToSupportedTypesPatterns`, `add`, `getContext`, `runOnOperation`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper; coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `populateExtendToSupportedTypesPatterns`, `add`, `getContext`, `runOnOperation`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并协调方言转换或 lowering 决策。

### Lines 126-134
```cpp
  // Parse target type
  FloatType targetType = arith::parseFloatType(ctx, targetTypeStr);
  if (!targetType) {
    emitError(UnknownLoc::get(ctx), "could not map target type '" +
                                        targetTypeStr +
                                        "' to a known floating-point type");
    return signalPassFailure();
  }

```
- **EN**: Implements logic around `parseFloatType`, `emitError`, `signalPassFailure`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `parseFloatType`, `emitError`, `signalPassFailure` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 135-148
```cpp
  // Parse source types
  llvm::SetVector<Type> sourceTypes;
  for (const auto &extraTypeStr : extraTypeStrs) {
    FloatType extraType = arith::parseFloatType(ctx, extraTypeStr);
    if (!extraType) {
      emitError(UnknownLoc::get(ctx), "could not map source type '" +
                                          extraTypeStr +
                                          "' to a known floating-point type");
      return signalPassFailure();
    }
    sourceTypes.insert(extraType);
  }
  // f64 and f32 are implicitly supported
  Builder b(ctx);
```
- **EN**: Implements logic around `parseFloatType`, `emitError`, `signalPassFailure`, `insert`, and 1 more symbols; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `parseFloatType`, `emitError`, `signalPassFailure`, `insert`, and 1 more symbols 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 149-161
```cpp
  sourceTypes.insert(b.getF64Type());
  sourceTypes.insert(b.getF32Type());

  TypeConverter typeConverter;
  math::populateExtendToSupportedTypesTypeConverter(typeConverter, sourceTypes,
                                                    targetType);
  ConversionTarget target(*ctx);
  math::populateExtendToSupportedTypesConversionTarget(target, typeConverter);
  RewritePatternSet patterns(ctx);
  math::populateExtendToSupportedTypesPatterns(patterns, typeConverter);
  if (failed(applyPartialConversion(op, target, std::move(patterns))))
    return signalPassFailure();
}
```
- **EN**: Implements logic around `insert`, `populateExtendToSupportedTypesTypeConverter`, `target`, `populateExtendToSupportedTypesConversionTarget`, and 4 more symbols; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `insert`, `populateExtendToSupportedTypesTypeConverter`, `target`, `populateExtendToSupportedTypesConversionTarget`, and 4 more symbols 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Dialect conversion pipeline / 方言转换流水线**:
  - **EN**: Coordinates legality checks, type adaptation, and target-specific lowering.
  - **CN**: 协调合法性检查、类型适配以及目标相关 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/Math/IR/Math.h`, `mlir/Dialect/Math/Transforms/Passes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/PatternMatch.h`, `mlir/IR/TypeUtilities.h`, `mlir/Transforms/DialectConversion.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SetVector.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
