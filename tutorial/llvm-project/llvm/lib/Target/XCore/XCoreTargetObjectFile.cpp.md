# XCoreTargetObjectFile.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreTargetObjectFile.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines target-specific object-file section placement, symbol handling, and lowering hooks.
  - **CN**: 定义目标相关的目标文件节区放置、符号处理以及 lowering 钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreTargetObjectFile.cpp - XCore object files --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-16
```cpp

#include "XCoreTargetObjectFile.h"
#include "XCoreSubtarget.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/Target/TargetMachine.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreTargetObjectFile.h`, `XCoreSubtarget.h`, `llvm/BinaryFormat/ELF.h`, `llvm/IR/DataLayout.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreTargetObjectFile.h`, `XCoreSubtarget.h`, `llvm/BinaryFormat/ELF.h`, `llvm/IR/DataLayout.h`。

### Lines 17-30
```cpp
using namespace llvm;


void XCoreTargetObjectFile::Initialize(MCContext &Ctx, const TargetMachine &TM){
  TargetLoweringObjectFileELF::Initialize(Ctx, TM);

  BSSSection = Ctx.getELFSection(".dp.bss", ELF::SHT_NOBITS,
                                 ELF::SHF_ALLOC | ELF::SHF_WRITE |
                                     ELF::XCORE_SHF_DP_SECTION);
  BSSSectionLarge = Ctx.getELFSection(".dp.bss.large", ELF::SHT_NOBITS,
                                      ELF::SHF_ALLOC | ELF::SHF_WRITE |
                                          ELF::XCORE_SHF_DP_SECTION);
  DataSection = Ctx.getELFSection(".dp.data", ELF::SHT_PROGBITS,
                                  ELF::SHF_ALLOC | ELF::SHF_WRITE |
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-44
```cpp
                                      ELF::XCORE_SHF_DP_SECTION);
  DataSectionLarge = Ctx.getELFSection(".dp.data.large", ELF::SHT_PROGBITS,
                                       ELF::SHF_ALLOC | ELF::SHF_WRITE |
                                           ELF::XCORE_SHF_DP_SECTION);
  DataRelROSection = Ctx.getELFSection(".dp.rodata", ELF::SHT_PROGBITS,
                                       ELF::SHF_ALLOC | ELF::SHF_WRITE |
                                           ELF::XCORE_SHF_DP_SECTION);
  DataRelROSectionLarge = Ctx.getELFSection(
      ".dp.rodata.large", ELF::SHT_PROGBITS,
      ELF::SHF_ALLOC | ELF::SHF_WRITE | ELF::XCORE_SHF_DP_SECTION);
  ReadOnlySection =
      Ctx.getELFSection(".cp.rodata", ELF::SHT_PROGBITS,
                        ELF::SHF_ALLOC | ELF::XCORE_SHF_CP_SECTION);
  ReadOnlySectionLarge =
```
- **EN**: Implements logic around `getELFSection`.
- **CN**: 围绕 `getELFSection` 实现具体逻辑。

### Lines 45-58
```cpp
      Ctx.getELFSection(".cp.rodata.large", ELF::SHT_PROGBITS,
                        ELF::SHF_ALLOC | ELF::XCORE_SHF_CP_SECTION);
  MergeableConst4Section = Ctx.getELFSection(
      ".cp.rodata.cst4", ELF::SHT_PROGBITS,
      ELF::SHF_ALLOC | ELF::SHF_MERGE | ELF::XCORE_SHF_CP_SECTION, 4);
  MergeableConst8Section = Ctx.getELFSection(
      ".cp.rodata.cst8", ELF::SHT_PROGBITS,
      ELF::SHF_ALLOC | ELF::SHF_MERGE | ELF::XCORE_SHF_CP_SECTION, 8);
  MergeableConst16Section = Ctx.getELFSection(
      ".cp.rodata.cst16", ELF::SHT_PROGBITS,
      ELF::SHF_ALLOC | ELF::SHF_MERGE | ELF::XCORE_SHF_CP_SECTION, 16);
  CStringSection =
      Ctx.getELFSection(".cp.rodata.string", ELF::SHT_PROGBITS,
                        ELF::SHF_ALLOC | ELF::SHF_MERGE | ELF::SHF_STRINGS |
```
- **EN**: Implements logic around `getELFSection`.
- **CN**: 围绕 `getELFSection` 实现具体逻辑。

### Lines 59-70
```cpp
                            ELF::XCORE_SHF_CP_SECTION);
  // TextSection       - see MObjectFileInfo.cpp
  // StaticCtorSection - see MObjectFileInfo.cpp
  // StaticDtorSection - see MObjectFileInfo.cpp
 }

static unsigned getXCoreSectionType(SectionKind K) {
  if (K.isBSS())
    return ELF::SHT_NOBITS;
  return ELF::SHT_PROGBITS;
}

```
- **EN**: Implements logic around `getXCoreSectionType`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getXCoreSectionType` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 71-83
```cpp
static unsigned getXCoreSectionFlags(SectionKind K, bool IsCPRel) {
  unsigned Flags = 0;

  if (!K.isMetadata())
    Flags |= ELF::SHF_ALLOC;

  if (K.isText())
    Flags |= ELF::SHF_EXECINSTR;
  else if (IsCPRel)
    Flags |= ELF::XCORE_SHF_CP_SECTION;
  else
    Flags |= ELF::XCORE_SHF_DP_SECTION;

```
- **EN**: Implements logic around `getXCoreSectionFlags`; this block applies conditional target rules.
- **CN**: 围绕 `getXCoreSectionFlags` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 84-90
```cpp
  if (K.isWriteable())
    Flags |= ELF::SHF_WRITE;

  if (K.isMergeableCString() || K.isMergeableConst4() ||
      K.isMergeableConst8() || K.isMergeableConst16())
    Flags |= ELF::SHF_MERGE;

```
- **EN**: Implements logic around `isMergeableConst8`; this block applies conditional target rules.
- **CN**: 围绕 `isMergeableConst8` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 91-104
```cpp
  if (K.isMergeableCString())
    Flags |= ELF::SHF_STRINGS;

  return Flags;
}

MCSection *XCoreTargetObjectFile::getExplicitSectionGlobal(
    const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
  StringRef SectionName = GO->getSection();
  // Infer section flags from the section name if we can.
  bool IsCPRel = SectionName.starts_with(".cp.");
  if (IsCPRel && !Kind.isReadOnly())
    report_fatal_error("Using .cp. section for writeable object.");
  return getContext().getELFSection(SectionName, getXCoreSectionType(Kind),
```
- **EN**: Implements logic around `getExplicitSectionGlobal`, `getSection`, `starts_with`, `report_fatal_error`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getExplicitSectionGlobal`, `getSection`, `starts_with`, `report_fatal_error`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 105-112
```cpp
                                    getXCoreSectionFlags(Kind, IsCPRel));
}

MCSection *XCoreTargetObjectFile::SelectSectionForGlobal(
    const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {

  bool UseCPRel = GO->hasLocalLinkage();

```
- **EN**: Implements logic around `getXCoreSectionFlags`, `SelectSectionForGlobal`, `hasLocalLinkage`.
- **CN**: 围绕 `getXCoreSectionFlags`, `SelectSectionForGlobal`, `hasLocalLinkage` 实现具体逻辑。

### Lines 113-126
```cpp
  if (Kind.isText())                    return TextSection;
  if (UseCPRel) {
    if (Kind.isMergeable1ByteCString()) return CStringSection;
    if (Kind.isMergeableConst4())       return MergeableConst4Section;
    if (Kind.isMergeableConst8())       return MergeableConst8Section;
    if (Kind.isMergeableConst16())      return MergeableConst16Section;
  }
  Type *ObjType = GO->getValueType();
  auto &DL = GO->getDataLayout();
  if (TM.getCodeModel() == CodeModel::Small || !ObjType->isSized() ||
      DL.getTypeAllocSize(ObjType) < CodeModelLargeSize) {
    if (Kind.isReadOnly())              return UseCPRel? ReadOnlySection
                                                       : DataRelROSection;
    if (Kind.isBSS() || Kind.isCommon())return BSSSection;
```
- **EN**: Implements logic around `getValueType`, `getDataLayout`, `getTypeAllocSize`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValueType`, `getDataLayout`, `getTypeAllocSize` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 127-138
```cpp
    if (Kind.isData())
      return DataSection;
    if (Kind.isReadOnlyWithRel())       return DataRelROSection;
  } else {
    if (Kind.isReadOnly())              return UseCPRel? ReadOnlySectionLarge
                                                       : DataRelROSectionLarge;
    if (Kind.isBSS() || Kind.isCommon())return BSSSectionLarge;
    if (Kind.isData())
      return DataSectionLarge;
    if (Kind.isReadOnlyWithRel())       return DataRelROSectionLarge;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 139-152
```cpp
  assert((Kind.isThreadLocal() || Kind.isCommon()) && "Unknown section kind");
  report_fatal_error("Target does not support TLS or Common sections");
}

MCSection *XCoreTargetObjectFile::getSectionForConstant(
    const DataLayout &DL, SectionKind Kind, const Constant *C, Align &Alignment,
    const Function *F) const {
  if (Kind.isMergeableConst4())           return MergeableConst4Section;
  if (Kind.isMergeableConst8())           return MergeableConst8Section;
  if (Kind.isMergeableConst16())          return MergeableConst16Section;
  assert((Kind.isReadOnly() || Kind.isReadOnlyWithRel()) &&
         "Unknown section kind");
  // We assume the size of the object is never greater than CodeModelLargeSize.
  // To handle CodeModelLargeSize changes to AsmPrinter would be required.
```
- **EN**: Implements logic around `assert`, `report_fatal_error`, `getSectionForConstant`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `assert`, `report_fatal_error`, `getSectionForConstant` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 153-154
```cpp
  return ReadOnlySection;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Object file lowering / 目标文件 lowering**:
  - **EN**: Chooses sections, symbols, and data placement rules
  - **CN**: 选择节区、符号以及数据放置规则

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreTargetObjectFile.h`, `XCoreSubtarget.h`, `llvm/BinaryFormat/ELF.h`, `llvm/IR/DataLayout.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCSectionELF.h`, `llvm/Target/TargetMachine.h`
- **LLVM subsystems / LLVM 子系统**: MC, IR
