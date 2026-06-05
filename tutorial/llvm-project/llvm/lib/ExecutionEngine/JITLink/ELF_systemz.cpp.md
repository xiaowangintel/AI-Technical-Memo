# ELF_systemz.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELF_systemz.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements JIT linker implementation for ELF/systemz.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===----- ELF_systemz.cpp - JIT linker implementation for ELF/systemz ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// ELF/systemz jit-link implementation.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-21
```cpp

#include "llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h"
#include "llvm/ExecutionEngine/JITLink/systemz.h"
#include "llvm/Object/ELFObjectFile.h"

#include "DefineExternalSectionStartAndEndSymbols.h"
#include "EHFrameSupportImpl.h"
#include "ELFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/systemz.h`, `llvm/Object/ELFObjectFile.h`, `DefineExternalSectionStartAndEndSymbols.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/systemz.h`, `llvm/Object/ELFObjectFile.h`, `DefineExternalSectionStartAndEndSymbols.h`。

### Lines 22-31
```cpp
#define DEBUG_TYPE "jitlink"

using namespace llvm;
using namespace llvm::jitlink;

namespace {

constexpr StringRef ELFGOTSymbolName = "_GLOBAL_OFFSET_TABLE_";
constexpr StringRef ELFTLSInfoSectionName = "$__TLSINFO";

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 32-49
```cpp
// TLS Info Builder.
class TLSInfoTableManager_ELF_systemz
    : public TableManager<TLSInfoTableManager_ELF_systemz> {
public:
  static StringRef getSectionName() { return ELFTLSInfoSectionName; }

  static const uint8_t TLSInfoEntryContent[16];

  bool visitEdge(LinkGraph &G, Block *B, Edge &E) {
    if (E.getKind() ==
        systemz::RequestTLSDescInGOTAndTransformToDelta64FromGOT) {
      LLVM_DEBUG({
        dbgs() << "  Fixing " << G.getEdgeKindName(E.getKind()) << " edge at "
               << formatv("{0:x}", B->getFixupAddress(E)) << " ("
               << formatv("{0:x}", B->getAddress()) << " + "
               << formatv("{0:x}", E.getOffset()) << ")\n";
      });
      E.setKind(systemz::Delta64FromGOT);
```
- **EN**: Introduces declarations for `TLSInfoTableManager_ELF_systemz`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `TLSInfoTableManager_ELF_systemz` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 50-65
```cpp
      E.setTarget(getEntryForTarget(G, E.getTarget()));
      return true;
    }
    return false;
  }

  Symbol &createEntry(LinkGraph &G, Symbol &Target) {
    // the TLS Info entry's key value will be written by the fixTLVSectionByName
    // pass, so create mutable content.
    auto &TLSInfoEntry = G.createMutableContentBlock(
        getTLSInfoSection(G), G.allocateContent(getTLSInfoEntryContent()),
        orc::ExecutorAddr(), 8, 0);
    TLSInfoEntry.addEdge(systemz::Pointer64, 8, Target, 0);
    return G.addAnonymousSymbol(TLSInfoEntry, 0, 16, false, false);
  }

```
- **EN**: Implements logic around `setTarget`, `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setTarget`, `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 66-77
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

### Lines 78-93
```cpp
  Section *TLSInfoTable = nullptr;
};

const uint8_t TLSInfoTableManager_ELF_systemz::TLSInfoEntryContent[16] = {
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};

Error buildTables_ELF_systemz(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Visiting edges in graph:\n");
  systemz::GOTTableManager GOT;
  systemz::PLTTableManager PLT(GOT);
  TLSInfoTableManager_ELF_systemz TLSInfo;
  visitExistingEdges(G, GOT, PLT, TLSInfo);
  return Error::success();
}

```
- **EN**: Implements logic around `buildTables_ELF_systemz`, `PLT`, `visitExistingEdges`, `success`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `buildTables_ELF_systemz`, `PLT`, `visitExistingEdges`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 94-110
```cpp
} // namespace

namespace llvm {
namespace jitlink {
class ELFJITLinker_systemz : public JITLinker<ELFJITLinker_systemz> {
  friend class JITLinker<ELFJITLinker_systemz>;

public:
  ELFJITLinker_systemz(std::unique_ptr<JITLinkContext> Ctx,
                       std::unique_ptr<LinkGraph> G,
                       PassConfiguration PassConfig)
      : JITLinker(std::move(Ctx), std::move(G), std::move(PassConfig)) {
    if (shouldAddDefaultTargetPasses(getGraph().getTargetTriple()))
      getPassConfig().PostAllocationPasses.push_back(
          [this](LinkGraph &G) { return getOrCreateGOTSymbol(G); });
  }

```
- **EN**: Introduces declarations for `llvm`, `jitlink`, `ELFJITLinker_systemz`, `JITLinker`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `jitlink`, `ELFJITLinker_systemz`, `JITLinker` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 111-128
```cpp
private:
  Symbol *GOTSymbol = nullptr;

  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return systemz::applyFixup(G, B, E, GOTSymbol);
  }

  Error getOrCreateGOTSymbol(LinkGraph &G) {
    auto DefineExternalGOTSymbolIfPresent =
        createDefineExternalSectionStartAndEndSymbolsPass(
            [&](LinkGraph &LG, Symbol &Sym) -> SectionRangeSymbolDesc {
              if (Sym.getName() != nullptr &&
                  *Sym.getName() == ELFGOTSymbolName)
                if (auto *GOTSection = G.findSectionByName(
                        systemz::GOTTableManager::getSectionName())) {
                  GOTSymbol = &Sym;
                  return {*GOTSection, true};
                }
```
- **EN**: Implements logic around `applyFixup`, `getOrCreateGOTSymbol`, `createDefineExternalSectionStartAndEndSymbolsPass`, `getName`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `applyFixup`, `getOrCreateGOTSymbol`, `createDefineExternalSectionStartAndEndSymbolsPass`, `getName`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 129-140
```cpp
              return {};
            });

    // Try to attach _GLOBAL_OFFSET_TABLE_ to the GOT if it's defined as an
    // external.
    if (auto Err = DefineExternalGOTSymbolIfPresent(G))
      return Err;

    // If we succeeded then we're done.
    if (GOTSymbol)
      return Error::success();

```
- **EN**: Implements logic around `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 141-153
```cpp
    // Otherwise look for a GOT section: If it already has a start symbol we'll
    // record it, otherwise we'll create our own.
    // If there's a GOT section but we didn't find an external GOT symbol...
    if (auto *GOTSection =
            G.findSectionByName(systemz::GOTTableManager::getSectionName())) {

      // Check for an existing defined symbol.
      for (auto *Sym : GOTSection->symbols())
        if (Sym->getName() != nullptr && *Sym->getName() == ELFGOTSymbolName) {
          GOTSymbol = Sym;
          return Error::success();
        }

```
- **EN**: Implements logic around `findSectionByName`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSectionByName`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 154-165
```cpp
      // If there's no defined symbol then create one.
      SectionRange SR(*GOTSection);
      if (SR.empty())
        GOTSymbol =
            &G.addAbsoluteSymbol(ELFGOTSymbolName, orc::ExecutorAddr(), 0,
                                 Linkage::Strong, Scope::Local, true);
      else
        GOTSymbol =
            &G.addDefinedSymbol(*SR.getFirstBlock(), 0, ELFGOTSymbolName, 0,
                                Linkage::Strong, Scope::Local, false, true);
    }

```
- **EN**: Implements logic around `SR`, `addAbsoluteSymbol`, `addDefinedSymbol`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `SR`, `addAbsoluteSymbol`, `addDefinedSymbol` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 166-181
```cpp
    // If we still haven't found a GOT symbol then double check the externals.
    // We may have a GOT-relative reference but no GOT section, in which case
    // we just need to point the GOT symbol at some address in this graph.
    if (!GOTSymbol) {
      for (auto *Sym : G.external_symbols()) {
        if (Sym->getName() != nullptr && *Sym->getName() == ELFGOTSymbolName) {
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

### Lines 182-192
```cpp
    return Error::success();
  }
};

class ELFLinkGraphBuilder_systemz
    : public ELFLinkGraphBuilder<object::ELF64BE> {
private:
  using ELFT = object::ELF64BE;
  using Base = ELFLinkGraphBuilder<ELFT>;
  using Base::G; // Use LinkGraph pointer from base class.

```
- **EN**: Introduces declarations for `ELFLinkGraphBuilder_systemz`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilder_systemz` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 193-205
```cpp
  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

    using Base = ELFLinkGraphBuilder<ELFT>;
    using Self = ELFLinkGraphBuilder_systemz;
    for (const auto &RelSect : Base::Sections) {
      if (RelSect.sh_type == ELF::SHT_REL)
        // Validate the section to read relocation entries from.
        return make_error<StringError>("No SHT_REL in valid " +
                                           G->getTargetTriple().getArchName() +
                                           " ELF object files",
                                       inconvertibleErrorCode());

```
- **EN**: Implements logic around `addRelocations`, `make_error<StringError>`, `getTargetTriple`, `inconvertibleErrorCode`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocations`, `make_error<StringError>`, `getTargetTriple`, `inconvertibleErrorCode` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 206-220
```cpp
      if (Error Err = Base::forEachRelaRelocation(RelSect, this,
                                                  &Self::addSingleRelocation))
        return Err;
    }

    return Error::success();
  }

  Error addSingleRelocation(const typename ELFT::Rela &Rel,
                            const typename ELFT::Shdr &FixupSect,
                            Block &BlockToFix) {
    using support::big32_t;
    using Base = ELFLinkGraphBuilder<ELFT>;
    auto ELFReloc = Rel.getType(false);

```
- **EN**: Implements logic around `success`, `addSingleRelocation`, `getType`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `addSingleRelocation`, `getType` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 221-229
```cpp
    // No reloc.
    if (LLVM_UNLIKELY(ELFReloc == ELF::R_390_NONE))
      return Error::success();

    uint32_t SymbolIndex = Rel.getSymbol(false);
    auto ObjSymbol = Base::Obj.getRelocationSymbol(Rel, Base::SymTabSec);
    if (!ObjSymbol)
      return ObjSymbol.takeError();

```
- **EN**: Implements logic around `success`, `getSymbol`, `getRelocationSymbol`, `takeError`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getSymbol`, `getRelocationSymbol`, `takeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 230-238
```cpp
    Symbol *GraphSymbol = Base::getGraphSymbol(SymbolIndex);
    if (!GraphSymbol)
      return make_error<StringError>(
          formatv("Could not find symbol at given index, did you add it to "
                  "JITSymbolTable? index: {0}, shndx: {1} Size of table: {2}",
                  SymbolIndex, (*ObjSymbol)->st_shndx,
                  Base::GraphSymbols.size()),
          inconvertibleErrorCode());

```
- **EN**: Implements logic around `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphSymbol`, `make_error<StringError>`, `formatv`, `size`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 239-256
```cpp
    // Validate the relocation kind.
    int64_t Addend = Rel.r_addend;
    Edge::Kind Kind = Edge::Invalid;

    switch (ELFReloc) {
    case ELF::R_390_PC64: {
      Kind = systemz::Delta64;
      break;
    }
    case ELF::R_390_PC32: {
      Kind = systemz::Delta32;
      break;
    }
    case ELF::R_390_PC16: {
      Kind = systemz::Delta16;
      break;
    }
    case ELF::R_390_PC32DBL: {
```
- **EN**: Implements subsystem-specific case analysis using a `switch` over kinds, opcodes, or states.
- **CN**: 通过对 kind、opcode 或状态执行 `switch` 分析来实现子系统相关逻辑。

### Lines 257-274
```cpp
      Kind = systemz::Delta32dbl;
      break;
    }
    case ELF::R_390_PC24DBL: {
      Kind = systemz::Delta24dbl;
      break;
    }
    case ELF::R_390_PC16DBL: {
      Kind = systemz::Delta16dbl;
      break;
    }
    case ELF::R_390_PC12DBL: {
      Kind = systemz::Delta12dbl;
      break;
    }
    case ELF::R_390_64: {
      Kind = systemz::Pointer64;
      break;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 275-292
```cpp
    }
    case ELF::R_390_32: {
      Kind = systemz::Pointer32;
      break;
    }
    case ELF::R_390_20: {
      Kind = systemz::Pointer20;
      break;
    }
    case ELF::R_390_16: {
      Kind = systemz::Pointer16;
      break;
    }
    case ELF::R_390_12: {
      Kind = systemz::Pointer12;
      break;
    }
    case ELF::R_390_8: {
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 293-310
```cpp
      Kind = systemz::Pointer8;
      break;
    }
    // Relocations targeting the PLT associated with the symbol.
    case ELF::R_390_PLT64: {
      Kind = systemz::DeltaPLT64;
      break;
    }
    case ELF::R_390_PLT32: {
      Kind = systemz::DeltaPLT32;
      break;
    }
    case ELF::R_390_PLT32DBL: {
      Kind = systemz::DeltaPLT32dbl;
      break;
    }
    case ELF::R_390_PLT24DBL: {
      Kind = systemz::DeltaPLT24dbl;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 311-328
```cpp
      break;
    }
    case ELF::R_390_PLT16DBL: {
      Kind = systemz::DeltaPLT16dbl;
      break;
    }
    case ELF::R_390_PLT12DBL: {
      Kind = systemz::DeltaPLT12dbl;
      break;
    }
    case ELF::R_390_PLTOFF64: {
      Kind = systemz::Delta64PLTFromGOT;
      break;
    }
    case ELF::R_390_PLTOFF32: {
      Kind = systemz::Delta32PLTFromGOT;
      break;
    }
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 329-346
```cpp
    case ELF::R_390_PLTOFF16: {
      Kind = systemz::Delta16PLTFromGOT;
      break;
    }
    // Relocations targeting the actual symbol (just relative to the GOT).
    case ELF::R_390_GOTOFF64: {
      Kind = systemz::Delta64FromGOT;
      break;
    }
    case ELF::R_390_GOTOFF: {
      Kind = systemz::Delta32FromGOT;
      break;
    }
    case ELF::R_390_GOTOFF16: {
      Kind = systemz::Delta16FromGOT;
      break;
    }
    // Relocations targeting the GOT entry associated with the symbol.
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 347-364
```cpp
    case ELF::R_390_GOT64:
    case ELF::R_390_GOTPLT64: {
      Kind = systemz::RequestGOTAndTransformToDelta64FromGOT;
      break;
    }
    case ELF::R_390_GOT32:
    case ELF::R_390_GOTPLT32: {
      Kind = systemz::RequestGOTAndTransformToDelta32FromGOT;
      break;
    }
    case ELF::R_390_GOT20:
    case ELF::R_390_GOTPLT20: {
      Kind = systemz::RequestGOTAndTransformToDelta20FromGOT;
      break;
    }
    case ELF::R_390_GOT16:
    case ELF::R_390_GOTPLT16: {
      Kind = systemz::RequestGOTAndTransformToDelta16FromGOT;
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 365-382
```cpp
      break;
    }
    case ELF::R_390_GOT12:
    case ELF::R_390_GOTPLT12: {
      Kind = systemz::RequestGOTAndTransformToDelta12FromGOT;
      break;
    }
    case ELF::R_390_GOTENT:
    case ELF::R_390_GOTPLTENT: {
      Kind = systemz::RequestGOTAndTransformToDelta32dbl;
      break;
    }
    // R_390_GOTPC and R_390_GOTPCDBL don't create GOT entry, they don't even
    // have symbol.
    case ELF::R_390_GOTPC: {
      Kind = systemz::Delta32GOTBase;
      break;
    }
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 383-400
```cpp
    case ELF::R_390_GOTPCDBL: {
      Kind = systemz::Delta32dblGOTBase;
      break;
    }
    // Tag for function call in general dynamic TLS code.
    case ELF::R_390_TLS_GDCALL: {
      break;
    }
    // Direct 64 bit for general dynamic thread local data.
    case ELF::R_390_TLS_GD64: {
      Kind = systemz::RequestTLSDescInGOTAndTransformToDelta64FromGOT;
      break;
    }
    default:
      return make_error<JITLinkError>(
          "In " + G->getName() + ": Unsupported systemz relocation type " +
          object::getELFRelocationTypeName(ELF::EM_S390, ELFReloc));
    }
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getName`, `getELFRelocationTypeName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 401-409
```cpp
    auto FixupAddress = orc::ExecutorAddr(FixupSect.sh_addr) + Rel.r_offset;
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();
    Edge GE(Kind, Offset, *GraphSymbol, Addend);
    LLVM_DEBUG({
      dbgs() << "    ";
      printEdge(dbgs(), BlockToFix, GE, systemz::getEdgeKindName(Kind));
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `ExecutorAddr`, `getAddress`, `GE`, `dbgs`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `ExecutorAddr`, `getAddress`, `GE`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作。

### Lines 410-424
```cpp
    BlockToFix.addEdge(std::move(GE));

    return Error::success();
  }

public:
  ELFLinkGraphBuilder_systemz(StringRef FileName,
                              const object::ELFFile<ELFT> &Obj,
                              std::shared_ptr<orc::SymbolStringPool> SSP,
                              Triple TT, SubtargetFeatures Features)
      : ELFLinkGraphBuilder<ELFT>(Obj, std::move(SSP), std::move(TT),
                                  std::move(Features), FileName,
                                  systemz::getEdgeKindName) {}
};

```
- **EN**: Implements logic around `addEdge`, `success`, `ELFLinkGraphBuilder_systemz`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addEdge`, `success`, `ELFLinkGraphBuilder_systemz`, `ELFLinkGraphBuilder<ELFT>`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 425-435
```cpp
Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromELFObject_systemz(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

  auto ELFObj = object::ObjectFile::createELFObjectFile(ObjectBuffer);
  if (!ELFObj)
    return ELFObj.takeError();

```
- **EN**: Implements logic around `createLinkGraphFromELFObject_systemz`, `dbgs`, `getBufferIdentifier`, `createELFObjectFile`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkGraphFromELFObject_systemz`, `dbgs`, `getBufferIdentifier`, `createELFObjectFile`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 436-449
```cpp
  auto Features = (*ELFObj)->getFeatures();
  if (!Features)
    return Features.takeError();

  assert((*ELFObj)->getArch() == Triple::systemz &&
         "Only SystemZ is supported");

  auto &ELFObjFile = cast<object::ELFObjectFile<object::ELF64BE>>(**ELFObj);
  return ELFLinkGraphBuilder_systemz(
             (*ELFObj)->getFileName(), ELFObjFile.getELFFile(), std::move(SSP),
             (*ELFObj)->makeTriple(), std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `getFeatures`, `takeError`, `assert`, `ELF64BE>>`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getFeatures`, `takeError`, `assert`, `ELF64BE>>`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 450-462
```cpp
void link_ELF_systemz(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;
  const Triple &TT = G->getTargetTriple();
  if (Ctx->shouldAddDefaultTargetPasses(TT)) {
    // Add eh-frame passes.
    Config.PrePrunePasses.push_back(DWARFRecordSectionSplitter(".eh_frame"));
    Config.PrePrunePasses.push_back(
        EHFrameEdgeFixer(".eh_frame", G->getPointerSize(), systemz::Pointer32,
                         systemz::Pointer64, systemz::Delta32, systemz::Delta64,
                         systemz::NegDelta32));
    Config.PrePrunePasses.push_back(EHFrameNullTerminator(".eh_frame"));

```
- **EN**: Implements logic around `link_ELF_systemz`, `getTargetTriple`, `push_back`, `EHFrameEdgeFixer`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_ELF_systemz`, `getTargetTriple`, `push_back`, `EHFrameEdgeFixer` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 463-471
```cpp
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(TT))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);

    // Add an in-place GOT/Stubs build pass.
    Config.PostPrunePasses.push_back(buildTables_ELF_systemz);

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 472-480
```cpp
    // Resolve any external section start / end symbols.
    Config.PostAllocationPasses.push_back(
        createDefineExternalSectionStartAndEndSymbolsPass(
            identifyELFSectionStartAndEndSymbols));

    // Add GOT/Stubs optimizer pass.
    Config.PreFixupPasses.push_back(systemz::optimizeGOTAndStubAccesses);
  }

```
- **EN**: Implements logic around `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `push_back`, `createDefineExternalSectionStartAndEndSymbolsPass` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 481-488
```cpp
  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

  ELFJITLinker_systemz::link(std::move(Ctx), std::move(G), std::move(Config));
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/systemz.h`, `llvm/Object/ELFObjectFile.h`, `DefineExternalSectionStartAndEndSymbols.h`, `EHFrameSupportImpl.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object
