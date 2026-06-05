# ARMELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMELFObjectWriter.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMELFObjectWriter` for the ARM backend, focusing on MC-layer target description and encoding support.
- 用途 (CN): 实现 ARM 后端中的 `ARMELFObjectWriter`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- ARMELFObjectWriter.cpp - ARM ELF Writer ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-22
```cpp
#include "MCTargetDesc/ARMFixupKinds.h"
#include "MCTargetDesc/ARMMCAsmInfo.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Object/ELF.h"
#include "llvm/Support/ErrorHandling.h"
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-24
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 26-26
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 28-29
```cpp
class ARMELFObjectWriter : public MCELFObjectTargetWriter {
  enum { DefaultEABIVersion = 0x05000000U };
```
- EN: Declares `ARMELFObjectWriter`, packaging target-specific state and APIs around `ARMELFObjectWriter`.
- CN: 这里声明 `ARMELFObjectWriter`，把与 `ARMELFObjectWriter` 相关的目标特定状态和 API 组织在一起。

### Lines 31-32
```cpp
public:
  ARMELFObjectWriter(uint8_t OSABI);
```
- EN: Declares `ARMELFObjectWriter`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMELFObjectWriter`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 34-34
```cpp
  ~ARMELFObjectWriter() override = default;
```
- EN: Declares `~ARMELFObjectWriter`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `~ARMELFObjectWriter`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-37
```cpp
  unsigned getRelocType(const MCFixup &, const MCValue &,
                        bool IsPCRel) const override;
```
- EN: Declares `getRelocType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getRelocType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 39-41
```cpp
  bool needsRelocateWithSymbol(const MCValue &Val,
                               unsigned Type) const override;
};
```
- EN: Declares `needsRelocateWithSymbol`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `needsRelocateWithSymbol`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 43-43
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 45-48
```cpp
ARMELFObjectWriter::ARMELFObjectWriter(uint8_t OSABI)
  : MCELFObjectTargetWriter(/*Is64Bit*/ false, OSABI,
                            ELF::EM_ARM,
                            /*HasRelocationAddend*/ false) {}
```
- EN: Implements `ARMELFObjectWriter::ARMELFObjectWriter`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMELFObjectWriter::ARMELFObjectWriter`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-57
```cpp
bool ARMELFObjectWriter::needsRelocateWithSymbol(const MCValue &V,
                                                 unsigned Type) const {
  // If the symbol is a thumb function the final relocation must set the lowest
  // bit. With a symbol that is done by just having the symbol have that bit
  // set, so we would lose the bit if we relocated with the section.
  // We could use the section but add the bit to the relocation value.
  if (Asm->isThumbFunc(V.getAddSym()))
    return true;
```
- EN: Implements `ARMELFObjectWriter::needsRelocateWithSymbol`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMELFObjectWriter::needsRelocateWithSymbol`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-64
```cpp
  // FIXME: This is extremely conservative. This really needs to use an
  // explicit list with a clear explanation for why each realocation needs to
  // point to the symbol, not to the section.
  switch (Type) {
  default:
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 66-70
```cpp
  case ELF::R_ARM_PREL31:
  case ELF::R_ARM_ABS32:
    return false;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 72-86
```cpp
// Need to examine the Fixup when determining whether to
// emit the relocation as an explicit symbol or as a section relative
// offset
unsigned ARMELFObjectWriter::getRelocType(const MCFixup &Fixup,
                                          const MCValue &Target,
                                          bool IsPCRel) const {
  auto Kind = Fixup.getKind();
  uint8_t Specifier = Target.getSpecifier();
  auto CheckFDPIC = [&](uint32_t Type) {
    if (getOSABI() != ELF::ELFOSABI_ARM_FDPIC)
      reportError(Fixup.getLoc(),
                  "relocation " +
                      object::getELFRelocationTypeName(ELF::EM_ARM, Type) +
                      " only supported in FDPIC mode");
    return Type;
```
- EN: Implements `ARMELFObjectWriter::getRelocType`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMELFObjectWriter::getRelocType`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-87
```cpp
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 89-103
```cpp
  switch (Specifier) {
  case ARM::S_GOTTPOFF:
  case ARM::S_GOTTPOFF_FDPIC:
  case ARM::S_TLSCALL:
  case ARM::S_TLSDESC:
  case ARM::S_TLSGD:
  case ARM::S_TLSGD_FDPIC:
  case ARM::S_TLSLDM:
  case ARM::S_TLSLDM_FDPIC:
  case ARM::S_TLSLDO:
  case ARM::S_TPOFF:
    if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
      static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
    break;
  default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 104-105
```cpp
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 107-121
```cpp
  if (IsPCRel) {
    switch (Fixup.getKind()) {
    default:
      reportError(Fixup.getLoc(), "unsupported relocation type");
      return ELF::R_ARM_NONE;
    case FK_Data_4:
      switch (Specifier) {
      default:
        reportError(Fixup.getLoc(),
                    "invalid fixup for 4-byte pc-relative data relocation");
        return ELF::R_ARM_NONE;
      case ARM::S_None: {
        if (const auto *SA = Target.getAddSym()) {
          // For GNU AS compatibility expressions such as
          // _GLOBAL_OFFSET_TABLE_ - label emit a R_ARM_BASE_PREL relocation.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 122-136
```cpp
          if (SA->getName() == "_GLOBAL_OFFSET_TABLE_")
            return ELF::R_ARM_BASE_PREL;
        }
        return ELF::R_ARM_REL32;
      }
      case ARM::S_GOTTPOFF:
        return ELF::R_ARM_TLS_IE32;
      case ARM::S_GOT_PREL:
        return ELF::R_ARM_GOT_PREL;
      case ARM::S_PREL31:
        return ELF::R_ARM_PREL31;
      }
    case ARM::fixup_arm_blx:
    case ARM::fixup_arm_uncondbl:
      switch (Specifier) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 137-151
```cpp
      case ARM::S_PLT:
        return ELF::R_ARM_CALL;
      case ARM::S_TLSCALL:
        return ELF::R_ARM_TLS_CALL;
      default:
        return ELF::R_ARM_CALL;
      }
    case ARM::fixup_arm_condbl:
    case ARM::fixup_arm_condbranch:
    case ARM::fixup_arm_uncondbranch:
      return ELF::R_ARM_JUMP24;
    case ARM::fixup_t2_condbranch:
      return ELF::R_ARM_THM_JUMP19;
    case ARM::fixup_t2_uncondbranch:
      return ELF::R_ARM_THM_JUMP24;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 152-166
```cpp
    case ARM::fixup_arm_movt_hi16:
      return ELF::R_ARM_MOVT_PREL;
    case ARM::fixup_arm_movw_lo16:
      return ELF::R_ARM_MOVW_PREL_NC;
    case ARM::fixup_t2_movt_hi16:
      return ELF::R_ARM_THM_MOVT_PREL;
    case ARM::fixup_t2_movw_lo16:
      return ELF::R_ARM_THM_MOVW_PREL_NC;
    case ARM::fixup_arm_thumb_upper_8_15:
      return ELF::R_ARM_THM_ALU_ABS_G3;
    case ARM::fixup_arm_thumb_upper_0_7:
      return ELF::R_ARM_THM_ALU_ABS_G2_NC;
    case ARM::fixup_arm_thumb_lower_8_15:
      return ELF::R_ARM_THM_ALU_ABS_G1_NC;
    case ARM::fixup_arm_thumb_lower_0_7:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 167-181
```cpp
      return ELF::R_ARM_THM_ALU_ABS_G0_NC;
    case ARM::fixup_arm_thumb_br:
      return ELF::R_ARM_THM_JUMP11;
    case ARM::fixup_arm_thumb_bcc:
      return ELF::R_ARM_THM_JUMP8;
    case ARM::fixup_arm_thumb_bl:
    case ARM::fixup_arm_thumb_blx:
      switch (Specifier) {
      case ARM::S_TLSCALL:
        return ELF::R_ARM_THM_TLS_CALL;
      default:
        return ELF::R_ARM_THM_CALL;
      }
    case ARM::fixup_arm_ldst_pcrel_12:
      return ELF::R_ARM_LDR_PC_G0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 182-196
```cpp
    case ARM::fixup_arm_pcrel_10_unscaled:
      return ELF::R_ARM_LDRS_PC_G0;
    case ARM::fixup_t2_ldst_pcrel_12:
      return ELF::R_ARM_THM_PC12;
    case ARM::fixup_arm_adr_pcrel_12:
      return ELF::R_ARM_ALU_PC_G0;
    case ARM::fixup_thumb_adr_pcrel_10:
      return ELF::R_ARM_THM_PC8;
    case ARM::fixup_t2_adr_pcrel_12:
      return ELF::R_ARM_THM_ALU_PREL_11_0;
    case ARM::fixup_bf_target:
      return ELF::R_ARM_THM_BF16;
    case ARM::fixup_bfc_target:
      return ELF::R_ARM_THM_BF12;
    case ARM::fixup_bfl_target:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 197-211
```cpp
      return ELF::R_ARM_THM_BF18;
    }
  }
  switch (Kind) {
  default:
    reportError(Fixup.getLoc(), "unsupported relocation type");
    return ELF::R_ARM_NONE;
  case FK_Data_1:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for 1-byte data relocation");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_ABS8;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 212-226
```cpp
  case FK_Data_2:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for 2-byte data relocation");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_ABS16;
    }
  case FK_Data_4:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for 4-byte data relocation");
      return ELF::R_ARM_NONE;
    case ARM::S_ARM_NONE:
      return ELF::R_ARM_NONE;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-241
```cpp
    case ARM::S_GOT:
      return ELF::R_ARM_GOT_BREL;
    case ARM::S_TLSGD:
      return ELF::R_ARM_TLS_GD32;
    case ARM::S_TPOFF:
      return ELF::R_ARM_TLS_LE32;
    case ARM::S_GOTTPOFF:
      return ELF::R_ARM_TLS_IE32;
    case ARM::S_None:
      return ELF::R_ARM_ABS32;
    case ARM::S_GOTOFF:
      return ELF::R_ARM_GOTOFF32;
    case ARM::S_GOT_PREL:
      return ELF::R_ARM_GOT_PREL;
    case ARM::S_TARGET1:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 242-256
```cpp
      return ELF::R_ARM_TARGET1;
    case ARM::S_TARGET2:
      return ELF::R_ARM_TARGET2;
    case ARM::S_PREL31:
      return ELF::R_ARM_PREL31;
    case ARM::S_SBREL:
      return ELF::R_ARM_SBREL32;
    case ARM::S_TLSLDO:
      return ELF::R_ARM_TLS_LDO32;
    case ARM::S_TLSCALL:
      return ELF::R_ARM_TLS_CALL;
    case ARM::S_TLSDESC:
      return ELF::R_ARM_TLS_GOTDESC;
    case ARM::S_TLSLDM:
      return ELF::R_ARM_TLS_LDM32;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 257-271
```cpp
    case ARM::S_TLSDESCSEQ:
      return ELF::R_ARM_TLS_DESCSEQ;
    case ARM::S_FUNCDESC:
      return CheckFDPIC(ELF::R_ARM_FUNCDESC);
    case ARM::S_GOTFUNCDESC:
      return CheckFDPIC(ELF::R_ARM_GOTFUNCDESC);
    case ARM::S_GOTOFFFUNCDESC:
      return CheckFDPIC(ELF::R_ARM_GOTOFFFUNCDESC);
    case ARM::S_TLSGD_FDPIC:
      return CheckFDPIC(ELF::R_ARM_TLS_GD32_FDPIC);
    case ARM::S_TLSLDM_FDPIC:
      return CheckFDPIC(ELF::R_ARM_TLS_LDM32_FDPIC);
    case ARM::S_GOTTPOFF_FDPIC:
      return CheckFDPIC(ELF::R_ARM_TLS_IE32_FDPIC);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 272-286
```cpp
  case ARM::fixup_arm_condbranch:
  case ARM::fixup_arm_uncondbranch:
    return ELF::R_ARM_JUMP24;
  case ARM::fixup_arm_movt_hi16:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for ARM MOVT instruction");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_MOVT_ABS;
    case ARM::S_SBREL:
      return ELF::R_ARM_MOVT_BREL;
    }
  case ARM::fixup_arm_movw_lo16:
    switch (Specifier) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 287-301
```cpp
    default:
      reportError(Fixup.getLoc(), "invalid fixup for ARM MOVW instruction");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_MOVW_ABS_NC;
    case ARM::S_SBREL:
      return ELF::R_ARM_MOVW_BREL_NC;
    }
  case ARM::fixup_t2_movt_hi16:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for Thumb MOVT instruction");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_THM_MOVT_ABS;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 302-314
```cpp
    case ARM::S_SBREL:
      return ELF::R_ARM_THM_MOVT_BREL;
    }
  case ARM::fixup_t2_movw_lo16:
    switch (Specifier) {
    default:
      reportError(Fixup.getLoc(), "invalid fixup for Thumb MOVW instruction");
      return ELF::R_ARM_NONE;
    case ARM::S_None:
      return ELF::R_ARM_THM_MOVW_ABS_NC;
    case ARM::S_SBREL:
      return ELF::R_ARM_THM_MOVW_BREL_NC;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 316-325
```cpp
  case ARM::fixup_arm_thumb_upper_8_15:
    return ELF::R_ARM_THM_ALU_ABS_G3;
  case ARM::fixup_arm_thumb_upper_0_7:
    return ELF::R_ARM_THM_ALU_ABS_G2_NC;
  case ARM::fixup_arm_thumb_lower_8_15:
    return ELF::R_ARM_THM_ALU_ABS_G1_NC;
  case ARM::fixup_arm_thumb_lower_0_7:
    return ELF::R_ARM_THM_ALU_ABS_G0_NC;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 327-330
```cpp
std::unique_ptr<MCObjectTargetWriter>
llvm::createARMELFObjectWriter(uint8_t OSABI) {
  return std::make_unique<ARMELFObjectWriter>(OSABI);
}
```
- EN: Implements `llvm::createARMELFObjectWriter`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createARMELFObjectWriter`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCAsmInfo.h`, `MCTargetDesc/ARMMCTargetDesc.h`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMFixupKinds.h`, `MCTargetDesc/ARMMCAsmInfo.h`, `MCTargetDesc/ARMMCTargetDesc.h`。
- EN: LLVM infrastructure headers: `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCValue.h` ... (+2 more).
  - CN: LLVM 基础设施头文件：`llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCValue.h` ... (+2 more)。
- EN: Standard/system headers: `cstdint`.
  - CN: 标准库/系统头文件：`cstdint`。
