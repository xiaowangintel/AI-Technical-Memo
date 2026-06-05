# M68kSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kSubtarget.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kSubtarget.h - Define Subtarget for the M68k ---------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file declares the M68k specific subclass of TargetSubtargetInfo.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_M68K_M68KSUBTARGET_H
  15: #define LLVM_LIB_TARGET_M68K_M68KSUBTARGET_H
  16: 
  17: #include "M68kFrameLowering.h"
  18: #include "M68kISelLowering.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kFrameLowering.h`, `M68kISelLowering.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kFrameLowering.h`, `M68kISelLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68kInstrInfo.h"
  20: 
  21: #include "llvm/CodeGen/GlobalISel/CallLowering.h"
  22: #include "llvm/CodeGen/GlobalISel/InstructionSelector.h"
  23: #include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
  24: #include "llvm/CodeGen/RegisterBankInfo.h"
  25: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  26: #include "llvm/IR/DataLayout.h"
  27: #include "llvm/MC/MCInstrItineraries.h"
  28: #include "llvm/Support/Alignment.h"
  29: 
  30: #define GET_SUBTARGETINFO_HEADER
  31: #include "M68kGenSubtargetInfo.inc"
  32: 
  33: extern bool M68kReserveGP;
  34: extern bool M68kNoCpload;
  35: 
  36: namespace llvm {
```
- **EN**: It imports dependencies such as `M68kInstrInfo.h`, `CallLowering.h`, `InstructionSelector.h`, `LegalizerInfo.h`, `RegisterBankInfo.h`, `TargetSubtargetInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `M68kInstrInfo.h`, `CallLowering.h`, `InstructionSelector.h`, `LegalizerInfo.h`, `RegisterBankInfo.h`, `TargetSubtargetInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 37-54 / 第 37-54 行
```cpp
  37: class StringRef;
  38: 
  39: class M68kTargetMachine;
  40: 
  41: class M68kSubtarget : public M68kGenSubtargetInfo {
  42:   virtual void anchor();
  43: 
  44: protected:
  45:   // These define which ISA is supported. Since each Motorola M68k ISA is
  46:   // built on top of the previous one whenever an ISA is selected the previous
  47:   // selected as well.
  48:   enum SubtargetEnum { M00, M10, M20, M30, M40, M60 };
  49:   SubtargetEnum SubtargetKind = M00;
  50: 
  51:   enum FPKindEnum { M881, M882 };
  52:   std::optional<FPKindEnum> FPUKind;
  53: 
  54:   std::bitset<M68k::NUM_TARGET_REGS> UserReservedRegister;
```
- **EN**: This block declares or refines TableGen records such as `StringRef`, `M68kTargetMachine`, `M68kSubtarget`.
- **CN**: 该代码块声明或细化了 `StringRef`, `M68kTargetMachine`, `M68kSubtarget` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   InstrItineraryData InstrItins;
  57: 
  58:   /// Small section is used.
  59:   bool UseSmallSection = true;
  60: 
  61:   const M68kTargetMachine &TM;
  62: 
  63:   M68kInstrInfo InstrInfo;
  64:   M68kFrameLowering FrameLowering;
  65:   M68kTargetLowering TLInfo;
  66: 
  67:   /// The minimum alignment known to hold of the stack frame on
  68:   /// entry to the function and which must be maintained by every function.
  69:   unsigned stackAlignment = 8;
  70: 
  71:   Triple TargetTriple;
  72: 
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: public:
  74:   /// This constructor initializes the data members to match that
  75:   /// of the specified triple.
  76:   M68kSubtarget(const Triple &TT, StringRef CPU, StringRef FS,
  77:                 const M68kTargetMachine &_TM);
  78: 
  79:   ~M68kSubtarget() override;
  80: 
  81:   /// Parses features string setting specified subtarget options.  Definition
  82:   /// of function is auto generated by tblgen.
  83:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
  84: 
  85:   bool atLeastM68000() const { return SubtargetKind >= M00; }
  86:   bool atLeastM68010() const { return SubtargetKind >= M10; }
  87:   bool atLeastM68020() const { return SubtargetKind >= M20; }
  88:   bool atLeastM68030() const { return SubtargetKind >= M30; }
  89:   bool atLeastM68040() const { return SubtargetKind >= M40; }
  90:   bool atLeastM68060() const { return SubtargetKind >= M60; }
```
- **EN**: The range implements or declares functions including `atLeastM68000`, `atLeastM68010`, `atLeastM68020`, `atLeastM68030`, `atLeastM68040`.
- **CN**: 这一段实现或声明了 `atLeastM68000`, `atLeastM68010`, `atLeastM68020`, `atLeastM68030`, `atLeastM68040` 等函数。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92:   /// Floating point support
  93:   bool hasFPU() const { return FPUKind.has_value(); }
  94:   bool atLeastM68881() const { return hasFPU() && *FPUKind >= M881; }
  95:   bool atLeastM68882() const { return hasFPU() && *FPUKind >= M882; }
  96: 
  97:   bool useSmallSection() const { return UseSmallSection; }
  98: 
  99:   const Triple &getTargetTriple() const { return TargetTriple; }
 100: 
 101:   bool isTargetELF() const { return TargetTriple.isOSBinFormatELF(); }
 102: 
 103:   /// Return true if the subtarget allows calls to immediate address.
 104:   bool isLegalToCallImmediateAddr() const;
 105: 
 106:   bool isPositionIndependent() const;
 107: 
 108:   bool isRegisterReservedByUser(Register R) const override {
```
- **EN**: The range implements or declares functions including `hasFPU`, `atLeastM68881`, `atLeastM68882`, `useSmallSection`, `isTargetELF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `hasFPU`, `atLeastM68881`, `atLeastM68882`, `useSmallSection`, `isTargetELF` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     assert(R < M68k::NUM_TARGET_REGS && "Register out of range");
 110:     return UserReservedRegister[R];
 111:   }
 112: 
 113:   /// Classify a global variable reference for the current subtarget according
 114:   /// to how we should reference it in a non-pcrel context.
 115:   unsigned char classifyLocalReference(const GlobalValue *GV) const;
 116: 
 117:   /// Classify a global variable reference for the current subtarget according
 118:   /// to how we should reference it in a non-pcrel context.
 119:   unsigned char classifyGlobalReference(const GlobalValue *GV,
 120:                                         const Module &M) const;
 121:   unsigned char classifyGlobalReference(const GlobalValue *GV) const;
 122: 
 123:   /// Classify a external variable reference for the current subtarget according
 124:   /// to how we should reference it in a non-pcrel context.
 125:   unsigned char classifyExternalReference(const Module &M) const;
 126: 
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   /// Classify a global function reference for the current subtarget.
 128:   unsigned char classifyGlobalFunctionReference(const GlobalValue *GV,
 129:                                                 const Module &M) const;
 130:   unsigned char
 131:   classifyGlobalFunctionReference(const GlobalValue *GV) const override;
 132: 
 133:   /// Classify a blockaddress reference for the current subtarget according to
 134:   /// how we should reference it in a non-pcrel context.
 135:   unsigned char classifyBlockAddressReference() const;
 136: 
 137:   unsigned getJumpTableEncoding() const;
 138: 
 139:   /// TODO this must be controlled by options like -malign-int and -mshort
 140:   Align getStackAlignment() const { return Align(stackAlignment); }
 141: 
 142:   /// getSlotSize - Stack slot size in bytes.
 143:   unsigned getSlotSize() const { return 4; }
 144: 
```
- **EN**: The range implements or declares functions including `getStackAlignment`, `getSlotSize`.
- **CN**: 这一段实现或声明了 `getStackAlignment`, `getSlotSize` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   M68kSubtarget &initializeSubtargetDependencies(StringRef CPU, Triple TT,
 146:                                                  StringRef FS,
 147:                                                  const M68kTargetMachine &TM);
 148: 
 149:   const M68kInstrInfo *getInstrInfo() const override { return &InstrInfo; }
 150: 
 151:   const M68kFrameLowering *getFrameLowering() const override {
 152:     return &FrameLowering;
 153:   }
 154: 
 155:   const M68kRegisterInfo *getRegisterInfo() const override {
 156:     return &InstrInfo.getRegisterInfo();
 157:   }
 158: 
 159:   const M68kTargetLowering *getTargetLowering() const override {
 160:     return &TLInfo;
 161:   }
 162: 
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   const InstrItineraryData *getInstrItineraryData() const override {
 164:     return &InstrItins;
 165:   }
 166: 
 167: protected:
 168:   // SelectionDAGISel related APIs.
 169:   std::unique_ptr<const SelectionDAGTargetInfo> TSInfo;
 170: 
 171:   // GlobalISel related APIs.
 172:   std::unique_ptr<CallLowering> CallLoweringInfo;
 173:   std::unique_ptr<InstructionSelector> InstSelector;
 174:   std::unique_ptr<LegalizerInfo> Legalizer;
 175:   std::unique_ptr<RegisterBankInfo> RegBankInfo;
 176: 
 177: public:
 178:   const SelectionDAGTargetInfo *getSelectionDAGInfo() const override;
 179:   const CallLowering *getCallLowering() const override;
 180:   InstructionSelector *getInstructionSelector() const override;
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-186 / 第 181-186 行
```cpp
 181:   const LegalizerInfo *getLegalizerInfo() const override;
 182:   const RegisterBankInfo *getRegBankInfo() const override;
 183: };
 184: } // namespace llvm
 185: 
 186: #endif // LLVM_LIB_TARGET_M68K_M68KSUBTARGET_H
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kFrameLowering.h`
- `M68kISelLowering.h`
- `M68kInstrInfo.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`
- `llvm/CodeGen/GlobalISel/InstructionSelector.h`
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h`
- `llvm/CodeGen/RegisterBankInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/MC/MCInstrItineraries.h`
- `llvm/Support/Alignment.h`
- `M68kGenSubtargetInfo.inc`
