# JITLinkLinker.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Rewrite/JITLinkLinker.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: BOLTLinker using JITLink. It also sits in code that implements BOLT rewriting, patching, and binary emission logic. / 该文件实现 BOLT 重写、补丁与二进制输出逻辑。 源码头部说明其职责是：BOLTLinker using JITLink。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```cpp
//===- bolt/Rewrite/JITLinkLinker.cpp - BOLTLinker using JITLink ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 9-18

```cpp
#include "bolt/Rewrite/JITLinkLinker.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Core/BinaryData.h"
#include "bolt/Core/BinarySection.h"
#include "llvm/ExecutionEngine/JITLink/ELF_riscv.h"
#include "llvm/ExecutionEngine/JITLink/JITLink.h"
#include "llvm/ExecutionEngine/Orc/Shared/ExecutorAddress.h"
#include "llvm/ExecutionEngine/Orc/Shared/ExecutorSymbolDef.h"
#include "llvm/Support/Debug.h"
```

- EN: Pulls in 9 header(s) from local project, LLVM dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 9 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 19-30

```cpp
#define DEBUG_TYPE "bolt"

namespace llvm {
namespace bolt {

namespace {

bool hasSymbols(const jitlink::Block &B) {
  return llvm::any_of(B.getSection().symbols(),
                      [&B](const auto &S) { return &S->getBlock() == &B; });
}
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `hasSymbols`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `hasSymbols`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 31-39

```cpp
/// Liveness in JITLink is based on symbols so sections that do not contain
/// any symbols will always be pruned. This pass adds anonymous symbols to
/// needed sections to prevent pruning.
Error markSectionsLive(jitlink::LinkGraph &G) {
  for (auto &Section : G.sections()) {
    // We only need allocatable sections.
    if (Section.getMemLifetime() == orc::MemLifetime::NoAlloc)
      continue;
```

- EN: Declares or implements routines including `markSectionsLive`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `markSectionsLive`.
- CN: 这里声明或实现函数，例如 `markSectionsLive`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `markSectionsLive`。

### Lines 40-48

```cpp
    // Skip empty sections.
    if (JITLinkLinker::sectionSize(Section) == 0)
      continue;

    for (auto *Block : Section.blocks()) {
      // No need to add symbols if it already has some.
      if (hasSymbols(*Block))
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 49-56

```cpp
      G.addAnonymousSymbol(*Block, /*Offset=*/0, /*Size=*/0,
                           /*IsCallable=*/false, /*IsLive=*/true);
    }
  }

  return jitlink::markAllSymbolsLive(G);
}
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 57-73

```cpp
void reassignSectionAddress(jitlink::LinkGraph &LG,
                            const BinarySection &BinSection, uint64_t Address) {
  auto *JLSection = LG.findSectionByName(BinSection.getSectionID());
  assert(JLSection && "cannot find section in LinkGraph");

  auto BlockAddress = Address;
  for (auto *Block : JITLinkLinker::orderedBlocks(*JLSection)) {
    // FIXME it would seem to make sense to align here. However, in
    // non-relocation mode, we simply use the original address of functions
    // which might not be aligned with the minimum alignment used by
    // BinaryFunction (2). Example failing test when aligning:
    // bolt/test/X86/addr32.s
    Block->setAddress(orc::ExecutorAddr(BlockAddress));
    BlockAddress += Block->getSize();
  }
}
```

- EN: Declares or implements routines including `assert`, `setAddress`, `getSize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `setAddress`, `getSize`.
- CN: 这里声明或实现函数，例如 `assert`, `setAddress`, `getSize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `setAddress`, `getSize`。

### Lines 74-83

```cpp
} // anonymous namespace

struct JITLinkLinker::Context : jitlink::JITLinkContext {
  JITLinkLinker &Linker;
  JITLinkLinker::SectionsMapper MapSections;

  Context(JITLinkLinker &Linker, JITLinkLinker::SectionsMapper MapSections)
      : JITLinkContext(&Linker.Dylib), Linker(Linker),
        MapSections(MapSections) {}
```

- EN: Works inside namespace scope `struct` to organize symbols. Introduces type definitions such as `JITLinkLinker`. Declares or implements routines including `Context`, `JITLinkContext`, `MapSections`. Notable symbols here include `JITLinkLinker`, `Context`, `JITLinkContext`, `MapSections`, `struct`.
- CN: 这里位于命名空间 `struct` 中，用于组织符号作用域。这里引入类型定义，例如 `JITLinkLinker`。这里声明或实现函数，例如 `Context`, `JITLinkContext`, `MapSections`。这里较值得关注的符号包括 `JITLinkLinker`, `Context`, `JITLinkContext`, `MapSections`, `struct`。

### Lines 84-95

```cpp
  jitlink::JITLinkMemoryManager &getMemoryManager() override {
    return *Linker.MM;
  }

  bool shouldAddDefaultTargetPasses(const Triple &TT) const override {
    // The default passes manipulate DWARF sections in a way incompatible with
    // BOLT.
    // TODO check if we can actually use these passes to remove some of the
    // DWARF manipulation done in BOLT.
    return false;
  }
```

- EN: Declares or implements routines including `getMemoryManager`, `shouldAddDefaultTargetPasses`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getMemoryManager`, `shouldAddDefaultTargetPasses`.
- CN: 这里声明或实现函数，例如 `getMemoryManager`, `shouldAddDefaultTargetPasses`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getMemoryManager`, `shouldAddDefaultTargetPasses`。

### Lines 96-105

```cpp
  Error modifyPassConfig(jitlink::LinkGraph &G,
                         jitlink::PassConfiguration &Config) override {
    Config.PrePrunePasses.push_back(markSectionsLive);
    Config.PostAllocationPasses.push_back([this](auto &G) {
      MapSections([&G](const BinarySection &Section, uint64_t Address) {
        reassignSectionAddress(G, Section, Address);
      });
      return Error::success();
    });
```

- EN: Declares or implements routines including `MapSections`, `reassignSectionAddress`. Notable symbols here include `MapSections`, `reassignSectionAddress`.
- CN: 这里声明或实现函数，例如 `MapSections`, `reassignSectionAddress`。这里较值得关注的符号包括 `MapSections`, `reassignSectionAddress`。

### Lines 106-113

```cpp
    if (G.getTargetTriple().isRISCV()) {
      Config.PostAllocationPasses.push_back(
          jitlink::createRelaxationPass_ELF_riscv());
    }

    return Error::success();
  }
```

- EN: Declares or implements routines including `createRelaxationPass_ELF_riscv`. Notable symbols here include `createRelaxationPass_ELF_riscv`.
- CN: 这里声明或实现函数，例如 `createRelaxationPass_ELF_riscv`。这里较值得关注的符号包括 `createRelaxationPass_ELF_riscv`。

### Lines 114-123

```cpp
  void notifyFailed(Error Err) override {
    errs() << "BOLT-ERROR: JITLink failed: " << Err << '\n';
    exit(1);
  }

  void
  lookup(const LookupMap &Symbols,
         std::unique_ptr<jitlink::JITLinkAsyncLookupContinuation> LC) override {
    jitlink::AsyncLookupResult AllResults;
```

- EN: Declares or implements routines including `notifyFailed`, `errs`, `exit`. Notable symbols here include `notifyFailed`, `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `notifyFailed`, `errs`, `exit`。这里较值得关注的符号包括 `notifyFailed`, `errs`, `exit`。

### Lines 124-135

```cpp
    for (const auto &Symbol : Symbols) {
      std::string SymName = (*Symbol.first).str();
      LLVM_DEBUG(dbgs() << "BOLT: looking for " << SymName << "\n");

      if (auto SymInfo = Linker.lookupSymbolInfo(SymName)) {
        LLVM_DEBUG(dbgs() << "Resolved to address 0x"
                          << Twine::utohexstr(SymInfo->Address) << "\n");
        AllResults[Symbol.first] = orc::ExecutorSymbolDef(
            orc::ExecutorAddr(SymInfo->Address), JITSymbolFlags());
        continue;
      }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`. Notable symbols here include `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`。这里较值得关注的符号包括 `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`。

### Lines 136-146

```cpp
      if (const BinaryData *I = Linker.BC.getBinaryDataByName(SymName)) {
        uint64_t Address = I->isMoved() && !I->isJumpTable()
                               ? I->getOutputAddress()
                               : I->getAddress();
        LLVM_DEBUG(dbgs() << "Resolved to address 0x"
                          << Twine::utohexstr(Address) << "\n");
        AllResults[Symbol.first] = orc::ExecutorSymbolDef(
            orc::ExecutorAddr(Address), JITSymbolFlags());
        continue;
      }
```

- EN: Declares or implements routines including `isMoved`, `getOutputAddress`, `getAddress`, `LLVM_DEBUG`, `utohexstr`, and 1 more. Notable symbols here include `isMoved`, `getOutputAddress`, `getAddress`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`.
- CN: 这里声明或实现函数，例如 `isMoved`, `getOutputAddress`, `getAddress`, `LLVM_DEBUG`, `utohexstr`, and 1 more。这里较值得关注的符号包括 `isMoved`, `getOutputAddress`, `getAddress`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`。

### Lines 147-158

```cpp
      if (Linker.BC.isGOTSymbol(SymName)) {
        if (const BinaryData *I = Linker.BC.getGOTSymbol()) {
          uint64_t Address =
              I->isMoved() ? I->getOutputAddress() : I->getAddress();
          LLVM_DEBUG(dbgs() << "Resolved to address 0x"
                            << Twine::utohexstr(Address) << "\n");
          AllResults[Symbol.first] = orc::ExecutorSymbolDef(
              orc::ExecutorAddr(Address), JITSymbolFlags());
          continue;
        }
      }
```

- EN: Declares or implements routines including `isMoved`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`. Notable symbols here include `isMoved`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`.
- CN: 这里声明或实现函数，例如 `isMoved`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`。这里较值得关注的符号包括 `isMoved`, `LLVM_DEBUG`, `utohexstr`, `ExecutorAddr`。

### Lines 159-166

```cpp
      LLVM_DEBUG(dbgs() << "Resolved to address 0x0\n");
      AllResults[Symbol.first] =
          orc::ExecutorSymbolDef(orc::ExecutorAddr(0), JITSymbolFlags());
    }

    LC->run(std::move(AllResults));
  }
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `ExecutorSymbolDef`, `run`. Notable symbols here include `LLVM_DEBUG`, `ExecutorSymbolDef`, `run`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `ExecutorSymbolDef`, `run`。这里较值得关注的符号包括 `LLVM_DEBUG`, `ExecutorSymbolDef`, `run`。

### Lines 167-174

```cpp
  Error notifyResolved(jitlink::LinkGraph &G) override {
    for (auto *Symbol : G.defined_symbols()) {
      SymbolInfo Info{Symbol->getAddress().getValue(), Symbol->getSize()};
      auto Name =
          Symbol->hasName() ? (*Symbol->getName()).str() : std::string();
      Linker.Symtab.insert({std::move(Name), Info});
    }
```

- EN: Declares or implements routines including `notifyResolved`, `getAddress`, `hasName`. Notable symbols here include `notifyResolved`, `getAddress`, `hasName`.
- CN: 这里声明或实现函数，例如 `notifyResolved`, `getAddress`, `hasName`。这里较值得关注的符号包括 `notifyResolved`, `getAddress`, `hasName`。

### Lines 175-185

```cpp
    return Error::success();
  }

  void notifyFinalized(
      jitlink::JITLinkMemoryManager::FinalizedAlloc Alloc) override {
    if (Alloc)
      Linker.Allocs.push_back(std::move(Alloc));
    ++Linker.MM->ObjectsLoaded;
  }
};
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 186-199

```cpp
JITLinkLinker::JITLinkLinker(BinaryContext &BC,
                             std::unique_ptr<ExecutableFileMemoryManager> MM)
    : BC(BC), MM(std::move(MM)) {}

JITLinkLinker::~JITLinkLinker() { cantFail(MM->deallocate(std::move(Allocs))); }

void JITLinkLinker::loadObject(MemoryBufferRef Obj,
                               SectionsMapper MapSections) {
  auto LG = jitlink::createLinkGraphFromObject(Obj, BC.getSymbolStringPool());
  if (auto E = LG.takeError()) {
    errs() << "BOLT-ERROR: JITLink failed: " << E << '\n';
    exit(1);
  }
```

- EN: Declares or implements routines including `BC`, `JITLinkLinker`, `createLinkGraphFromObject`, `errs`, `exit`. Notable symbols here include `BC`, `JITLinkLinker`, `createLinkGraphFromObject`, `errs`, `exit`.
- CN: 这里声明或实现函数，例如 `BC`, `JITLinkLinker`, `createLinkGraphFromObject`, `errs`, `exit`。这里较值得关注的符号包括 `BC`, `JITLinkLinker`, `createLinkGraphFromObject`, `errs`, `exit`。

### Lines 200-210

```cpp
  if ((*LG)->getTargetTriple().getArch() != BC.TheTriple->getArch()) {
    errs() << "BOLT-ERROR: linking object with arch "
           << (*LG)->getTargetTriple().getArchName()
           << " into context with arch " << BC.TheTriple->getArchName() << "\n";
    exit(1);
  }

  auto Ctx = std::make_unique<Context>(*this, MapSections);
  jitlink::link(std::move(*LG), std::move(Ctx));
}
```

- EN: Declares or implements routines including `errs`, `getArchName`, `exit`, `link`. Notable symbols here include `errs`, `getArchName`, `exit`, `link`.
- CN: 这里声明或实现函数，例如 `errs`, `getArchName`, `exit`, `link`。这里较值得关注的符号包括 `errs`, `getArchName`, `exit`, `link`。

### Lines 211-219

```cpp
std::optional<JITLinkLinker::SymbolInfo>
JITLinkLinker::lookupSymbolInfo(StringRef Name) const {
  auto It = Symtab.find(Name.data());
  if (It == Symtab.end())
    return std::nullopt;

  return It->second;
}
```

- EN: Declares or implements routines including `lookupSymbolInfo`. Notable symbols here include `lookupSymbolInfo`.
- CN: 这里声明或实现函数，例如 `lookupSymbolInfo`。这里较值得关注的符号包括 `lookupSymbolInfo`。

### Lines 220-228

```cpp
SmallVector<jitlink::Block *, 2>
JITLinkLinker::orderedBlocks(const jitlink::Section &Section) {
  SmallVector<jitlink::Block *, 2> Blocks(Section.blocks());
  llvm::sort(Blocks, [](const auto *LHS, const auto *RHS) {
    return LHS->getAddress() < RHS->getAddress();
  });
  return Blocks;
}
```

- EN: Declares or implements routines including `orderedBlocks`, `Blocks`, `sort`. Notable symbols here include `orderedBlocks`, `Blocks`, `sort`.
- CN: 这里声明或实现函数，例如 `orderedBlocks`, `Blocks`, `sort`。这里较值得关注的符号包括 `orderedBlocks`, `Blocks`, `sort`。

### Lines 229-236

```cpp
size_t JITLinkLinker::sectionSize(const jitlink::Section &Section) {
  size_t Size = 0;

  for (const auto *Block : orderedBlocks(Section)) {
    Size = jitlink::alignToBlock(Size, *Block);
    Size += Block->getSize();
  }
```

- EN: Declares or implements routines including `sectionSize`, `alignToBlock`, `getSize`. Notable symbols here include `sectionSize`, `alignToBlock`, `getSize`.
- CN: 这里声明或实现函数，例如 `sectionSize`, `alignToBlock`, `getSize`。这里较值得关注的符号包括 `sectionSize`, `alignToBlock`, `getSize`。

### Lines 237-241

```cpp
  return Size;
}

} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `JITLinkLinker`: class or struct interface / 类或结构体接口
- `hasSymbols`: function or method entry point / 函数或方法入口
- `markSectionsLive`: function or method entry point / 函数或方法入口
- `assert`: function or method entry point / 函数或方法入口
- `setAddress`: function or method entry point / 函数或方法入口
- `getSize`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Rewrite/JITLinkLinker.h`, `bolt/Core/BinaryContext.h`, `bolt/Core/BinaryData.h`, `bolt/Core/BinarySection.h`
- LLVM headers / LLVM 头文件: `llvm/ExecutionEngine/JITLink/ELF_riscv.h`, `llvm/ExecutionEngine/JITLink/JITLink.h`, `llvm/ExecutionEngine/Orc/Shared/ExecutorAddress.h`, `llvm/ExecutionEngine/Orc/Shared/ExecutorSymbolDef.h`, `llvm/Support/Debug.h`
- Directory context / 目录上下文: `bolt/lib/Rewrite` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Rewrite` 下的相邻文件通常与本文件协作组成对应子系统
