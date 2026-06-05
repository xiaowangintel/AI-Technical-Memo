# COFFLinkGraphBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/COFFLinkGraphBuilder.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements COFF LinkGraph builder.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
//=--------- COFFLinkGraphBuilder.cpp - COFF LinkGraph builder ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Generic COFF LinkGraph building code.
//
//===----------------------------------------------------------------------===//
#include "COFFLinkGraphBuilder.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `COFFLinkGraphBuilder.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `COFFLinkGraphBuilder.h`。

### Lines 14-27
```cpp
#include <memory>

#define DEBUG_TYPE "jitlink"

static const char *CommonSectionName = "__common";

namespace llvm {
namespace jitlink {

static Triple createTripleWithCOFFFormat(Triple T) {
  T.setObjectFormat(Triple::COFF);
  return T;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `memory`.
- **CN**: 引入该实现所需的头文件，其中包括 `memory`。

### Lines 28-43
```cpp
COFFLinkGraphBuilder::COFFLinkGraphBuilder(
    const object::COFFObjectFile &Obj,
    std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
    SubtargetFeatures Features,
    LinkGraph::GetEdgeKindNameFunction GetEdgeKindName)
    : Obj(Obj),
      G(std::make_unique<LinkGraph>(Obj.getFileName().str(), std::move(SSP),
                                    createTripleWithCOFFFormat(std::move(TT)),
                                    std::move(Features),
                                    std::move(GetEdgeKindName))) {
  LLVM_DEBUG({
    dbgs() << "Created COFFLinkGraphBuilder for \"" << Obj.getFileName()
           << "\"\n";
  });
}

```
- **EN**: Implements logic around `COFFLinkGraphBuilder`, `Obj`, `G`, `createTripleWithCOFFFormat`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `COFFLinkGraphBuilder`, `Obj`, `G`, `createTripleWithCOFFFormat`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 44-60
```cpp
COFFLinkGraphBuilder::~COFFLinkGraphBuilder() = default;

uint64_t COFFLinkGraphBuilder::getSectionSize(const object::COFFObjectFile &Obj,
                                              const object::coff_section *Sec) {
  // Consider the difference between executable form and object form.
  // More information is inside COFFObjectFile::getSectionSize
  if (Obj.getDOSHeader())
    return std::min(Sec->VirtualSize, Sec->SizeOfRawData);
  return Sec->SizeOfRawData;
}

uint64_t
COFFLinkGraphBuilder::getSectionAddress(const object::COFFObjectFile &Obj,
                                        const object::coff_section *Section) {
  return Section->VirtualAddress + Obj.getImageBase();
}

```
- **EN**: Implements logic around `~COFFLinkGraphBuilder`, `getSectionSize`, `min`, `getSectionAddress`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `~COFFLinkGraphBuilder`, `getSectionSize`, `min`, `getSectionAddress`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 61-72
```cpp
bool COFFLinkGraphBuilder::isComdatSection(
    const object::coff_section *Section) {
  return Section->Characteristics & COFF::IMAGE_SCN_LNK_COMDAT;
}

Section &COFFLinkGraphBuilder::getCommonSection() {
  if (!CommonSection)
    CommonSection = &G->createSection(CommonSectionName,
                                      orc::MemProt::Read | orc::MemProt::Write);
  return *CommonSection;
}

```
- **EN**: Implements logic around `isComdatSection`, `getCommonSection`, `createSection`; this block manipulates JITLink graph structures or link-time passes; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `isComdatSection`, `getCommonSection`, `createSection` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 73-85
```cpp
Expected<std::unique_ptr<LinkGraph>> COFFLinkGraphBuilder::buildGraph() {
  if (!Obj.isRelocatableObject())
    return make_error<JITLinkError>("Object is not a relocatable COFF file");

  if (auto Err = graphifySections())
    return std::move(Err);

  if (auto Err = graphifySymbols())
    return std::move(Err);

  if (auto Err = addRelocations())
    return std::move(Err);

```
- **EN**: Implements logic around `buildGraph`, `make_error<JITLinkError>`, `move`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `buildGraph`, `make_error<JITLinkError>`, `move` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 86-109
```cpp
  return std::move(G);
}

StringRef
COFFLinkGraphBuilder::getCOFFSectionName(COFFSectionIndex SectionIndex,
                                         const object::coff_section *Sec,
                                         object::COFFSymbolRef Sym) {
  switch (SectionIndex) {
  case COFF::IMAGE_SYM_UNDEFINED: {
    if (Sym.getValue())
      return "(common)";
    else
      return "(external)";
  }
  case COFF::IMAGE_SYM_ABSOLUTE:
    return "(absolute)";
  case COFF::IMAGE_SYM_DEBUG: {
    // Used with .file symbol
    return "(debug)";
  }
  default: {
    // Non reserved regular section numbers
    if (Expected<StringRef> SecNameOrErr = Obj.getSectionName(Sec))
      return *SecNameOrErr;
```
- **EN**: Implements logic around `move`, `getCOFFSectionName`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `getCOFFSectionName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 110-126
```cpp
  }
  }
  return "";
}

Error COFFLinkGraphBuilder::graphifySections() {
  LLVM_DEBUG(dbgs() << "  Creating graph sections...\n");

  GraphBlocks.resize(Obj.getNumberOfSections() + 1);
  // For each section...
  for (COFFSectionIndex SecIndex = 1;
       SecIndex <= static_cast<COFFSectionIndex>(Obj.getNumberOfSections());
       SecIndex++) {
    Expected<const object::coff_section *> Sec = Obj.getSection(SecIndex);
    if (!Sec)
      return Sec.takeError();

```
- **EN**: Implements logic around `graphifySections`, `resize`, `static_cast<COFFSectionIndex>`, `getSection`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `graphifySections`, `resize`, `static_cast<COFFSectionIndex>`, `getSection`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 127-139
```cpp
    StringRef SectionName;
    if (Expected<StringRef> SecNameOrErr = Obj.getSectionName(*Sec))
      SectionName = *SecNameOrErr;

    // FIXME: Skip debug info sections
    if (SectionName == ".voltbl") {
      LLVM_DEBUG({
        dbgs() << "    "
               << "Skipping section \"" << SectionName << "\"\n";
      });
      continue;
    }

```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

### Lines 140-153
```cpp
    LLVM_DEBUG({
      dbgs() << "    "
             << "Creating section for \"" << SectionName << "\"\n";
    });

    // Get the section's memory protection flags.
    orc::MemProt Prot = orc::MemProt::Read;
    if ((*Sec)->Characteristics & COFF::IMAGE_SCN_MEM_EXECUTE)
      Prot |= orc::MemProt::Exec;
    if ((*Sec)->Characteristics & COFF::IMAGE_SCN_MEM_READ)
      Prot |= orc::MemProt::Read;
    if ((*Sec)->Characteristics & COFF::IMAGE_SCN_MEM_WRITE)
      Prot |= orc::MemProt::Write;

```
- **EN**: Implements logic around `dbgs`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `dbgs` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 154-174
```cpp
    // Look for existing sections first.
    auto *GraphSec = G->findSectionByName(SectionName);
    if (!GraphSec) {
      GraphSec = &G->createSection(SectionName, Prot);
      if ((*Sec)->Characteristics & COFF::IMAGE_SCN_LNK_REMOVE)
        GraphSec->setMemLifetime(orc::MemLifetime::NoAlloc);
    }
    if (GraphSec->getMemProt() != Prot)
      return make_error<JITLinkError>("MemProt should match");

    Block *B = nullptr;
    if ((*Sec)->Characteristics & COFF::IMAGE_SCN_CNT_UNINITIALIZED_DATA)
      B = &G->createZeroFillBlock(
          *GraphSec, getSectionSize(Obj, *Sec),
          orc::ExecutorAddr(getSectionAddress(Obj, *Sec)),
          (*Sec)->getAlignment(), 0);
    else {
      ArrayRef<uint8_t> Data;
      if (auto Err = Obj.getSectionContents(*Sec, Data))
        return Err;

```
- **EN**: Implements logic around `findSectionByName`, `createSection`, `setMemLifetime`, `make_error<JITLinkError>`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `findSectionByName`, `createSection`, `setMemLifetime`, `make_error<JITLinkError>`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 175-187
```cpp
      auto CharData = ArrayRef<char>(
          reinterpret_cast<const char *>(Data.data()), Data.size());

      if (SectionName == getDirectiveSectionName())
        if (auto Err = handleDirectiveSection(
                StringRef(CharData.data(), CharData.size())))
          return Err;

      B = &G->createContentBlock(
          *GraphSec, CharData, orc::ExecutorAddr(getSectionAddress(Obj, *Sec)),
          (*Sec)->getAlignment(), 0);
    }

```
- **EN**: Implements logic around `ArrayRef<char>`, `data`, `StringRef`, `createContentBlock`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `ArrayRef<char>`, `data`, `StringRef`, `createContentBlock`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 188-200
```cpp
    setGraphBlock(SecIndex, B);
  }

  return Error::success();
}

Error COFFLinkGraphBuilder::graphifySymbols() {
  LLVM_DEBUG(dbgs() << "  Creating graph symbols...\n");

  SymbolSets.resize(Obj.getNumberOfSections() + 1);
  PendingComdatExports.resize(Obj.getNumberOfSections() + 1);
  GraphSymbols.resize(Obj.getNumberOfSymbols());

```
- **EN**: Implements logic around `setGraphBlock`, `success`, `graphifySymbols`, `resize`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setGraphBlock`, `success`, `graphifySymbols`, `resize` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 201-214
```cpp
  for (COFFSymbolIndex SymIndex = 0;
       SymIndex < static_cast<COFFSymbolIndex>(Obj.getNumberOfSymbols());
       SymIndex++) {
    Expected<object::COFFSymbolRef> Sym = Obj.getSymbol(SymIndex);
    if (!Sym)
      return Sym.takeError();

    StringRef SymbolName;
    if (Expected<StringRef> SymNameOrErr = Obj.getSymbolName(*Sym))
      SymbolName = *SymNameOrErr;

    COFFSectionIndex SectionIndex = Sym->getSectionNumber();
    const object::coff_section *Sec = nullptr;

```
- **EN**: Implements logic around `static_cast<COFFSymbolIndex>`, `getSymbol`, `takeError`, `getSectionNumber`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `static_cast<COFFSymbolIndex>`, `getSymbol`, `takeError`, `getSectionNumber` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 215-238
```cpp
    if (!COFF::isReservedSectionNumber(SectionIndex)) {
      auto SecOrErr = Obj.getSection(SectionIndex);
      if (!SecOrErr)
        return make_error<JITLinkError>(
            "Invalid COFF section number:" + formatv("{0:d}: ", SectionIndex) +
            " (" + toString(SecOrErr.takeError()) + ")");
      Sec = *SecOrErr;
    }
    auto InternedSymbolName = G->intern(std::move(SymbolName));

    // Create jitlink symbol
    jitlink::Symbol *GSym = nullptr;
    if (Sym->isFileRecord())
      LLVM_DEBUG({
        dbgs() << "    " << SymIndex << ": Skipping FileRecord symbol \""
               << InternedSymbolName << "\" in "
               << getCOFFSectionName(SectionIndex, Sec, *Sym)
               << " (index: " << SectionIndex << ") \n";
      });
    else if (Sym->isUndefined()) {
      GSym = createExternalSymbol(SymIndex, InternedSymbolName, *Sym, Sec);
    } else if (Sym->isWeakExternal()) {
      auto *WeakExternal = Sym->getAux<object::coff_aux_weak_external>();
      COFFSymbolIndex TagIndex = WeakExternal->TagIndex;
```
- **EN**: Implements logic around `getSection`, `make_error<JITLinkError>`, `formatv`, `toString`, and 5 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSection`, `make_error<JITLinkError>`, `formatv`, `toString`, and 5 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 239-259
```cpp
      uint32_t Characteristics = WeakExternal->Characteristics;
      WeakExternalRequests.push_back(
          {SymIndex, TagIndex, Characteristics, SymbolName});
    } else {
      Expected<jitlink::Symbol *> NewGSym =
          createDefinedSymbol(SymIndex, InternedSymbolName, *Sym, Sec);
      if (!NewGSym)
        return NewGSym.takeError();
      GSym = *NewGSym;
      if (GSym) {
        LLVM_DEBUG({
          dbgs() << "    " << SymIndex
                 << ": Creating defined graph symbol for COFF symbol \""
                 << InternedSymbolName << "\" in "
                 << getCOFFSectionName(SectionIndex, Sec, *Sym)
                 << " (index: " << SectionIndex << ") \n";
          dbgs() << "      " << *GSym << "\n";
        });
      }
    }

```
- **EN**: Implements logic around `push_back`, `createDefinedSymbol`, `takeError`, `dbgs`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `push_back`, `createDefinedSymbol`, `takeError`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 260-271
```cpp
    // Register the symbol
    if (GSym)
      setGraphSymbol(SectionIndex, SymIndex, *GSym);
    SymIndex += Sym->getNumberOfAuxSymbols();
  }

  if (auto Err = flushWeakAliasRequests())
    return Err;

  if (auto Err = handleAlternateNames())
    return Err;

```
- **EN**: Implements logic around `setGraphSymbol`, `getNumberOfAuxSymbols`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setGraphSymbol`, `getNumberOfAuxSymbols` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 272-295
```cpp
  if (auto Err = calculateImplicitSizeOfSymbols())
    return Err;

  return Error::success();
}

Error COFFLinkGraphBuilder::handleDirectiveSection(StringRef Str) {
  auto Parsed = DirectiveParser.parse(Str);
  if (!Parsed)
    return Parsed.takeError();
  for (auto *Arg : *Parsed) {
    StringRef S = Arg->getValue();
    switch (Arg->getOption().getID()) {
    case COFF_OPT_alternatename: {
      StringRef From, To;
      std::tie(From, To) = S.split('=');
      if (From.empty() || To.empty())
        return make_error<JITLinkError>(
            "Invalid COFF /alternatename directive");
      AlternateNames[G->intern(From)] = G->intern(To);
      break;
    }
    case COFF_OPT_incl: {
      auto Symbol = &G->addExternalSymbol(S, 0, false);
```
- **EN**: Implements logic around `success`, `handleDirectiveSection`, `parse`, `takeError`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `handleDirectiveSection`, `parse`, `takeError`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 296-312
```cpp
      Symbol->setLive(true);
      ExternalSymbols[Symbol->getName()] = Symbol;
      break;
    }
    case COFF_OPT_export:
      break;
    default: {
      LLVM_DEBUG({
        dbgs() << "Unknown coff directive: " << Arg->getSpelling() << "\n";
      });
      break;
    }
    }
  }
  return Error::success();
}

```
- **EN**: Implements logic around `setLive`, `getName`, `dbgs`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setLive`, `getName`, `dbgs`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 313-328
```cpp
Error COFFLinkGraphBuilder::flushWeakAliasRequests() {
  // Export the weak external symbols and alias it
  for (auto &WeakExternal : WeakExternalRequests) {
    if (auto *Target = getGraphSymbol(WeakExternal.Target)) {
      Expected<object::COFFSymbolRef> AliasSymbol =
          Obj.getSymbol(WeakExternal.Alias);
      if (!AliasSymbol)
        return AliasSymbol.takeError();

      // FIXME: IMAGE_WEAK_EXTERN_SEARCH_NOLIBRARY and
      // IMAGE_WEAK_EXTERN_SEARCH_LIBRARY are handled in the same way.
      Scope S =
          WeakExternal.Characteristics == COFF::IMAGE_WEAK_EXTERN_SEARCH_ALIAS
              ? Scope::Default
              : Scope::Local;

```
- **EN**: Implements logic around `flushWeakAliasRequests`, `getSymbol`, `takeError`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `flushWeakAliasRequests`, `getSymbol`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 329-349
```cpp
      auto NewSymbol = createAliasSymbol(G->intern(WeakExternal.SymbolName),
                                         Linkage::Weak, S, *Target);
      if (!NewSymbol)
        return NewSymbol.takeError();
      setGraphSymbol(AliasSymbol->getSectionNumber(), WeakExternal.Alias,
                     **NewSymbol);
      LLVM_DEBUG({
        dbgs() << "    " << WeakExternal.Alias
               << ": Creating weak external symbol for COFF symbol \""
               << WeakExternal.SymbolName << "\" in section "
               << AliasSymbol->getSectionNumber() << "\n";
        dbgs() << "      " << **NewSymbol << "\n";
      });
    } else
      return make_error<JITLinkError>("Weak symbol alias requested but actual "
                                      "symbol not found for symbol " +
                                      formatv("{0:d}", WeakExternal.Alias));
  }
  return Error::success();
}

```
- **EN**: Implements logic around `createAliasSymbol`, `takeError`, `setGraphSymbol`, `dbgs`, and 4 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `createAliasSymbol`, `takeError`, `setGraphSymbol`, `dbgs`, and 4 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 350-364
```cpp
Error COFFLinkGraphBuilder::handleAlternateNames() {
  for (auto &KeyValue : AlternateNames) {
    auto DefinedSymbolName = KeyValue.second;
    auto ExternalSymbolsName = KeyValue.first;
    if (DefinedSymbols.count(DefinedSymbolName) &&
        ExternalSymbols.count(ExternalSymbolsName)) {
      auto *Target = DefinedSymbols[DefinedSymbolName];
      auto *Alias = ExternalSymbols[ExternalSymbolsName];
      G->makeDefined(*Alias, Target->getBlock(), Target->getOffset(),
                     Target->getSize(), Linkage::Weak, Scope::Local, false);
    }
  }
  return Error::success();
}

```
- **EN**: Implements logic around `handleAlternateNames`, `count`, `makeDefined`, `getSize`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `handleAlternateNames`, `count`, `makeDefined`, `getSize`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 365-385
```cpp
Symbol *COFFLinkGraphBuilder::createExternalSymbol(
    COFFSymbolIndex SymIndex, orc::SymbolStringPtr SymbolName,
    object::COFFSymbolRef Symbol, const object::coff_section *Section) {
  llvm::jitlink::Symbol *Sym = nullptr;
  if (auto It = ExternalSymbols.find(SymbolName); It == ExternalSymbols.end()) {
    Sym = &G->addExternalSymbol(*SymbolName, Symbol.getValue(), false);
    ExternalSymbols[Sym->getName()] = Sym;
  } else {
    Sym = It->second;
  }

  LLVM_DEBUG({
    dbgs() << "    " << SymIndex
           << ": Creating external graph symbol for COFF symbol \""
           << Sym->getName() << "\" in "
           << getCOFFSectionName(Symbol.getSectionNumber(), Section, Symbol)
           << " (index: " << Symbol.getSectionNumber() << ") \n";
  });
  return Sym;
}

```
- **EN**: Implements logic around `createExternalSymbol`, `addExternalSymbol`, `getName`, `dbgs`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createExternalSymbol`, `addExternalSymbol`, `getName`, `dbgs`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 386-398
```cpp
Expected<Symbol *>
COFFLinkGraphBuilder::createAliasSymbol(orc::SymbolStringPtr SymbolName,
                                        Linkage L, Scope S, Symbol &Target) {
  if (!Target.isDefined()) {
    // FIXME: Support this when there's a way to handle this.
    return make_error<JITLinkError>("Weak external symbol with external "
                                    "symbol as alternative not supported.");
  }
  return &G->addDefinedSymbol(Target.getBlock(), Target.getOffset(), SymbolName,
                              Target.getSize(), L, S, Target.isCallable(),
                              false);
}

```
- **EN**: Implements logic around `createAliasSymbol`, `make_error<JITLinkError>`, `addDefinedSymbol`, `getSize`; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createAliasSymbol`, `make_error<JITLinkError>`, `addDefinedSymbol`, `getSize` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 399-422
```cpp
// In COFF, most of the defined symbols don't contain the size information.
// Hence, we calculate the "implicit" size of symbol by taking the delta of
// offsets of consecutive symbols within a block. We maintain a balanced tree
// set of symbols sorted by offset per each block in order to achieve
// logarithmic time complexity of sorted symbol insertion. Symbol is inserted to
// the set once it's processed in graphifySymbols. In this function, we iterate
// each collected symbol in sorted order and calculate the implicit size.
Error COFFLinkGraphBuilder::calculateImplicitSizeOfSymbols() {
  for (COFFSectionIndex SecIndex = 1;
       SecIndex <= static_cast<COFFSectionIndex>(Obj.getNumberOfSections());
       SecIndex++) {
    auto &SymbolSet = SymbolSets[SecIndex];
    if (SymbolSet.empty())
      continue;
    jitlink::Block *B = getGraphBlock(SecIndex);
    orc::ExecutorAddrDiff LastOffset = B->getSize();
    orc::ExecutorAddrDiff LastDifferentOffset = B->getSize();
    orc::ExecutorAddrDiff LastSize = 0;
    for (auto It = SymbolSet.rbegin(); It != SymbolSet.rend(); It++) {
      orc::ExecutorAddrDiff Offset = It->first;
      jitlink::Symbol *Symbol = It->second;
      orc::ExecutorAddrDiff CandSize;
      // Last offset can be same when aliasing happened
      if (Symbol->getOffset() == LastOffset)
```
- **EN**: Implements logic around `calculateImplicitSizeOfSymbols`, `static_cast<COFFSectionIndex>`, `getGraphBlock`, `getSize`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `calculateImplicitSizeOfSymbols`, `static_cast<COFFSectionIndex>`, `getGraphBlock`, `getSize` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 423-446
```cpp
        CandSize = LastSize;
      else
        CandSize = LastOffset - Offset;

      LLVM_DEBUG({
        if (Offset + Symbol->getSize() > LastDifferentOffset)
          dbgs() << "  Overlapping symbol range generated for the following "
                    "symbol:"
                 << "\n"
                 << "    " << *Symbol << "\n";
      });
      (void)LastDifferentOffset;
      if (LastOffset != Offset)
        LastDifferentOffset = Offset;
      LastSize = CandSize;
      LastOffset = Offset;
      if (Symbol->getSize()) {
        // Non empty symbol can happen in COMDAT symbol.
        // We don't consider the possibility of overlapping symbol range that
        // could be introduced by disparity between inferred symbol size and
        // defined symbol size because symbol size information is currently only
        // used by jitlink-check where we have control to not make overlapping
        // ranges.
        continue;
```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

### Lines 447-462
```cpp
      }

      LLVM_DEBUG({
        if (!CandSize)
          dbgs() << "  Empty implicit symbol size generated for the following "
                    "symbol:"
                 << "\n"
                 << "    " << *Symbol << "\n";
      });

      Symbol->setSize(CandSize);
    }
  }
  return Error::success();
}

```
- **EN**: Implements logic around `dbgs`, `setSize`, `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `setSize`, `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 463-475
```cpp
Expected<Symbol *> COFFLinkGraphBuilder::createDefinedSymbol(
    COFFSymbolIndex SymIndex, orc::SymbolStringPtr SymbolName,
    object::COFFSymbolRef Symbol, const object::coff_section *Section) {

  if (Symbol.isCommon()) {
    // FIXME: correct alignment
    return &G->addDefinedSymbol(
        G->createZeroFillBlock(getCommonSection(), Symbol.getValue(),
                               orc::ExecutorAddr(), Symbol.getValue(), 0),
        0, SymbolName, Symbol.getValue(), Linkage::Weak, Scope::Default,
        false, false);
  }

```
- **EN**: Implements logic around `createDefinedSymbol`, `addDefinedSymbol`, `createZeroFillBlock`, `ExecutorAddr`, and 1 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createDefinedSymbol`, `addDefinedSymbol`, `createZeroFillBlock`, `ExecutorAddr`, and 1 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 476-497
```cpp
  if (Symbol.isAbsolute())
    return &G->addAbsoluteSymbol(SymbolName,
                                 orc::ExecutorAddr(Symbol.getValue()), 0,
                                 Linkage::Strong, Scope::Local, false);

  if (llvm::COFF::isReservedSectionNumber(Symbol.getSectionNumber()))
    return make_error<JITLinkError>(
        "Reserved section number used in regular symbol " +
        formatv("{0:d}", SymIndex));

  Block *B = getGraphBlock(Symbol.getSectionNumber());
  if (!B) {
    LLVM_DEBUG({
      dbgs() << "    " << SymIndex
             << ": Skipping graph symbol since section was not created for "
                "COFF symbol \""
             << SymbolName << "\" in section " << Symbol.getSectionNumber()
             << "\n";
    });
    return nullptr;
  }

```
- **EN**: Implements logic around `addAbsoluteSymbol`, `ExecutorAddr`, `make_error<JITLinkError>`, `formatv`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addAbsoluteSymbol`, `ExecutorAddr`, `make_error<JITLinkError>`, `formatv`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 498-510
```cpp
  if (Symbol.isExternal()) {
    // This is not a comdat sequence, export the symbol as it is
    if (!isComdatSection(Section)) {
      auto GSym = &G->addDefinedSymbol(
          *B, Symbol.getValue(), SymbolName, 0, Linkage::Strong, Scope::Default,
          Symbol.getComplexType() == COFF::IMAGE_SYM_DTYPE_FUNCTION, false);
      DefinedSymbols[SymbolName] = GSym;
      return GSym;
    } else {
      if (!PendingComdatExports[Symbol.getSectionNumber()])
        return make_error<JITLinkError>("No pending COMDAT export for symbol " +
                                        formatv("{0:d}", SymIndex));

```
- **EN**: Implements logic around `addDefinedSymbol`, `getValue`, `getComplexType`, `make_error<JITLinkError>`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `addDefinedSymbol`, `getValue`, `getComplexType`, `make_error<JITLinkError>`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 511-534
```cpp
      return exportCOMDATSymbol(SymIndex, SymbolName, Symbol);
    }
  }

  if (Symbol.getStorageClass() == COFF::IMAGE_SYM_CLASS_STATIC ||
      Symbol.getStorageClass() == COFF::IMAGE_SYM_CLASS_LABEL) {
    const object::coff_aux_section_definition *Definition =
        Symbol.getSectionDefinition();
    if (!Definition || !isComdatSection(Section)) {
      // Handle typical static symbol
      return &G->addDefinedSymbol(
          *B, Symbol.getValue(), SymbolName, 0, Linkage::Strong, Scope::Local,
          Symbol.getComplexType() == COFF::IMAGE_SYM_DTYPE_FUNCTION, false);
    }
    if (Definition->Selection == COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE) {
      auto Target = Definition->getNumber(Symbol.isBigObj());
      auto GSym = &G->addDefinedSymbol(
          *B, Symbol.getValue(), SymbolName, 0, Linkage::Strong, Scope::Local,
          Symbol.getComplexType() == COFF::IMAGE_SYM_DTYPE_FUNCTION, false);
      getGraphBlock(Target)->addEdge(Edge::KeepAlive, 0, *GSym, 0);
      return GSym;
    }
    if (PendingComdatExports[Symbol.getSectionNumber()])
      return make_error<JITLinkError>(
```
- **EN**: Implements logic around `exportCOMDATSymbol`, `getStorageClass`, `getSectionDefinition`, `addDefinedSymbol`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `exportCOMDATSymbol`, `getStorageClass`, `getSectionDefinition`, `addDefinedSymbol`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 535-558
```cpp
          "COMDAT export request already exists before symbol " +
          formatv("{0:d}", SymIndex));
    return createCOMDATExportRequest(SymIndex, Symbol, Definition);
  }
  return make_error<JITLinkError>("Unsupported storage class " +
                                  formatv("{0:d}", Symbol.getStorageClass()) +
                                  " in symbol " + formatv("{0:d}", SymIndex));
}

// COMDAT handling:
// When IMAGE_SCN_LNK_COMDAT flag is set in the flags of a section,
// the section is called a COMDAT section. It contains two symbols
// in a sequence that specifes the behavior. First symbol is the section
// symbol which contains the size and name of the section. It also contains
// selection type that specifies how duplicate of the symbol is handled.
// Second symbol is COMDAT symbol which usually defines the external name and
// data type.
//
// Since two symbols always come in a specific order, we initiate pending COMDAT
// export request when we encounter the first symbol and actually exports it
// when we process the second symbol.
//
// Process the first symbol of COMDAT sequence.
Expected<Symbol *> COFFLinkGraphBuilder::createCOMDATExportRequest(
```
- **EN**: Implements logic around `formatv`, `createCOMDATExportRequest`, `make_error<JITLinkError>`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `formatv`, `createCOMDATExportRequest`, `make_error<JITLinkError>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 559-582
```cpp
    COFFSymbolIndex SymIndex, object::COFFSymbolRef Symbol,
    const object::coff_aux_section_definition *Definition) {
  Linkage L = Linkage::Strong;
  switch (Definition->Selection) {
  case COFF::IMAGE_COMDAT_SELECT_NODUPLICATES: {
    L = Linkage::Strong;
    break;
  }
  case COFF::IMAGE_COMDAT_SELECT_ANY: {
    L = Linkage::Weak;
    break;
  }
  case COFF::IMAGE_COMDAT_SELECT_EXACT_MATCH:
  case COFF::IMAGE_COMDAT_SELECT_SAME_SIZE: {
    // FIXME: Implement size/content validation when LinkGraph is able to
    // handle this.
    L = Linkage::Weak;
    break;
  }
  case COFF::IMAGE_COMDAT_SELECT_LARGEST: {
    // FIXME: Support IMAGE_COMDAT_SELECT_LARGEST properly when LinkGraph is
    // able to handle this.
    LLVM_DEBUG({
      dbgs() << "    " << SymIndex
```
- **EN**: Implements logic around `dbgs`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `dbgs` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 583-605
```cpp
             << ": Partially supported IMAGE_COMDAT_SELECT_LARGEST was used"
                " in section "
             << Symbol.getSectionNumber() << " (size: " << Definition->Length
             << ")\n";
    });
    L = Linkage::Weak;
    break;
  }
  case COFF::IMAGE_COMDAT_SELECT_NEWEST: {
    // Even link.exe doesn't support this selection properly.
    return make_error<JITLinkError>(
        "IMAGE_COMDAT_SELECT_NEWEST is not supported.");
  }
  default: {
    return make_error<JITLinkError>("Invalid comdat selection type: " +
                                    formatv("{0:d}", Definition->Selection));
  }
  }
  PendingComdatExports[Symbol.getSectionNumber()] = {SymIndex, L,
                                                     Definition->Length};
  return nullptr;
}

```
- **EN**: Implements logic around `getSectionNumber`, `make_error<JITLinkError>`, `formatv`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionNumber`, `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 606-629
```cpp
// Process the second symbol of COMDAT sequence.
Expected<Symbol *>
COFFLinkGraphBuilder::exportCOMDATSymbol(COFFSymbolIndex SymIndex,
                                         orc::SymbolStringPtr SymbolName,
                                         object::COFFSymbolRef Symbol) {
  Block *B = getGraphBlock(Symbol.getSectionNumber());
  auto &PendingComdatExport = PendingComdatExports[Symbol.getSectionNumber()];
  // NOTE: ComdatDef->Length is the size of "section" not size of symbol.
  // We use zero symbol size to not reach out of bound of block when symbol
  // offset is non-zero.
  auto GSym = &G->addDefinedSymbol(
      *B, Symbol.getValue(), SymbolName, 0, PendingComdatExport->Linkage,
      Scope::Default, Symbol.getComplexType() == COFF::IMAGE_SYM_DTYPE_FUNCTION,
      false);
  LLVM_DEBUG({
    dbgs() << "    " << SymIndex
           << ": Exporting COMDAT graph symbol for COFF symbol \"" << SymbolName
           << "\" in section " << Symbol.getSectionNumber() << "\n";
    dbgs() << "      " << *GSym << "\n";
  });
  setGraphSymbol(Symbol.getSectionNumber(), PendingComdatExport->SymbolIndex,
                 *GSym);
  DefinedSymbols[SymbolName] = GSym;
  PendingComdatExport = std::nullopt;
```
- **EN**: Implements logic around `exportCOMDATSymbol`, `getGraphBlock`, `getSectionNumber`, `addDefinedSymbol`, and 4 more symbols; this block coordinates ORC symbol lookup or materialization state; manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `exportCOMDATSymbol`, `getGraphBlock`, `getSectionNumber`, `addDefinedSymbol`, and 4 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，操作 JITLink 图结构或链接阶段 pass。

### Lines 630-634
```cpp
  return GSym;
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

- **Direct includes / 直接包含**: `COFFLinkGraphBuilder.h`, `memory`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
