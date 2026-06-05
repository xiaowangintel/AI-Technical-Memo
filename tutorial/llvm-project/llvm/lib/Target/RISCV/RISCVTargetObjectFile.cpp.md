# RISCVTargetObjectFile.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVTargetObjectFile.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements object-file lowering decisions such as sections and constant placement for RISC-V. / 实现RISC-V 的目标文件下降决策，如节选择与常量放置。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===-- RISCVTargetObjectFile.cpp - RISC-V Object Info --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "RISCVTargetObjectFile.h"
#include "MCTargetDesc/RISCVMCObjectFileInfo.h"
#include "RISCVTargetMachine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCValue.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 19-29: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
using namespace llvm;

unsigned RISCVELFTargetObjectFile::getTextSectionAlignment() const {
  return RISCVMCObjectFileInfo::getTextSectionAlignment(
      *getContext().getSubtargetInfo());
}

void RISCVELFTargetObjectFile::Initialize(MCContext &Ctx,
                                          const TargetMachine &TM) {
  TargetLoweringObjectFileELF::Initialize(Ctx, TM);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 30-47: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  PLTPCRelativeSpecifier = ELF::R_RISCV_PLT32;
  SupportIndirectSymViaGOTPCRel = true;

  SmallDataSection = getContext().getELFSection(
      ".sdata", ELF::SHT_PROGBITS, ELF::SHF_WRITE | ELF::SHF_ALLOC);
  SmallBSSSection = getContext().getELFSection(".sbss", ELF::SHT_NOBITS,
                                               ELF::SHF_WRITE | ELF::SHF_ALLOC);
  SmallRODataSection =
      getContext().getELFSection(".srodata", ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
  SmallROData4Section = getContext().getELFSection(
      ".srodata.cst4", ELF::SHT_PROGBITS, ELF::SHF_ALLOC | ELF::SHF_MERGE, 4);
  SmallROData8Section = getContext().getELFSection(
      ".srodata.cst8", ELF::SHT_PROGBITS, ELF::SHF_ALLOC | ELF::SHF_MERGE, 8);
  SmallROData16Section = getContext().getELFSection(
      ".srodata.cst16", ELF::SHT_PROGBITS, ELF::SHF_ALLOC | ELF::SHF_MERGE, 16);
  SmallROData32Section = getContext().getELFSection(
      ".srodata.cst32", ELF::SHT_PROGBITS, ELF::SHF_ALLOC | ELF::SHF_MERGE, 32);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 48-57: Function implementation: RISCVELFTargetObjectFile::getIndirectSymViaGOTPCRel / 函数实现：RISCVELFTargetObjectFile::getIndirectSymViaGOTPCRel
```cpp

const MCExpr *RISCVELFTargetObjectFile::getIndirectSymViaGOTPCRel(
    const GlobalValue *GV, const MCSymbol *Sym, const MCValue &MV,
    int64_t Offset, MachineModuleInfo *MMI, MCStreamer &Streamer) const {
  auto &Ctx = getContext();
  const MCExpr *Res = MCSymbolRefExpr::create(Sym, Ctx);
  Res = MCBinaryExpr::createAdd(
      Res, MCConstantExpr::create(Offset + MV.getConstant(), Ctx), Ctx);
  return MCSpecifierExpr::create(Res, ELF::R_RISCV_GOT32_PCREL, Ctx);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 58-67: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// A address must be loaded from a small section if its size is less than the
// small section size threshold. Data in this section could be addressed by
// using gp_rel operator.
bool RISCVELFTargetObjectFile::isInSmallSection(uint64_t Size) const {
  // gcc has traditionally not treated zero-sized objects as small data, so this
  // is effectively part of the ABI.
  return Size > 0 && Size <= SSThreshold;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 68-80: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
// Return true if this global address should be placed into small data/bss
// section.
bool RISCVELFTargetObjectFile::isGlobalInSmallSection(
    const GlobalObject *GO, const TargetMachine &TM) const {
  // Only global variables, not functions.
  const GlobalVariable *GVA = dyn_cast<GlobalVariable>(GO);
  if (!GVA)
    return false;

  // If the variable has an explicit section, it is placed in that section.
  if (GVA->hasSection()) {
    StringRef Section = GVA->getSection();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 81-90: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    // Explicitly placing any variable in the small data section overrides
    // the global -G value.
    if (Section == ".sdata" || Section == ".sbss")
      return true;

    // Otherwise reject putting the variable to small section if it has an
    // explicit section name.
    return false;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 91-101: Type declaration and data layout / 类型声明与数据布局
```cpp
  if (((GVA->hasExternalLinkage() && GVA->isDeclaration()) ||
       GVA->hasCommonLinkage()))
    return false;

  Type *Ty = GVA->getValueType();
  // It is possible that the type of the global is unsized, i.e. a declaration
  // of a extern struct. In this case don't presume it is in the small data
  // section. This happens e.g. when building the FreeBSD kernel.
  if (!Ty->isSized())
    return false;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 102-114: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return isInSmallSection(
      GVA->getDataLayout().getTypeAllocSize(Ty));
}

MCSection *RISCVELFTargetObjectFile::SelectSectionForGlobal(
    const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
  // Handle Small Section classification here.
  if (isGlobalInSmallSection(GO, TM)) {
    // Emit to an unique sdata/sbss section when -fdata-section is set.
    // However, if a symbol has an explicit sdata/sbss section, place it in that
    // section.
    bool EmitUniquedSection = TM.getDataSections() && !GO->hasSection();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 115-124: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (Kind.isBSS()) {
      if (EmitUniquedSection) {
        SmallString<128> Name(".sbss.");
        Name.append(GO->getName());
        return getContext().getELFSection(Name.str(), ELF::SHT_NOBITS,
                                          ELF::SHF_WRITE | ELF::SHF_ALLOC);
      }

      return SmallBSSSection;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 125-135: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

    if (Kind.isData()) {
      if (EmitUniquedSection) {
        SmallString<128> Name(".sdata.");
        Name.append(GO->getName());
        return getContext().getELFSection(Name.str(), ELF::SHT_PROGBITS,
                                          ELF::SHF_WRITE | ELF::SHF_ALLOC);
      }

      return SmallDataSection;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 136-146: Function implementation: TargetLoweringObjectFileELF::SelectSectionForGlobal / 函数实现：TargetLoweringObjectFileELF::SelectSectionForGlobal
```cpp
  }

  // Otherwise, we work the same as ELF.
  return TargetLoweringObjectFileELF::SelectSectionForGlobal(GO, Kind, TM);
}

void RISCVELFTargetObjectFile::getModuleMetadata(Module &M) {
  TargetLoweringObjectFileELF::getModuleMetadata(M);
  SmallVector<Module::ModuleFlagEntry, 8> ModuleFlags;
  M.getModuleFlagsMetadata(ModuleFlags);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 147-160: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (const auto &MFE : ModuleFlags) {
    StringRef Key = MFE.Key->getString();
    if (Key == "SmallDataLimit") {
      SSThreshold = mdconst::extract<ConstantInt>(MFE.Val)->getZExtValue();
      break;
    }
  }
}

/// Return true if this constant should be placed into small data section.
bool RISCVELFTargetObjectFile::isConstantInSmallSection(
    const DataLayout &DL, const Constant *CN) const {
  return isInSmallSection(DL.getTypeAllocSize(CN->getType()));
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 161-175: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

MCSection *RISCVELFTargetObjectFile::getSectionForConstant(
    const DataLayout &DL, SectionKind Kind, const Constant *C, Align &Alignment,
    const Function *F) const {

  // The large code model has to put constant pools close to the program, so we
  // put them in the .text section. Large code model doesn't support PIC, so
  // there should be no dynamic relocations that would require `.data.rel.ro`
  // (which could be too far away anyway).
  if (TM->getCodeModel() == CodeModel::Large) {
    if (F)
      return SectionForGlobal(F, SectionKind::getText(), *TM);
    else
      return TextSection;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 176-189: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (C && isConstantInSmallSection(DL, C)) {
    if (Kind.isMergeableConst4())
      return SmallROData4Section;
    if (Kind.isMergeableConst8())
      return SmallROData8Section;
    if (Kind.isMergeableConst16())
      return SmallROData16Section;
    if (Kind.isMergeableConst32())
      return SmallROData32Section;
    // LLVM only generate up to .rodata.cst32, and use .rodata section if more
    // than 32 bytes, so just use .srodata here.
    return SmallRODataSection;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 190-202: Function implementation: TargetLoweringObjectFileELF::getSectionForConstant / 函数实现：TargetLoweringObjectFileELF::getSectionForConstant
```cpp

  // Otherwise, we work the same as ELF.
  return TargetLoweringObjectFileELF::getSectionForConstant(DL, Kind, C,
                                                            Alignment, F);
}

void RISCVMachOTargetObjectFile::getNameWithPrefix(
    SmallVectorImpl<char> &OutName, const GlobalValue *GV,
    const TargetMachine &TM) const {
  // RISC-V does not use section-relative relocations so any global symbol must
  // be accessed via at least a linker-private symbol.
  getMangler().getNameWithPrefix(OutName, GV, /*CannotUsePrivateLabel=*/true);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVTargetObjectFile.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCObjectFileInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/BinaryFormat/ELF.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Mangler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Module.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCContext.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSectionELF.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCValue.h` — Directly referenced by this file. / 该文件直接引用的依赖。
