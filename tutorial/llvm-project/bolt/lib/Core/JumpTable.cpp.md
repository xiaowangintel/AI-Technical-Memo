# JumpTable.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/JumpTable.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Jump table at low-level IR. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Jump table at low-level IR。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/JumpTable.cpp - Jump table at low-level IR ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the JumpTable class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-22

```cpp
#include "bolt/Core/JumpTable.h"
#include "bolt/Core/BinaryFunction.h"
#include "bolt/Core/BinarySection.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "bolt"

using namespace llvm;
using namespace bolt;
```

- EN: Pulls in 4 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 23-35

```cpp
using JumpTable = bolt::JumpTable;

namespace opts {
extern cl::opt<JumpTableSupportLevel> JumpTables;
extern cl::opt<unsigned> Verbosity;
} // namespace opts

bolt::JumpTable::JumpTable(MCSymbol &Symbol, uint64_t Address, size_t EntrySize,
                           JumpTableType Type, LabelMapType &&Labels,
                           BinarySection &Section)
    : BinaryData(Symbol, Address, 0, EntrySize, Section), EntrySize(EntrySize),
      OutputEntrySize(EntrySize), Type(Type), Labels(Labels) {}
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `BinaryData`, `OutputEntrySize`. Notable symbols here include `BinaryData`, `OutputEntrySize`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `BinaryData`, `OutputEntrySize`。这里较值得关注的符号包括 `BinaryData`, `OutputEntrySize`, `opts`。

### Lines 36-45

```cpp
std::pair<size_t, size_t>
bolt::JumpTable::getEntriesForAddress(const uint64_t Addr) const {
  // Check if this is not an address, but a cloned JT id
  if ((int64_t)Addr < 0ll)
    return std::make_pair(0, Entries.size());

  const uint64_t InstOffset = Addr - getAddress();
  size_t StartIndex = 0, EndIndex = 0;
  uint64_t Offset = 0;
```

- EN: Declares or implements routines including `getEntriesForAddress`, `getAddress`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getEntriesForAddress`, `getAddress`.
- CN: 这里声明或实现函数，例如 `getEntriesForAddress`, `getAddress`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getEntriesForAddress`, `getAddress`。

### Lines 46-63

```cpp
  for (size_t I = 0; I < Entries.size(); ++I) {
    auto LI = Labels.find(Offset);
    if (LI != Labels.end()) {
      const auto NextLI = std::next(LI);
      const uint64_t NextOffset =
          NextLI == Labels.end() ? getSize() : NextLI->first;
      if (InstOffset >= LI->first && InstOffset < NextOffset) {
        StartIndex = I;
        EndIndex = I;
        while (Offset < NextOffset) {
          ++EndIndex;
          Offset += EntrySize;
        }
        break;
      }
    }
    Offset += EntrySize;
  }
```

- EN: Declares or implements routines including `next`. Notable symbols here include `next`.
- CN: 这里声明或实现函数，例如 `next`。这里较值得关注的符号包括 `next`。

### Lines 64-81

```cpp

  return std::make_pair(StartIndex, EndIndex);
}

bool bolt::JumpTable::replaceDestination(uint64_t JTAddress,
                                         const MCSymbol *OldDest,
                                         MCSymbol *NewDest) {
  bool Patched = false;
  const std::pair<size_t, size_t> Range = getEntriesForAddress(JTAddress);
  for (auto I = Range.first; I != Range.second; ++I) {
    if (Entries[I] == OldDest) {
      Patched = true;
      Entries[I] = NewDest;
    }
  }
  return Patched;
}
```

- EN: Declares or implements routines including `getEntriesForAddress`. Notable symbols here include `getEntriesForAddress`.
- CN: 这里声明或实现函数，例如 `getEntriesForAddress`。这里较值得关注的符号包括 `getEntriesForAddress`。

### Lines 82-99

```cpp
void bolt::JumpTable::updateOriginal() {
  BinaryContext &BC = getSection().getBinaryContext();
  const uint64_t BaseOffset = getAddress() - getSection().getAddress();
  uint64_t EntryOffset = BaseOffset;
  for (MCSymbol *Entry : Entries) {
    const uint32_t RelType =
        Type == JTT_NORMAL ? ELF::R_X86_64_64 : ELF::R_X86_64_PC32;
    const uint64_t RelAddend =
        Type == JTT_NORMAL ? 0 : EntryOffset - BaseOffset;
    // Replace existing relocation with the new one to allow any modifications
    // to the original jump table.
    if (BC.HasRelocations)
      getOutputSection().removeRelocationAt(EntryOffset);
    getOutputSection().addRelocation(EntryOffset, Entry, RelType, RelAddend);
    EntryOffset += EntrySize;
  }
}
```

- EN: Declares or implements routines including `updateOriginal`, `getSection`, `getAddress`, `getOutputSection`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `updateOriginal`, `getSection`, `getAddress`, `getOutputSection`.
- CN: 这里声明或实现函数，例如 `updateOriginal`, `getSection`, `getAddress`, `getOutputSection`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `updateOriginal`, `getSection`, `getAddress`, `getOutputSection`。

### Lines 100-117

```cpp
void bolt::JumpTable::print(raw_ostream &OS) const {
  uint64_t Offset = 0;
  if (Type == JTT_PIC)
    OS << "PIC ";
  ListSeparator LS;

  OS << "Jump table " << getName() << " for function ";
  for (BinaryFunction *Frag : Parents)
    OS << LS << *Frag;
  OS << " at 0x" << Twine::utohexstr(getAddress()) << " with a total count of "
     << Count << ":\n";
  for (const uint64_t EntryAddress : EntriesAsAddress)
    OS << "  absolute offset: 0x" << Twine::utohexstr(EntryAddress) << '\n';
  for (const MCSymbol *Entry : Entries) {
    auto LI = Labels.find(Offset);
    if (Offset && LI != Labels.end()) {
      OS << "Jump Table " << LI->second->getName() << " at 0x"
         << Twine::utohexstr(getAddress() + Offset)
```

- EN: Declares or implements routines including `print`, `getName`, `utohexstr`. Notable symbols here include `print`, `getName`, `utohexstr`.
- CN: 这里声明或实现函数，例如 `print`, `getName`, `utohexstr`。这里较值得关注的符号包括 `print`, `getName`, `utohexstr`。

### Lines 118-129

```cpp
         << " (possibly part of larger jump table):\n";
    }
    OS << format("  0x%04" PRIx64 " : ", Offset) << Entry->getName();
    if (!Counts.empty()) {
      OS << " : " << Counts[Offset / EntrySize].Mispreds << "/"
         << Counts[Offset / EntrySize].Count;
    }
    OS << '\n';
    Offset += EntrySize;
  }
  OS << "\n\n";
}
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

## Key Concepts / 关键概念

- `BinaryData`: function or method entry point / 函数或方法入口
- `OutputEntrySize`: function or method entry point / 函数或方法入口
- `getEntriesForAddress`: function or method entry point / 函数或方法入口
- `getAddress`: function or method entry point / 函数或方法入口
- `next`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/JumpTable.h`, `bolt/Core/BinaryFunction.h`, `bolt/Core/BinarySection.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
