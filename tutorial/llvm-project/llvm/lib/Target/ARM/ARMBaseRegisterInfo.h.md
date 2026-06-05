# ARMBaseRegisterInfo.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMBaseRegisterInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains the base ARM implementation of TargetRegisterInfo class.
- 用途 (CN): 声明 ARM 后端中的 `ARMBaseRegisterInfo`，并提供与共享寄存器定义与寄存器管理辅助逻辑相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMBaseRegisterInfo.h - ARM Register Information Impl ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the base ARM implementation of TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMBASEREGISTERINFO_H
#define LLVM_LIB_TARGET_ARM_ARMBASEREGISTERINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-22
```cpp
#include "MCTargetDesc/ARMBaseInfo.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/MC/MCRegisterInfo.h"
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-25
```cpp
#define GET_REGINFO_HEADER
#include "ARMGenRegisterInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 27-27
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 29-29
```cpp
class LiveIntervals;
```
- EN: Declares `LiveIntervals`, packaging target-specific state and APIs around `ARMBaseRegisterInfo`.
- CN: 这里声明 `LiveIntervals`，把与 `ARMBaseRegisterInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 31-32
```cpp
/// Register allocation hints.
namespace ARMRI {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 34-40
```cpp
  enum {
    // Used for LDRD register pairs
    RegPairOdd  = 1,
    RegPairEven = 2,
    // Used to hint for lr in t2DoLoopStart
    RegLR = 3
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 42-42
```cpp
} // end namespace ARMRI
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 44-50
```cpp
static inline bool isCalleeSavedRegister(MCRegister Reg,
                                         const MCPhysReg *CSRegs) {
  for (unsigned i = 0; CSRegs[i]; ++i)
    if (Reg == CSRegs[i])
      return true;
  return false;
}
```
- EN: Implements `isCalleeSavedRegister`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCalleeSavedRegister`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-57
```cpp
class ARMBaseRegisterInfo : public ARMGenRegisterInfo {
protected:
  /// BasePtr - ARM physical register used as a base ptr in complex stack
  /// frames. I.e., when we need a 3rd base, not just SP and FP, due to
  /// variable size stack objects.
  unsigned BasePtr = ARM::R6;
```
- EN: Declares `ARMBaseRegisterInfo`, packaging target-specific state and APIs around `ARMBaseRegisterInfo`.
- CN: 这里声明 `ARMBaseRegisterInfo`，把与 `ARMBaseRegisterInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 59-60
```cpp
  // Can be only subclassed.
  explicit ARMBaseRegisterInfo();
```
- EN: Declares `ARMBaseRegisterInfo`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMBaseRegisterInfo`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-71
```cpp
public:
  /// Code Generation virtual methods...
  const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const override;
  const MCPhysReg *
  getCalleeSavedRegsViaCopy(const MachineFunction *MF) const;
  const uint32_t *getCallPreservedMask(const MachineFunction &MF,
                                       CallingConv::ID) const override;
  const uint32_t *getNoPreservedMask() const override;
  const uint32_t *getTLSCallPreservedMask(const MachineFunction &MF) const;
  const uint32_t *getSjLjDispatchPreservedMask(const MachineFunction &MF) const;
```
- EN: Declares `getCalleeSavedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCalleeSavedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 73-82
```cpp
  /// getThisReturnPreservedMask - Returns a call preserved mask specific to the
  /// case that 'returned' is on an i32 first argument if the calling convention
  /// is one that can (partially) model this attribute with a preserved mask
  /// (i.e. it is a calling convention that uses the same register for the first
  /// i32 argument and an i32 return value)
  ///
  /// Should return NULL in the case that the calling convention does not have
  /// this property
  const uint32_t *getThisReturnPreservedMask(const MachineFunction &MF,
                                             CallingConv::ID) const;
```
- EN: Declares `getThisReturnPreservedMask`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThisReturnPreservedMask`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 84-85
```cpp
  ArrayRef<MCPhysReg>
  getIntraCallClobberedRegs(const MachineFunction *MF) const override;
```
- EN: Declares `getIntraCallClobberedRegs`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getIntraCallClobberedRegs`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-91
```cpp
  BitVector getReservedRegs(const MachineFunction &MF) const override;
  bool isAsmClobberable(const MachineFunction &MF,
                       MCRegister PhysReg) const override;
  bool isInlineAsmReadOnlyReg(const MachineFunction &MF,
                              MCRegister PhysReg) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 93-96
```cpp
  const TargetRegisterClass *
  getPointerRegClass(unsigned Kind = 0) const override;
  const TargetRegisterClass *
  getCrossCopyRegClass(const TargetRegisterClass *RC) const override;
```
- EN: Declares `getPointerRegClass`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getPointerRegClass`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-100
```cpp
  const TargetRegisterClass *
  getLargestLegalSuperClass(const TargetRegisterClass *RC,
                            const MachineFunction &MF) const override;
```
- EN: Declares `getLargestLegalSuperClass`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getLargestLegalSuperClass`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-103
```cpp
  unsigned getRegPressureLimit(const TargetRegisterClass *RC,
                               MachineFunction &MF) const override;
```
- EN: Declares `getRegPressureLimit`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegPressureLimit`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 105-108
```cpp
  bool getRegAllocationHints(Register VirtReg, ArrayRef<MCPhysReg> Order,
                             SmallVectorImpl<MCPhysReg> &Hints,
                             const MachineFunction &MF, const VirtRegMap *VRM,
                             const LiveRegMatrix *Matrix) const override;
```
- EN: Declares `getRegAllocationHints`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRegAllocationHints`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-111
```cpp
  void updateRegAllocHint(Register Reg, Register NewReg,
                          MachineFunction &MF) const override;
```
- EN: Declares `updateRegAllocHint`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `updateRegAllocHint`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 113-113
```cpp
  bool hasBasePointer(const MachineFunction &MF) const;
```
- EN: Declares `hasBasePointer`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `hasBasePointer`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 115-124
```cpp
  bool canRealignStack(const MachineFunction &MF) const override;
  int64_t getFrameIndexInstrOffset(const MachineInstr *MI,
                                   int Idx) const override;
  bool needsFrameBaseReg(MachineInstr *MI, int64_t Offset) const override;
  Register materializeFrameBaseRegister(MachineBasicBlock *MBB, int FrameIdx,
                                        int64_t Offset) const override;
  void resolveFrameIndex(MachineInstr &MI, Register BaseReg,
                         int64_t Offset) const override;
  bool isFrameOffsetLegal(const MachineInstr *MI, Register BaseReg,
                          int64_t Offset) const override;
```
- EN: Declares `canRealignStack`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `canRealignStack`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 126-126
```cpp
  bool cannotEliminateFrame(const MachineFunction &MF) const;
```
- EN: Declares `cannotEliminateFrame`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cannotEliminateFrame`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-130
```cpp
  // Debug information queries.
  Register getFrameRegister(const MachineFunction &MF) const override;
  Register getBaseRegister() const { return BasePtr; }
```
- EN: Implements `getFrameRegister`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFrameRegister`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 132-139
```cpp
  /// emitLoadConstPool - Emits a load from constpool to materialize the
  /// specified immediate.
  virtual void
  emitLoadConstPool(MachineBasicBlock &MBB, MachineBasicBlock::iterator &MBBI,
                    const DebugLoc &dl, Register DestReg, unsigned SubIdx,
                    int Val, ARMCC::CondCodes Pred = ARMCC::AL,
                    Register PredReg = Register(),
                    unsigned MIFlags = MachineInstr::NoFlags) const;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-142
```cpp
  /// Code Generation virtual methods...
  bool requiresRegisterScavenging(const MachineFunction &MF) const override;
```
- EN: Declares `requiresRegisterScavenging`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `requiresRegisterScavenging`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 144-144
```cpp
  bool requiresFrameIndexScavenging(const MachineFunction &MF) const override;
```
- EN: Declares `requiresFrameIndexScavenging`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `requiresFrameIndexScavenging`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-146
```cpp
  bool requiresVirtualBaseRegisters(const MachineFunction &MF) const override;
```
- EN: Declares `requiresVirtualBaseRegisters`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `requiresVirtualBaseRegisters`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 148-150
```cpp
  bool eliminateFrameIndex(MachineBasicBlock::iterator II,
                           int SPAdj, unsigned FIOperandNum,
                           RegScavenger *RS = nullptr) const override;
```
- EN: Declares `eliminateFrameIndex`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `eliminateFrameIndex`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-159
```cpp
  /// SrcRC and DstRC will be morphed into NewRC if this returns true
  bool shouldCoalesce(MachineInstr *MI,
                      const TargetRegisterClass *SrcRC,
                      unsigned SubReg,
                      const TargetRegisterClass *DstRC,
                      unsigned DstSubReg,
                      const TargetRegisterClass *NewRC,
                      LiveIntervals &LIS) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-162
```cpp
  int getSEHRegNum(unsigned i) const { return getEncodingValue(i); }
};
```
- EN: Implements `getSEHRegNum`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSEHRegNum`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 164-164
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 166-166
```cpp
#endif // LLVM_LIB_TARGET_ARM_ARMBASEREGISTERINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: shared register definitions and register-management helpers.
  - CN: 核心职责：共享寄存器定义与寄存器管理辅助逻辑。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMBaseInfo.h`, `ARMGenRegisterInfo.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMBaseInfo.h`, `ARMGenRegisterInfo.inc`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/IR/CallingConv.h`, `llvm/MC/MCRegisterInfo.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/IR/CallingConv.h`, `llvm/MC/MCRegisterInfo.h`。
- EN: Standard/system headers: `cstdint`.
  - CN: 标准库/系统头文件：`cstdint`。
