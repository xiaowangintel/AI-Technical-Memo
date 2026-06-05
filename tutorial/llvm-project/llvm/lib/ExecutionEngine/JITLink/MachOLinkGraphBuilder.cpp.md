# MachOLinkGraphBuilder.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/MachOLinkGraphBuilder.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements MachO LinkGraph builder.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//=--------- MachOLinkGraphBuilder.cpp - MachO LinkGraph builder ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Generic MachO LinkGraph building code.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 13-24
```cpp
#include "MachOLinkGraphBuilder.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/Sequence.h"
#include <optional>

#define DEBUG_TYPE "jitlink"

static const char *CommonSectionName = "__common";

namespace llvm {
namespace jitlink {

```
- **EN**: Pulls in the headers needed for this implementation, including `MachOLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Sequence.h`, `optional`.
- **CN**: 引入该实现所需的头文件，其中包括 `MachOLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Sequence.h`, `optional`。

### Lines 25-38
```cpp
MachOLinkGraphBuilder::~MachOLinkGraphBuilder() = default;

Expected<std::unique_ptr<LinkGraph>> MachOLinkGraphBuilder::buildGraph() {

  // We only operate on relocatable objects.
  if (!Obj.isRelocatableObject())
    return make_error<JITLinkError>("Object is not a relocatable MachO");

  if (auto Err = createNormalizedSections())
    return std::move(Err);

  if (auto Err = createNormalizedSymbols())
    return std::move(Err);

```
- **EN**: Implements logic around `~MachOLinkGraphBuilder`, `buildGraph`, `make_error<JITLinkError>`, `move`; this block manipulates JITLink graph structures or link-time passes; handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `~MachOLinkGraphBuilder`, `buildGraph`, `make_error<JITLinkError>`, `move` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 39-50
```cpp
  if (auto Err = graphifyRegularSymbols())
    return std::move(Err);

  if (auto Err = graphifySectionsWithCustomParsers())
    return std::move(Err);

  if (auto Err = addRelocations())
    return std::move(Err);

  return std::move(G);
}

```
- **EN**: Implements logic around `move`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `move` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 51-63
```cpp
MachOLinkGraphBuilder::MachOLinkGraphBuilder(
    const object::MachOObjectFile &Obj,
    std::shared_ptr<orc::SymbolStringPool> SSP, Triple TT,
    SubtargetFeatures Features,
    LinkGraph::GetEdgeKindNameFunction GetEdgeKindName)
    : Obj(Obj),
      G(std::make_unique<LinkGraph>(
          std::string(Obj.getFileName()), std::move(SSP), std::move(TT),
          std::move(Features), std::move(GetEdgeKindName))) {
  auto &MachHeader = Obj.getHeader64();
  SubsectionsViaSymbols = MachHeader.flags & MachO::MH_SUBSECTIONS_VIA_SYMBOLS;
}

```
- **EN**: Implements logic around `MachOLinkGraphBuilder`, `Obj`, `G`, `string`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `MachOLinkGraphBuilder`, `Obj`, `G`, `string`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 64-76
```cpp
void MachOLinkGraphBuilder::addCustomSectionParser(
    StringRef SectionName, SectionParserFunction Parser) {
  assert(!CustomSectionParserFunctions.count(SectionName) &&
         "Custom parser for this section already exists");
  CustomSectionParserFunctions[SectionName] = std::move(Parser);
}

Linkage MachOLinkGraphBuilder::getLinkage(uint16_t Desc) {
  if ((Desc & MachO::N_WEAK_DEF) || (Desc & MachO::N_WEAK_REF))
    return Linkage::Weak;
  return Linkage::Strong;
}

```
- **EN**: Implements logic around `addCustomSectionParser`, `assert`, `move`, `getLinkage`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `addCustomSectionParser`, `assert`, `move`, `getLinkage` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 77-90
```cpp
Scope MachOLinkGraphBuilder::getScope(StringRef Name, uint8_t Type) {
  if (Type & MachO::N_EXT) {
    if ((Type & MachO::N_PEXT) || Name.starts_with("l"))
      return Scope::Hidden;
    else
      return Scope::Default;
  }
  return Scope::Local;
}

bool MachOLinkGraphBuilder::isAltEntry(const NormalizedSymbol &NSym) {
  return NSym.Desc & MachO::N_ALT_ENTRY;
}

```
- **EN**: Implements logic around `getScope`, `isAltEntry`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getScope`, `isAltEntry` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 91-106
```cpp
bool MachOLinkGraphBuilder::isDebugSection(const NormalizedSection &NSec) {
  return (NSec.Flags & MachO::S_ATTR_DEBUG &&
          strcmp(NSec.SegName, "__DWARF") == 0);
}

bool MachOLinkGraphBuilder::isZeroFillSection(const NormalizedSection &NSec) {
  switch (NSec.Flags & MachO::SECTION_TYPE) {
  case MachO::S_ZEROFILL:
  case MachO::S_GB_ZEROFILL:
  case MachO::S_THREAD_LOCAL_ZEROFILL:
    return true;
  default:
    return false;
  }
}

```
- **EN**: Implements logic around `isDebugSection`, `strcmp`, `isZeroFillSection`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `isDebugSection`, `strcmp`, `isZeroFillSection` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 107-119
```cpp
Section &MachOLinkGraphBuilder::getCommonSection() {
  if (!CommonSection)
    CommonSection = &G->createSection(CommonSectionName,
                                      orc::MemProt::Read | orc::MemProt::Write);
  return *CommonSection;
}

Error MachOLinkGraphBuilder::createNormalizedSections() {
  // Build normalized sections. Verifies that section data is in-range (for
  // sections with content) and that address ranges are non-overlapping.

  LLVM_DEBUG(dbgs() << "Creating normalized sections...\n");

```
- **EN**: Implements logic around `getCommonSection`, `createSection`, `createNormalizedSections`; this block manipulates JITLink graph structures or link-time passes; drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `getCommonSection`, `createSection`, `createNormalizedSections` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 120-134
```cpp
  for (auto &SecRef : Obj.sections()) {
    NormalizedSection NSec;
    uint32_t DataOffset = 0;

    auto SecIndex = Obj.getSectionIndex(SecRef.getRawDataRefImpl());

    if (Obj.is64Bit()) {
      const MachO::section_64 &Sec64 =
          Obj.getSection64(SecRef.getRawDataRefImpl());

      memcpy(&NSec.SectName, &Sec64.sectname, 16);
      NSec.SectName[16] = '\0';
      memcpy(&NSec.SegName, Sec64.segname, 16);
      NSec.SegName[16] = '\0';

```
- **EN**: Implements logic around `getSectionIndex`, `getSection64`, `memcpy`.
- **CN**: 围绕 `getSectionIndex`, `getSection64`, `memcpy` 实现具体逻辑。

### Lines 135-147
```cpp
      NSec.Address = orc::ExecutorAddr(Sec64.addr);
      NSec.Size = Sec64.size;
      NSec.Alignment = 1ULL << Sec64.align;
      NSec.Flags = Sec64.flags;
      DataOffset = Sec64.offset;
    } else {
      const MachO::section &Sec32 = Obj.getSection(SecRef.getRawDataRefImpl());

      memcpy(&NSec.SectName, &Sec32.sectname, 16);
      NSec.SectName[16] = '\0';
      memcpy(&NSec.SegName, Sec32.segname, 16);
      NSec.SegName[16] = '\0';

```
- **EN**: Implements logic around `ExecutorAddr`, `getSection`, `memcpy`.
- **CN**: 围绕 `ExecutorAddr`, `getSection`, `memcpy` 实现具体逻辑。

### Lines 148-162
```cpp
      NSec.Address = orc::ExecutorAddr(Sec32.addr);
      NSec.Size = Sec32.size;
      NSec.Alignment = 1ULL << Sec32.align;
      NSec.Flags = Sec32.flags;
      DataOffset = Sec32.offset;
    }

    LLVM_DEBUG({
      dbgs() << "  " << NSec.SegName << "," << NSec.SectName << ": "
             << formatv("{0:x16}", NSec.Address) << " -- "
             << formatv("{0:x16}", NSec.Address + NSec.Size)
             << ", align: " << NSec.Alignment << ", index: " << SecIndex
             << "\n";
    });

```
- **EN**: Implements logic around `ExecutorAddr`, `dbgs`, `formatv`.
- **CN**: 围绕 `ExecutorAddr`, `dbgs`, `formatv` 实现具体逻辑。

### Lines 163-180
```cpp
    // Get the section data if any.
    if (!isZeroFillSection(NSec)) {
      if (DataOffset + NSec.Size > Obj.getData().size())
        return make_error<JITLinkError>(
            "Section data extends past end of file");

      NSec.Data = Obj.getData().data() + DataOffset;
    }

    // Get prot flags.
    // FIXME: Make sure this test is correct (it's probably missing cases
    // as-is).
    orc::MemProt Prot;
    if (NSec.Flags & MachO::S_ATTR_PURE_INSTRUCTIONS)
      Prot = orc::MemProt::Read | orc::MemProt::Exec;
    else
      Prot = orc::MemProt::Read | orc::MemProt::Write;

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getData`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getData` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 181-192
```cpp
    auto FullyQualifiedName =
        G->allocateContent(StringRef(NSec.SegName) + "," + NSec.SectName);
    NSec.GraphSection = &G->createSection(
        StringRef(FullyQualifiedName.data(), FullyQualifiedName.size()), Prot);

    // TODO: Are there any other criteria for NoAlloc lifetime?
    if (NSec.Flags & MachO::S_ATTR_DEBUG)
      NSec.GraphSection->setMemLifetime(orc::MemLifetime::NoAlloc);

    IndexToSection.insert(std::make_pair(SecIndex, std::move(NSec)));
  }

```
- **EN**: Implements logic around `allocateContent`, `createSection`, `StringRef`, `setMemLifetime`, and 1 more symbols.
- **CN**: 围绕 `allocateContent`, `createSection`, `StringRef`, `setMemLifetime`, and 1 more symbols 实现具体逻辑。

### Lines 193-209
```cpp
  std::vector<NormalizedSection *> Sections;
  Sections.reserve(IndexToSection.size());
  for (auto &KV : IndexToSection)
    Sections.push_back(&KV.second);

  // If we didn't end up creating any sections then bail out. The code below
  // assumes that we have at least one section.
  if (Sections.empty())
    return Error::success();

  llvm::sort(Sections,
             [](const NormalizedSection *LHS, const NormalizedSection *RHS) {
               assert(LHS && RHS && "Null section?");
               return std::tie(LHS->Address, LHS->Size) <
                      std::tie(RHS->Address, RHS->Size);
             });

```
- **EN**: Implements logic around `reserve`, `push_back`, `success`, `sort`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `reserve`, `push_back`, `success`, `sort`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 210-222
```cpp
  for (unsigned I = 0, E = Sections.size() - 1; I != E; ++I) {
    auto &Cur = *Sections[I];
    auto &Next = *Sections[I + 1];
    if (Next.Address < Cur.Address + Cur.Size)
      return make_error<JITLinkError>(
          "Address range for section " +
          formatv("\"{0}/{1}\" [ {2:x16} -- {3:x16} ] ", Cur.SegName,
                  Cur.SectName, Cur.Address, Cur.Address + Cur.Size) +
          "overlaps section \"" + Next.SegName + "/" + Next.SectName + "\"" +
          formatv("\"{0}/{1}\" [ {2:x16} -- {3:x16} ] ", Next.SegName,
                  Next.SectName, Next.Address, Next.Address + Next.Size));
  }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `formatv`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `formatv` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 223-237
```cpp
  return Error::success();
}

Error MachOLinkGraphBuilder::createNormalizedSymbols() {
  LLVM_DEBUG(dbgs() << "Creating normalized symbols...\n");

  for (auto &SymRef : Obj.symbols()) {

    unsigned SymbolIndex = Obj.getSymbolIndex(SymRef.getRawDataRefImpl());
    uint64_t Value;
    uint32_t NStrX;
    uint8_t Type;
    uint8_t Sect;
    uint16_t Desc;

```
- **EN**: Implements logic around `success`, `createNormalizedSymbols`, `getSymbolIndex`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `createNormalizedSymbols`, `getSymbolIndex` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 238-255
```cpp
    if (Obj.is64Bit()) {
      const MachO::nlist_64 &NL64 =
          Obj.getSymbol64TableEntry(SymRef.getRawDataRefImpl());
      Value = NL64.n_value;
      NStrX = NL64.n_strx;
      Type = NL64.n_type;
      Sect = NL64.n_sect;
      Desc = NL64.n_desc;
    } else {
      const MachO::nlist &NL32 =
          Obj.getSymbolTableEntry(SymRef.getRawDataRefImpl());
      Value = NL32.n_value;
      NStrX = NL32.n_strx;
      Type = NL32.n_type;
      Sect = NL32.n_sect;
      Desc = NL32.n_desc;
    }

```
- **EN**: Implements logic around `getSymbol64TableEntry`, `getSymbolTableEntry`.
- **CN**: 围绕 `getSymbol64TableEntry`, `getSymbolTableEntry` 实现具体逻辑。

### Lines 256-272
```cpp
    // Skip stabs.
    // FIXME: Are there other symbols we should be skipping?
    if (Type & MachO::N_STAB)
      continue;

    std::optional<StringRef> Name;
    if (NStrX) {
      if (auto NameOrErr = SymRef.getName())
        Name = *NameOrErr;
      else
        return NameOrErr.takeError();
    } else if (Type & MachO::N_EXT)
      return make_error<JITLinkError>("Symbol at index " +
                                      formatv("{0}", SymbolIndex) +
                                      " has no name (string table index 0), "
                                      "but N_EXT bit is set");

```
- **EN**: Implements logic around `takeError`, `make_error<JITLinkError>`, `formatv`, `name`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `make_error<JITLinkError>`, `formatv`, `name` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 273-288
```cpp
    LLVM_DEBUG({
      dbgs() << "  ";
      if (!Name)
        dbgs() << "<anonymous symbol>";
      else
        dbgs() << *Name;
      dbgs() << ": value = " << formatv("{0:x16}", Value)
             << ", type = " << formatv("{0:x2}", Type)
             << ", desc = " << formatv("{0:x4}", Desc) << ", sect = ";
      if (Sect)
        dbgs() << static_cast<unsigned>(Sect - 1);
      else
        dbgs() << "none";
      dbgs() << "\n";
    });

```
- **EN**: Implements logic around `dbgs`, `formatv`.
- **CN**: 围绕 `dbgs`, `formatv` 实现具体逻辑。

### Lines 289-300
```cpp
    // If this symbol has a section, verify that the addresses line up.
    if (Sect != 0) {
      auto NSec = findSectionByIndex(Sect - 1);
      if (!NSec)
        return NSec.takeError();

      if (orc::ExecutorAddr(Value) < NSec->Address ||
          orc::ExecutorAddr(Value) > NSec->Address + NSec->Size)
        return make_error<JITLinkError>("Address " + formatv("{0:x}", Value) +
                                        " for symbol " + *Name +
                                        " does not fall within section");

```
- **EN**: Implements logic around `findSectionByIndex`, `takeError`, `ExecutorAddr`, `make_error<JITLinkError>`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `findSectionByIndex`, `takeError`, `ExecutorAddr`, `make_error<JITLinkError>` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 301-314
```cpp
      if (!NSec->GraphSection) {
        LLVM_DEBUG({
          dbgs() << "  Skipping: Symbol is in section " << NSec->SegName << "/"
                 << NSec->SectName
                 << " which has no associated graph section.\n";
        });
        continue;
      }
    }

    IndexToSymbol[SymbolIndex] = &createNormalizedSymbol(
        Name, Value, Type, Sect, Desc, getLinkage(Desc), getScope(*Name, Type));
  }

```
- **EN**: Implements logic around `dbgs`, `createNormalizedSymbol`, `getLinkage`.
- **CN**: 围绕 `dbgs`, `createNormalizedSymbol`, `getLinkage` 实现具体逻辑。

### Lines 315-334
```cpp
  return Error::success();
}

void MachOLinkGraphBuilder::addSectionStartSymAndBlock(
    unsigned SecIndex, Section &GraphSec, orc::ExecutorAddr Address,
    const char *Data, orc::ExecutorAddrDiff Size, uint32_t Alignment,
    bool IsLive) {
  Block &B =
      Data ? G->createContentBlock(GraphSec, ArrayRef<char>(Data, Size),
                                   Address, Alignment, 0)
           : G->createZeroFillBlock(GraphSec, Size, Address, Alignment, 0);
  auto &Sym = G->addAnonymousSymbol(B, 0, Size, false, IsLive);
  auto SecI = IndexToSection.find(SecIndex);
  assert(SecI != IndexToSection.end() && "SecIndex invalid");
  auto &NSec = SecI->second;
  assert(!NSec.CanonicalSymbols.count(Sym.getAddress()) &&
         "Anonymous block start symbol clashes with existing symbol address");
  NSec.CanonicalSymbols[Sym.getAddress()] = &Sym;
}

```
- **EN**: Implements logic around `success`, `addSectionStartSymAndBlock`, `createContentBlock`, `createZeroFillBlock`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `addSectionStartSymAndBlock`, `createContentBlock`, `createZeroFillBlock`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 335-347
```cpp
Error MachOLinkGraphBuilder::graphifyRegularSymbols() {

  LLVM_DEBUG(dbgs() << "Creating graph symbols...\n");

  /// We only have 256 section indexes: Use a vector rather than a map.
  std::vector<std::vector<NormalizedSymbol *>> SecIndexToSymbols;
  SecIndexToSymbols.resize(256);

  // Create commons, externs, and absolutes, and partition all other symbols by
  // section.
  for (auto &KV : IndexToSymbol) {
    auto &NSym = *KV.second;

```
- **EN**: Implements logic around `graphifyRegularSymbols`, `resize`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `graphifyRegularSymbols`, `resize` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 348-371
```cpp
    switch (NSym.Type & MachO::N_TYPE) {
    case MachO::N_UNDF:
      if (NSym.Value) {
        if (!NSym.Name)
          return make_error<JITLinkError>("Anonymous common symbol at index " +
                                          Twine(KV.first));
        NSym.GraphSymbol = &G->addDefinedSymbol(
            G->createZeroFillBlock(getCommonSection(),
                                   orc::ExecutorAddrDiff(NSym.Value),
                                   orc::ExecutorAddr(),
                                   1ull << MachO::GET_COMM_ALIGN(NSym.Desc), 0),
            0, *NSym.Name, orc::ExecutorAddrDiff(NSym.Value), Linkage::Weak,
            NSym.S, false, NSym.Desc & MachO::N_NO_DEAD_STRIP);
      } else {
        if (!NSym.Name)
          return make_error<JITLinkError>("Anonymous external symbol at "
                                          "index " +
                                          Twine(KV.first));
        NSym.GraphSymbol = &G->addExternalSymbol(
            *NSym.Name, 0, (NSym.Desc & MachO::N_WEAK_REF) != 0);
      }
      break;
    case MachO::N_ABS:
      if (!NSym.Name)
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `Twine`, `addDefinedSymbol`, `createZeroFillBlock`, and 4 more symbols; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `Twine`, `addDefinedSymbol`, `createZeroFillBlock`, and 4 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 372-395
```cpp
        return make_error<JITLinkError>("Anonymous absolute symbol at index " +
                                        Twine(KV.first));
      NSym.GraphSymbol = &G->addAbsoluteSymbol(
          *NSym.Name, orc::ExecutorAddr(NSym.Value), 0, Linkage::Strong,
          getScope(*NSym.Name, NSym.Type), NSym.Desc & MachO::N_NO_DEAD_STRIP);
      break;
    case MachO::N_SECT:
      SecIndexToSymbols[NSym.Sect - 1].push_back(&NSym);
      break;
    case MachO::N_PBUD:
      return make_error<JITLinkError>(
          "Unupported N_PBUD symbol " +
          (NSym.Name ? ("\"" + *NSym.Name + "\"") : Twine("<anon>")) +
          " at index " + Twine(KV.first));
    case MachO::N_INDR:
      return make_error<JITLinkError>(
          "Unupported N_INDR symbol " +
          (NSym.Name ? ("\"" + *NSym.Name + "\"") : Twine("<anon>")) +
          " at index " + Twine(KV.first));
    default:
      return make_error<JITLinkError>(
          "Unrecognized symbol type " + Twine(NSym.Type & MachO::N_TYPE) +
          " for symbol " +
          (NSym.Name ? ("\"" + *NSym.Name + "\"") : Twine("<anon>")) +
```
- **EN**: Implements logic around `make_error<JITLinkError>`, `Twine`, `addAbsoluteSymbol`, `ExecutorAddr`, and 2 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `Twine`, `addAbsoluteSymbol`, `ExecutorAddr`, and 2 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 396-413
```cpp
          " at index " + Twine(KV.first));
    }
  }

  // Loop over sections performing regular graphification for those that
  // don't have custom parsers.
  for (auto &KV : IndexToSection) {
    auto SecIndex = KV.first;
    auto &NSec = KV.second;

    if (!NSec.GraphSection) {
      LLVM_DEBUG({
        dbgs() << "  " << NSec.SegName << "/" << NSec.SectName
               << " has no graph section. Skipping.\n";
      });
      continue;
    }

```
- **EN**: Implements logic around `Twine`, `dbgs`.
- **CN**: 围绕 `Twine`, `dbgs` 实现具体逻辑。

### Lines 414-432
```cpp
    // Skip sections with custom parsers.
    if (CustomSectionParserFunctions.count(NSec.GraphSection->getName())) {
      LLVM_DEBUG({
        dbgs() << "  Skipping section " << NSec.GraphSection->getName()
               << " as it has a custom parser.\n";
      });
      continue;
    } else if ((NSec.Flags & MachO::SECTION_TYPE) ==
               MachO::S_CSTRING_LITERALS) {
      if (auto Err = graphifyCStringSection(
              NSec, std::move(SecIndexToSymbols[SecIndex])))
        return Err;
      continue;
    } else
      LLVM_DEBUG({
        dbgs() << "  Graphifying regular section "
               << NSec.GraphSection->getName() << "...\n";
      });

```
- **EN**: Implements logic around `dbgs`, `move`, `getName`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `move`, `getName` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 433-456
```cpp
    bool SectionIsNoDeadStrip = NSec.Flags & MachO::S_ATTR_NO_DEAD_STRIP;
    bool SectionIsText = NSec.Flags & MachO::S_ATTR_PURE_INSTRUCTIONS;

    auto &SecNSymStack = SecIndexToSymbols[SecIndex];

    // If this section is non-empty but there are no symbols covering it then
    // create one block and anonymous symbol to cover the entire section.
    if (SecNSymStack.empty()) {
      if (NSec.Size > 0) {
        LLVM_DEBUG({
          dbgs() << "    Section non-empty, but contains no symbols. "
                    "Creating anonymous block to cover "
                 << formatv("{0:x16}", NSec.Address) << " -- "
                 << formatv("{0:x16}", NSec.Address + NSec.Size) << "\n";
        });
        addSectionStartSymAndBlock(SecIndex, *NSec.GraphSection, NSec.Address,
                                   NSec.Data, NSec.Size, NSec.Alignment,
                                   SectionIsNoDeadStrip);
      } else
        LLVM_DEBUG({
          dbgs() << "    Section empty and contains no symbols. Skipping.\n";
        });
      continue;
    }
```
- **EN**: Implements logic around `dbgs`, `formatv`, `addSectionStartSymAndBlock`; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `dbgs`, `formatv`, `addSectionStartSymAndBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 457-471
```cpp

    // Sort the symbol stack in by address, alt-entry status, scope, and name.
    // We sort in reverse order so that symbols will be visited in the right
    // order when we pop off the stack below.
    llvm::sort(SecNSymStack, [](const NormalizedSymbol *LHS,
                                const NormalizedSymbol *RHS) {
      if (LHS->Value != RHS->Value)
        return LHS->Value > RHS->Value;
      if (isAltEntry(*LHS) != isAltEntry(*RHS))
        return isAltEntry(*RHS);
      if (LHS->S != RHS->S)
        return static_cast<uint8_t>(LHS->S) < static_cast<uint8_t>(RHS->S);
      return LHS->Name < RHS->Name;
    });

```
- **EN**: Implements logic around `sort`, `isAltEntry`, `static_cast<uint8_t>`; this block executes or prepares runtime behavior for LLVM IR/JIT code; returns subsystem-specific computed results.
- **CN**: 围绕 `sort`, `isAltEntry`, `static_cast<uint8_t>` 实现具体逻辑；这一段执行或准备 LLVM IR/JIT 代码的运行时行为，返回子系统相关的计算结果。

### Lines 472-491
```cpp
    // The first symbol in a section can not be an alt-entry symbol.
    if (!SecNSymStack.empty() && isAltEntry(*SecNSymStack.back()))
      return make_error<JITLinkError>(
          "First symbol in " + NSec.GraphSection->getName() + " is alt-entry");

    // If the section is non-empty but there is no symbol covering the start
    // address then add an anonymous one.
    if (orc::ExecutorAddr(SecNSymStack.back()->Value) != NSec.Address) {
      auto AnonBlockSize =
          orc::ExecutorAddr(SecNSymStack.back()->Value) - NSec.Address;
      LLVM_DEBUG({
        dbgs() << "    Section start not covered by symbol. "
               << "Creating anonymous block to cover [ " << NSec.Address
               << " -- " << (NSec.Address + AnonBlockSize) << " ]\n";
      });
      addSectionStartSymAndBlock(SecIndex, *NSec.GraphSection, NSec.Address,
                                 NSec.Data, AnonBlockSize, NSec.Alignment,
                                 SectionIsNoDeadStrip);
    }

```
- **EN**: Implements logic around `make_error<JITLinkError>`, `getName`, `ExecutorAddr`, `dbgs`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `make_error<JITLinkError>`, `getName`, `ExecutorAddr`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 492-514
```cpp
    // Visit section symbols in order by popping off the reverse-sorted stack,
    // building graph symbols as we go.
    //
    // If MH_SUBSECTIONS_VIA_SYMBOLS is set we'll build a block for each
    // alt-entry chain.
    //
    // If MH_SUBSECTIONS_VIA_SYMBOLS is not set then we'll just build one block
    // for the whole section.
    while (!SecNSymStack.empty()) {
      SmallVector<NormalizedSymbol *, 8> BlockSyms;

      // Get the symbols in this alt-entry chain, or the whole section (if
      // !SubsectionsViaSymbols).
      BlockSyms.push_back(SecNSymStack.back());
      SecNSymStack.pop_back();
      while (!SecNSymStack.empty() &&
             (isAltEntry(*SecNSymStack.back()) ||
              SecNSymStack.back()->Value == BlockSyms.back()->Value ||
             !SubsectionsViaSymbols)) {
        BlockSyms.push_back(SecNSymStack.back());
        SecNSymStack.pop_back();
      }

```
- **EN**: Implements logic around `push_back`, `pop_back`, `isAltEntry`, `back`; this block manipulates JITLink graph structures or link-time passes; executes or prepares runtime behavior for LLVM IR/JIT code.
- **CN**: 围绕 `push_back`, `pop_back`, `isAltEntry`, `back` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，执行或准备 LLVM IR/JIT 代码的运行时行为。

### Lines 515-530
```cpp
      // BlockNSyms now contains the block symbols in reverse canonical order.
      auto BlockStart = orc::ExecutorAddr(BlockSyms.front()->Value);
      orc::ExecutorAddr BlockEnd =
          SecNSymStack.empty() ? NSec.Address + NSec.Size
                               : orc::ExecutorAddr(SecNSymStack.back()->Value);
      orc::ExecutorAddrDiff BlockOffset = BlockStart - NSec.Address;
      orc::ExecutorAddrDiff BlockSize = BlockEnd - BlockStart;

      LLVM_DEBUG({
        dbgs() << "    Creating block for " << formatv("{0:x16}", BlockStart)
               << " -- " << formatv("{0:x16}", BlockEnd) << ": "
               << NSec.GraphSection->getName() << " + "
               << formatv("{0:x16}", BlockOffset) << " with "
               << BlockSyms.size() << " symbol(s)...\n";
      });

```
- **EN**: Implements logic around `ExecutorAddr`, `empty`, `dbgs`, `formatv`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `ExecutorAddr`, `empty`, `dbgs`, `formatv`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 531-546
```cpp
      Block &B =
          NSec.Data
              ? G->createContentBlock(
                    *NSec.GraphSection,
                    ArrayRef<char>(NSec.Data + BlockOffset, BlockSize),
                    BlockStart, NSec.Alignment, BlockStart % NSec.Alignment)
              : G->createZeroFillBlock(*NSec.GraphSection, BlockSize,
                                       BlockStart, NSec.Alignment,
                                       BlockStart % NSec.Alignment);

      std::optional<orc::ExecutorAddr> LastCanonicalAddr;
      auto SymEnd = BlockEnd;
      while (!BlockSyms.empty()) {
        auto &NSym = *BlockSyms.back();
        BlockSyms.pop_back();

```
- **EN**: Implements logic around `createContentBlock`, `ArrayRef<char>`, `createZeroFillBlock`, `back`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `createContentBlock`, `ArrayRef<char>`, `createZeroFillBlock`, `back`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 547-562
```cpp
        bool SymLive =
            (NSym.Desc & MachO::N_NO_DEAD_STRIP) || SectionIsNoDeadStrip;

        auto &Sym = createStandardGraphSymbol(
            NSym, B, SymEnd - orc::ExecutorAddr(NSym.Value), SectionIsText,
            SymLive, LastCanonicalAddr != orc::ExecutorAddr(NSym.Value));

        if (LastCanonicalAddr != Sym.getAddress()) {
          if (LastCanonicalAddr)
            SymEnd = *LastCanonicalAddr;
          LastCanonicalAddr = Sym.getAddress();
        }
      }
    }
  }

```
- **EN**: Implements logic around `createStandardGraphSymbol`, `ExecutorAddr`, `getAddress`.
- **CN**: 围绕 `createStandardGraphSymbol`, `ExecutorAddr`, `getAddress` 实现具体逻辑。

### Lines 563-586
```cpp
  return Error::success();
}

Symbol &MachOLinkGraphBuilder::createStandardGraphSymbol(NormalizedSymbol &NSym,
                                                         Block &B, size_t Size,
                                                         bool IsText,
                                                         bool IsNoDeadStrip,
                                                         bool IsCanonical) {

  LLVM_DEBUG({
    dbgs() << "      " << formatv("{0:x16}", NSym.Value) << " -- "
           << formatv("{0:x16}", NSym.Value + Size) << ": ";
    if (!NSym.Name)
      dbgs() << "<anonymous symbol>";
    else
      dbgs() << *NSym.Name;
    if (IsText)
      dbgs() << " [text]";
    if (IsNoDeadStrip)
      dbgs() << " [no-dead-strip]";
    if (!IsCanonical)
      dbgs() << " [non-canonical]";
    dbgs() << "\n";
  });
```
- **EN**: Implements logic around `success`, `createStandardGraphSymbol`, `dbgs`, `formatv`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `createStandardGraphSymbol`, `dbgs`, `formatv` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 587-598
```cpp

  auto SymOffset = orc::ExecutorAddr(NSym.Value) - B.getAddress();
  auto &Sym =
      NSym.Name
          ? G->addDefinedSymbol(B, SymOffset, *NSym.Name, Size, NSym.L, NSym.S,
                                IsText, IsNoDeadStrip)
          : G->addAnonymousSymbol(B, SymOffset, Size, IsText, IsNoDeadStrip);
  NSym.GraphSymbol = &Sym;

  if (IsCanonical)
    setCanonicalSymbol(getSectionByIndex(NSym.Sect - 1), Sym);

```
- **EN**: Implements logic around `ExecutorAddr`, `addDefinedSymbol`, `addAnonymousSymbol`, `setCanonicalSymbol`.
- **CN**: 围绕 `ExecutorAddr`, `addDefinedSymbol`, `addAnonymousSymbol`, `setCanonicalSymbol` 实现具体逻辑。

### Lines 599-610
```cpp
  return Sym;
}

Error MachOLinkGraphBuilder::graphifySectionsWithCustomParsers() {
  // Graphify special sections.
  for (auto &KV : IndexToSection) {
    auto &NSec = KV.second;

    // Skip non-graph sections.
    if (!NSec.GraphSection)
      continue;

```
- **EN**: Implements logic around `graphifySectionsWithCustomParsers`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `graphifySectionsWithCustomParsers` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 611-626
```cpp
    auto HI = CustomSectionParserFunctions.find(NSec.GraphSection->getName());
    if (HI != CustomSectionParserFunctions.end()) {
      auto &Parse = HI->second;
      if (auto Err = Parse(NSec))
        return Err;
    }
  }

  return Error::success();
}

Error MachOLinkGraphBuilder::graphifyCStringSection(
    NormalizedSection &NSec, std::vector<NormalizedSymbol *> NSyms) {
  assert(NSec.GraphSection && "C string literal section missing graph section");
  assert(NSec.Data && "C string literal section has no data");

```
- **EN**: Implements logic around `find`, `success`, `graphifyCStringSection`, `assert`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `find`, `success`, `graphifyCStringSection`, `assert` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 627-650
```cpp
  LLVM_DEBUG({
    dbgs() << "  Graphifying C-string literal section "
           << NSec.GraphSection->getName() << "\n";
  });

  if (NSec.Data[NSec.Size - 1] != '\0')
    return make_error<JITLinkError>("C string literal section " +
                                    NSec.GraphSection->getName() +
                                    " does not end with null terminator");

  /// Sort into reverse order to use as a stack.
  llvm::sort(NSyms,
             [](const NormalizedSymbol *LHS, const NormalizedSymbol *RHS) {
               if (LHS->Value != RHS->Value)
                 return LHS->Value > RHS->Value;
               if (LHS->L != RHS->L)
                 return LHS->L > RHS->L;
               if (LHS->S != RHS->S)
                 return LHS->S > RHS->S;
               if (RHS->Name) {
                 if (!LHS->Name)
                   return true;
                 return *LHS->Name > *RHS->Name;
               }
```
- **EN**: Implements logic around `dbgs`, `getName`, `make_error<JITLinkError>`, `sort`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `dbgs`, `getName`, `make_error<JITLinkError>`, `sort` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 651-667
```cpp
               return false;
             });

  bool SectionIsNoDeadStrip = NSec.Flags & MachO::S_ATTR_NO_DEAD_STRIP;
  bool SectionIsText = NSec.Flags & MachO::S_ATTR_PURE_INSTRUCTIONS;
  orc::ExecutorAddrDiff BlockStart = 0;

  // Scan section for null characters.
  for (size_t I = 0; I != NSec.Size; ++I) {
    if (NSec.Data[I] == '\0') {
      size_t BlockSize = I + 1 - BlockStart;
      // Create a block for this null terminated string.
      auto &B = G->createContentBlock(*NSec.GraphSection,
                                      {NSec.Data + BlockStart, BlockSize},
                                      NSec.Address + BlockStart, NSec.Alignment,
                                      BlockStart % NSec.Alignment);

```
- **EN**: Implements logic around `createContentBlock`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `createContentBlock` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 668-683
```cpp
      LLVM_DEBUG({
        dbgs() << "    Created block " << B.getRange()
               << ", align = " << B.getAlignment()
               << ", align-ofs = " << B.getAlignmentOffset() << " for \"";
        for (size_t J = 0; J != std::min(B.getSize(), size_t(16)); ++J)
          switch (B.getContent()[J]) {
          case '\0': break;
          case '\n': dbgs() << "\\n"; break;
          case '\t': dbgs() << "\\t"; break;
          default:   dbgs() << B.getContent()[J]; break;
          }
        if (B.getSize() > 16)
          dbgs() << "...";
        dbgs() << "\"\n";
      });

```
- **EN**: Implements logic around `dbgs`, `getAlignment`, `getAlignmentOffset`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `dbgs`, `getAlignment`, `getAlignmentOffset` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 684-704
```cpp
      // If there's no symbol at the start of this block then create one.
      if (NSyms.empty() ||
          orc::ExecutorAddr(NSyms.back()->Value) != B.getAddress()) {
        auto &S = G->addAnonymousSymbol(B, 0, BlockSize, false, false);
        setCanonicalSymbol(NSec, S);
        LLVM_DEBUG({
          dbgs() << "      Adding symbol for c-string block " << B.getRange()
                 << ": <anonymous symbol> at offset 0\n";
        });
      }

      // Process any remaining symbols that point into this block.
      auto LastCanonicalAddr = B.getAddress() + BlockSize;
      while (!NSyms.empty() && orc::ExecutorAddr(NSyms.back()->Value) <
                                   B.getAddress() + BlockSize) {
        auto &NSym = *NSyms.back();
        size_t SymSize = (B.getAddress() + BlockSize) -
                         orc::ExecutorAddr(NSyms.back()->Value);
        bool SymLive =
            (NSym.Desc & MachO::N_NO_DEAD_STRIP) || SectionIsNoDeadStrip;

```
- **EN**: Implements logic around `ExecutorAddr`, `addAnonymousSymbol`, `setCanonicalSymbol`, `dbgs`, and 2 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `ExecutorAddr`, `addAnonymousSymbol`, `setCanonicalSymbol`, `dbgs`, and 2 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 705-720
```cpp
        bool IsCanonical = false;
        if (LastCanonicalAddr != orc::ExecutorAddr(NSym.Value)) {
          IsCanonical = true;
          LastCanonicalAddr = orc::ExecutorAddr(NSym.Value);
        }

        auto &Sym = createStandardGraphSymbol(NSym, B, SymSize, SectionIsText,
                                              SymLive, IsCanonical);
        (void)Sym;
        LLVM_DEBUG({
          dbgs() << "      Adding symbol for c-string block " << B.getRange()
                 << ": "
                 << (Sym.hasName() ? *Sym.getName() : "<anonymous symbol>")
                 << " at offset " << formatv("{0:x}", Sym.getOffset()) << "\n";
        });

```
- **EN**: Implements logic around `ExecutorAddr`, `createStandardGraphSymbol`, `dbgs`, `hasName`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `ExecutorAddr`, `createStandardGraphSymbol`, `dbgs`, `hasName`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 721-734
```cpp
        NSyms.pop_back();
      }

      BlockStart += BlockSize;
    }
  }

  assert(llvm::all_of(NSec.GraphSection->blocks(),
                      [](Block *B) { return isCStringBlock(*B); }) &&
         "All blocks in section should hold single c-strings");

  return Error::success();
}

```
- **EN**: Implements logic around `pop_back`, `assert`, `isCStringBlock`, `success`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `pop_back`, `assert`, `isCStringBlock`, `success` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 735-736
```cpp
} // end namespace jitlink
} // end namespace llvm
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

- **Direct includes / 直接包含**: `MachOLinkGraphBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Sequence.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
- **Generated macros / 生成宏**: `GET_COMM_ALIGN`
