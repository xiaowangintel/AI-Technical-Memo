# PPCTLSDynamicCall.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCTLSDynamicCall.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCTLSDynamicCall.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===---------- PPCTLSDynamicCall.cpp - TLS Dynamic Call Fixup ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-20

```cpp
//===----------------------------------------------------------------------===//
//
// This pass expands ADDItls{ld,gd}LADDR[32] machine instructions into
// separate ADDItls[gd]L[32] and GETtlsADDR[32] instructions, both of
// which define GPR3.  A copy is added from GPR3 to the target virtual
// register of the original instruction.  The GETtlsADDR[32] is really
// a call instruction, so its target register is constrained to be GPR3.
// This is not true of ADDItls[gd]L[32], but there is a legacy linker
// optimization bug that requires the target register of the addi of
// a local- or general-dynamic TLS access sequence to be GPR3.
//
// This is done in a late pass so that TLS variable accesses can be
// fully commoned by MachineCSE.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pass expands ADDItls{ld,gd}LADDR[32] machine instructions into".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pass expands ADDItls{ld,gd}LADDR[32] machine instructions into”。

### Lines 21-27

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCTargetMachine.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 28-35

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/InitializePasses.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 36-73

```cpp
#define DEBUG_TYPE "ppc-tls-dynamic-call"

namespace {
  struct PPCTLSDynamicCall : public MachineFunctionPass {
    static char ID;
    PPCTLSDynamicCall() : MachineFunctionPass(ID) {}

    const PPCInstrInfo *TII;

protected:
    bool processBlock(MachineBasicBlock &MBB) {
      bool Changed = false;
      bool NeedFence = true;
      const PPCSubtarget &Subtarget =
          MBB.getParent()->getSubtarget<PPCSubtarget>();
      bool Is64Bit = Subtarget.isPPC64();
      bool IsAIX = Subtarget.isAIXABI();
      bool IsLargeModel =
          Subtarget.getTargetMachine().getCodeModel() == CodeModel::Large;
      bool IsPCREL = false;
      MachineFunction *MF = MBB.getParent();
      MachineRegisterInfo &RegInfo = MF->getRegInfo();

      for (MachineBasicBlock::iterator I = MBB.begin(), IE = MBB.end();
           I != IE;) {
        MachineInstr &MI = *I;
        IsPCREL = isPCREL(MI);
        // There are a number of slight differences in code generation
        // when we call .__get_tpointer (32-bit AIX TLS).
        bool IsTLSTPRelMI = MI.getOpcode() == PPC::GETtlsTpointer32AIX;
        bool IsTLSLDAIXMI = (MI.getOpcode() == PPC::TLSLDAIX8 ||
                             MI.getOpcode() == PPC::TLSLDAIX);

        if (MI.getOpcode() != PPC::ADDItlsgdLADDR &&
            MI.getOpcode() != PPC::ADDItlsldLADDR &&
            MI.getOpcode() != PPC::ADDItlsgdLADDR32 &&
            MI.getOpcode() != PPC::ADDItlsldLADDR32 &&
            MI.getOpcode() != PPC::TLSGDAIX &&
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 74-111

```cpp
            MI.getOpcode() != PPC::TLSGDAIX8 && !IsTLSTPRelMI && !IsPCREL &&
            !IsTLSLDAIXMI) {
          // Although we create ADJCALLSTACKDOWN and ADJCALLSTACKUP
          // as scheduling fences, we skip creating fences if we already
          // have existing ADJCALLSTACKDOWN/UP to avoid nesting,
          // which causes verification error with -verify-machineinstrs.
          if (MI.getOpcode() == PPC::ADJCALLSTACKDOWN)
            NeedFence = false;
          else if (MI.getOpcode() == PPC::ADJCALLSTACKUP)
            NeedFence = true;

          ++I;
          continue;
        }

        LLVM_DEBUG(dbgs() << "TLS Dynamic Call Fixup:\n    " << MI);

        Register OutReg = MI.getOperand(0).getReg();
        Register InReg = PPC::NoRegister;
        Register GPR3 = Is64Bit ? PPC::X3 : PPC::R3;
        Register GPR4 = Is64Bit ? PPC::X4 : PPC::R4;
        if (!IsPCREL && !IsTLSTPRelMI)
          InReg = MI.getOperand(1).getReg();
        DebugLoc DL = MI.getDebugLoc();

        unsigned Opc1, Opc2;
        switch (MI.getOpcode()) {
        default:
          llvm_unreachable("Opcode inconsistency error");
        case PPC::ADDItlsgdLADDR:
          Opc1 = PPC::ADDItlsgdL;
          Opc2 = PPC::GETtlsADDR;
          break;
        case PPC::ADDItlsldLADDR:
          Opc1 = PPC::ADDItlsldL;
          Opc2 = PPC::GETtlsldADDR;
          break;
        case PPC::ADDItlsgdLADDR32:
```
- **EN**: Implements helper routine(s) `getOpcode`, `dbgs`, `getOperand` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `dbgs`, `getOperand`。

### Lines 112-149

```cpp
          Opc1 = PPC::ADDItlsgdL32;
          Opc2 = PPC::GETtlsADDR32;
          break;
        case PPC::ADDItlsldLADDR32:
          Opc1 = PPC::ADDItlsldL32;
          Opc2 = PPC::GETtlsldADDR32;
          break;
        case PPC::TLSLDAIX:
          // TLSLDAIX is expanded to one copy and GET_TLS_MOD, so we only set
          // Opc2 here.
          Opc2 = PPC::GETtlsMOD32AIX;
          break;
        case PPC::TLSLDAIX8:
          // TLSLDAIX8 is expanded to one copy and GET_TLS_MOD, so we only set
          // Opc2 here.
          Opc2 = PPC::GETtlsMOD64AIX;
          break;
        case PPC::TLSGDAIX8:
          // TLSGDAIX8 is expanded to two copies and GET_TLS_ADDR, so we only
          // set Opc2 here.
          Opc2 = PPC::GETtlsADDR64AIX;
          break;
        case PPC::TLSGDAIX:
          // TLSGDAIX is expanded to two copies and GET_TLS_ADDR, so we only
          // set Opc2 here.
          Opc2 = PPC::GETtlsADDR32AIX;
          break;
        case PPC::GETtlsTpointer32AIX:
          // GETtlsTpointer32AIX is expanded to a call to GET_TPOINTER on AIX
          // 32-bit mode within PPCAsmPrinter. This instruction does not need
          // to change, so Opc2 is set to the same instruction opcode.
          Opc2 = PPC::GETtlsTpointer32AIX;
          break;
        case PPC::PADDI8pc:
          assert(IsPCREL && "Expecting General/Local Dynamic PCRel");
          Opc1 = PPC::PADDI8pc;
          Opc2 = MI.getOperand(2).getTargetFlags() ==
                         PPCII::MO_GOT_TLSGD_PCREL_FLAG
```
- **EN**: Declares function entry points including `getOperand`, `getTargetFlags` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getOperand`, `getTargetFlags`。

### Lines 150-187

```cpp
                     ? PPC::GETtlsADDRPCREL
                     : PPC::GETtlsldADDRPCREL;
        }

        // We create ADJCALLSTACKUP and ADJCALLSTACKDOWN around _tls_get_addr
        // as scheduling fence to avoid it is scheduled before
        // mflr in the prologue and the address in LR is clobbered (PR25839).
        // We don't really need to save data to the stack - the clobbered
        // registers are already saved when the SDNode (e.g. PPCaddiTlsgdLAddr)
        // gets translated to the pseudo instruction (e.g. ADDItlsgdLADDR).
        if (NeedFence) {
          MBB.getParent()->getFrameInfo().setAdjustsStack(true);
          BuildMI(MBB, I, DL, TII->get(PPC::ADJCALLSTACKDOWN)).addImm(0)
                                                              .addImm(0);
        }

        if (IsAIX) {
          if (IsTLSLDAIXMI) {
            // The relative order between the node that loads the variable
            // offset from the TOC, and the .__tls_get_mod node is being tuned
            // here. It is better to put the variable offset TOC load after the
            // call, since this node can use clobbers r4/r5.
            // Search for the pattern of the two nodes that load from the TOC
            // (either for the variable offset or for the module handle), and
            // then move the variable offset TOC load right before the node that
            // uses the OutReg of the .__tls_get_mod node.
            unsigned LDTocOp =
                Is64Bit ? (IsLargeModel ? PPC::LDtocL : PPC::LDtoc)
                        : (IsLargeModel ? PPC::LWZtocL : PPC::LWZtoc);
            if (!RegInfo.use_empty(OutReg)) {
              std::set<MachineInstr *> Uses;
              // Collect all instructions that use the OutReg.
              for (MachineOperand &MO : RegInfo.use_operands(OutReg))
                Uses.insert(MO.getParent());
              // Find the first user (e.g.: lwax/stfdx) of the OutReg within the
              // current BB.
              MachineBasicBlock::iterator UseIter = MBB.begin();
              for (MachineBasicBlock::iterator IE = MBB.end(); UseIter != IE;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We create ADJCALLSTACKUP and ADJCALLSTACKDOWN around _tls_get_addr". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We create ADJCALLSTACKUP and ADJCALLSTACKDOWN around _tls_get_addr”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 188-225

```cpp
                   ++UseIter)
                if (Uses.count(&*UseIter))
                  break;

              // Additional handling is required when UserIter (the first user
              // of OutReg) is pointing to a valid node that loads from the TOC.
              // Check the pattern and do the movement if the pattern matches.
              if (UseIter != MBB.end()) {
                // Collect all associated nodes that load from the TOC. Use
                // hasOneDef() to guard against unexpected scenarios.
                std::set<MachineInstr *> LoadFromTocs;
                for (MachineOperand &MO : UseIter->operands())
                  if (MO.isReg() && MO.isUse()) {
                    Register MOReg = MO.getReg();
                    if (RegInfo.hasOneDef(MOReg)) {
                      MachineInstr *Temp =
                          RegInfo.getOneDef(MOReg)->getParent();
                      // For the current TLSLDAIX node, get the corresponding
                      // node that loads from the TOC for the InReg. Otherwise,
                      // Temp probably pointed to the variable offset TOC load
                      // we would like to move.
                      if (Temp == &MI && RegInfo.hasOneDef(InReg))
                        Temp = RegInfo.getOneDef(InReg)->getParent();
                      if (Temp->getOpcode() == LDTocOp)
                        LoadFromTocs.insert(Temp);
                    } else {
                      // FIXME: analyze this scenario if there is one.
                      LoadFromTocs.clear();
                      break;
                    }
                  }

                // Check the two nodes that loaded from the TOC: one should be
                // "_$TLSML", and the other will be moved before the node that
                // uses the OutReg of the .__tls_get_mod node.
                if (LoadFromTocs.size() == 2) {
                  MachineBasicBlock::iterator TLSMLIter = MBB.end();
                  MachineBasicBlock::iterator OffsetIter = MBB.end();
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Additional handling is required when UserIter (the first user". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Additional handling is required when UserIter (the first user”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 226-263

```cpp
                  // Make sure the two nodes that loaded from the TOC are within
                  // the current BB, and that one of them is from the "_$TLSML"
                  // pseudo symbol, while the other is from the variable.
                  for (MachineBasicBlock::iterator I = MBB.begin(),
                                                   IE = MBB.end();
                       I != IE; ++I)
                    if (LoadFromTocs.count(&*I)) {
                      MachineOperand MO = I->getOperand(1);
                      if (MO.isGlobal() && MO.getGlobal()->hasName() &&
                          MO.getGlobal()->getName() == "_$TLSML")
                        TLSMLIter = I;
                      else
                        OffsetIter = I;
                    }
                  // Perform the movement when the desired scenario has been
                  // identified, which should be when both of the iterators are
                  // valid.
                  if (TLSMLIter != MBB.end() && OffsetIter != MBB.end())
                    OffsetIter->moveBefore(&*UseIter);
                }
              }
            }
            // The module-handle is copied into r3. The copy is followed by
            // GETtlsMOD32AIX/GETtlsMOD64AIX.
            BuildMI(MBB, I, DL, TII->get(TargetOpcode::COPY), GPR3)
                .addReg(InReg);
            // The call to .__tls_get_mod.
            BuildMI(MBB, I, DL, TII->get(Opc2), GPR3).addReg(GPR3);
          } else if (!IsTLSTPRelMI) {
            // The variable offset and region handle (for TLSGD) are copied in
            // r4 and r3. The copies are followed by
            // GETtlsADDR32AIX/GETtlsADDR64AIX.
            BuildMI(MBB, I, DL, TII->get(TargetOpcode::COPY), GPR4)
                .addReg(MI.getOperand(1).getReg());
            BuildMI(MBB, I, DL, TII->get(TargetOpcode::COPY), GPR3)
                .addReg(MI.getOperand(2).getReg());
            BuildMI(MBB, I, DL, TII->get(Opc2), GPR3).addReg(GPR3).addReg(GPR4);
          } else
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Make sure the two nodes that loaded from the TOC are within". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Make sure the two nodes that loaded from the TOC are within”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 264-301

```cpp
            // The opcode of GETtlsTpointer32AIX does not change, because later
            // this instruction will be expanded into a call to .__get_tpointer,
            // which will return the thread pointer into r3.
            BuildMI(MBB, I, DL, TII->get(Opc2), GPR3);
        } else {
          MachineInstr *Addi;
          if (IsPCREL) {
            Addi = BuildMI(MBB, I, DL, TII->get(Opc1), GPR3).addImm(0);
          } else {
            // Expand into two ops built prior to the existing instruction.
            assert(InReg != PPC::NoRegister && "Operand must be a register");
            Addi = BuildMI(MBB, I, DL, TII->get(Opc1), GPR3).addReg(InReg);
          }

          Addi->addOperand(MI.getOperand(2));

          MachineInstr *Call =
              (BuildMI(MBB, I, DL, TII->get(Opc2), GPR3).addReg(GPR3));
          if (IsPCREL)
            Call->addOperand(MI.getOperand(2));
          else
            Call->addOperand(MI.getOperand(3));
        }
        if (NeedFence)
          BuildMI(MBB, I, DL, TII->get(PPC::ADJCALLSTACKUP)).addImm(0).addImm(0);

        BuildMI(MBB, I, DL, TII->get(TargetOpcode::COPY), OutReg)
          .addReg(GPR3);

        // Move past the original instruction and remove it.
        ++I;
        MI.removeFromParent();

        Changed = true;
      }

      return Changed;
    }
```
- **EN**: Implements helper routine(s) `BuildMI`, `get`, `addImm` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `BuildMI`, `get`, `addImm`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 302-339

```cpp

public:
  bool isPCREL(const MachineInstr &MI) {
    return (MI.getOpcode() == PPC::PADDI8pc) &&
           (MI.getOperand(2).getTargetFlags() ==
                PPCII::MO_GOT_TLSGD_PCREL_FLAG ||
            MI.getOperand(2).getTargetFlags() ==
                PPCII::MO_GOT_TLSLD_PCREL_FLAG);
  }

    bool runOnMachineFunction(MachineFunction &MF) override {
      TII = MF.getSubtarget<PPCSubtarget>().getInstrInfo();

      bool Changed = false;

      for (MachineBasicBlock &B : llvm::make_early_inc_range(MF))
        if (processBlock(B))
          Changed = true;

      return Changed;
    }

    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.addRequired<LiveIntervalsWrapperPass>();
      AU.addRequired<SlotIndexesWrapperPass>();
      MachineFunctionPass::getAnalysisUsage(AU);
    }
  };
}

INITIALIZE_PASS_BEGIN(PPCTLSDynamicCall, DEBUG_TYPE,
                      "PowerPC TLS Dynamic Call Fixup", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(SlotIndexesWrapperPass)
INITIALIZE_PASS_END(PPCTLSDynamicCall, DEBUG_TYPE,
                    "PowerPC TLS Dynamic Call Fixup", false, false)

char PPCTLSDynamicCall::ID = 0;
```
- **EN**: Implements helper routine(s) `isPCREL`, `getOpcode`, `getOperand` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isPCREL`, `getOpcode`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 340-341

```cpp
FunctionPass*
llvm::createPPCTLSDynamicCallPass() { return new PPCTLSDynamicCall(); }
```
- **EN**: Implements helper routine(s) `createPPCTLSDynamicCallPass`, `PPCTLSDynamicCall` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `createPPCTLSDynamicCallPass`, `PPCTLSDynamicCall`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `PPCTargetMachine.h`
- `llvm/CodeGen/LiveIntervals.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/InitializePasses.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
