# SystemZMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZMachineFunctionInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file stores per-function target-specific machine state for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责存储每个函数的目标相关机器状态。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //=== SystemZMachineFunctionInfo.h - SystemZ machine function info -*- C++ -*-//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZMACHINEFUNCTIONINFO_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZMACHINEFUNCTIONINFO_H
  11: 
  12: #include "llvm/CodeGen/MachineFunction.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `MachineFunction.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `MachineFunction.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 13-24 / 第 13-24 行
```cpp
  13: 
  14: namespace llvm {
  15: 
  16: namespace SystemZ {
  17: // A struct to hold the low and high GPR registers to be saved/restored as
  18: // well as the offset into the register save area of the low register.
  19: struct GPRRegs {
  20:   unsigned LowGPR = 0;
  21:   unsigned HighGPR = 0;
  22:   unsigned GPROffset = 0;
  23:   GPRRegs() = default;
  24:   };
```
- **EN**: It introduces interface types such as `GPRRegs`, shaping how other backend components interact with this file.
- **CN**: 它引入了 `GPRRegs` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 25-36 / 第 25-36 行
```cpp
  25: }
  26: 
  27: class SystemZMachineFunctionInfo : public MachineFunctionInfo {
  28:   virtual void anchor();
  29: 
  30:   /// Size of expected parameter area for current function. (Fixed args only).
  31:   unsigned SizeOfFnParams;
  32: 
  33:   SystemZ::GPRRegs SpillGPRRegs;
  34:   SystemZ::GPRRegs RestoreGPRRegs;
  35:   Register VarArgsFirstGPR;
  36:   Register VarArgsFirstFPR;
```
- **EN**: This block declares or refines TableGen records such as `SystemZMachineFunctionInfo`.
- **CN**: 该代码块声明或细化了 `SystemZMachineFunctionInfo` 等 TableGen 记录。

### Lines 37-48 / 第 37-48 行
```cpp
  37:   unsigned VarArgsFrameIndex;
  38:   unsigned RegSaveFrameIndex;
  39:   int FramePointerSaveIndex;
  40:   unsigned NumLocalDynamics;
  41:   /// z/OS XPLINK ABI: incoming ADA virtual register.
  42:   Register VRegADA;
  43: 
  44: public:
  45:   SystemZMachineFunctionInfo(const Function &F, const TargetSubtargetInfo *STI)
  46:       : SizeOfFnParams(0), VarArgsFirstGPR(0), VarArgsFirstFPR(0),
  47:         VarArgsFrameIndex(0), RegSaveFrameIndex(0), FramePointerSaveIndex(0),
  48:         NumLocalDynamics(0) {}
```
- **EN**: The range implements or declares functions including `SystemZMachineFunctionInfo`.
- **CN**: 这一段实现或声明了 `SystemZMachineFunctionInfo` 等函数。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   MachineFunctionInfo *
  51:   clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
  52:         const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
  53:       const override;
  54: 
  55:   // z/OS: Get and set the size of the expected parameter area for the
  56:   // current function. (ie. Size of param area in caller).
  57:   unsigned getSizeOfFnParams() const { return SizeOfFnParams; }
  58:   void setSizeOfFnParams(unsigned Size) { SizeOfFnParams = Size; }
  59: 
  60:   // Get and set the first and last call-saved GPR that should be saved by
```
- **EN**: The range implements or declares functions including `clone`, `getSizeOfFnParams`, `setSizeOfFnParams`.
- **CN**: 这一段实现或声明了 `clone`, `getSizeOfFnParams`, `setSizeOfFnParams` 等函数。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   // this function and the SP offset for the STMG.  These are 0 if no GPRs
  62:   // need to be saved or restored.
  63:   SystemZ::GPRRegs getSpillGPRRegs() const { return SpillGPRRegs; }
  64:   void setSpillGPRRegs(Register Low, Register High, unsigned Offs) {
  65:     SpillGPRRegs.LowGPR = Low;
  66:     SpillGPRRegs.HighGPR = High;
  67:     SpillGPRRegs.GPROffset = Offs;
  68:   }
  69: 
  70:   // Get and set the first and last call-saved GPR that should be restored by
  71:   // this function and the SP offset for the LMG.  These are 0 if no GPRs
  72:   // need to be saved or restored.
```
- **EN**: The range implements or declares functions including `getSpillGPRRegs`, `setSpillGPRRegs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getSpillGPRRegs`, `setSpillGPRRegs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   SystemZ::GPRRegs getRestoreGPRRegs() const { return RestoreGPRRegs; }
  74:   void setRestoreGPRRegs(Register Low, Register High, unsigned Offs) {
  75:     RestoreGPRRegs.LowGPR = Low;
  76:     RestoreGPRRegs.HighGPR = High;
  77:     RestoreGPRRegs.GPROffset = Offs;
  78:   }
  79: 
  80:   // Get and set the number of fixed (as opposed to variable) arguments
  81:   // that are passed in GPRs to this function.
  82:   Register getVarArgsFirstGPR() const { return VarArgsFirstGPR; }
  83:   void setVarArgsFirstGPR(Register GPR) { VarArgsFirstGPR = GPR; }
  84: 
```
- **EN**: The range implements or declares functions including `getRestoreGPRRegs`, `setRestoreGPRRegs`, `getVarArgsFirstGPR`, `setVarArgsFirstGPR`.
- **CN**: 这一段实现或声明了 `getRestoreGPRRegs`, `setRestoreGPRRegs`, `getVarArgsFirstGPR`, `setVarArgsFirstGPR` 等函数。

### Lines 85-96 / 第 85-96 行
```cpp
  85:   // Likewise FPRs.
  86:   Register getVarArgsFirstFPR() const { return VarArgsFirstFPR; }
  87:   void setVarArgsFirstFPR(Register FPR) { VarArgsFirstFPR = FPR; }
  88: 
  89:   // Get and set the frame index of the first stack vararg.
  90:   unsigned getVarArgsFrameIndex() const { return VarArgsFrameIndex; }
  91:   void setVarArgsFrameIndex(unsigned FI) { VarArgsFrameIndex = FI; }
  92: 
  93:   // Get and set the frame index of the register save area
  94:   // (i.e. the incoming stack pointer).
  95:   unsigned getRegSaveFrameIndex() const { return RegSaveFrameIndex; }
  96:   void setRegSaveFrameIndex(unsigned FI) { RegSaveFrameIndex = FI; }
```
- **EN**: The range implements or declares functions including `getVarArgsFirstFPR`, `setVarArgsFirstFPR`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`, `getRegSaveFrameIndex`.
- **CN**: 这一段实现或声明了 `getVarArgsFirstFPR`, `setVarArgsFirstFPR`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`, `getRegSaveFrameIndex` 等函数。

### Lines 97-108 / 第 97-108 行
```cpp
  97: 
  98:   // Get and set the frame index of where the old frame pointer is stored.
  99:   int getFramePointerSaveIndex() const { return FramePointerSaveIndex; }
 100:   void setFramePointerSaveIndex(int Idx) { FramePointerSaveIndex = Idx; }
 101: 
 102:   // Count number of local-dynamic TLS symbols used.
 103:   unsigned getNumLocalDynamicTLSAccesses() const { return NumLocalDynamics; }
 104:   void incNumLocalDynamicTLSAccesses() { ++NumLocalDynamics; }
 105: 
 106:   // Get and set the function's incoming special XPLINK ABI defined ADA
 107:   // register.
 108:   Register getADAVirtualRegister() const { return VRegADA; }
```
- **EN**: The range implements or declares functions including `getFramePointerSaveIndex`, `setFramePointerSaveIndex`, `getNumLocalDynamicTLSAccesses`, `incNumLocalDynamicTLSAccesses`, `getADAVirtualRegister`.
- **CN**: 这一段实现或声明了 `getFramePointerSaveIndex`, `setFramePointerSaveIndex`, `getNumLocalDynamicTLSAccesses`, `incNumLocalDynamicTLSAccesses`, `getADAVirtualRegister` 等函数。

### Lines 109-114 / 第 109-114 行
```cpp
 109:   void setADAVirtualRegister(Register Reg) { VRegADA = Reg; }
 110: };
 111: 
 112: } // end namespace llvm
 113: 
 114: #endif
```
- **EN**: The range implements or declares functions including `setADAVirtualRegister`.
- **CN**: 这一段实现或声明了 `setADAVirtualRegister` 等函数。

## Key Concepts / 关键概念
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `llvm/CodeGen/MachineFunction.h`
