# MipsSEISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSEISelDAGToDAG.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsSEISelDAGToDAG` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MipsSEISelDAGToDAG`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsSEISelDAGToDAG.cpp - A Dag to Dag Inst Selector for MipsSE ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Subclass of MipsDAGToDAGISel specialized for mips32/64.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MipsSEISelDAGToDAG.h"
#include "Mips.h"
#include "MipsAnalyzeImmediate.h"
#include "MipsMachineFunction.h"
#include "MipsRegisterInfo.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/SelectionDAGNodes.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsMips.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-31
```cpp
#include "llvm/IR/Type.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetMachine.h"
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 33-33
```cpp
#define DEBUG_TYPE "mips-isel"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 35-40
```cpp
bool MipsSEDAGToDAGISel::runOnMachineFunction(MachineFunction &MF) {
  Subtarget = &MF.getSubtarget<MipsSubtarget>();
  if (Subtarget->inMips16Mode())
    return false;
  return MipsDAGToDAGISel::runOnMachineFunction(MF);
}
```
- EN: Implements `MipsSEDAGToDAGISel::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-45
```cpp
void MipsSEDAGToDAGISelLegacy::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<DominatorTreeWrapperPass>();
  SelectionDAGISelLegacy::getAnalysisUsage(AU);
}
```
- EN: Implements `MipsSEDAGToDAGISelLegacy::getAnalysisUsage`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISelLegacy::getAnalysisUsage`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-52
```cpp
void MipsSEDAGToDAGISel::addDSPCtrlRegOperands(bool IsDef, MachineInstr &MI,
                                               MachineFunction &MF) {
  MachineInstrBuilder MIB(MF, &MI);
  unsigned Mask = MI.getOperand(1).getImm();
  RegState Flag =
      IsDef ? RegState::ImplicitDefine : RegState::Implicit | RegState::Undef;
```
- EN: Implements `MipsSEDAGToDAGISel::addDSPCtrlRegOperands`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::addDSPCtrlRegOperands`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 54-55
```cpp
  if (Mask & 1)
    MIB.addReg(Mips::DSPPos, Flag);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 57-58
```cpp
  if (Mask & 2)
    MIB.addReg(Mips::DSPSCount, Flag);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 60-61
```cpp
  if (Mask & 4)
    MIB.addReg(Mips::DSPCarry, Flag);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 63-64
```cpp
  if (Mask & 8)
    MIB.addReg(Mips::DSPOutFlag, Flag);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 66-67
```cpp
  if (Mask & 16)
    MIB.addReg(Mips::DSPCCond, Flag);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 69-71
```cpp
  if (Mask & 32)
    MIB.addReg(Mips::DSPEFI, Flag);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 73-76
```cpp
MCRegister MipsSEDAGToDAGISel::getMSACtrlReg(const SDValue RegIdx) const {
  uint64_t RegNum = RegIdx->getAsZExtVal();
  return Mips::MSACtrlRegClass.getRegister(RegNum);
}
```
- EN: Implements `MipsSEDAGToDAGISel::getMSACtrlReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::getMSACtrlReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 78-80
```cpp
bool MipsSEDAGToDAGISel::replaceUsesWithZeroReg(MachineRegisterInfo *MRI,
                                                const MachineInstr& MI) {
  unsigned DstReg = 0, ZeroReg = 0;
```
- EN: Implements `MipsSEDAGToDAGISel::replaceUsesWithZeroReg`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::replaceUsesWithZeroReg`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-95
```cpp
  // Check if MI is "addiu $dst, $zero, 0" or "daddiu $dst, $zero, 0".
  if ((MI.getOpcode() == Mips::ADDiu) &&
      (MI.getOperand(1).getReg() == Mips::ZERO) &&
      (MI.getOperand(2).isImm()) &&
      (MI.getOperand(2).getImm() == 0)) {
    DstReg = MI.getOperand(0).getReg();
    ZeroReg = Mips::ZERO;
  } else if ((MI.getOpcode() == Mips::DADDiu) &&
             (MI.getOperand(1).getReg() == Mips::ZERO_64) &&
             (MI.getOperand(2).isImm()) &&
             (MI.getOperand(2).getImm() == 0)) {
    DstReg = MI.getOperand(0).getReg();
    ZeroReg = Mips::ZERO_64;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 97-98
```cpp
  if (!DstReg)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 100-106
```cpp
  // Replace uses with ZeroReg.
  for (MachineRegisterInfo::use_iterator U = MRI->use_begin(DstReg),
       E = MRI->use_end(); U != E;) {
    MachineOperand &MO = *U;
    unsigned OpNo = U.getOperandNo();
    MachineInstr *MI = MO.getParent();
    ++U;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 108-110
```cpp
    // Do not replace if it is a phi's operand or is tied to def operand.
    if (MI->isPHI() || MI->isRegTiedToDefOperand(OpNo) || MI->isPseudo())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 112-115
```cpp
    // Also, we have to check that the register class of the operand
    // contains the zero register.
    if (!MRI->getRegClass(MO.getReg())->contains(ZeroReg))
      continue;
```
- EN: Declares `of`, packaging target-specific state and APIs around `MipsSEISelDAGToDAG`.
- CN: 这里声明 `of`，把与 `MipsSEISelDAGToDAG` 相关的目标特定状态和 API 组织在一起。

### Lines 117-118
```cpp
    MO.setReg(ZeroReg);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 120-121
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-137
```cpp
void MipsSEDAGToDAGISel::emitMCountABI(MachineInstr &MI, MachineBasicBlock &MBB,
                                       MachineFunction &MF) {
  MachineInstrBuilder MIB(MF, &MI);
  if (!Subtarget->isABI_O32()) { // N32, N64
    // Save current return address.
    BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(Mips::OR64))
        .addDef(Mips::AT_64)
        .addUse(Mips::RA_64, RegState::Undef)
        .addUse(Mips::ZERO_64);
    // Stops instruction above from being removed later on.
    MIB.addUse(Mips::AT_64, RegState::Implicit);
  } else {  // O32
    // Save current return address.
    BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(Mips::OR))
        .addDef(Mips::AT)
```
- EN: Implements `MipsSEDAGToDAGISel::emitMCountABI`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::emitMCountABI`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 138-148
```cpp
        .addUse(Mips::RA, RegState::Undef)
        .addUse(Mips::ZERO);
    // _mcount pops 2 words from stack.
    BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(Mips::ADDiu))
        .addDef(Mips::SP)
        .addUse(Mips::SP)
        .addImm(-8);
    // Stops first instruction above from being removed later on.
    MIB.addUse(Mips::AT, RegState::Implicit);
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 150-151
```cpp
void MipsSEDAGToDAGISel::processFunctionAfterISel(MachineFunction &MF) {
  MF.getInfo<MipsFunctionInfo>()->initGlobalBaseReg(MF);
```
- EN: Implements `MipsSEDAGToDAGISel::processFunctionAfterISel`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::processFunctionAfterISel`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 153-153
```cpp
  MachineRegisterInfo *MRI = &MF.getRegInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 155-169
```cpp
  for (auto &MBB: MF) {
    for (auto &MI: MBB) {
      switch (MI.getOpcode()) {
      case Mips::RDDSP:
        addDSPCtrlRegOperands(false, MI, MF);
        break;
      case Mips::WRDSP:
        addDSPCtrlRegOperands(true, MI, MF);
        break;
      case Mips::BuildPairF64_64:
      case Mips::ExtractElementF64_64:
        if (!Subtarget->useOddSPReg()) {
          MI.addOperand(MachineOperand::CreateReg(Mips::SP, false, true));
          break;
        }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 170-184
```cpp
        [[fallthrough]];
      case Mips::BuildPairF64:
      case Mips::ExtractElementF64:
        if (Subtarget->isABI_FPXX() && !Subtarget->hasMTHC1())
          MI.addOperand(MachineOperand::CreateReg(Mips::SP, false, true));
        break;
      case Mips::JAL:
      case Mips::JAL_MM:
        if (MI.getOperand(0).isGlobal() &&
            MI.getOperand(0).getGlobal()->hasExternalLinkage() &&
            MI.getOperand(0).getGlobal()->getName() == "_mcount")
          emitMCountABI(MI, MBB, MF);
        break;
      case Mips::JALRPseudo:
      case Mips::JALR64Pseudo:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 185-199
```cpp
      case Mips::JALR16_MM:
        if (MI.getOperand(2).isMCSymbol() &&
            MI.getOperand(2).getMCSymbol()->getName() == "_mcount")
          emitMCountABI(MI, MBB, MF);
        break;
      case Mips::JALR:
        if (MI.getOperand(3).isMCSymbol() &&
            MI.getOperand(3).getMCSymbol()->getName() == "_mcount")
          emitMCountABI(MI, MBB, MF);
        break;
      default:
        replaceUsesWithZeroReg(MRI, MI);
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 200-200
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 202-206
```cpp
void MipsSEDAGToDAGISel::selectAddE(SDNode *Node, const SDLoc &DL) const {
  SDValue InGlue = Node->getOperand(2);
  unsigned Opc = InGlue.getOpcode();
  SDValue LHS = Node->getOperand(0), RHS = Node->getOperand(1);
  EVT VT = LHS.getValueType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddE`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddE`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 208-214
```cpp
  // In the base case, we can rely on the carry bit from the addsc
  // instruction.
  if (Opc == ISD::ADDC) {
    SDValue Ops[3] = {LHS, RHS, InGlue};
    CurDAG->SelectNodeTo(Node, Mips::ADDWC, VT, MVT::Glue, Ops);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 216-216
```cpp
  assert(Opc == ISD::ADDE && "ISD::ADDE not in a chain of ADDE nodes!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 218-224
```cpp
  // The more complex case is when there is a chain of ISD::ADDE nodes like:
  // (adde (adde (adde (addc a b) c) d) e).
  //
  // The addwc instruction does not write to the carry bit, instead it writes
  // to bit 20 of the dsp control register. To match this series of nodes, each
  // intermediate adde node must be expanded to write the carry bit before the
  // addition.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 226-228
```cpp
  // Start by reading the overflow field for addsc and moving the value to the
  // carry field. The usage of 1 here with MipsISD::RDDSP / Mips::WRDSP
  // corresponds to reading/writing the entire control register to/from a GPR.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 230-230
```cpp
  SDValue CstOne = CurDAG->getTargetConstant(1, DL, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-232
```cpp
  SDValue OuFlag = CurDAG->getTargetConstant(20, DL, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 234-235
```cpp
  SDNode *DSPCtrlField = CurDAG->getMachineNode(Mips::RDDSP, DL, MVT::i32,
                                                MVT::Glue, CstOne, InGlue);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 237-238
```cpp
  SDNode *Carry = CurDAG->getMachineNode(
      Mips::EXT, DL, MVT::i32, SDValue(DSPCtrlField, 0), OuFlag, CstOne);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 240-243
```cpp
  SDValue Ops[4] = {SDValue(DSPCtrlField, 0),
                    CurDAG->getTargetConstant(6, DL, MVT::i32), CstOne,
                    SDValue(Carry, 0)};
  SDNode *DSPCFWithCarry = CurDAG->getMachineNode(Mips::INS, DL, MVT::i32, Ops);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 245-248
```cpp
  // My reading of the MIPS DSP 3.01 specification isn't as clear as I
  // would like about whether bit 20 always gets overwritten by addwc.
  // Hence take an extremely conservative view and presume it's sticky. We
  // therefore need to clear it.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 250-250
```cpp
  SDValue Zero = CurDAG->getRegister(Mips::ZERO, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 252-254
```cpp
  SDValue InsOps[4] = {Zero, OuFlag, CstOne, SDValue(DSPCFWithCarry, 0)};
  SDNode *DSPCtrlFinal =
      CurDAG->getMachineNode(Mips::INS, DL, MVT::i32, InsOps);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 256-257
```cpp
  SDNode *WrDSP = CurDAG->getMachineNode(Mips::WRDSP, DL, MVT::Glue,
                                         SDValue(DSPCtrlFinal, 0), CstOne);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 259-261
```cpp
  SDValue Operands[3] = {LHS, RHS, SDValue(WrDSP, 0)};
  CurDAG->SelectNodeTo(Node, Mips::ADDWC, VT, MVT::Glue, Operands);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 263-267
```cpp
/// Match frameindex
bool MipsSEDAGToDAGISel::selectAddrFrameIndex(SDValue Addr, SDValue &Base,
                                              SDValue &Offset) const {
  if (FrameIndexSDNode *FIN = dyn_cast<FrameIndexSDNode>(Addr)) {
    EVT ValTy = Addr.getValueType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrFrameIndex`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrFrameIndex`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 269-274
```cpp
    Base   = CurDAG->getTargetFrameIndex(FIN->getIndex(), ValTy);
    Offset = CurDAG->getTargetConstant(0, SDLoc(Addr), ValTy);
    return true;
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 276-283
```cpp
/// Match frameindex+offset and frameindex|offset
bool MipsSEDAGToDAGISel::selectAddrFrameIndexOffset(
    SDValue Addr, SDValue &Base, SDValue &Offset, unsigned OffsetBits,
    unsigned ShiftAmount = 0) const {
  if (CurDAG->isBaseWithConstantOffset(Addr)) {
    auto *CN = cast<ConstantSDNode>(Addr.getOperand(1));
    if (isIntN(OffsetBits + ShiftAmount, CN->getSExtValue())) {
      EVT ValTy = Addr.getValueType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrFrameIndexOffset`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrFrameIndexOffset`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 285-296
```cpp
      // If the first operand is a FI, get the TargetFI Node
      if (FrameIndexSDNode *FIN =
              dyn_cast<FrameIndexSDNode>(Addr.getOperand(0)))
        Base = CurDAG->getTargetFrameIndex(FIN->getIndex(), ValTy);
      else {
        Base = Addr.getOperand(0);
        // If base is a FI, additional offset calculation is done in
        // eliminateFrameIndex, otherwise we need to check the alignment
        const Align Alignment(1ULL << ShiftAmount);
        if (!isAligned(Alignment, CN->getZExtValue()))
          return false;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 298-304
```cpp
      Offset = CurDAG->getTargetConstant(CN->getZExtValue(), SDLoc(Addr),
                                         ValTy);
      return true;
    }
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 306-312
```cpp
/// ComplexPattern used on MipsInstrInfo
/// Used on Mips Load/Store instructions
bool MipsSEDAGToDAGISel::selectAddrRegImm(SDValue Addr, SDValue &Base,
                                          SDValue &Offset) const {
  // if Address is FI, get the TargetFrameIndex.
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrRegImm`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrRegImm`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 314-319
```cpp
  // on PIC code Load GA
  if (Addr.getOpcode() == MipsISD::Wrapper) {
    Base   = Addr.getOperand(0);
    Offset = Addr.getOperand(1);
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 321-325
```cpp
  if (!TM.isPositionIndependent()) {
    if ((Addr.getOpcode() == ISD::TargetExternalSymbol ||
        Addr.getOpcode() == ISD::TargetGlobalAddress))
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 327-329
```cpp
  // Addresses of the form FI+const or FI|const
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 16))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 331-345
```cpp
  // Operand is a result from an ADD.
  if (Addr.getOpcode() == ISD::ADD) {
    // When loading from constant pools, load the lower address part in
    // the instruction itself. Example, instead of:
    //  lui $2, %hi($CPI1_0)
    //  addiu $2, $2, %lo($CPI1_0)
    //  lwc1 $f0, 0($2)
    // Generate:
    //  lui $2, %hi($CPI1_0)
    //  lwc1 $f0, %lo($CPI1_0)($2)
    if (Addr.getOperand(1).getOpcode() == MipsISD::Lo ||
        Addr.getOperand(1).getOpcode() == MipsISD::GPRel) {
      SDValue Opnd0 = Addr.getOperand(1).getOperand(0);
      if (isa<ConstantPoolSDNode>(Opnd0) || isa<GlobalAddressSDNode>(Opnd0) ||
          isa<JumpTableSDNode>(Opnd0)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 346-351
```cpp
        Base = Addr.getOperand(0);
        Offset = Opnd0;
        return true;
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 353-354
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 356-363
```cpp
/// ComplexPattern used on MipsInstrInfo
/// Used on Mips Load/Store instructions
bool MipsSEDAGToDAGISel::selectAddrDefault(SDValue Addr, SDValue &Base,
                                           SDValue &Offset) const {
  Base = Addr;
  Offset = CurDAG->getTargetConstant(0, SDLoc(Addr), Addr.getValueType());
  return true;
}
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrDefault`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrDefault`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 365-369
```cpp
bool MipsSEDAGToDAGISel::selectIntAddr(SDValue Addr, SDValue &Base,
                                       SDValue &Offset) const {
  return selectAddrRegImm(Addr, Base, Offset) ||
    selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddr`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddr`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 371-374
```cpp
bool MipsSEDAGToDAGISel::selectAddrRegImm9(SDValue Addr, SDValue &Base,
                                           SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrRegImm9`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrRegImm9`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 376-377
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 9))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 379-380
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 382-386
```cpp
/// Used on microMIPS LWC2, LDC2, SWC2 and SDC2 instructions (11-bit offset)
bool MipsSEDAGToDAGISel::selectAddrRegImm11(SDValue Addr, SDValue &Base,
                                            SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrRegImm11`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrRegImm11`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 388-389
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 11))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 391-392
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 394-398
```cpp
/// Used on microMIPS Load/Store unaligned instructions (12-bit offset)
bool MipsSEDAGToDAGISel::selectAddrRegImm12(SDValue Addr, SDValue &Base,
                                            SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrRegImm12`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrRegImm12`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 400-401
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 12))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 403-404
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-409
```cpp
bool MipsSEDAGToDAGISel::selectAddrRegImm16(SDValue Addr, SDValue &Base,
                                            SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectAddrRegImm16`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectAddrRegImm16`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 411-412
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 16))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 414-415
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 417-421
```cpp
bool MipsSEDAGToDAGISel::selectIntAddr11MM(SDValue Addr, SDValue &Base,
                                         SDValue &Offset) const {
  return selectAddrRegImm11(Addr, Base, Offset) ||
    selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddr11MM`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddr11MM`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 423-427
```cpp
bool MipsSEDAGToDAGISel::selectIntAddr12MM(SDValue Addr, SDValue &Base,
                                         SDValue &Offset) const {
  return selectAddrRegImm12(Addr, Base, Offset) ||
    selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddr12MM`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddr12MM`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 429-433
```cpp
bool MipsSEDAGToDAGISel::selectIntAddr16MM(SDValue Addr, SDValue &Base,
                                         SDValue &Offset) const {
  return selectAddrRegImm16(Addr, Base, Offset) ||
    selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddr16MM`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddr16MM`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 435-439
```cpp
bool MipsSEDAGToDAGISel::selectIntAddrLSL2MM(SDValue Addr, SDValue &Base,
                                             SDValue &Offset) const {
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 7)) {
    if (isa<FrameIndexSDNode>(Base))
      return false;
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddrLSL2MM`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddrLSL2MM`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 441-444
```cpp
    if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(Offset)) {
      unsigned CnstOff = CN->getZExtValue();
      return (CnstOff == (CnstOff & 0x3c));
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 446-447
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 449-452
```cpp
  // For all other cases where "lw" would be selected, don't select "lw16"
  // because it would result in additional instructions to prepare operands.
  if (selectAddrRegImm(Addr, Base, Offset))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 454-455
```cpp
  return selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 457-458
```cpp
bool MipsSEDAGToDAGISel::selectIntAddrSImm10(SDValue Addr, SDValue &Base,
                                             SDValue &Offset) const {
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddrSImm10`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddrSImm10`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 460-461
```cpp
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 463-464
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 10))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 466-467
```cpp
  return selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 469-472
```cpp
bool MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl1(SDValue Addr, SDValue &Base,
                                                 SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl1`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl1`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 474-475
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 10, 1))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 477-478
```cpp
  return selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 480-483
```cpp
bool MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl2(SDValue Addr, SDValue &Base,
                                                 SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl2`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl2`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 485-486
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 10, 2))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 488-489
```cpp
  return selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 491-494
```cpp
bool MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl3(SDValue Addr, SDValue &Base,
                                                 SDValue &Offset) const {
  if (selectAddrFrameIndex(Addr, Base, Offset))
    return true;
```
- EN: Implements `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl3`, a target-specific routine centered on stack-frame access. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectIntAddrSImm10Lsl3`，它是一个围绕栈帧访问展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 496-497
```cpp
  if (selectAddrFrameIndexOffset(Addr, Base, Offset, 10, 3))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 499-500
```cpp
  return selectAddrDefault(Addr, Base, Offset);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 502-510
```cpp
// Select constant vector splats.
//
// Returns true and sets Imm if:
// * MSA is enabled
// * N is a ISD::BUILD_VECTOR representing a constant splat
bool MipsSEDAGToDAGISel::selectVSplat(SDNode *N, APInt &Imm,
                                      unsigned MinSizeInBits) const {
  if (!Subtarget->hasMSA())
    return false;
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplat`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplat`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 512-512
```cpp
  BuildVectorSDNode *Node = dyn_cast<BuildVectorSDNode>(N);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 514-515
```cpp
  if (!Node)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 517-519
```cpp
  APInt SplatValue, SplatUndef;
  unsigned SplatBitSize;
  bool HasAnyUndefs;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 521-523
```cpp
  if (!Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs,
                             MinSizeInBits, !Subtarget->isLittle()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 525-525
```cpp
  Imm = SplatValue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 527-528
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 530-544
```cpp
// Select constant vector splats.
//
// In addition to the requirements of selectVSplat(), this function returns
// true and sets Imm if:
// * The splat value is the same width as the elements of the vector
// * The splat value fits in an integer with the specified signed-ness and
//   width.
//
// This function looks through ISD::BITCAST nodes.
// TODO: This might not be appropriate for big-endian MSA since BITCAST is
//       sometimes a shuffle in big-endian mode.
//
// It's worth noting that this function is not used as part of the selection
// of ldi.[bhwd] since it does not permit using the wrong-typed ldi.[bhwd]
// instruction to achieve the desired bit pattern. ldi.[bhwd] is selected in
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 545-550
```cpp
// MipsSEDAGToDAGISel::selectNode.
bool MipsSEDAGToDAGISel::
selectVSplatCommon(SDValue N, SDValue &Imm, bool Signed,
                   unsigned ImmBitSize) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `selectVSplatCommon`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `selectVSplatCommon`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 552-553
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 555-556
```cpp
  if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
      ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 558-563
```cpp
    if (( Signed && ImmValue.isSignedIntN(ImmBitSize)) ||
        (!Signed && ImmValue.isIntN(ImmBitSize))) {
      Imm = CurDAG->getTargetConstant(ImmValue, SDLoc(N), EltTy);
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 565-566
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 568-580
```cpp
// Select constant vector splats whose value is a power of 2.
//
// In addition to the requirements of selectVSplat(), this function returns
// true and sets Imm if:
// * The splat value is the same width as the elements of the vector
// * The splat value is a power of two.
//
// This function looks through ISD::BITCAST nodes.
// TODO: This might not be appropriate for big-endian MSA since BITCAST is
//       sometimes a shuffle in big-endian mode.
bool MipsSEDAGToDAGISel::selectVSplatUimmPow2(SDValue N, SDValue &Imm) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplatUimmPow2`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplatUimmPow2`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 582-583
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 585-587
```cpp
  if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
      ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
    int32_t Log2 = ImmValue.exactLogBase2();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 589-593
```cpp
    if (Log2 != -1) {
      Imm = CurDAG->getTargetConstant(Log2, SDLoc(N), EltTy);
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 595-596
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 598-611
```cpp
// Select constant vector splats whose value only has a consecutive sequence
// of left-most bits set (e.g. 0b11...1100...00).
//
// In addition to the requirements of selectVSplat(), this function returns
// true and sets Imm if:
// * The splat value is the same width as the elements of the vector
// * The splat value is a consecutive sequence of left-most bits.
//
// This function looks through ISD::BITCAST nodes.
// TODO: This might not be appropriate for big-endian MSA since BITCAST is
//       sometimes a shuffle in big-endian mode.
bool MipsSEDAGToDAGISel::selectVSplatMaskL(SDValue N, SDValue &Imm) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplatMaskL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplatMaskL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 613-614
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 616-624
```cpp
  if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
      ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
    // Check if we have a leading one, then check if the whole value is a
    // shifted mask.
    if (ImmValue.isNegative() && ImmValue.isShiftedMask()) {
      Imm = CurDAG->getTargetConstant(ImmValue.popcount() - 1, SDLoc(N), EltTy);
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 626-627
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 629-642
```cpp
// Select constant vector splats whose value only has a consecutive sequence
// of right-most bits set (e.g. 0b00...0011...11).
//
// In addition to the requirements of selectVSplat(), this function returns
// true and sets Imm if:
// * The splat value is the same width as the elements of the vector
// * The splat value is a consecutive sequence of right-most bits.
//
// This function looks through ISD::BITCAST nodes.
// TODO: This might not be appropriate for big-endian MSA since BITCAST is
//       sometimes a shuffle in big-endian mode.
bool MipsSEDAGToDAGISel::selectVSplatMaskR(SDValue N, SDValue &Imm) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplatMaskR`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplatMaskR`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 644-645
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 647-653
```cpp
  if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
      ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
    if (ImmValue.isMask()) {
      Imm = CurDAG->getTargetConstant(ImmValue.popcount() - 1, SDLoc(N), EltTy);
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 655-656
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 658-661
```cpp
bool MipsSEDAGToDAGISel::selectVSplatUimmInvPow2(SDValue N,
                                                 SDValue &Imm) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplatUimmInvPow2`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplatUimmInvPow2`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 663-664
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 666-668
```cpp
  if (selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
      ImmValue.getBitWidth() == EltTy.getSizeInBits()) {
    int32_t Log2 = (~ImmValue).exactLogBase2();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 670-674
```cpp
    if (Log2 != -1) {
      Imm = CurDAG->getTargetConstant(Log2, SDLoc(N), EltTy);
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 676-677
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 679-682
```cpp
// Select const vector splat of 1.
bool MipsSEDAGToDAGISel::selectVSplatImmEq1(SDValue N) const {
  APInt ImmValue;
  EVT EltTy = N->getValueType(0).getVectorElementType();
```
- EN: Implements `MipsSEDAGToDAGISel::selectVSplatImmEq1`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::selectVSplatImmEq1`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 684-685
```cpp
  if (N->getOpcode() == ISD::BITCAST)
    N = N->getOperand(0);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 687-689
```cpp
  return selectVSplat(N.getNode(), ImmValue, EltTy.getSizeInBits()) &&
         ImmValue.getBitWidth() == EltTy.getSizeInBits() && ImmValue == 1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 691-693
```cpp
bool MipsSEDAGToDAGISel::trySelect(SDNode *Node) {
  unsigned Opcode = Node->getOpcode();
  SDLoc DL(Node);
```
- EN: Implements `MipsSEDAGToDAGISel::trySelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::trySelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 695-700
```cpp
  ///
  // Instruction Selection not handled by the auto-generated
  // tablegen selection should be handled here.
  ///
  switch(Opcode) {
  default: break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 702-709
```cpp
  case MipsISD::DOUBLE_SELECT_I:
  case MipsISD::DOUBLE_SELECT_I64: {
    MVT VT = Subtarget->isGP64bit() ? MVT::i64 : MVT::i32;
    SDValue cond = Node->getOperand(0);
    SDValue Hi1 = Node->getOperand(1);
    SDValue Lo1 = Node->getOperand(2);
    SDValue Hi2 = Node->getOperand(3);
    SDValue Lo2 = Node->getOperand(4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 711-718
```cpp
    SDValue ops[] = {cond, Hi1, Lo1, Hi2, Lo2};
    EVT NodeTys[] = {VT, VT};
    ReplaceNode(Node, CurDAG->getMachineNode(Subtarget->isGP64bit()
                                                 ? Mips::PseudoD_SELECT_I64
                                                 : Mips::PseudoD_SELECT_I,
                                             DL, NodeTys, ops));
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 720-723
```cpp
  case ISD::ADDE: {
    selectAddE(Node, DL);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 725-739
```cpp
  case ISD::ConstantFP: {
    auto *CN = cast<ConstantFPSDNode>(Node);
    if (Node->getValueType(0) == MVT::f64 && CN->isExactlyValue(+0.0)) {
      if (Subtarget->isGP64bit()) {
        SDValue Zero = CurDAG->getCopyFromReg(CurDAG->getEntryNode(), DL,
                                              Mips::ZERO_64, MVT::i64);
        ReplaceNode(Node,
                    CurDAG->getMachineNode(Mips::DMTC1, DL, MVT::f64, Zero));
      } else if (Subtarget->isFP64bit()) {
        SDValue Zero = CurDAG->getCopyFromReg(CurDAG->getEntryNode(), DL,
                                              Mips::ZERO, MVT::i32);
        ReplaceNode(Node, CurDAG->getMachineNode(Mips::BuildPairF64_64, DL,
                                                 MVT::f64, Zero, Zero));
      } else {
        SDValue Zero = CurDAG->getCopyFromReg(CurDAG->getEntryNode(), DL,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 740-747
```cpp
                                              Mips::ZERO, MVT::i32);
        ReplaceNode(Node, CurDAG->getMachineNode(Mips::BuildPairF64, DL,
                                                 MVT::f64, Zero, Zero));
      }
      return true;
    }
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 749-752
```cpp
  case ISD::Constant: {
    auto *CN = cast<ConstantSDNode>(Node);
    int64_t Imm = CN->getSExtValue();
    unsigned Size = CN->getValueSizeInBits(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 754-755
```cpp
    if (isInt<32>(Imm))
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 757-757
```cpp
    MipsAnalyzeImmediate AnalyzeImm;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 759-760
```cpp
    const MipsAnalyzeImmediate::InstSeq &Seq =
      AnalyzeImm.Analyze(Imm, Size, false);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 762-766
```cpp
    MipsAnalyzeImmediate::InstSeq::const_iterator Inst = Seq.begin();
    SDLoc DL(CN);
    SDNode *RegOpnd;
    SDValue ImmOpnd = CurDAG->getTargetConstant(SignExtend64<16>(Inst->ImmOpnd),
                                                DL, MVT::i64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 768-777
```cpp
    // The first instruction can be a LUi which is different from other
    // instructions (ADDiu, ORI and SLL) in that it does not have a register
    // operand.
    if (Inst->Opc == Mips::LUi64)
      RegOpnd = CurDAG->getMachineNode(Inst->Opc, DL, MVT::i64, ImmOpnd);
    else
      RegOpnd =
        CurDAG->getMachineNode(Inst->Opc, DL, MVT::i64,
                               CurDAG->getRegister(Mips::ZERO_64, MVT::i64),
                               ImmOpnd);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 779-785
```cpp
    // The remaining instructions in the sequence are handled here.
    for (++Inst; Inst != Seq.end(); ++Inst) {
      ImmOpnd = CurDAG->getTargetConstant(SignExtend64<16>(Inst->ImmOpnd), DL,
                                          MVT::i64);
      RegOpnd = CurDAG->getMachineNode(Inst->Opc, DL, MVT::i64,
                                       SDValue(RegOpnd, 0), ImmOpnd);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 787-789
```cpp
    ReplaceNode(Node, RegOpnd);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 791-795
```cpp
  case ISD::INTRINSIC_W_CHAIN: {
    const unsigned IntrinsicOpcode = Node->getConstantOperandVal(1);
    switch (IntrinsicOpcode) {
    default:
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 797-808
```cpp
    case Intrinsic::mips_cfcmsa: {
      SDValue ChainIn = Node->getOperand(0);
      SDValue RegIdx = Node->getOperand(2);
      SDValue Reg = CurDAG->getCopyFromReg(ChainIn, DL,
                                           getMSACtrlReg(RegIdx), MVT::i32);
      ReplaceNode(Node, Reg.getNode());
      return true;
    }
    case Intrinsic::mips_ldr_d:
    case Intrinsic::mips_ldr_w: {
      unsigned Op = (IntrinsicOpcode == Intrinsic::mips_ldr_d) ? Mips::LDR_D
                                                               : Mips::LDR_W;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 810-815
```cpp
      SDLoc DL(Node);
      assert(Node->getNumOperands() == 4 && "Unexpected number of operands.");
      const SDValue &Chain = Node->getOperand(0);
      const SDValue &Intrinsic = Node->getOperand(1);
      const SDValue &Pointer = Node->getOperand(2);
      const SDValue &Constant = Node->getOperand(3);
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 817-821
```cpp
      assert(Chain.getValueType() == MVT::Other);
      (void)Intrinsic;
      assert(Intrinsic.getOpcode() == ISD::TargetConstant &&
             Constant.getOpcode() == ISD::Constant &&
             "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 823-827
```cpp
      // Convert Constant to TargetConstant.
      const ConstantInt *Val =
          cast<ConstantSDNode>(Constant)->getConstantIntValue();
      SDValue Imm =
          CurDAG->getTargetConstant(*Val, DL, Constant.getValueType());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 829-829
```cpp
      SmallVector<SDValue, 3> Ops{Pointer, Imm, Chain};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 831-834
```cpp
      assert(Node->getNumValues() == 2);
      assert(Node->getValueType(0).is128BitVector());
      assert(Node->getValueType(1) == MVT::Other);
      SmallVector<EVT, 2> ResTys{Node->getValueType(0), Node->getValueType(1)};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 836-836
```cpp
      ReplaceNode(Node, CurDAG->getMachineNode(Op, DL, ResTys, Ops));
```
- EN: Declares `ReplaceNode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReplaceNode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 838-842
```cpp
      return true;
    }
    }
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 844-847
```cpp
  case ISD::INTRINSIC_WO_CHAIN: {
    switch (Node->getConstantOperandVal(0)) {
    default:
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 849-858
```cpp
    case Intrinsic::mips_move_v:
      // Like an assignment but will always produce a move.v even if
      // unnecessary.
      ReplaceNode(Node, CurDAG->getMachineNode(Mips::MOVE_V, DL,
                                               Node->getValueType(0),
                                               Node->getOperand(1)));
      return true;
    }
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 860-864
```cpp
  case ISD::INTRINSIC_VOID: {
    const unsigned IntrinsicOpcode = Node->getConstantOperandVal(1);
    switch (IntrinsicOpcode) {
    default:
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 866-878
```cpp
    case Intrinsic::mips_ctcmsa: {
      SDValue ChainIn = Node->getOperand(0);
      SDValue RegIdx  = Node->getOperand(2);
      SDValue Value   = Node->getOperand(3);
      SDValue ChainOut = CurDAG->getCopyToReg(ChainIn, DL,
                                              getMSACtrlReg(RegIdx), Value);
      ReplaceNode(Node, ChainOut.getNode());
      return true;
    }
    case Intrinsic::mips_str_d:
    case Intrinsic::mips_str_w: {
      unsigned Op = (IntrinsicOpcode == Intrinsic::mips_str_d) ? Mips::STR_D
                                                               : Mips::STR_W;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 880-886
```cpp
      SDLoc DL(Node);
      assert(Node->getNumOperands() == 5 && "Unexpected number of operands.");
      const SDValue &Chain = Node->getOperand(0);
      const SDValue &Intrinsic = Node->getOperand(1);
      const SDValue &Vec = Node->getOperand(2);
      const SDValue &Pointer = Node->getOperand(3);
      const SDValue &Constant = Node->getOperand(4);
```
- EN: Declares `DL`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DL`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 888-892
```cpp
      assert(Chain.getValueType() == MVT::Other);
      (void)Intrinsic;
      assert(Intrinsic.getOpcode() == ISD::TargetConstant &&
             Constant.getOpcode() == ISD::Constant &&
             "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 894-898
```cpp
      // Convert Constant to TargetConstant.
      const ConstantInt *Val =
          cast<ConstantSDNode>(Constant)->getConstantIntValue();
      SDValue Imm =
          CurDAG->getTargetConstant(*Val, DL, Constant.getValueType());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 900-900
```cpp
      SmallVector<SDValue, 4> Ops{Vec, Pointer, Imm, Chain};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 902-904
```cpp
      assert(Node->getNumValues() == 1);
      assert(Node->getValueType(0) == MVT::Other);
      SmallVector<EVT, 1> ResTys{Node->getValueType(0)};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 906-911
```cpp
      ReplaceNode(Node, CurDAG->getMachineNode(Op, DL, ResTys, Ops));
      return true;
    }
    }
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 913-921
```cpp
  case MipsISD::FAbs: {
    MVT ResTy = Node->getSimpleValueType(0);
    assert((ResTy == MVT::f64 || ResTy == MVT::f32) &&
           "Unsupported float type!");
    unsigned Opc = 0;
    if (ResTy == MVT::f64)
      Opc = (Subtarget->isFP64bit() ? Mips::FABS_D64 : Mips::FABS_D32);
    else
      Opc = Mips::FABS_S;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 923-937
```cpp
    if (Subtarget->inMicroMipsMode()) {
      switch (Opc) {
      case Mips::FABS_D64:
        Opc = Mips::FABS_D64_MM;
        break;
      case Mips::FABS_D32:
        Opc = Mips::FABS_D32_MM;
        break;
      case Mips::FABS_S:
        Opc = Mips::FABS_S_MM;
        break;
      default:
        llvm_unreachable("Unknown opcode for MIPS floating point abs!");
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 939-940
```cpp
    ReplaceNode(Node,
                CurDAG->getMachineNode(Opc, DL, ResTy, Node->getOperand(0)));
```
- EN: Declares `ReplaceNode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ReplaceNode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 942-943
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 945-951
```cpp
  // Manually match MipsISD::Ins nodes to get the correct instruction. It has
  // to be done in this fashion so that we respect the differences between
  // dins and dinsm, as the difference is that the size operand has the range
  // 0 < size <= 32 for dins while dinsm has the range 2 <= size <= 64 which
  // means SelectionDAGISel would have to test all the operands at once to
  // match the instruction.
  case MipsISD::Ins: {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 953-955
```cpp
    // Validating the node operands.
    if (Node->getValueType(0) != MVT::i32 && Node->getValueType(0) != MVT::i64)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 957-958
```cpp
    if (Node->getNumOperands() != 4)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 960-962
```cpp
    if (Node->getOperand(1)->getOpcode() != ISD::Constant ||
        Node->getOperand(2)->getOpcode() != ISD::Constant)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 964-966
```cpp
    MVT ResTy = Node->getSimpleValueType(0);
    uint64_t Pos = Node->getConstantOperandVal(1);
    uint64_t Size = Node->getConstantOperandVal(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 968-970
```cpp
    // Size has to be >0 for 'ins', 'dins' and 'dinsu'.
    if (!Size)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 972-973
```cpp
    if (Pos + Size > 64)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 975-976
```cpp
    if (ResTy != MVT::i32 && ResTy != MVT::i64)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 978-989
```cpp
    unsigned Opcode = 0;
    if (ResTy == MVT::i32) {
      if (Pos + Size <= 32)
        Opcode = Mips::INS;
    } else {
      if (Pos + Size <= 32)
        Opcode = Mips::DINS;
      else if (Pos < 32 && 1 < Size)
        Opcode = Mips::DINSM;
      else
        Opcode = Mips::DINSU;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 991-994
```cpp
    if (Opcode) {
      SDValue Ops[4] = {
          Node->getOperand(0), CurDAG->getTargetConstant(Pos, DL, MVT::i32),
          CurDAG->getTargetConstant(Size, DL, MVT::i32), Node->getOperand(3)};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 996-998
```cpp
      ReplaceNode(Node, CurDAG->getMachineNode(Opcode, DL, ResTy, Ops));
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1000-1001
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1003-1005
```cpp
  case MipsISD::ThreadPointer: {
    EVT PtrVT = getTargetLowering()->getPointerTy(CurDAG->getDataLayout());
    unsigned RdhwrOpc, DestReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1007-1013
```cpp
    if (PtrVT == MVT::i32) {
      RdhwrOpc = Mips::RDHWR;
      DestReg = Mips::V1;
    } else {
      RdhwrOpc = Mips::RDHWR64;
      DestReg = Mips::V1_64;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1015-1025
```cpp
    SDNode *Rdhwr =
        CurDAG->getMachineNode(RdhwrOpc, DL, Node->getValueType(0), MVT::Glue,
                               CurDAG->getRegister(Mips::HWR29, MVT::i32),
                               CurDAG->getTargetConstant(0, DL, MVT::i32));
    SDValue Chain = CurDAG->getCopyToReg(CurDAG->getEntryNode(), DL, DestReg,
                                         SDValue(Rdhwr, 0), SDValue(Rdhwr, 1));
    SDValue ResNode = CurDAG->getCopyFromReg(Chain, DL, DestReg, PtrVT,
                                             Chain.getValue(1));
    ReplaceNode(Node, ResNode.getNode());
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1027-1038
```cpp
  case ISD::BUILD_VECTOR: {
    // Select appropriate ldi.[bhwd] instructions for constant splats of
    // 128-bit when MSA is enabled. Fixup any register class mismatches that
    // occur as a result.
    //
    // This allows the compiler to use a wider range of immediates than would
    // otherwise be allowed. If, for example, v4i32 could only use ldi.h then
    // it would not be possible to load { 0x01010101, 0x01010101, 0x01010101,
    // 0x01010101 } without using a constant pool. This would be sub-optimal
    // when // 'ldi.b wd, 1' is capable of producing that bit-pattern in the
    // same set/ of registers. Similarly, ldi.h isn't capable of producing {
    // 0x00000000, 0x00000001, 0x00000000, 0x00000001 } but 'ldi.d wd, 1' can.
```
- EN: Declares `mismatches`, packaging target-specific state and APIs around `MipsSEISelDAGToDAG`.
- CN: 这里声明 `mismatches`，把与 `MipsSEISelDAGToDAG` 相关的目标特定状态和 API 组织在一起。

### Lines 1040-1041
```cpp
    const MipsABIInfo &ABI =
        static_cast<const MipsTargetMachine &>(TM).getABI();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1043-1049
```cpp
    BuildVectorSDNode *BVN = cast<BuildVectorSDNode>(Node);
    APInt SplatValue, SplatUndef;
    unsigned SplatBitSize;
    bool HasAnyUndefs;
    unsigned LdiOp;
    EVT ResVecTy = BVN->getValueType(0);
    EVT ViaVecTy;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1051-1052
```cpp
    if (!Subtarget->hasMSA() || !BVN->getValueType(0).is128BitVector())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1054-1057
```cpp
    if (!BVN->isConstantSplat(SplatValue, SplatUndef, SplatBitSize,
                              HasAnyUndefs, 8,
                              !Subtarget->isLittle()))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1059-1073
```cpp
    switch (SplatBitSize) {
    default:
      return false;
    case 8:
      LdiOp = Mips::LDI_B;
      ViaVecTy = MVT::v16i8;
      break;
    case 16:
      LdiOp = Mips::LDI_H;
      ViaVecTy = MVT::v8i16;
      break;
    case 32:
      LdiOp = Mips::LDI_W;
      ViaVecTy = MVT::v4i32;
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1074-1078
```cpp
    case 64:
      LdiOp = Mips::LDI_D;
      ViaVecTy = MVT::v2i64;
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1080-1080
```cpp
    SDNode *Res = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1082-1088
```cpp
    // If we have a signed 10 bit integer, we can splat it directly.
    //
    // If we have something bigger we can synthesize the value into a GPR and
    // splat from there.
    if (SplatValue.isSignedIntN(10)) {
      SDValue Imm = CurDAG->getTargetConstant(SplatValue, DL,
                                              ViaVecTy.getVectorElementType());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1090-1097
```cpp
      Res = CurDAG->getMachineNode(LdiOp, DL, ViaVecTy, Imm);
    } else if (SplatValue.isSignedIntN(16) &&
               ((ABI.IsO32() && SplatBitSize < 64) ||
                (ABI.IsN32() || ABI.IsN64()))) {
      // Only handle signed 16 bit values when the element size is GPR width.
      // MIPS64 can handle all the cases but MIPS32 would need to handle
      // negative cases specifically here. Instead, handle those cases as
      // 64bit values.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1099-1103
```cpp
      bool Is32BitSplat = ABI.IsO32() || SplatBitSize < 64;
      const unsigned ADDiuOp = Is32BitSplat ? Mips::ADDiu : Mips::DADDiu;
      const MVT SplatMVT = Is32BitSplat ? MVT::i32 : MVT::i64;
      SDValue ZeroVal = CurDAG->getRegister(
          Is32BitSplat ? Mips::ZERO : Mips::ZERO_64, SplatMVT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1105-1109
```cpp
      const unsigned FILLOp =
          SplatBitSize == 16
              ? Mips::FILL_H
              : (SplatBitSize == 32 ? Mips::FILL_W
                                    : (SplatBitSize == 64 ? Mips::FILL_D : 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1111-1113
```cpp
      assert(FILLOp != 0 && "Unknown FILL Op for splat synthesis!");
      assert((!ABI.IsO32() || (FILLOp != Mips::FILL_D)) &&
             "Attempting to use fill.d on MIPS32!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1115-1116
```cpp
      const unsigned Lo = SplatValue.getLoBits(16).getZExtValue();
      SDValue LoVal = CurDAG->getTargetConstant(Lo, DL, SplatMVT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1118-1119
```cpp
      Res = CurDAG->getMachineNode(ADDiuOp, DL, SplatMVT, ZeroVal, LoVal);
      Res = CurDAG->getMachineNode(FILLOp, DL, ViaVecTy, SDValue(Res, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1121-1126
```cpp
    } else if (SplatValue.isSignedIntN(32) && SplatBitSize == 32) {
      // Only handle the cases where the splat size agrees with the size
      // of the SplatValue here.
      const unsigned Lo = SplatValue.getLoBits(16).getZExtValue();
      const unsigned Hi = SplatValue.lshr(16).getLoBits(16).getZExtValue();
      SDValue ZeroVal = CurDAG->getRegister(Mips::ZERO, MVT::i32);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1128-1129
```cpp
      SDValue LoVal = CurDAG->getTargetConstant(Lo, DL, MVT::i32);
      SDValue HiVal = CurDAG->getTargetConstant(Hi, DL, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1131-1132
```cpp
      if (Hi)
        Res = CurDAG->getMachineNode(Mips::LUi, DL, MVT::i32, HiVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1134-1136
```cpp
      if (Lo)
        Res = CurDAG->getMachineNode(Mips::ORi, DL, MVT::i32,
                                     Hi ? SDValue(Res, 0) : ZeroVal, LoVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1138-1140
```cpp
      assert((Hi || Lo) && "Zero case reached 32 bit case splat synthesis!");
      Res =
          CurDAG->getMachineNode(Mips::FILL_W, DL, MVT::v4i32, SDValue(Res, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1142-1149
```cpp
    } else if (SplatValue.isSignedIntN(32) && SplatBitSize == 64 &&
               (ABI.IsN32() || ABI.IsN64())) {
      // N32 and N64 can perform some tricks that O32 can't for signed 32 bit
      // integers due to having 64bit registers. lui will cause the necessary
      // zero/sign extension.
      const unsigned Lo = SplatValue.getLoBits(16).getZExtValue();
      const unsigned Hi = SplatValue.lshr(16).getLoBits(16).getZExtValue();
      SDValue ZeroVal = CurDAG->getRegister(Mips::ZERO, MVT::i32);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1151-1152
```cpp
      SDValue LoVal = CurDAG->getTargetConstant(Lo, DL, MVT::i32);
      SDValue HiVal = CurDAG->getTargetConstant(Hi, DL, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1154-1155
```cpp
      if (Hi)
        Res = CurDAG->getMachineNode(Mips::LUi, DL, MVT::i32, HiVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1157-1159
```cpp
      if (Lo)
        Res = CurDAG->getMachineNode(Mips::ORi, DL, MVT::i32,
                                     Hi ? SDValue(Res, 0) : ZeroVal, LoVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1161-1163
```cpp
      Res = CurDAG->getMachineNode(
          Mips::SUBREG_TO_REG, DL, MVT::i64, SDValue(Res, 0),
          CurDAG->getTargetConstant(Mips::sub_32, DL, MVT::i64));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1165-1166
```cpp
      Res =
          CurDAG->getMachineNode(Mips::FILL_D, DL, MVT::v2i64, SDValue(Res, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1168-1182
```cpp
    } else if (SplatValue.isSignedIntN(64)) {
      // If we have a 64 bit Splat value, we perform a similar sequence to the
      // above:
      //
      // MIPS32:                            MIPS64:
      //   lui $res, %highest(val)            lui $res, %highest(val)
      //   ori $res, $res, %higher(val)       ori $res, $res, %higher(val)
      //   lui $res2, %hi(val)                lui $res2, %hi(val)
      //   ori $res2, %res2, %lo(val)         ori $res2, %res2, %lo(val)
      //   $res3 = fill $res2                 dinsu $res, $res2, 0, 32
      //   $res4 = insert.w $res3[1], $res    fill.d $res
      //   splat.d $res4, 0
      //
      // The ability to use dinsu is guaranteed as MSA requires MIPSR5.
      // This saves having to materialize the value by shifts and ors.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1183-1192
```cpp
      //
      // FIXME: Implement the preferred sequence for MIPS64R6:
      //
      // MIPS64R6:
      //   ori $res, $zero, %lo(val)
      //   daui $res, $res, %hi(val)
      //   dahi $res, $res, %higher(val)
      //   dati $res, $res, %highest(cal)
      //   fill.d $res
      //
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1194-1197
```cpp
      const unsigned Lo = SplatValue.getLoBits(16).getZExtValue();
      const unsigned Hi = SplatValue.lshr(16).getLoBits(16).getZExtValue();
      const unsigned Higher = SplatValue.lshr(32).getLoBits(16).getZExtValue();
      const unsigned Highest = SplatValue.lshr(48).getLoBits(16).getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1199-1203
```cpp
      SDValue LoVal = CurDAG->getTargetConstant(Lo, DL, MVT::i32);
      SDValue HiVal = CurDAG->getTargetConstant(Hi, DL, MVT::i32);
      SDValue HigherVal = CurDAG->getTargetConstant(Higher, DL, MVT::i32);
      SDValue HighestVal = CurDAG->getTargetConstant(Highest, DL, MVT::i32);
      SDValue ZeroVal = CurDAG->getRegister(Mips::ZERO, MVT::i32);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1205-1219
```cpp
      // Independent of whether we're targeting MIPS64 or not, the basic
      // operations are the same. Also, directly use the $zero register if
      // the 16 bit chunk is zero.
      //
      // For optimization purposes we always synthesize the splat value as
      // an i32 value, then if we're targetting MIPS64, use SUBREG_TO_REG
      // just before combining the values with dinsu to produce an i64. This
      // enables SelectionDAG to aggressively share components of splat values
      // where possible.
      //
      // FIXME: This is the general constant synthesis problem. This code
      //        should be factored out into a class shared between all the
      //        classes that need it. Specifically, for a splat size of 64
      //        bits that's a negative number we can do better than LUi/ORi
      //        for the upper 32bits.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1221-1222
```cpp
      if (Hi)
        Res = CurDAG->getMachineNode(Mips::LUi, DL, MVT::i32, HiVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1224-1226
```cpp
      if (Lo)
        Res = CurDAG->getMachineNode(Mips::ORi, DL, MVT::i32,
                                     Hi ? SDValue(Res, 0) : ZeroVal, LoVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1228-1230
```cpp
      SDNode *HiRes;
      if (Highest)
        HiRes = CurDAG->getMachineNode(Mips::LUi, DL, MVT::i32, HighestVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1232-1235
```cpp
      if (Higher)
        HiRes = CurDAG->getMachineNode(Mips::ORi, DL, MVT::i32,
                                       Highest ? SDValue(HiRes, 0) : ZeroVal,
                                       HigherVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1238-1240
```cpp
      if (ABI.IsO32()) {
        Res = CurDAG->getMachineNode(Mips::FILL_W, DL, MVT::v4i32,
                                     (Hi || Lo) ? SDValue(Res, 0) : ZeroVal);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1242-1245
```cpp
        Res = CurDAG->getMachineNode(
            Mips::INSERT_W, DL, MVT::v4i32, SDValue(Res, 0),
            (Highest || Higher) ? SDValue(HiRes, 0) : ZeroVal,
            CurDAG->getTargetConstant(1, DL, MVT::i32));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1247-1249
```cpp
        const TargetLowering *TLI = getTargetLowering();
        const TargetRegisterClass *RC =
            TLI->getRegClassFor(ViaVecTy.getSimpleVT());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1251-1253
```cpp
        Res = CurDAG->getMachineNode(
            Mips::COPY_TO_REGCLASS, DL, ViaVecTy, SDValue(Res, 0),
            CurDAG->getTargetConstant(RC->getID(), DL, MVT::i32));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1255-1258
```cpp
        Res = CurDAG->getMachineNode(
            Mips::SPLATI_D, DL, MVT::v2i64, SDValue(Res, 0),
            CurDAG->getTargetConstant(0, DL, MVT::i32));
      } else if (ABI.IsN64() || ABI.IsN32()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1260-1262
```cpp
        SDValue Zero64Val = CurDAG->getRegister(Mips::ZERO_64, MVT::i64);
        const bool HiResNonZero = Highest || Higher;
        const bool ResNonZero = Hi || Lo;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1264-1267
```cpp
        if (HiResNonZero)
          HiRes = CurDAG->getMachineNode(
              Mips::SUBREG_TO_REG, DL, MVT::i64, SDValue(HiRes, 0),
              CurDAG->getTargetConstant(Mips::sub_32, DL, MVT::i64));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1269-1272
```cpp
        if (ResNonZero)
          Res = CurDAG->getMachineNode(
              Mips::SUBREG_TO_REG, DL, MVT::i64, SDValue(Res, 0),
              CurDAG->getTargetConstant(Mips::sub_32, DL, MVT::i64));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1274-1288
```cpp
        // We have 3 cases:
        //   The HiRes is nonzero but Res is $zero  => dsll32 HiRes, 0
        //   The Res is nonzero but HiRes is $zero  => dinsu Res, $zero, 32, 32
        //   Both are non zero                      => dinsu Res, HiRes, 32, 32
        //
        // The obvious "missing" case is when both are zero, but that case is
        // handled by the ldi case.
        if (ResNonZero) {
          IntegerType *Int32Ty =
              IntegerType::get(MF->getFunction().getContext(), 32);
          const ConstantInt *Const32 = ConstantInt::get(Int32Ty, 32);
          SDValue Ops[4] = {HiResNonZero ? SDValue(HiRes, 0) : Zero64Val,
                            CurDAG->getConstant(*Const32, DL, MVT::i32),
                            CurDAG->getConstant(*Const32, DL, MVT::i32),
                            SDValue(Res, 0)};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1290-1297
```cpp
          Res = CurDAG->getMachineNode(Mips::DINSU, DL, MVT::i64, Ops);
        } else if (HiResNonZero) {
          Res = CurDAG->getMachineNode(
              Mips::DSLL32, DL, MVT::i64, SDValue(HiRes, 0),
              CurDAG->getTargetConstant(0, DL, MVT::i32));
        } else
          llvm_unreachable(
              "Zero splat value handled by non-zero 64bit splat synthesis!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1299-1302
```cpp
        Res = CurDAG->getMachineNode(Mips::FILL_D, DL, MVT::v2i64,
                                     SDValue(Res, 0));
      } else
        llvm_unreachable("Unknown ABI in MipsISelDAGToDAG!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1304-1305
```cpp
    } else
      return false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1307-1319
```cpp
    if (ResVecTy != ViaVecTy) {
      // If LdiOp is writing to a different register class to ResVecTy, then
      // fix it up here. This COPY_TO_REGCLASS should never cause a move.v
      // since the source and destination register sets contain the same
      // registers.
      const TargetLowering *TLI = getTargetLowering();
      MVT ResVecTySimple = ResVecTy.getSimpleVT();
      const TargetRegisterClass *RC = TLI->getRegClassFor(ResVecTySimple);
      Res = CurDAG->getMachineNode(Mips::COPY_TO_REGCLASS, DL,
                                   ResVecTy, SDValue(Res, 0),
                                   CurDAG->getTargetConstant(RC->getID(), DL,
                                                             MVT::i32));
    }
```
- EN: Declares `to`, packaging target-specific state and APIs around `MipsSEISelDAGToDAG`.
- CN: 这里声明 `to`，把与 `MipsSEISelDAGToDAG` 相关的目标特定状态和 API 组织在一起。

### Lines 1321-1323
```cpp
    ReplaceNode(Node, Res);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1325-1325
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1327-1328
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1330-1333
```cpp
bool MipsSEDAGToDAGISel::SelectInlineAsmMemoryOperand(
    const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
    std::vector<SDValue> &OutOps) {
  SDValue Base, Offset;
```
- EN: Implements `MipsSEDAGToDAGISel::SelectInlineAsmMemoryOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISel::SelectInlineAsmMemoryOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1335-1349
```cpp
  switch(ConstraintID) {
  default:
    llvm_unreachable("Unexpected asm memory constraint");
  // All memory constraints can at least accept raw pointers.
  case InlineAsm::ConstraintCode::m:
  case InlineAsm::ConstraintCode::o:
    if (selectAddrRegImm16(Op, Base, Offset)) {
      OutOps.push_back(Base);
      OutOps.push_back(Offset);
      return false;
    }
    OutOps.push_back(Op);
    OutOps.push_back(CurDAG->getTargetConstant(0, SDLoc(Op), MVT::i32));
    return false;
  case InlineAsm::ConstraintCode::R:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1350-1364
```cpp
    // The 'R' constraint is supposed to be much more complicated than this.
    // However, it's becoming less useful due to architectural changes and
    // ought to be replaced by other constraints such as 'ZC'.
    // For now, support 9-bit signed offsets which is supportable by all
    // subtargets for all instructions.
    if (selectAddrRegImm9(Op, Base, Offset)) {
      OutOps.push_back(Base);
      OutOps.push_back(Offset);
      return false;
    }
    OutOps.push_back(Op);
    OutOps.push_back(CurDAG->getTargetConstant(0, SDLoc(Op), MVT::i32));
    return false;
  case InlineAsm::ConstraintCode::ZC:
    // ZC matches whatever the pref, ll, and sc instructions can handle for the
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1365-1379
```cpp
    // given subtarget.
    if (Subtarget->inMicroMipsMode()) {
      // On microMIPS, they can handle 12-bit offsets.
      if (selectAddrRegImm12(Op, Base, Offset)) {
        OutOps.push_back(Base);
        OutOps.push_back(Offset);
        return false;
      }
    } else if (Subtarget->hasMips32r6()) {
      // On MIPS32r6/MIPS64r6, they can only handle 9-bit offsets.
      if (selectAddrRegImm9(Op, Base, Offset)) {
        OutOps.push_back(Base);
        OutOps.push_back(Offset);
        return false;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1380-1392
```cpp
    } else if (selectAddrRegImm16(Op, Base, Offset)) {
      // Prior to MIPS32r6/MIPS64r6, they can handle 16-bit offsets.
      OutOps.push_back(Base);
      OutOps.push_back(Offset);
      return false;
    }
    // In all cases, 0-bit offsets are acceptable.
    OutOps.push_back(Op);
    OutOps.push_back(CurDAG->getTargetConstant(0, SDLoc(Op), MVT::i32));
    return false;
  }
  return true;
}
```
- EN: Implements `selectAddrRegImm16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `selectAddrRegImm16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1394-1396
```cpp
MipsSEDAGToDAGISelLegacy::MipsSEDAGToDAGISelLegacy(MipsTargetMachine &TM,
                                                   CodeGenOptLevel OL)
    : MipsDAGToDAGISelLegacy(std::make_unique<MipsSEDAGToDAGISel>(TM, OL)) {}
```
- EN: Implements `MipsSEDAGToDAGISelLegacy::MipsSEDAGToDAGISelLegacy`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEDAGToDAGISelLegacy::MipsSEDAGToDAGISelLegacy`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1398-1401
```cpp
FunctionPass *llvm::createMipsSEISelDag(MipsTargetMachine &TM,
                                        CodeGenOptLevel OptLevel) {
  return new MipsSEDAGToDAGISelLegacy(TM, OptLevel);
}
```
- EN: Implements `llvm::createMipsSEISelDag`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsSEISelDag`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsSEISelDAGToDAG.h`, `Mips.h`, `MipsAnalyzeImmediate.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`.
  - CN: 后端本地头文件：`MipsSEISelDAGToDAG.h`, `Mips.h`, `MipsAnalyzeImmediate.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/IR/Dominators.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Instructions.h` ... (+5 more).
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/SelectionDAGNodes.h`, `llvm/IR/Dominators.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/Instructions.h` ... (+5 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
