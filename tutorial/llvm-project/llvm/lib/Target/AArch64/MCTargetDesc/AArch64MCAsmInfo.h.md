# AArch64MCAsmInfo.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64MCAsmInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file contains the declaration of the AArch64MCAsmInfo class. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Documented code section
```cpp
//=====-- AArch64MCAsmInfo.h - AArch64 asm properties ---------*- C++ -*--====//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the declaration of the AArch64MCAsmInfo class.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64MCASMINFO_H
#define LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64MCASMINFO_H

#include "Utils/AArch64BaseInfo.h"
#include "llvm/MC/MCAsmInfoCOFF.h"
#include "llvm/MC/MCAsmInfoDarwin.h"
#include "llvm/MC/MCAsmInfoELF.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/Support/Casting.h"

namespace llvm {
class MCStreamer;
class MCValue;
class Triple;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 27-53: Struct AArch64MCAsmInfoDarwin
```cpp

struct AArch64MCAsmInfoDarwin : public MCAsmInfoDarwin {
  explicit AArch64MCAsmInfoDarwin(bool IsILP32, const MCTargetOptions &Options);
  const MCExpr *
  getExprForPersonalitySymbol(const MCSymbol *Sym, unsigned Encoding,
                              MCStreamer &Streamer) const override;
  void printSpecifierExpr(raw_ostream &OS,
                          const MCSpecifierExpr &Expr) const override;
  bool evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr, MCValue &Res,
                                 const MCAssembler *Asm) const override;
};

struct AArch64MCAsmInfoELF : public MCAsmInfoELF {
  explicit AArch64MCAsmInfoELF(const Triple &T, const MCTargetOptions &Options);
  void printSpecifierExpr(raw_ostream &OS,
                          const MCSpecifierExpr &Expr) const override;
  bool evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr, MCValue &Res,
                                 const MCAssembler *Asm) const override;
};

struct AArch64MCAsmInfoMicrosoftCOFF : public MCAsmInfoMicrosoft {
  explicit AArch64MCAsmInfoMicrosoftCOFF(const MCTargetOptions &Options);
  void printSpecifierExpr(raw_ostream &OS,
                          const MCSpecifierExpr &Expr) const override;
  bool evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr, MCValue &Res,
                                 const MCAssembler *Asm) const override;
};
```
**EN:** This block defines AArch64MCAsmInfoDarwin, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64MCAsmInfoDarwin，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 54-81: Struct AArch64MCAsmInfoGNUCOFF
```cpp

struct AArch64MCAsmInfoGNUCOFF : public MCAsmInfoGNUCOFF {
  explicit AArch64MCAsmInfoGNUCOFF(const MCTargetOptions &Options);
  void printSpecifierExpr(raw_ostream &OS,
                          const MCSpecifierExpr &Expr) const override;
  bool evaluateAsRelocatableImpl(const MCSpecifierExpr &Expr, MCValue &Res,
                                 const MCAssembler *Asm) const override;
};

namespace AArch64 {
using Specifier = uint16_t;

enum {
  // clang-format off
  S_None         = 0,
  // Symbol locations specifying (roughly speaking) what calculation should be
  // performed to construct the final address for the relocated
  // symbol. E.g. direct, via the GOT, ...
  S_ABS          = 0x001,
  S_SABS         = 0x002,
  S_PREL         = 0x003,
  S_GOT          = 0x004,
  S_DTPREL       = 0x005,
  S_GOTTPREL     = 0x006,
  S_TPREL        = 0x007,
  S_TLSDESC      = 0x008,
  S_SECREL       = 0x009,
  S_AUTH         = 0x00a,
```
**EN:** This block defines AArch64MCAsmInfoGNUCOFF, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 AArch64MCAsmInfoGNUCOFF，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 82-105: Core AArch64 backend logic
```cpp
  S_AUTHADDR     = 0x00b,
  S_GOT_AUTH     = 0x00c,
  S_TLSDESC_AUTH = 0x00d,
  S_SymLocBits   = 0x00f,

  // Variants specifying which part of the final address calculation is
  // used. E.g. the low 12 bits for an ADD/LDR, the middle 16 bits for a
  // MOVZ/MOVK.
  S_PAGE         = 0x010,
  S_PAGEOFF      = 0x020,
  S_HI12         = 0x030,
  S_G0           = 0x040,
  S_G1           = 0x050,
  S_G2           = 0x060,
  S_G3           = 0x070,
  S_LO15         = 0x080,
  S_AddressFragBits = 0x0f0,

  // Whether the final relocation is a checked one (where a linker should
  // perform a range-check on the final address) or not. Note that this field
  // is unfortunately sometimes omitted from the assembly syntax. E.g. :lo12:
  // on its own is a non-checked relocation. We side with ELF on being
  // explicit about this!
  S_NC           = 0x100,
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 106-133: Documented code section
```cpp

  // Convenience definitions for referring to specific textual representations
  // of relocation specifiers. Note that this means the "_NC" is sometimes
  // omitted in line with assembly syntax here (S_LO12 rather than VK_LO12_NC
  // since a user would write ":lo12:").
  S_CALL              = S_ABS,
  S_ABS_PAGE          = S_ABS          | S_PAGE,
  S_ABS_PAGE_NC       = S_ABS          | S_PAGE    | S_NC,
  S_ABS_G3            = S_ABS          | S_G3,
  S_ABS_G2            = S_ABS          | S_G2,
  S_ABS_G2_S          = S_SABS         | S_G2,
  S_ABS_G2_NC         = S_ABS          | S_G2      | S_NC,
  S_ABS_G1            = S_ABS          | S_G1,
  S_ABS_G1_S          = S_SABS         | S_G1,
  S_ABS_G1_NC         = S_ABS          | S_G1      | S_NC,
  S_ABS_G0            = S_ABS          | S_G0,
  S_ABS_G0_S          = S_SABS         | S_G0,
  S_ABS_G0_NC         = S_ABS          | S_G0      | S_NC,
  S_LO12              = S_ABS          | S_PAGEOFF | S_NC,
  S_PREL_G3           = S_PREL         | S_G3,
  S_PREL_G2           = S_PREL         | S_G2,
  S_PREL_G2_NC        = S_PREL         | S_G2      | S_NC,
  S_PREL_G1           = S_PREL         | S_G1,
  S_PREL_G1_NC        = S_PREL         | S_G1      | S_NC,
  S_PREL_G0           = S_PREL         | S_G0,
  S_PREL_G0_NC        = S_PREL         | S_G0      | S_NC,
  S_GOT_LO12          = S_GOT          | S_PAGEOFF | S_NC,
  S_GOT_PAGE          = S_GOT          | S_PAGE,
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 134-161: Core AArch64 backend logic
```cpp
  S_GOT_PAGE_LO15     = S_GOT          | S_LO15    | S_NC,
  S_GOT_AUTH_LO12     = S_GOT_AUTH     | S_PAGEOFF | S_NC,
  S_GOT_AUTH_PAGE     = S_GOT_AUTH     | S_PAGE,
  S_DTPREL_G2         = S_DTPREL       | S_G2,
  S_DTPREL_G1         = S_DTPREL       | S_G1,
  S_DTPREL_G1_NC      = S_DTPREL       | S_G1      | S_NC,
  S_DTPREL_G0         = S_DTPREL       | S_G0,
  S_DTPREL_G0_NC      = S_DTPREL       | S_G0      | S_NC,
  S_DTPREL_HI12       = S_DTPREL       | S_HI12,
  S_DTPREL_LO12       = S_DTPREL       | S_PAGEOFF,
  S_DTPREL_LO12_NC    = S_DTPREL       | S_PAGEOFF | S_NC,
  S_GOTTPREL_PAGE     = S_GOTTPREL     | S_PAGE,
  S_GOTTPREL_LO12_NC  = S_GOTTPREL     | S_PAGEOFF | S_NC,
  S_GOTTPREL_G1       = S_GOTTPREL     | S_G1,
  S_GOTTPREL_G0_NC    = S_GOTTPREL     | S_G0      | S_NC,
  S_TPREL_G2          = S_TPREL        | S_G2,
  S_TPREL_G1          = S_TPREL        | S_G1,
  S_TPREL_G1_NC       = S_TPREL        | S_G1      | S_NC,
  S_TPREL_G0          = S_TPREL        | S_G0,
  S_TPREL_G0_NC       = S_TPREL        | S_G0      | S_NC,
  S_TPREL_HI12        = S_TPREL        | S_HI12,
  S_TPREL_LO12        = S_TPREL        | S_PAGEOFF,
  S_TPREL_LO12_NC     = S_TPREL        | S_PAGEOFF | S_NC,
  S_TLSDESC_LO12      = S_TLSDESC      | S_PAGEOFF,
  S_TLSDESC_PAGE      = S_TLSDESC      | S_PAGE,
  S_TLSDESC_AUTH_LO12 = S_TLSDESC_AUTH | S_PAGEOFF,
  S_TLSDESC_AUTH_PAGE = S_TLSDESC_AUTH | S_PAGE,
  S_SECREL_LO12       = S_SECREL       | S_PAGEOFF,
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 162-187: Core AArch64 backend logic
```cpp
  S_SECREL_HI12       = S_SECREL       | S_HI12,

  // ELF relocation specifiers in data directives:
  S_PLT          = 0x400,
  S_GOTPCREL,
  S_FUNCINIT,

  // Mach-O @ relocation specifiers:
  S_MACHO_GOT,
  S_MACHO_GOTPAGE,
  S_MACHO_GOTPAGEOFF,
  S_MACHO_PAGE,
  S_MACHO_PAGEOFF,
  S_MACHO_TLVP,
  S_MACHO_TLVPPAGE,
  S_MACHO_TLVPPAGEOFF,

  S_INVALID  = 0xfff
  // clang-format on
};

/// Return the string representation of the ELF relocation specifier
/// (e.g. ":got:", ":lo12:").
StringRef getSpecifierName(Specifier S);

Specifier parsePercentSpecifierName(StringRef);
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 188-210: Function getSymbolLoc
```cpp

inline Specifier getSymbolLoc(Specifier S) {
  return static_cast<Specifier>(S & AArch64::S_SymLocBits);
}

inline Specifier getAddressFrag(Specifier S) {
  return static_cast<Specifier>(S & AArch64::S_AddressFragBits);
}

inline bool isNotChecked(Specifier S) { return S & AArch64::S_NC; }
} // namespace AArch64

class AArch64AuthMCExpr final : public MCSpecifierExpr {
  uint16_t Discriminator;
  AArch64PACKey::ID Key;

  explicit AArch64AuthMCExpr(const MCExpr *Expr, uint16_t Discriminator,
                             AArch64PACKey::ID Key, bool HasAddressDiversity,
                             SMLoc Loc)
      : MCSpecifierExpr(
            Expr, HasAddressDiversity ? AArch64::S_AUTHADDR : AArch64::S_AUTH,
            Loc),
        Discriminator(Discriminator), Key(Key) {}
```
**EN:** This block implements getSymbolLoc, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getSymbolLoc，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 211-234: Function getKey
```cpp

public:
  static const AArch64AuthMCExpr *
  create(const MCExpr *Expr, uint16_t Discriminator, AArch64PACKey::ID Key,
         bool HasAddressDiversity, MCContext &Ctx, SMLoc Loc = SMLoc());

  AArch64PACKey::ID getKey() const { return Key; }
  uint16_t getDiscriminator() const { return Discriminator; }
  bool hasAddressDiversity() const {
    return getSpecifier() == AArch64::S_AUTHADDR;
  }

  void print(raw_ostream &OS, const MCAsmInfo *MAI) const;

  static bool classof(const MCExpr *E) {
    auto *SE = dyn_cast<MCSpecifierExpr>(E);
    return SE && (SE->getSpecifier() == AArch64::S_AUTH ||
                  SE->getSpecifier() == AArch64::S_AUTHADDR);
  }
};

} // namespace llvm

#endif
```
**EN:** This block implements getKey, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getKey，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: Utils/AArch64BaseInfo.h **CN:** 目标本地依赖：Utils/AArch64BaseInfo.h
- **EN:** Core LLVM interfaces: llvm/MC/MCAsmInfoCOFF.h, llvm/MC/MCAsmInfoDarwin.h, llvm/MC/MCAsmInfoELF.h, llvm/MC/MCExpr.h, llvm/Support/Casting.h **CN:** 核心 LLVM 接口：llvm/MC/MCAsmInfoCOFF.h, llvm/MC/MCAsmInfoDarwin.h, llvm/MC/MCAsmInfoELF.h, llvm/MC/MCExpr.h, llvm/Support/Casting.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
