# MachOObject.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/ObjCopy/MachO/MachOObject.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements Mach-O-specific object rewriting for llvm-objcopy.
  - **CN**: 实现 llvm-objcopy 的 Mach-O 专用目标文件改写逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MachOObject.cpp - Mach-O object file model ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp

#include "MachOObject.h"
#include "llvm/ADT/SmallPtrSet.h"

using namespace llvm;
using namespace llvm::objcopy::macho;

```
- **EN**: Pulls in the headers needed by this translation unit, including `MachOObject.h`, `llvm/ADT/SmallPtrSet.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `MachOObject.h`, `llvm/ADT/SmallPtrSet.h`。

### Lines 15-23
```cpp
Section::Section(StringRef SegName, StringRef SectName)
    : Segname(SegName), Sectname(SectName),
      CanonicalName((Twine(SegName) + Twine(',') + SectName).str()) {}

Section::Section(StringRef SegName, StringRef SectName, StringRef Content)
    : Segname(SegName), Sectname(SectName),
      CanonicalName((Twine(SegName) + Twine(',') + SectName).str()),
      Content(Content) {}

```
- **EN**: Implements logic around `Section`, `Segname`, `CanonicalName`, `Content`.
- **CN**: 围绕 `Section`, `Segname`, `CanonicalName`, `Content` 实现具体逻辑。

### Lines 24-33
```cpp
const SymbolEntry *SymbolTable::getSymbolByIndex(uint32_t Index) const {
  assert(Index < Symbols.size() && "invalid symbol index");
  return Symbols[Index].get();
}

SymbolEntry *SymbolTable::getSymbolByIndex(uint32_t Index) {
  return const_cast<SymbolEntry *>(
      static_cast<const SymbolTable *>(this)->getSymbolByIndex(Index));
}

```
- **EN**: Implements logic around `getSymbolByIndex`, `assert`, `get`.
- **CN**: 围绕 `getSymbolByIndex`, `assert`, `get` 实现具体逻辑。

### Lines 34-46
```cpp
void SymbolTable::updateSymbols(function_ref<void(SymbolEntry &)> Callable) {
  for (auto &Sym : Symbols)
    Callable(*Sym);

  // Partition symbols: local < defined external < undefined external.
  auto ExternalBegin = std::stable_partition(
      std::begin(Symbols), std::end(Symbols),
      [](const auto &Sym) { return Sym->isLocalSymbol(); });
  std::stable_partition(ExternalBegin, std::end(Symbols), [](const auto &Sym) {
    return !Sym->isUndefinedSymbol();
  });
}

```
- **EN**: Implements logic around `updateSymbols`, `Callable`, `stable_partition`, `begin`, and 2 more symbols.
- **CN**: 围绕 `updateSymbols`, `Callable`, `stable_partition`, `begin`, and 2 more symbols 实现具体逻辑。

### Lines 47-60
```cpp
void SymbolTable::removeSymbols(
    function_ref<bool(const std::unique_ptr<SymbolEntry> &)> ToRemove) {
  llvm::erase_if(Symbols, ToRemove);
}

void Object::updateLoadCommandIndexes() {
  static constexpr char TextSegmentName[] = "__TEXT";
  // Update indices of special load commands
  for (size_t Index = 0, Size = LoadCommands.size(); Index < Size; ++Index) {
    LoadCommand &LC = LoadCommands[Index];
    switch (LC.MachOLoadCommand.load_command_data.cmd) {
    case MachO::LC_CODE_SIGNATURE:
      CodeSignatureCommandIndex = Index;
      break;
```
- **EN**: Implements logic around `removeSymbols`, `function_ref`, `erase_if`, `updateLoadCommandIndexes`, and 1 more symbols; this block uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `removeSymbols`, `function_ref`, `erase_if`, `updateLoadCommandIndexes`, and 1 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派，并应用目标文件格式专用规则。

### Lines 61-74
```cpp
    case MachO::LC_SEGMENT:
      if (StringRef(LC.MachOLoadCommand.segment_command_data.segname) ==
          TextSegmentName)
        TextSegmentCommandIndex = Index;
      break;
    case MachO::LC_SEGMENT_64:
      if (StringRef(LC.MachOLoadCommand.segment_command_64_data.segname) ==
          TextSegmentName)
        TextSegmentCommandIndex = Index;
      break;
    case MachO::LC_SYMTAB:
      SymTabCommandIndex = Index;
      break;
    case MachO::LC_DYSYMTAB:
```
- **EN**: Implements logic around `StringRef`; this block applies object-format-specific rules.
- **CN**: 围绕 `StringRef` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 75-88
```cpp
      DySymTabCommandIndex = Index;
      break;
    case MachO::LC_DYLD_INFO:
    case MachO::LC_DYLD_INFO_ONLY:
      DyLdInfoCommandIndex = Index;
      break;
    case MachO::LC_DATA_IN_CODE:
      DataInCodeCommandIndex = Index;
      break;
    case MachO::LC_LINKER_OPTIMIZATION_HINT:
      LinkerOptimizationHintCommandIndex = Index;
      break;
    case MachO::LC_FUNCTION_STARTS:
      FunctionStartsCommandIndex = Index;
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 89-102
```cpp
      break;
    case MachO::LC_DYLIB_CODE_SIGN_DRS:
      DylibCodeSignDRsIndex = Index;
      break;
    case MachO::LC_DYLD_CHAINED_FIXUPS:
      ChainedFixupsCommandIndex = Index;
      break;
    case MachO::LC_DYLD_EXPORTS_TRIE:
      ExportsTrieCommandIndex = Index;
      break;
    }
  }
}

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 103-109
```cpp
Error Object::removeLoadCommands(
    function_ref<bool(const LoadCommand &)> ToRemove) {
  auto It = std::stable_partition(
      LoadCommands.begin(), LoadCommands.end(),
      [&](const LoadCommand &LC) { return !ToRemove(LC); });
  LoadCommands.erase(It, LoadCommands.end());

```
- **EN**: Implements logic around `removeLoadCommands`, `function_ref`, `stable_partition`, `begin`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `removeLoadCommands`, `function_ref`, `stable_partition`, `begin`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 110-123
```cpp
  updateLoadCommandIndexes();
  return Error::success();
}

Error Object::removeSections(
    function_ref<bool(const std::unique_ptr<Section> &)> ToRemove) {
  DenseMap<uint32_t, const Section *> OldIndexToSection;
  uint32_t NextSectionIndex = 1;
  for (LoadCommand &LC : LoadCommands) {
    auto It = std::stable_partition(
        std::begin(LC.Sections), std::end(LC.Sections),
        [&](const std::unique_ptr<Section> &Sec) { return !ToRemove(Sec); });
    for (auto I = LC.Sections.begin(), End = It; I != End; ++I) {
      OldIndexToSection[(*I)->Index] = I->get();
```
- **EN**: Implements logic around `updateLoadCommandIndexes`, `success`, `removeSections`, `function_ref`, and 4 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `updateLoadCommandIndexes`, `success`, `removeSections`, `function_ref`, and 4 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 124-133
```cpp
      (*I)->Index = NextSectionIndex++;
    }
    LC.Sections.erase(It, LC.Sections.end());
  }

  auto IsDead = [&](const std::unique_ptr<SymbolEntry> &S) -> bool {
    std::optional<uint32_t> Section = S->section();
    return (Section && !OldIndexToSection.count(*Section));
  };

```
- **EN**: Implements logic around `erase`, `section`, `count`.
- **CN**: 围绕 `erase`, `section`, `count` 实现具体逻辑。

### Lines 134-147
```cpp
  SmallPtrSet<const SymbolEntry *, 2> DeadSymbols;
  for (const std::unique_ptr<SymbolEntry> &Sym : SymTable.Symbols)
    if (IsDead(Sym))
      DeadSymbols.insert(Sym.get());

  for (const LoadCommand &LC : LoadCommands)
    for (const std::unique_ptr<Section> &Sec : LC.Sections)
      for (const RelocationInfo &R : Sec->Relocations)
        if (R.Symbol && *R.Symbol && DeadSymbols.count(*R.Symbol))
          return createStringError(std::errc::invalid_argument,
                                   "symbol '%s' defined in section with index "
                                   "'%u' cannot be removed because it is "
                                   "referenced by a relocation in section '%s'",
                                   (*R.Symbol)->Name.c_str(),
```
- **EN**: Implements logic around `IsDead`, `insert`, `count`, `createStringError`, and 1 more symbols; this block propagates recoverable errors through LLVM error utilities.
- **CN**: 围绕 `IsDead`, `insert`, `count`, `createStringError`, and 1 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误。

### Lines 148-156
```cpp
                                   *((*R.Symbol)->section()),
                                   Sec->CanonicalName.c_str());
  SymTable.removeSymbols(IsDead);
  for (std::unique_ptr<SymbolEntry> &S : SymTable.Symbols)
    if (S->section())
      S->n_sect = OldIndexToSection[S->n_sect]->Index;
  return Error::success();
}

```
- **EN**: Implements logic around `section`, `c_str`, `removeSymbols`, `success`.
- **CN**: 围绕 `section`, `c_str`, `removeSymbols`, `success` 实现具体逻辑。

### Lines 157-170
```cpp
uint64_t Object::nextAvailableSegmentAddress() const {
  uint64_t HeaderSize =
      is64Bit() ? sizeof(MachO::mach_header_64) : sizeof(MachO::mach_header);
  uint64_t Addr = HeaderSize + Header.SizeOfCmds;
  for (const LoadCommand &LC : LoadCommands) {
    const MachO::macho_load_command &MLC = LC.MachOLoadCommand;
    switch (MLC.load_command_data.cmd) {
    case MachO::LC_SEGMENT:
      Addr = std::max(Addr,
                      static_cast<uint64_t>(MLC.segment_command_data.vmaddr) +
                          MLC.segment_command_data.vmsize);
      break;
    case MachO::LC_SEGMENT_64:
      Addr = std::max(Addr, MLC.segment_command_64_data.vmaddr +
```
- **EN**: Implements logic around `nextAvailableSegmentAddress`, `is64Bit`, `max`, `static_cast`; this block uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `nextAvailableSegmentAddress`, `is64Bit`, `max`, `static_cast` 实现具体逻辑；该代码块使用 `switch` 风格分派，并应用目标文件格式专用规则。

### Lines 171-179
```cpp
                                MLC.segment_command_64_data.vmsize);
      break;
    default:
      continue;
    }
  }
  return Addr;
}

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 180-193
```cpp
template <typename SegmentType>
static void
constructSegment(SegmentType &Seg, llvm::MachO::LoadCommandType CmdType,
                 StringRef SegName, uint64_t SegVMAddr, uint64_t SegVMSize) {
  assert(SegName.size() <= sizeof(Seg.segname) && "too long segment name");
  memset(&Seg, 0, sizeof(SegmentType));
  Seg.cmd = CmdType;
  strncpy(Seg.segname, SegName.data(), SegName.size());
  Seg.maxprot |=
      (MachO::VM_PROT_READ | MachO::VM_PROT_WRITE | MachO::VM_PROT_EXECUTE);
  Seg.initprot |=
      (MachO::VM_PROT_READ | MachO::VM_PROT_WRITE | MachO::VM_PROT_EXECUTE);
  Seg.vmaddr = SegVMAddr;
  Seg.vmsize = SegVMSize;
```
- **EN**: Implements logic around `constructSegment`, `assert`, `memset`, `strncpy`; this block applies object-format-specific rules.
- **CN**: 围绕 `constructSegment`, `assert`, `memset`, `strncpy` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 194-205
```cpp
}

LoadCommand &Object::addSegment(StringRef SegName, uint64_t SegVMSize) {
  LoadCommand LC;
  const uint64_t SegVMAddr = nextAvailableSegmentAddress();
  if (is64Bit())
    constructSegment(LC.MachOLoadCommand.segment_command_64_data,
                     MachO::LC_SEGMENT_64, SegName, SegVMAddr, SegVMSize);
  else
    constructSegment(LC.MachOLoadCommand.segment_command_data,
                     MachO::LC_SEGMENT, SegName, SegVMAddr, SegVMSize);

```
- **EN**: Implements logic around `addSegment`, `nextAvailableSegmentAddress`, `is64Bit`, `constructSegment`; this block applies object-format-specific rules.
- **CN**: 围绕 `addSegment`, `nextAvailableSegmentAddress`, `is64Bit`, `constructSegment` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 206-215
```cpp
  LoadCommands.push_back(std::move(LC));
  return LoadCommands.back();
}

/// Extracts a segment name from a string which is possibly non-null-terminated.
static StringRef extractSegmentName(const char *SegName) {
  return StringRef(SegName,
                   strnlen(SegName, sizeof(MachO::segment_command::segname)));
}

```
- **EN**: Implements logic around `push_back`, `back`, `extractSegmentName`, `StringRef`, and 1 more symbols; this block applies object-format-specific rules.
- **CN**: 围绕 `push_back`, `back`, `extractSegmentName`, `StringRef`, and 1 more symbols 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 216-227
```cpp
std::optional<StringRef> LoadCommand::getSegmentName() const {
  const MachO::macho_load_command &MLC = MachOLoadCommand;
  switch (MLC.load_command_data.cmd) {
  case MachO::LC_SEGMENT:
    return extractSegmentName(MLC.segment_command_data.segname);
  case MachO::LC_SEGMENT_64:
    return extractSegmentName(MLC.segment_command_64_data.segname);
  default:
    return std::nullopt;
  }
}

```
- **EN**: Implements logic around `getSegmentName`, `extractSegmentName`; this block uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `getSegmentName`, `extractSegmentName` 实现具体逻辑；该代码块使用 `switch` 风格分派，并应用目标文件格式专用规则。

### Lines 228-238
```cpp
std::optional<uint64_t> LoadCommand::getSegmentVMAddr() const {
  const MachO::macho_load_command &MLC = MachOLoadCommand;
  switch (MLC.load_command_data.cmd) {
  case MachO::LC_SEGMENT:
    return MLC.segment_command_data.vmaddr;
  case MachO::LC_SEGMENT_64:
    return MLC.segment_command_64_data.vmaddr;
  default:
    return std::nullopt;
  }
}
```
- **EN**: Implements logic around `getSegmentVMAddr`; this block uses `switch`-style dispatch; applies object-format-specific rules.
- **CN**: 围绕 `getSegmentVMAddr` 实现具体逻辑；该代码块使用 `switch` 风格分派，并应用目标文件格式专用规则。

## Key Concepts / 关键概念

- **Object rewriting / 目标文件改写**:
  - **EN**: Edits sections, symbols, and metadata in existing object containers.
  - **CN**: 修改现有目标容器中的节、符号和元数据。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `MachOObject.h`, `llvm/ADT/SmallPtrSet.h`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
