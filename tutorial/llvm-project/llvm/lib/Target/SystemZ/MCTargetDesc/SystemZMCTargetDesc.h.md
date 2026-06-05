# SystemZMCTargetDesc.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZMCTargetDesc.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- SystemZMCTargetDesc.h - SystemZ target descriptions -----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZMCTARGETDESC_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZMCTARGETDESC_H
  11: 
  12: #include "llvm/Support/DataTypes.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `DataTypes.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `DataTypes.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 13-24 / 第 13-24 行
```cpp
  13: 
  14: #include <memory>
  15: 
  16: namespace llvm {
  17: 
  18: class MCAsmBackend;
  19: class MCCodeEmitter;
  20: class MCContext;
  21: class MCInstrInfo;
  22: class MCObjectTargetWriter;
  23: class MCRegisterInfo;
  24: class MCSubtargetInfo;
```
- **EN**: It imports dependencies such as `memory` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MCAsmBackend`, `MCCodeEmitter`, `MCContext`, `MCInstrInfo`, `MCObjectTargetWriter`, `MCRegisterInfo`.
- **CN**: 它引入了 `memory` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MCAsmBackend`, `MCCodeEmitter`, `MCContext`, `MCInstrInfo`, `MCObjectTargetWriter`, `MCRegisterInfo` 等 TableGen 记录。

### Lines 25-36 / 第 25-36 行
```cpp
  25: class MCTargetOptions;
  26: class Target;
  27: 
  28: namespace SystemZMC {
  29: // How many bytes are in the ABI-defined, caller-allocated part of
  30: // a stack frame.
  31: const int64_t ELFCallFrameSize = 160;
  32: 
  33: // The offset of the DWARF CFA from the incoming stack pointer.
  34: const int64_t ELFCFAOffsetFromInitialSP = ELFCallFrameSize;
  35: 
  36: // Maps of asm register numbers to LLVM register numbers, with 0 indicating
```
- **EN**: This block declares or refines TableGen records such as `MCTargetOptions`, `Target`.
- **CN**: 该代码块声明或细化了 `MCTargetOptions`, `Target` 等 TableGen 记录。

### Lines 37-48 / 第 37-48 行
```cpp
  37: // an invalid register.  In principle we could use 32-bit and 64-bit register
  38: // classes directly, provided that we relegated the GPR allocation order
  39: // in SystemZRegisterInfo.td to an AltOrder and left the default order
  40: // as %r0-%r15.  It seems better to provide the same interface for
  41: // all classes though.
  42: extern const unsigned GR32Regs[16];
  43: extern const unsigned GRH32Regs[16];
  44: extern const unsigned GR64Regs[16];
  45: extern const unsigned GR128Regs[16];
  46: extern const unsigned FP16Regs[16];
  47: extern const unsigned FP32Regs[16];
  48: extern const unsigned FP64Regs[16];
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49: extern const unsigned FP128Regs[16];
  50: extern const unsigned VR16Regs[32];
  51: extern const unsigned VR32Regs[32];
  52: extern const unsigned VR64Regs[32];
  53: extern const unsigned VR128Regs[32];
  54: extern const unsigned AR32Regs[16];
  55: extern const unsigned CR64Regs[16];
  56: 
  57: // Return the 0-based number of the first architectural register that
  58: // contains the given LLVM register.   E.g. R1D -> 1.
  59: unsigned getFirstReg(unsigned Reg);
  60: 
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-72 / 第 61-72 行
```cpp
  61: // Return the given register as a GR64.
  62: inline unsigned getRegAsGR64(unsigned Reg) {
  63:   return GR64Regs[getFirstReg(Reg)];
  64: }
  65: 
  66: // Return the given register as a low GR32.
  67: inline unsigned getRegAsGR32(unsigned Reg) {
  68:   return GR32Regs[getFirstReg(Reg)];
  69: }
  70: 
  71: // Return the given register as a high GR32.
  72: inline unsigned getRegAsGRH32(unsigned Reg) {
```
- **EN**: The range implements or declares functions including `getRegAsGR64`, `getRegAsGR32`, `getRegAsGRH32`.
- **CN**: 这一段实现或声明了 `getRegAsGR64`, `getRegAsGR32`, `getRegAsGRH32` 等函数。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   return GRH32Regs[getFirstReg(Reg)];
  74: }
  75: 
  76: // Return the given register as a VR128.
  77: inline unsigned getRegAsVR128(unsigned Reg) {
  78:   return VR128Regs[getFirstReg(Reg)];
  79: }
  80: } // end namespace SystemZMC
  81: 
  82: MCCodeEmitter *createSystemZMCCodeEmitter(const MCInstrInfo &MCII,
  83:                                           MCContext &Ctx);
  84: 
```
- **EN**: The range implements or declares functions including `getRegAsVR128`.
- **CN**: 这一段实现或声明了 `getRegAsVR128` 等函数。

### Lines 85-96 / 第 85-96 行
```cpp
  85: MCAsmBackend *createSystemZMCAsmBackend(const Target &T,
  86:                                         const MCSubtargetInfo &STI,
  87:                                         const MCRegisterInfo &MRI,
  88:                                         const MCTargetOptions &Options);
  89: 
  90: std::unique_ptr<MCObjectTargetWriter>
  91: createSystemZELFObjectWriter(uint8_t OSABI);
  92: std::unique_ptr<MCObjectTargetWriter> createSystemZGOFFObjectWriter();
  93: } // end namespace llvm
  94: 
  95: // Defines symbolic names for SystemZ registers.
  96: // This defines a mapping from register name to register number.
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 97-108 / 第 97-108 行
```cpp
  97: #define GET_REGINFO_ENUM
  98: #include "SystemZGenRegisterInfo.inc"
  99: 
 100: // Defines symbolic names for the SystemZ instructions.
 101: #define GET_INSTRINFO_ENUM
 102: #define GET_INSTRINFO_MC_HELPER_DECLS
 103: #include "SystemZGenInstrInfo.inc"
 104: 
 105: #define GET_SUBTARGETINFO_ENUM
 106: #include "SystemZGenSubtargetInfo.inc"
 107: 
 108: #endif
```
- **EN**: It imports dependencies such as `SystemZGenRegisterInfo.inc`, `SystemZGenInstrInfo.inc`, `SystemZGenSubtargetInfo.inc` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `SystemZGenRegisterInfo.inc`, `SystemZGenInstrInfo.inc`, `SystemZGenSubtargetInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `llvm/Support/DataTypes.h`
- `memory`
- `SystemZGenRegisterInfo.inc`
- `SystemZGenInstrInfo.inc`
- `SystemZGenSubtargetInfo.inc`
