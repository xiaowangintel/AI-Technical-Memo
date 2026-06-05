# Instrumentation.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/Instrumentation.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/Instrumentation.cpp This file implements the Instrumentation class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/Instrumentation.cpp This file implements the Instrumentation class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/Instrumentation.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the Instrumentation class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-24

```cpp
#include "bolt/Passes/Instrumentation.h"
#include "bolt/Core/ParallelUtilities.h"
#include "bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "bolt/Utils/Utils.h"
#include "llvm/ADT/StringSet.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/RWMutex.h"
#include <fstream>
#include <queue>
#include <stack>
```

- EN: Pulls in 11 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 11 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 25-37

```cpp
#define DEBUG_TYPE "bolt-instrumentation"

using namespace llvm;

namespace opts {
extern cl::OptionCategory BoltInstrCategory;

cl::opt<std::string> InstrumentationFilename(
    "instrumentation-file",
    cl::desc("file name where instrumented profile will be saved (default: "
             "/tmp/prof.fdata)"),
    cl::init("/tmp/prof.fdata"), cl::Optional, cl::cat(BoltInstrCategory));
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Declares or implements routines including `init`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `init`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 38-48

```cpp
cl::opt<std::string> InstrumentationBinpath(
    "instrumentation-binpath",
    cl::desc("path to instrumented binary in case if /proc/self/map_files "
             "is not accessible due to access restriction issues"),
    cl::Optional, cl::cat(BoltInstrCategory));

cl::opt<bool> InstrumentationFileAppendPID(
    "instrumentation-file-append-pid",
    cl::desc("append PID to saved profile file name (default: false)"),
    cl::init(false), cl::Optional, cl::cat(BoltInstrCategory));
```

- EN: Declares or implements routines including `cat`, `desc`, `init`. Notable symbols here include `cat`, `desc`, `init`.
- CN: 这里声明或实现函数，例如 `cat`, `desc`, `init`。这里较值得关注的符号包括 `cat`, `desc`, `init`。

### Lines 49-60

```cpp
cl::opt<bool> ConservativeInstrumentation(
    "conservative-instrumentation",
    cl::desc("disable instrumentation optimizations that sacrifice profile "
             "accuracy (for debugging, default: false)"),
    cl::init(false), cl::Optional, cl::cat(BoltInstrCategory));

cl::opt<uint32_t> InstrumentationMaxSize(
    "instrumentation-max-size",
    cl::desc("Set max memory size of the instrumentation bump allocator "
             "default: 0x6400000)"),
    cl::init(0x6400000), cl::Optional, cl::cat(BoltInstrCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 61-68

```cpp
cl::opt<uint32_t> InstrumentationSleepTime(
    "instrumentation-sleep-time",
    cl::desc("interval between profile writes (default: 0 = write only at "
             "program end).  This is useful for service workloads when you "
             "want to dump profile every X minutes or if you are killing the "
             "program and the profile is not being dumped at the end."),
    cl::init(0), cl::Optional, cl::cat(BoltInstrCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 69-80

```cpp
cl::opt<bool> InstrumentationNoCountersClear(
    "instrumentation-no-counters-clear",
    cl::desc("Don't clear counters across dumps "
             "(use with instrumentation-sleep-time option)"),
    cl::init(false), cl::Optional, cl::cat(BoltInstrCategory));

cl::opt<bool> InstrumentationWaitForks(
    "instrumentation-wait-forks",
    cl::desc("Wait until all forks of instrumented process will finish "
             "(use with instrumentation-sleep-time option)"),
    cl::init(false), cl::Optional, cl::cat(BoltInstrCategory));
```

- EN: Declares or implements routines including `init`. Notable symbols here include `init`.
- CN: 这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`。

### Lines 81-94

```cpp
cl::opt<std::string> InstrumentFuncsFile(
    "instrument-funcs-file",
    cl::desc("file with list of function names (one per line) to instrument; "
             "only functions whose name exactly matches a line in this file "
             "will be instrumented"),
    cl::Optional, cl::cat(BoltInstrCategory));

cl::opt<bool>
    InstrumentHotOnly("instrument-hot-only",
                      cl::desc("only insert instrumentation on hot functions "
                               "(needs profile, default: false)"),
                      cl::init(false), cl::Optional,
                      cl::cat(BoltInstrCategory));
```

- EN: Declares or implements routines including `desc`, `cat`, `init`. Notable symbols here include `desc`, `cat`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`, `init`。这里较值得关注的符号包括 `desc`, `cat`, `init`。

### Lines 95-104

```cpp
cl::opt<bool> InstrumentCalls("instrument-calls",
                              cl::desc("record profile for inter-function "
                                       "control flow activity (default: true)"),
                              cl::init(true), cl::Optional,
                              cl::cat(BoltInstrCategory));
} // namespace opts

namespace llvm {
namespace bolt {
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Declares or implements routines including `activity`, `init`, `cat`. Notable symbols here include `activity`, `init`, `cat`, `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `activity`, `init`, `cat`。这里较值得关注的符号包括 `activity`, `init`, `cat`, `opts`, `llvm`, `bolt`。

### Lines 105-115

```cpp
static bool
hasAArch64ExclusiveMemop(BinaryFunction &Function,
                         DenseSet<const BinaryBasicBlock *> &BBToSkip) {
  // FIXME ARMv8-a architecture reference manual says that software must avoid
  // having any explicit memory accesses between exclusive load and associated
  // store instruction. So for now skip instrumentation for basic blocks that
  // have these instructions, since it might lead to runtime deadlock.
  BinaryContext &BC = Function.getBinaryContext();
  std::queue<std::pair<BinaryBasicBlock *, bool>> BBQueue; // {BB, isLoad}
  DenseSet<BinaryBasicBlock *> Visited;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 116-128

```cpp
  if (Function.getLayout().block_begin() == Function.getLayout().block_end())
    return 0;

  BinaryBasicBlock *BBfirst = *Function.getLayout().block_begin();
  BBQueue.push({BBfirst, false});

  while (!BBQueue.empty()) {
    BinaryBasicBlock *BB = BBQueue.front().first;
    bool IsLoad = BBQueue.front().second;
    BBQueue.pop();
    if (!Visited.insert(BB).second)
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 129-138

```cpp
    for (const MCInst &Inst : *BB) {
      // Two loads one after another - skip whole function
      if (BC.MIB->isAArch64ExclusiveLoad(Inst) && IsLoad) {
        if (opts::Verbosity >= 2) {
          outs() << "BOLT-INSTRUMENTER: function " << Function.getPrintName()
                 << " has two exclusive loads. Ignoring the function.\n";
        }
        return true;
      }
```

- EN: Declares or implements routines including `outs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `outs`。

### Lines 139-149

```cpp
      if (BC.MIB->isAArch64ExclusiveLoad(Inst))
        IsLoad = true;

      if (IsLoad && BBToSkip.insert(BB).second) {
        if (opts::Verbosity >= 2) {
          outs() << "BOLT-INSTRUMENTER: skip BB " << BB->getName()
                 << " due to exclusive instruction in function "
                 << Function.getPrintName() << "\n";
        }
      }
```

- EN: Declares or implements routines including `outs`. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里较值得关注的符号包括 `outs`。

### Lines 150-158

```cpp
      if (!IsLoad && BC.MIB->isAArch64ExclusiveStore(Inst)) {
        if (opts::Verbosity >= 2) {
          outs() << "BOLT-INSTRUMENTER: function " << Function.getPrintName()
                 << " has exclusive store without corresponding load. Ignoring "
                    "the function.\n";
        }
        return true;
      }
```

- EN: Declares or implements routines including `outs`. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里较值得关注的符号包括 `outs`。

### Lines 159-172

```cpp
      if (IsLoad && (BC.MIB->isAArch64ExclusiveStore(Inst) ||
                     BC.MIB->isAArch64ExclusiveClear(Inst)))
        IsLoad = false;
    }

    if (IsLoad && BB->succ_size() == 0) {
      if (opts::Verbosity >= 2) {
        outs()
            << "BOLT-INSTRUMENTER: function " << Function.getPrintName()
            << " has exclusive load in trailing BB. Ignoring the function.\n";
      }
      return true;
    }
```

- EN: Declares or implements routines including `isAArch64ExclusiveClear`, `outs`. Notable symbols here include `isAArch64ExclusiveClear`, `outs`.
- CN: 这里声明或实现函数，例如 `isAArch64ExclusiveClear`, `outs`。这里较值得关注的符号包括 `isAArch64ExclusiveClear`, `outs`。

### Lines 173-185

```cpp
    for (BinaryBasicBlock *BBS : BB->successors())
      BBQueue.push({BBS, IsLoad});
  }

  if (BBToSkip.size() == Visited.size()) {
    if (opts::Verbosity >= 2) {
      outs() << "BOLT-INSTRUMENTER: all BBs are marked with true. Ignoring the "
                "function "
             << Function.getPrintName() << "\n";
    }
    return true;
  }
```

- EN: Declares or implements routines including `outs`. Notable symbols here include `outs`.
- CN: 这里声明或实现函数，例如 `outs`。这里较值得关注的符号包括 `outs`。

### Lines 186-199

```cpp
  return false;
}

uint32_t Instrumentation::getFunctionNameIndex(const BinaryFunction &Function) {
  auto Iter = FuncToStringIdx.find(&Function);
  if (Iter != FuncToStringIdx.end())
    return Iter->second;
  size_t Idx = Summary->StringTable.size();
  FuncToStringIdx.emplace(std::make_pair(&Function, Idx));
  Summary->StringTable.append(getEscapedName(Function.getOneName()));
  Summary->StringTable.append(1, '\0');
  return Idx;
}
```

- EN: Declares or implements routines including `getFunctionNameIndex`. Notable symbols here include `getFunctionNameIndex`.
- CN: 这里声明或实现函数，例如 `getFunctionNameIndex`。这里较值得关注的符号包括 `getFunctionNameIndex`。

### Lines 200-217

```cpp
bool Instrumentation::createCallDescription(FunctionDescription &FuncDesc,
                                            const BinaryFunction &FromFunction,
                                            uint32_t From, uint32_t FromNodeID,
                                            const BinaryFunction &ToFunction,
                                            uint32_t To, bool IsInvoke) {
  CallDescription CD;
  // Ordinarily, we don't augment direct calls with an explicit counter, except
  // when forced to do so or when we know this callee could be throwing
  // exceptions, in which case there is no other way to accurately record its
  // frequency.
  bool ForceInstrumentation = opts::ConservativeInstrumentation || IsInvoke;
  CD.FromLoc.FuncString = getFunctionNameIndex(FromFunction);
  CD.FromLoc.Offset = From;
  CD.FromNode = FromNodeID;
  CD.Target = &ToFunction;
  CD.ToLoc.FuncString = getFunctionNameIndex(ToFunction);
  CD.ToLoc.Offset = To;
  CD.Counter = ForceInstrumentation ? Summary->Counters.size() : 0xffffffff;
```

- EN: Declares or implements routines including `getFunctionNameIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getFunctionNameIndex`.
- CN: 这里声明或实现函数，例如 `getFunctionNameIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getFunctionNameIndex`。

### Lines 218-231

```cpp
  if (ForceInstrumentation)
    ++DirectCallCounters;
  FuncDesc.Calls.emplace_back(CD);
  return ForceInstrumentation;
}

void Instrumentation::createIndCallDescription(
    const BinaryFunction &FromFunction, uint32_t From) {
  IndCallDescription ICD;
  ICD.FromLoc.FuncString = getFunctionNameIndex(FromFunction);
  ICD.FromLoc.Offset = From;
  Summary->IndCallDescriptions.emplace_back(ICD);
}
```

- EN: Declares or implements routines including `getFunctionNameIndex`. Notable symbols here include `getFunctionNameIndex`.
- CN: 这里声明或实现函数，例如 `getFunctionNameIndex`。这里较值得关注的符号包括 `getFunctionNameIndex`。

### Lines 232-240

```cpp
void Instrumentation::createIndCallTargetDescription(
    const BinaryFunction &ToFunction, uint32_t To) {
  IndCallTargetDescription ICD;
  ICD.ToLoc.FuncString = getFunctionNameIndex(ToFunction);
  ICD.ToLoc.Offset = To;
  ICD.Target = &ToFunction;
  Summary->IndCallTargetDescriptions.emplace_back(ICD);
}
```

- EN: Declares or implements routines including `getFunctionNameIndex`. Notable symbols here include `getFunctionNameIndex`.
- CN: 这里声明或实现函数，例如 `getFunctionNameIndex`。这里较值得关注的符号包括 `getFunctionNameIndex`。

### Lines 241-258

```cpp
bool Instrumentation::createEdgeDescription(FunctionDescription &FuncDesc,
                                            const BinaryFunction &FromFunction,
                                            uint32_t From, uint32_t FromNodeID,
                                            const BinaryFunction &ToFunction,
                                            uint32_t To, uint32_t ToNodeID,
                                            bool Instrumented) {
  EdgeDescription ED;
  auto Result = FuncDesc.EdgesSet.insert(std::make_pair(FromNodeID, ToNodeID));
  // Avoid creating duplicated edge descriptions. This happens in CFGs where a
  // block jumps to its fall-through.
  if (Result.second == false)
    return false;
  ED.FromLoc.FuncString = getFunctionNameIndex(FromFunction);
  ED.FromLoc.Offset = From;
  ED.FromNode = FromNodeID;
  ED.ToLoc.FuncString = getFunctionNameIndex(ToFunction);
  ED.ToLoc.Offset = To;
  ED.ToNode = ToNodeID;
```

- EN: Declares or implements routines including `getFunctionNameIndex`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getFunctionNameIndex`.
- CN: 这里声明或实现函数，例如 `getFunctionNameIndex`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getFunctionNameIndex`。

### Lines 259-274

```cpp
  ED.Counter = Instrumented ? Summary->Counters.size() : 0xffffffff;
  if (Instrumented)
    ++BranchCounters;
  FuncDesc.Edges.emplace_back(ED);
  return Instrumented;
}

void Instrumentation::createLeafNodeDescription(FunctionDescription &FuncDesc,
                                                uint32_t Node) {
  InstrumentedNode IN;
  IN.Node = Node;
  IN.Counter = Summary->Counters.size();
  ++LeafNodeCounters;
  FuncDesc.LeafNodes.emplace_back(IN);
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 275-283

```cpp
InstructionListType
Instrumentation::createInstrumentationSnippet(BinaryContext &BC, bool IsLeaf) {
  auto L = BC.scopeLock();
  MCSymbol *Label = BC.Ctx->createNamedTempSymbol("InstrEntry");
  Summary->Counters.emplace_back(Label);
  return BC.MIB->createInstrIncMemory(Label, BC.Ctx.get(), IsLeaf,
                                      BC.AsmInfo->getCodePointerSize());
}
```

- EN: Declares or implements routines including `createInstrumentationSnippet`, `createNamedTempSymbol`, `getCodePointerSize`. Notable symbols here include `createInstrumentationSnippet`, `createNamedTempSymbol`, `getCodePointerSize`.
- CN: 这里声明或实现函数，例如 `createInstrumentationSnippet`, `createNamedTempSymbol`, `getCodePointerSize`。这里较值得关注的符号包括 `createInstrumentationSnippet`, `createNamedTempSymbol`, `getCodePointerSize`。

### Lines 284-294

```cpp
// Helper instruction sequence insertion function
static BinaryBasicBlock::iterator
insertInstructions(InstructionListType &Instrs, BinaryBasicBlock &BB,
                   BinaryBasicBlock::iterator Iter) {
  for (MCInst &NewInst : Instrs) {
    Iter = BB.insertInstruction(Iter, NewInst);
    ++Iter;
  }
  return Iter;
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 295-305

```cpp
void Instrumentation::instrumentLeafNode(BinaryBasicBlock &BB,
                                         BinaryBasicBlock::iterator Iter,
                                         bool IsLeaf,
                                         FunctionDescription &FuncDesc,
                                         uint32_t Node) {
  createLeafNodeDescription(FuncDesc, Node);
  InstructionListType CounterInstrs = createInstrumentationSnippet(
      BB.getFunction()->getBinaryContext(), IsLeaf);
  insertInstructions(CounterInstrs, BB, Iter);
}
```

- EN: Declares or implements routines including `createLeafNodeDescription`, `insertInstructions`. Notable symbols here include `createLeafNodeDescription`, `insertInstructions`.
- CN: 这里声明或实现函数，例如 `createLeafNodeDescription`, `insertInstructions`。这里较值得关注的符号包括 `createLeafNodeDescription`, `insertInstructions`。

### Lines 306-313

```cpp
void Instrumentation::instrumentIndirectTarget(BinaryBasicBlock &BB,
                                               BinaryBasicBlock::iterator &Iter,
                                               BinaryFunction &FromFunction,
                                               uint32_t From) {
  auto L = FromFunction.getBinaryContext().scopeLock();
  const size_t IndCallSiteID = Summary->IndCallDescriptions.size();
  createIndCallDescription(FromFunction, From);
```

- EN: Declares or implements routines including `createIndCallDescription`. Notable symbols here include `createIndCallDescription`.
- CN: 这里声明或实现函数，例如 `createIndCallDescription`。这里较值得关注的符号包括 `createIndCallDescription`。

### Lines 314-321

```cpp
  BinaryContext &BC = FromFunction.getBinaryContext();
  bool IsTailCall = BC.MIB->isTailCall(*Iter);
  InstructionListType CounterInstrs = BC.MIB->createInstrumentedIndirectCall(
      std::move(*Iter),
      IsTailCall ? IndTailCallHandlerExitBBFunction->getSymbol()
                 : IndCallHandlerExitBBFunction->getSymbol(),
      IndCallSiteID, &*BC.Ctx);
```

- EN: Declares or implements routines including `isTailCall`, `move`, `getSymbol`. Notable symbols here include `isTailCall`, `move`, `getSymbol`.
- CN: 这里声明或实现函数，例如 `isTailCall`, `move`, `getSymbol`。这里较值得关注的符号包括 `isTailCall`, `move`, `getSymbol`。

### Lines 322-339

```cpp
  Iter = BB.eraseInstruction(Iter);
  Iter = insertInstructions(CounterInstrs, BB, Iter);
  --Iter;
}

bool Instrumentation::instrumentOneTarget(
    SplitWorklistTy &SplitWorklist, SplitInstrsTy &SplitInstrs,
    BinaryBasicBlock::iterator &Iter, BinaryFunction &FromFunction,
    BinaryBasicBlock &FromBB, uint32_t From, BinaryFunction &ToFunc,
    BinaryBasicBlock *TargetBB, uint32_t ToOffset, bool IsLeaf, bool IsInvoke,
    FunctionDescription *FuncDesc, uint32_t FromNodeID, uint32_t ToNodeID) {
  BinaryContext &BC = FromFunction.getBinaryContext();
  {
    auto L = BC.scopeLock();
    bool Created = true;
    if (!TargetBB)
      Created = createCallDescription(*FuncDesc, FromFunction, From, FromNodeID,
                                      ToFunc, ToOffset, IsInvoke);
```

- EN: Declares or implements routines including `insertInstructions`. Notable symbols here include `insertInstructions`.
- CN: 这里声明或实现函数，例如 `insertInstructions`。这里较值得关注的符号包括 `insertInstructions`。

### Lines 340-347

```cpp
    else
      Created = createEdgeDescription(*FuncDesc, FromFunction, From, FromNodeID,
                                      ToFunc, ToOffset, ToNodeID,
                                      /*Instrumented=*/true);
    if (!Created)
      return false;
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 348-358

```cpp
  InstructionListType CounterInstrs = createInstrumentationSnippet(BC, IsLeaf);

  const MCInst &Inst = *Iter;
  if (BC.MIB->isCall(Inst)) {
    // This code handles both
    // - (regular) inter-function calls (cross-function control transfer),
    // - (rare) intra-function calls (function-local control transfer)
    Iter = insertInstructions(CounterInstrs, FromBB, Iter);
    return true;
  }
```

- EN: Declares or implements routines including `createInstrumentationSnippet`, `insertInstructions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createInstrumentationSnippet`, `insertInstructions`.
- CN: 这里声明或实现函数，例如 `createInstrumentationSnippet`, `insertInstructions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createInstrumentationSnippet`, `insertInstructions`。

### Lines 359-376

```cpp
  if (!TargetBB || !FuncDesc)
    return false;

  // Indirect branch, conditional branches or fall-throughs
  // Regular cond branch, put counter at start of target block
  //
  // N.B.: (FromBB != TargetBBs) checks below handle conditional jumps where
  // we can't put the instrumentation counter in this block because not all
  // paths that reach it at this point will be taken and going to the target.
  if (TargetBB->pred_size() == 1 && &FromBB != TargetBB &&
      !TargetBB->isEntryPoint()) {
    insertInstructions(CounterInstrs, *TargetBB, TargetBB->begin());
    return true;
  }
  if (FromBB.succ_size() == 1 && &FromBB != TargetBB) {
    Iter = insertInstructions(CounterInstrs, FromBB, Iter);
    return true;
  }
```

- EN: Declares or implements routines including `isEntryPoint`, `insertInstructions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isEntryPoint`, `insertInstructions`.
- CN: 这里声明或实现函数，例如 `isEntryPoint`, `insertInstructions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isEntryPoint`, `insertInstructions`。

### Lines 377-387

```cpp
  // Critical edge, create BB and put counter there
  SplitWorklist.emplace_back(&FromBB, TargetBB);
  SplitInstrs.emplace_back(std::move(CounterInstrs));
  return true;
}

void Instrumentation::instrumentFunction(BinaryFunction &Function,
                                         MCPlusBuilder::AllocatorIdTy AllocId) {
  if (Function.hasUnknownControlFlow())
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 388-395

```cpp
  BinaryContext &BC = Function.getBinaryContext();
  if (BC.isMachO() && Function.hasName("___GLOBAL_init_65535/1"))
    return;

  DenseSet<const BinaryBasicBlock *> BBToSkip;
  if (BC.isAArch64() && hasAArch64ExclusiveMemop(Function, BBToSkip))
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 396-405

```cpp
  SplitWorklistTy SplitWorklist;
  SplitInstrsTy SplitInstrs;

  FunctionDescription *FuncDesc = nullptr;
  {
    std::unique_lock<llvm::sys::RWMutex> L(FDMutex);
    Summary->FunctionDescriptions.emplace_back();
    FuncDesc = &Summary->FunctionDescriptions.back();
  }
```

- EN: Declares or implements routines including `L`. Notable symbols here include `L`.
- CN: 这里声明或实现函数，例如 `L`。这里较值得关注的符号包括 `L`。

### Lines 406-423

```cpp
  FuncDesc->Function = &Function;
  Function.disambiguateJumpTables(AllocId);
  Function.deleteConservativeEdges();

  DenseMap<const BinaryBasicBlock *, uint32_t> BBToID;
  uint32_t Id = 0;
  for (auto BBI = Function.begin(); BBI != Function.end(); ++BBI) {
    BBToID[&*BBI] = Id++;
  }
  DenseSet<const BinaryBasicBlock *> VisitedSet;
  // DFS to establish edges we will use for a spanning tree. Edges in the
  // spanning tree can be instrumentation-free since their count can be
  // inferred by solving flow equations on a bottom-up traversal of the tree.
  // Exit basic blocks are always instrumented so we start the traversal with
  // a minimum number of defined variables to make the equation solvable.
  std::stack<std::pair<const BinaryBasicBlock *, BinaryBasicBlock *>> Stack;
  DenseMap<const BinaryBasicBlock *, SmallVector<const BinaryBasicBlock *>>
      STOutSet;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 424-437

```cpp
  for (auto BBI = Function.getLayout().block_rbegin();
       BBI != Function.getLayout().block_rend(); ++BBI) {
    if ((*BBI)->isEntryPoint() || (*BBI)->isLandingPad()) {
      Stack.push(std::make_pair(nullptr, *BBI));
      if (opts::InstrumentCalls && (*BBI)->isEntryPoint()) {
        EntryNode E;
        E.Node = BBToID[&**BBI];
        E.Address = (*BBI)->getInputOffset();
        FuncDesc->EntryNodes.emplace_back(E);
        createIndCallTargetDescription(Function, (*BBI)->getInputOffset());
      }
    }
  }
```

- EN: Declares or implements routines including `createIndCallTargetDescription`. Notable symbols here include `createIndCallTargetDescription`.
- CN: 这里声明或实现函数，例如 `createIndCallTargetDescription`。这里较值得关注的符号包括 `createIndCallTargetDescription`。

### Lines 438-447

```cpp
  // Modified version of BinaryFunction::dfs() to build a spanning tree
  if (!opts::ConservativeInstrumentation) {
    while (!Stack.empty()) {
      BinaryBasicBlock *BB;
      const BinaryBasicBlock *Pred;
      std::tie(Pred, BB) = Stack.top();
      Stack.pop();
      if (llvm::is_contained(VisitedSet, BB))
        continue;
```

- EN: Declares or implements routines including `tie`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `tie`.
- CN: 这里声明或实现函数，例如 `tie`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `tie`。

### Lines 448-456

```cpp
      VisitedSet.insert(BB);
      if (Pred)
        STOutSet[Pred].push_back(BB);

      for (BinaryBasicBlock *SuccBB : BB->successors())
        Stack.push(std::make_pair(BB, SuccBB));
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 457-471

```cpp
  // Determine whether this is a leaf function, which needs special
  // instructions to protect the red zone
  bool IsLeafFunction = true;
  DenseSet<const BinaryBasicBlock *> InvokeBlocks;
  for (const BinaryBasicBlock &BB : Function) {
    for (const MCInst &Inst : BB) {
      if (BC.MIB->isCall(Inst)) {
        if (BC.MIB->isInvoke(Inst))
          InvokeBlocks.insert(&BB);
        if (!BC.MIB->isTailCall(Inst))
          IsLeafFunction = false;
      }
    }
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 472-482

```cpp
  for (auto BBI = Function.begin(), BBE = Function.end(); BBI != BBE; ++BBI) {
    BinaryBasicBlock &BB = *BBI;

    // Skip BBs with exclusive load/stores
    if (BBToSkip.find(&BB) != BBToSkip.end())
      continue;

    bool HasUnconditionalBranch = false;
    bool HasJumpTable = false;
    bool IsInvokeBlock = InvokeBlocks.count(&BB) > 0;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 483-495

```cpp
    for (auto I = BB.begin(); I != BB.end(); ++I) {
      const MCInst &Inst = *I;
      if (!BC.MIB->getOffset(Inst))
        continue;

      const bool IsJumpTable = Function.getJumpTable(Inst);
      if (IsJumpTable)
        HasJumpTable = true;
      else if (BC.MIB->isUnconditionalBranch(Inst))
        HasUnconditionalBranch = true;
      else if (!(BC.MIB->isCall(Inst) || BC.MIB->isConditionalBranch(Inst)))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 496-513

```cpp
      const uint32_t FromOffset = *BC.MIB->getOffset(Inst);
      const MCSymbol *Target = BC.MIB->getTargetSymbol(Inst);
      BinaryBasicBlock *TargetBB = Function.getBasicBlockForLabel(Target);
      uint32_t ToOffset = TargetBB ? TargetBB->getInputOffset() : 0;
      BinaryFunction *TargetFunc =
          TargetBB ? &Function : BC.getFunctionForSymbol(Target);
      if (TargetFunc && BC.MIB->isCall(Inst)) {
        if (opts::InstrumentCalls) {
          const BinaryBasicBlock *ForeignBB =
              TargetFunc->getBasicBlockForLabel(Target);
          if (ForeignBB)
            ToOffset = ForeignBB->getInputOffset();
          instrumentOneTarget(SplitWorklist, SplitInstrs, I, Function, BB,
                              FromOffset, *TargetFunc, TargetBB, ToOffset,
                              IsLeafFunction, IsInvokeBlock, FuncDesc,
                              BBToID[&BB]);
        }
        continue;
```

- EN: Declares or implements routines including `getOffset`, `getTargetSymbol`, `getInputOffset`, `getBasicBlockForLabel`. Notable symbols here include `getOffset`, `getTargetSymbol`, `getInputOffset`, `getBasicBlockForLabel`.
- CN: 这里声明或实现函数，例如 `getOffset`, `getTargetSymbol`, `getInputOffset`, `getBasicBlockForLabel`。这里较值得关注的符号包括 `getOffset`, `getTargetSymbol`, `getInputOffset`, `getBasicBlockForLabel`。

### Lines 514-531

```cpp
      }
      if (TargetFunc) {
        // Do not instrument edges in the spanning tree
        auto STIt = STOutSet.find(&BB);
        if (STIt != STOutSet.end() &&
            llvm::is_contained(STIt->second, TargetBB)) {
          auto L = BC.scopeLock();
          createEdgeDescription(*FuncDesc, Function, FromOffset, BBToID[&BB],
                                Function, ToOffset, BBToID[TargetBB],
                                /*Instrumented=*/false);
          continue;
        }
        instrumentOneTarget(SplitWorklist, SplitInstrs, I, Function, BB,
                            FromOffset, *TargetFunc, TargetBB, ToOffset,
                            IsLeafFunction, IsInvokeBlock, FuncDesc,
                            BBToID[&BB], BBToID[TargetBB]);
        continue;
      }
```

- EN: Declares or implements routines including `is_contained`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `is_contained`.
- CN: 这里声明或实现函数，例如 `is_contained`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `is_contained`。

### Lines 532-549

```cpp

      if (IsJumpTable) {
        auto STIt = STOutSet.find(&BB);
        bool Found = STIt != STOutSet.end();
        for (BinaryBasicBlock *&Succ : BB.successors()) {
          // Do not instrument edges in the spanning tree
          if (Found && llvm::is_contained(STIt->second, &*Succ)) {
            auto L = BC.scopeLock();
            createEdgeDescription(*FuncDesc, Function, FromOffset, BBToID[&BB],
                                  Function, Succ->getInputOffset(),
                                  BBToID[&*Succ], /*Instrumented=*/false);
            continue;
          }
          instrumentOneTarget(
              SplitWorklist, SplitInstrs, I, Function, BB, FromOffset, Function,
              &*Succ, Succ->getInputOffset(), IsLeafFunction, IsInvokeBlock,
              FuncDesc, BBToID[&BB], BBToID[&*Succ]);
        }
```

- EN: Declares or implements routines including `getInputOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getInputOffset`.
- CN: 这里声明或实现函数，例如 `getInputOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getInputOffset`。

### Lines 550-558

```cpp
        continue;
      }

      // Handle indirect calls -- could be direct calls with unknown targets
      // or secondary entry points of known functions, so check it is indirect
      // to be sure.
      if (opts::InstrumentCalls && BC.MIB->isIndirectCall(*I))
        instrumentIndirectTarget(BB, I, Function, FromOffset);
```

- EN: Declares or implements routines including `instrumentIndirectTarget`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `instrumentIndirectTarget`.
- CN: 这里声明或实现函数，例如 `instrumentIndirectTarget`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `instrumentIndirectTarget`。

### Lines 559-576

```cpp
    } // End of instructions loop

    // Instrument fallthroughs (when the direct jump instruction is missing)
    if (!HasUnconditionalBranch && !HasJumpTable && BB.succ_size() > 0 &&
        BB.size() > 0) {
      BinaryBasicBlock *FTBB = BB.getFallthrough();
      assert(FTBB && "expected valid fall-through basic block");
      auto I = BB.begin();
      auto LastInstr = BB.end();
      --LastInstr;
      while (LastInstr != I && BC.MIB->isPseudo(*LastInstr))
        --LastInstr;
      uint32_t FromOffset = 0;
      // The last instruction in the BB should have an annotation, except
      // if it was branching to the end of the function as a result of
      // __builtin_unreachable(), in which case it was deleted by fixBranches.
      // Ignore this case. FIXME: force fixBranches() to preserve the offset.
      if (!BC.MIB->getOffset(*LastInstr))
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 577-594

```cpp
        continue;
      FromOffset = *BC.MIB->getOffset(*LastInstr);

      // Do not instrument edges in the spanning tree
      auto STIt = STOutSet.find(&BB);
      if (STIt != STOutSet.end() && llvm::is_contained(STIt->second, FTBB)) {
        auto L = BC.scopeLock();
        createEdgeDescription(*FuncDesc, Function, FromOffset, BBToID[&BB],
                              Function, FTBB->getInputOffset(), BBToID[FTBB],
                              /*Instrumented=*/false);
        continue;
      }
      instrumentOneTarget(SplitWorklist, SplitInstrs, I, Function, BB,
                          FromOffset, Function, FTBB, FTBB->getInputOffset(),
                          IsLeafFunction, IsInvokeBlock, FuncDesc, BBToID[&BB],
                          BBToID[FTBB]);
    }
  } // End of BBs loop
```

- EN: Declares or implements routines including `getOffset`, `getInputOffset`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOffset`, `getInputOffset`.
- CN: 这里声明或实现函数，例如 `getOffset`, `getInputOffset`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOffset`, `getInputOffset`。

### Lines 595-606

```cpp

  // Instrument spanning tree leaves
  if (!opts::ConservativeInstrumentation) {
    for (auto BBI = Function.begin(), BBE = Function.end(); BBI != BBE; ++BBI) {
      BinaryBasicBlock &BB = *BBI;
      auto STIt = STOutSet.find(&BB);
      if (STIt == STOutSet.end() || STIt->second.empty())
        instrumentLeafNode(BB, BB.begin(), IsLeafFunction, *FuncDesc,
                           BBToID[&BB]);
    }
  }
```

- EN: Declares or implements routines including `instrumentLeafNode`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `instrumentLeafNode`.
- CN: 这里声明或实现函数，例如 `instrumentLeafNode`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `instrumentLeafNode`。

### Lines 607-616

```cpp
  // Consume list of critical edges: split them and add instrumentation to the
  // newly created BBs
  auto Iter = SplitInstrs.begin();
  for (std::pair<BinaryBasicBlock *, BinaryBasicBlock *> &BBPair :
       SplitWorklist) {
    BinaryBasicBlock *NewBB = Function.splitEdge(BBPair.first, BBPair.second);
    NewBB->addInstructions(Iter->begin(), Iter->end());
    ++Iter;
  }
```

- EN: Declares or implements routines including `addInstructions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addInstructions`.
- CN: 这里声明或实现函数，例如 `addInstructions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addInstructions`。

### Lines 617-634

```cpp
  // Unused now
  FuncDesc->EdgesSet.clear();
}

Error Instrumentation::runOnFunctions(BinaryContext &BC) {
  if (BC.usesBTI())
    return createFatalBOLTError(
        "BOLT-ERROR: instrumenting binaries using BTI is not supported.\n");
  /* BTI TODO:
   Instrumentation functions add indirect branches into the .text.injected
   section, see:
   - __bolt_instr_ind_call_handler
   - __bolt_instr_ind_tail_call_handler
   - __bolt_instr_ind_tailcall_handler_func
   - __bolt_start_trampoline
   - __bolt_fini_trampoline
   We cannot add BTIs to their targets when they are created, because the
   instrumentation snippets get added later to these targets, and the added BTI
```

- EN: Declares or implements routines including `runOnFunctions`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `runOnFunctions`。

### Lines 635-643

```cpp
   instruction will not be the first (rendering it useless).
   */

  const unsigned Flags = BinarySection::getFlags(/*IsReadOnly=*/false,
                                                 /*IsText=*/false,
                                                 /*IsAllocatable=*/true);
  BC.registerOrUpdateSection(".bolt.instr.counters", ELF::SHT_PROGBITS, Flags,
                             nullptr, 0, BC.RegularPageSize);
```

- EN: Declares or implements routines including `first`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `first`.
- CN: 这里声明或实现函数，例如 `first`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `first`。

### Lines 644-652

```cpp
  BC.registerOrUpdateNoteSection(".bolt.instr.tables", nullptr, 0,
                                 /*Alignment=*/1,
                                 /*IsReadOnly=*/true, ELF::SHT_NOTE);

  Summary->IndCallCounterFuncPtr =
      BC.Ctx->getOrCreateSymbol("__bolt_ind_call_counter_func_pointer");
  Summary->IndTailCallCounterFuncPtr =
      BC.Ctx->getOrCreateSymbol("__bolt_ind_tailcall_counter_func_pointer");
```

- EN: Declares or implements routines including `getOrCreateSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOrCreateSymbol`.
- CN: 这里声明或实现函数，例如 `getOrCreateSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOrCreateSymbol`。

### Lines 653-668

```cpp
  createAuxiliaryFunctions(BC);

  const bool HasInstrumentFuncsFilter = !opts::InstrumentFuncsFile.empty();
  StringSet<> InstrumentFuncsSet;
  if (HasInstrumentFuncsFilter) {
    std::ifstream FuncsFile(opts::InstrumentFuncsFile, std::ios::in);
    if (!FuncsFile)
      return createFatalBOLTError(Twine("instrument-funcs-file \"") +
                                  Twine(opts::InstrumentFuncsFile) +
                                  Twine("\" can't be opened."));
    std::string FuncName;
    while (std::getline(FuncsFile, FuncName))
      if (!FuncName.empty())
        InstrumentFuncsSet.insert(FuncName);
  }
```

- EN: Declares or implements routines including `createAuxiliaryFunctions`, `FuncsFile`, `Twine`. Notable symbols here include `createAuxiliaryFunctions`, `FuncsFile`, `Twine`.
- CN: 这里声明或实现函数，例如 `createAuxiliaryFunctions`, `FuncsFile`, `Twine`。这里较值得关注的符号包括 `createAuxiliaryFunctions`, `FuncsFile`, `Twine`。

### Lines 669-686

```cpp
  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    if (!BF.isSimple() || BF.isIgnored())
      return true;
    if (opts::InstrumentHotOnly && !BF.getKnownExecutionCount())
      return true;
    if (HasInstrumentFuncsFilter) {
      bool Found = false;
      for (const StringRef Name : BF.getNames()) {
        if (InstrumentFuncsSet.contains(Name)) {
          Found = true;
          break;
        }
      }
      if (!Found)
        return true;
    }
    return false;
  };
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 687-696

```cpp

  ParallelUtilities::WorkFuncWithAllocTy WorkFun =
      [&](BinaryFunction &BF, MCPlusBuilder::AllocatorIdTy AllocatorId) {
        instrumentFunction(BF, AllocatorId);
      };

  ParallelUtilities::runOnEachFunctionWithUniqueAllocId(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_QUADRATIC, WorkFun,
      SkipPredicate, "instrumentation", /* ForceSequential=*/true);
```

- EN: Declares or implements routines including `instrumentFunction`. Notable symbols here include `instrumentFunction`.
- CN: 这里声明或实现函数，例如 `instrumentFunction`。这里较值得关注的符号包括 `instrumentFunction`。

### Lines 697-708

```cpp
  if (BC.isMachO()) {
    if (BC.StartFunctionAddress) {
      BinaryFunction *Main =
          BC.getBinaryFunctionAtAddress(*BC.StartFunctionAddress);
      assert(Main && "Entry point function not found");
      BinaryBasicBlock &BB = Main->front();

      ErrorOr<BinarySection &> SetupSection =
          BC.getUniqueSectionByName("I__setup");
      if (!SetupSection)
        return createFatalBOLTError("Cannot find I__setup section\n");
```

- EN: Declares or implements routines including `assert`, `front`. Notable symbols here include `assert`, `front`.
- CN: 这里声明或实现函数，例如 `assert`, `front`。这里较值得关注的符号包括 `assert`, `front`。

### Lines 709-717

```cpp
      MCSymbol *Target = BC.registerNameAtAddress(
          "__bolt_instr_setup", SetupSection->getAddress(), 0, 0);
      MCInst NewInst;
      BC.MIB->createCall(NewInst, Target, BC.Ctx.get());
      BB.insertInstruction(BB.begin(), std::move(NewInst));
    } else {
      BC.errs() << "BOLT-WARNING: Entry point not found\n";
    }
```

- EN: Declares or implements routines including `getAddress`, `createCall`. Notable symbols here include `getAddress`, `createCall`.
- CN: 这里声明或实现函数，例如 `getAddress`, `createCall`。这里较值得关注的符号包括 `getAddress`, `createCall`。

### Lines 718-726

```cpp
    if (BinaryData *BD = BC.getBinaryDataByName("___GLOBAL_init_65535/1")) {
      BinaryFunction *Ctor = BC.getBinaryFunctionAtAddress(BD->getAddress());
      assert(Ctor && "___GLOBAL_init_65535 function not found");
      BinaryBasicBlock &BB = Ctor->front();
      ErrorOr<BinarySection &> FiniSection =
          BC.getUniqueSectionByName("I__fini");
      if (!FiniSection)
        return createFatalBOLTError("Cannot find I__fini section");
```

- EN: Declares or implements routines including `assert`, `front`. Notable symbols here include `assert`, `front`.
- CN: 这里声明或实现函数，例如 `assert`, `front`。这里较值得关注的符号包括 `assert`, `front`。

### Lines 727-737

```cpp
      MCSymbol *Target = BC.registerNameAtAddress(
          "__bolt_instr_fini", FiniSection->getAddress(), 0, 0);
      auto IsLEA = [&BC](const MCInst &Inst) { return BC.MIB->isLEA64r(Inst); };
      const auto LEA = std::find_if(
          std::next(llvm::find_if(reverse(BB), IsLEA)), BB.rend(), IsLEA);
      LEA->getOperand(4).setExpr(MCSymbolRefExpr::create(Target, *BC.Ctx));
    } else {
      BC.errs() << "BOLT-WARNING: ___GLOBAL_init_65535 not found\n";
    }
  }
```

- EN: Declares or implements routines including `getAddress`, `next`, `getOperand`. Notable symbols here include `getAddress`, `next`, `getOperand`.
- CN: 这里声明或实现函数，例如 `getAddress`, `next`, `getOperand`。这里较值得关注的符号包括 `getAddress`, `next`, `getOperand`。

### Lines 738-746

```cpp
  setupRuntimeLibrary(BC);
  return Error::success();
}

void Instrumentation::createAuxiliaryFunctions(BinaryContext &BC) {
  auto createSimpleFunction =
      [&](StringRef Title, InstructionListType Instrs) -> BinaryFunction * {
    BinaryFunction *Func = BC.createInjectedBinaryFunction(std::string(Title));
```

- EN: Declares or implements routines including `setupRuntimeLibrary`, `createAuxiliaryFunctions`. Notable symbols here include `setupRuntimeLibrary`, `createAuxiliaryFunctions`.
- CN: 这里声明或实现函数，例如 `setupRuntimeLibrary`, `createAuxiliaryFunctions`。这里较值得关注的符号包括 `setupRuntimeLibrary`, `createAuxiliaryFunctions`。

### Lines 747-757

```cpp
    std::vector<std::unique_ptr<BinaryBasicBlock>> BBs;
    BBs.emplace_back(Func->createBasicBlock());
    BBs.back()->addInstructions(Instrs.begin(), Instrs.end());
    BBs.back()->setCFIState(0);
    Func->insertBasicBlocks(nullptr, std::move(BBs),
                            /*UpdateLayout=*/true,
                            /*UpdateCFIState=*/false);
    Func->updateState(BinaryFunction::State::CFG_Finalized);
    return Func;
  };
```

- EN: Declares or implements routines including `insertBasicBlocks`, `updateState`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertBasicBlocks`, `updateState`.
- CN: 这里声明或实现函数，例如 `insertBasicBlocks`, `updateState`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertBasicBlocks`, `updateState`。

### Lines 758-768

```cpp
  // Here we are creating a set of functions to handle BB entry/exit.
  // IndCallHandlerExitBB contains instructions to finish handling traffic to an
  // indirect call. We pass it to createInstrumentedIndCallHandlerEntryBB(),
  // which will check if a pointer to runtime library traffic accounting
  // function was initialized (it is done during initialization of runtime
  // library). If it is so - calls it. Then this routine returns to normal
  // execution by jumping to exit BB.
  BinaryFunction *IndCallHandlerExitBB =
      createSimpleFunction("__bolt_instr_ind_call_handler",
                           BC.MIB->createInstrumentedIndCallHandlerExitBB());
```

- EN: Declares or implements routines including `createInstrumentedIndCallHandlerExitBB`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createInstrumentedIndCallHandlerExitBB`.
- CN: 这里声明或实现函数，例如 `createInstrumentedIndCallHandlerExitBB`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createInstrumentedIndCallHandlerExitBB`。

### Lines 769-778

```cpp
  IndCallHandlerExitBBFunction =
      createSimpleFunction("__bolt_instr_ind_call_handler_func",
                           BC.MIB->createInstrumentedIndCallHandlerEntryBB(
                               Summary->IndCallCounterFuncPtr,
                               IndCallHandlerExitBB->getSymbol(), &*BC.Ctx));

  BinaryFunction *IndTailCallHandlerExitBB = createSimpleFunction(
      "__bolt_instr_ind_tail_call_handler",
      BC.MIB->createInstrumentedIndTailCallHandlerExitBB());
```

- EN: Declares or implements routines including `getSymbol`, `createInstrumentedIndTailCallHandlerExitBB`. Notable symbols here include `getSymbol`, `createInstrumentedIndTailCallHandlerExitBB`.
- CN: 这里声明或实现函数，例如 `getSymbol`, `createInstrumentedIndTailCallHandlerExitBB`。这里较值得关注的符号包括 `getSymbol`, `createInstrumentedIndTailCallHandlerExitBB`。

### Lines 779-793

```cpp
  IndTailCallHandlerExitBBFunction = createSimpleFunction(
      "__bolt_instr_ind_tailcall_handler_func",
      BC.MIB->createInstrumentedIndCallHandlerEntryBB(
          Summary->IndTailCallCounterFuncPtr,
          IndTailCallHandlerExitBB->getSymbol(), &*BC.Ctx));

  createSimpleFunction("__bolt_num_counters_getter",
                       BC.MIB->createNumCountersGetter(BC.Ctx.get()));
  createSimpleFunction("__bolt_instr_locations_getter",
                       BC.MIB->createInstrLocationsGetter(BC.Ctx.get()));
  createSimpleFunction("__bolt_instr_tables_getter",
                       BC.MIB->createInstrTablesGetter(BC.Ctx.get()));
  createSimpleFunction("__bolt_instr_num_funcs_getter",
                       BC.MIB->createInstrNumFuncsGetter(BC.Ctx.get()));
```

- EN: Declares or implements routines including `getSymbol`, `createNumCountersGetter`, `createInstrLocationsGetter`, `createInstrTablesGetter`, `createInstrNumFuncsGetter`. Notable symbols here include `getSymbol`, `createNumCountersGetter`, `createInstrLocationsGetter`, `createInstrTablesGetter`, `createInstrNumFuncsGetter`.
- CN: 这里声明或实现函数，例如 `getSymbol`, `createNumCountersGetter`, `createInstrLocationsGetter`, `createInstrTablesGetter`, `createInstrNumFuncsGetter`。这里较值得关注的符号包括 `getSymbol`, `createNumCountersGetter`, `createInstrLocationsGetter`, `createInstrTablesGetter`, `createInstrNumFuncsGetter`。

### Lines 794-811

```cpp
  if (BC.isELF()) {
    if (BC.StartFunctionAddress) {
      BinaryFunction *Start =
          BC.getBinaryFunctionAtAddress(*BC.StartFunctionAddress);
      assert(Start && "Entry point function not found");
      const MCSymbol *StartSym = Start->getSymbol();
      createSimpleFunction(
          "__bolt_start_trampoline",
          BC.MIB->createSymbolTrampoline(StartSym, BC.Ctx.get()));
    }
    if (BC.FiniFunctionAddress) {
      BinaryFunction *Fini =
          BC.getBinaryFunctionAtAddress(*BC.FiniFunctionAddress);
      assert(Fini && "Finalization function not found");
      const MCSymbol *FiniSym = Fini->getSymbol();
      createSimpleFunction(
          "__bolt_fini_trampoline",
          BC.MIB->createSymbolTrampoline(FiniSym, BC.Ctx.get()));
```

- EN: Declares or implements routines including `assert`, `getSymbol`, `createSymbolTrampoline`. Notable symbols here include `assert`, `getSymbol`, `createSymbolTrampoline`.
- CN: 这里声明或实现函数，例如 `assert`, `getSymbol`, `createSymbolTrampoline`。这里较值得关注的符号包括 `assert`, `getSymbol`, `createSymbolTrampoline`。

### Lines 812-823

```cpp
    } else {
      // Create dummy return function for trampoline to avoid issues
      // with unknown symbol in runtime library. E.g. for static PIE
      // executable
      createSimpleFunction("__bolt_fini_trampoline",
                           BC.MIB->createReturnInstructionList(BC.Ctx.get()));
    }
    if (BC.isAArch64())
      BC.MIB->createInstrCounterIncrFunc(BC);
  }
}
```

- EN: Declares or implements routines including `createReturnInstructionList`, `createInstrCounterIncrFunc`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createReturnInstructionList`, `createInstrCounterIncrFunc`.
- CN: 这里声明或实现函数，例如 `createReturnInstructionList`, `createInstrCounterIncrFunc`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createReturnInstructionList`, `createInstrCounterIncrFunc`。

### Lines 824-841

```cpp
void Instrumentation::setupRuntimeLibrary(BinaryContext &BC) {
  uint32_t FuncDescSize = Summary->getFDSize();

  BC.outs() << "BOLT-INSTRUMENTER: Number of indirect call site descriptors: "
            << Summary->IndCallDescriptions.size() << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Number of indirect call target descriptors: "
            << Summary->IndCallTargetDescriptions.size() << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Number of function descriptors: "
            << Summary->FunctionDescriptions.size() << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Number of branch counters: "
            << BranchCounters << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Number of ST leaf node counters: "
            << LeafNodeCounters << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Number of direct call counters: "
            << DirectCallCounters << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Total number of counters: "
            << Summary->Counters.size() << "\n";
  BC.outs() << "BOLT-INSTRUMENTER: Total size of counters: "
```

- EN: Declares or implements routines including `setupRuntimeLibrary`, `getFDSize`. Notable symbols here include `setupRuntimeLibrary`, `getFDSize`.
- CN: 这里声明或实现函数，例如 `setupRuntimeLibrary`, `getFDSize`。这里较值得关注的符号包括 `setupRuntimeLibrary`, `getFDSize`。

### Lines 842-855

```cpp
            << (Summary->Counters.size() * 8)
            << " bytes (static alloc memory)\n";
  BC.outs() << "BOLT-INSTRUMENTER: Total size of string table emitted: "
            << Summary->StringTable.size() << " bytes in file\n";
  BC.outs() << "BOLT-INSTRUMENTER: Total size of descriptors: "
            << (FuncDescSize +
                Summary->IndCallDescriptions.size() *
                    sizeof(IndCallDescription) +
                Summary->IndCallTargetDescriptions.size() *
                    sizeof(IndCallTargetDescription))
            << " bytes in file\n";
  BC.outs() << "BOLT-INSTRUMENTER: Profile will be saved to file "
            << opts::InstrumentationFilename << "\n";
```

- EN: Declares or implements routines including `bytes`. Notable symbols here include `bytes`.
- CN: 这里声明或实现函数，例如 `bytes`。这里较值得关注的符号包括 `bytes`。

### Lines 856-862

```cpp
  InstrumentationRuntimeLibrary *RtLibrary =
      static_cast<InstrumentationRuntimeLibrary *>(BC.getRuntimeLibrary());
  assert(RtLibrary && "instrumentation runtime library object must be set");
  RtLibrary->setSummary(std::move(Summary));
}
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Declares or implements routines including `assert`, `setSummary`. Notable symbols here include `assert`, `setSummary`, `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里声明或实现函数，例如 `assert`, `setSummary`。这里较值得关注的符号包括 `assert`, `setSummary`, `bolt`, `llvm`。

## Key Concepts / 关键概念

- `init`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `activity`: function or method entry point / 函数或方法入口
- `outs`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `opts`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/Instrumentation.h`, `bolt/Core/ParallelUtilities.h`, `bolt/RuntimeLibs/InstrumentationRuntimeLibrary.h`, `bolt/Utils/CommandLineOpts.h`, `bolt/Utils/Utils.h`
- LLVM headers / LLVM 头文件: `llvm/ADT/StringSet.h`, `llvm/Support/CommandLine.h`, `llvm/Support/RWMutex.h`
- System headers / 系统头文件: `fstream`, `queue`, `stack`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
