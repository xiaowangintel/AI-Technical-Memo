# NamedToElementwise.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Linalg/Transforms/NamedToElementwise.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements rewriting those linalg named ops that are essentially elementwise e.g. `linalg.exp`, to `linalg.elementwise`. This allows further optimization on `linalg.elementwise` such as folding transpose, broadcast.
  - **CN**: 该文件位于 `mlir/lib/Dialect/Linalg/Transforms`，围绕 Linalg 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- NamedToElementwise.cpp - convert linalg named op into elementwise --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp
//
// This file implements rewriting those linalg named ops that are essentially
// elementwise e.g. `linalg.exp`, to `linalg.elementwise`. This allows further
// optimization on `linalg.elementwise` such as folding transpose, broadcast.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 14-22
```cpp

#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Passes.h"
#include "mlir/Dialect/Linalg/Transforms/Transforms.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`。

### Lines 23-27
```cpp
using namespace mlir;
using namespace mlir::linalg;

#define DEBUG_TYPE "linalg-named-to-elementwise"

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 28-37
```cpp
namespace {
ElementwiseKind getKind(Operation *op) {
  return llvm::TypeSwitch<Operation *, ElementwiseKind>(op)
      .Case([](SelectOp) { return ElementwiseKind::select; })
      .Case([](AddOp) { return ElementwiseKind::add; })
      .Case([](SubOp) { return ElementwiseKind::sub; })
      .Case([](MulOp) { return ElementwiseKind::mul; })
      .Case([](DivOp) { return ElementwiseKind::div; })
      .Case([](DivUnsignedOp) { return ElementwiseKind::div_unsigned; })
      .Case([](PowFOp) { return ElementwiseKind::powf; })
```
- **EN**: Implements logic around `getKind`, `ElementwiseKind>`, `Case`.
- **CN**: 围绕 `getKind`, `ElementwiseKind>`, `Case` 实现具体逻辑。

### Lines 38-47
```cpp
      .Case([](ExpOp) { return ElementwiseKind::exp; })
      .Case([](LogOp) { return ElementwiseKind::log; })
      .Case([](AbsOp) { return ElementwiseKind::abs; })
      .Case([](CeilOp) { return ElementwiseKind::ceil; })
      .Case([](FloorOp) { return ElementwiseKind::floor; })
      .Case([](NegFOp) { return ElementwiseKind::negf; })
      .Case([](ReciprocalOp) { return ElementwiseKind::reciprocal; })
      .Case([](RoundOp) { return ElementwiseKind::round; })
      .Case([](SqrtOp) { return ElementwiseKind::sqrt; })
      .Case([](RsqrtOp) { return ElementwiseKind::rsqrt; })
```
- **EN**: Implements logic around `Case`.
- **CN**: 围绕 `Case` 实现具体逻辑。

### Lines 48-53
```cpp
      .Case([](SquareOp) { return ElementwiseKind::square; })
      .Case([](TanhOp) { return ElementwiseKind::tanh; })
      .Case([](ErfOp) { return ElementwiseKind::erf; })
      .DefaultUnreachable("unhandled case in named to elementwise");
}

```
- **EN**: Implements logic around `Case`, `DefaultUnreachable`.
- **CN**: 围绕 `Case`, `DefaultUnreachable` 实现具体逻辑。

### Lines 54-63
```cpp
template <typename NamedOpTy>
struct NamedToElementwisePattern : public OpRewritePattern<NamedOpTy> {
  using OpRewritePattern<NamedOpTy>::OpRewritePattern;

  LogicalResult matchAndRewrite(NamedOpTy op,
                                PatternRewriter &rewriter) const override {
    SmallVector<NamedAttribute> attrs;
    auto kindAttr = ElementwiseKindAttr::get(op.getContext(), getKind(op));
    attrs.push_back(rewriter.getNamedAttr("kind", kindAttr));
    attrs.push_back(
```
- **EN**: Introduces declarations for `NamedToElementwisePattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `NamedToElementwisePattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 64-72
```cpp
        rewriter.getNamedAttr("indexing_maps", op.getIndexingMaps()));

    rewriter.replaceOpWithNewOp<ElementwiseOp>(op, op.getDpsInputs(),
                                               op.getDpsInits(), attrs);
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `getNamedAttr`, `replaceOpWithNewOp`, `getDpsInits`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getNamedAttr`, `replaceOpWithNewOp`, `getDpsInits`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 73-82
```cpp
void mlir::linalg::populateLinalgNamedToElementwisePatterns(
    RewritePatternSet &patterns) {
  patterns.add<NamedToElementwisePattern<SelectOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<AddOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<SubOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<MulOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<DivOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<DivUnsignedOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<PowFOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<ExpOp>>(patterns.getContext());
```
- **EN**: Implements logic around `populateLinalgNamedToElementwisePatterns`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateLinalgNamedToElementwisePatterns`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 83-92
```cpp
  patterns.add<NamedToElementwisePattern<LogOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<AbsOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<CeilOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<FloorOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<NegFOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<ReciprocalOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<RoundOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<SqrtOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<RsqrtOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<SquareOp>>(patterns.getContext());
```
- **EN**: Implements logic around `add`.
- **CN**: 围绕 `add` 实现具体逻辑。

### Lines 93-95
```cpp
  patterns.add<NamedToElementwisePattern<TanhOp>>(patterns.getContext());
  patterns.add<NamedToElementwisePattern<ErfOp>>(patterns.getContext());
}
```
- **EN**: Implements logic around `add`.
- **CN**: 围绕 `add` 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Linalg/IR/Linalg.h`, `mlir/Dialect/Linalg/Passes.h`, `mlir/Dialect/Linalg/Transforms/Transforms.h`, `mlir/IR/PatternMatch.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/TypeSwitch.h`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
