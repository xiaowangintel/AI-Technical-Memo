# RISCVInstrInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for instruction definitions, machine semantics, and lowering helpers for RISC-V. / 声明RISC-V 的指令定义、机器语义与下降辅助逻辑所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVInstrInfo.h - RISC-V Instruction Information -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the RISC-V implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_RISCV_RISCVINSTRINFO_H
#define LLVM_LIB_TARGET_RISCV_RISCVINSTRINFO_H
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-43: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#include "RISCV.h"
#include "RISCVRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/DiagnosticInfo.h"

#define GET_INSTRINFO_HEADER
#include "RISCVGenInstrInfo.inc"
#include "RISCVGenRegisterInfo.inc"

namespace llvm {

// If Value is of the form C1<<C2, where C1 = 3, 5 or 9,
// returns log2(C1 - 1) and assigns Shift = C2.
// Otherwise, returns 0.
template <typename T> int isShifted359(T Value, int &Shift) {
  if (Value == 0)
    return 0;
  Shift = llvm::countr_zero(Value);
  switch (Value >> Shift) {
  case 3:
    return 1;
  case 5:
    return 2;
  case 9:
    return 3;
  default:
    return 0;
  }
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 44-63: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
}

class RISCVSubtarget;

static const MachineMemOperand::Flags MONontemporalBit0 =
    MachineMemOperand::MOTargetFlag1;
static const MachineMemOperand::Flags MONontemporalBit1 =
    MachineMemOperand::MOTargetFlag2;

namespace RISCVCC {

enum CondCode {
  COND_EQ,
  COND_NE,
  COND_LT,
  COND_GE,
  COND_LTU,
  COND_GEU,
  COND_INVALID
};
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 64-79: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

CondCode getInverseBranchCondition(CondCode);
unsigned getInverseBranchOpcode(unsigned BCC);
unsigned getBrCond(CondCode CC, unsigned SelectOpc = 0);

} // end of namespace RISCVCC

// RISCV MachineCombiner patterns
enum RISCVMachineCombinerPattern : unsigned {
  FMADD_AX = MachineCombinerPattern::TARGET_PATTERN_START,
  FMADD_XA,
  FMSUB,
  FNMSUB,
  SHXADD_ADD_SLLI_OP1,
  SHXADD_ADD_SLLI_OP2,
};
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 80-99: Type declaration for RISCVInstrInfo / RISCVInstrInfo 的类型声明
```cpp

class RISCVInstrInfo : public RISCVGenInstrInfo {
  const RISCVRegisterInfo RegInfo;

public:
  explicit RISCVInstrInfo(const RISCVSubtarget &STI);

  const RISCVRegisterInfo &getRegisterInfo() const { return RegInfo; }

  MCInst getNop() const override;

  Register isLoadFromStackSlot(const MachineInstr &MI,
                               int &FrameIndex) const override;
  Register isLoadFromStackSlot(const MachineInstr &MI, int &FrameIndex,
                               TypeSize &MemBytes) const override;
  Register isStoreToStackSlot(const MachineInstr &MI,
                              int &FrameIndex) const override;
  Register isStoreToStackSlot(const MachineInstr &MI, int &FrameIndex,
                              TypeSize &MemBytes) const override;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 100-115: Function implementation: shouldBreakCriticalEdgeToSink / 函数实现：shouldBreakCriticalEdgeToSink
```cpp
  bool isReMaterializableImpl(const MachineInstr &MI) const override;

  bool shouldBreakCriticalEdgeToSink(MachineInstr &MI) const override {
    return MI.getOpcode() == RISCV::ADDI && MI.getOperand(1).isReg() &&
           MI.getOperand(1).getReg() == RISCV::X0;
  }

  void copyPhysRegVector(MachineBasicBlock &MBB,
                         MachineBasicBlock::iterator MBBI, const DebugLoc &DL,
                         MCRegister DstReg, MCRegister SrcReg, bool KillSrc,
                         const TargetRegisterClass *RegClass) const;
  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
                   const DebugLoc &DL, Register DstReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 116-135: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void storeRegToStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
      bool IsKill, int FrameIndex, const TargetRegisterClass *RC,

      Register VReg,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

  void loadRegFromStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register DstReg,
      int FrameIndex, const TargetRegisterClass *RC, Register VReg,
      unsigned SubReg = 0,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

  using TargetInstrInfo::foldMemoryOperandImpl;
  MachineInstr *foldMemoryOperandImpl(MachineFunction &MF, MachineInstr &MI,
                                      ArrayRef<unsigned> Ops, int FrameIndex,
                                      MachineInstr *&CopyMI,
                                      LiveIntervals *LIS = nullptr,
                                      VirtRegMap *VRM = nullptr) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 136-150: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  MachineInstr *foldMemoryOperandImpl(MachineFunction &MF, MachineInstr &MI,
                                      ArrayRef<unsigned> Ops,
                                      MachineInstr &LoadMI,
                                      MachineInstr *&CopyMI,
                                      LiveIntervals *LIS = nullptr,
                                      VirtRegMap *VRM = nullptr) const override;

  // Materializes the given integer Val into DstReg.
  void movImm(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
              const DebugLoc &DL, Register DstReg, uint64_t Val,
              MachineInstr::MIFlag Flag = MachineInstr::NoFlags,
              bool DstRenamable = false, bool DstIsDead = false) const;

  unsigned getInstSizeInBytes(const MachineInstr &MI) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 151-165: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify) const override;

  unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
                        MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
                        const DebugLoc &dl,
                        int *BytesAdded = nullptr) const override;

  void insertIndirectBranch(MachineBasicBlock &MBB,
                            MachineBasicBlock &NewDestBB,
                            MachineBasicBlock &RestoreBB, const DebugLoc &DL,
                            int64_t BrOffset, RegScavenger *RS) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 166-182: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  unsigned removeBranch(MachineBasicBlock &MBB,
                        int *BytesRemoved = nullptr) const override;

  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;

  bool optimizeCondBranch(MachineInstr &MI) const override;

  MachineBasicBlock *getBranchDestBlock(const MachineInstr &MI) const override;

  bool isBranchOffsetInRange(unsigned BranchOpc,
                             int64_t BrOffset) const override;

  MachineInstr *optimizeSelect(MachineInstr &MI,
                               SmallPtrSetImpl<MachineInstr *> &SeenMIs,
                               bool) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 183-197: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool isAsCheapAsAMove(const MachineInstr &MI) const override;

  std::optional<DestSourcePair>
  isCopyInstrImpl(const MachineInstr &MI) const override;

  bool verifyInstruction(const MachineInstr &MI,
                         StringRef &ErrInfo) const override;

  bool canFoldIntoAddrMode(const MachineInstr &MemI, Register Reg,
                           const MachineInstr &AddrI,
                           ExtAddrMode &AM) const override;

  MachineInstr *emitLdStWithAddr(MachineInstr &MemI,
                                 const ExtAddrMode &AM) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 198-214: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool getMemOperandsWithOffsetWidth(
      const MachineInstr &MI, SmallVectorImpl<const MachineOperand *> &BaseOps,
      int64_t &Offset, bool &OffsetIsScalable, LocationSize &Width,
      const TargetRegisterInfo *TRI) const override;

  bool shouldClusterMemOps(ArrayRef<const MachineOperand *> BaseOps1,
                           int64_t Offset1, bool OffsetIsScalable1,
                           ArrayRef<const MachineOperand *> BaseOps2,
                           int64_t Offset2, bool OffsetIsScalable2,
                           unsigned ClusterSize,
                           unsigned NumBytes) const override;

  bool getMemOperandWithOffsetWidth(const MachineInstr &LdSt,
                                    const MachineOperand *&BaseOp,
                                    int64_t &Offset, LocationSize &Width,
                                    const TargetRegisterInfo *TRI) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 215-228: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
                                       const MachineInstr &MIb) const override;


  std::pair<unsigned, unsigned>
  decomposeMachineOperandsTargetFlags(unsigned TF) const override;

  ArrayRef<std::pair<unsigned, const char *>>
  getSerializableDirectMachineOperandTargetFlags() const override;

  // Return true if the function can safely be outlined from.
  bool isFunctionSafeToOutlineFrom(MachineFunction &MF,
                                   bool OutlineFromLinkOnceODRs) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 229-244: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Return true if MBB is safe to outline from, and return any target-specific
  // information in Flags.
  bool isMBBSafeToOutlineFrom(MachineBasicBlock &MBB,
                              unsigned &Flags) const override;

  bool shouldOutlineFromFunctionByDefault(MachineFunction &MF) const override;

  // Return true if the candidate should be discarded from outlining.
  bool analyzeCandidate(outliner::Candidate &C) const;
  // Calculate target-specific information for a set of outlining candidates.
  std::optional<std::unique_ptr<outliner::OutlinedFunction>>
  getOutliningCandidateInfo(
      const MachineModuleInfo &MMI,
      std::vector<outliner::Candidate> &RepeatedSequenceLocs,
      unsigned MinRepeats) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 245-259: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Return if/how a given MachineInstr should be outlined.
  outliner::InstrType getOutliningTypeImpl(const MachineModuleInfo &MMI,
                                           MachineBasicBlock::iterator &MBBI,
                                           unsigned Flags) const override;

  // Insert a custom frame for outlined functions.
  void buildOutlinedFrame(MachineBasicBlock &MBB, MachineFunction &MF,
                          const outliner::OutlinedFunction &OF) const override;

  // Insert a call to an outlined function into a given basic block.
  MachineBasicBlock::iterator
  insertOutlinedCall(Module &M, MachineBasicBlock &MBB,
                     MachineBasicBlock::iterator &It, MachineFunction &MF,
                     outliner::Candidate &C) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 260-273: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  std::optional<RegImmPair> isAddImmediate(const MachineInstr &MI,
                                           Register Reg) const override;

  bool findCommutedOpIndices(const MachineInstr &MI, unsigned &SrcOpIdx1,
                             unsigned &SrcOpIdx2) const override;
  MachineInstr *commuteInstructionImpl(MachineInstr &MI, bool NewMI,
                                       unsigned OpIdx1,
                                       unsigned OpIdx2) const override;

  bool simplifyInstruction(MachineInstr &MI) const override;

  MachineInstr *convertToThreeAddress(MachineInstr &MI, LiveVariables *LV,
                                      LiveIntervals *LIS) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 274-288: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // MIR printer helper function to annotate Operands with a comment.
  std::string
  createMIROperandComment(const MachineInstr &MI, const MachineOperand &Op,
                          unsigned OpIdx,
                          const TargetRegisterInfo *TRI) const override;

  /// Generate code to multiply the value in DestReg by Amt - handles all
  /// the common optimizations for this idiom, and supports fallback for
  /// subtargets which don't support multiply instructions.
  void mulImm(MachineFunction &MF, MachineBasicBlock &MBB,
              MachineBasicBlock::iterator II, const DebugLoc &DL,
              Register DestReg, uint32_t Amt, MachineInstr::MIFlag Flag) const;

  bool useMachineCombiner() const override { return true; }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 289-306: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  MachineTraceStrategy getMachineCombinerTraceStrategy() const override;

  CombinerObjective getCombinerObjective(unsigned Pattern) const override;

  bool getMachineCombinerPatterns(MachineInstr &Root,
                                  SmallVectorImpl<unsigned> &Patterns,
                                  bool DoRegPressureReduce) const override;

  void
  finalizeInsInstrs(MachineInstr &Root, unsigned &Pattern,
                    SmallVectorImpl<MachineInstr *> &InsInstrs) const override;

  void genAlternativeCodeSequence(
      MachineInstr &Root, unsigned Pattern,
      SmallVectorImpl<MachineInstr *> &InsInstrs,
      SmallVectorImpl<MachineInstr *> &DelInstrs,
      DenseMap<Register, unsigned> &InstrIdxForVirtReg) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 307-321: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool hasReassociableOperands(const MachineInstr &Inst,
                               const MachineBasicBlock *MBB) const override;

  bool hasReassociableSibling(const MachineInstr &Inst,
                              bool &Commuted) const override;

  bool isAssociativeAndCommutative(const MachineInstr &Inst,
                                   bool Invert) const override;

  std::optional<unsigned> getInverseOpcode(unsigned Opcode) const override;

  void getReassociateOperandIndices(
      const MachineInstr &Root, unsigned Pattern,
      std::array<unsigned, 5> &OperandIndices) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 322-335: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  ArrayRef<std::pair<MachineMemOperand::Flags, const char *>>
  getSerializableMachineMemOperandTargetFlags() const override;

  unsigned getTailDuplicateSize(CodeGenOptLevel OptLevel) const override;

  std::unique_ptr<TargetInstrInfo::PipelinerLoopInfo>
  analyzeLoopForPipelining(MachineBasicBlock *LoopBB) const override;

  bool isHighLatencyDef(int Opc) const override;

  /// Return true if \p MI is a COPY to a vector register of a specific \p LMul,
  /// or any kind of vector registers when \p LMul is zero.
  bool isVRegCopy(const MachineInstr *MI, unsigned LMul = 0) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 336-352: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
  /// Return true if the instruction requires an NTL hint to be emitted.
  bool requiresNTLHint(const MachineInstr &MI) const;

  /// Return true if moving \p From down to \p To won't cause any physical
  /// register reads or writes to be clobbered and no visible side effects are
  /// affected. From and To must be in the same block.
  static bool isSafeToMove(const MachineInstr &From,
                           const MachineBasicBlock::iterator &To);

  /// Return true if pairing the given load or store may be paired with another.
  static bool isPairableLdStInstOpc(unsigned Opc);

  static bool isLdStSafeToPair(const MachineInstr &LdSt,
                               const TargetRegisterInfo *TRI);
#define GET_INSTRINFO_HELPER_DECLS
#include "RISCVGenInstrInfo.inc"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 353-366: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  static RISCVCC::CondCode getCondFromBranchOpc(unsigned Opc);

  /// Return the result of the evaluation of C0 CC C1, where CC is a
  /// RISCVCC::CondCode.
  static bool evaluateCondBranch(RISCVCC::CondCode CC, int64_t C0, int64_t C1);

  /// Return true if the operand is a load immediate instruction and
  /// sets Imm to the immediate value.
  static bool isFromLoadImm(const MachineRegisterInfo &MRI,
                            const MachineOperand &Op, int64_t &Imm);

protected:
  const RISCVSubtarget &STI;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 367-381: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
private:
  bool isVectorAssociativeAndCommutative(const MachineInstr &MI,
                                         bool Invert = false) const;
  bool areRVVInstsReassociable(const MachineInstr &MI1,
                               const MachineInstr &MI2) const;
  bool hasReassociableVectorSibling(const MachineInstr &Inst,
                                    bool &Commuted) const;
};

namespace RISCV {

// Returns true if the given MI is an RVV instruction opcode for which we may
// expect to see a FrameIndex operand.
bool isRVVSpill(const MachineInstr &MI);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 382-397: Definitions and supporting logic / 定义与支撑逻辑
```cpp
/// Return true if \p MI is a copy that will be lowered to one or more vmvNr.vs.
bool isVectorCopy(const TargetRegisterInfo *TRI, const MachineInstr &MI);

std::optional<std::pair<unsigned, unsigned>>
isRVVSpillForZvlsseg(unsigned Opcode);

// Return true if both input instructions have equal rounding mode. If at least
// one of the instructions does not have rounding mode, false will be returned.
bool hasEqualFRM(const MachineInstr &MI1, const MachineInstr &MI2);

// If \p Opcode is a .vx vector instruction, returns the lower number of bits
// that are used from the scalar .x operand for a given \p Log2SEW. Otherwise
// returns null.
std::optional<unsigned> getVectorLowDemandedScalarBits(unsigned Opcode,
                                                       unsigned Log2SEW);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 398-423: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
// Returns the MC opcode of RVV pseudo instruction.
unsigned getRVVMCOpcode(unsigned RVVPseudoOpcode);

// For a (non-pseudo) RVV instruction \p Desc and the given \p Log2SEW, returns
// the log2 EEW of the destination operand.
unsigned getDestLog2EEW(const MCInstrDesc &Desc, unsigned Log2SEW);

// Special immediate for AVL operand of V pseudo instructions to indicate VLMax.
static constexpr int64_t VLMaxSentinel = -1LL;

/// Given two VL operands, do we know that LHS <= RHS?
bool isVLKnownLE(const MachineOperand &LHS, const MachineOperand &RHS);

// Mask assignments for floating-point
static constexpr unsigned FPMASK_Negative_Infinity = 0x001;
static constexpr unsigned FPMASK_Negative_Normal = 0x002;
static constexpr unsigned FPMASK_Negative_Subnormal = 0x004;
static constexpr unsigned FPMASK_Negative_Zero = 0x008;
static constexpr unsigned FPMASK_Positive_Zero = 0x010;
static constexpr unsigned FPMASK_Positive_Subnormal = 0x020;
static constexpr unsigned FPMASK_Positive_Normal = 0x040;
static constexpr unsigned FPMASK_Positive_Infinity = 0x080;
static constexpr unsigned FPMASK_Signaling_NaN = 0x100;
static constexpr unsigned FPMASK_Quiet_NaN = 0x200;
} // namespace RISCV
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 424-438: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
namespace RISCVVPseudosTable {

struct PseudoInfo {
  uint16_t Pseudo;
  uint16_t BaseInstr;
  uint16_t VLMul : 3;
  uint16_t SEW : 8;
  uint16_t IsAltFmt : 1;
};

#define GET_RISCVVPseudosTable_DECL
#include "RISCVGenSearchableTables.inc"

} // end namespace RISCVVPseudosTable
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 439-451: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
namespace RISCV {

struct RISCVMaskedPseudoInfo {
  uint16_t MaskedPseudo;
  uint16_t UnmaskedPseudo;
  uint8_t MaskOpIdx;
};
#define GET_RISCVMaskedPseudosTable_DECL
#include "RISCVGenSearchableTables.inc"
} // end namespace RISCV

} // end namespace llvm
#endif
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/DiagnosticInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenInstrInfo.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenRegisterInfo.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenSearchableTables.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
