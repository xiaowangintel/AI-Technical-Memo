# VEELFObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/MCTargetDesc/VEELFObjectWriter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides MC-layer target descriptions such as fixups, asm info, code emission, object writing, and target registration.
  - **CN**: 提供 MC 层的目标描述，例如 fixup、汇编信息、编码发射、目标文件写出以及目标注册。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEELFObjectWriter.cpp - VE ELF Writer -----------------------------===//
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

#include "MCTargetDesc/VEMCAsmInfo.h"
#include "VEFixupKinds.h"
#include "VEMCAsmInfo.h"
#include "VEMCTargetDesc.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCValue.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEMCAsmInfo.h`, `VEFixupKinds.h`, `VEMCAsmInfo.h`, `VEMCTargetDesc.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEMCAsmInfo.h`, `VEFixupKinds.h`, `VEMCAsmInfo.h`, `VEMCTargetDesc.h`。

### Lines 20-27
```cpp
using namespace llvm;

namespace {
class VEELFObjectWriter : public MCELFObjectTargetWriter {
public:
  VEELFObjectWriter(uint8_t OSABI)
      : MCELFObjectTargetWriter(/* Is64Bit */ true, OSABI, ELF::EM_VE,
                                /* HasRelocationAddend */ true) {}
```
- **EN**: Introduces declarations for `llvm`, `VEELFObjectWriter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `VEELFObjectWriter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 28-34
```cpp

  ~VEELFObjectWriter() override = default;

protected:
  unsigned getRelocType(const MCFixup &, const MCValue &,
                        bool IsPCRel) const override;

```
- **EN**: Implements logic around `~VEELFObjectWriter`, `getRelocType`; this block maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `~VEELFObjectWriter`, `getRelocType` 实现具体逻辑；这一段映射 fixup 或重定位，工作在 MC 层。

### Lines 35-48
```cpp
  bool needsRelocateWithSymbol(const MCValue &, unsigned Type) const override;
};
} // namespace

unsigned VEELFObjectWriter::getRelocType(const MCFixup &Fixup,
                                         const MCValue &Target,
                                         bool IsPCRel) const {
  switch (Target.getSpecifier()) {
  case VE::S_TLS_GD_HI32:
  case VE::S_TLS_GD_LO32:
  case VE::S_TPOFF_HI32:
  case VE::S_TPOFF_LO32:
    if (auto *SA = const_cast<MCSymbol *>(Target.getAddSym()))
      static_cast<MCSymbolELF *>(SA)->setType(ELF::STT_TLS);
```
- **EN**: Implements logic around `needsRelocateWithSymbol`, `getRelocType`, `setType`; this block uses `switch`-based dispatch; applies conditional target rules; maps fixups or relocations; works at the MC layer.
- **CN**: 围绕 `needsRelocateWithSymbol`, `getRelocType`, `setType` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，映射 fixup 或重定位，工作在 MC 层。

### Lines 49-57
```cpp
    break;
  default:
    break;
  }
  if (auto *SExpr = dyn_cast<MCSpecifierExpr>(Fixup.getValue())) {
    if (SExpr->getSpecifier() == VE::S_PC_LO32)
      return ELF::R_VE_PC_LO32;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 58-71
```cpp
  if (IsPCRel) {
    switch (Fixup.getKind()) {
    default:
      reportError(Fixup.getLoc(), "Unsupported pc-relative fixup kind");
      return ELF::R_VE_NONE;
    case FK_Data_1:
      reportError(Fixup.getLoc(),
                  "1-byte pc-relative data relocation is not supported");
      return ELF::R_VE_NONE;
    case FK_Data_2:
      reportError(Fixup.getLoc(),
                  "2-byte pc-relative data relocation is not supported");
      return ELF::R_VE_NONE;
    case FK_Data_4:
```
- **EN**: Implements logic around `reportError`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 72-85
```cpp
      return ELF::R_VE_SREL32;
    case FK_Data_8:
      reportError(Fixup.getLoc(),
                  "8-byte pc-relative data relocation is not supported");
      return ELF::R_VE_NONE;
    case VE::fixup_ve_reflong:
    case VE::fixup_ve_srel32:
      return ELF::R_VE_SREL32;
    case VE::fixup_ve_pc_hi32:
      return ELF::R_VE_PC_HI32;
    case VE::fixup_ve_pc_lo32:
      return ELF::R_VE_PC_LO32;
    }
  }
```
- **EN**: Implements logic around `reportError`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 86-99
```cpp

  switch (Fixup.getKind()) {
  default:
    reportError(Fixup.getLoc(), "Unknown ELF relocation type");
    return ELF::R_VE_NONE;
  case FK_Data_1:
    reportError(Fixup.getLoc(), "1-byte data relocation is not supported");
    return ELF::R_VE_NONE;
  case FK_Data_2:
    reportError(Fixup.getLoc(), "2-byte data relocation is not supported");
    return ELF::R_VE_NONE;
  case FK_Data_4:
    return ELF::R_VE_REFLONG;
  case FK_Data_8:
```
- **EN**: Implements logic around `reportError`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 100-113
```cpp
    return ELF::R_VE_REFQUAD;
  case VE::fixup_ve_reflong:
    return ELF::R_VE_REFLONG;
  case VE::fixup_ve_srel32:
    reportError(Fixup.getLoc(),
                "A non pc-relative srel32 relocation is not supported");
    return ELF::R_VE_NONE;
  case VE::fixup_ve_hi32:
    return ELF::R_VE_HI32;
  case VE::fixup_ve_lo32:
    return ELF::R_VE_LO32;
  case VE::fixup_ve_pc_hi32:
    reportError(Fixup.getLoc(),
                "A non pc-relative pc_hi32 relocation is not supported");
```
- **EN**: Implements logic around `reportError`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 114-127
```cpp
    return ELF::R_VE_NONE;
  case VE::fixup_ve_pc_lo32:
    reportError(Fixup.getLoc(),
                "A non pc-relative pc_lo32 relocation is not supported");
    return ELF::R_VE_NONE;
  case VE::fixup_ve_got_hi32:
    return ELF::R_VE_GOT_HI32;
  case VE::fixup_ve_got_lo32:
    return ELF::R_VE_GOT_LO32;
  case VE::fixup_ve_gotoff_hi32:
    return ELF::R_VE_GOTOFF_HI32;
  case VE::fixup_ve_gotoff_lo32:
    return ELF::R_VE_GOTOFF_LO32;
  case VE::fixup_ve_plt_hi32:
```
- **EN**: Implements logic around `reportError`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `reportError` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 128-140
```cpp
    return ELF::R_VE_PLT_HI32;
  case VE::fixup_ve_plt_lo32:
    return ELF::R_VE_PLT_LO32;
  case VE::fixup_ve_tls_gd_hi32:
    return ELF::R_VE_TLS_GD_HI32;
  case VE::fixup_ve_tls_gd_lo32:
    return ELF::R_VE_TLS_GD_LO32;
  case VE::fixup_ve_tpoff_hi32:
    return ELF::R_VE_TPOFF_HI32;
  case VE::fixup_ve_tpoff_lo32:
    return ELF::R_VE_TPOFF_LO32;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 141-149
```cpp
  return ELF::R_VE_NONE;
}

bool VEELFObjectWriter::needsRelocateWithSymbol(const MCValue &,
                                                unsigned Type) const {
  switch (Type) {
  default:
    return false;

```
- **EN**: Implements logic around `needsRelocateWithSymbol`; this block uses `switch`-based dispatch; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `needsRelocateWithSymbol` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，映射 fixup 或重定位。

### Lines 150-163
```cpp
  // All relocations that use a GOT need a symbol, not an offset, as
  // the offset of the symbol within the section is irrelevant to
  // where the GOT entry is. Don't need to list all the TLS entries,
  // as they're all marked as requiring a symbol anyways.
  case ELF::R_VE_GOT_HI32:
  case ELF::R_VE_GOT_LO32:
  case ELF::R_VE_GOTOFF_HI32:
  case ELF::R_VE_GOTOFF_LO32:
  case ELF::R_VE_TLS_GD_HI32:
  case ELF::R_VE_TLS_GD_LO32:
    return true;
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 164-167
```cpp
std::unique_ptr<MCObjectTargetWriter>
llvm::createVEELFObjectWriter(uint8_t OSABI) {
  return std::make_unique<VEELFObjectWriter>(OSABI);
}
```
- **EN**: Implements logic around `createVEELFObjectWriter`, `make_unique<VEELFObjectWriter>`; this block returns target-specific results.
- **CN**: 围绕 `createVEELFObjectWriter`, `make_unique<VEELFObjectWriter>` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **MC layer / MC 层**:
  - **EN**: Connects the backend to LLVM's MC infrastructure
  - **CN**: 把后端接入 LLVM 的 MC 基础设施
- **Relocations / 重定位**:
  - **EN**: Chooses ELF relocation kinds for fixups
  - **CN**: 为 fixup 选择 ELF 重定位类型
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEMCAsmInfo.h`, `VEFixupKinds.h`, `VEMCAsmInfo.h`, `VEMCTargetDesc.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCValue.h`, `llvm/Support/ErrorHandling.h`
- **LLVM subsystems / LLVM 子系统**: MC, Support
