# SimplifyDepthwiseConv.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/SimplifyDepthwiseConv.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements conversions between named ops that can be seens as canonicalizations of named ops.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- NamedOpConversions.cpp - Implements conversions between named ops --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```cpp
//
// This file implements conversions between named ops that can be seens as
// canonicalizations of named ops.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Linalg/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h`。

### Lines 16-22
```cpp
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`。

### Lines 23-30
```cpp
namespace mlir {
#define GEN_PASS_DEF_SIMPLIFYDEPTHWISECONVPASS
#include "mlir/Dialect/Linalg/Passes.h.inc"
} // namespace mlir

using namespace mlir;
using namespace mlir::linalg;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/Passes.h.inc`。

### Lines 31-44
```cpp
static llvm::SmallVector<int64_t> getIndicesVector(int start, int end) {
  return llvm::to_vector<2>(llvm::seq<int64_t>(start, end));
}

static LogicalResult
matchAndReplaceDepthwiseConv(Operation *operation, Value input, Value kernel,
                             Value iZp, Value kZp, Value init, Attribute stride,
                             Attribute dilation, PatternRewriter &rewriter) {
  Location loc = operation->getLoc();
  auto linalgOp = dyn_cast<LinalgOp>(operation);
  // Exit out on the memref version of this operation.
  if (!linalgOp || !linalgOp.hasPureTensorSemantics())
    return failure();

```
- **EN**: Implements logic around `getIndicesVector`, `to_vector`, `matchAndReplaceDepthwiseConv`, `getLoc`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIndicesVector`, `to_vector`, `matchAndReplaceDepthwiseConv`, `getLoc`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 45-52
```cpp
  auto result = operation->getResult(0);

  auto kernelTy = dyn_cast<RankedTensorType>(kernel.getType());
  auto initTy = dyn_cast<RankedTensorType>(init.getType());
  auto resultTy = dyn_cast<RankedTensorType>(result.getType());
  if (!kernelTy || !initTy || !resultTy)
    return failure();

```
- **EN**: Implements logic around `getResult`, `getType`, `failure`.
- **CN**: 围绕 `getResult`, `getType`, `failure` 实现具体逻辑。

### Lines 53-64
```cpp
  if (kernelTy.getDimSize(3) != 1)
    return failure();

  // Collapse kernel dims.
  SmallVector<ReassociationIndices, 4> collapsedKernelDims = {
      getIndicesVector(0, 1), getIndicesVector(1, 2), getIndicesVector(2, 4)};
  auto newKernelTy = RankedTensorType::get(
      {kernelTy.getDimSize(0), kernelTy.getDimSize(1), kernelTy.getDimSize(2)},
      kernelTy.getElementType());
  auto collapsedKernel = tensor::CollapseShapeOp::create(
      rewriter, loc, newKernelTy, kernel, collapsedKernelDims);

```
- **EN**: Implements logic around `getDimSize`, `failure`, `getIndicesVector`, `get`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getDimSize`, `failure`, `getIndicesVector`, `get`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 65-75
```cpp
  // Collapse init dims.
  SmallVector<ReassociationIndices, 4> collapsedInitDims = {
      getIndicesVector(0, 1), getIndicesVector(1, 2), getIndicesVector(2, 3),
      getIndicesVector(3, 5)};
  auto newInitTy =
      RankedTensorType::get({initTy.getDimSize(0), initTy.getDimSize(1),
                             initTy.getDimSize(2), initTy.getDimSize(3)},
                            initTy.getElementType());
  auto collapsedInit = tensor::CollapseShapeOp::create(rewriter, loc, newInitTy,
                                                       init, collapsedInitDims);

```
- **EN**: Implements logic around `getIndicesVector`, `get`, `getDimSize`, `getElementType`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getIndicesVector`, `get`, `getDimSize`, `getElementType`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 76-89
```cpp
  SmallVector<NamedAttribute> preservedAttrs;
  Operation *newConv =
      TypeSwitch<Operation *, Operation *>(operation)
          .Case([&](DepthwiseConv2DNhwcHwcmOp op) {
            preservedAttrs = getPrunedAttributeList(op);
            return DepthwiseConv2DNhwcHwcOp::create(
                rewriter, loc, newInitTy, ValueRange{input, collapsedKernel},
                ValueRange{collapsedInit}, stride, dilation);
          })
          .Case([&](DepthwiseConv2DNhwcHwcmQOp op) {
            preservedAttrs = getPrunedAttributeList(op);
            return DepthwiseConv2DNhwcHwcQOp::create(
                rewriter, loc, newInitTy,
                ValueRange{input, collapsedKernel, iZp, kZp},
```
- **EN**: Implements logic around `Case`, `getPrunedAttributeList`, `create`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `Case`, `getPrunedAttributeList`, `create` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 90-97
```cpp
                ValueRange{collapsedInit}, stride, dilation);
          })
          .Default(nullptr);
  if (!newConv)
    return failure();
  for (auto attr : preservedAttrs)
    newConv->setAttr(attr.getName(), attr.getValue());

```
- **EN**: Implements logic around `Default`, `failure`, `setAttr`.
- **CN**: 围绕 `Default`, `failure`, `setAttr` 实现具体逻辑。

### Lines 98-104
```cpp
  // Expand dimensions back out to
  rewriter.replaceOpWithNewOp<tensor::ExpandShapeOp>(
      operation, resultTy, newConv->getResult(0), collapsedInitDims);
  return success();
}

namespace {
```
- **EN**: Implements logic around `ExpandShapeOp>`, `getResult`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `ExpandShapeOp>`, `getResult`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 105-115
```cpp
struct SimplifyDepthwiseConvOp
    : public OpRewritePattern<DepthwiseConv2DNhwcHwcmOp> {
  using OpRewritePattern<DepthwiseConv2DNhwcHwcmOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(DepthwiseConv2DNhwcHwcmOp op,
                                PatternRewriter &rewriter) const override {
    Operation *operation = op.getOperation();
    Value input = op.getDpsInputOperand(0)->get();
    Value kernel = op.getDpsInputOperand(1)->get();
    Value init = op.getDpsInitOperand(0)->get();

```
- **EN**: Introduces declarations for `SimplifyDepthwiseConvOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SimplifyDepthwiseConvOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 116-124
```cpp
    auto stride = op.getStrides();
    auto dilation = op.getDilations();

    return matchAndReplaceDepthwiseConv(operation, input, kernel, nullptr,
                                        nullptr, init, stride, dilation,
                                        rewriter);
  }
};

```
- **EN**: Implements logic around `getStrides`, `getDilations`, `matchAndReplaceDepthwiseConv`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getStrides`, `getDilations`, `matchAndReplaceDepthwiseConv` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 125-137
```cpp
struct SimplifyDepthwiseConvQOp
    : public OpRewritePattern<DepthwiseConv2DNhwcHwcmQOp> {
  using OpRewritePattern<DepthwiseConv2DNhwcHwcmQOp>::OpRewritePattern;

  LogicalResult matchAndRewrite(DepthwiseConv2DNhwcHwcmQOp op,
                                PatternRewriter &rewriter) const override {
    Operation *operation = op.getOperation();
    Value input = op.getDpsInputOperand(0)->get();
    Value kernel = op.getDpsInputOperand(1)->get();
    Value iZp = op.getDpsInputOperand(2)->get();
    Value kZp = op.getDpsInputOperand(3)->get();
    Value init = op.getDpsInitOperand(0)->get();

```
- **EN**: Introduces declarations for `SimplifyDepthwiseConvQOp`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SimplifyDepthwiseConvQOp` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 138-145
```cpp
    auto stride = op.getStrides();
    auto dilation = op.getDilations();

    return matchAndReplaceDepthwiseConv(operation, input, kernel, iZp, kZp,
                                        init, stride, dilation, rewriter);
  }
};

```
- **EN**: Implements logic around `getStrides`, `getDilations`, `matchAndReplaceDepthwiseConv`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getStrides`, `getDilations`, `matchAndReplaceDepthwiseConv` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 146-159
```cpp
struct SimplifyDepthwiseConvPass
    : public impl::SimplifyDepthwiseConvPassBase<SimplifyDepthwiseConvPass> {
  using impl::SimplifyDepthwiseConvPassBase<
      SimplifyDepthwiseConvPass>::SimplifyDepthwiseConvPassBase;

  void runOnOperation() override {
    Operation *op = getOperation();
    RewritePatternSet patterns(op->getContext());
    populateSimplifyDepthwiseConvPatterns(patterns);
    if (failed(applyPatternsGreedily(op, std::move(patterns))))
      return signalPassFailure();
  }
};
} // namespace
```
- **EN**: Introduces declarations for `SimplifyDepthwiseConvPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `SimplifyDepthwiseConvPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 160-165
```cpp

void mlir::linalg::populateSimplifyDepthwiseConvPatterns(
    RewritePatternSet &patterns) {
  patterns.add<SimplifyDepthwiseConvOp, SimplifyDepthwiseConvQOp>(
      patterns.getContext());
}
```
- **EN**: Implements logic around `populateSimplifyDepthwiseConvPatterns`, `SimplifyDepthwiseConvQOp>`, `getContext`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateSimplifyDepthwiseConvPatterns`, `SimplifyDepthwiseConvQOp>`, `getContext` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pattern rewriting / 模式重写**:
  - **EN**: Uses MLIR rewrite patterns to match operations and replace them with improved forms.
  - **CN**: 使用 MLIR 重写模式匹配操作，并将其替换为更合适的形式。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/TypeSwitch.h`, `mlir/Dialect/Linalg/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
