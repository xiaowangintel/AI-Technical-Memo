# SystemZ.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZ.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```tablegen
   1: //===-- SystemZ.td - Describe the SystemZ target machine -----*- tblgen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Target-independent interfaces which we are implementing
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```tablegen
  13: include "llvm/Target/Target.td"
  14: 
  15: //===----------------------------------------------------------------------===//
  16: // SystemZ subtarget features
  17: //===----------------------------------------------------------------------===//
  18: 
  19: include "SystemZFeatures.td"
  20: 
  21: //===----------------------------------------------------------------------===//
  22: // SystemZ subtarget scheduling models
  23: //===----------------------------------------------------------------------===//
  24: 
```
- **EN**: It composes TableGen records by including `Target.td`, `SystemZFeatures.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `Target.td`, `SystemZFeatures.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 25-36 / 第 25-36 行
```tablegen
  25: include "SystemZSchedule.td"
  26: 
  27: //===----------------------------------------------------------------------===//
  28: // SystemZ supported processors
  29: //===----------------------------------------------------------------------===//
  30: 
  31: include "SystemZProcessors.td"
  32: 
  33: //===----------------------------------------------------------------------===//
  34: // Register file description
  35: //===----------------------------------------------------------------------===//
  36: 
```
- **EN**: It composes TableGen records by including `SystemZSchedule.td`, `SystemZProcessors.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `SystemZSchedule.td`, `SystemZProcessors.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 37-48 / 第 37-48 行
```tablegen
  37: include "SystemZRegisterInfo.td"
  38: 
  39: //===----------------------------------------------------------------------===//
  40: // Calling convention description
  41: //===----------------------------------------------------------------------===//
  42: 
  43: include "SystemZCallingConv.td"
  44: 
  45: //===----------------------------------------------------------------------===//
  46: // Instruction descriptions
  47: //===----------------------------------------------------------------------===//
  48: 
```
- **EN**: It composes TableGen records by including `SystemZRegisterInfo.td`, `SystemZCallingConv.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `SystemZRegisterInfo.td`, `SystemZCallingConv.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 49-60 / 第 49-60 行
```tablegen
  49: include "SystemZOperators.td"
  50: include "SystemZOperands.td"
  51: include "SystemZPatterns.td"
  52: include "SystemZInstrFormats.td"
  53: include "SystemZInstrInfo.td"
  54: include "SystemZInstrVector.td"
  55: include "SystemZInstrFP.td"
  56: include "SystemZInstrHFP.td"
  57: include "SystemZInstrDFP.td"
  58: include "SystemZInstrSystem.td"
  59: 
  60: 
```
- **EN**: It composes TableGen records by including `SystemZOperators.td`, `SystemZOperands.td`, `SystemZPatterns.td`, `SystemZInstrFormats.td`, `SystemZInstrInfo.td`, `SystemZInstrVector.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `SystemZOperators.td`, `SystemZOperands.td`, `SystemZPatterns.td`, `SystemZInstrFormats.td`, `SystemZInstrInfo.td`, `SystemZInstrVector.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 61-72 / 第 61-72 行
```tablegen
  61: defm : RemapAllTargetPseudoPointerOperands<ADDR64Bit>;
  62: 
  63: def SystemZInstrInfo : InstrInfo { let guessInstructionProperties = 0; }
  64: 
  65: //===----------------------------------------------------------------------===//
  66: // Assembly parser
  67: //===----------------------------------------------------------------------===//
  68: 
  69: def SystemZAsmParser : AsmParser {
  70:   let ShouldEmitMatchRegisterName = 0;
  71: }
  72: 
```
- **EN**: This block declares or refines TableGen records such as `SystemZInstrInfo`, `SystemZAsmParser`.
- **CN**: 该代码块声明或细化了 `SystemZInstrInfo`, `SystemZAsmParser` 等 TableGen 记录。

### Lines 73-84 / 第 73-84 行
```tablegen
  73: def GNUAsmParserVariant : AsmParserVariant {
  74:   int Variant = 0;
  75: 
  76:   // Variant name.
  77:   string Name = "gnu";
  78: }
  79: 
  80: def HLASMAsmParserVariant : AsmParserVariant {
  81:   int Variant = 1;
  82: 
  83:   // Variant name.
  84:   string Name = "hlasm";
```
- **EN**: This block declares or refines TableGen records such as `GNUAsmParserVariant`, `HLASMAsmParserVariant`.
- **CN**: 该代码块声明或细化了 `GNUAsmParserVariant`, `HLASMAsmParserVariant` 等 TableGen 记录。

### Lines 85-96 / 第 85-96 行
```tablegen
  85: }
  86: 
  87: //===----------------------------------------------------------------------===//
  88: // Assembly writer
  89: //===----------------------------------------------------------------------===//
  90: 
  91: // The SystemZ target supports two different syntaxes for emitting machine code.
  92: def GNUAsmWriter : AsmWriter {
  93:   string AsmWriterClassName  = "GNUInstPrinter";
  94:   int Variant = 0;
  95: }
  96: def HLASMAsmWriter : AsmWriter {
```
- **EN**: This block declares or refines TableGen records such as `GNUAsmWriter`, `HLASMAsmWriter`.
- **CN**: 该代码块声明或细化了 `GNUAsmWriter`, `HLASMAsmWriter` 等 TableGen 记录。

### Lines 97-108 / 第 97-108 行
```tablegen
  97:   string AsmWriterClassName  = "HLASMInstPrinter";
  98:   int Variant = 1;
  99: }
 100: 
 101: //===----------------------------------------------------------------------===//
 102: // Top-level target declaration
 103: //===----------------------------------------------------------------------===//
 104: 
 105: def SystemZ : Target {
 106:   let InstructionSet = SystemZInstrInfo;
 107:   let AssemblyParsers = [SystemZAsmParser];
 108:   let AssemblyParserVariants = [GNUAsmParserVariant, HLASMAsmParserVariant];
```
- **EN**: This block declares or refines TableGen records such as `SystemZ`.
- **CN**: 该代码块声明或细化了 `SystemZ` 等 TableGen 记录。

### Lines 109-111 / 第 109-111 行
```tablegen
 109:   let AssemblyWriters = [GNUAsmWriter, HLASMAsmWriter];
 110:   let AllowRegisterRenaming = 1;
 111: }
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `llvm/Target/Target.td`
- `SystemZFeatures.td`
- `SystemZSchedule.td`
- `SystemZProcessors.td`
- `SystemZRegisterInfo.td`
- `SystemZCallingConv.td`
- `SystemZOperators.td`
- `SystemZOperands.td`
- `SystemZPatterns.td`
- `SystemZInstrFormats.td`
- `SystemZInstrInfo.td`
- `SystemZInstrVector.td`
- `SystemZInstrFP.td`
- `SystemZInstrHFP.td`
- `SystemZInstrDFP.td`
- `SystemZInstrSystem.td`
