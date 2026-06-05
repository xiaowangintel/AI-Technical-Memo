# PPCRegisterInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCRegisterInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCRegisterInfo.h - PowerPC Register Information Impl.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCRegisterInfo.h`，主要负责 PowerPC 后端的寄存器信息与栈帧交互逻辑。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCRegisterInfo.h - PowerPC Register Information Impl ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Register assignment and register-class constraints matter here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains the PowerPC implementation of the TargetRegisterInfo
// class.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains the PowerPC implementation of the TargetRegisterInfo". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains the PowerPC implementation of the TargetRegisterInfo”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 12-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCREGISTERINFO_H
#define LLVM_LIB_TARGET_POWERPC_PPCREGISTERINFO_H

#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "llvm/ADT/DenseMap.h"
```
- **EN**: Pulls in direct dependencies required by this register information and frame interaction logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该寄存器信息与栈帧交互逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-45

```cpp
#define GET_REGINFO_HEADER
#include "PPCGenRegisterInfo.inc"

namespace llvm {
class PPCTargetMachine;

inline static unsigned getCRFromCRBit(unsigned SrcReg) {
  unsigned Reg = 0;
  if (SrcReg == PPC::CR0LT || SrcReg == PPC::CR0GT ||
      SrcReg == PPC::CR0EQ || SrcReg == PPC::CR0UN)
    Reg = PPC::CR0;
  else if (SrcReg == PPC::CR1LT || SrcReg == PPC::CR1GT ||
           SrcReg == PPC::CR1EQ || SrcReg == PPC::CR1UN)
    Reg = PPC::CR1;
  else if (SrcReg == PPC::CR2LT || SrcReg == PPC::CR2GT ||
           SrcReg == PPC::CR2EQ || SrcReg == PPC::CR2UN)
    Reg = PPC::CR2;
  else if (SrcReg == PPC::CR3LT || SrcReg == PPC::CR3GT ||
           SrcReg == PPC::CR3EQ || SrcReg == PPC::CR3UN)
    Reg = PPC::CR3;
  else if (SrcReg == PPC::CR4LT || SrcReg == PPC::CR4GT ||
           SrcReg == PPC::CR4EQ || SrcReg == PPC::CR4UN)
    Reg = PPC::CR4;
  else if (SrcReg == PPC::CR5LT || SrcReg == PPC::CR5GT ||
           SrcReg == PPC::CR5EQ || SrcReg == PPC::CR5UN)
    Reg = PPC::CR5;
```
- **EN**: Pulls in direct dependencies required by this register information and frame interaction logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该寄存器信息与栈帧交互逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 46-56

```cpp
  else if (SrcReg == PPC::CR6LT || SrcReg == PPC::CR6GT ||
           SrcReg == PPC::CR6EQ || SrcReg == PPC::CR6UN)
    Reg = PPC::CR6;
  else if (SrcReg == PPC::CR7LT || SrcReg == PPC::CR7GT ||
           SrcReg == PPC::CR7EQ || SrcReg == PPC::CR7UN)
    Reg = PPC::CR7;

  assert(Reg != 0 && "Invalid CR bit register");
  return Reg;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 57-82

```cpp
class PPCRegisterInfo : public PPCGenRegisterInfo {
  DenseMap<unsigned, unsigned> ImmToIdxMap;
  const PPCTargetMachine &TM;

  void spillRegPair(MachineBasicBlock &MBB, MachineBasicBlock::iterator II,
                    DebugLoc DL, const TargetInstrInfo &TII,
                    unsigned FrameIndex, bool IsLittleEndian, bool IsKilled,
                    Register Reg, int Offset) const;

public:
  PPCRegisterInfo(const PPCTargetMachine &TM);

  /// getMappedIdxOpcForImmOpc - Return the mapped index form load/store opcode
  /// for a given imm form load/store opcode \p ImmFormOpcode.
  /// FIXME: move this to PPCInstrInfo class.
  unsigned getMappedIdxOpcForImmOpc(unsigned ImmOpcode) const {
    auto It = ImmToIdxMap.find(ImmOpcode);
    if (It == ImmToIdxMap.end())
      return PPC::INSTRUCTION_LIST_END;
    return It->second;
  }

  /// getPointerRegClass - Return the register class to use to hold pointers.
  /// This is used for addressing modes.
  const TargetRegisterClass *
  getPointerRegClass(unsigned Kind = 0) const override;
```
- **EN**: Declares a backend-facing type `PPCRegisterInfo`, `spillRegPair`, `getMappedIdxOpcForImmOpc` and outlines the API or state that nearby code will rely on. Register assignment and register-class constraints matter here.
- **CN**: 这里声明面向后端的类型 `PPCRegisterInfo`, `spillRegPair`, `getMappedIdxOpcForImmOpc`，并勾勒出周边代码会依赖的接口或状态。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 83-108

```cpp

  const TargetRegisterClass *
  getCrossCopyRegClass(const TargetRegisterClass *RC) const override;

  unsigned getRegPressureLimit(const TargetRegisterClass *RC,
                               MachineFunction &MF) const override;

  const TargetRegisterClass *
  getLargestLegalSuperClass(const TargetRegisterClass *RC,
                            const MachineFunction &MF) const override;

  /// Code Generation virtual methods...
  const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const override;
  const uint32_t *getCallPreservedMask(const MachineFunction &MF,
                                       CallingConv::ID CC) const override;
  const uint32_t *getNoPreservedMask() const override;

  void adjustStackMapLiveOutMask(uint32_t *Mask) const override;

  BitVector getReservedRegs(const MachineFunction &MF) const override;
  bool isAsmClobberable(const MachineFunction &MF,
                        MCRegister PhysReg) const override;
  bool isCallerPreservedPhysReg(MCRegister PhysReg,
                                const MachineFunction &MF) const override;

  // Provide hints to the register allocator for allocating subregisters
```
- **EN**: Declares function entry points including `getCrossCopyRegClass`, `getRegPressureLimit`, `getLargestLegalSuperClass` that other backend components call later. ABI and calling-convention details are important in this part of the code.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getCrossCopyRegClass`, `getRegPressureLimit`, `getLargestLegalSuperClass`。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 109-134

```cpp
  // of primed and unprimed accumulators. For example, if accumulator
  // ACC5 is assigned, we also want to assign UACC5 to the input.
  // Similarly if UACC5 is assigned, we want to assign VSRp10, VSRp11
  // to its inputs.
  bool getRegAllocationHints(Register VirtReg, ArrayRef<MCPhysReg> Order,
                             SmallVectorImpl<MCPhysReg> &Hints,
                             const MachineFunction &MF, const VirtRegMap *VRM,
                             const LiveRegMatrix *Matrix) const override;

  /// We require the register scavenger.
  bool requiresRegisterScavenging(const MachineFunction &MF) const override {
    return true;
  }

  bool requiresFrameIndexScavenging(const MachineFunction &MF) const override;

  bool requiresVirtualBaseRegisters(const MachineFunction &MF) const override;

  void lowerDynamicAlloc(MachineBasicBlock::iterator II) const;
  void lowerDynamicAreaOffset(MachineBasicBlock::iterator II) const;
  void prepareDynamicAlloca(MachineBasicBlock::iterator II,
                            Register &NegSizeReg, bool &KillNegSizeReg,
                            Register &FramePointer) const;
  void lowerPrepareProbedAlloca(MachineBasicBlock::iterator II) const;
  void lowerCRSpilling(MachineBasicBlock::iterator II,
                       unsigned FrameIndex) const;
```
- **EN**: Implements helper routine(s) `getRegAllocationHints`, `requiresRegisterScavenging`, `requiresFrameIndexScavenging` for this portion of the PowerPC backend register information and frame interaction logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `getRegAllocationHints`, `requiresRegisterScavenging`, `requiresFrameIndexScavenging`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 135-160

```cpp
  void lowerCRRestore(MachineBasicBlock::iterator II,
                      unsigned FrameIndex) const;
  void lowerCRBitSpilling(MachineBasicBlock::iterator II,
                          unsigned FrameIndex) const;
  void lowerCRBitRestore(MachineBasicBlock::iterator II,
                         unsigned FrameIndex) const;

  void lowerOctWordSpilling(MachineBasicBlock::iterator II,
                            unsigned FrameIndex) const;
  void lowerACCSpilling(MachineBasicBlock::iterator II,
                        unsigned FrameIndex) const;
  void lowerACCRestore(MachineBasicBlock::iterator II,
                       unsigned FrameIndex) const;

  void lowerWACCSpilling(MachineBasicBlock::iterator II,
                         unsigned FrameIndex) const;
  void lowerWACCRestore(MachineBasicBlock::iterator II,
                        unsigned FrameIndex) const;

  void lowerQuadwordSpilling(MachineBasicBlock::iterator II,
                             unsigned FrameIndex) const;
  void lowerQuadwordRestore(MachineBasicBlock::iterator II,
                            unsigned FrameIndex) const;

  void lowerDMRSpilling(MachineBasicBlock::iterator II,
                        unsigned FrameIndex) const;
```
- **EN**: Declares function entry points including `lowerCRRestore`, `lowerCRBitSpilling`, `lowerCRBitRestore` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `lowerCRRestore`, `lowerCRBitSpilling`, `lowerCRBitRestore`。

### Lines 161-186

```cpp
  void lowerDMRRestore(MachineBasicBlock::iterator II,
                       unsigned FrameIndex) const;

  static void emitAccCopyInfo(MachineBasicBlock &MBB, MCRegister DestReg,
                              MCRegister SrcReg);

  bool hasReservedSpillSlot(const MachineFunction &MF, Register Reg,
                            int &FrameIdx) const override;
  bool eliminateFrameIndex(MachineBasicBlock::iterator II, int SPAdj,
                           unsigned FIOperandNum,
                           RegScavenger *RS = nullptr) const override;

  // Support for virtual base registers.
  bool needsFrameBaseReg(MachineInstr *MI, int64_t Offset) const override;
  Register materializeFrameBaseRegister(MachineBasicBlock *MBB, int FrameIdx,
                                        int64_t Offset) const override;
  void resolveFrameIndex(MachineInstr &MI, Register BaseReg,
                         int64_t Offset) const override;
  bool isFrameOffsetLegal(const MachineInstr *MI, Register BaseReg,
                          int64_t Offset) const override;

  // Debug information queries.
  Register getFrameRegister(const MachineFunction &MF) const override;

  // Base pointer (stack realignment) support.
  Register getBaseRegister(const MachineFunction &MF) const;
```
- **EN**: Declares function entry points including `lowerDMRRestore`, `emitAccCopyInfo`, `hasReservedSpillSlot` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `lowerDMRRestore`, `emitAccCopyInfo`, `hasReservedSpillSlot`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 187-199

```cpp
  bool hasBasePointer(const MachineFunction &MF) const;

  bool isNonallocatableRegisterCalleeSave(MCRegister Reg) const override {
    return Reg == PPC::LR || Reg == PPC::LR8;
  }

  bool isVirtualFrameRegister(MCRegister Reg) const override {
    return Reg == PPC::FP || Reg == PPC::FP8;
  }
};

} // end namespace llvm
```
- **EN**: Implements helper routine(s) `hasBasePointer`, `isNonallocatableRegisterCalleeSave`, `isVirtualFrameRegister` for this portion of the PowerPC backend register information and frame interaction logic.
- **CN**: 这里实现了 PowerPC 后端该部分寄存器信息与栈帧交互逻辑所需的辅助例程 `hasBasePointer`, `isNonallocatableRegisterCalleeSave`, `isVirtualFrameRegister`。

### Lines 200-200

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Register information and frame interaction logic / 寄存器信息与栈帧交互逻辑
- Calling convention handling / 调用约定处理
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCMCTargetDesc.h`
- `llvm/ADT/DenseMap.h`
- `PPCGenRegisterInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
