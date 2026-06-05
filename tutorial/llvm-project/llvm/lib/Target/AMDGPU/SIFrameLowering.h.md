# SIFrameLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIFrameLowering.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SIFrameLowering in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SIFrameLowering 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===--------------------- SIFrameLowering.h --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_SIFRAMELOWERING_H
#define LLVM_LIB_TARGET_AMDGPU_SIFRAMELOWERING_H

#include "AMDGPUFrameLowering.h"
#include "SIRegisterInfo.h"

namespace llvm {

class SIFrameLowering final : public AMDGPUFrameLowering {
public:
  SIFrameLowering(StackDirection D, Align StackAl, int LAO,
                  Align TransAl = Align(1))
      : AMDGPUFrameLowering(D, StackAl, LAO, TransAl) {}
  ~SIFrameLowering() override = default;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIFrameLowering`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIFrameLowering`。

### Lines 24-47: Defines emitEntryFunctionPrologue
```cpp
  void emitEntryFunctionPrologue(MachineFunction &MF,
                                 MachineBasicBlock &MBB) const;
  void emitPrologue(MachineFunction &MF,
                    MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF,
                    MachineBasicBlock &MBB) const override;
  StackOffset getFrameIndexReference(const MachineFunction &MF, int FI,
                                     Register &FrameReg) const override;

  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS = nullptr) const override;
  void determineCalleeSavesSGPR(MachineFunction &MF, BitVector &SavedRegs,
                                RegScavenger *RS = nullptr) const;
  void determinePrologEpilogSGPRSaves(MachineFunction &MF, BitVector &SavedRegs,
                                      bool NeedExecCopyReservedReg) const;
  void emitCSRSpillStores(MachineFunction &MF, MachineBasicBlock &MBB,
                          MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                          LiveRegUnits &LiveUnits, Register FrameReg,
                          Register FramePtrRegScratchCopy) const;
  void emitCSRSpillRestores(MachineFunction &MF, MachineBasicBlock &MBB,
                            MachineBasicBlock::iterator MBBI, DebugLoc &DL,
                            LiveRegUnits &LiveUnits, Register FrameReg,
                            Register FramePtrRegScratchCopy) const;
  bool
```
**EN:** This section contains concrete logic for emitEntryFunctionPrologue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 emitEntryFunctionPrologue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 48-71: Defines assignCalleeSavedSpillSlots
```cpp
  assignCalleeSavedSpillSlots(MachineFunction &MF,
                              const TargetRegisterInfo *TRI,
                              std::vector<CalleeSavedInfo> &CSI) const override;

  bool assignCalleeSavedSpillSlotsImpl(MachineFunction &MF,
                                       const TargetRegisterInfo *TRI,
                                       std::vector<CalleeSavedInfo> &CSI) const;

  bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MI,
                                 ArrayRef<CalleeSavedInfo> CSI,
                                 const TargetRegisterInfo *TRI) const override;

  bool
  restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator MI,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              const TargetRegisterInfo *TRI) const override;

  bool allocateScavengingFrameIndexesNearIncomingSP(
    const MachineFunction &MF) const override;

  bool isSupportedStackID(TargetStackID::Value ID) const override;

```
**EN:** This section contains concrete logic for assignCalleeSavedSpillSlots. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 assignCalleeSavedSpillSlots 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 72-95: Defines processFunctionBeforeFrameFinalized
```cpp
  void processFunctionBeforeFrameFinalized(
    MachineFunction &MF,
    RegScavenger *RS = nullptr) const override;

  void processFunctionBeforeFrameIndicesReplaced(
      MachineFunction &MF, RegScavenger *RS = nullptr) const override;

  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF,
                                MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator MI) const override;

protected:
  bool hasFPImpl(const MachineFunction &MF) const override;

private:
  void emitEntryFunctionFlatScratchInit(MachineFunction &MF,
                                        MachineBasicBlock &MBB,
                                        MachineBasicBlock::iterator I,
                                        const DebugLoc &DL,
                                        Register ScratchWaveOffsetReg) const;

  Register getEntryFunctionReservedScratchRsrcReg(MachineFunction &MF) const;

```
**EN:** This section contains concrete logic for processFunctionBeforeFrameFinalized. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 processFunctionBeforeFrameFinalized 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 96-112: Preprocessor guards and macros
```cpp
  void emitEntryFunctionScratchRsrcRegSetup(
      MachineFunction &MF, MachineBasicBlock &MBB,
      MachineBasicBlock::iterator I, const DebugLoc &DL,
      Register PreloadedPrivateBufferReg, Register ScratchRsrcReg,
      Register ScratchWaveOffsetReg) const;

public:
  bool requiresStackPointerReference(const MachineFunction &MF) const;

  // Returns true if the function may need to reserve space on the stack for the
  // CWSR trap handler.
  bool mayReserveScratchForCWSR(const MachineFunction &MF) const;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_SIFRAMELOWERING_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `SIFrameLowering`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUFrameLowering.h"`
- `"SIRegisterInfo.h"`
