# ELF_ppc64.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/ELF_ppc64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements JITLink graph-based linking, relocation handling, and format-specific link graph passes for just-in-time compiled code.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===------- ELF_ppc64.cpp -JIT linker implementation for ELF/ppc64 -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// ELF/ppc64 jit-link implementation.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-22
```cpp

#include "llvm/ExecutionEngine/JITLink/ELF_ppc64.h"
#include "llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h"
#include "llvm/ExecutionEngine/JITLink/TableManager.h"
#include "llvm/ExecutionEngine/JITLink/ppc64.h"
#include "llvm/Object/ELFObjectFile.h"

#include "EHFrameSupportImpl.h"
#include "ELFLinkGraphBuilder.h"
#include "JITLinkGeneric.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/ELF_ppc64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/TableManager.h`, `llvm/ExecutionEngine/JITLink/ppc64.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/ELF_ppc64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/TableManager.h`, `llvm/ExecutionEngine/JITLink/ppc64.h`。

### Lines 23-34
```cpp
#define DEBUG_TYPE "jitlink"

namespace {

using namespace llvm;
using namespace llvm::jitlink;

constexpr StringRef ELFTOCSymbolName = ".TOC.";
constexpr StringRef TOCSymbolAliasIdent = "__TOC__";
constexpr uint64_t ELFTOCBaseOffset = 0x8000;
constexpr StringRef ELFTLSInfoSectionName = "$__TLSINFO";

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 35-52
```cpp
template <llvm::endianness Endianness>
class TLSInfoTableManager_ELF_ppc64
    : public TableManager<TLSInfoTableManager_ELF_ppc64<Endianness>> {
public:
  static const uint8_t TLSInfoEntryContent[16];

  static StringRef getSectionName() { return ELFTLSInfoSectionName; }

  bool visitEdge(LinkGraph &G, Block *B, Edge &E) {
    Edge::Kind K = E.getKind();
    switch (K) {
    case ppc64::RequestTLSDescInGOTAndTransformToTOCDelta16HA:
      E.setKind(ppc64::TOCDelta16HA);
      E.setTarget(this->getEntryForTarget(G, E.getTarget()));
      return true;
    case ppc64::RequestTLSDescInGOTAndTransformToTOCDelta16LO:
      E.setKind(ppc64::TOCDelta16LO);
      E.setTarget(this->getEntryForTarget(G, E.getTarget()));
```
- **EN**: Introduces declarations for `TLSInfoTableManager_ELF_ppc64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `TLSInfoTableManager_ELF_ppc64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 53-62
```cpp
      return true;
    case ppc64::RequestTLSDescInGOTAndTransformToDelta34:
      E.setKind(ppc64::Delta34);
      E.setTarget(this->getEntryForTarget(G, E.getTarget()));
      return true;
    default:
      return false;
    }
  }

```
- **EN**: Implements logic around `setKind`, `setTarget`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `setKind`, `setTarget` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 63-72
```cpp
  Symbol &createEntry(LinkGraph &G, Symbol &Target) {
    // The TLS Info entry's key value will be written by
    // `fixTLVSectionsAndEdges`, so create mutable content.
    auto &TLSInfoEntry = G.createMutableContentBlock(
        getTLSInfoSection(G), G.allocateContent(getTLSInfoEntryContent()),
        orc::ExecutorAddr(), 8, 0);
    TLSInfoEntry.addEdge(ppc64::Pointer64, 8, Target, 0);
    return G.addAnonymousSymbol(TLSInfoEntry, 0, 16, false, false);
  }

```
- **EN**: Implements logic around `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, `ExecutorAddr`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createEntry`, `createMutableContentBlock`, `getTLSInfoSection`, `ExecutorAddr`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 73-85
```cpp
private:
  Section &getTLSInfoSection(LinkGraph &G) {
    if (!TLSInfoTable)
      TLSInfoTable =
          &G.createSection(ELFTLSInfoSectionName, orc::MemProt::Read);
    return *TLSInfoTable;
  }

  ArrayRef<char> getTLSInfoEntryContent() const {
    return {reinterpret_cast<const char *>(TLSInfoEntryContent),
            sizeof(TLSInfoEntryContent)};
  }

```
- **EN**: Implements logic around `getTLSInfoSection`, `createSection`, `getTLSInfoEntryContent`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getTLSInfoSection`, `createSection`, `getTLSInfoEntryContent` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 86-95
```cpp
  Section *TLSInfoTable = nullptr;
};

template <>
const uint8_t TLSInfoTableManager_ELF_ppc64<
    llvm::endianness::little>::TLSInfoEntryContent[16] = {
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, /*pthread key */
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00  /*data address*/
};

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 96-107
```cpp
template <>
const uint8_t TLSInfoTableManager_ELF_ppc64<
    llvm::endianness::big>::TLSInfoEntryContent[16] = {
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, /*pthread key */
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00  /*data address*/
};

template <llvm::endianness Endianness>
Symbol &createELFGOTHeader(LinkGraph &G,
                           ppc64::TOCTableManager<Endianness> &TOC) {
  Symbol *TOCSymbol = nullptr;

```
- **EN**: Implements logic around `createELFGOTHeader`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createELFGOTHeader` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 108-121
```cpp
  for (Symbol *Sym : G.defined_symbols())
    if (LLVM_UNLIKELY(Sym->hasName() && *Sym->getName() == ELFTOCSymbolName)) {
      TOCSymbol = Sym;
      break;
    }

  if (LLVM_LIKELY(TOCSymbol == nullptr)) {
    for (Symbol *Sym : G.external_symbols())
      if (Sym->hasName() && *Sym->getName() == ELFTOCSymbolName) {
        TOCSymbol = Sym;
        break;
      }
  }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 122-139
```cpp
  if (!TOCSymbol)
    TOCSymbol = &G.addExternalSymbol(ELFTOCSymbolName, 0, false);

  return TOC.getEntryForTarget(G, *TOCSymbol);
}

// Register preexisting GOT entries with TOC table manager.
template <llvm::endianness Endianness>
inline void
registerExistingGOTEntries(LinkGraph &G,
                           ppc64::TOCTableManager<Endianness> &TOC) {
  auto isGOTEntry = [](const Edge &E) {
    return E.getKind() == ppc64::Pointer64 && E.getTarget().isExternal();
  };
  if (Section *dotTOCSection = G.findSectionByName(".toc")) {
    for (Block *B : dotTOCSection->blocks())
      for (Edge &E : B->edges())
        if (isGOTEntry(E))
```
- **EN**: Implements logic around `addExternalSymbol`, `getEntryForTarget`, `registerExistingGOTEntries`, `getKind`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addExternalSymbol`, `getEntryForTarget`, `registerExistingGOTEntries`, `getKind` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 140-157
```cpp
          TOC.registerPreExistingEntry(E.getTarget(),
                                       G.addAnonymousSymbol(*B, E.getOffset(),
                                                            G.getPointerSize(),
                                                            false, false));
  }
}

template <llvm::endianness Endianness>
Error buildTables_ELF_ppc64(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Visiting edges in graph:\n");
  ppc64::TOCTableManager<Endianness> TOC;
  // Before visiting edges, we create a header containing the address of TOC
  // base as ELFABIv2 suggests:
  //  > The GOT consists of an 8-byte header that contains the TOC base (the
  //  first TOC base when multiple TOCs are present), followed by an array of
  //  8-byte addresses.
  createELFGOTHeader(G, TOC);

```
- **EN**: Implements logic around `registerPreExistingEntry`, `addAnonymousSymbol`, `getPointerSize`, `buildTables_ELF_ppc64`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `registerPreExistingEntry`, `addAnonymousSymbol`, `getPointerSize`, `buildTables_ELF_ppc64`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 158-175
```cpp
  // There might be compiler-generated GOT entries in ELF relocatable file.
  registerExistingGOTEntries(G, TOC);

  ppc64::PLTTableManager<Endianness> PLT(TOC);
  TLSInfoTableManager_ELF_ppc64<Endianness> TLSInfo;
  visitExistingEdges(G, TOC, PLT, TLSInfo);

  // After visiting edges in LinkGraph, we have GOT entries built in the
  // synthesized section.
  // Merge sections included in TOC into synthesized TOC section,
  // thus TOC is compact and reducing chances of relocation
  // overflow.
  if (Section *TOCSection = G.findSectionByName(TOC.getSectionName())) {
    // .got and .plt are not normally present in a relocatable object file
    // because they are linker generated.
    if (Section *gotSection = G.findSectionByName(".got"))
      G.mergeSections(*TOCSection, *gotSection);
    if (Section *tocSection = G.findSectionByName(".toc"))
```
- **EN**: Implements logic around `registerExistingGOTEntries`, `PLT`, `visitExistingEdges`, `mergeSections`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `registerExistingGOTEntries`, `PLT`, `visitExistingEdges`, `mergeSections` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 176-188
```cpp
      G.mergeSections(*TOCSection, *tocSection);
    if (Section *sdataSection = G.findSectionByName(".sdata"))
      G.mergeSections(*TOCSection, *sdataSection);
    if (Section *sbssSection = G.findSectionByName(".sbss"))
      G.mergeSections(*TOCSection, *sbssSection);
    // .tocbss no longer appears in ELFABIv2. Leave it here to be compatible
    // with rtdyld.
    if (Section *tocbssSection = G.findSectionByName(".tocbss"))
      G.mergeSections(*TOCSection, *tocbssSection);
    if (Section *pltSection = G.findSectionByName(".plt"))
      G.mergeSections(*TOCSection, *pltSection);
  }

```
- **EN**: Implements logic around `mergeSections`.
- **CN**: 围绕 `mergeSections` 实现具体逻辑。

### Lines 189-202
```cpp
  return Error::success();
}

} // namespace

namespace llvm::jitlink {

template <llvm::endianness Endianness>
class ELFLinkGraphBuilder_ppc64
    : public ELFLinkGraphBuilder<object::ELFType<Endianness, true>> {
private:
  using ELFT = object::ELFType<Endianness, true>;
  using Base = ELFLinkGraphBuilder<ELFT>;

```
- **EN**: Introduces declarations for `llvm::jitlink`, `ELFLinkGraphBuilder_ppc64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::jitlink`, `ELFLinkGraphBuilder_ppc64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 203-216
```cpp
  using Base::G; // Use LinkGraph pointer from base class.

  Error addRelocations() override {
    LLVM_DEBUG(dbgs() << "Processing relocations:\n");

    using Self = ELFLinkGraphBuilder_ppc64<Endianness>;
    for (const auto &RelSect : Base::Sections) {
      // Validate the section to read relocation entries from.
      if (RelSect.sh_type == ELF::SHT_REL)
        return make_error<StringError>("No SHT_REL in valid " +
                                           G->getTargetTriple().getArchName() +
                                           " ELF object files",
                                       inconvertibleErrorCode());

```
- **EN**: Implements logic around `addRelocations`, `make_error<StringError>`, `getTargetTriple`, `inconvertibleErrorCode`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocations`, `make_error<StringError>`, `getTargetTriple`, `inconvertibleErrorCode` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 217-230
```cpp
      if (Error Err = Base::forEachRelaRelocation(RelSect, this,
                                                  &Self::addSingleRelocation))
        return Err;
    }

    return Error::success();
  }

  Error addSingleRelocation(const typename ELFT::Rela &Rel,
                            const typename ELFT::Shdr &FixupSection,
                            Block &BlockToFix) {
    using Base = ELFLinkGraphBuilder<ELFT>;
    auto ELFReloc = Rel.getType(false);

```
- **EN**: Implements logic around `success`, `addSingleRelocation`, `getType`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `addSingleRelocation`, `getType` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 231-241
```cpp
    // R_PPC64_NONE is a no-op.
    if (LLVM_UNLIKELY(ELFReloc == ELF::R_PPC64_NONE))
      return Error::success();

    // TLS model markers. We only support global-dynamic model now.
    if (ELFReloc == ELF::R_PPC64_TLSGD)
      return Error::success();
    if (ELFReloc == ELF::R_PPC64_TLSLD)
      return make_error<StringError>("Local-dynamic TLS model is not supported",
                                     inconvertibleErrorCode());

```
- **EN**: Implements logic around `success`, `make_error<StringError>`, `inconvertibleErrorCode`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `make_error<StringError>`, `inconvertibleErrorCode` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 242-253
```cpp
    if (ELFReloc == ELF::R_PPC64_PCREL_OPT)
      // TODO: Support PCREL optimization, now ignore it.
      return Error::success();

    if (ELFReloc == ELF::R_PPC64_TPREL34)
      return make_error<StringError>("Local-exec TLS model is not supported",
                                     inconvertibleErrorCode());

    auto ObjSymbol = Base::Obj.getRelocationSymbol(Rel, Base::SymTabSec);
    if (!ObjSymbol)
      return ObjSymbol.takeError();

```
- **EN**: Implements logic around `success`, `make_error<StringError>`, `inconvertibleErrorCode`, `getRelocationSymbol`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `make_error<StringError>`, `inconvertibleErrorCode`, `getRelocationSymbol`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 254-263
```cpp
    uint32_t SymbolIndex = Rel.getSymbol(false);
    Symbol *GraphSymbol = Base::getGraphSymbol(SymbolIndex);
    if (!GraphSymbol)
      return make_error<StringError>(
          formatv("Could not find symbol at given index, did you add it to "
                  "JITSymbolTable? index: {0}, shndx: {1} Size of table: {2}",
                  SymbolIndex, (*ObjSymbol)->st_shndx,
                  Base::GraphSymbols.size()),
          inconvertibleErrorCode());

```
- **EN**: Implements logic around `getSymbol`, `getGraphSymbol`, `make_error<StringError>`, `formatv`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbol`, `getGraphSymbol`, `make_error<StringError>`, `formatv`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 264-281
```cpp
    int64_t Addend = Rel.r_addend;
    orc::ExecutorAddr FixupAddress =
        orc::ExecutorAddr(FixupSection.sh_addr) + Rel.r_offset;
    Edge::OffsetT Offset = FixupAddress - BlockToFix.getAddress();
    Edge::Kind Kind = Edge::Invalid;

    switch (ELFReloc) {
    default:
      return make_error<JITLinkError>(
          "In " + G->getName() + ": Unsupported ppc64 relocation type " +
          object::getELFRelocationTypeName(ELF::EM_PPC64, ELFReloc));
    case ELF::R_PPC64_ADDR64:
      Kind = ppc64::Pointer64;
      break;
    case ELF::R_PPC64_ADDR32:
      Kind = ppc64::Pointer32;
      break;
    case ELF::R_PPC64_ADDR16:
```
- **EN**: Implements logic around `ExecutorAddr`, `getAddress`, `make_error<JITLinkError>`, `getName`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `ExecutorAddr`, `getAddress`, `make_error<JITLinkError>`, `getName`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 282-299
```cpp
      Kind = ppc64::Pointer16;
      break;
    case ELF::R_PPC64_ADDR16_DS:
      Kind = ppc64::Pointer16DS;
      break;
    case ELF::R_PPC64_ADDR16_HA:
      Kind = ppc64::Pointer16HA;
      break;
    case ELF::R_PPC64_ADDR16_HI:
      Kind = ppc64::Pointer16HI;
      break;
    case ELF::R_PPC64_ADDR16_HIGH:
      Kind = ppc64::Pointer16HIGH;
      break;
    case ELF::R_PPC64_ADDR16_HIGHA:
      Kind = ppc64::Pointer16HIGHA;
      break;
    case ELF::R_PPC64_ADDR16_HIGHER:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 300-317
```cpp
      Kind = ppc64::Pointer16HIGHER;
      break;
    case ELF::R_PPC64_ADDR16_HIGHERA:
      Kind = ppc64::Pointer16HIGHERA;
      break;
    case ELF::R_PPC64_ADDR16_HIGHEST:
      Kind = ppc64::Pointer16HIGHEST;
      break;
    case ELF::R_PPC64_ADDR16_HIGHESTA:
      Kind = ppc64::Pointer16HIGHESTA;
      break;
    case ELF::R_PPC64_ADDR16_LO:
      Kind = ppc64::Pointer16LO;
      break;
    case ELF::R_PPC64_ADDR16_LO_DS:
      Kind = ppc64::Pointer16LODS;
      break;
    case ELF::R_PPC64_ADDR14:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 318-335
```cpp
      Kind = ppc64::Pointer14;
      break;
    case ELF::R_PPC64_TOC:
      Kind = ppc64::TOC;
      break;
    case ELF::R_PPC64_TOC16:
      Kind = ppc64::TOCDelta16;
      break;
    case ELF::R_PPC64_TOC16_HA:
      Kind = ppc64::TOCDelta16HA;
      break;
    case ELF::R_PPC64_TOC16_HI:
      Kind = ppc64::TOCDelta16HI;
      break;
    case ELF::R_PPC64_TOC16_DS:
      Kind = ppc64::TOCDelta16DS;
      break;
    case ELF::R_PPC64_TOC16_LO:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 336-353
```cpp
      Kind = ppc64::TOCDelta16LO;
      break;
    case ELF::R_PPC64_TOC16_LO_DS:
      Kind = ppc64::TOCDelta16LODS;
      break;
    case ELF::R_PPC64_REL16:
      Kind = ppc64::Delta16;
      break;
    case ELF::R_PPC64_REL16_HA:
      Kind = ppc64::Delta16HA;
      break;
    case ELF::R_PPC64_REL16_HI:
      Kind = ppc64::Delta16HI;
      break;
    case ELF::R_PPC64_REL16_LO:
      Kind = ppc64::Delta16LO;
      break;
    case ELF::R_PPC64_REL32:
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 354-371
```cpp
      Kind = ppc64::Delta32;
      break;
    case ELF::R_PPC64_REL24_NOTOC:
      Kind = ppc64::RequestCallNoTOC;
      break;
    case ELF::R_PPC64_REL24:
      Kind = ppc64::RequestCall;
      // Determining a target is external or not is deferred in PostPrunePass.
      // We assume branching to local entry by default, since in PostPrunePass,
      // we don't have any context to determine LocalEntryOffset. If it finally
      // turns out to be an external call, we'll have a stub for the external
      // target, the target of this edge will be the stub and its addend will be
      // set 0.
      Addend += ELF::decodePPC64LocalEntryOffset((*ObjSymbol)->st_other);
      break;
    case ELF::R_PPC64_REL64:
      Kind = ppc64::Delta64;
      break;
```
- **EN**: Implements logic around `decodePPC64LocalEntryOffset`.
- **CN**: 围绕 `decodePPC64LocalEntryOffset` 实现具体逻辑。

### Lines 372-388
```cpp
    case ELF::R_PPC64_PCREL34:
      Kind = ppc64::Delta34;
      break;
    case ELF::R_PPC64_GOT_PCREL34:
      Kind = ppc64::RequestGOTAndTransformToDelta34;
      break;
    case ELF::R_PPC64_GOT_TLSGD16_HA:
      Kind = ppc64::RequestTLSDescInGOTAndTransformToTOCDelta16HA;
      break;
    case ELF::R_PPC64_GOT_TLSGD16_LO:
      Kind = ppc64::RequestTLSDescInGOTAndTransformToTOCDelta16LO;
      break;
    case ELF::R_PPC64_GOT_TLSGD_PCREL34:
      Kind = ppc64::RequestTLSDescInGOTAndTransformToDelta34;
      break;
    }

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 389-403
```cpp
    Edge GE(Kind, Offset, *GraphSymbol, Addend);
    BlockToFix.addEdge(std::move(GE));
    return Error::success();
  }

public:
  ELFLinkGraphBuilder_ppc64(StringRef FileName,
                            const object::ELFFile<ELFT> &Obj,
                            std::shared_ptr<orc::SymbolStringPool> SSP,
                            Triple TT, SubtargetFeatures Features)
      : ELFLinkGraphBuilder<ELFT>(Obj, std::move(SSP), std::move(TT),
                                  std::move(Features), FileName,
                                  ppc64::getEdgeKindName) {}
};

```
- **EN**: Implements logic around `GE`, `addEdge`, `success`, `ELFLinkGraphBuilder_ppc64`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `GE`, `addEdge`, `success`, `ELFLinkGraphBuilder_ppc64`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 404-416
```cpp
template <llvm::endianness Endianness>
class ELFJITLinker_ppc64 : public JITLinker<ELFJITLinker_ppc64<Endianness>> {
  using JITLinkerBase = JITLinker<ELFJITLinker_ppc64<Endianness>>;
  friend JITLinkerBase;

public:
  ELFJITLinker_ppc64(std::unique_ptr<JITLinkContext> Ctx,
                     std::unique_ptr<LinkGraph> G, PassConfiguration PassConfig)
      : JITLinkerBase(std::move(Ctx), std::move(G), std::move(PassConfig)) {
    JITLinkerBase::getPassConfig().PostAllocationPasses.push_back(
        [this](LinkGraph &G) { return defineTOCBase(G); });
  }

```
- **EN**: Introduces declarations for `ELFJITLinker_ppc64`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ELFJITLinker_ppc64` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 417-428
```cpp
private:
  Symbol *TOCSymbol = nullptr;

  Error defineTOCBase(LinkGraph &G) {
    for (Symbol *Sym : G.defined_symbols()) {
      if (LLVM_UNLIKELY(Sym->hasName() &&
                        *Sym->getName() == ELFTOCSymbolName)) {
        TOCSymbol = Sym;
        return Error::success();
      }
    }

```
- **EN**: Implements logic around `defineTOCBase`, `getName`, `success`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `defineTOCBase`, `getName`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 429-438
```cpp
    assert(TOCSymbol == nullptr &&
           "TOCSymbol should not be defined at this point");

    for (Symbol *Sym : G.external_symbols()) {
      if (Sym->hasName() && *Sym->getName() == ELFTOCSymbolName) {
        TOCSymbol = Sym;
        break;
      }
    }

```
- **EN**: Implements logic around `assert`.
- **CN**: 围绕 `assert` 实现具体逻辑。

### Lines 439-456
```cpp
    if (Section *TOCSection = G.findSectionByName(
            ppc64::TOCTableManager<Endianness>::getSectionName())) {
      assert(!TOCSection->empty() && "TOC section should have reserved an "
                                     "entry for containing the TOC base");

      SectionRange SR(*TOCSection);
      orc::ExecutorAddr TOCBaseAddr(SR.getFirstBlock()->getAddress() +
                                    ELFTOCBaseOffset);
      assert(TOCSymbol && TOCSymbol->isExternal() &&
             ".TOC. should be a external symbol at this point");
      G.makeAbsolute(*TOCSymbol, TOCBaseAddr);
      // Create an alias of .TOC. so that rtdyld checker can recognize.
      G.addAbsoluteSymbol(TOCSymbolAliasIdent, TOCSymbol->getAddress(),
                          TOCSymbol->getSize(), TOCSymbol->getLinkage(),
                          TOCSymbol->getScope(), TOCSymbol->isLive());
      return Error::success();
    }

```
- **EN**: Implements logic around `getSectionName`, `assert`, `SR`, `TOCBaseAddr`, and 5 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionName`, `assert`, `SR`, `TOCBaseAddr`, and 5 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 457-466
```cpp
    // If TOC section doesn't exist, which means no TOC relocation is found, we
    // don't need a TOCSymbol.
    return Error::success();
  }

  Error applyFixup(LinkGraph &G, Block &B, const Edge &E) const {
    return ppc64::applyFixup<Endianness>(G, B, E, TOCSymbol);
  }
};

```
- **EN**: Implements logic around `success`, `applyFixup`, `applyFixup<Endianness>`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `applyFixup`, `applyFixup<Endianness>` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 467-475
```cpp
template <llvm::endianness Endianness>
Expected<std::unique_ptr<LinkGraph>>
createLinkGraphFromELFObject_ppc64(MemoryBufferRef ObjectBuffer,
                                   std::shared_ptr<orc::SymbolStringPool> SSP) {
  LLVM_DEBUG({
    dbgs() << "Building jitlink graph for new input "
           << ObjectBuffer.getBufferIdentifier() << "...\n";
  });

```
- **EN**: Implements logic around `createLinkGraphFromELFObject_ppc64`, `dbgs`, `getBufferIdentifier`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createLinkGraphFromELFObject_ppc64`, `dbgs`, `getBufferIdentifier` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 476-491
```cpp
  auto ELFObj = object::ObjectFile::createELFObjectFile(ObjectBuffer);
  if (!ELFObj)
    return ELFObj.takeError();

  auto Features = (*ELFObj)->getFeatures();
  if (!Features)
    return Features.takeError();

  using ELFT = object::ELFType<Endianness, true>;
  auto &ELFObjFile = cast<object::ELFObjectFile<ELFT>>(**ELFObj);
  return ELFLinkGraphBuilder_ppc64<Endianness>(
             (*ELFObj)->getFileName(), ELFObjFile.getELFFile(), std::move(SSP),
             (*ELFObj)->makeTriple(), std::move(*Features))
      .buildGraph();
}

```
- **EN**: Implements logic around `createELFObjectFile`, `takeError`, `getFeatures`, `ELFObjectFile<ELFT>>`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createELFObjectFile`, `takeError`, `getFeatures`, `ELFObjectFile<ELFT>>`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 492-506
```cpp
template <llvm::endianness Endianness>
void link_ELF_ppc64(std::unique_ptr<LinkGraph> G,
                    std::unique_ptr<JITLinkContext> Ctx) {
  PassConfiguration Config;

  if (Ctx->shouldAddDefaultTargetPasses(G->getTargetTriple())) {
    // Construct a JITLinker and run the link function.

    // Add eh-frame passes.
    Config.PrePrunePasses.push_back(DWARFRecordSectionSplitter(".eh_frame"));
    Config.PrePrunePasses.push_back(EHFrameEdgeFixer(
        ".eh_frame", G->getPointerSize(), ppc64::Pointer32, ppc64::Pointer64,
        ppc64::Delta32, ppc64::Delta64, ppc64::NegDelta32));
    Config.PrePrunePasses.push_back(EHFrameNullTerminator(".eh_frame"));

```
- **EN**: Implements logic around `link_ELF_ppc64`, `push_back`, `getPointerSize`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `link_ELF_ppc64`, `push_back`, `getPointerSize` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 507-515
```cpp
    // Add a mark-live pass.
    if (auto MarkLive = Ctx->getMarkLivePass(G->getTargetTriple()))
      Config.PrePrunePasses.push_back(std::move(MarkLive));
    else
      Config.PrePrunePasses.push_back(markAllSymbolsLive);
  }

  Config.PostPrunePasses.push_back(buildTables_ELF_ppc64<Endianness>);

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 516-529
```cpp
  if (auto Err = Ctx->modifyPassConfig(*G, Config))
    return Ctx->notifyFailed(std::move(Err));

  ELFJITLinker_ppc64<Endianness>::link(std::move(Ctx), std::move(G),
                                       std::move(Config));
}

Expected<std::unique_ptr<LinkGraph>>
createLinkGraphFromELFObject_ppc64(MemoryBufferRef ObjectBuffer,
                                   std::shared_ptr<orc::SymbolStringPool> SSP) {
  return createLinkGraphFromELFObject_ppc64<llvm::endianness::big>(
      std::move(ObjectBuffer), std::move(SSP));
}

```
- **EN**: Implements logic around `notifyFailed`, `link`, `move`, `createLinkGraphFromELFObject_ppc64`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `notifyFailed`, `link`, `move`, `createLinkGraphFromELFObject_ppc64`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 530-541
```cpp
Expected<std::unique_ptr<LinkGraph>> createLinkGraphFromELFObject_ppc64le(
    MemoryBufferRef ObjectBuffer, std::shared_ptr<orc::SymbolStringPool> SSP) {
  return createLinkGraphFromELFObject_ppc64<llvm::endianness::little>(
      std::move(ObjectBuffer), std::move(SSP));
}

/// jit-link the given object buffer, which must be a ELF ppc64 object file.
void link_ELF_ppc64(std::unique_ptr<LinkGraph> G,
                    std::unique_ptr<JITLinkContext> Ctx) {
  return link_ELF_ppc64<llvm::endianness::big>(std::move(G), std::move(Ctx));
}

```
- **EN**: Implements logic around `createLinkGraphFromELFObject_ppc64le`, `little>`, `move`, `link_ELF_ppc64`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkGraphFromELFObject_ppc64le`, `little>`, `move`, `link_ELF_ppc64`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 542-548
```cpp
/// jit-link the given object buffer, which must be a ELF ppc64le object file.
void link_ELF_ppc64le(std::unique_ptr<LinkGraph> G,
                      std::unique_ptr<JITLinkContext> Ctx) {
  return link_ELF_ppc64<llvm::endianness::little>(std::move(G), std::move(Ctx));
}

} // end namespace llvm::jitlink
```
- **EN**: Introduces declarations for `llvm::jitlink`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm::jitlink` 等声明，定义本文件后续使用的数据结构或接口。

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

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/ELF_ppc64.h`, `llvm/ExecutionEngine/JITLink/DWARFRecordSectionSplitter.h`, `llvm/ExecutionEngine/JITLink/TableManager.h`, `llvm/ExecutionEngine/JITLink/ppc64.h`, `llvm/Object/ELFObjectFile.h`, `EHFrameSupportImpl.h`, `ELFLinkGraphBuilder.h`, `JITLinkGeneric.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, Object
