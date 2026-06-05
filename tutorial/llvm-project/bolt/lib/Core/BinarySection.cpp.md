# BinarySection.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/BinarySection.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Section in a binary file. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Section in a binary file。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/BinarySection.cpp - Section in a binary file -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the BinarySection class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-21

```cpp
#include "bolt/Core/BinarySection.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "bolt/Utils/Utils.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "bolt"
```

- EN: Pulls in 6 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 6 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 22-29

```cpp
using namespace llvm;
using namespace bolt;

namespace opts {
extern cl::opt<bool> HotData;
extern cl::opt<bool> PrintRelocations;
} // namespace opts
```

- EN: Works inside namespace scope `llvm`, `bolt`, `opts` to organize symbols. Notable symbols here include `llvm`, `bolt`, `opts`.
- CN: 这里位于命名空间 `llvm`, `bolt`, `opts` 中，用于组织符号作用域。这里较值得关注的符号包括 `llvm`, `bolt`, `opts`。

### Lines 30-42

```cpp
uint64_t BinarySection::Count = 0;

bool BinarySection::isELF() const { return BC.isELF(); }

bool BinarySection::isMachO() const { return BC.isMachO(); }

uint64_t
BinarySection::hash(const BinaryData &BD,
                    std::map<const BinaryData *, uint64_t> &Cache) const {
  auto Itr = Cache.find(&BD);
  if (Itr != Cache.end())
    return Itr->second;
```

- EN: Declares or implements routines including `isELF`, `isMachO`. Notable symbols here include `isELF`, `isMachO`.
- CN: 这里声明或实现函数，例如 `isELF`, `isMachO`。这里较值得关注的符号包括 `isELF`, `isMachO`。

### Lines 43-50

```cpp
  hash_code Hash =
      hash_combine(hash_value(BD.getSize()), hash_value(BD.getSectionName()));

  Cache[&BD] = Hash;

  if (!containsRange(BD.getAddress(), BD.getSize()))
    return Hash;
```

- EN: Declares or implements routines including `hash_combine`. Notable symbols here include `hash_combine`.
- CN: 这里声明或实现函数，例如 `hash_combine`。这里较值得关注的符号包括 `hash_combine`。

### Lines 51-65

```cpp
  uint64_t Offset = BD.getAddress() - getAddress();
  const uint64_t EndOffset = BD.getEndAddress() - getAddress();
  auto Begin = Relocations.lower_bound(Relocation{Offset, 0, 0, 0, 0});
  auto End = Relocations.upper_bound(Relocation{EndOffset, 0, 0, 0, 0});
  const StringRef Contents = getContents();

  while (Begin != End) {
    const Relocation &Rel = *Begin++;
    Hash = hash_combine(
        Hash, hash_value(Contents.substr(Offset, Begin->Offset - Offset)));
    if (BinaryData *RelBD = BC.getBinaryDataByName(Rel.Symbol->getName()))
      Hash = hash_combine(Hash, hash(*RelBD, Cache));
    Offset = Rel.Offset + Rel.getSize();
  }
```

- EN: Declares or implements routines including `getContents`, `hash_value`, `hash_combine`. Notable symbols here include `getContents`, `hash_value`, `hash_combine`.
- CN: 这里声明或实现函数，例如 `getContents`, `hash_value`, `hash_combine`。这里较值得关注的符号包括 `getContents`, `hash_value`, `hash_combine`。

### Lines 66-73

```cpp
  Hash = hash_combine(Hash,
                      hash_value(Contents.substr(Offset, EndOffset - Offset)));

  Cache[&BD] = Hash;

  return Hash;
}
```

- EN: Declares or implements routines including `hash_value`. Notable symbols here include `hash_value`.
- CN: 这里声明或实现函数，例如 `hash_value`。这里较值得关注的符号包括 `hash_value`。

### Lines 74-83

```cpp
void BinarySection::emitAsData(MCStreamer &Streamer,
                               const Twine &SectionName) const {
  StringRef SectionContents =
      isFinalized() ? getOutputContents() : getContents();
  MCSectionELF *ELFSection =
      BC.Ctx->getELFSection(SectionName, getELFType(), getELFFlags());

  Streamer.switchSection(ELFSection);
  Streamer.emitValueToAlignment(getAlign());
```

- EN: Declares or implements routines including `isFinalized`, `getELFSection`. Notable symbols here include `isFinalized`, `getELFSection`.
- CN: 这里声明或实现函数，例如 `isFinalized`, `getELFSection`。这里较值得关注的符号包括 `isFinalized`, `getELFSection`。

### Lines 84-98

```cpp
  if (BC.HasRelocations && opts::HotData && isReordered())
    Streamer.emitLabel(BC.Ctx->getOrCreateSymbol("__hot_data_start"));

  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: emitting "
                    << (isAllocatable() ? "" : "non-")
                    << "allocatable data section " << SectionName << '\n');

  if (!hasRelocations()) {
    Streamer.emitBytes(SectionContents);
  } else {
    uint64_t SectionOffset = 0;
    for (auto RI = Relocations.begin(), RE = Relocations.end(); RI != RE;) {
      auto RelocationOffset = RI->Offset;
      assert(RelocationOffset < SectionContents.size() && "overflow detected");
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `assert`. Notable symbols here include `LLVM_DEBUG`, `assert`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `assert`。这里较值得关注的符号包括 `LLVM_DEBUG`, `assert`。

### Lines 99-110

```cpp
      if (SectionOffset < RelocationOffset) {
        Streamer.emitBytes(SectionContents.substr(
            SectionOffset, RelocationOffset - SectionOffset));
        SectionOffset = RelocationOffset;
      }

      // Get iterators to all relocations with the same offset. Usually, there
      // is only one such relocation but there can be more for composed
      // relocations.
      auto ROI = RI;
      auto ROE = Relocations.upper_bound(RelocationOffset);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 111-120

```cpp
      // Start from the next offset on the next iteration.
      RI = ROE;

      // Skip undefined symbols.
      auto HasUndefSym = [](const auto &Relocation) {
        return Relocation.Symbol && Relocation.Symbol->isTemporary() &&
               Relocation.Symbol->isUndefined() &&
               !Relocation.Symbol->isRegistered();
      };
```

- EN: Declares or implements routines including `isUndefined`, `isRegistered`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isUndefined`, `isRegistered`.
- CN: 这里声明或实现函数，例如 `isUndefined`, `isRegistered`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isUndefined`, `isRegistered`。

### Lines 121-135

```cpp
      if (std::any_of(ROI, ROE, HasUndefSym))
        continue;

#ifndef NDEBUG
      for (const auto &Relocation : make_range(ROI, ROE)) {
        LLVM_DEBUG(
            dbgs() << "BOLT-DEBUG: emitting relocation for symbol "
                   << (Relocation.Symbol ? Relocation.Symbol->getName()
                                         : StringRef("<none>"))
                   << " at offset 0x" << Twine::utohexstr(Relocation.Offset)
                   << " with size "
                   << Relocation::getSizeForType(Relocation.Type) << '\n');
      }
#endif
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Declares or implements routines including `dbgs`, `StringRef`, `utohexstr`, `getSizeForType`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`, `StringRef`, `utohexstr`, `getSizeForType`.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里声明或实现函数，例如 `dbgs`, `StringRef`, `utohexstr`, `getSizeForType`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`, `StringRef`, `utohexstr`, `getSizeForType`。

### Lines 136-143

```cpp
      size_t RelocationSize = Relocation::emit(ROI, ROE, &Streamer);
      SectionOffset += RelocationSize;
    }
    assert(SectionOffset <= SectionContents.size() && "overflow error");
    if (SectionOffset < SectionContents.size())
      Streamer.emitBytes(SectionContents.substr(SectionOffset));
  }
```

- EN: Declares or implements routines including `emit`, `assert`. Notable symbols here include `emit`, `assert`.
- CN: 这里声明或实现函数，例如 `emit`, `assert`。这里较值得关注的符号包括 `emit`, `assert`。

### Lines 144-156

```cpp
  if (BC.HasRelocations && opts::HotData && isReordered())
    Streamer.emitLabel(BC.Ctx->getOrCreateSymbol("__hot_data_end"));
}

uint64_t BinarySection::write(raw_ostream &OS) const {
  const uint64_t NumValidContentBytes =
      std::min<uint64_t>(getOutputContents().size(), getOutputSize());
  OS.write(getOutputContents().data(), NumValidContentBytes);
  if (getOutputSize() > NumValidContentBytes)
    OS.write_zeros(getOutputSize() - NumValidContentBytes);
  return getOutputSize();
}
```

- EN: Declares or implements routines including `write`. Notable symbols here include `write`.
- CN: 这里声明或实现函数，例如 `write`。这里较值得关注的符号包括 `write`。

### Lines 157-174

```cpp
void BinarySection::flushPendingRelocations(raw_pwrite_stream &OS,
                                            SymbolResolverFuncTy Resolver) {
  if (PendingRelocations.empty() && Patches.empty())
    return;

  const uint64_t SectionAddress = getAddress();

  // We apply relocations to original section contents. For allocatable sections
  // this means using their input file offsets, since the output file offset
  // could change (e.g. for new instance of .text). For non-allocatable
  // sections, the output offset should always be a valid one.
  const uint64_t SectionFileOffset =
      isAllocatable() ? getInputFileOffset() : getOutputFileOffset();
  LLVM_DEBUG(
      dbgs() << "BOLT-DEBUG: flushing pending relocations for section "
             << getName() << '\n'
             << "  address: 0x" << Twine::utohexstr(SectionAddress) << '\n'
             << "  offset: 0x" << Twine::utohexstr(SectionFileOffset) << '\n');
```

- EN: Declares or implements routines including `getAddress`, `isAllocatable`, `dbgs`, `getName`, `utohexstr`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getAddress`, `isAllocatable`, `dbgs`, `getName`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `getAddress`, `isAllocatable`, `dbgs`, `getName`, `utohexstr`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getAddress`, `isAllocatable`, `dbgs`, `getName`, `utohexstr`。

### Lines 175-185

```cpp

  for (BinaryPatch &Patch : Patches)
    OS.pwrite(Patch.Bytes.data(), Patch.Bytes.size(),
              SectionFileOffset + Patch.Offset);

  uint64_t SkippedPendingRelocations = 0;
  for (Relocation &Reloc : PendingRelocations) {
    uint64_t Value = Reloc.Addend;
    if (Reloc.Symbol)
      Value += Resolver(Reloc.Symbol);
```

- EN: Declares or implements routines including `Resolver`. Notable symbols here include `Resolver`.
- CN: 这里声明或实现函数，例如 `Resolver`。这里较值得关注的符号包括 `Resolver`。

### Lines 186-196

```cpp
    // Safely skip any optional pending relocation that cannot be encoded.
    if (Reloc.isOptional() &&
        !Relocation::canEncodeValue(Reloc.Type, Value,
                                    SectionAddress + Reloc.Offset)) {

      ++SkippedPendingRelocations;
      continue;
    }
    Value = Relocation::encodeValue(Reloc.Type, Value,
                                    SectionAddress + Reloc.Offset);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 197-210

```cpp
    OS.pwrite(reinterpret_cast<const char *>(&Value),
              Relocation::getSizeForType(Reloc.Type),
              SectionFileOffset + Reloc.Offset);

    LLVM_DEBUG(
        dbgs() << "BOLT-DEBUG: writing value 0x" << Twine::utohexstr(Value)
               << " of size " << Relocation::getSizeForType(Reloc.Type)
               << " at section offset 0x" << Twine::utohexstr(Reloc.Offset)
               << " address 0x"
               << Twine::utohexstr(SectionAddress + Reloc.Offset)
               << " file offset 0x"
               << Twine::utohexstr(SectionFileOffset + Reloc.Offset) << '\n';);
  }
```

- EN: Declares or implements routines including `getSizeForType`, `dbgs`, `utohexstr`. Notable symbols here include `getSizeForType`, `dbgs`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `getSizeForType`, `dbgs`, `utohexstr`。这里较值得关注的符号包括 `getSizeForType`, `dbgs`, `utohexstr`。

### Lines 211-218

```cpp
  clearList(PendingRelocations);

  if (SkippedPendingRelocations > 0 && opts::Verbosity >= 1) {
    BC.outs() << "BOLT-INFO: skipped " << SkippedPendingRelocations
              << " out-of-range optional relocations\n";
  }
}
```

- EN: Declares or implements routines including `clearList`. Notable symbols here include `clearList`.
- CN: 这里声明或实现函数，例如 `clearList`。这里较值得关注的符号包括 `clearList`。

### Lines 219-228

```cpp
BinarySection::~BinarySection() { updateContents(nullptr, 0); }

void BinarySection::clearRelocations() { clearList(Relocations); }

void BinarySection::print(raw_ostream &OS) const {
  OS << getName() << ", "
     << "0x" << Twine::utohexstr(getAddress()) << ", " << getSize() << " (0x"
     << Twine::utohexstr(getOutputAddress()) << ", " << getOutputSize() << ")"
     << ", data = " << getData() << ", output data = " << getOutputData();
```

- EN: Declares or implements routines including `BinarySection`, `clearRelocations`, `print`, `getName`, `utohexstr`, and 1 more. Notable symbols here include `BinarySection`, `clearRelocations`, `print`, `getName`, `utohexstr`, `getData`.
- CN: 这里声明或实现函数，例如 `BinarySection`, `clearRelocations`, `print`, `getName`, `utohexstr`, and 1 more。这里较值得关注的符号包括 `BinarySection`, `clearRelocations`, `print`, `getName`, `utohexstr`, `getData`。

### Lines 229-237

```cpp
  if (isAllocatable())
    OS << " (allocatable)";

  if (isVirtual())
    OS << " (virtual)";

  if (isTLS())
    OS << " (tls)";
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 238-253

```cpp
  if (opts::PrintRelocations)
    for (const Relocation &R : relocations())
      OS << "\n  " << R;
}

BinarySection::RelocationSetType
BinarySection::reorderRelocations(bool Inplace) const {
  assert(PendingRelocations.empty() &&
         "reordering pending relocations not supported");
  RelocationSetType NewRelocations;
  for (const Relocation &Rel : relocations()) {
    uint64_t RelAddr = Rel.Offset + getAddress();
    BinaryData *BD = BC.getBinaryDataContainingAddress(RelAddr);
    BD = BD->getAtomicRoot();
    assert(BD);
```

- EN: Declares or implements routines including `reorderRelocations`, `assert`, `getAddress`, `getAtomicRoot`. Notable symbols here include `reorderRelocations`, `assert`, `getAddress`, `getAtomicRoot`.
- CN: 这里声明或实现函数，例如 `reorderRelocations`, `assert`, `getAddress`, `getAtomicRoot`。这里较值得关注的符号包括 `reorderRelocations`, `assert`, `getAddress`, `getAtomicRoot`。

### Lines 254-267

```cpp
    if ((!BD->isMoved() && !Inplace) || BD->isJumpTable())
      continue;

    Relocation NewRel(Rel);
    uint64_t RelOffset = RelAddr - BD->getAddress();
    NewRel.Offset = BD->getOutputOffset() + RelOffset;
    assert(NewRel.Offset < getSize());
    LLVM_DEBUG(dbgs() << "BOLT-DEBUG: moving " << Rel << " -> " << NewRel
                      << "\n");
    NewRelocations.emplace(std::move(NewRel));
  }
  return NewRelocations;
}
```

- EN: Declares or implements routines including `NewRel`, `getAddress`, `getOutputOffset`, `assert`, `LLVM_DEBUG`. Notable symbols here include `NewRel`, `getAddress`, `getOutputOffset`, `assert`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `NewRel`, `getAddress`, `getOutputOffset`, `assert`, `LLVM_DEBUG`。这里较值得关注的符号包括 `NewRel`, `getAddress`, `getOutputOffset`, `assert`, `LLVM_DEBUG`。

### Lines 268-285

```cpp
void BinarySection::reorderContents(const std::vector<BinaryData *> &Order,
                                    bool Inplace) {
  IsReordered = true;

  Relocations = reorderRelocations(Inplace);

  std::string Str;
  raw_string_ostream OS(Str);
  const char *Src = Contents.data();
  LLVM_DEBUG(dbgs() << "BOLT-DEBUG: reorderContents for " << Name << "\n");
  for (BinaryData *BD : Order) {
    assert((BD->isMoved() || !Inplace) && !BD->isJumpTable());
    assert(BD->isAtomic() && BD->isMoveable());
    const uint64_t SrcOffset = BD->getAddress() - getAddress();
    assert(SrcOffset < Contents.size());
    assert(SrcOffset == BD->getOffset());
    while (OS.tell() < BD->getOutputOffset())
      OS.write((unsigned char)0);
```

- EN: Declares or implements routines including `reorderRelocations`, `OS`, `LLVM_DEBUG`, `assert`, `getAddress`. Notable symbols here include `reorderRelocations`, `OS`, `LLVM_DEBUG`, `assert`, `getAddress`.
- CN: 这里声明或实现函数，例如 `reorderRelocations`, `OS`, `LLVM_DEBUG`, `assert`, `getAddress`。这里较值得关注的符号包括 `reorderRelocations`, `OS`, `LLVM_DEBUG`, `assert`, `getAddress`。

### Lines 286-296

```cpp
    LLVM_DEBUG(dbgs() << "BOLT-DEBUG: " << BD->getName() << " @ " << OS.tell()
                      << "\n");
    OS.write(&Src[SrcOffset], BD->getOutputSize());
  }
  if (Relocations.empty()) {
    // If there are no existing relocations, tack a phony one at the end
    // of the reordered segment to force LLVM to recognize and map this
    // section.
    MCSymbol *ZeroSym = BC.registerNameAtAddress("Zero", 0, 0, 0);
    addRelocation(OS.tell(), ZeroSym, Relocation::getAbs64(), 0xdeadbeef);
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `addRelocation`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`, `addRelocation`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `addRelocation`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`, `addRelocation`。

### Lines 297-304

```cpp
    uint64_t Zero = 0;
    OS.write(reinterpret_cast<const char *>(&Zero), sizeof(Zero));
  }
  auto *NewData = reinterpret_cast<char *>(copyByteArray(OS.str()));
  Contents = OutputContents = StringRef(NewData, OS.str().size());
  OutputSize = Contents.size();
}
```

- EN: Declares or implements routines including `StringRef`. Notable symbols here include `StringRef`.
- CN: 这里声明或实现函数，例如 `StringRef`。这里较值得关注的符号包括 `StringRef`。

### Lines 305-321

```cpp
std::string BinarySection::encodeELFNote(StringRef NameStr, StringRef DescStr,
                                         uint32_t Type) {
  std::string Str;
  raw_string_ostream OS(Str);
  const uint32_t NameSz = NameStr.size() + 1;
  const uint32_t DescSz = DescStr.size();
  OS.write(reinterpret_cast<const char *>(&(NameSz)), 4);
  OS.write(reinterpret_cast<const char *>(&(DescSz)), 4);
  OS.write(reinterpret_cast<const char *>(&(Type)), 4);
  OS << NameStr << '\0';
  for (uint64_t I = NameSz; I < alignTo(NameSz, 4); ++I)
    OS << '\0';
  OS << DescStr;
  for (uint64_t I = DescStr.size(); I < alignTo(DescStr.size(), 4); ++I)
    OS << '\0';
  return OS.str();
}
```

- EN: Declares or implements routines including `OS`. Notable symbols here include `OS`.
- CN: 这里声明或实现函数，例如 `OS`。这里较值得关注的符号包括 `OS`。

## Key Concepts / 关键概念

- `isELF`: function or method entry point / 函数或方法入口
- `isMachO`: function or method entry point / 函数或方法入口
- `hash_combine`: function or method entry point / 函数或方法入口
- `getContents`: function or method entry point / 函数或方法入口
- `hash_value`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/BinarySection.h`, `bolt/Core/BinaryContext.h`, `bolt/Utils/CommandLineOpts.h`, `bolt/Utils/Utils.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCStreamer.h`, `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
