# PPCAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCAsmBackend.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCAsmBackend.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCAsmBackend.cpp - PPC Assembler Backend -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCFixupKinds.h"
#include "MCTargetDesc/PPCMCAsmInfo.h"
#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/BinaryFormat/MachO.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 14-20

```cpp
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCMachObjectWriter.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 21-58

```cpp
#include "llvm/MC/MCSymbolELF.h"
#include "llvm/MC/MCSymbolXCOFF.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/ErrorHandling.h"
using namespace llvm;

static uint64_t adjustFixupValue(MCContext &Ctx, const MCFixup &Fixup,
                                 unsigned Kind, uint64_t Value) {
  auto checkBrFixup = [&](unsigned Bits) {
    int64_t SVal = int64_t(Value);
    if ((Value & 3) != 0) {
      Ctx.reportError(Fixup.getLoc(), "branch target not a multiple of four (" +
                                          Twine(SVal) + ")");
      return;
    }

    // Low two bits are not encoded.
    if (!isIntN(Bits + 2, Value)) {
      Ctx.reportError(Fixup.getLoc(), "branch target out of range (" +
                                          Twine(SVal) + " not between " +
                                          Twine(minIntN(Bits) * 4) + " and " +
                                          Twine(maxIntN(Bits) * 4) + ")");
    }
  };

  switch (Kind) {
  default:
    llvm_unreachable("Unknown fixup kind!");
  case FK_Data_1:
  case FK_Data_2:
  case FK_Data_4:
  case FK_Data_8:
  case PPC::fixup_ppc_nofixup:
    return Value;
  case PPC::fixup_ppc_brcond14:
  case PPC::fixup_ppc_brcond14abs:
    checkBrFixup(14);
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `adjustFixupValue`, `int64_t`, `reportError`.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `adjustFixupValue`, `int64_t`, `reportError`。

### Lines 59-96

```cpp
    return Value & 0xfffc;
  case PPC::fixup_ppc_br24:
  case PPC::fixup_ppc_br24abs:
  case PPC::fixup_ppc_br24_notoc:
    checkBrFixup(24);
    return Value & 0x3fffffc;
  case PPC::fixup_ppc_half16:
    return Value & 0xffff;
  case PPC::fixup_ppc_half16ds:
  case PPC::fixup_ppc_half16dq:
    return Value & 0xfffc;
  case PPC::fixup_ppc_pcrel32:
  case PPC::fixup_ppc_imm32:
    return Value & 0xffffffff;
  case PPC::fixup_ppc_pcrel34:
  case PPC::fixup_ppc_imm34:
    return Value & 0x3ffffffff;
  }
}

static unsigned getFixupKindNumBytes(unsigned Kind) {
  switch (Kind) {
  default:
    llvm_unreachable("Unknown fixup kind!");
  case FK_Data_1:
    return 1;
  case FK_Data_2:
  case PPC::fixup_ppc_half16:
  case PPC::fixup_ppc_half16ds:
  case PPC::fixup_ppc_half16dq:
    return 2;
  case FK_Data_4:
  case PPC::fixup_ppc_brcond14:
  case PPC::fixup_ppc_brcond14abs:
  case PPC::fixup_ppc_br24:
  case PPC::fixup_ppc_br24abs:
  case PPC::fixup_ppc_br24_notoc:
    return 4;
```
- **EN**: Implements helper routine(s) `checkBrFixup`, `getFixupKindNumBytes`, `llvm_unreachable` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `checkBrFixup`, `getFixupKindNumBytes`, `llvm_unreachable`。

### Lines 97-107

```cpp
  case PPC::fixup_ppc_pcrel32:
  case PPC::fixup_ppc_imm32:
  case PPC::fixup_ppc_pcrel34:
  case PPC::fixup_ppc_imm34:
  case FK_Data_8:
    return 8;
  case PPC::fixup_ppc_nofixup:
    return 0;
  }
}
```
- **EN**: Implements dispatch logic that chooses specialized target behavior for different opcodes, modes, or ABI cases.
- **CN**: 这一段实现分派逻辑，为不同操作码、模式或 ABI 情况选择特定的目标行为。

### Lines 108-145

```cpp
namespace {

class PPCAsmBackend : public MCAsmBackend {
protected:
  Triple TT;
public:
  PPCAsmBackend(const Target &T, const Triple &TT)
      : MCAsmBackend(TT.isLittleEndian() ? llvm::endianness::little
                                         : llvm::endianness::big),
        TT(TT) {}

  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;

  void applyFixup(const MCFragment &, const MCFixup &Fixup,
                  const MCValue &Target, uint8_t *Data, uint64_t Value,
                  bool IsResolved) override;

  bool shouldForceRelocation(const MCFixup &Fixup, const MCValue &Target) {
    // If there is a @ specifier, unless it is optimized out (e.g. constant @l),
    // force a relocation.
    if (Target.getSpecifier())
      return true;
    MCFixupKind Kind = Fixup.getKind();
    switch ((unsigned)Kind) {
    default:
      return false;
    case PPC::fixup_ppc_br24:
    case PPC::fixup_ppc_br24abs:
    case PPC::fixup_ppc_br24_notoc:
      // If the target symbol has a local entry point we must not attempt
      // to resolve the fixup directly.  Emit a relocation and leave
      // resolution of the final target address to the linker.
      if (const auto *A = Target.getAddSym()) {
        if (getContext().isELF()) {
          // The "other" values are stored in the last 6 bits of the second
          // byte. The traditional defines for STO values assume the full byte
          // and thus the shift to pack it.
          unsigned Other = static_cast<const MCSymbolELF *>(A)->getOther() << 2;
```
- **EN**: Declares a backend-facing type `PPCAsmBackend`, `MCAsmBackend`, `isLittleEndian` and outlines the API or state that nearby code will rely on. The logic interacts with LLVM's MC layer.
- **CN**: 这里声明面向后端的类型 `PPCAsmBackend`, `MCAsmBackend`, `isLittleEndian`，并勾勒出周边代码会依赖的接口或状态。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 146-183

```cpp
          if ((Other & ELF::STO_PPC64_LOCAL_MASK) != 0)
            return true;
        } else if (getContext().isXCOFF()) {
          auto *S = static_cast<const MCSymbolXCOFF *>(A);
          return !Target.isAbsolute() && S->isExternal() &&
                 S->getStorageClass() == XCOFF::C_WEAKEXT;
        }
      }
      return false;
    }
  }

  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override {
    uint64_t NumNops = Count / 4;
    for (uint64_t i = 0; i != NumNops; ++i)
      support::endian::write<uint32_t>(OS, 0x60000000, Endian);

    OS.write_zeros(Count % 4);

    return true;
  }
};
} // end anonymous namespace

MCFixupKindInfo PPCAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  // clang-format off
  const static MCFixupKindInfo InfosBE[PPC::NumTargetFixupKinds] = {
      // name                    offset  bits  flags
      {"fixup_ppc_br24", 6, 24, 0},
      {"fixup_ppc_br24_notoc", 6, 24, 0},
      {"fixup_ppc_brcond14", 16, 14, 0},
      {"fixup_ppc_br24abs", 6, 24, 0},
      {"fixup_ppc_brcond14abs", 16, 14, 0},
      {"fixup_ppc_half16", 0, 16, 0},
      {"fixup_ppc_half16ds", 0, 14, 0},
      {"fixup_ppc_pcrel32", 0, 32, 0},
      {"fixup_ppc_imm32", 0, 32, 0},
```
- **EN**: Implements helper routine(s) `getContext`, `isXCOFF`, `isAbsolute` for this portion of the PowerPC backend MC layer support for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getContext`, `isXCOFF`, `isAbsolute`。 子目标特性裁剪会影响这里的行为。

### Lines 184-221

```cpp
      {"fixup_ppc_pcrel34", 0, 34, 0},
      {"fixup_ppc_imm34", 0, 34, 0},
      {"fixup_ppc_nofixup", 0, 0, 0}};
  const static MCFixupKindInfo InfosLE[PPC::NumTargetFixupKinds] = {
      // name                    offset  bits  flags
      {"fixup_ppc_br24", 2, 24, 0},
      {"fixup_ppc_br24_notoc", 2, 24, 0},
      {"fixup_ppc_brcond14", 2, 14, 0},
      {"fixup_ppc_br24abs", 2, 24, 0},
      {"fixup_ppc_brcond14abs", 2, 14, 0},
      {"fixup_ppc_half16", 0, 16, 0},
      {"fixup_ppc_half16ds", 2, 14, 0},
      {"fixup_ppc_pcrel32", 0, 32, 0},
      {"fixup_ppc_imm32", 0, 32, 0},
      {"fixup_ppc_pcrel34", 0, 34, 0},
      {"fixup_ppc_imm34", 0, 34, 0},
      {"fixup_ppc_nofixup", 0, 0, 0}};
  // clang-format on

  // Fixup kinds from .reloc directive are like R_PPC_NONE/R_PPC64_NONE. They
  // do not require any extra processing.
  if (mc::isRelocation(Kind))
    return {};

  if (Kind < FirstTargetFixupKind)
    return MCAsmBackend::getFixupKindInfo(Kind);

  assert(Kind - FirstTargetFixupKind < PPC::NumTargetFixupKinds &&
         "Invalid kind!");
  return (Endian == llvm::endianness::little
              ? InfosLE
              : InfosBE)[Kind - FirstTargetFixupKind];
}

void PPCAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
                               const MCValue &TargetVal, uint8_t *Data,
                               uint64_t Value, bool IsResolved) {
  // In PPC64 ELFv1, .quad .TOC.@tocbase in the .opd section is expected to
```
- **EN**: Implements helper routine(s) `isRelocation`, `getFixupKindInfo`, `applyFixup` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `isRelocation`, `getFixupKindInfo`, `applyFixup`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 222-249

```cpp
  // reference the null symbol.
  auto Target = TargetVal;
  if (Target.getSpecifier() == PPC::S_TOCBASE)
    Target.setAddSym(nullptr);
  if (IsResolved && shouldForceRelocation(Fixup, Target))
    IsResolved = false;
  if (!IsResolved)
    Asm->getWriter().recordRelocation(F, Fixup, Target, Value);

  MCFixupKind Kind = Fixup.getKind();
  if (mc::isRelocation(Kind))
    return;
  Value = adjustFixupValue(getContext(), Fixup, Kind, Value);
  if (!Value)
    return; // Doesn't change encoding.

  unsigned NumBytes = getFixupKindNumBytes(Kind);

  // For each byte of the fragment that the fixup touches, mask in the bits
  // from the fixup value. The Value has been "split up" into the appropriate
  // bitfields above.
  for (unsigned i = 0; i != NumBytes; ++i) {
    unsigned Idx = Endian == llvm::endianness::little ? i : (NumBytes - 1 - i);
    Data[i] |= uint8_t((Value >> (Idx * 8)) & 0xff);
  }
}

// FIXME: This should be in a separate file.
```
- **EN**: Implements helper routine(s) `getSpecifier`, `setAddSym`, `shouldForceRelocation` for this portion of the PowerPC backend MC layer support for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getSpecifier`, `setAddSym`, `shouldForceRelocation`。

### Lines 250-265

```cpp
namespace {

class ELFPPCAsmBackend : public PPCAsmBackend {
public:
  ELFPPCAsmBackend(const Target &T, const Triple &TT) : PPCAsmBackend(T, TT) {}

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    uint8_t OSABI = MCELFObjectTargetWriter::getOSABI(TT.getOS());
    bool Is64 = TT.isPPC64();
    return createPPCELFObjectWriter(Is64, OSABI);
  }

  std::optional<MCFixupKind> getFixupKind(StringRef Name) const override;
};
```
- **EN**: Declares a backend-facing type `ELFPPCAsmBackend`, `PPCAsmBackend`, `createObjectTargetWriter` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `ELFPPCAsmBackend`, `PPCAsmBackend`, `createObjectTargetWriter`，并勾勒出周边代码会依赖的接口或状态。

### Lines 266-284

```cpp
class XCOFFPPCAsmBackend : public PPCAsmBackend {
public:
  XCOFFPPCAsmBackend(const Target &T, const Triple &TT)
      : PPCAsmBackend(T, TT) {}

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override {
    return createPPCXCOFFObjectWriter(TT.isArch64Bit());
  }
};

} // end anonymous namespace

std::optional<MCFixupKind>
ELFPPCAsmBackend::getFixupKind(StringRef Name) const {
  if (TT.isOSBinFormatELF()) {
    unsigned Type;
    if (TT.isPPC64()) {
      Type = llvm::StringSwitch<unsigned>(Name)
```
- **EN**: Declares a backend-facing type `XCOFFPPCAsmBackend`, `PPCAsmBackend`, `createObjectTargetWriter` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XCOFFPPCAsmBackend`, `PPCAsmBackend`, `createObjectTargetWriter`，并勾勒出周边代码会依赖的接口或状态。

### Lines 285-294

```cpp
#define ELF_RELOC(X, Y) .Case(#X, Y)
#include "llvm/BinaryFormat/ELFRelocs/PowerPC64.def"
#undef ELF_RELOC
                 .Case("BFD_RELOC_NONE", ELF::R_PPC64_NONE)
                 .Case("BFD_RELOC_16", ELF::R_PPC64_ADDR16)
                 .Case("BFD_RELOC_32", ELF::R_PPC64_ADDR32)
                 .Case("BFD_RELOC_64", ELF::R_PPC64_ADDR64)
                 .Default(-1u);
    } else {
      Type = llvm::StringSwitch<unsigned>(Name)
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `ELF_RELOC`, `Case`, `Default`.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `ELF_RELOC`, `Case`, `Default`。

### Lines 295-318

```cpp
#define ELF_RELOC(X, Y) .Case(#X, Y)
#include "llvm/BinaryFormat/ELFRelocs/PowerPC.def"
#undef ELF_RELOC
                 .Case("BFD_RELOC_NONE", ELF::R_PPC_NONE)
                 .Case("BFD_RELOC_16", ELF::R_PPC_ADDR16)
                 .Case("BFD_RELOC_32", ELF::R_PPC_ADDR32)
                 .Default(-1u);
    }
    if (Type != -1u)
      return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
  }
  return std::nullopt;
}

MCAsmBackend *llvm::createPPCAsmBackend(const Target &T,
                                        const MCSubtargetInfo &STI,
                                        const MCRegisterInfo &MRI,
                                        const MCTargetOptions &Options) {
  const Triple &TT = STI.getTargetTriple();
  if (TT.isOSBinFormatXCOFF())
    return new XCOFFPPCAsmBackend(T, TT);

  return new ELFPPCAsmBackend(T, TT);
}
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCFixupKinds.h`
- `MCTargetDesc/PPCMCAsmInfo.h`
- `MCTargetDesc/PPCMCTargetDesc.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/BinaryFormat/MachO.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCELFObjectWriter.h`
- `llvm/MC/MCMachObjectWriter.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCSymbolELF.h`
- `llvm/MC/MCSymbolXCOFF.h`
- `llvm/MC/MCValue.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/BinaryFormat/ELFRelocs/PowerPC64.def`
- `llvm/BinaryFormat/ELFRelocs/PowerPC.def`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
