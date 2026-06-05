# RetpolineInsertion.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/RetpolineInsertion.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/RetpolineInsertion.cpp This file implements RetpolineInsertion class, which replaces indirect branches (calls and jumps) with calls to retpolines to protect against branch target injection attacks.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/RetpolineInsertion.cpp This file implements RetpolineInsertion class, which replaces indirect branches (calls and jumps) with calls to retpolines to protect against branch target injection attacks.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- bolt/Passes/RetpolineInsertion.cpp ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements RetpolineInsertion class, which replaces indirect
// branches (calls and jumps) with calls to retpolines to protect against branch
// target injection attacks.
// A unique retpoline is created for each register holding the address of the
// callee, if the callee address is in memory %r11 is used if available to
// hold the address of the callee before calling the retpoline, otherwise an
// address pattern specific retpoline is called where the callee address is
// loaded inside the retpoline.
// The user can determine when to assume %r11 available using r11-availability
// option, by default %r11 is assumed not available.
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-27

```cpp
// Adding lfence instruction to the body of the speculate code is enabled by
// default and can be controlled by the user using retpoline-lfence option.
//
//===----------------------------------------------------------------------===//

#include "bolt/Passes/RetpolineInsertion.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/Support/raw_ostream.h"
```

- EN: Pulls in 3 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 28-35

```cpp
#define DEBUG_TYPE "bolt-retpoline"

using namespace llvm;
using namespace bolt;
namespace opts {

extern cl::OptionCategory BoltCategory;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 36-45

```cpp
static llvm::cl::opt<bool>
    InsertRetpolines("insert-retpolines",
                     cl::desc("run retpoline insertion pass"),
                     cl::cat(BoltCategory));

static llvm::cl::opt<bool> RetpolineLfence(
    "retpoline-lfence",
    cl::desc("determine if lfence instruction should exist in the retpoline"),
    cl::init(true), cl::ZeroOrMore, cl::Hidden, cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `cat`, `init`. Notable symbols here include `desc`, `cat`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`, `init`。这里较值得关注的符号包括 `desc`, `cat`, `init`。

### Lines 46-57

```cpp
static cl::opt<RetpolineInsertion::AvailabilityOptions> R11Availability(
    "r11-availability",
    cl::desc("determine the availability of r11 before indirect branches"),
    cl::init(RetpolineInsertion::AvailabilityOptions::NEVER),
    cl::values(clEnumValN(RetpolineInsertion::AvailabilityOptions::NEVER,
                          "never", "r11 not available"),
               clEnumValN(RetpolineInsertion::AvailabilityOptions::ALWAYS,
                          "always", "r11 available before calls and jumps"),
               clEnumValN(RetpolineInsertion::AvailabilityOptions::ABI, "abi",
                          "r11 available before calls but not before jumps")),
    cl::ZeroOrMore, cl::cat(BoltCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `cat`. Notable symbols here include `desc`, `init`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `cat`。

### Lines 58-75

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

// Retpoline function structure:
// BB0: call BB2
// BB1: pause
//      lfence
//      jmp BB1
// BB2: mov %reg, (%rsp)
//      ret
// or
// BB2: push %r11
//      mov Address, %r11
//      mov %r11, 8(%rsp)
//      pop %r11
//      ret
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 76-84

```cpp
BinaryFunction *createNewRetpoline(BinaryContext &BC,
                                   const std::string &RetpolineTag,
                                   const IndirectBranchInfo &BrInfo,
                                   bool R11Available) {
  auto &MIB = *BC.MIB;
  MCContext &Ctx = *BC.Ctx;
  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: Creating a new retpoline function["
                    << RetpolineTag << "]\n");
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 85-94

```cpp
  BinaryFunction *NewRetpoline =
      BC.createInjectedBinaryFunction(RetpolineTag, true);
  std::vector<std::unique_ptr<BinaryBasicBlock>> NewBlocks(3);
  for (int I = 0; I < 3; I++) {
    MCSymbol *Symbol =
        Ctx.createNamedTempSymbol(Twine(RetpolineTag + "_BB" + to_string(I)));
    NewBlocks[I] = NewRetpoline->createBasicBlock(Symbol);
    NewBlocks[I].get()->setCFIState(0);
  }
```

- EN: Declares or implements routines including `NewBlocks`, `createBasicBlock`. Notable symbols here include `NewBlocks`, `createBasicBlock`.
- CN: 这里声明或实现函数，例如 `NewBlocks`, `createBasicBlock`。这里较值得关注的符号包括 `NewBlocks`, `createBasicBlock`。

### Lines 95-106

```cpp
  BinaryBasicBlock &BB0 = *NewBlocks[0].get();
  BinaryBasicBlock &BB1 = *NewBlocks[1].get();
  BinaryBasicBlock &BB2 = *NewBlocks[2].get();

  BB0.addSuccessor(&BB2, 0, 0);
  BB1.addSuccessor(&BB1, 0, 0);

  // Build BB0
  MCInst DirectCall;
  MIB.createDirectCall(DirectCall, BB2.getLabel(), &Ctx, /*IsTailCall*/ false);
  BB0.addInstruction(DirectCall);
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 107-117

```cpp
  // Build BB1
  MCInst Pause;
  MIB.createPause(Pause);
  BB1.addInstruction(Pause);

  if (opts::RetpolineLfence) {
    MCInst Lfence;
    MIB.createLfence(Lfence);
    BB1.addInstruction(Lfence);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 118-133

```cpp
  InstructionListType Seq;
  MIB.createShortJmp(Seq, BB1.getLabel(), &Ctx);
  BB1.addInstructions(Seq.begin(), Seq.end());

  // Build BB2
  if (BrInfo.isMem()) {
    if (R11Available) {
      MCInst StoreToStack;
      MIB.createSaveToStack(StoreToStack, MIB.getStackPointer(), 0,
                            MIB.getX86R11(), 8);
      BB2.addInstruction(StoreToStack);
    } else {
      MCInst PushR11;
      MIB.createPushRegister(PushR11, MIB.getX86R11(), 8);
      BB2.addInstruction(PushR11);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 134-141

```cpp
      MCInst LoadCalleeAddrs;
      const IndirectBranchInfo::MemOpInfo &MemRef = BrInfo.Memory;
      MIB.createLoad(LoadCalleeAddrs, MemRef.BaseRegNum, MemRef.ScaleImm,
                     MemRef.IndexRegNum, MemRef.DispImm, MemRef.DispExpr,
                     MemRef.SegRegNum, MIB.getX86R11(), 8);

      BB2.addInstruction(LoadCalleeAddrs);
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 142-159

```cpp
      MCInst StoreToStack;
      MIB.createSaveToStack(StoreToStack, MIB.getStackPointer(), 8,
                            MIB.getX86R11(), 8);
      BB2.addInstruction(StoreToStack);

      MCInst PopR11;
      MIB.createPopRegister(PopR11, MIB.getX86R11(), 8);
      BB2.addInstruction(PopR11);
    }
  } else if (BrInfo.isReg()) {
    MCInst StoreToStack;
    MIB.createSaveToStack(StoreToStack, MIB.getStackPointer(), 0,
                          BrInfo.BranchReg, 8);
    BB2.addInstruction(StoreToStack);
  } else {
    llvm_unreachable("not expected");
  }
```

- EN: Declares or implements routines including `if`, `llvm_unreachable`. Notable symbols here include `if`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `if`, `llvm_unreachable`。这里较值得关注的符号包括 `if`, `llvm_unreachable`。

### Lines 160-167

```cpp
  // return
  MCInst Return;
  MIB.createReturn(Return);
  BB2.addInstruction(Return);
  NewRetpoline->insertBasicBlocks(nullptr, std::move(NewBlocks),
                                  /* UpdateLayout */ true,
                                  /* UpdateCFIState */ false);
```

- EN: Declares or implements routines including `insertBasicBlocks`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `insertBasicBlocks`.
- CN: 这里声明或实现函数，例如 `insertBasicBlocks`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `insertBasicBlocks`。

### Lines 168-178

```cpp
  NewRetpoline->updateState(BinaryFunction::State::CFG_Finalized);
  return NewRetpoline;
}

std::string createRetpolineFunctionTag(BinaryContext &BC,
                                       const IndirectBranchInfo &BrInfo,
                                       bool R11Available) {
  std::string Tag;
  llvm::raw_string_ostream TagOS(Tag);
  TagOS << "__retpoline_";
```

- EN: Declares or implements routines including `updateState`, `TagOS`. Notable symbols here include `updateState`, `TagOS`.
- CN: 这里声明或实现函数，例如 `updateState`, `TagOS`。这里较值得关注的符号包括 `updateState`, `TagOS`。

### Lines 179-188

```cpp
  if (BrInfo.isReg()) {
    BC.InstPrinter->printRegName(TagOS, BrInfo.BranchReg);
    TagOS << "_";
    return Tag;
  }

  // Memory Branch
  if (R11Available)
    return "__retpoline_r11";
```

- EN: Declares or implements routines including `printRegName`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `printRegName`.
- CN: 这里声明或实现函数，例如 `printRegName`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `printRegName`。

### Lines 189-201

```cpp
  const IndirectBranchInfo::MemOpInfo &MemRef = BrInfo.Memory;

  TagOS << "mem_";

  if (MemRef.BaseRegNum != BC.MIB->getNoRegister())
    BC.InstPrinter->printRegName(TagOS, MemRef.BaseRegNum);

  TagOS << "+";
  if (MemRef.DispExpr)
    BC.AsmInfo->printExpr(TagOS, *MemRef.DispExpr);
  else
    TagOS << MemRef.DispImm;
```

- EN: Declares or implements routines including `printRegName`, `printExpr`. Notable symbols here include `printRegName`, `printExpr`.
- CN: 这里声明或实现函数，例如 `printRegName`, `printExpr`。这里较值得关注的符号包括 `printRegName`, `printExpr`。

### Lines 202-211

```cpp
  if (MemRef.IndexRegNum != BC.MIB->getNoRegister()) {
    TagOS << "+" << MemRef.ScaleImm << "*";
    BC.InstPrinter->printRegName(TagOS, MemRef.IndexRegNum);
  }

  if (MemRef.SegRegNum != BC.MIB->getNoRegister()) {
    TagOS << "_seg_";
    BC.InstPrinter->printRegName(TagOS, MemRef.SegRegNum);
  }
```

- EN: Declares or implements routines including `printRegName`. Notable symbols here include `printRegName`.
- CN: 这里声明或实现函数，例如 `printRegName`。这里较值得关注的符号包括 `printRegName`。

### Lines 212-219

```cpp
  return Tag;
}

BinaryFunction *RetpolineInsertion::getOrCreateRetpoline(
    BinaryContext &BC, const IndirectBranchInfo &BrInfo, bool R11Available) {
  const std::string RetpolineTag =
      createRetpolineFunctionTag(BC, BrInfo, R11Available);
```

- EN: Declares or implements routines including `createRetpolineFunctionTag`. Notable symbols here include `createRetpolineFunctionTag`.
- CN: 这里声明或实现函数，例如 `createRetpolineFunctionTag`。这里较值得关注的符号包括 `createRetpolineFunctionTag`。

### Lines 220-237

```cpp
  if (CreatedRetpolines.count(RetpolineTag))
    return CreatedRetpolines[RetpolineTag];

  return CreatedRetpolines[RetpolineTag] =
             createNewRetpoline(BC, RetpolineTag, BrInfo, R11Available);
}

void createBranchReplacement(BinaryContext &BC,
                             const IndirectBranchInfo &BrInfo,
                             bool R11Available,
                             InstructionListType &Replacement,
                             const MCSymbol *RetpolineSymbol) {
  auto &MIB = *BC.MIB;
  // Load the branch address in r11 if available
  if (BrInfo.isMem() && R11Available) {
    const IndirectBranchInfo::MemOpInfo &MemRef = BrInfo.Memory;
    MCInst LoadCalleeAddrs;
    MIB.createLoad(LoadCalleeAddrs, MemRef.BaseRegNum, MemRef.ScaleImm,
```

- EN: Declares or implements routines including `createNewRetpoline`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createNewRetpoline`.
- CN: 这里声明或实现函数，例如 `createNewRetpoline`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createNewRetpoline`。

### Lines 238-247

```cpp
                   MemRef.IndexRegNum, MemRef.DispImm, MemRef.DispExpr,
                   MemRef.SegRegNum, MIB.getX86R11(), 8);
    Replacement.push_back(LoadCalleeAddrs);
  }

  // Call the retpoline
  MCInst RetpolineCall;
  MIB.createDirectCall(RetpolineCall, RetpolineSymbol, BC.Ctx.get(),
                       BrInfo.isJump() || BrInfo.isTailCall());
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 248-265

```cpp
  Replacement.push_back(RetpolineCall);
}

IndirectBranchInfo::IndirectBranchInfo(MCInst &Inst, MCPlusBuilder &MIB) {
  IsCall = MIB.isCall(Inst);
  IsTailCall = MIB.isTailCall(Inst);

  if (MIB.isBranchOnMem(Inst)) {
    IsMem = true;
    std::optional<MCPlusBuilder::X86MemOperand> MO =
        MIB.evaluateX86MemoryOperand(Inst);
    if (!MO)
      llvm_unreachable("not expected");
    Memory = MO.value();
  } else if (MIB.isBranchOnReg(Inst)) {
    assert(MCPlus::getNumPrimeOperands(Inst) == 1 && "expect 1 operand");
    BranchReg = Inst.getOperand(0).getReg();
  } else {
```

- EN: Declares or implements routines including `IndirectBranchInfo`, `llvm_unreachable`, `if`, `assert`. Notable symbols here include `IndirectBranchInfo`, `llvm_unreachable`, `if`, `assert`.
- CN: 这里声明或实现函数，例如 `IndirectBranchInfo`, `llvm_unreachable`, `if`, `assert`。这里较值得关注的符号包括 `IndirectBranchInfo`, `llvm_unreachable`, `if`, `assert`。

### Lines 266-273

```cpp
    llvm_unreachable("unexpected instruction");
  }
}

Error RetpolineInsertion::runOnFunctions(BinaryContext &BC) {
  if (!opts::InsertRetpolines)
    return Error::success();
```

- EN: Declares or implements routines including `llvm_unreachable`, `runOnFunctions`. Notable symbols here include `llvm_unreachable`, `runOnFunctions`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `runOnFunctions`。这里较值得关注的符号包括 `llvm_unreachable`, `runOnFunctions`。

### Lines 274-288

```cpp
  if (!BC.isX86()) {
    BC.errs() << "BOLT-ERROR: " << getName() << " is specific to X86\n";
    exit(1);
  }

  assert(BC.HasRelocations && "retpoline mode not supported in non-reloc");

  auto &MIB = *BC.MIB;
  uint32_t RetpolinedBranches = 0;
  for (auto &It : BC.getBinaryFunctions()) {
    BinaryFunction &Function = It.second;
    for (BinaryBasicBlock &BB : Function) {
      for (auto It = BB.begin(); It != BB.end(); ++It) {
        MCInst &Inst = *It;
```

- EN: Declares or implements routines including `exit`, `assert`. Notable symbols here include `exit`, `assert`.
- CN: 这里声明或实现函数，例如 `exit`, `assert`。这里较值得关注的符号包括 `exit`, `assert`。

### Lines 289-296

```cpp
        if (!MIB.isIndirectCall(Inst) && !MIB.isIndirectBranch(Inst))
          continue;

        IndirectBranchInfo BrInfo(Inst, MIB);
        bool R11Available = false;
        BinaryFunction *TargetRetpoline;
        InstructionListType Replacement;
```

- EN: Declares or implements routines including `BrInfo`. Notable symbols here include `BrInfo`.
- CN: 这里声明或实现函数，例如 `BrInfo`。这里较值得关注的符号包括 `BrInfo`。

### Lines 297-306

```cpp
        // Determine if r11 is available before this instruction
        if (BrInfo.isMem()) {
          if (MIB.hasAnnotation(Inst, "PLTCall"))
            R11Available = true;
          else if (opts::R11Availability == AvailabilityOptions::ALWAYS)
            R11Available = true;
          else if (opts::R11Availability == AvailabilityOptions::ABI)
            R11Available = BrInfo.isCall();
        }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 307-317

```cpp
        // If the instruction addressing pattern uses rsp and the retpoline
        // loads the callee address then displacement needs to be updated
        if (BrInfo.isMem() && !R11Available) {
          IndirectBranchInfo::MemOpInfo &MemRef = BrInfo.Memory;
          int Addend = (BrInfo.isJump() || BrInfo.isTailCall()) ? 8 : 16;
          if (MemRef.BaseRegNum == MIB.getStackPointer())
            MemRef.DispImm += Addend;
          if (MemRef.IndexRegNum == MIB.getStackPointer())
            MemRef.DispImm += Addend * MemRef.ScaleImm;
        }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 318-334

```cpp
        TargetRetpoline = getOrCreateRetpoline(BC, BrInfo, R11Available);

        createBranchReplacement(BC, BrInfo, R11Available, Replacement,
                                TargetRetpoline->getSymbol());

        It = BB.replaceInstruction(It, Replacement.begin(), Replacement.end());
        RetpolinedBranches++;
      }
    }
  }
  BC.outs() << "BOLT-INFO: The number of created retpoline functions is : "
            << CreatedRetpolines.size()
            << "\nBOLT-INFO: The number of retpolined branches is : "
            << RetpolinedBranches << "\n";
  return Error::success();
}
```

- EN: Declares or implements routines including `getOrCreateRetpoline`, `getSymbol`. Notable symbols here include `getOrCreateRetpoline`, `getSymbol`.
- CN: 这里声明或实现函数，例如 `getOrCreateRetpoline`, `getSymbol`。这里较值得关注的符号包括 `getOrCreateRetpoline`, `getSymbol`。

### Lines 335-336

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `LLVM_DEBUG`: function or method entry point / 函数或方法入口
- `NewBlocks`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/RetpolineInsertion.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCInstPrinter.h`, `llvm/Support/raw_ostream.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
