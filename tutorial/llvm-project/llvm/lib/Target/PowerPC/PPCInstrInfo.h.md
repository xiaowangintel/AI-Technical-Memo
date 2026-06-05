# PPCInstrInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrInfo.h - PowerPC Instruction Information.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrInfo.h`，主要负责 PowerPC 后端的指令信息与 lowering 辅助逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCInstrInfo.h - PowerPC Instruction Information --------*- C++ -*-===//
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
//
// This file contains the PowerPC implementation of the TargetInstrInfo class.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains the PowerPC implementation of the TargetInstrInfo class.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains the PowerPC implementation of the TargetInstrInfo class.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCINSTRINFO_H
#define LLVM_LIB_TARGET_POWERPC_PPCINSTRINFO_H

#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "PPC.h"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 18-25

```cpp
#include "PPCRegisterInfo.h"
#include "llvm/ADT/SmallSet.h"
#include "llvm/CodeGen/LiveVariables.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

#define GET_INSTRINFO_HEADER
#include "PPCGenInstrInfo.inc"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 26-51

```cpp
namespace llvm {

// Instructions that have an immediate form might be convertible to that
// form if the correct input is a result of a load immediate. In order to
// know whether the transformation is special, we might need to know some
// of the details of the two forms.
struct ImmInstrInfo {
  // Is the immediate field in the immediate form signed or unsigned?
  uint64_t SignedImm : 1;
  // Does the immediate need to be a multiple of some value?
  uint64_t ImmMustBeMultipleOf : 5;
  // Is R0/X0 treated specially by the original r+r instruction?
  // If so, in which operand?
  uint64_t ZeroIsSpecialOrig : 3;
  // Is R0/X0 treated specially by the new r+i instruction?
  // If so, in which operand?
  uint64_t ZeroIsSpecialNew : 3;
  // Is the operation commutative?
  uint64_t IsCommutative : 1;
  // The operand number to check for add-immediate def.
  uint64_t OpNoForForwarding : 3;
  // The operand number for the immediate.
  uint64_t ImmOpNo : 3;
  // The opcode of the new instruction.
  uint64_t ImmOpcode : 16;
  // The size of the immediate.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions that have an immediate form might be convertible to that". Notable symbols in this range include `ImmInstrInfo`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions that have an immediate form might be convertible to that”。 该区间中较显眼的符号包括 `ImmInstrInfo`。

### Lines 52-60

```cpp
  uint64_t ImmWidth : 5;
  // The immediate should be truncated to N bits.
  uint64_t TruncateImmTo : 5;
  // Is the instruction summing the operand
  uint64_t IsSummingOperands : 1;
};

// Information required to convert an instruction to just a materialized
// immediate.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The immediate should be truncated to N bits.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The immediate should be truncated to N bits.”。

### Lines 61-67

```cpp
struct LoadImmediateInfo {
  unsigned Imm : 16;
  unsigned Is64Bit : 1;
  unsigned SetCR : 1;
};

// Index into the OpcodesForSpill array.
```
- **EN**: Declares a backend-facing type `LoadImmediateInfo` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `LoadImmediateInfo`，并勾勒出周边代码会依赖的接口或状态。

### Lines 68-91

```cpp
enum SpillOpcodeKey {
  SOK_Int4Spill,
  SOK_Int8Spill,
  SOK_Float8Spill,
  SOK_Float4Spill,
  SOK_CRSpill,
  SOK_CRBitSpill,
  SOK_VRVectorSpill,
  SOK_VSXVectorSpill,
  SOK_VectorFloat8Spill,
  SOK_VectorFloat4Spill,
  SOK_SpillToVSR,
  SOK_PairedVecSpill,
  SOK_AccumulatorSpill,
  SOK_UAccumulatorSpill,
  SOK_WAccumulatorSpill,
  SOK_DMRpSpill,
  SOK_DMRSpill,
  SOK_SPESpill,
  SOK_PairedG8Spill,
  SOK_LastOpcodeSpill // This must be last on the enum.
};

// PPC MachineCombiner patterns
```
- **EN**: Introduces enumerated values such as `SpillOpcodeKey` to make target-specific cases explicit and type-safe.
- **CN**: 这里引入枚举值，例如 `SpillOpcodeKey`，用更显式且更安全的方式表达目标相关分支。

### Lines 92-104

```cpp
enum PPCMachineCombinerPattern : unsigned {
  // These are patterns matched by the PowerPC to reassociate FMA chains.
  REASSOC_XY_AMM_BMM = MachineCombinerPattern::TARGET_PATTERN_START,
  REASSOC_XMM_AMM_BMM,

  // These are patterns matched by the PowerPC to reassociate FMA and FSUB to
  // reduce register pressure.
  REASSOC_XY_BCA,
  REASSOC_XY_BAC,

};

// Define list of load and store spill opcodes.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These are patterns matched by the PowerPC to reassociate FMA chains.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These are patterns matched by the PowerPC to reassociate FMA chains.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 105-126

```cpp
#define NoInstr PPC::INSTRUCTION_LIST_END
#define Pwr8LoadOpcodes                                                        \
  {PPC::LWZ,                                                                   \
   PPC::LD,                                                                    \
   PPC::LFD,                                                                   \
   PPC::LFS,                                                                   \
   PPC::RESTORE_CR,                                                            \
   PPC::RESTORE_CRBIT,                                                         \
   PPC::LVX,                                                                   \
   PPC::LXVD2X,                                                                \
   PPC::LXSDX,                                                                 \
   PPC::LXSSPX,                                                                \
   PPC::SPILLTOVSR_LD,                                                         \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::EVLDD,                                                                 \
   PPC::RESTORE_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 127-147

```cpp
#define Pwr9LoadOpcodes                                                        \
  {PPC::LWZ,                                                                   \
   PPC::LD,                                                                    \
   PPC::LFD,                                                                   \
   PPC::LFS,                                                                   \
   PPC::RESTORE_CR,                                                            \
   PPC::RESTORE_CRBIT,                                                         \
   PPC::LVX,                                                                   \
   PPC::LXV,                                                                   \
   PPC::DFLOADf64,                                                             \
   PPC::DFLOADf32,                                                             \
   PPC::SPILLTOVSR_LD,                                                         \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::RESTORE_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 148-168

```cpp
#define Pwr10LoadOpcodes                                                       \
  {PPC::LWZ,                                                                   \
   PPC::LD,                                                                    \
   PPC::LFD,                                                                   \
   PPC::LFS,                                                                   \
   PPC::RESTORE_CR,                                                            \
   PPC::RESTORE_CRBIT,                                                         \
   PPC::LVX,                                                                   \
   PPC::LXV,                                                                   \
   PPC::DFLOADf64,                                                             \
   PPC::DFLOADf32,                                                             \
   PPC::SPILLTOVSR_LD,                                                         \
   PPC::LXVP,                                                                  \
   PPC::RESTORE_ACC,                                                           \
   PPC::RESTORE_UACC,                                                          \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::RESTORE_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 169-189

```cpp
#define FutureLoadOpcodes                                                      \
  {PPC::LWZ,                                                                   \
   PPC::LD,                                                                    \
   PPC::LFD,                                                                   \
   PPC::LFS,                                                                   \
   PPC::RESTORE_CR,                                                            \
   PPC::RESTORE_CRBIT,                                                         \
   PPC::LVX,                                                                   \
   PPC::LXV,                                                                   \
   PPC::DFLOADf64,                                                             \
   PPC::DFLOADf32,                                                             \
   PPC::SPILLTOVSR_LD,                                                         \
   PPC::LXVP,                                                                  \
   PPC::RESTORE_ACC,                                                           \
   PPC::RESTORE_UACC,                                                          \
   PPC::RESTORE_WACC,                                                          \
   PPC::RESTORE_DMRP,                                                          \
   PPC::RESTORE_DMR,                                                           \
   NoInstr,                                                                    \
   PPC::RESTORE_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 190-210

```cpp
#define Pwr8StoreOpcodes                                                       \
  {PPC::STW,                                                                   \
   PPC::STD,                                                                   \
   PPC::STFD,                                                                  \
   PPC::STFS,                                                                  \
   PPC::SPILL_CR,                                                              \
   PPC::SPILL_CRBIT,                                                           \
   PPC::STVX,                                                                  \
   PPC::STXVD2X,                                                               \
   PPC::STXSDX,                                                                \
   PPC::STXSSPX,                                                               \
   PPC::SPILLTOVSR_ST,                                                         \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::EVSTDD,                                                                \
   PPC::SPILL_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 211-231

```cpp
#define Pwr9StoreOpcodes                                                       \
  {PPC::STW,                                                                   \
   PPC::STD,                                                                   \
   PPC::STFD,                                                                  \
   PPC::STFS,                                                                  \
   PPC::SPILL_CR,                                                              \
   PPC::SPILL_CRBIT,                                                           \
   PPC::STVX,                                                                  \
   PPC::STXV,                                                                  \
   PPC::DFSTOREf64,                                                            \
   PPC::DFSTOREf32,                                                            \
   PPC::SPILLTOVSR_ST,                                                         \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::SPILL_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 232-252

```cpp
#define Pwr10StoreOpcodes                                                      \
  {PPC::STW,                                                                   \
   PPC::STD,                                                                   \
   PPC::STFD,                                                                  \
   PPC::STFS,                                                                  \
   PPC::SPILL_CR,                                                              \
   PPC::SPILL_CRBIT,                                                           \
   PPC::STVX,                                                                  \
   PPC::STXV,                                                                  \
   PPC::DFSTOREf64,                                                            \
   PPC::DFSTOREf32,                                                            \
   PPC::SPILLTOVSR_ST,                                                         \
   PPC::STXVP,                                                                 \
   PPC::SPILL_ACC,                                                             \
   PPC::SPILL_UACC,                                                            \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   NoInstr,                                                                    \
   PPC::SPILL_QUADWORD}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 253-274

```cpp
#define FutureStoreOpcodes                                                     \
  {PPC::STW,                                                                   \
   PPC::STD,                                                                   \
   PPC::STFD,                                                                  \
   PPC::STFS,                                                                  \
   PPC::SPILL_CR,                                                              \
   PPC::SPILL_CRBIT,                                                           \
   PPC::STVX,                                                                  \
   PPC::STXV,                                                                  \
   PPC::DFSTOREf64,                                                            \
   PPC::DFSTOREf32,                                                            \
   PPC::SPILLTOVSR_ST,                                                         \
   PPC::STXVP,                                                                 \
   PPC::SPILL_ACC,                                                             \
   PPC::SPILL_UACC,                                                            \
   PPC::SPILL_WACC,                                                            \
   PPC::SPILL_DMRP,                                                            \
   PPC::SPILL_DMR,                                                             \
   NoInstr,                                                                    \
   PPC::SPILL_QUADWORD}

// Initialize arrays for load and store spill opcodes on supported subtargets.
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 275-300

```cpp
#define StoreOpcodesForSpill                                                   \
  { Pwr8StoreOpcodes, Pwr9StoreOpcodes, Pwr10StoreOpcodes, FutureStoreOpcodes }
#define LoadOpcodesForSpill                                                    \
  { Pwr8LoadOpcodes, Pwr9LoadOpcodes, Pwr10LoadOpcodes, FutureLoadOpcodes }

class PPCSubtarget;
class PPCInstrInfo : public PPCGenInstrInfo {
  const PPCSubtarget &Subtarget;
  const PPCRegisterInfo RI;
  const unsigned StoreSpillOpcodesArray[4][SOK_LastOpcodeSpill] =
      StoreOpcodesForSpill;
  const unsigned LoadSpillOpcodesArray[4][SOK_LastOpcodeSpill] =
      LoadOpcodesForSpill;

  void StoreRegToStackSlot(MachineFunction &MF, unsigned SrcReg, bool isKill,
                           int FrameIdx, const TargetRegisterClass *RC,
                           SmallVectorImpl<MachineInstr *> &NewMIs) const;
  void LoadRegFromStackSlot(MachineFunction &MF, const DebugLoc &DL,
                            unsigned DestReg, int FrameIdx,
                            const TargetRegisterClass *RC,
                            SmallVectorImpl<MachineInstr *> &NewMIs) const;

  // Replace the instruction with single LI if possible. \p DefMI must be LI or
  // LI8.
  bool simplifyToLI(MachineInstr &MI, MachineInstr &DefMI,
                    unsigned OpNoForForwarding, MachineInstr **KilledDef,
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 301-326

```cpp
                    SmallSet<Register, 4> *RegsToUpdate = nullptr) const;
  // If the inst is imm-form and its register operand is produced by a ADDI, put
  // the imm into the inst directly and remove the ADDI if possible.
  bool transformToNewImmFormFedByAdd(MachineInstr &MI, MachineInstr &DefMI,
                                     unsigned OpNoForForwarding) const;
  // If the inst is x-form and has imm-form and one of its operand is produced
  // by a LI, put the imm into the inst directly and remove the LI if possible.
  bool transformToImmFormFedByLI(MachineInstr &MI, const ImmInstrInfo &III,
                                 unsigned ConstantOpNo,
                                 MachineInstr &DefMI) const;
  // If the inst is x-form and has imm-form and one of its operand is produced
  // by an add-immediate, try to transform it when possible.
  bool transformToImmFormFedByAdd(MachineInstr &MI, const ImmInstrInfo &III,
                                  unsigned ConstantOpNo, MachineInstr &DefMI,
                                  bool KillDefMI) const;
  // Try to find that, if the instruction 'MI' contains any operand that
  // could be forwarded from some inst that feeds it. If yes, return the
  // Def of that operand. And OpNoForForwarding is the operand index in
  // the 'MI' for that 'Def'. If we see another use of this Def between
  // the Def and the MI, SeenIntermediateUse becomes 'true'.
  MachineInstr *getForwardingDefMI(MachineInstr &MI,
                                   unsigned &OpNoForForwarding,
                                   bool &SeenIntermediateUse) const;

  // Can the user MI have it's source at index \p OpNoForForwarding
  // forwarded from an add-immediate that feeds it?
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "If the inst is imm-form and its register operand is produced by a ADDI, put". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“If the inst is imm-form and its register operand is produced by a ADDI, put”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 327-352

```cpp
  bool isUseMIElgibleForForwarding(MachineInstr &MI, const ImmInstrInfo &III,
                                   unsigned OpNoForForwarding) const;
  bool isDefMIElgibleForForwarding(MachineInstr &DefMI,
                                   const ImmInstrInfo &III,
                                   MachineOperand *&ImmMO,
                                   MachineOperand *&RegMO) const;
  bool isImmElgibleForForwarding(const MachineOperand &ImmMO,
                                 const MachineInstr &DefMI,
                                 const ImmInstrInfo &III,
                                 int64_t &Imm,
                                 int64_t BaseImm = 0) const;
  bool isRegElgibleForForwarding(const MachineOperand &RegMO,
                                 const MachineInstr &DefMI,
                                 const MachineInstr &MI, bool KillDefMI,
                                 bool &IsFwdFeederRegKilled,
                                 bool &SeenIntermediateUse) const;
  unsigned getSpillTarget() const;
  ArrayRef<unsigned> getStoreOpcodesForSpillArray() const;
  ArrayRef<unsigned> getLoadOpcodesForSpillArray() const;
  unsigned getSpillIndex(const TargetRegisterClass *RC) const;
  int16_t getFMAOpIdxInfo(unsigned Opcode) const;
  void reassociateFMA(MachineInstr &Root, unsigned Pattern,
                      SmallVectorImpl<MachineInstr *> &InsInstrs,
                      SmallVectorImpl<MachineInstr *> &DelInstrs,
                      DenseMap<Register, unsigned> &InstrIdxForVirtReg) const;
  Register
```
- **EN**: Declares function entry points including `isUseMIElgibleForForwarding`, `isDefMIElgibleForForwarding`, `isImmElgibleForForwarding` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `isUseMIElgibleForForwarding`, `isDefMIElgibleForForwarding`, `isImmElgibleForForwarding`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 353-378

```cpp
  generateLoadForNewConst(unsigned Idx, MachineInstr *MI, Type *Ty,
                          SmallVectorImpl<MachineInstr *> &InsInstrs) const;
  virtual void anchor();

protected:
  /// Commutes the operands in the given instruction.
  /// The commutable operands are specified by their indices OpIdx1 and OpIdx2.
  ///
  /// Do not call this method for a non-commutable instruction or for
  /// non-commutable pair of operand indices OpIdx1 and OpIdx2.
  /// Even though the instruction is commutable, the method may still
  /// fail to commute the operands, null pointer is returned in such cases.
  ///
  /// For example, we can commute rlwimi instructions, but only if the
  /// rotate amt is zero.  We also have to munge the immediates a bit.
  MachineInstr *commuteInstructionImpl(MachineInstr &MI, bool NewMI,
                                       unsigned OpIdx1,
                                       unsigned OpIdx2) const override;

public:
  explicit PPCInstrInfo(const PPCSubtarget &STI);

  bool isLoadFromConstantPool(MachineInstr *I) const;
  const Constant *getConstantFromConstantPool(MachineInstr *I) const;

  /// getRegisterInfo - TargetInstrInfo is a superset of MRegister info.  As
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Commutes the operands in the given instruction.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Commutes the operands in the given instruction.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 379-404

```cpp
  /// such, whenever a client has an instance of instruction info, it should
  /// always be able to get register info as well (through this method).
  ///
  const PPCRegisterInfo &getRegisterInfo() const { return RI; }

  bool isXFormMemOp(unsigned Opcode) const {
    return get(Opcode).TSFlags & PPCII::XFormMemOp;
  }
  bool isPrefixed(unsigned Opcode) const {
    return get(Opcode).TSFlags & PPCII::Prefixed;
  }
  bool isSExt32To64(unsigned Opcode) const {
    return get(Opcode).TSFlags & PPCII::SExt32To64;
  }
  bool isZExt32To64(unsigned Opcode) const {
    return get(Opcode).TSFlags & PPCII::ZExt32To64;
  }
  bool isMemriOp(unsigned Opcode) const {
    return get(Opcode).TSFlags & PPCII::MemriOp;
  }

  static bool isSameClassPhysRegCopy(unsigned Opcode) {
    unsigned CopyOpcodes[] = {PPC::OR,        PPC::OR8,   PPC::FMR,
                              PPC::VOR,       PPC::XXLOR, PPC::XXLORf,
                              PPC::XSCPSGNDP, PPC::MCRF,  PPC::CROR,
                              PPC::EVOR,      -1U};
```
- **EN**: Implements helper routine(s) `well`, `getRegisterInfo`, `isXFormMemOp` for this portion of the PowerPC backend instruction information and lowering helpers. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `well`, `getRegisterInfo`, `isXFormMemOp`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 405-430

```cpp
    for (int i = 0; CopyOpcodes[i] != -1U; i++)
      if (Opcode == CopyOpcodes[i])
        return true;
    return false;
  }

  static bool hasPCRelFlag(unsigned TF) {
    return TF == PPCII::MO_PCREL_FLAG || TF == PPCII::MO_GOT_TLSGD_PCREL_FLAG ||
           TF == PPCII::MO_GOT_TLSLD_PCREL_FLAG ||
           TF == PPCII::MO_GOT_TPREL_PCREL_FLAG ||
           TF == PPCII::MO_TPREL_PCREL_FLAG || TF == PPCII::MO_TLS_PCREL_FLAG ||
           TF == PPCII::MO_GOT_PCREL_FLAG;
  }

  static bool hasGOTFlag(unsigned TF) {
    return TF == PPCII::MO_GOT_FLAG || TF == PPCII::MO_GOT_TLSGD_PCREL_FLAG ||
           TF == PPCII::MO_GOT_TLSLD_PCREL_FLAG ||
           TF == PPCII::MO_GOT_TPREL_PCREL_FLAG ||
           TF == PPCII::MO_GOT_PCREL_FLAG;
  }

  static bool hasTLSFlag(unsigned TF) {
    return TF == PPCII::MO_TLSGD_FLAG || TF == PPCII::MO_TPREL_FLAG ||
           TF == PPCII::MO_TLSLD_FLAG || TF == PPCII::MO_TLSGDM_FLAG ||
           TF == PPCII::MO_GOT_TLSGD_PCREL_FLAG ||
           TF == PPCII::MO_GOT_TLSLD_PCREL_FLAG ||
```
- **EN**: Implements helper routine(s) `hasPCRelFlag`, `hasGOTFlag`, `hasTLSFlag` for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `hasPCRelFlag`, `hasGOTFlag`, `hasTLSFlag`。

### Lines 431-456

```cpp
           TF == PPCII::MO_GOT_TPREL_PCREL_FLAG || TF == PPCII::MO_TPREL_LO ||
           TF == PPCII::MO_TPREL_HA || TF == PPCII::MO_DTPREL_LO ||
           TF == PPCII::MO_TLSLD_LO || TF == PPCII::MO_TLS ||
           TF == PPCII::MO_TPREL_PCREL_FLAG || TF == PPCII::MO_TLS_PCREL_FLAG;
  }

  ScheduleHazardRecognizer *
  CreateTargetHazardRecognizer(const TargetSubtargetInfo *STI,
                               const ScheduleDAG *DAG) const override;
  ScheduleHazardRecognizer *
  CreateTargetPostRAHazardRecognizer(const InstrItineraryData *II,
                                     const ScheduleDAG *DAG) const override;

  unsigned getInstrLatency(const InstrItineraryData *ItinData,
                           const MachineInstr &MI,
                           unsigned *PredCost = nullptr) const override;

  std::optional<unsigned> getOperandLatency(const InstrItineraryData *ItinData,
                                            const MachineInstr &DefMI,
                                            unsigned DefIdx,
                                            const MachineInstr &UseMI,
                                            unsigned UseIdx) const override;
  std::optional<unsigned> getOperandLatency(const InstrItineraryData *ItinData,
                                            SDNode *DefNode, unsigned DefIdx,
                                            SDNode *UseNode,
                                            unsigned UseIdx) const override {
```
- **EN**: Implements helper routine(s) `CreateTargetHazardRecognizer`, `CreateTargetPostRAHazardRecognizer`, `getInstrLatency` for this portion of the PowerPC backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `CreateTargetHazardRecognizer`, `CreateTargetPostRAHazardRecognizer`, `getInstrLatency`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 457-482

```cpp
    return PPCGenInstrInfo::getOperandLatency(ItinData, DefNode, DefIdx,
                                              UseNode, UseIdx);
  }

  bool hasLowDefLatency(const TargetSchedModel &SchedModel,
                        const MachineInstr &DefMI,
                        unsigned DefIdx) const override {
    // Machine LICM should hoist all instructions in low-register-pressure
    // situations; none are sufficiently free to justify leaving in a loop
    // body.
    return false;
  }

  bool useMachineCombiner() const override {
    return true;
  }

  /// When getMachineCombinerPatterns() finds patterns, this function generates
  /// the instructions that could replace the original code sequence
  void genAlternativeCodeSequence(
      MachineInstr &Root, unsigned Pattern,
      SmallVectorImpl<MachineInstr *> &InsInstrs,
      SmallVectorImpl<MachineInstr *> &DelInstrs,
      DenseMap<Register, unsigned> &InstrIdxForVirtReg) const override;

  /// Return true when there is potentially a faster code sequence for a fma
```
- **EN**: Implements helper routine(s) `getOperandLatency`, `hasLowDefLatency`, `useMachineCombiner` for this portion of the PowerPC backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `getOperandLatency`, `hasLowDefLatency`, `useMachineCombiner`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 483-508

```cpp
  /// chain ending in \p Root. All potential patterns are output in the \p
  /// P array.
  bool getFMAPatterns(MachineInstr &Root, SmallVectorImpl<unsigned> &Patterns,
                      bool DoRegPressureReduce) const;

  CombinerObjective getCombinerObjective(unsigned Pattern) const override;

  /// Return true when there is potentially a faster code sequence
  /// for an instruction chain ending in <Root>. All potential patterns are
  /// output in the <Pattern> array.
  bool getMachineCombinerPatterns(MachineInstr &Root,
                                  SmallVectorImpl<unsigned> &Patterns,
                                  bool DoRegPressureReduce) const override;

  /// On PowerPC, we leverage machine combiner pass to reduce register pressure
  /// when the register pressure is high for one BB.
  /// Return true if register pressure for \p MBB is high and ABI is supported
  /// to reduce register pressure. Otherwise return false.
  bool shouldReduceRegisterPressure(
      const MachineBasicBlock *MBB,
      const RegisterClassInfo *RegClassInfo) const override;

  /// Fixup the placeholders we put in genAlternativeCodeSequence() for
  /// MachineCombiner.
  void
  finalizeInsInstrs(MachineInstr &Root, unsigned &Pattern,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "chain ending in \p Root. All potential patterns are output in the \p". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“chain ending in \p Root. All potential patterns are output in the \p”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 509-534

```cpp
                    SmallVectorImpl<MachineInstr *> &InsInstrs) const override;

  bool isAssociativeAndCommutative(const MachineInstr &Inst,
                                   bool Invert) const override;

  /// On PowerPC, we try to reassociate FMA chain which will increase
  /// instruction size. Set extension resource length limit to 1 for edge case.
  /// Resource Length is calculated by scaled resource usage in getCycles().
  /// Because of the division in getCycles(), it returns different cycles due to
  /// legacy scaled resource usage. So new resource length may be same with
  /// legacy or 1 bigger than legacy.
  /// We need to execlude the 1 bigger case even the resource length is not
  /// perserved for more FMA chain reassociations on PowerPC.
  int getExtendResourceLenLimit() const override { return 1; }

  // PowerPC specific version of setSpecialOperandAttr that copies Flags to MI
  // and clears nuw, nsw, and exact flags.
  using TargetInstrInfo::setSpecialOperandAttr;
  void setSpecialOperandAttr(MachineInstr &MI, uint32_t Flags) const;

  bool isCoalescableExtInstr(const MachineInstr &MI,
                             Register &SrcReg, Register &DstReg,
                             unsigned &SubIdx) const override;
  Register isLoadFromStackSlot(const MachineInstr &MI,
                               int &FrameIndex) const override;
  bool isReMaterializableImpl(const MachineInstr &MI) const override;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "On PowerPC, we try to reassociate FMA chain which will increase". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“On PowerPC, we try to reassociate FMA chain which will increase”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 535-560

```cpp
  Register isStoreToStackSlot(const MachineInstr &MI,
                              int &FrameIndex) const override;

  bool findCommutedOpIndices(const MachineInstr &MI, unsigned &SrcOpIdx1,
                             unsigned &SrcOpIdx2) const override;

  void insertNoop(MachineBasicBlock &MBB,
                  MachineBasicBlock::iterator MI) const override;


  // Branch analysis.
  bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
                     MachineBasicBlock *&FBB,
                     SmallVectorImpl<MachineOperand> &Cond,
                     bool AllowModify) const override;
  unsigned removeBranch(MachineBasicBlock &MBB,
                        int *BytesRemoved = nullptr) const override;
  unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
                        MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
                        const DebugLoc &DL,
                        int *BytesAdded = nullptr) const override;

  // Select analysis.
  bool canInsertSelect(const MachineBasicBlock &, ArrayRef<MachineOperand> Cond,
                       Register, Register, Register, int &, int &,
                       int &) const override;
```
- **EN**: Declares function entry points including `isStoreToStackSlot`, `findCommutedOpIndices`, `insertNoop` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `isStoreToStackSlot`, `findCommutedOpIndices`, `insertNoop`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 561-586

```cpp
  void insertSelect(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
                    const DebugLoc &DL, Register DstReg,
                    ArrayRef<MachineOperand> Cond, Register TrueReg,
                    Register FalseReg) const override;

  void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
                   const DebugLoc &DL, Register DestReg, Register SrcReg,
                   bool KillSrc, bool RenamableDest = false,
                   bool RenamableSrc = false) const override;

  void storeRegToStackSlot(
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
      bool isKill, int FrameIndex, const TargetRegisterClass *RC,

      Register VReg,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

  // Emits a register spill without updating the register class for vector
  // registers. This ensures that when we spill a vector register the
  // element order in the register is the same as it was in memory.
  void storeRegToStackSlotNoUpd(MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MBBI,
                                unsigned SrcReg, bool isKill, int FrameIndex,
                                const TargetRegisterClass *RC) const;

  void loadRegFromStackSlot(
```
- **EN**: Declares function entry points including `insertSelect`, `copyPhysReg`, `storeRegToStackSlot` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `insertSelect`, `copyPhysReg`, `storeRegToStackSlot`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 587-612

```cpp
      MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
      Register DestReg, int FrameIndex, const TargetRegisterClass *RC,
      Register VReg, unsigned SubReg = 0,
      MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;

  // Emits a register reload without updating the register class for vector
  // registers. This ensures that when we reload a vector register the
  // element order in the register is the same as it was in memory.
  void loadRegFromStackSlotNoUpd(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MBBI,
                                 unsigned DestReg, int FrameIndex,
                                 const TargetRegisterClass *RC) const;

  unsigned getStoreOpcodeForSpill(const TargetRegisterClass *RC) const;

  unsigned getLoadOpcodeForSpill(const TargetRegisterClass *RC) const;

  bool
  reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const override;

  bool foldImmediate(MachineInstr &UseMI, MachineInstr &DefMI, Register Reg,
                     MachineRegisterInfo *MRI) const override;

  bool onlyFoldImmediate(MachineInstr &UseMI, MachineInstr &DefMI,
                         Register Reg) const;
```
- **EN**: Declares function entry points including `loadRegFromStackSlotNoUpd`, `getStoreOpcodeForSpill`, `getLoadOpcodeForSpill` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `loadRegFromStackSlotNoUpd`, `getStoreOpcodeForSpill`, `getLoadOpcodeForSpill`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 613-638

```cpp
  // If conversion by predication (only supported by some branch instructions).
  // All of the profitability checks always return true; it is always
  // profitable to use the predicated branches.
  bool isProfitableToIfCvt(MachineBasicBlock &MBB,
                          unsigned NumCycles, unsigned ExtraPredCycles,
                          BranchProbability Probability) const override {
    return true;
  }

  bool isProfitableToIfCvt(MachineBasicBlock &TMBB,
                           unsigned NumT, unsigned ExtraT,
                           MachineBasicBlock &FMBB,
                           unsigned NumF, unsigned ExtraF,
                           BranchProbability Probability) const override;

  bool isProfitableToDupForIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
                                 BranchProbability Probability) const override {
    return true;
  }

  bool isProfitableToUnpredicate(MachineBasicBlock &TMBB,
                                 MachineBasicBlock &FMBB) const override {
    return false;
  }

  // Predication support.
```
- **EN**: Implements helper routine(s) `predication`, `isProfitableToIfCvt`, `isProfitableToDupForIfCvt` for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `predication`, `isProfitableToIfCvt`, `isProfitableToDupForIfCvt`。

### Lines 639-664

```cpp
  bool isPredicated(const MachineInstr &MI) const override;

  bool isSchedulingBoundary(const MachineInstr &MI,
                            const MachineBasicBlock *MBB,
                            const MachineFunction &MF) const override;

  bool PredicateInstruction(MachineInstr &MI,
                            ArrayRef<MachineOperand> Pred) const override;

  bool SubsumesPredicate(ArrayRef<MachineOperand> Pred1,
                         ArrayRef<MachineOperand> Pred2) const override;

  bool ClobbersPredicate(MachineInstr &MI, std::vector<MachineOperand> &Pred,
                         bool SkipDead) const override;

  // Comparison optimization.

  bool analyzeCompare(const MachineInstr &MI, Register &SrcReg,
                      Register &SrcReg2, int64_t &Mask,
                      int64_t &Value) const override;

  bool optimizeCompareInstr(MachineInstr &CmpInstr, Register SrcReg,
                            Register SrcReg2, int64_t Mask, int64_t Value,
                            const MachineRegisterInfo *MRI) const override;
```
- **EN**: Declares function entry points including `isPredicated`, `isSchedulingBoundary`, `PredicateInstruction` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `isPredicated`, `isSchedulingBoundary`, `PredicateInstruction`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 665-690

```cpp
  /// Return true if get the base operand, byte offset of an instruction and
  /// the memory width. Width is the size of memory that is being
  /// loaded/stored (e.g. 1, 2, 4, 8).
  bool getMemOperandWithOffsetWidth(const MachineInstr &LdSt,
                                    const MachineOperand *&BaseOp,
                                    int64_t &Offset, LocationSize &Width,
                                    const TargetRegisterInfo *TRI) const;

  bool optimizeCmpPostRA(MachineInstr &MI) const;

  /// Get the base operand and byte offset of an instruction that reads/writes
  /// memory.
  bool getMemOperandsWithOffsetWidth(
      const MachineInstr &LdSt,
      SmallVectorImpl<const MachineOperand *> &BaseOps, int64_t &Offset,
      bool &OffsetIsScalable, LocationSize &Width,
      const TargetRegisterInfo *TRI) const override;

  /// Returns true if the two given memory operations should be scheduled
  /// adjacent.
  bool shouldClusterMemOps(ArrayRef<const MachineOperand *> BaseOps1,
                           int64_t Offset1, bool OffsetIsScalable1,
                           ArrayRef<const MachineOperand *> BaseOps2,
                           int64_t Offset2, bool OffsetIsScalable2,
                           unsigned ClusterSize,
                           unsigned NumBytes) const override;
```
- **EN**: Declares function entry points including `stored`, `getMemOperandWithOffsetWidth`, `optimizeCmpPostRA` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `stored`, `getMemOperandWithOffsetWidth`, `optimizeCmpPostRA`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 691-716

```cpp

  /// Return true if two MIs access different memory addresses and false
  /// otherwise
  bool
  areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
                                  const MachineInstr &MIb) const override;

  /// GetInstSize - Return the number of bytes of code the specified
  /// instruction may be.  This returns the maximum number of bytes.
  ///
  unsigned getInstSizeInBytes(const MachineInstr &MI) const override;

  InstSizeVerifyMode
  getInstSizeVerifyMode(const MachineInstr &MI) const override;

  MCInst getNop() const override;

  std::pair<unsigned, unsigned>
  decomposeMachineOperandsTargetFlags(unsigned TF) const override;

  ArrayRef<std::pair<unsigned, const char *>>
  getSerializableDirectMachineOperandTargetFlags() const override;

  // Expand VSX Memory Pseudo instruction to either a VSX or a FP instruction.
  bool expandVSXMemPseudo(MachineInstr &MI) const;
```
- **EN**: Declares function entry points including `areMemAccessesTriviallyDisjoint`, `getInstSizeInBytes`, `getInstSizeVerifyMode` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `areMemAccessesTriviallyDisjoint`, `getInstSizeInBytes`, `getInstSizeVerifyMode`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 717-742

```cpp
  // Lower pseudo instructions after register allocation.
  bool expandPostRAPseudo(MachineInstr &MI) const override;
  bool expandAMOCSNEPseudo(MachineInstr &MI) const;

  const TargetRegisterClass *updatedRC(const TargetRegisterClass *RC) const;
  static int getRecordFormOpcode(unsigned Opcode);

  bool isTOCSaveMI(const MachineInstr &MI) const;

  std::pair<bool, bool>
  isSignOrZeroExtended(const unsigned Reg, const unsigned BinOpDepth,
                       const MachineRegisterInfo *MRI) const;

  // Return true if the register is sign-extended from 32 to 64 bits.
  bool isSignExtended(const unsigned Reg,
                      const MachineRegisterInfo *MRI) const {
    return isSignOrZeroExtended(Reg, 0, MRI).first;
  }

  // Return true if the register is zero-extended from 32 to 64 bits.
  bool isZeroExtended(const unsigned Reg,
                      const MachineRegisterInfo *MRI) const {
    return isSignOrZeroExtended(Reg, 0, MRI).second;
  }
  void promoteInstr32To64ForElimEXTSW(const Register &Reg,
                                      MachineRegisterInfo *MRI,
```
- **EN**: Implements helper routine(s) `expandPostRAPseudo`, `expandAMOCSNEPseudo`, `updatedRC` for this portion of the PowerPC backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `expandPostRAPseudo`, `expandAMOCSNEPseudo`, `updatedRC`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 743-768

```cpp
                                      unsigned BinOpDepth,
                                      LiveVariables *LV) const;

  bool convertToImmediateForm(MachineInstr &MI,
                              SmallSet<Register, 4> &RegsToUpdate,
                              MachineInstr **KilledDef = nullptr) const;
  bool foldFrameOffset(MachineInstr &MI) const;
  bool combineRLWINM(MachineInstr &MI, MachineInstr **ToErase = nullptr) const;
  bool isADDIInstrEligibleForFolding(MachineInstr &ADDIMI, int64_t &Imm) const;
  bool isADDInstrEligibleForFolding(MachineInstr &ADDMI) const;
  bool isImmInstrEligibleForFolding(MachineInstr &MI, unsigned &BaseReg,
                                    unsigned &XFormOpcode,
                                    int64_t &OffsetOfImmInstr,
                                    ImmInstrInfo &III) const;
  bool isValidToBeChangedReg(MachineInstr *ADDMI, unsigned Index,
                             MachineInstr *&ADDIMI, int64_t &OffsetAddi,
                             int64_t OffsetImm) const;

  void replaceInstrWithLI(MachineInstr &MI, const LoadImmediateInfo &LII) const;
  void replaceInstrOperandWithImm(MachineInstr &MI, unsigned OpNo,
                                  int64_t Imm) const;

  bool instrHasImmForm(unsigned Opc, bool IsVFReg, ImmInstrInfo &III,
                       bool PostRA) const;

  // In PostRA phase, try to find instruction defines \p Reg before \p MI.
```
- **EN**: Declares function entry points including `convertToImmediateForm`, `foldFrameOffset`, `combineRLWINM` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `convertToImmediateForm`, `foldFrameOffset`, `combineRLWINM`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 769-794

```cpp
  // \p SeenIntermediate is set to true if uses between DefMI and \p MI exist.
  MachineInstr *getDefMIPostRA(unsigned Reg, MachineInstr &MI,
                               bool &SeenIntermediateUse) const;

  // Materialize immediate after RA.
  void materializeImmPostRA(MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator MBBI,
                            const DebugLoc &DL, Register Reg,
                            int64_t Imm) const;

  /// Check \p Opcode is BDNZ (Decrement CTR and branch if it is still nonzero).
  bool isBDNZ(unsigned Opcode) const;

  /// Find the hardware loop instruction used to set-up the specified loop.
  /// On PPC, we have two instructions used to set-up the hardware loop
  /// (MTCTRloop, MTCTR8loop) with corresponding endloop (BDNZ, BDNZ8)
  /// instructions to indicate the end of a loop.
  MachineInstr *
  findLoopInstr(MachineBasicBlock &PreHeader,
                SmallPtrSet<MachineBasicBlock *, 8> &Visited) const;

  /// Analyze loop L, which must be a single-basic-block loop, and if the
  /// conditions can be understood enough produce a PipelinerLoopInfo object.
  std::unique_ptr<TargetInstrInfo::PipelinerLoopInfo>
  analyzeLoopForPipelining(MachineBasicBlock *LoopBB) const override;
};
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\p SeenIntermediate is set to true if uses between DefMI and \p MI exist.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\p SeenIntermediate is set to true if uses between DefMI and \p MI exist.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 795-798

```cpp

}

#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Instruction information and lowering helpers / 指令信息与 lowering 辅助逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCMCTargetDesc.h`
- `PPC.h`
- `PPCRegisterInfo.h`
- `llvm/ADT/SmallSet.h`
- `llvm/CodeGen/LiveVariables.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `PPCGenInstrInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
