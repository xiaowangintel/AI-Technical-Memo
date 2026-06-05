# LoopIdiomVectorize.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/LoopIdiomVectorize.cpp` | `llvm/lib/Transforms/Vectorize/LoopIdiomVectorize.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements loop idiom vectorization within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 LoopIdiomVectorize 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-97

```cpp
//===-------- LoopIdiomVectorize.cpp - Loop idiom vectorization -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass implements a pass that recognizes certain loop idioms and
// transforms them into more optimized versions of the same loop. In cases
// where this happens, it can be a significant performance win.
//
// We currently support two loops:
//
// 1. A loop that finds the first mismatched byte in an array and returns the
// index, i.e. something like:
//
//  while (++i != n) {
//    if (a[i] != b[i])
//      break;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

static cl::opt<bool> DisableAll("disable-loop-idiom-vectorize-all", cl::Hidden,
                                cl::init(false),
                                cl::desc("Disable Loop Idiom Vectorize Pass."));

static cl::opt<LoopIdiomVectorizeStyle>
    LITVecStyle("loop-idiom-vectorize-style", cl::Hidden,
                cl::desc("The vectorization style for loop idiom transform."),
                cl::values(clEnumValN(LoopIdiomVectorizeStyle::Masked, "masked",
                                      "Use masked vector intrinsics"),
                           clEnumValN(LoopIdiomVectorizeStyle::Predicated,
                                      "predicated", "Use VP intrinsics")),
                cl::init(LoopIdiomVectorizeStyle::Masked));

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 98-190

```cpp
static cl::opt<bool>
    DisableByteCmp("disable-loop-idiom-vectorize-bytecmp", cl::Hidden,
                   cl::init(false),
                   cl::desc("Proceed with Loop Idiom Vectorize Pass, but do "
                            "not convert byte-compare loop(s)."));

static cl::opt<unsigned>
    ByteCmpVF("loop-idiom-vectorize-bytecmp-vf", cl::Hidden,
              cl::desc("The vectorization factor for byte-compare patterns."),
              cl::init(16));

static cl::opt<bool>
    DisableFindFirstByte("disable-loop-idiom-vectorize-find-first-byte",
                         cl::Hidden, cl::init(false),
                         cl::desc("Do not convert find-first-byte loop(s)."));

static cl::opt<bool>
    VerifyLoops("loop-idiom-vectorize-verify", cl::Hidden, cl::init(false),
                cl::desc("Verify loops generated Loop Idiom Vectorize Pass."));

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *expandFindFirstByte(IRBuilder<> &Builder, DomTreeUpdater &DTU,
                             unsigned VF, Type *CharTy, Value *IndPhi,
                             BasicBlock *ExitSucc, BasicBlock *ExitFail,
                             Value *SearchStart, Value *SearchEnd,
                             Value *NeedleStart, Value *NeedleEnd);

  void transformFindFirstByte(PHINode *IndPhi, unsigned VF, Type *CharTy,
                              BasicBlock *ExitSucc, BasicBlock *ExitFail,
                              Value *SearchStart, Value *SearchEnd,
                              Value *NeedleStart, Value *NeedleEnd);
  /// @}
};
} // anonymous namespace

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include LoopIdiomVectorize, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 LoopIdiomVectorize，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 191-279

```cpp
PreservedAnalyses LoopIdiomVectorizePass::run(Loop &L, LoopAnalysisManager &AM,
                                              LoopStandardAnalysisResults &AR,
                                              LPMUpdater &) {
  if (DisableAll)
    return PreservedAnalyses::all();

  const auto *DL = &L.getHeader()->getDataLayout();

  LoopIdiomVectorizeStyle VecStyle = VectorizeStyle;
  if (LITVecStyle.getNumOccurrences())
    VecStyle = LITVecStyle;

  unsigned BCVF = ByteCompareVF;
  if (ByteCmpVF.getNumOccurrences())
    BCVF = ByteCmpVF;

  Function &F = *L.getHeader()->getParent();
  auto &FAMP = AM.getResult<FunctionAnalysisManagerLoopProxy>(L, AR);
  auto *ORE = FAMP.getCachedResult<OptimizationRemarkEmitterAnalysis>(F);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

static void fixSuccessorPhis(Loop *L, Value *ScalarRes, Value *VectorRes,
                             BasicBlock *SuccBB, BasicBlock *IncBB) {
  for (PHINode &PN : SuccBB->phis()) {
    // Look through the incoming values to find ScalarRes, meaning this is a
    // PHI collecting the results of the transformation.
    bool ResPhi = false;
    for (Value *Op : PN.incoming_values())
      if (Op == ScalarRes) {
        ResPhi = true;
        break;
      }

```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, OptimizationRemarkEmitter.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, OptimizationRemarkEmitter 等分析结果。

### Lines 280-372

```cpp
    // Any PHI that depended upon the result of the transformation needs a new
    // incoming value from IncBB.
    if (ResPhi)
      PN.addIncoming(VectorRes, IncBB);
    else {
      // There should be no other outside uses of other values in the
      // original loop. Any incoming values should either:
      //   1. Be for blocks outside the loop, which aren't interesting. Or ..
      //   2. These are from blocks in the loop with values defined outside
      //      the loop. We should a similar incoming value from CmpBB.
      for (BasicBlock *BB : PN.blocks())
        if (L->contains(BB)) {
          PN.addIncoming(PN.getIncomingValueForBlock(BB), IncBB);
          break;
        }
    }
  }
}

bool LoopIdiomVectorize::recognizeByteCompare() {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!Index || !Index->getType()->isIntegerTy(32) ||
      !match(Index, m_c_Add(m_Specific(PN), m_One())))
    return false;

  // If we match the pattern, PN and Index will be replaced with the result of
  // the cttz.elts intrinsic. If any other instructions are used outside of
  // the loop, we cannot replace it.
  for (BasicBlock *BB : LoopBlocks)
    for (Instruction &I : *BB)
      if (&I != PN && &I != Index)
        for (User *U : I.users())
          if (!CurLoop->contains(cast<Instruction>(U)))
            return false;

```
- EN: Core entities appearing here include recognizeByteCompare, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 recognizeByteCompare，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 373-468

```cpp
  // Match the branch instruction for the header
  Value *MaxLen;
  BasicBlock *EndBB, *WhileBB;
  if (!match(Header->getTerminator(),
             m_Br(m_SpecificICmp(ICmpInst::ICMP_EQ, m_Specific(Index),
                                 m_Value(MaxLen)),
                  m_BasicBlock(EndBB), m_BasicBlock(WhileBB))) ||
      !CurLoop->contains(WhileBB))
    return false;

  // WhileBB should contain the pattern of load & compare instructions. Match
  // the pattern and find the GEP instructions used by the loads.
  BasicBlock *FoundBB;
  BasicBlock *TrueBB;
  Value *LoadA, *LoadB;
  if (!match(WhileBB->getTerminator(),
             m_Br(m_SpecificICmp(ICmpInst::ICMP_EQ, m_Value(LoadA),
                                 m_Value(LoadB)),
                  m_BasicBlock(TrueBB), m_BasicBlock(FoundBB))) ||
      !CurLoop->contains(TrueBB))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Value *WhileCondVal = EndPN.getIncomingValueForBlock(Header);
      Value *WhileBodyVal = EndPN.getIncomingValueForBlock(WhileBB);

      // The value of the index when leaving the while.cond block is always the
      // same as the end value (MaxLen) so we permit either. The value when
      // leaving the while.body block should only be the index. Otherwise for
      // any other values we only allow ones that are same for both blocks.
      if (WhileCondVal != WhileBodyVal &&
          ((WhileCondVal != Index && WhileCondVal != MaxLen) ||
           (WhileBodyVal != Index)))
        return false;
    }
  }

```
- EN: This region continues the LoopIdiomVectorize implementation with local helper logic centered on Match, Value, MaxLen, BasicBlock.
- CN: 这一段延续了 LoopIdiomVectorize 的主体实现，围绕 Match, Value, MaxLen, BasicBlock 等局部辅助逻辑展开。

### Lines 469-570

```cpp
  LLVM_DEBUG(dbgs() << "FOUND IDIOM IN LOOP: \n"
                    << *(EndBB->getParent()) << "\n\n");

  // The index is incremented before the GEP/Load pair so we need to
  // add 1 to the start value.
  transformByteCompare(GEPA, GEPB, PN, MaxLen, Index, StartIdx, /*IncIdx=*/true,
                       FoundBB, EndBB);
  return true;
}

Value *LoopIdiomVectorize::createMaskedFindMismatch(
    IRBuilder<> &Builder, DomTreeUpdater &DTU, GetElementPtrInst *GEPA,
    GetElementPtrInst *GEPB, Value *ExtStart, Value *ExtEnd) {
  Type *I64Type = Builder.getInt64Ty();
  Type *ResType = Builder.getInt32Ty();
  Type *LoadType = Builder.getInt8Ty();
  Value *PtrA = GEPA->getPointerOperand();
  Value *PtrB = GEPB->getPointerOperand();

  ScalableVectorType *PredVTy =
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
       {DominatorTree::Insert, VectorLoopIncBlock, EndBlock}});

  // If we found a mismatch then we need to calculate which lane in the vector
  // had a mismatch and add that on to the current loop index.
  Builder.SetInsertPoint(VectorLoopMismatchBlock);
  PHINode *FoundPred = Builder.CreatePHI(PredVTy, 1, "mismatch_vec_found_pred");
  FoundPred->addIncoming(VectorMatchCmp, VectorLoopStartBlock);
  PHINode *LastLoopPred =
      Builder.CreatePHI(PredVTy, 1, "mismatch_vec_last_loop_pred");
  LastLoopPred->addIncoming(LoopPred, VectorLoopStartBlock);
  PHINode *VectorFoundIndex =
      Builder.CreatePHI(I64Type, 1, "mismatch_vec_found_index");
  VectorFoundIndex->addIncoming(VectorIndexPhi, VectorLoopStartBlock);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 571-663

```cpp
  Value *PredMatchCmp = Builder.CreateAnd(LastLoopPred, FoundPred);
  Value *Ctz = Builder.CreateCountTrailingZeroElems(ResType, PredMatchCmp);
  Ctz = Builder.CreateZExt(Ctz, I64Type);
  Value *VectorLoopRes64 = Builder.CreateAdd(VectorFoundIndex, Ctz, "",
                                             /*HasNUW=*/true, /*HasNSW=*/true);
  return Builder.CreateTrunc(VectorLoopRes64, ResType);
}

Value *LoopIdiomVectorize::createPredicatedFindMismatch(
    IRBuilder<> &Builder, DomTreeUpdater &DTU, GetElementPtrInst *GEPA,
    GetElementPtrInst *GEPB, Value *ExtStart, Value *ExtEnd) {
  Type *I64Type = Builder.getInt64Ty();
  Type *I32Type = Builder.getInt32Ty();
  Type *ResType = I32Type;
  Type *LoadType = Builder.getInt8Ty();
  Value *PtrA = GEPA->getPointerOperand();
  Value *PtrB = GEPB->getPointerOperand();

  auto *JumpToVectorLoop = UncondBrInst::Create(VectorLoopStartBlock);
  Builder.Insert(JumpToVectorLoop);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  VectorIndexPhi->addIncoming(NewVectorIndexPhi, VectorLoopIncBlock);
  Value *ExitCond = Builder.CreateICmpNE(NewVectorIndexPhi, ExtEnd);
  auto *VectorLoopBranchBack =
      CondBrInst::Create(ExitCond, VectorLoopStartBlock, EndBlock);
  Builder.Insert(VectorLoopBranchBack);

  DTU.applyUpdates(
      {{DominatorTree::Insert, VectorLoopIncBlock, VectorLoopStartBlock},
       {DominatorTree::Insert, VectorLoopIncBlock, EndBlock}});

  // If we found a mismatch then we need to calculate which lane in the vector
  // had a mismatch and add that on to the current loop index.
  Builder.SetInsertPoint(VectorLoopMismatchBlock);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 664-756

```cpp
  // Add LCSSA phis for CTZ and VectorIndexPhi.
  auto *CTZLCSSAPhi = Builder.CreatePHI(CTZ->getType(), 1, "ctz");
  CTZLCSSAPhi->addIncoming(CTZ, VectorLoopStartBlock);
  auto *VectorIndexLCSSAPhi =
      Builder.CreatePHI(VectorIndexPhi->getType(), 1, "mismatch_vector_index");
  VectorIndexLCSSAPhi->addIncoming(VectorIndexPhi, VectorLoopStartBlock);

  Value *CTZI64 = Builder.CreateZExt(CTZLCSSAPhi, I64Type);
  Value *VectorLoopRes64 = Builder.CreateAdd(VectorIndexLCSSAPhi, CTZI64, "",
                                             /*HasNUW=*/true, /*HasNSW=*/true);
  return Builder.CreateTrunc(VectorLoopRes64, ResType);
}

Value *LoopIdiomVectorize::expandFindMismatch(
    IRBuilder<> &Builder, DomTreeUpdater &DTU, GetElementPtrInst *GEPA,
    GetElementPtrInst *GEPB, Instruction *Index, Value *Start, Value *MaxLen) {
  Value *PtrA = GEPA->getPointerOperand();
  Value *PtrB = GEPB->getPointerOperand();

  // Get the arguments and types for the intrinsic.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (CurLoop->getParentLoop()) {
    CurLoop->getParentLoop()->addBasicBlockToLoop(MinItCheckBlock, *LI);
    CurLoop->getParentLoop()->addBasicBlockToLoop(MemCheckBlock, *LI);
    CurLoop->getParentLoop()->addBasicBlockToLoop(VectorLoopPreheaderBlock,
                                                  *LI);
    CurLoop->getParentLoop()->addChildLoop(VectorLoop);
    CurLoop->getParentLoop()->addBasicBlockToLoop(VectorLoopMismatchBlock, *LI);
    CurLoop->getParentLoop()->addBasicBlockToLoop(LoopPreHeaderBlock, *LI);
    CurLoop->getParentLoop()->addChildLoop(ScalarLoop);
  } else {
    LI->addTopLevelLoop(VectorLoop);
    LI->addTopLevelLoop(ScalarLoop);
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 757-852

```cpp
  // Add the new basic blocks to their associated loops.
  VectorLoop->addBasicBlockToLoop(VectorLoopStartBlock, *LI);
  VectorLoop->addBasicBlockToLoop(VectorLoopIncBlock, *LI);

  ScalarLoop->addBasicBlockToLoop(LoopStartBlock, *LI);
  ScalarLoop->addBasicBlockToLoop(LoopIncBlock, *LI);

  // Set up some types and constants that we intend to reuse.
  Type *I64Type = Builder.getInt64Ty();

  // Check the zero-extended iteration count > 0
  Builder.SetInsertPoint(MinItCheckBlock);
  Value *ExtStart = Builder.CreateZExt(Start, I64Type);
  Value *ExtEnd = Builder.CreateZExt(MaxLen, I64Type);
  // This check doesn't really cost us very much.

  Value *LimitCheck = Builder.CreateICmpULE(Start, MaxLen);
  CondBrInst *MinItCheckBr =
      CondBrInst::Create(LimitCheck, MemCheckBlock, LoopPreHeaderBlock);
  MinItCheckBr->setMetadata(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Therefore, we know that we can use a 64-bit induction variable that
  // starts from 0 -> ExtMaxLen and it will not overflow.
  Value *VectorLoopRes = nullptr;
  switch (VectorizeStyle) {
  case LoopIdiomVectorizeStyle::Masked:
    VectorLoopRes =
        createMaskedFindMismatch(Builder, DTU, GEPA, GEPB, ExtStart, ExtEnd);
    break;
  case LoopIdiomVectorizeStyle::Predicated:
    VectorLoopRes = createPredicatedFindMismatch(Builder, DTU, GEPA, GEPB,
                                                 ExtStart, ExtEnd);
    break;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 853-946

```cpp
  Builder.CreateBr(EndBlock);

  DTU.applyUpdates(
      {{DominatorTree::Insert, VectorLoopMismatchBlock, EndBlock}});

  // Generate code for scalar loop.
  Builder.SetInsertPoint(LoopPreHeaderBlock);
  Builder.CreateBr(LoopStartBlock);

  DTU.applyUpdates(
      {{DominatorTree::Insert, LoopPreHeaderBlock, LoopStartBlock}});

  Builder.SetInsertPoint(LoopStartBlock);
  PHINode *IndexPhi = Builder.CreatePHI(ResType, 2, "mismatch_index");
  IndexPhi->addIncoming(Start, LoopPreHeaderBlock);

  // Otherwise compare the values
  // Load bytes from each array and compare them.
  Value *GepOffset = Builder.CreateZExt(IndexPhi, I64Type);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                              BasicBlock *EndBB) {

  // Insert the byte compare code at the end of the preheader block
  BasicBlock *Preheader = CurLoop->getLoopPreheader();
  BasicBlock *Header = CurLoop->getHeader();
  UncondBrInst *PHBranch = cast<UncondBrInst>(Preheader->getTerminator());
  IRBuilder<> Builder(PHBranch);
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Lazy);
  Builder.SetCurrentDebugLocation(PHBranch->getDebugLoc());

  // Increment the pointer if this was done before the loads in the loop.
  if (IncIdx)
    Start = Builder.CreateAdd(Start, ConstantInt::get(Start->getType(), 1));

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 947-1039

```cpp
  Value *ByteCmpRes =
      expandFindMismatch(Builder, DTU, GEPA, GEPB, Index, Start, MaxLen);

  // Replaces uses of index & induction Phi with intrinsic (we already
  // checked that the the first instruction of Header is the Phi above).
  assert(IndPhi->hasOneUse() && "Index phi node has more than one use!");
  Index->replaceAllUsesWith(ByteCmpRes);

  // If no mismatch was found, we can jump to the end block. Create a
  // new basic block for the compare instruction.
  auto *CmpBB = BasicBlock::Create(Preheader->getContext(), "byte.compare",
                                   Preheader->getParent());
  CmpBB->moveBefore(EndBB);

  // Replace the branch in the preheader with an always-true conditional branch.
  // This ensures there is still a reference to the original loop.
  Builder.CreateCondBr(Builder.getTrue(), CmpBB, Header);
  PHBranch->eraseFromParent();

  BasicBlock *MismatchEnd = cast<Instruction>(ByteCmpRes)->getParent();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  auto *InnerLoop = CurLoop->getSubLoops().front();
  Function &F = *InnerLoop->getHeader()->getParent();

  // Bail if vectorization is disabled on inner loop.
  LoopVectorizeHints Hints(InnerLoop, /*InterleaveOnlyWhenForced=*/true, ORE);
  if (!Hints.allowVectorization(&F, InnerLoop,
                                /*VectorizeOnlyWhenForced=*/false)) {
    LLVM_DEBUG(dbgs() << DEBUG_TYPE << " is disabled on inner loop "
                      << InnerLoop->getName()
                      << " due to vectorization hints\n");
    return false;
  }

```
- EN: Core entities appearing here include recognizeFindFirstByte, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 recognizeFindFirstByte，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1040-1133

```cpp
  PHINode *IndPhi = dyn_cast<PHINode>(&Header->front());
  if (!IndPhi || IndPhi->getNumIncomingValues() != 2)
    return false;

  // Check instruction counts.
  auto LoopBlocks = CurLoop->getBlocks();
  if (LoopBlocks[0]->size() > 3 || LoopBlocks[1]->size() > 4 ||
      LoopBlocks[2]->size() > 3 || LoopBlocks[3]->size() > 3)
    return false;

  // Check that no instruction other than IndPhi has outside uses.
  for (BasicBlock *BB : LoopBlocks)
    for (Instruction &I : *BB)
      if (&I != IndPhi)
        for (User *U : I.users())
          if (!CurLoop->contains(cast<Instruction>(U)))
            return false;

  // Match the branch instruction in the header. We are expecting an
  // unconditional branch to the inner loop.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // The loads come from two PHIs, each with two incoming values.
  PHINode *PSearch = dyn_cast<PHINode>(Search);
  PHINode *PNeedle = dyn_cast<PHINode>(Needle);
  if (!PSearch || PSearch->getNumIncomingValues() != 2 || !PNeedle ||
      PNeedle->getNumIncomingValues() != 2)
    return false;

  // One PHI comes from the outer loop (PSearch), the other one from the inner
  // loop (PNeedle). PSearch effectively corresponds to IndPhi.
  if (InnerLoop->contains(PSearch))
    std::swap(PSearch, PNeedle);
  if (PSearch != &Header->front() || PNeedle != &MatchBB->front())
    return false;

```
- EN: This region continues the LoopIdiomVectorize implementation with local helper logic centered on PHINode, IndPhi, Header, Check.
- CN: 这一段延续了 LoopIdiomVectorize 的主体实现，围绕 PHINode, IndPhi, Header, Check 等局部辅助逻辑展开。

### Lines 1134-1219

```cpp
  // The incoming values of both PHI nodes should be a gep of 1.
  Value *SearchStart = PSearch->getIncomingValue(0);
  Value *SearchIndex = PSearch->getIncomingValue(1);
  if (CurLoop->contains(PSearch->getIncomingBlock(0)))
    std::swap(SearchStart, SearchIndex);

  Value *NeedleStart = PNeedle->getIncomingValue(0);
  Value *NeedleIndex = PNeedle->getIncomingValue(1);
  if (InnerLoop->contains(PNeedle->getIncomingBlock(0)))
    std::swap(NeedleStart, NeedleIndex);

  // Match the GEPs.
  if (!match(SearchIndex, m_GEP(m_Specific(PSearch), m_One())) ||
      !match(NeedleIndex, m_GEP(m_Specific(PNeedle), m_One())))
    return false;

  // Check the GEPs result type matches `CharTy'.
  GetElementPtrInst *GEPSearch = cast<GetElementPtrInst>(SearchIndex);
  GetElementPtrInst *GEPNeedle = cast<GetElementPtrInst>(NeedleIndex);
  if (GEPSearch->getResultElementType() != CharTy ||
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto *PredVTy = ScalableVectorType::get(Builder.getInt1Ty(), VF);
  auto *CharVTy = ScalableVectorType::get(CharTy, VF);
  auto *ConstVF = ConstantInt::get(I64Ty, VF);

  // Other common arguments.
  BasicBlock *Preheader = CurLoop->getLoopPreheader();
  LLVMContext &Ctx = Preheader->getContext();
  Value *Passthru = ConstantInt::getNullValue(CharVTy);

  // Split block in the original loop preheader.
  // SPH is the new preheader to the old scalar loop.
  BasicBlock *SPH = SplitBlock(Preheader, Preheader->getTerminator(), DT, LI,
                               nullptr, "scalar_preheader");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1220-1318

```cpp
  // Create the blocks that we're going to use.
  //
  // We will have the following loops:
  // (O) Outer loop where we iterate over the elements of the search array.
  // (I) Inner loop where we iterate over the elements of the needle array.
  //
  // Overall, the blocks do the following:
  // (0) Check if the arrays can't cross page boundaries. If so go to (1),
  //     otherwise fall back to the original scalar loop.
  // (1) Load the search array. Go to (2).
  // (2) (a) Load the needle array.
  //     (b) Splat the first element to the inactive lanes.
  //     (c) Accumulate any matches found. If we haven't reached the end of the
  //         needle array loop back to (2), otherwise go to (3).
  // (3) Test if we found any match. If so go to (4), otherwise go to (5).
  // (4) Compute the index of the first match and exit.
  // (5) Check if we've reached the end of the search array. If not loop back to
  //     (1), otherwise exit.
  // Blocks (0,4) are not part of any loop. Blocks (1,3,5) and (2) belong to the
  // outer and inner loops, respectively.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  const uint64_t AddrShiftAmt = llvm::Log2_64(MinPageSize);
  Value *SearchStartPage =
      Builder.CreateLShr(ISearchStart, AddrShiftAmt, "search_start_page");
  Value *SearchEndPage =
      Builder.CreateLShr(ISearchEnd, AddrShiftAmt, "search_end_page");
  Value *NeedleStartPage =
      Builder.CreateLShr(INeedleStart, AddrShiftAmt, "needle_start_page");
  Value *NeedleEndPage =
      Builder.CreateLShr(INeedleEnd, AddrShiftAmt, "needle_end_page");
  Value *SearchPageCmp =
      Builder.CreateICmpNE(SearchStartPage, SearchEndPage, "search_page_cmp");
  Value *NeedlePageCmp =
      Builder.CreateICmpNE(NeedleStartPage, NeedleEndPage, "needle_page_cmp");

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1319-1419

```cpp
  Value *CombinedPageCmp =
      Builder.CreateOr(SearchPageCmp, NeedlePageCmp, "combined_page_cmp");
  CondBrInst *CombinedPageBr = Builder.CreateCondBr(CombinedPageCmp, SPH, BB1);
  CombinedPageBr->setMetadata(LLVMContext::MD_prof,
                              MDBuilder(Ctx).createBranchWeights(10, 90));
  DTU.applyUpdates(
      {{DominatorTree::Insert, BB0, SPH}, {DominatorTree::Insert, BB0, BB1}});

  // (1) Load the search array and branch to the inner loop.
  Builder.SetInsertPoint(BB1);
  PHINode *SearchIdx = Builder.CreatePHI(I64Ty, 2, "search_idx");
  Value *PredSearch = Builder.CreateIntrinsic(
      Intrinsic::get_active_lane_mask, {PredVTy, I64Ty},
      {SearchIdx, SearchTripCount}, nullptr, "search_pred");
  PredSearch = Builder.CreateAnd(PredVF, PredSearch, "search_masked");
  Value *Search = Builder.CreateGEP(CharTy, SearchStart, SearchIdx, "psearch");
  Value *LoadSearch = Builder.CreateMaskedLoad(
      CharVTy, Search, Align(1), PredSearch, Passthru, "search_load_vec");
  Value *MatchInit = Constant::getNullValue(PredVTy);
  Builder.CreateBr(BB2);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                    {DominatorTree::Insert, BB5, ExitFail}});

  // Set up the PHI nodes.
  SearchIdx->addIncoming(SearchIdxInit, BB0);
  SearchIdx->addIncoming(NextSearchIdx, BB5);
  NeedleIdx->addIncoming(NeedleIdxInit, BB1);
  NeedleIdx->addIncoming(NextNeedleIdx, BB2);
  Match->addIncoming(MatchInit, BB1);
  Match->addIncoming(MatchAcc, BB2);
  // These are needed to retain LCSSA form.
  MatchPredAccLCSSA->addIncoming(MatchAcc, BB2);
  MatchLCSSA->addIncoming(Search, BB3);
  MatchPredLCSSA->addIncoming(MatchPredAccLCSSA, BB3);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1420-1455

```cpp
  // Ensure all Phis in the successors of BB4/BB5 have an incoming value from
  // them.
  fixSuccessorPhis(CurLoop, IndPhi, MatchVal, ExitSucc, BB4);
  if (ExitSucc != ExitFail)
    fixSuccessorPhis(CurLoop, IndPhi, MatchVal, ExitFail, BB5);

  if (VerifyLoops) {
    OuterLoop->verifyLoop();
    InnerLoop->verifyLoop();
    if (!OuterLoop->isRecursivelyLCSSAForm(*DT, *LI))
      report_fatal_error("Loops must remain in LCSSA form!");
  }

  return MatchVal;
}

void LoopIdiomVectorize::transformFindFirstByte(
    PHINode *IndPhi, unsigned VF, Type *CharTy, BasicBlock *ExitSucc,
    BasicBlock *ExitFail, Value *SearchStart, Value *SearchEnd,
    Value *NeedleStart, Value *NeedleEnd) {
  // Insert the find first byte code at the end of the preheader block.
  BasicBlock *Preheader = CurLoop->getLoopPreheader();
  UncondBrInst *PHBranch = cast<UncondBrInst>(Preheader->getTerminator());
  IRBuilder<> Builder(PHBranch);
  DomTreeUpdater DTU(DT, DomTreeUpdater::UpdateStrategy::Lazy);
  Builder.SetCurrentDebugLocation(PHBranch->getDebugLoc());

  expandFindFirstByte(Builder, DTU, VF, CharTy, IndPhi, ExitSucc, ExitFail,
                      SearchStart, SearchEnd, NeedleStart, NeedleEnd);

  if (VerifyLoops && CurLoop->getParentLoop()) {
    CurLoop->getParentLoop()->verifyLoop();
    if (!CurLoop->getParentLoop()->isRecursivelyLCSSAForm(*DT, *LI))
      report_fatal_error("Loops must remain in LCSSA form!");
  }
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `LoopIdiomVectorize, run, recognizeByteCompare, recognizeFindFirstByte` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LoopIdiomVectorize, run, recognizeByteCompare, recognizeFindFirstByte` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, SCEV, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, SCEV, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `DisableAll, LITVecStyle, DisableByteCmp, ByteCmpVF, DisableFindFirstByte` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `DisableAll, LITVecStyle, DisableByteCmp, ByteCmpVF, DisableFindFirstByte` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopPass.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Vectorize/LoopIdiomVectorize.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Vectorize/LoopIdiomVectorize.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Vectorize/LoopVectorizationLegality.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `SCEV`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `SCEV`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
