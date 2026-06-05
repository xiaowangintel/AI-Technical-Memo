# PPCMCAsmInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCMCAsmInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCMCAsmInfo.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCMCAsmInfo.cpp - PPC asm properties -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. The logic interacts with LLVM's MC layer.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains the declarations of the MCAsmInfoDarwin properties.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains the declarations of the MCAsmInfoDarwin properties.". The logic interacts with LLVM's MC layer.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains the declarations of the MCAsmInfoDarwin properties.”。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 11-48

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCMCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/Triple.h"

using namespace llvm;

void PPCELFMCAsmInfo::anchor() { }

const MCAsmInfo::AtSpecifier elfAtSpecifiers[] = {
    {PPC::S_DTPREL, "DTPREL"},
    {PPC::S_GOT, "GOT"},
    {PPC::S_GOT_HA, "got@ha"},
    {PPC::S_GOT_HI, "got@h"},
    {PPC::S_GOT_LO, "got@l"},
    {PPC::S_HA, "ha"},
    {PPC::S_HI, "h"},
    {PPC::S_HIGH, "high"},
    {PPC::S_HIGHA, "higha"},
    {PPC::S_HIGHER, "higher"},
    {PPC::S_HIGHERA, "highera"},
    {PPC::S_HIGHEST, "highest"},
    {PPC::S_HIGHESTA, "highesta"},
    {PPC::S_LO, "l"},
    {PPC::S_PCREL, "PCREL"},
    {PPC::S_PLT, "PLT"},
    {PPC::S_TLSGD, "tlsgd"},
    {PPC::S_TLSLD, "tlsld"},
    {PPC::S_TOC, "toc"},
    {PPC::S_TOCBASE, "tocbase"},
    {PPC::S_TOC_HA, "toc@ha"},
    {PPC::S_TOC_HI, "toc@h"},
    {PPC::S_TOC_LO, "toc@l"},
    {PPC::S_TPREL, "TPREL"},
    {PPC::S_AIX_TLSGD, "gd"},
    {PPC::S_AIX_TLSGDM, "m"},
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 49-86

```cpp
    {PPC::S_AIX_TLSIE, "ie"},
    {PPC::S_AIX_TLSLD, "ld"},
    {PPC::S_AIX_TLSLE, "le"},
    {PPC::S_AIX_TLSML, "ml"},
    {PPC::S_DTPMOD, "dtpmod"},
    {PPC::S_DTPREL_HA, "dtprel@ha"},
    {PPC::S_DTPREL_HI, "dtprel@h"},
    {PPC::S_DTPREL_HIGH, "dtprel@high"},
    {PPC::S_DTPREL_HIGHA, "dtprel@higha"},
    {PPC::S_DTPREL_HIGHER, "dtprel@higher"},
    {PPC::S_DTPREL_HIGHERA, "dtprel@highera"},
    {PPC::S_DTPREL_HIGHEST, "dtprel@highest"},
    {PPC::S_DTPREL_HIGHESTA, "dtprel@highesta"},
    {PPC::S_DTPREL_LO, "dtprel@l"},
    {PPC::S_GOT_DTPREL, "got@dtprel"},
    {PPC::S_GOT_DTPREL_HA, "got@dtprel@ha"},
    {PPC::S_GOT_DTPREL_HI, "got@dtprel@h"},
    {PPC::S_GOT_DTPREL_LO, "got@dtprel@l"},
    {PPC::S_GOT_PCREL, "got@pcrel"},
    {PPC::S_GOT_TLSGD, "got@tlsgd"},
    {PPC::S_GOT_TLSGD_HA, "got@tlsgd@ha"},
    {PPC::S_GOT_TLSGD_HI, "got@tlsgd@h"},
    {PPC::S_GOT_TLSGD_LO, "got@tlsgd@l"},
    {PPC::S_GOT_TLSGD_PCREL, "got@tlsgd@pcrel"},
    {PPC::S_GOT_TLSLD, "got@tlsld"},
    {PPC::S_GOT_TLSLD_HA, "got@tlsld@ha"},
    {PPC::S_GOT_TLSLD_HI, "got@tlsld@h"},
    {PPC::S_GOT_TLSLD_LO, "got@tlsld@l"},
    {PPC::S_GOT_TLSLD_PCREL, "got@tlsld@pcrel"},
    {PPC::S_GOT_TPREL, "got@tprel"},
    {PPC::S_GOT_TPREL_HA, "got@tprel@ha"},
    {PPC::S_GOT_TPREL_HI, "got@tprel@h"},
    {PPC::S_GOT_TPREL_LO, "got@tprel@l"},
    {PPC::S_GOT_TPREL_PCREL, "got@tprel@pcrel"},
    {PPC::S_LOCAL, "local"},
    {PPC::S_NOTOC, "notoc"},
    {PPC::S_PCREL_OPT, "<<invalid>>"},
    {PPC::S_TLS, "tls"},
```
- **EN**: Continues the PowerPC backend MC layer support for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 MC 层支持，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 87-124

```cpp
    {PPC::S_TLS_PCREL, "tls@pcrel"},
    {PPC::S_TPREL_HA, "tprel@ha"},
    {PPC::S_TPREL_HI, "tprel@h"},
    {PPC::S_TPREL_HIGH, "tprel@high"},
    {PPC::S_TPREL_HIGHA, "tprel@higha"},
    {PPC::S_TPREL_HIGHER, "tprel@higher"},
    {PPC::S_TPREL_HIGHERA, "tprel@highera"},
    {PPC::S_TPREL_HIGHEST, "tprel@highest"},
    {PPC::S_TPREL_HIGHESTA, "tprel@highesta"},
    {PPC::S_TPREL_LO, "tprel@l"},
};

const MCAsmInfo::AtSpecifier xcoffAtSpecifiers[] = {
    // clang-format off
    {PPC::S_AIX_TLSGD, "gd"},
    {PPC::S_AIX_TLSGDM, "m"},
    {PPC::S_AIX_TLSIE, "ie"},
    {PPC::S_AIX_TLSLD, "ld"},
    {PPC::S_AIX_TLSLE, "le"},
    {PPC::S_AIX_TLSML, "ml"},
    {PPC::S_L, "l"},
    {PPC::S_U, "u"},
    // clang-format on
};

static std::optional<int64_t> evaluateAsInt64(uint16_t specifier,
                                              int64_t Value) {
  switch (specifier) {
  case PPC::S_LO:
    return Value & 0xffff;
  case PPC::S_HI:
    return (Value >> 16) & 0xffff;
  case PPC::S_HA:
    return ((Value + 0x8000) >> 16) & 0xffff;
  case PPC::S_HIGH:
    return (Value >> 16) & 0xffff;
  case PPC::S_HIGHA:
    return ((Value + 0x8000) >> 16) & 0xffff;
```
- **EN**: Implements helper routine(s) `evaluateAsInt64` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `evaluateAsInt64`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 125-162

```cpp
  case PPC::S_HIGHER:
    return (Value >> 32) & 0xffff;
  case PPC::S_HIGHERA:
    return ((Value + 0x8000) >> 32) & 0xffff;
  case PPC::S_HIGHEST:
    return (Value >> 48) & 0xffff;
  case PPC::S_HIGHESTA:
    return ((Value + 0x8000) >> 48) & 0xffff;
  default:
    return {};
  }
}

bool PPC::evaluateAsConstant(const MCSpecifierExpr &Expr, int64_t &Res) {
  MCValue Value;

  if (!Expr.getSubExpr()->evaluateAsRelocatable(Value, nullptr))
    return false;

  if (!Value.isAbsolute())
    return false;
  auto Tmp = evaluateAsInt64(Expr.getSpecifier(), Value.getConstant());
  if (!Tmp)
    return false;
  Res = *Tmp;
  return true;
}

static bool evaluateAsRelocatable(const MCSpecifierExpr &Expr, MCValue &Res,
                                  const MCAssembler *Asm) {
  if (!Expr.getSubExpr()->evaluateAsRelocatable(Res, Asm))
    return false;

  // The signedness of the result is dependent on the instruction operand. E.g.
  // in addis 3,3,65535@l, 65535@l is signed. In the absence of information at
  // parse time (!Asm), disable the folding.
  std::optional<int64_t> MaybeInt =
      evaluateAsInt64(Expr.getSpecifier(), Res.getConstant());
```
- **EN**: Implements helper routine(s) `evaluateAsConstant`, `getSubExpr`, `evaluateAsRelocatable` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `evaluateAsConstant`, `getSubExpr`, `evaluateAsRelocatable`。

### Lines 163-200

```cpp
  if (Res.isAbsolute() && MaybeInt) {
    Res = MCValue::get(*MaybeInt);
  } else {
    Res.setSpecifier(Expr.getSpecifier());
  }

  return true;
}

PPCELFMCAsmInfo::PPCELFMCAsmInfo(bool is64Bit, const Triple &T,
                                 const MCTargetOptions &Options)
    : MCAsmInfoELF(Options) {
  // FIXME: This is not always needed. For example, it is not needed in the
  // v2 abi.
  NeedsLocalForSize = true;

  if (is64Bit) {
    CodePointerSize = CalleeSaveStackSlotSize = 8;
  }
  IsLittleEndian =
      T.getArch() == Triple::ppc64le || T.getArch() == Triple::ppcle;

  // ".comm align is in bytes but .align is pow-2."
  AlignmentIsInBytes = false;

  CommentString = "#";

  // Uses '.section' before '.bss' directive
  UsesELFSectionDirectiveForBSS = true;

  // Debug Information
  SupportsDebugInformation = true;

  DollarIsPC = true;

  // Set up DWARF directives
  MinInstAlignment = 4;
```
- **EN**: Implements helper routine(s) `isAbsolute`, `get`, `setSpecifier` for this portion of the PowerPC backend MC layer support for the backend. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `isAbsolute`, `get`, `setSpecifier`。 该区间与栈帧布局或栈访问相关。

### Lines 201-238

```cpp
  // Exceptions handling
  ExceptionsType = ExceptionHandling::DwarfCFI;

  ZeroDirective = "\t.space\t";
  Data64bitsDirective = is64Bit ? "\t.quad\t" : nullptr;
  AssemblerDialect = 1;           // New-Style mnemonics.
  LCOMMDirectiveAlignmentType = LCOMM::ByteAlignment;

  initializeAtSpecifiers(elfAtSpecifiers);
}

void PPCELFMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
                                         const MCSpecifierExpr &Expr) const {
  printExpr(OS, *Expr.getSubExpr());
  OS << '@' << getSpecifierName(Expr.getSpecifier());
}

bool PPCELFMCAsmInfo::evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr,
                                                MCValue &Res,
                                                const MCAssembler *Asm) const {
  return evaluateAsRelocatable(Expr, Res, Asm);
}

PPCXCOFFMCAsmInfo::PPCXCOFFMCAsmInfo(bool Is64Bit, const Triple &T,
                                     const MCTargetOptions &Options)
    : MCAsmInfoXCOFF(Options) {
  if (T.getArch() == Triple::ppc64le || T.getArch() == Triple::ppcle)
    report_fatal_error("XCOFF is not supported for little-endian targets");
  CodePointerSize = CalleeSaveStackSlotSize = Is64Bit ? 8 : 4;

  // A size of 8 is only supported by the assembler under 64-bit.
  Data64bitsDirective = Is64Bit ? "\t.vbyte\t8, " : nullptr;

  // Debug Information
  SupportsDebugInformation = true;

  // Set up DWARF directives
  MinInstAlignment = 4;
```
- **EN**: Implements helper routine(s) `initializeAtSpecifiers`, `printSpecifierExpr`, `printExpr` for this portion of the PowerPC backend MC layer support for the backend. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `initializeAtSpecifiers`, `printSpecifierExpr`, `printExpr`。 该区间与栈帧布局或栈访问相关。

### Lines 239-257

```cpp

  // Support $ as PC in inline asm
  DollarIsPC = true;

  UsesSetToEquateSymbol = true;

  initializeAtSpecifiers(xcoffAtSpecifiers);
}

void PPCXCOFFMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
                                           const MCSpecifierExpr &Expr) const {
  printExpr(OS, *Expr.getSubExpr());
  OS << '@' << getSpecifierName(Expr.getSpecifier());
}

bool PPCXCOFFMCAsmInfo::evaluateAsRelocatableImpl(
    const MCSpecifierExpr &Expr, MCValue &Res, const MCAssembler *Asm) const {
  return evaluateAsRelocatable(Expr, Res, Asm);
}
```
- **EN**: Implements helper routine(s) `initializeAtSpecifiers`, `printSpecifierExpr`, `printExpr` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `initializeAtSpecifiers`, `printSpecifierExpr`, `printExpr`。 相关逻辑会与 LLVM 的 MC 层交互。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Stack frame management / 栈帧管理
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCMCAsmInfo.h`
- `llvm/MC/MCExpr.h`
- `llvm/Support/raw_ostream.h`
- `llvm/TargetParser/Triple.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
