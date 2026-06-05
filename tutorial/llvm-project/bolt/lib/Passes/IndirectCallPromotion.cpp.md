# IndirectCallPromotion.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/IndirectCallPromotion.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/IndirectCallPromotion.cpp This file implements the IndirectCallPromotion class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/IndirectCallPromotion.cpp This file implements the IndirectCallPromotion class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/IndirectCallPromotion.cpp ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the IndirectCallPromotion class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/IndirectCallPromotion.h"
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Passes/DataflowInfoManager.h"
#include "bolt/Passes/Inliner.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/CommandLine.h"
#include <iterator>
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-29

```cpp
#define DEBUG_TYPE "ICP"
#define DEBUG_VERBOSE(Level, X)                                                \
  if (opts::Verbosity >= (Level)) {                                            \
    X;                                                                         \
  }

using namespace llvm;
using namespace bolt;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `DEBUG_TYPE`, `DEBUG_VERBOSE` for constants or compile-time switches. Contains local control flow that updates state or selects among execution branches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`, `DEBUG_VERBOSE`，用于常量或编译期开关。这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 30-37

```cpp
namespace opts {

extern cl::OptionCategory BoltOptCategory;

extern cl::opt<IndirectCallPromotionType> ICP;
extern cl::opt<unsigned> Verbosity;
extern cl::opt<unsigned> ExecutionCountThreshold;
```

- EN: Works inside namespace scope `opts` to organize symbols. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`。

### Lines 38-50

```cpp
static cl::opt<unsigned> ICPJTRemainingPercentThreshold(
    "icp-jt-remaining-percent-threshold",
    cl::desc("The percentage threshold against remaining unpromoted indirect "
             "call count for the promotion for jump tables"),
    cl::init(30), cl::ZeroOrMore, cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<unsigned> ICPJTTotalPercentThreshold(
    "icp-jt-total-percent-threshold",
    cl::desc(
        "The percentage threshold against total count for the promotion for "
        "jump tables"),
    cl::init(5), cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 51-63

```cpp
static cl::opt<unsigned> ICPCallsRemainingPercentThreshold(
    "icp-calls-remaining-percent-threshold",
    cl::desc("The percentage threshold against remaining unpromoted indirect "
             "call count for the promotion for calls"),
    cl::init(50), cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<unsigned> ICPCallsTotalPercentThreshold(
    "icp-calls-total-percent-threshold",
    cl::desc(
        "The percentage threshold against total count for the promotion for "
        "calls"),
    cl::init(30), cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 64-74

```cpp
static cl::opt<unsigned> ICPMispredictThreshold(
    "indirect-call-promotion-mispredict-threshold",
    cl::desc("misprediction threshold for skipping ICP on an "
             "indirect call"),
    cl::init(0), cl::cat(BoltOptCategory));

static cl::alias ICPMispredictThresholdAlias(
    "icp-mp-threshold",
    cl::desc("alias for --indirect-call-promotion-mispredict-threshold"),
    cl::aliasopt(ICPMispredictThreshold));
```

- EN: Declares or implements routines including `init`, `desc`, `aliasopt`. Notable symbols here include `init`, `desc`, `aliasopt`.
- CN: 这里声明或实现函数，例如 `init`, `desc`, `aliasopt`。这里较值得关注的符号包括 `init`, `desc`, `aliasopt`。

### Lines 75-82

```cpp
static cl::opt<bool> ICPUseMispredicts(
    "indirect-call-promotion-use-mispredicts",
    cl::desc("use misprediction frequency for determining whether or not ICP "
             "should be applied at a callsite.  The "
             "-indirect-call-promotion-mispredict-threshold value will be used "
             "by this heuristic"),
    cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`. Notable symbols here include `cat`.
- CN: 这里声明或实现函数，例如 `cat`。这里较值得关注的符号包括 `cat`。

### Lines 83-93

```cpp
static cl::alias ICPUseMispredictsAlias(
    "icp-use-mp",
    cl::desc("alias for --indirect-call-promotion-use-mispredicts"),
    cl::aliasopt(ICPUseMispredicts));

static cl::opt<unsigned>
    ICPTopN("indirect-call-promotion-topn",
            cl::desc("limit number of targets to consider when doing indirect "
                     "call promotion. 0 = no limit"),
            cl::init(3), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `aliasopt`, `init`. Notable symbols here include `desc`, `aliasopt`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `aliasopt`, `init`。这里较值得关注的符号包括 `desc`, `aliasopt`, `init`。

### Lines 94-104

```cpp
static cl::alias
    ICPTopNAlias("icp-topn",
                 cl::desc("alias for --indirect-call-promotion-topn"),
                 cl::aliasopt(ICPTopN));

static cl::opt<unsigned> ICPCallsTopN(
    "indirect-call-promotion-calls-topn",
    cl::desc("limit number of targets to consider when doing indirect "
             "call promotion on calls. 0 = no limit"),
    cl::init(0), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `aliasopt`, `init`. Notable symbols here include `desc`, `aliasopt`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `aliasopt`, `init`。这里较值得关注的符号包括 `desc`, `aliasopt`, `init`。

### Lines 105-115

```cpp
static cl::alias ICPCallsTopNAlias(
    "icp-calls-topn",
    cl::desc("alias for --indirect-call-promotion-calls-topn"),
    cl::aliasopt(ICPCallsTopN));

static cl::opt<unsigned> ICPJumpTablesTopN(
    "indirect-call-promotion-jump-tables-topn",
    cl::desc("limit number of targets to consider when doing indirect "
             "call promotion on jump tables. 0 = no limit"),
    cl::init(0), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `aliasopt`, `init`. Notable symbols here include `desc`, `aliasopt`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `aliasopt`, `init`。这里较值得关注的符号包括 `desc`, `aliasopt`, `init`。

### Lines 116-126

```cpp
static cl::alias ICPJumpTablesTopNAlias(
    "icp-jt-topn",
    cl::desc("alias for --indirect-call-promotion-jump-tables-topn"),
    cl::aliasopt(ICPJumpTablesTopN));

static cl::opt<bool> EliminateLoads(
    "icp-eliminate-loads",
    cl::desc("enable load elimination using memory profiling data when "
             "performing ICP"),
    cl::init(true), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `aliasopt`, `init`. Notable symbols here include `desc`, `aliasopt`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `aliasopt`, `init`。这里较值得关注的符号包括 `desc`, `aliasopt`, `init`。

### Lines 127-138

```cpp
static cl::opt<unsigned> ICPTopCallsites(
    "icp-top-callsites",
    cl::desc("optimize hottest calls until at least this percentage of all "
             "indirect calls frequency is covered. 0 = all callsites"),
    cl::init(99), cl::Hidden, cl::cat(BoltOptCategory));

static cl::list<std::string>
    ICPFuncsList("icp-funcs", cl::CommaSeparated,
                 cl::desc("list of functions to enable ICP for"),
                 cl::value_desc("func1,func2,func3,..."), cl::Hidden,
                 cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `init`, `desc`, `value_desc`, `cat`. Notable symbols here include `init`, `desc`, `value_desc`, `cat`.
- CN: 这里声明或实现函数，例如 `init`, `desc`, `value_desc`, `cat`。这里较值得关注的符号包括 `init`, `desc`, `value_desc`, `cat`。

### Lines 139-149

```cpp
static cl::opt<bool>
    ICPOldCodeSequence("icp-old-code-sequence",
                       cl::desc("use old code sequence for promoted calls"),
                       cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<bool> ICPJumpTablesByTarget(
    "icp-jump-tables-targets",
    cl::desc(
        "for jump tables, optimize indirect jmp targets instead of indices"),
    cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`。

### Lines 150-158

```cpp
static cl::alias
    ICPJumpTablesByTargetAlias("icp-jt-targets",
                               cl::desc("alias for --icp-jump-tables-targets"),
                               cl::aliasopt(ICPJumpTablesByTarget));

static cl::opt<bool> ICPPeelForInline(
    "icp-inline", cl::desc("only promote call targets eligible for inlining"),
    cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `aliasopt`, `cat`. Notable symbols here include `desc`, `aliasopt`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `aliasopt`, `cat`。这里较值得关注的符号包括 `desc`, `aliasopt`, `cat`。

### Lines 159-176

```cpp
} // namespace opts

#ifndef NDEBUG
static bool verifyProfile(std::map<uint64_t, BinaryFunction> &BFs) {
  bool IsValid = true;
  for (auto &BFI : BFs) {
    BinaryFunction &BF = BFI.second;
    if (!BF.isSimple())
      continue;
    for (const BinaryBasicBlock &BB : BF) {
      auto BI = BB.branch_info_begin();
      for (BinaryBasicBlock *SuccBB : BB.successors()) {
        if (BI->Count != BinaryBasicBlock::COUNT_NO_PROFILE && BI->Count > 0) {
          if (BB.getKnownExecutionCount() == 0 ||
              SuccBB->getKnownExecutionCount() == 0) {
            BF.getBinaryContext().errs()
                << "BOLT-WARNING: profile verification failed after ICP for "
                   "function "
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `verifyProfile`, `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `verifyProfile`, `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。

### Lines 177-188

```cpp
                << BF << '\n';
            IsValid = false;
          }
        }
        ++BI;
      }
    }
  }
  return IsValid;
}
#endif
```

- EN: Uses conditional compilation to choose platform- or feature-specific code paths. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里使用条件编译，根据平台或特性切换不同代码路径。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 189-201

```cpp
namespace llvm {
namespace bolt {

IndirectCallPromotion::Callsite::Callsite(BinaryFunction &BF,
                                          const IndirectCallProfile &ICP)
    : From(BF.getSymbol()), To(ICP.Offset), Mispreds(ICP.Mispreds),
      Branches(ICP.Count) {
  if (ICP.Symbol) {
    To.Sym = ICP.Symbol;
    To.Addr = 0;
  }
}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `From`, `Branches`. Notable symbols here include `From`, `Branches`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `From`, `Branches`。这里较值得关注的符号包括 `From`, `Branches`, `llvm`, `bolt`。

### Lines 202-215

```cpp
void IndirectCallPromotion::printDecision(
    llvm::raw_ostream &OS,
    std::vector<IndirectCallPromotion::Callsite> &Targets, unsigned N) const {
  uint64_t TotalCount = 0;
  uint64_t TotalMispreds = 0;
  for (const Callsite &S : Targets) {
    TotalCount += S.Branches;
    TotalMispreds += S.Mispreds;
  }
  if (!TotalCount)
    TotalCount = 1;
  if (!TotalMispreds)
    TotalMispreds = 1;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 216-233

```cpp
  OS << "BOLT-INFO: ICP decision for call site with " << Targets.size()
     << " targets, Count = " << TotalCount << ", Mispreds = " << TotalMispreds
     << "\n";

  size_t I = 0;
  for (const Callsite &S : Targets) {
    OS << "Count = " << S.Branches << ", "
       << format("%.1f", (100.0 * S.Branches) / TotalCount) << ", "
       << "Mispreds = " << S.Mispreds << ", "
       << format("%.1f", (100.0 * S.Mispreds) / TotalMispreds);
    if (I < N)
      OS << " * to be optimized *";
    if (!S.JTIndices.empty()) {
      OS << " Indices:";
      for (const uint64_t Idx : S.JTIndices)
        OS << " " << Idx;
    }
    OS << "\n";
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 234-246

```cpp
    I += S.JTIndices.empty() ? 1 : S.JTIndices.size();
  }
}

// Get list of targets for a given call sorted by most frequently
// called first.
std::vector<IndirectCallPromotion::Callsite>
IndirectCallPromotion::getCallTargets(BinaryBasicBlock &BB,
                                      const MCInst &Inst) const {
  BinaryFunction &BF = *BB.getFunction();
  const BinaryContext &BC = BF.getBinaryContext();
  std::vector<Callsite> Targets;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 247-264

```cpp
  if (const JumpTable *JT = BF.getJumpTable(Inst)) {
    // Don't support PIC jump tables for now
    if (!opts::ICPJumpTablesByTarget && JT->Type == JumpTable::JTT_PIC)
      return Targets;
    const Location From(BF.getSymbol());
    const std::pair<size_t, size_t> Range =
        JT->getEntriesForAddress(BC.MIB->getJumpTable(Inst));
    assert(JT->Counts.empty() || JT->Counts.size() >= Range.second);
    JumpTable::JumpInfo DefaultJI;
    const JumpTable::JumpInfo *JI =
        JT->Counts.empty() ? &DefaultJI : &JT->Counts[Range.first];
    const size_t JIAdj = JT->Counts.empty() ? 0 : 1;
    assert(JT->Type == JumpTable::JTT_PIC ||
           JT->EntrySize == BC.AsmInfo->getCodePointerSize());
    for (size_t I = Range.first; I < Range.second; ++I, JI += JIAdj) {
      MCSymbol *Entry = JT->Entries[I];
      const BinaryBasicBlock *ToBB = BF.getBasicBlockForLabel(Entry);
      if (!ToBB)
```

- EN: Declares or implements routines including `From`, `getEntriesForAddress`, `assert`, `getCodePointerSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `From`, `getEntriesForAddress`, `assert`, `getCodePointerSize`.
- CN: 这里声明或实现函数，例如 `From`, `getEntriesForAddress`, `assert`, `getCodePointerSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `From`, `getEntriesForAddress`, `assert`, `getCodePointerSize`。

### Lines 265-282

```cpp
        continue;
      const Location To(Entry);
      const BinaryBasicBlock::BinaryBranchInfo &BI = BB.getBranchInfo(*ToBB);
      Targets.emplace_back(From, To, BI.MispredictedCount, BI.Count,
                           I - Range.first);
    }

    // Sort by symbol then addr.
    llvm::sort(Targets, [](const Callsite &A, const Callsite &B) {
      if (A.To.Sym && B.To.Sym)
        return A.To.Sym < B.To.Sym;
      else if (A.To.Sym && !B.To.Sym)
        return true;
      else if (!A.To.Sym && B.To.Sym)
        return false;
      else
        return A.To.Addr < B.To.Addr;
    });
```

- EN: Declares or implements routines including `To`, `sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `To`, `sort`.
- CN: 这里声明或实现函数，例如 `To`, `sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `To`, `sort`。

### Lines 283-300

```cpp

    // Targets may contain multiple entries to the same target, but using
    // different indices. Their profile will report the same number of branches
    // for different indices if the target is the same. That's because we don't
    // profile the index value, but only the target via LBR.
    auto First = Targets.begin();
    auto Last = Targets.end();
    auto Result = First;
    while (++First != Last) {
      Callsite &A = *Result;
      const Callsite &B = *First;
      if (A.To.Sym && B.To.Sym && A.To.Sym == B.To.Sym)
        A.JTIndices.insert(A.JTIndices.end(), B.JTIndices.begin(),
                           B.JTIndices.end());
      else
        *(++Result) = *First;
    }
    ++Result;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 301-313

```cpp

    LLVM_DEBUG(if (Targets.end() - Result > 0) {
      dbgs() << "BOLT-INFO: ICP: " << (Targets.end() - Result)
             << " duplicate targets removed\n";
    });

    Targets.erase(Result, Targets.end());
  } else {
    // Don't try to optimize PC relative indirect calls.
    if (Inst.getOperand(0).isReg() &&
        Inst.getOperand(0).getReg() == BC.MRI->getProgramCounter())
      return Targets;
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `dbgs`。

### Lines 314-324

```cpp
    const auto ICSP = BC.MIB->tryGetAnnotationAs<IndirectCallSiteProfile>(
        Inst, "CallProfile");
    if (ICSP) {
      for (const IndirectCallProfile &CSP : ICSP.get()) {
        Callsite Site(BF, CSP);
        if (Site.isValid())
          Targets.emplace_back(std::move(Site));
      }
    }
  }
```

- EN: Declares or implements routines including `Site`. Notable symbols here include `Site`.
- CN: 这里声明或实现函数，例如 `Site`。这里较值得关注的符号包括 `Site`。

### Lines 325-337

```cpp
  // Sort by target count, number of indices in case of jump table, and
  // mispredicts. We prioritize targets with high count, small number of indices
  // and high mispredicts. Break ties by selecting targets with lower addresses.
  llvm::stable_sort(Targets, [](const Callsite &A, const Callsite &B) {
    if (A.Branches != B.Branches)
      return A.Branches > B.Branches;
    if (A.JTIndices.size() != B.JTIndices.size())
      return A.JTIndices.size() < B.JTIndices.size();
    if (A.Mispreds != B.Mispreds)
      return A.Mispreds > B.Mispreds;
    return A.To.Addr < B.To.Addr;
  });
```

- EN: Declares or implements routines including `stable_sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `stable_sort`.
- CN: 这里声明或实现函数，例如 `stable_sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `stable_sort`。

### Lines 338-352

```cpp
  // Remove non-symbol targets
  llvm::erase_if(Targets, [](const Callsite &CS) { return !CS.To.Sym; });

  LLVM_DEBUG(if (BF.getJumpTable(Inst)) {
    uint64_t TotalCount = 0;
    uint64_t TotalMispreds = 0;
    for (const Callsite &S : Targets) {
      TotalCount += S.Branches;
      TotalMispreds += S.Mispreds;
    }
    if (!TotalCount)
      TotalCount = 1;
    if (!TotalMispreds)
      TotalMispreds = 1;
```

- EN: Declares or implements routines including `erase_if`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `erase_if`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `erase_if`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `erase_if`, `LLVM_DEBUG`。

### Lines 353-366

```cpp
    dbgs() << "BOLT-INFO: ICP: jump table size = " << Targets.size()
           << ", Count = " << TotalCount << ", Mispreds = " << TotalMispreds
           << "\n";

    size_t I = 0;
    for (const Callsite &S : Targets) {
      dbgs() << "Count[" << I << "] = " << S.Branches << ", "
             << format("%.1f", (100.0 * S.Branches) / TotalCount) << ", "
             << "Mispreds[" << I << "] = " << S.Mispreds << ", "
             << format("%.1f", (100.0 * S.Mispreds) / TotalMispreds) << "\n";
      ++I;
    }
  });
```

- EN: Declares or implements routines including `dbgs`, `format`. Notable symbols here include `dbgs`, `format`.
- CN: 这里声明或实现函数，例如 `dbgs`, `format`。这里较值得关注的符号包括 `dbgs`, `format`。

### Lines 367-376

```cpp
  return Targets;
}

IndirectCallPromotion::JumpTableInfoType
IndirectCallPromotion::maybeGetHotJumpTableTargets(BinaryBasicBlock &BB,
                                                   MCInst &CallInst,
                                                   MCInst *&TargetFetchInst,
                                                   const JumpTable *JT) const {
  assert(JT && "Can't get jump table addrs for non-jump tables.");
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 377-394

```cpp
  BinaryFunction &Function = *BB.getFunction();
  BinaryContext &BC = Function.getBinaryContext();

  if (!Function.hasMemoryProfile() || !opts::EliminateLoads)
    return JumpTableInfoType();

  JumpTableInfoType HotTargets;
  MCInst *MemLocInstr;
  MCInst *PCRelBaseOut;
  MCInst *FixedEntryLoadInstr;
  unsigned BaseReg, IndexReg;
  int64_t DispValue;
  const MCExpr *DispExpr;
  MutableArrayRef<MCInst> Insts(&BB.front(), &CallInst);
  const IndirectBranchType Type = BC.MIB->analyzeIndirectBranch(
      CallInst, Insts.begin(), Insts.end(), BC.AsmInfo->getCodePointerSize(),
      MemLocInstr, BaseReg, IndexReg, DispValue, DispExpr, PCRelBaseOut,
      FixedEntryLoadInstr);
```

- EN: Declares or implements routines including `Insts`. Notable symbols here include `Insts`.
- CN: 这里声明或实现函数，例如 `Insts`。这里较值得关注的符号包括 `Insts`。

### Lines 395-409

```cpp

  assert(MemLocInstr && "There should always be a load for jump tables");
  if (!MemLocInstr)
    return JumpTableInfoType();

  LLVM_DEBUG({
    dbgs() << "BOLT-INFO: ICP attempting to find memory profiling data for "
           << "jump table in " << Function << " at @ "
           << (&CallInst - &BB.front()) << "\n"
           << "BOLT-INFO: ICP target fetch instructions:\n";
    BC.printInstruction(dbgs(), *MemLocInstr, 0, &Function);
    if (MemLocInstr != &CallInst)
      BC.printInstruction(dbgs(), CallInst, 0, &Function);
  });
```

- EN: Declares or implements routines including `assert`, `dbgs`. Notable symbols here include `assert`, `dbgs`.
- CN: 这里声明或实现函数，例如 `assert`, `dbgs`。这里较值得关注的符号包括 `assert`, `dbgs`。

### Lines 410-420

```cpp
  DEBUG_VERBOSE(1, {
    dbgs() << "Jmp info: Type = " << (unsigned)Type << ", "
           << "BaseReg = " << BC.MRI->getName(BaseReg) << ", "
           << "IndexReg = " << BC.MRI->getName(IndexReg) << ", "
           << "DispValue = " << Twine::utohexstr(DispValue) << ", "
           << "DispExpr = " << DispExpr << ", "
           << "MemLocInstr = ";
    BC.printInstruction(dbgs(), *MemLocInstr, 0, &Function);
    dbgs() << "\n";
  });
```

- EN: Declares or implements routines including `dbgs`, `getName`, `utohexstr`. Notable symbols here include `dbgs`, `getName`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `dbgs`, `getName`, `utohexstr`。这里较值得关注的符号包括 `dbgs`, `getName`, `utohexstr`。

### Lines 421-432

```cpp
  ++TotalIndexBasedCandidates;

  auto ErrorOrMemAccessProfile =
      BC.MIB->tryGetAnnotationAs<MemoryAccessProfile>(*MemLocInstr,
                                                      "MemoryAccessProfile");
  if (!ErrorOrMemAccessProfile) {
    DEBUG_VERBOSE(1, dbgs()
                         << "BOLT-INFO: ICP no memory profiling data found\n");
    return JumpTableInfoType();
  }
  MemoryAccessProfile &MemAccessProfile = ErrorOrMemAccessProfile.get();
```

- EN: Declares or implements routines including `DEBUG_VERBOSE`. Notable symbols here include `DEBUG_VERBOSE`.
- CN: 这里声明或实现函数，例如 `DEBUG_VERBOSE`。这里较值得关注的符号包括 `DEBUG_VERBOSE`。

### Lines 433-442

```cpp
  uint64_t ArrayStart;
  if (DispExpr) {
    ErrorOr<uint64_t> DispValueOrError =
        BC.getSymbolValue(*BC.MIB->getTargetSymbol(DispExpr));
    assert(DispValueOrError && "global symbol needs a value");
    ArrayStart = *DispValueOrError;
  } else {
    ArrayStart = static_cast<uint64_t>(DispValue);
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 443-451

```cpp
  if (BaseReg == BC.MRI->getProgramCounter())
    ArrayStart += Function.getAddress() + MemAccessProfile.NextInstrOffset;

  // This is a map of [symbol] -> [count, index] and is used to combine indices
  // into the jump table since there may be multiple addresses that all have the
  // same entry.
  std::map<MCSymbol *, std::pair<uint64_t, uint64_t>> HotTargetMap;
  const std::pair<size_t, size_t> Range = JT->getEntriesForAddress(ArrayStart);
```

- EN: Declares or implements routines including `getEntriesForAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getEntriesForAddress`.
- CN: 这里声明或实现函数，例如 `getEntriesForAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getEntriesForAddress`。

### Lines 452-460

```cpp
  for (const AddressAccess &AccessInfo : MemAccessProfile.AddressAccessInfo) {
    size_t Index;
    // Mem data occasionally includes nullprs, ignore them.
    if (!AccessInfo.MemoryObject && !AccessInfo.Offset)
      continue;

    if (AccessInfo.Offset % JT->EntrySize != 0) // ignore bogus data
      return JumpTableInfoType();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 461-471

```cpp
    if (AccessInfo.MemoryObject) {
      // Deal with bad/stale data
      if (!AccessInfo.MemoryObject->getName().starts_with(
              "JUMP_TABLE/" + Function.getOneName().str()))
        return JumpTableInfoType();
      Index =
          (AccessInfo.Offset - (ArrayStart - JT->getAddress())) / JT->EntrySize;
    } else {
      Index = (AccessInfo.Offset - ArrayStart) / JT->EntrySize;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 472-479

```cpp
    // If Index is out of range it probably means the memory profiling data is
    // wrong for this instruction, bail out.
    if (Index >= Range.second) {
      LLVM_DEBUG(dbgs() << "BOLT-INFO: Index out of range of " << Range.first
                        << ", " << Range.second << "\n");
      return JumpTableInfoType();
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 480-497

```cpp
    // Make sure the hot index points at a legal label corresponding to a BB,
    // e.g. not the end of function (unreachable) label.
    if (!Function.getBasicBlockForLabel(JT->Entries[Index + Range.first])) {
      LLVM_DEBUG({
        dbgs() << "BOLT-INFO: hot index " << Index << " pointing at bogus "
               << "label " << JT->Entries[Index + Range.first]->getName()
               << " in jump table:\n";
        JT->print(dbgs());
        dbgs() << "HotTargetMap:\n";
        for (std::pair<MCSymbol *const, std::pair<uint64_t, uint64_t>> &HT :
             HotTargetMap)
          dbgs() << "BOLT-INFO: " << HT.first->getName()
                 << " = (count=" << HT.second.first
                 << ", index=" << HT.second.second << ")\n";
      });
      return JumpTableInfoType();
    }
```

- EN: Declares or implements routines including `dbgs`, `getName`, `print`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`, `getName`, `print`.
- CN: 这里声明或实现函数，例如 `dbgs`, `getName`, `print`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`, `getName`, `print`。

### Lines 498-506

```cpp
    std::pair<uint64_t, uint64_t> &HotTarget =
        HotTargetMap[JT->Entries[Index + Range.first]];
    HotTarget.first += AccessInfo.Count;
    HotTarget.second = Index;
  }

  llvm::copy(llvm::make_second_range(HotTargetMap),
             std::back_inserter(HotTargets));
```

- EN: Declares or implements routines including `copy`, `back_inserter`. Notable symbols here include `copy`, `back_inserter`.
- CN: 这里声明或实现函数，例如 `copy`, `back_inserter`。这里较值得关注的符号包括 `copy`, `back_inserter`。

### Lines 507-516

```cpp
  // Sort with highest counts first.
  llvm::sort(reverse(HotTargets));

  LLVM_DEBUG({
    dbgs() << "BOLT-INFO: ICP jump table hot targets:\n";
    for (const std::pair<uint64_t, uint64_t> &Target : HotTargets)
      dbgs() << "BOLT-INFO:  Idx = " << Target.second << ", "
             << "Count = " << Target.first << "\n";
  });
```

- EN: Declares or implements routines including `sort`, `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`, `dbgs`.
- CN: 这里声明或实现函数，例如 `sort`, `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`, `dbgs`。

### Lines 517-532

```cpp
  BC.MIB->getOrCreateAnnotationAs<uint16_t>(CallInst, "JTIndexReg") = IndexReg;

  TargetFetchInst = MemLocInstr;

  return HotTargets;
}

IndirectCallPromotion::SymTargetsType
IndirectCallPromotion::findCallTargetSymbols(std::vector<Callsite> &Targets,
                                             size_t &N, BinaryBasicBlock &BB,
                                             MCInst &CallInst,
                                             MCInst *&TargetFetchInst) const {
  const BinaryContext &BC = BB.getFunction()->getBinaryContext();
  const JumpTable *JT = BB.getFunction()->getJumpTable(CallInst);
  SymTargetsType SymTargets;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 533-542

```cpp
  if (!JT) {
    for (size_t I = 0; I < N; ++I) {
      assert(Targets[I].To.Sym && "All ICP targets must be to known symbols");
      assert(Targets[I].JTIndices.empty() &&
             "Can't have jump table indices for non-jump tables");
      SymTargets.emplace_back(Targets[I].To.Sym, 0);
    }
    return SymTargets;
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 543-556

```cpp
  // Use memory profile to select hot targets.
  JumpTableInfoType HotTargets =
      maybeGetHotJumpTableTargets(BB, CallInst, TargetFetchInst, JT);

  auto findTargetsIndex = [&](uint64_t JTIndex) {
    for (size_t I = 0; I < Targets.size(); ++I)
      if (llvm::is_contained(Targets[I].JTIndices, JTIndex))
        return I;
    LLVM_DEBUG(dbgs() << "BOLT-ERROR: Unable to find target index for hot jump "
                      << " table entry in " << *BB.getFunction() << "\n");
    llvm_unreachable("Hot indices must be referred to by at least one "
                     "callsite");
  };
```

- EN: Declares or implements routines including `maybeGetHotJumpTableTargets`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `maybeGetHotJumpTableTargets`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `maybeGetHotJumpTableTargets`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `maybeGetHotJumpTableTargets`, `LLVM_DEBUG`。

### Lines 557-574

```cpp
  if (!HotTargets.empty()) {
    if (opts::Verbosity >= 1)
      for (size_t I = 0; I < HotTargets.size(); ++I)
        BC.outs() << "BOLT-INFO: HotTarget[" << I << "] = ("
                  << HotTargets[I].first << ", " << HotTargets[I].second
                  << ")\n";

    // Recompute hottest targets, now discriminating which index is hot
    // NOTE: This is a tradeoff. On one hand, we get index information. On the
    // other hand, info coming from the memory profile is much less accurate
    // than LBRs. So we may actually end up working with more coarse
    // profile granularity in exchange for information about indices.
    std::vector<Callsite> NewTargets;
    std::map<const MCSymbol *, uint32_t> IndicesPerTarget;
    uint64_t TotalMemAccesses = 0;
    for (size_t I = 0; I < HotTargets.size(); ++I) {
      const uint64_t TargetIndex = findTargetsIndex(HotTargets[I].second);
      ++IndicesPerTarget[Targets[TargetIndex].To.Sym];
```

- EN: Declares or implements routines including `findTargetsIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `findTargetsIndex`.
- CN: 这里声明或实现函数，例如 `findTargetsIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `findTargetsIndex`。

### Lines 575-592

```cpp
      TotalMemAccesses += HotTargets[I].first;
    }
    uint64_t RemainingMemAccesses = TotalMemAccesses;
    const size_t TopN =
        opts::ICPJumpTablesTopN ? opts::ICPJumpTablesTopN : opts::ICPTopN;
    size_t I = 0;
    for (; I < HotTargets.size(); ++I) {
      const uint64_t MemAccesses = HotTargets[I].first;
      if (100 * MemAccesses <
          TotalMemAccesses * opts::ICPJTTotalPercentThreshold)
        break;
      if (100 * MemAccesses <
          RemainingMemAccesses * opts::ICPJTRemainingPercentThreshold)
        break;
      if (TopN && I >= TopN)
        break;
      RemainingMemAccesses -= MemAccesses;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 593-610

```cpp
      const uint64_t JTIndex = HotTargets[I].second;
      Callsite &Target = Targets[findTargetsIndex(JTIndex)];

      NewTargets.push_back(Target);
      std::vector<uint64_t>({JTIndex}).swap(NewTargets.back().JTIndices);
      llvm::erase(Target.JTIndices, JTIndex);

      // Keep fixCFG counts sane if more indices use this same target later
      assert(IndicesPerTarget[Target.To.Sym] > 0 && "wrong map");
      NewTargets.back().Branches =
          Target.Branches / IndicesPerTarget[Target.To.Sym];
      NewTargets.back().Mispreds =
          Target.Mispreds / IndicesPerTarget[Target.To.Sym];
      assert(Target.Branches >= NewTargets.back().Branches);
      assert(Target.Mispreds >= NewTargets.back().Mispreds);
      Target.Branches -= NewTargets.back().Branches;
      Target.Mispreds -= NewTargets.back().Mispreds;
    }
```

- EN: Declares or implements routines including `erase`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `erase`, `assert`.
- CN: 这里声明或实现函数，例如 `erase`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `erase`, `assert`。

### Lines 611-622

```cpp
    llvm::copy(Targets, std::back_inserter(NewTargets));
    std::swap(NewTargets, Targets);
    N = I;

    if (N == 0 && opts::Verbosity >= 1) {
      BC.outs() << "BOLT-INFO: ICP failed in " << *BB.getFunction() << " in "
                << BB.getName() << ": failed to meet thresholds after memory "
                << "profile data was loaded.\n";
      return SymTargets;
    }
  }
```

- EN: Declares or implements routines including `copy`, `swap`. Notable symbols here include `copy`, `swap`.
- CN: 这里声明或实现函数，例如 `copy`, `swap`。这里较值得关注的符号包括 `copy`, `swap`。

### Lines 623-632

```cpp
  for (size_t I = 0, TgtIdx = 0; I < N; ++TgtIdx) {
    Callsite &Target = Targets[TgtIdx];
    assert(Target.To.Sym && "All ICP targets must be to known symbols");
    assert(!Target.JTIndices.empty() && "Jump tables must have indices");
    for (uint64_t Idx : Target.JTIndices) {
      SymTargets.emplace_back(Target.To.Sym, Idx);
      ++I;
    }
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 633-645

```cpp
  return SymTargets;
}

IndirectCallPromotion::MethodInfoType IndirectCallPromotion::maybeGetVtableSyms(
    BinaryBasicBlock &BB, MCInst &Inst,
    const SymTargetsType &SymTargets) const {
  BinaryFunction &Function = *BB.getFunction();
  BinaryContext &BC = Function.getBinaryContext();
  std::vector<std::pair<MCSymbol *, uint64_t>> VtableSyms;
  std::vector<MCInst *> MethodFetchInsns;
  unsigned VtableReg, MethodReg;
  uint64_t MethodOffset;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 646-661

```cpp
  assert(!Function.getJumpTable(Inst) &&
         "Can't get vtable addrs for jump tables.");

  if (!Function.hasMemoryProfile() || !opts::EliminateLoads)
    return MethodInfoType();

  MutableArrayRef<MCInst> Insts(&BB.front(), &Inst + 1);
  if (!BC.MIB->analyzeVirtualMethodCall(Insts.begin(), Insts.end(),
                                        MethodFetchInsns, VtableReg, MethodReg,
                                        MethodOffset)) {
    DEBUG_VERBOSE(
        1, dbgs() << "BOLT-INFO: ICP unable to analyze method call in "
                  << Function << " at @ " << (&Inst - &BB.front()) << "\n");
    return MethodInfoType();
  }
```

- EN: Declares or implements routines including `assert`, `Insts`, `dbgs`. Notable symbols here include `assert`, `Insts`, `dbgs`.
- CN: 这里声明或实现函数，例如 `assert`, `Insts`, `dbgs`。这里较值得关注的符号包括 `assert`, `Insts`, `dbgs`。

### Lines 662-670

```cpp
  ++TotalMethodLoadEliminationCandidates;

  DEBUG_VERBOSE(1, {
    dbgs() << "BOLT-INFO: ICP found virtual method call in " << Function
           << " at @ " << (&Inst - &BB.front()) << "\n";
    dbgs() << "BOLT-INFO: ICP method fetch instructions:\n";
    for (MCInst *Inst : MethodFetchInsns)
      BC.printInstruction(dbgs(), *Inst, 0, &Function);
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 671-685

```cpp
    if (MethodFetchInsns.back() != &Inst)
      BC.printInstruction(dbgs(), Inst, 0, &Function);
  });

  // Try to get value profiling data for the method load instruction.
  auto ErrorOrMemAccessProfile =
      BC.MIB->tryGetAnnotationAs<MemoryAccessProfile>(*MethodFetchInsns.back(),
                                                      "MemoryAccessProfile");
  if (!ErrorOrMemAccessProfile) {
    DEBUG_VERBOSE(1, dbgs()
                         << "BOLT-INFO: ICP no memory profiling data found\n");
    return MethodInfoType();
  }
  MemoryAccessProfile &MemAccessProfile = ErrorOrMemAccessProfile.get();
```

- EN: Declares or implements routines including `DEBUG_VERBOSE`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `DEBUG_VERBOSE`.
- CN: 这里声明或实现函数，例如 `DEBUG_VERBOSE`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `DEBUG_VERBOSE`。

### Lines 686-693

```cpp
  // Find the vtable that each method belongs to.
  std::map<const MCSymbol *, uint64_t> MethodToVtable;

  for (const AddressAccess &AccessInfo : MemAccessProfile.AddressAccessInfo) {
    uint64_t Address = AccessInfo.Offset;
    if (AccessInfo.MemoryObject)
      Address += AccessInfo.MemoryObject->getAddress();
```

- EN: Declares or implements routines including `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`.
- CN: 这里声明或实现函数，例如 `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`。

### Lines 694-703

```cpp
    // Ignore bogus data.
    if (!Address)
      continue;

    const uint64_t VtableBase = Address - MethodOffset;

    DEBUG_VERBOSE(1, dbgs() << "BOLT-INFO: ICP vtable = "
                            << Twine::utohexstr(VtableBase) << "+"
                            << MethodOffset << "/" << AccessInfo.Count << "\n");
```

- EN: Declares or implements routines including `DEBUG_VERBOSE`, `utohexstr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `DEBUG_VERBOSE`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `DEBUG_VERBOSE`, `utohexstr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `DEBUG_VERBOSE`, `utohexstr`。

### Lines 704-718

```cpp
    if (ErrorOr<uint64_t> MethodAddr = BC.getPointerAtAddress(Address)) {
      BinaryData *MethodBD = BC.getBinaryDataAtAddress(MethodAddr.get());
      if (!MethodBD) // skip unknown methods
        continue;
      MCSymbol *MethodSym = MethodBD->getSymbol();
      MethodToVtable[MethodSym] = VtableBase;
      DEBUG_VERBOSE(1, {
        const BinaryFunction *Method = BC.getFunctionForSymbol(MethodSym);
        dbgs() << "BOLT-INFO: ICP found method = "
               << Twine::utohexstr(MethodAddr.get()) << "/"
               << (Method ? Method->getPrintName() : "") << "\n";
      });
    }
  }
```

- EN: Declares or implements routines including `getSymbol`, `dbgs`, `utohexstr`. Notable symbols here include `getSymbol`, `dbgs`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `getSymbol`, `dbgs`, `utohexstr`。这里较值得关注的符号包括 `getSymbol`, `dbgs`, `utohexstr`。

### Lines 719-734

```cpp
  // Find the vtable for each target symbol.
  for (size_t I = 0; I < SymTargets.size(); ++I) {
    auto Itr = MethodToVtable.find(SymTargets[I].first);
    if (Itr != MethodToVtable.end()) {
      if (BinaryData *BD = BC.getBinaryDataContainingAddress(Itr->second)) {
        const uint64_t Addend = Itr->second - BD->getAddress();
        VtableSyms.emplace_back(BD->getSymbol(), Addend);
        continue;
      }
    }
    // Give up if we can't find the vtable for a method.
    DEBUG_VERBOSE(1, dbgs() << "BOLT-INFO: ICP can't find vtable for "
                            << SymTargets[I].first->getName() << "\n");
    return MethodInfoType();
  }
```

- EN: Declares or implements routines including `getAddress`, `DEBUG_VERBOSE`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`, `DEBUG_VERBOSE`, `getName`.
- CN: 这里声明或实现函数，例如 `getAddress`, `DEBUG_VERBOSE`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`, `DEBUG_VERBOSE`, `getName`。

### Lines 735-744

```cpp
  // Make sure the vtable reg is not clobbered by the argument passing code
  if (VtableReg != MethodReg) {
    for (MCInst *CurInst = MethodFetchInsns.front(); CurInst < &Inst;
         ++CurInst) {
      const MCInstrDesc &InstrInfo = BC.MII->get(CurInst->getOpcode());
      if (InstrInfo.hasDefOfPhysReg(*CurInst, VtableReg, *BC.MRI))
        return MethodInfoType();
    }
  }
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 745-755

```cpp
  return MethodInfoType(VtableSyms, MethodFetchInsns);
}

std::vector<std::unique_ptr<BinaryBasicBlock>>
IndirectCallPromotion::rewriteCall(
    BinaryBasicBlock &IndCallBlock, const MCInst &CallInst,
    MCPlusBuilder::BlocksVectorTy &&ICPcode,
    const std::vector<MCInst *> &MethodFetchInsns) const {
  BinaryFunction &Function = *IndCallBlock.getFunction();
  MCPlusBuilder *MIB = Function.getBinaryContext().MIB.get();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 756-769

```cpp
  // Create new basic blocks with correct code in each one first.
  std::vector<std::unique_ptr<BinaryBasicBlock>> NewBBs;
  const bool IsTailCallOrJT =
      (MIB->isTailCall(CallInst) || Function.getJumpTable(CallInst));

  // If we are tracking the indirect call/jump address, propagate the address to
  // the ICP code.
  const std::optional<uint32_t> IndirectInstrOffset = MIB->getOffset(CallInst);
  if (IndirectInstrOffset) {
    for (auto &[Symbol, Instructions] : ICPcode)
      for (MCInst &Inst : Instructions)
        MIB->setOffset(Inst, *IndirectInstrOffset);
  }
```

- EN: Declares or implements routines including `getOffset`, `setOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOffset`, `setOffset`.
- CN: 这里声明或实现函数，例如 `getOffset`, `setOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOffset`, `setOffset`。

### Lines 770-780

```cpp
  // Move instructions from the tail of the original call block
  // to the merge block.

  // Remember any pseudo instructions following a tail call.  These
  // must be preserved and moved to the original block.
  InstructionListType TailInsts;
  const MCInst *TailInst = &CallInst;
  if (IsTailCallOrJT)
    while (TailInst != &IndCallBlock.back() && MIB->isPseudo(*(TailInst + 1)))
      TailInsts.push_back(*++TailInst);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 781-788

```cpp
  InstructionListType MovedInst = IndCallBlock.splitInstructions(&CallInst);
  // Link new BBs to the original input offset of the indirect call site or its
  // containing BB, so we can map samples recorded in new BBs back to the
  // original BB seen in the input binary (if using BAT).
  const uint32_t OrigOffset = IndirectInstrOffset
                                  ? *IndirectInstrOffset
                                  : IndCallBlock.getInputOffset();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 789-799

```cpp
  IndCallBlock.eraseInstructions(MethodFetchInsns.begin(),
                                 MethodFetchInsns.end());
  if (IndCallBlock.empty() ||
      (!MethodFetchInsns.empty() && MethodFetchInsns.back() == &CallInst))
    IndCallBlock.addInstructions(ICPcode.front().second.begin(),
                                 ICPcode.front().second.end());
  else
    IndCallBlock.replaceInstruction(std::prev(IndCallBlock.end()),
                                    ICPcode.front().second);
  IndCallBlock.addInstructions(TailInsts.begin(), TailInsts.end());
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 800-812

```cpp
  for (auto Itr = ICPcode.begin() + 1; Itr != ICPcode.end(); ++Itr) {
    MCSymbol *&Sym = Itr->first;
    InstructionListType &Insts = Itr->second;
    assert(Sym);
    std::unique_ptr<BinaryBasicBlock> TBB = Function.createBasicBlock(Sym);
    TBB->setOffset(OrigOffset);
    for (MCInst &Inst : Insts) // sanitize new instructions.
      if (MIB->isCall(Inst))
        MIB->removeAnnotation(Inst, "CallProfile");
    TBB->addInstructions(Insts.begin(), Insts.end());
    NewBBs.emplace_back(std::move(TBB));
  }
```

- EN: Declares or implements routines including `assert`, `setOffset`, `removeAnnotation`, `addInstructions`. Notable symbols here include `assert`, `setOffset`, `removeAnnotation`, `addInstructions`.
- CN: 这里声明或实现函数，例如 `assert`, `setOffset`, `removeAnnotation`, `addInstructions`。这里较值得关注的符号包括 `assert`, `setOffset`, `removeAnnotation`, `addInstructions`。

### Lines 813-820

```cpp
  // Move tail of instructions from after the original call to
  // the merge block.
  if (!IsTailCallOrJT)
    NewBBs.back()->addInstructions(MovedInst.begin(), MovedInst.end());

  return NewBBs;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 821-829

```cpp
BinaryBasicBlock *
IndirectCallPromotion::fixCFG(BinaryBasicBlock &IndCallBlock,
                              const bool IsTailCall, const bool IsJumpTable,
                              IndirectCallPromotion::BasicBlocksVector &&NewBBs,
                              const std::vector<Callsite> &Targets) const {
  BinaryFunction &Function = *IndCallBlock.getFunction();
  using BinaryBranchInfo = BinaryBasicBlock::BinaryBranchInfo;
  BinaryBasicBlock *MergeBlock = nullptr;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 830-847

```cpp
  // Scale indirect call counts to the execution count of the original
  // basic block containing the indirect call.
  uint64_t TotalCount = IndCallBlock.getKnownExecutionCount();
  uint64_t TotalIndirectBranches = 0;
  for (const Callsite &Target : Targets)
    TotalIndirectBranches += Target.Branches;
  if (TotalIndirectBranches == 0)
    TotalIndirectBranches = 1;
  BinaryBasicBlock::BranchInfoType BBI;
  BinaryBasicBlock::BranchInfoType ScaledBBI;
  for (const Callsite &Target : Targets) {
    const size_t NumEntries =
        std::max(static_cast<std::size_t>(1UL), Target.JTIndices.size());
    for (size_t I = 0; I < NumEntries; ++I) {
      BBI.push_back(
          BinaryBranchInfo{(Target.Branches + NumEntries - 1) / NumEntries,
                           (Target.Mispreds + NumEntries - 1) / NumEntries});
      ScaledBBI.push_back(
```

- EN: Declares or implements routines including `max`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `max`.
- CN: 这里声明或实现函数，例如 `max`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `max`。

### Lines 848-858

```cpp
          BinaryBranchInfo{uint64_t(TotalCount * Target.Branches /
                                    (NumEntries * TotalIndirectBranches)),
                           uint64_t(TotalCount * Target.Mispreds /
                                    (NumEntries * TotalIndirectBranches))});
    }
  }

  if (IsJumpTable) {
    BinaryBasicBlock *NewIndCallBlock = NewBBs.back().get();
    IndCallBlock.moveAllSuccessorsTo(NewIndCallBlock);
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 859-868

```cpp
    std::vector<MCSymbol *> SymTargets;
    for (const Callsite &Target : Targets) {
      const size_t NumEntries =
          std::max(static_cast<std::size_t>(1UL), Target.JTIndices.size());
      for (size_t I = 0; I < NumEntries; ++I)
        SymTargets.push_back(Target.To.Sym);
    }
    assert(SymTargets.size() > NewBBs.size() - 1 &&
           "There must be a target symbol associated with each new BB.");
```

- EN: Declares or implements routines including `max`, `assert`. Notable symbols here include `max`, `assert`.
- CN: 这里声明或实现函数，例如 `max`, `assert`。这里较值得关注的符号包括 `max`, `assert`。

### Lines 869-876

```cpp
    for (uint64_t I = 0; I < NewBBs.size(); ++I) {
      BinaryBasicBlock *SourceBB = I ? NewBBs[I - 1].get() : &IndCallBlock;
      SourceBB->setExecutionCount(TotalCount);

      BinaryBasicBlock *TargetBB =
          Function.getBasicBlockForLabel(SymTargets[I]);
      SourceBB->addSuccessor(TargetBB, ScaledBBI[I]); // taken
```

- EN: Declares or implements routines including `setExecutionCount`, `addSuccessor`. Notable symbols here include `setExecutionCount`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `setExecutionCount`, `addSuccessor`。这里较值得关注的符号包括 `setExecutionCount`, `addSuccessor`。

### Lines 877-887

```cpp
      TotalCount -= ScaledBBI[I].Count;
      SourceBB->addSuccessor(NewBBs[I].get(), TotalCount); // fall-through

      // Update branch info for the indirect jump.
      BinaryBasicBlock::BinaryBranchInfo &BranchInfo =
          NewIndCallBlock->getBranchInfo(*TargetBB);
      if (BranchInfo.Count > BBI[I].Count)
        BranchInfo.Count -= BBI[I].Count;
      else
        BranchInfo.Count = 0;
```

- EN: Declares or implements routines including `addSuccessor`, `getBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addSuccessor`, `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `addSuccessor`, `getBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addSuccessor`, `getBranchInfo`。

### Lines 888-897

```cpp
      if (BranchInfo.MispredictedCount > BBI[I].MispredictedCount)
        BranchInfo.MispredictedCount -= BBI[I].MispredictedCount;
      else
        BranchInfo.MispredictedCount = 0;
    }
  } else {
    assert(NewBBs.size() >= 2);
    assert(NewBBs.size() % 2 == 1 || IndCallBlock.succ_empty());
    assert(NewBBs.size() % 2 == 1 || IsTailCall);
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 898-909

```cpp
    auto ScaledBI = ScaledBBI.begin();
    auto updateCurrentBranchInfo = [&] {
      assert(ScaledBI != ScaledBBI.end());
      TotalCount -= ScaledBI->Count;
      ++ScaledBI;
    };

    if (!IsTailCall) {
      MergeBlock = NewBBs.back().get();
      IndCallBlock.moveAllSuccessorsTo(MergeBlock);
    }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 910-927

```cpp
    // Fix up successors and execution counts.
    updateCurrentBranchInfo();
    IndCallBlock.addSuccessor(NewBBs[1].get(), TotalCount);
    IndCallBlock.addSuccessor(NewBBs[0].get(), ScaledBBI[0]);

    const size_t Adj = IsTailCall ? 1 : 2;
    for (size_t I = 0; I < NewBBs.size() - Adj; ++I) {
      assert(TotalCount <= IndCallBlock.getExecutionCount() ||
             TotalCount <= uint64_t(TotalIndirectBranches));
      uint64_t ExecCount = ScaledBBI[(I + 1) / 2].Count;
      if (I % 2 == 0) {
        if (MergeBlock)
          NewBBs[I]->addSuccessor(MergeBlock, ScaledBBI[(I + 1) / 2].Count);
      } else {
        assert(I + 2 < NewBBs.size());
        updateCurrentBranchInfo();
        NewBBs[I]->addSuccessor(NewBBs[I + 2].get(), TotalCount);
        NewBBs[I]->addSuccessor(NewBBs[I + 1].get(), ScaledBBI[(I + 1) / 2]);
```

- EN: Declares or implements routines including `updateCurrentBranchInfo`, `assert`, `uint64_t`, `addSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updateCurrentBranchInfo`, `assert`, `uint64_t`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `updateCurrentBranchInfo`, `assert`, `uint64_t`, `addSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updateCurrentBranchInfo`, `assert`, `uint64_t`, `addSuccessor`。

### Lines 928-944

```cpp
        ExecCount += TotalCount;
      }
      NewBBs[I]->setExecutionCount(ExecCount);
    }

    if (MergeBlock) {
      // Arrange for the MergeBlock to be the fallthrough for the first
      // promoted call block.
      std::unique_ptr<BinaryBasicBlock> MBPtr;
      std::swap(MBPtr, NewBBs.back());
      NewBBs.pop_back();
      NewBBs.emplace(NewBBs.begin() + 1, std::move(MBPtr));
      // TODO: is COUNT_FALLTHROUGH_EDGE the right thing here?
      NewBBs.back()->addSuccessor(MergeBlock, TotalCount); // uncond branch
    }
  }
```

- EN: Declares or implements routines including `setExecutionCount`, `swap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setExecutionCount`, `swap`.
- CN: 这里声明或实现函数，例如 `setExecutionCount`, `swap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setExecutionCount`, `swap`。

### Lines 945-954

```cpp
  // Update the execution count.
  NewBBs.back()->setExecutionCount(TotalCount);

  // Update BB and BB layout.
  Function.insertBasicBlocks(&IndCallBlock, std::move(NewBBs));
  assert(Function.validateCFG());

  return MergeBlock;
}
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 955-963

```cpp
size_t IndirectCallPromotion::canPromoteCallsite(
    const BinaryBasicBlock &BB, const MCInst &Inst,
    const std::vector<Callsite> &Targets, uint64_t NumCalls) {
  BinaryFunction *BF = BB.getFunction();
  const BinaryContext &BC = BF->getBinaryContext();

  if (BB.getKnownExecutionCount() < opts::ExecutionCountThreshold)
    return 0;
```

- EN: Declares or implements routines including `getBinaryContext`. Notable symbols here include `getBinaryContext`.
- CN: 这里声明或实现函数，例如 `getBinaryContext`。这里较值得关注的符号包括 `getBinaryContext`。

### Lines 964-973

```cpp
  const bool IsJumpTable = BF->getJumpTable(Inst);

  auto computeStats = [&](size_t N) {
    for (size_t I = 0; I < N; ++I)
      if (IsJumpTable)
        TotalNumFrequentJmps += Targets[I].Branches;
      else
        TotalNumFrequentCalls += Targets[I].Branches;
  };
```

- EN: Declares or implements routines including `getJumpTable`. Notable symbols here include `getJumpTable`.
- CN: 这里声明或实现函数，例如 `getJumpTable`。这里较值得关注的符号包括 `getJumpTable`。

### Lines 974-984

```cpp
  // If we have no targets (or no calls), skip this callsite.
  if (Targets.empty() || !NumCalls) {
    if (opts::Verbosity >= 1) {
      const ptrdiff_t InstIdx = &Inst - &(*BB.begin());
      BC.outs() << "BOLT-INFO: ICP failed in " << *BF << " @ " << InstIdx
                << " in " << BB.getName() << ", calls = " << NumCalls
                << ", targets empty or NumCalls == 0.\n";
    }
    return 0;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 985-992

```cpp
  size_t TopN = opts::ICPTopN;
  if (IsJumpTable)
    TopN = opts::ICPJumpTablesTopN ? opts::ICPJumpTablesTopN : TopN;
  else
    TopN = opts::ICPCallsTopN ? opts::ICPCallsTopN : TopN;

  const size_t TrialN = TopN ? std::min(TopN, Targets.size()) : Targets.size();
```

- EN: Declares or implements routines including `min`. Notable symbols here include `min`.
- CN: 这里声明或实现函数，例如 `min`。这里较值得关注的符号包括 `min`。

### Lines 993-1010

```cpp
  if (opts::ICPTopCallsites && !BC.MIB->hasAnnotation(Inst, "DoICP"))
    return 0;

  // Pick the top N targets.
  uint64_t TotalMispredictsTopN = 0;
  size_t N = 0;

  if (opts::ICPUseMispredicts &&
      (!IsJumpTable || opts::ICPJumpTablesByTarget)) {
    // Count total number of mispredictions for (at most) the top N targets.
    // We may choose a smaller N (TrialN vs. N) if the frequency threshold
    // is exceeded by fewer targets.
    double Threshold = double(opts::ICPMispredictThreshold);
    for (size_t I = 0; I < TrialN && Threshold > 0; ++I, ++N) {
      Threshold -= (100.0 * Targets[I].Mispreds) / NumCalls;
      TotalMispredictsTopN += Targets[I].Mispreds;
    }
    computeStats(N);
```

- EN: Declares or implements routines including `double`, `computeStats`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `double`, `computeStats`.
- CN: 这里声明或实现函数，例如 `double`, `computeStats`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `double`, `computeStats`。

### Lines 1011-1028

```cpp

    // Compute the misprediction frequency of the top N call targets.  If this
    // frequency is greater than the threshold, we should try ICP on this
    // callsite.
    const double TopNFrequency = (100.0 * TotalMispredictsTopN) / NumCalls;
    if (TopNFrequency == 0 || TopNFrequency < opts::ICPMispredictThreshold) {
      if (opts::Verbosity >= 1) {
        const ptrdiff_t InstIdx = &Inst - &(*BB.begin());
        BC.outs() << "BOLT-INFO: ICP failed in " << *BF << " @ " << InstIdx
                  << " in " << BB.getName() << ", calls = " << NumCalls
                  << ", top N mis. frequency " << format("%.1f", TopNFrequency)
                  << "% < " << opts::ICPMispredictThreshold << "%\n";
      }
      return 0;
    }
  } else {
    size_t MaxTargets = 0;
```

- EN: Declares or implements routines including `format`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `format`。

### Lines 1029-1046

```cpp
    // Count total number of calls for (at most) the top N targets.
    // We may choose a smaller N (TrialN vs. N) if the frequency threshold
    // is exceeded by fewer targets.
    const unsigned TotalThreshold = IsJumpTable
                                        ? opts::ICPJTTotalPercentThreshold
                                        : opts::ICPCallsTotalPercentThreshold;
    const unsigned RemainingThreshold =
        IsJumpTable ? opts::ICPJTRemainingPercentThreshold
                    : opts::ICPCallsRemainingPercentThreshold;
    uint64_t NumRemainingCalls = NumCalls;
    for (size_t I = 0; I < TrialN; ++I, ++MaxTargets) {
      if (100 * Targets[I].Branches < NumCalls * TotalThreshold)
        break;
      if (100 * Targets[I].Branches < NumRemainingCalls * RemainingThreshold)
        break;
      if (N + (Targets[I].JTIndices.empty() ? 1 : Targets[I].JTIndices.size()) >
          TrialN)
        break;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1047-1061

```cpp
      TotalMispredictsTopN += Targets[I].Mispreds;
      NumRemainingCalls -= Targets[I].Branches;
      N += Targets[I].JTIndices.empty() ? 1 : Targets[I].JTIndices.size();
    }
    computeStats(MaxTargets);

    // Don't check misprediction frequency for jump tables -- we don't really
    // care as long as we are saving loads from the jump table.
    if (!IsJumpTable || opts::ICPJumpTablesByTarget) {
      // Compute the misprediction frequency of the top N call targets.  If
      // this frequency is less than the threshold, we should skip ICP at
      // this callsite.
      const double TopNMispredictFrequency =
          (100.0 * TotalMispredictsTopN) / NumCalls;
```

- EN: Declares or implements routines including `computeStats`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `computeStats`.
- CN: 这里声明或实现函数，例如 `computeStats`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `computeStats`。

### Lines 1062-1075

```cpp
      if (TopNMispredictFrequency < opts::ICPMispredictThreshold) {
        if (opts::Verbosity >= 1) {
          const ptrdiff_t InstIdx = &Inst - &(*BB.begin());
          BC.outs() << "BOLT-INFO: ICP failed in " << *BF << " @ " << InstIdx
                    << " in " << BB.getName() << ", calls = " << NumCalls
                    << ", top N mispredict frequency "
                    << format("%.1f", TopNMispredictFrequency) << "% < "
                    << opts::ICPMispredictThreshold << "%\n";
        }
        return 0;
      }
    }
  }
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1076-1089

```cpp
  // Filter by inline-ability of target functions, stop at first target that
  // can't be inlined.
  if (!IsJumpTable && opts::ICPPeelForInline) {
    for (size_t I = 0; I < N; ++I) {
      const MCSymbol *TargetSym = Targets[I].To.Sym;
      const BinaryFunction *TargetBF = BC.getFunctionForSymbol(TargetSym);
      if (!TargetBF || !BinaryFunctionPass::shouldOptimize(*TargetBF) ||
          getInliningInfo(*TargetBF).Type == InliningType::INL_NONE) {
        N = I;
        break;
      }
    }
  }
```

- EN: Declares or implements routines including `getInliningInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getInliningInfo`.
- CN: 这里声明或实现函数，例如 `getInliningInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getInliningInfo`。

### Lines 1090-1097

```cpp
  // Filter functions that can have ICP applied (for debugging)
  if (!opts::ICPFuncsList.empty()) {
    for (std::string &Name : opts::ICPFuncsList)
      if (BF->hasName(Name))
        return N;
    return 0;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1098-1109

```cpp
  return N;
}

void IndirectCallPromotion::printCallsiteInfo(
    const BinaryBasicBlock &BB, const MCInst &Inst,
    const std::vector<Callsite> &Targets, const size_t N,
    uint64_t NumCalls) const {
  BinaryContext &BC = BB.getFunction()->getBinaryContext();
  const bool IsTailCall = BC.MIB->isTailCall(Inst);
  const bool IsJumpTable = BB.getFunction()->getJumpTable(Inst);
  const ptrdiff_t InstIdx = &Inst - &(*BB.begin());
```

- EN: Declares or implements routines including `isTailCall`. Notable symbols here include `isTailCall`.
- CN: 这里声明或实现函数，例如 `isTailCall`。这里较值得关注的符号包括 `isTailCall`。

### Lines 1110-1127

```cpp
  BC.outs() << "BOLT-INFO: ICP candidate branch info: " << *BB.getFunction()
            << " @ " << InstIdx << " in " << BB.getName()
            << " -> calls = " << NumCalls
            << (IsTailCall ? " (tail)" : (IsJumpTable ? " (jump table)" : ""))
            << "\n";
  for (size_t I = 0; I < N; I++) {
    const double Frequency = 100.0 * Targets[I].Branches / NumCalls;
    const double MisFrequency = 100.0 * Targets[I].Mispreds / NumCalls;
    BC.outs() << "BOLT-INFO:   ";
    if (Targets[I].To.Sym)
      BC.outs() << Targets[I].To.Sym->getName();
    else
      BC.outs() << Targets[I].To.Addr;
    BC.outs() << ", calls = " << Targets[I].Branches
              << ", mispreds = " << Targets[I].Mispreds
              << ", taken freq = " << format("%.1f", Frequency) << "%"
              << ", mis. freq = " << format("%.1f", MisFrequency) << "%";
    bool First = true;
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1128-1140

```cpp
    for (uint64_t JTIndex : Targets[I].JTIndices) {
      BC.outs() << (First ? ", indices = " : ", ") << JTIndex;
      First = false;
    }
    BC.outs() << "\n";
  }

  LLVM_DEBUG({
    dbgs() << "BOLT-INFO: ICP original call instruction:";
    BC.printInstruction(dbgs(), Inst, Targets[0].From.Addr, nullptr, true);
  });
}
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 1141-1149

```cpp
Error IndirectCallPromotion::runOnFunctions(BinaryContext &BC) {
  if (opts::ICP == ICP_NONE)
    return Error::success();

  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is supported only on X86\n";
    exit(1);
  }
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`. Notable symbols here include `runOnFunctions`, `exit`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`。这里较值得关注的符号包括 `runOnFunctions`, `exit`。

### Lines 1150-1162

```cpp
  auto &BFs = BC.getBinaryFunctions();

  const bool OptimizeCalls = (opts::ICP == ICP_CALLS || opts::ICP == ICP_ALL);
  const bool OptimizeJumpTables =
      (opts::ICP == ICP_JUMP_TABLES || opts::ICP == ICP_ALL);

  std::unique_ptr<RegAnalysis> RA;
  std::unique_ptr<BinaryFunctionCallGraph> CG;
  if (OptimizeJumpTables) {
    CG.reset(new BinaryFunctionCallGraph(buildCallGraph(BC)));
    RA.reset(new RegAnalysis(BC, &BFs, &*CG));
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1163-1174

```cpp
  // If icp-top-callsites is enabled, compute the total number of indirect
  // calls and then optimize the hottest callsites that contribute to that
  // total.
  SetVector<BinaryFunction *> Functions;
  if (opts::ICPTopCallsites == 0) {
    for (auto &KV : BFs)
      Functions.insert(&KV.second);
  } else {
    using IndirectCallsite = std::tuple<uint64_t, MCInst *, BinaryFunction *>;
    std::vector<IndirectCallsite> IndirectCalls;
    size_t TotalIndirectCalls = 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1175-1183

```cpp
    // Find all the indirect callsites.
    for (auto &BFIt : BFs) {
      BinaryFunction &Function = BFIt.second;

      if (!shouldOptimize(Function))
        continue;

      const bool HasLayout = !Function.getLayout().block_empty();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1184-1194

```cpp
      for (BinaryBasicBlock &BB : Function) {
        if (HasLayout && Function.isSplit() && BB.isCold())
          continue;

        for (MCInst &Inst : BB) {
          const bool IsJumpTable = Function.getJumpTable(Inst);
          const bool HasIndirectCallProfile =
              BC.MIB->hasAnnotation(Inst, "CallProfile");
          const bool IsDirectCall =
              (BC.MIB->isCall(Inst) && BC.MIB->getTargetSymbol(Inst, 0));
```

- EN: Declares or implements routines including `hasAnnotation`. Notable symbols here include `hasAnnotation`.
- CN: 这里声明或实现函数，例如 `hasAnnotation`。这里较值得关注的符号包括 `hasAnnotation`。

### Lines 1195-1208

```cpp
          if (!IsDirectCall &&
              ((HasIndirectCallProfile && !IsJumpTable && OptimizeCalls) ||
               (IsJumpTable && OptimizeJumpTables))) {
            uint64_t NumCalls = 0;
            for (const Callsite &BInfo : getCallTargets(BB, Inst))
              NumCalls += BInfo.Branches;
            IndirectCalls.push_back(
                std::make_tuple(NumCalls, &Inst, &Function));
            TotalIndirectCalls += NumCalls;
          }
        }
      }
    }
```

- EN: Declares or implements routines including `make_tuple`. Notable symbols here include `make_tuple`.
- CN: 这里声明或实现函数，例如 `make_tuple`。这里较值得关注的符号包括 `make_tuple`。

### Lines 1209-1226

```cpp
    // Sort callsites by execution count.
    llvm::sort(reverse(IndirectCalls));

    // Find callsites that contribute to the top "opts::ICPTopCallsites"%
    // number of calls.
    const float TopPerc = opts::ICPTopCallsites / 100.0f;
    int64_t MaxCalls = TotalIndirectCalls * TopPerc;
    uint64_t LastFreq = std::numeric_limits<uint64_t>::max();
    size_t Num = 0;
    for (const IndirectCallsite &IC : IndirectCalls) {
      const uint64_t CurFreq = std::get<0>(IC);
      // Once we decide to stop, include at least all branches that share the
      // same frequency of the last one to avoid non-deterministic behavior
      // (e.g. turning on/off ICP depending on the order of functions)
      if (MaxCalls <= 0 && CurFreq != LastFreq)
        break;
      MaxCalls -= CurFreq;
      LastFreq = CurFreq;
```

- EN: Declares or implements routines including `sort`, `max`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`, `max`.
- CN: 这里声明或实现函数，例如 `sort`, `max`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`, `max`。

### Lines 1227-1235

```cpp
      BC.MIB->addAnnotation(*std::get<1>(IC), "DoICP", true);
      Functions.insert(std::get<2>(IC));
      ++Num;
    }
    BC.outs() << "BOLT-INFO: ICP Total indirect calls = " << TotalIndirectCalls
              << ", " << Num << " callsites cover " << opts::ICPTopCallsites
              << "% of all indirect calls\n";
  }
```

- EN: Declares or implements routines including `addAnnotation`. Notable symbols here include `addAnnotation`.
- CN: 这里声明或实现函数，例如 `addAnnotation`。这里较值得关注的符号包括 `addAnnotation`。

### Lines 1236-1243

```cpp
  for (BinaryFunction *FuncPtr : Functions) {
    BinaryFunction &Function = *FuncPtr;

    if (!shouldOptimize(Function))
      continue;

    const bool HasLayout = !Function.getLayout().block_empty();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1244-1257

```cpp
    // Total number of indirect calls issued from the current Function.
    // (a fraction of TotalIndirectCalls)
    uint64_t FuncTotalIndirectCalls = 0;
    uint64_t FuncTotalIndirectJmps = 0;

    std::vector<BinaryBasicBlock *> BBs;
    for (BinaryBasicBlock &BB : Function) {
      // Skip indirect calls in cold blocks.
      if (!HasLayout || !Function.isSplit() || !BB.isCold())
        BBs.push_back(&BB);
    }
    if (BBs.empty())
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1258-1270

```cpp
    DataflowInfoManager Info(Function, RA.get(), nullptr);
    while (!BBs.empty()) {
      BinaryBasicBlock *BB = BBs.back();
      BBs.pop_back();

      for (unsigned Idx = 0; Idx < BB->size(); ++Idx) {
        MCInst &Inst = BB->getInstructionAtIndex(Idx);
        const ptrdiff_t InstIdx = &Inst - &(*BB->begin());
        const bool IsTailCall = BC.MIB->isTailCall(Inst);
        const bool HasIndirectCallProfile =
            BC.MIB->hasAnnotation(Inst, "CallProfile");
        const bool IsJumpTable = Function.getJumpTable(Inst);
```

- EN: Declares or implements routines including `Info`, `getInstructionAtIndex`, `isTailCall`, `hasAnnotation`. Notable symbols here include `Info`, `getInstructionAtIndex`, `isTailCall`, `hasAnnotation`.
- CN: 这里声明或实现函数，例如 `Info`, `getInstructionAtIndex`, `isTailCall`, `hasAnnotation`。这里较值得关注的符号包括 `Info`, `getInstructionAtIndex`, `isTailCall`, `hasAnnotation`。

### Lines 1271-1279

```cpp
        if (BC.MIB->isCall(Inst))
          TotalCalls += BB->getKnownExecutionCount();

        if (IsJumpTable && !OptimizeJumpTables)
          continue;

        if (!IsJumpTable && (!HasIndirectCallProfile || !OptimizeCalls))
          continue;
```

- EN: Declares or implements routines including `getKnownExecutionCount`. Notable symbols here include `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`。这里较值得关注的符号包括 `getKnownExecutionCount`。

### Lines 1280-1291

```cpp
        // Ignore direct calls.
        if (BC.MIB->isCall(Inst) && BC.MIB->getTargetSymbol(Inst, 0))
          continue;

        assert((BC.MIB->isCall(Inst) || BC.MIB->isIndirectBranch(Inst)) &&
               "expected a call or an indirect jump instruction");

        if (IsJumpTable)
          ++TotalJumpTableCallsites;
        else
          ++TotalIndirectCallsites;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 1292-1302

```cpp
        std::vector<Callsite> Targets = getCallTargets(*BB, Inst);

        // Compute the total number of calls from this particular callsite.
        uint64_t NumCalls = 0;
        for (const Callsite &BInfo : Targets)
          NumCalls += BInfo.Branches;
        if (!IsJumpTable)
          FuncTotalIndirectCalls += NumCalls;
        else
          FuncTotalIndirectJmps += NumCalls;
```

- EN: Declares or implements routines including `getCallTargets`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCallTargets`.
- CN: 这里声明或实现函数，例如 `getCallTargets`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCallTargets`。

### Lines 1303-1318

```cpp
        // If FLAGS regs is alive after this jmp site, do not try
        // promoting because we will clobber FLAGS.
        if (IsJumpTable) {
          ErrorOr<const BitVector &> State =
              Info.getLivenessAnalysis().getStateBefore(Inst);
          if (!State || (State && (*State)[BC.MIB->getFlagsReg()])) {
            if (opts::Verbosity >= 1)
              BC.outs() << "BOLT-INFO: ICP failed in " << Function << " @ "
                        << InstIdx << " in " << BB->getName()
                        << ", calls = " << NumCalls
                        << (State ? ", cannot clobber flags reg.\n"
                                  : ", no liveness data available.\n");
            continue;
          }
        }
```

- EN: Declares or implements routines including `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getName`.
- CN: 这里声明或实现函数，例如 `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getName`。

### Lines 1319-1329

```cpp
        // Should this callsite be optimized?  Return the number of targets
        // to use when promoting this call.  A value of zero means to skip
        // this callsite.
        size_t N = canPromoteCallsite(*BB, Inst, Targets, NumCalls);

        // If it is a jump table and it failed to meet our initial threshold,
        // proceed to findCallTargetSymbols -- it may reevaluate N if
        // memory profile is present
        if (!N && !IsJumpTable)
          continue;
```

- EN: Declares or implements routines including `canPromoteCallsite`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `canPromoteCallsite`.
- CN: 这里声明或实现函数，例如 `canPromoteCallsite`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `canPromoteCallsite`。

### Lines 1330-1337

```cpp
        if (opts::Verbosity >= 1)
          printCallsiteInfo(*BB, Inst, Targets, N, NumCalls);

        // Find MCSymbols or absolute addresses for each call target.
        MCInst *TargetFetchInst = nullptr;
        const SymTargetsType SymTargets =
            findCallTargetSymbols(Targets, N, *BB, Inst, TargetFetchInst);
```

- EN: Declares or implements routines including `printCallsiteInfo`, `findCallTargetSymbols`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `printCallsiteInfo`, `findCallTargetSymbols`.
- CN: 这里声明或实现函数，例如 `printCallsiteInfo`, `findCallTargetSymbols`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `printCallsiteInfo`, `findCallTargetSymbols`。

### Lines 1338-1355

```cpp
        // findCallTargetSymbols may have changed N if mem profile is available
        // for jump tables
        if (!N)
          continue;

        LLVM_DEBUG(printDecision(dbgs(), Targets, N));

        // If we can't resolve any of the target symbols, punt on this callsite.
        // TODO: can this ever happen?
        if (SymTargets.size() < N) {
          const size_t LastTarget = SymTargets.size();
          if (opts::Verbosity >= 1)
            BC.outs() << "BOLT-INFO: ICP failed in " << Function << " @ "
                      << InstIdx << " in " << BB->getName()
                      << ", calls = " << NumCalls
                      << ", ICP failed to find target symbol for "
                      << Targets[LastTarget].To.Sym->getName() << "\n";
          continue;
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 1356-1371

```cpp
        }

        MethodInfoType MethodInfo;

        if (!IsJumpTable) {
          MethodInfo = maybeGetVtableSyms(*BB, Inst, SymTargets);
          TotalMethodLoadsEliminated += MethodInfo.first.empty() ? 0 : 1;
          LLVM_DEBUG(dbgs()
                     << "BOLT-INFO: ICP "
                     << (!MethodInfo.first.empty() ? "found" : "did not find")
                     << " vtables for all methods.\n");
        } else if (TargetFetchInst) {
          ++TotalIndexBasedJumps;
          MethodInfo.second.push_back(TargetFetchInst);
        }
```

- EN: Declares or implements routines including `maybeGetVtableSyms`, `LLVM_DEBUG`, `if`. Notable symbols here include `maybeGetVtableSyms`, `LLVM_DEBUG`, `if`.
- CN: 这里声明或实现函数，例如 `maybeGetVtableSyms`, `LLVM_DEBUG`, `if`。这里较值得关注的符号包括 `maybeGetVtableSyms`, `LLVM_DEBUG`, `if`。

### Lines 1372-1380

```cpp
        // Generate new promoted call code for this callsite.
        MCPlusBuilder::BlocksVectorTy ICPcode =
            (IsJumpTable && !opts::ICPJumpTablesByTarget)
                ? BC.MIB->jumpTablePromotion(Inst, SymTargets,
                                             MethodInfo.second, BC.Ctx.get())
                : BC.MIB->indirectCallPromotion(
                      Inst, SymTargets, MethodInfo.first, MethodInfo.second,
                      opts::ICPOldCodeSequence, BC.Ctx.get());
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1381-1389

```cpp
        if (ICPcode.empty()) {
          if (opts::Verbosity >= 1)
            BC.outs() << "BOLT-INFO: ICP failed in " << Function << " @ "
                      << InstIdx << " in " << BB->getName()
                      << ", calls = " << NumCalls
                      << ", unable to generate promoted call code.\n";
          continue;
        }
```

- EN: Declares or implements routines including `getName`. Notable symbols here include `getName`.
- CN: 这里声明或实现函数，例如 `getName`。这里较值得关注的符号包括 `getName`。

### Lines 1390-1403

```cpp
        LLVM_DEBUG({
          uint64_t Offset = Targets[0].From.Addr;
          dbgs() << "BOLT-INFO: ICP indirect call code:\n";
          for (const auto &entry : ICPcode) {
            const MCSymbol *const &Sym = entry.first;
            const InstructionListType &Insts = entry.second;
            if (Sym)
              dbgs() << Sym->getName() << ":\n";
            Offset = BC.printInstructions(dbgs(), Insts.begin(), Insts.end(),
                                          Offset);
          }
          dbgs() << "---------------------------------------------------\n";
        });
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 1404-1411

```cpp
        // Rewrite the CFG with the newly generated ICP code.
        std::vector<std::unique_ptr<BinaryBasicBlock>> NewBBs =
            rewriteCall(*BB, Inst, std::move(ICPcode), MethodInfo.second);

        // Fix the CFG after inserting the new basic blocks.
        BinaryBasicBlock *MergeBlock =
            fixCFG(*BB, IsTailCall, IsJumpTable, std::move(NewBBs), Targets);
```

- EN: Declares or implements routines including `rewriteCall`, `fixCFG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `rewriteCall`, `fixCFG`.
- CN: 这里声明或实现函数，例如 `rewriteCall`, `fixCFG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `rewriteCall`, `fixCFG`。

### Lines 1412-1422

```cpp
        // Since the tail of the original block was split off and it may contain
        // additional indirect calls, we must add the merge block to the set of
        // blocks to process.
        if (MergeBlock)
          BBs.push_back(MergeBlock);

        if (opts::Verbosity >= 1)
          BC.outs() << "BOLT-INFO: ICP succeeded in " << Function << " @ "
                    << InstIdx << " in " << BB->getName()
                    << " -> calls = " << NumCalls << "\n";
```

- EN: Declares or implements routines including `getName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getName`.
- CN: 这里声明或实现函数，例如 `getName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getName`。

### Lines 1423-1434

```cpp
        if (IsJumpTable)
          ++TotalOptimizedJumpTableCallsites;
        else
          ++TotalOptimizedIndirectCallsites;

        Modified.insert(&Function);
      }
    }
    TotalIndirectCalls += FuncTotalIndirectCalls;
    TotalIndirectJmps += FuncTotalIndirectJmps;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1435-1452

```cpp
  BC.outs()
      << "BOLT-INFO: ICP total indirect callsites with profile = "
      << TotalIndirectCallsites << "\n"
      << "BOLT-INFO: ICP total jump table callsites = "
      << TotalJumpTableCallsites << "\n"
      << "BOLT-INFO: ICP total number of calls = " << TotalCalls << "\n"
      << "BOLT-INFO: ICP percentage of calls that are indirect = "
      << format("%.1f", (100.0 * TotalIndirectCalls) / TotalCalls) << "%\n"
      << "BOLT-INFO: ICP percentage of indirect calls that can be "
         "optimized = "
      << format("%.1f", (100.0 * TotalNumFrequentCalls) /
                            std::max<size_t>(TotalIndirectCalls, 1))
      << "%\n"
      << "BOLT-INFO: ICP percentage of indirect callsites that are "
         "optimized = "
      << format("%.1f", (100.0 * TotalOptimizedIndirectCallsites) /
                            std::max<uint64_t>(TotalIndirectCallsites, 1))
      << "%\n"
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1453-1470

```cpp
      << "BOLT-INFO: ICP number of method load elimination candidates = "
      << TotalMethodLoadEliminationCandidates << "\n"
      << "BOLT-INFO: ICP percentage of method calls candidates that have "
         "loads eliminated = "
      << format("%.1f",
                (100.0 * TotalMethodLoadsEliminated) /
                    std::max<uint64_t>(TotalMethodLoadEliminationCandidates, 1))
      << "%\n"
      << "BOLT-INFO: ICP percentage of indirect branches that are "
         "optimized = "
      << format("%.1f", (100.0 * TotalNumFrequentJmps) /
                            std::max<uint64_t>(TotalIndirectJmps, 1))
      << "%\n"
      << "BOLT-INFO: ICP percentage of jump table callsites that are "
      << "optimized = "
      << format("%.1f", (100.0 * TotalOptimizedJumpTableCallsites) /
                            std::max<uint64_t>(TotalJumpTableCallsites, 1))
      << "%\n"
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1471-1478

```cpp
      << "BOLT-INFO: ICP number of jump table callsites that can use hot "
      << "indices = " << TotalIndexBasedCandidates << "\n"
      << "BOLT-INFO: ICP percentage of jump table callsites that use hot "
         "indices = "
      << format("%.1f", (100.0 * TotalIndexBasedJumps) /
                            std::max<uint64_t>(TotalIndexBasedCandidates, 1))
      << "%\n";
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1479-1486

```cpp
#ifndef NDEBUG
  verifyProfile(BFs);
#endif
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `verifyProfile`. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `verifyProfile`。这里的注释记录了设计假设、不变量或使用说明。

## Key Concepts / 关键概念

- `init`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `aliasopt`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `value_desc`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `DEBUG_VERBOSE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/IndirectCallPromotion.h`, `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Passes/DataflowInfoManager.h`, `bolt/Passes/Inliner.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/STLExtras.h`, `llvm/Support/CommandLine.h`
- System headers / 系统头文件: `iterator`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
