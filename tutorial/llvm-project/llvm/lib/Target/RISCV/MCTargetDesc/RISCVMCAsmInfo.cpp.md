# RISCVMCAsmInfo.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVMCAsmInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V assembler syntax properties and MC defaults. / 实现RISC-V 汇编语法属性与 MC 默认设置。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVMCAsmInfo.cpp - RISC-V Asm properties ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the declarations of the RISCVMCAsmInfo properties.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-20: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVMCAsmInfo.h"
#include "llvm/BinaryFormat/Dwarf.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/TargetParser/Triple.h"
using namespace llvm;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 21-33: Function implementation: RISCVMCAsmInfo::anchor / 函数实现：RISCVMCAsmInfo::anchor
```cpp
void RISCVMCAsmInfo::anchor() {}

RISCVMCAsmInfo::RISCVMCAsmInfo(const Triple &TT, const MCTargetOptions &Options)
    : MCAsmInfoELF(Options) {
  IsLittleEndian = TT.isLittleEndian();
  CodePointerSize = CalleeSaveStackSlotSize = TT.isArch64Bit() ? 8 : 4;
  CommentString = "#";
  AlignmentIsInBytes = false;
  SupportsDebugInformation = true;
  ExceptionsType = ExceptionHandling::DwarfCFI;
  Data16bitsDirective = "\t.half\t";
  Data32bitsDirective = "\t.word\t";
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 34-49: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

const MCExpr *RISCVMCAsmInfo::getExprForFDESymbol(const MCSymbol *Sym,
                                                  unsigned Encoding,
                                                  MCStreamer &Streamer) const {
  if (!(Encoding & dwarf::DW_EH_PE_pcrel))
    return MCAsmInfo::getExprForFDESymbol(Sym, Encoding, Streamer);

  // The default symbol subtraction results in an ADD/SUB relocation pair.
  // Processing this relocation pair is problematic when linker relaxation is
  // enabled, so we follow binutils in using the R_RISCV_32_PCREL relocation
  // for the FDE initial location.
  MCContext &Ctx = Streamer.getContext();
  const MCExpr *ME = MCSymbolRefExpr::create(Sym, Ctx);
  assert(Encoding & dwarf::DW_EH_PE_sdata4 && "Unexpected encoding");
  return MCSpecifierExpr::create(ME, ELF::R_RISCV_32_PCREL, Ctx);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 50-60: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
                                        const MCSpecifierExpr &Expr) const {
  auto S = Expr.getSpecifier();
  bool HasSpecifier = S != RISCV::S_None && S != RISCV::S_CALL_PLT;
  if (HasSpecifier)
    OS << '%' << RISCV::getSpecifierName(S) << '(';
  printExpr(OS, *Expr.getSubExpr());
  if (HasSpecifier)
    OS << ')';
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 61-75: Function implementation: RISCVMCAsmInfoDarwin::RISCVMCAsmInfoDarwin / 函数实现：RISCVMCAsmInfoDarwin::RISCVMCAsmInfoDarwin
```cpp

RISCVMCAsmInfoDarwin::RISCVMCAsmInfoDarwin(const MCTargetOptions &Options)
    : MCAsmInfoDarwin(Options) {
  CodePointerSize = 4;
  InternalSymbolPrefix = "L";
  PrivateLabelPrefix = "L";
  SeparatorString = "%%";
  CommentString = ";";
  AlignmentIsInBytes = false;
  SupportsDebugInformation = true;
  UseDataRegionDirectives = true;
  ExceptionsType = ExceptionHandling::DwarfCFI;
  Data16bitsDirective = "\t.half\t";
  Data32bitsDirective = "\t.word\t";
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 76-86: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVMCAsmInfoDarwin::printSpecifierExpr(
    raw_ostream &OS, const MCSpecifierExpr &Expr) const {
  auto S = Expr.getSpecifier();
  bool HasSpecifier = S != RISCV::S_None && S != RISCV::S_CALL_PLT;
  if (HasSpecifier)
    OS << '%' << RISCV::getSpecifierName(S) << '(';
  printExpr(OS, *Expr.getSubExpr());
  if (HasSpecifier)
    OS << ')';
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `RISCVMCAsmInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/BinaryFormat/Dwarf.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/BinaryFormat/ELF.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCExpr.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCStreamer.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/TargetParser/Triple.h` — Directly referenced by this file. / 该文件直接引用的依赖。
