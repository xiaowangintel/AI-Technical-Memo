# MCContext.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCContext.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Machine Code Context.
  - **CN**: 实现 MCContext 状态管理，用于维护符号、节区、源码位置以及目标文件格式元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- lib/MC/MCContext.cpp - Machine Code Context ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/MC/MCContext.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/COFF.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/BinaryFormat/GOFF.h"
#include "llvm/BinaryFormat/Wasm.h"
#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCCodeView.h"
#include "llvm/MC/MCDwarf.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCLabel.h"
#include "llvm/MC/MCSectionCOFF.h"
#include "llvm/MC/MCSectionDXContainer.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCSectionGOFF.h"
#include "llvm/MC/MCSectionMachO.h"
#include "llvm/MC/MCSectionSPIRV.h"
#include "llvm/MC/MCSectionWasm.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCContext.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCContext.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`。

### Lines 33-57
```cpp
#include "llvm/MC/MCSectionXCOFF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolCOFF.h"
#include "llvm/MC/MCSymbolELF.h"
#include "llvm/MC/MCSymbolGOFF.h"
#include "llvm/MC/MCSymbolMachO.h"
#include "llvm/MC/MCSymbolWasm.h"
#include "llvm/MC/MCSymbolXCOFF.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/MC/SectionKind.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MemoryBuffer.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdlib>
#include <optional>
#include <tuple>
#include <utility>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCSectionXCOFF.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCSectionXCOFF.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`。

### Lines 58-80
```cpp
using namespace llvm;

static void defaultDiagHandler(const SMDiagnostic &SMD, bool, const SourceMgr &,
                               std::vector<const MDNode *> &) {
  SMD.print(nullptr, errs());
}

MCContext::MCContext(const Triple &TheTriple, const MCAsmInfo &mai,
                     const MCRegisterInfo &mri, const MCSubtargetInfo &msti,
                     const SourceMgr *mgr, bool DoAutoReset,
                     StringRef Swift5ReflSegmentName)
    : Swift5ReflectionSegmentName(Swift5ReflSegmentName), TT(TheTriple),
      SrcMgr(mgr), InlineSrcMgr(nullptr), DiagHandler(defaultDiagHandler),
      MAI(mai), MRI(&mri), MSTI(&msti), Symbols(Allocator),
      InlineAsmUsedLabelNames(Allocator),
      CurrentDwarfLoc(0, 0, 0, DWARF2_FLAG_IS_STMT, 0, 0),
      AutoReset(DoAutoReset) {
  const MCTargetOptions &TO = getTargetOptions();
  SaveTempLabels = TO.MCSaveTempLabels;
  if (SaveTempLabels)
    setUseNamesOnTempLabels(true);
  SecureLogFile = TO.AsSecureLogFile;

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 81-112
```cpp
  if (SrcMgr && SrcMgr->getNumBuffers())
    MainFileName = std::string(SrcMgr->getMemoryBuffer(SrcMgr->getMainFileID())
                                   ->getBufferIdentifier());

  switch (TheTriple.getObjectFormat()) {
  case Triple::MachO:
    Env = IsMachO;
    break;
  case Triple::COFF:
    if (!TheTriple.isOSWindows() && !TheTriple.isUEFI()) {
      reportFatalUsageError(
          "cannot initialize MC for non-Windows COFF object files");
    }

    Env = IsCOFF;
    break;
  case Triple::ELF:
    Env = IsELF;
    break;
  case Triple::Wasm:
    Env = IsWasm;
    break;
  case Triple::XCOFF:
    Env = IsXCOFF;
    break;
  case Triple::GOFF:
    Env = IsGOFF;
    break;
  case Triple::DXContainer:
    Env = IsDXContainer;
    break;
  case Triple::SPIRV:
```
- **EN**: Implements logic around `string`, `getBufferIdentifier`, `reportFatalUsageError`; this block uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `string`, `getBufferIdentifier`, `reportFatalUsageError` 实现具体逻辑；这一段使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 113-128
```cpp
    Env = IsSPIRV;
    break;
  case Triple::UnknownObjectFormat:
    report_fatal_error("Cannot initialize MC for unknown object file format.");
    break;
  }
}

const MCTargetOptions &MCContext::getTargetOptions() const {
  return MAI.getTargetOptions();
}

MCContext::~MCContext() {
  if (AutoReset)
    reset();

```
- **EN**: Implements logic around `report_fatal_error`, `getTargetOptions`, `~MCContext`, `reset`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `report_fatal_error`, `getTargetOptions`, `~MCContext`, `reset` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 129-147
```cpp
  // NOTE: The symbols are all allocated out of a bump pointer allocator,
  // we don't need to free them here.
}

void MCContext::initInlineSourceManager() {
  if (!InlineSrcMgr)
    InlineSrcMgr.reset(new SourceMgr());
}

//===----------------------------------------------------------------------===//
// Module Lifetime Management
//===----------------------------------------------------------------------===//

void MCContext::reset() {
  SrcMgr = nullptr;
  InlineSrcMgr.reset();
  LocInfos.clear();
  DiagHandler = defaultDiagHandler;

```
- **EN**: Implements logic around `initInlineSourceManager`, `reset`, `clear`; this block updates MC section or symbol state.
- **CN**: 围绕 `initInlineSourceManager`, `reset`, `clear` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 148-175
```cpp
  // Call the destructors so the fragments are freed
  COFFAllocator.DestroyAll();
  DXCAllocator.DestroyAll();
  ELFAllocator.DestroyAll();
  GOFFAllocator.DestroyAll();
  MachOAllocator.DestroyAll();
  WasmAllocator.DestroyAll();
  XCOFFAllocator.DestroyAll();
  MCInstAllocator.DestroyAll();
  SPIRVAllocator.DestroyAll();
  WasmSignatureAllocator.DestroyAll();

  CVContext.reset();

  MCSubtargetAllocator.DestroyAll();
  InlineAsmUsedLabelNames.clear();
  Symbols.clear();
  Allocator.Reset();
  Instances.clear();
  CompilationDir.clear();
  MainFileName.clear();
  MCDwarfLineTablesCUMap.clear();
  SectionsForRanges.clear();
  MCGenDwarfLabelEntries.clear();
  DwarfDebugFlags = StringRef();
  DwarfCompileUnitID = 0;
  CurrentDwarfLoc = MCDwarfLoc(0, 0, 0, DWARF2_FLAG_IS_STMT, 0, 0);

```
- **EN**: Implements logic around `DestroyAll`, `reset`, `clear`, `Reset`, and 2 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `DestroyAll`, `reset`, `clear`, `Reset`, and 2 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 176-192
```cpp
  MachOUniquingMap.clear();
  ELFUniquingMap.clear();
  GOFFUniquingMap.clear();
  COFFUniquingMap.clear();
  WasmUniquingMap.clear();
  XCOFFUniquingMap.clear();
  DXCUniquingMap.clear();

  RelSecNames.clear();
  MacroMap.clear();
  ELFEntrySizeMap.clear();
  ELFSeenGenericMergeableSections.clear();

  DwarfLocSeen = false;
  GenDwarfForAssembly = false;
  GenDwarfFileNumber = 0;

```
- **EN**: Implements logic around `clear`; this block updates MC section or symbol state.
- **CN**: 围绕 `clear` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 193-211
```cpp
  HadError = false;
}

//===----------------------------------------------------------------------===//
// MCInst Management
//===----------------------------------------------------------------------===//

MCInst *MCContext::createMCInst() {
  return new (MCInstAllocator.Allocate()) MCInst;
}

//===----------------------------------------------------------------------===//
// Symbol Manipulation
//===----------------------------------------------------------------------===//

MCSymbol *MCContext::getOrCreateSymbol(const Twine &Name) {
  SmallString<128> NameSV;
  StringRef NameRef = Name.toStringRef(NameSV);

```
- **EN**: Implements logic around `createMCInst`, `new`, `getOrCreateSymbol`, `toStringRef`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createMCInst`, `new`, `getOrCreateSymbol`, `toStringRef` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 212-227
```cpp
  assert(!NameRef.empty() && "Normal symbols cannot be unnamed!");

  MCSymbolTableEntry &Entry = getSymbolTableEntry(NameRef);
  if (!Entry.second.Symbol) {
    bool IsRenamable = NameRef.starts_with(MAI.getInternalSymbolPrefix());
    bool IsTemporary = IsRenamable && !SaveTempLabels;
    if (!Entry.second.Used) {
      Entry.second.Used = true;
      Entry.second.Symbol = createSymbolImpl(&Entry, IsTemporary);
    } else {
      assert(IsRenamable && "cannot rename non-private symbol");
      // Slow path: we need to rename a temp symbol from the user.
      Entry.second.Symbol = createRenamableSymbol(NameRef, false, IsTemporary);
    }
  }

```
- **EN**: Implements logic around `assert`, `getSymbolTableEntry`, `starts_with`, `createSymbolImpl`, and 1 more symbols; this block updates MC section or symbol state.
- **CN**: 围绕 `assert`, `getSymbolTableEntry`, `starts_with`, `createSymbolImpl`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 228-255
```cpp
  return Entry.second.Symbol;
}

MCSymbol *MCContext::parseSymbol(const Twine &Name) {
  SmallString<128> SV;
  StringRef NameRef = Name.toStringRef(SV);
  if (NameRef.contains('\\')) {
    SV = NameRef;
    size_t S = 0;
    // Support escaped \\ and \" as in GNU Assembler. GAS issues a warning for
    // other characters following \\, which we do not implement due to code
    // structure.
    for (size_t I = 0, E = SV.size(); I != E; ++I) {
      char C = SV[I];
      if (C == '\\' && I + 1 != E) {
        switch (SV[I + 1]) {
        case '"':
        case '\\':
          C = SV[++I];
          break;
        }
      }
      SV[S++] = C;
    }
    SV.resize(S);
    NameRef = SV;
  }

```
- **EN**: Implements logic around `parseSymbol`, `toStringRef`, `resize`; this block parses assembly syntax or operands; updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `parseSymbol`, `toStringRef`, `resize` 实现具体逻辑；这一段解析汇编语法或操作数，更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 256-274
```cpp
  return getOrCreateSymbol(NameRef);
}

MCSymbol *MCContext::getOrCreateFrameAllocSymbol(const Twine &FuncName,
                                                 unsigned Idx) {
  return getOrCreateSymbol(MAI.getInternalSymbolPrefix() + FuncName +
                           "$frame_escape_" + Twine(Idx));
}

MCSymbol *MCContext::getOrCreateParentFrameOffsetSymbol(const Twine &FuncName) {
  return getOrCreateSymbol(MAI.getInternalSymbolPrefix() + FuncName +
                           "$parent_frame_offset");
}

MCSymbol *MCContext::getOrCreateLSDASymbol(const Twine &FuncName) {
  return getOrCreateSymbol(MAI.getInternalSymbolPrefix() + "__ehtable$" +
                           FuncName);
}

```
- **EN**: Implements logic around `getOrCreateSymbol`, `getOrCreateFrameAllocSymbol`, `Twine`, `getOrCreateParentFrameOffsetSymbol`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateSymbol`, `getOrCreateFrameAllocSymbol`, `Twine`, `getOrCreateParentFrameOffsetSymbol`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 275-291
```cpp
MCSymbolTableEntry &MCContext::getSymbolTableEntry(StringRef Name) {
  return *Symbols.try_emplace(Name, MCSymbolTableValue{}).first;
}

MCSymbol *MCContext::createSymbolImpl(const MCSymbolTableEntry *Name,
                                      bool IsTemporary) {
  static_assert(std::is_trivially_destructible<MCSymbolCOFF>(),
                "MCSymbol classes must be trivially destructible");
  static_assert(std::is_trivially_destructible<MCSymbolELF>(),
                "MCSymbol classes must be trivially destructible");
  static_assert(std::is_trivially_destructible<MCSymbolMachO>(),
                "MCSymbol classes must be trivially destructible");
  static_assert(std::is_trivially_destructible<MCSymbolWasm>(),
                "MCSymbol classes must be trivially destructible");
  static_assert(std::is_trivially_destructible<MCSymbolXCOFF>(),
                "MCSymbol classes must be trivially destructible");

```
- **EN**: Implements logic around `getSymbolTableEntry`, `try_emplace`, `createSymbolImpl`, `static_assert`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSymbolTableEntry`, `try_emplace`, `createSymbolImpl`, `static_assert` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 292-312
```cpp
  switch (getObjectFileType()) {
  case MCContext::IsCOFF:
    return new (Name, *this) MCSymbolCOFF(Name, IsTemporary);
  case MCContext::IsELF:
    return new (Name, *this) MCSymbolELF(Name, IsTemporary);
  case MCContext::IsGOFF:
    return new (Name, *this) MCSymbolGOFF(Name, IsTemporary);
  case MCContext::IsMachO:
    return new (Name, *this) MCSymbolMachO(Name, IsTemporary);
  case MCContext::IsWasm:
    return new (Name, *this) MCSymbolWasm(Name, IsTemporary);
  case MCContext::IsXCOFF:
    return createXCOFFSymbolImpl(Name, IsTemporary);
  case MCContext::IsDXContainer:
    break;
  case MCContext::IsSPIRV:
    return new (Name, *this) MCSymbol(Name, IsTemporary);
  }
  return new (Name, *this) MCSymbol(Name, IsTemporary);
}

```
- **EN**: Implements logic around `new`, `createXCOFFSymbolImpl`; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `new`, `createXCOFFSymbolImpl` 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 313-338
```cpp
MCSymbol *MCContext::cloneSymbol(MCSymbol &Sym) {
  MCSymbol *NewSym = nullptr;
  auto Name = Sym.getNameEntryPtr();
  switch (getObjectFileType()) {
  case MCContext::IsCOFF:
    NewSym =
        new (Name, *this) MCSymbolCOFF(static_cast<const MCSymbolCOFF &>(Sym));
    break;
  case MCContext::IsELF:
    NewSym =
        new (Name, *this) MCSymbolELF(static_cast<const MCSymbolELF &>(Sym));
    break;
  case MCContext::IsMachO:
    NewSym = new (Name, *this)
        MCSymbolMachO(static_cast<const MCSymbolMachO &>(Sym));
    break;
  default:
    reportFatalUsageError(".set redefinition is not supported");
    break;
  }
  // Set the name and redirect the `Symbols` entry to `NewSym`.
  NewSym->getNameEntryPtr() = Name;
  const_cast<MCSymbolTableEntry *>(Name)->second.Symbol = NewSym;
  // Ensure the next `registerSymbol` call will add the new symbol to `Symbols`.
  NewSym->setIsRegistered(false);

```
- **EN**: Implements logic around `cloneSymbol`, `getNameEntryPtr`, `new`, `MCSymbolMachO`, and 2 more symbols; this block updates MC section or symbol state; uses `switch`-based dispatch over enums, opcodes, or kinds.
- **CN**: 围绕 `cloneSymbol`, `getNameEntryPtr`, `new`, `MCSymbolMachO`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，使用 `switch` 对枚举、opcode 或 kind 进行分派。

### Lines 339-355
```cpp
  // Ensure the original symbol is not emitted to the symbol table.
  Sym.IsTemporary = true;
  return NewSym;
}

MCSymbol *MCContext::createRenamableSymbol(const Twine &Name,
                                           bool AlwaysAddSuffix,
                                           bool IsTemporary) {
  SmallString<128> NewName;
  Name.toVector(NewName);
  size_t NameLen = NewName.size();

  MCSymbolTableEntry &NameEntry = getSymbolTableEntry(NewName.str());
  MCSymbolTableEntry *EntryPtr = &NameEntry;
  while (AlwaysAddSuffix || EntryPtr->second.Used) {
    AlwaysAddSuffix = false;

```
- **EN**: Implements logic around `createRenamableSymbol`, `toVector`, `size`, `getSymbolTableEntry`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createRenamableSymbol`, `toVector`, `size`, `getSymbolTableEntry` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 356-371
```cpp
    NewName.resize(NameLen);
    raw_svector_ostream(NewName) << NameEntry.second.NextUniqueID++;
    EntryPtr = &getSymbolTableEntry(NewName.str());
  }

  EntryPtr->second.Used = true;
  return createSymbolImpl(EntryPtr, IsTemporary);
}

MCSymbol *MCContext::createTempSymbol(const Twine &Name, bool AlwaysAddSuffix) {
  if (!UseNamesOnTempLabels)
    return createSymbolImpl(nullptr, /*IsTemporary=*/true);
  return createRenamableSymbol(MAI.getInternalSymbolPrefix() + Name,
                               AlwaysAddSuffix, /*IsTemporary=*/true);
}

```
- **EN**: Implements logic around `resize`, `raw_svector_ostream`, `getSymbolTableEntry`, `createSymbolImpl`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `resize`, `raw_svector_ostream`, `getSymbolTableEntry`, `createSymbolImpl`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 372-387
```cpp
MCSymbol *MCContext::createNamedTempSymbol(const Twine &Name) {
  return createRenamableSymbol(MAI.getInternalSymbolPrefix() + Name, true,
                               /*IsTemporary=*/!SaveTempLabels);
}

MCSymbol *MCContext::createBlockSymbol(const Twine &Name, bool AlwaysEmit) {
  if (AlwaysEmit)
    return getOrCreateSymbol(MAI.getPrivateLabelPrefix() + Name);

  bool IsTemporary = !SaveTempLabels;
  if (IsTemporary && !UseNamesOnTempLabels)
    return createSymbolImpl(nullptr, IsTemporary);
  return createRenamableSymbol(MAI.getPrivateLabelPrefix() + Name,
                               /*AlwaysAddSuffix=*/false, IsTemporary);
}

```
- **EN**: Implements logic around `createNamedTempSymbol`, `createRenamableSymbol`, `createBlockSymbol`, `getOrCreateSymbol`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createNamedTempSymbol`, `createRenamableSymbol`, `createBlockSymbol`, `getOrCreateSymbol`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 388-403
```cpp
MCSymbol *MCContext::createLinkerPrivateTempSymbol() {
  return createLinkerPrivateSymbol("tmp");
}

MCSymbol *MCContext::createLinkerPrivateSymbol(const Twine &Name) {
  return createRenamableSymbol(MAI.getLinkerPrivateGlobalPrefix() + Name,
                               /*AlwaysAddSuffix=*/true,
                               /*IsTemporary=*/false);
}

MCSymbol *MCContext::createTempSymbol() { return createTempSymbol("tmp"); }

MCSymbol *MCContext::createNamedTempSymbol() {
  return createNamedTempSymbol("tmp");
}

```
- **EN**: Implements logic around `createLinkerPrivateTempSymbol`, `createLinkerPrivateSymbol`, `createRenamableSymbol`, `createTempSymbol`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createLinkerPrivateTempSymbol`, `createLinkerPrivateSymbol`, `createRenamableSymbol`, `createTempSymbol`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 404-422
```cpp
MCSymbol *MCContext::createLocalSymbol(StringRef Name) {
  MCSymbolTableEntry &NameEntry = getSymbolTableEntry(Name);
  return createSymbolImpl(&NameEntry, /*IsTemporary=*/false);
}

unsigned MCContext::NextInstance(unsigned LocalLabelVal) {
  MCLabel *&Label = Instances[LocalLabelVal];
  if (!Label)
    Label = new (*this) MCLabel(0);
  return Label->incInstance();
}

unsigned MCContext::GetInstance(unsigned LocalLabelVal) {
  MCLabel *&Label = Instances[LocalLabelVal];
  if (!Label)
    Label = new (*this) MCLabel(0);
  return Label->getInstance();
}

```
- **EN**: Implements logic around `createLocalSymbol`, `getSymbolTableEntry`, `createSymbolImpl`, `NextInstance`, and 4 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createLocalSymbol`, `getSymbolTableEntry`, `createSymbolImpl`, `NextInstance`, and 4 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 423-443
```cpp
MCSymbol *MCContext::getOrCreateDirectionalLocalSymbol(unsigned LocalLabelVal,
                                                       unsigned Instance) {
  MCSymbol *&Sym = LocalSymbols[std::make_pair(LocalLabelVal, Instance)];
  if (!Sym)
    Sym = createNamedTempSymbol();
  return Sym;
}

MCSymbol *MCContext::createDirectionalLocalSymbol(unsigned LocalLabelVal) {
  unsigned Instance = NextInstance(LocalLabelVal);
  return getOrCreateDirectionalLocalSymbol(LocalLabelVal, Instance);
}

MCSymbol *MCContext::getDirectionalLocalSymbol(unsigned LocalLabelVal,
                                               bool Before) {
  unsigned Instance = GetInstance(LocalLabelVal);
  if (!Before)
    ++Instance;
  return getOrCreateDirectionalLocalSymbol(LocalLabelVal, Instance);
}

```
- **EN**: Implements logic around `getOrCreateDirectionalLocalSymbol`, `make_pair`, `createNamedTempSymbol`, `createDirectionalLocalSymbol`, and 3 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateDirectionalLocalSymbol`, `make_pair`, `createNamedTempSymbol`, `createDirectionalLocalSymbol`, and 3 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 444-472
```cpp
// Create a section symbol, with a distinct one for each section of the same.
// The first symbol is used for assembly code references.
template <typename Symbol>
Symbol *MCContext::getOrCreateSectionSymbol(StringRef Section) {
  Symbol *R;
  auto &SymEntry = getSymbolTableEntry(Section);
  MCSymbol *Sym = SymEntry.second.Symbol;
  if (Sym && Sym->isDefined() &&
      (!Sym->isInSection() || Sym->getSection().getBeginSymbol() != Sym)) {
    reportError(SMLoc(), "invalid symbol redefinition");
    // Don't reuse the conflicting symbol (e.g. an equated symbol from `x=0`)
    // as a section symbol, which would cause a crash in changeSection.
    Sym = nullptr;
  }
  // Use the symbol's index to track if it has been used as a section symbol.
  // Set to -1 to catch potential bugs if misused as a symbol index.
  if (Sym && Sym->getIndex() != -1u) {
    R = static_cast<Symbol *>(Sym);
  } else {
    SymEntry.second.Used = true;
    R = new (&SymEntry, *this) Symbol(&SymEntry, /*isTemporary=*/false);
    if (!Sym)
      SymEntry.second.Symbol = R;
  }
  // Mark as section symbol.
  R->setIndex(-1u);
  return R;
}

```
- **EN**: Implements logic around `getOrCreateSectionSymbol`, `getSymbolTableEntry`, `isInSection`, `reportError`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getOrCreateSectionSymbol`, `getSymbolTableEntry`, `isInSection`, `reportError`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 473-488
```cpp
MCSymbol *MCContext::lookupSymbol(const Twine &Name) const {
  SmallString<128> NameSV;
  StringRef NameRef = Name.toStringRef(NameSV);
  return Symbols.lookup(NameRef).Symbol;
}

void MCContext::setSymbolValue(MCStreamer &Streamer, const Twine &Sym,
                               uint64_t Val) {
  auto Symbol = getOrCreateSymbol(Sym);
  Streamer.emitAssignment(Symbol, MCConstantExpr::create(Val, *this));
}

void MCContext::registerInlineAsmLabel(MCSymbol *Sym) {
  InlineAsmUsedLabelNames[Sym->getName()] = Sym;
}

```
- **EN**: Implements logic around `lookupSymbol`, `toStringRef`, `lookup`, `setSymbolValue`, and 4 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `lookupSymbol`, `toStringRef`, `lookup`, `setSymbolValue`, and 4 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 489-505
```cpp
wasm::WasmSignature *MCContext::createWasmSignature() {
  return new (WasmSignatureAllocator.Allocate()) wasm::WasmSignature;
}

MCSymbolXCOFF *MCContext::createXCOFFSymbolImpl(const MCSymbolTableEntry *Name,
                                                bool IsTemporary) {
  if (!Name)
    return new (nullptr, *this) MCSymbolXCOFF(nullptr, IsTemporary);

  StringRef OriginalName = Name->first();
  if (OriginalName.starts_with("._Renamed..") ||
      OriginalName.starts_with("_Renamed.."))
    reportError(SMLoc(), "invalid symbol name from source");

  if (MAI.isValidUnquotedName(OriginalName))
    return new (Name, *this) MCSymbolXCOFF(Name, IsTemporary);

```
- **EN**: Implements logic around `createWasmSignature`, `new`, `createXCOFFSymbolImpl`, `first`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createWasmSignature`, `new`, `createXCOFFSymbolImpl`, `first`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 506-526
```cpp
  // Now we have a name that contains invalid character(s) for XCOFF symbol.
  // Let's replace with something valid, but save the original name so that
  // we could still use the original name in the symbol table.
  SmallString<128> InvalidName(OriginalName);

  // If it's an entry point symbol, we will keep the '.'
  // in front for the convention purpose. Otherwise, add "_Renamed.."
  // as prefix to signal this is an renamed symbol.
  const bool IsEntryPoint = InvalidName.starts_with(".");
  SmallString<128> ValidName =
      StringRef(IsEntryPoint ? "._Renamed.." : "_Renamed..");

  // Append the hex values of '_' and invalid characters with "_Renamed..";
  // at the same time replace invalid characters with '_'.
  for (char &C : InvalidName) {
    if (!MAI.isAcceptableChar(C) || C == '_') {
      raw_svector_ostream(ValidName).write_hex(C);
      C = '_';
    }
  }

```
- **EN**: Implements logic around `InvalidName`, `starts_with`, `StringRef`, `raw_svector_ostream`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state.
- **CN**: 围绕 `InvalidName`, `starts_with`, `StringRef`, `raw_svector_ostream` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态。

### Lines 527-544
```cpp
  // Skip entry point symbol's '.' as we already have a '.' in front of
  // "_Renamed".
  if (IsEntryPoint)
    ValidName.append(InvalidName.substr(1, InvalidName.size() - 1));
  else
    ValidName.append(InvalidName);

  MCSymbolTableEntry &NameEntry = getSymbolTableEntry(ValidName.str());
  assert(!NameEntry.second.Used && "This name is used somewhere else.");
  NameEntry.second.Used = true;
  // Have the MCSymbol object itself refer to the copy of the string
  // that is embedded in the symbol table entry.
  MCSymbolXCOFF *XSym =
      new (&NameEntry, *this) MCSymbolXCOFF(&NameEntry, IsTemporary);
  XSym->setSymbolTableName(MCSymbolXCOFF::getUnqualifiedName(OriginalName));
  return XSym;
}

```
- **EN**: Implements logic around `append`, `getSymbolTableEntry`, `assert`, `new`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `append`, `getSymbolTableEntry`, `assert`, `new`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 545-561
```cpp
//===----------------------------------------------------------------------===//
// Section Management
//===----------------------------------------------------------------------===//

MCSectionMachO *MCContext::getMachOSection(StringRef Segment, StringRef Section,
                                           unsigned TypeAndAttributes,
                                           unsigned Reserved2, SectionKind Kind,
                                           const char *BeginSymName) {
  // We unique sections by their segment/section pair.  The returned section
  // may not have the same flags as the requested section, if so this should be
  // diagnosed by the client as an error.

  // Form the name to look up.
  assert(Section.size() <= 16 && "section name is too long");
  assert(!memchr(Section.data(), '\0', Section.size()) &&
         "section name cannot contain NUL");

```
- **EN**: Implements logic around `getMachOSection`, `assert`; this block updates MC section or symbol state.
- **CN**: 围绕 `getMachOSection`, `assert` 实现具体逻辑；这一段更新 MC 节区或符号状态。

### Lines 562-579
```cpp
  // Do the lookup, if we have a hit, return it.
  auto R = MachOUniquingMap.try_emplace((Segment + Twine(',') + Section).str());
  if (!R.second)
    return R.first->second;

  MCSymbol *Begin = nullptr;
  if (BeginSymName)
    Begin = createTempSymbol(BeginSymName, false);

  // Otherwise, return a new section.
  StringRef Name = R.first->first();
  auto *Ret = new (MachOAllocator.Allocate())
      MCSectionMachO(Segment, Name.substr(Name.size() - Section.size()),
                     TypeAndAttributes, Reserved2, Kind, Begin);
  R.first->second = Ret;
  return Ret;
}

```
- **EN**: Implements logic around `try_emplace`, `createTempSymbol`, `first`, `new`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `try_emplace`, `createTempSymbol`, `first`, `new`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 580-598
```cpp
MCSectionELF *MCContext::createELFSectionImpl(StringRef Section, unsigned Type,
                                              unsigned Flags,
                                              unsigned EntrySize,
                                              const MCSymbolELF *Group,
                                              bool Comdat, unsigned UniqueID,
                                              const MCSymbolELF *LinkedToSym) {
  auto *R = getOrCreateSectionSymbol<MCSymbolELF>(Section);
  return new (ELFAllocator.Allocate()) MCSectionELF(
      Section, Type, Flags, EntrySize, Group, Comdat, UniqueID, R, LinkedToSym);
}

MCSectionELF *
MCContext::createELFRelSection(const Twine &Name, unsigned Type, unsigned Flags,
                               unsigned EntrySize, const MCSymbolELF *Group,
                               const MCSectionELF *RelInfoSection) {
  StringMap<bool>::iterator I;
  bool Inserted;
  std::tie(I, Inserted) = RelSecNames.insert(std::make_pair(Name.str(), true));

```
- **EN**: Implements logic around `createELFSectionImpl`, `getOrCreateSectionSymbol<MCSymbolELF>`, `new`, `createELFRelSection`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createELFSectionImpl`, `getOrCreateSectionSymbol<MCSymbolELF>`, `new`, `createELFRelSection`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 599-620
```cpp
  return createELFSectionImpl(
      I->getKey(), Type, Flags, EntrySize, Group, true, true,
      static_cast<const MCSymbolELF *>(RelInfoSection->getBeginSymbol()));
}

MCSectionELF *MCContext::getELFNamedSection(const Twine &Prefix,
                                            const Twine &Suffix, unsigned Type,
                                            unsigned Flags,
                                            unsigned EntrySize) {
  return getELFSection(Prefix + "." + Suffix, Type, Flags, EntrySize, Suffix,
                       /*IsComdat=*/true);
}

MCSectionELF *MCContext::getELFSection(const Twine &Section, unsigned Type,
                                       unsigned Flags, unsigned EntrySize,
                                       const Twine &Group, bool IsComdat,
                                       unsigned UniqueID,
                                       const MCSymbolELF *LinkedToSym) {
  MCSymbolELF *GroupSym = nullptr;
  if (!Group.isTriviallyEmpty() && !Group.str().empty())
    GroupSym = static_cast<MCSymbolELF *>(getOrCreateSymbol(Group));

```
- **EN**: Implements logic around `createELFSectionImpl`, `getKey`, `getBeginSymbol`, `getELFNamedSection`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createELFSectionImpl`, `getKey`, `getBeginSymbol`, `getELFNamedSection`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 621-652
```cpp
  return getELFSection(Section, Type, Flags, EntrySize, GroupSym, IsComdat,
                       UniqueID, LinkedToSym);
}

MCSectionELF *MCContext::getELFSection(const Twine &Section, unsigned Type,
                                       unsigned Flags, unsigned EntrySize,
                                       const MCSymbolELF *GroupSym,
                                       bool IsComdat, unsigned UniqueID,
                                       const MCSymbolELF *LinkedToSym) {
  assert(!(LinkedToSym && LinkedToSym->getName().empty()));

  // Sections are differentiated by the quadruple (section_name, group_name,
  // unique_id, link_to_symbol_name). Sections sharing the same quadruple are
  // combined into one section. As an optimization, non-unique sections without
  // group or linked-to symbol have a shorter unique-ing key.
  std::pair<StringMap<MCSectionELF *>::iterator, bool> EntryNewPair;
  // Length of the section name, which are the first SectionLen bytes of the key
  unsigned SectionLen;
  if (GroupSym || LinkedToSym || UniqueID != MCSection::NonUniqueID) {
    SmallString<128> Buffer;
    Section.toVector(Buffer);
    SectionLen = Buffer.size();
    Buffer.push_back(0); // separator which cannot occur in the name
    if (GroupSym)
      Buffer.append(GroupSym->getName());
    Buffer.push_back(0); // separator which cannot occur in the name
    if (LinkedToSym)
      Buffer.append(LinkedToSym->getName());
    support::endian::write(Buffer, UniqueID, endianness::native);
    StringRef UniqueMapKey = StringRef(Buffer);
    EntryNewPair = ELFUniquingMap.try_emplace(UniqueMapKey);
  } else if (!Section.isSingleStringRef()) {
```
- **EN**: Implements logic around `getELFSection`, `assert`, `toVector`, `size`, and 5 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getELFSection`, `assert`, `toVector`, `size`, and 5 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 653-672
```cpp
    SmallString<128> Buffer;
    StringRef UniqueMapKey = Section.toStringRef(Buffer);
    SectionLen = UniqueMapKey.size();
    EntryNewPair = ELFUniquingMap.try_emplace(UniqueMapKey);
  } else {
    StringRef UniqueMapKey = Section.getSingleStringRef();
    SectionLen = UniqueMapKey.size();
    EntryNewPair = ELFUniquingMap.try_emplace(UniqueMapKey);
  }

  if (!EntryNewPair.second)
    return EntryNewPair.first->second;

  StringRef CachedName = EntryNewPair.first->getKey().take_front(SectionLen);

  MCSectionELF *Result =
      createELFSectionImpl(CachedName, Type, Flags, EntrySize, GroupSym,
                           IsComdat, UniqueID, LinkedToSym);
  EntryNewPair.first->second = Result;

```
- **EN**: Implements logic around `toStringRef`, `size`, `try_emplace`, `getSingleStringRef`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `toStringRef`, `size`, `try_emplace`, `getSingleStringRef`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 673-695
```cpp
  recordELFMergeableSectionInfo(Result->getName(), Result->getFlags(),
                                Result->getUniqueID(), Result->getEntrySize());

  return Result;
}

MCSectionELF *MCContext::createELFGroupSection(const MCSymbolELF *Group,
                                               bool IsComdat) {
  return createELFSectionImpl(".group", ELF::SHT_GROUP, 0, 4, Group, IsComdat,
                              MCSection::NonUniqueID, nullptr);
}

void MCContext::recordELFMergeableSectionInfo(StringRef SectionName,
                                              unsigned Flags, unsigned UniqueID,
                                              unsigned EntrySize) {
  bool IsMergeable = Flags & ELF::SHF_MERGE;
  if (UniqueID == MCSection::NonUniqueID) {
    ELFSeenGenericMergeableSections.insert(SectionName);
    // Minor performance optimization: avoid hash map lookup in
    // isELFGenericMergeableSection, which will return true for SectionName.
    IsMergeable = true;
  }

```
- **EN**: Implements logic around `recordELFMergeableSectionInfo`, `getUniqueID`, `createELFGroupSection`, `createELFSectionImpl`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `recordELFMergeableSectionInfo`, `getUniqueID`, `createELFGroupSection`, `createELFSectionImpl`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 696-715
```cpp
  // For mergeable sections or non-mergeable sections with a generic mergeable
  // section name we enter their Unique ID into the ELFEntrySizeMap so that
  // compatible globals can be assigned to the same section.

  if (IsMergeable || isELFGenericMergeableSection(SectionName)) {
    ELFEntrySizeMap.insert(std::make_pair(
        std::make_tuple(SectionName, Flags, EntrySize), UniqueID));
  }
}

bool MCContext::isELFImplicitMergeableSectionNamePrefix(StringRef SectionName) {
  return SectionName.starts_with(".rodata.str") ||
         SectionName.starts_with(".rodata.cst");
}

bool MCContext::isELFGenericMergeableSection(StringRef SectionName) {
  return isELFImplicitMergeableSectionNamePrefix(SectionName) ||
         ELFSeenGenericMergeableSections.count(SectionName);
}

```
- **EN**: Implements logic around `insert`, `make_tuple`, `isELFImplicitMergeableSectionNamePrefix`, `starts_with`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `insert`, `make_tuple`, `isELFImplicitMergeableSectionNamePrefix`, `starts_with`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 716-738
```cpp
std::optional<unsigned>
MCContext::getELFUniqueIDForEntsize(StringRef SectionName, unsigned Flags,
                                    unsigned EntrySize) {
  auto I = ELFEntrySizeMap.find(std::make_tuple(SectionName, Flags, EntrySize));
  return (I != ELFEntrySizeMap.end()) ? std::optional<unsigned>(I->second)
                                      : std::nullopt;
}

template <typename TAttr>
MCSectionGOFF *MCContext::getGOFFSection(SectionKind Kind, StringRef Name,
                                         TAttr Attributes, MCSection *Parent,
                                         bool IsVirtual) {
  std::string UniqueName(Name);
  if (Parent) {
    UniqueName.append("/").append(Parent->getName());
    if (auto *P = static_cast<MCSectionGOFF *>(Parent)->getParent())
      UniqueName.append("/").append(P->getName());
  }
  // Do the lookup. If we don't have a hit, return a new section.
  auto [Iter, Inserted] = GOFFUniquingMap.try_emplace(UniqueName);
  if (!Inserted)
    return Iter->second;

```
- **EN**: Implements logic around `getELFUniqueIDForEntsize`, `find`, `getGOFFSection`, `UniqueName`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getELFUniqueIDForEntsize`, `find`, `getGOFFSection`, `UniqueName`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 739-758
```cpp
  StringRef CachedName = StringRef(Iter->first.c_str(), Name.size());
  MCSectionGOFF *GOFFSection = new (GOFFAllocator.Allocate())
      MCSectionGOFF(CachedName, Kind, IsVirtual, Attributes,
                    static_cast<MCSectionGOFF *>(Parent));
  Iter->second = GOFFSection;
  return GOFFSection;
}

MCSectionGOFF *MCContext::getGOFFSection(SectionKind Kind, StringRef Name,
                                         GOFF::SDAttr SDAttributes) {
  return getGOFFSection<GOFF::SDAttr>(Kind, Name, SDAttributes, nullptr,
                                      /*IsVirtual=*/true);
}

MCSectionGOFF *MCContext::getGOFFSection(SectionKind Kind, StringRef Name,
                                         GOFF::EDAttr EDAttributes,
                                         MCSection *Parent) {
  return getGOFFSection<GOFF::EDAttr>(
      Kind, Name, EDAttributes, Parent,
      /*IsVirtual=*/EDAttributes.BindAlgorithm == GOFF::ESD_BA_Merge);
```
- **EN**: Implements logic around `StringRef`, `new`, `MCSectionGOFF`, `getGOFFSection`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `StringRef`, `new`, `MCSectionGOFF`, `getGOFFSection`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 759-786
```cpp
}

MCSectionGOFF *MCContext::getGOFFSection(SectionKind Kind, StringRef Name,
                                         GOFF::PRAttr PRAttributes,
                                         MCSection *Parent) {
  return getGOFFSection<GOFF::PRAttr>(Kind, Name, PRAttributes, Parent,
                                      /*IsVirtual=*/false);
}

MCSectionCOFF *MCContext::getCOFFSection(StringRef Section,
                                         unsigned Characteristics,
                                         StringRef COMDATSymName, int Selection,
                                         unsigned UniqueID) {
  MCSymbol *COMDATSymbol = nullptr;
  if (!COMDATSymName.empty()) {
    COMDATSymbol = getOrCreateSymbol(COMDATSymName);
    assert(COMDATSymbol && "COMDATSymbol is null");
    COMDATSymName = COMDATSymbol->getName();
    // A non-associative COMDAT is considered to define the COMDAT symbol. Check
    // the redefinition error.
    if (Selection != COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE &&
        COMDATSymbol->isDefined() &&
        (!COMDATSymbol->isInSection() ||
         static_cast<const MCSectionCOFF &>(COMDATSymbol->getSection())
                 .getCOMDATSymbol() != COMDATSymbol))
      reportError(SMLoc(), "invalid symbol redefinition");
  }

```
- **EN**: Implements logic around `getGOFFSection`, `PRAttr>`, `getCOFFSection`, `getOrCreateSymbol`, and 7 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getGOFFSection`, `PRAttr>`, `getCOFFSection`, `getOrCreateSymbol`, and 7 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 787-807
```cpp
  // Do the lookup, if we have a hit, return it.
  COFFSectionKey T{Section, COMDATSymName, Selection, UniqueID};
  auto [Iter, Inserted] = COFFUniquingMap.try_emplace(T);
  if (!Inserted)
    return Iter->second;

  StringRef CachedName = Iter->first.SectionName;
  MCSymbol *Begin = getOrCreateSectionSymbol<MCSymbolCOFF>(Section);
  MCSectionCOFF *Result = new (COFFAllocator.Allocate()) MCSectionCOFF(
      CachedName, Characteristics, COMDATSymbol, Selection, UniqueID, Begin);
  Iter->second = Result;
  Begin->setFragment(&Result->getDummyFragment());
  return Result;
}

MCSectionCOFF *MCContext::getCOFFSection(StringRef Section,
                                         unsigned Characteristics) {
  return getCOFFSection(Section, Characteristics, "", 0,
                        MCSection::NonUniqueID);
}

```
- **EN**: Implements logic around `try_emplace`, `getOrCreateSectionSymbol<MCSymbolCOFF>`, `new`, `setFragment`, and 1 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `try_emplace`, `getOrCreateSectionSymbol<MCSymbolCOFF>`, `new`, `setFragment`, and 1 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 808-823
```cpp
MCSectionCOFF *MCContext::getAssociativeCOFFSection(MCSectionCOFF *Sec,
                                                    const MCSymbol *KeySym,
                                                    unsigned UniqueID) {
  // Return the normal section if we don't have to be associative or unique.
  if (!KeySym && UniqueID == MCSection::NonUniqueID)
    return Sec;

  // If we have a key symbol, make an associative section with the same name and
  // kind as the normal section.
  unsigned Characteristics = Sec->getCharacteristics();
  if (KeySym) {
    Characteristics |= COFF::IMAGE_SCN_LNK_COMDAT;
    return getCOFFSection(Sec->getName(), Characteristics, KeySym->getName(),
                          COFF::IMAGE_COMDAT_SELECT_ASSOCIATIVE, UniqueID);
  }

```
- **EN**: Implements logic around `getAssociativeCOFFSection`, `getCharacteristics`, `getCOFFSection`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getAssociativeCOFFSection`, `getCharacteristics`, `getCOFFSection` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 824-840
```cpp
  return getCOFFSection(Sec->getName(), Characteristics, "", 0, UniqueID);
}

MCSectionWasm *MCContext::getWasmSection(const Twine &Section, SectionKind K,
                                         unsigned Flags, const Twine &Group,
                                         unsigned UniqueID) {
  MCSymbolWasm *GroupSym = nullptr;
  if (!Group.isTriviallyEmpty() && !Group.str().empty()) {
    GroupSym = static_cast<MCSymbolWasm *>(getOrCreateSymbol(Group));
    GroupSym->setComdat(true);
    if (K.isMetadata() && !GroupSym->getType().has_value()) {
      // Comdat group symbol associated with a custom section is a section
      // symbol (not a data symbol).
      GroupSym->setType(wasm::WASM_SYMBOL_TYPE_SECTION);
    }
  }

```
- **EN**: Implements logic around `getCOFFSection`, `getWasmSection`, `getOrCreateSymbol`, `setComdat`, and 1 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getCOFFSection`, `getWasmSection`, `getOrCreateSymbol`, `setComdat`, and 1 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 841-857
```cpp
  return getWasmSection(Section, K, Flags, GroupSym, UniqueID);
}

MCSectionWasm *MCContext::getWasmSection(const Twine &Section, SectionKind Kind,
                                         unsigned Flags,
                                         const MCSymbolWasm *GroupSym,
                                         unsigned UniqueID) {
  StringRef Group = "";
  if (GroupSym)
    Group = GroupSym->getName();
  // Do the lookup, if we have a hit, return it.
  auto IterBool = WasmUniquingMap.insert(
      std::make_pair(WasmSectionKey{Section.str(), Group, UniqueID}, nullptr));
  auto &Entry = *IterBool.first;
  if (!IterBool.second)
    return Entry.second;

```
- **EN**: Implements logic around `getWasmSection`, `getName`, `insert`, `make_pair`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getWasmSection`, `getName`, `insert`, `make_pair` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 858-877
```cpp
  StringRef CachedName = Entry.first.SectionName;

  MCSymbol *Begin = createRenamableSymbol(CachedName, true, false);
  // Begin always has a different name than CachedName... see #48596.
  getSymbolTableEntry(Begin->getName()).second.Symbol = Begin;
  static_cast<MCSymbolWasm *>(Begin)->setType(wasm::WASM_SYMBOL_TYPE_SECTION);

  MCSectionWasm *Result = new (WasmAllocator.Allocate())
      MCSectionWasm(CachedName, Kind, Flags, GroupSym, UniqueID, Begin);
  Entry.second = Result;

  return Result;
}

bool MCContext::hasXCOFFSection(StringRef Section,
                                XCOFF::CsectProperties CsectProp) const {
  return XCOFFUniquingMap.count(
             XCOFFSectionKey(Section.str(), CsectProp.MappingClass)) != 0;
}

```
- **EN**: Implements logic around `createRenamableSymbol`, `getSymbolTableEntry`, `setType`, `new`, and 4 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `createRenamableSymbol`, `getSymbolTableEntry`, `setType`, `new`, and 4 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 878-895
```cpp
MCSectionXCOFF *MCContext::getXCOFFSection(
    StringRef Section, SectionKind Kind,
    std::optional<XCOFF::CsectProperties> CsectProp, bool MultiSymbolsAllowed,
    std::optional<XCOFF::DwarfSectionSubtypeFlags> DwarfSectionSubtypeFlags) {
  bool IsDwarfSec = DwarfSectionSubtypeFlags.has_value();
  assert((IsDwarfSec != CsectProp.has_value()) && "Invalid XCOFF section!");

  // Do the lookup. If we have a hit, return it.
  auto IterBool = XCOFFUniquingMap.insert(std::make_pair(
      IsDwarfSec ? XCOFFSectionKey(Section.str(), *DwarfSectionSubtypeFlags)
                 : XCOFFSectionKey(Section.str(), CsectProp->MappingClass),
      nullptr));
  auto &Entry = *IterBool.first;
  if (!IterBool.second) {
    MCSectionXCOFF *ExistedEntry = Entry.second;
    if (ExistedEntry->isMultiSymbolsAllowed() != MultiSymbolsAllowed)
      report_fatal_error("section's multiply symbols policy does not match");

```
- **EN**: Implements logic around `getXCOFFSection`, `has_value`, `assert`, `insert`, and 2 more symbols; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getXCOFFSection`, `has_value`, `assert`, `insert`, and 2 more symbols 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 896-922
```cpp
    return ExistedEntry;
  }

  // Otherwise, return a new section.
  StringRef CachedName = Entry.first.SectionName;
  MCSymbolXCOFF *QualName = nullptr;
  // Debug section don't have storage class attribute.
  if (IsDwarfSec)
    QualName = static_cast<MCSymbolXCOFF *>(getOrCreateSymbol(CachedName));
  else
    QualName = static_cast<MCSymbolXCOFF *>(getOrCreateSymbol(
        CachedName + "[" +
        XCOFF::getMappingClassString(CsectProp->MappingClass) + "]"));

  // QualName->getUnqualifiedName() and CachedName are the same except when
  // CachedName contains invalid character(s) such as '$' for an XCOFF symbol.
  MCSectionXCOFF *Result = nullptr;
  if (IsDwarfSec)
    Result = new (XCOFFAllocator.Allocate()) MCSectionXCOFF(
        QualName->getUnqualifiedName(), Kind, QualName,
        *DwarfSectionSubtypeFlags, QualName, CachedName, MultiSymbolsAllowed);
  else
    Result = new (XCOFFAllocator.Allocate())
        MCSectionXCOFF(QualName->getUnqualifiedName(), CsectProp->MappingClass,
                       CsectProp->Type, Kind, QualName, nullptr, CachedName,
                       MultiSymbolsAllowed);

```
- **EN**: Introduces declarations for `attribute`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `attribute` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 923-938
```cpp
  Entry.second = Result;
  return Result;
}

MCSectionSPIRV *MCContext::getSPIRVSection() {
  MCSectionSPIRV *Result = new (SPIRVAllocator.Allocate()) MCSectionSPIRV();
  return Result;
}

MCSectionDXContainer *MCContext::getDXContainerSection(StringRef Section,
                                                       SectionKind K) {
  // Do the lookup, if we have a hit, return it.
  auto ItInsertedPair = DXCUniquingMap.try_emplace(Section);
  if (!ItInsertedPair.second)
    return ItInsertedPair.first->second;

```
- **EN**: Implements logic around `getSPIRVSection`, `new`, `getDXContainerSection`, `try_emplace`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `getSPIRVSection`, `new`, `getDXContainerSection`, `try_emplace` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 939-954
```cpp
  auto MapIt = ItInsertedPair.first;
  // Grab the name from the StringMap. Since the Section is going to keep a
  // copy of this StringRef we need to make sure the underlying string stays
  // alive as long as we need it.
  StringRef Name = MapIt->first();
  MapIt->second =
      new (DXCAllocator.Allocate()) MCSectionDXContainer(Name, K, nullptr);

  // The first fragment will store the header
  return MapIt->second;
}

MCSubtargetInfo &MCContext::getSubtargetCopy(const MCSubtargetInfo &STI) {
  return *new (MCSubtargetAllocator.Allocate()) MCSubtargetInfo(STI);
}

```
- **EN**: Implements logic around `first`, `new`, `getSubtargetCopy`; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `first`, `new`, `getSubtargetCopy` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 955-970
```cpp
void MCContext::addDebugPrefixMapEntry(const std::string &From,
                                       const std::string &To) {
  DebugPrefixMap.emplace_back(From, To);
}

void MCContext::remapDebugPath(SmallVectorImpl<char> &Path) {
  for (const auto &[From, To] : llvm::reverse(DebugPrefixMap))
    if (llvm::sys::path::replace_path_prefix(Path, From, To))
      break;
}

void MCContext::RemapDebugPaths() {
  const auto &DebugPrefixMap = this->DebugPrefixMap;
  if (DebugPrefixMap.empty())
    return;

```
- **EN**: Implements logic around `addDebugPrefixMapEntry`, `emplace_back`, `remapDebugPath`, `RemapDebugPaths`.
- **CN**: 围绕 `addDebugPrefixMapEntry`, `emplace_back`, `remapDebugPath`, `RemapDebugPaths` 实现具体逻辑。

### Lines 971-990
```cpp
  // Remap compilation directory.
  remapDebugPath(CompilationDir);

  // Remap MCDwarfDirs and RootFile.Name in all compilation units.
  SmallString<256> P;
  for (auto &CUIDTablePair : MCDwarfLineTablesCUMap) {
    for (auto &Dir : CUIDTablePair.second.getMCDwarfDirs()) {
      P = Dir;
      remapDebugPath(P);
      Dir = std::string(P);
    }

    // Used by DW_TAG_compile_unit's DT_AT_name and DW_TAG_label's
    // DW_AT_decl_file for DWARF v5 generated for assembly source.
    P = CUIDTablePair.second.getRootFile().Name;
    remapDebugPath(P);
    CUIDTablePair.second.getRootFile().Name = std::string(P);
  }
}

```
- **EN**: Implements logic around `remapDebugPath`, `string`, `getRootFile`.
- **CN**: 围绕 `remapDebugPath`, `string`, `getRootFile` 实现具体逻辑。

### Lines 991-1022
```cpp
//===----------------------------------------------------------------------===//
// Dwarf Management
//===----------------------------------------------------------------------===//

EmitDwarfUnwindType MCContext::emitDwarfUnwindInfo() const {
  return getTargetOptions().EmitDwarfUnwind;
}

bool MCContext::emitCompactUnwindNonCanonical() const {
  return getTargetOptions().EmitCompactUnwindNonCanonical;
}

void MCContext::setGenDwarfRootFile(StringRef InputFileName, StringRef Buffer) {
  // MCDwarf needs the root file as well as the compilation directory.
  // If we find a '.file 0' directive that will supersede these values.
  std::optional<MD5::MD5Result> Cksum;
  if (getDwarfVersion() >= 5) {
    MD5 Hash;
    MD5::MD5Result Sum;
    Hash.update(Buffer);
    Hash.final(Sum);
    Cksum = Sum;
  }
  // Canonicalize the root filename. It cannot be empty, and should not
  // repeat the compilation dir.
  // The MCContext ctor initializes MainFileName to the name associated with
  // the SrcMgr's main file ID, which might be the same as InputFileName (and
  // possibly include directory components).
  // Or, MainFileName might have been overridden by a -main-file-name option,
  // which is supposed to be just a base filename with no directory component.
  // So, if the InputFileName and MainFileName are not equal, assume
  // MainFileName is a substitute basename and replace the last component.
```
- **EN**: Implements logic around `emitDwarfUnwindInfo`, `getTargetOptions`, `emitCompactUnwindNonCanonical`, `setGenDwarfRootFile`, and 2 more symbols; this block parses assembly syntax or operands; returns subsystem-specific computed results.
- **CN**: 围绕 `emitDwarfUnwindInfo`, `getTargetOptions`, `emitCompactUnwindNonCanonical`, `setGenDwarfRootFile`, and 2 more symbols 实现具体逻辑；这一段解析汇编语法或操作数，返回子系统相关的计算结果。

### Lines 1023-1038
```cpp
  SmallString<1024> FileNameBuf = InputFileName;
  if (FileNameBuf.empty() || FileNameBuf == "-")
    FileNameBuf = "<stdin>";
  if (!getMainFileName().empty() && FileNameBuf != getMainFileName()) {
    llvm::sys::path::remove_filename(FileNameBuf);
    llvm::sys::path::append(FileNameBuf, getMainFileName());
  }
  StringRef FileName = FileNameBuf;
  if (FileName.consume_front(getCompilationDir()))
    if (llvm::sys::path::is_separator(FileName.front()))
      FileName = FileName.drop_front();
  assert(!FileName.empty());
  setMCLineTableRootFile(
      /*CUID=*/0, getCompilationDir(), FileName, Cksum, std::nullopt);
}

```
- **EN**: Implements logic around `remove_filename`, `append`, `drop_front`, `assert`, and 2 more symbols.
- **CN**: 围绕 `remove_filename`, `append`, `drop_front`, `assert`, and 2 more symbols 实现具体逻辑。

### Lines 1039-1054
```cpp
/// getDwarfFile - takes a file name and number to place in the dwarf file and
/// directory tables.  If the file number has already been allocated it is an
/// error and zero is returned and the client reports the error, else the
/// allocated file number is returned.  The file numbers may be in any order.
Expected<unsigned>
MCContext::getDwarfFile(StringRef Directory, StringRef FileName,
                        unsigned FileNumber,
                        std::optional<MD5::MD5Result> Checksum,
                        std::optional<StringRef> Source, unsigned CUID) {
  MCDwarfLineTable &Table = MCDwarfLineTablesCUMap[CUID];
  return Table.tryGetFile(Directory, FileName, Checksum, Source, DwarfVersion,
                          FileNumber);
}

/// isValidDwarfFileNumber - takes a dwarf file number and returns true if it
/// currently is assigned and false otherwise.
```
- **EN**: Implements logic around `getDwarfFile`, `tryGetFile`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getDwarfFile`, `tryGetFile` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1055-1071
```cpp
bool MCContext::isValidDwarfFileNumber(unsigned FileNumber, unsigned CUID) {
  const MCDwarfLineTable &LineTable = getMCDwarfLineTable(CUID);
  if (FileNumber == 0)
    return getDwarfVersion() >= 5;
  if (FileNumber >= LineTable.getMCDwarfFiles().size())
    return false;

  return !LineTable.getMCDwarfFiles()[FileNumber].Name.empty();
}

/// Remove empty sections from SectionsForRanges, to avoid generating
/// useless debug info for them.
void MCContext::finalizeDwarfSections(MCStreamer &MCOS) {
  SectionsForRanges.remove_if(
      [&](MCSection *Sec) { return !MCOS.mayHaveInstructions(*Sec); });
}

```
- **EN**: Implements logic around `isValidDwarfFileNumber`, `getMCDwarfLineTable`, `getDwarfVersion`, `getMCDwarfFiles`, and 3 more symbols; this block drives emission, layout, or binary encoding behavior; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `isValidDwarfFileNumber`, `getMCDwarfLineTable`, `getDwarfVersion`, `getMCDwarfFiles`, and 3 more symbols 实现具体逻辑；这一段驱动输出、布局或二进制编码行为，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 1072-1095
```cpp
CodeViewContext &MCContext::getCVContext() {
  if (!CVContext)
    CVContext.reset(new CodeViewContext(this));
  return *CVContext;
}

//===----------------------------------------------------------------------===//
// Error Reporting
//===----------------------------------------------------------------------===//

void MCContext::diagnose(const SMDiagnostic &SMD) {
  assert(DiagHandler && "MCContext::DiagHandler is not set");
  bool UseInlineSrcMgr = false;
  const SourceMgr *SMP = nullptr;
  if (SrcMgr) {
    SMP = SrcMgr;
  } else if (InlineSrcMgr) {
    SMP = InlineSrcMgr.get();
    UseInlineSrcMgr = true;
  } else
    llvm_unreachable("Either SourceMgr should be available");
  DiagHandler(SMD, UseInlineSrcMgr, *SMP, LocInfos);
}

```
- **EN**: Implements logic around `getCVContext`, `reset`, `diagnose`, `assert`, and 3 more symbols; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getCVContext`, `reset`, `diagnose`, `assert`, and 3 more symbols 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 1096-1121
```cpp
void MCContext::reportCommon(
    SMLoc Loc,
    std::function<void(SMDiagnostic &, const SourceMgr *)> GetMessage) {
  // * MCContext::SrcMgr is null when the MC layer emits machine code for input
  //   other than assembly file, say, for .c/.cpp/.ll/.bc.
  // * MCContext::InlineSrcMgr is null when the inline asm is not used.
  // * A default SourceMgr is needed for diagnosing when both MCContext::SrcMgr
  //   and MCContext::InlineSrcMgr are null.
  SourceMgr SM;
  const SourceMgr *SMP = &SM;
  bool UseInlineSrcMgr = false;

  // FIXME: Simplify these by combining InlineSrcMgr & SrcMgr.
  //        For MC-only execution, only SrcMgr is used;
  //        For non MC-only execution, InlineSrcMgr is only ctor'd if there is
  //        inline asm in the IR.
  if (Loc.isValid()) {
    if (SrcMgr) {
      SMP = SrcMgr;
    } else if (InlineSrcMgr) {
      SMP = InlineSrcMgr.get();
      UseInlineSrcMgr = true;
    } else
      llvm_unreachable("Either SourceMgr should be available");
  }

```
- **EN**: Implements logic around `reportCommon`, `function<void`, `get`, `llvm_unreachable`.
- **CN**: 围绕 `reportCommon`, `function<void`, `get`, `llvm_unreachable` 实现具体逻辑。

### Lines 1122-1144
```cpp
  SMDiagnostic D;
  GetMessage(D, SMP);
  DiagHandler(D, UseInlineSrcMgr, *SMP, LocInfos);
}

void MCContext::reportError(SMLoc Loc, const Twine &Msg) {
  HadError = true;
  reportCommon(Loc, [&](SMDiagnostic &D, const SourceMgr *SMP) {
    D = SMP->GetMessage(Loc, SourceMgr::DK_Error, Msg);
  });
}

void MCContext::reportWarning(SMLoc Loc, const Twine &Msg) {
  if (getTargetOptions().MCNoWarn)
    return;
  if (getTargetOptions().MCFatalWarnings) {
    reportError(Loc, Msg);
  } else {
    reportCommon(Loc, [&](SMDiagnostic &D, const SourceMgr *SMP) {
      D = SMP->GetMessage(Loc, SourceMgr::DK_Warning, Msg);
    });
  }
}
```
- **EN**: Implements logic around `GetMessage`, `DiagHandler`, `reportError`, `reportCommon`, and 1 more symbols.
- **CN**: 围绕 `GetMessage`, `DiagHandler`, `reportError`, `reportCommon`, and 1 more symbols 实现具体逻辑。

## Key Concepts / 关键概念

- **MC global state / MC 全局状态**:
  - **EN**: Stores sections, symbols, source-loc information, and format-specific bookkeeping
  - **CN**: 存储节区、符号、源码位置信息以及格式相关的簿记状态
- **Directive handling / 伪指令处理**:
  - **EN**: Interprets assembler directives and maps them to streamer operations
  - **CN**: 解释汇编伪指令并把它们映射为 streamer 操作
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCContext.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/COFF.h`, `llvm/BinaryFormat/ELF.h`, `llvm/BinaryFormat/GOFF.h`, `llvm/BinaryFormat/Wasm.h`, `llvm/BinaryFormat/XCOFF.h`, `llvm/MC/MCAsmInfo.h` ... (+36 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support, BinaryFormat
