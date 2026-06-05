# CSKYSubtarget.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYSubtarget.h`
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
  13: #ifndef LLVM_LIB_TARGET_CSKY_CSKYSUBTARGET_H
  14: #define LLVM_LIB_TARGET_CSKY_CSKYSUBTARGET_H
  15: 
  16: #include "CSKYFrameLowering.h"
  17: #include "CSKYISelLowering.h"
  18: #include "CSKYInstrInfo.h"
  19: #include "CSKYRegisterInfo.h"
  20: #include "llvm/CodeGen/TargetSubtargetInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/Target/TargetMachine.h"
  22: 
  23: #define GET_SUBTARGETINFO_HEADER
  24: #include "CSKYGenSubtargetInfo.inc"
  25: 
  26: namespace llvm {
  27: class StringRef;
  28: 
  29: class CSKYSubtarget : public CSKYGenSubtargetInfo {
  30:   virtual void anchor();
  31: 
  32:   CSKYFrameLowering FrameLowering;
  33:   CSKYRegisterInfo RegInfo;
  34:   CSKYInstrInfo InstrInfo;
  35:   CSKYTargetLowering TLInfo;
  36:   std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
  37: 
  38:   enum CSKYProcFamilyEnum {
  39:     Others,
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as StringRef, CSKYSubtarget, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 StringRef, CSKYSubtarget 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:     CK801,
  42:     CK802,
  43:     CK803,
  44:     CK803S,
  45:     CK804,
  46:     CK805,
  47:     CK807,
  48:     CK810,
  49:     CK810V,
  50:     CK860,
  51:     CK860V
  52:   };
  53: 
  54:   /// CSKYProcFamily - CSKY processor family: CK801, CK802, and others.
  55:   CSKYProcFamilyEnum CSKYProcFamily = Others;
  56: 
  57:   bool UseHardFloat;
  58:   bool UseHardFloatABI;
  59:   bool HasFPUv2SingleFloat;
  60:   bool HasFPUv2DoubleFloat;
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61:   bool HasFPUv3HalfWord;
  62:   bool HasFPUv3HalfFloat;
  63:   bool HasFPUv3SingleFloat;
  64:   bool HasFPUv3DoubleFloat;
  65:   bool HasFdivdu;
  66:   bool HasFLOATE1;
  67:   bool HasFLOAT1E2;
  68:   bool HasFLOAT1E3;
  69:   bool HasFLOAT3E4;
  70:   bool HasFLOAT7E60;
  71:   bool HasBTST16;
  72:   bool HasExtendLrw;
  73:   bool HasTrust;
  74:   bool HasJAVA;
  75:   bool HasCache;
  76:   bool HasNVIC;
  77:   bool HasDSP;
  78:   bool HasDSP1E2;
  79:   bool HasDSPE60;
  80:   bool HasDSPV2;
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:   bool HasDSP_Silan;
  82:   bool HasDoloop;
  83:   bool HasHardwareDivide;
  84:   bool HasHighRegisters;
  85:   bool HasVDSPV2;
  86:   bool HasVDSP2E3;
  87:   bool HasVDSP2E60F;
  88:   bool ReadTPHard;
  89:   bool HasVDSPV1_128;
  90:   bool UseCCRT;
  91:   bool DumpConstPool;
  92:   bool EnableInterruptAttribute;
  93:   bool HasPushPop;
  94:   bool HasSTM;
  95:   bool SmartMode;
  96:   bool EnableStackSize;
  97: 
  98:   bool HasE1;
  99:   bool HasE2;
 100:   bool Has2E3;
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101:   bool HasMP;
 102:   bool Has3E3r1;
 103:   bool Has3r1E3r2;
 104:   bool Has3r2E3r3;
 105:   bool Has3E7;
 106:   bool HasMP1E2;
 107:   bool Has7E10;
 108:   bool Has10E60;
 109: 
 110: public:
 111:   CSKYSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
 112:                 StringRef FS, const TargetMachine &TM);
 113: 
 114:   ~CSKYSubtarget() override;
 115: 
 116:   const CSKYFrameLowering *getFrameLowering() const override {
 117:     return &FrameLowering;
 118:   }
 119:   const CSKYInstrInfo *getInstrInfo() const override { return &InstrInfo; }
 120:   const CSKYRegisterInfo *getRegisterInfo() const override { return &RegInfo; }
```

- EN: This range continues the implementation of the backend component described by CSKYSubtarget.h, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 121-140

```cpp
 121:   const CSKYTargetLowering *getTargetLowering() const override {
 122:     return &TLInfo;
 123:   }
 124: 
 125:   const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
 126: 
 127:   /// Initializes using the passed in CPU and feature strings so that we can
 128:   /// use initializer lists for subtarget initialization.
 129:   CSKYSubtarget &initializeSubtargetDependencies(const Triple &TT,
 130:                                                  StringRef CPU,
 131:                                                  StringRef TuneCPU,
 132:                                                  StringRef FS);
 133: 
 134:   // Generated by inc file
 135:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
 136: 
 137:   bool useHardFloatABI() const;
 138:   bool useHardFloat() const { return UseHardFloat; }
 139:   bool hasFPUv2SingleFloat() const { return HasFPUv2SingleFloat; }
 140:   bool hasFPUv2DoubleFloat() const { return HasFPUv2DoubleFloat; }
```

- EN: Function bodies or method definitions such as useHardFloat, hasFPUv2SingleFloat, hasFPUv2DoubleFloat contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: useHardFloat, hasFPUv2SingleFloat, hasFPUv2DoubleFloat 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-160

```cpp
 141:   bool hasFPUv2() const { return HasFPUv2SingleFloat || HasFPUv2DoubleFloat; }
 142:   bool hasFPUv3HalfWord() const { return HasFPUv3HalfWord; }
 143:   bool hasFPUv3HalfFloat() const { return HasFPUv3HalfFloat; }
 144:   bool hasFPUv3SingleFloat() const { return HasFPUv3SingleFloat; }
 145:   bool hasFPUv3DoubleFloat() const { return HasFPUv3DoubleFloat; }
 146:   bool hasFPUv3() const {
 147:     return HasFPUv3HalfFloat || HasFPUv3SingleFloat || HasFPUv3DoubleFloat;
 148:   }
 149:   bool hasAnyFloatExt() const { return hasFPUv2() || hasFPUv3(); };
 150:   bool hasFdivdu() const { return HasFdivdu; }
 151:   bool hasFLOATE1() const { return HasFLOATE1; }
 152:   bool hasFLOAT1E2() const { return HasFLOAT1E2; }
 153:   bool hasFLOAT1E3() const { return HasFLOAT1E3; }
 154:   bool hasFLOAT3E4() const { return HasFLOAT3E4; }
 155:   bool hasFLOAT7E60() const { return HasFLOAT7E60; }
 156:   bool hasExtendLrw() const { return HasExtendLrw; }
 157:   bool hasBTST16() const { return HasBTST16; }
 158:   bool hasTrust() const { return HasTrust; }
 159:   bool hasJAVA() const { return HasJAVA; }
 160:   bool hasCache() const { return HasCache; }
```

- EN: Function bodies or method definitions such as hasFPUv2, hasFPUv3HalfWord, hasFPUv3HalfFloat contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: hasFPUv2, hasFPUv3HalfWord, hasFPUv3HalfFloat 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 161-180

```cpp
 161:   bool hasNVIC() const { return HasNVIC; }
 162:   bool hasDSP() const { return HasDSP; }
 163:   bool hasDSP1E2() const { return HasDSP1E2; }
 164:   bool hasDSPE60() const { return HasDSPE60; }
 165:   bool hasDSPV2() const { return HasDSPV2; }
 166:   bool hasDSP_Silan() const { return HasDSP_Silan; }
 167:   bool hasDoloop() const { return HasDoloop; }
 168:   bool hasHighRegisters() const { return HasHighRegisters; }
 169:   bool hasVDSPV2() const { return HasVDSPV2; }
 170:   bool hasVDSPV2_FLOAT() const { return HasVDSPV2 && UseHardFloat; }
 171:   bool hasVDSPV2_HALF() const {
 172:     return HasVDSPV2 && UseHardFloat && HasFPUv3HalfFloat;
 173:   }
 174:   bool hasVDSP2E3() const { return HasVDSP2E3; }
 175:   bool hasVDSP2E60F() const { return HasVDSP2E60F; }
 176:   bool readTPHard() const { return ReadTPHard; }
 177:   bool hasVDSPV1_128() const { return HasVDSPV1_128; }
 178:   bool useCCRT() const { return UseCCRT; }
 179:   bool dumpConstPool() const { return DumpConstPool; }
 180:   bool enableInterruptAttribute() const { return EnableInterruptAttribute; }
```

- EN: Function bodies or method definitions such as hasNVIC, hasDSP, hasDSP1E2 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: hasNVIC, hasDSP, hasDSP1E2 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181:   bool hasPushPop() const { return HasPushPop; }
 182:   bool hasSTM() const { return HasSTM; }
 183:   bool smartMode() const { return SmartMode; }
 184:   bool enableStackSize() const { return EnableStackSize; }
 185: 
 186:   bool hasE1() const { return HasE1; }
 187:   bool hasE2() const { return HasE2; }
 188:   bool has2E3() const { return Has2E3; }
 189:   bool has3r1E3r2() const { return Has3r1E3r2; }
 190:   bool has3r2E3r3() const { return Has3r2E3r3; }
 191:   bool has3E3r1() const { return Has3E3r1; }
 192:   bool has3E7() const { return Has3E7; }
 193:   bool hasMP() const { return HasMP; }
 194:   bool hasMP1E2() const { return HasMP1E2; }
 195:   bool has7E10() const { return Has7E10; }
 196:   bool has10E60() const { return Has10E60; }
 197: 
 198:   bool isCK801() const { return CSKYProcFamily == CK801; }
 199:   bool isCK802() const { return CSKYProcFamily == CK802; }
 200:   bool isCK803() const { return CSKYProcFamily == CK803; }
```

- EN: Function bodies or method definitions such as hasPushPop, hasSTM, smartMode contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: hasPushPop, hasSTM, smartMode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-214

```cpp
 201:   bool isCK803S() const { return CSKYProcFamily == CK803S; }
 202:   bool isCK804() const { return CSKYProcFamily == CK804; }
 203:   bool isCK805() const { return CSKYProcFamily == CK805; }
 204:   bool isCK807() const { return CSKYProcFamily == CK807; }
 205:   bool isCK810() const { return CSKYProcFamily == CK810; }
 206:   bool isCK810V() const { return CSKYProcFamily == CK810V; }
 207:   bool isCK860() const { return CSKYProcFamily == CK860; }
 208:   bool isCK860V() const { return CSKYProcFamily == CK860V; }
 209: 
 210:   const unsigned XLen = 32;
 211: };
 212: } // namespace llvm
 213: 
 214: #endif // LLVM_LIB_TARGET_CSKY_CSKYSUBTARGET_H
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as isCK803S, isCK804, isCK805 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 isCK803S, isCK804, isCK805 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- Feature bits / 特性位
- CPU tuning / CPU 调优
- SelectionDAG lowering / SelectionDAG 降级
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYFrameLowering.h`, `CSKYISelLowering.h`, `CSKYInstrInfo.h`, `CSKYRegisterInfo.h`, `llvm/CodeGen/TargetSubtargetInfo.h`, `llvm/Target/TargetMachine.h`, `CSKYGenSubtargetInfo.inc`
- LLVM subsystems / LLVM 子系统: SelectionDAG
- Generated or companion files / 生成或配套文件: `CSKYGenSubtargetInfo.inc`
- Local companions / 本地配套文件: `CSKYSubtarget.cpp`
