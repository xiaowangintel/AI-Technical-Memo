# MipsMCAsmInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MCTargetDesc/MipsMCAsmInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the declarations of the MipsMCAsmInfo properties.
- 用途 (CN): 实现 Mips 后端中的 `MipsMCAsmInfo`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsMCAsmInfo.cpp - Mips Asm Properties ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the declarations of the MipsMCAsmInfo properties.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-17
```cpp
#include "MipsMCAsmInfo.h"
#include "MipsABIInfo.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/Casting.h"
#include "llvm/TargetParser/Triple.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 19-19
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 21-21
```cpp
void MipsELFMCAsmInfo::anchor() {}
```
- EN: Implements `MipsELFMCAsmInfo::anchor`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsELFMCAsmInfo::anchor`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 23-26
```cpp
MipsELFMCAsmInfo::MipsELFMCAsmInfo(const Triple &TheTriple,
                                   const MCTargetOptions &Options)
    : MCAsmInfoELF(Options) {
  IsLittleEndian = TheTriple.isLittleEndian();
```
- EN: Implements `MipsELFMCAsmInfo::MipsELFMCAsmInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsELFMCAsmInfo::MipsELFMCAsmInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 28-29
```cpp
  MipsABIInfo ABI =
      MipsABIInfo::computeTargetABI(TheTriple, Options.getABIName());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 31-32
```cpp
  if (TheTriple.isMIPS64() && !ABI.IsN32())
    CodePointerSize = CalleeSaveStackSlotSize = 8;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 34-38
```cpp
  if (ABI.IsO32())
    InternalSymbolPrefix = "$";
  else if (ABI.IsN32() || ABI.IsN64())
    InternalSymbolPrefix = ".L";
  PrivateLabelPrefix = InternalSymbolPrefix;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 40-50
```cpp
  AlignmentIsInBytes          = false;
  Data16bitsDirective         = "\t.2byte\t";
  Data32bitsDirective         = "\t.4byte\t";
  Data64bitsDirective         = "\t.8byte\t";
  CommentString               = "#";
  ZeroDirective               = "\t.space\t";
  UseAssignmentForEHBegin = true;
  SupportsDebugInformation = true;
  ExceptionsType = ExceptionHandling::DwarfCFI;
  DwarfRegNumForCFI = true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 52-52
```cpp
void MipsCOFFMCAsmInfo::anchor() {}
```
- EN: Implements `MipsCOFFMCAsmInfo::anchor`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCOFFMCAsmInfo::anchor`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 54-57
```cpp
MipsCOFFMCAsmInfo::MipsCOFFMCAsmInfo(const MCTargetOptions &Options)
    : MCAsmInfoGNUCOFF(Options) {
  HasSingleParameterDotFile = true;
  WinEHEncodingType = WinEH::EncodingType::Itanium;
```
- EN: Implements `MipsCOFFMCAsmInfo::MipsCOFFMCAsmInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCOFFMCAsmInfo::MipsCOFFMCAsmInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-59
```cpp
  ExceptionsType = ExceptionHandling::WinEH;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 61-64
```cpp
  InternalSymbolPrefix = ".L";
  PrivateLabelPrefix = ".L";
  AllowAtInName = true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-71
```cpp
const MCSpecifierExpr *Mips::createGpOff(const MCExpr *Expr, Mips::Specifier S,
                                         MCContext &Ctx) {
  Expr = MCSpecifierExpr::create(Expr, Mips::S_GPREL, Ctx);
  Expr = MCSpecifierExpr::create(Expr, Mips::S_NEG, Ctx);
  return MCSpecifierExpr::create(Expr, S, Ctx);
}
```
- EN: Implements `Mips::createGpOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips::createGpOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 73-75
```cpp
static void printImpl(const MCAsmInfo &MAI, raw_ostream &OS,
                      const MCSpecifierExpr &Expr) {
  int64_t AbsVal;
```
- EN: Implements `printImpl`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printImpl`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 77-91
```cpp
  switch (Expr.getSpecifier()) {
  case Mips::S_None:
  case Mips::S_Special:
    llvm_unreachable("Mips::S_None and MEK_Special are invalid");
    break;
  case Mips::S_DTPREL:
    // Mips::S_DTPREL is used for marking TLS DIEExpr only
    // and contains a regular sub-expression.
    MAI.printExpr(OS, *Expr.getSubExpr());
    return;
  case Mips::S_CALL_HI16:
    OS << "%call_hi";
    break;
  case Mips::S_CALL_LO16:
    OS << "%call_lo";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 92-106
```cpp
    break;
  case Mips::S_DTPREL_HI:
    OS << "%dtprel_hi";
    break;
  case Mips::S_DTPREL_LO:
    OS << "%dtprel_lo";
    break;
  case Mips::S_GOT:
    OS << "%got";
    break;
  case Mips::S_GOTTPREL:
    OS << "%gottprel";
    break;
  case Mips::S_GOT_CALL:
    OS << "%call16";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-121
```cpp
    break;
  case Mips::S_GOT_DISP:
    OS << "%got_disp";
    break;
  case Mips::S_GOT_HI16:
    OS << "%got_hi";
    break;
  case Mips::S_GOT_LO16:
    OS << "%got_lo";
    break;
  case Mips::S_GOT_PAGE:
    OS << "%got_page";
    break;
  case Mips::S_GOT_OFST:
    OS << "%got_ofst";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 122-136
```cpp
    break;
  case Mips::S_GPREL:
    OS << "%gp_rel";
    break;
  case Mips::S_HI:
    OS << "%hi";
    break;
  case Mips::S_HIGHER:
    OS << "%higher";
    break;
  case Mips::S_HIGHEST:
    OS << "%highest";
    break;
  case Mips::S_LO:
    OS << "%lo";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 137-151
```cpp
    break;
  case Mips::S_NEG:
    OS << "%neg";
    break;
  case Mips::S_PCREL_HI16:
    OS << "%pcrel_hi";
    break;
  case Mips::S_PCREL_LO16:
    OS << "%pcrel_lo";
    break;
  case Mips::S_TLSGD:
    OS << "%tlsgd";
    break;
  case Mips::S_TLSLDM:
    OS << "%tlsldm";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 152-159
```cpp
    break;
  case Mips::S_TPREL_HI:
    OS << "%tprel_hi";
    break;
  case Mips::S_TPREL_LO:
    OS << "%tprel_lo";
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-167
```cpp
  OS << '(';
  if (Expr.evaluateAsAbsolute(AbsVal))
    OS << AbsVal;
  else
    MAI.printExpr(OS, *Expr.getSubExpr());
  OS << ')';
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 169-182
```cpp
bool Mips::isGpOff(const MCSpecifierExpr &E) {
  if (E.getSpecifier() == Mips::S_HI || E.getSpecifier() == Mips::S_LO) {
    if (const auto *S1 = dyn_cast<const MCSpecifierExpr>(E.getSubExpr())) {
      if (const auto *S2 = dyn_cast<const MCSpecifierExpr>(S1->getSubExpr())) {
        if (S1->getSpecifier() == Mips::S_NEG &&
            S2->getSpecifier() == Mips::S_GPREL) {
          // S = E.getSpecifier();
          return true;
        }
      }
    }
  }
  return false;
}
```
- EN: Implements `Mips::isGpOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips::isGpOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-194
```cpp
static bool evaluate(const MCSpecifierExpr &Expr, MCValue &Res,
                     const MCAssembler *Asm) {
  // Look for the %hi(%neg(%gp_rel(X))) and %lo(%neg(%gp_rel(X)))
  // special cases.
  if (Mips::isGpOff(Expr)) {
    const MCExpr *SubExpr =
        cast<MCSpecifierExpr>(
            cast<MCSpecifierExpr>(Expr.getSubExpr())->getSubExpr())
            ->getSubExpr();
    if (!SubExpr->evaluateAsRelocatable(Res, Asm))
      return false;
```
- EN: Implements `Mips::isGpOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips::isGpOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-198
```cpp
    Res.setSpecifier(Mips::S_Special);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 200-204
```cpp
  if (!Expr.getSubExpr()->evaluateAsRelocatable(Res, Asm))
    return false;
  Res.setSpecifier(Expr.getSpecifier());
  return !Res.getSubSym();
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 206-209
```cpp
void MipsELFMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
                                          const MCSpecifierExpr &Expr) const {
  printImpl(*this, OS, Expr);
}
```
- EN: Implements `MipsELFMCAsmInfo::printSpecifierExpr`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsELFMCAsmInfo::printSpecifierExpr`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 211-215
```cpp
bool MipsELFMCAsmInfo::evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr,
                                                 MCValue &Res,
                                                 const MCAssembler *Asm) const {
  return evaluate(Expr, Res, Asm);
}
```
- EN: Implements `MipsELFMCAsmInfo::evaluateAsRelocatableImpl`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsELFMCAsmInfo::evaluateAsRelocatableImpl`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 217-220
```cpp
void MipsCOFFMCAsmInfo::printSpecifierExpr(raw_ostream &OS,
                                           const MCSpecifierExpr &Expr) const {
  printImpl(*this, OS, Expr);
}
```
- EN: Implements `MipsCOFFMCAsmInfo::printSpecifierExpr`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCOFFMCAsmInfo::printSpecifierExpr`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 222-225
```cpp
bool MipsCOFFMCAsmInfo::evaluateAsRelocatableImpl(
    const MCSpecifierExpr &Expr, MCValue &Res, const MCAssembler *Asm) const {
  return evaluate(Expr, Res, Asm);
}
```
- EN: Implements `MipsCOFFMCAsmInfo::evaluateAsRelocatableImpl`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsCOFFMCAsmInfo::evaluateAsRelocatableImpl`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsMCAsmInfo.h`, `MipsABIInfo.h`.
  - CN: 后端本地头文件：`MipsMCAsmInfo.h`, `MipsABIInfo.h`。
- EN: LLVM infrastructure headers: `llvm/MC/MCValue.h`, `llvm/Support/Casting.h`, `llvm/TargetParser/Triple.h`.
  - CN: LLVM 基础设施头文件：`llvm/MC/MCValue.h`, `llvm/Support/Casting.h`, `llvm/TargetParser/Triple.h`。
