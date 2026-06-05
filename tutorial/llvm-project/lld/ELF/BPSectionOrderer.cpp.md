# BPSectionOrderer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `lld/ELF/BPSectionOrderer.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: BPSectionOrderer.cpp. It also sits in code that implements ELF linking components in LLD. / 该文件实现 LLD 中的 ELF 链接组件。 源码头部说明其职责是：BPSectionOrderer.cpp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- BPSectionOrderer.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-16

```cpp
#include "BPSectionOrderer.h"
#include "InputFiles.h"
#include "InputSection.h"
#include "SymbolTable.h"
#include "Symbols.h"
#include "lld/Common/BPSectionOrdererBase.inc"
#include "llvm/Support/Endian.h"
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 17-28

```cpp
using namespace llvm;
using namespace lld::elf;

namespace {
struct BPOrdererELF;
}
template <> struct lld::BPOrdererTraits<struct BPOrdererELF> {
  using Section = elf::InputSectionBase;
  using Defined = elf::Defined;
};
namespace {
struct BPOrdererELF : lld::BPOrderer<BPOrdererELF> {
```

- EN: Works inside namespace scope `llvm`, `lld` to organize symbols. Introduces type definitions such as `BPOrdererELF`, `lld`. Notable symbols here include `BPOrdererELF`, `lld`, `llvm`.
- CN: 这里位于命名空间 `llvm`, `lld` 中，用于组织符号作用域。这里引入类型定义，例如 `BPOrdererELF`, `lld`。这里较值得关注的符号包括 `BPOrdererELF`, `lld`, `llvm`。

### Lines 29-42

```cpp
  DenseMap<const InputSectionBase *, Defined *> secToSym;

  static uint64_t getSize(const Section &sec) { return sec.getSize(); }
  static bool isCodeSection(const Section &sec) {
    return sec.flags & ELF::SHF_EXECINSTR;
  }
  static StringRef getSectionName(const Section &sec) { return sec.name; }
  ArrayRef<Defined *> getSymbols(const Section &sec) {
    auto it = secToSym.find(&sec);
    if (it == secToSym.end())
      return {};
    return ArrayRef(it->second);
  }
```

- EN: Declares or implements routines including `getSize`, `isCodeSection`, `getSectionName`, `getSymbols`. Notable symbols here include `getSize`, `isCodeSection`, `getSectionName`, `getSymbols`.
- CN: 这里声明或实现函数，例如 `getSize`, `isCodeSection`, `getSectionName`, `getSymbols`。这里较值得关注的符号包括 `getSize`, `isCodeSection`, `getSectionName`, `getSymbols`。

### Lines 43-55

```cpp
  static void
  getSectionHashes(const Section &sec, SmallVectorImpl<uint64_t> &hashes,
                   const DenseMap<const void *, uint64_t> &sectionToIdx) {
    constexpr unsigned windowSize = 4;

    // Calculate content hashes: k-mers and the last k-1 bytes.
    ArrayRef<uint8_t> data = sec.content();
    if (data.size() >= windowSize)
      for (size_t i = 0; i <= data.size() - windowSize; ++i)
        hashes.push_back(support::endian::read32le(data.data() + i));
    for (uint8_t byte : data.take_back(windowSize - 1))
      hashes.push_back(byte);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 56-65

```cpp
    llvm::sort(hashes);
    hashes.erase(llvm::unique(hashes), hashes.end());
  }

  static StringRef getSymName(const Defined &sym) { return sym.getName(); }
  static uint64_t getSymValue(const Defined &sym) { return sym.value; }
  static uint64_t getSymSize(const Defined &sym) { return sym.size; }
};
} // namespace
```

- EN: Declares or implements routines including `sort`, `getSymName`, `getSymValue`, `getSymSize`. Notable symbols here include `sort`, `getSymName`, `getSymValue`, `getSymSize`.
- CN: 这里声明或实现函数，例如 `sort`, `getSymName`, `getSymValue`, `getSymSize`。这里较值得关注的符号包括 `sort`, `getSymName`, `getSymValue`, `getSymSize`。

### Lines 66-75

```cpp
DenseMap<const InputSectionBase *, int> elf::runBalancedPartitioning(
    Ctx &ctx, StringRef profilePath,
    ArrayRef<BPCompressionSortSpec> compressionSortSpecs,
    bool forFunctionCompression, bool forDataCompression,
    bool compressionSortStartupFunctions, bool verbose) {
  // Collect candidate sections and associated symbols.
  SmallVector<InputSectionBase *> sections;
  DenseMap<CachedHashStringRef, std::set<unsigned>> rootSymbolToSectionIdxs;
  BPOrdererELF orderer;
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 76-92

```cpp
  auto addSection = [&](Symbol &sym) {
    auto *d = dyn_cast<Defined>(&sym);
    if (!d)
      return;
    auto *sec = dyn_cast_or_null<InputSection>(d->section);
    // Skip section symbols. Skip empty, discarded, ICF folded sections, .bss.
    // ICF folded sections are already dead (!isLive()), so no separate check
    // is needed.
    if (sym.isSection() || !sec || sec->size == 0 || !sec->isLive() ||
        !sec->content().data() || !orderer.secToSym.try_emplace(sec, d).second)
      return;
    rootSymbolToSectionIdxs[CachedHashStringRef(
                                lld::utils::getRootSymbol(sym.getName()))]
        .insert(sections.size());
    sections.emplace_back(sec);
  };
```

- EN: Declares or implements routines including `content`, `getRootSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `content`, `getRootSymbol`.
- CN: 这里声明或实现函数，例如 `content`, `getRootSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `content`, `getRootSymbol`。

### Lines 93-102

```cpp
  for (Symbol *sym : ctx.symtab->getSymbols())
    addSection(*sym);
  for (ELFFileBase *file : ctx.objectFiles)
    for (Symbol *sym : file->getLocalSymbols())
      addSection(*sym);
  return orderer.computeOrder(profilePath, compressionSortSpecs,
                              forFunctionCompression, forDataCompression,
                              compressionSortStartupFunctions, verbose,
                              sections, rootSymbolToSectionIdxs);
}
```

- EN: Declares or implements routines including `addSection`. Notable symbols here include `addSection`.
- CN: 这里声明或实现函数，例如 `addSection`。这里较值得关注的符号包括 `addSection`。

## Key Concepts / 关键概念

- `BPOrdererELF`: class or struct interface / 类或结构体接口
- `lld`: class or struct interface / 类或结构体接口
- `getSize`: function or method entry point / 函数或方法入口
- `isCodeSection`: function or method entry point / 函数或方法入口
- `getSectionName`: function or method entry point / 函数或方法入口
- `getSymbols`: function or method entry point / 函数或方法入口
- `sort`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `lld/Common/BPSectionOrdererBase.inc`
- LLVM headers / LLVM 头文件: `llvm/Support/Endian.h`
- System headers / 系统头文件: `BPSectionOrderer.h`, `InputFiles.h`, `InputSection.h`, `SymbolTable.h`, `Symbols.h`
- Directory context / 目录上下文: `lld/ELF` neighbors usually cooperate with this file to provide the surrounding subsystem / `lld/ELF` 下的相邻文件通常与本文件协作组成对应子系统
