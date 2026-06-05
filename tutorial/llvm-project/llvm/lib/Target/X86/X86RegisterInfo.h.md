# X86RegisterInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86RegisterInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces and shared data structures for register information in the core X86 backend. / 为X86 后端核心中的寄存器信息声明接口与共享数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86RegisterInfo.h - X86 Register Information Impl -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the X86 implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_X86_X86REGISTERINFO_H
#define LLVM_LIB_TARGET_X86_X86REGISTERINFO_H

#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"

#define GET_REGINFO_HEADER
#include "X86GenRegisterInfo.inc"

namespace llvm {
  class Triple;

class X86RegisterInfo final : public X86GenRegisterInfo {
private:
  /// Is64Bit - Is the target 64-bits.
  ///
  bool Is64Bit;

```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include Triple, X86RegisterInfo. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 Triple, X86RegisterInfo。这些内容定义了实现文件所依赖的契约。

### Lines 31-60: Comments and explanatory notes / 注释与说明性文字
```cpp
  /// Is this x86_64 with the LP64 programming model (standard AMD64, no x32)?
  bool IsTarget64BitLP64;

  /// IsWin64 - Is the target on of win64 flavours
  ///
  bool IsWin64;

  /// IsUEFI64 - Is UEFI 64 bit target.
  ///
  bool IsUEFI64;

  /// SlotSize - Stack slot size in bytes.
  ///
  unsigned SlotSize;

  /// StackPtr - X86 physical register used as stack ptr.
  ///
  unsigned StackPtr;

  /// FramePtr - X86 physical register used as frame ptr.
  ///
  unsigned FramePtr;

  /// BasePtr - X86 physical register used as a base ptr in complex stack
  /// frames. I.e., when we need a 3rd base, not just SP and FP, due to
  /// variable size stack objects.
  unsigned BasePtr;

public:
  explicit X86RegisterInfo(const Triple &TT);
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 61-90: Comments and explanatory notes / 注释与说明性文字
```cpp

  /// Return the number of registers for the function.
  unsigned getNumSupportedRegs(const MachineFunction &MF) const override;

  /// getMatchingSuperRegClass - Return a subclass of the specified register
  /// class A so that each register in it has a sub-register of the
  /// specified sub-register index which is in the specified register class B.
  const TargetRegisterClass *
  getMatchingSuperRegClass(const TargetRegisterClass *A,
                           const TargetRegisterClass *B,
                           unsigned Idx) const override;

  const TargetRegisterClass *
  getSubClassWithSubReg(const TargetRegisterClass *RC,
                        unsigned Idx) const override;

  const TargetRegisterClass *
  getLargestLegalSuperClass(const TargetRegisterClass *RC,
                            const MachineFunction &MF) const override;

  /// getPointerRegClass - Returns a TargetRegisterClass used for pointer
  /// values.
  const TargetRegisterClass *
  getPointerRegClass(unsigned Kind = 0) const override;

  /// getCrossCopyRegClass - Returns a legal register class to copy a register
  /// in the specified class to or from. Returns NULL if it is possible to copy
  /// between a two registers of the specified class.
  const TargetRegisterClass *
  getCrossCopyRegClass(const TargetRegisterClass *RC) const override;
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 91-120: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp

  unsigned getRegPressureLimit(const TargetRegisterClass *RC,
                               MachineFunction &MF) const override;

  /// getCalleeSavedRegs - Return a null-terminated list of all of the
  /// callee-save registers on this target.
  const MCPhysReg *
  getCalleeSavedRegs(const MachineFunction* MF) const override;
  /// getIPRACSRegs - This API can be removed when rbp is safe to optimized out
  /// when IPRA is on.
  const MCPhysReg *getIPRACSRegs(const MachineFunction *MF) const override;
  const MCPhysReg *
  getCalleeSavedRegsViaCopy(const MachineFunction *MF) const;
  const uint32_t *getCallPreservedMask(const MachineFunction &MF,
                                       CallingConv::ID) const override;
  const uint32_t *getNoPreservedMask() const override;

  // Calls involved in thread-local variable lookup save more registers than
  // normal calls, so they need a different mask to represent this.
  const uint32_t *getDarwinTLSCallPreservedMask() const;

  /// getReservedRegs - Returns a bitset indexed by physical register number
  /// indicating if a register is a special register that has particular uses and
  /// should be considered unavailable at all times, e.g. SP, RA. This is used by
  /// register scavenger to determine what registers are free.
  BitVector getReservedRegs(const MachineFunction &MF) const override;

  /// isArgumentReg - Returns true if Reg can be used as an argument to a
  /// function.
  bool isArgumentRegister(const MachineFunction &MF,
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 121-150: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
                          MCRegister Reg) const override;

  /// Return true if it is tile register class.
  bool isTileRegisterClass(const TargetRegisterClass *RC) const;

  /// Returns true if PhysReg is a fixed register.
  bool isFixedRegister(const MachineFunction &MF,
                       MCRegister PhysReg) const override;

  void adjustStackMapLiveOutMask(uint32_t *Mask) const override;

  bool hasBasePointer(const MachineFunction &MF) const;

  bool canRealignStack(const MachineFunction &MF) const override;

  bool shouldRealignStack(const MachineFunction &MF) const override;

  void eliminateFrameIndex(MachineBasicBlock::iterator II,
                           unsigned FIOperandNum, Register BaseReg,
                           int FIOffset) const;

  bool eliminateFrameIndex(MachineBasicBlock::iterator MI,
                           int SPAdj, unsigned FIOperandNum,
                           RegScavenger *RS = nullptr) const override;

  /// Process frame indices in forwards block order because
  /// X86InstrInfo::getSPAdjust relies on it when searching for the
  /// ADJCALLSTACKUP pseudo following a call.
  /// TODO: Fix this and return true like all other targets.
  bool eliminateFrameIndicesBackwards() const override { return false; }
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  /// findDeadCallerSavedReg - Return a caller-saved register that isn't live
  /// when it reaches the "return" instruction. We can then pop a stack object
  /// to this register without worry about clobbering it.
  unsigned findDeadCallerSavedReg(MachineBasicBlock &MBB,
                                  MachineBasicBlock::iterator &MBBI) const;

  // Debug information queries.
  Register getFrameRegister(const MachineFunction &MF) const override;
  Register getPtrSizedFrameRegister(const MachineFunction &MF) const;
  Register getPtrSizedStackRegister(const MachineFunction &MF) const;
  Register getStackRegister() const { return StackPtr; }
  Register getBaseRegister() const { return BasePtr; }
  /// Returns physical register used as frame pointer.
  /// This will always returns the frame pointer register, contrary to
  /// getFrameRegister() which returns the "base pointer" in situations
  /// involving a stack, frame and base pointer.
  Register getFramePtr() const { return FramePtr; }
  // FIXME: Move to FrameInfok
  unsigned getSlotSize() const { return SlotSize; }

  bool getRegAllocationHints(Register VirtReg, ArrayRef<MCPhysReg> Order,
                             SmallVectorImpl<MCPhysReg> &Hints,
                             const MachineFunction &MF, const VirtRegMap *VRM,
                             const LiveRegMatrix *Matrix) const override;

  const TargetRegisterClass *
  constrainRegClassToNonRex2(const TargetRegisterClass *RC) const;

  bool isNonRex2RegClass(const TargetRegisterClass *RC) const;
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 181-189: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

  bool requiresRegisterScavenging(const MachineFunction &MF) const override {
    return true;
  }
};

} // End llvm namespace

#endif
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

## Key Concepts / 关键概念
- Primary topic: register information. / 核心主题：寄存器信息。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: Triple, X86RegisterInfo. / 重要符号：Triple, X86RegisterInfo。
- The file emphasizes declarations, interfaces, and reusable helpers. / 该文件以声明、接口和可复用辅助逻辑为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: llvm/CodeGen/MachineFunction.h, llvm/CodeGen/TargetRegisterInfo.h, X86GenRegisterInfo.inc. / 直接包含：llvm/CodeGen/MachineFunction.h, llvm/CodeGen/TargetRegisterInfo.h, X86GenRegisterInfo.inc。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
- Implementation files in the same subsystem rely on these declarations. / 同一子系统中的实现文件依赖这些声明。
