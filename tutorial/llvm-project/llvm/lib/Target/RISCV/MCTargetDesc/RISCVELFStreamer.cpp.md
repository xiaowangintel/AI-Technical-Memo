# RISCVELFStreamer.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVELFStreamer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements ELF streamer hooks for RISC-V assembly emission. / 实现RISC-V 汇编输出的 ELF streamer 钩子。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVELFStreamer.cpp - RISC-V ELF Target Streamer Methods ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file provides RISC-V specific target streamer methods.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-24: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVELFStreamer.h"
#include "RISCVAsmBackend.h"
#include "RISCVBaseInfo.h"
#include "RISCVMCTargetDesc.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 25-37: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
using namespace llvm;

// This part is for ELF object output.
RISCVTargetELFStreamer::RISCVTargetELFStreamer(MCStreamer &S,
                                               const MCSubtargetInfo &STI)
    : RISCVTargetStreamer(S), CurrentVendor("riscv") {
  MCAssembler &MCA = getStreamer().getAssembler();
  const FeatureBitset &Features = STI.getFeatureBits();
  auto &MAB = static_cast<RISCVAsmBackend &>(MCA.getBackend());
  setTargetABI(RISCVABI::computeTargetABI(STI.getTargetTriple(), Features,
                                          MAB.getTargetOptions().getABIName()));
  setFlagsFromFeatures(STI);
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 38-49: Commentary and design intent / 注释与设计意图
```cpp
  // Compute the initial ISA string.  This serves two purposes:
  //   1. Deduplication: subsequent .option arch/rvc/norvc directives compare
  //      against ArchString to avoid propagating redundant ISA updates.
  //   2. Initial symbol: seed the streamer's active ISA so a "$x<ArchString>"
  //      mapping symbol is emitted before the first instruction, recording
  //      the full ISA in the object even when no .option directive is present.
  if (auto ParseResult = RISCVFeatures::parseFeatureBits(
          STI.hasFeature(RISCV::Feature64Bit), Features)) {
    InitialArchString = (*ParseResult)->toString();
    ArchString = InitialArchString;
    getStreamer().setMappingSymbolArch(ArchString);
  }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 50-60: Function implementation: RISCVELFStreamer::RISCVELFStreamer / 函数实现：RISCVELFStreamer::RISCVELFStreamer
```cpp
}

RISCVELFStreamer::RISCVELFStreamer(MCContext &C,
                                   std::unique_ptr<MCAsmBackend> MAB,
                                   std::unique_ptr<MCObjectWriter> MOW,
                                   std::unique_ptr<MCCodeEmitter> MCE)
    : MCELFStreamer(C, std::move(MAB), std::move(MOW), std::move(MCE)) {}

RISCVELFStreamer &RISCVTargetELFStreamer::getStreamer() {
  return static_cast<RISCVELFStreamer &>(Streamer);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 61-71: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVTargetELFStreamer::setArchString(StringRef Arch) {
  if (Arch == ArchString)
    return;
  ArchString = std::string(Arch);
  getStreamer().setMappingSymbolArch(Arch);
}

void RISCVTargetELFStreamer::emitDirectiveOptionPush() {
  ArchStringStack.push_back(ArchString);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 72-86: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVTargetELFStreamer::emitDirectiveOptionPop() {
  if (!ArchStringStack.empty())
    setArchString(ArchStringStack.pop_back_val());
}

void RISCVTargetELFStreamer::emitDirectiveOptionExact() {}
void RISCVTargetELFStreamer::emitDirectiveOptionNoExact() {}
void RISCVTargetELFStreamer::emitDirectiveOptionPIC() {}
void RISCVTargetELFStreamer::emitDirectiveOptionNoPIC() {}
void RISCVTargetELFStreamer::emitDirectiveOptionRelax() {}
void RISCVTargetELFStreamer::emitDirectiveOptionNoRelax() {}
void RISCVTargetELFStreamer::emitDirectiveOptionRVC() {}
void RISCVTargetELFStreamer::emitDirectiveOptionNoRVC() {}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 87-101: Function implementation: RISCVTargetELFStreamer::emitAttribute / 函数实现：RISCVTargetELFStreamer::emitAttribute
```cpp
void RISCVTargetELFStreamer::emitAttribute(unsigned Attribute, unsigned Value) {
  getStreamer().setAttributeItem(Attribute, Value, /*OverwriteExisting=*/true);
}

void RISCVTargetELFStreamer::emitTextAttribute(unsigned Attribute,
                                               StringRef String) {
  getStreamer().setAttributeItem(Attribute, String, /*OverwriteExisting=*/true);
}

void RISCVTargetELFStreamer::emitIntTextAttribute(unsigned Attribute,
                                                  unsigned IntValue,
                                                  StringRef StringValue) {
  getStreamer().setAttributeItems(Attribute, IntValue, StringValue,
                                  /*OverwriteExisting=*/true);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 102-111: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVTargetELFStreamer::finishAttributeSection() {
  RISCVELFStreamer &S = getStreamer();
  if (S.Contents.empty())
    return;

  S.emitAttributesSection(CurrentVendor, ".riscv.attributes",
                          ELF::SHT_RISCV_ATTRIBUTES, AttributeSection);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 112-123: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
void RISCVTargetELFStreamer::finish() {
  RISCVTargetStreamer::finish();
  ELFObjectWriter &W = getStreamer().getWriter();
  RISCVABI::ABI ABI = getTargetABI();

  unsigned EFlags = W.getELFHeaderEFlags();

  if (hasRVC())
    EFlags |= ELF::EF_RISCV_RVC;
  if (hasTSO())
    EFlags |= ELF::EF_RISCV_TSO;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 124-143: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  switch (ABI) {
  case RISCVABI::ABI_ILP32:
  case RISCVABI::ABI_IL32PC64:
  case RISCVABI::ABI_LP64:
  case RISCVABI::ABI_L64PC128:
    break;
  case RISCVABI::ABI_ILP32F:
  case RISCVABI::ABI_IL32PC64F:
  case RISCVABI::ABI_LP64F:
  case RISCVABI::ABI_L64PC128F:
    EFlags |= ELF::EF_RISCV_FLOAT_ABI_SINGLE;
    break;
  case RISCVABI::ABI_ILP32D:
  case RISCVABI::ABI_IL32PC64D:
  case RISCVABI::ABI_LP64D:
  case RISCVABI::ABI_L64PC128D:
    EFlags |= ELF::EF_RISCV_FLOAT_ABI_DOUBLE;
    break;
  case RISCVABI::ABI_ILP32E:
  case RISCVABI::ABI_IL32PC64E:
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 144-153: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case RISCVABI::ABI_LP64E:
    EFlags |= ELF::EF_RISCV_RVE;
    break;
  case RISCVABI::ABI_Unknown:
    llvm_unreachable("Improperly initialised target ABI");
  }

  W.setELFHeaderEFlags(EFlags);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 154-163: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
void RISCVTargetELFStreamer::reset() {
  AttributeSection = nullptr;
  ArchString = InitialArchString;
  ArchStringStack.clear();
  // Re-seed the streamer's active ISA so the first instruction after reset
  // still records the full ISA via "$x<ISA>", matching the behaviour set up
  // in the constructor.
  if (!InitialArchString.empty())
    getStreamer().setMappingSymbolArch(InitialArchString);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 164-180: Function implementation: RISCVTargetELFStreamer::emitDirectiveVariantCC / 函数实现：RISCVTargetELFStreamer::emitDirectiveVariantCC
```cpp

void RISCVTargetELFStreamer::emitDirectiveVariantCC(MCSymbol &Symbol) {
  getStreamer().getAssembler().registerSymbol(Symbol);
  static_cast<MCSymbolELF &>(Symbol).setOther(ELF::STO_RISCV_VARIANT_CC);
}

void RISCVELFStreamer::reset() {
  MCELFStreamer::reset();
  LastMappingSymbols.clear();
  LastEMS = EMS_None;
  MappingSymbolArch.clear();
  LastEmittedArch.clear();
  LastEmittedArchInSection.clear();
  // Call target streamer reset last: it may call setMappingSymbolArch to
  // re-seed the initial ISA after our state has been cleared.
  static_cast<RISCVTargetStreamer *>(getTargetStreamer())->reset();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 181-200: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVELFStreamer::emitDataMappingSymbol() {
  if (LastEMS == EMS_Data)
    return;
  emitMappingSymbol("$d");
  LastEMS = EMS_Data;
}

void RISCVELFStreamer::emitInstructionsMappingSymbol() {
  // Emit a mapping symbol at the start of each instruction run, and whenever
  // the active ISA has changed since the last one emitted in this section.
  // The symbol takes the form "$x<ISA>" when MappingSymbolArch is known, or
  // plain "$x" as a fallback.  The comparison with LastEmittedArch provides
  // deduplication: repeating .option arch with the same ISA, or re-entering a
  // section whose last mapping symbol already matches the active ISA, emits
  // no redundant symbol.
  bool NeedSymbol =
      LastEMS != EMS_Instructions || LastEmittedArch != MappingSymbolArch;
  if (NeedSymbol) {
    if (MappingSymbolArch.empty())
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 201-215: Function implementation: RISCVELFStreamer::emitMappingSymbol / 函数实现：RISCVELFStreamer::emitMappingSymbol
```cpp
      emitMappingSymbol("$x");
    else
      emitMappingSymbol("$x" + MappingSymbolArch);
    LastEmittedArch = MappingSymbolArch;
  }
  LastEMS = EMS_Instructions;
}

void RISCVELFStreamer::emitMappingSymbol(StringRef Name) {
  auto *Symbol =
      static_cast<MCSymbolELF *>(getContext().createLocalSymbol(Name));
  emitLabel(Symbol);
  Symbol->setType(ELF::STT_NOTYPE);
  Symbol->setBinding(ELF::STB_LOCAL);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 216-233: Function implementation: RISCVELFStreamer::setMappingSymbolArch / 函数实现：RISCVELFStreamer::setMappingSymbolArch
```cpp

void RISCVELFStreamer::setMappingSymbolArch(StringRef Arch) {
  MappingSymbolArch = std::string(Arch);
}

void RISCVELFStreamer::changeSection(MCSection *Section, uint32_t Subsection) {
  // We have to keep track of the mapping symbol state of any sections we
  // use. Each one should start off as EMS_None, which is provided as the
  // default constructor by DenseMap::lookup.  The last ISA suffix emitted in
  // each section is also preserved so that re-entering a section only emits a
  // new "$x<ISA>" symbol when the active ISA has actually changed.
  const MCSection *Prev = getPreviousSection().first;
  LastMappingSymbols[Prev] = LastEMS;
  LastEmittedArchInSection[Prev] = LastEmittedArch;
  LastEMS = LastMappingSymbols.lookup(Section);
  auto It = LastEmittedArchInSection.find(Section);
  LastEmittedArch = It != LastEmittedArchInSection.end() ? It->second : "";
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 234-246: Function implementation: MCELFStreamer::changeSection / 函数实现：MCELFStreamer::changeSection
```cpp
  MCELFStreamer::changeSection(Section, Subsection);
}

void RISCVELFStreamer::emitInstruction(const MCInst &Inst,
                                       const MCSubtargetInfo &STI) {
  emitInstructionsMappingSymbol();
  MCELFStreamer::emitInstruction(Inst, STI);
}

void RISCVELFStreamer::emitBytes(StringRef Data) {
  emitDataMappingSymbol();
  MCELFStreamer::emitBytes(Data);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 247-258: Function implementation: RISCVELFStreamer::emitFill / 函数实现：RISCVELFStreamer::emitFill
```cpp

void RISCVELFStreamer::emitFill(const MCExpr &NumBytes, uint64_t FillValue,
                                SMLoc Loc) {
  emitDataMappingSymbol();
  MCELFStreamer::emitFill(NumBytes, FillValue, Loc);
}

void RISCVELFStreamer::emitValueImpl(const MCExpr *Value, unsigned Size,
                                     SMLoc Loc) {
  emitDataMappingSymbol();
  MCELFStreamer::emitValueImpl(Value, Size, Loc);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 259-272: Function implementation: llvm::createRISCVELFStreamer / 函数实现：llvm::createRISCVELFStreamer
```cpp

MCStreamer *llvm::createRISCVELFStreamer(const Triple &, MCContext &C,
                                         std::unique_ptr<MCAsmBackend> &&MAB,
                                         std::unique_ptr<MCObjectWriter> &&MOW,
                                         std::unique_ptr<MCCodeEmitter> &&MCE) {
  return new RISCVELFStreamer(C, std::move(MAB), std::move(MOW),
                              std::move(MCE));
}

void RISCVTargetELFStreamer::emitNoteGnuPropertySection(
    const uint32_t Feature1And) {
  MCStreamer &OutStreamer = getStreamer();
  MCContext &Ctx = OutStreamer.getContext();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 273-283: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const Triple &Triple = Ctx.getTargetTriple();
  Align NoteAlign;
  uint64_t DescSize;
  if (Triple.isArch64Bit()) {
    NoteAlign = Align(8);
    DescSize = 16;
  } else {
    assert(Triple.isArch32Bit());
    NoteAlign = Align(4);
    DescSize = 12;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 284-297: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  assert(Ctx.getObjectFileType() == MCContext::Environment::IsELF);
  MCSection *const NoteSection =
      Ctx.getELFSection(".note.gnu.property", ELF::SHT_NOTE, ELF::SHF_ALLOC);
  OutStreamer.pushSection();
  OutStreamer.switchSection(NoteSection);

  // Emit the note header
  OutStreamer.emitValueToAlignment(NoteAlign);
  OutStreamer.emitIntValue(4, 4);                           // n_namsz
  OutStreamer.emitIntValue(DescSize, 4);                    // n_descsz
  OutStreamer.emitIntValue(ELF::NT_GNU_PROPERTY_TYPE_0, 4); // n_type
  OutStreamer.emitBytes(StringRef("GNU", 4));               // n_name
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 298-307: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Emit n_desc field

  // Emit the feature_1_and property
  OutStreamer.emitIntValue(ELF::GNU_PROPERTY_RISCV_FEATURE_1_AND, 4); // pr_type
  OutStreamer.emitIntValue(4, 4);              // pr_datasz
  OutStreamer.emitIntValue(Feature1And, 4);    // pr_data
  OutStreamer.emitValueToAlignment(NoteAlign); // pr_padding

  OutStreamer.popSection();
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `RISCVELFStreamer.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVAsmBackend.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/BinaryFormat/ELF.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAsmBackend.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAssembler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCCodeEmitter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCContext.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCELFObjectWriter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSubtargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
