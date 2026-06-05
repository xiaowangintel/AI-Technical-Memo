# SystemZMCAsmInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/MCTargetDesc/SystemZMCAsmInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //===-- SystemZMCAsmInfo.cpp - SystemZ asm properties ---------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZMCAsmInfo.h"
  10: #include "llvm/MC/MCContext.h"
  11: #include "llvm/MC/MCExpr.h"
  12: #include "llvm/MC/MCValue.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZMCAsmInfo.h`, `MCContext.h`, `MCExpr.h`, `MCValue.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZMCAsmInfo.h`, `MCContext.h`, `MCExpr.h`, `MCValue.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 13-24 / 第 13-24 行
```cpp
  13: 
  14: using namespace llvm;
  15: 
  16: const MCAsmInfo::AtSpecifier atSpecifiers[] = {
  17:     {SystemZ::S_DTPOFF, "DTPOFF"}, {SystemZ::S_GOT, "GOT"},
  18:     {SystemZ::S_GOTENT, "GOTENT"}, {SystemZ::S_INDNTPOFF, "INDNTPOFF"},
  19:     {SystemZ::S_NTPOFF, "NTPOFF"}, {SystemZ::S_PLT, "PLT"},
  20:     {SystemZ::S_TLSGD, "TLSGD"},   {SystemZ::S_TLSLD, "TLSLD"},
  21:     {SystemZ::S_TLSLDM, "TLSLDM"},
  22: };
  23: 
  24: SystemZMCAsmInfoELF::SystemZMCAsmInfoELF(const Triple &TT,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 25-36 / 第 25-36 行
```cpp
  25:                                          const MCTargetOptions &Options)
  26:     : MCAsmInfoELF(Options) {
  27:   AssemblerDialect = AD_GNU;
  28:   CalleeSaveStackSlotSize = 8;
  29:   CodePointerSize = 8;
  30:   Data64bitsDirective = "\t.quad\t";
  31:   ExceptionsType = ExceptionHandling::DwarfCFI;
  32:   IsLittleEndian = false;
  33:   MaxInstLength = 6;
  34:   SupportsDebugInformation = true;
  35:   UsesELFSectionDirectiveForBSS = true;
  36:   ZeroDirective = "\t.space\t";
```
- **EN**: The range implements or declares functions including `MCAsmInfoELF`.
- **CN**: 这一段实现或声明了 `MCAsmInfoELF` 等函数。

### Lines 37-48 / 第 37-48 行
```cpp
  37: 
  38:   initializeAtSpecifiers(atSpecifiers);
  39: }
  40: 
  41: SystemZMCAsmInfoGOFF::SystemZMCAsmInfoGOFF(const Triple &TT,
  42:                                            const MCTargetOptions &Options)
  43:     : MCAsmInfoGOFF(Options) {
  44:   AllowAdditionalComments = false;
  45:   AllowAtInName = true;
  46:   AllowAtAtStartOfIdentifier = true;
  47:   AllowDollarAtStartOfIdentifier = true;
  48:   AssemblerDialect = AD_HLASM;
```
- **EN**: The range implements or declares functions including `MCAsmInfoGOFF`.
- **CN**: 这一段实现或声明了 `MCAsmInfoGOFF` 等函数。

### Lines 49-60 / 第 49-60 行
```cpp
  49:   CalleeSaveStackSlotSize = 8;
  50:   CodePointerSize = 8;
  51:   CommentString = "*";
  52:   UsesSetToEquateSymbol = true;
  53:   ExceptionsType = ExceptionHandling::ZOS;
  54:   IsHLASM = true;
  55:   IsLittleEndian = false;
  56:   MaxInstLength = 6;
  57:   SupportsDebugInformation = true;
  58: 
  59:   initializeAtSpecifiers(atSpecifiers);
  60: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-72 / 第 61-72 行
```cpp
  61: 
  62: bool SystemZMCAsmInfoGOFF::isAcceptableChar(char C) const {
  63:   return MCAsmInfo::isAcceptableChar(C) || C == '#';
  64: }
  65: 
  66: void SystemZMCAsmInfoGOFF::printSpecifierExpr(
  67:     raw_ostream &OS, const MCSpecifierExpr &Expr) const {
  68:   switch (Expr.getSpecifier()) {
  69:   case SystemZ::S_None:
  70:     OS << "AD";
  71:     break;
  72:   case SystemZ::S_QCon:
```
- **EN**: The range implements or declares functions including `SystemZMCAsmInfoGOFF::isAcceptableChar`, `SystemZMCAsmInfoGOFF::printSpecifierExpr`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZMCAsmInfoGOFF::isAcceptableChar`, `SystemZMCAsmInfoGOFF::printSpecifierExpr` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 73-84 / 第 73-84 行
```cpp
  73:     OS << "QD";
  74:     break;
  75:   case SystemZ::S_RCon:
  76:     OS << "RD";
  77:     break;
  78:   case SystemZ::S_VCon:
  79:     OS << "VD";
  80:     break;
  81:   default:
  82:     llvm_unreachable("Invalid kind");
  83:   }
  84:   OS << '(';
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 85-95 / 第 85-95 行
```cpp
  85:   printExpr(OS, *Expr.getSubExpr());
  86:   OS << ')';
  87: }
  88: 
  89: bool SystemZMCAsmInfoGOFF::evaluateAsRelocatableImpl(
  90:     const MCSpecifierExpr &Expr, MCValue &Res, const MCAssembler *Asm) const {
  91:   if (!Expr.getSubExpr()->evaluateAsRelocatable(Res, Asm))
  92:     return false;
  93:   Res.setSpecifier(Expr.getSpecifier());
  94:   return true;
  95: }
```
- **EN**: The range implements or declares functions including `SystemZMCAsmInfoGOFF::evaluateAsRelocatableImpl`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZMCAsmInfoGOFF::evaluateAsRelocatableImpl` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **Target backend structure**: Shows how LLVM splits a backend into MC, CodeGen, and target-specific layers. / 展示 LLVM 如何将后端拆分为 MC、CodeGen 和目标相关层。

## Dependencies / 依赖关系
- `SystemZMCAsmInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCValue.h`
