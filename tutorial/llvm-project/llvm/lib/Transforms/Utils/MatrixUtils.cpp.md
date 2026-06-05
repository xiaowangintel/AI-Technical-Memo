# MatrixUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/MatrixUtils.cpp` | `llvm/lib/Transforms/Utils/MatrixUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utilities to lower matrix intrinsics within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 MatrixUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- MatrixUtils.cpp - Utilities to lower matrix intrinsics ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Utilities for generating tiled loops for matrix operations.
//
//===----------------------------------------------------------------------===//

```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 13-29

```cpp
#include "llvm/Transforms/Utils/MatrixUtils.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/ProfDataUtils.h"
#include "llvm/IR/Type.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;

namespace llvm {
extern cl::opt<bool> ProfcheckDisableMetadataFixes;
} // end namespace llvm

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 30-48

```cpp
BasicBlock *TileInfo::CreateLoop(BasicBlock *Preheader, BasicBlock *Exit,
                                 ConstantInt *Bound, ConstantInt *Step,
                                 StringRef Name, IRBuilderBase &B,
                                 DomTreeUpdater &DTU, Loop *L, LoopInfo &LI) {
  LLVMContext &Ctx = Preheader->getContext();
  BasicBlock *Header = BasicBlock::Create(
      Preheader->getContext(), Name + ".header", Preheader->getParent(), Exit);
  BasicBlock *Body = BasicBlock::Create(Header->getContext(), Name + ".body",
                                        Header->getParent(), Exit);
  BasicBlock *Latch = BasicBlock::Create(Header->getContext(), Name + ".latch",
                                         Header->getParent(), Exit);

  Type *I32Ty = Type::getInt64Ty(Ctx);
  UncondBrInst::Create(Body, Header);
  UncondBrInst::Create(Latch, Body);
  PHINode *IV =
      PHINode::Create(I32Ty, 2, Name + ".iv", Header->getTerminator()->getIterator());
  IV->addIncoming(ConstantInt::get(I32Ty, 0), Preheader);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 49-62

```cpp
  B.SetInsertPoint(Latch);
  Value *Inc = B.CreateAdd(IV, Step, Name + ".step");
  Value *Cond = B.CreateICmpNE(Inc, Bound, Name + ".cond");
  auto *BR = B.CreateCondBr(Cond, Header, Exit);
  if (!ProfcheckDisableMetadataFixes) {
    assert(Step->getZExtValue() != 0 &&
           "Expected a non-zero step size. This is chosen by the pass and "
           "should always be non-zero to imply a finite loop.");
    MDBuilder MDB(Preheader->getContext());
    setFittedBranchWeights(
        *BR, {Bound->getZExtValue() / Step->getZExtValue(), 1}, false);
  }
  IV->addIncoming(Inc, Latch);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 63-80

```cpp
  UncondBrInst *PreheaderBr = cast<UncondBrInst>(Preheader->getTerminator());
  BasicBlock *Tmp = PreheaderBr->getSuccessor();
  PreheaderBr->setSuccessor(0, Header);
  DTU.applyUpdatesPermissive({
      {DominatorTree::Delete, Preheader, Tmp},
      {DominatorTree::Insert, Header, Body},
      {DominatorTree::Insert, Body, Latch},
      {DominatorTree::Insert, Latch, Header},
      {DominatorTree::Insert, Latch, Exit},
      {DominatorTree::Insert, Preheader, Header},
  });

  L->addBasicBlockToLoop(Header, LI);
  L->addBasicBlockToLoop(Body, LI);
  L->addBasicBlockToLoop(Latch, LI);
  return Body;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 81-97

```cpp
// Creates the following loop nest skeleton:
//  for C = 0; C < NumColumns; C += TileSize
//    for R = 0; R < NumRows; R += TileSize
//      for K = 0; K < Inner ; K += TileSize
BasicBlock *TileInfo::CreateTiledLoops(BasicBlock *Start, BasicBlock *End,
                                       IRBuilderBase &B, DomTreeUpdater &DTU,
                                       LoopInfo &LI) {
  Loop *ColumnLoopInfo = LI.AllocateLoop();
  Loop *RowLoopInfo = LI.AllocateLoop();
  Loop *KLoopInfo = LI.AllocateLoop();
  RowLoopInfo->addChildLoop(KLoopInfo);
  ColumnLoopInfo->addChildLoop(RowLoopInfo);
  if (Loop *ParentL = LI.getLoopFor(Start))
    ParentL->addChildLoop(ColumnLoopInfo);
  else
    LI.addTopLevelLoop(ColumnLoopInfo);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 98-112

```cpp
  BasicBlock *ColBody =
      CreateLoop(Start, End, B.getInt64(NumColumns), B.getInt64(TileSize),
                 "cols", B, DTU, ColumnLoopInfo, LI);
  ColumnLoop.Latch = ColBody->getSingleSuccessor();
  BasicBlock *RowBody =
      CreateLoop(ColBody, ColumnLoop.Latch, B.getInt64(NumRows),
                 B.getInt64(TileSize), "rows", B, DTU, RowLoopInfo, LI);
  RowLoop.Latch = RowBody->getSingleSuccessor();

  BasicBlock *InnerBody =
      CreateLoop(RowBody, RowLoop.Latch, B.getInt64(NumInner),
                 B.getInt64(TileSize), "inner", B, DTU, KLoopInfo, LI);
  KLoop.Latch = InnerBody->getSingleSuccessor();
  ColumnLoop.Header = ColBody->getSinglePredecessor();
  RowLoop.Header = RowBody->getSinglePredecessor();
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 113-119

```cpp
  KLoop.Header = InnerBody->getSinglePredecessor();
  RowLoop.Index = &*RowLoop.Header->begin();
  ColumnLoop.Index = &*ColumnLoop.Header->begin();
  KLoop.Index = &*KLoop.Header->begin();

  return InnerBody;
}
```
- EN: This region continues the MatrixUtils implementation with local helper logic centered on KLoop, Header, InnerBody, RowLoop.
- CN: 这一段延续了 MatrixUtils 的主体实现，围绕 KLoop, Header, InnerBody, RowLoop 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Analysis coupling: it cooperates with `DominatorTree, DomTreeUpdater, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, DomTreeUpdater, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/MatrixUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/ProfDataUtils.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/MatrixUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `DomTreeUpdater`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `DomTreeUpdater`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
