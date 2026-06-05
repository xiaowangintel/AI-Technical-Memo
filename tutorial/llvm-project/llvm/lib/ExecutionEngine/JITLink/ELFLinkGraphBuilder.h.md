# ELFLinkGraphBuilder.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELFLinkGraphBuilder.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements ELF LinkGraph builder.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===------- ELFLinkGraphBuilder.h - ELF LinkGraph builder ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Generic ELF LinkGraph building code.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 13-26
```cpp
#ifndef LIB_EXECUTIONENGINE_JITLINK_ELFLINKGRAPHBUILDER_H
#define LIB_EXECUTIONENGINE_JITLINK_ELFLINKGRAPHBUILDER_H

#include "llvm/ExecutionEngine/JITLink/JITLink.h"
#include "llvm/Object/ELF.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Error.h"
#include "llvm/Support/FormatVariadic.h"

#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/ELF.h`, `llvm/Support/Debug.h`, `llvm/Support/Error.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/ELF.h`, `llvm/Support/Debug.h`, `llvm/Support/Error.h`。

### Lines 27-44
```cpp
/// Common link-graph building code shared between all ELFFiles.
class ELFLinkGraphBuilderBase {
public:
  ELFLinkGraphBuilderBase(std::unique_ptr<LinkGraph> G) : G(std::move(G)) {}
  virtual ~ELFLinkGraphBuilderBase();

protected:
  static bool isDwarfSection(StringRef SectionName) {
    return llvm::is_contained(DwarfSectionNames, SectionName);
  }

  Section &getCommonSection() {
    if (!CommonSection)
      CommonSection = &G->createSection(
          CommonSectionName, orc::MemProt::Read | orc::MemProt::Write);
    return *CommonSection;
  }

```
- **EN**: Introduces declarations for `ELFLinkGraphBuilderBase`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilderBase` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-59
```cpp
  std::unique_ptr<LinkGraph> G;

private:
  static StringRef CommonSectionName;
  static ArrayRef<const char *> DwarfSectionNames;

  Section *CommonSection = nullptr;
};

/// LinkGraph building code that's specific to the given ELFT, but common
/// across all architectures.
template <typename ELFT>
class ELFLinkGraphBuilder : public ELFLinkGraphBuilderBase {
  using ELFFile = object::ELFFile<ELFT>;

```
- **EN**: Introduces declarations for `ELFLinkGraphBuilder`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFLinkGraphBuilder` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 60-73
```cpp
public:
  ELFLinkGraphBuilder(const object::ELFFile<ELFT> &Obj,
                      std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
                      SubtargetFeatures Features, StringRef FileName,
                      LinkGraph::GetEdgeKindNameFunction GetEdgeKindName);

  /// Debug sections are included in the graph by default. Use
  /// setProcessDebugSections(false) to ignore them if debug info is not
  /// needed.
  ELFLinkGraphBuilder &setProcessDebugSections(bool ProcessDebugSections) {
    this->ProcessDebugSections = ProcessDebugSections;
    return *this;
  }

```
- **EN**: Implements logic around `ELFLinkGraphBuilder`, `setProcessDebugSections`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `ELFLinkGraphBuilder`, `setProcessDebugSections` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 74-88
```cpp
  /// Attempt to construct and return the LinkGraph.
  Expected<std::unique_ptr<LinkGraph>> buildGraph();

  /// Call to derived class to handle relocations. These require
  /// architecture specific knowledge to map to JITLink edge kinds.
  virtual Error addRelocations() = 0;

protected:
  using ELFSectionIndex = unsigned;
  using ELFSymbolIndex = unsigned;

  bool isRelocatable() const {
    return Obj.getHeader().e_type == llvm::ELF::ET_REL;
  }

```
- **EN**: Introduces declarations for `to`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `to` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 89-102
```cpp
  void setGraphBlock(ELFSectionIndex SecIndex, Block *B) {
    assert(!GraphBlocks.count(SecIndex) && "Duplicate section at index");
    GraphBlocks[SecIndex] = B;
  }

  Block *getGraphBlock(ELFSectionIndex SecIndex) {
    return GraphBlocks.lookup(SecIndex);
  }

  void setGraphSymbol(ELFSymbolIndex SymIndex, Symbol &Sym) {
    assert(!GraphSymbols.count(SymIndex) && "Duplicate symbol at index");
    GraphSymbols[SymIndex] = &Sym;
  }

```
- **EN**: Implements logic around `setGraphBlock`, `assert`, `getGraphBlock`, `lookup`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setGraphBlock`, `assert`, `getGraphBlock`, `lookup`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 103-114
```cpp
  Symbol *getGraphSymbol(ELFSymbolIndex SymIndex) {
    return GraphSymbols.lookup(SymIndex);
  }

  Expected<std::pair<Linkage, Scope>>
  getSymbolLinkageAndScope(const typename ELFT::Sym &Sym, StringRef Name);

  /// Set the target flags on the given Symbol.
  virtual TargetFlagsType makeTargetFlags(const typename ELFT::Sym &Sym) {
    return TargetFlagsType{};
  }

```
- **EN**: Implements logic around `getGraphSymbol`, `lookup`, `getSymbolLinkageAndScope`, `makeTargetFlags`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphSymbol`, `lookup`, `getSymbolLinkageAndScope`, `makeTargetFlags` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 115-126
```cpp
  /// Get the physical offset of the symbol on the target platform.
  virtual orc::ExecutorAddrDiff getRawOffset(const typename ELFT::Sym &Sym,
                                             TargetFlagsType Flags) {
    return Sym.getValue();
  }

  Error prepare();
  Error graphifySections();
  Error graphifySymbols();

  /// Override in derived classes to suppress certain sections in the link
  /// graph.
```
- **EN**: Implements logic around `getRawOffset`, `getValue`, `prepare`, `graphifySections`, and 1 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getRawOffset`, `getValue`, `prepare`, `graphifySections`, and 1 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 127-139
```cpp
  virtual bool excludeSection(const typename ELFT::Shdr &Sect) const {
    return false;
  }

  /// Traverse all matching ELFT::Rela relocation records in the given section.
  /// The handler function Func should be callable with this signature:
  ///   Error(const typename ELFT::Rela &,
  ///         const typename ELFT::Shdr &, Section &)
  ///
  template <typename RelocHandlerMethod>
  Error forEachRelaRelocation(const typename ELFT::Shdr &RelSect,
                              RelocHandlerMethod &&Func);

```
- **EN**: Implements logic around `excludeSection`, `forEachRelaRelocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `excludeSection`, `forEachRelaRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 140-151
```cpp
  /// Traverse all matching ELFT::Rel relocation records in the given section.
  /// The handler function Func should be callable with this signature:
  ///   Error(const typename ELFT::Rel &,
  ///         const typename ELFT::Shdr &, Section &)
  ///
  template <typename RelocHandlerMethod>
  Error forEachRelRelocation(const typename ELFT::Shdr &RelSect,
                             RelocHandlerMethod &&Func);

  /// Traverse all matching rela relocation records in the given section.
  /// Convenience wrapper to allow passing a member function for the handler.
  ///
```
- **EN**: Implements logic around `forEachRelRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `forEachRelRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 152-163
```cpp
  template <typename ClassT, typename RelocHandlerMethod>
  Error forEachRelaRelocation(const typename ELFT::Shdr &RelSect,
                              ClassT *Instance, RelocHandlerMethod &&Method) {
    return forEachRelaRelocation(
        RelSect,
        [Instance, Method](const auto &Rel, const auto &Target, auto &GS) {
          return (Instance->*Method)(Rel, Target, GS);
        });
  }

  /// Traverse all matching rel relocation records in the given section.
  /// Convenience wrapper to allow passing a member function for the handler.
```
- **EN**: Implements logic around `forEachRelaRelocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `forEachRelaRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 164-176
```cpp
  ///
  template <typename ClassT, typename RelocHandlerMethod>
  Error forEachRelRelocation(const typename ELFT::Shdr &RelSect,
                             ClassT *Instance, RelocHandlerMethod &&Method) {
    return forEachRelRelocation(
        RelSect,
        [Instance, Method](const auto &Rel, const auto &Target, auto &GS) {
          return (Instance->*Method)(Rel, Target, GS);
        });
  }

  const ELFFile &Obj;

```
- **EN**: Implements logic around `forEachRelRelocation`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `forEachRelRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 177-190
```cpp
  typename ELFFile::Elf_Shdr_Range Sections;
  const typename ELFFile::Elf_Shdr *SymTabSec = nullptr;
  StringRef SectionStringTab;
  bool ProcessDebugSections = true;

  // Maps ELF section indexes to LinkGraph Blocks.
  // Only SHF_ALLOC sections will have graph blocks.
  DenseMap<ELFSectionIndex, Block *> GraphBlocks;
  DenseMap<ELFSymbolIndex, Symbol *> GraphSymbols;
  DenseMap<const typename ELFFile::Elf_Shdr *,
           ArrayRef<typename ELFFile::Elf_Word>>
      ShndxTables;
};

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 191-203
```cpp
template <typename ELFT>
ELFLinkGraphBuilder<ELFT>::ELFLinkGraphBuilder(
    const ELFFile &Obj, std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
    SubtargetFeatures Features, StringRef FileName,
    LinkGraph::GetEdgeKindNameFunction GetEdgeKindName)
    : ELFLinkGraphBuilderBase(std::make_unique<LinkGraph>(
          FileName.str(), std::move(SSP), std::move(TT), std::move(Features),
          std::move(GetEdgeKindName))),
      Obj(Obj) {
  LLVM_DEBUG(
      { dbgs() << "Created ELFLinkGraphBuilder for \"" << FileName << "\""; });
}

```
- **EN**: Implements logic around `ELFLinkGraphBuilder`, `ELFLinkGraphBuilderBase`, `str`, `move`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `ELFLinkGraphBuilder`, `ELFLinkGraphBuilderBase`, `str`, `move`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 204-217
```cpp
template <typename ELFT>
Expected<std::unique_ptr<LinkGraph>> ELFLinkGraphBuilder<ELFT>::buildGraph() {
  if (!isRelocatable())
    return make_error<JITLinkError>("Object is not a relocatable ELF file");

  if (auto Err = prepare())
    return std::move(Err);

  if (auto Err = graphifySections())
    return std::move(Err);

  if (auto Err = graphifySymbols())
    return std::move(Err);

```
- **EN**: Implements logic around `buildGraph`, `make_error<JITLinkError>`, `move`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `buildGraph`, `make_error<JITLinkError>`, `move` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 218-230
```cpp
  if (auto Err = addRelocations())
    return std::move(Err);

  return std::move(G);
}

template <typename ELFT>
Expected<std::pair<Linkage, Scope>>
ELFLinkGraphBuilder<ELFT>::getSymbolLinkageAndScope(
    const typename ELFT::Sym &Sym, StringRef Name) {
  Linkage L = Linkage::Strong;
  Scope S = Scope::Default;

```
- **EN**: Implements logic around `move`, `getSymbolLinkageAndScope`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `move`, `getSymbolLinkageAndScope` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 231-248
```cpp
  switch (Sym.getBinding()) {
  case ELF::STB_LOCAL:
    S = Scope::Local;
    break;
  case ELF::STB_GLOBAL:
    // Nothing to do here.
    break;
  case ELF::STB_WEAK:
  case ELF::STB_GNU_UNIQUE:
    L = Linkage::Weak;
    break;
  default:
    return make_error<StringError>(
        "Unrecognized symbol binding " +
            Twine(static_cast<int>(Sym.getBinding())) + " for " + Name,
        inconvertibleErrorCode());
  }

```
- **EN**: Implements logic around `make_error<StringError>`, `Twine`, `inconvertibleErrorCode`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `Twine`, `inconvertibleErrorCode` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 249-267
```cpp
  switch (Sym.getVisibility()) {
  case ELF::STV_DEFAULT:
  case ELF::STV_PROTECTED:
    // FIXME: Make STV_DEFAULT symbols pre-emptible? This probably needs
    // Orc support.
    // Otherwise nothing to do here.
    break;
  case ELF::STV_HIDDEN:
    // Default scope -> Hidden scope. No effect on local scope.
    if (S == Scope::Default)
      S = Scope::Hidden;
    break;
  case ELF::STV_INTERNAL:
    return make_error<StringError>(
        "Unrecognized symbol visibility " +
            Twine(static_cast<int>(Sym.getVisibility())) + " for " + Name,
        inconvertibleErrorCode());
  }

```
- **EN**: Implements logic around `make_error<StringError>`, `Twine`, `inconvertibleErrorCode`; this block uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<StringError>`, `Twine`, `inconvertibleErrorCode` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 268-279
```cpp
  return std::make_pair(L, S);
}

template <typename ELFT> Error ELFLinkGraphBuilder<ELFT>::prepare() {
  LLVM_DEBUG(dbgs() << "  Preparing to build...\n");

  // Get the sections array.
  if (auto SectionsOrErr = Obj.sections())
    Sections = *SectionsOrErr;
  else
    return SectionsOrErr.takeError();

```
- **EN**: Implements logic around `make_pair`, `prepare`, `takeError`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `make_pair`, `prepare`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 280-295
```cpp
  // Get the section string table.
  if (auto SectionStringTabOrErr = Obj.getSectionStringTable(Sections))
    SectionStringTab = *SectionStringTabOrErr;
  else
    return SectionStringTabOrErr.takeError();

  // Get the SHT_SYMTAB section.
  for (auto &Sec : Sections) {
    if (Sec.sh_type == ELF::SHT_SYMTAB) {
      if (!SymTabSec)
        SymTabSec = &Sec;
      else
        return make_error<JITLinkError>("Multiple SHT_SYMTAB sections in " +
                                        G->getName());
    }

```
- **EN**: Implements logic around `takeError`, `make_error<JITLinkError>`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `make_error<JITLinkError>`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 296-309
```cpp
    // Extended table.
    if (Sec.sh_type == ELF::SHT_SYMTAB_SHNDX) {
      uint32_t SymtabNdx = Sec.sh_link;
      if (SymtabNdx >= Sections.size())
        return make_error<JITLinkError>("sh_link is out of bound");

      auto ShndxTable = Obj.getSHNDXTable(Sec);
      if (!ShndxTable)
        return ShndxTable.takeError();

      ShndxTables.insert({&Sections[SymtabNdx], *ShndxTable});
    }
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getSHNDXTable`, `takeError`, `insert`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getSHNDXTable`, `takeError`, `insert` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 310-332
```cpp
  return Error::success();
}

template <typename ELFT> Error ELFLinkGraphBuilder<ELFT>::graphifySections() {
  LLVM_DEBUG(dbgs() << "  Creating graph sections...\n");

  // For each section...
  for (ELFSectionIndex SecIndex = 0; SecIndex != Sections.size(); ++SecIndex) {

    auto &Sec = Sections[SecIndex];

    // Start by getting the section name.
    auto Name = Obj.getSectionName(Sec, SectionStringTab);
    if (!Name)
      return Name.takeError();
    if (excludeSection(Sec)) {
      LLVM_DEBUG({
        dbgs() << "    " << SecIndex << ": Skipping section \"" << *Name
               << "\" explicitly\n";
      });
      continue;
    }

```
- **EN**: Implements logic around `success`, `graphifySections`, `getSectionName`, `takeError`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `graphifySections`, `getSectionName`, `takeError`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 333-351
```cpp
    // Skip null sections.
    if (Sec.sh_type == ELF::SHT_NULL) {
      LLVM_DEBUG({
        dbgs() << "    " << SecIndex << ": has type SHT_NULL. Skipping.\n";
      });
      continue;
    }

    // If the name indicates that it's a debug section then skip it: We don't
    // support those yet.
    if (!ProcessDebugSections && isDwarfSection(*Name)) {
      LLVM_DEBUG({
        dbgs() << "    " << SecIndex << ": \"" << *Name
               << "\" is a debug section: "
                  "No graph section will be created.\n";
      });
      continue;
    }

```
- **EN**: Implements logic around `dbgs`.
- **CN**: 围绕 `dbgs` 实现具体逻辑。

### Lines 352-363
```cpp
    LLVM_DEBUG({
      dbgs() << "    " << SecIndex << ": Creating section for \"" << *Name
             << "\"\n";
    });

    // Get the section's memory protection flags.
    orc::MemProt Prot = orc::MemProt::Read;
    if (Sec.sh_flags & ELF::SHF_EXECINSTR)
      Prot |= orc::MemProt::Exec;
    if (Sec.sh_flags & ELF::SHF_WRITE)
      Prot |= orc::MemProt::Write;

```
- **EN**: Implements logic around `dbgs`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `dbgs` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 364-377
```cpp
    // Look for existing sections first.
    auto *GraphSec = G->findSectionByName(*Name);
    if (!GraphSec) {
      GraphSec = &G->createSection(*Name, Prot);
      // Non-SHF_ALLOC sections get NoAlloc memory lifetimes.
      if (!(Sec.sh_flags & ELF::SHF_ALLOC)) {
        GraphSec->setMemLifetime(orc::MemLifetime::NoAlloc);
        LLVM_DEBUG({
          dbgs() << "      " << SecIndex << ": \"" << *Name
                 << "\" is not a SHF_ALLOC section. Using NoAlloc lifetime.\n";
        });
      }
    }

```
- **EN**: Implements logic around `findSectionByName`, `createSection`, `setMemLifetime`, `dbgs`.
- **CN**: 围绕 `findSectionByName`, `createSection`, `setMemLifetime`, `dbgs` 实现具体逻辑。

### Lines 378-392
```cpp
    if (GraphSec->getMemProt() != Prot) {
      std::string ErrMsg;
      raw_string_ostream(ErrMsg)
          << "In " << G->getName() << ", section " << *Name
          << " is present more than once with different permissions: "
          << GraphSec->getMemProt() << " vs " << Prot;
      return make_error<JITLinkError>(std::move(ErrMsg));
    }

    Block *B = nullptr;
    if (Sec.sh_type != ELF::SHT_NOBITS) {
      auto Data = Obj.template getSectionContentsAsArray<char>(Sec);
      if (!Data)
        return Data.takeError();

```
- **EN**: Implements logic around `raw_string_ostream`, `getName`, `getMemProt`, `make_error<JITLinkError>`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `raw_string_ostream`, `getName`, `getMemProt`, `make_error<JITLinkError>`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 393-406
```cpp
      B = &G->createContentBlock(*GraphSec, *Data,
                                 orc::ExecutorAddr(Sec.sh_addr),
                                 Sec.sh_addralign, 0);
    } else
      B = &G->createZeroFillBlock(*GraphSec, Sec.sh_size,
                                  orc::ExecutorAddr(Sec.sh_addr),
                                  Sec.sh_addralign, 0);

    if (Sec.sh_type == ELF::SHT_ARM_EXIDX) {
      // Add live symbol to avoid dead-stripping for .ARM.exidx sections
      G->addAnonymousSymbol(*B, orc::ExecutorAddrDiff(),
                            orc::ExecutorAddrDiff(), false, true);
    }

```
- **EN**: Implements logic around `createContentBlock`, `ExecutorAddr`, `createZeroFillBlock`, `addAnonymousSymbol`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createContentBlock`, `ExecutorAddr`, `createZeroFillBlock`, `addAnonymousSymbol`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 407-419
```cpp
    setGraphBlock(SecIndex, B);
  }

  return Error::success();
}

template <typename ELFT> Error ELFLinkGraphBuilder<ELFT>::graphifySymbols() {
  LLVM_DEBUG(dbgs() << "  Creating graph symbols...\n");

  // No SYMTAB -- Bail out early.
  if (!SymTabSec)
    return Error::success();

```
- **EN**: Implements logic around `setGraphBlock`, `success`, `graphifySymbols`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `setGraphBlock`, `success`, `graphifySymbols` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 420-432
```cpp
  // Get the section content as a Symbols array.
  auto Symbols = Obj.symbols(SymTabSec);
  if (!Symbols)
    return Symbols.takeError();

  // Get the string table for this section.
  auto StringTab = Obj.getStringTableForSymtab(*SymTabSec, Sections);
  if (!StringTab)
    return StringTab.takeError();

  LLVM_DEBUG({
    StringRef SymTabName;

```
- **EN**: Implements logic around `symbols`, `takeError`, `getStringTableForSymtab`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `symbols`, `takeError`, `getStringTableForSymtab` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 433-444
```cpp
    if (auto SymTabNameOrErr = Obj.getSectionName(*SymTabSec, SectionStringTab))
      SymTabName = *SymTabNameOrErr;
    else {
      dbgs() << "Could not get ELF SHT_SYMTAB section name for logging: "
             << toString(SymTabNameOrErr.takeError()) << "\n";
      SymTabName = "<SHT_SYMTAB section with invalid name>";
    }

    dbgs() << "    Adding symbols from symtab section \"" << SymTabName
           << "\"\n";
  });

```
- **EN**: Implements logic around `dbgs`, `toString`.
- **CN**: 围绕 `dbgs`, `toString` 实现具体逻辑。

### Lines 445-465
```cpp
  for (ELFSymbolIndex SymIndex = 0; SymIndex != Symbols->size(); ++SymIndex) {
    auto &Sym = (*Symbols)[SymIndex];

    // Check symbol type.
    switch (Sym.getType()) {
    case ELF::STT_FILE:
      LLVM_DEBUG({
        if (auto Name = Sym.getName(*StringTab))
          dbgs() << "      " << SymIndex << ": Skipping STT_FILE symbol \""
                 << *Name << "\"\n";
        else {
          dbgs() << "Could not get STT_FILE symbol name: "
                 << toString(Name.takeError()) << "\n";
          dbgs() << "     " << SymIndex
                 << ": Skipping STT_FILE symbol with invalid name\n";
        }
      });
      continue;
      break;
    }

```
- **EN**: Implements logic around `dbgs`, `toString`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `dbgs`, `toString` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 466-480
```cpp
    // Get the symbol name.
    auto Name = Sym.getName(*StringTab);
    if (!Name)
      return Name.takeError();

    // Handle common symbols specially.
    if (Sym.isCommon()) {
      Symbol &GSym = G->addDefinedSymbol(
          G->createZeroFillBlock(getCommonSection(), Sym.st_size,
                                 orc::ExecutorAddr(), Sym.getValue(), 0),
          0, *Name, Sym.st_size, Linkage::Weak, Scope::Default, false, false);
      setGraphSymbol(SymIndex, GSym);
      continue;
    }

```
- **EN**: Implements logic around `getName`, `takeError`, `addDefinedSymbol`, `createZeroFillBlock`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `takeError`, `addDefinedSymbol`, `createZeroFillBlock`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 481-493
```cpp
    if (Sym.isDefined() &&
        (Sym.getType() == ELF::STT_NOTYPE || Sym.getType() == ELF::STT_FUNC ||
         Sym.getType() == ELF::STT_OBJECT ||
         Sym.getType() == ELF::STT_SECTION || Sym.getType() == ELF::STT_TLS)) {

      // Map Visibility and Binding to Scope and Linkage:
      Linkage L;
      Scope S;
      if (auto LSOrErr = getSymbolLinkageAndScope(Sym, *Name))
        std::tie(L, S) = *LSOrErr;
      else
        return LSOrErr.takeError();

```
- **EN**: Implements logic around `getType`, `tie`, `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getType`, `tie`, `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 494-512
```cpp
      // Handle extended tables.
      unsigned Shndx = Sym.st_shndx;
      if (Shndx == ELF::SHN_XINDEX) {
        auto ShndxTable = ShndxTables.find(SymTabSec);
        if (ShndxTable == ShndxTables.end())
          continue;
        auto NdxOrErr = object::getExtendedSymbolTableIndex<ELFT>(
            Sym, SymIndex, ShndxTable->second);
        if (!NdxOrErr)
          return NdxOrErr.takeError();
        Shndx = *NdxOrErr;
      }
      if (auto *B = getGraphBlock(Shndx)) {
        LLVM_DEBUG({
          dbgs() << "      " << SymIndex
                 << ": Creating defined graph symbol for ELF symbol \"" << *Name
                 << "\"\n";
        });

```
- **EN**: Implements logic around `find`, `getExtendedSymbolTableIndex<ELFT>`, `takeError`, `dbgs`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `getExtendedSymbolTableIndex<ELFT>`, `takeError`, `dbgs` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 513-531
```cpp
        TargetFlagsType Flags = makeTargetFlags(Sym);
        orc::ExecutorAddrDiff Offset = getRawOffset(Sym, Flags);

        if (Offset + Sym.st_size > B->getSize()) {
          std::string ErrMsg;
          raw_string_ostream ErrStream(ErrMsg);
          ErrStream << "In " << G->getName() << ", symbol ";
          if (!Name->empty())
            ErrStream << *Name;
          else
            ErrStream << "<anon>";
          ErrStream << " (" << (B->getAddress() + Offset) << " -- "
                    << (B->getAddress() + Offset + Sym.st_size) << ") extends "
                    << formatv("{0:x}", Offset + Sym.st_size - B->getSize())
                    << " bytes past the end of its containing block ("
                    << B->getRange() << ")";
          return make_error<JITLinkError>(std::move(ErrMsg));
        }

```
- **EN**: Implements logic around `makeTargetFlags`, `getRawOffset`, `ErrStream`, `getName`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `makeTargetFlags`, `getRawOffset`, `ErrStream`, `getName`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 532-544
```cpp
        // In RISCV, temporary symbols (Used to generate dwarf, eh_frame
        // sections...) will appear in object code's symbol table, and LLVM does
        // not use names on these temporary symbols (RISCV gnu toolchain uses
        // names on these temporary symbols). If the symbol is unnamed, add an
        // anonymous symbol.
        auto &GSym =
            Name->empty()
                ? G->addAnonymousSymbol(*B, Offset, Sym.st_size,
                                        false, false)
                : G->addDefinedSymbol(*B, Offset, *Name, Sym.st_size, L,
                                      S, Sym.getType() == ELF::STT_FUNC,
                                      false);

```
- **EN**: Implements logic around `empty`, `addAnonymousSymbol`, `addDefinedSymbol`, `getType`.
- **CN**: 围绕 `empty`, `addAnonymousSymbol`, `addDefinedSymbol`, `getType` 实现具体逻辑。

### Lines 545-562
```cpp
        GSym.setTargetFlags(Flags);
        setGraphSymbol(SymIndex, GSym);
      }
    } else if (Sym.isUndefined() && Sym.isExternal()) {
      LLVM_DEBUG({
        dbgs() << "      " << SymIndex
               << ": Creating external graph symbol for ELF symbol \"" << *Name
               << "\"\n";
      });

      if (Sym.getBinding() != ELF::STB_GLOBAL &&
          Sym.getBinding() != ELF::STB_WEAK)
        return make_error<StringError>(
            "Invalid symbol binding " +
                Twine(static_cast<int>(Sym.getBinding())) +
                " for external symbol " + *Name,
            inconvertibleErrorCode());

```
- **EN**: Implements logic around `setTargetFlags`, `setGraphSymbol`, `dbgs`, `getBinding`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setTargetFlags`, `setGraphSymbol`, `dbgs`, `getBinding`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 563-575
```cpp
      // If L is Linkage::Weak that means this is a weakly referenced symbol.
      auto &GSym = G->addExternalSymbol(*Name, Sym.st_size,
                                        Sym.getBinding() == ELF::STB_WEAK);
      setGraphSymbol(SymIndex, GSym);
    } else if (Sym.isUndefined() && Sym.st_value == 0 && Sym.st_size == 0 &&
               Sym.getType() == ELF::STT_NOTYPE &&
               Sym.getBinding() == ELF::STB_LOCAL && Name->empty()) {
      // Some relocations (e.g., R_RISCV_ALIGN) don't have a target symbol and
      // use this kind of null symbol as a placeholder.
      LLVM_DEBUG({
        dbgs() << "      " << SymIndex << ": Creating null graph symbol\n";
      });

```
- **EN**: Implements logic around `addExternalSymbol`, `getBinding`, `setGraphSymbol`, `getType`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `addExternalSymbol`, `getBinding`, `setGraphSymbol`, `getType`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 576-590
```cpp
      auto SymName =
          G->allocateContent("__jitlink_ELF_SYM_UND_" + Twine(SymIndex));
      auto SymNameRef = StringRef(SymName.data(), SymName.size());
      auto &GSym = G->addAbsoluteSymbol(SymNameRef, orc::ExecutorAddr(0), 0,
                                        Linkage::Strong, Scope::Local, false);
      setGraphSymbol(SymIndex, GSym);
    } else {
      LLVM_DEBUG({
        dbgs() << "      " << SymIndex
               << ": Not creating graph symbol for ELF symbol \"" << *Name
               << "\" with unrecognized type\n";
      });
    }
  }

```
- **EN**: Implements logic around `allocateContent`, `StringRef`, `addAbsoluteSymbol`, `setGraphSymbol`, and 1 more symbols.
- **CN**: 围绕 `allocateContent`, `StringRef`, `addAbsoluteSymbol`, `setGraphSymbol`, and 1 more symbols 实现具体逻辑。

### Lines 591-607
```cpp
  return Error::success();
}

template <typename ELFT>
template <typename RelocHandlerFunction>
Error ELFLinkGraphBuilder<ELFT>::forEachRelaRelocation(
    const typename ELFT::Shdr &RelSect, RelocHandlerFunction &&Func) {
  // Only look into sections that store relocation entries.
  if (RelSect.sh_type != ELF::SHT_RELA)
    return Error::success();

  // sh_info contains the section header index of the target (FixupSection),
  // which is the section to which all relocations in RelSect apply.
  auto FixupSection = Obj.getSection(RelSect.sh_info);
  if (!FixupSection)
    return FixupSection.takeError();

```
- **EN**: Implements logic around `success`, `forEachRelaRelocation`, `getSection`, `takeError`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `forEachRelaRelocation`, `getSection`, `takeError` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 608-623
```cpp
  // Target sections have names in valid ELF object files.
  Expected<StringRef> Name = Obj.getSectionName(**FixupSection);
  if (!Name)
    return Name.takeError();
  LLVM_DEBUG(dbgs() << "  " << *Name << ":\n");

  // Consider skipping these relocations.
  if (!ProcessDebugSections && isDwarfSection(*Name)) {
    LLVM_DEBUG(dbgs() << "    skipped (dwarf section)\n\n");
    return Error::success();
  }
  if (excludeSection(**FixupSection)) {
    LLVM_DEBUG(dbgs() << "    skipped (fixup section excluded explicitly)\n\n");
    return Error::success();
  }

```
- **EN**: Implements logic around `getSectionName`, `takeError`, `success`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionName`, `takeError`, `success` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 624-639
```cpp
  // Lookup the link-graph node corresponding to the target section name.
  auto *BlockToFix = getGraphBlock(RelSect.sh_info);
  if (!BlockToFix)
    return make_error<StringError>(
        "Refencing a section that wasn't added to the graph: " + *Name,
        inconvertibleErrorCode());

  auto RelEntries = Obj.relas(RelSect);
  if (!RelEntries)
    return RelEntries.takeError();

  // Let the callee process relocation entries one by one.
  for (const typename ELFT::Rela &R : *RelEntries)
    if (Error Err = Func(R, **FixupSection, *BlockToFix))
      return Err;

```
- **EN**: Implements logic around `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode`, `relas`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode`, `relas`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 640-651
```cpp
  LLVM_DEBUG(dbgs() << "\n");
  return Error::success();
}

template <typename ELFT>
template <typename RelocHandlerFunction>
Error ELFLinkGraphBuilder<ELFT>::forEachRelRelocation(
    const typename ELFT::Shdr &RelSect, RelocHandlerFunction &&Func) {
  // Only look into sections that store relocation entries.
  if (RelSect.sh_type != ELF::SHT_REL)
    return Error::success();

```
- **EN**: Implements logic around `success`, `forEachRelRelocation`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `forEachRelRelocation` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 652-663
```cpp
  // sh_info contains the section header index of the target (FixupSection),
  // which is the section to which all relocations in RelSect apply.
  auto FixupSection = Obj.getSection(RelSect.sh_info);
  if (!FixupSection)
    return FixupSection.takeError();

  // Target sections have names in valid ELF object files.
  Expected<StringRef> Name = Obj.getSectionName(**FixupSection);
  if (!Name)
    return Name.takeError();
  LLVM_DEBUG(dbgs() << "  " << *Name << ":\n");

```
- **EN**: Implements logic around `getSection`, `takeError`, `getSectionName`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSection`, `takeError`, `getSectionName` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 664-680
```cpp
  // Consider skipping these relocations.
  if (!ProcessDebugSections && isDwarfSection(*Name)) {
    LLVM_DEBUG(dbgs() << "    skipped (dwarf section)\n\n");
    return Error::success();
  }
  if (excludeSection(**FixupSection)) {
    LLVM_DEBUG(dbgs() << "    skipped (fixup section excluded explicitly)\n\n");
    return Error::success();
  }

  // Lookup the link-graph node corresponding to the target section name.
  auto *BlockToFix = getGraphBlock(RelSect.sh_info);
  if (!BlockToFix)
    return make_error<StringError>(
        "Refencing a section that wasn't added to the graph: " + *Name,
        inconvertibleErrorCode());

```
- **EN**: Implements logic around `success`, `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `getGraphBlock`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 681-693
```cpp
  auto RelEntries = Obj.rels(RelSect);
  if (!RelEntries)
    return RelEntries.takeError();

  // Let the callee process relocation entries one by one.
  for (const typename ELFT::Rel &R : *RelEntries)
    if (Error Err = Func(R, **FixupSection, *BlockToFix))
      return Err;

  LLVM_DEBUG(dbgs() << "\n");
  return Error::success();
}

```
- **EN**: Implements logic around `rels`, `takeError`, `success`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `rels`, `takeError`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 694-699
```cpp
} // end namespace jitlink
} // end namespace llvm

#undef DEBUG_TYPE

#endif // LIB_EXECUTIONENGINE_JITLINK_ELFLINKGRAPHBUILDER_H
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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/Object/ELF.h`, `llvm/Support/Debug.h`, `llvm/Support/Error.h`, `llvm/Support/FormatVariadic.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object, Support
