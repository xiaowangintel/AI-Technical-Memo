# R600ExpandSpecialInstrs.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600ExpandSpecialInstrs.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600ExpandSpecialInstrs for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600ExpandSpecialInstrs 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===- R600ExpandSpecialInstrs.cpp - Expand special instructions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Vector, Reduction, and Cube instructions need to fill the entire instruction
/// group to work correctly.  This pass expands these individual instructions
/// into several instructions that will completely fill the instruction group.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600.h"
#include "R600Defines.h"
#include "R600Subtarget.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"

using namespace llvm;

#define DEBUG_TYPE "r600-expand-special-instrs"

namespace {

class R600ExpandSpecialInstrsPass : public MachineFunctionPass {
private:
  const R600InstrInfo *TII = nullptr;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600ExpandSpecialInstrsPass`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600ExpandSpecialInstrsPass`。

### Lines 33-62: Registers LLVM passes
```cpp
  void SetFlagInNewMI(MachineInstr *NewMI, const MachineInstr *OldMI,
                      R600::OpName Op);

public:
  static char ID;

  R600ExpandSpecialInstrsPass() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "R600 Expand special instructions pass";
  }
};

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(R600ExpandSpecialInstrsPass, DEBUG_TYPE,
                     "R600 Expand Special Instrs", false, false)
INITIALIZE_PASS_END(R600ExpandSpecialInstrsPass, DEBUG_TYPE,
                    "R600ExpandSpecialInstrs", false, false)

char R600ExpandSpecialInstrsPass::ID = 0;

char &llvm::R600ExpandSpecialInstrsPassID = R600ExpandSpecialInstrsPass::ID;

FunctionPass *llvm::createR600ExpandSpecialInstrsPass() {
  return new R600ExpandSpecialInstrsPass();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createR600ExpandSpecialInstrsPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createR600ExpandSpecialInstrsPass`。

### Lines 63-84: Implements R600ExpandSpecialInstrsPass::SetFlagInNewMI
```cpp
void R600ExpandSpecialInstrsPass::SetFlagInNewMI(MachineInstr *NewMI,
                                                 const MachineInstr *OldMI,
                                                 R600::OpName Op) {
  int OpIdx = TII->getOperandIdx(*OldMI, Op);
  if (OpIdx > -1) {
    uint64_t Val = OldMI->getOperand(OpIdx).getImm();
    TII->setImmOperand(*NewMI, Op, Val);
  }
}

bool R600ExpandSpecialInstrsPass::runOnMachineFunction(MachineFunction &MF) {
  const R600Subtarget &ST = MF.getSubtarget<R600Subtarget>();
  TII = ST.getInstrInfo();

  const R600RegisterInfo &TRI = TII->getRegisterInfo();

  for (MachineBasicBlock &MBB : MF) {
    MachineBasicBlock::iterator I = MBB.begin();
    while (I != MBB.end()) {
      MachineInstr &MI = *I;
      I = std::next(I);

```
**EN:** This section contains concrete logic for R600ExpandSpecialInstrsPass::SetFlagInNewMI. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600ExpandSpecialInstrsPass::SetFlagInNewMI`, `R600ExpandSpecialInstrsPass::runOnMachineFunction`, `std::next`.
**CN:** 本节包含与 R600ExpandSpecialInstrsPass::SetFlagInNewMI 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600ExpandSpecialInstrsPass::SetFlagInNewMI`, `R600ExpandSpecialInstrsPass::runOnMachineFunction`, `std::next`。

### Lines 85-118: Switch-based control flow
```cpp
      // Expand LDS_*_RET instructions
      if (TII->isLDSRetInstr(MI.getOpcode())) {
        int DstIdx = TII->getOperandIdx(MI.getOpcode(), R600::OpName::dst);
        assert(DstIdx != -1);
        MachineOperand &DstOp = MI.getOperand(DstIdx);
        MachineInstr *Mov = TII->buildMovInstr(&MBB, I,
                                               DstOp.getReg(), R600::OQAP);
        DstOp.setReg(R600::OQAP);
        int LDSPredSelIdx = TII->getOperandIdx(MI.getOpcode(),
                                           R600::OpName::pred_sel);
        int MovPredSelIdx = TII->getOperandIdx(Mov->getOpcode(),
                                           R600::OpName::pred_sel);
        // Copy the pred_sel bit
        Mov->getOperand(MovPredSelIdx).setReg(
            MI.getOperand(LDSPredSelIdx).getReg());
      }

      switch (MI.getOpcode()) {
      default: break;
      // Expand PRED_X to one of the PRED_SET instructions.
      case R600::PRED_X: {
        uint64_t Flags = MI.getOperand(3).getImm();
        // The native opcode used by PRED_X is stored as an immediate in the
        // third operand.
        MachineInstr *PredSet = TII->buildDefaultInstruction(MBB, I,
                                            MI.getOperand(2).getImm(), // opcode
                                            MI.getOperand(0).getReg(), // dst
                                            MI.getOperand(1).getReg(), // src0
                                            R600::ZERO);             // src1
        TII->addFlag(*PredSet, 0, MO_FLAG_MASK);
        if (Flags & MO_FLAG_PUSH) {
          TII->setImmOperand(*PredSet, R600::OpName::update_exec_mask, 1);
        } else {
          TII->setImmOperand(*PredSet, R600::OpName::update_pred, 1);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 119-152: Conditional logic and checks
```cpp
        }
        MI.eraseFromParent();
        continue;
        }
      case R600::DOT_4: {

        const R600RegisterInfo &TRI = TII->getRegisterInfo();

        Register DstReg = MI.getOperand(0).getReg();
        unsigned DstBase = TRI.getEncodingValue(DstReg) & HW_REG_MASK;

        for (unsigned Chan = 0; Chan < 4; ++Chan) {
          bool Mask = (Chan != TRI.getHWRegChan(DstReg));
          unsigned SubDstReg =
              R600::R600_TReg32RegClass.getRegister((DstBase * 4) + Chan);
          MachineInstr *BMI =
              TII->buildSlotOfVectorInstruction(MBB, &MI, Chan, SubDstReg);
          if (Chan > 0) {
            BMI->bundleWithPred();
          }
          if (Mask) {
            TII->addFlag(*BMI, 0, MO_FLAG_MASK);
          }
          if (Chan != 3)
            TII->addFlag(*BMI, 0, MO_FLAG_NOT_LAST);
          unsigned Opcode = BMI->getOpcode();
          // While not strictly necessary from hw point of view, we force
          // all src operands of a dot4 inst to belong to the same slot.
          Register Src0 =
              BMI->getOperand(TII->getOperandIdx(Opcode, R600::OpName::src0))
                  .getReg();
          Register Src1 =
              BMI->getOperand(TII->getOperandIdx(Opcode, R600::OpName::src1))
                  .getReg();
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 153-170: Conditional logic and checks
```cpp
          (void) Src0;
          (void) Src1;
          if ((TRI.getEncodingValue(Src0) & 0xff) < 127 &&
              (TRI.getEncodingValue(Src1) & 0xff) < 127)
            assert(TRI.getHWRegChan(Src0) == TRI.getHWRegChan(Src1));
        }
        MI.eraseFromParent();
        continue;
      }
      }

      bool IsReduction = TII->isReductionOp(MI.getOpcode());
      bool IsVector = TII->isVector(MI);
      bool IsCube = TII->isCubeOp(MI.getOpcode());
      if (!IsReduction && !IsVector && !IsCube) {
        continue;
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 171-202: Implementation details and local logic
```cpp
      // Expand the instruction
      //
      // Reduction instructions:
      // T0_X = DP4 T1_XYZW, T2_XYZW
      // becomes:
      // TO_X = DP4 T1_X, T2_X
      // TO_Y (write masked) = DP4 T1_Y, T2_Y
      // TO_Z (write masked) = DP4 T1_Z, T2_Z
      // TO_W (write masked) = DP4 T1_W, T2_W
      //
      // Vector instructions:
      // T0_X = MULLO_INT T1_X, T2_X
      // becomes:
      // T0_X = MULLO_INT T1_X, T2_X
      // T0_Y (write masked) = MULLO_INT T1_X, T2_X
      // T0_Z (write masked) = MULLO_INT T1_X, T2_X
      // T0_W (write masked) = MULLO_INT T1_X, T2_X
      //
      // Cube instructions:
      // T0_XYZW = CUBE T1_XYZW
      // becomes:
      // TO_X = CUBE T1_Z, T1_Y
      // T0_Y = CUBE T1_Z, T1_X
      // T0_Z = CUBE T1_X, T1_Z
      // T0_W = CUBE T1_Y, T1_Z
      for (unsigned Chan = 0; Chan < 4; Chan++) {
        Register DstReg =
            MI.getOperand(TII->getOperandIdx(MI, R600::OpName::dst)).getReg();
        Register Src0 =
            MI.getOperand(TII->getOperandIdx(MI, R600::OpName::src0)).getReg();
        unsigned Src1 = 0;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 203-235: Conditional logic and checks
```cpp
        // Determine the correct source registers
        if (!IsCube) {
          int Src1Idx = TII->getOperandIdx(MI, R600::OpName::src1);
          if (Src1Idx != -1) {
            Src1 = MI.getOperand(Src1Idx).getReg();
          }
        }
        if (IsReduction) {
          unsigned SubRegIndex = R600RegisterInfo::getSubRegFromChannel(Chan);
          Src0 = TRI.getSubReg(Src0, SubRegIndex);
          Src1 = TRI.getSubReg(Src1, SubRegIndex);
        } else if (IsCube) {
          static const int CubeSrcSwz[] = {2, 2, 0, 1};
          unsigned SubRegIndex0 = R600RegisterInfo::getSubRegFromChannel(CubeSrcSwz[Chan]);
          unsigned SubRegIndex1 = R600RegisterInfo::getSubRegFromChannel(CubeSrcSwz[3 - Chan]);
          Src1 = TRI.getSubReg(Src0, SubRegIndex1);
          Src0 = TRI.getSubReg(Src0, SubRegIndex0);
        }

        // Determine the correct destination registers;
        bool Mask = false;
        bool NotLast = true;
        if (IsCube) {
          unsigned SubRegIndex = R600RegisterInfo::getSubRegFromChannel(Chan);
          DstReg = TRI.getSubReg(DstReg, SubRegIndex);
        } else {
          // Mask the write if the original instruction does not write to
          // the current Channel.
          Mask = (Chan != TRI.getHWRegChan(DstReg));
          unsigned DstBase = TRI.getEncodingValue(DstReg) & HW_REG_MASK;
          DstReg = R600::R600_TReg32RegClass.getRegister((DstBase * 4) + Chan);
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600RegisterInfo::getSubRegFromChannel`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600RegisterInfo::getSubRegFromChannel`。

### Lines 236-254: Switch-based control flow
```cpp
        // Set the IsLast bit
        NotLast = (Chan != 3 );

        // Add the new instruction
        unsigned Opcode = MI.getOpcode();
        switch (Opcode) {
        case R600::CUBE_r600_pseudo:
          Opcode = R600::CUBE_r600_real;
          break;
        case R600::CUBE_eg_pseudo:
          Opcode = R600::CUBE_eg_real;
          break;
        default:
          break;
        }

        MachineInstr *NewMI =
          TII->buildDefaultInstruction(MBB, I, Opcode, DstReg, Src0, Src1);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 255-275: Conditional logic and checks
```cpp
        if (Chan != 0)
          NewMI->bundleWithPred();
        if (Mask) {
          TII->addFlag(*NewMI, 0, MO_FLAG_MASK);
        }
        if (NotLast) {
          TII->addFlag(*NewMI, 0, MO_FLAG_NOT_LAST);
        }
        SetFlagInNewMI(NewMI, &MI, R600::OpName::clamp);
        SetFlagInNewMI(NewMI, &MI, R600::OpName::literal);
        SetFlagInNewMI(NewMI, &MI, R600::OpName::src0_abs);
        SetFlagInNewMI(NewMI, &MI, R600::OpName::src1_abs);
        SetFlagInNewMI(NewMI, &MI, R600::OpName::src0_neg);
        SetFlagInNewMI(NewMI, &MI, R600::OpName::src1_neg);
      }
      MI.eraseFromParent();
    }
  }
  finalizeBundles(MF);
  return false;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600ExpandSpecialInstrsPass`, `llvm::createR600ExpandSpecialInstrsPass`, `R600ExpandSpecialInstrsPass::SetFlagInNewMI`, `R600ExpandSpecialInstrsPass::runOnMachineFunction`, `std::next`, `R600RegisterInfo::getSubRegFromChannel`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600.h"`
- `"R600Defines.h"`
- `"R600Subtarget.h"`
- `"llvm/CodeGen/MachineFunction.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
