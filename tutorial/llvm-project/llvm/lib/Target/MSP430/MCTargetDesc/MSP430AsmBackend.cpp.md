# MSP430AsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MCTargetDesc/MSP430AsmBackend.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MSP430AsmBackend.cpp - MSP430 Assembler Backend -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-17
```cpp

#include "MCTargetDesc/MSP430FixupKinds.h"
#include "MCTargetDesc/MSP430MCTargetDesc.h"
#include "llvm/ADT/APInt.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/MSP430FixupKinds.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `llvm/ADT/APInt.h`, `llvm/MC/MCAsmBackend.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/MSP430FixupKinds.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `llvm/ADT/APInt.h`, `llvm/MC/MCAsmBackend.h`。

### Lines 18-23
```cpp
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/ErrorHandling.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCTargetOptions.h`, `llvm/Support/ErrorHandling.h`。

### Lines 24-29
```cpp
using namespace llvm;

namespace {
class MSP430AsmBackend : public MCAsmBackend {
  uint8_t OSABI;

```
- **EN**: Introduces declarations for `llvm`, `MSP430AsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `MSP430AsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 30-37
```cpp
  uint64_t adjustFixupValue(const MCFixup &Fixup, uint64_t Value,
                            MCContext &Ctx) const;

public:
  MSP430AsmBackend(const MCSubtargetInfo &STI, uint8_t OSABI)
      : MCAsmBackend(llvm::endianness::little), OSABI(OSABI) {}
  ~MSP430AsmBackend() override = default;

```
- **EN**: Implements logic around `adjustFixupValue`, `MSP430AsmBackend`, `MCAsmBackend`, `~MSP430AsmBackend`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `adjustFixupValue`, `MSP430AsmBackend`, `MCAsmBackend`, `~MSP430AsmBackend` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 38-45
```cpp
  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
                  uint8_t *Data, uint64_t Value, bool IsResolved) override;

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    return createMSP430ELFObjectWriter(OSABI);
  }

```
- **EN**: Implements logic around `applyFixup`, `createObjectTargetWriter`, `createMSP430ELFObjectWriter`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `createObjectTargetWriter`, `createMSP430ELFObjectWriter` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 46-55
```cpp
  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override {
    // clang-format off
    const static MCFixupKindInfo Infos[MSP430::NumTargetFixupKinds] = {
      // This table must be in the same order of enum in MSP430FixupKinds.h.
      //
      // name            offset bits flags
      {"fixup_32",            0, 32, 0},
      {"fixup_10_pcrel",      0, 10, 0},
      {"fixup_16",            0, 16, 0},
      {"fixup_16_pcrel",      0, 16, 0},
```
- **EN**: Introduces declarations for `in`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `in` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 56-65
```cpp
      {"fixup_16_byte",       0, 16, 0},
      {"fixup_16_pcrel_byte", 0, 16, 0},
      {"fixup_2x_pcrel",      0, 10, 0},
      {"fixup_rl_pcrel",      0, 16, 0},
      {"fixup_8",             0,  8, 0},
      {"fixup_sym_diff",      0, 32, 0},
    };
    // clang-format on
    static_assert((std::size(Infos)) == MSP430::NumTargetFixupKinds,
                  "Not all fixup kinds added to Infos array");
```
- **EN**: Implements logic around `static_assert`; this block maps fixups or relocations.
- **CN**: 围绕 `static_assert` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 66-72
```cpp

    if (Kind < FirstTargetFixupKind)
      return MCAsmBackend::getFixupKindInfo(Kind);
  
    return Infos[Kind - FirstTargetFixupKind];
  }

```
- **EN**: Implements logic around `getFixupKindInfo`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 73-82
```cpp
  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override;
};

uint64_t MSP430AsmBackend::adjustFixupValue(const MCFixup &Fixup,
                                            uint64_t Value,
                                            MCContext &Ctx) const {
  unsigned Kind = Fixup.getKind();
  switch (Kind) {
  case MSP430::fixup_10_pcrel: {
```
- **EN**: Implements logic around `writeNopData`, `adjustFixupValue`, `getKind`; this block uses `switch`-based dispatch; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `writeNopData`, `adjustFixupValue`, `getKind` 实现具体逻辑；这一段使用 `switch` 分派，映射 fixup 或重定位，工作在 MC 层。

### Lines 83-92
```cpp
    if (Value & 0x1)
      Ctx.reportError(Fixup.getLoc(), "fixup value must be 2-byte aligned");

    // Offset is signed
    int16_t Offset = Value;
    // Jumps are in words
    Offset >>= 1;
    // PC points to the next instruction so decrement by one
    --Offset;

```
- **EN**: Implements logic around `reportError`; this block applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位。

### Lines 93-98
```cpp
    if (Offset < -512 || Offset > 511)
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");

    // Mask 10 bits
    Offset &= 0x3ff;

```
- **EN**: Implements logic around `reportError`; this block applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位。

### Lines 99-105
```cpp
    return Offset;
  }
  default:
    return Value;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 106-114
```cpp
void MSP430AsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                                  const MCValue &Target, uint8_t *Data,
                                  uint64_t Value, bool IsResolved) {
  maybeAddReloc(F, Fixup, Target, Value, IsResolved);
  Value = adjustFixupValue(Fixup, Value, getContext());
  MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());
  if (!Value)
    return; // Doesn't change encoding.

```
- **EN**: Implements logic around `applyFixup`, `maybeAddReloc`, `adjustFixupValue`, `getFixupKindInfo`; this block applies conditional target rules; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `maybeAddReloc`, `adjustFixupValue`, `getFixupKindInfo` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位，工作在 MC 层。

### Lines 115-121
```cpp
  // Shift the value into position.
  Value <<= Info.TargetOffset;

  unsigned NumBytes = alignTo(Info.TargetSize + Info.TargetOffset, 8) / 8;
  assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
         "Invalid fixup offset!");

```
- **EN**: Implements logic around `alignTo`, `assert`; this block maps fixups or relocations.
- **CN**: 围绕 `alignTo`, `assert` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 122-128
```cpp
  // For each byte of the fragment that the fixup touches, mask in the
  // bits from the fixup value.
  for (unsigned i = 0; i != NumBytes; ++i) {
    Data[i] |= uint8_t((Value >> (i * 8)) & 0xff);
  }
}

```
- **EN**: Implements logic around `uint8_t`; this block maps fixups or relocations.
- **CN**: 围绕 `uint8_t` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 129-133
```cpp
bool MSP430AsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
                                    const MCSubtargetInfo *STI) const {
  if ((Count % 2) != 0)
    return false;

```
- **EN**: Implements logic around `writeNopData`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `writeNopData` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 134-138
```cpp
  // The canonical nop on MSP430 is mov #0, r3
  uint64_t NopCount = Count / 2;
  while (NopCount--)
    OS.write("\x03\x43", 2);

```
- **EN**: Implements logic around `write`.
- **CN**: 围绕 `write` 实现具体逻辑。

### Lines 139-143
```cpp
  return true;
}

} // end anonymous namespace

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 144-149
```cpp
MCAsmBackend *llvm::createMSP430MCAsmBackend(const Target &T,
                                             const MCSubtargetInfo &STI,
                                             const MCRegisterInfo &MRI,
                                             const MCTargetOptions &Options) {
  return new MSP430AsmBackend(STI, ELF::ELFOSABI_STANDALONE);
}
```
- **EN**: Implements logic around `createMSP430MCAsmBackend`, `MSP430AsmBackend`; this block returns target-specific results.
- **CN**: 围绕 `createMSP430MCAsmBackend`, `MSP430AsmBackend` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/MSP430FixupKinds.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `llvm/ADT/APInt.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/MCTargetOptions.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
