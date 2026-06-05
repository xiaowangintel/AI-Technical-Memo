# WebAssemblyWasmObjectWriter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyWasmObjectWriter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file handles Wasm-specific object emission, converting LLVM's internal fixups into the appropriate relocations.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/MCTargetDesc/WebAssemblyWasmObjectWriter.cpp`，主要负责 WebAssembly 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyWasmObjectWriter.cpp - WebAssembly Wasm Writer ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file handles Wasm-specific object emission, converting LLVM's
/// internal fixups into the appropriate relocations.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyFixupKinds.h"
#include "MCTargetDesc/WebAssemblyMCAsmInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/MC/MCAsmBackend.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 20-26

```cpp
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCSectionWasm.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/MCWasmObjectWriter.h"
#include "llvm/Support/Casting.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 27-64

```cpp
#include "llvm/Support/ErrorHandling.h"

using namespace llvm;

namespace {
class WebAssemblyWasmObjectWriter final : public MCWasmObjectTargetWriter {
public:
  explicit WebAssemblyWasmObjectWriter(bool Is64Bit, bool IsEmscripten);

private:
  unsigned getRelocType(const MCValue &Target, const MCFixup &Fixup,
                        const MCSectionWasm &FixupSection,
                        bool IsLocRel) const override;
};
} // end anonymous namespace

WebAssemblyWasmObjectWriter::WebAssemblyWasmObjectWriter(bool Is64Bit,
                                                         bool IsEmscripten)
    : MCWasmObjectTargetWriter(Is64Bit, IsEmscripten) {}

static const MCSection *getTargetSection(const MCExpr *Expr) {
  if (auto SyExp = dyn_cast<MCSymbolRefExpr>(Expr)) {
    if (SyExp->getSymbol().isInSection())
      return &SyExp->getSymbol().getSection();
    return nullptr;
  }

  if (auto BinOp = dyn_cast<MCBinaryExpr>(Expr)) {
    auto SectionLHS = getTargetSection(BinOp->getLHS());
    auto SectionRHS = getTargetSection(BinOp->getRHS());
    return SectionLHS == SectionRHS ? nullptr : SectionLHS;
  }

  if (auto UnOp = dyn_cast<MCUnaryExpr>(Expr))
    return getTargetSection(UnOp->getSubExpr());

  return nullptr;
}
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 65-102

```cpp

unsigned WebAssemblyWasmObjectWriter::getRelocType(
    const MCValue &Target, const MCFixup &Fixup,
    const MCSectionWasm &FixupSection, bool IsLocRel) const {
  auto &SymA = static_cast<const MCSymbolWasm &>(*Target.getAddSym());
  auto Spec = WebAssembly::Specifier(Target.getSpecifier());
  switch (Spec) {
  case WebAssembly::S_GOT:
    SymA.setUsedInGOT();
    return wasm::R_WASM_GLOBAL_INDEX_LEB;
  case WebAssembly::S_GOT_TLS:
    SymA.setUsedInGOT();
    SymA.setTLS();
    return wasm::R_WASM_GLOBAL_INDEX_LEB;
  case WebAssembly::S_TBREL:
    assert(SymA.isFunction());
    return is64Bit() ? wasm::R_WASM_TABLE_INDEX_REL_SLEB64
                     : wasm::R_WASM_TABLE_INDEX_REL_SLEB;
  case WebAssembly::S_TLSREL:
    SymA.setTLS();
    return is64Bit() ? wasm::R_WASM_MEMORY_ADDR_TLS_SLEB64
                     : wasm::R_WASM_MEMORY_ADDR_TLS_SLEB;
  case WebAssembly::S_MBREL:
    assert(SymA.isData());
    return is64Bit() ? wasm::R_WASM_MEMORY_ADDR_REL_SLEB64
                     : wasm::R_WASM_MEMORY_ADDR_REL_SLEB;
  case WebAssembly::S_TYPEINDEX:
    return wasm::R_WASM_TYPE_INDEX_LEB;
  case WebAssembly::S_None:
    break;
  case WebAssembly::S_FUNCINDEX:
    return wasm::R_WASM_FUNCTION_INDEX_I32;
  }

  switch (unsigned(Fixup.getKind())) {
  case WebAssembly::fixup_sleb128_i32:
    if (SymA.isFunction())
      return wasm::R_WASM_TABLE_INDEX_SLEB;
```
- **EN**: Implements helper routine(s) `getRelocType`, `getAddSym`, `Specifier` for this portion of the WebAssembly backend MC layer support for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `getRelocType`, `getAddSym`, `Specifier`。

### Lines 103-140

```cpp
    return wasm::R_WASM_MEMORY_ADDR_SLEB;
  case WebAssembly::fixup_sleb128_i64:
    if (SymA.isFunction())
      return wasm::R_WASM_TABLE_INDEX_SLEB64;
    return wasm::R_WASM_MEMORY_ADDR_SLEB64;
  case WebAssembly::fixup_uleb128_i32:
    if (SymA.isGlobal())
      return wasm::R_WASM_GLOBAL_INDEX_LEB;
    if (SymA.isFunction())
      return wasm::R_WASM_FUNCTION_INDEX_LEB;
    if (SymA.isTag())
      return wasm::R_WASM_TAG_INDEX_LEB;
    if (SymA.isTable())
      return wasm::R_WASM_TABLE_NUMBER_LEB;
    return wasm::R_WASM_MEMORY_ADDR_LEB;
  case WebAssembly::fixup_uleb128_i64:
    assert(SymA.isData());
    return wasm::R_WASM_MEMORY_ADDR_LEB64;
  case FK_Data_4:
    if (SymA.isFunction()) {
      if (FixupSection.isMetadata())
        return wasm::R_WASM_FUNCTION_OFFSET_I32;
      assert(FixupSection.isWasmData());
      return wasm::R_WASM_TABLE_INDEX_I32;
    }
    if (SymA.isGlobal())
      return wasm::R_WASM_GLOBAL_INDEX_I32;
    if (auto Section = static_cast<const MCSectionWasm *>(
            getTargetSection(Fixup.getValue()))) {
      if (Section->isText())
        return wasm::R_WASM_FUNCTION_OFFSET_I32;
      else if (!Section->isWasmData())
        return wasm::R_WASM_SECTION_OFFSET_I32;
    }
    return IsLocRel ? wasm::R_WASM_MEMORY_ADDR_LOCREL_I32
                    : wasm::R_WASM_MEMORY_ADDR_I32;
  case FK_Data_8:
    if (SymA.isFunction()) {
```
- **EN**: Implements helper routine(s) `isFunction`, `isGlobal`, `isTag` for this portion of the WebAssembly backend MC layer support for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `isFunction`, `isGlobal`, `isTag`。

### Lines 141-164

```cpp
      if (FixupSection.isMetadata())
        return wasm::R_WASM_FUNCTION_OFFSET_I64;
      return wasm::R_WASM_TABLE_INDEX_I64;
    }
    if (SymA.isGlobal())
      llvm_unreachable("unimplemented R_WASM_GLOBAL_INDEX_I64");
    if (auto Section = static_cast<const MCSectionWasm *>(
            getTargetSection(Fixup.getValue()))) {
      if (Section->isText())
        return wasm::R_WASM_FUNCTION_OFFSET_I64;
      else if (!Section->isWasmData())
        llvm_unreachable("unimplemented R_WASM_SECTION_OFFSET_I64");
    }
    assert(SymA.isData());
    return wasm::R_WASM_MEMORY_ADDR_I64;
  default:
    llvm_unreachable("unimplemented fixup kind");
  }
}

std::unique_ptr<MCObjectTargetWriter>
llvm::createWebAssemblyWasmObjectWriter(bool Is64Bit, bool IsEmscripten) {
  return std::make_unique<WebAssemblyWasmObjectWriter>(Is64Bit, IsEmscripten);
}
```
- **EN**: Implements helper routine(s) `isMetadata`, `isGlobal`, `llvm_unreachable` for this portion of the WebAssembly backend MC layer support for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分该后端的 MC 层支持所需的辅助例程 `isMetadata`, `isGlobal`, `llvm_unreachable`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyFixupKinds.h`
- `MCTargetDesc/WebAssemblyMCAsmInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `llvm/BinaryFormat/Wasm.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCFixup.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/MC/MCSectionWasm.h`
- `llvm/MC/MCSymbolWasm.h`
- `llvm/MC/MCValue.h`
- `llvm/MC/MCWasmObjectWriter.h`
- `llvm/Support/Casting.h`
- `llvm/Support/ErrorHandling.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
