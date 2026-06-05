# PPCLoopInstrFormPrep.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCLoopInstrFormPrep.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCLoopInstrFormPrep.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===------ PPCLoopInstrFormPrep.cpp - Loop Instr Form Prep Pass ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-44

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements a pass to prepare loops for ppc preferred addressing
// modes, leveraging different instruction form. (eg: DS/DQ form, D/DS form with
// update)
// Additional PHIs are created for loop induction variables used by load/store
// instructions so that preferred addressing modes can be used.
//
// 1: DS/DQ form preparation, prepare the load/store instructions so that they
//    can satisfy the DS/DQ form displacement requirements.
//    Generically, this means transforming loops like this:
//    for (int i = 0; i < n; ++i) {
//      unsigned long x1 = *(unsigned long *)(p + i + 5);
//      unsigned long x2 = *(unsigned long *)(p + i + 9);
//    }
//
//    to look like this:
//
//    unsigned NewP = p + 5;
//    for (int i = 0; i < n; ++i) {
//      unsigned long x1 = *(unsigned long *)(i + NewP);
//      unsigned long x2 = *(unsigned long *)(i + NewP + 4);
//    }
//
// 2: D/DS form with update preparation, prepare the load/store instructions so
//    that we can use update form to do pre-increment.
//    Generically, this means transforming loops like this:
//    for (int i = 0; i < n; ++i)
//      array[i] = c;
//
//    to look like this:
//
//    T *p = array[-1];
//    for (int i = 0; i < n; ++i)
//      *++p = c;
//
// 3: common multiple chains for the load/stores with same offsets in the loop,
//    so that we can reuse the offsets and reduce the register pressure in the
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements a pass to prepare loops for ppc preferred addressing".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements a pass to prepare loops for ppc preferred addressing”。

### Lines 45-75

```cpp
//    loop. This transformation can also increase the loop ILP as now each chain
//    uses its own loop induction add/addi. But this will increase the number of
//    add/addi in the loop.
//
//    Generically, this means transforming loops like this:
//
//    char *p;
//    A1 = p + base1
//    A2 = p + base1 + offset
//    B1 = p + base2
//    B2 = p + base2 + offset
//
//    for (int i = 0; i < n; i++)
//      unsigned long x1 = *(unsigned long *)(A1 + i);
//      unsigned long x2 = *(unsigned long *)(A2 + i)
//      unsigned long x3 = *(unsigned long *)(B1 + i);
//      unsigned long x4 = *(unsigned long *)(B2 + i);
//    }
//
//    to look like this:
//
//    A1_new = p + base1 // chain 1
//    B1_new = p + base2 // chain 2, now inside the loop, common offset is
//                       // reused.
//
//    for (long long i = 0; i < n; i+=count) {
//      unsigned long x1 = *(unsigned long *)(A1_new + i);
//      unsigned long x2 = *(unsigned long *)((A1_new + i) + offset);
//      unsigned long x3 = *(unsigned long *)(B1_new + i);
//      unsigned long x4 = *(unsigned long *)((B1_new + i) + offset);
//    }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "loop. This transformation can also increase the loop ILP as now each chain".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“loop. This transformation can also increase the loop ILP as now each chain”。

### Lines 76-82

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCSubtarget.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/DepthFirstIterator.h"
#include "llvm/ADT/SmallPtrSet.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 83-89

```cpp
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/CFG.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 90-96

```cpp
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/IntrinsicsPowerPC.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Value.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 97-103

```cpp
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 104-111

```cpp
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Utils/ScalarEvolutionExpander.h"
#include <cassert>
#include <cmath>
#include <utility>
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 112-149

```cpp
#define DEBUG_TYPE "ppc-loop-instr-form-prep"

using namespace llvm;

static cl::opt<unsigned>
    MaxVarsPrep("ppc-formprep-max-vars", cl::Hidden, cl::init(24),
                cl::desc("Potential common base number threshold per function "
                         "for PPC loop prep"));

static cl::opt<bool> PreferUpdateForm("ppc-formprep-prefer-update",
                                 cl::init(true), cl::Hidden,
  cl::desc("prefer update form when ds form is also a update form"));

static cl::opt<bool> EnableUpdateFormForNonConstInc(
    "ppc-formprep-update-nonconst-inc", cl::init(false), cl::Hidden,
    cl::desc("prepare update form when the load/store increment is a loop "
             "invariant non-const value."));

static cl::opt<bool> EnableChainCommoning(
    "ppc-formprep-chain-commoning", cl::init(false), cl::Hidden,
    cl::desc("Enable chain commoning in PPC loop prepare pass."));

// Sum of following 3 per loop thresholds for all loops can not be larger
// than MaxVarsPrep.
// now the thresholds for each kind prep are exterimental values on Power9.
static cl::opt<unsigned> MaxVarsUpdateForm("ppc-preinc-prep-max-vars",
                                 cl::Hidden, cl::init(3),
  cl::desc("Potential PHI threshold per loop for PPC loop prep of update "
           "form"));

static cl::opt<unsigned> MaxVarsDSForm("ppc-dsprep-max-vars",
                                 cl::Hidden, cl::init(3),
  cl::desc("Potential PHI threshold per loop for PPC loop prep of DS form"));

static cl::opt<unsigned> MaxVarsDQForm("ppc-dqprep-max-vars",
                                 cl::Hidden, cl::init(8),
  cl::desc("Potential PHI threshold per loop for PPC loop prep of DQ form"));
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `MaxVarsPrep`, `init`, `desc`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `MaxVarsPrep`, `init`, `desc`。

### Lines 150-182

```cpp
// Commoning chain will reduce the register pressure, so we don't consider about
// the PHI nodes number.
// But commoning chain will increase the addi/add number in the loop and also
// increase loop ILP. Maximum chain number should be same with hardware
// IssueWidth, because we won't benefit from ILP if the parallel chains number
// is bigger than IssueWidth. We assume there are 2 chains in one bucket, so
// there would be 4 buckets at most on P9(IssueWidth is 8).
static cl::opt<unsigned> MaxVarsChainCommon(
    "ppc-chaincommon-max-vars", cl::Hidden, cl::init(4),
    cl::desc("Bucket number per loop for PPC loop chain common"));

// If would not be profitable if the common base has only one load/store, ISEL
// should already be able to choose best load/store form based on offset for
// single load/store. Set minimal profitable value default to 2 and make it as
// an option.
static cl::opt<unsigned> DispFormPrepMinThreshold("ppc-dispprep-min-threshold",
                                    cl::Hidden, cl::init(2),
  cl::desc("Minimal common base load/store instructions triggering DS/DQ form "
           "preparation"));

static cl::opt<unsigned> ChainCommonPrepMinThreshold(
    "ppc-chaincommon-min-threshold", cl::Hidden, cl::init(4),
    cl::desc("Minimal common base load/store instructions triggering chain "
             "commoning preparation. Must be not smaller than 4"));

STATISTIC(PHINodeAlreadyExistsUpdate, "PHI node already in pre-increment form");
STATISTIC(PHINodeAlreadyExistsDS, "PHI node already in DS form");
STATISTIC(PHINodeAlreadyExistsDQ, "PHI node already in DQ form");
STATISTIC(DSFormChainRewritten, "Num of DS form chain rewritten");
STATISTIC(DQFormChainRewritten, "Num of DQ form chain rewritten");
STATISTIC(UpdFormChainRewritten, "Num of update form chain rewritten");
STATISTIC(ChainCommoningRewritten, "Num of commoning chains");
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Commoning chain will reduce the register pressure, so we don't consider about". Notable symbols in this range include `P9`, `MaxVarsChainCommon`, `init`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Commoning chain will reduce the register pressure, so we don't consider about”。 该区间中较显眼的符号包括 `P9`, `MaxVarsChainCommon`, `init`。

### Lines 183-191

```cpp
namespace {
  struct BucketElement {
    BucketElement(const SCEV *O, Instruction *I) : Offset(O), Instr(I) {}
    BucketElement(Instruction *I) : Offset(nullptr), Instr(I) {}

    const SCEV *Offset;
    Instruction *Instr;
  };
```
- **EN**: Implements helper routine(s) `BucketElement`, `Offset`, `Instr` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `BucketElement`, `Offset`, `Instr`。

### Lines 192-216

```cpp
  struct Bucket {
    Bucket(const SCEV *B, Instruction *I)
        : BaseSCEV(B), Elements(1, BucketElement(I)) {
      ChainSize = 0;
    }

    // The base of the whole bucket.
    const SCEV *BaseSCEV;

    // All elements in the bucket. In the bucket, the element with the BaseSCEV
    // has no offset and all other elements are stored as offsets to the
    // BaseSCEV.
    SmallVector<BucketElement, 16> Elements;

    // The potential chains size. This is used for chain commoning only.
    unsigned ChainSize;

    // The base for each potential chain. This is used for chain commoning only.
    SmallVector<BucketElement, 16> ChainBases;
  };

  // "UpdateForm" is not a real PPC instruction form, it stands for dform
  // load/store with update like ldu/stdu, or Prefetch intrinsic.
  // For DS form instructions, their displacements must be multiple of 4.
  // For DQ form instructions, their displacements must be multiple of 16.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The base of the whole bucket.". Notable symbols in this range include `Bucket`, `BaseSCEV`, `Elements`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The base of the whole bucket.”。 该区间中较显眼的符号包括 `Bucket`, `BaseSCEV`, `Elements`。

### Lines 217-254

```cpp
  enum PrepForm { UpdateForm = 1, DSForm = 4, DQForm = 16, ChainCommoning };

  class PPCLoopInstrFormPrep : public FunctionPass {
  public:
    static char ID; // Pass ID, replacement for typeid

    PPCLoopInstrFormPrep(PPCTargetMachine &TM) : FunctionPass(ID), TM(&TM) {}

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.addPreserved<DominatorTreeWrapperPass>();
      AU.addRequired<LoopInfoWrapperPass>();
      AU.addPreserved<LoopInfoWrapperPass>();
      AU.addRequired<ScalarEvolutionWrapperPass>();
    }

    bool runOnFunction(Function &F) override;

  private:
    PPCTargetMachine *TM = nullptr;
    const PPCSubtarget *ST;
    DominatorTree *DT;
    LoopInfo *LI;
    ScalarEvolution *SE;
    bool PreserveLCSSA;
    bool HasCandidateForPrepare;

    /// Successful preparation number for Update/DS/DQ form in all inner most
    /// loops. One successful preparation will put one common base out of loop,
    /// this may leads to register presure like LICM does.
    /// Make sure total preparation number can be controlled by option.
    unsigned SuccPrepCount;

    bool runOnLoop(Loop *L);

    /// Check if required PHI node is already exist in Loop \p L.
    bool alreadyPrepared(Loop *L, Instruction *MemI,
                         const SCEV *BasePtrStartSCEV,
                         const SCEV *BasePtrIncSCEV, PrepForm Form);
```
- **EN**: Implements helper routine(s) `PPCLoopInstrFormPrep`, `FunctionPass`, `TM` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `PPCLoopInstrFormPrep`, `FunctionPass`, `TM`。 子目标特性裁剪会影响这里的行为。

### Lines 255-292

```cpp

    /// Get the value which defines the increment SCEV \p BasePtrIncSCEV.
    Value *getNodeForInc(Loop *L, Instruction *MemI,
                         const SCEV *BasePtrIncSCEV);

    /// Common chains to reuse offsets for a loop to reduce register pressure.
    bool chainCommoning(Loop *L, SmallVector<Bucket, 16> &Buckets);

    /// Find out the potential commoning chains and their bases.
    bool prepareBasesForCommoningChains(Bucket &BucketChain);

    /// Rewrite load/store according to the common chains.
    bool rewriteLoadStoresForCommoningChains(
        Loop *L, Bucket &Bucket, SmallPtrSet<BasicBlock *, 16> &BBChanged);

    /// Collect condition matched(\p isValidCandidate() returns true)
    /// candidates in Loop \p L.
    SmallVector<Bucket, 16> collectCandidates(
        Loop *L,
        std::function<bool(const Instruction *, Value *, const Type *)>
            isValidCandidate,
        std::function<bool(const SCEV *)> isValidDiff,
        unsigned MaxCandidateNum);

    /// Add a candidate to candidates \p Buckets if diff between candidate and
    /// one base in \p Buckets matches \p isValidDiff.
    void addOneCandidate(Instruction *MemI, const SCEV *LSCEV,
                         SmallVector<Bucket, 16> &Buckets,
                         std::function<bool(const SCEV *)> isValidDiff,
                         unsigned MaxCandidateNum);

    /// Prepare all candidates in \p Buckets for update form.
    bool updateFormPrep(Loop *L, SmallVector<Bucket, 16> &Buckets);

    /// Prepare all candidates in \p Buckets for displacement form, now for
    /// ds/dq.
    bool dispFormPrep(Loop *L, SmallVector<Bucket, 16> &Buckets, PrepForm Form);
```
- **EN**: Declares function entry points including `getNodeForInc`, `chainCommoning`, `prepareBasesForCommoningChains` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getNodeForInc`, `chainCommoning`, `prepareBasesForCommoningChains`。

### Lines 293-330

```cpp
    /// Prepare for one chain \p BucketChain, find the best base element and
    /// update all other elements in \p BucketChain accordingly.
    /// \p Form is used to find the best base element.
    /// If success, best base element must be stored as the first element of
    /// \p BucketChain.
    /// Return false if no base element found, otherwise return true.
    bool prepareBaseForDispFormChain(Bucket &BucketChain, PrepForm Form);

    /// Prepare for one chain \p BucketChain, find the best base element and
    /// update all other elements in \p BucketChain accordingly.
    /// If success, best base element must be stored as the first element of
    /// \p BucketChain.
    /// Return false if no base element found, otherwise return true.
    bool prepareBaseForUpdateFormChain(Bucket &BucketChain);

    /// Rewrite load/store instructions in \p BucketChain according to
    /// preparation.
    bool rewriteLoadStores(Loop *L, Bucket &BucketChain,
                           SmallPtrSet<BasicBlock *, 16> &BBChanged,
                           PrepForm Form);

    /// Rewrite for the base load/store of a chain.
    std::pair<Instruction *, Instruction *>
    rewriteForBase(Loop *L, const SCEVAddRecExpr *BasePtrSCEV,
                   Instruction *BaseMemI, bool CanPreInc, PrepForm Form,
                   SCEVExpander &SCEVE, SmallPtrSet<Value *, 16> &DeletedPtrs);

    /// Rewrite for the other load/stores of a chain according to the new \p
    /// Base.
    Instruction *
    rewriteForBucketElement(std::pair<Instruction *, Instruction *> Base,
                            const BucketElement &Element, Value *OffToBase,
                            SmallPtrSet<Value *, 16> &DeletedPtrs);
  };

} // end anonymous namespace

char PPCLoopInstrFormPrep::ID = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Prepare for one chain \p BucketChain, find the best base element and". Notable symbols in this range include `prepareBaseForDispFormChain`, `prepareBaseForUpdateFormChain`, `rewriteLoadStores`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Prepare for one chain \p BucketChain, find the best base element and”。 该区间中较显眼的符号包括 `prepareBaseForDispFormChain`, `prepareBaseForUpdateFormChain`, `rewriteLoadStores`。

### Lines 331-368

```cpp
static const char *name = "Prepare loop for ppc preferred instruction forms";
INITIALIZE_PASS_BEGIN(PPCLoopInstrFormPrep, DEBUG_TYPE, name, false, false)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
INITIALIZE_PASS_END(PPCLoopInstrFormPrep, DEBUG_TYPE, name, false, false)

static constexpr StringRef PHINodeNameSuffix    = ".phi";
static constexpr StringRef CastNodeNameSuffix   = ".cast";
static constexpr StringRef GEPNodeIncNameSuffix = ".inc";
static constexpr StringRef GEPNodeOffNameSuffix = ".off";

FunctionPass *llvm::createPPCLoopInstrFormPrepPass(PPCTargetMachine &TM) {
  return new PPCLoopInstrFormPrep(TM);
}

static bool IsPtrInBounds(Value *BasePtr) {
  Value *StrippedBasePtr = BasePtr;
  while (BitCastInst *BC = dyn_cast<BitCastInst>(StrippedBasePtr))
    StrippedBasePtr = BC->getOperand(0);
  if (GetElementPtrInst *GEP = dyn_cast<GetElementPtrInst>(StrippedBasePtr))
    return GEP->isInBounds();

  return false;
}

static std::string getInstrName(const Value *I, StringRef Suffix) {
  assert(I && "Invalid paramater!");
  if (I->hasName())
    return (I->getName() + Suffix).str();
  else
    return "";
}

static Value *getPointerOperandAndType(Value *MemI,
                                       Type **PtrElementType = nullptr) {

  Value *PtrValue = nullptr;
  Type *PointerElementType = nullptr;
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS_BEGIN`, `INITIALIZE_PASS_DEPENDENCY`, `INITIALIZE_PASS_END` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `INITIALIZE_PASS_BEGIN`, `INITIALIZE_PASS_DEPENDENCY`, `INITIALIZE_PASS_END`。

### Lines 369-406

```cpp

  if (LoadInst *LMemI = dyn_cast<LoadInst>(MemI)) {
    PtrValue = LMemI->getPointerOperand();
    PointerElementType = LMemI->getType();
  } else if (StoreInst *SMemI = dyn_cast<StoreInst>(MemI)) {
    PtrValue = SMemI->getPointerOperand();
    PointerElementType = SMemI->getValueOperand()->getType();
  } else if (IntrinsicInst *IMemI = dyn_cast<IntrinsicInst>(MemI)) {
    PointerElementType = Type::getInt8Ty(MemI->getContext());
    if (IMemI->getIntrinsicID() == Intrinsic::prefetch ||
        IMemI->getIntrinsicID() == Intrinsic::ppc_vsx_lxvp) {
      PtrValue = IMemI->getArgOperand(0);
    } else if (IMemI->getIntrinsicID() == Intrinsic::ppc_vsx_stxvp) {
      PtrValue = IMemI->getArgOperand(1);
    }
  }
  /*Get ElementType if PtrElementType is not null.*/
  if (PtrElementType)
    *PtrElementType = PointerElementType;

  return PtrValue;
}

bool PPCLoopInstrFormPrep::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

  LI = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  SE = &getAnalysis<ScalarEvolutionWrapperPass>().getSE();
  auto *DTWP = getAnalysisIfAvailable<DominatorTreeWrapperPass>();
  DT = DTWP ? &DTWP->getDomTree() : nullptr;
  PreserveLCSSA = mustPreserveAnalysisID(LCSSAID);
  ST = TM ? TM->getSubtargetImpl(F) : nullptr;
  SuccPrepCount = 0;

  bool MadeChange = false;

  for (Loop *I : *LI)
```
- **EN**: Implements helper routine(s) `getPointerOperand`, `getType`, `getValueOperand` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getPointerOperand`, `getType`, `getValueOperand`。 子目标特性裁剪会影响这里的行为。

### Lines 407-444

```cpp
    for (Loop *L : depth_first(I))
      MadeChange |= runOnLoop(L);

  return MadeChange;
}

// Finding the minimal(chain_number + reusable_offset_number) is a complicated
// algorithmic problem.
// For now, the algorithm used here is simply adjusted to handle the case for
// manually unrolling cases.
// FIXME: use a more powerful algorithm to find minimal sum of chain_number and
// reusable_offset_number for one base with multiple offsets.
bool PPCLoopInstrFormPrep::prepareBasesForCommoningChains(Bucket &CBucket) {
  // The minimal size for profitable chain commoning:
  // A1 = base + offset1
  // A2 = base + offset2 (offset2 - offset1 = X)
  // A3 = base + offset3
  // A4 = base + offset4 (offset4 - offset3 = X)
  // ======>
  // base1 = base + offset1
  // base2 = base + offset3
  // A1 = base1
  // A2 = base1 + X
  // A3 = base2
  // A4 = base2 + X
  //
  // There is benefit because of reuse of offest 'X'.

  assert(ChainCommonPrepMinThreshold >= 4 &&
         "Thredhold can not be smaller than 4!\n");
  if (CBucket.Elements.size() < ChainCommonPrepMinThreshold)
    return false;

  // We simply select the FirstOffset as the first reusable offset between each
  // chain element 1 and element 0.
  const SCEV *FirstOffset = CBucket.Elements[1].Offset;

  // Figure out how many times above FirstOffset is used in the chain.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Finding the minimal(chain_number + reusable_offset_number) is a complicated". Notable symbols in this range include `depth_first`, `runOnLoop`, `minimal`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Finding the minimal(chain_number + reusable_offset_number) is a complicated”。 该区间中较显眼的符号包括 `depth_first`, `runOnLoop`, `minimal`。

### Lines 445-482

```cpp
  // For a success commoning chain candidate, offset difference between each
  // chain element 1 and element 0 must be also FirstOffset.
  unsigned FirstOffsetReusedCount = 1;

  // Figure out how many times above FirstOffset is used in the first chain.
  // Chain number is FirstOffsetReusedCount / FirstOffsetReusedCountInFirstChain
  unsigned FirstOffsetReusedCountInFirstChain = 1;

  unsigned EleNum = CBucket.Elements.size();
  bool SawChainSeparater = false;
  for (unsigned j = 2; j != EleNum; ++j) {
    if (SE->getMinusSCEV(CBucket.Elements[j].Offset,
                         CBucket.Elements[j - 1].Offset) == FirstOffset) {
      if (!SawChainSeparater)
        FirstOffsetReusedCountInFirstChain++;
      FirstOffsetReusedCount++;
    } else
      // For now, if we meet any offset which is not FirstOffset, we assume we
      // find a new Chain.
      // This makes us miss some opportunities.
      // For example, we can common:
      //
      // {OffsetA, Offset A, OffsetB, OffsetA, OffsetA, OffsetB}
      //
      // as two chains:
      // {{OffsetA, Offset A, OffsetB}, {OffsetA, OffsetA, OffsetB}}
      // FirstOffsetReusedCount = 4; FirstOffsetReusedCountInFirstChain = 2
      //
      // But we fail to common:
      //
      // {OffsetA, OffsetB, OffsetA, OffsetA, OffsetB, OffsetA}
      // FirstOffsetReusedCount = 4; FirstOffsetReusedCountInFirstChain = 1

      SawChainSeparater = true;
  }

  // FirstOffset is not reused, skip this bucket.
  if (FirstOffsetReusedCount == 1)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For a success commoning chain candidate, offset difference between each". Notable symbols in this range include `size`, `getMinusSCEV`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For a success commoning chain candidate, offset difference between each”。 该区间中较显眼的符号包括 `size`, `getMinusSCEV`。

### Lines 483-520

```cpp
    return false;

  unsigned ChainNum =
      FirstOffsetReusedCount / FirstOffsetReusedCountInFirstChain;

  // All elements are increased by FirstOffset.
  // The number of chains should be sqrt(EleNum).
  if (!SawChainSeparater)
    ChainNum = (unsigned)sqrt((double)EleNum);

  CBucket.ChainSize = (unsigned)(EleNum / ChainNum);

  // If this is not a perfect chain(eg: not all elements can be put inside
  // commoning chains.), skip now.
  if (CBucket.ChainSize * ChainNum != EleNum)
    return false;

  if (SawChainSeparater) {
    // Check that the offset seqs are the same for all chains.
    for (unsigned i = 1; i < CBucket.ChainSize; i++)
      for (unsigned j = 1; j < ChainNum; j++)
        if (CBucket.Elements[i].Offset !=
            SE->getMinusSCEV(CBucket.Elements[i + j * CBucket.ChainSize].Offset,
                             CBucket.Elements[j * CBucket.ChainSize].Offset))
          return false;
  }

  for (unsigned i = 0; i < ChainNum; i++)
    CBucket.ChainBases.push_back(CBucket.Elements[i * CBucket.ChainSize]);

  LLVM_DEBUG(dbgs() << "Bucket has " << ChainNum << " chains.\n");

  return true;
}

bool PPCLoopInstrFormPrep::chainCommoning(Loop *L,
                                          SmallVector<Bucket, 16> &Buckets) {
  bool MadeChange = false;
```
- **EN**: Implements helper routine(s) `sqrt`, `chain`, `getMinusSCEV` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `sqrt`, `chain`, `getMinusSCEV`。

### Lines 521-558

```cpp

  if (Buckets.empty())
    return MadeChange;

  SmallPtrSet<BasicBlock *, 16> BBChanged;

  for (auto &Bucket : Buckets) {
    if (prepareBasesForCommoningChains(Bucket))
      MadeChange |= rewriteLoadStoresForCommoningChains(L, Bucket, BBChanged);
  }

  if (MadeChange)
    for (auto *BB : BBChanged)
      DeleteDeadPHIs(BB);
  return MadeChange;
}

bool PPCLoopInstrFormPrep::rewriteLoadStoresForCommoningChains(
    Loop *L, Bucket &Bucket, SmallPtrSet<BasicBlock *, 16> &BBChanged) {
  bool MadeChange = false;

  assert(Bucket.Elements.size() ==
             Bucket.ChainBases.size() * Bucket.ChainSize &&
         "invalid bucket for chain commoning!\n");
  SmallPtrSet<Value *, 16> DeletedPtrs;

  BasicBlock *LoopPredecessor = L->getLoopPredecessor();

  SCEVExpander SCEVE(*SE, "loopprepare-chaincommon");

  for (unsigned ChainIdx = 0; ChainIdx < Bucket.ChainBases.size(); ++ChainIdx) {
    unsigned BaseElemIdx = Bucket.ChainSize * ChainIdx;
    const SCEV *BaseSCEV =
        ChainIdx ? SE->getAddExpr(Bucket.BaseSCEV,
                                  Bucket.Elements[BaseElemIdx].Offset)
                 : Bucket.BaseSCEV;
    const SCEVAddRecExpr *BasePtrSCEV = cast<SCEVAddRecExpr>(BaseSCEV);
```
- **EN**: Implements helper routine(s) `empty`, `prepareBasesForCommoningChains`, `rewriteLoadStoresForCommoningChains` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `empty`, `prepareBasesForCommoningChains`, `rewriteLoadStoresForCommoningChains`。

### Lines 559-596

```cpp
    // Make sure the base is able to expand.
    if (!SCEVE.isSafeToExpand(BasePtrSCEV->getStart()))
      return MadeChange;

    assert(BasePtrSCEV->isAffine() &&
           "Invalid SCEV type for the base ptr for a candidate chain!\n");

    std::pair<Instruction *, Instruction *> Base = rewriteForBase(
        L, BasePtrSCEV, Bucket.Elements[BaseElemIdx].Instr,
        false /* CanPreInc */, ChainCommoning, SCEVE, DeletedPtrs);

    if (!Base.first || !Base.second)
      return MadeChange;

    // Keep track of the replacement pointer values we've inserted so that we
    // don't generate more pointer values than necessary.
    SmallPtrSet<Value *, 16> NewPtrs;
    NewPtrs.insert(Base.first);

    for (unsigned Idx = BaseElemIdx + 1; Idx < BaseElemIdx + Bucket.ChainSize;
         ++Idx) {
      BucketElement &I = Bucket.Elements[Idx];
      Value *Ptr = getPointerOperandAndType(I.Instr);
      assert(Ptr && "No pointer operand");
      if (NewPtrs.count(Ptr))
        continue;

      const SCEV *OffsetSCEV =
          BaseElemIdx ? SE->getMinusSCEV(Bucket.Elements[Idx].Offset,
                                         Bucket.Elements[BaseElemIdx].Offset)
                      : Bucket.Elements[Idx].Offset;

      // Make sure offset is able to expand. Only need to check one time as the
      // offsets are reused between different chains.
      if (!BaseElemIdx)
        if (!SCEVE.isSafeToExpand(OffsetSCEV))
          return false;
```
- **EN**: Implements helper routine(s) `isSafeToExpand`, `getStart`, `isAffine` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isSafeToExpand`, `getStart`, `isAffine`。

### Lines 597-634

```cpp
      Value *OffsetValue = SCEVE.expandCodeFor(
          OffsetSCEV, OffsetSCEV->getType(), LoopPredecessor->getTerminator());

      Instruction *NewPtr = rewriteForBucketElement(Base, Bucket.Elements[Idx],
                                                    OffsetValue, DeletedPtrs);

      assert(NewPtr && "Wrong rewrite!\n");
      NewPtrs.insert(NewPtr);
    }

    ++ChainCommoningRewritten;
  }

  // Clear the rewriter cache, because values that are in the rewriter's cache
  // can be deleted below, causing the AssertingVH in the cache to trigger.
  SCEVE.clear();

  for (auto *Ptr : DeletedPtrs) {
    if (Instruction *IDel = dyn_cast<Instruction>(Ptr))
      BBChanged.insert(IDel->getParent());
    RecursivelyDeleteTriviallyDeadInstructions(Ptr);
  }

  MadeChange = true;
  return MadeChange;
}

// Rewrite the new base according to BasePtrSCEV.
// bb.loop.preheader:
//   %newstart = ...
// bb.loop.body:
//   %phinode = phi [ %newstart, %bb.loop.preheader ], [ %add, %bb.loop.body ]
//   ...
//   %add = getelementptr %phinode, %inc
//
// First returned instruciton is %phinode (or a type cast to %phinode), caller
// needs this value to rewrite other load/stores in the same chain.
// Second returned instruction is %add, caller needs this value to rewrite other
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Clear the rewriter cache, because values that are in the rewriter's cache". Notable symbols in this range include `expandCodeFor`, `getType`, `getTerminator`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Clear the rewriter cache, because values that are in the rewriter's cache”。 该区间中较显眼的符号包括 `expandCodeFor`, `getType`, `getTerminator`。

### Lines 635-672

```cpp
// load/stores in the same chain.
std::pair<Instruction *, Instruction *>
PPCLoopInstrFormPrep::rewriteForBase(Loop *L, const SCEVAddRecExpr *BasePtrSCEV,
                                     Instruction *BaseMemI, bool CanPreInc,
                                     PrepForm Form, SCEVExpander &SCEVE,
                                     SmallPtrSet<Value *, 16> &DeletedPtrs) {

  LLVM_DEBUG(dbgs() << "PIP: Transforming: " << *BasePtrSCEV << "\n");

  assert(BasePtrSCEV->getLoop() == L && "AddRec for the wrong loop?");

  Value *BasePtr = getPointerOperandAndType(BaseMemI);
  assert(BasePtr && "No pointer operand");

  Type *I8Ty = Type::getInt8Ty(BaseMemI->getParent()->getContext());
  Type *I8PtrTy =
      PointerType::get(BaseMemI->getParent()->getContext(),
                       BasePtr->getType()->getPointerAddressSpace());

  bool IsConstantInc = false;
  const SCEV *BasePtrIncSCEV = BasePtrSCEV->getStepRecurrence(*SE);
  Value *IncNode = getNodeForInc(L, BaseMemI, BasePtrIncSCEV);

  const SCEVConstant *BasePtrIncConstantSCEV =
      dyn_cast<SCEVConstant>(BasePtrIncSCEV);
  if (BasePtrIncConstantSCEV)
    IsConstantInc = true;

  // No valid representation for the increment.
  if (!IncNode) {
    LLVM_DEBUG(dbgs() << "Loop Increasement can not be represented!\n");
    return std::make_pair(nullptr, nullptr);
  }

  if (Form == UpdateForm && !IsConstantInc && !EnableUpdateFormForNonConstInc) {
    LLVM_DEBUG(
        dbgs()
        << "Update form prepare for non-const increment is not enabled!\n");
```
- **EN**: Implements helper routine(s) `rewriteForBase`, `dbgs`, `getLoop` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `rewriteForBase`, `dbgs`, `getLoop`。

### Lines 673-710

```cpp
    return std::make_pair(nullptr, nullptr);
  }

  const SCEV *BasePtrStartSCEV = nullptr;
  if (CanPreInc) {
    assert(SE->isLoopInvariant(BasePtrIncSCEV, L) &&
           "Increment is not loop invariant!\n");
    BasePtrStartSCEV = SE->getMinusSCEV(BasePtrSCEV->getStart(),
                                        IsConstantInc ? BasePtrIncConstantSCEV
                                                      : BasePtrIncSCEV);
  } else
    BasePtrStartSCEV = BasePtrSCEV->getStart();

  if (alreadyPrepared(L, BaseMemI, BasePtrStartSCEV, BasePtrIncSCEV, Form)) {
    LLVM_DEBUG(dbgs() << "Instruction form is already prepared!\n");
    return std::make_pair(nullptr, nullptr);
  }

  LLVM_DEBUG(dbgs() << "PIP: New start is: " << *BasePtrStartSCEV << "\n");

  BasicBlock *Header = L->getHeader();
  unsigned HeaderLoopPredCount = pred_size(Header);
  BasicBlock *LoopPredecessor = L->getLoopPredecessor();

  PHINode *NewPHI = PHINode::Create(I8PtrTy, HeaderLoopPredCount,
                                    getInstrName(BaseMemI, PHINodeNameSuffix));
  NewPHI->insertBefore(Header->getFirstNonPHIIt());

  Value *BasePtrStart = SCEVE.expandCodeFor(BasePtrStartSCEV, I8PtrTy,
                                            LoopPredecessor->getTerminator());

  // Note that LoopPredecessor might occur in the predecessor list multiple
  // times, and we need to add it the right number of times.
  for (auto *PI : predecessors(Header)) {
    if (PI != LoopPredecessor)
      continue;

    NewPHI->addIncoming(BasePtrStart, LoopPredecessor);
```
- **EN**: Implements helper routine(s) `make_pair`, `isLoopInvariant`, `getMinusSCEV` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `make_pair`, `isLoopInvariant`, `getMinusSCEV`。

### Lines 711-748

```cpp
  }

  Instruction *PtrInc = nullptr;
  Instruction *NewBasePtr = nullptr;
  if (CanPreInc) {
    BasicBlock::iterator InsPoint = Header->getFirstInsertionPt();
    PtrInc = GetElementPtrInst::Create(
        I8Ty, NewPHI, IncNode, getInstrName(BaseMemI, GEPNodeIncNameSuffix),
        InsPoint);
    cast<GetElementPtrInst>(PtrInc)->setIsInBounds(IsPtrInBounds(BasePtr));
    for (auto *PI : predecessors(Header)) {
      if (PI == LoopPredecessor)
        continue;

      NewPHI->addIncoming(PtrInc, PI);
    }
    if (PtrInc->getType() != BasePtr->getType())
      NewBasePtr =
          new BitCastInst(PtrInc, BasePtr->getType(),
                          getInstrName(PtrInc, CastNodeNameSuffix), InsPoint);
    else
      NewBasePtr = PtrInc;
  } else {
    // Note that LoopPredecessor might occur in the predecessor list multiple
    // times, and we need to make sure no more incoming value for them in PHI.
    for (auto *PI : predecessors(Header)) {
      if (PI == LoopPredecessor)
        continue;

      // For the latch predecessor, we need to insert a GEP just before the
      // terminator to increase the address.
      BasicBlock *BB = PI;
      BasicBlock::iterator InsPoint = BB->getTerminator()->getIterator();
      PtrInc = GetElementPtrInst::Create(
          I8Ty, NewPHI, IncNode, getInstrName(BaseMemI, GEPNodeIncNameSuffix),
          InsPoint);
      cast<GetElementPtrInst>(PtrInc)->setIsInBounds(IsPtrInBounds(BasePtr));
```
- **EN**: Implements helper routine(s) `getFirstInsertionPt`, `Create`, `getInstrName` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getFirstInsertionPt`, `Create`, `getInstrName`。

### Lines 749-786

```cpp
      NewPHI->addIncoming(PtrInc, PI);
    }
    PtrInc = NewPHI;
    if (NewPHI->getType() != BasePtr->getType())
      NewBasePtr = new BitCastInst(NewPHI, BasePtr->getType(),
                                   getInstrName(NewPHI, CastNodeNameSuffix),
                                   Header->getFirstInsertionPt());
    else
      NewBasePtr = NewPHI;
  }

  BasePtr->replaceAllUsesWith(NewBasePtr);

  DeletedPtrs.insert(BasePtr);

  return std::make_pair(NewBasePtr, PtrInc);
}

Instruction *PPCLoopInstrFormPrep::rewriteForBucketElement(
    std::pair<Instruction *, Instruction *> Base, const BucketElement &Element,
    Value *OffToBase, SmallPtrSet<Value *, 16> &DeletedPtrs) {
  Instruction *NewBasePtr = Base.first;
  Instruction *PtrInc = Base.second;
  assert((NewBasePtr && PtrInc) && "base does not exist!\n");

  Type *I8Ty = Type::getInt8Ty(PtrInc->getParent()->getContext());

  Value *Ptr = getPointerOperandAndType(Element.Instr);
  assert(Ptr && "No pointer operand");

  Instruction *RealNewPtr;
  if (!Element.Offset ||
      (isa<SCEVConstant>(Element.Offset) &&
       cast<SCEVConstant>(Element.Offset)->getValue()->isZero())) {
    RealNewPtr = NewBasePtr;
  } else {
    std::optional<BasicBlock::iterator> PtrIP = std::nullopt;
    if (Instruction *I = dyn_cast<Instruction>(Ptr))
```
- **EN**: Implements helper routine(s) `addIncoming`, `getType`, `BitCastInst` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `addIncoming`, `getType`, `BitCastInst`。

### Lines 787-824

```cpp
      PtrIP = I->getIterator();

    if (PtrIP && isa<Instruction>(NewBasePtr) &&
        cast<Instruction>(NewBasePtr)->getParent() == (*PtrIP)->getParent())
      PtrIP = std::nullopt;
    else if (PtrIP && isa<PHINode>(*PtrIP))
      PtrIP = (*PtrIP)->getParent()->getFirstInsertionPt();
    else if (!PtrIP)
      PtrIP = Element.Instr->getIterator();

    assert(OffToBase && "There should be an offset for non base element!\n");
    GetElementPtrInst *NewPtr = GetElementPtrInst::Create(
        I8Ty, PtrInc, OffToBase,
        getInstrName(Element.Instr, GEPNodeOffNameSuffix));
    if (PtrIP)
      NewPtr->insertBefore(*(*PtrIP)->getParent(), *PtrIP);
    else
      NewPtr->insertAfter(cast<Instruction>(PtrInc));
    NewPtr->setIsInBounds(IsPtrInBounds(Ptr));
    RealNewPtr = NewPtr;
  }

  Instruction *ReplNewPtr;
  if (Ptr->getType() != RealNewPtr->getType()) {
    ReplNewPtr = new BitCastInst(RealNewPtr, Ptr->getType(),
                                 getInstrName(Ptr, CastNodeNameSuffix));
    ReplNewPtr->insertAfter(RealNewPtr);
  } else
    ReplNewPtr = RealNewPtr;

  Ptr->replaceAllUsesWith(ReplNewPtr);
  DeletedPtrs.insert(Ptr);

  return ReplNewPtr;
}

void PPCLoopInstrFormPrep::addOneCandidate(
    Instruction *MemI, const SCEV *LSCEV, SmallVector<Bucket, 16> &Buckets,
```
- **EN**: Implements helper routine(s) `getIterator`, `getParent`, `getFirstInsertionPt` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getIterator`, `getParent`, `getFirstInsertionPt`。

### Lines 825-862

```cpp
    std::function<bool(const SCEV *)> isValidDiff, unsigned MaxCandidateNum) {
  assert((MemI && getPointerOperandAndType(MemI)) &&
         "Candidate should be a memory instruction.");
  assert(LSCEV && "Invalid SCEV for Ptr value.");

  bool FoundBucket = false;
  for (auto &B : Buckets) {
    if (cast<SCEVAddRecExpr>(B.BaseSCEV)->getStepRecurrence(*SE) !=
        cast<SCEVAddRecExpr>(LSCEV)->getStepRecurrence(*SE))
      continue;
    const SCEV *Diff = SE->getMinusSCEV(LSCEV, B.BaseSCEV);
    if (isValidDiff(Diff)) {
      B.Elements.push_back(BucketElement(Diff, MemI));
      FoundBucket = true;
      break;
    }
  }

  if (!FoundBucket) {
    if (Buckets.size() == MaxCandidateNum) {
      LLVM_DEBUG(dbgs() << "Can not prepare more chains, reach maximum limit "
                        << MaxCandidateNum << "\n");
      return;
    }
    Buckets.push_back(Bucket(LSCEV, MemI));
  }
}

SmallVector<Bucket, 16> PPCLoopInstrFormPrep::collectCandidates(
    Loop *L,
    std::function<bool(const Instruction *, Value *, const Type *)>
        isValidCandidate,
    std::function<bool(const SCEV *)> isValidDiff, unsigned MaxCandidateNum) {
  SmallVector<Bucket, 16> Buckets;

  for (const auto &BB : L->blocks())
    for (auto &J : *BB) {
      Value *PtrValue = nullptr;
```
- **EN**: Implements helper routine(s) `bool`, `getPointerOperandAndType`, `getStepRecurrence` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `bool`, `getPointerOperandAndType`, `getStepRecurrence`。

### Lines 863-900

```cpp
      Type *PointerElementType = nullptr;
      PtrValue = getPointerOperandAndType(&J, &PointerElementType);

      if (!PtrValue)
        continue;

      if (PtrValue->getType()->getPointerAddressSpace())
        continue;

      if (L->isLoopInvariant(PtrValue))
        continue;

      const SCEV *LSCEV = SE->getSCEVAtScope(PtrValue, L);
      const SCEVAddRecExpr *LARSCEV = dyn_cast<SCEVAddRecExpr>(LSCEV);
      if (!LARSCEV || LARSCEV->getLoop() != L)
        continue;

      // Mark that we have candidates for preparing.
      HasCandidateForPrepare = true;

      if (isValidCandidate(&J, PtrValue, PointerElementType))
        addOneCandidate(&J, LSCEV, Buckets, isValidDiff, MaxCandidateNum);
    }
  return Buckets;
}

bool PPCLoopInstrFormPrep::prepareBaseForDispFormChain(Bucket &BucketChain,
                                                       PrepForm Form) {
  // RemainderOffsetInfo details:
  // key:            value of (Offset urem DispConstraint). For DSForm, it can
  //                 be [0, 4).
  // first of pair:  the index of first BucketElement whose remainder is equal
  //                 to key. For key 0, this value must be 0.
  // second of pair: number of load/stores with the same remainder.
  DenseMap<unsigned, std::pair<unsigned, unsigned>> RemainderOffsetInfo;

  for (unsigned j = 0, je = BucketChain.Elements.size(); j != je; ++j) {
    if (!BucketChain.Elements[j].Offset)
```
- **EN**: Implements helper routine(s) `getPointerOperandAndType`, `getType`, `getPointerAddressSpace` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getPointerOperandAndType`, `getType`, `getPointerAddressSpace`。

### Lines 901-938

```cpp
      RemainderOffsetInfo[0] = std::make_pair(0, 1);
    else {
      unsigned Remainder = cast<SCEVConstant>(BucketChain.Elements[j].Offset)
                               ->getAPInt()
                               .urem(Form);
      if (!RemainderOffsetInfo.contains(Remainder))
        RemainderOffsetInfo[Remainder] = std::make_pair(j, 1);
      else
        RemainderOffsetInfo[Remainder].second++;
    }
  }
  // Currently we choose the most profitable base as the one which has the max
  // number of load/store with same remainder.
  // FIXME: adjust the base selection strategy according to load/store offset
  // distribution.
  // For example, if we have one candidate chain for DS form preparation, which
  // contains following load/stores with different remainders:
  // 1: 10 load/store whose remainder is 1;
  // 2: 9 load/store whose remainder is 2;
  // 3: 1 for remainder 3 and 0 for remainder 0;
  // Now we will choose the first load/store whose remainder is 1 as base and
  // adjust all other load/stores according to new base, so we will get 10 DS
  // form and 10 X form.
  // But we should be more clever, for this case we could use two bases, one for
  // remainder 1 and the other for remainder 2, thus we could get 19 DS form and
  // 1 X form.
  unsigned MaxCountRemainder = 0;
  for (unsigned j = 0; j < (unsigned)Form; j++)
    if (auto It = RemainderOffsetInfo.find(j);
        It != RemainderOffsetInfo.end() &&
        It->second.second > RemainderOffsetInfo[MaxCountRemainder].second)
      MaxCountRemainder = j;

  // Abort when there are too few insts with common base.
  if (RemainderOffsetInfo[MaxCountRemainder].second < DispFormPrepMinThreshold)
    return false;

  // If the first value is most profitable, no needed to adjust BucketChain
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Currently we choose the most profitable base as the one which has the max". Notable symbols in this range include `make_pair`, `getAPInt`, `urem`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Currently we choose the most profitable base as the one which has the max”。 该区间中较显眼的符号包括 `make_pair`, `getAPInt`, `urem`。

### Lines 939-976

```cpp
  // elements as they are substracted the first value when collecting.
  if (MaxCountRemainder == 0)
    return true;

  // Adjust load/store to the new chosen base.
  const SCEV *Offset =
      BucketChain.Elements[RemainderOffsetInfo[MaxCountRemainder].first].Offset;
  BucketChain.BaseSCEV = SE->getAddExpr(BucketChain.BaseSCEV, Offset);
  for (auto &E : BucketChain.Elements) {
    if (E.Offset)
      E.Offset = cast<SCEVConstant>(SE->getMinusSCEV(E.Offset, Offset));
    else
      E.Offset = cast<SCEVConstant>(SE->getNegativeSCEV(Offset));
  }

  std::swap(BucketChain.Elements[RemainderOffsetInfo[MaxCountRemainder].first],
            BucketChain.Elements[0]);
  return true;
}

// FIXME: implement a more clever base choosing policy.
// Currently we always choose an exist load/store offset. This maybe lead to
// suboptimal code sequences. For example, for one DS chain with offsets
// {-32769, 2003, 2007, 2011}, we choose -32769 as base offset, and left disp
// for load/stores are {0, 34772, 34776, 34780}. Though each offset now is a
// multipler of 4, it cannot be represented by sint16.
bool PPCLoopInstrFormPrep::prepareBaseForUpdateFormChain(Bucket &BucketChain) {
  // We have a choice now of which instruction's memory operand we use as the
  // base for the generated PHI. Always picking the first instruction in each
  // bucket does not work well, specifically because that instruction might
  // be a prefetch (and there are no pre-increment dcbt variants). Otherwise,
  // the choice is somewhat arbitrary, because the backend will happily
  // generate direct offsets from both the pre-incremented and
  // post-incremented pointer values. Thus, we'll pick the first non-prefetch
  // instruction in each bucket, and adjust the recurrence and other offsets
  // accordingly.
  for (int j = 0, je = BucketChain.Elements.size(); j != je; ++j) {
    if (auto *II = dyn_cast<IntrinsicInst>(BucketChain.Elements[j].Instr))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "elements as they are substracted the first value when collecting.". Notable symbols in this range include `getAddExpr`, `getMinusSCEV`, `getNegativeSCEV`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“elements as they are substracted the first value when collecting.”。 该区间中较显眼的符号包括 `getAddExpr`, `getMinusSCEV`, `getNegativeSCEV`。

### Lines 977-1014

```cpp
      if (II->getIntrinsicID() == Intrinsic::prefetch)
        continue;

    // If we'd otherwise pick the first element anyway, there's nothing to do.
    if (j == 0)
      break;

    // If our chosen element has no offset from the base pointer, there's
    // nothing to do.
    if (!BucketChain.Elements[j].Offset ||
        cast<SCEVConstant>(BucketChain.Elements[j].Offset)->isZero())
      break;

    const SCEV *Offset = BucketChain.Elements[j].Offset;
    BucketChain.BaseSCEV = SE->getAddExpr(BucketChain.BaseSCEV, Offset);
    for (auto &E : BucketChain.Elements) {
      if (E.Offset)
        E.Offset = cast<SCEVConstant>(SE->getMinusSCEV(E.Offset, Offset));
      else
        E.Offset = cast<SCEVConstant>(SE->getNegativeSCEV(Offset));
    }

    std::swap(BucketChain.Elements[j], BucketChain.Elements[0]);
    break;
  }
  return true;
}

bool PPCLoopInstrFormPrep::rewriteLoadStores(
    Loop *L, Bucket &BucketChain, SmallPtrSet<BasicBlock *, 16> &BBChanged,
    PrepForm Form) {
  bool MadeChange = false;

  const SCEVAddRecExpr *BasePtrSCEV =
      cast<SCEVAddRecExpr>(BucketChain.BaseSCEV);
  if (!BasePtrSCEV->isAffine())
    return MadeChange;
```
- **EN**: Implements helper routine(s) `getIntrinsicID`, `isZero`, `getAddExpr` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getIntrinsicID`, `isZero`, `getAddExpr`。

### Lines 1015-1052

```cpp
  SCEVExpander SCEVE(*SE, "loopprepare-formrewrite");
  if (!SCEVE.isSafeToExpand(BasePtrSCEV->getStart()))
    return MadeChange;

  SmallPtrSet<Value *, 16> DeletedPtrs;

  // For some DS form load/store instructions, it can also be an update form,
  // if the stride is constant and is a multipler of 4. Use update form if
  // prefer it.
  bool CanPreInc = (Form == UpdateForm ||
                    ((Form == DSForm) &&
                     isa<SCEVConstant>(BasePtrSCEV->getStepRecurrence(*SE)) &&
                     !cast<SCEVConstant>(BasePtrSCEV->getStepRecurrence(*SE))
                          ->getAPInt()
                          .urem(4) &&
                     PreferUpdateForm));

  std::pair<Instruction *, Instruction *> Base =
      rewriteForBase(L, BasePtrSCEV, BucketChain.Elements.begin()->Instr,
                     CanPreInc, Form, SCEVE, DeletedPtrs);

  if (!Base.first || !Base.second)
    return MadeChange;

  // Keep track of the replacement pointer values we've inserted so that we
  // don't generate more pointer values than necessary.
  SmallPtrSet<Value *, 16> NewPtrs;
  NewPtrs.insert(Base.first);

  for (const BucketElement &BE : llvm::drop_begin(BucketChain.Elements)) {
    Value *Ptr = getPointerOperandAndType(BE.Instr);
    assert(Ptr && "No pointer operand");
    if (NewPtrs.count(Ptr))
      continue;

    Instruction *NewPtr = rewriteForBucketElement(
        Base, BE,
        BE.Offset ? cast<SCEVConstant>(BE.Offset)->getValue() : nullptr,
```
- **EN**: Implements helper routine(s) `SCEVE`, `isSafeToExpand`, `getStart` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `SCEVE`, `isSafeToExpand`, `getStart`。

### Lines 1053-1090

```cpp
        DeletedPtrs);
    assert(NewPtr && "wrong rewrite!\n");
    NewPtrs.insert(NewPtr);
  }

  // Clear the rewriter cache, because values that are in the rewriter's cache
  // can be deleted below, causing the AssertingVH in the cache to trigger.
  SCEVE.clear();

  for (auto *Ptr : DeletedPtrs) {
    if (Instruction *IDel = dyn_cast<Instruction>(Ptr))
      BBChanged.insert(IDel->getParent());
    RecursivelyDeleteTriviallyDeadInstructions(Ptr);
  }

  MadeChange = true;

  SuccPrepCount++;

  if (Form == DSForm && !CanPreInc)
    DSFormChainRewritten++;
  else if (Form == DQForm)
    DQFormChainRewritten++;
  else if (Form == UpdateForm || (Form == DSForm && CanPreInc))
    UpdFormChainRewritten++;

  return MadeChange;
}

bool PPCLoopInstrFormPrep::updateFormPrep(Loop *L,
                                       SmallVector<Bucket, 16> &Buckets) {
  bool MadeChange = false;
  if (Buckets.empty())
    return MadeChange;
  SmallPtrSet<BasicBlock *, 16> BBChanged;
  for (auto &Bucket : Buckets)
    // The base address of each bucket is transformed into a phi and the others
    // are rewritten based on new base.
```
- **EN**: Implements helper routine(s) `insert`, `clear`, `getParent` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `insert`, `clear`, `getParent`。

### Lines 1091-1128

```cpp
    if (prepareBaseForUpdateFormChain(Bucket))
      MadeChange |= rewriteLoadStores(L, Bucket, BBChanged, UpdateForm);

  if (MadeChange)
    for (auto *BB : BBChanged)
      DeleteDeadPHIs(BB);
  return MadeChange;
}

bool PPCLoopInstrFormPrep::dispFormPrep(Loop *L,
                                        SmallVector<Bucket, 16> &Buckets,
                                        PrepForm Form) {
  bool MadeChange = false;

  if (Buckets.empty())
    return MadeChange;

  SmallPtrSet<BasicBlock *, 16> BBChanged;
  for (auto &Bucket : Buckets) {
    if (Bucket.Elements.size() < DispFormPrepMinThreshold)
      continue;
    if (prepareBaseForDispFormChain(Bucket, Form))
      MadeChange |= rewriteLoadStores(L, Bucket, BBChanged, Form);
  }

  if (MadeChange)
    for (auto *BB : BBChanged)
      DeleteDeadPHIs(BB);
  return MadeChange;
}

// Find the loop invariant increment node for SCEV BasePtrIncSCEV.
// bb.loop.preheader:
//   %start = ...
// bb.loop.body:
//   %phinode = phi [ %start, %bb.loop.preheader ], [ %add, %bb.loop.body ]
//   ...
//   %add = add %phinode, %inc  ; %inc is what we want to get.
```
- **EN**: Implements helper routine(s) `prepareBaseForUpdateFormChain`, `rewriteLoadStores`, `DeleteDeadPHIs` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `prepareBaseForUpdateFormChain`, `rewriteLoadStores`, `DeleteDeadPHIs`。

### Lines 1129-1166

```cpp
//
Value *PPCLoopInstrFormPrep::getNodeForInc(Loop *L, Instruction *MemI,
                                           const SCEV *BasePtrIncSCEV) {
  // If the increment is a constant, no definition is needed.
  // Return the value directly.
  if (isa<SCEVConstant>(BasePtrIncSCEV))
    return cast<SCEVConstant>(BasePtrIncSCEV)->getValue();

  if (!SE->isLoopInvariant(BasePtrIncSCEV, L))
    return nullptr;

  BasicBlock *BB = MemI->getParent();
  if (!BB)
    return nullptr;

  BasicBlock *LatchBB = L->getLoopLatch();

  if (!LatchBB)
    return nullptr;

  // Run through the PHIs and check their operands to find valid representation
  // for the increment SCEV.
  iterator_range<BasicBlock::phi_iterator> PHIIter = BB->phis();
  for (auto &CurrentPHI : PHIIter) {
    PHINode *CurrentPHINode = dyn_cast<PHINode>(&CurrentPHI);
    if (!CurrentPHINode)
      continue;

    if (!SE->isSCEVable(CurrentPHINode->getType()))
      continue;

    const SCEV *PHISCEV = SE->getSCEVAtScope(CurrentPHINode, L);

    const SCEVAddRecExpr *PHIBasePtrSCEV = dyn_cast<SCEVAddRecExpr>(PHISCEV);
    if (!PHIBasePtrSCEV)
      continue;

    const SCEV *PHIBasePtrIncSCEV = PHIBasePtrSCEV->getStepRecurrence(*SE);
```
- **EN**: Implements helper routine(s) `getNodeForInc`, `getValue`, `isLoopInvariant` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getNodeForInc`, `getValue`, `isLoopInvariant`。

### Lines 1167-1204

```cpp

    if (!PHIBasePtrIncSCEV || (PHIBasePtrIncSCEV != BasePtrIncSCEV))
      continue;

    // Get the incoming value from the loop latch and check if the value has
    // the add form with the required increment.
    if (CurrentPHINode->getBasicBlockIndex(LatchBB) < 0)
      continue;
    if (Instruction *I = dyn_cast<Instruction>(
            CurrentPHINode->getIncomingValueForBlock(LatchBB))) {
      Value *StrippedBaseI = I;
      while (BitCastInst *BC = dyn_cast<BitCastInst>(StrippedBaseI))
        StrippedBaseI = BC->getOperand(0);

      Instruction *StrippedI = dyn_cast<Instruction>(StrippedBaseI);
      if (!StrippedI)
        continue;

      // LSR pass may add a getelementptr instruction to do the loop increment,
      // also search in that getelementptr instruction.
      if (StrippedI->getOpcode() == Instruction::Add ||
          (StrippedI->getOpcode() == Instruction::GetElementPtr &&
           StrippedI->getNumOperands() == 2)) {
        if (SE->getSCEVAtScope(StrippedI->getOperand(0), L) == BasePtrIncSCEV)
          return StrippedI->getOperand(0);
        if (SE->getSCEVAtScope(StrippedI->getOperand(1), L) == BasePtrIncSCEV)
          return StrippedI->getOperand(1);
      }
    }
  }
  return nullptr;
}

// In order to prepare for the preferred instruction form, a PHI is added.
// This function will check to see if that PHI already exists and will return
// true if it found an existing PHI with the matched start and increment as the
// one we wanted to create.
bool PPCLoopInstrFormPrep::alreadyPrepared(Loop *L, Instruction *MemI,
```
- **EN**: Implements helper routine(s) `getBasicBlockIndex`, `getIncomingValueForBlock`, `getOperand` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getBasicBlockIndex`, `getIncomingValueForBlock`, `getOperand`。

### Lines 1205-1242

```cpp
                                           const SCEV *BasePtrStartSCEV,
                                           const SCEV *BasePtrIncSCEV,
                                           PrepForm Form) {
  BasicBlock *BB = MemI->getParent();
  if (!BB)
    return false;

  BasicBlock *PredBB = L->getLoopPredecessor();
  BasicBlock *LatchBB = L->getLoopLatch();

  if (!PredBB || !LatchBB)
    return false;

  // Run through the PHIs and see if we have some that looks like a preparation
  iterator_range<BasicBlock::phi_iterator> PHIIter = BB->phis();
  for (auto & CurrentPHI : PHIIter) {
    PHINode *CurrentPHINode = dyn_cast<PHINode>(&CurrentPHI);
    if (!CurrentPHINode)
      continue;

    if (!SE->isSCEVable(CurrentPHINode->getType()))
      continue;

    const SCEV *PHISCEV = SE->getSCEVAtScope(CurrentPHINode, L);

    const SCEVAddRecExpr *PHIBasePtrSCEV = dyn_cast<SCEVAddRecExpr>(PHISCEV);
    if (!PHIBasePtrSCEV)
      continue;

    const SCEVConstant *PHIBasePtrIncSCEV =
      dyn_cast<SCEVConstant>(PHIBasePtrSCEV->getStepRecurrence(*SE));
    if (!PHIBasePtrIncSCEV)
      continue;

    if (CurrentPHINode->getNumIncomingValues() == 2) {
      if ((CurrentPHINode->getIncomingBlock(0) == LatchBB &&
           CurrentPHINode->getIncomingBlock(1) == PredBB) ||
          (CurrentPHINode->getIncomingBlock(1) == LatchBB &&
```
- **EN**: Implements helper routine(s) `getParent`, `getLoopPredecessor`, `getLoopLatch` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getParent`, `getLoopPredecessor`, `getLoopLatch`。

### Lines 1243-1280

```cpp
           CurrentPHINode->getIncomingBlock(0) == PredBB)) {
        if (PHIBasePtrIncSCEV == BasePtrIncSCEV) {
          // The existing PHI (CurrentPHINode) has the same start and increment
          // as the PHI that we wanted to create.
          if ((Form == UpdateForm || Form == ChainCommoning ) &&
              PHIBasePtrSCEV->getStart() == BasePtrStartSCEV) {
            ++PHINodeAlreadyExistsUpdate;
            return true;
          }
          if (Form == DSForm || Form == DQForm) {
            const SCEVConstant *Diff = dyn_cast<SCEVConstant>(
                SE->getMinusSCEV(PHIBasePtrSCEV->getStart(), BasePtrStartSCEV));
            if (Diff && !Diff->getAPInt().urem(Form)) {
              if (Form == DSForm)
                ++PHINodeAlreadyExistsDS;
              else
                ++PHINodeAlreadyExistsDQ;
              return true;
            }
          }
        }
      }
    }
  }
  return false;
}

bool PPCLoopInstrFormPrep::runOnLoop(Loop *L) {
  bool MadeChange = false;

  // Only prep. the inner-most loop
  if (!L->isInnermost())
    return MadeChange;

  // Return if already done enough preparation.
  if (SuccPrepCount >= MaxVarsPrep)
    return MadeChange;
```
- **EN**: Implements helper routine(s) `getIncomingBlock`, `PHI`, `getStart` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getIncomingBlock`, `PHI`, `getStart`。

### Lines 1281-1318

```cpp
  LLVM_DEBUG(dbgs() << "PIP: Examining: " << *L << "\n");

  BasicBlock *LoopPredecessor = L->getLoopPredecessor();
  // If there is no loop predecessor, or the loop predecessor's terminator
  // returns a value (which might contribute to determining the loop's
  // iteration space), insert a new preheader for the loop.
  if (!LoopPredecessor ||
      !LoopPredecessor->getTerminator()->getType()->isVoidTy()) {
    LoopPredecessor = InsertPreheaderForLoop(L, DT, LI, nullptr, PreserveLCSSA);
    if (LoopPredecessor)
      MadeChange = true;
  }
  if (!LoopPredecessor) {
    LLVM_DEBUG(dbgs() << "PIP fails since no predecessor for current loop.\n");
    return MadeChange;
  }
  // Check if a load/store has update form. This lambda is used by function
  // collectCandidates which can collect candidates for types defined by lambda.
  auto isUpdateFormCandidate = [&](const Instruction *I, Value *PtrValue,
                                   const Type *PointerElementType) {
    assert((PtrValue && I) && "Invalid parameter!");
    // There are no update forms for Altivec vector load/stores.
    if (ST && ST->hasAltivec() && PointerElementType->isVectorTy())
      return false;
    // There are no update forms for P10 lxvp/stxvp intrinsic.
    auto *II = dyn_cast<IntrinsicInst>(I);
    if (II && ((II->getIntrinsicID() == Intrinsic::ppc_vsx_lxvp) ||
               II->getIntrinsicID() == Intrinsic::ppc_vsx_stxvp))
      return false;
    // See getPreIndexedAddressParts, the displacement for LDU/STDU has to
    // be 4's multiple (DS-form). For i64 loads/stores when the displacement
    // fits in a 16-bit signed field but isn't a multiple of 4, it will be
    // useless and possible to break some original well-form addressing mode
    // to make this pre-inc prep for it.
    if (PointerElementType->isIntegerTy(64)) {
      const SCEV *LSCEV = SE->getSCEVAtScope(PtrValue, L);
      const SCEVAddRecExpr *LARSCEV = dyn_cast<SCEVAddRecExpr>(LSCEV);
      if (!LARSCEV || LARSCEV->getLoop() != L)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "If there is no loop predecessor, or the loop predecessor's terminator". Notable symbols in this range include `dbgs`, `getLoopPredecessor`, `value`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“If there is no loop predecessor, or the loop predecessor's terminator”。 该区间中较显眼的符号包括 `dbgs`, `getLoopPredecessor`, `value`。

### Lines 1319-1356

```cpp
        return false;
      if (const SCEVConstant *StepConst =
              dyn_cast<SCEVConstant>(LARSCEV->getStepRecurrence(*SE))) {
        const APInt &ConstInt = StepConst->getValue()->getValue();
        if (ConstInt.isSignedIntN(16) && ConstInt.srem(4) != 0)
          return false;
      }
    }
    return true;
  };

  // Check if a load/store has DS form.
  auto isDSFormCandidate = [](const Instruction *I, Value *PtrValue,
                              const Type *PointerElementType) {
    assert((PtrValue && I) && "Invalid parameter!");
    if (isa<IntrinsicInst>(I))
      return false;
    return (PointerElementType->isIntegerTy(64)) ||
           (PointerElementType->isFloatTy()) ||
           (PointerElementType->isDoubleTy()) ||
           (PointerElementType->isIntegerTy(32) &&
            llvm::any_of(I->users(),
                         [](const User *U) { return isa<SExtInst>(U); }));
  };

  // Check if a load/store has DQ form.
  auto isDQFormCandidate = [&](const Instruction *I, Value *PtrValue,
                               const Type *PointerElementType) {
    assert((PtrValue && I) && "Invalid parameter!");
    // Check if it is a P10 lxvp/stxvp intrinsic.
    auto *II = dyn_cast<IntrinsicInst>(I);
    if (II)
      return II->getIntrinsicID() == Intrinsic::ppc_vsx_lxvp ||
             II->getIntrinsicID() == Intrinsic::ppc_vsx_stxvp;
    // Check if it is a P9 vector load/store.
    return ST && ST->hasP9Vector() && (PointerElementType->isVectorTy());
  };
```
- **EN**: Implements helper routine(s) `getStepRecurrence`, `getValue`, `isSignedIntN` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getStepRecurrence`, `getValue`, `isSignedIntN`。

### Lines 1357-1394

```cpp
  // Check if a load/store is candidate for chain commoning.
  // If the SCEV is only with one ptr operand in its start, we can use that
  // start as a chain separator. Mark this load/store as a candidate.
  auto isChainCommoningCandidate = [&](const Instruction *I, Value *PtrValue,
                                       const Type *PointerElementType) {
    const SCEVAddRecExpr *ARSCEV =
        cast<SCEVAddRecExpr>(SE->getSCEVAtScope(PtrValue, L));
    if (!ARSCEV)
      return false;

    if (!ARSCEV->isAffine())
      return false;

    const SCEV *Start = ARSCEV->getStart();

    // A single pointer. We can treat it as offset 0.
    if (isa<SCEVUnknown>(Start) && Start->getType()->isPointerTy())
      return true;

    const SCEVAddExpr *ASCEV = dyn_cast<SCEVAddExpr>(Start);

    // We need a SCEVAddExpr to include both base and offset.
    if (!ASCEV)
      return false;

    // Make sure there is only one pointer operand(base) and all other operands
    // are integer type.
    bool SawPointer = false;
    for (const SCEV *Op : ASCEV->operands()) {
      if (Op->getType()->isPointerTy()) {
        if (SawPointer)
          return false;
        SawPointer = true;
      } else if (!Op->getType()->isIntegerTy())
        return false;
    }

    return SawPointer;
```
- **EN**: Implements helper routine(s) `getSCEVAtScope`, `isAffine`, `getStart` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getSCEVAtScope`, `isAffine`, `getStart`。

### Lines 1395-1432

```cpp
  };

  // Check if the diff is a constant type. This is used for update/DS/DQ form
  // preparation.
  auto isValidConstantDiff = [](const SCEV *Diff) {
    return dyn_cast<SCEVConstant>(Diff) != nullptr;
  };

  // Make sure the diff between the base and new candidate is required type.
  // This is used for chain commoning preparation.
  auto isValidChainCommoningDiff = [](const SCEV *Diff) {
    assert(Diff && "Invalid Diff!\n");

    // Don't mess up previous dform prepare.
    if (isa<SCEVConstant>(Diff))
      return false;

    // A single integer type offset.
    if (isa<SCEVUnknown>(Diff) && Diff->getType()->isIntegerTy())
      return true;

    const SCEVNAryExpr *ADiff = dyn_cast<SCEVNAryExpr>(Diff);
    if (!ADiff)
      return false;

    for (const SCEV *Op : ADiff->operands())
      if (!Op->getType()->isIntegerTy())
        return false;

    return true;
  };

  HasCandidateForPrepare = false;

  LLVM_DEBUG(dbgs() << "Start to prepare for update form.\n");
  // Collect buckets of comparable addresses used by loads and stores for update
  // form.
  SmallVector<Bucket, 16> UpdateFormBuckets = collectCandidates(
```
- **EN**: Implements helper routine(s) `getType`, `isIntegerTy`, `operands` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getType`, `isIntegerTy`, `operands`。

### Lines 1433-1470

```cpp
      L, isUpdateFormCandidate, isValidConstantDiff, MaxVarsUpdateForm);

  // Prepare for update form.
  if (!UpdateFormBuckets.empty())
    MadeChange |= updateFormPrep(L, UpdateFormBuckets);
  else if (!HasCandidateForPrepare) {
    LLVM_DEBUG(
        dbgs()
        << "No prepare candidates found, stop praparation for current loop!\n");
    // If no candidate for preparing, return early.
    return MadeChange;
  }

  LLVM_DEBUG(dbgs() << "Start to prepare for DS form.\n");
  // Collect buckets of comparable addresses used by loads and stores for DS
  // form.
  SmallVector<Bucket, 16> DSFormBuckets = collectCandidates(
      L, isDSFormCandidate, isValidConstantDiff, MaxVarsDSForm);

  // Prepare for DS form.
  if (!DSFormBuckets.empty())
    MadeChange |= dispFormPrep(L, DSFormBuckets, DSForm);

  LLVM_DEBUG(dbgs() << "Start to prepare for DQ form.\n");
  // Collect buckets of comparable addresses used by loads and stores for DQ
  // form.
  SmallVector<Bucket, 16> DQFormBuckets = collectCandidates(
      L, isDQFormCandidate, isValidConstantDiff, MaxVarsDQForm);

  // Prepare for DQ form.
  if (!DQFormBuckets.empty())
    MadeChange |= dispFormPrep(L, DQFormBuckets, DQForm);

  // Collect buckets of comparable addresses used by loads and stores for chain
  // commoning. With chain commoning, we reuse offsets between the chains, so
  // the register pressure will be reduced.
  if (!EnableChainCommoning) {
    LLVM_DEBUG(dbgs() << "Chain commoning is not enabled.\n");
```
- **EN**: Implements helper routine(s) `empty`, `updateFormPrep`, `dbgs` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `empty`, `updateFormPrep`, `dbgs`。

### Lines 1471-1484

```cpp
    return MadeChange;
  }

  LLVM_DEBUG(dbgs() << "Start to prepare for chain commoning.\n");
  SmallVector<Bucket, 16> Buckets =
      collectCandidates(L, isChainCommoningCandidate, isValidChainCommoningDiff,
                        MaxVarsChainCommon);

  // Prepare for chain commoning.
  if (!Buckets.empty())
    MadeChange |= chainCommoning(L, Buckets);

  return MadeChange;
}
```
- **EN**: Declares function entry points including `dbgs`, `collectCandidates`, `empty` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `dbgs`, `collectCandidates`, `empty`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCSubtarget.h`
- `PPCTargetMachine.h`
- `llvm/ADT/DepthFirstIterator.h`
- `llvm/ADT/SmallPtrSet.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/Statistic.h`
- `llvm/Analysis/LoopInfo.h`
- `llvm/Analysis/ScalarEvolution.h`
- `llvm/Analysis/ScalarEvolutionExpressions.h`
- `llvm/IR/BasicBlock.h`
- `llvm/IR/CFG.h`
- `llvm/IR/Dominators.h`
- `llvm/IR/Instruction.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/IntrinsicInst.h`
- `llvm/IR/IntrinsicsPowerPC.h`
- `llvm/IR/Type.h`
- `llvm/IR/Value.h`
- `llvm/InitializePasses.h`
- `llvm/Pass.h`
- `llvm/Support/Casting.h`
- `llvm/Support/CommandLine.h`
- `llvm/Support/Debug.h`
- `llvm/Transforms/Scalar.h`
- `llvm/Transforms/Utils.h`
- `llvm/Transforms/Utils/BasicBlockUtils.h`
- `llvm/Transforms/Utils/Local.h`
- `llvm/Transforms/Utils/LoopUtils.h`
- `llvm/Transforms/Utils/ScalarEvolutionExpander.h`
- `cassert`
- `cmath`
- `utility`

### Important Collaborators / 重要协作组件

- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
