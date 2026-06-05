# AArch64PromoteConstant.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64PromoteConstant.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements the AArch64PromoteConstant pass which promotes constants to global variables when this is likely to be more efficient. Currently only types related to constant vector (i.e., constant vector, array of constant vectors, constant structure with a constant vector field, etc.) are promoted to global variables. Constant vectors are likely to be lowered in target constant pool during instruction selection already; therefore, the access will remain the same (memory load), but the structure types are not split into different constant pool accesses for each field. A bonus side effect is that created globals may be merged by the global merge pass. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Documented code section
```cpp
//==- AArch64PromoteConstant.cpp - Promote constant to global for AArch64 --==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AArch64PromoteConstant pass which promotes constants
// to global variables when this is likely to be more efficient. Currently only
// types related to constant vector (i.e., constant vector, array of constant
// vectors, constant structure with a constant vector field, etc.) are promoted
// to global variables. Constant vectors are likely to be lowered in target
// constant pool during instruction selection already; therefore, the access
// will remain the same (memory load), but the structure types are not split
// into different constant pool accesses for each field. A bonus side effect is
// that created globals may be merged by the global merge pass.
//
// FIXME: This pass may be useful for other targets too.
//===----------------------------------------------------------------------===//
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 21-55: Includes and compile-time dependencies
```cpp

#include "AArch64.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constant.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Type.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <utility>

using namespace llvm;

#define DEBUG_TYPE "aarch64-promote-const"

// Stress testing mode - disable heuristics.
static cl::opt<bool> Stress("aarch64-stress-promote-const", cl::Hidden,
                            cl::desc("Promote all vector constants"));
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 56-91: Core AArch64 backend logic
```cpp

STATISTIC(NumPromoted, "Number of promoted constants");
STATISTIC(NumPromotedUses, "Number of promoted constants uses");

//===----------------------------------------------------------------------===//
//                       AArch64PromoteConstant
//===----------------------------------------------------------------------===//

namespace {

/// Promotes interesting constant into global variables.
/// The motivating example is:
/// static const uint16_t TableA[32] = {
///   41944, 40330, 38837, 37450, 36158, 34953, 33826, 32768,
///   31776, 30841, 29960, 29128, 28340, 27595, 26887, 26215,
///   25576, 24967, 24386, 23832, 23302, 22796, 22311, 21846,
///   21400, 20972, 20561, 20165, 19785, 19419, 19066, 18725,
/// };
///
/// uint8x16x4_t LoadStatic(void) {
///   uint8x16x4_t ret;
///   ret.val[0] = vld1q_u16(TableA +  0);
///   ret.val[1] = vld1q_u16(TableA +  8);
///   ret.val[2] = vld1q_u16(TableA + 16);
///   ret.val[3] = vld1q_u16(TableA + 24);
///   return ret;
/// }
///
/// The constants in this example are folded into the uses. Thus, 4 different
/// constants are created.
///
/// As their type is vector the cheapest way to create them is to load them
/// for the memory.
///
/// Therefore the final assembly final has 4 different loads. With this pass
/// enabled, only one load is issued for the constants.
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 92-113: Class AArch64PromoteConstant
```cpp
class AArch64PromoteConstant : public ModulePass {
public:
  struct PromotedConstant {
    bool ShouldConvert = false;
    GlobalVariable *GV = nullptr;
  };
  using PromotionCacheTy = SmallDenseMap<Constant *, PromotedConstant, 16>;

  struct UpdateRecord {
    Constant *C;
    Instruction *User;
    unsigned Op;

    UpdateRecord(Constant *C, Instruction *User, unsigned Op)
        : C(C), User(User), Op(Op) {}
  };

  static char ID;

  AArch64PromoteConstant() : ModulePass(ID) {}

  StringRef getPassName() const override { return "AArch64 Promote Constant"; }
```
**EN:** This block defines AArch64PromoteConstant, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64PromoteConstant，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 114-148: Documented code section
```cpp

  /// Iterate over the functions and promote the interesting constants into
  /// global variables with module scope.
  bool runOnModule(Module &M) override {
    LLVM_DEBUG(dbgs() << getPassName() << '\n');
    if (skipModule(M))
      return false;
    bool Changed = false;
    PromotionCacheTy PromotionCache;
    for (auto &MF : M) {
      Changed |= runOnFunction(MF, PromotionCache);
    }
    return Changed;
  }

private:
  /// Look for interesting constants used within the given function.
  /// Promote them into global variables, load these global variables within
  /// the related function, so that the number of inserted load is minimal.
  bool runOnFunction(Function &F, PromotionCacheTy &PromotionCache);

  // This transformation requires dominator info
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.addPreserved<DominatorTreeWrapperPass>();
  }

  /// Type to store a list of Uses.
  using Uses = SmallVector<std::pair<Instruction *, unsigned>, 4>;
  /// Map an insertion point to all the uses it dominates.
  using InsertionPoints = DenseMap<Instruction *, Uses>;

  /// Find the closest point that dominates the given Use.
  Instruction *findInsertionPoint(Instruction &User, unsigned OpNo);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 149-178: Documented code section
```cpp

  /// Check if the given insertion point is dominated by an existing
  /// insertion point.
  /// If true, the given use is added to the list of dominated uses for
  /// the related existing point.
  /// \param NewPt the insertion point to be checked
  /// \param User the user of the constant
  /// \param OpNo the operand number of the use
  /// \param InsertPts existing insertion points
  /// \pre NewPt and all instruction in InsertPts belong to the same function
  /// \return true if one of the insertion point in InsertPts dominates NewPt,
  ///         false otherwise
  bool isDominated(Instruction *NewPt, Instruction *User, unsigned OpNo,
                   InsertionPoints &InsertPts);

  /// Check if the given insertion point can be merged with an existing
  /// insertion point in a common dominator.
  /// If true, the given use is added to the list of the created insertion
  /// point.
  /// \param NewPt the insertion point to be checked
  /// \param User the user of the constant
  /// \param OpNo the operand number of the use
  /// \param InsertPts existing insertion points
  /// \pre NewPt and all instruction in InsertPts belong to the same function
  /// \pre isDominated returns false for the exact same parameters.
  /// \return true if it exists an insertion point in InsertPts that could
  ///         have been merged with NewPt in a common dominator,
  ///         false otherwise
  bool tryAndMerge(Instruction *NewPt, Instruction *User, unsigned OpNo,
                   InsertionPoints &InsertPts);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 179-199: Documented code section
```cpp

  /// Compute the minimal insertion points to dominates all the interesting
  /// uses of value.
  /// Insertion points are group per function and each insertion point
  /// contains a list of all the uses it dominates within the related function
  /// \param User the user of the constant
  /// \param OpNo the operand number of the constant
  /// \param[out] InsertPts output storage of the analysis
  void computeInsertionPoint(Instruction *User, unsigned OpNo,
                             InsertionPoints &InsertPts);

  /// Insert a definition of a new global variable at each point contained in
  /// InsPtsPerFunc and update the related uses (also contained in
  /// InsPtsPerFunc).
  void insertDefinitions(Function &F, GlobalVariable &GV,
                         InsertionPoints &InsertPts);

  /// Do the constant promotion indicated by the Updates records, keeping track
  /// of globals in PromotionCache.
  void promoteConstants(Function &F, SmallVectorImpl<UpdateRecord> &Updates,
                        PromotionCacheTy &PromotionCache);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 200-233: Function appendAndTransferDominatedUses
```cpp

  /// Transfer the list of dominated uses of IPI to NewPt in InsertPts.
  /// Append Use to this list and delete the entry of IPI in InsertPts.
  static void appendAndTransferDominatedUses(Instruction *NewPt,
                                             Instruction *User, unsigned OpNo,
                                             InsertionPoints::iterator &IPI,
                                             InsertionPoints &InsertPts) {
    // Record the dominated use.
    IPI->second.emplace_back(User, OpNo);
    // Transfer the dominated uses of IPI to NewPt
    // Inserting into the DenseMap may invalidate existing iterator.
    // Keep a copy of the key to find the iterator to erase.  Keep a copy of the
    // value so that we don't have to dereference IPI->second.
    Instruction *OldInstr = IPI->first;
    Uses OldUses = std::move(IPI->second);
    InsertPts[NewPt] = std::move(OldUses);
    // Erase IPI.
    InsertPts.erase(OldInstr);
  }
};

} // end anonymous namespace

char AArch64PromoteConstant::ID = 0;

INITIALIZE_PASS_BEGIN(AArch64PromoteConstant, "aarch64-promote-const",
                      "AArch64 Promote Constant Pass", false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_END(AArch64PromoteConstant, "aarch64-promote-const",
                    "AArch64 Promote Constant Pass", false, false)

ModulePass *llvm::createAArch64PromoteConstantPass() {
  return new AArch64PromoteConstant();
}
```
**EN:** This block implements appendAndTransferDominatedUses, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 appendAndTransferDominatedUses，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 234-261: Function isConstantUsingVectorTy
```cpp

/// Check if the given type uses a vector type.
static bool isConstantUsingVectorTy(const Type *CstTy) {
  if (CstTy->isVectorTy())
    return true;
  if (CstTy->isStructTy()) {
    for (unsigned EltIdx = 0, EndEltIdx = CstTy->getStructNumElements();
         EltIdx < EndEltIdx; ++EltIdx)
      if (isConstantUsingVectorTy(CstTy->getStructElementType(EltIdx)))
        return true;
  } else if (CstTy->isArrayTy())
    return isConstantUsingVectorTy(CstTy->getArrayElementType());
  return false;
}

// Returns true if \p C contains only ConstantData leaves and no global values,
// block addresses or constant expressions. Traverses ConstantAggregates.
static bool containsOnlyConstantData(const Constant *C) {
  if (isa<ConstantData>(C))
    return true;

  if (isa<GlobalValue>(C) || isa<BlockAddress>(C) || isa<ConstantExpr>(C))
    return false;

  return all_of(C->operands(), [](const Use &U) {
    return containsOnlyConstantData(cast<Constant>(&U));
  });
}
```
**EN:** This block implements isConstantUsingVectorTy, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isConstantUsingVectorTy，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 262-296: Function use
```cpp

/// Check if the given use (Instruction + OpIdx) of Cst should be converted into
/// a load of a global variable initialized with Cst.
/// A use should be converted if it is legal to do so.
/// For instance, it is not legal to turn the mask operand of a shuffle vector
/// into a load of a global variable.
static bool shouldConvertUse(const Constant *Cst, const Instruction *Instr,
                             unsigned OpIdx) {
  // shufflevector instruction expects a const for the mask argument, i.e., the
  // third argument. Do not promote this use in that case.
  if (isa<const ShuffleVectorInst>(Instr) && OpIdx == 2)
    return false;

  // extractvalue instruction expects a const idx.
  if (isa<const ExtractValueInst>(Instr) && OpIdx > 0)
    return false;

  // extractvalue instruction expects a const idx.
  if (isa<const InsertValueInst>(Instr) && OpIdx > 1)
    return false;

  if (isa<const AllocaInst>(Instr) && OpIdx > 0)
    return false;

  // Alignment argument must be constant.
  if (isa<const LoadInst>(Instr) && OpIdx > 0)
    return false;

  // Alignment argument must be constant.
  if (isa<const StoreInst>(Instr) && OpIdx > 1)
    return false;

  // Index must be constant.
  if (isa<const GetElementPtrInst>(Instr) && OpIdx > 0)
    return false;
```
**EN:** This block implements use, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 use，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 297-318: Documented code section
```cpp

  // Personality function and filters must be constant.
  // Give up on that instruction.
  if (isa<const LandingPadInst>(Instr))
    return false;

  // Switch instruction expects constants to compare to.
  if (isa<const SwitchInst>(Instr))
    return false;

  // Expected address must be a constant.
  if (isa<const IndirectBrInst>(Instr))
    return false;

  // Do not mess with intrinsics.
  if (isa<const IntrinsicInst>(Instr))
    return false;

  // Do not mess with inline asm.
  const CallInst *CI = dyn_cast<const CallInst>(Instr);
  return !(CI && CI->isInlineAsm());
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 319-353: Documented code section
```cpp

/// Check if the given Cst should be converted into
/// a load of a global variable initialized with Cst.
/// A constant should be converted if it is likely that the materialization of
/// the constant will be tricky. Thus, we give up on zero or undef values.
///
/// \todo Currently, accept only vector related types.
/// Also we give up on all simple vector type to keep the existing
/// behavior. Otherwise, we should push here all the check of the lowering of
/// BUILD_VECTOR. By giving up, we lose the potential benefit of merging
/// constant via global merge and the fact that the same constant is stored
/// only once with this method (versus, as many function that uses the constant
/// for the regular approach, even for float).
/// Again, the simplest solution would be to promote every
/// constant and rematerialize them when they are actually cheap to create.
static bool shouldConvertImpl(const Constant *Cst) {
  if (isa<const UndefValue>(Cst))
    return false;

  // FIXME: In some cases, it may be interesting to promote in memory
  // a zero initialized constant.
  // E.g., when the type of Cst require more instructions than the
  // adrp/add/load sequence or when this sequence can be shared by several
  // instances of Cst.
  // Ideally, we could promote this into a global and rematerialize the constant
  // when it was a bad idea.
  if (Cst->isNullValue())
    return false;

  // Globals cannot be or contain scalable vectors.
  if (Cst->getType()->isScalableTy())
    return false;

  if (Stress)
    return true;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 354-385: Function shouldConvert
```cpp

  // FIXME: see function \todo
  if (Cst->getType()->isVectorTy())
    return false;
  return isConstantUsingVectorTy(Cst->getType());
}

static bool
shouldConvert(Constant &C,
              AArch64PromoteConstant::PromotionCacheTy &PromotionCache) {
  auto Converted = PromotionCache.insert(
      std::make_pair(&C, AArch64PromoteConstant::PromotedConstant()));
  if (Converted.second)
    Converted.first->second.ShouldConvert = shouldConvertImpl(&C);
  return Converted.first->second.ShouldConvert;
}

Instruction *AArch64PromoteConstant::findInsertionPoint(Instruction &User,
                                                        unsigned OpNo) {
  // If this user is a phi, the insertion point is in the related
  // incoming basic block.
  if (PHINode *PhiInst = dyn_cast<PHINode>(&User))
    return PhiInst->getIncomingBlock(OpNo)->getTerminator();

  return &User;
}

bool AArch64PromoteConstant::isDominated(Instruction *NewPt, Instruction *User,
                                         unsigned OpNo,
                                         InsertionPoints &InsertPts) {
  DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>(
      *NewPt->getParent()->getParent()).getDomTree();
```
**EN:** This block implements shouldConvert, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 shouldConvert，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 386-412: Documented code section
```cpp

  // Traverse all the existing insertion points and check if one is dominating
  // NewPt. If it is, remember that.
  for (auto &IPI : InsertPts) {
    if (NewPt == IPI.first || DT.dominates(IPI.first, NewPt) ||
        // When IPI.first is a terminator instruction, DT may think that
        // the result is defined on the edge.
        // Here we are testing the insertion point, not the definition.
        (IPI.first->getParent() != NewPt->getParent() &&
         DT.dominates(IPI.first->getParent(), NewPt->getParent()))) {
      // No need to insert this point. Just record the dominated use.
      LLVM_DEBUG(dbgs() << "Insertion point dominated by:\n");
      LLVM_DEBUG(IPI.first->print(dbgs()));
      LLVM_DEBUG(dbgs() << '\n');
      IPI.second.emplace_back(User, OpNo);
      return true;
    }
  }
  return false;
}

bool AArch64PromoteConstant::tryAndMerge(Instruction *NewPt, Instruction *User,
                                         unsigned OpNo,
                                         InsertionPoints &InsertPts) {
  DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>(
      *NewPt->getParent()->getParent()).getDomTree();
  BasicBlock *NewBB = NewPt->getParent();
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 413-436: Documented code section
```cpp

  // Traverse all the existing insertion point and check if one is dominated by
  // NewPt and thus useless or can be combined with NewPt into a common
  // dominator.
  for (InsertionPoints::iterator IPI = InsertPts.begin(),
                                 EndIPI = InsertPts.end();
       IPI != EndIPI; ++IPI) {
    BasicBlock *CurBB = IPI->first->getParent();
    if (NewBB == CurBB) {
      // Instructions are in the same block.
      // By construction, NewPt is dominating the other.
      // Indeed, isDominated returned false with the exact same arguments.
      LLVM_DEBUG(dbgs() << "Merge insertion point with:\n");
      LLVM_DEBUG(IPI->first->print(dbgs()));
      LLVM_DEBUG(dbgs() << "\nat considered insertion point.\n");
      appendAndTransferDominatedUses(NewPt, User, OpNo, IPI, InsertPts);
      return true;
    }

    // Look for a common dominator
    BasicBlock *CommonDominator = DT.findNearestCommonDominator(NewBB, CurBB);
    // If none exists, we cannot merge these two points.
    if (!CommonDominator)
      continue;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 437-468: Core AArch64 backend logic
```cpp

    if (CommonDominator != NewBB) {
      // By construction, the CommonDominator cannot be CurBB.
      assert(CommonDominator != CurBB &&
             "Instruction has not been rejected during isDominated check!");
      // Take the last instruction of the CommonDominator as insertion point
      NewPt = CommonDominator->getTerminator();
    }
    // else, CommonDominator is the block of NewBB, hence NewBB is the last
    // possible insertion point in that block.
    LLVM_DEBUG(dbgs() << "Merge insertion point with:\n");
    LLVM_DEBUG(IPI->first->print(dbgs()));
    LLVM_DEBUG(dbgs() << '\n');
    LLVM_DEBUG(NewPt->print(dbgs()));
    LLVM_DEBUG(dbgs() << '\n');
    appendAndTransferDominatedUses(NewPt, User, OpNo, IPI, InsertPts);
    return true;
  }
  return false;
}

void AArch64PromoteConstant::computeInsertionPoint(
    Instruction *User, unsigned OpNo, InsertionPoints &InsertPts) {
  LLVM_DEBUG(dbgs() << "Considered use, opidx " << OpNo << ":\n");
  LLVM_DEBUG(User->print(dbgs()));
  LLVM_DEBUG(dbgs() << '\n');

  Instruction *InsertionPoint = findInsertionPoint(*User, OpNo);

  LLVM_DEBUG(dbgs() << "Considered insertion point:\n");
  LLVM_DEBUG(InsertionPoint->print(dbgs()));
  LLVM_DEBUG(dbgs() << '\n');
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 469-497: Core AArch64 backend logic
```cpp

  if (isDominated(InsertionPoint, User, OpNo, InsertPts))
    return;
  // This insertion point is useful, check if we can merge some insertion
  // point in a common dominator or if NewPt dominates an existing one.
  if (tryAndMerge(InsertionPoint, User, OpNo, InsertPts))
    return;

  LLVM_DEBUG(dbgs() << "Keep considered insertion point\n");

  // It is definitely useful by its own
  InsertPts[InsertionPoint].emplace_back(User, OpNo);
}

static void ensurePromotedGV(Function &F, Constant &C,
                             AArch64PromoteConstant::PromotedConstant &PC) {
  assert(PC.ShouldConvert &&
         "Expected that we should convert this to a global");
  if (PC.GV)
    return;
  PC.GV = new GlobalVariable(
      *F.getParent(), C.getType(), true, GlobalValue::InternalLinkage, nullptr,
      "_PromotedConst", nullptr, GlobalVariable::NotThreadLocal);
  PC.GV->setInitializer(&C);
  LLVM_DEBUG(dbgs() << "Global replacement: ");
  LLVM_DEBUG(PC.GV->print(dbgs()));
  LLVM_DEBUG(dbgs() << '\n');
  ++NumPromoted;
}
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 498-516: Function AArch64PromoteConstant::insertDefinitions
```cpp

void AArch64PromoteConstant::insertDefinitions(Function &F,
                                               GlobalVariable &PromotedGV,
                                               InsertionPoints &InsertPts) {
#ifndef NDEBUG
  // Do more checking for debug purposes.
  DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>(F).getDomTree();
#endif
  assert(!InsertPts.empty() && "Empty uses does not need a definition");

  for (const auto &IPI : InsertPts) {
    // Create the load of the global variable.
    IRBuilder<> Builder(IPI.first);
    LoadInst *LoadedCst =
        Builder.CreateLoad(PromotedGV.getValueType(), &PromotedGV);
    LLVM_DEBUG(dbgs() << "**********\n");
    LLVM_DEBUG(dbgs() << "New def: ");
    LLVM_DEBUG(LoadedCst->print(dbgs()));
    LLVM_DEBUG(dbgs() << '\n');
```
**EN:** This block implements AArch64PromoteConstant::insertDefinitions, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64PromoteConstant::insertDefinitions，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 517-547: Documented code section
```cpp

    // Update the dominated uses.
    for (auto Use : IPI.second) {
#ifndef NDEBUG
      assert(DT.dominates(LoadedCst,
                          findInsertionPoint(*Use.first, Use.second)) &&
             "Inserted definition does not dominate all its uses!");
#endif
      LLVM_DEBUG({
        dbgs() << "Use to update " << Use.second << ":";
        Use.first->print(dbgs());
        dbgs() << '\n';
      });
      Use.first->setOperand(Use.second, LoadedCst);
      ++NumPromotedUses;
    }
  }
}

void AArch64PromoteConstant::promoteConstants(
    Function &F, SmallVectorImpl<UpdateRecord> &Updates,
    PromotionCacheTy &PromotionCache) {
  // Promote the constants.
  for (auto U = Updates.begin(), E = Updates.end(); U != E;) {
    LLVM_DEBUG(dbgs() << "** Compute insertion points **\n");
    auto First = U;
    Constant *C = First->C;
    InsertionPoints InsertPts;
    do {
      computeInsertionPoint(U->User, U->Op, InsertPts);
    } while (++U != E && U->C == C);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 548-580: Function AArch64PromoteConstant::runOnFunction
```cpp

    auto &Promotion = PromotionCache[C];
    ensurePromotedGV(F, *C, Promotion);
    insertDefinitions(F, *Promotion.GV, InsertPts);
  }
}

bool AArch64PromoteConstant::runOnFunction(Function &F,
                                           PromotionCacheTy &PromotionCache) {
  // Look for instructions using constant vector. Promote that constant to a
  // global variable. Create as few loads of this variable as possible and
  // update the uses accordingly.
  SmallVector<UpdateRecord, 64> Updates;
  for (Instruction &I : instructions(&F)) {
    // Traverse the operand, looking for constant vectors. Replace them by a
    // load of a global variable of constant vector type.
    for (Use &U : I.operands()) {
      Constant *Cst = dyn_cast<Constant>(U);
      // There is no point in promoting global values as they are already
      // global. Do not promote constants containing constant expression, global
      // values or blockaddresses either, as they may require some code
      // expansion.
      if (!Cst || isa<GlobalValue>(Cst) || !containsOnlyConstantData(Cst))
        continue;

      // Check if this constant is worth promoting.
      if (!shouldConvert(*Cst, PromotionCache))
        continue;

      // Check if this use should be promoted.
      unsigned OpNo = &U - I.op_begin();
      if (!shouldConvertUse(Cst, &I, OpNo))
        continue;
```
**EN:** This block implements AArch64PromoteConstant::runOnFunction, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64PromoteConstant::runOnFunction，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 581-591: Core AArch64 backend logic
```cpp

      Updates.emplace_back(Cst, &I, OpNo);
    }
  }

  if (Updates.empty())
    return false;

  promoteConstants(F, Updates, PromotionCache);
  return true;
}
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h **CN:** 目标本地依赖：AArch64.h
- **EN:** Core LLVM interfaces: llvm/ADT/DenseMap.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/IR/BasicBlock.h, llvm/IR/Constant.h, llvm/IR/Constants.h, llvm/IR/Dominators.h, llvm/IR/Function.h, llvm/IR/GlobalValue.h, llvm/IR/GlobalVariable.h ... **CN:** 核心 LLVM 接口：llvm/ADT/DenseMap.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/IR/BasicBlock.h, llvm/IR/Constant.h, llvm/IR/Constants.h, llvm/IR/Dominators.h, llvm/IR/Function.h, llvm/IR/GlobalValue.h, llvm/IR/GlobalVariable.h ...
- **EN:** Standard-library support: cassert, utility **CN:** 标准库支持：cassert, utility
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
