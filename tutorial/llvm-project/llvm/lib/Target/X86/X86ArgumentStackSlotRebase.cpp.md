# X86ArgumentStackSlotRebase.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86ArgumentStackSlotRebase.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===---- X86ArgumentStackSlotRebase.cpp - rebase argument stack slot -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass replace the frame register with a GPR virtual register and set
// the stack offset for each instruction which reference argument from stack.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86MachineFunctionInfo.h"
#include "X86RegisterInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/Function.h"
#include "llvm/Pass.h"

```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
using namespace llvm;

#define DEBUG_TYPE "x86-argument-stack-slot"

namespace {

class X86ArgumentStackSlotLegacy : public MachineFunctionPass {

public:
  static char ID; // Pass identification, replacement for typeid

  explicit X86ArgumentStackSlotLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // end anonymous namespace

char X86ArgumentStackSlotLegacy::ID = 0;

INITIALIZE_PASS(X86ArgumentStackSlotLegacy, DEBUG_TYPE, "Argument Stack Rebase",
                false, false)

FunctionPass *llvm::createX86ArgumentStackSlotLegacyPass() {
  return new X86ArgumentStackSlotLegacy();
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86ArgumentStackSlotLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86ArgumentStackSlotLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
}

static Register getArgBaseReg(MachineFunction &MF) {
  MachineRegisterInfo &MRI = MF.getRegInfo();
  const X86Subtarget &STI = MF.getSubtarget<X86Subtarget>();
  const Function &F = MF.getFunction();
  CallingConv::ID CC = F.getCallingConv();
  Register NoReg;
  const TargetRegisterClass *RC = nullptr;
  switch (CC) {
  // We need a virtual register in case there is inline assembly
  // clobber argument base register.
  case CallingConv::C:
    RC = STI.is64Bit() ? &X86::GR64_ArgRefRegClass : &X86::GR32_ArgRefRegClass;
    break;
  case CallingConv::X86_RegCall:
    // FIXME: For regcall there is no scratch register on 32-bit target.
    // We may use a callee saved register as argument base register and
    // save it before being changed as base pointer. We need DW_CFA to
    // indicate where the callee saved register is saved, so that it can
    // be correctly unwind.
    // push      ebx
    // mov       ebx, esp
    // and       esp, -128
    // ...
    // pop       ebx
    // ret
    RC = STI.is64Bit() ? &X86::GR64_ArgRefRegClass : nullptr;
    break;
  // TODO: Refine register class for each calling convention.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  default:
    break;
  }
  if (RC)
    return MRI.createVirtualRegister(RC);
  else
    return NoReg;
}

static bool argumentStackSlotRebase(MachineFunction &MF) {
  const Function &F = MF.getFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const X86Subtarget &STI = MF.getSubtarget<X86Subtarget>();
  const X86RegisterInfo *TRI = STI.getRegisterInfo();
  const X86InstrInfo *TII = STI.getInstrInfo();
  X86MachineFunctionInfo *X86FI = MF.getInfo<X86MachineFunctionInfo>();
  bool Changed = false;

  if (F.hasFnAttribute(Attribute::Naked))
    return false;
  // Only support Linux and ELF.
  if (!STI.isTargetLinux() && !STI.isTargetELF())
    return false;
  if (!TRI->hasBasePointer(MF))
    return false;
  // Don't support X32
  if (STI.isTarget64BitILP32())
    return false;

  Register BasePtr = TRI->getBaseRegister();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  auto IsBaseRegisterClobbered = [&]() {
    for (MachineBasicBlock &MBB : MF) {
      for (MachineInstr &MI : MBB) {
        if (!MI.isInlineAsm())
          continue;
        for (MachineOperand &MO : MI.operands()) {
          if (!MO.isReg())
            continue;
          Register Reg = MO.getReg();
          if (!Reg.isPhysical())
            continue;
          if (TRI->isSuperOrSubRegisterEq(BasePtr, Reg))
            return true;
        }
      }
    }
    return false;
  };
  if (!IsBaseRegisterClobbered())
    return false;

  Register ArgBaseReg = getArgBaseReg(MF);
  if (!ArgBaseReg.isValid())
    return false;
  // leal    4(%esp), %reg
  MachineBasicBlock &MBB = MF.front();
  MachineBasicBlock::iterator MBBI = MBB.begin();
  DebugLoc DL;
  // Emit instruction to copy get stack pointer to a virtual register
  // and save the instruction to x86 machine functon info. We can get
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  // physical register of ArgBaseReg after register allocation. The
  // stack slot is used to save/restore argument base pointer. We can
  // get the index from the instruction.
  unsigned SlotSize = TRI->getSlotSize();
  int FI = MFI.CreateSpillStackObject(SlotSize, Align(SlotSize));
  // Use pseudo LEA to prevent the instruction from being eliminated.
  // TODO: if it is duplicated we can expand it to lea.
  MachineInstr *LEA =
      BuildMI(MBB, MBBI, DL,
              TII->get(STI.is64Bit() ? X86::PLEA64r : X86::PLEA32r), ArgBaseReg)
          .addFrameIndex(FI)
          .addImm(1)
          .addUse(X86::NoRegister)
          .addImm(SlotSize)
          .addUse(X86::NoRegister)
          .setMIFlag(MachineInstr::FrameSetup);
  X86FI->setStackPtrSaveMI(LEA);

  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      int I = 0;
      for (MachineOperand &MO : MI.operands()) {
        if (MO.isFI()) {
          int Idx = MO.getIndex();
          if (!MFI.isFixedObjectIndex(Idx))
            continue;
          int64_t Offset = MFI.getObjectOffset(Idx);
          if (Offset < 0)
            continue;
          // TODO replace register for debug instruction
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-205: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
          if (MI.isDebugInstr())
            continue;
          // Replace frame register with argument base pointer and its offset.
          TRI->eliminateFrameIndex(MI.getIterator(), I, ArgBaseReg, Offset);
          Changed = true;
        }
        ++I;
      }
    }
  }

  return Changed;
}

bool X86ArgumentStackSlotLegacy::runOnMachineFunction(MachineFunction &MF) {
  return argumentStackSlotRebase(MF);
}

PreservedAnalyses
X86ArgumentStackSlotPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &MFAM) {
  return argumentStackSlotRebase(MF) ? getMachineFunctionPassPreservedAnalyses()
                                           .preserveSet<CFGAnalyses>()
                                     : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86ArgumentStackSlotLegacy. / 重要符号：X86ArgumentStackSlotLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86MachineFunctionInfo.h, X86RegisterInfo.h, X86Subtarget.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h. / 直接包含：X86.h, X86MachineFunctionInfo.h, X86RegisterInfo.h, X86Subtarget.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h。
- Additional include dependencies: 6 more headers. / 额外包含依赖：还有 6 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
