# SparcAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/MCTargetDesc/SparcAsmBackend.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcAsmBackend.cpp - Sparc Assembler Backend ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-20
```cpp

#include "MCTargetDesc/SparcFixupKinds.h"
#include "MCTargetDesc/SparcMCTargetDesc.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/EndianStream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/SparcFixupKinds.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCAsmBackend.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/SparcFixupKinds.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCAsmBackend.h`。

### Lines 21-33
```cpp
using namespace llvm;

static unsigned adjustFixupValue(unsigned Kind, uint64_t Value) {
  switch (Kind) {
  default:
    assert(uint16_t(Kind) < FirstTargetFixupKind && "Unknown fixup kind!");
    return Value;
  case FK_Data_1:
  case FK_Data_2:
  case FK_Data_4:
  case FK_Data_8:
    return Value;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-42
```cpp
  case Sparc::fixup_sparc_call30:
    return (Value >> 2) & 0x3fffffff;

  case ELF::R_SPARC_WDISP22:
    return (Value >> 2) & 0x3fffff;

  case ELF::R_SPARC_WDISP19:
    return (Value >> 2) & 0x7ffff;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 43-51
```cpp
  case ELF::R_SPARC_WDISP16: {
    // A.3 Branch on Integer Register with Prediction (BPr)
    // Inst{21-20} = d16hi;
    // Inst{13-0}  = d16lo;
    unsigned d16hi = (Value >> 16) & 0x3;
    unsigned d16lo = (Value >> 2) & 0x3fff;
    return (d16hi << 20) | d16lo;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 52-63
```cpp
  case ELF::R_SPARC_WDISP10: {
    // FIXME this really should be an error reporting check.
    assert((Value & 0x3) == 0);

    // 7.17 Compare and Branch
    // Inst{20-19} = d10hi;
    // Inst{12-5}  = d10lo;
    unsigned d10hi = (Value >> 10) & 0x3;
    unsigned d10lo = (Value >> 2) & 0xff;
    return (d10hi << 19) | (d10lo << 5);
  }

```
- **EN**: Implements logic around `assert`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 64-71
```cpp
  case ELF::R_SPARC_HIX22:
    return (~Value >> 10) & 0x3fffff;

  case ELF::R_SPARC_PC22:
  case ELF::R_SPARC_HI22:
  case ELF::R_SPARC_LM22:
    return (Value >> 10) & 0x3fffff;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 72-80
```cpp
  case Sparc::fixup_sparc_13:
    return Value & 0x1fff;

  case ELF::R_SPARC_5:
    return Value & 0x1f;

  case ELF::R_SPARC_LOX10:
    return (Value & 0x3ff) | 0x1c00;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 81-91
```cpp
  case ELF::R_SPARC_PC10:
  case ELF::R_SPARC_LO10:
    return Value & 0x3ff;

  case ELF::R_SPARC_H44:
    return (Value >> 22) & 0x3fffff;
  case ELF::R_SPARC_M44:
    return (Value >> 12) & 0x3ff;
  case ELF::R_SPARC_L44:
    return Value & 0xfff;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 92-98
```cpp
  case ELF::R_SPARC_HH22:
    return (Value >> 42) & 0x3fffff;
  case ELF::R_SPARC_HM10:
    return (Value >> 32) & 0x3ff;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 99-112
```cpp
/// getFixupKindNumBytes - The number of bytes the fixup may change.
static unsigned getFixupKindNumBytes(unsigned Kind) {
    switch (Kind) {
  default:
    return 4;
  case FK_Data_1:
    return 1;
  case FK_Data_2:
    return 2;
  case FK_Data_8:
    return 8;
  }
}

```
- **EN**: Implements logic around `getFixupKindNumBytes`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindNumBytes` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 113-126
```cpp
namespace {
class SparcAsmBackend : public MCAsmBackend {
protected:
  bool Is64Bit;
  bool IsV8Plus;

public:
  SparcAsmBackend(const MCSubtargetInfo &STI)
      : MCAsmBackend(STI.getTargetTriple().isLittleEndian()
                         ? llvm::endianness::little
                         : llvm::endianness::big),
        Is64Bit(STI.getTargetTriple().isArch64Bit()),
        IsV8Plus(STI.hasFeature(Sparc::FeatureV8Plus)) {}

```
- **EN**: Introduces declarations for `SparcAsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcAsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 127-134
```cpp
  std::optional<MCFixupKind> getFixupKind(StringRef Name) const override;
  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;
  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
                  uint8_t *Data, uint64_t Value, bool IsResolved) override;

  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override {

```
- **EN**: Implements logic around `getFixupKind`, `getFixupKindInfo`, `applyFixup`, `writeNopData`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getFixupKind`, `getFixupKindInfo`, `applyFixup`, `writeNopData` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 135-143
```cpp
    // If the count is not 4-byte aligned, we must be writing data into the
    // text section (otherwise we have unaligned instructions, and thus have
    // far bigger problems), so just write zeros instead.
    OS.write_zeros(Count % 4);

    uint64_t NumNops = Count / 4;
    for (uint64_t i = 0; i != NumNops; ++i)
      support::endian::write<uint32_t>(OS, 0x01000000, Endian);

```
- **EN**: Implements logic around `write_zeros`, `write<uint32_t>`.
- **CN**: 围绕 `write_zeros`, `write<uint32_t>` 实现具体逻辑。

### Lines 144-150
```cpp
    return true;
  }
};

class ELFSparcAsmBackend : public SparcAsmBackend {
  Triple::OSType OSType;

```
- **EN**: Introduces declarations for `ELFSparcAsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFSparcAsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 151-162
```cpp
public:
  ELFSparcAsmBackend(const MCSubtargetInfo &STI, Triple::OSType OSType)
      : SparcAsmBackend(STI), OSType(OSType) {}

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    uint8_t OSABI = MCELFObjectTargetWriter::getOSABI(OSType);
    return createSparcELFObjectWriter(Is64Bit, IsV8Plus, OSABI);
  }
};
} // end anonymous namespace

```
- **EN**: Implements logic around `ELFSparcAsmBackend`, `SparcAsmBackend`, `createObjectTargetWriter`, `getOSABI`, ...; this block returns target-specific results.
- **CN**: 围绕 `ELFSparcAsmBackend`, `SparcAsmBackend`, `createObjectTargetWriter`, `getOSABI`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 163-176
```cpp
std::optional<MCFixupKind> SparcAsmBackend::getFixupKind(StringRef Name) const {
  unsigned Type;
  Type = llvm::StringSwitch<unsigned>(Name)
#define ELF_RELOC(X, Y) .Case(#X, Y)
#include "llvm/BinaryFormat/ELFRelocs/Sparc.def"
#undef ELF_RELOC
             .Case("BFD_RELOC_NONE", ELF::R_SPARC_NONE)
             .Case("BFD_RELOC_8", ELF::R_SPARC_8)
             .Case("BFD_RELOC_16", ELF::R_SPARC_16)
             .Case("BFD_RELOC_32", ELF::R_SPARC_32)
             .Case("BFD_RELOC_64", ELF::R_SPARC_64)
             .Default(-1u);
  if (Type == -1u)
    return std::nullopt;
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/BinaryFormat/ELFRelocs/Sparc.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/BinaryFormat/ELFRelocs/Sparc.def`。

### Lines 177-187
```cpp
  return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
}

MCFixupKindInfo SparcAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  // clang-format off
  const static MCFixupKindInfo InfosBE[Sparc::NumTargetFixupKinds] = {
      // name                    offset bits flags
      {"fixup_sparc_call30",     2,     30,  0},
      {"fixup_sparc_13",        19,     13,  0},
  };

```
- **EN**: Implements logic around `static_cast<MCFixupKind>`, `getFixupKindInfo`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `static_cast<MCFixupKind>`, `getFixupKindInfo` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 188-194
```cpp
  const static MCFixupKindInfo InfosLE[Sparc::NumTargetFixupKinds] = {
      // name                    offset bits flags
      {"fixup_sparc_call30",     0,     30,  0},
      {"fixup_sparc_13",         0,     13,  0},
  };
  // clang-format on

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 195-202
```cpp
  if (!mc::isRelocation(Kind)) {
    if (Kind < FirstTargetFixupKind)
      return MCAsmBackend::getFixupKindInfo(Kind);
    assert(unsigned(Kind - FirstTargetFixupKind) < Sparc::NumTargetFixupKinds &&
           "Invalid kind!");
    if (Endian == llvm::endianness::little)
      return InfosLE[Kind - FirstTargetFixupKind];

```
- **EN**: Implements logic around `getFixupKindInfo`, `assert`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindInfo`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 203-216
```cpp
    return InfosBE[Kind - FirstTargetFixupKind];
  }

  MCFixupKindInfo Info{};
  switch (uint16_t(Kind)) {
  case ELF::R_SPARC_PC10:
    Info = {"", 22, 10, 0};
    break;
  case ELF::R_SPARC_PC22:
    Info = {"", 10, 22, 0};
    break;
  case ELF::R_SPARC_WDISP10:
    Info = {"", 0, 32, 0};
    break;
```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 217-226
```cpp
  case ELF::R_SPARC_WDISP16:
    Info = {"", 0, 32, 0};
    break;
  case ELF::R_SPARC_WDISP19:
    Info = {"", 13, 19, 0};
    break;
  case ELF::R_SPARC_WDISP22:
    Info = {"", 10, 22, 0};
    break;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 227-240
```cpp
  case ELF::R_SPARC_HI22:
    Info = {"", 10, 22, 0};
    break;
  case ELF::R_SPARC_LO10:
    Info = {"", 22, 10, 0};
    break;
  case ELF::R_SPARC_HH22:
    Info = {"", 10, 22, 0};
    break;
  case ELF::R_SPARC_HM10:
    Info = {"", 22, 10, 0};
    break;
  case ELF::R_SPARC_LM22:
    Info = {"", 10, 22, 0};
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 241-253
```cpp
    break;
  case ELF::R_SPARC_HIX22:
    Info = {"", 10, 22, 0};
    break;
  case ELF::R_SPARC_LOX10:
    Info = {"", 19, 13, 0};
    break;
  }
  if (Endian == llvm::endianness::little)
    Info.TargetOffset = 32 - Info.TargetOffset - Info.TargetSize;
  return Info;
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 254-261
```cpp
void SparcAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                                 const MCValue &Target, uint8_t *Data,
                                 uint64_t Value, bool IsResolved) {
  maybeAddReloc(F, Fixup, Target, Value, IsResolved);
  if (!IsResolved)
    return;
  Value = adjustFixupValue(Fixup.getKind(), Value);

```
- **EN**: Implements logic around `applyFixup`, `maybeAddReloc`, `adjustFixupValue`; this block applies conditional target rules; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `maybeAddReloc`, `adjustFixupValue` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位，工作在 MC 层。

### Lines 262-270
```cpp
  unsigned NumBytes = getFixupKindNumBytes(Fixup.getKind());
  // For each byte of the fragment that the fixup touches, mask in the
  // bits from the fixup value.
  for (unsigned i = 0; i != NumBytes; ++i) {
    unsigned Idx = Endian == llvm::endianness::little ? i : (NumBytes - 1) - i;
    Data[Idx] |= uint8_t((Value >> (i * 8)) & 0xff);
  }
}

```
- **EN**: Implements logic around `getFixupKindNumBytes`, `uint8_t`; this block maps fixups or relocations.
- **CN**: 围绕 `getFixupKindNumBytes`, `uint8_t` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 271-276
```cpp
MCAsmBackend *llvm::createSparcAsmBackend(const Target &T,
                                          const MCSubtargetInfo &STI,
                                          const MCRegisterInfo &MRI,
                                          const MCTargetOptions &Options) {
  return new ELFSparcAsmBackend(STI, STI.getTargetTriple().getOS());
}
```
- **EN**: Implements logic around `createSparcAsmBackend`, `ELFSparcAsmBackend`; this block returns target-specific results.
- **CN**: 围绕 `createSparcAsmBackend`, `ELFSparcAsmBackend` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/SparcFixupKinds.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCValue.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/EndianStream.h`, `llvm/BinaryFormat/ELFRelocs/Sparc.def`
- **LLVM subsystems / LLVM 子系统**: MC, Support
