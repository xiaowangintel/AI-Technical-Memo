# BinaryPasses.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/BinaryPasses.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Binary-level passes. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Binary-level passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/BinaryPasses.cpp - Binary-level passes -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements multiple passes for binary optimization and analysis.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-24

```cpp
#include "bolt/Passes/BinaryPasses.h"
#include "bolt/Core/FunctionLayout.h"
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/Passes/ReorderAlgorithm.h"
#include "bolt/Passes/ReorderFunctions.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/Support/CommandLine.h"
#include <atomic>
#include <mutex>
#include <numeric>
#include <vector>
```

- EN: Pulls in 11 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 25-33

```cpp
#define DEBUG_TYPE "bolt-opts"

using namespace llvm;
using namespace bolt;

static const char *dynoStatsOptName(const bolt::DynoStats::Category C) {
  assert(C > bolt::DynoStats::FIRST_DYNO_STAT &&
         C < DynoStats::LAST_DYNO_STAT && "Unexpected dyno stat category.");
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `dynoStatsOptName`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `dynoStatsOptName`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 34-42

```cpp
  static std::string OptNames[bolt::DynoStats::LAST_DYNO_STAT + 1];

  OptNames[C] = bolt::DynoStats::Description(C);

  llvm::replace(OptNames[C], ' ', '-');

  return OptNames[C].c_str();
}
```

- EN: Declares or implements routines including `Description`, `replace`. Notable symbols here include `Description`, `replace`.
- CN: 这里声明或实现函数，例如 `Description`, `replace`。这里较值得关注的符号包括 `Description`, `replace`。

### Lines 43-53

```cpp
namespace opts {

extern cl::OptionCategory BoltCategory;
extern cl::OptionCategory BoltOptCategory;

extern cl::opt<unsigned> Verbosity;
extern cl::opt<bool> EnableBAT;
extern cl::opt<unsigned> ExecutionCountThreshold;
extern cl::opt<bool> UpdateDebugSections;
extern cl::opt<bolt::ReorderFunctions::ReorderType> ReorderFunctions;
```

- EN: Works inside namespace scope `opts` to organize symbols. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`。

### Lines 54-69

```cpp
enum DynoStatsSortOrder : char {
  Ascending,
  Descending
};

static cl::opt<DynoStatsSortOrder> DynoStatsSortOrderOpt(
    "print-sorted-by-order",
    cl::desc("use ascending or descending order when printing functions "
             "ordered by dyno stats"),
    cl::init(DynoStatsSortOrder::Descending),
    cl::values(clEnumValN(DynoStatsSortOrder::Ascending, "ascending",
                          "Ascending order"),
               clEnumValN(DynoStatsSortOrder::Descending, "descending",
                          "Descending order")),
    cl::cat(BoltOptCategory));
```

- EN: Defines enumerations such as `DynoStatsSortOrder` to encode states or modes. Declares or implements routines including `init`, `cat`. Notable symbols here include `DynoStatsSortOrder`, `init`, `cat`.
- CN: 这里定义枚举 `DynoStatsSortOrder`，用于表达状态或模式。这里声明或实现函数，例如 `init`, `cat`。这里较值得关注的符号包括 `DynoStatsSortOrder`, `init`, `cat`。

### Lines 70-79

```cpp
cl::list<std::string>
HotTextMoveSections("hot-text-move-sections",
  cl::desc("list of sections containing functions used for hugifying hot text. "
           "BOLT makes sure these functions are not placed on the same page as "
           "the hot text. (default=\'.stub,.mover\')."),
  cl::value_desc("sec1,sec2,sec3,..."),
  cl::CommaSeparated,
  cl::ZeroOrMore,
  cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `value_desc`, `cat`. Notable symbols here include `value_desc`, `cat`.
- CN: 这里声明或实现函数，例如 `value_desc`, `cat`。这里较值得关注的符号包括 `value_desc`, `cat`。

### Lines 80-89

```cpp
bool isHotTextMover(const BinaryFunction &Function) {
  for (std::string &SectionName : opts::HotTextMoveSections) {
    if (Function.getOriginSectionName() &&
        *Function.getOriginSectionName() == SectionName)
      return true;
  }

  return false;
}
```

- EN: Declares or implements routines including `isHotTextMover`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isHotTextMover`.
- CN: 这里声明或实现函数，例如 `isHotTextMover`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isHotTextMover`。

### Lines 90-107

```cpp
static cl::opt<bool> MinBranchClusters(
    "min-branch-clusters",
    cl::desc("use a modified clustering algorithm geared towards minimizing "
             "branches"),
    cl::Hidden, cl::cat(BoltOptCategory));

static cl::list<Peepholes::PeepholeOpts> Peepholes(
    "peepholes", cl::CommaSeparated, cl::desc("enable peephole optimizations"),
    cl::value_desc("opt1,opt2,opt3,..."),
    cl::values(clEnumValN(Peepholes::PEEP_NONE, "none", "disable peepholes"),
               clEnumValN(Peepholes::PEEP_DOUBLE_JUMPS, "double-jumps",
                          "remove double jumps when able"),
               clEnumValN(Peepholes::PEEP_TAILCALL_TRAPS, "tailcall-traps",
                          "insert tail call traps"),
               clEnumValN(Peepholes::PEEP_USELESS_BRANCHES, "useless-branches",
                          "remove useless conditional branches"),
               clEnumValN(Peepholes::PEEP_ALL, "all",
                          "enable all peephole optimizations")),
```

- EN: Declares or implements routines including `cat`, `desc`, `value_desc`, `values`. Notable symbols here include `cat`, `desc`, `value_desc`, `values`.
- CN: 这里声明或实现函数，例如 `cat`, `desc`, `value_desc`, `values`。这里较值得关注的符号包括 `cat`, `desc`, `value_desc`, `values`。

### Lines 108-120

```cpp
    cl::ZeroOrMore, cl::cat(BoltOptCategory));

static cl::opt<unsigned>
    PrintFuncStat("print-function-statistics",
                  cl::desc("print statistics about basic block ordering"),
                  cl::init(0), cl::cat(BoltOptCategory));

static cl::opt<bool> PrintLargeFunctions(
    "print-large-functions",
    cl::desc("print functions that could not be overwritten due to excessive "
             "size"),
    cl::init(false), cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`, `desc`, `init`. Notable symbols here include `cat`, `desc`, `init`.
- CN: 这里声明或实现函数，例如 `cat`, `desc`, `init`。这里较值得关注的符号包括 `cat`, `desc`, `init`。

### Lines 121-134

```cpp
static cl::list<bolt::DynoStats::Category>
    PrintSortedBy("print-sorted-by", cl::CommaSeparated,
                  cl::desc("print functions sorted by order of dyno stats"),
                  cl::value_desc("key1,key2,key3,..."),
                  cl::values(
#define D(name, description, ...)                                              \
  clEnumValN(bolt::DynoStats::name, dynoStatsOptName(bolt::DynoStats::name),   \
             description),
                      REAL_DYNO_STATS
#undef D
                          clEnumValN(bolt::DynoStats::LAST_DYNO_STAT, "all",
                                     "sorted by all names")),
                  cl::ZeroOrMore, cl::cat(BoltOptCategory));
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Declares or implements routines including `desc`, `value_desc`, `clEnumValN`, `cat`. Defines macros such as `D` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里声明或实现函数，例如 `desc`, `value_desc`, `clEnumValN`, `cat`。这里定义宏 `D`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 135-144

```cpp
static cl::opt<bool>
    PrintUnknown("print-unknown",
                 cl::desc("print names of functions with unknown control flow"),
                 cl::cat(BoltCategory), cl::Hidden);

static cl::opt<bool>
    PrintUnknownCFG("print-unknown-cfg",
                    cl::desc("dump CFG of functions with unknown control flow"),
                    cl::cat(BoltCategory), cl::ReallyHidden);
```

- EN: Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`。

### Lines 145-162

```cpp
// Please MSVC19 with a forward declaration: otherwise it reports an error about
// an undeclared variable inside a callback.
extern cl::opt<bolt::ReorderBasicBlocks::LayoutType> ReorderBlocks;
cl::opt<bolt::ReorderBasicBlocks::LayoutType> ReorderBlocks(
    "reorder-blocks", cl::desc("change layout of basic blocks in a function"),
    cl::init(bolt::ReorderBasicBlocks::LT_NONE),
    cl::values(
        clEnumValN(bolt::ReorderBasicBlocks::LT_NONE, "none",
                   "do not reorder basic blocks"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_REVERSE, "reverse",
                   "layout blocks in reverse order"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE, "normal",
                   "perform optimal layout based on profile"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE_BRANCH,
                   "branch-predictor",
                   "perform optimal layout prioritizing branch "
                   "predictions"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE_CACHE, "cache",
```

- EN: Declares or implements routines including `desc`, `init`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `desc`, `init`。

### Lines 163-179

```cpp
                   "perform optimal layout prioritizing I-cache "
                   "behavior"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE_CACHE_PLUS, "cache+",
                   "perform layout optimizing I-cache behavior"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE_EXT_TSP, "ext-tsp",
                   "perform layout optimizing I-cache behavior"),
        clEnumValN(bolt::ReorderBasicBlocks::LT_OPTIMIZE_SHUFFLE,
                   "cluster-shuffle", "perform random layout of clusters")),
    cl::ZeroOrMore, cl::cat(BoltOptCategory),
    cl::callback([](const bolt::ReorderBasicBlocks::LayoutType &option) {
      if (option == bolt::ReorderBasicBlocks::LT_OPTIMIZE_CACHE_PLUS) {
        errs() << "BOLT-WARNING: '-reorder-blocks=cache+' is deprecated, please"
               << " use '-reorder-blocks=ext-tsp' instead\n";
        ReorderBlocks = bolt::ReorderBasicBlocks::LT_OPTIMIZE_EXT_TSP;
      }
    }));
```

- EN: Declares or implements routines including `cat`, `callback`, `errs`. Notable symbols here include `cat`, `callback`, `errs`.
- CN: 这里声明或实现函数，例如 `cat`, `callback`, `errs`。这里较值得关注的符号包括 `cat`, `callback`, `errs`。

### Lines 180-189

```cpp
static cl::opt<unsigned> ReportBadLayout(
    "report-bad-layout",
    cl::desc("print top <uint> functions with suboptimal code layout on input"),
    cl::init(0), cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<bool>
    ReportStaleFuncs("report-stale",
                     cl::desc("print the list of functions with stale profile"),
                     cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 190-207

```cpp
enum SctcModes : char {
  SctcAlways,
  SctcPreserveDirection,
  SctcHeuristic
};

static cl::opt<SctcModes>
SctcMode("sctc-mode",
  cl::desc("mode for simplify conditional tail calls"),
  cl::init(SctcAlways),
  cl::values(clEnumValN(SctcAlways, "always", "always perform sctc"),
    clEnumValN(SctcPreserveDirection,
      "preserve",
      "only perform sctc when branch direction is "
      "preserved"),
    clEnumValN(SctcHeuristic,
      "heuristic",
      "use branch prediction data to control sctc")),
```

- EN: Defines enumerations such as `SctcModes` to encode states or modes. Declares or implements routines including `desc`, `init`, `values`. Notable symbols here include `SctcModes`, `desc`, `init`, `values`.
- CN: 这里定义枚举 `SctcModes`，用于表达状态或模式。这里声明或实现函数，例如 `desc`, `init`, `values`。这里较值得关注的符号包括 `SctcModes`, `desc`, `init`, `values`。

### Lines 208-218

```cpp
  cl::ZeroOrMore,
  cl::cat(BoltOptCategory));

static cl::opt<unsigned>
StaleThreshold("stale-threshold",
    cl::desc(
      "maximum percentage of stale functions to tolerate (default: 100)"),
    cl::init(100),
    cl::Hidden,
    cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`, `tolerate`, `init`. Notable symbols here include `cat`, `tolerate`, `init`.
- CN: 这里声明或实现函数，例如 `cat`, `tolerate`, `init`。这里较值得关注的符号包括 `cat`, `tolerate`, `init`。

### Lines 219-231

```cpp
static cl::opt<unsigned> TSPThreshold(
    "tsp-threshold",
    cl::desc(
        "maximum number of hot basic blocks in a function for which to use "
        "a precise TSP solution while re-ordering basic blocks"),
    cl::init(10), cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<unsigned> TopCalledLimit(
    "top-called-limit",
    cl::desc("maximum number of functions to print in top called "
             "functions section"),
    cl::init(100), cl::Hidden, cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 232-243

```cpp
// Profile density options, synced with llvm-profgen/ProfileGenerator.cpp
static cl::opt<int> ProfileDensityCutOffHot(
    "profile-density-cutoff-hot", cl::init(990000),
    cl::desc("Total samples cutoff for functions used to calculate "
             "profile density."));

static cl::opt<double> ProfileDensityThreshold(
    "profile-density-threshold", cl::init(60),
    cl::desc("If the profile density is below the given threshold, it "
             "will be suggested to increase the sampling rate."),
    cl::Optional);
```

- EN: Declares or implements routines including `init`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `init`。

### Lines 244-253

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

bool BinaryFunctionPass::shouldOptimize(const BinaryFunction &BF) const {
  return BF.isSimple() && BF.getState() == BinaryFunction::State::CFG &&
         !BF.isIgnored();
}
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `shouldOptimize`. Notable symbols here include `shouldOptimize`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `shouldOptimize`。这里较值得关注的符号包括 `shouldOptimize`, `opts`, `llvm`, `bolt`。

### Lines 254-265

```cpp
bool BinaryFunctionPass::shouldPrint(const BinaryFunction &BF) const {
  return BF.isSimple() && !BF.isIgnored();
}

void NormalizeCFG::runOnFunction(BinaryFunction &BF) {
  uint64_t NumRemoved = 0;
  uint64_t NumDuplicateEdges = 0;
  uint64_t NeedsFixBranches = 0;
  for (BinaryBasicBlock &BB : BF) {
    if (!BB.empty())
      continue;
```

- EN: Declares or implements routines including `shouldPrint`, `runOnFunction`. Notable symbols here include `shouldPrint`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `shouldPrint`, `runOnFunction`。这里较值得关注的符号包括 `shouldPrint`, `runOnFunction`。

### Lines 266-280

```cpp
    if (BB.isEntryPoint() || BB.isLandingPad())
      continue;

    // Handle a dangling empty block.
    if (BB.succ_size() == 0) {
      // If an empty dangling basic block has a predecessor, it could be a
      // result of codegen for __builtin_unreachable. In such case, do not
      // remove the block.
      if (BB.pred_size() == 0) {
        BB.markValid(false);
        ++NumRemoved;
      }
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 281-290

```cpp
    // The block should have just one successor.
    BinaryBasicBlock *Successor = BB.getSuccessor();
    assert(Successor && "invalid CFG encountered");

    // Redirect all predecessors to the successor block.
    while (!BB.pred_empty()) {
      BinaryBasicBlock *Predecessor = *BB.pred_begin();
      if (Predecessor->hasJumpTable())
        break;
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 291-301

```cpp
      if (Predecessor == Successor)
        break;

      BinaryBasicBlock::BinaryBranchInfo &BI = Predecessor->getBranchInfo(BB);
      Predecessor->replaceSuccessor(&BB, Successor, BI.Count,
                                    BI.MispredictedCount);
      // We need to fix branches even if we failed to replace all successors
      // and remove the block.
      NeedsFixBranches = true;
    }
```

- EN: Declares or implements routines including `getBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBranchInfo`.
- CN: 这里声明或实现函数，例如 `getBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBranchInfo`。

### Lines 302-311

```cpp
    if (BB.pred_empty()) {
      BB.removeAllSuccessors();
      BB.markValid(false);
      ++NumRemoved;
    }
  }

  if (NumRemoved)
    BF.eraseInvalidBBs();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 312-322

```cpp
  // Check for duplicate successors. Do it after the empty block elimination as
  // we can get more duplicate successors.
  for (BinaryBasicBlock &BB : BF)
    if (!BB.hasJumpTable() && BB.succ_size() == 2 &&
        BB.getConditionalSuccessor(false) == BB.getConditionalSuccessor(true))
      ++NumDuplicateEdges;

  // fixBranches() will get rid of duplicate edges and update jump instructions.
  if (NumDuplicateEdges || NeedsFixBranches)
    BF.fixBranches();
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 323-340

```cpp
  NumDuplicateEdgesMerged += NumDuplicateEdges;
  NumBlocksRemoved += NumRemoved;
}

Error NormalizeCFG::runOnFunctions(BinaryContext &BC) {
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_BB_LINEAR,
      [&](BinaryFunction &BF) { runOnFunction(BF); },
      [&](const BinaryFunction &BF) { return !shouldOptimize(BF); },
      "NormalizeCFG");
  if (NumBlocksRemoved)
    BC.outs() << "BOLT-INFO: removed " << NumBlocksRemoved << " empty block"
              << (NumBlocksRemoved == 1 ? "" : "s") << '\n';
  if (NumDuplicateEdgesMerged)
    BC.outs() << "BOLT-INFO: merged " << NumDuplicateEdgesMerged
              << " duplicate CFG edge"
              << (NumDuplicateEdgesMerged == 1 ? "" : "s") << '\n';
  return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 341-358

```cpp
}

void EliminateUnreachableBlocks::runOnFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  unsigned Count;
  uint64_t Bytes;
  Function.markUnreachableBlocks();
  LLVM_DEBUG({
    bool HasInvalidBB = false;
    for (BinaryBasicBlock &BB : Function) {
      if (!BB.isValid()) {
        HasInvalidBB = true;
        dbgs() << "BOLT-INFO: UCE found unreachable block " << BB.getName()
               << " in function " << Function << "\n";
      }
    }
    if (HasInvalidBB)
      Function.dump();
```

- EN: Declares or implements routines including `runOnFunction`, `dbgs`. Notable symbols here include `runOnFunction`, `dbgs`.
- CN: 这里声明或实现函数，例如 `runOnFunction`, `dbgs`。这里较值得关注的符号包括 `runOnFunction`, `dbgs`。

### Lines 359-374

```cpp
  });
  BinaryContext::IndependentCodeEmitter Emitter =
      BC.createIndependentMCCodeEmitter();
  std::tie(Count, Bytes) = Function.eraseInvalidBBs(Emitter.MCE.get());
  DeletedBlocks += Count;
  DeletedBytes += Bytes;
  if (Count) {
    auto L = BC.scopeLock();
    Modified.insert(&Function);
    if (opts::Verbosity > 0)
      BC.outs() << "BOLT-INFO: removed " << Count
                << " dead basic block(s) accounting for " << Bytes
                << " bytes in function " << Function << '\n';
  }
}
```

- EN: Declares or implements routines including `tie`, `block`. Notable symbols here include `tie`, `block`.
- CN: 这里声明或实现函数，例如 `tie`, `block`。这里较值得关注的符号包括 `tie`, `block`。

### Lines 375-383

```cpp
Error EliminateUnreachableBlocks::runOnFunctions(BinaryContext &BC) {
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    runOnFunction(BF);
  };

  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    return !shouldOptimize(BF) || BF.getLayout().block_empty();
  };
```

- EN: Declares or implements routines including `runOnFunctions`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `runOnFunction`。

### Lines 384-393

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_CONSTANT, WorkFun,
      SkipPredicate, "elimininate-unreachable");

  if (DeletedBlocks)
    BC.outs() << "BOLT-INFO: UCE removed " << DeletedBlocks << " blocks and "
              << DeletedBytes << " bytes of code\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 394-403

```cpp
bool ReorderBasicBlocks::shouldPrint(const BinaryFunction &BF) const {
  return (BinaryFunctionPass::shouldPrint(BF) &&
          opts::ReorderBlocks != ReorderBasicBlocks::LT_NONE);
}

bool ReorderBasicBlocks::shouldOptimize(const BinaryFunction &BF) const {
  // Apply execution count threshold
  if (BF.getKnownExecutionCount() < opts::ExecutionCountThreshold)
    return false;
```

- EN: Declares or implements routines including `shouldPrint`, `shouldOptimize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `shouldPrint`, `shouldOptimize`.
- CN: 这里声明或实现函数，例如 `shouldPrint`, `shouldOptimize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `shouldPrint`, `shouldOptimize`。

### Lines 404-414

```cpp
  return BinaryFunctionPass::shouldOptimize(BF);
}

Error ReorderBasicBlocks::runOnFunctions(BinaryContext &BC) {
  if (opts::ReorderBlocks == ReorderBasicBlocks::LT_NONE)
    return Error::success();

  std::atomic_uint64_t ModifiedFuncCount(0);
  std::mutex FunctionEditDistanceMutex;
  DenseMap<const BinaryFunction *, uint64_t> FunctionEditDistance;
```

- EN: Declares or implements routines including `runOnFunctions`, `ModifiedFuncCount`. Notable symbols here include `runOnFunctions`, `ModifiedFuncCount`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `ModifiedFuncCount`。这里较值得关注的符号包括 `runOnFunctions`, `ModifiedFuncCount`。

### Lines 415-431

```cpp
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    SmallVector<const BinaryBasicBlock *, 0> OldBlockOrder;
    if (opts::PrintFuncStat > 0)
      llvm::copy(BF.getLayout().blocks(), std::back_inserter(OldBlockOrder));

    const bool LayoutChanged =
        modifyFunctionLayout(BF, opts::ReorderBlocks, opts::MinBranchClusters);
    if (LayoutChanged) {
      ModifiedFuncCount.fetch_add(1, std::memory_order_relaxed);
      if (opts::PrintFuncStat > 0) {
        const uint64_t Distance = BF.getLayout().getEditDistance(OldBlockOrder);
        std::lock_guard<std::mutex> Lock(FunctionEditDistanceMutex);
        FunctionEditDistance[&BF] = Distance;
      }
    }
  };
```

- EN: Declares or implements routines including `copy`, `modifyFunctionLayout`, `Lock`. Notable symbols here include `copy`, `modifyFunctionLayout`, `Lock`.
- CN: 这里声明或实现函数，例如 `copy`, `modifyFunctionLayout`, `Lock`。这里较值得关注的符号包括 `copy`, `modifyFunctionLayout`, `Lock`。

### Lines 432-441

```cpp
  ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
    return !shouldOptimize(BF);
  };

  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_BB_LINEAR, WorkFun, SkipFunc,
      "ReorderBasicBlocks");
  const size_t NumAllProfiledFunctions =
      BC.NumProfiledFuncs + BC.NumStaleProfileFuncs;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 442-450

```cpp
  BC.outs() << "BOLT-INFO: basic block reordering modified layout of "
            << format(
                   "%zu functions (%.2lf%% of profiled, %.2lf%% of total)\n",
                   ModifiedFuncCount.load(std::memory_order_relaxed),
                   100.0 * ModifiedFuncCount.load(std::memory_order_relaxed) /
                       NumAllProfiledFunctions,
                   100.0 * ModifiedFuncCount.load(std::memory_order_relaxed) /
                       BC.getBinaryFunctions().size());
```

- EN: Declares or implements routines including `functions`. Notable symbols here include `functions`.
- CN: 这里声明或实现函数，例如 `functions`。这里较值得关注的符号包括 `functions`。

### Lines 451-459

```cpp
  if (opts::PrintFuncStat > 0) {
    raw_ostream &OS = BC.outs();
    // Copy all the values into vector in order to sort them
    std::map<uint64_t, BinaryFunction &> ScoreMap;
    auto &BFs = BC.getBinaryFunctions();
    for (auto It = BFs.begin(); It != BFs.end(); ++It)
      ScoreMap.insert(std::pair<uint64_t, BinaryFunction &>(
          It->second.getFunctionScore(), It->second));
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 460-472

```cpp
    OS << "\nBOLT-INFO: Printing Function Statistics:\n\n";
    OS << "           There are " << BFs.size() << " functions in total. \n";
    OS << "           Number of functions being modified: "
       << ModifiedFuncCount.load(std::memory_order_relaxed) << "\n";
    OS << "           User asks for detailed information on top "
       << opts::PrintFuncStat << " functions. (Ranked by function score)"
       << "\n\n";
    uint64_t I = 0;
    for (std::map<uint64_t, BinaryFunction &>::reverse_iterator Rit =
             ScoreMap.rbegin();
         Rit != ScoreMap.rend() && I < opts::PrintFuncStat; ++Rit, ++I) {
      BinaryFunction &Function = Rit->second;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 473-486

```cpp
      OS << "           Information for function of top: " << (I + 1) << ": \n";
      OS << "             Function Score is: " << Function.getFunctionScore()
         << "\n";
      OS << "             There are " << Function.size()
         << " number of blocks in this function.\n";
      OS << "             There are " << Function.getInstructionCount()
         << " number of instructions in this function.\n";
      OS << "             The edit distance for this function is: "
         << FunctionEditDistance.lookup(&Function) << "\n\n";
    }
  }
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 487-495

```cpp
bool ReorderBasicBlocks::modifyFunctionLayout(BinaryFunction &BF,
                                              LayoutType Type,
                                              bool MinBranchClusters) const {
  if (BF.size() == 0 || Type == LT_NONE)
    return false;

  BinaryFunction::BasicBlockOrderType NewLayout;
  std::unique_ptr<ReorderAlgorithm> Algo;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 496-508

```cpp
  // Cannot do optimal layout without profile.
  if (Type != LT_REVERSE && !BF.hasValidProfile())
    return false;

  if (Type == LT_REVERSE) {
    Algo.reset(new ReverseReorderAlgorithm());
  } else if (BF.size() <= opts::TSPThreshold && Type != LT_OPTIMIZE_SHUFFLE) {
    // Work on optimal solution if problem is small enough
    LLVM_DEBUG(dbgs() << "finding optimal block layout for " << BF << "\n");
    Algo.reset(new TSPReorderAlgorithm());
  } else {
    LLVM_DEBUG(dbgs() << "running block layout heuristics on " << BF << "\n");
```

- EN: Declares or implements routines including `if`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `if`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `if`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `if`, `LLVM_DEBUG`。

### Lines 509-519

```cpp
    std::unique_ptr<ClusterAlgorithm> CAlgo;
    if (MinBranchClusters)
      CAlgo.reset(new MinBranchGreedyClusterAlgorithm());
    else
      CAlgo.reset(new PHGreedyClusterAlgorithm());

    switch (Type) {
    case LT_OPTIMIZE:
      Algo.reset(new OptimizeReorderAlgorithm(std::move(CAlgo)));
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 520-527

```cpp
    case LT_OPTIMIZE_BRANCH:
      Algo.reset(new OptimizeBranchReorderAlgorithm(std::move(CAlgo)));
      break;

    case LT_OPTIMIZE_CACHE:
      Algo.reset(new OptimizeCacheReorderAlgorithm(std::move(CAlgo)));
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 528-535

```cpp
    case LT_OPTIMIZE_EXT_TSP:
      Algo.reset(new ExtTSPReorderAlgorithm());
      break;

    case LT_OPTIMIZE_SHUFFLE:
      Algo.reset(new RandomClusterReorderAlgorithm(std::move(CAlgo)));
      break;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 536-545

```cpp
    default:
      llvm_unreachable("unexpected layout type");
    }
  }

  Algo->reorderBasicBlocks(BF, NewLayout);

  return BF.getLayout().update(NewLayout);
}
```

- EN: Declares or implements routines including `llvm_unreachable`, `reorderBasicBlocks`. Notable symbols here include `llvm_unreachable`, `reorderBasicBlocks`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `reorderBasicBlocks`。这里较值得关注的符号包括 `llvm_unreachable`, `reorderBasicBlocks`。

### Lines 546-556

```cpp
Error FixupBranches::runOnFunctions(BinaryContext &BC) {
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (!BC.shouldEmit(Function) || !Function.isSimple())
      continue;

    Function.fixBranches();
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 557-567

```cpp
Error PopulateOutputFunctions::runOnFunctions(BinaryContext &BC) {
  BinaryFunctionListType &OutputFunctions = BC.getOutputBinaryFunctions();

  assert(OutputFunctions.empty() && "Output function list already initialized");

  OutputFunctions.reserve(BC.getBinaryFunctions().size() +
                          BC.getInjectedBinaryFunctions().size());
  llvm::transform(llvm::make_second_range(BC.getBinaryFunctions()),
                  std::back_inserter(OutputFunctions),
                  [](BinaryFunction &BF) { return &BF; });
```

- EN: Declares or implements routines including `runOnFunctions`, `assert`, `transform`, `back_inserter`. Notable symbols here include `runOnFunctions`, `assert`, `transform`, `back_inserter`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `assert`, `transform`, `back_inserter`。这里较值得关注的符号包括 `runOnFunctions`, `assert`, `transform`, `back_inserter`。

### Lines 568-575

```cpp
  llvm::erase_if(OutputFunctions,
                 [&BC](BinaryFunction *BF) { return !BC.shouldEmit(*BF); });

  llvm::stable_sort(OutputFunctions, compareBinaryFunctionByIndex);

  llvm::copy(BC.getInjectedBinaryFunctions(),
             std::back_inserter(OutputFunctions));
```

- EN: Declares or implements routines including `stable_sort`, `copy`, `back_inserter`. Notable symbols here include `stable_sort`, `copy`, `back_inserter`.
- CN: 这里声明或实现函数，例如 `stable_sort`, `copy`, `back_inserter`。这里较值得关注的符号包括 `stable_sort`, `copy`, `back_inserter`。

### Lines 576-588

```cpp
  // Place hot text movers in front.
  if (opts::HotText) {
    std::stable_partition(
        OutputFunctions.begin(), OutputFunctions.end(),
        [](const BinaryFunction *A) { return opts::isHotTextMover(*A); });
  }

  if (opts::HotFunctionsAtEnd) {
    std::stable_partition(
        OutputFunctions.begin(), OutputFunctions.end(),
        [](const BinaryFunction *A) { return !A->hasValidIndex(); });
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 589-605

```cpp
  return Error::success();
}

Error FinalizeFunctions::runOnFunctions(BinaryContext &BC) {
  std::atomic<bool> HasFatal{false};
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    if (!BF.finalizeCFIState()) {
      if (BC.HasRelocations) {
        BC.errs() << "BOLT-ERROR: unable to fix CFI state for function " << BF
                  << ". Exiting.\n";
        HasFatal = true;
        return;
      }
      BF.setSimple(false);
      return;
    }
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 606-615

```cpp
    BF.setFinalized();

    // Update exception handling information.
    BF.updateEHRanges();
  };

  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    return !BC.shouldEmit(BF);
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 616-623

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_CONSTANT, WorkFun,
      SkipPredicate, "FinalizeFunctions");
  if (HasFatal)
    return createFatalBOLTError("finalize CFI state failure");
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 624-641

```cpp
Error CheckLargeFunctions::runOnFunctions(BinaryContext &BC) {
  if (BC.HasRelocations)
    return Error::success();

  // If the function wouldn't fit, mark it as non-simple. Otherwise, we may emit
  // incorrect meta data.
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    uint64_t HotSize, ColdSize;
    std::tie(HotSize, ColdSize) =
        BC.calculateEmittedSize(BF, /*FixBranches=*/false);
    uint64_t MainFragmentSize = HotSize;
    if (BF.hasIslandsInfo()) {
      MainFragmentSize +=
          offsetToAlignment(BF.getAddress() + MainFragmentSize,
                            Align(BF.getConstantIslandAlignment()));
      MainFragmentSize += BF.estimateConstantIslandSize();
    }
    if (MainFragmentSize > BF.getMaxSize()) {
```

- EN: Declares or implements routines including `runOnFunctions`, `tie`, `offsetToAlignment`, `Align`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`, `tie`, `offsetToAlignment`, `Align`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `tie`, `offsetToAlignment`, `Align`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`, `tie`, `offsetToAlignment`, `Align`。

### Lines 642-649

```cpp
      if (opts::PrintLargeFunctions)
        BC.outs() << "BOLT-INFO: " << BF << " size of " << MainFragmentSize
                  << " bytes exceeds allocated space by "
                  << (MainFragmentSize - BF.getMaxSize()) << " bytes\n";
      BF.setSimple(false);
    }
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 650-657

```cpp
  ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
    return !shouldOptimize(BF);
  };

  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
      SkipFunc, "CheckLargeFunctions");
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 658-665

```cpp
  return Error::success();
}

bool CheckLargeFunctions::shouldOptimize(const BinaryFunction &BF) const {
  // Unlike other passes, allow functions in non-CFG state.
  return BF.isSimple() && !BF.isIgnored();
}
```

- EN: Declares or implements routines including `shouldOptimize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `shouldOptimize`.
- CN: 这里声明或实现函数，例如 `shouldOptimize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `shouldOptimize`。

### Lines 666-677

```cpp
Error LowerAnnotations::runOnFunctions(BinaryContext &BC) {
  // Convert GnuArgsSize annotations into CFIs.
  for (BinaryFunction *BF : BC.getAllBinaryFunctions()) {
    for (FunctionFragment &FF : BF->getLayout().fragments()) {
      // Reset at the start of the new fragment.
      int64_t CurrentGnuArgsSize = 0;

      for (BinaryBasicBlock *const BB : FF) {
        for (auto II = BB->begin(); II != BB->end(); ++II) {
          if (!BF->usesGnuArgsSize() || !BC.MIB->isInvoke(*II))
            continue;
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 678-694

```cpp
          const int64_t NewGnuArgsSize = BC.MIB->getGnuArgsSize(*II);
          assert(NewGnuArgsSize >= 0 && "Expected non-negative GNU_args_size.");
          if (NewGnuArgsSize == CurrentGnuArgsSize)
            continue;

          auto InsertII = BF->addCFIInstruction(
              BB, II,
              MCCFIInstruction::createGnuArgsSize(nullptr, NewGnuArgsSize));
          CurrentGnuArgsSize = NewGnuArgsSize;
          II = std::next(InsertII);
        }
      }
    }
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `getGnuArgsSize`, `assert`, `createGnuArgsSize`, `next`. Notable symbols here include `getGnuArgsSize`, `assert`, `createGnuArgsSize`, `next`.
- CN: 这里声明或实现函数，例如 `getGnuArgsSize`, `assert`, `createGnuArgsSize`, `next`。这里较值得关注的符号包括 `getGnuArgsSize`, `assert`, `createGnuArgsSize`, `next`。

### Lines 695-712

```cpp
// Check for dirty state in MCSymbol objects that might be a consequence
// of running calculateEmittedSize() in parallel, during split functions
// pass. If an inconsistent state is found (symbol already registered or
// already defined), clean it.
Error CleanMCState::runOnFunctions(BinaryContext &BC) {
  MCContext &Ctx = *BC.Ctx;
  for (const auto &SymMapEntry : Ctx.getSymbols()) {
    const MCSymbol *S = SymMapEntry.getValue().Symbol;
    if (!S)
      continue;
    if (S->isDefined()) {
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: Symbol \"" << S->getName()
                        << "\" is already defined\n");
      const_cast<MCSymbol *>(S)->setFragment(nullptr);
    }
    if (S->isRegistered()) {
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: Symbol \"" << S->getName()
                        << "\" is already registered\n");
```

- EN: Declares or implements routines including `runOnFunctions`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`, `LLVM_DEBUG`。

### Lines 713-721

```cpp
      const_cast<MCSymbol *>(S)->setIsRegistered(false);
    }
    LLVM_DEBUG(if (S->isVariable()) {
      dbgs() << "BOLT-DEBUG: Symbol \"" << S->getName() << "\" is variable\n";
    });
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dbgs`. Notable symbols here include `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dbgs`。这里较值得关注的符号包括 `LLVM_DEBUG`, `dbgs`。

### Lines 722-737

```cpp
// This peephole fixes jump instructions that jump to another basic
// block with a single jump instruction, e.g.
//
// B0: ...
//     jmp  B1   (or jcc B1)
//
// B1: jmp  B2
//
// ->
//
// B0: ...
//     jmp  B2   (or jcc B2)
//
static uint64_t fixDoubleJumps(BinaryFunction &Function, bool MarkInvalid) {
  uint64_t NumDoubleJumps = 0;
```

- EN: Declares or implements routines including `fixDoubleJumps`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fixDoubleJumps`.
- CN: 这里声明或实现函数，例如 `fixDoubleJumps`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fixDoubleJumps`。

### Lines 738-747

```cpp
  MCContext *Ctx = Function.getBinaryContext().Ctx.get();
  MCPlusBuilder *MIB = Function.getBinaryContext().MIB.get();
  for (BinaryBasicBlock &BB : Function) {
    auto checkAndPatch = [&](BinaryBasicBlock *Pred, BinaryBasicBlock *Succ,
                             const MCSymbol *SuccSym,
                             std::optional<uint32_t> Offset) {
      // Ignore infinite loop jumps or fallthrough tail jumps.
      if (Pred == Succ || Succ == &BB)
        return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 748-760

```cpp
      if (Succ) {
        const MCSymbol *TBB = nullptr;
        const MCSymbol *FBB = nullptr;
        MCInst *CondBranch = nullptr;
        MCInst *UncondBranch = nullptr;
        bool Res = Pred->analyzeBranch(TBB, FBB, CondBranch, UncondBranch);
        if (!Res) {
          LLVM_DEBUG(dbgs() << "analyzeBranch failed in peepholes in block:\n";
                     Pred->dump());
          return false;
        }
        Pred->replaceSuccessor(&BB, Succ);
```

- EN: Declares or implements routines including `analyzeBranch`, `LLVM_DEBUG`, `dump`, `replaceSuccessor`. Notable symbols here include `analyzeBranch`, `LLVM_DEBUG`, `dump`, `replaceSuccessor`.
- CN: 这里声明或实现函数，例如 `analyzeBranch`, `LLVM_DEBUG`, `dump`, `replaceSuccessor`。这里较值得关注的符号包括 `analyzeBranch`, `LLVM_DEBUG`, `dump`, `replaceSuccessor`。

### Lines 761-778

```cpp
        // We must patch up any existing branch instructions to match up
        // with the new successor.
        assert((CondBranch || (!CondBranch && Pred->succ_size() == 1)) &&
               "Predecessor block has inconsistent number of successors");
        if (CondBranch && MIB->getTargetSymbol(*CondBranch) == BB.getLabel()) {
          MIB->replaceBranchTarget(*CondBranch, Succ->getLabel(), Ctx);
        } else if (UncondBranch &&
                   MIB->getTargetSymbol(*UncondBranch) == BB.getLabel()) {
          MIB->replaceBranchTarget(*UncondBranch, Succ->getLabel(), Ctx);
        } else if (!UncondBranch) {
          assert(Function.getLayout().getBasicBlockAfter(Pred, false) != Succ &&
                 "Don't add an explicit jump to a fallthrough block.");
          Pred->addBranchInstruction(Succ);
        }
      } else {
        // Succ will be null in the tail call case.  In this case we
        // need to explicitly add a tail call instruction.
        MCInst *Branch = Pred->getLastNonPseudoInstr();
```

- EN: Declares or implements routines including `assert`, `replaceBranchTarget`, `getTargetSymbol`, `if`, `addBranchInstruction`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `replaceBranchTarget`, `getTargetSymbol`, `if`, `addBranchInstruction`, `getLastNonPseudoInstr`.
- CN: 这里声明或实现函数，例如 `assert`, `replaceBranchTarget`, `getTargetSymbol`, `if`, `addBranchInstruction`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `replaceBranchTarget`, `getTargetSymbol`, `if`, `addBranchInstruction`, `getLastNonPseudoInstr`。

### Lines 779-793

```cpp
        if (Branch && MIB->isUnconditionalBranch(*Branch)) {
          assert(MIB->getTargetSymbol(*Branch) == BB.getLabel());
          Pred->removeSuccessor(&BB);
          Pred->eraseInstruction(Pred->findInstruction(Branch));
          Pred->addTailCallInstruction(SuccSym);
          if (Offset) {
            MCInst *TailCall = Pred->getLastNonPseudoInstr();
            assert(TailCall);
            MIB->setOffset(*TailCall, *Offset);
          }
        } else {
          return false;
        }
      }
```

- EN: Declares or implements routines including `assert`, `removeSuccessor`, `eraseInstruction`, `addTailCallInstruction`, `getLastNonPseudoInstr`, and 1 more. Notable symbols here include `assert`, `removeSuccessor`, `eraseInstruction`, `addTailCallInstruction`, `getLastNonPseudoInstr`, `setOffset`.
- CN: 这里声明或实现函数，例如 `assert`, `removeSuccessor`, `eraseInstruction`, `addTailCallInstruction`, `getLastNonPseudoInstr`, and 1 more。这里较值得关注的符号包括 `assert`, `removeSuccessor`, `eraseInstruction`, `addTailCallInstruction`, `getLastNonPseudoInstr`, `setOffset`。

### Lines 794-802

```cpp
      ++NumDoubleJumps;
      LLVM_DEBUG(dbgs() << "Removed double jump in " << Function << " from "
                        << Pred->getName() << " -> " << BB.getName() << " to "
                        << Pred->getName() << " -> " << SuccSym->getName()
                        << (!Succ ? " (tail)\n" : "\n"));

      return true;
    };
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`. Notable symbols here include `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`。

### Lines 803-811

```cpp
    if (BB.getNumNonPseudos() != 1 || BB.isLandingPad())
      continue;

    MCInst *Inst = BB.getFirstNonPseudoInstr();
    const bool IsTailCall = MIB->isTailCall(*Inst);

    if (!MIB->isUnconditionalBranch(*Inst) && !IsTailCall)
      continue;
```

- EN: Declares or implements routines including `isTailCall`. Notable symbols here include `isTailCall`.
- CN: 这里声明或实现函数，例如 `isTailCall`。这里较值得关注的符号包括 `isTailCall`。

### Lines 812-821

```cpp
    // If we operate after SCTC make sure it's not a conditional tail call.
    if (IsTailCall && MIB->isConditionalBranch(*Inst))
      continue;

    const MCSymbol *SuccSym = MIB->getTargetSymbol(*Inst);
    BinaryBasicBlock *Succ = BB.getSuccessor();

    if (((!Succ || &BB == Succ) && !IsTailCall) || (IsTailCall && !SuccSym))
      continue;
```

- EN: Declares or implements routines including `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 822-837

```cpp
    std::vector<BinaryBasicBlock *> Preds = {BB.pred_begin(), BB.pred_end()};

    for (BinaryBasicBlock *Pred : Preds) {
      if (Pred->isLandingPad())
        continue;

      if (Pred->getSuccessor() == &BB ||
          (Pred->getConditionalSuccessor(true) == &BB && !IsTailCall) ||
          Pred->getConditionalSuccessor(false) == &BB)
        if (checkAndPatch(Pred, Succ, SuccSym, MIB->getOffset(*Inst)) &&
            MarkInvalid)
          BB.markValid(BB.pred_size() != 0 || BB.isLandingPad() ||
                       BB.isEntryPoint());
    }
  }
```

- EN: Declares or implements routines including `getConditionalSuccessor`. Notable symbols here include `getConditionalSuccessor`.
- CN: 这里声明或实现函数，例如 `getConditionalSuccessor`。这里较值得关注的符号包括 `getConditionalSuccessor`。

### Lines 838-846

```cpp
  return NumDoubleJumps;
}

bool SimplifyConditionalTailCalls::shouldRewriteBranch(
    const BinaryBasicBlock *PredBB, const MCInst &CondBranch,
    const BinaryBasicBlock *BB, const bool DirectionFlag) {
  if (BeenOptimized.count(PredBB))
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 847-856

```cpp
  const bool IsForward = BinaryFunction::isForwardBranch(PredBB, BB);

  if (IsForward)
    ++NumOrigForwardBranches;
  else
    ++NumOrigBackwardBranches;

  if (opts::SctcMode == opts::SctcAlways)
    return true;
```

- EN: Declares or implements routines including `isForwardBranch`. Notable symbols here include `isForwardBranch`.
- CN: 这里声明或实现函数，例如 `isForwardBranch`。这里较值得关注的符号包括 `isForwardBranch`。

### Lines 857-865

```cpp
  if (opts::SctcMode == opts::SctcPreserveDirection)
    return IsForward == DirectionFlag;

  const ErrorOr<std::pair<double, double>> Frequency =
      PredBB->getBranchStats(BB);

  // It's ok to rewrite the conditional branch if the new target will be
  // a backward branch.
```

- EN: Declares or implements routines including `getBranchStats`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getBranchStats`.
- CN: 这里声明或实现函数，例如 `getBranchStats`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getBranchStats`。

### Lines 866-874

```cpp
  // If no data available for these branches, then it should be ok to
  // do the optimization since it will reduce code size.
  if (Frequency.getError())
    return true;

  // TODO: should this use misprediction frequency instead?
  const bool Result = (IsForward && Frequency.get().first >= 0.5) ||
                      (!IsForward && Frequency.get().first <= 0.5);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 875-882

```cpp
  return Result == DirectionFlag;
}

uint64_t SimplifyConditionalTailCalls::fixTailCalls(BinaryFunction &BF) {
  // Need updated indices to correctly detect branch' direction.
  BF.getLayout().updateLayoutIndices();
  BF.markUnreachableBlocks();
```

- EN: Declares or implements routines including `fixTailCalls`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fixTailCalls`.
- CN: 这里声明或实现函数，例如 `fixTailCalls`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fixTailCalls`。

### Lines 883-891

```cpp
  MCPlusBuilder *MIB = BF.getBinaryContext().MIB.get();
  MCContext *Ctx = BF.getBinaryContext().Ctx.get();
  uint64_t NumLocalCTCCandidates = 0;
  uint64_t NumLocalCTCs = 0;
  uint64_t LocalCTCTakenCount = 0;
  uint64_t LocalCTCExecCount = 0;
  std::vector<std::pair<BinaryBasicBlock *, const BinaryBasicBlock *>>
      NeedsUncondBranch;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 892-901

```cpp
  // Will block be deleted by UCE?
  auto isValid = [](const BinaryBasicBlock *BB) {
    return (BB->pred_size() != 0 || BB->isLandingPad() || BB->isEntryPoint());
  };

  for (BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    // Locate BB with a single direct tail-call instruction.
    if (BB->getNumNonPseudos() != 1)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 902-909

```cpp
    MCInst *Instr = BB->getFirstNonPseudoInstr();
    if (!MIB->isTailCall(*Instr) || MIB->isConditionalBranch(*Instr))
      continue;

    const MCSymbol *CalleeSymbol = MIB->getTargetSymbol(*Instr);
    if (!CalleeSymbol)
      continue;
```

- EN: Declares or implements routines including `getFirstNonPseudoInstr`, `getTargetSymbol`. Notable symbols here include `getFirstNonPseudoInstr`, `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getFirstNonPseudoInstr`, `getTargetSymbol`。这里较值得关注的符号包括 `getFirstNonPseudoInstr`, `getTargetSymbol`。

### Lines 910-918

```cpp
    // Detect direction of the possible conditional tail call.
    const bool IsForwardCTC = BF.isForwardCall(CalleeSymbol);

    // Iterate through all predecessors.
    for (BinaryBasicBlock *PredBB : BB->predecessors()) {
      BinaryBasicBlock *CondSucc = PredBB->getConditionalSuccessor(true);
      if (!CondSucc)
        continue;
```

- EN: Declares or implements routines including `getConditionalSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getConditionalSuccessor`.
- CN: 这里声明或实现函数，例如 `getConditionalSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getConditionalSuccessor`。

### Lines 919-926

```cpp
      ++NumLocalCTCCandidates;

      const MCSymbol *TBB = nullptr;
      const MCSymbol *FBB = nullptr;
      MCInst *CondBranch = nullptr;
      MCInst *UncondBranch = nullptr;
      bool Result = PredBB->analyzeBranch(TBB, FBB, CondBranch, UncondBranch);
```

- EN: Declares or implements routines including `analyzeBranch`. Notable symbols here include `analyzeBranch`.
- CN: 这里声明或实现函数，例如 `analyzeBranch`。这里较值得关注的符号包括 `analyzeBranch`。

### Lines 927-936

```cpp
      // analyzeBranch() can fail due to unusual branch instructions, e.g. jrcxz
      if (!Result) {
        LLVM_DEBUG(dbgs() << "analyzeBranch failed in SCTC in block:\n";
                   PredBB->dump());
        continue;
      }

      assert(Result && "internal error analyzing conditional branch");
      assert(CondBranch && "conditional branch expected");
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dump`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `dump`, `assert`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dump`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `dump`, `assert`。

### Lines 937-950

```cpp
      // Skip dynamic branches for now.
      if (BF.getBinaryContext().MIB->isDynamicBranch(*CondBranch))
        continue;

      // It's possible that PredBB is also a successor to BB that may have
      // been processed by a previous iteration of the SCTC loop, in which
      // case it may have been marked invalid.  We should skip rewriting in
      // this case.
      if (!PredBB->isValid()) {
        assert(PredBB->isSuccessor(BB) &&
               "PredBB should be valid if it is not a successor to BB");
        continue;
      }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 951-959

```cpp
      // We don't want to reverse direction of the branch in new order
      // without further profile analysis.
      const bool DirectionFlag = CondSucc == BB ? IsForwardCTC : !IsForwardCTC;
      if (!shouldRewriteBranch(PredBB, *CondBranch, BB, DirectionFlag))
        continue;

      // Record this block so that we don't try to optimize it twice.
      BeenOptimized.insert(PredBB);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 960-977

```cpp
      uint64_t Count = 0;
      if (CondSucc != BB) {
        // Patch the new target address into the conditional branch.
        MIB->reverseBranchCondition(*CondBranch, CalleeSymbol, Ctx);
        // Since we reversed the condition on the branch we need to change
        // the target for the unconditional branch or add a unconditional
        // branch to the old target.  This has to be done manually since
        // fixupBranches is not called after SCTC.
        NeedsUncondBranch.emplace_back(PredBB, CondSucc);
        Count = PredBB->getFallthroughBranchInfo().Count;
      } else {
        // Change destination of the conditional branch.
        MIB->replaceBranchTarget(*CondBranch, CalleeSymbol, Ctx);
        Count = PredBB->getTakenBranchInfo().Count;
      }
      const uint64_t CTCTakenFreq =
          Count == BinaryBasicBlock::COUNT_NO_PROFILE ? 0 : Count;
```

- EN: Declares or implements routines including `reverseBranchCondition`, `getFallthroughBranchInfo`, `replaceBranchTarget`, `getTakenBranchInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `reverseBranchCondition`, `getFallthroughBranchInfo`, `replaceBranchTarget`, `getTakenBranchInfo`.
- CN: 这里声明或实现函数，例如 `reverseBranchCondition`, `getFallthroughBranchInfo`, `replaceBranchTarget`, `getTakenBranchInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `reverseBranchCondition`, `getFallthroughBranchInfo`, `replaceBranchTarget`, `getTakenBranchInfo`。

### Lines 978-990

```cpp
      // Annotate it, so "isCall" returns true for this jcc
      MIB->setConditionalTailCall(*CondBranch);
      // Add info about the conditional tail call frequency, otherwise this
      // info will be lost when we delete the associated BranchInfo entry
      auto &CTCAnnotation =
          MIB->getOrCreateAnnotationAs<uint64_t>(*CondBranch, "CTCTakenCount");
      CTCAnnotation = CTCTakenFreq;
      // Preserve Offset annotation, used in BAT.
      // Instr is a direct tail call instruction that was created when CTCs are
      // first expanded, and has the original CTC offset set.
      if (std::optional<uint32_t> Offset = MIB->getOffset(*Instr))
        MIB->setOffset(*CondBranch, *Offset);
```

- EN: Declares or implements routines including `setConditionalTailCall`, `setOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setConditionalTailCall`, `setOffset`.
- CN: 这里声明或实现函数，例如 `setConditionalTailCall`, `setOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setConditionalTailCall`, `setOffset`。

### Lines 991-999

```cpp
      // Remove the unused successor which may be eliminated later
      // if there are no other users.
      PredBB->removeSuccessor(BB);
      // Update BB execution count
      if (CTCTakenFreq && CTCTakenFreq <= BB->getKnownExecutionCount())
        BB->setExecutionCount(BB->getExecutionCount() - CTCTakenFreq);
      else if (CTCTakenFreq > BB->getKnownExecutionCount())
        BB->setExecutionCount(0);
```

- EN: Declares or implements routines including `removeSuccessor`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `removeSuccessor`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `removeSuccessor`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `removeSuccessor`, `setExecutionCount`。

### Lines 1000-1008

```cpp
      ++NumLocalCTCs;
      LocalCTCTakenCount += CTCTakenFreq;
      LocalCTCExecCount += PredBB->getKnownExecutionCount();
    }

    // Remove the block from CFG if all predecessors were removed.
    BB->markValid(isValid(BB));
  }
```

- EN: Declares or implements routines including `getKnownExecutionCount`, `markValid`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getKnownExecutionCount`, `markValid`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`, `markValid`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getKnownExecutionCount`, `markValid`。

### Lines 1009-1020

```cpp
  // Add unconditional branches at the end of BBs to new successors
  // as long as the successor is not a fallthrough.
  for (auto &Entry : NeedsUncondBranch) {
    BinaryBasicBlock *PredBB = Entry.first;
    const BinaryBasicBlock *CondSucc = Entry.second;

    const MCSymbol *TBB = nullptr;
    const MCSymbol *FBB = nullptr;
    MCInst *CondBranch = nullptr;
    MCInst *UncondBranch = nullptr;
    PredBB->analyzeBranch(TBB, FBB, CondBranch, UncondBranch);
```

- EN: Declares or implements routines including `analyzeBranch`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `analyzeBranch`.
- CN: 这里声明或实现函数，例如 `analyzeBranch`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `analyzeBranch`。

### Lines 1021-1031

```cpp
    // Find the next valid block.  Invalid blocks will be deleted
    // so they shouldn't be considered fallthrough targets.
    const BinaryBasicBlock *NextBlock =
        BF.getLayout().getBasicBlockAfter(PredBB, false);
    while (NextBlock && !isValid(NextBlock))
      NextBlock = BF.getLayout().getBasicBlockAfter(NextBlock, false);

    // Get the unconditional successor to this block.
    const BinaryBasicBlock *PredSucc = PredBB->getSuccessor();
    assert(PredSucc && "The other branch should be a tail call");
```

- EN: Declares or implements routines including `getSuccessor`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSuccessor`, `assert`.
- CN: 这里声明或实现函数，例如 `getSuccessor`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSuccessor`, `assert`。

### Lines 1032-1045

```cpp
    const bool HasFallthrough = (NextBlock && PredSucc == NextBlock);

    if (UncondBranch) {
      if (HasFallthrough)
        PredBB->eraseInstruction(PredBB->findInstruction(UncondBranch));
      else
        MIB->replaceBranchTarget(*UncondBranch, CondSucc->getLabel(), Ctx);
    } else if (!HasFallthrough) {
      MCInst Branch;
      MIB->createUncondBranch(Branch, CondSucc->getLabel(), Ctx);
      PredBB->addInstruction(Branch);
    }
  }
```

- EN: Declares or implements routines including `eraseInstruction`, `replaceBranchTarget`, `if`, `createUncondBranch`, `addInstruction`. Notable symbols here include `eraseInstruction`, `replaceBranchTarget`, `if`, `createUncondBranch`, `addInstruction`.
- CN: 这里声明或实现函数，例如 `eraseInstruction`, `replaceBranchTarget`, `if`, `createUncondBranch`, `addInstruction`。这里较值得关注的符号包括 `eraseInstruction`, `replaceBranchTarget`, `if`, `createUncondBranch`, `addInstruction`。

### Lines 1046-1055

```cpp
  if (NumLocalCTCs > 0) {
    NumDoubleJumps += fixDoubleJumps(BF, true);
    // Clean-up unreachable tail-call blocks.
    const std::pair<unsigned, uint64_t> Stats = BF.eraseInvalidBBs();
    DeletedBlocks += Stats.first;
    DeletedBytes += Stats.second;

    assert(BF.validateCFG());
  }
```

- EN: Declares or implements routines including `fixDoubleJumps`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fixDoubleJumps`, `assert`.
- CN: 这里声明或实现函数，例如 `fixDoubleJumps`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fixDoubleJumps`, `assert`。

### Lines 1056-1067

```cpp
  LLVM_DEBUG(dbgs() << "BOLT: created " << NumLocalCTCs
                    << " conditional tail calls from a total of "
                    << NumLocalCTCCandidates << " candidates in function " << BF
                    << ". CTCs execution count for this function is "
                    << LocalCTCExecCount << " and CTC taken count is "
                    << LocalCTCTakenCount << "\n";);

  NumTailCallsPatched += NumLocalCTCs;
  NumCandidateTailCalls += NumLocalCTCCandidates;
  CTCExecCount += LocalCTCExecCount;
  CTCTakenCount += LocalCTCTakenCount;
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 1068-1077

```cpp
  return NumLocalCTCs > 0;
}

Error SimplifyConditionalTailCalls::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86())
    return Error::success();

  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1078-1086

```cpp
    if (!shouldOptimize(Function))
      continue;

    if (fixTailCalls(Function)) {
      Modified.insert(&Function);
      Function.setHasCanonicalCFG(false);
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1087-1100

```cpp
  if (NumTailCallsPatched)
    BC.outs() << "BOLT-INFO: SCTC: patched " << NumTailCallsPatched
              << " tail calls (" << NumOrigForwardBranches << " forward)"
              << " tail calls (" << NumOrigBackwardBranches << " backward)"
              << " from a total of " << NumCandidateTailCalls
              << " while removing " << NumDoubleJumps << " double jumps"
              << " and removing " << DeletedBlocks << " basic blocks"
              << " totalling " << DeletedBytes
              << " bytes of code. CTCs total execution count is "
              << CTCExecCount << " and the number of times CTCs are taken is "
              << CTCTakenCount << "\n";
  return Error::success();
}
```

- EN: Declares or implements routines including `calls`. Notable symbols here include `calls`.
- CN: 这里声明或实现函数，例如 `calls`。这里较值得关注的符号包括 `calls`。

### Lines 1101-1109

```cpp
uint64_t ShortenInstructions::shortenInstructions(BinaryFunction &Function) {
  uint64_t Count = 0;
  const BinaryContext &BC = Function.getBinaryContext();
  for (BinaryBasicBlock &BB : Function) {
    for (MCInst &Inst : BB) {
      // Skip shortening instructions with Size annotation.
      if (BC.MIB->getSize(Inst))
        continue;
```

- EN: Declares or implements routines including `shortenInstructions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `shortenInstructions`.
- CN: 这里声明或实现函数，例如 `shortenInstructions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `shortenInstructions`。

### Lines 1110-1124

```cpp
      MCInst OriginalInst;
      if (opts::Verbosity > 2)
        OriginalInst = Inst;

      if (!BC.MIB->shortenInstruction(Inst, *BC.STI))
        continue;

      if (opts::Verbosity > 2) {
        BC.scopeLock();
        BC.outs() << "BOLT-INFO: shortening:\nBOLT-INFO:    ";
        BC.printInstruction(BC.outs(), OriginalInst, 0, &Function);
        BC.outs() << "BOLT-INFO: to:";
        BC.printInstruction(BC.outs(), Inst, 0, &Function);
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1125-1136

```cpp
      ++Count;
    }
  }

  return Count;
}

Error ShortenInstructions::runOnFunctions(BinaryContext &BC) {
  std::atomic<uint64_t> NumShortened{0};
  if (!BC.isX86())
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1137-1147

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR,
      [&](BinaryFunction &BF) { NumShortened += shortenInstructions(BF); },
      nullptr, "ShortenInstructions");

  if (NumShortened)
    BC.outs() << "BOLT-INFO: " << NumShortened
              << " instructions were shortened\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1148-1160

```cpp
void Peepholes::addTailcallTraps(BinaryFunction &Function) {
  MCPlusBuilder *MIB = Function.getBinaryContext().MIB.get();
  for (BinaryBasicBlock &BB : Function) {
    MCInst *Inst = BB.getLastNonPseudoInstr();
    if (Inst && MIB->isTailCall(*Inst) && MIB->isIndirectBranch(*Inst)) {
      MCInst Trap;
      MIB->createTrap(Trap);
      BB.addInstruction(Trap);
      ++TailCallTraps;
    }
  }
}
```

- EN: Declares or implements routines including `addTailcallTraps`, `createTrap`. Notable symbols here include `addTailcallTraps`, `createTrap`.
- CN: 这里声明或实现函数，例如 `addTailcallTraps`, `createTrap`。这里较值得关注的符号包括 `addTailcallTraps`, `createTrap`。

### Lines 1161-1170

```cpp
void Peepholes::removeUselessCondBranches(BinaryFunction &Function) {
  for (BinaryBasicBlock &BB : Function) {
    if (BB.succ_size() != 2)
      continue;

    BinaryBasicBlock *CondBB = BB.getConditionalSuccessor(true);
    BinaryBasicBlock *UncondBB = BB.getConditionalSuccessor(false);
    if (CondBB != UncondBB)
      continue;
```

- EN: Declares or implements routines including `removeUselessCondBranches`. Notable symbols here include `removeUselessCondBranches`.
- CN: 这里声明或实现函数，例如 `removeUselessCondBranches`。这里较值得关注的符号包括 `removeUselessCondBranches`。

### Lines 1171-1181

```cpp
    const MCSymbol *TBB = nullptr;
    const MCSymbol *FBB = nullptr;
    MCInst *CondBranch = nullptr;
    MCInst *UncondBranch = nullptr;
    bool Result = BB.analyzeBranch(TBB, FBB, CondBranch, UncondBranch);

    // analyzeBranch() can fail due to unusual branch instructions,
    // e.g. jrcxz, or jump tables (indirect jump).
    if (!Result || !CondBranch)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1182-1193

```cpp
    BB.removeDuplicateConditionalSuccessor(CondBranch);
    ++NumUselessCondBranches;
  }
}

Error Peepholes::runOnFunctions(BinaryContext &BC) {
  const char Opts =
      std::accumulate(opts::Peepholes.begin(), opts::Peepholes.end(), 0,
                      [](const char A, const PeepholeOpts B) { return A | B; });
  if (Opts == PEEP_NONE)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`, `accumulate`. Notable symbols here include `runOnFunctions`, `accumulate`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `accumulate`。这里较值得关注的符号包括 `runOnFunctions`, `accumulate`。

### Lines 1194-1211

```cpp
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (shouldOptimize(Function)) {
      if (Opts & PEEP_DOUBLE_JUMPS)
        NumDoubleJumps += fixDoubleJumps(Function, false);
      if (Opts & PEEP_TAILCALL_TRAPS)
        addTailcallTraps(Function);
      if (Opts & PEEP_USELESS_BRANCHES)
        removeUselessCondBranches(Function);
      assert(Function.validateCFG());
    }
  }
  BC.outs() << "BOLT-INFO: Peephole: " << NumDoubleJumps
            << " double jumps patched.\n"
            << "BOLT-INFO: Peephole: " << TailCallTraps
            << " tail call traps inserted.\n"
            << "BOLT-INFO: Peephole: " << NumUselessCondBranches
            << " useless conditional branches removed.\n";
```

- EN: Declares or implements routines including `fixDoubleJumps`, `addTailcallTraps`, `removeUselessCondBranches`, `assert`. Notable symbols here include `fixDoubleJumps`, `addTailcallTraps`, `removeUselessCondBranches`, `assert`.
- CN: 这里声明或实现函数，例如 `fixDoubleJumps`, `addTailcallTraps`, `removeUselessCondBranches`, `assert`。这里较值得关注的符号包括 `fixDoubleJumps`, `addTailcallTraps`, `removeUselessCondBranches`, `assert`。

### Lines 1212-1223

```cpp
  return Error::success();
}

bool SimplifyRODataLoads::simplifyRODataLoads(BinaryFunction &BF) {
  BinaryContext &BC = BF.getBinaryContext();
  MCPlusBuilder *MIB = BC.MIB.get();

  uint64_t NumLocalLoadsSimplified = 0;
  uint64_t NumDynamicLocalLoadsSimplified = 0;
  uint64_t NumLocalLoadsFound = 0;
  uint64_t NumDynamicLocalLoadsFound = 0;
```

- EN: Declares or implements routines including `simplifyRODataLoads`. Notable symbols here include `simplifyRODataLoads`.
- CN: 这里声明或实现函数，例如 `simplifyRODataLoads`。这里较值得关注的符号包括 `simplifyRODataLoads`。

### Lines 1224-1232

```cpp
  for (BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    for (MCInst &Inst : *BB) {
      unsigned Opcode = Inst.getOpcode();
      const MCInstrDesc &Desc = BC.MII->get(Opcode);

      // Skip instructions that do not load from memory.
      if (!Desc.mayLoad())
        continue;
```

- EN: Declares or implements routines including `get`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `get`.
- CN: 这里声明或实现函数，例如 `get`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `get`。

### Lines 1233-1242

```cpp
      // Try to statically evaluate the target memory address;
      uint64_t TargetAddress;

      if (MIB->hasPCRelOperand(Inst)) {
        // Try to find the symbol that corresponds to the PC-relative operand.
        MCOperand *DispOpI = MIB->getMemOperandDisp(Inst);
        assert(DispOpI != Inst.end() && "expected PC-relative displacement");
        assert(DispOpI->isExpr() &&
               "found PC-relative with non-symbolic displacement");
```

- EN: Declares or implements routines including `getMemOperandDisp`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getMemOperandDisp`, `assert`.
- CN: 这里声明或实现函数，例如 `getMemOperandDisp`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getMemOperandDisp`, `assert`。

### Lines 1243-1252

```cpp
        // Get displacement symbol.
        const MCSymbol *DisplSymbol;
        uint64_t DisplOffset;

        std::tie(DisplSymbol, DisplOffset) =
            MIB->getTargetSymbolInfo(DispOpI->getExpr());

        if (!DisplSymbol)
          continue;
```

- EN: Declares or implements routines including `tie`, `getTargetSymbolInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tie`, `getTargetSymbolInfo`.
- CN: 这里声明或实现函数，例如 `tie`, `getTargetSymbolInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tie`, `getTargetSymbolInfo`。

### Lines 1253-1262

```cpp
        // Look up the symbol address in the global symbols map of the binary
        // context object.
        BinaryData *BD = BC.getBinaryDataByName(DisplSymbol->getName());
        if (!BD)
          continue;
        TargetAddress = BD->getAddress() + DisplOffset;
      } else if (!MIB->evaluateMemOperandTarget(Inst, TargetAddress)) {
        continue;
      }
```

- EN: Declares or implements routines including `getAddress`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`, `if`.
- CN: 这里声明或实现函数，例如 `getAddress`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`, `if`。

### Lines 1263-1273

```cpp
      // Get the contents of the section containing the target address of the
      // memory operand. We are only interested in read-only sections.
      ErrorOr<BinarySection &> DataSection =
          BC.getSectionForAddress(TargetAddress);
      if (!DataSection || DataSection->isWritable())
        continue;

      if (BC.getRelocationAt(TargetAddress) ||
          BC.getDynamicRelocationAt(TargetAddress))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1274-1288

```cpp
      uint32_t Offset = TargetAddress - DataSection->getAddress();
      StringRef ConstantData = DataSection->getContents();

      ++NumLocalLoadsFound;
      if (BB->hasProfile())
        NumDynamicLocalLoadsFound += BB->getExecutionCount();

      if (MIB->replaceMemOperandWithImm(Inst, ConstantData, Offset)) {
        ++NumLocalLoadsSimplified;
        if (BB->hasProfile())
          NumDynamicLocalLoadsSimplified += BB->getExecutionCount();
      }
    }
  }
```

- EN: Declares or implements routines including `getAddress`, `getContents`, `getExecutionCount`. Notable symbols here include `getAddress`, `getContents`, `getExecutionCount`.
- CN: 这里声明或实现函数，例如 `getAddress`, `getContents`, `getExecutionCount`。这里较值得关注的符号包括 `getAddress`, `getContents`, `getExecutionCount`。

### Lines 1289-1296

```cpp
  NumLoadsFound += NumLocalLoadsFound;
  NumDynamicLoadsFound += NumDynamicLocalLoadsFound;
  NumLoadsSimplified += NumLocalLoadsSimplified;
  NumDynamicLoadsSimplified += NumDynamicLocalLoadsSimplified;

  return NumLocalLoadsSimplified > 0;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1297-1312

```cpp
Error SimplifyRODataLoads::runOnFunctions(BinaryContext &BC) {
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    if (shouldOptimize(Function) && simplifyRODataLoads(Function))
      Modified.insert(&Function);
  }

  BC.outs() << "BOLT-INFO: simplified " << NumLoadsSimplified << " out of "
            << NumLoadsFound << " loads from a statically computed address.\n"
            << "BOLT-INFO: dynamic loads simplified: "
            << NumDynamicLoadsSimplified << "\n"
            << "BOLT-INFO: dynamic loads found: " << NumDynamicLoadsFound
            << "\n";
  return Error::success();
}
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1313-1328

```cpp
Error AssignSections::runOnFunctions(BinaryContext &BC) {
  // In non-relocation mode functions have pre-assigned section names.
  if (!BC.HasRelocations)
    return Error::success();

  const bool UseColdSection =
      BC.NumProfiledFuncs > 0 ||
      opts::ReorderFunctions == ReorderFunctions::RT_USER;
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (opts::isHotTextMover(Function)) {
      Function.setCodeSectionName(BC.getHotTextMoverSectionName());
      Function.setColdCodeSectionName(BC.getHotTextMoverSectionName());
      continue;
    }
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1329-1339

```cpp
    if (!UseColdSection || Function.hasValidIndex())
      Function.setCodeSectionName(BC.getMainCodeSectionName());
    else
      Function.setCodeSectionName(BC.getColdCodeSectionName());

    if (Function.isSplit())
      Function.setColdCodeSectionName(BC.getColdCodeSectionName());
  }
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1340-1352

```cpp
Error PrintProfileStats::runOnFunctions(BinaryContext &BC) {
  double FlowImbalanceMean = 0.0;
  size_t NumBlocksConsidered = 0;
  double WorstBias = 0.0;
  const BinaryFunction *WorstBiasFunc = nullptr;

  // For each function CFG, we fill an IncomingMap with the sum of the frequency
  // of incoming edges for each BB. Likewise for each OutgoingMap and the sum
  // of the frequency of outgoing edges.
  using FlowMapTy = std::unordered_map<const BinaryBasicBlock *, uint64_t>;
  std::unordered_map<const BinaryFunction *, FlowMapTy> TotalIncomingMaps;
  std::unordered_map<const BinaryFunction *, FlowMapTy> TotalOutgoingMaps;
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1353-1370

```cpp
  // Compute mean
  for (const auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &Function = BFI.second;
    if (Function.empty() || !Function.isSimple())
      continue;
    FlowMapTy &IncomingMap = TotalIncomingMaps[&Function];
    FlowMapTy &OutgoingMap = TotalOutgoingMaps[&Function];
    for (const BinaryBasicBlock &BB : Function) {
      uint64_t TotalOutgoing = 0ULL;
      auto SuccBIIter = BB.branch_info_begin();
      for (BinaryBasicBlock *Succ : BB.successors()) {
        uint64_t Count = SuccBIIter->Count;
        if (Count == BinaryBasicBlock::COUNT_NO_PROFILE || Count == 0) {
          ++SuccBIIter;
          continue;
        }
        TotalOutgoing += Count;
        IncomingMap[Succ] += Count;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1371-1386

```cpp
        ++SuccBIIter;
      }
      OutgoingMap[&BB] = TotalOutgoing;
    }

    size_t NumBlocks = 0;
    double Mean = 0.0;
    for (const BinaryBasicBlock &BB : Function) {
      // Do not compute score for low frequency blocks, entry or exit blocks
      if (IncomingMap[&BB] < 100 || OutgoingMap[&BB] == 0 || BB.isEntryPoint())
        continue;
      ++NumBlocks;
      const double Difference = (double)OutgoingMap[&BB] - IncomingMap[&BB];
      Mean += fabs(Difference / IncomingMap[&BB]);
    }
```

- EN: Declares or implements routines including `fabs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `fabs`.
- CN: 这里声明或实现函数，例如 `fabs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `fabs`。

### Lines 1387-1399

```cpp
    FlowImbalanceMean += Mean;
    NumBlocksConsidered += NumBlocks;
    if (!NumBlocks)
      continue;
    double FuncMean = Mean / NumBlocks;
    if (FuncMean > WorstBias) {
      WorstBias = FuncMean;
      WorstBiasFunc = &Function;
    }
  }
  if (NumBlocksConsidered > 0)
    FlowImbalanceMean /= NumBlocksConsidered;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1400-1417

```cpp
  // Compute standard deviation
  NumBlocksConsidered = 0;
  double FlowImbalanceVar = 0.0;
  for (const auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &Function = BFI.second;
    if (Function.empty() || !Function.isSimple())
      continue;
    FlowMapTy &IncomingMap = TotalIncomingMaps[&Function];
    FlowMapTy &OutgoingMap = TotalOutgoingMaps[&Function];
    for (const BinaryBasicBlock &BB : Function) {
      if (IncomingMap[&BB] < 100 || OutgoingMap[&BB] == 0)
        continue;
      ++NumBlocksConsidered;
      const double Difference = (double)OutgoingMap[&BB] - IncomingMap[&BB];
      FlowImbalanceVar +=
          pow(fabs(Difference / IncomingMap[&BB]) - FlowImbalanceMean, 2);
    }
  }
```

- EN: Declares or implements routines including `pow`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `pow`.
- CN: 这里声明或实现函数，例如 `pow`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `pow`。

### Lines 1418-1433

```cpp
  if (NumBlocksConsidered) {
    FlowImbalanceVar /= NumBlocksConsidered;
    FlowImbalanceVar = sqrt(FlowImbalanceVar);
  }

  // Report to user
  BC.outs() << format("BOLT-INFO: Profile bias score: %.4lf%% StDev: %.4lf%%\n",
                      (100.0 * FlowImbalanceMean), (100.0 * FlowImbalanceVar));
  if (WorstBiasFunc && opts::Verbosity >= 1) {
    BC.outs() << "Worst average bias observed in "
              << WorstBiasFunc->getPrintName() << "\n";
    LLVM_DEBUG(WorstBiasFunc->dump());
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `sqrt`, `getPrintName`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sqrt`, `getPrintName`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `sqrt`, `getPrintName`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sqrt`, `getPrintName`, `LLVM_DEBUG`。

### Lines 1434-1449

```cpp
Error PrintProgramStats::runOnFunctions(BinaryContext &BC) {
  uint64_t NumRegularFunctions = 0;
  uint64_t NumStaleProfileFunctions = 0;
  uint64_t NumAllStaleFunctions = 0;
  uint64_t NumInferredFunctions = 0;
  uint64_t NumNonSimpleProfiledFunctions = 0;
  uint64_t NumUnknownControlFlowFunctions = 0;
  uint64_t TotalSampleCount = 0;
  uint64_t StaleSampleCount = 0;
  uint64_t InferredSampleCount = 0;
  std::vector<const BinaryFunction *> ProfiledFunctions;
  std::vector<std::pair<double, uint64_t>> FuncDensityList;
  const char *StaleFuncsHeader = "BOLT-INFO: Functions with stale profile:\n";
  for (auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &Function = BFI.second;
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1450-1460

```cpp
    // Ignore PLT functions for stats.
    if (Function.isPLTFunction())
      continue;

    // Adjustment for BAT mode: the profile for BOLT split fragments is combined
    // so only count the hot fragment.
    const uint64_t Address = Function.getAddress();
    bool IsHotParentOfBOLTSplitFunction = !Function.getFragments().empty() &&
                                          BAT && BAT->isBATFunction(Address) &&
                                          !BAT->fetchParentAddress(Address);
```

- EN: Declares or implements routines including `isBATFunction`, `fetchParentAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isBATFunction`, `fetchParentAddress`.
- CN: 这里声明或实现函数，例如 `isBATFunction`, `fetchParentAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isBATFunction`, `fetchParentAddress`。

### Lines 1461-1471

```cpp
    ++NumRegularFunctions;

    // In BOLTed binaries split functions are non-simple (due to non-relocation
    // mode), but the original function is known to be simple and we have a
    // valid profile for it.
    if (!Function.isSimple() && !IsHotParentOfBOLTSplitFunction) {
      if (Function.hasProfile())
        ++NumNonSimpleProfiledFunctions;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1472-1480

```cpp
    if (Function.hasUnknownControlFlow()) {
      if (opts::PrintUnknownCFG)
        Function.dump();
      else if (opts::PrintUnknown)
        BC.errs() << "function with unknown control flow: " << Function << '\n';

      ++NumUnknownControlFlowFunctions;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1481-1498

```cpp
    if (!Function.hasProfile())
      continue;

    uint64_t SampleCount = Function.getRawSampleCount();
    TotalSampleCount += SampleCount;

    if (Function.hasValidProfile()) {
      ProfiledFunctions.push_back(&Function);
      if (Function.hasInferredProfile()) {
        ++NumInferredFunctions;
        InferredSampleCount += SampleCount;
        ++NumAllStaleFunctions;
      }
    } else {
      if (opts::ReportStaleFuncs) {
        BC.outs() << StaleFuncsHeader;
        StaleFuncsHeader = "";
        BC.outs() << "  " << Function << '\n';
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1499-1516

```cpp
      }
      ++NumStaleProfileFunctions;
      StaleSampleCount += SampleCount;
      ++NumAllStaleFunctions;
    }

    if (opts::ShowDensity) {
      uint64_t Size = Function.getSize();
      // In case of BOLT split functions registered in BAT, executed traces are
      // automatically attributed to the main fragment. Add up function sizes
      // for all fragments.
      if (IsHotParentOfBOLTSplitFunction)
        for (const BinaryFunction *Fragment : Function.getFragments())
          Size += Fragment->getSize();
      double Density = (double)1.0 * Function.getSampleCountInBytes() / Size;
      FuncDensityList.emplace_back(Density, SampleCount);
      LLVM_DEBUG(BC.outs() << Function << ": executed bytes "
                           << Function.getSampleCountInBytes() << ", size (b) "
```

- EN: Declares or implements routines including `getSize`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSize`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `getSize`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSize`, `LLVM_DEBUG`。

### Lines 1517-1534

```cpp
                           << Size << ", density " << Density
                           << ", sample count " << SampleCount << '\n');
    }
  }
  BC.NumProfiledFuncs = ProfiledFunctions.size();
  BC.NumStaleProfileFuncs = NumStaleProfileFunctions;

  const size_t NumAllProfiledFunctions =
      ProfiledFunctions.size() + NumStaleProfileFunctions;
  BC.outs() << "BOLT-INFO: " << NumAllProfiledFunctions << " out of "
            << NumRegularFunctions << " functions in the binary ("
            << format("%.1f", NumAllProfiledFunctions /
                                  (float)NumRegularFunctions * 100.0f)
            << "%) have non-empty execution profile\n";
  if (NumNonSimpleProfiledFunctions) {
    BC.outs() << "BOLT-INFO: " << NumNonSimpleProfiledFunctions << " function"
              << (NumNonSimpleProfiledFunctions == 1 ? "" : "s")
              << " with profile could not be optimized\n";
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1535-1552

```cpp
  }
  if (NumAllStaleFunctions) {
    const float PctStale =
        NumAllStaleFunctions / (float)NumAllProfiledFunctions * 100.0f;
    auto printErrorOrWarning = [&]() {
      if (PctStale > opts::StaleThreshold)
        BC.errs() << "BOLT-ERROR: ";
      else
        BC.errs() << "BOLT-WARNING: ";
    };
    printErrorOrWarning();
    BC.errs() << NumAllStaleFunctions
              << format(" (%.1f%% of all profiled)", PctStale) << " function"
              << (NumAllStaleFunctions == 1 ? "" : "s")
              << " have invalid (possibly stale) profile."
                 " Use -report-stale to see the list.\n";
    if (TotalSampleCount > 0) {
      printErrorOrWarning();
```

- EN: Declares or implements routines including `printErrorOrWarning`, `format`, `invalid`. Notable symbols here include `printErrorOrWarning`, `format`, `invalid`.
- CN: 这里声明或实现函数，例如 `printErrorOrWarning`, `format`, `invalid`。这里较值得关注的符号包括 `printErrorOrWarning`, `format`, `invalid`。

### Lines 1553-1570

```cpp
      BC.errs() << (StaleSampleCount + InferredSampleCount) << " out of "
                << TotalSampleCount << " samples in the binary ("
                << format("%.1f",
                          ((100.0f * (StaleSampleCount + InferredSampleCount)) /
                           TotalSampleCount))
                << "%) belong to functions with invalid"
                   " (possibly stale) profile.\n";
    }
    if (PctStale > opts::StaleThreshold) {
      return createFatalBOLTError(
          Twine("BOLT-ERROR: stale functions exceed specified threshold of ") +
          Twine(opts::StaleThreshold.getValue()) + Twine("%. Exiting.\n"));
    }
  }
  if (NumInferredFunctions) {
    BC.outs() << format(
        "BOLT-INFO: inferred profile for %d (%.2f%% of profiled, "
        "%.2f%% of stale) functions responsible for %.2f%% samples"
```

- EN: Declares or implements routines including `Twine`. Notable symbols here include `Twine`.
- CN: 这里声明或实现函数，例如 `Twine`。这里较值得关注的符号包括 `Twine`。

### Lines 1571-1588

```cpp
        " (%zu out of %zu)\n",
        NumInferredFunctions,
        100.0 * NumInferredFunctions / NumAllProfiledFunctions,
        100.0 * NumInferredFunctions / NumAllStaleFunctions,
        100.0 * InferredSampleCount / TotalSampleCount, InferredSampleCount,
        TotalSampleCount);
    BC.outs() << format(
        "BOLT-INFO: inference found an exact match for %.2f%% of basic blocks"
        " (%zu out of %zu stale) responsible for %.2f%% samples"
        " (%zu out of %zu stale)\n",
        100.0 * BC.Stats.NumExactMatchedBlocks / BC.Stats.NumStaleBlocks,
        BC.Stats.NumExactMatchedBlocks, BC.Stats.NumStaleBlocks,
        100.0 * BC.Stats.ExactMatchedSampleCount / BC.Stats.StaleSampleCount,
        BC.Stats.ExactMatchedSampleCount, BC.Stats.StaleSampleCount);
    BC.outs() << format(
        "BOLT-INFO: inference found an exact pseudo probe match for %.2f%% of "
        "basic blocks (%zu out of %zu stale) responsible for %.2f%% samples"
        " (%zu out of %zu stale)\n",
```

- EN: Declares or implements routines including `blocks`. Notable symbols here include `blocks`.
- CN: 这里声明或实现函数，例如 `blocks`。这里较值得关注的符号包括 `blocks`。

### Lines 1589-1606

```cpp
        100.0 * BC.Stats.NumPseudoProbeExactMatchedBlocks /
            BC.Stats.NumStaleBlocks,
        BC.Stats.NumPseudoProbeExactMatchedBlocks, BC.Stats.NumStaleBlocks,
        100.0 * BC.Stats.PseudoProbeExactMatchedSampleCount /
            BC.Stats.StaleSampleCount,
        BC.Stats.PseudoProbeExactMatchedSampleCount, BC.Stats.StaleSampleCount);
    BC.outs() << format(
        "BOLT-INFO: inference found a loose pseudo probe match for %.2f%% of "
        "basic blocks (%zu out of %zu stale) responsible for %.2f%% samples"
        " (%zu out of %zu stale)\n",
        100.0 * BC.Stats.NumPseudoProbeLooseMatchedBlocks /
            BC.Stats.NumStaleBlocks,
        BC.Stats.NumPseudoProbeLooseMatchedBlocks, BC.Stats.NumStaleBlocks,
        100.0 * BC.Stats.PseudoProbeLooseMatchedSampleCount /
            BC.Stats.StaleSampleCount,
        BC.Stats.PseudoProbeLooseMatchedSampleCount, BC.Stats.StaleSampleCount);
    BC.outs() << format(
        "BOLT-INFO: inference found a call match for %.2f%% of basic "
```

- EN: Declares or implements routines including `blocks`. Notable symbols here include `blocks`.
- CN: 这里声明或实现函数，例如 `blocks`。这里较值得关注的符号包括 `blocks`。

### Lines 1607-1624

```cpp
        "blocks"
        " (%zu out of %zu stale) responsible for %.2f%% samples"
        " (%zu out of %zu stale)\n",
        100.0 * BC.Stats.NumCallMatchedBlocks / BC.Stats.NumStaleBlocks,
        BC.Stats.NumCallMatchedBlocks, BC.Stats.NumStaleBlocks,
        100.0 * BC.Stats.CallMatchedSampleCount / BC.Stats.StaleSampleCount,
        BC.Stats.CallMatchedSampleCount, BC.Stats.StaleSampleCount);
    BC.outs() << format(
        "BOLT-INFO: inference found a loose match for %.2f%% of basic "
        "blocks"
        " (%zu out of %zu stale) responsible for %.2f%% samples"
        " (%zu out of %zu stale)\n",
        100.0 * BC.Stats.NumLooseMatchedBlocks / BC.Stats.NumStaleBlocks,
        BC.Stats.NumLooseMatchedBlocks, BC.Stats.NumStaleBlocks,
        100.0 * BC.Stats.LooseMatchedSampleCount / BC.Stats.StaleSampleCount,
        BC.Stats.LooseMatchedSampleCount, BC.Stats.StaleSampleCount);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1625-1642

```cpp
  if (const uint64_t NumUnusedObjects = BC.getNumUnusedProfiledObjects()) {
    BC.outs() << "BOLT-INFO: profile for " << NumUnusedObjects
              << " objects was ignored\n";
  }

  if (ProfiledFunctions.size() > 10) {
    if (opts::Verbosity >= 1) {
      BC.outs() << "BOLT-INFO: top called functions are:\n";
      llvm::sort(ProfiledFunctions,
                 [](const BinaryFunction *A, const BinaryFunction *B) {
                   return B->getExecutionCount() < A->getExecutionCount();
                 });
      auto SFI = ProfiledFunctions.begin();
      auto SFIend = ProfiledFunctions.end();
      for (unsigned I = 0u; I < opts::TopCalledLimit && SFI != SFIend;
           ++SFI, ++I)
        BC.outs() << "  " << **SFI << " : " << (*SFI)->getExecutionCount()
                  << '\n';
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1643-1657

```cpp
    }
  }

  if (!opts::PrintSortedBy.empty()) {
    BinaryFunctionListType Functions;
    std::map<const BinaryFunction *, DynoStats> Stats;

    for (auto &BFI : BC.getBinaryFunctions()) {
      BinaryFunction &BF = BFI.second;
      if (shouldOptimize(BF) && BF.hasValidProfile()) {
        Functions.push_back(&BF);
        Stats.emplace(&BF, getDynoStats(BF));
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1658-1671

```cpp
    const bool SortAll =
        llvm::is_contained(opts::PrintSortedBy, DynoStats::LAST_DYNO_STAT);

    const bool Ascending =
        opts::DynoStatsSortOrderOpt == opts::DynoStatsSortOrder::Ascending;

    std::function<bool(const DynoStats &, const DynoStats &)>
        DynoStatsComparator =
            SortAll ? [](const DynoStats &StatsA,
                         const DynoStats &StatsB) { return StatsA < StatsB; }
                    : [](const DynoStats &StatsA, const DynoStats &StatsB) {
                        return StatsA.lessThan(StatsB, opts::PrintSortedBy);
                      };
```

- EN: Declares or implements routines including `is_contained`, `bool`. Notable symbols here include `is_contained`, `bool`.
- CN: 这里声明或实现函数，例如 `is_contained`, `bool`。这里较值得关注的符号包括 `is_contained`, `bool`。

### Lines 1672-1682

```cpp
    llvm::stable_sort(Functions,
                      [Ascending, &Stats, DynoStatsComparator](
                          const BinaryFunction *A, const BinaryFunction *B) {
                        auto StatsItr = Stats.find(A);
                        assert(StatsItr != Stats.end());
                        const DynoStats &StatsA = StatsItr->second;

                        StatsItr = Stats.find(B);
                        assert(StatsItr != Stats.end());
                        const DynoStats &StatsB = StatsItr->second;
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 1683-1700

```cpp
                        return Ascending ? DynoStatsComparator(StatsA, StatsB)
                                         : DynoStatsComparator(StatsB, StatsA);
                      });

    BC.outs() << "BOLT-INFO: top functions sorted by ";
    if (SortAll) {
      BC.outs() << "dyno stats";
    } else {
      BC.outs() << "(";
      bool PrintComma = false;
      for (const DynoStats::Category Category : opts::PrintSortedBy) {
        if (PrintComma)
          BC.outs() << ", ";
        BC.outs() << DynoStats::Description(Category);
        PrintComma = true;
      }
      BC.outs() << ")";
    }
```

- EN: Declares or implements routines including `DynoStatsComparator`. Notable symbols here include `DynoStatsComparator`.
- CN: 这里声明或实现函数，例如 `DynoStatsComparator`。这里较值得关注的符号包括 `DynoStatsComparator`。

### Lines 1701-1718

```cpp

    BC.outs() << " are:\n";
    auto SFI = Functions.begin();
    for (unsigned I = 0; I < 100 && SFI != Functions.end(); ++SFI, ++I) {
      const DynoStats Stats = getDynoStats(**SFI);
      BC.outs() << "  " << **SFI;
      if (!SortAll) {
        BC.outs() << " (";
        bool PrintComma = false;
        for (const DynoStats::Category Category : opts::PrintSortedBy) {
          if (PrintComma)
            BC.outs() << ", ";
          BC.outs() << dynoStatsOptName(Category) << "=" << Stats[Category];
          PrintComma = true;
        }
        BC.outs() << ")";
      }
      BC.outs() << "\n";
```

- EN: Declares or implements routines including `getDynoStats`. Notable symbols here include `getDynoStats`.
- CN: 这里声明或实现函数，例如 `getDynoStats`。这里较值得关注的符号包括 `getDynoStats`。

### Lines 1719-1735

```cpp
    }
  }

  if (!BC.TrappedFunctions.empty()) {
    BC.errs() << "BOLT-WARNING: " << BC.TrappedFunctions.size() << " function"
              << (BC.TrappedFunctions.size() > 1 ? "s" : "")
              << " will trap on entry. Use -trap-avx512=0 to disable"
                 " traps.";
    if (opts::Verbosity >= 1 || BC.TrappedFunctions.size() <= 5) {
      BC.errs() << '\n';
      for (const BinaryFunction *Function : BC.TrappedFunctions)
        BC.errs() << "  " << *Function << '\n';
    } else {
      BC.errs() << " Use -v=1 to see the list.\n";
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1736-1743

```cpp
  // Collect and print information about suboptimal code layout on input.
  if (opts::ReportBadLayout) {
    BinaryFunctionListType SuboptimalFuncs;
    for (auto &BFI : BC.getBinaryFunctions()) {
      BinaryFunction &BF = BFI.second;
      if (!BF.hasValidProfile())
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 1744-1758

```cpp
      const uint64_t HotThreshold =
          std::max<uint64_t>(BF.getKnownExecutionCount(), 1);
      bool HotSeen = false;
      for (const BinaryBasicBlock *BB : BF.getLayout().rblocks()) {
        if (!HotSeen && BB->getKnownExecutionCount() > HotThreshold) {
          HotSeen = true;
          continue;
        }
        if (HotSeen && BB->getKnownExecutionCount() == 0) {
          SuboptimalFuncs.push_back(&BF);
          break;
        }
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1759-1776

```cpp
    if (!SuboptimalFuncs.empty()) {
      llvm::sort(SuboptimalFuncs,
                 [](const BinaryFunction *A, const BinaryFunction *B) {
                   return A->getKnownExecutionCount() / A->getSize() >
                          B->getKnownExecutionCount() / B->getSize();
                 });

      BC.outs() << "BOLT-INFO: " << SuboptimalFuncs.size()
                << " functions have "
                   "cold code in the middle of hot code. Top functions are:\n";
      for (unsigned I = 0;
           I < std::min(static_cast<size_t>(opts::ReportBadLayout),
                        SuboptimalFuncs.size());
           ++I)
        SuboptimalFuncs[I]->print(BC.outs());
    }
  }
```

- EN: Declares or implements routines including `getKnownExecutionCount`, `min`, `print`. Notable symbols here include `getKnownExecutionCount`, `min`, `print`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`, `min`, `print`。这里较值得关注的符号包括 `getKnownExecutionCount`, `min`, `print`。

### Lines 1777-1784

```cpp
  if (NumUnknownControlFlowFunctions) {
    BC.outs() << "BOLT-INFO: " << NumUnknownControlFlowFunctions
              << " functions have instructions with unknown control flow";
    if (!opts::PrintUnknown)
      BC.outs() << ". Use -print-unknown to see the list.";
    BC.outs() << '\n';
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1785-1802

```cpp
  if (opts::ShowDensity) {
    double Density = 0.0;
    llvm::sort(FuncDensityList);

    uint64_t AccumulatedSamples = 0;
    assert(opts::ProfileDensityCutOffHot <= 1000000 &&
           "The cutoff value is greater than 1000000(100%)");
    // Subtract samples in zero-density functions (no fall-throughs) from
    // TotalSampleCount (not used anywhere below).
    for (const auto [CurDensity, CurSamples] : FuncDensityList) {
      if (CurDensity != 0.0)
        break;
      TotalSampleCount -= CurSamples;
    }
    const uint64_t CutoffSampleCount =
        1.f * TotalSampleCount * opts::ProfileDensityCutOffHot / 1000000;
    // Process functions in decreasing density order
    for (const auto [CurDensity, CurSamples] : llvm::reverse(FuncDensityList)) {
```

- EN: Declares or implements routines including `sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `sort`.
- CN: 这里声明或实现函数，例如 `sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `sort`。

### Lines 1803-1819

```cpp
      if (AccumulatedSamples >= CutoffSampleCount)
        break;
      AccumulatedSamples += CurSamples;
      Density = CurDensity;
    }
    if (Density == 0.0) {
      BC.errs() << "BOLT-WARNING: the output profile is empty or the "
                   "--profile-density-cutoff-hot option is "
                   "set too low. Please check your command.\n";
    } else if (Density < opts::ProfileDensityThreshold) {
      BC.errs()
          << "BOLT-WARNING: BOLT is estimated to optimize better with "
          << format("%.1f", opts::ProfileDensityThreshold / Density)
          << "x more samples. Please consider increasing sampling rate or "
             "profiling for longer duration to get more samples.\n";
    }
```

- EN: Declares or implements routines including `if`, `format`. Notable symbols here include `if`, `format`.
- CN: 这里声明或实现函数，例如 `if`, `format`。这里较值得关注的符号包括 `if`, `format`。

### Lines 1820-1829

```cpp
    BC.outs() << "BOLT-INFO: Functions with density >= "
              << format("%.1f", Density) << " account for "
              << format("%.2f",
                        static_cast<double>(opts::ProfileDensityCutOffHot) /
                            10000)
              << "% total sample counts.\n";
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 1830-1837

```cpp
Error InstructionLowering::runOnFunctions(BinaryContext &BC) {
  for (auto &BFI : BC.getBinaryFunctions())
    for (BinaryBasicBlock &BB : BFI.second)
      for (MCInst &Instruction : BB)
        BC.MIB->lowerTailCall(Instruction);
  return Error::success();
}
```

- EN: Declares or implements routines including `runOnFunctions`, `lowerTailCall`. Notable symbols here include `runOnFunctions`, `lowerTailCall`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `lowerTailCall`。这里较值得关注的符号包括 `runOnFunctions`, `lowerTailCall`。

### Lines 1838-1850

```cpp
Error StripRepRet::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86())
    return Error::success();

  uint64_t NumPrefixesRemoved = 0;
  uint64_t NumBytesSaved = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    for (BinaryBasicBlock &BB : BFI.second) {
      auto LastInstRIter = BB.getLastNonPseudo();
      if (LastInstRIter == BB.rend() || !BC.MIB->isReturn(*LastInstRIter) ||
          !BC.MIB->deleteREPPrefix(*LastInstRIter))
        continue;
```

- EN: Declares or implements routines including `runOnFunctions`, `deleteREPPrefix`. Notable symbols here include `runOnFunctions`, `deleteREPPrefix`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `deleteREPPrefix`。这里较值得关注的符号包括 `runOnFunctions`, `deleteREPPrefix`。

### Lines 1851-1863

```cpp
      NumPrefixesRemoved += BB.getKnownExecutionCount();
      ++NumBytesSaved;
    }
  }

  if (NumBytesSaved)
    BC.outs() << "BOLT-INFO: removed " << NumBytesSaved
              << " 'repz' prefixes"
                 " with estimated execution count of "
              << NumPrefixesRemoved << " times.\n";
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1864-1874

```cpp
Error InlineMemcpy::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86() && !BC.isAArch64())
    return Error::success();

  uint64_t NumInlined = 0;
  uint64_t NumInlinedDyno = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    for (BinaryBasicBlock &BB : BFI.second) {
      for (auto II = BB.begin(); II != BB.end(); ++II) {
        MCInst &Inst = *II;
```

- EN: Declares or implements routines including `runOnFunctions`. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 1875-1884

```cpp
        if (!BC.MIB->isCall(Inst) || MCPlus::getNumPrimeOperands(Inst) != 1 ||
            !Inst.getOperand(0).isExpr())
          continue;

        const MCSymbol *CalleeSymbol = BC.MIB->getTargetSymbol(Inst);
        if (CalleeSymbol->getName() != "memcpy" &&
            CalleeSymbol->getName() != "memcpy@PLT" &&
            CalleeSymbol->getName() != "_memcpy8")
          continue;
```

- EN: Declares or implements routines including `getTargetSymbol`, `getName`. Notable symbols here include `getTargetSymbol`, `getName`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `getName`。这里较值得关注的符号包括 `getTargetSymbol`, `getName`。

### Lines 1885-1892

```cpp
        const bool IsMemcpy8 = (CalleeSymbol->getName() == "_memcpy8");
        const bool IsTailCall = BC.MIB->isTailCall(Inst);

        // Extract size from preceding instructions (AArch64 only).
        // Pattern: MOV X2, #nb-bytes; BL memcpy src, dest, X2.
        std::optional<uint64_t> KnownSize =
            BC.MIB->findMemcpySizeInBytes(BB, II);
```

- EN: Declares or implements routines including `isTailCall`, `findMemcpySizeInBytes`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isTailCall`, `findMemcpySizeInBytes`.
- CN: 这里声明或实现函数，例如 `isTailCall`, `findMemcpySizeInBytes`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isTailCall`, `findMemcpySizeInBytes`。

### Lines 1893-1905

```cpp
        if (BC.isAArch64() && (!KnownSize.has_value() || *KnownSize > 64))
          continue;

        const InstructionListType NewCode =
            BC.MIB->createInlineMemcpy(IsMemcpy8, KnownSize);
        II = BB.replaceInstruction(II, NewCode);
        std::advance(II, NewCode.size() - 1);
        if (IsTailCall) {
          MCInst Return;
          BC.MIB->createReturn(Return);
          II = BB.insertInstruction(std::next(II), std::move(Return));
        }
```

- EN: Declares or implements routines including `createInlineMemcpy`, `advance`, `createReturn`. Notable symbols here include `createInlineMemcpy`, `advance`, `createReturn`.
- CN: 这里声明或实现函数，例如 `createInlineMemcpy`, `advance`, `createReturn`。这里较值得关注的符号包括 `createInlineMemcpy`, `advance`, `createReturn`。

### Lines 1906-1921

```cpp
        ++NumInlined;
        NumInlinedDyno += BB.getKnownExecutionCount();
      }
    }
  }

  if (NumInlined) {
    BC.outs() << "BOLT-INFO: inlined " << NumInlined << " memcpy() calls";
    if (NumInlinedDyno)
      BC.outs() << ". The calls were executed " << NumInlinedDyno
                << " times based on profile.";
    BC.outs() << '\n';
  }
  return Error::success();
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1922-1931

```cpp
bool SpecializeMemcpy1::shouldOptimize(const BinaryFunction &Function) const {
  if (!BinaryFunctionPass::shouldOptimize(Function))
    return false;

  for (const std::string &FunctionSpec : Spec) {
    StringRef FunctionName = StringRef(FunctionSpec).split(':').first;
    if (Function.hasNameRegex(FunctionName))
      return true;
  }
```

- EN: Declares or implements routines including `shouldOptimize`, `StringRef`. Notable symbols here include `shouldOptimize`, `StringRef`.
- CN: 这里声明或实现函数，例如 `shouldOptimize`, `StringRef`。这里较值得关注的符号包括 `shouldOptimize`, `StringRef`。

### Lines 1932-1945

```cpp
  return false;
}

std::set<size_t> SpecializeMemcpy1::getCallSitesToOptimize(
    const BinaryFunction &Function) const {
  StringRef SitesString;
  for (const std::string &FunctionSpec : Spec) {
    StringRef FunctionName;
    std::tie(FunctionName, SitesString) = StringRef(FunctionSpec).split(':');
    if (Function.hasNameRegex(FunctionName))
      break;
    SitesString = "";
  }
```

- EN: Declares or implements routines including `tie`. Notable symbols here include `tie`.
- CN: 这里声明或实现函数，例如 `tie`。这里较值得关注的符号包括 `tie`。

### Lines 1946-1956

```cpp
  std::set<size_t> Sites;
  SmallVector<StringRef, 4> SitesVec;
  SitesString.split(SitesVec, ':');
  for (StringRef SiteString : SitesVec) {
    if (SiteString.empty())
      continue;
    size_t Result;
    if (!SiteString.getAsInteger(10, Result))
      Sites.emplace(Result);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 1957-1966

```cpp
  return Sites;
}

Error SpecializeMemcpy1::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName()
              << " is currently supported only on X86\n";
    exit(1);
  }
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`. Notable symbols here include `runOnFunctions`, `exit`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`。这里较值得关注的符号包括 `runOnFunctions`, `exit`。

### Lines 1967-1978

```cpp
  uint64_t NumSpecialized = 0;
  uint64_t NumSpecializedDyno = 0;
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &Function = BFI.second;
    if (!shouldOptimize(Function))
      continue;

    std::set<size_t> CallsToOptimize = getCallSitesToOptimize(Function);
    auto shouldOptimize = [&](size_t N) {
      return CallsToOptimize.empty() || CallsToOptimize.count(N);
    };
```

- EN: Declares or implements routines including `getCallSitesToOptimize`. Notable symbols here include `getCallSitesToOptimize`.
- CN: 这里声明或实现函数，例如 `getCallSitesToOptimize`。这里较值得关注的符号包括 `getCallSitesToOptimize`。

### Lines 1979-1988

```cpp
    std::vector<BinaryBasicBlock *> Blocks(Function.pbegin(), Function.pend());
    size_t CallSiteID = 0;
    for (BinaryBasicBlock *CurBB : Blocks) {
      for (auto II = CurBB->begin(); II != CurBB->end(); ++II) {
        MCInst &Inst = *II;

        if (!BC.MIB->isCall(Inst) || MCPlus::getNumPrimeOperands(Inst) != 1 ||
            !Inst.getOperand(0).isExpr())
          continue;
```

- EN: Declares or implements routines including `Blocks`. Notable symbols here include `Blocks`.
- CN: 这里声明或实现函数，例如 `Blocks`。这里较值得关注的符号包括 `Blocks`。

### Lines 1989-1996

```cpp
        const MCSymbol *CalleeSymbol = BC.MIB->getTargetSymbol(Inst);
        if (CalleeSymbol->getName() != "memcpy" &&
            CalleeSymbol->getName() != "memcpy@PLT")
          continue;

        if (BC.MIB->isTailCall(Inst))
          continue;
```

- EN: Declares or implements routines including `getTargetSymbol`, `getName`. Notable symbols here include `getTargetSymbol`, `getName`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `getName`。这里较值得关注的符号包括 `getTargetSymbol`, `getName`。

### Lines 1997-2004

```cpp
        ++CallSiteID;

        if (!shouldOptimize(CallSiteID))
          continue;

        // Create a copy of a call to memcpy(dest, src, size).
        MCInst MemcpyInstr = Inst;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 2005-2016

```cpp
        BinaryBasicBlock *OneByteMemcpyBB = CurBB->splitAt(II);

        BinaryBasicBlock *NextBB = nullptr;
        if (OneByteMemcpyBB->getNumNonPseudos() > 1) {
          NextBB = OneByteMemcpyBB->splitAt(OneByteMemcpyBB->begin());
          NextBB->eraseInstruction(NextBB->begin());
        } else {
          NextBB = OneByteMemcpyBB->getSuccessor();
          OneByteMemcpyBB->eraseInstruction(OneByteMemcpyBB->begin());
          assert(NextBB && "unexpected call to memcpy() with no return");
        }
```

- EN: Declares or implements routines including `splitAt`, `eraseInstruction`, `getSuccessor`, `assert`. Notable symbols here include `splitAt`, `eraseInstruction`, `getSuccessor`, `assert`.
- CN: 这里声明或实现函数，例如 `splitAt`, `eraseInstruction`, `getSuccessor`, `assert`。这里较值得关注的符号包括 `splitAt`, `eraseInstruction`, `getSuccessor`, `assert`。

### Lines 2017-2024

```cpp
        BinaryBasicBlock *MemcpyBB = Function.addBasicBlock();
        MemcpyBB->setOffset(CurBB->getInputOffset());
        InstructionListType CmpJCC =
            BC.MIB->createCmpJE(BC.MIB->getIntArgRegister(2), 1,
                                OneByteMemcpyBB->getLabel(), BC.Ctx.get());
        CurBB->addInstructions(CmpJCC);
        CurBB->addSuccessor(MemcpyBB);
```

- EN: Declares or implements routines including `setOffset`, `createCmpJE`, `getLabel`, `addInstructions`, `addSuccessor`. Notable symbols here include `setOffset`, `createCmpJE`, `getLabel`, `addInstructions`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `setOffset`, `createCmpJE`, `getLabel`, `addInstructions`, `addSuccessor`。这里较值得关注的符号包括 `setOffset`, `createCmpJE`, `getLabel`, `addInstructions`, `addSuccessor`。

### Lines 2025-2034

```cpp
        MemcpyBB->addInstruction(std::move(MemcpyInstr));
        MemcpyBB->addSuccessor(NextBB);
        MemcpyBB->setCFIState(NextBB->getCFIState());
        MemcpyBB->setExecutionCount(0);

        // To prevent the actual call from being moved to cold, we set its
        // execution count to 1.
        if (CurBB->getKnownExecutionCount() > 0)
          MemcpyBB->setExecutionCount(1);
```

- EN: Declares or implements routines including `addInstruction`, `addSuccessor`, `setCFIState`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addInstruction`, `addSuccessor`, `setCFIState`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `addInstruction`, `addSuccessor`, `setCFIState`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addInstruction`, `addSuccessor`, `setCFIState`, `setExecutionCount`。

### Lines 2035-2042

```cpp
        InstructionListType OneByteMemcpy = BC.MIB->createOneByteMemcpy();
        OneByteMemcpyBB->addInstructions(OneByteMemcpy);

        ++NumSpecialized;
        NumSpecializedDyno += CurBB->getKnownExecutionCount();

        CurBB = NextBB;
```

- EN: Declares or implements routines including `createOneByteMemcpy`, `addInstructions`, `getKnownExecutionCount`. Notable symbols here include `createOneByteMemcpy`, `addInstructions`, `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `createOneByteMemcpy`, `addInstructions`, `getKnownExecutionCount`。这里较值得关注的符号包括 `createOneByteMemcpy`, `addInstructions`, `getKnownExecutionCount`。

### Lines 2043-2059

```cpp
        // Note: we don't expect the next instruction to be a call to memcpy.
        II = CurBB->begin();
      }
    }
  }

  if (NumSpecialized) {
    BC.outs() << "BOLT-INFO: specialized " << NumSpecialized
              << " memcpy() call sites for size 1";
    if (NumSpecializedDyno)
      BC.outs() << ". The calls were executed " << NumSpecializedDyno
                << " times based on profile.";
    BC.outs() << '\n';
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `begin`, `memcpy`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`, `memcpy`.
- CN: 这里声明或实现函数，例如 `begin`, `memcpy`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`, `memcpy`。

### Lines 2060-2070

```cpp
void RemoveNops::runOnFunction(BinaryFunction &BF) {
  const BinaryContext &BC = BF.getBinaryContext();
  for (BinaryBasicBlock &BB : BF) {
    for (int64_t I = BB.size() - 1; I >= 0; --I) {
      MCInst &Inst = BB.getInstructionAtIndex(I);
      if (BC.MIB->isNoop(Inst) && BC.MIB->hasAnnotation(Inst, "NOP"))
        BB.eraseInstructionAtIndex(I);
    }
  }
}
```

- EN: Declares or implements routines including `runOnFunction`. Notable symbols here include `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunction`。这里较值得关注的符号包括 `runOnFunction`。

### Lines 2071-2079

```cpp
Error RemoveNops::runOnFunctions(BinaryContext &BC) {
  ParallelUtilities::WorkFuncTy WorkFun = [&](BinaryFunction &BF) {
    runOnFunction(BF);
  };

  ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
    return BF.shouldPreserveNops();
  };
```

- EN: Declares or implements routines including `runOnFunctions`, `runOnFunction`. Notable symbols here include `runOnFunctions`, `runOnFunction`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `runOnFunction`。这里较值得关注的符号包括 `runOnFunctions`, `runOnFunction`。

### Lines 2080-2087

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, WorkFun,
      SkipFunc, "RemoveNops");
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `DynoStatsSortOrder`: enumeration of modes or states / 模式或状态枚举
- `SctcModes`: enumeration of modes or states / 模式或状态枚举
- `dynoStatsOptName`: function or method entry point / 函数或方法入口
- `Description`: function or method entry point / 函数或方法入口
- `replace`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/BinaryPasses.h`, `bolt/Core/FunctionLayout.h`, `bolt/Core/ParallelUtilities.h`, `bolt/Passes/ReorderAlgorithm.h`, `bolt/Passes/ReorderFunctions.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- System headers / 系统头文件: `atomic`, `mutex`, `numeric`, `vector`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
