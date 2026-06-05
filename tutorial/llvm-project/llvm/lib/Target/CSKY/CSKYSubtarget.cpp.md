# CSKYSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYSubtarget.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file declares the CSKY specific subclass of TargetSubtargetInfo.
- 目的（中文）: 定义目标子架构的 CPU、特性位以及调优信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYSubtarget.h - Define Subtarget for the CSKY----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the CSKY specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYSubtarget.h"
  14: #include "CSKYSelectionDAGInfo.h"
  15: #include "llvm/CodeGen/MachineFrameInfo.h"
  16: 
  17: using namespace llvm;
  18: 
  19: #define DEBUG_TYPE "csky-subtarget"
  20: #define GET_SUBTARGETINFO_TARGET_DESC
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 21-40

```cpp
  21: #define GET_SUBTARGETINFO_CTOR
  22: #include "CSKYGenSubtargetInfo.inc"
  23: 
  24: void CSKYSubtarget::anchor() {}
  25: 
  26: CSKYSubtarget &CSKYSubtarget::initializeSubtargetDependencies(
  27:     const Triple &TT, StringRef CPUName, StringRef TuneCPUName, StringRef FS) {
  28: 
  29:   if (CPUName.empty())
  30:     CPUName = "generic";
  31:   if (TuneCPUName.empty())
  32:     TuneCPUName = CPUName;
  33: 
  34:   UseHardFloat = false;
  35:   UseHardFloatABI = false;
  36:   HasFPUv2SingleFloat = false;
  37:   HasFPUv2DoubleFloat = false;
  38:   HasFPUv3HalfWord = false;
  39:   HasFPUv3HalfFloat = false;
  40:   HasFPUv3SingleFloat = false;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as anchor contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 anchor 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 41-60

```cpp
  41:   HasFPUv3DoubleFloat = false;
  42:   HasFdivdu = false;
  43:   HasFLOATE1 = false;
  44:   HasFLOAT1E2 = false;
  45:   HasFLOAT1E3 = false;
  46:   HasFLOAT3E4 = false;
  47:   HasFLOAT7E60 = false;
  48:   HasExtendLrw = false;
  49:   HasBTST16 = false;
  50:   HasTrust = false;
  51:   HasJAVA = false;
  52:   HasCache = false;
  53:   HasNVIC = false;
  54:   HasDSP = false;
  55:   HasDSP1E2 = false;
  56:   HasDSPE60 = false;
  57:   HasDSPV2 = false;
  58:   HasDSP_Silan = false;
  59:   HasDoloop = false;
  60:   HasHardwareDivide = false;
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61:   HasHighRegisters = false;
  62:   HasVDSPV2 = false;
  63:   HasVDSP2E3 = false;
  64:   HasVDSP2E60F = false;
  65:   ReadTPHard = false;
  66:   HasVDSPV1_128 = false;
  67:   UseCCRT = false;
  68:   DumpConstPool = false;
  69:   EnableInterruptAttribute = false;
  70:   HasPushPop = false;
  71:   HasSTM = false;
  72:   SmartMode = false;
  73:   EnableStackSize = false;
  74: 
  75:   HasE1 = false;
  76:   HasE2 = false;
  77:   Has2E3 = false;
  78:   HasMP = false;
  79:   Has3E3r1 = false;
  80:   Has3r1E3r2 = false;
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:   Has3r2E3r3 = false;
  82:   Has3E7 = false;
  83:   HasMP1E2 = false;
  84:   Has7E10 = false;
  85:   Has10E60 = false;
  86: 
  87:   ParseSubtargetFeatures(CPUName, TuneCPUName, FS);
  88:   return *this;
  89: }
  90: 
  91: CSKYSubtarget::CSKYSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
  92:                              StringRef FS, const TargetMachine &TM)
  93:     : CSKYGenSubtargetInfo(TT, CPU, TuneCPU, FS),
  94:       FrameLowering(initializeSubtargetDependencies(TT, CPU, TuneCPU, FS)),
  95:       InstrInfo(*this, RegInfo), TLInfo(TM, *this) {
  96:   TSInfo = std::make_unique<CSKYSelectionDAGInfo>();
  97: }
  98: 
  99: CSKYSubtarget::~CSKYSubtarget() = default;
 100: 
```

- EN: Function bodies or method definitions such as CSKYGenSubtargetInfo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: CSKYGenSubtargetInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 101-112

```cpp
 101: const SelectionDAGTargetInfo *CSKYSubtarget::getSelectionDAGInfo() const {
 102:   return TSInfo.get();
 103: }
 104: 
 105: bool CSKYSubtarget::useHardFloatABI() const {
 106:   auto FloatABI = getTargetLowering()->getTargetMachine().Options.FloatABIType;
 107: 
 108:   if (FloatABI == FloatABI::Default)
 109:     return UseHardFloatABI;
 110:   else
 111:     return FloatABI == FloatABI::Hard;
 112: }
```

- EN: Function bodies or method definitions such as useHardFloatABI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: useHardFloatABI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Feature bits / 特性位
- CPU tuning / CPU 调优
- SelectionDAG lowering / SelectionDAG 降级
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYSubtarget.h`, `CSKYSelectionDAGInfo.h`, `llvm/CodeGen/MachineFrameInfo.h`, `CSKYGenSubtargetInfo.inc`
- LLVM subsystems / LLVM 子系统: SelectionDAG
- Generated or companion files / 生成或配套文件: `CSKYGenSubtargetInfo.inc`
- Local companions / 本地配套文件: `CSKYSubtarget.h`
