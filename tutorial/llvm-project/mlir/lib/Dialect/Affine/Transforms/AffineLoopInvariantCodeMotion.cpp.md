# AffineLoopInvariantCodeMotion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/Affine/Transforms/AffineLoopInvariantCodeMotion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements loop invariant code motion.
  - **CN**: 实现 Affine 方言与仿射循环推理 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AffineLoopInvariantCodeMotion.cpp - Code to perform loop fusion-----===//
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
// This file implements loop invariant code motion.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/Affine/Transforms/Passes.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Transforms/Passes.h`。

### Lines 15-21
```cpp
#include "mlir/Dialect/Affine/Analysis/LoopAnalysis.h"
#include "mlir/Dialect/Affine/Analysis/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"

namespace mlir {
namespace affine {
```
- **EN**: Introduces declarations for `mlir`, `affine`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `affine` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 22-28
```cpp
#define GEN_PASS_DEF_AFFINELOOPINVARIANTCODEMOTION
#include "mlir/Dialect/Affine/Transforms/Passes.h.inc"
} // namespace affine
} // namespace mlir

#define DEBUG_TYPE "affine-licm"

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 29-35
```cpp
using namespace mlir;
using namespace mlir::affine;

namespace {

/// Affine loop invariant code motion (LICM) pass.
/// TODO: When compared to the other standard LICM pass, this pass
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 36-48
```cpp
/// has some special handling for affine read/write ops but such handling
/// requires aliasing to be sound, and as such this pass is unsound. In
/// addition, this handling is nothing particular to affine memory ops but would
/// apply to any memory read/write effect ops. Either aliasing should be handled
/// or this pass can be removed and the standard LICM can be used.
struct LoopInvariantCodeMotion
    : public affine::impl::AffineLoopInvariantCodeMotionBase<
          LoopInvariantCodeMotion> {
  void runOnOperation() override;
  void runOnAffineForOp(AffineForOp forOp);
};
} // namespace

```
- **EN**: Introduces declarations for `LoopInvariantCodeMotion`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LoopInvariantCodeMotion` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 49-56
```cpp
static bool
checkInvarianceOfNestedIfOps(AffineIfOp ifOp, AffineForOp loop,
                             SmallPtrSetImpl<Operation *> &opsWithUsers,
                             SmallPtrSetImpl<Operation *> &opsToHoist);
static bool isOpLoopInvariant(Operation &op, AffineForOp loop,
                              SmallPtrSetImpl<Operation *> &opsWithUsers,
                              SmallPtrSetImpl<Operation *> &opsToHoist);

```
- **EN**: Implements logic around `checkInvarianceOfNestedIfOps`, `isOpLoopInvariant`.
- **CN**: 围绕 `checkInvarianceOfNestedIfOps`, `isOpLoopInvariant` 实现具体逻辑。

### Lines 57-67
```cpp
static bool
areAllOpsInTheBlockListInvariant(Region &blockList, AffineForOp loop,
                                 SmallPtrSetImpl<Operation *> &opsWithUsers,
                                 SmallPtrSetImpl<Operation *> &opsToHoist);

/// Returns true if `op` is invariant on `loop`.
static bool isOpLoopInvariant(Operation &op, AffineForOp loop,
                              SmallPtrSetImpl<Operation *> &opsWithUsers,
                              SmallPtrSetImpl<Operation *> &opsToHoist) {
  Value iv = loop.getInductionVar();

```
- **EN**: Implements logic around `areAllOpsInTheBlockListInvariant`, `isOpLoopInvariant`, `getInductionVar`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `areAllOpsInTheBlockListInvariant`, `isOpLoopInvariant`, `getInductionVar` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 68-81
```cpp
  if (auto ifOp = dyn_cast<AffineIfOp>(op)) {
    if (!checkInvarianceOfNestedIfOps(ifOp, loop, opsWithUsers, opsToHoist))
      return false;
  } else if (auto forOp = dyn_cast<AffineForOp>(op)) {
    if (!areAllOpsInTheBlockListInvariant(forOp.getRegion(), loop, opsWithUsers,
                                          opsToHoist))
      return false;
  } else if (auto parOp = dyn_cast<AffineParallelOp>(op)) {
    if (!areAllOpsInTheBlockListInvariant(parOp.getRegion(), loop, opsWithUsers,
                                          opsToHoist))
      return false;
  } else if (!isMemoryEffectFree(&op) &&
             !isa<AffineReadOpInterface, AffineWriteOpInterface>(&op)) {
    // Check for side-effecting ops. Affine read/write ops are handled
```
- **EN**: Implements logic around `dyn_cast`, `checkInvarianceOfNestedIfOps`, `areAllOpsInTheBlockListInvariant`, `isMemoryEffectFree`, and 1 more symbols; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `dyn_cast`, `checkInvarianceOfNestedIfOps`, `areAllOpsInTheBlockListInvariant`, `isMemoryEffectFree`, and 1 more symbols 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 82-95
```cpp
    // separately below.
    return false;
  } else if (isa<AffineReadOpInterface, AffineWriteOpInterface>(op)) {
    // Register op in the set of ops that have users.
    opsWithUsers.insert(&op);
    SmallVector<AffineForOp, 8> userIVs;
    auto read = dyn_cast<AffineReadOpInterface>(op);
    Value memref =
        read ? read.getMemRef() : cast<AffineWriteOpInterface>(op).getMemRef();
    for (auto *user : memref.getUsers()) {
      // If the memref used by the load/store is used in a store elsewhere in
      // the loop nest, we do not hoist. Similarly, if the memref used in a
      // load is also being stored too, we do not hoist the load.
      // FIXME: This is missing checking aliases.
```
- **EN**: Implements logic around `AffineWriteOpInterface>`, `insert`, `dyn_cast`, `getMemRef`, and 1 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `AffineWriteOpInterface>`, `insert`, `dyn_cast`, `getMemRef`, and 1 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 96-109
```cpp
      if (&op == user)
        continue;
      if (hasEffect<MemoryEffects::Write>(user, memref) ||
          (hasEffect<MemoryEffects::Read>(user, memref) &&
           isa<AffineWriteOpInterface>(op))) {
        userIVs.clear();
        getAffineForIVs(*user, &userIVs);
        // Check that userIVs don't contain the for loop around the op.
        if (llvm::is_contained(userIVs, loop))
          return false;
      }
    }
  }

```
- **EN**: Implements logic around `Write>`, `Read>`, `isa`, `clear`, and 2 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `Write>`, `Read>`, `isa`, `clear`, and 2 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 110-118
```cpp
  // Check operands.
  ValueRange iterArgs = loop.getRegionIterArgs();
  for (unsigned int i = 0; i < op.getNumOperands(); ++i) {
    auto *operandSrc = op.getOperand(i).getDefiningOp();

    // If the loop IV is the operand, this op isn't loop invariant.
    if (iv == op.getOperand(i))
      return false;

```
- **EN**: Implements logic around `getRegionIterArgs`, `getNumOperands`, `getOperand`.
- **CN**: 围绕 `getRegionIterArgs`, `getNumOperands`, `getOperand` 实现具体逻辑。

### Lines 119-131
```cpp
    // If the one of the iter_args is the operand, this op isn't loop invariant.
    if (llvm::is_contained(iterArgs, op.getOperand(i)))
      return false;

    if (operandSrc) {
      // If the value was defined in the loop (outside of the if/else region),
      // and that operation itself wasn't meant to be hoisted, then mark this
      // operation loop dependent.
      if (opsWithUsers.count(operandSrc) && opsToHoist.count(operandSrc) == 0)
        return false;
    }
  }

```
- **EN**: Implements logic around `is_contained`, `count`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `is_contained`, `count` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 132-142
```cpp
  // If no operand was loop variant, mark this op for motion.
  opsToHoist.insert(&op);
  return true;
}

// Checks if all ops in a region (i.e. list of blocks) are loop invariant.
static bool
areAllOpsInTheBlockListInvariant(Region &blockList, AffineForOp loop,
                                 SmallPtrSetImpl<Operation *> &opsWithUsers,
                                 SmallPtrSetImpl<Operation *> &opsToHoist) {

```
- **EN**: Implements logic around `insert`, `areAllOpsInTheBlockListInvariant`; this block manipulates MLIR regions, blocks, or control-flow edges.
- **CN**: 围绕 `insert`, `areAllOpsInTheBlockListInvariant` 实现具体逻辑；该代码块处理 MLIR region、block 或控制流边。

### Lines 143-149
```cpp
  for (auto &b : blockList) {
    for (auto &op : b) {
      if (!isOpLoopInvariant(op, loop, opsWithUsers, opsToHoist))
        return false;
    }
  }

```
- **EN**: Implements logic around `isOpLoopInvariant`.
- **CN**: 围绕 `isOpLoopInvariant` 实现具体逻辑。

### Lines 150-161
```cpp
  return true;
}

// Returns true if the affine.if op can be hoisted.
static bool
checkInvarianceOfNestedIfOps(AffineIfOp ifOp, AffineForOp loop,
                             SmallPtrSetImpl<Operation *> &opsWithUsers,
                             SmallPtrSetImpl<Operation *> &opsToHoist) {
  if (!areAllOpsInTheBlockListInvariant(ifOp.getThenRegion(), loop,
                                        opsWithUsers, opsToHoist))
    return false;

```
- **EN**: Implements logic around `checkInvarianceOfNestedIfOps`, `areAllOpsInTheBlockListInvariant`; this block performs affine reasoning or shape/bounds manipulation.
- **CN**: 围绕 `checkInvarianceOfNestedIfOps`, `areAllOpsInTheBlockListInvariant` 实现具体逻辑；该代码块执行仿射推理或形状/边界处理。

### Lines 162-168
```cpp
  if (!areAllOpsInTheBlockListInvariant(ifOp.getElseRegion(), loop,
                                        opsWithUsers, opsToHoist))
    return false;

  return true;
}

```
- **EN**: Implements logic around `areAllOpsInTheBlockListInvariant`.
- **CN**: 围绕 `areAllOpsInTheBlockListInvariant` 实现具体逻辑。

### Lines 169-176
```cpp
void LoopInvariantCodeMotion::runOnAffineForOp(AffineForOp forOp) {
  // This is the place where hoisted instructions would reside.
  OpBuilder b(forOp.getOperation());

  SmallPtrSet<Operation *, 8> opsToHoist;
  SmallVector<Operation *, 8> opsToMove;
  SmallPtrSet<Operation *, 8> opsWithUsers;

```
- **EN**: Implements logic around `runOnAffineForOp`, `b`.
- **CN**: 围绕 `runOnAffineForOp`, `b` 实现具体逻辑。

### Lines 177-183
```cpp
  // Only hoist side-effectful ops when the loop is statically known to execute
  // at least once. For unknown (dynamic) or zero trip counts we cannot prove
  // the body executes, so hoisting a side-effectful op would change observable
  // program semantics. Pure (side-effect-free) ops may always be hoisted.
  auto tripCount = getConstantTripCount(forOp);
  bool guaranteedToExecute = tripCount.has_value() && *tripCount > 0;

```
- **EN**: Implements logic around `getConstantTripCount`, `has_value`.
- **CN**: 围绕 `getConstantTripCount`, `has_value` 实现具体逻辑。

### Lines 184-197
```cpp
  for (Operation &op : *forOp.getBody()) {
    // Register op in the set of ops that have users. This set is used
    // to prevent hoisting ops that depend on these ops that are
    // not being hoisted.
    if (!op.use_empty())
      opsWithUsers.insert(&op);
    if (!isa<AffineYieldOp>(op)) {
      // Do not hoist ops with side effects unless the loop is guaranteed to
      // execute at least once.
      if (!guaranteedToExecute && !isPure(&op))
        continue;
      if (isOpLoopInvariant(op, forOp, opsWithUsers, opsToHoist)) {
        opsToMove.push_back(&op);
      }
```
- **EN**: Implements logic around `getBody`, `use_empty`, `insert`, `isa`, and 3 more symbols.
- **CN**: 围绕 `getBody`, `use_empty`, `insert`, `isa`, and 3 more symbols 实现具体逻辑。

### Lines 198-207
```cpp
    }
  }

  // For all instructions that we found to be invariant, place sequentially
  // right before the for loop.
  for (auto *op : opsToMove) {
    op->moveBefore(forOp);
  }
}

```
- **EN**: Implements logic around `moveBefore`.
- **CN**: 围绕 `moveBefore` 实现具体逻辑。

### Lines 208-214
```cpp
void LoopInvariantCodeMotion::runOnOperation() {
  // Walk through all loops in a function in innermost-loop-first order.  This
  // way, we first LICM from the inner loop, and place the ops in
  // the outer loop, which in turn can be further LICM'ed.
  getOperation().walk([&](AffineForOp op) { runOnAffineForOp(op); });
}

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getOperation` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 215-218
```cpp
std::unique_ptr<OperationPass<func::FuncOp>>
mlir::affine::createAffineLoopInvariantCodeMotionPass() {
  return std::make_unique<LoopInvariantCodeMotion>();
}
```
- **EN**: Implements logic around `createAffineLoopInvariantCodeMotionPass`, `make_unique`; this block participates in pass execution or pass construction; performs affine reasoning or shape/bounds manipulation; works with symbol tables or function-like operations.
- **CN**: 围绕 `createAffineLoopInvariantCodeMotionPass`, `make_unique` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并执行仿射推理或形状/边界处理，并处理符号表或类函数操作。

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
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Transforms/Passes.h`, `mlir/Dialect/Affine/Analysis/LoopAnalysis.h`, `mlir/Dialect/Affine/Analysis/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Interfaces/SideEffectInterfaces.h`, `mlir/Dialect/Affine/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (5), MLIR interface declarations / MLIR 接口声明 (1)
