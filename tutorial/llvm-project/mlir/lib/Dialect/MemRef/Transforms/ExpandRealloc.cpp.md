# ExpandRealloc.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/MemRef/Transforms/ExpandRealloc.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MemRef dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ExpandRealloc`.
  - **CN**: 实现 MemRef 方言中围绕 `ExpandRealloc` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ExpandRealloc.cpp - Expand memref.realloc ops into it's components -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-16
```cpp

#include "mlir/Dialect/MemRef/Transforms/Passes.h"
#include "mlir/Dialect/MemRef/Transforms/Transforms.h"

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
#include "mlir/Transforms/DialectConversion.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MemRef/Transforms/Passes.h`, `mlir/Dialect/MemRef/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MemRef/Transforms/Passes.h`, `mlir/Dialect/MemRef/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`。

### Lines 17-23
```cpp
namespace mlir {
namespace memref {
#define GEN_PASS_DEF_EXPANDREALLOCPASS
#include "mlir/Dialect/MemRef/Transforms/Passes.h.inc"
} // namespace memref
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MemRef/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MemRef/Transforms/Passes.h.inc`。

### Lines 24-30
```cpp
using namespace mlir;

namespace {

/// The `realloc` operation performs a conditional allocation and copy to
/// increase the size of a buffer if necessary. This pattern converts the
/// `realloc` operation into this sequence of simpler operations.
```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 31-37
```cpp

/// Example of an expansion:
/// ```mlir
/// %realloc = memref.realloc %alloc (%size) : memref<?xf32> to memref<?xf32>
/// ```
/// is expanded to
/// ```mlir
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 38-44
```cpp
/// %c0 = arith.constant 0 : index
/// %dim = memref.dim %alloc, %c0 : memref<?xf32>
/// %is_old_smaller = arith.cmpi ult, %dim, %arg1
/// %realloc = scf.if %is_old_smaller -> (memref<?xf32>) {
///   %new_alloc = memref.alloc(%size) : memref<?xf32>
///   %subview = memref.subview %new_alloc[0] [%dim] [1]
///   memref.copy %alloc, %subview
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 45-51
```cpp
///   memref.dealloc %alloc
///   scf.yield %alloc_0 : memref<?xf32>
/// } else {
///   %reinterpret_cast = memref.reinterpret_cast %alloc to
///     offset: [0], sizes: [%size], strides: [1]
///   scf.yield %reinterpret_cast : memref<?xf32>
/// }
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 52-65
```cpp
/// ```
struct ExpandReallocOpPattern : public OpRewritePattern<memref::ReallocOp> {
  ExpandReallocOpPattern(MLIRContext *ctx, bool emitDeallocs)
      : OpRewritePattern(ctx), emitDeallocs(emitDeallocs) {}

  LogicalResult matchAndRewrite(memref::ReallocOp op,
                                PatternRewriter &rewriter) const final {
    Location loc = op.getLoc();
    assert(op.getType().getRank() == 1 &&
           "result MemRef must have exactly one rank");
    assert(op.getSource().getType().getRank() == 1 &&
           "source MemRef must have exactly one rank");
    assert(op.getType().getLayout().isIdentity() &&
           "result MemRef must have identity layout (or none)");
```
- **EN**: Introduces declarations for `ExpandReallocOpPattern`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ExpandReallocOpPattern` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 66-79
```cpp
    assert(op.getSource().getType().getLayout().isIdentity() &&
           "source MemRef must have identity layout (or none)");

    // Get the size of the original buffer.
    int64_t inputSize =
        cast<BaseMemRefType>(op.getSource().getType()).getDimSize(0);
    OpFoldResult currSize = rewriter.getIndexAttr(inputSize);
    if (ShapedType::isDynamic(inputSize)) {
      Value dimZero = getValueOrCreateConstantIndexOp(rewriter, loc,
                                                      rewriter.getIndexAttr(0));
      currSize = memref::DimOp::create(rewriter, loc, op.getSource(), dimZero)
                     .getResult();
    }

```
- **EN**: Implements logic around `assert`, `layout`, `getSource`, `getIndexAttr`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `assert`, `layout`, `getSource`, `getIndexAttr`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 80-86
```cpp
    // Get the requested size that the new buffer should have.
    int64_t outputSize =
        cast<BaseMemRefType>(op.getResult().getType()).getDimSize(0);
    OpFoldResult targetSize = ShapedType::isDynamic(outputSize)
                                  ? OpFoldResult{op.getDynamicResultSize()}
                                  : rewriter.getIndexAttr(outputSize);

```
- **EN**: Implements logic around `getResult`, `isDynamic`, `getDynamicResultSize`, `getIndexAttr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getResult`, `isDynamic`, `getDynamicResultSize`, `getIndexAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 87-100
```cpp
    // Only allocate a new buffer and copy over the values in the old buffer if
    // the old buffer is smaller than the requested size.
    Value lhs = getValueOrCreateConstantIndexOp(rewriter, loc, currSize);
    Value rhs = getValueOrCreateConstantIndexOp(rewriter, loc, targetSize);
    Value cond = arith::CmpIOp::create(rewriter, loc, arith::CmpIPredicate::ult,
                                       lhs, rhs);
    auto ifOp = scf::IfOp::create(
        rewriter, loc, cond,
        [&](OpBuilder &builder, Location loc) {
          // Allocate the new buffer. If it is a dynamic memref we need to pass
          // an additional operand for the size at runtime, otherwise the static
          // size is encoded in the result type.
          SmallVector<Value> dynamicSizeOperands;
          if (op.getDynamicResultSize())
```
- **EN**: Implements logic around `getValueOrCreateConstantIndexOp`, `create`, `getDynamicResultSize`; this block packages logic as an MLIR pass or pass helper; uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getValueOrCreateConstantIndexOp`, `create`, `getDynamicResultSize` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 101-114
```cpp
            dynamicSizeOperands.push_back(op.getDynamicResultSize());

          Value newAlloc = memref::AllocOp::create(
              builder, loc, op.getResult().getType(), dynamicSizeOperands,
              op.getAlignmentAttr());

          // Take a subview of the new (bigger) buffer such that we can copy the
          // old values over (the copy operation requires both operands to have
          // the same shape).
          Value subview = memref::SubViewOp::create(
              builder, loc, newAlloc,
              ArrayRef<OpFoldResult>{rewriter.getIndexAttr(0)},
              ArrayRef<OpFoldResult>{currSize},
              ArrayRef<OpFoldResult>{rewriter.getIndexAttr(1)});
```
- **EN**: Implements logic around `push_back`, `create`, `getResult`, `getAlignmentAttr`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `create`, `getResult`, `getAlignmentAttr`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 115-121
```cpp
          memref::CopyOp::create(builder, loc, op.getSource(), subview);

          // Insert the deallocation of the old buffer only if requested
          // (enabled by default).
          if (emitDeallocs)
            memref::DeallocOp::create(builder, loc, op.getSource());

```
- **EN**: Implements logic around `create`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 122-135
```cpp
          scf::YieldOp::create(builder, loc, newAlloc);
        },
        [&](OpBuilder &builder, Location loc) {
          // We need to reinterpret-cast here because either the input or output
          // type might be static, which means we need to cast from static to
          // dynamic or vice-versa. If both are static and the original buffer
          // is already bigger than the requested size, the cast represents a
          // subview operation.
          Value casted = memref::ReinterpretCastOp::create(
              builder, loc, cast<MemRefType>(op.getResult().getType()),
              op.getSource(), rewriter.getIndexAttr(0),
              ArrayRef<OpFoldResult>{targetSize},
              ArrayRef<OpFoldResult>{rewriter.getIndexAttr(1)});
          scf::YieldOp::create(builder, loc, casted);
```
- **EN**: Implements logic around `create`, `getResult`, `getSource`, `getIndexAttr`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `create`, `getResult`, `getSource`, `getIndexAttr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 136-142
```cpp
        });

    rewriter.replaceOp(op, ifOp.getResult(0));
    return success();
  }

private:
```
- **EN**: Implements logic around `replaceOp`, `success`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `replaceOp`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 143-149
```cpp
  const bool emitDeallocs;
};

struct ExpandReallocPass
    : public memref::impl::ExpandReallocPassBase<ExpandReallocPass> {
  using Base::Base;

```
- **EN**: Introduces declarations for `ExpandReallocPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ExpandReallocPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 150-156
```cpp
  void runOnOperation() override {
    MLIRContext &ctx = getContext();

    RewritePatternSet patterns(&ctx);
    memref::populateExpandReallocPatterns(patterns, emitDeallocs.getValue());
    ConversionTarget target(ctx);

```
- **EN**: Implements logic around `runOnOperation`, `getContext`, `patterns`, `populateExpandReallocPatterns`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper; coordinates dialect conversion or lowering decisions; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `runOnOperation`, `getContext`, `patterns`, `populateExpandReallocPatterns`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并协调方言转换或 lowering 决策，并协调核心结构化 MLIR 方言之间的行为。

### Lines 157-165
```cpp
    target.addLegalDialect<arith::ArithDialect, scf::SCFDialect,
                           memref::MemRefDialect>();
    target.addIllegalOp<memref::ReallocOp>();
    if (failed(applyPartialConversion(getOperation(), target,
                                      std::move(patterns))))
      signalPassFailure();
  }
};

```
- **EN**: Implements logic around `MemRefDialect>`, `ReallocOp>`, `failed`, `move`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `MemRefDialect>`, `ReallocOp>`, `failed`, `move`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 166-171
```cpp
} // namespace

void mlir::memref::populateExpandReallocPatterns(RewritePatternSet &patterns,
                                                 bool emitDeallocs) {
  patterns.add<ExpandReallocOpPattern>(patterns.getContext(), emitDeallocs);
}
```
- **EN**: Implements logic around `populateExpandReallocPatterns`, `add`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `populateExpandReallocPatterns`, `add` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/MemRef/Transforms/Passes.h`, `mlir/Dialect/MemRef/Transforms/Transforms.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/SCF/IR/SCF.h`, `mlir/Transforms/DialectConversion.h`, `mlir/Dialect/MemRef/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
