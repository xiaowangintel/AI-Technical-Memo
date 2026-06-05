# LoongArchSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchSubtarget.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===- LoongArchSubtarget.h - Define Subtarget for the LoongArch -*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the LoongArch specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #ifndef LLVM_LIB_TARGET_LOONGARCH_LOONGARCHSUBTARGET_H
  14: #define LLVM_LIB_TARGET_LOONGARCH_LOONGARCHSUBTARGET_H
  15: 
  16: #include "LoongArchFrameLowering.h"
  17: #include "LoongArchISelLowering.h"
  18: #include "LoongArchInstrInfo.h"
  19: #include "LoongArchRegisterInfo.h"
  20: #include "MCTargetDesc/LoongArchBaseInfo.h"
  21: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  22: #include "llvm/IR/DataLayout.h"
  23: #include "llvm/Target/TargetMachine.h"
  24: 
```
- **EN**: It imports dependencies such as `LoongArchFrameLowering.h`, `LoongArchISelLowering.h`, `LoongArchInstrInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchBaseInfo.h`, `TargetSubtargetInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchFrameLowering.h`, `LoongArchISelLowering.h`, `LoongArchInstrInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchBaseInfo.h`, `TargetSubtargetInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25: #define GET_SUBTARGETINFO_HEADER
  26: #include "LoongArchGenSubtargetInfo.inc"
  27: 
  28: namespace llvm {
  29: class StringRef;
  30: 
  31: class LoongArchSubtarget : public LoongArchGenSubtargetInfo {
  32:   virtual void anchor();
  33: 
  34: #define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  35:   bool ATTRIBUTE = DEFAULT;
  36: #include "LoongArchGenSubtargetInfo.inc"
```
- **EN**: It imports dependencies such as `LoongArchGenSubtargetInfo.inc`, `LoongArchGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `StringRef`, `LoongArchSubtarget`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchGenSubtargetInfo.inc`, `LoongArchGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `StringRef`, `LoongArchSubtarget` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38:   unsigned GRLen = 32;
  39:   // TODO: The default value is empirical and conservative. Override the
  40:   // default in initializeProperties once we support optimizing for more
  41:   // uarches.
  42:   uint8_t MaxInterleaveFactor = 2;
  43:   MVT GRLenVT = MVT::i32;
  44:   LoongArchABI::ABI TargetABI = LoongArchABI::ABI_Unknown;
  45:   LoongArchFrameLowering FrameLowering;
  46:   LoongArchInstrInfo InstrInfo;
  47:   LoongArchTargetLowering TLInfo;
  48:   std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   Align PrefFunctionAlignment;
  51:   Align PrefLoopAlignment;
  52:   unsigned MaxBytesForAlignment;
  53: 
  54:   /// Initializes using the passed in CPU and feature strings so that we can
  55:   /// use initializer lists for subtarget initialization.
  56:   LoongArchSubtarget &initializeSubtargetDependencies(const Triple &TT,
  57:                                                       StringRef CPU,
  58:                                                       StringRef TuneCPU,
  59:                                                       StringRef FS,
  60:                                                       StringRef ABIName);
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-72 / 第 61-72 行
```cpp
  61: 
  62:   /// Initialize properties based on the selected processor family.
  63:   void initializeProperties(StringRef TuneCPU);
  64: 
  65: public:
  66:   // Initializes the data members to match that of the specified triple.
  67:   LoongArchSubtarget(const Triple &TT, StringRef CPU, StringRef TuneCPU,
  68:                      StringRef FS, StringRef ABIName, const TargetMachine &TM);
  69: 
  70:   ~LoongArchSubtarget() override;
  71: 
  72:   // Parses features string setting specified subtarget options. The
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   // definition of this function is auto-generated by tblgen.
  74:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
  75: 
  76:   const LoongArchFrameLowering *getFrameLowering() const override {
  77:     return &FrameLowering;
  78:   }
  79:   const LoongArchInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  80:   const LoongArchRegisterInfo *getRegisterInfo() const override {
  81:     return &InstrInfo.getRegisterInfo();
  82:   }
  83:   const LoongArchTargetLowering *getTargetLowering() const override {
  84:     return &TLInfo;
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 85-96 / 第 85-96 行
```cpp
  85:   }
  86: 
  87:   const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
  88: 
  89: #define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  90:   bool GETTER() const { return ATTRIBUTE; }
  91: #include "LoongArchGenSubtargetInfo.inc"
  92: 
  93:   bool is64Bit() const { return HasLA64; }
  94:   MVT getGRLenVT() const { return GRLenVT; }
  95:   unsigned getGRLen() const { return GRLen; }
  96:   LoongArchABI::ABI getTargetABI() const { return TargetABI; }
```
- **EN**: It imports dependencies such as `LoongArchGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `GETTER`, `is64Bit`, `getGRLenVT`, `getGRLen`, `getTargetABI`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `GETTER`, `is64Bit`, `getGRLenVT`, `getGRLen`, `getTargetABI` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 97-108 / 第 97-108 行
```cpp
  97:   bool isSoftFPABI() const {
  98:     return TargetABI == LoongArchABI::ABI_LP64S ||
  99:            TargetABI == LoongArchABI::ABI_ILP32S;
 100:   }
 101:   bool isXRaySupported() const override { return is64Bit(); }
 102:   Align getPrefFunctionAlignment() const { return PrefFunctionAlignment; }
 103:   Align getPrefLoopAlignment() const { return PrefLoopAlignment; }
 104:   unsigned getMaxBytesForAlignment() const { return MaxBytesForAlignment; }
 105:   unsigned getMaxInterleaveFactor() const { return MaxInterleaveFactor; }
 106:   bool enableMachineScheduler() const override { return true; }
 107:   bool useAA() const override;
 108: };
```
- **EN**: The range implements or declares functions including `isSoftFPABI`, `getPrefFunctionAlignment`, `getPrefLoopAlignment`, `getMaxBytesForAlignment`, `getMaxInterleaveFactor`.
- **CN**: 这一段实现或声明了 `isSoftFPABI`, `getPrefFunctionAlignment`, `getPrefLoopAlignment`, `getMaxBytesForAlignment`, `getMaxInterleaveFactor` 等函数。

### Lines 109-111 / 第 109-111 行
```cpp
 109: } // end namespace llvm
 110: 
 111: #endif // LLVM_LIB_TARGET_LOONGARCH_LOONGARCHSUBTARGET_H
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `LoongArchFrameLowering.h`
- `LoongArchISelLowering.h`
- `LoongArchInstrInfo.h`
- `LoongArchRegisterInfo.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/Target/TargetMachine.h`
- `LoongArchGenSubtargetInfo.inc`
