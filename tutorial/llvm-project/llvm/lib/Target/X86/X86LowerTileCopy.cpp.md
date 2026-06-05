# X86LowerTileCopy.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86LowerTileCopy.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86LowerTileCopy.cpp - Expand Tile Copy Instructions---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the pass which lower AMX tile copy instructions. Since
// there is no tile copy instruction, we need store tile register to stack
// and load from stack to another tile register. We need extra GR to hold
// the stride, and we need stack slot to hold the tile data register.
// We would run this pass after copy propagation, so that we don't miss copy
// optimization. And we would run this pass before prolog/epilog insertion,
// so that we can allocate stack slot.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrBuilder.h"
#include "X86InstrInfo.h"
#include "X86MachineFunctionInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/LiveRegUnits.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/DebugLoc.h"

using namespace llvm;

#define DEBUG_TYPE "x86-lower-tile-copy"

namespace {

class X86LowerTileCopyLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86LowerTileCopyLegacy() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override;

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override { return "X86 Lower Tile Copy"; }
};

} // namespace

char X86LowerTileCopyLegacy::ID = 0;

INITIALIZE_PASS_BEGIN(X86LowerTileCopyLegacy, DEBUG_TYPE, "Tile Copy Lowering",
                      false, false)
INITIALIZE_PASS_END(X86LowerTileCopyLegacy, DEBUG_TYPE, "Tile Copy Lowering",
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86LowerTileCopyLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86LowerTileCopyLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
                    false, false)

void X86LowerTileCopyLegacy::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.setPreservesAll();
  MachineFunctionPass::getAnalysisUsage(AU);
}

FunctionPass *llvm::createX86LowerTileCopyLegacyPass() {
  return new X86LowerTileCopyLegacy();
}

static bool lowerTileCopy(MachineFunction &MF) {
  X86MachineFunctionInfo *FuncInfo = MF.getInfo<X86MachineFunctionInfo>();
  if (FuncInfo->getAMXProgModel() != AMXProgModelEnum::ManagedRA)
    return false;

  const X86Subtarget &ST = MF.getSubtarget<X86Subtarget>();
  assert(ST.hasAMXTILE() && "Only supported on AMXTILE targets");

  const X86InstrInfo *TII = ST.getInstrInfo();
  const TargetRegisterInfo *TRI = ST.getRegisterInfo();
  BitVector GR64Regs =
      TRI->getAllocatableSet(MF, TRI->getRegClass(X86::GR64RegClassID));
  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    LiveRegUnits UsedRegs(*TRI);
    UsedRegs.addLiveOuts(MBB);
    for (MachineInstr &MI : llvm::make_early_inc_range(reverse(MBB))) {
      UsedRegs.stepBackward(MI);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
      if (!MI.isCopy())
        continue;
      MachineOperand &DstMO = MI.getOperand(0);
      MachineOperand &SrcMO = MI.getOperand(1);
      Register SrcReg = SrcMO.getReg();
      Register DstReg = DstMO.getReg();
      if (!X86::TILERegClass.contains(DstReg, SrcReg))
        continue;

      // Allocate stack slot for tile register
      unsigned Size = TRI->getSpillSize(X86::TILERegClass);
      Align Alignment = TRI->getSpillAlign(X86::TILERegClass);
      int TileSS = MF.getFrameInfo().CreateSpillStackObject(Size, Alignment);

      int StrideSS = 0;

      // Pick a killed register to avoid a save/reload.
      Register GR64Cand = X86::NoRegister;
      for (auto RegT : GR64Regs.set_bits()) {
        if (UsedRegs.available(RegT)) {
          GR64Cand = RegT;
          break;
        }
      }

      const DebugLoc &DL = MI.getDebugLoc();
      if (GR64Cand) {
        // mov 64 %reg
        BuildMI(MBB, MI, DL, TII->get(X86::MOV64ri), GR64Cand).addImm(64);
      } else {
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
        // No available register? Save RAX and reload it after use.

        // Allocate stack slot for stride register
        Size = TRI->getSpillSize(X86::GR64RegClass);
        Alignment = TRI->getSpillAlign(X86::GR64RegClass);
        StrideSS = MF.getFrameInfo().CreateSpillStackObject(Size, Alignment);

        // mov %reg (%sp)
        addFrameReference(BuildMI(MBB, MI, DL, TII->get(X86::MOV64mr)),
                          StrideSS)
            .addReg(X86::RAX);
        // mov 64 %reg
        BuildMI(MBB, MI, DL, TII->get(X86::MOV64ri), X86::RAX).addImm(64);
      }
      // tilestored %tmm, (%sp, %idx)
#define GET_EGPR_IF_ENABLED(OPC) (ST.hasEGPR() ? OPC##_EVEX : OPC)
      unsigned Opc = GET_EGPR_IF_ENABLED(X86::TILESTORED);
      MachineInstr *NewMI =
          addFrameReference(BuildMI(MBB, MI, DL, TII->get(Opc)), TileSS)
              .addReg(SrcReg, getKillRegState(SrcMO.isKill()));
      MachineOperand *MO = &NewMI->getOperand(X86::AddrIndexReg);
      MO->setReg(GR64Cand ? GR64Cand : X86::RAX);
      // tileloadd (%sp, %idx), %tmm
      Opc = GET_EGPR_IF_ENABLED(X86::TILELOADD);
#undef GET_EGPR_IF_ENABLED
      NewMI = addFrameReference(BuildMI(MBB, MI, DL, TII->get(Opc), DstReg),
                                TileSS);
      MO = &NewMI->getOperand(1 + X86::AddrIndexReg);
      MO->setReg(GR64Cand ? GR64Cand : X86::RAX);
      MO->setIsKill(true);
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-174: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
      if (!GR64Cand) {
        // restore %rax
        // mov (%sp) %rax
        addFrameReference(
            BuildMI(MBB, MI, DL, TII->get(X86::MOV64rm), X86::RAX), StrideSS);
      }
      MI.eraseFromParent();
      Changed = true;
    }
  }
  return Changed;
}

bool X86LowerTileCopyLegacy::runOnMachineFunction(MachineFunction &MF) {
  return lowerTileCopy(MF);
}

PreservedAnalyses
X86LowerTileCopyPass::run(MachineFunction &MF,
                          MachineFunctionAnalysisManager &MFAM) {
  return lowerTileCopy(MF) ? getMachineFunctionPassPreservedAnalyses()
                                 .preserveSet<CFGAnalyses>()
                           : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86LowerTileCopyLegacy. / 重要符号：X86LowerTileCopyLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrBuilder.h, X86InstrInfo.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/LiveRegUnits.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h. / 直接包含：X86.h, X86InstrBuilder.h, X86InstrInfo.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/LiveRegUnits.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h。
- Additional include dependencies: 5 more headers. / 额外包含依赖：还有 5 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
