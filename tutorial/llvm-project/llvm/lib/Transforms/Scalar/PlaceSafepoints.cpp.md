# PlaceSafepoints.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/PlaceSafepoints.cpp` | `llvm/lib/Transforms/Scalar/PlaceSafepoints.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements place GC Safepoints within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 PlaceSafepoints 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-53

```cpp
//===- PlaceSafepoints.cpp - Place GC Safepoints --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Place garbage collection safepoints at appropriate locations in the IR. This
// does not make relocation semantics or variable liveness explicit.  That's
// done by RewriteStatepointsForGC.
//
// Terminology:
// - A call is said to be "parseable" if there is a stack map generated for the
// return PC of the call.  A runtime can determine where values listed in the
// deopt arguments and (after RewriteStatepointsForGC) gc arguments are located
// on the stack when the code is suspended inside such a call.  Every parse
// point is represented by a call wrapped in an gc.statepoint intrinsic.
// - A "poll" is an explicit check in the generated code to determine if the
// runtime needs the generated code to cooperate by calling a helper routine
// and thus suspending its execution at a known state. The call to the helper
// routine will be parseable.  The (gc & runtime specific) logic of a poll is
// assumed to be provided in a function of the name "gc.safepoint_poll".
//
// We aim to insert polls such that running code can quickly be brought to a
// well defined state for inspection by the collector.  In the current
// implementation, this is done via the insertion of poll sites at method entry
// and the backedge of most loops.  We try to avoid inserting more polls than
// are necessary to ensure a finite period between poll sites.  This is not
// because the poll itself is expensive in the generated code; it's not.  Polls
// do tend to impact the optimizer itself in negative ways; we'd like to avoid
// perturbing the optimization of the method as much as we can.
//
// We also need to make most call sites parseable.  The callee might execute a
// poll (or otherwise be inspected by the GC).  If so, the entire stack
// (including the suspended frame of the current method) must be parseable.
//
// This pass will insert:
// - Call parse points ("call safepoints") for any call which may need to
// reach a safepoint during the execution of the callee function.
// - Backedge safepoint polls and entry safepoint polls to ensure that
// executing code reaches a safepoint poll in a finite amount of time.
//
// We do not currently support return statepoints, but adding them would not
// be hard.  They are not required for correctness - entry safepoints are an
// alternative - but some GCs may prefer them.  Patches welcome.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/PlaceSafepoints.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 54-115

```cpp
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/CFG.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/Analysis/ScalarEvolution.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Statepoint.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Scalar.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include "llvm/Transforms/Utils/Local.h"

using namespace llvm;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// An analysis pass whose purpose is to identify each of the backedges in
/// the function which require a safepoint poll to be inserted.
class PlaceBackedgeSafepointsLegacyPass : public FunctionPass {
public:
  static char ID;

  /// The output of the pass - gives a list of each backedge (described by
  /// pointing at the branch) which need a poll inserted.
  std::vector<Instruction *> PollLocations;

  /// True unless we're running spp-no-calls in which case we need to disable
  /// the call-dependent placement opts.
  bool CallSafepointsEnabled;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 116-175

```cpp
  PlaceBackedgeSafepointsLegacyPass(bool CallSafepoints = false)
      : FunctionPass(ID), CallSafepointsEnabled(CallSafepoints) {
    initializePlaceBackedgeSafepointsLegacyPassPass(
        *PassRegistry::getPassRegistry());
  }

  bool runOnLoop(Loop *);

  void runOnLoopAndSubLoops(Loop *L) {
    // Visit all the subloops
    for (Loop *I : *L)
      runOnLoopAndSubLoops(I);
    runOnLoop(L);
  }

  bool runOnFunction(Function &F) override {
    SE = &getAnalysis<ScalarEvolutionWrapperPass>().getSE();
    DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
    LI = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
    TLI = &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(F);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
static cl::opt<bool> NoBackedge("spp-no-backedge", cl::Hidden, cl::init(false));

char PlaceBackedgeSafepointsLegacyPass::ID = 0;

INITIALIZE_PASS_BEGIN(PlaceBackedgeSafepointsLegacyPass,
                      "place-backedge-safepoints-impl",
                      "Place Backedge Safepoints", false, false)
INITIALIZE_PASS_DEPENDENCY(ScalarEvolutionWrapperPass)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass)
INITIALIZE_PASS_END(PlaceBackedgeSafepointsLegacyPass,
                    "place-backedge-safepoints-impl",
                    "Place Backedge Safepoints", false, false)

```
- EN: This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include FunctionPass, runOnLoopAndSubLoops, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, ScalarEvolution, TargetLibraryInfo.
- CN: 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 FunctionPass, runOnLoopAndSubLoops，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, ScalarEvolution, TargetLibraryInfo 等分析结果。

### Lines 176-236

```cpp
static bool containsUnconditionalCallSafepoint(Loop *L, BasicBlock *Header,
                                               BasicBlock *Pred,
                                               DominatorTree &DT,
                                               const TargetLibraryInfo &TLI);

static bool mustBeFiniteCountedLoop(Loop *L, ScalarEvolution *SE,
                                    BasicBlock *Pred);

static Instruction *findLocationForEntrySafepoint(Function &F,
                                                  DominatorTree &DT);

static bool isGCSafepointPoll(Function &F);
static bool shouldRewriteFunction(Function &F);
static bool enableEntrySafepoints(Function &F);
static bool enableBackedgeSafepoints(Function &F);
static bool enableCallSafepoints(Function &F);

static void
InsertSafepointPoll(BasicBlock::iterator InsertBefore,
                    std::vector<CallBase *> &ParsePointsNeeded /*rval*/,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        // might latter loose this call safepoint.
        LLVM_DEBUG(
            dbgs()
            << "skipping safepoint placement due to unconditional call\n");
        CallInLoop++;
        continue;
      }
    }

    // TODO: We can create an inner loop which runs a finite number of
    // iterations with an outer loop which contains a safepoint.  This would
    // not help runtime performance that much, but it might help our ability to
    // optimize the inner loop.

```
- EN: Core entities appearing here include runOnLoop, containsUnconditionalCallSafepoint, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, ScalarEvolution, TargetLibraryInfo.
- CN: 此处出现的核心实体包括 runOnLoop, containsUnconditionalCallSafepoint，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, ScalarEvolution, TargetLibraryInfo 等分析结果。

### Lines 237-297

```cpp
    // Safepoint insertion would involve creating a new basic block (as the
    // target of the current backedge) which does the safepoint (of all live
    // variables) and branches to the true header
    Instruction *Term = Pred->getTerminator();

    LLVM_DEBUG(dbgs() << "[LSP] terminator instruction: " << *Term);

    PollLocations.push_back(Term);
  }

  return false;
}

bool PlaceSafepointsPass::runImpl(Function &F, const TargetLibraryInfo &TLI) {
  if (F.isDeclaration() || F.empty()) {
    // This is a declaration, nothing to do.  Must exit early to avoid crash in
    // dom tree calculation
    return false;
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  if (enableBackedgeSafepoints(F)) {
    // Construct a pass manager to run the LoopPass backedge logic.  We
    // need the pass manager to handle scheduling all the loop passes
    // appropriately.  Doing this by hand is painful and just not worth messing
    // with for the moment.
    legacy::FunctionPassManager FPM(F.getParent());
    bool CanAssumeCallSafepoints = enableCallSafepoints(F);

    FPM.add(new TargetLibraryInfoWrapperPass(TLI));
    auto *PBS = new PlaceBackedgeSafepointsLegacyPass(CanAssumeCallSafepoints);
    FPM.add(PBS);
    FPM.run(F);

```
- EN: Core entities appearing here include runImpl, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetLibraryInfo.
- CN: 此处出现的核心实体包括 runImpl，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetLibraryInfo 等分析结果。

### Lines 298-364

```cpp
    // We preserve dominance information when inserting the poll, otherwise
    // we'd have to recalculate this on every insert
    DT.recalculate(F);

    auto &PollLocations = PBS->PollLocations;

    auto OrderByBBName = [](Instruction *a, Instruction *b) {
      return a->getParent()->getName() < b->getParent()->getName();
    };
    // We need the order of list to be stable so that naming ends up stable
    // when we split edges.  This makes test cases much easier to write.
    llvm::sort(PollLocations, OrderByBBName);

    // We can sometimes end up with duplicate poll locations.  This happens if
    // a single loop is visited more than once.   The fact this happens seems
    // wrong, but it does happen for the split-backedge.ll test case.
    PollLocations.erase(llvm::unique(PollLocations), PollLocations.end());

    // Insert a poll at each point the analysis pass identified
    // The poll location must be the terminator of a loop latch block.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
    }
  }

  if (enableEntrySafepoints(F)) {
    if (Instruction *Location = findLocationForEntrySafepoint(F, DT)) {
      PollsNeeded.push_back(Location);
      Modified = true;
      NumEntrySafepoints++;
    }
    // TODO: else we should assert that there was, in fact, a policy choice to
    // not insert a entry safepoint poll.
  }

```
- EN: This region continues the PlaceSafepoints implementation with local helper logic centered on PollLocations, PBS, OrderByBBName, Instruction.
- CN: 这一段延续了 PlaceSafepoints 的主体实现，围绕 PollLocations, PBS, OrderByBBName, Instruction 等局部辅助逻辑展开。

### Lines 365-418

```cpp
  // Now that we've identified all the needed safepoint poll locations, insert
  // safepoint polls themselves.
  for (Instruction *PollLocation : PollsNeeded) {
    std::vector<CallBase *> RuntimeCalls;
    InsertSafepointPoll(PollLocation->getIterator(), RuntimeCalls, TLI);
    llvm::append_range(ParsePointNeeded, RuntimeCalls);
  }

  return Modified;
}

PreservedAnalyses PlaceSafepointsPass::run(Function &F,
                                           FunctionAnalysisManager &AM) {
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);

  if (!runImpl(F, TLI))
    return PreservedAnalyses::all();

  // TODO: can we preserve more?
  return PreservedAnalyses::none();
}

static bool needsStatepoint(CallBase *Call, const TargetLibraryInfo &TLI) {
  if (callsGCLeafFunction(Call, TLI))
    return false;
  if (auto *CI = dyn_cast<CallInst>(Call)) {
    if (CI->isInlineAsm())
      return false;
  }

  return !(isa<GCStatepointInst>(Call) || isa<GCRelocateInst>(Call) ||
           isa<GCResultInst>(Call));
}

/// Returns true if this loop is known to contain a call safepoint which
/// must unconditionally execute on any iteration of the loop which returns
/// to the loop header via an edge from Pred.  Returns a conservative correct
/// answer; i.e. false is always valid.
static bool containsUnconditionalCallSafepoint(Loop *L, BasicBlock *Header,
                                               BasicBlock *Pred,
                                               DominatorTree &DT,
                                               const TargetLibraryInfo &TLI) {
  // In general, we're looking for any cut of the graph which ensures
  // there's a call safepoint along every edge between Header and Pred.
  // For the moment, we look only for the 'cuts' that consist of a single call
  // instruction in a block which is dominated by the Header and dominates the
  // loop latch (Pred) block.  Somewhat surprisingly, walking the entire chain
  // of such dominating blocks gets substantially more occurrences than just
  // checking the Pred and Header blocks themselves.  This may be due to the
  // density of loop exit conditions caused by range and null checks.
  // TODO: structure this as an analysis pass, cache the result for subloops,
  // avoid dom tree recalculations
  assert(DT.dominates(Header, Pred) && "loop latch not dominated by header?");

```
- EN: Core entities appearing here include needsStatepoint, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetLibraryInfo.
- CN: 此处出现的核心实体包括 needsStatepoint，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetLibraryInfo 等分析结果。

### Lines 419-479

```cpp
  BasicBlock *Current = Pred;
  while (true) {
    for (Instruction &I : *Current) {
      if (auto *Call = dyn_cast<CallBase>(&I))
        // Note: Technically, needing a safepoint isn't quite the right
        // condition here.  We should instead be checking if the target method
        // has an
        // unconditional poll. In practice, this is only a theoretical concern
        // since we don't have any methods with conditional-only safepoint
        // polls.
        if (needsStatepoint(Call, TLI))
          return true;
    }

    if (Current == Header)
      break;
    Current = DT.getNode(Current)->getIDom()->getBlock();
  }

  return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  return /* not finite */ false;
}

static void scanOneBB(Instruction *Start, Instruction *End,
                      std::vector<CallInst *> &Calls,
                      DenseSet<BasicBlock *> &Seen,
                      std::vector<BasicBlock *> &Worklist) {
  for (BasicBlock::iterator BBI(Start), BBE0 = Start->getParent()->end(),
                                        BBE1 = BasicBlock::iterator(End);
       BBI != BBE0 && BBI != BBE1; BBI++) {
    if (CallInst *CI = dyn_cast<CallInst>(&*BBI))
      Calls.push_back(CI);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as ScalarEvolution, SCEV.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 ScalarEvolution, SCEV 等分析结果。

### Lines 480-538

```cpp
    // FIXME: This code does not handle invokes
    assert(!isa<InvokeInst>(&*BBI) &&
           "support for invokes in poll code needed");

    // Only add the successor blocks if we reach the terminator instruction
    // without encountering end first
    if (BBI->isTerminator()) {
      BasicBlock *BB = BBI->getParent();
      for (BasicBlock *Succ : successors(BB)) {
        if (Seen.insert(Succ).second) {
          Worklist.push_back(Succ);
        }
      }
    }
  }
}

static void scanInlinedCode(Instruction *Start, Instruction *End,
                            std::vector<CallInst *> &Calls,
                            DenseSet<BasicBlock *> &Seen) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // growth.  In particular, the optimizer likes to form things like memsets
      // out of stores in the original IR.  Another important example is
      // llvm.localescape which must occur in the entry block.  Inserting a
      // safepoint before it is not legal since it could push the localescape
      // out of the entry block.
      return true;
    }
  }
  return false;
}

static Instruction *findLocationForEntrySafepoint(Function &F,
                                                  DominatorTree &DT) {

```
- EN: Core entities appearing here include doesNotRequireEntrySafepointBefore, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 此处出现的核心实体包括 doesNotRequireEntrySafepointBefore，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 539-594

```cpp
  // Conceptually, this poll needs to be on method entry, but in
  // practice, we place it as late in the entry block as possible.  We
  // can place it as late as we want as long as it dominates all calls
  // that can grow the stack.  This, combined with backedge polls,
  // give us all the progress guarantees we need.

  // hasNextInstruction and nextInstruction are used to iterate
  // through a "straight line" execution sequence.

  auto HasNextInstruction = [](Instruction *I) {
    if (!I->isTerminator())
      return true;

    BasicBlock *nextBB = I->getParent()->getUniqueSuccessor();
    return nextBB && (nextBB->getUniquePredecessor() != nullptr);
  };

  auto NextInstruction = [&](Instruction *I) {
    assert(HasNextInstruction(I) &&
           "first check if there is a next instruction!");

    if (I->isTerminator())
      return &I->getParent()->getUniqueSuccessor()->front();
    return &*++I->getIterator();
  };

  Instruction *Cursor = nullptr;
  for (Cursor = &F.getEntryBlock().front(); HasNextInstruction(Cursor);
       Cursor = NextInstruction(Cursor)) {

    // We need to ensure a safepoint poll occurs before any 'real' call.  The
    // easiest way to ensure finite execution between safepoints in the face of
    // recursive and mutually recursive functions is to enforce that each take
    // a safepoint.  Additionally, we need to ensure a poll before any call
    // which can grow the stack by an unbounded amount.  This isn't required
    // for GC semantics per se, but is a common requirement for languages
    // which detect stack overflow via guard pages and then throw exceptions.
    if (auto *Call = dyn_cast<CallBase>(Cursor)) {
      if (doesNotRequireEntrySafepointBefore(Call))
        continue;
      break;
    }
  }

  assert((HasNextInstruction(Cursor) || Cursor->isTerminator()) &&
         "either we stopped because of a call, or because of terminator");

  return Cursor;
}

const char GCSafepointPollName[] = "gc.safepoint_poll";

static bool isGCSafepointPoll(Function &F) {
  return F.getName() == GCSafepointPollName;
}

```
- EN: Core entities appearing here include isGCSafepointPoll, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isGCSafepointPoll，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 595-655

```cpp
/// Returns true if this function should be rewritten to include safepoint
/// polls and parseable call sites.  The main point of this function is to be
/// an extension point for custom logic.
static bool shouldRewriteFunction(Function &F) {
  // TODO: This should check the GCStrategy
  if (F.hasGC()) {
    const auto &FunctionGCName = F.getGC();
    const StringRef StatepointExampleName("statepoint-example");
    const StringRef CoreCLRName("coreclr");
    return (StatepointExampleName == FunctionGCName) ||
           (CoreCLRName == FunctionGCName);
  } else
    return false;
}

// TODO: These should become properties of the GCStrategy, possibly with
// command line overrides.
static bool enableEntrySafepoints(Function &F) { return !NoEntry; }
static bool enableBackedgeSafepoints(Function &F) { return !NoBackedge; }
static bool enableCallSafepoints(Function &F) { return !NoCall; }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (Before == OrigBB->begin())
    IsBegin = true;
  else
    Before--;

  After++;
  assert(After != OrigBB->end() && "must have successor");

  // Do the actual inlining
  InlineFunctionInfo IFI;
  bool InlineStatus = InlineFunction(*PollCall, IFI).isSuccess();
  assert(InlineStatus && "inline must succeed");
  (void)InlineStatus; // suppress warning in release-asserts

```
- EN: Core entities appearing here include shouldRewriteFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 shouldRewriteFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 656-689

```cpp
  // Check post-conditions
  assert(IFI.StaticAllocas.empty() && "can't have allocs");

  std::vector<CallInst *> Calls; // new calls
  DenseSet<BasicBlock *> BBs;    // new BBs + insertee

  // Include only the newly inserted instructions, Note: begin may not be valid
  // if we inserted to the beginning of the basic block
  BasicBlock::iterator Start = IsBegin ? OrigBB->begin() : std::next(Before);

  // If your poll function includes an unreachable at the end, that's not
  // valid.  Bugpoint likes to create this, so check for it.
  assert(isPotentiallyReachable(&*Start, &*After) &&
         "malformed poll function");

  scanInlinedCode(&*Start, &*After, Calls, BBs);
  assert(!Calls.empty() && "slow path not found for safepoint poll");

  // Record the fact we need a parsable state at the runtime call contained in
  // the poll function.  This is required so that the runtime knows how to
  // parse the last frame when we actually take  the safepoint (i.e. execute
  // the slow path)
  assert(ParsePointsNeeded.empty());
  for (auto *CI : Calls) {
    // No safepoint needed or wanted
    if (!needsStatepoint(CI, TLI))
      continue;

    // These are likely runtime calls.  Should we assert that via calling
    // convention or something?
    ParsePointsNeeded.push_back(CI);
  }
  assert(ParsePointsNeeded.size() <= Calls.size());
}
```
- EN: This region continues the PlaceSafepoints implementation with local helper logic centered on Check, IFI, StaticAllocas, CallInst.
- CN: 这一段延续了 PlaceSafepoints 的主体实现，围绕 Check, IFI, StaticAllocas, CallInst 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `PlaceBackedgeSafepointsLegacyPass, FunctionPass, runOnLoopAndSubLoops, runOnLoop, containsUnconditionalCallSafepoint, runImpl, needsStatepoint, doesNotRequireEntrySafepointBefore` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PlaceBackedgeSafepointsLegacyPass, FunctionPass, runOnLoopAndSubLoops, runOnLoop, containsUnconditionalCallSafepoint, runImpl, needsStatepoint, doesNotRequireEntrySafepointBefore` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, LoopInfo, ScalarEvolution, SCEV, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, LoopInfo, ScalarEvolution, SCEV, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `AllBackedges, CountedLoopTripWidth, SplitBackedge, NoEntry, NoCall` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `AllBackedges, CountedLoopTripWidth, SplitBackedge, NoEntry, NoCall` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CFG.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CFG.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/ScalarEvolution.h`, `llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/LegacyPassManager.h`, `llvm/IR/Module.h`, `llvm/IR/Statepoint.h`, `llvm/Transforms/Scalar/PlaceSafepoints.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Dominators.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/LegacyPassManager.h`, `llvm/IR/Module.h`, `llvm/IR/Statepoint.h`, `llvm/Transforms/Scalar/PlaceSafepoints.h`, `llvm/Transforms/Scalar.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/Local.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/Statistic.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `LoopInfo`, `ScalarEvolution`, `SCEV`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
