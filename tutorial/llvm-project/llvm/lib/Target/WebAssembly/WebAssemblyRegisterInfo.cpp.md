# WebAssemblyRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyRegisterInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains the WebAssembly implementation of the TargetRegisterInfo class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyRegisterInfo.cpp`，主要负责 WebAssembly 后端的寄存器信息与栈帧交互逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyRegisterInfo.cpp - WebAssembly Register Information ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Register assignment and register-class constraints matter here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the WebAssembly implementation of the
/// TargetRegisterInfo class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyRegisterInfo.h"
#include "GISel/WebAssemblyRegisterBankInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssemblyFrameLowering.h"
#include "WebAssemblyInstrInfo.h"
```
- **EN**: Pulls in direct dependencies required by this register information and frame interaction logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该寄存器信息与栈帧交互逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 20-28

```cpp
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
#include "llvm/Target/TargetOptions.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this register information and frame interaction logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该寄存器信息与栈帧交互逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 29-66

```cpp
#define DEBUG_TYPE "wasm-reg-info"

#define GET_REGINFO_TARGET_DESC
#include "WebAssemblyGenRegisterInfo.inc"

WebAssemblyRegisterInfo::WebAssemblyRegisterInfo(const Triple &TT)
    : WebAssemblyGenRegisterInfo(0), TT(TT) {}

const MCPhysReg *
WebAssemblyRegisterInfo::getCalleeSavedRegs(const MachineFunction *) const {
  static const MCPhysReg CalleeSavedRegs[] = {0};
  return CalleeSavedRegs;
}

BitVector
WebAssemblyRegisterInfo::getReservedRegs(const MachineFunction & /*MF*/) const {
  BitVector Reserved(getNumRegs());
  for (auto Reg : {WebAssembly::SP32, WebAssembly::SP64, WebAssembly::FP32,
                   WebAssembly::FP64})
    Reserved.set(Reg);
  return Reserved;
}

bool WebAssemblyRegisterInfo::eliminateFrameIndex(
    MachineBasicBlock::iterator II, int SPAdj, unsigned FIOperandNum,
    RegScavenger * /*RS*/) const {
  assert(SPAdj == 0);
  MachineInstr &MI = *II;

  MachineBasicBlock &MBB = *MI.getParent();
  MachineFunction &MF = *MBB.getParent();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  int64_t FrameOffset = MFI.getStackSize() + MFI.getObjectOffset(FrameIndex);

  assert(MFI.getObjectSize(FrameIndex) != 0 &&
         "We assume that variable-sized objects have already been lowered, "
```
- **EN**: Pulls in direct dependencies required by this register information and frame interaction logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该寄存器信息与栈帧交互逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 67-104

```cpp
         "and don't use FrameIndex operands.");
  Register FrameRegister = getFrameRegister(MF);

  // If this is the address operand of a load or store, make it relative to SP
  // and fold the frame offset directly in.
  unsigned AddrOperandNum = WebAssembly::getNamedOperandIdx(
      MI.getOpcode(), WebAssembly::OpName::addr);
  if (AddrOperandNum == FIOperandNum) {
    unsigned OffsetOperandNum = WebAssembly::getNamedOperandIdx(
        MI.getOpcode(), WebAssembly::OpName::off);
    assert(FrameOffset >= 0 && MI.getOperand(OffsetOperandNum).getImm() >= 0);
    int64_t Offset = MI.getOperand(OffsetOperandNum).getImm() + FrameOffset;

    if (static_cast<uint64_t>(Offset) <= std::numeric_limits<uint32_t>::max()) {
      MI.getOperand(OffsetOperandNum).setImm(Offset);
      MI.getOperand(FIOperandNum)
          .ChangeToRegister(FrameRegister, /*isDef=*/false);
      return false;
    }
  }

  // If this is an address being added to a constant, fold the frame offset
  // into the constant.
  if (MI.getOpcode() == WebAssemblyFrameLowering::getOpcAdd(MF)) {
    MachineOperand &OtherMO = MI.getOperand(3 - FIOperandNum);
    if (OtherMO.isReg()) {
      Register OtherMOReg = OtherMO.getReg();
      if (OtherMOReg.isVirtual()) {
        MachineInstr *Def = MF.getRegInfo().getUniqueVRegDef(OtherMOReg);
        // TODO: For now we just opportunistically do this in the case where
        // the CONST_I32/64 happens to have exactly one def and one use. We
        // should generalize this to optimize in more cases.
        if (Def && Def->getOpcode() ==
              WebAssemblyFrameLowering::getOpcConst(MF) &&
            MRI.hasOneNonDBGUse(Def->getOperand(0).getReg())) {
          MachineOperand &ImmMO = Def->getOperand(1);
          if (ImmMO.isImm()) {
            ImmMO.setImm(ImmMO.getImm() + uint32_t(FrameOffset));
```
- **EN**: Implements helper routine(s) `getFrameRegister`, `getNamedOperandIdx`, `getOpcode` for this portion of the WebAssembly backend register information and frame interaction logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `getFrameRegister`, `getNamedOperandIdx`, `getOpcode`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 105-142

```cpp
            MI.getOperand(FIOperandNum)
                .ChangeToRegister(FrameRegister, /*isDef=*/false);
            return false;
          }
        }
      }
    }
  }

  // Otherwise create an i32/64.add SP, offset and make it the operand.
  const auto *TII = MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  unsigned FIRegOperand = FrameRegister;
  if (FrameOffset) {
    // Create i32/64.add SP, offset and make it the operand.
    const TargetRegisterClass *PtrRC =
        MRI.getTargetRegisterInfo()->getPointerRegClass();
    Register OffsetOp = MRI.createVirtualRegister(PtrRC);
    BuildMI(MBB, *II, II->getDebugLoc(),
            TII->get(WebAssemblyFrameLowering::getOpcConst(MF)),
            OffsetOp)
        .addImm(FrameOffset);
    FIRegOperand = MRI.createVirtualRegister(PtrRC);
    BuildMI(MBB, *II, II->getDebugLoc(),
            TII->get(WebAssemblyFrameLowering::getOpcAdd(MF)),
            FIRegOperand)
        .addReg(FrameRegister)
        .addReg(OffsetOp);
  }
  MI.getOperand(FIOperandNum).ChangeToRegister(FIRegOperand, /*isDef=*/false);
  return false;
}

Register
WebAssemblyRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  // If the PReg has been replaced by a VReg, return that.
  const auto &MFI = MF.getInfo<WebAssemblyFunctionInfo>();
  if (MFI->isFrameBaseVirtual())
```
- **EN**: Implements helper routine(s) `getOperand`, `ChangeToRegister`, `getInstrInfo` for this portion of the WebAssembly backend register information and frame interaction logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `getOperand`, `ChangeToRegister`, `getInstrInfo`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 143-180

```cpp
    return MFI->getFrameBaseVreg();
  static const unsigned Regs[2][2] = {
      /*            !isArch64Bit       isArch64Bit      */
      /* !hasFP */ {WebAssembly::SP32, WebAssembly::SP64},
      /*  hasFP */ {WebAssembly::FP32, WebAssembly::FP64}};
  const WebAssemblyFrameLowering *TFI = getFrameLowering(MF);
  return Regs[TFI->hasFP(MF)][TT.isArch64Bit()];
}

const TargetRegisterClass *
WebAssemblyRegisterInfo::getPointerRegClass(unsigned Kind) const {
  assert(Kind == 0 && "Only one kind of pointer on WebAssembly");
  return TT.getArch() == Triple::wasm64 ? &WebAssembly::I64RegClass
                                        : &WebAssembly::I32RegClass;
}

static const TargetRegisterClass &getRegClassForBank(const RegisterBank &RB) {
  switch (RB.getID()) {
  case WebAssembly::I32RegBankID:
    return WebAssembly::I32RegClass;
  case WebAssembly::I64RegBankID:
    return WebAssembly::I64RegClass;
  case WebAssembly::F32RegBankID:
    return WebAssembly::F32RegClass;
  case WebAssembly::F64RegBankID:
    return WebAssembly::F64RegClass;
  case WebAssembly::EXNREFRegBankID:
    return WebAssembly::EXNREFRegClass;
  case WebAssembly::EXTERNREFRegBankID:
    return WebAssembly::EXTERNREFRegClass;
  case WebAssembly::FUNCREFRegBankID:
    return WebAssembly::FUNCREFRegClass;
  case WebAssembly::V128RegBankID:
    return WebAssembly::V128RegClass;
  default:
    llvm_unreachable("Found unexpected RegisterBank in `getRegClassForBank`");
  }
}
```
- **EN**: Implements helper routine(s) `getFrameBaseVreg`, `getFrameLowering`, `hasFP` for this portion of the WebAssembly backend register information and frame interaction logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `getFrameBaseVreg`, `getFrameLowering`, `hasFP`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 181-202

```cpp

const TargetRegisterClass *
WebAssemblyRegisterInfo::getConstrainedRegClassForOperand(
    const MachineOperand &MO, const MachineRegisterInfo &MRI) const {
  assert(MO.isReg());

  const RegClassOrRegBank &RegClassOrBank =
      MRI.getRegClassOrRegBank(MO.getReg());

  if (RegClassOrBank.isNull())
    return nullptr;

  const TargetRegisterClass *DefRC =
      dyn_cast<const TargetRegisterClass *>(RegClassOrBank);

  if (!DefRC) {
    const RegisterBank &RB = *cast<const RegisterBank *>(RegClassOrBank);
    DefRC = &getRegClassForBank(RB);
  }

  return DefRC;
}
```
- **EN**: Implements helper routine(s) `getConstrainedRegClassForOperand`, `isReg`, `getRegClassOrRegBank` for this portion of the WebAssembly backend register information and frame interaction logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `getConstrainedRegClassForOperand`, `isReg`, `getRegClassOrRegBank`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Register information and frame interaction logic / 寄存器信息与栈帧交互逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyRegisterInfo.h`
- `GISel/WebAssemblyRegisterBankInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssemblyFrameLowering.h`
- `WebAssemblyInstrInfo.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/Target/TargetOptions.h`
- `WebAssemblyGenRegisterInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
