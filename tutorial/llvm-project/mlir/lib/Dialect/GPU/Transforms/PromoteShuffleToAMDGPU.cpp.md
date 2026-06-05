# PromoteShuffleToAMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Transforms/PromoteShuffleToAMDGPU.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file contains patterns to try to promote `gpu.shuffle`s to specialized AMDGPU intrinsics.
  - **CN**: 实现 GPU 方言与异构加速器支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- PromoteShuffleToAMDGPU.cpp - Promote shuffle to AMDGPU -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-12
```cpp
//
// This file contains patterns to try to promote `gpu.shuffle`s to specialized
// AMDGPU intrinsics.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 13-22
```cpp

#include "mlir/Dialect/AMDGPU/Utils/Chipset.h"
#include "mlir/Dialect/GPU/Transforms/Passes.h"

#include "mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/IR/PatternMatch.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`。

### Lines 23-28
```cpp
using namespace mlir;

namespace {

constexpr amdgpu::Chipset kGfx950 = amdgpu::Chipset(9, 5, 0);

```
- **EN**: Implements logic around `Chipset`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `Chipset` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 29-34
```cpp
/// Try to promote `gpu.shuffle` to `amdgpu.swizzle_bitmode`, width must be 64
/// and offset must be a constant integer in the range [0, 31].
struct PromoteShuffleToSwizzlePattern
    : public OpRewritePattern<gpu::ShuffleOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `PromoteShuffleToSwizzlePattern`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PromoteShuffleToSwizzlePattern` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 35-40
```cpp
  LogicalResult matchAndRewrite(gpu::ShuffleOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getMode() != gpu::ShuffleMode::XOR)
      return rewriter.notifyMatchFailure(op,
                                         "only xor shuffle mode is supported");

```
- **EN**: Implements logic around `matchAndRewrite`, `getMode`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `getMode`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 41-49
```cpp
    if (!isConstantIntValue(op.getWidth(), 64))
      return rewriter.notifyMatchFailure(op,
                                         "only 64 width shuffle is supported");

    std::optional<int64_t> offset = getConstantIntValue(op.getOffset());
    if (!offset)
      return rewriter.notifyMatchFailure(op,
                                         "offset must be a constant integer");

```
- **EN**: Implements logic around `isConstantIntValue`, `notifyMatchFailure`, `getConstantIntValue`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isConstantIntValue`, `notifyMatchFailure`, `getConstantIntValue` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 50-54
```cpp
    int64_t offsetValue = *offset;
    if (offsetValue < 0 || offsetValue >= 32)
      return rewriter.notifyMatchFailure(op,
                                         "offset must be in the range [0, 31]");

```
- **EN**: Implements logic around `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 55-64
```cpp
    Location loc = op.getLoc();
    Value res = amdgpu::SwizzleBitModeOp::create(
        rewriter, loc, op.getResult(0).getType(), op.getValue(),
        /*and_mask=*/31,
        /*orMask=*/0, /*xorMask=*/offsetValue);
    Value valid = arith::ConstantIntOp::create(rewriter, loc, 1, /*width*/ 1);
    rewriter.replaceOp(op, {res, valid});
    return success();
  }
};
```
- **EN**: Implements logic around `getLoc`, `create`, `getResult`, `replaceOp`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `getLoc`, `create`, `getResult`, `replaceOp`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 65-71
```cpp

/// Try to promote `gpu.shuffle` to `amdgpu.permlane_swap`, width must be 64
/// and offset must be a constant integer in the set {16, 32}.
struct PromoteShuffleToPermlanePattern
    : public OpRewritePattern<gpu::ShuffleOp> {
  using OpRewritePattern::OpRewritePattern;

```
- **EN**: Introduces declarations for `PromoteShuffleToPermlanePattern`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `PromoteShuffleToPermlanePattern` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 72-77
```cpp
  LogicalResult matchAndRewrite(gpu::ShuffleOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getMode() != gpu::ShuffleMode::XOR)
      return rewriter.notifyMatchFailure(op,
                                         "only xor shuffle mode is supported");

```
- **EN**: Implements logic around `matchAndRewrite`, `getMode`, `notifyMatchFailure`; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `matchAndRewrite`, `getMode`, `notifyMatchFailure` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 78-86
```cpp
    if (!isConstantIntValue(op.getWidth(), 64))
      return rewriter.notifyMatchFailure(op,
                                         "only 64 width shuffle is supported");

    std::optional<int64_t> offset = getConstantIntValue(op.getOffset());
    if (!offset)
      return rewriter.notifyMatchFailure(op,
                                         "offset must be a constant integer");

```
- **EN**: Implements logic around `isConstantIntValue`, `notifyMatchFailure`, `getConstantIntValue`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `isConstantIntValue`, `notifyMatchFailure`, `getConstantIntValue` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 87-96
```cpp
    int64_t offsetValue = *offset;
    if (offsetValue != 16 && offsetValue != 32)
      return rewriter.notifyMatchFailure(op, "offset must be either 15 or 31");

    Location loc = op.getLoc();
    Value res = amdgpu::PermlaneSwapOp::create(
        rewriter, loc, op.getResult(0).getType(), op.getValue(), offsetValue);
    Value valid = arith::ConstantIntOp::create(rewriter, loc, 1, /*width*/ 1);
    rewriter.replaceOp(op, {res, valid});
    return success();
```
- **EN**: Implements logic around `notifyMatchFailure`, `getLoc`, `create`, `getResult`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `notifyMatchFailure`, `getLoc`, `create`, `getResult`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施，并编码加速器专用执行或 lowering 规则。

### Lines 97-101
```cpp
  }
};

} // namespace

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 102-109
```cpp
void mlir::populateGpuPromoteShuffleToAMDGPUPatterns(
    RewritePatternSet &patterns, std::optional<amdgpu::Chipset> maybeChipset) {
  patterns.add<PromoteShuffleToSwizzlePattern>(patterns.getContext(),
                                               /*benefit*/ 1);
  if (maybeChipset && *maybeChipset >= kGfx950)
    patterns.add<PromoteShuffleToPermlanePattern>(patterns.getContext(),
                                                  /*benefit*/ 2);
}
```
- **EN**: Implements logic around `populateGpuPromoteShuffleToAMDGPUPatterns`, `add`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `populateGpuPromoteShuffleToAMDGPUPatterns`, `add` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/AMDGPU/Utils/Chipset.h`, `mlir/Dialect/GPU/Transforms/Passes.h`, `mlir/Dialect/AMDGPU/IR/AMDGPUDialect.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/IR/PatternMatch.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR IR core abstractions / MLIR IR 核心抽象 (1)
