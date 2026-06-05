# JTFootprintReduction.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/JTFootprintReduction.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/JTFootprintReduction.cpp This file implements JTFootprintReduction class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/JTFootprintReduction.cpp This file implements JTFootprintReduction class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/JTFootprintReduction.cpp -------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements JTFootprintReduction class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "bolt/Passes/JTFootprintReduction.h"
#include "bolt/Core/BinaryFunctionCallGraph.h"
#include "bolt/Passes/DataflowInfoManager.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "JT"

using namespace llvm;
using namespace bolt;
```

- EN: Pulls in 4 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 23-30

```cpp
namespace opts {

extern cl::OptionCategory BoltOptCategory;

extern cl::opt<unsigned> Verbosity;

extern cl::opt<JumpTableSupportLevel> JumpTables;
```

- EN: Works inside namespace scope `opts` to organize symbols. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`。

### Lines 31-38

```cpp
static cl::opt<bool> JTFootprintOnlyPIC(
    "jt-footprint-optimize-for-icache",
    cl::desc("with jt-footprint-reduction, only process PIC jumptables and turn"
             " off other transformations that increase code size"),
    cl::init(false), cl::ZeroOrMore, cl::cat(BoltOptCategory));

} // namespace opts
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `init`. Notable symbols here include `init`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `init`。这里较值得关注的符号包括 `init`, `opts`。

### Lines 39-46

```cpp
namespace llvm {
namespace bolt {

void JTFootprintReduction::checkOpportunities(BinaryFunction &Function,
                                              DataflowInfoManager &Info) {
  BinaryContext &BC = Function.getBinaryContext();
  std::map<JumpTable *, uint64_t> AllJTs;
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Notable symbols here include `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 47-55

```cpp
  for (BinaryBasicBlock &BB : Function) {
    for (MCInst &Inst : BB) {
      JumpTable *JumpTable = Function.getJumpTable(Inst);
      if (!JumpTable)
        continue;

      AllJTs[JumpTable] += BB.getKnownExecutionCount();
      ++IndJmps;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 56-73

```cpp
      if (BlacklistedJTs.count(JumpTable)) {
        ++IndJmpsDenied;
        continue;
      }

      uint64_t Scale;
      // Try a standard indirect jump matcher
      std::unique_ptr<MCPlusBuilder::MCInstMatcher> IndJmpMatcher =
          BC.MIB->matchIndJmp(BC.MIB->matchAnyOperand(),
                              BC.MIB->matchImm(Scale), BC.MIB->matchReg(),
                              BC.MIB->matchAnyOperand());
      if (!opts::JTFootprintOnlyPIC &&
          IndJmpMatcher->match(*BC.MRI, *BC.MIB,
                               MutableArrayRef<MCInst>(&*BB.begin(), &Inst + 1),
                               -1) &&
          Scale == 8) {
        if (Info.getLivenessAnalysis().scavengeRegAfter(&Inst))
          continue;
```

- EN: Declares or implements routines including `matchIndJmp`, `matchImm`, `matchAnyOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `matchIndJmp`, `matchImm`, `matchAnyOperand`.
- CN: 这里声明或实现函数，例如 `matchIndJmp`, `matchImm`, `matchAnyOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `matchIndJmp`, `matchImm`, `matchAnyOperand`。

### Lines 74-91

```cpp
        BlacklistedJTs.insert(JumpTable);
        ++IndJmpsDenied;
        ++NumJTsNoReg;
        continue;
      }

      // Try a PIC matcher. The pattern we are looking for is a PIC JT ind jmp:
      //    addq    %rdx, %rsi
      //    addq    %rdx, %rdi
      //    leaq    DATAat0x402450(%rip), %r11
      //    movslq  (%r11,%rdx,4), %rcx
      //    addq    %r11, %rcx
      //    jmpq    *%rcx # JUMPTABLE @0x402450
      MCPhysReg BaseReg1;
      MCPhysReg BaseReg2;
      uint64_t Offset;
      std::unique_ptr<MCPlusBuilder::MCInstMatcher> PICIndJmpMatcher =
          BC.MIB->matchIndJmp(BC.MIB->matchAdd(
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 92-109

```cpp
              BC.MIB->matchReg(BaseReg1),
              BC.MIB->matchLoad(BC.MIB->matchReg(BaseReg2),
                                BC.MIB->matchImm(Scale), BC.MIB->matchReg(),
                                BC.MIB->matchImm(Offset))));
      std::unique_ptr<MCPlusBuilder::MCInstMatcher> PICBaseAddrMatcher =
          BC.MIB->matchIndJmp(
              BC.MIB->matchAdd(BC.MIB->matchLoadAddr(BC.MIB->matchSymbol()),
                               BC.MIB->matchAnyOperand()));
      if (!PICIndJmpMatcher->match(
              *BC.MRI, *BC.MIB,
              MutableArrayRef<MCInst>(&*BB.begin(), &Inst + 1), -1) ||
          Scale != 4 || BaseReg1 != BaseReg2 || Offset != 0 ||
          !PICBaseAddrMatcher->match(
              *BC.MRI, *BC.MIB,
              MutableArrayRef<MCInst>(&*BB.begin(), &Inst + 1), -1)) {
        BlacklistedJTs.insert(JumpTable);
        ++IndJmpsDenied;
        ++NumJTsBadMatch;
```

- EN: Declares or implements routines including `matchReg`, `matchLoad`, `matchImm`, `matchAdd`, `matchAnyOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `matchReg`, `matchLoad`, `matchImm`, `matchAdd`, `matchAnyOperand`.
- CN: 这里声明或实现函数，例如 `matchReg`, `matchLoad`, `matchImm`, `matchAdd`, `matchAnyOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `matchReg`, `matchLoad`, `matchImm`, `matchAdd`, `matchAnyOperand`。

### Lines 110-127

```cpp
        continue;
      }
    }
  }

  // Statistics only
  for (const auto &JTFreq : AllJTs) {
    JumpTable *JT = JTFreq.first;
    uint64_t CurScore = JTFreq.second;
    TotalJTScore += CurScore;
    if (!BlacklistedJTs.count(JT)) {
      OptimizedScore += CurScore;
      if (JT->EntrySize == 8)
        BytesSaved += JT->getSize() >> 1;
    }
  }
  TotalJTs += AllJTs.size();
  TotalJTsDenied += BlacklistedJTs.size();
```

- EN: Declares or implements routines including `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getSize`.
- CN: 这里声明或实现函数，例如 `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getSize`。

### Lines 128-135

```cpp
}

bool JTFootprintReduction::tryOptimizeNonPIC(
    BinaryContext &BC, BinaryBasicBlock &BB, BinaryBasicBlock::iterator Inst,
    uint64_t JTAddr, JumpTable *JumpTable, DataflowInfoManager &Info) {
  if (opts::JTFootprintOnlyPIC)
    return false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 136-148

```cpp
  MCOperand Base;
  uint64_t Scale;
  MCPhysReg Index;
  MCOperand Offset;
  std::unique_ptr<MCPlusBuilder::MCInstMatcher> IndJmpMatcher =
      BC.MIB->matchIndJmp(BC.MIB->matchAnyOperand(Base),
                          BC.MIB->matchImm(Scale), BC.MIB->matchReg(Index),
                          BC.MIB->matchAnyOperand(Offset));
  if (!IndJmpMatcher->match(*BC.MRI, *BC.MIB,
                            MutableArrayRef<MCInst>(&*BB.begin(), &*Inst + 1),
                            -1))
    return false;
```

- EN: Declares or implements routines including `matchIndJmp`, `matchImm`, `matchAnyOperand`. Notable symbols here include `matchIndJmp`, `matchImm`, `matchAnyOperand`.
- CN: 这里声明或实现函数，例如 `matchIndJmp`, `matchImm`, `matchAnyOperand`。这里较值得关注的符号包括 `matchIndJmp`, `matchImm`, `matchAnyOperand`。

### Lines 149-159

```cpp
  assert(Scale == 8 && "Wrong scale");

  Scale = 4;
  IndJmpMatcher->annotate(*BC.MIB, "DeleteMe");

  LivenessAnalysis &LA = Info.getLivenessAnalysis();
  MCPhysReg Reg = LA.scavengeRegAfter(&*Inst);
  assert(Reg != 0 && "Register scavenger failed!");
  MCOperand RegOp = MCOperand::createReg(Reg);
  SmallVector<MCInst, 4> NewFrag;
```

- EN: Declares or implements routines including `assert`, `annotate`, `createReg`. Notable symbols here include `assert`, `annotate`, `createReg`.
- CN: 这里声明或实现函数，例如 `assert`, `annotate`, `createReg`。这里较值得关注的符号包括 `assert`, `annotate`, `createReg`。

### Lines 160-169

```cpp
  BC.MIB->createIJmp32Frag(NewFrag, Base, MCOperand::createImm(Scale),
                           MCOperand::createReg(Index), Offset, RegOp);
  BC.MIB->setJumpTable(NewFrag.back(), JTAddr, Index);

  JumpTable->OutputEntrySize = 4;

  BB.replaceInstruction(Inst, NewFrag.begin(), NewFrag.end());
  return true;
}
```

- EN: Declares or implements routines including `createIJmp32Frag`, `createReg`, `setJumpTable`. Notable symbols here include `createIJmp32Frag`, `createReg`, `setJumpTable`.
- CN: 这里声明或实现函数，例如 `createIJmp32Frag`, `createReg`, `setJumpTable`。这里较值得关注的符号包括 `createIJmp32Frag`, `createReg`, `setJumpTable`。

### Lines 170-187

```cpp
bool JTFootprintReduction::tryOptimizePIC(BinaryContext &BC,
                                          BinaryBasicBlock &BB,
                                          BinaryBasicBlock::iterator Inst,
                                          uint64_t JTAddr, JumpTable *JumpTable,
                                          DataflowInfoManager &Info) {
  MCPhysReg BaseReg;
  uint64_t Scale;
  MCPhysReg Index;
  MCOperand Offset;
  MCOperand JumpTableRef;
  std::unique_ptr<MCPlusBuilder::MCInstMatcher> PICIndJmpMatcher =
      BC.MIB->matchIndJmp(BC.MIB->matchAdd(
          BC.MIB->matchLoadAddr(BC.MIB->matchAnyOperand(JumpTableRef)),
          BC.MIB->matchLoad(BC.MIB->matchReg(BaseReg), BC.MIB->matchImm(Scale),
                            BC.MIB->matchReg(Index),
                            BC.MIB->matchAnyOperand())));
  if (!PICIndJmpMatcher->match(
          *BC.MRI, *BC.MIB, MutableArrayRef<MCInst>(&*BB.begin(), &*Inst + 1),
```

- EN: Declares or implements routines including `matchLoadAddr`, `matchLoad`, `matchReg`, `matchAnyOperand`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `matchLoadAddr`, `matchLoad`, `matchReg`, `matchAnyOperand`.
- CN: 这里声明或实现函数，例如 `matchLoadAddr`, `matchLoad`, `matchReg`, `matchAnyOperand`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `matchLoadAddr`, `matchLoad`, `matchReg`, `matchAnyOperand`。

### Lines 188-197

```cpp
          -1))
    return false;

  assert(Scale == 4 && "Wrong scale");

  PICIndJmpMatcher->annotate(*BC.MIB, "DeleteMe");

  MCOperand RegOp = MCOperand::createReg(BaseReg);
  SmallVector<MCInst, 4> NewFrag;
```

- EN: Declares or implements routines including `assert`, `annotate`, `createReg`. Notable symbols here include `assert`, `annotate`, `createReg`.
- CN: 这里声明或实现函数，例如 `assert`, `annotate`, `createReg`。这里较值得关注的符号包括 `assert`, `annotate`, `createReg`。

### Lines 198-206

```cpp
  BC.MIB->createIJmp32Frag(NewFrag, MCOperand::createReg(0),
                           MCOperand::createImm(Scale),
                           MCOperand::createReg(Index), JumpTableRef, RegOp);
  BC.MIB->setJumpTable(NewFrag.back(), JTAddr, Index);

  JumpTable->OutputEntrySize = 4;
  // DePICify
  JumpTable->Type = JumpTable::JTT_NORMAL;
```

- EN: Declares or implements routines including `createIJmp32Frag`, `createImm`, `createReg`, `setJumpTable`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createIJmp32Frag`, `createImm`, `createReg`, `setJumpTable`.
- CN: 这里声明或实现函数，例如 `createIJmp32Frag`, `createImm`, `createReg`, `setJumpTable`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createIJmp32Frag`, `createImm`, `createReg`, `setJumpTable`。

### Lines 207-217

```cpp
  BB.replaceInstruction(Inst, NewFrag.begin(), NewFrag.end());
  return true;
}

void JTFootprintReduction::optimizeFunction(BinaryFunction &Function,
                                            DataflowInfoManager &Info) {
  BinaryContext &BC = Function.getBinaryContext();
  for (BinaryBasicBlock &BB : Function) {
    if (!BB.getNumNonPseudos())
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 218-228

```cpp
    auto IndJmpRI = BB.getLastNonPseudo();
    auto IndJmp = std::prev(IndJmpRI.base());
    const uint64_t JTAddr = BC.MIB->getJumpTable(*IndJmp);

    if (!JTAddr)
      continue;

    JumpTable *JumpTable = Function.getJumpTable(*IndJmp);
    if (BlacklistedJTs.count(JumpTable))
      continue;
```

- EN: Declares or implements routines including `prev`, `getJumpTable`. Notable symbols here include `prev`, `getJumpTable`.
- CN: 这里声明或实现函数，例如 `prev`, `getJumpTable`。这里较值得关注的符号包括 `prev`, `getJumpTable`。

### Lines 229-237

```cpp
    if (tryOptimizeNonPIC(BC, BB, IndJmp, JTAddr, JumpTable, Info) ||
        tryOptimizePIC(BC, BB, IndJmp, JTAddr, JumpTable, Info)) {
      Modified.insert(&Function);
      continue;
    }

    llvm_unreachable("Should either optimize PIC or NonPIC successfully");
  }
```

- EN: Declares or implements routines including `tryOptimizePIC`, `llvm_unreachable`. Notable symbols here include `tryOptimizePIC`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `tryOptimizePIC`, `llvm_unreachable`。这里较值得关注的符号包括 `tryOptimizePIC`, `llvm_unreachable`。

### Lines 238-248

```cpp
  if (!Modified.count(&Function))
    return;

  for (BinaryBasicBlock &BB : Function)
    for (auto I = BB.begin(); I != BB.end();)
      if (BC.MIB->hasAnnotation(*I, "DeleteMe"))
        I = BB.eraseInstruction(I);
      else
        ++I;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 249-257

```cpp
Error JTFootprintReduction::runOnFunctions(BinaryContext &BC) {
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is supported only on X86\n";
    exit(1);
  }

  if (opts::JumpTables == JTS_BASIC && BC.HasRelocations)
    return Error::success();
```

- EN: Declares or implements routines including `runOnFunctions`, `exit`. Notable symbols here include `runOnFunctions`, `exit`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `exit`。这里较值得关注的符号包括 `runOnFunctions`, `exit`。

### Lines 258-266

```cpp
  std::unique_ptr<RegAnalysis> RA;
  std::unique_ptr<BinaryFunctionCallGraph> CG;
  if (!opts::JTFootprintOnlyPIC) {
    CG.reset(new BinaryFunctionCallGraph(buildCallGraph(BC)));
    RA.reset(new RegAnalysis(BC, &BC.getBinaryFunctions(), &*CG));
  }
  for (auto &BFIt : BC.getBinaryFunctions()) {
    BinaryFunction &Function = BFIt.second;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 267-278

```cpp
    if (!Function.isSimple() || Function.isIgnored())
      continue;

    if (Function.getKnownExecutionCount() == 0)
      continue;

    DataflowInfoManager Info(Function, RA.get(), nullptr);
    BlacklistedJTs.clear();
    checkOpportunities(Function, Info);
    optimizeFunction(Function, Info);
  }
```

- EN: Declares or implements routines including `Info`, `checkOpportunities`, `optimizeFunction`. Notable symbols here include `Info`, `checkOpportunities`, `optimizeFunction`.
- CN: 这里声明或实现函数，例如 `Info`, `checkOpportunities`, `optimizeFunction`。这里较值得关注的符号包括 `Info`, `checkOpportunities`, `optimizeFunction`。

### Lines 279-296

```cpp
  if (TotalJTs == TotalJTsDenied) {
    BC.outs() << "BOLT-INFO: JT Footprint reduction: no changes were made.\n";
    return Error::success();
  }

  BC.outs() << "BOLT-INFO: JT Footprint reduction stats (simple funcs only):\n";
  if (OptimizedScore)
    BC.outs() << format("\t   %.2lf%%", (OptimizedScore * 100.0 / TotalJTScore))
              << " of dynamic JT entries were reduced.\n";
  BC.outs() << "\t   " << TotalJTs - TotalJTsDenied << " of " << TotalJTs
            << " jump tables affected.\n";
  BC.outs() << "\t   " << IndJmps - IndJmpsDenied << " of " << IndJmps
            << " indirect jumps to JTs affected.\n";
  BC.outs() << "\t   " << NumJTsBadMatch
            << " JTs discarded due to unsupported jump pattern.\n";
  BC.outs() << "\t   " << NumJTsNoReg
            << " JTs discarded due to register unavailability.\n";
  BC.outs() << "\t   " << BytesSaved << " bytes saved.\n";
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 297-301

```cpp
  return Error::success();
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `init`: function or method entry point / 函数或方法入口
- `matchIndJmp`: function or method entry point / 函数或方法入口
- `matchImm`: function or method entry point / 函数或方法入口
- `matchAnyOperand`: function or method entry point / 函数或方法入口
- `matchReg`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/JTFootprintReduction.h`, `bolt/Core/BinaryFunctionCallGraph.h`, `bolt/Passes/DataflowInfoManager.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
