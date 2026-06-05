# ACCLoopTiling.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCLoopTiling.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass implements the OpenACC loop tiling transformation for acc.loop operations that have the tile clause (OpenACC 3.4 spec, section 2.9.8).
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ACCLoopTiling.cpp - Tile ACC Loops ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-21
```cpp
//
// This pass implements the OpenACC loop tiling transformation for acc.loop
// operations that have the tile clause (OpenACC 3.4 spec, section 2.9.8).
//
// Overview:
// ---------
// The tile clause specifies that the iterations of the associated loops should
// be divided into tiles (rectangular blocks). This pass transforms a single
// or nested acc.loop with tile clauses into a structure of "tile loops"
// (iterating over tiles) containing "element loops" (iterating within tiles).
//
// For example, tiling a 2-level nested loop with tile(T1, T2) produces:
//
//   // Before tiling:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
//   acc.loop tile(T1, T2) control(%i, %j) = (lb1, lb2) to (ub1, ub2) step (s1,
//   s2)
//
//   // After tiling:
//   acc.loop control(%i) = (lb1) to (ub1) step (s1*T1) {      // tile loop 1
//     acc.loop control(%j) = (lb2) to (ub2) step (s2*T2) {    // tile loop 2
//       acc.loop control(%ii) = (%i) to (min(ub1, %i+s1*T1)) step (s1) { //
//       element 1
//         acc.loop control(%jj) = (%j) to (min(ub2, %j+s2*T2)) step (s2) { //
//         element 2
//           // loop body using %ii, %jj
//         }
//       }
//     }
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-49
```cpp
//   }
//
// Gang/worker/vector attributes are distributed as follows:
// - gang: applied to tile loops
// - vector: applied to element loops
// - worker: removed from inner loops
//
// Unknown Tile Sizes:
// -------------------
// The OpenACC tile(*) syntax indicates an implementation-defined tile size.
// In the IR, this is represented as -1. The pass resolves these to the
// default tile size (configurable via pass option).
//
// Requirements:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 50-56
```cpp
// -------------
// 1. The pass uses the OpenACCSupport analysis for remark and NYI (not yet
//    implemented) emission. Custom implementations can be registered via
//    setImplementation() to provide pipeline-specific handling.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 57-68
```cpp
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/OpenACC/OpenACCUtilsTiling.h"
#include "mlir/Dialect/OpenACC/Transforms/Passes.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`。

### Lines 69-75
```cpp
namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCLOOPTILING
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 76-86
```cpp
#define DEBUG_TYPE "acc-loop-tile"

namespace {
using namespace mlir;

struct ACCLoopTilingImpl : public OpRewritePattern<acc::LoopOp> {
  ACCLoopTilingImpl(MLIRContext *context, int32_t defaultTileSize,
                    acc::OpenACCSupport &accSupport)
      : OpRewritePattern<acc::LoopOp>(context),
        defaultTileSize(defaultTileSize), accSupport(accSupport) {}

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 87-96
```cpp
  // Check that tile size types are not narrower than IV types.
  // We only check when both types are IntegerType. For IndexType, the width
  // is target-dependent and the casting utility will handle it correctly.
  LogicalResult checkTileSizeTypes(acc::LoopOp loop,
                                   ArrayRef<Value> tileSizes) const {
    auto ivTypes = loop.getBody().getArgumentTypes();
    for (size_t i = 0; i < tileSizes.size() && i < ivTypes.size(); ++i) {
      Type tileType = tileSizes[i].getType();
      Type ivType = ivTypes[i];

```
- **EN**: Implements logic around `checkTileSizeTypes`, `getBody`, `size`, `getType`.
- **CN**: 围绕 `checkTileSizeTypes`, `getBody`, `size`, `getType` 实现具体逻辑。

### Lines 97-110
```cpp
      // Skip unknown tile sizes (will be created with correct type)
      auto constVal = getConstantIntValue(tileSizes[i]);
      if (constVal && *constVal < 0)
        continue;

      // Only compare when both are integer types (not index)
      auto tileIntType = dyn_cast<IntegerType>(tileType);
      auto ivIntType = dyn_cast<IntegerType>(ivType);
      if (tileIntType && ivIntType) {
        if (tileIntType.getWidth() > ivIntType.getWidth()) {
          accSupport.emitNYI(loop.getLoc(),
                             "tile size type (i" +
                                 std::to_string(tileIntType.getWidth()) +
                                 ") is wider than loop IV type (i" +
```
- **EN**: Implements logic around `getConstantIntValue`, `getWidth`, `emitNYI`, `type`, and 1 more symbols.
- **CN**: 围绕 `getConstantIntValue`, `getWidth`, `emitNYI`, `type`, and 1 more symbols 实现具体逻辑。

### Lines 111-118
```cpp
                                 std::to_string(ivIntType.getWidth()) + ")");
          return failure();
        }
      }
    }
    return success();
  }

```
- **EN**: Implements logic around `to_string`, `failure`, `success`.
- **CN**: 围绕 `to_string`, `failure`, `success` 实现具体逻辑。

### Lines 119-132
```cpp
  void emitTilingRemarks(acc::LoopOp loop, ArrayRef<Value> tileSizes) const {
    // Emit remarks for loop tiling
    accSupport.emitRemark(
        loop,
        [&]() {
          auto getTileSizeStr = [&](Value v) -> std::string {
            std::string name = accSupport.getVariableName(v);
            // Use "*" for unknown tile sizes (represented as -1 or empty)
            if (name.empty() || name == "-1")
              return "*";
            return name;
          };
          SmallVector<std::string> tileStrs;
          for (Value v : tileSizes)
```
- **EN**: Implements logic around `emitTilingRemarks`, `emitRemark`, `getVariableName`, `empty`.
- **CN**: 围绕 `emitTilingRemarks`, `emitRemark`, `getVariableName`, `empty` 实现具体逻辑。

### Lines 133-139
```cpp
            tileStrs.push_back(getTileSizeStr(v));
          return "Tiling " + std::to_string(tileSizes.size()) +
                 "-level loop nest with tile(" + llvm::join(tileStrs, ",") +
                 ")";
        },
        DEBUG_TYPE);

```
- **EN**: Implements logic around `push_back`, `to_string`, `tile`.
- **CN**: 围绕 `push_back`, `to_string`, `tile` 实现具体逻辑。

### Lines 140-153
```cpp
    // Emit remarks for unknown tile sizes that will be resolved to default
    // TODO: Need to base the default tile size on some heuristics.
    for (Value tileSize : tileSizes) {
      std::optional<int64_t> val = getConstantIntValue(tileSize);
      if (val && *val < 0) {
        accSupport.emitRemark(
            loop,
            [&]() {
              return "Picking default tile size " +
                     std::to_string(defaultTileSize) +
                     " for unknown tile size '*'";
            },
            DEBUG_TYPE);
      }
```
- **EN**: Implements logic around `getConstantIntValue`, `emitRemark`, `to_string`.
- **CN**: 围绕 `getConstantIntValue`, `emitRemark`, `to_string` 实现具体逻辑。

### Lines 154-162
```cpp
    }
  }

  LogicalResult matchAndRewrite(acc::LoopOp origLoop,
                                PatternRewriter &rewriter) const override {

    if (origLoop.getTileValues().empty())
      return failure();

```
- **EN**: Implements logic around `matchAndRewrite`, `getTileValues`, `failure`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `matchAndRewrite`, `getTileValues`, `failure` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 163-171
```cpp
    SmallVector<Value> tileSizes(origLoop.getTileValues().begin(),
                                 origLoop.getTileValues().end());
    unsigned tileCount = tileSizes.size();
    unsigned collapseCount = origLoop.getCollapseValue().value_or(1);

    // Sanity check tile size types
    if (failed(checkTileSizeTypes(origLoop, tileSizes)))
      return failure();

```
- **EN**: Implements logic around `tileSizes`, `getTileValues`, `size`, `getCollapseValue`, and 2 more symbols.
- **CN**: 围绕 `tileSizes`, `getTileValues`, `size`, `getCollapseValue`, and 2 more symbols 实现具体逻辑。

### Lines 172-184
```cpp
    // Emit remarks for loop tiling. This is emitted before the original loop
    // is modified. However, it assumes that tiling will not fail.
    emitTilingRemarks(origLoop, tileSizes);

    LLVM_DEBUG(llvm::dbgs() << "\nBefore tiling:\n" << *origLoop << "\n");

    // Clear tile operands from origLoop
    rewriter.startOpModification(origLoop);
    origLoop.getTileOperandsMutable().clear();
    origLoop.removeTileOperandsSegmentsAttr();
    origLoop.removeTileOperandsDeviceTypeAttr();
    rewriter.finalizeOpModification(origLoop);

```
- **EN**: Implements logic around `emitTilingRemarks`, `dbgs`, `startOpModification`, `getTileOperandsMutable`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `emitTilingRemarks`, `dbgs`, `startOpModification`, `getTileOperandsMutable`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 185-196
```cpp
    SmallVector<acc::LoopOp> loopsToTile;
    if (collapseCount < tileCount) {
      // Uncollapse tile loops before tiling if necessary
      loopsToTile =
          acc::uncollapseLoops(origLoop, tileCount, collapseCount, rewriter);
      rewriter.replaceOp(origLoop, loopsToTile[0]);
      LLVM_DEBUG(llvm::dbgs() << "\nAfter uncollapsing:\n"
                              << *loopsToTile[0] << "\n");
    } else {
      loopsToTile.push_back(origLoop);
    }

```
- **EN**: Implements logic around `uncollapseLoops`, `replaceOp`, `dbgs`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `uncollapseLoops`, `replaceOp`, `dbgs`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 197-205
```cpp
    // loopsToTile is a vector of perfectly nested loops. The outermost loop
    // may have multiple IVs but inner loops can only have one IV.
    // The utility handles unknown tile sizes (*) by using `defaultTileSize`.
    acc::tileACCLoops(loopsToTile, tileSizes, defaultTileSize, rewriter);

    LLVM_DEBUG(llvm::dbgs() << "\nAfter tiling:\n " << *loopsToTile[0] << "\n");
    return success();
  }

```
- **EN**: Implements logic around `tileACCLoops`, `dbgs`, `success`; this block uses rewrite-pattern infrastructure to transform operations; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `tileACCLoops`, `dbgs`, `success` 实现具体逻辑；该代码块使用重写模式基础设施变换操作，并协调核心结构化 MLIR 方言之间的行为。

### Lines 206-212
```cpp
private:
  int32_t defaultTileSize;
  acc::OpenACCSupport &accSupport;
};

class ACCLoopTiling : public acc::impl::ACCLoopTilingBase<ACCLoopTiling> {
public:
```
- **EN**: Introduces declarations for `ACCLoopTiling`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ACCLoopTiling` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 213-219
```cpp
  using ACCLoopTilingBase<ACCLoopTiling>::ACCLoopTilingBase;

  void runOnOperation() override {
    func::FuncOp funcOp = getOperation();
    MLIRContext *context = funcOp.getContext();
    acc::OpenACCSupport &accSupport = getAnalysis<acc::OpenACCSupport>();

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `getContext`, `OpenACCSupport>`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `getContext`, `OpenACCSupport>` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 220-228
```cpp
    RewritePatternSet patterns(context);
    patterns.insert<ACCLoopTilingImpl>(context, defaultTileSize, accSupport);
    GreedyRewriteConfig grc;
    grc.setUseTopDownTraversal(true);
    grc.setMaxIterations(1);
    (void)applyPatternsGreedily(funcOp, std::move(patterns), grc);
  }
};

```
- **EN**: Implements logic around `patterns`, `insert`, `setUseTopDownTraversal`, `setMaxIterations`, and 1 more symbols.
- **CN**: 围绕 `patterns`, `insert`, `setUseTopDownTraversal`, `setMaxIterations`, and 1 more symbols 实现具体逻辑。

### Lines 229-229
```cpp
} // namespace
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCUtilsTiling.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/PatternMatch.h`, `mlir/Support/LLVM.h`, `mlir/Transforms/GreedyPatternRewriteDriver.h`, `llvm/ADT/StringExtras.h` ... (+2 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (6), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
