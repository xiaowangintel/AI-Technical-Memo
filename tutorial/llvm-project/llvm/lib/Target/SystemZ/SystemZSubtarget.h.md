# SystemZSubtarget.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZSubtarget.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file models CPU/subtarget features and scheduling properties for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责建模 CPU/子目标特性与调度属性。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZSubtarget.h - SystemZ subtarget information -----*- C++ -*--===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the SystemZ specific subclass of TargetSubtargetInfo.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZSUBTARGET_H
  14: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZSUBTARGET_H
  15: 
  16: #include "SystemZFrameLowering.h"
  17: #include "SystemZISelLowering.h"
  18: #include "SystemZInstrInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZFrameLowering.h`, `SystemZISelLowering.h`, `SystemZInstrInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZFrameLowering.h`, `SystemZISelLowering.h`, `SystemZInstrInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "SystemZRegisterInfo.h"
  20: #include "SystemZSelectionDAGInfo.h"
  21: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  22: #include "llvm/IR/DataLayout.h"
  23: #include "llvm/TargetParser/Triple.h"
  24: #include <string>
  25: 
  26: #define GET_SUBTARGETINFO_HEADER
  27: #include "SystemZGenSubtargetInfo.inc"
  28: 
  29: namespace llvm {
  30: class GlobalValue;
  31: class StringRef;
  32: 
  33: class SystemZSubtarget : public SystemZGenSubtargetInfo {
  34:   virtual void anchor();
  35: protected:
  36: // Bool members corresponding to the SubtargetFeatures defined in tablegen.
```
- **EN**: It imports dependencies such as `SystemZRegisterInfo.h`, `SystemZSelectionDAGInfo.h`, `TargetSubtargetInfo.h`, `DataLayout.h`, `Triple.h`, `string` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `GlobalValue`, `StringRef`, `SystemZSubtarget`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZRegisterInfo.h`, `SystemZSelectionDAGInfo.h`, `TargetSubtargetInfo.h`, `DataLayout.h`, `Triple.h`, `string` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `GlobalValue`, `StringRef`, `SystemZSubtarget` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: #define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
  38:   bool ATTRIBUTE = DEFAULT;
  39: #include "SystemZGenSubtargetInfo.inc"
  40: 
  41: private:
  42:   Triple TargetTriple;
  43:   std::unique_ptr<SystemZCallingConventionRegisters> SpecialRegisters;
  44:   SystemZInstrInfo InstrInfo;
  45:   SystemZTargetLowering TLInfo;
  46:   SystemZSelectionDAGInfo TSInfo;
  47:   std::unique_ptr<const SystemZFrameLowering> FrameLowering;
  48: 
  49:   SystemZSubtarget &initializeSubtargetDependencies(StringRef CPU,
  50:                                                     StringRef TuneCPU,
  51:                                                     StringRef FS);
  52:   SystemZCallingConventionRegisters *initializeSpecialRegisters();
  53: 
  54: public:
```
- **EN**: It imports dependencies such as `SystemZGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `SystemZGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   SystemZSubtarget(const Triple &TT, const std::string &CPU,
  56:                    const std::string &TuneCPU, const std::string &FS,
  57:                    const TargetMachine &TM);
  58: 
  59:   SystemZCallingConventionRegisters *getSpecialRegisters() const {
  60:     assert(SpecialRegisters && "Unsupported SystemZ calling convention");
  61:     return SpecialRegisters.get();
  62:   }
  63: 
  64:   template <class SR> SR &getSpecialRegisters() const {
  65:     return *static_cast<SR *>(getSpecialRegisters());
  66:   }
  67: 
  68:   const TargetFrameLowering *getFrameLowering() const override {
  69:     return FrameLowering.get();
  70:   }
  71: 
  72:   template <class TFL> const TFL *getFrameLowering() const {
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return static_cast<const TFL *>(getFrameLowering());
  74:   }
  75: 
  76:   const SystemZInstrInfo *getInstrInfo() const override { return &InstrInfo; }
  77:   const SystemZRegisterInfo *getRegisterInfo() const override {
  78:     return &InstrInfo.getRegisterInfo();
  79:   }
  80:   const SystemZTargetLowering *getTargetLowering() const override {
  81:     return &TLInfo;
  82:   }
  83:   const SelectionDAGTargetInfo *getSelectionDAGInfo() const override {
  84:     return &TSInfo;
  85:   }
  86: 
  87:   // True if the subtarget should run MachineScheduler after aggressive
  88:   // coalescing. This currently replaces the SelectionDAG scheduler with the
  89:   // "source" order scheduler.
  90:   bool enableMachineScheduler() const override { return true; }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92:   // This is important for reducing register pressure in vector code.
  93:   bool useAA() const override { return true; }
  94: 
  95:   // Always enable the early if-conversion pass.
  96:   bool enableEarlyIfConversion() const override { return true; }
  97: 
  98:   // Enable tracking of subregister liveness in register allocator.
  99:   bool enableSubRegLiveness() const override;
 100: 
 101:   // Automatically generated by tblgen.
 102:   void ParseSubtargetFeatures(StringRef CPU, StringRef TuneCPU, StringRef FS);
 103: 
 104: // Getters for SubtargetFeatures defined in tablegen.
 105: #define GET_SUBTARGETINFO_MACRO(ATTRIBUTE, DEFAULT, GETTER)                    \
 106:   bool GETTER() const { return ATTRIBUTE; }
 107: #include "SystemZGenSubtargetInfo.inc"
 108: 
```
- **EN**: It imports dependencies such as `SystemZGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `GETTER`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `GETTER` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   bool isXRaySupported() const override { return true; }
 110: 
 111:   bool isAddressedViaADA(const GlobalValue *GV) const;
 112: 
 113:   // Return true if GV can be accessed using LARL for reloc model RM
 114:   // and code model CM.
 115:   bool isPC32DBLSymbol(const GlobalValue *GV, CodeModel::Model CM) const;
 116: 
 117:   bool isTargetELF() const { return TargetTriple.isOSBinFormatELF(); }
 118: 
 119:   // Returns TRUE if we are generating GOFF object code
 120:   bool isTargetGOFF() const { return TargetTriple.isOSBinFormatGOFF(); }
 121: 
 122:   // Returns TRUE if we are using XPLINK64 linkage convention
 123:   bool isTargetXPLINK64() const { return (isTargetGOFF() && isTargetzOS()); }
 124: 
 125:   // Returns TRUE if we are generating code for a s390x machine running zOS
 126:   bool isTargetzOS() const { return TargetTriple.isOSzOS(); }
```
- **EN**: The range implements or declares functions including `isTargetELF`, `isTargetGOFF`, `isTargetXPLINK64`, `isTargetzOS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isTargetELF`, `isTargetGOFF`, `isTargetXPLINK64`, `isTargetzOS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-130 / 第 127-130 行
```cpp
 127: };
 128: } // end namespace llvm
 129: 
 130: #endif
```
- **EN**: This span continues the file's main responsibility: this file models CPU/subtarget features and scheduling properties for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZFrameLowering.h`
- `SystemZISelLowering.h`
- `SystemZInstrInfo.h`
- `SystemZRegisterInfo.h`
- `SystemZSelectionDAGInfo.h`
- `llvm/CodeGen/TargetSubtargetInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/TargetParser/Triple.h`
- `string`
- `SystemZGenSubtargetInfo.inc`
