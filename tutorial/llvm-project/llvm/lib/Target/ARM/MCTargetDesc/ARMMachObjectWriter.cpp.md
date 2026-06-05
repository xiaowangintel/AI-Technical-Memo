# ARMMachObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMMachObjectWriter.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMMachObjectWriter` for the ARM backend, focusing on MC-layer target description and encoding support.
- 用途 (CN): 实现 ARM 后端中的 `ARMMachObjectWriter`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ARMMachObjectWriter.cpp - ARM Mach Object Writer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "MCTargetDesc/ARMFixupKinds.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/MachO.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCMachObjectWriter.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCSection.h"
#include "llvm/MC/MCSymbolMachO.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/ErrorHandling.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 25-25
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 27-39
```cpp
namespace {
class ARMMachObjectWriter : public MCMachObjectTargetWriter {
  void recordARMScatteredRelocation(MachObjectWriter *Writer,
                                    const MCAssembler &Asm,
                                    const MCFragment *Fragment,
                                    const MCFixup &Fixup, MCValue Target,
                                    unsigned Type, unsigned Log2Size,
                                    uint64_t &FixedValue);
  void recordARMScatteredHalfRelocation(MachObjectWriter *Writer,
                                        const MCAssembler &Asm,
                                        const MCFragment *Fragment,
                                        const MCFixup &Fixup, MCValue Target,
                                        uint64_t &FixedValue);
```
- EN: Declares `ARMMachObjectWriter`, packaging target-specific state and APIs around `ARMMachObjectWriter`.
- CN: 这里声明 `ARMMachObjectWriter`，把与 `ARMMachObjectWriter` 相关的目标特定状态和 API 组织在一起。

### Lines 41-44
```cpp
  bool requiresExternRelocation(MachObjectWriter *Writer,
                                const MCAssembler &Asm,
                                const MCFragment &Fragment, unsigned RelocType,
                                const MCSymbol &S, uint64_t FixedValue);
```
- EN: Declares `requiresExternRelocation`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `requiresExternRelocation`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-48
```cpp
public:
  ARMMachObjectWriter(bool Is64Bit, uint32_t CPUType, uint32_t CPUSubtype)
      : MCMachObjectTargetWriter(Is64Bit, CPUType, CPUSubtype) {}
```
- EN: Implements `ARMMachObjectWriter`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMachObjectWriter`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-54
```cpp
  void recordRelocation(MachObjectWriter *Writer, MCAssembler &Asm,
                        const MCFragment *Fragment, const MCFixup &Fixup,
                        MCValue Target, uint64_t &FixedValue) override;
};
}
```
- EN: Declares `recordRelocation`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `recordRelocation`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-59
```cpp
static bool getARMFixupKindMachOInfo(unsigned Kind, unsigned &RelocType,
                              unsigned &Log2Size) {
  RelocType = unsigned(MachO::ARM_RELOC_VANILLA);
  Log2Size = ~0U;
```
- EN: Implements `getARMFixupKindMachOInfo`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getARMFixupKindMachOInfo`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 61-63
```cpp
  switch (Kind) {
  default:
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 65-76
```cpp
  case FK_Data_1:
    Log2Size = llvm::Log2_32(1);
    return true;
  case FK_Data_2:
    Log2Size = llvm::Log2_32(2);
    return true;
  case FK_Data_4:
    Log2Size = llvm::Log2_32(4);
    return true;
  case FK_Data_8:
    Log2Size = llvm::Log2_32(8);
    return false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 78-84
```cpp
    // These fixups are expected to always be resolvable at assembly time and
    // have no relocations supported.
  case ARM::fixup_arm_ldst_pcrel_12:
  case ARM::fixup_arm_pcrel_10:
  case ARM::fixup_arm_adr_pcrel_12:
  case ARM::fixup_arm_thumb_br:
    return false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 86-95
```cpp
    // Handle 24-bit branch kinds.
  case ARM::fixup_arm_condbranch:
  case ARM::fixup_arm_uncondbranch:
  case ARM::fixup_arm_uncondbl:
  case ARM::fixup_arm_condbl:
  case ARM::fixup_arm_blx:
    RelocType = unsigned(MachO::ARM_RELOC_BR24);
    // Report as 'long', even though that is not quite accurate.
    Log2Size = llvm::Log2_32(4);
    return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 97-102
```cpp
  case ARM::fixup_t2_uncondbranch:
  case ARM::fixup_arm_thumb_bl:
  case ARM::fixup_arm_thumb_blx:
    RelocType = unsigned(MachO::ARM_THUMB_RELOC_BR22);
    Log2Size = llvm::Log2_32(4);
    return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-118
```cpp
  // For movw/movt r_type relocations they always have a pair following them and
  // the r_length bits are used differently.  The encoding of the r_length is as
  // follows:
  //   low bit of r_length:
  //      0 - :lower16: for movw instructions
  //      1 - :upper16: for movt instructions
  //   high bit of r_length:
  //      0 - arm instructions
  //      1 - thumb instructions
  case ARM::fixup_arm_movt_hi16:
    RelocType = unsigned(MachO::ARM_RELOC_HALF);
    Log2Size = 1;
    return true;
  case ARM::fixup_t2_movt_hi16:
    RelocType = unsigned(MachO::ARM_RELOC_HALF);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 119-120
```cpp
    Log2Size = 3;
    return true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 122-131
```cpp
  case ARM::fixup_arm_movw_lo16:
    RelocType = unsigned(MachO::ARM_RELOC_HALF);
    Log2Size = 0;
    return true;
  case ARM::fixup_t2_movw_lo16:
    RelocType = unsigned(MachO::ARM_RELOC_HALF);
    Log2Size = 2;
    return true;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 133-137
```cpp
void ARMMachObjectWriter::recordARMScatteredHalfRelocation(
    MachObjectWriter *Writer, const MCAssembler &Asm,
    const MCFragment *Fragment, const MCFixup &Fixup, MCValue Target,
    uint64_t &FixedValue) {
  uint32_t FixupOffset = Asm.getFragmentOffset(*Fragment) + Fixup.getOffset();
```
- EN: Implements `ARMMachObjectWriter::recordARMScatteredHalfRelocation`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMachObjectWriter::recordARMScatteredHalfRelocation`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-144
```cpp
  if (FixupOffset & 0xff000000) {
    reportError(Fixup.getLoc(), "can not encode offset '0x" +
                                    utohexstr(FixupOffset) +
                                    "' in resulting scattered relocation.");
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 146-147
```cpp
  unsigned IsPCRel = Fixup.isPCRel();
  unsigned Type = MachO::ARM_RELOC_HALF;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 149-150
```cpp
  // See <reloc.h>.
  const MCSymbol *A = Target.getAddSym();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 152-157
```cpp
  if (!A->getFragment()) {
    reportError(Fixup.getLoc(),
                "symbol '" + A->getName() +
                    "' can not be undefined in a subtraction expression");
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 159-162
```cpp
  uint32_t Value = Writer->getSymbolAddress(*A);
  uint32_t Value2 = 0;
  uint64_t SecAddr = Writer->getSectionAddress(A->getFragment()->getParent());
  FixedValue += SecAddr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-170
```cpp
  if (const MCSymbol *SB = Target.getSubSym()) {
    if (!SB->getFragment()) {
      reportError(Fixup.getLoc(),
                  "symbol '" + SB->getName() +
                      "' can not be undefined in a subtraction expression");
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 172-176
```cpp
    // Select the appropriate difference relocation type.
    Type = MachO::ARM_RELOC_HALF_SECTDIFF;
    Value2 = Writer->getSymbolAddress(*SB);
    FixedValue -= Writer->getSectionAddress(SB->getFragment()->getParent());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 178-192
```cpp
  // Relocations are written out in reverse order, so the PAIR comes first.
  // ARM_RELOC_HALF and ARM_RELOC_HALF_SECTDIFF abuse the r_length field:
  //
  // For these two r_type relocations they always have a pair following them and
  // the r_length bits are used differently.  The encoding of the r_length is as
  // follows:
  //   low bit of r_length:
  //      0 - :lower16: for movw instructions
  //      1 - :upper16: for movt instructions
  //   high bit of r_length:
  //      0 - arm instructions
  //      1 - thumb instructions
  // the other half of the relocated expression is in the following pair
  // relocation entry in the low 16 bits of r_address field.
  unsigned ThumbBit = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 193-207
```cpp
  unsigned MovtBit = 0;
  switch (Fixup.getKind()) {
  default: break;
  case ARM::fixup_arm_movt_hi16:
    MovtBit = 1;
    // The thumb bit shouldn't be set in the 'other-half' bit of the
    // relocation, but it will be set in FixedValue if the base symbol
    // is a thumb function. Clear it out here.
    if (Asm.isThumbFunc(A))
      FixedValue &= 0xfffffffe;
    break;
  case ARM::fixup_t2_movt_hi16:
    if (Asm.isThumbFunc(A))
      FixedValue &= 0xfffffffe;
    MovtBit = 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 208-212
```cpp
    [[fallthrough]];
  case ARM::fixup_t2_movw_lo16:
    ThumbBit = 1;
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 214-216
```cpp
  if (Type == MachO::ARM_RELOC_HALF_SECTDIFF) {
    uint32_t OtherHalf = MovtBit
      ? (FixedValue & 0xffff) : ((FixedValue & 0xffff0000) >> 16);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 218-227
```cpp
    MachO::any_relocation_info MRE;
    MRE.r_word0 = ((OtherHalf             <<  0) |
                   (MachO::ARM_RELOC_PAIR << 24) |
                   (MovtBit               << 28) |
                   (ThumbBit              << 29) |
                   (IsPCRel               << 30) |
                   MachO::R_SCATTERED);
    MRE.r_word1 = Value2;
    Writer->addRelocation(nullptr, Fragment->getParent(), MRE);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-238
```cpp
  MachO::any_relocation_info MRE;
  MRE.r_word0 = ((FixupOffset <<  0) |
                 (Type        << 24) |
                 (MovtBit     << 28) |
                 (ThumbBit    << 29) |
                 (IsPCRel     << 30) |
                 MachO::R_SCATTERED);
  MRE.r_word1 = Value;
  Writer->addRelocation(nullptr, Fragment->getParent(), MRE);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 240-244
```cpp
void ARMMachObjectWriter::recordARMScatteredRelocation(
    MachObjectWriter *Writer, const MCAssembler &Asm,
    const MCFragment *Fragment, const MCFixup &Fixup, MCValue Target,
    unsigned Type, unsigned Log2Size, uint64_t &FixedValue) {
  uint32_t FixupOffset = Asm.getFragmentOffset(*Fragment) + Fixup.getOffset();
```
- EN: Implements `ARMMachObjectWriter::recordARMScatteredRelocation`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMMachObjectWriter::recordARMScatteredRelocation`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-251
```cpp
  if (FixupOffset & 0xff000000) {
    reportError(Fixup.getLoc(), "can not encode offset '0x" +
                                    utohexstr(FixupOffset) +
                                    "' in resulting scattered relocation.");
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 253-253
```cpp
  unsigned IsPCRel = Fixup.isPCRel();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 255-256
```cpp
  // See <reloc.h>.
  const MCSymbol *A = Target.getAddSym();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 258-263
```cpp
  if (!A->getFragment()) {
    reportError(Fixup.getLoc(),
                "symbol '" + A->getName() +
                    "' can not be undefined in a subtraction expression");
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 265-268
```cpp
  uint32_t Value = Writer->getSymbolAddress(*A);
  uint64_t SecAddr = Writer->getSectionAddress(A->getFragment()->getParent());
  FixedValue += SecAddr;
  uint32_t Value2 = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 270-271
```cpp
  if (const MCSymbol *SB = Target.getSubSym()) {
    assert(Type == MachO::ARM_RELOC_VANILLA && "invalid reloc for 2 symbols");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 273-278
```cpp
    if (!SB->getFragment()) {
      reportError(Fixup.getLoc(),
                  "symbol '" + SB->getName() +
                      "' can not be undefined in a subtraction expression");
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 280-284
```cpp
    // Select the appropriate difference relocation type.
    Type = MachO::ARM_RELOC_SECTDIFF;
    Value2 = Writer->getSymbolAddress(*SB);
    FixedValue -= Writer->getSectionAddress(SB->getFragment()->getParent());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 286-297
```cpp
  // Relocations are written out in reverse order, so the PAIR comes first.
  if (Type == MachO::ARM_RELOC_SECTDIFF ||
      Type == MachO::ARM_RELOC_LOCAL_SECTDIFF) {
    MachO::any_relocation_info MRE;
    MRE.r_word0 = ((0                     <<  0) |
                   (MachO::ARM_RELOC_PAIR << 24) |
                   (Log2Size              << 28) |
                   (IsPCRel               << 30) |
                   MachO::R_SCATTERED);
    MRE.r_word1 = Value2;
    Writer->addRelocation(nullptr, Fragment->getParent(), MRE);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 299-307
```cpp
  MachO::any_relocation_info MRE;
  MRE.r_word0 = ((FixupOffset <<  0) |
                 (Type        << 24) |
                 (Log2Size    << 28) |
                 (IsPCRel     << 30) |
                 MachO::R_SCATTERED);
  MRE.r_word1 = Value;
  Writer->addRelocation(nullptr, Fragment->getParent(), MRE);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 309-323
```cpp
bool ARMMachObjectWriter::requiresExternRelocation(MachObjectWriter *Writer,
                                                   const MCAssembler &Asm,
                                                   const MCFragment &Fragment,
                                                   unsigned RelocType,
                                                   const MCSymbol &S,
                                                   uint64_t FixedValue) {
  // Most cases can be identified purely from the symbol.
  if (Writer->doesSymbolRequireExternRelocation(S))
    return true;
  int64_t Value = (int64_t)FixedValue;  // The displacement is signed.
  int64_t Range;
  switch (RelocType) {
  default:
    return false;
  case MachO::ARM_RELOC_BR24:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 324-330
```cpp
    // An ARM call might be to a Thumb function, in which case the offset may
    // not be encodable in the instruction and we must use an external
    // relocation that explicitly mentions the function. Not a problem if it's
    // to a temporary "Lwhatever" symbol though, and in fact trying to use an
    // external relocation there causes more issues.
    if (!S.isTemporary())
       return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 332-346
```cpp
    // PC pre-adjustment of 8 for these instructions.
    Value -= 8;
    // ARM BL/BLX has a 25-bit offset.
    Range = 0x1ffffff;
    break;
  case MachO::ARM_THUMB_RELOC_BR22:
    // PC pre-adjustment of 4 for these instructions.
    Value -= 4;
    // Thumb BL/BLX has a 24-bit offset.
    Range = 0xffffff;
  }
  // BL/BLX also use external relocations when an internal relocation
  // would result in the target being out of range. This gives the linker
  // enough information to generate a branch island.
  Value += Writer->getSectionAddress(&S.getSection());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 347-353
```cpp
  Value -= Writer->getSectionAddress(Fragment.getParent());
  // If the resultant value would be out of range for an internal relocation,
  // use an external instead.
  if (Value > Range || Value < -(Range + 1))
    return true;
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 355-369
```cpp
void ARMMachObjectWriter::recordRelocation(MachObjectWriter *Writer,
                                           MCAssembler &Asm,
                                           const MCFragment *Fragment,
                                           const MCFixup &Fixup, MCValue Target,
                                           uint64_t &FixedValue) {
  unsigned IsPCRel = Fixup.isPCRel();
  unsigned Log2Size;
  unsigned RelocType = MachO::ARM_RELOC_VANILLA;
  if (!getARMFixupKindMachOInfo(Fixup.getKind(), RelocType, Log2Size)) {
    // If we failed to get fixup kind info, it's because there's no legal
    // relocation type for the fixup kind. This happens when it's a fixup that's
    // expected to always be resolvable at assembly time and not have any
    // relocations needed.
    reportError(Fixup.getLoc(), "unsupported relocation type");
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 370-370
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 372-381
```cpp
  // If this is a difference or a defined symbol plus an offset, then we need a
  // scattered relocation entry.  Differences always require scattered
  // relocations.
  if (Target.getSubSym()) {
    if (RelocType == MachO::ARM_RELOC_HALF)
      return recordARMScatteredHalfRelocation(Writer, Asm, Fragment, Fixup,
                                              Target, FixedValue);
    return recordARMScatteredRelocation(Writer, Asm, Fragment, Fixup, Target,
                                        RelocType, Log2Size, FixedValue);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 383-384
```cpp
  // Get the symbol data, if any.
  const MCSymbol *A = Target.getAddSym();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 386-397
```cpp
  // FIXME: For other platforms, we need to use scattered relocations for
  // internal relocations with offsets.  If this is an internal relocation with
  // an offset, it also needs a scattered relocation entry.
  //
  // Is this right for ARM?
  uint32_t Offset = Target.getConstant();
  if (IsPCRel && RelocType == MachO::ARM_RELOC_VANILLA)
    Offset += 1 << Log2Size;
  if (Offset && A && !Writer->doesSymbolRequireExternRelocation(*A) &&
      RelocType != MachO::ARM_RELOC_HALF)
    return recordARMScatteredRelocation(Writer, Asm, Fragment, Fixup, Target,
                                        RelocType, Log2Size, FixedValue);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 399-403
```cpp
  // See <reloc.h>.
  uint32_t FixupOffset = Asm.getFragmentOffset(*Fragment) + Fixup.getOffset();
  unsigned Index = 0;
  unsigned Type = 0;
  const MCSymbol *RelSymbol = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 405-419
```cpp
  if (!A) { // constant
    // FIXME! This is Target.isAbsolute() case as we check SymB above. We check
    // !A to ensure that null pointer isn't dereferenced and suppress static
    // analyzer warnings.
    report_fatal_error("FIXME: relocations to absolute targets "
                       "not yet implemented");
  } else {
    // Resolve constant variables.
    if (A->isVariable()) {
      MCValue Val;
      bool Relocatable =
          A->getVariableValue()->evaluateAsRelocatable(Val, &Asm);
      int64_t Res = Val.getConstant();
      bool isAbs = Val.isAbsolute();
      if (Relocatable && Val.getAddSym() && Val.getSubSym()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 420-428
```cpp
        Res += Writer->getSymbolAddress(*Val.getAddSym()) -
               Writer->getSymbolAddress(*Val.getSubSym());
        isAbs = true;
      }
      if (isAbs) {
        FixedValue = Res;
        return;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 430-433
```cpp
    // Check whether we need an external or internal relocation.
    if (requiresExternRelocation(Writer, Asm, *Fragment, RelocType, *A,
                                 FixedValue)) {
      RelSymbol = A;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 435-447
```cpp
      // For external relocations, make sure to offset the fixup value to
      // compensate for the addend of the symbol address, if it was
      // undefined. This occurs with weak definitions, for example.
      if (!A->isUndefined())
        FixedValue -= Asm.getSymbolOffset(*A);
    } else {
      // The index is the section ordinal (1-based).
      const MCSection &Sec = A->getSection();
      Index = Sec.getOrdinal() + 1;
      FixedValue += Writer->getSectionAddress(&Sec);
    }
    if (IsPCRel)
      FixedValue -= Writer->getSectionAddress(Fragment->getParent());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 449-451
```cpp
    // The type is determined by the fixup kind.
    Type = RelocType;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 453-457
```cpp
  // struct relocation_info (8 bytes)
  MachO::any_relocation_info MRE;
  MRE.r_word0 = FixupOffset;
  MRE.r_word1 =
      (Index << 0) | (IsPCRel << 24) | (Log2Size << 25) | (Type << 28);
```
- EN: Declares `relocation_info`, packaging target-specific state and APIs around `ARMMachObjectWriter`.
- CN: 这里声明 `relocation_info`，把与 `ARMMachObjectWriter` 相关的目标特定状态和 API 组织在一起。

### Lines 459-473
```cpp
  // Even when it's not a scattered relocation, movw/movt always uses
  // a PAIR relocation.
  if (Type == MachO::ARM_RELOC_HALF) {
    // The entire addend is needed to correctly apply a relocation. One half is
    // extracted from the instruction itself, the other comes from this
    // PAIR. I.e. it's correct that we insert the high bits of the addend in the
    // MOVW case here.  relocation entries.
    uint32_t Value = 0;
    switch (Fixup.getKind()) {
    default: break;
    case ARM::fixup_arm_movw_lo16:
    case ARM::fixup_t2_movw_lo16:
      Value = (FixedValue >> 16) & 0xffff;
      break;
    case ARM::fixup_arm_movt_hi16:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 474-482
```cpp
    case ARM::fixup_t2_movt_hi16:
      Value = FixedValue & 0xffff;
      break;
    }
    MachO::any_relocation_info MREPair;
    MREPair.r_word0 = Value;
    MREPair.r_word1 = ((0xffffff              <<  0) |
                       (Log2Size              << 25) |
                       (MachO::ARM_RELOC_PAIR << 28));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 484-485
```cpp
    Writer->addRelocation(nullptr, Fragment->getParent(), MREPair);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-488
```cpp
  Writer->addRelocation(RelSymbol, Fragment->getParent(), MRE);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 490-494
```cpp
std::unique_ptr<MCObjectTargetWriter>
llvm::createARMMachObjectWriter(bool Is64Bit, uint32_t CPUType,
                                uint32_t CPUSubtype) {
  return std::make_unique<ARMMachObjectWriter>(Is64Bit, CPUType, CPUSubtype);
}
```
- EN: Implements `llvm::createARMMachObjectWriter`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMMachObjectWriter`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 496-510
```cpp
namespace {
class ARMTargetMachOStreamer : public ARMTargetStreamer {
public:
  ARMTargetMachOStreamer(MCStreamer &S) : ARMTargetStreamer(S) {}
  MCObjectStreamer &getStreamer() {
    return static_cast<MCObjectStreamer &>(Streamer);
  }
  void emitThumbFunc(MCSymbol *Symbol) override {
    // Remember that the function is a thumb function. Fixup and relocation
    // values will need adjusted.
    getStreamer().getAssembler().setIsThumbFunc(Symbol);
    static_cast<MCSymbolMachO *>(Symbol)->setThumbFunc();
  }
};
} // namespace
```
- EN: Declares `ARMTargetMachOStreamer`, packaging target-specific state and APIs around `ARMMachObjectWriter`.
- CN: 这里声明 `ARMTargetMachOStreamer`，把与 `ARMMachObjectWriter` 相关的目标特定状态和 API 组织在一起。

### Lines 512-514
```cpp
MCTargetStreamer *llvm::createARMObjectTargetMachOStreamer(MCStreamer &S) {
  return new ARMTargetMachOStreamer(S);
}
```
- EN: Implements `llvm::createARMObjectTargetMachOStreamer`, a query/helper routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMObjectTargetMachOStreamer`，它是一个围绕MC streamer 交互展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCTargetDesc.h`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCTargetDesc.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/MachO.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCMachObjectWriter.h` ... (+5 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringExtras.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/MachO.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCMachObjectWriter.h` ... (+5 more)。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
