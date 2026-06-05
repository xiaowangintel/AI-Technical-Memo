# BinaryFunctionCallGraph.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/BinaryFunctionCallGraph.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Core/BinaryFunctionCallGraph.cpp This file implements the BinaryFunctionCallGraph class.. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：bolt/Core/BinaryFunctionCallGraph.cpp This file implements the BinaryFunctionCallGraph class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/BinaryFunctionCallGraph.cpp ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the BinaryFunctionCallGraph class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryFunction.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Timer.h"
#include <stack>

#define DEBUG_TYPE "callgraph"
```

- EN: Pulls in 6 header(s) from local project, LLVM, system dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-29

```cpp
using namespace llvm;

namespace opts {

extern cl::opt<bool> TimeOpts;
extern cl::opt<unsigned> Verbosity;
extern cl::OptionCategory BoltCategory;
```

- EN: Works inside namespace scope `llvm`, `opts` to organize symbols. Notable symbols here include `llvm`, `opts`.
- CN: 这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `opts`。

### Lines 30-38

```cpp
static cl::opt<std::string>
    DumpCGDot("dump-cg", cl::desc("dump callgraph to the given file"),
              cl::cat(BoltCategory));

} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `DumpCGDot`, `cat`. Notable symbols here include `DumpCGDot`, `cat`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `DumpCGDot`, `cat`。这里较值得关注的符号包括 `DumpCGDot`, `cat`, `opts`, `llvm`, `bolt`。

### Lines 39-49

```cpp
CallGraph::NodeId BinaryFunctionCallGraph::addNode(BinaryFunction *BF,
                                                   uint32_t Size,
                                                   uint64_t Samples) {
  NodeId Id = CallGraph::addNode(Size, Samples);
  assert(size_t(Id) == Funcs.size());
  Funcs.push_back(BF);
  FuncToNodeId[BF] = Id;
  assert(Funcs[Id] == BF);
  return Id;
}
```

- EN: Declares or implements routines including `addNode`, `assert`. Notable symbols here include `addNode`, `assert`.
- CN: 这里声明或实现函数，例如 `addNode`, `assert`。这里较值得关注的符号包括 `addNode`, `assert`。

### Lines 50-57

```cpp
std::deque<BinaryFunction *> BinaryFunctionCallGraph::buildTraversalOrder() {
  NamedRegionTimer T1("buildcgorder", "Build cg traversal order",
                      "CG breakdown", "CG breakdown", opts::TimeOpts);
  std::deque<BinaryFunction *> TopologicalOrder;
  enum NodeStatus { NEW, VISITING, VISITED };
  std::vector<NodeStatus> NodeStatus(Funcs.size());
  std::stack<NodeId> Worklist;
```

- EN: Defines enumerations such as `NodeStatus` to encode states or modes. Declares or implements routines including `buildTraversalOrder`, `NodeStatus`. Notable symbols here include `NodeStatus`, `buildTraversalOrder`.
- CN: 这里定义枚举 `NodeStatus`，用于表达状态或模式。这里声明或实现函数，例如 `buildTraversalOrder`, `NodeStatus`。这里较值得关注的符号包括 `NodeStatus`, `buildTraversalOrder`。

### Lines 58-65

```cpp
  for (BinaryFunction *Func : Funcs) {
    auto It = FuncToNodeId.find(Func);
    assert(It != FuncToNodeId.end());
    const NodeId Id = It->second;
    Worklist.push(Id);
    NodeStatus[Id] = NEW;
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 66-78

```cpp
  while (!Worklist.empty()) {
    const NodeId FuncId = Worklist.top();
    Worklist.pop();

    if (NodeStatus[FuncId] == VISITED)
      continue;

    if (NodeStatus[FuncId] == VISITING) {
      TopologicalOrder.push_back(Funcs[FuncId]);
      NodeStatus[FuncId] = VISITED;
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 79-88

```cpp
    assert(NodeStatus[FuncId] == NEW);
    NodeStatus[FuncId] = VISITING;
    Worklist.push(FuncId);
    for (const NodeId Callee : successors(FuncId)) {
      if (NodeStatus[Callee] == VISITING || NodeStatus[Callee] == VISITED)
        continue;
      Worklist.push(Callee);
    }
  }
```

- EN: Declares or implements routines including `assert`. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里较值得关注的符号包括 `assert`。

### Lines 89-102

```cpp
  return TopologicalOrder;
}

BinaryFunctionCallGraph
buildCallGraph(BinaryContext &BC, CgFilterFunction Filter, bool CgFromPerfData,
               bool IncludeSplitCalls, bool UseFunctionHotSize,
               bool UseSplitHotSize, bool UseEdgeCounts,
               bool IgnoreRecursiveCalls) {
  NamedRegionTimer T1("buildcg", "Callgraph construction", "CG breakdown",
                      "CG breakdown", opts::TimeOpts);
  BinaryFunctionCallGraph Cg;
  static constexpr uint64_t COUNT_NO_PROFILE =
      BinaryBasicBlock::COUNT_NO_PROFILE;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 103-120

```cpp
  // Compute function size
  auto functionSize = [&](const BinaryFunction *Function) {
    return UseFunctionHotSize && Function->isSplit()
               ? Function->estimateHotSize(UseSplitHotSize)
               : Function->estimateSize();
  };

  // Add call graph nodes.
  auto lookupNode = [&](BinaryFunction *Function) {
    const CallGraph::NodeId Id = Cg.maybeGetNodeId(Function);
    if (Id == CallGraph::InvalidId) {
      // It's ok to use the hot size here when the function is split.  This is
      // because emitFunctions will emit the hot part first in the order that is
      // computed by ReorderFunctions.  The cold part will be emitted with the
      // rest of the cold functions and code.
      const size_t Size = functionSize(Function);
      // NOTE: for functions without a profile, we set the number of samples
      // to zero.  This will keep these functions from appearing in the hot
```

- EN: Declares or implements routines including `estimateHotSize`, `estimateSize`, `functionSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `estimateHotSize`, `estimateSize`, `functionSize`.
- CN: 这里声明或实现函数，例如 `estimateHotSize`, `estimateSize`, `functionSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `estimateHotSize`, `estimateSize`, `functionSize`。

### Lines 121-131

```cpp
      // section.  This is a little weird because we wouldn't be trying to
      // create a node for a function unless it was the target of a call from
      // a hot block.  The alternative would be to set the count to one or
      // accumulate the number of calls from the callsite into the function
      // samples.  Results from performance testing seem to favor the zero
      // count though, so I'm leaving it this way for now.
      return Cg.addNode(Function, Size, Function->getKnownExecutionCount());
    }
    return Id;
  };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 132-140

```cpp
  // Add call graph edges.
  uint64_t NotProcessed = 0;
  uint64_t TotalCallsites = 0;
  uint64_t NoProfileCallsites = 0;
  uint64_t NumFallbacks = 0;
  uint64_t RecursiveCallsites = 0;
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction *Function = &It.second;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 141-156

```cpp
    if (Filter(*Function))
      continue;

    const CallGraph::NodeId SrcId = lookupNode(Function);
    // Offset of the current basic block from the beginning of the function
    uint64_t Offset = 0;

    auto recordCall = [&](const MCSymbol *DestSymbol, const uint64_t Count) {
      BinaryFunction *DstFunc =
          DestSymbol ? BC.getFunctionForSymbol(DestSymbol) : nullptr;
      if (!DstFunc) {
        LLVM_DEBUG(if (opts::Verbosity > 1) dbgs()
                   << "BOLT-DEBUG: buildCallGraph: no function for symbol\n");
        return false;
      }
```

- EN: Declares or implements routines including `lookupNode`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `lookupNode`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `lookupNode`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `lookupNode`, `LLVM_DEBUG`。

### Lines 157-170

```cpp
      if (DstFunc == Function) {
        LLVM_DEBUG(dbgs() << "BOLT-INFO: recursive call detected in "
                          << *DstFunc << "\n");
        ++RecursiveCallsites;
        if (IgnoreRecursiveCalls)
          return false;
      }
      if (Filter(*DstFunc)) {
        LLVM_DEBUG(if (opts::Verbosity > 1) dbgs()
                   << "BOLT-DEBUG: buildCallGraph: filtered " << *DstFunc
                   << '\n');
        return false;
      }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 171-183

```cpp
      const CallGraph::NodeId DstId = lookupNode(DstFunc);
      const bool IsValidCount = Count != COUNT_NO_PROFILE;
      const uint64_t AdjCount = UseEdgeCounts && IsValidCount ? Count : 1;
      if (!IsValidCount)
        ++NoProfileCallsites;
      Cg.incArcWeight(SrcId, DstId, AdjCount, Offset);
      LLVM_DEBUG(if (opts::Verbosity > 1) {
        dbgs() << "BOLT-DEBUG: buildCallGraph: call " << *Function << " -> "
               << *DstFunc << " @ " << Offset << "\n";
      });
      return true;
    };
```

- EN: Declares or implements routines including `lookupNode`, `LLVM_DEBUG`, `dbgs`. Notable symbols here include `lookupNode`, `LLVM_DEBUG`, `dbgs`.
- CN: 这里声明或实现函数，例如 `lookupNode`, `LLVM_DEBUG`, `dbgs`。这里较值得关注的符号包括 `lookupNode`, `LLVM_DEBUG`, `dbgs`。

### Lines 184-194

```cpp
    // Pairs of (symbol, count) for each target at this callsite.
    using TargetDesc = std::pair<const MCSymbol *, uint64_t>;
    using CallInfoTy = std::vector<TargetDesc>;

    // Get pairs of (symbol, count) for each target at this callsite.
    // If the call is to an unknown function the symbol will be nullptr.
    // If there is no profiling data the count will be COUNT_NO_PROFILE.
    auto getCallInfo = [&](const BinaryBasicBlock *BB, const MCInst &Inst) {
      CallInfoTy Counts;
      const MCSymbol *DstSym = BC.MIB->getTargetSymbol(Inst);
```

- EN: Declares or implements routines including `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 195-207

```cpp
      // If this is an indirect call use perf data directly.
      if (!DstSym && BC.MIB->hasAnnotation(Inst, "CallProfile")) {
        const auto &ICSP = BC.MIB->getAnnotationAs<IndirectCallSiteProfile>(
            Inst, "CallProfile");
        for (const IndirectCallProfile &CSI : ICSP)
          if (CSI.Symbol)
            Counts.emplace_back(CSI.Symbol, CSI.Count);
      } else {
        const uint64_t Count = BC.MIB->getAnnotationWithDefault(
            Inst, "Count", BB->getExecutionCount());
        Counts.emplace_back(DstSym, Count);
      }
```

- EN: Declares or implements routines including `getExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getExecutionCount`.
- CN: 这里声明或实现函数，例如 `getExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getExecutionCount`。

### Lines 208-223

```cpp
      return Counts;
    };

    // If the function has an invalid profile, try to use the perf data
    // directly (if requested).  If there is no perf data for this function,
    // fall back to the CFG walker which attempts to handle missing data.
    if (!Function->hasValidProfile() && CgFromPerfData &&
        !Function->getAllCallSites().empty()) {
      LLVM_DEBUG(
          dbgs() << "BOLT-DEBUG: buildCallGraph: Falling back to perf data"
                 << " for " << *Function << "\n");
      ++NumFallbacks;
      const size_t Size = functionSize(Function);
      for (const IndirectCallProfile &CSI : Function->getAllCallSites()) {
        ++TotalCallsites;
```

- EN: Declares or implements routines including `getAllCallSites`, `dbgs`, `functionSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAllCallSites`, `dbgs`, `functionSize`.
- CN: 这里声明或实现函数，例如 `getAllCallSites`, `dbgs`, `functionSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAllCallSites`, `dbgs`, `functionSize`。

### Lines 224-232

```cpp
        if (!CSI.Symbol)
          continue;

        // The computed offset may exceed the hot part of the function; hence,
        // bound it by the size.
        Offset = CSI.Offset;
        if (Offset > Size)
          Offset = Size;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 233-241

```cpp
        if (!recordCall(CSI.Symbol, CSI.Count))
          ++NotProcessed;
      }
    } else {
      for (BinaryBasicBlock *BB : Function->getLayout().blocks()) {
        // Don't count calls from split blocks unless requested.
        if (BB->isSplit() && !IncludeSplitCalls)
          continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 242-253

```cpp
        // Determine whether the block is included in Function's (hot) size
        // See BinaryFunction::estimateHotSize
        bool BBIncludedInFunctionSize = false;
        if (UseFunctionHotSize && Function->isSplit()) {
          if (UseSplitHotSize)
            BBIncludedInFunctionSize = !BB->isSplit();
          else
            BBIncludedInFunctionSize = BB->getKnownExecutionCount() != 0;
        } else {
          BBIncludedInFunctionSize = true;
        }
```

- EN: Declares or implements routines including `isSplit`, `getKnownExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isSplit`, `getKnownExecutionCount`.
- CN: 这里声明或实现函数，例如 `isSplit`, `getKnownExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isSplit`, `getKnownExecutionCount`。

### Lines 254-271

```cpp
        for (MCInst &Inst : *BB) {
          // Find call instructions and extract target symbols from each one.
          if (BC.MIB->isCall(Inst)) {
            const CallInfoTy CallInfo = getCallInfo(BB, Inst);

            if (!CallInfo.empty()) {
              for (const TargetDesc &CI : CallInfo) {
                ++TotalCallsites;
                if (!recordCall(CI.first, CI.second))
                  ++NotProcessed;
              }
            } else {
              ++TotalCallsites;
              ++NotProcessed;
            }
          }
          // Increase Offset if needed
          if (BBIncludedInFunctionSize)
```

- EN: Declares or implements routines including `getCallInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getCallInfo`.
- CN: 这里声明或实现函数，例如 `getCallInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getCallInfo`。

### Lines 272-289

```cpp
            Offset += BC.computeCodeSize(&Inst, &Inst + 1);
        }
      }
    }
  }

#ifndef NDEBUG
  bool PrintInfo = DebugFlag && isCurrentDebugType("callgraph");
#else
  bool PrintInfo = false;
#endif
  if (PrintInfo || opts::Verbosity > 0)
    BC.outs() << format("BOLT-INFO: buildCallGraph: %u nodes, %u callsites "
                        "(%u recursive), density = %.6lf, %u callsites not "
                        "processed, %u callsites with invalid profile, "
                        "used perf data for %u stale functions.\n",
                        Cg.numNodes(), TotalCallsites, RecursiveCallsites,
                        Cg.density(), NotProcessed, NoProfileCallsites,
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Declares or implements routines including `isCurrentDebugType`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isCurrentDebugType`.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里声明或实现函数，例如 `isCurrentDebugType`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isCurrentDebugType`。

### Lines 290-297

```cpp
                        NumFallbacks);

  if (opts::DumpCGDot.getNumOccurrences()) {
    Cg.printDot(opts::DumpCGDot, [&](CallGraph::NodeId Id) {
      return Cg.nodeIdToFunc(Id)->getPrintName();
    });
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 298-302

```cpp
  return Cg;
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `NodeStatus`: enumeration of modes or states / 模式或状态枚举
- `DumpCGDot`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `addNode`: function or method entry point / 函数或方法入口
- `assert`: function or method entry point / 函数或方法入口
- `buildTraversalOrder`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryFunction.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`, `llvm/Support/Timer.h`
- System headers / 系统头文件: `stack`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
