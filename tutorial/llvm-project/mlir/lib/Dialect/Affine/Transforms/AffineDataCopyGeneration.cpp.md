# AffineDataCopyGeneration.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/AffineDataCopyGeneration.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to automatically promote accessed memref regions to buffers in a faster memory space that is explicitly managed, with the necessary data movement operations performed through either regular point-wise load/store's or DMAs. Such explicit copying (also referred to as array packing/unpacking in the literature), when done on arrays that exhibit reuse, results in near elimination of conflict misses, TLB misses, reduced use of hardware prefetch streams, and reduced false sharing. It is also necessary for hardware that explicitly managed levels in the memory.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineDataCopyGeneration.cpp - Explicit memref copying pass ------*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-20
```cpp
//
// This file implements a pass to automatically promote accessed memref regions
// to buffers in a faster memory space that is explicitly managed, with the
// necessary data movement operations performed through either regular
// point-wise load/store's or DMAs. Such explicit copying (also referred to as
// array packing/unpacking in the literature), when done on arrays that exhibit
// reuse, results in near elimination of conflict misses, TLB misses, reduced
// use of hardware prefetch streams, and reduced false sharing. It is also
// necessary for hardware that explicitly managed levels in the memory
// hierarchy, and where DMAs may have to be used. This optimization is often
// performed on already tiled code.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 21-34
```cpp

#include "mlir/Dialect/Affine/Transforms/Passes.h"

#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/Support/CommandLine.h"
#include <algorithm>
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`。

### Lines 35-41
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINEDATACOPYGENERATION
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 42-48
```cpp
#define DEBUG_TYPE "affine-data-copy-generate"

using namespace mlir;
using namespace mlir::affine;

namespace {

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 49-55
```cpp
/// Replaces all loads and stores on memref's living in 'slowMemorySpace' by
/// introducing copy operations to transfer data into `fastMemorySpace` and
/// rewriting the original load's/store's to instead load/store from the
/// allocated fast memory buffers. Additional options specify the identifier
/// corresponding to the fast memory space and the amount of fast memory space
/// available. The pass traverses through the nesting structure, recursing to
/// inner levels if necessary to determine at what depth copies need to be
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 56-69
```cpp
/// placed so that the allocated buffers fit within the memory capacity
/// provided.
// TODO: We currently can't generate copies correctly when stores
// are strided. Check for strided stores.
struct AffineDataCopyGeneration
    : public affine::impl::AffineDataCopyGenerationBase<
          AffineDataCopyGeneration> {
  AffineDataCopyGeneration() = default;
  explicit AffineDataCopyGeneration(unsigned slowMemorySpace,
                                    unsigned fastMemorySpace,
                                    unsigned tagMemorySpace,
                                    int minDmaTransferSize,
                                    uint64_t fastMemCapacityBytes) {
    this->slowMemorySpace = slowMemorySpace;
```
- **EN**: Introduces declarations for `AffineDataCopyGeneration`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AffineDataCopyGeneration` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 70-78
```cpp
    this->fastMemorySpace = fastMemorySpace;
    this->tagMemorySpace = tagMemorySpace;
    this->minDmaTransferSize = minDmaTransferSize;
    this->fastMemoryCapacity = fastMemCapacityBytes / 1024;
  }

  void runOnOperation() override;
  void runOnBlock(Block *block, DenseSet<Operation *> &copyNests);

```
- **EN**: Implements logic around `runOnOperation`, `runOnBlock`; this block participates in pass execution or pass construction; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `runOnOperation`, `runOnBlock` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理 MLIR region、block 或控制流边。

### Lines 79-85
```cpp
  // Constant zero index to avoid too many duplicates.
  Value zeroIndex = nullptr;
};

} // namespace

/// Generates copies for memref's living in 'slowMemorySpace' into newly created
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 86-99
```cpp
/// buffers in 'fastMemorySpace', and replaces memory operations to the former
/// by the latter.
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createAffineDataCopyGenerationPass(
    unsigned slowMemorySpace, unsigned fastMemorySpace, unsigned tagMemorySpace,
    int minDmaTransferSize, uint64_t fastMemCapacityBytes) {
  return std::make_unique<AffineDataCopyGeneration>(
      slowMemorySpace, fastMemorySpace, tagMemorySpace, minDmaTransferSize,
      fastMemCapacityBytes);
}
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createAffineDataCopyGenerationPass() {
  return std::make_unique<AffineDataCopyGeneration>();
}
```
- **EN**: Implements logic around `createAffineDataCopyGenerationPass`, `make_unique`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `createAffineDataCopyGenerationPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

### Lines 100-109
```cpp

/// Generate copies for this block. The block is partitioned into separate
/// ranges: each range is either a sequence of one or more operations starting
/// and ending with an affine load or store op, or just an affine.for op (which
/// could have other affine for op's nested within).
void AffineDataCopyGeneration::runOnBlock(Block *block,
                                          DenseSet<Operation *> &copyNests) {
  if (block->empty())
    return;

```
- **EN**: Implements logic around `runOnBlock`, `empty`; this block performs affine reasoning or shape/bounds manipulation; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `runOnBlock`, `empty` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理 MLIR region、block 或控制流边。

### Lines 110-117
```cpp
  uint64_t fastMemCapacityBytes =
      fastMemoryCapacity != std::numeric_limits<uint64_t>::max()
          ? fastMemoryCapacity * 1024
          : fastMemoryCapacity;
  AffineCopyOptions copyOptions = {generateDma, slowMemorySpace,
                                   fastMemorySpace, tagMemorySpace,
                                   fastMemCapacityBytes};

```
- **EN**: Implements logic around `max`.
- **CN**: 围绕 `max` 实现具体逻辑。

### Lines 118-125
```cpp
  // Every affine.for op in the block starts and ends a block range for copying;
  // in addition, a contiguous sequence of operations starting with a
  // load/store op but not including any copy nests themselves is also
  // identified as a copy block range. Straightline code (a contiguous chunk of
  // operations excluding AffineForOp's) are always assumed to not exhaust
  // memory. As a result, this approach is conservative in some cases at the
  // moment; we do a check later and report an error with location info.

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 126-139
```cpp
  // Get to the first load, store, or for op (that is not a copy nest itself).
  auto curBegin = llvm::find_if(*block, [&](Operation &op) {
    return isa<AffineLoadOp, AffineStoreOp, AffineForOp>(op) &&
           copyNests.count(&op) == 0;
  });

  // Create [begin, end) ranges.
  auto it = curBegin;
  while (it != block->end()) {
    AffineForOp forOp;
    // If you hit a non-copy for loop, we will split there.
    if ((forOp = dyn_cast<AffineForOp>(&*it)) && copyNests.count(forOp) == 0) {
      // Perform the copying up unti this 'for' op first.
      (void)affineDataCopyGenerate(/*begin=*/curBegin, /*end=*/it, copyOptions,
```
- **EN**: Implements logic around `find_if`, `AffineForOp>`, `count`, `end`, and 2 more symbols; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `find_if`, `AffineForOp>`, `count`, `end`, and 2 more symbols 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 140-150
```cpp
                                   /*filterMemRef=*/std::nullopt, copyNests);

      // Returns true if the footprint is known to exceed capacity.
      auto exceedsCapacity = [&](AffineForOp forOp) {
        std::optional<int64_t> footprint =
            getMemoryFootprintBytes(forOp,
                                    /*memorySpace=*/0);
        return (footprint.has_value() &&
                static_cast<uint64_t>(*footprint) > fastMemCapacityBytes);
      };

```
- **EN**: Implements logic around `getMemoryFootprintBytes`, `has_value`, `static_cast`.
- **CN**: 围绕 `getMemoryFootprintBytes`, `has_value`, `static_cast` 实现具体逻辑。

### Lines 151-164
```cpp
      // If the memory footprint of the 'affine.for' loop is higher than fast
      // memory capacity (when provided), we recurse to copy at an inner level
      // until we find a depth at which footprint fits in fast mem capacity. If
      // the footprint can't be calculated, we assume for now it fits. Recurse
      // inside if footprint for 'forOp' exceeds capacity, or when
      // skipNonUnitStrideLoops is set and the step size is not one.
      bool recurseInner = skipNonUnitStrideLoops ? forOp.getStep() != 1
                                                 : exceedsCapacity(forOp);
      if (recurseInner) {
        // We'll recurse and do the copies at an inner level for 'forInst'.
        // Recurse onto the body of this loop.
        runOnBlock(forOp.getBody(), copyNests);
      } else {
        // We have enough capacity, i.e., copies will be computed for the
```
- **EN**: Implements logic around `getStep`, `exceedsCapacity`, `runOnBlock`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `getStep`, `exceedsCapacity`, `runOnBlock` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 165-178
```cpp
        // portion of the block until 'it', and for 'it', which is 'forOp'. Note
        // that for the latter, the copies are placed just before this loop (for
        // incoming copies) and right after (for outgoing ones).

        // Inner loop copies have their own scope - we don't thus update
        // consumed capacity. The footprint check above guarantees this inner
        // loop's footprint fits.
        (void)affineDataCopyGenerate(/*begin=*/it, /*end=*/std::next(it),
                                     copyOptions,
                                     /*filterMemRef=*/std::nullopt, copyNests);
      }
      // Get to the next load or store op after 'forOp'.
      curBegin = std::find_if(std::next(it), block->end(), [&](Operation &op) {
        return isa<AffineLoadOp, AffineStoreOp, AffineForOp>(op) &&
```
- **EN**: Implements logic around `affineDataCopyGenerate`, `find_if`, `AffineForOp>`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `affineDataCopyGenerate`, `find_if`, `AffineForOp>` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 179-189
```cpp
               copyNests.count(&op) == 0;
      });
      it = curBegin;
    } else {
      assert(copyNests.count(&*it) == 0 &&
             "all copy nests generated should have been skipped above");
      // We simply include this op in the current range and continue for more.
      ++it;
    }
  }

```
- **EN**: Implements logic around `count`, `assert`.
- **CN**: 围绕 `count`, `assert` 实现具体逻辑。

### Lines 190-201
```cpp
  // Generate the copy for the final block range.
  if (curBegin != block->end()) {
    // Can't be a terminator because it would have been skipped above.
    assert(!curBegin->hasTrait<OpTrait::IsTerminator>() &&
           "can't be a terminator");
    // Exclude the affine.yield - hence, the std::prev.
    (void)affineDataCopyGenerate(/*begin=*/curBegin,
                                 /*end=*/std::prev(block->end()), copyOptions,
                                 /*filterMemRef=*/std::nullopt, copyNests);
  }
}

```
- **EN**: Implements logic around `end`, `assert`, `affineDataCopyGenerate`, `prev`; this block performs affine reasoning or shape/bounds manipulation; manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `end`, `assert`, `affineDataCopyGenerate`, `prev` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理 MLIR region、block 或控制流边。

### Lines 202-210
```cpp
void AffineDataCopyGeneration::runOnOperation() {
  func::FuncOp f = getOperation();
  OpBuilder topBuilder(f.getBody());
  zeroIndex = arith::ConstantIndexOp::create(topBuilder, f.getLoc(), 0);

  // Nests that are copy-in's or copy-out's; the root AffineForOps of those
  // nests are stored herein.
  DenseSet<Operation *> copyNests;

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `topBuilder`, `create`; this block participates in pass execution or pass construction; works with symbol tables or function-like operations.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `topBuilder`, `create` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理符号表或类函数操作。

### Lines 211-224
```cpp
  // Clear recorded copy nests.
  copyNests.clear();

  for (auto &block : f)
    runOnBlock(&block, copyNests);

  // Promote any single iteration loops in the copy nests and collect
  // load/stores to simplify.
  SmallVector<Operation *, 4> copyOps;
  for (Operation *nest : copyNests)
    // With a post order walk, the erasure of loops does not affect
    // continuation of the walk or the collection of load/store ops.
    nest->walk([&](Operation *op) {
      if (auto forOp = dyn_cast<AffineForOp>(op))
```
- **EN**: Implements logic around `clear`, `runOnBlock`, `walk`, `dyn_cast`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `clear`, `runOnBlock`, `walk`, `dyn_cast` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 225-238
```cpp
        (void)promoteIfSingleIteration(forOp);
      else if (isa<AffineLoadOp, AffineStoreOp>(op))
        copyOps.push_back(op);
    });

  // Promoting single iteration loops could lead to simplification of
  // contained load's/store's, and the latter could anyway also be
  // canonicalized.
  RewritePatternSet patterns(&getContext());
  AffineLoadOp::getCanonicalizationPatterns(patterns, &getContext());
  AffineStoreOp::getCanonicalizationPatterns(patterns, &getContext());
  FrozenRewritePatternSet frozenPatterns(std::move(patterns));
  (void)applyOpPatternsGreedily(
      copyOps, frozenPatterns,
```
- **EN**: Implements logic around `promoteIfSingleIteration`, `AffineStoreOp>`, `push_back`, `patterns`, and 3 more symbols; this block registers dialect entities or dialect-level hooks.
- **CN**: 围绕 `promoteIfSingleIteration`, `AffineStoreOp>`, `push_back`, `patterns`, and 3 more symbols 实现具体逻辑；该代码块注册方言实体或方言级钩子。

### Lines 239-241
```cpp
      GreedyRewriteConfig().setStrictness(
          GreedyRewriteStrictness::ExistingAndNewOps));
}
```
- **EN**: Implements logic around `GreedyRewriteConfig`.
- **CN**: 围绕 `GreedyRewriteConfig` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **Affine reasoning / 仿射推理**:
  - **EN**: Works with affine maps, bounds, dependence reasoning, and loop structure.
  - **CN**: 处理仿射映射、边界、依赖推理与循环结构。
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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/Support/CommandLine.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Standard-library headers / 标准库头文件**: `<algorithm>`, `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (8), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1), LLVM support-library helpers / LLVM Support 库辅助功能 (1)
