# X86FixupSetCC.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86FixupSetCC.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements fixup helpers for the core X86 backend. / 实现X86 后端核心中的修正辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===- X86FixupSetCC.cpp - fix zero-extension of setcc patterns -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines a pass that fixes zero-extension of setcc patterns.
// X86 setcc instructions are modeled to have no input arguments, and a single
// GR8 output argument. This is consistent with other similar instructions
// (e.g. movb), but means it is impossible to directly generate a setcc into
// the lower GR8 of a specified GR32.
// This means that ISel must select (zext (setcc)) into something like
// seta %al; movzbl %al, %eax.
// Unfortunately, this can cause a stall due to the partial register write
// performed by the setcc. Instead, we can use:
// xor %eax, %eax; seta %al
// This both avoids the stall, and encodes shorter.
//
// Furthurmore, we can use:
// setzua %al
// if feature zero-upper is available. It's faster than the xor+setcc sequence.
// When r16-r31 is used, it even encodes shorter.
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/ADT/Statistic.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"

using namespace llvm;

#define DEBUG_TYPE "x86-fixup-setcc"

STATISTIC(NumSubstZexts, "Number of setcc + zext pairs substituted");

namespace {
class X86FixupSetCCLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86FixupSetCCLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "X86 Fixup SetCC"; }

  bool runOnMachineFunction(MachineFunction &MF) override;
};
} // end anonymous namespace

char X86FixupSetCCLegacy::ID = 0;

INITIALIZE_PASS(X86FixupSetCCLegacy, DEBUG_TYPE, DEBUG_TYPE, false, false)

FunctionPass *llvm::createX86FixupSetCCLegacyPass() {
  return new X86FixupSetCCLegacy();
}
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86FixupSetCCLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86FixupSetCCLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

static bool fixupSetCC(MachineFunction &MF) {
  bool Changed = false;
  MachineRegisterInfo *MRI = &MF.getRegInfo();
  const X86Subtarget *ST = &MF.getSubtarget<X86Subtarget>();
  const X86InstrInfo *TII = ST->getInstrInfo();

  SmallVector<MachineInstr*, 4> ToErase;

  for (auto &MBB : MF) {
    MachineInstr *FlagsDefMI = nullptr;
    for (auto &MI : MBB) {
      // Remember the most recent preceding eflags defining instruction.
      if (MI.definesRegister(X86::EFLAGS, /*TRI=*/nullptr))
        FlagsDefMI = &MI;

      // Find a setcc/setzucc (if ZU is enabled) that is used by a zext.
      // This doesn't have to be the only use, the transformation is safe
      // regardless.
      if (MI.getOpcode() != X86::SETCCr && MI.getOpcode() != X86::SETZUCCr)
        continue;

      MachineInstr *ZExt = nullptr;
      Register Reg0 = MI.getOperand(0).getReg();
      for (auto &Use : MRI->use_instructions(Reg0))
        if (Use.getOpcode() == X86::MOVZX32rr8)
          ZExt = &Use;

      if (!ZExt)
        continue;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include fixupSetCC. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 fixupSetCC。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

      if (!FlagsDefMI)
        continue;

      // We'd like to put something that clobbers eflags directly before
      // FlagsDefMI. This can't hurt anything after FlagsDefMI, because
      // it, itself, by definition, clobbers eflags. But it may happen that
      // FlagsDefMI also *uses* eflags, in which case the transformation is
      // invalid.
      if (!ST->hasZU() &&
          FlagsDefMI->readsRegister(X86::EFLAGS, /*TRI=*/nullptr))
        continue;

      // On 32-bit, we need to be careful to force an ABCD register.
      const TargetRegisterClass *RC =
          ST->is64Bit() ? &X86::GR32RegClass : &X86::GR32_ABCDRegClass;
      if (!MRI->constrainRegClass(ZExt->getOperand(0).getReg(), RC)) {
        // If we cannot constrain the register, we would need an additional copy
        // and are better off keeping the MOVZX32rr8 we have now.
        continue;
      }

      ++NumSubstZexts;
      Changed = true;

      // X86 setcc/setzucc only takes an output GR8, so fake a GR32 input by
      // inserting the setcc/setzucc result into the low byte of the zeroed
      // register.
      Register ZeroReg = MRI->createVirtualRegister(RC);
      if (ST->hasZU()) {
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
        if (!ST->preferLegacySetCC())
          assert((MI.getOpcode() == X86::SETZUCCr) &&
                 "Expect setzucc instruction!");
        else
          MI.setDesc(TII->get(X86::SETZUCCr));
        BuildMI(*ZExt->getParent(), ZExt, ZExt->getDebugLoc(),
                TII->get(TargetOpcode::IMPLICIT_DEF), ZeroReg);
      } else {
        // Initialize a register with 0. This must go before the eflags def
        BuildMI(MBB, FlagsDefMI, MI.getDebugLoc(), TII->get(X86::MOV32r0),
                ZeroReg);
      }

      BuildMI(*ZExt->getParent(), ZExt, ZExt->getDebugLoc(),
              TII->get(X86::INSERT_SUBREG), ZExt->getOperand(0).getReg())
          .addReg(ZeroReg)
          .addReg(Reg0)
          .addImm(X86::sub_8bit);

      // Redirect the debug-instr-number to the setcc.
      if (unsigned InstrNum = ZExt->peekDebugInstrNum())
        MF.makeDebugValueSubstitution({InstrNum, 0},
                                      {MI.getDebugInstrNum(), 0});

      ToErase.push_back(ZExt);
    }
  }

  for (auto &I : ToErase)
    I->eraseFromParent();
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-164: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  return Changed;
}

bool X86FixupSetCCLegacy::runOnMachineFunction(MachineFunction &MF) {
  return fixupSetCC(MF);
}

PreservedAnalyses X86FixupSetCCPass::run(MachineFunction &MF,
                                         MachineFunctionAnalysisManager &MFAM) {
  return fixupSetCC(MF) ? getMachineFunctionPassPreservedAnalyses()
                              .preserveSet<CFGAnalyses>()
                        : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: fixup helpers. / 核心主题：修正辅助逻辑。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86FixupSetCCLegacy. / 重要符号：X86FixupSetCCLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
