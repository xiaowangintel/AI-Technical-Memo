# StraightLineStrengthReduce.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/StraightLineStrengthReduce.cpp` | `llvm/lib/Transforms/Scalar/StraightLineStrengthReduce.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Straight Line Strength Reduce within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 StraightLineStrengthReduce 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-85

```cpp
//===- StraightLineStrengthReduce.cpp - -----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements straight-line strength reduction (SLSR). Unlike loop
// strength reduction, this algorithm is designed to reduce arithmetic
// redundancy in straight-line code instead of loops. It has proven to be
// effective in simplifying arithmetic statements derived from an unrolled loop.
// It can also simplify the logic of SeparateConstOffsetFromGEP.
//
// There are many optimizations we can perform in the domain of SLSR.
// We look for strength reduction candidates in the following forms:
//
// Form Add: B + i * S
// Form Mul: (B + i) * S
// Form GEP: &B[i * S]
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/GetElementPtrTypeIterator.h"
#include "llvm/IR/IRBuilder.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, ScalarEvolution, TargetTransformInfo, ValueTracking. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, ScalarEvolution, TargetTransformInfo, ValueTracking 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 86-174

```cpp
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Operator.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Value.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/DebugCounter.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/Local.h"
#include <cassert>
#include <cstdint>
#include <limits>
#include <list>
#include <queue>
#include <vector>
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    enum Kind {
      Invalid, // reserved for the default constructor
      Add,     // B + i * S
      Mul,     // (B + i) * S
      GEP,     // &B[..][i * S][..]
    };

    enum DKind {
      InvalidDelta, // reserved for the default constructor
      IndexDelta,   // Delta is a constant from Index
      BaseDelta,    // Delta is a constant or variable from Base
      StrideDelta,  // Delta is a constant or variable from Stride
    };

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 175-263

```cpp
    Candidate() = default;
    Candidate(Kind CT, const SCEV *B, ConstantInt *Idx, Value *S,
              Instruction *I, const SCEV *StrideSCEV)
        : CandidateKind(CT), Base(B), Index(Idx), Stride(S), Ins(I),
          StrideSCEV(StrideSCEV) {}

    Kind CandidateKind = Invalid;

    const SCEV *Base = nullptr;
    // TODO: Swap Index and Stride's name.
    // Note that Index and Stride of a GEP candidate do not necessarily have the
    // same integer type. In that case, during rewriting, Stride will be
    // sign-extended or truncated to Index's type.
    ConstantInt *Index = nullptr;

    Value *Stride = nullptr;

    // The instruction this candidate corresponds to. It helps us to rewrite a
    // candidate with respect to its immediate basis. Note that one instruction
    // can correspond to multiple candidates depending on how you associate the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      bool IsZeroStride =
          IsConstantStride && cast<ConstantInt>(Stride)->isZero();
      // All constants
      if (IsConstantBase && IsConstantStride)
        return ZeroInst;

      // (Base + Index) * Stride
      if (CandidateKind == Mul) {
        if (IsZeroStride)
          return ZeroInst;
        if (Index->isZero())
          return (IsConstantStride || IsConstantBase) ? OneInstOneVar
                                                      : OneInstTwoVar;

```
- EN: Core entities appearing here include EfficiencyLevel, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 EfficiencyLevel，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 264-360

```cpp
        if (IsConstantBase)
          return IsZeroBase && (Index->isOne() || Index->isMinusOne())
                     ? ZeroInst
                     : OneInstOneVar;

        if (IsConstantStride) {
          auto *CI = cast<ConstantInt>(Stride);
          return (CI->isOne() || CI->isMinusOne()) ? OneInstOneVar
                                                   : TwoInstOneVar;
        }
        return TwoInstTwoVar;
      }

      // Base + Index * Stride
      assert(CandidateKind == Add || CandidateKind == GEP);
      if (Index->isZero() || IsZeroStride)
        return ZeroInst;

      bool IsSimpleIndex = Index->isOne() || Index->isMinusOne();

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        // Index differs, Base and Stride must match
        return Base == Basis.Base && StrideSCEV == Basis.StrideSCEV;
      case StrideDelta:
        // Stride differs, Base and Index must match
        return Base == Basis.Base && Index == Basis.Index;
      case BaseDelta:
        // Base differs, Stride and Index must match
        return StrideSCEV == Basis.StrideSCEV && Index == Basis.Index;
      default:
        return false;
      }
    }
  };

```
- EN: Core entities appearing here include isProfitableRewrite, getRewriteEfficiency, isHighEfficiency, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isProfitableRewrite, getRewriteEfficiency, isHighEfficiency，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 361-448

```cpp
  bool runOnFunction(Function &F);

private:
  // Fetch straight-line basis for rewriting C, update C.Basis to point to it,
  // and store the delta between C and its Basis in C.Delta.
  void setBasisAndDeltaFor(Candidate &C);
  // Returns whether the candidate can be folded into an addressing mode.
  bool isFoldable(const Candidate &C, TargetTransformInfo *TTI);

  // Checks whether I is in a candidate form. If so, adds all the matching forms
  // to Candidates, and tries to find the immediate basis for each of them.
  void allocateCandidatesAndFindBasis(Instruction *I);

  // Allocate candidates and find bases for Add instructions.
  void allocateCandidatesAndFindBasisForAdd(Instruction *I);

  // Given I = LHS + RHS, factors RHS into i * S and makes (LHS + i * S) a
  // candidate.
  void allocateCandidatesAndFindBasisForAdd(Value *LHS, Value *RHS,
                                            Instruction *I);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  private:
    // Index delta Basis must have the same (Base, StrideSCEV, Inst.Type)
    using IndexDeltaKeyTy = std::tuple<const SCEV *, const SCEV *, Type *>;
    DenseMap<IndexDeltaKeyTy, BBToCandsTy> IndexDeltaCandidates;

    // Base delta Basis must have the same (StrideSCEV, Index, Inst.Type)
    using BaseDeltaKeyTy = std::tuple<const SCEV *, ConstantInt *, Type *>;
    DenseMap<BaseDeltaKeyTy, BBToCandsTy> BaseDeltaCandidates;

    // Stride delta Basis must have the same (Base, Index, Inst.Type)
    using StrideDeltaKeyTy = std::tuple<const SCEV *, ConstantInt *, Type *>;
    DenseMap<StrideDeltaKeyTy, BBToCandsTy> StrideDeltaCandidates;

```
- EN: Core entities appearing here include CandidateDictTy, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, ScalarEvolution, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 CandidateDictTy，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, ScalarEvolution, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 449-537

```cpp
  public:
    // TODO: Disable index delta on GEP after we completely move
    // from typed GEP to PtrAdd.
    const BBToCandsTy *getCandidatesWithDeltaKind(const Candidate &C,
                                                  Candidate::DKind K) const {
      assert(K != Candidate::InvalidDelta);
      if (K == Candidate::IndexDelta) {
        IndexDeltaKeyTy IndexDeltaKey(C.Base, C.StrideSCEV, C.Ins->getType());
        auto It = IndexDeltaCandidates.find(IndexDeltaKey);
        if (It != IndexDeltaCandidates.end())
          return &It->second;
      } else if (K == Candidate::BaseDelta) {
        BaseDeltaKeyTy BaseDeltaKey(C.StrideSCEV, C.Index, C.Ins->getType());
        auto It = BaseDeltaCandidates.find(BaseDeltaKey);
        if (It != BaseDeltaCandidates.end())
          return &It->second;
      } else {
        assert(K == Candidate::StrideDelta);
        StrideDeltaKeyTy StrideDeltaKey(C.Base, C.Index, C.Ins->getType());
        auto It = StrideDeltaCandidates.find(StrideDeltaKey);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Instructions are sorted in depth-first order, so search for the nearest
    // instruction by walking the list in reverse order.
    for (Instruction *I : reverse(It->second))
      if (DT->dominates(I, CI))
        return I;

    return nullptr;
  }

  struct DeltaInfo {
    Candidate *Cand;
    Candidate::DKind DeltaKind;
    Value *Delta;

```
- EN: Core entities appearing here include add, clear, DeltaInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 add, clear, DeltaInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 538-625

```cpp
    DeltaInfo()
        : Cand(nullptr), DeltaKind(Candidate::InvalidDelta), Delta(nullptr) {}
    DeltaInfo(Candidate *Cand, Candidate::DKind DeltaKind, Value *Delta)
        : Cand(Cand), DeltaKind(DeltaKind), Delta(Delta) {}
    operator bool() const { return Cand != nullptr; }
  };

  friend raw_ostream &operator<<(raw_ostream &OS, const DeltaInfo &DI);

  DeltaInfo compressPath(Candidate &C, Candidate *Basis) const;

  Candidate *pickRewriteCandidate(Instruction *I) const;
  void sortCandidateInstructions();
  Value *getDelta(const Candidate &C, const Candidate &Basis,
                  Candidate::DKind K) const;
  static bool isSimilar(Candidate &C, Candidate &Basis, Candidate::DKind K);

  // Add Basis -> C in DependencyGraph and propagate
  // C.Stride and C.Delta's dependency to C
  void addDependency(Candidate &C, Candidate *Basis) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

} // end anonymous namespace

char StraightLineStrengthReduceLegacyPass::ID = 0;

INITIALIZE_PASS_BEGIN(StraightLineStrengthReduceLegacyPass, "slsr",
                      "Straight line strength reduction", false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
INITIALIZE_PASS_DEPENDENCY(TargetTransformInfoWrapperPass)
INITIALIZE_PASS_END(StraightLineStrengthReduceLegacyPass, "slsr",
                    "Straight line strength reduction", false, false)

```
- EN: Core entities appearing here include addDependency, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo.
- CN: 此处出现的核心实体包括 addDependency，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo 等分析结果。

### Lines 626-713

```cpp
FunctionPass *llvm::createStraightLineStrengthReducePass() {
  return new StraightLineStrengthReduceLegacyPass();
}

// A helper function that unifies the bitwidth of A and B.
static void unifyBitWidth(APInt &A, APInt &B) {
  if (A.getBitWidth() < B.getBitWidth())
    A = A.sext(B.getBitWidth());
  else if (A.getBitWidth() > B.getBitWidth())
    B = B.sext(A.getBitWidth());
}

Value *StraightLineStrengthReduce::getDelta(const Candidate &C,
                                            const Candidate &Basis,
                                            Candidate::DKind K) const {
  if (K == Candidate::IndexDelta) {
    APInt Idx = C.Index->getValue();
    APInt BasisIdx = Basis.Index->getValue();
    unifyBitWidth(Idx, BasisIdx);
    APInt IndexDelta = Idx - BasisIdx;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Y = B + S,
  // rewriting Y to X - 7 * S is probably a bad idea.
  // So, we need to check if the rewrite form's computation efficiency
  // is better than the original form.
  if (K == Candidate::IndexDelta &&
      !C.isProfitableRewrite(*Delta, Candidate::IndexDelta))
    return false;

  // If there is a Delta that we can reuse Basis to rewrite C,
  // clean up DropPoisonGeneratingInsts returned by successful
  // SE->canReuseInstruction()
  for (Instruction *I : DropPoisonGeneratingInsts)
    I->dropPoisonGeneratingAnnotations();

```
- EN: Core entities appearing here include unifyBitWidth, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 unifyBitWidth，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 714-810

```cpp
  // Record delta if none has been found yet, or the new delta is
  // a constant that is better than the existing delta.
  if (!C.Delta || isa<ConstantInt>(Delta)) {
    C.Delta = Delta;
    C.Basis = Basis;
    C.DeltaKind = K;
  }
  return isa<ConstantInt>(C.Delta);
}

// return true if find a Basis with constant delta and stop searching,
// return false if did not find a Basis or the delta is not a constant
// and continue searching for a Basis with constant delta
bool StraightLineStrengthReduce::searchFrom(
    const CandidateDictTy::BBToCandsTy &BBToCands, Candidate &C,
    Candidate::DKind K) {

  // Stride delta rewrite on Mul form is usually non-profitable, and Base
  // delta rewrite sometimes is profitable, so we do not support them on Mul.
  if (C.CandidateKind == Candidate::Mul && K != Candidate::IndexDelta)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// ->
// X = A + 1
// Y = A + 2
// Z = A + 3
// Return the delta info for C aginst the new Basis
auto StraightLineStrengthReduce::compressPath(Candidate &C,
                                              Candidate *Basis) const
    -> DeltaInfo {
  if (!Basis || !Basis->Basis || C.CandidateKind == Candidate::Mul)
    return {};
  Candidate *Root = Basis;
  Value *NewDelta = nullptr;
  auto NewKind = Candidate::InvalidDelta;

```
- EN: Core entities appearing here include setBasisAndDeltaFor, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 setBasisAndDeltaFor，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 811-901

```cpp
  while (Root->Basis) {
    Candidate *NextRoot = Root->Basis;
    if (C.Base == NextRoot->Base && C.StrideSCEV == NextRoot->StrideSCEV &&
        isSimilar(C, *NextRoot, Candidate::IndexDelta)) {
      ConstantInt *CI =
          cast<ConstantInt>(getDelta(C, *NextRoot, Candidate::IndexDelta));
      if (CI->isZero() || CI->isOne() || isa<SCEVConstant>(C.StrideSCEV)) {
        Root = NextRoot;
        NewKind = Candidate::IndexDelta;
        NewDelta = CI;
        continue;
      }
    }

    const SCEV *CandPart = nullptr;
    const SCEV *BasisPart = nullptr;
    auto CurrKind = Candidate::InvalidDelta;
    if (C.Base == NextRoot->Base && C.Index == NextRoot->Index) {
      CandPart = C.StrideSCEV;
      BasisPart = NextRoot->StrideSCEV;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Instruction *I = WorkList.front();
    WorkList.pop();
    if (!Visited.insert(I).second)
      continue;

    SortedCandidateInsts.push_back(I);

    for (auto *Next : DependencyGraph[I]) {
      auto &Degree = InDegree[Next];
      if (--Degree == 0)
        WorkList.push(Next);
    }
  }

```
- EN: Core entities appearing here include isSimilar, sortCandidateInstructions, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 isSimilar, sortCandidateInstructions，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 902-986

```cpp
  assert(SortedCandidateInsts.size() == DependencyGraph.size() &&
         "Dependency graph should not have cycles");
}

auto StraightLineStrengthReduce::pickRewriteCandidate(Instruction *I) const
    -> Candidate * {
  // Return the candidate of instruction I that has the highest profit.
  auto It = RewriteCandidates.find(I);
  if (It == RewriteCandidates.end())
    return nullptr;

  Candidate *BestC = nullptr;
  auto BestEfficiency = Candidate::Unknown;
  for (Candidate *C : reverse(It->second))
    if (C->Basis) {
      auto Efficiency = C->getRewriteEfficiency();
      if (Efficiency > BestEfficiency) {
        BestEfficiency = Efficiency;
        BestC = C;
      }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (auto Res = compressPath(C, C.Basis)) {
      C.Basis = Res.Cand;
      C.DeltaKind = Res.DeltaKind;
      C.Delta = Res.Delta;
    }
  }
  // Regardless of whether we find a basis for C, we need to push C to the
  // candidate list so that it can be the basis of other candidates.
  LLVM_DEBUG(dbgs() << "Allocated Candidate: " << C << "\n");
  Candidates.push_back(C);
  RewriteCandidates[C.Ins].push_back(&Candidates.back());
  CandidateDict.add(Candidates.back());
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV, TargetTransformInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 SCEV, TargetTransformInfo 等分析结果。

### Lines 987-1073

```cpp
void StraightLineStrengthReduce::allocateCandidatesAndFindBasis(
    Instruction *I) {
  switch (I->getOpcode()) {
  case Instruction::Add:
    allocateCandidatesAndFindBasisForAdd(I);
    break;
  case Instruction::Mul:
    allocateCandidatesAndFindBasisForMul(I);
    break;
  case Instruction::GetElementPtr:
    allocateCandidatesAndFindBasisForGEP(cast<GetElementPtrInst>(I));
    break;
  }
}

void StraightLineStrengthReduce::allocateCandidatesAndFindBasisForAdd(
    Instruction *I) {
  // Try matching B + i * S.
  if (!isa<IntegerType>(I->getType()))
    return;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Otherwise, at least try the form (LHS + 0) * RHS.
    ConstantInt *Zero = ConstantInt::get(cast<IntegerType>(I->getType()), 0);
    allocateCandidatesAndFindBasis(Candidate::Mul, SE->getSCEV(LHS), Zero, RHS,
                                   I);
  }
}

void StraightLineStrengthReduce::allocateCandidatesAndFindBasisForMul(
    Instruction *I) {
  // Try matching (B + i) * S.
  // TODO: we could extend SLSR to float and vector types.
  if (!isa<IntegerType>(I->getType()))
    return;

```
- EN: Core entities appearing here include matchesAdd, matchesOr, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as SCEV.
- CN: 此处出现的核心实体包括 matchesAdd, matchesOr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 SCEV 等分析结果。

### Lines 1074-1164

```cpp
  assert(I->getNumOperands() == 2 && "isn't I a mul?");
  Value *LHS = I->getOperand(0), *RHS = I->getOperand(1);
  allocateCandidatesAndFindBasisForMul(LHS, RHS, I);
  if (LHS != RHS) {
    // Symmetrically, try to split RHS to Base + Index.
    allocateCandidatesAndFindBasisForMul(RHS, LHS, I);
  }
}

void StraightLineStrengthReduce::allocateCandidatesAndFindBasisForGEP(
    GetElementPtrInst *GEP) {
  // TODO: handle vector GEPs
  if (GEP->getType()->isVectorTy())
    return;

  SmallVector<SCEVUse, 4> IndexExprs;
  for (Use &Idx : GEP->indices())
    IndexExprs.push_back(SE->getSCEV(Idx));

  gep_type_iterator GTI = gep_type_begin(GEP);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        return Builder.CreateNeg(Builder.CreateShl(LHS, Exponent));
      }
    }

    return Builder.CreateMul(LHS, RHS);
  };

  Value *Delta = C.Delta;
  // If Delta is 0, C is a fully redundant of C.Basis,
  // just replace C.Ins with Basis.Ins
  if (ConstantInt *CI = dyn_cast<ConstantInt>(Delta);
      CI && CI->getValue().isZero())
    return nullptr;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, SCEV. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, SCEV 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1165-1249

```cpp
  if (C.DeltaKind == Candidate::IndexDelta) {
    APInt IndexDelta = cast<ConstantInt>(C.Delta)->getValue();
    // IndexDelta
    // X = B + i * S
    // Y = B + i` * S
    //   = B + (i + IndexDelta) * S
    //   = B + i * S + IndexDelta * S
    //   = X + IndexDelta * S
    // Bump = (i' - i) * S

    // Common case 1: if (i' - i) is 1, Bump = S.
    if (IndexDelta == 1)
      return C.Stride;
    // Common case 2: if (i' - i) is -1, Bump = -S.
    if (IndexDelta.isAllOnes())
      return Builder.CreateNeg(C.Stride);

    IntegerType *DeltaType =
        IntegerType::get(Basis.Ins->getContext(), IndexDelta.getBitWidth());
    Value *ExtendedStride = Builder.CreateSExtOrTrunc(C.Stride, DeltaType);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // If delta is 0, C is a fully redundant of Basis, and Bump is nullptr,
  // just replace C.Ins with Basis.Ins
  if (!Bump)
    Reduced = Basis.Ins;
  else {
    switch (C.CandidateKind) {
    case Candidate::Add:
    case Candidate::Mul: {
      // C = Basis + Bump
      Value *NegBump;
      if (match(Bump, m_Neg(m_Value(NegBump)))) {
        // If Bump is a neg instruction, emit C = Basis - (-Bump).
        Reduced = Builder.CreateSub(Basis.Ins, NegBump);
        // We only use the negative argument of Bump, and Bump itself may be
```
- EN: Core entities appearing here include rewriteCandidate, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 rewriteCandidate，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1250-1337

```cpp
        // trivially dead.
        RecursivelyDeleteTriviallyDeadInstructions(Bump);
      } else {
        // It's tempting to preserve nsw on Bump and/or Reduced. However, it's
        // usually unsound, e.g.,
        //
        // X = (-2 +nsw 1) *nsw INT_MAX
        // Y = (-2 +nsw 3) *nsw INT_MAX
        //   =>
        // Y = X + 2 * INT_MAX
        //
        // Neither + and * in the resultant expression are nsw.
        Reduced = Builder.CreateAdd(Basis.Ins, Bump);
      }
      break;
    }
    case Candidate::GEP: {
      bool InBounds = cast<GetElementPtrInst>(C.Ins)->isInBounds();
      // C = (char *)Basis + Bump
      Reduced = Builder.CreatePtrAdd(Basis.Ins, Bump, "", InBounds);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // First clear all references to candidates in the list
  CandidateDict.clear();
  // Then destroy the list
  Candidates.clear();
  return Ret;
}

PreservedAnalyses
StraightLineStrengthReducePass::run(Function &F, FunctionAnalysisManager &AM) {
  const DataLayout *DL = &F.getDataLayout();
  auto *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  auto *SE = &AM.getResult<ScalarEvolutionAnalysis>(F);
  auto *TTI = &AM.getResult<TargetIRAnalysis>(F);

```
- EN: Core entities appearing here include runOnFunction, run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, ScalarEvolution, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runOnFunction, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, ScalarEvolution, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1338-1347

```cpp
  if (!StraightLineStrengthReduce(DL, DT, SE, TTI).runOnFunction(F))
    return PreservedAnalyses::all();

  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  PA.preserve<DominatorTreeAnalysis>();
  PA.preserve<ScalarEvolutionAnalysis>();
  PA.preserve<TargetIRAnalysis>();
  return PA;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `StraightLineStrengthReduceLegacyPass, StraightLineStrengthReduce, Candidate, Kind, DKind, EfficiencyLevel, CandidateDictTy, DeltaInfo` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`StraightLineStrengthReduceLegacyPass, StraightLineStrengthReduce, Candidate, Kind, DKind, EfficiencyLevel, CandidateDictTy, DeltaInfo` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, ScalarEvolution, SCEV, TargetTransformInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnablePoisonReuseGuard` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnablePoisonReuseGuard` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/ScalarEvolutionExpressions.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/IR/Operator.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/Type.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/IR/Operator.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/Type.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint`, `limits`, `list`, `queue`, `vector` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint`, `limits`, `list`, `queue`, `vector` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `ScalarEvolution`, `SCEV`, `TargetTransformInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
