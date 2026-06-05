# RISCVVSETVLIInfoAnalysis.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVVSETVLIInfoAnalysis.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements analysis of vector configuration state tracked by vsetvli-related instructions. / 实现分析由 vsetvli 相关指令跟踪的向量配置状态。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===- RISCVVSETVLIInfoAnalysis.cpp - VSETVLI Info Analysis ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements an analysis of the vtype/vl information that is needed
// by RISCVInsertVSETVLI pass and others.
//
//===----------------------------------------------------------------------===//

#include "RISCVVSETVLIInfoAnalysis.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/LiveIntervals.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 18-32: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace llvm {
namespace RISCV {

/// Given a virtual register \p Reg, return the corresponding VNInfo for it.
/// This will return nullptr if the virtual register is an implicit_def or
/// if LiveIntervals is not available.
static VNInfo *getVNInfoFromReg(Register Reg, const MachineInstr &MI,
                                const LiveIntervals *LIS) {
  assert(Reg.isVirtual());
  if (!LIS)
    return nullptr;
  auto &LI = LIS->getInterval(Reg);
  SlotIndex SI = LIS->getSlotIndexes()->getInstructionIndex(MI);
  return LI.getVNInfoBefore(SI);
}
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 33-48: Function implementation: RISCVII::getVLOpNum / 函数实现：RISCVII::getVLOpNum
```cpp

static const MachineOperand &getVLOp(const MachineInstr &MI) {
  return MI.getOperand(RISCVII::getVLOpNum(MI.getDesc()));
}

static const MachineOperand &getSEWOp(const MachineInstr &MI) {
  return MI.getOperand(RISCVII::getSEWOpNum(MI.getDesc()));
}

static const MachineOperand &getVecPolicyOp(const MachineInstr &MI) {
  return MI.getOperand(RISCVII::getVecPolicyOpNum(MI.getDesc()));
}

static const MachineOperand &getTWidenOp(const MachineInstr &MI) {
  return MI.getOperand(RISCVII::getTWidenOpNum(MI.getDesc()));
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 49-76: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

/// Get the EEW for a load or store instruction.  Return std::nullopt if MI is
/// not a load or store which ignores SEW.
static std::optional<unsigned> getEEWForLoadStore(const MachineInstr &MI) {
  switch (RISCV::getRVVMCOpcode(MI.getOpcode())) {
  default:
    return std::nullopt;
  case RISCV::VLE8_V:
  case RISCV::VLSE8_V:
  case RISCV::VSE8_V:
  case RISCV::VSSE8_V:
    return 8;
  case RISCV::VLE16_V:
  case RISCV::VLSE16_V:
  case RISCV::VSE16_V:
  case RISCV::VSSE16_V:
    return 16;
  case RISCV::VLE32_V:
  case RISCV::VLSE32_V:
  case RISCV::VSE32_V:
  case RISCV::VSSE32_V:
    return 32;
  case RISCV::VLE64_V:
  case RISCV::VLSE64_V:
  case RISCV::VSE64_V:
  case RISCV::VSSE64_V:
    return 64;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 77-99: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

/// Return true if this is an operation on mask registers.  Note that
/// this includes both arithmetic/logical ops and load/store (vlm/vsm).
static bool isMaskRegOp(const MachineInstr &MI) {
  if (!RISCVII::hasSEWOp(MI.getDesc().TSFlags))
    return false;
  const unsigned Log2SEW = getSEWOp(MI).getImm();
  // A Log2SEW of 0 is an operation on mask registers only.
  return Log2SEW == 0;
}

/// Return true if the inactive elements in the result are entirely undefined.
/// Note that this is different from "agnostic" as defined by the vector
/// specification.  Agnostic requires each lane to either be undisturbed, or
/// take the value -1; no other value is allowed.
static bool hasUndefinedPassthru(const MachineInstr &MI) {
  unsigned UseOpIdx;
  if (!MI.isRegTiedToUseOperand(0, &UseOpIdx))
    // If there is no passthrough operand, then the pass through
    // lanes are undefined.
    return true;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 100-127: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // All undefined passthrus should be $noreg: see
  // RISCVDAGToDAGISel::doPeepholeNoRegPassThru
  const MachineOperand &UseMO = MI.getOperand(UseOpIdx);
  return !UseMO.getReg().isValid() || UseMO.isUndef();
}

static bool isLMUL1OrSmaller(RISCVVType::VLMUL LMUL) {
  auto [LMul, Fractional] = RISCVVType::decodeVLMUL(LMUL);
  return Fractional || LMul == 1;
}

/// Return true if moving from CurVType to NewVType is
/// indistinguishable from the perspective of an instruction (or set
/// of instructions) which use only the Used subfields and properties.
bool areCompatibleVTYPEs(uint64_t CurVType, uint64_t NewVType,
                         const DemandedFields &Used) {
  switch (Used.SEW) {
  case DemandedFields::SEWNone:
    break;
  case DemandedFields::SEWEqual:
    if (RISCVVType::getSEW(CurVType) != RISCVVType::getSEW(NewVType))
      return false;
    break;
  case DemandedFields::SEWGreaterThanOrEqual:
    if (RISCVVType::getSEW(NewVType) < RISCVVType::getSEW(CurVType))
      return false;
    break;
  case DemandedFields::SEWGreaterThanOrEqualAndLessThan64:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 128-145: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (RISCVVType::getSEW(NewVType) < RISCVVType::getSEW(CurVType) ||
        RISCVVType::getSEW(NewVType) >= 64)
      return false;
    break;
  }

  switch (Used.LMUL) {
  case DemandedFields::LMULNone:
    break;
  case DemandedFields::LMULEqual:
    if (RISCVVType::getVLMUL(CurVType) != RISCVVType::getVLMUL(NewVType))
      return false;
    break;
  case DemandedFields::LMULLessThanOrEqualToM1:
    if (!isLMUL1OrSmaller(RISCVVType::getVLMUL(NewVType)))
      return false;
    break;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 146-172: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (Used.SEWLMULRatio) {
    auto Ratio1 = RISCVVType::getSEWLMULRatio(RISCVVType::getSEW(CurVType),
                                              RISCVVType::getVLMUL(CurVType));
    auto Ratio2 = RISCVVType::getSEWLMULRatio(RISCVVType::getSEW(NewVType),
                                              RISCVVType::getVLMUL(NewVType));
    if (Ratio1 != Ratio2)
      return false;
  }

  if (Used.TailPolicy && RISCVVType::isTailAgnostic(CurVType) !=
                             RISCVVType::isTailAgnostic(NewVType))
    return false;
  if (Used.MaskPolicy && RISCVVType::isMaskAgnostic(CurVType) !=
                             RISCVVType::isMaskAgnostic(NewVType))
    return false;
  if (Used.TWiden && (RISCVVType::hasXSfmmWiden(CurVType) !=
                          RISCVVType::hasXSfmmWiden(NewVType) ||
                      (RISCVVType::hasXSfmmWiden(CurVType) &&
                       RISCVVType::getXSfmmWiden(CurVType) !=
                           RISCVVType::getXSfmmWiden(NewVType))))
    return false;
  if (Used.AltFmt &&
      RISCVVType::isAltFmt(CurVType) != RISCVVType::isAltFmt(NewVType))
    return false;
  return true;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 173-198: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

/// Return the fields and properties demanded by the provided instruction.
DemandedFields getDemanded(const MachineInstr &MI, const RISCVSubtarget *ST) {
  // This function works in coalesceVSETVLI too. We can still use the value of a
  // SEW, VL, or Policy operand even though it might not be the exact value in
  // the VL or VTYPE, since we only care about what the instruction originally
  // demanded.

  // Most instructions don't use any of these subfeilds.
  DemandedFields Res;
  // Start conservative if registers are used
  if (MI.isCall() || MI.isInlineAsm() ||
      MI.readsRegister(RISCV::VL, /*TRI=*/nullptr))
    Res.demandVL();
  if (MI.isCall() || MI.isInlineAsm() ||
      MI.readsRegister(RISCV::VTYPE, /*TRI=*/nullptr))
    Res.demandVTYPE();
  // Start conservative on the unlowered form too
  uint64_t TSFlags = MI.getDesc().TSFlags;
  if (RISCVII::hasSEWOp(TSFlags)) {
    Res.demandVTYPE();
    if (RISCVII::hasVLOp(TSFlags))
      if (const MachineOperand &VLOp = getVLOp(MI);
          !VLOp.isReg() || !VLOp.isUndef())
        Res.demandVL();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 199-213: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Behavior is independent of mask policy.
    if (!RISCVII::usesMaskPolicy(TSFlags))
      Res.MaskPolicy = false;
  }

  // Loads and stores with implicit EEW do not demand SEW or LMUL directly.
  // They instead demand the ratio of the two which is used in computing
  // EMUL, but which allows us the flexibility to change SEW and LMUL
  // provided we don't change the ratio.
  // Note: We assume that the instructions initial SEW is the EEW encoded
  // in the opcode.  This is asserted when constructing the VSETVLIInfo.
  if (RISCV::getEEWForLoadStore(MI)) {
    Res.SEW = DemandedFields::SEWNone;
    Res.LMUL = DemandedFields::LMULNone;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 214-228: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Store instructions don't use the policy fields.
  if (RISCVII::hasSEWOp(TSFlags) && MI.getNumExplicitDefs() == 0) {
    Res.TailPolicy = false;
    Res.MaskPolicy = false;
  }

  // If this is a mask reg operation, it only cares about VLMAX.
  // TODO: Possible extensions to this logic
  // * Probably ok if available VLMax is larger than demanded
  // * The policy bits can probably be ignored..
  if (isMaskRegOp(MI)) {
    Res.SEW = DemandedFields::SEWNone;
    Res.LMUL = DemandedFields::LMULNone;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 229-248: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // For vmv.s.x and vfmv.s.f, there are only two behaviors, VL = 0 and VL > 0.
  if (RISCVInstrInfo::isScalarInsertInstr(MI)) {
    Res.LMUL = DemandedFields::LMULNone;
    Res.SEWLMULRatio = false;
    Res.VLAny = false;
    // For vmv.s.x and vfmv.s.f, if the passthru is *undefined*, we don't
    // need to preserve any other bits and are thus compatible with any larger,
    // etype and can disregard policy bits.  Warning: It's tempting to try doing
    // this for any tail agnostic operation, but we can't as TA requires
    // tail lanes to either be the original value or -1.  We are writing
    // unknown bits to the lanes here.
    if (RISCV::hasUndefinedPassthru(MI)) {
      if (RISCVInstrInfo::isFloatScalarMoveOrScalarSplatInstr(MI) &&
          !ST->hasVInstructionsF64())
        Res.SEW = DemandedFields::SEWGreaterThanOrEqualAndLessThan64;
      else
        Res.SEW = DemandedFields::SEWGreaterThanOrEqual;
      Res.TailPolicy = false;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 249-276: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  // vmv.x.s, and vfmv.f.s are unconditional and ignore everything except SEW.
  if (RISCVInstrInfo::isScalarExtractInstr(MI)) {
    assert(!RISCVII::hasVLOp(TSFlags));
    Res.LMUL = DemandedFields::LMULNone;
    Res.SEWLMULRatio = false;
    Res.TailPolicy = false;
    Res.MaskPolicy = false;
  }

  if (RISCVII::hasVLOp(MI.getDesc().TSFlags)) {
    const MachineOperand &VLOp = getVLOp(MI);
    // A slidedown/slideup with an *undefined* passthru can freely clobber
    // elements not copied from the source vector (e.g. masked off, tail, or
    // slideup's prefix). Notes:
    // * We can't modify SEW here since the slide amount is in units of SEW.
    // * VL=1 is special only because we have existing support for zero vs
    //   non-zero VL.  We could generalize this if we had a VL > C predicate.
    // * The LMUL1 restriction is for machines whose latency may depend on LMUL.
    // * As above, this is only legal for tail "undefined" not "agnostic".
    // * We avoid increasing vl if the subtarget has +vl-dependent-latency
    if (RISCVInstrInfo::isVSlideInstr(MI) && VLOp.isImm() &&
        VLOp.getImm() == 1 && RISCV::hasUndefinedPassthru(MI) &&
        !ST->hasVLDependentLatency()) {
      Res.VLAny = false;
      Res.VLZeroness = true;
      Res.LMUL = DemandedFields::LMULLessThanOrEqualToM1;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 277-298: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      Res.TailPolicy = false;
    }

    // A tail undefined vmv.v.i/x or vfmv.v.f with VL=1 can be treated in the
    // same semantically as vmv.s.x.  This is particularly useful since we don't
    // have an immediate form of vmv.s.x, and thus frequently use vmv.v.i in
    // it's place. Since a splat is non-constant time in LMUL, we do need to be
    // careful to not increase the number of active vector registers (unlike for
    // vmv.s.x.)
    if (RISCVInstrInfo::isScalarSplatInstr(MI) && VLOp.isImm() &&
        VLOp.getImm() == 1 && RISCV::hasUndefinedPassthru(MI) &&
        !ST->hasVLDependentLatency()) {
      Res.LMUL = DemandedFields::LMULLessThanOrEqualToM1;
      Res.SEWLMULRatio = false;
      Res.VLAny = false;
      if (RISCVInstrInfo::isFloatScalarMoveOrScalarSplatInstr(MI) &&
          !ST->hasVInstructionsF64())
        Res.SEW = DemandedFields::SEWGreaterThanOrEqualAndLessThan64;
      else
        Res.SEW = DemandedFields::SEWGreaterThanOrEqual;
      Res.TailPolicy = false;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 299-314: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  // In §32.16.6, whole vector register moves have a dependency on SEW. At the
  // MIR level though we don't encode the element type, and it gives the same
  // result whatever the SEW may be.
  //
  // However it does need valid SEW, i.e. vill must be cleared. The entry to a
  // function, calls and inline assembly may all set it, so make sure we clear
  // it for whole register copies. Do this by leaving VILL demanded.
  if (RISCV::isVectorCopy(ST->getRegisterInfo(), MI)) {
    Res.LMUL = DemandedFields::LMULNone;
    Res.SEW = DemandedFields::SEWNone;
    Res.SEWLMULRatio = false;
    Res.TailPolicy = false;
    Res.MaskPolicy = false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 315-328: Function implementation: RISCVII::getAltFmtType / 函数实现：RISCVII::getAltFmtType
```cpp

  Res.AltFmt = RISCVII::getAltFmtType(MI.getDesc().TSFlags) !=
               RISCVII::AltFmtType::DontCare;
  Res.TWiden = RISCVII::hasTWidenOp(MI.getDesc().TSFlags) ||
               RISCVInstrInfo::isXSfmmVectorConfigInstr(MI);

  return Res;
}

bool VSETVLIInfo::hasCompatibleVTYPE(const DemandedFields &Used,
                                     const VSETVLIInfo &Require) const {
  return areCompatibleVTYPEs(Require.encodeVTYPE(), encodeVTYPE(), Used);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 329-347: Commentary and design intent / 注释与设计意图
```cpp
// If the AVL is defined by a vsetvli's output vl with the same VLMAX, we can
// replace the AVL operand with the AVL of the defining vsetvli. E.g.
//
// %vl = PseudoVSETVLI %avl:gpr, SEW=32, LMUL=M1
// $x0 = PseudoVSETVLI %vl:gpr, SEW=32, LMUL=M1
// ->
// %vl = PseudoVSETVLI %avl:gpr, SEW=32, LMUL=M1
// $x0 = PseudoVSETVLI %avl:gpr, SEW=32, LMUL=M1
void RISCVVSETVLIInfoAnalysis::forwardVSETVLIAVL(VSETVLIInfo &Info) const {
  if (!Info.hasAVLReg())
    return;
  const MachineInstr *DefMI = Info.getAVLDefMI(LIS);
  if (!DefMI || !RISCVInstrInfo::isVectorConfigInstr(*DefMI))
    return;
  VSETVLIInfo DefInstrInfo = getInfoForVSETVLI(*DefMI);
  if (!DefInstrInfo.hasSameVLMAX(Info))
    return;
  Info.setAVL(DefInstrInfo);
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 348-367: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Return a VSETVLIInfo representing the changes made by this VSETVLI or
// VSETIVLI instruction.
VSETVLIInfo
RISCVVSETVLIInfoAnalysis::getInfoForVSETVLI(const MachineInstr &MI) const {
  VSETVLIInfo NewInfo;
  if (MI.getOpcode() == RISCV::PseudoVSETIVLI) {
    NewInfo.setAVLImm(MI.getOperand(1).getImm());
  } else if (RISCVInstrInfo::isXSfmmVectorConfigTNInstr(MI)) {
    switch (MI.getOpcode()) {
    default:
      llvm_unreachable("Unexpected opcode");
    case RISCV::PseudoSF_VSETTNTX0:
      NewInfo.setAVLVLMAX();
      break;
    case RISCV::PseudoSF_VSETTNT:
      Register ATNReg = MI.getOperand(1).getReg();
      NewInfo.setAVLRegDef(getVNInfoFromReg(ATNReg, MI, LIS), ATNReg);
      break;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 368-381: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  } else {
    assert(MI.getOpcode() == RISCV::PseudoVSETVLI ||
           MI.getOpcode() == RISCV::PseudoVSETVLIX0);
    if (MI.getOpcode() == RISCV::PseudoVSETVLIX0)
      NewInfo.setAVLVLMAX();
    else if (MI.getOperand(1).isUndef())
      // Otherwise use an AVL of 1 to avoid depending on previous vl.
      NewInfo.setAVLImm(1);
    else {
      Register AVLReg = MI.getOperand(1).getReg();
      VNInfo *VNI = getVNInfoFromReg(AVLReg, MI, LIS);
      NewInfo.setAVLRegDef(VNI, AVLReg);
    }
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 382-397: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  NewInfo.setVTYPE(MI.getOperand(2).getImm());

  forwardVSETVLIAVL(NewInfo);

  return NewInfo;
}

static unsigned computeVLMAX(unsigned VLEN, unsigned SEW,
                             RISCVVType::VLMUL VLMul) {
  auto [LMul, Fractional] = RISCVVType::decodeVLMUL(VLMul);
  if (Fractional)
    VLEN = VLEN / LMul;
  else
    VLEN = VLEN * LMul;
  return VLEN / SEW;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 398-420: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

VSETVLIInfo
RISCVVSETVLIInfoAnalysis::computeInfoForInstr(const MachineInstr &MI) const {
  VSETVLIInfo InstrInfo;
  const uint64_t TSFlags = MI.getDesc().TSFlags;

  bool TailAgnostic = true;
  bool MaskAgnostic = true;
  if (!RISCV::hasUndefinedPassthru(MI)) {
    // Start with undisturbed.
    TailAgnostic = false;
    MaskAgnostic = false;

    // If there is a policy operand, use it.
    if (RISCVII::hasVecPolicyOp(TSFlags)) {
      const MachineOperand &Op = getVecPolicyOp(MI);
      uint64_t Policy = Op.getImm();
      assert(Policy <=
                 (RISCVVType::TAIL_AGNOSTIC | RISCVVType::MASK_AGNOSTIC) &&
             "Invalid Policy Value");
      TailAgnostic = Policy & RISCVVType::TAIL_AGNOSTIC;
      MaskAgnostic = Policy & RISCVVType::MASK_AGNOSTIC;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 421-435: Definitions and supporting logic / 定义与支撑逻辑
```cpp

    if (!RISCVII::usesMaskPolicy(TSFlags))
      MaskAgnostic = true;
  }

  RISCVVType::VLMUL VLMul = RISCVII::getLMul(TSFlags);

  bool AltFmt = RISCVII::getAltFmtType(TSFlags) == RISCVII::AltFmtType::AltFmt;
  InstrInfo.setAltFmt(AltFmt);

  unsigned Log2SEW = getSEWOp(MI).getImm();
  // A Log2SEW of 0 is an operation on mask registers only.
  unsigned SEW = Log2SEW ? 1 << Log2SEW : 8;
  assert(RISCVVType::isValidSEW(SEW) && "Unexpected SEW");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 436-449: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (RISCVII::hasTWidenOp(TSFlags)) {
    const MachineOperand &TWidenOp = getTWidenOp(MI);
    unsigned TWiden = TWidenOp.getImm();

    InstrInfo.setAVLVLMAX();
    if (RISCVII::hasVLOp(TSFlags)) {
      const MachineOperand &TNOp =
          MI.getOperand(RISCVII::getTNOpNum(MI.getDesc()));

      if (TNOp.getReg().isVirtual())
        InstrInfo.setAVLRegDef(getVNInfoFromReg(TNOp.getReg(), MI, LIS),
                               TNOp.getReg());
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 450-476: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    InstrInfo.setVTYPE(VLMul, SEW, TailAgnostic, MaskAgnostic, AltFmt, TWiden);

    return InstrInfo;
  }

  if (RISCVII::hasVLOp(TSFlags)) {
    const MachineOperand &VLOp = getVLOp(MI);
    if (VLOp.isImm()) {
      int64_t Imm = VLOp.getImm();
      // Convert the VLMax sentintel to X0 register.
      if (Imm == RISCV::VLMaxSentinel) {
        // If we know the exact VLEN, see if we can use the constant encoding
        // for the VLMAX instead.  This reduces register pressure slightly.
        const unsigned VLMAX = computeVLMAX(ST->getRealMaxVLen(), SEW, VLMul);
        if (ST->getRealMinVLen() == ST->getRealMaxVLen() && VLMAX <= 31)
          InstrInfo.setAVLImm(VLMAX);
        else
          InstrInfo.setAVLVLMAX();
      } else
        InstrInfo.setAVLImm(Imm);
    } else if (VLOp.isUndef()) {
      // Otherwise use an AVL of 1 to avoid depending on previous vl.
      InstrInfo.setAVLImm(1);
    } else {
      VNInfo *VNI = getVNInfoFromReg(VLOp.getReg(), MI, LIS);
      InstrInfo.setAVLRegDef(VNI, VLOp.getReg());
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 477-491: Header guard and interface framing / 头文件保护与接口框架
```cpp
  } else {
    assert(RISCVInstrInfo::isScalarExtractInstr(MI));
    // Pick a random value for state tracking purposes, will be ignored via
    // the demanded fields mechanism
    InstrInfo.setAVLImm(1);
  }
#ifndef NDEBUG
  if (std::optional<unsigned> EEW = RISCV::getEEWForLoadStore(MI)) {
    assert(SEW == EEW && "Initial SEW doesn't match expected EEW");
  }
#endif
  // TODO: Propagate the twiden from previous vtype for potential reuse.
  InstrInfo.setVTYPE(VLMul, SEW, TailAgnostic, MaskAgnostic, AltFmt,
                     /*TWiden*/ 0);
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 492-497: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  forwardVSETVLIAVL(InstrInfo);

  return InstrInfo;
}
} // namespace RISCV
} // namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

## Key Concepts / 关键概念
- **Vector configuration tracking** / **向量配置跟踪**

## Dependencies / 依赖关系
- `RISCVVSETVLIInfoAnalysis.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/LiveIntervals.h` — Directly referenced by this file. / 该文件直接引用的依赖。
