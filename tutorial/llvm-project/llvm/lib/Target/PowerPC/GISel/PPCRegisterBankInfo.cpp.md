# PPCRegisterBankInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/GISel/PPCRegisterBankInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the targeting of the RegisterBankInfo class for PowerPC.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/GISel/PPCRegisterBankInfo.cpp`，主要负责 PowerPC 后端的寄存器银行映射逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- PPCRegisterBankInfo.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
/// \file
/// This file implements the targeting of the RegisterBankInfo class for
/// PowerPC.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPCRegisterBankInfo.h"
#include "PPCRegisterInfo.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/MachineFunction.h"
```
- **EN**: Pulls in direct dependencies required by this register bank mapping logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该寄存器银行映射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 18-25

```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"

#define DEBUG_TYPE "ppc-reg-bank-info"

#define GET_TARGET_REGBANK_IMPL
#include "PPCGenRegisterBank.inc"

// This file will be TableGen'ed at some point.
```
- **EN**: Pulls in direct dependencies required by this register bank mapping logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该寄存器银行映射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 26-63

```cpp
#include "PPCGenRegisterBankInfo.def"

using namespace llvm;

PPCRegisterBankInfo::PPCRegisterBankInfo(const TargetRegisterInfo &TRI) {}

const RegisterBank &
PPCRegisterBankInfo::getRegBankFromRegClass(const TargetRegisterClass &RC,
                                            LLT Ty) const {
  switch (RC.getID()) {
  case PPC::VSFRCRegClassID:
  case PPC::SPILLTOVSRRC_and_VSFRCRegClassID:
  case PPC::SPILLTOVSRRC_and_VFRCRegClassID:
  case PPC::SPILLTOVSRRC_and_F4RCRegClassID:
  case PPC::F8RCRegClassID:
  case PPC::VFRCRegClassID:
  case PPC::VSSRCRegClassID:
  case PPC::F4RCRegClassID:
    return getRegBank(PPC::FPRRegBankID);
  default:
    return PPCGenRegisterBankInfo::getRegBankFromRegClass(RC, Ty);
  }
}

const RegisterBankInfo::InstructionMapping &
PPCRegisterBankInfo::getInstrMapping(const MachineInstr &MI) const {
  const unsigned Opc = MI.getOpcode();

  // Try the default logic for non-generic instructions that are either copies
  // or already have some operands assigned to banks.
  if (!isPreISelGenericOpcode(Opc) || Opc == TargetOpcode::G_PHI) {
    const RegisterBankInfo::InstructionMapping &Mapping =
        getInstrMappingImpl(MI);
    if (Mapping.isValid())
      return Mapping;
  }

  const MachineFunction &MF = *MI.getParent()->getParent();
```
- **EN**: Pulls in direct dependencies required by this register bank mapping logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该寄存器银行映射逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 64-101

```cpp
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const TargetSubtargetInfo &STI = MF.getSubtarget();
  const TargetRegisterInfo &TRI = *STI.getRegisterInfo();

  unsigned NumOperands = MI.getNumOperands();
  const ValueMapping *OperandsMapping = nullptr;
  unsigned Cost = 1;
  unsigned MappingID = DefaultMappingID;

  switch (Opc) {
    // Arithmetic ops.
  case TargetOpcode::G_ADD:
  case TargetOpcode::G_SUB:
    // Bitwise ops.
  case TargetOpcode::G_AND:
  case TargetOpcode::G_OR:
  case TargetOpcode::G_XOR:
    // Extension ops.
  case TargetOpcode::G_SEXT:
  case TargetOpcode::G_ZEXT:
  case TargetOpcode::G_ANYEXT: {
    assert(NumOperands <= 3 &&
           "This code is for instructions with 3 or less operands");
    LLT Ty = MRI.getType(MI.getOperand(0).getReg());
    unsigned Size = Ty.getSizeInBits();
    switch (Size) {
    case 128:
      OperandsMapping = getValueMapping(PMI_VEC128);
      break;
    default:
      OperandsMapping = getValueMapping(PMI_GPR64);
      break;
    }
    break;
  }
  case TargetOpcode::G_FADD:
  case TargetOpcode::G_FSUB:
  case TargetOpcode::G_FMUL:
```
- **EN**: Implements helper routine(s) `getRegInfo`, `getSubtarget`, `getRegisterInfo` for this portion of the PowerPC backend register bank mapping logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getRegInfo`, `getSubtarget`, `getRegisterInfo`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 102-139

```cpp
  case TargetOpcode::G_FDIV: {
    Register SrcReg = MI.getOperand(1).getReg();
    unsigned Size = getSizeInBits(SrcReg, MRI, TRI);

    assert((Size == 32 || Size == 64 || Size == 128) &&
           "Unsupported floating point types!\n");
    switch (Size) {
    case 32:
      OperandsMapping = getValueMapping(PMI_FPR32);
      break;
    case 64:
      OperandsMapping = getValueMapping(PMI_FPR64);
      break;
    case 128:
      OperandsMapping = getValueMapping(PMI_VEC128);
      break;
    }
    break;
  }
  case TargetOpcode::G_FCMP: {
    unsigned CmpSize = MRI.getType(MI.getOperand(2).getReg()).getSizeInBits();

    OperandsMapping = getOperandsMapping(
        {getValueMapping(PMI_CR), nullptr,
         getValueMapping(CmpSize == 32 ? PMI_FPR32 : PMI_FPR64),
         getValueMapping(CmpSize == 32 ? PMI_FPR32 : PMI_FPR64)});
    break;
  }
  case TargetOpcode::G_CONSTANT:
    OperandsMapping = getOperandsMapping({getValueMapping(PMI_GPR64), nullptr});
    break;
  case TargetOpcode::G_CONSTANT_POOL:
    OperandsMapping = getOperandsMapping({getValueMapping(PMI_GPR64), nullptr});
    break;
  case TargetOpcode::G_FPTOUI:
  case TargetOpcode::G_FPTOSI: {
    Register SrcReg = MI.getOperand(1).getReg();
    unsigned Size = getSizeInBits(SrcReg, MRI, TRI);
```
- **EN**: Implements helper routine(s) `getOperand`, `getReg`, `getSizeInBits` for this portion of the PowerPC backend register bank mapping logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getOperand`, `getReg`, `getSizeInBits`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 140-177

```cpp

    OperandsMapping = getOperandsMapping(
        {getValueMapping(PMI_GPR64),
         getValueMapping(Size == 32 ? PMI_FPR32 : PMI_FPR64)});
    break;
  }
  case TargetOpcode::G_UITOFP:
  case TargetOpcode::G_SITOFP: {
    Register SrcReg = MI.getOperand(0).getReg();
    unsigned Size = getSizeInBits(SrcReg, MRI, TRI);

    OperandsMapping =
        getOperandsMapping({getValueMapping(Size == 32 ? PMI_FPR32 : PMI_FPR64),
                            getValueMapping(PMI_GPR64)});
    break;
  }
  case TargetOpcode::G_LOAD: {
    unsigned Size = MRI.getType(MI.getOperand(0).getReg()).getSizeInBits();
    // Check if that load feeds fp instructions.
    if (any_of(MRI.use_nodbg_instructions(MI.getOperand(0).getReg()),
               [&](const MachineInstr &UseMI) {
                 // If we have at least one direct use in a FP instruction,
                 // assume this was a floating point load in the IR. If it was
                 // not, we would have had a bitcast before reaching that
                 // instruction.
                 //
                 // Int->FP conversion operations are also captured in
                 // onlyDefinesFP().
                 return onlyUsesFP(UseMI, MRI, TRI);
               }))
      OperandsMapping = getOperandsMapping(
          {getValueMapping(Size == 64 ? PMI_FPR64 : PMI_FPR32),
           getValueMapping(PMI_GPR64)});
    else
      OperandsMapping = getOperandsMapping(
          {getValueMapping(Size == 64 ? PMI_GPR64 : PMI_GPR32),
           getValueMapping(PMI_GPR64)});
    break;
```
- **EN**: Implements helper routine(s) `getOperandsMapping`, `getValueMapping`, `getOperand` for this portion of the PowerPC backend register bank mapping logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getOperandsMapping`, `getValueMapping`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 178-215

```cpp
  }
  case TargetOpcode::G_STORE: {
    // Check if the store is fed by fp instructions.
    MachineInstr *DefMI = MRI.getVRegDef(MI.getOperand(0).getReg());
    unsigned Size = MRI.getType(MI.getOperand(0).getReg()).getSizeInBits();
    if (onlyDefinesFP(*DefMI, MRI, TRI))
      OperandsMapping = getOperandsMapping(
          {getValueMapping(Size == 64 ? PMI_FPR64 : PMI_FPR32),
           getValueMapping(PMI_GPR64)});
    else
      OperandsMapping = getOperandsMapping(
          {getValueMapping(Size == 64 ? PMI_GPR64 : PMI_GPR32),
           getValueMapping(PMI_GPR64)});
    break;
  }
  case TargetOpcode::G_INTRINSIC_W_SIDE_EFFECTS: {
    // FIXME: We have to check every operand in this MI and compute value
    // mapping accordingly.
    SmallVector<const ValueMapping *, 8> OpdsMapping(NumOperands);
    OperandsMapping = getOperandsMapping(OpdsMapping);
    break;
  }
  case TargetOpcode::G_BITCAST: {
    LLT DstTy = MRI.getType(MI.getOperand(0).getReg());
    LLT SrcTy = MRI.getType(MI.getOperand(1).getReg());
    unsigned DstSize = DstTy.getSizeInBits();

    bool DstIsGPR = !DstTy.isVector();
    bool SrcIsGPR = !SrcTy.isVector();
    // TODO: Currently, only vector and GPR register banks are handled.
    //       This needs to be extended to handle floating point register
    //       banks in the future.
    const RegisterBank &DstRB = DstIsGPR ? PPC::GPRRegBank : PPC::VECRegBank;
    const RegisterBank &SrcRB = SrcIsGPR ? PPC::GPRRegBank : PPC::VECRegBank;

    return getInstructionMapping(
        MappingID, Cost, getCopyMapping(DstRB.getID(), SrcRB.getID(), DstSize),
        NumOperands);
```
- **EN**: Implements helper routine(s) `getVRegDef`, `getOperand`, `getReg` for this portion of the PowerPC backend register bank mapping logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getVRegDef`, `getOperand`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 216-253

```cpp
  }
  default:
    return getInvalidInstructionMapping();
  }

  return getInstructionMapping(MappingID, Cost, OperandsMapping, NumOperands);
}

/// \returns true if a given intrinsic \p ID only uses and defines FPRs.
static bool isFPIntrinsic(unsigned ID) {
  // TODO: Add more intrinsics.
  return false;
}

/// FIXME: this is copied from target AArch64. Needs some code refactor here to
/// put this function in class RegisterBankInfo.
bool PPCRegisterBankInfo::hasFPConstraints(const MachineInstr &MI,
                                           const MachineRegisterInfo &MRI,
                                           const TargetRegisterInfo &TRI,
                                           unsigned Depth) const {
  unsigned Op = MI.getOpcode();

  if (auto *GI = dyn_cast<GIntrinsic>(&MI)) {
    if (isFPIntrinsic(GI->getIntrinsicID()))
      return true;
  }

  // Do we have an explicit floating point instruction?
  if (isPreISelGenericFloatingPointOpcode(Op))
    return true;

  // No. Check if we have a copy-like instruction. If we do, then we could
  // still be fed by floating point instructions.
  if (Op != TargetOpcode::COPY && !MI.isPHI() &&
      !isPreISelGenericOptimizationHint(Op))
    return false;

  // Check if we already know the register bank.
```
- **EN**: Implements helper routine(s) `getInvalidInstructionMapping`, `getInstructionMapping`, `isFPIntrinsic` for this portion of the PowerPC backend register bank mapping logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getInvalidInstructionMapping`, `getInstructionMapping`, `isFPIntrinsic`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 254-291

```cpp
  auto *RB = getRegBank(MI.getOperand(0).getReg(), MRI, TRI);
  if (RB == &PPC::FPRRegBank)
    return true;
  if (RB == &PPC::GPRRegBank)
    return false;

  // We don't know anything.
  //
  // If we have a phi, we may be able to infer that it will be assigned a FPR
  // based off of its inputs.
  if (!MI.isPHI() || Depth > MaxFPRSearchDepth)
    return false;

  return any_of(MI.explicit_uses(), [&](const MachineOperand &Op) {
    return Op.isReg() &&
           onlyDefinesFP(*MRI.getVRegDef(Op.getReg()), MRI, TRI, Depth + 1);
  });
}

/// FIXME: this is copied from target AArch64. Needs some code refactor here to
/// put this function in class RegisterBankInfo.
bool PPCRegisterBankInfo::onlyUsesFP(const MachineInstr &MI,
                                     const MachineRegisterInfo &MRI,
                                     const TargetRegisterInfo &TRI,
                                     unsigned Depth) const {
  switch (MI.getOpcode()) {
  case TargetOpcode::G_FPTOSI:
  case TargetOpcode::G_FPTOUI:
  case TargetOpcode::G_FCMP:
  case TargetOpcode::G_LROUND:
  case TargetOpcode::G_LLROUND:
    return true;
  default:
    break;
  }
  return hasFPConstraints(MI, MRI, TRI, Depth);
}
```
- **EN**: Implements helper routine(s) `getRegBank`, `getOperand`, `getReg` for this portion of the PowerPC backend register bank mapping logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `getRegBank`, `getOperand`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 292-312

```cpp
/// FIXME: this is copied from target AArch64. Needs some code refactor here to
/// put this function in class RegisterBankInfo.
bool PPCRegisterBankInfo::onlyDefinesFP(const MachineInstr &MI,
                                        const MachineRegisterInfo &MRI,
                                        const TargetRegisterInfo &TRI,
                                        unsigned Depth) const {
  switch (MI.getOpcode()) {
  case TargetOpcode::G_SITOFP:
  case TargetOpcode::G_UITOFP:
    return true;
  default:
    break;
  }
  return hasFPConstraints(MI, MRI, TRI, Depth);
}

RegisterBankInfo::InstructionMappings
PPCRegisterBankInfo::getInstrAlternativeMappings(const MachineInstr &MI) const {
  // TODO Implement.
  return RegisterBankInfo::getInstrAlternativeMappings(MI);
}
```
- **EN**: Implements helper routine(s) `onlyDefinesFP`, `getOpcode`, `hasFPConstraints` for this portion of the PowerPC backend register bank mapping logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器银行映射逻辑所需的辅助例程 `onlyDefinesFP`, `getOpcode`, `hasFPConstraints`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Register bank mapping logic / 寄存器银行映射逻辑
- GlobalISel pipeline / GlobalISel 流水线
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCRegisterBankInfo.h`
- `PPCRegisterInfo.h`
- `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`
- `llvm/CodeGen/GlobalISel/Utils.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `PPCGenRegisterBank.inc`
- `PPCGenRegisterBankInfo.def`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
