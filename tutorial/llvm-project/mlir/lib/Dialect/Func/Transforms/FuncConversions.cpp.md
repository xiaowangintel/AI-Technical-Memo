# FuncConversions.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Func/Transforms/FuncConversions.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the Func dialect and function-like IR support.
  - **CN**: 实现 Func 方言与类函数 IR 支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- FuncConversions.cpp - Function conversions -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-15
```cpp

#include "mlir/Dialect/Func/Transforms/FuncConversions.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Transforms/DialectConversion.h"

using namespace mlir;
using namespace mlir::func;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Transforms/DialectConversion.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Transforms/DialectConversion.h`。

### Lines 16-23
```cpp
/// Flatten the given value ranges into a single vector of values.
static SmallVector<Value> flattenValues(ArrayRef<ValueRange> values) {
  SmallVector<Value> result;
  for (const auto &vals : values)
    llvm::append_range(result, vals);
  return result;
}

```
- **EN**: Implements logic around `flattenValues`, `append_range`.
- **CN**: 围绕 `flattenValues`, `append_range` 实现具体逻辑。

### Lines 24-30
```cpp
namespace {
/// Converts the operand and result types of the CallOp, used together with the
/// FuncOpSignatureConversion.
struct CallOpSignatureConversion : public OpConversionPattern<CallOp> {
  using OpConversionPattern<CallOp>::OpConversionPattern;

  /// Hook for derived classes to implement combined matching and rewriting.
```
- **EN**: Introduces declarations for `CallOpSignatureConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `CallOpSignatureConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 31-44
```cpp
  LogicalResult
  matchAndRewrite(CallOp callOp, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const override {
    // Convert the original function results. Keep track of how many result
    // types an original result type is converted into.
    SmallVector<size_t> numResultsReplacments;
    SmallVector<Type, 1> convertedResults;
    size_t numFlattenedResults = 0;
    for (auto [idx, type] : llvm::enumerate(callOp.getResultTypes())) {
      if (failed(typeConverter->convertTypes(type, convertedResults)))
        return failure();
      numResultsReplacments.push_back(convertedResults.size() -
                                      numFlattenedResults);
      numFlattenedResults = convertedResults.size();
```
- **EN**: Implements logic around `matchAndRewrite`, `enumerate`, `failed`, `failure`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `matchAndRewrite`, `enumerate`, `failed`, `failure`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 45-58
```cpp
    }

    // Substitute with the new result types from the corresponding FuncType
    // conversion.
    auto newCallOp =
        CallOp::create(rewriter, callOp.getLoc(), callOp.getCallee(),
                       convertedResults, flattenValues(adaptor.getOperands()));
    SmallVector<ValueRange> replacements;
    size_t offset = 0;
    for (int i = 0, e = callOp->getNumResults(); i < e; ++i) {
      replacements.push_back(
          newCallOp->getResults().slice(offset, numResultsReplacments[i]));
      offset += numResultsReplacments[i];
    }
```
- **EN**: Implements logic around `create`, `flattenValues`, `getNumResults`, `push_back`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `create`, `flattenValues`, `getNumResults`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 59-66
```cpp
    assert(offset == convertedResults.size() &&
           "expected that all converted results are used");
    rewriter.replaceOpWithMultiple(callOp, replacements);
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `assert`, `replaceOpWithMultiple`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure; works with symbol tables or function-like operations.
- **CN**: 围绕 `assert`, `replaceOpWithMultiple`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理符号表或类函数操作。

### Lines 67-73
```cpp
void mlir::populateCallOpTypeConversionPattern(RewritePatternSet &patterns,
                                               const TypeConverter &converter,
                                               PatternBenefit benefit) {
  patterns.add<CallOpSignatureConversion>(converter, patterns.getContext(),
                                          benefit);
}

```
- **EN**: Implements logic around `populateCallOpTypeConversionPattern`, `add`.
- **CN**: 围绕 `populateCallOpTypeConversionPattern`, `add` 实现具体逻辑。

### Lines 74-80
```cpp
namespace {
/// Only needed to support partial conversion of functions where this pattern
/// ensures that the branch operation arguments matches up with the succesor
/// block arguments.
class BranchOpInterfaceTypeConversion
    : public OpInterfaceConversionPattern<BranchOpInterface> {
public:
```
- **EN**: Introduces declarations for `BranchOpInterfaceTypeConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `BranchOpInterfaceTypeConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 81-90
```cpp
  using OpInterfaceConversionPattern<
      BranchOpInterface>::OpInterfaceConversionPattern;

  BranchOpInterfaceTypeConversion(
      const TypeConverter &typeConverter, MLIRContext *ctx,
      function_ref<bool(BranchOpInterface, int)> shouldConvertBranchOperand,
      PatternBenefit benefit)
      : OpInterfaceConversionPattern(typeConverter, ctx, benefit),
        shouldConvertBranchOperand(shouldConvertBranchOperand) {}

```
- **EN**: Implements logic around `BranchOpInterfaceTypeConversion`, `function_ref`, `OpInterfaceConversionPattern`, `shouldConvertBranchOperand`.
- **CN**: 围绕 `BranchOpInterfaceTypeConversion`, `function_ref`, `OpInterfaceConversionPattern`, `shouldConvertBranchOperand` 实现具体逻辑。

### Lines 91-103
```cpp
  LogicalResult
  matchAndRewrite(BranchOpInterface op, ArrayRef<Value> operands,
                  ConversionPatternRewriter &rewriter) const final {
    // For a branch operation, only some operands go to the target blocks, so
    // only rewrite those.
    SmallVector<Value, 4> newOperands(op->operand_begin(), op->operand_end());
    for (int succIdx = 0, succEnd = op->getBlock()->getNumSuccessors();
         succIdx < succEnd; ++succIdx) {
      OperandRange forwardedOperands =
          op.getSuccessorOperands(succIdx).getForwardedOperands();
      if (forwardedOperands.empty())
        continue;

```
- **EN**: Implements logic around `matchAndRewrite`, `newOperands`, `getBlock`, `getSuccessorOperands`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `matchAndRewrite`, `newOperands`, `getBlock`, `getSuccessorOperands`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并处理 MLIR region、block 或控制流边。

### Lines 104-115
```cpp
      for (int idx = forwardedOperands.getBeginOperandIndex(),
               eidx = idx + forwardedOperands.size();
           idx < eidx; ++idx) {
        if (!shouldConvertBranchOperand || shouldConvertBranchOperand(op, idx))
          newOperands[idx] = operands[idx];
      }
    }
    rewriter.modifyOpInPlace(
        op, [newOperands, op]() { op->setOperands(newOperands); });
    return success();
  }

```
- **EN**: Implements logic around `getBeginOperandIndex`, `size`, `shouldConvertBranchOperand`, `modifyOpInPlace`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getBeginOperandIndex`, `size`, `shouldConvertBranchOperand`, `modifyOpInPlace`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 116-122
```cpp
private:
  function_ref<bool(BranchOpInterface, int)> shouldConvertBranchOperand;
};
} // namespace

namespace {
/// Only needed to support partial conversion of functions where this pattern
```
- **EN**: Implements logic around `function_ref`.
- **CN**: 围绕 `function_ref` 实现具体逻辑。

### Lines 123-136
```cpp
/// ensures that the branch operation arguments matches up with the succesor
/// block arguments.
class ReturnOpTypeConversion : public OpConversionPattern<ReturnOp> {
public:
  using OpConversionPattern<ReturnOp>::OpConversionPattern;

  LogicalResult
  matchAndRewrite(ReturnOp op, OneToNOpAdaptor adaptor,
                  ConversionPatternRewriter &rewriter) const final {
    rewriter.replaceOpWithNewOp<ReturnOp>(op,
                                          flattenValues(adaptor.getOperands()));
    return success();
  }
};
```
- **EN**: Introduces declarations for `ReturnOpTypeConversion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ReturnOpTypeConversion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 137-147
```cpp
} // namespace

void mlir::populateBranchOpInterfaceTypeConversionPattern(
    RewritePatternSet &patterns, const TypeConverter &typeConverter,
    function_ref<bool(BranchOpInterface, int)> shouldConvertBranchOperand,
    PatternBenefit benefit) {
  patterns.add<BranchOpInterfaceTypeConversion>(
      typeConverter, patterns.getContext(), shouldConvertBranchOperand,
      benefit);
}

```
- **EN**: Implements logic around `populateBranchOpInterfaceTypeConversionPattern`, `function_ref`, `add`, `getContext`.
- **CN**: 围绕 `populateBranchOpInterfaceTypeConversionPattern`, `function_ref`, `add`, `getContext` 实现具体逻辑。

### Lines 148-160
```cpp
bool mlir::isLegalForBranchOpInterfaceTypeConversionPattern(
    Operation *op, const TypeConverter &converter) {
  // All successor operands of branch like operations must be rewritten.
  if (auto branchOp = dyn_cast<BranchOpInterface>(op)) {
    for (int p = 0, e = op->getBlock()->getNumSuccessors(); p < e; ++p) {
      auto successorOperands = branchOp.getSuccessorOperands(p);
      if (!converter.isLegal(
              successorOperands.getForwardedOperands().getTypes()))
        return false;
    }
    return true;
  }

```
- **EN**: Implements logic around `isLegalForBranchOpInterfaceTypeConversionPattern`, `dyn_cast`, `getBlock`, `getSuccessorOperands`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `isLegalForBranchOpInterfaceTypeConversionPattern`, `dyn_cast`, `getBlock`, `getSuccessorOperands`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 161-170
```cpp
  return false;
}

void mlir::populateReturnOpTypeConversionPattern(
    RewritePatternSet &patterns, const TypeConverter &typeConverter,
    PatternBenefit benefit) {
  patterns.add<ReturnOpTypeConversion>(typeConverter, patterns.getContext(),
                                       benefit);
}

```
- **EN**: Implements logic around `populateReturnOpTypeConversionPattern`, `add`.
- **CN**: 围绕 `populateReturnOpTypeConversionPattern`, `add` 实现具体逻辑。

### Lines 171-178
```cpp
bool mlir::isLegalForReturnOpTypeConversionPattern(
    Operation *op, const TypeConverter &converter, bool returnOpAlwaysLegal) {
  // If this is a `return` and the user pass wants to convert/transform across
  // function boundaries, then `converter` is invoked to check whether the
  // `return` op is legal.
  if (isa<ReturnOp>(op) && !returnOpAlwaysLegal)
    return converter.isLegal(op);

```
- **EN**: Implements logic around `isLegalForReturnOpTypeConversionPattern`, `isa`, `isLegal`.
- **CN**: 围绕 `isLegalForReturnOpTypeConversionPattern`, `isa`, `isLegal` 实现具体逻辑。

### Lines 179-188
```cpp
  // ReturnLike operations have to be legalized with their parent. For
  // return this is handled, for other ops they remain as is.
  return op->hasTrait<OpTrait::ReturnLike>();
}

bool mlir::isNotBranchOpInterfaceOrReturnLikeOp(Operation *op) {
  // If it is not a terminator, ignore it.
  if (!op->mightHaveTrait<OpTrait::IsTerminator>())
    return true;

```
- **EN**: Implements logic around `ReturnLike>`, `isNotBranchOpInterfaceOrReturnLikeOp`, `IsTerminator>`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `ReturnLike>`, `isNotBranchOpInterfaceOrReturnLikeOp`, `IsTerminator>` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 189-199
```cpp
  // If it is not the last operation in the block, also ignore it. We do
  // this to handle unknown operations, as well.
  Block *block = op->getBlock();
  if (!block || &block->back() != op)
    return true;

  // We don't want to handle terminators in nested regions, assume they are
  // always legal.
  if (!isa_and_nonnull<FuncOp>(op->getParentOp()))
    return true;

```
- **EN**: Implements logic around `getBlock`, `back`, `isa_and_nonnull`; this block manipulates MLIR regions, blocks, or control-flow edges; works with symbol tables or function-like operations.
- **CN**: 围绕 `getBlock`, `back`, `isa_and_nonnull` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边，并处理符号表或类函数操作。

### Lines 200-201
```cpp
  return false;
}
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Function-like regions / 类函数 Region**:
  - **EN**: Provides function operations, callable interfaces, and symbol-aware bodies.
  - **CN**: 提供函数操作、可调用接口以及符号感知的函数体。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/Transforms/FuncConversions.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Transforms/DialectConversion.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (2), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
