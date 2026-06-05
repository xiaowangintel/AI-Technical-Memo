# SystemZTargetStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZTargetStreamer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //==-- SystemZTargetStreamer.cpp - SystemZ Target Streamer Methods ----------=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file defines SystemZ-specific target streamer classes.
  11: /// These are for implementing support for target-specific assembly directives.
  12: ///
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "SystemZTargetStreamer.h"
  16: #include "SystemZHLASMAsmStreamer.h"
  17: #include "llvm/MC/MCAsmInfo.h"
  18: #include "llvm/MC/MCGOFFStreamer.h"
  19: #include "llvm/MC/MCObjectFileInfo.h"
  20: 
  21: using namespace llvm;
  22: 
  23: void SystemZTargetStreamer::emitConstantPools() {
  24:   // Emit EXRL target instructions.
```
- **EN**: It imports dependencies such as `SystemZTargetStreamer.h`, `SystemZHLASMAsmStreamer.h`, `MCAsmInfo.h`, `MCGOFFStreamer.h`, `MCObjectFileInfo.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZTargetStreamer::emitConstantPools`.
- **CN**: 它引入了 `SystemZTargetStreamer.h`, `SystemZHLASMAsmStreamer.h`, `MCAsmInfo.h`, `MCGOFFStreamer.h`, `MCObjectFileInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZTargetStreamer::emitConstantPools` 等函数。

### Lines 25-36 / 第 25-36 行
```cpp
  25:   if (EXRLTargets2Sym.empty())
  26:     return;
  27:   // Switch to the .text section.
  28:   const MCObjectFileInfo &OFI = *Streamer.getContext().getObjectFileInfo();
  29:   Streamer.switchSection(OFI.getTextSection());
  30:   for (auto &I : EXRLTargets2Sym) {
  31:     Streamer.emitLabel(I.second);
  32:     const MCInstSTIPair &MCI_STI = I.first;
  33:     Streamer.emitInstruction(MCI_STI.first, *MCI_STI.second);
  34:   }
  35:   EXRLTargets2Sym.clear();
  36: }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38: SystemZHLASMAsmStreamer &SystemZTargetHLASMStreamer::getHLASMStreamer() {
  39:   return static_cast<SystemZHLASMAsmStreamer &>(getStreamer());
  40: }
  41: 
  42: // HLASM statements can only perform a single operation at a time
  43: const MCExpr *SystemZTargetHLASMStreamer::createWordDiffExpr(
  44:     MCContext &Ctx, const MCSymbol *Hi, const MCSymbol *Lo) {
  45:   assert(Hi && Lo && "Symbols required to calculate expression");
  46:   MCSymbol *Temp = Ctx.createTempSymbol();
  47:   OS << Temp->getName() << " EQU ";
  48:   const MCBinaryExpr *TempExpr = MCBinaryExpr::createSub(
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 49-60 / 第 49-60 行
```cpp
  49:       MCSymbolRefExpr::create(Hi, Ctx), MCSymbolRefExpr::create(Lo, Ctx), Ctx);
  50:   Ctx.getAsmInfo().printExpr(OS, *TempExpr);
  51:   OS << "\n";
  52:   return MCBinaryExpr::createLShr(MCSymbolRefExpr::create(Temp, Ctx),
  53:                                   MCConstantExpr::create(1, Ctx), Ctx);
  54: }
  55: 
  56: const MCExpr *SystemZTargetGOFFStreamer::createWordDiffExpr(
  57:     MCContext &Ctx, const MCSymbol *Hi, const MCSymbol *Lo) {
  58:   assert(Hi && Lo && "Symbols required to calculate expression");
  59:   return MCBinaryExpr::createLShr(
  60:       MCBinaryExpr::createSub(MCSymbolRefExpr::create(Hi, Ctx),
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 61-63 / 第 61-63 行
```cpp
  61:                               MCSymbolRefExpr::create(Lo, Ctx), Ctx),
  62:       MCConstantExpr::create(1, Ctx), Ctx);
  63: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。

## Dependencies / 依赖关系
- `SystemZTargetStreamer.h`
- `SystemZHLASMAsmStreamer.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCGOFFStreamer.h`
- `llvm/MC/MCObjectFileInfo.h`
