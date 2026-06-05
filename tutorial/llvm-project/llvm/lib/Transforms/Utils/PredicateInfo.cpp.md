# PredicateInfo.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/PredicateInfo.cpp` | `llvm/lib/Transforms/Utils/PredicateInfo.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements predicateInfo Builder within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 PredicateInfo 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-81

```cpp
//===-- PredicateInfo.cpp - PredicateInfo Builder--------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------===//
//
// This file implements the PredicateInfo class.
//
//===----------------------------------------------------------------===//

#include "llvm/Transforms/Utils/PredicateInfo.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/Analysis/AssumeBundleQueries.h"
#include "llvm/Analysis/AssumptionCache.h"
#include "llvm/IR/AssemblyAnnotationWriter.h"
#include "llvm/IR/Dominators.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}
}

namespace llvm {
enum LocalNum {
  // Operations that must appear first in the block.
  LN_First,
  // Operations that are somewhere in the middle of the block, and are sorted on
  // demand.
  LN_Middle,
  // Operations that must appear last in a block, like successor phi node uses.
  LN_Last
};

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 82-159

```cpp
// Associate global and local DFS info with defs (PInfo set) and uses (U set),
// so we can sort them into a global domination ordering.
struct ValueDFS {
  int DFSIn = 0;
  int DFSOut = 0;
  unsigned int LocalNum = LN_Middle;
  // Only one of U or PInfo will be set.
  Use *U = nullptr;
  PredicateBase *PInfo = nullptr;
};

// This compares ValueDFS structures. Doing so allows us to walk the minimum
// number of instructions necessary to compute our def/use ordering.
struct ValueDFS_Compare {
  DominatorTree &DT;
  ValueDFS_Compare(DominatorTree &DT) : DT(DT) {}

  bool operator()(const ValueDFS &A, const ValueDFS &B) const {
    if (&A == &B)
      return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

#ifndef NDEBUG
    // This function should only be used for values in the same BB, check that.
    DomTreeNode *DomASrc = DT.getNode(ASrc);
    DomTreeNode *DomBSrc = DT.getNode(BSrc);
    assert(DomASrc->getDFSNumIn() == (unsigned)A.DFSIn &&
           "DFS numbers for A should match the ones of the source block");
    assert(DomBSrc->getDFSNumIn() == (unsigned)B.DFSIn &&
           "DFS numbers for B should match the ones of the source block");
    assert(A.DFSIn == B.DFSIn && "Values must be in the same block");
#endif
    (void)ASrc;
    (void)BSrc;

```
- EN: Core entities appearing here include ValueDFS, ValueDFS_Compare, operator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ValueDFS, ValueDFS_Compare, operator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 160-234

```cpp
    // Use DFS numbers to compare destination blocks, to guarantee a
    // deterministic order.
    DomTreeNode *DomADest = DT.getNode(ADest);
    DomTreeNode *DomBDest = DT.getNode(BDest);
    unsigned AIn = DomADest->getDFSNumIn();
    unsigned BIn = DomBDest->getDFSNumIn();
    bool isAUse = A.U;
    bool isBUse = B.U;
    assert((!A.PInfo || !A.U) && (!B.PInfo || !B.U) &&
           "Def and U cannot be set at the same time");
    // Now sort by edge destination and then defs before uses.
    return std::tie(AIn, isAUse) < std::tie(BIn, isBUse);
  }

  const Instruction *getDefOrUser(const ValueDFS &VD) const {
    if (VD.U)
      return cast<Instruction>(VD.U->getUser());

    // For the purpose of ordering, we pretend the def is right after the
    // assume, because that is where we will insert the info.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  void processAssume(AssumeInst *, BasicBlock *,
                     SmallVectorImpl<Value *> &OpsToRename);
  void processBranch(CondBrInst *, BasicBlock *,
                     SmallVectorImpl<Value *> &OpsToRename);
  void processSwitch(SwitchInst *, BasicBlock *,
                     SmallVectorImpl<Value *> &OpsToRename);
  void renameUses(SmallVectorImpl<Value *> &OpsToRename);
  void addInfoFor(SmallVectorImpl<Value *> &OpsToRename, Value *Op,
                  PredicateBase *PB);

  struct StackEntry {
    const ValueDFS *V;
    Value *Def = nullptr;

```
- EN: Core entities appearing here include localComesBefore, PredicateInfoBuilder, ValueInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 localComesBefore, PredicateInfoBuilder, ValueInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 235-305

```cpp
    StackEntry(const ValueDFS *V) : V(V) {}
  };

  using ValueDFSStack = SmallVectorImpl<StackEntry>;
  void convertUsesToDFSOrdered(Value *, SmallVectorImpl<ValueDFS> &);
  Value *materializeStack(unsigned int &, ValueDFSStack &, Value *);
  bool stackIsInScope(const ValueDFSStack &, const ValueDFS &) const;
  void popStackUntilDFSScope(ValueDFSStack &, const ValueDFS &);

public:
  PredicateInfoBuilder(PredicateInfo &PI, Function &F, DominatorTree &DT,
                       AssumptionCache &AC, BumpPtrAllocator &Allocator)
      : PI(PI), F(F), DT(DT), AC(AC), Allocator(Allocator) {
    // Push an empty operand info so that we can detect 0 as not finding one
    ValueInfos.resize(1);
  }

  void buildPredicateInfo();
};

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Stack.pop_back();
}

// Convert the uses of Op into a vector of uses, associating global and local
// DFS info with each one.
void PredicateInfoBuilder::convertUsesToDFSOrdered(
    Value *Op, SmallVectorImpl<ValueDFS> &DFSOrderedSet) {
  for (auto &U : Op->uses()) {
    if (auto *I = dyn_cast<Instruction>(U.getUser())) {
      // Lifetime intrinsics must work directly on alloca, do not replace them
      // with a predicated copy.
      if (I->isLifetimeStartOrEnd())
        continue;

```
- EN: Core entities appearing here include PI, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 PI，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 306-386

```cpp
      ValueDFS VD;
      // Put the phi node uses in the incoming block.
      BasicBlock *IBlock;
      if (auto *PN = dyn_cast<PHINode>(I)) {
        IBlock = PN->getIncomingBlock(U);
        // Make phi node users appear last in the incoming block
        // they are from.
        VD.LocalNum = LN_Last;
      } else {
        // If it's not a phi node use, it is somewhere in the middle of the
        // block.
        IBlock = I->getParent();
        VD.LocalNum = LN_Middle;
      }
      DomTreeNode *DomNode = DT.getNode(IBlock);
      // It's possible our use is in an unreachable block. Skip it if so.
      if (!DomNode)
        continue;
      VD.DFSIn = DomNode->getDFSNumIn();
      VD.DFSOut = DomNode->getDFSNumOut();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }
    return;
  }

  SmallVector<Value *, 4> Worklist;
  SmallPtrSet<Value *, 4> Visited;
  Worklist.push_back(II->getOperand(0));
  while (!Worklist.empty()) {
    Value *Cond = Worklist.pop_back_val();
    if (!Visited.insert(Cond).second)
      continue;
    if (Visited.size() > MaxCondsPerBranch)
      break;

```
- EN: Core entities appearing here include shouldRename, collectCmpOps, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 shouldRename, collectCmpOps，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 387-466

```cpp
    Value *Op0, *Op1;
    if (match(Cond, m_LogicalAnd(m_Value(Op0), m_Value(Op1)))) {
      Worklist.push_back(Op1);
      Worklist.push_back(Op0);
    }

    SmallVector<Value *, 4> Values;
    Values.push_back(Cond);
    if (auto *Cmp = dyn_cast<CmpInst>(Cond))
      collectCmpOps(Cmp, Values);
    else if (match(Cond, m_NUWTrunc(m_Value(Op0))))
      Values.push_back(Op0);

    for (Value *V : Values) {
      if (shouldRename(V)) {
        auto *PA = new (Allocator) PredicateConditionAssume(V, II, Cond);
        addInfoFor(OpsToRename, V, PA);
      }
    }
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        }
      }
    }
  }
}
// Process a block terminating switch, and place relevant operations to be
// renamed into OpsToRename.
void PredicateInfoBuilder::processSwitch(
    SwitchInst *SI, BasicBlock *BranchBB,
    SmallVectorImpl<Value *> &OpsToRename) {
  Value *Op = SI->getCondition();
  if ((!isa<Instruction>(Op) && !isa<Argument>(Op)) || Op->hasOneUse())
    return;

```
- EN: Core entities appearing here include match, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 467-540

```cpp
  // Remember how many outgoing edges there are to every successor.
  SmallDenseMap<BasicBlock *, unsigned, 16> SwitchEdges;
  for (BasicBlock *TargetBlock : successors(BranchBB))
    ++SwitchEdges[TargetBlock];

  // Now propagate info for each case value
  for (auto C : SI->cases()) {
    BasicBlock *TargetBlock = C.getCaseSuccessor();
    if (SwitchEdges.lookup(TargetBlock) == 1) {
      PredicateSwitch *PS = new (Allocator) PredicateSwitch(
          Op, SI->getParent(), TargetBlock, C.getCaseValue(), SI);
      addInfoFor(OpsToRename, Op, PS);
    }
  }
}

// Build predicate info for our function
void PredicateInfoBuilder::buildPredicateInfo() {
  DT.updateDFSNumbers();
  // Collect operands to rename from all conditional branch terminators, as well
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
       RenameIter != RenameStack.end(); ++RenameIter) {
    auto *Op =
        RenameIter == RenameStack.begin() ? OrigOp : (RenameIter - 1)->Def;
    StackEntry &Result = *RenameIter;
    auto *ValInfo = Result.V->PInfo;
    ValInfo->RenamedOp = (RenameStack.end() - Start) == RenameStack.begin()
                             ? OrigOp
                             : (RenameStack.end() - Start - 1)->Def;
    auto CreateSSACopy = [](Instruction *InsertPt, Value *Op,
                            const Twine &Name = "") {
      // Use a no-op bitcast to represent ssa copy.
      return new BitCastInst(Op, Op->getType(), Name, InsertPt->getIterator());
    };
    // For edge predicates, we can just place the operand in the block before
```
- EN: Core entities appearing here include buildPredicateInfo, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 buildPredicateInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 541-614

```cpp
    // the terminator. For assume, we have to place it right after the assume
    // to ensure we dominate all uses except assume itself. Always insert
    // right before the terminator or after the assume, so that we insert in
    // proper order in the case of multiple predicateinfo in the same block.
    if (isa<PredicateWithEdge>(ValInfo)) {
      BitCastInst *PIC = CreateSSACopy(getBranchTerminator(ValInfo), Op,
                                       Op->getName() + "." + Twine(Counter++));
      PI.PredicateMap.insert({PIC, ValInfo});
      Result.Def = PIC;
    } else {
      auto *PAssume = dyn_cast<PredicateAssume>(ValInfo);
      assert(PAssume &&
             "Should not have gotten here without it being an assume");
      // Insert the predicate directly after the assume. While it also holds
      // directly before it, assume(i1 true) is not a useful fact.
      BitCastInst *PIC = CreateSSACopy(PAssume->AssumeInst->getNextNode(), Op);
      PI.PredicateMap.insert({PIC, ValInfo});
      Result.Def = PIC;
    }
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (const auto *PAssume = dyn_cast<PredicateAssume>(PossibleCopy)) {
        VD.LocalNum = LN_Middle;
        DomTreeNode *DomNode = DT.getNode(PAssume->AssumeInst->getParent());
        if (!DomNode)
          continue;
        VD.DFSIn = DomNode->getDFSNumIn();
        VD.DFSOut = DomNode->getDFSNumOut();
        VD.PInfo = PossibleCopy;
        OrderedUses.push_back(VD);
      } else if (isa<PredicateWithEdge>(PossibleCopy)) {
        // If we can only do phi uses, we treat it like it's in the branch
        // block, and handle it specially. We know that it goes last, and only
        // dominate phi uses.
        auto BlockEdge = getBlockEdge(PossibleCopy);
```
- EN: Core entities appearing here include renameUses, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 renameUses，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 615-697

```cpp
        if (!BlockEdge.second->getSinglePredecessor()) {
          VD.LocalNum = LN_Last;
          auto *DomNode = DT.getNode(BlockEdge.first);
          if (DomNode) {
            VD.DFSIn = DomNode->getDFSNumIn();
            VD.DFSOut = DomNode->getDFSNumOut();
            VD.PInfo = PossibleCopy;
            OrderedUses.push_back(VD);
          }
        } else {
          // Otherwise, we are in the split block (even though we perform
          // insertion in the branch block).
          // Insert a possible copy at the split block and before the branch.
          VD.LocalNum = LN_First;
          auto *DomNode = DT.getNode(BlockEdge.second);
          if (DomNode) {
            VD.DFSIn = DomNode->getDFSNumIn();
            VD.DFSOut = DomNode->getDFSNumOut();
            VD.PInfo = PossibleCopy;
            OrderedUses.push_back(VD);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // ends up with predicateinfo.
      if (!Result.Def)
        Result.Def = materializeStack(Counter, RenameStack, Op);

      LLVM_DEBUG(dbgs() << "Found replacement " << *Result.Def << " for "
                        << *VD.U->get() << " in " << *(VD.U->getUser())
                        << "\n");
      assert(DT.dominates(cast<Instruction>(Result.Def), *VD.U) &&
             "Predicateinfo def should have dominated this use");
      VD.U->set(Result.Def);
    }
  }
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 698-772

```cpp
PredicateInfoBuilder::ValueInfo &
PredicateInfoBuilder::getOrCreateValueInfo(Value *Operand) {
  auto Res = ValueInfoNums.try_emplace(Operand, ValueInfos.size());
  if (Res.second) {
    // Allocate space for new ValueInfo.
    ValueInfos.resize(ValueInfos.size() + 1);
  }
  return ValueInfos[Res.first->second];
}

const PredicateInfoBuilder::ValueInfo &
PredicateInfoBuilder::getValueInfo(Value *Operand) const {
  auto OINI = ValueInfoNums.lookup(Operand);
  assert(OINI != 0 && "Operand was not really in the Value Info Numbers");
  assert(OINI < ValueInfos.size() &&
         "Value Info Number greater than size of Value Info Table");
  return ValueInfos[OINI];
}

PredicateInfo::PredicateInfo(Function &F, DominatorTree &DT,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Pred = Cmp->getPredicate();
      OtherOp = Cmp->getOperand(1);
    } else if (Cmp->getOperand(1) == RenamedOp) {
      Pred = Cmp->getSwappedPredicate();
      OtherOp = Cmp->getOperand(0);
    } else {
      // TODO: Make this an assertion once RenamedOp is fully accurate.
      return std::nullopt;
    }

    // Invert predicate along false edge.
    if (!TrueEdge)
      Pred = CmpInst::getInversePredicate(Pred);

```
- EN: Core entities appearing here include getOrCreateValueInfo, getValueInfo, F, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getOrCreateValueInfo, getValueInfo, F，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 773-846

```cpp
    return {{Pred, OtherOp}};
  }
  case PT_Switch:
    if (Condition != RenamedOp) {
      // TODO: Make this an assertion once RenamedOp is fully accurate.
      return std::nullopt;
    }

    return {{CmpInst::ICMP_EQ, cast<PredicateSwitch>(this)->CaseValue}};
  }
  llvm_unreachable("Unknown predicate type");
}

void PredicateInfo::verifyPredicateInfo() const {}

// Replace bitcasts created by PredicateInfo with their operand.
static void replaceCreatedSSACopys(PredicateInfo &PredInfo, Function &F) {
  for (Instruction &Inst : llvm::make_early_inc_range(instructions(F))) {
    const auto *PI = PredInfo.getPredicateInfoFor(&Inst);
    if (!PI)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        PB->From->printAsOperand(OS);
        OS << ",";
        PB->To->printAsOperand(OS);
        OS << "]";
      } else if (const auto *PS = dyn_cast<PredicateSwitch>(PI)) {
        OS << "; switch predicate info { CaseValue: " << *PS->CaseValue
           << " Edge: [";
        PS->From->printAsOperand(OS);
        OS << ",";
        PS->To->printAsOperand(OS);
        OS << "]";
      } else if (const auto *PA = dyn_cast<PredicateAssume>(PI)) {
        OS << "; assume predicate info {";
        if (auto *PBA = dyn_cast<PredicateBundleAssume>(PA)) {
```
- EN: Core entities appearing here include replaceCreatedSSACopys, PredicateInfoAnnotatedWriter, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 replaceCreatedSSACopys, PredicateInfoAnnotatedWriter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 847-879

```cpp
          OS << " Attribute: " << Attribute::getNameFromAttrKind(PBA->AttrKind);
        } else {
          assert(isa<PredicateConditionAssume>(PA));
          OS << " Comparison:" << *PA->Condition;
        }
      }
      OS << ", RenamedOp: ";
      PI->RenamedOp->printAsOperand(OS, false);
      OS << " }\n";
    }
  }
};

void PredicateInfo::print(raw_ostream &OS) const {
  PredicateInfoAnnotatedWriter Writer(this);
  F.print(OS, &Writer);
}

void PredicateInfo::dump() const {
  PredicateInfoAnnotatedWriter Writer(this);
  F.print(dbgs(), &Writer);
}

PreservedAnalyses PredicateInfoVerifierPass::run(Function &F,
                                                 FunctionAnalysisManager &AM) {
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
  auto &AC = AM.getResult<AssumptionAnalysis>(F);
  BumpPtrAllocator Allocator;
  std::make_unique<PredicateInfo>(F, DT, AC, Allocator)->verifyPredicateInfo();

  return PreservedAnalyses::all();
}
}
```
- EN: Core entities appearing here include print, dump, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 print, dump，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LocalNum, ValueDFS, ValueDFS_Compare, PredicateInfoBuilder, ValueInfo, StackEntry, PredicateInfoAnnotatedWriter, getBlockEdge` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LocalNum, ValueDFS, ValueDFS_Compare, PredicateInfoBuilder, ValueInfo, StackEntry, PredicateInfoAnnotatedWriter, getBlockEdge` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, DominatorTree` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, DominatorTree` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `VerifyPredicateInfo` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `VerifyPredicateInfo` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/AssumptionCache.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AssumeBundleQueries.h`, `llvm/Analysis/AssumptionCache.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/AssemblyAnnotationWriter.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/PredicateInfo.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/AssemblyAnnotationWriter.h`, `llvm/IR/Dominators.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstIterator.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/PredicateInfo.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/FormattedStream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/FormattedStream.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallPtrSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `AssumptionCache`, `DominatorTree` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `DominatorTree` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
