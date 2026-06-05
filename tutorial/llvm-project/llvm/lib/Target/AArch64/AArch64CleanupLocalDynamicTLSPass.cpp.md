# AArch64CleanupLocalDynamicTLSPass.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64CleanupLocalDynamicTLSPass.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements AArch64 backend logic for the AArch64 backend. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Documented code section
```cpp
//===-- AArch64CleanupLocalDynamicTLSPass.cpp ---------------------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Local-dynamic access to thread-local variables proceeds in three stages.
//
// 1. The offset of this Module's thread-local area from TPIDR_EL0 is calculated
//    in much the same way as a general-dynamic TLS-descriptor access against
//    the special symbol _TLS_MODULE_BASE.
// 2. The variable's offset from _TLS_MODULE_BASE_ is calculated using
//    instructions with "dtprel" modifiers.
// 3. These two are added, together with TPIDR_EL0, to obtain the variable's
//    true address.
//
// This is only better than general-dynamic access to the variable if two or
// more of the first stage TLS-descriptor calculations can be combined. This
// pass looks through a function and performs such combinations.
//
//===----------------------------------------------------------------------===//
#include "AArch64.h"
#include "AArch64MachineFunctionInfo.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 29-53: Includes and compile-time dependencies
```cpp
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
using namespace llvm;

#define TLSCLEANUP_PASS_NAME "AArch64 Local Dynamic TLS Access Clean-up"

namespace {
struct LDTLSCleanup : public MachineFunctionPass {
  static char ID;
  LDTLSCleanup() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override {
    if (skipFunction(MF.getFunction()))
      return false;

    AArch64FunctionInfo *AFI = MF.getInfo<AArch64FunctionInfo>();
    if (AFI->getNumLocalDynamicTLSAccesses() < 2) {
      // No point folding accesses if there isn't at least two.
      return false;
    }

    MachineDominatorTree *DT =
        &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
    return VisitNode(DT->getRootNode(), 0);
  }
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 54-72: Function VisitNode
```cpp

  // Visit the dominator subtree rooted at Node in pre-order.
  // If TLSBaseAddrReg is non-null, then use that to replace any
  // TLS_base_addr instructions. Otherwise, create the register
  // when the first such instruction is seen, and then use it
  // as we encounter more instructions.
  bool VisitNode(MachineDomTreeNode *Node, unsigned TLSBaseAddrReg) {
    MachineBasicBlock *BB = Node->getBlock();
    bool Changed = false;

    // Traverse the current block.
    for (MachineBasicBlock::iterator I = BB->begin(), E = BB->end(); I != E;
         ++I) {
      switch (I->getOpcode()) {
      case AArch64::TLSDESC_CALLSEQ:
        // Make sure it's a local dynamic access.
        if (!I->getOperand(0).isSymbol() ||
            strcmp(I->getOperand(0).getSymbolName(), "_TLS_MODULE_BASE_"))
          break;
```
**EN:** This block implements VisitNode, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 VisitNode，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 73-98: Core AArch64 backend logic
```cpp

        if (TLSBaseAddrReg)
          I = replaceTLSBaseAddrCall(*I, TLSBaseAddrReg);
        else
          I = setRegister(*I, &TLSBaseAddrReg);
        Changed = true;
        break;
      default:
        break;
      }
    }

    // Visit the children of this block in the dominator tree.
    for (MachineDomTreeNode *N : *Node) {
      Changed |= VisitNode(N, TLSBaseAddrReg);
    }

    return Changed;
  }

  // Replace the TLS_base_addr instruction I with a copy from
  // TLSBaseAddrReg, returning the new instruction.
  MachineInstr *replaceTLSBaseAddrCall(MachineInstr &I,
                                       unsigned TLSBaseAddrReg) {
    MachineFunction *MF = I.getParent()->getParent();
    const TargetInstrInfo *TII = MF->getSubtarget().getInstrInfo();
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 99-124: Documented code section
```cpp

    // Insert a Copy from TLSBaseAddrReg to x0, which is where the rest of the
    // code sequence assumes the address will be.
    MachineInstr *Copy = BuildMI(*I.getParent(), I, I.getDebugLoc(),
                                 TII->get(TargetOpcode::COPY), AArch64::X0)
                             .addReg(TLSBaseAddrReg);

    // Update the call info.
    if (I.shouldUpdateAdditionalCallInfo())
      I.getMF()->eraseAdditionalCallInfo(&I);

    // Erase the TLS_base_addr instruction.
    I.eraseFromParent();

    return Copy;
  }

  // Create a virtual register in *TLSBaseAddrReg, and populate it by
  // inserting a copy instruction after I. Returns the new instruction.
  MachineInstr *setRegister(MachineInstr &I, unsigned *TLSBaseAddrReg) {
    MachineFunction *MF = I.getParent()->getParent();
    const TargetInstrInfo *TII = MF->getSubtarget().getInstrInfo();

    // Create a virtual register for the TLS base address.
    MachineRegisterInfo &RegInfo = MF->getRegInfo();
    *TLSBaseAddrReg = RegInfo.createVirtualRegister(&AArch64::GPR64RegClass);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 125-151: Function getPassName
```cpp

    // Insert a copy from X0 to TLSBaseAddrReg for later.
    MachineInstr *Copy =
        BuildMI(*I.getParent(), ++I.getIterator(), I.getDebugLoc(),
                TII->get(TargetOpcode::COPY), *TLSBaseAddrReg)
            .addReg(AArch64::X0);

    return Copy;
  }

  StringRef getPassName() const override { return TLSCLEANUP_PASS_NAME; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
}

INITIALIZE_PASS(LDTLSCleanup, "aarch64-local-dynamic-tls-cleanup",
                TLSCLEANUP_PASS_NAME, false, false)

char LDTLSCleanup::ID = 0;
FunctionPass *llvm::createAArch64CleanupLocalDynamicTLSPass() {
  return new LDTLSCleanup();
}
```
**EN:** This block implements getPassName, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getPassName，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, AArch64MachineFunctionInfo.h **CN:** 目标本地依赖：AArch64.h, AArch64MachineFunctionInfo.h
- **EN:** Core LLVM interfaces: llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h **CN:** 核心 LLVM 接口：llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
