# MCAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCAsmBackend.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Target MC Assembly Backend.
  - **CN**: 实现汇编后端钩子，例如 fixup 应用、指令松弛决策以及与对象写出器的协同。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MCAsmBackend.cpp - Target MC Assembly Backend ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-17
```cpp

#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDXContainerWriter.h"
#include "llvm/MC/MCELFObjectWriter.h"
#include "llvm/MC/MCGOFFObjectWriter.h"
#include "llvm/MC/MCMachObjectWriter.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSPIRVObjectWriter.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDXContainerWriter.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDXContainerWriter.h`。

### Lines 18-24
```cpp
#include "llvm/MC/MCWasmObjectWriter.h"
#include "llvm/MC/MCWinCOFFObjectWriter.h"
#include "llvm/MC/MCXCOFFObjectWriter.h"
#include <cassert>
#include <cstddef>
#include <cstdint>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCWasmObjectWriter.h`, `llvm/MC/MCWinCOFFObjectWriter.h`, `llvm/MC/MCXCOFFObjectWriter.h`, `cassert`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCWasmObjectWriter.h`, `llvm/MC/MCWinCOFFObjectWriter.h`, `llvm/MC/MCXCOFFObjectWriter.h`, `cassert`。

### Lines 25-30
```cpp
using namespace llvm;

MCAsmBackend::~MCAsmBackend() = default;

MCContext &MCAsmBackend::getContext() const { return Asm->getContext(); }

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 31-40
```cpp
std::unique_ptr<MCObjectWriter>
MCAsmBackend::createObjectWriter(raw_pwrite_stream &OS) const {
  auto TW = createObjectTargetWriter();
  bool IsLE = Endian == llvm::endianness::little;
  switch (TW->getFormat()) {
  case Triple::MachO:
    return std::make_unique<MachObjectWriter>(
        cast<MCMachObjectTargetWriter>(std::move(TW)), OS, IsLE);
  case Triple::COFF:
    return createWinCOFFObjectWriter(
```
- **EN**: Implements logic around `createObjectWriter`, `createObjectTargetWriter`, `make_unique<MachObjectWriter>`, `cast<MCMachObjectTargetWriter>`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `createObjectWriter`, `createObjectTargetWriter`, `make_unique<MachObjectWriter>`, `cast<MCMachObjectTargetWriter>`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 41-50
```cpp
        cast<MCWinCOFFObjectTargetWriter>(std::move(TW)), OS);
  case Triple::ELF:
    return std::make_unique<ELFObjectWriter>(
        cast<MCELFObjectTargetWriter>(std::move(TW)), OS, IsLE);
  case Triple::SPIRV:
    return createSPIRVObjectWriter(
        cast<MCSPIRVObjectTargetWriter>(std::move(TW)), OS);
  case Triple::Wasm:
    return createWasmObjectWriter(cast<MCWasmObjectTargetWriter>(std::move(TW)),
                                  OS);
```
- **EN**: Implements logic around `cast<MCWinCOFFObjectTargetWriter>`, `make_unique<ELFObjectWriter>`, `cast<MCELFObjectTargetWriter>`, `createSPIRVObjectWriter`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `cast<MCWinCOFFObjectTargetWriter>`, `make_unique<ELFObjectWriter>`, `cast<MCELFObjectTargetWriter>`, `createSPIRVObjectWriter`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 51-60
```cpp
  case Triple::GOFF:
    return createGOFFObjectWriter(cast<MCGOFFObjectTargetWriter>(std::move(TW)),
                                  OS);
  case Triple::XCOFF:
    return createXCOFFObjectWriter(
        cast<MCXCOFFObjectTargetWriter>(std::move(TW)), OS);
  case Triple::DXContainer:
    return std::make_unique<DXContainerObjectWriter>(
        cast<MCDXContainerTargetWriter>(std::move(TW)), OS);
  default:
```
- **EN**: Implements logic around `createGOFFObjectWriter`, `createXCOFFObjectWriter`, `cast<MCXCOFFObjectTargetWriter>`, `make_unique<DXContainerObjectWriter>`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `createGOFFObjectWriter`, `createXCOFFObjectWriter`, `cast<MCXCOFFObjectTargetWriter>`, `make_unique<DXContainerObjectWriter>`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 61-70
```cpp
    llvm_unreachable("unexpected object format");
  }
}

std::unique_ptr<MCObjectWriter>
MCAsmBackend::createDwoObjectWriter(raw_pwrite_stream &OS,
                                    raw_pwrite_stream &DwoOS) const {
  auto TW = createObjectTargetWriter();
  switch (TW->getFormat()) {
  case Triple::COFF:
```
- **EN**: Implements logic around `llvm_unreachable`, `createDwoObjectWriter`, `createObjectTargetWriter`; this block drives emission, layout, or binary encoding behavior; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `llvm_unreachable`, `createDwoObjectWriter`, `createObjectTargetWriter` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 71-80
```cpp
    return createWinCOFFDwoObjectWriter(
        cast<MCWinCOFFObjectTargetWriter>(std::move(TW)), OS, DwoOS);
  case Triple::ELF:
    return std::make_unique<ELFObjectWriter>(
        cast<MCELFObjectTargetWriter>(std::move(TW)), OS, DwoOS,
        Endian == llvm::endianness::little);
  case Triple::Wasm:
    return createWasmDwoObjectWriter(
        cast<MCWasmObjectTargetWriter>(std::move(TW)), OS, DwoOS);
  default:
```
- **EN**: Implements logic around `createWinCOFFDwoObjectWriter`, `cast<MCWinCOFFObjectTargetWriter>`, `make_unique<ELFObjectWriter>`, `cast<MCELFObjectTargetWriter>`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `createWinCOFFDwoObjectWriter`, `cast<MCWinCOFFObjectTargetWriter>`, `make_unique<ELFObjectWriter>`, `cast<MCELFObjectTargetWriter>`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 81-88
```cpp
    report_fatal_error("dwo only supported with COFF, ELF, and Wasm");
  }
}

std::optional<MCFixupKind> MCAsmBackend::getFixupKind(StringRef Name) const {
  return std::nullopt;
}

```
- **EN**: Implements logic around `report_fatal_error`, `getFixupKind`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `report_fatal_error`, `getFixupKind` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 89-98
```cpp
MCFixupKindInfo MCAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  // clang-format off
  static const MCFixupKindInfo Builtins[] = {
      {"FK_NONE", 0, 0, 0},
      {"FK_Data_1", 0, 8, 0},
      {"FK_Data_2", 0, 16, 0},
      {"FK_Data_4", 0, 32, 0},
      {"FK_Data_8", 0, 64, 0},
      {"FK_Data_leb128", 0, 0, 0},
      {"FK_SecRel_1", 0, 8, 0},
```
- **EN**: Implements logic around `getFixupKindInfo`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getFixupKindInfo` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 99-104
```cpp
      {"FK_SecRel_2", 0, 16, 0},
      {"FK_SecRel_4", 0, 32, 0},
      {"FK_SecRel_8", 0, 64, 0},
  };
  // clang-format on

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 105-114
```cpp
  assert(size_t(Kind - FK_NONE) < std::size(Builtins) && "Unknown fixup kind");
  return Builtins[Kind - FK_NONE];
}

bool MCAsmBackend::fixupNeedsRelaxationAdvanced(const MCFragment &,
                                                const MCFixup &,
                                                const MCValue &, uint64_t,
                                                bool Resolved) const {
  return !Resolved;
}
```
- **EN**: Implements logic around `assert`, `fixupNeedsRelaxationAdvanced`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `assert`, `fixupNeedsRelaxationAdvanced` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 115-122
```cpp

void MCAsmBackend::maybeAddReloc(const MCFragment &F, const MCFixup &Fixup,
                                 const MCValue &Target, uint64_t &Value,
                                 bool IsResolved) {
  if (!IsResolved)
    Asm->getWriter().recordRelocation(F, Fixup, Target, Value);
}

```
- **EN**: Implements logic around `maybeAddReloc`, `getWriter`; this block handles relocation, fixup, or symbol-resolution work; drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `maybeAddReloc`, `getWriter` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，驱动输出、布局或二进制编码行为。

### Lines 123-129
```cpp
bool MCAsmBackend::isDarwinCanonicalPersonality(const MCSymbol *Sym) const {
  assert(getContext().isMachO());
  // Consider a NULL personality (ie., no personality encoding) to be canonical
  // because it's always at 0.
  if (!Sym)
    return true;

```
- **EN**: Implements logic around `isDarwinCanonicalPersonality`, `assert`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `isDarwinCanonicalPersonality`, `assert` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 130-136
```cpp
  StringRef name = Sym->getName();
  // XXX: We intentionally leave out "___gcc_personality_v0" because, despite
  // being system-defined like these two, it is not very commonly-used.
  // Reserving an empty slot for it seems silly.
  return name == "___gxx_personality_v0" || name == "___objc_personality_v0";
}

```
- **EN**: Implements logic around `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 137-142
```cpp
const MCSubtargetInfo *MCAsmBackend::getSubtargetInfo(const MCFragment &F) {
  const MCSubtargetInfo *STI = nullptr;
  STI = F.getSubtargetInfo();
  assert(!F.hasInstructions() || STI != nullptr);
  return STI;
}
```
- **EN**: Implements logic around `getSubtargetInfo`, `assert`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getSubtargetInfo`, `assert` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

## Key Concepts / 关键概念

- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDXContainerWriter.h`, `llvm/MC/MCELFObjectWriter.h`, `llvm/MC/MCGOFFObjectWriter.h`, `llvm/MC/MCMachObjectWriter.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCSPIRVObjectWriter.h`, `llvm/MC/MCWasmObjectWriter.h`, `llvm/MC/MCWinCOFFObjectWriter.h`, `llvm/MC/MCXCOFFObjectWriter.h` ... (+3 more)
- **LLVM subsystems / LLVM 子系统**: MC
