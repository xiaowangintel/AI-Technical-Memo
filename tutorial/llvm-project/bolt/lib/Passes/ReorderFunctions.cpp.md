# ReorderFunctions.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/ReorderFunctions.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Function reordering pass. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Function reordering pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/ReorderFunctions.cpp - Function reordering pass --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements ReorderFunctions class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Passes/ReorderFunctions.h"
#include "bolt/Passes/HFSort.h"
#include "bolt/Utils/Utils.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/CodeLayout.h"
#include <fstream>
```

- EN: Pulls in 8 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 8 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-31

```cpp
#define DEBUG_TYPE "hfsort"

using namespace llvm;

namespace opts {

extern cl::OptionCategory BoltOptCategory;
extern cl::opt<unsigned> Verbosity;
extern cl::opt<uint32_t> RandomSeed;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 32-49

```cpp
extern size_t padFunctionBefore(const bolt::BinaryFunction &Function);
extern size_t padFunctionAfter(const bolt::BinaryFunction &Function);

extern cl::opt<bolt::ReorderFunctions::ReorderType> ReorderFunctions;
cl::opt<bolt::ReorderFunctions::ReorderType> ReorderFunctions(
    "reorder-functions",
    cl::desc("reorder and cluster functions (works only with relocations)"),
    cl::init(bolt::ReorderFunctions::RT_NONE),
    cl::values(clEnumValN(bolt::ReorderFunctions::RT_NONE, "none",
                          "do not reorder functions"),
               clEnumValN(bolt::ReorderFunctions::RT_EXEC_COUNT, "exec-count",
                          "order by execution count"),
               clEnumValN(bolt::ReorderFunctions::RT_HFSORT, "hfsort",
                          "use hfsort algorithm"),
               clEnumValN(bolt::ReorderFunctions::RT_HFSORT_PLUS, "hfsort+",
                          "use cache-directed sort"),
               clEnumValN(bolt::ReorderFunctions::RT_CDSORT, "cdsort",
                          "use cache-directed sort"),
```

- EN: Declares or implements routines including `padFunctionBefore`, `padFunctionAfter`, `desc`, `init`. Notable symbols here include `padFunctionBefore`, `padFunctionAfter`, `desc`, `init`.
- CN: 这里声明或实现函数，例如 `padFunctionBefore`, `padFunctionAfter`, `desc`, `init`。这里较值得关注的符号包括 `padFunctionBefore`, `padFunctionAfter`, `desc`, `init`。

### Lines 50-64

```cpp
               clEnumValN(bolt::ReorderFunctions::RT_PETTIS_HANSEN,
                          "pettis-hansen", "use Pettis-Hansen algorithm"),
               clEnumValN(bolt::ReorderFunctions::RT_RANDOM, "random",
                          "reorder functions randomly"),
               clEnumValN(bolt::ReorderFunctions::RT_USER, "user",
                          "use function order specified by -function-order")),
    cl::ZeroOrMore, cl::cat(BoltOptCategory),
    cl::callback([](const bolt::ReorderFunctions::ReorderType &option) {
      if (option == bolt::ReorderFunctions::RT_HFSORT_PLUS) {
        errs() << "BOLT-WARNING: '-reorder-functions=hfsort+' is deprecated,"
               << " please use '-reorder-functions=cdsort' instead\n";
        ReorderFunctions = bolt::ReorderFunctions::RT_CDSORT;
      }
    }));
```

- EN: Declares or implements routines including `cat`, `callback`, `errs`. Notable symbols here include `cat`, `callback`, `errs`.
- CN: 这里声明或实现函数，例如 `cat`, `callback`, `errs`。这里较值得关注的符号包括 `cat`, `callback`, `errs`。

### Lines 65-75

```cpp
static cl::opt<bool> ReorderFunctionsUseHotSize(
    "reorder-functions-use-hot-size",
    cl::desc("use a function's hot size when doing clustering"), cl::init(true),
    cl::cat(BoltOptCategory));

static cl::opt<std::string> FunctionOrderFile(
    "function-order",
    cl::desc("file containing an ordered list of functions to use for function "
             "reordering; can be combined with --reorder-functions algorithms"),
    cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`。

### Lines 76-87

```cpp
static cl::opt<std::string> GenerateFunctionOrderFile(
    "generate-function-order",
    cl::desc("file to dump the ordered list of functions to use for function "
             "reordering"),
    cl::cat(BoltOptCategory));

static cl::opt<std::string> LinkSectionsFile(
    "generate-link-sections",
    cl::desc("generate a list of function sections in a format suitable for "
             "inclusion in a linker script"),
    cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`. Notable symbols here include `cat`.
- CN: 这里声明或实现函数，例如 `cat`。这里较值得关注的符号包括 `cat`。

### Lines 88-98

```cpp
static cl::opt<bool>
    UseEdgeCounts("use-edge-counts",
                  cl::desc("use edge count data when doing clustering"),
                  cl::init(true), cl::cat(BoltOptCategory));

static cl::opt<bool> CgFromPerfData(
    "cg-from-perf-data",
    cl::desc("use perf data directly when constructing the call graph"
             " for stale functions"),
    cl::init(true), cl::ZeroOrMore, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 99-109

```cpp
static cl::opt<bool> CgIgnoreRecursiveCalls(
    "cg-ignore-recursive-calls",
    cl::desc("ignore recursive calls when constructing the call graph"),
    cl::init(true), cl::cat(BoltOptCategory));

static cl::opt<bool> CgUseSplitHotSize(
    "cg-use-split-hot-size",
    cl::desc("use hot/cold data on basic blocks to determine hot sizes for "
             "call graph functions"),
    cl::init(false), cl::ZeroOrMore, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 110-118

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

using NodeId = CallGraph::NodeId;
using Arc = CallGraph::Arc;
using Node = CallGraph::Node;
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 119-126

```cpp
void ReorderFunctions::reorder(BinaryContext &BC,
                               std::vector<Cluster> &&Clusters,
                               std::map<uint64_t, BinaryFunction> &BFs,
                               uint32_t StartIndex) {
  std::vector<uint64_t> FuncAddr(Cg.numNodes()); // Just for computing stats
  uint64_t TotalSize = 0;
  uint32_t Index = StartIndex;
```

- EN: Declares or implements routines including `FuncAddr`. Notable symbols here include `FuncAddr`.
- CN: 这里声明或实现函数，例如 `FuncAddr`。这里较值得关注的符号包括 `FuncAddr`。

### Lines 127-135

```cpp
  // Set order of hot functions based on clusters.
  for (const Cluster &Cluster : Clusters) {
    for (const NodeId FuncId : Cluster.targets()) {
      Cg.nodeIdToFunc(FuncId)->setIndex(Index++);
      FuncAddr[FuncId] = TotalSize;
      TotalSize += Cg.size(FuncId);
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 136-145

```cpp
  // Assign valid index for functions with valid profile.
  for (auto &It : BFs) {
    BinaryFunction &BF = It.second;
    if (!BF.hasValidIndex() && BF.hasValidProfile())
      BF.setIndex(Index++);
  }

  if (opts::ReorderFunctions == RT_NONE)
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 146-160

```cpp
  printStats(BC, Clusters, FuncAddr);
}

void ReorderFunctions::printStats(BinaryContext &BC,
                                  const std::vector<Cluster> &Clusters,
                                  const std::vector<uint64_t> &FuncAddr) {
  if (opts::Verbosity == 0) {
#ifndef NDEBUG
    if (!DebugFlag || !isCurrentDebugType("hfsort"))
      return;
#else
    return;
#endif
  }
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Declares or implements routines including `printStats`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `printStats`.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里声明或实现函数，例如 `printStats`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `printStats`。

### Lines 161-178

```cpp
  bool PrintDetailed = opts::Verbosity > 1;
#ifndef NDEBUG
  PrintDetailed |=
      (DebugFlag && isCurrentDebugType("hfsort") && opts::Verbosity > 0);
#endif
  uint64_t TotalSize = 0;
  uint64_t CurPage = 0;
  uint64_t Hotfuncs = 0;
  double TotalDistance = 0;
  double TotalCalls = 0;
  double TotalCalls64B = 0;
  double TotalCalls4KB = 0;
  double TotalCalls2MB = 0;
  if (PrintDetailed)
    BC.outs() << "BOLT-INFO: Function reordering page layout\n"
              << "BOLT-INFO: ============== page 0 ==============\n";
  for (const Cluster &Cluster : Clusters) {
    if (PrintDetailed)
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 179-186

```cpp
      BC.outs() << format(
          "BOLT-INFO: -------- density = %.3lf (%u / %u) --------\n",
          Cluster.density(), Cluster.samples(), Cluster.size());

    for (NodeId FuncId : Cluster.targets()) {
      if (Cg.samples(FuncId) > 0) {
        Hotfuncs++;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 187-204

```cpp
        if (PrintDetailed)
          BC.outs() << "BOLT-INFO: hot func " << *Cg.nodeIdToFunc(FuncId)
                    << " (" << Cg.size(FuncId) << ")\n";

        uint64_t Dist = 0;
        uint64_t Calls = 0;
        for (NodeId Dst : Cg.successors(FuncId)) {
          if (FuncId == Dst) // ignore recursive calls in stats
            continue;
          const Arc &Arc = *Cg.findArc(FuncId, Dst);
          const auto D = std::abs(FuncAddr[Arc.dst()] -
                                  (FuncAddr[FuncId] + Arc.avgCallOffset()));
          const double W = Arc.weight();
          if (D < 64 && PrintDetailed && opts::Verbosity > 2)
            BC.outs() << "BOLT-INFO: short (" << D << "B) call:\n"
                      << "BOLT-INFO:   Src: " << *Cg.nodeIdToFunc(FuncId)
                      << "\n"
                      << "BOLT-INFO:   Dst: " << *Cg.nodeIdToFunc(Dst) << "\n"
```

- EN: Declares or implements routines including `abs`. Notable symbols here include `abs`.
- CN: 这里声明或实现函数，例如 `abs`。这里较值得关注的符号包括 `abs`。

### Lines 205-222

```cpp
                      << "BOLT-INFO:   Weight = " << W << "\n"
                      << "BOLT-INFO:   AvgOffset = " << Arc.avgCallOffset()
                      << "\n";
          Calls += W;
          if (D < 64)
            TotalCalls64B += W;
          if (D < 4096)
            TotalCalls4KB += W;
          if (D < (2 << 20))
            TotalCalls2MB += W;
          Dist += Arc.weight() * D;
          if (PrintDetailed)
            BC.outs() << format("BOLT-INFO: arc: %u [@%lu+%.1lf] -> %u [@%lu]: "
                                "weight = %.0lf, callDist = %f\n",
                                Arc.src(), FuncAddr[Arc.src()],
                                Arc.avgCallOffset(), Arc.dst(),
                                FuncAddr[Arc.dst()], Arc.weight(), D);
        }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 223-240

```cpp
        TotalCalls += Calls;
        TotalDistance += Dist;
        TotalSize += Cg.size(FuncId);

        if (PrintDetailed) {
          BC.outs() << format("BOLT-INFO: start = %6u : avgCallDist = %lu : ",
                              TotalSize, Calls ? Dist / Calls : 0)
                    << Cg.nodeIdToFunc(FuncId)->getPrintName() << '\n';
          const uint64_t NewPage = TotalSize / HugePageSize;
          if (NewPage != CurPage) {
            CurPage = NewPage;
            BC.outs() << format(
                "BOLT-INFO: ============== page %u ==============\n", CurPage);
          }
        }
      }
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 241-258

```cpp
  BC.outs() << "BOLT-INFO: Function reordering stats\n"
            << format("BOLT-INFO:  Number of hot functions: %u\n"
                      "BOLT-INFO:  Number of clusters: %lu\n",
                      Hotfuncs, Clusters.size())
            << format("BOLT-INFO:  Final average call distance = %.1lf "
                      "(%.0lf / %.0lf)\n",
                      TotalCalls ? TotalDistance / TotalCalls : 0,
                      TotalDistance, TotalCalls)
            << format("BOLT-INFO:  Total Calls = %.0lf\n", TotalCalls);
  if (TotalCalls)
    BC.outs()
        << format("BOLT-INFO:  Total Calls within 64B = %.0lf (%.2lf%%)\n",
                  TotalCalls64B, 100 * TotalCalls64B / TotalCalls)
        << format("BOLT-INFO:  Total Calls within 4KB = %.0lf (%.2lf%%)\n",
                  TotalCalls4KB, 100 * TotalCalls4KB / TotalCalls)
        << format("BOLT-INFO:  Total Calls within 2MB = %.0lf (%.2lf%%)\n",
                  TotalCalls2MB, 100 * TotalCalls2MB / TotalCalls);
}
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 259-267

```cpp

Error ReorderFunctions::readFunctionOrderFile(
    std::vector<std::string> &FunctionNames) {
  std::ifstream FuncsFile(opts::FunctionOrderFile, std::ios::in);
  if (!FuncsFile)
    return createFatalBOLTError(Twine("Ordered functions file \"") +
                                Twine(opts::FunctionOrderFile) +
                                Twine("\" can't be opened."));
```

- EN: Declares or implements routines including `FuncsFile`, `Twine`. Notable symbols here include `FuncsFile`, `Twine`.
- CN: 这里声明或实现函数，例如 `FuncsFile`, `Twine`。这里较值得关注的符号包括 `FuncsFile`, `Twine`。

### Lines 268-283

```cpp
  std::string FuncName;
  while (std::getline(FuncsFile, FuncName))
    FunctionNames.push_back(FuncName);
  return Error::success();
}

Expected<uint32_t> ReorderFunctions::assignFunctionOrder(
    BinaryContext &BC, std::map<uint64_t, BinaryFunction> &BFs,
    uint32_t StartIndex, DenseSet<const BinaryFunction *> *OrderedFuncs) {
  // Build LTOCommonNameMap for name resolution.
  StringMap<std::vector<uint64_t>> LTOCommonNameMap;
  for (const BinaryFunction &BF : llvm::make_second_range(BFs))
    for (StringRef Name : BF.getNames())
      if (std::optional<StringRef> LTOCommonName = getLTOCommonName(Name))
        LTOCommonNameMap[*LTOCommonName].push_back(BF.getAddress());
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 284-292

```cpp
  uint32_t Index = StartIndex;
  uint32_t InvalidEntries = 0;
  std::vector<std::string> FunctionNames;
  if (Error E = readFunctionOrderFile(FunctionNames))
    return std::move(E);

  for (const std::string &Function : FunctionNames) {
    std::vector<uint64_t> FuncAddrs;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 293-310

```cpp
    BinaryData *BD = BC.getBinaryDataByName(Function);
    if (!BD) {
      // If we can't find the main symbol name, look for alternates.
      uint32_t LocalID = 1;
      while (true) {
        const std::string FuncName = Function + "/" + std::to_string(LocalID);
        BD = BC.getBinaryDataByName(FuncName);
        if (BD)
          FuncAddrs.push_back(BD->getAddress());
        else
          break;
        LocalID++;
      }
      // Strip LTO suffixes.
      if (std::optional<StringRef> CommonName = getLTOCommonName(Function))
        if (LTOCommonNameMap.contains(*CommonName))
          llvm::append_range(FuncAddrs, LTOCommonNameMap[*CommonName]);
    } else {
```

- EN: Declares or implements routines including `to_string`, `append_range`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `to_string`, `append_range`.
- CN: 这里声明或实现函数，例如 `to_string`, `append_range`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `to_string`, `append_range`。

### Lines 311-321

```cpp
      FuncAddrs.push_back(BD->getAddress());
    }

    if (FuncAddrs.empty()) {
      if (opts::Verbosity >= 1)
        BC.errs() << "BOLT-WARNING: Reorder functions: can't find function "
                  << "for " << Function << "\n";
      ++InvalidEntries;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 322-339

```cpp
    for (const uint64_t FuncAddr : FuncAddrs) {
      const BinaryData *FuncBD = BC.getBinaryDataAtAddress(FuncAddr);
      assert(FuncBD);

      BinaryFunction *BF = BC.getFunctionForSymbol(FuncBD->getSymbol());
      if (!BF) {
        if (opts::Verbosity >= 1)
          BC.errs() << "BOLT-WARNING: Reorder functions: can't find function "
                    << "for " << Function << "\n";
        ++InvalidEntries;
        break;
      }
      if (!BF->hasValidIndex()) {
        BF->setIndex(Index++);
        if (OrderedFuncs)
          OrderedFuncs->insert(BF);
      } else if (opts::Verbosity > 0) {
        BC.errs() << "BOLT-WARNING: Duplicate reorder entry for " << Function
```

- EN: Declares or implements routines including `assert`, `setIndex`, `insert`, `if`. Notable symbols here include `assert`, `setIndex`, `insert`, `if`.
- CN: 这里声明或实现函数，例如 `assert`, `setIndex`, `insert`, `if`。这里较值得关注的符号包括 `assert`, `setIndex`, `insert`, `if`。

### Lines 340-347

```cpp
                  << "\n";
      }
    }
  }
  if (InvalidEntries)
    BC.errs() << "BOLT-WARNING: Reorder functions: can't find functions for "
              << InvalidEntries << " entries in -function-order list\n";
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 348-365

```cpp
  return Index;
}

Error ReorderFunctions::runOnFunctions(BinaryContext &BC) {
  auto &BFs = BC.getBinaryFunctions();

  // If a function order file is provided but no reorder algorithm was
  // explicitly specified, default to RT_USER.
  if (!opts::FunctionOrderFile.empty()) {
    if (opts::ReorderFunctions.getNumOccurrences() == 0) {
      opts::ReorderFunctions = RT_USER;
      BC.outs()
          << "BOLT-INFO: --function-order specified without "
          << "--reorder-functions, defaulting to --reorder-functions=user\n";
    } else if (opts::ReorderFunctions == RT_NONE) {
      BC.errs() << "BOLT-WARNING: --reorder-functions=none is incompatible "
                << "with --function-order, resetting to "
                << "--reorder-functions=user\n";
```

- EN: Declares or implements routines including `runOnFunctions`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`, `if`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`, `if`。

### Lines 366-383

```cpp
      opts::ReorderFunctions = RT_USER;
    } else if (opts::ReorderFunctions != RT_USER) {
      StringRef AlgName;
      switch (opts::ReorderFunctions) {
      case RT_EXEC_COUNT:
        AlgName = "exec-count";
        break;
      case RT_HFSORT:
        AlgName = "hfsort";
        break;
      case RT_CDSORT:
        AlgName = "cdsort";
        break;
      case RT_PETTIS_HANSEN:
        AlgName = "pettis-hansen";
        break;
      case RT_RANDOM:
        AlgName = "random";
```

- EN: Declares or implements routines including `if`. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里较值得关注的符号包括 `if`。

### Lines 384-393

```cpp
        break;
      default:
        llvm_unreachable("unexpected reorder type in hybrid mode");
      }
      BC.outs() << "BOLT-INFO: hybrid mode: functions from order file will be "
                << "pinned first, remaining functions ordered by " << AlgName
                << "\n";
    }
  }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 394-407

```cpp
  // Process order file if provided. Assign indices 0..N-1 to listed functions.
  // For algorithmic modes, the algorithm orders remaining functions starting
  // at index N. For RT_USER, remaining functions keep original order.
  // UserFileEndIndex tracks the number of functions pinned by the order file.
  uint32_t UserFileEndIndex = 0;
  DenseSet<const BinaryFunction *> UserOrderedFuncs;

  if (!opts::FunctionOrderFile.empty()) {
    Expected<uint32_t> NextIndexOrErr =
        assignFunctionOrder(BC, BFs, 0, &UserOrderedFuncs);
    if (!NextIndexOrErr)
      return NextIndexOrErr.takeError();
    UserFileEndIndex = *NextIndexOrErr;
```

- EN: Declares or implements routines including `assignFunctionOrder`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assignFunctionOrder`.
- CN: 这里声明或实现函数，例如 `assignFunctionOrder`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assignFunctionOrder`。

### Lines 408-425

```cpp
    BC.outs() << "BOLT-INFO: " << UserFileEndIndex
              << " functions pinned by order file, remaining functions will be "
              << "ordered by algorithm\n";
  }

  // Build call graph (needed for clustering algorithms).
  // Exclude functions already ordered by the user order file.
  if (opts::ReorderFunctions != RT_NONE &&
      opts::ReorderFunctions != RT_EXEC_COUNT &&
      opts::ReorderFunctions != RT_USER) {
    Cg = buildCallGraph(
        BC,
        [&UserOrderedFuncs](const BinaryFunction &BF) {
          if (!BF.hasProfile())
            return true;
          if (BF.getState() != BinaryFunction::State::CFG)
            return true;
          if (UserOrderedFuncs.contains(&BF))
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 426-435

```cpp
            return true;
          return false;
        },
        opts::CgFromPerfData,
        /*IncludeSplitCalls=*/false, opts::ReorderFunctionsUseHotSize,
        opts::CgUseSplitHotSize, opts::UseEdgeCounts,
        opts::CgIgnoreRecursiveCalls);
    Cg.normalizeArcWeights();
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 436-453

```cpp
  // Run selected algorithm on remaining functions.
  std::vector<Cluster> Clusters;

  switch (opts::ReorderFunctions) {
  case RT_NONE:
    break;
  case RT_EXEC_COUNT: {
    BinaryFunctionListType SortedFunctions(BFs.size());
    llvm::transform(llvm::make_second_range(BFs), SortedFunctions.begin(),
                    [](BinaryFunction &BF) { return &BF; });
    llvm::stable_sort(
        SortedFunctions, [&](const BinaryFunction *A, const BinaryFunction *B) {
          if (A->isIgnored())
            return false;
          if (B->isIgnored())
            return true;
          const size_t PadA =
              opts::padFunctionBefore(*A) + opts::padFunctionAfter(*A);
```

- EN: Declares or implements routines including `SortedFunctions`, `transform`, `padFunctionBefore`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SortedFunctions`, `transform`, `padFunctionBefore`.
- CN: 这里声明或实现函数，例如 `SortedFunctions`, `transform`, `padFunctionBefore`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SortedFunctions`, `transform`, `padFunctionBefore`。

### Lines 454-471

```cpp
          const size_t PadB =
              opts::padFunctionBefore(*B) + opts::padFunctionAfter(*B);
          if (!PadA || !PadB) {
            if (PadA)
              return true;
            if (PadB)
              return false;
          }
          if (!A->hasProfile())
            return false;
          if (!B->hasProfile())
            return true;
          return A->getExecutionCount() > B->getExecutionCount();
        });
    uint32_t Index = UserFileEndIndex;
    for (BinaryFunction *BF : SortedFunctions)
      if (BF->hasProfile() && !BF->hasValidIndex()) {
        BF->setIndex(Index++);
```

- EN: Declares or implements routines including `padFunctionBefore`, `setIndex`. Notable symbols here include `padFunctionBefore`, `setIndex`.
- CN: 这里声明或实现函数，例如 `padFunctionBefore`, `setIndex`。这里较值得关注的符号包括 `padFunctionBefore`, `setIndex`。

### Lines 472-486

```cpp
        LLVM_DEBUG(if (opts::Verbosity > 1) {
          dbgs() << "BOLT-INFO: hot func " << BF->getPrintName() << " ("
                 << BF->getExecutionCount() << ")\n";
        });
      }
  } break;
  case RT_HFSORT:
    Clusters = clusterize(Cg);
    break;
  case RT_CDSORT: {
    // It is required that the sum of incoming arc weights is not greater
    // than the number of samples for every function. Ensuring the call graph
    // obeys the property before running the algorithm.
    Cg.adjustArcWeights();
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `dbgs`, `getExecutionCount`, `clusterize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `dbgs`, `getExecutionCount`, `clusterize`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `dbgs`, `getExecutionCount`, `clusterize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `dbgs`, `getExecutionCount`, `clusterize`。

### Lines 487-501

```cpp
    // Initialize CFG nodes and their data
    std::vector<uint64_t> FuncSizes;
    std::vector<uint64_t> FuncCounts;
    std::vector<codelayout::EdgeCount> CallCounts;
    std::vector<uint64_t> CallOffsets;
    for (NodeId F = 0; F < Cg.numNodes(); ++F) {
      FuncSizes.push_back(Cg.size(F));
      FuncCounts.push_back(Cg.samples(F));
      for (NodeId Succ : Cg.successors(F)) {
        const Arc &Arc = *Cg.findArc(F, Succ);
        CallCounts.push_back({F, Succ, uint64_t(Arc.weight())});
        CallOffsets.push_back(uint64_t(Arc.avgCallOffset()));
      }
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 502-519

```cpp
    // Run the layout algorithm.
    std::vector<uint64_t> Result = codelayout::computeCacheDirectedLayout(
        FuncSizes, FuncCounts, CallCounts, CallOffsets);

    // Create a single cluster from the computed order of hot functions.
    std::vector<CallGraph::NodeId> NodeOrder(Result.begin(), Result.end());
    Clusters.emplace_back(Cluster(NodeOrder, Cg));
  } break;
  case RT_PETTIS_HANSEN:
    Clusters = pettisAndHansen(Cg);
    break;
  case RT_RANDOM:
    std::srand(opts::RandomSeed);
    Clusters = randomClusters(Cg);
    break;
  case RT_USER:
    // Order file already processed above; nothing more to do.
    break;
```

- EN: Declares or implements routines including `NodeOrder`, `pettisAndHansen`, `srand`, `randomClusters`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `NodeOrder`, `pettisAndHansen`, `srand`, `randomClusters`.
- CN: 这里声明或实现函数，例如 `NodeOrder`, `pettisAndHansen`, `srand`, `randomClusters`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `NodeOrder`, `pettisAndHansen`, `srand`, `randomClusters`。

### Lines 520-527

```cpp
  default:
    llvm_unreachable("unexpected layout type");
  }

  reorder(BC, std::move(Clusters), BFs, UserFileEndIndex);

  BC.HasFinalizedFunctionOrder = true;
```

- EN: Declares or implements routines including `llvm_unreachable`, `reorder`. Notable symbols here include `llvm_unreachable`, `reorder`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `reorder`。这里较值得关注的符号包括 `llvm_unreachable`, `reorder`。

### Lines 528-538

```cpp
  std::unique_ptr<std::ofstream> FuncsFile;
  if (!opts::GenerateFunctionOrderFile.empty()) {
    FuncsFile = std::make_unique<std::ofstream>(opts::GenerateFunctionOrderFile,
                                                std::ios::out);
    if (!FuncsFile) {
      BC.errs() << "BOLT-ERROR: ordered functions file "
                << opts::GenerateFunctionOrderFile << " cannot be opened\n";
      return createFatalBOLTError("");
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 539-549

```cpp
  std::unique_ptr<std::ofstream> LinkSectionsFile;
  if (!opts::LinkSectionsFile.empty()) {
    LinkSectionsFile =
        std::make_unique<std::ofstream>(opts::LinkSectionsFile, std::ios::out);
    if (!LinkSectionsFile) {
      BC.errs() << "BOLT-ERROR: link sections file " << opts::LinkSectionsFile
                << " cannot be opened\n";
      return createFatalBOLTError("");
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 550-557

```cpp
  if (FuncsFile || LinkSectionsFile) {
    BinaryFunctionListType SortedFunctions(BFs.size());
    llvm::transform(llvm::make_second_range(BFs), SortedFunctions.begin(),
                    [](BinaryFunction &BF) { return &BF; });

    // Sort functions by index.
    llvm::stable_sort(SortedFunctions, compareBinaryFunctionByIndex);
```

- EN: Declares or implements routines including `SortedFunctions`, `transform`, `stable_sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `SortedFunctions`, `transform`, `stable_sort`.
- CN: 这里声明或实现函数，例如 `SortedFunctions`, `transform`, `stable_sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `SortedFunctions`, `transform`, `stable_sort`。

### Lines 558-566

```cpp
    for (const BinaryFunction *Func : SortedFunctions) {
      if (!Func->hasValidIndex())
        break;
      if (Func->isPLTFunction())
        continue;

      if (FuncsFile)
        *FuncsFile << Func->getOneName().str() << '\n';
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 567-584

```cpp
      if (LinkSectionsFile) {
        const char *Indent = "";
        std::vector<StringRef> AllNames = Func->getNames();
        llvm::sort(AllNames);
        for (StringRef Name : AllNames) {
          const size_t SlashPos = Name.find('/');
          if (SlashPos != std::string::npos) {
            // Avoid duplicates for local functions.
            if (Name.find('/', SlashPos + 1) != std::string::npos)
              continue;
            Name = Name.substr(0, SlashPos);
          }
          *LinkSectionsFile << Indent << ".text." << Name.str() << '\n';
          Indent = " ";
        }
      }
    }
```

- EN: Declares or implements routines including `getNames`, `sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getNames`, `sort`.
- CN: 这里声明或实现函数，例如 `getNames`, `sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getNames`, `sort`。

### Lines 585-599

```cpp
    if (FuncsFile) {
      FuncsFile->close();
      BC.outs() << "BOLT-INFO: dumped function order to "
                << opts::GenerateFunctionOrderFile << '\n';
    }

    if (LinkSectionsFile) {
      LinkSectionsFile->close();
      BC.outs() << "BOLT-INFO: dumped linker section order to "
                << opts::LinkSectionsFile << '\n';
    }
  }
  return Error::success();
}
```

- EN: Declares or implements routines including `close`. Notable symbols here include `close`.
- CN: 这里声明或实现函数，例如 `close`。这里较值得关注的符号包括 `close`。

### Lines 600-601

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `padFunctionBefore`: function or method entry point / 函数或方法入口
- `padFunctionAfter`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/ReorderFunctions.h`, `bolt/Passes/HFSort.h`, `bolt/Utils/Utils.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/STLExtras.h`, `llvm/ADT/StringMap.h`, `llvm/Support/CommandLine.h`, `llvm/Transforms/Utils/CodeLayout.h`
- System headers / 系统头文件: `fstream`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
