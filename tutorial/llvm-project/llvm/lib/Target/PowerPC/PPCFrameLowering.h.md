# PPCFrameLowering.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCFrameLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCFrameLowering.h - Define frame lowering for PowerPC.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCFrameLowering.h`，主要负责 PowerPC 后端的栈帧 lowering 逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCFrameLowering.h - Define frame lowering for PowerPC --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 该区间与栈帧布局或栈访问相关。

### Lines 7-9

```cpp
//===----------------------------------------------------------------------===//
//
//
```
- **EN**: Continues the PowerPC backend stack frame lowering logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的栈帧 lowering 逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 10-16

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCFRAMELOWERING_H
#define LLVM_LIB_TARGET_POWERPC_PPCFRAMELOWERING_H

#include "llvm/ADT/STLExtras.h"
#include "llvm/CodeGen/TargetFrameLowering.h"
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 17-42

```cpp
#include "llvm/Target/TargetMachine.h"

namespace llvm {
class PPCSubtarget;

class PPCFrameLowering: public TargetFrameLowering {
  const PPCSubtarget &Subtarget;
  const uint64_t ReturnSaveOffset;
  const uint64_t TOCSaveOffset;
  const uint64_t FramePointerSaveOffset;
  const unsigned LinkageSize;
  const uint64_t BasePointerSaveOffset;
  const uint64_t CRSaveOffset;

  // Map each group of one or two GPRs to corresponding VSR for spilling.
  // TODO: Use local table in methods to avoid this mutable member.
  mutable DenseMap<unsigned, std::pair<Register, Register>> VSRContainingGPRs;

  /**
   * Find register[s] that can be used in function prologue and epilogue
   *
   * Find register[s] that can be use as scratch register[s] in function
   * prologue and epilogue to save various registers (Link Register, Base
   * Pointer, etc.). Prefer R0/R12, if available. Otherwise choose whatever
   * register[s] are available.
   *
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 43-68

```cpp
   * This method will return true if it is able to find enough unique scratch
   * registers (1 or 2 depending on the requirement). If it is unable to find
   * enough available registers in the block, it will return false and set
   * any passed output parameter that corresponds to a required unique register
   * to PPC::NoRegister.
   *
   * \param[in] MBB The machine basic block to find an available register for
   * \param[in] UseAtEnd Specify whether the scratch register will be used at
   *                     the end of the basic block (i.e., will the scratch
   *                     register kill a register defined in the basic block)
   * \param[in] TwoUniqueRegsRequired Specify whether this basic block will
   *                                  require two unique scratch registers.
   * \param[out] SR1 The scratch register to use
   * \param[out] SR2 The second scratch register. If this pointer is not null
   *                 the function will attempt to set it to an available
   *                 register regardless of whether there is a hard requirement
   *                 for two unique scratch registers.
   * \return true if the required number of registers was found.
   *         false if the required number of scratch register weren't available.
   *         If either output parameter refers to a required scratch register
   *         that isn't available, it will be set to an invalid value.
   */
  bool findScratchRegister(MachineBasicBlock *MBB,
                           bool UseAtEnd,
                           bool TwoUniqueRegsRequired = false,
                           Register *SR1 = nullptr,
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This method will return true if it is able to find enough unique scratch". Notable symbols in this range include `registers`, `block`, `findScratchRegister`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This method will return true if it is able to find enough unique scratch”。 该区间中较显眼的符号包括 `registers`, `block`, `findScratchRegister`。

### Lines 69-94

```cpp
                           Register *SR2 = nullptr) const;
  bool twoUniqueScratchRegsRequired(MachineBasicBlock *MBB) const;

  /**
   * Create branch instruction for PPC::TCRETURN* (tail call return)
   *
   * \param[in] MBB that is terminated by PPC::TCRETURN*
   */
  void createTailCallBranchInstr(MachineBasicBlock &MBB) const;

  /**
    * Check if the conditions are correct to allow for the stack update
    * to be moved past the CSR save/restore code.
    */
  bool stackUpdateCanBeMoved(MachineFunction &MF) const;

public:
  PPCFrameLowering(const PPCSubtarget &STI);

  /**
   * Determine the frame layout and update the machine function.
   */
  uint64_t determineFrameLayoutAndUpdate(MachineFunction &MF,
                                         bool UseEstimate = false) const;

  /**
```
- **EN**: Declares function entry points including `twoUniqueScratchRegsRequired`, `createTailCallBranchInstr`, `stackUpdateCanBeMoved` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `twoUniqueScratchRegsRequired`, `createTailCallBranchInstr`, `stackUpdateCanBeMoved`。 该区间与栈帧布局或栈访问相关。

### Lines 95-120

```cpp
   * Determine the frame layout but do not update the machine function.
   * The MachineFunction object can be const in this case as it is not
   * modified.
   */
  uint64_t determineFrameLayout(const MachineFunction &MF,
                                bool UseEstimate = false,
                                unsigned *NewMaxCallFrameSize = nullptr) const;

  /// emitProlog/emitEpilog - These methods insert prolog and epilog code into
  /// the function.
  void emitPrologue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void emitEpilogue(MachineFunction &MF, MachineBasicBlock &MBB) const override;
  void inlineStackProbe(MachineFunction &MF,
                        MachineBasicBlock &PrologMBB) const override;

  bool needsFP(const MachineFunction &MF) const;
  void replaceFPWithRealFP(MachineFunction &MF) const;

  void determineCalleeSaves(MachineFunction &MF, BitVector &SavedRegs,
                            RegScavenger *RS = nullptr) const override;
  void processFunctionBeforeFrameFinalized(MachineFunction &MF,
                                     RegScavenger *RS = nullptr) const override;
  void addScavengingSpillSlot(MachineFunction &MF, RegScavenger *RS) const;

  bool spillCalleeSavedRegisters(MachineBasicBlock &MBB,
                                 MachineBasicBlock::iterator MI,
```
- **EN**: Declares function entry points including `determineFrameLayout`, `emitPrologue`, `emitEpilogue` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `determineFrameLayout`, `emitPrologue`, `emitEpilogue`。 该区间与栈帧布局或栈访问相关。

### Lines 121-146

```cpp
                                 ArrayRef<CalleeSavedInfo> CSI,
                                 const TargetRegisterInfo *TRI) const override;
  /// This function will assign callee saved gprs to volatile vector registers
  /// for prologue spills when applicable. It returns false if there are any
  /// registers which were not spilled to volatile vector registers.
  bool
  assignCalleeSavedSpillSlots(MachineFunction &MF,
                              const TargetRegisterInfo *TRI,
                              std::vector<CalleeSavedInfo> &CSI) const override;

  MachineBasicBlock::iterator
  eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                                MachineBasicBlock::iterator I) const override;

  bool
  restoreCalleeSavedRegisters(MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator MI,
                              MutableArrayRef<CalleeSavedInfo> CSI,
                              const TargetRegisterInfo *TRI) const override;

  /// targetHandlesStackFrameRounding - Returns true if the target is
  /// responsible for rounding up the stack frame (probably at emitPrologue
  /// time).
  bool targetHandlesStackFrameRounding() const override { return true; }

  /// getReturnSaveOffset - Return the previous frame offset to save the
```
- **EN**: Implements helper routine(s) `assignCalleeSavedSpillSlots`, `eliminateCallFramePseudoInstr`, `restoreCalleeSavedRegisters` for this portion of the PowerPC backend stack frame lowering logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分栈帧 lowering 逻辑所需的辅助例程 `assignCalleeSavedSpillSlots`, `eliminateCallFramePseudoInstr`, `restoreCalleeSavedRegisters`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 147-172

```cpp
  /// return address.
  uint64_t getReturnSaveOffset() const { return ReturnSaveOffset; }

  /// getTOCSaveOffset - Return the previous frame offset to save the
  /// TOC register -- 64-bit SVR4 ABI only.
  uint64_t getTOCSaveOffset() const;

  /// getFramePointerSaveOffset - Return the previous frame offset to save the
  /// frame pointer.
  uint64_t getFramePointerSaveOffset() const;

  /// getBasePointerSaveOffset - Return the previous frame offset to save the
  /// base pointer.
  uint64_t getBasePointerSaveOffset() const;

  /// getLinkageSize - Return the size of the PowerPC ABI linkage area.
  ///
  unsigned getLinkageSize() const { return LinkageSize; }

  const SpillSlot *
  getCalleeSavedSpillSlots(unsigned &NumEntries) const override;

  bool enableShrinkWrapping(const MachineFunction &MF) const override;

  /// Methods used by shrink wrapping to determine if MBB can be used for the
  /// function prologue/epilogue.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "return address.". Notable symbols in this range include `getReturnSaveOffset`, `getTOCSaveOffset`, `getFramePointerSaveOffset`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“return address.”。 该区间中较显眼的符号包括 `getReturnSaveOffset`, `getTOCSaveOffset`, `getFramePointerSaveOffset`。

### Lines 173-183

```cpp
  bool canUseAsPrologue(const MachineBasicBlock &MBB) const override;
  bool canUseAsEpilogue(const MachineBasicBlock &MBB) const override;
  void updateCalleeSaves(const MachineFunction &MF, BitVector &SavedRegs) const;

  uint64_t getStackThreshold() const override;

protected:
  bool hasFPImpl(const MachineFunction &MF) const override;
};
} // End llvm namespace
```
- **EN**: Declares function entry points including `canUseAsPrologue`, `canUseAsEpilogue`, `updateCalleeSaves` that other backend components call later. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `canUseAsPrologue`, `canUseAsEpilogue`, `updateCalleeSaves`。 该区间与栈帧布局或栈访问相关。

### Lines 184-184

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Stack frame lowering logic / 栈帧 lowering 逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/ADT/STLExtras.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/Target/TargetMachine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
