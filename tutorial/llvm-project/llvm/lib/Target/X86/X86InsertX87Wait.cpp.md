# X86InsertX87Wait.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InsertX87Wait.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
// X86InsertX87Wait.cpp - Strict-Fp:Insert wait instruction X87 instructions //
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the pass which insert x86 wait instructions after each
// X87 instructions when strict float is enabled.
//
// The logic to insert a wait instruction after an X87 instruction is as below:
// 1. If the X87 instruction don't raise float exception nor is a load/store
//    instruction, or is a x87 control instruction, don't insert wait.
// 2. If the X87 instruction is an instruction which the following instruction
//    is an X87 exception synchronizing X87 instruction, don't insert wait.
// 3. For other situations, insert wait instruction.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/DebugLoc.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/Support/Debug.h"

using namespace llvm;

#define DEBUG_TYPE "x86-insert-x87-wait"

namespace {

class X86InsertX87WaitLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86InsertX87WaitLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "X86 insert wait instruction";
  }
};
} // end anonymous namespace

char X86InsertX87WaitLegacy::ID = 0;

FunctionPass *llvm::createX86InsertX87WaitLegacyPass() {
  return new X86InsertX87WaitLegacy();
}

static bool isX87ControlInstruction(MachineInstr &MI) {
  switch (MI.getOpcode()) {
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86InsertX87WaitLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86InsertX87WaitLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  case X86::FNINIT:
  case X86::FLDCW16m:
  case X86::FNSTCW16m:
  case X86::FNSTSW16r:
  case X86::FNSTSWm:
  case X86::FNCLEX:
  case X86::FLDENVm:
  case X86::FSTENVm:
  case X86::FRSTORm:
  case X86::FSAVEm:
  case X86::FINCSTP:
  case X86::FDECSTP:
  case X86::FFREE:
  case X86::FFREEP:
  case X86::FNOP:
  case X86::WAIT:
    return true;
  default:
    return false;
  }
}

static bool isX87NonWaitingControlInstruction(MachineInstr &MI) {
  // a few special control instructions don't perform a wait operation
  switch (MI.getOpcode()) {
  case X86::FNINIT:
  case X86::FNSTSW16r:
  case X86::FNSTSWm:
  case X86::FNSTCW16m:
  case X86::FNCLEX:
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    return true;
  default:
    return false;
  }
}

static bool insertWaitInstruction(MachineFunction &MF) {
  if (!MF.getFunction().hasFnAttribute(Attribute::StrictFP))
    return false;

  const X86Subtarget &ST = MF.getSubtarget<X86Subtarget>();
  const X86InstrInfo *TII = ST.getInstrInfo();
  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    for (MachineBasicBlock::iterator MI = MBB.begin(); MI != MBB.end(); ++MI) {
      // Jump non X87 instruction.
      if (!X86::isX87Instruction(*MI))
        continue;
      // If the instruction instruction neither has float exception nor is
      // a load/store instruction, or the instruction is x87 control
      // instruction, do not insert wait.
      if (!(MI->mayRaiseFPException() || MI->mayLoadOrStore()) ||
          isX87ControlInstruction(*MI))
        continue;
      // If the following instruction is an X87 instruction and isn't an X87
      // non-waiting control instruction, we can omit insert wait instruction.
      MachineBasicBlock::iterator AfterMI = std::next(MI);
      if (AfterMI != MBB.end() && X86::isX87Instruction(*AfterMI) &&
          !isX87NonWaitingControlInstruction(*AfterMI))
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-142: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
        continue;

      BuildMI(MBB, AfterMI, MI->getDebugLoc(), TII->get(X86::WAIT));
      LLVM_DEBUG(dbgs() << "\nInsert wait after:\t" << *MI);
      // Jump the newly inserting wait
      ++MI;
      Changed = true;
    }
  }
  return Changed;
}

bool X86InsertX87WaitLegacy::runOnMachineFunction(MachineFunction &MF) {
  return insertWaitInstruction(MF);
}

PreservedAnalyses X86InsertX87WaitPass::run(MachineFunction &MF,
                                            MachineFunctionAnalysisManager &) {
  return insertWaitInstruction(MF) ? getMachineFunctionPassPreservedAnalyses()
                                         .preserveSet<CFGAnalyses>()
                                   : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86InsertX87WaitLegacy. / 重要符号：X86InsertX87WaitLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/IR/DebugLoc.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/IR/DebugLoc.h。
- Additional include dependencies: 1 more headers. / 额外包含依赖：还有 1 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
