# R600InstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600InstrInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600InstrInfo for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600InstrInfo 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: File banner, includes, and setup
```cpp
//===-- R600InstrInfo.cpp - R600 Instruction Information ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// R600 Implementation of TargetInstrInfo.
//
//===----------------------------------------------------------------------===//

#include "R600InstrInfo.h"
#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600Defines.h"
#include "R600Subtarget.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/CodeGen/MachineFrameInfo.h"

using namespace llvm;

#define GET_INSTRINFO_CTOR_DTOR
#include "R600GenDFAPacketizer.inc"

#define GET_INSTRINFO_CTOR_DTOR
#define GET_INSTRMAP_INFO
#define GET_INSTRINFO_NAMED_OPS
#include "R600GenInstrInfo.inc"

R600InstrInfo::R600InstrInfo(const R600Subtarget &ST)
    : R600GenInstrInfo(ST, RI, -1, -1), RI(), ST(ST) {}

bool R600InstrInfo::isVector(const MachineInstr &MI) const {
  return get(MI.getOpcode()).TSFlags & R600_InstFlag::VECTOR;
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600InstrInfo::R600InstrInfo`, `R600InstrInfo::isVector`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600InstrInfo::R600InstrInfo`, `R600InstrInfo::isVector`。

### Lines 38-81: Implements R600InstrInfo::copyPhysReg
```cpp
void R600InstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MI,
                                const DebugLoc &DL, Register DestReg,
                                Register SrcReg, bool KillSrc,
                                bool RenamableDest, bool RenamableSrc) const {
  unsigned VectorComponents = 0;
  if ((R600::R600_Reg128RegClass.contains(DestReg) ||
      R600::R600_Reg128VerticalRegClass.contains(DestReg)) &&
      (R600::R600_Reg128RegClass.contains(SrcReg) ||
       R600::R600_Reg128VerticalRegClass.contains(SrcReg))) {
    VectorComponents = 4;
  } else if((R600::R600_Reg64RegClass.contains(DestReg) ||
            R600::R600_Reg64VerticalRegClass.contains(DestReg)) &&
            (R600::R600_Reg64RegClass.contains(SrcReg) ||
             R600::R600_Reg64VerticalRegClass.contains(SrcReg))) {
    VectorComponents = 2;
  }

  if (VectorComponents > 0) {
    for (unsigned I = 0; I < VectorComponents; I++) {
      unsigned SubRegIndex = R600RegisterInfo::getSubRegFromChannel(I);
      buildDefaultInstruction(MBB, MI, R600::MOV,
                              RI.getSubReg(DestReg, SubRegIndex),
                              RI.getSubReg(SrcReg, SubRegIndex))
                              .addReg(DestReg,
                                      RegState::Define | RegState::Implicit);
    }
  } else {
    MachineInstr *NewMI = buildDefaultInstruction(MBB, MI, R600::MOV,
                                                  DestReg, SrcReg);
    NewMI->getOperand(getOperandIdx(*NewMI, R600::OpName::src0))
                                    .setIsKill(KillSrc);
  }
}

/// \returns true if \p MBBI can be moved into a new basic.
bool R600InstrInfo::isLegalToSplitMBBAt(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator MBBI) const {
  for (const MachineOperand &MO : MBBI->all_uses())
    if (!MO.getReg().isVirtual() && RI.isPhysRegLiveAcrossClauses(MO.getReg()))
      return false;
  return true;
}

```
**EN:** This section contains concrete logic for R600InstrInfo::copyPhysReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::copyPhysReg`, `R600RegisterInfo::getSubRegFromChannel`, `R600InstrInfo::isLegalToSplitMBBAt`.
**CN:** 本节包含与 R600InstrInfo::copyPhysReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::copyPhysReg`, `R600RegisterInfo::getSubRegFromChannel`, `R600InstrInfo::isLegalToSplitMBBAt`。

### Lines 82-124: Implements R600InstrInfo::isMov
```cpp
bool R600InstrInfo::isMov(unsigned Opcode) const {
  switch(Opcode) {
  default:
    return false;
  case R600::MOV:
  case R600::MOV_IMM_F32:
  case R600::MOV_IMM_I32:
    return true;
  }
}

bool R600InstrInfo::isReductionOp(unsigned Opcode) const {
  return false;
}

bool R600InstrInfo::isCubeOp(unsigned Opcode) const {
  switch(Opcode) {
    default: return false;
    case R600::CUBE_r600_pseudo:
    case R600::CUBE_r600_real:
    case R600::CUBE_eg_pseudo:
    case R600::CUBE_eg_real:
      return true;
  }
}

bool R600InstrInfo::isALUInstr(unsigned Opcode) const {
  unsigned TargetFlags = get(Opcode).TSFlags;

  return (TargetFlags & R600_InstFlag::ALU_INST);
}

bool R600InstrInfo::hasInstrModifiers(unsigned Opcode) const {
  unsigned TargetFlags = get(Opcode).TSFlags;

  return ((TargetFlags & R600_InstFlag::OP1) |
          (TargetFlags & R600_InstFlag::OP2) |
          (TargetFlags & R600_InstFlag::OP3));
}

bool R600InstrInfo::isLDSInstr(unsigned Opcode) const {
  unsigned TargetFlags = get(Opcode).TSFlags;

```
**EN:** This section contains concrete logic for R600InstrInfo::isMov. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isMov`, `R600InstrInfo::isReductionOp`, `R600InstrInfo::isCubeOp`.
**CN:** 本节包含与 R600InstrInfo::isMov 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isMov`, `R600InstrInfo::isReductionOp`, `R600InstrInfo::isCubeOp`。

### Lines 125-165: Implements R600InstrInfo::isLDSRetInstr
```cpp
  return ((TargetFlags & R600_InstFlag::LDS_1A) |
          (TargetFlags & R600_InstFlag::LDS_1A1D) |
          (TargetFlags & R600_InstFlag::LDS_1A2D));
}

bool R600InstrInfo::isLDSRetInstr(unsigned Opcode) const {
  return isLDSInstr(Opcode) && getOperandIdx(Opcode, R600::OpName::dst) != -1;
}

bool R600InstrInfo::canBeConsideredALU(const MachineInstr &MI) const {
  if (isALUInstr(MI.getOpcode()))
    return true;
  if (isVector(MI) || isCubeOp(MI.getOpcode()))
    return true;
  switch (MI.getOpcode()) {
  case R600::PRED_X:
  case R600::INTERP_PAIR_XY:
  case R600::INTERP_PAIR_ZW:
  case R600::INTERP_VEC_LOAD:
  case R600::COPY:
  case R600::DOT_4:
    return true;
  default:
    return false;
  }
}

bool R600InstrInfo::isTransOnly(unsigned Opcode) const {
  if (ST.hasCaymanISA())
    return false;
  return (get(Opcode).getSchedClass() == R600::Sched::TransALU);
}

bool R600InstrInfo::isTransOnly(const MachineInstr &MI) const {
  return isTransOnly(MI.getOpcode());
}

bool R600InstrInfo::isVectorOnly(unsigned Opcode) const {
  return (get(Opcode).getSchedClass() == R600::Sched::VecALU);
}

```
**EN:** This section contains concrete logic for R600InstrInfo::isLDSRetInstr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isLDSRetInstr`, `R600InstrInfo::canBeConsideredALU`, `R600InstrInfo::isTransOnly`.
**CN:** 本节包含与 R600InstrInfo::isLDSRetInstr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isLDSRetInstr`, `R600InstrInfo::canBeConsideredALU`, `R600InstrInfo::isTransOnly`。

### Lines 166-208: Implements R600InstrInfo::isVectorOnly
```cpp
bool R600InstrInfo::isVectorOnly(const MachineInstr &MI) const {
  return isVectorOnly(MI.getOpcode());
}

bool R600InstrInfo::isExport(unsigned Opcode) const {
  return (get(Opcode).TSFlags & R600_InstFlag::IS_EXPORT);
}

bool R600InstrInfo::usesVertexCache(unsigned Opcode) const {
  return ST.hasVertexCache() && IS_VTX(get(Opcode));
}

bool R600InstrInfo::usesVertexCache(const MachineInstr &MI) const {
  const MachineFunction *MF = MI.getMF();
  return !AMDGPU::isCompute(MF->getFunction().getCallingConv()) &&
         usesVertexCache(MI.getOpcode());
}

bool R600InstrInfo::usesTextureCache(unsigned Opcode) const {
  return (!ST.hasVertexCache() && IS_VTX(get(Opcode))) || IS_TEX(get(Opcode));
}

bool R600InstrInfo::usesTextureCache(const MachineInstr &MI) const {
  const MachineFunction *MF = MI.getMF();
  return (AMDGPU::isCompute(MF->getFunction().getCallingConv()) &&
          usesVertexCache(MI.getOpcode())) ||
          usesTextureCache(MI.getOpcode());
}

bool R600InstrInfo::mustBeLastInClause(unsigned Opcode) const {
  switch (Opcode) {
  case R600::KILLGT:
  case R600::GROUP_BARRIER:
    return true;
  default:
    return false;
  }
}

bool R600InstrInfo::usesAddressRegister(MachineInstr &MI) const {
  return MI.findRegisterUseOperandIdx(R600::AR_X, &RI, false) != -1;
}

```
**EN:** This section contains concrete logic for R600InstrInfo::isVectorOnly. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isVectorOnly`, `R600InstrInfo::isExport`, `R600InstrInfo::usesVertexCache`.
**CN:** 本节包含与 R600InstrInfo::isVectorOnly 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isVectorOnly`, `R600InstrInfo::isExport`, `R600InstrInfo::usesVertexCache`。

### Lines 209-249: Implements R600InstrInfo::definesAddressRegister
```cpp
bool R600InstrInfo::definesAddressRegister(MachineInstr &MI) const {
  return MI.findRegisterDefOperandIdx(R600::AR_X, &RI, false, false) != -1;
}

bool R600InstrInfo::readsLDSSrcReg(const MachineInstr &MI) const {
  if (!isALUInstr(MI.getOpcode())) {
    return false;
  }
  for (const MachineOperand &MO : MI.all_uses())
    if (MO.getReg().isPhysical() &&
        R600::R600_LDS_SRC_REGRegClass.contains(MO.getReg()))
      return true;
  return false;
}

int R600InstrInfo::getSelIdx(unsigned Opcode, unsigned SrcIdx) const {
  static const R600::OpName SrcSelTable[][2] = {
      {R600::OpName::src0, R600::OpName::src0_sel},
      {R600::OpName::src1, R600::OpName::src1_sel},
      {R600::OpName::src2, R600::OpName::src2_sel},
      {R600::OpName::src0_X, R600::OpName::src0_sel_X},
      {R600::OpName::src0_Y, R600::OpName::src0_sel_Y},
      {R600::OpName::src0_Z, R600::OpName::src0_sel_Z},
      {R600::OpName::src0_W, R600::OpName::src0_sel_W},
      {R600::OpName::src1_X, R600::OpName::src1_sel_X},
      {R600::OpName::src1_Y, R600::OpName::src1_sel_Y},
      {R600::OpName::src1_Z, R600::OpName::src1_sel_Z},
      {R600::OpName::src1_W, R600::OpName::src1_sel_W}};

  for (const auto &Row : SrcSelTable) {
    if (getOperandIdx(Opcode, Row[0]) == (int)SrcIdx) {
      return getOperandIdx(Opcode, Row[1]);
    }
  }
  return -1;
}

SmallVector<std::pair<MachineOperand *, int64_t>, 3>
R600InstrInfo::getSrcs(MachineInstr &MI) const {
  SmallVector<std::pair<MachineOperand *, int64_t>, 3> Result;

```
**EN:** This section contains concrete logic for R600InstrInfo::definesAddressRegister. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::definesAddressRegister`, `R600InstrInfo::readsLDSSrcReg`, `R600InstrInfo::getSelIdx`.
**CN:** 本节包含与 R600InstrInfo::definesAddressRegister 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::definesAddressRegister`, `R600InstrInfo::readsLDSSrcReg`, `R600InstrInfo::getSelIdx`。

### Lines 250-280: Conditional logic and checks
```cpp
  if (MI.getOpcode() == R600::DOT_4) {
    static const R600::OpName OpTable[8][2] = {
        {R600::OpName::src0_X, R600::OpName::src0_sel_X},
        {R600::OpName::src0_Y, R600::OpName::src0_sel_Y},
        {R600::OpName::src0_Z, R600::OpName::src0_sel_Z},
        {R600::OpName::src0_W, R600::OpName::src0_sel_W},
        {R600::OpName::src1_X, R600::OpName::src1_sel_X},
        {R600::OpName::src1_Y, R600::OpName::src1_sel_Y},
        {R600::OpName::src1_Z, R600::OpName::src1_sel_Z},
        {R600::OpName::src1_W, R600::OpName::src1_sel_W},
    };

    for (const auto &Op : OpTable) {
      MachineOperand &MO = MI.getOperand(getOperandIdx(MI.getOpcode(), Op[0]));
      Register Reg = MO.getReg();
      if (Reg == R600::ALU_CONST) {
        MachineOperand &Sel =
            MI.getOperand(getOperandIdx(MI.getOpcode(), Op[1]));
        Result.push_back(std::pair(&MO, Sel.getImm()));
        continue;
      }
    }
    return Result;
  }

  static const R600::OpName OpTable[3][2] = {
      {R600::OpName::src0, R600::OpName::src0_sel},
      {R600::OpName::src1, R600::OpName::src1_sel},
      {R600::OpName::src2, R600::OpName::src2_sel},
  };

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::pair`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::pair`。

### Lines 281-305: Conditional logic and checks
```cpp
  for (const auto &Op : OpTable) {
    int SrcIdx = getOperandIdx(MI.getOpcode(), Op[0]);
    if (SrcIdx < 0)
      break;
    MachineOperand &MO = MI.getOperand(SrcIdx);
    Register Reg = MO.getReg();
    if (Reg == R600::ALU_CONST) {
      MachineOperand &Sel = MI.getOperand(getOperandIdx(MI.getOpcode(), Op[1]));
      Result.push_back(std::pair(&MO, Sel.getImm()));
      continue;
    }
    if (Reg == R600::ALU_LITERAL_X) {
      MachineOperand &Operand =
          MI.getOperand(getOperandIdx(MI.getOpcode(), R600::OpName::literal));
      if (Operand.isImm()) {
        Result.push_back(std::pair(&MO, Operand.getImm()));
        continue;
      }
      assert(Operand.isGlobal());
    }
    Result.push_back(std::pair(&MO, 0));
  }
  return Result;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::pair`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::pair`。

### Lines 306-338: Implements R600InstrInfo::ExtractSrcs
```cpp
std::vector<std::pair<int, unsigned>>
R600InstrInfo::ExtractSrcs(MachineInstr &MI,
                           const DenseMap<unsigned, unsigned> &PV,
                           unsigned &ConstCount) const {
  ConstCount = 0;
  const std::pair<int, unsigned> DummyPair(-1, 0);
  std::vector<std::pair<int, unsigned>> Result;
  unsigned i = 0;
  for (const auto &Src : getSrcs(MI)) {
    ++i;
    Register Reg = Src.first->getReg();
    int Index = RI.getEncodingValue(Reg) & 0xff;
    if (Reg == R600::OQAP) {
      Result.emplace_back(Index, 0U);
    }
    if (PV.contains(Reg)) {
      // 255 is used to tells its a PS/PV reg
      Result.emplace_back(255, 0U);
      continue;
    }
    if (Index > 127) {
      ConstCount++;
      Result.push_back(DummyPair);
      continue;
    }
    unsigned Chan = RI.getHWRegChan(Reg);
    Result.emplace_back(Index, Chan);
  }
  for (; i < 3; ++i)
    Result.push_back(DummyPair);
  return Result;
}

```
**EN:** This section contains concrete logic for R600InstrInfo::ExtractSrcs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::ExtractSrcs`.
**CN:** 本节包含与 R600InstrInfo::ExtractSrcs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::ExtractSrcs`。

### Lines 339-367: Defines Swizzle
```cpp
static std::vector<std::pair<int, unsigned>>
Swizzle(std::vector<std::pair<int, unsigned>> Src,
        R600InstrInfo::BankSwizzle Swz) {
  if (Src[0] == Src[1])
    Src[1].first = -1;
  switch (Swz) {
  case R600InstrInfo::ALU_VEC_012_SCL_210:
    break;
  case R600InstrInfo::ALU_VEC_021_SCL_122:
    std::swap(Src[1], Src[2]);
    break;
  case R600InstrInfo::ALU_VEC_102_SCL_221:
    std::swap(Src[0], Src[1]);
    break;
  case R600InstrInfo::ALU_VEC_120_SCL_212:
    std::swap(Src[0], Src[1]);
    std::swap(Src[0], Src[2]);
    break;
  case R600InstrInfo::ALU_VEC_201:
    std::swap(Src[0], Src[2]);
    std::swap(Src[0], Src[1]);
    break;
  case R600InstrInfo::ALU_VEC_210:
    std::swap(Src[0], Src[2]);
    break;
  }
  return Src;
}

```
**EN:** This section contains concrete logic for Swizzle. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::swap`.
**CN:** 本节包含与 Swizzle 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::swap`。

### Lines 368-391: Defines getTransSwizzle
```cpp
static unsigned getTransSwizzle(R600InstrInfo::BankSwizzle Swz, unsigned Op) {
  assert(Op < 3 && "Out of range swizzle index");
  switch (Swz) {
  case R600InstrInfo::ALU_VEC_012_SCL_210: {
    unsigned Cycles[3] = { 2, 1, 0};
    return Cycles[Op];
  }
  case R600InstrInfo::ALU_VEC_021_SCL_122: {
    unsigned Cycles[3] = { 1, 2, 2};
    return Cycles[Op];
  }
  case R600InstrInfo::ALU_VEC_120_SCL_212: {
    unsigned Cycles[3] = { 2, 1, 2};
    return Cycles[Op];
  }
  case R600InstrInfo::ALU_VEC_102_SCL_221: {
    unsigned Cycles[3] = { 2, 2, 1};
    return Cycles[Op];
  }
  default:
    llvm_unreachable("Wrong Swizzle for Trans Slot");
  }
}

```
**EN:** This section contains concrete logic for getTransSwizzle. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getTransSwizzle 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 392-435: Implements R600InstrInfo::isLegalUpTo
```cpp
/// returns how many MIs (whose inputs are represented by IGSrcs) can be packed
/// in the same Instruction Group while meeting read port limitations given a
/// Swz swizzle sequence.
unsigned  R600InstrInfo::isLegalUpTo(
    const std::vector<std::vector<std::pair<int, unsigned>>> &IGSrcs,
    const std::vector<R600InstrInfo::BankSwizzle> &Swz,
    const std::vector<std::pair<int, unsigned>> &TransSrcs,
    R600InstrInfo::BankSwizzle TransSwz) const {
  int Vector[4][3];
  memset(Vector, -1, sizeof(Vector));
  for (unsigned i = 0, e = IGSrcs.size(); i < e; i++) {
    const std::vector<std::pair<int, unsigned>> &Srcs =
        Swizzle(IGSrcs[i], Swz[i]);
    for (unsigned j = 0; j < 3; j++) {
      const std::pair<int, unsigned> &Src = Srcs[j];
      if (Src.first < 0 || Src.first == 255)
        continue;
      if (Src.first == GET_REG_INDEX(RI.getEncodingValue(R600::OQAP))) {
        if (Swz[i] != R600InstrInfo::ALU_VEC_012_SCL_210 &&
            Swz[i] != R600InstrInfo::ALU_VEC_021_SCL_122) {
            // The value from output queue A (denoted by register OQAP) can
            // only be fetched during the first cycle.
            return false;
        }
        // OQAP does not count towards the normal read port restrictions
        continue;
      }
      if (Vector[Src.second][j] < 0)
        Vector[Src.second][j] = Src.first;
      if (Vector[Src.second][j] != Src.first)
        return i;
    }
  }
  // Now check Trans Alu
  for (unsigned i = 0, e = TransSrcs.size(); i < e; ++i) {
    const std::pair<int, unsigned> &Src = TransSrcs[i];
    unsigned Cycle = getTransSwizzle(TransSwz, i);
    if (Src.first < 0)
      continue;
    if (Src.first == 255)
      continue;
    if (Vector[Src.second][Cycle] < 0)
      Vector[Src.second][Cycle] = Src.first;
    if (Vector[Src.second][Cycle] != Src.first)
```
**EN:** This section contains concrete logic for R600InstrInfo::isLegalUpTo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isLegalUpTo`.
**CN:** 本节包含与 R600InstrInfo::isLegalUpTo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isLegalUpTo`。

### Lines 436-477: Defines NextPossibleSolution
```cpp
      return IGSrcs.size() - 1;
  }
  return IGSrcs.size();
}

/// Given a swizzle sequence SwzCandidate and an index Idx, returns the next
/// (in lexicographic term) swizzle sequence assuming that all swizzles after
/// Idx can be skipped
static bool
NextPossibleSolution(
    std::vector<R600InstrInfo::BankSwizzle> &SwzCandidate,
    unsigned Idx) {
  assert(Idx < SwzCandidate.size());
  int ResetIdx = Idx;
  while (ResetIdx > -1 && SwzCandidate[ResetIdx] == R600InstrInfo::ALU_VEC_210)
    ResetIdx --;
  for (unsigned i = ResetIdx + 1, e = SwzCandidate.size(); i < e; i++) {
    SwzCandidate[i] = R600InstrInfo::ALU_VEC_012_SCL_210;
  }
  if (ResetIdx == -1)
    return false;
  int NextSwizzle = SwzCandidate[ResetIdx] + 1;
  SwzCandidate[ResetIdx] = (R600InstrInfo::BankSwizzle)NextSwizzle;
  return true;
}

/// Enumerate all possible Swizzle sequence to find one that can meet all
/// read port requirements.
bool R600InstrInfo::FindSwizzleForVectorSlot(
    const std::vector<std::vector<std::pair<int, unsigned>>> &IGSrcs,
    std::vector<R600InstrInfo::BankSwizzle> &SwzCandidate,
    const std::vector<std::pair<int, unsigned>> &TransSrcs,
    R600InstrInfo::BankSwizzle TransSwz) const {
  unsigned ValidUpTo = 0;
  do {
    ValidUpTo = isLegalUpTo(IGSrcs, SwzCandidate, TransSrcs, TransSwz);
    if (ValidUpTo == IGSrcs.size())
      return true;
  } while (NextPossibleSolution(SwzCandidate, ValidUpTo));
  return false;
}

```
**EN:** This section contains concrete logic for NextPossibleSolution. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::FindSwizzleForVectorSlot`.
**CN:** 本节包含与 NextPossibleSolution 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::FindSwizzleForVectorSlot`。

### Lines 478-521: Defines isConstCompatible
```cpp
/// Instructions in Trans slot can't read gpr at cycle 0 if they also read
/// a const, and can't read a gpr at cycle 1 if they read 2 const.
static bool
isConstCompatible(R600InstrInfo::BankSwizzle TransSwz,
                  const std::vector<std::pair<int, unsigned>> &TransOps,
                  unsigned ConstCount) {
  // TransALU can't read 3 constants
  if (ConstCount > 2)
    return false;
  for (unsigned i = 0, e = TransOps.size(); i < e; ++i) {
    const std::pair<int, unsigned> &Src = TransOps[i];
    unsigned Cycle = getTransSwizzle(TransSwz, i);
    if (Src.first < 0)
      continue;
    if (ConstCount > 0 && Cycle == 0)
      return false;
    if (ConstCount > 1 && Cycle == 1)
      return false;
  }
  return true;
}

bool
R600InstrInfo::fitsReadPortLimitations(const std::vector<MachineInstr *> &IG,
                                       const DenseMap<unsigned, unsigned> &PV,
                                       std::vector<BankSwizzle> &ValidSwizzle,
                                       bool isLastAluTrans)
    const {
  //Todo : support shared src0 - src1 operand

  std::vector<std::vector<std::pair<int, unsigned>>> IGSrcs;
  ValidSwizzle.clear();
  unsigned ConstCount;
  BankSwizzle TransBS = ALU_VEC_012_SCL_210;
  for (MachineInstr *MI : IG) {
    IGSrcs.push_back(ExtractSrcs(*MI, PV, ConstCount));
    unsigned Op = getOperandIdx(MI->getOpcode(), R600::OpName::bank_swizzle);
    ValidSwizzle.push_back(
        (R600InstrInfo::BankSwizzle)MI->getOperand(Op).getImm());
  }
  std::vector<std::pair<int, unsigned>> TransOps;
  if (!isLastAluTrans)
    return FindSwizzleForVectorSlot(IGSrcs, ValidSwizzle, TransOps, TransBS);

```
**EN:** This section contains concrete logic for isConstCompatible. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::fitsReadPortLimitations`.
**CN:** 本节包含与 isConstCompatible 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::fitsReadPortLimitations`。

### Lines 522-545: Implements std::move
```cpp
  TransOps = std::move(IGSrcs.back());
  IGSrcs.pop_back();
  ValidSwizzle.pop_back();

  static const R600InstrInfo::BankSwizzle TransSwz[] = {
    ALU_VEC_012_SCL_210,
    ALU_VEC_021_SCL_122,
    ALU_VEC_120_SCL_212,
    ALU_VEC_102_SCL_221
  };
  for (R600InstrInfo::BankSwizzle TransBS : TransSwz) {
    if (!isConstCompatible(TransBS, TransOps, ConstCount))
      continue;
    bool Result = FindSwizzleForVectorSlot(IGSrcs, ValidSwizzle, TransOps,
        TransBS);
    if (Result) {
      ValidSwizzle.push_back(TransBS);
      return true;
    }
  }

  return false;
}

```
**EN:** This section contains concrete logic for std::move. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::move`.
**CN:** 本节包含与 std::move 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::move`。

### Lines 546-579: Implements R600InstrInfo::fitsConstReadLimitations
```cpp
bool
R600InstrInfo::fitsConstReadLimitations(const std::vector<unsigned> &Consts)
    const {
  assert (Consts.size() <= 12 && "Too many operands in instructions group");
  unsigned Pair1 = 0, Pair2 = 0;
  for (unsigned Const : Consts) {
    unsigned ReadConstHalf = Const & 2;
    unsigned ReadConstIndex = Const & (~3);
    unsigned ReadHalfConst = ReadConstIndex | ReadConstHalf;
    if (!Pair1) {
      Pair1 = ReadHalfConst;
      continue;
    }
    if (Pair1 == ReadHalfConst)
      continue;
    if (!Pair2) {
      Pair2 = ReadHalfConst;
      continue;
    }
    if (Pair2 != ReadHalfConst)
      return false;
  }
  return true;
}

bool
R600InstrInfo::fitsConstReadLimitations(const std::vector<MachineInstr *> &MIs)
    const {
  std::vector<unsigned> Consts;
  SmallSet<int64_t, 4> Literals;
  for (MachineInstr *MI : MIs) {
    if (!isALUInstr(MI->getOpcode()))
      continue;

```
**EN:** This section contains concrete logic for R600InstrInfo::fitsConstReadLimitations. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::fitsConstReadLimitations`.
**CN:** 本节包含与 R600InstrInfo::fitsConstReadLimitations 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::fitsConstReadLimitations`。

### Lines 580-623: Switch-based control flow
```cpp
    for (const auto &Src : getSrcs(*MI)) {
      if (Src.first->getReg() == R600::ALU_LITERAL_X)
        Literals.insert(Src.second);
      if (Literals.size() > 4)
        return false;
      if (Src.first->getReg() == R600::ALU_CONST)
        Consts.push_back(Src.second);
      if (R600::R600_KC0RegClass.contains(Src.first->getReg()) ||
          R600::R600_KC1RegClass.contains(Src.first->getReg())) {
        unsigned Index = RI.getEncodingValue(Src.first->getReg()) & 0xff;
        unsigned Chan = RI.getHWRegChan(Src.first->getReg());
        Consts.push_back((Index << 2) | Chan);
      }
    }
  }
  return fitsConstReadLimitations(Consts);
}

DFAPacketizer *
R600InstrInfo::CreateTargetScheduleState(const TargetSubtargetInfo &STI) const {
  const InstrItineraryData *II = STI.getInstrItineraryData();
  return static_cast<const R600Subtarget &>(STI).createDFAPacketizer(II);
}

static bool
isPredicateSetter(unsigned Opcode) {
  switch (Opcode) {
  case R600::PRED_X:
    return true;
  default:
    return false;
  }
}

static MachineInstr *
findFirstPredicateSetterFrom(MachineBasicBlock &MBB,
                             MachineBasicBlock::iterator I) {
  while (I != MBB.begin()) {
    --I;
    MachineInstr &MI = *I;
    if (isPredicateSetter(MI.getOpcode()))
      return &MI;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600InstrInfo::CreateTargetScheduleState`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600InstrInfo::CreateTargetScheduleState`。

### Lines 624-665: Defines isJump
```cpp
  return nullptr;
}

static
bool isJump(unsigned Opcode) {
  return Opcode == R600::JUMP || Opcode == R600::JUMP_COND;
}

static bool isBranch(unsigned Opcode) {
  return Opcode == R600::BRANCH || Opcode == R600::BRANCH_COND_i32 ||
      Opcode == R600::BRANCH_COND_f32;
}

bool R600InstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                  MachineBasicBlock *&TBB,
                                  MachineBasicBlock *&FBB,
                                  SmallVectorImpl<MachineOperand> &Cond,
                                  bool AllowModify) const {
  // Most of the following comes from the ARM implementation of analyzeBranch

  // If the block has no terminators, it just falls into the block after it.
  MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
  if (I == MBB.end())
    return false;

  // R600::BRANCH* instructions are only available after isel and are not
  // handled
  if (isBranch(I->getOpcode()))
    return true;
  if (!isJump(I->getOpcode())) {
    return false;
  }

  // Remove successive JUMP
  while (I != MBB.begin() && std::prev(I)->getOpcode() == R600::JUMP) {
      MachineBasicBlock::iterator PriorI = std::prev(I);
      if (AllowModify)
        I->removeFromParent();
      I = PriorI;
  }
  MachineInstr &LastInst = *I;

```
**EN:** This section contains concrete logic for isJump. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::analyzeBranch`, `std::prev`.
**CN:** 本节包含与 isJump 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::analyzeBranch`, `std::prev`。

### Lines 666-708: Declares getOpcode
```cpp
  // If there is only one terminator instruction, process it.
  unsigned LastOpc = LastInst.getOpcode();
  if (I == MBB.begin() || !isJump((--I)->getOpcode())) {
    if (LastOpc == R600::JUMP) {
      TBB = LastInst.getOperand(0).getMBB();
      return false;
    }
    if (LastOpc == R600::JUMP_COND) {
      auto predSet = I;
      while (!isPredicateSetter(predSet->getOpcode())) {
        predSet = --I;
      }
      TBB = LastInst.getOperand(0).getMBB();
      Cond.push_back(predSet->getOperand(1));
      Cond.push_back(predSet->getOperand(2));
      Cond.push_back(MachineOperand::CreateReg(R600::PRED_SEL_ONE, false));
      return false;
    }
    return true;  // Can't handle indirect branch.
  }

  // Get the instruction before it if it is a terminator.
  MachineInstr &SecondLastInst = *I;
  unsigned SecondLastOpc = SecondLastInst.getOpcode();

  // If the block ends with a B and a Bcc, handle it.
  if (SecondLastOpc == R600::JUMP_COND && LastOpc == R600::JUMP) {
    auto predSet = --I;
    while (!isPredicateSetter(predSet->getOpcode())) {
      predSet = --I;
    }
    TBB = SecondLastInst.getOperand(0).getMBB();
    FBB = LastInst.getOperand(0).getMBB();
    Cond.push_back(predSet->getOperand(1));
    Cond.push_back(predSet->getOperand(2));
    Cond.push_back(MachineOperand::CreateReg(R600::PRED_SEL_ONE, false));
    return false;
  }

  // Otherwise, can't handle this.
  return true;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MachineOperand::CreateReg`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MachineOperand::CreateReg`。

### Lines 709-738: Defines FindLastAluClause
```cpp
static
MachineBasicBlock::iterator FindLastAluClause(MachineBasicBlock &MBB) {
  for (MachineBasicBlock::reverse_iterator It = MBB.rbegin(), E = MBB.rend();
      It != E; ++It) {
    if (It->getOpcode() == R600::CF_ALU ||
        It->getOpcode() == R600::CF_ALU_PUSH_BEFORE)
      return It.getReverse();
  }
  return MBB.end();
}

unsigned R600InstrInfo::insertBranch(MachineBasicBlock &MBB,
                                     MachineBasicBlock *TBB,
                                     MachineBasicBlock *FBB,
                                     ArrayRef<MachineOperand> Cond,
                                     const DebugLoc &DL,
                                     int *BytesAdded) const {
  assert(TBB && "insertBranch must not be told to insert a fallthrough");
  assert(!BytesAdded && "code size not handled");

  if (!FBB) {
    if (Cond.empty()) {
      BuildMI(&MBB, DL, get(R600::JUMP)).addMBB(TBB);
      return 1;
    }
    MachineInstr *PredSet = findFirstPredicateSetterFrom(MBB, MBB.end());
    assert(PredSet && "No previous predicate !");
    addFlag(*PredSet, 0, MO_FLAG_PUSH);
    PredSet->getOperand(2).setImm(Cond[1].getImm());

```
**EN:** This section contains concrete logic for FindLastAluClause. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::insertBranch`.
**CN:** 本节包含与 FindLastAluClause 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::insertBranch`。

### Lines 739-773: Declares findFirstPredicateSetterFrom
```cpp
    BuildMI(&MBB, DL, get(R600::JUMP_COND))
        .addMBB(TBB)
        .addReg(R600::PREDICATE_BIT, RegState::Kill);
    MachineBasicBlock::iterator CfAlu = FindLastAluClause(MBB);
    if (CfAlu == MBB.end())
      return 1;
    assert (CfAlu->getOpcode() == R600::CF_ALU);
    CfAlu->setDesc(get(R600::CF_ALU_PUSH_BEFORE));
    return 1;
  }
  MachineInstr *PredSet = findFirstPredicateSetterFrom(MBB, MBB.end());
  assert(PredSet && "No previous predicate !");
  addFlag(*PredSet, 0, MO_FLAG_PUSH);
  PredSet->getOperand(2).setImm(Cond[1].getImm());
  BuildMI(&MBB, DL, get(R600::JUMP_COND))
      .addMBB(TBB)
      .addReg(R600::PREDICATE_BIT, RegState::Kill);
  BuildMI(&MBB, DL, get(R600::JUMP)).addMBB(FBB);
  MachineBasicBlock::iterator CfAlu = FindLastAluClause(MBB);
  if (CfAlu == MBB.end())
    return 2;
  assert(CfAlu->getOpcode() == R600::CF_ALU);
  CfAlu->setDesc(get(R600::CF_ALU_PUSH_BEFORE));
  return 2;
}

unsigned R600InstrInfo::removeBranch(MachineBasicBlock &MBB,
                                     int *BytesRemoved) const {
  assert(!BytesRemoved && "code size not handled");

  // Note : we leave PRED* instructions there.
  // They may be needed when predicating instructions.

  MachineBasicBlock::iterator I = MBB.end();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600InstrInfo::removeBranch`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600InstrInfo::removeBranch`。

### Lines 774-797: Switch-based control flow
```cpp
  if (I == MBB.begin()) {
    return 0;
  }
  --I;
  switch (I->getOpcode()) {
  default:
    return 0;
  case R600::JUMP_COND: {
    MachineInstr *predSet = findFirstPredicateSetterFrom(MBB, I);
    clearFlag(*predSet, 0, MO_FLAG_PUSH);
    I->eraseFromParent();
    MachineBasicBlock::iterator CfAlu = FindLastAluClause(MBB);
    if (CfAlu == MBB.end())
      break;
    assert (CfAlu->getOpcode() == R600::CF_ALU_PUSH_BEFORE);
    CfAlu->setDesc(get(R600::CF_ALU));
    break;
  }
  case R600::JUMP:
    I->eraseFromParent();
    break;
  }
  I = MBB.end();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 798-838: Switch-based control flow
```cpp
  if (I == MBB.begin()) {
    return 1;
  }
  --I;
  switch (I->getOpcode()) {
    // FIXME: only one case??
  default:
    return 1;
  case R600::JUMP_COND: {
    MachineInstr *predSet = findFirstPredicateSetterFrom(MBB, I);
    clearFlag(*predSet, 0, MO_FLAG_PUSH);
    I->eraseFromParent();
    MachineBasicBlock::iterator CfAlu = FindLastAluClause(MBB);
    if (CfAlu == MBB.end())
      break;
    assert (CfAlu->getOpcode() == R600::CF_ALU_PUSH_BEFORE);
    CfAlu->setDesc(get(R600::CF_ALU));
    break;
  }
  case R600::JUMP:
    I->eraseFromParent();
    break;
  }
  return 2;
}

bool R600InstrInfo::isPredicated(const MachineInstr &MI) const {
  int idx = MI.findFirstPredOperandIdx();
  if (idx < 0)
    return false;

  Register Reg = MI.getOperand(idx).getReg();
  switch (Reg) {
  default: return false;
  case R600::PRED_SEL_ONE:
  case R600::PRED_SEL_ZERO:
  case R600::PREDICATE_BIT:
    return true;
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600InstrInfo::isPredicated`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600InstrInfo::isPredicated`。

### Lines 839-878: Implements R600InstrInfo::isPredicable
```cpp
bool R600InstrInfo::isPredicable(const MachineInstr &MI) const {
  // XXX: KILL* instructions can be predicated, but they must be the last
  // instruction in a clause, so this means any instructions after them cannot
  // be predicated.  Until we have proper support for instruction clauses in the
  // backend, we will mark KILL* instructions as unpredicable.

  if (MI.getOpcode() == R600::KILLGT)
    return false;
  if (MI.getOpcode() == R600::CF_ALU) {
    // If the clause start in the middle of MBB then the MBB has more
    // than a single clause, unable to predicate several clauses.
    if (MI.getParent()->begin() != MachineBasicBlock::const_iterator(MI))
      return false;
    // TODO: We don't support KC merging atm
    return MI.getOperand(3).getImm() == 0 && MI.getOperand(4).getImm() == 0;
  }
  if (isVector(MI))
    return false;
  return TargetInstrInfo::isPredicable(MI);
}

bool
R600InstrInfo::isProfitableToIfCvt(MachineBasicBlock &MBB,
                                   unsigned NumCycles,
                                   unsigned ExtraPredCycles,
                                   BranchProbability Probability) const{
  return true;
}

bool
R600InstrInfo::isProfitableToIfCvt(MachineBasicBlock &TMBB,
                                   unsigned NumTCycles,
                                   unsigned ExtraTCycles,
                                   MachineBasicBlock &FMBB,
                                   unsigned NumFCycles,
                                   unsigned ExtraFCycles,
                                   BranchProbability Probability) const {
  return true;
}

```
**EN:** This section contains concrete logic for R600InstrInfo::isPredicable. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isPredicable`, `MachineBasicBlock::const_iterator`, `TargetInstrInfo::isPredicable`.
**CN:** 本节包含与 R600InstrInfo::isPredicable 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isPredicable`, `MachineBasicBlock::const_iterator`, `TargetInstrInfo::isPredicable`。

### Lines 879-912: Implements R600InstrInfo::isProfitableToDupForIfCvt
```cpp
bool
R600InstrInfo::isProfitableToDupForIfCvt(MachineBasicBlock &MBB,
                                         unsigned NumCycles,
                                         BranchProbability Probability)
                                         const {
  return true;
}

bool
R600InstrInfo::isProfitableToUnpredicate(MachineBasicBlock &TMBB,
                                         MachineBasicBlock &FMBB) const {
  return false;
}

bool
R600InstrInfo::reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const {
  MachineOperand &MO = Cond[1];
  switch (MO.getImm()) {
  case R600::PRED_SETE_INT:
    MO.setImm(R600::PRED_SETNE_INT);
    break;
  case R600::PRED_SETNE_INT:
    MO.setImm(R600::PRED_SETE_INT);
    break;
  case R600::PRED_SETE:
    MO.setImm(R600::PRED_SETNE);
    break;
  case R600::PRED_SETNE:
    MO.setImm(R600::PRED_SETE);
    break;
  default:
    return true;
  }

```
**EN:** This section contains concrete logic for R600InstrInfo::isProfitableToDupForIfCvt. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::isProfitableToDupForIfCvt`, `R600InstrInfo::isProfitableToUnpredicate`, `R600InstrInfo::reverseBranchCondition`.
**CN:** 本节包含与 R600InstrInfo::isProfitableToDupForIfCvt 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::isProfitableToDupForIfCvt`, `R600InstrInfo::isProfitableToUnpredicate`, `R600InstrInfo::reverseBranchCondition`。

### Lines 913-955: Switch-based control flow
```cpp
  MachineOperand &MO2 = Cond[2];
  switch (MO2.getReg()) {
  case R600::PRED_SEL_ZERO:
    MO2.setReg(R600::PRED_SEL_ONE);
    break;
  case R600::PRED_SEL_ONE:
    MO2.setReg(R600::PRED_SEL_ZERO);
    break;
  default:
    return true;
  }
  return false;
}

bool R600InstrInfo::ClobbersPredicate(MachineInstr &MI,
                                      std::vector<MachineOperand> &Pred,
                                      bool SkipDead) const {
  return isPredicateSetter(MI.getOpcode());
}

bool R600InstrInfo::PredicateInstruction(MachineInstr &MI,
                                         ArrayRef<MachineOperand> Pred) const {
  int PIdx = MI.findFirstPredOperandIdx();

  if (MI.getOpcode() == R600::CF_ALU) {
    MI.getOperand(8).setImm(0);
    return true;
  }

  if (MI.getOpcode() == R600::DOT_4) {
    MI.getOperand(getOperandIdx(MI, R600::OpName::pred_sel_X))
        .setReg(Pred[2].getReg());
    MI.getOperand(getOperandIdx(MI, R600::OpName::pred_sel_Y))
        .setReg(Pred[2].getReg());
    MI.getOperand(getOperandIdx(MI, R600::OpName::pred_sel_Z))
        .setReg(Pred[2].getReg());
    MI.getOperand(getOperandIdx(MI, R600::OpName::pred_sel_W))
        .setReg(Pred[2].getReg());
    MachineInstrBuilder MIB(*MI.getMF(), MI);
    MIB.addReg(R600::PREDICATE_BIT, RegState::Implicit);
    return true;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600InstrInfo::ClobbersPredicate`, `R600InstrInfo::PredicateInstruction`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600InstrInfo::ClobbersPredicate`, `R600InstrInfo::PredicateInstruction`。

### Lines 956-984: Implements R600InstrInfo::getPredicationCost
```cpp
  if (PIdx != -1) {
    MachineOperand &PMO = MI.getOperand(PIdx);
    PMO.setReg(Pred[2].getReg());
    MachineInstrBuilder MIB(*MI.getMF(), MI);
    MIB.addReg(R600::PREDICATE_BIT, RegState::Implicit);
    return true;
  }

  return false;
}

unsigned int R600InstrInfo::getPredicationCost(const MachineInstr &) const {
  return 2;
}

unsigned int R600InstrInfo::getInstrLatency(const InstrItineraryData *ItinData,
                                            const MachineInstr &,
                                            unsigned *PredCost) const {
  if (PredCost)
    *PredCost = 2;
  return 2;
}

unsigned R600InstrInfo::calculateIndirectAddress(unsigned RegIndex,
                                                   unsigned Channel) const {
  assert(Channel == 0);
  return RegIndex;
}

```
**EN:** This section contains concrete logic for R600InstrInfo::getPredicationCost. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::getPredicationCost`, `R600InstrInfo::getInstrLatency`, `R600InstrInfo::calculateIndirectAddress`.
**CN:** 本节包含与 R600InstrInfo::getPredicationCost 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::getPredicationCost`, `R600InstrInfo::getInstrLatency`, `R600InstrInfo::calculateIndirectAddress`。

### Lines 985-1028: Implements R600InstrInfo::expandPostRAPseudo
```cpp
bool R600InstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  default: {
    MachineBasicBlock *MBB = MI.getParent();
    int OffsetOpIdx =
        R600::getNamedOperandIdx(MI.getOpcode(), R600::OpName::addr);
    // addr is a custom operand with multiple MI operands, and only the
    // first MI operand is given a name.
    int RegOpIdx = OffsetOpIdx + 1;
    int ChanOpIdx =
        R600::getNamedOperandIdx(MI.getOpcode(), R600::OpName::chan);
    if (isRegisterLoad(MI)) {
      int DstOpIdx =
          R600::getNamedOperandIdx(MI.getOpcode(), R600::OpName::dst);
      unsigned RegIndex = MI.getOperand(RegOpIdx).getImm();
      unsigned Channel = MI.getOperand(ChanOpIdx).getImm();
      unsigned Address = calculateIndirectAddress(RegIndex, Channel);
      Register OffsetReg = MI.getOperand(OffsetOpIdx).getReg();
      if (OffsetReg == R600::INDIRECT_BASE_ADDR) {
        buildMovInstr(MBB, MI, MI.getOperand(DstOpIdx).getReg(),
                      getIndirectAddrRegClass()->getRegister(Address));
      } else {
        buildIndirectRead(MBB, MI, MI.getOperand(DstOpIdx).getReg(), Address,
                          OffsetReg);
      }
    } else if (isRegisterStore(MI)) {
      int ValOpIdx =
          R600::getNamedOperandIdx(MI.getOpcode(), R600::OpName::val);
      unsigned RegIndex = MI.getOperand(RegOpIdx).getImm();
      unsigned Channel = MI.getOperand(ChanOpIdx).getImm();
      unsigned Address = calculateIndirectAddress(RegIndex, Channel);
      Register OffsetReg = MI.getOperand(OffsetOpIdx).getReg();
      if (OffsetReg == R600::INDIRECT_BASE_ADDR) {
        buildMovInstr(MBB, MI, getIndirectAddrRegClass()->getRegister(Address),
                      MI.getOperand(ValOpIdx).getReg());
      } else {
        buildIndirectWrite(MBB, MI, MI.getOperand(ValOpIdx).getReg(),
                           calculateIndirectAddress(RegIndex, Channel),
                           OffsetReg);
      }
    } else {
      return false;
    }

```
**EN:** This section contains concrete logic for R600InstrInfo::expandPostRAPseudo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::expandPostRAPseudo`, `R600::getNamedOperandIdx`.
**CN:** 本节包含与 R600InstrInfo::expandPostRAPseudo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::expandPostRAPseudo`, `R600::getNamedOperandIdx`。

### Lines 1029-1071: Conditional logic and checks
```cpp
    MBB->erase(MI);
    return true;
  }
  case R600::R600_EXTRACT_ELT_V2:
  case R600::R600_EXTRACT_ELT_V4:
    buildIndirectRead(MI.getParent(), MI, MI.getOperand(0).getReg(),
                      RI.getHWRegIndex(MI.getOperand(1).getReg()), //  Address
                      MI.getOperand(2).getReg(),
                      RI.getHWRegChan(MI.getOperand(1).getReg()));
    break;
  case R600::R600_INSERT_ELT_V2:
  case R600::R600_INSERT_ELT_V4:
    buildIndirectWrite(MI.getParent(), MI, MI.getOperand(2).getReg(), // Value
                       RI.getHWRegIndex(MI.getOperand(1).getReg()),   // Address
                       MI.getOperand(3).getReg(),                     // Offset
                       RI.getHWRegChan(MI.getOperand(1).getReg()));   // Channel
    break;
  }
  MI.eraseFromParent();
  return true;
}

void R600InstrInfo::reserveIndirectRegisters(BitVector &Reserved,
                                             const MachineFunction &MF,
                                             const R600RegisterInfo &TRI) const {
  const R600Subtarget &ST = MF.getSubtarget<R600Subtarget>();
  const R600FrameLowering *TFL = ST.getFrameLowering();

  unsigned StackWidth = TFL->getStackWidth(MF);
  int End = getIndirectIndexEnd(MF);

  if (End == -1)
    return;

  for (int Index = getIndirectIndexBegin(MF); Index <= End; ++Index) {
    for (unsigned Chan = 0; Chan < StackWidth; ++Chan) {
      MCRegister Reg =
          R600::R600_TReg32RegClass.getRegister((4 * Index) + Chan);
      TRI.reserveRegisterTuples(Reserved, Reg);
    }
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `R600InstrInfo::reserveIndirectRegisters`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`R600InstrInfo::reserveIndirectRegisters`。

### Lines 1072-1114: Implements R600InstrInfo::getIndirectAddrRegClass
```cpp
const TargetRegisterClass *R600InstrInfo::getIndirectAddrRegClass() const {
  return &R600::R600_TReg32_XRegClass;
}

MachineInstrBuilder R600InstrInfo::buildIndirectWrite(MachineBasicBlock *MBB,
                                       MachineBasicBlock::iterator I,
                                       unsigned ValueReg, unsigned Address,
                                       unsigned OffsetReg) const {
  return buildIndirectWrite(MBB, I, ValueReg, Address, OffsetReg, 0);
}

MachineInstrBuilder R600InstrInfo::buildIndirectWrite(MachineBasicBlock *MBB,
                                       MachineBasicBlock::iterator I,
                                       unsigned ValueReg, unsigned Address,
                                       unsigned OffsetReg,
                                       unsigned AddrChan) const {
  MCRegister AddrReg;
  switch (AddrChan) {
    default: llvm_unreachable("Invalid Channel");
    case 0: AddrReg = R600::R600_AddrRegClass.getRegister(Address); break;
    case 1: AddrReg = R600::R600_Addr_YRegClass.getRegister(Address); break;
    case 2: AddrReg = R600::R600_Addr_ZRegClass.getRegister(Address); break;
    case 3: AddrReg = R600::R600_Addr_WRegClass.getRegister(Address); break;
  }
  MachineInstr *MOVA = buildDefaultInstruction(*MBB, I, R600::MOVA_INT_eg,
                                               R600::AR_X, OffsetReg);
  setImmOperand(*MOVA, R600::OpName::write, 0);

  MachineInstrBuilder Mov = buildDefaultInstruction(*MBB, I, R600::MOV,
                                      AddrReg, ValueReg)
                                      .addReg(R600::AR_X,
                                           RegState::Implicit | RegState::Kill);
  setImmOperand(*Mov, R600::OpName::dst_rel, 1);
  return Mov;
}

MachineInstrBuilder R600InstrInfo::buildIndirectRead(MachineBasicBlock *MBB,
                                       MachineBasicBlock::iterator I,
                                       unsigned ValueReg, unsigned Address,
                                       unsigned OffsetReg) const {
  return buildIndirectRead(MBB, I, ValueReg, Address, OffsetReg, 0);
}

```
**EN:** This section contains concrete logic for R600InstrInfo::getIndirectAddrRegClass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::getIndirectAddrRegClass`, `R600InstrInfo::buildIndirectWrite`, `R600InstrInfo::buildIndirectRead`.
**CN:** 本节包含与 R600InstrInfo::getIndirectAddrRegClass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::getIndirectAddrRegClass`, `R600InstrInfo::buildIndirectWrite`, `R600InstrInfo::buildIndirectRead`。

### Lines 1115-1154: Implements R600InstrInfo::buildIndirectRead
```cpp
MachineInstrBuilder R600InstrInfo::buildIndirectRead(MachineBasicBlock *MBB,
                                       MachineBasicBlock::iterator I,
                                       unsigned ValueReg, unsigned Address,
                                       unsigned OffsetReg,
                                       unsigned AddrChan) const {
  MCRegister AddrReg;
  switch (AddrChan) {
    default: llvm_unreachable("Invalid Channel");
    case 0: AddrReg = R600::R600_AddrRegClass.getRegister(Address); break;
    case 1: AddrReg = R600::R600_Addr_YRegClass.getRegister(Address); break;
    case 2: AddrReg = R600::R600_Addr_ZRegClass.getRegister(Address); break;
    case 3: AddrReg = R600::R600_Addr_WRegClass.getRegister(Address); break;
  }
  MachineInstr *MOVA = buildDefaultInstruction(*MBB, I, R600::MOVA_INT_eg,
                                                       R600::AR_X,
                                                       OffsetReg);
  setImmOperand(*MOVA, R600::OpName::write, 0);
  MachineInstrBuilder Mov = buildDefaultInstruction(*MBB, I, R600::MOV,
                                      ValueReg,
                                      AddrReg)
                                      .addReg(R600::AR_X,
                                           RegState::Implicit | RegState::Kill);
  setImmOperand(*Mov, R600::OpName::src0_rel, 1);

  return Mov;
}

int R600InstrInfo::getIndirectIndexBegin(const MachineFunction &MF) const {
  const MachineRegisterInfo &MRI = MF.getRegInfo();
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  int Offset = -1;

  if (MFI.getNumObjects() == 0) {
    return -1;
  }

  if (MRI.livein_empty()) {
    return 0;
  }

```
**EN:** This section contains concrete logic for R600InstrInfo::buildIndirectRead. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::buildIndirectRead`, `R600InstrInfo::getIndirectIndexBegin`.
**CN:** 本节包含与 R600InstrInfo::buildIndirectRead 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::buildIndirectRead`, `R600InstrInfo::getIndirectIndexBegin`。

### Lines 1155-1195: Declares getIndirectAddrRegClass
```cpp
  const TargetRegisterClass *IndirectRC = getIndirectAddrRegClass();
  for (std::pair<MCRegister, Register> LI : MRI.liveins()) {
    Register Reg = LI.first;
    if (Reg.isVirtual() || !IndirectRC->contains(Reg))
      continue;

    unsigned RegIndex;
    unsigned RegEnd;
    for (RegIndex = 0, RegEnd = IndirectRC->getNumRegs(); RegIndex != RegEnd;
                                                          ++RegIndex) {
      if (IndirectRC->getRegister(RegIndex) == (unsigned)Reg)
        break;
    }
    Offset = std::max(Offset, (int)RegIndex);
  }

  return Offset + 1;
}

int R600InstrInfo::getIndirectIndexEnd(const MachineFunction &MF) const {
  int Offset = 0;
  const MachineFrameInfo &MFI = MF.getFrameInfo();

  // Variable sized objects are not supported
  if (MFI.hasVarSizedObjects()) {
    return -1;
  }

  if (MFI.getNumObjects() == 0) {
    return -1;
  }

  const R600Subtarget &ST = MF.getSubtarget<R600Subtarget>();
  const R600FrameLowering *TFL = ST.getFrameLowering();

  Register IgnoredFrameReg;
  Offset = TFL->getFrameIndexReference(MF, -1, IgnoredFrameReg).getFixed();

  return getIndirectIndexBegin(MF) + Offset;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `std::max`, `R600InstrInfo::getIndirectIndexEnd`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`std::max`, `R600InstrInfo::getIndirectIndexEnd`。

### Lines 1196-1237: Implements R600InstrInfo::getMaxAlusPerClause
```cpp
unsigned R600InstrInfo::getMaxAlusPerClause() const {
  return 115;
}

MachineInstrBuilder R600InstrInfo::buildDefaultInstruction(MachineBasicBlock &MBB,
                                                  MachineBasicBlock::iterator I,
                                                  unsigned Opcode,
                                                  unsigned DstReg,
                                                  unsigned Src0Reg,
                                                  unsigned Src1Reg) const {
  MachineInstrBuilder MIB = BuildMI(MBB, I, MBB.findDebugLoc(I), get(Opcode),
    DstReg);           // $dst

  if (Src1Reg) {
    MIB.addImm(0)     // $update_exec_mask
       .addImm(0);    // $update_predicate
  }
  MIB.addImm(1)        // $write
     .addImm(0)        // $omod
     .addImm(0)        // $dst_rel
     .addImm(0)        // $dst_clamp
     .addReg(Src0Reg)  // $src0
     .addImm(0)        // $src0_neg
     .addImm(0)        // $src0_rel
     .addImm(0)        // $src0_abs
     .addImm(-1);       // $src0_sel

  if (Src1Reg) {
    MIB.addReg(Src1Reg) // $src1
       .addImm(0)       // $src1_neg
       .addImm(0)       // $src1_rel
       .addImm(0)       // $src1_abs
       .addImm(-1);      // $src1_sel
  }

  //XXX: The r600g finalizer expects this to be 1, once we've moved the
  //scheduling to the backend, we can change the default to 0.
  MIB.addImm(1)        // $last
      .addReg(R600::PRED_SEL_OFF) // $pred_sel
      .addImm(0)         // $literal
      .addImm(0);        // $bank_swizzle

```
**EN:** This section contains concrete logic for R600InstrInfo::getMaxAlusPerClause. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::getMaxAlusPerClause`, `R600InstrInfo::buildDefaultInstruction`.
**CN:** 本节包含与 R600InstrInfo::getMaxAlusPerClause 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::getMaxAlusPerClause`, `R600InstrInfo::buildDefaultInstruction`。

### Lines 1238-1273: Preprocessor guards and macros
```cpp
  return MIB;
}

#define OPERAND_CASE(Label)                                                    \
  case Label: {                                                                \
    static const R600::OpName Ops[] = {Label##_X, Label##_Y, Label##_Z,        \
                                       Label##_W};                             \
    return Ops[Slot];                                                          \
  }

static R600::OpName getSlotedOps(R600::OpName Op, unsigned Slot) {
  switch (Op) {
  OPERAND_CASE(R600::OpName::update_exec_mask)
  OPERAND_CASE(R600::OpName::update_pred)
  OPERAND_CASE(R600::OpName::write)
  OPERAND_CASE(R600::OpName::omod)
  OPERAND_CASE(R600::OpName::dst_rel)
  OPERAND_CASE(R600::OpName::clamp)
  OPERAND_CASE(R600::OpName::src0)
  OPERAND_CASE(R600::OpName::src0_neg)
  OPERAND_CASE(R600::OpName::src0_rel)
  OPERAND_CASE(R600::OpName::src0_abs)
  OPERAND_CASE(R600::OpName::src0_sel)
  OPERAND_CASE(R600::OpName::src1)
  OPERAND_CASE(R600::OpName::src1_neg)
  OPERAND_CASE(R600::OpName::src1_rel)
  OPERAND_CASE(R600::OpName::src1_abs)
  OPERAND_CASE(R600::OpName::src1_sel)
  OPERAND_CASE(R600::OpName::pred_sel)
  default:
    llvm_unreachable("Wrong Operand");
  }
}

#undef OPERAND_CASE

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

### Lines 1274-1311: Implements R600InstrInfo::buildSlotOfVectorInstruction
```cpp
MachineInstr *R600InstrInfo::buildSlotOfVectorInstruction(
    MachineBasicBlock &MBB, MachineInstr *MI, unsigned Slot, unsigned DstReg)
    const {
  assert (MI->getOpcode() == R600::DOT_4 && "Not Implemented");
  unsigned Opcode;
  if (ST.getGeneration() <= AMDGPUSubtarget::R700)
    Opcode = R600::DOT4_r600;
  else
    Opcode = R600::DOT4_eg;
  MachineBasicBlock::iterator I = MI;
  MachineOperand &Src0 = MI->getOperand(
      getOperandIdx(MI->getOpcode(), getSlotedOps(R600::OpName::src0, Slot)));
  MachineOperand &Src1 = MI->getOperand(
      getOperandIdx(MI->getOpcode(), getSlotedOps(R600::OpName::src1, Slot)));
  MachineInstr *MIB = buildDefaultInstruction(
      MBB, I, Opcode, DstReg, Src0.getReg(), Src1.getReg());
  static const R600::OpName Operands[14] = {
      R600::OpName::update_exec_mask,
      R600::OpName::update_pred,
      R600::OpName::write,
      R600::OpName::omod,
      R600::OpName::dst_rel,
      R600::OpName::clamp,
      R600::OpName::src0_neg,
      R600::OpName::src0_rel,
      R600::OpName::src0_abs,
      R600::OpName::src0_sel,
      R600::OpName::src1_neg,
      R600::OpName::src1_rel,
      R600::OpName::src1_abs,
      R600::OpName::src1_sel,
  };

  MachineOperand &MO = MI->getOperand(getOperandIdx(MI->getOpcode(),
      getSlotedOps(R600::OpName::pred_sel, Slot)));
  MIB->getOperand(getOperandIdx(Opcode, R600::OpName::pred_sel))
      .setReg(MO.getReg());

```
**EN:** This section contains concrete logic for R600InstrInfo::buildSlotOfVectorInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::buildSlotOfVectorInstruction`.
**CN:** 本节包含与 R600InstrInfo::buildSlotOfVectorInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::buildSlotOfVectorInstruction`。

### Lines 1312-1354: Declares getOperand
```cpp
  for (R600::OpName Operand : Operands) {
    MachineOperand &MO = MI->getOperand(
        getOperandIdx(MI->getOpcode(), getSlotedOps(Operand, Slot)));
    assert (MO.isImm());
    setImmOperand(*MIB, Operand, MO.getImm());
  }
  MIB->getOperand(20).setImm(0);
  return MIB;
}

MachineInstr *R600InstrInfo::buildMovImm(MachineBasicBlock &BB,
                                         MachineBasicBlock::iterator I,
                                         unsigned DstReg,
                                         uint64_t Imm) const {
  MachineInstr *MovImm = buildDefaultInstruction(BB, I, R600::MOV, DstReg,
                                                  R600::ALU_LITERAL_X);
  setImmOperand(*MovImm, R600::OpName::literal, Imm);
  return MovImm;
}

MachineInstr *R600InstrInfo::buildMovInstr(MachineBasicBlock *MBB,
                                       MachineBasicBlock::iterator I,
                                       unsigned DstReg, unsigned SrcReg) const {
  return buildDefaultInstruction(*MBB, I, R600::MOV, DstReg, SrcReg);
}

int R600InstrInfo::getOperandIdx(const MachineInstr &MI,
                                 R600::OpName Op) const {
  return getOperandIdx(MI.getOpcode(), Op);
}

int R600InstrInfo::getOperandIdx(unsigned Opcode, R600::OpName Op) const {
  return R600::getNamedOperandIdx(Opcode, Op);
}

void R600InstrInfo::setImmOperand(MachineInstr &MI, R600::OpName Op,
                                  int64_t Imm) const {
  int Idx = getOperandIdx(MI, Op);
  assert(Idx != -1 && "Operand not supported for this instruction.");
  assert(MI.getOperand(Idx).isImm());
  MI.getOperand(Idx).setImm(Imm);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600InstrInfo::buildMovImm`, `R600InstrInfo::buildMovInstr`, `R600InstrInfo::getOperandIdx`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600InstrInfo::buildMovImm`, `R600InstrInfo::buildMovInstr`, `R600InstrInfo::getOperandIdx`。

### Lines 1355-1393: Implements R600InstrInfo::getFlagOp
```cpp
//===----------------------------------------------------------------------===//
// Instruction flag getters/setters
//===----------------------------------------------------------------------===//

MachineOperand &R600InstrInfo::getFlagOp(MachineInstr &MI, unsigned SrcIdx,
                                         unsigned Flag) const {
  unsigned TargetFlags = get(MI.getOpcode()).TSFlags;
  int FlagIndex = 0;
  if (Flag != 0) {
    // If we pass something other than the default value of Flag to this
    // function, it means we are want to set a flag on an instruction
    // that uses native encoding.
    assert(HAS_NATIVE_OPERANDS(TargetFlags));
    bool IsOP3 = (TargetFlags & R600_InstFlag::OP3) == R600_InstFlag::OP3;
    switch (Flag) {
    case MO_FLAG_CLAMP:
      FlagIndex = getOperandIdx(MI, R600::OpName::clamp);
      break;
    case MO_FLAG_MASK:
      FlagIndex = getOperandIdx(MI, R600::OpName::write);
      break;
    case MO_FLAG_NOT_LAST:
    case MO_FLAG_LAST:
      FlagIndex = getOperandIdx(MI, R600::OpName::last);
      break;
    case MO_FLAG_NEG:
      switch (SrcIdx) {
      case 0:
        FlagIndex = getOperandIdx(MI, R600::OpName::src0_neg);
        break;
      case 1:
        FlagIndex = getOperandIdx(MI, R600::OpName::src1_neg);
        break;
      case 2:
        FlagIndex = getOperandIdx(MI, R600::OpName::src2_neg);
        break;
      }
      break;

```
**EN:** This section contains concrete logic for R600InstrInfo::getFlagOp. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::getFlagOp`.
**CN:** 本节包含与 R600InstrInfo::getFlagOp 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::getFlagOp`。

### Lines 1394-1423: Switch-based control flow
```cpp
    case MO_FLAG_ABS:
      assert(!IsOP3 && "Cannot set absolute value modifier for OP3 "
                       "instructions.");
      (void)IsOP3;
      switch (SrcIdx) {
      case 0:
        FlagIndex = getOperandIdx(MI, R600::OpName::src0_abs);
        break;
      case 1:
        FlagIndex = getOperandIdx(MI, R600::OpName::src1_abs);
        break;
      }
      break;

    default:
      FlagIndex = -1;
      break;
    }
    assert(FlagIndex != -1 && "Flag not supported for this instruction");
  } else {
      FlagIndex = GET_FLAG_OPERAND_IDX(TargetFlags);
      assert(FlagIndex != 0 &&
         "Instruction flags not supported for this instruction");
  }

  MachineOperand &FlagOp = MI.getOperand(FlagIndex);
  assert(FlagOp.isImm());
  return FlagOp;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 1424-1457: Implements R600InstrInfo::addFlag
```cpp
void R600InstrInfo::addFlag(MachineInstr &MI, unsigned SrcIdx,
                            unsigned Flag) const {
  unsigned TargetFlags = get(MI.getOpcode()).TSFlags;
  if (Flag == 0) {
    return;
  }
  if (HAS_NATIVE_OPERANDS(TargetFlags)) {
    MachineOperand &FlagOp = getFlagOp(MI, SrcIdx, Flag);
    if (Flag == MO_FLAG_NOT_LAST) {
      clearFlag(MI, SrcIdx, MO_FLAG_LAST);
    } else if (Flag == MO_FLAG_MASK) {
      clearFlag(MI, SrcIdx, Flag);
    } else {
      FlagOp.setImm(1);
    }
  } else {
    MachineOperand &FlagOp = getFlagOp(MI, SrcIdx);
    FlagOp.setImm(FlagOp.getImm() | (Flag << (NUM_MO_FLAGS * SrcIdx)));
  }
}

void R600InstrInfo::clearFlag(MachineInstr &MI, unsigned SrcIdx,
                              unsigned Flag) const {
  unsigned TargetFlags = get(MI.getOpcode()).TSFlags;
  if (HAS_NATIVE_OPERANDS(TargetFlags)) {
    MachineOperand &FlagOp = getFlagOp(MI, SrcIdx, Flag);
    FlagOp.setImm(0);
  } else {
    MachineOperand &FlagOp = getFlagOp(MI);
    unsigned InstFlags = FlagOp.getImm();
    InstFlags &= ~(Flag << (NUM_MO_FLAGS * SrcIdx));
    FlagOp.setImm(InstFlags);
  }
}
```
**EN:** This section contains concrete logic for R600InstrInfo::addFlag. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600InstrInfo::addFlag`, `R600InstrInfo::clearFlag`.
**CN:** 本节包含与 R600InstrInfo::addFlag 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600InstrInfo::addFlag`, `R600InstrInfo::clearFlag`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600InstrInfo::R600InstrInfo`, `R600InstrInfo::isVector`, `R600InstrInfo::copyPhysReg`, `R600RegisterInfo::getSubRegFromChannel`, `R600InstrInfo::isLegalToSplitMBBAt`, `R600InstrInfo::isMov`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600InstrInfo.h"`
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600Defines.h"`
- `"R600Subtarget.h"`
- `"llvm/ADT/SmallSet.h"`
- `"llvm/CodeGen/MachineFrameInfo.h"`
- `"R600GenDFAPacketizer.inc"`
- `"R600GenInstrInfo.inc"`
