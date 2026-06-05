# XtensaAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/MCTargetDesc/XtensaAsmBackend.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----------------------------------------------------------------------===//
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

#include "MCTargetDesc/XtensaFixupKinds.h"
#include "MCTargetDesc/XtensaMCTargetDesc.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/raw_ostream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/XtensaFixupKinds.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/XtensaFixupKinds.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`。

### Lines 21-30
```cpp
using namespace llvm;

namespace llvm {
class MCObjectTargetWriter;
}
namespace {
class XtensaAsmBackend : public MCAsmBackend {
  uint8_t OSABI;
  bool IsLittleEndian;

```
- **EN**: Introduces declarations for `llvm`, `MCObjectTargetWriter`, `XtensaAsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `MCObjectTargetWriter`, `XtensaAsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-43
```cpp
public:
  XtensaAsmBackend(uint8_t osABI, bool isLE)
      : MCAsmBackend(llvm::endianness::little), OSABI(osABI),
        IsLittleEndian(isLE) {}

  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;
  std::optional<bool> evaluateFixup(const MCFragment &, MCFixup &, MCValue &,
                                    uint64_t &) override;
  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
                  uint8_t *Data, uint64_t Value, bool IsResolved) override;
  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override;

```
- **EN**: Implements logic around `XtensaAsmBackend`, `MCAsmBackend`, `IsLittleEndian`, `getFixupKindInfo`, ...; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `XtensaAsmBackend`, `MCAsmBackend`, `IsLittleEndian`, `getFixupKindInfo`, ... 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 44-57
```cpp
  std::unique_ptr<MCObjectTargetWriter> createObjectTargetWriter() const override {
    return createXtensaObjectWriter(OSABI, IsLittleEndian);
  }
};
} // namespace

MCFixupKindInfo XtensaAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  const static MCFixupKindInfo Infos[Xtensa::NumTargetFixupKinds] = {
      // name                     offset bits  flags
      {"fixup_xtensa_branch_6", 0, 16, 0},
      {"fixup_xtensa_branch_8", 16, 8, 0},
      {"fixup_xtensa_branch_12", 12, 12, 0},
      {"fixup_xtensa_jump_18", 6, 18, 0},
      {"fixup_xtensa_call_18", 6, 18, 0},
```
- **EN**: Implements logic around `createObjectTargetWriter`, `createXtensaObjectWriter`, `getFixupKindInfo`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `createObjectTargetWriter`, `createXtensaObjectWriter`, `getFixupKindInfo` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 58-68
```cpp
      {"fixup_xtensa_l32r_16", 8, 16, 0},
      {"fixup_xtensa_loop_8", 16, 8, 0},
  };

  if (Kind < FirstTargetFixupKind)
    return MCAsmBackend::getFixupKindInfo(Kind);
  assert(unsigned(Kind - FirstTargetFixupKind) < Xtensa::NumTargetFixupKinds &&
         "Invalid kind!");
  return Infos[Kind - FirstTargetFixupKind];
}

```
- **EN**: Implements logic around `getFixupKindInfo`, `assert`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindInfo`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 69-82
```cpp
static uint64_t adjustFixupValue(const MCFixup &Fixup, uint64_t Value,
                                 MCContext &Ctx) {
  unsigned Kind = Fixup.getKind();
  switch (Kind) {
  default:
    llvm_unreachable("Unknown fixup kind!");
  case FK_Data_1:
  case FK_Data_2:
  case FK_Data_4:
  case FK_Data_8:
    return Value;
  case Xtensa::fixup_xtensa_branch_6: {
    if (!Value)
      return 0;
```
- **EN**: Implements logic around `adjustFixupValue`, `getKind`, `llvm_unreachable`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `adjustFixupValue`, `getKind`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 83-96
```cpp
    Value -= 4;
    if (!isUInt<6>(Value))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    unsigned Hi2 = (Value >> 4) & 0x3;
    unsigned Lo4 = Value & 0xf;
    return (Hi2 << 4) | (Lo4 << 12);
  }
  case Xtensa::fixup_xtensa_branch_8:
    Value -= 4;
    if (!isInt<8>(Value))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    return (Value & 0xff);
  case Xtensa::fixup_xtensa_branch_12:
    Value -= 4;
```
- **EN**: Implements logic around `reportError`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 97-110
```cpp
    if (!isInt<12>(Value))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    return (Value & 0xfff);
  case Xtensa::fixup_xtensa_jump_18:
    Value -= 4;
    if (!isInt<18>(Value))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    return (Value & 0x3ffff);
  case Xtensa::fixup_xtensa_call_18:
    Value -= 4;
    if (!isInt<20>(Value))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    if (Value & 0x3)
      Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned");
```
- **EN**: Implements logic around `reportError`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 111-124
```cpp
    return (Value & 0xffffc) >> 2;
  case Xtensa::fixup_xtensa_loop_8:
    Value -= 4;
    if (!isUInt<8>(Value))
      Ctx.reportError(Fixup.getLoc(), "loop fixup value out of range");
    return (Value & 0xff);
  case Xtensa::fixup_xtensa_l32r_16:
    unsigned Offset = Fixup.getOffset();
    if (Offset & 0x3)
      Value -= 4;
    if (!isInt<18>(Value) && (Value & 0x20000))
      Ctx.reportError(Fixup.getLoc(), "fixup value out of range");
    if (Value & 0x3)
      Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned");
```
- **EN**: Implements logic around `reportError`, `getOffset`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError`, `getOffset` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 125-138
```cpp
    return (Value & 0x3fffc) >> 2;
  }
}

static unsigned getSize(unsigned Kind) {
  switch (Kind) {
  default:
    return 3;
  case FK_Data_4:
    return 4;
  case Xtensa::fixup_xtensa_branch_6:
    return 2;
  }
}
```
- **EN**: Implements logic around `getSize`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getSize` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 139-152
```cpp

std::optional<bool> XtensaAsmBackend::evaluateFixup(const MCFragment &F,
                                                    MCFixup &Fixup, MCValue &,
                                                    uint64_t &Value) {
  // For a few PC-relative fixups, offsets need to be aligned down. We
  // compensate here because the default handler's `Value` decrement doesn't
  // account for this alignment.
  switch (Fixup.getKind()) {
  case Xtensa::fixup_xtensa_call_18:
  case Xtensa::fixup_xtensa_l32r_16:
    Value = (Asm->getFragmentOffset(F) + Fixup.getOffset()) % 4;
  }
  return {};
}
```
- **EN**: Implements logic around `evaluateFixup`, `getFragmentOffset`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `evaluateFixup`, `getFragmentOffset` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 153-160
```cpp

void XtensaAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                                  const MCValue &Target, uint8_t *Data,
                                  uint64_t Value, bool IsResolved) {
  maybeAddReloc(F, Fixup, Target, Value, IsResolved);
  MCContext &Ctx = getContext();
  MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());

```
- **EN**: Implements logic around `applyFixup`, `maybeAddReloc`, `getContext`, `getFixupKindInfo`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `maybeAddReloc`, `getContext`, `getFixupKindInfo` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 161-168
```cpp
  Value = adjustFixupValue(Fixup, Value, Ctx);

  // Shift the value into position.
  Value <<= Info.TargetOffset;

  if (!Value)
    return; // Doesn't change encoding.

```
- **EN**: Implements logic around `adjustFixupValue`; this block applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `adjustFixupValue` 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位。

### Lines 169-175
```cpp
  unsigned FullSize = getSize(Fixup.getKind());

  for (unsigned i = 0; i != FullSize; ++i) {
    Data[i] |= uint8_t((Value >> (i * 8)) & 0xff);
  }
}

```
- **EN**: Implements logic around `getSize`, `uint8_t`; this block maps fixups or relocations.
- **CN**: 围绕 `getSize`, `uint8_t` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 176-189
```cpp
bool XtensaAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
                                    const MCSubtargetInfo *STI) const {
  uint64_t NumNops24b = Count / 3;

  for (uint64_t i = 0; i != NumNops24b; ++i) {
    // Currently just little-endian machine supported,
    // but probably big-endian will be also implemented in future
    if (IsLittleEndian) {
      OS.write("\xf0", 1);
      OS.write("\x20", 1);
      OS.write("\0x00", 1);
    } else {
      report_fatal_error("Big-endian mode currently is not supported!");
    }
```
- **EN**: Implements logic around `writeNopData`, `write`, `report_fatal_error`; this block applies conditional target rules.
- **CN**: 围绕 `writeNopData`, `write`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 190-203
```cpp
    Count -= 3;
  }

  // TODO maybe function should return error if (Count > 0)
  switch (Count) {
  default:
    break;
  case 1:
    OS.write("\0", 1);
    break;
  case 2:
    // NOP.N instruction
    OS.write("\x3d", 1);
    OS.write("\xf0", 1);
```
- **EN**: Implements logic around `write`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `write` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 204-217
```cpp
    break;
  }

  return true;
}

MCAsmBackend *llvm::createXtensaAsmBackend(const Target &T,
                                           const MCSubtargetInfo &STI,
                                           const MCRegisterInfo &MRI,
                                           const MCTargetOptions &Options) {
  uint8_t OSABI =
      MCELFObjectTargetWriter::getOSABI(STI.getTargetTriple().getOS());
  return new XtensaAsmBackend(OSABI, true);
}
```
- **EN**: Implements logic around `createXtensaAsmBackend`, `getOSABI`, `XtensaAsmBackend`; this block returns target-specific results.
- **CN**: 围绕 `createXtensaAsmBackend`, `getOSABI`, `XtensaAsmBackend` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/XtensaFixupKinds.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCValue.h`, `llvm/Support/raw_ostream.h`
- **LLVM subsystems / LLVM 子系统**: MC, Support
