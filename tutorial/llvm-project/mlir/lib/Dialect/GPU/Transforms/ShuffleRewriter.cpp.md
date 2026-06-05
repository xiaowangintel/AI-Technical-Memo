# ShuffleRewriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/ShuffleRewriter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements in-dialect rewriting of the shuffle op for types i64 and f64, rewriting 64bit shuffles into two 32bit shuffles. This particular implementation using shifts and truncations can be obtained using clang: by emitting IR for shuffle operations with `-O3`.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ShuffleRewriter.cpp - Implementation of shuffle rewriting  ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This file implements in-dialect rewriting of the shuffle op for types i64 and
// f64, rewriting 64bit shuffles into two 32bit shuffles. This particular
// implementation using shifts and truncations can be obtained using clang: by
// emitting IR for shuffle operations with `-O3`.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-21
```cpp

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Transforms/Passes.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/PatternMatch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/IR/Builders.h`。

### Lines 22-27
```cpp
using namespace mlir;

namespace {
struct GpuShuffleRewriter : public OpRewritePattern<gpu::ShuffleOp> {
  using OpRewritePattern<gpu::ShuffleOp>::OpRewritePattern;

```
- **EN**: Introduces declarations for `GpuShuffleRewriter`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `GpuShuffleRewriter` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 28-37
```cpp
  void initialize() {
    // Required as the pattern will replace the Op with 2 additional ShuffleOps.
    setHasBoundedRewriteRecursion();
  }
  LogicalResult matchAndRewrite(gpu::ShuffleOp op,
                                PatternRewriter &rewriter) const override {
    auto loc = op.getLoc();
    auto value = op.getValue();
    auto valueType = value.getType();
    auto valueLoc = value.getLoc();
```
- **EN**: Implements logic around `initialize`, `setHasBoundedRewriteRecursion`, `matchAndRewrite`, `getLoc`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `initialize`, `setHasBoundedRewriteRecursion`, `matchAndRewrite`, `getLoc`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 38-45
```cpp
    auto i32 = rewriter.getI32Type();
    auto i64 = rewriter.getI64Type();

    // If the type of the value is either i32 or f32, the op is already valid.
    if (!valueType.isIntOrFloat() || valueType.getIntOrFloatBitWidth() != 64)
      return rewriter.notifyMatchFailure(
          op, "only 64-bit int/float types are supported");

```
- **EN**: Implements logic around `getI32Type`, `getI64Type`, `isIntOrFloat`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getI32Type`, `getI64Type`, `isIntOrFloat`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 46-51
```cpp
    Value lo, hi;

    // Float types must be converted to i64 to extract the bits.
    if (isa<FloatType>(valueType))
      value = arith::BitcastOp::create(rewriter, valueLoc, i64, value);

```
- **EN**: Implements logic around `isa`, `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isa`, `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 52-60
```cpp
    // Get the low bits by trunc(value).
    lo = arith::TruncIOp::create(rewriter, valueLoc, i32, value);

    // Get the high bits by trunc(value >> 32).
    auto c32 = arith::ConstantOp::create(rewriter, valueLoc,
                                         rewriter.getIntegerAttr(i64, 32));
    hi = arith::ShRUIOp::create(rewriter, valueLoc, value, c32);
    hi = arith::TruncIOp::create(rewriter, valueLoc, i32, hi);

```
- **EN**: Implements logic around `create`, `getIntegerAttr`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `getIntegerAttr` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 61-70
```cpp
    // Shuffle the values.
    ValueRange loRes =
        gpu::ShuffleOp::create(rewriter, op.getLoc(), lo, op.getOffset(),
                               op.getWidth(), op.getMode())
            .getResults();
    ValueRange hiRes =
        gpu::ShuffleOp::create(rewriter, op.getLoc(), hi, op.getOffset(),
                               op.getWidth(), op.getMode())
            .getResults();

```
- **EN**: Implements logic around `create`, `getWidth`, `getResults`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `create`, `getWidth`, `getResults` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 71-77
```cpp
    // Convert lo back to i64.
    lo = arith::ExtUIOp::create(rewriter, valueLoc, i64, loRes[0]);

    // Convert hi back to i64.
    hi = arith::ExtUIOp::create(rewriter, valueLoc, i64, hiRes[0]);
    hi = arith::ShLIOp::create(rewriter, valueLoc, hi, c32);

```
- **EN**: Implements logic around `create`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 78-84
```cpp
    // Obtain the shuffled bits hi | lo.
    value = arith::OrIOp::create(rewriter, loc, hi, lo);

    // Convert the value back to float.
    if (isa<FloatType>(valueType))
      value = arith::BitcastOp::create(rewriter, valueLoc, valueType, value);

```
- **EN**: Implements logic around `create`, `isa`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `isa` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 85-94
```cpp
    // Obtain the shuffle validity by combining both validities.
    auto validity = arith::AndIOp::create(rewriter, loc, loRes[1], hiRes[1]);

    // Replace the op.
    rewriter.replaceOp(op, {value, validity});
    return success();
  }
};
} // namespace

```
- **EN**: Implements logic around `create`, `replaceOp`, `success`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `create`, `replaceOp`, `success` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 95-97
```cpp
void mlir::populateGpuShufflePatterns(RewritePatternSet &patterns) {
  patterns.add<GpuShuffleRewriter>(patterns.getContext());
}
```
- **EN**: Implements logic around `populateGpuShufflePatterns`, `add`.
- **CN**: 围绕 `populateGpuShufflePatterns`, `add` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/IR/Builders.h`, `mlir/IR/PatternMatch.h`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (3), MLIR IR core abstractions / MLIR IR 核心抽象 (2)
