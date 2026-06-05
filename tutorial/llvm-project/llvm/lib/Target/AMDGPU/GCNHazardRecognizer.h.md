# GCNHazardRecognizer.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNHazardRecognizer.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for GCNHazardRecognizer in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 GCNHazardRecognizer 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: File banner, includes, and setup
```cpp
//===-- GCNHazardRecognizers.h - GCN Hazard Recognizers ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines hazard recognizers for scheduling on GCN processors.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPUHAZARDRECOGNIZERS_H
#define LLVM_LIB_TARGET_AMDGPUHAZARDRECOGNIZERS_H

#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/ScheduleHazardRecognizer.h"
#include "llvm/CodeGen/TargetSchedule.h"
#include <list>

namespace llvm {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 25-42: Declares class MachineFunction
```cpp
class MachineFunction;
class MachineInstr;
class MachineOperand;
class MachineRegisterInfo;
class SIInstrInfo;
class SIRegisterInfo;
class GCNSubtarget;

class GCNHazardRecognizer final : public ScheduleHazardRecognizer {
public:
  typedef function_ref<bool(const MachineInstr &)> IsHazardFn;
  typedef function_ref<bool(const MachineInstr &, int WaitStates)> IsExpiredFn;
  typedef function_ref<unsigned int(const MachineInstr &)> GetNumWaitStatesFn;

private:
  // Distinguish if we are called from scheduler or hazard recognizer
  bool IsHazardRecognizerMode;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MachineFunction`, `MachineInstr`, `MachineOperand`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MachineFunction`, `MachineInstr`, `MachineOperand`。

### Lines 43-64: Implementation details and local logic
```cpp
  // This variable stores the instruction that has been emitted this cycle. It
  // will be added to EmittedInstrs, when AdvanceCycle() or RecedeCycle() is
  // called.
  MachineInstr *CurrCycleInstr;
  std::list<MachineInstr*> EmittedInstrs;
  const MachineFunction &MF;
  const GCNSubtarget &ST;
  const SIInstrInfo &TII;
  const SIRegisterInfo &TRI;
  const TargetSchedModel &TSchedModel;

  // Loop info for V_NOP hoisting, passed from the pass manager.
  MachineLoopInfo *MLI = nullptr;

  bool RunLdsBranchVmemWARHazardFixup;

  /// RegUnits of uses in the current soft memory clause.
  mutable BitVector ClauseUses;

  /// RegUnits of defs in the current soft memory clause.
  mutable BitVector ClauseDefs;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 65-83: Defines resetClause
```cpp
  void resetClause() const {
    ClauseUses.reset();
    ClauseDefs.reset();
  }

  void addClauseInst(const MachineInstr &MI) const;

  /// \returns the number of wait states before another MFMA instruction can be
  /// issued after \p MI.
  unsigned getMFMAPipelineWaitStates(const MachineInstr &MI) const;

  // Advance over a MachineInstr bundle. Look for hazards in the bundled
  // instructions.
  void processBundle();

  // Run on an individual instruction in hazard recognizer mode. This can be
  // used on a newly inserted instruction before returning from PreEmitNoops.
  void runOnInstruction(MachineInstr *MI);

```
**EN:** This section contains concrete logic for resetClause. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 resetClause 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 84-107: Defines getWaitStatesSince
```cpp
  int getWaitStatesSince(IsHazardFn IsHazard, int Limit,
                         GetNumWaitStatesFn GetNumWaitStates) const;
  int getWaitStatesSince(IsHazardFn IsHazard, int Limit) const;
  int getWaitStatesSinceDef(unsigned Reg, IsHazardFn IsHazardDef,
                            int Limit) const;
  int getWaitStatesSinceSetReg(IsHazardFn IsHazard, int Limit) const;

  int checkSoftClauseHazards(MachineInstr *SMEM) const;
  int checkSMRDHazards(MachineInstr *SMRD) const;
  int checkVMEMHazards(MachineInstr *VMEM) const;
  int checkDPPHazards(MachineInstr *DPP) const;
  int checkDivFMasHazards(MachineInstr *DivFMas) const;
  int checkGetRegHazards(MachineInstr *GetRegInstr) const;
  int checkSetRegHazards(MachineInstr *SetRegInstr) const;
  int createsVALUHazard(const MachineInstr &MI) const;
  int checkVALUHazards(MachineInstr *VALU) const;
  int checkVALUHazardsHelper(const MachineOperand &Def,
                             const MachineRegisterInfo &MRI) const;
  int checkRWLaneHazards(MachineInstr *RWLane) const;
  int checkRFEHazards(MachineInstr *RFE) const;
  int checkInlineAsmHazards(MachineInstr *IA) const;
  int checkReadM0Hazards(MachineInstr *SMovRel) const;
  int checkNSAtoVMEMHazard(MachineInstr *MI) const;
  int checkFPAtomicToDenormModeHazard(MachineInstr *MI) const;
```
**EN:** This section contains concrete logic for getWaitStatesSince. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getWaitStatesSince 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 108-131: Defines emitVNops
```cpp
  // Emit \p WaitStatesNeeded V_NOP instructions before \p InsertPt.
  // If IsHoisting is true, uses empty DebugLoc for compiler-inserted NOPs.
  void emitVNops(MachineBasicBlock &MBB, MachineBasicBlock::iterator InsertPt,
                 int WaitStatesNeeded, bool IsHoisting = false);
  void fixHazards(MachineInstr *MI);
  bool fixVcmpxPermlaneHazards(MachineInstr *MI);
  bool fixVMEMtoScalarWriteHazards(MachineInstr *MI);
  bool fixSMEMtoVectorWriteHazards(MachineInstr *MI);
  bool fixVcmpxExecWARHazard(MachineInstr *MI);
  bool fixLdsBranchVmemWARHazard(MachineInstr *MI);
  bool fixLdsDirectVALUHazard(MachineInstr *MI);
  bool fixLdsDirectVMEMHazard(MachineInstr *MI);
  bool fixVALUPartialForwardingHazard(MachineInstr *MI);
  bool fixVALUTransUseHazard(MachineInstr *MI);
  bool fixVALUTransCoexecutionHazards(MachineInstr *MI);
  bool fixWMMAHazards(MachineInstr *MI);
  int checkWMMACoexecutionHazards(MachineInstr *MI) const;
  bool fixWMMACoexecutionHazards(MachineInstr *MI);
  bool tryHoistWMMAVnopsFromLoop(MachineInstr *MI, int WaitStatesNeeded);
  bool hasWMMAHazardInLoop(MachineLoop *L, MachineInstr *MI,
                           bool IncludeSubloops = true);
  bool hasWMMAToWMMARegOverlap(const MachineInstr &WMMA,
                               const MachineInstr &MI) const;
  bool hasWMMAToVALURegOverlap(const MachineInstr &WMMA,
```
**EN:** This section contains concrete logic for emitVNops. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 emitVNops 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 132-155: Defines isCoexecutionHazardFor
```cpp
                               const MachineInstr &MI) const;
  bool isCoexecutionHazardFor(const MachineInstr &I,
                              const MachineInstr &MI) const;
  bool fixShift64HighRegBug(MachineInstr *MI);
  bool fixVALUMaskWriteHazard(MachineInstr *MI);
  bool fixRequiredExportPriority(MachineInstr *MI);
  bool fixGetRegWaitIdle(MachineInstr *MI);
  bool fixDsAtomicAsyncBarrierArriveB64(MachineInstr *MI);
  bool fixScratchBaseForwardingHazard(MachineInstr *MI);
  bool fixSetRegMode(MachineInstr *MI);

  int checkMAIHazards(MachineInstr *MI) const;
  int checkMAIHazards908(MachineInstr *MI) const;
  int checkMAIHazards90A(MachineInstr *MI) const;
  /// Pad the latency between neighboring MFMA instructions with s_nops. The
  /// percentage of wait states to fill with s_nops is specified by the command
  /// line option '-amdgpu-mfma-padding-ratio'.
  ///
  /// For example, with '-amdgpu-mfma-padding-ratio=100':
  ///
  /// 2 pass MFMA instructions have a latency of 2 wait states. Therefore, a
  /// 'S_NOP 1' will be added between sequential MFMA instructions.
  ///
  /// V_MFMA_F32_4X4X1F32
```
**EN:** This section contains concrete logic for isCoexecutionHazardFor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isCoexecutionHazardFor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 156-174: Declares checkMFMAPadding
```cpp
  /// V_MFMA_F32_4X4X1F32
  ///-->
  /// V_MFMA_F32_4X4X1F32
  /// S_NOP 1
  /// V_MFMA_F32_4X4X1F32
  int checkMFMAPadding(MachineInstr *MI) const;
  int checkMAIVALUHazards(MachineInstr *MI) const;
  int checkMAILdStHazards(MachineInstr *MI) const;
  int checkPermlaneHazards(MachineInstr *MI) const;

public:
  GCNHazardRecognizer(const MachineFunction &MF,
                      MachineLoopInfo *MLI = nullptr);
  // We can only issue one instruction per cycle.
  bool atIssueLimit() const override { return true; }
  void EmitInstruction(SUnit *SU) override;
  void EmitInstruction(MachineInstr *MI) override;
  HazardType getHazardType(SUnit *SU, int Stalls) override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 175-191: Preprocessor guards and macros
```cpp
  /// Returns the number of wait states until all hazards for \p MI are
  /// resolved. This is useful for scheduling heuristics that want
  /// cycle-accurate hazard information rather than just a boolean.  Unlike
  /// PreEmitNoops, this does not modify state or fix hazards.
  unsigned getHazardWaitStates(MachineInstr *MI) const;
  void EmitNoop() override;
  unsigned PreEmitNoops(MachineInstr *) override;
  unsigned PreEmitNoopsCommon(MachineInstr *) const;
  void AdvanceCycle() override;
  void RecedeCycle() override;
  bool ShouldPreferAnother(SUnit *SU) const override;
  void Reset() override;
};

} // end namespace llvm

#endif //LLVM_LIB_TARGET_AMDGPUHAZARDRECOGNIZERS_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MachineFunction`, `MachineInstr`, `MachineOperand`, `MachineRegisterInfo`, `SIInstrInfo`, `SIRegisterInfo`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/ADT/BitVector.h"`
- `"llvm/ADT/STLExtras.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/ScheduleHazardRecognizer.h"`
- `"llvm/CodeGen/TargetSchedule.h"`
- `<list>`
