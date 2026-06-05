# X86GlobalBaseReg.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86GlobalBaseReg.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===- X86GlobalBaseReg.cpp - PIC Global Base Register Initialization -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the pass that initializes the PIC global base register
// for x86-32.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86MachineFunctionInfo.h"
#include "X86Subtarget.h"
#include "X86TargetMachine.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "x86-global-base-reg"

namespace {
class X86GlobalBaseRegLegacy : public MachineFunctionPass {
public:
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. Key symbols include X86GlobalBaseRegLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 X86GlobalBaseRegLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  static char ID;
  X86GlobalBaseRegLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "X86 PIC Global Base Reg Initialization";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
} // end anonymous namespace

char X86GlobalBaseRegLegacy::ID = 0;

FunctionPass *llvm::createX86GlobalBaseRegLegacyPass() {
  return new X86GlobalBaseRegLegacy();
}

static bool initGlobalBaseReg(MachineFunction &MF) {
  const X86TargetMachine *TM =
      static_cast<const X86TargetMachine *>(&MF.getTarget());
  const X86Subtarget &STI = MF.getSubtarget<X86Subtarget>();

  // Only emit a global base reg in PIC mode.
  if (!TM->isPositionIndependent())
    return false;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

  X86MachineFunctionInfo *X86FI = MF.getInfo<X86MachineFunctionInfo>();
  Register GlobalBaseReg = X86FI->getGlobalBaseReg();

  // If we didn't need a GlobalBaseReg, don't insert code.
  if (GlobalBaseReg == 0)
    return false;

  // Insert the set of GlobalBaseReg into the first MBB of the function
  MachineBasicBlock &FirstMBB = MF.front();
  MachineBasicBlock::iterator MBBI = FirstMBB.begin();
  DebugLoc DL = FirstMBB.findDebugLoc(MBBI);
  MachineRegisterInfo &RegInfo = MF.getRegInfo();
  const X86InstrInfo *TII = STI.getInstrInfo();

  Register PC;
  if (STI.isPICStyleGOT())
    PC = RegInfo.createVirtualRegister(&X86::GR32RegClass);
  else
    PC = GlobalBaseReg;

  if (STI.is64Bit()) {
    if (TM->getCodeModel() == CodeModel::Large) {
      // In the large code model, we are aiming for this code, though the
      // register allocation may vary:
      //   leaq .LN$pb(%rip), %rax
      //   movq $_GLOBAL_OFFSET_TABLE_ - .LN$pb, %rcx
      //   addq %rcx, %rax
      // RAX now holds address of _GLOBAL_OFFSET_TABLE_.
      Register PBReg = RegInfo.createVirtualRegister(&X86::GR64RegClass);
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
      Register GOTReg = RegInfo.createVirtualRegister(&X86::GR64RegClass);
      BuildMI(FirstMBB, MBBI, DL, TII->get(X86::LEA64r), PBReg)
          .addReg(X86::RIP)
          .addImm(0)
          .addReg(0)
          .addSym(MF.getPICBaseSymbol())
          .addReg(0);
      std::prev(MBBI)->setPreInstrSymbol(MF, MF.getPICBaseSymbol());
      BuildMI(FirstMBB, MBBI, DL, TII->get(X86::MOV64ri), GOTReg)
          .addExternalSymbol("_GLOBAL_OFFSET_TABLE_",
                             X86II::MO_PIC_BASE_OFFSET);
      BuildMI(FirstMBB, MBBI, DL, TII->get(X86::ADD64rr), PC)
          .addReg(PBReg, RegState::Kill)
          .addReg(GOTReg, RegState::Kill);
    } else {
      // In other code models, use a RIP-relative LEA to materialize the
      // GOT.
      BuildMI(FirstMBB, MBBI, DL, TII->get(X86::LEA64r), PC)
          .addReg(X86::RIP)
          .addImm(0)
          .addReg(0)
          .addExternalSymbol("_GLOBAL_OFFSET_TABLE_")
          .addReg(0);
    }
  } else {
    // Operand of MovePCtoStack is completely ignored by asm printer. It's
    // only used in JIT code emission as displacement to pc.
    BuildMI(FirstMBB, MBBI, DL, TII->get(X86::MOVPC32r), PC).addImm(0);

    // If we're using vanilla 'GOT' PIC style, we should use relative
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-145: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    // addressing not to pc, but to _GLOBAL_OFFSET_TABLE_ external.
    if (STI.isPICStyleGOT()) {
      // Generate addl $__GLOBAL_OFFSET_TABLE_ + [.-piclabel],
      // %some_register
      BuildMI(FirstMBB, MBBI, DL, TII->get(X86::ADD32ri), GlobalBaseReg)
          .addReg(PC)
          .addExternalSymbol("_GLOBAL_OFFSET_TABLE_",
                             X86II::MO_GOT_ABSOLUTE_ADDRESS);
    }
  }

  return true;
}

bool X86GlobalBaseRegLegacy::runOnMachineFunction(MachineFunction &MF) {
  return initGlobalBaseReg(MF);
}

PreservedAnalyses
X86GlobalBaseRegPass::run(MachineFunction &MF,
                          MachineFunctionAnalysisManager &MFAM) {
  return initGlobalBaseReg(MF) ? getMachineFunctionPassPreservedAnalyses()
                                     .preserveSet<CFGAnalyses>()
                               : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86GlobalBaseRegLegacy. / 重要符号：X86GlobalBaseRegLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86MachineFunctionInfo.h, X86Subtarget.h, X86TargetMachine.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/InitializePasses.h. / 直接包含：X86.h, X86InstrInfo.h, X86MachineFunctionInfo.h, X86Subtarget.h, X86TargetMachine.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/InitializePasses.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
