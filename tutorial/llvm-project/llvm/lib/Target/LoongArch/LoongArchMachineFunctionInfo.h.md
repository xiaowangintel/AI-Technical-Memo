# LoongArchMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchMachineFunctionInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file stores per-function target-specific machine state for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责存储每个函数的目标相关机器状态。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //=- LoongArchMachineFunctionInfo.h - LoongArch machine function info -----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares LoongArch-specific per-machine-function information.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHMACHINEFUNCTIONINFO_H
  14: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHMACHINEFUNCTIONINFO_H
  15: 
  16: #include "LoongArchSubtarget.h"
  17: #include "llvm/CodeGen/MachineFrameInfo.h"
  18: #include "llvm/CodeGen/MachineFunction.h"
  19: 
  20: namespace llvm {
  21: 
  22: /// LoongArchMachineFunctionInfo - This class is derived from
  23: /// MachineFunctionInfo and contains private LoongArch-specific information for
  24: /// each MachineFunction.
```
- **EN**: It imports dependencies such as `LoongArchSubtarget.h`, `MachineFrameInfo.h`, `MachineFunction.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchSubtarget.h`, `MachineFrameInfo.h`, `MachineFunction.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25: class LoongArchMachineFunctionInfo : public MachineFunctionInfo {
  26: private:
  27:   /// FrameIndex for start of varargs area
  28:   int VarArgsFrameIndex = 0;
  29:   /// Size of the save area used for varargs
  30:   int VarArgsSaveSize = 0;
  31: 
  32:   /// Size of stack frame to save callee saved registers
  33:   unsigned CalleeSavedStackSize = 0;
  34: 
  35:   /// FrameIndex of the spill slot when there is no scavenged register in
  36:   /// insertIndirectBranch.
```
- **EN**: This block declares or refines TableGen records such as `LoongArchMachineFunctionInfo`.
- **CN**: 该代码块声明或细化了 `LoongArchMachineFunctionInfo` 等 TableGen 记录。

### Lines 37-48 / 第 37-48 行
```cpp
  37:   int BranchRelaxationSpillFrameIndex = -1;
  38: 
  39:   /// Registers that have been sign extended from i32.
  40:   SmallVector<Register, 8> SExt32Registers;
  41: 
  42:   /// Pairs of `jr` instructions and corresponding JTI operands, used for the
  43:   /// `annotate-tablejump` option.
  44:   SmallVector<std::pair<MachineInstr *, int>, 4> JumpInfos;
  45: 
  46: public:
  47:   LoongArchMachineFunctionInfo(const Function &F,
  48:                                const TargetSubtargetInfo *STI) {}
```
- **EN**: The range implements or declares functions including `LoongArchMachineFunctionInfo`.
- **CN**: 这一段实现或声明了 `LoongArchMachineFunctionInfo` 等函数。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   MachineFunctionInfo *
  51:   clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
  52:         const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
  53:       const override {
  54:     return DestMF.cloneInfo<LoongArchMachineFunctionInfo>(*this);
  55:   }
  56: 
  57:   int getVarArgsFrameIndex() const { return VarArgsFrameIndex; }
  58:   void setVarArgsFrameIndex(int Index) { VarArgsFrameIndex = Index; }
  59: 
  60:   unsigned getVarArgsSaveSize() const { return VarArgsSaveSize; }
```
- **EN**: The range implements or declares functions including `clone`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`, `getVarArgsSaveSize`.
- **CN**: 这一段实现或声明了 `clone`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`, `getVarArgsSaveSize` 等函数。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   void setVarArgsSaveSize(int Size) { VarArgsSaveSize = Size; }
  62: 
  63:   unsigned getCalleeSavedStackSize() const { return CalleeSavedStackSize; }
  64:   void setCalleeSavedStackSize(unsigned Size) { CalleeSavedStackSize = Size; }
  65: 
  66:   int getBranchRelaxationSpillFrameIndex() {
  67:     return BranchRelaxationSpillFrameIndex;
  68:   }
  69:   void setBranchRelaxationSpillFrameIndex(int Index) {
  70:     BranchRelaxationSpillFrameIndex = Index;
  71:   }
  72: 
```
- **EN**: The range implements or declares functions including `setVarArgsSaveSize`, `getCalleeSavedStackSize`, `setCalleeSavedStackSize`, `getBranchRelaxationSpillFrameIndex`, `setBranchRelaxationSpillFrameIndex`.
- **CN**: 这一段实现或声明了 `setVarArgsSaveSize`, `getCalleeSavedStackSize`, `setCalleeSavedStackSize`, `getBranchRelaxationSpillFrameIndex`, `setBranchRelaxationSpillFrameIndex` 等函数。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   void addSExt32Register(Register Reg) { SExt32Registers.push_back(Reg); }
  74: 
  75:   bool isSExt32Register(Register Reg) const {
  76:     return is_contained(SExt32Registers, Reg);
  77:   }
  78: 
  79:   void setJumpInfo(MachineInstr *JrMI, int JTIIdx) {
  80:     JumpInfos.push_back(std::make_pair(JrMI, JTIIdx));
  81:   }
  82:   unsigned getJumpInfoSize() { return JumpInfos.size(); }
  83:   MachineInstr *getJumpInfoJrMI(unsigned Idx) { return JumpInfos[Idx].first; }
  84:   int getJumpInfoJTIIndex(unsigned Idx) { return JumpInfos[Idx].second; }
```
- **EN**: The range implements or declares functions including `addSExt32Register`, `isSExt32Register`, `setJumpInfo`, `getJumpInfoSize`, `getJumpInfoJTIIndex`.
- **CN**: 这一段实现或声明了 `addSExt32Register`, `isSExt32Register`, `setJumpInfo`, `getJumpInfoSize`, `getJumpInfoJTIIndex` 等函数。

### Lines 85-89 / 第 85-89 行
```cpp
  85: };
  86: 
  87: } // end namespace llvm
  88: 
  89: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHMACHINEFUNCTIONINFO_H
```
- **EN**: This span continues the file's main responsibility: this file stores per-function target-specific machine state for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchSubtarget.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
