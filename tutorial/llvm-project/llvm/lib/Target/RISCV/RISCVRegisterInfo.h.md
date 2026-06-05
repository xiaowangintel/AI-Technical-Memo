# RISCVRegisterInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVRegisterInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for physical register metadata, frame-index elimination, and register constraints for RISC-V. / 声明RISC-V 的物理寄存器元数据、frame index 消除与寄存器约束所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVRegisterInfo.h - RISC-V Register Information Impl --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the RISC-V implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-21: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#ifndef LLVM_LIB_TARGET_RISCV_RISCVREGISTERINFO_H
#define LLVM_LIB_TARGET_RISCV_RISCVREGISTERINFO_H

#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/TargetParser/RISCVTargetParser.h"

#define GET_REGINFO_HEADER
#include "RISCVGenRegisterInfo.inc"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 22-33: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace llvm {

namespace RISCVRI {
enum : uint8_t {
  // The IsVRegClass value of this RegisterClass.
  IsVRegClassShift = 0,
  IsVRegClassShiftMask = 0b1 << IsVRegClassShift,
  // The VLMul value of this RegisterClass. This value is valid iff IsVRegClass
  // is true.
  VLMulShift = IsVRegClassShift + 1,
  VLMulShiftMask = 0b11 << VLMulShift,
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 34-45: Enumeration and symbolic state / 枚举与符号状态
```cpp
  // The NF value of this RegisterClass. This value is valid iff IsVRegClass is
  // true.
  NFShift = VLMulShift + 2,
  NFShiftMask = 0b111 << NFShift,
};

/// Register allocation hints for Zilsd register pairs
enum {
  // Used for Zilsd LD/SD register pairs
  RegPairOdd = 1,
  RegPairEven = 2,
};
```
**EN:** This block defines named constants or state encodings that make later target logic more explicit and less error-prone.

**CN:** 该区段定义具名常量或状态编码，使后续目标相关逻辑更清晰且更不易出错。

### Lines 46-56: Type declaration for isVRegClass / isVRegClass 的类型声明
```cpp

/// \returns the IsVRegClass for the register class.
static inline bool isVRegClass(uint8_t TSFlags) {
  return (TSFlags & IsVRegClassShiftMask) >> IsVRegClassShift;
}

/// \returns the LMUL for the register class.
static inline RISCVVType::VLMUL getLMul(uint8_t TSFlags) {
  return static_cast<RISCVVType::VLMUL>((TSFlags & VLMulShiftMask) >>
                                        VLMulShift);
}
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 57-67: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

/// \returns the NF for the register class.
static inline unsigned getNF(uint8_t TSFlags) {
  return static_cast<unsigned>((TSFlags & NFShiftMask) >> NFShift) + 1;
}
} // namespace RISCVRI

struct RISCVRegisterInfo : public RISCVGenRegisterInfo {

  RISCVRegisterInfo(unsigned HwMode);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 68-77: Function implementation: getCSRCost / 函数实现：getCSRCost
```cpp
  const uint32_t *getCallPreservedMask(const MachineFunction &MF,
                                       CallingConv::ID) const override;

  unsigned getCSRCost() const override {
    // The cost will be compared against BlockFrequency where entry has the
    // value of 1 << 14. A value of 5 will choose to spill or split cold
    // path instead of using a callee-saved register.
    return 5;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 78-87: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  const MCPhysReg *getCalleeSavedRegs(const MachineFunction *MF) const override;

  const TargetRegisterClass *getConstrainedRegClassForOperand(
      const MachineOperand &MO, const MachineRegisterInfo &MRI) const override;

  const TargetRegisterClass *
  getRegClassForTypeOnBank(LLT Ty, const RegisterBank &RB, bool Is64Bit) const;

  const MCPhysReg *getIPRACSRegs(const MachineFunction *MF) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 88-102: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  BitVector getReservedRegs(const MachineFunction &MF) const override;
  bool isAsmClobberable(const MachineFunction &MF,
                        MCRegister PhysReg) const override;

  const uint32_t *getNoPreservedMask() const override;

  // Update DestReg to have the value SrcReg plus an offset.  This is
  // used during frame layout, and we may need to ensure that if we
  // split the offset internally that the DestReg is always aligned,
  // assuming that source reg was.
  void adjustReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator II,
                 const DebugLoc &DL, Register DestReg, Register SrcReg,
                 StackOffset Offset, MachineInstr::MIFlag Flag,
                 MaybeAlign RequiredAlign) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 103-113: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool eliminateFrameIndex(MachineBasicBlock::iterator MI, int SPAdj,
                           unsigned FIOperandNum,
                           RegScavenger *RS = nullptr) const override;

  bool requiresVirtualBaseRegisters(const MachineFunction &MF) const override;

  bool needsFrameBaseReg(MachineInstr *MI, int64_t Offset) const override;

  bool isFrameOffsetLegal(const MachineInstr *MI, Register BaseReg,
                          int64_t Offset) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 114-125: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  Register materializeFrameBaseRegister(MachineBasicBlock *MBB, int FrameIdx,
                                        int64_t Offset) const override;

  void resolveFrameIndex(MachineInstr &MI, Register BaseReg,
                         int64_t Offset) const override;

  int64_t getFrameIndexInstrOffset(const MachineInstr *MI,
                                   int Idx) const override;

  void lowerSegmentSpillReload(MachineBasicBlock::iterator II,
                               bool IsSpill) const;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 126-136: Function implementation: requiresRegisterScavenging / 函数实现：requiresRegisterScavenging
```cpp
  Register getFrameRegister(const MachineFunction &MF) const override;

  StringRef getRegAsmName(MCRegister Reg) const override;

  bool requiresRegisterScavenging(const MachineFunction &MF) const override {
    return true;
  }

  bool requiresFrameIndexScavenging(const MachineFunction &MF) const override {
    return true;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 137-146: Function implementation: getPointerRegClass / 函数实现：getPointerRegClass
```cpp

  const TargetRegisterClass *
  getPointerRegClass(unsigned Kind = 0) const override {
    return &RISCV::GPRRegClass;
  }

  const TargetRegisterClass *
  getLargestLegalSuperClass(const TargetRegisterClass *RC,
                            const MachineFunction &) const override;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 147-158: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void getOffsetOpcodes(const StackOffset &Offset,
                        SmallVectorImpl<uint64_t> &Ops) const override;

  unsigned getRegisterCostTableIndex(const MachineFunction &MF) const override;

  float getSpillWeightScaleFactor(const TargetRegisterClass *RC) const override;

  bool getRegAllocationHints(Register VirtReg, ArrayRef<MCPhysReg> Order,
                             SmallVectorImpl<MCPhysReg> &Hints,
                             const MachineFunction &MF, const VirtRegMap *VRM,
                             const LiveRegMatrix *Matrix) const override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 159-168: Function implementation: RISCVRI::isVRegClass / 函数实现：RISCVRI::isVRegClass
```cpp
  void updateRegAllocHint(Register Reg, Register NewReg,
                          MachineFunction &MF) const override;

  Register findVRegWithEncoding(const TargetRegisterClass &RegClass,
                                uint16_t Encoding) const;

  static bool isVRRegClass(const TargetRegisterClass *RC) {
    return RISCVRI::isVRegClass(RC->TSFlags) &&
           RISCVRI::getNF(RC->TSFlags) == 1;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 169-179: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

  static bool isVRNRegClass(const TargetRegisterClass *RC) {
    return RISCVRI::isVRegClass(RC->TSFlags) && RISCVRI::getNF(RC->TSFlags) > 1;
  }

  static bool isRVVRegClass(const TargetRegisterClass *RC) {
    return RISCVRI::isVRegClass(RC->TSFlags);
  }
};
} // namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 180-180: Header guard and interface framing / 头文件保护与接口框架
```cpp
#endif
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **Register metadata** / **寄存器元数据**

## Dependencies / 依赖关系
- `llvm/CodeGen/TargetRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/TargetParser/RISCVTargetParser.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenRegisterInfo.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
