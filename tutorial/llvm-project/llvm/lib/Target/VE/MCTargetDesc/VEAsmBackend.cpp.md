# VEAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/MCTargetDesc/VEAsmBackend.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEAsmBackend.cpp - VE Assembler Backend ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-19
```cpp

#include "MCTargetDesc/VEFixupKinds.h"
#include "MCTargetDesc/VEMCTargetDesc.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/EndianStream.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEFixupKinds.h`, `MCTargetDesc/VEMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCELFObjectWriter.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEFixupKinds.h`, `MCTargetDesc/VEMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCELFObjectWriter.h`。

### Lines 20-33
```cpp
using namespace llvm;

static uint64_t adjustFixupValue(unsigned Kind, uint64_t Value) {
  switch (Kind) {
  default:
    llvm_unreachable("Unknown fixup kind!");
  case FK_Data_1:
  case FK_Data_2:
  case FK_Data_4:
  case FK_Data_8:
    return Value;
  case VE::fixup_ve_hi32:
  case VE::fixup_ve_pc_hi32:
  case VE::fixup_ve_got_hi32:
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 34-47
```cpp
  case VE::fixup_ve_gotoff_hi32:
  case VE::fixup_ve_plt_hi32:
  case VE::fixup_ve_tls_gd_hi32:
  case VE::fixup_ve_tpoff_hi32:
    return (Value >> 32) & 0xffffffff;
  case VE::fixup_ve_reflong:
  case VE::fixup_ve_srel32:
  case VE::fixup_ve_lo32:
  case VE::fixup_ve_pc_lo32:
  case VE::fixup_ve_got_lo32:
  case VE::fixup_ve_gotoff_lo32:
  case VE::fixup_ve_plt_lo32:
  case VE::fixup_ve_tls_gd_lo32:
  case VE::fixup_ve_tpoff_lo32:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 48-61
```cpp
    return Value & 0xffffffff;
  }
}

/// getFixupKindNumBytes - The number of bytes the fixup may change.
static unsigned getFixupKindNumBytes(unsigned Kind) {
  switch (Kind) {
  default:
    llvm_unreachable("Unknown fixup kind!");
  case FK_Data_1:
    return 1;
  case FK_Data_2:
    return 2;
  case FK_Data_4:
```
- **EN**: Implements logic around `getFixupKindNumBytes`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindNumBytes`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 62-75
```cpp
  case VE::fixup_ve_reflong:
  case VE::fixup_ve_srel32:
  case VE::fixup_ve_hi32:
  case VE::fixup_ve_lo32:
  case VE::fixup_ve_pc_hi32:
  case VE::fixup_ve_pc_lo32:
  case VE::fixup_ve_got_hi32:
  case VE::fixup_ve_got_lo32:
  case VE::fixup_ve_gotoff_hi32:
  case VE::fixup_ve_gotoff_lo32:
  case VE::fixup_ve_plt_hi32:
  case VE::fixup_ve_plt_lo32:
  case VE::fixup_ve_tls_gd_hi32:
  case VE::fixup_ve_tls_gd_lo32:
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 76-83
```cpp
  case VE::fixup_ve_tpoff_hi32:
  case VE::fixup_ve_tpoff_lo32:
    return 4;
  case FK_Data_8:
    return 8;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 84-92
```cpp
namespace {
class VEAsmBackend : public MCAsmBackend {
protected:
  const Target &TheTarget;

public:
  VEAsmBackend(const Target &T)
      : MCAsmBackend(llvm::endianness::little), TheTarget(T) {}

```
- **EN**: Introduces declarations for `VEAsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEAsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 93-105
```cpp
  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override {
    const static MCFixupKindInfo Infos[VE::NumTargetFixupKinds] = {
        // name, offset, bits, flags
        {"fixup_ve_reflong", 0, 32, 0},     {"fixup_ve_srel32", 0, 32, 0},
        {"fixup_ve_hi32", 0, 32, 0},        {"fixup_ve_lo32", 0, 32, 0},
        {"fixup_ve_pc_hi32", 0, 32, 0},     {"fixup_ve_pc_lo32", 0, 32, 0},
        {"fixup_ve_got_hi32", 0, 32, 0},    {"fixup_ve_got_lo32", 0, 32, 0},
        {"fixup_ve_gotoff_hi32", 0, 32, 0}, {"fixup_ve_gotoff_lo32", 0, 32, 0},
        {"fixup_ve_plt_hi32", 0, 32, 0},    {"fixup_ve_plt_lo32", 0, 32, 0},
        {"fixup_ve_tls_gd_hi32", 0, 32, 0}, {"fixup_ve_tls_gd_lo32", 0, 32, 0},
        {"fixup_ve_tpoff_hi32", 0, 32, 0},  {"fixup_ve_tpoff_lo32", 0, 32, 0},
    };

```
- **EN**: Implements logic around `getFixupKindInfo`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getFixupKindInfo` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 106-113
```cpp
    if (Kind < FirstTargetFixupKind)
      return MCAsmBackend::getFixupKindInfo(Kind);

    assert(unsigned(Kind - FirstTargetFixupKind) < VE::NumTargetFixupKinds &&
           "Invalid kind!");
    return Infos[Kind - FirstTargetFixupKind];
  }

```
- **EN**: Implements logic around `getFixupKindInfo`, `assert`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getFixupKindInfo`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 114-124
```cpp
  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &,
                  uint8_t *, uint64_t Value, bool IsResolved) override;

  bool mayNeedRelaxation(unsigned Opcode, ArrayRef<MCOperand> Operands,
                         const MCSubtargetInfo &STI) const override {
    // Not implemented yet.  For example, if we have a branch with
    // lager than SIMM32 immediate value, we want to relaxation such
    // branch instructions.
    return false;
  }

```
- **EN**: Implements logic around `applyFixup`, `mayNeedRelaxation`; this block returns target-specific results; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `mayNeedRelaxation` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位，工作在 MC 层。

### Lines 125-133
```cpp
  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override {
    if ((Count % 8) != 0)
      return false;

    for (uint64_t i = 0; i < Count; i += 8)
      support::endian::write<uint64_t>(OS, 0x7900000000000000ULL,
                                       llvm::endianness::little);

```
- **EN**: Implements logic around `writeNopData`, `write<uint64_t>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `writeNopData`, `write<uint64_t>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 134-140
```cpp
    return true;
  }
};

class ELFVEAsmBackend : public VEAsmBackend {
  Triple::OSType OSType;

```
- **EN**: Introduces declarations for `ELFVEAsmBackend`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFVEAsmBackend` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 141-152
```cpp
public:
  ELFVEAsmBackend(const Target &T, Triple::OSType OSType)
      : VEAsmBackend(T), OSType(OSType) {}

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    uint8_t OSABI = MCELFObjectTargetWriter::getOSABI(OSType);
    return createVEELFObjectWriter(OSABI);
  }
};
} // end anonymous namespace

```
- **EN**: Implements logic around `ELFVEAsmBackend`, `VEAsmBackend`, `createObjectTargetWriter`, `getOSABI`, ...; this block returns target-specific results.
- **CN**: 围绕 `ELFVEAsmBackend`, `VEAsmBackend`, `createObjectTargetWriter`, `getOSABI`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 153-166
```cpp
void VEAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                              const MCValue &Target, uint8_t *Data,
                              uint64_t Value, bool IsResolved) {
  switch (Fixup.getKind()) {
  case VE::fixup_ve_tls_gd_hi32:
  case VE::fixup_ve_tls_gd_lo32:
  case VE::fixup_ve_tpoff_hi32:
  case VE::fixup_ve_tpoff_lo32:
    IsResolved = false;
    break;
  }
  maybeAddReloc(F, Fixup, Target, Value, IsResolved);
  Value = adjustFixupValue(Fixup.getKind(), Value);
  if (!Value)
```
- **EN**: Implements logic around `applyFixup`, `maybeAddReloc`, `adjustFixupValue`; this block uses `switch`-based dispatch; applies conditional target rules; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `applyFixup`, `maybeAddReloc`, `adjustFixupValue` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，映射 fixup 或重定位，工作在 MC 层。

### Lines 167-173
```cpp
    return; // Doesn't change encoding.

  MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());

  // Shift the value into position.
  Value <<= Info.TargetOffset;

```
- **EN**: Implements logic around `getFixupKindInfo`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `getFixupKindInfo` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 174-185
```cpp
  unsigned NumBytes = getFixupKindNumBytes(Fixup.getKind());
  assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
         "Invalid fixup offset!");
  // For each byte of the fragment that the fixup touches, mask in the bits
  // from the fixup value. The Value has been "split up" into the
  // appropriate bitfields above.
  for (unsigned i = 0; i != NumBytes; ++i) {
    unsigned Idx = Endian == llvm::endianness::little ? i : (NumBytes - 1) - i;
    Data[Idx] |= static_cast<uint8_t>((Value >> (i * 8)) & 0xff);
  }
}

```
- **EN**: Implements logic around `getFixupKindNumBytes`, `assert`, `static_cast<uint8_t>`; this block maps fixups or relocations.
- **CN**: 围绕 `getFixupKindNumBytes`, `assert`, `static_cast<uint8_t>` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 186-191
```cpp
MCAsmBackend *llvm::createVEAsmBackend(const Target &T,
                                       const MCSubtargetInfo &STI,
                                       const MCRegisterInfo &MRI,
                                       const MCTargetOptions &Options) {
  return new ELFVEAsmBackend(T, STI.getTargetTriple().getOS());
}
```
- **EN**: Implements logic around `createVEAsmBackend`, `ELFVEAsmBackend`; this block returns target-specific results.
- **CN**: 围绕 `createVEAsmBackend`, `ELFVEAsmBackend` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEFixupKinds.h`, `MCTargetDesc/VEMCTargetDesc.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCValue.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/EndianStream.h`
- **LLVM subsystems / LLVM 子系统**: MC, Support
