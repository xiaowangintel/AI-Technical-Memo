# X86SuppressAPXForReloc.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86SuppressAPXForReloc.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===- X86SuppressAPXForReloc.cpp - Suppress APX features for relocations -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// This pass is added to suppress APX features for relocations. It's used to
/// keep backward compatibility with old version of linker having no APX
/// support. It can be removed after APX support is included in the default
/// linker on OS.
///
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86RegisterInfo.h"
#include "X86Subtarget.h"

#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

#define DEBUG_TYPE "x86-suppress-apx-for-relocation"

cl::opt<bool> X86EnableAPXForRelocation(
    "x86-enable-apx-for-relocation",
    cl::desc("Enable APX features (EGPR, NDD and NF) for instructions with "
             "relocations on x86-64 ELF"),
    cl::init(false));

namespace {
class X86SuppressAPXForRelocationLegacy : public MachineFunctionPass {
public:
  X86SuppressAPXForRelocationLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "X86 Suppress APX features for relocation";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  static char ID;
};
} // namespace

char X86SuppressAPXForRelocationLegacy::ID = 0;

INITIALIZE_PASS_BEGIN(X86SuppressAPXForRelocationLegacy, DEBUG_TYPE,
                      "X86 Suppress APX features for relocation", false, false)
INITIALIZE_PASS_END(X86SuppressAPXForRelocationLegacy, DEBUG_TYPE,
                    "X86 Suppress APX features for relocation", false, false)
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86SuppressAPXForRelocationLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86SuppressAPXForRelocationLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

FunctionPass *llvm::createX86SuppressAPXForRelocationLegacyPass() {
  return new X86SuppressAPXForRelocationLegacy();
}

static void suppressEGPRRegClass(MachineRegisterInfo *MRI, MachineInstr &MI,
                                 const X86Subtarget &ST, unsigned int OpNum) {
  Register Reg = MI.getOperand(OpNum).getReg();
  if (!Reg.isVirtual()) {
    assert(!X86II::isApxExtendedReg(Reg) && "APX EGPR is used unexpectedly.");
    return;
  }
  const TargetRegisterClass *RC = MRI->getRegClass(Reg);
  const X86RegisterInfo *RI = ST.getRegisterInfo();
  const TargetRegisterClass *NewRC = RI->constrainRegClassToNonRex2(RC);
  MRI->setRegClass(Reg, NewRC);
}

// Suppress EGPR in operand 0 of uses to avoid APX relocation types emitted. The
// register in operand 0 of instruction with relocation may be replaced with
// operand 0 of uses which may be EGPR. That may lead to emit APX relocation
// types which breaks the backward compatibility with builtin linkers on
// existing OS. For example, the register in operand 0 of instruction with
// relocation is used in PHI instruction, and it may be replaced with operand 0
// of PHI instruction after PHI elimination and Machine Copy Propagation pass.
static void suppressEGPRRegClassInRegAndUses(MachineRegisterInfo *MRI,
                                             MachineInstr &MI,
                                             const X86Subtarget &ST,
                                             unsigned int OpNum) {
  suppressEGPRRegClass(MRI, MI, ST, OpNum);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  Register Reg = MI.getOperand(OpNum).getReg();
  for (MachineInstr &Use : MRI->use_instructions(Reg))
    if (Use.getOpcode() == X86::PHI)
      suppressEGPRRegClass(MRI, Use, ST, 0);
}

static bool handleInstructionWithEGPR(MachineFunction &MF,
                                      const X86Subtarget &ST) {
  if (!ST.hasEGPR())
    return false;

  MachineRegisterInfo *MRI = &MF.getRegInfo();
  auto suppressEGPRInInstrWithReloc = [&](MachineInstr &MI,
                                          ArrayRef<unsigned> OpNoArray) {
    int MemOpNo = X86II::getMemoryOperandNo(MI.getDesc().TSFlags) +
                  X86II::getOperandBias(MI.getDesc());
    const MachineOperand &MO = MI.getOperand(X86::AddrDisp + MemOpNo);
    if (MO.getTargetFlags() == X86II::MO_GOTTPOFF ||
        MO.getTargetFlags() == X86II::MO_GOTPCREL) {
      LLVM_DEBUG(dbgs() << "Transform instruction with relocation type:\n  "
                        << MI);
      for (unsigned OpNo : OpNoArray)
        suppressEGPRRegClassInRegAndUses(MRI, MI, ST, OpNo);
      LLVM_DEBUG(dbgs() << "to:\n  " << MI << "\n");
    }
  };

  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      unsigned Opcode = MI.getOpcode();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
      switch (Opcode) {
        // For GOTPC32_TLSDESC, it's emitted with physical register (EAX/RAX) in
        // X86AsmPrinter::LowerTlsAddr, and there is no corresponding target
        // flag for it, so we don't need to handle LEA64r with TLSDESC and EGPR
        // in this pass (before emitting assembly).
      case X86::TEST32mr:
      case X86::TEST64mr: {
        suppressEGPRInInstrWithReloc(MI, {5});
        break;
      }
      case X86::CMP32rm:
      case X86::CMP64rm:
      case X86::MOV32rm:
      case X86::MOV64rm: {
        suppressEGPRInInstrWithReloc(MI, {0});
        break;
      }
      case X86::ADC32rm:
      case X86::ADD32rm:
      case X86::AND32rm:
      case X86::OR32rm:
      case X86::SBB32rm:
      case X86::SUB32rm:
      case X86::XOR32rm:
      case X86::ADC64rm:
      case X86::ADD64rm:
      case X86::AND64rm:
      case X86::OR64rm:
      case X86::SBB64rm:
      case X86::SUB64rm:
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
      case X86::XOR64rm: {
        suppressEGPRInInstrWithReloc(MI, {0, 1});
        break;
      }
      }
    }
  }
  return true;
}

static bool handleNDDOrNFInstructions(MachineFunction &MF,
                                      const X86Subtarget &ST) {
  if (!ST.hasNDD() && !ST.hasNF())
    return false;

  const X86InstrInfo *TII = ST.getInstrInfo();
  MachineRegisterInfo *MRI = &MF.getRegInfo();
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
      unsigned Opcode = MI.getOpcode();
      switch (Opcode) {
      case X86::ADD64rm_NF:
      case X86::ADD64mr_NF_ND:
      case X86::ADD64rm_NF_ND: {
        int MemOpNo = X86II::getMemoryOperandNo(MI.getDesc().TSFlags) +
                      X86II::getOperandBias(MI.getDesc());
        const MachineOperand &MO = MI.getOperand(X86::AddrDisp + MemOpNo);
        if (MO.getTargetFlags() == X86II::MO_GOTTPOFF)
          llvm_unreachable("Unexpected NF instruction!");
        break;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
      }
      case X86::ADD64rm_ND: {
        int MemOpNo = X86II::getMemoryOperandNo(MI.getDesc().TSFlags) +
                      X86II::getOperandBias(MI.getDesc());
        const MachineOperand &MO = MI.getOperand(X86::AddrDisp + MemOpNo);
        if (MO.getTargetFlags() == X86II::MO_GOTTPOFF ||
            MO.getTargetFlags() == X86II::MO_GOTPCREL) {
          LLVM_DEBUG(dbgs() << "Transform instruction with relocation type:\n  "
                            << MI);
          Register Reg = MRI->createVirtualRegister(&X86::GR64_NOREX2RegClass);
          [[maybe_unused]] MachineInstrBuilder CopyMIB =
              BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(TargetOpcode::COPY),
                      Reg)
                  .addReg(MI.getOperand(1).getReg());
          MI.getOperand(1).setReg(Reg);
          const MCInstrDesc &NewDesc = TII->get(X86::ADD64rm);
          MI.setDesc(NewDesc);
          suppressEGPRRegClassInRegAndUses(MRI, MI, ST, 0);
          MI.tieOperands(0, 1);
          LLVM_DEBUG(dbgs() << "to:\n  " << *CopyMIB << "\n");
          LLVM_DEBUG(dbgs() << "  " << MI << "\n");
        }
        break;
      }
      case X86::ADD64mr_ND: {
        int MemRefBegin = X86II::getMemoryOperandNo(MI.getDesc().TSFlags);
        const MachineOperand &MO = MI.getOperand(MemRefBegin + X86::AddrDisp);
        if (MO.getTargetFlags() == X86II::MO_GOTTPOFF) {
          LLVM_DEBUG(dbgs() << "Transform instruction with relocation type:\n  "
                            << MI);
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-240: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
          suppressEGPRRegClassInRegAndUses(MRI, MI, ST, 0);
          Register Reg = MRI->createVirtualRegister(&X86::GR64_NOREX2RegClass);
          [[maybe_unused]] MachineInstrBuilder CopyMIB =
              BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(TargetOpcode::COPY),
                      Reg)
                  .addReg(MI.getOperand(6).getReg());
          MachineInstrBuilder NewMIB =
              BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(X86::ADD64rm),
                      MI.getOperand(0).getReg())
                  .addReg(Reg)
                  .addReg(MI.getOperand(1).getReg())
                  .addImm(MI.getOperand(2).getImm())
                  .addReg(MI.getOperand(3).getReg())
                  .add(MI.getOperand(4))
                  .addReg(MI.getOperand(5).getReg());
          MachineOperand *FlagDef =
              MI.findRegisterDefOperand(X86::EFLAGS, /*TRI=*/nullptr);
          if (FlagDef && FlagDef->isDead()) {
            MachineOperand *NewFlagDef =
                NewMIB->findRegisterDefOperand(X86::EFLAGS, /*TRI=*/nullptr);
            if (NewFlagDef)
              NewFlagDef->setIsDead();
          }
          MI.eraseFromParent();
          LLVM_DEBUG(dbgs() << "to:\n  " << *CopyMIB << "\n");
          LLVM_DEBUG(dbgs() << "  " << *NewMIB << "\n");
        }
        break;
      }
      }
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 241-268: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    }
  }
  return true;
}

static bool suppressAPXForRelocation(MachineFunction &MF) {
  if (X86EnableAPXForRelocation)
    return false;
  const X86Subtarget &ST = MF.getSubtarget<X86Subtarget>();
  bool Changed = handleInstructionWithEGPR(MF, ST);
  Changed |= handleNDDOrNFInstructions(MF, ST);

  return Changed;
}

bool X86SuppressAPXForRelocationLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  return suppressAPXForRelocation(MF);
}

PreservedAnalyses
X86SuppressAPXForRelocationPass::run(MachineFunction &MF,
                                     MachineFunctionAnalysisManager &MFAM) {
  return suppressAPXForRelocation(MF)
             ? getMachineFunctionPassPreservedAnalyses()
                   .preserveSet<CFGAnalyses>()
             : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86SuppressAPXForRelocationLegacy. / 重要符号：X86SuppressAPXForRelocationLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86RegisterInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetRegisterInfo.h. / 直接包含：X86.h, X86InstrInfo.h, X86RegisterInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetRegisterInfo.h。
- Additional include dependencies: 1 more headers. / 额外包含依赖：还有 1 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
