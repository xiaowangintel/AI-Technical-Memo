# ELF_aarch64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELF_aarch64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker implementation for ELF/aarch64.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===----- ELF_aarch64.cpp - JIT linker implementation for ELF/aarch64 ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// ELF/aarch64 jit-link implementation.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 13-24
```cpp
#include "llvm/ExecutionEngine/JITLink/ELF_aarch64.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h"
#include "llvm/ExecutionEngine/JITLink/aarch64.h"
#include "llvm/Object/ELFObjectFile.h"
#include "llvm/Support/Endian.h"

#include "DefineExternalSectionStartAndEndSymbols.h"
#include "EHFrameSupportImpl.h"
#include "ELFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/ELF_aarch64.h`, `llvm/BinaryFormat/ELF.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/ELF_aarch64.h`, `llvm/BinaryFormat/ELF.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`。

### Lines 25-36
```cpp
#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

namespace {

constexpr StringRef ELFGOTSymbolName = "_GLOBAL_OFFSET_TABLE_";

class ELFJITLinker_aarch64 : public JITLinker<ELFJITLinker_aarch64> {
  friend class JITLinker<ELFJITLinker_aarch64>;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 37-49
```cpp
public:
  ELFJITLinker_aarch64(std::unique_ptr<JITLinkContext> Ctx,
                       std::unique_ptr<LinkGraph> G,
                       PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {
    if (shouldAddDefaultTargetPasses(getGraph().getTargetTriple()))
      getPassConfig().PostAllocationPasses.push_back(
          [this](LinkGraph &G) { return getOrCreateGOTSymbol(G); });
  }

private:
  Symbol *GOTSymbol = nullptr;

```
- **EN**: Implements logic around `ELFJITLinker_aarch64`, `JITLinker`, `getPassConfig`, `getOrCreateGOTSymbol`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `ELFJITLinker_aarch64`, `JITLinker`, `getPassConfig`, `getOrCreateGOTSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 50-69
```cpp
  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return aarch64::applyFixup(G, B, E, GOTSymbol);
  }

  Error getOrCreateGOTSymbol(LinkGraph &G) {
    auto InteredGOTSymbolName =
        G.getSymbolStringPool()->intern(ELFGOTSymbolName);

    auto DefineExternalGOTSymbolIfPresent =
        createDefineExternalSectionStartAndEndSymbolsPass(
            [&](LinkGraph &LG, Symbol &Sym) -> SectionRangeSymbolDesc {
              if (*Sym.getName() == ELFGOTSymbolName)
                if (auto *GOTSection = G.findSectionByName(
                        aarch64::GOTTableManager::getSectionName())) {
                  GOTSymbol = &Sym;
                  return {*GOTSection, true};
                }
              return {};
            });

```
- **EN**: Implements logic around `applyFixup`, `getOrCreateGOTSymbol`, `getSymbolStringPool`, `createDefineExternalSectionStartAndEndSymbolsPass`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `applyFixup`, `getOrCreateGOTSymbol`, `getSymbolStringPool`, `createDefineExternalSectionStartAndEndSymbolsPass`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 70-84
```cpp
    // Try to attach _GLOBAL_OFFSET_TABLE_ to the GOT if it's defined as an
    // external.
    if (auto Err = DefineExternalGOTSymbolIfPresent(G))
      return Err;

    // If we succeeded then we're done.
    if (GOTSymbol)
      return Error::success();

    // Otherwise look for a GOT section: If it already has a start symbol we'll
    // record it, otherwise we'll create our own.
    // If there's a GOT section but we didn't find an external GOT symbol...
    if (auto *GOTSection =
            G.findSectionByName(aarch64::GOTTableManager::getSectionName())) {

```
- **EN**: Implements logic around `success`, `findSectionByName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `findSectionByName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 85-104
```cpp
      // Check for an existing defined symbol.
      for (auto *Sym : GOTSection->symbols())
        if (Sym->getName() == InteredGOTSymbolName) {
          GOTSymbol = Sym;
          return Error::success();
        }

      // If there's no defined symbol then create one.
      SectionRange SR(*GOTSection);
      if (SR.empty())
        GOTSymbol =
            // FIXME: we should only do this once
            &G.addAbsoluteSymbol(InteredGOTSymbolName, orc::ExecutorAddr(), 0,
                                 Linkage::Strong, Scope::Local, true);
      else
        GOTSymbol =
            &G.addDefinedSymbol(*SR.getFirstBlock(), 0, InteredGOTSymbolName, 0,
                                Linkage::Strong, Scope::Local, false, true);
    }

```
- **EN**: Implements logic around `success`, `SR`, `addAbsoluteSymbol`, `addDefinedSymbol`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `SR`, `addAbsoluteSymbol`, `addDefinedSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 105-120
```cpp
    // If we still haven't found a GOT symbol then double check the externals.
    // We may have a GOT-relative reference but no GOT section, in which case
    // we just need to point the GOT symbol at some address in this graph.
    if (!GOTSymbol) {
      for (auto *Sym : G.external_symbols()) {
        if (*Sym->getName() == ELFGOTSymbolName) {
          auto Blocks = G.blocks();
          if (!Blocks.empty()) {
            G.makeAbsolute(*Sym, (*Blocks.begin())->getAddress());
            GOTSymbol = Sym;
            break;
          }
        }
      }
    }

```
- **EN**: Implements logic around `blocks`, `makeAbsolute`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `blocks`, `makeAbsolute` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 121-144
```cpp
    return Error::success();
  }
};

template <typename ELFT>
class ELFLinkGraphBuilder_aarch64 : public ELFLinkGraphBuilder<ELFT> {
private:
  enum ELFAArch64RelocationKind : Edge::Kind {
    ELFCall26 = Edge::FirstRelocation,
    ELFLdrLo19,
    ELFAdrLo21,
    ELFAdrPage21,
    ELFAddAbs12,
    ELFLdSt8Abs12,
    ELFLdSt16Abs12,
    ELFLdSt32Abs12,
    ELFLdSt64Abs12,
    ELFLdSt128Abs12,
    ELFMovwAbsG0,
    ELFMovwAbsG1,
    ELFMovwAbsG2,
    ELFMovwAbsG3,
    ELFTstBr14,
    ELFCondBr19,
```
- **EN**: Introduces declarations for `ELFLinkGraphBuilder_aarch64`, `ELFAArch64RelocationKind`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilder_aarch64`, `ELFAArch64RelocationKind` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 145-157
```cpp
    ELFAbs32,
    ELFAbs64,
    ELFPrel32,
    ELFPrel64,
    ELFAdrGOTPage21,
    ELFLd64GOTLo12,
    ELFLd64GOTPAGELo15,
    ELFTLSDescAdrPage21,
    ELFTLSDescAddLo12,
    ELFTLSDescLd64Lo12,
    ELFTLSDescCall,
  };

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 158-181
```cpp
  static Expected<ELFAArch64RelocationKind>
  getRelocationKind(const uint32_t Type) {
    using namespace aarch64;
    switch (Type) {
    case ELF::R_AARCH64_CALL26:
    case ELF::R_AARCH64_JUMP26:
      return ELFCall26;
    case ELF::R_AARCH64_LD_PREL_LO19:
      return ELFLdrLo19;
    case ELF::R_AARCH64_ADR_PREL_LO21:
      return ELFAdrLo21;
    case ELF::R_AARCH64_ADR_PREL_PG_HI21:
      return ELFAdrPage21;
    case ELF::R_AARCH64_ADD_ABS_LO12_NC:
      return ELFAddAbs12;
    case ELF::R_AARCH64_LDST8_ABS_LO12_NC:
      return ELFLdSt8Abs12;
    case ELF::R_AARCH64_LDST16_ABS_LO12_NC:
      return ELFLdSt16Abs12;
    case ELF::R_AARCH64_LDST32_ABS_LO12_NC:
      return ELFLdSt32Abs12;
    case ELF::R_AARCH64_LDST64_ABS_LO12_NC:
      return ELFLdSt64Abs12;
    case ELF::R_AARCH64_LDST128_ABS_LO12_NC:
```
- **EN**: Introduces declarations for `aarch64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `aarch64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 182-205
```cpp
      return ELFLdSt128Abs12;
    case ELF::R_AARCH64_MOVW_UABS_G0_NC:
      return ELFMovwAbsG0;
    case ELF::R_AARCH64_MOVW_UABS_G1_NC:
      return ELFMovwAbsG1;
    case ELF::R_AARCH64_MOVW_UABS_G2_NC:
      return ELFMovwAbsG2;
    case ELF::R_AARCH64_MOVW_UABS_G3:
      return ELFMovwAbsG3;
    case ELF::R_AARCH64_TSTBR14:
      return ELFTstBr14;
    case ELF::R_AARCH64_CONDBR19:
      return ELFCondBr19;
    case ELF::R_AARCH64_ABS32:
      return ELFAbs32;
    case ELF::R_AARCH64_ABS64:
      return ELFAbs64;
    case ELF::R_AARCH64_PREL32:
      return ELFPrel32;
    case ELF::R_AARCH64_PREL64:
      return ELFPrel64;
    case ELF::R_AARCH64_ADR_GOT_PAGE:
      return ELFAdrGOTPage21;
    case ELF::R_AARCH64_LD64_GOT_LO12_NC:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 206-218
```cpp
      return ELFLd64GOTLo12;
    case ELF::R_AARCH64_LD64_GOTPAGE_LO15:
      return ELFLd64GOTPAGELo15;
    case ELF::R_AARCH64_TLSDESC_ADR_PAGE21:
      return ELFTLSDescAdrPage21;
    case ELF::R_AARCH64_TLSDESC_ADD_LO12:
      return ELFTLSDescAddLo12;
    case ELF::R_AARCH64_TLSDESC_LD64_LO12:
      return ELFTLSDescLd64Lo12;
    case ELF::R_AARCH64_TLSDESC_CALL:
      return ELFTLSDescCall;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 219-233
```cpp
    return make_error<JITLinkError>(
        "Unsupported aarch64 relocation:" + formatv("{0:d}: ", Type) +
        object::getELFRelocationTypeName(ELF::EM_AARCH64, Type));
  }

  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

    using Base = ELFLinkGraphBuilder<ELFT>;
    using Self = ELFLinkGraphBuilder_aarch64<ELFT>;
    for (const auto &RelSect : Base::Sections)
      if (Error Err = Base::forEachRelaRelocation(RelSect, this,
                                                  &Self::addSingleRelocation))
        return Err;

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`, `getELFRelocationTypeName`, `addRelocations`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv`, `getELFRelocationTypeName`, `addRelocations` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 234-247
```cpp
    return Error::success();
  }

  Error addSingleRelocation(const typename ELFT::Rela &Rel,
                            const typename ELFT::Shdr &FixupSect,
                            Block &BlockToFix) {
    using support::ulittle32_t;
    using Base = ELFLinkGraphBuilder<ELFT>;

    uint32_t SymbolIndex = Rel.getSymbol(false);
    auto ObjSymbol = Base::Obj.getRelocationSymbol(Rel, Base::SymTabSec);
    if (!ObjSymbol)
      return ObjSymbol.takeError();

```
- **EN**: Implements logic around `success`, `addSingleRelocation`, `getSymbol`, `getRelocationSymbol`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `addSingleRelocation`, `getSymbol`, `getRelocationSymbol`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 248-261
```cpp
    Symbol *GraphSymbol = Base::getGraphSymbol(SymbolIndex);
    if (!GraphSymbol)
      return make_error<StringError>(
          formatv("Could not find symbol at given index, did you add it to "
                  "JITSymbolTable? index: {0}, shndx: {1} Size of table: {2}",
                  SymbolIndex, (*ObjSymbol)->st_shndx,
                  Base::GraphSymbols.size()),
          inconvertibleErrorCode());

    uint32_t Type = Rel.getType(false);
    Expected<ELFAArch64RelocationKind> RelocKind = getRelocationKind(Type);
    if (!RelocKind)
      return RelocKind.takeError();

```
- **EN**: Implements logic around `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 4 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 4 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 262-283
```cpp
    int64_t Addend = Rel.r_addend;
    orc::ExecutorAddr FixupAddress =
        orc::ExecutorAddr(FixupSect.sh_addr) + Rel.r_offset;
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();

    // Get a pointer to the fixup content.
    const void *FixupContent = BlockToFix.getContent().data() +
                               (FixupAddress - BlockToFix.getAddress());

    Edge::Kind Kind = Edge::Invalid;

    switch (*RelocKind) {
    case ELFCall26: {
      Kind = aarch64::Branch26PCRel;
      break;
    }
    case ELFLdrLo19: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLDRLiteral(Instr))
        return make_error<JITLinkError>(
            "R_AARCH64_LDR_PREL_LO19 target is not an LDR Literal instruction");

```
- **EN**: Implements logic around `ExecutorAddr`, `getAddress`, `getContent`, `make_error<JITLinkError>`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `ExecutorAddr`, `getAddress`, `getContent`, `make_error<JITLinkError>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 284-307
```cpp
      Kind = aarch64::LDRLiteral19;
      break;
    }
    case ELFAdrLo21: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isADR(Instr))
        return make_error<JITLinkError>(
            "R_AARCH64_ADR_PREL_LO21 target is not an ADR instruction");

      Kind = aarch64::ADRLiteral21;
      break;
    }
    case ELFAdrPage21: {
      Kind = aarch64::Page21;
      break;
    }
    case ELFAddAbs12: {
      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFLdSt8Abs12: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLoadStoreImm12(Instr) ||
          aarch64::getPageOffset12Shift(Instr) != 0)
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getPageOffset12Shift`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getPageOffset12Shift` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 308-322
```cpp
        return make_error<JITLinkError>(
            "R_AARCH64_LDST8_ABS_LO12_NC target is not a "
            "LDRB/STRB (imm12) instruction");

      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFLdSt16Abs12: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLoadStoreImm12(Instr) ||
          aarch64::getPageOffset12Shift(Instr) != 1)
        return make_error<JITLinkError>(
            "R_AARCH64_LDST16_ABS_LO12_NC target is not a "
            "LDRH/STRH (imm12) instruction");

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `STRB`, `getPageOffset12Shift`, `STRH`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `STRB`, `getPageOffset12Shift`, `STRH` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 323-344
```cpp
      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFLdSt32Abs12: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLoadStoreImm12(Instr) ||
          aarch64::getPageOffset12Shift(Instr) != 2)
        return make_error<JITLinkError>(
            "R_AARCH64_LDST32_ABS_LO12_NC target is not a "
            "LDR/STR (imm12, 32 bit) instruction");

      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFLdSt64Abs12: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLoadStoreImm12(Instr) ||
          aarch64::getPageOffset12Shift(Instr) != 3)
        return make_error<JITLinkError>(
            "R_AARCH64_LDST64_ABS_LO12_NC target is not a "
            "LDR/STR (imm12, 64 bit) instruction");

```
- **EN**: Implements logic around `getPageOffset12Shift`, `make_error<JITLinkError>`, `STR`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getPageOffset12Shift`, `make_error<JITLinkError>`, `STR` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 345-366
```cpp
      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFLdSt128Abs12: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isLoadStoreImm12(Instr) ||
          aarch64::getPageOffset12Shift(Instr) != 4)
        return make_error<JITLinkError>(
            "R_AARCH64_LDST128_ABS_LO12_NC target is not a "
            "LDR/STR (imm12, 128 bit) instruction");

      Kind = aarch64::PageOffset12;
      break;
    }
    case ELFMovwAbsG0: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isMoveWideImm16(Instr) ||
          aarch64::getMoveWide16Shift(Instr) != 0)
        return make_error<JITLinkError>(
            "R_AARCH64_MOVW_UABS_G0_NC target is not a "
            "MOVK/MOVZ (imm16, LSL #0) instruction");

```
- **EN**: Implements logic around `getPageOffset12Shift`, `make_error<JITLinkError>`, `STR`, `getMoveWide16Shift`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getPageOffset12Shift`, `make_error<JITLinkError>`, `STR`, `getMoveWide16Shift`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 367-388
```cpp
      Kind = aarch64::MoveWide16;
      break;
    }
    case ELFMovwAbsG1: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isMoveWideImm16(Instr) ||
          aarch64::getMoveWide16Shift(Instr) != 16)
        return make_error<JITLinkError>(
            "R_AARCH64_MOVW_UABS_G1_NC target is not a "
            "MOVK/MOVZ (imm16, LSL #16) instruction");

      Kind = aarch64::MoveWide16;
      break;
    }
    case ELFMovwAbsG2: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isMoveWideImm16(Instr) ||
          aarch64::getMoveWide16Shift(Instr) != 32)
        return make_error<JITLinkError>(
            "R_AARCH64_MOVW_UABS_G2_NC target is not a "
            "MOVK/MOVZ (imm16, LSL #32) instruction");

```
- **EN**: Implements logic around `getMoveWide16Shift`, `make_error<JITLinkError>`, `MOVZ`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getMoveWide16Shift`, `make_error<JITLinkError>`, `MOVZ` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 389-408
```cpp
      Kind = aarch64::MoveWide16;
      break;
    }
    case ELFMovwAbsG3: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isMoveWideImm16(Instr) ||
          aarch64::getMoveWide16Shift(Instr) != 48)
        return make_error<JITLinkError>(
            "R_AARCH64_MOVW_UABS_G3 target is not a "
            "MOVK/MOVZ (imm16, LSL #48) instruction");

      Kind = aarch64::MoveWide16;
      break;
    }
    case ELFTstBr14: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isTestAndBranchImm14(Instr))
        return make_error<JITLinkError>("R_AARCH64_TSTBR14 target is not a "
                                        "test and branch instruction");

```
- **EN**: Implements logic around `getMoveWide16Shift`, `make_error<JITLinkError>`, `MOVZ`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getMoveWide16Shift`, `make_error<JITLinkError>`, `MOVZ` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 409-432
```cpp
      Kind = aarch64::TestAndBranch14PCRel;
      break;
    }
    case ELFCondBr19: {
      uint32_t Instr = *(const ulittle32_t *)FixupContent;
      if (!aarch64::isCondBranchImm19(Instr) &&
          !aarch64::isCompAndBranchImm19(Instr))
        return make_error<JITLinkError>("R_AARCH64_CONDBR19 target is not a "
                                        "conditional branch instruction");

      Kind = aarch64::CondBranch19PCRel;
      break;
    }
    case ELFAbs32: {
      Kind = aarch64::Pointer32;
      break;
    }
    case ELFAbs64: {
      Kind = aarch64::Pointer64;
      break;
    }
    case ELFPrel32: {
      Kind = aarch64::Delta32;
      break;
```
- **EN**: Implements logic around `isCompAndBranchImm19`, `make_error<JITLinkError>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `isCompAndBranchImm19`, `make_error<JITLinkError>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 433-456
```cpp
    }
    case ELFPrel64: {
      Kind = aarch64::Delta64;
      break;
    }
    case ELFAdrGOTPage21: {
      Kind = aarch64::RequestGOTAndTransformToPage21;
      break;
    }
    case ELFLd64GOTLo12: {
      Kind = aarch64::RequestGOTAndTransformToPageOffset12;
      break;
    }
    case ELFLd64GOTPAGELo15: {
      Kind = aarch64::RequestGOTAndTransformToPageOffset15;
      break;
    }
    case ELFTLSDescAdrPage21: {
      Kind = aarch64::RequestTLSDescEntryAndTransformToPage21;
      break;
    }
    case ELFTLSDescAddLo12:
    case ELFTLSDescLd64Lo12: {
      Kind = aarch64::RequestTLSDescEntryAndTransformToPageOffset12;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 457-470
```cpp
      break;
    }
    case ELFTLSDescCall: {
      return Error::success();
    }
    };

    Edge GE(Kind, Offset, *GraphSymbol, Addend);
    LLVM_DEBUG({
      dbgs() << "    ";
      printEdge(dbgs(), BlockToFix, GE, aarch64::getEdgeKindName(Kind));
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `success`, `GE`, `dbgs`, `printEdge`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `GE`, `dbgs`, `printEdge` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 471-494
```cpp
    BlockToFix.addEdge(std::move(GE));

    return Error::success();
  }

  /// Return the string name of the given ELF aarch64 edge kind.
  const char *getELFAArch64RelocationKindName(Edge::Kind R) {
    switch (R) {
    case ELFCall26:
      return "ELFCall26";
    case ELFAdrPage21:
      return "ELFAdrPage21";
    case ELFAddAbs12:
      return "ELFAddAbs12";
    case ELFLdSt8Abs12:
      return "ELFLdSt8Abs12";
    case ELFLdSt16Abs12:
      return "ELFLdSt16Abs12";
    case ELFLdSt32Abs12:
      return "ELFLdSt32Abs12";
    case ELFLdSt64Abs12:
      return "ELFLdSt64Abs12";
    case ELFLdSt128Abs12:
      return "ELFLdSt128Abs12";
```
- **EN**: Implements logic around `addEdge`, `success`, `getELFAArch64RelocationKindName`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `addEdge`, `success`, `getELFAArch64RelocationKindName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 495-518
```cpp
    case ELFMovwAbsG0:
      return "ELFMovwAbsG0";
    case ELFMovwAbsG1:
      return "ELFMovwAbsG1";
    case ELFMovwAbsG2:
      return "ELFMovwAbsG2";
    case ELFMovwAbsG3:
      return "ELFMovwAbsG3";
    case ELFAbs32:
      return "ELFAbs32";
    case ELFAbs64:
      return "ELFAbs64";
    case ELFPrel32:
      return "ELFPrel32";
    case ELFPrel64:
      return "ELFPrel64";
    case ELFAdrGOTPage21:
      return "ELFAdrGOTPage21";
    case ELFLd64GOTLo12:
      return "ELFLd64GOTLo12";
    case ELFLd64GOTPAGELo15:
      return "ELFLd64GOTPAGELo15";
    case ELFTLSDescAdrPage21:
      return "ELFTLSDescAdrPage21";
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 519-535
```cpp
    case ELFTLSDescAddLo12:
      return "ELFTLSDescAddLo12";
    case ELFTLSDescLd64Lo12:
      return "ELFTLSDescLd64Lo12";
    case ELFTLSDescCall:
      return "ELFTLSDescCall";
    default:
      return getGenericEdgeKindName(R);
    }
  }

public:
  ELFLinkGraphBuilder_aarch64(StringRef FileName,
                              const object::ELFFile<ELFT> &Obj,
                              std::shared_ptr<orc::SymbolStringPool> SSP,
                              Triple TT, SubtargetFeatures Features)

```
- **EN**: Implements logic around `getGenericEdgeKindName`, `ELFLinkGraphBuilder_aarch64`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getGenericEdgeKindName`, `ELFLinkGraphBuilder_aarch64` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 536-548
```cpp
      : ELFLinkGraphBuilder<ELFT>(Obj, std::move(SSP), std::move(TT),
                                  std::move(Features), FileName,
                                  aarch64::getEdgeKindName) {}
};

// TLS Info Builder.
class TLSInfoTableManager_ELF_aarch64
    : public TableManager<TLSInfoTableManager_ELF_aarch64> {
public:
  static StringRef getSectionName() { return "$__TLSINFO"; }

  static const uint8_t TLSInfoEntryContent[16];

```
- **EN**: Introduces declarations for `TLSInfoTableManager_ELF_aarch64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `TLSInfoTableManager_ELF_aarch64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 549-560
```cpp
  bool visitEdge(LinkGraph &G, Block *B, Edge &E) { return false; }

  Symbol &createEntry(LinkGraph &G, Symbol &Target) {
    // the TLS Info entry's key value will be written by the fixTLVSectionByName
    // pass, so create mutable content.
    auto &TLSInfoEntry = G.createMutableContentBlock(
        getTLSInfoSection(G), G.allocateContent(getTLSInfoEntryContent()),
        orc::ExecutorAddr(), 8, 0);
    TLSInfoEntry.addEdge(aarch64::Pointer64, 8, Target, 0);
    return G.addAnonymousSymbol(TLSInfoEntry, 0, 16, false, false);
  }

```
- **EN**: Implements logic around `visitEdge`, `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `visitEdge`, `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 561-572
```cpp
private:
  Section &getTLSInfoSection(LinkGraph &G) {
    if (!TLSInfoTable)
      TLSInfoTable = &G.createSection(getSectionName(), orc::MemProt::Read);
    return *TLSInfoTable;
  }

  ArrayRef<char> getTLSInfoEntryContent() const {
    return {reinterpret_cast<const char *>(TLSInfoEntryContent),
            sizeof(TLSInfoEntryContent)};
  }

```
- **EN**: Implements logic around `getTLSInfoSection`, `createSection`, `getTLSInfoEntryContent`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getTLSInfoSection`, `createSection`, `getTLSInfoEntryContent` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 573-588
```cpp
  Section *TLSInfoTable = nullptr;
};

const uint8_t TLSInfoTableManager_ELF_aarch64::TLSInfoEntryContent[16] = {
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, /*pthread key */
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00  /*data address*/
};

// TLS Descriptor Builder.
class TLSDescTableManager_ELF_aarch64
    : public TableManager<TLSDescTableManager_ELF_aarch64> {
public:
  TLSDescTableManager_ELF_aarch64(
      TLSInfoTableManager_ELF_aarch64 &TLSInfoTableManager)
      : TLSInfoTableManager(TLSInfoTableManager) {}

```
- **EN**: Introduces declarations for `TLSDescTableManager_ELF_aarch64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `TLSDescTableManager_ELF_aarch64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 589-612
```cpp
  static StringRef getSectionName() { return "$__TLSDESC"; }

  static const uint8_t TLSDescEntryContent[16];

  bool visitEdge(LinkGraph &G, Block *B, Edge &E) {
    Edge::Kind KindToSet = Edge::Invalid;
    switch (E.getKind()) {
    case aarch64::RequestTLSDescEntryAndTransformToPage21: {
      KindToSet = aarch64::Page21;
      break;
    }
    case aarch64::RequestTLSDescEntryAndTransformToPageOffset12: {
      KindToSet = aarch64::PageOffset12;
      break;
    }
    default:
      return false;
    }
    assert(KindToSet != Edge::Invalid &&
           "Fell through switch, but no new kind to set");
    DEBUG_WITH_TYPE("jitlink", {
      dbgs() << "  Fixing " << G.getEdgeKindName(E.getKind()) << " edge at "
             << B->getFixupAddress(E) << " (" << B->getAddress() << " + "
             << formatv("{0:x}", E.getOffset()) << ")\n";
```
- **EN**: Implements logic around `getSectionName`, `visitEdge`, `assert`, `DEBUG_WITH_TYPE`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; executes or prepares runtime behavior for LLVM IR/JIT code; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionName`, `visitEdge`, `assert`, `DEBUG_WITH_TYPE`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，执行或准备 LLVM IR/JIT 代码的运行时行为，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 613-628
```cpp
    });
    E.setKind(KindToSet);
    E.setTarget(getEntryForTarget(G, E.getTarget()));
    return true;
  }

  Symbol &createEntry(LinkGraph &G, Symbol &Target) {
    auto &EntryBlock =
        G.createContentBlock(getTLSDescSection(G), getTLSDescBlockContent(),
                             orc::ExecutorAddr(), 8, 0);
    EntryBlock.addEdge(aarch64::Pointer64, 0, getTLSDescResolver(G), 0);
    EntryBlock.addEdge(aarch64::Pointer64, 8,
                       TLSInfoTableManager.getEntryForTarget(G, Target), 0);
    return G.addAnonymousSymbol(EntryBlock, 0, 8, false, false);
  }

```
- **EN**: Implements logic around `setKind`, `setTarget`, `createEntry`, `createContentBlock`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `setKind`, `setTarget`, `createEntry`, `createContentBlock`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 629-641
```cpp
private:
  Section &getTLSDescSection(LinkGraph &G) {
    if (!GOTSection)
      GOTSection = &G.createSection(getSectionName(), orc::MemProt::Read);
    return *GOTSection;
  }

  Symbol &getTLSDescResolver(LinkGraph &G) {
    if (!TLSDescResolver)
      TLSDescResolver = &G.addExternalSymbol("__tlsdesc_resolver", 8, false);
    return *TLSDescResolver;
  }

```
- **EN**: Implements logic around `getTLSDescSection`, `createSection`, `getTLSDescResolver`, `addExternalSymbol`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getTLSDescSection`, `createSection`, `getTLSDescResolver`, `addExternalSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 642-658
```cpp
  ArrayRef<char> getTLSDescBlockContent() {
    return {reinterpret_cast<const char *>(TLSDescEntryContent),
            sizeof(TLSDescEntryContent)};
  }

  Section *GOTSection = nullptr;
  Symbol *TLSDescResolver = nullptr;
  TLSInfoTableManager_ELF_aarch64 &TLSInfoTableManager;
};

const uint8_t TLSDescTableManager_ELF_aarch64::TLSDescEntryContent[16] = {
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, /*resolver function pointer*/
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00 /*pointer to tls info*/
};

```
- **EN**: Implements logic around `getTLSDescBlockContent`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getTLSDescBlockContent` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 659-671
```cpp
Error buildTables_ELF_aarch64(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Visiting edges in graph:\n");

  aarch64::GOTTableManager GOT(G);
  aarch64::PLTTableManager PLT(G, GOT);
  TLSInfoTableManager_ELF_aarch64 TLSInfo;
  TLSDescTableManager_ELF_aarch64 TLSDesc(TLSInfo);
  visitExistingEdges(G, GOT, PLT, TLSDesc, TLSInfo);
  return Error::success();
}

} // namespace

```
- **EN**: Implements logic around `buildTables_ELF_aarch64`, `GOT`, `PLT`, `TLSDesc`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `buildTables_ELF_aarch64`, `GOT`, `PLT`, `TLSDesc`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 672-685
```cpp
namespace llvm {
namespace jitlink {

Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromELFObject_aarch64(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

  auto ELFObj = object::ObjectFile::createELFObjectFile(ObjectBuffer);
  if (!ELFObj)
    return ELFObj.takeError();

```
- **EN**: Introduces declarations for `llvm`, `jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 686-699
```cpp
  auto Features = (*ELFObj)->getFeatures();
  if (!Features)
    return Features.takeError();

  assert((*ELFObj)->getArch() == Triple::aarch64 &&
         "Only AArch64 (little endian) is supported for now");

  auto &ELFObjFile = cast<object::ELFObjectFile<object::ELF64LE>>(**ELFObj);
  return ELFLinkGraphBuilder_aarch64<object::ELF64LE>(
             (*ELFObj)->getFileName(), ELFObjFile.getELFFile(), std::move(SSP),
             (*ELFObj)->makeTriple(), std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `getFeatures`, `takeError`, `assert`, `AArch64`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getFeatures`, `takeError`, `assert`, `AArch64`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 700-711
```cpp
void link_ELF_aarch64(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;
  const Triple &TT = G->getTargetTriple();
  if (Ctx->shouldAddDefaultTargetPasses(TT)) {
    // Add eh-frame passes.
    Config.PrePrunePasses.push_back(DWARFRecordSectionSplitter(".eh_frame"));
    Config.PrePrunePasses.push_back(EHFrameEdgeFixer(
        ".eh_frame", 8, aarch64::Pointer32, aarch64::Pointer64,
        aarch64::Delta32, aarch64::Delta64, aarch64::NegDelta32));
    Config.PrePrunePasses.push_back(EHFrameNullTerminator(".eh_frame"));

```
- **EN**: Implements logic around `link_ELF_aarch64`, `getTargetTriple`, `push_back`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_ELF_aarch64`, `getTargetTriple`, `push_back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 712-726
```cpp
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(TT))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

    // Resolve any external section start / end symbols.
    Config.PostAllocationPasses.push_back(
        createDefineExternalSectionStartAndEndSymbolsPass(
            identifyELFSectionStartAndEndSymbols));

    // Add an in-place GOT/TLS/Stubs build pass.
    Config.PostPrunePasses.push_back(buildTables_ELF_aarch64);
  }

```
- **EN**: Implements logic around `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 727-734
```cpp
  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

  ELFJITLinker_aarch64::link(std::move(Ctx), std::move(G), std::move(Config));
}

} // namespace jitlink
} // namespace llvm
```
- **EN**: Introduces declarations for `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **JITLink graph linking / JITLink 图链接**:
  - **EN**: Represents object code as graphs of blocks and edges so passes can rewrite relocations before final linking
  - **CN**: 把目标代码表示为块与边构成的图，使 pass 能在最终链接前重写重定位
- **Link graphs / 链接图**:
  - **EN**: Represents atoms of code/data and relocation edges explicitly for JIT-time rewriting
  - **CN**: 显式表示代码/数据原子及其重定位边，以便在 JIT 期重写
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/ELF_aarch64.h`, `llvm/BinaryFormat/ELF.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/aarch64.h`, `llvm/Object/ELFObjectFile.h`, `llvm/Support/Endian.h`, `DefineExternalSectionStartAndEndSymbols.h`, `EHFrameSupportImpl.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support, BinaryFormat
