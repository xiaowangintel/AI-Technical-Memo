# RISCVVectorPeephole.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVVectorPeephole.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements peephole optimizations for RISC-V vector machine instructions. / 实现RISC-V 向量机器指令的窥孔优化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Commentary and design intent / 注释与设计意图
```cpp
//===- RISCVVectorPeephole.cpp - MI Vector Pseudo Peepholes ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass performs various vector pseudo peephole optimisations after
// instruction selection.
//
// Currently it converts vmerge.vvm to vmv.v.v
// PseudoVMERGE_VVM %false, %false, %true, %allonesmask, %vl, %sew
// ->
// PseudoVMV_V_V %false, %true, %vl, %sew
//
// And masked pseudos to unmasked pseudos
// PseudoVADD_V_V_MASK %passthru, %a, %b, %allonesmask, %vl, sew, policy
// ->
// PseudoVADD_V_V %passthru %a, %b, %vl, sew, policy
//
// It also converts AVLs to VLMAX where possible
// %vl = VLENB * something
// PseudoVADD_V_V %passthru, %a, %b, %vl, sew, policy
// ->
// PseudoVADD_V_V %passthru, %a, %b, -1, sew, policy
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 29-42: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"

using namespace llvm;

#define DEBUG_TYPE "riscv-vector-peephole"

namespace {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 43-56: Type declaration for RISCVVectorPeephole / RISCVVectorPeephole 的类型声明
```cpp
class RISCVVectorPeephole : public MachineFunctionPass {
public:
  static char ID;
  const TargetInstrInfo *TII;
  MachineRegisterInfo *MRI;
  const TargetRegisterInfo *TRI;
  const RISCVSubtarget *ST;
  RISCVVectorPeephole() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;
  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setIsSSA();
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 57-70: Function implementation: getPassName / 函数实现：getPassName
```cpp
  StringRef getPassName() const override {
    return "RISC-V Vector Peephole Optimization";
  }

private:
  bool convertToVLMAX(MachineInstr &MI) const;
  bool convertToWholeRegister(MachineInstr &MI) const;
  bool convertToUnmasked(MachineInstr &MI) const;
  bool convertAllOnesVMergeToVMv(MachineInstr &MI) const;
  bool convertSameMaskVMergeToVMv(MachineInstr &MI);
  bool foldUndefPassthruVMV_V_V(MachineInstr &MI);
  bool foldVMV_V_V(MachineInstr &MI);
  bool foldVMergeToMask(MachineInstr &MI) const;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 71-86: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  bool hasSameEEW(const MachineInstr &User, const MachineInstr &Src) const;
  bool isAllOnesMask(const MachineInstr *MaskDef) const;
  std::optional<unsigned> getConstant(const MachineOperand &VL) const;
  bool ensureDominates(const MachineOperand &Use, MachineInstr &Src) const;
  Register
  lookThruCopies(Register Reg, bool OneUseOnly = false,
                 SmallVectorImpl<MachineInstr *> *Copies = nullptr) const;
};

} // namespace

char RISCVVectorPeephole::ID = 0;

INITIALIZE_PASS(RISCVVectorPeephole, DEBUG_TYPE, "RISC-V Fold Masks", false,
                false)
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 87-105: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
/// Given \p User that has an input operand with EEW=SEW, which uses the dest
/// operand of \p Src with an unknown EEW, return true if their EEWs match.
bool RISCVVectorPeephole::hasSameEEW(const MachineInstr &User,
                                     const MachineInstr &Src) const {
  unsigned UserLog2SEW =
      User.getOperand(RISCVII::getSEWOpNum(User.getDesc())).getImm();
  unsigned SrcLog2SEW =
      Src.getOperand(RISCVII::getSEWOpNum(Src.getDesc())).getImm();
  unsigned SrcLog2EEW = RISCV::getDestLog2EEW(
      TII->get(RISCV::getRVVMCOpcode(Src.getOpcode())), SrcLog2SEW);
  return SrcLog2EEW == UserLog2SEW;
}

/// Check if an operand is an immediate or a materialized ADDI $x0, imm.
std::optional<unsigned>
RISCVVectorPeephole::getConstant(const MachineOperand &VL) const {
  if (VL.isImm())
    return VL.getImm();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 106-127: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineInstr *Def = MRI->getVRegDef(VL.getReg());
  if (!Def || Def->getOpcode() != RISCV::ADDI ||
      Def->getOperand(1).getReg() != RISCV::X0)
    return std::nullopt;
  return Def->getOperand(2).getImm();
}

/// Convert AVLs that are known to be VLMAX to the VLMAX sentinel.
bool RISCVVectorPeephole::convertToVLMAX(MachineInstr &MI) const {
  if (!RISCVII::hasVLOp(MI.getDesc().TSFlags) ||
      !RISCVII::hasSEWOp(MI.getDesc().TSFlags))
    return false;

  auto LMUL = RISCVVType::decodeVLMUL(RISCVII::getLMul(MI.getDesc().TSFlags));
  // Fixed-point value, denominator=8
  unsigned LMULFixed = LMUL.second ? (8 / LMUL.first) : 8 * LMUL.first;
  unsigned Log2SEW = MI.getOperand(RISCVII::getSEWOpNum(MI.getDesc())).getImm();
  // A Log2SEW of 0 is an operation on mask registers only
  unsigned SEW = Log2SEW ? 1 << Log2SEW : 8;
  assert(RISCVVType::isValidSEW(SEW) && "Unexpected SEW");
  assert(8 * LMULFixed / SEW > 0);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 128-143: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If the exact VLEN is known then we know VLMAX, check if the AVL == VLMAX.
  MachineOperand &VL = MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
  if (auto VLen = ST->getRealVLen(), AVL = getConstant(VL);
      VLen && AVL && (*VLen * LMULFixed) / SEW == *AVL * 8) {
    VL.ChangeToImmediate(RISCV::VLMaxSentinel);
    return true;
  }

  // If an AVL is a VLENB that's possibly scaled to be equal to VLMAX, convert
  // it to the VLMAX sentinel value.
  if (!VL.isReg())
    return false;
  MachineInstr *Def = MRI->getVRegDef(VL.getReg());
  if (!Def)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 144-159: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Fixed-point value, denominator=8
  uint64_t ScaleFixed = 8;
  // Check if the VLENB was potentially scaled with slli/srli
  if (Def->getOpcode() == RISCV::SLLI) {
    assert(Def->getOperand(2).getImm() < 64);
    ScaleFixed <<= Def->getOperand(2).getImm();
    Def = MRI->getVRegDef(Def->getOperand(1).getReg());
  } else if (Def->getOpcode() == RISCV::SRLI) {
    assert(Def->getOperand(2).getImm() < 64);
    ScaleFixed >>= Def->getOperand(2).getImm();
    Def = MRI->getVRegDef(Def->getOperand(1).getReg());
  }

  if (!Def || Def->getOpcode() != RISCV::PseudoReadVLENB)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 160-173: Commentary and design intent / 注释与设计意图
```cpp
  // AVL = (VLENB * Scale)
  //
  // VLMAX = (VLENB * 8 * LMUL) / SEW
  //
  // AVL == VLMAX
  // -> VLENB * Scale == (VLENB * 8 * LMUL) / SEW
  // -> Scale == (8 * LMUL) / SEW
  if (ScaleFixed != 8 * LMULFixed / SEW)
    return false;

  VL.ChangeToImmediate(RISCV::VLMaxSentinel);

  return true;
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 174-193: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVVectorPeephole::isAllOnesMask(const MachineInstr *MaskDef) const {
  while (MaskDef->isCopy() && MaskDef->getOperand(1).getReg().isVirtual())
    MaskDef = MRI->getVRegDef(MaskDef->getOperand(1).getReg());

  // TODO: Check that the VMSET is the expected bitwidth? The pseudo has
  // undefined behaviour if it's the wrong bitwidth, so we could choose to
  // assume that it's all-ones? Same applies to its VL.
  switch (MaskDef->getOpcode()) {
  case RISCV::PseudoVMSET_M_B1:
  case RISCV::PseudoVMSET_M_B2:
  case RISCV::PseudoVMSET_M_B4:
  case RISCV::PseudoVMSET_M_B8:
  case RISCV::PseudoVMSET_M_B16:
  case RISCV::PseudoVMSET_M_B32:
  case RISCV::PseudoVMSET_M_B64:
    return true;
  default:
    return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 194-219: Header guard and interface framing / 头文件保护与接口框架
```cpp
}

/// Convert unit strided unmasked loads and stores to whole-register equivalents
/// to avoid the dependency on $vl and $vtype.
///
/// %x = PseudoVLE8_V_M1 %passthru, %ptr, %vlmax, policy
/// PseudoVSE8_V_M1 %v, %ptr, %vlmax
///
/// ->
///
/// %x = VL1RE8_V %ptr
/// VS1R_V %v, %ptr
bool RISCVVectorPeephole::convertToWholeRegister(MachineInstr &MI) const {
#define CASE_WHOLE_REGISTER_LMUL_SEW(lmul, sew)                                \
  case RISCV::PseudoVLE##sew##_V_M##lmul:                                      \
    NewOpc = RISCV::VL##lmul##RE##sew##_V;                                     \
    break;                                                                     \
  case RISCV::PseudoVSE##sew##_V_M##lmul:                                      \
    NewOpc = RISCV::VS##lmul##R_V;                                             \
    break;
#define CASE_WHOLE_REGISTER_LMUL(lmul)                                         \
  CASE_WHOLE_REGISTER_LMUL_SEW(lmul, 8)                                        \
  CASE_WHOLE_REGISTER_LMUL_SEW(lmul, 16)                                       \
  CASE_WHOLE_REGISTER_LMUL_SEW(lmul, 32)                                       \
  CASE_WHOLE_REGISTER_LMUL_SEW(lmul, 64)
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 220-233: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  unsigned NewOpc;
  switch (MI.getOpcode()) {
    CASE_WHOLE_REGISTER_LMUL(1)
    CASE_WHOLE_REGISTER_LMUL(2)
    CASE_WHOLE_REGISTER_LMUL(4)
    CASE_WHOLE_REGISTER_LMUL(8)
  default:
    return false;
  }

  MachineOperand &VLOp = MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
  if (!VLOp.isImm() || VLOp.getImm() != RISCV::VLMaxSentinel)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 234-247: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Whole register instructions aren't pseudos so they don't have
  // policy/SEW/AVL ops, and they don't have passthrus.
  if (RISCVII::hasVecPolicyOp(MI.getDesc().TSFlags))
    MI.removeOperand(RISCVII::getVecPolicyOpNum(MI.getDesc()));
  MI.removeOperand(RISCVII::getSEWOpNum(MI.getDesc()));
  MI.removeOperand(RISCVII::getVLOpNum(MI.getDesc()));
  if (RISCVII::isFirstDefTiedToFirstUse(MI.getDesc()))
    MI.removeOperand(1);

  MI.setDesc(TII->get(NewOpc));

  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 248-262: Header guard and interface framing / 头文件保护与接口框架
```cpp
static unsigned getVMV_V_VOpcodeForVMERGE_VVM(const MachineInstr &MI) {
#define CASE_VMERGE_TO_VMV(lmul)                                               \
  case RISCV::PseudoVMERGE_VVM_##lmul:                                         \
    return RISCV::PseudoVMV_V_V_##lmul;
  switch (MI.getOpcode()) {
  default:
    return 0;
    CASE_VMERGE_TO_VMV(MF8)
    CASE_VMERGE_TO_VMV(MF4)
    CASE_VMERGE_TO_VMV(MF2)
    CASE_VMERGE_TO_VMV(M1)
    CASE_VMERGE_TO_VMV(M2)
    CASE_VMERGE_TO_VMV(M4)
    CASE_VMERGE_TO_VMV(M8)
  }
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 263-276: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

/// Convert a PseudoVMERGE_VVM with an all ones mask to a PseudoVMV_V_V.
///
/// %x = PseudoVMERGE_VVM %passthru, %false, %true, %allones, sew, vl
/// ->
/// %x = PseudoVMV_V_V %passthru, %true, vl, sew, tu_mu
bool RISCVVectorPeephole::convertAllOnesVMergeToVMv(MachineInstr &MI) const {
  unsigned NewOpc = getVMV_V_VOpcodeForVMERGE_VVM(MI);
  if (!NewOpc)
    return false;
  if (!isAllOnesMask(MRI->getVRegDef(MI.getOperand(4).getReg())))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 277-290: Type declaration for for / for 的类型声明
```cpp
  MI.setDesc(TII->get(NewOpc));
  MI.removeOperand(2); // False operand
  MI.removeOperand(3); // Mask operand
  MI.addOperand(
      MachineOperand::CreateImm(RISCVVType::TAIL_UNDISTURBED_MASK_UNDISTURBED));

  // vmv.v.v doesn't have a mask operand, so we may be able to inflate the
  // register class for the destination and passthru operands e.g. VRNoV0 -> VR
  MRI->recomputeRegClass(MI.getOperand(0).getReg());
  if (MI.getOperand(1).getReg().isValid())
    MRI->recomputeRegClass(MI.getOperand(1).getReg());
  return true;
}
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 291-307: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
// If \p Reg is defined by one or more COPYs of virtual registers, traverses
// the chain and returns the root non-COPY source.
Register RISCVVectorPeephole::lookThruCopies(
    Register Reg, bool OneUseOnly,
    SmallVectorImpl<MachineInstr *> *Copies) const {
  while (MachineInstr *Def = MRI->getUniqueVRegDef(Reg)) {
    if (!Def->isFullCopy())
      break;
    Register Src = Def->getOperand(1).getReg();
    if (!Src.isVirtual())
      break;
    if (OneUseOnly && !MRI->hasOneNonDBGUse(Reg))
      break;
    if (Copies)
      Copies->push_back(Def);
    Reg = Src;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 308-325: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return Reg;
}

/// If a PseudoVMERGE_VVM's true operand is a masked pseudo and both have the
/// same mask, and the masked pseudo's passthru is the same as the false
/// operand, we can convert the PseudoVMERGE_VVM to a PseudoVMV_V_V.
///
/// %true = PseudoVADD_VV_M1_MASK %false, %x, %y, %mask, vl1, sew, policy
/// %x = PseudoVMERGE_VVM %passthru, %false, %true, %mask, vl2, sew
/// ->
/// %true = PseudoVADD_VV_M1_MASK %false, %x, %y, %mask, vl1, sew, policy
/// %x = PseudoVMV_V_V %passthru, %true, vl2, sew, tu_mu
bool RISCVVectorPeephole::convertSameMaskVMergeToVMv(MachineInstr &MI) {
  unsigned NewOpc = getVMV_V_VOpcodeForVMERGE_VVM(MI);
  if (!NewOpc)
    return false;
  MachineInstr *True = MRI->getVRegDef(MI.getOperand(3).getReg());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 326-339: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (!True || True->getParent() != MI.getParent())
    return false;

  auto *TrueMaskedInfo = RISCV::getMaskedPseudoInfo(True->getOpcode());
  if (!TrueMaskedInfo || !hasSameEEW(MI, *True))
    return false;

  Register TrueMaskReg = lookThruCopies(
      True->getOperand(TrueMaskedInfo->MaskOpIdx + True->getNumExplicitDefs())
          .getReg());
  Register MIMaskReg = lookThruCopies(MI.getOperand(4).getReg());
  if (!TrueMaskReg.isVirtual() || TrueMaskReg != MIMaskReg)
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 340-358: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Masked off lanes past TrueVL will come from False, and converting to vmv
  // will lose these lanes unless MIVL <= TrueVL.
  // We can relax this when False == Passthru and True's tail policy is TU,
  // because True's tail lanes will preserve its passthru (= False = Passthru).
  const MachineOperand &MIVL = MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
  const MachineOperand &TrueVL =
      True->getOperand(RISCVII::getVLOpNum(True->getDesc()));
  Register FalseReg = MI.getOperand(2).getReg();
  if (!RISCV::isVLKnownLE(MIVL, TrueVL)) {
    Register PassthruReg = MI.getOperand(1).getReg();
    if (FalseReg.isValid() && FalseReg != PassthruReg)
      return false;
    if (!RISCVII::hasVecPolicyOp(True->getDesc().TSFlags))
      return false;
    uint64_t TruePolicy =
        True->getOperand(RISCVII::getVecPolicyOpNum(True->getDesc())).getImm();
    if (TruePolicy & RISCVVType::TAIL_AGNOSTIC)
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 359-372: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // True's passthru needs to be equivalent to False
  Register TruePassthruReg = True->getOperand(1).getReg();
  if (TruePassthruReg != FalseReg) {
    // If True's passthru is undef see if we can change it to False
    if (TruePassthruReg.isValid() ||
        !MRI->hasOneUse(MI.getOperand(3).getReg()) ||
        !ensureDominates(MI.getOperand(2), *True))
      return false;
    True->getOperand(1).setReg(MI.getOperand(2).getReg());
    // If True is masked then its passthru needs to be in VRNoV0.
    MRI->constrainRegClass(True->getOperand(1).getReg(),
                           TII->getRegClass(True->getDesc(), 1));
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 373-386: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // If True is mask agnostic, we need to make it mask undisturbed.
  if (RISCVII::hasVecPolicyOp(True->getDesc().TSFlags)) {
    MachineOperand &PolicyOp =
        True->getOperand(RISCVII::getVecPolicyOpNum(True->getDesc()));
    PolicyOp.setImm(PolicyOp.getImm() & ~RISCVVType::MASK_AGNOSTIC);
  }

  MI.setDesc(TII->get(NewOpc));
  MI.removeOperand(2); // False operand
  MI.removeOperand(3); // Mask operand
  MI.addOperand(
      MachineOperand::CreateImm(RISCVVType::TAIL_UNDISTURBED_MASK_UNDISTURBED));
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 387-400: Type declaration for for / for 的类型声明
```cpp
  // vmv.v.v doesn't have a mask operand, so we may be able to inflate the
  // register class for the destination and passthru operands e.g. VRNoV0 -> VR
  MRI->recomputeRegClass(MI.getOperand(0).getReg());
  if (MI.getOperand(1).getReg().isValid())
    MRI->recomputeRegClass(MI.getOperand(1).getReg());
  return true;
}

bool RISCVVectorPeephole::convertToUnmasked(MachineInstr &MI) const {
  const RISCV::RISCVMaskedPseudoInfo *I =
      RISCV::getMaskedPseudoInfo(MI.getOpcode());
  if (!I)
    return false;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 401-420: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (!isAllOnesMask(MRI->getVRegDef(
          MI.getOperand(I->MaskOpIdx + MI.getNumExplicitDefs()).getReg())))
    return false;

  // There are two classes of pseudos in the table - compares and
  // everything else.  See the comment on RISCVMaskedPseudo for details.
  const unsigned Opc = I->UnmaskedPseudo;
  const MCInstrDesc &MCID = TII->get(Opc);
  [[maybe_unused]] const bool HasPolicyOp =
      RISCVII::hasVecPolicyOp(MCID.TSFlags);
  const bool HasPassthru = RISCVII::isFirstDefTiedToFirstUse(MCID);
  const MCInstrDesc &MaskedMCID = TII->get(MI.getOpcode());
  assert((RISCVII::hasVecPolicyOp(MaskedMCID.TSFlags) ||
          !RISCVII::hasVecPolicyOp(MCID.TSFlags)) &&
         "Unmasked pseudo has policy but masked pseudo doesn't?");
  assert(HasPolicyOp == HasPassthru && "Unexpected pseudo structure");
  assert(!(HasPassthru && !RISCVII::isFirstDefTiedToFirstUse(MaskedMCID)) &&
         "Unmasked with passthru but masked with no passthru?");
  (void)HasPolicyOp;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 421-435: Type declaration for RISCVII::hasVecPolicyOp / RISCVII::hasVecPolicyOp 的类型声明
```cpp
  MI.setDesc(MCID);

  // Drop the policy operand if unmasked doesn't need it.
  if (RISCVII::hasVecPolicyOp(MaskedMCID.TSFlags) &&
      !RISCVII::hasVecPolicyOp(MCID.TSFlags))
    MI.removeOperand(RISCVII::getVecPolicyOpNum(MaskedMCID));

  // TODO: Increment all MaskOpIdxs in tablegen by num of explicit defs?
  unsigned MaskOpIdx = I->MaskOpIdx + MI.getNumExplicitDefs();
  MI.removeOperand(MaskOpIdx);

  // The unmasked pseudo will no longer be constrained to the vrnov0 reg class,
  // so try and relax it to vr.
  MRI->recomputeRegClass(MI.getOperand(0).getReg());
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 436-457: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If the original masked pseudo had a passthru, relax it or remove it.
  if (RISCVII::isFirstDefTiedToFirstUse(MaskedMCID)) {
    unsigned PassthruOpIdx = MI.getNumExplicitDefs();
    if (HasPassthru) {
      if (MI.getOperand(PassthruOpIdx).getReg())
        MRI->recomputeRegClass(MI.getOperand(PassthruOpIdx).getReg());
    } else
      MI.removeOperand(PassthruOpIdx);
  }

  return true;
}

/// Given A and B are in the same MBB, returns true if A comes before B.
static bool dominates(MachineBasicBlock::const_iterator A,
                      MachineBasicBlock::const_iterator B) {
  assert(A->getParent() == B->getParent());
  const MachineBasicBlock *MBB = A->getParent();
  auto MBBEnd = MBB->end();
  if (B == MBBEnd)
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 458-473: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineBasicBlock::const_iterator I = MBB->begin();
  for (; &*I != A && &*I != B; ++I)
    ;

  return &*I == A;
}

/// If the register in \p MO doesn't dominate \p Src, try to move \p Src so it
/// does. Returns false if doesn't dominate and we can't move. \p MO must be in
/// the same basic block as \Src.
bool RISCVVectorPeephole::ensureDominates(const MachineOperand &MO,
                                          MachineInstr &Src) const {
  assert(MO.getParent()->getParent() == Src.getParent());
  if (!MO.isReg() || !MO.getReg().isValid())
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 474-490: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  MachineInstr *Def = MRI->getVRegDef(MO.getReg());
  if (Def->getParent() == Src.getParent() && !dominates(Def, Src)) {
    if (!RISCVInstrInfo::isSafeToMove(Src, *Def->getNextNode()))
      return false;
    Src.moveBefore(Def->getNextNode());
  }

  return true;
}

/// If a PseudoVMV_V_V's passthru is undef then we can replace it with its input
bool RISCVVectorPeephole::foldUndefPassthruVMV_V_V(MachineInstr &MI) {
  if (RISCV::getRVVMCOpcode(MI.getOpcode()) != RISCV::VMV_V_V)
    return false;
  if (MI.getOperand(1).getReg().isValid())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 491-504: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // If the input was a pseudo with a policy operand, we can give it a tail
  // agnostic policy if MI's undef tail subsumes the input's.
  MachineInstr *Src = MRI->getVRegDef(MI.getOperand(2).getReg());
  if (Src && !Src->hasUnmodeledSideEffects() &&
      MRI->hasOneUse(MI.getOperand(2).getReg()) &&
      RISCVII::hasVLOp(Src->getDesc().TSFlags) &&
      RISCVII::hasVecPolicyOp(Src->getDesc().TSFlags) && hasSameEEW(MI, *Src)) {
    const MachineOperand &MIVL = MI.getOperand(3);
    const MachineOperand &SrcVL =
        Src->getOperand(RISCVII::getVLOpNum(Src->getDesc()));

    MachineOperand &SrcPolicy =
        Src->getOperand(RISCVII::getVecPolicyOpNum(Src->getDesc()));
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 505-530: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (RISCV::isVLKnownLE(MIVL, SrcVL))
      SrcPolicy.setImm(SrcPolicy.getImm() | RISCVVType::TAIL_AGNOSTIC);
  }

  MRI->constrainRegClass(MI.getOperand(2).getReg(),
                         MRI->getRegClass(MI.getOperand(0).getReg()));
  MRI->replaceRegWith(MI.getOperand(0).getReg(), MI.getOperand(2).getReg());
  MRI->clearKillFlags(MI.getOperand(2).getReg());
  MI.eraseFromParent();
  return true;
}

/// If a PseudoVMV_V_V is the only user of its input, fold its passthru and VL
/// into it.
///
/// %x = PseudoVADD_V_V_M1 %passthru, %a, %b, %vl1, sew, policy
/// %y = PseudoVMV_V_V_M1 %passthru, %x, %vl2, sew, policy
///    (where %vl1 <= %vl2)
///
/// ->
///
/// %y = PseudoVADD_V_V_M1 %passthru, %a, %b, vl1, sew, policy
bool RISCVVectorPeephole::foldVMV_V_V(MachineInstr &MI) {
  if (RISCV::getRVVMCOpcode(MI.getOpcode()) != RISCV::VMV_V_V)
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 531-546: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  MachineOperand &Passthru = MI.getOperand(1);

  if (!MRI->hasOneUse(MI.getOperand(2).getReg()))
    return false;

  MachineInstr *Src = MRI->getVRegDef(MI.getOperand(2).getReg());
  if (!Src || Src->hasUnmodeledSideEffects() ||
      Src->getParent() != MI.getParent() ||
      !RISCVII::isFirstDefTiedToFirstUse(Src->getDesc()) ||
      !RISCVII::hasVLOp(Src->getDesc().TSFlags))
    return false;

  // Src's dest needs to have the same EEW as MI's input.
  if (!hasSameEEW(MI, *Src))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 547-563: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  std::optional<std::pair<unsigned, unsigned>> NeedsCommute;

  // Src needs to have the same passthru as VMV_V_V
  MachineOperand &SrcPassthru = Src->getOperand(Src->getNumExplicitDefs());
  if (SrcPassthru.getReg().isValid() &&
      SrcPassthru.getReg() != Passthru.getReg()) {
    // If Src's passthru != Passthru, check if it uses Passthru in another
    // operand and try to commute it.
    int OtherIdx = Src->findRegisterUseOperandIdx(Passthru.getReg(), TRI);
    if (OtherIdx == -1)
      return false;
    unsigned OpIdx1 = OtherIdx;
    unsigned OpIdx2 = Src->getNumExplicitDefs();
    if (!TII->findCommutedOpIndices(*Src, OpIdx1, OpIdx2))
      return false;
    NeedsCommute = {OpIdx1, OpIdx2};
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 564-581: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Src VL will have already been reduced if legal by RISCVVLOptimizer,
  // so we don't need to handle a smaller source VL here.  However, the
  // user's VL may be larger
  MachineOperand &SrcVL = Src->getOperand(RISCVII::getVLOpNum(Src->getDesc()));
  if (!RISCV::isVLKnownLE(SrcVL, MI.getOperand(3)))
    return false;

  // If the new passthru doesn't dominate Src, try to move Src so it does.
  if (!ensureDominates(Passthru, *Src))
    return false;

  if (NeedsCommute) {
    auto [OpIdx1, OpIdx2] = *NeedsCommute;
    [[maybe_unused]] bool Commuted =
        TII->commuteInstruction(*Src, /*NewMI=*/false, OpIdx1, OpIdx2);
    assert(Commuted && "Failed to commute Src?");
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 582-599: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (SrcPassthru.getReg() != Passthru.getReg()) {
    SrcPassthru.setReg(Passthru.getReg());
    // If Src is masked then its passthru needs to be in VRNoV0.
    if (Passthru.getReg().isValid())
      MRI->constrainRegClass(
          Passthru.getReg(),
          TII->getRegClass(Src->getDesc(), SrcPassthru.getOperandNo()));
  }

  if (RISCVII::hasVecPolicyOp(Src->getDesc().TSFlags)) {
    // If MI was tail agnostic and the VL didn't increase, preserve it.
    int64_t Policy = RISCVVType::TAIL_UNDISTURBED_MASK_UNDISTURBED;
    if ((MI.getOperand(5).getImm() & RISCVVType::TAIL_AGNOSTIC) &&
        RISCV::isVLKnownLE(MI.getOperand(3), SrcVL))
      Policy |= RISCVVType::TAIL_AGNOSTIC;
    Src->getOperand(RISCVII::getVecPolicyOpNum(Src->getDesc())).setImm(Policy);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 600-627: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  MRI->constrainRegClass(Src->getOperand(0).getReg(),
                         MRI->getRegClass(MI.getOperand(0).getReg()));
  MRI->replaceRegWith(MI.getOperand(0).getReg(), Src->getOperand(0).getReg());
  MI.eraseFromParent();

  return true;
}

/// Try to fold away VMERGE_VVM instructions into their operands:
///
/// %true = PseudoVADD_VV ...
/// %x = PseudoVMERGE_VVM_M1 %false, %false, %true, %mask
/// ->
/// %x = PseudoVADD_VV_M1_MASK %false, ..., %mask
///
/// We can only fold if vmerge's passthru operand, vmerge's false operand and
/// %true's passthru operand (if it has one) are the same. This is because we
/// have to consolidate them into one passthru operand in the result.
///
/// If %true is masked, then we can use its mask instead of vmerge's if vmerge's
/// mask is all ones.
///
/// The resulting VL is the minimum of the two VLs.
///
/// The resulting policy is the effective policy the vmerge would have had,
/// i.e. whether or not it's passthru operand was implicit-def.
bool RISCVVectorPeephole::foldVMergeToMask(MachineInstr &MI) const {
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 628-646: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if (RISCV::getRVVMCOpcode(MI.getOpcode()) != RISCV::VMERGE_VVM)
    return false;

  // Collect chain of COPYs on True's result for later cleanup.
  SmallVector<MachineInstr *, 4> TrueCopies;
  Register PassthruReg = lookThruCopies(MI.getOperand(1).getReg());
  const MachineOperand &FalseOp = MI.getOperand(2);
  Register FalseReg = lookThruCopies(FalseOp.getReg());
  Register TrueReg = lookThruCopies(MI.getOperand(3).getReg(),
                                    /*OneUseOnly=*/true, &TrueCopies);
  if (!TrueReg.isVirtual() || !MRI->hasOneUse(TrueReg))
    return false;
  MachineInstr &True = *MRI->getUniqueVRegDef(TrueReg);
  if (True.getParent() != MI.getParent())
    return false;
  const MachineOperand &MaskOp = MI.getOperand(4);
  MachineInstr *Mask = MRI->getUniqueVRegDef(MaskOp.getReg());
  assert(Mask);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 647-660: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  const RISCV::RISCVMaskedPseudoInfo *Info =
      RISCV::lookupMaskedIntrinsicByUnmasked(True.getOpcode());
  if (!Info)
    return false;

  // If the EEW of True is different from vmerge's SEW, then we can't fold.
  if (!hasSameEEW(MI, True))
    return false;

  // We require that either passthru and false are the same, or that passthru
  // is undefined.
  if (PassthruReg && !(PassthruReg.isVirtual() && PassthruReg == FalseReg))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 661-680: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  std::optional<std::pair<unsigned, unsigned>> NeedsCommute;

  // If True has a passthru operand then it needs to be the same as vmerge's
  // False, since False will be used for the result's passthru operand.
  Register TruePassthru;
  if (RISCVII::isFirstDefTiedToFirstUse(True.getDesc()))
    TruePassthru =
        lookThruCopies(True.getOperand(True.getNumExplicitDefs()).getReg());
  if (TruePassthru && !(TruePassthru.isVirtual() && TruePassthru == FalseReg)) {
    // If True's passthru != False, check if it uses False in another operand
    // and try to commute it.
    int OtherIdx = True.findRegisterUseOperandIdx(FalseReg, TRI);
    if (OtherIdx == -1)
      return false;
    unsigned OpIdx1 = OtherIdx;
    unsigned OpIdx2 = True.getNumExplicitDefs();
    if (!TII->findCommutedOpIndices(True, OpIdx1, OpIdx2))
      return false;
    NeedsCommute = {OpIdx1, OpIdx2};
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 681-702: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Make sure it doesn't raise any observable fp exceptions, since changing the
  // active elements will affect how fflags is set.
  if (True.hasUnmodeledSideEffects() || True.mayRaiseFPException())
    return false;

  const MachineOperand &VMergeVL =
      MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
  const MachineOperand &TrueVL =
      True.getOperand(RISCVII::getVLOpNum(True.getDesc()));

  MachineOperand MinVL = MachineOperand::CreateImm(0);
  if (RISCV::isVLKnownLE(TrueVL, VMergeVL))
    MinVL = TrueVL;
  else if (RISCV::isVLKnownLE(VMergeVL, TrueVL))
    MinVL = VMergeVL;
  else if (!TruePassthru && !True.mayLoadOrStore())
    // If True's passthru is undef, we can use vmerge's vl.
    MinVL = VMergeVL;
  else
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 703-720: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  unsigned RVVTSFlags =
      TII->get(RISCV::getRVVMCOpcode(True.getOpcode())).TSFlags;
  if (RISCVII::elementsDependOnVL(RVVTSFlags) && !TrueVL.isIdenticalTo(MinVL))
    return false;
  if (RISCVII::elementsDependOnMask(RVVTSFlags) && !isAllOnesMask(Mask))
    return false;

  // Use a tumu policy, relaxing it to tail agnostic provided that the passthru
  // operand is undefined.
  //
  // However, if the VL became smaller than what the vmerge had originally, then
  // elements past VL that were previously in the vmerge's body will have moved
  // to the tail. In that case we always need to use tail undisturbed to
  // preserve them.
  uint64_t Policy = RISCVVType::TAIL_UNDISTURBED_MASK_UNDISTURBED;
  if (!PassthruReg && RISCV::isVLKnownLE(VMergeVL, MinVL))
    Policy |= RISCVVType::TAIL_AGNOSTIC;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 721-734: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  assert(RISCVII::hasVecPolicyOp(True.getDesc().TSFlags) &&
         "Foldable unmasked pseudo should have a policy op already");

  // Make sure both mask and false dominate True and its copies, otherwise move
  // down True so it does. VL will always dominate since if it's a register they
  // need to be the same.
  const MachineOperand *DomOp = &MaskOp;
  MachineInstr *False = MRI->getUniqueVRegDef(FalseReg);
  if (False && False->getParent() == Mask->getParent() &&
      dominates(Mask, False))
    DomOp = &FalseOp;
  if (!ensureDominates(*DomOp, True))
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 735-750: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (NeedsCommute) {
    auto [OpIdx1, OpIdx2] = *NeedsCommute;
    [[maybe_unused]] bool Commuted =
        TII->commuteInstruction(True, /*NewMI=*/false, OpIdx1, OpIdx2);
    assert(Commuted && "Failed to commute True?");
    Info = RISCV::lookupMaskedIntrinsicByUnmasked(True.getOpcode());
  }

  True.setDesc(TII->get(Info->MaskedPseudo));

  // Insert the mask operand.
  // TODO: Increment MaskOpIdx by number of explicit defs?
  True.insert(True.operands_begin() + Info->MaskOpIdx +
                  True.getNumExplicitDefs(),
              MachineOperand::CreateReg(MaskOp.getReg(), false));
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 751-765: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Update the passthru, AVL and policy.
  True.getOperand(True.getNumExplicitDefs()).setReg(FalseReg);
  True.removeOperand(RISCVII::getVLOpNum(True.getDesc()));
  True.insert(True.operands_begin() + RISCVII::getVLOpNum(True.getDesc()),
              MinVL);
  True.getOperand(RISCVII::getVecPolicyOpNum(True.getDesc())).setImm(Policy);

  MRI->replaceRegWith(True.getOperand(0).getReg(), MI.getOperand(0).getReg());
  // Now that True is masked, constrain its operands from vr -> vrnov0.
  for (MachineOperand &MO : True.explicit_operands()) {
    if (!MO.isReg() || !MO.getReg().isVirtual())
      continue;
    MRI->constrainRegClass(
        MO.getReg(), True.getRegClassConstraint(MO.getOperandNo(), TII, TRI));
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 766-781: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // We should clear the IsKill flag since we have a new use now.
  MRI->clearKillFlags(FalseReg);
  MI.eraseFromParent();

  // Cleanup all the COPYs on True's value. We have to manually do this because
  // sometimes sinking True causes these COPY to be invalid (use before define).
  for (MachineInstr *TrueCopy : TrueCopies)
    TrueCopy->eraseFromParent();

  return true;
}

bool RISCVVectorPeephole::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 782-796: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Skip if the vector extension is not enabled.
  ST = &MF.getSubtarget<RISCVSubtarget>();
  if (!ST->hasVInstructions())
    return false;

  TII = ST->getInstrInfo();
  MRI = &MF.getRegInfo();
  TRI = MRI->getTargetRegisterInfo();

  bool Changed = false;

  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : make_early_inc_range(MBB))
      Changed |= foldVMergeToMask(MI);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 797-810: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    for (MachineInstr &MI : make_early_inc_range(MBB)) {
      Changed |= convertToVLMAX(MI);
      Changed |= convertToUnmasked(MI);
      Changed |= convertToWholeRegister(MI);
      Changed |= convertAllOnesVMergeToVMv(MI);
      Changed |= convertSameMaskVMergeToVMv(MI);
      if (foldUndefPassthruVMV_V_V(MI)) {
        Changed |= true;
        continue; // MI is erased
      }
      Changed |= foldVMV_V_V(MI);
    }
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 811-816: Function implementation: llvm::createRISCVVectorPeepholePass / 函数实现：llvm::createRISCVVectorPeepholePass
```cpp
  return Changed;
}

FunctionPass *llvm::createRISCVVectorPeepholePass() {
  return new RISCVVectorPeephole();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **Vector extension support** / **向量扩展支持**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
