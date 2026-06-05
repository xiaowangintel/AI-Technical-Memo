# AArch64TargetStreamer.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64TargetStreamer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements the AArch64TargetStreamer class. / 该文件实现 AArch64 后端中的MC 流与目标文件输出。
## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Documented code section
```cpp
//===- AArch64TargetStreamer.cpp - AArch64TargetStreamer class ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AArch64TargetStreamer class.
//
//===----------------------------------------------------------------------===//

#include "AArch64TargetStreamer.h"
#include "AArch64MCAsmInfo.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/ConstantPools.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFStreamer.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/CommandLine.h"

using namespace llvm;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 25-47: Function AArch64TargetStreamer::AArch64TargetStreamer
```cpp

static cl::opt<bool> MarkBTIProperty(
    "aarch64-mark-bti-property", cl::Hidden,
    cl::desc("Add .note.gnu.property with BTI to assembly files"),
    cl::init(false));

//
// AArch64TargetStreamer Implementation
//
AArch64TargetStreamer::AArch64TargetStreamer(MCStreamer &S)
    : MCTargetStreamer(S), ConstantPools(new AssemblerConstantPools()) {}

AArch64TargetStreamer::~AArch64TargetStreamer() = default;

void AArch64TargetStreamer::emitAuthValue(const MCExpr *Expr,
                                          uint16_t Discriminator,
                                          AArch64PACKey::ID Key,
                                          bool HasAddressDiversity) {
  Streamer.emitValueImpl(AArch64AuthMCExpr::create(Expr, Discriminator, Key,
                                                   HasAddressDiversity,
                                                   Streamer.getContext()),
                         8);
}
```
**EN:** This block implements AArch64TargetStreamer::AArch64TargetStreamer, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64TargetStreamer::AArch64TargetStreamer，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 48-70: Function AArch64TargetStreamer::addConstantPoolEntry
```cpp

// The constant pool handling is shared by all AArch64TargetStreamer
// implementations.
const MCExpr *AArch64TargetStreamer::addConstantPoolEntry(const MCExpr *Expr,
                                                          unsigned Size,
                                                          SMLoc Loc) {
  return ConstantPools->addEntry(Streamer, Expr, Size, Loc);
}

void AArch64TargetStreamer::emitCurrentConstantPool() {
  ConstantPools->emitForCurrentSection(Streamer);
}

void AArch64TargetStreamer::emitConstantPools() {
  ConstantPools->emitAll(Streamer);
}

// finish() - write out any non-empty assembler constant pools and
//   write out note.gnu.properties if need.
void AArch64TargetStreamer::finish() {
  if (MarkBTIProperty)
    emitNoteSection(ELF::GNU_PROPERTY_AARCH64_FEATURE_1_BTI);
}
```
**EN:** This block implements AArch64TargetStreamer::addConstantPoolEntry, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64TargetStreamer::addConstantPoolEntry，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 71-98: Function AArch64TargetStreamer::emitNoteSection
```cpp

void AArch64TargetStreamer::emitNoteSection(unsigned Flags,
                                            uint64_t PAuthABIPlatform,
                                            uint64_t PAuthABIVersion) {
  assert((PAuthABIPlatform == uint64_t(-1)) ==
         (PAuthABIVersion == uint64_t(-1)));
  uint64_t DescSz = 0;
  if (Flags != 0)
    DescSz += 4 * 4;
  if (PAuthABIPlatform != uint64_t(-1))
    DescSz += 4 + 4 + 8 * 2;
  if (DescSz == 0)
    return;

  MCStreamer &OutStreamer = getStreamer();
  MCContext &Context = OutStreamer.getContext();
  // Emit a .note.gnu.property section with the flags.
  MCSectionELF *Nt = Context.getELFSection(".note.gnu.property", ELF::SHT_NOTE,
                                           ELF::SHF_ALLOC);
  if (Nt->isRegistered()) {
    SMLoc Loc;
    Context.reportWarning(
        Loc,
        "The .note.gnu.property is not emitted because it is already present.");
    return;
  }
  MCSection *Cur = OutStreamer.getCurrentSectionOnly();
  OutStreamer.switchSection(Nt);
```
**EN:** This block implements AArch64TargetStreamer::emitNoteSection, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64TargetStreamer::emitNoteSection，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 99-125: Documented code section
```cpp

  // Emit the note header.
  OutStreamer.emitValueToAlignment(Align(8));
  OutStreamer.emitIntValue(4, 4);     // data size for "GNU\0"
  OutStreamer.emitIntValue(DescSz, 4); // Elf_Prop array size
  OutStreamer.emitIntValue(ELF::NT_GNU_PROPERTY_TYPE_0, 4);
  OutStreamer.emitBytes(StringRef("GNU", 4)); // note name

  // Emit the PAC/BTI properties.
  if (Flags != 0) {
    OutStreamer.emitIntValue(ELF::GNU_PROPERTY_AARCH64_FEATURE_1_AND, 4);
    OutStreamer.emitIntValue(4, 4);     // data size
    OutStreamer.emitIntValue(Flags, 4); // data
    OutStreamer.emitIntValue(0, 4);     // pad
  }

  // Emit the PAuth ABI compatibility info
  if (PAuthABIPlatform != uint64_t(-1)) {
    OutStreamer.emitIntValue(ELF::GNU_PROPERTY_AARCH64_FEATURE_PAUTH, 4);
    OutStreamer.emitIntValue(8 * 2, 4); // data size
    OutStreamer.emitIntValue(PAuthABIPlatform, 8);
    OutStreamer.emitIntValue(PAuthABIVersion, 8);
  }

  OutStreamer.endSection(Nt);
  OutStreamer.switchSection(Cur);
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 126-150: Function AArch64TargetStreamer::emitInst
```cpp

void AArch64TargetStreamer::emitInst(uint32_t Inst) {
  char Buffer[4];

  // We can't just use EmitIntValue here, as that will swap the
  // endianness on big-endian systems (instructions are always
  // little-endian).
  for (char &C : Buffer) {
    C = uint8_t(Inst);
    Inst >>= 8;
  }

  getStreamer().emitBytes(StringRef(Buffer, 4));
}

MCTargetStreamer *
llvm::createAArch64ObjectTargetStreamer(MCStreamer &S,
                                        const MCSubtargetInfo &STI) {
  const Triple &TT = STI.getTargetTriple();
  if (TT.isOSBinFormatELF())
    return new AArch64TargetELFStreamer(S);
  if (TT.isOSBinFormatCOFF())
    return new AArch64TargetWinCOFFStreamer(S);
  return nullptr;
}
```
**EN:** This block implements AArch64TargetStreamer::emitInst, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64TargetStreamer::emitInst，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 151-174: Function llvm::createAArch64NullTargetStreamer
```cpp

MCTargetStreamer *llvm::createAArch64NullTargetStreamer(MCStreamer &S) {
  return new AArch64TargetStreamer(S);
}

void AArch64TargetStreamer::emitAttributesSubsection(
    StringRef VendorName, AArch64BuildAttributes::SubsectionOptional IsOptional,
    AArch64BuildAttributes::SubsectionType ParameterType) {

  // If exists, return.
  for (MCELFStreamer::AttributeSubSection &SubSection : AttributeSubSections) {
    if (VendorName == SubSection.VendorName) {
      activateAttributesSubsection(VendorName);
      return;
    }
  }
  // else, add the subsection
  MCELFStreamer::AttributeSubSection AttSubSection;
  AttSubSection.VendorName = VendorName;
  AttSubSection.IsOptional = IsOptional;
  AttSubSection.ParameterType = ParameterType;
  AttributeSubSections.push_back(AttSubSection);
  activateAttributesSubsection(VendorName);
}
```
**EN:** This block implements llvm::createAArch64NullTargetStreamer, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 llvm::createAArch64NullTargetStreamer，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 175-197: Function AArch64TargetStreamer::getActiveAttributesSubsection
```cpp

std::unique_ptr<MCELFStreamer::AttributeSubSection>
AArch64TargetStreamer::getActiveAttributesSubsection() {
  for (MCELFStreamer::AttributeSubSection &SubSection : AttributeSubSections) {
    if (SubSection.IsActive) {
      return std::make_unique<MCELFStreamer::AttributeSubSection>(SubSection);
    }
  }
  return nullptr;
}

std::unique_ptr<MCELFStreamer::AttributeSubSection>
AArch64TargetStreamer::getAttributesSubsectionByName(StringRef Name) {
  for (MCELFStreamer::AttributeSubSection &SubSection : AttributeSubSections) {
    if (Name == SubSection.VendorName) {
      return std::make_unique<MCELFStreamer::AttributeSubSection>(SubSection);
    }
  }
  return nullptr;
}

void AArch64TargetStreamer::emitAttribute(StringRef VendorName, unsigned Tag,
                                          unsigned Value, std::string String) {
```
**EN:** This block implements AArch64TargetStreamer::getActiveAttributesSubsection, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64TargetStreamer::getActiveAttributesSubsection，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 198-225: Core AArch64 backend logic
```cpp

  if (unsigned(-1) == Value && "" == String) {
    assert(0 && "Arguments error");
    return;
  }
  if (AttributeSubSections.size() == 0) {
    assert(0 &&
           "Can not add AArch64 build attribute: no AArch64 subsection exists");
    return;
  }

  for (MCELFStreamer::AttributeSubSection &SubSection : AttributeSubSections) {
    if (VendorName == SubSection.VendorName) {
      if (!SubSection.IsActive) {
        assert(0 &&
               "Can not add AArch64 build attribute: subsection is not active");
        return;
      }
      for (MCELFStreamer::AttributeItem &Item : SubSection.Content) {
        // Tag already exists
        if (Item.Tag == Tag) {
          Item.Type = unsigned(-1) != Value
                          ? MCELFStreamer::AttributeItem::NumericAttribute
                          : MCELFStreamer::AttributeItem::TextAttribute;
          Item.IntValue = unsigned(-1) != Value ? Value : unsigned(-1);
          Item.StringValue = unsigned(-1) != Value ? "" : String;
          return;
        }
```
**EN:** This block continues the file's main MC streaming and object emission logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的MC 流与目标文件输出主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 226-249: Core AArch64 backend logic
```cpp
      }
      if (unsigned(-1) != Value)
        SubSection.Content.push_back(MCELFStreamer::AttributeItem(
            MCELFStreamer::AttributeItem::NumericAttribute, Tag, Value, ""));
      if ("" != String)
        SubSection.Content.push_back(MCELFStreamer::AttributeItem(
            MCELFStreamer::AttributeItem::TextAttribute, Tag, unsigned(-1),
            String));
      return;
    }
  }
  assert(0 && "Can not add AArch64 build attribute: required subsection does "
              "not exist");
}

void AArch64TargetStreamer::activateAttributesSubsection(StringRef VendorName) {
  for (MCELFStreamer::AttributeSubSection &SubSection : AttributeSubSections) {
    if (VendorName == SubSection.VendorName) {
      SubSection.IsActive = true;
    } else {
      SubSection.IsActive = false;
    }
  }
}
```
**EN:** This block continues the file's main MC streaming and object emission logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的MC 流与目标文件输出主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64TargetStreamer.h, AArch64MCAsmInfo.h **CN:** 目标本地依赖：AArch64TargetStreamer.h, AArch64MCAsmInfo.h
- **EN:** Core LLVM interfaces: llvm/BinaryFormat/ELF.h, llvm/MC/ConstantPools.h, llvm/MC/MCContext.h, llvm/MC/MCELFStreamer.h, llvm/MC/MCSection.h, llvm/MC/MCSectionELF.h, llvm/MC/MCSubtargetInfo.h, llvm/Support/CommandLine.h **CN:** 核心 LLVM 接口：llvm/BinaryFormat/ELF.h, llvm/MC/ConstantPools.h, llvm/MC/MCContext.h, llvm/MC/MCELFStreamer.h, llvm/MC/MCSection.h, llvm/MC/MCSectionELF.h, llvm/MC/MCSubtargetInfo.h, llvm/Support/CommandLine.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for MC streaming and object emission. **CN:** 与周边负责MC 流与目标文件输出的 AArch64 后端组件紧密协作。
