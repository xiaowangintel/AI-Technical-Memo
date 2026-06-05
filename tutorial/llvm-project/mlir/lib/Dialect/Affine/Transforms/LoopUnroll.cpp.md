# LoopUnroll.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/LoopUnroll.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements loop unrolling.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopUnroll.cpp - Code to perform loop unrolling --------------------===//
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
// This file implements loop unrolling.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 13-20
```cpp
#include "mlir/Dialect/Affine/Transforms/Passes.h"

#include "mlir/Dialect/Affine/Analysis/LoopAnalysis.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "llvm/Support/CommandLine.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`。

### Lines 21-27
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINELOOPUNROLL
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 28-32
```cpp
#define DEBUG_TYPE "affine-loop-unroll"

using namespace mlir;
using namespace mlir::affine;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or generated-code switches.
- **CN**: 定义受预处理器控制的结构、特性开关或生成代码开关。

### Lines 33-37
```cpp
namespace {

// TODO: this is really a test pass and should be moved out of dialect
// transforms.

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 38-42
```cpp
/// Loop unrolling pass. Unrolls all innermost loops unless full unrolling and a
/// full unroll threshold was specified, in which case, fully unrolls all loops
/// with trip count less than the specified threshold. The latter is for testing
/// purposes, especially for testing outer loop unrolling.
struct LoopUnroll : public affine::impl::AffineLoopUnrollBase<LoopUnroll> {
```
- **EN**: Introduces declarations for `LoopUnroll`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LoopUnroll` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 43-52
```cpp
  // Callback to obtain unroll factors; if this has a callable target, takes
  // precedence over command-line argument or passed argument.
  const std::function<unsigned(AffineForOp)> getUnrollFactor;

  LoopUnroll() : getUnrollFactor(nullptr) {}
  LoopUnroll(const LoopUnroll &other) = default;
  explicit LoopUnroll(
      std::optional<unsigned> unrollFactor = std::nullopt,
      bool unrollUpToFactor = false,
      const std::function<unsigned(AffineForOp)> &getUnrollFactor = nullptr)
```
- **EN**: Implements logic around `function`, `LoopUnroll`; this block works with symbol tables or function-like operations.
- **CN**: 围绕 `function`, `LoopUnroll` 实现具体逻辑；该代码块处理符号表或类函数操作。

### Lines 53-58
```cpp
      : getUnrollFactor(getUnrollFactor) {
    if (unrollFactor)
      this->unrollFactor = *unrollFactor;
    this->unrollUpToFactor = unrollUpToFactor;
  }

```
- **EN**: Implements logic around `getUnrollFactor`.
- **CN**: 围绕 `getUnrollFactor` 实现具体逻辑。

### Lines 59-65
```cpp
  void runOnOperation() override;

  /// Unroll this for op. Returns failure if nothing was done.
  LogicalResult runOnAffineForOp(AffineForOp forOp);
};
} // namespace

```
- **EN**: Implements logic around `runOnOperation`, `runOnAffineForOp`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `runOnAffineForOp` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 66-74
```cpp
/// Returns true if no other affine.for ops are nested within `op`.
static bool isInnermostAffineForOp(AffineForOp op) {
  return !op.getBody()
              ->walk([&](AffineForOp nestedForOp) {
                return WalkResult::interrupt();
              })
              .wasInterrupted();
}

```
- **EN**: Implements logic around `isInnermostAffineForOp`, `getBody`, `walk`, `interrupt`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `isInnermostAffineForOp`, `getBody`, `walk`, `interrupt`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 75-83
```cpp
/// Gathers loops that have no affine.for's nested within.
static void gatherInnermostLoops(FunctionOpInterface f,
                                 SmallVectorImpl<AffineForOp> &loops) {
  f.walk([&](AffineForOp forOp) {
    if (isInnermostAffineForOp(forOp))
      loops.push_back(forOp);
  });
}

```
- **EN**: Implements logic around `gatherInnermostLoops`, `walk`, `isInnermostAffineForOp`, `push_back`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `gatherInnermostLoops`, `walk`, `isInnermostAffineForOp`, `push_back` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 84-93
```cpp
void LoopUnroll::runOnOperation() {
  if (!(unrollFactor.getValue() > 0 || unrollFactor.getValue() == -1)) {
    emitError(UnknownLoc::get(&getContext()),
              "Invalid option: 'unroll-factor' should be greater than 0 or "
              "equal to -1");
    return signalPassFailure();
  }
  FunctionOpInterface func = getOperation();
  if (func.isExternal())
    return;
```
- **EN**: Implements logic around `runOnOperation`, `getValue`, `emitError`, `signalPassFailure`, and 2 more symbols; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getValue`, `emitError`, `signalPassFailure`, and 2 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 94-98
```cpp

  if (unrollFactor.getValue() == -1 && unrollFullThreshold.hasValue()) {
    // Store short loops as we walk.
    SmallVector<AffineForOp, 4> loops;

```
- **EN**: Implements logic around `getValue`.
- **CN**: 围绕 `getValue` 实现具体逻辑。

### Lines 99-108
```cpp
    // Gathers all loops with trip count <= minTripCount. Do a post order walk
    // so that loops are gathered from innermost to outermost (or else
    // unrolling an outer one may delete gathered inner ones).
    getOperation().walk([&](AffineForOp forOp) {
      std::optional<uint64_t> tripCount = getConstantTripCount(forOp);
      if (tripCount && *tripCount <= unrollFullThreshold)
        loops.push_back(forOp);
    });
    for (auto forOp : loops)
      (void)loopUnrollFull(forOp);
```
- **EN**: Implements logic around `getOperation`, `getConstantTripCount`, `push_back`, `loopUnrollFull`.
- **CN**: 围绕 `getOperation`, `getConstantTripCount`, `push_back`, `loopUnrollFull` 实现具体逻辑。

### Lines 109-118
```cpp
    return;
  }

  // If the call back is provided, we will recurse until no loops are found.
  SmallVector<AffineForOp, 4> loops;
  for (unsigned i = 0; i < numRepetitions || getUnrollFactor; i++) {
    loops.clear();
    gatherInnermostLoops(func, loops);
    if (loops.empty())
      break;
```
- **EN**: Implements logic around `clear`, `gatherInnermostLoops`, `empty`.
- **CN**: 围绕 `clear`, `gatherInnermostLoops`, `empty` 实现具体逻辑。

### Lines 119-127
```cpp
    bool unrolled = false;
    for (auto forOp : loops)
      unrolled |= succeeded(runOnAffineForOp(forOp));
    if (!unrolled)
      // Break out if nothing was unrolled.
      break;
  }
}

```
- **EN**: Implements logic around `succeeded`.
- **CN**: 围绕 `succeeded` 实现具体逻辑。

### Lines 128-137
```cpp
/// Unrolls a 'affine.for' op. Returns success if the loop was unrolled,
/// failure otherwise. The default unroll factor is 4.
LogicalResult LoopUnroll::runOnAffineForOp(AffineForOp forOp) {
  // Use the function callback if one was provided.
  if (getUnrollFactor)
    return loopUnrollByFactor(forOp, getUnrollFactor(forOp),
                              /*annotateFn=*/nullptr, cleanUpUnroll);
  // Unroll completely if full loop unroll was specified.
  if (unrollFactor.getValue() == -1)
    return loopUnrollFull(forOp);
```
- **EN**: Implements logic around `runOnAffineForOp`, `loopUnrollByFactor`, `getValue`, `loopUnrollFull`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `runOnAffineForOp`, `loopUnrollByFactor`, `getValue`, `loopUnrollFull` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 138-144
```cpp
  // Otherwise, unroll by the given unroll factor.
  if (unrollUpToFactor)
    return loopUnrollUpToFactor(forOp, unrollFactor);
  return loopUnrollByFactor(forOp, unrollFactor, /*annotateFn=*/nullptr,
                            cleanUpUnroll);
}

```
- **EN**: Implements logic around `loopUnrollUpToFactor`, `loopUnrollByFactor`.
- **CN**: 围绕 `loopUnrollUpToFactor`, `loopUnrollByFactor` 实现具体逻辑。

### Lines 145-152
```cpp
std::unique_ptr<InterfacePass<FunctionOpInterface>>
mlir::affine::createLoopUnrollPass(
    int unrollFactor, bool unrollUpToFactor,
    const std::function<unsigned(AffineForOp)> &getUnrollFactor) {
  return std::make_unique<LoopUnroll>(
      unrollFactor == -1 ? std::nullopt : std::optional<unsigned>(unrollFactor),
      unrollUpToFactor, getUnrollFactor);
}
```
- **EN**: Implements logic around `createLoopUnrollPass`, `function`, `make_unique`, `optional`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `createLoopUnrollPass`, `function`, `make_unique`, `optional` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `llvm/Support/CommandLine.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
