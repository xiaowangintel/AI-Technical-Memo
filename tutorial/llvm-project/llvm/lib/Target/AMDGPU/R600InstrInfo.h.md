# R600InstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600InstrInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for R600InstrInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 R600InstrInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===-- R600InstrInfo.h - R600 Instruction Info Interface -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Interface definition for R600InstrInfo
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_R600INSTRINFO_H
#define LLVM_LIB_TARGET_AMDGPU_R600INSTRINFO_H

#include "R600RegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

#define GET_INSTRINFO_HEADER
#define GET_INSTRINFO_OPERAND_ENUM
#include "R600GenInstrInfo.inc"

namespace llvm {

namespace R600InstrFlags {
enum : uint64_t {
 REGISTER_STORE = UINT64_C(1) << 62,
 REGISTER_LOAD = UINT64_C(1) << 63
};
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 33-53: Declares class DFAPacketizer
```cpp
class DFAPacketizer;
class MachineFunction;
class MachineInstr;
class MachineInstrBuilder;
class R600Subtarget;

class R600InstrInfo final : public R600GenInstrInfo {
private:
  const R600RegisterInfo RI;
  const R600Subtarget &ST;

  std::vector<std::pair<int, unsigned>>
  ExtractSrcs(MachineInstr &MI, const DenseMap<unsigned, unsigned> &PV,
              unsigned &ConstCount) const;

  MachineInstrBuilder buildIndirectRead(MachineBasicBlock *MBB,
                                        MachineBasicBlock::iterator I,
                                        unsigned ValueReg, unsigned Address,
                                        unsigned OffsetReg,
                                        unsigned AddrChan) const;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `DFAPacketizer`, `MachineFunction`, `MachineInstr`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`DFAPacketizer`, `MachineFunction`, `MachineInstr`。

### Lines 54-84: Defines buildIndirectWrite
```cpp
  MachineInstrBuilder buildIndirectWrite(MachineBasicBlock *MBB,
                                         MachineBasicBlock::iterator I,
                                         unsigned ValueReg, unsigned Address,
                                         unsigned OffsetReg,
                                         unsigned AddrChan) const;
public:
  enum BankSwizzle {
    ALU_VEC_012_SCL_210 = 0,
    ALU_VEC_021_SCL_122,
    ALU_VEC_120_SCL_212,
    ALU_VEC_102_SCL_221,
    ALU_VEC_201,
    ALU_VEC_210
  };

  explicit R600InstrInfo(const R600Subtarget &);

  const R600RegisterInfo &getRegisterInfo() const {
    return RI;
  }

  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;
  bool isLegalToSplitMBBAt(MachineBasicBlock &MBB,
                           MachineBasicBlock::iterator MBBI) const override;

  bool isReductionOp(unsigned opcode) const;
  bool isCubeOp(unsigned opcode) const;

```
**EN:** This section contains concrete logic for buildIndirectWrite. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `BankSwizzle`.
**CN:** 本节包含与 buildIndirectWrite 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`BankSwizzle`。

### Lines 85-114: Declares isALUInstr
```cpp
  /// \returns true if this \p Opcode represents an ALU instruction.
  bool isALUInstr(unsigned Opcode) const;
  bool hasInstrModifiers(unsigned Opcode) const;
  bool isLDSInstr(unsigned Opcode) const;
  bool isLDSRetInstr(unsigned Opcode) const;

  /// \returns true if this \p Opcode represents an ALU instruction or an
  /// instruction that will be lowered in ExpandSpecialInstrs Pass.
  bool canBeConsideredALU(const MachineInstr &MI) const;

  bool isTransOnly(unsigned Opcode) const;
  bool isTransOnly(const MachineInstr &MI) const;
  bool isVectorOnly(unsigned Opcode) const;
  bool isVectorOnly(const MachineInstr &MI) const;
  bool isExport(unsigned Opcode) const;

  bool usesVertexCache(unsigned Opcode) const;
  bool usesVertexCache(const MachineInstr &MI) const;
  bool usesTextureCache(unsigned Opcode) const;
  bool usesTextureCache(const MachineInstr &MI) const;

  bool mustBeLastInClause(unsigned Opcode) const;
  bool usesAddressRegister(MachineInstr &MI) const;
  bool definesAddressRegister(MachineInstr &MI) const;
  bool readsLDSSrcReg(const MachineInstr &MI) const;

  /// \returns The operand Index for the Sel operand given an index to one
  /// of the instruction's src operands.
  int getSelIdx(unsigned Opcode, unsigned SrcIdx) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 115-148: Declares getSrcs
```cpp
  /// \returns a pair for each src of an ALU instructions.
  /// The first member of a pair is the register id.
  /// If register is ALU_CONST, second member is SEL.
  /// If register is ALU_LITERAL, second member is IMM.
  /// Otherwise, second member value is undefined.
  SmallVector<std::pair<MachineOperand *, int64_t>, 3>
  getSrcs(MachineInstr &MI) const;

  unsigned  isLegalUpTo(
    const std::vector<std::vector<std::pair<int, unsigned> > > &IGSrcs,
    const std::vector<R600InstrInfo::BankSwizzle> &Swz,
    const std::vector<std::pair<int, unsigned> > &TransSrcs,
    R600InstrInfo::BankSwizzle TransSwz) const;

  bool FindSwizzleForVectorSlot(
    const std::vector<std::vector<std::pair<int, unsigned> > > &IGSrcs,
    std::vector<R600InstrInfo::BankSwizzle> &SwzCandidate,
    const std::vector<std::pair<int, unsigned> > &TransSrcs,
    R600InstrInfo::BankSwizzle TransSwz) const;

  /// Given the order VEC_012 < VEC_021 < VEC_120 < VEC_102 < VEC_201 < VEC_210
  /// returns true and the first (in lexical order) BankSwizzle affectation
  /// starting from the one already provided in the Instruction Group MIs that
  /// fits Read Port limitations in BS if available. Otherwise returns false
  /// and undefined content in BS.
  /// isLastAluTrans should be set if the last Alu of MIs will be executed on
  /// Trans ALU. In this case, ValidTSwizzle returns the BankSwizzle value to
  /// apply to the last instruction.
  /// PV holds GPR to PV registers in the Instruction Group MIs.
  bool fitsReadPortLimitations(const std::vector<MachineInstr *> &MIs,
                               const DenseMap<unsigned, unsigned> &PV,
                               std::vector<BankSwizzle> &BS,
                               bool isLastAluTrans) const;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 149-182: Declares fitsConstReadLimitations
```cpp
  /// An instruction group can only access 2 channel pair (either [XY] or [ZW])
  /// from KCache bank on R700+. This function check if MI set in input meet
  /// this limitations
  bool fitsConstReadLimitations(const std::vector<MachineInstr *> &) const;
  /// Same but using const index set instead of MI set.
  bool fitsConstReadLimitations(const std::vector<unsigned>&) const;

  /// Vector instructions are instructions that must fill all
  /// instruction slots within an instruction group.
  bool isVector(const MachineInstr &MI) const;

  bool isMov(unsigned Opcode) const;

  DFAPacketizer *
  CreateTargetScheduleState(const TargetSubtargetInfo &) const override;

  bool reverseBranchCondition(
    SmallVectorImpl<MachineOperand> &Cond) const override;

  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify) const override;

  unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
                        MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
                        const DebugLoc &DL,
                        int *BytesAdded = nullptr) const override;

  unsigned removeBranch(MachineBasicBlock &MBB,
                        int *BytesRemoved = nullptr) const override;

  bool isPredicated(const MachineInstr &MI) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 183-214: Declares isPredicable
```cpp
  bool isPredicable(const MachineInstr &MI) const override;

  bool isProfitableToDupForIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
                                 BranchProbability Probability) const override;

  bool isProfitableToIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
                           unsigned ExtraPredCycles,
                           BranchProbability Probability) const override ;

  bool isProfitableToIfCvt(MachineBasicBlock &TMBB,
                           unsigned NumTCycles, unsigned ExtraTCycles,
                           MachineBasicBlock &FMBB,
                           unsigned NumFCycles, unsigned ExtraFCycles,
                           BranchProbability Probability) const override;

  bool ClobbersPredicate(MachineInstr &MI, std::vector<MachineOperand> &Pred,
                         bool SkipDead) const override;

  bool isProfitableToUnpredicate(MachineBasicBlock &TMBB,
                                 MachineBasicBlock &FMBB) const override;

  bool PredicateInstruction(MachineInstr &MI,
                            ArrayRef<MachineOperand> Pred) const override;

  unsigned int getPredicationCost(const MachineInstr &) const override;

  unsigned int getInstrLatency(const InstrItineraryData *ItinData,
                               const MachineInstr &MI,
                               unsigned *PredCost = nullptr) const override;

  bool expandPostRAPseudo(MachineInstr &MI) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 215-241: Defines reserveIndirectRegisters
```cpp
  /// Reserve the registers that may be accessed using indirect addressing.
  void reserveIndirectRegisters(BitVector &Reserved,
                                const MachineFunction &MF,
                                const R600RegisterInfo &TRI) const;

  /// Calculate the "Indirect Address" for the given \p RegIndex and
  /// \p Channel
  ///
  /// We model indirect addressing using a virtual address space that can be
  /// accessed with loads and stores.  The "Indirect Address" is the memory
  /// address in this virtual address space that maps to the given \p RegIndex
  /// and \p Channel.
  unsigned calculateIndirectAddress(unsigned RegIndex, unsigned Channel) const;


  /// \returns The register class to be used for loading and storing values
  /// from an "Indirect Address" .
  const TargetRegisterClass *getIndirectAddrRegClass() const;

  /// \returns the smallest register index that will be accessed by an indirect
  /// read or write or -1 if indirect addressing is not used by this program.
  int getIndirectIndexBegin(const MachineFunction &MF) const;

  /// \returns the largest register index that will be accessed by an indirect
  /// read or write or -1 if indirect addressing is not used by this program.
  int getIndirectIndexEnd(const MachineFunction &MF) const;

```
**EN:** This section contains concrete logic for reserveIndirectRegisters. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 reserveIndirectRegisters 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 242-273: Defines buildIndirectWrite
```cpp
  /// Build instruction(s) for an indirect register write.
  ///
  /// \returns The instruction that performs the indirect register write
  MachineInstrBuilder buildIndirectWrite(MachineBasicBlock *MBB,
                                         MachineBasicBlock::iterator I,
                                         unsigned ValueReg, unsigned Address,
                                         unsigned OffsetReg) const;

  /// Build instruction(s) for an indirect register read.
  ///
  /// \returns The instruction that performs the indirect register read
  MachineInstrBuilder buildIndirectRead(MachineBasicBlock *MBB,
                                        MachineBasicBlock::iterator I,
                                        unsigned ValueReg, unsigned Address,
                                        unsigned OffsetReg) const;

  unsigned getMaxAlusPerClause() const;

  /// buildDefaultInstruction - This function returns a MachineInstr with all
  /// the instruction modifiers initialized to their default values.  You can
  /// use this function to avoid manually specifying each instruction modifier
  /// operand when building a new instruction.
  ///
  /// \returns a MachineInstr with all the instruction modifiers initialized
  /// to their default values.
  MachineInstrBuilder buildDefaultInstruction(MachineBasicBlock &MBB,
                                              MachineBasicBlock::iterator I,
                                              unsigned Opcode,
                                              unsigned DstReg,
                                              unsigned Src0Reg,
                                              unsigned Src1Reg = 0) const;

```
**EN:** This section contains concrete logic for buildIndirectWrite. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 buildIndirectWrite 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 274-306: Defines buildSlotOfVectorInstruction
```cpp
  MachineInstr *buildSlotOfVectorInstruction(MachineBasicBlock &MBB,
                                             MachineInstr *MI,
                                             unsigned Slot,
                                             unsigned DstReg) const;

  MachineInstr *buildMovImm(MachineBasicBlock &BB,
                            MachineBasicBlock::iterator I,
                            unsigned DstReg,
                            uint64_t Imm) const;

  MachineInstr *buildMovInstr(MachineBasicBlock *MBB,
                              MachineBasicBlock::iterator I,
                              unsigned DstReg, unsigned SrcReg) const;

  /// Get the index of Op in the MachineInstr.
  ///
  /// \returns -1 if the Instruction does not contain the specified \p Op.
  int getOperandIdx(const MachineInstr &MI, R600::OpName Op) const;

  /// Get the index of \p Op for the given Opcode.
  ///
  /// \returns -1 if the Instruction does not contain the specified \p Op.
  int getOperandIdx(unsigned Opcode, R600::OpName Op) const;

  /// Helper function for setting instruction flag values.
  void setImmOperand(MachineInstr &MI, R600::OpName Op, int64_t Imm) const;

  /// Add one of the MO_FLAG* flags to the operand at \p SrcIdx.
  void addFlag(MachineInstr &MI, unsigned SrcIdx, unsigned Flag) const;

  /// Determine if the specified \p Flag is set on operand at \p SrcIdx.
  bool isFlagSet(const MachineInstr &MI, unsigned SrcIdx, unsigned Flag) const;

```
**EN:** This section contains concrete logic for buildSlotOfVectorInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 buildSlotOfVectorInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 307-335: Preprocessor guards and macros
```cpp
  /// \param SrcIdx The register source to set the flag on (e.g src0, src1, src2)
  /// \param Flag The flag being set.
  ///
  /// \returns the operand containing the flags for this instruction.
  MachineOperand &getFlagOp(MachineInstr &MI, unsigned SrcIdx = 0,
                            unsigned Flag = 0) const;

  /// Clear the specified flag on the instruction.
  void clearFlag(MachineInstr &MI, unsigned SrcIdx, unsigned Flag) const;

  // Helper functions that check the opcode for status information
  bool isRegisterStore(const MachineInstr &MI) const {
    return get(MI.getOpcode()).TSFlags & R600InstrFlags::REGISTER_STORE;
  }

  bool isRegisterLoad(const MachineInstr &MI) const {
    return get(MI.getOpcode()).TSFlags & R600InstrFlags::REGISTER_LOAD;
  }
};

namespace R600 {

int32_t getLDSNoRetOp(uint32_t Opcode);

} //End namespace AMDGPU

} // End llvm namespace

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `DFAPacketizer`, `MachineFunction`, `MachineInstr`, `MachineInstrBuilder`, `R600Subtarget`, `R600InstrInfo`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600RegisterInfo.h"`
- `"llvm/CodeGen/TargetInstrInfo.h"`
- `"R600GenInstrInfo.inc"`
