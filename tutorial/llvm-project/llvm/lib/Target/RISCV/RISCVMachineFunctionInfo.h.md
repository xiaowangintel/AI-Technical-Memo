# RISCVMachineFunctionInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVMachineFunctionInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for per-function machine-level state tracked by the RISC-V backend. / 声明RISC-V 后端跟踪的逐函数机器级状态所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//=- RISCVMachineFunctionInfo.h - RISC-V machine function info ----*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares RISCV-specific per-machine-function information.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-22: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#ifndef LLVM_LIB_TARGET_RISCV_RISCVMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_RISCV_RISCVMACHINEFUNCTIONINFO_H

#include "RISCVSubtarget.h"
#include "llvm/CodeGen/MIRYamlMapping.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"

namespace llvm {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 23-32: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
class RISCVMachineFunctionInfo;

namespace yaml {
struct RISCVMachineFunctionInfo final : public yaml::MachineFunctionInfo {
  int VarArgsFrameIndex;
  int VarArgsSaveSize;

  RISCVMachineFunctionInfo() = default;
  RISCVMachineFunctionInfo(const llvm::RISCVMachineFunctionInfo &MFI);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 33-42: Type declaration for MappingTraits<RISCVMachineFunctionInfo> / MappingTraits<RISCVMachineFunctionInfo> 的类型声明
```cpp
  void mappingImpl(yaml::IO &YamlIO) override;
  ~RISCVMachineFunctionInfo() override = default;
};

template <> struct MappingTraits<RISCVMachineFunctionInfo> {
  static void mapping(IO &YamlIO, RISCVMachineFunctionInfo &MFI) {
    YamlIO.mapOptional("varArgsFrameIndex", MFI.VarArgsFrameIndex);
    YamlIO.mapOptional("varArgsSaveSize", MFI.VarArgsSaveSize);
  }
};
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 43-62: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // end namespace yaml

/// RISCVMachineFunctionInfo - This class is derived from MachineFunctionInfo
/// and contains private RISCV-specific information for each MachineFunction.
class RISCVMachineFunctionInfo : public MachineFunctionInfo {
private:
  /// FrameIndex for start of varargs area
  int VarArgsFrameIndex = 0;
  /// Size of the save area used for varargs
  int VarArgsSaveSize = 0;
  /// FrameIndex used for transferring values between 64-bit FPRs and a pair
  /// of 32-bit GPRs via the stack.
  int MoveF64FrameIndex = -1;
  /// FrameIndex of the spill slot for the scratch register in BranchRelaxation.
  int BranchRelaxationScratchFrameIndex = -1;
  /// Size of any opaque stack adjustment due to save/restore libcalls.
  unsigned LibCallStackSize = 0;
  /// Size of RVV stack.
  uint64_t RVVStackSize = 0;
  /// Alignment of RVV stack.
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 63-73: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  Align RVVStackAlign;
  /// Padding required to keep RVV stack aligned within the main stack.
  uint64_t RVVPadding = 0;
  /// Size of stack frame to save callee saved registers
  unsigned CalleeSavedStackSize = 0;
  /// Is there any vector argument or return?
  bool IsVectorCall = false;

  /// Registers that have been sign extended from i32.
  SmallVector<Register, 8> SExt32Registers;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 74-83: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  /// Size of stack frame for Zcmp PUSH/POP
  unsigned RVPushStackSize = 0;
  unsigned RVPushRegs = 0;

  /// Size of any opaque stack adjustment due to QCI Interrupt instructions.
  unsigned QCIInterruptStackSize = 0;

  /// Store Frame Indexes for Interrupt-Related CSR Spills.
  SmallVector<int, 2> InterruptCSRFrameIndexes;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 84-94: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  int64_t StackProbeSize = 0;

  /// Does it probe the stack for a dynamic allocation?
  bool HasDynamicAllocation = false;

  /// Whether the function has cf-protection-branch module flag set.
  bool CFProtectionBranch = false;

public:
  RISCVMachineFunctionInfo(const Function &F, const RISCVSubtarget *STI);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 95-105: Function implementation: getVarArgsFrameIndex / 函数实现：getVarArgsFrameIndex
```cpp
  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;

  int getVarArgsFrameIndex() const { return VarArgsFrameIndex; }
  void setVarArgsFrameIndex(int Index) { VarArgsFrameIndex = Index; }

  unsigned getVarArgsSaveSize() const { return VarArgsSaveSize; }
  void setVarArgsSaveSize(int Size) { VarArgsSaveSize = Size; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 106-115: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  int getMoveF64FrameIndex(MachineFunction &MF) {
    if (MoveF64FrameIndex == -1)
      MoveF64FrameIndex =
          MF.getFrameInfo().CreateStackObject(8, Align(8), false);
    return MoveF64FrameIndex;
  }

  int getBranchRelaxationScratchFrameIndex() const {
    return BranchRelaxationScratchFrameIndex;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 116-126: Function implementation: setBranchRelaxationScratchFrameIndex / 函数实现：setBranchRelaxationScratchFrameIndex
```cpp
  void setBranchRelaxationScratchFrameIndex(int Index) {
    BranchRelaxationScratchFrameIndex = Index;
  }

  unsigned getReservedSpillsSize() const {
    return LibCallStackSize + RVPushStackSize + QCIInterruptStackSize;
  }

  unsigned getLibCallStackSize() const { return LibCallStackSize; }
  void setLibCallStackSize(unsigned Size) { LibCallStackSize = Size; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 127-138: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  bool useSaveRestoreLibCalls(const MachineFunction &MF) const {
    // We cannot use fixed locations for the callee saved spill slots if the
    // function uses a varargs save area, or is an interrupt handler.
    return !isPushable(MF) &&
           MF.getSubtarget<RISCVSubtarget>().enableSaveRestore() &&
           VarArgsSaveSize == 0 && !MF.getFrameInfo().hasTailCall() &&
           !MF.getFunction().hasFnAttribute("interrupt");
  }

  uint64_t getRVVStackSize() const { return RVVStackSize; }
  void setRVVStackSize(uint64_t Size) { RVVStackSize = Size; }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 139-149: Type declaration for PushPopKind / PushPopKind 的类型声明
```cpp
  Align getRVVStackAlign() const { return RVVStackAlign; }
  void setRVVStackAlign(Align StackAlign) { RVVStackAlign = StackAlign; }

  uint64_t getRVVPadding() const { return RVVPadding; }
  void setRVVPadding(uint64_t Padding) { RVVPadding = Padding; }

  unsigned getCalleeSavedStackSize() const { return CalleeSavedStackSize; }
  void setCalleeSavedStackSize(unsigned Size) { CalleeSavedStackSize = Size; }

  enum class PushPopKind { None = 0, StdExtZcmp, VendorXqccmp };
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 150-161: Function implementation: isPushable / 函数实现：isPushable
```cpp
  PushPopKind getPushPopKind(const MachineFunction &MF) const;

  bool isPushable(const MachineFunction &MF) const {
    return getPushPopKind(MF) != PushPopKind::None;
  }

  unsigned getRVPushRegs() const { return RVPushRegs; }
  void setRVPushRegs(unsigned Regs) { RVPushRegs = Regs; }

  unsigned getRVPushStackSize() const { return RVPushStackSize; }
  void setRVPushStackSize(unsigned Size) { RVPushStackSize = Size; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 162-172: Type declaration for InterruptStackKind / InterruptStackKind 的类型声明
```cpp
  enum class InterruptStackKind {
    None = 0,
    QCINest,
    QCINoNest,
    SiFiveCLICPreemptible,
    SiFiveCLICStackSwap,
    SiFiveCLICPreemptibleStackSwap
  };

  InterruptStackKind getInterruptStackKind(const MachineFunction &MF) const;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 173-187: Function implementation: useQCIInterrupt / 函数实现：useQCIInterrupt
```cpp
  bool useQCIInterrupt(const MachineFunction &MF) const {
    InterruptStackKind Kind = getInterruptStackKind(MF);
    return Kind == InterruptStackKind::QCINest ||
           Kind == InterruptStackKind::QCINoNest;
  }

  unsigned getQCIInterruptStackSize() const { return QCIInterruptStackSize; }
  void setQCIInterruptStackSize(unsigned Size) { QCIInterruptStackSize = Size; }

  bool useSiFiveInterrupt(const MachineFunction &MF) const {
    InterruptStackKind Kind = getInterruptStackKind(MF);
    return Kind == InterruptStackKind::SiFiveCLICPreemptible ||
           Kind == InterruptStackKind::SiFiveCLICStackSwap ||
           Kind == InterruptStackKind::SiFiveCLICPreemptibleStackSwap;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 188-199: Function implementation: isSiFivePreemptibleInterrupt / 函数实现：isSiFivePreemptibleInterrupt
```cpp

  bool isSiFivePreemptibleInterrupt(const MachineFunction &MF) const {
    InterruptStackKind Kind = getInterruptStackKind(MF);
    return Kind == InterruptStackKind::SiFiveCLICPreemptible ||
           Kind == InterruptStackKind::SiFiveCLICPreemptibleStackSwap;
  }

  bool isSiFiveStackSwapInterrupt(const MachineFunction &MF) const {
    InterruptStackKind Kind = getInterruptStackKind(MF);
    return Kind == InterruptStackKind::SiFiveCLICStackSwap ||
           Kind == InterruptStackKind::SiFiveCLICPreemptibleStackSwap;
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 200-213: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  void pushInterruptCSRFrameIndex(int FI) {
    InterruptCSRFrameIndexes.push_back(FI);
  }
  int getInterruptCSRFrameIndex(size_t Idx) const {
    return InterruptCSRFrameIndexes[Idx];
  }

  // Some Stack Management Variants automatically update FP in a frame-pointer
  // convention compatible way - which means we don't need to manually update
  // the FP, but we still need to emit the correct CFI information for
  // calculating the CFA based on FP.
  bool hasImplicitFPUpdates(const MachineFunction &MF) const;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 214-224: Function implementation: isVectorCall / 函数实现：isVectorCall
```cpp
  void initializeBaseYamlFields(const yaml::RISCVMachineFunctionInfo &YamlMFI);

  void addSExt32Register(Register Reg);
  bool isSExt32Register(Register Reg) const;

  bool isVectorCall() const { return IsVectorCall; }
  void setIsVectorCall() { IsVectorCall = true; }

  bool hasDynamicAllocation() const { return HasDynamicAllocation; }
  void setDynamicAllocation() { HasDynamicAllocation = true; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 225-230: Header guard and interface framing / 头文件保护与接口框架
```cpp
  bool hasCFProtectionBranch() const { return CFProtectionBranch; }
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_RISCV_RISCVMACHINEFUNCTIONINFO_H
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MIRYamlMapping.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFrameInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunction.h` — Directly referenced by this file. / 该文件直接引用的依赖。
