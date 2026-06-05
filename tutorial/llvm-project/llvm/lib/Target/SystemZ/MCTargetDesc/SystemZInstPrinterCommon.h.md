# SystemZInstPrinterCommon.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZInstPrinterCommon.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //== SystemZInstPrinterCommon.h - Common SystemZ InstPrinter funcs *- C++ -*==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This class prints a SystemZ MCInst to a .s file.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: #ifndef LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZINSTPRINTERCOMMON_H
  14: #define LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZINSTPRINTERCOMMON_H
  15: 
  16: #include "SystemZMCAsmInfo.h"
  17: #include "llvm/MC/MCInstPrinter.h"
  18: #include "llvm/MC/MCRegister.h"
  19: #include <cstdint>
  20: 
  21: namespace llvm {
  22: 
  23: class MCOperand;
  24: 
```
- **EN**: It imports dependencies such as `SystemZMCAsmInfo.h`, `MCInstPrinter.h`, `MCRegister.h`, `cstdint` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `MCOperand`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZMCAsmInfo.h`, `MCInstPrinter.h`, `MCRegister.h`, `cstdint` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `MCOperand` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25: class SystemZInstPrinterCommon : public MCInstPrinter {
  26: public:
  27:   SystemZInstPrinterCommon(const MCAsmInfo &MAI, const MCInstrInfo &MII,
  28:                            const MCRegisterInfo &MRI)
  29:       : MCInstPrinter(MAI, MII, MRI) {}
  30: 
  31:   // Print an address with the given base, displacement and index.
  32:   void printAddress(const MCAsmInfo *MAI, MCRegister Base,
  33:                     const MCOperand &DispMO, MCRegister Index, raw_ostream &O);
  34: 
  35:   // Print the given operand.
  36:   void printOperand(const MCOperand &MO, const MCAsmInfo *MAI, raw_ostream &O);
```
- **EN**: This block declares or refines TableGen records such as `SystemZInstPrinterCommon`. The range implements or declares functions including `SystemZInstPrinterCommon`.
- **CN**: 该代码块声明或细化了 `SystemZInstPrinterCommon` 等 TableGen 记录。 这一段实现或声明了 `SystemZInstPrinterCommon` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38:   virtual void printFormattedRegName(const MCAsmInfo *MAI, MCRegister Reg,
  39:                                      raw_ostream &O) {}
  40: 
  41:   // Override MCInstPrinter.
  42:   void printRegName(raw_ostream &O, MCRegister Reg) override;
  43: 
  44: protected:
  45:   template <unsigned N>
  46:   void printUImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  47:   template <unsigned N>
  48:   void printSImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
```
- **EN**: The range implements or declares functions including `printFormattedRegName`.
- **CN**: 这一段实现或声明了 `printFormattedRegName` 等函数。

### Lines 49-60 / 第 49-60 行
```cpp
  49: 
  50:   // Print various types of operand.
  51:   void printOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  52:   void printOperand(const MCInst *MI, uint64_t /*Address*/, unsigned OpNum,
  53:                     raw_ostream &O) {
  54:     printOperand(MI, OpNum, O);
  55:   }
  56:   void printBDAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  57:   void printBDXAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  58:   void printBDLAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  59:   void printBDRAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  60:   void printBDVAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
```
- **EN**: The range implements or declares functions including `printOperand`.
- **CN**: 这一段实现或声明了 `printOperand` 等函数。

### Lines 61-72 / 第 61-72 行
```cpp
  61:   void printLXAAddrOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  62:   void printU1ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  63:   void printU2ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  64:   void printU3ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  65:   void printU4ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  66:   void printS8ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  67:   void printU8ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  68:   void printU12ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  69:   void printS16ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  70:   void printU16ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  71:   void printS32ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  72:   void printU32ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-84 / 第 73-84 行
```cpp
  73:   void printU48ImmOperand(const MCInst *MI, int OpNum, raw_ostream &O);
  74:   void printPCRelOperand(const MCInst *MI, uint64_t Address, int OpNum,
  75:                          raw_ostream &O);
  76:   void printPCRelTLSOperand(const MCInst *MI, uint64_t Address, int OpNum,
  77:                             raw_ostream &O);
  78: 
  79:   // Print the mnemonic for a condition-code mask ("ne", "lh", etc.)
  80:   // This forms part of the instruction name rather than the operand list.
  81:   void printCond4Operand(const MCInst *MI, int OpNum, raw_ostream &O);
  82: };
  83: 
  84: } // end namespace llvm
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 85-86 / 第 85-86 行
```cpp
  85: 
  86: #endif // LLVM_LIB_TARGET_SYSTEMZ_MCTARGETDESC_SYSTEMZINSTPRINTERCOMMON_H
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `SystemZMCAsmInfo.h`
- `llvm/MC/MCInstPrinter.h`
- `llvm/MC/MCRegister.h`
- `cstdint`
