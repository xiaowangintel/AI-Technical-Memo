# BPFSubtarget.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFSubtarget.h`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file declares the BPF specific subclass of TargetSubtargetInfo.
- 目的（中文）: 定义目标子架构的 CPU、特性位以及调优信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFSubtarget.h - Define Subtarget for the BPF -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the BPF specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_BPF_BPFSUBTARGET_H
  14: #define LLVM_LIB_TARGET_BPF_BPFSUBTARGET_H
  15: 
  16: #include "BPFFrameLowering.h"
  17: #include "BPFISelLowering.h"
  18: #include "BPFInstrInfo.h"
  19: #include "BPFRegisterInfo.h"
  20: #include "BPFSelectionDAGInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/GlobalISel/CallLowering.h"
  22: #include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
  23: #include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
  24: #include "llvm/CodeGen/RegisterBankInfo.h"
  25: #include "llvm/CodeGen/SelectionDAGTargetInfo.h"
  26: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  27: #include "llvm/IR/DataLayout.h"
  28: #include "llvm/Target/TargetMachine.h"
  29: 
  30: #define GET_SUBTARGETINFO_HEADER
  31: #include "BPFGenSubtargetInfo.inc"
  32: 
  33: namespace llvm {
  34: class StringRef;
  35: 
  36: class BPFSubtarget : public BPFGenSubtargetInfo {
  37:   virtual void anchor();
  38:   BPFInstrInfo InstrInfo;
  39:   BPFFrameLowering FrameLowering;
  40:   BPFTargetLowering TLInfo;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as StringRef, BPFSubtarget, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 StringRef, BPFSubtarget 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:   BPFSelectionDAGInfo TSInfo;
  42: 
  43: private:
  44:   void initializeEnvironment();
  45:   void initSubtargetFeatures(StringRef CPU, StringRef FS);
  46: 
  47: protected:
  48:   // unused
  49:   bool isDummyMode;
  50: 
  51:   bool IsLittleEndian;
  52: 
  53:   // whether the cpu supports jmp ext
  54:   bool HasJmpExt;
  55: 
  56:   // whether the cpu supports jmp32 ext.
  57:   // NOTE: jmp32 is not enabled when alu32 enabled.
  58:   bool HasJmp32;
  59: 
  60:   // whether the cpu supports alu32 instructions.
```

- EN: This range continues the implementation of the backend component described by BPFSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61:   bool HasAlu32;
  62: 
  63:   // whether we should enable MCAsmInfo DwarfUsesRelocationsAcrossSections
  64:   bool UseDwarfRIS;
  65: 
  66:   // whether we allows misaligned memory access
  67:   bool AllowsMisalignedMemAccess;
  68: 
  69:   // whether cpu v4 insns are enabled.
  70:   bool HasLdsx, HasMovsx, HasBswap, HasSdivSmod, HasGotol, HasStoreImm,
  71:       HasLoadAcqStoreRel, HasGotox;
  72: 
  73:   std::unique_ptr<CallLowering> CallLoweringInfo;
  74:   std::unique_ptr<InstructionSelector> InstSelector;
  75:   std::unique_ptr<LegalizerInfo> Legalizer;
  76:   std::unique_ptr<RegisterBankInfo> RegBankInfo;
  77: 
  78: public:
  79:   // This constructor initializes the data members to match that
  80:   // of the specified triple.
```

- EN: This range continues the implementation of the backend component described by BPFSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:   BPFSubtarget(const Triple &TT, const std::string &CPU, const std::string &FS,
  82:                const TargetMachine &TM);
  83: 
  84:   BPFSubtarget &initializeSubtargetDependencies(StringRef CPU, StringRef FS);
  85: 
  86:   // ParseSubtargetFeatures - Parses features string setting specified
  87:   // subtarget options.  Definition of function is auto generated by tblgen.
  88:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
  89:   bool getHasJmpExt() const { return HasJmpExt; }
  90:   bool getHasJmp32() const { return HasJmp32; }
  91:   bool getHasAlu32() const { return HasAlu32; }
  92:   bool getUseDwarfRIS() const { return UseDwarfRIS; }
  93:   bool getAllowsMisalignedMemAccess() const {
  94:     return AllowsMisalignedMemAccess;
  95:   }
  96:   bool hasLdsx() const { return HasLdsx; }
  97:   bool hasMovsx() const { return HasMovsx; }
  98:   bool hasBswap() const { return HasBswap; }
  99:   bool hasSdivSmod() const { return HasSdivSmod; }
 100:   bool hasGotol() const { return HasGotol; }
```

- EN: Function bodies or method definitions such as getHasJmpExt, getHasJmp32, getHasAlu32 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getHasJmpExt, getHasJmp32, getHasAlu32 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-120

```cpp
 101:   bool hasStoreImm() const { return HasStoreImm; }
 102:   bool hasLoadAcqStoreRel() const { return HasLoadAcqStoreRel; }
 103:   bool hasGotox() const { return HasGotox; }
 104: 
 105:   bool isLittleEndian() const { return IsLittleEndian; }
 106: 
 107:   const BPFInstrInfo *getInstrInfo() const override { return &InstrInfo; }
 108:   const BPFFrameLowering *getFrameLowering() const override {
 109:     return &FrameLowering;
 110:   }
 111:   const BPFTargetLowering *getTargetLowering() const override {
 112:     return &TLInfo;
 113:   }
 114:   const BPFSelectionDAGInfo *getSelectionDAGInfo() const override {
 115:     return &TSInfo;
 116:   }
 117:   const BPFRegisterInfo *getRegisterInfo() const override {
 118:     return &InstrInfo.getRegisterInfo();
 119:   }
 120: 
```

- EN: Function bodies or method definitions such as hasStoreImm, hasLoadAcqStoreRel, hasGotox contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: hasStoreImm, hasLoadAcqStoreRel, hasGotox 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-128

```cpp
 121:   const CallLowering *getCallLowering() const override;
 122:   InstructionSelector *getInstructionSelector() const override;
 123:   const LegalizerInfo *getLegalizerInfo() const override;
 124:   const RegisterBankInfo *getRegBankInfo() const override;
 125: };
 126: } // End llvm namespace
 127: 
 128: #endif
```

- EN: This range continues the implementation of the backend component described by BPFSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Feature bits / 特性位
- CPU tuning / CPU 调优
- SelectionDAG lowering / SelectionDAG 降级
- GlobalISel pipeline / GlobalISel 管线
- Register banks / 寄存器银行
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFFrameLowering.h`, `BPFISelLowering.h`, `BPFInstrInfo.h`, `BPFRegisterInfo.h`, `BPFSelectionDAGInfo.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/InstructionSelector.h`, `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
- LLVM subsystems / LLVM 子系统: SelectionDAG, GlobalISel
- Generated or companion files / 生成或配套文件: `BPFGenSubtargetInfo.inc`
- Local companions / 本地配套文件: `BPFSubtarget.cpp`
