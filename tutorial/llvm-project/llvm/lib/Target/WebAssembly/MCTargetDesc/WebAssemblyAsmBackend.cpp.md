# WebAssemblyAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyAsmBackend.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the WebAssemblyAsmBackend class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyAsmBackend.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyAsmBackend.cpp - WebAssembly Assembler Backend ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the WebAssemblyAsmBackend class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyFixupKinds.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 19-28

```cpp
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/MCWasmObjectWriter.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 29-55

```cpp
namespace {

class WebAssemblyAsmBackend final : public MCAsmBackend {
  bool Is64Bit;
  bool IsEmscripten;

public:
  explicit WebAssemblyAsmBackend(bool Is64Bit, bool IsEmscripten)
      : MCAsmBackend(llvm::endianness::little), Is64Bit(Is64Bit),
        IsEmscripten(IsEmscripten) {}

  std::optional<MCFixupKind> getFixupKind(StringRef Name) const override;
  MCFixupKindInfo getFixupKindInfo(MCFixupKind Kind) const override;

  void applyFixup(const MCFragment &, const MCFixup &, const MCValue &Target,
                  uint8_t *Data, uint64_t Value, bool) override;

  std::unique_ptr<MCObjectTargetWriter>
  createObjectTargetWriter() const override;

  bool writeNopData(raw_ostream &OS, uint64_t Count,
                    const MCSubtargetInfo *STI) const override;
};

std::optional<MCFixupKind>
WebAssemblyAsmBackend::getFixupKind(StringRef Name) const {
  unsigned Type = llvm::StringSwitch<unsigned>(Name)
```
- **EN**: Declares a backend-facing type `WebAssemblyAsmBackend`, `MCAsmBackend`, `Is64Bit` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `WebAssemblyAsmBackend`, `MCAsmBackend`, `Is64Bit`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 56-93

```cpp
#define WASM_RELOC(NAME, ID) .Case(#NAME, ID)
#include "llvm/BinaryFormat/WasmRelocs.def"
#undef WASM_RELOC
                      .Default(-1u);
  if (Type != -1u)
    return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
  return std::nullopt;
}

MCFixupKindInfo
WebAssemblyAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  const static MCFixupKindInfo Infos[WebAssembly::NumTargetFixupKinds] = {
      // This table *must* be in the order that the fixup_* kinds are defined in
      // WebAssemblyFixupKinds.h.
      //
      // Name                     Offset (bits) Size (bits)     Flags
      {"fixup_sleb128_i32", 0, 5 * 8, 0},
      {"fixup_sleb128_i64", 0, 10 * 8, 0},
      {"fixup_uleb128_i32", 0, 5 * 8, 0},
      {"fixup_uleb128_i64", 0, 10 * 8, 0},
  };

  // Fixup kinds from raw relocation types and .reloc directives force
  // relocations and do not use these fields.
  if (mc::isRelocation(Kind))
    return {};

  if (Kind < FirstTargetFixupKind)
    return MCAsmBackend::getFixupKindInfo(Kind);

  assert(unsigned(Kind - FirstTargetFixupKind) <
             WebAssembly::NumTargetFixupKinds &&
         "Invalid kind!");
  return Infos[Kind - FirstTargetFixupKind];
}

bool WebAssemblyAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
                                         const MCSubtargetInfo *STI) const {
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 94-131

```cpp
  for (uint64_t I = 0; I < Count; ++I)
    OS << char(WebAssembly::Nop);

  return true;
}

void WebAssemblyAsmBackend::applyFixup(const MCFragment &F,
                                       const MCFixup &Fixup,
                                       const MCValue &Target, uint8_t *Data,
                                       uint64_t Value, bool IsResolved) {
  if (!IsResolved)
    Asm->getWriter().recordRelocation(F, Fixup, Target, Value);

  MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());
  assert(Info.Flags == 0 && "WebAssembly does not use MCFixupKindInfo flags");

  unsigned NumBytes = alignTo(Info.TargetSize, 8) / 8;
  if (Value == 0)
    return; // Doesn't change encoding.

  // Shift the value into position.
  Value <<= Info.TargetOffset;

  assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
         "Invalid fixup offset!");

  // For each byte of the fragment that the fixup touches, mask in the
  // bits from the fixup value.
  for (unsigned I = 0; I != NumBytes; ++I)
    Data[I] |= uint8_t((Value >> (I * 8)) & 0xff);
}

std::unique_ptr<MCObjectTargetWriter>
WebAssemblyAsmBackend::createObjectTargetWriter() const {
  return createWebAssemblyWasmObjectWriter(Is64Bit, IsEmscripten);
}

} // end anonymous namespace
```
- **EN**: Implements helper routine(s) `char`, `applyFixup`, `getWriter` for this portion of the WebAssembly backend MC layer support for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `char`, `applyFixup`, `getWriter`。

### Lines 132-135

```cpp

MCAsmBackend *llvm::createWebAssemblyAsmBackend(const Triple &TT) {
  return new WebAssemblyAsmBackend(TT.isArch64Bit(), TT.isOSEmscripten());
}
```
- **EN**: Implements helper routine(s) `createWebAssemblyAsmBackend`, `WebAssemblyAsmBackend`, `isArch64Bit` for this portion of the WebAssembly backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `createWebAssemblyAsmBackend`, `WebAssemblyAsmBackend`, `isArch64Bit`。 相关逻辑会与 LLVM 的 MC 层交互。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Subtarget features / 子目标特性
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyFixupKinds.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `llvm/ADT/StringSwitch.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/MCSymbol.h`
- `llvm/MC/MCValue.h`
- `llvm/MC/MCWasmObjectWriter.h`
- `llvm/Support/raw_ostream.h`
- `llvm/BinaryFormat/WasmRelocs.def`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
