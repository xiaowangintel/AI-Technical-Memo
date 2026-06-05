# LoopConstrainer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LoopConstrainer.cpp` | `llvm/lib/Transforms/Utils/LoopConstrainer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements first, try to prove the predicate without applying loop guards. within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LoopConstrainer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-84

```cpp
#include "llvm/Transforms/Utils/LoopConstrainer.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include "llvm/IR/Dominators.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include "llvm/Transforms/Utils/LoopSimplify.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"

using namespace llvm;

static const char *ClonedLoopTag = "loop_constrainer.loop.clone";

#define DEBUG_TYPE "loop-constrainer"

static bool isLoopEntryGuardedByCond(ScalarEvolution &SE, Loop *L,
                                     ICmpInst::Predicate Pred,
                                     const SCEV *Start, const SCEV *Bound) {
  // First, try to prove the predicate without applying loop guards.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return isLoopEntryGuardedByCond(SE, L, BoundPred, Start, MinusOne) &&
         isLoopEntryGuardedByCond(SE, L, BoundPred, BoundSCEV, Limit);
}

/// Given a loop with an increasing induction variable, is it possible to
/// safely calculate the bounds of a new loop using the given Predicate.
static bool isSafeIncreasingBound(const SCEV *Start, const SCEV *BoundSCEV,
                                  const SCEV *Step, ICmpInst::Predicate Pred,
                                  unsigned LatchBrExitIdx, Loop *L,
                                  ScalarEvolution &SE) {
  if (Pred != ICmpInst::ICMP_SLT && Pred != ICmpInst::ICMP_SGT &&
      Pred != ICmpInst::ICMP_ULT && Pred != ICmpInst::ICMP_UGT)
    return false;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 85-164

```cpp
  if (!SE.isAvailableAtLoopEntry(BoundSCEV, L))
    return false;

  LLVM_DEBUG(dbgs() << "isSafeIncreasingBound with:\n");
  LLVM_DEBUG(dbgs() << "Start: " << *Start << "\n");
  LLVM_DEBUG(dbgs() << "Step: " << *Step << "\n");
  LLVM_DEBUG(dbgs() << "BoundSCEV: " << *BoundSCEV << "\n");
  LLVM_DEBUG(dbgs() << "Pred: " << Pred << "\n");
  LLVM_DEBUG(dbgs() << "LatchExitBrIdx: " << LatchBrExitIdx << "\n");

  bool IsSigned = ICmpInst::isSigned(Pred);
  // The predicate that we need to check that the induction variable lies
  // within bounds.
  ICmpInst::Predicate BoundPred =
      IsSigned ? CmpInst::ICMP_SLT : CmpInst::ICMP_ULT;

  if (LatchBrExitIdx == 1)
    return isLoopEntryGuardedByCond(SE, L, BoundPred, Start, BoundSCEV);

  assert(LatchBrExitIdx == 0 && "LatchBrExitIdx should be 0 or 1");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  BasicBlock *Header = L.getHeader();
  BasicBlock *Preheader = L.getLoopPreheader();
  if (!Preheader) {
    FailureReason = "no preheader";
    return std::nullopt;
  }

  CondBrInst *LatchBr = dyn_cast<CondBrInst>(Latch->getTerminator());
  if (!LatchBr) {
    FailureReason = "latch terminator not conditional branch";
    return std::nullopt;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 165-246

```cpp
  unsigned LatchBrExitIdx = LatchBr->getSuccessor(0) == Header ? 1 : 0;

  ICmpInst *ICI = dyn_cast<ICmpInst>(LatchBr->getCondition());
  if (!ICI || !isa<IntegerType>(ICI->getOperand(0)->getType())) {
    FailureReason = "latch terminator branch not conditional on integral icmp";
    return std::nullopt;
  }

  const SCEV *MaxBETakenCount = getNarrowestLatchMaxTakenCountEstimate(SE, L);
  if (isa<SCEVCouldNotCompute>(MaxBETakenCount)) {
    FailureReason = "could not compute latch count";
    return std::nullopt;
  }
  assert(SE.getLoopDisposition(MaxBETakenCount, &L) ==
             ScalarEvolution::LoopInvariant &&
         "loop variant exit count doesn't make sense!");

  ICmpInst::Predicate Pred = ICI->getPredicate();
  Value *LeftValue = ICI->getOperand(0);
  const SCEV *LeftSCEV = SE.getSCEV(LeftValue);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    FailureReason = "LHS in cmp is not an AddRec for this loop";
    return std::nullopt;
  }
  if (!IndVarBase->isAffine()) {
    FailureReason = "LHS in icmp not induction variable";
    return std::nullopt;
  }
  const SCEV *StepRec = IndVarBase->getStepRecurrence(SE);
  if (!isa<SCEVConstant>(StepRec)) {
    FailureReason = "LHS in icmp not induction variable";
    return std::nullopt;
  }
  ConstantInt *StepCI = cast<SCEVConstant>(StepRec)->getValue();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 247-320

```cpp
  if (ICI->isEquality() && !HasNoSignedWrap(IndVarBase)) {
    FailureReason = "LHS in icmp needs nsw for equality predicates";
    return std::nullopt;
  }

  assert(!StepCI->isZero() && "Zero step?");
  bool IsIncreasing = !StepCI->isNegative();
  bool IsSignedPredicate;
  const SCEV *StartNext = IndVarBase->getStart();
  const SCEV *Addend = SE.getNegativeSCEV(IndVarBase->getStepRecurrence(SE));
  const SCEV *IndVarStart = SE.getAddExpr(StartNext, Addend);
  const SCEV *Step = SE.getSCEV(StepCI);

  const SCEV *FixedRightSCEV = nullptr;

  // If RightValue resides within loop (but still being loop invariant),
  // regenerate it as preheader.
  if (auto *I = dyn_cast<Instruction>(RightValue))
    if (L.contains(I->getParent()))
      FixedRightSCEV = RightSCEV;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    bool FoundExpectedPred =
        (LTPred && LatchBrExitIdx == 1) || (GTPred && LatchBrExitIdx == 0);

    if (!FoundExpectedPred) {
      FailureReason = "expected icmp slt semantically, found something else";
      return std::nullopt;
    }

    IsSignedPredicate = ICmpInst::isSigned(Pred);
    if (!IsSignedPredicate && !AllowUnsignedLatchCond) {
      FailureReason = "unsigned latch conditions are explicitly prohibited";
      return std::nullopt;
    }

```
- EN: Core entities appearing here include cannotBeMinInLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 cannotBeMinInLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 321-404

```cpp
    if (!isSafeIncreasingBound(IndVarStart, RightSCEV, Step, Pred,
                               LatchBrExitIdx, &L, SE)) {
      FailureReason = "Unsafe loop bounds";
      return std::nullopt;
    }
    if (LatchBrExitIdx == 0) {
      // We need to increase the right value unless we have already decreased
      // it virtually when we replaced EQ with SGT.
      if (!DecreasedRightValueByOne)
        FixedRightSCEV =
            SE.getAddExpr(RightSCEV, SE.getOne(RightSCEV->getType()));
    } else {
      assert(!DecreasedRightValueByOne &&
             "Right value can be decreased only for LatchBrExitIdx == 0!");
    }
  } else {
    bool IncreasedRightValueByOne = false;
    if (StepCI->isMinusOne()) {
      // Try to turn eq/ne predicates to those we can work with.
      if (Pred == ICmpInst::ICMP_NE && LatchBrExitIdx == 1)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

    if (LatchBrExitIdx == 0) {
      // We need to decrease the right value unless we have already increased
      // it virtually when we replaced EQ with SLT.
      if (!IncreasedRightValueByOne)
        FixedRightSCEV =
            SE.getMinusSCEV(RightSCEV, SE.getOne(RightSCEV->getType()));
    } else {
      assert(!IncreasedRightValueByOne &&
             "Right value can be increased only for LatchBrExitIdx == 0!");
    }
  }
  BasicBlock *LatchExit = LatchBr->getSuccessor(LatchBrExitIdx);

```
- EN: Core entities appearing here include cannotBeMaxInLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 cannotBeMaxInLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 405-488

```cpp
  assert(!L.contains(LatchExit) && "expected an exit block!");
  SCEVExpander Expander(SE, "loop-constrainer");
  Instruction *Ins = Preheader->getTerminator();

  if (FixedRightSCEV)
    RightValue =
        Expander.expandCodeFor(FixedRightSCEV, FixedRightSCEV->getType(), Ins);

  Value *IndVarStartV = Expander.expandCodeFor(IndVarStart, IndVarTy, Ins);
  IndVarStartV->setName("indvar.start");

  LoopStructure Result;

  Result.Tag = "main";
  Result.Header = Header;
  Result.Latch = Latch;
  Result.LatchBr = LatchBr;
  Result.LatchExit = LatchExit;
  Result.LatchBrExitIdx = LatchBrExitIdx;
  Result.IndVarStart = IndVarStartV;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  for (BasicBlock *BB : OriginalLoop.getBlocks()) {
    BasicBlock *Clone = CloneBasicBlock(BB, Result.Map, Twine(".") + Tag, &F);
    Result.Blocks.push_back(Clone);
    Result.Map[BB] = Clone;
  }

  auto GetClonedValue = [&Result](Value *V) {
    assert(V && "null values not in domain!");
    auto It = Result.Map.find(V);
    if (It == Result.Map.end())
      return V;
    return static_cast<Value *>(It->second);
  };

```
- EN: Core entities appearing here include DisableAllLoopOptsOnLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 DisableAllLoopOptsOnLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 489-565

```cpp
  auto *ClonedLatch =
      cast<BasicBlock>(GetClonedValue(OriginalLoop.getLoopLatch()));
  ClonedLatch->getTerminator()->setMetadata(ClonedLoopTag,
                                            MDNode::get(Ctx, {}));

  Result.Structure = MainLoopStructure.map(GetClonedValue);
  Result.Structure.Tag = Tag;

  for (unsigned i = 0, e = Result.Blocks.size(); i != e; ++i) {
    BasicBlock *ClonedBB = Result.Blocks[i];
    BasicBlock *OriginalBB = OriginalLoop.getBlocks()[i];

    assert(Result.Map[OriginalBB] == ClonedBB && "invariant!");

    for (Instruction &I : *ClonedBB)
      RemapInstruction(&I, Result.Map,
                       RF_NoModuleLevelChanges | RF_IgnoreMissingLocals);

    // Exit blocks will now have one more predecessor and their PHI nodes need
    // to be edited to reflect that.  No phi nodes need to be introduced because
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //            |   |                    |
  //            +--->   original exit    |
  //                |                    |
  //                +--------------------+
  //
  // We change the control flow to look like
  //
  //
  //    +--------------------+
  //    |                    |
  //    |     preheader      >-------------------------+
  //    |                    |                         |
  //    +--------v-----------+                         |
  //             |    /-------------+                  |
```
- EN: This region continues the LoopConstrainer implementation with local helper logic centered on ClonedLatch, BasicBlock, GetClonedValue, OriginalLoop.
- CN: 这一段延续了 LoopConstrainer 的主体实现，围绕 ClonedLatch, BasicBlock, GetClonedValue, OriginalLoop 等局部辅助逻辑展开。

### Lines 566-649

```cpp
  //             |   /              |                  |
  //    +--------v--v--------+      |                  |
  //    |                    |      |                  |
  //    |      header        |      |   +--------+     |
  //    |                    |      |   |        |     |
  //    +--------------------+      |   |  +-----v-----v-----------+
  //                                |   |  |                       |
  //                                |   |  |     .pseudo.exit      |
  //                                |   |  |                       |
  //                                |   |  +-----------v-----------+
  //                                |   |              |
  //            .....               |   |              |
  //                                |   |     +--------v-------------+
  //    +--------------------+      |   |     |                      |
  //    |                    |      |   |     |   ContinuationBlock  |
  //    |       latch        >------+   |     |                      |
  //    |                    |          |     +----------------------+
  //    +---------v----------+          |
  //              |                     |
  //              |                     |
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                             ? TakeBackedgeLoopCond
                             : B.CreateNot(TakeBackedgeLoopCond);

  LS.LatchBr->setCondition(CondForBranch);

  B.SetInsertPoint(RRI.ExitSelector);

  // IterationsLeft - are there any more iterations left, given the original
  // upper bound on the induction variable?  If not, we branch to the "real"
  // exit.
  Value *LoopExitAt = NoopOrExt(LS.LoopExitAt);
  Value *IterationsLeft = B.CreateICmp(Pred, IndVarBase, LoopExitAt);
  B.CreateCondBr(IterationsLeft, RRI.PseudoExit, LS.LatchExit);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 650-727

```cpp
  UncondBrInst *BranchToContinuation =
      UncondBrInst::Create(ContinuationBlock, RRI.PseudoExit);

  // We emit PHI nodes into `RRI.PseudoExit' that compute the "latest" value of
  // each of the PHI nodes in the loop header.  This feeds into the initial
  // value of the same PHI nodes if/when we continue execution.
  for (PHINode &PN : LS.Header->phis()) {
    PHINode *NewPHI = PHINode::Create(PN.getType(), 2, PN.getName() + ".copy",
                                      BranchToContinuation->getIterator());

    NewPHI->addIncoming(PN.getIncomingValueForBlock(Preheader), Preheader);
    NewPHI->addIncoming(PN.getIncomingValueForBlock(LS.Latch),
                        RRI.ExitSelector);
    RRI.PHIValuesAtPseudoExit.push_back(NewPHI);
  }

  RRI.IndVarEnd = PHINode::Create(IndVarBase->getType(), 2, "indvar.end",
                                  BranchToContinuation->getIterator());
  RRI.IndVarEnd->addIncoming(IndVarStart, Preheader);
  RRI.IndVarEnd->addIncoming(IndVarBase, RRI.ExitSelector);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Parent->addChildLoop(&New);
  else
    LI.addTopLevelLoop(&New);
  LPMAddNewLoop(&New, IsSubloop);

  // Add all of the blocks in Original to the new loop.
  for (auto *BB : Original->blocks())
    if (LI.getLoopFor(BB) == Original)
      New.addBasicBlockToLoop(cast<BasicBlock>(VM[BB]), LI);

  // Add all of the subloops to the new loop.
  for (Loop *SubLoop : *Original)
    createClonedLoopStructure(SubLoop, &New, VM, /* IsSubloop */ true);

```
- EN: Core entities appearing here include addToParentLoopIfNeeded, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 addToParentLoopIfNeeded，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 728-807

```cpp
  return &New;
}

bool LoopConstrainer::run() {
  BasicBlock *Preheader = OriginalLoop.getLoopPreheader();
  assert(Preheader != nullptr && "precondition!");

  OriginalPreheader = Preheader;
  MainLoopPreheader = Preheader;
  bool IsSignedPredicate = MainLoopStructure.IsSignedPredicate;
  bool Increasing = MainLoopStructure.IndVarIncreasing;
  IntegerType *IVTy = cast<IntegerType>(RangeTy);

  SCEVExpander Expander(SE, "loop-constrainer");
  Instruction *InsertPt = OriginalPreheader->getTerminator();

  // It would have been better to make `PreLoop' and `PostLoop'
  // `std::optional<ClonedLoop>'s, but `ValueToValueMapTy' does not have a copy
  // constructor.
  ClonedLoop PreLoop, PostLoop;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      LLVM_DEBUG(dbgs() << "could not prove no-overflow when computing "
                        << "mainloop exit limit.  LowLimit = "
                        << *(*SR.LowLimit) << "\n");
      return false;
    }

    if (!Expander.isSafeToExpandAt(ExitMainLoopAtSCEV, InsertPt)) {
      LLVM_DEBUG(dbgs() << "could not prove that it is safe to expand the"
                        << " main loop exit limit " << *ExitMainLoopAtSCEV
                        << " at block " << InsertPt->getParent()->getName()
                        << "\n");
      return false;
    }

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 808-891

```cpp
    ExitMainLoopAt = Expander.expandCodeFor(ExitMainLoopAtSCEV, IVTy, InsertPt);
    ExitMainLoopAt->setName("exit.mainloop.at");
  }

  // We clone these ahead of time so that we don't have to deal with changing
  // and temporarily invalid IR as we transform the loops.
  if (NeedsPreLoop)
    cloneLoop(PreLoop, "preloop");
  if (NeedsPostLoop)
    cloneLoop(PostLoop, "postloop");

  RewrittenRangeInfo PreLoopRRI;

  if (NeedsPreLoop) {
    Preheader->getTerminator()->replaceUsesOfWith(MainLoopStructure.Header,
                                                  PreLoop.Structure.Header);

    MainLoopPreheader =
        createPreheader(MainLoopStructure, Preheader, "mainloop");
    PreLoopRRI = changeIterationSpaceEnd(PreLoop.Structure, Preheader,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto CanonicalizeLoop = [&](Loop *L, bool IsOriginalLoop) {
    formLCSSARecursively(*L, DT, &LI, &SE);
    simplifyLoop(L, &DT, &LI, &SE, nullptr, nullptr, true);
    // Pre/post loops are slow paths, we do not need to perform any loop
    // optimizations on them.
    if (!IsOriginalLoop)
      DisableAllLoopOptsOnLoop(*L);
  };
  if (PreL)
    CanonicalizeLoop(PreL, false);
  if (PostL)
    CanonicalizeLoop(PostL, false);
  CanonicalizeLoop(&OriginalLoop, true);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 892-915

```cpp
  /// At this point:
  /// - We've broken a "main loop" out of the loop in a way that the "main loop"
  /// runs with the induction variable in a subset of [Begin, End).
  /// - There is no overflow when computing "main loop" exit limit.
  /// - Max latch taken count of the loop is limited.
  /// It guarantees that induction variable will not overflow iterating in the
  /// "main loop".
  if (isa<OverflowingBinaryOperator>(MainLoopStructure.IndVarBase))
    if (IsSignedPredicate)
      cast<BinaryOperator>(MainLoopStructure.IndVarBase)
          ->setHasNoSignedWrap(true);
  /// TODO: support unsigned predicate.
  /// To add NUW flag we need to prove that both operands of BO are
  /// non-negative. E.g:
  /// ...
  /// %iv.next = add nsw i32 %iv, -1
  /// %cmp = icmp ult i32 %iv.next, %n
  /// br i1 %cmp, label %loopexit, label %loop
  ///
  /// -1 is MAX_UINT in terms of unsigned int. Adding anything but zero will
  /// overflow, therefore NUW flag is not legal here.

  return true;
}
```
- EN: This region continues the LoopConstrainer implementation with local helper logic centered on Begin, End, There, Max.
- CN: 这一段延续了 LoopConstrainer 的主体实现，围绕 Begin, End, There, Max 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `cannotBeMinInLoop, cannotBeMaxInLoop, DisableAllLoopOptsOnLoop, addToParentLoopIfNeeded, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`cannotBeMinInLoop, cannotBeMaxInLoop, DisableAllLoopOptsOnLoop, addToParentLoopIfNeeded, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, LoopInfo, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, LoopInfo, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/Transforms/Utils/LoopConstrainer.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/LoopSimplify.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/Transforms/Utils/LoopConstrainer.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/LoopSimplify.h`, `llvm/Transforms/Utils/LoopUtils.h`, `llvm/Transforms/Utils/ScalarEvolutionExpander.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
