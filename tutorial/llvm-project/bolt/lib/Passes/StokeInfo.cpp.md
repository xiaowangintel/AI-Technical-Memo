# StokeInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/StokeInfo.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/StokeInfo.cpp This file implements the StokeInfo class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/StokeInfo.cpp This file implements the StokeInfo class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/StokeInfo.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the StokeInfo class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "bolt/Passes/StokeInfo.h"
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Passes/DataflowInfoManager.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "stoke"

using namespace llvm;
using namespace bolt;
```

- EN: Pulls in 4 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 23-32

```cpp
namespace opts {
static cl::OptionCategory StokeOptCategory("STOKE pass options");

static cl::opt<std::string>
StokeOutputDataFilename("stoke-out",
  cl::desc("output data (.csv) for Stoke's use"),
  cl::Optional,
  cl::cat(StokeOptCategory));
}
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `StokeOptCategory`, `desc`, `cat`. Notable symbols here include `StokeOptCategory`, `desc`, `cat`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `StokeOptCategory`, `desc`, `cat`。这里较值得关注的符号包括 `StokeOptCategory`, `desc`, `cat`, `opts`。

### Lines 33-45

```cpp
namespace llvm {
namespace bolt {

void getRegNameFromBitVec(const BinaryContext &BC, const BitVector &RegV,
                          std::set<std::string> *NameVec = nullptr) {
  for (int RegIdx : RegV.set_bits()) {
    LLVM_DEBUG(dbgs() << BC.MRI->getName(RegIdx) << " ");
    if (NameVec)
      NameVec->insert(std::string(BC.MRI->getName(RegIdx)));
  }
  LLVM_DEBUG(dbgs() << "\n");
}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `LLVM_DEBUG`, `insert`. Notable symbols here include `LLVM_DEBUG`, `insert`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `LLVM_DEBUG`, `insert`。这里较值得关注的符号包括 `LLVM_DEBUG`, `insert`, `llvm`, `bolt`。

### Lines 46-58

```cpp
void StokeInfo::checkInstr(const BinaryFunction &BF, StokeFuncInfo &FuncInfo) {
  MCPlusBuilder *MIB = BF.getBinaryContext().MIB.get();
  BitVector RegV(NumRegs, false);
  for (const BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    if (BB->empty())
      continue;

    // Skip function with exception handling.
    if (BB->throw_size() || BB->lp_size()) {
      FuncInfo.Omitted = true;
      return;
    }
```

- EN: Declares or implements routines including `checkInstr`, `RegV`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `checkInstr`, `RegV`.
- CN: 这里声明或实现函数，例如 `checkInstr`, `RegV`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `checkInstr`, `RegV`。

### Lines 59-76

```cpp
    for (const MCInst &It : *BB) {
      if (MIB->isPseudo(It))
        continue;
      // skip function with exception handling yet
      if (MIB->isInvoke(It)) {
        FuncInfo.Omitted = true;
        return;
      }
      // check if this function contains call instruction
      if (MIB->isCall(It)) {
        FuncInfo.HasCall = true;
        const MCSymbol *TargetSymbol = MIB->getTargetSymbol(It);
        // if it is an indirect call, skip
        if (TargetSymbol == nullptr) {
          FuncInfo.Omitted = true;
          return;
        }
      }
```

- EN: Declares or implements routines including `getTargetSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`。

### Lines 77-86

```cpp
      // check if this function modify stack or heap
      // TODO: more accurate analysis
      bool IsPush = MIB->isPush(It);
      bool IsRipAddr = MIB->hasPCRelOperand(It);
      if (IsPush)
        FuncInfo.StackOut = true;

      if (MIB->mayStore(It) && !IsPush && !IsRipAddr)
        FuncInfo.HeapOut = true;
```

- EN: Declares or implements routines including `isPush`, `hasPCRelOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isPush`, `hasPCRelOperand`.
- CN: 这里声明或实现函数，例如 `isPush`, `hasPCRelOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isPush`, `hasPCRelOperand`。

### Lines 87-95

```cpp
      if (IsRipAddr)
        FuncInfo.HasRipAddr = true;
    } // end of for (auto &It : ...)
  }   // end of for (auto *BB : ...)
}

bool StokeInfo::checkFunction(BinaryFunction &BF, DataflowInfoManager &DInfo,
                              RegAnalysis &RA, StokeFuncInfo &FuncInfo) {
```

- EN: Declares or implements routines including `for`. Notable symbols here include `for`.
- CN: 这里声明或实现函数，例如 `for`。这里较值得关注的符号包括 `for`。

### Lines 96-111

```cpp
  std::string Name = BF.getSymbol()->getName().str();

  if (!BF.isSimple() || BF.isMultiEntry() || BF.empty())
    return false;
  BF.getBinaryContext().outs()
      << " STOKE-INFO: analyzing function " << Name << "\n";

  FuncInfo.FuncName = Name;
  FuncInfo.Offset = BF.getFileOffset();
  FuncInfo.Size = BF.getMaxSize();
  FuncInfo.NumInstrs = BF.getNumNonPseudos();
  FuncInfo.NumBlocks = BF.size();
  // early stop for large functions
  if (FuncInfo.NumInstrs > 500)
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 112-122

```cpp
  FuncInfo.IsLoopFree = BF.isLoopFree();
  if (!FuncInfo.IsLoopFree) {
    const BinaryLoopInfo &BLI = BF.getLoopInfo();
    FuncInfo.NumLoops = BLI.OuterLoops;
    FuncInfo.MaxLoopDepth = BLI.MaximumDepth;
  }

  FuncInfo.HotSize = BF.estimateHotSize();
  FuncInfo.TotalSize = BF.estimateSize();
  FuncInfo.Score = BF.getFunctionScore();
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 123-132

```cpp
  checkInstr(BF, FuncInfo);

  // register analysis
  BinaryBasicBlock &EntryBB = BF.front();
  assert(EntryBB.isEntryPoint() && "Weird, this should be the entry block!");

  MCInst *FirstNonPseudo = EntryBB.getFirstNonPseudoInstr();
  if (!FirstNonPseudo)
    return false;
```

- EN: Declares or implements routines including `checkInstr`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `checkInstr`, `assert`.
- CN: 这里声明或实现函数，例如 `checkInstr`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `checkInstr`, `assert`。

### Lines 133-143

```cpp
  LLVM_DEBUG(dbgs() << "\t [DefIn]\n\t ");
  BitVector LiveInBV =
      *(DInfo.getLivenessAnalysis().getStateAt(FirstNonPseudo));
  LiveInBV &= DefaultDefInMask;
  getRegNameFromBitVec(BF.getBinaryContext(), LiveInBV, &FuncInfo.DefIn);

  LLVM_DEBUG(dbgs() << "\t [LiveOut]\n\t ");
  BitVector LiveOutBV = RA.getFunctionClobberList(&BF);
  LiveOutBV &= DefaultLiveOutMask;
  getRegNameFromBitVec(BF.getBinaryContext(), LiveOutBV, &FuncInfo.LiveOut);
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getRegNameFromBitVec`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `getRegNameFromBitVec`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getRegNameFromBitVec`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `getRegNameFromBitVec`。

### Lines 144-153

```cpp
  BF.getBinaryContext().outs() << " STOKE-INFO: end function \n";
  return true;
}

Error StokeInfo::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is specific to X86\n";
    exit(1);
  }
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`. Notable symbols here include `runOnFunctions`, `exit`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`。这里较值得关注的符号包括 `runOnFunctions`, `exit`。

### Lines 154-163

```cpp
  BC.outs() << "STOKE-INFO: begin of stoke pass\n";

  std::ofstream Outfile;
  if (!opts::StokeOutputDataFilename.empty()) {
    Outfile.open(opts::StokeOutputDataFilename);
  } else {
    BC.errs() << "STOKE-INFO: output file is required\n";
    return Error::success();
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 164-171

```cpp
  // check some context meta data
  LLVM_DEBUG(dbgs() << "\tTarget: " << BC.TheTarget->getName() << "\n");
  LLVM_DEBUG(dbgs() << "\tTripleName " << BC.TripleName << "\n");
  LLVM_DEBUG(dbgs() << "\tgetNumRegs " << BC.MRI->getNumRegs() << "\n");

  BinaryFunctionCallGraph CG = buildCallGraph(BC);
  RegAnalysis RA(BC, &BC.getBinaryFunctions(), &CG);
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `buildCallGraph`, `RA`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `buildCallGraph`, `RA`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `buildCallGraph`, `RA`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `buildCallGraph`, `RA`。

### Lines 172-180

```cpp
  NumRegs = BC.MRI->getNumRegs();
  assert(NumRegs > 0 && "STOKE-INFO: the target register number is incorrect!");

  DefaultDefInMask.resize(NumRegs, false);
  DefaultLiveOutMask.resize(NumRegs, false);

  BC.MIB->getDefaultDefIn(DefaultDefInMask);
  BC.MIB->getDefaultLiveOut(DefaultLiveOutMask);
```

- EN: Declares or implements routines including `getNumRegs`, `assert`, `getDefaultDefIn`, `getDefaultLiveOut`. Notable symbols here include `getNumRegs`, `assert`, `getDefaultDefIn`, `getDefaultLiveOut`.
- CN: 这里声明或实现函数，例如 `getNumRegs`, `assert`, `getDefaultDefIn`, `getDefaultLiveOut`。这里较值得关注的符号包括 `getNumRegs`, `assert`, `getDefaultDefIn`, `getDefaultLiveOut`。

### Lines 181-193

```cpp
  getRegNameFromBitVec(BC, DefaultDefInMask);
  getRegNameFromBitVec(BC, DefaultLiveOutMask);

  StokeFuncInfo FuncInfo;
  // analyze all functions
  FuncInfo.printCsvHeader(Outfile);
  for (auto &BF : BC.getBinaryFunctions()) {
    DataflowInfoManager DInfo(BF.second, &RA, nullptr);
    FuncInfo.reset();
    if (checkFunction(BF.second, DInfo, RA, FuncInfo))
      FuncInfo.printData(Outfile);
  }
```

- EN: Declares or implements routines including `getRegNameFromBitVec`, `DInfo`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getRegNameFromBitVec`, `DInfo`.
- CN: 这里声明或实现函数，例如 `getRegNameFromBitVec`, `DInfo`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getRegNameFromBitVec`, `DInfo`。

### Lines 194-199

```cpp
  BC.outs() << "STOKE-INFO: end of stoke pass\n";
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `StokeOptCategory`: function or method entry point / 函数或方法入口
- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `LLVM_DEBUG`: function or method entry point / 函数或方法入口
- `insert`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/StokeInfo.h`, `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Passes/DataflowInfoManager.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
