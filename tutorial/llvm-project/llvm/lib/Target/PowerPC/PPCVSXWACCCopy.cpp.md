# PPCVSXWACCCopy.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCVSXWACCCopy.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCVSXWACCCopy.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===--------- PPCVSXWACCCopy.cpp - VSX and WACC Copy Legalization --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
//
// A pass which deals with the complexity of generating legal VSX register
// copies to/from register classes which partially overlap with the VSX
// register file and combines the wacc/wacc_hi copies when needed.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A pass which deals with the complexity of generating legal VSX register".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A pass which deals with the complexity of generating legal VSX register”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Statistic.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-28

```cpp
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 29-66

```cpp
#define DEBUG_TYPE "ppc-vsx-copy"

namespace {
// PPCVSXWACCCopy pass - For copies between VSX registers and non-VSX registers
// (Altivec and scalar floating-point registers), we need to transform the
// copies into subregister copies with other restrictions.
struct PPCVSXWACCCopy : public MachineFunctionPass {
  static char ID;
  PPCVSXWACCCopy() : MachineFunctionPass(ID) {}

  const TargetInstrInfo *TII;

  bool IsRegInClass(unsigned Reg, const TargetRegisterClass *RC,
                    MachineRegisterInfo &MRI) {
    if (Register::isVirtualRegister(Reg)) {
      return RC->hasSubClassEq(MRI.getRegClass(Reg));
    } else if (RC->contains(Reg)) {
      return true;
    }

    return false;
  }

  bool IsVSReg(unsigned Reg, MachineRegisterInfo &MRI) {
    return IsRegInClass(Reg, &PPC::VSRCRegClass, MRI);
  }

  bool IsVRReg(unsigned Reg, MachineRegisterInfo &MRI) {
    return IsRegInClass(Reg, &PPC::VRRCRegClass, MRI);
  }

  bool IsF8Reg(unsigned Reg, MachineRegisterInfo &MRI) {
    return IsRegInClass(Reg, &PPC::F8RCRegClass, MRI);
  }

  bool IsVSFReg(unsigned Reg, MachineRegisterInfo &MRI) {
    return IsRegInClass(Reg, &PPC::VSFRCRegClass, MRI);
  }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Register assignment and register-class constraints matter here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 67-104

```cpp

  bool IsVSSReg(unsigned Reg, MachineRegisterInfo &MRI) {
    return IsRegInClass(Reg, &PPC::VSSRCRegClass, MRI);
  }

protected:
  bool processBlock(MachineBasicBlock &MBB) {
    bool Changed = false;

    MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
    for (MachineInstr &MI : MBB) {
      if (!MI.isFullCopy())
        continue;

      MachineOperand &DstMO = MI.getOperand(0);
      MachineOperand &SrcMO = MI.getOperand(1);

      if (IsVSReg(DstMO.getReg(), MRI) && !IsVSReg(SrcMO.getReg(), MRI)) {
        // This is a copy *to* a VSX register from a non-VSX register.
        Changed = true;

        const TargetRegisterClass *SrcRC = &PPC::VSLRCRegClass;
        assert((IsF8Reg(SrcMO.getReg(), MRI) || IsVSSReg(SrcMO.getReg(), MRI) ||
                IsVSFReg(SrcMO.getReg(), MRI)) &&
               "Unknown source for a VSX copy");

        Register NewVReg = MRI.createVirtualRegister(SrcRC);
        BuildMI(MBB, MI, MI.getDebugLoc(),
                TII->get(TargetOpcode::SUBREG_TO_REG), NewVReg)
            .add(SrcMO)
            .addImm(PPC::sub_64);

        // The source of the original copy is now the new virtual register.
        SrcMO.setReg(NewVReg);
      } else if (!IsVSReg(DstMO.getReg(), MRI) &&
                 IsVSReg(SrcMO.getReg(), MRI)) {
        // This is a copy *from* a VSX register to a non-VSX register.
        Changed = true;
```
- **EN**: Implements helper routine(s) `IsVSSReg`, `IsRegInClass`, `processBlock` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `IsVSSReg`, `IsRegInClass`, `processBlock`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 105-142

```cpp

        const TargetRegisterClass *DstRC = &PPC::VSLRCRegClass;
        assert((IsF8Reg(DstMO.getReg(), MRI) || IsVSFReg(DstMO.getReg(), MRI) ||
                IsVSSReg(DstMO.getReg(), MRI)) &&
               "Unknown destination for a VSX copy");

        // Copy the VSX value into a new VSX register of the correct subclass.
        Register NewVReg = MRI.createVirtualRegister(DstRC);
        BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(TargetOpcode::COPY),
                NewVReg)
            .add(SrcMO);

        // Transform the original copy into a subregister extraction copy.
        SrcMO.setReg(NewVReg);
        SrcMO.setSubReg(PPC::sub_64);
      } else if (IsRegInClass(DstMO.getReg(), &PPC::WACC_HIRCRegClass, MRI) &&
                 IsRegInClass(SrcMO.getReg(), &PPC::WACCRCRegClass, MRI)) {
        // Matches the pattern:
        //   %a:waccrc = COPY %b.sub_wacc_hi:dmrrc
        //   %c:wacc_hirc = COPY %a:waccrc
        // And replaces it with:
        //   %c:wacc_hirc = COPY %b.sub_wacc_hi:dmrrc
        MachineInstr *DefMI = MRI.getUniqueVRegDef(SrcMO.getReg());
        if (!DefMI || !DefMI->isCopy())
          continue;

        MachineOperand &OrigSrc = DefMI->getOperand(1);

        if (!IsRegInClass(OrigSrc.getReg(), &PPC::DMRRCRegClass, MRI))
          continue;

        if (OrigSrc.getSubReg() != PPC::sub_wacc_hi)
          continue;

        // Rewrite the second copy to use the original register's subreg
        SrcMO.setReg(OrigSrc.getReg());
        SrcMO.setSubReg(PPC::sub_wacc_hi);
        Changed = true;
```
- **EN**: Implements helper routine(s) `IsF8Reg`, `getReg`, `IsVSFReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `IsF8Reg`, `getReg`, `IsVSFReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 143-180

```cpp

        // Remove the intermediate copy if safe
        if (MRI.use_nodbg_empty(DefMI->getOperand(0).getReg()))
          DefMI->eraseFromParent();
      }
    }

    return Changed;
  }

public:
  bool runOnMachineFunction(MachineFunction &MF) override {
    // If we don't have VSX on the subtarget, don't do anything.
    const PPCSubtarget &STI = MF.getSubtarget<PPCSubtarget>();
    if (!STI.hasVSX())
      return false;
    TII = STI.getInstrInfo();

    bool Changed = false;

    for (MachineBasicBlock &B : llvm::make_early_inc_range(MF))
      if (processBlock(B))
        Changed = true;

    return Changed;
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
} // end anonymous namespace

INITIALIZE_PASS(PPCVSXWACCCopy, DEBUG_TYPE, "PowerPC VSX Copy Legalization",
                false, false)

char PPCVSXWACCCopy::ID = 0;
FunctionPass *llvm::createPPCVSXWACCCopyPass() { return new PPCVSXWACCCopy(); }
```
- **EN**: Implements helper routine(s) `use_nodbg_empty`, `getOperand`, `getReg` for this portion of the PowerPC backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `use_nodbg_empty`, `getOperand`, `getReg`。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `PPCTargetMachine.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineMemOperand.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/Support/ErrorHandling.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
