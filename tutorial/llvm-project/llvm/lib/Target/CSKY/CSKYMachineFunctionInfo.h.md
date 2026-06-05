# CSKYMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYMachineFunctionInfo.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file declares CSKY-specific per-machine-function information.
- 目的（中文）: 定义目标后端在每个 MachineFunction 上跟踪的附加状态。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //=- CSKYMachineFunctionInfo.h - CSKY machine function info -------*- C++ -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares CSKY-specific per-machine-function information.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_CSKY_CSKYMACHINEFUNCTIONINFO_H
  14: #define LLVM_LIB_TARGET_CSKY_CSKYMACHINEFUNCTIONINFO_H
  15: 
  16: #include "llvm/CodeGen/MachineFunction.h"
  17: 
  18: namespace llvm {
  19: 
  20: class CSKYMachineFunctionInfo : public MachineFunctionInfo {
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYMachineFunctionInfo, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYMachineFunctionInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21:   Register GlobalBaseReg = 0;
  22:   bool SpillsCR = false;
  23: 
  24:   int VarArgsFrameIndex = 0;
  25:   unsigned VarArgsSaveSize = 0;
  26: 
  27:   int spillAreaSize = 0;
  28: 
  29:   bool LRSpilled = false;
  30: 
  31:   unsigned PICLabelUId = 0;
  32: 
  33: public:
  34:   CSKYMachineFunctionInfo(const Function &F, const TargetSubtargetInfo *STI) {}
  35: 
  36:   MachineFunctionInfo *
  37:   clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
  38:         const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
  39:       const override {
  40:     return DestMF.cloneInfo<CSKYMachineFunctionInfo>(*this);
```

- EN: Function bodies or method definitions such as CSKYMachineFunctionInfo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: CSKYMachineFunctionInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 41-60

```cpp
  41:   }
  42: 
  43:   Register getGlobalBaseReg() const { return GlobalBaseReg; }
  44:   void setGlobalBaseReg(Register Reg) { GlobalBaseReg = Reg; }
  45: 
  46:   void setSpillsCR() { SpillsCR = true; }
  47:   bool isCRSpilled() const { return SpillsCR; }
  48: 
  49:   void setVarArgsFrameIndex(int v) { VarArgsFrameIndex = v; }
  50:   int getVarArgsFrameIndex() { return VarArgsFrameIndex; }
  51: 
  52:   unsigned getVarArgsSaveSize() const { return VarArgsSaveSize; }
  53:   void setVarArgsSaveSize(int Size) { VarArgsSaveSize = Size; }
  54: 
  55:   bool isLRSpilled() const { return LRSpilled; }
  56:   void setLRIsSpilled(bool s) { LRSpilled = s; }
  57: 
  58:   void setCalleeSaveAreaSize(int v) { spillAreaSize = v; }
  59:   int getCalleeSaveAreaSize() const { return spillAreaSize; }
  60: 
```

- EN: Function bodies or method definitions such as getGlobalBaseReg, setGlobalBaseReg, setSpillsCR contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getGlobalBaseReg, setGlobalBaseReg, setSpillsCR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-67

```cpp
  61:   unsigned createPICLabelUId() { return ++PICLabelUId; }
  62:   void initPICLabelUId(unsigned UId) { PICLabelUId = UId; }
  63: };
  64: 
  65: } // namespace llvm
  66: 
  67: #endif // LLVM_LIB_TARGET_CSKY_CSKYMACHINEFUNCTIONINFO_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as createPICLabelUId, initPICLabelUId contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 createPICLabelUId, initPICLabelUId 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/CodeGen/MachineFunction.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
