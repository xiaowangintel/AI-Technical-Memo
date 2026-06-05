# TransposeMatmul.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/TransposeMatmul.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This is intended to be a simple high-level (target-agnostic) matmul transposition transformation.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- TransposeMatmul.cpp - Convert Linalg matmul to transposed variants -===//
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
// This is intended to be a simple high-level (target-agnostic) matmul
// transposition transformation.
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`。

### Lines 15-21
```cpp
#define DEBUG_TYPE "linalg-transpose-matmul"

using namespace mlir;
using namespace mlir::linalg;

/// Pattern to replace
///
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 22-28
```cpp
///   linalg.matmul(a, b)
///
/// with
///
///   linalg.matmul_transpose_a(linalg.transpose(a), b)
///
/// By default the LHS is transposed. Set `transposeLHS=false` to
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 29-39
```cpp
/// transpose RHS instead.
FailureOr<Operation *> mlir::linalg::transposeMatmul(RewriterBase &rewriter,
                                                     linalg::MatmulOp matmulOp,
                                                     bool transposeLHS) {
  // Check to not let go the matmul with extended semantic, through this
  // transform.
  if (matmulOp.hasUserDefinedMaps()) {
    return rewriter.notifyMatchFailure(
        matmulOp, "only matmul ops with non-extended semantics are supported");
  }

```
- **EN**: Implements logic around `transposeMatmul`, `hasUserDefinedMaps`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transposeMatmul`, `hasUserDefinedMaps`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 40-47
```cpp
  if (!matmulOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        matmulOp, "only matmul ops with tensors are supported");

  Location loc = matmulOp.getLoc();
  Value input = matmulOp.getInputs()[transposeLHS ? 0 : 1];
  auto type = cast<ShapedType>(input.getType());

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `notifyMatchFailure`, `getLoc`, `getInputs`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasPureTensorSemantics`, `notifyMatchFailure`, `getLoc`, `getInputs`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 48-61
```cpp
  SmallVector<Value> dynamicDims;
  if (type.isDynamicDim(1))
    dynamicDims.push_back(tensor::DimOp::create(rewriter, loc, input, 1));
  if (type.isDynamicDim(0))
    dynamicDims.push_back(tensor::DimOp::create(rewriter, loc, input, 0));

  ArrayRef<int64_t> shape = type.getShape();
  Value empty = tensor::EmptyOp::create(rewriter, loc,
                                        ArrayRef<int64_t>{shape[1], shape[0]},
                                        type.getElementType(), dynamicDims);
  auto transposeOp = linalg::TransposeOp::create(rewriter, loc, input, empty,
                                                 ArrayRef<int64_t>{1, 0});
  Operation *newMatmulOp;
  if (transposeLHS) {
```
- **EN**: Implements logic around `isDynamicDim`, `push_back`, `getShape`, `create`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isDynamicDim`, `push_back`, `getShape`, `create`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 62-75
```cpp
    newMatmulOp = MatmulTransposeAOp::create(
        rewriter, loc, matmulOp.getResultTypes(),
        ValueRange{transposeOp->getResult(0), matmulOp.getInputs()[1]},
        matmulOp.getOutputs());
  } else {
    newMatmulOp = MatmulTransposeBOp::create(
        rewriter, loc, matmulOp.getResultTypes(),
        ValueRange{matmulOp.getInputs()[0], transposeOp->getResult(0)},
        matmulOp.getOutputs());
  }
  rewriter.replaceOp(matmulOp, newMatmulOp);
  return newMatmulOp;
}

```
- **EN**: Implements logic around `create`, `getResultTypes`, `getResult`, `getOutputs`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `getResultTypes`, `getResult`, `getOutputs`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 76-82
```cpp
/// Pattern to replace
///
///   linalg.batch_matmul(a, b)
///
/// with
///
///   linalg.batch_matmul_transpose_a(linalg.transpose(a), b)
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 83-94
```cpp
///
/// Only the non-batch dimensions are transposed. By default the LHS is
/// transposed. Set `transposeLHS=false` to transpose RHS instead.
FailureOr<Operation *>
mlir::linalg::transposeBatchMatmul(RewriterBase &rewriter,
                                   linalg::BatchMatmulOp batchMatmulOp,
                                   bool transposeLHS) {
  if (batchMatmulOp.hasUserDefinedMaps()) {
    return rewriter.notifyMatchFailure(
        batchMatmulOp, "ops with user-defined maps are not supported");
  }

```
- **EN**: Implements logic around `transposeBatchMatmul`, `hasUserDefinedMaps`, `notifyMatchFailure`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `transposeBatchMatmul`, `hasUserDefinedMaps`, `notifyMatchFailure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 95-102
```cpp
  if (!batchMatmulOp.hasPureTensorSemantics())
    return rewriter.notifyMatchFailure(
        batchMatmulOp, "only matmul ops with tensors are supported");

  Location loc = batchMatmulOp.getLoc();
  Value input = batchMatmulOp.getInputs()[transposeLHS ? 0 : 1];
  auto type = cast<ShapedType>(input.getType());

```
- **EN**: Implements logic around `hasPureTensorSemantics`, `notifyMatchFailure`, `getLoc`, `getInputs`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `hasPureTensorSemantics`, `notifyMatchFailure`, `getLoc`, `getInputs`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 103-110
```cpp
  SmallVector<Value> dynamicDims;
  if (type.isDynamicDim(0))
    dynamicDims.push_back(tensor::DimOp::create(rewriter, loc, input, 0));
  if (type.isDynamicDim(2))
    dynamicDims.push_back(tensor::DimOp::create(rewriter, loc, input, 2));
  if (type.isDynamicDim(1))
    dynamicDims.push_back(tensor::DimOp::create(rewriter, loc, input, 1));

```
- **EN**: Implements logic around `isDynamicDim`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isDynamicDim`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 111-124
```cpp
  ArrayRef<int64_t> shape = type.getShape();
  Value empty = tensor::EmptyOp::create(
      rewriter, loc, ArrayRef<int64_t>{shape[0], shape[2], shape[1]},
      type.getElementType(), dynamicDims);
  auto transposeOp = linalg::TransposeOp::create(rewriter, loc, input, empty,
                                                 ArrayRef<int64_t>{0, 2, 1});
  Operation *newMatmulOp;
  if (transposeLHS) {
    newMatmulOp = BatchMatmulTransposeAOp::create(
        rewriter, loc, batchMatmulOp.getResultTypes(),
        ValueRange{transposeOp->getResult(0), batchMatmulOp.getInputs()[1]},
        batchMatmulOp.getOutputs());
  } else {
    newMatmulOp = BatchMatmulTransposeBOp::create(
```
- **EN**: Implements logic around `getShape`, `create`, `getElementType`, `getResultTypes`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getShape`, `create`, `getElementType`, `getResultTypes`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 125-132
```cpp
        rewriter, loc, batchMatmulOp.getResultTypes(),
        ValueRange{batchMatmulOp.getInputs()[0], transposeOp->getResult(0)},
        batchMatmulOp.getOutputs());
  }
  rewriter.replaceOp(batchMatmulOp, newMatmulOp);
  return newMatmulOp;
}

```
- **EN**: Implements logic around `getResultTypes`, `getInputs`, `getOutputs`, `replaceOp`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getResultTypes`, `getInputs`, `getOutputs`, `replaceOp` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 133-145
```cpp
namespace {
struct TransposeMatmul final : public OpRewritePattern<linalg::MatmulOp> {
  TransposeMatmul(MLIRContext *ctx, bool transposeLHS)
      : OpRewritePattern(ctx), transposeLHS(transposeLHS) {}

  LogicalResult matchAndRewrite(linalg::MatmulOp op,
                                PatternRewriter &rewriter) const override {
    if (failed(transposeMatmul(rewriter, op, transposeLHS))) {
      return failure();
    }
    return success();
  }

```
- **EN**: Introduces declarations for `TransposeMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `TransposeMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 146-154
```cpp
private:
  bool transposeLHS;
};

struct TransposeBatchMatmul final
    : public OpRewritePattern<linalg::BatchMatmulOp> {
  TransposeBatchMatmul(MLIRContext *ctx, bool transposeLHS)
      : OpRewritePattern(ctx), transposeLHS(transposeLHS) {}

```
- **EN**: Introduces declarations for `TransposeBatchMatmul`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `TransposeBatchMatmul` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 155-162
```cpp
  LogicalResult matchAndRewrite(linalg::BatchMatmulOp op,
                                PatternRewriter &rewriter) const override {
    if (failed(transposeBatchMatmul(rewriter, op, transposeLHS))) {
      return failure();
    }
    return success();
  }

```
- **EN**: Implements logic around `matchAndRewrite`, `failed`, `failure`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `matchAndRewrite`, `failed`, `failure`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 163-172
```cpp
private:
  bool transposeLHS;
};
} // namespace

void mlir::linalg::populateTransposeMatmulPatterns(RewritePatternSet &patterns,
                                                   bool transposeLHS) {
  patterns.add<TransposeMatmul, TransposeBatchMatmul>(patterns.getContext(),
                                                      transposeLHS);
}
```
- **EN**: Implements logic around `populateTransposeMatmulPatterns`, `TransposeBatchMatmul>`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateTransposeMatmulPatterns`, `TransposeBatchMatmul>` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (1), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
