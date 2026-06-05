# LoopTiling.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/LoopTiling.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass to tile affine loop nests.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- LoopTiling.cpp --- Loop tiling pass ------------------------------*-===//
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
// This file implements a pass to tile affine loop nests.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 15-26
```cpp
#include "mlir/Dialect/Affine/Analysis/AffineAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/AffineStructures.h"
#include "mlir/Dialect/Affine/Analysis/LoopAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Affine/IR/AffineOps.h"
#include "mlir/Dialect/Affine/LoopUtils.h"
#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`。

### Lines 27-33
```cpp
namespace mlir {
namespace affine {
#define GEN_PASS_DEF_AFFINELOOPTILING
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 34-40
```cpp
using namespace mlir;
using namespace mlir::affine;

#define DEBUG_TYPE "affine-loop-tile"

namespace {

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 41-48
```cpp
/// A pass to perform loop tiling on all suitable loop nests of a func op.
struct LoopTiling : public affine::impl::AffineLoopTilingBase<LoopTiling> {
  LoopTiling() = default;
  explicit LoopTiling(uint64_t cacheSizeBytes, bool avoidMaxMinBounds = true)
      : avoidMaxMinBounds(avoidMaxMinBounds) {
    this->cacheSizeInKiB = cacheSizeBytes / 1024;
  }

```
- **EN**: Introduces declarations for `LoopTiling`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LoopTiling` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 49-55
```cpp
  void runOnOperation() override;
  void getTileSizes(ArrayRef<AffineForOp> band,
                    SmallVectorImpl<unsigned> *tileSizes);

  // Default tile size if nothing is provided.
  constexpr static unsigned kDefaultTileSize = 4;

```
- **EN**: Implements logic around `runOnOperation`, `getTileSizes`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getTileSizes` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 56-62
```cpp
  // If true, tile sizes are set to avoid max/min in bounds if possible.
  bool avoidMaxMinBounds = true;
};

} // namespace

/// Get bands of loops that are valid to tile from the top-level of `f`.
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 63-75
```cpp
static void
getTopLevelTileableBands(func::FuncOp f,
                         std::vector<SmallVector<AffineForOp, 6>> &bands) {
  // Get maximal perfect nest of 'affine.for' ops starting from root
  // (inclusive).
  for (AffineForOp forOp : f.getOps<AffineForOp>()) {
    SmallVector<AffineForOp, 6> band;
    getPerfectlyNestedLoops(band, forOp);
    if (isTilingValid(band))
      bands.push_back(band);
  }
}

```
- **EN**: Implements logic around `getTopLevelTileableBands`, `getOps`, `getPerfectlyNestedLoops`, `isTilingValid`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `getTopLevelTileableBands`, `getOps`, `getPerfectlyNestedLoops`, `isTilingValid`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

### Lines 76-86
```cpp
/// Creates a pass to perform loop tiling on all suitable loop nests of a
/// Function.
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createLoopTilingPass(uint64_t cacheSizeBytes) {
  return std::make_unique<LoopTiling>(cacheSizeBytes);
}
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createLoopTilingPass() {
  return std::make_unique<LoopTiling>();
}

```
- **EN**: Implements logic around `createLoopTilingPass`, `make_unique`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `createLoopTilingPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

### Lines 87-100
```cpp
/// Reduces each tile size to the largest divisor of the corresponding trip
/// count (if the trip count is known).
static void adjustToDivisorsOfTripCounts(ArrayRef<AffineForOp> band,
                                         SmallVectorImpl<unsigned> *tileSizes) {
  assert(band.size() == tileSizes->size() && "invalid tile size count");
  for (unsigned i = 0, e = band.size(); i < e; i++) {
    unsigned &tSizeAdjusted = (*tileSizes)[i];
    std::optional<uint64_t> mayConst = getConstantTripCount(band[i]);
    if (!mayConst)
      continue;
    // Adjust the tile size to largest factor of the trip count less than
    // tSize.
    uint64_t constTripCount = *mayConst;
    if (constTripCount > 1 && tSizeAdjusted > constTripCount / 2)
```
- **EN**: Implements logic around `adjustToDivisorsOfTripCounts`, `assert`, `size`, `getConstantTripCount`.
- **CN**: 围绕 `adjustToDivisorsOfTripCounts`, `assert`, `size`, `getConstantTripCount` 实现具体逻辑。

### Lines 101-114
```cpp
      tSizeAdjusted = constTripCount / 2;
    while (constTripCount % tSizeAdjusted != 0)
      tSizeAdjusted--;
  }
}

// Returns tile sizes to use. Checks CL options; if none are specified, sets it
// based on a simple model that looks at the memory footprint and determines
// tile sizes assuming identity accesses / 1:1 tile size proportional footprint
// along each of the dimensions being tiled.
// TODO: evolve this model. Tile size determination is a large area
// to play with in general.
void LoopTiling::getTileSizes(ArrayRef<AffineForOp> band,
                              SmallVectorImpl<unsigned> *tileSizes) {
```
- **EN**: Implements logic around `getTileSizes`.
- **CN**: 围绕 `getTileSizes` 实现具体逻辑。

### Lines 115-123
```cpp
  if (band.empty())
    return;

  // Use command-line tileSize for all loops if specified.
  if (tileSize) {
    tileSizes->assign(band.size(), tileSize);
    return;
  }

```
- **EN**: Implements logic around `empty`, `assign`.
- **CN**: 围绕 `empty`, `assign` 实现具体逻辑。

### Lines 124-131
```cpp
  // Use supplied tile sizes and fill them with default tile size if it's short.
  if (!this->tileSizes.empty()) {
    tileSizes->assign(this->tileSizes.begin(), this->tileSizes.end());
    tileSizes->resize(band.size(), kDefaultTileSize);
    return;
  }
  tileSizes->resize(band.size());

```
- **EN**: Implements logic around `empty`, `assign`, `resize`.
- **CN**: 围绕 `empty`, `assign`, `resize` 实现具体逻辑。

### Lines 132-138
```cpp
  // If the cache size is zero, set the minimum valid tile size. No good reason
  // to pick another specific size over this.
  if (cacheSizeInKiB == 0) {
    llvm::fill(*tileSizes, 1);
    return;
  }

```
- **EN**: Implements logic around `fill`.
- **CN**: 围绕 `fill` 实现具体逻辑。

### Lines 139-152
```cpp
  // Obtain memory footprint and set tile sizes so that a tile fits in
  // the cache size. This is an approximation with the assumption that the
  // footprint increases with the tile size linearly in that dimension (i.e.,
  // assumes one-to-one access function).
  std::optional<int64_t> fp = getMemoryFootprintBytes(band[0], 0);
  if (!fp) {
    // Fill with default tile sizes if footprint is unknown.
    llvm::fill(*tileSizes, LoopTiling::kDefaultTileSize);
    if (avoidMaxMinBounds)
      adjustToDivisorsOfTripCounts(band, tileSizes);
    // The first loop in the band.
    AffineForOp rootForOp = band[0];
    (void)rootForOp;
    LLVM_DEBUG(
```
- **EN**: Implements logic around `getMemoryFootprintBytes`, `fill`, `adjustToDivisorsOfTripCounts`.
- **CN**: 围绕 `getMemoryFootprintBytes`, `fill`, `adjustToDivisorsOfTripCounts` 实现具体逻辑。

### Lines 153-166
```cpp
        rootForOp.emitWarning("memory footprint unknown: using default tile "
                              "sizes adjusted to trip count divisors"));
    return;
  }

  // Check how many times larger the cache size is when compared to footprint.
  uint64_t cacheSizeBytes = cacheSizeInKiB * 1024;
  uint64_t excessFactor = llvm::divideCeil(*fp, cacheSizeBytes);
  if (excessFactor <= 1) {
    // No need of any tiling - set tile size to 1.
    llvm::fill(*tileSizes, 1);
    return;
  }

```
- **EN**: Implements logic around `emitWarning`, `divideCeil`, `fill`.
- **CN**: 围绕 `emitWarning`, `divideCeil`, `fill` 实现具体逻辑。

### Lines 167-180
```cpp
  // Divide all loops equally in an attempt to reduce footprint.
  // TODO: this is approximate. Ideally, obtain reuse factor /
  // profitability along each dimension and weight tile sizes based on that as
  // one possible approach. Or compute a polynomial in tile sizes and solve for
  // it.

  // For an n-d tileable band, compute the n^th root of the excess.
  unsigned tSize =
      static_cast<unsigned>(floorl(std::pow(excessFactor, 1.0 / band.size())));
  // We'll keep a running product to determine the last tile size better.
  unsigned cumulProductOfTileSizes = 1;
  for (unsigned i = 0, e = band.size(); i < e; i++) {
    if (i < e - 1)
      (*tileSizes)[i] = tSize;
```
- **EN**: Implements logic around `static_cast`, `size`.
- **CN**: 围绕 `static_cast`, `size` 实现具体逻辑。

### Lines 181-190
```cpp
    else
      // Set last tile size to cover the balance.
      (*tileSizes)[i] = std::max(
          1U, static_cast<unsigned>(excessFactor / cumulProductOfTileSizes));
    cumulProductOfTileSizes *= (*tileSizes)[i];
  }
  if (avoidMaxMinBounds)
    adjustToDivisorsOfTripCounts(band, tileSizes);
}

```
- **EN**: Implements logic around `max`, `static_cast`, `adjustToDivisorsOfTripCounts`.
- **CN**: 围绕 `max`, `static_cast`, `adjustToDivisorsOfTripCounts` 实现具体逻辑。

### Lines 191-204
```cpp
void LoopTiling::runOnOperation() {
  // Bands of loops to tile.
  std::vector<SmallVector<AffineForOp, 6>> bands;
  getTopLevelTileableBands(getOperation(), bands);

  // Tile each band.
  for (auto &band : bands) {
    // Set up tile sizes; fill missing tile sizes at the end with default tile
    // size or tileSize if one was provided.
    SmallVector<unsigned, 6> tileSizes;
    getTileSizes(band, &tileSizes);
    if (llvm::DebugFlag) {
      auto diag = band[0].emitRemark("using tile sizes [");
      llvm::interleaveComma(tileSizes, llvm::dbgs());
```
- **EN**: Implements logic around `runOnOperation`, `getTopLevelTileableBands`, `getTileSizes`, `emitRemark`, and 1 more symbols; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getTopLevelTileableBands`, `getTileSizes`, `emitRemark`, and 1 more symbols 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 205-214
```cpp
      diag << "]\n";
    }
    SmallVector<AffineForOp, 6> tiledNest;
    if (failed(tilePerfectlyNested(band, tileSizes, &tiledNest))) {
      // An empty band always succeeds.
      assert(!band.empty() && "guaranteed to succeed on empty bands");
      LLVM_DEBUG(band.front()->emitRemark("loop tiling failed!\n"));
      continue;
    }

```
- **EN**: Implements logic around `failed`, `assert`, `front`.
- **CN**: 围绕 `failed`, `assert`, `front` 实现具体逻辑。

### Lines 215-227
```cpp
    // Separate full and partial tiles.
    if (separate) {
      auto intraTileLoops =
          MutableArrayRef<AffineForOp>(tiledNest).drop_front(band.size());
      if (failed(separateFullTiles(intraTileLoops))) {
        assert(!intraTileLoops.empty() &&
               "guaranteed to succeed on empty bands");
        LLVM_DEBUG(intraTileLoops.front()->emitRemark(
            "separation post tiling failed!"));
      }
    }
  }
}
```
- **EN**: Implements logic around `MutableArrayRef`, `failed`, `assert`, `front`.
- **CN**: 围绕 `MutableArrayRef`, `failed`, `assert`, `front` 实现具体逻辑。

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
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/AffineAnalysis.h`, `mlir/Dialect/Affine/Analysis/AffineStructures.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Affine/IR/AffineOps.h`, `mlir/Dialect/Affine/LoopUtils.h`, `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `llvm/Support/CommandLine.h` ... (+2 more)
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (10), LLVM support-library helpers / LLVM Support 库辅助功能 (2)
