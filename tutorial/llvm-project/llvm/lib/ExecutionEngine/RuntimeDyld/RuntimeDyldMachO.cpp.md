# RuntimeDyldMachO.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/RuntimeDyld/RuntimeDyldMachO.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implementation of the MC-JIT runtime dynamic linker.
  - **CN**: 实现 RuntimeDyld，在运行时把目标代码装入内存、应用重定位并解析符号。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- RuntimeDyldMachO.cpp - Run-time dynamic linker for MC-JIT -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implementation of the MC-JIT runtime dynamic linker.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 12-22
```cpp

#include "RuntimeDyldMachO.h"
#include "Targets/RuntimeDyldMachOAArch64.h"
#include "Targets/RuntimeDyldMachOARM.h"
#include "Targets/RuntimeDyldMachOI386.h"
#include "Targets/RuntimeDyldMachOX86_64.h"
#include "llvm/ADT/StringRef.h"

using namespace llvm;
using namespace llvm::object;

```
- **EN**: Pulls in the headers needed for this implementation, including `RuntimeDyldMachO.h`, `Targets/RuntimeDyldMachOAArch64.h`, `Targets/RuntimeDyldMachOARM.h`, `Targets/RuntimeDyldMachOI386.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `RuntimeDyldMachO.h`, `Targets/RuntimeDyldMachOAArch64.h`, `Targets/RuntimeDyldMachOARM.h`, `Targets/RuntimeDyldMachOI386.h`。

### Lines 23-34
```cpp
#define DEBUG_TYPE "dyld"

namespace {

class LoadedMachOObjectInfo final
    : public LoadedObjectInfoHelper<LoadedMachOObjectInfo,
                                    RuntimeDyld::LoadedObjectInfo> {
public:
  LoadedMachOObjectInfo(RuntimeDyldImpl &RTDyld,
                        ObjSectionToIDMap ObjSecToIDMap)
      : LoadedObjectInfoHelper(RTDyld, std::move(ObjSecToIDMap)) {}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 35-44
```cpp
  OwningBinary<ObjectFile>
  getObjectForDebug(const ObjectFile &Obj) const override {
    return OwningBinary<ObjectFile>();
  }
};

}

namespace llvm {

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 45-62
```cpp
int64_t RuntimeDyldMachO::memcpyAddend(const RelocationEntry &RE) const {
  unsigned NumBytes = 1 << RE.Size;
  uint8_t *Src = Sections[RE.SectionID].getAddress() + RE.Offset;

  return static_cast<int64_t>(readBytesUnaligned(Src, NumBytes));
}

Expected<relocation_iterator>
RuntimeDyldMachO::processScatteredVANILLA(
                          unsigned SectionID, relocation_iterator RelI,
                          const ObjectFile &BaseObjT,
                          RuntimeDyldMachO::ObjSectionToIDMap &ObjSectionToID,
                          bool TargetIsLocalThumbFunc) {
  const MachOObjectFile &Obj =
    static_cast<const MachOObjectFile&>(BaseObjT);
  MachO::any_relocation_info RE =
    Obj.getRelocation(RelI->getRawDataRefImpl());

```
- **EN**: Implements logic around `memcpyAddend`, `getAddress`, `static_cast<int64_t>`, `processScatteredVANILLA`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `memcpyAddend`, `getAddress`, `static_cast<int64_t>`, `processScatteredVANILLA`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 63-71
```cpp
  SectionEntry &Section = Sections[SectionID];
  uint32_t RelocType = Obj.getAnyRelocationType(RE);
  bool IsPCRel = Obj.getAnyRelocationPCRel(RE);
  unsigned Size = Obj.getAnyRelocationLength(RE);
  uint64_t Offset = RelI->getOffset();
  uint8_t *LocalAddress = Section.getAddressWithOffset(Offset);
  unsigned NumBytes = 1 << Size;
  int64_t Addend = readBytesUnaligned(LocalAddress, NumBytes);

```
- **EN**: Implements logic around `getAnyRelocationType`, `getAnyRelocationPCRel`, `getAnyRelocationLength`, `getOffset`, and 2 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getAnyRelocationType`, `getAnyRelocationPCRel`, `getAnyRelocationLength`, `getOffset`, and 2 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 72-84
```cpp
  unsigned SymbolBaseAddr = Obj.getScatteredRelocationValue(RE);
  section_iterator TargetSI = getSectionByAddress(Obj, SymbolBaseAddr);
  assert(TargetSI != Obj.section_end() && "Can't find section for symbol");
  uint64_t SectionBaseAddr = TargetSI->getAddress();
  SectionRef TargetSection = *TargetSI;
  bool IsCode = TargetSection.isText();
  uint32_t TargetSectionID = ~0U;
  if (auto TargetSectionIDOrErr =
        findOrEmitSection(Obj, TargetSection, IsCode, ObjSectionToID))
    TargetSectionID = *TargetSectionIDOrErr;
  else
    return TargetSectionIDOrErr.takeError();

```
- **EN**: Implements logic around `getScatteredRelocationValue`, `getSectionByAddress`, `assert`, `getAddress`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getScatteredRelocationValue`, `getSectionByAddress`, `assert`, `getAddress`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 85-93
```cpp
  Addend -= SectionBaseAddr;
  RelocationEntry R(SectionID, Offset, RelocType, Addend, IsPCRel, Size);
  R.IsTargetThumbFunc = TargetIsLocalThumbFunc;

  addRelocationForSection(R, TargetSectionID);

  return ++RelI;
}

```
- **EN**: Implements logic around `R`, `addRelocationForSection`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `R`, `addRelocationForSection` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 94-105
```cpp

Expected<RelocationValueRef>
RuntimeDyldMachO::getRelocationValueRef(
    const ObjectFile &BaseTObj, const relocation_iterator &RI,
    const RelocationEntry &RE, ObjSectionToIDMap &ObjSectionToID) {

  const MachOObjectFile &Obj =
      static_cast<const MachOObjectFile &>(BaseTObj);
  MachO::any_relocation_info RelInfo =
      Obj.getRelocation(RI->getRawDataRefImpl());
  RelocationValueRef Value;

```
- **EN**: Implements logic around `getRelocationValueRef`, `getRelocation`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `getRelocationValueRef`, `getRelocation` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 106-123
```cpp
  bool IsExternal = Obj.getPlainRelocationExternal(RelInfo);
  if (IsExternal) {
    symbol_iterator Symbol = RI->getSymbol();
    StringRef TargetName;
    if (auto TargetNameOrErr = Symbol->getName())
      TargetName = *TargetNameOrErr;
    else
      return TargetNameOrErr.takeError();
    RTDyldSymbolTable::const_iterator SI =
      GlobalSymbolTable.find(TargetName.data());
    if (SI != GlobalSymbolTable.end()) {
      const auto &SymInfo = SI->second;
      Value.SectionID = SymInfo.getSectionID();
      Value.Offset = SymInfo.getOffset() + RE.Addend;
    } else {
      Value.SymbolName = TargetName.data();
      Value.Offset = RE.Addend;
    }
```
- **EN**: Implements logic around `getPlainRelocationExternal`, `getSymbol`, `takeError`, `find`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getPlainRelocationExternal`, `getSymbol`, `takeError`, `find`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 124-135
```cpp
  } else {
    SectionRef Sec = Obj.getAnyRelocationSection(RelInfo);
    bool IsCode = Sec.isText();
    if (auto SectionIDOrErr = findOrEmitSection(Obj, Sec, IsCode,
                                                ObjSectionToID))
      Value.SectionID = *SectionIDOrErr;
    else
      return SectionIDOrErr.takeError();
    uint64_t Addr = Sec.getAddress();
    Value.Offset = RE.Addend - Addr;
  }

```
- **EN**: Implements logic around `getAnyRelocationSection`, `isText`, `takeError`, `getAddress`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getAnyRelocationSection`, `isText`, `takeError`, `getAddress` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 136-146
```cpp
  return Value;
}

void RuntimeDyldMachO::makeValueAddendPCRel(RelocationValueRef &Value,
                                            const relocation_iterator &RI,
                                            unsigned OffsetToNextPC) {
  auto &O = *cast<MachOObjectFile>(RI->getObject());
  section_iterator SecI = O.getRelocationRelocatedSection(RI);
  Value.Offset += RI->getOffset() + OffsetToNextPC + SecI->getAddress();
}

```
- **EN**: Implements logic around `makeValueAddendPCRel`, `cast<MachOObjectFile>`, `getRelocationRelocatedSection`, `getOffset`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `makeValueAddendPCRel`, `cast<MachOObjectFile>`, `getRelocationRelocatedSection`, `getOffset` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 147-160
```cpp
void RuntimeDyldMachO::dumpRelocationToResolve(const RelocationEntry &RE,
                                               uint64_t Value) const {
  const SectionEntry &Section = Sections[RE.SectionID];
  uint8_t *LocalAddress = Section.getAddress() + RE.Offset;
  uint64_t FinalAddress = Section.getLoadAddress() + RE.Offset;

  dbgs() << "resolveRelocation Section: " << RE.SectionID
         << " LocalAddress: " << format("%p", LocalAddress)
         << " FinalAddress: " << format("0x%016" PRIx64, FinalAddress)
         << " Value: " << format("0x%016" PRIx64, Value) << " Addend: " << RE.Addend
         << " isPCRel: " << RE.IsPCRel << " MachoType: " << RE.RelType
         << " Size: " << (1 << RE.Size) << "\n";
}

```
- **EN**: Implements logic around `dumpRelocationToResolve`, `getAddress`, `getLoadAddress`, `dbgs`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `dumpRelocationToResolve`, `getAddress`, `getLoadAddress`, `dbgs`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 161-173
```cpp
section_iterator
RuntimeDyldMachO::getSectionByAddress(const MachOObjectFile &Obj,
                                      uint64_t Addr) {
  section_iterator SI = Obj.section_begin();
  section_iterator SE = Obj.section_end();

  for (; SI != SE; ++SI) {
    uint64_t SAddr = SI->getAddress();
    uint64_t SSize = SI->getSize();
    if ((Addr >= SAddr) && (Addr < SAddr + SSize))
      return SI;
  }

```
- **EN**: Implements logic around `getSectionByAddress`, `section_begin`, `section_end`, `getAddress`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getSectionByAddress`, `section_begin`, `section_end`, `getAddress`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 174-185
```cpp
  return SE;
}


// Populate __pointers section.
Error RuntimeDyldMachO::populateIndirectSymbolPointersSection(
                                                    const MachOObjectFile &Obj,
                                                    const SectionRef &PTSection,
                                                    unsigned PTSectionID) {
  assert(!Obj.is64Bit() &&
         "Pointer table section not supported in 64-bit MachO.");

```
- **EN**: Implements logic around `populateIndirectSymbolPointersSection`, `assert`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `populateIndirectSymbolPointersSection`, `assert` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 186-196
```cpp
  MachO::dysymtab_command DySymTabCmd = Obj.getDysymtabLoadCommand();
  MachO::section Sec32 = Obj.getSection(PTSection.getRawDataRefImpl());
  uint32_t PTSectionSize = Sec32.size;
  unsigned FirstIndirectSymbol = Sec32.reserved1;
  const unsigned PTEntrySize = 4;
  unsigned NumPTEntries = PTSectionSize / PTEntrySize;
  unsigned PTEntryOffset = 0;

  assert((PTSectionSize % PTEntrySize) == 0 &&
         "Pointers section does not contain a whole number of stubs?");

```
- **EN**: Implements logic around `getDysymtabLoadCommand`, `getSection`, `assert`.
- **CN**: 围绕 `getDysymtabLoadCommand`, `getSection`, `assert` 实现具体逻辑。

### Lines 197-214
```cpp
  LLVM_DEBUG(dbgs() << "Populating pointer table section "
                    << Sections[PTSectionID].getName() << ", Section ID "
                    << PTSectionID << ", " << NumPTEntries << " entries, "
                    << PTEntrySize << " bytes each:\n");

  for (unsigned i = 0; i < NumPTEntries; ++i) {
    unsigned SymbolIndex =
      Obj.getIndirectSymbolTableEntry(DySymTabCmd, FirstIndirectSymbol + i);
    symbol_iterator SI = Obj.getSymbolByIndex(SymbolIndex);
    StringRef IndirectSymbolName;
    if (auto IndirectSymbolNameOrErr = SI->getName())
      IndirectSymbolName = *IndirectSymbolNameOrErr;
    else
      return IndirectSymbolNameOrErr.takeError();
    LLVM_DEBUG(dbgs() << "  " << IndirectSymbolName << ": index " << SymbolIndex
                      << ", PT offset: " << PTEntryOffset << "\n");
    RelocationEntry RE(PTSectionID, PTEntryOffset,
                       MachO::GENERIC_RELOC_VANILLA, 0, false, 2);
```
- **EN**: Implements logic around `getName`, `getIndirectSymbolTableEntry`, `getSymbolByIndex`, `takeError`, and 1 more symbols; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `getIndirectSymbolTableEntry`, `getSymbolByIndex`, `takeError`, and 1 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 215-224
```cpp
    addRelocationForSymbol(RE, IndirectSymbolName);
    PTEntryOffset += PTEntrySize;
  }
  return Error::success();
}

bool RuntimeDyldMachO::isCompatibleFile(const object::ObjectFile &Obj) const {
  return Obj.isMachO();
}

```
- **EN**: Implements logic around `addRelocationForSymbol`, `success`, `isCompatibleFile`, `isMachO`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `addRelocationForSymbol`, `success`, `isCompatibleFile`, `isMachO` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 225-239
```cpp
template <typename Impl>
Error
RuntimeDyldMachOCRTPBase<Impl>::finalizeLoad(const ObjectFile &Obj,
                                             ObjSectionToIDMap &SectionMap) {
  unsigned EHFrameSID = RTDYLD_INVALID_SECTION_ID;
  unsigned TextSID = RTDYLD_INVALID_SECTION_ID;
  unsigned ExceptTabSID = RTDYLD_INVALID_SECTION_ID;

  for (const auto &Section : Obj.sections()) {
    StringRef Name;
    if (Expected<StringRef> NameOrErr = Section.getName())
      Name = *NameOrErr;
    else
      consumeError(NameOrErr.takeError());

```
- **EN**: Implements logic around `finalizeLoad`, `consumeError`; this block handles relocation, fixup, or symbol-resolution work.
- **CN**: 围绕 `finalizeLoad`, `consumeError` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作。

### Lines 240-257
```cpp
    // Force emission of the __text, __eh_frame, and __gcc_except_tab sections
    // if they're present. Otherwise call down to the impl to handle other
    // sections that have already been emitted.
    if (Name == "__text") {
      if (auto TextSIDOrErr = findOrEmitSection(Obj, Section, true, SectionMap))
        TextSID = *TextSIDOrErr;
      else
        return TextSIDOrErr.takeError();
    } else if (Name == "__eh_frame") {
      if (auto EHFrameSIDOrErr = findOrEmitSection(Obj, Section, false,
                                                   SectionMap))
        EHFrameSID = *EHFrameSIDOrErr;
      else
        return EHFrameSIDOrErr.takeError();
    } else if (Name == "__gcc_except_tab") {
      if (auto ExceptTabSIDOrErr = findOrEmitSection(Obj, Section, true,
                                                     SectionMap))
        ExceptTabSID = *ExceptTabSIDOrErr;
```
- **EN**: Implements logic around `takeError`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 258-269
```cpp
      else
        return ExceptTabSIDOrErr.takeError();
    } else {
      auto I = SectionMap.find(Section);
      if (I != SectionMap.end())
        if (auto Err = impl().finalizeSection(Obj, I->second, Section))
          return Err;
    }
  }
  UnregisteredEHFrameSections.push_back(
    EHFrameRelatedSections(EHFrameSID, TextSID, ExceptTabSID));

```
- **EN**: Implements logic around `takeError`, `find`, `push_back`, `EHFrameRelatedSections`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `find`, `push_back`, `EHFrameRelatedSections` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 270-278
```cpp
  return Error::success();
}

template <typename Impl>
unsigned char *RuntimeDyldMachOCRTPBase<Impl>::processFDE(uint8_t *P,
                                                          int64_t DeltaForText,
                                                          int64_t DeltaForEH) {
  typedef typename Impl::TargetPtrT TargetPtrT;

```
- **EN**: Implements logic around `success`, `processFDE`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `success`, `processFDE` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 279-287
```cpp
  LLVM_DEBUG(dbgs() << "Processing FDE: Delta for text: " << DeltaForText
                    << ", Delta for EH: " << DeltaForEH << "\n");
  uint32_t Length = readBytesUnaligned(P, 4);
  P += 4;
  uint8_t *Ret = P + Length;
  uint32_t Offset = readBytesUnaligned(P, 4);
  if (Offset == 0) // is a CIE
    return Ret;

```
- **EN**: Implements logic around `readBytesUnaligned`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `readBytesUnaligned` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 288-297
```cpp
  P += 4;
  TargetPtrT FDELocation = readBytesUnaligned(P, sizeof(TargetPtrT));
  TargetPtrT NewLocation = FDELocation - DeltaForText;
  writeBytesUnaligned(NewLocation, P, sizeof(TargetPtrT));

  P += sizeof(TargetPtrT);

  // Skip the FDE address range
  P += sizeof(TargetPtrT);

```
- **EN**: Implements logic around `readBytesUnaligned`, `writeBytesUnaligned`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `readBytesUnaligned`, `writeBytesUnaligned` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 298-308
```cpp
  uint8_t Augmentationsize = *P;
  P += 1;
  if (Augmentationsize != 0) {
    TargetPtrT LSDA = readBytesUnaligned(P, sizeof(TargetPtrT));
    TargetPtrT NewLSDA = LSDA - DeltaForEH;
    writeBytesUnaligned(NewLSDA, P, sizeof(TargetPtrT));
  }

  return Ret;
}

```
- **EN**: Implements logic around `readBytesUnaligned`, `writeBytesUnaligned`; this block drives emission, layout, or binary encoding behavior; returns subsystem-specific computed results.
- **CN**: 围绕 `readBytesUnaligned`, `writeBytesUnaligned` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，返回子系统相关的计算结果。

### Lines 309-318
```cpp
static int64_t computeDelta(SectionEntry *A, SectionEntry *B) {
  int64_t ObjDistance = static_cast<int64_t>(A->getObjAddress()) -
                        static_cast<int64_t>(B->getObjAddress());
  int64_t MemDistance = A->getLoadAddress() - B->getLoadAddress();
  return ObjDistance - MemDistance;
}

template <typename Impl>
void RuntimeDyldMachOCRTPBase<Impl>::registerEHFrames() {

```
- **EN**: Implements logic around `computeDelta`, `static_cast<int64_t>`, `getLoadAddress`, `registerEHFrames`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `computeDelta`, `static_cast<int64_t>`, `getLoadAddress`, `registerEHFrames` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 319-329
```cpp
  for (int i = 0, e = UnregisteredEHFrameSections.size(); i != e; ++i) {
    EHFrameRelatedSections &SectionInfo = UnregisteredEHFrameSections[i];
    if (SectionInfo.EHFrameSID == RTDYLD_INVALID_SECTION_ID ||
        SectionInfo.TextSID == RTDYLD_INVALID_SECTION_ID)
      continue;
    SectionEntry *Text = &Sections[SectionInfo.TextSID];
    SectionEntry *EHFrame = &Sections[SectionInfo.EHFrameSID];
    SectionEntry *ExceptTab = nullptr;
    if (SectionInfo.ExceptTabSID != RTDYLD_INVALID_SECTION_ID)
      ExceptTab = &Sections[SectionInfo.ExceptTabSID];

```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 330-340
```cpp
    int64_t DeltaForText = computeDelta(Text, EHFrame);
    int64_t DeltaForEH = 0;
    if (ExceptTab)
      DeltaForEH = computeDelta(ExceptTab, EHFrame);

    uint8_t *P = EHFrame->getAddress();
    uint8_t *End = P + EHFrame->getSize();
    while (P != End) {
      P = processFDE(P, DeltaForText, DeltaForEH);
    }

```
- **EN**: Implements logic around `computeDelta`, `getAddress`, `getSize`, `processFDE`.
- **CN**: 围绕 `computeDelta`, `getAddress`, `getSize`, `processFDE` 实现具体逻辑。

### Lines 341-358
```cpp
    MemMgr.registerEHFrames(EHFrame->getAddress(), EHFrame->getLoadAddress(),
                            EHFrame->getSize());
  }
  UnregisteredEHFrameSections.clear();
}

std::unique_ptr<RuntimeDyldMachO>
RuntimeDyldMachO::create(Triple::ArchType Arch,
                         RuntimeDyld::MemoryManager &MemMgr,
                         JITSymbolResolver &Resolver) {
  switch (Arch) {
  default:
    llvm_unreachable("Unsupported target for RuntimeDyldMachO.");
    break;
  case Triple::arm:
    return std::make_unique<RuntimeDyldMachOARM>(MemMgr, Resolver);
  case Triple::aarch64:
    return std::make_unique<RuntimeDyldMachOAArch64>(MemMgr, Resolver);
```
- **EN**: Implements logic around `registerEHFrames`, `getSize`, `clear`, `create`, and 3 more symbols; this block handles relocation, fixup, or symbol-resolution work; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `registerEHFrames`, `getSize`, `clear`, `create`, and 3 more symbols 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 359-367
```cpp
  case Triple::aarch64_32:
    return std::make_unique<RuntimeDyldMachOAArch64>(MemMgr, Resolver);
  case Triple::x86:
    return std::make_unique<RuntimeDyldMachOI386>(MemMgr, Resolver);
  case Triple::x86_64:
    return std::make_unique<RuntimeDyldMachOX86_64>(MemMgr, Resolver);
  }
}

```
- **EN**: Implements logic around `make_unique<RuntimeDyldMachOAArch64>`, `make_unique<RuntimeDyldMachOI386>`, `make_unique<RuntimeDyldMachOX86_64>`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `make_unique<RuntimeDyldMachOAArch64>`, `make_unique<RuntimeDyldMachOI386>`, `make_unique<RuntimeDyldMachOX86_64>` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 368-380
```cpp
std::unique_ptr<RuntimeDyld::LoadedObjectInfo>
RuntimeDyldMachO::loadObject(const object::ObjectFile &O) {
  if (auto ObjSectionToIDOrErr = loadObjectImpl(O))
    return std::make_unique<LoadedMachOObjectInfo>(*this,
                                                    *ObjSectionToIDOrErr);
  else {
    HasError = true;
    raw_string_ostream ErrStream(ErrorStr);
    logAllUnhandledErrors(ObjSectionToIDOrErr.takeError(), ErrStream);
    return nullptr;
  }
}

```
- **EN**: Implements logic around `loadObject`, `make_unique<LoadedMachOObjectInfo>`, `ErrStream`, `logAllUnhandledErrors`; this block handles relocation, fixup, or symbol-resolution work; returns subsystem-specific computed results.
- **CN**: 围绕 `loadObject`, `make_unique<LoadedMachOObjectInfo>`, `ErrStream`, `logAllUnhandledErrors` 实现具体逻辑；这一段处理重定位、fixup 或符号解析工作，返回子系统相关的计算结果。

### Lines 381-381
```cpp
} // end namespace llvm
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Runtime relocation / 运行时重定位**:
  - **EN**: Loads object code into memory and resolves relocations against runtime symbol tables
  - **CN**: 把目标代码装入内存并针对运行时符号表解析重定位
- **Relocations and fixups / 重定位与 fixup**:
  - **EN**: Bridges symbolic references to concrete addresses or relocation records
  - **CN**: 把符号引用连接到具体地址或重定位记录
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `RuntimeDyldMachO.h`, `Targets/RuntimeDyldMachOAArch64.h`, `Targets/RuntimeDyldMachOARM.h`, `Targets/RuntimeDyldMachOI386.h`, `Targets/RuntimeDyldMachOX86_64.h`, `llvm/ADT/StringRef.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
