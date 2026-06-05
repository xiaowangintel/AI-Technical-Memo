# MipsTargetObjectFile.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsTargetObjectFile.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsTargetObjectFile` for the Mips backend, focusing on object-file sections, relocations, and data placement.
- 用途 (CN): 实现 Mips 后端中的 `MipsTargetObjectFile`，重点处理目标文件节区、重定位与数据放置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MipsTargetObjectFile.cpp - Mips Object Files ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-20
```cpp
#include "MipsTargetObjectFile.h"
#include "MCTargetDesc/MipsMCAsmInfo.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Target/TargetMachine.h"
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 22-25
```cpp
static cl::opt<unsigned>
SSThreshold("mips-ssection-threshold", cl::Hidden,
            cl::desc("Small data and bss section threshold size (default=8)"),
            cl::init(8));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 27-30
```cpp
static cl::opt<bool>
LocalSData("mlocal-sdata", cl::Hidden,
           cl::desc("MIPS: Use gp_rel for object-local data."),
           cl::init(true));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 32-36
```cpp
static cl::opt<bool>
ExternSData("mextern-sdata", cl::Hidden,
            cl::desc("MIPS: Use gp_rel for data that is not defined by the "
                     "current object."),
            cl::init(true));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 38-42
```cpp
static cl::opt<bool>
EmbeddedData("membedded-data", cl::Hidden,
             cl::desc("MIPS: Try to allocate variables in the following"
                      " sections if possible: .rodata, .sdata, .data ."),
             cl::init(false));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 44-45
```cpp
void MipsTargetObjectFile::Initialize(MCContext &Ctx, const TargetMachine &TM){
  TargetLoweringObjectFileELF::Initialize(Ctx, TM);
```
- EN: Implements `MipsTargetObjectFile::Initialize`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::Initialize`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 47-49
```cpp
  SmallDataSection = getContext().getELFSection(
      ".sdata", ELF::SHT_PROGBITS,
      ELF::SHF_WRITE | ELF::SHF_ALLOC | ELF::SHF_MIPS_GPREL);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 51-55
```cpp
  SmallBSSSection = getContext().getELFSection(".sbss", ELF::SHT_NOBITS,
                                               ELF::SHF_WRITE | ELF::SHF_ALLOC |
                                                   ELF::SHF_MIPS_GPREL);
  this->TM = &static_cast<const MipsTargetMachine &>(TM);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 57-64
```cpp
// A address must be loaded from a small section if its size is less than the
// small section size threshold. Data in this section must be addressed using
// gp_rel operator.
static bool IsInSmallSection(uint64_t Size) {
  // gcc has traditionally not treated zero-sized objects as small data, so this
  // is effectively part of the ABI.
  return Size > 0 && Size <= SSThreshold;
}
```
- EN: Implements `IsInSmallSection`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsInSmallSection`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-74
```cpp
/// Return true if this global address should be placed into small data/bss
/// section.
bool MipsTargetObjectFile::IsGlobalInSmallSection(
    const GlobalObject *GO, const TargetMachine &TM) const {
  // We first check the case where global is a declaration, because finding
  // section kind using getKindForGlobal() is only allowed for global
  // definitions.
  if (GO->isDeclaration() || GO->hasAvailableExternallyLinkage())
    return IsGlobalInSmallSectionImpl(GO, TM);
```
- EN: Implements `MipsTargetObjectFile::IsGlobalInSmallSection`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::IsGlobalInSmallSection`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 76-77
```cpp
  return IsGlobalInSmallSection(GO, TM, getKindForGlobal(GO, TM));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 79-87
```cpp
/// Return true if this global address should be placed into small data/bss
/// section.
bool MipsTargetObjectFile::
IsGlobalInSmallSection(const GlobalObject *GO, const TargetMachine &TM,
                       SectionKind Kind) const {
  return IsGlobalInSmallSectionImpl(GO, TM) &&
         (Kind.isData() || Kind.isBSS() || Kind.isCommon() ||
          Kind.isReadOnly());
}
```
- EN: Implements `IsGlobalInSmallSection`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsGlobalInSmallSection`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-96
```cpp
/// Return true if this global address should be placed into small data/bss
/// section. This method does all the work, except for checking the section
/// kind.
bool MipsTargetObjectFile::
IsGlobalInSmallSectionImpl(const GlobalObject *GO,
                           const TargetMachine &TM) const {
  const MipsSubtarget &Subtarget =
      *static_cast<const MipsTargetMachine &>(TM).getSubtargetImpl();
```
- EN: Implements `IsGlobalInSmallSectionImpl`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `IsGlobalInSmallSectionImpl`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-100
```cpp
  // Return if small section is not available.
  if (!Subtarget.useSmallSection())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 102-105
```cpp
  // Only global variables, not functions.
  const GlobalVariable *GVA = dyn_cast<GlobalVariable>(GO);
  if (!GVA)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 107-110
```cpp
  // If the variable has an explicit section, it is placed in that section but
  // it's addressing mode may change.
  if (GVA->hasSection()) {
    StringRef Section = GVA->getSection();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 112-115
```cpp
    // Explicitly placing any variable in the small data section overrides
    // the global -G value.
    if (Section == ".sdata" || Section == ".sbss")
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 117-121
```cpp
    // Otherwise reject accessing it through the gp pointer. There are some
    // historic cases which GCC doesn't appear to respect any more. These
    // are .lit4, .lit8 and .srdata. For the moment reject these as well.
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-125
```cpp
  // Enforce -mlocal-sdata.
  if (!LocalSData && GVA->hasLocalLinkage())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 127-130
```cpp
  // Enforce -mextern-sdata.
  if (!ExternSData && ((GVA->hasExternalLinkage() && GVA->isDeclaration()) ||
                       GVA->hasCommonLinkage()))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 132-134
```cpp
  // Enforce -membedded-data.
  if (EmbeddedData && GVA->isConstant())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 136-136
```cpp
  Type *Ty = GVA->getValueType();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 138-142
```cpp
  // It is possible that the type of the global is unsized, i.e. a declaration
  // of a extern struct. In this case don't presume it is in the small data
  // section. This happens e.g. when building the FreeBSD kernel.
  if (!Ty->isSized())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 144-146
```cpp
  return IsInSmallSection(
      GVA->getDataLayout().getTypeAllocSize(Ty));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 148-151
```cpp
MCSection *MipsTargetObjectFile::SelectSectionForGlobal(
    const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
  // TODO: Could also support "weak" symbols as well with ".gnu.linkonce.s.*"
  // sections?
```
- EN: Implements `MipsTargetObjectFile::SelectSectionForGlobal`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::SelectSectionForGlobal`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 153-159
```cpp
  // Handle Small Section classification here.
  if (Kind.isBSS() && IsGlobalInSmallSection(GO, TM, Kind))
    return SmallBSSSection;
  if (Kind.isData() && IsGlobalInSmallSection(GO, TM, Kind))
    return SmallDataSection;
  if (Kind.isReadOnly() && IsGlobalInSmallSection(GO, TM, Kind))
    return SmallDataSection;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 161-163
```cpp
  // Otherwise, we work the same as ELF.
  return TargetLoweringObjectFileELF::SelectSectionForGlobal(GO, Kind, TM);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 165-172
```cpp
/// Return true if this constant should be placed into small data section.
bool MipsTargetObjectFile::IsConstantInSmallSection(
    const DataLayout &DL, const Constant *CN, const TargetMachine &TM) const {
  return (static_cast<const MipsTargetMachine &>(TM)
              .getSubtargetImpl()
              ->useSmallSection() &&
          LocalSData && IsInSmallSection(DL.getTypeAllocSize(CN->getType())));
}
```
- EN: Implements `MipsTargetObjectFile::IsConstantInSmallSection`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::IsConstantInSmallSection`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 174-179
```cpp
/// Return true if this constant should be placed into small data section.
MCSection *MipsTargetObjectFile::getSectionForConstant(
    const DataLayout &DL, SectionKind Kind, const Constant *C, Align &Alignment,
    const Function *F) const {
  if (IsConstantInSmallSection(DL, C, *TM))
    return SmallDataSection;
```
- EN: Implements `MipsTargetObjectFile::getSectionForConstant`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::getSectionForConstant`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-184
```cpp
  // Otherwise, we work the same as ELF.
  return TargetLoweringObjectFileELF::getSectionForConstant(DL, Kind, C,
                                                            Alignment, F);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-192
```cpp
const MCExpr *
MipsTargetObjectFile::getDebugThreadLocalSymbol(const MCSymbol *Sym) const {
  const MCExpr *Expr = MCSymbolRefExpr::create(Sym, getContext());
  Expr = MCBinaryExpr::createAdd(
      Expr, MCConstantExpr::create(0x8000, getContext()), getContext());
  return MCSpecifierExpr::create(Expr, Mips::S_DTPREL, getContext());
}
```
- EN: Implements `MipsTargetObjectFile::getDebugThreadLocalSymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetObjectFile::getDebugThreadLocalSymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: object-file sections, relocations, and data placement.
  - CN: 核心职责：目标文件节区、重定位与数据放置。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsTargetObjectFile.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`.
  - CN: 后端本地头文件：`MipsTargetObjectFile.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MipsSubtarget.h`, `MipsTargetMachine.h`。
- EN: LLVM infrastructure headers: `llvm/BinaryFormat/ELF.h`, `llvm/IR/DataLayout.h`, `llvm/IR/GlobalVariable.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCSectionELF.h`, `llvm/Support/CommandLine.h`, `llvm/Target/TargetMachine.h`.
  - CN: LLVM 基础设施头文件：`llvm/BinaryFormat/ELF.h`, `llvm/IR/DataLayout.h`, `llvm/IR/GlobalVariable.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCSectionELF.h`, `llvm/Support/CommandLine.h`, `llvm/Target/TargetMachine.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
